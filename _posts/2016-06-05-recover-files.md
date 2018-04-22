---
layout: post
cover: True
cover: 'https://www.infosec-csusb.org/media/forensicsbanner-300px.jpg'
title: "Recover deleted files forensically"
date:   2016-06-05 
tags: ["forensics", "IT security"]
categories: 'Forensics'
navigation: True
---
Hello everyone, today I would like to share this article with you guys, because most of us, and I am one of them, delete or format drive unexpectedly. For this post, I have 64GB USB drive that formatted as FAT32. 

requirements:
=> Linux operating system (Kali).
=> USB, internal or external disk that has deleted files.

For this post, I am not going to explain the theory behind it and why we are able to retrieve the deleted files. However, I will walk you through the steps to recover all deleted files practically! 

I am going to use Kali Linux as the main machine that will retrieve the data. You can use other operating systems. The first step is to check the full path of the sub drive name on the system by using the following command:
{% highlight bash %}
root@p0wnb0x:~# lsblk 
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 298.1G  0 disk 
├─sda1   8:1    0 290.3G  0 part /
├─sda2   8:2    0     1K  0 part 
└─sda5   8:5    0   7.8G  0 part [SWAP]
sdb      8:16   1  57.9G  0 disk 
└─sdb1   8:17   1  57.9G  0 part /media/root/5183-A036
sr0     11:0    1  1024M  0 rom  
{% endhighlight %}

Using `lsblk` command, we could identify all drives that have been connected to the computer. As obvious, my USB drive has `sdb` as a disk, and `sdb1` as a partition. Therefore, the full path will be `/dev/sdb` if you would like to recover all deleted files from the entire disk. Other cases, it could have more than partitions such as `sdb1` and `sdb2`. In my case, I will use `sdb1` because it represents the entire disk.

Another way to know the full path by using `fdisk` command:
{% highlight bash %}
root@p0wnb0x:~# fdisk -l
Disk /dev/sda: 298.1 GiB, 320072933376 bytes, 625142448 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: dos
Disk identifier: 0xe374435a

Device     Boot     Start       End   Sectors   Size Id Type
/dev/sda1  *         2048 608741375 608739328 290.3G 83 Linux
/dev/sda2       608743422 625141759  16398338   7.8G  5 Extended
/dev/sda5       608743424 625141759  16398336   7.8G 82 Linux swap / Solar

Partition 2 does not start on physical sector boundary.

Disk /dev/sdb: 57.9 GiB, 62109253632 bytes, 121307136 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x00000000

Device     Boot Start       End   Sectors  Size Id Type
/dev/sdb1          32 121307135 121307104 57.9G  c W95 FAT32 (LBA)
{% endhighlight %}

Next, I am going to make an exact copy of my USB drive into my Kali Linux using the forensics tools such as dd, dc3dd and other. `dd` is a tool that copies bit-by-bit from the destination! `dc3dd` is an advanced tool that based on `dd` command, and it has many forensics features such as log, hash, wipe and others. I will be using dc3dd to create an exact copy of my USB to my Desktop using the following command:
{% highlight bash %}
root@p0wnb0x:~# dc3dd if=/dev/sdb1 of=/root/Desktop/usb.dd 

dc3dd 7.2.641 started at 2016-06-04 19:37:23 -0400
compiled options:
command line: dc3dd if=/dev/sdb1 of=/root/Desktop/usb.dd
device size: 121307104 sectors (probed),   62,109,237,248 bytes
sector size: 512 bytes (probed)
 62109237248 bytes ( 58 G ) copied ( 100% ), 2387 s, 25 M/s                   

input results for device `/dev/sdb1':
   121307104 sectors in
   0 bad sectors replaced by zeros

output results for file `/root/Desktop/usb.dd':
   121307104 sectors out

dc3dd completed at 2016-06-04 20:17:10 -0400
root@p0wnb0x:~/Desktop# ls -lh
total 58G
-rw-r--r-- 1 root root 58G Jun  4 20:17 usb.dd
{% endhighlight %}

Please be careful when you using such these tools, because if you don't know how to deal with them, you may lose and delete your files!!!

Usually, for integrity, we calculate the md5 hash for the source and destination disk, but for now, it will take time since it has 64 GB, and it could take a while.

Now, I will be using the `usb.dd` and try to recover some files, but first, I would like to mount the drive to the system and check if there are any files.
{% highlight bash %}
root@p0wnb0x:~# mkdir -p /tmp/usb
root@p0wnb0x:~# ls /tmp |grep "usb"
usb
root@p0wnb0x:~# mount ~/Desktop/usb.dd /tmp/usb/
root@p0wnb0x:~# 
root@p0wnb0x:~# cd /tmp/usb/
root@p0wnb0x:/tmp/usb# ls -l

drwxr-xr-x 4 root root      16384 Mar 30 20:03 books
drwxr-xr-x 2 root root      16384 Feb 24  2015 SanDiskSecureAccess
drwxr-xr-x 2 root root      16384 Mar 18 21:14 System Volume Information
root@p0wnb0x:/tmp/usb# umount /tmp/usb 
{% endhighlight %}

Next, I will be using `foremost` to recover all kind of files including jpg,png,exe,pdf, and more others.

{% highlight bash %}
root@p0wnb0x# foremost -v -t all -i ~/Desktop/usb.dd -o ~/Desktop/recv 
.
.
.
4700:	121262816.jpg 	     330 KB 	 62086561792 	 
4701:	121263488.jpg 	     229 KB 	 62086905856 	 
4702:	121266624.jpg 	      25 KB 	 62088511488 	 
4703:	121266688.jpg 	     260 KB 	 62088544256 	 
4704:	121267232.jpg 	     395 KB 	 62088822784 	 
4705:	121268032.jpg 	       2 MB 	 62089232384 	 
4706:	121272448.jpg 	     401 KB 	 62091493376 	 
4707:	121273280.jpg 	     385 KB 	 62091919360 	 
4708:	121274080.jpg 	     364 KB 	 62092328960 	 
4709:	121274816.jpg 	     370 KB 	 62092705792 	 
4710:	121275584.jpg 	     382 KB 	 62093099008 	 
4711:	121276352.jpg 	     372 KB 	 62093492224 	 
4712:	121263968.png 	       1 MB 	 62087151616 	  (640 x 1136)
*|
Finish: Sat Jun  4 21:23:04 2016

4713 FILES EXTRACTED
	
jpg:= 4116
gif:= 70
bmp:= 1
mov:= 1
mp4:= 12
htm:= 43
zip:= 4
rar:= 1
exe:= 3
png:= 414
pdf:= 48
------------------------------------------------------------------

Foremost finished at Sat Jun  4 21:23:04 2016
{% endhighlight %}

As you can see, the result is fantastic; I have successfully recovered deleted files.
Many tools could do the task. You can search for them using Google or any security websites. For example, Kali Linux has many forensics tools such as `recoverjpeg` for recovering the jpeg files only and `recovermov` for `.mov` files.

I hope you enjoy the article and getting something new! Thank you!
