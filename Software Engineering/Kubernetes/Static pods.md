
Source: https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/?utm_source=chatgpt.com

Summarized by Google NotebookLM

Here's a summary with important technical details:

- **What are Static Pods?**
    
    - **Managed directly by the kubelet daemon** on a specific node, without the Kubernetes API server observing them.
    - Unlike Pods managed by the control plane (e.g., Deployments), the **kubelet watches each static Pod and restarts it if it fails**.
    - They are **always bound to one Kubelet on a specific node**.
    - The kubelet automatically attempts to create a **mirror Pod on the Kubernetes API server** for each static Pod.
    - Mirror Pods make static Pods visible on the API server, but **they cannot be controlled from there**.
    - The names of mirror Pods are suffixed with the node hostname (e.g., `static-web-my-node1`).
    - **Labels from the static Pod are propagated to the mirror Pods**, allowing them to be used with selectors.
    - Static Pods **do not support ephemeral containers**.
    - The `spec` of a static Pod **cannot refer to other API objects** like `ServiceAccount`, `ConfigMap`, or `Secret`.
    - If you need to run a Pod on every node in a clustered Kubernetes environment, a **DaemonSet is generally recommended instead of static Pods**.
- **How to Create a Static Pod:** Static Pods can be configured using either a filesystem-hosted or web-hosted configuration file.
    
    - **Filesystem-hosted static Pod manifest:**
        
        - Manifests are standard Pod definitions in JSON or YAML format.
        - You specify a directory using the `staticPodPath: <the directory>` field in the kubelet configuration file.
        - The kubelet periodically scans this directory and creates/deletes static Pods as YAML/JSON files appear or disappear.
        - **Files starting with dots are ignored** by the kubelet.
        - Example: Create a `static-web.yaml` file in `/etc/kubernetes/manifests` on the chosen node (e.g., `my-node1`).
        - **Configure the kubelet** on that node to set the `staticPodPath` value in its configuration file.
        - An alternative, **deprecated method** is to start the kubelet with the `--pod-manifest-path=/etc/kubernetes/manifests/` argument.
        - After configuration, **restart the kubelet** (e.g., `systemctl restart kubelet` on Fedora).
    - **Web-hosted static Pod manifest:**
        
        - Kubelet periodically downloads a file specified by the `--manifest-url=<URL>` argument.
        - This file is interpreted as a JSON/YAML file containing Pod definitions.
        - The kubelet refetches the manifest on a schedule and applies changes.
        - To use this, create a YAML file and store it on a web server.
        - **Configure the kubelet** on your selected node by editing its configuration (e.g., `/etc/kubernetes/kubelet` on Fedora) to include `--manifest-url=<manifest-url>` in `KUBELET_ARGS`.
        - **Restart the kubelet** (e.g., `systemctl restart kubelet` on Fedora).
- **Observing Static Pod Behavior:**
    
    - When the kubelet starts, it automatically starts all defined static Pods.
    - You can view running containers (including static Pods) on the node using `crictl ps`.
    - You can see the mirror Pod on the API server using `kubectl get pods`. The Pod name will include the node's hostname.
    - **The kubelet must have permission to create the mirror Pod** in the API server.
    - If you use `kubectl delete pod` to delete the mirror Pod from the API server, the **kubelet _doesn't_ remove the static Pod**; it will reappear as the kubelet restarts it.
    - If you manually stop the container on the node (e.g., `crictl stop <container_id>`), the **kubelet will notice and automatically restart the Pod** after a short delay.
    - Logs for static Pod containers can be viewed using `crictl logs <container_id>`.
- **Dynamic Addition and Removal:**
    
    - For filesystem-hosted configurations, the running kubelet **periodically scans the configured directory for changes**.
    - It adds or removes Pods as files appear or disappear in this directory. Moving a static Pod manifest out of the directory will stop the Pod, and moving it back will restart it.

The article also provides information on what's next, such as generating static Pod manifests for control plane components and local etcd, and debugging nodes with `crictl`.