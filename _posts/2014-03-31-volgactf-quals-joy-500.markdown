---
layout: post
title:  "VolgaCTF Quals 2014 Joy 500"
date:   2014-03-31
categories: writeups volgactf quals 2014 joy 500
author: phil9l
comments: true
---

The *[the+monument+rocket.PSD]({{ site.baseurl }}/assets/2014-03-31-volgactf-quals-joy-500/the%2Bmonument%2Brocket.PSD)* was given. Opening it with Photoshop fails. So I checked the file type:

{% highlight bash %}
$ file the+monument+rocket.PSD
the+monument+rocket.PSD: JPEG image data, EXIF standard 2.21
{% endhighlight %}

Well, let's try to open this file as jpeg. It opens succesfully (there's a reduced version below).

![]({{ site.baseurl }}/assets/2014-03-31-volgactf-quals-joy-500/the%2Bmonument%2Brocket.jpeg){: .center-image }
<!-- more -->

After that I tried to view [EXIF data]({{ site.baseurl }}/assets/2014-03-31-volgactf-quals-joy-500/exif_data.txt).

The only useful think here can be the owner name. I have found only a girl in vk but she seemed useless for this task.

The file name looks unusual. I googled it and found some mentions of Samara.

Well, let's now take a closer look to the photo. First, I saw a man with a poster and inscription "Мяги 7". While googling this phrase I found only a [vk post](http://vk.com/wall-9173984_212799). Hm, desription of Samara again, I hoped I can believe them and look for the answer in this city.

![]({{ site.baseurl }}/assets/2014-03-31-volgactf-quals-joy-500/screen2.png){: .center-image }

Ok, let's try to choose some unusual buildings. I found remarkable only a church but for a long time I could not find it. Later I found the number of a house (99 or 199). It can be really useful for us!

![]({{ site.baseurl }}/assets/2014-03-31-volgactf-quals-joy-500/screen3.png){: .center-image }

Maybe now we have everything to solve this task? I tried to mark all 199 houses on map (blue color), all churches (red color). It's easy to find answer now just bruting all the houses and comparing them with photo. But I had a problem: it is less then 5 minutes left to solve this task. I almost resigned but tried to find something in google. And it happened! I found this church! I marked it with a yellow point on the map.

<script type="text/javascript" charset="utf-8" src="//api-maps.yandex.ru/services/constructor/1.0/js/?sid=UYn3wiEw5_OOPzYlJ9Xw-axYDQBLEUld&width=600&height=450"></script>

Very nice, only two houses are near this church but I have about two minutes. The nearest one looks wrong so I thought that the needed house is a green one on the map. Now we only need to find a point that the foreshortening will be the same as the one on the. Obviously, it is a *Karla Marksa prospect 192* (purple point). So the flag is **KarlaMarksa192** and it's even 5 seconds to the end.
