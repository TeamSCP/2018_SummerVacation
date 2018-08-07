# :speech_balloon: Ultimate Handle Hijacking

해당 기술은 UC 포럼의 `harakrinox`의 글을 참조 하였습니다.<br>
원문 글을 보고 싶은 분들은 <a href="https://www.unknowncheats.me/forum/anti-cheat-bypass/261176-silentjack-ultimate-handle-hijacking-user-mode-multi-ac-bypass-eac-tested.html">링크</a>를 클릭 해 주세요 :)

## :green_book: ReadMe

기존의 `Handle Hijacking`을 AC(Anti Cheat)가 탐지 할 수 있는 요소(벡터)들을 생각하여 보완된 기술 입니다.<br>
Ultimate Handle Hijacking 에서는 탐지 당할 수 있는 벡터를 아래와 같이 4가지로 정의 하였습니다.<br>

```
  - 프로세스간 통신에 핸들을 사용하지 않습니다.
  - 새로운 모듈을 적재하지 않습니다.
  - 새로운 스레드를 생성하지 않습니다.
  - 새로운 실행 가능한 메모리 페이지를 생성하지 않습니다.
```

위의 4가지의 개념을 이해하기 위해선<br>

```
  - Share Memory, SpinLock을 이용한 프로세스간 통신 방법.
  - 프로세스의 영향을 미치는 스레드와 스레드의 EIP를 바꾸는 방법.
  - 메모리 페이지를 검색 할 수 있는 방법.
  - 스레드를 검색 할 수 있는 방법.
  - _HANDLE_TABLE을 복사 할 수 있는 방법.
```
에 대해 이해 하여야 합니다.<br>
마지막으로 위의 내용에 대해서 구현을 하기 위해 필요한 주된 API 목록은 다음과 같습니다.<br>

```
  - CreateFileMapping, OpenFileMapping, MapViewOfFile
  - VirtualQueryEx
  - GetThreadContext, SetThreadContext
```

## :: So what different?

기존의 Handle Hijacking 구상 방법은 아래의 이미지와 같습니다.<br>
<img src="https://user-images.githubusercontent.com/40850499/43772484-3fcf8672-9a7d-11e8-8ff9-b965a82579e2.PNG" />
클라이언트와 lsass, csrss에 삽입된 DLL과의 NamedPipe를 통해 서로 통신을 하여 실질적인 작업은 삽입된 DLL에서 해주는 방법입니다.<br>

반면에 Ultimate Handle Hijacking은 어떤가요? 생각외로 복잡한 방법이네요 :( <br>
<img src="https://user-images.githubusercontent.com/40850499/43793129-9f16660e-9ab5-11e8-975f-b179c4c40ebf.png" />

이 기법에 주된 우회 방법은 `재사용`이라는 키워드로 정리 할 수 있을 것 같습니다.<br>
첫 번째로 Lsass.exe 같은 시스템 프로세스가 이름있는 파이프를 열어서 통신 할 일이 있을까요? AC에서는 Lsass.exe 프로세스에 대해서 파이프가 생성되면 악의적인 작업을 하는 것으로 간주 할 수 있습니다.<br>
두 번째로 DLL 목록을 검사하여 코드를 실행시키는 블록이 있는지를 확인하는 작업 또한 AC에서 가능 할 것이라 예상 할 수 있습니다.<br>
세 번째로 메모리 페이지가 실행옵션이 걸린 상태로 생성 된다면 코드케이빙이 가능하기 때문에 이 또한 탐지벡터에 걸릴 수 있습니다.<br>
마지막으로, 새로운 스레드가 생성되면 실행분기가 하나 더 생긴다 라고 가정 할 수 있고 CreateRemoteThread를 이용한 DLL Injection 또한 생각 해볼 수 있을 것입니다.<br>
이제 AC를 우회하기 위한 방법을 Step by step으로 천천히 진행 해보려 합니다.<br>

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
MSDN을 읽어보시면 Remarks에서 핸들을 닫아 주어도 공유메모리는 그대로 사용 될 수 있음을 명시합니다.<br>
그러면 핸들은 어디에서 사용 될까요? 핸들은 만들어진 공유메모리 섹션을 연결 할 때 사용됩니다.<br>
기존에 파이프를 이용한 IPC는 핸들을 계속 참조하여야 했습니다. lsass 프로세스는 파이프를 사용하지 않음에 불구하고 말이지요.<br>
이러한 상황은 보안 솔루션이 충분히 탐지 가능한 영역입니다.<br>
그러기 때문에 IPC에서 사용될 메모리 영역만 남겨두고, 핸들은 닫아 버리는 것입니다. 어차피 MapViewOfFile 리턴 값으로 공유메모리의 주소가 넘어 오기 때문이지요.<br>

## :: Step by Step \- Windows Authority

## :: Step by Step \- DuplicateHandle

## :: Step by Step \- Searching R/W/E Section

## :: Step by Step \- Finding Thread

## :: Step by Step \- Execute Shell-code by Thread context->EIP
