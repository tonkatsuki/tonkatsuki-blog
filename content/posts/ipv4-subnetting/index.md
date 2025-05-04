+++
title = 'IPv4 Subnetting: A quick(er) way'
date = 2025-05-03T00:00:10-05:00
tags = ["networking"]
featured_image = '/posts/ipv4-subnetting/feature.png'
draft = false
comments = true
+++

# Intro

This blog post is dedicated to elaborating, and a bit of re-explanation, of something I read many years ago on TechExams.net. Unfortunately the post is gone now, but there is a re-upload on infosecinstitute.com. Link is [here](https://community.infosecinstitute.com/discussion/38772/subnetting-made-easy)! You can also view it on [archive.org](https://web.archive.org/web/20250321210521/https://community.infosecinstitute.com/discussion/38772/subnetting-made-easy) if the original link isn't working.    
Traditional subnetting tutorials will explain network/host bits, etc. Although this is important to understand, it's not necessary to subnet quickly. This guide is solely to show you a way to subnet on the go, only needing some quick math.

The reason I made this was although the above link is quite good, it doesn't explain a couple other quick hacks for figuring out information on networks.

The main thing to take away is we're doing some simple subtraction, and then working in powers of two the whole time: **2<sup>2</sup> = 4, 2<sup>6</sup> = 64, and so forth.**

# Examples

## 192.168.4.10/29

Remember, the smallest IPv4 subnet we can have is a /32. That's one IP address. And the largest IPv4 subnet is technically a /0, which is all possible IPv4. That's probably not helpful just yet, but don't worry, you'll see why it's important soon.

First, let's separate our IPv4 into boundaries.

| Octet 1| Octet 2 | Octet 3 | Octet 4 |
| ----: | ----: | ----: | ----: |
| 192 | 168 | 4 | 10 |
| 8 | 16 | 24 | 32 |


Each octet corresponds to a number below, 8, 16, 24, and 32. It's no coincidence, if you've learned about subnetting you might've read about the binary behind it all. An IPv4 address is 32 bits. Remember a subnet is determined by how many **network** bits are set. 1 bit is 8 bytes, let's look at this below, so if you were to have your network bits set like this: 
>11111111 00000000 00000000 00000000

Would be a network that is x.x.x.x/8, or x.x.x.x 255.0.0.0

That means all the zeroes above, well, they are your host bits!
And inversely, if everything was set to 1's, you'd be left with a /32 or 255.255.255.255 as the mask. There's no host bits left to subnet with, so the network identifier and the single usable IP is the only IP possible, so a 192.168.4.3/32 is solely that, one IP.

One last thing you should remember before we dive in. Those boundaries above, 8, 16, 24, 32. They're convenient for another reason- if you see these, you don't need any special tricks to subnet. Let's say we have the IP **192.168.1.1**.

If this was a /8, it would mean the last 3 octets are occupied with host information. If it was a /16, the last 2, and so forth.

Now that we're through the explanation on these boundaries, let's do something useful:

***How many usable IP's can we get?***

>192.168.4.10/29

The subnet is a /29, we need to find the closest boundary to that. 29 cannot go into 8, 16, or 24. It has to be our 32 boundary.

>32-29 = 3

So wait, what is this 3 really? Well, it's the remaining host bits that are used for the network hosts itself. Remember those power of two's I mentioned? This is where it comes into play:

>2<sup>3</sup> = 8

Now let's stop for a second, we're working in the last boundary or octet of this, right? This **8** is important, because we know how many IP's (like a block!) are going to be in this network, but we need to figure out the boundaries between networks. Luckily, it's easy. Our /29 mask made us realize we're only ever subnetting in the last octet, and we have this 8, well, that's your boundary!

>192.168.4.0/29

>192.168.4.8/29 <-- You are here!

>192.168.4.16/29

192.168.4.10 is between 192.168.4.8/29 and 192.168.4.16/29, meaning we've found the **boundary** between our subnets. We now know that the only possible IP addresses in our subnet are 8, 9, 10, 11, 12, 13, 14, 15. But if you know a little IPv4, you know there's two things we need to remove... We want to find the usable IP's, but in IPv4, two addresses in a typical subnet shorter<sup>[a](#a)</sup> than a /31<sup>[b](#b)</sup> are special. One of them is our **network identifier** and one of them is our **broadcast** address. Our network identifier is always the **first** address and our broadcast is always our **last** address. Meaning for 192.168.4.10/29:

192.168.4.8 is our **network identifier**, aka the first address

192.168.4.15 is our **broadcast address**, aka the last address

Therefore, to answer our first question, what is the usable IP address in the subnet 192.168.4.10/29?

### 192.168.4.9 - 192.168.4.14

Now, if you read my link earlier, you probably found this very redundant. But what if I had some extra tips for you? They aren't explicitly explained in the original post, but some quick understanding of the math we're using will get you there.

## 10.34.99.4/22

Let's try a different subnet, with a different question.

For the above subnet, how many IP addresses are there?

Once again, consult the boundaries:

| Octet 1| Octet 2 | Octet 3 | Octet 4 |
| ----: | ----: | ----: | ----: |
| 10 | 34 | 99 | 4 |
| 8 | 16 | 24 | 32 |

22 is bigger than 8 and 16, but not 24. We know the boundary we're in, the third octet! Let's subtract 22 from 24.

>24-22 = 2

We now have that power of two we need:

>2<sup>2</sup> = 4

So wait, does that mean we only have 4 total IP's? Nope! Remember, we're working in the **third octet**. If we remember our binary, we have an entire extra 8 bits of hosts. Don't worry, we're getting there. Let's practice again, where's our boundary for each subnet?

>10.34.0.0/22

>10.34.4.0/22

>...

Wait. This might take a while, we have to reach 99 right? Counting 4 at a time seems a little silly. How can we find out the nearest when it's all the way up there, close that 100?

Well, we could do simple division: 99/4 = 24.75. Round up and you're at 25. Well, 25 isn't very useful, is it? We're working in powers of two, and there's no 2<sup>x</sup>  = 25, is there? However, we could quickly figure out that **100** is divisible by 4, meaning one of our subnets has to be 10.34.100.0/22. Minus 4 from 100, and you're at 96. We got our boundary!

>10.34.92.0/22

>10.34.96.0/22

>10.34.100.0/22

But before we move on, I want to teach you a hack about this. Know how we're always doing that thing where we work in powers of two? If you're not good at basic math or multiplication tables (I certainly stink at it!), you can hack your way through this. Just remember your **powers of two** and you will have to do barely any math at all. Our subnets always need to align with powers of two, right?

>2<sup>1</sup> = 2

>2<sup>2</sup> = 4

>2<sup>3</sup> = 8

>2<sup>4</sup> = 16

And so forth. So we've basically got 2, 4, 8, 16, 32, 64, 128, 256. If we can remember these, we can make life REALLY easy for us. Let's look at it again from a different lens.

99 is close to that 128 from our 2<sup>7</sup> operation. We could count down by 4's from 128, or we could minus an even smaller power of two to get us closer. How about 32? That puts us right on 10.34.96.0/22! By remembering the powers of two, we need to barely think about division, multiplication, or exponents. Although it's good to know the math, remember, we want to subnet *quick*!

Another great way, if you love tech like me, is also to just think about RAM sizes. RAM usually comes in 1GB, 2GB, 4GB, 8GB, 64GB, 128GB, and so forth. If you remember other ram sizes that aren't directly powers of two, like 24GB, 96GB, etc, you'll be very quick on your feet.

Since we got that out of the way, let's figure out one last thing on this tangent, what's our IP range? We're working in 10.34.96.0/22, and the next boundary is 10.34.100.0/22, so...

>10.34.96.0 - 10.34.99.255

Is our range, and we know the network identifier is the first IP, and the broadcast is the last IP. Let's answer the actual question:

How many IP addresses are there total? We have the range, but without some math, we don't know the actual number. How can we get it quickly? Back to the mask, that was  a /22, right? Let's think about something:

A /32 is our smallest IPv4 subnet, a /0 is our biggest, that encompasses all the IPv4 possible. Here's a thought, if you took the entire /0, aka all of IPv4, and made them all /32, that'd mean you'd have only 1 IP per subnet, right? Remember how we're using those power of two's? What if we plugged in 2<sup>32</sup>?

That would get us: 4,294,967,296. That's all of the possible IPv4 in the world, showing our constriction of 32 bit addresses (IPv6 fixed this for us by going all the way up to 128). So if your subnet was /0, it gets you the number of total IPv4, what if instead of a /0 we used our mask, a /22? 32-22=10. 2<sup>10</sup> = 1,024.

That means our total IP addresses for 10.34.96.0/22 is 1,024!

So why would you need this? Imagine you were tasked with building a subnet that can handle up to 20 hosts, what mask would you use? Understanding that 32 minus whatever mask, would tell you how many hosts you can have. Just remember to account for your network identifier and broadcast!


## How many /28 subnets can you fit inside a /19?

We figured out 32-mask=total hosts. What if you want to figure out how many subnets can fit inside another subnet? Pretty easy!

>28-19 = 9

>2<sup>9</sup> = 512

We can fit 512 /28's into a /19. Pretty convenient, right?


## 256!

One last thing I haven't covered yet. You may have gotten a subnet mask in this notation: 255.255.255.0 (That's a /24 by the way!). A network address can only ever go up to 255, though it's technically 256, remember we start at 0. Two quick things with that:

If you are given a network in this form: 172.17.4.4 255.255.255.192, it's even easier to do the above tricks. We know we're working in the fourth octet because that's the one with the host bits available- it is less than 255. Now how do we figure out the usable addresses? Simple subtraction! 256-192 = 64.

So what's that 64? That's 2<sup>6</sup>. Get it? The full mask is actually easier to work with! We don't need to subtract from the boundary, then do a power of two. We've found the number of addresses in each subnet. We can work further backwards, 32-6 = 26, our CIDR notation.

So for 172.17.4.4 255.255.255.192, that's 172.17.4.0/26, and our block size is 64, so the next network after ours is:

>172.17.4.0/26

>172.17.4.64/26

And finally, let's round it all off!

### What's the total number of IP's?

32-26 = 6

2<sup>6</sup> = 64

Total number of IP's: 64

### Total usable?

Take out the first and last address:

172.17.4.0 = network identifier

172.17.63.255 = broadcast

172.17.4.1 - 172.17.63.254 = usable range

### How many /30's fit in a /26?

30-26 = 4

2<sup>4</sup> = 16

16 /30's fit in a /26

# IPv6

Notice how we were only working with IPv4. All of this holds true with IPv6, just remember, an IPv6 address is hexadecimal and not solely numerically defined. Additionally a /128 address in v6 is a single host IP, much like a /32. The smallest subnet you can make is a /64<sup>[c](#c)</sup>. All of the operations are pretty much the same, calculating total number of hosts, usable IP range, etc.

# Final thoughts

I hope you stuck with me on this to learn IP subnetting. Feel free to provide comments below via GitHub if you think this could be fixed up a little, or just anything you want to add!

## Notes

### a

You may see people refer to prefixes (read: subnets or networks) as being long or short. A long prefix is one that is more defined, meaning more network bits are set. For instance, a /31 network would be a LONG prefix, and a /8 might be considered a short prefix. This is important because in routing, the longest (aka more specific) prefix is matched before a shorter (aka less specific) prefix.

### b
A /31 network has only two addresses. [RFC3021](https://datatracker.ietf.org/doc/html/rfc3021) provided a path for developers to use this mask, even though in typical IPv4 networking, the first address in a subnet is the identifier and not to be used, and the last is a broadcast, a /31 provides a way to provide a 2 IP address subnet, typically only used for point to point links between devices, meaning you can ignore that whole thing about NIDs and Broadcast addresses for a /31, they don't apply! You can still assign those IP's to interfaces, but remember, you only have two total. The benefit of this is a /30 would consume two additional IP's, but they would be unusable. Think of this as a way to conserve IP space with point to point links. Be wary though, some manufacturers still do not support this, so check your vendor documentation before using!

### c

There are smaller subnets in IPv6 than a /64, but it doesn't conform to IPv6 standards. Typically vendor gear will not support this operation, and [RFC5375 Section 3.1](https://datatracker.ietf.org/doc/html/rfc5375#section-3.1) specifically recommends against it
