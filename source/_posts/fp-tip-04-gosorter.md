---
title: "FP技巧04：Go排序器"
date: 2019-03-25 20:37:30
tags: fp
---
> 代码传送：https://github.com/a2dict/sorter

在函数式编程中，通过（纯）函数的组合，可以方便地构造出复杂的函数，且没有副作用，对并发友好。我觉得这正是FP优雅、迷人的地方。
Go语言虽然不是函数式编程语言，但支持First-class Function（一等函数？即支持函数作为基本数据），所以可能利用部分函数式特性，扩展编程能力。

工作中，需要对结构数据进行内存排序，无奈Go的排序接口特别丑陋，所以产生了扩展函数式排序能力，编写一个比较器的想法。

```go
// Go的排序接口
// A type, typically a collection, that satisfies sort.Interface can be
// sorted by the routines in this package. The methods require that the
// elements of the collection be enumerated by an integer index.
type Interface interface {
	// Len is the number of elements in the collection.
	Len() int
	// Less reports whether the element with
	// index i should sort before the element with index j.
	Less(i, j int) bool
	// Swap swaps the elements with indexes i and j.
	Swap(i, j int)
}
```

我的想法是，将`函数式比较器`和`数据`包装成一种实现`排序接口`的数据类型，这样就可以借助FP的的力量，构建复杂排序器。

<!-- more -->

```go
// 函数类型
type Comparator func(a, b interface{}) int
type Predicate func(a interface{}) bool
type Extractor func(a interface{}) interface{}
// 数据结构
type sorter struct {
	data interface{}
	cmpr Comparator
}

// 实现 排序接口，应用reflect
func (s sorter) Len() int {
	return reflect.ValueOf(s.data).Elem().Len()
}
func (s sorter) Less(i, j int) bool {
	arr := reflect.ValueOf(s.data).Elem()
	a := arr.Index(i).Interface()
	b := arr.Index(j).Interface()
	res := s.cmpr(a, b)
	if res < 0 {
		return true
	}
	return false
}
func (s sorter) Swap(i, j int) {
	if i > j {
		i, j = j, i
	}
	arr := reflect.ValueOf(s.data).Elem()

	tmp := arr.Index(i).Interface()
	arr.Index(i).Set(arr.Index(j))
	arr.Index(j).Set(reflect.ValueOf(tmp))
}

// 一些辅助函数
// 参数反转，用于倒序排序
func (c Comparator) flip() Comparator {
	return func(a, b interface{}) int {
		return c(b, a)
	}
}
// Extractor转Comparator
func (e Extractor) toComparator() Comparator {
	return func(a, b interface{}) int {
		ea := e(a)
		eb := e(b)
		return ordering(ea, eb)
	}
}
```

然后，包装下排序函数，排序器就成了
```go
// 构造方法
func NewSorter() *sorter {
	return &sorter{}
}
// 基础排序方法
func (s *sorter) Comparing(comparator Comparator) *sorter {
	if s.cmpr == nil {
		return &sorter{cmpr: comparator}
	}
	return &sorter{cmpr: func(a, b interface{}) int {
		res := s.cmpr(a, b)
		if res != 0 {
			return res
		} else {
			return comparator(a, b)
		}
	}}
}
// 函数组合构造更多排序方法
func (s *sorter) ComparingBy(extractor Extractor) *sorter {
	return s.Comparing(extractor.toComparator())
}
func (s *sorter) ReversedComparing(comparator Comparator) *sorter {
	return s.Comparing(comparator.flip())
}
func (s *sorter) ReversedComparingBy(extractor Extractor) *sorter {
	return s.Comparing(extractor.toComparator().flip())
}
```

使用示例：
```go
func TestSorter(t *testing.T) {
	data := []Man{
		Man{Name: "zhang3", Age: 24, Gender: 1},
		Man{Name: "li4", Age: 21, Gender: 0},
		Man{Name: "wang5", Age: 26, Gender: 0},
		Man{Name: "zhao6", Age: 24, Gender: 1},
	}

	NewSorter().ComparingBy(func(a interface{}) interface{} { //先按Gender升序
		return a.(Man).Gender
	}).ReversedComparingBy(func(a interface{}) interface{} { //再按Age倒序
		return a.(Man).Age
	}).ComparingBy(func(a interface{}) interface{} { //再按Name升序
		return a.(Man).Name
	}).Sort(&data)
	t.Log(data)
}
```