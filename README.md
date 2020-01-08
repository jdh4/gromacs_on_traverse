# Gromacs on Traverse

The problem is that the performance is less on Traverse than TigerGPU or the Adroit node.

We can make the performance of 1 GPU and several CPU-cores better on Traverse than the other choices but in a variable way
where the performance varies from one run to the next.

We will focus on the case of 1 CPU-core and 1 GPU.

   * Gromacs fails a hardware test on Traverse involving NUMA.
   * The output shows the number of M-flops is different between the Adroit and Traverse cases.
   * Gromacs complains about the threads not being pinned.
   * One thing to check is that the GPU and cpu-core are on the same socket. And can we force them to be on different sockets
   to measure the penalty.
   * One expects that the solution will not come from recompiling Gromacs in some way or modifying the source code.
   * The hope is that can be solved via the Slurm script.
   * Maybe a version of Gromacs specifically for POWERPC will become available.
   * How are cpu-cores indices related to the GPU indices?
   * Why aren't other institutions having this problem?
   * Are other codes suffering from this problem?
   * How do we know that when trying to use 1 thread per physical core that that is what is happening?
   * Looks like KMP_AFFINITY is the analog of OMP_PLACES for Intel
   
## Simple OpenMP Code
   
```
#!/bin/bash
#SBATCH --reservation=test
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=31
#SBATCH --mem=1G
#SBATCH --time=00:00:30
#SBATCH --nodelist=traverse-k01g2

taskset -c -p $$
numactl -s
env | grep SLURM | sort
hostname

export OMP_NUM_THREADS=3
export OMP_PLACES="{33,34,59}"
export OMP_DISPLAY_ENV=TRUE
echo $OMP_NUM_THREADS

srun -vvv ./hw_omp
```

The following is the output:

```
pid 130079's current affinity list: 32-63
policy: default
preferred node: current
physcpubind: 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63 
cpubind: 0 
nodebind: 0 
membind: 0 8 252 253 254 255 
SLURM_CLUSTER_NAME=traverse
SLURM_CPUS_ON_NODE=32
SLURM_CPUS_PER_TASK=31
SLURMD_NODENAME=traverse-k01g2
SLURM_GTIDS=0
SLURM_JOB_ACCOUNT=cses
SLURM_JOB_CPUS_PER_NODE=32
SLURM_JOB_GID=20121
SLURM_JOB_ID=42761
SLURM_JOBID=42761
SLURM_JOB_NAME=job.slurm
SLURM_JOB_NODELIST=traverse-k01g2
SLURM_JOB_NUM_NODES=1
SLURM_JOB_PARTITION=all
SLURM_JOB_QOS=trav-test
SLURM_JOB_RESERVATION=test
SLURM_JOB_UID=150340
SLURM_JOB_USER=jdh4
SLURM_LOCALID=0
SLURM_MEM_PER_NODE=1024
SLURM_NNODES=1
SLURM_NODE_ALIASES=(null)
SLURM_NODEID=0
SLURM_NODELIST=traverse-k01g2
SLURM_NPROCS=1
SLURM_NTASKS=1
SLURM_PROCID=0
SLURM_SUBMIT_DIR=/home/jdh4/junk_omp_tasket
SLURM_SUBMIT_HOST=traverse.princeton.edu
SLURM_TASK_PID=130079
SLURM_TASKS_PER_NODE=1
SLURM_TOPOLOGY_ADDR_PATTERN=node
SLURM_TOPOLOGY_ADDR=traverse-k01g2
SLURM_WORKING_CLUSTER=traverse:traverse-nfs:6817:8704:109
traverse-k01g2
3
srun: defined options
srun: -------------------- --------------------
srun: (null)              : traverse-k01g2
srun: cpus-per-task       : 31
srun: jobid               : 42761
srun: job-name            : job.slurm
srun: mem                 : 1G
srun: nodes               : 1
srun: ntasks              : 1
srun: verbose             : 3
srun: -------------------- --------------------
srun: end of defined options
srun: debug:  propagating RLIMIT_CPU=18446744073709551615
srun: debug:  propagating RLIMIT_FSIZE=18446744073709551615
srun: debug:  propagating RLIMIT_DATA=18446744073709551615
srun: debug:  propagating RLIMIT_STACK=18446744073709551615
srun: debug:  propagating RLIMIT_CORE=0
srun: debug:  propagating RLIMIT_RSS=1073741824
srun: debug:  propagating RLIMIT_NPROC=4096
srun: debug:  propagating RLIMIT_NOFILE=5120
srun: debug:  propagating RLIMIT_MEMLOCK=18446744073709551615
srun: debug:  propagating RLIMIT_AS=18446744073709551615
srun: debug:  propagating SLURM_PRIO_PROCESS=1
srun: debug:  propagating UMASK=0022
srun: debug2: srun PMI messages to port=33135
srun: debug:  Munge authentication plugin loaded
srun: jobid 42761: nodes(1):`traverse-k01g2', cpu counts: 32(x1)
srun: debug2: creating job with 1 tasks
srun: debug:  requesting job 42761, user 150340, nodes 1 including ((null))
srun: debug:  cpus 31, tasks 1, name hw_omp, relative 65534
srun: CpuBindType=(null type)
srun: debug:  Entering slurm_step_launch
srun: debug:  mpi type = (null)
srun: debug:  Using mpi/none
srun: debug:  Entering _msg_thr_create()
srun: debug:  initialized stdio listening socket, port 46161
srun: debug:  Started IO server thread (35184397906352)
srun: debug:  Entering _launch_tasks
srun: debug2: Called _file_readable
srun: debug2: Called _file_writable
srun: launching 42761.0 on host traverse-k01g2, 1 tasks: 0
srun: debug2: Called _file_writable
srun: route default plugin loaded
srun: debug2: Tree head got back 0 looking for 1
srun: debug2: Tree head got back 1
srun: debug:  launch returned msg_rc=0 err=0 type=8001
srun: debug2: Activity on IO listening socket 12
srun: debug2: Entering io_init_msg_read_from_fd
srun: debug2: Leaving  io_init_msg_read_from_fd
srun: debug2: Entering io_init_msg_validate
srun: debug2: Leaving  io_init_msg_validate
srun: debug2: Validated IO connection from 10.33.25.4, node rank 0, sd=13
srun: debug2: eio_message_socket_accept: got message connection from 10.33.25.4:40520 14
srun: debug2: received task launch
srun: Node traverse-k01g2, 1 tasks started
srun: debug2: Called _file_readable
srun: debug2: Called _file_writable
srun: debug2: Called _file_writable
srun: debug2: Entering _file_read
srun: debug2: Called _file_readable
srun: debug2: Called _file_writable
srun: debug2: Called _file_writable
srun: debug2: Called _file_readable
srun: debug2: Called _file_writable
srun: debug2: Called _file_writable
srun: debug2: Called _file_readable
srun: debug2: Called _file_writable
srun: debug2: Called _file_writable
srun: debug2: Entering _file_write

OPENMP DISPLAY ENVIRONMENT BEGIN
  _OPENMP = '201511'
  OMP_DYNAMIC = 'FALSE'
  OMP_NESTED = 'FALSE'
srun: debug2: Leaving  _file_write
srun: debug2: Called _file_readable
srun: debug2: Called _file_writable
srun: debug2: Called _file_writable
srun: debug2: eio_message_socket_accept: got message connection from 10.33.25.4:40522 14
srun: debug2: received task exit
srun: Received task exit notification for 1 task of step 42761.0 (status=0x0000).
srun: traverse-k01g2: task 0: Completed
srun: First task exited. Terminating job in 60s
srun: debug:  task 0 done
srun: debug2:   false, shutdown
srun: debug2:   false, shutdown
srun: debug2: Called _file_readable
srun: debug2: Called _file_writable
srun: debug2: Called _file_writable
srun: debug2:   false, shutdown
srun: debug2: Called _file_readable
srun: debug2: Called _file_writable
srun: debug2: Called _file_writable
srun: debug2:   false, shutdown
srun: debug2: Entering _file_write
  OMP_NUM_THREADS = '3'
  OMP_SCHEDULE = 'DYNAMIC'
  OMP_PROC_BIND = 'TRUE'
srun: debug2: Leaving  _file_write
srun: debug2: Called _file_readable
srun: debug2: Called _file_writable
srun: debug2: Called _file_writable
srun: debug2:   false, shutdown
srun: debug2: Entering _file_write
  OMP_PLACES = '{33:2,59}'
  OMP_STACKSIZE = '1'
  OMP_WAIT_POLICY = 'PASSIVE'
  OMP_THREAD_LIMIT = '4294967295'
  OMP_MAX_ACTIVE_LEVELS = '2147483647'
  OMP_CANCELLATION = 'FALSE'
  OMP_DEFAULT_DEVICE = '0'
  OMP_MAX_TASK_PRIORITY = '0'
OPENMP DISPLAY ENVIRONMENT END
srun: debug2: Leaving  _file_write
srun: debug2: Called _file_readable
srun: debug2: Called _file_writable
srun: debug2: Called _file_writable
srun: debug2:   false, shutdown
srun: debug2: Entering _file_write
Hello from thread Hello from thread 20 of  of 33

srun: debug2: Leaving  _file_write
srun: debug2: Called _file_readable
srun: debug2: Called _file_writable
srun: debug2: Called _file_writable
srun: debug2:   false, shutdown
srun: debug2: Entering _file_write
Hello from thread 1 of 3
srun: debug2: Leaving  _file_write
srun: debug2: Called _file_readable
srun: debug2: Called _file_writable
srun: debug2: Called _file_writable
srun: debug2:   false, shutdown
srun: debug2: Called _file_readable
srun: debug2: Called _file_writable
srun: debug2: Called _file_writable
srun: debug2:   false, shutdown
srun: debug:  IO thread exiting
```

In the above case it was known that another user was using the first 32 codes. When the OMP_PLACES was set to "{11,34,159}" is seemed to silently ignore 11 and use `OMP_PLACES = '{34,59}'`. When it was set to "{11,34,159}" the `libgomp: Logical CPU number 159 out of range; libgomp: Invalid value for environment variable OMP_PLACES`.
