**Platfrom:** Hack The Box(HTB)

**Lab Name:** Footprinting Lab

**Difficulty:** Hard

**Date Completed:** 2026-05-10

**Attacker OS:** Kali Linux

**Target OS:** Ubuntu Linux

**Target IP:** 10.129.202.20

**Goal:** Find the HTB password

---

**Tools Used:**
- nmap
- onesixtyone
- snmpwalk
- curl
- ssh
- mysql

---

**Port Scanning:**

&nbsp;&nbsp;&nbsp;&nbsp;I started by splitting the TCP scans into two separate runs. The reason is simple — running them together is slow. First, Scan all the ports and say which prots are open.
<br>

```bash
nmap -p- --min-rate 5000 -T4 10.129.202.20
```

<img width="312" height="241" alt="h-1" src="https://github.com/user-attachments/assets/96844d2a-e656-408d-a604-5b6bec68cd96" />

<br>
&nbsp;&nbsp;&nbsp;&nbsp;Get more detail about the open ports.
<br><br>


```bash
nmap -sV -sC -p 22,110,143,993,995 10.129.202.20
```

<img width="340" height="656" alt="h-2" src="https://github.com/user-attachments/assets/c0f3777f-6138-4a1f-8b33-14c187ab3bcb" />

<br>
&nbsp;&nbsp;&nbsp;&nbsp;I could see POP3, POP3s, IMAP and IMAPS are the open ports. these are all ports used for reading email. But the port used for sending email, which is SMTP was missing. SMTP also use UDP port to send emails. let scan all the ports by UDP.
<br><br>

```bash
nmap -p- -sU --min-rate 5000 -T4 10.129.202.20
```

<img width="302" height="227" alt="h-3" src="https://github.com/user-attachments/assets/bfc221c4-31e3-4d13-979a-b33a3d6b2848" />

<br>
&nbsp;&nbsp;&nbsp;&nbsp; SMTP port was open in UPD scan. Let's get more detail
<br><br>

```bash
nmap -sV -sC -sU -p 161 10.129.202.20
```

<img width="343" height="250" alt="h-4" src="https://github.com/user-attachments/assets/58464a8d-cf03-4863-8796-b81de91ff6c3" />

---

**SNMP Enumeration:**

**NOT COMPLETED**
