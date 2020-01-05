# hook msgsend

**为什么会有hook `objc_msgsend`这个需求：**

统计每个方法的耗时

在OC中做方法拦截都很熟悉`Method Swizzle` 

`objc_msgsend`和OC中做方法交换的区别是：**这是一个C函数**

OC中之所以会有`Method Swizzle`是因为OC是一门**动态语言**。而C语言的函数不能使用`Method Swizzle`也正是因为C语言是**静态语言**

## 什么是动态语言，什么是静态语言

很多资料都是这样介绍静态语言和动态语言

> *动态语言：函数的调用是在运行时才会确定*
>
> *静态语言：函数的地址在编译时就确定*

* 动态语言

	对于动态语言(如OC)，我们比较熟悉，当代码中`[obj methodA]`这样调用一个方法时，在编译后其实只是调用了`objc_msgsend`函数。`methodA`只是作为这个函数的参数。

	只有当运行的时候才会根据`methodA`这个方法名去方法名和方法地址的对应表中查找函数地址的具体位置然后去执行这个方法。

	所以在OC中调用了一个只在.h中声明的方法，而没有在.m中对方法进行实现。在编译的时候不会报错。

	正是由于这个特性，我们只要在运行函数之前，在方法名和方法地址的对应表中将函数的地址进行替换。就能实现hook。

* 静态语言

	而对于静态语言C语言。当调用一个函数。如果这个函数只在.h中有声明，而在.c中没有函数实现，在编译时就会报错。 *<代码演示>*

	为什么会报错。因为静态语言在编译的时候就要知道函数的地址在什么位置。一个.m或.c文件，经过编译之后会生成一个.o文件。这个.o文件中放的就是汇编形式的代码。

	从汇编的角度看一下C的函数调用过程。简化汇编的调用过程的话就如下图所示

	![]()

	从上图可以看出来，C的函数调用，编译之后就已经确定执行到函数调用的位置时，就是要跳转到指定的函数地址位置去执行。

	**所以，对于C的函数，编译之后，函数地址已经写进在了二进制文件中，是没办法hook的。**

分析到这里，仿佛这个需求就终结了

**但是**，FaceBook开源了一个库。`fishHook`，这个库可以对C函数进行hook

## fishHook

先尝试体验一下这个库

实验一下用`fishHook`去Hook一下`NSLog` *代码演示*

```
@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    // 因为NSLog是懒加载的，所以，必须在调用了这个函数之后，在懒加载符号表里才会有指针和地址的对应关系
    NSLog(@"此时还没有被Hook");
    
    struct rebinding logReBind;
    logReBind.name = "NSLog"; // 被Hook的函数的函数名
    logReBind.replacement = userLog; // 新的函数的函数地址
    logReBind.replaced = (void *)&origin_log; // 将原来的函数地址放到这个指针中
    
    // 定义一个数组，用来放所有将进行函数hook的结构体
    struct rebinding rebs[] = {logReBind};
    
    // 进行hook操作
    rebind_symbols(rebs, 1); //第一个参数就是数组，第二个参数是数组长度
    
}

// 定义一个函数指针类型的变量 origin_log，这个指针用来存放原来函数的函数地址
static void(*origin_log)(NSString *format, ...);

// 定义一个自定义的Log函数
void userLog(NSString *format, ...) {
    // 在函数里面调用原来的函数进行输出
    origin_log(@"函数已经被替换了");
}

@end
```

实验一下用`fishHook`去Hook一个自定义的C函数。*代码演示*

通过实验发现`fishHook`可以实现hook 系统的C函数。但是同时发现fishHook无法hook自定义的C函数。

无法hook自定义的C函数很好理解，因为C函数是静态的。

**为什么可以hook系统的C函数。这就需要看一下C函数的加载过程。C函数的调用是因为App中的调用，那就从App的加载来找原因**

## App的加载过程

当我们将一个App从App Store下载安装下来，其实是将这个App的可执行文件存储到手机的硬盘中

![App下载示意图]()

当运行App的时候，是通过dyld(dynamic link editor 动态链接器)将这个可执行文件加载到内存中。

![运行app示意图]()

在App运行的时候会使用到动态库，我们知道动态库在内存中是只有一份，所有App共享的。

**那动态库是怎么加载的？**当App启动的时候DYLD会先从Mach-O的LoadCommands拿到所依赖的动态库，然后将动态库加载到内存。*（如果每次都从硬盘去读取动态库速度太慢，所以苹果搞了一个共享缓存库来提高加载速度）*

而且，我们的可执行文件里都是汇编代码，从上面静态语言函数调用的分析可以看到，汇编里面都是直接访问的内存地址。比如调用NSLog，就是直接跳转到NSLog的函数地址去执行指令的。

*到这里，猜一个问题：动态库每次加载到内存中是不是在固定的位置？*

不是的！苹果为了防止被逆向破解，采取了一种叫做**ASLR**的技术，翻译过来叫：**地址空间布局随机化**。这样每次加载可执行文件到内存的时候都是一个随机地址。这样就能防止通过固定地址去寻找到指定函数。

**那我们调用动态库中的方法，是怎么找到的动态库的位置和动态库中的函数的。**

### 动态库的链接

苹果为了解决Mach-O中访问动态库中的函数，采用了一个技术，叫做**PIC (位置代码独立)技术**。

当Mach-O中需要访问动态库中的函数时，会在Mach-O的__DATA段建立一个指针(就是一个8字节的变量，初始化时都是0)，这个指针将来用来指向动态库函数的地址

当App启动时候，DYLD会动态的将这个指针和系统库函数的地址进行绑定

*所以，所谓代码位置独立也就是说每一个动态库、每一个可执行文件的在内存中的位置都是独立的，互相之间不会有关联。当存在调用关系的时候，再通过指针进行地址绑定*

所以，正是因为这个PIC技术，让App在启动的时候存在了一个动态绑定的过程，

fishHook实现Hook系统的C函数的基本原理就是在动态绑定的时候将指针的地址进行修改。

到这里就可以解释为什么可以hook系统的函数，而不能hook我们自定义的C函数，因为自定义的C函数没有这个绑定的过程。

## fishHook是怎么找到的系统函数

从实验显示的输出结果我们看到了系统的C函数确实被我们已经替换了。

现在我们回到代码来看一下在这个过程中**内存地址的变化过程**

那么首先我们要先看一下被替换之前的NSLog这个指针指向的内存地址

要想看NSLog这个符号指向的内存地址，就先得要找到NSLog这个指针

**那怎么找到`NSLog`这个指针**

NSLog指针的位置 = Mach-O文件的偏移地址 + NSLog在懒加载符号表中的偏移地址。

接下来就是要找到这两个地址

问题1 ：怎么查看Mach-O文件的偏移地址。

在断点的时候，用`image list`就可以看到内存加载的所有可执行文件的地址。

![imageAddress.png](https://upload-images.jianshu.io/upload_images/1284329-8586cd15b487c870.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第二个问题：怎么查看NSLog在懒加载表中的偏移

用MachOView打开可执行文件，就可以看到懒加载符号表(`__DATA,_la_symbol_ptr`),在这个表中找到`NSLog`,就可以看到这个函数在表中的偏移量offset

![offset_in_symbol_table.png](https://upload-images.jianshu.io/upload_images/1284329-802c584300dbd73e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

两个地址现在都已经拿到，两个地址相加就是NSLog这个指针在内存中的地址。

然后用`x`(`examine`)命令就可以查看内存地址的值，也就是这个指针的值，即这个指针指向的地址。

 `x 0x104718000+0xC000`。

![address_in_RAM.png](https://upload-images.jianshu.io/upload_images/1284329-e4a97e60037ffdb2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

那计算出来的这个指针指向的地址是不是NSLog呢。对指针指向的地址用反汇编来验证一下。

这里会有一个疑问：为什么得到的打印出来的地址是`34 b7 f9 88 01 00 00 00 bc 07 f9 99 01 00 00 00` 而我们用的地址是`00 00 00 01 88 f9 b7 34`

第一个问题为什么要取8位？

因为上面说过一个指针变量占用的的大小就是8字节。所以我们只取前8位。

第二个问题为什么从第8位逆序向前取值？

因为iOS的ARM架构默认是小端模式，所以从第八位开始向前逐个取值。取出来的就是 `0x0000000188f9b734`

> 小端模式，即 数据的低字节放在内存的低地址中，简单理解就是反过来放数据了

`dis -s 0x0000000188f9b734`

![dis_result.png](https://upload-images.jianshu.io/upload_images/1284329-f6ed27d2b4332922.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从上面的图可以看出来，现在这个指针指向的地址是`Foundation`的`NSLog`函数的地址

**在`rebind_symbols`执行之后**

我们看一下NSLog这个指针的地址有没有发生改变。以及改变之后指向的函数是哪里

![替换之后指针指向的地址.png](https://upload-images.jianshu.io/upload_images/1284329-466690f6fd233553.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从上面的调试截图可以看到，NSLog这个指针存储的地址已经发生了改变。

原来它指向的是`0x188f9b734`,而当执行了`rebind_symbols`之后,它指向的地址已经变成了`0x10471d4f8`。

并且通过反编译这个地址也可以看到，这个地址是我们可执行文件中我们自定义的函数

到这里已经验证了已经将`__DATA`段的指向外部函数的指针指向了自定义的函数

**接下来有一个问题，我们只告诉fishHook我们要hook的函数的函数名，它是怎么通过这个函数名找到函数指针的。**

## 如何通过函数名找到指针



## `objc_msgSend`和`NSLog`的不同之处 

