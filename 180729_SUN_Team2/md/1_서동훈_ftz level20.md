











F.T.Z level20
---
![Alt text](https://user-images.githubusercontent.com/37978105/43369520-fd0fa6ec-93aa-11e8-8f9b-7bd403068b16.png)
[그림 1]

![Alt text](https://user-images.githubusercontent.com/37978105/43369521-fe8521a0-93aa-11e8-9008-8d413fc2e96d.png)
[그림 2]

level20을 푸는데 사용할 포맷 스트링을 간단하게 정리하면

### Format String
>일반적으로 사용자로부터 입력을 받아 들이거나 결과를 출력하기위하여 사용하는 형식
![Alt text](https://t1.daumcdn.net/cfile/tistory/2522014854BFBC0125)
>[포맷 인자]

>이러한 Format String 을 이용한 공격기법으로 printf()함수의 취약점을 이용하는방법으로
printf()함수 계열을 사용할떄 문자열일 때는 문자열로 인식하지만 서식문자를 넣을 경우 서식문자로 인식하여 기능하게된다
즉 서식문자를 만나면 메모리 다음 4바이트 위치를 참조하여 그 >서식문자의 기능대로 출력한다는것이다

>간단한 예로들자면

>![Alt text](https://user-images.githubusercontent.com/37978105/43369813-3709a25e-93af-11e8-82c1-e01593d43bc4.png)
>[cr의 소스코드]
>
>![Alt text](https://user-images.githubusercontent.com/37978105/43369818-5c9bffc6-93af-11e8-95f8-0366a2d09b78.png)
>
>입력과 출력이 같은것을 볼수있다.

>반대로는 

>![Alt text](https://user-images.githubusercontent.com/37978105/43369839-ccaf543e-93af-11e8-9cc9-bfa8df13eb6c.png)
>[wr소스코드]
>
>![Alt text](https://user-images.githubusercontent.com/37978105/43369844-d7152598-93af-11e8-870f-e5c7205c7db5.png)


>입력한 문자열을 포맷 스트링으로 인식하여 스택의 값을 출력하는 것을 볼 수 있다.


다시 문제를 풀어보면  [그림 2]를 보면 문자열을 인자로 넣지않고 bleh변수를 바로 넣어 호출한것을 볼 수 있다.

![Alt text](https://user-images.githubusercontent.com/37978105/43370139-b411af62-93b4-11e8-925f-58383ccc38a1.png)

attackme 또한 4번째에 스택의 값을 출력하는것을 볼수있다.

본격적인 공격을위해 gdb로 main의 disassembly보려하지만

![Alt text](https://user-images.githubusercontent.com/37978105/43370162-2a045b70-93b5-11e8-8691-f475a9677c8d.png)

symbol이 없다고한다고나온다

심볼을찾기위해 nm명령어를 사용한다

일반적인명령어를 주고 실행하면

![Alt text](https://user-images.githubusercontent.com/37978105/43370192-7afd7142-93b5-11e8-9cf8-01ae8ba740c0.png)

다 null로나온다 

한번 출력 형식을 바꿔서 출력을해보자

`
nm -f sysv attackme
`

![Alt text](https://user-images.githubusercontent.com/37978105/43370210-06d30baa-93b6-11e8-80b5-ea773159749c.png)


.ctors .dtors가 보이는대 이를 이용한다 .ctors main이 실행되기전에 호출되고 .dtors는 main이 종료되기 직전에 호출된다

즉 이 dtors를 이용하여 문제를 풀것이다.

우선 쉘코드를 환경변수에 등록한다

![Alt text](https://user-images.githubusercontent.com/37978105/43370953-3bf366b0-93c3-11e8-9d77-54052f566138.png)

![Alt text](https://user-images.githubusercontent.com/37978105/43370956-46095e70-93c3-11e8-9660-80c30f50e6f3.png)

![Alt text](https://user-images.githubusercontent.com/37978105/43370959-634e23da-93c3-11e8-9fee-3ce7fe28dd1e.png)

쉘코드의주소는 0xbffffd87이다

![Alt text](https://user-images.githubusercontent.com/37978105/43371143-b9f24e84-93c6-11e8-8f01-f33d23e7ee7c.png)

0x08049594에서 4바이트 더한 값인 0x08049598가 dtors의 주소가 된다

>주소를 덮으려할때 %n 서식자를 이용하면된다 %n같은경우 %n전까지 출력된 문자의 개수를 지정된 변수에 10진 정수 형식으로 쓴다

>![Alt text](https://user-images.githubusercontent.com/37978105/43371263-14be6e7c-93c9-11e8-935c-a38cecc4cbde.png)[expl.c]

>![Alt text](https://user-images.githubusercontent.com/37978105/43371264-1ca445f8-93c9-11e8-99c7-963c2b9ecc9b.png)


dtors의 주소에 쉘코드 주소를 덮으면 된다
     0x08049598 <- 0xbffffd87

쉘코드 주소를 2바이트씩 정수형으로 변환시키
0xfd87=64903은 쉘코드의 하위주소 값이다 상위주소같은 경우는  상위주소에서 하위주소를뺴면 음수로나오기때문에 한자리증가시킨0x1bfff로만든후 계산을한다 
0x1bfff=114687

쉘코드의 상위주소 값은114687-64903=49784


attackme에서 입력한 문자열이 4번쨰 서식 지정자에 오는걸 알고있으니 $-flag 기법을 이용할것이다 공격 Payload는 다음과같 다

dtors 하위주소(4)+dtors 상위주소(4)+쉘코드 하위주소 (4)+ %4$hn+쉘코드 상위주소(4)+%5$hn

![Alt text](https://user-images.githubusercontent.com/37978105/43371113-300e32be-93c6-11e8-8bf7-55f1de06418b.png)

![Alt text](https://user-images.githubusercontent.com/37978105/43371308-d6dc564a-93c9-11e8-9aac-35369bf509c4.png)

끝
