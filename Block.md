# Block
### Block定义及本质</br>
block本质上也是一个**OC对象**,它内部有个isa指针(有isa指针就可以认为是OC对象)</br>
block是**封装了函数调用以及函数调用环境的OC对象**</br>
>函数调用环境:函数调用所需要的参数及外部参数等

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

### Block的类型</br>
block有3种类型,可以通过调用class方法或者isa指针查看具体类型,最终都是继承自NSBlock类型</br>
* `__NSGlobalBlock__`(`_NSConcreteGlobalBlock`)
* `__NSStackBlock__`(`_NSConcreteStackBlock`)
* `__NSMallocBlock__`(`_NSConcreteMallocBlock`)

>block的类型一切以运行时的结果为准</br>
同时通过clang转成的代码(C++)并不是真正的底层代码</br>
clang是属于LLVM中的一部分 

![](Snip20180716_12.png)

>堆:动态分配内存,需要程序员申请,也需要程序员自己管理内存

![](Snip20180716_15.png)

* 只要没有访问auto变量就是global类型,哪怕有访问static变量或者局部变量
* 在栈上的block由于处于栈区,受作用域影响,在作用域之外,有可能被销毁,导致数据错乱
* globalBlock调用copy方法,依然为global类型

### Block的copy操作</br>
![](Snip20180716_17.png)

在ARC环境下,编译器会根据情况自动将栈上的block复制到堆上,比如以下情况:</br>
* block作为函数的返回值
* 将block赋值给`__strong`指针时
* block作为cocoaAPI中方法含有usingBlock的方法参数时
* block作为GCD方法的参数时

MRC环境下block属性建议写法:使用copy关键字</br>
>MRC环境下若使用retain,只会令block的引用计数+1,并不会将block复制到栈区</br>

ARC环境下,copy和strong都是可以的,因为在ARC环境下,对block强引用也会对block进行一次copy操作,但还是建议统一写copy

### Block的访问外部对象类型auto变量</br>
* 栈空间上的block,是不会持有外部对象的
* 堆空间上的block,则会持有外部对象,当block使用到外部变量时,会对外部变量进行一次retain操作,在block自己销毁时,也会对内部用到的外部变量做一次release操作

当block内部访问了对象类型的auto变量时:
 
* 如果block是在栈上:</br>
无论block内部对对象类型的auto变量是强引用还是弱引用,都不会持有该对象,即不会对auto变量产生强引用

* 如果block被拷贝到堆上:</br>
会调用block内部的Desc中的copy函数</br>
copy函数内部会调用`_Block_object_assign`函数</br>
`_Block_object_assign`函数会根据auto变量的修饰符(`__strong`,`__weak`,`__unsafe_unretained`)做出相应的操作,类似于retain(形成强引用、弱引用)

* 当block从堆上移除:</br>
会调用block内部的Desc中的dispose函数</br>
dispose函数内部会调用`_Block_object_dispose`函数</br>
`_Block_object_dispose`函数会自动释放引用的auto变量,类似于release

>copy函数和dispose函数都是做内存管理用的,只要见到这两个函数,就大概知道与对象有关

![](Snip20180717_19.png)

当访问的是个对象类型对象,会自动生成`_Block_object_assign`函数和`_Block_object_dispose`函数对该对象进行内存管理

看强引用什么时候被释放,则该对象就什么时候释放,与弱引用无关

### Block的修改外部auto变量</br>
默认情况下,在block中是无法修改外部的变量的</br>
从本质上看,定义变量是在一个函数,而block调用的函数又是另一个函数,二者作用域不同,故肯定是无法修改(代码)

要想修改外部变量,做法:</br>
* 将变量变为static或者为全局变量
* `__block`关键字

`__block`可以用于解决block内部无法修改auto变量值的问题</br>
`__block`不能修饰全局变量,静态变量(static)

编译器会将`__block`包装成一个对象,故其也有copy与dispose函数</br>

```objc
//包装的对象结构体为:
struct __Block_byref_age_0 {
  void *__isa;
  ___Block_byref_age_0 *__forwarding;//该指针指向自己
  int __flags;
  int __size;
  int age;
}
```

在block中定义了一个指针,指向上面的这个结构体,当需要修改值时,通过指针找到对应的`__forwarding`,在找到对应的age,从而能够成功地修改和读取值了

此时在外边打印age的地址值,实际上就是`__block_byref_age_0`内部的age的地址,即实际上外部的age就是结构体内部的age

>注意点1:若block外部定义了一个可变数组array,在block内部使用方法[array addObject:@“111”],是可以的,因为其本质是将array用来使用,而不是直接修改array的指针的东西,例如array = nil,(该语句就会报错 )</br>
>注意点2:若对象类型外部在使用`__block`修饰,则可以修改对象类型的指针变量,即可以令array = nil,其内部也会生成一个`__Block_byref_XXX`的结构体,结构体内部还会存在一个copy函数和dispose函数(因为block外部需要管理结构体对象,故需要一个copy和dispose,而结构体内部需要管理真正的对象类型,故还是需要一个copy和dispose函数)

### `__block`的内存管理</br>
* 当block在栈上时,并不会对__block变量产生强引用
* 当block被copy到堆上的时候,会调用block内部的copy函数,copy函数内部会调用`_Block_object_assign`函数,`_Block_object_assign`函数会对`__block变`量形成强引用(retain)(一般都是强引用)

![](Snip20180717_22.png)

当block从栈copy到堆上,会将内部用到的`__block`变量也一起拷贝到堆上,并对其进行强引用,当时`__block`修饰的对象类型,在`__block`的变量结构体被拷贝到堆上时,会调用结构体内部的copy函数对结构体里边的对象变量进行管理
若另外一个block进行同样的操作,也使用到一样的`__block`变量,则该block拷贝到堆上的同时,对该变量进行强引用,由于之前`__block`已经在堆上了,故直接强引用就好了,不需要再次复制

* 当block从堆中移除时,会调用block内部的dispose函数,dispose函数内部会调用`_Block_object_dispose`函数,`_Block_object_dispose`函数会自动释放引用的`__block变量`(release)

![](Snip20180717_23.png)

#### block与对象类型的auto对象变量,`__block`变量内存管理总结</br>
* 当block在栈上时,对他们都不会产生强引用
* 当block拷贝到堆上时,都会通过copy函数来处理它们
* 当block从堆中移除时,都会调用dispose函数来释放

使用`__block`修饰基本数据类型和直接传入OC对象的不同是:</br>
使用`__block`修饰的对象,在block内部都是对其有着强引用的</br>
使用OC对象传入会根据外部是强指针(`__strong`)还是弱引用(`__weak`)来决定内部对该对象是强引用还是弱引用

>注:不存在`__block __weak int age`这样的写法,因为`__weak`只能用来修饰普通OC对象

#### `__forwarding`指针</br>
* 当`__block`修饰的变量在栈上时,其`__forwarding`指针指向自己
* 当`__block`修饰的变量被拷贝到堆上时,栈上的对象中的`__forwarding`指针指向堆上的对象,堆上对象的`__forwarding`指针指向自己

![](Snip20180717_25.png)

这样的好处是,无论访问堆上还是栈上的`__block`对象,都能确保读取的对象一定是堆中的`__block`对象

#### 被`__block`修饰的对象类型</br>
* block内部指向被`__block`包装的对象结构体的指针一定是强指针,而结构体内部的指向外部对象的指针会根据外部是强指针还是弱指针,对应的是强引用还是弱引用
* 当栈上的block调用了copy操作,会调用block中Desc函数内部的copy函数将修饰的变量中的结构体也复制一份到堆上
* 在MRC环境中,使用`__block`修饰的对象变量中结构体指向外部对象变量的指针始终都是弱引用(即不会进行retain操作),只有`__block`对象才会这样.比较特殊

### 循环引用</br>
解决方法:</br>
* ARC:</br>
1.使用`__weak`或`__unsafe_unretained `指针</br>
`__weak`:不会产生强引用,当指向的对象被销毁,会自动将指针置为nil</br>
`__unsafe_unretained` :不会产生强引用,不安全,指向的对象销毁时,指针存储的地址值不变(有可能产生野指针错误)</br>
2.使用`__block`,在block内部将`__block`修饰的指针置为空,且必须要执行block

```objc
__block MXPerson *person = [[MXPerson alloc] init];
person.age = 10;
person.block = ^{
  NSLog(@"%d",person.age);
  person = nil;
}

person.block();
```

* MRC</br>
1.因为MRC是不支持`__weak`的,所以在MRC环境中,可以使用`__unsafe_unretained`</br>
2.可以使用__block
