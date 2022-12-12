# 如何调试无响应的 Elasticsearch 集群

> 原文：<https://www.moesif.com/blog/technical/elasticsearch/How-to-Debug-an-Unresponsive-Elasticsearch-Cluster/>

Elasticsearch 是一个开源搜索引擎和分析商店，被各种应用程序使用，从电子商务商店的搜索到使用 ELK stack 的内部日志管理工具(简称“Elasticsearch，Logstash，Kibana”)。作为一个分布式数据库，您的数据被分割成“碎片”,然后分配给一个或多个服务器。

由于这种分片，对 Elasticsearch 集群的读或写请求需要在多个节点之间进行协调，因为在单个服务器上没有数据的“全局视图”。虽然这使得 Elasticsearch 高度可伸缩，但这也使得它的设置和调优比 MongoDB 或 PostgresSQL 等其他流行数据库复杂得多，这些数据库*可以*在单个服务器上运行。

当可靠性问题出现时，如果您的弹性搜索设置有问题或不稳定，救火可能会很有压力。您的事故可能会影响客户，从而对收入和您的商业声誉产生负面影响。快速补救措施很重要，但在事故或停电期间花费大量时间在线研究解决方案对大多数工程师来说并不奢侈。本指南旨在为工程师运行的常见问题提供一份备忘单，这些问题可能会导致 Elasticsearch 出现问题，以及需要查找的内容。

作为一个通用工具，Elasticsearch 有数千种不同的配置，这使它能够适应各种不同的工作负载。即使在线发布，适用于一家公司的数据模型或配置可能并不适合您的公司。没有灵丹妙药可以让 Elasticsearch 规模化，需要勤奋的性能测试和反复试验。

## 无响应的弹性搜索集群问题

集群稳定性问题是最难调试的问题之一，尤其是在您的数据量或代码库没有任何变化的情况下。

### 检查集群状态的大小

#### 它有什么作用:

*   [Elasticsearch 集群状态](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-state.html)跟踪我们集群的全局状态，是控制流量和集群的核心。集群状态包括集群中节点的元数据、碎片的状态以及它们如何映射到节点、索引映射(即模式)等等。
*   集群状态通常不会经常改变。但是，某些操作(如向索引映射添加新字段)会触发更新。
*   因为集群更新会广播到集群中的所有节点，所以它应该很小(< 100MB)。
*   大型集群状态会很快使集群变得不稳定。发生这种情况的常见方式是通过[映射爆炸](https://www.elastic.co/blog/found-crash-elasticsearch#mapping-explosion)(一个索引中有太多的键)或太多的索引。

#### 寻找什么:

*   使用下面的命令下载集群状态，并查看返回的 JSON 的大小。

    ```py
    curl -XGET 'http://localhost:9200/_cluster/state' 
    ```

*   特别是，查看哪些索引在集群状态中有最多的字段，这可能是导致稳定性问题的违规索引。如果集群状态较大且不断增加。您还可以查看单个索引或与如下索引模式进行匹配:

    ```py
    curl -XGET 'http://localhost:9200/_cluster/state/_all/my_index-*' 
    ```

*   您还可以使用以下命令查看违规索引的映射:

    ```py
    curl -XGET 'http://localhost:9200/my_index/_mapping' 
    ```

#### 如何修复:

*   看看数据是如何被索引的。当高基数标识符被用作 JSON 键时，通常会出现映射爆炸。每次看到类似“4”和“5”的新键时，群集状态都会更新。例如，下面的 JSON 将很快导致 Elasticsearch 的稳定性问题，因为每个键都被添加到全局状态。

    ```py
     {  "1":  {  "status":  "ACTIVE"  },  "2":  {  "status":  "ACTIVE"  },  "3":  {  "status":  "DISABLED"  }  } 
    ```

    T4】
*   为了解决这个问题，将你的数据扁平化，使之对弹性搜索友好:

    ```py
    {  [  {  "id":  "1",  "status":  "ACTIVE"  },  {  "id":  "2",  "status":  "ACTIVE"  },  {  "id":  "3",  "status":  "DISABLED"  }  ]  } 
    ```

### 检查弹性搜索任务队列

#### 它有什么作用:

*   当对 elasticsearch(索引操作、查询操作等)发出请求时，它首先被插入到任务队列中，直到一个工作线程可以拾取它。
*   一旦工作池有一个空闲线程，它将从任务队列中选取一个任务并处理它。
*   这些操作通常由您通过 HTTP 请求在`:9200`和`:9300`端口上完成，但是它们也可以在内部处理索引的维护任务
*   在给定时间，可能有数百或数千个正在进行的操作，但应该非常快地完成(如微秒或毫秒)。

#### 寻找什么:

*   运行下面的命令，查找长时间(如几分钟或几小时)停滞运行的[任务](https://www.elastic.co/guide/en/elasticsearch/reference/current/tasks.html)。
*   这意味着某些东西正在使集群饥饿，阻止它向前发展。
*   对于某些长时间运行的任务来说，比如移动一个索引需要很长时间，这是可以接受的。然而，普通的查询和索引操作应该很快。

    ```py
    curl -XGET 'http://localhost:9200/_cat/tasks?detailed' 
    ```

*   使用`?detailed`参数，您可以获得关于目标索引和查询的更多信息。
*   寻找任务总是排在列表首位的模式。是同一个指数吗？是同一个节点吗？
*   如果是这样，可能是该索引的数据有问题，或者是节点过载。

#### 如何修复:

*   如果请求量高于正常水平，那么就寻找优化请求的方法(比如使用批量 API 或更高效的查询/写入)
*   如果数量没有变化，并且看起来是随机的，这意味着有其他东西正在降低集群的速度。任务备份只是一个更大问题的征兆。
*   如果你不知道请求来自哪里，给你的 Elasticsearch 客户端添加 [`X-Opaque-Id`](https://www.elastic.co/guide/en/elasticsearch/reference/current/tasks.html#_identifying_running_tasks) 头来识别哪个客户端触发了查询。

### 检查弹性搜索待定任务

#### 它有什么作用:

*   [未决任务](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-pending.html)是对集群状态的未决更新，例如创建新索引或更新其映射。
*   与前面的任务队列不同，挂起的更新需要多步握手来将更新广播到群集中的所有节点，这可能需要一些时间。
*   在给定的时间内，飞行中的任务应该几乎为零。请记住，快照恢复等昂贵的操作可能会导致这种情况暂时加剧。

#### 寻找什么:

*   运行命令并确保没有或很少任务在运行。

    ```py
    curl curl curl -XGET 'http://localhost:9200/_cat/pending_tasks' 
    ```

*   如果它看起来是一个持续的集群更新流，并且很快完成，那么看看是什么触发了它们。是映射爆炸还是创建了太多指数？
*   如果只是几个，但是它们似乎卡住了，那么查看主节点的日志和指标，看看是否有任何问题。例如，主节点是否遇到内存或网络问题，以至于无法处理集群更新？

### 热线程

#### 它有什么作用:

*   hot threads API 是一个有价值的内置分析器，可以告诉你 Elasticseach 在哪里花费的时间最多。
*   这可以提供一些见解，如 Elasticsearch 是否在索引刷新或执行昂贵的查询上花费了太多时间。

#### 寻找什么:

*   调用[热线程 API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-nodes-hot-threads.html) 。为了提高精度，建议使用`?snapshots`参数

    ```py
    curl -XGET 'http://localhost:9200/_nodes/hot_threads?snapshots=1000' 
    ```

    捕捉多个快照
*   这将返回拍摄快照时看到的堆栈跟踪。
*   在许多不同的快照中寻找相同的堆栈。例如，您可能会看到文本`5/10 snapshots sharing following 20 elements`。这意味着一个线程在 5 个快照期间花费时间在代码的那个区域。
*   你还应该看看 CPU 的百分比。如果一个代码区域同时具有高快照共享和高 CPU %,这就是一个热门代码路径。
*   通过查看代码模块，分解 Elasticseach 在做什么。
*   如果您看到等待或暂留状态，这通常是正常的。

#### 如何修复:

*   如果在索引刷新上花费了大量的 CPU 时间，那么请尝试增加刷新间隔，使其超过默认的 1 秒。
*   如果您看到大量缓存，可能您的默认缓存设置不是最佳的，并导致严重的未命中。

## 内存问题

### 检查弹性搜索堆/垃圾收集

#### 它有什么作用:

*   作为一个 JVM 进程，堆是存储大量弹性搜索数据结构的内存区域，需要垃圾收集周期来修剪旧对象。
*   对于典型的生产设置，Elasticsearch 在启动时使用`mlockall`锁定所有内存并禁用交换。如果你不想做，现在就做。
*   如果某个节点的堆一直在 85%或 90%以上，这意味着我们即将用尽内存。

#### 寻找什么:

*   在 Elasticsearch 日志中搜索`collecting in the last`。如果这些都存在，这意味着 Elasticsearch 在垃圾收集上花费了更高的开销(占用了其他生产任务的时间)。
*   只要 Elasticsearch 没有将大部分 CPU 周期花费在垃圾收集上(计算花费在收集上的时间相对于所提供的总时间的百分比),偶尔这样做是可以的
*   一个 100%时间都花在垃圾收集上的节点停滞不前，无法继续前进。
*   看似有网络问题(如超时)的节点实际上可能是由于内存问题。这是因为在垃圾收集周期中，节点无法响应传入的请求。

#### 如何修复:

*   最简单的方法是添加更多的节点来增加集群可用的堆。然而，Elasticsearch 需要时间来将碎片重新平衡到空节点。
*   如果只有一小部分节点具有高堆使用率，您可能需要更好地平衡您的客户。例如，如果您的碎片大小变化很大，或者具有不同的查询/索引带宽，那么您可能向同一组节点分配了太多的热碎片。要移动碎片，使用[重新路由 API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-reroute.html) 。只要调整[碎片感知灵敏度](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cluster.html#shard-allocation-awareness)确保它不会被移回来。

    ```py
    curl -XPOST -H "Content-Type: application/json" localhost:9200/_cluster/reroute -d '
    {
    "commands": [
        {
          "move": {
            "index": "test", "shard": 0,
            "from_node": "node1", "to_node": "node2"
          }
        }
      ]
    }' 
    ```

*   如果您向 Elasticsearch 发送大量请求，请尝试减少批量大小，使每批小于 100MB。虽然较大的批处理有助于减少网络开销，但它们需要分配更多的内存来缓冲请求，这些内存在请求完成和下一个 GC 周期之前无法释放。

### 检查弹性搜索旧的记忆压力

#### 它有什么作用:

*   旧内存池包含经历了多个垃圾收集周期的对象，并且是长期存在的对象。
*   如果[老内存](https://www.elastic.co/blog/found-understanding-memory-pressure-indicator)超过 75%，你可能要关注一下了。当填充超过 85%时，会发生更多的 GC 循环，但是对象不能被清除。

#### 寻找什么:

*   看旧池已用/旧池最大。如果这一比例超过 85%，那就令人担忧了

#### 如何修复:

*   您是否急切地加载大量现场数据。这些在内存中驻留很长时间。
*   您是否正在执行许多长期运行的分析任务？某些任务应该卸载到为 map/reduce 操作设计的分布式计算框架，如 Apache Spark。

### 检查弹性搜索字段数据大小

#### 它有什么作用:

*   FieldData 用于计算字段的聚合，如`terms`聚合
*   通常，在第一次对字段执行聚合之前，字段的数据不会加载到内存中。
*   然而，如果设置了`eager_load_ordinals`，这也可以在索引刷新时预先计算。

#### 寻找什么:

*   查看一个索引或所有索引字段数据大小，如下所示:

    ```py
    curl -XGET 'http://localhost:9200/index_1/_stats/fielddata' 
    ```

*   如果我们对错误类型的数据使用索引，那么索引可能会有非常大的字段数据结构。您是否正在对基数非常高的字段(如 UUID 或跟踪 ID)执行聚合？Fielddata 不适合非常高基数的字段，因为它们会创建大量的 fielddata 结构。
*   您是否有许多设置了`eager_load_ordinals`的字段或向 fielddata 缓存分配了大量数据。这导致 fielddata 在刷新时间而不是查询时间生成。虽然它可以加速聚合，但如果您在索引刷新时计算许多字段的字段数据，并且从不在查询中使用它，那么它就不是最佳选择。

#### 如何修复:

*   对您的查询或映射进行调整，不要在非常高的基数键上进行聚合。
*   审核您的映射以减少将`eager_load_ordinals`设置为 true 的数量。

## 弹性搜索网络问题

### 节点向左或节点断开

#### 它有什么作用:

*   如果一个节点不响应请求，它最终将从集群中删除。
*   这允许将碎片复制到其他节点，以满足复制因子并确保高可用性，即使某个节点被删除。

#### 寻找什么:

*   查看主节点日志。即使有多个主节点，您也应该查看当前选出的主节点。您可以使用 nodes API 或类似于 Cerebro 的工具来完成这项工作。
*   查看是否有一致的节点超时或有问题。例如，您可以通过在主节点的日志中查找短语`pending nodes`来查看哪些节点仍在等待集群更新。
*   如果您看到同一个节点不断被添加，然后又被删除，这可能意味着该节点过载或无响应。
*   如果您无法从主节点访问该节点，这可能意味着存在网络问题。您也可能遇到网卡或 CPU 带宽限制

#### 如何修复:

*   将设置`transport.compression`设置为 true 进行测试。这将压缩节点之间的流量(例如从接收节点到数据节点),以 CPU 带宽为代价减少网络带宽。
*   注意:早期版本称此设置为`transport.tcp.compression`
*   如果您也有内存问题，请尝试增加内存。由于在垃圾收集上花费了大量时间，节点可能会变得无响应。

## 主节点问题不足

#### 它有什么作用:

*   主节点和其他节点需要相互发现以形成集群。
*   在第一次启动时，你必须提供一组静态的主节点，这样你就不会有 T2 裂脑问题。
*   只要主节点存在，其他节点就会自动发现集群。

#### 寻找什么:

*   [启用跟踪记录](https://www.elastic.co/blog/elasticsearch-logging-secrets)以查看与发现相关的活动。

    ```py
    curl -XPUT -H "Content-Type: application/json" localhost:9200/_cluster/_settings -d '
    {
      "transient": {"logger.discovery.zen":"TRACE"}
    }' 
    ```

*   查看配置，如`minimum_master_nodes`(如果比 6.x 版本旧)。
*   查看您的初始主节点列表中的所有主节点是否可以相互 ping 通。
*   审核自己是否有*法定人数*，应该是`number of master nodes / 2 +1`。如果少于法定数量，将不会更新群集状态以保护数据完整性。

#### 如何修复:

*   有时，网络或 DNS 问题会导致无法到达原始主节点。
*   检查当前至少有`number of master nodes / 2 +1`个主节点在运行。

## 碎片分配错误

### 黄色或红色状态的弹性搜索(未分配的碎片)

#### 它有什么作用:

*   当节点重新启动或群集恢复开始时，碎片不会立即可用。
*   恢复受到限制，以确保群集不会不堪重负。
*   黄色状态表示主索引已分配，但辅助(副本)碎片尚未分配。虽然黄色索引既可读又可写，但可用性会降低。当集群复制碎片时，黄色状态通常是可以自我修复的。
*   红色索引表示主碎片未被分配。这可能是暂时的，例如在快照恢复操作期间，但也可能意味着重大问题，例如丢失数据。

#### 寻找什么:

*   了解分配停止的原因

    ```py
    curl -XGET 'http://localhost:9200/_cluster/allocation/explain'

    curl -XGET 'http://localhost:9200/_cat/shards?h=index,shard,prirep,state,unassigned.reason' 
    ```

*   获取红色指数列表，以了解哪些指数导致了红色状态。只要至少有一个索引是红色的，集群状态就会是红色的。

    ```py
    curl -XGET 'http:localhost:9200/_cat/indices' | grep red 
    ```

*   有关单个索引的更多详细信息，您可以查看违规索引的恢复状态

    ```py
    curl -XGET 'http:localhost:9200/index_1/_recovery' 
    ```

#### 如何修复:

*   如果您看到 max_retries 超时(可能是集群在分配期间很忙)，您可以临时增加断路器阈值(默认值为 5)。一旦数字高于断路器，Elasticsearch 将开始初始化未分配的碎片。

    ```py
    curl -XPUT -H "Content-Type: application/json" localhost:9200/index1,index2/_settings -d '
    {
      "index.allocation.max_retries": 7
    }' 
    ```

## 弹性搜索磁盘问题

### 索引是只读的

#### 它有什么作用:

*   Elasticsearch 有三个影响碎片分配的[基于磁盘的水印](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cluster.html#disk-based-shard-allocation)。`cluster.routing.allocation.disk.watermark.low`水印防止新碎片被分配给磁盘已满的节点。默认情况下，这是所用磁盘的 85%。
*   `cluster.routing.allocation.disk.watermark.high`水印将迫使集群开始将碎片从该节点转移到其他节点。默认情况下，这是 90%。这将开始移动数据，直到低于高水位线。如果 Elasticsearch 磁盘超过泛滥阶段水位线`cluster.routing.allocation.disk.watermark.flood_stage`，则说明磁盘已满，在磁盘空间耗尽之前，移动速度可能不够快。到达时，索引被置于只读状态，以避免数据损坏。

#### 寻找什么:

*   查看每个节点的磁盘空间
*   查看如下消息的节点日志:

    ```py
    high disk watermark [90%] exceeded on XXXXXXXX free: 5.9gb[9.5%], shards will be relocated away from this node 
    ```

*   一旦到达洪水阶段，您会看到如下日志:

    ```py
    flood stage disk watermark [95%] exceeded on XXXXXXXX free: 1.6gb[2.6%], all indices on this node will be marked read-only 
    ```

*   一旦发生这种情况，该节点上的索引就是只读的。
*   要进行确认，请查看哪些索引的`read_only_allow_delete`设置为真。

    ```py
    curl -XGET 'http://localhost:9200/_all/_settings?pretty' | grep read_only 
    ```

#### 如何修复:

*   首先，清理磁盘空间，比如删除本地日志或 tmp 文件。
*   要删除此只读块，请执行以下命令:

    ```py
    curl -XPUT -H "Content-Type: application/json" localhost:9200/_all/_settings -d '
    {
      "index.blocks.read_only_allow_delete": null
    }' 
    ```

## 结论

对稳定性和性能问题进行故障排除具有挑战性。找到根本原因的最佳方法是使用假设的科学方法，并证明其正确或不正确。使用这些工具和 Elasticsearch 管理 API，您可以深入了解 Elasticsearch 的表现和问题所在。