A ReplicationController ensures that a specified number of pod replicas are running at any one time. In other words, a ReplicationController makes sure that a pod or a homogeneous set of pods is always up and available.

>[!note]
A [[deployment]] that configures a [[replicasets]] is now the recommended way to set up replication.


### How a ReplicationController Works ?
If there are too many pods, the ReplicationController terminates the extra pods. If there are too few, the ReplicationController starts more pods. Unlike manually created pods, the pods maintained by a ReplicationController are automatically replaced if they fail, are deleted, or are terminated. For example, your pods are re-created on a node after disruptive maintenance such as a kernel upgrade. For this reason, you should use a ReplicationController even if your application requires only a single pod. A ReplicationController is similar to a process supervisor, but instead of supervising individual processes on a single node, the ReplicationController supervises multiple pods across multiple nodes.

### Running an example ReplicationController

This example ReplicationController config runs three copies of the nginx web server.

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80

```
### Writing a ReplicationController Spec
As with all other [[openshift]] config, a ReplicationController needs `apiVersion`, `kind`, and `metadata` fields. The name of a ReplicationController object must be a valid DNS subdomain name.

A ReplicationController also needs a `.spec` section.

#### Pod template
The `.spec.template` is the only required field of the .spec.

The `.spec.template` is a pod template. It has exactly the same schema as a Pod, except it is nested and does not have an apiVersion or kind.

In addition to required fields for a Pod, a pod template in a ReplicationController must specify appropriate labels and an appropriate restart policy.

Only a `.spec.template.spec.restartPolicy` equal to `Always` is allowed, which is the default if not specified.

#### Labels on the ReplicationController
The ReplicationController can itself have labels (`.metadata.labels`). Typically, you would set these the same as the .spec.template`.metadata.labels`; if `.metadata.labels` is not specified then it defaults to .spec.template`.metadata.labels`. However, they are allowed to be different, and the `.metadata.labels` do not affect the behavior of the ReplicationController.

#### Pod Selector
The `.spec.selector` field is a label selector. A ReplicationController manages all the pods with labels that match the selector. It does not distinguish between pods that it created or deleted and pods that another person or process created or deleted. This allows the ReplicationController to be replaced without affecting the running pods.

If specified, the `.spec.template.metadata.labels` must be equal to the `.spec.selector`, or it will be rejected by the API. If `.spec.selector` is unspecified, it will be defaulted to `.spec.template.metadata.labels`.

Also you should not normally create any pods whose labels match this selector, either directly, with another ReplicationController, or with another controller such as Job. If you do so, the ReplicationController thinks that it created the other pods. Kubernetes does not stop you from doing this.

#### Multiple Replicas
You can specify how many pods should run concurrently by setting `.spec.replicas` to the number of pods you would like to have running concurrently. The number running at any time may be higher or lower, such as if the replicas were just increased or decreased, or if a pod is gracefully shutdown, and a replacement starts early.

If you do not specify `.spec.replicas`, then it defaults to 1.

### Working with ReplicationControllers

#### Deleting a ReplicationController and its Pods
To delete a ReplicationController and all its pods, use `kubectl delete`. Kubectl will scale the ReplicationController to zero and wait for it to delete each pod before deleting the ReplicationController itself. If this kubectl command is interrupted, it can be restarted.

#### Deleting only a ReplicationController
You can delete a ReplicationController without affecting any of its pods.

Using kubectl, specify the `--cascade=orphan` option to kubectl delete.

Once the original is deleted, you can create a new ReplicationController to replace it. As long as the old and new `.spec.selector` are the same, then the new one will adopt the old pods. However, it will not make any effort to make existing pods match a new, different pod template.

#### Isolating pods from a ReplicationController 
Pods may be removed from a ReplicationController's target set by changing their labels. This technique may be used to remove pods from service for debugging and data recovery. Pods that are removed in this way will be replaced automatically (assuming that the number of replicas is not also changed).