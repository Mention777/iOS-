# Runtime

Runtime:运行时,提供了一套C语言的api来支撑OC的动态性

### isa内部结构</br>
* 在arm64架构之前,isa就是一个普通指针,存储着类对象或原类对象的内存地址</br>
* 在arm64架构开始,对isa进行了优化,变成了一个共用体(union)结构,还使用位域来存储更多的信息,即内部结构如下

```objc
union isa_t 
{
    Class cls;
    uintptr_t bits;
    struct {
        uintptr_t nonpointer        : 1;
        uintptr_t has_assoc         : 1;
        uintptr_t has_cxx_dtor      : 1;
        uintptr_t shiftcls          : 33; 
        uintptr_t magic             : 6;
        uintptr_t weakly_referenced : 1;
        uintptr_t deallocating      : 1;
        uintptr_t has_sidetable_rc  : 1;
        uintptr_t extra_rc          : 19;
    };
};
```

* isa中各位存储的信息</br>
![](Snip20180613_2.png)

* 由于1个字节有8位,故可以通过位为最基本单位存储许多信息,但掩码的设计必须为特定的取值方式</br>
 `&`掩码 :可以用来取出特定的位</br>
 `!!`可以令一个值转换成bool类型</br>
 `|`掩码 :可以用来输入特定的位为YES</br>
 `&~`掩码 :可以用来输入特定的位为NO</br>
 
* 位域</br>

```objc
struct {
    char tall :1;
    char rich :1;
    char handsome :1;
}_tallRichHandsome;
//结构体内的1即表示位域,与左边的char及时什么类型都无关
```

* 当使用位域来进行取值时,若结果为1,则转换为bool类型会输出-1的结果,是因为bool包含8位,而一位的0b1转换为bool值会被填充为`0b1111 1111`,故结果为-1</br>
解决方案有:</br>
1.输出结果前为!!即可</br>
2.将位域改为2位,即

```objc
struct {
	char tall : 2;
	char rich : 2;
	char hansome : 2;
}
```

>tip.</br>
>结构体是无法直接位运算的</br>
>由于isa指针其中33位放地址值的,且后面3位一定为0</br>
>真机即为arm64,模拟器和mac即为x86_64

* 共用体(union):大家共用一个内存,往共用体中添加一个结构体是不影响的</br>

### Class内部结构</br>
![](Snip20180622_11.png)

* 类对象调用data()方法,结果相当就是如下的class_rw_t结构体</br>

* class_rw_t里面的methods,properties,protocols是二维数组,是可读可写的,包含了类的初始内容,分类的内容,即类和分类的声明的属性,方法,协议都在里面</br>

* class_ro_t里面的baseMethodList,baseProtocols,baseProperties是一维数组,是只读的,包含了类的初始内容,相当于只装着类声明的属性和方法,协议等初始信息,不包含分类

* 原先的bits原先是指向class_ro_t,后来重新创建了一个class_rw_t,再讲bits指向class_rw_t,class_rw_t里面的class_ro_t又指向原先的class_ro_t

>举例:methods是一个二维数组,里面每个元素是method_list_t,而method_list_t又是一个数组,数组里存放着每个元素是method_t类型元素,另外两个依次类推</br>
ro中的数组为一维数组,里面就是method_t类型,另外两个依次类推</br>
这么设计的好处是:**便于动态的添加方法**

>两者的结合:类一开始声明的属性和方法,协议等初始信息,存储在class_ro_t中对应的baseMethodList,baseProtocols,baseProperties中,在程序运行时,再将分类中的方法,协议等信息重新组合,成class_rw_t对应的二维数组,即class_rw_t中部分信息是从class_ro_t中来的


* 原类对象是一种特殊的类对象,只是里面存储的只有类方法
* 