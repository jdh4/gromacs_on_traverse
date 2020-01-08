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
