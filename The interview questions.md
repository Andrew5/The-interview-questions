# The interview questions

##### 1、多网络请求 

答：

##### 2、项目协议

答：

##### 3、RAC的zip 实现，swift的数组

答：

##### 4、网络请求的缓存处理，NSCache原理，LRU MRU 缓存原理，SKU SPU组合算法探究

答：

##### 5、pod install 与 pod update的区别

答：（*添加*、*更新*、*删除*）使用。

每一次运行`pod install`命令后，都会去下载安装新的库，并且会修改`Podfile.lock`文件中记录的库的版本。`Podfile.lock`文件是用来追踪和锁定这些库的版本的。

运行`pod install`后，它仅仅只能解决`Podfile.lock`中没有列出来的依赖关系。

在`Podfile.lock`中列出的那些库，也仅仅只是去下载`Podfile.lock`中指定的版本，并不会去检查最新的版本。

没有在`Podfile.lock`中列出的那些库，会去检索`Podfile`中指定的版本，比如`pod ‘myPod’, ‘~>1.2’

当你运行了 `pod update PODNAME`命令，**CocoaPods**将不会考虑`Podfile.lock`中列出的版本，而直接去查找该库的新版本。它将更新到这个库尽可能新的版本，只要符合`Podfile`中的版本限制要求。

如果使用`pod update` 命令不带库名称参数，**CocoaPods**将会去更新`Podfile`中每一个库的尽可能新的版本。

使用`pod update`仅仅只是去更新指定库的版本（或者全部库）。

提交你的Podfile.lock文件:

提醒一下，即使你一向不**commit**你的库文件到你的共享仓库，你也应该总是**commit & push**到你的`Podfile.lcok`文件中。

否则，就会破坏掉`pod install` 的整个设计逻辑，造成`Podfile.lock`文件无法锁定你已经安装的库。



##### 6、手写链表是否有环？

答：

##### 7、lldb 咋调试 iOS 真机的LLVM 

答：

##### 8、SSA 静态单赋值咋做的？

答：

##### 9、NSMutableArray copy ？

答：

##### 10、NSMutableArray 增删 ？

答：

##### 11、Block 内部修改临时变量及内存变化 ？

答：

##### 12、Category添加属性？分类和扩展的区别？为什么不能直接添加属性

答：分类（Category）是OC中的特有语法，它是表示一个指向分类的结构体的指针。原则上它只能增加方法，不能增加成员（实例）变量。具体原因看源码组成，Category 是表示一个指向分类的结构体的指针，其定义如下：

`**struct** objc_category {

  **char** * **_Nonnull** category_name              OBJC2_UNAVAILABLE;

  **char** * **_Nonnull** class_name                OBJC2_UNAVAILABLE;

  **struct** objc_method_list * **_Nullable** instance_methods   OBJC2_UNAVAILABLE;

  **struct** objc_method_list * **_Nullable** class_methods    OBJC2_UNAVAILABLE;

  **struct** objc_protocol_list * **_Nullable** protocols     OBJC2_UNAVAILABLE;

}  `

根本没有属性列表；

注意：
1.分类是用于给原有类添加方法的,因为分类的结构体指针中，没有属性列表，只有方法列表。所以< 原则上讲它只能添加方法, 不能添加属性(成员变量),实际上可以通过其它方式添加属性> ;
2.分类中的可以写@property, 但不会生成setter/getter方法, 也不会生成实现以及私有的成员变量（编译时会报警告）;
3.可以在分类中访问原有类中.h中的属性;
4.如果分类中有和原有类同名的方法, 会优先调用分类中的方法, 就是说会忽略原有类的方法。所以同名方法调用的优先级为 分类 > 本类 > 父类。因此在开发中尽量不要覆盖原有类;
5.如果多个分类中都有和原有类中同名的方法, 那么调用该方法的时候执行谁由编译器决定；编译器会执行最后一个参与编译的分类中的方法。

类别与类扩展的区别：
①类别中原则上只能增加方法（能添加属性的的原因只是通过runtime解决无setter/getter的问题而已）；
②类扩展不仅可以增加方法，还可以增加实例变量（或者属性），只是该实例变量默认是@private类型的（
用范围只能在自身类，而不是子类或其他地方）；
③类扩展中声明的方法没被实现，编译器会报警，但是类别中的方法没被实现编译器是不会有任何警告的。这是因为类扩展是在编译阶段被添加到类中，而类别是在运行时添加到类中。
④类扩展不能像类别那样拥有独立的实现部分（@implementation部分），也就是说，类扩展所声明的方法必须依托对应类的实现部分来实现。
⑤定义在 .m 文件中的类扩展方法为私有的，定义在 .h 文件（头文件）中的类扩展方法为公有的。类扩展是在 .m 文件中声明私有方法的非常好的方式。

##### 13、类是怎么被创建出来的？

答：

##### 14、NSMutableArray 对临时变量addObject 时，临时变量内存变化 failed

答：

##### 15、NSDictionary存储结构如何解决hash冲突 failed

答：

##### 16、NSNotification原理，如何处理对象释放问题 ？

答：

##### 17、Category为什么只能加方法不能加属性 ？

答：

##### 18、async mainThread 与 runloop 关系 ？

###### 答：

##### 19、sdwebimage图片缓存框架设计 ？

答：

##### 20、多个线程，操作同一个可变数组，会有什么后果？

答：

##### 21、MJ底层班说继承NSProxy的类本类的方法列表找不到imp，不会去父类查找，测试下来会去父类查找，请问一下Proxy的查找机制在哪里看到呢？

答：

##### 22、for in，是怎么实现的？

答：

##### 23、for in 跟for int i, i<10, i++的写法有什么不同？

答：

##### 24、FIFO FILO FRU原理？

答：

##### 25、编译原理和计算机组成原理

答：

##### 26、KVO的实现原理？

答：

##### 27、KVC的实现原理？

答：

##### 28、top level code

在App工程里， `.swift`文件都是编译成模块的，不能有`top level code`。

先明确一个概念，一个`.swift`文件执行是从它的第一条`非声明语句（`表达式、控制结构）开始的，同时包括声明中的赋值部分，所有这些语句，构成了该`.swift`文件的`top_level_code()`函数。而所有的声明，包括结构体、类、枚举及其方法，都不属于 `top_level_code()`代码部分，其中的代码逻辑，包含在其他区域，`top_level_code()`可以直接调用他们。程序的入口是隐含的一个 `main(argc, argv)`函数，该函数执行逻辑是设置全局变量`C_ARGC C_ARGV`，然后调用 top_level_code()。不是所有的 .swift 文件都可以作为模块，目前看，任何包含表达式语句和控制语句的 .swift 文件都不可以作为模块。正常情况下模块可以包含全局变量(var)、全局常量(let)、结构体(struct)、类(class)、枚举(enum)、协议(protocol)、扩展(extension)、函数(func)、以及全局属性(var { get set })。这里的全局，指的是定义在 top level 。这里说的表达式指`expression`，语句指 `statement` ，声明指`declaration` 。因此，如果代码中直接在类的外面调用类内部的方法，则该.swift 文件是编译不成的模块的，所以会编译报错。

##### 29、

答：





