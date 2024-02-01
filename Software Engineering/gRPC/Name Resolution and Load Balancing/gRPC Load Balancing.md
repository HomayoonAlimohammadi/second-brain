#grpc #load_balancing #kubernetes #software_engineering #http2  

### related:
[[gRPC Load Balancing on Kubernetes without Tears]]
[[Load balancing and scaling long-lived connections in Kubernetes]]

### references:
https://grpc.io/blog/grpc-load-balancing/ (makdharma - Google - 2017)

- **Introduction to gRPC Load Balancing**:
  - Large-scale gRPC deployments typically involve numerous identical backend instances and clients, necessitating effective load balancing to optimally distribute client load.

- **Advantages of gRPC**:
  - Built on HTTP/2, providing benefits like binary protocol efficiency, multiplexing, header compression, and strongly typed definitions via Protobuf.
  - Seamless integration with ecosystem components like service discovery, load balancing, and monitoring.

- **Load Balancing Options**: Proxy vs. Client Side
  - **Proxy Load Balancing (Server-Side)**:
    - Clients send requests to a Load Balancer (LB) proxy, which then distributes the call to backend servers.
    - Suitable for user-facing services with untrusted clients from the open internet.
    - Backends report load to the LB for fair distribution.
      ![[grpclb-1.png]]
  - **Client-Side Load Balancing**:
    - Clients are aware of multiple backends and choose one for each RPC, using server load reports to inform decisions.
    - Simpler configurations may not consider server load, relying on round-robin selection.
      ![[grpclb-2.png]]
      
      #### Proxy or Client side?
| Proxy | Client Side |  |
| ---- | ---- | ---- |
| Pros | - Simple client<br>- No client-side awareness of backend<br>- Works with untrusted clients | - High performance because elimination of extra hop |
| Cons | - LB is in the data path<br>- Higher latency<br>- LB throughput may limit scalability | - Complex client<br>- Client keeps track of server load and health<br>- Client implements load balancing algorithm<br>- Per-language implementation and maintenance burden<br>- Client needs to be trusted, or the trust boundary needs to be handled by a lookaside LB. |


- **Proxy Load Balancer Types**:
  - **L3/L4 (Transport Level)**:
    - Minimal processing and latency, cost-effective, copies application data between client and backend connections.
  - **L7 (Application Level)**:
    - Terminates and parses HTTP/2, allowing request inspection and backend assignment based on content, supports HTTP/2 stream distribution among multiple backends.
      
      #### L3/L4 (Transport) vs L7 (Application)
| Use case | Recommendation |
| ---- | ---- |
| RPC load varies a lot among connections | Use Application level LB |
| Storage or compute affinity is important | Use Application level LB and use cookies or similar for routing requests to correct backend |
| Minimizing resource utilization in proxy is more important than features | Use L3/L4 LB |
| Latency is paramount | Use L3/L4 LB |


- **Client Side LB Options**:
  - **Thick Client**:
    - Integrates load balancing logic, maintaining server availability, workload, and selection algorithms, often in conjunction with service discovery and other infrastructure libraries.
  - **Lookaside Load Balancing**:
    - Offloads balancing logic to a special LB server, with clients querying for the best servers to use, allowing sophisticated algorithm implementation on the LB side while enabling simple client-side logic.
      ![[grpclb-3.png]]


#### Recommendations and best practices
| Setup | Recommendation |
| ---- | ---- |
| - Very high traffic between clients and servers<br>- Clients can be trusted | - Thick client-side load balancing<br>- Client side LB with ZooKeeper/Etcd/Consul/Eureka. [ZooKeeper example](https://github.com/makdharma/grpc-zookeeper-lb). |
| - Traditional setup - Many clients connecting to services behind a proxy<br>- Need trust boundary between servers and clients | - Proxy Load Balancing<br>- L3/L4 LB with GCLB (if using GCP)<br>- L3/L4 LB with haproxy - [config file](https://gist.github.com/thpham/114d20de8472b2cef966)<br>- Nginx coming soon<br>- If need session stickiness - L7 LB with Envoy as proxy |
| - Microservices - N clients, M servers in the data center<br>- Very high performance requirements (low latency, high traffic)<br>- Client can be untrusted | - Look-aside Load Balancing<br>- Client-side LB using [gRPC-LB protocol](https://github.com/grpc/grpc/blob/master/doc/load-balancing.md). Roll your own implementation (Q2’17), hosted gRPC-LB in the works. |
| - Existing Service-mesh like setup using Linkerd or Istio | - Service Mesh<br>- Use built-in LB with [Istio](https://istio.io/), or [Envoy](https://github.com/lyft/envoy). |




