# GO语言学习笔记

> 对《Go编程语言》学习的记录和一些个人理解

## 第一章 初识Go语言

> Go是一门静态语言，对静态语言和动态语言的区分是在数据类型的确定是否是在编译还是运行时确认。典型的动态语言：`JavaScript`,静态语言：`Java,C++`

### 语言特性
- 自动垃圾回收
- 更丰富的内置类型
- 函数多返回值
- 错误处理
- 匿名函数和闭包  类型和接口
- 并发编程 反射
- 语言交互性

### 新增内置类型
> 相较于Java，Go语言将内置了一些新的基础类型：数据字典(Map)，数据切片（Slice，类似于C++标准库中的vector）

### 函数多返回值
> Go语言支持函数直接返回多个返回值
```go
func getName()(firstName, middleName, lastName, nickName string) { 
  return "May", "M", "Chen", "Babe"
}

fn, mn, ln, nn := getName()

//如果开发者只对该函数其中的某几个返回值感兴趣的话，也可以直接用下划线作为占位符来 忽略其他不关心的返回值。
//下面的调用表示调用者只希望接收lastName的值，这样可以避免声 明完全没用的变量:
_, _, lastName, _ := getName()

```
### 错误处理
> Go语言引入了3个关键字用于标准的错误处理流程，这3个关键字分别为defer、panic和 recover

### 匿名函数和闭包
> 在Go语言中，所有的函数也是值类型，可以作为参数传递；Go的匿名函数就是一个闭包，闭包是可以包含自由(未绑定到特定对象)变量的代码块，这些变量不在这个代码快或者任何全局上下文中定义，而是在定义代码块的环境中定义。
```go
f := func(x, y int) int { 
  return x + y //第二个int是返回值类型；x,y 是相同类型 int
}
```
### 类型和接口
> Go语言的类型定义非常接近于C语言中的结构(struct)，甚至直接沿用了struct关键字。相 比而言，Go语言并没有直接沿袭C++和Java的传统去设计一个超级复杂的类型系统，不支持继承 和重载，而只是支持了最基本的类型组合功能。
```go
type Bird struct { ...
}
func (b *Bird) Fly() { // 以鸟的方式飞行
}

// 在实现Bird类型时完全没有任何IFly的信息。我们可以在另外一个地方定义这个IFly接口:
type IFly interface { 
  Fly()
}

// 这两者目前看起来完全没有关系，现在看看我们如何使用它们:
func main() {
  var fly IFly = new(Bird) fly.Fly()
}
```
### 并发编程
> Go语言引入了goroutine概念，通过使用goroutine而不是裸用 操作系统的并发机制，以及使用消息传递来共享内存而不是使用共享内存来通信，Go语言让并 发编程变得更加轻盈和安全。函数调用前使用关键字go，我们即可让该函数以goroutine方式执行。goroutine是一种 比线程更加轻盈、更省资源的协程。Go语言通过系统的线程来多路派遣这些函数的执行，使得 每个用go关键字执行的函数可以运行成为一个单位协程。当一个协程阻塞的时候，调度器就会自 动把其他协程安排到另外的线程中去执行，从而实现了程序无等待并行化运行。而且调度的开销 非常小，一颗CPU调度的规模不下于每秒百万次，这使得我们能够创建大量的goroutine，从而可 以很轻松地编写高并发程序，达到我们想要的目的。
```go
//这是一个并行计算的例子，由两个goroutine进行并行的累加计算，待这两个计算过程都完成后打印计算结果，来演示goroutine和channel的使用方式：
package main

import "fmt"

func sum(values [] int, resultChan chan int) { 
  sum := 0
  for _, value := range values { 
    sum += value
  }
  resultChan <- sum // 将计算结果发送到channel中
}
func main() {
  values := [] int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
  
  resultChan := make(chan int, 2)
  go sum(values[:len(values)/2], resultChan)
  go sum(values[len(values)/2:], resultChan)
  sum1, sum2 := <-resultChan, <-resultChan // 接收结果
  
  fmt.Println("Result:", sum1, sum2, sum1 + sum2)
}
```
### 反射
> 反射可以获取对象类型的详细信息，并可动态操作对象Go语言的反射实现了反射的大部分功能，但没有像Java语言那样内置类型工厂，故而无法做 到像Java那样通过类型字符串创建对象实例。反射最常见的使用场景是做对象的序列化(serialization，有时候也叫Marshal & Unmarshal)。
```go
package main

import (
  "fmt"
  "reflect"
)

type Bird struct {
  Name string
  LifeExpectance int 
}

func (b *Bird) Fly() { 
  fmt.Println("I am flying...")
}

func main() {
  sparrow := &Bird{"Sparrow", 3}
  s := reflect.ValueOf(sparrow).Elem() 
  typeOfT := s.Type()
  for i := 0; i < s.NumField(); i++ {
    f := s.Field(i)
    fmt.Printf("%d: %s %s = %v\n", i, typeOfT.Field(i).Name, f.Type(),
      f.Interface())
  } 
}
//该程序的输出结果为:
    0: Name string = Sparrow
    1: LifeExpectance int = 3
```
### 语言交互性
> Go语言可以按照Cgo的特定语法去调用C模块，如下：
```go
package main
/*
#include <stdio.h> */
import "C"
import "unsafe"
func main() {
  cstr := C.CString("Hello, world") 
  C.puts(cstr) 
  C.free(unsafe.Pointer(cstr))
}
```



