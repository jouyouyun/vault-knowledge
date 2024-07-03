---
title: Booting Linux off of Google Drive
data: 2024-07-03T17:07:41+08:00
categories:
  - webclipper
tags:
  - boot
  - nfs
  - cloud
draft: false
origin-link: https://ersei.net/en/blog/fuse-root
---
Competitiveness is a vice of mine. When I heard that a friend got Linux to [boot off of NFS](https://www.kernel.org/doc/html/latest/admin-guide/nfs/nfsroot.html), I had to one-up her. I had to prove that I could create something _harder_, something _better_, _faster_, _stronger_.

Like all good projects, this began with an Idea.

My mind reached out and grabbed wispy tendrils from the æther, forcing the disparate concepts to coalesce. The Mass gained weight in my hands, and a dark, swirling colour promising doom to those who gazed into it for long.

On the brink of insanity, my tattered mind unable to comprehend the twisted interplay of millennia of arcane programmer-time and the ragged screech of madness, I reached into the Mass and steeled myself to the ground lest I be pulled in, and found my _magnum opus_.

Booting Linux off of a Google Drive root.

## But How?

I wanted this to remain self-contained, so I couldn't have a second machine act as a "helper". My mind went immediately to [FUSE](https://en.wikipedia.org/wiki/Filesystem_in_Userspace)—a program that acts as a filesystem driver in userspace (with cooperation from the kernel).

I just had to get FUSE programs installed in the Linux kernel [initramfs](https://www.kernel.org/doc/html/latest/filesystems/ramfs-rootfs-initramfs.html) and configure networking. How bad could it be?

## The Linux Boot Process

The Linux boot process is, technically speaking, very funny. Allow me to pretend I understand for a moment<sup id="fnref1:1" data-immersive-translate-walked="20296f53-7fa3-4ee5-8746-2c26e9dd8fa1"><a href="https://ersei.net/en/blog/fuse-root#fn:1">1</a></sup>:

1.  The firmware (BIOS/UEFI) starts up and loads the bootloader
2.  The bootloader loads the kernel
3.  The kernel unpacks a temporary filesystem into RAM which has the tools to mount the real filesystem
4.  The kernel mounts the real filesystem and switches the process to the init system running on the new filesystem

As strange as the third step may seem, it's very helpful! We can mount a FUSE filesystem in that step and boot normally.

## A Proof of Concept
The initramfs needs to have both network support as well as the proper FUSE binaries. Thankfully, [Dracut](https://github.com/dracutdevs/dracut) makes it easy enough to build a custom initramfs.

I decide to build this on top of Arch Linux because it's relatively lightweight and I'm familiar with how it works, as opposed to something like Alpine.

```shell
$ git clone https://github.com/dracutdevs/dracut $ podman run -it --name arch -v ./dracut:/dracut docker.io/archlinux:latest bash
```

In the container, I installed some packages (including the `linux` package because I need a functioning kernel), compiled `dracut` from source, and wrote a simple module script in `modules.d/90fuse/module-setup.sh`:

```bash
#!/bin/bash check() { require_binaries fusermount fuseiso mkisofs || return 1 return 0 } depends() { return 0 } install() { inst_multiple fusermount fuseiso mkisofs return 0 }
```

That's it. That's all the code I had to write. Buoyed by my newfound confidence, I powered ahead, building the EFI image.

```shell
$ ./dracut.sh --kver 6.9.6-arch1-1 \ --uefi efi_firmware/EFI/BOOT/BOOTX64.efi \ --force -l -N --no-hostonly-cmdline \ --modules "base bash fuse shutdown network" \ --add-drivers "target_core_mod target_core_file e1000" \ --kernel-cmdline "ip=dhcp rd.shell=1 console=ttyS0" $ qemu-kvm -bios ./FV/OVMF.fd -m 4G \ -drive format=raw,file=fat:rw:./efi_firmware \ -netdev user,id=network0 -device e1000,netdev=network0 -nographic ... ... dracut Warning: dracut: FATAL: No or empty root= argument dracut Warning: dracut: Refusing to continue Generating "/run/initramfs/rdsosreport.txt" You might want to save "/run/initramfs/rdsosreport.txt" to a USB stick or /boot after mounting them and attach it to a bug report. To get more debug information in the report, reboot with "rd.debug" added to the kernel command line. Dropping to debug shell. dracut:/#
```

_Hacker voice_ I'm in. Now to enable networking and mount a test root. I have already extracted an Arch Linux root into a S3 bucket running locally, so this should be pretty easy, right? I just have to manually set up networking routes and load the drivers.

```shell
dracut:/# modprobe fuse dracut:/# modprobe e1000 dracut:/# ip link set lo up dracut:/# ip link set eth0 up dracut:/# dhclient eth0 dhcp: PREINIT eth0 up dhcp: BOUND setting up eth0 dracut:/# ip route add default via 10.0.2.2 dev eth0 proto dhcp src 10.0.2.15 dracut:/# s3fs -o url=http://192.168.2.209:9000 -o use_path_request_style fuse /sysroot dracut:/# ls /sysroot bin dev home lib64 opt root sbin sys usr boot etc lib mnt proc run srv tmp var dracut:/# switch_root /sysroot /sbin/init switch_root: failed to execute /lib/systemd/systemd: Input/output error dracut:/# ls sh: ls: command not found
```

Honestly, I don't know what I expected. Seems like everything is just... _gone_. Alas, not even tab completion can save me. At this point, I was stuck. I had no idea what to do. I spent days just looking around, poking at the `switch_root` source code, all for naught. Until I remembered a link [Anthony](https://a.exozy.me/) had sent me: [How to shrink root filesystem without booting a livecd](https://unix.stackexchange.com/questions/226872/how-to-shrink-root-filesystem-without-booting-a-livecd/227318#227318). In there, there was a command called `pivot_root` that `switch_root` seems to call internally. Let's try that out.

```shell
dracut:/# logout ... [ 430.817269] ---[ end Kernel panic - not syncing: Attempted to kill init! exitcode=0x00000100 ]--- ... dracut:/# cd /sysroot dracut:/sysroot# mkdir oldroot dracut:/sysroot# pivot_root . oldroot pivot_root: failed to change root from `.' to `oldroot': Invalid argument
```

Apparently, `pivot_root` is [not allowed](https://unix.stackexchange.com/a/455224) to pivot roots if the root being switched is in the initramfs. Unfortunate. The Stack Exchange answer tells me to use `switch_root`, which doesn't work either. However, part of that answer sticks out to me:

> initramfs is rootfs: you can neither pivot\_root rootfs, nor unmount it. Instead delete everything out of rootfs to free up the space (find -xdev / -exec rm '{}' ';'), overmount rootfs with the new root (cd /newmount; mount --move . /; chroot .), attach stdin/stdout/stderr to the new /dev/console, and exec the new init.

Would it be possible to manually switch the root _without_ a specialized system call? What if I just chroot?

```shell
... dracut:/# mount --rbind /sys /sysroot/sys dracut:/# mount --rbind /dev /sysroot/dev dracut:/# mount -t proc /proc /sysroot/proc dracut:/# chroot /sysroot /sbin/init Explicit --user argument required to run as user manager.
```

Oh, I need to run the `chroot` command as PID 1 so Systemd can start up properly. I can actually tweak the initramfs's init script and just put my startup commands in there, and replace the `switch_root` call with `exec chroot /sbin/init`.

I put this in `modules.d/99base/init.sh` in the Dracut source after the udev rules are loaded and bypassed the `root` variable checks earlier.

```bash
modprobe fuse modprobe e1000 ip link set lo up ip link set eth0 up dhclient eth0 ip route add default via 10.0.2.2 dev eth0 proto dhcp src 10.0.2.15 s3fs -o url=http://192.168.2.209:9000 -o use_path_request_style fuse /sysroot mount --rbind /sys /sysroot/sys mount --rbind /dev /sysroot/dev mount -t proc /proc /sysroot/proc
```

I also added `exec chroot /sysroot /sbin/init` at the end instead of the `switch_root` command.

Rebuilding the EFI image and...

![A screenshot of a Linux login screen](https://ersei.net/user/pages/03.blog/40.fuse-root/itworks.png)

I sit there, in front of my computer, staring. It can't have been that easy, can it? Surely, this is a profane act, and the spirit of Dennis Ritchie ought't've stopped me, right?

Nobody stopped me, so I kept going.

I log in with the very secure password `root` as `root`, and it unceremoniously drops me into a shell.

```shell
[root@archlinux ~]# mount s3fs on / type fuse.s3fs (rw,nosuid,nodev,relatime,user_id=0,group_id=0) ... [root@archlinux ~]#
```

At last, Linux booted off of an S3 bucket. I was compelled to share my achievement with others—all I needed was a fetch program to include in the screenshot:

```shell
[root@archlinux ~]# pacman -Sy fastfetch :: Synchronizing package databases... core.db failed to download error: failed retrieving file 'core.db' from geo.mirror.pkgbuild.com : Could not resolve host: geo.mirror.pkgbuild.com warning: fatal error from geo.mirror.pkgbuild.com, skipping for the remainder of this transaction error: failed retrieving file 'core.db' from mirror.rackspace.com : Could not resolve host: mirror.rackspace.com warning: fatal error from mirror.rackspace.com, skipping for the remainder of this transaction error: failed retrieving file 'core.db' from mirror.leaseweb.net : Could not resolve host: mirror.leaseweb.net warning: fatal error from mirror.leaseweb.net, skipping for the remainder of this transaction error: failed to synchronize all databases (invalid url for server) [root@archlinux ~]#
```

Uh, seems like DNS isn't working, and I'm missing `dig` and other debugging tools.

Wait a minute! My root filesystem is on S3! I can just mount it somewhere else with functional networking, `chroot` in, and install all my utilities!

Some debugging later, it seems like systemd-resolved doesn't want to run because it `Failed to connect stdout to the journal socket, ignoring: Permission denied`. I'm not about to try to debug systemd because it's too complicated and I'm lazy, so instead I'll just use Cloudflare's.

```shell
[root@archlinux ~]# echo "nameserver 1.1.1.1" > /etc/resolv.conf [root@archlinux ~]# pacman -Sy fastfetch :: Synchronizing package databases... core is up to date extra is up to date ... [root@archlinux ~]# fastfetch
```

![Fastfetch showing the system running in QEMU](https://ersei.net/user/pages/03.blog/40.fuse-root/fastfetch.png)

I look around, making sure that nobody had tried to stop me. My window was intact, my security system had not tripped, the various canaries I had set up around the house had not been touched. I was safe to continue.

I was ready to have it run on Google Drive.

## Google Gets Involved
There's a project already that does Google Drive over FUSE for me already: [google-drive-ocamlfuse](https://github.com/astrada/google-drive-ocamlfuse). Thankfully, I have a Google account lying around that I haven't touched in years ready to go! I follow the instructions, accept the terms of service I didn't read, create all the oauth2 secrets, enable the APIs, install `google-drive-ocamlfuse` from the AUR into my Arch Linux VM, patch some `PKGBUILD`s (it's been a while), and lo and behold! I have mounted Google Drive! Mounting Drive and a few _very long_`rsync` runs later, I have Arch Linux on Google Drive.

Just kidding, it's never that easy. Here's a non-exhausive list of problems I ran into:

1.  Symlinks to symlinks don't work (very important for stuff in `/usr/lib`)
2.  Hardlinks don't work
3.  It's so slowwwww
4.  Relative symlinks don't work at all
5.  No dangling symlinks (important for stuff that links to `/proc` and isn't mounted, or stuff that just hasn't copied over yet)
6.  Symlinks outside of Google Drive don't work
7.  Permissions don't work (neither do attributes)
8.  Did I mention it's SLOW

With how many problems there are with symlinks, I have half a mind to change the FUSE driver code to just create a file that ends in `.internalsymlink` to fix all of that, Google Drive compatibility be damned.

But, I have challenged myself to do this without modifying anything important (no kernel tweaking, no FUSE driver tweaking), so I'll just have to live with it and manually create the symlinks that `rsync` fails to make with a hacky `sed` command to the `rsync` error logs.

In the meantime, I added the token files generated from my laptop into the initramfs, as well as the Google Drive FUSE binary and SSL certificates, and tweaked a few settings<sup id="fnref1:2" data-immersive-translate-walked="20296f53-7fa3-4ee5-8746-2c26e9dd8fa1"><a href="https://ersei.net/en/blog/fuse-root#fn:2">2</a></sup> to make my life slighty easier.

```bash
... inst ./gdfuse-config /.gdfuse/default/config inst ./gdfuse-state /.gdfuse/default/state find /etc/ssl -type f -or -type l | while read file; do inst "$file"; done find /etc/ca-certificates -type f -or -type l | while read file; do inst "$file"; done ...
```

![A screenshot of Google Drive showing the root of a typical Linux filesystem](https://ersei.net/user/pages/03.blog/40.fuse-root/google-drive-root.png)

It's nice to see that timestamps kinda work, at least. Now all that's left is to wait for the agonizingly slow boot!

```
chroot: /sbin/init: File not found
```

Perhaps they did not bother to stop me because they knew I would fail.

I know the file exists since, well, it _exists_, so why is it not found? Simple: Linux is kinda weird and if the binary you call depends on a library that's not found, then you'll get "File not found".

```shell
dracut:/# ldd /sysroot/bin/bash linux-vdso.so.1 (0x00007e122b196000) libreadline.so.8 => /usr/lib/libreadline.so.8 (0x00007e122b01a000) libc.so.6 => /usr/lib/libc.so.6 (0x00007e122ae2e000) libncursesw.so.6 => /usr/lib/libncursesw.so.6 (0x00007e122adbf000) /lib64/ld-linux-x86-64.so.2 => /usr/lib64/ld-linux-x86-64.so.2 (0x00007e122b198000)
```

However, these symlinks don't actually exist! Remember how earlier we noted that relative symlinks don't work? Well, that's come back to bite me. The Kernel is looking for files in `/sysroot` inside `/sysroot/sysroot`. Luckily, this is an easy enough fix: we just need to have `/sysroot` linked to `/sysroot/sysroot` without links:

```shell
dracut:/# mkdir /sysroot/sysroot dracut:/# mount --rbind /sysroot /sysroot/sysroot
```

Now time to boot!

It took five minutes for Arch to rebuild the dynamic linker cache, another minute per systemd unit, and then, nothing. The startup halted in its tracks.

```
[ TIME ] Timed out waiting for device /dev/ttyS0.
[DEPEND] Dependency failed for Serial Getty on ttyS0.
```

Guess I have to increase the timeout and reboot. In `/etc/systemd/system/dev-ttyS0.device`, I put:

```
[Unit]
Description=Serial device ttyS0
DefaultDependencies=no
Before=sysinit.target
JobTimeoutSec=infinity
```

Luckily, it did not take infinite time to boot.

![A Linux login prompt](https://ersei.net/user/pages/03.blog/40.fuse-root/gdrive-booted.png)

I'm so close to victory I can _taste_ it! I just have to increase _another_ timeout. I set `LOGIN_TIMEOUT` to `0` in `/etc/login.defs` in Google Drive, and tried logging in again.

Thankfully, there's a cache, so subsequent file reads aren't nearly as slow.

![Fastfetch in Google Drive root, showing that the root partition is mounted as fuse.google-drive-ocaml](https://ersei.net/user/pages/03.blog/40.fuse-root/gdrive-fastfetch.png)

Here I am, laurel crown perched upon my head, my chimera of Linux and Google Drive lurching around.

But I'm not satisfied yet. Nobody had stopped me because they _want_ me to succeed. I have to take this further. I need this to work on _real hardware_.

## Now Do It On Real Hardware

Fortunately for me, I [switched servers](https://ersei.net/en/blog/updates-2024-02) and now have an extra laptop with no storage just lying around! A wonderful victim<sup id="fnref1:3" data-immersive-translate-walked="20296f53-7fa3-4ee5-8746-2c26e9dd8fa1"><a href="https://ersei.net/en/blog/fuse-root#fn:3">3</a></sup> for my test!

There are a few changes I'll have to make:

1.  Use the right ethernet driver and not the default `e1000`
2.  Do not use a serial display
3.  Change the network settings to match my house's network topology

All I need is the `r8169` driver for my ethernet port, and let's throw in a [Powerline](https://en.wikipedia.org/wiki/Power-line_communication) into the mix, because it's not going to impact the performance in any way that matters, and I don't have an ethernet cord that can reach my room.

I build the unified EFI file, throw it on a USB drive under `/BOOT/EFI`, and stick it in my old server. Despite my best attempts, I couldn't figure out what the modprobe directive is for the laptop's built-in keyboard, so I just modprobed `hid_usb` and used an external keyboard to set up networking.

![A screenshot of fastfetch and mount on bare metal showing that we're booted off of Google Drive](https://ersei.net/user/pages/03.blog/40.fuse-root/bare-metal-gdrive.png)

This is my _magnum opus_. My Great Work. This is the mark I will leave on this planet long after I am gone: The Cloud Native Computer.

Nice thing is, I can just grab the screenshot<sup id="fnref1:screenshot" data-immersive-translate-walked="20296f53-7fa3-4ee5-8746-2c26e9dd8fa1"><a href="https://ersei.net/en/blog/fuse-root#fn:screenshot">4</a></sup> from Google Drive and put it here!

## Woe! Cloud Native Computer Be Upon Ye

Despite how silly this project is, there are a few less-silly uses I can think of, like booting Linux off of [SSH](https://github.com/libfuse/sshfs), or perhaps booting Linux off of a Git repository and tracking every change in Git using [gitfs](https://wiki.archlinux.org/title/Gitfs). The possibilities are endless, despite the middling usefulness.

If there is anything I know about technology, it's that moving everything to The Cloud is the current trend. As such, I am prepared to commercialize this for any company wishing to leave their unreliable hardware storage behind and move entirely to The Cloud. Please [request a quote](https://ersei.net/en/contact-me) if you are interested in True Cloud Native Computing.

Unfortunately, I don't know what to do next with this. Maybe I should install Nix?

___

Thoughts? Comments? Opinions? Feel free to share (relevant) ones with me! [Contact me here if you want.](https://ersei.net/en/contact-me)