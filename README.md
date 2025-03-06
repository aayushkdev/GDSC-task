# CTF Writeup: Hidden Pixels

## Challenge Overview

I was trying to analyse and extract hidden data from `output.bmp` using various steganography and forensic techniques. Here’s a step-by-step log of everything I tried.

---

## 1. Visual Analysis

First I decided to analyze the image visually to check if any information was hidden in different color channels.
I used `Stegsolve` which allowed me to isolate different colour channels
When I isolated the blue colour channel I was able to see the full image

![b](https://github.com/user-attachments/assets/1d02874c-892f-46c7-81fe-da5ddab5e19f)

Visual inspection didnt lead to anything as the text was meaningless

---

## 2. Checking Metadata

First, I used `exiftool` to extract metadata from the image:

```
exiftool output.bmp
```

**Findings:**

* Found that the image is a Windows V5 BMP with 32-bit depth.
* Other then that there was nothing unusual here.

Additionally, when I ran:

```
file output.bmp  
```

- Initially, the file showed a negative width, which was unusual and I suspected there might be something hidden in the header.
- But after analysing the header thoroughly it didn't give me any meaningful results.
- After conversion of file using ffmpeg in `step 4`, the width displayed was shown as positive, which lead me to believe this was just a formatting issue.

---

## 3. Checking for Steganography with steghide

I ran `steghide` to check if any data is hidden:

```
steghide info output.bmp
```

**Result:**

* Got an error: the bmp file has a format that is not supported (biSize: 124).
* Since `steghide` does not support BMP V5, I had to convert it to an older format.

---

## 4. Converting BMP Version

Tried converting the BMP file using `ffmpeg`:

```
ffmpeg -i output.bmp -pix_fmt rgb24 output_v3.bmp
```

**Result:**

* This converted successfully to a Windows 3.x format, 24-bit depth BMP.
* Also the negative width issue was resolved.

---

## 5. Checking steghide Again

Tried `steghide` on `output_v3.bmp`:

```
steghide info output_v3.bmp
```

**Result:**

* This time, it asked for a password, meaning there might be hidden data inside.

---

## 6. Bruteforcing steghide

Since `steghide` needed a password, I tried brute-forcing:

```
stegcracker output_v3.bmp rockyou.txt
```

**Result:**

* Bruteforcing didn't work, so the password remains unknown.

---

## 7. Using zsteg for LSB Steganography

I ran `zsteg` to check for hidden Least Significant Bit (LSB) data:

```
zsteg output_v3.bmp
```

**Findings:**

* Found some hidden text:
  ```
  `b4,r,lsb,xy -> "DC4&vjhf"`
  `b4,g,lsb,xy -> "3WDeQTC13>F"`
  `b3,g,lsb,xy -> "u(/dFdLG"`
  `b3,rgb,lsb,xy -> "oX =@\rp%"`
  `b3,r,msb,xy -> OpenPGP Public Key`
  ```
* The presence of an OpenPGP Public Key seemed promising, so I attempted to extract and analyze it using gpg and strings but was unable to retrieve any meaningful data.
* This suggests that the detected key might be false positive or just noise in the image.

---

## 8. Searching for hidden files using binwalk

I ran binwalk to check if any embedded data or files were hidden inside the BMP 

binwalk gave different results for output.bmp (the original file) and output_v3.bmp (the file converted to bmp 3.0 format)

## Binwalk Analysis
Scanning `output.bmp`
```
$ binwalk output.bmp   

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
785639        0xBFCE7         JBOOT STAG header, image id: 12, timestamp 0x14E28F10, image size: 721693439 bytes, image JBOOT checksum: 0xA010, header JBOOT checksum: 0x1493        
```

Scanning `output_v3.bmp`
```
$ binwalk output_v3.bmp                                   

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PC bitmap, Windows 3.x format,, 1024 x 1024 x 24
141912        0x22A58         JBOOT STAG header, image id: 2, timestamp 0x384C4C18, image size: 2503822460 bytes, image JBOOT checksum: 0x3949, header JBOOT checksum: 0x469A
873229        0xD530D         JBOOT STAG header, image id: 9, timestamp 0xA7A48F2D, image size: 3220171210 bytes, image JBOOT checksum: 0xEDE5, header JBOOT checksum: 0xE7C1x
```

I tried extracting the files through `binwalk -e output.bmp` which failed, meaning it was most likely a false positive but to investigate further, I manually extracted the files using dd. After running the file command on each extracted file, most turned out to be unhelpful, except one from output_v3.bmp:

```
$ file jboot1.bin   
jboot1.bin: AmigaOS bitmap font (TFCH) "\030LL8|L=\225I9\232F8\237I@\245C<\235B;\233J>\250M<\257L=\257I<\251G;\246G;\244K=\245J=\232E9\211/$f\020\0057\010", tfc_TagCount 22860, tfc_YSize 4294950742, 9259 elements, 2nd "PE\270OD\271TH\273RD\261]P\300WK\275ZN\304YL\302WJ\276UH\273ZL\274YM\267XI\260^L\253bP\242aQ\2206*W\016\004#\013\002 %\030I[K\220wf\275rb\301tf\301ti\273ti\261wk\253sd\226I>]\030\020 \011\002\022\013\001#\036\017G>-uP@\220J:\224I9\224H:\227C7\226E9\232G8"                  
```

I analyzed the extracted files using strings, exiftool, and ent to search for any hidden data. Unfortunately, none of the results were meaningful, further confirming that these were likely false positives.

---

# Result
Despite trying multiple Steganographic and forensic techniques I was unable to get the flag

`steghide` hinted at hidden data but required a password, which I was unable to brute-force successfully.

