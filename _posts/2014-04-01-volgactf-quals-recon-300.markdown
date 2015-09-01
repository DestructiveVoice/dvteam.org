---
layout: post
title:  "VolgaCTF Quals 2014 Recon 300"
date:   2014-04-01
categories: writeups volgactf quals 2014 recon 300
author: phil9l
comments: true
---

We have [a facebook page](https://www.facebook.com/yan.vetrov.96). Firstly, let's check the information. We can find his [Instagram](https://instagram.com/fin_163/), [vk page](https://vk.com/vetroyan) and [a site](http://www.colorflip.com/).

![]({{ site.baseurl }}/assets/2014-04-01-volgactf-quals-recon-300/screen1.png){: .center-image }

To finish with facebook page I decided to read his post. In [one message](https://www.facebook.com/yan.vetrov.96/posts/1390152937928284?stream_ref=10) he writes that he want to download free music and about VK music.
<!-- more -->

I wished to start with the [site](http://www.colorflip.com/) where we can just flip the pages. I read the site's code fast and found nothing interesting there but the comments are nice. May be I missed something? Ok, we always can return to this site. Let's now go to the vk page.

![]({{ site.baseurl }}/assets/2014-04-01-volgactf-quals-recon-300/screen2.png){: .center-image }

Ok, we still remember about VK music, so let's visit his [vk page](https://vk.com/vetroyan). Firstly, we can see [the post](http://vk.com/wall43944111_5) where he asks about opening password-protected zip. Bruting password works fine and we can get it easy: *228322* (funny one). There is only a *O_o.txt* file with text "*o_O what are u doing here?*". Just a joke I think. Ok, no more interesting posts but we remember about music, don't we?

![]({{ site.baseurl }}/assets/2014-04-01-volgactf-quals-recon-300/screen3.png){: .center-image }

One song is really suspicious: *Easy flag &mdash; here*. Listening it seems useless. But we can download it, maybe we would find something interesting there? Of course, it is here! On the cover. So the flag is **s0_much_music**.

![]({{ site.baseurl }}/assets/2014-04-01-volgactf-quals-recon-300/screen4.png){: .center-image }
