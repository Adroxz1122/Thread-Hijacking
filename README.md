# Windows Thread Hijacking Techniques

This repository contains four C programs that demonstrate different approaches to thread hijacking and code injection on Windows. They are intended for educational purposes only – to help security researchers and developers understand the underlying mechanisms and risks.
Overview

Thread hijacking is a technique where an attacker (or security tool) modifies the execution context of an existing thread to run arbitrary code. This can be used for process injection, evasion, or persistence. The four examples cover:

    Local (same process) vs Remote (different process).

    Enumeration (finding a suitable thread) vs Creation (spawning a new thread to hijack).

    Classic hijacking (changing instruction pointer) vs Stack pivoting (allocating a new stack for the payload).

The ThreadCreation codes use msfvenom reverse shell shellcode, while the enumeration one uses the calc.exe.
Files

**1. LocalThreadCreation.c**

   - Purpose: Creates a new suspended thread within the current process and hijacks it to run shellcode.

   - Technique: Allocates memory for shellcode, copies it, sets RIP to the payload address via SetThreadContext, then resumes the thread.

   - Thread source: New thread created with CreateThread (suspended).

   - Hijack: Classic (only changes RIP).

**2. LocalThreadEnumeration.c**

   - Purpose: Enumerates all threads in the current process, picks one that is not the main thread, and hijacks it.

   - Technique: Uses CreateToolhelp32Snapshot to list threads, finds a suitable target, suspends it, allocates a new stack for the payload (stack pivoting), sets RIP and RSP to the new stack, then resumes.

   - Thread source: Existing thread (enumeration).

   - Hijack: Stack pivoting (new stack with return address to ExitThread).

**3. RemoteThreadCreation.c**

   - Purpose: Creates a suspended process (notepad.exe) and hijacks its main thread to run shellcode remotely.

   - Technique: Uses CreateProcess with CREATE_SUSPENDED, allocates memory in the remote process with VirtualAllocEx, writes shellcode, changes RIP via SetThreadContext, then resumes.

   - Thread source: New process's main thread (suspended).

   - Hijack: Classic (only changes RIP).

**4. RemoteThreadEnumeration.c**

   - Purpose: Enumerates threads in the current process (but intended for remote; it uses GetCurrentProcessId for simplicity) and hijacks a thread with stack pivoting.

   - Technique: Same as LocalThreadEnumeration.c but with a function GetRemoteThreadHandle (though it uses the current process ID – could be easily adapted to target another process).

   - Thread source: Existing thread in the same process (demonstrates remote-style enumeration).

   - Hijack: Stack pivoting.

   `Note: RemoteThreadEnumeration.c currently targets the same process; to use it against another process, modify the OwnerProcessID passed to GetRemoteThreadHandle.`

# How It Works (Technical Details)
**Thread Hijacking Basics**

   - Allocate memory for the shellcode (local or remote).

   - Write the shellcode into that memory.

   - Change memory protection to PAGE_EXECUTE_READ (or RWX).

   - Suspend the target thread (if not already suspended).

   - Get its context (CONTEXT structure).

   - Modify RIP (instruction pointer) to point to the shellcode.

        Optionally, also modify RSP to a new stack (stack pivoting) to avoid corrupting the original stack.

   - Set the context.

   - Resume the thread.

**Stack Pivoting**

- In LocalThreadEnumeration.c and RemoteThreadEnumeration.c, a new stack is allocated and RSP is set to the top of that stack. A return address (ExitThread) is pushed so that after the shellcode finishes, the thread exits cleanly.
Remote Process Injection

- RemoteThreadCreation.c uses VirtualAllocEx and WriteProcessMemory to place the shellcode in the target process's address space.
```
Important Notes

    Malicious Use: These techniques are often used by malware. Do not use them for illegal purposes. This code is for learning and defensive research.

    Anti-Virus: The shellcode and techniques may trigger AV alerts.

    Stability: Hijacking arbitrary threads can lead to crashes or instability. The examples are simplified and may not handle all edge cases.

    Platform: x64 only. For x86, adjust the context structure and register names.
```
