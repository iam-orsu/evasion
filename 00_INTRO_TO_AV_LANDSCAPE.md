# Module 00 - The Antivirus Landscape (2026)

## What Is This Course About?

You want to be a red teamer.

A red teamer is someone who gets hired to hack into companies.
But legally. With permission. To find problems before real attackers do.

The biggest thing that stops you from doing your job?
Antivirus. Specifically, Windows Defender.

Defender is the security software that comes with every Windows computer.
It watches everything you do. It scans every file. It blocks attack tools.
It logs your actions. It reports to the cloud.

If you cannot get past Defender, you cannot do your job.

This course teaches you how to get past it.

Every technique. From beginner to expert.
Step by step. Using only free, open source tools.

---

## How This Course Is Built

This is not a "read and memorize" course.
This is a "build and break" course.

The flow is simple:

```
1. UNDERSTAND what something is and why it exists
2. SET UP your lab so you can practice safely
3. LEARN the technique with clear explanation
4. PRACTICE it hands-on in your lab
5. SEE the detection (understand how defenders catch you)
6. LEARN to change your technique when it gets caught
```

Every module follows this pattern.

---

## The Big Picture: What Are We Up Against?

Before we go into details, here is the big picture.

When you try to run an attack tool on a Windows computer,
you have to get past several layers of security.

Think of it as a building with multiple security guards.
Each guard checks for different things.
You need to get past ALL of them, not just one.

Here are the guards:

```
GUARD 1: File Scanner
   Checks every file that touches the disk.
   If the file matches known malware, it gets deleted.
   This is the oldest and simplest form of detection.

GUARD 2: AMSI (Anti-Malware Scan Interface)
   Checks scripts and code BEFORE they run.
   Even if your script is hidden inside another file,
   AMSI sees the actual code right before it runs.

GUARD 3: Behavior Monitor
   Watches what running programs DO.
   Even if your file passes all scans,
   if it does something suspicious (like reading passwords),
   the behavior monitor catches it.

GUARD 4: ETW (Event Tracing for Windows)
   Logs everything that happens on the system.
   Even if your attack works, the logs show what you did.
   The security team reads these logs and finds you.

GUARD 5: Cloud Protection
   Sends unknown files to Microsoft's servers.
   Microsoft has huge computing power and AI models.
   They check your file and send back a verdict in seconds.

GUARD 6: Kernel Protection
   Defender runs partly in the deepest level of Windows.
   It protects itself so you cannot just turn it off.
```

In this course, you will learn how to deal with each of these guards.

---

## What Is Windows Defender?

Windows Defender is the antivirus built into Windows 10 and Windows 11.
Every Windows computer has it. It is turned on by default.

A lot of people think Defender is weak. That was true years ago.
In 2026, Defender is one of the best antivirus products in the world.
Many paid antivirus programs are not better than Defender.

As a red teamer, Defender is your main problem.
The good news: if you can get past Defender, you can get past almost anything.

Defender does these things:

**It scans files.**
The moment a file appears on disk (downloaded, copied, created),
Defender scans it. If the file matches known malware, it gets blocked.
This is called "real-time protection."

**It scans scripts before they run.**
If you try to run a PowerShell script or a VBScript,
Defender uses AMSI to read the script before it runs.
Even if the script was encoded or hidden, AMSI sees the final version.
If it matches known attack patterns, it gets blocked.

**It watches program behavior.**
Even if your file and script look clean, Defender watches what happens next.
If your program opens another program's memory and writes code into it,
that is suspicious. Defender will flag or block it.

**It talks to the cloud.**
When Defender sees a file it does not recognize,
it can send the file (or info about it) to Microsoft's cloud servers.
Microsoft checks it using AI and huge databases.
A verdict comes back in a few seconds.
If Microsoft says it is bad, Defender blocks it on your machine AND
shares that info with every other Defender user worldwide.

**It protects itself.**
You cannot just open Settings and turn Defender off.
There is a feature called Tamper Protection that blocks anyone
from disabling Defender - even administrators.
Defender also runs as a Protected Process, which means
normal programs cannot mess with it.

---

## What Is AMSI?

AMSI stands for Anti-Malware Scan Interface.

Here is the problem AMSI solves:

In the old days, antivirus only scanned FILES on disk.
Attackers figured out: "What if I never save a file?
What if I run everything directly in memory?"

They would encode their attack script, pass it to PowerShell,
and PowerShell would decode it and run it - all in memory.
The antivirus never saw the actual attack code because it
only looked at the encoded version on disk.

AMSI fixed this.

Now, when PowerShell (or any supported program) is about to run code,
it sends that code to AMSI first. AMSI passes it to Defender.
Defender checks the ACTUAL code that is about to run - not the
encoded version, not the file on disk, but the real, decoded,
ready-to-execute code.

If Defender says it is bad, the code is blocked before it runs.

AMSI checks code in:
- PowerShell scripts
- VBScript and JavaScript (Windows Script Host)
- .NET programs loaded in memory
- Office macros (VBA)
- WMI operations

This is why many old attack techniques stopped working.
Encoding your payload in Base64 used to be enough.
Now AMSI sees through the encoding.

Why does AMSI matter for you?

If you want to run any attack tool in PowerShell or .NET,
you need to get past AMSI first. If you do not, your tool
will be blocked the moment it tries to run.

AMSI bypass is usually the FIRST thing a red teamer does
after getting access to a target machine.

We will learn exactly how AMSI works inside and how to bypass it
in Modules 03 and 04 (after you have a lab to practice in).

---

## What Is ETW?

ETW stands for Event Tracing for Windows.

ETW is a logging system built into Windows.
It records what happens on the system.

When a program starts, ETW can log it.
When a network connection is made, ETW can log it.
When a PowerShell command runs, ETW can log it.
When a file is created, ETW can log it.

Windows has over 1000 different ETW "providers."
Each provider logs different types of events.

Why does this matter for red teaming?

Even if your attack works perfectly - you bypass Defender,
you bypass AMSI, you steal the credentials - ETW recorded
everything you did.

The next morning, the security team opens their log dashboard.
They see:
- "PowerShell ran this suspicious command at 2:13 AM"
- "A .NET assembly was loaded in memory at 2:14 AM"
- "Process X accessed the password storage at 2:15 AM"

They trace your whole attack. They know exactly what you did,
what tools you used, and which computers you touched.

This is why red teamers need to deal with ETW too.
You either disable the logging before your attack,
or you clean up the logs after.

We will learn how ETW works and how to deal with it in Module 05.

---

## The Cat and Mouse Game

This is the most important concept in this whole course.

Security is a cat and mouse game.
Attackers find new ways to get past defenses.
Defenders find new ways to catch attackers.
Then attackers find new ways again.
This never ends.

What does this mean for you?

**Every technique has a shelf life.**

A technique that works today might get caught next month.
Microsoft updates Defender signatures every single day.
Sometimes multiple times per day.

When a new bypass technique is published online,
this is roughly what happens:

```
Week 1: Researcher publishes a new technique on a blog
Week 2: Red teamers start using it
Week 3: Microsoft sees the technique being used in the wild
Week 4: Microsoft releases a detection update
Week 5: The technique no longer works on updated systems
```

This cycle repeats forever.

**So why learn techniques if they get caught?**

Because the CONCEPTS do not change. Only the details change.

For example: AMSI bypass works by changing code in memory
so the scan function does not work anymore.
Microsoft might catch the SPECIFIC way you change it.
But the concept - "change the scan function in memory" - still works.
You just need to change it in a DIFFERENT way.

This is the difference between a tool user and a real red teamer:

```
TOOL USER:
- Downloads a bypass script from GitHub
- Runs it. It works!
- Microsoft blocks the script next month
- Stuck. Looks for another script. 

REAL RED TEAMER:
- Understands WHY the bypass works
- When the specific script gets caught,
  changes the approach slightly
- Makes a new version that works again
- Never stuck. Never dependent on one script.
```

This course teaches you to be the second type.
For every technique, we explain:
1. What it does
2. WHY it works
3. How defenders catch it
4. How to change it when it gets caught

---

## What Tools Will We Use?

This course uses only free, open source tools.
You do not need to buy anything.

Here is a quick overview. Do not worry about installing
these yet - we will set everything up in the lab module.

**Metasploit / msfvenom**
The most well-known attack framework.
We use it mainly to create raw payload code (shellcode).
The raw payload by itself gets caught by Defender.
That is the point - we learn to wrap it in ways that do not get caught.

**Sliver C2**
A modern command and control framework by Bishop Fox.
When you compromise a target, you need a way to send commands to it.
That is what a C2 framework does.
Sliver is actively maintained in 2026 and is very popular with red teams.

**Mythic C2**
Another C2 framework. This one has a nice web interface
and supports many different types of agents.
Good to know as an alternative to Sliver.

**PowerShell**
Already installed on every Windows machine.
We use it for scripting, AMSI bypass practice, and running tools.
It is heavily monitored by Defender, which makes it good for learning -
if your technique works in PowerShell, it works against real monitoring.

**C# and .NET**
Many red team tools are written in C#.
We will write some tools in C# to practice building our own.
You do not need to be a C# expert. We will explain the code.

**Python**
We use Python for some helper scripts and payload building.
Basic Python knowledge is helpful but not required.

**SysWhispers**
A tool that helps you talk directly to the Windows kernel,
skipping the security hooks that Defender and EDR products place.
We will learn why this matters and how to use it.

**Donut**
Converts regular programs into shellcode (small, injectable code).
This lets you take any .NET tool and inject it into another process
without ever saving a file to disk.

**Process Hacker / System Informer**
A tool that shows you everything running on a Windows system.
What processes are running, what DLLs they loaded, what memory
they are using. Essential for understanding what is happening
under the hood.

---

## Course Module Map

Here is every module and what it covers.
This is your roadmap for the whole course.

```
MODULE 00 (This One)
What: The landscape. What we are up against. What we will learn.

MODULE 01 - LAB SETUP
What: Build your practice lab from scratch.
      VMware Pro, Windows 11 VM, attacker machine.
      Step by step, screenshot by screenshot.
      You MUST complete this before anything else.

MODULE 02 - WINDOWS INTERNALS FOR RED TEAMERS
What: How Windows works under the hood.
      What is a process, what is memory, what is a DLL.
      How programs talk to Windows. User mode vs kernel mode.
      The API chain that every attack tool uses.
      No attacks yet - just understanding the battlefield.

MODULE 03 - AMSI BYPASSES PART 1
What: How AMSI works inside (the technical details).
      Your first AMSI bypass using PowerShell reflection.
      String tricks to avoid detection.
      Hands-on: run blocked scripts after bypassing AMSI.

MODULE 04 - AMSI BYPASSES PART 2
What: Advanced AMSI bypass methods.
      Memory patching techniques.
      Hardware breakpoint bypass (no code changes in memory).
      Building your own bypass tool.

MODULE 05 - ETW PATCHING AND LOGGING
What: How ETW works inside.
      How to stop Windows from logging your actions.
      Patching the logging functions in memory.
      Before-and-after: show that logs are empty.

MODULE 06 - PROCESS INJECTION FUNDAMENTALS
What: Putting your code inside another running program.
      The key Windows functions you need to know.
      DLL injection step by step.
      Shellcode injection step by step.

MODULE 07 - PROCESS INJECTION ADVANCED
What: Reflective DLL injection (invisible to file scanners).
      Process hollowing (hiding inside a legit program).
      APC injection (triggering code through thread queues).
      Real malware case studies.

MODULE 08 - CODE OBFUSCATION AND STRINGS
What: Making your code unreadable to scanners.
      String encoding (Base64, hex, XOR).
      Dynamic string building.
      Tools that do obfuscation for you.

MODULE 09 - SYSCALLS AND API UNHOOKING
What: How security products hook Windows functions.
      Calling the kernel directly (skipping the hooks).
      Removing hooks from memory.
      Building your own syscall wrapper.

MODULE 10 - LIVING OFF THE LAND
What: Using Windows built-in tools for attacks.
      certutil, mshta, bitsadmin, rundll32 and more.
      No custom tools needed - just what Windows gives you.
      How Defender detects this and how to avoid it.

MODULE 11 - PAYLOAD ENCRYPTION AND PACKING
What: Encrypting your attack code so scanners cannot read it.
      AES, XOR, RC4 encryption.
      Building custom loaders that decrypt and run payloads.
      Custom packers.

MODULE 12 - ADVANCED MEMORY TECHNIQUES
What: Memory permission tricks.
      Hiding shellcode in unexpected places.
      ROP chains basics.
      Code caves.

MODULE 13 - TIMESTOMPING AND ARTIFACT CLEANUP
What: Changing file timestamps to hide your tracks.
      Clearing event logs.
      Removing evidence of your presence.
      What investigators look for.

MODULE 14 - PROCESS MANIPULATION
What: Faking parent process relationships.
      Hiding command-line arguments.
      Stealing and using other processes' security tokens.

MODULE 15 - PPL AND KERNEL ATTACKS
What: Protected Process Light - what it is and why it matters.
      Using vulnerable drivers to get kernel access (BYOVD).
      Kernel-level evasion.
```

---

## What You Need Before Starting

**Skills you need:**
- You can use Windows normally (open programs, download files, use settings)
- You can type commands in a terminal (PowerShell or Command Prompt)
- You have basic understanding of what hacking/pentesting is

That is it. Everything else, this course teaches you.

**Hardware you need:**
- A computer with at least 4 CPU cores
- At least 16 GB RAM (32 GB is better)
- At least 100 GB free disk space
- Internet connection

**Software you need:**
- All free. We will download everything in Module 01 (Lab Setup).

---

## Important Rules

**1. ALWAYS use a lab.**
Never practice on systems you do not own.
We will build a safe lab with virtual machines in Module 01.
Everything stays inside your lab. Nothing touches real systems.

**2. ALWAYS get written permission.**
If you ever test these techniques on a company's systems,
you must have written permission FIRST.
Without permission, it is a crime. No exceptions.

**3. DO the hands-on labs.**
Reading is not enough. You need to practice.
Every module has labs. Do all of them.
If you skip labs, you will not learn.

**4. BREAK things.**
Your lab is for breaking. That is why we use virtual machines.
If you break something, restore from a snapshot and try again.
You learn the most from things that go wrong.

**5. KEEP notes.**
Write down what works and what does not.
Write down which Defender version you tested against.
Note the date - techniques that work today might not work next week.

---

## How Attackers Get Caught (The Quick Version)

Before we start learning techniques, you should know
what mistakes get attackers caught.

This gives you a mental checklist for everything you will learn later.

**Mistake 1: Using known tools without changing them.**
You download a famous hacking tool from GitHub.
You run it on the target. Defender recognizes it instantly.
Blocked in less than 1 second.
Why? Defender has a database of every known attack tool.
The file's fingerprint matches. Game over.

**Mistake 2: Running attack scripts without dealing with AMSI.**
You type an attack command in PowerShell.
AMSI sees the command text and recognizes it as an attack.
Blocked before it even runs.
Why? AMSI reads the actual command before PowerShell runs it.

**Mistake 3: Leaving logs behind.**
Your attack works. You steal credentials. You feel good.
Next morning, the security team opens the logs.
They see every command you ran, every file you touched,
every connection you made. They trace your whole attack.
Why? ETW was logging everything in the background.

**Mistake 4: Creating suspicious patterns.**
Your malware starts cmd.exe from inside Microsoft Word.
Word should never start cmd.exe. This pattern is a known
indicator of a macro attack. Defender blocks it.
Why? Defender watches parent-child process relationships.

**Mistake 5: Writing files to disk.**
You save your attack tool as a file on the target computer.
Defender scans it the moment it hits the disk.
If the file has any known pattern, it gets quarantined.
Why? Defender scans every file that appears on disk.

This course teaches you how to avoid ALL of these mistakes.

---

## What Makes This Course Different

Most courses and blog posts show you ONE technique.
"Here is a script, run it, it bypasses AMSI."

That is fine until the script gets caught by the next Defender update.
Then you are stuck.

This course teaches you:
- **What** the technique does
- **Why** it works (the part most courses skip)
- **How** defenders detect it
- **How to change it** when the current version gets caught

By the end, you will not just know techniques.
You will understand the game well enough to create your own.

---

## Glossary (Terms You Will See in This Course)

Do not memorize these now. Come back to this list when
you see a term you do not recognize.

```
AV          Antivirus - software that finds and blocks malware
Defender    Windows Defender - the AV built into Windows
AMSI        Anti-Malware Scan Interface - scans code before it runs
ETW         Event Tracing for Windows - logs system activity
EDR         Endpoint Detection and Response - advanced security tool
C2          Command and Control - framework to control compromised machines
Payload     The code you want to run on the target
Shellcode   Small, self-contained code (often the payload)
Beacon      A C2 agent that checks in with your server regularly
DLL         Dynamic Link Library - shared code files on Windows
API         Application Programming Interface - functions you can call
Syscall     System Call - a direct request to the Windows kernel
Hook        Code placed by security tools to monitor function calls
Unhooking   Removing those monitoring hooks
Process     A running program
Injection   Putting your code inside another running process
LOLBin      Living off the Land Binary - using built-in Windows tools to attack
Obfuscation Making code hard for scanners to read
PPL         Protected Process Light - Windows protection for important processes
BYOVD       Bring Your Own Vulnerable Driver - using a weak driver to attack
Signature   A known pattern that identifies specific malware
Heuristic   A rule-based guess about whether something is suspicious
Fileless    An attack that never saves files to disk
VM          Virtual Machine - a computer running inside your computer
Snapshot    A saved state of a VM you can go back to
```

---

## Ready?

Next module: we build the lab.

No lab = no practice.
No practice = no learning.

Go to Module 01 and set up your lab.
Once your lab is running, the real fun starts.

---

Next: [Module 01 - Lab Setup](./01_LAB_SETUP.md)

---

*This course is for authorized security testing and education only.*
*Always get written permission before testing any system.*
*The author is not responsible for misuse of this information.*
