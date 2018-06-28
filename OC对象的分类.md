# OC对象的分类

OC对象可以分为3种:</br>
1.instance对象(实例对象)</br>
2.class对象(类对象)</br>
3.meta-class对象(元类对象)

### instance对象</br>
* **定义**:就是通过类alloc出来的对象,每次调用alloc都会产生新的instance对象</br>
* **内存中存储的信息:**</br>
　　1)isa指针</br> 
　　2)其他成员变量</br>
>实例对象的内存地址值与isa的地址是相同的

### Class对象</br>
* **定义**:一个类的类对象在内存中是唯一的,即每个类在内存中有且只有一个class对象</br>
* **内存中存储的信息:**</br>
　　1)isa指针</br>
　　2)superclass指针</br>
　　3)类的属性信息(@property),类的对象方法信息(instance method)</br>
　　4.类的协议信息(protocol),类的成员变量信息(ivar)</br>
*  
