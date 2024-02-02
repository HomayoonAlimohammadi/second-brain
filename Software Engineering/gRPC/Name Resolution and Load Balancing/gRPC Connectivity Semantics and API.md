#grpc #software_engineering 

### related:
[[Balancing gRPC Traffic in K8S Without a Service Mesh]]
[[gRPC Load Balancing]]
[[gRPC Load Balancing on Kubernetes without Tears]]
[[Load balancing and scaling long-lived connections in Kubernetes]]
[[Load Balancing in gRPC]]
[[Name Resolution and Load Balancing in gRPC]]
[[Service Config in gRPC]]

### reference:
https://github.com/grpc/grpc/blob/master/doc/connectivity-semantics-and-api.md

P.S. Created by GPT4, nice try!
![[grpclb-gpt4-connection-state-machine.png]]

But Seriously:


|From/To|CONNECTING|READY|TRANSIENT_FAILURE|IDLE|SHUTDOWN|
|---|---|---|---|---|---|
|CONNECTING|Incremental progress during connection establishment|All steps needed to establish a connection succeeded|Any failure in any of the steps needed to establish connection|No RPC activity on channel for IDLE_TIMEOUT|Shutdown triggered by application.|
|READY||Incremental successful communication on established channel.|Any failure encountered while expecting successful communication on established channel.|No RPC activity on channel for IDLE_TIMEOUT  <br>OR  <br>upon receiving a GOAWAY while there are no pending RPCs.|Shutdown triggered by application.|
|TRANSIENT_FAILURE|Wait time required to implement (exponential) backoff is over.||||Shutdown triggered by application.|
|IDLE|Any new RPC activity on the channel||||Shutdown triggered by application.|
|SHUTDOWN||||||

### gRPC Connectivity States
gRPC channels abstract client-server communication, handling name resolution, TCP connections, TLS handshakes, and error management. To simplify user interaction while providing insight into channel status, gRPC defines five connectivity states:

- **CONNECTING**: Channel is attempting to establish a connection, including name resolution, TCP, and TLS handshakes.
- **READY**: A successful connection has been established, and the channel can facilitate communication.
- **TRANSIENT_FAILURE**: Temporary failures have occurred, prompting the channel to retry connection establishment with exponential backoff.
- **IDLE**: The channel is not actively attempting to establish a connection due to lack of RPC activity. New RPCs can transition the channel out of this state.
- **SHUTDOWN**: The channel is shutting down, either by user request or due to a non-recoverable error, and no new RPCs will be initiated.

### State Transitions
The transitions between these states are governed by specific rules and triggers, such as connection successes, failures, inactivity, or explicit shutdown commands. Notably:
- Channels move from **IDLE** to **CONNECTING** upon new or pending RPCs.
- Failure to connect transitions a channel to **TRANSIENT_FAILURE**, then back to **CONNECTING** as retries are attempted.
- A specified **IDLE_TIMEOUT** with no RPC activity shifts channels from **READY** or **CONNECTING** to **IDLE**.
- **SHUTDOWN** state is entered upon explicit shutdown request or irrecoverable errors.

### Channel State API
gRPC provides APIs to query and monitor channel states:
- **GetState(bool try_to_connect)**: Returns the current state and optionally attempts to connect if the channel is in **IDLE**.
- **WaitForStateChange(grpc_connectivity_state source_state, gpr_timespec deadline)**: Synchronously waits for a state change from the specified source state, useful for applications to respond to connectivity changes.
- <mark>Note that a notification is delivered every time there is a transition from any state to any _other_ state. On the other hand the rules for legal state transition, require a transition from CONNECTING to TRANSIENT_FAILURE and back to CONNECTING for every recoverable failure, even if the corresponding exponential backoff requires no wait before retry. The combined effect is that the application may receive state change notifications that appear spurious. e.g., an application waiting for state changes on a channel that is CONNECTING may receive a state change notification but find the channel in the same CONNECTING state on polling for current state because the channel may have spent infinitesimally small amount of time in the TRANSIENT_FAILURE state.</mark>

