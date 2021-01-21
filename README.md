# Operators Subtask And Stream Partitions

<br/> ![](OperatorsSubtaskAndStreamPartitions/1.PNG)

* Here is a big picture overview of how operators that you define make up a directed‑acyclic graph transforming your input stream till the final results are written out using a sink. 

* The input stream is read in using a source operator, and final results are written out using a sink operator. 
* In between the source and the sink, you can have multiple transformation operators which work on your data.


<br/> ![](OperatorsSubtaskAndStreamPartitions/2.PNG)

* Flink programs are inherently parallel and distributed, which means that the operators that operate on streaming dataflows are spread across multiple nodes in your Flink cluster. 

* Data transformations run on subsets of streaming data, which means the streams within your Flink applications are composed of stream partitions. 

* Any operator performing transformation can be composed of operator subtasks. 
* Stream partitions and operator subtasks together are responsible for the distributed and parallel execution of a Flink application.


<br/> ![](OperatorsSubtaskAndStreamPartitions/3.PNG)
* At the very top here is the directed‑acyclic graph of your Flink application. 
* We read in data from the source, the source is an operator, and we write out data to a sink, and we perform a number of transformations on the data within the application. 
* Every node in this graph is an operator, and every edge in this graph is a stream. 
* The operators in your Flink application may be divided into operator subtasks that work on subsets of the input stream. 
* In this example here, the source operator is executed using two subtasks. 
* The source has a parallelism of 2. Every subtask will output results which form a stream. 
* These results are referred to as stream partitions because they are partitions of the original stream that we're working with. 
* The entities in each stream partition may then be forwarded along to other operators, which then operate on these stream partitions using operators subtasks. 
* As you can see, the map transformation here has two subtasks. 
* It has a parallelism of 2. Now, this continues. The remaining transformation, the keyBy window and apply, also runs with a parallelism of 2 till finally the output of the final operator is sent along to the sink operator, which writes it out to persistent storage somewhere.

<br/> ![](OperatorsSubtaskAndStreamPartitions/4.PNG)
* Every operator can be split into subtasks to process your input stream, and these subtasks run completely independently of one another. * These subtasks may be executed in different threads. 
* It's also possible for these subtasks to run on different machines or on different containers within a cluster. 
* The parallel execution of your dataflow operations is possible because of these subtasks. 
* The number of subtasks refer to the parallelism of the individual operator.

<br/> ![](OperatorsSubtaskAndStreamPartitions/5.PNG)
* Operators which make up the notes in our directed‑acyclic graph perform the actual transformation of input data. 
* The edges in our directed‑acyclic graph are responsible for transporting data between operators. 
* Edges are referred to as streams. 
* Now, transporting data between operators can be done in two ways. 
* You can have the forwarding pattern, which is a 1:1 pattern which preserves the partitioning and order of elements in the stream. 
* The elements in the stream partition are not shuffled or reordered but passed along as is. On the other hand, data can be transported using the redistributing pattern. 
* This is where every operator subtask sends data to different target subtasks. The original order of the data is not reserved.

<br/> ![](OperatorsSubtaskAndStreamPartitions/6.PNG)
* If you look back at our example here, you can see the forwarding pattern in action. 
* The stream partitions that move data between the two operator subtasks for the source and the map operator subtask transport data using the forwarding pattern.

<br/> ![](OperatorsSubtaskAndStreamPartitions/7.PNG)
* On the other hand, when data is transported between the map operator subtask and the keyBy window subtask, you can see that the data is redistributed. 
* The original stream partitions have changed, which means one operator subtask can send data to several target subtasks. 
* This is the redistributing pattern in action.

# Stateless And Stateful Transformations

* Flink allows you to specify different kinds of data transformations on input streams. 
* The kind of processing that you perform on input data can be broadly divided into stateless transformations and stateful transformations. 

<br/> ![](StatelessAndStatefulTransformations/1.PNG)

* Stateless transformations are those which are applied on a single streaming entity at a time. 
* The processing code for a stateless transformation looks at only one element in the input stream, it then acts on that single element to produce 0, 1, or more elements in the output stream. 
* On the other hand, stateful transformations are transformations which accumulate across multiple stream entities. 
* Stateful transformations don't operate on elements in isolation. 
* Instead, elements that have come previously in the stream, or are yet to arrive in the stream, may be included in the transform code. 

<br/> ![](StatelessAndStatefulTransformations/2.PNG)

* Stateless transformations are easier to understand and work with because they involve processing only one entity at a time. 
* Examples of stateless transformations in Flink include the map operation, the flatMap, and the filter operation. 
* The map operation produces a transformed output for every input entity. 
* The flatMap may produce 0 or more output entities for every input entity. 
* And the filter operation will filter entities from the output stream using some condition that has been defined.

<br/> ![](StatelessAndStatefulTransformations/3.PNG)

* Stateful transformations don't just look at one entity from the input stream, but they include data from more than one entity, either entities that have arrived earlier in the stream or entities that are yet to arrive in the stream. 
* Stateful operations thus involve accumulating data across a longer time interval. 
* This time interval can be the entire stream, a window of time within the stream, or this data can be accumulated per key or per operator. 
* Stateful transformations involve partitioning the stream in some way and then applying some kind of aggregation operation.

# Job Manager And Task Manager

<br/> ![](JobManagerAndTaskManager/1.PNG)

* There are two categories of processes running on a Flink cluster. 
* We have the Flink JobManager and you have one or more Flink TaskManagers. 
* The JobManager and TaskManagers can be started in a variety of different ways. 
* You can start them directly on machines which make up a standalone cluster, we can start them within Docker containers, or they can be managed by a resource framework such as Yarn or Mesos. 
* TaskManagers connect to JobManagers and announce themselves as available to be assigned work. 
* Let's talk about the JobManager in detail first to understand what exactly this process does.

<br/> ![](JobManagerAndTaskManager/2.PNG)

* The JobManager is responsible for coordinating the distributed execution of Flink applications. 
* The JobManager will work with the scheduler to schedule tasks and manage any failures that occur during your Flink job execution. 
* The JobManager is also responsible for coordinating checkpoints within your program to ensure that in the case of failures your program can recover from a previously checkpointed state.

<br/> ![](JobManagerAndTaskManager/3.PNG)

* The JobManager receives the JobGraph, which is the directed acycli graph of operators and streams which make up your processing pipeline. 
* The JobManager then transforms this JobGraph to an ExecutionGraph. 
* The difference between a JobGraph and an ExecutionGraph is that the ExecutionGraph is parallelized. 
* The ExecutionGraph is made up of the operator, subtask, and the stream partitions which actually run on the distributed cluster. 
* A JobStatus is also associated with an ExecutionGraph. The ExecutionGraph is parallelized, which means it contains one vertex per parallel subtask.

<br/> ![](JobManagerAndTaskManager/4.PNG)

* The JobManager has three different components which it uses to perform its operations. 
* The ResourceManager is responsible for resource allocation and deallocation and provisioning of a cluster. 
* The ResourceManager assigns resources to individual jobs using task slots. 
* Task slots are the smallest unit of resource scheduling in a Flink cluster. 
* Flink implements multiple ResourceManagers for different environment. 
* There are resource providers for Yarn, Mesos, Kubernetes, and standalone deployments. 
* Another component that the JobManager has is the dispatcher, which provides a REST interface to submit Flink applications for execution and starts a new JobMaster for each submitted job. 
* The dispatcher is also responsible for running the Flink web UI, which you can use to monitor your applications submitted to Flink. And finally, we have the JobMaster component which manages the execution of a single job. When you submit multiple Flink applications to a cluster each will have its own JobMaster. 

<br/> ![](JobManagerAndTaskManager/5.PNG)

* Every Flink application will have at least one JobMaster
* It can have one or more TaskManagers
* TaskManagers are the workers which execute the tasks of your streaming data flow
* The actual operations defined by the operators in your processing code are executed by the TaskManager
* The TaskManager is also responsible for buffering and exchanging data streams between operators
* The TaskManager assigns tasks using units of resource scheduling called task slots
* Every TaskManager has access to some resources on the machine on which it executes
 
<br/> ![](JobManagerAndTaskManager/6.PNG)

* A task slot represents a fixed subset of resources of the TaskManager
*  So a single TaskManager can be subdivided into multiple task slots, and a task slot is the smallest unit of resource scheduling in a Flink cluster
*  The number of task slots in a TaskManager indicates the number of concurrent processing tasks that can be executed by that TaskManager
*  Now it's possible for multiple operators to execute within the same task slot, so there is not a one‑is‑to‑one correspondence between a task slot and an operator
*  Let's visualize how TaskManagers and task slots work

<br/> ![](JobManagerAndTaskManager/7.PNG)

* Think of the TaskManager as executing processes which run your transformation code
*  Every TaskManager is divided into task slots, and task slots are run on different threads
*  The ExecutionGraph of your Flink application will contain operator subtasks and stream partitions where operator subtask other units of parallelism
*  Operator subtasks will be assigned to task slots within your TaskManager in order to make use of the resources within that task slot
*  Every task slot will have dedicated memory resources that are assigned to the operator that runs within that slot

<br/> ![](JobManagerAndTaskManager/8.PNG)

* Let's bring together everything that we've discussed so far and see the anatomy of a Flink cluster
* Here is a Flink cluster on three different machines
* One machine runs the JobManager
* We have two separate machines running TaskManagers
* The Flink client is not part of the runtime and program execution, but is used to prepare and send a data flow to the JobManager
* After that, the client can disconnect, which means it runs in detached mode, or it can stay connected to receive progress reports
* That is attached mode
* The JobManager receives the JobGraph, which it converts to an ExecutionGraph for parallelized data flow
* It then schedules the application on TaskManagers
* TaskManagers connect themselves to the JobManager in the cluster and make themselves available for the execution of processing code

# Task Lifecycle and Operator Lifecycle

<br/> ![](TaskLifecyleAndOperatorLifecycle/1.PNG)

* We've seen that any Flink application can be considered to be a directed acyclic graph where operators act on streams. 
* This job graph depiction of a Flink application is converted to an execution graph where every vertex represents parallel execution. 
* Operators are made up of operator subtasks that perform processing in parallel. 
* Streams are subdivided into stream partitions, subsets of data flows. 

<br/> ![](TaskLifecyleAndOperatorLifecycle/3.PNG)
* A task in Flink is the basic unit of execution
* This task is run within a task slot
* A task can be thought of as a parallel instance of an operator that is executed to process your input data
* So if an operator has a parallelism equal to 5, five tasks will be instantiated and run within different task slots of the TaskManager
* The base task for all tasks executed in Flink is the StreamTask

<br/> ![](TaskLifecyleAndOperatorLifecycle/4.PNG)
* The task is an instance of the operator, which means the lifecycle of a task is tightly coupled with the lifecycle of an operator
* Before a task can execute, the initializeState method is invoked to set up the initial state of the task before it's run
* SnapshotState method is invoked to asynchronously checkpoint the state of that particular task

<br/> ![](TaskLifecyleAndOperatorLifecycle/5.PNG)

* Consecutive operators that are run within a task are open from last to first. 
* Consecutive operators in a task are closed from first to last.