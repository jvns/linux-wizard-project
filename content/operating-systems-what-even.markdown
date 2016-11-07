---
title: "What even is an operating system?"
date: 2016-11-06T09:34:26Z
weight: 101
url: /operating-systems-what-even/
---

# Operating systems

Hello! Our plan in this project is to become GREAT FRIENDS WITH YOUR
OPERATING SYSTEM. When you understand your operating system, it can
really help you out with debugging and understanding what your programs
are doing!

But first, we have to understand what an operating system even is. We're
going to be talking about Linux, but all of this actually applies to ANY
operating system.

### What does your operating system even do?

![caption](/secret-project/images/os-responsibilities.svg)

Your computer needs to do a ton of stuff like

* read files
* display stuff to your laptop screen
* do networking so you can watch YouTube videos

The programs on your have computer have **no idea** how to do any of
this stuff. They don't know how computer networks work! If they had to
communicate with a hard drive by themselves they'd have a heart attack.

### the "kernel": it's a really big program

So we said that programs have no idea how to read files! But **someone**
needs to know about it. The program that's responsible for everything is
called the "kernel".

The Linux kernel is a (really big!) program responsible for all the
tasks we talked about in the diagram above.

It's written in C and it's several million lines of code. We're not going to spend too
much time reading kernel code, because how to do that is a whole book by
itself. But just to show you that the kernel is a real program that you can look at -- check out this code that
[helps makes the keyboard on the 2015 Macbook Pro work](https://github.com/torvalds/linux/blob/v4.5/drivers/hid/hid-apple.c).

One of the lines in that program is `if (asc->quirks &
APPLE_MIGHTYMOUSE)`! Which makes sense, the "mighty mouse" is a kind of
mouse you can buy for the Macbook!

The Linux kernel is 4 million lines of code partly because it has to
understand **all kinds of weird things**. That file was just for Apple
keyboards! There are much more complicated devices than keyboards (hard
drives and filesystems are a lot more work to understand).

### Exercise: search the Linux kernel code

We've arrived at our first exercise! Your mission: search the Linux
kernel and find some words that almost make sense. I don't expect you to
understand what's going on, necessarily. It's okay if you don't know C!
The goal is to

1. Understand that the Linux kernel is a (really big) program
1. Look at some of the code and realize that, whether or not you
   understand what it's doing, it's not magic. It's code!

You can search the kernel's code by going to
[https://livegrep.com/search/linux](https://livegrep.com/search/linux)
and typing in some words. Try searching for "fancy" or "tree walk" or
"memory"!

In general we're going to be using more friendly ways to interact with
the kernel than reading its code, don't worry :)
