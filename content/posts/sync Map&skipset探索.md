---
title: "Sync Map&skipset探索"
date: 2021-07-31T01:13:14+08:00
draft: false
original: true
categories: 
  - Golang 
tags: 
  - Golang源码
---


# 背景

最近看到一个开源项目([https://github.com/zhangyunhao116/skipset](https://github.com/zhangyunhao116/skipset))，在readme中有如下一段描述

![](/syncMap&skipset探索/snapshot.png)

意思是大多数情况下skipset比sync.Map的性能好，特别是并发的多个操作，而且节省50%的内存占用

哟哟，看这个表述感觉挺屌的样子，我自己来跑一跑bencmark

<!--more-->

```go
go test -bench=. -run=none
goos: darwin
goarch: amd64
pkg: github.com/zhangyunhao116/skipset
cpu: Intel(R) Core(TM) i7-4750HQ CPU @ 2.00GHz
BenchmarkInt64/Add/skipset-8         	 4139220	       390.2 ns/op
BenchmarkInt64/Add/sync.Map-8        	 1316097	       796.0 ns/op
BenchmarkInt64/Contains50Hits/skipset-8         	76820310	        16.01 ns/op
BenchmarkInt64/Contains50Hits/sync.Map-8        	74427480	        15.66 ns/op
BenchmarkInt64/30Add70Contains/skipset-8        	10846078	       229.4 ns/op
BenchmarkInt64/30Add70Contains/sync.Map-8       	 1641924	       814.8 ns/op
BenchmarkInt64/1Remove9Add90Contains/skipset-8  	17981565	       159.5 ns/op
BenchmarkInt64/1Remove9Add90Contains/sync.Map-8 	 1810402	       693.8 ns/op
BenchmarkInt64/1Range9Remove90Add900Contains/skipset-8         	 5693479	      1773 ns/op
BenchmarkInt64/1Range9Remove90Add900Contains/sync.Map-8        	 1000000	      7876 ns/op
BenchmarkString/Add/skipset-8                                  	 3339792	       448.2 ns/op
BenchmarkString/Add/sync.Map-8                                 	 1000000	      1196 ns/op
BenchmarkString/Contains50Hits/skipset-8                       	27357584	        42.01 ns/op
BenchmarkString/Contains50Hits/sync.Map-8                      	32648217	        34.42 ns/op
BenchmarkString/30Add70Contains/skipset-8                      	 6355838	       273.8 ns/op
BenchmarkString/30Add70Contains/sync.Map-8                     	 1231134	      1000 ns/op
BenchmarkString/1Remove9Add90Contains/skipset-8                	11251975	       198.7 ns/op
BenchmarkString/1Remove9Add90Contains/sync.Map-8               	 1567581	       801.1 ns/op
BenchmarkString/1Range9Remove90Add900Contains/skipset-8        	 4519983	      1490 ns/op
BenchmarkString/1Range9Remove90Add900Contains/sync.Map-8       	 1000000	     10476 ns/op
PASS
ok  	github.com/zhangyunhao116/skipset	66.633s
```

```go
go test -bench=. -benchmem  -run=none
goos: darwin
goarch: amd64
pkg: github.com/zhangyunhao116/skipset
cpu: Intel(R) Core(TM) i7-4750HQ CPU @ 2.00GHz
BenchmarkInt64/Add/skipset-8         	 4202815	       423.1 ns/op	      64 B/op	       1 allocs/op
BenchmarkInt64/Add/sync.Map-8        	 1342520	       785.6 ns/op	     140 B/op	       4 allocs/op
BenchmarkInt64/Contains50Hits/skipset-8         	78394792	        14.66 ns/op	       0 B/op	       0 allocs/op
BenchmarkInt64/Contains50Hits/sync.Map-8        	80596683	        12.83 ns/op	       0 B/op	       0 allocs/op
BenchmarkInt64/30Add70Contains/skipset-8        	10422588	       218.8 ns/op	      19 B/op	       0 allocs/op
BenchmarkInt64/30Add70Contains/sync.Map-8       	 1614126	       736.2 ns/op	      73 B/op	       1 allocs/op
BenchmarkInt64/1Remove9Add90Contains/skipset-8  	17636092	       159.1 ns/op	       5 B/op	       0 allocs/op
BenchmarkInt64/1Remove9Add90Contains/sync.Map-8 	 2004118	       663.7 ns/op	      55 B/op	       0 allocs/op
BenchmarkInt64/1Range9Remove90Add900Contains/skipset-8         	 2605401	       741.3 ns/op	       5 B/op	       0 allocs/op
BenchmarkInt64/1Range9Remove90Add900Contains/sync.Map-8        	 1000000	      7596 ns/op	    2341 B/op	       0 allocs/op
BenchmarkString/Add/skipset-8                                  	 4148596	       450.2 ns/op	      96 B/op	       2 allocs/op
BenchmarkString/Add/sync.Map-8                                 	 1000000	      1182 ns/op	     195 B/op	       5 allocs/op
BenchmarkString/Contains50Hits/skipset-8                       	25208853	        42.15 ns/op	      15 B/op	       1 allocs/op
BenchmarkString/Contains50Hits/sync.Map-8                      	32053101	        33.92 ns/op	      15 B/op	       1 allocs/op
BenchmarkString/30Add70Contains/skipset-8                      	 6945487	       264.6 ns/op	      39 B/op	       1 allocs/op
BenchmarkString/30Add70Contains/sync.Map-8                     	 1319289	       848.7 ns/op	      79 B/op	       2 allocs/op
BenchmarkString/1Remove9Add90Contains/skipset-8                	11410796	       193.3 ns/op	      23 B/op	       1 allocs/op
BenchmarkString/1Remove9Add90Contains/sync.Map-8               	 1614619	       797.1 ns/op	      73 B/op	       1 allocs/op
BenchmarkString/1Range9Remove90Add900Contains/skipset-8        	 4616151	      1541 ns/op	      23 B/op	       1 allocs/op
BenchmarkString/1Range9Remove90Add900Contains/sync.Map-8       	 1000000	     10453 ns/op	    2158 B/op	       1 allocs/op
PASS
ok  	github.com/zhangyunhao116/skipset	57.953s
```

从数据来看，整体上skipset确实比sync.Map好些，但是从数据中可以发现sync.Map在读多写少的情况下确实比skipset好；

于是我就有了2个问题

1. sync.Map底层实现是怎样的，为啥读多写少的情况下NB，写变多了之后为啥就不行了呢
2. skipset为啥在写多的时候性能比sync.Map好呢，它底层又是怎么做的呢

# sync.Map

首先看sync.Map的结构

```go
// Map is like a Go map[interface{}]interface{} but is safe for concurrent use
// by multiple goroutines without additional locking or coordination.
// Loads, stores, and deletes run in amortized constant time.
//
// The Map type is specialized. Most code should use a plain Go map instead,
// with separate locking or coordination, for better type safety and to make it
// easier to maintain other invariants along with the map content.
//
// The Map type is optimized for two common use cases: (1) when the entry for a given
// key is only ever written once but read many times, as in caches that only grow,
// or (2) when multiple goroutines read, write, and overwrite entries for disjoint
// sets of keys. In these two cases, use of a Map may significantly reduce lock
// contention compared to a Go map paired with a separate Mutex or RWMutex.
//
// The zero Map is empty and ready for use. A Map must not be copied after first use.
type Map struct {
	mu Mutex

	// read contains the portion of the map's contents that are safe for
	// concurrent access (with or without mu held).
	//
	// The read field itself is always safe to load, but must only be stored with
	// mu held.
	//
	// Entries stored in read may be updated concurrently without mu, but updating
	// a previously-expunged entry requires that the entry be copied to the dirty
	// map and unexpunged with mu held.
	read atomic.Value // readOnly

	// dirty contains the portion of the map's contents that require mu to be
	// held. To ensure that the dirty map can be promoted to the read map quickly,
	// it also includes all of the non-expunged entries in the read map.
	//
	// Expunged entries are not stored in the dirty map. An expunged entry in the
	// clean map must be unexpunged and added to the dirty map before a new value
	// can be stored to it.
	//
	// If the dirty map is nil, the next write to the map will initialize it by
	// making a shallow copy of the clean map, omitting stale entries.
	dirty map[interface{}]*entry

	// misses counts the number of loads since the read map was last updated that
	// needed to lock mu to determine whether the key was present.
	//
	// Once enough misses have occurred to cover the cost of copying the dirty
	// map, the dirty map will be promoted to the read map (in the unamended
	// state) and the next store to the map will make a new dirty copy.
	misses int
}
```

从sync.Map的注释中可以看到，sync.Map是为了2种情况做了优化：

1. 一次写入多次读取
2. 并发读取，写入和覆盖不同的key

在2种情况下比使用内置map+Mutex或RWMutex更好

sync.Map定义中可以看到一下4个字段

- mu Mutex：锁
- read atomic.Value：指向readOnly结构
- dirty map[interface{}]*entry
- misses int

readOnly结构定义

```go
type readOnly struct {
	m       map[interface{}]*entry
	amended bool // true if the dirty map contains some key not in m.
}
```

readOnly是一个map和amended的封装，从amended字段的注释知道，当amended为true的时候，dirty这个map存在readOnly这个map里面没有的key

## 删除流程

删除的流程比较简单，直接看代码注释吧

```go
// Delete deletes the value for a key.
func (m *Map) Delete(key interface{}) {
	m.LoadAndDelete(key)
}
```

```go
// LoadAndDelete deletes the value for a key, returning the previous value if any.
// The loaded result reports whether the key was present.
func (m *Map) LoadAndDelete(key interface{}) (value interface{}, loaded bool) {
	read, _ := m.read.Load().(readOnly)
	e, ok := read.m[key]
	if !ok && read.amended { // 如果readOnly中不存在并且dirty中存在readOnly中没有的key
		m.mu.Lock()
		read, _ = m.read.Load().(readOnly) // 再次获取readOnly，前面获取了锁，不会在对readOnly有修改
		e, ok = read.m[key]
		if !ok && read.amended { // 如果readOnly中不存在并且dirty中存在readOnly中没有的key，直接从dirty中删除
			e, ok = m.dirty[key]
			delete(m.dirty, key)
			// Regardless of whether the entry was present, record a miss: this key
			// will take the slow path until the dirty map is promoted to the read
			// map.
			m.missLocked()
		}
		m.mu.Unlock()
	}
	if ok { // 当readOnly这个map中存在这个key就直接调用entry#delete()方法
		return e.delete()
	}
	return nil, false
}
```

```go
func (m *Map) missLocked() {
	m.misses++
	if m.misses < len(m.dirty) {
		return
	}
	m.read.Store(readOnly{m: m.dirty}) // 如果misses的次数>dirty的len，将dirty赋值给readOnly
	m.dirty = nil
	m.misses = 0
}
```

```go
func (e *entry) delete() (value interface{}, ok bool) {
	for {
		p := atomic.LoadPointer(&e.p)
		if p == nil || p == expunged { // 如果之前的值为nil或expunged(被标记为删除)
			return nil, false
		}
		if atomic.CompareAndSwapPointer(&e.p, p, nil) { // 如果p是正常值，将p置为nil
			return *(*interface{})(p), true
		}
	}
}
```

简单说就是如果readOnly里面有key，就将value的p置为nil，如果readOnly没有这个key，从dirty里面删除key

## Store流程

知道了sync.Map的结构后，接着来看

```go
// Store sets the value for a key.
func (m *Map) Store(key, value interface{}) {
	read, _ := m.read.Load().(readOnly)
	if e, ok := read.m[key]; ok && e.tryStore(&value) {
		return
	}

	m.mu.Lock()
	read, _ = m.read.Load().(readOnly)
	if e, ok := read.m[key]; ok {
		if e.unexpungeLocked() {
			// The entry was previously expunged, which implies that there is a
			// non-nil dirty map and this entry is not in it.
			m.dirty[key] = e
		}
		e.storeLocked(&value)
	} else if e, ok := m.dirty[key]; ok {
		e.storeLocked(&value)
	} else {
		if !read.amended {
			// We're adding the first new key to the dirty map.
			// Make sure it is allocated and mark the read-only map as incomplete.
			m.dirtyLocked()
			m.read.Store(readOnly{m: read.m, amended: true})
		}
		m.dirty[key] = newEntry(value)
	}
	m.mu.Unlock()
}
```

首先从readOnly这个map中根据key获取value，如果获取到了，调用entry#tryStore方法将value保存

```go
read, _ := m.read.Load().(readOnly)
if e, ok := read.m[key]; ok && e.tryStore(&value) {
	return
}
```

```go
// expunged is an arbitrary pointer that marks entries which have been deleted
// from the dirty map.
var expunged = unsafe.Pointer(new(interface{}))

func (e *entry) tryStore(i *interface{}) bool {
	for {
		p := atomic.LoadPointer(&e.p)
		if p == expunged {
			return false
		}
		if atomic.CompareAndSwapPointer(&e.p, p, unsafe.Pointer(i)) {
			return true
		}
	}
}
```

tryStore这个方法就可以看出，是存在Store失败的情况，什么情况下会Store失败，可以看到p == expunged时返回false。

expunged是1个固定的变量，我们可以把它当作是一个删除标志，当p == expunged时，说明这个value已经被删除了。

如果tryStore成功，也就是复用了占着的这个坑位，直接就结束Store的流程，如果失败了，就进入下面的流程。

```go
m.mu.Lock()
read, _ = m.read.Load().(readOnly)
if e, ok := read.m[key]; ok { // readOnly中存在key
	if e.unexpungeLocked() { //
		// The entry was previously expunged, which implies that there is a
		// non-nil dirty map and this entry is not in it.
		m.dirty[key] = e // 要理解上面这个注释需要看懂dirtyLocked()方法
	}
	e.storeLocked(&value)
} else if e, ok := m.dirty[key]; ok { // dirty中存在key
	// TODO 这里中dirty中存在readOnly没有的key，为什么没有把amended置为true呢
	e.storeLocked(&value)
} else { // readOnly和dirty中都不存在的key
	if !read.amended { // readOnly和dirty一致
		// We're adding the first new key to the dirty map.
		// Make sure it is allocated and mark the read-only map as incomplete.
		m.dirtyLocked()
		m.read.Store(readOnly{m: read.m, amended: true}) // amended置为true，因为要想dirty中添加新的元素
	}
	// 新key直接添加到dirty
	m.dirty[key] = newEntry(value)
}
m.mu.Unlock()
```

首先获取锁，防止并发操作，再次获取readOnly，防止再获取锁之前对readOnly有修改，从readOnly中读取key，会进入3个分支

- readOnly中存在key
- readOnly中不存在key，但是dirty中存在key
- readOnly和dirty中都不存在key

第1种情况(readOnly中存在key)，由于最开始会走tryStore()方法，如果走到这个分支，说明readOnly中key对应为value有2种情况

1. 已经被标记为expunged
2. 由于并发修改CAS失败

如果时第1种情况(已经被标记为expunged)，unexpungeLocked()方法会将expunged置为nil，m.dirty[key] = e的注释中可以知道，如果unexpungeLocked()方法返回true，dirty时没有这个key的，所以需要添加到dity中，了解为什么需要看懂dirtyLocked()方法

第2种情况，也就是tryStore()方法中CAS修改value失败，此时已经获取到锁，直接修改value值

第2种情况(readOnly中不存在key，但是dirty中存在key)，直接修改dirty中value的值，这里其实dirty中存在readOnly没有的key，为什么没有把amended置为true呢？不造啊！

第3种情况(readOnly和dirty中都不存在key)，也就是向sync.Map种添加新的key

```go
if !read.amended {
		// We're adding the first new key to the dirty map.
		// Make sure it is allocated and mark the read-only map as incomplete.
		m.dirtyLocked()
		m.read.Store(readOnly{m: read.m, amended: true})
	}
m.dirty[key] = newEntry(value)
```

如果dirty和readOnly时一致的，也就是dirty中不存在readOnly中不存在的key，但是dirty中可能存在有些key被删除了(也就是entry.p是nil，而不是expunged)，所以此时就会调用dirtyLocked，将entry.p是nil的标记成expunged

```go
// 进入这个方法会有如下场景
// 1. 初始化时dirty为nil，第一次Store的时候会在这个方法中完成对dirty的初始化
// 2. dirty中有key被删除(nil)，而readOnly中没有被标记(expunged)或删除(nil)
func (m *Map) dirtyLocked() {
	if m.dirty != nil {
		return
	}

	// 遍历readOnly，将p为nil的设置成expunged
	read, _ := m.read.Load().(readOnly)
	m.dirty = make(map[interface{}]*entry, len(read.m))
	for k, e := range read.m {
		if !e.tryExpungeLocked() {
			m.dirty[k] = e // TODO 不理解为什么
		}
	}
}

func (e *entry) tryExpungeLocked() (isExpunged bool) {
	p := atomic.LoadPointer(&e.p)
	for p == nil {
		if atomic.CompareAndSwapPointer(&e.p, nil, expunged) {
			return true
		}
		p = atomic.LoadPointer(&e.p)
	}
	return p == expunged
}
```

其实dirtyLocked()方法还有1个功能，就是完成ditry的初始化，因为dirty默认值时nil，当第一次Store的时候会走到这个方法初始化dirty

## Load流程

流程也比较简单，从readOnly中读取没有删除的key，如果readOnly中没有，dirty中有新数据的情况下从dirty中读取

```go
// Load returns the value stored in the map for a key, or nil if no
// value is present.
// The ok result indicates whether value was found in the map.
func (m *Map) Load(key interface{}) (value interface{}, ok bool) {
	read, _ := m.read.Load().(readOnly)
	e, ok := read.m[key]
	if !ok && read.amended { // readOnly中没有，dirty中可能有
		m.mu.Lock()
		// Avoid reporting a spurious miss if m.dirty got promoted while we were
		// blocked on m.mu. (If further loads of the same key will not miss, it's
		// not worth copying the dirty map for this key.)
		read, _ = m.read.Load().(readOnly)
		e, ok = read.m[key]
		if !ok && read.amended {
			e, ok = m.dirty[key] // 从dirty中读取
			// Regardless of whether the entry was present, record a miss: this key
			// will take the slow path until the dirty map is promoted to the read
			// map.
			m.missLocked()
		}
		m.mu.Unlock()
	}
	if !ok { // 如果dirty和readOnly一致，但是不存在key，说明没有这个key
		return nil, false
	}
	// 直接从readOnly中读取
	return e.load()
}

func (e *entry) load() (value interface{}, ok bool) {
	p := atomic.LoadPointer(&e.p)
	if p == nil || p == expunged {
		return nil, false
	}
	return *(*interface{})(p), true
}
```

## sync.Map总结与思考

前面看了sync.Map的Store，Delete和Load流程，个人对sync.Map理解如下

sync.Map内部有2个map（ditry和readOnly），readOnly相当于时一层缓存，从readOnly读取的时候是没有加锁的

Store()方法存在多次读取的操作，

在并发的情况下，写入多的情况下，大概dirty和readOnly差距比较大的时候，会将dirty赋值给readOnly，

当dirty中有删除数据并添加新的key的时候，会为dirty重新分配空间，并将readOnly中没有删除的拷贝到新的dirty中

这就解释了官方为啥推荐在一次写入多次读取的情况下推荐用sync.Map，而其他情况推荐使用内置map+Mutex，因为sync.Map是对读取做了优化的，但是写入操作太复杂了

# skipset

skipset其实就是实现了跳表结构，可以将它视为并发安全的redis的zset。

首先我用一张图先画出跳表大概长啥样子，然后再来分析skipset的代码。

![](/syncMap&skipset探索/skiplist.png)

首先看结构定义

```go
type IntSet struct {
	header       *intNode
	length       int64
	highestLevel int64
}

type intNode struct {
	value int
	next  optionalArray
	mu    sync.Mutex
	flags bitflag
	level uint32
}

// 为什么需要分成base和extra
type optionalArray struct {
	base  [op1]unsafe.Pointer
	extra [op2]unsafe.Pointer
}
```

其中 intNode 表示的就是跳表中的1个节点，其包含value和指向下一个节点的next指针，next是一个数组，包含了许多指向下一个节点的指针

## 添加流程

```go
func (s *IntSet) Add(value int) bool {
	level := s.randomLevel()
	var preds, succs [maxLevel]*intNode
	for {
		lFound := s.findNodeAdd(value, &preds, &succs)
		if lFound != -1 { // 说明value已经在set中了
			nodeFound := succs[lFound]
			if !nodeFound.flags.Get(marked) { // 没有被删除
				for !nodeFound.flags.Get(fullyLinked) { // 说明其他线程正在添加该value
					// 等待变成 fullyLinked
				}
				return false
			}
			// 如果node被标记成marked，说明其他的线程正在处理删除这个node
			continue
		}

		var (
			highestLocked        = -1
			valid                = true
			pred, succ, prevPred *intNode
		)

		for layer := 0; valid && layer < level; layer++ {
			pred = preds[layer]
			succ = succs[layer]

			if pred != prevPred { //  防止重复Lock
				pred.mu.Lock()
				highestLocked = layer
				prevPred = pred
			}

			valid = !pred.flags.Get(marked) && (succ == nil || !succ.flags.Get(marked)) && pred.loadNext(layer) == succ
		}

		if !valid {
			unlockInt(preds, highestLocked)
			continue
		}

		nn := newIntNode(value, level)
		for layer := 0; layer < level; layer++ {
			nn.storeNext(layer, succs[layer])
			preds[layer].atomicStoreNext(layer, nn)
		}

		nn.flags.SetTrue(fullyLinked)
		unlockInt(preds, highestLocked)
		atomic.AddInt64(&s.length, 1)
		return true
	}
}
```

首先随机生成该节点所在的层数level，level是基于概率生成的，这个和Redis中的跳表level生成是类似的，level越低，概率越高，level越高概率越低 

```go
func (s *IntSet) randomLevel() int {
	level := randomLevel()
	for {
		hl := atomic.LoadInt64(&s.highestLevel)
		if int64(level) < hl {
			break
		}
		if atomic.CompareAndSwapInt64(&s.highestLevel, hl, int64(level)) {
			break
		}
	}
	return level
}
```

```go
const (
	p                   = 0.25
	maxLevel            = 16
	defaultHighestLevel = 3
)

func randomLevel() int {
	level := 1
	for fastrand.Intn(1/p) == 0 {
		level++
	}
	if level > maxLevel {
		return maxLevel
	}
	return level
}
```

然后从skipset中查找该节点插入的位置

```go
// preds 和 succs 满足 preds[i] < value <= succs[i]
// 返回-1表示value在set中不存在
// 返回>=0表示value存在于set中
func (s *IntSet) findNodeAdd(value int, preds, succs *[maxLevel]*intNode) int {
	x := s.header
	for i := int(atomic.LoadInt64(&s.highestLevel) - 1); i >= 0; i-- {
		succ := x.atomicLoadNext(i)
		for succ != nil && succ.lessThan(value) {
			x = succ
			succ = x.atomicLoadNext(i)
		}

		preds[i] = x
		succs[i] = succ // 当value大于所有 or set为空，succ为nil

		// 判断是否等于
		if succ != nil && succ.equal(value) {
			return i
		}
		// 如果succ>value，说明当前层已经找完了，level就下钻，继续下一层
	}
	return -1
}
```

从最高层开始找起，从每层中找到前驱节点小于value，后继节点大于value的node，并将每层的结果保存到preds，succs2个map中。如果在某层找到于value相等的节点，返回该节点的level

```go
if lFound != -1 { // 说明value已经在set中了
	nodeFound := succs[lFound]
	if !nodeFound.flags.Get(marked) { // 没有被删除
		for !nodeFound.flags.Get(fullyLinked) {
			// 等待变成 fullyLinked
		}
		return false
	}
	// 如果node被标记成marked，说明其他的线程正在处理删除这个node
	continue
}
```

如果返回的level ≠ -1，说明在skipset中找到了和value相等的节点，然后判断这个节点有没有被标记成删除，

- 如果被标记删除了，等待其他线程处理完成，重新开始之前的步骤
- 如果没被标记删除，但是只想后继节点的链接还完成，说明其他线程正在添加该value，空转直到完成

如果返回的level == -1，说明在skipset中没有找到该value，需要向skipset中添加该节点。在向skipset中添加之前，为了防止并发的问题，从最底层到该节点所在的level层，每层都将前驱结点lock住，其中检查节点的状态是否正常，如果不正常了（其他线程对skipset作了修改），需要将释放前驱节点的锁，然后重新再来

```go
var (
	highestLocked        = -1
	valid                = true
	pred, succ, prevPred *intNode
)

for layer := 0; valid && layer < level; layer++ {
	pred = preds[layer]
	succ = succs[layer]

	if pred != prevPred { //  防止重复Lock
		pred.mu.Lock()
		highestLocked = layer
		prevPred = pred
	}

	valid = !pred.flags.Get(marked) && (succ == nil || !succ.flags.Get(marked)) && pred.loadNext(layer) == succ
}

if !valid {
	unlockInt(preds, highestLocked)
	continue
}
```

当前驱节点都已经 加上锁之后，现在就可以放心的 向skipset中添加节点了

```go
nn := newIntNode(value, level)
for layer := 0; layer < level; layer++ {
	nn.storeNext(layer, succs[layer])
	preds[layer].atomicStoreNext(layer, nn)
}

nn.flags.SetTrue(fullyLinked)
unlockInt(preds, highestLocked)
atomic.AddInt64(&s.length, 1)
return true
```

首先初始化 intNode节点，然后把next指向后驱节点，前驱节点的 next指向该node，将 node 的状态设置成fullyLinked，最后释放前驱节点的锁，length进行+1。

## 删除流程

```go
func (s *IntSet) Remove(value int) bool {
	var (
		nodeToRemove *intNode
		isMarked     bool
		topLayer     = -1
		preds, succs [maxLevel]*intNode
	)
	for {
		lFound := s.findNodeRemove(value, &preds, &succs)
		if isMarked || lFound != -1 &&
			succs[lFound].flags.MGet(fullyLinked|marked, fullyLinked) &&
			int(succs[lFound].level)-1 == lFound { // level从1开始，layer从0开始
			if !isMarked {
				nodeToRemove = succs[lFound]
				topLayer = lFound
				nodeToRemove.mu.Lock()
				// 这个node正在被其他线程删除
				if nodeToRemove.flags.Get(marked) {
					nodeToRemove.mu.Unlock()
					return false
				}
				nodeToRemove.flags.SetTrue(marked)
				isMarked = true
			}

			// 进行物理删除
			var (
				valid                = true
				highestLocked        = -1
				pred, succ, prevPred *intNode
			)
			for layer := 0; valid && layer <= topLayer; layer++ {
				pred, succ = preds[layer], succs[layer]
				if pred != prevPred {
					pred.mu.Lock()
					highestLocked = layer
					prevPred = pred
				}

				valid = !pred.flags.Get(marked) && pred.loadNext(layer) == succ
			}

			if !valid {
				unlockInt(preds, highestLocked)
				continue
			}

			for i := topLayer; i >= 0; i-- {
				preds[i].atomicStoreNext(i, nodeToRemove.loadNext(i))
			}
			nodeToRemove.mu.Unlock()
			atomic.AddInt64(&s.length, -1)
			return true
		}
		return false
	}
}
```

首先也是找到节点

```go
func (s *IntSet) findNodeRemove(value int, preds, succs *[maxLevel]*intNode) int {
	lFound, x := -1, s.header
	for i := int(atomic.LoadInt64(&s.highestLevel)) - 1; i >= 0; i-- {
		succ := x.atomicLoadNext(i)
		for succ != nil && succ.lessThan(value) {
			x = succ
			succ = x.atomicLoadNext(i)
		}

		preds[i] = x
		succs[i] = succ

		if lFound == -1 && succ != nil && succ.equal(value) {
			lFound = i
		}
	}
	return lFound
}
```

如果没找到直接直接返回false，如果找到了(lFound ≠ -1)，就将node标记成marked

```go
if !isMarked {
	nodeToRemove = succs[lFound]
	topLayer = lFound
	nodeToRemove.mu.Lock()
	// 这个node正在被其他线程删除
	if nodeToRemove.flags.Get(marked) {
		nodeToRemove.mu.Unlock()
		return false
	}
	nodeToRemove.flags.SetTrue(marked)
	isMarked = true
}
```

然后也是将前驱节点加锁，如果状态验证失败，释放前驱节点的 锁，然后重试

```go
	// 进行物理删除
	var (
		valid                = true
		highestLocked        = -1
		pred, succ, prevPred *intNode
	)
	for layer := 0; valid && layer <= topLayer; layer++ {
		pred, succ = preds[layer], succs[layer]
		if pred != prevPred {
			pred.mu.Lock()
			highestLocked = layer
			prevPred = pred
		}

		valid = !pred.flags.Get(marked) && pred.loadNext(layer) == succ
	}

	if !valid {
		unlockInt(preds, highestLocked)
		continue
	}
```

最后每一层将前驱节点的next直接指向删除的节点的后驱，length进行-1

```go
for i := topLayer; i >= 0; i-- {
	preds[i].atomicStoreNext(i, nodeToRemove.loadNext(i))
}
nodeToRemove.mu.Unlock()
atomic.AddInt64(&s.length, -1)
return true
```

## 查找流程

```go
func (s *IntSet) Contains(value int) bool {
	x := s.header
	for i := int(atomic.LoadInt64(&s.highestLevel) - 1); i >= 0; i-- {
		next := x.atomicLoadNext(i)
		for next != nil && next.lessThan(value) {
			x = next
			next = x.atomicLoadNext(i)
		}
		// 判断是否等于
		if next != nil && next.equal(value) {
			// marked表示node已经被删除了，只有当flags == fullyLinked，才能表示node存在
			return next.flags.MGet(fullyLinked|marked, fullyLinked)
		}
		// 如果大于，level就下钻
	}
	return false
}
```

其实查找在添加和删除流程中也有体现，这里就不详细展开了

## skipset总结

skipset其实就是跳表的实现，在并发控制方面，在读取的时候使用CAS，当需要对节点做操作的时候，会进行加锁

# 总结

sync.Map由于readOnly充当了缓存层，在从readOnly读取数据的时候无需加锁，但是在写操作的时候会有2次读取的逻辑，并且加锁，在一定情况下还会想readOnly拷贝数据，所以在一次写入多次读取的情况下sync.Map的性能很好，在多写的时候，性能就下降得很厉害

skipset同样是读取的时候使用CAS的思想实现了无锁读取，但是在写操作的时候，只是在插入node的前驱节点上加锁，加锁粒度相对较小，在多写的情况下仍然能保持相对较高的性能

所以大家根据自己业务情况，酌情使用。除了 skipset，该仓库作者使用跳表的方式实现了skipmap，思路都差不多，也可以去了解一下。

# 参考资料

[一口气搞懂 Go sync.map 所有知识点](https://mp.weixin.qq.com/s/8aufz1IzElaYR43ccuwMyA)

[GitHub - zhangyunhao116/skipset: skipset is a high-performance, scalable concurrent sorted set based on skip-list. Up to 15x faster than sync.Map in the typical pattern.](https://github.com/zhangyunhao116/skipset)