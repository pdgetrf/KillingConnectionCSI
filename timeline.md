

### Common Test Setup (specify specific test change below )

- 5000 nodes
- Load test only
- WCM disabled
- Perf-test tool: https://github.com/sonyafenge/perf-tests.git
- ETCD 3.4.3
- Leader-election disabled
- Coredump enabled
- Insecure-port enabled
- Prometheus enabled

### Build Date: Pre-Alkaid  - Baseline: density test passed, load test failed but completed with timeouts
- Commit: 75e94764
- Runner: Vinay
- Date of run: Sep 22
- Test: density + load
- Result:
  - Density test passed. Load test failed but completed with object timeouts.
  - Logs don't show "killing connection" or panics or apiserver kills.
  - sonyafenge@sonyadev:/home/sonyafenge/vinaykul/logs/perf-test/sep22_pre_alkaid_etcd343_5k_1api

### Build Date: Apr 05  - density and load tests failed but completed with timeouts
- Commit: 4fa2fd3d1385cc71a11a4e5216ea928c7a1e4ad8
- Runner: Vinay
- Date of run: Sep 23
- Test: density + load
- Result:
  - Density test passed. Load test completed with object timeouts.
  - Logs don't show "killing connection" or panics or apiserver kills.
  - sonyafenge@sonyadev:/home/sonyafenge/vinaykul/logs/perf-test/sep23_ark_apr_etcd343_5k_1api

### Build Date: 06/10 No fail


### Build Date: 06/12 No fail
date of run:


### Build Date: 06/18

- Commit: 91f3f2bb1d89038223264eb246552e856559c050

- Runner: Joe

- Date of run:

- Result: 
  - Many  Kube-apiserver panics. “killing connection/stream”
  -  The load test completed with 1 failure that many objects got timed out as usual
  - Not many “took too long” warning of “registry/leases/kube-node-lease/hollow-node” as usual. There are only 2712 out of the 38294 “took too long” warnings. It used to be more than 80%.
  - There are many context canceled errors in the etcd.log
  - Log: jundev2:/home/shaoj9/logs/perf-test/gce-5000/arktos/0929-0618-91f3



### Build Date: 06/25

- Commit: ???

- Runner: Vinay

- Date of run:

- Result: 
  -  load test completed with timeouts. No panic or killing connection logs.

---

- Commit: ???

- Runner: Joe

- Date of run:

- Result: 
  -  N/A


### Build Date: 09/24

- Commit: 40fd8c82fbd0446c2bf32a558004849335025120

- Runner: Sonya

- Date of run:

- Result: 
  - Kube-apiserver Panic: killing connection/stream
  - Perf-tests tool panic: runtime error: invalid memory address or nil pointer dereference
  - logs can be found under GCP project: workload-controller-manager on sonya-useast1: /home/sonyali/logs/perf-test/gce-5000/arktos/0929-communityperf-1a0w1e
  - Prometheus report can be found at [http://35.231.121.60:9090/](https://nam11.safelinks.protection.outlook.com/?url=http%3A%2F%2F35.231.121.60%3A9090%2F&data=02|01|peng.du%40futurewei.com|50f8201d0df9413f5dbf08d865752edb|0fee8ff2a3b240189c753a1d5591fedc|1|0|637370901289650492&sdata=uINL%2B1heyhSHkiGabpRKXAXtUwaE6Nstz0qmVzd24Sc%3D&reserved=0) or snapshot can be found on sonya-useast1: /home/sonyali/logs/perf-test/gce-5000/arktos/0929-communityperf-1a0w1e/logs/crashed/kubemarkmaster/prometheus/snapshots

 



