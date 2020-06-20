

lab1 主要实现了一个简单版本的mapreduce系统

主要思路:测试脚本test-mr.sh中我们看到测试主要是通过启动一个master 进程读取需要map任务的文件,然后通过启动N 个 worker进程 来动态加载map reduce func 

这要求我们:
* master 跟worker 必须通过rpc的方式进行通信
* 一个master 多个worker
* master需要对map,reduce任务状态,worker状态的维护
* worker状态 包括心跳,任务被分配,任务完成等
  

在crash.go 中模拟了两种worker崩溃的情况,一种程序直接退出,一种是worker运行特别长时间.

* 所以每个map or reduce任务应该有一个大概时间,如果worker超时的话应该分配另一个worker去执行(master去通知worker暂停or退出,然后master重新分配)

结果不要求汇总到一起,可以根据 ihash(kva[i].Key)%nReduce  ==reduceindex的方式存储到 mr-out-reduceindex中



本lab1的代码主要修改了6.824/src/mr/master.go,worker.go,rpc.go,新增了6.824/src/mr/common.go 来保存一些基本结构体信息





结果:
```
$ sh test-mr.sh  
*** Starting wc test.
2020/06/20 rpc.Register: method "Done" has 1 input parameters; needs exactly three
2020/06/20 rpc.Register: method "MapDone" has 1 input parameters; needs exactly three
--- wc test: PASS
*** Starting indexer test.
2020/06/20 rpc.Register: method "Done" has 1 input parameters; needs exactly three
2020/06/20 rpc.Register: method "MapDone" has 1 input parameters; needs exactly three
--- indexer test: PASS
*** Starting map parallelism test.
2020/06/20 rpc.Register: method "Done" has 1 input parameters; needs exactly three
2020/06/20 rpc.Register: method "MapDone" has 1 input parameters; needs exactly three
--- map parallelism test: PASS
*** Starting reduce parallelism test.
2020/06/20 rpc.Register: method "Done" has 1 input parameters; needs exactly three
2020/06/20 rpc.Register: method "MapDone" has 1 input parameters; needs exactly three
--- reduce parallelism test: PASS
*** Starting crash test.
2020/06/20 rpc.Register: method "Done" has 1 input parameters; needs exactly three
2020/06/20 rpc.Register: method "MapDone" has 1 input parameters; needs exactly three
2020/06/20 dialing: dial unix /var/tmp/824-mr-worker9831: connect: connection refused
2020/06/20 dialing: dial unix /var/tmp/824-mr-worker9860: connect: connection refused
2020/06/20 dialing: dial unix /var/tmp/824-mr-worker9833: connect: connection refused
illegal worker done map task
2020/06/20 dialing: dial unix /var/tmp/824-mr-worker9833: connect: connection refused
2020/06/20 dialing: dial unix /var/tmp/824-mr-worker9825: connect: connection refused
illegal worker done reduce task,and redo the reduce task 
2020/06/20 dialing: dial unix /var/tmp/824-mr-worker10036: connect: connection refused
illegal worker done reduce task,and redo the reduce task 
2020/06/20 dialing: dial unix /var/tmp/824-mr-worker9832: connect: connection refused
--- crash test: PASS
*** PASSED ALL TESTS

```