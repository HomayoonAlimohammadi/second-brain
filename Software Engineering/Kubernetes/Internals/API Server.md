
## Generic API server
basically something that builds the API servers that create the delegation chain?

Yes—that’s essentially what the “GenericAPIServer” package is. It’s a reusable framework in the kube-apiserver codebase that lets you stitch together:

- **Storage backends** for your resources
    
- **Authentication/authorization** layers
    
- **Admission chains** (webhooks, defaulting, validation)
    
- **OpenAPI discovery**
    
- **Delegation handlers** to another GenericAPIServer
    

Both the core kube-apiserver and the Aggregator (and even custom extension servers you build) are just instances wired up via GenericAPIServer. Each instance can be given a “delegate” handler—so requests flow through your custom logic, then delegate to the next GenericAPIServer in the chain (e.g. the core server or another extension).

## Delegation

Order of handlers and their delegation targret:
Aggregator -> kube-apiserver (core) -> API extensions server -> notFound

each handler for a group/version either serves the request or calls its `delegate` to try the next one.

Kubernetes’ API server is built on a pluggable “delegation” pattern: each request is passed through a chain of “handlers,” and if a handler doesn’t claim responsibility for a given group/version/path, it defers (delegates) to the next one.

- **GenericAPIServer Core Registration**  
    When kube-apiserver boots, it registers all built-in API groups (core/v1, apps/v1, batch/v1, etc.) with its in-process handler chain.
    
- **Aggregator Layer**  
    On top of that, the Aggregator (an instance of the same GenericAPIServer code) mounts under `/apis`. It first checks its internal “proxy map” (built from APIService CRDs that point to backing Services).
    
    1. **Proxy lookup**: if the request’s group+version matches an Available APIService with a `.spec.service`, the Aggregator proxies the HTTP call to that external Service.
        
    2. **Delegate**: if there’s no proxy, it delegates transparently back to the core handler—so core groups continue to be served without interruption.
        

> **Example Flow**:
> 
> ```text
> GET /apis/apps/v1/deployments
> ├─ Aggregator: look for APIService “apps/v1” → none pointing at a Service → delegate
> └─ Core kube-apiserver: handle via its in-process apps/v1 Deployment REST storage
> ```

something to note is that this delegation chain lives in a single process (the `kube-apiserver` process). When querying an endpoint or a resource in the kubernetes API, if it is managed using a custom API server (defined in the APIService `spec.service` field), then it's proxied to a completely different process, otherwise everything is handled in the `kube-apiserver`.

## API services

- you can basically edit an APIService responsible for pods or deployments, add a backing service to them and overshadow the core kube-apiserver handler.

We can basically write our own API server that uses a ==custom storage backend== and serve our desired group/version using an API service (which is responsible for proxying requets about the specified group/version to the appropriate k8s service)

**It looks like a CRD but isn’t one**.

- A CRD uses the built-in `apiextensions.k8s.io` handler and default etcd storage; you never write Go code or wire up storage.
    
- With a custom API-server you define the Go types _and_ implement your own REST storage (could be etcd, SQL, in-memory, Kafka, etc.), lifecycle, admission, metrics, etc.
    
- In short: CRDs give you new types “for free” on the core server; custom API-servers give you full control over the implementation stack.

### 2. APIService Objects

An **APIService** is a Kubernetes CRD (in the `apiregistration.k8s.io` group) that tells the Aggregator how to include an extension API group/version:

- **Key fields**
    
    - `metadata.name`: `<version>.<group>` (e.g. `v1.mygroup.io`)
        
    - `spec.group` & `spec.version`: define the GV served
        
    - `spec.service`: **optional** reference to a Service+port where the extension’s webhook-style server is running
        
    - `status.conditions`: tracks whether the target Service endpoints are reachable
        
- **Discovery and Routing**
    
    - Appearance in `/apis`: all APIService names (even those without `.spec.service`) show up in discovery.
        
    - Only those with a healthy `.spec.service` get added to the proxy map and actually receive traffic.
        

> **Example: Registering a “widgets.mycompany.io/v1alpha1” API**
> 
> ```yaml
> apiVersion: apiregistration.k8s.io/v1
> kind: APIService
> metadata:
>   name: v1alpha1.widgets.mycompany.io
> spec:
>   group: widgets.mycompany.io
>   version: v1alpha1
>   insecureSkipTLSVerify: true
>   service:
>     namespace: widget-system
>     name: widget-apiserver
>     port: 443
>   groupPriorityMinimum: 1000
>   versionPriority: 10
> ```
> 
> Once created, `/apis/widgets.mycompany.io/v1alpha1/...` is routed to the `widget-apiserver` Service.


There are multiple ==controllers== which are responsible for creating ==APIService objects for the CRDs== that are applied on the cluster. By creating this APIService objects, the newly added group/version/kind (the CR) will be discoverable in the main API.

## Mutating and validating webhooks

### 3. Mutating and Validating Webhooks

Webhooks let you inject custom logic into create/update/delete operations. They sit in the request flow **after** authentication/authorization and **before** persistence:

1. **Admission Chain Order**
    
    1. **MutatingAdmissionWebhook(s)** (can change the object)
        
    2. **Defaulting** (apply default values from schemas)
        
    3. **ValidatingAdmissionWebhook(s)** (must only approve or reject)
        
2. **Webhook Configuration Objects**
    
    - **MutatingWebhookConfiguration**
        
    - **ValidatingWebhookConfiguration**
        


## storage backend?

When installing APIs in `InstallAPIs`, for each storage provider (which is basically one for each group (e.g. apps)), we call `NewRESTStorage` which returns `APIGroupInfo` that contains the `VersionedResourcesStorageMap`. Here we create Storage instances for each resource (e.g. deployment) in each version (e.g. v1).  These `ApiGroupVersions` will later get dedicated HTTP handlers in the `registerResourceHandler` function.


for each resource a new instance of Etcd is created to handle storage related stuff. But these instances share the same underlying TCP connection due to the same transport.

`registry.Store` -(implements)-> `rest.StandardStorage`
	|
	-> s.Storage (`DryrunnableStorage`) .Storage (`storage.Interface`) -(implemented by)-> `etcd.store`


## Admission chains?

## authentication / authorization layers?

## openAPI discovery?

`/api` -> core resources (group name is "" and version is "v1" only for now)
`/apis` -> all other things?
`kubectl get --raw /api`
`kubectl get --raw /apis/<group>/<version>` 
`kubectl get --raw /openapi/v2` (or v3?)

### Why it matters

- **kubectl and clients** use discovery under the covers (`kubectl api-versions`, `kubectl api-resources`) so you don’t have to hard-code endpoints.
    
- **Aggregator** merges discovery from the core API-server and any APIService backends, so you get a single unified list under `/apis`.
    
- **Dynamic**: when you install a CRD or an APIService, discovery immediately updates—no manual config needed.

`kubectl get --raw /apis` reflect this immediately, but `kubectl` caches this under `~/.kube/cache/discovery`.

#### Important

there's something even more confusing. when I delete the `v1.apps`, it's not available in `kubectl get --raw /apis`, but it's still available in `kubectl get --raw /apis/apps/v1`.

Exactly—the Aggregator’s **discovery list** (`GET /apis`) and the **delegation of actual requests** (`GET /apis/<group>/<version>`) are driven by two different mechanisms:

1. **Discovery list is built from APIService CRs**
    
    - When you hit `GET /apis`, the Aggregator reads **all** `APIService` objects (no matter Local or not) and spits out those group/versions in its JSON.
        
    - Deleting `v1.apps` removes it from that JSON, so you no longer see “apps” in the raw `/apis` output.
        
2. **Request routing uses built-in registrations + proxy map**
    
    - On any incoming request to `/apis/apps/v1/...`, the Aggregator first checks “Do I have a proxy for `apps/v1` in my map?” → no (you deleted the APIService with a `.spec.service`), so it **delegates** to its in-process GenericAPIServer.
        
    - That in-process server still has a built-in handler for `apps/v1`, so it serves the request exactly as before.

- **Discovery** shows only what your APIService CRDs advertise.
    
- **Handler lookup** falls back to core registrations when there’s no external service to proxy to—so `/apis/apps/v1` remains live even if its APIService is gone.

Seems like APIService objects with empty `spec.service` are dummies and only there for discovery. They'll appear in the output of `GET /apis` but if you hit `GET /apis/apps/v1`, since the aggregator does not have a specific service to proxy this request to (even with an APIService, the `spec.service` is empty), you'll get a response from the core kube-apiserver which does register this group/version hard-coded in itself. So by deleting a dummy APIService you're just removing it from the discovery list. The request is going to be "delegated" to the kube-apiserver (delegation target of aggregator) anyways.

#### Ordering in discovery

If you run a command without a group (e.g. `kubectl get pods`), client-go will pick the first group in the discovery list that contains “pods.” Likewise, if you refer to a resource without a version, the first version listed under that group is used by default. That default-version behavior comes straight from discovery ordering, so you’ll get the “preferred” (usually GA) version first

### role of aggregator

the aggregator server has a controller that is responsible for keeping the OpenAPI up to date. 



### handler

for each "group", there's a version -> resource -> storage map that is used to register REST routes for each resource and its actions. The handler for each route (for a given resource) is an action that is done via the storage instance for that resource.

Like for deployments, the "GET" handler, is deploymentStorage.Get() (note that deploymentStorage is an arbitrary name)