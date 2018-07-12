##### Check that non mongo installation is running;

```bash
localhost-3@ /tmp () () $ helm ls | grep mongo
localhost-3@ /tmp () () $
```

##### Install the current stable mongodb-replicaset chart and wait:

```bash
localhost-3@ /tmp () () $ helm install --name mongodb --namespace mongodb stable/mongodb-replicaset --debug --wait
```

##### Install Prometheus

Before upgrading we need to have prometheus running, we will use prometheus-operator
although the chart works with standard prometheus installation via annotations.

#source: https://github.com/coreos/prometheus-operator/tree/master/helm

```bash
helm repo add coreos https://s3-eu-west-1.amazonaws.com/coreos-charts/stable/
helm install coreos/prometheus-operator --name prometheus-operator --namespace monitoring
```


#### Download the new version of mongodb-replicaset with exporter

```bash
localhost-3@ /tmp () () $ git clone git@github.com:grebois/charts.git
Cloning into 'charts'...
remote: Counting objects: 34227, done.
remote: Compressing objects: 100% (10/10), done.
remote: Total 34227 (delta 8), reused 12 (delta 5), pack-reused 34212
Receiving objects: 100% (34227/34227), 8.72 MiB | 1.86 MiB/s, done.
Resolving deltas: 100% (23802/23802), done.
localhost-3@ /tmp () () $ cd charts/stable/mongodb-replicaset/
localhost-3@ /tmp/charts/stable/mongodb-replicaset (master) () $ git checkout mongodb-replicaset-exporter
Branch mongodb-replicaset-exporter set up to track remote branch mongodb-replicaset-exporter from origin.
Switched to a new branch 'mongodb-replicaset-exporter'
localhost-3@ /tmp/charts/stable/mongodb-replicaset (mongodb-replicaset-exporter) () $
```

The exporter is disable by default so we need to enable it on values.yaml

```yaml
prometheus:
  ## Configures Prometheus Exporter to expose and scrape stats.
  exporter:
    enabled: true

  ## Prometheus is using Operator.  Setting to true will create Operator specific resources like ServiceMonitors and Alerts
  operator:
    ## Are you using Prometheus Operator? [Blog Post](https://coreos.com/blog/the-prometheus-operator.html)
    enabled: true

    serviceMonitor:
      ## Interval at which Prometheus scrapes MongoDB Exporter
      interval: 10s

      # Namespace Prometheus is installed in
      namespace: monitoring

      ## Defaults to whats used if you follow CoreOS [Prometheus Install Instructions](https://github.com/coreos/prometheus-operator/tree/master/helm#tldr)
      ## [Prometheus Selector Label](https://github.com/coreos/prometheus-operator/blob/master/helm/prometheus/templates/prometheus.yaml#L65)
      ## [Kube Prometheus Selector Label](https://github.com/coreos/prometheus-operator/blob/master/helm/kube-prometheus/values.yaml#L298)
      selector:
        prometheus: kube-prometheus
```

#### Upgrade the chart

```bash
localhost-3@ /tmp/charts/stable/mongodb-replicaset (mongodb-replicaset-exporter) () $ helm upgrade mongodb . --debug --wait
```

### Verify the metrics are working

```bash
localhost-3@ /tmp/charts/stable/mongodb-replicaset (mongodb-replicaset-exporter) () $ kubectl -n mongodb port-forward mongodb-mongodb-replicaset-0 9001:9001 &
localhost-3@ /tmp/charts/stable/mongodb-replicaset (mongodb-replicaset-exporter) () $ Forwarding from 127.0.0.1:9001 -> 9001
localhost-3@ /tmp/charts/stable/mongodb-replicaset (mongodb-replicaset-exporter) () $ curl localhost:9001/metrics
Handling connection for 9001
# HELP go_gc_duration_seconds A summary of the GC invocation durations.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 0.000332618
go_gc_duration_seconds{quantile="0.25"} 0.000332618
go_gc_duration_seconds{quantile="0.5"} 0.000366548
go_gc_duration_seconds{quantile="0.75"} 0.000441249
go_gc_duration_seconds{quantile="1"} 0.000441249
go_gc_duration_seconds_sum 0.001140415
go_gc_duration_seconds_count 3
# HELP go_goroutines Number of goroutines that currently exist.
# TYPE go_goroutines gauge
go_goroutines 13
# HELP go_memstats_alloc_bytes Number of bytes allocated and still in use.
# TYPE go_memstats_alloc_bytes gauge
go_memstats_alloc_bytes 2.201464e+06
# HELP go_memstats_alloc_bytes_total Total number of bytes allocated, even if freed.
# TYPE go_memstats_alloc_bytes_total counter
go_memstats_alloc_bytes_total 9.69984e+06
# HELP go_memstats_buck_hash_sys_bytes Number of bytes used by the profiling bucket hash table.
# TYPE go_memstats_buck_hash_sys_bytes gauge
go_memstats_buck_hash_sys_bytes 1.445025e+06
# HELP go_memstats_frees_total Total number of frees.
# TYPE go_memstats_frees_total counter
go_memstats_frees_total 65422
# HELP go_memstats_gc_sys_bytes Number of bytes used for garbage collection system metadata.
# TYPE go_memstats_gc_sys_bytes gauge
go_memstats_gc_sys_bytes 536576
# HELP go_memstats_heap_alloc_bytes Number of heap bytes allocated and still in use.
# TYPE go_memstats_heap_alloc_bytes gauge
go_memstats_heap_alloc_bytes 2.201464e+06
# HELP go_memstats_heap_idle_bytes Number of heap bytes waiting to be used.
# TYPE go_memstats_heap_idle_bytes gauge
go_memstats_heap_idle_bytes 2.818048e+06
# HELP go_memstats_heap_inuse_bytes Number of heap bytes that are in use.
# TYPE go_memstats_heap_inuse_bytes gauge
go_memstats_heap_inuse_bytes 3.309568e+06
# HELP go_memstats_heap_objects Number of allocated objects.
# TYPE go_memstats_heap_objects gauge
go_memstats_heap_objects 9225
# HELP go_memstats_heap_released_bytes_total Total number of heap bytes released to OS.
# TYPE go_memstats_heap_released_bytes_total counter
go_memstats_heap_released_bytes_total 0
# HELP go_memstats_heap_sys_bytes Number of heap bytes obtained from system.
# TYPE go_memstats_heap_sys_bytes gauge
go_memstats_heap_sys_bytes 6.127616e+06
# HELP go_memstats_last_gc_time_seconds Number of seconds since 1970 of last garbage collection.
# TYPE go_memstats_last_gc_time_seconds gauge
go_memstats_last_gc_time_seconds 1.5314143072426116e+09
# HELP go_memstats_lookups_total Total number of pointer lookups.
# TYPE go_memstats_lookups_total counter
go_memstats_lookups_total 235
# HELP go_memstats_mallocs_total Total number of mallocs.
# TYPE go_memstats_mallocs_total counter
go_memstats_mallocs_total 74647
# HELP go_memstats_mcache_inuse_bytes Number of bytes in use by mcache structures.
# TYPE go_memstats_mcache_inuse_bytes gauge
go_memstats_mcache_inuse_bytes 28800
# HELP go_memstats_mcache_sys_bytes Number of bytes used for mcache structures obtained from system.
# TYPE go_memstats_mcache_sys_bytes gauge
go_memstats_mcache_sys_bytes 32768
# HELP go_memstats_mspan_inuse_bytes Number of bytes in use by mspan structures.
# TYPE go_memstats_mspan_inuse_bytes gauge
go_memstats_mspan_inuse_bytes 46080
# HELP go_memstats_mspan_sys_bytes Number of bytes used for mspan structures obtained from system.
# TYPE go_memstats_mspan_sys_bytes gauge
go_memstats_mspan_sys_bytes 49152
# HELP go_memstats_next_gc_bytes Number of heap bytes when next garbage collection will take place.
# TYPE go_memstats_next_gc_bytes gauge
go_memstats_next_gc_bytes 4.194304e+06
# HELP go_memstats_other_sys_bytes Number of bytes used for other system allocations.
# TYPE go_memstats_other_sys_bytes gauge
go_memstats_other_sys_bytes 3.841111e+06
# HELP go_memstats_stack_inuse_bytes Number of bytes in use by the stack allocator.
# TYPE go_memstats_stack_inuse_bytes gauge
go_memstats_stack_inuse_bytes 2.260992e+06
# HELP go_memstats_stack_sys_bytes Number of bytes obtained from system for stack allocator.
# TYPE go_memstats_stack_sys_bytes gauge
go_memstats_stack_sys_bytes 2.260992e+06
# HELP go_memstats_sys_bytes Number of bytes obtained by system. Sum of all system allocations.
# TYPE go_memstats_sys_bytes gauge
go_memstats_sys_bytes 1.429324e+07
# HELP http_request_duration_microseconds The HTTP request latencies in microseconds.
# TYPE http_request_duration_microseconds summary
http_request_duration_microseconds{handler="prometheus",quantile="0.5"} 14005.013
http_request_duration_microseconds{handler="prometheus",quantile="0.9"} 15878.442
http_request_duration_microseconds{handler="prometheus",quantile="0.99"} 15878.442
http_request_duration_microseconds_sum{handler="prometheus"} 99666.197
http_request_duration_microseconds_count{handler="prometheus"} 7
# HELP http_request_size_bytes The HTTP request sizes in bytes.
# TYPE http_request_size_bytes summary
http_request_size_bytes{handler="prometheus",quantile="0.5"} 170
http_request_size_bytes{handler="prometheus",quantile="0.9"} 170
http_request_size_bytes{handler="prometheus",quantile="0.99"} 170
http_request_size_bytes_sum{handler="prometheus"} 1190
http_request_size_bytes_count{handler="prometheus"} 7
# HELP http_requests_total Total number of HTTP requests made.
# TYPE http_requests_total counter
http_requests_total{code="200",handler="prometheus",method="get"} 7
# HELP http_response_size_bytes The HTTP response sizes in bytes.
# TYPE http_response_size_bytes summary
http_response_size_bytes{handler="prometheus",quantile="0.5"} 5883
http_response_size_bytes{handler="prometheus",quantile="0.9"} 5911
http_response_size_bytes{handler="prometheus",quantile="0.99"} 5911
http_response_size_bytes_sum{handler="prometheus"} 40999
http_response_size_bytes_count{handler="prometheus"} 7
# HELP mongodb_asserts_total The asserts document reports the number of asserts on the database. While assert errors are typically uncommon, if there are non-zero values for the asserts, you should check the log file for the mongod process for more information. In many cases these errors are trivial, but are worth investigating.
# TYPE mongodb_asserts_total counter
mongodb_asserts_total{type="msg"} 0
mongodb_asserts_total{type="regular"} 0
mongodb_asserts_total{type="rollovers"} 0
mongodb_asserts_total{type="user"} 0
mongodb_asserts_total{type="warning"} 0
# HELP mongodb_connections The connections sub document data regarding the current status of incoming connections and availability of the database server. Use these values to assess the current load and capacity requirements of the server
# TYPE mongodb_connections gauge
mongodb_connections{state="available"} 838857
mongodb_connections{state="current"} 3
# HELP mongodb_connections_metrics_created_total totalCreated provides a count of all incoming connections created to the server. This number includes connections that have since closed
# TYPE mongodb_connections_metrics_created_total counter
mongodb_connections_metrics_created_total 21
# HELP mongodb_extra_info_heap_usage_bytes The heap_usage_bytes field is only available on Unix/Linux systems, and reports the total size in bytes of heap space used by the database process
# TYPE mongodb_extra_info_heap_usage_bytes gauge
mongodb_extra_info_heap_usage_bytes 0
# HELP mongodb_extra_info_page_faults_total The page_faults Reports the total number of page faults that require disk operations. Page faults refer to operations that require the database server to access data which isn't available in active memory. The page_faults counter may increase dramatically during moments of poor performance and may correlate with limited memory environments and larger data sets. Limited and sporadic page faults do not necessarily indicate an issue
# TYPE mongodb_extra_info_page_faults_total gauge
mongodb_extra_info_page_faults_total 0
# HELP mongodb_global_lock_client The activeClients data structure provides more granular information about the number of connected clients and the operation types (e.g. read or write) performed by these clients
# TYPE mongodb_global_lock_client gauge
mongodb_global_lock_client{type="reader"} 0
mongodb_global_lock_client{type="writer"} 0
# HELP mongodb_global_lock_current_queue The currentQueue data structure value provides more granular information concerning the number of operations queued because of a lock
# TYPE mongodb_global_lock_current_queue gauge
mongodb_global_lock_current_queue{type="reader"} 0
mongodb_global_lock_current_queue{type="writer"} 0
# HELP mongodb_global_lock_ratio The value of ratio displays the relationship between lockTime and totalTime. Low values indicate that operations have held the globalLock frequently for shorter periods of time. High values indicate that operations have held globalLock infrequently for longer periods of time
# TYPE mongodb_global_lock_ratio gauge
mongodb_global_lock_ratio 0
# HELP mongodb_global_lock_total The value of totalTime represents the time, in microseconds, since the database last started and creation of the globalLock. This is roughly equivalent to total server uptime
# TYPE mongodb_global_lock_total counter
mongodb_global_lock_total 0
# HELP mongodb_instance_local_time The localTime value is the current time, according to the server, in UTC specified in an ISODate format.
# TYPE mongodb_instance_local_time counter
mongodb_instance_local_time 1.531414311e+09
# HELP mongodb_instance_uptime_estimate_seconds uptimeEstimate provides the uptime as calculated from MongoDB's internal course-grained time keeping system.
# TYPE mongodb_instance_uptime_estimate_seconds counter
mongodb_instance_uptime_estimate_seconds 69
# HELP mongodb_instance_uptime_seconds The value of the uptime field corresponds to the number of seconds that the mongos or mongod process has been active.
# TYPE mongodb_instance_uptime_seconds counter
mongodb_instance_uptime_seconds 69
# HELP mongodb_locks_time_acquiring_global_microseconds_total amount of time in microseconds that any database has spent waiting for the global lock
# TYPE mongodb_locks_time_acquiring_global_microseconds_total counter
mongodb_locks_time_acquiring_global_microseconds_total{database="Collection",type="read"} 0
mongodb_locks_time_acquiring_global_microseconds_total{database="Collection",type="write"} 0
mongodb_locks_time_acquiring_global_microseconds_total{database="Database",type="read"} 0
mongodb_locks_time_acquiring_global_microseconds_total{database="Database",type="write"} 0
mongodb_locks_time_acquiring_global_microseconds_total{database="Global",type="read"} 0
mongodb_locks_time_acquiring_global_microseconds_total{database="Global",type="write"} 0
mongodb_locks_time_acquiring_global_microseconds_total{database="oplog",type="read"} 0
mongodb_locks_time_acquiring_global_microseconds_total{database="oplog",type="write"} 0
# HELP mongodb_locks_time_locked_global_microseconds_total amount of time in microseconds that any database has held the global lock
# TYPE mongodb_locks_time_locked_global_microseconds_total counter
mongodb_locks_time_locked_global_microseconds_total{database="Collection",type="read"} 0
mongodb_locks_time_locked_global_microseconds_total{database="Collection",type="write"} 0
mongodb_locks_time_locked_global_microseconds_total{database="Database",type="read"} 0
mongodb_locks_time_locked_global_microseconds_total{database="Database",type="write"} 0
mongodb_locks_time_locked_global_microseconds_total{database="Global",type="read"} 0
mongodb_locks_time_locked_global_microseconds_total{database="Global",type="write"} 0
mongodb_locks_time_locked_global_microseconds_total{database="oplog",type="read"} 0
mongodb_locks_time_locked_global_microseconds_total{database="oplog",type="write"} 0
# HELP mongodb_locks_time_locked_local_microseconds_total amount of time in microseconds that any database has held the local lock
# TYPE mongodb_locks_time_locked_local_microseconds_total counter
mongodb_locks_time_locked_local_microseconds_total{database="Collection",type="read"} 0
mongodb_locks_time_locked_local_microseconds_total{database="Collection",type="write"} 0
mongodb_locks_time_locked_local_microseconds_total{database="Database",type="read"} 0
mongodb_locks_time_locked_local_microseconds_total{database="Database",type="write"} 0
mongodb_locks_time_locked_local_microseconds_total{database="Global",type="read"} 0
mongodb_locks_time_locked_local_microseconds_total{database="Global",type="write"} 0
mongodb_locks_time_locked_local_microseconds_total{database="oplog",type="read"} 0
mongodb_locks_time_locked_local_microseconds_total{database="oplog",type="write"} 0
# HELP mongodb_memory The mem data structure holds information regarding the target system architecture of mongod and current memory use
# TYPE mongodb_memory gauge
mongodb_memory{type="mapped"} 0
mongodb_memory{type="mapped_with_journal"} 0
mongodb_memory{type="resident"} 81
mongodb_memory{type="virtual"} 1432
# HELP mongodb_metrics_cursor_open The open is an embedded document that contains data regarding open cursors
# TYPE mongodb_metrics_cursor_open gauge
mongodb_metrics_cursor_open{state="pinned"} 0
mongodb_metrics_cursor_open{state="timed_out"} 0
mongodb_metrics_cursor_open{state="total"} 0
# HELP mongodb_metrics_cursor_timed_out_total timedOut provides the total number of cursors that have timed out since the server process started. If this number is large or growing at a regular rate, this may indicate an application error
# TYPE mongodb_metrics_cursor_timed_out_total counter
mongodb_metrics_cursor_timed_out_total 0
# HELP mongodb_metrics_document_total The document holds a document of that reflect document access and modification patterns and data use. Compare these values to the data in the opcounters document, which track total number of operations
# TYPE mongodb_metrics_document_total counter
mongodb_metrics_document_total{state="deleted"} 0
mongodb_metrics_document_total{state="inserted"} 0
mongodb_metrics_document_total{state="returned"} 16
mongodb_metrics_document_total{state="updated"} 0
# HELP mongodb_metrics_get_last_error_wtime_num_total num reports the total number of getLastError operations with a specified write concern (i.e. w) that wait for one or more members of a replica set to acknowledge the write operation (i.e. a w value greater than 1.)
# TYPE mongodb_metrics_get_last_error_wtime_num_total gauge
mongodb_metrics_get_last_error_wtime_num_total 0
# HELP mongodb_metrics_get_last_error_wtime_total_milliseconds total_millis reports the total amount of time in milliseconds that the mongod has spent performing getLastError operations with write concern (i.e. w) that wait for one or more members of a replica set to acknowledge the write operation (i.e. a w value greater than 1.)
# TYPE mongodb_metrics_get_last_error_wtime_total_milliseconds counter
mongodb_metrics_get_last_error_wtime_total_milliseconds 0
# HELP mongodb_metrics_get_last_error_wtimeouts_total wtimeouts reports the number of times that write concern operations have timed out as a result of the wtimeout threshold to getLastError.
# TYPE mongodb_metrics_get_last_error_wtimeouts_total counter
mongodb_metrics_get_last_error_wtimeouts_total 0
# HELP mongodb_metrics_operation_total operation is a sub-document that holds counters for several types of update and query operations that MongoDB handles using special operation types
# TYPE mongodb_metrics_operation_total counter
mongodb_metrics_operation_total{type="fastmod"} 0
mongodb_metrics_operation_total{type="idhack"} 0
mongodb_metrics_operation_total{type="scan_and_order"} 1
# HELP mongodb_metrics_query_executor_total queryExecutor is a document that reports data from the query execution system
# TYPE mongodb_metrics_query_executor_total counter
mongodb_metrics_query_executor_total{state="scanned"} 0
mongodb_metrics_query_executor_total{state="scanned_objects"} 16
# HELP mongodb_metrics_record_moves_total moves reports the total number of times documents move within the on-disk representation of the MongoDB data set. Documents move as a result of operations that increase the size of the document beyond their allocated record size
# TYPE mongodb_metrics_record_moves_total counter
mongodb_metrics_record_moves_total 0
# HELP mongodb_metrics_repl_apply_batches_num_total num reports the total number of batches applied across all databases
# TYPE mongodb_metrics_repl_apply_batches_num_total counter
mongodb_metrics_repl_apply_batches_num_total 8
# HELP mongodb_metrics_repl_apply_batches_total_milliseconds total_millis reports the total amount of time the mongod has spent applying operations from the oplog
# TYPE mongodb_metrics_repl_apply_batches_total_milliseconds counter
mongodb_metrics_repl_apply_batches_total_milliseconds 2
# HELP mongodb_metrics_repl_apply_ops_total ops reports the total number of oplog operations applied
# TYPE mongodb_metrics_repl_apply_ops_total counter
mongodb_metrics_repl_apply_ops_total 0
# HELP mongodb_metrics_repl_buffer_count count reports the current number of operations in the oplog buffer
# TYPE mongodb_metrics_repl_buffer_count gauge
mongodb_metrics_repl_buffer_count 0
# HELP mongodb_metrics_repl_buffer_max_size_bytes maxSizeBytes reports the maximum size of the buffer. This value is a constant setting in the mongod, and is not configurable
# TYPE mongodb_metrics_repl_buffer_max_size_bytes counter
mongodb_metrics_repl_buffer_max_size_bytes 2.68435456e+08
# HELP mongodb_metrics_repl_buffer_size_bytes sizeBytes reports the current size of the contents of the oplog buffer
# TYPE mongodb_metrics_repl_buffer_size_bytes gauge
mongodb_metrics_repl_buffer_size_bytes 0
# HELP mongodb_metrics_repl_network_bytes_total bytes reports the total amount of data read from the replication sync source
# TYPE mongodb_metrics_repl_network_bytes_total counter
mongodb_metrics_repl_network_bytes_total 990
# HELP mongodb_metrics_repl_network_getmores_num_total num reports the total number of getmore operations, which are operations that request an additional set of operations from the replication sync source.
# TYPE mongodb_metrics_repl_network_getmores_num_total counter
mongodb_metrics_repl_network_getmores_num_total 22
# HELP mongodb_metrics_repl_network_getmores_total_milliseconds total_millis reports the total amount of time required to collect data from getmore operations
# TYPE mongodb_metrics_repl_network_getmores_total_milliseconds counter
mongodb_metrics_repl_network_getmores_total_milliseconds 66047
# HELP mongodb_metrics_repl_network_ops_total ops reports the total number of operations read from the replication source.
# TYPE mongodb_metrics_repl_network_ops_total counter
mongodb_metrics_repl_network_ops_total 9
# HELP mongodb_metrics_repl_network_readers_created_total readersCreated reports the total number of oplog query processes created. MongoDB will create a new oplog query any time an error occurs in the connection, including a timeout, or a network operation. Furthermore, readersCreated will increment every time MongoDB selects a new source fore replication.
# TYPE mongodb_metrics_repl_network_readers_created_total counter
mongodb_metrics_repl_network_readers_created_total 1
# HELP mongodb_metrics_repl_oplog_insert_bytes_total insertBytes the total size of documents inserted into the oplog.
# TYPE mongodb_metrics_repl_oplog_insert_bytes_total counter
mongodb_metrics_repl_oplog_insert_bytes_total 0
# HELP mongodb_metrics_repl_oplog_insert_num_total num reports the total number of items inserted into the oplog.
# TYPE mongodb_metrics_repl_oplog_insert_num_total counter
mongodb_metrics_repl_oplog_insert_num_total 0
# HELP mongodb_metrics_repl_oplog_insert_total_milliseconds total_millis reports the total amount of time spent for the mongod to insert data into the oplog.
# TYPE mongodb_metrics_repl_oplog_insert_total_milliseconds counter
mongodb_metrics_repl_oplog_insert_total_milliseconds 0
# HELP mongodb_metrics_repl_preload_docs_num_total num reports the total number of documents loaded during the pre-fetch stage of replication
# TYPE mongodb_metrics_repl_preload_docs_num_total counter
mongodb_metrics_repl_preload_docs_num_total 0
# HELP mongodb_metrics_repl_preload_docs_total_milliseconds total_millis reports the total amount of time spent loading documents as part of the pre-fetch stage of replication
# TYPE mongodb_metrics_repl_preload_docs_total_milliseconds counter
mongodb_metrics_repl_preload_docs_total_milliseconds 0
# HELP mongodb_metrics_repl_preload_indexes_num_total num reports the total number of index entries loaded by members before updating documents as part of the pre-fetch stage of replication
# TYPE mongodb_metrics_repl_preload_indexes_num_total counter
mongodb_metrics_repl_preload_indexes_num_total 0
# HELP mongodb_metrics_repl_preload_indexes_total_milliseconds total_millis reports the total amount of time spent loading index entries as part of the pre-fetch stage of replication
# TYPE mongodb_metrics_repl_preload_indexes_total_milliseconds counter
mongodb_metrics_repl_preload_indexes_total_milliseconds 0
# HELP mongodb_metrics_storage_freelist_search_total metrics about searching records in the database.
# TYPE mongodb_metrics_storage_freelist_search_total counter
mongodb_metrics_storage_freelist_search_total{type="bucket_exhausted"} 0
mongodb_metrics_storage_freelist_search_total{type="requests"} 0
mongodb_metrics_storage_freelist_search_total{type="scanned"} 0
# HELP mongodb_metrics_ttl_deleted_documents_total deletedDocuments reports the total number of documents deleted from collections with a ttl index.
# TYPE mongodb_metrics_ttl_deleted_documents_total counter
mongodb_metrics_ttl_deleted_documents_total 0
# HELP mongodb_metrics_ttl_passes_total passes reports the number of times the background process removes documents from collections with a ttl index
# TYPE mongodb_metrics_ttl_passes_total counter
mongodb_metrics_ttl_passes_total 0
# HELP mongodb_network_bytes_total The network data structure contains data regarding MongoDB's network use
# TYPE mongodb_network_bytes_total counter
mongodb_network_bytes_total{state="in_bytes"} 34757
mongodb_network_bytes_total{state="out_bytes"} 369438
# HELP mongodb_network_metrics_num_requests_total The numRequests field is a counter of the total number of distinct requests that the server has received. Use this value to provide context for the bytesIn and bytesOut values to ensure that MongoDB's network utilization is consistent with expectations and application use
# TYPE mongodb_network_metrics_num_requests_total counter
mongodb_network_metrics_num_requests_total 174
# HELP mongodb_op_counters_repl_total The opcountersRepl data structure, similar to the opcounters data structure, provides an overview of database replication operations by type and makes it possible to analyze the load on the replica in more granular manner. These values only appear when the current host has replication enabled
# TYPE mongodb_op_counters_repl_total counter
mongodb_op_counters_repl_total{type="command"} 0
mongodb_op_counters_repl_total{type="delete"} 0
mongodb_op_counters_repl_total{type="getmore"} 0
mongodb_op_counters_repl_total{type="insert"} 0
mongodb_op_counters_repl_total{type="query"} 0
mongodb_op_counters_repl_total{type="update"} 0
# HELP mongodb_op_counters_total The opcounters data structure provides an overview of database operations by type and makes it possible to analyze the load on the database in more granular manner. These numbers will grow over time and in response to database use. Analyze these values over time to track database utilization
# TYPE mongodb_op_counters_total counter
mongodb_op_counters_total{type="command"} 160
mongodb_op_counters_total{type="delete"} 0
mongodb_op_counters_total{type="getmore"} 0
mongodb_op_counters_total{type="insert"} 0
mongodb_op_counters_total{type="query"} 16
mongodb_op_counters_total{type="update"} 0
# HELP mongodb_replset_heatbeat_interval_millis The frequency in milliseconds of the heartbeats
# TYPE mongodb_replset_heatbeat_interval_millis gauge
mongodb_replset_heatbeat_interval_millis{set="rs0"} 2000
# HELP mongodb_replset_member_config_version The configVersion value is the replica set configuration version.
# TYPE mongodb_replset_member_config_version gauge
mongodb_replset_member_config_version{name="mongodb-mongodb-replicaset-0.mongodb-mongodb-replicaset.mongodb.svc.cluster.local:27017",set="rs0",state="SECONDARY"} 3
mongodb_replset_member_config_version{name="mongodb-mongodb-replicaset-1.mongodb-mongodb-replicaset.mongodb.svc.cluster.local:27017",set="rs0",state="SECONDARY"} 3
mongodb_replset_member_config_version{name="mongodb-mongodb-replicaset-2.mongodb-mongodb-replicaset.mongodb.svc.cluster.local:27017",set="rs0",state="PRIMARY"} 3
# HELP mongodb_replset_member_election_date The timestamp the node was elected as replica leader
# TYPE mongodb_replset_member_election_date gauge
mongodb_replset_member_election_date{name="mongodb-mongodb-replicaset-2.mongodb-mongodb-replicaset.mongodb.svc.cluster.local:27017",set="rs0",state="PRIMARY"} 1.531414199e+09
# HELP mongodb_replset_member_health This field conveys if the member is up (1) or down (0).
# TYPE mongodb_replset_member_health gauge
mongodb_replset_member_health{name="mongodb-mongodb-replicaset-0.mongodb-mongodb-replicaset.mongodb.svc.cluster.local:27017",set="rs0",state="SECONDARY"} 1
mongodb_replset_member_health{name="mongodb-mongodb-replicaset-1.mongodb-mongodb-replicaset.mongodb.svc.cluster.local:27017",set="rs0",state="SECONDARY"} 1
mongodb_replset_member_health{name="mongodb-mongodb-replicaset-2.mongodb-mongodb-replicaset.mongodb.svc.cluster.local:27017",set="rs0",state="PRIMARY"} 1
# HELP mongodb_replset_member_last_heartbeat The lastHeartbeat value provides an ISODate formatted date and time of the transmission time of last heartbeat received from this member
# TYPE mongodb_replset_member_last_heartbeat gauge
mongodb_replset_member_last_heartbeat{name="mongodb-mongodb-replicaset-1.mongodb-mongodb-replicaset.mongodb.svc.cluster.local:27017",set="rs0",state="SECONDARY"} 1.531414311e+09
mongodb_replset_member_last_heartbeat{name="mongodb-mongodb-replicaset-2.mongodb-mongodb-replicaset.mongodb.svc.cluster.local:27017",set="rs0",state="PRIMARY"} 1.531414309e+09
# HELP mongodb_replset_member_last_heartbeat_recv The lastHeartbeatRecv value provides an ISODate formatted date and time that the last heartbeat was received from this member
# TYPE mongodb_replset_member_last_heartbeat_recv gauge
mongodb_replset_member_last_heartbeat_recv{name="mongodb-mongodb-replicaset-1.mongodb-mongodb-replicaset.mongodb.svc.cluster.local:27017",set="rs0",state="SECONDARY"} 1.531414311e+09
mongodb_replset_member_last_heartbeat_recv{name="mongodb-mongodb-replicaset-2.mongodb-mongodb-replicaset.mongodb.svc.cluster.local:27017",set="rs0",state="PRIMARY"} 1.53141431e+09
# HELP mongodb_replset_member_optime_date The last entry from the oplog that this member applied.
# TYPE mongodb_replset_member_optime_date gauge
mongodb_replset_member_optime_date{name="mongodb-mongodb-replicaset-0.mongodb-mongodb-replicaset.mongodb.svc.cluster.local:27017",set="rs0",state="SECONDARY"} 1.531414311e+09
mongodb_replset_member_optime_date{name="mongodb-mongodb-replicaset-1.mongodb-mongodb-replicaset.mongodb.svc.cluster.local:27017",set="rs0",state="SECONDARY"} 1.531414311e+09
mongodb_replset_member_optime_date{name="mongodb-mongodb-replicaset-2.mongodb-mongodb-replicaset.mongodb.svc.cluster.local:27017",set="rs0",state="PRIMARY"} 1.531414301e+09
# HELP mongodb_replset_member_ping_ms The pingMs represents the number of milliseconds (ms) that a round-trip packet takes to travel between the remote member and the local instance.
# TYPE mongodb_replset_member_ping_ms gauge
mongodb_replset_member_ping_ms{name="mongodb-mongodb-replicaset-1.mongodb-mongodb-replicaset.mongodb.svc.cluster.local:27017",set="rs0",state="SECONDARY"} 0
mongodb_replset_member_ping_ms{name="mongodb-mongodb-replicaset-2.mongodb-mongodb-replicaset.mongodb.svc.cluster.local:27017",set="rs0",state="PRIMARY"} 1
# HELP mongodb_replset_member_state The value of state is an integer between 0 and 10 that represents the replica state of the member.
# TYPE mongodb_replset_member_state gauge
mongodb_replset_member_state{name="mongodb-mongodb-replicaset-0.mongodb-mongodb-replicaset.mongodb.svc.cluster.local:27017",set="rs0",state="SECONDARY"} 2
mongodb_replset_member_state{name="mongodb-mongodb-replicaset-1.mongodb-mongodb-replicaset.mongodb.svc.cluster.local:27017",set="rs0",state="SECONDARY"} 2
mongodb_replset_member_state{name="mongodb-mongodb-replicaset-2.mongodb-mongodb-replicaset.mongodb.svc.cluster.local:27017",set="rs0",state="PRIMARY"} 1
# HELP mongodb_replset_member_uptime The uptime field holds a value that reflects the number of seconds that this member has been online.
# TYPE mongodb_replset_member_uptime counter
mongodb_replset_member_uptime{name="mongodb-mongodb-replicaset-0.mongodb-mongodb-replicaset.mongodb.svc.cluster.local:27017",set="rs0",state="SECONDARY"} 69
mongodb_replset_member_uptime{name="mongodb-mongodb-replicaset-1.mongodb-mongodb-replicaset.mongodb.svc.cluster.local:27017",set="rs0",state="SECONDARY"} 67
mongodb_replset_member_uptime{name="mongodb-mongodb-replicaset-2.mongodb-mongodb-replicaset.mongodb.svc.cluster.local:27017",set="rs0",state="PRIMARY"} 67
# HELP mongodb_replset_my_state An integer between 0 and 10 that represents the replica state of the current member
# TYPE mongodb_replset_my_state gauge
mongodb_replset_my_state{set="rs0"} 2
# HELP mongodb_replset_number_of_members The number of replica set mebers
# TYPE mongodb_replset_number_of_members gauge
mongodb_replset_number_of_members{set="rs0"} 3
# HELP mongodb_replset_oplog_head_timestamp The timestamp of the newest change in the oplog
# TYPE mongodb_replset_oplog_head_timestamp gauge
mongodb_replset_oplog_head_timestamp 1.531411139e+09
# HELP mongodb_replset_oplog_items_total The total number of changes in the oplog
# TYPE mongodb_replset_oplog_items_total gauge
mongodb_replset_oplog_items_total 323
# HELP mongodb_replset_oplog_size_bytes Size of oplog in bytes
# TYPE mongodb_replset_oplog_size_bytes gauge
mongodb_replset_oplog_size_bytes{type="current"} 36226
mongodb_replset_oplog_size_bytes{type="storage"} 53248
# HELP mongodb_replset_oplog_tail_timestamp The timestamp of the oldest change in the oplog
# TYPE mongodb_replset_oplog_tail_timestamp gauge
mongodb_replset_oplog_tail_timestamp 1.531414311e+09
# HELP mongodb_replset_term The election count for the replica set, as known to this replica set member
# TYPE mongodb_replset_term gauge
mongodb_replset_term{set="rs0"} 3
# HELP process_cpu_seconds_total Total user and system CPU time spent in seconds.
# TYPE process_cpu_seconds_total counter
process_cpu_seconds_total 0.09
# HELP process_max_fds Maximum number of open file descriptors.
# TYPE process_max_fds gauge
process_max_fds 1.048576e+06
# HELP process_open_fds Number of open file descriptors.
# TYPE process_open_fds gauge
process_open_fds 11
# HELP process_resident_memory_bytes Resident memory size in bytes.
# TYPE process_resident_memory_bytes gauge
process_resident_memory_bytes 1.1112448e+07
# HELP process_start_time_seconds Start time of the process since unix epoch in seconds.
# TYPE process_start_time_seconds gauge
process_start_time_seconds 1.5314142465e+09
# HELP process_virtual_memory_bytes Virtual memory size in bytes.
# TYPE process_virtual_memory_bytes gauge
process_virtual_memory_bytes 2.0127744e+07
localhost-3@ /tmp/charts/stable/mongodb-replicaset (mongodb-replicaset-exporter) () 
```

#### Finally verify that the target is in prometheus

```bash
localhost-3@ /tmp/charts/stable/mongodb-replicaset (mongodb-replicaset-exporter) () $ curl http://prometheus./api/v1/targets
```

```json
{
        "discoveredLabels": {
          "__address__": "192.168.64.55:9001",
          "__meta_kubernetes_endpoint_port_name": "metrics",
          "__meta_kubernetes_endpoint_port_protocol": "TCP",
          "__meta_kubernetes_endpoint_ready": "true",
          "__meta_kubernetes_endpoints_name": "mongodb-mongodb-replicaset",
          "__meta_kubernetes_namespace": "mongodb",
          "__meta_kubernetes_pod_container_name": "mongodb-exporter",
          "__meta_kubernetes_pod_container_port_name": "metrics",
          "__meta_kubernetes_pod_container_port_number": "9001",
          "__meta_kubernetes_pod_container_port_protocol": "TCP",
          "__meta_kubernetes_pod_host_ip": "10.105.144.79",
          "__meta_kubernetes_pod_ip": "192.168.64.55",
          "__meta_kubernetes_pod_label_app": "mongodb-replicaset",
          "__meta_kubernetes_pod_label_controller_revision_hash": "mongodb-mongodb-replicaset-68797754db",
          "__meta_kubernetes_pod_label_release": "mongodb",
          "__meta_kubernetes_pod_label_statefulset_kubernetes_io_pod_name": "mongodb-mongodb-replicaset-1",
          "__meta_kubernetes_pod_name": "mongodb-mongodb-replicaset-1",
          "__meta_kubernetes_pod_node_name": "int00web12.ams01.auto.moovel.ibm.com",
          "__meta_kubernetes_pod_ready": "true",
          "__meta_kubernetes_pod_uid": "8768b756-85f3-11e8-9f0c-002590f2f522",
          "__meta_kubernetes_service_annotation_service_alpha_kubernetes_io_tolerate_unready_endpoints": "true",
          "__meta_kubernetes_service_label_app": "mongodb-replicaset",
          "__meta_kubernetes_service_label_chart": "mongodb-replicaset-3.5.1",
          "__meta_kubernetes_service_label_heritage": "Tiller",
          "__meta_kubernetes_service_label_release": "mongodb",
          "__meta_kubernetes_service_name": "mongodb-mongodb-replicaset",
          "__metrics_path__": "/metrics",
          "__scheme__": "http",
          "job": "monitoring/mongodb-mongodb-replicaset/0"
        },
        "labels": {
          "endpoint": "metrics",
          "instance": "192.168.64.55:9001",
          "job": "mongodb-mongodb-replicaset",
          "namespace": "mongodb",
          "pod": "mongodb-mongodb-replicaset-1",
          "service": "mongodb-mongodb-replicaset"
        },
        "scrapeUrl": "http://192.168.64.55:9001/metrics",
        "lastError": "",
        "lastScrape": "2018-07-12T16:54:27.656763815Z",
        "health": "up"
      },
      {
        "discoveredLabels": {
          "__address__": "192.168.88.73:9001",
          "__meta_kubernetes_endpoint_port_name": "metrics",
          "__meta_kubernetes_endpoint_port_protocol": "TCP",
          "__meta_kubernetes_endpoint_ready": "true",
          "__meta_kubernetes_endpoints_name": "mongodb-mongodb-replicaset",
          "__meta_kubernetes_namespace": "mongodb",
          "__meta_kubernetes_pod_container_name": "mongodb-exporter",
          "__meta_kubernetes_pod_container_port_name": "metrics",
          "__meta_kubernetes_pod_container_port_number": "9001",
          "__meta_kubernetes_pod_container_port_protocol": "TCP",
          "__meta_kubernetes_pod_host_ip": "10.105.144.69",
          "__meta_kubernetes_pod_ip": "192.168.88.73",
          "__meta_kubernetes_pod_label_app": "mongodb-replicaset",
          "__meta_kubernetes_pod_label_controller_revision_hash": "mongodb-mongodb-replicaset-68797754db",
          "__meta_kubernetes_pod_label_release": "mongodb",
          "__meta_kubernetes_pod_label_statefulset_kubernetes_io_pod_name": "mongodb-mongodb-replicaset-2",
          "__meta_kubernetes_pod_name": "mongodb-mongodb-replicaset-2",
          "__meta_kubernetes_pod_node_name": "int00web11.ams01.auto.moovel.ibm.com",
          "__meta_kubernetes_pod_ready": "true",
          "__meta_kubernetes_pod_uid": "57f9431d-85f3-11e8-9f0c-002590f2f522",
          "__meta_kubernetes_service_annotation_service_alpha_kubernetes_io_tolerate_unready_endpoints": "true",
          "__meta_kubernetes_service_label_app": "mongodb-replicaset",
          "__meta_kubernetes_service_label_chart": "mongodb-replicaset-3.5.1",
          "__meta_kubernetes_service_label_heritage": "Tiller",
          "__meta_kubernetes_service_label_release": "mongodb",
          "__meta_kubernetes_service_name": "mongodb-mongodb-replicaset",
          "__metrics_path__": "/metrics",
          "__scheme__": "http",
          "job": "monitoring/mongodb-mongodb-replicaset/0"
        },
        "labels": {
          "endpoint": "metrics",
          "instance": "192.168.88.73:9001",
          "job": "mongodb-mongodb-replicaset",
          "namespace": "mongodb",
          "pod": "mongodb-mongodb-replicaset-2",
          "service": "mongodb-mongodb-replicaset"
        },
        "scrapeUrl": "http://192.168.88.73:9001/metrics",
        "lastError": "",
        "lastScrape": "2018-07-12T16:54:24.99104592Z",
        "health": "up"
      },
      {
        "discoveredLabels": {
          "__address__": "192.168.96.71:9001",
          "__meta_kubernetes_endpoint_port_name": "metrics",
          "__meta_kubernetes_endpoint_port_protocol": "TCP",
          "__meta_kubernetes_endpoint_ready": "true",
          "__meta_kubernetes_endpoints_name": "mongodb-mongodb-replicaset",
          "__meta_kubernetes_namespace": "mongodb",
          "__meta_kubernetes_pod_container_name": "mongodb-exporter",
          "__meta_kubernetes_pod_container_port_name": "metrics",
          "__meta_kubernetes_pod_container_port_number": "9001",
          "__meta_kubernetes_pod_container_port_protocol": "TCP",
          "__meta_kubernetes_pod_host_ip": "10.105.144.101",
          "__meta_kubernetes_pod_ip": "192.168.96.71",
          "__meta_kubernetes_pod_label_app": "mongodb-replicaset",
          "__meta_kubernetes_pod_label_controller_revision_hash": "mongodb-mongodb-replicaset-68797754db",
          "__meta_kubernetes_pod_label_release": "mongodb",
          "__meta_kubernetes_pod_label_statefulset_kubernetes_io_pod_name": "mongodb-mongodb-replicaset-0",
          "__meta_kubernetes_pod_name": "mongodb-mongodb-replicaset-0",
          "__meta_kubernetes_pod_node_name": "int00web10.ams01.auto.moovel.ibm.com",
          "__meta_kubernetes_pod_ready": "true",
          "__meta_kubernetes_pod_uid": "aee15253-85f3-11e8-9f0c-002590f2f522",
          "__meta_kubernetes_service_annotation_service_alpha_kubernetes_io_tolerate_unready_endpoints": "true",
          "__meta_kubernetes_service_label_app": "mongodb-replicaset",
          "__meta_kubernetes_service_label_chart": "mongodb-replicaset-3.5.1",
          "__meta_kubernetes_service_label_heritage": "Tiller",
          "__meta_kubernetes_service_label_release": "mongodb",
          "__meta_kubernetes_service_name": "mongodb-mongodb-replicaset",
          "__metrics_path__": "/metrics",
          "__scheme__": "http",
          "job": "monitoring/mongodb-mongodb-replicaset/0"
        },
        "labels": {
          "endpoint": "metrics",
          "instance": "192.168.96.71:9001",
          "job": "mongodb-mongodb-replicaset",
          "namespace": "mongodb",
          "pod": "mongodb-mongodb-replicaset-0",
          "service": "mongodb-mongodb-replicaset"
        },
        "scrapeUrl": "http://192.168.96.71:9001/metrics",
        "lastError": "",
        "lastScrape": "2018-07-12T16:54:27.229485336Z",
        "health": "up"
      },
```