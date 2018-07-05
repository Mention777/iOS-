# Runloop

Runloop作用:</br>
* 保持程序的持续运行
* 处理程序的各种事件(触摸事件、定时器事件等)
* 节约CPU资源,提高程序性能

程序中Main函数中的UIApplicationMain函数主要作用就是创建了一个主运行循环

Runloop有两种形式:</br>
* OC语言-- NSRunloop
* C语言 -- CFRunloopRef

Runloop与线程的关系:
* 每一条线程都有一个与之对应的runloop对象
* runloop保存在全局的字典内,是以线程为key,runloop为value
* 线程刚创建时没有runloop对象的,runloop会在第一次获取他的时候创建
* runloop在线程销毁时销毁


Runloop与Mode的关系:</br>
![](Snip20180601_27.png)


