# OC对象的本质

####从一道面试题开始####
>Q:一个OC对象占用多少内存?</br>
>该问题即是问`NSObject *obj = [[NSObject alloc]init]`,中`obj`占多少内存?

>A:系统分配了16个字节给`NSObject`对象(通过`malloc_size`函数获得),但`NSObject`对象内部只使用了8个字节的空间(64位环境下,通过`class_getInstanceSize`获得)

OC对象/类主要是基于****结构体****这个数据结构实现的

要想探究OC对象的本质,需要先将OC代码转化为C/C++代码窥探其源码,具体的指令如下:</br>
`xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc oc源文件 -o  输出的文件名.cpp`


