---
title: YARN简介
date: 2016-07-05 15:55:42
categories: Big Data
---
## Apache Hadoop 简介
Apache Hadoop 是一个开源软件框架，可安装在一个商用机器集群中，使机器可彼此通信并协同工作，以高度分布式的方式共同存储和处理大量数据。最初，Hadoop 包含以下两个主要组件：Hadoop Distributed File System (HDFS) 和一个分布式计算引擎，该引擎支持以 MapReduce 作业的形式实现和运行程序。
MapReduce 是 Google 推广的一个简单的编程模型，它对以高度并行和可扩展的方式处理大数据集很有用。MapReduce 的灵感来源于函数式编程，用户可将他们的计算表达为 map 和 reduce 函数，将数据作为键值对来处理。Hadoop 提供了一个高级 API 来在各种语言中实现自定义的 map 和 reduce 函数。
Hadoop 还提供了软件基础架构，以一系列 map 和 reduce 任务的形式运行 MapReduce 作业。Map 任务 在输入数据的子集上调用 map 函数。在完成这些调用后，reduce 任务 开始在 map 函数所生成的中间数据上调用 reduce 任务，生成最终的输出。 map 和 reduce 任务彼此单独运行，这支持并行和容错的计算。
最重要的是，Hadoop 基础架构负责处理分布式处理的所有复杂方面：并行化、调度、资源管理、机器间通信、软件和硬件故障处理，等等。得益于这种干净的抽象，实现处理数百（或者甚至数千）个机器上的数 TB 数据的分布式应用程序从未像现在这么容易过，甚至对于之前没有使用分布式系统的经验的开发人员也是如此。
<hr/>
## Hadoop 的黄金时代
尽管 MapReduce 模型存在着多种开源实现，但 Hadoop MapReduce 很快就变得非常流行。Hadoop 也是全球最令人兴奋的开源项目之一，它提供了多项出色的功能：高级 API、近线性的可伸缩性、开源许可、在商用硬件上运行的能力，以及容错。它已获得数百（或许已达数千）个公司的成功部署，是大规模分布式存储和处理的最新标准。
一些早期的 Hadoop 采用者，比如 Yahoo! 和 Facebook，构建了包含 4,000 个节点的大型集群，以满足不断增长和变化的数据处理需求。但是，在构建自己的集群后，他们开始注意到了 Hadoop MapReduce 框架的一些局限性。
<hr/>
## 经典 MapReduce 的局限性
经典 MapReduce 的最严重的限制主要关系到可伸缩性、资源利用和对与 MapReduce 不同的工作负载的支持。在 MapReduce 框架中，作业执行受两种类型的进程控制：
* 一个称为 JobTracker 的主要进程，它协调在集群上运行的所有作业，分配要在 TaskTracker 上运行的 map 和 reduce 任务。
* 许多称为 TaskTracker 的下级进程，它们运行分配的任务并定期向 JobTracker 报告进度。

### Apache Hadoop 的经典版本 (MRv1)
![该图显示了 Apache Hadoop 的经典版本 (MRv1)](/img/post_4/1.png)
大型的 Hadoop 集群显现出了由单个 JobTracker 导致的可伸缩性瓶颈。依据 Yahoo!，在集群中有 5,000 个节点和 40,000 个任务同时运行时，这样一种设计实际上就会受到限制。由于此限制，必须创建和维护更小的、功能更差的集群。
此外，较小和较大的 Hadoop 集群都从未最高效地使用他们的计算资源。在 Hadoop MapReduce 中，每个从属节点上的计算资源由集群管理员分解为固定数量的 map 和 reduce slot，这些 slot 不可替代。设定 map slot 和 reduce slot 的数量后，节点在任何时刻都不能运行比 map slot 更多的 map 任务，即使没有 reduce 任务在运行。这影响了集群的利用率，因为在所有 map slot 都被使用（而且我们还需要更多）时，我们无法使用任何 reduce slot，即使它们可用，反之亦然。
最后但同样重要的是，Hadoop 设计为仅运行 MapReduce 作业。随着替代性的编程模型（比如 Apache Giraph 所提供的图形处理）的到来，除 MapReduce 外，越来越需要为可通过高效的、公平的方式在同一个集群上运行并共享资源的其他编程模型提供支持。
2010 年，Yahoo! 的工程师开始研究一种全新的 Hadoop 架构，用这种架构来解决上述所有限制并增加多种附加功能。
<hr/>
## 解决可伸缩性问题
在 Hadoop MapReduce 中，JobTracker 具有两种不同的职责：
* 管理集群中的计算资源，这涉及到维护活动节点列表、可用和占用的 map 和 reduce slots 列表，以及依据所选的调度策略将可用 slots 分配给合适的作业和任务
* 协调在集群上运行的所有任务，这涉及到指导 TaskTracker 启动 map 和 reduce 任务，监视任务的执行，重新启动失败的任务，推测性地运行缓慢的任务，计算作业计数器值的总和，等等

为单个进程安排大量职责会导致重大的可伸缩性问题，尤其是在较大的集群上，JobTracker 必须不断跟踪数千个 TaskTracker、数百个作业，以及数万个 map 和 reduce 任务。下图演示了这一问题。相反，TaskTracker 通常近运行十来个任务，这些任
务由勤勉的 JobTracker 分配给它们。
### 大型 Apache Hadoop 集群 (MRv1) 上繁忙的 JobTracker
![该图显示了大型 Apache Hadoop 集群 (MRv1) 上繁忙的 JobTracker](/img/post_4/2.png)
为了解决可伸缩性问题，一个简单而又绝妙的想法应运而生：我们减少了单个 JobTracker 的职责，将部分职责委派给 TaskTracker，因为集群中有许多 TaskTracker。在新设计中，这个概念通过将 JobTracker 的双重职责（集群资源管理和任务协调）分开为两种不同类型的进程来反映。
不再拥有单个 JobTracker，一种新方法引入了一个集群管理器，它惟一的职责就是跟踪集群中的活动节点和可用资源，并将它们分配给任务。对于提交给集群的每个作业，会启动一个专用的、短暂的 JobTracker 来控制该作业中的任务的执行。有趣的是，短暂的 JobTracker 由在从属节点上运行的 TaskTracker 启动。因此，作业的生命周期的协调工作分散在集群中所有可用的机器上。得益于这种行为，更多工作可并行运行，可伸缩性得到了显著提高。
## YARN：下一代 Hadoop 计算平台
我们现在稍微改变一下用辞。以下名称的改动有助于更好地了解 YARN 的设计：
+ ResourceManager 代替集群管理器
+ ApplicationMaster 代替一个专用且短暂的 JobTracker
+ NodeManager 代替 TaskTracker
+ 一个分布式应用程序代替一个 MapReduce 作业

YARN 是下一代 Hadoop 计算平台，如下所示。
### YARN 的架构
![该图显示了 YARN 的架构](/img/post_4/3.png)
在 YARN 架构中，一个全局 ResourceManager 以主要后台进程的形式运行，它通常在专用机器上运行，在各种竞争的应用程序之间仲裁可用的集群资源。ResourceManager 会追踪集群中有多少可用的活动节点和资源，协调用户提交的哪些应用程序应该在何时获取这些资源。ResourceManager 是惟一拥有此信息的进程，所以它可通过某种共享的、安全的、多租户的方式制定分配（或者调度）决策（例如，依据应用程序优先级、队列容量、ACLs、数据位置等）。
在用户提交一个应用程序时，一个称为 ApplicationMaster 的轻量型进程实例会启动来协调应用程序内的所有任务的执行。这包括监视任务，重新启动失败的任务，推测性地运行缓慢的任务，以及计算应用程序计数器值的总和。这些职责以前分配给所有作业的单个 JobTracker。ApplicationMaster 和属于它的应用程序的任务，在受 NodeManager 控制的资源容器中运行。
NodeManager 是 TaskTracker 的一种更加普通和高效的版本。没有固定数量的 map 和 reduce slots，NodeManager 拥有许多动态创建的资源容器。容器的大小取决于它所包含的资源量，比如内存、CPU、磁盘和网络 IO。目前，仅支持内存和 CPU (YARN-3)。未来可使用 cgroups 来控制磁盘和网络 IO。一个节点上的容器数量，由配置参数与专用于从属后台进程和操作系统的资源以外的节点资源总量（比如总 CPU 数和总内存）共同决定。
有趣的是，ApplicationMaster 可在容器内运行任何类型的任务。例如，MapReduce ApplicationMaster 请求一个容器来启动 map 或 reduce 任务，而 Giraph ApplicationMaster 请求一个容器来运行 Giraph 任务。您还可以实现一个自定义的 ApplicationMaster 来运行特定的任务，进而发明出一种全新的分布式应用程序框架，改变大数据世界的格局。您可以查阅 Apache Twill，它旨在简化 YARN 之上的分布式应用程序的编写。
在 YARN 中，MapReduce 降级为一个分布式应用程序的一个角色（但仍是一个非常流行且有用的角色），现在称为 MRv2。MRv2 是经典 MapReduce 引擎（现在称为 MRv1）的重现，运行在 YARN 之上。
<hr/>
## 一个可运行任何分布式应用程序的集群
ResourceManager、NodeManager 和容器都不关心应用程序或任务的类型。所有特定于应用程序框架的代码都转移到它的 ApplicationMaster，以便任何分布式框架都可以受 YARN 支持 — 只要有人为它实现了相应的 ApplicationMaster。
得益于这个一般性的方法，Hadoop YARN 集群运行许多不同工作负载的梦想才得以实现。想像一下：您数据中心中的一个 Hadoop 集群可运行 MapReduce、Giraph、Storm、Spark、Tez/Impala、MPI 等。
单一集群方法明显提供了大量优势，其中包括：
- 更高的集群利用率，一个框架未使用的资源可由另一个框架使用
- 更低的操作成本，因为只有一个 “包办一切的” 集群需要管理和调节
- 更少的数据移动，无需在 Hadoop YARN 与在不同机器集群上运行的系统之间移动数据

管理单个集群还会得到一个更环保的数据处理解决方案。使用的数据中心空间更少，浪费的硅片更少，使用的电源更少，排放的碳更少，这只是因为我们在更小但更高效的 Hadoop 集群上运行同样的计算。
<hr/>
## YARN 中的应用程序提交
本节讨论在应用程序提交到 YARN 集群时，ResourceManager、ApplicationMaster、NodeManagers 和容器如何相互交互。下图显示了一个例子。
### YARN 中的应用程序提交
![YARN 中的应用程序提交](/img/post_4/4.png)
假设用户采用与 MRv1 中相同的方式键入 hadoop jar 命令，将应用程序提交到 ResourceManager。ResourceManager 维护在集群上运行的应用程序列表，以及每个活动的 NodeManager 上的可用资源列表。ResourceManager 需要确定哪个应用程序接下来应该获得一部分集群资源。该决策受到许多限制，比如队列容量、ACL 和公平性。ResourceManager 使用一个可插拔的 Scheduler。Scheduler 仅执行调度；它管理谁在何时获取集群资源（以容器的形式），但不会对应用程序内的任务执行任何监视，所以它不会尝试重新启动失败的任务。
在 ResourceManager 接受一个新应用程序提交时，Scheduler 制定的第一个决策是选择将用来运行 ApplicationMaster 的容器。在 ApplicationMaster 启动后，它将负责此应用程序的整个生命周期。首先也是最重要的是，它将资源请求发送到 ResourceManager，请求运行应用程序的任务所需的容器。资源请求是对一些容器的请求，用以满足一些资源需求，比如：
* 一定量的资源，目前使用 MB 内存和 CPU 份额来表示
* 一个首选的位置，由主机名、机架名称指定，或者使用 * 来表示没有偏好
* 此应用程序中的一个优先级，而不是跨多个应用程序

如果可能的话，ResourceManager 会分配一个满足 ApplicationMaster 在资源请求中所请求的需求的容器（表达为容器 ID 和主机名）。该容器允许应用程序使用特定主机上给定的资源量。分配一个容器后，ApplicationMaster 会要求 NodeManager（管理分配容器的主机）使用这些资源来启动一个特定于应用程序的任务。此任务可以是在任何框架中编写的任何进程（比如一个 MapReduce 任务或一个 Giraph 任务）。NodeManager 不会监视任务；它仅监视容器中的资源使用情况，举例而言，如果一个容器消耗的内存比最初分配的更多，它会结束该容器。
ApplicationMaster 会竭尽全力协调容器，启动所有需要的任务来完成它的应用程序。它还监视应用程序及其任务的进度，在新请求的容器中重新启动失败的任务，以及向提交应用程序的客户端报告进度。应用程序完成后，ApplicationMaster 会关闭自己并释放自己的容器。
尽管 ResourceManager 不会对应用程序内的任务执行任何监视，但它会检查 ApplicationMaster 的健康状况。如果 ApplicationMaster 失败，ResourceManager 可在一个新容器中重新启动它。您可以认为 ResourceManager 负责管理 ApplicationMaster，而 ApplicationMasters 负责管理任务。
<hr/>
## 有趣的事实和特性
YARN 提供了多种其他的优秀特性。介绍所有这些特性不属于本文的范畴，我仅列出一些值得注意的特性：
+ 如果作业足够小，Uberization 支持在 ApplicationMaster 的 JVM 中运行一个 MapReduce 作业的所有任务。这样，您就可避免从 ResourceManager 请求容器以及要求 NodeManagers 启动（可能很小的）任务的开销。
+ 与为 MRv1 编写的 MapReduce 作业的二进制或源代码兼容性 (MAPREDUCE-5108)。
+ 针对 ResourceManager 的高可用性 (YARN-149)。此工作正在进行中，已由一些供应商完成。
+ 重新启动 ResourceManager 后的应用程序恢复 (YARN-128)。ResourceManager 将正在运行的应用程序和已完成的任务的信息存储在 HDFS 中。如果 ResourceManager 重新启动，它会重新创建应用程序的状态，仅重新运行不完整的任务。此工作已接近完成，社区正在积极测试。它已由一些供应商完成。
+ 简化的用户日志管理和访问。应用程序生成的日志不会留在各个从属节点上（像 MRv1 一样），而转移到一个中央存储区，比如 HDFS。在以后，它们可用于调试用途，或者用于历史分析来发现性能问题。
+ Web 界面的新外观。

<hr/>
## 结束语
YARN 是一个完全重写的 Hadoop 集群架构。它似乎在商用机器集群上实现和执行分布式应用程序的方式上带来了变革。
与第一版 Hadoop 中经典的 MapReduce 引擎相比，YARN 在可伸缩性、效率和灵活性上提供了明显的优势。小型和大型 Hadoop 集群都从 YARN 中受益匪浅。对于最终用户（开发人员，而不是管理员），这些更改几乎是不可见的，因为可以使用相同的 MapReduce API 和 CLI 运行未经修改的 MapReduce 作业。
没有理由不将 MRv1 迁移到 YARN。最大型的 Hadoop 供应商都同意这一点，而且为 Hadoop YARN 的运行提供了广泛的支持。如今，YARN 已被许多公司成功应用在生产中，比如 Yahoo!、eBay、Spotify、Xing、Allegro 等。
(本文转载，原文：[http://www.ibm.com/developerworks/cn/data/library/bd-yarn-intro/](http://www.ibm.com/developerworks/cn/data/library/bd-yarn-intro/))
