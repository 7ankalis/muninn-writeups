# Mr robot

## Enumeration & Footprinting

### Nmap

Beginning with nmap, discovering 443/80 ports open.

<figure><img src="../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

```shell-session
$ sudo sh -c 'echo "10.10.175.73 mrrobot.thm" >> /etc/hosts'
```

At this point i want to let a UDP scan run in the background while i continue with the enumeration:

```shell-session
$ sudo nmap 10.10.175.73 -sU -oN nmap-initial-scan-UDP -vv -T4
```

Upon poking around the website a bit, there's nothing in there even intercepting the mail requests sent through the join command in the website lead to nothing.

### Directories/Subdomains fuzzing

#### Directories

```shell-session
$ ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://mrrobot.thm/FUZZ
```

<figure><img src="../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

Even tho there's a potential login portal, i prefer reading the content of robots and readme as they give me more information before going in and testing the login page. And Indeed they did:

<figure><img src="../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

{% hint style="success" %}
First key acquired.
{% endhint %}

#### Subdomains

For subdomains, we got no hits so I'm going to poke around the directories we found.

<figure><img src="../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

Going for the RSS and Comments we stumble upon a version number. We'll see if we're going to need it.

<figure><img src="../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

Checking the login portal, we got LOTS and LOTS of possible scripts that works on the 4.3.1 version but it seems like a rabbit hole for now.

<figure><img src="../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

Checking the `/license` directory to find this :&#x20;

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

Scrolling down a bit to find a base64 encoded string :&#x20;

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

## Exploitation

### Foothold

Which logs us in to the WordPress site and we can confirm from the bottom of the page that it is indeed version 4.3.1

<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

After successfully logging in and poking around the panels we find this file in the Appearance Editor that's loaded whenever we access the user's blog we visited earlier (https://mrrobot.thm/image).

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

We added our [reverse shell](https://app.gitbook.com/s/2x96g0dngcMbgd6cnWUj/introduction-and-getting-started/shells-and-ssh/setting-up-shells#reverse-shells) at the end of the file, then saved it and accessed https://mrrobot.thm/image and we successfully got a hit back to our nc listener:

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

## Post Exploitation

### Lateral Movement

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

So we simply throw it to hashcat :

```shell-session
$ hashcat -m 0 -a 0 c3fcd3d76192e4007dfb496cca67e13b /usr/share/eaphammer/wordlists/rockyou.txt

c3fcd3d76192e4007dfb496cca67e13b:abcdefghijklmnopqrstuvwxyz
```

Then switching to robot user:

<figure><img src="../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

{% hint style="success" %}
Second Key acquired.
{% endhint %}

### Privilege Escalation

First things first, i usually look at the kernel and system information. But it lead us to nothing, kernel is "secure".

After poking around a bit in the filesystem we found no interesting stuff. Right before looking for files with SUID bit:

```shell-session
$ find / -perm /4000 -type f 2>/dev/null
```

<figure><img src="../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
The SUID bit in sudo is usually intended since sudo allows users to execute commands as root. In our case, `sudo -l` lead us to nothing, `sudo su` too. So we look for the others; specifically nmap binary.
{% endhint %}

<figure><img src="../.gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>

It got to my attention that interactive mode i didn't know nmap had one, so poking around here's what we found:

<figure><img src="../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

Since this binary is run as root (SUID bit) the shell commands we will run will also run as root since a child process inherits its parent's group and user id:

<figure><img src="../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

{% hint style="success" %}
Third key acquired.
{% endhint %}
