---
title: "【译】Go并发模式"
date: 2021-07-27T22:19:26+08:00
draft: false
original: true
categories: 
  - Golang
tags: 
  - Golang基础
---

原文：[https://github.com/lotusirous/go-concurrency-patterns](https://github.com/lotusirous/go-concurrency-patterns)

# 资料

- [Concurrency is not parallelism](https://blog.golang.org/waza-talk)
- [Go Concurrency Patterns](https://talks.golang.org/2012/concurrency.slide#1) and [source](https://talks.golang.org/2012/concurrency/support/)
- [Advanced Go Concurrency Patterns](https://talks.golang.org/2013/advconc.slide)
- [Rethinking classical concurrency pattern](https://www.youtube.com/watch?v=5zXAHh5tJqQ)
- [Go Concurrency Patterns: Pipelines and cancellation](https://blog.golang.org/pipelines)
- [Go Concurrency Patterns: Timing out, moving on](https://blog.golang.org/concurrency-timeouts)
- [Complex Concurrency Patterns with Go](https://www.youtube.com/watch?v=2HOO5gIgyMg)

# Context

- [Go Concurrency Patterns: Context](https://blog.golang.org/context)
- [How to correctly use package context](https://www.youtube.com/watch?v=-_B5uQ4UGi0)
- [justforfunc #9: The Context Package](https://www.youtube.com/watch?v=LSzR0VEraWw)
- [Contexts and structs](https://blog.golang.org/context-and-structs)

<!--more-->
# 正文

## 1. boring(goroutine入门)

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func boring(msg string) {
	for i := 0; ; i++ {
		fmt.Println(msg, i)
		time.Sleep(time.Duration(rand.Intn(1e3)) * time.Millisecond)
	}
}

func main() {
  // 这行运行之后，main协程就完成了
	// main协程作为调用者，没有等待boring函数执行完成
  // 因此，我们看不到任何输出
	// after run this line, the main goroutine is finished.
	// main goroutine is a caller. It doesn't wait for func boring finished
	// Thus, we don't see anything
	go boring("boring!") // spawn a goroutine. (1)

	// To solve it, we can make the main go routine run forever by `for {}` statement.
  // 为了解决这个问题，我们可以在main协程中添加for{}语句让其一直运行

	// for {
	// }
	
	// 更有趣的是main协程退出了，整个程序也就退出了
	// A little more interesting is the main goroutine exit. the program also exited
	// 此代码hang住
	// This code hang
	fmt.Println("I'm listening")
	time.Sleep(2 * time.Second)
	fmt.Println("You're boring. I'm leaving")
	
	// 然而，main协程和boring协程没有交流
	// 因此，上面的代码有点作弊，因为boring协程是自己打印到stdout的
  // 命令行的`boring! 1`是boring协程打印的
	// However, the main goroutine and boring goroutine does not communicate each other.
	// Thus, the above code is cheated because the boring goroutine prints to stdout by its own function.
	// the line `boring! 1` that we see on terminal is the output from boring goroutine.
	
	// 真正的对话是需要交流的
	// real conversation requires a communication
}
```

## 2-chan(channel入门)

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

// 现在，boring函数有一个额外的参数
// `c chan string`是一个channel
// now, the boring function additional parametter
// `c chan string` is a channel
func boring(msg string, c chan string) {
	for i := 0; ; i++ {
		// 发送数据到channel
		// 也会等待receiver ready
		// send the value to channel
		// it also waits for receiver to be ready
		c <- fmt.Sprintf("%s %d", msg, i)
		time.Sleep(time.Duration(rand.Intn(1e3)) * time.Millisecond)
	}
}

func main() {
	// 初始化我们的channel
	// init our channel
	c := make(chan string)
	// 传给我们的协程
	// placeholdering our goroutine
	go boring("boring!", c)

	for i := 0; i < 5; i++ {
    // <-c 从boring函数中读取数据
		// <-c 等待发送数据到c
		// <-c read the value from boring function
		// <-c waits for a value to be sent
		fmt.Printf("You say: %q\n", <-c)
	}
	fmt.Println("You're boring. I'm leaving")

}
```

## 3-generator(类python的生成器)

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

// boring是一个返回与之通信的channel的函数
// <-chan string 表示只能从channl中读取string类型的数据
// boring is a function that returns a channel to communicate with it.
// <-chan string means receives-only channel of string.
func boring(msg string) <-chan string {
	c := make(chan string)
	// 我们在函数内部启动一个协程
  // 这个写成会发送数据到channel中
	// we launch goroutine inside a function
	// that sends the data to channel
	go func() {
		// 这个for循环模拟无限的sender
		// The for loop simulate the infinite sender.
		for i := 0; i < 10; i++ {
			c <- fmt.Sprintf("%s %d", msg, i)
			time.Sleep(time.Duration(rand.Intn(1e3)) * time.Millisecond)
		}
		// sender负责关闭channel
		// The sender should close the channel
		close(c)

	}()
	// 返回给调用者
	return c // return a channel to caller.
}

func main() {

	joe := boring("Joe")
	ahn := boring("Ahn")
	
	// 这个循环会依次从2个channel中获取数据
	// This loop yields 2 channels in sequence
	for i := 0; i < 10; i++ {
		fmt.Println(<-joe)
		fmt.Println(<-ahn)
	}
	// 或者我们可以使用for range来获取
	// or we can simply use the for range
	// for msg := range joe {
	// 	fmt.Println(msg)
	// }
	fmt.Println("You're both boring. I'm leaving")

}
```

## 4-fanin(fan in模式)

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

// the boring function return a channel to communicate with it.
func boring(msg string) <-chan string { // <-chan string means receives-only channel of string.
	c := make(chan string)
	go func() { // we launch goroutine inside a function.
		for i := 0; ; i++ {
			c <- fmt.Sprintf("%s %d", msg, i)
			time.Sleep(time.Duration(rand.Intn(1e3)) * time.Millisecond)
		}

	}()
	return c // return a channel to caller.
}

// <-chan string only get the receive value
// fanIn spawns 2 goroutines to reads the value from 2 channels
// then it sends to value to result channel( `c` channel)
func fanIn(c1, c2 <-chan string) <-chan string {
	c := make(chan string)
	go func() {
		for { // infinite loop to read value from channel.
			v1 := <-c1 // read value from c2. This line will wait when receiving value.
			c <- v1
		}
	}()
	go func() {
		for {
			c <- <-c2 // read value from c2 and send it to c
		}
	}()
	return c
}

func fanInSimple(cs ...<-chan string) <-chan string {
	c := make(chan string)
	for _, ci := range cs { // spawn channel based on the number of input channel

		go func(cv <-chan string) { // cv is a channel value
			for {
				c <- <-cv
			}
		}(ci) // send each channel to

	}
	return c
}

func main() {
  // 合并2个channel到1个channel
	// merge 2 channels into 1 channel
	// c := fanIn(boring("Joe"), boring("Ahn"))
	c := fanInSimple(boring("Joe"), boring("Ahn"))

	for i := 0; i < 5; i++ {
		fmt.Println(<-c) // now we can read from 1 channel
	}
	fmt.Println("You're both boring. I'm leaving")
}
```

## 5-restore-sequence(Restore sequence)

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

type Message struct {
	str  string
	wait chan bool
}

func fanIn(inputs ...<-chan Message) <-chan Message {
	c := make(chan Message)
	for i := range inputs {
		input := inputs[i]
		go func() {
			for {
				c <- <-input
			}
		}()
	}
	return c
}

// the boring function return a channel to communicate with it.
func boring(msg string) <-chan Message { // <-chan string means receives-only channel of string.
	c := make(chan Message)
	waitForIt := make(chan bool) // share between all messages
	go func() {                  // we launch goroutine inside a function.
		for i := 0; ; i++ {
			c <- Message{
				str:  fmt.Sprintf("%s %d", msg, i),
				wait: waitForIt,
			}
			time.Sleep(time.Duration(rand.Intn(1e3)) * time.Millisecond)

			// every time the goroutine send message.
			// This code waits until the value to be received.
			<-waitForIt
		}

	}()
	return c // return a channel to caller.
}

func main() {
	// merge 2 channels into 1 channel
	c := fanIn(boring("Joe"), boring("Ahn"))

	for i := 0; i < 5; i++ {
		msg1 := <-c // wait to receive message
		fmt.Println(msg1.str)
		msg2 := <-c
		fmt.Println(msg2.str)

		// each go routine have to wait
		msg1.wait <- true // main goroutine allows the boring goroutine to send next value to message channel.
		msg2.wait <- true
	}
	fmt.Println("You're both boring. I'm leaving")
}

// main: goroutine                                          boring: goroutine
//    |                                                           |
//    |                                                           |
// wait for receiving msg from channel c                    c <- Message{} // send message
//   <-c                                                          |
//    |                                                           |
//    |                                                     <-waitForIt // wait for wake up signal
// send value to channel                                          |
// hey, boring. You can send next value to me                     |
//   wait <-true                                                  |
///                            REPEAT THE PROCESS
```

## 6-select-timeout(给goroutine添加超时)

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

// the boring function return a channel to communicate with it.
func boring(msg string) <-chan string { // <-chan string means receives-only channel of string.
	c := make(chan string)
	go func() { // we launch goroutine inside a function.
		for i := 0; ; i++ {
			c <- fmt.Sprintf("%s %d", msg, i)
			time.Sleep(time.Duration(rand.Intn(1500)) * time.Millisecond)
		}

	}()
	return c // return a channel to caller.
}

func main() {
	c := boring("Joe")

	// timeout for the whole conversation
	timeout := time.After(5 * time.Second)
	for {
		select {
		case s := <-c:
			fmt.Println(s)
		case <-timeout:
			fmt.Println("You talk too much.")
			return
		}
	}
}
```

## 7-quit-signal(Quit信号)

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

// the boring function return a channel to communicate with it.
func boring(msg string, quit chan string) <-chan string { // <-chan string means receives-only channel of string.
	c := make(chan string)
	go func() { // we launch goroutine inside a function.
		for i := 0; ; i++ {
			select {
			case c <- fmt.Sprintf("%s %d", msg, i):
				// do nothing
			case <-quit:
				fmt.Println("clean up")
				quit <- "See you!"
				return
			}
			time.Sleep(time.Duration(rand.Intn(1e3)) * time.Millisecond)
		}

	}()
	return c // return a channel to caller
}

func main() {
	quit := make(chan string)
	c := boring("Joe", quit)
	for i := 3; i >= 0; i-- {
		fmt.Println(<-c)
	}
	quit <- "Bye"
	fmt.Println("Joe say:", <-quit)
}
```

## 8-daisy-chan(Daisy chan模式)

```go
package main

import (
	"fmt"
)

func f(left, right chan int) {
	left <- 1 + <-right // get the value from the right and add 1 to it
}

func main() {
	const n = 1000
	leftmost := make(chan int)
	left := leftmost
	right := leftmost

	for i := 0; i < n; i++ {
		right = make(chan int)
		go f(left, right)
		left = right
	}

	go func(c chan int) { c <- 1 }(right)
	fmt.Println(<-leftmost)

}
```

## 9-google1.0()

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

type Result string
type Search func(query string) Result

var (
	Web   = fakeSearch("web")
	Image = fakeSearch("image")
	Video = fakeSearch("video")
)

func fakeSearch(kind string) Search {
	return func(query string) Result {
		time.Sleep(time.Duration(rand.Intn(100)) * time.Millisecond)
		return Result(fmt.Sprintf("%s result for %q\n", kind, query))
	}
}

// it invokes serially Web, Image and Video appending them to the Results
func Google(query string) (results []Result) {
	results = append(results, Web(query))
	results = append(results, Image(query))
	results = append(results, Video(query))
	return
}

func main() {
	rand.Seed(time.Now().UnixNano())
	start := time.Now()
	results := Google("golang")
	elapsed := time.Since(start)
	fmt.Println(results)
	fmt.Println(elapsed)
}
```

## 10-google2.0()

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

type Result string
type Search func(query string) Result

var (
	Web   = fakeSearch("web")
	Image = fakeSearch("image")
	Video = fakeSearch("video")
)

func fakeSearch(kind string) Search {
	return func(query string) Result {
		time.Sleep(time.Duration(rand.Intn(100)) * time.Millisecond)
		return Result(fmt.Sprintf("%s result for %q\n", kind, query))
	}
}

func Google(query string) []Result {
	c := make(chan Result)

	// each search performs in a goroutine
	go func() {
		c <- Web(query)
	}()
	go func() {
		c <- Image(query)
	}()
	go func() {
		c <- Video(query)
	}()

	var results []Result
	for i := 0; i < 3; i++ {
		results = append(results, <-c)
	}

	return results
}

func main() {
	rand.Seed(time.Now().UnixNano())
	start := time.Now()
	results := Google("golang")
	elapsed := time.Since(start)
	fmt.Println(results)
	fmt.Println(elapsed)
}
```

## 11-google2.1()

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

type Result string
type Search func(query string) Result

var (
	Web   = fakeSearch("web")
	Image = fakeSearch("image")
	Video = fakeSearch("video")
)

func fakeSearch(kind string) Search {
	return func(query string) Result {
		time.Sleep(time.Duration(rand.Intn(100)) * time.Millisecond)
		return Result(fmt.Sprintf("%s result for %q\n", kind, query))
	}
}

// I don't want to wait for slow server
func Google(query string) []Result {
	c := make(chan Result)

	// each search performs in a goroutine
	go func() {
		c <- Web(query)
	}()
	go func() {
		c <- Image(query)
	}()
	go func() {
		c <- Video(query)
	}()

	var results []Result

	// the global timeout for 3 queries
	// it means after 50ms, it ignores the result from the server that taking response greater than 50ms
	//
	timeout := time.After(50 * time.Millisecond)
	for i := 0; i < 3; i++ {
		select {
		case r := <-c:
			results = append(results, r)
		// this line ignore the slow server.
		case <-timeout:
			fmt.Println("timeout")
			return results
		}
	}

	return results
}

func main() {
	rand.Seed(time.Now().UnixNano())
	start := time.Now()
	results := Google("golang")
	elapsed := time.Since(start)
	fmt.Println(results)
	fmt.Println(elapsed)
}
```

## 12-google3.0()

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

type Result string
type Search func(query string) Result

var (
	Web1   = fakeSearch("web1")
	Web2   = fakeSearch("web2")
	Image1 = fakeSearch("image1")
	Image2 = fakeSearch("image2")
	Video1 = fakeSearch("video1")
	Video2 = fakeSearch("video2")
)

func fakeSearch(kind string) Search {
	return func(query string) Result {
		time.Sleep(time.Duration(rand.Intn(100)) * time.Millisecond)
		return Result(fmt.Sprintf("%s result for %q\n", kind, query))
	}
}

// How do we avoid discarding result from the slow server.
// We duplicates to many instance, and perform parallel request.
func First(query string, replicas ...Search) Result {
	c := make(chan Result)
	for i := range replicas {
		go func(idx int) {
			c <- replicas[idx](query)
		}(i)
	}
	// the magic is here. First function always waits for 1 time after receiving the result
	return <-c
}

// I don't want to wait for slow server
func Google(query string) []Result {
	c := make(chan Result)

	// each search performs in a goroutine
	go func() {
		c <- First(query, Web1, Web2)
	}()
	go func() {
		c <- First(query, Image1, Image2)
	}()
	go func() {
		c <- First(query, Video1, Video2)
	}()

	var results []Result

	// the global timeout for 3 queries
	// it means after 50ms, it ignores the result from the server that taking response greater than 50ms
	timeout := time.After(50 * time.Millisecond)
	for i := 0; i < 3; i++ {
		select {
		case r := <-c:
			results = append(results, r)
		// this line ignore the slow server.
		case <-timeout:
			fmt.Println("timeout")
			return results
		}
	}

	return results
}

func main() {
	rand.Seed(time.Now().UnixNano())
	start := time.Now()
	results := Google("golang")
	elapsed := time.Since(start)
	fmt.Println(results)
	fmt.Println(elapsed)

}
```

## 13-adv-pinpong()

```go
package main

import (
	"fmt"
	"time"
)

type Ball struct{ hits int }

func player(name string, table chan *Ball) {
	for {
		ball := <-table // player grabs the ball
		ball.hits++
		fmt.Println(name, ball.hits)
		time.Sleep(100 * time.Millisecond)
		table <- ball // pass the ball
	}
}

func main() {
	table := make(chan *Ball)

	go player("ping", table)
	go player("pong", table)
	table <- new(Ball) // game on; toss the ball
	time.Sleep(1 * time.Second)
	<-table // game over, grab the ball
	panic("show me the stack")

}
```

## 14-adv-subscription()

```go
package main

import "time"

type Item struct{ Tile, Channel, GUID string }

// Fetcher fetches Items and returns the time when the next fetch
// should be attempted. On failure, fetch returns non-nil error.
type Fetcher interface {
	Fetch() (item []Item, next time.Time, err error)
}

// A Subscription delivers Items over a channel.  Close cancels the
// subscription, closes the Updates channel, and returns the last fetch error,
// if any.
type Subscription interface {
	Updates() <-chan Item
	Close() error
}

func Subscribe(fetcher Fetcher) Subscription {
	s := &sub{
		fetcher: fetcher,
		updates: make(chan Item),
		closing: make(chan chan error),
	}
	return s
}

type sub struct {
	fetcher Fetcher
	updates chan Item
	closing chan chan error
}

func (s *sub) Updates() <-chan Item {
	return s.updates
}

func (s *sub) Close() error {
	// STOPCLOSESIG OMIT
	errc := make(chan error)
	s.closing <- errc // HLchan
	return <-errc     // HLchan
}

func main() {

}
```

## 15-bounded-parallelism()

```go
package main

import (
	"crypto/md5"
	"errors"
	"fmt"
	"io/ioutil"
	"os"
	"path/filepath"
	"sort"
	"sync"
)

// walkFiles starts a goroutine to walk the directory tree at root and send the
// path of each regular file on the string channel.  It sends the result of the
// walk on the error channel.  If done is closed, walkFiles abandons its work.
func walkFiles(done <-chan struct{}, root string) (<-chan string, <-chan error) {
	paths := make(chan string)
	errc := make(chan error, 1)
	go func() { // HL
		// Close the paths channel after Walk returns.
		defer close(paths) // HL
		// No select needed for this send, since errc is buffered.
		errc <- filepath.Walk(root, func(path string, info os.FileInfo, err error) error { // HL
			if err != nil {
				return err
			}
			if !info.Mode().IsRegular() {
				return nil
			}
			select {
			case paths <- path: // HL
			case <-done: // HL
				return errors.New("walk canceled")
			}
			return nil
		})
	}()
	return paths, errc
}

// A result is the product of reading and summing a file using MD5.
type result struct {
	path string
	sum  [md5.Size]byte
	err  error
}

// digester reads path names from paths and sends digests of the corresponding
// files on c until either paths or done is closed.
func digester(done <-chan struct{}, paths <-chan string, c chan<- result) {
	for path := range paths { // HLpaths
		data, err := ioutil.ReadFile(path)
		select {
		case c <- result{path, md5.Sum(data), err}:
		case <-done:
			return
		}
	}
}

// MD5All reads all the files in the file tree rooted at root and returns a map
// from file path to the MD5 sum of the file's contents.  If the directory walk
// fails or any read operation fails, MD5All returns an error.  In that case,
// MD5All does not wait for inflight read operations to complete.
func MD5All(root string) (map[string][md5.Size]byte, error) {
	// MD5All closes the done channel when it returns; it may do so before
	// receiving all the values from c and errc.
	done := make(chan struct{})
	defer close(done)

	paths, errc := walkFiles(done, root)

	// Start a fixed number of goroutines to read and digest files.
	c := make(chan result) // HLc
	var wg sync.WaitGroup
	const numDigesters = 20
	wg.Add(numDigesters)
	for i := 0; i < numDigesters; i++ {
		go func() {
			digester(done, paths, c) // HLc
			wg.Done()
		}()
	}
	go func() {
		wg.Wait()
		close(c) // HLc
	}()
	// End of pipeline. OMIT

	m := make(map[string][md5.Size]byte)
	for r := range c {
		if r.err != nil {
			return nil, r.err
		}
		m[r.path] = r.sum
	}
	// Check whether the Walk failed.
	if err := <-errc; err != nil { // HLerrc
		return nil, err
	}
	return m, nil
}

func main() {
	// Calculate the MD5 sum of all files under the specified directory,
	// then print the results sorted by path name.
	m, err := MD5All(os.Args[1])
	if err != nil {
		fmt.Println(err)
		return
	}
	var paths []string
	for path := range m {
		paths = append(paths, path)
	}
	sort.Strings(paths)
	for _, path := range paths {
		fmt.Printf("%x  %s\n", m[path], path)
	}
}
```

## 16-context()

```go
// This implemented a sample from the video
// https://www.youtube.com/watch?v=LSzR0VEraWw
package main

import (
	"context"
	"log"
	"time"
)

func sleepAndTalk(ctx context.Context, d time.Duration, msg string) {
	select {
	case <-time.After(d):
		log.Println(msg)
	case <-ctx.Done():
		log.Println(ctx.Err())
	}
}

func main() {
	log.Println("started")
	ctx := context.Background()
	ctx, cancel := context.WithCancel(ctx)

	time.AfterFunc(time.Second, cancel)
	sleepAndTalk(ctx, 5*time.Second, "hello")
}
```

## 17-ring-buffer-channel

```go
package main

import "log"

// A channel-based ring buffer removes the oldest item when the queue is full
// Ref:
// https://tanzu.vmware.com/content/blog/a-channel-based-ring-buffer-in-go

func NewRingBuffer(inCh, outCh chan int) *ringBuffer {
	return &ringBuffer{
		inCh:  inCh,
		outCh: outCh,
	}
}

// ringBuffer throttle buffer for implement async channel.
type ringBuffer struct {
	inCh  chan int
	outCh chan int
}

func (r *ringBuffer) Run() {
	for v := range r.inCh {
		select {
		case r.outCh <- v:
		default:
			<-r.outCh // pop one item from outchan
			r.outCh <- v
		}
	}
	close(r.outCh)
}

func main() {
	inCh := make(chan int)
	outCh := make(chan int, 4) // try to change outCh buffer to understand the result
	rb := NewRingBuffer(inCh, outCh)
	go rb.Run()

	for i := 0; i < 10; i++ {
		inCh <- i
	}

	close(inCh)

	for res := range outCh {
		log.Println(res)
	}

}
```

## 18-worker-pool(工作池模式)

```go
// Credit:
// https://gobyexample.com/worker-pools

// Worker pool benefits:
// - Efficiency because it distributes the work across threads.
// - Flow control: Limit work in flight

// Disadvantage of worker:
// Lifetimes complexity: clean up and idle worker

// Principles:
// Start goroutines whenever you have the concurrent work to do.
// The goroutine should exit as soon as posible the work is done. This helps us
// to clean up the resources and manage the lifetimes correctly.
package main

import (
	"fmt"
	"sync"
	"time"
)

func worker(id int, jobs <-chan int, results chan<- int) {
	for j := range jobs {
		fmt.Println("worker", id, "started job", j)
		time.Sleep(time.Second)
		fmt.Println("worker", id, "fnished job", j)
		results <- j * 2
	}
}

func workerEfficient(id int, jobs <-chan int, results chan<- int) {
	// sync.WaitGroup helps us to manage the job
	var wg sync.WaitGroup
	for j := range jobs {

		wg.Add(1)
		// we start a goroutine to run the job
		go func(job int) {
			// start the job
			fmt.Println("worker", id, "started job", job)
			time.Sleep(time.Second)
			fmt.Println("worker", id, "fnished job", job)
			results <- job * 2
			wg.Done()

		}(j)

	}
	// With a help to manage the lifetimes of goroutines
	// we can add more handler when a goroutine finished
	wg.Wait()
}
func main() {
	const numbJobs = 8
	jobs := make(chan int, numbJobs)
	results := make(chan int, numbJobs)

	// 1. Start the worker
	// it is a fixed pool of goroutines receive and perform tasks from a channel

	// In this example, we define a fixed 3 workers
	// they receive the `jobs` from the channel jobs
	// we also naming the worker name with `w` variable.
	for w := 1; w <= 3; w++ {
		go workerEfficient(w, jobs, results)
	}

	// 2. send the work
	// other goroutine sends the work to the channels

	// in this example, the `main` goroutine sends the work to the channel `jobs`
	for j := 1; j <= numbJobs; j++ {
		jobs <- j
	}
	close(jobs)
	fmt.Println("Closed job")
	for a := 1; a <= numbJobs; a++ {
		<-results
	}
	close(results)

}
```