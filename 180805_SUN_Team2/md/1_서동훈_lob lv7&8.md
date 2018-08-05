L.O.B
----
###darkelf -> orge

![Alt text](https://user-images.githubusercontent.com/37978105/43688090-82f77934-991d-11e8-8492-3d5091174636.png)

orge의 소스코드이다.

위에서부터 보면


>
>argc값이 2를넘겨야된다
>
>argv[0]길이가 77 여기서 argv[0]실행파일경로가77자리가되어야한다
>
>egghunter는환경변수 사용못하게하고
>
>argv[1][47] !='\xbf'는 첫번쨰 매개변수 문자열의 48번위치가 \xbf가 되어야한다
>
>첫번쨰매개변수의길이가 48을넘기면안된다

>buffer hunter는 버퍼를 0으로 초기화한다


우선 첫번쨰 매개변수는 사용하지못하기때문에 두번쨰매개변수에 쉘코드를 저장하고 RET에 그주소를 입력하는 형식으로 할것이다.

이를 위해서는 argv[0]을 길이를 77로 만들필요가있다 그러기 위해 다음과 같은 방법을 사용할 것이다
![Alt text](https://user-images.githubusercontent.com/37978105/43688204-ad75a7ba-991f-11e8-8f74-b8f23de25bf3.png)

위 사진을 보면 두번쨰 실행에 /를 다중으로 입력을해도 똑같이 실행을 한다는것을 알수있는대 이는 .(dot)이 현재 디렉터리를 가리키기 떄문에 같은 결과가 나온것을 알수있다 즉 .과orge를 제외한 72를 /로 입력을하면 argv[0]의 조건을 채울수 있을것이다.

![Alt text](https://user-images.githubusercontent.com/37978105/43688252-460c0c1c-9920-11e8-9a3a-1bafa68f2efb.png)

모든 조건을 만족시킨것을 볼수있다

두번쨰 매개변수에 주소를 알아내기위해 아무값을 넣고 실행을하였다

![Alt text](https://user-images.githubusercontent.com/37978105/43688277-aa91ccda-9920-11e8-9131-75c8b4b8d387.png)

여기서 A=(NOP 100+ shellcode 25)

![Alt text](https://user-images.githubusercontent.com/37978105/43688303-3230f01c-9921-11e8-8055-55c2807427e8.png)

두번쨰 매개변수의 주소인것을 알아냈으니 RET 에넣으면 끝이다


![Alt text](https://user-images.githubusercontent.com/37978105/43688326-89ab4216-9921-11e8-83f9-a95d98b1f04f.png)

-----------
###orge -> troll

![Alt text](https://user-images.githubusercontent.com/37978105/43688368-57c1a96a-9922-11e8-9c60-967b5303c78d.png)

troll의 소스코드이다

>
아까와 유사하지만 다른점이있다면 우선 첫번쨰매개변수까지 사용하게 되었고

>버퍼와 마찬가지로 첫번쨰매개변수도 저장된값을 0으로 초기화시킨다

그러니 argv[0]에 쉘코드를 저장시켜서 그주소를 RET에 넣으면 될것이다
실행경로인 argv[0]에 쉘코드를 저장시키위해 심볼릭 링크를 이용할것이다.

![Alt text](https://user-images.githubusercontent.com/37978105/43688646-a17d6440-9927-11e8-9122-d5fc665cb691.png)

쉘코드를 심볼릭링크를 걸어준다 이때 쉘코드안에 **0x2f**가 없는 쉘코드를 써야한다 **0x2f**는 **/**로 인식되어버리기떄문이다.

![Alt text](https://user-images.githubusercontent.com/37978105/43688671-1c92d962-9928-11e8-94a5-4c587db41646.png)

심볼릭 링크를 걸면 `ll`로 확인할수있다

![Alt text](https://user-images.githubusercontent.com/37978105/43688678-4647a4a4-9928-11e8-8c6b-75b6e1a45e96.png)

core dump시켜 argv[0]의 주소를 알아낸다

![Alt text](https://user-images.githubusercontent.com/37978105/43688691-764dc642-9928-11e8-93e6-68ae3757c9f8.png)


이제 주소를 RET에 넣으면 쉘을 얻을수있다.

![Alt text](https://user-images.githubusercontent.com/37978105/43688704-a9080912-9928-11e8-8b84-bd3c72e6fcaa.png)

(argv[0]까지 알아낸것은 따로 tmp로 복사한 torll파일에 심볼릭을 걸은후 진행한것이므로 기존의 troll또한 같은방법으로 심볼릭링크를 걸고 진행하면된다)

끄읕
