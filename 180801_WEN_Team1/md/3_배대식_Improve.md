# :speech_balloon: Improve

UC 포럼의 마지막 가이드 스레드 <a href="https://www.unknowncheats.me/forum/anti-cheat-bypass/261176-silentjack-ultimate-handle-hijacking-user-mode-multi-ac-bypass-eac-tested.html">Stealth-aware handle hijacking</a>를 진행 하기전에 텍스트로만 정리하여 넘어갔던 부분,<br>정확히 하지 않았던 부분들을 커널 디버깅과 함께 이해 해보는 시간을 가질 것 입니다.<br>

## :green_book: Improve What?
- 커널 오브젝트와 오브젝트 핸들 테이블의 정리
- 권한 상승을 통한 PPL Bypass
- C++ and class

## :heavy_check_mark: Handle

A 프로세스와 B 프로세스가 있습니다.<br>
C 프로세스의 핸들을 A와 B가 요청 하였을 때 A와 B의 핸들 값은 같을까요 다를까요?<br>
정답은 같을 수도 있고 다를 수도 있습니다.<br>
왜 그럴까요? 그리고 윈도우는 핸들 값을 가지고 어떻게 커널 오브젝트까지 찾아 갈 수 있을까요?<br>

## :pushpin: More deeper

__\# 디버깅 환경__
```
Windows 7 Kernel Version 7601 MP (1 procs) Free x86 compatible
Built by: 7601.17514.x86fre.win7sp1_rtm.101119-1850
```

__\# 테스팅 환경__<br>
<img src="https://user-images.githubusercontent.com/40850499/43530597-f10477e8-95e8-11e8-8acc-e811a4131fcc.png"/>

__\# 무엇을 확인 해 볼 껀가요?__<br>
- Handle 값의 의미
- 구조체마다 가지고 있는 특수 bit
- 디버깅으로 커널 오브젝트 얻어보기

__\# 전체 과정 요약__<br>
1. _EPROCESS → ObjectTable(_HANDLE_TABLE)
2. TableCode → _HANDLE_TABLE_ENTRY
3. _OBJECT_HEADER → Body
4. (_TYPE*)input = _OBJECT_HEADER → Body

__\# 핸들의 구조__<br>

| UnUsed<2^6> | Top<2^5> | Middle<2^10> | Sub<2^9> | Tag<2^2> |
| ------ | --- | ------ | --- | --- |
| 000000 | 00000 | 0000000000 | 000001000 | 00 |

Tag로 인해 `핸들의 값은 4씩 증가`하게 된다.<br>
이로 인해 _HANDLE_TABLE_ENTRY를 계산 할 때 `(HandleValue/4 * 8)`이 된다.<br>
Sub Table에서 0번째 구조체는 _HANDLE_TABLE_ENTRY 라는 것을 알리므로 커널 오브젝트 주소를 할당 할 수 없다.<br>
즉, 핸들 생성 가능 개수는 `(2^5)*(2^10)*(2^9-1) = 16744448` 이다.<br>

__\# TableCode의 구조와 특성__<br>

`(_HANDLE_TABLE_ENTRY | Level Index)`의 형식을 따른다.<br>
ex) 0x9131ec02 = (0x9131ec00 | 2)<br>

```python
>> Level Index Values
  0 - Sub table
  1 - Middle table
  2 - Top table
```
Level Index = TableCode & 0x3<br>
_HANDLE_TABLE_ENTRY(sub_table) = TableCode & ~2

__\# \_HANDLE\_TABLE\_ENTRY의 구조와 특성__<br>

| _OBJECT_HEADER | AIL Bits |
| ------------- | -------- | 
| 00000000000000000000000000000 | 111 | 

```python
>> LIAP Bits List
  Bit #0  - Lock flag
  Bit #1  - Inherit Flag
  Bit #2  - Admit Flag
  Bit #25 - Protect on close flag 
```
각 비트는 API의 생성시 특정 인자를 이용해 설정 하거나, SetHandleInformation API를 사용하여 설정 해 줄 수 있다.<br>
_OBJECT_HEADER의 포인터로 사용되는 주소는 `(TableCode & ~0x7)`을 해야 확인 할 수 있다.<br>


__\# 윈도우에서 사용하는 커널 오브젝트에 대한 구조체__

| File| Process | Token | Driver |
| --- | ------- | ----- | ------ |
|   _FILE_OBJECT   | _EPROCESS     | _TOKEN     | _DRIVER_OBJECT|

## :cupid: Let's XD
<img src="https://user-images.githubusercontent.com/40850499/43531579-0a3f9434-95eb-11e8-95de-9191da47df1f.PNG"/>

---
:speech_balloon: JOP_H와 JOP_NH는 같은 핸들 값을 가지고 있다. 하지만 전혀 문제 될 것이 없다.<br>

---

<img src="https://user-images.githubusercontent.com/40850499/43531760-768e1444-95eb-11e8-8bab-ba48b6b8ce57.PNG"/>

---
:speech_balloon: 이유는 간단하다. 핸들 값을 참조하는 _HANDLE_TABLE의 주소가 틀리기 때문이다.<br>

---

<img src="https://user-images.githubusercontent.com/40850499/43531946-e524a15c-95eb-11e8-9ff3-ee6bc2e80107.PNG"/>

---
:speech_balloon: 위의 이미지에선 _HANDLE_TABLE의 주소를 바로 구했지만, _EPROCESS->ObjectTable에 위치해 있다.<br>

---

<img src="https://user-images.githubusercontent.com/40850499/43532561-5c62092a-95ed-11e8-9716-baa7c005a9ed.PNG"/>

---
:speech_balloon: _HANDLE_TABLE의 첫 번째 멤버인 `TableCode`는 _HANDLE_TABLE_ENTRY의 첫 번째 주소를 가지고 있다. <br>

---

<img src="https://user-images.githubusercontent.com/40850499/43538417-f475452e-95fc-11e8-8d71-9b9087f20575.PNG"/>

---
:speech_balloon: SubTable의 첫 번째 멤버는 자신이 _HANDLE_TABLE_ENTRY임을 알리는 역할을 하며, 핸들로 사용 될 수 없다.<br>
그러므로 SubTable의 생성 개수는 `2^9 - 1`인 것이다.

---

<img src="https://user-images.githubusercontent.com/40850499/43539581-3b2ab988-9600-11e8-8afb-191398a22162.PNG"/>

---
:speech_balloon: _HANDLE_TABLE_ENTRY는 8Byte의 구조체이고 HANDLE 값은 Tag에 의해 4씩 증가한다.<br>즉, `0x20/4 * 8`를 한 값이 맞는 인덱스가 될 것이다.<br>그 다음 Object의 빨간색으로 표시된 3을 주시해보면 LIA Bit중 Lock과 Inherit 비트가 활성화 되있는 것을 확인 할 수 있다.<br>최대 값을 7까지 가질 수 있으므로 `~7을 And 연산`하면 _OBJECT_HEADER 구조체의 값을 구할 수 있다.

---

<img src="https://user-images.githubusercontent.com/40850499/43540525-f533fda6-9602-11e8-97a0-8b2368fa52f2.PNG"/>

---
:speech_balloon: _OBJECT_HEADER는 Header와 Body로 나뉘는데 Body가 커널 오브젝트의 시작 주소이다.<br>
이미지를 확인해보면 body offset은 0x18이다.<br>
`_OBJECT_HEADER의 시작주소 + 0x18 = 핸들에 대한 커널 오브젝트의 시작주소`일 것이다.

---

<img src="https://user-images.githubusercontent.com/40850499/43540638-424b3cb2-9603-11e8-81e6-535fff6f14a2.PNG"/>

---
:speech_balloon: 길고 길었던 기본적인 핸들 값 찾기가 완료 되었다.<br>
calc.exe 프로세스의 EPROCESS 주소와 핸들 값으로부터 구한 EPROCESS 주소가 일치한다.

---
