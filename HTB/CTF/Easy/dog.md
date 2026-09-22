<h1>DOG WRITE UP</h1>

{dog.png}<br>

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

{ss1.png}<br>

As I can see, port 22(SSH), 80(HTTP) and 5555 are open. The system uses "BackdropCMS" and there is a "/.git/" directory. Also, on port 5555, "SimpleHTTPServer 0.6 (Python 3.8.10)" is running.

<b>2-</b> I visit "10.10.11.58:80".

{ss2.png}<br>

In the About page, I find "support@dog.htb". This indicates that the host name is "dog.htb".

{ss3.png}<br>

<b>3-</b> I add "dog.htb" host name to my "/etc/hosts" file.

<code>echo "10.10.11.58 dog.htb" | sudo tee -a /etc/hosts</code>

<b>4-</b> There is nothing useful to do in the web application, so I visit "http://dog.htb/.git/".

{ss4.png}<br>

<b>5-</b> To gather more information, I use "git-dumper" to download all the files to my system.

<code>mkdir dog_git_files</code>

<code>git-dumper http://dog.htb/.git ./dog_git_files</code>

Result:

{ss5.png}<br>

And there it is, all source code and configuration files.

<b>6-</b> In the "settings.php" file, there is important information. First, I find the database configuration.

{ss6.png}<br>

As I can see, the system uses MySQL and the database credentials are:

<code>username= root</code>

<code>password= BackDropJ2024DS2024</code>

<b>7-</b> I also find some paths for configuration files in this file.

{ss7.png}<br>

To get more information, I need to visit this path.

<b>8-</b> When I visit the path, I find a bunch of JSON files.

{ss8.png}<br>

<b>9-</b> I need to find a valid username. For that:

<code>cat * | grep "htb"</code>

Result:

{ss9.png}<br>

So I find a valid username:

<code>username= tiffany</code>

<b>10-</b> I log in to the web application with "tiffany" as the username and "BackDropJ2024DS2024" as the password.

{ss10.png}<br>

Successfully logged in to the system.

<b>11-</b> I start researching "BackdropCMS" to find any vulnerabilities. I find this exploit:

<a href="https://www.exploit-db.com/exploits/52021">https://www.exploit-db.com/exploits/52021</a>

<b>12-</b> According to the exploit source, I must upload my module file to the system. For that, I use the "Manual Installation" option at the "/modules/install" path.

{ss11.png}<br>

After that, I upload my ZIP file to the system and access it through "/zip_file_name/reverse_shell_name.php".

<b>13-</b> First, I need a reverse shell. I prefer to use the "PentestMonkey" reverse shell and name it "php-reverse-shell.php". In the code, I must change the IP address and port to my own values.

{ss12.png}<br>

Second, I need an info file for the module upload.

{ss13.png}<br>

I create a file called "fnwn_rev" and put the reverse shell and info file into this directory. After compressing the directory, the file is ready to upload to the system.

<b>14-</b> I compress the file.

<code>tar -xcvf fnwn_rev.tar.gz fnwn_rev</code>

Result:

{ss14.png}<br>

<b>15-</b> Now I start listening on my system.

<code>nc -lvnp 8282</code>

I upload the file and then visit "/modules/fnwn_rev/php-reverse-shell.php". I should receive a response from the server in my Netcat listener.

<b>16-</b> When I upload the file to the system:

{ss15.png}<br>

I visit the "/modules/fnwn_rev/php-reverse-shell.php" path and get a response.

Result:

{ss16.png}<br>

And I got the reverse shell!!

<b>17-</b> I find the user flag in the system, but I do not have permission to read it yet.

{ss17.png}<br>

<b>18-</b> So I try to connect via SSH as the "johncusack" user with the password "BackDropJ2024DS2024".

<code>ssh johncusack@dog.htb</code>

Result:

{ss18.png}<br>

And this is how I get the user flag!!!

<b>19-</b> I look at the sudo permissions for the "johncusack" user.

<code>sudo -l</code>

Result:

{ss19.png}<br>

As I can see, there is a command called "bee" that can be run with sudo privileges.

<b>20-</b> I try to run the "bee" command and it shows me the helper function list. There are two important sections that catch my attention.

First, the "Global Options" section:

{ss20.png}<br>

Second, the "ADVANCED" section:

{ss21.png}<br>

So I start researching how to use it.

<b>21-</b> I find the <a href="https://github.com/backdrop-contrib/bee">GitHub page</a> about "bee" command usage. According to the documentation, I need to specify the "--root" flag and then I can use the "eval" command with a script to execute.

<a href="https://github.com/backdrop-contrib/bee/blob/1.x-1.x/docs/Usage.md">https://github.com/backdrop-contrib/bee/blob/1.x-1.x/docs/Usage.md</a>

<b>22-</b> So my command is:

<code>sudo bee --root=/var/www/html eval "system('cat /root/root.txt');"</code>

Result:

{ss22.png}<br>

This is how I get the root flag!!!

<b>NOTE:</b> I alter the command to avoid giving the flag value directly.

<h2>RESOURCE</h2> <ul> <li><a href="https://www.exploit-db.com/exploits/52021">BACKDROP CMS EXPLOIT</a></li> <li><a href="https://github.com/backdrop-contrib/bee/blob/1.x-1.x/docs/Usage.md">BACKDROP CMS BEE DOCUMENTATION</a></li> </ul>
