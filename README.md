In this lab you'll build a MapReduce system.

You'll implement a worker process that calls application Map and Reduce functions and handles reading and writing files, and a coordinator process that hands out tasks to workers and copes with failed workers.

Code Design and Implementation
The design and implementation of the code are mainly divided into three parts:

RPC communication between Coordinator and Worker, corresponding to mr/rpc.go file
Coordinator scheduling logic, corresponding to mr/coordinator.go file
Worker calculation logic, corresponding to mr/worker.go file

RPC communication
There are two main communication between Coordinator and Worker:

The Worker initiates a Task request to the Coordinator when it is idle, and the Coordinator responds with a Task assigned to the Worker
Worker reports to Coordinator after the previous Task has finished running

Coordinator
The logic of the Coordinator component is relatively complicated due to the scheduling of the entire MR job and the processing of Worker Failover.

First, the Coordinator needs to maintain the following state information:

Basic configuration information, including the total number of MAP Tasks and the total number of Reduce Tasks
Information required for scheduling, including
Current stage, MAP or REDUCE
All unfinished Tasks and their Workers and Deadlines (if any), implemented using the Golang Map structure
All unallocated Task pools are used to respond to Worker's application and reallocation during Failover, implemented using Golang Channel

Then, the Coordinator needs to implement the following processes:

At startup, generate a MAP Task based on the specified input file into the available Task pool
Process the Worker's Task application RPC, allocate an available Task from the pool to the Worker and respond
Process the Worker's Task completion notification and complete the Task's final result data Commit
After all the MAP Tasks are completed, transfer to the REDUCE stage and generate the REDUCE Task to the available Task pool
After the REDUCE Task is all completed, mark the MR job as completed and exit
Periodically inspect the running Task, and reassign it to a new Worker after it is found that the running time exceeds 10s.


worker
The core logic of Worker is relatively simple, mainly an infinite loop, which continuously calls ApplyForTask RPC to Coordinator:

Coordinator returns an empty response, indicating that the MR job has been completed, then exit the loop and end the Worker process
Coordinator returns MAP Task, then
Read the content of the corresponding input file
Pass it to the Map function specified by MR APP to get the corresponding intermediate result
Buckets are divided according to the Hash value of the intermediate result Key, and saved to the intermediate result file
Coordinator returns REDUCE Task, then
Read all the intermediate result file data belonging to the REDUCE Task
Sort all intermediate results and merge by Key value
Pass the merged data to the REDUCE function specified by MR APP to get the final result
write out to result file.



