---
title: "Badger Memtable刷入流程分析"
date: 2022-12-01T16:44:55+08:00
draft: true
original: true
categories: 
  - 存储
tags: 
  - Badger
---

[上一篇文章](./badger写入流程源码分析)中已经知道，当memtable满了的时候，会将memtable写入到flushChan中，badger启动的时候会启动协程处理flushChan。

```go
func Open(opt Options) (*DB, error) {
  go func() {
    _ = db.flushMemtable(db.closers.memtable)
  }()
}
```

刷入之前可能会根据MemTableSize选择多个memtable一起刷入

```go
func (db *DB) flushMemtable(lc *z.Closer) error {
  for ft := range db.flushChan {
    // 可能会选择多个memtable一起刷入
    slurp()
    
    for {
      err := db.handleFlushTask(ft)
      if err == nil {
        db.lock.Lock()
        // 刷入成功后，删除内存中的memtable
        for _, mt := range mts {
          y.AssertTrue(mt == db.imm[0])
          db.imm = db.imm[1:]
          mt.DecrRef()
        }
        db.lock.Unlock()

        break
      }
      time.Sleep(time.Second)
    }
  }
  return nil
}
```


```go
// handleFlushTask must be run serially.
func (db *DB) handleFlushTask(ft flushTask) error {
  builder := buildL0Table(ft, bopts)
  defer builder.Close()

  fileID := db.lc.reserveFileID()
  tbl, err = table.CreateTable(table.NewFilename(fileID, db.opt.Dir), builder)

  err = db.lc.addLevel0Table(tbl)
  _ = tbl.DecrRef()
  return err
}
```


```go
func buildL0Table(ft flushTask, bopts table.Options) *table.Builder {
	b := table.NewTableBuilder(bopts)
	for iter.Rewind(); iter.Valid(); iter.Next() {
		vs := iter.Value()
		var vp valuePointer
		if vs.Meta&bitValuePointer > 0 {
			vp.Decode(vs.Value)
		}
		b.Add(iter.Key(), iter.Value(), vp.Len)
	}

	return b
}

func (b *Builder) Add(key []byte, value y.ValueStruct, valueLen uint32) {
  b.addInternal(key, value, valueLen, false)
}

func (b *Builder) addInternal(key []byte, value y.ValueStruct, valueLen uint32, isState bool) {
  if b.shouldFinishBlock(key, value) {
    b.finishBlock()
    b.curBlock = &bblock{
      data: b.alloc.Allocate(b.opts.BlockSize + padding),
    }
  }
  b.addHelper(key, value, valueLen)
}
```

```go
func (b *Builder) addHelper(key []byte, v y.ValueStruct, vpLen uint32) {
  if version := y.ParseTs(key); version > b.maxVersion {
    b.maxVersion = version
  }

  var diffKey []byte
  if len(b.curBlock.baseKey) == 0 {
    b.curBlock.baseKey = append(b.curBlock.baseKey[:0], key...)
    diffKey = key
  } else {
    diffKey = b.keyDiff(key)
  }

  h := header{
    overlap: uint16(len(key) - len(diffKey)),
    diff:    uint16(len(diffKey)),
  }

  b.curBlock.entryOffsets = append(b.curBlock.entryOffsets, uint32(b.curBlock.end))

  b.append(h.Encode())
  b.append(diffKey)

  dst := b.allocate(int(v.EncodedSize()))
  v.Encode(dst)

  b.onDiskSize += vpLen
}
```