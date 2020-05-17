---
title: CSP in Go
tags:
  - 编程
date: 2020-05-15 23:16:15
---


CSP全称为Communicating Sequential Process，中文名叫*通信顺序进程*，是一种并发编程模型，由Tony Hoare于1977年提出。

在CSP模型中，线程只使用Channel传递消息进行通信，消除“共享状态”，从而简化了并发程序设计。

Go原生实现了CSP模型，很方便并发编程

<!-- more -->

## 栗子
假设一个消息系统，它接收并处理消息，处理流程如下
1. 持久化消息
2. 生成消息任务，并提交异步执行引擎
3. 异步回调处理
4. 输出并持久化任务结果

思考下，该系统应该如何设计？
也许需要两个线程池？accept pool & worker pool？消息传递使用MQ？用Manager管理整个流程？

应用CSP并发模型，工作流程如下

{% asset_img gocsp.jpg %}


在Go中，执行实体为goroutine，goroutine通过context统一管理。
使用**method**构建流程，源码如下

```go 
// 第一步，持久化并生成*任务*
func PersistInComingMessage(ctx context.Context, in <-chan *InComingMessage) <-chan *MessageTask {
    ret := make(chan *MessageTask)
    // 指定协程数量
    threads := 4
    for i := 0; i < threads; i++ {
        go func(idx int) {
            log.Printf("start persisting, id:%v\n", idx)
            for {
                select {
                case <-ctx.Done():
                    log.Printf("stop persisting thread, id:%v\n", idx)
                    return
                case msg := <-in:
                    // persisting
                    log.Printf("persist msg:%v\n", msg)

                    // new MessageTask
                    t := &MessageTask{
                        TaskID:  msg.MsgID,
                        Content: msg.Content,
                        Callback: func(stat string) {
                            log.Println("task callback..")
                        },
                    }
                    ret <- t
                }

            }
        }(i)
    }
    return ret
}

// 第二步，异步执行消息任务并生成报告
func HandleMessageTask(ctx context.Context, in <-chan *MessageTask) <-chan *TaskResult {
    ret := make(chan *TaskResult)
    threads := 5
    for i := 0; i < threads; i++ {
        go func(idx int) {
            log.Printf("start task handler, id:%v\n", idx)
            for {
                select {
                case <-ctx.Done():
                    log.Printf("stop task handler, id:%v\n", idx)
                    return
                case t := <-in:
                    // handle message task and callback
                    log.Printf("handle message task, task:%v\n", t)
                

                    tr := &TaskResult{
                        TaskID: t.TaskID,
                        Stat:   "success",
                    }
                    ret <- tr
                }
            }

        }(i)
    }

    return ret
}
```

接下来，把channel串起来吧

```go
func main() {
    // for graceful shutdown
    signalC := make(chan os.Signal, 1)
    signal.Notify(signalC, os.Interrupt, os.Kill, syscall.SIGTERM)

    ctx, cancel := context.WithCancel(context.Background())

    // 组装Channel流水线。实际代码为这3行！！
    msgC := make(chan *chain.InComingMessage)
    taskC := chain.PersistInComingMessage(ctx, msgC)
    resC := chain.HandleMessageTask(ctx, taskC)

    // 测试
    r := rand.New(rand.NewSource(time.Now().UnixNano()))
    f := fuzz.New()
    // generate message
    go func() {
        for {
            time.Sleep(time.Duration(r.Intn(8)) * time.Second)
            msg := &chain.InComingMessage{}
            f.Fuzz(msg)
            msgC <- msg
        }
    }()

    // handling task_result
    go func() {
        for {
            select {
            case <-ctx.Done():
                log.Printf("stop handling task_result")
                return
            case res := <-resC:
                log.Printf("handling task_result:%v", res)
            }

        }
    }()

    // 优雅关闭
    <-signalC
    cancel()

    time.Sleep(3 * time.Second)
    log.Fatal("exit")

```

注意，上面代码组装Channel流水线的实际代码只有3行。

## SumUp
通过这个简单例子，我们看到CSP模型对并发的抽象如何极大简化程序设计。

> Programming is not about typing, it's about thinking. - Rich Hickey

任何一种编程技术，背后都代表着某种对问题的抽象和思考方法，并提供解决方案。
这也是编程的迷人的之处——你总是能和这世界最聪明的人打交道，感受他们的智慧，并解决真实的问题。

## Appendix

- [示例源码](https://github.com/a2dict/goplayground/blob/master/gocsp/main.go)