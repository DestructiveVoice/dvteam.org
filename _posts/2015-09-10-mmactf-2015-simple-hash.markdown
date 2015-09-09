---
layout: post
title:  "MMA CTF 1st 2015: Simple Hash (Reverse & PPC 200)"
date:   2015-09-10
categories: writeups mmactf 2015 simple-hash
author: hx
comments: true
---

<div style="border: 1px solid #333; padding: 0 20px;" markdown="block">
**Task:**

Get the flag!

    nc milkyway.chal.mmactf.link 6669

[simple_hash.7z]({{ site.baseurl }}/assets/2015-09-10-mmactf-2015-simple-hash/simple_hash.7z-8debe103674b214f4edd6f9a1b2d56dcff9ab45169770b1dd2e06984da363c74)
</div>

We are given an archive that contains a 32-bit Linux executable. Let's analyze it in IDA Pro, starting from `main` function.
<!-- more -->

![]({{ site.baseurl }}/assets/2015-09-10-mmactf-2015-simple-hash/ida1.png){: .center-image }

The first thing we can note is that there's a condition in the end of the `main`: if the input string satisfies some requirements, the program will print message "Correct!" and the flag read from a file. Otherwise, the program will print "Wrong". So, the task is to form an appropriate string and pass it to the server to get the contents of `./flag.txt`.

Let's examine what the `main` does.

### Checking the input

![]({{ site.baseurl }}/assets/2015-09-10-mmactf-2015-simple-hash/ida2.png){: .center-image }

After reading the input string, the program calculates its length and, if the length isn't zero, reduces the string by assigning the last character to NULL. This removes a newline character appended by the [fgets](http://www.cplusplus.com/reference/cstdio/fgets/).

![]({{ site.baseurl }}/assets/2015-09-10-mmactf-2015-simple-hash/ida3.png){: .center-image }

Then the program calls another function with a loop inside. The loop iterates over characters of the input string and checks whether all of them are alphanumeric using [isalnum](http://www.cplusplus.com/reference/cctype/isalnum/) function. I've renamed this function to `are_alnum_only` (it's very convenient IDA feature to rename variables and functions if you've understood their purpose). The function returns 0 if some character doesn't satisfy the requirement (then `main` jumps to code that prints "Wrong") and 1 otherwise.

### Calculating a hash

Further, after the input check, we can see an interesting call of `calc_hash` function. Actually, IDA shows it as `_Z9calc_hashv`; these complex names contain information about types and are derived by C++ compiler to implement function overloading. Let's open this function (click on the function name and press `Enter`). I've already renamed some variables here:

![]({{ site.baseurl }}/assets/2015-09-10-mmactf-2015-simple-hash/ida4.png){: .center-image }

#### 64-bit arithmetics

We can note here that a lot of operations are executed on pairs of variables (I've named these pairs as `var_name1` and `var_name2`). The important thing to know is how 64-bit data types  (e.g. `long long`) are implemented in 32-bit programs. It can be a good question - how the program can operate with 64-bit numbers if it's available only CPU instructions for 32-bit arithmetics?

The answer is that the program uses software simulation of 64-bit types. A `long long` is represented by two 32-bit `int`s. So, for example, during addition the program firstly adds "lower" `int`s (representing lower bits), then adds "upper" `int`s considering a possible carry from the previous operation.

Usually, a return value of a C/C++ function is placed to 32-bit `eax` register, but if the function returns a `long long`, the result is placed to a pair of registers `eax:edx` (lower bits in `eax`). Similarly, a 64-bit function argument on the stack is represented like two 32-bit arguments.

By the way, some 64-bit compilers provide an analogous type `__int128`.

Now we can realize that the `calc_hash` operates with `long long`s and returns a `long long` too (it really writes to the `eax:edx` pair). In fact, it's a reason why code of these function so big.

#### Functions `_Z2mmx` and `___moddi3`

Another question is what functions `_Z2mmxx` and `___moddi3` (note that they get and return `long long`s) do. The purpose of the last function (it's built-in) can be understood from its name. The purpose of the `_Z2mmxx` is less clear. Its code is placed in the executable; we can open it, but it's too big to analyze. Maybe this function implements some simple arithmetic with `long long`s too?

We can check our assumptions by an experiment. Let's open gdb, load the executable and run these functions. It's important to specify arguments types (initially gdb doesn't know that it's needed to pass to the function four 32-bit values, not two ones). Of course, we also have a disparity with the return type, but it's enough to get only lower 32 bits if we're sure that the number isn't big and upper bits are zeros:

    $ gdb -q simple_hash
    Reading symbols from simple_hash...(no debugging symbols found)...done.
    (gdb) break main
    Breakpoint 1 at 0x8048a1e
    (gdb) run
    Starting program: /tmp/simple_hash

    Breakpoint 1, 0x08048a1e in main ()
    (gdb) print __moddi3(17LL, 3LL)
    $1 = 2
    (gdb) print __moddi3(66992LL, 67LL)
    $2 = 59
    (gdb) print _Z2mmxx(3LL, 2LL)
    $3 = 6
    (gdb) print _Z2mmxx(6000LL, 43LL)
    $4 = 258000

Yeah! The assumptions were right, `__moddi3` returns the remainder, `_Z2mmxx` implements multiplication.

#### Polynomial hashing

Now we can examine an algorithm of the `calc_hash`. It isn't complex and implements [Rabin-Karp rolling hashing](https://en.wikipedia.org/wiki/Rolling_hash#Rabin-Karp_rolling_hash) (also well-known as "polynomial hashing"). The function code is equal to the following:

{% highlight python linenos=table %}
long long calc_hash() {
    long long result = 0;
    int ch_index;
    for (ch_index = 0; input[ch_index]; ch_index++) {
        long long tmp = result * 0x241LL + input[ch_index];
        result = tmp % 0x38D7EA4C68025LL;
    }
    return result;
}
{% endhighlight %}

#### Comparing the result

Now let's see how the hash value is checked in the program. We can return to the `main` function (press `Esc`):

![]({{ site.baseurl }}/assets/2015-09-10-mmactf-2015-simple-hash/ida5.png){: .center-image }

The program checks the returned `edx:eax` pair by comparing `eax` with `0xB437EEB0` and `edx` with `0x1E1EA` using XOR. If any of these XORs gives a non-zero value (`(eax ^ 0xB437EEB0) | (edx ^ 0x1E1EA) != 0`), the program sets `eax` to 0 and further jumps to printing "Wrong" message, else the program sets `eax` to 1, and we'll get the flag.

We can conclude that the required hash value is `0x1E1EAB437EEB0`. And now we move to a PPC part of the task - how to get a string with this hash value that contains only alphanumeric characters?

### Cracking the hash

Firstly, if the hash is calculated without taking the modulo and without precision restrictions, we can restore the whole string value. Actually, in the last loop iteration in the `calc_hash`, variable `result` without a code of the last character is multiplied by `0x241`, so if we'll divide the hash by `0x241`, the remainder, that lies in range `0 <= r < 0x241 = 577`, will contain a code of the last character (all other summands of the polynomial are now divisible by `0x241`).

Further, we can evenly divide the hash by `0x241` to get a value of `result` variable from the previous iteration of the loop, and then extract the penultimate character, and so on. We can implement this in Python, which has built-in arbitrary-precision arithmetics:

{% highlight python linenos=table %}
COEFF = 0x241


def decode_full_hash_value(value):
    res = ''
    while value:
        code = value % COEFF
        res += chr(code)
        value /= COEFF
    return ''.join(reversed(res))
{% endhighlight %}

Secondly, from mathematics we know that if we take the modulo in every iteration of the loop, the result will be equal to the case if we use the arbitrary-precision arithmetics and get the modulo only after calculation of the whole polynomial.

So, we know a remainder of division of a full hash value of the desired string by modulo `0x38D7EA4C68025`. Okay, let's bruteforce full hash values with this modulo (starting from small values corresponding to small strings), for each of them restore the corresponding string and check whether it contains only alphanumeric characters.

You can download the script on Python 2 from [pastebin](http://pastebin.com/Ku0HZFDL).

It's convenient to run it with [PyPy](http://pypy.org/) to speed up the bruteforce. After several minutes of execution we can find a result among debug output:

    9 / 9 \x33\x64\x43\x67\x79\x41\x64\x42\x54
    Found: 3dCgyAdBT

Perfect! Let's pass the string to the server:

    $ nc milkyway.chal.mmactf.link 6669
    3dCgyAdBT
    Correct!

    MMA{mocho is cute}

As a result, the task isn't very hard, but requires some experience in reverse engineering and algorithms.
