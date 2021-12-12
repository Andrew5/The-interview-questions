# The interview questions

##### 1、多网络请求 

答：

##### 2、项目协议

答：

##### 3、RAC的zip 实现，swift的数组

答：

```c
// 组合操作——zipWith、zip：按照信号执行时间顺序依次进行叠加，列出全部的元素。拉链式结构、元组一一对应。
// 信号一方先结束另一方也跟着结束。
// 注意信号A和信号B的执行顺序
RACSignal *signalA = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
  [subscriber sendNext:@1];
  [subscriber sendNext:@2];
  [subscriber sendNext:@4];
//    [subscriber sendError:[NSError new]];
  [subscriber sendCompleted];
  return [RACDisposable disposableWithBlock:^{
    NSLog(@"signalA完成");
  }];
}];

RACSignal *signalB = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
  [subscriber sendNext:@11];
  [subscriber sendNext:@22];
  // [subscriber sendError:[NSError new]];
  [subscriber sendNext:@44];
  [subscriber sendCompleted];
  return [RACDisposable disposableWithBlock:^{
    NSLog(@"signalB完成");
  }];
}];

//    RACSignal *signalC = [signalA zipWith:signalB];
//    RACSignal *signalC = [RACSignal zip:@[signalA, signalB]];
RACSignal *signalC = [RACSignal zip:RACTuplePack(signalA, signalB)];
[[signalC subscribeNext:^(id x) {
  NSLog(@"subscribeNext:%@",x);
}] dispose];
```

##### 4、网络请求的缓存处理，NSCache原理，LRU MRU 缓存原理，SKU SPU组合算法探究

答：

##### 5、pod install 与 pod update的区别

答：（*添加*、*更新*、*删除*）使用。
每一次运行pod install命令后，都会去下载安装新的库，并且会修改Podfile.lock文件中记录的库的版本。Podfile.lock文件是用来追踪和锁定这些库的版本的。
运行pod install后，它仅仅只能解决Podfile.lock中没有列出来的依赖关系。
在Podfile.lock中列出的那些库，也仅仅只是去下载Podfile.lock中指定的版本，并不会去检查最新的版本。
没有在Podfile.lock中列出的那些库，会去检索Podfile中指定的版本，比如pod ‘myPod’, ‘~>1.2’
当你运行了 pod update PODNAME命令，CocoaPods将不会考虑Podfile.lock中列出的版本，而直接去查找该库的新版本。它将更新到这个库尽可能新的版本，只要符合Podfile中的版本限制要求。
如果使用pod update命令不带库名称参数，CocoaPods将会去更新Podfile中每一个库的尽可能新的版本。
使用pod update仅仅只是去更新指定库的版本（或者全部库）。
提交你的Podfile.lock文件:
提醒一下，即使你一向不**commit**你的库文件到你的共享仓库，你也应该总是**commit & push**到你的Podfile.lcok文件中。
否则，就会破坏掉pod install的整个设计逻辑，造成Podfile.lock文件无法锁定你已经安装的库。

##### 6、手写链表是否有环？

答：

1. 如何判断链表是否存在环?
2. 如何知道环的长度?
3. 如何找出环的连接点在哪里?
4. 带环链表的长度?

解法:
1. 对于问题1
使用追赶的方法,设定两个指针show,fast,从头节点开始,每次分别前进1步,2步,如存在环则两者必然会相遇,如不存在环,则fast遇到null退出,并且碰撞点到头节点的距离为环中节点数n.
2. 对于问题2:
记录下问题1的碰撞点p,碰撞点p肯定是在环中的,从这个节点触发,一边移动一边计数再次回到这个节点就得到了环中节点数n
3. 对于问题3:
有定理: 碰撞点p到连接点的距离=头节点到连接点的距离,因此分别从碰撞点,头节点相同速度开始走相遇的那个点就是连接点.
定理证明如下:
设头节点到连接点的距离为x,连接点到碰撞点的距离为y,碰撞点到连接点的距离为z,环的长度为n,
(1) 碰撞点到头节点的距离为n,则有x+y=n
(2) 又因为环的长度为n,则有y+z=n
(3) 由(1),(2) 可得,碰撞点p到连接点的距离=头节点到连接点的距离
4. 对于问题4
问题3中已经求出连接点距离头指针的长度,加上问题2中求出的环的长度,二者之和就是带环链表的长度.

```python
# 有环单链表

class Node(object):
   def __init__(self, data, next=None):
      self.data = data
      self.next = next


class CircleLinklist(object):
   def __init__(self):
      self.head = None
      self.length = 0

   def insert(self, node):
      if self.length == 0:
         self.head = node
         self.length += 1
         return

      cur = self.head
      while cur.next:
         cur = cur.next

      cur.next = node
      self.length += 1

   def traverse(self):
      cur = self.head
      while cur is not None:
         print(cur.data)
         cur = cur.next

   def judge_circle_quick_slow_pointer(self):
      """
      判断单链表是否有环
      如果在某个时候,快指针追上了慢指针,那么便说明有环
      """
      # 快指针,每次走两个节点
      quick = self.head
      # 慢指针,每次走一个节点
      slow = self.head

      # 标志,判断是否有环
      flag = False
      # 每个指针的步数
      count = 0
      # 加上try-except，因为在没环的情况下，快指针会出错
      try:
         # 循环终止条件,如果某个节点的next为空,就说明没有环了
         while quick.next.next is not None and slow.next is not None:
            count += 1
            # 排除第一次相等的情况
            # 用后面那个条件做判断不是个好想法，万一环的起点就是head，然后节点个数又是偶数的话，不会停止了吧
            if quick.data == slow.data and count != 1:  # quick != self.head
               print('快指针追上慢指针时的节点的值: ',quick.data)
               flag = True
               break
            # 一次跨两个节点
            quick = quick.next.next
            # 一次跨一个节点
            slow = slow.next
      except Exception as e:
         print(e.args)
         print('quick条件报错，说明没有环')
      finally:
         print('步数: ', count)
         print('是否有环: ', flag)

   def judge_circle_by_traversing(self):
      """
      将每个遍历过的节点保存到集合中，如果下一次遍历的节点在集合中了，就说明有环
      这个方法很傻，但却可以得到环的点是哪个
      快慢指针只能判断是否有环，不能得到环的起点
      """
      nodes = []
      cur = self.head
      while cur is not None:
         if cur not in nodes:
            nodes.append(cur)
         else:
            print('有环，环的值是: ', cur.data)
            break
         cur = cur.next
      else:
         print('没有环')


if __name__ == '__main__':
   cl = CircleLinklist()
   nodes = []
   for i in 'ABCDEFGHIJKLMN':
      nodes.append(Node(i))
   nodes[len(nodes) - 1].next = nodes[5]  # 将最后一个节点的next指向F

   for node in nodes:
      cl.insert(node)

   # cl.traverse()

   cl.judge_circle_quick_slow_pointer()
   print('-' * 80)
   cl.judge_circle_by_traversing()
```

使用快慢指针，时间复杂度O(n)，空间复杂度O(1)

```c
 bool hasCycle(ListNode *head) {
        if(NULL == head){
            return false;
        }
        ListNode *pFirst = head;
        ListNode *pSecond = head;
        while((NULL != pFirst->next) && (NULL != pSecond->next) && (NULL != pSecond->next->next)){
            pFirst = pFirst->next;
            pSecond = pSecond->next->next;
            if(pFirst == pSecond){
                return true;
            }
        }
        return false;
    }
```



有一个链表，我们需要判断链表中是否存在环。有环则输出true，否则输出false。

输入有多行，每行为由空格分隔的两个整数m和n，m是当前结点的数据，n代表当前结点的指针域指向第n个结点。

n存在四种情形：
①为-1，代表该结点的指针域指向NULL，输入结束；
②指向该结点之前的结点，如第3个结点的指针域指向n = 2的结点；
③指向自己，如第3个结点的指针域指向n = 3的结点；
④指向其直接后继结点，如第3个结点的指针域指向n = 4的结点，不能指向n = 5的结点。

当输入为：
1 2
2 3
3 -1
时，代表：第1个结点的数据为1，指向第2个结点；第2个结点的数据为2，指向第3个结点；第3个结点的数据为3，指向NULL，输入结束。

```c
分析：先按正常顺序建好链表，之后遍历链表调整指向，最后再遍历判断节点是否被重复访问。
代码：

#include <iostream>
#include<cstdio>
#include<algorithm>
#include<cstring>
using namespace std;
struct node
{
    int data;
    node *next;
    int x=0;
} ;
int num[1000];
int main()
{
    int m,n;
    memset(num,0,sizeof(num));
    node *head=NULL,*p=NULL,*q=NULL;
    int i=1;
    while(scanf("%d%d",&m,&n)!=EOF)//先按正常顺序建立链表
    {
        num[i]=n;
        i++;
        p=new node;
        p->data=m;
        if(head==NULL)
            head=p;
        else
            q->next=p;
        q=p;
        if(n==-1)
        break;
    }
    //q->next=NULL;
    i=1;
    p=head;
    while(p->next)//修改链表的指向
    {
        if(num[i]!=i+1)
        {
            if(num[i]==-1)
                break;
            q=head;
            for(int j=1;j<num[i];j++)
                q=q->next;
            p->next=q;
            break;
        }
        p=p->next;
        i++;
    }
    p=head;
    int flag=0;
    while(p->next)//判断某个节点是否被重复访问
    {
        if(!p->x)
            p->x=1;
        else
            {
                flag=1;
                break;
            }
        p=p->next;
    }
    if(flag)
        cout<<"true"<<endl;
    else
        cout<<"false"<<endl;
    return 0;
}
```



##### 7、lldb 咋调试 iOS 真机的LLVM 

答：

##### 8、SSA 静态单赋值咋做的？

答：

##### 9、NSMutableArray copy ？

答：

[NSArray copy] 浅拷贝 还是那个对象
[NSArray mutableCopy] 深拷贝 得到NSMutableArray
[NSMutableArray copy] 深拷贝 得到 NSArray
[NSMutableArray mutableCopy] 深拷贝 得到 NSMutableArray

```objective-c
- (void)viewDidLoad {
    [super viewDidLoad];

    NSString *str = @"abc";
// 1.原来是一个可变数组
    NSMutableArray *muArray = [NSMutableArray arrayWithObjects:str, nil];//__NSArrayM
// 2.深拷贝  得到的是一个NSArray
    NSMutableArray *copyMutableArray = [muArray copy];//__NSArrayM
// 3.浅拷贝  得到的是一个 NSMutableArray
    NSMutableArray *mutablCopyMutableArray = [muArray mutableCopy];//__NSArrayM
//    [copyMutableArray addObject:@"ppp"];
//    [mutablCopyMutableArray addObject:@"lll"];
// 4. 初始化一个 NSArray    
    NSArray *array = [[NSArray alloc] initWithObjects:@"yyyy", nil];//__NSArrayI
// 5. 浅拷贝  还是那个对象
    NSArray *copyArray = [array copy];//__NSArrayI
// 6. 深拷贝 得到的是 NSMutaleArray
    NSMutableArray *mutableCopyArray = [array mutableCopy];//__NSArrayM
    [mutableCopyArray addObject:@"oooo"];
}

```



##### 10、消息转发机制

答：当调用一个NSObject对象不存在的方法时，并不会马上抛出异常，而是会经过多层转发，层层调用对象的
-resolveInstanceMethod: , 
-forwardingTargetForSelector: , 
-methodSignatureForSelector: ,
-forwardInvocation:等方法，其中最后-forwardInvocation：是会有一个NSInvocation对象，这个NSInvocation对象保存了这个方法调用的所有信息，包括Selector名，参数和返回值类型，最重要的是有所有参数值，可以从这个NSInvocation对象里拿到调用的所有参数值。

若想令类能理解某条消息，我们必须实现对应的方法才行，但是在编译期向类发送了其无法理解的消息并不会报错，因为在运行期可以继续向类中添加方法，所以编译器在编译时还不确定类中到底会不会有某个方法的实现。当对象接收到无法解读的消息后，就会启动“消息转发”机制，程序可由此过程告诉对象应该如何处理未知消息。

      消息转发分为两大阶段，第一阶段先征询接收者所属的类，看其是否能动态添加方法，以处理当前这个“未知的选择子”，这叫做“动态方法解析”。如果运行期系统已经把第一阶段执行完了，那么接收者自己就无法再以动态新增方法的手段来响应包含该选择的消息了。此时运行期系统会请求接收者以其他手段来处理与消息相关的方法调用。这又细分为两小步。首先请接收者看看有没有其他对象可以处理这条消息，若有，则运行期会把消息转给那个对象，于是消息转发过程结束，一切如常。若没有“备援的接收者”，则启动完整的消息转发机制，运行期会把与消息有关的全部细节都封装到NSInvocation对象中，再给接收者最后一次机会，令其设法解决当前还未处理的这条消息。

1.动态方法解析

      对象在无法解读消息会首先调用所属类的下列类方法：
    
      + (BOOL) resolveInstanceMethod:(SEL)selector
    
      参数为那个未知的选择子，返回值表示这个类能否新增一个实例方法处理此选择子。假如尚未实现的方法不是实例方法而是类方法则运行期会调用另一个方法：+ (BOOL) resolveClassMethod:(SEL)selector。使用这种方法的前提是：相关方法的实现代码已经写好，只等着运行的时候动态插入到类里面就可以了。此方案常用来实现@dynamic属性。

2.备援接收者

      当前接收者还有第二次机会能处理未知的选择子，这一步中，运行期会问它：能不能把这条消息转发给其他接收者来处理。与该步骤对应的处理方法：
    
      - (id) forwardingTargetForSelector:(SEL)selector
    
      方法参数代表未知的选择子，若当前接收者能找到备援对象，则将其返回，若找不到就返回nil。通过此方案我们可以用“组合”来模拟出“多重继承”的某些特性（因为OC属于单继承，一个字类只能继承一个基类）。在一个对象内部，可能还有一系列其他对象，该对象可能由此方法将能够处理某选择子的相关内部对象返回，这样的话，在外界看来，好像该对象亲自处理了这些消息。

3.完整的消息转发

      如果转发算法已经到了这一步的话，那么唯一能做的就是启用完整的消息转发机制了。首先创建NSInvocation对象，把与尚未处理的那条消息有关的全部细节都封装与其中。此对象包含选择子、目标（target）、参数。在出发NSInvocation对象时“消息派发系统”将亲自出马，把消息指派给目标对象。
    
      - (void) forwardInvocation:(NSInvocation *)invocation
    
    这个方法可以实现很简单：只需改变调用目标，使消息在新目标上得以调用即可，然而这样实现出来的方法与“备援接收者”方案所实现的方法等效，所以很少有人采用这么简单的实现方式。比较有用的实现方式为：在出发消息前，现已某种方式改变消息内容，比如追加另外一个参数，或者改换选择子等等。
    
    实现了此方法若发现某调用操作不应由本类处理，则需调用超类的同名方法。这样的话继承体系中的每个类都有机会处理此调用请求，直至NSObject。如果最后调用了NSObject类的方法，那么该方法还会继续调用“doesNotRecognizeSelector:”以抛出异常，此异常表明选择子最终未能得道处理。
我们使用一个现实场景来解释这个问题：当一个用点击屏幕上的一个按钮，这个过程具体发生了什么。

1.用户触摸屏幕，系统硬件进程会获取到这个点击事件，将事件简单处理封装后存到系统中，由于硬件检测进程和当前App进程是两个进程，所以进程两者之间传递事件用的是端口通信。硬件检测进程会将事件放到APP检测的端口。

2.APP启动主线程RunLoop会注册一个端口事件，来检测触摸事件的发生。当事件到达，系统会唤起当前APP主线程的RunLoop。来源就是App主线程事件，主线程会分析这个事件。

3.最后，系统判断该次触摸是否导致了一个新的事件, 也就是说是否是第一个手指开始触碰，如果是，系统会先从响应网中 寻找响应链。如果不是，说明该事件是当前正在进行中的事件产生的一个Touch message， 也就是说已经有保存好的响应链.

大致的过程 initial view –> super view –> ……–> view controller –> window –> Application

这里呢 UIResponder是所有可以响应事件的类的基类，其中包括最常见的UIView和UIViewController甚至是UIApplication，通过Hit-Test`过程找到这个`initial view，当用户点击了手机屏幕时，UIApplication就会调用UIWindow的hitTest: withEvent:方法。这个方法最终返回一个UiView。也就是我们要找到的那个最前的view

响应链中是有 controller 的，因为 controller 继承自 UIResponder。
UIApplication –> UIWindow –>递归找到最合适处理的控件 –> 控件调用 touches 方法 –> 判断是否实现 touches 方法 –> 没有实现默认会将事件传递给上一个响应者 –> 找到上一个响应者 –> 找不到方法作废

##### 11、Block 内部修改临时变量及内存变化 ？

答：

```objective-c
 //Block不允许修改外部变量的值，这里所说的外部变量的值，指的是栈中指针的内存地址。__block 所起到的作用就是只要观察到该变量被 block 所持有，就将“外部变量”在栈中的内存地址放到了堆中。进而在block内部也可以修改外部变量的值;block 内部的变量会被 copy 到堆区，“block内部”打印的是堆地址，因而也就可以知道，“定义后”打印的也是堆的地址。
//把三个16进制的内存地址转成10进制就是：

//定义后前：6171559672
//block内部：5732708296
//定义后后：5732708296
//中间相差438851376个字节，也就是 418.5M 的空间，因为堆地址要小于栈地址，又因为iOS中一个进程的栈区内存只有1M，Mac也只有8M，显然a已经是在堆区了。
//a 在定义前是栈区，但只要进入了 block 区域，就变成了堆区。这才是 __block 关键字的真正作用。
//__block 关键字修饰后，int类型也从4字节变成了32字节，这是 Foundation 框架 malloc 出来的。这也同样能证实上面的结论。（PS：居然比 NSObject alloc 出来的 16 字节要多一倍）。


__block int a = 0;
NSLog(@"定义前：%p", &a);         //栈区
void (^foo)(void) = ^{
    a = 1;
    NSLog(@"block内部：%p", &a);    //堆区
};
NSLog(@"定义后：%p", &a);         //堆区
foo();
//修改其block值
NSMutableString *a = [NSMutableString stringWithString:@"Tom"];
NSLog(@"\n 定以前：------------------------------------\n\
      a指向的堆中地址：%p；a在栈中的指针地址：%p", a, &a);               //a在栈区
void (^foo)(void) = ^{
    a.string = @"Jerry";
    NSLog(@"\n block内部：------------------------------------\n\
     a指向的堆中地址：%p；a在栈中的指针地址：%p", a, &a);               //a在栈区
    a = [NSMutableString stringWithString:@"William"];
};
foo();
NSLog(@"\n 定以后：------------------------------------\n\
      a指向的堆中地址：%p；a在栈中的指针地址：%p", a, &a);               //a在栈区
```



##### 12、Category添加属性？分类和扩展的区别？为什么不能直接添加属性

答：分类（Category）是OC中的特有语法，它是表示一个指向分类的结构体的指针。原则上它只能增加方法，不能增加成员（实例）变量。具体原因看源码组成，Category 是表示一个指向分类的结构体的指针，其定义如下：

```objective-c
struct objc_category {
  char *_Nonnull category_name             OBJC2_UNAVAILABLE;
  char *_Nonnull class_name                OBJC2_UNAVAILABLE;
  struct objc_method_list *_Nullable instance_methods   OBJC2_UNAVAILABLE;
  struct objc_method_list *_Nullable class_methods    OBJC2_UNAVAILABLE;
  struct objc_protocol_list *_Nullable protocols     OBJC2_UNAVAILABLE;
}
```

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

##### 13、网络请求

答：• CFSocket 是最底层的接口，只负责 socket 通信。
• CFNetwork 是基于 CFSocket 等接口的上层封装，ASIHttpRequest 工作于这一层。
• NSURLConnection 是基于 CFNetwork 的更高层的封装，提供面向对象的接口，AFNetworking 工作于这一层。
• NSURLSession 是 iOS7 中新增的接口，表面上是和 NSURLConnection 并列的，但底层仍然用到了 NSURLConnection 的部分功能 (比如 com.apple.NSURLConnectionLoader 线程)，AFNetworking2 和 Alamofire 工作于这一层。

下面主要介绍下 NSURLConnection 的工作过程。
通常使用 NSURLConnection 时，你会传入一个 Delegate，当调用了 [connection start] 后，这个 Delegate 就会不停收到事件回调。实际上，start 这个函数的内部会会获取 CurrentRunLoop，然后在其中的 DefaultMode 添加了4个 Source0 (即需要手动触发的Source)。CFMultiplexerSource 是负责各种 Delegate 回调的，CFHTTPCookieStorage 是处理各种 Cookie 的。
当开始网络传输时，我们可以看到 NSURLConnection 创建了两个新线程：com.apple.NSURLConnectionLoader 和 com.apple.CFSocket.private。其中 CFSocket 线程是处理底层 socket 连接的。NSURLConnectionLoader 这个线程内部会使用 RunLoop 来接收底层 socket 的事件，并通过之前添加的 Source0 通知到上层的 Delegate。

NSURLConnectionLoader 中的 RunLoop 通过一些基于 mach port 的 Source 接收来自底层 CFSocket 的通知。当收到通知后，其会在合适的时机向 CFMultiplexerSource 等 Source0 发送通知，同时唤醒 Delegate 线程的 RunLoop 来让其处理这些通知。CFMultiplexerSource 会在 Delegate 线程的 RunLoop 对 Delegate 执行实际的回调。

[AFURLConnectionOperation](https://link.jianshu.com/?t=https://github.com/AFNetworking/AFNetworking/blob/master/AFNetworking%2FAFURLConnectionOperation.m) 这个类是基于 NSURLConnection 构建的，其希望能在后台线程接收 Delegate 回调。为此 AFNetworking 单独创建了一个线程，并在这个线程中启动了一个 RunLoop：

```objectivec
+ (void)networkRequestThreadEntryPoint:(id)__unused object {
    @autoreleasepool {
        [[NSThread currentThread] setName:@"AFNetworking"];
        NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
        [runLoop addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
        [runLoop run];
    }
}
 
+ (NSThread *)networkRequestThread {
    static NSThread *_networkRequestThread = nil;
    static dispatch_once_t oncePredicate;
    dispatch_once(&oncePredicate, ^{
        _networkRequestThread = [[NSThread alloc] initWithTarget:self selector:@selector(networkRequestThreadEntryPoint:) object:nil];
        [_networkRequestThread start];
    });
    return _networkRequestThread;
}
```

RunLoop 启动前内部必须要有至少一个 Timer/Observer/Source，所以 AFNetworking 在 [runLoop run] 之前先创建了一个新的 NSMachPort 添加进去了。通常情况下，调用者需要持有这个 NSMachPort (mach_port) 并在外部线程通过这个 port 发送消息到 loop 内；但此处添加 port 只是为了让 RunLoop 不至于退出，并没有用于实际的发送消息。

```objectivec
- (void)start {
    [self.lock lock];
    if ([self isCancelled]) {
        [self performSelector:@selector(cancelConnection) onThread:[[self class] networkRequestThread] withObject:nil waitUntilDone:NO modes:[self.runLoopModes allObjects]];
    } else if ([self isReady]) {
        self.state = AFOperationExecutingState;
        [self performSelector:@selector(operationDidStart) onThread:[[self class] networkRequestThread] withObject:nil waitUntilDone:NO modes:[self.runLoopModes allObjects]];
    }
    [self.lock unlock];
}
```

当需要这个后台线程执行任务时，AFNetworking 通过调用 [NSObject performSelector:onThread:..] 将这个任务扔到了后台线程的 RunLoop 中。

AsyncDisplayKit

[AsyncDisplayKit](https://link.jianshu.com/?t=https://github.com/facebook/AsyncDisplayKit) 是 Facebook 推出的用于保持界面流畅性的框架，其原理大致如下：
UI 线程中一旦出现繁重的任务就会导致界面卡顿，这类任务通常分为3类：排版，绘制，UI对象操作。
排版通常包括计算视图大小、计算文本高度、重新计算子式图的排版等操作。绘制一般有文本绘制 (例如 CoreText)、图片绘制 (例如预先解压)、元素绘制 (Quartz)等操作。UI对象操作通常包括 UIView/CALayer 等 UI 对象的创建、设置属性和销毁。
其中前两类操作可以通过各种方法扔到后台线程执行，而最后一类操作只能在主线程完成，并且有时后面的操作需要依赖前面操作的结果 （例如TextView创建时可能需要提前计算出文本的大小）。ASDK 所做的，就是尽量将能放入后台的任务放入后台，不能的则尽量推迟 (例如视图的创建、属性的调整)。
为此，ASDK 创建了一个名为 ASDisplayNode 的对象，并在内部封装了 UIView/CALayer，它具有和 UIView/CALayer 相似的属性，例如 frame、backgroundColor等。所有这些属性都可以在后台线程更改，开发者可以只通过 Node 来操作其内部的 UIView/CALayer，这样就可以将排版和绘制放入了后台线程。但是无论怎么操作，这些属性总需要在某个时刻同步到主线程的 UIView/CALayer 去。
ASDK 仿照 QuartzCore/UIKit 框架的模式，实现了一套类似的界面更新的机制：即在主线程的 RunLoop 中添加一个 Observer，监听了 kCFRunLoopBeforeWaiting 和 kCFRunLoopExit 事件，在收到回调时，遍历所有之前放入队列的待处理的任务，然后一一执行。具体的代码可以看这里：[_ASAsyncTransactionGroup](https://link.jianshu.com/?t=https://github.com/facebook/AsyncDisplayKit/blob/master/AsyncDisplayKit%2FDetails%2FTransactions%2F_ASAsyncTransactionGroup.m)。

自己的理解：
客户端携带协议版本号、客户端随机数、客户端所支持的加密算法发送给服务器，服务器接收到客户端的随机数并确认支持改加密算法则返回证书继续；否则取消连接；
服务端携带公钥数字证书并附服务端随机数发送到客户端，客户端检验数字证书的有效性，无效就取消连接，有效就产生3个随机数并使用公钥加密发送回服务器；
客户端携带经过公钥加密后的随机数发送给服务端，服务端用私钥解密得到3个随机数；
此时，通信双方使用握手过程中产生的三个随机数生成session key 而它将作为往后传输数据时使用对称加密的密钥。
摘抄补充：

> 服务器会在建立真正的数据传输之前返回一个公钥数字证书。这里客户端需要在 URLSession 进行认证挑战方法回调里进行判断然后确定是否要继续进行请求；可以这样理解，URLSession 做 https 网络请求的时候其实会把请求鉴权的权限通过代理的方法给暴露出来，是否信任并继续建立连接可以按照特定规则去执行（如自签证书），只有 https 请求会走代理方法，http 则不进行回调，这也是为什么 iOS系统 为什么提倡使用 https 的原因。
>
> 生成一个用于请求数据对称加密的密钥（对称加密更快），用这个公钥进行非对称加密，在由服务器的私钥进行解密，得到这个密钥，那么，真正建立的数据传输就以此密钥进行加、解密。认证通过；

##### 14、post和get区别,应用层协议里的 GET 和 POST 有啥区别

答：

<u>**get方法**</u>，应用于 等幂操作 的请求；会被缓存且是主动行为；可以收藏书签；可以保留历史记录，有缓存；获取数据；get的querystring（仅支持urlencode编码）；querystring 是url的一部分get、post都可以带上；get请求长度最多1024kb；请求参数会被完整保留在浏览器历史记录里；

**<u>post方法</u>**，应用于 非等幂操作 的请求；POST 向指定资源提交数据进行处理请求（例如提交表单或者上传文件）。数据被包含在请求体中。POST请求可能会导致新的资源的建立和/或已有资源的修改；提交表单、数据等，没有历史记录且安全；querystring 是url的一部分get、post都可以带上；post的参数是放在body（支持多种编码）；post对请求数据没有限制；post在浏览器中返回的时候，会有再次提交请求的提示；POST中的参数不会被保留；

TCP 三次握手：握手过程中并不传输数据，在握手后服务器与客户端才开始传输数据，理想状态下，TCP 连接一旦建立，在通讯双方中的任何一方主动断开连接之前 TCP 连接会一直保持下去。
Socket 是对 TCP/IP 协议的封装，Socket 只是个接口不是协议，通过 Socket 我们才能使用 TCP/IP 协议，除了 TCP，也可以使用 UDP 协议来传递数据。
创建 Socket 连接的时候，可以指定传输层协议，可以是 TCP 或者 UDP，当用 TCP 连接，该Socket就是个TCP连接，反之。
Socket 原理
Socket 连接,至少需要一对套接字，分为 clientSocket，serverSocket 
连接分为3个步骤:
(1) 服务器监听:服务器并不定位具体客户端的套接字，而是时刻处于监听状态；
(2) 客户端请求:客户端的套接字要描述它要连接的服务器的套接字，提供地址和端口号，然后向服务器套接字提出连接请求；
(3) 连接确认:当服务器套接字收到客户端套接字发来的请求后，就响应客户端套接字的请求,并建立一个新的线程,把服务器端的套接字的描述发给客户端。一旦客户端确认了此描述，就正式建立连接。而服务器套接字继续处于监听状态，继续接收其他客户端套接字的连接请求.
Socket为长连接：通常情况下Socket 连接就是 TCP 连接，因此 Socket 连接一旦建立,通讯双方开始互发数据内容，直到双方断开连接。在实际应用中，由于网络节点过多，在传输过程中，会被节点断开连接，因此要通过轮询高速网络，该节点处于活跃状态。
很多情况下，都是需要服务器端向客户端主动推送数据，保持客户端与服务端的实时同步。
若双方是 Socket 连接，可以由服务器直接向客户端发送数据。
若双方是 HTTP 连接，则服务器需要等客户端发送请求后，才能将数据回传给客户端。
因此，客户端定时向服务器端发送请求，不仅可以保持在线，同时也询问服务器是否有新数据，如果有就将数据传给客户端

##### 15、NSDictionary存储结构如何解决hash冲突 failed

答：

##### 16、NSNotification原理，如何处理对象释放问题 ？

答：[通知原理解析](https://www.jianshu.com/p/e93b81fd3aa9)

NSNotificationCenter 内部根据每个通知名称name都引用着一个数组（这个数组应该是弱引用数组`NSHashTable`），数组里存着每个注册者的`target-action`，当发通知时（`[[NSNotificationCenter defaultCenter] postNotificationName:@"target" object:nil];`），会根据`NotificationName`找到对应的数组，然后便利数组里的`target-action`，达到批量分发的功能。

对于Observation持有observer：持有的是weak类型指针，当observer释放时observer会置nil，nil对象performSelector不再会崩溃

name和Observation是映射关系，observer和sel包含在Observation结构体中，NSNotification维护了全局对象表NCTable结构，结构体里包含GSIMapTable表的结构，用于存储Observation。

```objective-c
#define CHUNKSIZE   128
#define CACHESIZE   16
typedef struct NCTbl {
  Observation       *wildcard;  /* Get ALL messages.保存既没有通知名称又没有传入object的通知单链表； */
  GSIMapTable       nameless;   /* Get messages for any name.存储没有传入名字的通知名称的hash表。*/
  GSIMapTable       named;      /* Getting named messages only.存储传入了名字的通知的hash表。 */
  unsigned      lockCount;  /* Count recursive operations.  */
  NSRecursiveLock   *_lock;     /* Lock out other threads.  */
  Observation       *freeList;
  Observation       **chunks;
  unsigned      numChunks;
  GSIMapTable       cache[CACHESIZE];//用于快速缓存.
  unsigned short    chunkIndex;
  unsigned short    cacheIndex;
} NCTable;
//Observation结构
typedef struct  Obs {
  id observer;  //接受消息的对象   
  SEL selector; //执行的方法
  struct Obs    *next;  //下一Obs节点指针
  int   retained;   //引用计数
  struct NCTbl *link; //执向chunk table指针
} Observation;

```

没有传入名字的nameless表，key就是object参数，vaule为Observation结构体

- 在named表中GSIMapTable结构如下：

| key  | value    |
| ---- | -------- |
| name | maptable |
| name | maptable |
| name | maptable |

- maptable也是一个hash表，结构如下：

| key    | value       |
| ------ | ----------- |
| object | Observation |
| object | Observation |
| object | Observation |

> 传入名字的通知是存放在叫named的hash表
> kay为name，value还是maptable也是一个hash表
> maptable表的key为object参数，value为Observation参数

addObserver:selector:name:object: 方法内部实现原理

```objective-c
- (void) addObserver: (id)observer
            selector: (SEL)selector
                name: (NSString*)name
              object: (id)object
{
    Observation *list;
    Observation *o;
    GSIMapTable m;
    GSIMapNode  n;
    //入参检查异常处理
    ...
        //table加锁保持数据一致性,同一个线程按顺序执行，是同步的
    lockNCTable(TABLE);
        //创建Observation对象包装相应的调用函数
    o = obsNew(TABLE, selector, observer);
        //处理存在通知名称的情况
    if (name)
    {
        //table表中获取相应name的节点
        n = GSIMapNodeForKey(NAMED, (GSIMapKey)(id)name);
        if (n == 0)
        {
           //未找到相应的节点，则创建内部GSIMapTable表，以name作为key添加到talbe中
          m = mapNew(TABLE);
          name = [name copyWithZone: NSDefaultMallocZone()];
          GSIMapAddPair(NAMED, (GSIMapKey)(id)name, (GSIMapVal)(void*)m);
          GS_CONSUMED(name)
        }
        else
        {
            //找到则直接获取相应的内部table
            m = (GSIMapTable)n->value.ptr;
        }

        //内部table表中获取相应object对象作为key的节点
        n = GSIMapNodeForSimpleKey(m, (GSIMapKey)object);
        if (n == 0)
        {
            //不存在此节点，则直接添加observer对象到table中
            o->next = ENDOBS;//单链表observer末尾指向ENDOBS
            GSIMapAddPair(m, (GSIMapKey)object, (GSIMapVal)o);
        }
        else
        {
            //存在此节点，则获取并将obsever添加到单链表observer中
            list = (Observation*)n->value.ptr;
            o->next = list->next;
            list->next = o;
        }
    }
    //只有观察者对象情况
    else if (object)
    {
        //获取对应object的table
        n = GSIMapNodeForSimpleKey(NAMELESS, (GSIMapKey)object);
        if (n == 0)
        {
            //未找到对应object key的节点，则直接添加observergnustep-base-1.25.0
            o->next = ENDOBS;
            GSIMapAddPair(NAMELESS, (GSIMapKey)object, (GSIMapVal)o);
        }
        else
        {
            //找到相应的节点则直接添加到链表中
            list = (Observation*)n->value.ptr;
            o->next = list->next;
            list->next = o;
        }
    }
    //处理即没有通知名称也没有观察者对象的情况
    else
    {
        //添加到单链表中
        o->next = WILDCARD;
        WILDCARD = o;
    }
        //解锁
    unlockNCTable(TABLE);
}
```

添加通知的基本逻辑：

1. 根据传入的selector和observer创建Observation，并存入GSIMaptable中，如果已存在，则是从cache中取。
2. 如果name存在:
   - 则向named表中插入元素，key为name,value为GSIMaptable。
   - GSIMaptable里面key为object，value为Observation,结束
3. 如果name不存在:
   - 则向nameless表中插入元素,key为object,value为Observation，结束
4. 如果name和object都不存在，则把这个Observation添加WILDCARD链表中

addObserverForName:object:queueusingBlock:实现原理

```objective-c
//对于block形式，里面创建了GSNotificationObserver对象，然后在调用addObserver: selector: name: object:
- (id) addObserverForName: (NSString *)name 
                   object: (id)object 
                    queue: (NSOperationQueue *)queue 
               usingBlock: (GSNotificationBlock)block
{
    GSNotificationObserver *observer = 
        [[GSNotificationObserver alloc] initWithQueue: queue block: block];

    [self addObserver: observer 
             selector: @selector(didReceiveNotification:) 
                 name: name 
               object: object];
    return observer;
}

/*
1.初始化该队列会创建Block_copy 拷贝block
2.并确定通知操作队列
*/
- (id) initWithQueue: (NSOperationQueue *)queue 
               block: (GSNotificationBlock)block
{
    self = [super init];
    if (self == nil)
        return nil;
    ASSIGN(_queue, queue);
    _block = Block_copy(block);
    return self;
}

/*
1.通知的接受处理函数didReceiveNotification，
2.如果queue不为空，通过addOperation来实现指定操作队列处理
3.如果queue不为空,直接在当前线程执行block。
*/
- (void) didReceiveNotification: (NSNotification *)notif
{
    if (_queue != nil)
    {
        GSNotificationBlockOperation *op = [[GSNotificationBlockOperation alloc] 
            initWithNotification: notif block: _block];

        [_queue addOperation: op];
    }
    else
    {
        CALL_BLOCK(_block, notif);
    }
}
```

发送通知的实现 postNotificationName: name: object：

```objective-c
 - (void) _postAndRelease: (NSNotification*)notification
{
    1.入参检查校验
    2.创建存储所有匹配通知的数组GSIArray
    3.加锁table避免数据一致性问题
    4.查找既不监听name也不监听object所有的wildcard类型的Observation，加入数组GSIArray中
    5.查找NAMELESS表中指定对应观察者对象object的Observation并添加到数组中
    6.查找NAMED表中相应的Observation并添加到数组中
        1. 首先查找name与object的一致的Observation加入数组中
        2. 当object为nil的Observation加入数组中
        3. 当object不为nil，并且object和发送通知的object不一致不为添加到数组中
    //解锁table
    //遍历整个数组并依次调用performSelector:withObject处理通知消息发送
    //解锁table并释放资源
}
```

对于addObserver方法，为什么需要object参数?

###### 1、对addObserver当你不传入name也可以，传入object，当postNotification方法同样发出这个object时，就会触发通知方法。

> 因为当name不存在的时候，会继续判断object，则向nameless的maptable表中插入元素，key为object，value为Observation

###### 2、都传入null对象会怎么样

你可能也注意到了，addObserver方法name和object都可以为空，这表示将会把observer赋值为 wildcard，他将会监听所有的通知。

###### 3、通知的发送时同步的，还是异步的。

同步异步这个问题，由于TABLE资源的问题，同一个线程会按顺序遍历数组执行，自然是同步的。

###### 4、NSNotificationCenter接受消息和发送消息是在一个线程里吗？如何异步发送消息

由于是使用的performSelector方法，没有进行转线程，默认是postNotification方法的线程。

```php
[o->observer performSelector: o->selector 
withObject: notification];
```

对于异步发送消息，可以使用NSNotificationQueue，queue顾明意思，我们是需要将NSNotification放入queue中执行的。

NSNotificationQueue发送消息的三种模式：

```objectivec
typedef NS_ENUM(NSUInteger, NSPostingStyle) {
    NSPostWhenIdle = 1, // 当runloop处于空闲状态时post
    NSPostASAP = 2, // 当当前runloop完成之后立即post
    NSPostNow = 3  // 立即post
};

NSNotification *notification = [NSNotification notificationWithName:@"111" object:nil];
[[NSNotificationQueue defaultQueue] enqueueNotification: notification postingStyle:NSPostASAP];
```

- NSPostingStyle为NSPostNow 模式是同步发送,
- NSPostWhenIdle或者NSPostASAP是异步发送

###### 5、NSNotificationQueue和runloop的关系？

NSNotificationQueue 是依赖runloop才能成功触发通知，如果去掉runloop的方法，你会发现无法触发通知。

```objectivec
dispatch_async(dispatch_get_global_queue(0, 0), ^{
    //子线程的runloop需要自己主动开启     
   NSNotification *notification = [NSNotification notificationWithName:@"TEST" object:nil];
        [[NSNotificationQueue defaultQueue] enqueueNotification:notification postingStyle:NSPostWhenIdle coalesceMask:NSNotificationNoCoalescing forModes:@[NSDefaultRunLoopMode]];
        // run runloop
        [[NSRunLoop currentRunLoop] addPort:[NSPort port] forMode:NSRunLoopCommonModes];
        CFRunLoopRun();
        NSLog(@"3");
    });
```

```objectivec
NSNotification *notification = [NSNotification notificationWithName:@"111" object:nil];
[[NSNotificationQueue defaultQueue] enqueueNotification: notification postingStyle:NSPostASAP];
```

> NSNotificationQueue将通知添加到队列中时，其中postringStyle参数就是定义通知调用和runloop状态之间关系。

###### 6、如何保证通知接收的线程在主线程？

1. 保证主线程发送消息或者接受消息方法里切换到主线程

2. 接收到通知后跳转到主线程，苹果建议使用NSMachPort进行消息转发到主线程。

   ```objective-c
   @interface GKViewController : UIViewController<NSMachPortDelegate>
   /** 储存子线程发出的通知的队列 */
   @property (nonatomic, strong) NSMutableArray *notificationsQueue;
   /** 处理通知事件的预期线程 */
   @property (nonatomic, strong) NSThread *mainThread;
   /** 用于对通知队列枷锁的锁对象，避免线程冲突 */
   @property (nonatomic, strong) NSLock *lock;
   /** 用于向期望线程发送信号的通信端口 */
   @property (nonatomic, strong) NSMachPort *machPort;
   
   @end
     
     //对相关的成员属性进行初处
   -(void)setUpThredingSupport{
       if(self.notificationsQueue){
           return;
       }
       self.notificationsQueue =[[NSMutableArray alloc] init];
       //队列:用来暂存其他线程发出的通知
       self.lock = [[NSLock alloc] init];
       //负责栈操作的原子性
       self.mainThread = [NSThread currentThread];
       //记录处理通知的线程
       self.machPort  = [[NSMachPort alloc] init];
       //负责往处理通如的线程所对应RunLoop中发送消息
       [self.machPort setDelegate:self];
       //将Mac Port添加到处理通知的线程中的RunLoop中
       [[NSRunLoop currentRunLoop] addPort:self.machPort forMode:(__bridge NSString *)kCFRunLoopCommonModes];
   }
   -(void)processNotification: (NSNotification *)notification {
       if ([NSThread currentThread] == _mainThread) {
           //处理出队列中的通知
           NSLog(@"handle notification thread = %@", [NSThread currentThread]);
           NSLog(@"process notification");
       } else {
           // 在子线程中收到通知后，将收到的通知放入到队列中存储，然后给主线程的RunLoop发送处理通知的消息
           NSLog(@"transfer notification thread %@", [NSThread currentThread]);
           // Forward the notification to the corroct thread.
           [self.lock lock];
           [self.notificationsQueue addObject:notification]; //将其他线程中发过来的通知不做处理，入队列暂存
           [self.lock unlock];
           // 通过MacPort给处理通知的线程发送通知，使其处理队列中所暂存的队列
           [self.machPort sendBeforeDate:[NSDate date]
                              components:nil
                                    from:nil
                                reserved:0];
       }
   }
   //从子线程收到Mach Port发出的消息后所执行的方法
   //在该方法中从队列中获取子线程中发出的NSNotification然后使用当前线程来处理该通知
   //RunLoop收到Mac Port发出的消息时所执行的回调方法
   -(void)handleMachMessage: (void *)msg {
       NSLog(@"handle Mach Message thread = %@", [NSThread currentThread]);
       //在子线程中对notificationsQueue队列操作时，需要加锁，保持队列中数据的正确性
       [self.lock lock];
       //依次取出队列中所暂存的Notification,然后在当前线程中处理该通知
       while ([self.notificationsQueue count]) {
           NSNotification *notification = [self.notificationsQueue objectAtIndex:0]; [self.notificationsQueue removeObjectAtIndex:8]; //取出队列中第一个[
           [self.lock unlock];
           [self processNotification:notification]; //处理从队列中取出的通知
           [self.lock lock];
           [self.lock unlock];
       }
   }
   ```

   3. 使用block接口addObserverForName:object:queue:usingBlock:指定主线程

###### 7、系统通知的原理，以及出现的常见问题

答：使用target-action方式的原理大概是这个样子的：

在ios9之前，有些类是不支持weak指针的，所以使用的是unsafe_unretained类型的指针；顾名思义，unsafe_unretained的意思就是不安全的，不会对对象持有的指针，weak所指向的地址被注销时，指针自动指向nil，向nil指针发送消息是不会崩溃的；而unsafe_unretained指针指向的内存被销毁时，是不会指向nil的，从而导致野指针，所以ios9之前是需要注销的；
当你调用注销方法时，NSNotificationCenter会把指针指向nil，避免出现野指针

内存大概是这个样子的

NSNotificationCenter 内部根据每个通知名称name都引用着一个数组（这个数组应该是弱引用数组NSHashTable），数组里存着每个注册者的target-action，当发通知时（[[NSNotificationCenter defaultCenter] postNotificationName:@"target" object:nil];），会根据NotificationName找到对应的数组，然后便利数组里的target-action，达到批量分发的功能。

使用block方式大概是这个样子的：

注册通知时，不需要你传Observer，NSNotificationCenter根据NotificationName持有一个数组，然后把block放到这个数组中，当你调用postNotification时，根据NotificationName找到对应数组并遍历调用；（但是会出现block不会被释放的问题）

子线程发送通知时：

在发送时都是遍历target-action或block调用方法，因此，在子线程发送通知，方法调用也是发生在子线程，也就是说，接收也是默认在子线程的，所以如果你需要在接收通知时刷新UI，建议跳转到主线程哦。

target-action方式需要声明单独的方法接收，如果支持ios9之前的版本还得注销，真是好麻烦。block方式呢，还会出现内存问题，API提供的较少，因此我决定，封装一款自己的通知 - PSSNotificationCenter

##### 17、APN 推送原理

答：

##### 18、async mainThread 与 runloop 关系 ？RunLoop 的作用是什么？它的内部工作机制了解么？

答：首先，iOS 开发中能遇到两个线程对象: pthread_t 和 NSThread。过去苹果有份[文档](https://link.jianshu.com/?t=http://www.fenestrated.net/~macman/mirrors/Apple Technotes (As of 2002)/tn/tn2028.html)标明了 NSThread 只是 pthread_t 的封装，但那份文档已经失效了，现在它们也有可能都是直接包装自最底层的 mach thread。苹果并没有提供这两个对象相互转换的接口，但不管怎么样，可以肯定的是 pthread_t 和 NSThread 是一一对应的。比如，你可以通过 pthread_main_thread_np() 或 [NSThread mainThread] 来获取主线程；也可以通过 pthread_self() 或 [NSThread currentThread] 来获取当前线程。CFRunLoop 是基于 pthread 来管理的。
苹果不允许直接创建 RunLoop，它只提供了两个自动获取的函数：CFRunLoopGetMain() 和 CFRunLoopGetCurrent()。 这两个函数内部的逻辑大概是下面这样:

```objectivec
/// 全局的Dictionary，key 是 pthread_t， value 是 CFRunLoopRef
static CFMutableDictionaryRef loopsDic;
/// 访问 loopsDic 时的锁
static CFSpinLock_t loopsLock;
 
/// 获取一个 pthread 对应的 RunLoop。
CFRunLoopRef _CFRunLoopGet(pthread_t thread) {
    OSSpinLockLock(&loopsLock);
    if (!loopsDic) {
        // 第一次进入时，初始化全局Dic，并先为主线程创建一个 RunLoop。
        loopsDic = CFDictionaryCreateMutable();
        CFRunLoopRef mainLoop = _CFRunLoopCreate();
        CFDictionarySetValue(loopsDic, pthread_main_thread_np(), mainLoop);
    }
    
    /// 直接从 Dictionary 里获取。
    CFRunLoopRef loop = CFDictionaryGetValue(loopsDic, thread));
    if (!loop) {
        /// 取不到时，创建一个
        loop = _CFRunLoopCreate();
        CFDictionarySetValue(loopsDic, thread, loop);
        /// 注册一个回调，当线程销毁时，顺便也销毁其对应的 RunLoop。
        _CFSetTSD(..., thread, loop, __CFFinalizeRunLoop);
    }
    OSSpinLockUnLock(&loopsLock);
    return loop;
}
 
CFRunLoopRef CFRunLoopGetMain() {
    return _CFRunLoopGet(pthread_main_thread_np());
}
CFRunLoopRef CFRunLoopGetCurrent() {
    return _CFRunLoopGet(pthread_self());
}
```

从上面的代码可以看出，线程和 RunLoop 之间是一一对应的，其关系是保存在一个全局的 Dictionary 里。线程刚创建时并没有 RunLoop，如果你不主动获取，那它一直都不会有。RunLoop 的创建是发生在第一次获取时，RunLoop 的销毁是发生在线程结束时。你只能在一个线程的内部获取其 RunLoop（主线程除外）。

RunLoop 对外的接口

在 CoreFoundation 里面关于 RunLoop 有5个类:
CFRunLoopRef CFRunLoopModeRef CFRunLoopSourceRef CFRunLoopTimerRef CFRunLoopObserverRef

一个 RunLoop 包含若干个 Mode，每个 Mode 又包含若干个 Source/Timer/Observer。每次调用 RunLoop 的主函数时，只能指定其中一个 Mode，这个Mode被称作 CurrentMode。如果需要切换 Mode，只能退出 Loop，再重新指定一个 Mode 进入。这样做主要是为了分隔开不同组的 Source/Timer/Observer，让其互不影响。

CFRunLoopSourceRef 是事件产生的地方。Source有两个版本：Source0 和 Source1。
• Source0 只包含了一个回调（函数指针），它并不能主动触发事件。使用时，需要先调用 CFRunLoopSourceSignal(source)，将这个 Source 标记为待处理，然后手动调用 CFRunLoopWakeUp(runloop) 来唤醒 RunLoop，让其处理这个事件。
• Source1 包含了一个 mach_port 和一个回调（函数指针），被用于通过内核和其他线程相互发送消息。这种 Source 能主动唤醒 RunLoop 的线程。

CFRunLoopTimerRef 是基于时间的触发器，它和 NSTimer 是toll-free bridged 的，可以混用。其包含一个时间长度和一个回调（函数指针）。当其加入到 RunLoop 时，RunLoop会注册对应的时间点，当时间点到时，RunLoop会被唤醒以执行那个回调。

CFRunLoopObserverRef 是观察者，每个 Observer 都包含了一个回调（函数指针），当 RunLoop 的状态发生变化时，观察者就能通过回调接受到这个变化。可以观测的时间点有以下几个：

```objectivec
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    kCFRunLoopEntry         = (1UL << 0), // 即将进入Loop
    kCFRunLoopBeforeTimers  = (1UL << 1), // 即将处理 Timer
    kCFRunLoopBeforeSources = (1UL << 2), // 即将处理 Source
    kCFRunLoopBeforeWaiting = (1UL << 5), // 即将进入休眠
    kCFRunLoopAfterWaiting  = (1UL << 6), // 刚从休眠中唤醒
    kCFRunLoopExit          = (1UL << 7), // 即将退出Loop
};
```

上面的 Source/Timer/Observer 被统称为 mode item，一个 item 可以被同时加入多个 mode。但一个 item 被重复加入同一个 mode 时是不会有效果的。如果一个 mode 中一个 item 都没有，则 RunLoop 会直接退出，不进入循环。

RunLoop 的 Mode

CFRunLoopMode 和 CFRunLoop 的结构大致如下：

```objectivec
struct __CFRunLoopMode {
    CFStringRef _name;            // Mode Name, 例如 @"kCFRunLoopDefaultMode"
    CFMutableSetRef _sources0;    // Set
    CFMutableSetRef _sources1;    // Set
    CFMutableArrayRef _observers; // Array
    CFMutableArrayRef _timers;    // Array
    ...
};
 
struct __CFRunLoop {
    CFMutableSetRef _commonModes;     // Set
    CFMutableSetRef _commonModeItems; // Set<Source/Observer/Timer>
    CFRunLoopModeRef _currentMode;    // Current Runloop Mode
    CFMutableSetRef _modes;           // Set
    ...
};
```

这里有个概念叫 "CommonModes"：一个 Mode 可以将自己标记为"Common"属性（通过将其 ModeName 添加到 RunLoop 的 "commonModes" 中）。每当 RunLoop 的内容发生变化时，RunLoop 都会自动将 _commonModeItems 里的 Source/Observer/Timer 同步到具有 "Common" 标记的所有Mode里。

应用场景举例：主线程的 RunLoop 里有两个预置的 Mode：kCFRunLoopDefaultMode 和 UITrackingRunLoopMode。这两个 Mode 都已经被标记为"Common"属性。DefaultMode 是 App 平时所处的状态，TrackingRunLoopMode 是追踪 ScrollView 滑动时的状态。当你创建一个 Timer 并加到 DefaultMode 时，Timer 会得到重复回调，但此时滑动一个TableView时，RunLoop 会将 mode 切换为 TrackingRunLoopMode，这时 Timer 就不会被回调，并且也不会影响到滑动操作。

有时你需要一个 Timer，在两个 Mode 中都能得到回调，一种办法就是将这个 Timer 分别加入这两个 Mode。还有一种方式，就是将 Timer 加入到顶层的 RunLoop 的 "commonModeItems" 中。"commonModeItems" 被 RunLoop 自动更新到所有具有"Common"属性的 Mode 里去。

CFRunLoop对外暴露的管理 Mode 接口只有下面2个:

```objectivec
CFRunLoopAddCommonMode(CFRunLoopRef runloop, CFStringRef modeName);
CFRunLoopRunInMode(CFStringRef modeName, ...);
```

Mode 暴露的管理 mode item 的接口有下面几个：

```objectivec
CFRunLoopAddSource(CFRunLoopRef rl, CFRunLoopSourceRef source, CFStringRef modeName);
CFRunLoopAddObserver(CFRunLoopRef rl, CFRunLoopObserverRef observer, CFStringRef modeName);
CFRunLoopAddTimer(CFRunLoopRef rl, CFRunLoopTimerRef timer, CFStringRef mode);
CFRunLoopRemoveSource(CFRunLoopRef rl, CFRunLoopSourceRef source, CFStringRef modeName);
CFRunLoopRemoveObserver(CFRunLoopRef rl, CFRunLoopObserverRef observer, CFStringRef modeName);
CFRunLoopRemoveTimer(CFRunLoopRef rl, CFRunLoopTimerRef timer, CFStringRef mode);
```

你只能通过 mode name 来操作内部的 mode，当你传入一个新的 mode name 但 RunLoop 内部没有对应 mode 时，RunLoop会自动帮你创建对应的 CFRunLoopModeRef。对于一个 RunLoop 来说，其内部的 mode 只能增加不能删除。

苹果公开提供的 Mode 有两个：kCFRunLoopDefaultMode (NSDefaultRunLoopMode) 和 UITrackingRunLoopMode，你可以用这两个 Mode Name 来操作其对应的 Mode。

同时苹果还提供了一个操作 Common 标记的字符串：kCFRunLoopCommonModes (NSRunLoopCommonModes)，你可以用这个字符串来操作 Common Items，或标记一个 Mode 为 "Common"。使用时注意区分这个字符串和其他 mode nam

 1. 通知Observers，即将进入RunLoop（此处有Observer会创建AutoreleasePool）
    2. 通知 Observers: 即将触发 Timer 回调。
    3. 通知 Observers: 即将触发 Source (非基于port的,Source0) 回调。
    4. 触发 Source0 (非基于port的) 回调。
    5. 通知Observers，即将进入休眠
7. sleep to wait msg.
 7. 通知Observers，线程被唤醒
 8. 如果是被Timer唤醒的，回调Timer
 9. 如果是被dispatch唤醒的，执行所有调用 dispatch_async 等方法放入main queue 的 block
 10. 如果如果Runloop是被 Source1 (基于port的) 的事件唤醒了，处理这个事件
 11. 通知Observers，即将退出RunLoop（此处有Observer释放AutoreleasePool）

/// 用DefaultMode启动
/// 用指定的Mode启动，允许设置RunLoop超时时间

/// RunLoop的实现
    /// 首先根据modeName找到对应mode
    /// 如果mode里没有source/timer/observer, 直接返回。
    /// 1. 通知 Observers: RunLoop 即将进入 loop。
    /// 内部函数，进入loop
            /// 2. 通知 Observers: RunLoop 即将触发 Timer 回调。
            /// 3. 通知 Observers: RunLoop 即将触发 Source0 (非port) 回调。
            /// 执行被加入的block
            /// 4. RunLoop 触发 Source0 (非port) 回调。
            /// 执行被加入的block
            /// 5. 如果有 Source1 (基于port) 处于 ready 状态，直接处理这个 Source1 然后跳转去处理消息。
            /// 通知 Observers: RunLoop 的线程即将进入休眠(sleep)。
/// 7. 调用 mach_msg 等待接受 mach_port 的消息。线程将进入休眠, 直到被下面某一个事件唤醒。
            /// • 一个基于 port 的Source 的事件。
            /// • 一个 Timer 到时间了
            /// • RunLoop 自身的超时时间到了
            /// • 被其他什么调用者手动唤醒       
/// 8. 通知 Observers: RunLoop 的线程刚刚被唤醒了。
            /// 收到消息，处理消息。
            /// 9.1 如果一个 Timer 到时间了，触发这个Timer的回调。
            /// 9.2 如果有dispatch到main_queue的block，执行block。
            /// 9.3 如果一个 Source1 (基于port) 发出事件了，处理这个事件
            /// 执行加入到Loop的block
                /// 进入loop时参数说处理完事件就返回。
                /// 超出传入参数标记的超时时间了
                /// 被外部调用者强制停止了
                /// source/timer/observer一个都没有了
            /// 如果没超时，mode里没空，loop也没被停止，那继续loop。
    /// 10. 通知 Observers: RunLoop 即将退出。

线程和 RunLoop 之间是一一对应的，其关系是保存在一个全局的 Dictionary 里。线程刚创建时并没有 RunLoop，如果你不主动获取，那它一直都不会有。RunLoop 的创建是发生在第一次获取时，RunLoop 的销毁是发生在线程结束时。你只能在一个线程的内部获取其 RunLoop（主线程除外）

##### 19、sdwebimage图片缓存框架设计 ？

答：

##### 20、多个线程，操作同一个变量，会有什么后果？

答：

```objective-c
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
```

bookName的声明中指定属性nonatomic，表示为非线程安全的，set方法没有上锁。多线程同时进入到set方法进行操作时会引发问题，只要改换为atomic即可避免该问题,那为什么使用nonatomic就不行？多线程同时进入set方法执行时为什么会出错？原因也很简单，因为在ARC下编译器自动生成的set方法中，需要先对之前持有的对象进行release释放操作，如果此时没有加锁，就有可能发生多个线程执行同一段代码，导致release语句被执行了多次，从而导致对同一块内存多次发送release消息，导致的对已经回收的内存进行重复回收，即会报错。

如果修改为：就不会报错

```objective-c
[p1 setBookName:[NSString stringWithFormat:@"1234 %d",i]];
```

苹果当中为了节省内存和提高执行效率，提出的一种Tagged Pointer的概念，在8个字节之内能存放的范围之内，系统就会以Tagged Pointer的方式生成指针，如果 8 字节承载不了就会用普通的方式来生成普通指针。这个TaggedPointer与普通指针不同，普通指针是一个变量，存放在栈上，其内容是一块内存块的基地址，所以说指针指向一个内存块，但TaggedPointer并不是一个真正的对象，实际上他的内容就是真正的值，而不是内存地址，所以他本质上就是一个普通变量，也不持有某个内存块。所以在set方法中，对这样的并没有持有内存块的普通变量进行赋值是不需要提前release的，是完全没有问题的，所以不会报错。

##### 21、MJ底层班说继承NSProxy的类本类的方法列表找不到imp，不会去父类查找，测试下来会去父类查找，请问一下Proxy的查找机制在哪里看到呢？

答：

##### 22、for in，是怎么实现的？

答：

##### 23、for in 跟for( int i, i<10, i++)的写法有什么不同？

答：

##### 24、FIFO FILO FRU原理？

答：

##### 25、编译原理和计算机组成原理

答：

##### 26、KVO的实现原理？

答：就是为对象添加一个观察者“Observer”，当其属性值发生改变时，就会调用"observeValueForKeyPath:"方法，为我们提供一个“对象值改变了！”的时机进行一些操作。Key-Value Obersver，即键值观察。它是观察者模式的一种衍生。基本思想是，对目标对象的某属性添加观察，当该属性发生变化时，会自动的通知观察者。这里所谓的通知是触发观察者对象实现的KVO的接口方法。
** KVO是解决model和view同步的好法子。**
另外，KVO的优点是当被观察的属性值改变时是会自动发送通知的，这比通知中心需要post通知来说，简单了许多。
KVO:当指定的对象的属性被修改了，允许对象接收到通知的机制。
利用RuntimeAPI动态生成一个子类，并且让instance对象的isa指向这个全新的子类，当修改instance对象的属性时，会调用Foundation的，_NSSetXXXValueAndNotify函数，此函数的内部实现为 调用willChangeValueForKey，调用父类(原来)的setter实现，调用didChangeValueForKey:，didChangeValueForKey:内部会调用observer的observeValueForKeyPath:ofObject:change:context:方法。
Apple使用了isa混写（isa-swizzling）来实现KVO。当观察对象A时，KVO机制动态创建一个新的名为NSKVONotifying_A的新类，该类集成自对象A的本类，且KVO为NSKVONotifying_A重写观察属性的setter方法，setter方法会负责在调用元setter方法之前和之后，通知所有观察对象属性值的更改情况。（备注：isa混写（isa-swizzling）isa:is a kind of ; swizzling: 混合，搅合）
1、NSKVONotifying_A类剖析：在这个过程，被观察对象的isa指针从指向原来的A类，被KVO机制修改为指向系统创建的自NSKVONotifying_A类，来实现当前类属性值改变的监听；
所以当我们从应用层面来看，完全没有意识到有新的类出现，这是系统“隐瞒”了对KVO的底层想实现过程，让我们误以为还是原来的类。但是此时如果我们创建一个新的名为“NSKVONotifying_A”的类，就会发现系统运行到注册KVO的那段代码时程序就崩溃，因为系统在注册监听的时候动态创建了名为NSKVONotifying_A的中间类，并指向这个中间类了
（isa指针的作用：每个对象都有isa指针，指向该对象的类，他告诉Runtime系统这个对象的类是什么。所以对象注册为观察者时，isa指针指向新子类，那么这个被观察的对象就神奇地变成新子类的对象（或实例）了。）因而在该对象上对setter的调用就会调用已重写的setter，从而激活键值通知机制。
2、子类setter方法剖析：KVO的键值观察通知依赖与NSObject的两个方法：willChangeValueForKey:和didChangeValueForKey:，在存取数值的前后分别调用2个方法：
被观察属性发生改变之前，willChangeValueForkey:被调用，通知系统该keyPath的属性值即将变更；当改变发生后，didChangeValueForkey:被调用，通知系统该keyPath的属性值已经变更；之后，observeValueForKey:ofObject:context:也会被调用。且重写观察属性的setter方法这种继承方式的注入是在运行时而不是编译时实现的。

##### 27、KVC的实现原理？

答：俗称“键值编码”，通过一个key来访问某个属性；一种间接访问对象属性的机制，甚至可以通过KVC来访问对象的私有属性！修改textField的placeholder也是通过KVC修改的，KVC对多种数据类型的支持，KVC在某种程度上提供了替代存取方法（访问器方法）的方案，不过存取方法终究是个好东西，以至于只要有可能，KVC也尽可能先尝试使用存取方法访问属性。当使用KVC访问属性时，它内部其实做了很多事：
1.首先查找有无<property>，set<property>，is<property>等property属性对应的存取方法，若有，则直接使用这些方法;
2.若无，则继续查找_<property>，_get<property>，set<property>等方法，若有就使用；
3.若查询不到以上任何存取方法，则尝试直接访问实例变量<property>， 
4.若连该成员变量也访问不到，则会在下面方法中抛出异常。之所以提供这两个方法，就是让你在因访问不到该属性而程序即将崩掉前，供你重写，在内做些处理，防止程序直接崩掉。
5.利用KVC即键值编码来给对象的私有属性赋值。
6.如何手动触发KVO，valueForUndefinedKey:和setValue:forUndefinedKey:方法。

##### 28、top level code

答：在App工程里， `.swift`文件都是编译成模块的，不能有`top level code`。

先明确一个概念，一个`.swift`文件执行是从它的第一条`非声明语句（`表达式、控制结构）开始的，同时包括声明中的赋值部分，所有这些语句，构成了该`.swift`文件的`top_level_code()`函数。而所有的声明，包括结构体、类、枚举及其方法，都不属于 `top_level_code()`代码部分，其中的代码逻辑，包含在其他区域，`top_level_code()`可以直接调用他们。程序的入口是隐含的一个 `main(argc, argv)`函数，该函数执行逻辑是设置全局变量`C_ARGC C_ARGV`，然后调用 top_level_code()。不是所有的 .swift 文件都可以作为模块，目前看，任何包含表达式语句和控制语句的 .swift 文件都不可以作为模块。正常情况下模块可以包含全局变量(var)、全局常量(let)、结构体(struct)、类(class)、枚举(enum)、协议(protocol)、扩展(extension)、函数(func)、以及全局属性(var { get set })。这里的全局，指的是定义在 top level 。这里说的表达式指`expression`，语句指 `statement` ，声明指`declaration` 。因此，如果代码中直接在类的外面调用类内部的方法，则该.swift 文件是编译不成的模块的，所以会编译报错。

##### 29、mach port的具体的原理过程

答：

##### 30、单例

答：一个单例类，在整个程序中只有一个实例，并且提供一个类方法供全局调用，在编译时初始化这个类，然后一直保存在内存中，到程序（APP）退出时由系统自动释放这部分内存

##### 31、方法重载和方法重写

答：重载发生在**编译时**，而重载发生在运行时:重载方法调用与其定义的绑定(bind)发生在编译时，但是重载方法调用对其定义的绑定(bind)发生在**运行时**。

静态方法可以重载，这意味着一个类可以具有多个同名的静态方法。静态方法不能被覆盖，即使您在子类中声明了相同的静态方法，也与父类的相同方法无关。

最基本的区别在于，重载是在同一类中完成的，而要覆盖基类和子类，则需要重载。重写就是为父类的继承方法提供特定的实现。

**静态绑定****(bind)**用于重载方法，**动态绑定****(bind)**用于重载/覆盖方法。

性能:与覆盖相比，重载可提供更好的性能。原因是重写的方法的绑定(bind)是在运行时完成的。

private和final方法可以重载，但是不能被覆盖。这意味着一个类可以具有多个同名的私有(private)/ final方法，但是子类不能覆盖其基类的私有(private)/ final方法。

在方法重载的情况下，方法的返回类型无关紧要，可以相同也可以不同。但是，在方法重写的情况下，重写方法可以具有更特定的返回类型([refer this](https://stackoverflow.com/questions/14694852/can-overridden-methods-differ-in-return-type))。

执行方法重载时，参数列表应有所不同。参数列表应与方法覆盖中的相同。

覆盖需要继承，而无需重载。

##### 32、提问 retrofit 的adapter 使用了哪些设计模式

答：

##### 33、讲讲你对atomic & nonatomic的理解

答：nonatomic：非原子操作，决定编译器生成的setter getter是否是原子操作,不会为setter方法加锁。系统自动生成的 getter/setter 方法不一样。如果自己写 getter/setter，那 atomic/nonatomic/retain/assign/copy 这些关键字只起提示作用，写不写都一样。线程不安全,如有两个线程访问同一个属性，会出现无法预料的结果.

atomic和nonatomic用来决定编译器生成的getter和setter是否为原子操作。在多线程环境下，原子操作是必要的，否则有可能引起错误的结果。

对于atomic的属性，atomic表示多线程安全，atomic意为操作是原子的，系统生成的 getter/setter 会保证 get、set 操作的完整性，不受其他线程影响。比如，线程 A 的 getter 方法运行到一半，线程 B 调用了 setter：那么线程 A 的 getter 还是能得到一个完好无损的对象。意味着只有一个线程访问实例变量(生成的setter和getter方法是一个原子操作)而nonatomic就没有这个保证了。所以，nonatomic的速度要比atomic快;不过atomic可并不能保证线程安全。如果线程 A 调了 getter，与此同时线程 B 、线程 C 都调了 setter——那最后线程 A get 到的值，3种都有可能：可能是 B、C set 之前原始的值，也可能是 B set 的值，也可能是 C set 的值。同时，最终这个属性的值，可能是 B set 的值，也有可能是 C set 的值.

atomic是线程安全的,需要消耗大量的资源，至少在当前的存取器上是安全的。它是一个默认的特性，但是很少使用，因为比较影响效率,加了atomic，setter函数会变成下面这样： 

```objective-c
if (property != newValue) {
　　[property release];
　　property = [newValue retain];
}
```

假设有一个 atomic 的属性 "name"，如果线程 A 调[self setName:@"A"]，线程 B 调[self setName:@"B"]，线程 C 调[self name]，那么所有这些不同线程上的操作都将依次顺序执行——也就是说，如果一个线程正在执行 getter/setter，其他线程就得等待。因此，属性 name 是读/写安全的。

但是，如果有另一个线程 D 同时在调[name release]，那可能就会crash，因为 release 不受 getter/setter 操作的限制。也就是说，这个属性只能说是读/写安全的，但并不是线程安全的，因为别的线程还能进行读写之外的其他操作。线程安全需要开发者自己来保证。

如果 name 属性是 nonatomic 的，那么上面例子里的所有线程 A、B、C、D 都可以同时执行，可能导致无法预料的结果。如果是 atomic 的，那么 A、B、C 会串行，而 D 还是并行的。

```objectivec
//nonatomic如下解释：
//@property(nonatomic, retain) UITextField *userName;
//系统生成的代码如下：

- (UITextField *) userName {
    return userName;
}

- (void) setUserName:(UITextField *)userName_ {
    [userName_ retain];
    [userName release];
    userName = userName_;
}
//atomic如下解释：
//@property(retain) UITextField *userName;
//系统生成的代码如下：

- (UITextField *) userName {
    UITextField *retval = nil;
    @synchronized(self) {
        retval = [[userName retain] autorelease];
    }
    return retval;
}

- (void) setUserName:(UITextField *)userName_ {
    @synchronized(self) {
      [userName release];
      userName = [userName_ retain];
    }
}
// atomic 会加一个锁来保障线程安全，并且引用计数会 +1，来向调用者保证这个对象会一直存在。假如不这样做，如有另一个线程调 setter，可能会出现线程问题，导致引用计数降到0，原来那个对象就会被释放掉；

```

##### 34、被 weak 修饰的对象在被释放的时候会发生什么？是如何实现的？知道sideTable 么？里面的结构可以画出来么？

答：weak表示指向但不拥有该对象。其修饰的对象引用计数不会增加。无需手动设置，该对象会自行在内存中销毁。

__weak 修饰表明一种关系“非拥有关系”。弱引用，不决定对象的存亡。即使一个对象被持有无数个弱引用，只要没有强引用指向它，那么还是会被销毁。

若附有weak 修饰符的变量所引用的对象被废弃，则将nil赋值给该变量。
假设变量obj附加strong修饰符且对象被赋值。
{
  // 声明一个weak指针
  id weak obj1 = obj;
}
模拟编译器编译后的代码：
id obj1;
objc_initWeak(&obj1, obj);
objc_release(obj);
objc_destroyWeak(&obj1);
通过objc_initWeak 函数初始化附有weak修饰符的变量：
/* 编译器的模拟代码 /
id obj1;
obj1 = 0;
objc_storeWeak(&obj1, obj);
objc_storeWeak函数把第二参数的赋值对象的地址作为键值，将第一参数的附有__weak修饰符的变量的地址注册到weak表中。如果第二参数为0，则把变量的地址从weak表中删除。
weak 表与引用计数表相同，作为散列表被实现。如果使用weak表，将废弃对象的地址作为键值进行检索，就能高速地获取对应的附有weak修饰符的变量的地址。另外，由于一个对象可同时赋值给多个附有weak修饰符的变量中，所以对于一个键值，可注册多个变量的地址。
在变量作用域结束时通过 objc_destroyWeak函数释放该变量：
/ 编译器的模拟代码 /
objc_storeWeak(&obj1, 0);
释放对象时，废弃谁都不持有的对象的同时，程序的动作是怎样的呢？下面我们来跟踪观察。对象将通过objc_release函数释放。
（1）objc_release
（2）因为引用计数为0所以执行dealloc
（3）objc_rootDealloc
（4）object_dispose
（5）objc_destructInstance
（6）objc_clear_deallocating
对象被废弃时最后调用的objc_clear_deallocating函数的动作如下：
（1）从weak表中获取废弃对象的地址为键值的记录。
（2）将包含在记录中的所有附有weak修饰符变量的地址，赋值为nil。
（3）从weak表中删除该记录。
（4）从引用计数表中删除废弃对象的地址为键值的记录。
根据以上步骤，前面说的如果附有weak修饰符的变量所引用的对象被废弃，则将nil赋值给该变量这一功能即被实现。由此可知，如果大量使用附有weak修饰符的变量，则会消耗相应的CPU资源。良策是只在需要避免循环引用时使用weak修饰符。
以上就是一个 weak 指针从初始化到被置为nil 的全过程，在写这篇文章之前我一直有疑惑，如果是objc_clear_deallocating函数进行了weak指针置为nil的操作，那objc_destroyWeak函数是干嘛的？我反复推敲，想起来文中早已说明了用途 “在变量作用域结束时通过 objc_destroyWeak函数释放该变量”，也就是说objc_destroyWeak函数是在weak指针被置为nil后，用来将weak释放掉。
weak立即释放对象
使用weak修饰符时，以下源代码会引起编译器警告。
{
  id weak obj = [[NSObject alloc] init];
}
因为该源代码将自己生成并持有的对象赋值给附有weak修饰符的变量中，所以自己不能持有该对象，这时会被释放并被废弃，因此会引起编译器警告：warning: Assigning retained object to weak variable; object will be released after assignment
编译器如何处理该源代码呢?
/编译器的模拟代码/
id obj；
id tmp = objc_msgSend(NSObject, @selector(alloc));
objc_msgSend(tmp, @selector(init));
objc_initweak(&obj, tmp);
objc_destroyWeak(&object)；
虽然自己生成并持有的对象通过objc_initWeak函数被赋值给附有weak修饰符的变量中，但编译器判断其没有持有者，故该对象立即通过objc_release函数被释放和废弃。
这样一来，nil就会被赋值给引用废弃对象的附有weak修饰符的变量中。下面我们通过NSLog函数来验证一下:
id weak obj= [[NSObject alloc] init];
NSLog(@"obj=%@"，obj);
以下为该源代码的输出结果，其中用%@输出nil。
obj=（null）
如上所述，以下源代码会引起编译器警告。
id weak obj= [[NSObject alloc] init];
这是由于编译器判断生成并持有的对象不能继续持有。附有unsafe_unretained修饰符的变量又如何呢?
id unsafe_unretained obj=[[NSObject alloc] init];
与_weak修饰符完全相同，编译器判断生成并持有的对象不能继续持有，从而发出警告：
Assigning retained object to unsafe_unretained variable; object will be released after assignment
该源代码通过编译器转换为以下形式。
/编译器的模拟代码/
id obj = objc_msgSend( NSObject, @selector(alloc))；
objc_msgSend(obj,@selector(init))；
objc_release(obj);
objc_release函数立即释放了生成并持有的对象，这样该对象的悬垂指针被赋值给变量obj中。
那么如果最初不赋值变量又会如何呢？下面的源代码在MRC时必定会发生内存泄漏。
[[NSObject alloc] init]；
由于源代码不使用返回值的对象，所以编译器发出警告。
warning：expression result unused [-Wunused-value]
 [[NSObject alloc] init]；
可像下面这样通过向void型转换来避免发生警告。
（void)[[NSObject alloc] init]；
不管是否转换为void，该源代码都会转换为以下形式
/ 编译器的模拟代码 */
id tmp = objc_msgSend( NSObject, @selector(alloc));
objc_msgSend(tmp, @selector(init))；
objc_release(tmp)；
在调用了生成并持有对象的实例方法后，该对象被释放。看来“由编译器进行内存管理”这句话应该是正确的。
Runtime维护了一个weak表，用于存储指向某个对象的所有weak指针。weak表其实是一个hash（哈希）表，key是所指对象的地址，value是weak指针的地址（这个地址的值是所指对象的地址）数组。为什么value是数组？因为一个对象可能被多个弱引用指针指向。
weak 的实现原理可以概括一下三步：
1、初始化时：runtime会调用objc_initWeak函数，初始化一个新的weak指针指向对象的地址。
2、添加引用时：objc_initWeak函数会调用objc_storeWeak() 函数，objc_storeWeak() 的作用是更新指针指向，创建对应的弱引用表。
3、释放时，调用clearDeallocating函数。clearDeallocating函数首先根据对象地址获取所有weak指针地址的数组，然后遍历这个数组把其中的数据设为nil，最后把这个entry从weak表中删除，最后清理对象的记录。
浅谈iOS之weak底层实现原理
1、Runtime会维护一个Weak表,用于维护指向对象的所有weak指针。Weak表是一个哈希表,其key为所指对象的指针,vaue为Weak指针的地址数组。
具体过程如下
1、初始化时: runtime会调用 objc_initWeak函数初始化一个新的weak指针指向对象的地址。
2、添加引用时: objc_initWeak函数会调用objc storeWeak(0函数,更新指针指向,创建对应的弱引用表。
3、释放时,调用 clearDeallocating函数clearDeallocating函数首先根据对象地址获取所有Weak指针地址的数组,然后遍历这个数组把其中的数据设为n,最后把这个enty从Weak表中删除,最后清理对象的记录。
当weak引用指向的对象被释放时，又是如何去处理weak指针的呢？当释放对象时，其基本流程如下：
1、   1、调用objc_release
2、   2、因为对象的引用计数为0，所以执行dealloc
3、   3、在dealloc中，调用了objc_rootDealloc函数
4、   4、在_objc_rootDealloc中，调用了object_dispose函数
5、   5、调用objc_destructInstance
6、   6、最后调用objc_clear_deallocating,详细过程如下：
7、     a. 从weak表中获取废弃对象的地址为键值的记录
8、     b. 将包含在记录中的所有附有 weak修饰符变量的地址，赋值为  nil
9、     c. 将weak表中该记录删除
  d. 从引用计数表中删除废弃对象的地址为键值的记录

##### 35、block 用什么修饰？strong 可以？

答：

##### 36、block 为什么能够捕获外界变量？__block做了什么事？

答：_NSConcreteStackBlock，_NSConcreteMallocBlock，_NSConcreteGlobalBlock

##### 37、谈谈你对事件的传递链和响应链的理解

答：苹果注册了一个 Source1 (基于 mach port 的) 用来接收系统事件，其回调函数为 __IOHIDEventSystemClientQueueCallback()。
当一个硬件事件(触摸/锁屏/摇晃等)发生后，首先由 IOKit.framework 生成一个 IOHIDEvent 事件并由 SpringBoard 接收。这个过程的详细情况可以参考这里。SpringBoard 只接收按键(锁屏/静音等)，触摸，加速，接近传感器等几种 Event，随后用 mach port 转发给需要的App进程。随后苹果注册的那个 Source1 就会触发回调，并调用 _UIApplicationHandleEventQueue() 进行应用内部的分发。
_UIApplicationHandleEventQueue() 会把 IOHIDEvent 处理并包装成 UIEvent 进行处理或分发，其中包括识别 UIGesture/处理屏幕旋转/发送给 UIWindow 等。通常事件比如 UIButton 点击、touchesBegin/Move/End/Cancel 事件都是在这个回调中完成的。
手势识别
当上面的 _UIApplicationHandleEventQueue() 识别了一个手势时，其首先会调用 Cancel 将当前的 touchesBegin/Move/End 系列回调打断。随后系统将对应的 UIGestureRecognizer 标记为待处理。
苹果注册了一个 Observer 监测 BeforeWaiting (Loop即将进入休眠) 事件，这个Observer的回调函数是 _UIGestureRecognizerUpdateObserver()，其内部会获取所有刚被标记为待处理的 GestureRecognizer，并执行GestureRecognizer的回调。
当有 UIGestureRecognizer 的变化(创建/销毁/状态改变)时，这个回调都会进行相应处理。

##### 38、定时器

答：NSTimer 其实就是 CFRunLoopTimerRef，他们之间是 toll-free bridged 的。一个 NSTimer 注册到 RunLoop 后，RunLoop 会为其重复的时间点注册好事件。例如 10:00, 10:10, 10:20 这几个时间点。RunLoop为了节省资源，并不会在非常准确的时间点回调这个Timer。Timer 有个属性叫做 Tolerance (宽容度)，标示了当时间点到后，容许有多少最大误差。
如果某个时间点被错过了，例如执行了一个很长的任务，则那个时间点的回调也会跳过去，不会延后执行。就比如等公交，如果 10:10 时我忙着玩手机错过了那个点的公交，那我只能等 10:20 这一趟了。
CADisplayLink 是一个和屏幕刷新率一致的定时器（但实际实现原理更复杂，和 NSTimer 并不一样，其内部实际是操作了一个 Source）。如果在两次屏幕刷新之间执行了一个长任务，那其中就会有一帧被跳过去（和 NSTimer 相似），造成界面卡顿的感觉。在快速滑动TableView时，即使一帧的卡顿也会让用户有所察觉。Facebook 开源的 AsyncDisplayLink 就是为了解决界面卡顿的问题，其内部也用到了 RunLoop，这个稍后我会再单独写一页博客来分析

##### 39、TCP/IP协议在哪一层？

答：TCP/IP通讯协议采用了4层的层级结构，分别为：应用层，传输层，网络层，接口层；

1、传输层:在此层中，它提供了节点间的数据传送，应用程序之间的通信服务，主要功能是数据格式化、数据确认和丢失重传等。如传输控制协议(TCP)、用户数据报协议(UDP)等，TCP和UDP给数据包加入传输数据并把它传输到下一层中，这一层负责传送数据，并且确定数据已被送达并接收；
2、网络层:负责提供基本的数据封包传送功能，让每一块数据包都能够到达目的主机(但不检查是否被正确接收)，如网际协议(IP)；
3、[接口层]:接收IP数据报并进行传输，从网络上接收物理帧，抽取IP数据报转交给下一层，对实际的网络媒体的管理，定义如何使用实际网络(如Ethernet、Serial Line等)来传送数据；

##### 40、苹果是如何实现 autoreleasepool的？

答：App启动后，苹果在主线程 RunLoop 里注册了两个 Observer，其回调都是 _wrapRunLoopWithAutoreleasePoolHandler()。
第一个 Observer 监视的事件是 Entry(即将进入Loop)，其回调内会调用 _objc_autoreleasePoolPush() 创建自动释放池。其 order 是-2147483647，优先级最高，保证创建释放池发生在其他所有回调之前。
第二个 Observer 监视了两个事件： BeforeWaiting(准备进入休眠) 时调用_objc_autoreleasePoolPop() 和 _objc_autoreleasePoolPush() 释放旧的池并创建新池；Exit(即将退出Loop) 时调用 _objc_autoreleasePoolPop() 来释放自动释放池。这个 Observer 的 order 是 2147483647，优先级最低，保证其释放池子发生在其他所有回调之后。
在主线程执行的代码，通常是写在诸如事件回调、Timer回调内的。这些回调会被 RunLoop 创建好的 AutoreleasePool 环绕着，所以不会出现内存泄漏，开发者也不必显示创建 Pool 了。

##### 41、谈谈你对 FRP (函数响应式) 的理解，延伸一下 RxSwift 或者 RAC！

答：

#### Runtime相关问题

##### 42、 什么是 isa，isa 的作用是什么？

答：

##### 43、一个实例对象的isa 指向什么？类对象指向什么？元类isa 指向什么？

答：

##### 44、objc中类方法和实例方法有什么本质区别和联系？

答：

##### 45、 load 和 initialize 的区别？

答：

##### 46、 _objc_msgForward 函数是做什么的？直接调用会发生什么问题？

答：

##### 47、 简述下 Objective-C 中调用方法的过程

答：

##### 48、能否想象编译后得到的类中增加实例变量？能否向运行时创建的类中添加实例变量？为什么？

答：

##### 49、谈谈你对切面编程的理解

答：

#### 网络&多线程问题

##### 50、 HTTP的缺陷是什么？

答：

##### 50、谈谈三次握手，四次挥手！为什么是三次握手，四次挥手？

答：

##### 51、socket 连接和 Http 连接的区别、socket的原理分析

答：网络七层：物理层、链路层、网络层、传输层、应用层

TCP 三次握手：握手过程中并不传输数据，在握手后服务器与客户端才开始传输数据，理想状态下，TCP 连接一旦建立，在通讯双方中的任何一方主动断开连接之前 TCP 连接会一直保持下去。

Socket 是对 TCP/IP 协议的封装，Socket 只是个接口不是协议，通过 Socket 我们才能使用 TCP/IP 协议，除了 TCP，也可以使用 UDP 协议来传递数据。

创建 Socket 连接的时候，可以指定传输层协议，可以是 TCP 或者 UDP，当用 TCP 连接，该Socket就是个TCP连接，反之。

***Socket\*** **原理**

*Socket* 连接*,*至少需要一对套接字，分为 *clientSocket*，*serverSocket* 连接分为*3*个步骤*:*

*(1)* 服务器监听*:*服务器并不定位具体客户端的套接字，而是时刻处于监听状态；

*(2)* 客户端请求*:*客户端的套接字要描述它要连接的服务器的套接字，提供地址和端口号，然后向服务器套接字提出连接请求；

*(3)* 连接确认*:*当服务器套接字收到客户端套接字发来的请求后，就响应客户端套接字的请求*,*并建立一个新的线程*,*把服务器端的套接字的描述发给客户端。一旦客户端确认了此描述，就正式建立连接。而服务器套接字继续处于监听状态，继续接收其他客户端套接字的连接请求*.*

**Socket****为长连接：**通常情况下Socket 连接就是 TCP 连接，因此 Socket 连接一旦建立,通讯双方开始互发数据内容，直到双方断开连接。在实际应用中，由于网络节点过多，在传输过程中，会被节点断开连接，因此要通过轮询高速网络，该节点处于活跃状态。

 

很多情况下，都是需要服务器端向客户端主动推送数据，保持客户端与服务端的实时同步。

若双方是 Socket 连接，可以由服务器直接向客户端发送数据。

若双方是 HTTP 连接，则服务器需要等客户端发送请求后，才能将数据回传给客户端。

因此，客户端定时向服务器端发送请求，不仅可以保持在线，同时也询问服务器是否有新数据，如果有就将数据传给客户端。

普通报头用于所有的请求和响应消息，但并不用于被传输的实体，只用于传输的消息。比如：

Cache-Control：用于指定缓存指令，缓存指令是单向的(响应中出现的缓存指令在请求中未必会出现)，且是独立的(一个消息的缓存指令不会影响另一个消息处理的缓存机制)；

Date：表示消息产生的日期和时间；

Connection：允许发送指定连接的选项，例如指定连接是连续，或者指定“close”选项，通知服务器在响应完成后关闭连接。

请求报头允许客户端向服务器端传递请求的附加信息以及客户端自身的信息。常用的请求报头如下：

Host：指定被请求资源的 Internet 主机和端口号，它通常是从HTTP URL中提取出来的；

User-Agent：允许客户端将它的操作系统、浏览器和其它属性告诉服务器；

Accept：指定客户端接受哪些类型的信息，eg:Accept:image/gif，表明客户端希望接受GIF图象格式的资源；

Accept-Charset：指定客户端接受的字符集，缺省是任何字符集都可以接受；

Accept-Encoding：指定可接受的内容编码，缺省是各种内容编码都可以接受；

Authorization：证明客户端有权查看某个资源，当浏览器访问一个页面，如果收到服务器的响应代码为401(未授权)，可以发送一个包含Authorization请求报头域的请求，要求服务器对其进行验证。

实体报头定义了关于实体正文（eg：有无实体正文）和请求所标识的资源的元信息。常用的实体报头如下：

Allow：GET,POST

Content-Encoding：文档的编码（Encode）方法，例如：gzip；

Content-Language：内容的语言类型，例如：zh-cn；

Content-Length：表示内容长度，eg：80

##### 52、HTTPS，安全层除了SSL还有，最新的？参数握手时首先客户端要发什么额外参数

答：

##### 53、什么时候POP网络，有了 Alamofire 封装网络 URLSession为什么还要用Moya ？

答：

##### 54、如何实现 dispatch_once

答：

##### 55、能否写一个读写锁？谈谈具体的分析

答：

##### 56、什么时候会出现死锁？如何避免？

答：

##### 57、 有哪几种锁？各自的原理？它们之间的区别是什么？

答：

#### 数据结构问题

##### 58、 数据结构的存储一般常用的有几种？各有什么特点？

答：

##### 59、 集合结构 线性结构 树形结构 图形结构

答：

##### 60、 单向链表 双向链表 循环链表 

答：

####  数组和链表区别 

##### 61、 堆、栈和队列

答：

##### 62、 输入一棵二叉树的根结点，求该树的深度？

答：

##### 63、 输入一棵二叉树的根结点，判断该树是不是平衡二叉树？

答：

#### 算法问题

##### 64、 时间复杂度

答：

##### 65、 空间复杂度

答：

##### 66、 常用的排序算法

答：

##### 67、 字符串反转

答：

##### 68、 链表反转（头差法）

答：

##### 69、 有序数组合并

答：

##### 70、 查找第一个只出现一次的字符（Hash查找）

答：

##### 71、 查找两个子视图的共同父视图

答：

##### 72、 无序数组中的中位数(快排思想)

答：

##### 73、 给定一个整数数组和一个目标值，找出数组中和为目标值的两个数。

答：

#### 架构设计问题

##### 74、 设计模式是为了解决什么问题的？

答：

##### 75、 看过哪些第三方框架的源码，它们是怎么设计的？

答：

##### 76、 可以说几个重构的技巧么？你觉得重构适合什么时候来做？

答：

##### 77、 开发中常用架构设计模式你怎么选型?

答：

##### 78、 你是如何组件化解耦的？

答：

#### 性能优化问题

##### 79、 tableView 有什么好的性能优化方案？

答：

##### 80、 界面卡顿和检测你都是怎么处理？

答：

##### 81、 谈谈你对离屏渲染的理解？

答：

##### 82、 如何降低APP包的大小

答：

##### 83、 日常如何检查内存泄露？

答：

##### 84、 APP启动时间应从哪些方面优化？

答：

##### 85、堆栈解释一下，内存分配的方向？为什么堆低到高？栈高到低？

答：

##### 86、如何自定义一个三角形按钮？点击区域怎么做？

答：

##### 87、解决循环引用的几种方式？重点问__block，局部变量，全局变量，静态变量，可变数组，哪些需要__block修饰，哪些不需要？为什么

##### __block捕获的变量，为什么可以修改变量值（forwarding指针）？

答：

##### 88、自动释放池原理讲一下？为什么要用双向链表？

答：

##### 89、性能优化你了解吗？说说你了解的方面，介绍一下

答：

##### 90、UIKit这个框架，在程序运行的时候是放在内存中什么区域的？

答：

##### 91、category与extension，category可以添加私有方法吗？category为什么不能添加成员变量？为什么能添加方法？

答：

##### 92、category重写了类的方法后，调用该方法，会发生什么？为什么走到category里面了？

答：

##### 93、load方法和initialize方法介绍一下调用时机

答：

##### 94、block嵌套block（第一个block外部已经用了weakSelf，内部用strongSelf重新强引用了），想在第二个block内部用strongSelf，怎么办？

答：

##### 95、weak的原理介绍一下

答：

##### 96、为什么block内部对weakSelf使用strongSelf没修饰有造成强引用？

答：

##### 97、为什么要用strongSelf（防止block执行时self被释放），那你举例说一个self被释放的场景

答：

##### 98、哈希冲突解决办法介绍一下

答：

##### 99、常见的内存泄露有哪几种？分别怎么解决

答：

##### 100、手写一个属性在mrc下的set方法

答：

##### 101、@dynamic的使用场景

答：

##### 103、RSA加密和MD5加密你了解吗（不了解）？

答：

##### 104、Charles可以抓https的原理

答：

##### 105、MVVM是如何进行双向绑定的？在viewmodel内部可以处理哪些事情？

答：

##### 106、UILable自适应是怎么实现的？

答：

##### 107、介绍一下多线程，GCD的高级用法，NSoperation重写要注意什么？

答：实际上 RunLoop 底层也会用到 GCD 的东西，~~比如 RunLoop 是用 dispatch_source_t 实现的 Timer~~（评论中有人提醒，NSTimer 是用了 XNU 内核的 mk_timer，我也仔细调试了一下，发现 NSTimer 确实是由 mk_timer 驱动，而非 GCD 驱动的）。但同时 GCD 提供的某些接口也用到了 RunLoop， 例如 dispatch_async()。
当调用 dispatch_async(dispatch_get_main_queue(), block) 时，libDispatch 会向主线程的 RunLoop 发送消息，RunLoop会被唤醒，并从消息中取得这个 block，并在回调 **CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE**() 里执行这个 block。但这个逻辑仅限于 dispatch 到主线程，dispatch 到其他线程仍然是由 libDispatch 处理的。

##### 108、OC如何实现多继承？

答：

##### 109、讲讲组件化方面的东西，你认为什么是组件化，哪些可以实现组件化

答：

##### 110、调用runtime registerClass方法，创建出来了几个类？

答：实例对象、类对象、runtime，
成员变量保存在class_rw_t->class_ro_t 实例对象 本质 objc结构题就是一个对象ISA从objc_object继承过来的 
rw什么时候创建的 ：运行时创建的 copy进来 可读写的 属性 方法 协议 脏内存 数据昂贵  一直存在
ro什么时候创建的 ：编译的时候创建的 只读状态 成员变量 可移除 保持清洁的数据 永远不可改变 
ISA 实例对象 类对象 指向元类 根元类 指向自己
SEL 方法编号
IMP 函数指针 保存了方法的地址
Method 参数类型描述字符串
设计元类的原类 (复用消息通道，类方法也可以放在Class里，但发送消息时，需要增加一个参数) 是实例 类对象方法
发送消息 消息发送 消息转发 runtime 是查找的元类 类方法

##### 111、iOS内存区域介绍一下

答：

#### 112、悬垂指针和野指针的区别是什么

答：*悬垂指针*之前指向的是有效的地址,但是现在不是这样了,通常是那个内存地址被释放掉了. 而*野指针是*没有被正确初始化,它指向了内存中随机的位置。

#### 113、flutter  事件循环

答：

#### 114、flutter 是单线程还是多线程

答：单线程

#### 115、Stream  feature  的区别

答：

#### 116、isolate有了解吗

答：

#### 117、Stateless Widget和Stateful Widget区别

答：

#### 118、StatefulWidget 的生命周期

答：

#### 119、介绍下widget、state、context

答：

#### 120、Widget和element和RenderObject之间的关系

答：

#### 121、flutter是如何实现多任务并行的，谈谈Isolate理解

答：

#### 面试题巩固

##### 122、对GCD的使用有哪些了解？对NSLock的原理分析

答：

##### 123、git commit 原理，链表数组

答：

##### 124、如何保证单例的唯一？

答：

##### 125、原子性跟非原子性的分析

答：

##### 126、

答：

##### 127、KVO如何触发事件响应链的

答：

##### 128、依赖的优先级,如何判断哪一个View是最佳响应者？

答：

##### 129、事件响应链的过程中soure0跟source1的分析

答：

##### 130、

答：

##### 131、

答：

#### 字节面试

##### 1、怎么在汇编实现方法调用时间的计算？

答：

##### 2、fishHook的原理是什么？

答：将指向系统方法（外部函数）的符号重新进行绑定指向内部的函数。把系统方法与自己定义的方法进行了交换。C的内部函数修改不了，自定义的函数修改不了，只能修改 Mach-O 外部的函数

##### 3、pod install 的内部怎么实现的？

答：

##### 4、xcode的space，project，target的区别和联系是什么？

答：

##### 5、category怎么实现一个weak属性？

答：

##### 6、xcode的编译流程做了啥？

答：https://www.jianshu.com/p/14612abdeb26

https://objccn.io/issue-6-1/

> 推荐一些 Code Review 工具(https://www.zhihu.com/question/41089988/answer/544543949)
>
> 1、**[Crucible](https://link.zhihu.com/?target=https%3A//www.atlassian.com/software/crucible)：**Atlassian 内部代码审查工具；
>
> 2、**[Gerrit](https://link.zhihu.com/?target=https%3A//www.gerritcodereview.com/)：**Google 开源的 git 代码审查工具；
>
> 3、**[GitHub](https://link.zhihu.com/?target=https%3A//github.com/)：**程序员应该很熟悉了，上面的 "Pull Request" 在代码审查这里很好用；
>
> 4、**[LGTM](https://link.zhihu.com/?target=https%3A//lgtm.com/)：**可用于 GitHub 和 Bitbucket 的 PR 代码安全漏洞和代码质量审查辅助工具；
>
> 5、**[Phabricator](https://link.zhihu.com/?target=https%3A//www.phacility.com/phabricator/)：**Facebook 开源的 git/mercurial/svn 代码审查工具； 
>
> 6、**[PullRequest](https://link.zhihu.com/?target=https%3A//www.pullrequest.com/)：**GitHub pull requests 代码审查辅助工具；
>
> 7、**[Pull Reminders](https://link.zhihu.com/?target=https%3A//pullreminders.com/)：**GitHub 上有 PR 需要你审核，该插件自动通过 Slack 提醒你；
>
> 8、**[Reviewable](https://link.zhihu.com/?target=https%3A//reviewable.io/)：**基于 GitHub pull requests 的代码审查辅助工具；
>
> 9、**[Sider](https://link.zhihu.com/?target=https%3A//sider.review/)：**GitHub 自动代码审查辅助工具；
>
> 10、**[Upsource](https://link.zhihu.com/?target=https%3A//www.jetbrains.com/upsource/)：**JetBrain 内部部署的 git/mercurial/perforce/svn 代码审查工具。





  
