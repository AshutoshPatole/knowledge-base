[[machine config operator]]
## Bundle
In the Bundle Format, a bundle is a collection of an Operator [CSV](#cluster-service-version), manifests, and metadata. Together, they form a unique version of an Operator that can be installed onto the cluster.

## Bundle image
In the Bundle Format, a bundle image is a container image that is built from Operator manifests and that contains one bundle. Bundle images are stored and distributed by __Open Container Initiative (OCI)__ spec container registries, such as _Quay.io_ or _DockerHub_.

## Catalog source
A catalog source is a repository of CSVs, CRDs, and packages that define an application.

## Catalog image
In the Package Manifest Format, a catalog image is a containerized datastore that describes a set of Operator metadata and update metadata that can be installed onto a cluster using OLM.

## Channel
A channel defines a stream of updates for an Operator and is used to roll out updates for subscribers. The head points to the latest version of that channel. For example, a stable channel would have all stable versions of an Operator arranged from the earliest to the latest.

An Operator can have several channels, and a subscription binding to a certain channel would only look for updates in that channel.

## Channel head
A channel head refers to the latest known update in a particular channel.

## Cluster service version
A cluster service version (CSV) is a YAML manifest created from Operator metadata that assists OLM in running the Operator in a cluster. It is the metadata that accompanies an Operator container image, used to populate user interfaces with information such as its logo, description, and version.

It is also a source of technical information that is required to run the Operator, like the RBAC rules it requires and which custom resources (CRs) it manages or depends on.

## Dependency
An Operator may have a dependency on another Operator being present in the cluster. For example, the Vault Operator has a dependency on the etcd Operator for its data persistence layer.

OLM resolves dependencies by ensuring that all specified versions of Operators and CRDs are installed on the cluster during the installation phase. This dependency is resolved by finding and installing an Operator in a catalog that satisfies the required CRD API, and is not related to packages or bundles.

## Index image
In the Bundle Format, an index image refers to an image of a database (a database snapshot) that contains information about Operator bundles including CSVs and CRDs of all versions. This index can host a history of Operators on a cluster and be maintained by adding or removing Operators using the opm CLI tool.

## Install plan
An install plan is a calculated list of resources to be created to automatically install or upgrade a CSV.

## Operator group
An Operator group configures all Operators deployed in the same namespace as the OperatorGroup object to watch for their CR in a list of namespaces or cluster-wide.

## Package
In the Bundle Format, a package is a directory that encloses all released history of an Operator with each version. A released version of an Operator is described in a CSV manifest alongside the CRDs.

## Registry
A registry is a database that stores bundle images of Operators, each with all of its latest and historical versions in all channels.

## Subscription
A subscription keeps CSVs up to date by tracking a channel in a package.

## Update graph
An update graph links versions of CSVs together, similar to the update graph of any other packaged software. Operators can be installed sequentially, or certain versions can be skipped. The update graph is expected to grow only at the head with newer versions being added.