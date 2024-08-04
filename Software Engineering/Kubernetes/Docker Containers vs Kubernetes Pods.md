#docker #kubernetes #container #pod #networking #cgroup #ipc #pid #uts #netns

### Containers vs. Pods

**Authors:**  
**Date:**

#### Introduction
Containers, popularized by Docker and standardized by OCI, generally run a single service per container, promoting isolation and scalability. However, this approach limits their use as virtual machine replacements. Kubernetes introduces Pods, groups of cohesive containers, as the smallest deployable unit.

#### Understanding Pods
- Pods are assigned unique IPs and hostnames.
- Containers in a Pod communicate via localhost, resembling a mini server.
- Containers in a Pod have isolated filesystems and processes but share memory and IPC mechanisms.

#### Examining a Container
![[container-schematic.png]]
**Setup:**
```sh
docker run --name foo --rm -d --memory='512MB' --cpus='0.5' nginx:alpine
```

**Inspecting Namespaces:**
```sh
NGINX_PID=$(pgrep nginx | sort | head -n 1)
sudo lsns -p ${NGINX_PID}
```

**Namespaces used:**
- **mnt** (Mount): Isolated mount table.
- **uts** (UNIX Time-Sharing): Own hostname and domain name.
- **ipc** (Interprocess Communication): System-level IPC within the container.
- **pid** (Process ID): Visible processes only within the container.
- **net** (Network): Own set of network devices.
- **cgroup** (Cgroup): Isolated cgroup filesystem view.

Docker does not use the **user** namespace by default due to complexity and limitations.

**Inspecting Cgroups:**
```sh
sudo systemd-cgls --no-pager
ls -l /sys/fs/cgroup/system.slice/docker-<container-id>.scope/
cat /sys/fs/cgroup/system.slice/docker-<container-id>.scope/memory.max
```
Cgroups limit resources to prevent unfair consumption by neighboring processes.

#### Examining a Pod
![[pod-schematic.png]]
**Pod Setup:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: foo
spec:
  containers:
    - name: app
      image: nginx:alpine
      ports:
        - containerPort: 80
      resources:
        limits:
          memory: "256Mi"
    - name: sidecar
      image: curlimages/curl:8.3.0
      command: ["/bin/sleep", "3650d"]
      resources:
        limits:
          memory: "128Mi"
```

**Start Pod:**
```sh
kubectl apply -f pod.yaml
```

**Inspecting Pod's Containers:**
```sh
ps auxf
sudo ctr --namespace=k8s.io containers ls
sudo crictl ps
```

**Auxiliary Containers:**
Pods have auxiliary containers like `pause` to manage resources and namespaces.

**Namespaces in Pods:**
```sh
sudo lsns
sudo ls -l /proc/<pid>/ns
sudo crictl inspect <container-id> | jq .info.runtimeSpec.linux.namespaces
```
Containers in Pods share **net**, **uts**, and **ipc** namespaces of the `pause` container. Flags like `shareProcessNamespace`, `hostIPC`, `hostNetwork`, and `hostPID` can alter namespace sharing.

**Pod's Cgroups:**
```sh
sudo systemd-cgls --no-pager
```
Pods have a parent cgroup, with each container having individual resource limits.

#### Implementing Pods with Docker
**Creating a Parent Cgroup:**
```sh
sudo cat <<EOF > /etc/systemd/system/mypod.slice
[Unit]
Description=My Pod Slice

[Slice]
MemoryLimit=512M
CPUQuota=50%
EOF
sudo systemctl daemon-reload
sudo systemctl start mypod.slice
sudo systemd-cgls --no-pager --all
```

**Creating Sandbox Container:**
```sh
docker run -d --rm --name mypod_sandbox --cgroup-parent mypod.slice --ipc 'shareable' alpine sleep infinity
```

**Creating Payload Containers:**
```sh
docker run -d --rm --name app --cgroup-parent mypod.slice --network container:mypod_sandbox --ipc container:mypod_sandbox nginx:alpine
docker run -d --rm --name sidecar --cgroup-parent mypod.slice --network container:mypod_sandbox --ipc container:mypod_sandbox curlimages/curl sleep 365d
```
The **uts** namespace cannot be shared with Docker's current capabilities.

**Namespace and Cgroup Validation:**
```sh
sudo lsns
sudo ls -l /proc/<pid>/ns
```

#### Summary
Containers and Pods use Linux namespaces and cgroups. Pods, however, are higher-level constructs that synchronize container lifecycles and simplify inter-container communication, closely resembling traditional VMs.

**References**
- [Containers vs. Pods](https://labs.iximiuz.com/tutorials/containers-vs-pods)
