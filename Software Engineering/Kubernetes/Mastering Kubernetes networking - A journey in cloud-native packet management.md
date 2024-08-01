#kubernetes #networking 

## References:
https://otterize.com/blog/mastering-kubernetes-networking-otterize-s-journey-in-cloud-native-packet-management

![[pod-overlay.webp]]
![[svc.webp]]
![[ingress.webp]]
![[nodeport.webp]]
![[pod-overlay.webp]]
![[loadbalancer.webp]]
![[user-to-ingress.webp]]
![[ingress-to-application.webp]]
![[voting-app-architecture-w-ingress.webp]]
#### Kubernetes Networking Overview
- **Deep Networking Technologies**: Initial attraction to Kubernetes networking due to its differences from traditional datacenter technologies.
- **Abstraction and Scalability**: Kubernetes abstracts network management, enhancing scalability and simplicity using the Container Network Interface (CNI).

#### CNI Plugins
- **Flexibility**: Users can choose different CNI plugins like Flannel, Cilium, and MetalLB based on their networking needs.
- **Standard Functions**: CNIs implement functions like AddNetwork, DeleteNetwork, and CheckNetwork in Go.

#### Core Components of Kubernetes Networking
- **Pod Networking**: Each pod gets a unique IP address, simplifying direct communication across the cluster without complex routing or NAT.
  - **Routed vs. Overlay Networking**: Options for traffic routing and encapsulation.

- **Service Networking**: Services provide stable entry points with ClusterIP, NodePort, and LoadBalancer types.
  - **ClusterIP**: Internal cluster communication.
  - **NodePort**: Exposes services on specific ports across all nodes.
  - **LoadBalancer**: Uses cloud provider capabilities for external access.
  - **Kube-proxy**: Manages traffic routing using iptables, IPVS, or nftables.

- **Ingress Controllers**: Manage external access with routing rules for HTTP/HTTPS traffic.
  - Examples include Traefik, Nginx, and HAProxy.

- **DNS in Kubernetes**: Simplifies service discovery with DNS names for services, maintaining robust internal communication.

#### Network Policies
- **Access Control**: Use Network Policies for fine-grained control over pod communication.
- **Zero-Trust Security Model**: Default deny-all policies with explicit allow rules to minimize the attack surface.

#### Example Implementation
- **Network Policies**: Define ingress and egress rules using YAML manifests.
- **Otterize**: Automates Network Policy creation based on real traffic patterns using Client Intents.

#### Packet Walk
- **From User to Ingress Controller**: Explains the flow of a request through Traefik, GCLB, kube-proxy, and to the backend pod.
- **Traffic Monitoring**: Using tcpdump and ephemeral containers to monitor traffic.
- **Returning Traffic**: Describes the reverse flow from the backend pod to the user.

### Tools and Resources
- **Network Mapping and Network Policy Tutorials**: For hands-on exploration.
- **Community Slack Channel**: For updates and use cases.

### Revisiting Historical Networking Innovations
- **Software-Defined Networking (SDN)**: Contributions from Nicira, OpenFlow, and OpenStack's Neutron project influenced Kubernetes' CNI design.
- **Virtual Network Function (VNF) and 5G**: Kubernetes' adaptability to complex environments.

### Conclusion
- **Manual Management of Network Policies**: Can be overwhelming, but tools like Otterize automate and streamline the process.
- **Continuous Adaptation**: Otterize ensures security rules remain up-to-date, aligning with application dynamics to reduce the blast radius of potential attacks.

