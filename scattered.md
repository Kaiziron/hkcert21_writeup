# Scattered 漫漫碎 (100 point, 16 solves, ★☆☆☆☆)

Solved by: Kaiziron

### Description :
```
Looks like the binary contains no flag.....

Can you help us to find the flag? There is only weird strings inside....

Attachments
scattered_605e04699fe3f83e375fc02c4ba09fe2.zip
```
---
### Solution :

Inside the zip, there is a ELF binary named `string`
```
file ./strings 
./strings: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, not stripped
```

It will print some strings when it is executed
```
./strings
Hello world!
Seems putting flag in binary will leak flag easily...
I heard string obfuscation can prevent that :thinking:
Now I obfuscate all the strings, find me if you can!
```

I tried to run strings on this binary to see if the flag can be found, but failed

```
__gmon_start__
_ZN2ay15obfuscated_dataILy53ELy6875107067980908869EE7decryptEv
_ZN2ay15obfuscated_dataILy13ELy17277989058917004229EEC2ERKNS_10obfuscatorILy13ELy17277989058917004229EEE
_ZNSt8ios_base4InitD1Ev@@GLIBCXX_3.4
_ZN2ay15obfuscated_dataILy53ELy6875107067980908869EEcvPcEv
```

However many lines like this is found, so I think the flag might be obfuscated

By running strace on the binary `strace ./strings` , I found that its using the write function to print out the strings

```
write(1, "Hello world!\n", 13Hello world!
)          = 13
write(1, "Seems putting flag in binary wil"..., 54Seems putting flag in binary will leak flag easily...
) = 54
write(1, "I heard string obfuscation can p"..., 55I heard string obfuscation can prevent that :thinking:
) = 55
write(1, "Now I obfuscate all the strings,"..., 53Now I obfuscate all the strings, find me if you can!
) = 53
```

So I just use gdb-peda and set breakpoint on the write syscall
```
gdb-peda$ b write
Breakpoint 1 at 0x7ffff7ce8f20: write. (3 locations)
```

When it reach the breakpoint, I search for the flag, and part of the flag is found

```
gdb-peda$ find hkcert21{
Searching for 'hkcert21{' in: None ranges
Found 1 results, display max 1 items:
strings : 0x6031b0 ("hkcert21{str_0bfU5c4t1oN_x0r_X0r")
```
Then I just view the memory near the first half of the flag, then the second half is found

```
gdb-peda$ x/50s 0x6031b0
0x6031b0 <_ZZNK3$_0clEvE15obfuscated_data>:	"hkcert21{str_0bfU5c4t1oN_x0r_X0r"
0x6031d1 <_ZZNK3$_0clEvE15obfuscated_data+33>:	""
0x6031d2:	""
0x6031d3:	""
0x6031d4:	""
0x6031d5:	""
0x6031d6:	""
0x6031d7:	""
0x6031d8 <_ZGVZNK3$_0clEvE15obfuscated_data>:	"\001"
0x6031da <_ZGVZNK3$_0clEvE15obfuscated_data+2>:	""
0x6031db <_ZGVZNK3$_0clEvE15obfuscated_data+3>:	""
0x6031dc <_ZGVZNK3$_0clEvE15obfuscated_data+4>:	""
0x6031dd <_ZGVZNK3$_0clEvE15obfuscated_data+5>:	""
0x6031de <_ZGVZNK3$_0clEvE15obfuscated_data+6>:	""
0x6031df <_ZGVZNK3$_0clEvE15obfuscated_data+7>:	""
0x6031e0 <_ZZNK3$_1clEvE15obfuscated_data>:	"_w1tH_d3t3rm1n1st1c_h45h3s}"
0x6031fc <_ZZNK3$_1clEvE15obfuscated_data+28>:	""
```

### Flag :
`hkcert21{str_0bfU5c4t1oN_x0r_X0r_w1tH_d3t3rm1n1st1c_h45h3s}`


