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
* The OpenPGP key might be interesting, but I already tried extracting and analyzing it, and it did not give any results.

---

# Result
Despite trying multiple Steganographic and forensic techniques I was unable to get the flag

`steghide` hinted at hidden data but required a password, which I was unable to brute-force successfully.

