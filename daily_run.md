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
- Prometheus enabled
- Env: export MASTER_DISK_SIZE=500GB MASTER_ROOT_DISK_SIZE=500GB KUBE_GCE_ZONE=us-central1-b MASTER_SIZE=n1-highmem-96 NODE_SIZE=n1-highmem-16 NUM_NODES=55 NODE_DISK_SIZE=500GB GOPATH=$HOME/go KUBE_GCE_ENABLE_IP_ALIASES=true KUBE_GCE_PRIVATE_CLUSTER=true CREATE_CUSTOM_NETWORK=true ETCD_QUOTA_BACKEND_BYTES=8589934592 TEST_CLUSTER_LOG_LEVEL=--v=2 ENABLE_KCM_LEADER_ELECT=false ENABLE_SCHEDULER_LEADER_ELECT=false SHARE_PARTITIONSERVER=true APISERVERS_EXTRA_NUM=0 WORKLOADCONTROLLER_EXTRA_NUM=0 ETCD_EXTRA_NUM=0 LOGROTATE_FILES_MAX_COUNT=50 LOGROTATE_MAX_SIZE=200M KUBEMARK_NUM_NODES=5000 KUBE_GCE_INSTANCE_PREFIX=daily1005-1a0w1e KUBE_GCE_NETWORK=daily1005-1a0w1e
- Cmd: 
GOPATH=$HOME/go nohup ./perf-tests/clusterloader2/run-e2e.sh --nodes=5000 --provider=kubemark --kubeconfig=/home/sonyali/go/src/k8s.io/arktos/test/kubemark/resources/kubeconfig.kubemark --report-dir=/home/sonyali/logs/perf-test/gce-5000/arktos/1005-daily1005-1a0w1e --testconfig=testing/load/config.yaml --testconfig=testing/density/config.yaml 


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