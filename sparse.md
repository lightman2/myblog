Sparse诞生于2004年，是由Linux之父开发的，目的就是提供一个静态检查代码的工具，从而减少Linux内核的隐患。起始，在Sparse之前已经有了一个不错的代码静态检查工具（SWAT），只不过这个工具不是免费软件，使用上有一些限制。所以Linus自己开发了一个静态检查工具。
版本: linux4.4.166
参考文档:Documentation/sparse.txt
Sparse通过gcc的扩展属性__attribute__以及自己定义的__context__来对代码进行静态检查。
__xxx双下划线开头的宏,表示编译器相关的一些属性设置
1	#ifdef __CHECKER__
2	# define __user     __attribute__((noderef, address_space(1)))
3	# define __kernel   __attribute__((address_space(0)))
4	# define __safe     __attribute__((safe))
5	# define __force    __attribute__((force))
6	# define __nocast   __attribute__((nocast))
7	# define __iomem    __attribute__((noderef, address_space(2)))
8	# define __must_hold(x) __attribute__((context(x,1,1)))
9	# define __acquires(x)  __attribute__((context(x,0,1)))
10	# define __releases(x)  __attribute__((context(x,1,0)))
11	# define __acquire(x)   __context__(x,1)
12	# define __release(x)   __context__(x,-1)
13	# define __cond_lock(x,c)   ((c) ? ({ __acquire(x); 1; }) : 0)
14	# define __percpu   __attribute__((noderef, address_space(3)))
15	# define __pmem     __attribute__((noderef, address_space(5)))
16	#ifdef CONFIG_SPARSE_RCU_POINTER
17	# define __rcu      __attribute__((noderef, address_space(4)))
18	#else
19	# define __rcu
20	#endif
21	extern void __chk_user_ptr(const volatile void __user *);
22	extern void __chk_io_ptr(const volatile void __iomem *);
23	#else
file: include/linux/compiler.h
1	#ifdef __CHECKER__
2	#define __bitwise__ __attribute__((bitwise))
3	#else
4	#define __bitwise__
5	#endif
6	#ifdef __CHECK_ENDIAN__
7	#define __bitwise __bitwise__
8	#else
9	#define __bitwise
10	#endif
file: tools/include/linux/types.h
宏名称	定义	说明
__bitwise	__attribute__((bitwise))	确保变量是相同的位方式(比如 bit-endian, little-endiandeng)
__user	__attribute__((noderef, address_space(1)))	指针地址必须在用户地址空间
__kernel	__attribute__((noderef, address_space(0)))	指针地址必须在内核地址空间
__iomem	__attribute__((noderef, address_space(2)))	指针地址必须在设备地址空间
__safe	__attribute__((safe))	变量可以为空
__force	__attribute__((force))	变量可以进行强制转换
__nocast	__attribute__((nocast))	参数类型与实际参数类型必须一致
__acquires(x)	__attribute__((context(x, 0, 1)))	参数x 在执行前引用计数必须是0,执行后,引用计数必须为1
__releases(x)	__attribute__((context(x, 1, 0)))	与__acquires(x)相反
__acquire(x)	__context__(x, 1)	参数x的引用计数+1
__release(x)	__context__(x, 1)	与__acquire(x)相反
__cond_lock(x,c)	((c) ? ({ __acquire(x); 1; }) : 0)	参数c 不为0时,引用计数 + 1, 并返回1
其中__acquires(x)和__releases(x)，__acquire(x)和__release(x)必须配对使用,都和锁有关，否则Sparse会发出警告

Sparse 在编译内核中的使用
1	make C=1 检查所有重新编译的代码
2	make C=2 检查所有代码, 不管是不是被重新编译
如果进行-Wbitwise的检查,需要定义#define __CHECK_ENDIAN__,可以通过CF进行传参

1	>  make C=2 CF="-D__CHECK_ENDIAN__"
2	>

示例
1	static int
2	fb_open(struct inode *inode, struct file *file)
3	__acquires(&info->lock)
4	__releases(&info->lock)
5	{
6	...
7	return 0;
8	}
在编译阶段检查锁,防止死锁.
