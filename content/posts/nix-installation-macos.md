---
title: "Nix Installation on macOS Catalina"
date: 2020-06-14T12:39:29+03:00
draft: true
tags: [100 Days, nixology]
---

This is kind of a part 1.5 to [Getting Started with Nix](/posts/getting-started-with-nix), we're going into a bit more of details regarding installation of Nix on macOS. This is also an execuse to dig in a little and write about Apple's changes to the filesystem in macOS Catalina, and other APFS concepts.

This can totally be skipped if you're not on any Linux based system, or if you don't care much about how APFS handles things under the hood and would rather to just blindly follow the instructions given by the Nix installer!

## APFS

APFS is Apple's filesystem on iOS (since iOS 10.3) and macOS (since 10.13 High Sierra). Conceptually APFS divides your disc into 2 layers, a container layer and a file system (volumes) layer.

The container layer exists within the disk's partitioning scheme and is the more high level of the two, it provides checkpoints for the actions executed in the drive, ensures those actions are atomic, and provides crash protection, among other things. It also contains volume metadata and snapshots, as well its encryption state.

The file-system layer, or the volume, contains the actual data structures that store information, you know your directory structures, file metadata, and file content. In APFS, Volumes can be assigned specific roles, one of 16 roles to be exact (as of macOS Catalina 10.15). This allows the system to treat each volume in a more optimized manner depending on its role, examples on the roles are preboot, virtual memory, or recovery volumes, those three were one of the original volume roles in APFS.

Partitions in APFS contain exactly one container, and containers contain inside them the volumes (file system layers) that host your actual data. Containers allow the volumes inside them to "share" the space allocated to the container, and in a sense allows each volume to dynamically expand or shrink as needed, within the confides of the container itself. However containers themselves requires more manual labor and are more risky to resize, as they can only be expanded in one direction and only into adjacent free space.

As an example of the quality of life improvements introduced by containers allowing inner volumes to resize dynamic, [taken from Apples APFS reference](https://developer.apple.com/support/downloads/Apple-File-System-Reference.pdf), a drive can be configured with two bootable volumes — one with a shipping version of macOS and one with a beta version — as well as a user data volume. All three of these volumes share free space, meaning you donʼt have to decide ahead of time how to divide space between them.

After the release of macOS Catalina, Apple introduces a fancy new concepts to APFS called volume groups. Volume groups ar a logical grouping of multiple volumes within a container, it allows the system to treat them in a special way, and creates the illusion that these two separate volumes are one.

Following is a diagram of APFS, courtesy of [https://bombich.com](https://bombich.com/kb/ccc5/working-apfs-volume-groups)

![APFS concepts](https://bombich.com/images/ccc5/apfs/apfs_concepts.png)

Volume groups are directly observable when you upgrade to macOS Catlina, as you will notice if you open the Disk Utility that the volume your OS used to have is split in two, "Macintosh HD" and "Macintosh HD - Data". Even though from a user precpetive they are treated as one single entity, both in Finder and in the terminal.

### Read only root system volume

> [macOS Catalina runs in a dedicated, read-only system volume — which means it is completely separate from all other data and helps improve the reliability of macOS.](https://www.apple.com/macos/catalina/features/#security)

As means to provide an extra layer of security, the OS itself is installed in a volume with the system read only role, `APFS_VOL_ROLE_SYSTEM`. This makes it practically impossible for an attacker to overwrite or manipulate the OS installation!

### The data volume

Not everything can reside in a volume that is readonly, that's why there exists a nother volume that contains all of the user data, as well as any system portions that require write access. This volume is always mounted but is effectively hidden from the users, in Finder it is practically mashed together with the system volume as to appear as one volume.

In order to make the volumes split transparent APFS supports what's called **firmlinks**. A firmlink conceptually similar symlink, except it only works on folders. It creates a "bi-directional wormwhole" that transparently merges folders across volumes and handle, transparently, the transition between one volume to the other when moving in the directory structure!

An great example of this is the `/Users` folder, it appears to reside in the root of the filesystem, but in reality it exists in the data volume, the `/Users` we see is a firmlink between the system readonly volume and the actual folder in the data volume!

You can list all system firmlinks by viewing the `/usr/share/firmlinks` file.

Firmlinks allow you to transperantly link up one volume as a folder in another, this can be used to give the illusion of writing to the system volume even though it's in read only mode always. There are multiple istances of this, Applications folder, user home folder, ... etc. But firmlinks are only allowed to be used by macOS itself, not by any userland programs.

`synthetic.conf` is the only way that macOS exposes a limited set of firmlinks capabilities for end users. ...

### Writing to root volume

<!-- To write to the system root volume ... -->

## Installing Nix

Ok, what the hell does any of that have to do with Nix? Well Nix uses the folder `/nix` for it's installation, as well as everything else it ever writes to disk. As you can tell this means that Nix needs write access to the now read-only system volume.

The officially endorsed way for installing Nix on macOS is by utilizing the `synthetic.conf` to add a firmlink under `/nix` that points to a user writable volume. This can be done manually if you want, or automatically by utilizing the

### Script internals

<!-- The default behaviour the installer supports. -->
<!-- Other options and their tradeoffs are listed here https://nixos.org/nix/manual/#sect-macos-installation. -->

### System state restore and encryption

<!-- keeping encryption while supporting system restore ? -->

## More readings

- [Nix manual, macOS Installation](https://nixos.org/nix/manual/#sect-macos-installation)
- [APFS Reference](https://developer.apple.com/support/downloads/Apple-File-System-Reference.pdf)
- [Working with APFS Volume Groups](https://bombich.com/kb/ccc5/working-apfs-volume-groups)
- [macOS Catalina Boot Volume Layout](https://eclecticlight.co/2019/10/08/macos-catalina-boot-volume-layout/)
- [Creating root-level directories and symbolic links on macOS Catalina](https://derflounder.wordpress.com/2020/01/18/creating-root-level-directories-and-symbolic-links-on-macos-catalina/)
- [What’s New in Apple Filesystems presentation slides](https://devstreaming-cdn.apple.com/videos/wwdc/2019/710aunvynji5emrl/710/710_whats_new_in_apple_file_systems.pdf)
