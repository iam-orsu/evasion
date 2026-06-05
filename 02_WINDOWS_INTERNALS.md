# Module 02 - Windows Internals for Red Teamers

## Why This Module Matters

You cannot bypass something you do not understand. Before you touch any evasion technique, you need to know how Windows actually works under the hood.

This module covers the internals that every red teamer needs. By the end, you will understand how programs run, how they talk to the operating system, where security products sit in that chain, and why certain attack techniques work.

Everything here connects directly to the attack techniques in later modules. When we patch AMSI in Module 03, you will know exactly what we are patching and why. When we do process injection in Module 06, you will understand what memory we are writing to and how.

Open your Windows 11 VM and follow along. Every section has hands-on commands you should run.

---

## Part 1: What Is a Process

### Definition

A process is a running instance of a program. When a program sits on your disk as a `.exe` file, it is just a file. The moment Windows loads it into memory and starts running it, it becomes a process.

Every program you see running on your computer is a process. Notepad is a process. Chrome is a process. PowerShell is a process. Windows Defender (MsMpEng.exe) is a process.

### What a Process Contains

When Windows creates a process, it sets up several things for it:

- **A private memory space** - each process gets its own isolated chunk of memory. Process A cannot see or touch Process B's memory (under normal conditions). This isolation is a core security feature of Windows.
- **A Process ID (PID)** - a unique number that identifies this specific process. You see PIDs in Task Manager.
- **At least one thread** - a thread is the part that actually runs code. A process is the container; threads are the workers inside it. More on threads in a moment.
- **A security token** - this defines what the process is allowed to do. Is it running as Administrator? As a normal user? What groups does it belong to? The token answers all of these.
- **A handle table** - a list of system resources (files, registry keys, other processes) that this process has opened.
- **A PEB (Process Environment Block)** - an internal data structure that holds metadata about the process. We will cover this in detail later because it is very important for red teaming.

### Why Processes Matter for Red Teaming

Process injection means putting your code inside another process's memory and making it run there. This works because each process is isolated - if you can get your code inside a trusted process like `svchost.exe`, security tools think it is `svchost.exe` doing the work, not your malware.

To do injection, you need to understand what a process is, how its memory is organized, and what APIs let you interact with other processes.

### Hands-On: Exploring Processes

Open PowerShell on your Windows 11 VM and run these commands.

List all running processes:
```powershell
Get-Process | Format-Table Id, ProcessName, Path -AutoSize
```

Find a specific process:
```powershell
Get-Process -Name explorer
```

Get detailed info about the current PowerShell process:
```powershell
Get-Process -Id $PID | Select-Object Id, ProcessName, StartTime, Path, WorkingSet64, VirtualMemorySize64
```

`$PID` is a built-in variable in PowerShell that holds the Process ID of the current PowerShell session.

See how much memory each process uses:
```powershell
Get-Process | Sort-Object WorkingSet64 -Descending | Select-Object -First 10 Id, ProcessName, @{N='RAM_MB';E={[math]::Round($_.WorkingSet64/1MB,1)}}
```

Find Windows Defender's process:
```powershell
Get-Process -Name MsMpEng | Select-Object Id, ProcessName, Path
```

This is the Defender engine process. Later in the course, you will learn why you cannot simply open this process and modify it (it is a Protected Process).

---

## Part 2: What Is a Thread

A thread is the unit of execution inside a process. The process is the container that holds memory, handles, and tokens. The thread is the thing that actually runs instructions on the CPU.

Every process starts with one thread (the main thread). A process can create additional threads to do work in parallel. For example, a web browser has threads for rendering pages, handling network requests, and running JavaScript.

### Why Threads Matter for Red Teaming

Several attack techniques involve threads:

- **CreateRemoteThread injection** - you create a new thread inside another process that runs your code
- **Thread hijacking** - you pause an existing thread in another process, change where it is executing, and resume it so it runs your code instead
- **APC injection** - you queue your code to run when a thread enters a certain state

### Hands-On: Viewing Threads

```powershell
# See how many threads the current PowerShell process has
$proc = Get-Process -Id $PID
Write-Host "PowerShell (PID $PID) has $($proc.Threads.Count) threads"

# List thread IDs
$proc.Threads | Select-Object Id, StartTime, ThreadState | Format-Table
```

---

## Part 3: Memory - Virtual Address Space

### What Is Virtual Memory

Each process gets its own **Virtual Address Space (VAS)**. This is a range of memory addresses that the process can use. On a 64-bit system, each process has a theoretical address space of 128 TB.

The key word is "virtual." The process thinks it has a huge flat chunk of memory all to itself. In reality, Windows maps these virtual addresses to real physical RAM (and disk, via the page file) behind the scenes. The process does not know or care about the physical layout.

The important consequence: Process A's memory at address `0x7FF6A000` is completely separate from Process B's memory at the same address. They are different virtual address spaces. This is what keeps processes isolated from each other.

### How Process Memory Is Organized

When a process is running, its virtual address space is divided into regions:

```
HIGH ADDRESSES
+------------------------------------------+
|                                          |
|   KERNEL SPACE                           |
|   (off limits to user-mode code)         |
|                                          |
+------------------------------------------+  <- 0x7FFFFFFFFFFF (boundary)
|                                          |
|   STACK                                  |
|   Local variables, function call info.   |
|   Grows downward toward lower addresses. |
|                                          |
+------------------------------------------+
|         (unmapped/free space)            |
+------------------------------------------+
|                                          |
|   HEAP                                   |
|   Dynamically allocated memory.          |
|   Grows upward toward higher addresses.  |
|                                          |
+------------------------------------------+
|                                          |
|   LOADED DLLs                            |
|   ntdll.dll, kernel32.dll, amsi.dll...   |
|                                          |
+------------------------------------------+
|                                          |
|   .data / .rdata / .bss                  |
|   Global variables, read-only data       |
|                                          |
+------------------------------------------+
|                                          |
|   .text                                  |
|   The program's executable code          |
|                                          |
+------------------------------------------+
|   PE Headers                             |
+------------------------------------------+
LOW ADDRESSES
```

Two areas matter most for red teaming:

- **Stack** - when a function is called, its local variables and return address are pushed onto the stack. Stack-based buffer overflows target this area.
- **Heap** - when a program calls `VirtualAlloc()` or similar functions to request memory at runtime, it comes from the heap. When you inject shellcode into a process, you typically allocate heap memory to hold it.

### Memory Permissions

Every region of memory has permissions that control what you can do with it:

| Permission | Meaning |
|------------|---------|
| **R** (Read) | You can read data from this memory |
| **W** (Write) | You can write/modify data in this memory |
| **X** (Execute) | The CPU can run this memory as code |

Common permission combinations:

| Combination | Typical Use | Suspicious? |
|-------------|-------------|-------------|
| R-- | Read-only data, strings | No |
| RW- | Variables, heap data | No |
| R-X | Code sections (.text) | No |
| **RWX** | **Write AND execute** | **Yes** |

**Why RWX is suspicious:** Normal programs do not need memory that is both writable and executable at the same time. Code sections are R-X (run but do not modify). Data sections are RW- (modify but do not run). When a program creates RWX memory, it usually means "I want to write code into memory and then run it" - which is exactly what shellcode injection does. Defender watches for this.

**The safer approach for attackers:** Allocate memory as RW- (writable), write your shellcode into it, then change the permissions to R-X (executable) using `VirtualProtect`. This two-step approach is less suspicious than directly allocating RWX.

### Hands-On: Examining Process Memory

```powershell
# See memory usage of the current PowerShell process
$proc = Get-Process -Id $PID
Write-Host "=== Memory for PowerShell (PID $PID) ==="
Write-Host "Working Set (physical RAM): $([math]::Round($proc.WorkingSet64 / 1MB, 2)) MB"
Write-Host "Private Memory: $([math]::Round($proc.PrivateMemorySize64 / 1MB, 2)) MB"
Write-Host "Virtual Memory: $([math]::Round($proc.VirtualMemorySize64 / 1MB, 2)) MB"
```

For a more detailed view of memory regions and their permissions, use System Informer:
1. Open **System Informer** (installed in Module 01)
2. Find **powershell.exe** in the process list
3. Double-click it
4. Click the **Memory** tab
5. You will see every memory region with its address, size, and protection level (R, RW, RX, etc.)

Look through the list. You should see:
- Regions marked **RX** - these are code sections of loaded DLLs
- Regions marked **RW** - these are data sections and heap
- If you see any region marked **RWX** - that is worth investigating

---

## Part 4: What Is a PE File

### Definition

PE stands for **Portable Executable**. It is the file format that Windows uses for executable files. Every `.exe`, `.dll`, `.sys` (driver), and `.scr` (screensaver) file on Windows follows the PE format.

When Defender scans a file, it reads the PE structure. When you build a custom payload, you are creating a PE file. When you do reflective DLL injection, you are manually loading a PE file into memory. Understanding this format is essential.

### PE Structure Overview

A PE file is organized into headers and sections:

```
+----------------------------+
|  DOS Header                |  Legacy header for backward compatibility.
|  (starts with "MZ")        |  Contains a pointer to the PE header.
+----------------------------+
|  PE Signature              |  The bytes "PE\0\0" (0x50450000).
|                            |  Confirms this is a PE file.
+----------------------------+
|  File Header               |  Machine type, number of sections,
|  (COFF Header)             |  timestamp, characteristics.
+----------------------------+
|  Optional Header           |  Entry point address, image base,
|                            |  section alignment, subsystem.
+----------------------------+
|  Section Table             |  Describes each section below:
|                            |  name, virtual address, size, permissions.
+----------------------------+
|  .text section             |  The actual executable code.
+----------------------------+
|  .rdata section            |  Read-only data. Import and export tables.
+----------------------------+
|  .data section             |  Initialized global variables.
+----------------------------+
|  .rsrc section             |  Resources: icons, version info, embedded data.
+----------------------------+
|  .reloc section            |  Relocation information (for ASLR).
+----------------------------+
```

### Key Sections for Red Teaming

- **.text** - contains the CPU instructions. This is the actual code that runs. When you patch a function in memory (like AmsiScanBuffer), you are modifying bytes in the .text section of a loaded DLL.
- **.rdata** - contains the **Import Address Table (IAT)** and **Export Table**. The IAT lists every function the program imports from DLLs. Security tools inspect the IAT to see what APIs a program uses. If your payload imports `VirtualAllocEx`, `WriteProcessMemory`, and `CreateRemoteThread`, that is a dead giveaway for process injection.
- **.rsrc** - the resources section. Attackers sometimes hide encrypted payloads here as embedded resources.

### The Import Address Table (IAT)

When a program calls a function from a DLL (like `CreateFileW` from kernel32.dll), it does not hardcode the function's address. Instead, it has an entry in the IAT. When Windows loads the program, the loader fills in the IAT with the actual addresses of each imported function.

This matters because:
- **Static analysis** - security tools read the IAT to see what functions a program uses without running it. Suspicious imports raise red flags.
- **IAT hooking** - EDR products can overwrite IAT entries to redirect function calls through their monitoring code.
- **Dynamic resolution** - to avoid suspicious imports in the IAT, attackers use `GetProcAddress()` to find function addresses at runtime instead.

### Hands-On: Examining a PE File

```powershell
# Let's verify that notepad.exe is a valid PE file
$path = "C:\Windows\System32\notepad.exe"
$bytes = [System.IO.File]::ReadAllBytes($path)

# Check the DOS header (first two bytes should be "MZ" = 0x4D 0x5A)
$mz = [System.Text.Encoding]::ASCII.GetString($bytes[0..1])
Write-Host "DOS Header: $mz (should be 'MZ')"

# Find the PE header offset (stored at offset 0x3C in the DOS header)
$peOffset = [BitConverter]::ToInt32($bytes, 0x3C)
Write-Host "PE Header is at offset: 0x$($peOffset.ToString('X'))"

# Read the PE signature (should be 0x00004550 = "PE\0\0")
$peSig = [BitConverter]::ToInt32($bytes, $peOffset)
if ($peSig -eq 0x00004550) {
    Write-Host "PE Signature: Valid (0x$($peSig.ToString('X')))"
} else {
    Write-Host "Not a valid PE file"
}
```

---

## Part 5: What Are DLLs

### Definition

DLL stands for **Dynamic Link Library**. A DLL is a file that contains code and data that multiple programs can share. Instead of every program including its own copy of common functions, they all use the shared copies in DLLs.

For example, every program that needs to create a file calls `CreateFileW`. This function lives in `kernel32.dll`. Notepad, Chrome, Word - they all use the same `kernel32.dll` instead of each having their own copy.

When a process starts, Windows loads the required DLLs into that process's memory space. The DLL code becomes part of the process's address space. This is important: when we say "amsi.dll is loaded in PowerShell," we mean the code of amsi.dll is sitting in PowerShell's memory, and PowerShell calls functions in that code.

### Key Windows DLLs

These are the DLLs you will encounter constantly in this course:

| DLL | Role | Why It Matters for Red Teaming |
|-----|------|-------------------------------|
| **ntdll.dll** | Native API. Bridge between user mode and kernel. Loaded into every process. | Contains syscall stubs. EDRs hook functions here. Direct syscalls bypass these hooks. |
| **kernel32.dll** | Win32 API. High-level functions like CreateFile, VirtualAlloc, CreateProcess. | The "official" way programs talk to Windows. Most red team tools call functions here. |
| **kernelbase.dll** | Actual implementation of many kernel32 functions. | kernel32 forwards many calls here. |
| **advapi32.dll** | Security functions: tokens, registry, services. | Used for privilege escalation, token manipulation. |
| **user32.dll** | GUI functions: windows, messages, input. | SetWindowsHookEx injection uses this. |
| **amsi.dll** | Anti-Malware Scan Interface. | Loaded into PowerShell and .NET. This is what we bypass in Module 03. |

### Hands-On: See DLLs Loaded in PowerShell

```powershell
# List all DLLs loaded in the current PowerShell process
$proc = Get-Process -Id $PID

Write-Host "=== DLLs loaded in PowerShell (PID $PID) ==="
Write-Host ""

# Show key DLLs we care about
$keyDlls = @("ntdll.dll", "KERNEL32.DLL", "KERNELBASE.dll", "amsi.dll", "advapi32.dll", "user32.dll")

foreach ($name in $keyDlls) {
    $mod = $proc.Modules | Where-Object { $_.ModuleName -eq $name }
    if ($mod) {
        $addr = "0x" + $mod.BaseAddress.ToString("X")
        $sizeMB = [math]::Round($mod.ModuleMemorySize / 1KB, 1)
        Write-Host "[LOADED] $($mod.ModuleName) at $addr ($sizeMB KB)"
    } else {
        Write-Host "[NOT FOUND] $name"
    }
}

Write-Host ""
Write-Host "Total DLLs loaded: $($proc.Modules.Count)"
```

Pay attention to two things in the output:
1. **amsi.dll** is loaded. Its base address is where AMSI lives in this PowerShell process. In Module 03, we will patch code at an offset from this address.
2. **ntdll.dll** is loaded. This is where syscall stubs and ETW functions live. In Module 05, we will patch `EtwEventWrite` inside this DLL.

---

## Part 6: The API Chain

### How Programs Talk to Windows

When your program wants to do something that requires the operating system's help (create a file, allocate memory, start a process), it cannot do it directly. It must ask the Windows kernel.

This request travels through a chain of DLLs before reaching the kernel. Understanding this chain is one of the most important things in this course, because the chain is where both attacks and defenses happen.

### The Full Chain

Here is what happens when a program calls `CreateFile` to open a file:

```
YOUR PROGRAM
     |
     | calls CreateFileW(...)
     v
kernel32.dll  (Win32 API - high level)
     |
     | forwards to kernelbase.dll
     v
kernelbase.dll  (actual implementation)
     |
     | calls NtCreateFile(...)
     v
ntdll.dll  (Native API - low level)        <-- EDR hooks go HERE
     |
     | puts syscall number in EAX register
     | runs the 'syscall' CPU instruction
     v
========= USER MODE / KERNEL MODE BOUNDARY =========
     |
     v
ntoskrnl.exe  (Windows Kernel)
     |
     | actually creates the file on disk
     | returns the result
     v
========= BACK TO USER MODE =========
     |
     v
YOUR PROGRAM receives the file handle
```

Every Windows API call follows this pattern. `VirtualAlloc` goes through `NtAllocateVirtualMemory`. `OpenProcess` goes through `NtOpenProcess`. `CreateRemoteThread` goes through `NtCreateThreadEx`.

### Why This Chain Matters

This chain is where the entire cat and mouse game plays out:

1. **EDR products hook ntdll.dll** - they replace the first instructions of functions like `NtAllocateVirtualMemory` with a jump to their own monitoring code. Every time any program calls these functions, the EDR sees it first.

2. **Direct syscalls skip ntdll.dll** - if you put the syscall instruction in your own code, you bypass the EDR hooks entirely. The EDR never sees your call because it never passes through the hooked function.

3. **AMSI works the same way** - PowerShell loads amsi.dll and calls `AmsiScanBuffer` before running scripts. If you break that function, the scan never happens.

---

## Part 7: User Mode vs Kernel Mode

### Two Privilege Levels

Windows has two privilege levels enforced by the CPU:

**User Mode (Ring 3):**
- Where all normal programs run
- Limited access - cannot touch hardware directly
- Cannot access other processes' memory (normally)
- Where DLLs like ntdll.dll, kernel32.dll, amsi.dll live
- Where YOUR code runs

**Kernel Mode (Ring 0):**
- Where the OS kernel and drivers run
- Full access to everything - all memory, all hardware
- Where Defender's kernel driver (WdFilter.sys) runs
- Where the real work happens (file I/O, memory management, process creation)

Programs in user mode cannot do anything important by themselves. They cannot create files, allocate memory, or start processes without asking the kernel. The way they ask is through **system calls**.

---

## Part 8: System Calls (Syscalls)

### What Is a Syscall

A system call is the mechanism that transitions the CPU from user mode (Ring 3) to kernel mode (Ring 0). It is how user-mode code asks the kernel to do something.

Each kernel function has a unique number called the **System Service Number (SSN)**. When making a syscall, the program puts the SSN in the `EAX` CPU register and runs the `syscall` instruction. The CPU switches to kernel mode and looks up which kernel function to run based on the SSN.

### The Syscall Stub in ntdll.dll

Every `Nt*` function in ntdll.dll follows the same pattern. Here is what `NtAllocateVirtualMemory` looks like in assembly:

```asm
NtAllocateVirtualMemory:
    mov r10, rcx          ; save first argument (syscall destroys rcx)
    mov eax, 0x18         ; put the SSN in eax (0x18 for this function)
    syscall               ; transition to kernel mode
    ret                   ; return to caller
```

That is all ntdll does for most Nt functions. Four instructions. It puts the number in EAX and runs `syscall`. The kernel does the actual work.

**Important:** The SSN (0x18 in this example) changes between Windows versions and even between builds of the same version. You cannot hardcode SSNs and expect them to work everywhere.

### How EDR Hooking Works

EDR products need to monitor what programs do. The easiest place to monitor is ntdll.dll, because every API call passes through it.

Here is what the function looks like before and after an EDR hooks it:

**Before hooking (clean ntdll.dll):**
```asm
NtAllocateVirtualMemory:
    4C 8B D1          mov r10, rcx
    B8 18 00 00 00    mov eax, 0x18
    0F 05             syscall
    C3                ret
```

**After hooking (EDR modified the first bytes):**
```asm
NtAllocateVirtualMemory:
    E9 xx xx xx xx    jmp EDR_Monitor_Function    <-- EDR replaced this!
    B8 18 00 00 00    mov eax, 0x18
    0F 05             syscall
    C3                ret
```

Now when any program calls `NtAllocateVirtualMemory`:
1. Execution jumps to the EDR's code first
2. The EDR logs: "Process X is allocating memory with these parameters"
3. The EDR decides: is this suspicious?
4. If OK, the EDR calls the real syscall
5. If suspicious, the EDR blocks it

This is why process injection gets caught. The EDR sees you calling `NtAllocateVirtualMemory` in another process, then `NtWriteVirtualMemory`, then `NtCreateThreadEx`. That pattern = injection = blocked.

### How Direct Syscalls Bypass This

If you know the SSN, you can write the syscall stub in your own code:

```asm
; Your own code, not inside ntdll.dll
mov r10, rcx
mov eax, 0x18       ; SSN for NtAllocateVirtualMemory
syscall              ; go directly to kernel
ret
```

The EDR hook is on the `NtAllocateVirtualMemory` function inside ntdll.dll. Your code never calls that function. It goes directly to the kernel. The EDR never sees it.

This is what tools like **SysWhispers** do - they generate these syscall stubs for you. We will use this in Module 09.

### Hands-On: See Syscall Numbers

```powershell
# Let's read the actual bytes of NtAllocateVirtualMemory from ntdll.dll in memory
# This shows you the real syscall stub

$ntdll = (Get-Process -Id $PID).Modules | Where-Object { $_.ModuleName -eq "ntdll.dll" }
Write-Host "ntdll.dll base address: 0x$($ntdll.BaseAddress.ToString('X'))"
Write-Host "ntdll.dll size: $([math]::Round($ntdll.ModuleMemorySize / 1KB)) KB"

# We can use Add-Type to call GetProcAddress and find function addresses
Add-Type @"
using System;
using System.Runtime.InteropServices;
public class NtdllHelper {
    [DllImport("kernel32.dll")]
    public static extern IntPtr GetProcAddress(IntPtr hModule, string procName);

    [DllImport("kernel32.dll")]
    public static extern IntPtr GetModuleHandle(string lpModuleName);
}
"@

$ntdllHandle = [NtdllHelper]::GetModuleHandle("ntdll.dll")
$funcAddr = [NtdllHelper]::GetProcAddress($ntdllHandle, "NtAllocateVirtualMemory")
Write-Host "NtAllocateVirtualMemory is at: 0x$($funcAddr.ToString('X'))"

# Read the first 8 bytes of the function
$bytes = New-Object byte[] 8
[System.Runtime.InteropServices.Marshal]::Copy($funcAddr, $bytes, 0, 8)

Write-Host ""
Write-Host "First 8 bytes of NtAllocateVirtualMemory:"
Write-Host ($bytes | ForEach-Object { "0x{0:X2}" -f $_ }) -Separator " "

# If the first bytes are 4C 8B D1 B8 xx 00 00 00, the function is NOT hooked
# The xx after B8 is the syscall number
# If the first byte is E9 or FF 25, an EDR has hooked this function

if ($bytes[0] -eq 0x4C -and $bytes[1] -eq 0x8B -and $bytes[2] -eq 0xD1) {
    $ssn = $bytes[4]
    Write-Host ""
    Write-Host "Function is CLEAN (not hooked)"
    Write-Host "Syscall number (SSN): 0x$($ssn.ToString('X2')) ($ssn)"
} else {
    Write-Host ""
    Write-Host "Function appears to be HOOKED or has a different structure"
    Write-Host "First byte is 0x$($bytes[0].ToString('X2')) instead of expected 0x4C"
}
```

Run this on your clean Windows 11 VM. Since you do not have an EDR installed, the function should be clean and you will see the actual syscall number. If you later install an EDR, run this again and you will see the hook.

---

## Part 9: The PEB (Process Environment Block)

### What Is the PEB

The PEB is a data structure that Windows creates for every process. It lives in the process's own user-mode memory (not in kernel space), which means the process itself can read and modify it.

The PEB contains metadata about the process:

- **ImageBaseAddress** - where the main .exe is loaded in memory
- **ProcessParameters** - command line arguments, current directory, environment variables
- **Ldr** (Loader Data) - a linked list of every DLL loaded in the process
- **BeingDebugged** - a flag (0 or 1) indicating whether a debugger is attached
- **OSMajorVersion / OSMinorVersion** - Windows version info

### Why the PEB Matters for Red Teaming

The PEB is used in several attack and defense techniques:

1. **Finding DLLs without imports** - shellcode cannot use normal imports (it has no IAT). Instead, it reads the PEB's Ldr field to walk the list of loaded DLLs and find the base address of `kernel32.dll` or `ntdll.dll`. This is called **PEB walking**.

2. **Anti-debugging** - the `BeingDebugged` field is what the `IsDebuggerPresent()` API reads. Malware checks this to detect if it is being analyzed. You can also manually set it to 0 to hide a debugger.

3. **Process hollowing** - this technique reads the PEB's `ImageBaseAddress` to find and replace the main executable's code in memory.

4. **Direct syscalls** - tools that resolve syscall numbers at runtime walk the PEB's module list to find ntdll.dll, then parse its export table to find the function addresses and extract SSNs.

### Hands-On: Reading PEB Information

```powershell
# Check if a debugger is attached (reads PEB.BeingDebugged internally)
$debuggerAttached = [System.Diagnostics.Debugger]::IsAttached
Write-Host "Debugger attached: $debuggerAttached"

# List all DLLs from the PEB's module list
# (Get-Process .Modules reads from the PEB's Ldr data)
$proc = Get-Process -Id $PID
Write-Host ""
Write-Host "=== Modules from PEB (first 15) ==="
$proc.Modules | Select-Object -First 15 | ForEach-Object {
    $addr = "0x" + $_.BaseAddress.ToString("X")
    Write-Host "$($_.ModuleName) at $addr"
}
```

The order of modules in this list follows the PEB's `InMemoryOrderModuleList`. The first entry is always the main executable (powershell.exe). The second is always `ntdll.dll`. This consistent ordering is what makes PEB walking reliable for shellcode.

---

## Part 10: Where Security Products Fit In

Now that you understand the internals, here is where Windows Defender and EDR products insert themselves into this picture.

### Defender's Components and Where They Sit

| Component | File | Where It Runs | What It Does |
|-----------|------|---------------|-------------|
| Filter Driver | WdFilter.sys | **Kernel mode** | Intercepts all file I/O. Triggers scans when files are created/modified. |
| Scan Engine | MsMpEng.exe + MpEngine.dll | **User mode** (service) | Runs signature, heuristic, emulation, and ML scans. |
| AMSI Provider | amsi.dll + MpOav.dll | **User mode** (per-process) | Scans scripts and .NET code before execution. |
| Network Filter | WdNisDrv.sys | **Kernel mode** | Monitors network traffic. |

### The Evasion Map

Based on everything you learned in this module, here is where each evasion technique targets:

```
YOUR CODE
     |
     | You call a function
     v
kernel32.dll ─────────────────────── IAT hooking happens here
     |
     v
ntdll.dll ────────────────────────── EDR inline hooks go here
     |                                AMSI/ETW functions live here
     |                                Direct syscalls SKIP this
     v
========= MODE BOUNDARY ═══════════
     |
     v
Kernel ───────────────────────────── WdFilter.sys scans files here
                                     Kernel callbacks monitor here
```

| Evasion Technique | What It Bypasses | Module |
|-------------------|-----------------|--------|
| AMSI patching | Script scanning (amsi.dll) | 03, 04 |
| ETW patching | Event logging (ntdll!EtwEventWrite) | 05 |
| Process injection | File-based scanning (nothing on disk) | 06, 07 |
| Direct syscalls | EDR hooks on ntdll.dll | 09 |
| ntdll unhooking | EDR hooks on ntdll.dll | 09 |
| Payload encryption | Signature scanning | 11 |
| LOLBins | File-based detection (no custom binaries) | 10 |

---

## Part 11: Process Creation - What Happens When You Double-Click an EXE

Now that you understand all the components, let's trace the full sequence of what happens when you run a program. This ties everything together.

When you double-click `tool.exe` on the desktop:

1. **Explorer.exe** (the Windows shell) detects your double-click and calls `CreateProcessW` in `kernel32.dll`.

2. **kernel32.dll** forwards to `CreateProcessInternalW` in `kernelbase.dll`, which validates the parameters, checks the security token, and locates the file.

3. **kernelbase.dll** calls `NtCreateUserProcess` in `ntdll.dll`. This is the lowest user-mode step. If an EDR has hooked this function, the EDR sees the process creation at this point.

4. **ntdll.dll** executes the `syscall` instruction, transitioning the CPU to kernel mode.

5. **The kernel** (ntoskrnl.exe) does the heavy work:
   - Creates an EPROCESS structure (the kernel's internal process object)
   - Creates the PEB in user-mode memory
   - Sets up the virtual address space
   - The **Windows Image Loader** reads the PE headers of `tool.exe`, maps its sections into memory, and loads all required DLLs (ntdll.dll, kernel32.dll, etc.)
   - Creates the initial thread
   - **WdFilter.sys** sees the file being read from disk and can trigger a Defender scan at this point

6. **CSRSS.exe** (Client Server Runtime Subsystem) is notified about the new process. This is required for the process to function correctly under the Windows subsystem.

7. The initial thread is **resumed** and begins executing at the program's entry point.

8. As the process runs, if it loads PowerShell or .NET, **amsi.dll** gets loaded into the process for script scanning.

### Hands-On: Watch a Process Being Created

```powershell
# Start a process and observe its details
$proc = Start-Process notepad -PassThru
Start-Sleep -Seconds 1

Write-Host "=== New Process Details ==="
Write-Host "Name: $($proc.ProcessName)"
Write-Host "PID: $($proc.Id)"
Write-Host "Start Time: $($proc.StartTime)"

# See its parent (should be this PowerShell process)
Write-Host "Parent PID: This PowerShell ($PID)"

# See its loaded DLLs
$proc = Get-Process -Id $proc.Id
Write-Host ""
Write-Host "Loaded DLLs ($($proc.Modules.Count) total):"
$proc.Modules | Select-Object -First 10 | ForEach-Object {
    Write-Host "  $($_.ModuleName)"
}

# Clean up
Stop-Process -Id $proc.Id -Force
Write-Host ""
Write-Host "Notepad closed."
```

Notice how even a simple program like Notepad loads dozens of DLLs. Each of those DLLs is code sitting in the process's memory that could potentially be patched, hooked, or replaced.

---

## Summary

Here is what you now know and why it matters:

| Concept | What It Is | Why It Matters |
|---------|-----------|---------------|
| Process | A running program with its own memory, threads, and token | Injection = putting code in another process |
| Thread | The unit that runs code inside a process | CreateRemoteThread, APC injection, thread hijacking |
| Virtual Address Space | Each process has its own isolated memory | Injection breaks this isolation on purpose |
| Memory Permissions | R, W, X flags on memory regions | RWX allocations are a red flag for Defender |
| PE Format | Structure of .exe and .dll files | IAT reveals imports; .text is where we patch |
| DLLs | Shared code loaded into process memory | amsi.dll, ntdll.dll are our main targets |
| API Chain | kernel32 → ntdll → syscall → kernel | EDR hooks sit on ntdll; direct syscalls skip them |
| User/Kernel Mode | Ring 3 (limited) vs Ring 0 (full access) | Syscalls are the bridge between them |
| Syscalls | Direct requests from user mode to kernel | Direct syscalls bypass user-mode EDR hooks |
| PEB | Per-process metadata structure in user memory | PEB walking finds DLLs; BeingDebugged for anti-debug |

---

## What Is Next

You now understand the battlefield. You know where programs live in memory, how they talk to Windows, and where security products insert themselves.

The next module puts this knowledge to use. You will learn exactly how AMSI works inside, and you will bypass it hands-on in your lab. Everything you learned about DLLs, memory, and function patching comes together there.

---

Next: [Module 03 - AMSI Bypass Part 1](./03_AMSI_BYPASS_PART1.md)

---

*This course is for authorized security testing and education only.
Always get written permission before testing any system.
The author is not responsible for misuse of this information.*
