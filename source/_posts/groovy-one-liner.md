---
title: "酷到没朋友的Groovy代码"
date: 2019-04-15 21:59:20
tags: groovy
---

> 原文地址: 
> https://www.javaworld.com/article/2074145/ten-groovy-one-liners-to-impress-your-friends.html
> https://www.javaworld.com/article/2074147/ten-more-groovy-one-liners.html

1、数组 x2

```groovy
(1..10).collect{it * 2}
```

2、数组求和

```groovy
(1..1000).sum()
```

3、检查字符串是否包含单词
```groovy
def wordList = ["Groovy", "dynamic", "Grails", "Gradle", "scripting"]
def string = "This is an example blog talking about Groovy and Gradle."
wordList.inject(false){ acc, value -> acc || string.contains(value)}
```
<!-- more -->
4、读文件
```groovy
new File("data.txt").text
new File("data.txt").readLines()
```

5、生日快乐
```groovy
(1..4).collect{"Happy Birthday " + ((it == 3) ? "dear ${name}" : 'to You')}.each{line -> println line}
```

6、数组过滤
```groovy
def (passed, failed) = [49, 58, 76, 82, 88, 90].split{it > 60}
```

7、xml解析
```groovy
def content = new XmlSlurper().parse("http://search.twitter.com/search.atom?&q=groovy")
```

8、最大/小值
```groovy
[14, 35, -7, 46, 98].min()
[14, 35, -7, 46, 98].max()
```

9、并行
```groovy
groovyx.gpars.GParsPool.withPool{def result = dataList.collectParallel{processItem(it)}}
```

10、质数刷(Sieve of Eratosthenes)
```groovy
def t = 2..100
(2..Math.sqrt(t.last())).each { n -> t -= ((2*n)..(t.last())).step(n) }
println t
```

11、FizzBuzz
```groovy
(1..100).each{println "${it%3?'':'Fizz'}${it%5?'':'Buzz'}" ?: it }
```

12、JVM时区及ID
```groovy
TimeZone.getAvailableIDs().sort().each{ println it }
```

13、文件拷贝
```groovy
new File("NewWordDocument.doc").bytes = new File("WordDocument.doc").bytes
```

14、获取主机信息
```groovy
println "${InetAddress.getLocalHost()}"
println "${InetAddress.getLocalHost()}\n${InetAddress.getLocalHost().getLoopbackAddress()}"
```

15、抓网页
```groovy
println "http://java.net".toURL().text 
```

16、字符串拼接
```groovy
['Dustin', 'Inspired', 'Actual', 'Events'].join("_")
```

17、环境变量
```groovy
System.getenv().each{println it}
```

18、删除目录及子目录、文件
```groovy
new File('trashDir').deleteDir()
```