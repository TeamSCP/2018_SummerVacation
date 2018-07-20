 SQL Injection
================

##### 1. = 1 or 1 취약점
##### 2. union
--------
## 1. = 1 or 1 취약점

간단하게 실습을 해본 후, 웹해킹 실습용 웹페이지에 적용해보자<br><br><br>
![](https://user-images.githubusercontent.com/37801624/42921493-3c74b5fe-8b57-11e8-813e-b26ba1c0191f.PNG)
간단한 테이블을 만들어준다.

![](https://user-images.githubusercontent.com/37801624/42921494-3ca3894c-8b57-11e8-98d0-2420996a3ab3.PNG)
<br>이런식의 테이블이 형성된다.
<br><br>
![](https://user-images.githubusercontent.com/37801624/42921496-3cf428ac-8b57-11e8-9b43-1bf9924476c5.PNG)
<br>select문을 이용해서 검색했을 때 id가 1인 값을 검색하게 되면 다음과 같은 결과가 나오게 된다.<br><br>
![](https://user-images.githubusercontent.com/37801624/42921497-3d2ea996-8b57-11e8-8357-9e4d1fbcfa59.PNG)
<br> 다음과 같이 or 1=1을 넣게되면 1=1은 항상 참이므로 or연산에 의해 모든 조건에서 참이된다.
<br><br><br>
다음을 이용해서 실습용 웹페이지에서 실습해보자.<br>
[demo.testfire.net](demo.testfire.net) 이라는 곳이다.



![](https://user-images.githubusercontent.com/37801624/42921503-3e25edf0-8b57-11e8-81b0-57ed44c13942.PNG)
<br>다음과 같은 로그인 창이 있을것이다.
<br>
![](https://user-images.githubusercontent.com/37801624/42921502-3e01180e-8b57-11e8-86c0-814fc9d8eb61.PNG)
<br>우선 Username과 password에 아무거나 입력을하고 로그인을 누르면,<br>

![](https://user-images.githubusercontent.com/37801624/42921501-3dd975c4-8b57-11e8-99b6-fc927b509143.PNG)
<br>다음과 같은 화면이 나온다.
username에 '를 입력했고 password에 d를 입력했으므로<br>
username = ''의 따옴표 사이에 입력한 '가 들어가고 password=''의 따옴표 사이에 입력한 d 가 들어갔다는 것을 알 수 있다.

<br>
![](https://user-images.githubusercontent.com/37801624/42921499-3d86c662-8b57-11e8-9f78-63f80f76d724.PNG)
<br>username에 ' or 1=1; --을 넣으면
'username = ''or 1=1; --'AND password = '입력한 Password값' 이 될텐데<br>
-- 이후는 주석처리가 되어서 'username = ''or 1=1; 만 남을것이다.<br>
항상 참이 되므로 로그인 가능하다.<br><br>
![](https://user-images.githubusercontent.com/37801624/42921504-3e4ba608-8b57-11e8-9a8a-494f92bb9a94.PNG)
<br>
<hr>
## 2. Union 이용

![](https://user-images.githubusercontent.com/37801624/42921509-3f15d086-8b57-11e8-80f8-bc61931349f0.PNG)
일반유저로 로그인을 한 후, Recent transactions에 들어가게 되면 최근 거래 목록을 볼 수 있다.
<br>여기에 있는 검색창에 마찬가지로 아무거나 넣어주게되면,<br><br>
![](https://user-images.githubusercontent.com/37801624/42921511-3f849336-8b57-11e8-872a-34531c2647e6.PNG)<br>
<br>![](https://user-images.githubusercontent.com/37801624/42921510-3f47de32-8b57-11e8-9d6e-bb411bcb9eb6.PNG)<br>
이런식의 오류 메세지가 뜨게된다.<br>
t.trans_date >= 'After에 입력한 값'이들어가는 것을 알 수 있다.<br>
![](https://user-images.githubusercontent.com/37801624/42921512-3fb2f7bc-8b57-11e8-881c-58925821e646.PNG)<br>
After에 1/5/2014 union select null from users--를 넣어주게되면<br>
t.trans_date >= 1/5/2014 union select null from users--(이후는 주석처리)라는 쿼리가 나오게되는데,<br><br><br>
![](https://user-images.githubusercontent.com/37801624/42921514-4009d640-8b57-11e8-8f7b-a1b04fd7d520.PNG)<br>
다음과 같은 오류가 나는데, union의 쿼리와 테이블의 컬럼수가 일치하지 않는다고 나온다.
union select null from users-- 에서
null의 갯수가 컬럼갯수와 일치할때만 오류가 나지 않으므로 null을 계속 입력함으로써,<br>
 해당 테이블의 컬럼갯수를 알 수 있다.(테이블명은 아직 능력이 미치지 않아서 구글링으로 얻어왔습니다.)
null의 갯수를 계속 늘리다보면 null이 4개일때 오류가 나지 않는것을 확인할 수 있다.<br>
<br><br>
컬럼의 갯수를 얻어왔으니, 각 컬럼의 이름을 얻어오고난 후, (이부분도 구글링을 찾아왔습니다.)
다시한번 union을 이용해서
1/5/2014 union select null, username, password, null from users--(union을 통해서 username과 password값을 출력하게 됩니다.)<br>
 삽입해보면(2번째 컬럼명과 3번째 컬럼명은 구글링을 통해서 얻어왔습니다.)
![](https://user-images.githubusercontent.com/37801624/42939652-82e1d9ba-8b91-11e8-9c53-2d082281e628.PNG)<br>
2번째 필드에 id값과 3번째 필드에 비밀번호 값이 나오는 것을 확인할 수 있습니다.
