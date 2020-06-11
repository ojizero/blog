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

Nix applies concepts taken from functional programming to try and solve the problem of package management, in essence it treats the whole state of the environment it manages as an immutable object, and only ever applies to it atomic operations to transform this state, never mutating it.

This has the nice side effect that pretty much any operation done by Nix, can be rolledback at any given point in time. In essence this versions the whole environment (system) state, so you can perform (almost?) any operation safely, since at the worse case you can just run `nix-env --rollback` and be back to the previous state of the system.

Whenever you install anything, Nix would install it in a folder called the Nix store, typically in the path `/nix/store`. Each installed package is a self contained unit with all of its dependencies referenced, and is cryptographically hashed to ensure the content and versions of the package and its dependencies.

When you want to use the installed package, what you're really using is a symlink found in your Nix profile that points to the actual package. The Nix profile acts like a dictionary mapping the packages you installed to their appropriate versions and paths.

In practice this is extended to include even the configurations of the installed package, so you can share these configs by simply reusing the installed path for them, or setting any new profile to use them.

An example for user environments, taken from the [Nix manual on user profiles](https://nixos.org/nix/manual/#sec-profiles) is shown below.

![User environments diagram](https://nixos.org/nix/manual/figures/user-environments.png)

This effectively allows even multiple users, to install multiple versions of packages, even if they have conflicting versions or have conflicting dependencies. Since each package refers to symlinks to the proper versions of their dependencies and each package/dependency is identified by it's hash instead of name (at the lowest level).

It also allows for easy rollbacks since you would just revert to using an older profile version, that refers to the older symlinks. In a sense, these profiles represent the state of the system at some point in time, and they are versioned so you can switch between them easily.

Another neat side effect from this, is that anything that is installed as a dependency is only ever used but what depends on it, you can't really see or use it yourself. For example if you install something that depends on some library, you can't easily access this library from any global point, unless you install it explicitly. This immediately fixes the problem were you could depend on some globally installed dependency, that "disappears" when you remove whatever dependended on it.

### Installation

Installing Nix is quite straightforward, you can follow along the [Nix manual section on installation](https://nixos.org/nix/manual/#chap-installation). But for the sake of experimenting in case you don't wanna install Nix just yet, you can get away with using their Nix Docker image `nixos/nix`, this of course also would grant you more cool points because Docker.

TL;DR:
- If you wanna install it on your system NOW, you can run `sh <(curl https://nixos.org/nix/install)`.
- If you wanna play around with it a little before you install it:
  - You can use docker `docker run -it nixos/nix`.
  - Or you can install it in some VM.

## Nix store and profiles in action

Alright, so before we end this part, let's do a little experiment to show off the above jargon in action. I'm starting of from a freshly pulled `nixos/nix` Docker image just for the sake of experimentation for now.

First, this is the list of Nix commands we'll be needing, they all come from the `nix-env` binary.

- `nix-env -i <pkg-name>`: used to install some package.
- `nix-env --rollback`: used to rollback the state of the system.

Okay, now that that's out of the way, off to our dummy demo! Let's start by installing AWS CLI by running `nix-env -i awscli`, here we are instructing Nix to install a package named Git for us, if we check the logs we find the following,

<details>
<summary>
You can expand to see the full log if you want to, but let's just focus on this portion

```
--- snip ---
  /nix/store/ihy2vly61ndky6qlv1q4dfdiv28vszkh-python3-3.7.7
--- snip ---
copying path '/nix/store/ihy2vly61ndky6qlv1q4dfdiv28vszkh-python3-3.7.7' from 'https://cache.nixos.org'...
--- snip ---
```
</summary>

```
$ nix-env -i awscli
installing 'awscli-1.17.13'
these paths will be fetched (29.65 MiB download, 178.34 MiB unpacked):
  /nix/store/1adrbrb250ymy1x5g65n40r45ny71958-python3.7-PyYAML-5.2
  /nix/store/1cygbkyil4flvw8q1fpxn6m783xkxbdq-libffi-3.3
  /nix/store/36id84qj9lxnckdp744y2fina6ghj6rj-python3.7-six-1.14.0
  /nix/store/3ir36s2d3vvy7kwh195m409gh4dvia1b-python3.7-bcdoc-0.16.0
  /nix/store/41r6ggm6q6r2h3qdld7c3lfwa958vyky-python3.7-cryptography-2.9.1-dev
  /nix/store/4c8m0xvx32zlwvdvv9qjaslxb932j8kv-python3.7-jmespath-0.9.5
  /nix/store/5spl4xmmbxqs9bvdqw8crnn7iy31bj04-python3.7-setuptools_scm-3.4.3
  /nix/store/616wv2cx19id1vlmyqgafsxdghl60z3l-python3.7-pycparser-2.19
  /nix/store/8gf4lbnnjb26fdmkihcwmxs0k02a37ad-python3.7-pysocks-1.7.1
  /nix/store/9l6d9k9f0i9pnkfjkvsm7xicpzn4cv2c-libidn2-2.3.0
  /nix/store/ak6y0lprj7rg6fa48fddm48vh4d9xv41-python3.7-docutils-0.16
  /nix/store/awv6k7rjz527ad0vgr5ysf1rz2r5m151-python3.7-botocore-1.14.13
  /nix/store/axg1h1qjmm32l2kzv29ypm11dm8s663n-expat-2.2.8
  /nix/store/b7j954is4iwdaxhpmsfn927cy6k0jhnl-libyaml-0.2.4
  /nix/store/bvjzy3bw33jdp2bzmapsmnxq06dd6cgy-groff-1.22.4
  /nix/store/bwzra330vib0ik4d3l8rq6gp6y2ah1fr-glibc-2.30
  /nix/store/cvnhd6xmn4dnv2q95z08l1b4w9x3nbw2-python3.7-pyOpenSSL-19.1.0
  /nix/store/fn92v9h7gxj9w4z0s2qxsjxb4awczl3h-ncurses-6.2
  /nix/store/fw660zwgpp8hvrq7wpg0wjyx4b89j243-python3.7-pyOpenSSL-19.1.0-dev
  /nix/store/g92iwf82casdmrd7pwqv3hcmc3d00046-python3.7-certifi-2019.11.28
  /nix/store/gs8vlv30rvsbwqba5hb7z79vwild8rkm-readline-6.3p08
  /nix/store/hjy5c5c46gd2wz5k4qqzlrbda0r1igsn-python3.7-urllib3-1.25.8
  /nix/store/hk87010wz87bqvg5az5vzpsxhq9bl9ky-zlib-1.2.11
  /nix/store/i9q02g2f05x41s9yxcn1y4awqhwny6c1-openssl-1.1.1g
  /nix/store/ihy2vly61ndky6qlv1q4dfdiv28vszkh-python3-3.7.7
  /nix/store/ijax7siria3laky7c3dg515725z5w9lw-python3.7-s3transfer-0.3.3
  /nix/store/is24cfjpdxfn1a9g10hralmg7dsp0lq3-bzip2-1.0.6.0.1
  /nix/store/kgp3vq8l9yb8mzghbw83kyr3f26yqvsz-bash-4.4-p23
  /nix/store/khqrq2sqjwdagd1lddxrk5hm839xb6rl-python3.7-rsa-3.4.2
  /nix/store/kl1crmayilmk5f4wlafsv6cxs1cgczz5-awscli-1.17.13
  /nix/store/knf2swcf77p51xh9xm9vijfw0ax84h09-python3.7-colorama-0.4.3
  /nix/store/ly2qp12q8b0jvdjqkp9qibnfcd52anbk-python3.7-ordereddict-1.1
  /nix/store/mdsyxg6f6sk2w7v5s3sxrx75j4g7kwnd-python3.7-idna-2.8
  /nix/store/mh51mmx4l2b3pgn92hyqv25pbz8j88a1-python3.7-packaging-20.1
  /nix/store/n587ax9k9rc5cxm66aiibsss60pqk6b4-gcc-9.3.0-lib
  /nix/store/n88bdh9b8zw55j7mgb6rm8q0vbcp1c8y-python3.7-cffi-1.14.0
  /nix/store/pgj5vsdly7n4rc8jax3x3sill06l44qp-libunistring-0.9.10
  /nix/store/sk0fg37dhmv3f6a7b6flc3vj6kzgxx4i-python3.7-cffi-1.14.0-dev
  /nix/store/swpxsj84v4m39fbc4wd7gypmw5hcn7p6-gdbm-1.18.1
  /nix/store/vds2vrmxwin9dx4xn1hgv2b9v8fb2dgn-python3.7-python-dateutil-2.8.1
  /nix/store/vqhivcqzccmr4rx74zzbnw05yrkg1djy-python3.7-pyasn1-0.4.8
  /nix/store/w3ag8l2cqqh8d7klqsg4ipgjlzawmms3-sqlite-3.31.1
  /nix/store/wgwbrbca0cwsmpqlx5yq7cz4pbn2hlmv-python3.7-cryptography-2.9.1
  /nix/store/wvjn62fs1v2vdqwcrv11l2svinrz5fmc-python3.7-pyparsing-2.4.6
  /nix/store/wx3xk3sk6gxr5sgkd1kbg0g6fgjh0fv0-xz-5.2.5
  /nix/store/y1f478vsrdbym1x0vikpw2x63spbdfdp-libffi-3.3-dev
  /nix/store/y3iic1x1bifc6wrpir6l0rh52wk2nq7w-less-551
  /nix/store/z0vcl1zv6qcjmdakybx23zjklxgbkr56-python3.7-ply-3.11
  /nix/store/zix5m45ka1vf7iss04ylfgagym9iwanx-python3.7-simplejson-3.17.0
copying path '/nix/store/cvnhd6xmn4dnv2q95z08l1b4w9x3nbw2-python3.7-pyOpenSSL-19.1.0' from 'https://cache.nixos.org'...
copying path '/nix/store/pgj5vsdly7n4rc8jax3x3sill06l44qp-libunistring-0.9.10' from 'https://cache.nixos.org'...
copying path '/nix/store/9l6d9k9f0i9pnkfjkvsm7xicpzn4cv2c-libidn2-2.3.0' from 'https://cache.nixos.org'...
copying path '/nix/store/bwzra330vib0ik4d3l8rq6gp6y2ah1fr-glibc-2.30' from 'https://cache.nixos.org'...
copying path '/nix/store/kgp3vq8l9yb8mzghbw83kyr3f26yqvsz-bash-4.4-p23' from 'https://cache.nixos.org'...
copying path '/nix/store/is24cfjpdxfn1a9g10hralmg7dsp0lq3-bzip2-1.0.6.0.1' from 'https://cache.nixos.org'...
copying path '/nix/store/axg1h1qjmm32l2kzv29ypm11dm8s663n-expat-2.2.8' from 'https://cache.nixos.org'...
copying path '/nix/store/n587ax9k9rc5cxm66aiibsss60pqk6b4-gcc-9.3.0-lib' from 'https://cache.nixos.org'...
copying path '/nix/store/swpxsj84v4m39fbc4wd7gypmw5hcn7p6-gdbm-1.18.1' from 'https://cache.nixos.org'...
copying path '/nix/store/bvjzy3bw33jdp2bzmapsmnxq06dd6cgy-groff-1.22.4' from 'https://cache.nixos.org'...
copying path '/nix/store/1cygbkyil4flvw8q1fpxn6m783xkxbdq-libffi-3.3' from 'https://cache.nixos.org'...
copying path '/nix/store/b7j954is4iwdaxhpmsfn927cy6k0jhnl-libyaml-0.2.4' from 'https://cache.nixos.org'...
copying path '/nix/store/y1f478vsrdbym1x0vikpw2x63spbdfdp-libffi-3.3-dev' from 'https://cache.nixos.org'...
copying path '/nix/store/fn92v9h7gxj9w4z0s2qxsjxb4awczl3h-ncurses-6.2' from 'https://cache.nixos.org'...
copying path '/nix/store/i9q02g2f05x41s9yxcn1y4awqhwny6c1-openssl-1.1.1g' from 'https://cache.nixos.org'...
copying path '/nix/store/y3iic1x1bifc6wrpir6l0rh52wk2nq7w-less-551' from 'https://cache.nixos.org'...
copying path '/nix/store/gs8vlv30rvsbwqba5hb7z79vwild8rkm-readline-6.3p08' from 'https://cache.nixos.org'...
copying path '/nix/store/wx3xk3sk6gxr5sgkd1kbg0g6fgjh0fv0-xz-5.2.5' from 'https://cache.nixos.org'...
copying path '/nix/store/hk87010wz87bqvg5az5vzpsxhq9bl9ky-zlib-1.2.11' from 'https://cache.nixos.org'...
copying path '/nix/store/w3ag8l2cqqh8d7klqsg4ipgjlzawmms3-sqlite-3.31.1' from 'https://cache.nixos.org'...
copying path '/nix/store/ihy2vly61ndky6qlv1q4dfdiv28vszkh-python3-3.7.7' from 'https://cache.nixos.org'...
copying path '/nix/store/1adrbrb250ymy1x5g65n40r45ny71958-python3.7-PyYAML-5.2' from 'https://cache.nixos.org'...
copying path '/nix/store/3ir36s2d3vvy7kwh195m409gh4dvia1b-python3.7-bcdoc-0.16.0' from 'https://cache.nixos.org'...
copying path '/nix/store/g92iwf82casdmrd7pwqv3hcmc3d00046-python3.7-certifi-2019.11.28' from 'https://cache.nixos.org'...
copying path '/nix/store/n88bdh9b8zw55j7mgb6rm8q0vbcp1c8y-python3.7-cffi-1.14.0' from 'https://cache.nixos.org'...
copying path '/nix/store/knf2swcf77p51xh9xm9vijfw0ax84h09-python3.7-colorama-0.4.3' from 'https://cache.nixos.org'...
copying path '/nix/store/wgwbrbca0cwsmpqlx5yq7cz4pbn2hlmv-python3.7-cryptography-2.9.1' from 'https://cache.nixos.org'...
copying path '/nix/store/ak6y0lprj7rg6fa48fddm48vh4d9xv41-python3.7-docutils-0.16' from 'https://cache.nixos.org'...
copying path '/nix/store/mdsyxg6f6sk2w7v5s3sxrx75j4g7kwnd-python3.7-idna-2.8' from 'https://cache.nixos.org'...
copying path '/nix/store/ly2qp12q8b0jvdjqkp9qibnfcd52anbk-python3.7-ordereddict-1.1' from 'https://cache.nixos.org'...
copying path '/nix/store/z0vcl1zv6qcjmdakybx23zjklxgbkr56-python3.7-ply-3.11' from 'https://cache.nixos.org'...
copying path '/nix/store/vqhivcqzccmr4rx74zzbnw05yrkg1djy-python3.7-pyasn1-0.4.8' from 'https://cache.nixos.org'...
copying path '/nix/store/4c8m0xvx32zlwvdvv9qjaslxb932j8kv-python3.7-jmespath-0.9.5' from 'https://cache.nixos.org'...
copying path '/nix/store/616wv2cx19id1vlmyqgafsxdghl60z3l-python3.7-pycparser-2.19' from 'https://cache.nixos.org'...
copying path '/nix/store/wvjn62fs1v2vdqwcrv11l2svinrz5fmc-python3.7-pyparsing-2.4.6' from 'https://cache.nixos.org'...
copying path '/nix/store/sk0fg37dhmv3f6a7b6flc3vj6kzgxx4i-python3.7-cffi-1.14.0-dev' from 'https://cache.nixos.org'...
copying path '/nix/store/8gf4lbnnjb26fdmkihcwmxs0k02a37ad-python3.7-pysocks-1.7.1' from 'https://cache.nixos.org'...
copying path '/nix/store/khqrq2sqjwdagd1lddxrk5hm839xb6rl-python3.7-rsa-3.4.2' from 'https://cache.nixos.org'...
copying path '/nix/store/5spl4xmmbxqs9bvdqw8crnn7iy31bj04-python3.7-setuptools_scm-3.4.3' from 'https://cache.nixos.org'...
copying path '/nix/store/zix5m45ka1vf7iss04ylfgagym9iwanx-python3.7-simplejson-3.17.0' from 'https://cache.nixos.org'...
copying path '/nix/store/36id84qj9lxnckdp744y2fina6ghj6rj-python3.7-six-1.14.0' from 'https://cache.nixos.org'...
copying path '/nix/store/mh51mmx4l2b3pgn92hyqv25pbz8j88a1-python3.7-packaging-20.1' from 'https://cache.nixos.org'...
copying path '/nix/store/vds2vrmxwin9dx4xn1hgv2b9v8fb2dgn-python3.7-python-dateutil-2.8.1' from 'https://cache.nixos.org'...
copying path '/nix/store/41r6ggm6q6r2h3qdld7c3lfwa958vyky-python3.7-cryptography-2.9.1-dev' from 'https://cache.nixos.org'...
copying path '/nix/store/fw660zwgpp8hvrq7wpg0wjyx4b89j243-python3.7-pyOpenSSL-19.1.0-dev' from 'https://cache.nixos.org'...
copying path '/nix/store/hjy5c5c46gd2wz5k4qqzlrbda0r1igsn-python3.7-urllib3-1.25.8' from 'https://cache.nixos.org'...
copying path '/nix/store/awv6k7rjz527ad0vgr5ysf1rz2r5m151-python3.7-botocore-1.14.13' from 'https://cache.nixos.org'...
copying path '/nix/store/ijax7siria3laky7c3dg515725z5w9lw-python3.7-s3transfer-0.3.3' from 'https://cache.nixos.org'...
copying path '/nix/store/kl1crmayilmk5f4wlafsv6cxs1cgczz5-awscli-1.17.13' from 'https://cache.nixos.org'...
building '/nix/store/cd9b8y0z1xav9g1llzp8y5gnwixinj02-user-environment.drv'...
created 42 symlinks in user environment
```
</details>

Here we can see that when installing the AWS CLI, Python 3 was installed as a dependency, in a normal package manager, this would mean now we can use this installed Bash since it would be installed globally, so let's try and get the version of Python3 on the `nixos/nix` image.

```
$ python3 --version
/bin/sh: python3: not found
$ python --version
/bin/sh: python: not found
```

Oh, not exactly what we expect, for some reason Python isn't "installed", but how come so. Well this is the magic of symlinks coupled with the Nix store and Nix profiles! Although Python was installed to satisfy AWS CLI, it was not globally added anywhere, so the system isn't aware that Python exists, thus if I have a conflciting version of Python I would have no issues what so ever.

And if say I wanna write anything in Python and later remove the AWS CLI I won't be surprised that Python doesn't exist after removing the AWS CLI, because Python isn't "installed" I have to explicitly install it to be able to use it thus preventing it from being deleted should I remove the AWS CLI!

Python is still installed somewhere in the system, specifically in the Nix store, from the log you can tell it was installed to `/nix/store/ihy2vly61ndky6qlv1q4dfdiv28vszkh-python3-3.7.7`, so let's check that folder out!

```
$ ls /nix/store/ihy2vly61ndky6qlv1q4dfdiv28vszkh-python3-3.7.7
bin          include      lib          nix-support  share
$ ls /nix/store/ihy2vly61ndky6qlv1q4dfdiv28vszkh-python3-3.7.7/bin
2to3               idle3              pydoc3             python-config      python3.7          python3.7m-config
2to3-3.7           idle3.7            pydoc3.7           python3            python3.7-config   pyvenv
idle               pydoc              python             python3-config     python3.7m         pyvenv-3.7
$ /nix/store/ihy2vly61ndky6qlv1q4dfdiv28vszkh-python3-3.7.7/bin/python --version
Python 3.7.7
```

Doing the same thing in another package manager, say APK from Alpine Linux leads to a different result.

```
$ docker run -it alpine
--- snip ---
$ apk update
--- snip ---
$ apk add aws-cli
--- snip ---
$ aws --version
aws-cli/1.18.55 Python/3.8.3 Linux/4.19.76-linuxkit botocore/1.16.16
$ python3 --version
Python 3.8.3
```

The thing to note from the Alpine example is that Python 3 was gobally installed, thus if I already had a version of it conflciting with what the AWS CLI wanted I could be in trouble! Or if I needed a version that is incompatible with AWS CLI I would be blocked and have to do some kind of workaround to get both the AWS CLI and the version of Python I want!
