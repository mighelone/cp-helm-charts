# CP-Zookeeper Helm Chart
This chart bootstraps an an ensemble of Confluent Zookeeper

## Prerequisites
* Kubernetes 1.9.2+
* Helm 2.8.2+

## Developing Environment: 
* [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine/)
* [Pivotal Container Service (PKS)](https://pivotal.io/platform/pivotal-container-service)

## Docker Image Source: 
* DockerHub - ConfluentInc: https://hub.docker.com/u/confluentinc/

## Installing the Chart
```console
$ git clone https://github.com/confluentinc/cp-helm-charts.git
$ cd cp-helm-charts/charts
$ helm install ./cp-zookeeper
```
To install with a specific name, you can do:
```console
helm install --name my-zookeeper ./cp-zookeeper
```

### Installed Components
You can use `helm status <release name>` to view all of the installed components.

For example:
```console{%raw}
$ helm status unsung-salamander

RESOURCES:
==> v1/Service
NAME                                     TYPE       CLUSTER-IP     EXTERNAL-IP  PORT(S)            AGE
unsung-salamander-cp-zookeeper-headless  ClusterIP  None           <none>       2888/TCP,3888/TCP  6m
unsung-salamander-cp-zookeeper           ClusterIP  10.19.241.242  <none>       2181/TCP           6m

==> v1beta1/StatefulSet
NAME                            DESIRED  CURRENT  AGE
unsung-salamander-cp-zookeeper  3        3        6m

==> v1beta1/PodDisruptionBudget
NAME                                MIN AVAILABLE  MAX UNAVAILABLE  ALLOWED DISRUPTIONS  AGE
unsung-salamander-cp-zookeeper-pdb  N/A            1                1                    6m

==> v1/Pod(related)
NAME                              READY  STATUS   RESTARTS  AGE
unsung-salamander-cp-zookeeper-0  1/1    Running  0         6m
unsung-salamander-cp-zookeeper-1  1/1    Running  0         6m
unsung-salamander-cp-zookeeper-2  1/1    Running  0         6m
```
There are 
1. A [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) `unsung-salamander-cp-zookeeper` which contains 3 Zookeeper [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/): `unsung-salamander-cp-zookeeper-<0|1|2>`. Each Pod has a single container running a ZooKeeper server.
1. A [PodDisruptionBudget](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/) `unsung-salamander-cp-zookeeper-pdb` to ensure service availability during planned maintenance.
1. A [Service](https://kubernetes.io/docs/concepts/services-networking/service/) `unsung-salamander-cp-zookeeper` for clients to connect to Zookeeper.
1. A [Headless Service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services) `unsung-salamander-cp-zookeeper-headless` to control the network domain for the ZooKeeper processes.

## Configuration
You can specify each parameter using the `--set key=value[,key=value]` argument to `helm install`.

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example,

```console
$ helm install --name my-zookeeper -f my-values.yaml ./cp-zookeeper
```

> **Tip**: A default [values.yaml](values.yaml) is provided

### Zookeeper Ensemble
The configuration parameters in this section control the resources requested and utilized by the ZooKeeper ensemble.

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `servers` | The number of ZooKeeper servers. This should always be (1,3,5, or 7). | `3` |

### PodDisruptionBudget
| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `minAvailable` | The minimum number of servers that must be available during evictions. This should in the interval `[(servers/2) + 1,(servers - 1)]`. If not set, `maxUnavailable: 1` will be applied. | `servers-1` |

### Image
| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `image` | Docker Image of Confluent Zookeeper. | `confluentinc/cp-zookeeper` |
| `imageTag` | Docker Image Tag of Confluent Zookeeper. | `4.0.1` |
| `imagePullPolicy` | Docker Image Tag of Confluent Zookeeper. | `IfNotPresent` |

### StatefulSet Configurations
| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `podManagementPolicy` | The Zookeeper StatefulSet Pod Management Policy: `Parallel` or `OrderedReady`. | `Parallel` |
| `updateStrategy` | The ZooKeeper StatefulSet update strategy: `RollingUpdate` or `OnDelete`. | `OnDelete` |

### Liveness and Readiness Probe Configuration
| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `probeInitialDelaySeconds` | The initial delay before the liveness and readiness probes will be invoked. | `15` |
| `probeTimeoutSeconds` | The amount of time before the probes are considered to be failed due to a timeout. | `5` |

### Confluent Zookeeper Configuration
| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `tickTime` | The length of a single tick, which is the basic time unit used by ZooKeeper, as measured in milliseconds. It is used to regulate heartbeats, and timeouts. For example, the minimum session timeout will be two ticks. | `2000` |
| `syncLimit` | Amount of time, in ticks (see `tickTime`), to allow followers to sync with ZooKeeper. If followers fall too far behind a leader, they will be dropped.  | `5` |

### Port
| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `serverPort` | The port on which the ZooKeeper servers listen for requests from other servers in the ensemble. | `2888` |
| `leaderElectionPort` | The port on which the ZooKeeper servers perform leader election. | `3888` |
| `clientPort` | The port to listen for client connections; that is, the port that clients attempt to connect to. | `2181` |

### Persistence
| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `persistence.enabled` | Whether to create a PVC. If `false`, an `emptyDir` on the host will be used. | `true` |
| `persistence.size` | Size of PVC that gets created. For production deployments this value should likely be much larger. | `5Gi` |
| `persistence.storageClass` | Valid options: `nil`, `"-"`, or storage class name. See values.yaml for details. | `nil` |

### Resources
| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `resources.requests.cpu` | The amount of CPU to request. | see [values.yaml](values.yaml) for details |
| `resources.requests.memory` | The amount of memory to request. | see [values.yaml](values.yaml) for details |
| `resources.requests.limit` | The upper limit CPU usage for a Zookeeper Pod. | see [values.yaml](values.yaml) for details |
| `resources.requests.limit` | The upper limit memory usage for a Zookeeper Pod. | see [values.yaml](values.yaml) for details |

### JMX Configuration
| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `jmx.port` | The jmx port which JMX style metrics are exposed. | `5555` |

### Prometheus JMX Exporter Configuration
| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `prometheus.jmx.enabled` | Whether or not to install Prometheus JMX Exporter as a sidecar container and expose JMX metrics to Prometheus. | `true` |
| `prometheus.jmx.image` | Docker Image for Prometheus JMX Exporter container. | `solsson/kafka-prometheus-jmx-exporter@sha256` | 
| `prometheus.jmx.imageTag` | Docker Image Tag for Prometheus JMX Exporter container. | `a23062396cd5af1acdf76512632c20ea6be76885dfc20cd9ff40fb23846557e8` |
| `prometheus.jmx.port` | JMX Exporter Port which exposes metrics in Prometheus format for scraping. | `5556` |