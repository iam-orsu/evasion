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

This is a hands-on course. You will build things, break things,
and fix things. Reading alone will not make you good at this.

Every module follows the same pattern. First you understand a concept.
Then you set up your lab to practice it. Then you do it yourself.
Then you see how defenders detect it. Then you learn how to change
your approach when the current one stops working.

The course is split into modules. Each module focuses on one skill.
You should do them in order because later modules build on earlier ones.

---

## The Course Map

Here is what you will learn, module by module.

**Module 00** is this introduction. You are reading it now.

**Module 01** is lab setup. You will build a safe practice environment
with virtual machines. A Windows 11 machine to attack, and a Kali Linux
machine to attack from. Nothing leaves your computer. Nothing touches
real systems.

**Module 02** covers Windows internals. Before you can bypass security,
you need to understand how Windows actually runs programs. What happens
when you double-click an EXE. How programs talk to the operating system.
Where security products insert themselves into this process. This is the
foundation everything else builds on.

**Module 03** is your first real attack technique. You will learn about
AMSI - the system that scans your scripts before they run - and how to
get past it. This is the starting point for almost every red team
operation that uses PowerShell or .NET tools.

**Module 04** goes deeper into AMSI bypass. More methods, more advanced
approaches, and you will build your own bypass tool from scratch.

**Module 05** covers ETW, the logging system that records everything
you do. You will learn how to stop it from recording your actions,
so the security team has nothing to investigate.

**Module 06** teaches process injection fundamentals. This is how you
put your code inside another running program - making it look like
a normal, trusted process is doing the work instead of your tool.

**Module 07** takes injection further. Reflective loading, process
hollowing, and other advanced techniques that leave almost no trace.

**Module 08** is about code obfuscation. Making your code unreadable
to automated scanners. String tricks, encoding, and tools that
transform your code so it slips past pattern matching.

**Module 09** covers direct system calls and unhooking. This is how
you talk directly to the Windows kernel, skipping the monitoring
hooks that security products place in your way.

**Module 10** teaches Living off the Land. Using tools that are already
installed on every Windows machine - certutil, mshta, bitsadmin, and
others - to do your work without bringing any custom tools at all.

**Module 11** is payload encryption and packing. Wrapping your
attack code in encryption so scanners cannot read it, and building
custom loaders that decrypt and run it in memory.

**Module 12** covers advanced memory techniques. Tricks with memory
permissions, hiding code in unexpected places, and other low-level
methods that avoid detection.

**Module 13** is about cleaning up. Changing file timestamps, clearing
logs, removing evidence. Making it look like you were never there.

**Module 14** covers process manipulation. Faking parent-child process
relationships, hiding command-line arguments, and borrowing security
tokens from other processes.

**Module 15** is the advanced module. Protected Process Light, vulnerable
drivers, and kernel-level attacks. This is expert territory.

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

**Use a lab.** Never practice on systems you do not own. We will build
a safe lab with virtual machines in Module 01. Everything stays inside
your computer.

**Get permission.** If you ever use these techniques on a real company's
systems, you need written permission first. Without it, it is a crime.

**Do the labs.** Reading is not enough. You learn by doing. Every module
has hands-on exercises. Do all of them.

**Take notes.** Write down what works and what does not. Include the date
and the Defender version. Techniques that work today might not work
next week. Your notes become your personal playbook.

**Break things.** Your lab is for breaking. That is why we have snapshots.
If you mess something up, restore the snapshot and try again. You learn
the most from things that go wrong.

---

## Tools We Will Use

Everything in this course is free and open source. Here is a quick list
of the main tools. You do not need to download any of these yet. We
will set everything up properly in the lab module.

**Metasploit and msfvenom** for generating payloads. These are the
starting point - raw payload code that we then wrap in our own
evasion techniques.

**Sliver** is a modern command and control framework. When you
compromise a machine, you need a way to send it commands. Sliver
handles that. It is actively maintained and widely used by real
red teams.

**Mythic** is another command and control framework with a web
interface. Good to know as an alternative.

**PowerShell** is already on every Windows machine. We use it
for scripting, testing, and as a target for AMSI practice.

**SysWhispers** helps you talk directly to the Windows kernel,
bypassing security monitoring.

**Donut** converts regular programs into injectable code that
runs entirely in memory without touching disk.

**System Informer** shows you everything happening inside
a running Windows system. Processes, memory, DLLs, network
connections. Essential for understanding what is going on.

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
