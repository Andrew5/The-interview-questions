# The interview questions

#### iOS必问基础问题

###### 1: 讲讲你对atomic & nonatomic的理解

答：nonatomic：非原子操作，决定编译器生成的setter getter是否是原子操作,不会为setter方法加锁。系统自动生成的   getter/setter   方法不一样。如果自己写   getter/setter  ，那   atomic/nonatomic/retain/assign/copy   这些关键字只起提示作用，写不写都一样。线程不安全  ,如有两个线程访问同一个属性，会出现无法预料的结果.
atomic  和  nonatomic  用来决定编译器生成的  getter  和  setter  是否为原子操作。在多线程环境下，原子操作是必要的，否则有可能引起错误的结果。
 对于  atomic  的属性，  atomic  表示多线程安全，  atomic  意为操作是原子的，系统生成的   getter/setter   会保证   get  、  set   操作的完整性，不受其他线程影响。比如，线程   A   的   getter   方法运行到一半，线程   B   调用了   setter  ：那么线程   A   的   getter   还是能得到一个完好无损的对象。意味着只有一个线程访问实例变量  (  生成的  setter  和  getter  方法是一个原子操作  )  而  nonatomic  就没有这个保证了。所以，  nonatomic  的速度要比  atomic  快  ;  不过  atomic  可并不能保证线程安全。如果线程   A   调了   getter  ，与此同时线程   B   、线程   C   都调了   setter——  那最后线程   A get   到的值，  3  种都有可能：可能是   B  、  C set   之前原始的值，也可能是   B set   的值，也可能是   C set   的值。同时，最终这个属性的值，可能是   B set   的值，也有可能是   C set   的值  . 
atomic  是线程安全的  ,  需要消耗大量的资源，至少在当前的存取器上是安全的。它是一个默认的特性，但是很少使用，因为比较影响效率  ,  加了  atomic  ，  setter  函数会变成下面这样：
if (property != newValue) {
[property release];
property = [newValue retain];
}
假设有一个   atomic   的属性   "name"  ，如果线程   A   调  [self setName:@"A"]  ，线程   B   调  [self setName:@"B"]  ，线程   C   调  [self name]  ，那么所有这些不同线程上的操作都将依次顺序执行  ——  也就是说，如果一个线程正在执行   getter/setter  ，其他线程就得等待。因此，属性   name   是读  /  写安全的。 
但是，如果有另一个线程   D   同时在调  [name release]  ，那可能就会  crash  ，因为   release   不受   getter/setter   操作的限制。也就是说，这个属性只能说是读  /  写安全的，但并不是线程安全的，因为别的线程还能进行读写之外的其他操作。线程安全需要开发者自己来保证。
如果   name   属性是   nonatomic   的，那么上面例子里的所有线程   A  、  B  、  C  、  D   都可以同时执行，可能导致无法预料的结果。如果是   atomic   的，那么   A  、  B  、  C   会串行，而   D   还是并行的。 
//nonatomic如下解释：
//@property(nonatomic, retain) UITextField *userName;
//系统生成的代码如下：
\- (UITextField *) userName {
return userName;
}
\- (void) setUserName:(UITextField *)userName {
 [userName retain];
 [userName release];
 userName = userName;
}
//atomic如下解释：
//@property(retain) UITextField *userName;
//系统生成的代码如下：
\- (UITextField *) userName {
  UITextField *retval = nil;*
  @synchronized(self) {
  retval = [[userName retain] autorelease];
 }
 return retval;
}
\- (void) setUserName:(UITextField *)userName {
  @synchronized(self) { 
   [userName release];
   userName = [userName_ retain];
 }
}
// atomic 会加一个锁来保障线程安全，并且引用计数会 +1，来向调用者保证这个对象会一直存在。假如不这样做，如有另一个线程调 setter，可能会出现线程问题，导致引用计数降到0，原来那个对象就会被释放掉；
 @synthesize  的语义：如果没有手动实现  setter  和  getter  方法，编译器会自动添加这两个方法。【强调合成】 
@dynamic  的语义：告知编译器  ,  属性的  setter  与  getter  方法由自己实现，不需要自动生成。【对于  readonly  的属性只需要提供  geter  】  ;  当没有用  @dynamic  修饰属性的时候，编译器默认是实现了  getter  和  setter  方法。还顺便插入了实例变量 

###### 2: 被 weak 修饰的对象在被释放的时候会发生什么？是如何实现的？知道sideTable 么？里面的结构可以画出来么？

答：weak表示指向但不拥有该对象。其修饰的对象引用计数不会增加。无需手动设置，该对象会自行在内存中销毁。 __weak(assign) 修饰表明一种关系“非拥有关系”。弱引用，不决定对象的存亡。即使一个对象被持有无数个弱引用，只要没有强引用指向它，那么还是会被销毁；在一个对象被释放后，weak会自动将指针指向nil，而assign则不会，向nil发送消息时不会导致崩溃的，所以assign就会导致野指针的错误unrecognized selector sent to instance。 若附有weak 修饰符的变量所引用的对象被废弃，则将nil赋值给该变量。 假设变量obj附加strong修饰符且对象被赋值。 { // 声明一个weak指针 id weak obj1 = obj; } 模拟编译器编译后的代码： id obj1;objc_initWeak(&obj1, obj); objc_release(obj); objc_destroyWeak(&obj1); 通过objc_initWeak 函数初始化附有weak修饰符的变量： /* 编译器的模拟代码 / id obj1; obj1 = 0; objc_storeWeak(&obj1, obj); objc_storeWeak函数把第二参数的赋值对象的地址作为键值，将第一参数的附有weak修饰符的变量的地址注册到weak表中。如果第二参数为0，则把变量的地址从weak表中删除。 weak 表与引用计数表相同，作为散列表被实现。如果使用weak表，将废弃对象的地址作为键值进行检索，就能高速地获取对应的附有weak修饰符的变量的地址。另外，由于一个对象可同时赋值给多个附有weak修饰符的变量中，所以对于一个键值，可注册多个变量的地址。 在变量作用域结束时通过 objc_destroyWeak函数释放该变量： / 编译器的模拟代码 / objc_storeWeak(&obj1, 0); 释放对象时，废弃谁都不持有的对象的同时，程序的动作是怎样的呢？下面我们来跟踪观察。对象将通过objc_release函数释放。 （1）objc_release （2）因为引用计数为0所以执行dealloc （3）objc_rootDealloc （4）object_dispose （5）objc_destructInstance （6）objc_clear_deallocating 对象被废弃时最后调用的objc_clear_deallocating函数的动作如下： （1）从weak表中获取废弃对象的地址为键值的记录。 （2）将包含在记录中的所有附有weak修饰符变量的地址，赋值为nil。 （3）从weak表中删除该记录。 （4）从引用计数表中删除废弃对象的地址为键值的记录。 根据以上步骤，前面说的如果附有weak修饰符的变量所引用的对象被废弃，则将nil赋值给该变量这一功能即被实现。由此可知，如果大量使用附有weak修饰符的变量，则会消耗相应的CPU资源。良策是只在需要避免循环引用时使用weak修饰符。 以上就是一个 weak 指针从初始化到被置为nil 的全过程，在写这篇文章之前我一直有疑惑，如果是objc_clear_deallocating函数进行了weak指针置为nil的操作，那objc_destroyWeak函数是干嘛的？我反复推敲，想起来文中早已说明了用途 “在变量作用域结束时通过 objc_destroyWeak函数释放该变量”，也就是说objc_destroyWeak函数是在weak指针被置为nil后，用来将weak释放掉。 weak立即释放对象 使用weak修饰符时，以下源代码会引起编译器警告。 { id weak obj = [[NSObject alloc] init]; } 因为该源代码将自己生成并持有的对象赋值给附有weak修饰符的变量中，所以自己不能持有该对象，这时会被释放并被废弃，因此会引起编译器警告：warning: Assigning retained object to weak variable; object will be released after assignment 编译器如何处理该源代码呢? /编译器的模拟代码/ id obj； id tmp = objc_msgSend(NSObject, @selector(alloc)); objc_msgSend(tmp, @selector(init)); objc_initweak(&obj, tmp); objc_destroyWeak(&object)； 虽然自己生成并持有的对象通过objc_initWeak函数被赋值给附有weak修饰符的变量中，但编译器判断其没有持有者，故该对象立即通过objc_release函数被释放和废弃。 这样一来，nil就会被赋值给引用废弃对象的附有weak修饰符的变量中。下面我们通过NSLog函数来验证一下: id weak obj= [[NSObject alloc] init]; NSLog(@"obj=%@"，obj); 以下为该源代码的输出结果，其中用%@输出nil。 obj=（null） 如上所述，以下源代码会引起编译器警告。 id weak obj= [[NSObject alloc] init]; 这是由于编译器判断生成并持有的对象不能继续持有。附有unsafe_unretained修饰符的变量又如何呢? id unsafe_unretained obj=[[NSObject alloc] init]; 与weak修饰符完全相同，编译器判断生成并持有的对象不能继续持有，从而发出警告： Assigning retained object to unsafe_unretained variable; object will be released after assignment 该源代码通过编译器转换为以下形式。 /编译器的模拟代码/ id obj = objc_msgSend( NSObject, @selector(alloc))； objc_msgSend(obj,@selector(init))； objc_release(obj); objc_release函数立即释放了生成并持有的对象，这样该对象的悬垂指针被赋值给变量obj中。 那么如果最初不赋值变量又会如何呢？下面的源代码在MRC时必定会发生内存泄漏。 [[NSObject alloc] init]； 由于源代码不使用返回值的对象，所以编译器发出警告。 warning：expression result unused [-Wunused-value] [[NSObject alloc] init]； 可像下面这样通过向void型转换来避免发生警告。 （void)[[NSObject alloc] init]； 不管是否转换为void，该源代码都会转换为以下形式 / 编译器的模拟代码 */ id tmp = objc_msgSend( NSObject, @selector(alloc)); objc_msgSend(tmp, @selector(init))； objc_release(tmp)； 在调用了生成并持有对象的实例方法后，该对象被释放。看来“由编译器进行内存管理”这句话应该是正确的。 Runtime维护了一个weak表，用于存储指向某个对象的所有weak指针。weak表其实是一个hash（哈希）表，key是所指对象的地址，value是weak指针的地址（这个地址的值是所指对象的地址）数组。为什么value是数组？因为一个对象可能被多个弱引用指针指向。 weak 的实现原理可以概括一下三步：1、初始化时：runtime会调用objc_initWeak函数，初始化一个新的weak指针指向对象的地址。2、添加引用时：objc_initWeak函数会调用objc_storeWeak() 函数，objc_storeWeak() 的作用是更新指针指向，创建对应的弱引用表。3、释放时，调用clearDeallocating函数。clearDeallocating函数首先根据对象地址获取所有weak指针地址的数组，然后遍历这个数组把其中的数据设为nil，最后把这个entry从weak表中删除，最后清理对象的记录。浅谈iOS之weak底层实现原理1、Runtime会维护一个Weak表,用于维护指向对象的所有weak指针。Weak表是一个哈希表,其key为所指对象的指针,vaue为Weak指针的地址数组。具体过程如下1、初始化时: runtime会调用 objc_initWeak函数初始化一个新的weak指针指向对象的地址。2、添加引用时: objc_initWeak函数会调用objc storeWeak(0函数,更新指针指向,创建对应的弱引用表。3、释放时,调用 clearDeallocating函数clearDeallocating函数首先根据对象地址获取所有Weak指针地址的数组,然后遍历这个数组把其中的数据设为n,最后把这个enty从Weak表中删除,最后清理对象的记录。当weak引用指向的对象被释放时，又是如何去处理weak指针的呢？当释放对象时，其基本流程如下：1、调用objc_release2、因为对象的引用计数为0，所以执行dealloc3、在dealloc中，调用了objc_rootDealloc函数4、在objc_rootDealloc中，调用了object_dispose函数5、调用objc_destructInstance6、最后调用objc_clear_deallocating,详细过程如下：a. 从weak表中获取废弃对象的地址为键值的记录b. 将包含在记录中的所有附有 weak修饰符变量的地址，赋值为 nilc. 将weak表中该记录删除d. 从引用计数表中删除废弃对象的地址为键值的记录

###### 3: block 用什么修饰？strong 可以？

答：block的本质是oc对象，可以通过获取class得知最终继承自NSObiect。block分为全局block保存在数据区，栈block保存在栈区，堆block保存在堆区。
		block如果捕获了自动变量就是栈block，但是在arc环境下系统会对栈block进行copy操作，拷贝到堆区。在mrc下就是栈block，需要手动调用 copy操作拷贝到栈区。block要想修改变量就需要使用block。用block修饰的变量会被包装成对象。编译成c++文件后可以查看。在编译后的结构体中会持有被修饰的变量。如果是强类型结构体中也是强类型，如果是弱类型结构体中也是弱类型。而且结构体会有两个函数指针copy和dispose。block编译后如果捕获后的变量需要block来管理内存，那么 block的desc成员中也会有copy和dispose指针。**在修改被block修饰的方法中会通过forwarding指针去取，forwarding指针指向自己栈上的block如果被拷贝到堆上，栈上的forwarding会指向堆上的结构体。这样保障能够访问到正确的值**。然后内存管理方面主要是copy函数和 dispose函数。循环引用的解决手段就是打破强引用改为弱引用。(备注：这个很多地方有讲，block的本质和底层可以看objective-c高级编程这本书很详细)

###### 4: block 为什么能够捕获外界变量？ block  做了什么事？ 

答："block内部也有个isa指针。block是封装了函数调用以及函数调用环境的OC对象。block内部有isa指针，以及包含指向方法实现的地址，对于局部自动变量，是值传递，对于局部静态变量，是指针传递，全局变量直接访问，__block可以用于解决block内部无法修改auto变量值的问题，编译器会将__block变量包装成一个对象。"

###### 5: 谈谈你对事件的传递链和响应链的理解

答：1.用户触摸屏幕，系统硬件进程会获取到这个点击事件，将事件简单处理封装后存到系统中，由于硬件检测进程和当前App进程是两个进程，所以进程两者之间传递事件用的是端口通信。硬件检测进程会将事件放到APP检测的端口。2.APP启动主线程RunLoop会注册一个端口事件，来检测触摸事件的发生。当事件到达，系统会唤起当前APP主线程的RunLoop。来源就是App主线程事件，主线程会分析这个事件。3.最后，系统判断该次触摸是否导致了一个新的事件, 也就是说是否是第一个手指开始触碰，如果是，系统会先从响应网中 寻找响应链。如果不是，说明该事件是当前正在进行中的事件产生的一个Touch message， 也就是说已经有保存好的响应链.大致的过程 initial view –> super view –> ……–> view controller –> window –> Application这里呢 UIResponder是所有可以响应事件的类的基类，其中包括最常见的UIView和UIViewController甚至是UIApplication，通过Hit-Test过程找到这个initial view，当用户点击了手机屏幕时，UIApplication就会调用UIWindow的hitTest: withEvent:方法。这个方法最终返回一个UiView。也就是我们要找到的那个最前的view响应链中是有 controller 的，因为 controller 继承自 UIResponder。UIApplication –> UIWindow –>递归找到最合适处理的控件 –> 控件调用 touches 方法 –> 判断是否实现 touches 方法 –> 没有实现默认会将事件传递给上一个响应者 –> 找到上一个响应者 –> 找不到方法作废
		苹果注册了一个 Source1 (基于 mach port 的) 用来接收系统事件，其回调函数为 __IOHIDEventSystemClientQueueCallback()。	当一个硬件事件(触摸/锁屏/摇晃等)发生后，首先由 IOKit.framework 生成一个 IOHIDEvent 事件并由 SpringBoard 接收。这个过程的详细情况可以参考这里。SpringBoard 只接收按键(锁屏/静音等)，触摸，加速，接近传感器等几种 Event，随后用 mach port 转发给需要的App进程。随后苹果注册的那个 Source1 就会触发回调，并调用 _UIApplicationHandleEventQueue() 进行应用内部的分发。	UIApplicationHandleEventQueue() 会把 IOHIDEvent 处理并包装成 UIEvent 进行处理或分发，其中包括识别 UIGesture/处理屏幕旋转/发送给 UIWindow 等。通常事件比如 UIButton 点击、touchesBegin/Move/End/Cancel 事件都是在这个回调中完成的。	手势识别	当上面的 _UIApplicationHandleEventQueue() 识别了一个手势时，其首先会调用 Cancel 将当前的 touchesBegin/Move/End 系列回调打断。随后系统将对应的 UIGestureRecognizer 标记为待处理。	苹果注册了一个 Observer 监测 BeforeWaiting (Loop即将进入休眠) 事件，这个Observer的回调函数是 _UIGestureRecognizerUpdateObserver()，其内部会获取所有刚被标记为待处理的 GestureRecognizer，并执行GestureRecognizer的回调。	当有 UIGestureRecognizer 的变化(创建/销毁/状态改变)时，这个回调都会进行相应处理。

###### 6: 谈谈 KVC 以及 KVO 的理解

答：就是为对象添加一个观察者“Observer”，当其属性值发生改变时，就会调用"observeValueForKeyPath:"方法，为我们提供一个“对象值改变了！”的时机进行一些操作。Key-Value Obersver，即键值观察。它是观察者模式的一种衍生。基本思想是，对目标对象的某属性添加观察，当该属性发生变化时，会自动的通知观察者。这里所谓的通知是触发观察者对象实现的KVO的接口方法。 
		KVO是解决model和view同步的好法子。另外，KVO的优点是当被观察的属性值改变时是会自动发送通知的，这比通知中心需要post通知来说，简单了许多。
		KVO:当指定的对象的属性被修改了，允许对象接收到通知的机制。利用RuntimeAPI动态生成一个子类，并且让instance对象的isa指向这个全新的子类，当修改instance对象的属性时，会调用Foundation的，_NSSetXXXValueAndNotify函数，此函数的内部实现为 调用willChangeValueForKey，调用父类(原来)的setter实现，调用didChangeValueForKey:，didChangeValueForKey:内部会调用observer的observeValueForKeyPath:ofObject:change:context:方法。Apple使用了isa混写（isa-swizzling）来实现KVO。当观察对象A时，KVO机制动态创建一个新的名为NSKVONotifying_A的新类，该类集成自对象A的本类，且KVO为NSKVONotifying_A重写观察属性的setter方法，setter方法会负责在调用元setter方法之前和之后，通知所有观察对象属性值的更改情况。（备注：isa混写（isa-swizzling）isa:is a kind of ; swizzling: 混合，搅合）
		1、NSKVONotifying_A类剖析：在这个过程，被观察对象的isa指针从指向原来的A类，被KVO机制修改为指向系统创建的自NSKVONotifying_A类，来实现当前类属性值改变的监听；所以当我们从应用层面来看，完全没有意识到有新的类出现，这是系统“隐瞒”了对KVO的底层想实现过程，让我们误以为还是原来的类。但是此时如果我们创建一个新的名为“NSKVONotifying_A”的类，就会发现系统运行到注册KVO的那段代码时程序就崩溃，因为系统在注册监听的时候动态创建了名为NSKVONotifying_A的中间类，并指向这个中间类了（isa指针的作用：每个对象都有isa指针，指向该对象的类，他告诉Runtime系统这个对象的类是什么。所以对象注册为观察者时，isa指针指向新子类，那么这个被观察的对象就神奇地变成新子类的对象（或实例）了。）因而在该对象上对setter的调用就会调用已重写的setter，从而激活键值通知机制。
		2、子类setter方法剖析：KVO的键值观察通知依赖与NSObject的两个方法：willChangeValueForKey:和didChangeValueForKey:，
在存取数值的前后分别调用2个方法：被观察属性发生改变之前，willChangeValueForkey:被调用，通知系统该keyPath的属性值即将变更；
		当改变发生后，didChangeValueForkey:被调用，通知系统该keyPath的属性值已经变更；
之后，observeValueForKey:ofObject:context:也会被调用。且重写观察属性的setter方法这种继承方式的注入是在运行时而不是编译时实现的。
		KVO方法列表中包含：setValueclass-deallocisKVOA-self.p isa addobserve 指向NSKVONotifying_A 动态生成这个子类self.p isa removeobserve 指向A 类临时帮我们实现KVO 默默付出 依靠setValue方法 成员变量并没有set方法 watchpoint set variable self->p>name 系统set方法内部调用willChangeValueForkey、didChangeValueForkey实现KVO监听成员变量不会执行set方法didChangeValueForkey 调用nitifyforkey了
		KVC的实现原理
		答：俗称“键值编码”，通过一个key来访问某个属性；一种间接访问对象属性的机制，甚至可以通过KVC来访问对象的私有属性！修改textField的placeholder也是通过KVC修改的，KVC对多种数据类型的支持，KVC在某种程度上提供了替代存取方法（访问器方法）的方案，不过存取方法终究是个好东西，以至于只要有可能，KVC也尽可能先尝试使用存取方法访问属性。
当使用KVC访问属性时，它内部其实做了很多事：
		1.首先查找有无<property>，set<property>，is<property>等property属性对应的存取方法，若有，则直接使用这些方法;
		2.若无，则继续查找<property>，get<property>，set<property>等方法，若有就使用；
		3.若查询不到以上任何存取方法，则尝试直接访问实例变量<property>， 
		4.若连该成员变量也访问不到，则会在下面方法中抛出异常。之所以提供这两个方法，就是让你在因访问不到该属性而程序即将崩掉前，供你重写，在内做些处理，防止程序直接崩掉。
		5.利用KVC即键值编码来给对象的私有属性赋值。6.如何手动触发KVO，valueForUndefinedKey:和setValue:forUndefinedKey:方法。

###### 7: RunLoop 的作用是什么？它的内部工作机制了解么？

答：

###### 8: 苹果是如何实现 autoreleasepool的？为什么这么设计？

答：由若干个AutoreleasePoolPage以双向链表的形式组合而成，对应数据结构中的parent指针和child指针,并且autorelease还存着对象的地址和内部成员变量地址；AutoreleasePool是按线程一一对应的；
之所以用双链表设计，是因为每张链表都是表头尾相连接，每创建page，会在会在首部创建一个哨兵对象作为标记，最外层page的顶端会有一个next指针，next指针初始值指向该page中可以存储对象地址的开始位置，当满的时候，就会在链表的顶端指向下一张表；
删除对象地址也就是释放某一个对象时会根据链表去查找链表所在的节点位置，这样删除节点，链表依然是连续的，说一下链表删除核心思想，其主要是找到指定位置节点的前一个节点，找到指定位置节点的下一个节点，将指定位置节点的前一个节点指向指定位置节点的下一个节点，删除指定位置节点。虽然这么做增加删除节点复杂了点但是提高了检索速度，双向查找节点值的时候方便，可进可退。

备注：通知使用单链表数据结构：没有增容问题,插入一个开辟一个空间，用链表不用数组，利于增删改查，增删。而且 查找方面，数组基于角标查找才是 O(1)，数组在内存中是按照顺序存储的，每个元素出现顺序与我们可见的index，也就是索引一一对应。这就意味着，只要知道其中一个元素地址，便能在O(1)时间内随机访问任意一个元素。基于元素查找 跟 链表没区别。数组扩容问题，其实关系不大，只是没必要，如果数组有一方面优于链表，肯定选择数组，因为数组对cpu友好。实际上，你实在找不到在 通知这一方面  数组能比链表好。而这种能力以链表为代表的非顺序存储模型是无法具备的。我们都知道链表某个节点引用了前驱或后继节点，因此它只能一次移动一步，像蜗牛似的爬到想要访问的节点。如果已知节点和待查找的节点中间间隔N个元素，它就得移动N步。综上，数组能一步到位，链表N步到位，结论：数组随机访问能力强于链表。


###### 9: 谈谈你对 FRP (函数响应式) 的理解，延伸一下 RxSwift 或者 RAC ?

答：

###### 10:property的作用是什么，有哪些关键词，分别是什么含义？

答：

###### 11: 父类的property是如何查找的？

答：

###### 12: NSArray、NSDictionary应该如何选关键词？

答：

###### 13: copy和muteCopy有什么区别，深复制和浅复制是什么意思，如何实现深复制？

答：

###### 14: 用runtime做过什么事情？runtime中的方法交换是如何实现的？

答：

###### 15: 讲一下对KVC合KVO的了解，KVC是否会调用setter方法？

答：

###### 16: __block有什么作用

答：在block内修改某局部变量需加block, MRC 环境下block在使用过程中不会对原来值进行copy，可以直接修改该变量 ，ARC环境下会对原值进行copy，内存地址也发生变化;
block可以直接修改 全局和静态变量 ，不会copy该变量的值.
不加__block, MRC 和 ARC block中都是对（原来指针的copy），也就是有两个不同的指针，指向同一个对象。_
_使用_block ,MRC环境block中不会对原来的指针进行copy，所以可以更改属性，也可以更改对象本身 ；而ARC环境则是对原对象的copy，内存地址也发生变化
block所起到的作用就是只要观察到该变量被 block 所持有,就将“外部变量”在栈中的内存地址放到了堆中。进而在block内部也可以修改外部变量的值,

###### 17: 说一下对GCD的了解，它有那些方法，分别是做什么用的？

答：

###### 18、ARC和MRC的区别，iOS是如何管理引用计数的，什么情况下引用计数加1什么情况引用计数减一？

答：

###### 19、在MRC下执行[object autorelease]会发生什么，autorelease是如何实现的？

答：

###### 20、CoreAnimation是如何绘制图像的，动画过程中的frame能否获取到？

答：

###### 21、谈一下对Runlop的了解？

答：

###### 22、OC如何实现多继承？（其实借助于消息转发，protocol和类别都可以间接实现多继承。）

答：

###### 23、对设计模式有什么了解，讲一下其中一种是如何使用的。

答：

###### 24、有没有哪个开源库让你用的很舒服，讲一下让你舒服的地方。

答：

###### 25、pod install 与 pod update的区别？

答：每一次运行pod install命令后，都会去下载安装新的库，并且会修改Podfile.lock文件中记录的库的版本。Podfile.lock文件是用来追踪和锁定这些库的版本的。
 运行pod install后，它仅仅只能解决Podfile.lock中没有列出来的依赖关系。
 在Podfile.lock中列出的那些库，也仅仅只是去下载Podfile.lock中指定的版本，并不会去检查最新的版本。
 没有在Podfile.lock中列出的那些库，会去检索Podfile中指定的版本，比如pod ‘myPod’, ‘~>1.2’
 当你运行了 pod update PODNAME命令，CocoaPods将不会考虑Podfile.lock中列出的版本，而直接去查找该库的新版本。它将更新到这个库尽可能新的版本，只要符合Podfile中的版本限制要求。
 如果使用pod update命令不带库名称参数，CocoaPods将会去更新Podfile中每一个库的尽可能新的版本。
 使用pod update仅仅只是去更新指定库的版本（或者全部库）。
 提交你的Podfile.lock文件:
 提醒一下，即使你一向不**commit**你的库文件到你的共享仓库，你也应该总是**commit & push**到你的Podfile.lcok文件中。
 否则，就会破坏掉pod install的整个设计逻辑，造成Podfile.lock文件无法锁定你已经安装的库。

###### 26、NSNotification原理，如何处理对象释放问题 ？

答：[通知原理解析](https://www.jianshu.com/p/e93b81fd3aa9)，[通知底层探究](https://blog.csdn.net/streamery/article/details/115963752)
		NSNotificationCenter 内部根据每个通知名称name都引用着一个数组（这个数组应该是弱引用数组NSHashTable），数组里存着每个注册者的target-action，当发通知时（[[NSNotificationCenter defaultCenter] postNotificationName:@"target" object:nil];），会根据NotificationName找到对应的数组，然后便利数组里的target-action，达到批量分发的功能。
		对于Observation持有observer：持有的是weak类型指针，当observer释放时observer会置nil，nil对象performSelector不再会崩溃。		name和Observation是映射关系，observer和sel包含在Observation结构体中，NSNotification维护了全局对象表NCTable结构，结构体里包含GSIMapTable表的结构，用于存储Observation。

###### 27、多个线程，操作同一个变量，会有什么后果？

答：
@property (nonatomic, strong) NSString *bookName;
Data *p1 = [[Data alloc]init];
dispatch_queue_t queue = dispatch_queue_create("temp", DISPATCH_QUEUE_CONCURRENT);
for(int i = 1;i<10000;i++)
{
dispatch_async(queue, ^{
[p1 setBookName:[NSString stringWithFormat:@"dddddd:%d",i]];
 });
}
崩溃->  0x10961b928 <+8>:  testb  $0x4, 0x20(%rax)
bookName的声明中指定属性nonatomic，表示为非线程安全的，set方法没有上锁。多线程同时进入到set方法进行操作时会引发问题，只要改换为atomic即可避免该问题,那为什么使用nonatomic就不行？多线程同时进入set方法执行时为什么会出错？原因也很简单，因为在ARC下编译器自动生成的set方法中，需要先对之前持有的对象进行release释放操作，如果此时没有加锁，就有可能发生多个线程执行同一段代码，导致release语句被执行了多次，从而导致对同一块内存多次发送release消息，导致的对已经回收的内存进行重复回收，即会报错。
如果修改为：就不会报错
[p1 setBookName:[NSString stringWithFormat:@"1234 %d",i]];
苹果当中为了节省内存和提高执行效率，提出的一种Tagged Pointer的概念，在8个字节之内能存放的范围之内，系统就会以Tagged Pointer的方式生成指针，如果 8 字节承载不了就会用普通的方式来生成普通指针。这个TaggedPointer与普通指针不同，普通指针是一个变量，存放在栈上，其内容是一块内存块的基地址，所以说指针指向一个内存块，但TaggedPointer并不是一个真正的对象，实际上他的内容就是真正的值，而不是内存地址，所以他本质上就是一个普通变量，也不持有某个内存块。所以在set方法中，对这样的并没有持有内存块的普通变量进行赋值是不需要提前release的，是完全没有问题的，所以不会报错。		

28、view生命周期
		答：view的创建：loadView    视图控制器(UIViewController)及其子类,无论是手写代码还是storyboard、xib肯定会调用loadView方法。其它的视图不会调用比如UIButton，UILabel等，因为他们不是视图控制器。下面是视图控制器被创建时会被调用的其它方法：    Storyboard/XIB会调用的方法:    	◦	initWithCoder    	◦	awakeFromNib：此时frame还没有完成。    手写代码调用的代码(必须是UIView比如自定义MDDButton : UIButton)    	◦	initWithCoder    initWithFrame，创建时init会被调用此方法(可以继承UIView，做下测试)，不过frame为0.除非显示调用此方法，frame才会有值，比如：[[MDDButton alloc] initWithFrame:CGRectMake(10, 10, 100, 40)];这样显示的调用frame不为0。    view采用懒加载的方式，只有用到view时才会被创建，即才会被调用 loadView ——>viewDidLoad这一系列函数        iOS6及以后，内存警告时系统会回收ViewController的View的CALayer里的BitMap（CABackingStore类型，它的内容是直接用于渲染到屏幕，它是View消耗内存的大户）。view和calayer占的内存极少， 数量级也就在byte和kbyte之间，所以系统只回收了BitMap，但是这里所谓的回收只是给BitMap占用的内存打了一个volatile标记表明这部分内存是可能随时被其它数据占用,平时没内存警告时正在使用的内存标记为In use，完全被释放回收的标记为Not in use。概括起来也就是说：iOS6及以后的内存警告时，系统会给用于渲染视图的数据(BitMap)内存打一个volatile， ViewController的View的架子结构并不会回收，当View再次被访问时，虽然View的架子结构会用重建，但触发drawRect来渲染界面时，如果view对应的BitMap数据内存没有被占用则会被View的drawRect方法直接渲染出来且内存被标记为in use，从而这块内存又可以独享了；如果已被其它数据占用，那么BitMap必须要重建。所以可以看到整个重建过程不再是由loadView来做的，它是通过对view的访问来触发的。但是，请注意， 如果说在iOS6及以后ViewController的loadView方法只会被调用一次，这种说法是不完全准确的。因为：如果在didReceiveMemoryWarning里把ViewController的View也回收了([self.view removeFromSuperview];self.view = nil;)，那么当再次有对View访问时，loadView会被调用以进行完全最彻底的重建(想想也是，ViewController的View都没了，不调loadView来重建那怎么办呢)。    viewDidLoad    加载到内存完成后会调用此函数，在视图切换中，只要控制器不从内存中移除此方法就不会被调用。一般在此方法中添加一些子控件，设置视图的初始属性等等，类似初始化。    viewWillAppear    即将加载到窗口时调用此方法。一般在此方法做一些较为耗时的。这样就可以先显示基本的视图，呈现给用户(让用户感觉不是那么卡)，然后再显示比较耗时的。以免显示一个白屏给用户。    viewDidAppear    视图已经加载到窗口时调用。    
		◦	viewWillDisappear－视图即将消失、被覆盖或是隐藏时调用；    	
		◦	viewDidDisappear－视图已经消失、被覆盖或是隐藏时调用；    	
		◦	viewVillUnload－当内存过低时，需要释放一些不需要使用的视图时，即将释放时调用；    	
		◦	viewDidUnload－当内存过低，释放一些不需要的视图时调用。    
布局    我们能看到手机上的视图都是UIView还有它的子UIView，当然不能杂乱无章的显示。要进行布局，父UIView需要布局、排列这些子UIView。UIView提供了layoutSubviews方法来处理。    需要注意的是layoutSubviews方法由系统来调用，不能程序员来手动调用。可以调用setNeedsLayout方法进行标记，以保证在UI下个刷屏循环中系统会调用layoutSubviews。或者调用layoutIfNeeded直接请求系统调用layoutSubviews。    layoutSubviews的被调用的时机：    	
		◦	addSubview会触发layoutSubviews，比如viewA add viewB，第一次添加A和B的layoutSubviews都会被调用，而第二次(viewA已经有了viewB)只调用viewB的    	
		◦	view的Frame变化会触发layoutSubviews    	
		◦	滚动一个UIScrollView会触发layoutSubviews    	
		◦	旋转Screen会触发父UIView上的layoutSubviews    	
		◦	改变transform属性时，当然frame也会变    	
		◦	处于key window的UIView才会调用(程序同一时间只有一个window为keyWindow，可以简单理解为显示在最前面的window为keywindow)    最后总结一句话就是，有必要时才会调用，比如设置Frame值没有变化，是不会被调用的，很明显没有必要

###### 线程锁

答：循环锁NSRecursiveLock，条件锁NSConditionLock，分布式锁NSDistributedLock

#### Runtime相关问题

###### 1: 什么是 isa，isa 的作用是什么？

答：

###### 2: 一个实例对象的isa 指向什么？类对象指向什么？元类isa 指向什么？

答：

###### 3: objc中类方法和实例方法有什么本质区别和联系？

答：

###### 4: load 和 initialize 的区别？

答：load是main函数之前调用的，initialize是类第一次接收到消息的时候调用的，每一个类只会initialize一次(如果子类没有实现+initialize，会调用父类的+initialize（所以父类的+initialize可能会被调用多次）,如果分类实现了+initialize，就覆盖类本身的+initialize调用)

###### 5: _objc_msgForward 函数是做什么的？直接调用会发生什么问题？

答：

###### 6: 简述下 Objective-C 中调用方法的过程

答：

###### 7: 能否想象编译后得到的类中增加实例变量？能否向运行时创建的类中添加实例变量？为什么？

答：

###### 8: 谈谈你对切面编程的理解

答：

###### 9: 消息转发机制实现过程？

答：当调用一个NSObject对象不存在的方法时，并不会马上抛出异常，而是会经过多层转发，层层调用对象的-resolveInstanceMethod: , -forwardingTargetForSelector: , -methodSignatureForSelector: ,-forwardInvocation:等方法，其中最后-forwardInvocation：是会有一个NSInvocation对象，这个NSInvocation对象保存了这个方法调用的所有信息，包括Selector名，参数和返回值类型，最重要的是有所有参数值，可以从这个NSInvocation对象里拿到调用的所有参数值。若想令类能理解某条消息，我们必须实现对应的方法才行，但是在编译期向类发送了其无法理解的消息并不会报错，因为在运行期可以继续向类中添加方法，所以编译器在编译时还不确定类中到底会不会有某个方法的实现。当对象接收到无法解读的消息后，就会启动“消息转发”机制，程序可由此过程告诉对象应该如何处理未知消息。消息转发分为两大阶段，第一阶段先征询接收者所属的类，看其是否能动态添加方法，以处理当前这个“未知的选择子”，这叫做“动态方法解析”。如果运行期系统已经把第一阶段执行完了，那么接收者自己就无法再以动态新增方法的手段来响应包含该选择的消息了。此时运行期系统会请求接收者以其他手段来处理与消息相关的方法调用。这又细分为两小步。首先请接收者看看有没有其他对象可以处理这条消息，若有，则运行期会把消息转给那个对象，于是消息转发过程结束，一切如常。若没有“备援的接收者”，则启动完整的消息转发机制，运行期会把与消息有关的全部细节都封装到NSInvocation对象中，再给接收者最后一次机会，令其设法解决当前还未处理的这条消息。1.动态方法解析对象在无法解读消息会首先调用所属类的下列类方法：+ (BOOL) resolveInstanceMethod:(SEL)selector参数为那个未知的选择子，返回值表示这个类能否新增一个实例方法处理此选择子。假如尚未实现的方法不是实例方法而是类方法则运行期会调用另一个方法：+ (BOOL) resolveClassMethod:(SEL)selector。使用这种方法的前提是：相关方法的实现代码已经写好，只等着运行的时候动态插入到类里面就可以了。此方案常用来实现@dynamic属性。2.备援接收者当前接收者还有第二次机会能处理未知的选择子，这一步中，运行期会问它：能不能把这条消息转发给其他接收者来处理。与该步骤对应的处理方法：- (id) forwardingTargetForSelector:(SEL)selector方法参数代表未知的选择子，若当前接收者能找到备援对象，则将其返回，若找不到就返回nil。通过此方案我们可以用“组合”来模拟出“多重继承”的某些特性（因为OC属于单继承，一个字类只能继承一个基类）。在一个对象内部，可能还有一系列其他对象，该对象可能由此方法将能够处理某选择子的相关内部对象返回，这样的话，在外界看来，好像该对象亲自处理了这些消息。3.完整的消息转发如果转发算法已经到了这一步的话，那么唯一能做的就是启用完整的消息转发机制了。首先创建NSInvocation对象，把与尚未处理的那条消息有关的全部细节都封装与其中。此对象包含选择子、目标（target）、参数。在出发NSInvocation对象时“消息派发系统”将亲自出马，把消息指派给目标对象。- (void) forwardInvocation:(NSInvocation *)invocation这个方法可以实现很简单：只需改变调用目标，使消息在新目标上得以调用即可，然而这样实现出来的方法与“备援接收者”方案所实现的方法等效，所以很少有人采用这么简单的实现方式。比较有用的实现方式为：在出发消息前，现已某种方式改变消息内容，比如追加另外一个参数，或者改换选择子等等。实现了此方法若发现某调用操作不应由本类处理，则需调用超类的同名方法。这样的话继承体系中的每个类都有机会处理此调用请求，直至NSObject。如果最后调用了NSObject类的方法，那么该方法还会继续调用“doesNotRecognizeSelector:”以抛出异常，此异常表明选择子最终未能得道处理。

###### 10: 调用runtime registerClass方法，创建出来了几个类？	

答：实例对象、类对象、runtime，	成员变量保存在class_rw_t->class_ro_t 实例对象 本质 objc结构题就是一个对象ISA从objc_object继承过来的     class_ro_t存储了当前类在编译期就已经确定的属性、方法以及遵循的协议，里面是没有分类的方法的。那些运行时添加的方法将会存储在运行时生成的class_rw_t中。    // 可读可写    struct class_rw_t {      // Be warned that Symbolication knows the layout of this structure.      uint32_t flags;      uint32_t version;      const class_ro_t *ro; // 指向只读的结构体,存放类初始信息      /*      这三个都是二位数组，是可读可写的，包含了类的初始内容、分类的内容。      methods中，存储 method_list_t ----> method_t      二维数组，method_list_t --> method_t      这三个二位数组中的数据有一部分是从class_ro_t中合并过来的。      */      method_array_t methods; // 方法列表（类对象存放对象方法，元类对象存放类方法）      property_array_t properties; // 属性列表      protocol_array_t protocols; //协议列表          Class firstSubclass;      Class nextSiblingClass;      //...     }        rw什么时候创建的 ：运行时创建的 copy进来 可读写的 属性 方法 协议 脏内存 数据昂贵 一直存在    ro什么时候创建的 ：编译的时候创建的 只读状态 成员变量 可移除 保持清洁的数据 永远不可改变     ISA 实例对象 类对象 指向元类 根元类 指向自己    SEL 方法编号    IMP 函数指针 保存了方法的地址    Method 参数类型描述字符串    设计元类的原类 (复用消息通道，类方法也可以放在Class里，但发送消息时，需要增加一个参数) 是实例 类对象方法    发送消息 消息发送 消息转发 runtime 是查找的元类 类方法    https://www.jianshu.com/p/da200b79a6af     

###### 11: Objc为什么要设计消息发送机制，直接调函数有什么缺点？

答：消息发送机制对于直接调函数
 主要是多了一个消息动态解析以及消息转发
 消息动态解析可以在运行时动态的去添加实现调用，更加灵活
 消息转发，则是更加灵活的去更改方法的调用对象，以及方法的执行。
 总的来说我感觉消息发送机制是运行时的基础。更加灵活方便吧

#### 网络&多线程问题

###### 1: HTTP的缺陷是什么？

答：

###### 2: 谈谈三次握手，四次挥手！为什么是三次握手，四次挥手？

答：

###### 3: socket 连接和 Http 连接的区别

答：HTTP协议：简单对象访问协议，对应于应用层 ，HTTP协议是基于TCP连接的
tcp协议： 对应于传输层
ip协议： 对应于网络层
TCP/IP是传输层协议，主要解决数据如何在网络中传输；而HTTP是应用层协议，主要解决如何包装数据。
Socket是对TCP/IP协议的封装，Socket本身并不是协议，而是一个调用接口（API），通过Socket，才能使用TCP/IP协议。
http连接：http连接就是所谓的短连接，即客户端向服务器端发送一次请求，服务器端响应后连接即会断掉；
socket连接：socket连接就是所谓的长连接，理论上客户端和服务器端一旦建立起连接将不会主动断掉；但是由于各种环境因素可能会是连接断开，比如说：服务器端或客户端主机down了，网络故障，或者两者之间长时间没有数据传输，网络防火墙可能会断开该连接以释放网络资源。
http连接就是所谓的短连接，即客户端向服务器端发送一次请求，服务器端响应后连接即会断掉。
**HTTP 这个应用层协议是以 TCP 为基础来传输数据的**。当你想访问一个资源（资源在网络中就是一个 URL）时，你需要先解析这个资源的 IP 地址和端口号，从而和这个 IP 和端口号所在的服务器建立 TCP 连接，然后 HTTP 客户端发起服务请求（GET）报文，服务器对服务器的请求报文做出响应，等到不需要交换报文时，客户端会关闭连接

###### 4: HTTPS，安全层除了SSL还有，最新的？参数握手时首先客户端要发什么额外参数

答：

###### 5: 什么时候POP网络，有了 Alamofire 封装网络 URLSession为什么还要用Moya ？

答：

###### 6: 如何实现 dispatch_once

答：

###### 7: 能否写一个读写锁？谈谈具体的分析

答：这个场景用于我之前写的IM单聊的时候用到，实现方式是在写入数据的时候其实有很多读取的操作，读写很有可能是同时进行的，我主要控制读写互斥，首先进行读取锁加锁，锁住修改数据源的过程，在加锁的情况下进行写入锁的变更；当数据读取完后，就对写入锁解锁，解锁后写入操作就可以正常操作了，实现读与写的互斥，并行读 串行写的逻辑。

###### 8: 什么时候会出现死锁？如何避免？

答：

###### 9: 有哪几种锁？各自的原理？它们之间的区别是什么？

答：

###### 10、网络请求的缓存处理，NSCache原理，LRU MRU 缓存原理，SKU SPU组合算法探究

答：

###### 11、post和get区别,应用层协议里的 GET 和 POST 有啥区别

答：get方法，应用于 等幂操作 的请求；会被缓存且是主动行为；可以收藏书签；可以保留历史记录，有缓存；获取数据；get的querystring（仅支持urlencode编码）；querystring 是url的一部分get、post都可以带上；get请求长度最多1024kb；请求参数会被完整保留在浏览器历史记录里；
post方法，应用于 非等幂操作 的请求；POST 向指定资源提交数据进行处理请求（例如提交表单或者上传文件）。数据被包含在请求体中。POST请求可能会导致新的资源的建立和/或已有资源的修改；提交表单、数据等，没有历史记录且安全；querystring 是url的一部分get、post都可以带上；post的参数是放在body（支持多种编码）；post对请求数据没有限制；post在浏览器中返回的时候，会有再次提交请求的提示；POST中的参数不会被保留；

###### 12、如何切换到指定线程？

答： performSelector:onThread:withObject:waitUntilDone: 

###### 13、为什么说子线程不能更新UI呢？

​	答：UI操作涉及到渲染访问视图的各种对象属性,异步操作会存在各种读写问题,虽可以通过加锁解决,但是这种另辟蹊径的方式会耗费大量的资源导致运行速度耗损；程序加载完景象就会进入main函数，此时UIApplication初始化主线程进行用户的事件，例如点击 手势操作等传递在主线程进行；所有的用户事件只能在主线程进行响应，渲染则需要30帧左右同步在屏幕上等待Sync垂直信号到来进行更新；非主线程异步操作的情况下不能保证实现同步更新机制。   
 		两个异步线程同时做某些操作 可能发生不被预期的情况：    
​	 	同时设置一个视图的背景图片很有可能因为当前图片被释放了两次导致应用程序的crash;   
 		同时设置一个视图的背景颜色很有可能渲染树显示的是颜色属性A，而此时的视图的逻辑树储存的是颜色属性B；   
 		同时操作一个视图的属性结构比如线程A循环便利所有的子视图，而此时线程B中将某个子视图直接删除，这样不仅会导致A的错乱		还可能导致应用程序的崩溃；    
​		将大部分视图绘制方法属性改为了线程安全，但仍强烈建议将UI操作放在主线程，原因是是属性图存在读写，不可能全部改写为线程安全。    
​		UI更新一定要在主线程实现的原因：    
​		系统默认是没有加锁的，多个线程被允许同时访问更新同一个UI控件，也就是所谓的Ui访问时候系统当中的UI控件都不是安全的，在多线程中变得不可控；    
​		规定只能在主线程访问UI，相当于是一种UI访问加的锁；    
​		多线程更多的是算力，而非不断的进行UI的更新，保证处理UI的事件都在串行队列中进行。

#### 数据结构问题

###### 1: 数据结构的存储一般常用的有几种？各有什么特点？

答：

###### 2: 集合结构 线性结构 树形结构 图形结构

答：

###### 3: 单向链表 双向链表 循环链表 

答：

###### 4: 数组和链表区别 

答：

###### 5: 堆、栈和队列

答：

###### 6: 输入一棵二叉树的根结点，求该树的深度？

答：

###### 7: 输入一棵二叉树的根结点，判断该树是不是平衡二叉树？

答：

#### 算法问题

###### 1: 时间复杂度

答：

###### 2: 空间复杂度

答：

###### 3: 常用的排序算法

答：

###### 4: 字符串反转

答：

###### 5: 链表反转（头差法）

答：

###### 6: 有序数组合并

答：

###### 7: 查找第一个只出现一次的字符（Hash查找）

答：

###### 8: 查找两个子视图的共同父视图

答：

###### 9: 无序数组中的中位数(快排思想)

答：

###### 10: 给定一个整数数组和一个目标值，找出数组中和为目标值的两个数

答：

#### 架构设计问题

###### 1: 设计模式是为了解决什么问题的？

答：1). MVC模式：Model View Control，把模型 视图 控制器 层进行解耦合编写。
		2). MVVM模式：Model View ViewModel 把模型 视图 业务逻辑 层进行解耦和编写。
		3). 单例模式：通过static关键词，声明全局变量。在整个进程运行期间只会被赋值一次。
		4). 观察者模式：KVO是典型的观察者模式，观察某个属性的状态，状态发生变化时通知观察者。
		5). 委托模式：代理+协议的组合。实现1对1的反向传值操作。
		6). 工厂模式：通过一个类方法，批量的根据已有模板生产对象。

###### 2: 看过哪些第三方框架的源码，它们是怎么设计的？

答：

###### 3: 可以说几个重构的技巧么？你觉得重构适合什么时候来做？

答：

###### 4: 开发中常用架构设计模式你怎么选型?

答：

###### 5: 怎么区分组件化和模块化？怎么区分框架与架构？你是如何组件化解耦的？

答：
 1、组件多是半成品（甚至可以是代码片段）强调的是重用。一般作为框架的最底层。独立性强以供模块使用。
 2、模块多是指定业务代码进行的封装。可独立运行，以界面或者功能粒度进行划分。一般作为框架的业务层。通过接口等手段通信。目的是降低业务耦合。
 3、举例：一个登录的UI界面呈现出来的可以是一个登录模块，而登录模块有按钮、输入框等组件构成（也可以是一个）。
 4、它俩一般是一起呈现，一个叫做组件\模块。多个就成了组件\模块化
 1、架构是一种抽象，框架是一种具体的实现。架构（动词）是框架（名词）的指导。
 2、框架是指以具体的软件实现某种或多种特定功能需求（强调先通用在专用，项目里边都会对第三方框架做二次封装）。架构是思想（强调先大局在局部）。
 3、框架是抽象的解决方案（关注大局忽略细节的实现，因为还强调通用性所以多是半成品）
 4、两者都是为解决软件开发越来越复杂而采取的策略手段。

#### 性能优化问题

###### 1: tableView 有什么好的性能优化方案？

答：

###### 2: 界面卡顿和检测你都是怎么处理？

答：

###### 3: 谈谈你对离屏渲染的理解？

答：

###### 4: 如何降低APP包的大小

答：

###### 5: 日常如何检查内存泄露？

答：

###### 6: APP启动时间应从哪些方面优化？

答：
 1.准备编译 （选择build system, building targets）
 2.Build target 
 1.创建了几个文件夹，写入辅助文件：将项目的文件结构对应表、将要执行的脚本、项目依赖库的文件结构对应表写成文件,写入Entitlements.plist文件 ，方便后面使用；并且创建一个 .app 包，后面编译后的文件都会被放入包中； 
 2.运行预设脚本： Build Phases / Cocoapods 会预设一些脚本。
 3.编译类文件：针对每一个文件进行编译，生成可执行文件 Mach-O，这过程 LLVM 的完整流程，前端、优化器、后端；
 4.链接文件：将项目中的多个可执行文件合并成一个文件；
 5.拷贝资源文件：将项目中的资源文件拷贝到目标包；
 6.编译 Asset 文件：我们的图片如果使用 Assets.xcassets 来管理图片，那么这些图片将会被编译成机器码，除了 icon 和 launchImage；
 7.编译XIB文件， storyboard 文件：storyboard 文件也是会被编译的；
 8.链接 storyboard 文件：将编译后的 storyboard 文件链接成一个文件；
 9.生成DSYM文件
 10.运行 Cocoapods 脚本：将在编译项目之前已经编译好的依赖库和相关资源拷贝到包中。(如果有bugly脚本 还会执行)
 11.生成 .app 包
 12.将 Swift 标准库拷贝到包中（如果有swift）
 13.对包进行签名
 14.完成打包
 LLVM 编译过程
 	▪	预处理
 	▪	词法分析
 	▪	语法分析
 	▪	生成 AST
 	▪	静态分析
 	▪	生成 LLVM IR
 	▪	编译器优化
 	▪	Bitcode （可选）
 	▪	生成汇编
 	▪	生成目标文件
 	▪	生成可执行文件
 void * _Nullable NSMapGet(NSMapTable * _Nonnull, const void * _Nullable): map table argument is NULL**
 runtime时系统创建了多少个全局对象 数据结构是什么 作用是什么
 面试题：
 1.用什么集合，几个集合区别是什么
 2.java类型有什么，我说是数据类型， 八大数据类型 他还是问，string是什么类型， 我说是引用类型，接着问 string 和stringbuffer stringbuilding区别是什么
 3.消息队列
 4.mq
 5.sql优化 
 6.为什么tableView需要数据源来实现协议方法而不是直接把数据通过属性传给tableView？
 面试题A：
 1.重写和重载分别是什么，区别是什么
 2.hashMap的底层远离
 3.创建线程的几个方法是什么，区别是什么
 4.linux命令中用什么打开文件， 有几种
 5.vi命令中用什么搜索
 6.linux中用什么查询
 面试题B
 1.java多线程，
 https://www.runoob.com/java/java-multithreading.html
 2.事务管理，
 https://blog.csdn.net/zhaohong_bo/article/details/90443545 
 3.Zookeeper 
 https://www.cnblogs.com/areyouready/p/10014947.html 
 4.Redis，
 https://www.runoob.com/redis/redis-intro.html 
 5.java集合类的用法，如:HashMap的用法，
 https://baijiahao.baidu.com/s?id=1635677223045847111&wfr=spider&for=pc 
 6.数据库分区分表， 
 https://blog.csdn.net/qq_28289405/article/details/80576614 
 7.索引优化 ， 
 https://blog.csdn.net/yangxingpa/article/details/81477607 
 8.单点登录的问题 
 https://www.cnblogs.com/ywlaker/p/6113927.html 
 面试题C
 springcloud的五大组件是什么
 面试题d
 1.消息队列
 2.redius
 3.集合有哪些，hashmap,hashset 区别
 4.springboot 注解 ，核心注解是什么，核心注解是哪几个组合
 5.Arreylist linkedlist 区别
 6.spring,springbuilder,springbuffer 区别
 7.字符串的方法(忘了具体怎么问，)就是回答。append 等，
 8.用过哪些中间价
 9.sql优化，数据库优化做过那
 10.分页
 11.分库分表
 12.ioc,aop是什么
 15:30
 面试题E：
 1.抽象类和接口的区别是什么
 2.异常都有几种，都是怎么处理的
 3.== 和equals什么区别

 面试题A：
 1.重写和重载分别是什么，区别是什么
 2.hashMap的底层远离
 3.创建线程的几个方法是什么，区别是什么
 4.linux命令中用什么打开文件， 有几种
 5.vi命令中用什么搜索
 6.linux中用什么查询
 面试题B
 1.java多线程，2.事务管理，
 3.zookeeper，
 4.Redis，
 5.java集合类的用法，如:hashmap的用法，
 6.数据库分区分表，
 7.索引优化 ，
 8.单点登录的问题
 面试题C
 springcloud的五大组件是什么
 面试题d
 1.消息队列
 2.redius
 3.集合有哪些，hashmap,hashset 区别
 4.springboot 注解 ，核心注解是什么，核心注解是哪几个组合
 5.Arreylist linkedlist 区别
 6.spring,springbuilder,springbuffer 区别
 7.字符串的方法(忘了具体怎么问，)就是回答。append 等，
 8.用过哪些中间价
 9.sql优化，数据库优化做过那
 10.分页
 11.分库分表
 12.ioc,aop是什么
 面试题E：
 1.抽象类和接口的区别是什么
 2.异常都有几种，都是怎么处理的
 3.== 和equals什么区别
 面试题F：
 1.SpringCloud具有以下特点：
  做到了良好的开箱即用，可扩展性机制覆盖
  包括服务发现和服务注册
  服务地址路由
  软负载均衡
  断路器
  分布式消息传递
 2.springboot的注解， 关于参数的都有哪个
 3.l
 面试题G:
 1.重载与重写的区别？
 2.string、stringbuffer、springbuilder的区别？
 3.说说你知道的集合（一般会问）
 4.Arraylist与linkedlist的区别？（问的较多）
 5.hashtable与hashmap的区别？（问的较多）
 面试题F：
 1.控制器view的加载过程
 2.应用程序的启动流程
 3.事件传递与响应的完整过程
 4.XIB文件的构成分为哪三个图标？都具有什么功能？
 什么是指针、指针地址
 什么是cpu：CPU是中央处理器，是电脑内超大规模的集成电路，主控电脑的运算核心与控制核心，可以处理与解释计算机软件的数据。一个成功的CPU不仅有助于降低内部整体计算的成本，还可以通过向外输出给客户更高性价比的选择。
 **runtime**
  1、OC的动态性就是由runtime来支撑和实现的，方法的调用 其实都是转化为 objc_msgSend函数的调用
 数据结构：objc_object，objc_class，isa，class_data_bits_t，cache_t，method_t对象，类对象，元类对象消息传递
 		 2、利用关联函数给分类添加属性
  		3、遍历类的成员变量以及方法
  		4、交换系统方法
 		 5、利用消息转发机制解决方法找不到的问题
  首先判断消息接收者是否为nil 如果为空直接退出
 1、去自己类的方法缓存列表中查找该方法如果找到则执行该方法，resolveInstanceMethod class
 2、否则去自己类的方法列表中查找该方法 如果找到先缓存该方法然后再执行，
 3、否则去自己父类的方法缓存列表中查找该方法 如果找到先缓存该方法到
 自己类中，然后执行该方法，
 4、否则去自己父类的方法列表中查找该方法如果找到先缓存该方法到自己类中，
 然后执行该方法，自己没有能力处理这个方法 将此方法转交给别人类处理 执行消息转发机制。
 1、首先调用 forwardingTargetForSelector:方法 
 在此方法中可以返回一个可以处理消息的对象，
 如果对象存在则直接给对象转发消息，
 如果对象不存在否则进入第2步 调用方法签名函数
 2、调用methodSignatureForSelector:方法（方法签名函数）在此方法中：
 如果返回值为空 则直接调用 调用doesNotRecognizeSelector:方法 
 然后程序报错：unrecognized selector sent to instance 经典错误
 如果返回值不为空 则直接进入第3步 调用 forwardInvocation方法
 3、forwardInvocation：方法
 开发人员可以在此方法中返回一个对象来转发此消息
 UIView为CALayer提供内容，以及负责处理触摸等事件，参与响应链
 CALayer负责显示内容contents
 **事件响应链**
 1、点击屏幕-> UIApplication -> UIWindow -> hittest -> pointInside -> subviews -> 遍历view -> subview hittest->hit != nil 执行事件
 **runloop**
 	 1、全局的Dictionary，key 是 pthread_t， value 是 CFRunLoopRef
 	2、线程和 RunLoop 之间是一一对应的，其关系是保存在一个全局的 Dictionary 里。线程刚创建时并没有 RunLoop，如果你不主动获取，那它一直都不会有。RunLoop 的创建是发生在第一次获取时，RunLoop 的销毁是发生在线程结束时
 	3、一个 RunLoop 包含若干个 Mode，每个 Mode 又包含若干个 Source/Timer/Observer
 	4、Source1：port通信，系统事件捕捉；source0只有一个回调函数 没有port
  5、主线程的RunLoop是默认开启的，所以程序在开启后，会一直运行，不会退出。其他线程的RunLoop如果需要开启，就手动开启
  6、 即将进入Loop 即将处理 Timer 即将处理 Source 即将进入休眠 刚从休眠中唤醒 即将退出Loop
 **weak**
 weak表示指向但不拥有该对象。其修饰的对象引用计数不会增加。无需手动设置，该对象会自行在内存中销毁weak表有两个参数，分别是赋值对象的地址作为键值 weak修饰符的变量的地址注册到weak表中，如果对象地址为0，就把变量的地址从weak表中删除weak 策略在属性所指的对象销毁时，系统会将 weak 修饰的属性对象的指针指 向 nil，在 OC 给 nil 发消息是不会有什么问题的; 如果使用 assign 在属性所指 的对象销毁时，属性对象指针还指向原来的对象，由于对象已经被销毁，这时候就产生了野指针，如果这时候在给此对象发送消息，很容造成程序奔溃 assigin 可以用于修饰非 OC 对象,而 weak 必须用于 OC 对象
 **进程**
 进程和线程的关系
 进程是操作系统的基本单元，也是一段程序的执行过程，是独立运行的
 1、线程是进程的执行单元，进程的所有任务都在线程中执行，一个进程要想执行任务,必须至少有一条线程.
 2.线程是 CPU 分配资源和调度的最小单位
 3.一个程序可以对应多个进程(多进程),一个进程中可有多个线程,但至少要有一条线程
 4.同一个进程内的线程共享进程资源
 **7****、****KVC**
 KVC全称（key-Value coding）称呼为键值编码，在iOS开发中。允许开发者通过key名直接访问对象的属性，或者给对象的属性赋值。需要调用明确的存取方法，这样就可以在运行时动态访问和修改对象的属性，而不是在编译时确定。
 首先查找有无<property>，set<property>，is<property>等property属性对应的存取方法，若有，则直接使用这些方法;
 2.若无，则继续查找*<property>*，get<property>，set<property>等方法，若有就使用；
 3.若查询不到以上任何存取方法，则尝试直接访问实例变量<property>， 
 4.若连该成员变量也访问不到，则会在下面方法中抛出异常。之所以提供这两个方法，就是让你在因访问不到该属性而程序即将崩掉前，供你重写，在内做些处理，防止程序直接崩掉。
 5.利用KVC即键值编码来给对象的私有属性赋值。
 **KVO** KVO是观察者模式的另一实现,使用了isa混写(isa-swizzling)来实现KVO,当其属性值发生改变时，
  利用RuntimeAPI动态生成一个子类NSKVONotifying_A，并且让instance对象的isa指向这个全新的子类
  调用willChangeValueForKey，调用父类(原来)的setter实现，调用didChangeValueForKey:内部会调用observer的observeValueForKeyPath:ofObject:change:context:方法。
 **AutoreleasePool**
 objc_autoreleasePoolPush：
 把当前next位置置为nil，即哨兵对象,然后next指针指向下一个可入栈位置，
 AutoreleasePool的多层嵌套，即每次objc_autoreleasePoolPush，实际上是不断地向栈中插入哨兵对象。
 objc_autoreleasePoolPop:
 根据传入的哨兵对象找到对应位置。
 给上次push操作之后添加的对象依次发送release消息。
 回退next指针到正确的位置。
 **10.****分类** 声明私有方法，分解体积大的类文件，把framework的私有方法公开，运行时决议，可以为系统类添加分类 。然后会递归调用所有类的 load 方法，可以添加实例方法，类方法，协议，属性（添加getter和setter方法，并没有实例变量，添加实例变量需要用关联对象）+load这一切都是在main函数之前执行的。
 4.如果工程里有两个分类A和B，两个分类中有一个同名的方法，哪个方法最终生效？
 取决于分类的编译顺序，最后编译的那个分类的同名方法最终生效，而之前的都会被覆盖掉(这里并不是真正的覆盖，因为其余方法仍然存在，只是访问不到，因为在动态添加类的方法的时候是倒序遍历方法列表的，而最后编译的分类的方法会放在方法列表前面，访问的时候就会先被访问到，同理如果声明了一个和原类方法同名的方法，也会覆盖掉原类的方法)。
 6.分类能添加成员变量吗？
 不能。只能通过关联对象(objc_setAssociatedObject)来模拟实现成员变量，但其实质是关联内容，所有对象的关联内容都放在同一个全局容器哈希表中:AssociationsHashMap,由AssociationsManager统一管理。
 **扩展** 声明私有属性，声明方法（没什么意义），声明私有成员变量 ,编译时决议，只能以声明的形式存在，多数情况下寄生在宿主类的.m中，不能为系统类添加扩展。

 **5\****、block**
 Block不允许修改外部变量的值，这里所说的外部变量的值，指的是栈中指针的内存地址。__block 所起到的作用就是只要观察到该变量被 block 所持有，就将“外部变量”在栈中的内存地址放到了堆中，这里拷贝的是指向该结构体对象的指针。这样就可以在block内部就可以修改外部变量的值。
 //对于auto age是捕获到内部，把外部age的值存起来，而对于static height，是把外部变量的指针保存起来,
  //对于static height值， block在访问static变量（局部变量）时，block内部会捕获到外部变量的地址值，后面修改外部static变量值的时候，（通过指针变量所指向的内存的值）地址访问到的是最新修改后的值。
 局部变量被编译成值形式，而静态变量被编成指针形式，全局变量并未截获
 而__block修饰的变量也是以指针形式
  //思考：为什么会出现这两种情况？原因很简单，因为auto是自动变量，出了作用域后会自动销毁的，如果我们不保留他的指针，就会存在访问野指针的情况，所以去要捕获他并把值保存起来。
  //局部变量需要跨函数访问，而且随时可能会被销毁所以需要提前捕获。
  //全局变量随时都可以访问，且一直存在，所以不需要捕获。
  //auto 值传递
  //static 指针传递
  //全局变量 直接访问
  //成员变量与属性在block中调用时，需要被捕获吗？
  //因为调用成员变量与属性时，需要通过self来调用，而self对block来说是局部变量，所以是需要捕获的。clang编译后
 __block_impl结构体为
 struct __block_impl {
 void *isa;//isa指针，所以说Block是对象
 int Flags;
 int Reserved;
 void *FuncPtr;//函数指针
 };

#### 备注：

###### 如何看待重构？

答：二次求导就变成一次求导函数，依赖注入、tableview、代理在安卓实现过程
如果重复三遍就写成工具
###### C 语言
int a,b;
a = 2;
b = 10;
bool d = a > b ? : b;
//a > b ? : b 等价于 a > b ? a > b : b;
NSLog(@"%d",d);//10
int x = 3;
int y = ++x ? ++x : 3;
NSLog(@"%d",y);//10

int xx = 0;
int yy = ++xx ? : 3;
NSLog(@"%d",yy);//10

###### xcode的编译流程做了啥？

答：https://www.jianshu.com/p/14612abdeb26          	https://objccn.io/issue-6-1/

###### 推荐一些 Code Review 工具

(https://www.zhihu.com/question/41089988/answer/544543949)
1、[Crucible](https://link.zhihu.com/?target=https%3A//www.atlassian.com/software/crucible)：Atlassian 内部代码审查工具；
2、[Gerrit](https://link.zhihu.com/?target=https%3A//www.gerritcodereview.com/)：Google 开源的 git 代码审查工具；
3、[GitHub](https://link.zhihu.com/?target=https%3A//github.com/)：程序员应该很熟悉了，上面的 "Pull Request" 在代码审查这里很好用；
4、[LGTM](https://link.zhihu.com/?target=https%3A//lgtm.com/)：可用于 GitHub 和 Bitbucket 的 PR 代码安全漏洞和代码质量审查辅助工具；
5、[Phabricator](https://link.zhihu.com/?target=https%3A//www.phacility.com/phabricator/)：Facebook 开源的 git/mercurial/svn 代码审查工具； 
6、[PullRequest](https://link.zhihu.com/?target=https%3A//www.pullrequest.com/)：GitHub pull requests 代码审查辅助工具；
7、[Pull Reminders](https://link.zhihu.com/?target=https%3A//pullreminders.com/)：GitHub 上有 PR 需要你审核，该插件自动通过 Slack 提醒你；
8、[Reviewable](https://link.zhihu.com/?target=https%3A//reviewable.io/)：基于 GitHub pull requests 的代码审查辅助工具；
9、[Sider](https://link.zhihu.com/?target=https%3A//sider.review/)：GitHub 自动代码审查辅助工具；
10、[Upsource](https://link.zhihu.com/?target=https%3A//www.jetbrains.com/upsource/)：JetBrain 内部部署的 git/mercurial/perforce/svn 代码审查工具。

###### 开发过程中遇到的最大的困难, 难点在哪里, 怎么解决的，

遇到一个崩溃， bugly也不显示具体位置，然后 通过符号表， 地址偏移，逆向工程，查到了一个sdk的问题， 然后 我通过代码注入的方式，帮他修复了bug。

###### flutter 事件循环

答：

###### flutter 是单线程还是多线程

答：单线程

###### Stream feature 的区别

答：

###### isolate有了解吗

答：

###### Stateless Widget和Stateful Widget区别

答：

###### StatefulWidget 的生命周期

答：

###### 介绍下widget、state、context

答：

###### Widget和element和RenderObject之间的关系

答：

###### flutter是如何实现多任务并行的，谈谈Isolate理解

答：
