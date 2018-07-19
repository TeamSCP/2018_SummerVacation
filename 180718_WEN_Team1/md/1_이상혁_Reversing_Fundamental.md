Reversing Fundamental

20180718 이승혁

올해 1월 1일부터 하던 내용이지만 완성이 되지 않아서 다시 한번 발표하려고 합니다.



- 목 차 -

I.Structure

II.Register

III.Stack

IV.Potable Executive

V.Assembly

VI.Reversing

---

I.Structure

Computer Basic Component
<img width="1401" alt="screen shot 2018-07-18 at 9 57 25 pm" src="https://user-images.githubusercontent.com/40850499/42896424-d19e2f30-8af7-11e8-9e41-b10d2480df81.png">
컴퓨터의 기본적인 구성 요소는 위와 같이 CPU, Memory, Hard Disk입니다. 

실행 파일(.exe)은 기본적으로 하드 디스크에 저장되며 윈도우에서는 PE(Portable Executable)라고 부릅니다. PE 파일에는 프로그램을 실행하는데 필요한 기본 정보와 파일을 메모리 어디에 저장해야 할 지 알려주는 배치 정보가 들어있습니다. PE 파일은 헤더(Header)와 바디(Body)로 구성되는데 헤더에는 앞서 말한 중요 정보들이 들어있고, 바디에는 코드와 데이터가 들어있습니다. PE에는 exe, dll, ocx 등 다양한 종류가 있지만 여기서는 exe만 다루도록 하겠습니다.

PE 포맷(헤더 + 바디)으로 구성된 실행 파일을 클릭하면 운영 체제에 있는 로더(Loader)가 헤더에 있는 정보를 분석해 PE 바디에 있는 코드와 데이터를 메모리에 배치합니다. 메모리는 프로그램 코드가 들어가는 코드 영역, 정적 변수와 전역 변수가 들어가는 데이터 영역, 함수 호출 시 사용되는 매개 변수와 지역 변수가 저장되는 스택 영역 그리고 동적 메모리  할당에 사용되는 힙 영역으로 구성됩니다. PE 파일이 메모리에 로딩 될 때는 코드 영역과 데이터 영역에 자료가 들어갑니다. 프로그램이 실행되면서 스택 영역과 힙 영역에 데이터가 쌓이게 됩니다.

PE 파일의 시작 지점은 정해져 있는데 그 지점을 엔트리 포인트(Entry Point)라고 부릅니다. 운영체제는 메모리에 있는 PE 파일을 실행시키기 위해 PE 헤더의 정보에서 엔트리 포인트 위치를 찾아 그곳부터 프로그램을 시작합니다.

프로그램 안의 명령어를 실행하는 것은 CPU 안의 제어장치(Control Unit)와 연산장치(ALU)입니다. 하지만 제어장치와 연산장치가 프로그램을 실행하는 데 필요한 데이터는 모두 메모리 안에 있어야 합니다. 따라서 메모리 안의 데이터를 레지스터로 복사하는 과정이 필요합니다.

컴퓨터는 하나의 CPU를 가지고 동시에 여러 가지 프로그램을 실행시켜야 하기 때문에 여러 프로그램이 번갈아가며 CPU를 사용합니다. 각 프로그램들은 CPU 사용이 끝나면 다른 프로그램에 사용 권한을 넘겨주며 자신이 사용하던 모든 레지스터를 메모리 영역으로 복사해둡니다. 이것을 컨텍스트 스위칭(Context Switching)이라고 합니다.



---

II.Register

Intel x86 CPU(IA-32)

레지스터는 CPU에서 사용하는 고속기억장치 입니다. CPU는 연산을 수행하기 위해 메모리에 있는 데이터를 CPU 내부에 있는 레지스터를 가지고 오며 연산 중간에도 레지스터에 데이터를 저장합니다.

Intel x86 CPU의 기본 구조인 IA-32 아키텍처에서는 프로그램에서 사용하는 EAX, EBX, ECX, EDX, ESI, EDI, EBP, ESP 레지스터와 운영체제에서 사용하는 EIP. 이렇게 9개의 범용 레지스터를 제공합니다. 이외에도 다양한 레지스터를 제공하고 있지만, 리버싱을 할 때는 범용 레지스터를 중점적으로 살펴봐야 합니다. 
<img width="1026" alt="screen shot 2018-07-18 at 9 58 24 pm" src="https://user-images.githubusercontent.com/40850499/42896425-d1ffdd84-8af7-11e8-9ab6-bac5f34d8845.png">
EAX(Extended Accumulator Register)

 곱셈과 나눗셈 명령에서 사용되며, 함수의 반환값을 저장합니다.



EBX(Extended Base Register)

ESI나 EDI와 결합해 인덱스에 사용됩니다.



ECX(Extended Counter Register)

반복 명령어를 사용할 때 반복 카운터를 저장합니다. ECX 레지스터에 반복할 횟수를 지정해 두고 반복 작업을 수행합니다.



EDX(Extended Data Register)

EAX와 같이 사용되며 부호 확장 명령 등에 활용됩니다.



ESI(Extended Source Index)

데이터를 복사하거나 조작할 때 소스 데이터 주소가 저장됩니다. ESI 레지스터가 가리키는 주소에 있는 데이터를 EDI 레지스터가 가리키는 주소로 복사하는 용도로 많이 사용됩니다.



EDI(Extended Destination Index)

복사 작업을 할 때 목적지 주소가 저장됩니다. 주로 ESI 레지스터가 가리키는 주소의 데이터가 복사됩니다.



EBP(Extended Base Pointer)

하나의 스택 프레임의 시작 주소가 저장됩니다. 현재 사용되는 스택 프레임이 살아있는 동안 EBP의 값은 변하지 않습니다. 현재 사용한 스택 프레임이 사라지면 이전에 사용되었던 스택 프레임을 가리키게 됩니다.



ESP(Extended Stack Pointer)

하나의 스택 프레임의 끝 지점 주소가 저장됩니다. PUSH, POP 명령어에 따라서 ESP의 값이 4바이트씩 변하게 됩니다.



EIP(Extended Instruction Pointer)

다음에 실행할 명령어가 저장된 메모리 주소가 저장됩니다. 현재 명령어를 모두 실행한 다음에 EIP 레지스터에 저장된 주소에 있는 명령어를 실행합니다. 실행 전에 EIP 레지스터에는 다음에 실행해야 할 명령어가 있는 주솟값이 저장됩니다.



E

32비트 레지스터는 용도에 따라서 8비트 단위로 나누어 사용할 수 있습니다. EAX 레지스터는 하위 16비트만 AX라는 이름으로 사용할 수 있습니다. 또한 8비트 단위로 상위 8비트는 AH 레지스터, 하위 8비트는 AL 레지스터라는 이름으로 사용할 수 있습니다.

---

III.Stack

Stack and Stack Frame









---

IV.Portable Executable

Basic Concept of PE File















---

V.Assembly

SEQUENCE





---

VI.Reversing

SEQUENCE


