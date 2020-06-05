# 分布式共识算法 Raft

**Author:**[杭州电子科技大学](http://www.hdu.edu.cn/)  [唐涛](https://www.promiselee.cn/tao)
**CreateTime: **`**2020-6-5 14:28:07**`
**UpdateTime: `2020-6-6 01:08:38`**
**Copyright:**  唐涛 [HOME | 首页](https://www.promiselee.cn/tao) **©**  **2020**  ![](https://cdn.nlark.com/yuque/0/2020/svg/389953/1591376910320-8e9b819b-f2f3-4a9c-8b91-fb7defb538bf.svg#align=left&display=inline&height=45&margin=%5Bobject%20Object%5D&originHeight=45&originWidth=45&status=done&style=none&width=45)  ![](https://cdn.nlark.com/yuque/0/2020/svg/389953/1591376910399-07fb36e8-1b36-4d94-a519-f611fd92fd1c.svg#align=left&display=inline&height=35&margin=%5Bobject%20Object%5D&originHeight=35&originWidth=36&status=done&style=none&width=36)
**Email:**  [tangtao2099@outlook.com](mailto:tangtao2099@outlook.com) [16011324@hdu.edu.cn](mailto:16011324@hdu.edu.cn)
**Link:**  [知乎](https://www.zhihu.com/people/tang-tao-24-36/activities)  [GitHub](https://github.com/tangtaoshadow) [语雀](https://www.yuque.com/tangtao-frbji)
## 动机(Motivation)


现代计算基础架构变得越来越分散有很多原因，首先，当今的CPU时钟速度在很长时间内没有很大提升，而通常一台计算机可以处理的速度是有限的，随着我们需要处理越来越多的数据和流量，迫使我们使用多机协同计算来处理这些日益庞大的工作负载，从而为其提供了可伸缩性。 随着数据量和复杂性的不断增加以及数据更新速度的加快，大数据的商业价值和持续的技术进步助长了这一趋势。


通常，即使在负载变化和系统出现部分故障的情况下，服务仍可以保持高度可用。但由于维护或停机，导致停机变得无法接受。只有通过复制（使用多个冗余计算机）才能实现这种弹性：当一台计算机发生故障或过载时，另一台计算机可以接管给定的角色。为了提供抵御数据丢失的弹性，必须存储这些数据，并在所有复制的计算机上进行连贯更新。


❗另外，对于具有全球范围的服务，企业更希望缩短延迟，让请求由地理位置最接近的复制实例提供服务。


在大多数服务中，使用并需要三台（或更多，但通常是奇数）服务器来提供最佳的动态可伸缩性，通过复制的弹性数据存储以及低延迟，以实现最佳的质量和服务保证。为了使计算在分布式系统中给出合理的结果，必须确定地执行该计算，这在分布式系统中，说起来容易做起来难。


**如何确保任何两台计算机在任何给定时间都具有相同的状态？**


假设任意的网络延迟和可能的网络临时分区，这实际上从根本上是不可能的。在不同的折衷方案下，可以使用不同的较弱（服务基本可用）的保证，但最简单的保证是使主和副本之间最终彼此一致，这意味着如果没有新的写入操作，对于相同的请求，两台计算机最终将返回相同的值。


为了了解分布式计算中的确定性执行，必须区分不会相互干扰的操作（比如：简单的读取操作）和会干扰彼此的操作（比如：对同一变量的写操作），执行冲突操作的服务器需要以某种方式就一个共同的状态达成共识，即寻求共识。


多数的分布式服务器都建立在共识之上：容错复制状态机，分布式数据库和组成员系统都依赖于共识解决方案。实际上，就共识而言，即使是分布式系统中的另一个基本问题，即原子广播，也从根本上说是同一个问题，即达成共识。


总之，共识是使用分布式系统中基本通用的抽象：只要不超过一定数量的故障，共识就可以使所有参与服务器达成共识。从根本上讲，**共识的目标不是协商某种最佳值的目标**，而是在那一轮共识算法中，参与服务器之一先前就某个价值提出集体协议。在共识的帮助下，分布式系统可以像单个实体一样运行。


## 有哪些分布式共识算法?


- Paxos：被认为是分布式共识算法的根本，其他都是其变种，但是 paxos 论文中只给出了单个提案的过程，并没有给出复制状态机中需要的 multi-paxos 的相关细节的描述，实现 paxos 具有很高的工程复杂度（如多点可写，允许日志空洞等）Paxos 算法包括：Basic Paxos 和 Multi-Paxos。 Basic Paxos 算法描述的是多节点之间如何就某个值（提案 Value）达成共识； Multi-Paxos 思想描述的是执行多个 Basic Paxos 实例，就一系列值达成共识。
- Zab：被应用在 zookeeper 中，业界使用广泛，但没有抽象成通用的 library
- Raft：以容易理解著称，业界也涌现出很多 raft 实现，比如大名鼎鼎的 etcd, braft, tikv 等



## 分布式一致性


分布式一致性(`distributed consensus`) 是分布式系统中最基本的问题， 用来保证一个分布式系统的可靠性以及容灾能力。简单的来讲，就是如何在多个机器间对某一个值达成一致, 并且当达成一致之后，无论之后这些机器间发生怎样的故障，这个值能保持不变。


在抽象定义上， 一个分布式系统里的所有进程要确定一个值v，如果这个系统满足如下几个性质， 就可以认为它解决了分布式一致性问题:

| Termination | 所有正常的进程都会决定v具体的值，不会出现一直在循环的进程 |
| --- | --- |
| **Validity** | **任何正常的进程确定的值v', 那么v'肯定是某个进程提交的。比如随机数生成器就不满足这个性质** |
| **Agreement** | **所有正常的进程选择的值都是一样的** |



## Raft


Raft是一种新型易于理解的分布式一致性复制协议，由斯坦福大学的Diego Ongaro和John Ousterhout[提出](http://wiki.baidu.com/download/attachments/142056196/raft.pdf?version=1&modificationDate=1457941130000&api=v2)，作为[RAMCloud](https://ramcloud.atlassian.net/wiki/display/RAM/RAMCloud)项目中的中心协调组件。`Raft`是一种Leader-Based的Multi-Paxos变种，相比Paxos、Zab、View Stamped Replication等协议提供了更完整更清晰的协议描述，并提供了清晰的节点增删描述。它在容错和性能上与Paxos等效。不同之处在于它被分解为相对独立的子问题，并且干净地解决了实际系统所需的所有主要部分。


`Raft`作为复制状态机，是分布式系统中最核心最基础的组件，提供命令在多个节点之间有序复制和执行，当多个节点初始状态一致的时候，保证节点之间状态一致。系统只要多数节点存活就可以正常处理，它允许消息的延迟、丢弃和乱序，但是不允许消息的篡改（非拜占庭场景）。


## 基本原理


在深入探讨之前，让我们看一下Raft共识算法的一些基础知识。


Raft基于领导者驱动的共识模型，其中将选举一位**杰出**的领导者(Leader)，而该Leader将完全负责管理集群，Leader负责管理Raft集群的所有节点之间的复制日志。


为了了解Raft算法的基本术语，让我们先看一下五个节点之间的共识情况，并了解Raft如何实现它。


下图中，将在启动过程中选择集群的Leader（S1），并为来自客户端的所有命令/请求提供服务。 Raft集群中的所有节点都维护一个分布式日志（复制日志）以存储和提交由客户端发出的命令（日志条目）。 Leader接受来自客户端的日志条目，并在Raft集群中的所有关注者（S2，S3，S4，S5）之间复制它们。


![0_nEoP3_pCS6V3OcRL.png](https://cdn.nlark.com/yuque/0/2020/png/389953/1591372647703-717e462b-8aa1-409b-9f35-cb578a57daad.png#align=left&display=inline&height=704&margin=%5Bobject%20Object%5D&name=0_nEoP3_pCS6V3OcRL.png&originHeight=704&originWidth=971&size=34244&status=done&style=none&width=971#align=left&display=inline&height=704&margin=%5Bobject%20Object%5D&originHeight=704&originWidth=971&status=done&style=none&width=971)
在Raft集群中，需要满足最少数量的节点才能提供预期的级别共识保证， 这也称为法定人数。 在Raft集群中执行操作所需的最少投票数为 `(N / 2 +1)`，其中N是组中成员总数，即投票至少超过一半，这也就是为什么集群节点通常为奇数的原因。 因此，在上面的示例中，我们至少需要3个节点才能具有共识保证。


如果法定仲裁节点由于任何原因不可用，也就是投票没有超过半数，则此次协商没有达成一致，并且无法提交新日志。


Raft通过解决围绕Leader选举的三个主要子问题，管理分布式日志和算法的安全性功能来解决分布式共识问题。


### 领导选举 (Election)


当我们启动一个新的Raft集群或某个领导者不可用时，将通过集群中所有成员节点之间协商来选举一个新的领导者。 因此，在给定的实例中，Raft集群的节点可以处于以下任何状态: 追随者(Follower)，候选人(Candidate)或领导者(Leader)。


**Raft-node 的 3 种角色/状态**
![](https://gw.alipayobjects.com/mdn/rms_da499f/afts/img/A*EK6gQYwiBXkAAAAAAAAAAABjARQnAQ#align=left&display=inline&height=874&margin=%5Bobject%20Object%5D&originHeight=874&originWidth=1128&status=done&style=none&width=1128#align=left&display=inline&height=874&margin=%5Bobject%20Object%5D&originHeight=874&originWidth=1128&status=done&style=none&width=1128)


![1_tb8ZFnQCt5Qf9SQZwW6WLg.png](https://cdn.nlark.com/yuque/0/2020/png/389953/1591373194144-082b9146-5f1e-4e4d-8893-39bf2f76b154.png#align=left&display=inline&height=305&margin=%5Bobject%20Object%5D&name=1_tb8ZFnQCt5Qf9SQZwW6WLg.png&originHeight=255&originWidth=570&size=42595&status=done&style=none&width=682#align=left&display=inline&height=255&margin=%5Bobject%20Object%5D&originHeight=255&originWidth=570&status=done&style=none&width=570)


1. Follower：完全被动，不能发送任何请求，只接受并响应来自 leader 和 candidate 的 message，每个节点启动后的初始状态一定是 follower
1. Leader：处理所有来自客户端的请求，以及复制 log 到所有 followers
1. Candidate：用来竞选一个新 leader （candidate 由 follower 触发超时而来）



> 领导者负责管理Raft集群，并处理所有传入的客户端命令。跟随者是被动的，他们仅响应领导者或候选人的请求。 当Raft集群中没有领导者时，任何追随者都可以成为候选人，要求该群组中其他成员投票。



在Raft算法中，我们将时间划分为任期（Term）， 如图，除非原来的Leader节点变得不可用，否则每个任期均以选举开始，当选的领导者直至任期结束。


![1_6n94HwJoq0ccTBXk8Er0Sw.png](https://cdn.nlark.com/yuque/0/2020/png/389953/1591373478994-a3c685b1-f2fe-491d-acea-ba5947b5021f.png#align=left&display=inline&height=285&margin=%5Bobject%20Object%5D&name=1_6n94HwJoq0ccTBXk8Er0Sw.png&originHeight=204&originWidth=485&size=14927&status=done&style=none&width=678#align=left&display=inline&height=204&margin=%5Bobject%20Object%5D&originHeight=204&originWidth=485&status=done&style=none&width=485)


如果在选举期间，两名候选人可能会获得相同的票数（分开投票的情况，通常算法设计上会让**每个Follower随机等待一段时间**，再发起投票），这时需要立即开始新的选举。 Term表示为随时间单调增加的数字，使用Term值在节点之间进行通信， Raft集群中的所有这些节点都使用远程过程调用（RPC）进行通信。




### 开始选举


Raft使用基于心跳的RPC机制来检测何时开始新的选举。 在正常期间，**Leader**会定期向所有可用的**Follower**发送心跳消息（实际中可能把日志和心跳一起发过去）。 因此，其他节点以**Follower**状态启动，只要它从当前**Leader**那里收到周期性的心跳，就一直保持在**Follower**状态。
![0_7rg29MHNkTFNb3Ms.png](https://cdn.nlark.com/yuque/0/2020/png/389953/1591374019981-a76ed2e9-b4e5-4244-af04-a7297aa6d414.png#align=left&display=inline&height=509&margin=%5Bobject%20Object%5D&name=0_7rg29MHNkTFNb3Ms.png&originHeight=509&originWidth=539&size=30725&status=done&style=none&width=539#align=left&display=inline&height=509&margin=%5Bobject%20Object%5D&originHeight=509&originWidth=539&status=done&style=none&width=539)
在实际中，这些心跳是基于**Leader**用来向**Follower**发送新日志条目的相同消息类型， 因此，**Leader**将没有携带日志条目的RPC当做单纯的心跳消息。


如果追随者在一段时间内未收到心跳消息，就会触发选举超时，该选举超时将启动新的选举以选择领导者。
当**Follower**达到其超时时间时，它将通过以下方式启动选举程序：


- 增加当前Term，
- 为自己投票，并将“ RequestVote” RPC发送给集群中的所有其他人，这时也就是从**Follower**转换为**Candidate**



如图，关注者S1首先达到了选举超时时间，然后成为候选者。 然后，它请求其他节点投票作为领导者。


   ![](https://cdn.nlark.com/yuque/0/2020/png/389953/1591374377185-dff1c948-8ffb-4119-8fa4-b008abe8cad1.png#align=left&display=inline&height=579&margin=%5Bobject%20Object%5D&originHeight=507&originWidth=553&status=done&style=none&width=632)


根据**Candidate**从集群中其他节点收到的响应，可以得出选举的三个结果。


- 如果大多数节点以“是”投票支持RequestVote请求，则候选人S1赢得选举。
- 在S1等待期间，它可能会从另一个声称是领导者的节点接收AppendEntries RPC。 如果S1的候选Term低于AppendEntries RPC的接收Term，则候选S1放弃并接受另一个节点作为合法领导者。
- 拆分投票方案：当有多个**Follower**同时成为**Candidate**时，任何候选人都无法获得多数。 这被称为分裂投票情况。 在这种情况下，每个**Candidate**都将超时，并且将触发新的选举。



> 🛑 为了最大程度地减少拆分投票的情况，Raft使用了随机选举超时机制，该机制将随机超时值分配给每个节点。



### 日志复制 (Log Replication)


一旦节点成为**Leader**，它就可以从客户端接收命令/日志条目。 使用AppendEntries RPC发送日志条目


![0_0qwTOWUxevWKigsv.png](https://cdn.nlark.com/yuque/0/2020/png/389953/1591374656240-e9b5cfeb-e823-403c-8f02-383dd9add22a.png#align=left&display=inline&height=644&margin=%5Bobject%20Object%5D&name=0_0qwTOWUxevWKigsv.png&originHeight=509&originWidth=539&size=31549&status=done&style=none&width=682#align=left&display=inline&height=509&margin=%5Bobject%20Object%5D&originHeight=509&originWidth=539&status=done&style=none&width=539)
从客户端收到命令后，**Leader**将为命令分配Term和日志索引Index。 然后，**Leader**尝试在集群中的大多数节点上执行复制命令。 如果复制成功，则将命令提交给集群，并将响应发送回客户端。
在下图中，显示了具有五个节点的Raft集群的示例分布式日志。
图中每一行代表给定节点的日志，并对日志进行索引以唯一标识**Leader**在其余节点上接收和复制的每个命令。
该命令使用语法（x←value）以及前导词的关联Term表示（为清晰起见，此处每个Term用不同的颜色表示）。
Term编号用于标识日志的不一致。


![0_gxPiMLOFrGDpiufd.png](https://cdn.nlark.com/yuque/0/2020/png/389953/1591374825993-b1b56ae7-80fb-49bc-b684-b5b66acf845d.png#align=left&display=inline&height=649&margin=%5Bobject%20Object%5D&name=0_gxPiMLOFrGDpiufd.png&originHeight=649&originWidth=760&size=73064&status=done&style=none&width=760#align=left&display=inline&height=649&margin=%5Bobject%20Object%5D&originHeight=649&originWidth=760&status=done&style=none&width=760)


让我们尝试看看从客户端发送到领导者的新命令（y←9）时日志复制的工作方式。 因此，见下图，在接收到新命令时，所有节点的状态都不一致，因为它们具有相同的日志条目，并在2处提交索引。
**Leader**将命令附加到日志，并使用该命令广播AppendEntries RPC。


- 每个节点都在本地申请条目并成功答复。
- 当大多数跟随者节点已在本地成功提交日志条目时，**Leader**将提交（前一阶段相当于Try，接下来是Commit，类似两阶段提交协议）命令并将成功响应发送回客户端。
- 当领导者提交日志条目时，它还会更新提交索引（在此示例中为3），并且下一条AppendEntries广播消息会将更新的提交索引复制到所有跟随者节点。
- 当领导者提交一个条目时，它还将在当前日志索引之前提交所有全部内容。



![0_a1xquMOMleD4xRH_.png](https://cdn.nlark.com/yuque/0/2020/png/389953/1591375043420-d90a351d-81e8-4865-a3d3-fac4728e5703.png#align=left&display=inline&height=258&margin=%5Bobject%20Object%5D&name=0_a1xquMOMleD4xRH_.png&originHeight=543&originWidth=1572&size=99533&status=done&style=none&width=746#align=left&display=inline&height=543&margin=%5Bobject%20Object%5D&originHeight=543&originWidth=1572&status=done&style=none&width=1572)
如果某些节点不可用于接收日志条目，或者消息在运行中丢失，则日志中可能存在不一致之处。**Leader**负责调和此类不一致。 当**Follower**收到AppendEntries RPC时，它也包含Term和上一个日志条目的日志索引。 它与**Follower**日志的最后一个条目不匹配，那么**Follower**将发送失败的响应。 因此，**Leader**知道该特定节点的日志中存在不一致之处。
领导者通过跟踪每个**Follower**的nextIndex来解决日志不一致问题。 当给定的**Follower**的日志不一致时（即，如果发送的响应失败），则**Leader**递减nextIndex值，然后重试AppendEntries RPC。 此过程将继续进行，直到**Follower**日志与领导者一致为止。 此外，每个节点还将本地日志保存在持久存储中。


### Message 的 3 种类型


上面提到了RPC通信，通常节点之间的通信主要由以下三种构成：


1. RequestVote RPC：由 candidate 发出，用于发送投票请求
1. AppendEntries (Heartbeat) RPC：由 leader 发出，用于 leader 向 followers 复制日志条目，也会用作 Heartbeat （日志条目为空即为 Heartbeat）
1. InstallSnapshot RPC：由 leader 发出，用于快照传输，虽然多数情况都是每个服务器独立创建快照，但是leader 有时候必须发送快照给一些落后太多的 follower，这通常发生在 leader 已经丢弃了下一条要发给该follower 的日志条目(Log Compaction 时清除掉了) 的情况下



### 联合共识


当集群中节点的状态发生变化（集群配置发生变化）时，系统容易受到系统故障。 因此，为防止这种情况，Raft使用了一种称为两阶段的方法来更改集群成员身份。 因此，在这种方法中，集群在实现新的成员身份配置之前首先更改为中间状态（称为联合共识）。 联合共识使系统即使在配置之间进行转换时也可用于响应客户端请求，它的主要目的是提升分布式系统的可用性。


### 安全


为了确保上述领导者选举和日志复制在所有条件下都能正常工作，我们需要为Raft算法增加一些安全条件。


#### 选举限制-“领导的额外条件”


选举限制条件：要求候选人的日志至少与其他节点一样最新。如果不是，则跟随者节点将不投票给候选者。
这意味着每个提交的条目都必须存在于这些服务器中的至少一个中。如果候选人的日志至少与该多数日志中的其他日志一样最新，则它将保存所有已提交的条目。
因此，RequestVote RPC包含有关候选人日志的信息，如果投票者自己的日志比候选人的日志更新，则投票者将拒绝投票。


#### 以前条款中的承诺条目-“承诺的附加条件”


如果同一领导者已成功复制了当前Term的条目，则此条件是限制该领导者仅提交任何先前Term的条目。


#### 当涉及分布式共识时，Raft算法可提供以下保证


- **选举安全**：在给定的情况下最多可以选举一位领导人Term
- **Leader Append-Only**：领导者永远不会覆盖或删除
- **日志中的条目**：它仅附加新条目
- 日志匹配：如果两个日志包含相同条目：索引和Term，那么直到并包括给定索引的所有条目中的日志都是相同的，即只要索引和Term相同，日志就是相同的。
- **领导者完整性**：如果在给定的期限内提交了日志条目，则该条目将出现在**Leader**的日志中，用于比所有编号更高的Term。
- **状态机安全**：如果节点已将给定索引的日志条目应用于其状态机，则其他节点将不会为同一索引应用不同的日志条目。
- **跟随者节点崩溃** ：**Follower**节点崩溃时，发送到该**Follower**节点的所有请求都将被忽略。此外，崩溃的**Follower**节点无法参加**Leader**选举。节点重新启动后，它将与**Leader**节点同步其日志。



## Raft的局限性


- Raft严格是单个**Leader**协议，而且太多的流量会阻塞系统，存在解决此瓶颈的Paxos算法的某些变体。
- 有许多假设在起作用，例如未发生拜占庭故障，这就降低了现实生活中的适用性。
- Raft是针对达成共识时出现的问题的**子集**的一种更专业的方法。
- Raft在服务器集群中只有一个节点在运行时也可以工作。概括地说，K + 1的复制服务器可以容忍K服务器中的关闭/故障。
