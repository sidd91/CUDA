Shared memory uses three instructions to exchange data between threads
1. wriing data to shared mem
2. synchronizing 
3. reading the data back from shared memory 

Kepler introduced shuffle instruction to allow a thread to read a register from another thread in the same warp.

There are four instructions:
1. int __shfl_down(int var, unsigned delta, int width = warpSIze)
it calculates a source lane ID by adding delta to the caller's lane ID 
2. int __shfl_up() 

__syncthreads() is a barrier primitive designed to protect you from read-after-write memory race conditions within a block.



The rules of use are pretty simple:



1. Put a __syncthreads() after the write and before the read when there is a possibility of a thread reading a memory location that another thread has written to.



2. __syncthreads() is only a barrier within a block, so it cannot protect you from read-after-write race conditions in global memory unless the only possible conflict is between threads in the same block. __syncthreads() is pretty much always used to protect shared memory read-after-write.



3. Do not use a __syncthreads() call in a branch or a loop until you are sure every single thread will reach the same __syncthreads() call. This can sometimes require that you break your if-blocks into several pieces to put __syncthread() calls at the top-level where all threads (including those which failed the if predicate) will execute them.



4. When looking for read-after-write situations in loops, it helps to unroll the loop in your head when figuring out where to put __syncthread() calls. For example, you often need an extra __syncthreads() call at the end of the loop if there are reads and writes from different threads to the same shared memory location in the loop.



5. __syncthreads() does not mark a critical section, so don't use it like that.



6. Do not put a __syncthreads() at the end of a kernel call. There's no need for it.



7. Many kernels do not need __syncthreads() at all because two different threads never access the same memory location. 


# Block level understanding

* Understanding the mapping of the block threads
  *  A block in CUDA always executes on a single SM
  *  The threads in a block can execute concurrently 

* In case of 10SMs with 4 thread blocks
  1. will they be allotted 4 out of 10 SMs?? is it fixed 4 or not??

You will likely see these 4 thread blocks running on 4 SMs. But don't make any assumptions about which SMs the block scheduler picks.

 2. Suppose i want to have 32 threaded blocks running on 10 SMs , how they will be scheduled ??

 a. This is not documented, and may vary depending on GPU architecture (generation)

 b. It also depends a lot on the the achievable occupancy of your kernel. It may be that one SM is capable of executing 2, 3, 4 or more blocks simultaneously - given the constraints of the registers per thread, shared memory requested and number of texture units used by each kernel allow for it. In this case you may see that all 32 thread blocks execute concurrently.  
