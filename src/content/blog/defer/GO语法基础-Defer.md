---
title: GO语法基础-Defer
publishDate: 2026-01-27 14:50:00
description: defer用于函数/方法的延迟调用
tags:
  - Go
  - 基础语法
language: 中文
heroImage: '{"src":"./image.png","color":"#B4C6DA"}'
---
# Defer

> Go's `defer` statement schedules a function call (the _deferred_ function) to be run immediately before the function executing the `defer` returns. It's an unusual but effective way to deal with situations such as resources that must be released regardless of which path a function takes to return. The canonical examples are unlocking a mutex or closing a file.

[官方文档](https://go.dev/doc/effective_go#defer)
# 执行顺序

LIFO 依次压栈，执行顺序与defer的顺序相反
# 参数求值时机

> The arguments to the deferred function (which include the receiver if the function is a method) are evaluated when the _defer_ executes, not when the _call_ executes.Besides avoiding worries about variables changing values as the function executes, this means that a single deferred call site can defer multiple function executions.

defer的函数的参数，在defer执行时就已经确定，而不是函数调用时：
```go
type Test struct{
	a int
}
func (t Test) print(x int){
	fmt.Printf("a is %v, x is %v ", t.a, x)
}
func deferRun() int {
	test := Test{1}
	num := 1 
	defer test.print(num)
	num++
	test.a++
	
	return 0
}
func main(){
	deferRun()	
}
```

运行结果：
```go
a is 1, x is 1
```

# return时具体做了什么？

我们知道，defer 函数在return时调用，但是具体做了什么？

只需弄清楚三步：
1.  设置result
2.  执行defer
3.  返回result

来看具体例子：

```go
func main() {
   res := deferRun()
   fmt.Println(res)//输出2
}
func deferRun() (res int) {
  num := 1
  defer func() {
    res++
  }()  
  return num
  // res = num 
  // res ++
  // return res
}
```

无论怎样的过程，从这三步分析就不会出错