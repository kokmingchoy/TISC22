# Level 3

![Screenshot from 2022-08-28 14-43-00](https://user-images.githubusercontent.com/82754379/187061288-e9a0aca9-d607-4972-b7e9-c0b18f3af65b.png)

Running the **file** command on the provided *PATIENT0* file gave the following output:
```bash
$ file PATIENT0
PATIENT0: DOS/MBR boot sector, code offset 0x52+2, OEM-ID "NTFS    ", sectors/cluster 8, Media descriptor 0xf8, 
sectors/track 0, FAT (1Y bit by descriptor); NTFS, physical drive 0xab3566f7, sectors 12287, 
$MFT start cluster 4, $MFTMirror start cluster 767, bytes/RecordSegment 2^(-1*246), 
clusters/index block 1, serial number 05c66c6b160cddda1
```

So, this was a disk image which included a DOS Master Boot Record, so I assumed the corrupted bytes were in the first 512 bytes of the file.

Viewing the file in Ghex, the letters "TISC" at offset 0x20 immediately jumped out.

![image](https://user-images.githubusercontent.com/82754379/187074799-9b0dced8-649c-43ca-95a4-6ff37f5776a6.png)

Assuming "TISC" was the first 4 of 8 contiguous corrupted bytes, the flag may be derived from the next 4 bytes (`F7 66 35 AB`).

<br>

:triangular_flag_on_post: **Level 3 Part 1 flag: `TISC{f76635ab}`**

<hr>

![Screenshot from 2022-08-28 19-45-02](https://user-images.githubusercontent.com/82754379/187075009-a7231499-61ea-4d15-abab-b64150021a0c.png)

This second part of Level 3 included 3 hints:

![Screenshot from 2022-08-28 19-46-32](https://user-images.githubusercontent.com/82754379/187075062-33376b1d-b1d0-407a-9b8b-3cb1dd31d093.png)

![Screenshot from 2022-08-28 19-47-15](https://user-images.githubusercontent.com/82754379/187075072-1b91f634-f1d0-48a2-8892-b0d088d24246.png)

![Screenshot from 2022-08-28 19-47-40](https://user-images.githubusercontent.com/82754379/187075080-71f56ecf-a4e9-4f9f-a327-3505126688b1.png)

Ran **binwalk** to identify what may be in that disk image:
```bash
$ binwalk PATIENT0

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
5312512       0x511000        PNG image, 1227 x 57, 8-bit/color RGB, non-interlaced
5312603       0x51105B        Zlib compressed data, compressed
5496832       0x53E000        PDF document, version: "1.7"
5496978       0x53E092        Zlib compressed data, default compression
5691000       0x56D678        Zlib compressed data, default compression
5869970       0x599192        JPEG image data, JFIF standard 1.01
5966510       0x5B0AAE        JPEG image data, JFIF standard 1.01
6004382       0x5B9E9E        Zlib compressed data, default compression
```

Ran **binwalk** again to extract the 8 files identified:
```bash
$ binwalk --dd='.*' PATIENT0
```

**binwalk** helpfully extracted all 8 files and even deflated those that were zlib-compressed.

After giving the files useful file name extensions based on what the command **file** reported, we have the following:
```bash
511000.png: PNG image data, 1227 x 57, 8-bit/color RGB, non-interlaced
51105B.bin: data
53E000.pdf: PDF document, version 1.7, 1 pages
53E092.ttf: TrueType Font data, digitally signed, 22 tables, 1st "DSIG", 45 names, Unicode, \251 2018 Microsoft Corporation. All Rights Reserved.
56D678.ttf: TrueType Font data, digitally signed, 22 tables, 1st "DSIG", 45 names, Unicode, \251 2018 Microsoft Corporation. All Rights Reserved.
599192.jpg: JPEG image data, JFIF standard 1.01, resolution (DPI), density 96x96, segment length 16, baseline, precision 8, 1882x1028, components 3
5B0AAE.jpg: JPEG image data, JFIF standard 1.01, resolution (DPI), density 96x96, segment length 16, baseline, precision 8, 1882x514, components 3
5B9E9E.txt: ASCII text, with very long lines (556)
```

Examining each file:

## File 1 - 511000.png
Looks like Base32-encoded string:
```
GIXFI2DJOJZXI6JAMZXXEIDUNBSSAZTMMFTT6ICHN4QGM2LOMQQHI2DFEBZXI4TFMFWS4CQ=
```
When Base32-decoded it read:
```
2.Thirsty for the flag? Go find the stream.
```

## File 2 - 51105B.bin
Binary data of unknown format. Could this be the "stream" in the clue from the Base32-encoded string?

## File 3 - 53E000.pdf
![image](https://user-images.githubusercontent.com/82754379/187078336-74d0bf0a-685c-4ca5-904d-a9da9db15ea7.png)

This was interesting. The initials BPB can also refer to the term "BIOS Parameter Block", which is where the 8 corrupted bytes were in the MBR of the disk image (from offset 0x20 through 0x27).

Was this an advice to determine what the correct 8 bytes were?

## File 4 - 53E092.ttf
Supposedly a Microsoft True Type Font file

## File 5 - 56D678.ttf
Supposedly a Microsoft True Type Font file

## File 6 - 599192.jpg
Looks like the top part of the image seen in **53E000.pdf**.

## File 7 - 5B0AAE.jpg
Looks like the bottom part of the image seen in **53E000.pdf**.

## File 8 - 5B9E9E.txt
Looks like some readings possibly from medical instrumentation (given the theme of this Level 3 challenge).


