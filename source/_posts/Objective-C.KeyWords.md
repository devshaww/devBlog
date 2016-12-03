---
title: Objective-C属性约束关键字杂谈
date: 2016-04-06 21:57:49
categories: iOS
toc: true
---
```bash
** copy retain assign的差别在于对象属性的"set"方法 **
```
## NSString 与 NSMutableString
NSString是`不可变`字符串对象，这句话的意思，结合代码：
<!-- more -->
```
#import <Foundation/Foundation.h>

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        NSString *str = @"Shaw";
        NSString *str1 = @"Root";   // NSString *str1的意思是str1指向的@"Root"对象是不可变的,但str1是可以改变指向的。
        NSLog(@"str = %@, str1 = %@",str,str1);
        NSLog(@"str:%p, str1:%p",str,str1);
        str = [str stringByAppendingString:@"andRoot"];  // 打印可以看到str地址变了，因为原地址下的对象是不可变的。
        NSLog(@"str:%p, str1:%p",str,str1);
        str1 = str;  // 使str1指向str对象
        NSLog(@"str = %@, str1 = %@",str,str1);
        NSLog(@"str:%p, str1:%p",str,str1);
    }
    return 0;
}
// 输出结果
2016-04-06 13:32:45.320 test[40011:4790098] str = Shaw, str1 = Root
2016-04-06 13:32:45.321 test[40011:4790098] str:0x100001050, str1:0x100001070
2016-04-06 13:32:45.321 test[40011:4790098] str:0x1001026b0, str1:0x100001070
2016-04-06 13:32:45.321 test[40011:4790098] str = ShawandRoot, str1 = ShawandRoot
2016-04-06 13:32:45.321 test[40011:4790098] str:0x1001026b0, str1:0x1001026b0
Program ended with exit code: 0
```

同理，`NSMutableString`就是`可变`字符串对象。

`stringByAppendingString:`方法的定义为
```bash
- (NSString *)stringByAppendingString:(NSString *)aString;
```

```
NSMutableString *mStr = [NSMutableString stringWithString:@"Shaw"];
        NSLog(@"%@ %p",mStr,mStr);
        [mStr appendString:@"andRoot"]; //  可以看到输出地址为同一个，即对当前对象做了改变。
        NSLog(@"%@ %p",mStr,mStr);

// 输出结果
2016-04-06 16:17:05.789 test[40118:4806417] Shaw 0x100203470
2016-04-06 16:17:05.790 test[40118:4806417] ShawandRoot 0x100203470
Program ended with exit code: 0
```
如果用`NSMutableString`对象调用`stringByAppendingString:`方法会出现警告`"Incompatible pointer types assigning NSMutableString to NSString"`

## 

## mutableCopy(遵从`NSMutableCopying`协议的对象可用) 与 copy (遵从`NSCopying`协议的对象可用)

`mutableCopy`返回的对象是`可变`的, `copy`返回的是`不可变`的。

所以用`copy`返回的字符串是`NSString *`, `mutableCopy`返回的字符串是`NSMutableString *`.

* 复制不可变对象时

	* `copy`是浅复制，即指针复制，两个指针都指向那块空间

	* `mutableCopy`是深复制，即新开辟一块空间，将对象复制过去

* 复制可变对象时

	* `mutableCopy`与`copy`都是深复制,但`copy`返回的对象不可变
	> 这个很好理解，因为`copy`返回的是`不可变对象`，那么当用`copy`复制一个`可变对象`，复制前`可变`，复制后`不可变`，当然得另开空间。

** 下面可以用代码一一验证 **

```Objective-C

//      复制不可变对象
        NSString *str = @"Shaw";
        NSString *strCopy = [str copy];    // 相当于 [str retain]
        NSMutableString *strMCopy = [str mutableCopy];

        NSLog(@"%@:%p %@:%p %@:%p",str,str,strCopy,strCopy,strMCopy,strMCopy);
        strCopy = [str stringByAppendingString:@"andRoot"];
           strMCopy = [strMCopy stringByAppendingString:@"andRoot"];
        NSLog(@"%@:%p %@:%p %@:%p",str,str,strCopy,strCopy,strMCopy,strMCopy);

//  输出结果
2016-04-06 18:06:17.399 test[40355:4843127] Shaw:0x100001050 Shaw:0x100001050 Shaw:0x100600380
2016-04-06 18:06:17.400 test[40355:4843127] Shaw:0x100001050 ShawandRoot:0x100103870 Shaw:0x100600380

```

> 用`copy`方法复制`NSString`对象, 出于性能原因, 既然二者本身都不可变，那么不如直接返回源对象，所以二者返回地址一样，所以是浅复制，也就相当于对源对象做`retain`.

> 因为`strCopy`是`NSString`对象，向其发送`stringByAppendingString:`消息，是开辟了另一块空间`(0x100103870)`的，并没有改变之前那块空间`(0x100001050)`的对象.

```

// 复制可变对象

        NSMutableString *mStr = [NSMutableString stringWithString:@"Shaw"];
        NSMutableString *strMCopy = [mStr mutableCopy];
        NSString *strCopy = [mStr copy];

//        NSLog(@"%@:%p %@:%p %@:%p",mStr,mStr,strMCopy,strMCopy,strCopy,strCopy);

        NSString *mStrCopy = [mStr copy]; // 打印结果可以看出mStrCopy和strCopy其实是同一个对象，也就是说可变对象的copy只能新开一块空间出来，之后再copy也都是指向这块空间的对象。

        NSLog(@"%@:%p %@:%p %@:%p %@:%p",mStr,mStr,strMCopy,strMCopy,strCopy,strCopy,mStrCopy,mStrCopy);

        strMCopy = [strMCopy stringByAppendingString:@"andRoot"]; // 会放在新地址
        strCopy = [strCopy stringByAppendingString:@"andRoot"];  //  会放在新地址

        NSLog(@"%@:%p %@:%p %@:%p",mStr,mStr,strMCopy,strMCopy,strCopy,strCopy);


//  输出结果

2016-04-06 19:15:09.413 test[40489:4866634] Shaw:0x100400290 Shaw:0x1004004d0 Shaw:0x7761685345 Shaw:0x7761685345
2016-04-06 19:15:09.414 test[40489:4866634] Shaw:0x100400290 ShawandRoot:0x100400900 ShawandRoot:0x100400550

```

* * * * * * * * *

## 为什么NSString对象在@property属性声明时写的是copy?

```Objective-C

// 假设Person类中有声明如下
// .h

@property (nonatomic, copy) NSString *name;

// main.m

Person *per = [Person alloc] init];
NSMutableString *mStr = [NSMutableString stringWithFormat:@"Root"];
per.name = mStr;
NSLog(@"name is %@ now", per.name);
[mStr appendString:@"andShaw"];
NSLog(@"name is %@ now ", per.name);

// 输出结果会是这样
name is Root now
name is Root now

```
> 而将copy改为retain
> 输出结果会是
> name is Root now
> name is RootandShaw now

###### set方法的例子

```
setName:(NSString *)name {
    if (_name != name) {
        [_name release];
        _name = [name retain];     // copy时此处将retain换成copy
    }
}

```

## 小结
复制方法存在的目的就是为了复制出一个当对它做出改变而不会影响源对象的对象.

当然如果想改变一个NSString对象也不是不可以

比如
```
NSString *str =@"Shaw";
NSLog("str = %@, addr = %p", str, str);
NSString *__strong *p = &str;
*p = @"Root";
NSLog("str = %@, addr = %p", str, str);

//  输出结果在同一个地址，且已经改变

```
至于`__strong` ,  是对象`ownership`的话题了.
> 这方面的知识我是在[Objective-C高级编程:iOS与OS X多线程和内存管理](https://www.amazon.cn/gp/product/B00DE60G3S/ref=oh_aui_detailpage_o03_s00?ie=UTF8&psc=1)上获取的。

## retain weak strong（图来自[RayWenderlich](https://www.raywenderlich.com/5677/beginning-arc-in-ios-5-part-1)）
`weak` `strong`是`ARC`引入的关键词

* `strong`

```bash
NSString *firstName = @"Ray";
```
firstName对@"Ray"强引用

![image](http://www.raywenderlich.com/wp-content/uploads/2011/10/Pointers1.png)

此时猜想一个textField,当输入Ray

```bash
self.textField.text = @"Ray";
```

此时有两个strong指针指向此对象
![image](http://www.raywenderlich.com/wp-content/uploads/2011/10/Pointers2.png)

textField中的文字变化 变成Rayman

此时就变成下图的样子
![image](http://www.raywenderlich.com/wp-content/uploads/2011/10/Pointers3.png)

只有当firstName被赋予新值，或者含有此局部变量的方法结束，或者因为firstName是一个实例变量且它所属的那个对象已经deallocated，这种所有权才结束。

当@"Ray"不再被任何强指针拥有，它就被释放了。
![image](http://www.raywenderlich.com/wp-content/uploads/2011/10/Pointers4.png)

把firstName和self.textField.text这种指针称为strong指针，因为它们使对象存在于内存中。

默认情况下的实例变量，局部变量都是强指针。

<br/>

* `weak`
```bash
__weak NSString *weakName = self.textField.text;
```
> `weak`指针是需要显式声明的,用`__weak`关键字

![image](http://www.raywenderlich.com/wp-content/uploads/2011/10/Pointers5.png)
weakName指向对象但并没有拥有它，如果textField内容发生变动，那么@"Rayman"对象不再被任何强指针指向，它会被释放掉。

![image](http://www.raywenderlich.com/wp-content/uploads/2011/10/Pointers6.png)

weak比assign多了一个作用就是当它指向的对象已经被销毁了，它会自己置成nil。

这是非常方便的，因为这防止弱指针继续指向那片已经被释放了的空间，曾经因为这个问题造成了很多bug，你也许有听过"悬摆指针""野指针"，但是有了weak指针，这些问题不会出现了。

weak指针多数被用到有父子关系的两个对象上，父对象用strong指向子对象，子对象用weak指向父对象，这样就避免了内存循环，下面是一个例子

![image](http://www.raywenderlich.com/wp-content/uploads/2011/10/Pointers7.png)

如果是从`storyboard`中连了线到代码块的，那都是添加到了父视图的子视图树里的，也就是说视图是有父对象的强指针指向的.

```
//
//  ViewController.h
//  test2
//
//  Created by Shaw on 16/4/6.
//  Copyright © 2016年 Shaw. All rights reserved.
//

#import <UIKit/UIKit.h>

@interface ViewController : UIViewController

@property (weak, nonatomic) IBOutlet UITableView *tableView;

@property (nonatomic,weak) UIScrollView *scrollView;

@end

```

```
#import "ViewController.h"

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    UIScrollView *scrollView = [[UIScrollView alloc]initWithFrame:CGRectMake(0, 353, 400, 200)];
    _scrollView = scrollView;
    [self.view addSubview:self.scrollView];

}

```
> 这几句代码可以用下图来描述

![image](https://drops.azureedge.net/drops/files/acc_521713/Ia1G?rscd=inline%3B%20filename%3DScreenshot%2520on%25209.20.2016%2520at%25203.05.12%2520PM.png&rsct=image%2Fpng&se=2016-09-23T02%3A53%3A25Z&sig=oB01ChXXlLW7KOw%2BWLwFskHPT6jOHZDkuLfRvoZJUas%3D&sp=r&sr=b&st=2016-09-23T01%3A53%3A25Z&sv=2013-08-15)

担心`scrollView`对象有两个强指针指向不好释放？

`scrollView`变量是个局部变量，出了那个方法，就被释放掉了，之后就只有`self`一个强指针指向它了。
<br/>
* `retain`
`retain`在`ARC`下是不能显式写的，但是在`@property(nonatomic,retain)`这样写是没问题的。

`retain`的属性的`setter`是先`release`旧值，再`retain`新值
```

@property (nonatomic,retain) NSString *string;   
// 当赋给string属性的对象总是NSString *,那么用retain和copy都是一样的

-(void)setString:(NSString *)str{

//　　if(str == _string){

//　　　　return;

// }

　　[_string release];

　　_string = [str retain];

}

```
