# Module 00 - Welcome to the Game

You tried to get a red teaming job. You failed.

Not because you are stupid. Not because you cannot learn.
You failed because nobody taught you how this world actually works.

Most "hacking tutorials" online show you how to run someone else's tool.
Type this command. Click this button. Copy this script.
It works once, and then it stops working, and you have no idea why.

That is not how real red teamers operate.

Real red teamers understand the machine they are attacking.
They understand the defenses they are bypassing.
They know why something works, not just that it works.
And when a technique stops working, they know how to fix it.

This course will turn you into that person.

---

## What This Course Is About

Every company that runs Windows has security software trying to stop you.
The most common one is Windows Defender. It comes built into every Windows
computer. It runs automatically. It watches everything.

Your job as a red teamer is to get past it.

Not by turning it off. Not by asking the admin to disable it.
By understanding how it works and finding the gaps.

This course teaches you how to do that. From zero to professional level.
Every technique is explained, practiced, and tested against real, fully
updated Windows Defender. You will use only free, open source tools.

By the end of this course, you will know how to:

- Get your code running on a protected Windows machine without Defender catching it
- Stop Windows from logging your activity
- Hide your tools inside legitimate programs
- Encrypt your payloads so scanners cannot read them
- Use Windows' own built-in tools to do your work
- Clean up after yourself so investigators find nothing
- Adapt your techniques when defenders catch up

---

## Who This Course Is For

You know how to use a computer. You can download files, open programs,
type commands in a terminal. Maybe you have done some basic pentesting -
run Nmap, used Metasploit once, watched some hacking videos.

That is all you need.

This course does not assume you know programming, Windows internals,
or any evasion techniques. We start from scratch and build up.

If you already know some of this stuff, great. You will move faster.
If you know nothing, that is fine too. Every concept is explained
before it is used.

---

## How This Course Works

This is a hands-on course. Reading alone will not make you good at this.

Every module follows the same pattern:

1. **Understand** the concept - what it is and why it exists
2. **Practice** it in your lab with step-by-step guidance
3. **See the detection** - how defenders catch this technique
4. **Learn to adapt** - how to change your approach when it gets caught

Modules are meant to be done in order. Later ones build on earlier ones.

---

## The Course Map

Here is what you will learn, module by module.

| Module | Topic | What You Will Learn |
|--------|-------|---------------------|
| 00 | **Introduction** | This page. The landscape, the mindset, the roadmap. |
| 01 | **Lab Setup** | Build your practice environment with Windows 11 + Kali Linux VMs. |
| 02 | **Windows Internals** | How Windows runs programs, how programs talk to the OS, where security products sit. |
| 03 | **AMSI Bypass Part 1** | The system that scans scripts before they run, and your first bypasses. |
| 04 | **AMSI Bypass Part 2** | Advanced bypass methods. Build your own bypass tool. |
| 05 | **ETW Patching** | The logging system that records everything. How to stop it. |
| 06 | **Process Injection Basics** | Put your code inside another running program. DLL and shellcode injection. |
| 07 | **Process Injection Advanced** | Reflective loading, process hollowing, APC injection. |
| 08 | **Code Obfuscation** | Make your code unreadable to scanners. String tricks, encoding, transforms. |
| 09 | **Syscalls & Unhooking** | Talk directly to the Windows kernel, skip security hooks. |
| 10 | **Living off the Land** | Use Windows built-in tools (certutil, mshta, etc.) for attacks. |
| 11 | **Payload Encryption** | Encrypt payloads so scanners can't read them. Build custom loaders. |
| 12 | **Advanced Memory** | Memory permission tricks, code caves, hiding code in unexpected places. |
| 13 | **Artifact Cleanup** | Change timestamps, clear logs, remove evidence of your presence. |
| 14 | **Process Manipulation** | Fake parent processes, hide command-line args, steal tokens. |
| 15 | **PPL & Kernel Attacks** | Protected processes, vulnerable drivers, kernel-level evasion. |

---

## The One Thing You Must Understand Before Starting

Security is a game between two sides. Attackers find ways to get past
defenses. Defenders find ways to catch attackers. Then attackers adapt.
Then defenders adapt. This never stops.

Every technique in this course will eventually get detected by some
future update to Defender. That is normal. That is expected.

The goal is not to memorize techniques that work forever.
The goal is to understand how things work deeply enough that when
a technique gets caught, you can figure out what changed and adapt.

A person who memorizes ten bypass scripts is useless when script
number eleven is needed. A person who understands why bypasses work
can create script number eleven themselves.

This course teaches you to be the second person.

For every technique, we will cover what it does, why it works, how
defenders detect it, and how to change your approach when the current
version gets caught. This "adapt and overcome" mindset is what
separates someone who passes the interview from someone who does not.

---

## Rules

1. **Use a lab.** Never practice on systems you do not own. We build a safe lab with virtual machines in Module 01.
2. **Get permission.** Testing on a real company's systems without written permission is a crime. No exceptions.
3. **Do the labs.** Reading is not enough. Every module has hands-on exercises. Do all of them.
4. **Take notes.** Write down what works, what does not, the date, and the Defender version. This becomes your personal playbook.
5. **Break things.** Your lab is for breaking. Restore a snapshot and try again. You learn the most from failures.

---

## Tools We Will Use

Everything is free and open source. You do not need to download any of
these yet - we set everything up in Module 01.

- **Metasploit / msfvenom** - generate raw payload code that we wrap in our own evasion techniques
- **Sliver** - modern command and control framework, actively maintained, used by real red teams
- **Mythic** - alternative C2 framework with a web interface
- **PowerShell** - already on every Windows machine, used for scripting and AMSI practice
- **SysWhispers** - talk directly to the Windows kernel, bypassing security hooks
- **Donut** - convert regular programs into injectable code that runs in memory
- **System Informer** - see everything inside a running system: processes, memory, DLLs, connections

---

## What Makes This Course Different

Most tutorials show you one technique and move on. Here is a script,
it bypasses AMSI, good luck. That works until Defender updates its
signatures next week and the script gets caught.

This course teaches the thinking behind the techniques. When you
understand why a bypass works, you can modify it when it breaks.
When you understand what defenders are looking for, you can avoid
those patterns. When you understand the internals of the system,
you can find new bypass methods that nobody has published yet.

That is the difference between following a recipe and knowing
how to cook.

---

## Ready?

The next module is lab setup. You will build your practice environment
from scratch. Take your time with it. Make sure everything works
before moving on.

No lab means no practice. No practice means no learning.

Let's go.

---

Next: [Module 01 - Lab Setup](./01_LAB_SETUP.md)

---

*This course is for authorized security testing and education only.
Always get written permission before testing any system.
The author is not responsible for misuse of this information.*
