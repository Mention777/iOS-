# OC对象的分类

OC对象可以分为3种:</br>
　1.instance对象(实例对象)</br>
　2.class对象(类对象)</br>
　3.meta-class对象(元类对象)

### instance对象</br>
* **定义**:就是通过类alloc出来的对象,每次调用alloc都会产生新的instance对象</br>
* **内存中存储的信息:**</br>
　　1.isa指针</br> 
　　2.其他成员变量</br>
>Tip.</br>
>实例对象的内存地址值与isa的地址是相同的

### Class对象</br>
* **定义**:一个类的类对象在内存中是唯一的,即每个类在内存中有且只有一个class对象</br>
* **内存中存储的信息:**</br>
　　1.isa指针</br>
　　2.superclass指针</br>
　　3.类的属性信息(@property),类的对象方法信息(instance method)</br>
　　4.类的协议信息(protocol),类的成员变量信息(ivar)</br>
>Tip.</br>
>1.此处的类的成员变量信息指的是,成员变量的描述信息(类型,变量名等) </br>
2.class的底层实现实际就是调用object_getClass方法

### Meta-Class对象</br>
* **定义**:每个类在内存中有且只有一个meta-class对象</br>
* **内存中存储的信息:**</br>
　　1.isa指针</br>
　　2.superclass指针</br>
　　3.类的类方法信息</br>

>Tip</br>
>1.获取原类对象方式:将类对象作为参数传入object_getClass方法中 ,即可获得原类对象</br>
2.注:class方法返回的一直是class对象,类对象,故[[NSObject class]class]无论调用多少次class方法,都为类对象</br>
3.meta-class对象和class对象的内存结构是一样的,但是用途不一样,即普通的class对象对应存储的信息(类的属性名等)为NULL,同理,对于类对象而言,类方法信息也为NULL</br>
4.调用class_isMetaClass可以判断是否为原类对象


