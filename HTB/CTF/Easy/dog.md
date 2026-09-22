<h1>DOG WRITE UP</h1>

<img width="694" height="380" alt="dog" src="https://github.com/user-attachments/assets/efd8a2ff-7a99-47ee-82e9-eba89141c083" /><br>

<h2>MACHINE INFORMATION</h2>

<b>Difficulty:</b> Easy<br>
<b>Operating System:</b> Linux<br>
<b>IP Address:</b> 10.10.11.58<br>
<b>Hostname:</b> dog.htb<br>
<b>Open Ports:</b> 22/tcp, 80/tcp, 5555/tcp<br>
<b>Web Server:</b> Apache v2.4.41(Ubuntu)<br>
<b>Addition:</b> BackdropCMS<br>

<h2>SUMMARY</h2>

Dog is an easy-rated Linux machine that involves reading sensitive information through an exposed Git repository and discovering credentials that provide administrator access to "BackdropCMS". The administrator privileges allow me to exploit Remote Code Execution by uploading a malicious archive containing a PHP backdoor and gain an initial foothold. The "johncusack" user account also reuses the "BackdropCMS" password. After compromising the "johncusack" account, I find that the user can run the "bee" executable with "sudo" privileges, which allows me to gain root privileges.

<h3>STEP BY STEP</h3>

<b>1-</b> First start with nmap scan.

<code>nmap -sC -sV 10.10.11.58</code>

Result:

<img width="776" height="506" alt="ss1" src="https://github.com/user-attachments/assets/e74205dd-3fba-4a8f-a5c5-6793caeaea45" /><br>

As I can see, port 22(SSH), 80(HTTP) and 5555 are open. The system uses "BackdropCMS" and there is a "/.git/" directory. Also, on port 5555, "SimpleHTTPServer 0.6 (Python 3.8.10)" is running.

<b>2-</b> I visit "10.10.11.58:80".

<img width="1366" height="687" alt="ss2" src="https://github.com/user-attachments/assets/d1d091fb-8164-40e0-8a28-f3f6d012afa8" /><br>

In the About page, I find "support@dog.htb". This indicates that the host name is "dog.htb".

<img width="1366" height="687" alt="ss3" src="https://github.com/user-attachments/assets/2ea983b8-dc44-4188-8425-45be1acf1cac" /><br>

<b>3-</b> I add "dog.htb" host name to my "/etc/hosts" file.

<code>echo "10.10.11.58 dog.htb" | sudo tee -a /etc/hosts</code>

<b>4-</b> There is nothing useful to do in the web application, so I visit "http://dog.htb/.git/".

<img width="594" height="571" alt="ss4" src="https://github.com/user-attachments/assets/34e5bbe6-ca1f-4693-bf17-daccc5ef3257" /><br>

<b>5-</b> To gather more information, I use "git-dumper" to download all the files to my system.

<code>mkdir dog_git_files</code>

<code>git-dumper http://dog.htb/.git ./dog_git_files</code>

Result:

<img width="791" height="96" alt="ss5" src="https://github.com/user-attachments/assets/dfd34d8f-98fe-437b-a8d5-500b39d3d91c" /><br>

And there it is, all source code and configuration files.

<b>6-</b> In the "settings.php" file, there is important information. First, I find the database configuration.

<img width="543" height="43" alt="ss6" src="https://github.com/user-attachments/assets/39ef6dc7-73e4-48e1-bcc2-bb6ef7f7553d" /><br>

As I can see, the system uses MySQL and the database credentials are:

<code>username= root</code>

<code>password= BackDropJ2024DS2024</code>

<b>7-</b> I also find some paths for configuration files in this file.

<img width="743" height="40" alt="ss7" src="https://github.com/user-attachments/assets/db3f45ed-51f6-4704-822f-7b3ea48953fe" /><br>

To get more information, I need to visit this path.

<b>8-</b> When I visit the path, I find a bunch of JSON files.

<img width="1308" height="329" alt="ss8" src="https://github.com/user-attachments/assets/8da32786-6237-49ed-8e41-911c9c393e3d" /><br>

<b>9-</b> I need to find a valid username. For that:

<code>cat * | grep "htb"</code>

Result:

<img width="207" height="43" alt="ss9" src="https://github.com/user-attachments/assets/b407dd3d-7df7-4490-b2a8-1b618064ad03" /><br>

So I find a valid username:

<code>username= tiffany</code>

<b>10-</b> I log in to the web application with "tiffany" as the username and "BackDropJ2024DS2024" as the password.

<img width="1366" height="688" alt="ss10" src="https://github.com/user-attachments/assets/70683e4a-017c-4045-9b0c-05db46703a6a" /><br>

Successfully logged in to the system.

<b>11-</b> I start researching "BackdropCMS" to find any vulnerabilities. I find this exploit:

<a href="https://www.exploit-db.com/exploits/52021">https://www.exploit-db.com/exploits/52021</a>

<b>12-</b> According to the exploit source, I must upload my module file to the system. For that, I use the "Manual Installation" option at the "/modules/install" path.

<img width="1366" height="688" alt="ss11" src="https://github.com/user-attachments/assets/9351542d-603b-4206-bdb6-5c0655a781e9" /><br>

After that, I upload my ZIP file to the system and access it through "/zip_file_name/reverse_shell_name.php".

<b>13-</b> First, I need a reverse shell. I prefer to use the "PentestMonkey" reverse shell and name it "php-reverse-shell.php". In the code, I must change the IP address and port to my own values.

<img width="309" height="44" alt="ss12" src="https://github.com/user-attachments/assets/e39e0f41-4698-4261-a181-9637b426cf2b" /><br>

Second, I need an info file for the module upload.

<img width="629" height="278" alt="ss13" src="https://github.com/user-attachments/assets/86e27a54-496e-4694-aed2-0fc45831f515" /><br>

I create a file called "fnwn_rev" and put the reverse shell and info file into this directory. After compressing the directory, the file is ready to upload to the system.

<b>14-</b> I compress the file.

<code>tar -xcvf fnwn_rev.tar.gz fnwn_rev</code>

Result:

<img width="322" height="80" alt="ss14" src="https://github.com/user-attachments/assets/396a30ce-49ff-483b-aa65-ed2c171f161e" /><br>

<b>15-</b> Now I start listening on my system.

<code>nc -lvnp 8282</code>

I upload the file and then visit "/modules/fnwn_rev/php-reverse-shell.php". I should receive a response from the server in my Netcat listener.

<b>16-</b> When I upload the file to the system:

<img width="572" height="277" alt="ss15" src="https://github.com/user-attachments/assets/78badf8c-bdc2-4a8a-b200-b02ef5e2a9c5" /><br>

I visit the "/modules/fnwn_rev/php-reverse-shell.php" path and get a response.

Result:

<img width="840" height="184" alt="ss16" src="https://github.com/user-attachments/assets/17f2746f-8a59-4e47-b763-ed493f0d22c5" /><br>

And I got the reverse shell!!

<b>17-</b> I find the user flag in the system, but I do not have permission to read it yet.

<img width="277" height="165" alt="ss17" src="https://github.com/user-attachments/assets/363cb13f-1abb-44b6-8c62-479b72da1b56" /><br>

<b>18-</b> So I try to connect via SSH as the "johncusack" user with the password "BackDropJ2024DS2024".

<code>ssh johncusack@dog.htb</code>

Result:

<img width="838" height="575" alt="ss18" src="https://github.com/user-attachments/assets/62a3e222-efc8-473f-81fe-ae3ef8648406" /><br>

And this is how I get the user flag!!!

<b>19-</b> I look at the sudo permissions for the "johncusack" user.

<code>sudo -l</code>

Result:

<img width="953" height="120" alt="ss19" src="https://github.com/user-attachments/assets/ef8f386b-d9bb-4b3b-b294-b73a471d91f3" /><br>

As I can see, there is a command called "bee" that can be run with sudo privileges.

<b>20-</b> I try to run the "bee" command and it shows me the helper function list. There are two important sections that catch my attention.

First, the "Global Options" section:

<img width="1363" height="337" alt="ss20" src="https://github.com/user-attachments/assets/4569e932-d877-4ac7-8986-52b1c0d0e81f" /><br>

Second, the "ADVANCED" section:

<img width="628" height="271" alt="ss21" src="https://github.com/user-attachments/assets/43ba5058-d6ad-4023-8083-a79c760d2caa" /><br>

So I start researching how to use it.

<b>21-</b> I find the <a href="https://github.com/backdrop-contrib/bee">GitHub page</a> about "bee" command usage. According to the documentation, I need to specify the "--root" flag and then I can use the "eval" command with a script to execute.

<a href="https://github.com/backdrop-contrib/bee/blob/1.x-1.x/docs/Usage.md">https://github.com/backdrop-contrib/bee/blob/1.x-1.x/docs/Usage.md</a>

<b>22-</b> So my command is:

<code>sudo bee --root=/var/www/html eval "system('cat /root/root.txt');"</code>

Result:

<img width="607" height="33" alt="ss22" src="https://github.com/user-attachments/assets/ff20a108-d718-4051-a04f-33d69bd8ed34" /><br>

This is how I get the root flag!!!

<b>NOTE:</b> I alter the command to avoid giving the flag value directly.

<h2>RESOURCE</h2> <ul> <li><a href="https://www.exploit-db.com/exploits/52021">BACKDROP CMS EXPLOIT</a></li> <li><a href="https://github.com/backdrop-contrib/bee/blob/1.x-1.x/docs/Usage.md">BACKDROP CMS BEE DOCUMENTATION</a></li> </ul>
