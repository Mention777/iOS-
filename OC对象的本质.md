# OC对象的本质

从一道面试题开始
>Q:一个OC对象占用多少内存?</br>
>该问题即是问`NSObject *obj = [[NSObject alloc]init]`,中`obj`占多少内存?

>A:系统分配了16个字节给`NSObject`对象(通过`malloc_size`函数获得),但`NSObject`对象内部只使用了8个字节的空间(64位环境下,通过`class_getInstanceSize`获得)

要想探究OC对象的本质,需要先将OC代码转化为C/C++代码窥探其源码,具体的指令如下:</br>
`xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc oc源文件 -o  输出的文件名.cpp`

OC对象/类主要是基于****结构体****这个数据结构实现的,在内存中即是如下的结构体</br>
```objc
struct NSObject_IMPL {
      Class isa;
}
//其中Class是一个指向结构体的指针,在64位架构中,占8位字节(32位占4字节)
```

>P.S结构体中第一个元素在内存中的地址就是结构体的地址

```objc
Person *person = [[Person alloc] init];

//获得Person实例对象的成员变量所占用的大小,该方法返回的是一个对齐过的成员变量的大小
class_getInstanceSize([Person class])
//获得person指针所指向内存的大小
malloc_size((__bridge const void *)person)

//p.s 将oc对象转换为c语言对象,需要对其进行桥接,及(__bridge C语言类型)变量 的方式

```
若存在继承关系,会将父类的属性放在最前边,即父类结构体的实现

ios都为小端模式,读取数据是从高地址开始读取

**内存对齐**:结构体的大小必须是最大成员大小的倍数

若存在多继承,则子类的成员会优先填充父类填充后的内存,若大小还够填充,则会直接在剩余位置填充

创建出来的实例对象是不放方法的,只存放成员变量,方法是不会放到实例对象中的,因为对应的`setter`和`getter`都是共用一份,所以没必要存放,每个对象存放的只要是存放自己特有的东西即可

由源码可以看出,`malloc_size`方法本质上调用的是`class_getInstanceSize`的值进行内存分配,即`malloc_size`分配内存的大小与`class_getInstanceSize`是一样的,
内存分配时,也存在着内存对齐,只是,该内存对齐是16的倍数

**class_getInstanceSize**:实例对象至少需要多少存储空间,与sizeof函数基本相同</br>
**malloc_size**:实际分配多少内存

**sizeof**:其实是一个运算符,一编译就会被转换为一个常数,与另外两个函数是有区别的.

```objc
int p = 10;
sizeof(p)
//sizeof(p)在汇编时会直接转化成sizeof(int) = 4
//即在编译时就已经知道占据4个字节
```
