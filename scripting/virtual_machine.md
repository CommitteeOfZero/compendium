# The virtual machine

All numbers are *32-bit integers*, unless otherwise noted.

## Script buffers

Not the entire script body is loaded at once. Instead, currently required scripts are loaded into **script buffers** (of which there are 16 in *Steins;Gate 0*). The runtime will start by loading the **startup script** (e.g. `_Startup_win.scx`); any further required scripts are loaded with the *LoadScript* instruction at runtime (TODO link). This is *not an implementation detail*; a reimplementation *must* follow this behaviour, as *far jump/call instructions*, which reference other script files, refer to **script buffer IDs**, not the files themselves. Judging by *SCS* source code found in *11eyes*, scripts appear to be assigned to script buffers by the programmer manually, not allocated by the compiler.

## Threads

The scripting VM uses **threads** to be able to perform operations while part of the script is **blocked** (usually by waiting for user input). For every frame, the interpreter *must* run each available thread until it relinquishes control (via a blocking instruction (TODO example)), then move on to the next thread, and so on, until each pending thread has been visited once. On the next frame, all threads are unsuspended and their blocking condition is re-evaluated. The interpreter *must not* loop back to the first thread when all threads have been suspended and unsuspend them again immediately, these threads need to wait for the next frame.

A thread includes the following context:

* **Program counter**
* **Script buffer ID**
* **Thread group ID**
* **Instruction-specific registers** (e.g. for loops and switches). These are detailed in the appropriate instruction definitions.
* A **call stack** containing:
  * current **stack depth** (maximum 8 in reference implementation)
  * tuples ([return address ID](/scripting/scx_file_format.md), script buffer ID) as **stack frames**
* **Thread ID** (TODO check if this is *necessary*)
* **Position in thread group** (TODO ? - reference implementation has linked list; is this per thread group?)
* Space for **32 thread-local variables**.

An **important note about thread-local variables:** Thread-local variable accesses are indexed with the start of the thread context structure in the reference implementation, not the start of the thread-local variables array. That begins at `SC3ThreadContext+0xBC` (in bytes), so to access the first actual thread-local variable, a script has to access `ThreadVars[47]`. We **have seen scripts read and write lower indexes** and need to reverse engineer this further. Replicating the precise structure from the reference implementation **may be necessary**:

```C
typedef struct __declspec(align(4))
{
  /* 0000 */  int accumulator;
  /* 0004 */  _BYTE gap4[16];
  /* 0014 */  unsigned int thread_group_id;
  /* 0018 */  unsigned int sleep_timeout;
  /* 001C */  _BYTE gap28[8];
  /* 0024 */  unsigned int loop_counter;
  /* 0028 */  unsigned int loop_target_label_id;
  /* 002C */  unsigned int call_stack_depth;
  /* 0030 */  unsigned int ret_address_ids[8];
  /* 0050 */  unsigned int ret_address_script_buffer_ids[8];
  /* 0070 */  int thread_id;
  /* 0074 */  int script_buffer_id;
  /* 0078 */  _BYTE gap120[68];
  /* 00BC */  int thread_local_variables[32];
  /* 013C */  int somePageNumber;
  /* 0140 */  SC3ThreadContext *next_context;
  /* 0144 */  SC3ThreadContext *prev_context;
  /* 0148 */  SC3ThreadContext *next_free_context;
  /* 014C */  void *pc;
} SC3ThreadContext;
```

Notably, there are indeed **no general-purpose registers**, function-local parameters/variables, return values, a freely accessible stack etc.

In the reference implementation, executing most *undefined* operations causes the offending thread to lock up because the program counter is not changed. As they never relinquish control, this also locks up the game unless they are killed by other threads. Some undefined operations (e.g. some undefined two-byte opcodes) cause out-of-bounds reads that crash the program instead.

## Memory

Scripts have access to the following kinds of memory:

* **32 thread-local variables**, one set per thread, as mentioned above.
* **8,000 global variables**. Similar to thread-local variables, but global across all threads. Some of these variables are only used by scripts, some are only used by the runtime (and thus need not be reimplemented), some are shared. The reference implementation uses an array of 8,000 32-bit integers for this; we recommend doing the same, as some game systems loop over arrays of structures inside this memory.
* **6,400 global flags**. Like global variables, except they're single bits, and thus officially implemented as an 800 byte bitfield.
* **Indexed data**. Scripts can contain local data arrays. These are accessed via the array's [label ID](/scripting/scx_file_format.md) and an element index.

Details on how these accesses are specified can be found in the [expression specification](/scripting/expressions.md).

(TODO link to symbol list based on worklist.txt)