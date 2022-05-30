# ALP-PPSSPP

using prebuit image from batocera https://batocera.org/download and select RG552 model

## Get the image and extract 
Copy the 'Direct Link' URL and get the image.
For example, the URL is https://mirrors.o2switch.fr/batocera/rg552/stable/last/batocera-rg552-34-20220523.img.gz when writing this document
```
$ wget https://mirrors.o2switch.fr/batocera/rg552/stable/last/batocera-rg552-34-20220309.img.gz
$ gunzip ./batocera-rg552-34-20220309.img.gz
```
 
## Extract
```
$ fdisk -l ./batocera-rg552-34-20220309.img 
Disk ./batocera-rg552-34-20220309.img: 3.27 GiB, 3506438144 bytes, 6848512 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x00000000
 
Device                            Boot   Start     End Sectors  Size Id Type
./batocera-rg552-34-20220309.img1        32768 6324223 6291456    3G  c W95 FAT
./batocera-rg552-34-20220309.img2      6324224 6848511  524288  256M 83 Linux
```
## Mount the first partition
offset=32768*512=16777216
```
$ mkdir batocera_partition
$ sudo mount -o loop,offset=16777216  ./batocera-rg552-34-20220309.img batocera_partition
```

## copy the root filesystem from first partition
```
$ cp batocera_partition/boot/batocera .

## unmount it
$ sudo umount batocera_partition
 
## mount root filesystem
$ sudo mount batocera batocera_partition/
```
 
## copy PPSSPP binary and unmount
```
$ cp batocera_partition/usr/bin/PPSSPP  .
$ sudo umount batocera_partition 
```

## need to patch elf for PPSSPP binary on ALP
Ref: https://stackoverflow.com/questions/847179/multiple-glibc-libraries-on-a-single-host
```
$ source ~/rk3399env.sh
$ patchelf --set-interpreter mnt/lib/ld-linux-aarch64.so.1 --set-rpath mnt/lib/ ./PPSSPP
```
The patched path → mnt/lib (relative path)
```
$ file ./PPSSPP
./PPSSPP: ELF 64-bit LSB executable, ARM aarch64, version 1 (GNU/Linux), dynamically linked, interpreter mnt/lib/ld-linux-aarch64.so.1, for GNU/Linux 4.4.0, stripped
```

## copy SQUASHFS/lib and SQUASHFS/usr/lib to mnt/lib and mnt/usr/lib
```
$ sudo mount batocera mnt_batocera  # batocera is the squashfs file from batocera, mnt_batocera  is a mount directory
$ cp -P mnt_batocera/lib/* mnt/lib/
$ cp -P mnt_batocera/usr/lib/* mnt/usr/lib/ 
```

## launch step
```$ LD_PRELOAD=mnt/usr/lib/libstdc++.so.6:mnt/lib/libc.so.6  ./PPSSPP XXXX.cso```
