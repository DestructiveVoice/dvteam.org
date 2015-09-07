---
layout: post
title:  "MMA CTF 1st 2015: pieceofeight reloaded (Misc 200)"
date:   2015-09-07
categories: writeups mmactf 2015 misc 200
author: artli
comments: true
---

![]({{ site.baseurl }}/assets/2015-09-07-mmactf-2015-pieceofeight_reloaded/0-description.png){: .center-image }

We are given an address, an image of our goal (which looks like a 3x3 version of the [15 puzzle](https://en.wikipedia.org/wiki/15_puzzle)) and a strange hint.

Let's connect to the given address:
<!-- more -->

![]({{ site.baseurl }}/assets/2015-09-07-mmactf-2015-pieceofeight_reloaded/1-connect.png){: .center-image }

Hmm... If it really is a smaller version of the 15 puzzle, we should be able to move the numbers around the board. Let's type "**d**" and send it:

![]({{ site.baseurl }}/assets/2015-09-07-mmactf-2015-pieceofeight_reloaded/2-move.png){: .center-image }

Indeed, the tile below the gap moved up switching places with the blank tile. Seems like we can move the numbers by sending "**u**", "**d**", "**l**" and "**r**".

Is that enough to solve the puzzle? Let's try doing it manually (if you're unfamiliar with solving this kind of puzzles, [this](http://www.instructables.com/id/How-To-Solve-The-15-Puzzle/?ALLSTEPS) might help):

<center><i>(After some moves)</i></center>

![]({{ site.baseurl }}/assets/2015-09-07-mmactf-2015-pieceofeight_reloaded/3-solved_fail.png){: .center-image }

...and we fail! This state is almost identical to the goal state, but two tiles (*5* and *6*) are switched. Unfortunately, this almost-solved position is known to be unsolvable, so we can't get the flag by simply moving the pieces around.

But there was something else! The continuation keys are mentioned in the hint and appear below each game state. They surely mean something, but what?

If each game state is assigned its own unique continuation key, then maybe entering a key will let us continue the game from the corresponding state. Let's try to do it.

We connect and copy the key:

![]({{ site.baseurl }}/assets/2015-09-07-mmactf-2015-pieceofeight_reloaded/4-connect_key.png){: .center-image }

Then we make a move and send "**c &lt;continuation key&gt;**" as suggested by the hint:

![]({{ site.baseurl }}/assets/2015-09-07-mmactf-2015-pieceofeight_reloaded/5-restore_key.png){: .center-image }

As expected, this restores the previous game state. No big deal, let's keep going. Sending "**l**":

![]({{ site.baseurl }}/assets/2015-09-07-mmactf-2015-pieceofeight_reloaded/6-weird_shit.png){: .center-image }

Wait, *WHAT?!* Did tile *2* just magically switch places with tile *5*? I think we're onto something.

It seems that after a game state is restored you lose the ability to move tiles normally, but instead the tile you moved the last (in our case it's tile *2*) can pass through any other tile switching places with it. This bug is very convenient for solving our unsolvable position.

So now we have a plan:
<li>Get to the unsolvable <i>1234<b>65</b>78</i> position and copy its continuation key.</li>
<li>Move tile <i>5</i> down.</li>
<li>Enter the continuation key to restore the previous position.</li>
<li>Move tile <i>5</i> to the left to switch it with tile <i>6</i>.</li>

Let's do it! This is what we get after the last step:

![]({{ site.baseurl }}/assets/2015-09-07-mmactf-2015-pieceofeight_reloaded/7-pwned.png){: .center-image }

*Pwned!* 9 more puzzles to go.

There is a slight twist though:

![]({{ site.baseurl }}/assets/2015-09-07-mmactf-2015-pieceofeight_reloaded/8-too_slow.png){: .center-image }

This message awaits those who unwisely decide to solve all 10 puzzles manually. Unfortunately, you can't get the flag without writing a script. You can write your own 8 puzzle solver (for example, using a simple BFS-based algorithm) or google one. The rest is simple: follow our pwning plan 10 times and get the flag.

The only important piece of information to reveal is that you can send multiple commands in one line: just write them all consecutively. Without using this option it's practically impossible to solve the task.

After solving all 10 puzzles we finally get the flag:

![]({{ site.baseurl }}/assets/2015-09-07-mmactf-2015-pieceofeight_reloaded/9-flag.png){: .center-image }