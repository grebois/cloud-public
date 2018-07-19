
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [RabbitMQ](#rabbitmq)
- [Using with make file:](#using-with-make-file)
  - [Install:](#install)
  - [Updating:](#updating)
  - [Deleting:](#deleting)
  - [Listing helm charts:](#listing-helm-charts)
- [Using Manually:](#using-manually)
- [Updating](#updating)
  - [Enabling SSL Support](#enabling-ssl-support)
- [Updating from 1.6.1](#updating-from-161)
- [Exporting metric to Prometheus](#exporting-metric-to-prometheus)
- [Verify metrics are being published](#verify-metrics-are-being-published)
- [Finally verify that the target is in prometheus](#finally-verify-that-the-target-is-in-prometheus)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

RabbitMQ
============

We are using this helm chart: https://github.com/kubernetes/charts/tree/master/stable/rabbitmq-ha

Using helm version: 2.8.1

# Using with make file:

```
export KUBE_NAMESPACE=rabbitmq
```

## Install:
```
make install
```

## Updating:
```
make upgrade
```

## Deleting:
```
make delete
```

## Listing helm charts:
```
make list
```

# Using Manually:
```
export KUBE_NAMESPACE=rabbitmq
```

```
helm repo add stable https://kubernetes-charts.storage.googleapis.com/

helm install \
--version 1.6.1 \
--name ${KUBE_NAMESPACE}-rabbitmq \
--namespace ${KUBE_NAMESPACE} \
--values ./kubernetes/pods/rabbitmq-helm/values.yaml \
--debug \
stable/rabbitmq-ha
```

# Updating
```
helm upgrade \
--version 1.6.1 \
--values ./kubernetes/pods/rabbitmq-helm/values.yaml \
${KUBE_NAMESPACE}-rabbitmq \
stable/rabbitmq-ha
```

## Enabling SSL Support

RabbitMQ Documentation: https://www.rabbitmq.com/ssl.html#enabling-ssl

Add in the certificates in this section:

```
rabbitmqCert:
  enabled: true

  # Specifies an existing secret to be used for SSL Certs
  existingSecret: ""

  ## Create a new secret using these values
  cacertfile: |
  certfile: |
  keyfile: |
```

The certs has to be base64 encoded:
```
cat certfile.ca | base64 -w0
```

Enable the SSL configurations:
```
rabbitmqAmqpsSupport:
  enabled: true

  # NodePort
  amqpsNodePort: 5671

  # SSL configuration
  config: |
    listeners.ssl.default             = 5671
    ssl_options.cacertfile            = /etc/cert/cacert.pem
    ssl_options.certfile              = /etc/cert/cert.pem
    ssl_options.keyfile               = /etc/cert/key.pem
    ssl_options.verify                = verify_peer
    ssl_options.fail_if_no_peer_cert  = false
```

# Updating from 1.6.1

First we update the helm repo

```bash
$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Skip local chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈
```

and the we run:

```bash
$ helm upgrade \
--values values.yaml \
${KUBE_NAMESPACE}-rabbitmq \
stable/rabbitmq-ha

Release "devops-rabbitmq" has been upgraded. Happy Helming!
LAST DEPLOYED: Thu Jul 19 11:55:28 2018
NAMESPACE: devops
STATUS: DEPLOYED

RESOURCES:
==> v1beta1/RoleBinding
NAME                         AGE
devops-rabbitmq-rabbitmq-ha  13m

==> v1/Service
NAME                                   TYPE       CLUSTER-IP      EXTERNAL-IP  PORT(S)                      AGE
devops-rabbitmq-rabbitmq-ha-discovery  ClusterIP  None            <none>       15672/TCP,5672/TCP,4369/TCP  13m
devops-rabbitmq-rabbitmq-ha            ClusterIP  192.168.37.135  <none>       15672/TCP,5672/TCP,4369/TCP  13m

==> v1beta1/StatefulSet
NAME                         DESIRED  CURRENT  AGE
devops-rabbitmq-rabbitmq-ha  3        3        13m

==> v1/Pod(related)
NAME                           READY  STATUS   RESTARTS  AGE
devops-rabbitmq-rabbitmq-ha-0  1/1    Running  0         13m
devops-rabbitmq-rabbitmq-ha-1  1/1    Running  0         13m
devops-rabbitmq-rabbitmq-ha-2  1/1    Running  0         12m

==> v1/Secret
NAME                         TYPE    DATA  AGE
devops-rabbitmq-rabbitmq-ha  Opaque  2     13m

==> v1/ConfigMap
NAME                         DATA  AGE
devops-rabbitmq-rabbitmq-ha  2     13m

==> v1/ServiceAccount
NAME                         SECRETS  AGE
devops-rabbitmq-rabbitmq-ha  1        13m

==> v1beta1/Role
NAME                         AGE
devops-rabbitmq-rabbitmq-ha  13m


NOTES:
** Please be patient while the chart is being deployed **

  Credentials:

    Username      : guest

    Password      : $(kubectl get secret --namespace devops devops-rabbitmq-rabbitmq-ha -o jsonpath="{.data.rabbitmq-password}" | base64 --decode)
    ErLang Cookie : $(kubectl get secret --namespace devops devops-rabbitmq-rabbitmq-ha -o jsonpath="{.data.rabbitmq-erlang-cookie}" | base64 --decode)


  RabbitMQ can be accessed within the cluster on port 5672 at devops-rabbitmq-rabbitmq-ha.devops.svc.cluster.local

  To access for outside the cluster execute the following commands:

    export POD_NAME=$(kubectl get pods --namespace devops -l "app=rabbitmq-ha" -o jsonpath="{.items[0].metadata.name}")
    kubectl port-forward $POD_NAME --namespace devops 5672:5672 15672:15672

  To Access the RabbitMQ AMQP port:

    amqp://127.0.0.1:5672/

  To Access the RabbitMQ Management interface:

    URL : http://127.0.0.1:15672


To enable mirroring for all the host:

  export POD_NAME=$(kubectl get pods --namespace devops -l "app=rabbitmq-ha" -o jsonpath="{.items[0].metadata.name}")
  kubectl exec $POD_NAME --namespace devops -- rabbitmqctl set_policy ha-all "." '{"ha-mode":"all", "ha-sync-mode":"automatic"}' --apply-to all --priority 0
```

# Exporting metric to Prometheus

The exporter is enable by default on the local valus.yaml:

```yaml
prometheus:
  ## Configures Prometheus Exporter to expose and scrape stats.
  exporter:
    enabled: true
    env: {}
    image:
      repository: kbudde/rabbitmq-exporter
      tag: v0.28.0
      pullPolicy: IfNotPresent

    ## Port Prometheus scrapes for metrics
    port: 9090

    ## Allow overriding of container resources
    resources: {}
     # limits:
     #   cpu: 200m
     #   memory: 1Gi
     # requests:
     #   cpu: 100m
     #   memory: 100Mi

  ## Prometheus is using Operator.  Setting to true will create Operator specific resources like ServiceMonitors and Alerts
  operator:
    ## Are you using Prometheus Operator? [Blog Post](https://coreos.com/blog/the-prometheus-operator.html)
    enabled: true

    ## Configures Alerts, which will be setup via Prometheus Operator / ConfigMaps.
    alerts:
      ## Prometheus exporter must be enabled as well
      enabled: true

      ## Selector must be configured to match Prometheus Install, defaulting to whats done by Prometheus Operator
      ## See [CoreOS Prometheus Chart](https://github.com/coreos/prometheus-operator/tree/master/helm)
      selector:
        role: alert-rules
      labels: {}

    serviceMonitor:
      ## Interval at which Prometheus scrapes RabbitMQ Exporter
      interval: 10s

      # Namespace Prometheus is installed in
      namespace: monitoring

      ## Defaults to whats used if you follow CoreOS [Prometheus Install Instructions](https://github.com/coreos/prometheus-operator/tree/master/helm#tldr)
      ## [Prometheus Selector Label](https://github.com/coreos/prometheus-operator/blob/master/helm/prometheus/templates/prometheus.yaml#L65)
      ## [Kube Prometheus Selector Label](https://github.com/coreos/prometheus-operator/blob/master/helm/kube-prometheus/values.yaml#L298)
      selector:
        prometheus: kube-prometheus
```

# Verify metrics are being published

```bash
$ kubectl -n rabbitmq port-forward rabbitmq-rabbitmq-ha-0 9090:9090 &
$ Forwarding from 127.0.0.1:9090 -> 9090
$ curl localhost:9090/metrics
Handling connection for 9090
# HELP go_gc_duration_seconds A summary of the GC invocation durations.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 0.000219785
go_gc_duration_seconds{quantile="0.25"} 0.000219785
go_gc_duration_seconds{quantile="0.5"} 0.000219785
go_gc_duration_seconds{quantile="0.75"} 0.000219785
go_gc_duration_seconds{quantile="1"} 0.000219785
go_gc_duration_seconds_sum 0.000219785
go_gc_duration_seconds_count 1
# HELP go_goroutines Number of goroutines that currently exist.
# TYPE go_goroutines gauge
go_goroutines 12
# HELP go_memstats_alloc_bytes Number of bytes allocated and still in use.
# TYPE go_memstats_alloc_bytes gauge
go_memstats_alloc_bytes 3.247984e+06
# HELP go_memstats_alloc_bytes_total Total number of bytes allocated, even if freed.
# TYPE go_memstats_alloc_bytes_total counter
go_memstats_alloc_bytes_total 5.633664e+06
# HELP go_memstats_buck_hash_sys_bytes Number of bytes used by the profiling bucket hash table.
# TYPE go_memstats_buck_hash_sys_bytes gauge
go_memstats_buck_hash_sys_bytes 1.509124e+06
# HELP go_memstats_frees_total Total number of frees.
# TYPE go_memstats_frees_total counter
go_memstats_frees_total 18509
# HELP go_memstats_gc_sys_bytes Number of bytes used for garbage collection system metadata.
# TYPE go_memstats_gc_sys_bytes gauge
go_memstats_gc_sys_bytes 471040
# HELP go_memstats_heap_alloc_bytes Number of heap bytes allocated and still in use.
# TYPE go_memstats_heap_alloc_bytes gauge
go_memstats_heap_alloc_bytes 3.247984e+06
# HELP go_memstats_heap_idle_bytes Number of heap bytes waiting to be used.
# TYPE go_memstats_heap_idle_bytes gauge
go_memstats_heap_idle_bytes 770048
# HELP go_memstats_heap_inuse_bytes Number of heap bytes that are in use.
# TYPE go_memstats_heap_inuse_bytes gauge
go_memstats_heap_inuse_bytes 4.603904e+06
# HELP go_memstats_heap_objects Number of allocated objects.
# TYPE go_memstats_heap_objects gauge
go_memstats_heap_objects 13600
# HELP go_memstats_heap_released_bytes_total Total number of heap bytes released to OS.
# TYPE go_memstats_heap_released_bytes_total counter
go_memstats_heap_released_bytes_total 0
# HELP go_memstats_heap_sys_bytes Number of heap bytes obtained from system.
# TYPE go_memstats_heap_sys_bytes gauge
go_memstats_heap_sys_bytes 5.373952e+06
# HELP go_memstats_last_gc_time_seconds Number of seconds since 1970 of last garbage collection.
# TYPE go_memstats_last_gc_time_seconds gauge
go_memstats_last_gc_time_seconds 1.531866935995865e+09
# HELP go_memstats_lookups_total Total number of pointer lookups.
# TYPE go_memstats_lookups_total counter
go_memstats_lookups_total 101
# HELP go_memstats_mallocs_total Total number of mallocs.
# TYPE go_memstats_mallocs_total counter
go_memstats_mallocs_total 32109
# HELP go_memstats_mcache_inuse_bytes Number of bytes in use by mcache structures.
# TYPE go_memstats_mcache_inuse_bytes gauge
go_memstats_mcache_inuse_bytes 41664
# HELP go_memstats_mcache_sys_bytes Number of bytes used for mcache structures obtained from system.
# TYPE go_memstats_mcache_sys_bytes gauge
go_memstats_mcache_sys_bytes 49152
# HELP go_memstats_mspan_inuse_bytes Number of bytes in use by mspan structures.
# TYPE go_memstats_mspan_inuse_bytes gauge
go_memstats_mspan_inuse_bytes 49552
# HELP go_memstats_mspan_sys_bytes Number of bytes used for mspan structures obtained from system.
# TYPE go_memstats_mspan_sys_bytes gauge
go_memstats_mspan_sys_bytes 65536
# HELP go_memstats_next_gc_bytes Number of heap bytes when next garbage collection will take place.
# TYPE go_memstats_next_gc_bytes gauge
go_memstats_next_gc_bytes 4.194304e+06
# HELP go_memstats_other_sys_bytes Number of bytes used for other system allocations.
# TYPE go_memstats_other_sys_bytes gauge
go_memstats_other_sys_bytes 1.517913e+06
# HELP go_memstats_stack_inuse_bytes Number of bytes in use by the stack allocator.
# TYPE go_memstats_stack_inuse_bytes gauge
go_memstats_stack_inuse_bytes 851968
# HELP go_memstats_stack_sys_bytes Number of bytes obtained from system for stack allocator.
# TYPE go_memstats_stack_sys_bytes gauge
go_memstats_stack_sys_bytes 851968
# HELP go_memstats_sys_bytes Number of bytes obtained by system. Sum of all system allocations.
# TYPE go_memstats_sys_bytes gauge
go_memstats_sys_bytes 9.838685e+06
# HELP http_request_duration_microseconds The HTTP request latencies in microseconds.
# TYPE http_request_duration_microseconds summary
http_request_duration_microseconds{handler="prometheus",quantile="0.5"} 37701.645
http_request_duration_microseconds{handler="prometheus",quantile="0.9"} 69328.598
http_request_duration_microseconds{handler="prometheus",quantile="0.99"} 69328.598
http_request_duration_microseconds_sum{handler="prometheus"} 142579.742
http_request_duration_microseconds_count{handler="prometheus"} 3
# HELP http_request_size_bytes The HTTP request sizes in bytes.
# TYPE http_request_size_bytes summary
http_request_size_bytes{handler="prometheus",quantile="0.5"} 170
http_request_size_bytes{handler="prometheus",quantile="0.9"} 170
http_request_size_bytes{handler="prometheus",quantile="0.99"} 170
http_request_size_bytes_sum{handler="prometheus"} 510
http_request_size_bytes_count{handler="prometheus"} 3
# HELP http_requests_total Total number of HTTP requests made.
# TYPE http_requests_total counter
http_requests_total{code="200",handler="prometheus",method="get"} 3
# HELP http_response_size_bytes The HTTP response sizes in bytes.
# TYPE http_response_size_bytes summary
http_response_size_bytes{handler="prometheus",quantile="0.5"} 2332
http_response_size_bytes{handler="prometheus",quantile="0.9"} 2402
http_response_size_bytes{handler="prometheus",quantile="0.99"} 2402
http_response_size_bytes_sum{handler="prometheus"} 7024
http_response_size_bytes_count{handler="prometheus"} 3
# HELP process_cpu_seconds_total Total user and system CPU time spent in seconds.
# TYPE process_cpu_seconds_total counter
process_cpu_seconds_total 0.07
# HELP process_max_fds Maximum number of open file descriptors.
# TYPE process_max_fds gauge
process_max_fds 1.048576e+06
# HELP process_open_fds Number of open file descriptors.
# TYPE process_open_fds gauge
process_open_fds 9
# HELP process_resident_memory_bytes Resident memory size in bytes.
# TYPE process_resident_memory_bytes gauge
process_resident_memory_bytes 1.1616256e+07
# HELP process_start_time_seconds Start time of the process since unix epoch in seconds.
# TYPE process_start_time_seconds gauge
process_start_time_seconds 1.53186684346e+09
# HELP process_virtual_memory_bytes Virtual memory size in bytes.
# TYPE process_virtual_memory_bytes gauge
process_virtual_memory_bytes 2.0570112e+07
# HELP rabbitmq_channelsTotal Total number of open channels.
# TYPE rabbitmq_channelsTotal gauge
rabbitmq_channelsTotal 0
# HELP rabbitmq_connectionsTotal Total number of open connections.
# TYPE rabbitmq_connectionsTotal gauge
rabbitmq_connectionsTotal 0
# HELP rabbitmq_consumersTotal Total number of message consumers.
# TYPE rabbitmq_consumersTotal gauge
rabbitmq_consumersTotal 0
# HELP rabbitmq_exchangesTotal Total number of exchanges in use.
# TYPE rabbitmq_exchangesTotal gauge
rabbitmq_exchangesTotal 7
# HELP rabbitmq_exporter_build_info A metric with a constant '1' value labeled by version, revision, branch and build date on which the rabbitmq_exporter was built.
# TYPE rabbitmq_exporter_build_info gauge
rabbitmq_exporter_build_info{branch="heads/v0.28.0",builddate="20180425-20:02:20",revision="8cdbef10fc4c30ab1d127e85b663fa339e322071",version="0.28.0"} 1
# HELP rabbitmq_fd_total File descriptors available
# TYPE rabbitmq_fd_total gauge
rabbitmq_fd_total{node="rabbit@rabbitmq-rabbitmq-ha-0.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 1.048576e+06
rabbitmq_fd_total{node="rabbit@rabbitmq-rabbitmq-ha-1.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 1.048576e+06
rabbitmq_fd_total{node="rabbit@rabbitmq-rabbitmq-ha-2.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 1.048576e+06
# HELP rabbitmq_fd_used Used File descriptors
# TYPE rabbitmq_fd_used gauge
rabbitmq_fd_used{node="rabbit@rabbitmq-rabbitmq-ha-0.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 75
rabbitmq_fd_used{node="rabbit@rabbitmq-rabbitmq-ha-1.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 75
rabbitmq_fd_used{node="rabbit@rabbitmq-rabbitmq-ha-2.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 74
# HELP rabbitmq_node_disk_free Disk free space in bytes.
# TYPE rabbitmq_node_disk_free gauge
rabbitmq_node_disk_free{node="rabbit@rabbitmq-rabbitmq-ha-0.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 2.4876343296e+10
rabbitmq_node_disk_free{node="rabbit@rabbitmq-rabbitmq-ha-1.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 2.4876363776e+10
rabbitmq_node_disk_free{node="rabbit@rabbitmq-rabbitmq-ha-2.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 2.4876371968e+10
# HELP rabbitmq_node_disk_free_alarm Whether the disk alarm has gone off.
# TYPE rabbitmq_node_disk_free_alarm gauge
rabbitmq_node_disk_free_alarm{node="rabbit@rabbitmq-rabbitmq-ha-0.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 0
rabbitmq_node_disk_free_alarm{node="rabbit@rabbitmq-rabbitmq-ha-1.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 0
rabbitmq_node_disk_free_alarm{node="rabbit@rabbitmq-rabbitmq-ha-2.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 0
# HELP rabbitmq_node_disk_free_limit Point at which the disk alarm will go off.
# TYPE rabbitmq_node_disk_free_limit gauge
rabbitmq_node_disk_free_limit{node="rabbit@rabbitmq-rabbitmq-ha-0.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 5e+07
rabbitmq_node_disk_free_limit{node="rabbit@rabbitmq-rabbitmq-ha-1.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 5e+07
rabbitmq_node_disk_free_limit{node="rabbit@rabbitmq-rabbitmq-ha-2.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 5e+07
# HELP rabbitmq_node_mem_alarm Whether the memory alarm has gone off
# TYPE rabbitmq_node_mem_alarm gauge
rabbitmq_node_mem_alarm{node="rabbit@rabbitmq-rabbitmq-ha-0.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 0
rabbitmq_node_mem_alarm{node="rabbit@rabbitmq-rabbitmq-ha-1.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 0
rabbitmq_node_mem_alarm{node="rabbit@rabbitmq-rabbitmq-ha-2.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 0
# HELP rabbitmq_node_mem_limit Point at which the memory alarm will go off
# TYPE rabbitmq_node_mem_limit gauge
rabbitmq_node_mem_limit{node="rabbit@rabbitmq-rabbitmq-ha-0.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 2.56e+08
rabbitmq_node_mem_limit{node="rabbit@rabbitmq-rabbitmq-ha-1.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 2.56e+08
rabbitmq_node_mem_limit{node="rabbit@rabbitmq-rabbitmq-ha-2.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 2.56e+08
# HELP rabbitmq_node_mem_used Memory used in bytes
# TYPE rabbitmq_node_mem_used gauge
rabbitmq_node_mem_used{node="rabbit@rabbitmq-rabbitmq-ha-0.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 1.08335104e+08
rabbitmq_node_mem_used{node="rabbit@rabbitmq-rabbitmq-ha-1.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 1.07794432e+08
rabbitmq_node_mem_used{node="rabbit@rabbitmq-rabbitmq-ha-2.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 1.0733568e+08
# HELP rabbitmq_partitions Current Number of network partitions. 0 is ok. If the cluster is splitted the value is at least 2
# TYPE rabbitmq_partitions gauge
rabbitmq_partitions{node="rabbit@rabbitmq-rabbitmq-ha-0.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 0
rabbitmq_partitions{node="rabbit@rabbitmq-rabbitmq-ha-1.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 0
rabbitmq_partitions{node="rabbit@rabbitmq-rabbitmq-ha-2.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 0
# HELP rabbitmq_queue_messages_ready_total Total number of messages ready to be delivered to clients.
# TYPE rabbitmq_queue_messages_ready_total gauge
rabbitmq_queue_messages_ready_total 0
# HELP rabbitmq_queue_messages_total Total number ready and unacknowledged messages in cluster.
# TYPE rabbitmq_queue_messages_total gauge
rabbitmq_queue_messages_total 0
# HELP rabbitmq_queue_messages_unacknowledged_total Total number of messages delivered to clients but not yet acknowledged.
# TYPE rabbitmq_queue_messages_unacknowledged_total gauge
rabbitmq_queue_messages_unacknowledged_total 0
# HELP rabbitmq_queuesTotal Total number of queues in use.
# TYPE rabbitmq_queuesTotal gauge
rabbitmq_queuesTotal 0
# HELP rabbitmq_running number of running nodes
# TYPE rabbitmq_running gauge
rabbitmq_running{node="rabbit@rabbitmq-rabbitmq-ha-0.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 1
rabbitmq_running{node="rabbit@rabbitmq-rabbitmq-ha-1.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 1
rabbitmq_running{node="rabbit@rabbitmq-rabbitmq-ha-2.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 1
# HELP rabbitmq_sockets_total File descriptors available for use as sockets
# TYPE rabbitmq_sockets_total gauge
rabbitmq_sockets_total{node="rabbit@rabbitmq-rabbitmq-ha-0.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 943626
rabbitmq_sockets_total{node="rabbit@rabbitmq-rabbitmq-ha-1.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 943626
rabbitmq_sockets_total{node="rabbit@rabbitmq-rabbitmq-ha-2.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 943626
# HELP rabbitmq_sockets_used File descriptors used as sockets.
# TYPE rabbitmq_sockets_used gauge
rabbitmq_sockets_used{node="rabbit@rabbitmq-rabbitmq-ha-0.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 0
rabbitmq_sockets_used{node="rabbit@rabbitmq-rabbitmq-ha-1.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 0
rabbitmq_sockets_used{node="rabbit@rabbitmq-rabbitmq-ha-2.rabbitmq-rabbitmq-ha-discovery.rabbitmq.svc.cluster.local"} 0
# HELP rabbitmq_up Was the last scrape of rabbitmq successful.
# TYPE rabbitmq_up gauge
rabbitmq_up 1
```

# Finally verify that the target is in prometheus

```bash
$ curl http://prometheus./api/v1/targets
```

```json
{
        "discoveredLabels": {
          "__address__": "192.168.64.53:9090",
          "__meta_kubernetes_endpoint_port_name": "exporter",
          "__meta_kubernetes_endpoint_port_protocol": "TCP",
          "__meta_kubernetes_endpoint_ready": "true",
          "__meta_kubernetes_endpoints_name": "rabbitmq-rabbitmq-ha",
          "__meta_kubernetes_namespace": "rabbitmq",
          "__meta_kubernetes_pod_annotation_checksum_config": "83fd30c96d16e9eb4a48b7cbf43637cb0a27cbee9ec47f02b784dfee90065cae",
          "__meta_kubernetes_pod_container_name": "rabbitmq-ha-exporter",
          "__meta_kubernetes_pod_container_port_name": "exporter",
          "__meta_kubernetes_pod_container_port_number": "9090",
          "__meta_kubernetes_pod_container_port_protocol": "TCP",
          "__meta_kubernetes_pod_host_ip": "10.105.144.79",
          "__meta_kubernetes_pod_ip": "192.168.64.53",
          "__meta_kubernetes_pod_label_app": "rabbitmq-ha",
          "__meta_kubernetes_pod_label_controller_revision_hash": "rabbitmq-rabbitmq-ha-6794dc7b95",
          "__meta_kubernetes_pod_label_release": "rabbitmq",
          "__meta_kubernetes_pod_label_statefulset_kubernetes_io_pod_name": "rabbitmq-rabbitmq-ha-1",
          "__meta_kubernetes_pod_name": "rabbitmq-rabbitmq-ha-1",
          "__meta_kubernetes_pod_node_name": "int00web12.ams01.auto.moovel.ibm.com",
          "__meta_kubernetes_pod_ready": "true",
          "__meta_kubernetes_pod_uid": "89c8f609-8a11-11e8-9f0c-002590f2f522",
          "__meta_kubernetes_service_label_app": "rabbitmq-ha",
          "__meta_kubernetes_service_label_chart": "rabbitmq-ha-1.6.3",
          "__meta_kubernetes_service_label_heritage": "Tiller",
          "__meta_kubernetes_service_label_release": "rabbitmq",
          "__meta_kubernetes_service_name": "rabbitmq-rabbitmq-ha",
          "__metrics_path__": "/metrics",
          "__scheme__": "http",
          "job": "monitoring/rabbitmq-rabbitmq-ha/0"
        },
        "labels": {
          "endpoint": "exporter",
          "instance": "192.168.64.53:9090",
          "job": "rabbitmq-rabbitmq-ha",
          "namespace": "rabbitmq",
          "pod": "rabbitmq-rabbitmq-ha-1",
          "service": "rabbitmq-rabbitmq-ha"
        },
        "scrapeUrl": "http://192.168.64.53:9090/metrics",
        "lastError": "",
        "lastScrape": "2018-07-17T22:36:36.351764244Z",
        "health": "up"
      },
      {
        "discoveredLabels": {
          "__address__": "192.168.88.72:9090",
          "__meta_kubernetes_endpoint_port_name": "exporter",
          "__meta_kubernetes_endpoint_port_protocol": "TCP",
          "__meta_kubernetes_endpoint_ready": "true",
          "__meta_kubernetes_endpoints_name": "rabbitmq-rabbitmq-ha",
          "__meta_kubernetes_namespace": "rabbitmq",
          "__meta_kubernetes_pod_annotation_checksum_config": "83fd30c96d16e9eb4a48b7cbf43637cb0a27cbee9ec47f02b784dfee90065cae",
          "__meta_kubernetes_pod_container_name": "rabbitmq-ha-exporter",
          "__meta_kubernetes_pod_container_port_name": "exporter",
          "__meta_kubernetes_pod_container_port_number": "9090",
          "__meta_kubernetes_pod_container_port_protocol": "TCP",
          "__meta_kubernetes_pod_host_ip": "10.105.144.69",
          "__meta_kubernetes_pod_ip": "192.168.88.72",
          "__meta_kubernetes_pod_label_app": "rabbitmq-ha",
          "__meta_kubernetes_pod_label_controller_revision_hash": "rabbitmq-rabbitmq-ha-6794dc7b95",
          "__meta_kubernetes_pod_label_release": "rabbitmq",
          "__meta_kubernetes_pod_label_statefulset_kubernetes_io_pod_name": "rabbitmq-rabbitmq-ha-0",
          "__meta_kubernetes_pod_name": "rabbitmq-rabbitmq-ha-0",
          "__meta_kubernetes_pod_node_name": "int00web11.ams01.auto.moovel.ibm.com",
          "__meta_kubernetes_pod_ready": "true",
          "__meta_kubernetes_pod_uid": "7c8a4b31-8a11-11e8-9f0c-002590f2f522",
          "__meta_kubernetes_service_label_app": "rabbitmq-ha",
          "__meta_kubernetes_service_label_chart": "rabbitmq-ha-1.6.3",
          "__meta_kubernetes_service_label_heritage": "Tiller",
          "__meta_kubernetes_service_label_release": "rabbitmq",
          "__meta_kubernetes_service_name": "rabbitmq-rabbitmq-ha",
          "__metrics_path__": "/metrics",
          "__scheme__": "http",
          "job": "monitoring/rabbitmq-rabbitmq-ha/0"
        },
        "labels": {
          "endpoint": "exporter",
          "instance": "192.168.88.72:9090",
          "job": "rabbitmq-rabbitmq-ha",
          "namespace": "rabbitmq",
          "pod": "rabbitmq-rabbitmq-ha-0",
          "service": "rabbitmq-rabbitmq-ha"
        },
        "scrapeUrl": "http://192.168.88.72:9090/metrics",
        "lastError": "",
        "lastScrape": "2018-07-17T22:36:39.750488148Z",
        "health": "up"
      },
      {
        "discoveredLabels": {
          "__address__": "192.168.96.85:9090",
          "__meta_kubernetes_endpoint_port_name": "exporter",
          "__meta_kubernetes_endpoint_port_protocol": "TCP",
          "__meta_kubernetes_endpoint_ready": "true",
          "__meta_kubernetes_endpoints_name": "rabbitmq-rabbitmq-ha",
          "__meta_kubernetes_namespace": "rabbitmq",
          "__meta_kubernetes_pod_annotation_checksum_config": "83fd30c96d16e9eb4a48b7cbf43637cb0a27cbee9ec47f02b784dfee90065cae",
          "__meta_kubernetes_pod_container_name": "rabbitmq-ha-exporter",
          "__meta_kubernetes_pod_container_port_name": "exporter",
          "__meta_kubernetes_pod_container_port_number": "9090",
          "__meta_kubernetes_pod_container_port_protocol": "TCP",
          "__meta_kubernetes_pod_host_ip": "10.105.144.101",
          "__meta_kubernetes_pod_ip": "192.168.96.85",
          "__meta_kubernetes_pod_label_app": "rabbitmq-ha",
          "__meta_kubernetes_pod_label_controller_revision_hash": "rabbitmq-rabbitmq-ha-6794dc7b95",
          "__meta_kubernetes_pod_label_release": "rabbitmq",
          "__meta_kubernetes_pod_label_statefulset_kubernetes_io_pod_name": "rabbitmq-rabbitmq-ha-2",
          "__meta_kubernetes_pod_name": "rabbitmq-rabbitmq-ha-2",
          "__meta_kubernetes_pod_node_name": "int00web10.ams01.auto.moovel.ibm.com",
          "__meta_kubernetes_pod_ready": "true",
          "__meta_kubernetes_pod_uid": "94fbff44-8a11-11e8-9f0c-002590f2f522",
          "__meta_kubernetes_service_label_app": "rabbitmq-ha",
          "__meta_kubernetes_service_label_chart": "rabbitmq-ha-1.6.3",
          "__meta_kubernetes_service_label_heritage": "Tiller",
          "__meta_kubernetes_service_label_release": "rabbitmq",
          "__meta_kubernetes_service_name": "rabbitmq-rabbitmq-ha",
          "__metrics_path__": "/metrics",
          "__scheme__": "http",
          "job": "monitoring/rabbitmq-rabbitmq-ha/0"
        },
        "labels": {
          "endpoint": "exporter",
          "instance": "192.168.96.85:9090",
          "job": "rabbitmq-rabbitmq-ha",
          "namespace": "rabbitmq",
          "pod": "rabbitmq-rabbitmq-ha-2",
          "service": "rabbitmq-rabbitmq-ha"
        },
        "scrapeUrl": "http://192.168.96.85:9090/metrics",
        "lastError": "",
        "lastScrape": "2018-07-17T22:36:41.201780875Z",
        "health": "up"
      },
```