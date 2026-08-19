# home-lab-notes
Documenting my home lab setup and IT/security learning projects
<img width="1919" height="1020" alt="image" src="https://github.com/user-attachments/assets/34f8ac59-85a8-4b4e-a785-16bb5414f966" />
Set up Ubuntu 25.04 as a VM in VirtualBox for my home lab environment. Allocated 4GB RAM and 2 CPU cores, 25GB storage on a SATA controller. Chose NAT networking to keep it isolated from my host network while still having internet access for updates. Next step is configuring static IP and setting up SSH access.
<img width="911" height="909" alt="image" src="https://github.com/user-attachments/assets/2fff332a-fd95-4893-a0c4-052e5cae121d" />
Ran sudo apt update to refresh package lists, then installed neofetch with sudo apt install neofetch to test package management. Confirmed it worked by running neofetch and viewing system info output. Removed it afterward with sudo apt remove neofetch, and learned that this only removes the specified package, not its dependencies, which apt flags separately and requires sudo apt autoremove to clean up properly.
<img width="878" height="587" alt="image" src="https://github.com/user-attachments/assets/4b0c1761-2e32-42e4-98aa-0d022a9d1dd8" />
Used ping google.com to test internet connectivity from the VM, confirming DNS resolution worked by resolving google.com to an IP address automatically, and verified low latency around 27ms average with minor packet loss on this test. Learned that ping is typically the first diagnostic step for connectivity troubleshooting in a help desk scenario.
