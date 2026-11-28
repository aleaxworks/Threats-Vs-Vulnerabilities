# TryHackMe - Pickle Rick Walkthrough

**Name:** NUR ALEA BATRISYIA BINTI MOHD FADZLY  
**Class:** L01-B03  
**Course:** BCT  
**Date:** 28 November 2026

**Platform:** TryHackMe  
**Room:** Pickle Rick  
**Room URL:** https://tryhackme.com/room/picklerick

---

## 1. Target Information

| Item | Details |
|---|---|
| Target Name | Pickle Rick |
| Target IP | 10.49.167.193 |
| Operating System | Ubuntu Linux |
| Web Server | Apache 2.4.41 |
| Port 22 | SSH |
| Port 80 | HTTP |

## 2. Attacker Machine

| Item | Details |
|---|---|
| Machine | TryHackMe AttackBox |
| Operating System | Kali Linux |
| Attacker IP | 10.49.188.191 |
| Tools Used | Nmap, Firefox, Linux Commands |

## 3. Methodology Used

The following methodologies were used:

1. Reconnaissance
2. Network Scanning
3. Website Enumeration
4. Credential Discovery
5. Command Execution
6. Privilege Escalation
7. Ingredient Retrieval

## 4. Step-by-Step Walkthrough

### Step 1: Network Scanning

I used Nmap to identify open ports and running services on the target machine.

**Command:**

```bash
nmap -sC -sV 10.49.167.193
```

**Results:**

| Port | Service | Version |
|---|---|---|
| 22 | SSH | OpenSSH 8.2p1 |
| 80 | HTTP | Apache 2.4.41 |

**Evidence:**

<img width="673" height="543" alt="image" src="https://github.com/user-attachments/assets/b9dc89e6-f80c-4fad-a56f-55102ba8d418" />


### Step 2: Website Enumeration

I accessed the target website using Firefox.

URL: `http://10.49.167.193`

I inspected the HTML source code and discovered a hidden username.

**Username:** `R1ckRul3s`

**Evidence:**

<img width="944" height="489" alt="image" src="https://github.com/user-attachments/assets/edc88603-4966-4c94-9d97-8f414967665d" />

<img width="316" height="181" alt="image" src="https://github.com/user-attachments/assets/f1782b45-48f4-40dd-a2ff-33c37f5bcaa5" />


### Step 3: Inspecting robots.txt

I accessed the robots.txt file to find additional information.

URL: `http://10.49.167.193/robots.txt`

The file revealed a password clue:

**Password Clue:** `Wubbalubbadubdub`

**Evidence:**

<img width="771" height="125" alt="image" src="https://github.com/user-attachments/assets/6ab2c2de-53bc-4f58-b174-d578f5231030" />


### Step 4: Accessing Rick Portal

I used the discovered username and password clue to log in to the Rick Portal.

After successful login, I gained access to the Command Panel.

**Evidence:**

<img width="988" height="427" alt="image" src="https://github.com/user-attachments/assets/482d9bbb-db37-469e-a1a7-4d795fef6d70" />

### Step 5: Command Execution

I executed Linux commands through the Command Panel.

**Command:**

```bash
whoami
```

**Output:**

```text
www-data
```

This showed that the commands were executed under the web server account.

Next, I listed the files in the current directory.

**Command:**

```bash
ls
```

I discovered a file named:

`Sup3rS3cretPickl3Ingred.txt`

I used the `less` command to read the file.

**Command:**

```bash
less Sup3rS3cretPickl3Ingred.txt
```

**First Ingredient:** `mr. meeseek hair`

**Evidence:**

<img width="867" height="420" alt="image" src="https://github.com/user-attachments/assets/044b5a90-29f4-469f-a3ab-8dc713a1f2d3" />
<img width="366" height="197" alt="image" src="https://github.com/user-attachments/assets/f9037360-3629-4b88-8ef9-9480afcf7d06" />
<img width="794" height="229" alt="image" src="https://github.com/user-attachments/assets/6b9bf93b-1d82-47b7-8609-702381da68da" />

### Step 6: Finding the Second Ingredient

I inspected the clue file.

**Command:**

```bash
less clue.txt
```

The clue instructed me to look around the file system for the other ingredient.

I checked the home directory.

```bash
ls -la /home
```

I discovered a directory named `rick`.

Next, I listed its contents.

```bash
ls -la /home/rick
```

A file named `second ingredients` was found.

I used the following command to read it:

```bash
less "/home/rick/second ingredients"
```

**Second Ingredient:** `jerry tear`

**Evidence:**

<img width="581" height="347" alt="image" src="https://github.com/user-attachments/assets/68143a22-d9ac-492b-8468-d62b3d80e44b" />

<img width="823" height="330" alt="image" src="https://github.com/user-attachments/assets/8f4f1c0b-a313-4108-b051-76e9ea64097c" />

### Step 7: Privilege Escalation

I checked the sudo permissions of the current account.

**Command:**

```bash
sudo -l
```

The output showed:

```text
User www-data may run the following commands:
(ALL) NOPASSWD: ALL
```

This indicated that the www-data account could execute commands with root privileges without a password.

I then accessed the root directory.

**Command:**

```bash
sudo ls -la /root
```

I discovered a file named `3rd.txt`.

**Evidence:**

<img width="567" height="413" alt="image" src="https://github.com/user-attachments/assets/a4591d7a-f7bd-4166-b998-be7c90c9e2b9" />

<img width="887" height="353" alt="image" src="https://github.com/user-attachments/assets/31f8e28d-6038-49e3-baab-f6732b423a28" />

### Step 8: Finding the Third Ingredient

I used the following command to read the file.

```bash
sudo less /root/3rd.txt
```

The output revealed the third ingredient.

**Third Ingredient:** `fleeb juice`

**Evidence:**

![Third Ingredient](images/third-ingredient.png)

## 5. Final Answers

All three ingredients were successfully discovered.

| No. | Ingredient |
|---|---|
| 1 | mr. meeseek hair |
| 2 | jerry tear |
| 3 | fleeb juice |

**Room Status:** Completed  
**Points Earned:** 90

**Evidence:**

![Room Completed](images/room-completed.png)

## 6. Security Findings

During this challenge, I identified the following security issues:

1. **Information Disclosure:** A username was exposed in the HTML source code.
2. **Information Disclosure:** A password clue was exposed through robots.txt.
3. **Remote Command Execution:** The web portal allowed commands to be executed on the target server.
4. **Misconfigured Sudo Permissions:** The www-data account had unrestricted sudo permissions, allowing privilege escalation.

## 7. Recommended Security Improvements

- Avoid exposing sensitive information in HTML source code.
- Do not store password clues in publicly accessible files.
- Restrict command execution through web applications.
- Apply the principle of least privilege.
- Review sudo permissions regularly.

## 8. Conclusion

In conclusion, this TryHackMe challenge helped me understand the basic process of penetration testing.

I learned how to perform network scanning using Nmap, inspect website source code, identify exposed information, execute Linux commands and investigate privilege escalation.

The challenge also demonstrated how security misconfigurations can allow attackers to gain higher privileges.

The Pickle Rick room was successfully completed after retrieving all three required ingredients.

## 9. References

- TryHackMe Pickle Rick: https://tryhackme.com/room/picklerick
- Nmap Documentation: https://nmap.org/book/man.html
- Kali Linux Documentation: https://www.kali.org/docs/
