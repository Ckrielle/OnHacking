(I'm gonna organize things later, just dropping things I know)


## Local File Inclusion
### Basic Info
Local File Inclusion (LFI) occurs (as most do) because of user-supplied input without proper validation. More specifically,it is the process of including file 
that are on the server through vulnerable inclusion procedure. For example when a page receives the path of a file as input and it doesn't sanitize it, allowing
directory traversal.

LFI is common in [web template engines](https://en.wikipedia.org/wiki/Comparison_of_web_template_engines). Websites want to keep some parts of their functions
when changing pages, like headers, navigation bars, etc.. That is why you can see a parameter like /index.php?page=info. index.php will dynamically load
php scripts (e.g. info.php, headers.php, ...). But as we can control the url, we can specify whichever file we want our script to load. Another place is when
specifying languages. If you see something like ?lang=en, it grabs file from the /en/ dir.

### Finding LFI
LFI can often be found when paths or filenames are passed to include statements with GET parameter requests, like so:
```
include($_GET['lang']);
```
Below are some common occurences of LFI:

#### Basic LFI

Let's imagine this url

http://example.com?lang=gr.php

We can suspect that in the backend, PHP executes the above include command. So we can change the filename to a spicier one

http://example.com?lang=/etc/passwd (If we were testing a Windows server, we'd check C:\Windows\boot.ini)

#### Directory Traversal LFI

Sometimes the below code might be implemented
```
include("./langs/" . $_GET['lang']);
```
Meaning we already have a transfixed path from where the site will search for our supplied file. The above payload won't work. In this case, we will abuse the
parent directory of every child directory, i.e. we will supply ..

http://example.com?lang=../../../../../../../../../etc/passwd

The number of parent directories we give doesn't matter

#### Modified Directory Traversal LFI

Some developers may know of the above, and so do something like this
```
include("lang_" . $_GET['language']);
```
Now if we give as input the previous payload, we will end up with this lang_../../../../../../../../../etc/passwd. Again, the bypass is pretty simple. We just
add / to the beginning of the supplied path

http://example.com?lang=/../../../../../../../../../etc/passwd



If a website does CURL requests and you can specify not only the website, but also the protocol, you can use the file protocol to access any file on the server
