首先概括一下这几个概念。其中SM（Streaming Multiprocessor）和SP（streaming Processor）是硬件层次的，其中一个SM可以包含多个SP。thread是一个线程，多个thread组成一个线程块block，多个block又组成一个线程网格grid。
现在就说一下一个kenerl函数是怎么执行的。一个kernel程式会有一个grid，grid底下又有数个block，每个block是一个thread群组。在同一个block中thread可以通过共享内存（shared memory）来通信，同步。而不同block之间的thread是无法通信的。
CUDA的设备在实际执行过程中，会以block为单位。把一个个block分配给SM进行运算；而block中的thread又会以warp（线程束）为单位，对thread进行分组计算。目前CUDA的warp大小都是32，也就是说32个thread会被组成一个warp来一起执行。同一个warp中的thread执行的指令是相同的，只是处理的数据不同。
基本上warp 分组的动作是由SM 自动进行的，会以连续的方式来做分组。比如说如果有一个block 里有128 个thread 的话，就会被分成四组warp，第0-31 个thread 会是warp 1、32-63 是warp 2、64-95是warp 3、96-127 是warp 4。而如果block 里面的thread 数量不是32 的倍数，那他会把剩下的thread独立成一个warp；比如说thread 数目是66 的话，就会有三个warp：0-31、32-63、64-65 。由于最后一个warp 里只剩下两个thread，所以其实在计算时，就相当于浪费了30 个thread 的计算能力；这点是在设定block 中thread 数量一定要注意的事！
一个SM 一次只会执行一个block 里的一个warp，但是SM 不见得会一次就把这个warp 的所有指令都执行完；当遇到正在执行的warp 需要等待的时候（例如存取global memory 就会要等好一段时间），就切换到别的warp来继续做运算，借此避免为了等待而浪费时间。所以理论上效率最好的状况，就是在SM 中有够多的warp 可以切换，让在执行的时候，不会有「所有warp 都要等待」的情形发生；因为当所有的warp 都要等待时，就会变成SM 无事可做的状况了。
实际上，warp 也是CUDA 中，每一个SM 执行的最小单位；如果GPU 有16 组SM 的话，也就代表他真正在执行的thread 数目会是32*16 个。不过由于CUDA 是要透过warp 的切换来隐藏thread 的延迟、等待，来达到大量平行化的目的，所以会用所谓的active thread 这个名词来代表一个SM 里同时可以处理的thread 数目。而在block 的方面，一个SM 可以同时处理多个thread block，当其中有block 的所有thread 都处理完后，他就会再去找其他还没处理的block 来处理。假设有16 个SM、64 个block、每个SM 可以同时处理三个block 的话，那一开始执行时，device 就会同时处理48 个block；而剩下的16 个block 则会等SM 有处理完block 后，再进到SM 中处理，直到所有block 都处理结束



每个 thread 都有自己的一份 register 和 local memory 的空间。
一组thread构成一个 block，这些thread 则共享有一份shared memory。
所有的 thread(包括不同 block 的 thread)都共享一份global memory、constant memory、和 texture memory。
不同的 grid 则有各自的 global memory、constant memory 和 texture memory。
每一个时钟周期内，warp（一个block里面一起运行的thread，其中各
个线程对应的数据资源不同(指令相同但是数据不同）包含的thread数量是有限的，现在的规定是32个。一个block中含有16个warp。所以一个block中最多含有512个线程.每次Device（就是GPU）只