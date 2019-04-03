---
published: true
image: /img/angr.png
date: '2019-04-03'
title: Introduction to angr Part 2.1
subtitle: 'Same shit, other day?'
---
I told you in the [last post](https://blog.notso.pro/2019-03-26-angr-introduction-part2/) we would have tested angr on a reverse engineering challenge different from the ones we've seen so far in angr_ctf. Since I'm a ~~lamer~~ lazy person I did not want to completely reanalyze a new binary so I went for the one we saw in [this](https://blog.notso.pro/2019-03-13-Enigma2017-Crackme0-writeup/) post. If you haven't read it, please do it now as I won't go over it thoroughly, but rather I will show a different approach to solving it using angr. But first, let's do a quick recap of what we need to know in order to solve this challenge.

![ssod0]({{site.baseurl}}/img/ssod0.png)

As you can see I highlighted in red the code path we are not interested in (the ones leading to `wrong()`), in green the one we are interested in (the one leading to "That is correct!") and in blue the instruction from which angr will start the analysis. Let's have a look at `fromhex()` and see if we can rule out any uninteresting paths.

![ssod1]({{site.baseurl}}/img/ssod1.png)

Mmmmh, as we've seen previously, `fromhex()` will return different values based on the input it gets, but we know from this code in `main()` we are only interested in the state that leads to 0 being returned through `EAX`:

![ssod4]({{site.baseurl}}/img/ssod4.png)

Basically a `JE` instruction is the same as a `JZ` (a.k.a. Jump if Zero) instruction. The `TEST EAX, EAX` instruction right before it sets the zero flag in the `EFLAGS` register if `EAX` is zero. The `JE` and the `JZ` instructions jump to the address specified if the zero flag is set, hence we are interested only in the code path that leads to 0 being stored in `EAX`. Knowing this we can go back to `fromhex()` and take note of all the code path leading to anything else than 0 being returned.

![ssod2]({{site.baseurl}}/img/ssod2.png)

![ssod3]({{site.baseurl}}/img/ssod3.png)

Ok, now that we have all the code paths we deem interesting and the ones we want to avoid let's see where we need to put our symbolic buffer.

![ssod5]({{site.baseurl}}/img/ssod5.png)

From this screenshot we can see that the pointer to our input string is pushed on the stack right before calling `fromhex()` and this means we can basically store our input string wherever we want, then put the address we chose (which can be any address in the stack) inside `EAX` and the program will take care of the rest. Don't worry, we will see how to do it in a moment.

Let's see what we have got so far:
1. the address we will start from is `0x8048692`, which is the one of `PUSH EAX` right before the call to `fromhex()`
2. the address we want to reach is `0x80486d3`, which is the starting address of the code block that leads to printing "That is correct!"
3. a list of addresses leading to uninteresting code paths `[0x8048541, 0x8048624, 0x8048599, 0x8048585]`
4. we know the pointer to our string is stored in `EAX`