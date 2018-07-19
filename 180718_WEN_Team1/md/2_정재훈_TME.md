# TME:Too Much Exploit

------

[TOC]

------

주제가 TME다. ftz 11번을 쓸데없이 많이 풀어볼 것이다. 1번부터 4번풀이는 bof를 이용한 공격이고 5번 풀이는 fsb취약점을 이용한 공격이다. 아 생각해보니 선수지식이 필요하다. 다들 [여기](https://bpsecblog.wordpress.com/memory_protect_linux/)서 메모리 보호기법부터 공부하고 와야한다. 아 또하나 필요하다. PLT랑 GOT다. 그건 [여기](https://bpsecblog.wordpress.com/about_got_plt/)서 공부할 수 있다.

## 1. Egg Shell

에그쉘이라는 기법이다. 에그쉘이란 쉘코드를 환경 변수에 넣어 입력 함수가 쉘코드의 크기 보다 작은 값을 입력받게 되면 그 쉘코드는 사용할 수 없게 될것이다. 따라서 환경변수에 넣게 되면, 4byte(환경변수의 주소)만으로 쉘코드를 실행시킬 수 있다.[^1] 따라서 페이로드를 작성하면

```bash
[level11@ftz level11]$ ./attackme $(python -c 'print [buf(256)] + [dummy(12)] + [SFP(4)] + [SHELLCODE의 환경변수 주소]')
```

로 입력하게 되면 쉘이 솩 따진다. 만약에 환경변수를 설정할 수 없는 환경이라면 이 방법은 불가능 하다.

## 2. NOP Sled

썰매를 타고 아래로 솨아아아아아악 미끄러지듯이 NOP을 타고 솨아아아아악 미끄러져서 쉘코드를 실행시키는 방법이다. NOP은 0x90으로 어셈딴에서 0x90을 만나게되면 다음을 실행시킨다. 따라서 RET를 NOP이 깔려있는 수많은 공간 중 한 부분으로 덮어씌우게 되면 nop을 쫘아아아아악 타고 가서 쉘코드를 실행시키게 된다. 이버네도 페이로드를 작성해보자

```bash
[level11@ftz level11]$ ./attackme $(python -c 'print [buf(256)] + [dummy(12)] + [SFP(4)] + [NOP주소 중 하나] + [NOP(100000)] + [SHELLCODE]')
```

이 방법은 입력값의 크기의 제한이 있다면 불가능한 방법이 될 수 있다. 하지만 ASLR을 우회하기위해 NOP Sled는 좋은 기법 중 하나라고 생각한다.

## 3. RTL: Return to Library

rtl은 DEP/NX를 우회하기 위한 기법이다.

바이너리에 포함 된 라이브러리를 이용해 system함수의 주소를 구해서 RET을 system 함수의 주소로 변조하고 `system() + AAAA(system return값;무시해도 노상관) + "/bin/sh"`와 같은 방식으로 공격할 수 있다. 

먼저 system 함수의 주소는 `ldd`명령어로 바이너리의 포함된 라이브러리를 확인한다. 그리고 gdb를 이용해 system함수의 주소를 구하고 objdump -s 명령어를 이용해 라이브러리 내의 system함수 주소를 찾아 차이를 구하면 그게 라이브러리의 base address가 된다. 또, objdump -s 명령어를 통해 "/bin/sh"의 주소를 찾고 base address와 더하면 그게 "/bin/sh"의 실제 주소값이 된다. 이 모든 주소값을 이용해 페이로드를 작성해보자

```bash
[level11@ftz level11]$ ./attackme $(python -c 'print [buf(256)] + [dummy(12)] + [SFP(4)] + [system() 주소] + [dummy(4)] + ["/bin/sh"의 주소])
```

## 4. Chaining RTL

rtl을 연속적으로 일으켜 메모리 보호기법을 우회하는 방법이다. 페이로드를 먼저 보고 설명하는게 빠를 것 같다.

```
[level11@ftz level11]$ ./attackme $(python -c 'print [버퍼에서 ret까지의 거리]
+ [strcpy()시작] + [PPR주소] + [BSS주소+0] + [/문자주소] 
+ [strcpy()시작] + [PPR주소] + [BSS주소+1] + [b문자주소] 
+ [strcpy()시작] + [PPR주소] + [BSS주소+2] + [i문자주소] 
+ [strcpy()시작] + [PPR주소] + [BSS주소+3] + [n문자주소] 
+ [strcpy()시작] + [PPR주소] + [BSS주소+4] + [/문자주소] 
+ [strcpy()시작] + [PPR주소] + [BSS주소+5] + [s문자주소] 
+ [strcpy()시작] + [PPR주소] + [BSS주소+6] + [h문자주소] 
+ [strcpy()시작] + [PPR주소] + [BSS주소+7] + [Null문자주소] 
+ [system()시작] + [빈공간4Byte] + [BSS주소+0]')
```

RET를 strcpy()의 주소로 덮고, PPR가젯 (pop pop ret ; 함수 실행 후 함수의 인자 값 개수대로 pop이 일어난 후 끝날 때 ret을 한다)을 이용해 bss(쓸 수 있고 주소가 변하지 않는 영역)에 "/", "b", ... , "NULL"을 입력하고 마지막에 system함수를 호출하고 인자 값으로 bss주소를 호출 하면서 페이로드를 완성한다.

즉, 실행 해도 변하지 않는 메모리상의 주소에, 어딘가에 존재하는 "/bin/sh"의 각각의 문자들을 조합해 하나로 넣고 그것을 system함수의 인자로 넘겨 실행을 시킨다. 결론적으로 rtl과같이 system("/bin/sh")이 실행되면서 쉘이 따진다.

## 5. Format String Bug Attack

대망의 fsb다. 원래 Double staged FSB Attack 을 공부해 pwnable.kr 의 fsb 문제와 plaid CTF 2015의 EBP를 풀어보려 했으나 fsb조차 쉽지 않았다. **FSB**(Format String Bug)란 C의 기본적인 함수 중 printf에는 다양한 사용법이 있다. 그 중에 한 가지는 변수를 **하나의 문자열만 입력**해 출력하는 방법인데, 이때 예상하지 못한 버그가 발생하는 경우가 있다. 

### 5.1 Format String

우선 포맷스트링 이란, C언어를 배운사람이라면 모를 수 없는  %d %c %f %s %x %hn %n 과 같은것이다. 사이에 숫자를 넣어 그 크기만큼의 문자를 출력하여 출력결과를 이이이이이쁘게 정렬할 수 도 있는 아아아아아주 좋은 것이다.

### 5.2 %n

많은 포맷스트링이 있다. 하지만 본 적 없는 것도 있는 것 같다. 바로바로 **`%n`**이놈이다. 얘는 **자기가 읽어들인 변수가 지시하는 주소에 이번** **printf함수에서** **지금까지 출력시킨 문자의 수를 입력한다**. 쉽게 1234%n 이면 4를 입력시킨다. 어디에? 앞에있는 것에~

printf 를 간단히 사용해보자 `printf("%d", 123);`하면 123이 출력되는건 C언어를 오늘 배운사람도 알 수 있다. 이런 것과 같이 `printf(buf);`와 같이 fsb 취약점이 발생하는 코드에서 buf에 특정 주소 값을 넣고, 1234%n을 넣으면, 해당 주소에 4가 들어간다고 보면 된다.

###  5.3 이크스푸로이또

시나리오를 먼저 생각 해보자. 특정 주소 값에 원하는 값을 넣을 수 있다....... 그럼 여태껏 bof에서 해온 짓을 생각해 볼 수 있다. 계속 해서 해온 짓은 바로 RET을 변조해서 쉘코드를 집어넣은 것이다. fsb에서도 원하는 주소(RET)에 원하는 값(shellcode환경변수의 주소)을 넣어보자.

```bash
[level11@ftz level11]$ ./attackme $(python -c 'print [dtor의앞주소] + [4Byte공간] + [dtor의뒷주소] + %문자*2개 + %[쉘코드뒷주소만큼의문자수]c + %n + %[쉘코드앞주소만큼의문자수]c + %n')
```





다음에 FSB를 더 자세히 가져오겠습니다.





------

[^1]: ftz 모든 문제를 Eggshell을 이용해 풀 수 있는 건 안비밀~