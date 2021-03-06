---
title: JavaScript-类型
date: 2018-04-11 10:10:04
tags: JavaScript
categories: JavaScript
---
# 1.首先要搞清楚什么是内存
干吗用的？存东西用的。通电存东西，断电就GG。
外存没有内存快
最快的外存不就是SSD嘛，因为它能接近内存的速度。
机械硬盘，就很容易坏，十年基本就坏了。
SSD，不容易坏，但是一坏就全坏了。
## I.它是怎么工作的
### i.举个例子
假设内存有2G
开机
操作系统 512M
浏览器 1G （浏览器真的很占内存）
假设浏览器两个页面

```
页面1（100M）
        HTML+CSS+JS+网络HTTP+其他 
页面2（100M）
        ...
```

假设JS分到了100M
我们写了一个`var a = 1`
小心翼翼分成两个大区

```     
代码区
        a
        
数据区
        1
```

如果写复杂点

```
var a = 1
var b = 'Oracle'
b = a
var o = {
        name:'Oracle',
        age: 24
}
var c = true
o.gender = 'male'
var o2 = {
        name: 'Protoss'
}
o2 = o
第一步：变量提升，把var都放在前面
之后依次执行
```

其实数据区也分为两个——
Stack（栈）和Heap（堆）

```
Stack（栈内存）
        a:64位浮点数储存a
        b:64位浮点数储存b
        b=a:把a存的东西，覆盖到b上
        c:第一位为1后面全是0存一个true
        var o ={}:存一个地址ADDR 100 //去Heap（堆内存）找
        o.gender = 'male':找到o的地址（ADDR 100）把它加进去。
        var o2 = {}:和o一样存一个地址ADDR 200
        o2 = o: 和b = a的操作一样，不一样的是把o的地址给o2
                o2的ADDR 200变成了ADDR 100


Heap（堆内存）
        ADDR 100
                name:'Oracle'
                age: 24
                /////////////
                gender: 'male'
        ADDR 200
                name:'protoss'
```

这一套操作
### ii.数据
数字：64位
字符：16位（ES3）
值：
**简单**:存Stack
**复杂**(obj):存Heap地址（还是存在Stack里面）
### iii.引用
object存入ADDR 100
就可以称object是对象的**引用**
所有变量和对象的**关系**，都是**引用关系**
### iv.存在对象问题的时候，走一遍i里的流程就清楚了
PS：连续赋值从右往左顺序拆开写。

# 2.垃圾回收(GC)
**Garbage Collection**

我们热爱的V8引擎就是为了JavaScript在浏览器执行而打造的。
可申请的最大内存不是很高：
- 64位系统下1.4GB
- 32位系统下约700MB

如果一个`对象`**没有被引用**
它就是**垃圾**
将被回收

## i.关于活动对象
在一个函数对象被调用的时候，会创建一个活动对象
首先将该函数的每个形参和实参，都添加为该活动对象的属性和值。
将该函数体内显示声明和变量的函数，也添加为该活动的属性（在刚进入该函数执行环境的时候，未赋值，所以值为undefined，这个是JS提前声明机制）
然后将这个活动对象作为该函数执行环境的作用于链的最前端，并将这个函数对象的`[scope]`属性里作用域链接接入到该函数执行环境作用域链的后端。

这就是我们常说的，大哥罩小弟的模式。

如果一个对象和其关联对象不再通过引用关系被当前活动对象引用了，这个对象就会被垃圾回收。
**没人罩着就会被干掉**

## ii.如何判断
变量进入环境（在函数中声明一个变量）的时候，就将这个变量标记为`"进入环境"`。
变量离开环境时，则将其标记为`"离开环境"`

可以用任何方式来标记变量。例如：
- 可以通过翻转某个特殊标记来记录一个变量何时进入环境
- 或者使用一个变量列表来计入进入环境的变量，以及另一个变量列表来记录离开环境的变量

1. 垃圾回收器会在运行的时候给存储在内存中所有变量加上标记（如上所说）
2. 然后，会去掉运行环境中的变量以及被环境中变量所引用的变量的标记
3. 在此之后，依然有标记的变量就会被视为准备清除的变量，原因在运行环境中已经无法访问到这些变量了。
4. 最后，垃圾回收器完成内存清除工作，销毁那些带标记的值并回收它们所占用的内存空间。
## iii.例

```
var a = {name:'a'}
var b = {name:'b'}
a = b
```

这时候，b和a指向了同一个ADDR
那么a原来的ADDR就变成了垃圾
**没人罩着，就要被干掉...**

## vi.V8引擎的垃圾回收策略
但是由于不同对象的生存周期不同，所以无法只用一种回收策略解决问题，这样效率也不高
V8采用了一种`'代'`回收的策略，将内存分为两个`'代'`。没错，就是代沟的那个代
新生代(new generation)和老生代(old generation)

新生代中的对象为存活时间较短的对象。
老生代中的对象为存活时间较长或者常驻内存的对象，分别对新老声带采用不同的垃圾回收算法以提高效率。
对象最开始都会先被分到新生代(如果新生代占用的内存满了，那就直接分到老生代)
新生代对象会在满足某些条件后，晋升到老生代。
### 两代的内存分配
默认情况下：
32位：
- 新生代16MB
- 老生代700MB

64位：
- 新生代32MB
- 老生代1.4GB

### 新生代
新生代存的都是生存周期短的对象
垃圾回收机制——在储存空间快满的时候，就进行一次垃圾回收

1. 将新生代内存分为两部分：From(使用状态)和To(闲置状态)
2. GC进入From空间搜查一遍，看看谁没有被其他活动对象引用，没有被罩着的被无情拖走。
3. From空间里依旧被活动对象罩着的家伙被复制一遍以后放进To空间
4. 清空From空间里所有内存(上一步中被罩着的家伙在To里面幸存了下来)
5. 交换From空间和To空间(让这两个房间名交换一下)
6. 在From空间里新增新对象
7. 这时候GC又进来搜查了，接下来的故事就这么循环下去了。

由于仅仅复制存活的对象，而且对于生命周期短的场景存货对象只占少部分，所以它在时间效率上很优秀。
但是每次只能使用堆内存的一半，这是由划分空间和复制机制决定的——牺牲空间，换取时间
### 晋升
当一个对象经过多次复制依旧存活的时候
它就会被认为是生命周期较长的对象——（GC：你居然一直有大哥罩着？）
这种较长生命周期的对象随后会被移动到老生代中，采用新算法进行管理。

条件：
1. 对象从From空间复制到To空间的时候，会检查它内存地址来判断这个对象是否已经经历过一次回收。如果已经经历过了，会将该对象从From空间移动到老生代中，如果没有，则复制到To空间。**就是说，如果一个对象是二次经理从From空间复制到To空间，那么这个对象就会被移动到老生代中**
2. 要从From空间复制一个对象到To空间时，如果To空间已经使用超过25%，则这个对象直接晋升到老生代中。设置25%这个值的原因是，当本次回收完成之后，这个To空间会变成From空间，接下来的内存分配将在这个空间中进行。如果占比过高，则会影响后续的内存分配。

### 老生代
老生代中，存货对象占较大比重，如果继续采用新生代的算法管理(Scavenge)，会存在两个问题：
1. 存活对象较多，复制存活对象效率很低。
2. 采用Scavenge算法会浪费一半内存，老生代占用内存很大（远超新生代），所以浪费严重。

所以在V8老生代中主要采用Mark-Sweep和与Mark-Conpact相结合的方式进行垃圾回收

#### Mark-Sweep
Mark-Sweep（标记清除）,分为标记和清除两个截断。

与Scavenge不同，Mark-Sweep并不会将内存分为两份，所以不存在浪费一半空间的行为。
Mark-Sweep在标记截断遍历堆内存中的所有对象，并标记活着的对象
总结一下：
1. Scavenge仅复制活着的对象，而Mark-Sweep只清除死了的对象。
2. 活对象在新生代中只占较少部分，死对象在老生代中只占较少部分。

流程：
1. 遍历堆内存中所有的对象
2. GC大魔王标记存活对象
3. 大魔王GC进来巡视，回收死亡的对象（被标记的存活对象得以幸免）

但是这种方式存在问题：
- 在进行一次清除回收以后，内存空间会存在不连续（GC把没有大哥罩着的干掉了，但是他们的座位还在那空着）
- 这种内存碎片会对后续的内存分配造成问题
- 如果需要分配一个大内存的情况，由于剩余的碎片空间不足以完成此次分配，就会提前触发垃圾回收，但是这次回收并不需要被触发。

#### Mark-Compact
为了解决Mark-Sweep的内存碎片问题，提出了Mark-Compact（标记整理）
Mark-Compact由Mark-Sweep演变而来。
作用是在标记完存货对象以后，会将活着的对象向内存空间的一端移动
移动完成后，清理边界外的所有内存

流程：
1. 遍历内存中所有对象
2. GC进入标记阶段，标记存活对象
3. GC进入整理阶段，标记为存活的对象向内存空间的一侧移动，没被标记的原地待命
4. GC进入清除截断，大魔王GC把另一侧的内存全都回收了

#### 结合
- 回收策略中， Mark-Sweep 和 Mark-Compact 结合使用
- 由于 Mark-Conpact 需要移动对象，所以它执行速度慢
- 在V8取舍上，主要使用 Mark-Sweep ,在空间不足以从新生代中晋升来的对象进行分配的时候，才使用Mark-Compact

### V8垃圾处理机制总结
V8的垃圾回收机制分为新生代和老生代

新生代：
- 使用Scavenge
- Cheney算法
- 内存分两块From(使用)和To(闲置)
- 进入From + 快占满的时候存活对象进入To + 清空From + 调换From和To + 循环
- 两个条件晋升到老生代

老生代：
- Mark-Sweep （标记清除）和 Mark-Compact （标记整理）
- Mark-Sweep 碎片内存
- Mark-Compact 清除前整理 + 活的去一侧 + 清空另一侧 + 速度慢
## v.IE有个BUG
如果把页面关了(body没了，他引用的地址断了)
地址之间互相引用，但不和外部联系
他们本应该被清掉
但是（尤其IE6）IE里，你只要不关掉整个IE，他们会一直存着。
垃圾一直存着...内存boom。
所以需要在页面关闭之前执行：

```
window.onunload = function(){
        document.body.onclick = null
}
//然而需要把所有的监听的函数都设为null，需要写很多行。
```

这就是传说中的——内存泄漏：
由于浏览器的BUG,使得该被标记为垃圾的东西，没有被标记为垃圾。

# 3.深拷贝？浅拷贝？
## I.深拷贝

```
var a = 1
var b = a
b = 2
//a = 1不改变
```

b的改变不影响a
这就是深拷贝
基本类型 赋值 就是深拷贝
## II.浅拷贝

```
var a = {
        name: 'a'
}
var b = a
b.name = 'b'
a.name // 也是b了
```

指向同一个ADDR改变地址对应的内容后
a也变了。
这就是浅拷贝。
# 4.类型互相转换
## I.方法
### i.转字符串`.toString`
`null`不可以，会报错
`undefined`不可以，会报错
`object`并不能得到想要的`key:value`只会得到`[obj,Obj]`

我们热爱的`console.log`就是这个原理
如果你`console.log`的东西不是字符串，他会自动加上一个`toString`

#### `.toString`太长了老司机怎么转的？
数字：加上一个空字符串就好了。
`1 + ''`
布尔：加上一个空字符串就好了。
`true + ''`
顺序可反。
而且`'' + null`和`'' + undefined`也可以哟

#### 这时候就引出一个问题
`1+1`等于2
`1 + '1'`算什么
加号会优先加相同类型的东西，所以他会先把数字变成字符串。

#### 全局方法`window.String()`
把不是字符串的东西，变成字符串。（可以变`null`和`undefined`）

#### 所以最方便的转字符串方法是加上一个空字符串

### ii.转布尔`Boolean()`
字符串:  `Boolean(1) //true`
        `Boolean(0) //false`
        `Boolean(2) //true`
        `Boolean( ) //true`
null:   `Boolean(null) //false`
undefined: `Boolean(undefined) //false`
对象:    `Boolean({}) //true`
        `Boolean({name:'Oracle'})  //true`

#### 这个API好长...
`!`可以取反
那么取反两次不就是原来的吗

```
!!null //false
!!0 //false
!!{} //true
!!{name:'Oracle'} //true
```

#### 所有类型都可以变成布尔但是好难记哦
只有五个特殊的
number： 只有两个是false`0`和`NaN`
String： 只有空字符串是`false`其他全是`true`
null： 就一个值，`false`
undefined：就一个值`false`
object：都是`true`所有的数组和函数

也就是有五个`falsy`值
`0` 
`NaN` 
`''` 
`null` 
`undefined`
还有个`false`因为他本来就是`false`
### iii.转number
#### 朴实方法`Number('1')`
#### 全局方法`parseInt('1',10)`
`10`代表是看做十进制转换
但是`parseInt('011')  //11`
因为后面的参数没写，所以当成10进制。
用它转非数字`parseInt('s') //NaN`
数字非数字混合 `parseInt('1s') //1`
所以得到他的规则：从头开始解析能解析的，一旦遇到不能解析的，就返回。
#### 有浮点数的话`parseFloat('1.23')`
`parse`是解析
#### `'1'-0`骚方法（最常用）
任何一个东西-0就会得到一个`number`
#### 取正`+ '1'`更骚的方法
负数也可以取正？`+ '-1'`这个是可以的，结果还是`-1`。
### iv.转object
