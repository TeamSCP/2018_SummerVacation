# :speech_balloon: Handle Hijacking

해당 기술은 UC 포럼의 `harakrinox`의 글을 참조 하였습니다.<br>
원문 글을 보고 싶은 분들은 <a href="https://www.unknowncheats.me/forum/anti-cheat-bypass/217995-poc-remote-memory-operations-using-existing-handle.html">링크</a>를 클릭 해 주세요 :)

## :green_book: ReadMe
  
  - 프로세스간 통신(IPC)중 Named Pipe를 사용합니다.
  - `공격 프로그램이 핸들을 구해 올 수 없다`의 가정 하에 진행 됩니다.
  - `개념증명(PoC)`에 사용 되는 코드는 대상이 둘 다 실행 파일이지만, 파이프 서버쪽을 `하이재킹 대상` 혹은 `dll injection`의 대상이라고 생각 해주세요.
  - 유저 모드에서 사용 되는 우회 기법입니다.

## :airplane: 하이재킹?

개념은 아주 간단합니다. 이미 사용되고 있는 무언가(세션, 쿠키, 핸들 ···)를 가져와 사용 하는 것입니다.<br>
<a href="https://en.wikipedia.org/wiki/Hijacking">Wiki 링크</a> 

## :heavy_check_mark: 핸들에 대해 복습

프로세스, 파일 같은 것들은 모두 커널에서 사용되는 리소스입니다.<br>
이러한 커널 객체를 유저모드에서 접근 하기 위해서 커널 객체에 대한 인스턴스를 만들어주고,<br>
인스턴스에 대해 접근 할 수 있는 방법이 핸들인 것입니다.

## :question: 핸들을 빌려 쓰는 것이 왜 우회인가요?
오래 전(중학교~대학교 1학년)에는 공격 대상이 타겟 프로세스인 경우가 많았습니다.<br>
RPM/WPM으로 메모리를 읽기/쓰기를 하던, Dll을 인젝션을 하여 코드를 수정하던,<br>
대상 프로세스에 직접 접근하여야 했습니다.<br>
그렇다면 Anti-Cheat 개발자 입장에서 바라 봤을 때 대상 프로세스에 대한 핸들을 얻어 오지 못하게 하려는 작업을 해두지 않을까요?<br>

## :exclamation: 결론부터 말하자면 가능합니다!
<img src="https://user-images.githubusercontent.com/40850499/42859182-6858e3b8-8a8d-11e8-9a8c-105a4f39ec9f.PNG"/>
위 이미지를 참조하시면 특정 프로세스(lsass)에서 타겟 프로세스에 대한(Tslgame.exe) 핸들을 가지고 있습니다.<br>
핸들을 이미 가지고 있는 프로세스에 대해 dll을 주입 한 뒤 dll을 실제 명령을 시행하는 서버로 두고 명령을 내리는 클라이언트를 만들어주면<br>
치트 프로그램은 이미 핸들을 가지고 있는 프로세스에게 명령을 내려 메모리를 읽고, 쓰는 원하는 행동을 할 수 있게 됩니다.

## :pushpin: 프로세스 간 통신(Inter-Process Communication, IPC)

프로세스간 통신방법에는 아래와 같이 많은 방법이 있습니다.
  - File
  - Anonymous Pipe
  - **Named Pipe**
  - Socket
  - Shared Memory
  - Memory Mapped File
  - Windows Message
  
위의 이미지의 서버가 되는 부분에는 Dll Injection되어 타겟 프로세스의 핸들 정보를 가지고 있을 것입니다.<br>
이 상태에서 클라이언트 측에서 명령을 요청하면 서버는 알아 듣고 명령에 맞는 작업을 해주어야 합니다.<br>
즉, 서버-클라이언트 혹은 프로세스간의 통신을 할 수 있는 방법이 필요 하게됩니다.<br>
이번 기술 문서에서는 Named Pipe(이름있는 파이프)를 사용하여 서버-클라이언트간의 원격명령을 시행 할 것입니다.

## :pushpin: Named Pipe
- 파이프의 이름은 특별한 규약이 있습니다. `\\.\pipe\pipeNAME` 입니다.
- 이름이 정의된 파이프이기 때문에 관계 없이 파이프 이름을 가지고 통신 할 수 있습니다.
- Anonymous Pipe와 다르게 양방향 통신이 가능합니다.(PIPE_ACCESS_DUPLEX)
- Named Pipe의 사용법에 대해서는 <a href="https://www.joinc.co.kr/w/man/4200/CreateNamedPipe">여기</a>를 참조 해 주세요.

## :one: 테스트 프로그램 작성(타겟 프로그램)
  - Int, String 타입의 변수의 상태를 확인 할 수 있는 프로그램
  ```
#include <iostream>
#include <string>
#include <stdint.h>

using namespace std;

int main()
{
	bool isExit = true;
	uint32_t varNum = 0;
	string varString;

	cout << "Enter a number to load in memory: ";
	cin >> varNum;
	cout << "Enter a string to load in memory: ";
	cin >> varString;

	do {
		cout << "varNum (address = " << hex << &varNum << ") = " << dec << varNum << endl;
		cout << "varString (address = " << hex << &varString << ") = " << varString << endl;
		cout << endl;
		cout << "Enter 1 to exit or 0 to loop :" ;
		cin >> isExit;
	} while (isExit != 0);

	return 0;
}
```
  
## :two: 테스트 프로그램 작성(파이프 서버)
  - 하이재킹, Dll Injection이 되는 대상입니다.
  - 타겟 프로그램의 핸들을 가지고 있어야 합니다.
  - CreateNamedPipe로 파이프 핸들을 열어줍니다.
  - ConnectNamedPipe로 클라이언트에서 파이프에 연결이 되어있는지 확인합니다.
  - Request, Reply 구조체를 가지고 있습니다. 이것으로 서로 명령이나 상태를 확인 합니다.
  - ReadFile, WriteFile로 서버와 클라이언트간 수신, 송신을 합니다.
```
#include <iostream>
#include <stdint.h>
#include <windows.h>

#define _CRT_SECURE_NO_WARNINGS
#define PIPE_NAME "\\\\.\\pipe\\mypipe"
#pragma warning(disable: 4996)

using namespace std;

typedef struct RPMReq {
	char message[10];
	uint32_t Address;
	uint8_t nRead;
}RPMReq;

typedef struct RPMRep {
	uint8_t nRead;
	char buf[10];
}RPMRep;

int main()
{
	HANDLE hProcess, hPipe;
	DWORD PID;
	char buf[256] = { 0, };
	RPMReq* req = (RPMReq*)malloc(sizeof(RPMReq));
	RPMRep* rep = (RPMRep*)malloc(sizeof(RPMRep));

	cout << "Open handle on PID: ";
	cin >> PID;
	hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, PID);
	if (hProcess != NULL)
	{
		cout << "[ OK ] Process " << PID << " " << "successfully opened" << endl;
		cout << "[ OK ] Process handle: " << hProcess << endl;
		cout << "[ OK ] Process handle memory address: " << hex << &hProcess << endl;
	}
	hPipe = CreateNamedPipe(
		PIPE_NAME,
		PIPE_ACCESS_DUPLEX,
		PIPE_TYPE_MESSAGE | PIPE_WAIT,
		PIPE_UNLIMITED_INSTANCES,
		0,
		0,
		NMPWAIT_USE_DEFAULT_WAIT,
		NULL
	);
	
	if (!hPipe)
	{
		cout << "Error : " << GetLastError() << endl;
		return false;
	}
	else
	{
		cout << "[ OP ] Pipe Created.." << endl;
	}
	while (1)
	{
		if (ConnectNamedPipe(hPipe, NULL))
		{
			ReadFile(hPipe, req, sizeof(RPMReq), NULL, NULL);
			if (!strncmp(req->message, "doit", 4))
			{
				if (ReadProcessMemory(hProcess, (LPCVOID)req->Address, &buf, req->nRead, NULL))
				{
					cout << "RPM Success Sending.." << endl;
					memcpy(rep->buf, buf, 10);
					rep->nRead = req->nRead;
					WriteFile(hPipe, rep, sizeof(RPMRep), NULL, NULL);
				}
				else
				{
					cout << "RPM Failed.. " << endl;
				}
			}
			else
			{
				cout << "Unknown Commands" << endl;
			}
		}
	}
	DisconnectNamedPipe(hPipe);
	CloseHandle(hPipe);
	free(req);
	free(rep);
}
```

  
## :three: 테스트 프로그램 작성(파이프 클라이언트)
  - 원격 명령을 내릴 클라이언트 프로그램입니다.
  - CreateFile로 파이프 핸들을 가져옵니다.
  - Request, Reply 구조체를 가지고 있습니다. 이것으로 서로 명령이나 상태를 확인 합니다.
  - ReadFile, WriteFile로 서버와 클라이언트간 수신, 송신을 합니다.
  
```
#include <iostream>
#include <windows.h>

#define PIPE_NAME "\\\\.\\pipe\\mypipe"

using namespace std;

typedef struct RPMReq {
	char message[10];
	uint32_t Address;
	uint8_t nRead;
}RPMReq;

typedef struct RPMRep {
	uint8_t nRead;
	char buf[10];
}RPMRep;

int main()
{
	HANDLE pipe;
	RPMReq* req = (RPMReq*)malloc(sizeof(RPMReq));
	RPMRep* rep = (RPMRep*)malloc(sizeof(RPMRep));
	char buf[256] = { 0, };
	pipe = CreateFile(PIPE_NAME, GENERIC_READ | GENERIC_WRITE, 0, NULL, OPEN_EXISTING, 0, NULL);

	cout << "> Input command: ";
	fgets(req->message, 256, stdin);
	cout << "> Input Address: ";
	cin >> hex >> req->Address;
	cout << "> Input Number: ";
	cin >> dec >> req->nRead;
	while (1)
	{
		WriteFile(pipe, req, sizeof(RPMReq), NULL, NULL);
		if (ReadFile(pipe, rep, sizeof(RPMRep), NULL, NULL))
		{
			printf("RPM Success : %s", rep->buf+4);
		}
	}
	free(req);
	free(rep);
	CloseHandle(pipe);

	return 0;
}
```
