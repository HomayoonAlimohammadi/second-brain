#grpc #load_balancing #kubernetes #software_engineering #golang #http2 

### related:
[[gRPC Load Balancing on Kubernetes without Tears]]
[[gRPC Load Balancing]]
[[Load balancing and scaling long-lived connections in Kubernetes]]

### references:
https://medium.com/swlh/balancing-grpc-traffic-in-k8s-without-a-service-mesh-7005be902ef3
https://github.com/grpc/grpc-go/tree/master/examples/features/keepalive

- **Challenge with gRPC Load Balancing in Kubernetes**:
  - gRPC uses long-lived TCP connections over HTTP/2, complicating load balancing due to multiplexing multiple requests over a single connection.
  - Common mistakes affecting gRPC load balancing include incorrect gRPC client configuration, and improper Kubernetes service setup.

- **Correcting gRPC Client Configuration**:
  - The default gRPC client setup creates a single connection, suitable for 1-1 connections but not for production environments.
  - A modified setup involves using a DNS resolver scheme and specifying a balancer (e.g., round-robin) to enable connection to multiple servers.

- **Wrong Kubernetes Service Configuration**:
  - The default Kubernetes service setup links to a single IP, limiting the connection to one pod.
  - A headless service configuration (with `clusterIP: None`) exposes all pod IPs, enabling direct client connections to multiple pods.
    ![[grpclb-clusterIP.webp]]

- **Headless Service for Direct Pod Connections**:
  - Headless services return multiple pod IPs, allowing the gRPC client to see all available servers.
  - This shifts the balancing responsibility to the client, requiring Kubernetes only to keep the list of pods updated.
    ![[grpclb-headless.webp]]

- **Service Mesh as an Alternative**:
  - A service mesh simplifies setup and is language-agnostic, leveraging sidecars and a control plane for traffic orchestration.
  - Without a service mesh, clients must connect directly to multiple servers or use an L7 proxy for load balancing.

- **Custom Resolver and gRPC Proxy**:
  - The author created a custom resolver for Alpine Linux images to handle domain changes and pod scaling dynamically.
  - A custom gRPC proxy was also developed, leveraging HTTP/2 to proxy requests without modifying the proto payload.

- **Conclusion and Recommendations**:
  - For gRPC clients needing to connect with many servers, using a proxy is recommended to reduce complexity and resource consumption.
  - A proxy setup (1-M-N) is more efficient than direct connections (1-N), as each proxy can handle multiple connections to servers.

- Client
```go
func main() {
	conn, err := grpc.Dial(
		"dns:///service-name:port",
		grpc.WithBalancerName(roundrobin.Name), // or?
		grpc.WithDefaultServiceConfig(`{"loadBalancingPolicy":"round_robin"}`),
		...,
	)
	...
}
```

- Server
```go
func main() {
	opts := []grpc.ServerOption{ 
		grpc.KeepaliveParams(
			keepalive.ServerParameters{
				MaxConnectionAge: time.Minute * 5,
			},
		),
	}
	...
}
```