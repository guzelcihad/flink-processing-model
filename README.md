# Operators Subtask And Stream Partitions

<br/> ![Screenshot](OperatorsSubtaskAndStreamPartitions\1.PNG)

* Here is a big picture overview of how operators that you define make up a directed‑acyclic graph transforming your input stream till the final results are written out using a sink. 

* The input stream is read in using a source operator, and final results are written out using a sink operator. 
* In between the source and the sink, you can have multiple transformation operators which work on your data.


<br/> ![Screenshot](OperatorsSubtaskAndStreamPartitions\2.PNG)

* Flink programs are inherently parallel and distributed, which means that the operators that operate on streaming dataflows are spread across multiple nodes in your Flink cluster. 

* Data transformations run on subsets of streaming data, which means the streams within your Flink applications are composed of stream partitions. 

* Any operator performing transformation can be composed of operator subtasks. 
* Stream partitions and operator subtasks together are responsible for the distributed and parallel execution of a Flink application.


<br/> ![Screenshot](OperatorsSubtaskAndStreamPartitions\3.PNG)
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

<br/> ![Screenshot](OperatorsSubtaskAndStreamPartitions\4.PNG)
* Every operator can be split into subtasks to process your input stream, and these subtasks run completely independently of one another. * These subtasks may be executed in different threads. 
* It's also possible for these subtasks to run on different machines or on different containers within a cluster. 
* The parallel execution of your dataflow operations is possible because of these subtasks. 
* The number of subtasks refer to the parallelism of the individual operator.

<br/> ![Screenshot](OperatorsSubtaskAndStreamPartitions\5.PNG)
* Operators which make up the notes in our directed‑acyclic graph perform the actual transformation of input data. 
* The edges in our directed‑acyclic graph are responsible for transporting data between operators. 
* Edges are referred to as streams. 
* Now, transporting data between operators can be done in two ways. 
* You can have the forwarding pattern, which is a 1:1 pattern which preserves the partitioning and order of elements in the stream. 
* The elements in the stream partition are not shuffled or reordered but passed along as is. On the other hand, data can be transported using the redistributing pattern. 
* This is where every operator subtask sends data to different target subtasks. The original order of the data is not reserved.

<br/> ![Screenshot](OperatorsSubtaskAndStreamPartitions\6.PNG)
* If you look back at our example here, you can see the forwarding pattern in action. 
* The stream partitions that move data between the two operator subtasks for the source and the map operator subtask transport data using the forwarding pattern.

<br/> ![Screenshot](OperatorsSubtaskAndStreamPartitions\7.PNG)
* On the other hand, when data is transported between the map operator subtask and the keyBy window subtask, you can see that the data is redistributed. 
* The original stream partitions have changed, which means one operator subtask can send data to several target subtasks. 
* This is the redistributing pattern in action.