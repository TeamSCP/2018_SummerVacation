stack based overflow (저번 파일에 이어서 올리겠습니다.)
(Corelan팀의 Exploit writing tutorial part 1 : Stack Based Overflows를 기반으로
작성한 문서입니다.)



<br>

![](https://user-images.githubusercontent.com/37801624/44163614-dfb33080-a0fe-11e8-9729-6686ee3c0ea0.PNG)


<br>

windbg -I 옵션으로 windbg를 postmodern명령어로 등록해줍니다.
<br>(자동으로 crash dump파일을 열어줌)

<br>
<br>

![](https://user-images.githubusercontent.com/37801624/44163618-e2ae2100-a0fe-11e8-8f63-6b07d86e7fdf.PNG)


<br>
<br>

![](https://user-images.githubusercontent.com/37801624/44163619-e3df4e00-a0fe-11e8-8045-7939cbdfe89a.PNG)

<br>

저번 실습에서 A를 30000개 넣으면 오류가 발생했었습니다.

<br>
<br>
<br>

![](https://user-images.githubusercontent.com/37801624/44163621-e5107b00-a0fe-11e8-9db4-3a5817387aba.PNG)
<br><br>
그러면 다음과 같은 창이 뜨는데, eip값이 41414141(=AAAA)로 덮인것을 볼 수 있습니다.
<br><br><br>
![](https://user-images.githubusercontent.com/37801624/44163625-e5a91180-a0fe-11e8-8f87-2953d39fcecf.PNG)
<br><br>
스택에서의 eip위치를 찾기위해서 A 25000개와 B 5000개를 생성하는 perl스크립트를 작성해줍시다.
<br><br>
<br>
![](https://user-images.githubusercontent.com/37801624/44163627-e6da3e80-a0fe-11e8-88fe-b74ad8922152.PNG)
<br>
<br>
이번에는 eip값이 42424242(=BBBB)로 덮인것을 볼 수 있습니다. 그러므로,
eip는 25000~30000사이에 위치한다는 것을 알 수 있습니다.
<br>
<br>
<br>
![](https://user-images.githubusercontent.com/37801624/44163632-e80b6b80-a0fe-11e8-969a-57f3fab00fce.PNG)
<br>
<br>
eip의 정확한 위치를 알기위해서 Kail Linux에 있는(위치 = /usr/share/metasploit_framework/tools/exploit) pattern_create.rb 를 이용해서
5000개의 문자열을 만들어주도록 합시다.
<br>
<br>
<br>
![](https://user-images.githubusercontent.com/37801624/44163635-e9d52f00-a0fe-11e8-854c-83265c7257dd.PNG)
<br>
<br>
만들어진 문자열을 25000개의 A뒤에 넣어주도록 합시다.
<br><br>
<br>
![](https://user-images.githubusercontent.com/37801624/44163639-eb9ef280-a0fe-11e8-8756-e2d657fbcb1b.PNG)
<br>
<br>
그러면 eip가 42336a42로 덮인것을 볼 수 있습니다.
<br>
<br>
<br>
![](https://user-images.githubusercontent.com/37801624/44163644-ee014c80-a0fe-11e8-8b39-656ee4575dc7.PNG)
<br>
<br>
다시 Kali Linux로 돌아가서 같은 위치에 있는 pattern_offset을 이용해서 42336a42의
offset을 파악합니다.
<br>
1059가 나왔으므로, A 25000 + 1059이므로 eip의 위치는 26059임을 알 수 있습니다.
<br>
<br>
<br>
![](https://user-images.githubusercontent.com/37801624/44163649-f35e9700-a0fe-11e8-99a6-40aac9261d32.PNG)
<br>
<br>
![](https://user-images.githubusercontent.com/37801624/44163647-f194d380-a0fe-11e8-9dcf-6d86b95c3073.PNG)
<br>
<br>
다음과같은 코드를 입력해주면 성공적으로 eip가 42424242(=BBBB)로 덮인것을 확인할 수 있습니다. 우리는 이제 eip값을 제어할 수 있습니다.
<br>
<br>
<br>
![](https://user-images.githubusercontent.com/37801624/44163662-f8bbe180-a0fe-11e8-8ee9-e15fb195206f.PNG)
<br>
<br>
![](https://user-images.githubusercontent.com/37801624/44163656-f6598780-a0fe-11e8-953b-3f7eb24cfdf0.PNG)
<br>
<br>
eip값을 BBBB로 덮고 C를 1000개정도 넣어주고, esp값을 확인해보면 esp의 스택값이
43(=C)로 덮인것을 확인할 수 있습니다.
<br>
<br>
<br>
![](https://user-images.githubusercontent.com/37801624/44163664-fbb6d200-a0fe-11e8-9962-21e24db430ec.PNG)
<br>
<br>
esp의 위치도 확인해보기 위해서 다시 144개의 문자열을 생성해주고
<br>
<br>
<br>
![](https://user-images.githubusercontent.com/37801624/44163665-fd809580-a0fe-11e8-8608-b60fa1339d13.PNG)
<br>
<br>
eip값 뒤에 넣어주면
<br>
<br>
<br>
![](https://user-images.githubusercontent.com/37801624/44163667-ff4a5900-a0fe-11e8-943c-d1cb458df403.PNG)
<br>
<br>
5번째 문자열부터 들어가는 것을 확인할 수 있습니다.
<br>
<br>
<br>
![](https://user-images.githubusercontent.com/37801624/44163703-0e310b80-a0ff-11e8-81cc-248f09a9277d.PNG)
<br>
<br>
그러면 eip에 esp의 주소값을 넣고 shell코드로 break문을 넣어주면
프로그램이 정상종료 되지 않을까라는 생각을 해볼 수 있다.
<br>
<br>
<br>
![](https://user-images.githubusercontent.com/37801624/44163705-0ffacf00-a0ff-11e8-9c53-1eea8b40f0c8.PNG)
<br>
<br>
는 실패.<br>
000ffd38은 문자열 종료자인 null바이트를 가지고 있기 때문에 A값은 첫번째 버퍼 뒤에 저장된다고 한다. 따라서 esp의 직접적인 주소값을 이용하는 것은 좋은 방법이 아니다.
<br>
<br>
<br>
![](https://user-images.githubusercontent.com/37801624/44163670-01141c80-a0ff-11e8-86e0-4ea3da04aade.PNG)
<br>
<br>
windbg를 켜서 RM2MP3Converter.exe에 attach한뒤,
<br>
<br>
<br>
![](https://user-images.githubusercontent.com/37801624/44163672-02dde000-a0ff-11e8-8517-52058aed03aa.PNG)
<br>
<br>
a를 입력하고 엔터,
후에 jmp esp를 입력해주면
<br>
<br>
<br>
![](https://user-images.githubusercontent.com/37801624/44163676-040f0d00-a0ff-11e8-95cc-053a3411c10d.PNG)
<br>
<br>
jmp esp의 operation code가 ffe4라는 것을 알 수 있습니다.
<br>
<br>
<br>
![](https://user-images.githubusercontent.com/37801624/44163682-05d8d080-a0ff-11e8-806c-9ab1966d0ade.PNG)
<br>
<br>
이제 operation code인 ffe4를 다음 dll중에서 찾아야합니다.(.....)
<br>
<br>
<br>
![](https://user-images.githubusercontent.com/37801624/44163685-07a29400-a0ff-11e8-83b0-f0cf0a52ba9d.PNG)
<br>
<br>
s 함수범위 함수범위 ff e4를 입력해주면
<br>
<br>
<br>
![](https://user-images.githubusercontent.com/37801624/44163692-08d3c100-a0ff-11e8-836f-7aaf11b8b6ff.PNG)
<br>
<br>
jmp esp의 명령어가있는 주소값이 나옵니다.
<br>
<br>
<br>
![](https://user-images.githubusercontent.com/37801624/44164439-04100c80-a101-11e8-8154-66f48d933a96.PNG)
<br>
<br>
이제 다시 eip에 jmp esp의 명령어가 있는 주소값을 덮어씌우고, esp에 원하는 쉘코드값을 넣으면(저는 계산기 shell코드를 넣었습니다.)
<br>
<br>
<br>
![](https://user-images.githubusercontent.com/37801624/44163700-0c674800-a0ff-11e8-9b47-de9063d8f376.PNG)
<br>
<br>
짜자잔 계산기를 띄워서 첫 exploit을 완성했습니다.
