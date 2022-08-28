# Level 3

![Screenshot from 2022-08-28 14-43-00](https://user-images.githubusercontent.com/82754379/187061288-e9a0aca9-d607-4972-b7e9-c0b18f3af65b.png)

Running the **file** command on the provided *PATIENT0* file gives the following output:
```bash
$ file PATIENT0
PATIENT0: DOS/MBR boot sector, code offset 0x52+2, OEM-ID "NTFS    ", sectors/cluster 8, Media descriptor 0xf8, 
sectors/track 0, FAT (1Y bit by descriptor); NTFS, physical drive 0xab3566f7, sectors 12287, 
$MFT start cluster 4, $MFTMirror start cluster 767, bytes/RecordSegment 2^(-1*246), 
clusters/index block 1, serial number 05c66c6b160cddda1
```

So, this was a disk image which includes a DOS Master Boot Record, so I assume the corrupted bytes were in the first 512 bytes of the file.

Viewing the file in Ghex, the letters "TISC" at offset 0x20 immediately jump out.

![image](https://user-images.githubusercontent.com/82754379/187074799-9b0dced8-649c-43ca-95a4-6ff37f5776a6.png)

Assuming "TISC" was the first 4 of 8 contiguous corrupted bytes, the flag may be derived from the next 4 bytes (`F7 66 35 AB`).

<br>

:triangular_flag_on_post: **Level 3 Part 1 flag: `TISC{f76635ab}`**

<hr>

![Screenshot from 2022-08-28 19-45-02](https://user-images.githubusercontent.com/82754379/187075009-a7231499-61ea-4d15-abab-b64150021a0c.png)

This second part of Level 3 includes 3 hints:

![Screenshot from 2022-08-28 19-46-32](https://user-images.githubusercontent.com/82754379/187075062-33376b1d-b1d0-407a-9b8b-3cb1dd31d093.png)

![Screenshot from 2022-08-28 19-47-15](https://user-images.githubusercontent.com/82754379/187075072-1b91f634-f1d0-48a2-8892-b0d088d24246.png)

![Screenshot from 2022-08-28 19-47-40](https://user-images.githubusercontent.com/82754379/187075080-71f56ecf-a4e9-4f9f-a327-3505126688b1.png)

