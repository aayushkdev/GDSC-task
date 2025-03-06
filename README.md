# CTF Writeup: Pack and Ship

## Challenge Overview

In this challenge, I was given a binary file named `release`. The task was to extract an obfuscated password and flag using reverse engineering techniques.

---


## 🔹 Step 1: Initial Analysis

First, I used the `strings` command to see if there was any readable text inside the binary.

```
strings release | less
```

**Findings:**

* The binary appeared to be packed with UPX.
---

## 🔹 Step 2: Unpacking the Binary

Since the binary was UPX-packed, I unpacked it.

```
upx -d release
```

Then, I ran the `strings` command again.

```
strings release | less
```

This time, I saw more useful information, including references to `obfuscated_password`, `obfuscated_flag`, and a function called `deobfuscate`.

---

## 🔹 Step 3: Disassembling the Binary

Next, I opened the binary in GDB for further analysis.

```
gdb release
```

I disassembled the `main` function to understand its logic.

```
disassemble main
```

```assembly
Dump of assembler code for function main:
    0x0000000000001284 <+0>:     push    %rbp
    0x0000000000001285 <+1>:     mov     %rsp,%rbp
    0x0000000000001288 <+4>:     sub     $0xb0,%rsp
    0x000000000000128f <+11>:    mov     %fs:0x28,%rax
    0x0000000000001298 <+20>:    mov     %rax,-0x8(%rbp)
    0x000000000000129c <+24>:    xor     %eax,%eax
    0x000000000000129e <+26>:    movq    $0x0,-0x40(%rbp)
    0x00000000000012a6 <+34>:    movq    $0x0,-0x38(%rbp)
    0x00000000000012ae <+42>:    movq    $0x0,-0x30(%rbp)
    0x00000000000012b6 <+50>:    movq    $0x0,-0x28(%rbp)
    0x00000000000012be <+58>:    movq    $0x0,-0x20(%rbp)
    0x00000000000012c6 <+66>:    movq    $0x0,-0x18(%rbp)
    0x00000000000012ce <+74>:    movl    $0x0,-0x11(%rbp)
    0x00000000000012d5 <+81>:    lea     -0xa0(%rbp),%rax
    0x00000000000012dc <+88>:    mov     $0x20,%edx
    0x00000000000012e1 <+93>:    lea     0xd58(%rip),%rcx          # 0x2040 <obfuscated_password>
    0x00000000000012e8 <+100>:   mov     %rcx,%rsi
    0x00000000000012eb <+103>:   mov     %rax,%rdi
    0x00000000000012ee <+106>:   call    0x1189 <deobfuscate> 
    0x00000000000012f3 <+111>:   lea     -0x70(%rbp),%rax
    0x00000000000012f7 <+115>:   mov     $0x23,%edx
    0x00000000000012fc <+120>:   lea     0xd5d(%rip),%rcx          # 0x2060 <obfuscated_flag>
    0x0000000000001303 <+127>:   mov     %rcx,%rsi
    0x0000000000001306 <+130>:   mov     %rax,%rdi
    0x0000000000001309 <+133>:   call    0x1189 <deobfuscate>
    0x000000000000130e <+138>:   lea     0xd6e(%rip),%rax          # 0x2083
    0x0000000000001315 <+145>:   mov     %rax,%rdi
```

**Findings in `main`:**

* The `main` function called `deobfuscate()` twice:
    * Once for `obfuscated_password`.
    * Once for `obfuscated_flag`.
* The obfuscated password was stored at memory location `0x2040`.
* The password after deobfuscation was stored at memory location `0xa0`.

---

## 🔹 Step 4: Setting a Breakpoint in `deobfuscate()`

To find the password, I had to inspect the variable where it was stored after the `deobfuscate` function executed.

I set a breakpoint at the `deobfuscate` function.

```
b deobfuscate
```

Then, I ran the program.

```
run
```

When it stopped at the breakpoint, I executed the function completely to return to the main function.

```
finish
```

Now `0xa0` address had the value of the deobfuscated password.

---

## 🔹 Step 5: Extracting the Deobfuscated Password

I inspected the memory address of `obfuscated_password` to see the deobfuscated password.

```
x/s $rbp-0xa0
```

The output was: `bt8uA1uxF350yZLuto9GmqTWJBpP2jUq`

This is the correct password

---

## 🔹 Step 7: Testing the Password and Retrieving the Flag

I ran the program directly and entered the extracted password.
```
./release
Enter the password: bt8uA1uxF350yZLuto9GmqTWJBpP2jUq
Correct password! Here is the flag: gdsc{unp4ck1n6_b1n4r135_15_n4u6h7y}
```

---

# Result:

I obtained the flag `gdsc{unp4ck1n6_b1n4r135_15_n4u6h7y}`.
