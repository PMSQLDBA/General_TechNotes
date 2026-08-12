# Technical Article: Installing WSL 2 and Ubuntu on Windows 11 — Complete Step-by-Step Linux Environment Setup

## 1. Introduction

This article provides a complete, step-by-step procedure for setting up a **real Linux environment on Windows 11 using Windows Subsystem for Linux 2 (WSL 2)**.

The procedure is tailored to the current state of your Windows 11 system:

* **Operating System:** Windows 11 Home Single Language
* **Architecture:** x64-based PC
* **Computer name:** PMSQLDBA
* **RAM:** 24 GB
* **Processor:** Intel Core i5-1135G7
* **Current shell:** Cygwin/Git-Bash-style shell
* **WSL:** Already installed
* **Virtual Machine Platform:** Already enabled
* **Linux distribution:** **Not yet installed**

--Software to download and install on Windows Personal PC:

-[CYGWIN](https://cygwin.com/install.html) 

Below output confirms that the WSL components have successfully been installed:

DELL@PMSQLDBA ~
$ wsl --install
The requested operation requires elevation.
Downloading: Windows Subsystem for Linux 2.7.11
Installing: Windows Subsystem for Linux 2.7.11
Windows Subsystem for Linux 2.7.11 has been installed.
Installing Windows optional component: VirtualMachinePlatform

Deployment Image Servicing and Management tool
Version: 10.0.26100.8972

Image Version: 10.0.26200.9168

Enabling feature(s)
[==========================100.0%==========================]
The operation completed successfully.
The requested operation is successful. Changes will not be effective until the system is rebooted.
The requested operation is successful. Changes will not be effective until the system is rebooted.


```text
Windows Subsystem for Linux 2.7.11 has been installed.
Installing Windows optional component: VirtualMachinePlatform

The operation completed successfully.

The requested operation is successful.
Changes will not be effective until the system is rebooted.
```

Therefore, **you do not need to reinstall WSL**. The next stage is to reboot Windows and install a Linux distribution, preferably Ubuntu.

Microsoft's current documentation states that WSL requires a Linux distribution before you can begin using the Linux environment. The `wsl --install` workflow can install WSL and Ubuntu, while `wsl --install -d <DistributionName>` can install a specific distribution when WSL itself is already installed. ([Microsoft Learn][1])

---

# 2. Understand What Has Already Been Installed

It is important to understand the distinction between **WSL** and a **Linux distribution**.

Your current setup is approximately:

```text
Windows 11
    |
    +-- WSL 2 components
    |
    +-- Virtual Machine Platform
    |
    +-- Cygwin
    |
    +-- NO Linux distribution yet
```

You have installed the WSL infrastructure, but you have **not installed Ubuntu**.

Therefore, this command from your current Cygwin terminal:

```bash
lsblk
```

still produces:

```text
-bash: lsblk: command not found
```

That is expected.

`lsblk` is a Linux command. Installing WSL does not convert your existing Cygwin terminal into a Linux terminal.

You need to install Ubuntu first.

---

# 3. Cygwin, WSL 2, and Ubuntu Are Different

There are three different components involved here.

## 3.1 Cygwin

Cygwin provides a UNIX/POSIX-like environment on Windows.

For example:

```text
Cygwin
   |
   +-- Bash
   +-- grep
   +-- awk
   +-- sed
   +-- ssh
   +-- Git
   +-- UNIX utilities
```

Cygwin is **not the Linux kernel**.

---

## 3.2 WSL 2

WSL 2 provides the Linux virtualization/runtime infrastructure.

Conceptually:

```text
Windows 11
    |
    +-- WSL 2
          |
          +-- Linux kernel
```

But WSL 2 by itself is not the Linux distribution you will normally interact with.

---

## 3.3 Ubuntu

Ubuntu provides the Linux userspace:

```text
WSL 2
   |
   +-- Linux kernel
   |
   +-- Ubuntu
        |
        +-- bash
        +-- apt
        +-- sudo
        +-- lsblk
        +-- ps
        +-- df
        +-- ssh
        +-- Linux filesystem
```

This is what you need next.

---

# 4. Why Your Current `lsblk` Command Does Not Work

You are currently seeing:

```text
DELL@PMSQLDBA ~
$
```

and:

```bash
lsblk
```

returns:

```text
-bash: lsblk: command not found
```

This is because you are still operating inside your existing Bash environment.

Think of the environments as separate:

```text
Windows 11
│
├── Cygwin Terminal
│     └── bash
│          └── lsblk not installed/available
│
└── WSL 2
      └── Ubuntu
           └── lsblk
```

After Ubuntu is installed, you will open **Ubuntu**, not Cygwin, and execute:

```bash
lsblk
```

there.

---

# 5. Step 1 — Reboot Windows 11

Your WSL installation output explicitly says:

```text
The requested operation is successful.
Changes will not be effective until the system is rebooted.
```

Therefore, the first step is:

**Restart Windows 11.**

Use:

```text
Start
   → Power
      → Restart
```

Do not skip this step.

The Virtual Machine Platform component must be fully activated after the reboot.

Microsoft's WSL documentation also specifies restarting Windows after the required components have been enabled. ([Microsoft Learn][2])

---

# 6. Step 2 — Open PowerShell After Reboot

After Windows restarts:

1. Click **Start**
2. Search for:

```text
PowerShell
```

3. Right-click **Windows PowerShell**
4. Select:

```text
Run as administrator
```

You should see a prompt similar to:

```text
PS C:\Windows\System32>
```

Do **not** use the Cygwin terminal for the following WSL installation commands.

Use **PowerShell**.

---

# 7. Step 3 — Verify WSL

Run:

```powershell
wsl --status
```

This checks the current WSL configuration.

You can also run:

```powershell
wsl --version
```

This displays the installed WSL version information where supported.

You should see that WSL is installed.

---

# 8. Step 4 — Check Whether a Linux Distribution Is Installed

Run:

```powershell
wsl --list --verbose
```

or the shorter form:

```powershell
wsl -l -v
```

Microsoft documents `wsl --list --verbose` as the command for displaying installed distributions, their state, and whether they use WSL 1 or WSL 2. ([Microsoft Learn][3])

Because you haven't installed Ubuntu yet, you may see something indicating that no distributions are installed.

That is expected.

---

# 9. Step 5 — Display Available Linux Distributions

Now run:

```powershell
wsl --list --online
```

or:

```powershell
wsl -l -o
```

Microsoft provides this command specifically for displaying Linux distributions available for installation through WSL. ([Microsoft Learn][1])

You should see distributions such as:

```text
NAME
Ubuntu
Debian
kali-linux
openSUSE-Tumbleweed
...
```

The exact list may change over time.

---

# 10. Step 6 — Install Ubuntu

For your purpose, I recommend **Ubuntu**.

Run:

```powershell
wsl --install -d Ubuntu
```

Microsoft documents this syntax for installing a specific Linux distribution after WSL is already installed. ([Microsoft Learn][1])

You may see output similar to:

```text
Downloading: Ubuntu
Installing: Ubuntu
Ubuntu has been installed.
```

The exact version and wording may differ.

---

# 11. If `wsl --install -d Ubuntu` Does Not Work

If the installation appears to hang at 0% or encounters a download problem, Microsoft documents the `--web-download` option.

Try:

```powershell
wsl --install --web-download -d Ubuntu
```

This downloads the distribution from an online source instead of relying on the Microsoft Store path. ([Microsoft Learn][1])

Do not use this option unless you actually encounter a problem with the normal command.

---

# 12. Step 7 — Wait for Ubuntu Installation

The first time Ubuntu is initialized, WSL needs to create the Linux filesystem.

You may see:

```text
Installing, this may take a few minutes...
```

Do not close the window.

Eventually you should see something like:

```text
Enter new UNIX username:
```

This is an important point.

You are now creating a **Linux user account**.

---

# 13. Step 8 — Create Your Linux Username

When Ubuntu asks:

```text
Enter new UNIX username:
```

you could use:

```text
dell
```

or another simple lowercase Linux username.

For example:

```text
Enter new UNIX username: dell
```

The Linux username is separate from your Windows account.

Your Windows account is:

```text
DELL
```

Your Linux account could be:

```text
dell
```

They do not have to be identical.

---

# 14. Step 9 — Create the Linux Password

Ubuntu will then ask:

```text
New password:
```

Enter a password.

You will notice that **nothing appears while you type**.

You won't see:

```text
****
```

or:

```text
••••
```

This is normal Linux terminal behavior.

Type the password and press Enter.

Ubuntu will ask you to confirm it.

Enter it again.

---

# 15. Step 10 — Understand the Linux User You Just Created

The user you created becomes the normal Linux user.

For example:

```text
dell
```

You will typically see a prompt such as:

```text
dell@PMSQLDBA:~$
```

The components mean approximately:

```text
dell
 |
 +-- Linux username

@
 
PMSQLDBA
 |
 +-- hostname

:
 
~
 |
 +-- home directory

$
 |
 +-- normal user
```

The `$` means you are operating as a normal user.

---

# 16. Step 11 — Verify the Linux Environment

Run:

```bash
whoami
```

Expected:

```text
dell
```

Then:

```bash
id
```

You should see your Linux UID/GID information.

For example:

```text
uid=1000(dell) gid=1000(dell) groups=1000(dell)...
```

Then:

```bash
pwd
```

You should see something similar to:

```text
/home/dell
```

This is a genuine Linux filesystem path.

---

# 17. Step 12 — Verify the Linux Distribution

Run:

```bash
cat /etc/os-release
```

You should see information identifying Ubuntu.

For example:

```text
NAME="Ubuntu"
ID=ubuntu
```

The exact Ubuntu version may vary.

---

# 18. Step 13 — Verify the Linux Kernel

Run:

```bash
uname -a
```

You should see Linux kernel information.

This is a major difference from Cygwin.

Cygwin:

```text
Windows kernel
    |
Cygwin compatibility layer
    |
Bash
```

WSL 2:

```text
Windows 11
    |
WSL 2
    |
Linux kernel
    |
Ubuntu
    |
Bash
```

---

# 19. Step 14 — Test `lsblk`

Now execute:

```bash
lsblk
```

This is the command you originally wanted.

You should now be running the Linux `lsblk` utility inside Ubuntu.

You can also check its location:

```bash
which lsblk
```

Typically, it will be located under a standard Linux binary directory such as:

```text
/usr/bin/lsblk
```

depending on the Ubuntu package layout.

---

# 20. Step 15 — Test Other Linux Commands

Now test:

```bash
df -h
```

Then:

```bash
free -h
```

Then:

```bash
ps aux
```

Then:

```bash
uname -a
```

Then:

```bash
ip addr
```

Then:

```bash
ss -tuln
```

Then:

```bash
ls -la /
```

These commands should now operate in the Ubuntu/WSL Linux environment.

---

# 21. Step 16 — Test `sudo`

Now test:

```bash
sudo -i
```

Ubuntu will ask for the password you created during Ubuntu initialization.

Enter it.

Your prompt should change from something like:

```text
dell@PMSQLDBA:~$
```

to:

```text
root@PMSQLDBA:~#
```

Now run:

```bash
whoami
```

You should receive:

```text
root
```

This is the Linux `sudo`/root model you were originally trying to use.

---

# 22. Why Your Earlier `sudo` Message Was Confusing

Earlier you ran:

```bash
sudo su -
```

and received:

```text
Sudo is disabled on this machine.
To enable it, go to the Developer Settings page in the Settings app
```

That was happening in your existing Windows/Cygwin environment.

The message refers to Windows' sudo functionality.

It was **not evidence that Ubuntu's Linux sudo was broken**.

After Ubuntu is installed, you will have:

```text
Ubuntu
   |
   +-- sudo
   |
   +-- root
```

and:

```bash
sudo -i
```

will operate according to the Linux security model.

---

# 23. Step 17 — Verify WSL Version from PowerShell

Leave Ubuntu with:

```bash
exit
```

If you were root, you may need to run `exit` again to return completely to Windows.

Then open PowerShell.

Run:

```powershell
wsl --list --verbose
```

You want to see something similar to:

```text
  NAME      STATE           VERSION
* Ubuntu    Running         2
```

The critical field is:

```text
VERSION
2
```

Microsoft documents this command as the standard way to verify whether an installed distribution is using WSL 1 or WSL 2. ([Microsoft Learn][3])

---

# 24. Step 18 — Set WSL 2 as the Default

Although new distributions installed using the modern `wsl --install` workflow use WSL 2 by default, you can explicitly establish WSL 2 as the default for future distributions:

```powershell
wsl --set-default-version 2
```

Microsoft documents this command for selecting WSL 1 or WSL 2 as the default version for new Linux distributions. ([Microsoft Learn][1])

---

# 25. Step 19 — Set Ubuntu as the Default Distribution

If you install only Ubuntu, it will normally be the default distribution.

You can explicitly set it with:

```powershell
wsl --set-default Ubuntu
```

Then:

```powershell
wsl
```

will launch Ubuntu.

Microsoft documents `wsl --set-default <Distro>` for selecting the default distribution. ([Microsoft Learn][1])

---

# 26. Step 20 — Launch Ubuntu from PowerShell

Instead of opening Ubuntu from the Start menu, you can launch it from PowerShell:

```powershell
wsl
```

This starts your default Linux distribution.

You can also explicitly launch Ubuntu:

```powershell
wsl -d Ubuntu
```

Microsoft documents `wsl --distribution <DistroName>` for launching a specific installed distribution. ([Microsoft Learn][1])

---

# 27. Step 21 — Launch Linux as Root When Necessary

From PowerShell:

```powershell
wsl -d Ubuntu -u root
```

This starts Ubuntu directly as the Linux `root` user.

Inside Ubuntu:

```bash
whoami
```

should return:

```text
root
```

This is useful for recovery and administration.

For normal daily work, however, use your regular Linux account and elevate with:

```bash
sudo command
```

or:

```bash
sudo -i
```

when necessary.

---

# 28. Step 22 — Update Ubuntu

Once Ubuntu is running, update the package database:

```bash
sudo apt update
```

Then upgrade installed packages:

```bash
sudo apt upgrade -y
```

This is now real Ubuntu package management.

Unlike Cygwin, Ubuntu uses:

```bash
apt
```

for Debian/Ubuntu package management.

---

# 29. Step 23 — Install Common Linux Administration Tools

For a Linux administration/DBA learning environment, I recommend:

```bash
sudo apt install -y \
    vim \
    nano \
    curl \
    wget \
    git \
    unzip \
    zip \
    tree \
    htop \
    net-tools \
    dnsutils \
    lsof \
    rsync \
    openssh-client
```

This gives you useful commands such as:

```text
vim
nano
curl
wget
git
tree
htop
ifconfig
nslookup
dig
lsof
rsync
ssh
```

---

# 30. Step 24 — Install Additional DBA Utilities

For a database administration learning environment, you can add:

```bash
sudo apt install -y \
    procps \
    psmisc \
    util-linux \
    sysstat \
    iotop
```

Then test:

```bash
lsblk
```

```bash
iostat
```

```bash
vmstat
```

```bash
pidstat
```

```bash
lsof
```

These are useful for learning Linux system and database performance administration.

---

# 31. Step 25 — Understand the WSL Filesystem

Inside Ubuntu:

```bash
ls -la /
```

You should see directories such as:

```text
bin
boot
dev
etc
home
lib
mnt
opt
proc
root
run
sbin
sys
tmp
usr
var
```

This is a Linux filesystem hierarchy.

Your Linux user's home directory is:

```text
/home/dell
```

You can navigate there:

```bash
cd ~
```

or:

```bash
cd /home/dell
```

---

# 32. Step 26 — Access Windows Files from Ubuntu

WSL automatically exposes Windows drives under `/mnt`.

Your C: drive is normally:

```bash
/mnt/c
```

Test:

```bash
cd /mnt/c
```

Then:

```bash
ls
```

You should see Windows directories such as:

```text
Program Files
Users
Windows
```

Your Windows user directory can normally be accessed as:

```bash
cd /mnt/c/Users/DELL
```

Then:

```bash
ls
```

This allows Linux applications to work with Windows files.

---

# 33. Step 27 — Understand Linux and Windows Paths

Windows:

```text
C:\Users\DELL\Documents
```

WSL:

```text
/mnt/c/Users/DELL/Documents
```

Linux home:

```text
/home/dell
```

These are different filesystems.

For Linux development work, Microsoft recommends keeping Linux projects inside the WSL/Linux filesystem when possible rather than constantly working through `/mnt/c`. This can provide better Linux filesystem performance for Linux workloads.

---

# 34. Step 28 — Verify the Network

Inside Ubuntu:

```bash
ip addr
```

Then:

```bash
ip route
```

Then:

```bash
cat /etc/resolv.conf
```

Test Internet connectivity:

```bash
ping -c 4 8.8.8.8
```

Then test DNS:

```bash
ping -c 4 google.com
```

If these work, your Linux environment has network connectivity.

---

# 35. Step 29 — Verify DNS

Install DNS tools if necessary:

```bash
sudo apt install -y dnsutils
```

Then:

```bash
nslookup google.com
```

or:

```bash
dig google.com
```

This is useful for practicing Linux network troubleshooting.

---

# 36. Step 30 — Verify SSH Client

Run:

```bash
ssh -V
```

If OpenSSH is installed, you should see an OpenSSH version.

You can later use this environment to connect to Linux servers:

```bash
ssh username@server
```

This is particularly useful for DBA administration.

---

# 37. Step 31 — Understand `lsblk` Under WSL

Although `lsblk` now exists in Ubuntu, there is another important technical consideration.

WSL 2 is not identical to a physical Linux server.

A conventional Linux server might show:

```text
NAME        SIZE TYPE
sda         500G disk
├─sda1       1G part
└─sda2     499G part
```

Your WSL environment has a virtualized storage architecture.

Therefore, don't expect WSL `lsblk` output to represent your physical Windows NVMe/SSD exactly as a native Linux server would.

For Windows physical disks, use:

```powershell
Get-Disk
```

and:

```powershell
Get-Partition
```

For Linux filesystem administration inside Ubuntu, use:

```bash
lsblk
df -h
du -sh
mount
findmnt
```

---

# 38. Step 32 — Understand the Three Disk Views

You now have three different perspectives.

### Windows

```powershell
Get-Disk
```

shows Windows disks.

### WSL/Linux

```bash
lsblk
```

shows Linux/WSL block-device information.

### Filesystem usage

```bash
df -h
```

shows mounted filesystem capacity and usage.

These commands answer different questions.

---

# 39. Step 33 — Install Windows Terminal

For daily WSL usage, Windows Terminal is highly recommended.

You can use separate tabs for:

```text
PowerShell
Ubuntu
Cygwin
Command Prompt
```

For example:

```text
Windows Terminal
├── PowerShell
├── Ubuntu
└── Cygwin
```

Microsoft specifically recommends Windows Terminal as a way to run and manage multiple WSL distributions and other command-line environments. ([Microsoft Learn][1])

---

# 40. Step 34 — Learn to Recognize Your Environment

This is extremely important when working with Windows, Cygwin, and WSL simultaneously.

## Cygwin

You may see:

```text
DELL@PMSQLDBA ~
$
```

Check:

```bash
uname
```

Cygwin will identify itself differently from a native Linux kernel.

---

## Ubuntu / WSL

You may see:

```text
dell@PMSQLDBA:~$
```

Check:

```bash
cat /etc/os-release
```

You should see:

```text
Ubuntu
```

And:

```bash
uname -a
```

should show Linux.

---

## PowerShell

You will see:

```text
PS C:\Users\DELL>
```

PowerShell commands include:

```powershell
Get-Disk
Get-Process
Get-Service
Get-ComputerInfo
```

---

# 41. Step 35 — Do Not Run Linux Commands in the Wrong Environment

This is the main lesson from your original problem.

You ran:

```bash
lsblk
```

inside:

```text
Cygwin
```

and expected:

```text
Linux
```

The correct workflow is:

```text
PowerShell
    |
    +-- wsl -d Ubuntu
             |
             +-- Linux
                  |
                  +-- lsblk
```

Or:

```text
Start Menu
    |
    +-- Ubuntu
          |
          +-- lsblk
```

---

# 42. Complete Installation Command Sequence

Once Windows has been rebooted, the Windows-side sequence is:

```powershell
wsl --status
```

```powershell
wsl --list --online
```

```powershell
wsl --install -d Ubuntu
```

If necessary:

```powershell
wsl --install --web-download -d Ubuntu
```

After Ubuntu installation:

```powershell
wsl --list --verbose
```

Verify:

```text
Ubuntu    Running    2
```

Then launch:

```powershell
wsl -d Ubuntu
```

---

# 43. Complete Ubuntu Initial Configuration

Inside Ubuntu:

```bash
whoami
```

```bash
id
```

```bash
pwd
```

```bash
cat /etc/os-release
```

```bash
uname -a
```

Then:

```bash
sudo apt update
```

```bash
sudo apt upgrade -y
```

Then:

```bash
sudo apt install -y \
    vim \
    nano \
    curl \
    wget \
    git \
    unzip \
    zip \
    tree \
    htop \
    net-tools \
    dnsutils \
    lsof \
    rsync \
    openssh-client \
    sysstat
```

Then test:

```bash
lsblk
```

```bash
df -h
```

```bash
free -h
```

```bash
ps aux
```

```bash
ip addr
```

```bash
ss -tuln
```

---

# 44. Complete Verification Procedure

At the end of the installation, verify the Windows side:

```powershell
wsl --status
```

```powershell
wsl --list --verbose
```

Expected:

```text
NAME      STATE      VERSION
Ubuntu    Running    2
```

Then verify the Linux side:

```bash
whoami
```

```bash
cat /etc/os-release
```

```bash
uname -a
```

```bash
lsblk
```

```bash
df -h
```

```bash
free -h
```

```bash
ps aux
```

```bash
ip addr
```

```bash
sudo -i
```

Then:

```bash
whoami
```

Expected:

```text
root
```

Exit root:

```bash
exit
```

Exit Ubuntu:

```bash
exit
```

---

# 45. Final Architecture

After completing this procedure, your Windows 11 workstation will look conceptually like this:

```text
                         DELL PMSQLDBA
                         Windows 11
                              |
          +-------------------+-------------------+
          |                   |                   |
          v                   v                   v
     PowerShell            Cygwin              WSL 2
          |                   |                   |
          |                   |                   v
          |                   |                Ubuntu
          |                   |                   |
          |                   |            +------+------+
          |                   |            |             |
          |                   |            v             v
          |                   |          Bash          sudo
          |                   |            |             |
          |                   |            +------+------+
          |                   |                   |
          |                   |              Linux kernel
          |                   |                   |
          |                   |                   v
          |                   |            Linux filesystem
          |                   |
          |                   +-- POSIX tools
          |
          +-- Windows administration
```

---

# 46. Recommended DBA Learning Environment

Because your goal appears to involve Linux commands and database administration, I recommend keeping **both Cygwin and WSL 2**, but giving them different purposes.

### Cygwin

Use for:

```text
Bash
SSH
Git
grep
awk
sed
UNIX scripting
Windows-integrated shell work
```

### WSL 2 + Ubuntu

Use for:

```text
Linux administration
Linux filesystem
lsblk
df
du
mount
findmnt
ps
top
htop
free
ip
ss
lsof
systemd-related learning where supported/configured
apt
sudo
SSH
database software
Linux scripting
```

### PowerShell

Use for:

```text
Windows disks
Windows services
Windows processes
Windows networking
Windows configuration
Get-Disk
Get-Partition
Get-Process
Get-Service
```

---

# 47. Important Conclusion

Your current installation is **not broken**.

You are at this exact stage:

```text
                         CURRENT STATUS

Windows 11
    |
    +-- Cygwin -------------------- Installed
    |
    +-- WSL 2 -------------------- Installed
    |
    +-- Virtual Machine Platform -- Enabled
    |
    +-- Ubuntu -------------------- NOT INSTALLED
```

Therefore, your next steps are simply:

### Step 1

**Restart Windows.**

### Step 2

Open **PowerShell as Administrator**.

### Step 3

Run:

```powershell
wsl --status
```

### Step 4

Run:

```powershell
wsl --list --online
```

### Step 5

Install Ubuntu:

```powershell
wsl --install -d Ubuntu
```

### Step 6

Complete the Ubuntu username/password setup.

### Step 7

Verify:

```powershell
wsl --list --verbose
```

You want:

```text
Ubuntu    Running    2
```

### Step 8

Open **Ubuntu**, not Cygwin.

### Step 9

Run:

```bash
uname -a
```

```bash
cat /etc/os-release
```

```bash
lsblk
```

### Step 10

Configure Ubuntu:

```bash
sudo apt update
sudo apt upgrade -y
```

At that point, you will have a genuine **Linux userspace running under WSL 2 on Windows 11**, rather than merely a Bash shell such as Cygwin.

Microsoft's current WSL documentation confirms that WSL must be paired with a Linux distribution, that `wsl --install -d Ubuntu` is the appropriate command for installing Ubuntu when WSL is already present, and that `wsl --list --verbose` verifies the installed distribution and WSL version. ([Microsoft Learn][1])

**Official references:** 
[Microsoft Learn — Install WSL](https://learn.microsoft.com/en-us/windows/wsl/install?utm_source=chatgpt.com) 

[Microsoft Learn — Basic WSL Commands](https://learn.microsoft.com/en-us/windows/wsl/basic-commands?utm_source=chatgpt.com) 

[Microsoft Learn — Set up a WSL environment](https://learn.microsoft.com/en-us/windows/wsl/setup/environment?utm_source=chatgpt.com) 

# Technical Runbook: Install Ubuntu on WSL 2 and Build a Linux Environment on Windows 11

## Objective

Install Ubuntu on an existing WSL 2 installation and configure a functional Linux administration environment on Windows 11.

## Current State

The following components are already installed:

* Windows 11
* Windows Subsystem for Linux 2
* Virtual Machine Platform
* Cygwin

Ubuntu is not yet installed.

## Procedure

### Step 1 — Restart Windows

Restart Windows to activate the Virtual Machine Platform changes.

### Step 2 — Open PowerShell as Administrator

Start PowerShell using:

```text
Start → PowerShell → Run as administrator
```

### Step 3 — Verify WSL

```powershell
wsl --status
```

### Step 4 — List Available Distributions

```powershell
wsl --list --online
```

### Step 5 — Install Ubuntu

```powershell
wsl --install -d Ubuntu
```

If the download hangs or fails:

```powershell
wsl --install --web-download -d Ubuntu
```

### Step 6 — Complete Ubuntu Initialization

When Ubuntu asks:

```text
Enter new UNIX username:
```

create a Linux username such as:

```text
dell
```

Create and confirm a Linux password.

### Step 7 — Verify Ubuntu

Inside Ubuntu:

```bash
whoami
id
pwd
cat /etc/os-release
uname -a
```

### Step 8 — Test Linux Commands

```bash
lsblk
df -h
free -h
ps aux
ip addr
ss -tuln
```

### Step 9 — Test Linux sudo

```bash
sudo -i
```

Then:

```bash
whoami
```

Expected:

```text
root
```

Exit root:

```bash
exit
```

### Step 10 — Update Ubuntu

```bash
sudo apt update
sudo apt upgrade -y
```

### Step 11 — Install Administration Tools

```bash
sudo apt install -y \
    vim \
    nano \
    curl \
    wget \
    git \
    unzip \
    zip \
    tree \
    htop \
    net-tools \
    dnsutils \
    lsof \
    rsync \
    openssh-client \
    sysstat
```

### Step 12 — Verify WSL Version

From PowerShell:

```powershell
wsl --list --verbose
```

Expected:

```text
NAME      STATE      VERSION
Ubuntu    Running    2
```

### Step 13 — Set WSL 2 as Default

```powershell
wsl --set-default-version 2
```

### Step 14 — Set Ubuntu as Default

```powershell
wsl --set-default Ubuntu
```

### Step 15 — Launch Ubuntu from PowerShell

```powershell
wsl
```

or:

```powershell
wsl -d Ubuntu
```

### Step 16 — Access Windows Files

Inside Ubuntu:

```bash
cd /mnt/c
ls
```

Windows user files:

```bash
cd /mnt/c/Users/DELL
```

Linux home:

```bash
cd ~
```

### Step 17 — Verify Networking

```bash
ip addr
ip route
```

Test connectivity:

```bash
ping -c 4 8.8.8.8
```

Test DNS:

```bash
ping -c 4 google.com
```

### Step 18 — Final Linux Verification

Run:

```bash
whoami
id
pwd
uname -a
cat /etc/os-release
lsblk
df -h
free -h
ps aux
ip addr
ss -tuln
ssh -V
git --version
curl --version
```

## Environment Rules

Use **Cygwin** for Windows-integrated POSIX/UNIX tooling.

Use **Ubuntu on WSL 2** for Linux administration and Linux software.

Use **PowerShell** for Windows disk and operating-system administration.

For Windows disks:

```powershell
Get-Disk
Get-Partition
```

For Linux/WSL filesystems:

```bash
lsblk
df -h
du -sh
findmnt
```

Do not expect Cygwin, WSL 2, and Windows PowerShell to expose the same device and filesystem model.

## Final Target Architecture

```text
Windows 11
|
+-- PowerShell
|   +-- Windows administration
|   +-- Get-Disk
|   +-- Get-Partition
|   +-- Windows services/processes
|
+-- Cygwin
|   +-- Bash
|   +-- UNIX/POSIX utilities
|   +-- SSH
|   +-- Git
|
+-- WSL 2
    |
    +-- Linux kernel
    |
    +-- Ubuntu
        +-- bash
        +-- sudo
        +-- apt
        +-- lsblk
        +-- df
        +-- ps
        +-- ip
        +-- ssh
        +-- Linux filesystem
```

## Completion Criteria

The installation is complete when all of the following are true:

```text
[ ] Windows has been rebooted
[ ] WSL status is available
[ ] Ubuntu is installed
[ ] Ubuntu initializes successfully
[ ] A Linux user has been created
[ ] Linux password has been configured
[ ] `cat /etc/os-release` identifies Ubuntu
[ ] `uname -a` identifies Linux
[ ] `lsblk` executes inside Ubuntu
[ ] `sudo -i` works
[ ] `sudo apt update` works
[ ] `wsl --list --verbose` reports Ubuntu as VERSION 2
[ ] Windows files are accessible through /mnt/c
[ ] Linux networking works
```

[1]: https://learn.microsoft.com/en-us/windows/wsl/install "Install WSL | Microsoft Learn"
[2]: https://learn.microsoft.com/en-us/windows/wsl/setup/environment "Set up a WSL development environment | Microsoft Learn"
[3]: https://learn.microsoft.com/en-us/windows/wsl/basic-commands "Basic commands for WSL | Microsoft Learn"
