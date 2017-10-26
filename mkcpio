#!/bin/sh

cd ./initramfs && (find . | cpio -o -H newc | gzip > ../rootfs.cpio)
