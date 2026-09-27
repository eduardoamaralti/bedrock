# Bedrock -- Operations Log

## Project goal
Build and run a self-hosted Linux server from scratch, treated as real production infrastructure. The visible product is a personal portfolio site, but the real goal is hands-on, lasting mastery of Linux server administration -- networking, security, databases, APIs, containers -- documented step by step for future reference and to demonstrate practical experience.

---


## 2026-09-25 — Creating the Bedrock VM

**What I did:**
- Created VM in Hyper-V (Generation 2, 4GB fixed RAM, 40GB dynamic disk, Default Switch)
- Installed Ubuntu Server 26.04 LTS
- Installed OpenSSH server during installation

**Issues found and resolved:**

1. **Secure Boot blocked the boot process** -- error "signed image's hash is not allowed"
   - Cause: Secure Boot template was set to "Windows" instead of recognizing Linux
   - Fix: changed the template to "Microsoft UEFI Certificate Authority" in the VM Settings

2. **Kernel Panic during installation** -- "System is deadlocked on memory"
   - Cause: Dynamic Memory conflicting with the installer
   - Fix: disabled Dynamic Memory, fixed at 4096 MB

3. **Wrong keyboard layout** (not typing PT-PT correctly)
   - Fix: `sudo dpkg-reconfigure keyboard-configuration` - Generic 105-key - Portuguese - Portuguese

**Result:** Bedrock VM installed, running, PT-PT keyboard working, SSH server installed (key auth still pending).


## 2026-09-26 -- SSH Key Authentication

**What I did:**
- Generated a dedicated SSH key pair for this project: `ssh-keygen -t ed25519 -C "bedrock-project" -f $env:USERPROFILE\.ssh\id_ed25519_bedrock`
- Copied the public key into the VM's `~/.ssh/authorized_keys`
- Disabled password authentication in `/etc/ssh/sshd_config.d/50-cloud-init.conf` (`PasswordAuthentication no`)
- Restarted the SSH service to apply the change

**Issues found and resolved:**

1. **`PasswordAuthentication had no effect at first**
   - Cause: `/etc/ssh/sshd_config.d/50-cloud-init.conf` overrides the main `sshd_config` and still had `PasswordAuthentication yes`
   - Fix: edited that specific file and restarted `ssh`

2. **Key seemed to fail (`Permission denied (publickey)`)**
   - Cause: ran the debug command inside the VM console instead of the PowerShell window on the host PC
   - Fix: re-ran it in the correct terminal -- worked immediately

**Result:** SSH access to Bedrock now requires the dedicated key; password login is fully disabled.


## 2026-09-27 -- User Permissions & Static IP

**What I did:**
- Created a second user (`testuser`) without sudo, to verify permission separation
- Confirmed `testuser` cannot run privileged commands (`sudo apt update` correctly denied)
- Configured a static IP for the VM via Netplan (`172.30.224.50`), since DHCP was assigning a different IP on every reboot

**Result:** Confirmed sudo/non-sudo permission boundaries work as expected. VM now always reachable at the same IP -- no more hunting for it after every restart.


## 2026-09-27 -- System Update, Firewall & fail2ban

**What I did:**
- Ran full system update (`apt update && apt upgrade`)
- Installed and enabled UFW (firewall), allowing only OpenSSH before activating it
- Installed fail2ban, created a local jail config, and confirmed SSH protection is active
- Set up an SSH config shortcut (`ssh bedrock`) on the host PC for faster, safer daily use
- Colorized terminal output (ping, df, ps) using `grc` for better readability

**Result:** Phase 1 (Foundation & Hardening) fully complete -- key-based SSH, disabled password login, verified sudo/non-sudo permissions, static IP, active firewall, and brute-force protection via fail2ban.