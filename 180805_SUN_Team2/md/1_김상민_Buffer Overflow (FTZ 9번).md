# Buffer Overflow (FTZ 9번)

8 / 6, 2018 Jinyan 김상민

FTZ 9번 풀이와 풀이에서  사용되는 Buffer Overflow에 대해 이야기해 보려고 합니다.

(스택 구조와 어셈블리어 해석이 있습니다.)



> 환경 : Windows 10
>
> - Red Hat Linux 9.0 (VMware Workstation)
> - putty
>
> 

먼저 Buffer Overflow는 시스템 해킹의 대표적인 공격 방법중 하나 입니다. 버퍼라는 것은 보통 데이터가 저장되는 메모리 공강을 일컫는데 단 순히 메인 메모리만이 아닌 다른 하드웨어에서 사용하는 임시 저장 공간 역시 버퍼라고 불릅니다. 오버플로우는 데이터가 지정된 크기의 공간보다 커서 해당 메모리 공간을 벗어나는 경우사용하게 되는데, 결론적으로 버퍼 공간의 크기보다 큰 데이터를 저장하게 해서 일어나는 오버플로우 공격입니다.

취약점 함수는 strcpy, strcat, gets, fscanf, scanf, sprintf, sscanf, vfscanf, vsprintf, vscanf, streadd, strecpy, strtrns  등이 있습니다.

<img src=https://user-images.githubusercontent.com/37801676/43690405-538bf572-9944-11e8-9415-484825740bc5.jpg>

그럼 FTZ 9번의 문제풀이를 에 대한 설명을 하겠습니다.



#### Hint

<img src=https://user-images.githubusercontent.com/37801676/43689336-db3ed8e2-9933-11e8-82a1-da236527b689.PNG>

힌트를 보았을 때 어떤 소스코드를 보여주고 level10의 권한을 얻으라고 합니다.

[buf2]와 [buf]를 각각 10Byte씩 할당을 해주고 [buf]입력을 40Byte로 할당 해주고 오버플로우가 가능하다고 합니다.

그리고 [buf2]와 비교해서 "go"가 맞으면 Setueid로 level10의 권한을 얻을 수 있는걸 확인할 수 있었습니다.

: 문제는 [buf2]에 go를 넣기 위해서는 buf에 어떠한 값 입력을 먼저 넣어야 했습니다.



먼저 스택 구조를 확인하기 위해서 gdb로 디버깅을 해봐야하는데 힌트에서 나와있는 파일의 권한은 실행 권한 밖에 없었습니다.

<img src=https://user-images.githubusercontent.com/37801676/43689337-db6e361e-9933-11e8-998d-694e34ae0330.PNG>



그래서 힌트에 나와있는 소스를 이용하여 복사하여 새로운 임시디렉터리(tmp)에 bof.c를 만들었습니다.

<img src=https://user-images.githubusercontent.com/37801676/43689338-db99b9f6-9933-11e8-9eca-5da59ad43944.PNG>



그런 다음 소스파일(bof.c)를 컴파일 해주었고

<img src=https://user-images.githubusercontent.com/37801676/43689339-dbc649f8-9933-11e8-9dfc-496fe69daf4d.PNG>



스택구조 - main 함수에 대응하는 어셈블리어 언어를 확인해 보았습니다.

<img src=https://user-images.githubusercontent.com/37801676/43689340-dbf39070-9933-11e8-8c8a-41705213f7ae.PNG>

<img src=https://user-images.githubusercontent.com/37801676/43689341-dc2476c2-9933-11e8-8215-7f174d096952.PNG>

<img src=https://user-images.githubusercontent.com/37801676/43689342-dc5a6246-9933-11e8-9be7-bb254a70b23f.PNG>



1. 먼저 <main+0> : push %ebp를 수행하여 이전 함수의 base pointer를 저장하면 stack pointer는 4Byte 아래를 가리키게 됩니다.
2. <main+1>: mov %esp,ebp를 수행하여 esp값을 ebp에 복사하였습니다. 그래서 esp와 ebp는 함수의 base pointer와 stack pointer이 같은 지점을 가르키게 됬습니다.
3. <main+3>: sub $0x26,%esp를 수행 하면 esp에서 40byte아래 지점을 가리키게 되고 스택에 40byte공간이 생기게 됩니다. ( 40byte가 확정 되었습니다. )
4. <main+6>: and $0xfffffff0,%esp는 11111111 11111111 11111111 11110000 과 AND 연산을 하는데 esp 주소값의 맨 뒤 0으로 만들기 위한 명령어로 의미 없는 명령입니다.
5. <main+9>: mov $0x0, %eax는 eax에 0을 넣어 줍니다.
6. <main+14>: sub %eax,%esp는 esp에 들어있는 값에서 eax에 들어 있는 값만큼 빼줍니다. (stack pointer를 확장 시켜주는 명령이지만 0으로 의미 없음)
7. <main+ 16>: sub %0xc,%esp는 스택을 12Byte 확장 하였습니다.
8. <main+ 19>: push $0x8048554  + <main+24>: call print 부분 
9. <main+ 39>: add $0x10,%esp는 16Byte 스택을 줄여서 이전 위치로 돌아가게 한다.
10. <main+ 35> + <main+41>으로 [buf]의 주소와 40 Byte를 준것을 확인하였습니다. 
11. <main+28> + <main+60>은 "go"라는 문자의 주소를 확인 하였습니다.

<main+43> 이랑 <main+65>를 보았을때 둘 사이의 간격은 16byte를 확인하였습니다.



소스파일을 보았을때 buf의 크기는 각각 10Byte로 정해졌지만 gcc의 버전에따라 달라진다고합니다.

gcc 2.96이후의 버전에서는 스택의 16배수를 할당하기에 10Byte를 주지만 16Byte로 스택이 생성이 되게 됩니다.



어셈블리어를 구조로 그려보았을때의 모습으로 이렇습니다.

<img src=https://user-images.githubusercontent.com/37801676/43690227-a278567e-9941-11e8-9b6d-8c952f3371da.jpg>

즉 (buf)에 16문자를 넣으면 그 다음 문자부터 (buf2)에 들어가게 되서 "go"를 입력하게 되면 level10의 권한을 얻을 수 있게 됩니다.

<img src=https://user-images.githubusercontent.com/37801676/43690248-1e3bfba8-9942-11e8-93cb-7e13b64392f8.PNG>

