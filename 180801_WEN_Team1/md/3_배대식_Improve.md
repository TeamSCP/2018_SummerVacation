# :speech_balloon: Improve

UC 포럼의 마지막 글 <a href="https://www.unknowncheats.me/forum/anti-cheat-bypass/261176-silentjack-ultimate-handle-hijacking-user-mode-multi-ac-bypass-eac-tested.html">Stealth-aware handle hijacking</a>을 진행 하기전에 텍스트로만 정리하여 넘어갔던 부분,<br>정확히 하지 않았던 부분들을 커널 디버깅과 함께 이해 해보는 시간을 가질 것 입니다.<br>

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

디버깅 환경은 다음과 같습니다.
```
Windows 7 Kernel Version 7601 MP (1 procs) Free x86 compatible
Built by: 7601.17514.x86fre.win7sp1_rtm.101119-1850
```

핸들로 커널 오브젝트를 구하는 과정은 다음과 같습니다.
1. _EPROCESS → ObjectTable(_HANDLE_TABLE)
2. TableCode → _HANDLE_TABLE_ENTRY
3. _OBJECT_HEADER → Body
4. (_TYPE*)input = _OBJECT_HEADER → Body

핸들의 구조는 다음과 같습니다.

| UNUSED | TOP | MIDDLE | SUB | TAG |
| ------ | --- | ------ | --- | --- |
| UNUSED | 11111 | 1111111111 | 111111111 | 11 |


커널 오브젝트를 구조체로 표현 할 수 있는 방법은 다음과 같습니다.

| File| Process | Token | Driver |
| --- | ------- | ----- | ------ |
|   _FILE_OBJECT   | _EPROCESS     | _TOKEN     | _DRIVER_OBJECT|
