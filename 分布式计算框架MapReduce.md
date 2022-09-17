#分布式计算框架MapReduce

标签（空格分隔）： 未分类

---

**MapReduce是一种用于在大型商用硬件集群中（成千上万的节点）对海量数据（多个兆字节数据集）实施可靠的、高容错的分布式计算的框架，也是一种经典的并行计算模型。基本原理是将一个复杂的问题（数据集）分成若干个简单的子问题（数据块）进行解决。**

##一、MapReduce编程模型
MapReduce编程模型主要由两个抽象类构成，即Mapper类和Reducer类，Mapper用于对切分过的原始数据进行处理，Reducer则对Mapper的结果进行汇总。

##二、MapReduce数据流
###2.1分片、格式化数据源(InputFormat)
- InputFormat主要有两个任务，一个是对源文件进行分片，并确定Mapper的数量；另一个是对各分片进行格式化，处理成<key,value>形式的数据流并传给Mapper。
###2.2 Map过程
- Mapper接收<key,value>形式的数据，并处理成<key,value>形式的数据，具体的处理过程可由用户定义。在WordCount中，Mapper会解析传过来的Key值，以“空字符”为标识符，如果碰到“空字符”，就会把之前累计的字符串作为输出的key值，并以1作为当前key的value值，形成<word,1>的形式。
###2.3 Combiner过程
- 每一个map()都可能会产生大量的本地输出，Combiner()的作用就是对map()端的输出先做一次合并，以减少在Map和Reduce结点之间的数据传输量，提高网络I/O性能，是MapReduce的一种优化手段之一。如在WordCount中，map()在传递给Combiner()前，map端的输出会先做一次合作。
###2.4 Shuffle过程
- Shuffle过程是指从Mapper产生的直接输出结果，经过一系列的处理，成为最终的Reducer直接输入数据为止的整个过程，这一过程也是MapReduce的核心过程。
- 整个Shuffle过程可以分为两个阶段，Mapper端的Shuffle和Reducer端的Shuffle。由Mapper产生的数据并不会直接写入磁盘，而是先存储在内存中，当内存中的数据达到设定阈值时，再把数据写到本地磁盘，并同时进行sort(排序)、combine操作是把key值相同的相邻记录进行合并；partition操作涉及如何把数据均衡地分配给多个Reducer，它直接关系到Reducer的负载均衡。其中combine操作不一定会有，因为在某些场景不适用，但为了使Mapper的输出结果更加紧凑，大部分情况都会使用。
###2.5 Reduce过程
- Reducer接受<key,{value list}>形式的数据流，形成<key,value>形式的数据输出，输出数据直接写入HDFS,具体的处理过程可由用户定义。在WordCount中，Reducer会将相同key的value list进行累加，得到这个单词出现的总次数，然后输出。
##三、MapReduce任务运行流程
###3.1 MRv2基本组成
- MRv2舍弃了MRv1(是Hadoop1中的MapReduce任务运行流程)中的JobTrack和TaskTrack，而采用一种新的MRAppMaster进行单一任务管理，并与Yarn中的ResourceManager和NodeManage协同调度与控制任务，避免了由单一服务（MRv1中的JobTrack）管理和调度所以任务而产生的负载过重的问题。
- MRv2基本组成
    - 1)客户端
    - 2）MRAppMaster
    - 3)Map Task和Reduce Task
###3.2 Yarn基本组成
Yarn是一个资源管理平台，它监控和调度整个集群资源，并负责管理集群所有任务的运行和任务资源的分配。
基本组成
- 1）Resource Manager(RM)主要包括两个组件：Resource Schedule(资源调度器)和Applications Mangaer(应用程序管理器)。
- 2）NodeManager
- 3) ApplicationMaster
- 4) container
###3.3 任务流程
在Yarn中，资源管理由ResourceManage和NodeManager共同完成，其中，ResourceManager中的调度器负责资源的分配，NodeManager负责资源的供给和隔离。ResourceManager将某个NodeManager上资源分配给任务（所谓的“资源调度”）后，NodeManager需按照要求为任务提供相应的资源，并保证这些资源具有独占性，为任务运行提供基础的保证。Yarn架构中的MapReduce任务运行流程主要可以分为两个部分：一个是客户端向ResourceManage提交任务，ResourceManage通知相应的NodeManager启动MRAppMaster;二是MRAppMaster启动成功后，则由它调度整个任务的运行，直到任务完成。
- 1）clint向ResourceManager提交任务。
- 2）ResourceManage分配该任务的第一个container，并通知相应的NodeManager启动MRAppMaster。
- 3）NodeManager接受命令后，开辟一个container资源空间，并在container中启动相应的MRAppMaster
- 4）MRAppMaster启动后，第一步会向ResourceManage注册，这样用户可以直接通过MRAppMaster监控任务的运行状态；之后则直接由MRAppMaster调度任务运行，重负5~8，知道任务结束。
- 5）MRAppMaster轮询的方式向ResourceManage申请任务运行所需的资源。
- 6）一旦ResourceManage配给了资源，MRAppMaster便会与相应的通信，让它划分container并启动相应的任务
- 7）NodeManager准备号运行环境，启动任务。
- 8）各任务运行，并定时通过RPC协议向MRAppMaster汇报自己的运行状态和进度。MRAppMaster也会实时的监控任务的运行，当发现某个Task假死或失败是，便杀死它重新启动任务。
- 9）任务完成，MRAppMaster向ResourceManage通信，注销并关闭自己。





