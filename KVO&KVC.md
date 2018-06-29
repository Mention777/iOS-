# KVO&KVC

**KVO**:键值监听,可以用于监听某个对象属性值的改变

```objc
//定义了一个MXPerson类
#import <Foundation/Foundation.h>

@interface MXPerson : NSObject

@property (nonatomic, assign) int age;

@end
```

```objc

- (void)viewDidLoad {
    [super viewDidLoad];
    
    self.person1 = [[MXPerson alloc] init];
    self.person1.age = 1;
    
    self.person2 = [[MXPerson alloc] init];
    self.person2.age = 2;
    
    [self.person1 addObserver:self forKeyPath:@"age" options:NSKeyValueObservingOptionNew|NSKeyValueObservingOptionOld context:NULL];
    
}

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    self.person1.age = 20;
    self.person2.age = 10;
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context {
    NSLog(@"%@",change);
}
```

以上是最简单的KVO的使用,通过打印person1和person2的isa指针可知</br>person1对象的isa指向**NSKVONotifying_MXPerson**类对象</br>person2对象的isa指向**MXPerson**对象

这里研究一下**NSKVONotifying_MXPerson**这个系统派生类,是由runtime在程序运行时动态创建一个类,是MXPerson的子类,该类中有**isa指针**、**superclass指针**、**set方法**、**class方法**、**dealloc方法**、**_isKVOA方法**</br>

此时,系统中的关系是:person实例对象的isa指针指向`NSKVONotifying_MXPerson`类,该类的superclass指针指向MXPerson类

当值发生改变时,person1会通过isa指针查询到类对象(`NSKVONotifying_MXPerson`),调用其内部的set方法,该set方法实际是调用foundation框架的**_NSSetIntValueAndNotify**方法

```objc
//_NSSetIntValueAndNotify方法内部伪代码

[self willChangeValueForKey:@"age"];
[super setAge:age];//重新调用父类setter进行赋值
[self didChangeValueForKey:@"age"];//该方法内部会通知监听器所监听的属性值已经发生了改变


- (void)didChangeValueForKey:(NSString *)key
{
    // 通知监听器，某某属性值发生了改变
    [oberser observeValueForKeyPath:key ofObject:self change:nil context:nil];
}
```

>注:`_NSSetIntValueAndNotify`中int为监听的属性类型,若为double,则类型为`_NSSetDoubleValueAndNotify`,以此类推

KVO使用原理本质就是动态生成了一个NSKVONotifying打头的类,使监听对象的isa指向该类,再做一些特殊的操作

```objc
    NSLog(@"person1添加KVO监听之前 - %@ %@",object_getClass(self.person1),object_getClass(self.person2));
    
    [self.person1 addObserver:self forKeyPath:@"age" options:NSKeyValueObservingOptionNew|NSKeyValueObservingOptionOld context:NULL];
    
    NSLog(@"person1添加KVO监听之后 - %@ %@",object_getClass(self.person1),object_getClass(self.person2));
    
    //由结果可知,添加完KVO监听,person1对象的类对象就已经发生了改变
```

>需要注意的是,`SKVONotifying_Person`这个类的isa是指向自己的原类对象,Person类是指向自己的原类对象

>此时:[person1 class]的结果为MXPerson</br>object_getClass(person1)的结果为NSKVONotifying_MXPerson</br>
>原因:object_getClass获取的是真正isa指向的类对象,而由于对象重写了class,有可能在内部重写的class方法,直接返回了原先的类对象,不然若调用NSObject的class方法,其内部就是通过返回object_getClass()的方式,故结果不一致,这么做的好处是屏蔽了内部实现,隐藏了类的存在

>注:C语言中指针可以当做数组来使用

### 几道面试题目</br>
Q1:iOS用什么方式实现对一个对象的KVO?(KVO的本质是什么)</br>
   * 利用runtimeAPI动态生成一个子类,并且让instance对象的isa指向这个全新的子类</br>
   * 当修改instance对象的属性时,会调用fuoundation的_NSSetXXXValueAndNotify函数(内部实现如上述)</br>
   
Q2:如何手动触发KVO?</br>
   * 手动调用`willChangeValueForKey:`和`didChangeValueForKey:`

解析:</br>
　KVO自动触发:即有人手动修改了某个属性,系统会自动触发调用对应函数通知监听者,调用监听方法</br>
　KVO手动触发:手动实现willchange和didchange方法,本来是didchange方法会触发监听者的监听方法,所以只需实现该方法即可,但需要实现willchange方法是因为didchange内部有判断willchange是否有执行
 
>KVO的本质是替换了原先的set方法,所以能不能监听,要看该属性有没有set方法,所以直接修改成员变量时不会触发KVO的

**KVC**:键值编码,可以通过一个key来访问某个属性

```objc
//KVC常用的API有:
- (void)setValue:(id)value forKey:(NSString *)key;
- (void)setValue:(id)value forKeyPath:(NSString *)keyPath;
- (id)valueForKey:(NSString *)key;
- (id)valueForKeyPath:(NSString *)keyPath;

//key和keyPath的区别:
//key只能访问当前对象的属性,keyPath支持一级一级的访问属性
```