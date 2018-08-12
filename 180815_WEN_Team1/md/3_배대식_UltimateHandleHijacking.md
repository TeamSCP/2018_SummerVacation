# :speech_balloon: Ultimate Handle Hijacking

해당 기술은 UC 포럼의 `harakrinox`의 글을 참조 하였습니다.<br>
원문 글을 보고 싶은 분들은 <a href="https://www.unknowncheats.me/forum/anti-cheat-bypass/261176-silentjack-ultimate-handle-hijacking-user-mode-multi-ac-bypass-eac-tested.html">링크</a>를 클릭 해 주세요 :)

## :green_book: ReadMe

기존의 `Handle Hijacking`을 AC(Anti Cheat)가 탐지 할 수 있는 벡터(요소)들을 생각하여 보완된 기술 입니다.<br>
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
에 대해 구현을 할 줄 알아야 합니다.<br>
위의 내용에 대해서 구현을 하기 위한 필요한 API 목록은 다음과 같습니다.<br>

```
  - CreateFileMapping, OpenFileMapping, MapViewOfFile
  - SuspendThread, GetThreadContext, SetThreadContext, ResumeThread
  - VirtualQueryEx
  - CreateToolhelp32Snapshot, Thread32First, Thread32Next, NtQueryInformationThread, EnumProcessModules, GetModuleFileNameEx, GetModuleInformation
  - DuplicateHandle
```

## :grey_question: So what different?

기존의 Handle Hijacking 구상 방법은 아래의 이미지와 같습니다.<br>
<img src="https://user-images.githubusercontent.com/40850499/43772484-3fcf8672-9a7d-11e8-8ff9-b965a82579e2.PNG" />
클라이언트와 LSASS 혹은 CSRSS에 삽입된 DLL과의 NamedPipe를 통해 서로 통신을 하여 실질적인 작업은 삽입된 DLL에서 처리 해주는 방법입니다.<br>

반면에 Ultimate Handle Hijacking은 어떤가요? 생각외로 복잡한 방법이네요 :flushed: <br>
<img src="https://user-images.githubusercontent.com/40850499/43793129-9f16660e-9ab5-11e8-975f-b179c4c40ebf.png" />

이 보완된 기법에 주된 우회 방법은 `재사용`이라는 키워드로 정리 할 수 있을 것 같습니다.<br>
AC가 어떠한 이유로 기존의 핸들 하이재킹을 탐지가 가능하고, 이것을 우회 할 수 있는 방법을 단계별로 진행 하겠습니다.<br>

## :pencil2: Step 1 \- Create handless share memory

윈도우에서 API를 통해 공유메모리를 생성 하는 방법은 다음과 같습니다.<br>
두 가지의 경우가 있는데, 첫 번째는 공유메모리를 생성 하는 방법이고 두 번째는 만들어진 공유메모리를 사용하는 방법입니다.<br>

__\# 방법1. 공유메모리를 생성하고 닫는 과정__<br>
```
1. CreateFileMapping
2. MapViewOfFile
3. UnMapViewOfFile
4. CloseHandle
```
__\# 방법2. 만들어진 공유메모리를 사용 하는 방법__<br>
```
1. OpenFileMapping
2. MapViewOfFile
3. CloseHandle
```
MSDN 항목 중 Remarks에서 핸들을 닫아 주어도 공유메모리는 그대로 사용 될 수 있음을 명시합니다.<br>

```
Mapped views of a file mapping object maintain internal references to the object, and a file mapping object does not close until all references to it are released.
Therefore, to fully close a file mapping object, an application must unmap all mapped views of the file mapping object by calling UnmapViewOfFile and close the file mapping object handle by calling CloseHandle. 
These functions can be called in any order.
```

그러면 핸들은 어디에서 사용 될까요? 핸들은 만들어진 공유메모리 섹션을 연결 할 때 사용됩니다.<br>
OpenFileMapping의 마지막 인자인 lpName으로 핸들의 이름을 넣어 공유 메모리 공간을 매핑 시킬 수 있습니다.<br>

기존의 파이프를 이용한 IPC는 핸들을 계속 참조하여야 했습니다.<br>
LSASS는 파이프를 사용하지 않음에 불구하고 말이지요. 이러한 상황은 AC가 충분히 탐지 가능한 영역입니다.<br>
그러기 때문에 핸들로 접근 하는것이 아닌 실제 메모리 영역의 주소를 탐색하여 접근 할 수 있게끔 설계를 해두는 것 입니다.<BR>

여기서 한 가지 더 생각 해야 하는 점이 있습니다.<br>
공유메모리의 생성은 어떤 프로세스가 하는지, 만들어진 공유메모리에 접근은 어떤 프로세스가 하는지를 명확히 할 필요가 있습니다.<br>
LSASS, CSRSS는 시스템에서 사용하는 프로세스입니다. 커널과도 통신을 하는 프로세스지요.<br>
즉, 좀 더 높은 권한을 가지고 있는 프로세스에 공유메모리 생성을 하는 것은 권한 문제로 부적절 할 수 있습니다.<br>

## :pencil2: Step2 \- Windows Authority
이 기법에서는 LSASS라는 시스템 프로세스를 OpenProcess로 연 뒤 메모리를 조사하여야 합니다.<br>
하지만 프로세스의 핸들을 요청했지만 `Access is denied` 에러가 발생 하면서 핸들을 주는 것을 거부합니다.<br>
접근 거부 에러가 발생 하는 이유는 권한이 부족하기 때문입니다. 하지만 저는 Administrator 계정으로 로그인 되어 있는 상태인데 무엇이 문제일까요?<br>

이러한 상태를 이해하려면 윈도우 권한에 대해 이해하여야 합니다.<br>
윈도우를 로그인 할 때 계정에 대한 `토큰`을 발급 받고, 프로세스들은 각자의 권한에 맞게 `토큰의 복제본`을 발급 받습니다.<br>
그리고 이 토큰은 권한을 가지고 있는데 이 권한에 따라 운영체제에게 요청 할 수 있는 `범위가 늘어나거나 줄어들 수` 있습니다.<br>
예를 들어, 관리자 권한으로 실행되지 않은 프로세스들은 매우 적은 권한을 가지고 있는 것을 프로세스 모니터링 툴로 확인 할 수 있습니다.<br>

```
1. OpenProcessToken
2. LookupPrivilegeValue
3. AdjustTokenPrivileges
```

위의 내용을 API로 정리한다면 OpenProcessToken 함수로 `토큰 요청`, LookupPrivilegeValue 함수로 텍스트에 대한 `LUID(LocalUniqueID) 값` 요청, AdustTokenPrivleges 함수로 권한이 변경된 토큰 적용하여 특정 권한을 활성화 시킵니다.<br>

```c++
#define SE_TCB_NAME                                  TEXT("SeTcbPrivilege")
#define SE_LOAD_DRIVER_NAME                          TEXT("SeLoadDriverPrivilege")
#define SE_SYSTEM_PROFILE_NAME                       TEXT("SeSystemProfilePrivilege")
#define SE_SYSTEMTIME_NAME                           TEXT("SeSystemtimePrivilege")
#define SE_PROF_SINGLE_PROCESS_NAME                  TEXT("SeProfileSingleProcessPrivilege")
#define SE_INC_BASE_PRIORITY_NAME                    TEXT("SeIncreaseBasePriorityPrivilege")
#define SE_CREATE_PAGEFILE_NAME                      TEXT("SeCreatePagefilePrivilege")
#define SE_CREATE_PERMANENT_NAME                     TEXT("SeCreatePermanentPrivilege")
#define SE_BACKUP_NAME                               TEXT("SeBackupPrivilege")
#define SE_RESTORE_NAME                              TEXT("SeRestorePrivilege")
#define SE_SHUTDOWN_NAME                             TEXT("SeShutdownPrivilege")
#define SE_DEBUG_NAME                                TEXT("SeDebugPrivilege")
#define SE_AUDIT_NAME                                TEXT("SeAuditPrivilege")
```

윈도우에선 위와 같이 많은 권한을 가지고 있는데 예를 들어 시스템을 종료하고 싶다면 SeShutdownPrivilege의 LUID 값이 필요하게 되고 프로세스를 디버깅 하고 싶다면 SeDebugPrivilege가 필요하게 되는 것 입니다.<br>

아래의 코드는 lpszPrivilege로 권한의 텍스트 이름을 bEnablePrivilege로 활성, 비활성 여부를 설정합니다.

```c++
bool SetPrivilege(LPCSTR lpszPrivilege, BOOL bEnablePrivilege) {
	HANDLE hToken;
	TOKEN_PRIVILEGES priv = { 0,0,0,0 };
	LUID luid = { 0, };
	
	if (!OpenProcessToken(GetCurrentProcess(), TOKEN_ADJUST_PRIVILEGES, &hToken)) // Get process token
	{
		if (hToken) {
			CloseHandle(hToken);
		}
		cout << "OpenProcessToken Failed. GetLastError: " << GetLastError() << endl;
		return false;
	}
	if (!LookupPrivilegeValue(NULL, lpszPrivilege, &luid)) // Find privilege value with text macros.
	{
		if (hToken) {
			CloseHandle(hToken);
		}
		cout << "LookupPrivilegeValue Failed. GetLastError: " << GetLastError() << endl;
		return false;
	}

	priv.PrivilegeCount = 1;
	priv.Privileges[0].Attributes = bEnablePrivilege ? SE_PRIVILEGE_ENABLED : SE_PRIVILEGE_REMOVED; 
	priv.Privileges[0].Luid = luid; // Privileges

	if (!AdjustTokenPrivileges(hToken, false, &priv, sizeof(TOKEN_PRIVILEGES), 0, 0)) // Change privileges
	{
		if (hToken) {
			CloseHandle(hToken);
		}
		cout << "AdjustTokenPrivileges Failed. GetLastError: " << GetLastError() << endl;
		return false;
	}
	if (hToken) {
		CloseHandle(hToken);
	}
	return true;
}
```

## :pencil2: Step3 \- DuplicateHandle

Create handless share memory를 생성 할 때 핸들을 닫게 되는데, 다시 공유메모리에 대한 경로를 남겨 두기 위해 DuplicationHandle API를 사용해<br>
핸들을 화이트 프로세스(explorer.exe)로 복사해 둡니다.

## :pencil2: Step4 \- Searching R/W/E Section
메모리 영역이 실행 될 수 있는지 여부는 게임 해킹에서 매우 중요한 요소로 작용합니다.<br>
예를 들어, Read/Write 가능한 영역에 어셈블리 코드를 작성하여 넣었습니다. 하지만 실행 권한이 없으므로 EIP가 그 영역으로 간다 한들 아무 의미 없을 것입니다.<br>

위에서 공유 메모리를 매핑하기 위해선 OpenFileMapping을 시스템 프로세스인 LSASS에서 호출 하여야 합니다.<br>
그러한 코드를 어셈블리로 작성하여 실행 될 수 있는 영역에 주어야 정상적으로 그 API를 호출 할 수 있겠죠?<br>

프로세스에서 사용되고 있는 메모리 섹션을 조사하는 방법은 `VirtualQuery, VirtualQueryEx`라는 API가 있습니다.<br>
API는 out으로 MEMORY_BASIC_INFORMATION 구조체에 메모리 섹션에 대한 정보를 담게 됩니다.<br>

```c++
typedef struct _MEMORY_BASIC_INFORMATION {
    PVOID BaseAddress;		// 실 메모리 섹션 주소
    PVOID AllocationBase;
    DWORD AllocationProtect;
    SIZE_T RegionSize;		// 메모리 섹션의 크기
    DWORD State;
    DWORD Protect;		// 메모리의 읽기, 쓰기, 실행 속성
    DWORD Type;
} MEMORY_BASIC_INFORMATION, *PMEMORY_BASIC_INFORMATION;
```

아래의 코드는 쉘 코드보다 작은 공간을 검색하기 위해 메모리 섹션이 실행 옵션이 있고, 섹션의 마지막으로부터 0이아닌 부분을 찾아 코드를 삽입 할 수 있는 크기를 구하는 과정의 코드입니다.<br>

```c++
int main()
{
	HANDLE hProcess;
	DWORD PID;
	MEMORY_BASIC_INFORMATION mbi;
	DWORD Address = 0;
	vector<MEMORY_BASIC_INFORMATION> exec_mbi;

	if (!SetPrivilege(SE_DEBUG_NAME, TRUE)) // VQE Need Debug privileges
	{
		cout << "SetPrivilege Failed. GetLastError: " << GetLastError() << endl;
	}
	cout << "Input LSASS PID: ";
	cin >> dec >> PID;
	hProcess = OpenProcess(PROCESS_ALL_ACCESS, NULL, PID);
	if (!hProcess)
	{
		cout << "OpenProcess Failed. GetLastError: " << GetLastError() << endl;
		return FALSE;
	}
	do {
		VirtualQueryEx(hProcess, (LPCVOID)Address, &mbi, sizeof(MEMORY_BASIC_INFORMATION));

		if (mbi.Protect == PAGE_EXECUTE_READ || mbi.Protect == PAGE_EXECUTE_READWRITE || mbi.Protect == PAGE_EXECUTE_WRITECOPY || mbi.Protect == PAGE_EXECUTE) {
			exec_mbi.push_back(mbi);
		}
		Address += mbi.RegionSize;
	} while (Address <= (DWORD)0x7FFFFFFF); // 32bit

	for (int i = 0; i < exec_mbi.size(); i++)
	{
		unsigned int size = 0;
		unsigned char* Mem_Test = 0;

		cout << "[+] AllocationBase = " << exec_mbi[i].AllocationBase << endl;
		cout << "[+] AllocationProtect = " << exec_mbi[i].AllocationProtect << endl;
		cout << "[+] BaseAddress = " << exec_mbi[i].BaseAddress << endl;
		cout << "[+] Protect = " << exec_mbi[i].Protect << endl;
		cout << "[+] RegionSize = " << exec_mbi[i].RegionSize << endl;
		Mem_Test = (unsigned char*)malloc(exec_mbi[i].RegionSize); // Allocating be region size
		ReadProcessMemory(hProcess, exec_mbi[i].BaseAddress, Mem_Test, exec_mbi[i].RegionSize, NULL); // Copy memory section
		for (size = (exec_mbi[i].RegionSize - 1); size > 0; size--) 
		{
			if (Mem_Test[size] != 0)
			{
				break;
			}
		}
		cout << "Usable Size : " << (exec_mbi[i].RegionSize - size) << endl;
		free(Mem_Test);
	}
}
```

공유 메모리를 매핑시키는 코드와 IPC를 구현 하기 위한 Spinlock 코드보다 작은 메모리 공간은 사용 될 수 없기 때문에, 크기 확인은 꼭 필요합니다.<br>

## :pencil2: Step5 \- How to make shellcode and What is spinlock? 
```asm
SpinLock: 
	repe nop
	cmp bl, [sharemem address]
	jne SpinLock
	ret
```
스핀 락의 개념은 특정 조건을 만족 할 때 까지 계속 도는 것입니다. 조건이 만족하면? 리턴 하는 간단한 구조입니다.<br>
```asm
SpinLock:
	repe nop
	xor eax,eax
	mov eax, [sharemem_rpm]
	cmp eax, 1 // on
	jne IsEnd
	push 0
	push [sharemem_size]
	push [sharemem_buffer]
	push [sharemem_target_mem]
	call ReadProcessMemory
IsEnd:
	cmp bl, [spinlock on]
	jne SpinLock
	ret
```
이런식으로 구현하여 공유메모리의 특정 공간을 통해 요청을 하면 프로세스간 통신이 성립 될 수 있다는 것이지요.<br>

## :pencil2: Step6 \- Finding Thread information

ToolHelp를 통한 TID 수집, NtQueryInformationThread를 통한 스레드 시작주소 수집, 시작주소가 어디 모듈에 속해 있는지 확인.

```c++
#include <iostream>
#include <Windows.h>
#include <tlhelp32.h>
#include <Winternl.h>
#include <Psapi.h>
#include <vector>

using namespace std;

#pragma comment (lib, "ntdll.lib")
#define ThreadQuerySetWin32StartAddress 9

int main()
{
	THREADENTRY32 th_list;
	DWORD PID;
	HANDLE hSnapShot;
	HANDLE hProcess;
	HMODULE hModules[1024];
	DWORD cbNeed;
	MODULEINFO mi;
	vector<THREADENTRY32> tid_list;
	vector<DWORD> thread_start_list;

	if (!SetPrivilege(SE_DEBUG_NAME, TRUE))
	{
		cout << "SetPrivilege Failed. GetLastError: " << GetLastError() << endl;
	}

	cout << "Input lsass PID : ";
	cin >> dec >> PID;

	th_list.dwSize = sizeof(THREADENTRY32);

	hSnapShot = CreateToolhelp32Snapshot(TH32CS_SNAPTHREAD, PID);
	if (!hSnapShot)
	{
		return FALSE;
	}

	if (!Thread32First(hSnapShot, &th_list))
	{
		CloseHandle(hSnapShot);
		return FALSE;
	}
	do {
		if ((DWORD)th_list.th32OwnerProcessID == PID)
		{
			tid_list.push_back(th_list);
		}
	} while (Thread32Next(hSnapShot, &th_list));

	for (int i = 0; i < tid_list.size(); i++)
	{
		HANDLE hThread = OpenThread(THREAD_ALL_ACCESS, false, tid_list[i].th32ThreadID);
		DWORD StartAddress = 0;

		if (!hThread)
		{
			cout << "OpenThread Failed Reason : " << GetLastError() << endl;
			return false;
		}

		NtQueryInformationThread(hThread, (THREADINFOCLASS)ThreadQuerySetWin32StartAddress, &StartAddress, sizeof(StartAddress), NULL);
		CloseHandle(hThread);
		thread_start_list.push_back((DWORD)StartAddress);
		cout << "Thread ID : " << tid_list[i].th32ThreadID << endl;
		cout << "Thread Start Address : " << hex << thread_start_list[i] << endl;
	}
	hProcess = OpenProcess(PROCESS_QUERY_INFORMATION | PROCESS_VM_READ, FALSE, PID);

	if (!hProcess)
	{
		cout << "OpenProcess Failed Reason : " << GetLastError() << endl;
		return false;
	}
	if (!EnumProcessModules(hProcess, hModules, sizeof(hModules), &cbNeed))
	{
		cout << "EnumProcessModules Call failed.. Reason : " << GetLastError() << endl;
	}

	for (int i = 0; i < (cbNeed/sizeof(HMODULE)); i++) {
		TCHAR szModeName[256];
		GetModuleFileNameEx(hProcess, hModules[i], szModeName, sizeof(szModeName) / sizeof(TCHAR));
		GetModuleInformation(hProcess, hModules[i], &mi, sizeof(mi));

		cout << "Module Name : " << szModeName << endl;
		cout << "Base Address : " << mi.lpBaseOfDll << endl;
	}

	CloseHandle(hProcess);
	CloseHandle(hSnapShot);

	return 0;
}
```

## :pencil2: Step7 \- Execute Shell-code by Thread context->EIP
`Searching R/W/E Section` 에서 코드가 실행 될 수 있는 영역을 찾았습니다.<br>
그리고 쉘 코드의 길이를 조사해 남아있는 공간과 비교 한 뒤, 어셈블리 코드를 복사한 상태입니다.<br>

하지만 이 영역을 실행 시켜줄, 매개체를 어디서 찾을까요?<br>

스레드로 새로운 실행 흐름을 만들 수도 없고, DLL Injection으로 DLLMain의 시작 지점을 호출되게 할 수 없는 상태입니다.<br>
여기서 사용 될 수 있는 방법이 크리티컬하지 않은 스레드를 찾아 그 스레드를 멈추고 Context를 바꾸어 주는 작업을 하야 EIP 위치를 쉘코드로 바꾸어 주는 것 입니다.<br>

```
OpenThread
SuspenThread
GetThreadContext
SetThreadContext
ResumeThread
```

여기서 크리티컬하지 않다는 의미는 스레드가 사용 되지 않아도 LSASS 로직상 문제가 발생하지 않을 경우의 스레드를 의미합니다.<br>
스레드들은 각각의 실행 흐름을 가지고 있습니다. 그리고 스레드의 특성상 같은 메모리는 공유하나 레지스터는 틀리죠.<br>
즉 Context 내에 각각의 레지스터의 정보를 가지고 있을 것이고 EIP 값을 쉘코드의 주소로 바꾸어 주면 작업은 완료됩니다.<br>

```
typedef struct _CONTEXT {
    DWORD ContextFlags;

    DWORD   Dr0;
    DWORD   Dr1;
    DWORD   Dr2;
    DWORD   Dr3;
    DWORD   Dr6;
    DWORD   Dr7;

    FLOATING_SAVE_AREA FloatSave;
    
    DWORD   SegGs;
    DWORD   SegFs;
    DWORD   SegEs;
    DWORD   SegDs;

    DWORD   Edi;
    DWORD   Esi;
    DWORD   Ebx;
    DWORD   Edx;
    DWORD   Ecx;
    DWORD   Eax;

    DWORD   Ebp;
    DWORD   Eip;
    DWORD   SegCs;              // MUST BE SANITIZED
    DWORD   EFlags;             // MUST BE SANITIZED
    DWORD   Esp;
    DWORD   SegSs;
    
    BYTE    ExtendedRegisters[MAXIMUM_SUPPORTED_EXTENSION];
} CONTEXT;
```

## :star: 후기
어렵다 -_-..

## :bookmark_tabs: 참조 링크

<a href="https://docs.microsoft.com/ko-kr/windows/desktop/TermServ/kernel-object-namespaces">Kernel-object-namespaces</a><br>
<a href="https://docs.microsoft.com/ko-kr/windows/desktop/SecAuthZ/enabling-and-disabling-privileges-in-c--">Enable and disabling privileges in c++</a>
