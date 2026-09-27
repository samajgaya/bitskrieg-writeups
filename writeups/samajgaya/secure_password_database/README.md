# Secure Password Database - Reverse Engineering (medium)

## Overview
Attached is an executable ELF file as indicated by `file(1)`
```sh
$ file system.out
system.out: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV),
dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2,
BuildID[sha1]=63224e5a94fa31cb071c82105d8a70ffc806ac0b, for GNU/Linux 3.2.0,
not stripped
```

Executing it:
```sh
$ ./system.out                        
Please set a password for your account:
aa
How many bytes in length is your password?
42
You entered: 42
Your successfully stored password:
97 97 10 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 
0 0 0 0 0 0 0 0 
Enter your hash to access your account!
99
```

## Solving
Seems like we have an arbitrary read into the password buffer, it blindly prints
the number of bytes we provide as password length.

The ELF file isn't stripped, so de-compiling should be easy!
Here's a cleaned up decompilation:
```c
undefined8 main(void) {
  uint password_length;
  char *ret;
  undefined8 res;
  int i;
  char *endptr;
  ulong j;
  char *mem;
  size_t hashlen;
  ulong hash;
  ulong secret_hash;
  FILE *f;
  byte secret [13];
  char password_length_s [32];
  char password [64];
  char flag [104];
  long local_10;
  
  mem = calloc(90,1);
  for (j = 0; j < 13; j = j + 1) {
    mem[j + 60] = obf_bytes[j] ^ 0xaa;
  }
  puts("Please set a password for your account:");
  ret = fgets(password,50,stdin);
  if (ret != (char *)0x0) {
    strcpy(mem,password);
    puts("How many bytes in length is your password?");
    ret = fgets(password_length_s,20,stdin);
    if (ret != (char *)0x0) {
      password_length = atoi(password_length_s);
      printf("You entered: %d\n",(ulong)password_length);
      puts("Your successfully stored password:");
      for (i = 0; (i <= (int)password_length && (i < 90)); i = i + 1) {
        printf("%d ",(ulong)(uint)(int)mem[i]);
      }
      putchar(10);
    }
  }
  puts("Enter your hash to access your account!");
  ret = fgets(password,50,stdin);
  if (ret != (char *)0x0) {
    hashlen = strlen(password);
    if ((hashlen != 0) && (password_length_s[hashlen + 0x1f] == '\n')) {
      password_length_s[hashlen + 0x1f] = '\0';
    }
    hash = strtoul(password,&endptr,10);
    if (endptr == password) {
      printf("No digits were found");
                    /* WARNING: Subroutine does not return */
      __assert_fail("1 == 0","heartbleed.c",0x45,"main");
    }
    secret_hash = make_secret(secret);
    if (secret_hash == hash) {
      f = fopen("flag.txt","r");
      if (f == (FILE *)0x0) {
        perror("Could not open flag.txt");
        res = 1;
        goto bad;
      }
      ret = fgets(flag,100,f);
      if (ret == (char *)0x0) {
        puts("Failed to read the flag");
      }
      else {
        printf("%s",flag);
      }
      fclose(f);
    }
  }
  free(mem);
  res = 0;
bad:
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return res;
}
```
All we have to do is provide the right hash when prompted and the flag is ours!

Looking into `make_secret()`

```c
long make_secret(byte *secret)

{
  long h;
  long i;
  
  for (i = 0; obf_bytes[i] != 0; i = i + 1) {
    secret[i] = obf_bytes[i] ^ 0xaa;
  }
  secret[12] = 0;
  h = hash(secret);
  return h;
}
```

and `hash()`

```c
long hash(byte *secret) {
  byte *p;
  long hash;
  
  hash = 5381;
  p = secret;
  while( true ) {
    if (*p == 0) break;
    hash = (long)(int)(uint)*p + hash * 33;
    p = p + 1;
  }
  return hash;
}
```

`hash()` seems to be a simple djb2 hash function, while make secret just writes
to secret, hashes and returns the result.

Since it is independent of the password input and password length we provide,
this can be solved statically. However to make things easy, I open it in gdb
breaking right before the hash is returned in `make_secret()`, at that point
`%rax` should contain the hash of our secret.
```sh
$ gdb ./system.out
(gdb) b make_secret 
Breakpoint 2 at 0x136a
(gdb) c
❌️ The program is not being run.
(gdb) r
...
Please set a password for your account:
a
How many bytes in length is your password?
b
You entered: 0
Your successfully stored password:
97 
Enter your hash to access your account!
1

Breakpoint 2, 0x000055555555536a in make_secret ()
(gdb) disas
Dump of assembler code for function make_secret:
   0x000055555555535e <+0>:     endbr64
   0x0000555555555362 <+4>:     push   rbp
   0x0000555555555363 <+5>:     mov    rbp,rsp
   0x0000555555555366 <+8>:     sub    rsp,0x18
=> 0x000055555555536a <+12>:    mov    QWORD PTR [rbp-0x18],rdi
   0x000055555555536e <+16>:    mov    QWORD PTR [rbp-0x8],0x0
   0x0000555555555376 <+24>:    jmp    0x5555555553a2 <make_secret+68>
   0x0000555555555378 <+26>:    lea    rdx,[rip+0xc89]        # 0x5555555
56008 <obf_bytes>
   0x000055555555537f <+33>:    mov    rax,QWORD PTR [rbp-0x8]
   0x0000555555555383 <+37>:    add    rax,rdx
   0x0000555555555386 <+40>:    movzx  eax,BYTE PTR [rax]
   0x0000555555555389 <+43>:    xor    eax,0xffffffaa
   0x000055555555538c <+46>:    mov    ecx,eax
   0x000055555555538e <+48>:    mov    rdx,QWORD PTR [rbp-0x18]
   0x0000555555555392 <+52>:    mov    rax,QWORD PTR [rbp-0x8]
   0x0000555555555396 <+56>:    add    rax,rdx
   0x0000555555555399 <+59>:    mov    edx,ecx
   0x000055555555539b <+61>:    mov    BYTE PTR [rax],dl
   0x000055555555539d <+63>:    add    QWORD PTR [rbp-0x8],0x1
   0x00005555555553a2 <+68>:    lea    rdx,[rip+0xc5f]        # 0x5555555
56008 <obf_bytes>
   0x00005555555553a9 <+75>:    mov    rax,QWORD PTR [rbp-0x8]
   0x00005555555553ad <+79>:    add    rax,rdx
   0x00005555555553b0 <+82>:    movzx  eax,BYTE PTR [rax]
   0x00005555555553b3 <+85>:    test   al,al
   0x00005555555553b5 <+87>:    jne    0x555555555378 <make_secret+26>
   0x00005555555553b7 <+89>:    mov    rax,QWORD PTR [rbp-0x18]
   0x00005555555553bb <+93>:    add    rax,0xc
   0x00005555555553bf <+97>:    mov    BYTE PTR [rax],0x0
   0x00005555555553c2 <+100>:   mov    rax,QWORD PTR [rbp-0x18]
   0x00005555555553c6 <+104>:   mov    rdi,rax
   0x00005555555553c9 <+107>:   call   0x555555555309 <hash>
   0x00005555555553ce <+112>:   leave
   0x00005555555553cf <+113>:   ret
End of assembler dump.
(gdb) b *make_secret+112
Breakpoint 3 at 0x5555555553ce
(gdb) c
Continuing.

Breakpoint 3, 0x00005555555553ce in make_secret ()
(gdb) info reg rax
rax            0xd3770d6251b31be2  -3209081493549540382
(gdb) 
```
Here `0xd3770d6251b31be2` is the required hash, converting to decimal,
(since that's how our hash is processed in `main()`) we have
`15237662580160011234`. Provide that as the hash, and we have the flag!

```sh
$ nc $ADDRESS $PORT

Please set a password for your account:
How many bytes in length is your password?
a
You entered: 0
Your successfully stored password:
10 
Enter your hash to access your account!
15237662580160011234               
picoCTF{....}
```
