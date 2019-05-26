### STL空间配置器介绍
1. 一般的C++内存配置操作和释放操作如下：
    ```
    class A{};
    A *pa=new A;
    delete pa;
    ```
    第二行和第三行虽然只有一句，但是完成了两个动作，new一个对象的时候两个动作是：先调用::operator new分配一个对象大小的内存，然后在这个内存上调用A::A()构造对象。同样，delete一个对象的时候两个动作是：先调用A::~A()析构掉对象，再调用::operator delete将对象所处的内存释放。
2. 但是在STL中，为了精密分工，STL里的allocator将这两个阶段分开，分别用4个函数来实现：  
（1）内存的配置：alloc::allocate();  
（2）对象的构造：::construct();
（3）对象的析构：::destroy();
（4）内存的释放：alloc::deallocate();  
其中construct()和destroy()定义在STL的库文件中。

### construct和destroy函数
1. 接受一个指针参数的destroy函数直接调用指针指向对象的析构函数即可，而接受两个迭代器参数的destroy函数判断迭代器类型是否有trival destructor(默认析构函数)，如果有的话就什么也不用做因为内置类型会自动析构，而没有trival destructor的话则要调用destroy()函数对两个迭代器之间的对象元素进行一个个析构。
2. construct相比destroy函数要简单很多，用placement new在p指针所指对象上创建一个对象，value是初始化对象的值。
    ```
    template<class T1,class T2>
    inline void construct(T1* p,const T2& value){
        new (p) T1(value);
    }
    ```

### alloc::allocate和alloc::deallocate函数
1. 这两个函数在<stl_alloc.h>头文件中，这个头文件中代码设计的原则如下：  
（1）向system heap要求空间  
（2）考虑多线程状态  
（3）考虑内存不足时的应变措施  
（4）考虑过多“小型区块”可能造成的内存碎片问题
2. 考虑到小型区块可能造成内存破碎的问题（即形成内存碎片），SGI STL设计了双层级配置器。第一层配置器直接使用malloc()和free()，第二层配置器则视情况采用不同的策略：但配置区块超过128bytes，调用第一级配置器，当配置区块小于128bytes时，采用复杂的memeory pool方式。
3. 第一级配置器 __malloc_alloc_template，流程如下：  
（1）我们通过allocate()申请内存，通过deallocate()来释放内存，通过reallocate()重新分配内存  
（2）当allocate()或reallocate()分配内存不足时会调用oom_malloc()或oom_remalloc()来处理  
（3）当oom_malloc()或oom_remalloc()还是没能分配到申请的内存时，会转如下两步中的一步：  
a. 调用用户自定义的内存分配不足处理函数（这个函数通过set_malloc_handler()来设定），然后继续申请内存！  
b. 如果用户未定义内存分配不足处理函数，程序就会抛出bad_alloc异常或是利用exit(1)终止程序。
4. 第二级配置器 __default_alloc_template，第二层配置器如何维护128bytes以下内存的配置呢？ SGI 第二层配置器定义了一个 free-lists,这个free-list是一个数组，如下图：
    ![](https://images0.cnblogs.com/i/566545/201404/291845299705479.png)  
    这数组的元素都是指针，用来指向16个链表的表头。这16个链表上面挂的都是可以用的内存块。只是不同链表中元素的内存块大小不一样，16个链表上分别挂着大小为8,16,24,32,40,48,56,64,72,80,88,96,104,112,120,128 bytes的小额区块，图如下：
    ![](https://images0.cnblogs.com/i/566545/201404/291901512525167.png)  
    寻找 16 个free lists中恰当的一个如果没有找到可用的free list，就需要准备填充free list了。
5. allocate时有两个函数我来提一下，一个是ROUND_UP()，这个是将要申请的内存字节数上调为8的倍数。因为我们free-lists中挂的内存块大小都是8的倍数嘛，这样才知道应该去找哪一个链表。另一个就是refill()。这个是在没找到可用的free list的时候调用，准备填充free lists.意思是：参考上图，假设我现在要申请大小为 56bytes 的内存空间，那么就会到free lists 的第 7 个元素所指的链表上去找。如果此时#7元素所指的链表为空怎么办？这个时候就要调用refill()函数向内存池申请N(一般为20个)个大小为56bytes的内存区块，然后挂到 #7 所指的链表上。这样，申请者就可以得到内存块了。
6. deallocate时使用链表的插入操作将待释放的内存块加到链表上。
