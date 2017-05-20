# Introduction

What you are looking at is an *unofficial* specification of the *MAGES. visual novel engine* and guide for working with it. The MAGES. visual novel engine (named - by us - after [its creator](https://ja.wikipedia.org/wiki/MAGES.) and source file names found in various builds) is mostly used in many titles in the [Science Adventure](https://en.wikipedia.org/wiki/Science_Adventure) series. A descendant of the engine developed by [KID](https://en.wikipedia.org/wiki/KID), which many 5pb./MAGES. developers worked for prior to their closure, first sighted (by us) in 2007's [Umisho](https://vndb.org/v6207), it has been used on platforms from PS2 all the way to iOS.

## Disclaimer

All the information in this document has been gained by reverse-engineering publically released video games. We are a fan group and not associated with 5pb./MAGES. in any way. We've only just begun to write this guide and **it is currently highly incomplete.**

At first, we will *only* describe the specific version used in *Steins;Gate 0's* PC release. We are aware of several differences between various versions, but we cannot cover them so early on.

## Who this guide is for

Our primary end goal is to make Science Adventure visual novels fan translatable without requiring significant technical skill. To accomplish this, we are working on a toolset for extracting, decoding and modifying game assets. But we're not quite there yet. We intend to add a User's Guide / manual for this when we're closer.

For now, we are focusing on specifying the scripting virtual machine and how it fits in with the rest of the engine, mostly to serve as a reference for our own projects. For this section, the reader is assumed to be familiar with the basic concepts of designing and implementing embedded programming languages. It is intended to be helpful to people creating tools for working with *SC3* scripts or reimplementing the runtime. Note that this documentation alone may not be sufficient information to accomplish those tasks, and you may need to do some reverse-engineering yourself.

Later on, we may also document implementation detail of at least Windows builds for those who are curious.

## About the authors

We are the [Committee of Zero](http://sonome.dareno.me/), a visual novel hacking/fan translation group that mostly works on the *Science Adventure* series. So far, we have released an [improvement patch](http://sonome.dareno.me/projects/sghd.html) for the *Steam* release of *Steins;Gate* and a [translation port](http://sonome.dareno.me/projects/sg0.html) that inserts the official English localisation from the *Vita* version of *Steins;Gate 0* into the Japanese PC version, along with various improvements. Throughout this work we have *probably* gained more knowledge about the inner workings of these games than anyone else without source code access, and we'd like to pass it on to others who don't have the skill or patience or time to waste to repeat the process.

This guide is currently being written by *DrDaxxy*, and is largely based on reverse-engineering done by *SomeAnon*. Both our RE work and this guide *heavily* draw from a previous SC3 spec (written in Russian) contributed by a third party. In particular, the [expression operator table](/scripting/expressions.md) is almost a straight translation. We have unfortunately not been able to contact the author so far and are thus unable to give credit for or republish this work.