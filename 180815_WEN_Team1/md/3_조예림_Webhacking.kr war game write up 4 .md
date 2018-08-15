# Webhacking.kr war game write up 4 (18.08.16) #

[Webhacking.kr](http://webhacking.kr) Wargame 이번주는 15번 , 17번, 25번을 풀었습니다.



## 1. Webhacking.kr 15번 ##

이 문제는 굉장히 쉽게 해결 한 문제이다.

 일단 문제를 보면

![15](https://user-images.githubusercontent.com/40850499/44166241-d24d7480-a105-11e8-949a-a783fd38f1be.PNG)


들어가는 순간 Access_Denied (접근 불가) 가 뜹니다.

 저 상태에서 페이지 소스를 보려고 해도 뜨지 않고 확인을 누르면 다시 문제 푸는 곳으로 되돌아가게 됩니다.

저렇게 뜨는 순간 Esc (강제 종료)를 누르면

![kakaotalk_20180815_233458292](https://user-images.githubusercontent.com/40850499/44166252-db3e4600-a105-11e8-8a06-661d038e3531.png)

바로 패스워드가 나와요..!

그래서 쉽게 클. 리. 어 !

( 문제 해결 된 창을 캡쳐 못했어요 ㅠㅠ)



------





## 2. Webhacking.kr 17번 ##

이번 문제는,, 
![17](https://user-images.githubusercontent.com/40850499/44166266-e3968100-a105-11e8-8d96-d29e3a312d3e.PNG)


이렇게 폼이랑 체크만 있고 아무 것도 나와있지 않다.

이렇게 저 폼에 admin,guset,1234 를 넣어보면 Wrong 이라고 뜬다.
![17-2](https://user-images.githubusercontent.com/40850499/44166279-ec875280-a105-11e8-9adb-06402a84787a.PNG)


>대표적으로 admin 으로 했을 때를 캡쳐한 것.

페이지 소스에서 분홍 박스를 중요시 하게 보시면

![17-3](https://user-images.githubusercontent.com/40850499/44166295-fb6e0500-a105-11e8-8415-ef6a1700f73e.PNG)


unlock 값이 저렇게 길게 나와있고,

작아서 안보이시겠지만 다음과 같이 쓰여있습니다.

```php
function sub(){ if(login.pw.value==unlock){ alert("Password is "+unlock/10); }else { alert("Wrong");  }
```

이 소스를 쉽게 해석을 하자면 ,

unlock 값을 저 폼에다 넣으면 Password 값 (unlock 값에서 10을 나눈 값)을 창에 띄워주고 아니면 Wrong을 띄워준다고 저는 해석을 했습니다.

결론은 저 unlock 값을 구해야 이 문제가 해결이 됩니다.



Q . 그렇다면 어떻게 저 값을 구해야할까?????????????

- A . 크롬에 있는 콘솔창을 이용해보쟈!! 



F12를 눌르시고 다음 그림과 같이 unlock 부분을 찾아야 된다.

![kakaotalk_20180816_004214493](https://user-images.githubusercontent.com/40850499/44166304-0759c700-a106-11e8-84b8-af02f316b1c1.png)

그리고 나서 왼쪽 밑 부분 콘솔을 누르신 후,

unlock값을 넣어주고 엔터를 누르게 되면 다음과 같이 노란색 박스에 있는 값이 나옵니다!
![17-5](https://user-images.githubusercontent.com/40850499/44166370-36703880-a106-11e8-8aca-3c25a46002a4.PNG)



저 값을 이제 폼에다 넣어주면

![17-6](https://user-images.githubusercontent.com/40850499/44166317-117bc580-a106-11e8-898e-5e5df25d16b1.PNG)


제가 찾던 Password 값이 나옵니다 .

저 값을 Auth 에가서 flag 값에 넣어주면 !!

![17-7](https://user-images.githubusercontent.com/40850499/44166388-41c36400-a106-11e8-86f8-19a87476917e.PNG)




짜란 해 ! 결 ! 





------





## 3.Webhacking.kr 25번 ##

문제를 보면 리눅스에서 ls -l 명령어를 쓸때 보던 형태가 나온다.. (리알못)

![25](https://user-images.githubusercontent.com/40850499/44166413-50aa1680-a106-11e8-9478-fa1dbd5175e3.PNG)


일단 페이지 소스 코드를 보자.![25-1](C:\Users\Yellme\Desktop\25-1.PNG)

볼게 없다... 아무 것도 나와있는게 없다.

그렇다면 URL 를 중요시하게 봐야한다.
![25-2](https://user-images.githubusercontent.com/40850499/44166431-57d12480-a106-11e8-998e-b0815accba28.png)


url를 보면

```
?file=hello 
```

라고 나와있고  회색 박스에 'hello world' 라고 나와 있는 것을 보니 hello.txt 를 get 방식으로 쿼리를 날려 보여주는 것 같다. 

그럼 index는 볼 것도 없이 이 문제를 해결 하려면 password.php 로 쿼리를 날려서 해결을 하면 되겠다.

그래서 다음과 같이 쳐보았더니 달라지는게 없다,,,,,

```
?file=password.php
```

![25-44](https://user-images.githubusercontent.com/40850499/44166459-661f4080-a106-11e8-868c-7bc140b46705.PNG)


뭐지..? 왜 안달라지지?

그러면 .php 를 빼고 해보자 ! 했는데 역시나 달라지는게 없다..

hello 라는 쿼리만 보냈을 때 hello.txt 가 나오고 , 다른 값을 입력을 했을 때도 hello.txt 가 읽어지고 있으니까  .txt 가 고정되어있다고 생각이 들었다.

더 쉽게 말하자면 .txt 값이 고정이 되어있어서 내가 아무리 password.php 라는 쿼리를 보내도 

서버에서 자체적으로 .txt 을 붙혀 password.php.txt 로 받아드린다는 말이다.

그렇다면 다음과 같이 php 뒤에  NULL값을 넣어 문자열을 끝내게 해줘야한다.

```
?file=password.php%00
```

> 프로그램에서는 문자열을 끝내고 싶을 때 NULL값을 넣어서 끝낸다.
>
> NULL 값을 url 인코딩하면 %00 가 된다.

위에처럼 쿼리를 날리게 되면

![default](https://user-images.githubusercontent.com/40850499/44166483-720b0280-a106-11e8-9e10-5b25a93bce8a.PNG)

password가 나온다 ㅎㅎ

저 password 값을 flag 에 넣어주면
![25-6](https://user-images.githubusercontent.com/40850499/44166501-7b946a80-a106-11e8-85e9-ed715a9deda7.PNG)


 

짜란 해 ! 결 !

