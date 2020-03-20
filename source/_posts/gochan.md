---
title: go chanel组织多阶段任务
date: 2020-01-10 15:10:48
tags: 编程
---

以下是一个使用go chan组织多阶段任务的例子。多用`func`，少用`struct`，对所有语言都适用 :)

<!-- more -->

```go
import (
	"fmt"
	"time"
)

type Flour string
type Mixed string
type Pie string

// 混合面粉。 第一步
func Mix(ch chan Flour) chan Mixed {
	ret := make(chan Mixed)
	go func() {
		for f := range ch {
			ret <- Mixed("mixed with" + string(f))
		}
	}()
	return ret
}

// 烤。 第二步
func Bake(ch chan Mixed) chan Pie {
	ret := make(chan Pie)
	go func() {
		for m := range ch {
			ret <- Pie("pie, " + string(m))
		}
	}()
	return ret
}

func main() {
	// make pie
	flourIn := make(chan Flour)
	pieOut := Bake(Mix(flourIn))
	go func() {
		i := 1
		for {
			time.Sleep(3 * time.Second)
			flourIn <- Flour(fmt.Sprintf("flour%02d", i))
			i++
		}
	}()

	done := time.After(10 * time.Second)
	for {
		select {
		case pie := <-pieOut:
			fmt.Printf("Get a Pie: %v\n", pie)
		case <-done:
			fmt.Println("done.")
			return
		}
	}
}
```
