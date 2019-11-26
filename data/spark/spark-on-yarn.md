# Spark on Yarn

![](../../.gitbook/assets/data/spark/spark-on-yarn.png)

Spark on Yarn 的启动

## 注册

```scala
java SparkSubmit

// JVM
- Process(SparkSubmit)

// 启动进程
- main

    - SparkSubmitArguments

    // 提交
    - SUBMIT  ->  submit()

        // 准备环境
        - prepareSubmitEnvironment

            - childMainClass = "org.apache.spark.deploy.yarn.Client"
            - childMainClass = args.mainClass
        - doRunMain  ->  runMain

            反射加载类
            - Utils.ClassForName(childMainClass)
            查找main方法
            - mainClass.getMethod("main", new Array[String](0).getClass)
            调用main方法
            - mainMethod.invoke


2 Client

- main

    - new ClientArguments

    - new Client

        - yarnClient = YarnClient.createYarnClient

    - client.run

        - appId = submitApplication

        // 封装指令
        // command = bin/java org.apache.spark.deploy.yarn.ApplicationMaster
        // (command = bin/java org.apache.spark.deploy.yarn.ExecutorLauncher  client)
        - createContainerLaunchContext
        - createApplicationSubmissionContext

        // 向Yarn提交应用，提交指令
        - yarnClient.submitApplication(appCentext)
```


```scala
3 ApplicationMaster

// 启动进程
- main

    - new ApplicationMasterArguments(args)

    // 创建应用管理器
    - new ApplicationMaster(amArgs, new YarnRMClient)

    // 运行
    - runDriver

        // 启动用户指定的spark应用
        - startUserApplication

            // 获取用户应用的main方法
            - userClassLoader.loadClass(args.userClass).getMethod("main", classOf[Array[String]])

            // 启动Driver线程，执行用户类的main方法
            - new Thread.start

        // 注册ApplicationMaster
        - registerAM

            // 获取Yarn资源
            // client: YarnRMClient
            - client.register

            // 分配Yarn资源
            - allocator.allocateResources()

                // 处理可分配资源
                - handleAllocatedContainers(allocatedContainers.asScala)

                    // 运行可分配的Container
                    - runAllocatedContainers(containersToUse)

                        - new ExecutorRunnable().run

                            - startContainer

                                // 封装指令
                                // command = bin/java org.apache.spark.executor.CoarseGrainedExecutorBackend
                                - perpareCommand
```

```scala
4 CoarseGrainedExecutorBackend

// RpcEndpint
/*
 * The life-cycle of an endpoint is:
 *
 * constructor -> onStart -> receive* -> onStop
 */

    - onStart
        // ExecutorBackend向ApplicationMaster反向注册
        - ref.ask[Boolean](RegisterExecutor(executorId, self, hostname, cores, extractLogUrls))

    - receive

        // 注册Executor
        - case RegisteredExecutor
            - new Executor(executorId, hostname, env, userClassPath, isLocal = false)

        // 运行任务
        - case LaunchTask(data)
            - executor.launchTask(this, taskDesc)

// 启动进程
- main

    - run

        env.rpcEnv.setupEndpoint("Executor", new CoarseGrainedExecutorBackend(env.rpcEnv, driverUrl, executorId, hostname, cores, userClassPath, env))

        dispatcher.registerRpcEndpoint(name, endpoint)

// Executor的后台进程
=> CoarseGrainedExecutorBackend
    // 注册
    -> ref.ask[Boolean](RegisterExecutor(executorId, self, hostname, cores, extractLogUrls))
// Scheduler的后台进程
=> CoarseGrainedSchedulerBackend(SparkContext)
    -> addressToExecutorId(executorAddress) = executorId
    -> totalCoreCount.addAndGet(Executor.cores)
    -> totalRegisteredExecutors.addAndGet(1)
    -> executorRef.send(RegisteredExecutor)
    -> executorDataMap.put(executorId, data)
// ExecutorBackend进程 新建 Executor对象
=> CoarseGrainedExecutorBackend
    -> executor = new Executor(executorId, hostname, env, userClassPath, isLocal = false)

```

执行action算子，最终都是调用了`RDD.SparkContext.runJob`

```scala
1 SparkContext

- runJob

    - dagScheduler.runJob(rdd, cleanedFunc, partitions, callSite, resultHandler, localProperties.get)
```

## DAG任务

### 划分Stage，

```scala
2 DAGScheduler

- runJob

    - submitJob

        // 提交 JobSubmitted
        // 此处最终为 BlockingQueue(阻塞队列) 的 put(提交)
        - eventProcessLoop.post(JobSubmitted(jobId, rdd, func2, partitions.toArray, callSite, waiter, SerializationUtils.clone(properties)))

        // 提交后 EventLoop 执行 run() -> onReceive -> EventLoop 的实现类 DAGSchedulerEventProcessLoop 的onReceive
        => onReceive(eventQueue.take())

        - case JobSubmitted => dagScheduler.handleJobSubmitted

            -> finalStage = createResultStage

                -> val parents = getOrCreateParentStages(rdd, jobId)

                    -> getShuffleDependencies(rdd)

                        // 遍历 rdd 的所有依赖，遇到ShuffleDependecy则放入parents: HashSet[ShuffleDependency]中最后一起返回
                        -> waitingForVisit.push(rdd)
                        -> waitingForVistit .pop -> toVisit
                            toVisit.dependencies.foreach {
                            case shuffleDep: ShuffleDependency[_, _, _] => parents += shuffleDep
                            case dependency => waitingForVisit.push(dependency.rdd)
                            }

                    -> getOrCreateShuffleMapStage(shuffleDep, firstJobId)

                        -> shuffleIdToMapStage.get

                        // 逻辑同getShuffleDependencies 最终返回 ancestors: Stack[ShuffleDependency]
                        -> None => getMissingAncestorShuffleDependencies

                        -> 

                -> val stage = new ResultStage(id, rdd, func, partitions, parents, jobId, callSite)
            
            // Job 里包含了 Stage (finalStage) ，finalStage中包含了 parents: List[Stage]
            -> val job = new ActiveJob(jobId, finalStage, callSite, listener, properties)

            // 设置 Job 到 fianlStage: ResultStage 中
            -> finalStage.setActiveJob(job)

            -> submitStage(finalStage)
```

### 提交Stage

生成Task
```scala
2 DAGScheduler

//提交任务
-> submitStage(finalStage)

    // 获取 stage里的rdd，遍历rdd.dependecies，获取其中的ShuffleDependecy，放入missing中并返回
    -> val missing = getMissingParentStages(stage).sortBy(_.id)

    // 如果 missing 为空，则提交任务，
    // 否则递归调用 submitStage(missing.parent)
    // 先提交最底依赖的父Stage，然后子Stage提交
    -> submitMissingTasks(stage, jobId.get)

        // 每个partitions创建一个Task
        -> val tasks: Seq[Task[_]] = stage match {
            case stage: ShuffleMapStage => partitionsToCompute.map(new ShuffleMapTask)
            case stage: ResultStage => partitionsToCompute.map(new ResultTask)
           }
        
        // 把前面创建的tasks创建为TaskSet
        // 如果DAGScheduler中的`!tasks.size>0`，submitWaitingChildStages(stage)
        -> taskScheduler.submitTasks(new TaskSet(tasks, ...))
            
            // TaskScheDuler发送提交Task的唤醒提交的请求
            -> CoarseGrainedSchedulerBackend.reviveOffers
```

Driver发送Task给Executor，Executor启动Task
```scala
3 CoarseGrainedSchedulerBackend

-> reviveOffers

    // 给Driver终端发送ReviveOffers
    -> driverEndpoint.send(ReviveOffers)

        -> makeOffers()

            -> launchTasks(taskDescs)

                // task的 HashMap 序列化发送给Executor
                -> executorDataMap(task.executorId).executorEndpoint.send(LaunchTask(new SerializableBuffer(serializedTask)))

4 CoarseGrainedExecutorBackend

- LaunchTask

    // 接收到的task(data.value)反编码成TaskDescription
    // 启动Task
    -> executor.launchTask(this, TaskDescription.decode(data.value))

```

以上，Spark On Yarn的 Cluter模式的调度。
