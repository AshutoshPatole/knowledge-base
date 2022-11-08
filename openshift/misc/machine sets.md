
OpenShift Container Platform assigns hosts different roles. These roles define the function of the machine within the cluster. The cluster contains definitions for the standard master and worker role types.

In a Kubernetes cluster, the worker nodes are where the actual workloads requested by Kubernetes users run and are managed. The worker nodes advertise their capacity and the scheduler, which is part of the master services, determines on which nodes to start containers and pods. Important services run on each worker node, including CRI-O, which is the container engine, Kubelet, which is the service that accepts and fulfills requests for running and stopping container workloads, and a service proxy, which manages communication for pods across workers.

In OpenShift Container Platform, **machine sets** control the worker machines. Machines with the worker role drive compute workloads that are governed by a specific machine pool that autoscales them. Because OpenShift Container Platform has the capacity to support multiple machine types, the worker machines are classed as compute machines. 

>[!info]
Machine sets are groupings of machine resources under the `machine-api` namespace. Machine sets are configurations that are designed to start new machines on a specific cloud provider. Conversely, [[machine config pool]] are part of the [[machine config operator]] namespace. An MCP is used to group machines together so the MCO can manage their configurations and facilitate their upgrades.


Why master nodes are not inside machine set?
In a Kubernetes cluster, the control plane nodes (also known as the master nodes) run services that are required to control the Kubernetes cluster. They contain more than just the Kubernetes services for managing the OpenShift Container Platform cluster. Instead of being grouped into a machine set, control plane machines are defined by a series of standalone machine API resources. Extra controls apply to control plane machines to prevent you from deleting all control plane machines and breaking your cluster.
