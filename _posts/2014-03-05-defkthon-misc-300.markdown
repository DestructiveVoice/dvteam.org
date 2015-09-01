---
layout: post
title:  "DEFKTHON 2014 Misc 300"
date:   2014-03-05
categories: writeups defkthon 2014 misc 300
author: phil9l
comments: true
---

Try to extract downloaded *73168.zip*. It's password protected, but we can see that there is only *46783.zip* file inside. So try password *46783*.

![]({{ site.baseurl }}/assets/2014-03-05-defkthon-misc-300/screen_01.png){: .center-image }
<!-- more -->

It's successfuly unpacked. Try to do the same with *46783.zip*. Now we have the next one - *47096.zip*. Unpacking by hands seems too hard so write the python script:

{% highlight python linenos=table %}
#!/usr/bin/env python2
# -*- coding: utf-8 -*-

import zipfile

sFile = '42819.zip'
while True:
  tz = zipfile.ZipFile(sFile).namelist()
  if (len(tz) > 1):
  print '[!]{0} contains more then one file!'.format(sFile)
  if tz[0][-4:] == '.zip':
  tzip = zipfile.ZipFile(sFile, "r")
  tzip.setpassword(tz[0][:-4])
  tzip.extractall()
  tzip.close()
  print '[+]{0} was unpacked successfuly.'.format(sFile)
  sFile = tz[0]
  else:
  print '[!]Finished.'
{% endhighlight %}

Wait about 40 minutes. Than we have 1.510 archives and there is *mess.wav* file in the last one. Brute password for this archive (it is "**b0yzz**").

Try to listen *mess.wav*, undestand that we can hear nothing. Try to vi this file, nothing interesting again. Let's now try to see it's spectogram. We can now see the key here - "**BallsRealBolls**".

![]({{ site.baseurl }}/assets/2014-03-05-defkthon-misc-300/screen_02.png){: .center-image }
