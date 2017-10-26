# kernel environment
This is a pretty simple kernel environment using busybox, run with Qemu.

# Usage

If you want to try diffenrent kernel, just replace `bzImage`, to get `bzImage`, try compile yours.

If you want to upload any programs into file system, just copy it in `initramfs`, and run `mkcpio`.

Then run `./run` to run.

# How to make your own?

In case you want to make your very own environment like this, you need the following:
* busybox
* cpio
* gzip(maybe)

## basic file system
First, make a file system directory, and its structure:
```
mkdir initramfs
# standard file system entries
mkdir -pv bin lib dev etc mnt/root proc root sbin sys
```

Then compile busybox using `make`, just do these like it is said on busybox page. I don't need to mention that here, in case it changes.

Now you need to install busybox to the fs:
```
cd initramfs # we are at root of our fs
cp busybox/_install/* . -rv # copy those of busybox/_install/ directory here
```

What's left here is the init file, which will do some mount, typing these lines to `fs/init`:
```
#!/bin/sh

mount -t proc none /proc
mount -t sysfs none /sys
mount -t devtmpfs devtmpfs /dev

setsid cttyhack setuidgid 0 sh

umount /proc
umount /sys
poweroff -d 0 -f
```

And, basic file system is done.

## make cpio
File system like these cannot get into qemu, we need to make an cpio. This is done by using `cpio`, and it is like this:
```
cd initramfs
find . | cpio -o -H newc | gzip > ../rootfs.cpio
```
I just write this into a script file named mkcpio, you can do this manually as well. Note that you need to be in `initramfs` and do `find .`.

## run
Run using Qemu is not difficult, but the argument is too much:
```
qemu-system-x86_64 -initrd initrd.cpio.gz -kernel bzImage -append 'console=ttyS0 root=/dev/ram oops=panic panic=1' -enable-kvm -monitor /dev/null -m 64M --nographic  -smp cores=1,threads=1 -cpu kvm64,+smep -s
```

This doesn't has a lot to explain, you can also add your own argument in it, or do this manually.

Enjoy.
