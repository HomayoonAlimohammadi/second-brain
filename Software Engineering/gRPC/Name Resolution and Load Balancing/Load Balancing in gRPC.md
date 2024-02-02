#software_engineering #grpc #load_balancing #http2 #kubernetes 
### related
[[Name Resolution and Load Balancing in gRPC]]
[[Balancing gRPC Traffic in K8S Without a Service Mesh]]
[[gRPC Load Balancing]]
[[gRPC Load Balancing on Kubernetes without Tears]]
[[Load balancing and scaling long-lived connections in Kubernetes]]

### references:
https://github.com/grpc/grpc/blob/master/doc/load-balancing.md
https://github.com/grpc/grpc/blob/master/doc/connection-backoff.md


- **Scope and Background**:
  - Explains gRPC's per-call basis load balancing, emphasizing distribution of requests across all servers regardless of the client's number.

- **Architecture Overview**:
  - gRPC client supports load balancing policies (LB policies) that can be implemented and plugged in.
  - LB policies are responsible for receiving server addresses and configurations, managing subchannels, setting channel connectivity state, and determining subchannel for each RPC.

- **Key Load Balancing Policies**:
  - **pick_first** (Default):
    - Connects to addresses one at a time until a reachable one is found.
    - All RPCs are sent to the first reachable address.
    - Channel state set to `TRANSIENT_FAILURE` if no addresses are reachable, attempts reconnection with backoff.
		- **Key Parameters**:
		  - **`INITIAL_BACKOFF`**: Time to wait after the first failure before retrying (e.g., 1 second).
		  - **`MULTIPLIER`**: Factor to multiply the backoff after a failed retry (e.g., 1.6).
		  - **`JITTER`**: Amount to randomize backoffs (e.g., 0.2, or 20% of the current backoff).
		  - **`MAX_BACKOFF`**: Upper limit on the backoff time (e.g., 120 seconds).
		  - **`MIN_CONNECT_TIMEOUT`**: Minimum time to allow a connection attempt to complete (e.g., 20 seconds).
		
		- **Proposed Backoff Algorithm**:
		  - Begin with an initial backoff.
		  - On each failed connection attempt, wait for the current backoff period before retrying.
		  - Increase the backoff for the next attempt by multiplying the current backoff by the multiplier, not exceeding the max backoff, and apply jitter.
		  - The process continues until a successful connection is established.
		
		- **Algorithm Steps**:
		  1. Set `current_backoff` to `INITIAL_BACKOFF`.
		  2. Attempt to connect, respecting `MIN_CONNECT_TIMEOUT` if the current deadline is in the past.
		  3. If connection fails, sleep until `current_deadline`.
		  4. Update `current_backoff` using the multiplier, capped at `MAX_BACKOFF`.
		  5. Calculate `current_deadline` by adding the updated backoff to the current time and apply jitter.
		  6. Repeat the process until a successful connection is made.
		
		- **Reset Backoff**:
		  - Backoff is reset to `INITIAL_BACKOFF` when a SETTINGS frame is received, indicating a successful connection acceptance by the server.
		
		```go
		ConnectWithBackoff()
		  current_backoff = INITIAL_BACKOFF
		  current_deadline = now() + INITIAL_BACKOFF
		  while (TryConnect(Max(current_deadline, now() + MIN_CONNECT_TIMEOUT))
		         != SUCCESS)
		    SleepUntil(current_deadline)
		    current_backoff = Min(current_backoff * MULTIPLIER, MAX_BACKOFF)
		    current_deadline = now() + current_backoff +
		      UniformRandom(-JITTER * current_backoff, JITTER * current_backoff)
		```
		    
  - **round_robin**:
    - Creates a subchannel for each address, constantly monitoring connectivity.
    - Channel's state is aggregated from subchannels' states:
      - `READY` if any subchannel is ready.
      - `CONNECTING` if any is connecting.
      - `IDLE` if any is idle.
      - `TRANSIENT_FAILURE` if all are in transient failure.
    - RPCs are distributed across all ready subchannels in a cyclic manner.

- **Workflow**:
  - Upon startup, a name resolution request is issued by the gRPC client.
  - The client then instantiates the LB policy with the resolved addresses, service config, and attributes.
  - The LB policy creates and manages subchannels based on the server IPs.
  - For each RPC, the LB policy selects an appropriate subchannel.

- **grpclb Policy**:
  - Special considerations for `grpclb` policy not detailed in the summary but involves communication with load balancers to obtain server lists.

