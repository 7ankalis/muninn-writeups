# Wonderland

{% hint style="info" %}
Overall it was a fun box. Before diving into thee walkthrough here's a summary of some of the techniques we'll cover shortly:

The enumeration didn't have anything complicated: <mark style="color:blue;">basic nmap scan, few fuzzing</mark> and we find some <mark style="color:blue;">cleartext credentials</mark> that'll ssh us into the machine as a regular user. The user flag was in a very strange directory (as the comments stated as well)

To get to root, we'll be doing some <mark style="color:blue;">lateral movement</mark> as well. from the foothold as Alice we'll exploit a basic <mark style="color:blue;">Module Hijacking</mark> to gain access to rabbit.

From there, we'll find a binary with a <mark style="color:blue;">SUID bit</mark> that we'll use to gain access to hatter account through a <mark style="color:blue;">PATH Hijacking</mark>.

Finally, for root, back to the basics after some manual inspection, linpeas.sh (after <mark style="color:blue;">File transferring</mark> it to the target) shows <mark style="color:blue;">CAP\_SETUID</mark> available for the Perl interpreter!

Pwned successfully!
{% endhint %}

## Enumeration

```bash
┌──(alyosha㉿Karamazov)-[~/thm/wonderland]
└─$ sudo sh -c 'echo "<IP> wonderland.thm" >> /etc/hosts'
```

### Nmap

```bash
┌──(alyosha㉿Karamazov)-[~/thm/wonderland]
└─$ nmap wonderland.thm -sV -sC -Pn -T4 -oN def-scripts-scan -vv

PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8e:ee:fb:96:ce:ad:70:dd:05:a9:3b:0d:b0:71:b8:63 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDe20sKMgKSMTnyRTmZhXPxn+xLggGUemXZLJDkaGAkZSMgwM3taNTc8OaEku7BvbOkqoIya4ZI8vLuNdMnESFfB22kMWfkoB0zKCSWzaiOjvdMBw559UkLCZ3bgwDY2RudNYq5YEwtqQMFgeRCC1/rO4h4Hl0YjLJufYOoIbK0EPaClcDPYjp+E1xpbn3kqKMhyWDvfZ2ltU1Et2MkhmtJ6TH2HA+eFdyMEQ5SqX6aASSXM7OoUHwJJmptyr2aNeUXiytv7uwWHkIqk3vVrZBXsyjW4ebxC3v0/Oqd73UWd5epuNbYbBNls06YZDVI8wyZ0eYGKwjtogg5+h82rnWN
|   256 7a:92:79:44:16:4f:20:43:50:a9:a8:47:e2:c2:be:84 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBHH2gIouNdIhId0iND9UFQByJZcff2CXQ5Esgx1L96L50cYaArAW3A3YP3VDg4tePrpavcPJC2IDonroSEeGj6M=
|   256 00:0b:80:44:e6:3d:4b:69:47:92:2c:55:14:7e:2a:c9 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAsWAdr9g04J7Q8aeiWYg03WjPqGVS6aNf/LF+/hMyKh
80/tcp open  http    syn-ack Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Follow the white rabbit.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```





