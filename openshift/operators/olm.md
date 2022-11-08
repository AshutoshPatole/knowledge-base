Operator Lifecycle Manager (OLM) helps users install, update, and manage the lifecycle of Kubernetes native applications (Operators) and their associated services running across their [[OpenShift]] Container Platform clusters. It is part of the [Operator Framework](https://operatorframework.io/), an open source toolkit designed to manage Operators in an effective, automated, and scalable way.

## OLM resources
The following custom resource definitions (CRDs) are defined and managed by Operator Lifecycle Manager (OLM):

| Resource                      | Short name | Description                                                                                                                                                            |
| ----------------------------- | ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ClusterServiceVersion` | `csv`      | Application metadata. For example: name, version, icon, required resources.                                                                                            |
| `CatalogSource`               | `catsrc`   | A repository of CSVs, CRDs, and packages that define an application.                                                                                                   |
| `Subscription`                | `sub`      | Keeps CSVs up to date by tracking a channel in a package.                                                                                                              |
| `InstallPlan`                 | `ip`       | Calculated list of resources to be created to automatically install or upgrade a CSV.                                                                                  |
| `OperatorGroup`               | `og`       | Configures all Operators deployed in the same namespace as the `OperatorGroup` object to watch for their custom resource (CR) in a list of namespaces or cluster-wide. |

## Cluster service version
A _cluster service version (CSV)_ represents a specific version of a running Operator on an OpenShift Container Platform cluster. It is a YAML manifest created from Operator metadata that assists Operator Lifecycle Manager (OLM) in running the Operator in the cluster.

OLM requires this metadata about an Operator to ensure that it can be kept running safely on a cluster, and to provide information about how updates should be applied as new versions of the Operator are published. This is similar to packaging software for a traditional operating system; think of the packaging step for OLM as the stage at which you make your rpm, deb, or apk bundle.

A CSV includes the metadata that accompanies an Operator container image, used to populate user interfaces with information such as its name, version, description, labels, repository link, and logo.

A CSV is also a source of technical information required to run the Operator, such as which custom resources (CRs) it manages or depends on, RBAC rules, cluster requirements, and install strategies. This information tells OLM how to create required resources and set up the Operator as a deployment.

## Catalog source
A catalog source represents a store of metadata, typically by referencing an index image stored in a container registry. Operator Lifecycle Manager (OLM) queries catalog sources to discover and install Operators and their dependencies. The OperatorHub in the OpenShift Container Platform web console also displays the Operators provided by catalog sources.

>[!tip]
Cluster administrators can view the full list of Operators provided by an enabled catalog source on a cluster by using the **Administration → Cluster Settings → Global Configuration → OperatorHub** page in the web console.


The spec of a CatalogSource object indicates how to construct a pod or how to communicate with a service that serves the Operator Registry gRPC API.

## Subscription
A subscription, defined by a Subscription object, represents an intention to install an Operator. It is the custom resource that relates an Operator to a catalog source.

Subscriptions describe which channel of an Operator package to subscribe to, and whether to perform updates automatically or manually. If set to automatic, the subscription ensures Operator Lifecycle Manager (OLM) manages and upgrades the Operator to ensure that the latest version is always running in the cluster.

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: example-operator
  namespace: example-namespace
spec:
  channel: stable
  name: example-operator
  source: example-catalog
  sourceNamespace: openshift-marketplace
```

This Subscription object defines the name and namespace of the Operator, as well as the catalog from which the Operator data can be found. The channel, such as alpha, beta, or stable, helps determine which Operator stream should be installed from the catalog source.

In addition to being easily visible from the OpenShift Container Platform web console, it is possible to identify when there is a newer version of an Operator available by inspecting the status of the related subscription. The value associated with the `currentCSV` field is the newest version that is known to OLM, and `installedCSV` is the version that is installed on the cluster.

## Install plan
An install plan, defined by an `InstallPlan` object, describes a set of resources that Operator Lifecycle Manager (OLM) creates to install or upgrade to a specific version of an Operator. The version is defined by a cluster service version (CSV).

To install an Operator, a cluster administrator, or a user who has been granted Operator installation permissions, must first create a `Subscription` object. A subscription represents the intent to subscribe to a stream of available versions of an Operator from a catalog source. The subscription then creates an `InstallPlan` object to facilitate the installation of the resources for the Operator.

The install plan must then be approved according to one of the following approval strategies:

- If the subscription's `spec.installPlanApproval` field is set to `Automatic`, the install plan is approved automatically.

- If the subscription's `spec.installPlanApproval` field is set to `Manual`, the install plan must be manually approved by a cluster administrator or user with proper permissions.

After the install plan is approved, OLM creates the specified resources and installs the Operator in the namespace that is specified by the subscription.

## Operator groups
An Operator group, defined by the `OperatorGroup` resource, provides multitenant configuration to OLM-installed Operators. An Operator group selects target namespaces in which to generate required RBAC access for its member Operators.

The set of target namespaces is provided by a comma-delimited string stored in the `olm.targetNamespaces` annotation of a cluster service version (CSV). This annotation is applied to the CSV instances of member Operators and is projected into their deployments.