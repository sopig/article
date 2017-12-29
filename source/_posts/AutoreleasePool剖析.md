title: AutoreleasePool剖析
date: 2015-04-19 22:06:06
tags: iOS AutoreleasePool
---

##概述
MRC时代，我们手动管理对象的创建与销毁，OC内部是通过引用计数的机制去管理对象内存。一般我们创建一个对象需要手动retain与release，有时候我们不使用release，而是使用autorelease，当时模糊的概念也就是说这个对象会在稍后某个时候自动释放，无须手动调用release，看上去很神奇，可是一想疑惑很多。到底什么时候释放，它是如何释放的？

ARC时代之后，我们逐渐抛弃了retain和release等内存管理的操作，把编程的重心放在了代码逻辑上，语言的进化让每个人更专注于自己要实现的功能而更少地关注语言底层的实现细节，我们不用去处理对象的释放问题，那这时候MRC时代的autorelease功能是不是就可以放弃了？

看似ARC为我们做了一切，实际上苹果只是在编译层面帮我们转换成类似MRC的代码，底层内存管理机制依旧是原来的引用计数那一套。

##AutoreleasePool原理
为了显示调用autoreleasepool，苹果引入了一个block
```objc
@autoreleasepool {
    NSString *str = [NSString stringWithFormat:@"yzj"];
    
    //创建的对象都会被丢入pool中
}
```
在该block内创建的对象（非alloc、new、copy、mutableCopy）都会自动调用autorelease方法加入autoreleasepool，在block结束时autoreleasepool自动对丢入pool中的对象执行release方法，使引用计数减1。

这里有些人会有疑惑了，我要这干嘛呢，普通的{}作用域不也可以干这件事么？{}作用域内创建的变量ARC都会帮我们在{}结尾处插入变量调用的release方法不是么？
```objc
{
    id a = [NSObject new];
    //[a release];
}
```

看上去类似，但是事实上内部机制完全不同。
ARC时代苹果优化了对象创建时的返回值策略，大体上会把下面这段代码改写成另一段代码：
```objc
Jacob *jb = [Jacob new];
```

```objc
id imp = objc_retainAutoreleasedReturnValue([Jacob new]);
Jacob *jb = imp;
```
查看objc4对这个方法的定义如下：
```objc
id 
objc_retainAutoreleaseReturnValue(id obj)
{
    return objc_autoreleaseReturnValue(objc_retain(obj));
}

id 
objc_autoreleaseReturnValue(id obj)
{
    if (fastAutoreleaseForReturn(obj)) return obj;

    return objc_autorelease(obj);
}

id objc_autorelease(id obj) { return [obj autorelease]; }
```

我们先来剖析autorelease这个方法：
```objc
- (id)autorelease {
    return ((id)self)->rootAutorelease();
}

inline id 
objc_object::rootAutorelease()
{
    assert(!UseGC);

    if (isTaggedPointer()) return (id)this;
    if (fastAutoreleaseForReturn((id)this)) return (id)this;

    return rootAutorelease2();
}

__attribute__((noinline,used))
id 
objc_object::rootAutorelease2()
{
    assert(!isTaggedPointer());
    return AutoreleasePoolPage::autorelease((id)this);
}

static inline id autorelease(id obj)
{
    assert(obj);
    assert(!obj->isTaggedPointer());
    id *dest __unused = autoreleaseFast(obj);
    assert(!dest  ||  *dest == obj);
    return obj;
}
```

忽略之前的一堆判断条件，最终我们跳转到这里，这里涉及到AutoreleasePoolPage的概念，稍后会详细说明
```objc
static inline id *autoreleaseFast(id obj)
{
    AutoreleasePoolPage *page = hotPage();
    if (page && !page->full()) {
        return page->add(obj);
    } else if (page) {
        return autoreleaseFullPage(obj, page);
    } else {
        return autoreleaseNoPage(obj);
    }
}
```
这几个if-else调用的语句最终都会走一个add方法：
```objc
id *add(id obj)
{
    assert(!full());
    unprotect();
    id *ret = next;  // faster than `return next-1` because of aliasing
    *next++ = obj;
    protect();
    return ret;
}
```
add方法就是把该对象的指针加入poolpage中，也就是我们理解的把对象加入到autoreleasepool中，那autoreleasepoolpage又是什么鬼？
我们来看它的数据结构：
```objc
class AutoreleasePoolPage 
{
#define POOL_SENTINEL nil//哨兵对象
    static pthread_key_t const key = AUTORELEASE_POOL_KEY;//TLS用的数组KEY
    static uint8_t const SCRIBBLE = 0xA3;  // 0xA3A3A3A3 after releasing
    static size_t const SIZE = 
#if PROTECT_AUTORELEASEPOOL
        PAGE_MAX_SIZE;  // must be multiple of vm page size
#else
        PAGE_MAX_SIZE;  // size and alignment, power of 2
#endif//page的大小
    static size_t const COUNT = SIZE / sizeof(id);

    magic_t const magic;//校验用的黑魔法
    id *next;//当前page中最新对象的下一个位置
    pthread_t const thread;//当前线程
    AutoreleasePoolPage * const parent;//前一个page
    AutoreleasePoolPage *child;//后一个page
    uint32_t const depth;//page的index
    uint32_t hiwat;//我也不知道这是什么鬼
}
```
page是一个双链表结构，runtime自己维护这个链表，当往page中插入对象时，会直接插入next的位置，next往后移一位，如果当前page插入对象已满，则新建一个page，index+1，做校验等操作后继续插入新的对象。

我们把这段代码用编译器重写后看到如下代码
```objc
@autoreleasepool {
}
```

```objc
struct __AtAutoreleasePool {
  __AtAutoreleasePool() {atautoreleasepoolobj = objc_autoreleasePoolPush();}
  ~__AtAutoreleasePool() {objc_autoreleasePoolPop(atautoreleasepoolobj);}
  void * atautoreleasepoolobj;
};

/* @autoreleasepool */ { 
    __AtAutoreleasePool __autoreleasepool; 
}
```
相当于在每个autoreleasepool开始和结尾处分别执行了push和pop操作，我们看源代码可以知道，push操作在page中插入一个哨兵对象（nil），当pop的时候，从高地址往低地址对每一个page中的对象执行release操作，直到遇到nil，也就是哨兵对象。这种机制可以实现autoreleasepool的嵌套结构，内层的pool可以不断创建，释放，而不影响外层的pool。从这我们也大致理解了autoreleasepool释放对象的原理。

最后提两点：
1.每一个RunLoop循环开始时会push一个autoreleasepool，结束时会pop一下
根据实验可以得出：viewdidload,viewwillappear在一个RunLoop中，viewdidappear在下一个RunLoop中，所以可以解释为什么viewdidload中丢入pool中的对象可以在viewwillappear中拿到，但在viewdidappear中却拿不到。
2.数组的这个遍历方法内部会有一个autoreleasepool，每次遍历都会释放当前循环的所有临时变量，内存占用小，所以推荐使用
```objc
[NSArray enumerate...^{

}];
```
相当于：
```objc
[NSArray enumerate...^{
@autoreleasepool{
//inset code
}
}];
```


