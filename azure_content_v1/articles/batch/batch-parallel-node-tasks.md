<properties
    pageTitle="Maximize Batch node use with parallel tasks | Microsoft Azure"
    description="Increase efficiency and lower costs by using fewer compute nodes while running concurrent tasks on each node in an Azure Batch pool"
    services="batch"
    documentationCenter=".net"
    authors="mmacy"
    manager="timlt"
    editor="" />

<tags
    ms.service="batch"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows"
    ms.workload="big-compute"
    ms.date="01/22/2016"
    ms.author="marsma" />

# Maximize Azure Batch compute resource usage with concurrent node tasks
In this article, you will learn how to run more than one task simultaneously on each compute node within your Azure Batch pool. By enabling concurrent task execution on a pool's compute nodes, you can maximize resource usage on a smaller number of nodes within the pool. For some workloads, this can save you both time and money.

While some scenarios benefit from all of a node's resources being available for allocation to a single task, a number of situations benefit from allowing multiple tasks to share those resources:

* **Minimizing data transfer** when tasks are able to share data. In this scenario, you can dramatically reduce data transfer charges by copying shared data to a smaller number of nodes and executing tasks in parallel on each node. This especially applies if the data to be copied to each node must be transferred between geographic regions.

* **Maximizing memory usage** when tasks require a large amount of memory, but only during short periods of time, and at variable times during execution. You can employ fewer, but larger, compute nodes with more memory to efficiently handle such spikes. These nodes would have multiple tasks running in parallel on each node, but each task would take advantage of the nodes' plentiful memory at different times.

* **Mitigating node number limits** when inter-node communication is required within a pool. Currently, pools configured for inter-node communication are limited to 50 compute nodes. Therefore, a greater number of tasks can be executed simultaneously if each node in such a pool is able to execute tasks in parallel.

* **Replicating an on-premises compute cluster**, such as when you first move a compute environment to Azure. You can increase the maximum number of node tasks to more closely mirror an existing physical configuration if that configuration currently executes multiple tasks per compute node.


## Example scenario
Here's an example that illustrates the benefits of parallel task execution. Let's say that your task application has CPU and memory requirements such that a Standard\_D1 node size is suitable. But in order to execute the job in the required time, 1,000 such nodes are needed.

Instead of using Standard\_D1 nodes which have 1 CPU core, you could employ Standard\_D14 nodes that have 16 cores each, and enable parallel task execution. In this case, *16 times fewer nodes* could therefore be used--instead of 1,000 nodes, only 63 would be required. This will greatly improve job execution time and efficiency if large application files or reference data are required for each node.

## Enable parallel task execution
You configure the compute nodes in your Batch solution for parallel task execution at the pool level. When using the Batch .NET library, you set the [CloudPool.MaxTasksPerComputeNode](http://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.maxtaskspercomputenode.aspx) property when you create a pool. If you are using the Batch REST API, you set the [maxTasksPerNode](https://msdn.microsoft.com/library/azure/dn820174.aspx) element in the request body during pool creation.

Azure Batch allows you to set maximum tasks per node up to four times (4x) the number of node cores. For example, if the pool is configured with nodes of size "Large" (four cores), then `maxTasksPerNode` may be set to 16. For details on the number of cores for each of the node sizes, see [Sizes for Cloud Services](./../cloud-services/cloud-services-sizes-specs.md). For more information on service limits, see [Quotas and limits for the Azure Batch service](batch-quota-limit.md).

> [!TIP]
> Be sure to take into account the `maxTasksPerNode` value when you construct an [autoscale formula](https://msdn.microsoft.com/library/azure/dn820173.aspx) for your pool. For example, a formula that evaluates `$RunningTasks` could be dramatically affected by an increase in tasks per node. See [Automatically scale compute nodes in an Azure Batch pool](batch-automatic-scaling.md) for more information.
> 
> 
## Distribution of tasks
When the compute nodes within a pool are able to execute tasks concurrently, it is important to specify how you want your tasks distributed across the nodes within the pool.

By using the [CloudPool.TaskSchedulingPolicy](https://msdn.microsoft.com/library/microsoft.azure.batch.cloudpool.taskschedulingpolicy.aspx) property, you can specify that tasks should be assigned evenly across all nodes in the pool ("spreading"). Or you can specify that as many tasks as possible should be assigned to each node before tasks are assigned to another node in the pool ("packing").

As an example of how this feature is valuable, consider the pool of Standard\_D14 nodes (in the example above) that is configured with a [CloudPool.MaxTasksPerComputeNode](http://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.maxtaskspercomputenode.aspx) value of 16. If the [CloudPool.TaskSchedulingPolicy](https://msdn.microsoft.com/library/microsoft.azure.batch.cloudpool.taskschedulingpolicy.aspx) is configured with a [ComputeNodeFillType](https://msdn.microsoft.com/library/microsoft.azure.batch.common.computenodefilltype.aspx) of *Pack*, it would maximize usage of all 16 cores of each node and allow an [autoscaling pool](./batch-automatic-scaling.md) to prune unused nodes from the pool (nodes without any tasks assigned). This minimizes resource usage and saves money.

## Batch .NET example
This [Batch .NET](http://msdn.microsoft.com/library/azure/mt348682.aspx) API code snippet shows a request to create a pool that contains four large nodes with a maximum of four tasks per node. It specifies a task scheduling policy that will fill each node with tasks prior to assigning tasks to another node in the pool. For more information on adding pools by using the Batch .NET API, see [BatchClient.PoolOperations.CreatePool](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.createpool.aspx).

        CloudPool pool = batchClient.PoolOperations.CreatePool(poolId: "mypool",
                                                            osFamily: "2",
                                                            virtualMachineSize: "large",
                                                            targetDedicated: 4);
        pool.MaxTasksPerComputeNode = 4;
        pool.TaskSchedulingPolicy = new TaskSchedulingPolicy(ComputeNodeFillType.Pack);
        pool.Commit();

## Batch REST example
This [Batch REST](http://msdn.microsoft.com/library/azure/dn820158.aspx) API snippet shows a request to create a pool that contains two large nodes with a maximum of four tasks per node. For more information on adding pools by using the REST API, see [Add a pool to an account](https://msdn.microsoft.com/library/azure/dn820174.aspx).

        {
          "id": "mypool",
          "vmSize": "Large",
          "osFamily": "2",
          "targetOSVersion": "*",
          "targetDedicated": 2,
          "enableInterNodeCommunication": false,
          "maxTasksPerNode": 4
        }

> [!NOTE]
> You can set the `maxTasksPerNode` element and [MaxTasksPerComputeNode](http://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.maxtaskspercomputenode.aspx) property only at pool creation time. They cannot be modified after a pool has already been created.
> 
> 
## Explore the sample project
Check out the [ParallelNodeTasks](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/ParallelTasks) project on GitHub. It is a working code sample that illustrates the use of [CloudPool.MaxTasksPerComputeNode](http://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.maxtaskspercomputenode.aspx).

This C# console application uses the [Batch .NET](http://msdn.microsoft.com/library/azure/mt348682.aspx) library to create a pool with one or more compute nodes. It executes a configurable number of tasks on those nodes to simulate variable load. Output from the application specifies which nodes executed each task. The application also provides a summary of the job parameters and duration. The summary portion of the output from two different runs of the sample application appears below.

```
Nodes: 1
Node size: large
Max tasks per node: 1
Tasks: 32
Duration: 00:30:01.4638023
```

The first execution of the sample application shows that with a single node in the pool and the default setting of one task per node, the job duration is over 30 minutes.

```
Nodes: 1
Node size: large
Max tasks per node: 4
Tasks: 32
Duration: 00:08:48.2423500
```

The second run of the sample shows a significant decrease in job duration. This is because the pool was configured with four tasks per node, which allows for parallel task execution to complete the job in nearly a quarter of the time.

> [!NOTE]
> The job durations in the summaries above do not include pool creation time. Each of the jobs above was submitted to previously created pools whose compute nodes were in the *Idle* state at submission time.
> 
> 
## Batch Explorer Heat Map
The [Azure Batch Explorer](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchExplorer), one of the Azure Batch [sample applications](https://github.com/Azure/azure-batch-samples), contains a *Heat Map* feature that provides visualization of task execution. When you're executing the [ParallelTasks](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/ParallelTasks) sample application, you can use the Heat Map feature to easily visualize the execution of parallel tasks on each node.

![Batch Explorer Heat Map][1]

*Batch Explorer Heat Map showing a pool of four nodes, with each node currently executing four tasks*

[api_net]: http://msdn.microsoft.com/library/azure/mt348682.aspx
[api_rest]: http://msdn.microsoft.com/library/azure/dn820158.aspx
[batch_explorer]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchExplorer
[cloudpool]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.aspx
[enable_autoscaling]: https://msdn.microsoft.com/library/azure/dn820173.aspx
[fill_type]: https://msdn.microsoft.com/library/microsoft.azure.batch.common.computenodefilltype.aspx
[github_samples]: https://github.com/Azure/azure-batch-samples
[maxtasks_net]: http://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.maxtaskspercomputenode.aspx  
[maxtasks_rest]: https://msdn.microsoft.com/library/azure/dn820174.aspx
[parallel_tasks_sample]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/ParallelTasks
[poolcreate_net]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.createpool.aspx
[task_schedule]: https://msdn.microsoft.com/library/microsoft.azure.batch.cloudpool.taskschedulingpolicy.aspx

[1]: ./media/batch-parallel-node-tasks\heat_map.png
