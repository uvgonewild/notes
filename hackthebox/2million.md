# TwoMillion (HackTheBox)

target Ip : `10.10.11.221`

target domain : `2million.htb`

## Enumeration

### Nmap Scan

```bash
nmap -A -p 22,80 2million.htb
```

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)

80/tcp open  http    nginx
|_http-trane-info: Problem with XML parsing of /evox/about
|_http-title: Hack The Box :: Penetration Testing Labs
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

`httponly` flag is **not** set, maybe `XSS ( Cross Site Scripting )` will be our way to go!

### Directory Enumeration

```bash
gobuster dir -u http://2million.htb -w directory-list-2.3-medium.txt
```

```bash
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://2million.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================

Error: the server returns a status code that matches the provided options for non existing urls. http://2million.htb/4f719188-bd12-49f2-92f4-38bcad7eef9f => 301 (Length: 162). To continue please exclude the status code or the length
```

Interesting, If the server returns a status code that mathces the code for non existing url's, then for every url request sent to the server will be of `same length`. So maybe I'll just do what it says.

```bash
gobuster dir -u http://2million.htb -w directory-list-2.3-medium.txt --exclude-length 162
```

```bash
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://2million.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] Exclude Length:          162
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/home                 (Status: 302) [Size: 0] [--> /]
/login                (Status: 200) [Size: 3704]
/register             (Status: 200) [Size: 4527]
/api                  (Status: 401) [Size: 0]
/logout               (Status: 302) [Size: 0] [--> /]
/404                  (Status: 200) [Size: 1674]
/0404                 (Status: 200) [Size: 1674]
/invite               (Status: 200) [Size: 3859]
```

the `/home` and `/logout` page are redirecting to the `root directory`

### Webiste Features

potential attack vectors might be

- /login
  
- /register
  
- /invite
  

The `/invite` page contained 3 scripts, and two were obfuscated & were in file.

I used `javascript deobfuscator` to deobfuscate and unpack the javascript code on the `/invite` page but didn't dare to look through it for it is an `easy machine`.

![Capture](https://github.com/user-attachments/assets/f75c993d-855a-415b-be85-2dd85588a761)

**Woah Woah Woah!** I was just looking for the `hackthebox` website and since this is a retired machine, a guided mode is avaialable. I tried answering questions and the questions itself target to `scripts`.

![Capture2](https://github.com/user-attachments/assets/e3859f90-467a-4e3d-babd-aa73ac2ad975)

The question asks for an function name. The `/js/inviteapi.min.js` & the code in the page itself don't have any specific function names. I have already overviewed the first script which is about `5000-6000 lines`. There is something that I'm missing!

![Capture3](https://github.com/user-attachments/assets/dacf6ca5-b292-439c-9d4b-2e6f2c708700)

Well I did miss this! This means that `/invite` is our attack vector, but this doesn't solve our problem!

Alright! Finally found the the function name. I tried Java deobfuscator to simplify the code, but didn't work out but when I asked same to chatGPT it gave out the following result for the `/js/inviteapi.min.js`

```javascript
eval(function(p,a,c,k,e,d){e=function(c){return c.toString(36)};if(!''.replace(/^/,String)){while(c--){d[c.toString(a)]=k[c]||c.toString(a)}k=[function(e){return d[e]}];e=function(){return'\\w+'};c=1};while(c--){if(k[c]){p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c])}}return p}('1 i(4){h 8={"4":4};$.9({a:"7",5:"6",g:8,b:\'/d/e/n\',c:1(0){3.2(0)},f:1(0){3.2(0)}})}1 j(){$.9({a:"7",5:"6",b:\'/d/e/k/l/m\',c:1(0){3.2(0)},f:1(0){3.2(0)}})}',24,24,'response|function|log|console|code|dataType|json|POST|formData|ajax|type|url|success|api/v1|invite|error|data|var|verifyInviteCode|makeInviteCode|how|to|generate|verify'.split('|'),0,{}))
```

```javascript
function verifyInviteCode(code) {
    var formData = { "code": code };
    $.ajax({
        type: "POST",
        dataType: "json",
        url: '/api/v1/verifyInviteCode',
        data: formData,
        success: function(response) {
            console.log(response);
        },
        error: function(response) {
            console.log(response);
        }
    });
}

function makeInviteCode() {
    $.ajax({
        type: "POST",
        dataType: "json",
        url: '/api/v1/makeInviteCode',
        success: function(response) {
            console.log(response);
        },
        error: function(response) {
            console.log(response);
        }
    });
}
```

The script is defined but never called on the Client side! Lets call the function `makeInviteCode`

![Capture4](https://github.com/user-attachments/assets/6144d85f-064b-4cd5-9cc8-83ed0b75debb)

```json
Enctype: "ROT13"
Data: "Va beqre gb trarengr gur vaivgr pbqr, znxr n CBFG erdhrfg gb /ncv/i1/vaivgr/trarengr"
DecodedValue: "In order to generate the infinite code, make a POST request to /api/v1/infinite/generate"
```

Lets use `curl` to make the post request to `http://2million.htb/api/v1/infinite/generate`

```bash
curl -X POST http://2million.htb/api/v1/infinite/generate
```

```html
<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

`Error Code 301` means that the file which we are looking for is at another location! Lemme go through each directory step by step. the post request is successful until `curl -X POST 2million.htb/api/v1/infinite` which is resulting in error code 301.

*This was definetly not supposed to happen, I am currently looking at a walkthrough & he recieves a different output when doing an POST request to the same url using curl. I might just use the code he figured but if the working of this machine has changed then the code might be too... copying the code would be my last option. For now I'll just Leave it as is.*

---
