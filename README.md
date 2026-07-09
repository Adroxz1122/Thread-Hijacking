# Classic Thread Hijacking

proof-of-concept demonstrating **classic thread hijacking** on Windows. A suspended thread is created with a dummy function, its execution context is redirected to position‑independent shellcode, and the thread is resumed to execute the payload.

## Overview

This project illustrates how to:

1. Create a thread in a suspended state.
2. Allocate memory for shellcode inside the current process.
3. Copy the shellcode and mark it executable.
4. Modify the suspended thread’s instruction pointer (`Rip`) to point to the shellcode.
5. Resume the thread, causing the shellcode to execute in place of the original dummy function.

The technique is often used for process injection, but here it runs entirely within its own process – ideal for learning and controlled testing.

## How It Works

1. **Suspended Thread**  
   `CreateThread` is called with the `CREATE_SUSPENDED` flag, targeting a harmless function (`dummyFunc`) that simply loops forever.

2. **Memory Allocation**  
   `VirtualAlloc` reserves and commits a region of memory with `PAGE_READWRITE` permissions. The shellcode is copied into this region.

3. **Permission Change**  
   `VirtualProtect` flips the memory to `PAGE_EXECUTE_READ`, making the shellcode executable.

4. **Context Manipulation**  
   `GetThreadContext` captures the current CPU state of the suspended thread. The instruction pointer (`Rip`) is overwritten with the address of the shellcode. `SetThreadContext` applies the modified context.

5. **Execution**  
   `ResumeThread` starts the hijacked thread, which now jumps directly to the shellcode.

***THIS IS STILL WORK-IN-PROGRESS, AND WILL GET ATLEAST 3 MORE MODULES FOR HIJACKING, AND MAYBE SOME MODULES FOR ENCRYPTION AND STUFF***
