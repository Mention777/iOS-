# Category

### Category内部实现</br>

```objc
//分类的结构:
struct _category_t {
    const char *name;//分类所属的类名
    struct _class_t *cls;
    const struct _method_list_t *instance_methods;
    const struct _method_list_t *class_methods;
    const struct _protocol_list_t *protocols;
    const struct _prop_list_t *properties;
}
```
程序一编译,分类的信息都会存储在_category_t这个结构体下,相当于编写出一个分类,就生成了一个对应的结构体对象,例如有5个分类,到时就会生成对应5个结构体对象,每个结构体变量的变量名和内部存储的值是不同的,此时并没有合并到类对象对应的方法中

* 误区:一个类值存在一个类对象,所以不存在说创建一个分类就创建一个对应分类的对象这么一个情况
* 分类中的方法合并:通过runtime动态将分类的方法合并到类对象、元类对象中

>分类的内部实现过程:</br>
>1.通过runtime加载某个类的所有category数据</br>
>2.把所有category的方法、属性、协议数据合并到一个大数组中(其中,后面参与编译的category数据,会在数组的前面)</br>
>3.将合并后的分类数据(方法、属性、协议),插入到原来数据的前面</br>
>>* 由源码可以得出,最后面编译的分类,在方法列表中位置最靠前
>>* 在xcode中Build Phases -  Compile Sources中可以手动设置编译的顺序

category(分类)与extension(类扩展)的区别:
* 类扩展内容一编译就已经合并到类中了,分类中是在运行时通过runtime合并到类信息中

category的实现原理:
* category编译之后的底层结构是struct category_t,里面存储着分类的对象方法、类方法、属性、协议信息
* 在程序运行的时候,runtime会将category的数据,合并到类信息中(类对象、元类对象中)

通过查看源码,发现底层将category插入数组时,调用两个函数`memmove`和`memcpy`,二者的区别是:</br>
memmove:会先判断是往左挪还是往右挪(会保证完整性)</br>
memcpy:会一个一个拷贝(从小地址开始)
>例如存在下面4块内存空间 1  2  3  4</br>
>现在要将1和2,移动到2和3的位置,通过memmove可以实现3 1 2 4</br>
>通过memcpy实现即为:1 1 1 4

### +load()方法</br>
* 会在runtime加载类、分类时调用
* 每个类、分类的+load,在程序运行过程中只调用一次

从源码分析,会先调用类的load方法,再调用分类的load方法</br>
之所以运行时,类调用load方法和分类调用load方法不会像其他方法一样被覆盖,是因为内部实现中,会直接取出load方法的函数地址直接进行调用,而不是通过消息转发(objc_msgSend)

>如果是通过消息机制调用方法,流程都是通过isa指针,找到对应的类方法/元类方法,在方法列表中按顺序查找</br>
而load方法是直接通过load方法在内存中的地址值直接调用的

* 调用顺序:</br>
  1.先调用类的load方法</br>
  2.按照编译的先后顺序调用(先编译,先调用)</br>
  3.调用子类的load方法之前会先调用父类的load方法</br>
  4.调用分类的load方法</br>
  5.按照编译的先后顺序调用(先编译,先调用)</br>