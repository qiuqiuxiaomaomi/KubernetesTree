# KubernetesTree
Kubernetes容器技术研究

<pre>
Kubernetes简介

Kubernetes 是一个全新的基于容器技术的分布式架构领先方案，它是谷歌十几年以来大规模应用容器
技术的经验积累和升华的重要成果，确切的说，Kubernetes是谷歌严格保密十几年的秘密武器--Borg
的开源版本，Borg是谷歌的一个久负盛名的内部使用的大规模集群管理系统，它基于容器技术，目的是
实现资源管理的自动化，以及跨多个数据中心的资源利用率的最大化。

Kubernetes是一个开放的平台，不局限于任何一种语言。
Kubernetes是一个完备的分布式系统支撑平台。Kubernetes具有完备的集群管理能力，包括多层次的
安全防护和准入机制，多组合应用支撑能力，透明的服务注册和服务发现机制，内建智能负载均衡器，
强大的故障发现和自我修复能力，服务滚动升级和在线扩容能力，可扩展的资源自动调度机制，以及
多粒度的资源配额管理能力，同时Kubernetes提供了完善的管理工具，这些工具覆盖了包括开发，
部署测试，运维监控在内的各个环节，因此Kubernetes是一个全新的基于容器技术的分布式架构解
决方案，并且是一个一站式的完备的分布式系统开发和支撑平台。

在Kubernetes中，Service是分布式集群的核心，一个Service对象拥有如下关键特征。
    1）拥有一个唯一指定的名字
    2）拥有一个虚拟IP
    3）能够提供某种远程服务能力
    5）被映射到了提供这种服务能力的一组容器应用之上。

    Service的服务进程目前都基于Socket通信方式对外提供服务，比如Redis, Memcache, Mysql, Web Server或者实现了某个具体业务的一个特定TCP Server进程。

    虽然一个Service通常由多个相关的服务进程提供服务，每个服务进程都有一个独立的
    Endpoint(IP + Port)访问点，但Kubernetes能够让我们通过Service（虚拟IP + 服务端口）
    连接到指定的Service上，有了K8s内建的透明负载均衡和故障恢复机制，不管后端有多少服务进
    程，也不管某个服务进程是否会由于发生故障而重新部署到其他机器，都不会影响到我们队服务的
    正常调用。

    容器提供了强大的隔离功能，所有有必要把为Service提供服务的这组进程放入容器中进行隔离，
    为此K8s设计了Pod对象，将每个服务进程包装到相应的Pod中，使其成为Pod中运行的一个容
    器(Container)，为了建立Service和Pod间的映射关系，K8s首先给每个Pod贴上一个标签，
    然后给相应的Service定义标签选择器（Label Selector），比如Mysql Service的标签选
    择器的选择条件为name = mysql，意为该Service要作用于所有包含name = mysql Label的
    Pod上，这样巧妙的解决了Service与Pod的关联问题。

    Pod运行在一个我们称之为节点的环境中，这个节点既可以是物理的，也可以是私有云或者公有
    云中的一个虚拟机，通常在一个节点上运行几百个Pod;其次每个Pod里运行着一个特殊的被称之
    为Pause的容器，其他则为业务容器，这些业务容器共享Pause容器的网络栈和Volume挂载卷，
    因此它们之间的通信和数据交换更为高效，在设计时，我们可以充分利用这一特性将一组密切相
    关的服务进程放入同一个Pod中；最后，需要注意的是，并不是每个Pod和它里面运行的容器都
    能“映射到”一个Service上，只有那些提供服务的一组Pod才会被映射成一个服务。


    在集群管理方面，K8s将集群中的机器划分为一个Master节点和一群工作节点(Node)，其中，在Master节点上运行着集群管理相关的一组进程 kube-apiserver, kube-controller-manager和kube-scheduler，这些进程实现了整个集群的资源管理，Pod调度，弹性伸缩，安全控制，系统监控，和纠错等管理功能，并且全都是自动完成的，Node作为集群中的工作节点，运行真正的应用程序，在Node上，K8s管理的最小单元是Pod,Node上运行着K8s的kubelet,kube-proxy服务进程，这些服务进程负责Pod的创建，启动，监控，重启，销毁，以及实现软件模式的负载均衡。

    在K8s集群中，你只需为需要扩容的Service关联的Pod创建一个Replication Controller（简称RC），则该Service的扩容以至于后来的Service升级等头疼问题都将迎刃而解。在一个RC定义文件中包括以下3个关键信息
         1）目标Pod的定义
         2）目标Pod需要运行的副本数量
         3) 要监控的目标Pod的标签
         在创建好RC(系统自动创建Pod)后，K8s会通过RC中定义的Label筛选出对应的Pod实例并实时监控器状态和数量，如果实例数量少于定义的副本数量，则会根据RC中定义的Pod模板来创建一个新的Pod,然后将此Pod调度到合适的Node上启动运行，直到Pod实例的数量达到预定目标，这个过程完全是自动化的，无需人工干预。有了RC，服务的扩容就编程了一个纯粹的数字游戏了，只要修改RC中副本的数量即可，后续的Service升级也将通过RC来自动完成。
</pre>

<pre>
为什么使用Kubernetes
   K8s作为当前唯一被业界广泛任何和看好的Docker分布式系统解决方案，K8s有什么好处呢？
   1）轻装上阵
   2）K8s全面拥抱微服务架构。微服务架构的核心是将一个巨大的单体应用拆分为很多小的互相连接的微服务，一个微服务背后可能有多个实例副本在支撑，副本的数量可能会随着系统的负荷变化而进行调整，内嵌的负载均衡器在这里发挥了重要作用，微服务架构使得每个服务都可以由专门的开发团队来开发，开发者可以自由选择开发 技术，这对于大规模团队来说很有价值，另外每个微服务独立开发，升级，扩容，因此系统具备很高的稳定和快速迭代进化能力，
   3）业务系统可以随时随地的整体搬迁到公有云上，K8s最初的设计目标就是运行在Google自家的公有云中，未来会支持更多的公有云及基于OpenStack的私有云，同时在K8s的架构方案中，底层网络的细节被完全屏蔽，基于服务的ClusterIP甚至都无须我们改变运行期的配置文件，就能将系统从屋里环境无缝迁移到公有云平台中，或者在服务高峰期将部分服务对应的Pod副本放入公有云中以提升系统的吞吐量，不仅节省了公司的硬件投入，还大大改善了客户体验。
   5）K8s系统架构具备了超强的横向扩容能力
</pre>

<pre>
Master
     K8s里的Master指的是集群的控制节点，每个K8s集群里需要有一个Master节点来负责整个集群的管理和控制，基本上，K8s所有的空值命令都是发送给它，它来负责具体的执行过程，我们后面所有执行的命令基本上都是在Master节点上运行的，Master节点通常占据一个独立的X86服务器（或者一个虚拟机），一个主要的原因是它太重要了，它是整个集群的首脑，如果宕机或者不可用，那么我们所有的控制命令都将失效。
     Master节点上运行着一组关键进程
         1）Kubernetes API Server（kube-apiserver）提供了HTTP REST接口的关键服务进程，是Kubernetes里所有资源的增删改查等操作的唯一入口，也是集群控制的入口进程。
         2）Kubernetes Controller Manager（kube-controller-manager），K8s里所有资源对象的 自动化控制中心，可以理解为资源对象的“大总管”
         3) Kubernetes Scheduler（kube-scheduler）负责资源调度（Pod调度）的进程，相当于公交公司的调度室
         其实Master节点上往往还启动了一个etcd Server进程，因为K8s里的所有资源对象的数据全部保存在etcd中的。
</pre>

<pre>
Node 
    除了 Master节点，K8s集群中的其他机器被称为Node节点，在较早的版本中被称为Minion，与Master一样，Node节点可以是一台物理主机，也可以是一台虚拟机，Node节点才是K8s集群中的工作节点，每个Node都会被Master分配一些工作负载，当某个额Node宕机时，其上的工作负载会被Master自动转移到其他节点上去。
    每个Node节点上都运行着一下一组关键进程
       1）kubelet: 负责Pod对应的容器的创建，启停等任务，同时与Master节点密切协作，实现集群管理的基本功能
       2) kube-proxy: 实现Kubernetes Service的通信与负载均衡机制的重要组件
       3）Docker Engin： Docker引擎，负责本机的容器创建与负责均衡机制的重要组件

       Node节点可以再运行期间动态增加到K8s集群中，前提是这个节点已经正确安装，配置，启动了上述关键进程，在默认情况下，kubelet会向Master注册自己，这也是Kubernetes推荐的Node管理方式，一旦Node被纳入集群管理方位，kubelet进程就会定时向Master节点汇报自身的情报，例如操作系统，Docker版本，机器的CPU和内存情况，以及之前有哪些Pod在运行等，这样Master可以获知每个Node的资源使用情况，并实现高效的负载均衡的资源调度策略，而某个Node超过指定时间不上报信息时，会被Master判定为失联，Node的状态被标记为不可用，随后Master触发工作负载大转移的自动流程。

       查看集群中有多少个Node
       #kubectl get nodes
       查看Node详细信息
       $ kubectl describe node ''''''
</pre>

![](https://i.imgur.com/GQpfBPP.png)

<pre>
Pod
   Pod是K8s的最重要也是最基本的概念。
   每个Pod都有
       1）一个特殊的被称为“根容器”的Pause容器
       2）一个或者多个紧密相关的用户业务容器
   K8s之所以设计出一个全新的Pod的概念并且Pod有这样特殊的组成结构
       1）原因之一： 在一组容器作为单元的情况下，我们难以对“整体”简单地进行判断及有效地进行活动。比如一个容器死亡了，此时算是整体死亡？是N/M的死亡率么？引入业务无关且不易死亡的Pause容器作为Pod的根容器，以它的状态代表整个容器组的状态，就简单巧妙的解决了这个问题。
       2）原因之二：Pod里的多个业务容器共享Pause容器的IP，共享Pause容器挂接的Volume，这样既简化了密切关联的业务容器之间的通信问题，也很好了解决了他们之间的文件共享问题。
          K8s为每个Pod都分配了唯一的IP地址，称之为Pod IP,一个Pod里的多个容器共享Pod IP地址，K8s要求底层网络支持集群内任意两个Pod之间的TCP/IP直接通信，这通常采用虚拟二层网络技术来实现，例如Flannel, Openvswitch等。
         
   Pod分类
       1）普通的Pod
       2) 静态的Pod
   其中静态的Pod并不存放在K8s的etcd存储里，而是存放在某个具体的Node上的一个具体文件中，并且只在此Node上启动运行，而普通的Node一旦被创建爱你，就会被放入到etcd中存储，随后会被K8s Master调度到买某个具体的Node上并进行绑定，随后该Pod被对应的Node上的kubelet进程实例化成一组相关的Docker容器并启动起来，在默认情况下，当Pod里的某个容器停止时，K8s会自动检测到这个问题并且重新启动这个Pod（重启Pod里的所有容器），如果Pod所在的Node宕机，则会将这个Node上所有的Pod重新调度到其他节点上。    
</pre>

<pre>
Label（标签）
   Label Selector
   使用Label可以给对象创建爱你多组标签，Label和Label Selector共同构成了K8s系统中最核心的应用模型，是的被管理对向能够被精细地分组管理，同时实现整个集群的高可用性。
</pre>

<pre>
Repliction Controller
     删除RC并不会删除通过该RC已创建好的Pod,为了删除所有Pod,可以设置replicas的值为0，然后再更新该RC,另外,kubectl提供了stop和delete命令来一次删除RC和RC控制的全部Pod

     当应用升级时，通常会通过Build一个新的Docker镜像，并用新的镜像版本来替代旧的版本的方式来达到目的。在系统升级的过程中，我们希望的是平滑的方式，比如当前系统中10个对应的旧版本的Pod,最佳的方式是旧版本的Pod每次停止一个，同时创建一个新版本的Pod，在整个升级的过程中，此消彼长，而运行中的Pod数量始终是10个，几分钟后，当所有的Pod都已经是最新的版本的时候，升级过程完成，通过RC的机制，K8s很容易就实现了这种高级实用的特性，被称为滚动升级，
</pre>

<pre>
Horizontal Pod AutoScale（HPA 横向扩容）
    通过手工执行kubectl scale命令，我们可以实现Pod的扩容和缩容。
    原理： 通过追踪分析RC控制的所有目标Pod的负载变化情况，来确定是否需要针对性地调整目标Pod的副本数，这是HPA的实现原理。
    当前HPA有一下两种方式作为Pod负载的度量指标。
        1)CPUUtilizationPercentage
        2)应用程序自定义的度量指标，比如服务在每秒内的相应请求数（TPS, QPS）

    CPUUtilizationPercentage是一个算数平均值，即目标Pod所有副本吱声的CPU利用率的平均值。
    如果定义一个Pod的Pod Request为0.4，而当前CPU的利用率为0.2，则它的CPU使用率为50%
    如果某一时刻CPUUtilizationPercentage的值超过80%，则意味着当前的Pod副本数很可能不足以支撑下来更多的请求，需要进行动态扩容，而当请求高峰时段过去后，Pod的CPU利用率又会降下来，此时对应的Pod副本数应该自动减少到一个合理的水平。

    CPUUtilizationPercentage计算过程使用到的Pod的CPU使用量通常是1分钟内的平均值，目前通过查询Heapster扩展组件来得到这个值，所以需要安装部署Heapster，这样一来便增加了系统的复杂度和实施HPA特性的复杂度，因此未来的计划是K8s自身实现一个基础性能采集模块，从而更好的支持HPA和其他需要用到基础性能数据的功能模块。
</pre>

![](https://i.imgur.com/pRR44Bb.png)

<pre>
Service服务
   Service也是K8s里最核心的资源对象之一，K8s里的每个Service其实就是我们经常提起的微服务架构中的一个微服务。 

   K8s的Service定义了一个服务的访问入口地址，前端的应用（Pod）通过这个入口地址访问其背后的一组由Pod副本组成的集群实例，Service与其后端的Pod副本集群则是通过Label Selector来实现“无缝对接”的，而RC的作用实际上是保证Service的服务能力和服务质量始终处于预期的标准。

   通过分析，识别并建模系统中的所有服务未微服务---Kubernetes Service,最终我们的额系统由多个提供不同业务能力而又彼此独立的微服务单元所组成，服务之间通过TCP/IP进行通信，从而形成了我们强大而又灵活的弹性网格，拥有了强大的分布式能力，弹性扩展能力，容错能力，于此同时，我们的额程序架构也变得简单和直观许多。

   既然每个Pod都会被分配了一个单独的IP地址，而且每个Pod都提供了一个独立的Endpoint以被客户端访问，现在多个Pod副本组成了一个集群来提供服务，那么客户端如何来访问它们呢？一半的做法是部署一个负载均衡器（软件或者硬件）,为这组Pod开启一个对外的服务端口，并且将这些Pod的EndPoint列表加入这个端口的转发列表中，客户端就可以通过负载均衡器的对外IP地址+端口来访问次服务，而客户端的请求最后被转发到那个Pod，由负载均衡器决定。

   Kubernetes也遵循了上述常规做法，运行在每个Node上的kube-proxy进程其实就是一个智能的软件负载均衡器，它负责把对Service的请求转发到后端的某个Pod实例上，并在内部实现服务的负载均衡与会话保持机制，但K8s发明了一种很巧妙又影响深远的设计：Service不是共用一个负载均衡器的IP地址，而是每个Service分配了一个全局唯一的虚拟IP地址，这个虚拟IP地址被称为Cluster IP,这样一来，每个服务就变成了具备唯一IP地址的通信节点，服务调用就编程了最基础的TCP网络通信问题。

   Pod的Endpoint地址会随着Pod的销毁和重新创建而发生改变，因为新的Pod的IP地址与以前旧的Pod不同，而Service一旦创建，K8s就会自动为它分配一个可用的Cluster IP,而且在Service的整个生命周期内，它的Cluster IP不会发生改变，于是服务发现这个棘手的问题再K8s的架构里也得以轻松解决，只需要Service的Name与Service的Cluster IP地址做一个DNS域名映射即可完美解决这一问题。
</pre>

![](https://i.imgur.com/v5olSl3.png)

<pre>
K8s的服务发现机制
   首先，每个K8s中的Service都有一个唯一的Cluster IP以及唯一的名字，而名字是由开发者自己定义的，部署的时候没必要改变，所以完全可以固定在配置中，接下来的问题就是如何通过Service的名字找到对应的Cluster IP?
   最早的时候K8s采用了Linux环境变量的方式解决这个问题，即每个Service生成一个对应的Linuxt环境变量，并在每个Pod的容器启动时，自动注入这些环境变量。
   环境变量的方式不够直观，后来K8s通过Add-On增值包的方式引入DNS系统，把服务名称作为DNS域名，这样一来，程序就可以直接使用服务名来建立通信连接了，目前K8s上的大部分应用都已经采用了DNS这些新兴的服务发现机制。

   外部系统访问Service的问题
   三种IP
       1) Node IP: Node节点的IP
       2）Pod IP: Pod的IP
          虚拟的二层网络，Pod的IP是Docker Engine根据docker0网桥的IP地址段进行分配的
       3）Cluster IP: Service 的IP
          1）Cluster IP仅仅作用于Kubernetes Service这个对象，并由K8s管理和分配IP地址
          2）Cluster IP无法ping通，因为没有一个实体网络对象来相应
          3）Cluster IP只能结合Service port组成一个具体的通信端口，单独的Cluster IP不具备通信的基础。
          5）在K8s集群内部，Node IP网络，Pod IP网，Cluster IP之间的通信采用的是K8s自己设计的一种编程方式的特殊的路由规则，与我们所熟知的IP路由有很大的不同
</pre>

<pre>
Volume(存储卷)
    Volume是Pod中能够被多个容器访问的共享目录，K8s中的卷的概念，用户，目的与Docker中的卷比较类似，但是两者不能等价，首先K8s中的Volume定义在Pod上，然后被一个Pod里的多个容器挂在到具体的文件目录下，其次K8s中的Volume与Pod的生命周期相同，但是与容器的生命周期不相关，当容器终止或者重启时，Volume中的数据也不会丢失，最后K8s支持多种类型的Volume，例如GlusterFS, Ceph等先进的分布式文件系统。
</pre>

<pre>
Namespace（命名空间）
   命名空间是K8s系统中的另一个非常重要的概念，Namespace在很多情况下用于实现多租户的资源隔离，Namespace通过将集群内部的资源对象分配到不同的Namespace中，新城逻辑上分组的不同项目，小组，用户组，便于不同的分组在共享使用整个集群的资源的同时还能被分别管理。

   K8s在启动后，会创建一个名为default的Namespace，通过kubectl可以查看到
   $ kubectl get namespace
   接下来如果特别指明Namespace，则用户创建的Pod，RC，Service都将被系统创建到这个默认的名为default的Namespace中。
   当我们给每个租户创建一个Namespace来实现多租户的资源隔离时，还能结合K8s的资源配额管理，限定不同租户能占用的资源，例如CPU使用量，内存使用量等。
</pre>

<pre>
Annotation（注解）
    Annotion与Label类似，也使用key/value键值对的形式进行定义，不同的是Label具有严格的命名规则，它定义的是K8s对象的元数据，并且勇于Label Selector，而Annotation则是用户任意定义的附加信息，以便于外部工具进行查找，很多时候，K8s的模块吱声通过Annotation的方式标记资源对象的一些特殊信息。
    一般来说Annotation用来标记的信息如下
        1）build信息，release信息，Docker镜像信息，例如时间戳，release id号，PR号，镜像HASH值，docker registry地址等
        2）日志库，监控库，分析库等资源库的地址信息
        3）程序调式工具信息，例如工具名称，版本号等
        5）团队的联系信息，例如电话号码，负责人名称，网址等。
</pre>

![](https://i.imgur.com/zUe1Qlp.png)

![](https://i.imgur.com/EcLLMO1.png)

<pre>
K8s的安装与配置
   K8s系统由一组可执行程序组成，Github上的K8s项目页下载编译好的二级制包，或者瞎子源代码并编译后进行安装。
   最简单的安装方法是使用yum install kubernetes命令完成K8s集群的安装，但仍需要修改各组件的启动参数，才能完成K8s集群的配置。 

   在K8s Master节点安装部署etcd, kube-apiserver, kube-controller-manager, kube-scheduler服务进程，使用kubectl作为客户端与Master进程交互操作，在工作Node上仅仅需要部署kubelet和kube-proxy服务进程，K8s还提供了一个all-in-one的hyperkube程序来完成对应上服务程序的启动

   K8s的服务都可以通过知己运行二进制文件加上启动参数完成，为了便于管理，常见的做法是将K8s服务程序配置为Linux的系统开机自动启动的任务。
   
   需要关闭防火墙
   # systemctl disable firewalld
   # systemctl stop firewalld

   1) etcd服务
      etcd服务作为K8s集群的主数据库，在安装K8s个服务之前需要首先安装和启动
</pre>

<pre>
K8s集群的安全设置。
   在一个安全的内网环境中，K8s的各个组件与Master之间可以通过apiserver的非安全端口8080进行访问，但如果apiserver需要对外提供服务，或者及群众的某些容器也需要访问apiserver以获取及群众的某些信息，则更安全的做法是启用HTTPS安全机制，K8s提供了基于CA签名的双向数字证书认证认识和简单的基于HTTP BASE或TOKEN的认证方式，其中CA证书方式的安全性最高。
</pre>

<pre>
K8s的版本升级
   K8s的版本升级需要考虑当前及群众正在运行的容器不受影响，应对集群中的各Node逐个记性隔离，然后等待在其上运行的容器全部执行完毕，再更新该Node上的kubelet,kube-proxy服务，将全部Node都更新完成后，最后更新Master服务。
</pre>

<pre>
内网中的K8s的相关配置
    K8s在能够访问Internet网络的环境中使用起来非常方便，一方面在docker.io和gcr.io网站已经存在了大量官方制作的Docker镜像，另一方面GCE，AWS提供的云平台已经很成熟了，用户通过租用一定的空间来部署K8s集群也很方便。
    
    但是，许多企业内部由于安全性的原因无法访问Internet，对于这些企业就需要通过创建一个内部的私有Docker Registry，并修改K8s的配置，来启动内网中的K8s集群。

    第一步：
    Docker private Registry(私有Docker镜像库)
        使用Docker提供的Registry镜像创建一个私有镜像库

    第二步：
    kubelet配置
        由于K8s中是以Pod而不是Docker容器来管理单元的，在Kubelet创建Pod时，还通过启动一个名为google_containers/pause的镜像来实现Pod的概念。
</pre>

<pre>
K8s核心服务配置详解
</pre> 

<pre>
K8s集群网络配置方案
   在多个Node组成的K8s集群内，跨主机的容器间网络互通是K8s集群能够正常工作的前提条件,K8s本身并不会对跨主机容器网络进行设置，这需要额外的工具来实现，除了Google公有云平台GCE平台提供的网络设置，一些开源的工具包括flannel, Open vSwitch, Weave, Calico等都实现跨主机的荣期间网络互通。
</pre>

![](https://i.imgur.com/F7AQgjC.png)

<pre>
Kubectl命令行工具用法详解
   kubectl作为客户端CLI工具，可以让用户通过命令行的方式对K8s集群进行操作。

   kubectl命令行的语法
   $ kubectl [command] [TYPE] [NAME] [flags]
     如 kubectl get pods pod1
</pre>


