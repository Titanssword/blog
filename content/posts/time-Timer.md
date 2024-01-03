---
title: Go 的定时器的使用
date: 2018-01-21　
tags:
- Go
- example

categories:
- Goprogram
description: "implementation about time"
---

## time.Newtimer 函数
初始化一个到期时间据此时的间隔为 3 小时 30 分的定时器

t := time.Newtimer(3*time.Hour + 30*time.Minute)

注意，这里的变量 t 是 * time.NewTimer 类型的，这个指针类型的方法集合包含两个方法

- Rest
  用于重置定时器  
  该方法返回一个 bool 类型的值

- Stop
  用来停止定时器
  该方法返回一个 bool 类型的值，如果返回 false，说明该定时器在之前已经到期或者已经被停止了, 反之返回 true。

通过定时器的字段 C, 我们可以及时得知定时器到期的这个事件来临，C 是一个 chan time.Time 类型的缓冲通道　，一旦触及到期时间，定时器就会向自己的 C 字段发送一个 time.Time 类型的元素值

```
package main

import (
    "fmt"
    "time"
)

func main(){
    // 初始化定时器
    t := time.NewTimer(2 * time.Second)
    // 当前时间
    now := time.Now()
    fmt.Printf("Now time : %v.\n", now)

    expire := <- t.C
    fmt.Printf("Expiration time: %v.\n", expire)
}


```

## example of Timeout
超时对于连接到外部资源或否则需要限制执行时间的程序很重要。在 Go 中实现超时很简单，而且非常优雅，这要感谢渠道和选择。

对于我们的例子，假设我们正在执行一个外部调用，它在 2s 后在一个通道 c1 上返回它的结果。
Here’s the select implementing a timeout. res := <-c1 awaits the result and <-Time.After awaits a value to be sent after the timeout of 1s. Since select proceeds with the first receive that’s ready, we’ll take the timeout case if the operation takes more than the allowed 1s.
If we allow a longer timeout of 3s, then the receive from c2 will succeed and we’ll print the result.
```

package main
import "time"
import "fmt"
func main() {
    c1 := make(chan string, 1)
    go func() {
        time.Sleep(time.Second * 2)
        c1 <- "result 1"
    }()

    select {
    case res := <-c1:
        fmt.Println(res)
    case <-time.After(time.Second * 1):
        fmt.Println("timeout 1")
    }
    c2 := make(chan string, 1)
    go func() {
        time.Sleep(time.Second * 2)
        c2 <- "result 2"
    }()
    select {
    case res := <-c2:
        fmt.Println(res)
    case <-time.After(time.Second * 3):
        fmt.Println("timeout 2")
    }
}


```

## 自定义计时器
下面是一个定时器的实例，两个进程，一个进程负责计时，另一个进程是用来往 channel 中写入数据，当计时器到时时，输出 timeout

```
package main

import (
    "fmt"
    "time"
)
func main(){
    // 初始化通道
    ch11 := make(chan int, 1000)
    sign := make(chan byte, 1)

    // 给 ch11 通道写入数据
    go func ()  {
        for i := 0; i < 100; i++ {
            // go func ()  {
            //     time.Sleep(1e9)
            //         ch11 <- i
            // }()
            time.Sleep(1e9)
            ch11 <- i
        }
    }()
    // 单独起一个 Goroutine 执行 select
    //go func(){
    var e int
    ok := true
    // 首先声明一个 * time.Timer 类型的值，然后在相关 case 之后声明的匿名函数中尽可能的复用它
    var timer *time.Timer
    timer = time.NewTimer(10*time.Second)
    //expire := <- timer.C
    //fmt.Printf("Expiration time: %v.\n", expire)
    go func ()  {
        for{
            select {
                case <- timer.C:
                    fmt.Println("Timeout.")
                    ok = false
                    break
                case e = <- ch11:
                    fmt.Printf("ch11 -> %d\n",e)
            }
            // 终止 for 循环
            if !ok {
                sign <- 0
                break
            }
        }
    }()


    //}()

    // 惯用手法，读取 sign 通道数据，为了等待 select 的 Goroutine 执行。
    flag := <- sign
    fmt.Println(flag)
}


```

## 断续器
结构体类型 time.Ticker 表示了断续器的静态结构。
就是周期性的传达到期时间的装置。这种装置的行为方式与仅有秒针的钟表有些类似，只不过间隔时间可以不是 1s。
初始化一个断续器
var ticker * timeTicker = time.NewTicker(time.Second)

### 示例一：使用时间控制停止 ticke
```
package main

import (
    "fmt"
    "time"
)

func main(){
    // 初始化断续器, 间隔 2s
    var ticker *time.Ticker = time.NewTicker(1 * time.Second)

    go func() {
        for t := range ticker.C {
            fmt.Println("Tick at", t)
        }
    }()

    time.Sleep(time.Second * 5)   // 阻塞，则执行次数为 sleep 的休眠时间 / ticker 的时间
    ticker.Stop()     
    fmt.Println("Ticker stopped")
}

```

<Tick at 2015-10-31 01:29:34.41859284 +0800 CST
Tick at 2015-10-31 01:29:35.420131668 +0800 CST
Tick at 2015-10-31 01:29:36.420565647 +0800 CST
Tick at 2015-10-31 01:29:37.421038416 +0800 CST
Tick at 2015-10-31 01:29:38.41944582 +0800 CST
Ticker stopped

### 示例二：使用 channel 控制停止 ticker
```
package main

import (
    "fmt"
    "time"
)

func main(){
    // 初始化断续器, 间隔 2s
    var ticker * time.Ticker = time.NewTicker(100 * time.Millisecond)

    //num 为指定的执行次数
    num := 2
    c := make(chan int, num)
    go func() {
        for t := range ticker.C {
            c <- 1
            fmt.Println("Tick at", t)
        }
    }()

    time.Sleep(time.Millisecond * 1500)
    ticker.Stop()     
    fmt.Println("Ticker stopped")
}

```
