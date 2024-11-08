---
description: A free retired easy machine.
---

# Lame

<figure><img src="../.gitbook/assets/image (83).png" alt=""><figcaption></figcaption></figure>

## Enumeration & Information gathering&#x20;

### Nmap

```bash
$nmap 10.10.10.3 -sV -sC -T4 -oA lame-def-nmap -Pn --disable-arp-ping

# -Pn --disable-arp-ping: skip host discovery phase
```

<figure><img src="../.gitbook/assets/image (19).png" alt=""><figcaption><p>SSH FTP SMB</p></figcaption></figure>

<img src="../.gitbook/assets/file.excalidraw (2).svg" alt="Illustration of the nmap output." class="gitbook-drawing">

### FTP

Usually, we choose one of the protocols and use all the footprinting techniques then move on to the next one. In this case, we're attacking FTP since we have a command to directly download all of available file (anonymous login).

<figure><img src="../.gitbook/assets/image (20).png" alt=""><figcaption><p>Nothing.</p></figcaption></figure>

FTP contains nothing even with ls -a it shows no hidden files so yes it provides nothing. But, upon checking for any public vulnerabilities, we ran into an interesting one:

{% embed url="https://www.exploit-db.com/exploits/49757" %}
vsFTPd 2.3.4 CVE
{% endembed %}

After trying the exploit it turns out it does not work, probably some firewall is blocking our connection. So yeah, we move on.

### SMB

As provided in [(139,445)SMB](https://app.gitbook.com/s/2x96g0dngcMbgd6cnWUj/enumeration-and-attack-planning/footprinting/host-based-enumeration/139-445-smb "mention") we will try the commands one by one and see if we got access to anything.

We normally try out `rpcclient` but this time, since it did not provide anything useful and is too slow to respond, we jumped to the next command: `smbmap`

<figure><img src="../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

The comment on the `tmp` share was interesting enough for me to try and access it even when it shows that i got no permissions to any of the shares. But as shown above i entered it anyways and after poking around a bit, nothing interesting.

Time to look or any public vulnerabilities. And not so long before we found this:

{% embed url="https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2007-2447" %}
SMB CVE
{% endembed %}

Following the exploit found on GitHub we easily get a root shell.

<figure><img src="../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

As  or the user flag, a simple find command would provide you with its location. try to to add another option to this command so that it prints out the user flag directly. `$find / -name user.txt -type f 2>/dev/null`

{% hint style="warning" %}
* Generally, we should always look for any public vuln first, but we're here to learn not just root the machine, so we should try out our tools and why not stumble upon one or two rabbit holes before we find our way through the machine!
* We did not look for upgrading the shell we got nor upgrade it because we won't be there for long since we already got root access, we could've used the python command provided in [id-3.-upgrading-tty](https://app.gitbook.com/s/2x96g0dngcMbgd6cnWUj/introduction-and-getting-started/shells-and-ssh/setting-up-shells#id-3.-upgrading-tty "mention") or use SSH by adding a private key on the target host and use our public key to access it as provided in [SSH for Pentesters:](https://app.gitbook.com/s/2x96g0dngcMbgd6cnWUj/introduction-and-getting-started/shells-and-ssh#ssh-for-pentesters "mention")
{% endhint %}
