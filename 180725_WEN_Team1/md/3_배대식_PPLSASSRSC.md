
# :speech_balloon: PPL + LSASS + CSRSS

해당 기술은 Windows Internals의 저자인 `Alex Ionescu`의 글을 참조 하였습니다.</br>
원문 글을 보고 싶은 분들은 <a href="http://www.alex-ionescu.com/?p=97">링크</a>를 클릭 해 주세요 :)

## :green_book: ReadMe
  - LSASS
  - CSRSS
  - PPL
  - PPL Bypass
  
## :purple_heart: LSASS(Local Security Authority)

`LSASS` 프로세스는 `Windows 2000` 부터 윈도우 보안 모델에 적용된 기술입니다.</br>
Windows2000, Windows XP, ···, Windows 10에서 LSASS 프로세스를 찾아 볼 수 있습니다.</br>
`LSASS` 는 윈도우에서 런타임 상태와 모든 로그인 작업을 담당합니다.</br>
> 그렇다면 왜 LSASS는 다른 프로세스의 핸들을 가지고 있나요?

예를 들어 프로그램A가 프로그램B의 프로세스 핸들을 OpenProcess로 요청했습니다.<br>
그렇지만 프로그램B는 프로그램A보다 권한이 높습니다.<br>
결과적으로 프로그램A는 프로그램B의 핸들을 얻어 올 수 없게 됩니다.<br>
이러한 권한 작업을 확인하는 놈이 `SRM(SecurityReferenceMoniter)`인 것입니다<br>
다시 한번 정리하자면 프로그램A가 프로그램B의 핸들을 요청하기 위해 Lsass에게 보내고<br> 
이 요청을 SRM에서 확인 후 핸들 리턴 여부를 확인 하는 것 입니다.<br>

## :heart: CSRSS(Client/Server Runtime Subsystem)

<a href="https://ko.wikipedia.org/wiki/%ED%81%B4%EB%9D%BC%EC%9D%B4%EC%96%B8%ED%8A%B8/%EC%84%9C%EB%B2%84_%EB%9F%B0%ED%83%80%EC%9E%84_%ED%95%98%EC%9C%84_%EC%8B%9C%EC%8A%A4%ED%85%9C">위키</a>를 읽어보면 사용자 프로세스가 콘솔, 프로세스, 스레드 관련 함수를 호출 할 때 CSRSS 프로세스간 호출을 보낸다 한다.<br>
이 정보론 왜 CSRSS가 다른 프로세스의 핸들을 가지고 있는지 이해를 할 수 없다.<br>
<a href="https://www.microsoftpressstore.com/articles/article.aspx?p=2233328&seqNum=3">링크</a>에는 윈도우에서 프로세스/쓰레드가 생성 될 때 어떤 작업을 거치는지 상세하게 설명 해준다.<br>

<img src="https://user-images.githubusercontent.com/40850499/43233770-dde97ee6-90b2-11e8-85cd-723b38bb6ce6.jpg"/>

위의 이미지중 stage 5 과 stage7 사이에 sub system을 거치는데 이 부분이 csrss이다.
즉, csrss에서 유저모드에서 접근 할 수 있게 프로세스와 스레드의 핸들 값 복사등 여러 초기화 작업을 먼저 해주기 때문에 다른 프로세스의 핸들을 가지고 있는 것 이다<br>
그리고 이 프로세스는 시스템 프로세스이며 `SeTcbPrivilege` 이상의 권한을 가지고 있을 때 열 수 있다.<br>
결과적으로 CSRSS 프로세스에게 인젝션이나 핸들을 가져올려면 실행 권한과, 보호된 프로세스를 우회 해야할 것 이다.<br>

## :blue_heart: PPL(Protected Process Light)

Windows 8.1 부터 도입된 개념이다.</br>
저자의 글을 살펴보면 LSASS 프로세스에서 해쉬화된 로컬 관련 데이터들이 평문으로 유출 되는 점 외에도 문제점이 많아 보호된 프로세스 개념을 도입 했다고 칸다..<br>
이제 PPL을 사용하여 lsass 프로세스로의 dll injection을 차단하는 작업을 해볼 것입니다.<br>

우선 같은 Dll Injector로 `LSASS` 프로세스에 더미 dll 파일을 인젝션 해 보았습니다.

> Windows7 32bit Dll Injection
<img src="https://user-images.githubusercontent.com/40850499/43191449-2cc2ed72-9036-11e8-9318-4d9575e8eeaf.PNG"/>

> Windows10 64bit Dll Injection
<img src="https://user-images.githubusercontent.com/40850499/43191450-2dbeed2a-9036-11e8-96f1-5805c086ee20.PNG"/>

두 운영체제 모두 다 잘 되는것을 확인 할 수 있습니다.<br>
그렇다면 Windows10에서 `_PS_PROTECTION` 구조체의 값을 수정하여 DLL Injection을 차단 해 보겠습니다.<br>
`_PS_PROTECTION` 구조체는 `EPROCESS` 구조체의 `멤버`입니다.<br>
하지만 `EPROCESS` 구조체는 커널모드에서 사용하는 리소스이기 때문에 커널 디버거로 데이터를 확인 해야 합니다.</br>

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

> EPROCESS → Protection

```C
1 _PS_PROTECTION
2   +0x000 Level            : UChar
3   +0x000 Type             : Pos 0, 3 Bits
4   +0x000 Audit            : Pos 3, 1 Bit
5   +0x000 Signer           : Pos 4, 4 Bits
```
_PS_PROTECTION 구조체는 8Bit의 크기이다.<br>

- 타입은 아래의 10진수 값이 적용되며 PPL은 1의 값을 가지고 있다.

```C
1 _PS_PROTECTED_TYPE
2   PsProtectedTypeNone = 0n0
3   PsProtectedTypeProtectedLight = 0n1
4   PsProtectedTypeProtected = 0n2
5   PsProtectedTypeMax = 0n3
```

- 서명자는 ProtectedSigner 뒤의 문자열이 서명자가 된다.

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

자 그러면 실제 LSASS 프로세스의 _PS_PROTECTED 구조체를 확인 해보고, 비트를 설정 해준 뒤 다시 인젝션을 해보자!<br>

> kd : process 0 0 lsass.exe

<img src="https://user-images.githubusercontent.com/40850499/43199611-82948276-904d-11e8-81f3-99eebeb0f21d.PNG"/>

빨간색 박스 부분이 EPROCESS의 시작 주소이다.

> kd : ?? ((nt!_EPROCESS*)0xADDRESS)

ProtectedProcess 멤버의 오프셋<br>
<img src="https://user-images.githubusercontent.com/40850499/43199618-83151e18-904d-11e8-89c6-60579042ddaf.PNG"/>

EPROCESS 시작주소를 심볼에서 가져온 EPROCESS랑 매칭 시켜준다.<br>
이미지를 확인해보면 0x6B2에 1Byte를 사용 하는 것을 알 수 있다!<br>
상세한 내용을 확인해보면?<br>

<img src="https://user-images.githubusercontent.com/40850499/43199616-82eb85ee-904d-11e8-95ed-c98caa702f66.PNG"/>

아무런 보호가 되어 있지 않다.<br>

> kd : eb <EPROCESS+0x6B2> <0x41>
	
<img src="https://user-images.githubusercontent.com/40850499/43199613-82c04d52-904d-11e8-9607-70a8823622b1.PNG"/>

나는 dll injection을 막고 싶은거니 Signer를 Lsa 4와 ProtectedLight 1비트를 합치면 0x41이 된다.<br>

<img src="https://user-images.githubusercontent.com/40850499/43199620-8343f602-904d-11e8-958c-70043ace44fe.PNG"/>

> 최종 결과
<img src="https://user-images.githubusercontent.com/40850499/43199609-82671b24-904d-11e8-851a-c9119fc18e2c.PNG"/>


## PPL Bypass

매우 간단합니다. 보호 된 프로세스의 EPROCESS 주소를 구한 뒤 ProtectedProcess bit를 0으로 off 해주면 됩니다.<br>
하지만 커널모드에 접근 해야 되므로 드라이버를 로드하여 작업을 해야되는 번거로움이 있습니다.<br>
그 외에도 패치가드나 인증된 드라이버등 잡다한 기능이 우리를 가로 막고 있지만, 자기 컴퓨터에 적용하는거라 상관 없는 내용이긴 합니다.<br>

- Sample.cpp
```CPP
#include <iostream>
#include <windows.h>
#include <stdint.h>

using namespace std;

bool SetPrivilege(LPCWSTR lpszPrivilege, BOOL bEnablePrivilege) {
	// coded by harakrinox - I using test code for privilege check. sorry :(
	TOKEN_PRIVILEGES priv = { 0,0,0,0 };
	HANDLE hToken = NULL, hProcess = NULL;
	LUID luid = { 0,0 };

	if (!OpenProcessToken(GetCurrentProcess(), TOKEN_ADJUST_PRIVILEGES, &hToken)) {
		if (hToken)
			CloseHandle(hToken);
		return false;
	}
	if (!LookupPrivilegeValueW(0, lpszPrivilege, &luid)) {
		if (hToken)
			CloseHandle(hToken);
		return false;
	}
	priv.PrivilegeCount = 1;
	priv.Privileges[0].Luid = luid;
	priv.Privileges[0].Attributes = bEnablePrivilege ? SE_PRIVILEGE_ENABLED : SE_PRIVILEGE_REMOVED;
	if (!AdjustTokenPrivileges(hToken, false, &priv, 0, 0, 0)) {
		if (hToken)
			CloseHandle(hToken);
		return false;
	}
	if (hToken)
		CloseHandle(hToken);
	return true;
}

int main()
{
	DWORD PID;
	HANDLE hProcess;
	uint8_t byte[10];
	
	cout << "Input pid : ";
	cin >> PID;

	cout << SetPrivilege(SE_DEBUG_NAME, TRUE) << endl;

	hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, PID);

	if (!hProcess)
	{
		cout << "Error code : " << GetLastError() << endl;
	}
	else
	{
		cout << "handle : " << hProcess << endl;
	}
	return 0;
}
```
위의 코드는 PID를 요청 했을 때 OpenProcess로 프로세스 핸들을 출력 해주는 간단한 프로그램입니다.<br>
만약 ProtectedProcess가 걸려있을 경우 ErrorCode: 5를 리턴하며 끝나게 되는데요.<br>
WinDbg로 직접 해당 값을 수정하여 핸들을 구해보도록 하겠습니다.<br>

1. csrss.exe의 핸들 값 구하기 시도.

<img src="https://user-images.githubusercontent.com/40850499/43200955-250bd2a8-9052-11e8-86a4-804bf1576787.PNG"/>

2. csrss.exe의 권한 확인

<img src="https://user-images.githubusercontent.com/40850499/43200701-539a116c-9051-11e8-922b-793f971718a2.PNG"/>

3. PPL Off

<img src="https://user-images.githubusercontent.com/40850499/43200700-536fe87e-9051-11e8-8e4c-6e126e9a4dea.PNG"/>

4. 성공!

<img src="https://user-images.githubusercontent.com/40850499/43200699-5346ead2-9051-11e8-85b9-27592b44f611.PNG"/>
