# Ret's pwn~!

##### pwnable.kr [Toddler's Bottle] fd write up (feat. pwntools)

> 작성 : f.killrra
>
> 작성일 : 2018-07-08

> 들어가기전에...
>
> 이 문서는 [pwnable.kr](http://pwnable.kr) 의 직접적인 풀이법이 있습니다. 풀이를 한 후 보시는것을 추천드립니다.

0x01. fd

[힌트 및 문제]

> Mommy! what is a file descriptor in Linux?
>
> * try to play the wargame your self but if you are ABSOLUTE beginner, follow this tutorial link:
>   https://youtu.be/971eZhMHQQw
>
> ssh fd@pwnable.kr -p2222 (pw:guest)

SSH 로 접속하여 문제를 확인해보자.

![2](C:\Users\fkill\Desktop\2.PNG)

접속하여 ls -al 명령어로 모든 파일의 목록을 확인하면 이와 같다.

fd라는 바이너리에 setuid가 걸려있기 때문에 우리는 이 파일의 취약점을 이용해 fd_pwn이라는 권한으로 권한 상승을 꿈 꿀 수 있다.

하지만 문제의 의도는 그것이 아닌것을 fd.c 파일을 통해 확인 할 수 있다.

![3](C:\Users\fkill\Desktop\3.PNG)

fd.c 파일을 열어보면 fd 바이너리의 c코드가 보인다.

바이너리 이름에서부터 알 수 있듯이 이 문제는 fd 즉 파일 디스크립터에 관한 문제이다.

코드를 분석해보면 아래와 같다.

> 1. argc 즉 인자가 파일명 빼고 하나 더 있어야한다.
> 2. fd라는 지역 변수에 atoi() 함수로 argv[1]을 받아 정수형으로 반환하고 0x1234를 뺀 뒤 저장한다.
> 3. len이라는 지역 변수에 read()의 return 값을 저장한다.
> 4. strcmp() 함수로 buf에서 LETMEWIN\n 이라는 값을 비교하고 맞다면 system() 함수를 통해 flag파일을 열어준다.

파일 디스크립터 문제인 만큼 파일 디스크립터의 개념을 잡고가자.

[file descriptor]

- 시스템으로 부터 할당 받을 파일을 대표하는 0이 아닌 정수값
- 프로세스에서 열린 파일의 목록을 관리하는 테이블의 인덱스

-> 흔히 유닉스 시스템에서는 모든 것을 파일이라고 한다.

유닉스 시스템에서는 모든 장치를 파일로 관리하는데, 내부/외부 모든 장치도 파일로 취급한다.

이 파일을 관리하는 것이 file descriptor이다.



여기서 file descriptor로 사용될 수 없는 정수값이 존재하는데 0과 1, 2이다.

각각은 예약되어있는 값으로서 아래와 같은 역할을 한다.

|   0   | **표준 입력** | **STDIN**  |
| :---: | :-----------: | :--------: |
| **1** | **표준 출력** | **STDOUT** |
| **2** | **표준 오류** | **STDERR** |

이 처럼 예약되어 있는 값이 2까지 있으므로 파일을 가리킬 때 사용하는 파일 디스크립터는 3부터 그 이상의 값으로 할당 받는다.

<br>

다시 문제 풀이로 돌아와서 파일을 가리키는 값 즉 fd를 직접 인자값을 받아서 지정해 줄 수 있는데 이 값을 0으로 해준다면 read()에서 buf 에 LETMEWIN\n 이라는 문자열을 입력하여 넣어줄 수 있을 것이다.

<br>

![4](C:\Users\fkill\Desktop\4.PNG)

0x1234 를 빼준 뒤 fd에 저장을 하기 때문에 fd에 넘길 값은 4660이다.

![캡처](C:\Users\fkill\Desktop\캡처.PNG)

이와 같이 4660을 argv[1]에 넘겨주면 바이너리는 입력을 기다리는 상태가 된다.

이제 LETMEWIN을 입력한 뒤 \n (엔터)를 입력하면 flag값을 뱉는다.

여기서 끝을 내도 되지만 공부를 위해 python 코드를 작성해줬다. 

![5](C:\Users\fkill\Desktop\5.PNG)

pwn 모듈을 이용하여 ssh()로 ssh 연결을 한뒤

argv[0] 에 fd(파일명), argv[1]에 인자값 4660을 넣을 뒤 실행 해준뒤 sendline() 함수로 LETMEWIN 문자열을 입력하고 해당 바이너리가 뱉는 값 (flag값, good job:) 을 받아왔다.

잘 작동하는지 확인해보자.

![11](C:\Users\fkill\Desktop\11.PNG)

이상 풀이를 마친다.