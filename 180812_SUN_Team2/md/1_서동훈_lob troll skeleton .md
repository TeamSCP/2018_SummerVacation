
L.O.B
----
###troll>vampire

![Alt text](https://user-images.githubusercontent.com/37978105/43994273-61f7fa76-9dd5-11e8-94e8-ae4d393bbd37.png)

vampire의 소스코드이다.

이전 문제들과는 다르게 오직 리턴주소에만 제한을두었다
>1. return address 시작은  bf로 되야한다 ex:("0xbfxxxx")

>2. return address 상위 2번쨰 byte에 ff가들어가면 종료시킨다 ex:("0xbfffxx")이면 종료

즉 스택영역안에 2번째조건을 만족한곳에 쉘코드를 저장하면된다 그러기위해서 간단하게 스택영역을 짚고넘어가면

![Alt text](https://user-images.githubusercontent.com/37978105/43994450-effb9f1a-9dd7-11e8-83a5-f7434568d5fa.png)

밑으로갈수록 높은주소이다.

즉 버퍼에 쉘코드를 저장한다고 가정했을때 매개변수에 많은수를 저장하면 버퍼에주소는 점차 낮아질것이며 2번쨰조건을 만족시킬수 있을것이다.

![Alt text](https://user-images.githubusercontent.com/37978105/43994487-89a95b0c-9dd8-11e8-8f99-1f2f724e713b.png)

브레이크 포인터를잡는다 
그리고 매개변수에 A를 14000개넣고 실행해보았다

![Alt text](https://user-images.githubusercontent.com/37978105/43994525-758c1cbc-9dd9-11e8-97c4-150a4fda4e0f.png)
![Alt text](https://user-images.githubusercontent.com/37978105/43994526-78fc806c-9dd9-11e8-8210-6c752d9749ab.png)

아직 상위 2번쨰 가 ff인것을 볼수있다 그러니 이번에는 A를 100000개 넣어서 실행을 해보았다.

![Alt text](https://user-images.githubusercontent.com/37978105/43994514-3f6ee4e8-9dd9-11e8-910a-f9b69aaeec74.png)

그러니 "0xbffexxxx"가 되어있으므로 2번쨰 조건을 만족시킨다
이제 RET주소에 넣어주면 된다

![Alt text](https://user-images.githubusercontent.com/37978105/43994569-0ec753ce-9dda-11e8-8912-1d334012b059.png)

core dumped가 났으니 core를 gdb로열어 정확한 주소를 입력해주자

![Alt text](https://user-images.githubusercontent.com/37978105/43994610-d1a28094-9dda-11e8-92d7-818d6a3e97e6.png)

다시 RET에 주소를넣고 공격을하면

![Alt text](https://user-images.githubusercontent.com/37978105/43994627-2ed7530c-9ddb-11e8-9a54-ee8f8bd6ca7e.png)

쉘을 얻을수있다.

---

###vampire>skeleton

![Alt text](https://user-images.githubusercontent.com/37978105/43994644-6f5b6292-9ddb-11e8-8ddf-c04d20a928ab.png)

skeleton의 소스코드이다.

소스코드를보면
>1.환경변수 사용x

>2.return address의 시작이 bf ex:("0xbfxxxx")

>3.argv[1]의길이가 48이상이면 종료

>4.버퍼 0으로 초기화

>5.모든 argv 0으로 초기화

우선 2번쨰 조건을 보면 우선 쉘코드는 스택영역에 저장이되어야한다

전 문제를 풀때 사용한 스택영역을 간단히 표현한것을 보면 *argv가있는대 argv가 초기화되어도 *argv[0]는 초기화되지 않는다.

![Alt text](https://user-images.githubusercontent.com/37978105/43994817-113545e0-9dde-11e8-85e3-47dddd73c373.png)

argv초기화이후 브레이크 포인트를 잡고
인자를 몇개넣고 실행을해보면

![Alt text](https://user-images.githubusercontent.com/37978105/43994844-9148e14c-9dde-11e8-8ebf-6b8fa480c27e.png)

![Alt text](https://user-images.githubusercontent.com/37978105/43994851-b2de74fc-9dde-11e8-8bd2-5d77f42b0535.png)

argv[0]만 남아있다 즉 심볼릭링크를 이용해 쉘코드를저장하고 저주소를 RET에 넣으면 쉘을 얻을수있다.

![Alt text](https://user-images.githubusercontent.com/37978105/43994933-074dbf1a-9de0-11e8-8d04-45ec8a1f06b7.png)

심볼릭링크를 설정하고

![Alt text](https://user-images.githubusercontent.com/37978105/43994963-bacc65dc-9de0-11e8-8cbb-9c112b672773.png)

core dumped 시킨후 gdb로 core열어 주소를 찾는다

![Alt text](https://user-images.githubusercontent.com/37978105/43994968-e0c06edc-9de0-11e8-8d15-369070e61f58.png)

저주소를 RET에 넣으면 쉘을 얻을수있다.

![Alt text](https://user-images.githubusercontent.com/37978105/43994977-0d83cb30-9de1-11e8-9740-f941b811246b.png)

끄읕
