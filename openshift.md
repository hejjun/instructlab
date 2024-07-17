# Db2 for Red Hat OpenShift and Kubernetes
Last Updated: 2024-07-10
Db2 can be deployed in a Red Hat® OpenShift®(RHOS) cluster as a containerized micro-service, or pod, managed by Kubernetes.

While it is possible to deploy the containerized version of Db2 on other Kubernetes-managed container platforms, the documentation focuses on the Red Hat OpenShift deployment.

You deploy Db2 to your OpenShift cluster through a series of API calls to the Db2® Operator. The table below lists the Db2 Operators that have been released in the version v110508.x to v110509.x channels, and their supported Db2 versions for deployment on OpenShift.

The Db2 containerized solution utilizes both OpenShift and Kubernetes. This relationship determines the lifespan of each Db2 containerized solution. When the supporting OpenShift and Kubernetes versions are no longer supported, any Db2 containerized solution that is based on those versions is also no longer supported.
