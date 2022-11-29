---
title: "Badger源码分析"
date: 2022-11-28T10:56:34+08:00
draft: true
original: true
categories: 
  - 存储
tags: 
  - Badger
---

简单使用
```go
path := "badger-test"
opt := badger.DefaultOptions(path)
db, err := badger.Open(opt)
if err != nil {
  log.Fatal(err)
}

err = db.Update(func(txn *badger.Txn) error {
  err := txn.Set([]byte("answer"), []byte("42"))
  return err
})
if err != nil {
  log.Fatal(err)
}

err = db.View(func(txn *badger.Txn) error {
  item, err := txn.Get([]byte("answer"))
  if err != nil {
    return err
  }
  err = item.Value(func(val []byte) error {
    fmt.Printf("The answer is: %s\n", val)
    return nil
  })
  if err != nil {
    return err
  }
  return nil
})
if err != nil {
  log.Fatal(err)
}
```

Update逻辑

创建Transaction
```go
func (db *DB) Update(fn func(txn *Txn) error) error {
  txn := db.NewTransaction(true)
  if err := fn(txn); err != nil {
    return err
  }
  return txn.Commit()
}
```

创建pendingWrites的map，txn的所有写入操作都会以`key->Entry`的形式保存在这个map中
```go
func (db *DB) newTransaction(update, isManaged bool) *Txn {
  txn := &Txn{
    db:     db,
    update: update,
  }
  if update {
    txn.pendingWrites = make(map[string]*Entry)
  }
  txn.readTs = db.orc.readTs()
  return txn
}
```

最后会由`oracle.nextTxnTs`初始化一个readTs,`oracle.nextTxnTs`是一个递增的id generator,txn的readTs和commitTs都是它创建

```go
func (o *oracle) readTs() uint64 {
	o.Lock()
	readTs := o.nextTxnTs - 1
	o.readMark.Begin(readTs)
	o.Unlock()

	o.txnMark.WaitForMark(context.Background(), readTs)
	return readTs
}
```

在这里调用了`txnMark.WaitForMark`,而且这个方法当txnMark的doneUntil < readTs时,会存在阻塞。

```go
func (w *WaterMark) WaitForMark(ctx context.Context, index uint64) error {
	if w.DoneUntil() >= index {
		return nil
	}

	waitCh := make(chan struct{})
	w.markCh <- mark{index: index, waiter: waitCh}

	select {
	case <-ctx.Done():
		return ctx.Err()
	case <-waitCh:
		return nil
	}
}
```

而txnMark的doneUntil是在Commit之后,将数据写入到vlog和LSM中后在txnMark的process循环逻辑中修改的。

```go
func (o *oracle) doneCommit(cts uint64) {
  o.txnMark.Done(cts)
}

func (w *WaterMark) Done(index uint64) {
  w.markCh <- mark{index: index, done: true}
}
```

```go
func (txn *Txn) Commit() error {
  txnCb, err := txn.commitAndSend()
  if err != nil {
    return err
  }
  return txnCb()
}
```

```go
func (txn *Txn) commitAndSend() (func() error, error) {
  commitTs, conflict := orc.newCommitTs(txn)
  req, err := txn.db.sendToWriteCh(entries) 
  if err != nil {
    orc.doneCommit(commitTs)
    return nil, err
  }
  ret := func() error {
    err := req.Wait()
    orc.doneCommit(commitTs)
    return err
  }
  return ret, nil
}
```

```go
func (o *oracle) newCommitTs(txn *Txn) (uint64, bool) {
  o.doneRead(txn)
  ts = o.nextTxnTs
  o.nextTxnTs++
  o.txnMark.Begin(ts)

  return ts, false
}
```


```go
func (db *DB) sendToWriteCh(entries []*Entry) (*request, error){
  req.Entries = entries
  req.Wg.Add(1)
  db.writeCh <- req
  return req, nil
}
```

```go
func (db *DB) doWrites(lc *z.Closer){
  writeRequests := func(reqs []*request) {
    if err := db.writeRequests(reqs); err != nil {
      db.opt.Errorf("writeRequests: %v", err)
    }
    <-pendingCh
  }


  writeCase:
    go writeRequests(reqs)
}
```

```go
func (db *DB) writeRequests(reqs []*request) error {
  done := func(err error) {
    for _, r := range reqs {
      r.Err = err
      r.Wg.Done()
    }
  }

  err := db.vlog.write(reqs)
  if err != nil {
    done(err)
    return err
  }

  for _, b := range reqs {
    if err := db.writeToLSM(b); err != nil {
      done(err)
      return y.Wrapf(err, "writeRequests")
    }
  }
  done(nil)
  return nil
}
```

所以从这段逻辑中得到如下结论：

1. 并发高的情况下，txnMark的doneUntil还没变化时，初始化了很多txn
   1. 这些txn的readTs其实是一样的, 并且不会被`txnMark.WaitForMark`住
2. 同一个txn的commitTs > readTs。而且`readTs := o.nextTxnTs - 1`,所以`txnMark.WaitForMark`不会阻塞最新的1个txn
3. 当上一个txn调用`orc.newCommitTs(txn)`(nextTxnTs增加)，但是上一个txn还没有调用Commit时，这段时间内新创建的txn就会被`txnMark.WaitForMark`阻塞住。
   1. 阻塞的时间是`orc.newCommitTs(txn)`调用到数据被写入到vlog和LSM中的时间，由于vlog是顺序写磁盘，LSM是内存操作，所以这个时间段是比较短的
4. 按照这个逻辑，可能会多个txn的readTs相同，但是commitTs不同的数据会写入，所以一定会存在不同txn对同一个key写入不同value的冲突?这个问题怎么解决？


看完了Update的大概流程，就下来深入了解写入时候具体细节。

从Commit开始就是写入逻辑了，txn中的Update都会变成一个一个的Entry，
1. 首先会将Entry#version设置成commitTs，
2. 并且Entry#Key会使用`y.KeyWithTs`重新编码一下Key
3. 在所有的Entry后面再添加一个特殊的Entry表示这个txn的变动的结束

```go
func (txn *Txn) commitAndSend() (func() error, error) {
  commitTs, conflict := orc.newCommitTs(txn)
  if conflict {
    return nil, ErrConflict
  }

  keepTogether := true
  setVersion := func(e *Entry) {
    if e.version == 0 {
      e.version = commitTs
    } else {
      keepTogether = false
    }
  }
  for _, e := range txn.pendingWrites {
    setVersion(e)
  }

  entries := make([]*Entry, 0, len(txn.pendingWrites))

  processEntry := func(e *Entry) {
    e.Key = y.KeyWithTs(e.Key, e.version)
    entries = append(entries, e)
  }
  for _, e := range txn.pendingWrites {
    processEntry(e)
  }

  if keepTogether {
    e := &Entry{ // 结束标志entry
      Key:   y.KeyWithTs(txnKey, commitTs),
      Value: []byte(strconv.FormatUint(commitTs, 10)),
      meta:  bitFinTxn,
    }
    entries = append(entries, e)
  }
}
```

这些entries会被包装成request，写入到writeCh中

```go
func (db *DB) sendToWriteCh(entries []*Entry) (*request, error) {
  if atomic.LoadInt32(&db.blockWrites) == 1 {
    return nil, ErrBlockWrites
  }
  req := requestPool.Get().(*request)
  req.reset()
  req.Entries = entries
  req.Wg.Add(1)
  db.writeCh <- req
  return req, nil
}
```

在DB#doWrites方法中会不断的读取writeCh中的request进行处理。

这个for循环的逻辑比较简单：
1. 将request合并成[]request数组给`DB#writeRequests`一起处理
2. 保证`DB#writeRequests`是串行的，通过`pendingCh`实现的
3. 通过 `case r = <-db.writeCh`, `case pendingCh <- struct{}{}`的随机性和`len(reqs) >= 3*kvWriteChCapacity`处理request的合并。也就是说`len(reqs)<3*kvWriteChCapacity`时，有一定概率会走到`DB#writeRequests`，`len(reqs)>=3*kvWriteChCapacity`时一定是走到`DB#writeRequests`

```go
func (db *DB) doWrites(lc *z.Closer) {
  pendingCh := make(chan struct{}, 1)

  writeRequests := func(reqs []*request) {
    if err := db.writeRequests(reqs); err != nil {
      db.opt.Errorf("writeRequests: %v", err)
    }
    <-pendingCh // 写完后解除pending
  }

  reqs := make([]*request, 0, 10)
  for {
    var r *request
    select {
    case r = <-db.writeCh:
    case <-lc.HasBeenClosed():
      goto closedCase
    }

    for {
      reqs = append(reqs, r)
      if len(reqs) >= 3*kvWriteChCapacity {
        pendingCh <- struct{}{} // 阻塞
        goto writeCase
      }

      select {
      // 因为select case是随机的，所以要么从writeCh中pick，要么就push pending,进行写入
      case r = <-db.writeCh:
      case pendingCh <- struct{}{}:
        goto writeCase
      case <-lc.HasBeenClosed():
        goto closedCase
      }
    }

  closedCase:
    for {
      select { // 当close的时候，从writeCh中读取出所有的request进行写入
      case r = <-db.writeCh:
        reqs = append(reqs, r)
      default:
        pendingCh <- struct{}{} // 在写入之前 push to pending，控制串行的调用 writeRequests 进行写入
        writeRequests(reqs)
        return
      }
    }

  writeCase:
    go writeRequests(reqs)
    reqs = make([]*request, 0, 10) // 重制reqs缓存数组
  }
}
```

主要是写入vlog和LSM(memTable)

```go
func (db *DB) writeRequests(reqs []*request) error {
  // vlog保存所有的key/value，落盘
  err := db.vlog.write(reqs)
  if err != nil {
    done(err)
    return err
  }

  var count int
  for _, b := range reqs {
    if len(b.Entries) == 0 {
      continue
    }
    count += len(b.Entries)
    var err error
    for err = db.ensureRoomForWrite(); err == errNoRoom; err = db.ensureRoomForWrite() {
      time.Sleep(10 * time.Millisecond)
    }

    if err := db.writeToLSM(b); err != nil {
      done(err)
      return y.Wrapf(err, "writeRequests")
    }
  }
}
```

首先是写入vlog

```go
func (vlog *valueLog) write(reqs []*request) error {
  buf := new(bytes.Buffer)
  for i := range reqs {
    b := reqs[i]
    b.Ptrs = b.Ptrs[:0]

    for j := range b.Entries {
      buf.Reset()
      e := b.Entries[j]

      var p valuePointer
      p.Fid = curlf.fid
      p.Offset = vlog.woffset()

      plen, err := curlf.encodeEntry(buf, e, 0)
      if err != nil {
        return err
      }

      p.Len = uint32(plen)
      b.Ptrs = append(b.Ptrs, p)
      if err := write(buf); err != nil {
        return err
      }
    }
  }
}

// write方法，当endOffset没有达到len(curlf.Data)时，不会进行刷盘，靠mmap机制进行刷盘
// 当endOffset达到len(curlf.Data)时，会调用Truncate方法，进行刷盘
write := func(buf *bytes.Buffer) error {
  if buf.Len() == 0 {
    return nil
  }
  n := uint32(buf.Len())
  endOffset := atomic.AddUint32(&vlog.writableLogOffset, n)
  // 说明当前logFile写满了，但是为了保证同一txn中的Entry写入到同一个logFile中，继续写入文件
  if int(endOffset) >= len(curlf.Data) { // len(curlf.Data)是文件open时传入的size，2*vlog.opt.ValueLogFileSize
    curlf.Truncate(int64(endOffset)) // Truncate内部会调用Sync写入到磁盘
  }
  start := endOffset - n
  copy(curlf.Data[start:], buf.Bytes())

  atomic.StoreUint32(&curlf.size, endOffset)
  return nil
}

toDisk := func() error {
  if vlog.woffset() > uint32(vlog.opt.ValueLogFileSize) ||
    vlog.numEntriesWritten > vlog.opt.ValueLogMaxEntries {
    if err := curlf.doneWriting(vlog.woffset()); err != nil {
      return err
    }
    newlf, err := vlog.createVlogFile()
    if err != nil {
      return err
    }
    curlf = newlf
  }
  return nil
}
```

```go
// encodeEntry will encode entry to the buf
// layout of entry
// +--------+-----+-------+-------+
// | header | key | value | crc32 |
// +--------+-----+-------+-------+
func (lf *logFile) encodeEntry(buf *bytes.Buffer, e *Entry, offset uint32) (int, error) {
  h := header{
    klen:      uint32(len(e.Key)),
    vlen:      uint32(len(e.Value)),
    expiresAt: e.ExpiresAt,
    meta:      e.meta,
    userMeta:  e.UserMeta,
  }

  hash := crc32.New(y.CastanoliCrcTable)
  writer := io.MultiWriter(buf, hash)

  var headerEnc [maxHeaderSize]byte
  sz := h.Encode(headerEnc[:])
  y.Check2(writer.Write(headerEnc[:sz]))

  y.Check2(writer.Write(e.Key))
  y.Check2(writer.Write(e.Value))

  var crcBuf [crc32.Size]byte
  binary.BigEndian.PutUint32(crcBuf[:], hash.Sum32())
  y.Check2(writer.Write(crcBuf[:]))
  return len(headerEnc[:sz]) + len(e.Key) + len(e.Value) + len(crcBuf), nil
}

// Encode encodes the header into []byte. The provided []byte should be atleast 5 bytes. The
// function will panic if out []byte isn't large enough to hold all the values.
// The encoded header looks like
// +------+----------+------------+--------------+-----------+
// | Meta | UserMeta | Key Length | Value Length | ExpiresAt |
// +------+----------+------------+--------------+-----------+
func (h header) Encode(out []byte) int {
  out[0], out[1] = h.meta, h.userMeta
  index := 2
  index += binary.PutUvarint(out[index:], uint64(h.klen))
  index += binary.PutUvarint(out[index:], uint64(h.vlen))
  index += binary.PutUvarint(out[index:], h.expiresAt)
  return index
}
```

Entry在logFile中的结构

![](/Badger源码分析/EntryInLogFile.png)