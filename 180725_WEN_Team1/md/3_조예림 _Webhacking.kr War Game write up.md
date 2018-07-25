# War Game write up (18.07.26) #

[Webhacking.kr](http://webhacking.kr) 으로 워게임 문제를 풀고 write up 을 하려고 합니다.

오늘은 webhacking.kr 38번과 26번을 풀었는데용,,



##  Webhacking.kr 38번 

이문제를 푸시려면 **[ Log Injection](http://unabated.tistory.com/entry/Log-Injection) **를 꼭 보세요!!

![1](C:\Users\Yellme\Desktop\1.PNG)

일단 문제를 보면 Log Injection 이라고 나옵니다. 

 Log Injection이 무엇인지 구글링을 해본 결과!

Log Injection은 sql Injection 처럼 파라미터를 통해 로깅을 조절하는 방법을 말한다고 합니다.

> 로깅 (Logging) : 프로그램 개발이나 운영 시 발생하는 문제점을 추적하거나 운영 상태를 모니터링 하기 위한 텍스트 



다시 문제로 돌아와서 페이지 소스를 살펴보니

![2](C:\Users\Yellme\Desktop\2.PNG)

login 버튼을 누르면 값이 전달되고 admin 값을 누르면 admin.php 로 이동한다는 걸 알게 된다.

그래서 admin을 치고 login을 누르니

![3](C:\Users\Yellme\Desktop\3.PNG)

 난 admin이 아니라고 한다............

그래서 일단 yell 을 치고 login하고 admin에 들어가보니..!

​                       ![4](C:\Users\Yellme\Desktop\4.PNG)

어떠한 ip : yell 이라는 log가 나타남을 확인 할 수 있다.

왜 yell은 되면서 admin은 안될까? 생각을 하면서 admin 앞에 공백을 줘봤다.

\n admin을 해보니

 ![5](C:\Users\Yellme\Desktop\5.PNG)

저렇게 밑에로 내려갔다.  

admin 앞에 어떠한게 있어야된다고 깨닫고  Log Injection에 대해 찾은 블로그를 참고해서 답을 적었더니 다음과 같이 문제를 풀고 100점을 겟또----!!! 

![6](C:\Users\Yellme\Desktop\6.PNG)





------



 ## Webhacking.kr 26번 ##

![21](C:\Users\Yellme\Desktop\21.PNG)

문제를 보면 저렇게 나온다...... 뭐지 하고 index.phps 를 눌러보면

![22](C:\Users\Yellme\Desktop\22.PNG)

소스코드가 보인다.  php 소스코드를 첫줄 부터 분석을 해보면 

```
get 방식으로 id 변수에 저장된 값이 admin 이면 나감.

get 방식의 id 변수에 저장된 값을 urldecode 을 한 후 다시 다시 id 변수에 저장.

만약 id 변수에 저장된 값이 admin 이면 해결.
```

이렇게 분석을 하고 get방식으로 url에 admin을 써보면 eregi 함수로 인해 강제 종료가 된다.

그러면 url 인코딩을 해서 넣어보자!!

admin 을 인코딩하면 **%61%64%6D%69%6E** 이다. 이것을 get 방식으로 url에 넣어주면...

![23](C:\Users\Yellme\Desktop\23.PNG)

이렇게 뜬다.......... 그리고 url를 보면 id=admin이라고 필터링이 된다. 

 왜 그런지 구글링 해보니,, 인코딩 값을 전달해도 서버에 전달이 되서 동시에 서버 php가 해석을 하기 때문이라고 한다..!!!

그러면 한번 더 인코딩을 해준 뒤, get 방식으로 전달을 하게 되면 .........................!!!!

짜라안!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!![24](C:\Users\Yellme\Desktop\24.PNG)





끝 - !

