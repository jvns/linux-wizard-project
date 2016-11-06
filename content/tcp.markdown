---
title: "tcp"
date: 2016-11-05T22:56:10Z
url: /tcp/
weight: 500
---


This isn't about understanding *everything* about TCP or reading through [TCP/IP Illustrated](http://www.amazon.com/TCP-Illustrated-Vol-Addison-Wesley-Professional/dp/0201633469). It's about how a little bit of TCP knowledge is essential. Here's why.

When I was at the [Recurse Center](http://recurse.com), I wrote a TCP stack in Python ([and wrote about what happens if you write a TCP stack in Python](http://jvns.ca/blog/2014/08/12/what-happens-if-you-write-a-tcp-stack-in-python/)). This was a fun learning experience, and I thought that was all.

A year later, at work, someone mentioned on Slack "hey I'm publishing messages
to NSQ and it's taking 40ms each time". I'd already been thinking about this
problem on and off for a week, and hadn't gotten anywhere.

A little background: NSQ is a queue that you send to messages to. The way you
publish a message is to make an HTTP request on localhost. It really should not
take **40 milliseconds** to send a HTTP request to localhost. Something was
terribly wrong. The NSQ daemon wasn't under high CPU load, it wasn't using a
lot of memory, it didn't seem to be a garbage collection pause. Help.

Then I remembered an article I'd read a week before called [In search of performance - how we shaved 200ms off every POST request](https://gocardless.com/blog/in-search-of-performance-how-we-shaved-200ms-off-every-post-request/). In that article, they talk about why every one of their POST requests were taking 200 extra milliseconds. That's.. weird. Here's the key paragraph from the post

### Delayed ACKs & TCP_NODELAY

> Ruby's Net::HTTP splits POST requests across two TCP packets - one for the
> headers, and another for the body. curl, by contrast, combines the two if
> they'll fit in a single packet. To make things worse, Net::HTTP doesn't set
> TCP_NODELAY on the TCP socket it opens, so it waits for acknowledgement of the
> first packet before sending the second. This behaviour is a consequence of
> Nagle's algorithm.

> Moving to the other end of the connection, HAProxy has to choose how to
> acknowledge those two packets. In version 1.4.18 (the one we were using), it
> opted to use TCP delayed acknowledgement. Delayed acknowledgement interacts
> badly with Nagle's algorithm, and causes the request to pause until the server
> reaches its delayed acknowledgement timeout..


Let's unpack what this paragraph is saying.

* TCP is an algorithm where you send data in **packets**
* Their HTTP library was sending POST requests in 2 small packets

Here's what the rest of the TCP exchange looked like after that:

> application: hi! Here's packet 1. <br>
> HAProxy: &lt;silence, waiting for the second packet&gt;<br>
> HAProxy: &lt;well I'll ack eventually but nbd&gt;<br>
> application: &lt;silence&gt;<br>
> application: &lt;well I'm waiting for an ACK maybe there's network congestion&gt;<br>
> HAProxy: ok i'm bored. here's an ack<br>
> application: great here's the second packet!!!!<br>
> HAProxy: sweet. we're done here<br>

That period where the application and HAProxy are both passive-aggressively
waiting for the other to send information? That's the extra 200ms. The application is doing it because of Nagle's algorithm, and HAProxy because of delayed ACKs.

Delayed ACKs happen, as far as I understand, by default on *every* Linux system.
So this isn't an edge case or an anomaly -- if you send your data in more than 1
TCP packet, it can happen to you.

### in which we become wizards

So I read this article, and forgot about it. But I was stewing about my extra 40ms, and then I remembered.

And I thought -- that can't be my problem, can it? can it??? And I sent an email to my team saying "I think I might be crazy but this might be a TCP problem".

So I committed a change turning on `TCP_NODELAY` for our application, and BOOM.

All of the 40ms delays **instantly disappeared**. Everything was fixed. I was a wizard.


### should we stop using delayed ACKs entirely

A quick sidebar -- I just read [this comment on Hacker News](https://news.ycombinator.com/item?id=9048947) from John Nagle (of Nagle's algorithm) via [this awesome tweet](https://twitter.com/alicemazzy/status/667799010317574145) by @alicemazzy.

> The real problem is ACK delays. The 200ms "ACK delay" timer is a bad idea that
> someone at Berkeley stuck into BSD around 1985 because they didn't really
> understand the problem. A delayed ACK is a bet that there will be a reply from
> the application level within 200ms. TCP continues to use delayed ACKs even if
> it's losing that bet every time.

He goes on to comment that ACKs are small and inexpensive, and that the problems
caused in practice by delayed ACKs are probably much worse than the problems
they solve.

### you can't fix TCP problems without understanding TCP

I used to think that TCP was really low-level and that I did not need to understand it. Which is mostly true! But sometimes in real life you have a bug and that bug is because of something in the TCP algorithm. So it turns out that understanding TCP is important. (as we frequently discuss on this blog, this turns out to be true for a lot of things, like, system calls & operating systems :) :))

This delayed ACKs / TCP_NODELAY interaction is particularly bad -- it could affect anyone writing code that makes HTTP requests, in any programming language. You don't have to be a systems programming wizard to run into this. Understanding a tiny bit about how TCP worked really helped me work through this and recognize that that thing the blog post was describing also might be my problem. I also used strace, though. strace forever.

