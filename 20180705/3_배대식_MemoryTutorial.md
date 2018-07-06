# Memory Tutorial

다른 프로세스의 메모리를 읽기 위한 방법과 거기서 사용되는 개념
그리고 CheatEngine을 사용하여 가상메모리 주소와 물리메모리 주소 변경을
연습 할 것입니다.

> PID

Process Identifier의 약자이며 프로세스를 구분하기 위해서 사용하는 식별자이다.
크기는 DWORD(4Byte)이며 윈도우에서 예약되어 있는 PID는
`0 - System Idle Time Process`
`4 - System Process`
가 있다. 프로세스를 실행 시 PID값은 달라지지만 중복되는 경우는 없다.

PID를 구하는 방법은
1. NtQuerySystemInformation
3. CreateTootHelpSnapShot
4. EnumProcess
5. GetCurrentProcessID
6. GetProcessID

등이 있다.

이렇게 구한 PID를 나는 프로세스의 핸들을 얻기 위해
`OpenProcess(DWORD dwDesiredAccess, BOOL bInheritHandle, DWORD dwProcessId)`
를 사용하면 얻어 올 수 있다!

> HANDLE

핸들은 어떠한 윈도우 오브젝트들에 접근하기 위해 사용하는 고유의 값이다.
구분의 용도로도 사용하며 윈도우 메시지를 처리 할 때 이 값을 보고 어디서 왔는지 확인 할 수 있는용도로 사용되는 것이다.
결론적으로 프로세스에서는 파일, 프로세스, 그래픽적인 핸들을 여러개 가질 수 있고 오브젝트에 접근하기 위해서는 핸들을 발급 받아서 접근 해야 된다는 것이다.

> OpenProcess, ReadProcessMemory, WritProcessMemory

API 사용방법은 생략하고 사용시에 주의 해야 할 점만..
- OpenProcess의 dwAccess의 권한을 요구하는 API에 맞게 권한 설정 해주어야 한다.
- 핸들을 열어준 뒤에는 CloseHandle로 핸들을 닫아준다.
- RPM과 WPM은 핸들권한에 PROCESS_ALL_ACCESS 혹은 PROCESS_VM_READ | PROCESS_VM_WRITE 권한이 있어야 한다. 없으면 299에러가 뜸

> Virtual Memory <-> Physical Memory

인텔 규격에서 컨트롤 레지스터3(CR3)에 각 프로세스 마다 페이지 디렉토리 엔트리의주소를 가지고 있다.

