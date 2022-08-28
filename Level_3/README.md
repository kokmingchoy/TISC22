# Level 3

![Screenshot from 2022-08-28 14-43-00](https://user-images.githubusercontent.com/82754379/187061288-e9a0aca9-d607-4972-b7e9-c0b18f3af65b.png)

Running the **file** command on the provided *PATIENT0* file gives the following output:
```
PATIENT0: DOS/MBR boot sector, code offset 0x52+2, OEM-ID "NTFS    ", sectors/cluster 8, Media descriptor 0xf8, 
sectors/track 0, FAT (1Y bit by descriptor); NTFS, physical drive 0xab3566f7, sectors 12287, 
$MFT start cluster 4, $MFTMirror start cluster 767, bytes/RecordSegment 2^(-1*246), 
clusters/index block 1, serial number 05c66c6b160cddda1
```

Running the **fdisk** command on the provided *PATIENT0* file gives the following output:
```
$ fdisk -l PATIENT0
Disk PATIENT0: 6 MiB, 6291456 bytes, 12288 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x00000000
```

