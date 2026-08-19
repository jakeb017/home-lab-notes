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
