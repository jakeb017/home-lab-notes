# home-lab-notes
Documenting my home lab setup and IT/security learning projects
<img width="1919" height="1020" alt="image" src="https://github.com/user-attachments/assets/34f8ac59-85a8-4b4e-a785-16bb5414f966" />
Set up Ubuntu 25.04 as a VM in VirtualBox for my home lab environment. Allocated 4GB RAM and 2 CPU cores, 25GB storage on a SATA controller. Chose NAT networking to keep it isolated from my host network while still having internet access for updates. Next step is configuring static IP and setting up SSH access.
<img width="911" height="909" alt="image" src="https://github.com/user-attachments/assets/2fff332a-fd95-4893-a0c4-052e5cae121d" />
Ran sudo apt update to refresh package lists, then installed neofetch with sudo apt install neofetch to test package management. Confirmed it worked by running neofetch and viewing system info output. Removed it afterward with sudo apt remove neofetch, and learned that this only removes the specified package, not its dependencies, which apt flags separately and requires sudo apt autoremove to clean up properly.
<img width="878" height="587" alt="image" src="https://github.com/user-attachments/assets/4b0c1761-2e32-42e4-98aa-0d022a9d1dd8" />
Used ping google.com to test internet connectivity from the VM, confirming DNS resolution worked by resolving google.com to an IP address automatically, and verified low latency around 27ms average with minor packet loss on this test. Learned that ping is typically the first diagnostic step for connectivity troubleshooting in a help desk scenario.
# Linux Fundamentals & Networking Basics

## VM Setup
Set up Ubuntu 25.04 as a virtual machine in VirtualBox for my home lab environment. Allocated 4GB RAM and 2 CPU cores, 25GB storage on a SATA controller. Chose NAT networking to keep it isolated from my host network while still having internet access for updates.

## Terminal & File Permissions
- Practiced basic navigation with `pwd`, `ls`, `cd`
- Learned the difference between `/` (root of the filesystem, mostly owned by root, limited access for regular users) and `~` (my own home directory where I have full control)
- Attempted to create a file while in `/` and got `Permission denied`, which showed regular users can't write to system level directories. Moved to `~` and it worked fine
- Used `chmod 755` to change file permissions, going from `rw-rw-r--` to `rwxr-xr-x`. Learned the numeric system: read = 4, write = 2, execute = 1, added together for owner, group, and others

## Users & Groups
- Created a new user `testuser` with `sudo adduser`, which automatically created a home directory and copied default files from `/etc/skel`
- Confirmed with `id testuser` that the account had no sudo access by default (correct security behaviour for a new account)
- Created a custom group `sales` with `groupadd` and added testuser to it using `sudo usermod -aG sales testuser`, making sure to use `-a` (append) so I didn't wipe their existing group memberships
- Added testuser to the `sudo` group the same way, then logged in as testuser using `su - testuser` and confirmed `sudo whoami` returned `root`, proving the admin access actually worked end to end, not just checking `id` and assuming

## Package Management
- Ran `sudo apt update` to refresh package lists before installing anything
- Installed `neofetch` with `sudo apt install neofetch` to test the install process, confirmed with system info output
- Removed it with `sudo apt remove neofetch`, then learned this leaves orphaned dependency packages behind, which `sudo apt autoremove` cleans up properly

## Networking Basics
- Used `ip a` to view network interfaces, identified `lo` (loopback, 127.0.0.1, the machine talking to itself) and `enp0s3` (main adapter, IP `10.0.2.15/24` assigned dynamically via NAT)
- Tested connectivity with `ping`, learned that stopping a ping manually with Ctrl+C can give a misleading packet loss reading since the in flight packet doesn't get a chance to return. Using `ping -c 10` for a fixed, complete test gives an accurate result
- Learned `8.8.8.8` (Google Public DNS) is the standard first test address for checking raw internet connectivity, since it's reliable, always on, and testing by IP directly rules out DNS as a factor
- Practiced a full troubleshooting order: ping `8.8.8.8` to check general internet access, ping the domain name to check DNS resolution specifically, then ping the local router/gateway to work out if a fault is local (home network) or upstream (ISP)
- Ran `traceroute` and saw it fail past the first hop for an external destination — learned this is often routers deliberately not responding to traceroute probes for security, not an actual connectivity problem, since `ping` to the same destination worked fine
- Used `netstat -tuln` to view listening services, learned to read the flags (`t` = TCP, `u` = UDP, `l` = listening only, `n` = numeric output) and identified DNS (port 53), CUPS printing (port 631), and mDNS (port 5353). Learned the security difference between a service listening on `127.0.0.1` (local access only) versus `0.0.0.0` (listening on all interfaces, potentially reachable externally)

## Remote Troubleshooting Process (Client Scenario)
Worked through how I'd actually handle a client call for a wifi issue where I can't run commands myself:
1. Clarify the exact symptom (one device or all, wifi or ethernet too, when it started)
2. Get them to check router lights for anything abnormal
3. Talk them through a power cycle (unplug 30 seconds, plug back in, wait for full reboot)
4. Test again after reboot
5. If still broken, compare wifi vs mobile data to narrow down local network vs ISP issue
6. Escalate to ISP or a technician if it's beyond a local fix
7. Document what was tried and the outcome

# SSH & Simulated Support Ticket

## Setting Up SSH
Installed and configured SSH on the VM to practice remote access, a core skill for real help desk and sysadmin work.

```
sudo apt install openssh-server
sudo systemctl start ssh
sudo systemctl enable ssh
sudo systemctl status ssh
```

Confirmed it was actively listening on port 22 (`0.0.0.0` and `::`) via the systemd status log. Learned the difference between installing a service and actually starting/enabling it — a fresh install is inactive by default until explicitly started, and won't survive a reboot unless enabled.

## Connecting via SSH
```
ssh jake@localhost
```

- First connection triggered a host authenticity/fingerprint check — SSH's way of protecting against man-in-the-middle attacks by confirming you trust this specific machine before connecting. Accepting it (`yes`) adds it to the local known_hosts list so future connections don't ask again.
- Authenticated with the account password, then confirmed the session was genuinely active with `whoami` and `hostname`.
- Exited cleanly with `exit`.

**Troubleshooting note:** hit a keyboard layout issue where `@` produced `"` instead — traced this back to the VM's keyboard layout being set to US (`setxkbmap -query` showed `Layout: us`) rather than Australian. Worked around it in the short term by using `"` in place of `@`, since a layout fix via `setxkbmap` didn't take effect properly under the Xwayland session in use.

## Real World Application
Practiced logging into a different user's account over SSH (`ssh carlton@localhost`) to simulate remote support — logging in as the affected user directly rather than just using `sudo` as myself, to see the exact environment and permissions they're experiencing.

## Simulated Support Ticket: Permission Denied
Set up and resolved a realistic file permissions issue end to end:

1. **Baseline confirmed** — created a file as carlton (`touch myfile.txt`), confirmed normal default permissions (`-rw-rw-r--`)
2. **Problem simulated** — as jake, ran `sudo chmod 000 /home/carlton/myfile.txt` to strip all permissions from the file, simulating a genuine "can't access my file" ticket
3. **Reported symptom** — logged back in as carlton, ran `cat myfile.txt`, got `Permission denied`
4. **Diagnosed before fixing** — ran `ls -l myfile.txt` to actually confirm the fault (`----------`, zero permissions for anyone) rather than guessing at a cause
5. **Applied the fix** — `chmod 664 myfile.txt`, restoring standard read/write access for the owner and group, read-only for others
6. **Verified the fix** — ran `ls -l` again to confirm permissions were restored, then re-ran `cat myfile.txt` to confirm the file was genuinely accessible again, rather than assuming the permission change alone was proof enough

This reflects the actual real-world troubleshooting workflow: reproduce/confirm the symptom, diagnose with evidence, apply the minimal fix needed, then verify the fix actually resolved the issue rather than just assuming it worked.

## chmod Number Reference
Consolidated understanding of common permission combinations:
```
777 = full access for everyone (rwx rwx rwx) — generally avoided, considered poor security practice
755 = owner full access, group/others can read and execute but not write
664 = owner and group can read/write, others read only
644 = owner can read/write, others read only
700 = only the owner has any access at all
```
Read = 4, Write = 2, Execute = 1 — added together per group (owner/group/others) to form each digit.

<img width="912" height="1009" alt="image" src="https://github.com/user-attachments/assets/7a00b6b3-7c4d-4ebe-9370-4ca3ef9614db" />
# Ports, Nmap & Firewall (UFW)

## Ports
Ports identify what type of traffic is going where on a device. Common ones:
- 22 = SSH
- 80 = HTTP
- 443 = HTTPS
- 53 = DNS

These are standard conventions, not hard rules — a service can be configured to run on a different port (sometimes done deliberately for security).

## Scanning with Nmap
Installed nmap and scanned the VM to see what ports were actually open:
```
sudo nmap localhost
```
Result showed port 22 (ssh) and 631 (ipp — CUPS printing) open, matching what was already known from earlier netstat output.

## Configuring UFW (Firewall)
```
sudo ufw allow ssh
sudo ufw enable
sudo ufw status
```
Allowed SSH *before* enabling the firewall — enabling first would have blocked the active SSH connection and locked myself out, a common real-world mistake.

Added an explicit deny rule to test blocking a specific port:
```
sudo ufw deny 631
```

## Key Lesson
Re-ran nmap after adding the deny rule — port 631 still showed as open. This wasn't a failure: loopback traffic (a machine scanning itself via `localhost`/`127.0.0.1`) generally bypasses firewall filtering, since it's treated as trusted local traffic rather than real network traffic. The firewall rule was correctly configured and would block an actual external connection — properly testing it requires scanning from a separate machine on the network, not the same one.
