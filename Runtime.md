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

* 由于1个字节有8位,故可以通过位为最基本单位存储许多信息</br>
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
