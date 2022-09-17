#Mapreduce的预习报告



---
##1 分布式计算框架MapReduce

       1.可靠的、高容错的分布式计算框架，经典的并行的计算框架。
       2.基本原理：将一个复杂的问题（数据集）分成若干个简单的子问题（数据块）进行解决（Map函数）；然后对子问题的结果进行合并（Reduce函数），得到问题解决的解（结果）。 
        3.MapReduce模型适合于大文件处理，对很多小文件处理效率不是很高。

###1.1 MapReduce编程模型

    1. mapreduce是一种思想或是一种编程模型。
    2. mapreduce编程模型由两个抽象类构成，分别为mapper类和reducer类。
    3.根据工作原理，可将mapReduce编程模型分为两类：mapreduce简单模型和mapreduce复杂模型。
    4.mapreduce简单模型：不一定需要reduce过程。
    5. mapreduce复杂模型：一般都需要reduce过程。

###1.2 MapReduce数据流
    
    

    1.分片格式化数据源（InputFormat):InputFormat主要有两个任务，一是对文件进行分片，并确定Mapper的数量；二是对各分片进行格式化处理，处理成<key,value>形式的数据流并传给Mapper。
    2.Map过程：Mapper接收<key,value>形式的数据，并处理成<key,value>形式的数据，具体的处理过程可由用户定义。
    3.combiner过程：对Map（）端的输出先做一次合并，以减少在Map和Reduce结点之间的数据传输量，提高网络I/O性能，是mapreduce优化手段之一。
    4.shuffle过程：指从Mapper产生的直接输出结果，经过一系列的处理，成为最终的Reducer直接输入数据为止的整个过程，这一过程也是mapreduce的核心过程。
    5.Reduce过程：Reducer接收<key,{value     list}>形式的数据流，形成<key,value>形式的数据输出，输出数据直接写入HDFS，具体的处理过程可由用户定义。

###1.3 MapReduce任务运行流程
####1.3.1MRv2基本组成

    1.客户端：客户端用于向Yarn集群提交任务，是MapReduce用户和Yarn集群通信的唯一途径。
    2.MRAppMaster：MRAppMaster只负责任务管理，并不负责资源的调配。
    3.Map Task和Reduce Task:只能运行在Yarn给定的资源限制下，由MRAppMaster和NodeManage协同管理和调度。
####1.3.2Yarn基本组成

    1.Resource Manage(RM):运行于NameNode，为整个集群的资源调度器，主要包括两个组件：Resource Schedule（资源调度器）和Application Manager（应用程序管理器）。
    2.NodeManager：运行于DataNode，监控并管理单个节点的计算资源，并定时向RM汇报结点的资源使用情况，在节点上有任务时，还负责对container进行创建、运行状态的监控及最终销毁。
    3.ApplicationMaster（AM）：负责对一个任务流的调度、管理，包括任务注册、资源申请、以及和NodeManage通信以及开启和杀掉任务。
    4.container：Yarn架构下对运算资源的一种描述
    
####1.3.3任务流程
    

    1.client 向ResourceManager提交任务。
    2.ResourceManager分务第一个container，并通知相应的NodeManager启动 MRAppMaster.
    3.NodeManager接收命令后，开辟一个container 资源空间，并在container中启动相应的 MRAppMaster.
    4.MRAppMaster启动之后，第一步会向ResourceManager注册，这样用户可以直接通过 MRAppMaster监控任务的运行状态;之后则直接由MRAppMaster调度任务运行，重复5~8，直到任务结束。
    5.MRAppMaster 以轮询的方式向ResourceManager申请任务运行所需的资源。
    6.一旦ResourceManager配给了资源，MRAppMaster便会与相应的NodeManager通信让它划分Container并启动相应的任务(MapTask或ReduceTask)。
    7.NodeManager准备好运行环境，启动任务。
    8.各任务运行，并定时通过RPC 协议向MRAppMaster汇报自己的运行状态和进度。 MRAppMaster也会实时地监控任务的运行，当发现某个Task假死或失败时，便杀死它重新启动任务。
    9.任务完成，MRAppMaster向ResourceManager通信，注销并关闭自己。

