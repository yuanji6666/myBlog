---
title: GO设计模式-单例模式
publishDate: 2026-01-14 22:00:00
description: 单例模式是设计模式中最简单基础的，本篇文章研究它的Go语言实现
heroImage: { src: 'image.png', color: '#B4C6DA' }
tags:
  - Go
  - 设计模式
language: 中文
---
# 应用背景

声明一个类，保证这个类只存在全局唯一实例供外部反复调用

# 实现模式

## 饿汉模式

程序启动之初就初始化，常用init函数初始化

规范性问题：
---
规范性问题，就是在对外可导出的 GetInstance 方法中，返回了不可导出的类型 singleton.

代码执行流程上 ok，但这种实现方式存在代码坏味道，相应的问题在 stackoverflow 上引起过讨论，对应链接如下，大家感兴趣可以去了解原贴中的讨论内容：

https://stackoverflow.com/questions/21470398/return-an-unexported-type-from-a-function

不建议这么做的原因主要在于：

-  singleton 是包内的不可导出类型，在包外即便获取到了，也无法直接作为方法的入参或者出参进行传递，显得很呆
-  singleton 的对外暴露，使得 singleton 所在 package 的代码设计看起来是自相矛盾的，混淆了 singleton 这个不可导出类型的边界和定位

综上，规范的处理方式是，在不可导出单例类 singleton 的基础上包括一层接口 interface，将其作为对对导出方法 GetInstance 的返回参数类型:

---
## 懒汉模式

只声明但是不初始化，避免浪费资源，当被调用时，判断是否被初始化，若没有才进行初始化

懒汉模式的并发问题：
---
初始化实例时，不同goroutine同时调用，会进行了多次初始化

解决方法是**一把锁，double check**

存在——return
不存在——加锁——**double_check**——初始化——解锁（doublecheck通过直接return）

---

### 标准库sync.Once解法

就是用上面的算法实现

```go
func (once sync.Once)Do(c func)
```

Do calls the function f if and only if Do is being called for the first time for this instance of [Once](https://pkg.go.dev/sync@go1.25.5#Once).

