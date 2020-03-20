---
title: Groovy 3.0 发行说明
date: 2020-03-13 17:16:58
tags: 编程
---
[原文](https://groovy-lang.org/releasenotes/groovy-3.0.html)

Groovy是门被严重低估的敏捷编程语言。最近发行了3.0，简单做了下翻译

<!-- more -->

# Parrot解析器
Groovy 3.0 使用了新的代码解析器——Parrot，更具扩展性和可维护性。
Parrot解析器早期目标是，解析**代码**产生的**输出**与旧解析器完全一致，所以叫做「Parrot」。新解析器扩展了新的语法，支持新的语言特性。

## do/while 循环
现在支持Java风格的`do/while`循环。eg：
```groovy
// classic Java-style do..while loop
def count = 5
def fact = 1
do {
    fact *= count--
} while(count > 1)
assert fact == 120
```

## 增加版Java风格`for`循环
支持逗号(`,`)分隔。eg:
```groovy
def facts = []
def count = 5
for (int fact = 1, i = 1; i <= count; i++, fact *= i) {
    facts << fact
}
assert facts == [1, 2, 6, 24, 120]
```

## for循环中多参数绑定
Groovy1.6+ 支持多参数绑定：
```groovy
// multi-assignment with types
def (String x, int y) = ['foo', 42]
assert "$x $y" == 'foo 42'
```
现在也可以在for循环中使用:
```groovy
// multi-assignment goes loopy
def baNums = []
for (def (String u, int v) = ['bar', 42]; v < 45; u++, v++) {
    baNums << "$u $v"
}
assert baNums == ['bar 42', 'bas 43', 'bat 44']
```

## Java风格数组初始化
Groovy支持**方括号**(`[]`)初始化list/array，**花括号**(`{}`)预留给closure(闭包)。
Java风格的初始化与closure不冲突，所以现在支持java风格数组初始化。

eg:
```groovy
def primes = new int[] {2, 3, 5, 7, 11}
assert primes.size() == 5 && primes.sum() == 28
assert primes.class.name == '[I'

def pets = new String[] {'cat', 'dog'}
assert pets.size() == 2 && pets.sum() == 'catdog'
assert pets.class.name == '[Ljava.lang.String;'

// traditional Groovy alternative still supported
String[] groovyBooks = [ 'Groovy in Action', 'Making Java Groovy' ]
assert groovyBooks.every{ it.contains('Groovy') }
```

## Java8 lambda语法
Groovy3支持Java8 lambda语法。eg:
```groovy
(1..10).forEach(e -> { println e })

assert (1..10).stream()
                .filter(e -> e % 2 == 0)
                .map(e -> e * 2)
                .toList() == [4, 8, 12, 16, 20]
```

Groovy3还增加了**参数默认值**等特性。

```groovy
// general form
def add = (int x, int y) -> { def z = y; return x + z }
assert add(3, 4) == 7

// curly braces are optional for a single expression
def sub = (int x, int y) -> x - y
assert sub(4, 3) == 1

// parameter types are optional
def mult = (x, y) -> x * y
assert mult(3, 4) == 12

// no parentheses required for a single parameter with no type
def isEven = n -> n % 2 == 0
assert isEven(6)
assert !isEven(7)

// no arguments case
def theAnswer = () -> 42
assert theAnswer() == 42

// any statement requires braces
def checkMath = () -> { assert 1 + 1 == 2 }
checkMath()

// example showing default parameter values (no Java equivalent)
def addWithDefault = (int x, int y = 100) -> x + y
assert addWithDefault(1, 200) == 201
assert addWithDefault(1) == 101
```

### 实现细节与静态优化
动态编译时，lambda被转换成groovy closure，`(e) → { println e }`等价于`{e → println e}`。
使用`@CompileStatic`注解静态编译时，lambda被编译成java8 lambda。

## 方法引用

现在支持Java8的`::`方法引用，看例子吧。

下面是类的静态方法引用：
```groovy
import java.util.stream.Stream

// class::staticMethod
assert ['1', '2', '3'] ==
        Stream.of(1, 2, 3)
                .map(String::valueOf)
                .toList()

// class::instanceMethod
assert ['A', 'B', 'C'] ==
        ['a', 'b', 'c'].stream()
                .map(String::toUpperCase)
                .toList()
```

对象实例方法引用的例子：
```groovy
// instance::instanceMethod
def sizeAlphabet = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'::length
assert sizeAlphabet() == 26

// instance::staticMethod
def hexer = 42::toHexString
assert hexer(127) == '7f'
```

还有构造函数引用：
```groovy
// normal constructor
def r = Random::new
assert r().nextInt(10) in 0..9

// array constructor refs are handy when working with various Java libraries, e.g. streams
assert [1, 2, 3].stream().toArray().class.name == '[Ljava.lang.Object;'
assert [1, 2, 3].stream().toArray(Integer[]::new).class.name == '[Ljava.lang.Integer;'

// works with multi-dimensional arrays too
def make2d = String[][]::new
def tictac = make2d(3, 3)
tictac[0] = ['X', 'O', 'X']
tictac[1] = ['X', 'X', 'O']
tictac[2] = ['O', 'X', 'O']
assert tictac*.join().join('\n') == '''
XOX
XXO
OXO
'''.trim()

// also useful for your own classes
import groovy.transform.Canonical
import java.util.stream.Collectors

@Canonical
class Animal {
    String kind
}

def a = Animal::new
assert a('lion').kind == 'lion'

def c = Animal
assert c::new('cat').kind == 'cat'

def pets = ['cat', 'dog'].stream().map(Animal::new)
def names = pets.map(Animal::toString).collect(Collectors.joining( "," ))
assert names == 'Animal(cat),Animal(dog)'
```

### 实现细节与静态优化
虽然大部分情况下你不需要太关注方法引用的实现细节，但某些场景下可能会有用。
动态Groovy，方法引用是由closure实现的，所以`String::toUpperCase` 和`String.&toUpperCase`是一样的。
静态Groovy，方法引用使用java原生实现。

看看例子（使用JDK12）:
```groovy
@groovy.transform.CompileStatic
def method() {
  assert 'Hi'.transform(String::toUpperCase) == 'HI'
}
```
编译器产生的字节码与编译Java产生的字节码很像（包含`INVOKEDYNAMIC`、method handles和 `LambdaMetafactory`）。

如果你使用动态Groovy方法引用，不能使用`@CompileStatic`。 eg:
```groovy
def convertCase(boolean upper, String arg) {
    arg.transform(String::"${upper ? 'toUpperCase' : 'toLowerCase'}")
}
assert convertCase(true, 'Hi') == 'HI'
assert convertCase(false, 'Bye') == 'bye'
```
因为编译器不知道如何优化代码。当然实现这个功能有更合适的方法，这里只是作为演示。

同样地，当你动态使用closure时，会有警告：
```groovy
def upper = String::toUpperCase
assert upper('hi') == 'HI'
def upperBye = upper.curry('bye')
assert upperBye() == 'BYE'
```

## !in和!instanceof

现在可以`!in`和`!instanceof`了
```groovy
/* assert !(45 instanceof Date) // old form */
assert 45 !instanceof Date

assert 4 !in [1, 3, 5, 7]
```

## Elvis赋值
新加了`?=`，直接看例子吧：
```groovy
import groovy.transform.ToString

@ToString
class Element {
    String name
    int atomicNumber
}

def he = new Element(name: 'Helium')
he.with {
    name = name ?: 'Hydrogen'   // existing Elvis operator
    atomicNumber ?= 2           // new Elvis assignment shorthand
}
assert he.toString() == 'Element(Helium, 2)'
```

## 同一性（Identity）比较
在Groovy中 `==` 对应java的`equals()`
`===`和`!==` 分别对应Groovy的`is()`和`!is()`。


```groovy
import groovy.transform.EqualsAndHashCode

@EqualsAndHashCode
class Creature { String type }

def cat = new Creature(type: 'cat')
def copyCat = cat
def lion = new Creature(type: 'cat')

assert cat.equals(lion) // Java logical equality
assert cat == lion      // Groovy shorthand operator

assert cat.is(copyCat)  // Groovy identity
assert cat === copyCat  // operator shorthand
assert cat !== lion     // negated operator shorthand
```

## （空指针）安全索引

```groovy
String[] array = ['a', 'b']
assert 'b' == array?[1]      // get using normal array index
array?[1] = 'c'              // set using normal array index
assert 'c' == array?[1]

array = null
assert null == array?[1]     // return null for all index values
array?[1] = 'c'              // quietly ignore attempt to set value
assert null == array?[1]

def personInfo = [name: 'Daniel.Sun', location: 'Shanghai']
assert 'Daniel.Sun' == personInfo?['name']      // get using normal map index
personInfo?['name'] = 'sunlan'                  // set using normal map index
assert 'sunlan' == personInfo?['name']

personInfo = null
assert null == personInfo?['name']              // return null for all map values
personInfo?['name'] = 'sunlan'                  // quietly ignore attempt to set value
assert null == personInfo?['name']
```

## "var"保留类型
Groovy提供`def`用于声明类型（类型占位符），可用于类变量、本地变量、方法参数和方法返回值。动态Groovy的`def`编译时不做类型检查；在静态Groovy中，编译时推断出`def`具体类型。

Groovy3增加了`var`作为类型点位符，语法和Java10提供的`var`一致，并且可用于定义lambda参数（Java11特性）。`var`可以看做`def`的别名。

```groovy
var two = 2                                                      // Java 10
IntFunction<Integer> twice = (final var x) -> x * two            // Java 11
assert [1, 2, 3].collect{ twice.apply(it) } == [2, 4, 6]
```

## Try-with-resources

（旧）Groovy提供了更好用的Java7 try-with-resources替代品，现在Groovy3支持try-with-resources语法。

```groovy
class FromResource extends ByteArrayInputStream {
    @Override
    void close() throws IOException {
        super.close()
        println "FromResource closing"
    }

    FromResource(String input) {
        super(input.toLowerCase().bytes)
    }
}

class ToResource extends ByteArrayOutputStream {
    @Override
    void close() throws IOException {
        super.close()
        println "ToResource closing"
    }
}

def wrestle(s) {
    try (
            FromResource from = new FromResource(s)
            ToResource to = new ToResource()
    ) {
        to << from
        return to.toString()
    }
}

def wrestle2(s) {
    FromResource from = new FromResource(s)
    try (from; ToResource to = new ToResource()) { // Enhanced try-with-resources in Java 9+
        to << from
        return to.toString()
    }
}

assert wrestle("ARM was here!").contains('arm')
assert wrestle2("ARM was here!").contains('arm')
```

将输出：

```
ToResource closing
FromResource closing
ToResource closing
FromResource closing
```

## 嵌套代码块
在Java中嵌套代码块并不常见，通常这意味着应该重构代码。但某些情形它很有用，可以用来限定变量、方法作用域

```groovy
{
    def a = 1
    a++
    assert 2 == a
}
try {
    a++ // not defined at this point
} catch(MissingPropertyException ex) {
    println ex.message
}
{
    {
        // inner nesting is another scope
        def a = 'banana'
        assert a.size() == 6
    }
    def a = 1
    assert a == 1
}
```

注意Groovy的代码块和closure语法很像。当方法后面接代码快时，会被当成closure并且作为方法的最后一个参数。如果代码需要嵌套多层代码块，建议重构吧。


## Java风格非静态内部类初始化

Groovy3支持了
```groovy
public class Computer {
    public class Cpu {
        int coreNumber

        public Cpu(int coreNumber) {
            this.coreNumber = coreNumber
        }
    }
}

assert 4 == new Computer().new Cpu(4).coreNumber
```


## 接口默认方法

Java8 支持接口默认方法，现在Groovy3也支持了。不过你也可以考虑更强大的`traits`。

```groovy
interface Greetable {
    String target()

    default String salutation() {
        'Greetings'
    }

    default String greet() {
        "${salutation()}, ${target()}"
    }
}

class Greetee implements Greetable {
    String name
    @Override
    String target() { name }
}

def daniel = new Greetee(name: 'Daniel')
assert 'Greetings, Daniel' == "${daniel.salutation()}, ${daniel.target()}"
assert 'Greetings, Daniel' == daniel.greet()
```

## Parrot编译器相关

Parrot是Groovy3的默认编译器，但你也可以使用`-Dgroovy.antlr4=false`去禁用它并使用旧编译器。它用于解决旧代码可能存在的编译问题。
旧编译器不再支持语法扩展，将在Groovy4移除。

# GDK改进

Groovy3增加了大概80个新的扩展方法，有

## 用于数组和可迭代对象的`average()`
```groovy
assert 3 == [1, 2, 6].average()
takeBetween() on Strings, CharSequences and GStrings
assert 'Groovy'.takeBetween( 'r', 'v' ) == 'oo'
shuffle() and shuffled() on arrays and iterables
def orig = [1, 3, 5, 7]
def mixed = orig.shuffled()
assert mixed.size() == orig.size()
assert mixed.toString() ==~ /\[(\d, ){3}\d\]/
```
## Future的`collect{ } `
```groovy
Future<String> foobar = executor.submit{ "foobar" }
Future<Integer> foobarSize = foobar.collect{ it.size() } // async
assert foobarSize.get() == 6
```
## LocalDate的`minus()`
```groovy
def xmas = LocalDate.of(2019, Month.DECEMBER, 25)
def newYear = LocalDate.of(2020, Month.JANUARY, 1)
assert newYear - xmas == 7 // a week apart
```

# 其它改进

## 内嵌Groovydoc
(略)
## 包拆分
(包结构改了，不太关注)

# 其它「破坏性」的变更
(见原文吧)
- JDK13+用户，使用`stripIndent(true)`替换`stripIndent()`
- switch代码块，默认分支应该放在最后
- 如果扩展了`ProcessingUnit`，现在需要覆写`configure`(以前是`setConfiguration`)
- 如果重写了`GroovyClassLoader`，现在`sourceCache`和`classCache`
- Groovy3输出空Map时，括号不能省略`println([:])`

# JDK要求
编译Groovy3.0编译需要JDK9以上版本，运行最低需要JDK8。
