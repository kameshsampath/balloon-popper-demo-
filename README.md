# Getting Started with Streaming Analytics

**TODO**

## Pre-req

## Tools

- docker desktop
- [k3d](https://k3d.io)
- [direnv](https://direnv.net)
- rpk(optional)

### DNSmasq

- https://gist.github.com/ogrrd/5831371?permalink_comment_id=4790334#gistcomment-4790334
- https://gist.github.com/ogrrd/5831371?permalink_comment_id=4825523#gistcomment-4825523

## Setup

### Prepare Kubernetes Environment

Install Operator Hub,

```shell
$PROJECT_HOME/bin/setup.sh
```

The script creates [k3s](https://k3s.io) Kubernetes cluster with the following components

- Cert Manager
- Strimzi Kafka
- RisingWave
- MINIO for Iceberg Storage
- Postgres for Iceberg Catalog Store
- Apache Polaris/Nessie - Iceberg REST Catalog impl

> [!IMPORTANT]
> It will take approx 2-5 mins for the cluster to be ready depending on the internet bandwidth

### Ensure all Services

### Kafka

Services and Pods in `kafka` namespace:

```shell
kubectl get pods,svc -n kafka
```

```text
NAME                                             READY   STATUS    RESTARTS   AGE
pod/my-cluster-dual-role-0                       1/1     Running   0          17m
pod/my-cluster-entity-operator-9987555fc-lhbhb   2/2     Running   0          16m
pod/strimzi-cluster-operator-76b947897f-j9h54    1/1     Running   0          17m

NAME                                          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                                        AGE
service/my-cluster-dual-role-extplain-0       NodePort    10.43.124.55    <none>        9094:31700/TCP                                 17m
service/my-cluster-kafka-bootstrap            ClusterIP   10.43.249.38    <none>        9091/TCP,9092/TCP,9093/TCP                     17m
service/my-cluster-kafka-brokers              ClusterIP   None            <none>        9090/TCP,9091/TCP,8443/TCP,9092/TCP,9093/TCP   17m
service/my-cluster-kafka-extplain-bootstrap   NodePort    10.43.172.231   <none>        9094:31905/TCP                                 17m
```

If you had [rpk](https://docs.redpanda.com/current/get-started/rpk/) cli installed check the cluster info using the command:

```
rpk -X brokers=localhost:19094 cluster info
```

A successful setup should list the following information:

```text
CLUSTER
=======
xRXsZDpAQuCw2gz7_8LXHg

BROKERS
=======
ID    HOST        PORT
0*    172.18.0.3  31700

TOPICS
======
NAME          PARTITIONS  REPLICAS
balloon-game  1           1
game-scores   1           1
test          1           1
```

### Risingwave

Services and Pods in `risingwave` namespace:

```shell
kubectl get pods,svc -n risingwave
```

```shell
NAME                                       READY   STATUS    RESTARTS   AGE
pod/risingwave-compactor-994cbb46d-f8cfw   1/1     Running   0          20m
pod/risingwave-compute-0                   1/1     Running   0          20m
pod/risingwave-frontend-7bc5d75fbf-xfmzc   1/1     Running   0          20m
pod/risingwave-meta-0                      1/1     Running   0          20m
pod/risingwave-minio-866bf464fb-kdf56      1/1     Running   0          20m
pod/risingwave-postgresql-0                1/1     Running   0          20m

NAME                                  TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
service/risingwave                    NodePort    10.43.37.192   <none>        4567:31910/TCP               20m
service/risingwave-compute-headless   ClusterIP   None           <none>        5688/TCP,1222/TCP            20m
service/risingwave-meta-headless      ClusterIP   None           <none>        5690/TCP,5691/TCP,1250/TCP   20m
service/risingwave-minio              ClusterIP   10.43.4.95     <none>        9000/TCP,9001/TCP            20m
service/risingwave-postgresql         ClusterIP   10.43.86.233   <none>        5432/TCP                     20m
service/risingwave-postgresql-hl      ClusterIP   None           <none>        5432/TCP                     20m
```

If you got `psql` installed you can shell into SQL using the command,

```shell
psql -h localhost -p 14567 -U root -d dev
```

### Iceberg - REST Catalog

The `iceberg` namespace will house MINIO for local s3 compatible object storage, Postgres to be used as catalog store for REST Catalog implementations like Nessie, Polaris.

```shell
kubectl get pods,svc -n iceberg
```

```text
NAME                         READY   STATUS    RESTARTS   AGE
pod/minio-58d6bc6686-wzggl   1/1     Running   0          24m
pod/postgresql-0             1/1     Running   0          10m

NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                         AGE
service/minio           NodePort    10.43.226.2     <none>        9000:31915/TCP,9001:31920/TCP   24m
service/postgresql      ClusterIP   10.43.196.157   <none>        5432/TCP                        10m
service/postgresql-hl   ClusterIP   None            <none>        5432/TCP                        10m
```

The MINIO can be accessed using the URL <http://localhost:19090/browser>, default credentials `admin/password`.

#### Nessie

#### Polaris
