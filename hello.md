I. Multiprocessing systems
II. Symmetric Multiprocessing (SMP)
III. Cluster Systems - parallel decomposition followed by result aggregation

I. Interconnection Networks
II. Tree, Crossbar, Omega and hypercube Network Topologies
III. Routing Mechanisms in Interconnection Network Systems
IV. Contention and Deadlock Problems in Parallel Systems
    a. Contention
    b. Deadlock

          Memory modules
          M0  M1  M2  M3
           |   |   |   |
P0 --------X---X---X---X
P1 --------X---X---X---X
P2 --------X---X---X---X
P3 --------X---X---X---X
           Crosspoints

for a ring with n nodes - D(ring) = N / 2
for square k * k mesh - D(mesh) = 2(k-1)
for a 4 * 4 mesh D = 2(4-1)=6

Input 0 ───┐       ┌─── Output 0
           │  2×2  │
Input 1 ───┘ Switch└─── Output 1

formulaes for Omega Network
number of stages - S = log(2)N
switches per stage - W = N/2
total number of switches - T = N/2(log[2]N)

for ex we have 8 * 8 Omega network
no of stages - 3
switches per stage - 4
total switches - 12

Inputs       Stage 1    Stage 2    Stage 3      Outputs

 0  ──────── [2×2] ──── [2×2] ──── [2×2] ─────── 0
 1  ──────── [2×2] ──── [2×2] ──── [2×2] ─────── 1
 2  ──────── [2×2] ──── [2×2] ──── [2×2] ─────── 2
 3  ──────── [2×2] ──── [2×2] ──── [2×2] ─────── 3
 4  ──────── [2×2] ──── [2×2] ──── [2×2] ─────── 4
 5  ──────── [2×2] ──── [2×2] ──── [2×2] ─────── 5
 6  ──────── [2×2] ──── [2×2] ──── [2×2] ─────── 6
 7  ──────── [2×2] ──── [2×2] ──── [2×2] ─────── 7

General Hypercube formulas

Let n be the number of dimensions.
Number of nodes	- 2^n
Degree of each node - n
Number of links	- (n times 2^{n-1})
Diameter - n
Average path length	- (n/2)

Before recovery:
P1 waits for P2
P2 waits for P1

Recovery:
Abort P2
Release P2's resources

After recovery:
P1 can continue


***CHAPTER 2 BEGINS***
programming using OpenMP

components of OPENMP - compiler directives, runtime library, env vars

include <omp.h>

pragma omp parallel{
#race condition prevention
    pragma omp critical{
        std::cout << "thread number: " omp_get_thread_num() << " ";
    }
}

Fork-join model
Divide -> Fork -> Join
main worker gets the task, it divides into sub workers and waits untill all of them complete it simulataneouly and then bought together

controls number of threads
I. env vars
II. runtime functions (omp_set_num_thread)
III. check thread information

advantages
Easy to learn through simple directives.
Supports incremental parallelization.
Portable across platforms and compilers.
Provides dynamic thread management.

race condition - the result depends on the timing or ordering of concurrent operations.

critical section for race condition

atomic operation

reduction - instead of updating one shared variable openMP gives each thread its private copy and combine results in the end
int counter = 0;

    #pragma omp parallel for reduction(+:counter) num_threads(4)
    for (int i = 0; i < 4000; i++) {
        counter++;
    }

counter = counter0 + counter1 + ......

barrier

sections - different threads do different tasks rather than dividing loop

shared and private variables

int x = 100;
#pragma omp parallel private(x)
{
    x = omp_get_thread_num();
}

suppose We want to calculate the sum of an array
OpenMp way
              Shared Array
           [10, 20, 30, 40]
                   |
          -------------------
          |    |    |    |
         T0   T1   T2   T3

MPI way
       Process 0       Process 1
       [10, 20]        [30, 40]
          |                |
       Local RAM        Local RAM
          |                |
          +---- Network ---+

       Process 2       Process 3
       [ ... ]         [ ... ]


***MPI***
in MPI processes communicate by sending messages

include "mpi.h"
MPI.Init()
    ----- whole block
MPI.Finalize()

basic idea of distributed computing using MPI
Computer 1 stores an array in its RAM.
Computer 2 needs part of that array.
Computer 1 must explicitly send the required data to Computer 2.
Computer 2 receives the data and stores it in its own memory.

Imagine a weather simulation that needs to process atmospheric data for millions of geographical locations.

One computer may take too long to perform the entire simulation.

             Weather Simulation
                     |
          -----------------------
          |          |          |
       Process 0  Process 1  Process 2
       Region A   Region B   Region C
          |          |          |
          -----------+-----------
                     |
              Combine Results
                     |
               Final Output

Process
Rank - process has unique identification number
Communicator - MPI_COMM_WORLD
MPI_Comm_size(MPI_COMM_WORLD, &size);
MPI_Comm_rank(MPI_COMM_WORLD, &rank);

#include <stdio.h>
#include <mpi.h>

int main(int argc, char *argv[]) {

    int rank, size;

    MPI_Init(&argc, &argv);

    MPI_Comm_size(MPI_COMM_WORLD, &size);

    MPI_Comm_rank(MPI_COMM_WORLD, &rank);

    printf("Hello from process %d of %d\n", rank, size);

    MPI_Finalize();

    return 0;
}

point to point communication
can be blocking or non blocking communication

       Process 0                 Process 1
           |                         |
           |------ Send Data ------->|
           |                         |
           |<----- Receive Data -----|
           |                         |

blocking
MPI_Send(&data,
    count,
    datatype,
    destination,
    tag,
    communicator)
MPI_Recv(&data,
    count,
    datatype,
    source,
    tag,
    communicator,
    &status)

non-blocking
MPI_Isend()
MPI_Irecv()
MPI_Test() - checks whether an operation is completed

***Deadlock in MPI***

Process 0                 Process 1

MPI_Send()                MPI_Send()
    |                         |
    v                         v
Waits for receiver       Waits for receiver
    |                         |
    +-------- DEADLOCK -------+


collective communication
Involves a group of processes in a communicator 

MPI_Bcast - Broadcast
MPI_Bcast(
    &data,
    count,
    datatype,
    root,
    communicator
);

MPI_Scatter - Scatter [distributes portion of array]
                P0
         [10, 20, 30, 40]
                  |
          MPI_Scatter()
                  |
       ---------------------
       |      |      |     |
       v      v      v     v
      P0     P1     P2    P3
     [10]   [20]   [30]  [40]

MPI_Scatter(
    sendbuf,
    sendcount,
    sendtype,
    recvbuf,
    recvcount,
    recvtype,
    root,
    communicator
);

MPI_Gather - Gather[collects data from all processors and stores in the root]
       P0       P1       P2       P3
      [10]     [20]     [30]     [40]
        \        |        |       /
         \       |        |      /
                  |
             MPI_Gather()
                  |
                  v
                 P0
        [10, 20, 30, 40]

MPI_Gather(
    sendbuf,
    sendcount,
    sendtype,
    recvbuf,
    recvcount,
    recvtype,
    root,
    communicator
);

MPI_Reduce - Reduce[collects data from multiple processes into one result]

MPI_Reduce(
    sendbuf,
    recvbuf,
    count,
    datatype,
    operation,
    root,
    communicator
);

MPI_Barrier — Barrier
MPI_Barrier(MPI_COMM_WORLD);

***Object Oriented Parallel Programming***

distributed Object model
       Computer A                    Computer B
   +----------------+             +----------------+
   |                |             |                |
   |  Client Object | ----------> |  Server Object |
   |                |  Remote     |                |
   +----------------+  Method     +----------------+
                       Call

A client object runs on Computer A.
A server object runs on Computer B.
The client invokes a method on the server object

Active Object model
Unlike an ordinary passive object, which performs work only when another thread invokes one of its methods, an active object has its own execution activity

       +----------------+
       | Active Object 1|
       |    Thread 1    |
       +----------------+
             /    \
            /      \
           v        v
   +-------------+  +-------------+
   | Active Obj 2|  | Active Obj 3|
   |   Thread 2  |  |   Thread 3  |
   +-------------+  +-------------+

Actor model
independent entities that communicated through message passing
Receives messages.
Processes those messages.
Can create other actors.
Can send messages to other actors.
Can change its own internal state.

Master-Slave Model - master decides and retreive results from slaves
Work Pool Model - availlable workers gets tasks from task pool
Pipeline Model - output of one stage becomes input of another stage 
Data Parallel Model - different data on same process 

what is a good design ??
I. Selection of proper partitioning and mapping techniques
    a. static mapping
    b. dynamic mapping
II. Reducing interaction between processes 

***Election Algorithms***

Bully Algorithms
When process P detects coordinator failure:

    Send ELECTION to every process
    having a higher ID than P.

    Wait for responses.

    If no higher process responds:
        Declare P as coordinator.
        Broadcast COORDINATOR to lower-ID processes.

    Else:
        Wait for a COORDINATOR announcement.

        If no announcement arrives:
            Restart the election.

S = 1 / (1-p) + p/n
S(P) = P - f(P-1)
