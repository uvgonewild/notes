# Lo-Fi (TryHackMe)

target IP : `10.10.49.28`

target Domain Name : `lofi.thm`

---

## Enumeration

### Nmap Scan

```bash
nmap -A lofi.thm
```

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Uuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 61:38:e3:38:32:a3:17:fd:50:60:51:9b:68:7b:1e:f7 (RSA)
|   256 ac:89:fa:35:63:c9:b8:4b:4a:e8:b1:3e:8e:47:ab:e0 (ECDSA)
|_  256 fa:60:83:b2:88:48:79:44:9a:58:3b:66:00:d3:9f:44 (ED25519)
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)                                                                            |_http-title: Lo-Fi Music                                                                                               Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel 
|_http-title: Lo-Fi Music
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### SSH Enumeration (22)

```bash
ssh root@10.10.49.28
```

```bash
root@10.10.49.28: Permission denied (publickey)
```

I guess that SSH is not what I should be looking for!

### Directory Enumeration

```bash
gobuster dir -u http://lofi.thm -w seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
```

Nothing Interesting!

### Website Features

Looked at the `Source Code` but found nothing interesting. Went through Different `web pages` and looked at the url.

![Capture2](https://github.com/user-attachments/assets/b9fe6204-59b7-47e5-8059-22f651865ad4)

The `url` seemed to be vulnerable, and it actually is!

`http://lofi.thm/?page=` is vulnerable to `Path Traversal`!

![Capture3](https://github.com/user-attachments/assets/6b5d39aa-d154-4eee-9c0e-4e81fc818d60)

![Capture](https://github.com/user-attachments/assets/3c3ca326-2360-4a0b-8917-d8cccfdfd2b1)

---

I thought reading the whole scenario again & read this -

![Capture4](https://github.com/user-attachments/assets/76154ef7-9bec-4f50-913e-a6bbc80ef6f4)

so I thought that I might be able to read a file named `flag.txt` in the **root** of the filesystem i.e. where I found the `/etc/passwd` file

![](![flag](https://github.com/user-attachments/assets/68ba4e94-2aca-4835-874c-5c03e3a19c44)

And there is the flag!
