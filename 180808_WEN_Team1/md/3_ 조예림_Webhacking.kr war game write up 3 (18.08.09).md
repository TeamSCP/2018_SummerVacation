# Webhacking.kr war game write up 3 (18.08.09) #

[Webhacking.kr](Webhacking.kr) 에서 16번 



## 1. Webhacking 16번 

문제를 먼저 봅시다!!
![16-1](https://user-images.githubusercontent.com/40850499/44003902-74e1aeaa-9e95-11e8-8c20-95afe7aa2f58.PNG)


이렇게 큰 * 과 작은 * 만 나와있다

저희 집 댕댕이가 노트북 밟고 지나가는 바람에 놀랬는데 갑자기 모니터가 다음과 같이 달라졌다.
![16-22](https://user-images.githubusercontent.com/40850499/44003905-7e52f3f4-9e95-11e8-9680-b2fa83ee2d59.PNG)

없던 * 들이 생겨났다..! 그래서 어떻게 했는지 찾아보다가 아무 키보드를 눌러보니까 *들이 생겨나는 것을 확인 할 수 있었다. 

왜 저렇게 되는지 소스코드를 확인을 해보자

![16-3](https://user-images.githubusercontent.com/40850499/44003908-8b12b700-9e95-11e8-85a8-532ec6228b9a.PNG)


보라색 박스가 중요한 부분인 것 같다. 분석을 해보자면

일단, mv 함수를 선언 해주고 cd 값을 mv 함수에 입력을 해준다.

그리고 kk (별 모양 만들어주는 함수)를 호출 한 뒤 위치를 정해준다.

만약 cd 가 100 이면 위치를 왼쪽으로 +50 만큼 이동 , cd가 97이면 위치를 왼쪽에서 -50 이동,

cd가 119이면 위치를 위쪽으로 -50 만큼 이동, cd가 115이면 위치를 위쪽으로 +50 이동을 한다.

그리고 마지막으로 cd 값이 124이면 어떤 페이지로 이동한다고 분석을 했다.

여기서 100,97,119,115,124 숫자들은 아스키 코드표라고 생각이 되어 아스키 코드표를 보았다.
![default](https://user-images.githubusercontent.com/40850499/44003912-939e3692-9e95-11e8-9088-7f63f26350f3.PNG)


아스키 코드표에서 100은 d를 의미를 한다. 

그러면 우리가 필요한 숫자 124가 어떤 것인지 찾아보니 | (파이프) 를 의미한다.

그러면 | (파이프) 를 입력을 해보면 어떻게 될까?

![16-4](https://user-images.githubusercontent.com/40850499/44003915-9dd49520-9e95-11e8-96dc-1defde3a604f.PNG)


| ( 파이프) 를 입력해보니 패스워드가 나왔다!!!!

Auto에 들어가서 flag 값을 입력을 하면 
![default](https://user-images.githubusercontent.com/40850499/44003919-a95aaf74-9e95-11e8-8a2d-925aede7a0e2.PNG)




짜란 성! 공! 



------



 ## 2. Webhacking 24번 ##

문제를 봐요,,!
![24](https://user-images.githubusercontent.com/40850499/44003922-b37c04da-9e95-11e8-9015-5ea77c5f3c7b.PNG)

내 ip와 브라우저 정보가 나와있다.. 

일단 페이지 소스를 확인 해보면 주석 처리가 되어있다.
![24-1](https://user-images.githubusercontent.com/40850499/44003927-bae4cb6c-9e95-11e8-98a9-ede1df36b575.PNG)


본 페이지에서 index.phps 로 들어가보면 주석 처리 되었던 소스를 확인 할 수 있다.

![24-2](https://user-images.githubusercontent.com/40850499/44003931-c2180106-9e95-11e8-8554-802fde282cd3.PNG)




소스를 분석해 본 결과 이 문제를 해결을 하려면 ip를 127.0.0.1로 변환하면 해결이 된다고 나와있다.

하지만 그 위에 if문을 보면 ip가 12, 7. , 0. 이 되면 공백으로 바뀐다고 나와있다.

맨 먼저  ip를 어떻게 변환을 시켜야할까?

위 소스를 보면 REMOTE_ADDR 쿠키 값을 이용해서 변환 시켜야 한다고 한다.

쿠키를 이용해 ip를 만들어야한다.

하지만 이 문제를 해결 하려면 ip값이 127.0.0.1 이 되어야하는데 12, 7. , 0. 이가 되면 공백으로 변한다. 

어떻게 하면 변하지 않고 해결할 수 있을지 생각을 잘 하시고 !

공백으로 변하지 않게 만들어준 후, 

REMOTE_ADDR 쿠키를 생성해주고 만들어준 ip를 써준 다음 확인을 눌러준다.
![default](https://user-images.githubusercontent.com/40850499/44003943-cee29e00-9e95-11e8-980b-a0e03f28f2c0.PNG)



그렇게 한 다음 새로 고침을 해주게 되면

![default](https://user-images.githubusercontent.com/40850499/44003946-d750c530-9e95-11e8-9049-0279b52a38a5.PNG)


짜란 해 ! 결 !







