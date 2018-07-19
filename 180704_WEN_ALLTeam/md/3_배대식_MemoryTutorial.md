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
```
참조:
페이지 디렉토리 엔트리, 페이지 테이블 엔트리는 각각 1024의 인덱스를 가지고 있으며 32bit 환경에서는 4byte의 64bit 환경에서는 8byte의 크기를 가지고 있다.
```

그리고 가상메모리 주소가 각각의 `페이지 디렉토리 인덱스, 페이지 테이블 인덱스, 페이지의 오프셋` 으로 사용되게 된다.

예를 들어, `004637E9` 라는 가상주소가 있다.
이러한 32bit 주소를 `10bit, 10bit, 12bit` 로 나누면
`1, 99(dec), 0x8E9`가 된다.

이것을 가지고 치트엔진을 사용하여 변환 해보도록 하자!
`Kenrel tools → Paging`에 들어가 위의 정보를 가지고 찾아 들어가는 이미지 파일이다.

<img src="https://user-images.githubusercontent.com/40850499/42355759-a6b12ca2-8109-11e8-9650-3ae28cc0add6.PNG" style="zoom:100%" />
<img src="https://user-images.githubusercontent.com/40850499/42355761-a946a2c6-8109-11e8-9791-9285ede9fe5f.PNG" style="zoom:100%" />
<img src="https://user-images.githubusercontent.com/40850499/42355762-aae95b8c-8109-11e8-97b2-11d77df7e9f6.PNG" style="zoom:100%" />
<img src="https://user-images.githubusercontent.com/40850499/42355764-ac0584d2-8109-11e8-8c25-92a835fecdac.PNG" style="zoom:100%" />

```
참조:
주소를 직접 따라가게 되면(메모리 뷰로) 페이지 오프셋 만큼(4KB = 2^12) 하위 비트를 0으로 AND연산 해주어야 한다.
```

최종적으로 나온 물리메모리 주소인 `20CF4A000`에 페이지의 오프셋 값인 `8E9` 를 더해주게 되면 최종 물리주소가 나오게 된다.
