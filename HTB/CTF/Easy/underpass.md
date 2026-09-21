<h1>UNDERPASS WRITE UP</h1>

<img width="701" height="388" alt="underpass" src="https://github.com/user-attachments/assets/7366e248-6e6c-4a01-98b1-87c1cfa38329" /></br>

<h2>MACHINE INFORMATION</h2>

<b>Difficulty:</b> Easy<br>
<b>Operating System:</b> Linux<br>
<b>IP Address:</b> 10.10.11.48<br>
<b>Hostname:</b> underpass.htb<br>
<b>Open Ports:</b> 22/tcp, 80/tcp, 161/udp, 1812/udp, 1813/udp<br>
<b>Web Server:</b> Apache2 v2.4.52(Ubuntu)<br>
<b>Addition:</b> DaloRADIUS, Mosh<br>

<h2>SUMMARY</h2>

Underpass is starting with a default Apache Ubuntu page. This leads me to a UDP ports enumeration. As a result of an enumeration I find SNMP service up and discover 'DaloRADIUS' is running on the machine. Machine operators panel can be accessed with default credentials. Inside the panel I find there is a user name called 'svcMosh' and password for this user was hashed type MD5. I crack the hash and use this credentials for SSH connection and get the user flag. 'svcMosh' user configured to run 'mosh-server' as 'root'. I use this server connection and it allows me to connect to the server from their local machine as 'root' and with that I get root flag.

<h3>STEP BY STEP</h3>

<b>1-</b> First start with nmap scan

<code>nmap -sC -sV 10.10.11.48</code>

Result:

<img width="760" height="281" alt="ss1" src="https://github.com/user-attachments/assets/9d94f2b7-1d95-45c0-bb66-fcd259af0eb4" /></br>

As I can see port 22(SSH) and 80(HTTP) are open.

<b>2-</b> visit "10.10.11.48:80"

<img width="1366" height="687" alt="ss2" src="https://github.com/user-attachments/assets/527fa75a-3796-467a-ac5d-14e314644656" /></br>

On port 80 there is just Default Apache Ubuntu page here. There is nothing as "/robots.txt" or anything in source code.

<b>3-</b> nmap scan for udp ports

<code>nmap -sU 10.10.11.48</code>

<b>NOTE:</b> UDP scans take much longer time than TCP scans

Result:

<img width="539" height="191" alt="ss3" src="https://github.com/user-attachments/assets/6ddbe7dd-3d4c-484c-a970-39d7a3784b8d" /></br>

There it is on port 161 open snmp service.

<b>4-</b> I just want to discover what snmp service can give me

<code>snmpwalk -v 2c -c public 10.10.11.48</code>

Result:

<img width="952" height="133" alt="ss4" src="https://github.com/user-attachments/assets/325a0da2-f3f0-499a-beed-2eb51bb3fe99" /></br>

I got two things from this result. First one the machine's host name is "underpass.htb". Second one "UnDerPass.htb is the only daloradius server in the basin!" give me a clue about server running daloRADIUS.

<b>5-</b> first things first, lets add hostname to our machine

<code>sudo vim /etc/hosts</code>

<img width="477" height="153" alt="ss5" src="https://github.com/user-attachments/assets/812d768c-8307-4b7c-b769-81b34f6e122b" /></br>

From now on i can reach the server through "underpass.htb" host name

<b>6-</b> I start to research about daloRADIUS and find github page. That github page gives me some directory to search in server. When I find github page about running projects, I first try to reach README.md if it is possible

<img width="1366" height="689" alt="ss6" src="https://github.com/user-attachments/assets/5540b194-797c-4159-a554-9a06a35598e6" /></br>

<b>7-</b> As my research continue, I find a login mechanism on "http://underpass.htb/daloradius/app/operators/login.php". I try to default credentials for login the system.

<img width="1366" height="688" alt="ss7" src="https://github.com/user-attachments/assets/86d5be27-6b30-4904-aa70-c652b048864c" /></br>

And it works!!!

<b>8-</b> From now on I can access the management panel.

<img width="1366" height="688" alt="ss8" src="https://github.com/user-attachments/assets/5fe32ad7-9283-4c9d-b387-8ac037faa8d6" /></br>

And I can see on "Users" tab, system has a user

<b>9-</b> When I look at "Users" tab, I find a user with the username "svcMosh" and password "412DD4759978ACFCC81DEAB01B382403"

<img width="1366" height="687" alt="ss9" src="https://github.com/user-attachments/assets/168ed9eb-08e1-4082-8d4d-ec421e705db3" /></br>

<b>10-</b> When I see that kind of password, I directly think about hashing. And I know this is MD5 hashing. So I use hashcat for that

<code>echo "412DD4759978ACFCC81DEAB01B382403" > svcMosh_hash && hashcat -m 0 -a 0 svcMosh_hash /usr/share/wordlists/rockyou.txt</code>

Result:

<img width="426" height="143" alt="ss10" src="https://github.com/user-attachments/assets/349970d9-be2c-49e7-879c-82ae3c5deb70" /></br>

<b>11-</b> For an alternative solution, I can also use "crackstation.net" for this kind of situation. CrackStation can even detect type of hash with easy usage.

<img width="1366" height="606" alt="ss11" src="https://github.com/user-attachments/assets/0b9a5133-b80b-4751-aaf8-01153ad5b1a6" /></br>

<b>12-</b> As a result, I have the username "svcMosh" and now as a password "underwaterfriends". Now I can try to connect to SSH with this credentials.

<code>ssh svcMosh@underpass.htb	password:underwaterfriends</code>

<img width="958" height="531" alt="ss12" src="https://github.com/user-attachments/assets/496a884a-4ba3-430a-8958-98025f1a702f" /></br>

And Yes!!! I can reach user flag from "user.txt"

<b>13-</b> After that I need to collect information about current user's permissions and belongings to a group etc. For that

<code>id</code>

<code>sudo -l</code>

Result:

<img width="1026" height="153" alt="ss13" src="https://github.com/user-attachments/assets/6336ff20-d63d-4fb7-a8a1-5bdb0ae1d867" /></br>

<b>14-</b> I can see current user has sudo permission on "/usr/bin/mosh-server". So I try to use that

<code>sudo /usr/bin/mosh-server</code>

Result:

<img width="639" height="212" alt="ss14" src="https://github.com/user-attachments/assets/66dc05b9-391a-425c-baca-70c4e98124e3" /></br>

<b>15-</b> When I look for documentation about "Mosh", I find a github page. With "mosh-server" command I start mosh server on remote machine local server on port 60001 and it gives me a key value "ynRu02EHiu2mSnecOpVwhw". From now on I just need to connect to that mosh server as a client. For that

<code>MOSH_KEY=ynRu02EHiu2mSnecOpVwhw mosh-client 127.0.0.1 60001</code>

Result:

<img width="950" height="502" alt="ss15" src="https://github.com/user-attachments/assets/96fa84a8-6da3-4fd5-b96d-3cec511d2826" /></br>

And as you can see, I get root access to terminal. This is how I get root flag

<h2>RESOURCE</h2>
<ul>
<li><a href="https://github.com/lirantal/daloradius">DALORADIUS</a></li>
<li><a href="https://github.com/mobile-shell/mosh">MOSH SERVER</a></li>
</ul>
