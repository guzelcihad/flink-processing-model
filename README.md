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

# Flink Clusters and Deployment

<br/> ![](FlinkClustersAndDeployments/1.PNG)

* There are three types off Flink clusters that you can set up
*  You can set up a Flink session cluster, a Flink job cluster or a Flink application cluster

<br/> ![](FlinkClustersAndDeployments/2.PNG)

*  The session cluster is a long running, pre‑existing cluster
*  The cluster remains up and running and can accept multiple jobs submissions
*  The cluster continues to remain alive after the jobs have completed execution
*  The lifetime of a Flink session cluster is thus not bound to the lifetime of any particular Flink job
*  When you use a session cluster, TaskManager slots are allocated by the resource manager on job submission and released once the job is complete
*  Because all the jobs share the same cluster, the jobs will compete for resources
*  Also, if a JobManager or TaskManager crashes, all jobs using that JobManager or TaskManager will fail

<br/> ![](FlinkClustersAndDeployments/3.PNG)

*  In previous versions of Flink, the Flink session cluster used to be called the Flink Cluster in session mode, so if you see references to this, it's the same thing as the Flink session cluster
*  The main use case for the Flink session cluster is to run interactive queries, which are short duration queries, short running queries
*  Having a pre‑existing cluster in such condition saves a considerable amount of time when you apply for resources and start TaskManagers
*  In the case of interactive short queries, it's important that jobs can quickly perform computations using existing resources
*  Resources don't have to be spun up explicitly for the job

<br/> ![](FlinkClustersAndDeployments/4.PNG)

*  A Flink job cluster, as its name suggests, is a cluster that is specifically created for a particular job that you want to execute
*  This job cluster will be spun up by a cluster manager such as YARN or Kubernetes
*  Every submitted job will spin up a new instance of a Flink job cluster, and the entire cluster is then available for that job alone
*  All cluster resources are utilized by that one job
*  The cluster will be torn down once the job completes execution
*  A major advantage of the job cluster is that jobs running on this cluster are isolated from other jobs
*  A crash in the JobManager or TaskManager affects only the one job

<br/> ![](FlinkClustersAndDeployments/5.PNG)

*  The Flink job cluster in earlier versions used to be referred to
*  as the Flink Cluster in job or per‑job mode
*  The main use case for the job cluster is for very long running jobs, where jobs can run for hours or even days
*  Such long running jobs have very high stability requirements and such jobs are typically not very sensitive to longer startup times
*  So even if it takes time to provision startup resources from scratch, that's totally fine

<br/> ![](FlinkClustersAndDeployments/6.PNG)

*  The third kind of Flink cluster is the Flink application cluster
*  Here you have a single cluster, a dedicated cluster for one Flink application that you need to execute
*  Here, the main entry point of your Flink code will not be executed on your client, rather, it will be executed on the cluster itself
*  The lifetime of the cluster is closely tied to the lifetime of the application
*  The cluster is no longer needed if the application doesn't run
*  In this case, the lifetime of the Flink application cluster is bound to the lifetime of the application
*  This cluster offers the best separation of concerns because all of the resources of the cluster are dedicated to a single application
*  The ResourceManager and the Dispatcher are scoped to the one application that runs on this cluster

<br/> ![](FlinkClustersAndDeployments/7.PNG)

*  Corresponding to each type of cluster, Flink applications can be deployed in three different modes
*  In the session mode, the application assumes an already running cluster and uses the resources of the cluster to execute the job
*  Applications executed in the same session cluster compete for the same resources
*  A crashing application can affect other applications on that cluster
*  An alternative is to deploy a Flink application in the per‑job mode
*  This is aimed at providing better resource isolation guarantees
*  The per‑job more uses the available cluster manager framework, either YARN or Kubernetes, to spin up a separate cluster for every submitted job, and finally, Flink applications can be deployed in application mode, where we have a long running cluster specifically scoped to that application
*  The application mode creates a cluster per submitted application, but the main method of the application is executed on the JobManager of the cluster
*  Apache Flink comes with first class support for a number of common deployment targets

<br/> ![](FlinkClustersAndDeployments/8.PNG)

*  You can deploy locally, that is, run Flink locally for basic testing and experimentation
*  You can deploy Flink on a standalone cluster, on bare‑metal or virtual machines
*  You can deploy Flink on top of Apache Hadoop's resource manager, YARN
*  You can use Docker to run Flink within a containerized environment
*  You can use Kubernetes, which offers an automated system for deploying containerized applications, or you can use Mesos, a generic resource manager for running distributed systems

# Job Manager High Availability

<br/> ![](JobManagerHighAvailability/1.PNG)

* A regular Flink cluster has exactly one JobManager and can have multiple TaskManagers
*  It's possible for you to configure the JobManager of a Flink cluster to work in high‑availability mode
*  Let's discuss how
*  The default configuration of a Flink cluster is to use a single JobManager
*  When you have just one JobManager, that becomes a single point of failure, or an SPOF
*  The JobManager is responsible for scheduling all your Flink applications on that cluster
*  So if the JobManager crashes, no new applications can be submitted
*  Any running program on the cluster will fail as well
*  You can configure your Flink cluster in high‑availability mode to mitigate this
*  In high‑availability mode, you will have more than one JobManager running

<br/> ![](JobManagerHighAvailability/2.PNG)

*  The high‑availability configuration for your JobManager will be a little different if you're working with a standalone cluster or if you're working with a YARN cluster
*  Let's talk about the standalone cluster first

<br/> ![](JobManagerHighAvailability/3.PNG)

*  In a standalone cluster, which runs on bare metal machines or on virtual machines, you can have multiple JobManager processes running
*  Only one of these JobManagers will be the leader at any point in time
*  You can have multiple standby JobManagers, which can take over if the leader fails
*  So if for some reason the leader JobManager crashes, any of the other JobManagers can take on the leader role
*  There is no explicit distinction between standby JobManager instances and the leader JobManager instance

<br/> ![](JobManagerHighAvailability/4.PNG)

*  Imagine that you have a cluster with three JobManager instances configured
*  One of these JobManagers will be elected the leader
*  The cluster functions fine with one leader and multiple JobManager standbys
*  At some point, it's possible that the node on which the JobManager runs crashes
*  The JobManager leader no longer exists, which means one of the standby JobManagers have to take on the leader role
*  One of the JobManagers will be elected the leader while the original JobManager leader is recovering
*  Even after the original leader has completely recovered, the new leader continues to lead the cluster, and the recovered JobManager becomes the second standby

<br/> ![](JobManagerHighAvailability/5.PNG)

*  The high‑availability configuration mode in a standalone cluster requires the use of ZooKeeper
*  ZooKeeper is what we use to elect the leader JobManager
*  You need to configure the quorum for the ZooKeeper service so that the quorum can then elect a new leader if one is needed
*  You then set up your masters file with all of the JobManager hosts and their web UI ports

<br/> ![](JobManagerHighAvailability/6.PNG)

*  If you're running a high‑availability Apache Flink cluster using the YARN resource manager, you do not need to run multiple JobManager instances and elect one of them the leader
*  You just need a single ApplicationMaster instance, and that is enough
*  The YARN cluster manager will restart this instance in case of failures to ensure high availability

# Flink API

<br/> ![](FlinkAPI/1.PNG)

*  Flink actually offers a number of different APIs at different levels of abstraction
*  All of these APIs interact and integrate with one another very neatly, and you can mix and match these APIs to process your input stream

<br/> ![](FlinkAPI/2.PNG)

*  At the very lowest level, we have the APIs for stateful stream processing
*  This is the lowest level of abstraction, on top of which other high‑level APIs are built
*  This abstraction offers timely stream processing
*  This is basically stateful stream processing where time plays an important role
*  It is these APIs that offer the notion of event time and processing time of streaming entities
*  A discussion of event time and processing time is beyond the scope of this particular beginner course in Flink
*  Stateful stream processing API allows you to manage state and runtime context
*  These APIs provide consistent, fault‑tolerant state, allowing programs to realize very sophisticated computations

<br/> ![](FlinkAPI/3.PNG)

*  The data stream API that we've been working with so far is a higher level abstraction built on top of stateful stream processing functions
*  Data streams are used to work with streaming data
*  There is a corresponding dataset API that you can use to work with batch data for batch processing operations
*  These APIs are core APIs to work with both unbounded as well as bounded data
*  Both the data stream and dataset API contain common building blocks for data processing
*  Data stream and dataset APIs are what we would use to define user‑specified transformations on input data
*  These can be used to perform join operations on two or more streams
*  These can do be used to perform aggregation computations on input data, as well as perform windowing operations

<br/> ![](FlinkAPI/4.PNG)

*  Flink also offers other higher‑level abstractions to process data
*  The table API is a declarative domain specific language centered around the concept of data stored in tables, that is relational data
*  Tables can be defined over batch data, as well as streaming data
*  Tables defined over streams are dynamic tables
*  That's because the contents of these tables can change dynamically
*  The table API follows the relational model of processing data
*  The APIs are intuitive, easy to use, and self‑explanatory
*  You can perform common table‑specific operations such as selection, projection, join operations, group‑by queries, and aggregations
*  The table APIs declaratively define what logical operation should be performed, rather than specifying exactly how the code for the operation looks
*  All programs using the table API go through an optimizer that applies optimization rules before execution

<br/> ![](FlinkAPI/5.PNG)

*  The highest level API for data processing operations is the use of SQL queries on your input streams
*  The operations that you can perform using SQL queries are the same operations that you can perform using the table API
*  So SQL is similar to the table API in semantics and expressiveness
*  But for the many developers who are familiar with using SQL queries to access and aggregate data, these APIs may prove to be more intuitive
*  With SQL queries, you represent programs as SQL query expressions
*  These expressions can then be executed over tables, which have been defined using the table API

# Fault Tolerance Guarantees

<br/> ![](FaultToleranceGuarantees/1.PNG)

*  There are three types of guarantees that stream processing applications can give us; at most once, at least once, exactly once
*  The semantics of streaming systems are captured in terms of how many times each record can be processed by the system
*  At most once means that each record will be processed once or not processed at all
*  At least once, each record will be processed one or more times
*  This is stronger than at most once, because it ensures that no data will be lost, but there might be duplicates, that one record may be processed more than once
*  And the strongest of all guarantees is the exactly once guarantee

<br/> ![](FaultToleranceGuarantees/2.PNG)

*  Each record will be processed exactly once
*  No data will be lost
*  No data will be processed multiple times
*  For a streaming application to be able to provide an exactly once guarantee, every step in the stream processing system must provide the same exactly once guarantee

<br/> ![](FaultToleranceGuarantees/3.PNG)

*  Stream processing is performed on distributed computing environments
*  This means that there can be job failures or node failures
*  To ensure recovery from job crashes, the input sources that you use for your streaming data should be replayable
*  Replayable streams essentially mean that if data is lost during processing, that data can be replayed from the source and processed again

<br/> ![](FaultToleranceGuarantees/4.PNG)

*  Now Apache Flink can guarantee exactly‑once fault tolerant semantics under certain conditions
*  Exactly‑once guarantees are possible when the source participates in the snapshotting mechanism
*  That is the state of the source is snapshotted and backed up somewhere periodically
*  In addition to exactly once state semantics to guarantee end‑to‑end exactly once record delivery, the data sink that you use with Apache Flink needs to take part in the checkpointing mechanism
*  Checkpointing is what Flink programs can use to periodically back up the state of their stream processing system
*  So in case of failures, data can be restored and state can be restored from the checkpoint
*  You might source your streaming data from different connectors

<br/> ![](FaultToleranceGuarantees/5.PNG)

*  Your source might be Apache Kafka, Kinesis Streams, collections, files, Rabbit MQ, and so on
*  Here are the fault tolerance guarantees for common sources that you would use with Flink programs
*  You can see that Kafka and Kinesis offer exactly‑once guarantees
*  With Sockets and the Twitter streaming API, you have the at‑most‑once guarantee
*  And with Google's PubSub messaging system, the guarantee is at least once

<br/> ![](FaultToleranceGuarantees/6.PNG)

*  Similarly, here are the fault tolerance guarantees for Flink with different types of sinks
*  You have exactly‑once guarantees using HDFS and file sinks and Kafka, and at‑least‑once guarantees using other sinks such as ElasticSearch

# Keyed Streams

<br/> ![](KeyedStreams/1.PNG)

* Stateful transformations in Apache Flink are often performed on keyed streams, so before we go on to discussing state, let's understand the idea of keyed streams first
*  Imagine that you have a stream of records coming in from some kind of streaming source
*  All of these records are numbers
*  Let's assume that these are students' scores out of 100 in the various courses that a student has bean enrolled in

<br/> ![](KeyedStreams/2.PNG)

*  Just having these scores is not very meaningful
*  You want additional information about who scored these scores and in which subject maybe, so such a stream makes much more sense where every record has a name field as well
*  In our example here, let's assume that the input stream comprises of records with the name of the student and the score for that particular student in some course
*  If a student has enrolled in multiple courses, the name of that student appears in multiple records
*  Now, if I were to ask you a question like this one, what is the highest score for Jane across all of her classes? What you would do is basically look at the incoming records, perform some kind of grouping action or collection action by the name of the student, and then calculate the highest score

<br/> ![](KeyedStreams/3.PNG)

*  We've collected all scores for Jane together, and then once we've done this, we've identified that the highest score for Jane is 94
*  In essence, what we've done here is created a keyed stream where all entities which have the same key are sent to the same process
*  One of the most important details to realize in a keyed stream in Apache Flink is that the keys are not physical keys, they are virtual keys

<br/> ![](KeyedStreams/4.PNG)

*  Keys are a way to virtually partition the incoming stream so all entities with the same key are processed together
*  So keyed streams can be thought of as a logical grouping of the underlying keys
*  This logical grouping is done via virtual keys and not physical keys
*  You're not physically extracting the key and separating it from the rest of the record, instead, you're physically tagging a field or a combination of fields as a key
*  The way you specify a key is using a grouping operator called the keyBy, to which you pass in a key selector
*  The key selector can perform any kind of computation that identifies a key for that record
*  Now the key can be of any type, it can even be objects, so long as the way you figure out the key for a record is deterministic

<br/> ![](KeyedStreams/5.PNG)

*  The keyBy operation is performed on streams in your Flink application
*  When you perform the keyBy, Flink will partition a single stream into multiple independent output streams
*  Each of these output streams will have those elements which contain the same key
*  These partitions streams that contain entities which have the same key, are then sent to the same operator for processing
*  When the operator is a stateful one, that is it maintains state, that operator only has access to elements with the same key
*  Access to state within an operator is on a per‑key basis

<br/> ![](KeyedStreams/6.PNG)

*  Here is a visualization of how keyed streams work
*  Notice that we have two source operators running in parallel and two instances of a stateful operator
*  Let's break this down to see what's happening
*  The two source operators run in parallel, and they receive records with different values for key

<br/> ![](KeyedStreams/7.PNG)

*  All of the records with the key A, B, or Y are sent to the same instance of the next stateful operator
*  Instance 1 of the stateful operator next in sequence will only receive records for A, B, and Y, and will not receive other records

<br/> ![](KeyedStreams/8.PNG)

*  Similarly, instance 2 of the stateful operator will only receive records with the key D, E and Z, and it will not receive other records
*  Thus, you can see that the input stream from the source operator is partitioned by key before it's sent along to the next stateful operator in sequence
*  Each instance of the stateful operator has an embedded key value stored, which stores the state associated with each key

# States in Flink

<br/> ![](StatesInFlink/1.PNG)

* So what are the different kinds of state that you can work with when you're using Flink for your data processing operations? We've already discussed that stateful transformations in Flink accumulate information across entities in a stream
*  Stateful transformations can be performed on a per‑key basis
*  We've discussed keyed streams in detail in the previous clip
*  Stateful transformations can also accumulate entities over an interval, such as a window
*  Flink applications maintain two broad types of state
*  We have operator state where state is maintained on a per‑operator basis
*  Every stateful operator that performs processing on your input stream will maintain its own state
*  State in Flink can also be stored on a per‑key basis
*  Such state is referred to as the keyed state

<br/> ![](StatesInFlink/2.PNG)

*  When you write code for a Flink application, typically you won't be using operator state at all
*  Operator state is a special kind of state that's associated with source and sink operators
*  So when you specify your own source and sink implementations to connect Flink programs to different data repositories, that's when you'll have to deal with operator state
*  So this is a fairly advanced use case
*  The operator state is also useful in scenarios where streams do not have a key that you can use to partition state
*  When you work with Flink, you'll rarely use operator state directly
*  You're most likely going to be working with keyed state

<br/> ![](StatesInFlink/3.PNG)

*  Operator state is state based on parallel instances of an operator
*  We've seen that the basic components of a Flink application are operators and streams
*  Parallel instances of an operator are referred to as operator sub tasks
*  So you can have a state based on these parallel instances, the operators can be source connectors or transformations that you apply on streaming data
*  Flink partitions the data and then runs the operator in parallel
*  Every instance of the operator is associated with some operator state

<br/> ![](StatesInFlink/4.PNG)

*  Flink also supports something known as the broadcast state, which is a special type of operator state
*  This is where records of one stream will be broadcast to all downstream tasks, which need access to those records
*  Let's say you have one stream of records called stream A, and you know that tasks downstream need access to records in stream A while they're processing stream B
*  So state can be accessed while processing records of a second stream B
*  Stream A can be broadcast so that downstream operators have access to it while processing stream B

<br/> ![](StatesInFlink/5.PNG)

*  When you're working with keyed streams, Flink stores state across all elements, the state associated with the same key
*  This is keyed state
*  Keyed state is maintained in what can be thought of as an embedded key value store
*  The state is partitioned and distributed strictly together with the streams that are read by the stateful operators
*  So access to the key value state is only possible on keyed streams

<br/> ![](StatesInFlink/6.PNG)

*  Another kind of state that you'll see discussed in Flink is the queryable state
*  This is a feature, a relatively new feature, that allows client APIs to query job state from outside of Flink
*  As of version v1
* 11 in Flink, this feature is currently in beta, and it's quite possible that breaking API changes will be present in subsequent versions

# Keyed State Interfaces And Rich Functions

<br/> ![](KeyedStateInterfacesAndRichFunctions/1.PNG)

* When you're writing your processing code, Flink has primitives that gives you access to the various types of state scoped to the current key
*  Let's take a look at some of these keyed state interfaces
*  When you're working with keyed streams, you can access the ValueState, the ListState, the ReducingState, the AggregatingState, and the MapState
*  All of these state objects are interfaces that gives you access to the state that is scoped to the key of the current input element
*  Each of the state primitives that we just saw can be instantiated within a processing function, and every state is scoped to a key
*  That means for each key that you process, a different state object will be instantiated
*  The simplest state primitive is the ValueState
*  As you can see, the ValueState is a generic interface
*  It has the generic type parameter, <T>
*  This essentially means that the ValueState can hold the value of any data type
*  During your processing, you can access the value held within the ValueState
*  You can update the value as needed
*  Another state primitive that you can use to hold state is the ListState
*  As its name suggests, it can hold a list of elements where the elements can be of any type specified by the generic type parameter
*  When new elements come in, which you believe should be part of this ListState, you can append new elements to the list
*  You can also access an iterable of elements contained in the list to perform some kind of aggregation operation
*  If you want to hold some kind of aggregate state, the ReducingState state primitive is one that you would choose
*  The ReducingState holds only a single value at a time, but that value is the result of an aggregation over all of the input elements that you've seen
*  The aggregation can be anything that you specify, the sum of all of the elements seen so far, the count of elements seen so far, the average of elements seen so far, anything
*  When you use the ReducingState, the value that is stored within the ReducingState should have the same type as the input elements
*  So if the input elements that you're viewing are of type integer, the ReducingState should have data type integer
*  The ReducingState does not hold the individual elements which make up the aggregation
*  Rather, the final aggregate result is stored at any point in time, so elements added to the state are reduced to the aggregate and stored
*  The AggregatingState works exactly like the ReducingState does, but is a little more flexible
*  It holds a single value of any type, and the value is the result of an aggregation over input elements
*  The AggregatingState is different from the ReducingState in the fact that the value stored within the AggregatingState can be of a different type from the input elements that you use to calculate the aggregate
*  Let's say the aggregate value that you're storing is the average
*  The average can be of type double
*  The input elements that you examine can be of type integer
*  This is not possible in the ReducingState
*  Just like in the ReducingState, elements added to the state are reduced to the aggregate
*  Individual elements are not stored
*  And finally, another state primitive that you can use with keyed streams is the MapState
*  It simply keeps a list of mappings from key to value
*  At any point in time within your processing function, you can access an iterable of currently stored mappings

<br/> ![](KeyedStateInterfacesAndRichFunctions/2.PNG)

*  All of the state objects that we've discussed so far are what you use to access and update state as you perform processing on the input stream These are interfaces that allow you to programmatically interact with state
*  The actual state can be stored externally in some kind of persistent store
*  In order to get a handle to access state, you need to define state descriptors, which describe the kind of state you wish to store and access
*  Every Flink application has a RuntimeContext
*  That is the environment in which that Flink application is executed
*  State is accessed using this RuntimeContext, and this RuntimeContext is not available in all processing functions that you work within Flink
*  It's specifically available in RichFunctions

<br/> ![](KeyedStateInterfacesAndRichFunctions/3.PNG)

*  All of the transformations that we've performed on input streams so far involve the use of specific functional interfaces
*  All of these same transformations can be represented using RichFunctions as well
*  In addition to the method that we override to perform the actual processing of the input data, RichFunctions provide additional methods for parameterizing the function, managing state from within the function, giving us access to state handles to update and retrieve state, accessing runtime information about the Flink job, and accessing any broadcast variables that may be available
*  In summary, when you work with state objects in Flink, you'll use RichFunctions to define your processing rather than regular functions