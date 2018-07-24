
# :speech_balloon: PPL + LSASS

해당 기술은 Windows Internals의 저자인 `Alex Ionescu`의 글을 참조 하였습니다.<br>
원문 글을 보고 싶은 분들은 <a href="http://www.alex-ionescu.com/?p=97">링크</a>를 클릭 해 주세요 :)

## :green_book: ReadMe
  - LSASS
  - PPL
  - PPL Bypass
  
## LSASS(Local Security Authority)

`lsass` 프로세스는 `Windows 2000`부터 윈도우 보안 모델에 적용된 기술입니다.<br>
Windows2000, Windows XP, ···, Windows 10에서 lsass 프로세스를 찾아 볼 수 있습니다.<br>
`lsass`는 윈도우에서 런타임 상태와 모든 로그인 작업을 담당합니다.<br>

> 그렇다면 왜 lsass는 프로세스의 핸들을 가지고 있는가?

프로그램을 실행하기전 윈도우 비스타부터 `관리자 권한으로 실행` 이라는 항목이 있다.<br>
`lsass`는 로컬 로그인의 권한을 확인하는 프로세스이므로 프로세스의 권한에 관련된 작업을 다루고 있기 때문에, <br>
실행 되고 있는 프로세스의 핸들과 실행 될 프로세스의 핸들을 가지고 있다고 유추하고 있습니다.

## PPL(Protected Process Light)

> Windows 8.1 부터 적용 되었습니다.

- Demo.cpp
```c++
#include <iostream>
#include <windows.h>

using namespace std;

bool SetPrivilege(LPCWSTR lpszPrivilege, BOOL bEnablePrivilege) {
  // coded by harakrinox
	TOKEN_PRIVILEGES priv = { 0,0,0,0 };
	HANDLE hToken = NULL;
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
	
	cout << "Input lasss.exe id : ";
	cin >> PID;

	cout << SetPrivilege(SE_DEBUG_NAME, TRUE) << endl;

	hProcess = OpenProcess(PROCESS_QUERY_INFORMATION | PROCESS_VM_READ, FALSE, PID);

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

- Windows 7에서 lsass 프로세스 핸들 
- Windows 10에서 lsass 프로세스 핸들

발단: lsass 프로세스를 이용하여 로컬 사용자의 비밀번호를 유추 할 수 있는 점이 있었습니다.



