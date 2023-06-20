---
tip: translate by openai@2023-06-20 08:19:39
title: Load Balancing Techniques
---

# Virtual Server via NAT

NAT Routing is used when the Load-Balancer (or LVS Router) has two Network Interface Cards (NICs), one assigned an outside-facing IP address and the other, a private, inside-facing IP address. In this method, the Load-Balancer receives requests from users on the public network and uses network address translation (NAT) to forward those requests to the real servers located on the private network. The replies are also translated in the reverse direction, when the real servers reply to the users' requests.

> NAT 路由在负载均衡器（或 LVS 路由器）有两个网络接口卡（NICs）时使用，其中一个分配给外部面向的 IP 地址，另一个分配给私有内部面向的 IP 地址。在这种方法中，负载均衡器从公共网络中接收用户的请求，并使用网络地址转换（NAT）将这些请求转发到位于私有网络上的真实服务器。当真实服务器回复用户的请求时，也会反向进行转换。

As a result, an advantage is that the real servers are protected from the public network as they are hidden behind the Load-Balancer. Another advantage is IP address preservation, as the private network can use private address ranges.

> 结果，一个优势是真实服务器可以被负载均衡器隐藏在公共网络之外，从而受到保护。另一个优势是 IP 地址保护，因为私有网络可以使用私有地址范围。

The main disadvantage is that the Load-Balancer becomes a bottleneck. It has to serve not only requests but also replies to and from the public users, while also forwarding to and from the private real servers.

> 主要缺点是负载均衡器成为瓶颈。它不仅要处理公共用户的请求，还要处理来自公共用户的和发往私有真实服务器的回复。

# Virtual Server via Tunneling

In Tunneling mode, the Load-Balancer sends requests to real servers through IP tunnel in the former, and the Load-Balancer sends request to real servers via network address translation in the latter.

> 在隧道模式下，负载均衡器通过 IP 隧道向真实服务器发送请求，而负载均衡器通过网络地址转换向真实服务器发送请求。

The main advantage of this method is scalability, Load-Balancer will forward incoming request to farm nodes, latter nodes will then respond directly to the client requests without having to proxy through Load-Balancer. It offers you a way to locate nodes in different networking segments.

> 主要优势在于可扩展性，负载均衡器会将传入的请求转发到农场节点，后续节点将直接响应客户端请求，而不需要通过负载均衡器代理。它可以让您在不同的网络段找到节点。

The main disadvantage is the cost you will put into it to finally get a working env since it is deeply dependent upon your network architecture.

> 主要缺点是您将为最终获得一个可用的环境而付出的成本，因为它深受您的网络架构的影响。

# Virtual Server via Direct Routing

In Direct Routing, users issue requests to the VIP on the Load-Balancer. The Load-Balancer uses its predefined scheduling (distribution) algorithm and forwards the requests to the appropriate real server. Unlike using NAT Routing, the real servers respond directly to the public users, bypassing the need to route through the Load-Balancer.

> 在直接路由中，用户向负载均衡器上的 VIP 发出请求。负载均衡器使用其预定义的调度（分配）算法，将请求转发到适当的真实服务器。与使用 NAT 路由不同，真实服务器直接响应公共用户，无需经过负载均衡器路由。

The main advantage with this routing method is scalability, as the Load-Balancer does not have the additional responsibility of routing outgoing packets from the real servers to the public users.

> 主要优势是路由方法的可扩展性，因为负载均衡器不需要承担将真实服务器发送到公众用户的出站数据包的额外责任。

The disadvantage to this routing method lies in its ARP limitation. In order for the real servers to directly respond to the public users' requests, each real server must use the VIP as its source address when sending replies. As a result, the VIP and MAC address combination are shared amongst the Load-Balancer itself as well as each of the real servers that can lead to situations where the real servers receive the requests directly, bypassing the Load-Balancer on incoming requests. There are methods available to solve this problem at the expense of added configuration complexity and manageability.

> 这种路由方法的缺点在于它的 ARP 限制。为了让真实服务器直接响应公众用户的请求，每个真实服务器在发送响应时必须使用 VIP 作为其源地址。因此，VIP 和 MAC 地址组合被负载均衡器本身以及每个真实服务器共享，可能导致真实服务器直接接收请求而绕过负载均衡器的情况。有一些方法可以解决这个问题，但需要增加配置复杂性和可管理性。
