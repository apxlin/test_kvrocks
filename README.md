# **CaaS-LSM**

## **Dependencies**

- Linux - Ubuntu
    - Prepare for the dependencies of RocksDB: [https://github.com/facebook/rocksdb/blob/main/INSTALL.md](https://github.com/facebook/rocksdb/blob/main/INSTALL.md)
    - Install and config HDFS server
    - Install gRPC

## Baselines

- Notice that multiple CSAs should bind with one Control Plane.
### **Rocks-Local**
Search the repository for this code and delete it.
```c++
tmp_options.compaction_service = std::make_shared<MyTestCompactionService>(
      dbname, compaction_options, compaction_stats, remote_listeners,
      remote_table_properties_collector_factories);
```


### **CaaS-LSM**

- Config the address of Control Plane, CSA, and HDFS server in `include/rocksdb/options.h`
- Build and compile
- Run Control Plane and CSA

```shell
cd $build
./procp_server #run Control Plane server
./csa_server #run CSA server
```


### **Disaggre-RocksDB**
- Config the address of CSA, and HDFS server in `include/rocksdb/options.h`
- Build and compile
- Run CSA

```
git checkout disaggre-rocksdb
cd $build
./csa_server # The name is the same, but the function of CSA is different with that of CaaS-LSM
```

### **Terark-Local**
- clone repo: https://github.com/bytedance/terarkdb
- Build and compile

### **Terark-Native**
- checkout branch to ```terark-native```
- Build and compile
- Use ```remote_compaction_worker_101```

### **Terark-CaaS**
- Copy the code in ```db/compaction/remote_compaction``` of CaaS-LSM, including ```procp_server.cc```, ```csa_server.cc```, ```utils.h```, ```compaction_service.proto```
- Change ```CompactionArgs``` to ```string```, since TerarkDB uses encoded string in network transmit.
- Use the same way in CaaS-LSM to start.


## Test CaaS-LSM in distributed applications

### Test Nebula
- Use the branch ```nebula```, follow the tips in https://docs.nebula-graph.io/3.2.0/4.deployment-and-installation/2.compile-and-install-nebula-graph/1.install-nebula-graph-by-compiling-the-source-code/ 
- when compiling, replace ```build/third-party/install/include/rocksdb/``` and ```build/third-party/install/lib/librocksdb.a``` with the header files and libs produced by ```main``` branch of this repo.
- If the `main` branch cannot compile, you can try `rest_rpc` branch

### Test Kvrocks
- Clone `Kvrocks` at https://github.com/apache/incubator-kvrocks
- Before build:
    - modify this part in "cmake/rocksdb.cmake" to switch the branch of the default RocksDB to this repository
     ```
     FetchContent_DeclareGitHubWithMirror(rocksdb
        facebook/rocksdb v7.8.3
        MD5=f0cbf71b1f44ce8f50407415d38b9d44
      )
     ```

- Build: ```./x.py build```
- Single mode:
    - build/kvrocks -c kvrocks.conf 
- Cluster mode:
    - Based on ```kvrocks controller``` https://github.com/KvrocksLabs/kvrocks_controller.git with commit ```df83752849ef41ce91037ca5c9cc6c670a480d56```
    - Dependencies: etcd https://etcd.io/docs/v3.5/install/
    - Build ```kvrocks controller```: make
    - Start controller server: ```./_build/kvrocks-controller-server -c ./config/config.yaml```
    - A fast way to build cluster: ```python scripts/e2e_test.py```
    - Check cluster status: ```./_build/kvrocks-controller-cli -c ./config/kc_cli_config.yaml```
    - modify kvrocks.conf: port(e.g., 30001-30006), cluster-enabled(yes), dir /tmp/kvrocks(/tmp/kvrocks1-6)
