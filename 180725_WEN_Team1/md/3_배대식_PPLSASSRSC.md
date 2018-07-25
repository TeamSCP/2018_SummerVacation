
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

프로그램을 실행하기전 윈도우 비스타부터 `관리자 권한으로 실행` 이라는 항목이 있습니다.</br>
`LSASS` 는 로컬 로그인의 권한을 확인하는 프로세스이므로 프로세스의 권한에 관련된 작업을 다루고 있기 때문에, </br>
실행 되고 있는 프로세스의 핸들과 실행 될 프로세스의 핸들을 가지고 있다고 유추하고 있습니다.

## :heart: CSRSS(Client/Server Runtime Subsystem)



## :blue_heart: PPL(Protected Process Light)

같은 Dll Injector로 `LSASS` 프로세스에 더미 dll 파일을 인젝션 해 보았습니다.

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
위의 코드는 PID를 요청 했을 때 OpenProcess로 프로세스 핸들을 출력 해주는 간단한 프로그램입니다.
만약 ProtectedProcess가 걸려있을 경우 ErrorCode: 5를 리턴하며 끝나게 되는데요.

WinDbg로 직접 해당 값을 수정하여 핸들을 구해보도록 하겠습니다.

