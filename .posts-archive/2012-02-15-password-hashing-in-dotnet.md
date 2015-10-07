---
layout: post
title: "Password Hashing in .NET"
tags: [security, dotnet, csharp]
---

I'm currently in the process of extracting a lot of the generic, boilerplate code that I use regularly and building a series of re-usable libraries. One of the first pieces that I wanted to tackle was to build a reliable password hashing library that followed current best practices for how to securely hash user passwords.

The vast majority of projects that I've seen have used either MD5 or SHA for hashing their passwords. Both of those hashing algorithms have valid use cases, but they are both far too fast to be used for hashing password data. [Coda Hale](http://codahale.com)'s article on [How To Safely Store A Password](http://codahale.com/how-to-safely-store-a-password/) goes in depth into the problems caused by using a general purpose hashing algorithm for passwords. Unfortunately there doesn't seem to be a verified implementation of bcrypt for .NET, but there is a built-in implementation of [PBKDF2](http://en.wikipedia.org/wiki/PBKDF2) in the .NET Framework.

The hashing functionality was extracted from the [Stack Exchange OpenID Project](http://code.google.com/p/stackid/). You can find the source code for the library [on github](https://github.com/joeyb/JoeyB.Security).