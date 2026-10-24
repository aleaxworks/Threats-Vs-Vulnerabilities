# Threats vs Vulnerabilities

**Course:** IKB21403 – Vulnerability Analysis
**Assignment:** LAB 1 (Week 1)
**Name:** <NUR ALEA BATRISYIA BINTI MOHD FADZLY>
**Student:** <52215251683>

---

## 1. Overview

In information security, the terms **threat** and **vulnerability** are often confused, but they mean different things. Both are measured against the **CIA triad** (Confidentiality, Integrity, Availability), the foundation of information security.

| | **Threat** | **Vulnerability** |
|---|---|---|
| **Definition** | Anything that can cause harm to a system or its data | A weakness in a system that can be exploited |
| **Origin** | Usually external (attacker, malware), but can be internal (insider) | Internal to the system, its design, code, or configuration |
| **Control** | Often outside the organisation's control | Can be found and fixed by the organisation |
| **Question it answers** | *Who or what can hurt us?* | *Where are we weak?* |
| **Remedy** | Detect, deter, and respond | Patch, configure, and harden |

**Simple analogy:** A house key left in the front door lock is the **vulnerability**. A burglar walking down the street is the **threat**. The **risk** is the chance that the burglar notices the key and gets in.

> **Risk = Threat × Vulnerability** (impact is considered as well)

If there is no vulnerability, a threat has nothing to exploit. If there is no threat, a vulnerability may never be abused. Risk appears when the two meet.

---

## 2. Types of Threats

- **Insider threats**: employees or contractors who misuse their access
- **Malware / Ransomware**: malicious software that damages, spies on, or locks systems
- **Botnets**: networks of infected devices controlled remotely
- **Drive-by download attacks**: malware downloaded just by visiting a compromised site
- **Phishing attacks**: deceptive messages that trick users into revealing information
- **DDoS attacks**: flooding a service so legitimate users cannot reach it
- **Exploit kits**: toolkits that automatically look for and exploit known flaws
- **Advanced Persistent Threats (APT)**: long-term, targeted, stealthy intrusions
- **Malvertising**: malicious code delivered through online advertising

## 3. Types of Vulnerabilities

- **Misconfigurations**: default passwords, open ports, wrong permissions
- **Unsecured APIs**: missing authentication or input validation
- **Outdated or unpatched software**: known flaws left unfixed
- **Zero-day vulnerabilities**: flaws unknown to the vendor, with no patch available yet
- **Weak or stolen user credentials**
- **Poor access control / unauthorised access**
- **Misunderstanding the "Shared Responsibility Model"** in cloud environments

---

## 4. How a Threat Exploits a Vulnerability

The **Cyber Kill Chain** shows how a threat actor turns a vulnerability into damage:

1. **Reconnaissance**: gather information about the target (OSINT, social engineering)
2. **Weaponisation**: create a deliverable payload (malware, exploit)
3. **Delivery**: send it to the target (phishing email, USB drop, drive-by download)
4. **Exploitation**: trigger the payload against a **vulnerability** (buffer overflow, SQLi, XSS)
5. **Installation**: install persistent access (rootkit, trojan, backdoor)
6. **Command and Control (C2)**: open a remote-control channel (HTTP/HTTPS, DNS tunnelling)
7. **Actions on Objectives**: achieve the goal (data exfiltration, ransomware, sabotage)

Stage 4 is where the threat and the vulnerability meet. Fixing the vulnerability breaks the chain at this point.

---

## 5. Real-World Examples

### Internet Archive data breach (Oct 2024)
- **Threat:** an attacker who compromised the Internet Archive's website.
- **Impact:** a user authentication database with about 31 million records (email addresses and screen names) was stolen, a loss of **confidentiality**.

### WordPress "Social Network Tabs" plugin (CVE-2018-20555)
- **Vulnerability:** a flaw in a popular plugin that leaked users' linked social media details.
- **Response:** security researcher Baptiste Robert found it and notified Twitter, which helped protect affected accounts. This is an example of ethical (white-hat) disclosure.

---

## 6. Reducing Risk

| Goal | Security measures |
|---|---|
| Reduce **vulnerabilities** | Patch management, security audits and penetration testing, secure configuration, multi-factor authentication, Zero Trust architecture |
| Reduce **threat impact** | Threat hunting, AI-driven threat detection, incident management, employee cybersecurity training |
| Recover and comply | Backups and disaster recovery planning, end-to-end encryption, compliance with industry regulations |

---

## 7. Conclusion

A **threat** is the *danger*; a **vulnerability** is the *weakness* that lets the danger succeed. Organisations cannot always stop threats from existing, but they can find and fix vulnerabilities before attackers exploit them. This is the purpose of vulnerability analysis and ethical hacking.

---

## References

- IKB21403 Vulnerability Analysis, Chapter 1: Introduction to Vulnerabilities and Exploitation (Dr. Shafiza Mohd Shariff; edited by Ts. Herny Ramadhani)
- Lockheed Martin Cyber Kill Chain framework
- MITRE CVE database: https://cve.mitre.org
