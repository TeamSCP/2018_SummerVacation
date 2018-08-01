# Webhacking.kr war game write up 2 (18.08.02) #

이번주에는  [Webhacking.kr](http://webhacking.kr) 1번, 4번, 14번  풀었다.



## 1. Webhacking.kr 4번 ##

![문ㄴ제](C:\Users\Yellme\Desktop\문ㄴ제.PNG)

 문제를 보면 이상한 값이 나와있다....... 

password에 'yell' , 'admin', '1234' 등 을 넣어봤지만 다 실패,,, 

그래서 저 것을 일단 한번 디코딩을 해보자! 

base64로 디코딩을 해보면 다음과 같이 변환이 된다.

```
c4033bff94b567a190e33faa551f411caef444f2
```

40 글자인 더 이상한 값이 나왔다............... 이것이 과연 뭘까? 하고 구글신에게 검색을 해보니,,

![캡처](C:\Users\Yellme\Desktop\캡처.PNG)

SHA-1 이라고 나왔다.



___* SHA-1이란 ?___

___SHA 함수 종류 중 하나이고 최대  2^64 비트의 메시지로부터 160bit의 해시값을 만들어 낸다.___

> SHA : 암호학적 해시함수로 160bit를 생성한다. MD4가 발전한 형태이고 MD5보다 느리지만 더 안전하다.

> 해시 값 :  해시 함수는 임의의 길이를 갖는 임의의 데이터에 대해 고정된 길이의 데이터로 매핑하는 함수를 말하고, 이러한 해시 함수를 적용하여 나온 고정된 길이의 값을 말한다.



그래서 SHA-1 디코딩을 해보면 다음과 같은 값이 나온다.

![fff](C:\Users\Yellme\Desktop\fff.PNG)

```
a94a8fe5ccb19ba61c4c0873d391e987982fbbd3
```

또 다른 SHA-1 값이 나와 제출을 해보았더니 아니라고 한다.  

그렇다면 다시 한번 더 디코딩을 해줍시다..!

한번 더 디코딩을 해주면 다음과 같이 어떤 값이 나온다. 

![fffff2](C:\Users\Yellme\Desktop\fffff2.PNG)

저 값을 제출을 해보니!!![dnjfjdkfds](C:\Users\Yellme\Desktop\dnjfjdkfds.PNG)

짜란, 문제 해결 --- !

 

------



## 2. Webhacking.kr 14번 

일단 문제를 봅시다

![14](C:\Users\Yellme\Desktop\14.PNG)

음.. 아무것도 모르니까 한번 'admin' 을 넣어보면 다음과 같이 Wrong이라고 뜬다..![14-3](C:\Users\Yellme\Desktop\14-3.PNG)

'1234' 넣어보니 또 Wrong 이라고 뜬다...

![14-4](C:\Users\Yellme\Desktop\14-4.PNG)



그래서 일단은 소스 코드를 분석을 해보쟈

![14-1](C:\Users\Yellme\Desktop\14-1.PNG)

빨간 박스를 위주로 분석을 해보면 

ul 변수에 document.URL값을 넣어주고 document.URL은 현재 실행되는 url값을 읽는다.

index0f 함수로 .kr 위치가 ul변수이다.

> index0f 함수 :  문자열에서 특정 문자열의 위치를 찾아서 lndex를 반환 해주는 함수 
>
> ​                           즉, 배열의 조건 문장을 숫자로 가져오는 것!

그리고 그 ul 변수 값에 * 30 을 해준다. 그리고 만약 ul이라는 값이 pw와 같으면 password를 출력한다. 아니면 Wrong을 출력한다.

URL 주소는 http://webhacking.kr/challenge/javascript/js1.html 이다.

url 주소를 배열로 만들어보면

| [ 0 ] = h | [ 6 ] = /  | [ 12 ] = c | [ 18 ] = k |
| --------- | ---------- | ---------- | ---------- |
| [ 1 ] = t | [ 7 ] = w  | [ 13 ] = k | [ 19 ] = r |
| [ 2 ] = t | [ 8 ] = e  | [ 14 ] = i | [ 20 ] = / |
| [ 3 ] = p | [ 9 ] = b  | [ 15 ] = n | [ 21 ] = c |
| [ 4 ] = : | [ 10 ] = h | [ 16 ] = g | [ 22 ] = h |
| [ 5 ] = / | [ 11] = a  | [ 17 ] = . | [ 23 ] = a |

이렇게 됩니다 

ul 값의 처음은 . 이니 배열로 하면 17이 된다.  따라서 ul = 17 인걸 알 수 있다. 

ul = ul * 30 이니까 17 * 30 = 510 이 된다. 

폼에 510을 넣어주면

![14-5](C:\Users\Yellme\Desktop\14-5.PNG)

Password 값이 나온다 !!!!! 

그 패스워드 값을 flag에 넣어주면 ![14-6](C:\Users\Yellme\Desktop\14-6.PNG)



짜아란 문제 해결 ---- !