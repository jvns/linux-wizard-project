---
title: "Talking to your operating system (system calls!)"
date: 2016-11-06T12:35:05Z
weight: 102
url: /talking-to-your-os/
---

## Talking to your operating system (system calls!!)

We just learned that our operating system knows how to do ALL KINDS OF
AMAZING THINGS. 

This is amazing because when I write programs, my program doesn't need
to understand how a hard drive works to read a file. But how do I ask
the operating system to do work for me? The answer: ★ SYSTEM CALLS ★!

System calls are basically your operating system's interface. Want to
open a file? Use the `open` system call! Want to read from the file you
just opened? It's just a `read` away!

All Unix operating systems (macOS, Linux, BSD) have basically the same
system calls.

### Every programming language uses system calls!

Here are code snippets in 4 different programming languages that all
use the `open` system call under the hood:

**Python** & **Ruby**:

```
open("./awesome.txt")
```

**Java**

```
Files.readAllBytes(Paths.get("file.txt"));
```

**C**:

```
fopen ("myfile.txt","w");
```

These programs do slightly different things (the Java program reads the
whole file, while the Python program just opens it). But they all use
exactly the same system call to open a file! From the operating system's
perspective, they're all super similar.
