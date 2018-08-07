# :speech_balloon: Ultimate Handle Hijacking

해당 기술은 UC 포럼의 `harakrinox`의 글을 참조 하였습니다.<br>
원문 글을 보고 싶은 분들은 <a href="https://www.unknowncheats.me/forum/anti-cheat-bypass/261176-silentjack-ultimate-handle-hijacking-user-mode-multi-ac-bypass-eac-tested.html">링크</a>를 클릭 해 주세요 :)

## :green_book: ReadMe

기존의 `Handle Hijacking`을 보안 모듈이 탐지 할 수 있는 요소(벡터)들을 생각하여 보완된 기술 입니다.<br>
탐지 당할 수 있는 벡터를 harakrinox는 다음과 같이 4개의 요소를 생각 했습니다.<br>

```
  - 새로운 핸들을 사용하지 않습니다.
  - 새로운 모듈을 적재하지 않습니다.
  - 새로운 스레드를 생성하지 않습니다.
  - 새로운 실행 가능한 메모리 페이지를 생성하지 않습니다.
```

위의 4가지의 개념을 이해하기 위해선<br>

```
  - SharedMemory, SpinLock을 이용한 프로세스간 통신 방법.
  - 프로세스의 영향을 미치는 스레드와 스레드의 EIP를 바꾸는 방법.
  - 메모리 페이지를 검색 할 수 있는 방법.
  - 스레드를 검색 할 수 있는 방법.
  - _HANDLE_TABLE을 복사 할 수 있는 방법.
```
에 대해 이해하여야 합니다.<br>
마지막으로 위의 내용에 대해서 구현을 하기 위해 필요한 주된 API 목록은 다음과 같습니다.<br>

```
  - CreateFileMapping, OpenFileMapping, MapViewOfFile
  - VirtualQueryEx
  - GetThreadContext, SetThreadContext
```

## :: So what different?

기존의 핸들 하이재킹 구상 이미지를 한 번 보고 가 봅시다.

1. 클라이언트 프로그램에서 파이프를 생성하고 서버의 연결을 기다린다.
2. 타겟 프로세스 핸들을 가지고 있는 lsass, csrss에 파이프 서버 DLL을 주입한다.
3. 클라이언트 프로그램에서 ReadProcessMemory를 요청
4. 서버에서 값을 대신 읽어 준 뒤 전송.

변경된 부분

1. 클라이언트 프로그램에서 Read/Write/Execute 권한을 가진 메모리 섹션을 찾는다.
2. 클라이언트에서 CreateFileMapping, MapViewOfFile을 호출해 Read/Write가 가능한 메모리 섹션을 생성 해 준 뒤, 핸들을 닫는다.
3. R/W 권한을 가진 메모리 섹션에 OpenFileMapping, MapViewOfFile을 호출해 메모리를 공유 가능한 영역으로 만들어 준다.
4. R/W 권한을 가진 메모리 섹션에 스핀락의 상태, API들의 바뀔 수 있는 파라미터들의 정보들을 설정 해 둔다.
5. R/W/E 권한을 가진 메모리 섹션에는 스핀락, Read, Write 명령을 실행 시킬 수 잇는 쉘코드를 설정 해 둔다.
6. lsass 프로세스내 스레드 중 멈추거나 컨텍스트를 변경해도 문제 없는 스레드를 찾는다.
7. 스레드를 멈춘 뒤, Context 구조체내에 EIP 값을 수정하여 쉘코드로 이동하게 설정 해준다.

## :: Step by Step \- Create handless share memory

윈도우에서 API를 통해 공유메모리를 생성 하는 방법은 다음과 같습니다.<br>
두가지의 경우가 있는데, 첫 번째는 공유메모리를 생성 하는 방법이고 두 번째는 만들어진 공유메모리를 사용하는 방법입니다.<br>

\# 방법1. 공유메모리를 생성하고 닫는 과정<br>
```
1. CreateFileMapping
2. MapViewOfFile
3. UnMapViewOfFile
4. CloseHandle
```
\# 방법2. 만들어진 공유메모리를 사용 하는 방법<br>
```
1. OpenFileMapping
2. MapViewOfFile
3. CloseHandle
```
MSDN을 읽어보시면 Remark에서 CloseHandle로 핸들을 닫아 주어도 공유메모리는 그대로 사용 될 수 있음을 명시합니다.<br>
그러면 핸들은 어디에서 사용 될까요? 핸들은 만들어진 공유메모리 섹션을 연결 할 때 사용됩니다. OpenFileMapping에서 사용되지요.<br>
기존에 파이프를 이용한 IPC는 핸들을 계속 참조하여야 했습니다. lsass 프로세스는 파이프를 사용하지 않음에 불구하고 말이지요.<br>
이러한 상황은 보안 솔루션이 충분히 탐지 가능한 영역입니다.<br>
그러기 때문에 IPC에서 사용될 메모리 영역만 남겨두고, 핸들은 닫아 버리는 것입니다. 어차피 MapViewOfFile 리턴 값으로 공유메모리의 주소가 넘어 오기 때문이지요.<br>

## :: Step by Step \- Windows Authority

## :: Step by Step \- DuplicateHandle

## :: Step by Step \- Searching R/W/E Section

## :: Step by Step \- Finding Thread

## :: Step by Step \- Execute Shell-code by Thread context->EIP
