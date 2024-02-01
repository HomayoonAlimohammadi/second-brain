#grpc #kubernetes #linkerd #load_balancing #software_engineering #http2

### related
[[Load balancing and scaling long-lived connections in Kubernetes]]
### references:
https://kubernetes.io/blog/2018/11/07/grpc-load-balancing-on-kubernetes-without-tears/ (William Morgan - 2018)

- **Issue Highlighted**: Kubernetes' default load balancing does not effectively work with gRPC applications due to gRPC's reliance on HTTP/2's long-lived TCP connections which multiplex requests, leading to traffic being pinned to a single pod.

- **Contrast with HTTP/1.1**: 
  - HTTP/1.1 connections cycle due to lack of request multiplexing and connection expiration, allowing "good enough" connection-level balancing.
  - gRPC's multiplexing over HTTP/2 prevents this natural cycling, necessitating a different approach to load balancing.

- **Solutions for gRPC Load Balancing**:
  - **Application-Managed**: Complex due to dynamic Kubernetes environments.
  - **Headless Services**: Limited by client capabilities and not universally applicable.
  - **Linkerd Service Mesh**: Recommended for ease of use, automatic balancing, and broad compatibility.
    ![[grpclb-linkerd.png]]

- **Advantages of Using Linkerd**:
  - Works with any language, gRPC client, and deployment model.
  - Performs sophisticated L7 load balancing by watching Kubernetes API and adjusting traffic based on pod availability and response latencies.
  - Introduces minimal latency (<1ms at p99) and memory footprint (<10mb RSS per pod).

- **Implementation and Impact**:
  - Easy to integrate with a few CLI commands, no changes to application code required.
  - Demonstrably balances traffic across all pods, improving resource utilization and service responsiveness.
  - Provides detailed traffic dashboards for monitoring and troubleshooting.

- **Conclusion**:
  - Linkerd offers a simple yet powerful solution for adding gRPC load balancing in Kubernetes, enhancing the scalability and efficiency of microservices architectures.

