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

This looked like an NTFS disk image with a Master Boot Record (MBR), so I assumed the corrupted bytes were in the first 512 bytes of the file.

Viewing the file in Ghex, the letters "TISC" at offset 0x20 immediately jumped out.

![image](https://user-images.githubusercontent.com/82754379/187074799-9b0dced8-649c-43ca-95a4-6ff37f5776a6.png)

Assuming "TISC" was the first 4 of 8 contiguous corrupted bytes, the flag may be derived from the next 4 bytes (`F7 66 35 AB`).

<br>

:triangular_flag_on_post: **Level 3 Part 1 flag: `TISC{f76635ab}`**

<hr>

![Screenshot from 2022-08-28 19-45-02](https://user-images.githubusercontent.com/82754379/187075009-a7231499-61ea-4d15-abab-b64150021a0c.png)

This second part of Level 3 included 4 hints:

![Screenshot from 2022-08-28 19-46-32](https://user-images.githubusercontent.com/82754379/187075062-33376b1d-b1d0-407a-9b8b-3cb1dd31d093.png)

![Screenshot from 2022-08-28 19-47-15](https://user-images.githubusercontent.com/82754379/187075072-1b91f634-f1d0-48a2-8892-b0d088d24246.png)

Doing an image search in Google on the above turned up [TrueCrypt](https://en.wikipedia.org/wiki/TrueCrypt), which can "can create a virtual encrypted disk within a file, or encrypt a partition or the whole storage device (pre-boot authentication)."

TrueCrypt has been discontinued, but there is a fork of it called [VeraCrypt](https://en.wikipedia.org/wiki/VeraCrypt) which I managed to install on my machine.

<br>

![image](https://user-images.githubusercontent.com/82754379/187083445-4695a255-62fa-4450-962b-1a54700f77b6.png)

![image](https://user-images.githubusercontent.com/82754379/188256547-09b12403-a76b-491b-ac72-37dd76475f38.png)

<hr width=2>

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

After giving the files relevant file name extensions, we have the following:
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
I'm not so sure because the beginning of this data (at offset 0x51105B) overlapped with part of the contents of **511000.png** which came from offset 0x511000. It could have been an error on the part of **binwalk** to consider it as data just because of the text string "DAT" that appeared just before this binary data in the disk image.


## File 3 - 53E000.pdf
![image](https://user-images.githubusercontent.com/82754379/187078336-74d0bf0a-685c-4ca5-904d-a9da9db15ea7.png)

This was interesting. The initials BPB can also refer to the term "BIOS Parameter Block", which is where the 8 corrupted bytes were in the MBR of the disk image (from offset 0x20 through 0x27).

Was this an advice to determine what the correct 8 bytes were?

## File 4 - 53E092.ttf
Supposedly a Microsoft True Type Font file.
Opened to show "Calibri, Bold".

## File 5 - 56D678.ttf
Supposedly a Microsoft True Type Font file
Opened to show "Calibri, Regular".

## File 6 - 599192.jpg
Looks like the top part of the image seen in **53E000.pdf**.

## File 7 - 5B0AAE.jpg
Looks like the bottom part of the image seen in **53E000.pdf**.

## File 8 - 5B9E9E.txt
Looks like some readings possibly from medical instrumentation (given the theme of this Level 3 challenge).

<hr height="2">

Trying to mount the given NTFS disk image ("PATIENT0") on my Ubuntu machine:
```
$ sudo mount -t ntfs -o ro PATIENT0 /mnt/patient0/ 

Reserved fields aren't zero (0, 0, 0, 0, 1129531732, 0). 
Failed to mount '/dev/loop42': Invalid argument 
The device '/dev/loop42' doesn't seem to have a valid NTFS. 
Maybe the wrong device is used? Or the whole disk instead of a 
partition (e.g. /dev/sda, not /dev/sda1)? Or the other way around? 
```

The integer number 1129531732 may be represented by the hexadecimal value 0x43534954. When rearranged in little endian format, we get the hexadecimal bytes 0x54 0x49 0x53 0x43, which happened to be ASCII for "TISC"!

On a hunch, I zero'ed out the 8 "corrupted" bytes in a copy of that disk image ("PATIENT0.mod") and tried to mount the disk image again:
```bash
$ sudo mount -t ntfs -o ro PATIENT0.mod /mnt/patient0/
$
```
> Postscript: While I had zero'ed out all the "corrupted" bytes at offset 0x20 through 0x27, and managed to mount the disk image, in reality the correct  values should have been `00 00 00 00 80 00 80 00` (hexadecimal), taking reference from a duplicate of the MBR at the end of the disk image, starting at  offset 0x5FFE00. The original uncorrupted bytes are at offset 0x5FFE20 through 0x5FFE27.

<br>

The disk image mounted successfully and it was possible to list the contents at the mount point:
```bash
$ ls -l /mnt/patient0/
total 8
-rwxrwxrwx 1 root root 6049 Aug 20 00:40 message.png
```

This **message.png** displays:
![image](https://user-images.githubusercontent.com/82754379/188159930-bc96bbd5-7126-4f73-a42a-215300d280d6.png)

The text represents a Base32-encoded string which decoded into:
```
2.Thirsty for the flag? Go find the stream.
```

The clue mentioned "stream" and it occurred to me that the NTFS disk image may contain Alternate Data Streams (ADS).

We can look for listing of files (including ADS) in an NTFS disk image with the **fls** program:
```
$ fls PATIENT0.mod

r/r 4-128-1: $AttrDef 
r/r 8-128-2: $BadClus 
r/r 8-128-1: $BadClus:$Bad 
r/r 6-128-1: $Bitmap 
r/r 7-128-1: $Boot 
d/d 11-144-2: $Extend 
r/r 2-128-1: $LogFile 
r/r 0-128-1: $MFT 
r/r 1-128-1: $MFTMirr 
r/r 9-128-2: $Secure:$SDS 
r/r 9-144-7: $Secure:$SDH 
r/r 9-144-4: $Secure:$SII 
r/r 10-128-1: $UpCase 
r/r 10-128-2: $UpCase:$Info 
r/r 3-128-3: $Volume 
r/r 31-128-1: message.png 
r/r 31-128-3: message.png:$RAND 
-/r * 32-128-1: broken.pdf 
V/V 256: $OrphanFiles 
```

There was indeed an Alternate Data Stream behind **message.png**, which could be extracted with the **icat** program:
```
$ icat PATIENT0.mod 31-128-3 > volume.bin
$ ls -l volume.bin
```

The extracted file was 2,097,401 bytes in length.

Viewing it in Ghex revealed the following at the beginning of the file:
![image](https://user-images.githubusercontent.com/82754379/188261779-9314f3f0-7d82-4bc8-8d8b-293f1282fabb.png)

The sentence read:
```
3. Are these True random bytes for Cryptology?
```

This extracted file **volume.bin** could be a VeraCrypt volume.
I tried to "mount" it in VeraCrypt with the password `f76635ab` (flag from Part 1) but it did not work.

I modifed **volume.bin** to get rid of the bytes that made up the sentence (the clue) at the beginning of the file.
I managed to eventually mount the modified **volume.bin** in VeraCrypt with the password `f76635ab`, with the option "TrueCrypt mode" checked.

The VeraCrypt volume that was mounted had a single file **outer.jpg** :
![outer](https://user-images.githubusercontent.com/82754379/188261374-3a16e39a-99ff-4854-ba20-7de9239751b2.jpg)

The end of **volume.bin** had the following bytes:
![image](https://user-images.githubusercontent.com/82754379/188261817-2c20884a-6919-4ae6-81d2-76497ebd4f9e.png)

The clue read:
```
If you need a password, the original reading of the BPB was actually Checked and ReChecked 32 times!
```

The unusual capitalisation in the phrase "Checked and ReChecked" and the numeral 32 suggested the clue was "CRC32".

It so happens that a CRC32 checksum is 4 bytes in length. From the clue in **outer.jpg** and Hint 4, I assume we needed to find a 9-letter English word which produces the CRC32 checksum value of `F7 66 35 AB` (in hexadecimal). That word would be the password for the hidden VeraCrypt volume in **volume.bin**.

I wrote a small Python script to take in words from *stdin* and print out the word and its corresponding CRC32 checksum only if the checksum matches what we are looking for (0xf76635ab):

**crc32.py**
```python
import sys
import zlib

def calculate_crc(s):
    return hex(zlib.crc32(s) & 0xffffffff)

def main():
    for line in sys.stdin:
        input = bytes(line.rstrip(), 'utf-8')
        crc32 = calculate_crc(input)
        if (crc32 == '0xf76635ab'):
            print("{}: {}".format(input, crc32))

if __name__=="__main__" :
    main()
```

Initially I tried 9-character long words from word lists like the infamous **rockyou.txt**. Nothing worked. <br>
Also, brute-forcing with a 9-character lowercase word would take too long, so it's not the correct approach here.

Eventually I realised that **outer.jpg** was actually very specific about the nature of the word we are looking for. 
Not only was it 9 characters, it started with the letter 'c' and ended with the letter 'n', as in the word "collision".
*However*, just like many of the words in the clue were written in Leet, this suggested the word was indeed "collision" but some of the characters (notably 'o', 'l' and 's') may have to be replaced by their Leet equivalents (i.e. '0', '1' and '5' respectively).

I decided to use the word-generating program **crunch** to generate the necessary words to try:
```
$ crunch 9 9 cilnos015 | egrep "^c.......n" > collision_words.txt 

$ wc ???l collision_words.txt 
4782969 collision_words.txt 

$ cat collision_words.txt | ./crc32.py 
*****  b'c01lis1on': 0xf76635ab
```

The password to the hidden VeraCrypt volume was `c01lis1on`.

<hr height="2">

We are now on the last stretch of this Level!

Selecting the file **volume.bin** again, this time using the password `c01lis1on`, I mounted the hidden volume with VeraCrypt ("TrueCrypt mode" is checked).

The hidden volume revealed a **flag.ppsm** file, which when opened, displayed the following image and played some background music:
![Screenshot from 2022-09-03 23-22-48](https://user-images.githubusercontent.com/82754379/188277914-549ee345-9c78-400c-8f5c-600ec7ff8df4.png)

The clue read:
```
TISC{md5 hash of sound clip}
```

Ah, we are so close to the flag!

The **flag.ppsm** file is actually a ZIP container, so it was possible to list the files in it using **unzip** and then selectively extract only the audio file:
```
$ unzip -l flag.ppsm

Archive:  flag.ppsm
  Length      Date    Time    Name
---------  ---------- -----   ----
     3273  1980-01-01 00:00   [Content_Types].xml
      738  1980-01-01 00:00   _rels/.rels
      838  1980-01-01 00:00   ppt/slides/_rels/slide1.xml.rels
     3653  1980-01-01 00:00   ppt/slides/slide1.xml
     3380  1980-01-01 00:00   ppt/presentation.xml
      976  1980-01-01 00:00   ppt/_rels/presentation.xml.rels
    12846  1980-01-01 00:00   ppt/slideMasters/slideMaster1.xml
      311  1980-01-01 00:00   ppt/slideLayouts/_rels/slideLayout2.xml.rels
      311  1980-01-01 00:00   ppt/slideLayouts/_rels/slideLayout1.xml.rels
     3170  1980-01-01 00:00   ppt/slideLayouts/slideLayout11.xml
     1991  1980-01-01 00:00   ppt/slideMasters/_rels/slideMaster1.xml.rels
     3648  1980-01-01 00:00   ppt/slideLayouts/slideLayout1.xml
     2891  1980-01-01 00:00   ppt/slideLayouts/slideLayout2.xml
     4384  1980-01-01 00:00   ppt/slideLayouts/slideLayout3.xml
     3756  1980-01-01 00:00   ppt/slideLayouts/slideLayout4.xml
     6285  1980-01-01 00:00   ppt/slideLayouts/slideLayout5.xml
     2223  1980-01-01 00:00   ppt/slideLayouts/slideLayout6.xml
     1897  1980-01-01 00:00   ppt/slideLayouts/slideLayout7.xml
     4704  1980-01-01 00:00   ppt/slideLayouts/slideLayout8.xml
     4623  1980-01-01 00:00   ppt/slideLayouts/slideLayout9.xml
     2946  1980-01-01 00:00   ppt/slideLayouts/slideLayout10.xml
      311  1980-01-01 00:00   ppt/slideLayouts/_rels/slideLayout3.xml.rels
      311  1980-01-01 00:00   ppt/slideLayouts/_rels/slideLayout4.xml.rels
      311  1980-01-01 00:00   ppt/slideLayouts/_rels/slideLayout5.xml.rels
      311  1980-01-01 00:00   ppt/slideLayouts/_rels/slideLayout6.xml.rels
      311  1980-01-01 00:00   ppt/slideLayouts/_rels/slideLayout7.xml.rels
      311  1980-01-01 00:00   ppt/slideLayouts/_rels/slideLayout8.xml.rels
      311  1980-01-01 00:00   ppt/slideLayouts/_rels/slideLayout9.xml.rels
      311  1980-01-01 00:00   ppt/slideLayouts/_rels/slideLayout10.xml.rels
      311  1980-01-01 00:00   ppt/slideLayouts/_rels/slideLayout11.xml.rels
    16169  1980-01-01 00:00   ppt/media/image1.png
   147982  1980-01-01 00:00   ppt/media/image2.jpg
    11197  1980-01-01 00:00   docProps/thumbnail.jpeg
     6807  1980-01-01 00:00   ppt/theme/theme1.xml
      816  1980-01-01 00:00   ppt/presProps.xml
      182  1980-01-01 00:00   ppt/tableStyles.xml
      810  1980-01-01 00:00   ppt/viewProps.xml
     1299  1980-01-01 00:00   docProps/app.xml
   961443  1980-01-01 00:00   ppt/media/media1.mp3
      644  1980-01-01 00:00   docProps/core.xml
---------                     -------
  1218992                     40 files
  ```
  ```
  $ unzip flag.ppsm ppt/media/media1.mp3
  ```
  
  Finally we generate the MD5 hash of that media file:
  ```
  $ md5sum media1.mp3
  
  f9fc54d767edc937fc24f7827bf91cfe  media1.mp3
  ```
  
  ???? **Level 3 Part 2 flag: `TISC{f9fc54d767edc937fc24f7827bf91cfe}`**
