
# :speech_balloon: PPL + LSASS

해당 기술은 Windows Internals의 저자인 `Alex Ionescu`의 글을 참조 하였습니다.<br>
원문 글을 보고 싶은 분들은 <a href="http://www.alex-ionescu.com/?p=97">링크</a>를 클릭 해 주세요 :)

## :green_book: ReadMe
  - LSASS
  - PPL
  - PPL Bypass
  
## LSASS(Local Security Authority)

lsass.exe 프로세스는 Windows 2000부터 윈도우 보안 모델에 적용된 기술입니다.
Windows2000, Windows XP, ···, Windows 10에서 lsass.exe 프로세스를 찾아 볼 수 있습니다.
