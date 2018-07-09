# reversing.kr - ransomware

packing 암호가 걸려있는 파일을 unpacking 한후 리버싱을이용해 풀기 



## 다운로드

ollydbg - http://www.ollydbg.de/

HxD - http://hobbang.net/file/412

PEID -  http://www.softpedia.com/get/Programming/Packers-Crypters-Protectors/PEiD-updated.shtml

UPX - https://upx.kr.uptodown.com/windows/download

## HxD

![3](https://user-images.githubusercontent.com/40848203/42411799-4b397a7a-823d-11e8-8f5b-707e04533349.png)

HxD를이용해 file을 분석해보면 93 9A 8B 8C 8F 93 9E 86 9C 97 9A 8C 8C 가 반복된걸 확인할수있습니다.

(key 값이 13글자인걸 확인할수있다.)

 

## PEID 

![1](https://user-images.githubusercontent.com/40848203/42411244-c0ebdac4-8233-11e8-8194-e80aeee3e39d.PNG)

PEID 를 통해 UPX 패커로 패킹이돼있는걸 알수있다.

```
패킹에 잠시설명
1.특정 파일의 크기를 줄여 배포용으로 사용한다.(압축)
2.디버깅을 조금더 어렵게 만들어 준다(데이터를 보호 --> 안티 디버깅)
3.중요한 데이터 정보를 감춰 준다.
```



## UPX 

![2](https://user-images.githubusercontent.com/40848203/42411580-5d906368-8239-11e8-9d0e-d5b40bf45937.PNG)

UPX 를 이용해 패킹된 run.exe를 언패킹 해줍니다.

## ollydbg

```assembly
0044A775  |.  68 ACC14400   PUSH run.0044C1AC                        ; /format = "Key : "
0044A77A  |.  FF15 B0C04400 CALL DWORD PTR DS:[<&MSVCR100.printf>]   ; \printf
0044A780  |.  83C4 04       ADD ESP,4
0044A783  |.  E8 7868FBFF   CALL run.00401000
0044A788  |.  68 70D34400   PUSH run.0044D370
0044A78D  |.  68 B4C14400   PUSH run.0044C1B4                        ; /format = "%s"
0044A792  |.  FF15 B8C04400 CALL DWORD PTR DS:[<&MSVCR100.scanf>]    ; \scanf
```

언패킹된 run.exe 파일을 올리디버깅을 이용해 디버깅 해보면 밑에 키입력하라는 부분이있습니다.



```assembly
0044A8A5  |> /8B45 F8       /MOV EAX,DWORD PTR SS:[EBP-8]
0044A8A8  |. |83C0 01       |ADD EAX,1
0044A8AB  |. |8945 F8       |MOV DWORD PTR SS:[EBP-8],EAX
0044A8AE  |> \8B4D F8        MOV ECX,DWORD PTR SS:[EBP-8]
```

내려보면 입력한 key 값과 가지고있던정보를 이용하여 복호화시키는 코드이고 여기서 EAX에 한바이트를넣고 

그걸 ECX가 불러들여 ECX 값에 한바이트 를 저장한다.



```assembly
0044A8B9  |.  0FBE8A B81554>|MOVSX ECX,BYTE PTR DS:[EDX+5415B8]
0044A8C0  |.  8B45 F8       |MOV EAX,DWORD PTR SS:[EBP-8]
0044A8C3  |.  33D2          |XOR EDX,EDX
```

EDX 값에 입력한 key 값을 저장합니다.



```asm
0044A8C8  |.  0FBE92 70D344>|MOVSX EDX,BYTE PTR DS:[EDX+44D370]
0044A8CF  |.  33CA          |XOR ECX,EDX
0044A8D1  |.  8B45 F8       |MOV EAX,DWORD PTR SS:[EBP-8]
```

ECX 와 EDX 를 XOR 연산후 저장합니다.

```assembly
0044A8E2  |.  0FBE91 B81554>|MOVSX EDX,BYTE PTR DS:[ECX+5415B8]
0044A8E9  |.  81F2 FF000000 |XOR EDX,0FF
0044A8EF  |.  8B45 F8       |MOV EAX,DWORD PTR SS:[EBP-8]
```

저장된 값을 0FF 값을 XOR 연산한후 저장 시킵니다.

```
정리)파일 한바이트 XOR 키 한바이트 XOR OFF = 현재 저장된 데이터
```

## C -XOR 연산코딩

```c
#include<stdio.h>

int main(){
    int decode[] = {0xF0,0xFB,0xE5,0xE2,0xE0,0xE7,0xBE,0xE4,0xF9,0xB7,0xE8,0xF9,0xE2,0x00};
    char *origin = "cannot be run";
    printf("key:");
    char *keys=(char*)malloc(sizeof(char)*14);
    for (int j=0;j<14;++j){
        for(int key = 0x00; key <0xff; key++){
            if((char)((decode[j] ^ key) ^ 0xff) == origin[j]){
                printf("%c",key);
            }
        }
    }
    getchar();
    return (0);
}

```
