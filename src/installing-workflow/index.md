# Installing Deis Workflow

This document is aimed at those who have already provisioned a [Kubernetes v1.2 or v1.3.4+][] cluster
and want to install Deis Workflow. If you are just getting started with Kubernetes and
Deis Workflow, follow our [quickstart guide](../quickstart/index.md) for help.

## Prerequisites

1. Verify the [Kubernetes system requirements](system-requirements.md)
1. Install [Helm Classic and Deis Workflow CLI](../quickstart/install-cli-tools.md) tools

## Check Your Setup

Check that the `helmc` command is available and the version is 0.8 or newer.

```
$ helmc --version
helmc version 0.8.0+f3cafbc
```

Ensure the `kubectl` client is installed and can connect to your Kubernetes cluster. `helmc` uses `kubectl` to interact
with your Kubernetes cluster.

You can test that `helmc` and `kubectl` are working properly by running:

```
$ helmc target
Kubernetes master is running at https://52.9.206.49
Elasticsearch is running at https://52.9.206.49/api/v1/proxy/namespaces/kube-system/services/elasticsearch-logging
Heapster is running at https://52.9.206.49/api/v1/proxy/namespaces/kube-system/services/heapster
Kibana is running at https://52.9.206.49/api/v1/proxy/namespaces/kube-system/services/kibana-logging
KubeDNS is running at https://52.9.206.49/api/v1/proxy/namespaces/kube-system/services/kube-dns
kubernetes-dashboard is running at https://52.9.206.49/api/v1/proxy/namespaces/kube-system/services/kubernetes-dashboard
Grafana is running at https://52.9.206.49/api/v1/proxy/namespaces/kube-system/services/monitoring-grafana
InfluxDB is running at https://52.9.206.49/api/v1/proxy/namespaces/kube-system/services/monitoring-influxdb
```

If you see a list of targets like the one above, `helmc` can communicate with the Kubernetes master. Double check that
the master returned by `helmc target` matches the expected cluster.

## Choose Your Deployment Strategy

Deis Workflow includes everything it needs to run out of the box. However, these defaults are aimed at simplicity rather than
production readiness. Production and staging deployments of Workflow should, at a minimum, use off-cluster storage.
Which is used by Workflow components to store and backup critical data. Should an operator need to completely re-install
Workflow, the required components can recover from off-cluster storage. See our documentation for [configuring object
storage](configuring-object-storage.md) for more details.

Workflow may also be configured to use off-cluster persistence for [Postgres](configuring-postgres.md) and
Redis. A deployment strategy that mirrors the "stateless" clusters from Deis v1 PaaS.

Last but not least, Workflow may also use a dedicated off-cluster image registry, including Docker Hub, Quay.io, ECR or
GCR for all container images. Read more about [configuring your registry](configuring-registry.md).

## Add the Deis Chart Repository

The [Deis Chart Repository](https://github.com/deis/charts) contains everything you need to install Deis Workflow onto
your Kubernetes cluster, with a single `helmc install` command.

Add this repository to Helm Classic:

```
$ helmc repo add deis https://github.com/deis/charts
```

## Install Deis Workflow

Now that you have Helm Classic installed and have added the Deis Chart Repository, install Workflow by running:

```
$ helmc fetch deis/workflow-v2.4.0            # fetches the chart into a
                                              # local workspace
$ helmc generate -x manifests workflow-v2.4.0 # generates various secrets
$ helmc install workflow-v2.4.0               # injects resources into
                                              # your cluster
```

Helm Classic will install a variety of Kubernetes resources in the `deis` namespace.
You'll need to wait for the pods that it launched to be ready. Monitor their status
by running:

```
$ kubectl --namespace=deis get pods
```

If you would like `kubectl` to automatically update as the pod states change, run (type Ctrl-C to stop the watch):
```
$ kubectl --namespace=deis get pods -w
```

Depending on the order in which the Workflow components initialize, some pods may restart. This is common during the
installation: if a component's dependencies are not yet available, that component will exit and Kubernetes will
automatically restart it.

Here, you can see that controller, builder and registry all took a few loops before there were able to start:
```
$ kubectl --namespace=deis get pods
NAME                          READY     STATUS    RESTARTS   AGE
deis-builder-hy3xv            1/1       Running   5          5m
deis-controller-g3cu8         1/1       Running   5          5m
deis-database-rad1o           1/1       Running   0          5m
deis-logger-fluentd-1v8uk     1/1       Running   0          5m
deis-logger-fluentd-esm60     1/1       Running   0          5m
deis-logger-sm8b3             1/1       Running   0          5m
deis-minio-4ww3t              1/1       Running   0          5m
deis-registry-asozo           1/1       Running   1          5m
deis-router-k1ond             1/1       Running   0          5m
deis-workflow-manager-68nu6   1/1       Running   0          5m
```

Once you see all of the pods in the `READY` state, Deis Workflow is up and running!

[Kubernetes v1.2 or v1.3.4+]: system-requirements.md#kubernetes-versions
