
# :speech_balloon: PPL + LSASS + CSRSS

해당 기술은 Windows Internals의 저자인 `Alex Ionescu`의 글을 참조 하였습니다.</br>
원문 글을 보고 싶은 분들은 <a href="http://www.alex-ionescu.com/?p=97">링크</a>를 클릭 해 주세요 :)

## :green_book: ReadMe
  - LSASS
  - CSRSS
  - PPL
  - PPL Bypass
  
## :purple_heart: LSASS(Local Security Authority)

`LSASS`프로세스는 `Windows 2000`부터 윈도우 보안 모델에 적용된 기술입니다.</br>
Windows2000, Windows XP, ···, Windows 10에서 LSASS 프로세스를 찾아 볼 수 있습니다.</br>
`LSASS`는 윈도우에서 런타임 상태와 모든 로그인 작업을 담당합니다.</br>
> 그렇다면 왜 LSASS는 프로세스의 핸들을 가지고 있는가?

프로그램을 실행하기전 윈도우 비스타부터 `관리자 권한으로 실행` 이라는 항목이 있습니다.</br>
`LSASS`는 로컬 로그인의 권한을 확인하는 프로세스이므로 프로세스의 권한에 관련된 작업을 다루고 있기 때문에, </br>
실행 되고 있는 프로세스의 핸들과 실행 될 프로세스의 핸들을 가지고 있다고 유추하고 있습니다.

## :heart: CSRSS(Client/Server Runtime Subsystem)

## :blue_heart: PPL(Protected Process Light)

같은 Dll Injector로 `LSASS.exe` 프로세스에 더미 dll 파일을 인젝션 해 보았습니다.

> Windows7 64bit Dll Injection
<img src="https://user-images.githubusercontent.com/40850499/43158769-7eabde12-8fbb-11e8-9849-49e41b6f571d.PNG"/>

> Windows10 64bit Dll Injection
<img src="https://user-images.githubusercontent.com/40850499/43158763-7b6af940-8fbb-11e8-9eda-d16c10357b20.PNG"/>

두 운영체제 모두 다 잘 되는것을 확인 할 수 있습니다.
그렇다면 Windows10에서 `_PS_PROTECTION` 구조체의 값을 수정하여 DLL Injection을 차단 해 보겠습니다.<br>
`_PS_PROTECTION` 구조체는 EPROCESS 구조체의 멤버입니다.<br>
하지만 EPROCESS 구조체는 커널모드에서 사용하는 리소스이기 때문에 커널 디버거로 데이터를 확인 해야 합니다.</br>

> Settings

1. WinDBG 설치(<a href="http://www.windbg.org/">링크</a>)
2. VMWare serial port 생성
3. bcdedit /copy {current} /d "Debug"
4. msconfig → 부팅 탭 → Debug → 고급 옵션 → 디버그 틱 → 디버그 포트 COM2
5. WinDbg → Kernel Debug(CTRL+K) → COM 탭 → Port: \\\\.\\pipe\\pipe_name → Pipe 틱 → Reconnect 틱
6. Debug로 부팅
7. Debug → Break
8. Enjoy debugging :smile:

사진과 함께 있는 자세한 설명은 <a href="http://ruinick.tistory.com/96">여기</a>를 참조하세요!

> EPROCESS -> Protection

```C
1 _PS_PROTECTION
2   +0x000 Level            : UChar
3   +0x000 Type             : Pos 0, 3 Bits
4   +0x000 Audit            : Pos 3, 1 Bit
5   +0x000 Signer           : Pos 4, 4 Bits
```
_PS_PROTECTION 구조체는 8Bit를 사용하며  Type은 아래의 규격을 </br>

```C
1 _PS_PROTECTED_TYPE
2   PsProtectedTypeNone = 0n0
3   PsProtectedTypeProtectedLight = 0n1
4   PsProtectedTypeProtected = 0n2
5   PsProtectedTypeMax = 0n3
```

_PS_PROTECTED_SIGNER
```C
1 _PS_PROTECTED_SIGNER
2   PsProtectedSignerNone = 0n0
3   PsProtectedSignerAuthenticode = 0n1
4   PsProtectedSignerCodeGen = 0n2
5   PsProtectedSignerAntimalware = 0n3
6   PsProtectedSignerLsa = 0n4
7   PsProtectedSignerWindows = 0n5
8   PsProtectedSignerWinTcb = 0n6
9   PsProtectedSignerMax = 0n7
```
