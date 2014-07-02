#Making Reliable Distributed Systems

#Erlang 简介
>Erlang 属于一种纯消息传递语言——即一种基于独立性很强的并行进程的语言，我们的  
编程模型广泛使用了速错（fail-fast）进程。这项技术在构建可容错系统的硬件   
平台中被普遍使用，但是在软件设计方面却用得不多。这主要是因为传统的编程  
语言并不允许不同的软件模块以彼此互不干扰的方式存在。当前普遍使用的是多  
线程的编程模型，该模型中资源是共享的，这样就造成了线程很难真正隔离起来，  
就可能导致一个线程中的错误会传播到另一个线程中，这就破坏了系统内部的坚  
固性。  
>

>Erlang 是一门“面 向并发编程”（Concurrency Oriented Programming， COP）


#怎么把隔离错误
>我们怎么才能够构建出在软件存在错误的时候具有合理行为的可容错的软 
件系统呢？这是本论文余下部分要回答的问题。我先给出一个简洁的答案，在本  
文的剩余部分会对其进行细化。  
为了构建出在软件存在错误的时候仍具有合理行为的可容错软件系统，我们  
>


*通常要求在 40 年里停机时间不超过 2 小时


*我们将软件组织成一个系统要完成的任务的层次结构，每一个任务对应
于一组目标，具有给定任务的软件必须尝试去完成和该任务相关的目
标。
所有任务按照复杂性排序。最顶层的任务最复杂。如果最顶层任务
完的目标都被完成，那么整个系统就运转正常。较低层次的任务应当能
够保持系统以一种可接受的方式运转，即使系统所提供的服务有所折
扣。 
系统中低层任务较高层任务更容易完成其目标。


*我们将尽力完成顶层的任务。


* 当在完成某一目标的过程中检测到了一个错误，我们将尝试纠正这个错
误。当我们不能够纠正该错误的时候，我们将立即取消当前的任务而启
动一个更简单一些的任务。


>
编写这样一个任务层次需要一套强有力的封装方法。我们需要强有力的封装
方法来隔离错误。我们不想再去编写那种系统中的一个部分发生的错误会对其他
部分产生不利影响的系统。
我们需要以一种能够检测到在试图完成目标时所发生的所有错误的方式，来
隔离为了完成某一目标而编写的所有代码。并且，当我们在试图同时完成多个目
标时，我们不希望系统中某个部分所发发生的错误，会传播到系统的另外一个部
分中。
因此，在构建可容错软件系统的过程中要解决的本质问题就是故障隔离。不
同的程序员会编写不同的模块，有的模块正确，有的存在错误。我们不希望有错
误的模块对没有错误的模块产生任何不利的影响。
为了提供这种故障隔离机制，我们采用了传统操作系统中进程的概念。进程
提供了保护区域，一个进程出错，不会影响到其他进程的运行。不同程序员编写
的不同应用程序分别跑在不同的进程中；一个应用程序的错误不会对系统中运行
的其他应用程序产生副作用。
这种选择当然满足了初步的要求。然而因为所有进程使用同一片 CPU、同
一块物理内存，所以当不同进程争抢 CPU 资源或者使用大量内存的时候，还是
可能对系统中的其他进程产生负面影响。进程间的相互冲突程度取决于操作系统
的设计特性。
在我们的系统中，进程和并发编程是语言的一部分，而不是由宿主操作系统
提供的。这样做比直接采用操作系统进程拥有很多优势：
>

*并发程序可以一致地运行在不同的操作系统上——不同的特定操作系
中是如何实现进程的不会对我们造成限制。我们的程序运行在不同的操
作系统和处理器上唯一可见的差异就是 CPU 的处理速度和内存的大小。
所有的同步问题和进程间通信都应当跟宿主的操作系统的特性没有一
点关系。

* 我们这种基于语言的进程比传统的操作系统进程要轻量得多。在我们的
语言里，创建一个进程是非常高效的，要比大多数操作系统中进程的创
建快几个数量级[12,14]，甚至比大多数语言中线程的创建都快几个数量
级。

* 我们的系统对操作系统的要求非常少。我们只用了操作系统很小的一部
分服务，所以把我们的系统移植到譬如嵌入式系统等特定环境下是相当
简单的。

>
我们的应用程序是通过大量互相通信的并行进程构建起来的。我们采用这种
方式是因为：
>

* 它提供了一个架构基础设施——我们可以用一组相互通信的进程组织
起我们的系统。通过枚举出系统中的所有进程，并定义出进程间消息传
递的通道，我们就可以很方便地把系统划分成定义良好的子部件，并可
以对这些子部件进行单独实现和测试。这种方法学也是 SDL[45]系统设
计方法学的最高境界。

* 巨大的潜在效率——设计成以许多独立的并行进程来实现的系统，可以
很方便地实现在多处理器上，或者运行在分布式的处理器网络上。注意，
这种效率的提升只是潜在的，只有当应用程序可以被分解成许多真正独
立的任务时，才能产生实效。如果任务之间有很强的数据依赖，这种提
升往往是不可能的。

* 故障隔离——没有共享数据的并发进程提供了一种强大的故障隔离方
法。一个并发进程的软件错误不会影响到系统中其他进程的运行。