---
title: Things about `extern`
date: 2016-08-03 16:12:42
categories: C语言
---

> All Learned From [Here](https://www.mindstick.com/Articles/1809/objective-c-understanding-extern-keyword)

## 前言
`C`和`Objective-C`的`function`前面都有个隐含的`extern`,对于`function`来说，有没有`extern`都无所谓，但变量不一样。
<!--more-->
#### `declaration` 声明可以无数次但定义只能有一次
* 声明但不定义：no memory allocation for it

```bash
extern int var;
```
> This could be done as many times as you want

> If we prepend extern in variable as default,then the memory for them will not be allocated ever.


#### `definition` 定义且声明一个变量：memory allocated

```bash
int var;
```

#### Examples that work

```
int var;

int main(void) {

var = 10;

return 0;

}
```

```
extern int var;

int main(void) {

return 0;

}
```

```
extern int var = 0;  //如果有初始化 那么内存是被分配了的

int main() {

var = 10;

return 0;

}
```

```
#include "somefile.h"

extern int var;

int main() {

var = 10;

return 0;

}
// 如果somefile.h里有var的definition，那么是可行的
```


### Examples that don't work

```
extern int var;

int main(void) {

var = 10;

return 0;

}

// var 被声明了但没有被定义,也就是没有被分配内存,所以赋值行为是错误的
```

## 总结
```bash
1. Declaration can be done any number of times but definition only once.
```
```bash
2. “extern” keyword is used to extend the visibility of variables/functions().
```
```bash
3. Since functions are visible through out the program by default. The use of extern is not needed in function declaration/definition. Its use is redundant.
```
```bash
4. When extern is used with a variable, it’s only declared not defined.
```
```bash
5. As an exception, when an extern variable is declared with initialization, it is taken as definition of the variable as well.
```