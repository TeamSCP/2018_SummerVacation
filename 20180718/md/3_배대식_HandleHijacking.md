# :speech_balloon: Handle Hijacking

해당 기술은 UC 포럼의 `harakrinox`의 글을 참조 하였습니다.<br>
원문 글을 보고 싶은 분들은 <a href="https://www.unknowncheats.me/forum/anti-cheat-bypass/217995-poc-remote-memory-operations-using-existing-handle.html">`링크`</a>를 클릭 해 주세요 :)

## :green_book: ReadMe
  
  - 프로세스간 통신(IPC)중 Named Pipe를 사용합니다.
  - `공격 프로그램이 핸들을 구해 올 수 없다`의 가정 하에 진행 됩니다.
  - `개념증명(PoC)`에 사용 되는 코드는 대상이 둘 다 실행 파일이지만, 파이프 서버쪽을 `하이재킹 대상` 혹은 `dll injection`의 대상이라고 생각 해주세요.
  - 유저 모드에서 사용 되는 우회 기법입니다.

## :airplane: 하이재킹?

개념은 아주 간단합니다. 이미 사용되고 있는 무언가(세션, 쿠키, 핸들 ···)를 가져와 사용 하는 것입니다.<br>
<a href="https://en.wikipedia.org/wiki/Hijacking">Wiki 링크</a> 

## :heavy_check_mark: 핸들에 대해 복습

프로세스, 파일 같은 것들은 모두 커널에서 사용되는 객체입니다.<br>
이러한 객체를 유저모드에서 접근 하기 위해서 객체에 대한 인스턴스를 만들어주고,<br>
인스턴스에 대해 접근 할 수 있는 방법이 핸들인 것입니다.

## :question: 핸들을 빌려 쓰는 것이 왜 우회인가요?
오래 전(중학교~대학교 1학년)에는 공격 대상이 타겟 프로세스인 경우가 많았습니다.<br>
RPM/WPM으로 메모리를 읽기/쓰기를 하던, Dll을 인젝션을 하여 코드를 수정하던, 그 프로세스에 대한 핸들을 직접 가져 오는 것이였습니다.<br>
그렇다면 Anti-Cheat 개발자 분들은 타겟 프로세스에 대한 핸들을 얻어 오지 못하게 하려는 작업을 해두지 않을까요?<br>

## :exclamation: 결론부터 말하자면 가능합니다!
<img src="https://user-images.githubusercontent.com/40850499/42859182-6858e3b8-8a8d-11e8-9a8c-105a4f39ec9f.PNG"/>
위 이미지를 참조하시면 특정 프로세스에서 타겟 프로세스에 대한 핸들을 가지고 있습니다.<br>
이미 가지고 있는 프로세스에 대해 dll을 주입 한 뒤, dll을 실제 명령을 시행하는 서버로 두고 명령을 내리는 클라이언트를 만들어주면
타겟 프로세스에 직접 접근하는 코드는 없어 지는 것이죠.

## :pushpin: 프로세스 간 통신(Inter-Process Communication, IPC)

프로세스간 통신방법에는 아래와 같이 많은 방법이 있습니다.
  - File (*)
  - Anonymous Pipe
  - **Named Pipe**
  - Socket (*)
  - Shared Memory
  - Memory Mapped File
  - Windows Message (*)
  
위의 이미지의 서버가 되는 부분에는 Dll Injection되어 타겟 프로세스의 핸들 정보를 가지고 있을 것입니다.<br>
이 상태에서 클라이언트 측에서 명령을 요청하면 서버는 알아 듣고 해당 작업을 해주어야 합니다.<br>
즉, 서버-클라이언트 혹은 프로세스간의 통신을 할 수 있는 방법이 필요하게됩니다.<br>
여기서 *가 되어있는 부분은 제가 사용 해봤던 IPC 방법입니다.<br>
이번 기술문서에서는 Named Pipe(이름있는 파이프)를 사용하여 서버-클라이언트간의 원격명령을 시행 하는 것을 최종목표로 두고 있습니다.

## :pushpin: Named Pipe
- 파이프의 이름은 특별한 규약이 있습니다. \\.\pipe\pipeNAME 입니다.
- 이름이 정의된 파이프이기 때문에 관계 없이 파이프 이름을 가지고 통신 할 수 있습니다.
- Anonymous Pipe와 다르게 양방향 통신이 가능합니다(PIPE_ACCESS_DUPLEX)<br>
Named Pipe의 사용법에 대해서는 <a href="https://www.joinc.co.kr/w/man/4200/CreateNamedPipe">여기</a>를 참조 해 주세요.

## :one: 테스트 프로그램 작성(타겟 프로그램)
  - Int, String 타입의 변수의 상태를 확인 할 수 있는 프로그램
  
## :two: 테스트 프로그램 작성(파이프 서버)
  - 하이재킹 , Dll Injection이 되는 대상입니다.
  - 타겟 프로그램의 핸들을 가지고 있어야 합니다.
  - CreateNamedPipe로 파이프 핸들을 열어줍니다.
  - ConnectNamedPipe로 클라이언트에서 파이프에 연결이 되어있는지 확인합니다.
  - ReadFile, WriteFile로 서버와 클라이언트간 수신, 송신을 합니다.
  
## :three: 테스트 프로그램 작성(파이프 클라이언트)
  - 원격 명령을 내릴 클라이언트 프로그램입니다.
  - CreateFile로 파이프 핸들을 가져옵니다.
  - ReadFile, WriteFile로 서버와 클라이언트간 수신, 송신을 합니다.
