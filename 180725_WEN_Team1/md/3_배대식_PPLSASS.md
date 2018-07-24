
# :speech_balloon: PPL + LSASS

해당 기술은 Windows Internals의 저자인 `Alex Ionescu`의 글을 참조 하였습니다.<br>
원문 글을 보고 싶은 분들은 <a href="http://www.alex-ionescu.com/?p=97">링크</a>를 클릭 해 주세요 :)

## :green_book: ReadMe
  - LSASS
  - PPL
  - PPL Bypass
  
## LSASS(Local Security Authority)

lsass.exe 프로세스는 Windows 2000부터 윈도우 보안 모델에 적용된 기술입니다.<br>
Windows2000, Windows XP, ···, Windows 10에서 lsass.exe 프로세스를 찾아 볼 수 있습니다.<br>
lsass는 윈도우에서 런타임 상태와 모든 로그인 작업을 담당합니다.<br>
작업 관리자에서 표기되는 사용자 이름에 따라 접근 할 수 있는 프로세스가 정해져 있거나, 혹은 관리자 권한 실행에 대한 권한을 다르고 있기 때문에 실행되고 있는 프로세스나 실행 될 프로세스의 핸들을 가지고 있는 것지요.
