#grpc #load_balancing #kubernetes #software_engineering #http2 

### related:
[[Balancing gRPC Traffic in K8S Without a Service Mesh]]
[[gRPC Load Balancing on Kubernetes without Tears]]
[[gRPC Load Balancing]]
[[Load balancing and scaling long-lived connections in Kubernetes]]

### references:
https://fuchsia.googlesource.com/third_party/grpc/+/HEAD/doc/load-balancing.md

## Name Resolution

- **Overview of gRPC Name Resolution**:
  - gRPC defaults to DNS for name resolution but supports various other name systems through a general API.
  - The gRPC client library across languages offers a plugin mechanism allowing for different resolvers to be integrated.

- **Detailed Design**:
  - **Name Syntax**: Uses `scheme://authority/endpoint_name` format, where:
    - `scheme` identifies the name-system (e.g., `dns`, `ipv4`, `ipv6`, `unix`).
    - `authority` provides scheme-specific bootstrap information (e.g., DNS server IP and port).
    - `endpoint_name` denotes the concrete name to resolve within the chosen name-system.
  - Future support may include additional schemes like `etcd`.

- **Resolver Plugins**:
  - The gRPC client library selects a resolver plugin based on the specified scheme, passing it the fully qualified name string.
  - Resolver plugins are tasked with contacting the authority to obtain a resolution, which includes:
    - A list of resolved addresses, each with an IP address, port, a boolean indicating if it's a backend or balancer address, and the balancer name for peer authorization.
    - A service config.
  - Plugins can continuously monitor an endpoint, updating resolutions as necessary.


## Load Balancing
![[grpclb-flow.png]]
- **gRPC Client Load Balancing Workflow**:
  - **Name Resolution**: On startup, the gRPC client requests the server name resolution, which resolves to one or more IP addresses. Each address indicates whether it's a server or a load balancer address. A service config specifies the client-side load-balancing policy to use (e.g., `round_robin` or `grpclb`).
  
  - **Policy Instantiation**: The client instantiates the load balancing policy indicated by the service config. If any address is a balancer address, `grpclb` policy is used regardless of the service config. If no policy is specified, the client defaults to a policy that selects the first available server address.

  - **Subchannel Creation**:
    - For all policies except `grpclb`, a subchannel is created for each server address returned by the resolver, ignoring balancer addresses.
    - For `grpclb` policy, a stream is opened to a balancer address. The balancer provides server addresses for the original server name requested by the client.

  - **Server List and Load Reporting**:
    - In `grpclb`, gRPC servers may report load to the balancers if required by the balancer's configuration.
    - The load balancer returns a server list to the client's `grpclb` policy, which then creates a subchannel for each server.

  - **RPC Dispatching**:
    - The load balancing policy determines which subchannel (server) an RPC should be sent to.
    - With `grpclb`, requests are sent to servers in the order they were returned by the load balancer. If the server list is empty, calls block until a non-empty list is received.


## Service Config

The service config is a mechanism that allows service owners to publish parameters to be automatically used by all clients of their service.

A service config is associated with a server name. The name resolver plugin, when asked to resolve a particular server name, will return both the resolved addresses and the service config.

The service config is used in the following APIs:
- In the resolver API, used by resolver plugins to return the service config to the gRPC client.
- In the gRPC client API, where users can query the channel to obtain the service config associated with the channel (for debugging purposes).
- In the gRPC client API, where users can set the service config explicitly. This is intended for use in unit tests.

The service config is a JSON string of the following form:

```json
{
  // Load balancing policy name.
  // Currently, the only selectable client-side policy provided with gRPC
  // is 'round_robin', but third parties may add their own policies.
  // This field is optional; if unset, the default behavior is to pick
  // the first available backend.
  // If the policy name is set via the client API, that value overrides
  // the value specified here.
  //
  // Note that if the resolver returns at least one balancer address (as
  // opposed to backend addresses), gRPC will use grpclb (see
  // https://github.com/grpc/grpc/blob/master/doc/load-balancing.md),
  // regardless of what LB policy is requested either here or via the
  // client API.
  'loadBalancingPolicy': string,

  // Per-method configuration.  Optional.
  'methodConfig': [
    {
      // The names of the methods to which this method config applies. There
      // must be at least one name. Each name entry must be unique across the
      // entire service config. If the 'method' field is empty, then this
      // method config specifies the defaults for all methods for the specified
      // service.
      //
      // For example, let's say that the service config contains the following
      // method config entries:
      //
      // 'methodConfig': [
      //   { 'name': [ { 'service': 'MyService' } ] ... },
      //   { 'name': [ { 'service': 'MyService', 'method': 'Foo' } ] ... }
      // ]
      //
      // For a request for MyService/Foo, we will use the second entry, because
      // it exactly matches the service and method name.
      // For a request for MyService/Bar, we will use the first entry, because
      // it provides the default for all methods of MyService.
      'name': [
        {
          // RPC service name.  Required.
          // If using gRPC with protobuf as the IDL, then this will be of
          // the form "pkg.service_name", where "pkg" is the package name
          // defined in the proto file.
          'service': string,

          // RPC method name.  Optional (see above).
          'method': string,
        }
      ],

      // Whether RPCs sent to this method should wait until the connection is
      // ready by default. If false, the RPC will abort immediately if there
      // is a transient failure connecting to the server. Otherwise, gRPC will
      // attempt to connect until the deadline is exceeded.
      //
      // The value specified via the gRPC client API will override the value
      // set here. However, note that setting the value in the client API will
      // also affect transient errors encountered during name resolution,
      // which cannot be caught by the value here, since the service config
      // is obtained by the gRPC client via name resolution.
      'waitForReady': bool,

      // The default timeout in seconds for RPCs sent to this method. This can
      // be overridden in code. If no reply is received in the specified amount
      // of time, the request is aborted and a deadline-exceeded error status
      // is returned to the caller.
      //
      // The actual deadline used will be the minimum of the value specified
      // here and the value set by the application via the gRPC client API.
      // If either one is not set, then the other will be used.
      // If neither is set, then the request has no deadline.
      //
      // The format of the value is that of the 'Duration' type defined here:
      // https://developers.google.com/protocol-buffers/docs/proto3#json
      'timeout': string,

      // The maximum allowed payload size for an individual request or object
      // in a stream (client->server) in bytes. The size which is measured is
      // the serialized, uncompressed payload in bytes. This applies both
      // to streaming and non-streaming requests.
      //
      // The actual value used is the minimum of the value specified here and
      // the value set by the application via the gRPC client API.
      // If either one is not set, then the other will be used.
      // If neither is set, then the built-in default is used.
      //
      // If a client attempts to send an object larger than this value, it
      // will not be sent and the client will see an error.
      // Note that 0 is a valid value, meaning that the request message must
      // be empty.
      'maxRequestMessageBytes': number,

      // The maximum allowed payload size for an individual response or object
      // in a stream (server->client) in bytes. The size which is measured is
      // the serialized, uncompressed payload in bytes. This applies both
      // to streaming and non-streaming requests.
      //
      // The actual value used is the minimum of the value specified here and
      // the value set by the application via the gRPC client API.
      // If either one is not set, then the other will be used.
      // If neither is set, then the built-in default is used.
      //
      // If a server attempts to send an object larger than this value, it
      // will not be sent, and the client will see an error.
      // Note that 0 is a valid value, meaning that the response message must
      // be empty.
      'maxResponseMessageBytes': number
    }
  ]
}
```

