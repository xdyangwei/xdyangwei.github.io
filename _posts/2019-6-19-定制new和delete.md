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
5. 