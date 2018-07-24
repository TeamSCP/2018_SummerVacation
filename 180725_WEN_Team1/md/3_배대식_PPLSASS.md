
# :speech_balloon: PPL + LSASS

해당 기술은 Windows Internals의 저자인 `Alex Ionescu`의 글을 참조 하였습니다.<br>
원문 글을 보고 싶은 분들은 <a href="http://www.alex-ionescu.com/?p=97">링크</a>를 클릭 해 주세요 :)

## :green_book: ReadMe
  - LSASS
  - PPL
  - PPL Bypass
  
## :purple_heart: LSASS(Local Security Authority)

`lsass` 프로세스는 `Windows 2000`부터 윈도우 보안 모델에 적용된 기술입니다.<br>
Windows2000, Windows XP, ···, Windows 10에서 lsass 프로세스를 찾아 볼 수 있습니다.<br>
`lsass`는 윈도우에서 런타임 상태와 모든 로그인 작업을 담당합니다.<br>

> 그렇다면 왜 lsass는 프로세스의 핸들을 가지고 있는가?

프로그램을 실행하기전 윈도우 비스타부터 `관리자 권한으로 실행` 이라는 항목이 있다.<br>
`lsass`는 로컬 로그인의 권한을 확인하는 프로세스이므로 프로세스의 권한에 관련된 작업을 다루고 있기 때문에, <br>
실행 되고 있는 프로세스의 핸들과 실행 될 프로세스의 핸들을 가지고 있다고 유추하고 있습니다.

## :blue_heart: PPL(Protected Process Light)

Windows 8.1 부터 적용 되었습니다.

- Windows7 64bit Dll Injection
<img src="https://user-images.githubusercontent.com/40850499/42859182-6858e3b8-8a8d-11e8-9a8c-105a4f39ec9f.PNG"/>
- Windows10 64bit Dll Injection
<img src="https://user-images.githubusercontent.com/40850499/42859182-6858e3b8-8a8d-11e8-9a8c-105a4f39ec9f.PNG"/>

