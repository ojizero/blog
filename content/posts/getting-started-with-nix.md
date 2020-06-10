---
title: "Getting Started with Nix"
date: 2020-06-01T13:38:53+03:00
draft: true
tags: [100 Days, nixology]
---

> Nix, the purely functional package manager.

The Nix package manager has been, on and off, on my radar for a couple of years now, the premise of it, using "functional programing"-esque behaviours in order to create a reproducible package management, seemed very intriguing! I mean you know, _it has to be cool because it has **purely functional** in its tagline_! This post is a beginning of a group of posts as I experiment and play around with Nix, partially inspired (and tagged under the same name) by this ongoing [playlist from Shopify's Burke Libbey](https://www.youtube.com/playlist?list=PLRGI9KQ3_HP_OFRG6R-p4iFgMSK1t5BHs), who's videos reignited my interest in Nix, enough to starts attempting to move to solely using it!

## Introduction

Pretty much every system has it's own package management solution, be it a native solution or a community built one, from operating systems and the likes APT or Pacman (and much much more) in the Linux world, to HomeBrew and MacPorts (and probably more) in the world of macOS, even Windows with Chocolatey (and probably more)!

This isn't strictly limited to operating systems, userland systems also have their own package management, like Cargo, GoMods, NPM, RubyGems, and more. Pretty much every toolkit or development kit uses some form of package management somewhere.

### Dependency hell

Dependency hell is a term refering to a plethora of nasty problems and hardships in managng dependencies, package managers have the responsibility of finding solutions to this, exteremely hard, problem!

Those set of problems include, but not limited to, conflicting versions required for dependencies. Depending on dependencies required by another package, ther other package is gone, removed or whatever, the dependent is now broke cause the dependency is gone!

Most package managers have no way to work around this, I still vividly remember that one time ~5 years ago, when Pacman (the package manager for ArchLinux) broke and because of a nasty conflict in dependenies I faced when upgrading, the system ended up in an unupgradable (is that a word?) state.

In the end some packages ended up getting corrupted while I was trying to fix the issue myself, ended up costing me a whole weekend and a format, not fun!

### Idempotency

Idempotency is the property (mainly used in mathematics and computer science) that some operations produce the exact same behaviour and output, however many times you execute it with the same input.

A very widely used example, and where this concept is applied, is in REST APIs. Whenever you run some HTTP request, say a `GET` request, and get the exact same result returned, we call this API idempotent.

What the heck does that have to do with package management I hear you say? Will simply put, most package managers are not idempotent by default, this means that most operations are not idempotent, specifically installation operations. In Homebrew for example, running `brew install jq` multiple times could result on different version of JQ being installed depending on the time this command was run.

This ends us up in the unfortunate state were we can't really reproduce a certain installation of tools across multiple machines, or installations of OS, without some tedious manual labour, and even with that it may be to no avail!

An example were we do want multiple machines to have the exact same versions of tools installed is when multiple people develop some project, and each person has slightly different version of some tool, have fun managing all that when each dev environment is different and unique, and try to debug why it builds on your machine and not on your collegues!

## An initial, brief, primer in Nix

Alright, now that we've quickly given some small context around package management and (some of) the hurdles it tries to fix! Let's get started with a quick primer on Nix, and how it attempts to solve package management. We won't dive into details just yet, let's first just get to know a little about Nix and how it works.

Nix applies concepts taken from functional programming to try and solve the problem of package management, in essence it treats the whole state of the environment it manages as an immutable object, and only ever applies to it atomic operations to transform this state not mutate it.

This has the nice side effect that pretty much any operation done by Nix, can be rolledback at any given point in time. In essence this versions the whole environment (system) state, so you can perform (almost?) any operation safely, since at the worse case you can just run `nix-env --rollback` and be back to the previous state of the system.





<!-- In the next posts, we'll dive a bit deeper into Nix expressions and how they relate to Nix the package manager -->
