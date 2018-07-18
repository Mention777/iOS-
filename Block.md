# Block
### 定义及本质</br>
block本质上也是一个**OC对象**,它内部有个isa指针(有isa指针就可以认为是OC对象)</br>
block是**封装了函数调用以及函数调用环境的OC对象**

```objc
int age = 20;
void (^Block)() = ^{
  NSLog(@"age = %d",age);
}
```

Block的内部结构

```objc
//以上block的内部结构
struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  int age;
};

//impl内部结构
struct __block_impl {
  void *isa;
  int Flags;
  int Reserved;
  void *FuncPtr;
}

//Desc内部结构
struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
}
```

通过将oc代码转换为c++代码可以看出

```objc

int main(int argc, const char * argv[]) { 
// 定义block变量
// 调用__main_block_impl_0函数,返回结构体变量地址
void (*block)(void) = &__main_block_impl_0(
                                           __main_block_func_0,
                                           &__main_block_desc_0_DATA
                                          );

// 执行block内部的代码
// 此时,若block有参数时,会将参数一起传入
   block->FuncPtr(block);
   //之所以block能直接调用FuncPtr是因为存储FuncPtr的impl结构体位于结构体的第一位,所以其地址与结构体的地址是一样的
   //从另一方面看,由于impl直接是结构体对象,相当于可以直接将__block_impl结构体的东西赋值过去到__main_block_impl_0中,故从这方面看也是可以直接调用的
}
 

struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  // 构造函数（类似于OC的init方法），返回结构体对象
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
 
// 封装了block执行逻辑的函数
// 若block中含有参数,则参数会在这个函数中被当做参数传递进来
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
    //定义block时,block内部执行的函数
}

```

![](Snip20180716_11.png)


### Block的变量捕获</br>
当在block内部使用外部局部变量时,block的结构体`__main_block_impl_0`中也会定义一个同名的成员变量,并在构造函数中,将外部局部变量的值赋值给内部的变量,在block定义中,若有使用到局部变量,会将外部的局部变量的值存储到结构体内部的成员变量中,所以外边的变量怎么改变,都不会修改函数定义中的变量值

**变量捕获** : 专门新建一个成员变量来保存外部的变量,称为捕获</br>
**auto变量** : 自动变量,离开作用域就销毁

![](Snip20180716_8.png)

* 当使用static修饰的局部变量,在变量捕获时,会将变量的地址值传入,在结构体内部定义的,就是一个对应的指针类型

* auto变量值传递,static变量指针传递是因为auto变量什么时候被释放是不确定的,而static修饰的局部变量会始终在内存中

* 全局变量并没有捕获,是因为函数在哪里都是可以直接使用全局变量的,故不需要捕获,而局部变量需要捕获,是因为局部变量定义是在一个函数,使用又是在另一个函数,是跨函数使用的,故需要捕获

>注:所有的方法都会附带两个参数,一个是self(方法调用者),一个是SEL _cmd(方法名),而参数都是局部变量,故在方法中的block使用self,也是会捕获的

* 今后凡是涉及到会不会捕获,只需要判断是否为局部变量即可

* 在方法中的block直接使用成员变量,即_name时,也是捕获方法调用者self,再通过self去取_name的值

