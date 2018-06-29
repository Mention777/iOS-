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
