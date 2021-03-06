### 11月2日学习`InnoDB`笔记 ###

1. `InnoDB`有多个内存块，可以认为这些内存块组成了一个大的内存池，负责一下一些工作:
   1. 维护所有进程/线程需要访问的多个内部数据结构
   2. 缓存磁盘上的数据，方便快速读取，同时在对磁盘文件的修改之前在这里缓存
   3. 重做日志（redo log) 缓冲

2. `InnoDB`存储引擎是多线程模型，因此后台有多个线程，后台线程的主要作用是负责刷新内存池中的数据，保证缓存池的内存缓存的是最新的数据，  此外将已经修改的数据文件刷新到新的磁盘文件，同时保证异常情况下`InnoDB`恢复到正常状态

3. 具体分析线程:

   1. `Master Thread`是一个非常核心的线程，主要负责将缓冲池中的数据__异步__刷新到磁盘，保证数据的**__一致性__**
   2. `IO Thread`：`InnoDB`大量使用了`AIO(Async IO)`来处理写IO请求，极大的提高数据库的性能。而IO Thread的主要任务是负责这些IO请求的回调（`call back`)处理。
   3. `Perge Thread`:事务被提交后，其所使用的`undolog`可能不再需要，因此需要`Purge Thread`来回收已经使用并分配的undo页。`InnoDB1.1`以后`purge`操作独立到单独的线程，不在`Master Thread`中完成了，减轻了`Master Thread`的工作，从而提高了性能。`InnoDB1.2`以后支持多个`Purge Thread`加快对undo页的回收。同时由于`Purge Thread`需要离散的读取undo页，这样也更能利用磁盘的随机性读取性能
   4. `Purge Cleaner Thread`：在`InnoDB1.2`后引入的，其作用是将之前版本中脏页的刷新操作独立到单独线程，减轻`Master Thread`的工作以及对于用户查询线程的阻塞。

   

