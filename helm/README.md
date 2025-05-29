# Install the cloudnative-pg with helm

## Add the repository

```console
❯ helm repo add cnpg https://cloudnative-pg.github.io/charts

"cnpg" has been added to your repositories

❯ helm repo list
NAME    URL
cnpg    https://cloudnative-pg.github.io/charts

❯ helm search repo cnpg
NAME                    CHART VERSION   APP VERSION     DESCRIPTION
cnpg/cnpg-sandbox       0.6.1           1.17.1          A sandbox for CloudNativePG
cnpg/cloudnative-pg     0.24.0          1.26.0          CloudNativePG Operator Helm Chart
cnpg/cluster            0.3.1                           Deploys and manages a CloudNativePG cluster and...
cnpg/pgbench            0.1.0                           A Helm chart that starts a CNPG Cluster and exe...
```

## Show chart default values
```console
❯ helm show values cnpg/cloudnative-pg > helm/values-default-cloudnative-pg.yaml
```

## Deploy cloudnative-pg with helm

```console
❯ helm upgrade --install cnpg \
  --namespace cnpg-system \
  --create-namespace \
  cnpg/cloudnative-pg

Release "cnpg" does not exist. Installing it now.
NAME: cnpg
LAST DEPLOYED: Wed May 28 15:12:05 2025
NAMESPACE: cnpg-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CloudNativePG operator should be installed in namespace "cnpg-system".
You can now create a PostgreSQL cluster with 3 nodes as follows:

cat <<EOF | kubectl apply -f -
# Example of PostgreSQL cluster
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: cluster-example

spec:
  instances: 3
  storage:
    size: 1Gi
EOF

kubectl get -A cluster
```

## Check K8s resouces

```console
❯ kubectl get all -n cnpg-system
NAME                                       READY   STATUS    RESTARTS   AGE
pod/cnpg-cloudnative-pg-67db9b97fd-q4frn   1/1     Running   0          60s

NAME                           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/cnpg-webhook-service   ClusterIP   10.96.72.233   <none>        443/TCP   61s

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cnpg-cloudnative-pg   1/1     1            1           60s

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/cnpg-cloudnative-pg-67db9b97fd   1         1         1       60s
```

## Deploy the postgreSQL instance with cloudnative-pg

```console
❯ kubectl apply -f helm/cluster-example.yaml
namespace/example created
cluster.postgresql.cnpg.io/cluster-example created
```

## Check K8s resouces
```console
❯ kubectl get all -n example
NAME                    READY   STATUS    RESTARTS   AGE
pod/cluster-example-1   1/1     Running   0          74s
pod/cluster-example-2   1/1     Running   0          43s
pod/cluster-example-3   1/1     Running   0          15s

NAME                         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/cluster-example-r    ClusterIP   10.96.97.10    <none>        5432/TCP   91s
service/cluster-example-ro   ClusterIP   10.96.217.2    <none>        5432/TCP   91s
service/cluster-example-rw   ClusterIP   10.96.65.127   <none>        5432/TCP   91s

❯ kubectl get pvc -n example
NAME                STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
cluster-example-1   Bound    pvc-c0ac8701-4007-4e5b-a13e-7cf5178f5bcd   1Gi        RWO            standard       <unset>                 2m50s
cluster-example-2   Bound    pvc-903dd3c1-fd06-49be-8f26-74e59be3484d   1Gi        RWO            standard       <unset>                 2m20s
cluster-example-3   Bound    pvc-0d097b19-b367-4399-8105-ca86b44b1461   1Gi        RWO            standard       <unset>                 110s

❯ k get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                       STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pvc-0d097b19-b367-4399-8105-ca86b44b1461   1Gi        RWO            Delete           Bound    example/cluster-example-3   standard       <unset>                          111s
pvc-903dd3c1-fd06-49be-8f26-74e59be3484d   1Gi        RWO            Delete           Bound    example/cluster-example-2   standard       <unset>                          2m21s
pvc-c0ac8701-4007-4e5b-a13e-7cf5178f5bcd   1Gi        RWO            Delete           Bound    example/cluster-example-1   standard       <unset>                          2m50
```

### Connect the postgresql instance status
```console
❯ kubectl get cluster -n example
NAME              AGE    INSTANCES   READY   STATUS                     PRIMARY
cluster-example   123m   3           3       Cluster in healthy state   cluster-example-1

❯ kubectl describe cluster -n example cluster-example
Name:         cluster-example
Namespace:    example
Labels:       <none>
Annotations:  <none>
API Version:  postgresql.cnpg.io/v1
Kind:         Cluster
Metadata:
  Creation Timestamp:  2025-05-28T20:29:25Z
  Generation:          1
  Resource Version:    3446
  UID:                 4521cf01-be5f-498c-90c1-adb979a4efb5
Spec:
  Affinity:
    Pod Anti Affinity Type:  preferred
  Bootstrap:
    Initdb:
      Database:             app
      Encoding:             UTF8
      Locale C Type:        C
      Locale Collate:       C
      Owner:                app
  Enable PDB:               true
  Enable Superuser Access:  false
  Failover Delay:           0
  Image Name:               ghcr.io/cloudnative-pg/postgresql:17.5
  Instances:                3
  Log Level:                info
  Max Sync Replicas:        0
  Min Sync Replicas:        0
  Monitoring:
    Custom Queries Config Map:
      Key:                    queries
      Name:                   cnpg-default-monitoring
    Disable Default Queries:  false
    Enable Pod Monitor:       false
  Postgres GID:               26
  Postgres UID:               26
  Postgresql:
    Parameters:
      archive_mode:                on
      archive_timeout:             5min
      dynamic_shared_memory_type:  posix
      full_page_writes:            on
      log_destination:             csvlog
      log_directory:               /controller/log
      log_filename:                postgres
      log_rotation_age:            0
      log_rotation_size:           0
      log_truncate_on_rotation:    false
      logging_collector:           on
      max_parallel_workers:        32
      max_replication_slots:       32
      max_worker_processes:        32
      shared_memory_type:          mmap
      shared_preload_libraries:
      ssl_max_protocol_version:    TLSv1.3
      ssl_min_protocol_version:    TLSv1.3
      wal_keep_size:               512MB
      wal_level:                   logical
      wal_log_hints:               on
      wal_receiver_timeout:        5s
      wal_sender_timeout:          5s
    Sync Replica Election Constraint:
      Enabled:              false
  Primary Update Method:    restart
  Primary Update Strategy:  unsupervised
  Replication Slots:
    High Availability:
      Enabled:      true
      Slot Prefix:  _cnpg_
    Synchronize Replicas:
      Enabled:        true
    Update Interval:  30
  Resources:
  Smart Shutdown Timeout:  180
  Start Delay:             3600
  Stop Delay:              1800
  Storage:
    Resize In Use Volumes:  true
    Size:                   1Gi
  Switchover Delay:         3600
Status:
  Available Architectures:
    Go Arch:  amd64
    Hash:     024c5162e1dee336b21121d781397b347d04939a857d9d8a47664d1796e8b7c9
    Go Arch:  arm64
    Hash:     63f3a3fbe19b8fe1c26a0e516ecb965c3412e390729a02706825399f585d9298
  Certificates:
    Client CA Secret:  cluster-example-ca
    Expirations:
      Cluster - Example - Ca:           2025-08-26 20:24:25 +0000 UTC
      Cluster - Example - Replication:  2025-08-26 20:24:25 +0000 UTC
      Cluster - Example - Server:       2025-08-26 20:24:25 +0000 UTC
    Replication TLS Secret:             cluster-example-replication
    Server Alt DNS Names:
      cluster-example-rw
      cluster-example-rw.example
      cluster-example-rw.example.svc
      cluster-example-rw.example.svc.cluster.local
      cluster-example-r
      cluster-example-r.example
      cluster-example-r.example.svc
      cluster-example-r.example.svc.cluster.local
      cluster-example-ro
      cluster-example-ro.example
      cluster-example-ro.example.svc
      cluster-example-ro.example.svc.cluster.local
    Server CA Secret:             cluster-example-ca
    Server TLS Secret:            cluster-example-server
  Cloud Native PG Commit Hash:    1535f3c17
  Cloud Native PG Operator Hash:  024c5162e1dee336b21121d781397b347d04939a857d9d8a47664d1796e8b7c9
  Conditions:
    Last Transition Time:  2025-05-28T20:35:18Z
    Message:               Cluster is Ready
    Reason:                ClusterIsReady
    Status:                True
    Type:                  Ready
    Last Transition Time:  2025-05-28T20:31:17Z
    Message:               Continuous archiving is working
    Reason:                ContinuousArchivingSuccess
    Status:                True
    Type:                  ContinuousArchiving
  Config Map Resource Version:
    Metrics:
      Cnpg - Default - Monitoring:  2377
  Current Primary:                  cluster-example-1
  Current Primary Timestamp:        2025-05-28T20:31:16.970395Z
  Healthy PVC:
    cluster-example-1
    cluster-example-2
    cluster-example-3
  Image:  ghcr.io/cloudnative-pg/postgresql:17.5
  Instance Names:
    cluster-example-1
    cluster-example-2
    cluster-example-3
  Instances:  3
  Instances Reported State:
    cluster-example-1:
      Ip:            10.244.2.4
      Is Primary:    true
      Time Line ID:  1
    cluster-example-2:
      Ip:            10.244.3.4
      Is Primary:    false
      Time Line ID:  1
    cluster-example-3:
      Ip:            10.244.1.5
      Is Primary:    false
      Time Line ID:  1
  Instances Status:
    Healthy:
      cluster-example-1
      cluster-example-2
      cluster-example-3
  Latest Generated Node:  3
  Managed Roles Status:
  Pg Data Image Info:
    Image:          ghcr.io/cloudnative-pg/postgresql:17.5
    Major Version:  17
  Phase:            Cluster in healthy state
  Pooler Integrations:
    Pg Bouncer Integration:
  Pvc Count:        3
  Read Service:     cluster-example-r
  Ready Instances:  3
  Secrets Resource Version:
    Application Secret Version:  2344
    Client Ca Secret Version:    2341
    Replication Secret Version:  2343
    Server Ca Secret Version:    2341
    Server Secret Version:       2342
  Switch Replica Cluster Status:
  Target Primary:            cluster-example-1
  Target Primary Timestamp:  2025-05-28T20:29:25.965912Z
  Timeline ID:               1
  Topology:
    Instances:
      cluster-example-1:
      cluster-example-2:
      cluster-example-3:
    Nodes Used:              3
    Successfully Extracted:  true
  Write Service:             cluster-example-rw
Events:                      <none>
```

### Connect to the postgresql instance
```console
❯ kubectl get -n example secrets cluster-example-app -o jsonpath="{.data.pgpass}" | base64 -d
RT6rGgM26rcnNWrNQhgF9D42mrj0kXlrvOkXzMmJDtCaBJ7PwNDiry2hXlhJ0MzL

❯ kubectl exec -n example --stdin --tty cluster-example-1 -c postgres -- psql -h cluster-example-rw -p 5432 -U app app
Password for user app:
psql (17.5 (Debian 17.5-1.pgdg110+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off, ALPN: postgresql)
Type "help" for help.

app=>
```

```console
❯ kubectl exec -n example --stdin --tty cluster-example-1 -c postgres -- bash
postgres@cluster-example-1:/$ psql -U postgres
psql (17.5 (Debian 17.5-1.pgdg110+1))
Type "help" for help.

postgres=#

postgres=# SELECT application_name, state, sync_state, write_lag, flush_lag, replay_lag
FROM pg_stat_replication;
 application_name  |   state   | sync_state | write_lag | flush_lag | replay_lag
-------------------+-----------+------------+-----------+-----------+------------
 cluster-example-2 | streaming | quorum     |           |           |
 cluster-example-3 | streaming | quorum     |           |           |
```

```
❯ curl -sSfL \
  https://github.com/cloudnative-pg/cloudnative-pg/raw/main/hack/install-cnpg-plugin.sh | \
  sudo sh -s -- -b /usr/local/bin
cloudnative-pg/cloudnative-pg info checking GitHub for latest tag
cloudnative-pg/cloudnative-pg info found version: 1.26.0 for v1.26.0/linux/x86_64
cloudnative-pg/cloudnative-pg info installed /usr/local/bin/kubectl-cnpg

❯ which kubectl-cnpg
/usr/local/bin/kubectl-cnpg
```

### Delete netbox with helm

```console
❯ helm uninstall -n netbox netbox
release "netbox" uninstalled

❯ helm list -n netbox
NAME    NAMESPACE       REVISION        UPDATED STATUS  CHART   APP VERSION

❯ kubectl delete namespaces netbox
namespace "netbox" deleted

❯ kubectl delete -n netbox pvc --all
persistentvolumeclaim "data-netbox-postgresql-0" deleted
persistentvolumeclaim "valkey-data-netbox-valkey-primary-0" deleted
persistentvolumeclaim "valkey-data-netbox-valkey-replicas-0" deleted
persistentvolumeclaim "valkey-data-netbox-valkey-replicas-1" deleted
persistentvolumeclaim "valkey-data-netbox-valkey-replicas-2" deleted

❯ kubectl -n netbox get pvc,pv
No resources found
```
