## Common Test Setup (specify specific test change below )

- 5000 nodes
- 1 Apiserver
- 1 ETCD instance
- Load test only
- WCM disabled
- Perf-test tool: https://github.com/sonyafenge/perf-tests.git
- ETCD 3.4.3
- Leader-election disabled
- Coredump enabled
- Insecure-port enabled
- apiserver pprof enabled
- Prometheus enabled
- Env: export MASTER_DISK_SIZE=500GB MASTER_ROOT_DISK_SIZE=500GB KUBE_GCE_ZONE=us-central1-b MASTER_SIZE=n1-highmem-96 NODE_SIZE=n1-highmem-16 NUM_NODES=55 NODE_DISK_SIZE=500GB GOPATH=$HOME/go KUBE_GCE_ENABLE_IP_ALIASES=true KUBE_GCE_PRIVATE_CLUSTER=true CREATE_CUSTOM_NETWORK=true ETCD_QUOTA_BACKEND_BYTES=8589934592 TEST_CLUSTER_LOG_LEVEL=--v=2 ENABLE_KCM_LEADER_ELECT=false ENABLE_SCHEDULER_LEADER_ELECT=false SHARE_PARTITIONSERVER=true APISERVERS_EXTRA_NUM=0 WORKLOADCONTROLLER_EXTRA_NUM=0 ETCD_EXTRA_NUM=0 LOGROTATE_FILES_MAX_COUNT=50 LOGROTATE_MAX_SIZE=200M KUBEMARK_NUM_NODES=5000 KUBE_GCE_INSTANCE_PREFIX=daily1005-1a0w1e KUBE_GCE_NETWORK=daily1005-1a0w1e
- Cmd: 
GOPATH=$HOME/go nohup ./perf-tests/clusterloader2/run-e2e.sh --nodes=5000 --provider=kubemark --kubeconfig=/home/sonyali/go/src/k8s.io/arktos/test/kubemark/resources/kubeconfig.kubemark --report-dir=/home/sonyali/logs/perf-test/gce-5000/arktos/1005-daily1005-1a0w1e --testconfig=testing/load/config.yaml --testconfig=testing/density/config.yaml 

## Start Prometheus Server Using Stored Data
```bash
sudo /home/sonyali/tools/prometheus-2.2.1.linux-amd64/prometheus --config.file="/home/sonyali/tools/prometheus-2.2.1.linux-amd64/prometheus.yml" --web.listen-address=":9090" --storage.tsdb.path [path to prometheus snapshots]
```

For example:
On sonyadev4:

```bash
sudo prometheus  --web.listen-address=":9090" --config.file="/home/sonyali/tools/prometheus-2.2.1.linux-amd64/prometheus.yml"  --storage.tsdb.path /home/sonyali/logs/perf-test/gce-5000/arktos/1009-daily1009-1a0w1e/logs/crashed/kubemarkmaster/prometheus/snapshots/20201010T155128Z-7d0bde96aec85605
```

Some queries aobut the [general healthy of etcd](http://35.188.21.94:9090/graph?g0.range_input=6h&g0.end_input=2020-10-10%2009%3A00&g0.expr=etcd_server_proposals_failed_total&g0.tab=0&g1.range_input=12h&g1.end_input=2020-10-10%2009%3A00&g1.expr=histogram_quantile(0.99%2C%20rate(etcd_disk_wal_fsync_duration_seconds_bucket%5B5m%5D))&g1.tab=0&g2.range_input=6h&g2.end_input=2020-10-10%2009%3A00&g2.expr=histogram_quantile(0.99%2C%20rate(etcd_disk_backend_commit_duration_seconds_bucket%5B5m%5D))&g2.tab=0&g3.range_input=6h&g3.end_input=2020-10-10%2009%3A00&g3.expr=etcd_mvcc_db_total_size_in_bytes&g3.tab=0&g4.range_input=6h&g4.end_input=2020-10-10%2009%3A00&g4.expr=apiserver_current_inflight_requests&g4.tab=0&g5.range_input=6h&g5.end_input=2020-10-10%2009%3A00&g5.expr=rate(etcd_server_slow_apply_total%5B5m%5D)&g5.tab=0&g6.range_input=6h&g6.end_input=2020-10-10%2009%3A00&g6.expr=sum(rate(apiserver_request_total%5B5m%5D))%20by%20(client%2C%20verb%2C%20scope%2C%20resource%2C%20code)&g6.tab=0)

Here are some queries about [memory usage](http://35.188.21.94:9090/graph?g0.range_input=6h&g0.end_input=2020-10-10%2009%3A00&g0.expr=process_resident_memory_bytes&g0.tab=0&g1.range_input=6h&g1.end_input=2020-10-10%2009%3A00&g1.expr=process_virtual_memory_bytes&g1.tab=0&g2.range_input=6h&g2.end_input=2020-10-10%2009%3A00&g2.expr=go_memstats_heap_alloc_bytes&g2.tab=0)

Set the end time to about an hour after the first time kubelet kills apiserver

## 10/05/2020
### Changes
https://github.com/futurewei-cloud/arktos-perftest/commits/perf-20201005
- Bugfix in ETCD3 store
- Enable mutex/block profiling for kube-apiserver
- Add trace for ETCD create and delete
### Result
- Perf-tests load testing finished with bunch of object creation failure because “connection refused”
- Kube-apiserver crashed with error “killing connection/stream”
- Checked ETCD logs, there is no error log except warning “took too long to execute”
- logs can be found under GCP project: workload-controller-manager on sonyadev4: /home/sonyali/logs/perf-test/gce-5000/arktos/1005-daily1005-1a0w1e
 
## 10/06/2020
### Changes
https://github.com/futurewei-cloud/arktos-perftest/commits/perf-20201006
- private change to getClient() in etcd3
- Add WatchListPageSize into reflector trace
- Bugfix in ETCD3 store
- Enable mutex/block profiling for kube-apiserver
- Add trace for ETCD create and delete
### Result
- Perf-tests stopped after 6 hrs.
- Perf-tests load testing failed with bunch of “connection refused” error, first error is at 07:44:22
- Perf-test load testing stopped per one panic: runtime error: invalid memory address or nil pointer dereference
- Kube-apiserver crashed with error “killing connection/stream”, first error is at 07:46.
- Checked ETCD logs, there is no error log except warning “took too long to execute”
- logs can be found under GCP project: workload-controller-manager on sonyadev4: /home/sonyali/logs/perf-test/gce-5000/arktos/1006-daily1006-1a0w1e

## 10/07/2020
### Changes
https://github.com/futurewei-cloud/arktos-perftest/tree/perf-20201007
- add pprof collecting
- logs to trace etcd operation
- log out cacher sent events
- Set ListAndWatch list always from "0" - read from cache.
- private change to getClient() in etcd3
- Add WatchListPageSize into reflector trace
- Bugfix in ETCD3 store
- Enable mutex/block profiling for kube-apiserver
- Add trace for ETCD create and delete
### Result
- Perf-tests stopped after 5 hrs.
- Perf-tests load testing failed with bunch of “connection refused” error, first error is at 09:09:25
- Kube-apiserver crashed with error “killing connection/stream”, first error is at 09:11:19.
- Checked ETCD logs, there is no error log except warning “took too long to execute”
- logs can be found under GCP project: workload-controller-manager on sonyadev4: /home/sonyali/logs/perf-test/gce-5000/arktos/1007-daily1007-1a0w1e
- kube-apiserver.log increased a lot by “cacher.go:1248] Sent event resourceVersion…”. we may need fix or remove this logs, just in case this may impact regular log investigation.
- Warining disappeared: "W1007 02:05:29.210768   14518 etcd_metrics.go:206] empty etcd cert or key, using http". all previous run has this warning keep 10-12 mins if we started from load testing. but this run don't have this warning.
- previous run warning detail:
```
I1007 02:04:29.462079   14518 simple_test_executor.go:162] Step "Starting measurements" ended
I1007 02:04:29.462088   14518 simple_test_executor.go:135] Step "Creating SVCs" started
W1007 02:05:29.210768   14518 etcd_metrics.go:206] empty etcd cert or key, using http
W1007 02:06:29.392507   14518 etcd_metrics.go:206] empty etcd cert or key, using http
W1007 02:07:29.576302   14518 etcd_metrics.go:206] empty etcd cert or key, using http
W1007 02:08:29.755684   14518 etcd_metrics.go:206] empty etcd cert or key, using http
W1007 02:09:29.941178   14518 etcd_metrics.go:206] empty etcd cert or key, using http
W1007 02:10:30.124754   14518 etcd_metrics.go:206] empty etcd cert or key, using http
W1007 02:11:30.299306   14518 etcd_metrics.go:206] empty etcd cert or key, using http
W1007 02:12:30.496042   14518 etcd_metrics.go:206] empty etcd cert or key, using http
W1007 02:13:30.671285   14518 etcd_metrics.go:206] empty etcd cert or key, using http
W1007 02:14:30.852418   14518 etcd_metrics.go:206] empty etcd cert or key, using http
W1007 02:15:31.033455   14518 etcd_metrics.go:206] empty etcd cert or key, using http
I1007 02:16:18.058678   14518 simple_test_executor.go:162] Step "Creating SVCs" ended
I1007 02:16:18.058744   14518 simple_test_executor.go:135] Step "Creating PriorityClass for DaemonSets" started
```
- this run logs:
```
I1008 04:10:50.402085   20528 simple_test_executor.go:162] Step "Starting measurements" ended
I1008 04:10:50.402093   20528 simple_test_executor.go:135] Step "Creating SVCs" started
I1008 04:11:47.355919   20528 simple_test_executor.go:162] Step "Creating SVCs" ended
I1008 04:11:47.355968   20528 simple_test_executor.go:135] Step "Creating PriorityClass for DaemonSets" started
```
- Comparing to 10/06, etcd log "took too long" is much less (616,985 5 hours run vs 3,984,223 on 10/06 6 hours run; community has 11,705 aug25_k8s-5k-1apiserver_qps200); more "cacher <objectType>: List" (1,236,025 vs. 450,311). This means API server cache was used much more than 10/07. Traffic shifted from ETCD to API server cache.
```
trace.go:81] Trace[2063583751]: "cacher *core.Endpoints: List" (started: 2020-10-08 15:16:58.277917468 +0000 UTC m=+37.589668692) (total time: 3.115902002s):
```


## 10/08/2020
### Changes
https://github.com/futurewei-cloud/arktos-perftest/tree/perf-20201008
- 10/07 build plus following
- Add log to check secret watch increasing.
- revert the logging change, we got the info in 10-07 run already
- Use original watch cache - does not support etcd partition
- Add watched from resource verson into error
- disable mizar controllers
### Result
- Perf-tests stopped after 5 hrs.
- Perf-tests load testing failed with bunch of “connection refused” error, first error is at 07:54:29
- Kube-apiserver crashed with error “killing connection/stream”, first error is at 07:56:31.
- Checked ETCD logs, there is no error log except warning “took too long to execute”
- logs can be found under GCP project: workload-controller-manager on sonyadev4: /home/sonyali/logs/perf-test/gce-5000/arktos/1008-daily1008-1a0w1e


## 10/09/2020
### Changes
https://github.com/futurewei-cloud/arktos-perftest/tree/perf-20201009
- 10/08 build plus following
- Added API server watch cacher creation log.
- Revert etcd partition changes in etcd3 store.List
- Added some logs to find out watcher leaks
- ETCD_SNAPSHOT_COUNT to 100000
- Update log - reduce log from perf test; remove verified information
- set GOMAXPROCS=96
- Added some logs to find out watcher leaks
- Removed the etcd partition config to start watching
### Result
- Perf-tests stopped after 5 hrs.
- Perf-tests load testing failed with bunch of “connection refused” error, first error is at 07:28:56
- Kube-apiserver crashed with error “killing connection/stream”, first error is at 07:32:31.
- Checked ETCD logs, there is no error log except warning “took too long to execute”
- logs can be found under GCP project: workload-controller-manager on sonyadev4: /home/sonyali/logs/perf-test/gce-5000/arktos/1009-daily1009-1a0w1e
- GCE cpu, network & disk metrics [here](screenshots/20201009)
### Analysis
- First time kubelet killed apiserver 07:35:20.273837 UTC
- **"took too long”** : 
    - Corresponds to ["etcd_server_slow_apply_total"](http://35.188.21.94:9090/graph?g0.range_input=6h&g0.end_input=2020-10-10%2009%3A00&g0.expr=etcd_server_slow_apply_total&g0.tab=0) in Prometheus
    - Appeared 753300 times in 763319 total (98%) lines in etcd.log
    - First one starts at 02:23:11.439679 UTC (5 mintes after etcd started at 02:18:23.766323 UTC)
    - Most *walk_fsync* and *backend_commit* duration below 4ms ([base on this query](http://35.188.21.94:9090/graph?g0.range_input=6h&g0.end_input=2020-10-10%2009%3A00&g0.expr=etcd_disk_wal_fsync_duration_seconds_bucket&g0.tab=0&g1.range_input=6h&g1.end_input=2020-10-10%2009%3A00&g1.expr=etcd_disk_wal_fsync_duration_seconds_bucket&g1.tab=0)), meaning no slow disk issue
    - Number of keys stops increasing around 6:50 UTC ([metrics](http://35.188.21.94:9090/graph?g0.range_input=6h&g0.end_input=2020-10-10%2009%3A00&g0.expr=etcd_debugging_mvcc_keys_total&g0.tab=0))
    - [Etcd latency distribution](http://35.188.21.94:9090/graph?g0.range_input=6h&g0.end_input=2020-10-10%2009%3A00&g0.expr=histogram_quantile(0.99%2Csum(rate(etcd_request_duration_seconds_bucket%5B1m%5D))%20by%20(le%2C%20operation%2C%20type))&g0.tab=0)
    - Resident memory usage around 40G, virtual memory around 70G at time of crash ([metrics](http://35.188.21.94:9090/graph?g0.range_input=6h&g0.end_input=2020-10-10%2009%3A00&g0.expr=process_resident_memory_bytes&g0.tab=0&g1.range_input=6h&g1.end_input=2020-10-10%2009%3A00&g1.expr=process_virtual_memory_bytes&g1.tab=0&g2.range_input=6h&g2.end_input=2020-10-10%2009%3A00&g2.expr=go_memstats_heap_alloc_bytes&g2.tab=0))


## 10/12/2020 Pre-Alkaid
### Changes
https://github.com/futurewei-cloud/arktos-perftest/tree/perf-20201012-prealkaid
- track etcd latency
- Add process event log for Secret
- Added logs to find out the watchers adding event with blocking
- Add log for Reflector watchlistpagesize, createWatchChan by key, watc… 
- ETCD 3.4.3
- Insecure-port enabled
- Coredump enabled
- apiserver pprof enabled,  also enable metux and block pprof for kube-apiserver
- Prometheus enabled
- Perf-tests build: https://github.com/sonyafenge/perf-tests/commit/d9a552b4f2e00dfc4c12e74eddae0a5e10aeed71

### Commands: [here](pre-alkaid-commands.md)

### Result
- Perf-tests load testing finished with timeout error and took 11+ hrs.
- Cluster is running good after perf-test finished
- Logs can be found under GCP project: workload-controller-manager on sonyadev4: /home/sonyali/logs/perf-test/gce-5000/kubernetes/prealkaid1012-1a0w1e

### Prometheus Server
- [Pre-Alkaid](http://35.188.21.94:9091/graph), set end time to 2020-10-13 18:30 and length to 12 hr
- [10/09/2020 run](http://35.188.21.94:9090/graph), set end time to 2020-10-10 08:00 and length to 6 hr

### GCP Metrics
- [cpu](screenshots/pre-alkaid/cpu.png)
- [disk](screenshots/pre-alkaid/disk.png)
    - [book disk](screenshots/pre-alkaid/disk_details_boot_disk.png)
    - [book disk](screenshots/pre-alkaid/master_pd.png)
- [mirror network](screenshots/pre-alkaid/mirrored_Network.png), all zero
- [network packets and bytes](screenshots/pre-alkaid/network_Bytes_Packets.png)

### Analysis

- **"took too long”** : 
    - Appeared 71991 times ouf of 74282 lines in etcd.log **(97%)**
    - First one appeared at 06:36:54.343561 UTC, **13 mintues** after run started at 06:23:55.331373 UTC


## 10/13/2020
### Changes
https://github.com/futurewei-cloud/arktos-perftest/tree/perf-20201013
- 10/09 build plus following
- Disable APIServerConfigManager in kubelet/hollow nod
### Result
- Perf-tests stopped after 5 hrs.
- Perf-tests load testing failed with bunch of “connection refused” error
- Kube-apiserver crashed with error “killing connection/stream”
- Logs can be found under GCP project: workload-controller-manager on sonyadev4: /home/sonyali/logs/perf-test/gce-5000/arktos/1013-daily1013-1a0w1e


### Prometheus Server
- [Pre-Alkaid](http://35.188.21.94:9091/graph), set end time to 2020-10-13 18:30 and length to 12 hr
- [10/09/2020 run](http://35.188.21.94:9090/graph), set end time to 2020-10-10 08:00 and length to 6 hr
- **[10/13/2020 run](http://35.188.21.94:9092/graph), set end time to 2020-10-14 06:00 and length to 6 hr**

### Analysis
- Time of Kill **05:10:42.168898 UTC**
- ETCD "took too long":
    - 285,247 times (288,895 total etcd lines, 98%) vs. 753,300 times on 10/09
    - first time it appeared at 00:00:41.855512 UTC, **7 minutes** after etcd starts
    - apiserver memory usage is still higher than baseline ([this](http://35.188.21.94:9092/graph?g0.range_input=6h&g0.end_input=2020-10-14%2006%3A00&g0.expr=process_resident_memory_bytes&g0.tab=0) vs [Pre-Alkaid](http://35.188.21.94:9091/graph?g0.range_input=12h&g0.end_input=2020-10-13%2018%3A30&g0.expr=process_resident_memory_bytes&g0.tab=0))
        - apiserver is running with "--target-ram-mb=300000"
        - This value is used for kube-apiserver to guess the size of the cluster and to configure the deserialize cache size and watch cache sizes inside the API server based on [this](https://applatix.com/making-kubernetes-production-ready-part-2/)


## 10/14/2020
### Changes
https://github.com/futurewei-cloud/arktos-perftest/tree/perf-20201014
- 10/13 build plus following
- Remove bcast from AggregatedWatcher
### Result
- Load testing finished with only timeout error and daemonset-priorityclass-0 creation error ( this is known issue for arktos), totally took 11hrs  
- Density successed and totally took 7 hrs
- no error “killing connection/stream”
- no crash in any of master components
- Logs can be found under GCP project: workload-controller-manager on sonyadev4: /home/sonyali/logs/perf-test/gce-5000/arktos/1014-daily1014-1a0w1e


## 10/15/2020 Qian's private build
### Changes
https://github.com/futurewei-cloud/arktos-perftest/commits/perf-20201014-qianprivate
- 10/13 build plus following
- ETCD grpc proxy
### Result
- debug is still in progress
- etcd grpc proxy doesn't support "cert has non empty CN" https://github.com/etcd-io/etcd/pull/10343, current client cert we are using in etcd client doesn't work for this.
- Qian is trying to generate different cert to start etcd grpc proxy.
