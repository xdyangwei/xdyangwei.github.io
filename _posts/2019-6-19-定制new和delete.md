### 了解new-handler的行为
1. C++内存管理的主要操作是operator new和operator delete，当operator new无法满足客户的内存需求时会调用new-handler。
2. 多线程下的内存管理，由于heap是一个可被该改动的全局性资源，调用内存管理操作符可能很容易导致管理heap的数据结构内容败坏。
3. 当operator new抛出异常以反映一个未获满足的内存需求之前，它会先调用一个客户指定的错误处理函数，也就是new-handler。为了指定这个函数，我们可以使用set_new_handler这个函数。
4. 一个设计良好的new-handler函数必须做以下事情：  
（1）让更多内存可被使用，比如一开始分配一大块内存当new handler第一次被调用时将它们释还给程序使用  
（2）安装另一个new handler，让能取得更多内存的handler来代替自己（使用set_new_handler）即可  
（3）卸装new handler，将null指针传递给set_new_handler，一旦没有安装任何new-handler，operator new会在内存分配不成功时抛出异常。  
（4）抛出bad_alloc的异常  
（5）不返回，通常调用abort或exit。
5. 当我们希望视被分配物属于哪个class以确定以不同的方式处理内存分配失败的情况时，由于C++并不支持class专属之new-handler，我们只需令每个class提供自己专属的set_new_handler和operator new即可，set_new_handler使我们得以指定class的专属new_handler，同时以这个new_handler取代全局的new_handler，operator new则确保以专属的new_handler替换全局的。
6. Nothrow new是一个颇为局限的工具，因为它只适用于内存分配；后续的构造函数调用还是可能抛出异常。

### 了解new和delete的合理替换时机
1. 我们使用定制的new和delete替换编译器提供的operator new和operator delete的理由最常见的无非三个：  
（1）用来检测运用上的错误：比如数据的overruns或underruns等  
（2）为了强化性能，使用定制版本的new和delete能提升性能  
（3）为了收集使用上的统计数据
2. 许多计算机体系结构要求特定的类型必须放在特定的内存地址上，C++要求所有operator news返回的指针都有适当的对齐（取决于数据类型）。operator new返回一个得自malloc的指针是安全的，但是如果这个指针有所偏移则没人能保证其安全。
3. 何时应当使用定制new和delete替换默认的版本：  
（1）为了检测运行错误  
（2）为了收集动态分配内存之使用统计信息  
（3）为了增加分配和归还的速度  
（4）为了降低缺省内存管理器带来的空间额外开销  
（5）为了弥补缺省分配器中的非最佳齐位  
（6）为了将相关对象成簇集中  
（7）为了获得非传统的行为

### 编写new和delete时需固守常规
1. 实现一致性operator new必须返回正确的值，并在每次失败后调用new-handling函数，只有当指向new-handling函数的指针是null时，operator new才会抛出bad_alloc异常。
2. C++规定，即使客户要求0bytes，operator new也得返回一个合法指针。
3. operator new成员函数会被派生类所继承，而我们定制new的原因可能是为针对某特定class对象分配行为提供最优化，并不是为了该类的任何派生类，一旦继承很可能基类的operator new被调用用以分配派生类对象。解决这一问题的最佳做法就是将“内存申请量错误”的调用行为改采用标准operator new。
4. 撰写operator delete的注意事项为C++永远保证“删除null指针永远安全”
5. 如果即将被删除的对象派生自某个基类而这个基类缺少虚析构函数，那么C++传递给operator delete的size_t数值可能不正确，因此我们需要为基类撰写虚析构函数。

### 写了placement new也要写placement delete
1. 当我们new分配内存时未发生异常，而在构造函数时发生异常，这时收回分配内存的责任就到了C++运行期系统上。当我们只使用正常形式的new和delete时，运行期系统毫无问题可以找出能够回收内存的delete，但是当声明了非正常形式的operator new时想找出对应的operator delete就复杂了。
2. 如果operator new接受的参数除了一定会有的size_t之外还有其他，这个就被称为placement new，其中特别有用的一个版本是“接受一个指针指向对象该被构造之处”。operator delete如果接受额外参数便称为placement delete。
3. 如果一个带额外参数的operator new没有“带相同额外参数”的对应版operator delete，那么当new的内存分配动作需要取消并恢复旧观时就没有任何operator delete会被调用。
4. 当placement new调用成功返回指针时，再调用delete就是正常形式的delete，而非placement delete。placement delete只有在“伴随placement new调用而出发的构造函数”出现异常时才会调用。
5. 由于成员函数名称会掩盖其外围作用域中的相同名称，我们必须小心避免让class专属的news掩盖其他news（包括正常版本），一个解决方案就是建立一个基类内含所有正常形式的new和delete。凡是想以自定形式扩充标准形式的用户，可利用继承机制及using声明式取得标准形式。
