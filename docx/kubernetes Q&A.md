1. **Describe the steps from packing container images to running containers.**
- To run an application in Kubernetes, we first need to package it up into one or more container images, push those images to an image registry, and then post a description of our app to the Kubernetes API server.
- The description includes information such as the container image or images that contain our application components, how those components are related to each other, and which ones need to be run co-located (together on the same node) and which don’t.
- For each component, we can also specify how many replicas we want to run. Additionally, the description also includes which of those components provide a service to either internal or external clients and should be exposed through a single IP address and made discoverable to the other components.
- When the API server processes our app's description, the Scheduler schedules the specified groups of containers onto the available worker nodes based on computational resources required by each group and the unallocated resources on each node at that moment.
- The Kubelet on those nodes then instructs the Container Runtime (Docker, rkt) to pull the required container images and run the containers.
2. **Why do we even need pods? Why can't we use containers directly?**
- Containers are designed to run only a single process per container (unless the process itself spawns child processes). If we run multiple unrelated processes in a single container, it is our responsibility to keep all those processes running, manage their logs, and so on.
- For example, we'd have to include a mechanism for automatically restarting individual processes if they crash. Also, all those processes would log to the same standard output, so we'd have a hard time figuring out what process logged what.
- Therefore, we need to run each process in its own container. That's how Docker and Kubernetes are meant to be used.
All containers of a pod run under the same Network (so containers share network interaces hence they share the same IP address and port space) and UTS (UNIX Time Sharing) namespaces (share the same hsotname).
- Because containers in a pod run in the same Network namespace, which means processes running in containers of the same pod need to take care not to bind to the same port numbers, or they'll run into port conflicts.
- All the containers in a pod also have the same loopback network interface, so a container can communicate with other containers in the same pod through localhost.
3. **Namespace**
- We should know what namespaces don't provide, at least, not out of the box.
Although namespaces allow us to isolate objects into distinct groups, they don't provide any kind of isolation of running objects.
- For example, we may think that when different users deploy pods across different namespaces, those pods are isolated from each other and can't communicate, but that's not necessarily the case.
- Whether namespaces provide network isolation depends on which networking solution is deployed with Kubernetes. When the solution doesn't provide inter-namespace network isolation, if a pod in namespace "foo" knows the IP address of a pod in namespace "bar", there is nothing preventing it from sending traffic, such as HTTP requests, to the other pod.

4. **Why Ingresses are needed?**
- One important reason is that each LoadBalancer service requires its own load balancer with its own public IP address, whereas an Ingress only requires one, even when providing access to dozens of services.
- The downside of the LoadBalancer serviceis that each service we expose with a LoadBalancer will get its own IP address, and we have to pay for a LoadBalancer per exposed service, which can get expensive.
- When a client sends an HTTP request to the Ingress, the host and path in the request determine which service the request is forwarded to.
- Ingress is the most useful if we want to expose multiple services under the same IP address, and we only pay for one load balancer.
- Ingresses operate at the application layer of the network stack (HTTP) and can provide features such as cookie-based session affinity and the like, which services can't.

5. **Autoscaling:**
   
*Horizontal Pod Autoscaling (HPA):*
- We can add or remove pod replicas in response to changes in demand for our applications. The Horizontal Pod Autoscaler (HPA) can manage scaling these workloads for us automatically.
The HPA is ideal for scaling stateless applications, although it can also support scaling stateful sets. Using HPA in combination with cluster autoscaling can help us achieve cost savings for workloads that see regular changes in demand by reducing the number of active nodes as the number of pods decreases.

*Cluster Autoscaler:*
- While HPA scales the number of running pods in a cluster, the cluster autoscaler can change the number of nodes in a cluster.

*Vertical Pod Autoscaling (VPA):*
- The default Kubernetes scheduler overcommits CPU and memory reservations on a node, with the philosophy that most containers will stick closer to their initial requests than to their requested upper limit. The Vertical Pod Autoscaler (VPA) can increase and decrease the CPU and memory resource requests of pod containers to better match the allocated cluster resource allotment to actual usage.

6. **Kubernetes compute resources**:\
- When authoring the pods, we can optionally specify how much CPU and memory (RAM) each container needs in order to better schedule pods in the cluster and ensure satisfactory performance.\
CPU is measured in **millicores**.\
Each node in the cluster introspects the operating system to determine the amount of CPU cores on the node and then multiples that value by 1000 to express its total capacity. For example, If we have 4 cores, then the CPU capacity of the node is 4000m.

                    4 * 1000m = 4000m

  **CPU requests**:\
    Each container in a pod may specify the amount of CPU it requests on a node.\
    CPU requests are used by the scheduler to find a node with an appropriate fit for our container.\
    The CPU request represents a minimum amount of CPU that our container may consume, but if there is no contention for CPU, it may burst to use as much CPU as is available on the node.\
    If there is CPU contention on the node, CPU requests provide a relative weight across all containers on the system for how much CPU time the container may use.
  **CPU limits**:\
    Each container in a pod may specify the amount of CPU it is limited to use on a node.\
    CPU limits are used to control the maximum amount of CPU that our container may use independent of contention on the node. If a container attempts to use more than the specified limit, the system will **throttle** the container.\
    This allows our container to have a consistent level of service independent of the number of pods scheduled to the node.
  **Memory requests**:\
    By default, a container is able to consume as much memory on the node as possible.\
    In order to improve placement of pods in the cluster, it is recommended to specify the amount of memory required for a container to run.\
    The scheduler will then take available node memory capacity into account prior to binding our pod to a node.\
    A container is still able to consume as much memory on the node as possible even when specifying a request.
  **Memory limits**:\
    If we specify a memory limit, we can constrain the amount of memory our container can use.\
    For example, if we specify a limit of 200Mi, our container will be limited to using that amount of memory on the node, and if it exceeds the specified memory limit, it will be terminated and potentially restarted dependent upon the container restart policy.
**Note**:

  **Requests** (guaranted):\
    Requests are guaranteed resources that Kubernetes will ensure for the container on a node. If the required Requests resources are not available, the pod is not scheduled and lies in a Pending state until the required resources are made available (by cluster autoscaler).
  **Limits** (maximum):\
    Limits are the maximum resource that can be utilized by a container. If the container exceeds this quota, it is forcefully killed with a status OOMKilled. The resources mentioned in limits are not guaranteed to be available to the container, it may or may not be fulfilled depending upon resource allocation situation on the node.

# Tham khảo
- https://www.bogotobogo.com/DevOps/Docker/Docker_Kubernetes_Q_A_2.php