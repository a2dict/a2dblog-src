---
title: go chan 实现质数刷
date: 2019-03-09 13:15:24
tags: 编程
---

Go是一门为并发而生的编程语言，它将协程（goroutine）与CSP（channel）集成进语言中，这是其它编程语言中很少见的。
goroutine和channel为并发编程提供了流式的组织方式。

Show me the code!!

<!-- more -->

```go
package main

import (
    "fmt"
    "time"
)

type IntChan chan int

func NumbersChan() IntChan {
    numbers := make(IntChan)
    go func() {
        for i := 2; ; i++ {
            numbers <- i
        }
    }()
    return numbers
}

// 过滤channel中能被factor整除的val
func filter(ch IntChan, factor int) IntChan {
    ret := make(IntChan)
    go func() {
        for val := range ch {
            if val%factor != 0 {
                ret <- val
            }
        }
    }()

    return ret
}

func PrimesChan() IntChan {
    primes := make(IntChan)
    go func() {
        numbers := NumbersChan()
        for {
            prime := <-numbers
            numbers = filter(numbers, prime)
            primes <- prime
        }
    }()

    return primes

}

func main() {

    primes := PrimesChan()
    timeout := make(chan bool)
    go func() {
        time.Sleep(3 * time.Second)
        timeout <- true
    }()

    for {
        select {
        case prime := <-primes:
            fmt.Println(prime)
        case <-timeout:
            fmt.Println("timeout.")
            return
        }
    }
}
```

最后，关于chan的使用建议：

- 不要使用buffer。无论如何，buffer一定会超。chan更重要的是提供一种组织代码的方式。
- 可以考虑使用chan进行流程控制