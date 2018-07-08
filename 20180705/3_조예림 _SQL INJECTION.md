# SQL INJECTION (18.07.05)

- SQL INJECTION 이란  웹 사이트 보안상의 허점을 이용해 특성 sql문을 보내서 데이터 베이스의 중요 정보나 공격자가 원하는 정보를 가져오는 해킹 기법이다. (쉽게 말해 관리자 권한 가지는 것)

  > sql = DB의 데이터를 다루는 프로그램 언어

  

  ### * sql injection EX ###

  ![KakaoTalk_20180708_221212758](C:\Users\Yellme\Desktop\KakaoTalk_20180708_221212758.png)

  

  ```
  웹 프로그램 sql 실행을 예로 들자면, DB에 학생들의 성적이 담긴 성적 DB가 있다고 가정을 합시다. 오세훈 학생의 성적을 알고 싶을 때 http에서 성적 찾기를 누르게 되면 웹 프로그램에서 "name = getParameter" , "select 국어,영어 from 성적 where 이름 = '오세훈'" 이라는 sql문이 실행이 된다. 그 후 학생들 성적 DB에서 오세훈 학생의 성적을 찾아 결과 데이터로 보여주게 된다. 
  ```

  



![KakaoTalk_20180708_222029269](C:\Users\Yellme\Desktop\KakaoTalk_20180708_222029269.png)

```
이때, 공격자가 sql문을 악의적으로 "select 국어,영어 from 성적 where 이름 = '황선홍'" 으로 바꾸게 되면  황선홍 학생의 성적을 보여달라는 것으로 받아드려 황선홍 학생의 성적 데이터를 결과로 보여주게 된다. 이로써, 학생 본인의 성적만 확인 할 수 있었던 것을 sql 구문을 변경 시켜 다른 사람들의 성적 정보를 조회할 수 있게 된다. 
이것이 바로 간단한 sql 공격의 예인 특정 데이터 조회이다.   
```



### * SQL INJECTION 공격의 종류

1. 특정 데이터 조회
2. 인증 우회
3. 기밀 데이터 접근
4. 웹 사이트 콘텐츠 변경
5. DB 서버 shutdown



## LOS Wargame  ##

[Lord of SQL Injection](https://los.eagle-jump.org/) : sql injaction 의 해킹 기법으로 문제를 푸는 wargame 사이트

에서 1번 문제인 gremlin 문제를 풀었다.

![22394843582D6EE618](C:\Users\Yellme\Desktop\22394843582D6EE618.png)

이 문제는 sql문과 php언어만 알면 굉장히 간단한 문제이다.

문제를 보면  id와 pw 값을 입력 받아 DB에 쿼리를 전송하고,  

id의 결과 값과 내가 입력해주는 id이 참 일때,  이 문제는 해결이 된다. 

일단 id와 pw를 모르게 때문에 우회를 해서 알아내야 하고, GET 메소드를 쓰고 있으니 GET 방식으로 데이터를 url 뒤에 붙혀 보내야한다

> https://los.eagle-jump.org/gremlin_bbc5af7bed14aa50b84986f2de742f31.php?id=
>
> 이런식으로,,,,,,



?id=yell&pw=1234 를 해보니

![ㅇ옐](C:\Users\Yellme\Desktop\ㅇ옐.png)

쿼리에서 id 와 pw가 싱글 쿼터(' ')로 묶여져 있는 것을 확인 할 수 있다.

그리고 id의 결과 값이 참이면 되니까 pw값은 필요가 없으므로 주석 처리를 해준뒤

 OR 연산 1=1 (항상 참) 을 사용해 참이 되게 만들면 된다.

최종적으로 참이 되게 만든 후 url 뒤에 데이터를 보내게 되면 !!!

![옐 답](C:\Users\Yellme\Desktop\옐 답.png)



짜란 ,, 문제를 풀었습니다..!  구문을 어떻게 만들어야 될 지는 잘 생각을 해보세요!! 

