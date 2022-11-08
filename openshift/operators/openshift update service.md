
The [[OpenShift]] Update Service (OSUS) provides over-the-air updates to OpenShift Container Platform, including RHEL CoreOS (RHCOS). It provides a graph, or diagram, that contains the vertices of component Operators and the edges that connect them. The edges in the graph show which versions you can safely update to. The vertices are update payloads that specify the intended state of the managed cluster components.

The **Cluster Version Operator (CVO)** in your cluster checks with the OpenShift Update Service to see the valid updates and update paths based on current component versions and information in the graph. When you request an update, the CVO uses the release image for that update to update your cluster. The release artifacts are hosted in Quay as container images.

The Operator that manages the other Operators in an OpenShift cluster is the **Cluster Version Operator (CVO)**.

To allow the OpenShift Update Service to provide only compatible updates, a release verification pipeline drives automation. Each release artifact is verified for compatibility with supported cloud platforms and system architectures, as well as other component packages. After the pipeline confirms the suitability of a release, the OpenShift Update Service notifies you that it is available.

Two controllers run during continuous update mode. The first controller continuously updates the payload manifests, applies the manifests to the cluster, and outputs the controlled rollout status of the Operators to indicate whether they are available, upgrading, or failed. The second controller polls the OpenShift Update Service to determine if updates are available.

During the update process, the [[machine config operator]] applies the new configuration to your cluster machines. The MCO cordons the number of nodes as specified by the maxUnavailable field on the machine configuration pool and marks them as unavailable. By default, this value is set to 1. The MCO then applies the new configuration and reboots the machine.

If you use RHEL machines as workers, the MCO **does not** update the kubelet because you must update the OpenShift API on the machines first.

With the specification for the new version applied to the old kubelet, the RHEL machine cannot return to the Ready state. You cannot complete the update until the machines are available. However, the maximum number of unavailable nodes is set to ensure that normal cluster operations can continue with that number of machines out of service.