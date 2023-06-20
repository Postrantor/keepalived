---
tip: translate by openai@2023-06-20 08:21:38
title: IPVS Scheduling Algorithms
---

The following scheduling algorithms are supported by the IPVS kernel code. (heavily stolen from LVS website).

> 以下调度算法由 IPVS 内核代码支持（大量借鉴自 LVS 网站）。

# Round Robin (rr)

The round-robin scheduling algorithm sends each incoming request to the next server in its list. Thus in a three server cluster (servers A, B, and C) request 1 would go to server A, request 2 would go to server B, request 3 would go to server C, and request 4 would go to server A, thus completing the cycling or \'round-robin\' of servers. It treats all real servers as equals regardless of the number of incoming connections or response time each server is experiencing. Virtual Server provides a few advantages over traditional round-robin DNS. Round-robin DNS resolves a single domain to the different IP addresses, the scheduling granularity is host-based, and the caching of DNS queries hinders the basic algorithm, these factors lead to significant dynamic load imbalances among the real servers. The scheduling granularity of Virtual Server is network connection-based, and it is much superior to round-robin DNS due to the fine scheduling granularity.

> 环形调度算法将每个传入请求发送到其列表中的下一个服务器。因此，在三个服务器集群（服务器 A，B 和 C）中，请求 1 将发送到服务器 A，请求 2 将发送到服务器 B，请求 3 将发送到服务器 C，请求 4 将发送到服务器 A，从而完成服务器的循环或“环形”。它无论传入连接数量或每个服务器所经历的响应时间如何，都将所有真实服务器视为平等的。虚拟服务器比传统的环形轮询 DNS 提供了一些优势。环形轮询 DNS 将单个域解析为不同的 IP 地址，调度粒度基于主机，DNS 查询的缓存妨碍了基本算法，这些因素导致真实服务器之间出现显著的动态负载不平衡。虚拟服务器的调度粒度是基于网络连接的，由于细粒度的调度，它比环形轮询 DNS 要优越得多。

# Weighted Round Robin (wrr)

The weighted round-robin scheduling is designed to better handle servers with different processing capacities. Each server can be assigned a weight, an integer value that indicates the processing capacity. Servers with higher weights receive new connections first than those with less weights, and servers with higher weights get more connections than those with less weights and servers with equal weights get equal connections. For example, the real servers, A, B, and C, have the weights, 4, 3, 2 respectively, a good scheduling sequence will be AABABCABC in a scheduling period (mod sum(Wi)). In the implementation of the weighted round-robin scheduling, a scheduling sequence will be generated according to the server weights after the rules of Virtual Server are modified. The network connections are directed to the different real servers based on the scheduling sequence in a round-robin manner.

> 加权轮询调度旨在更好地处理具有不同处理能力的服务器。每个服务器可以分配一个权重，一个整数值，表示处理能力。权重较高的服务器首先接收新连接，而权重较低的服务器则接收较少的连接，而权重相等的服务器则获得相等的连接。例如，实际服务器 A，B 和 C 的权重分别为 4，3，2，在调度期间（mod sum（Wi））的一个良好的调度序列将是 AABABCABC。在加权轮询调度的实施中，根据虚拟服务器的规则修改后，将根据服务器权重生成调度序列。网络连接以轮询方式按照调度序列指向不同的实际服务器。

The weighted round-robin scheduling is better than the round-robin scheduling, when the processing capacity of real servers is different. However, it may lead to dynamic load imbalance among the real servers if the load of the requests vary highly. In short, there is the possibility that a majority of requests requiring large responses may be directed to the same real server.

> 加权轮询调度比普通轮询调度更适用于真实服务器的处理能力不同的情况。但是，如果请求的负载变化很大，可能会导致真实服务器之间的动态负载不平衡。简而言之，有可能大部分需要大响应的请求会被指向同一个真实服务器。

Actually, the round-robin scheduling is a special instance of the weighted round-robin scheduling, in which all the weights are equal.

> 实际上，轮叫调度是加权轮叫调度的一种特殊情况，其中所有权重均相等。

# Least Connection (lc)

The least-connection scheduling algorithm directs network connections to the server with the least number of established connections. This is one of the dynamic scheduling algorithms; because it needs to count live connections for each server dynamically. For a Virtual Server that is managing a collection of servers with similar performance, least-connection scheduling is good to smooth distribution when the load of requests varies a lot. Virtual Server will direct requests to the real server with the fewest active connections.

> 最少连接调度算法将网络连接指向具有最少已建立连接的服务器。这是动态调度算法之一，因为它需要为每个服务器动态计数活动连接。对于管理具有相似性能的服务器集合的虚拟服务器来说，最少连接调度是平滑分布请求负载变化很大的好方法。虚拟服务器将请求指向具有最少活动连接的实际服务器。

At a first glance, it might seem that least-connection scheduling can also perform well even when there are servers of various processing capacities, because the faster server will get more network connections. In fact, it cannot perform very well because of the TCP\'s TIME_WAIT state. The TCP\'s TIME_WAIT is usually 2 minutes, during this 2 minutes a busy website often receives thousands of connections, for example, the server A is twice as powerful as the server B, the server A is processing thousands of requests and keeping them in the TCP\'s TIME_WAIT state, but server B is crawling to get its thousands of connections finished. So, the least-connection scheduling cannot get load well balanced among servers with various processing capacities.

> 第一眼看去，当服务器处理能力不同时，最少连接调度似乎也能表现良好，因为更快的服务器会获得更多网络连接。实际上，由于 TCP 的 TIME_WAIT 状态，它不能表现得很好。TCP 的 TIME_WAIT 通常是 2 分钟，在这 2 分钟内，繁忙的网站通常会收到成千上万的连接，例如，服务器 A 的处理能力是服务器 B 的两倍，服务器 A 正在处理成千上万的请求并将它们保持在 TCP 的 TIME_WAIT 状态，但服务器 B 却在慢慢地完成它的成千上万的连接。因此，最少连接调度无法在具有不同处理能力的服务器之间很好地平衡负载。

# Weighted Least Connection (wlc)

The weighted least-connection scheduling is a superset of the least-connection scheduling, in which you can assign a performance weight to each real server. The servers with a higher weight value will receive a larger percentage of live connections at any one time. The Virtual Server Administrator can assign a weight to each real server, and network connections are scheduled to each server in which the percentage of the current number of live connections for each server is a ratio to its weight. The default weight is one.

> 加权最少连接调度是最少连接调度的超集，您可以为每个真实服务器分配性能权重。权重值较高的服务器将在任何一个时刻接收更大比例的实时连接。虚拟服务器管理员可以为每个真实服务器分配权重，网络连接按照每个服务器当前活动连接数量占其权重的比例进行调度。默认权重为 1。

The weighted least-connections scheduling works as follows:

> 加权最少连接调度的工作原理如下：

> Supposing there are n real servers, each server i has weight Wi (i=1,..,n), and alive connections Ci (i=1,..,n), ALL_CONNECTIONS is the sum of Ci (i=1,..,n), the next network connection will be directed to the server j, in which
>
> > (Cj/ALL_CONNECTIONS)/Wj = min { (Ci/ALL_CONNECTIONS)/Wi } (i=1,..,n)
>
> Since the ALL_CONNECTIONS is a constant in this lookup, there is no need to divide Ci by ALL_CONNECTIONS, it can be optimized as
>
> > Cj/Wj = min { Ci/Wi } (i=1,..,n)

The weighted least-connection scheduling algorithm requires additional division than the least-connection. In a hope to minimize the overhead of scheduling when servers have the same processing capacity, both the least-connection scheduling and the weighted least-connection scheduling algorithms are implemented.

> 加权最少连接调度算法比最少连接算法需要更多的分配。为了尽量减少服务器具有相同处理能力时调度的开销，实现了最少连接调度和加权最少连接调度算法。

# Locality-Based Least Connection (lblc)

The locality-based least-connection scheduling algorithm is for destination IP load balancing. It is usually used in cache cluster. This algorithm usually directs packet destined for an IP address to its server if the server is alive and under load. If the server is overloaded (its active connection numbers is larger than its weight) and there is a server in its half load, then allocate the weighted least-connection server to this IP address.

> 本地基于最少连接调度算法是为了目的 IP 负载均衡。它通常用于缓存集群。此算法通常会将针对 IP 地址的数据包发送到其服务器，如果服务器是活动的并且处于负载状态。如果服务器超载（其活动连接数大于其权重）并且有一个服务器处于半负载状态，则将加权最少连接服务器分配给此 IP 地址。

# Locality-Based Least Connection with Replication (lblcr)

The locality-based least-connection with replication scheduling algorithm is also for destination IP load balancing. It is usually used in cache cluster. It differs from the LBLC scheduling as follows: the load balancer maintains mappings from a target to a set of server nodes that can serve the target. Requests for a target are assigned to the least-connection node in the target\'s server set. If all the node in the server set are over loaded, it picks up a least-connection node in the cluster and adds it in the sever set for the target. If the server set has not been modified for the specified time, the most loaded node is removed from the server set, in order to avoid high degree of replication.

> 结合位置的最少连接复制调度算法也可用于目标 IP 负载均衡。它通常用于缓存集群。它与 LBLC 调度的不同之处在于：负载均衡器维护从目标到可以提供目标的服务器节点集的映射。对目标的请求分配给目标服务器集中最少连接的节点。如果服务器集中的所有节点都超负载，它将从集群中选择最少连接的节点并将其添加到目标的服务器集中。如果服务器集在指定的时间内没有被修改，则从服务器集中移除负载最大的节点，以避免高度重复。

# Destination Hashing (dh)

The destination hashing scheduling algorithm assigns network connections to the servers through looking up a statically assigned hash table by their destination IP addresses.

> 算法把网络连接分配给服务器，是通过查找根据目的地 IP 地址静态分配的散列表来实现的。

# Source Hashing (sh)

The source hashing scheduling algorithm assigns network connections to the servers through looking up a statically assigned hash table by their source IP addresses.

> 算法源哈希调度通过查找其来源 IP 地址的静态分配哈希表，将网络连接分配给服务器。

# Shortest Expected Delay (sed)

The shortest expected delay scheduling algorithm assigns network connections to the server with the shortest expected delay. The expected delay that the job will experience is (Ci + 1) / Ui if sent to the ith server, in which Ci is the number of connections on the ith server and Ui is the fixed service rate (weight) of the ith server.

> 最短预期延迟调度算法将网络连接分配给具有最短预期延迟的服务器。如果发送到第 i 个服务器，作业将经历的预期延迟为(Ci + 1) / Ui，其中 Ci 是第 i 个服务器上的连接数，Ui 是第 i 个服务器的固定服务速率（权重）。

# Never Queue (nq)

The never queue scheduling algorithm adopts a two-speed model. When there is an idle server available, the job will be sent to the idle server, instead of waiting for a fast one. When there is no idle server available, the job will be sent to the server that minimizes its expected delay (The Shortest Expected Delay scheduling algorithm).

> 绝对不排队调度算法采用双速模式。当有空闲服务器可用时，将作业发送到空闲服务器，而不是等待快速服务器。当没有空闲服务器可用时，将作业发送到最小化其预期延迟的服务器（最短预期延迟调度算法）。

# Overflow-Connection (ovf)

The Overflow connection scheduling algorithm implements \"overflow\" loadbalancing according to a number of active connections, will keep all connections to the node with the highest weight and overflow to the next node if the number of connections exceeds the node\'s weight. Note that this scheduler might not be suitable for UDP because it only uses active connections

> 突发连接调度算法根据活动连接的数量实现“突发”负载均衡，将所有连接保持在具有最高权重的节点上，如果连接数超过节点的权重，就会突发到下一个节点。请注意，此调度程序可能不适用于 UDP，因为它只使用活动连接。
