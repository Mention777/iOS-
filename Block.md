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