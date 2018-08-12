## CRLF Injection_[HTTP Response Splitting]

2018-08-12 
Plit00 김두형



Root-me.org 라는 워게임 페이지에서 Web Server 의 CRLF 20 point 문제 Write-up

CRLF / HTTP Response Splitting ~

문제를 보자 

<img width="581" alt="challenges web - server crlf root me hacking and information security learning platform 2018-08-12 17-27-50" src="https://user-images.githubusercontent.com/40850499/44000122-1ed9c078-9e55-11e8-9fbc-87393cb67514.png">



hint! => 로그에 잘못된 데이터 주입~!



일단 문제의 이름인  CRLF이 무엇인지를 알아보자.

#### Analysis

---------------------------

> **HTTP 응답 분할 Injection**

> > CRLF를 이용해 생기는 취약점을 통해 HTTP Response에 악의적인 코드나 스크립트를 삽입해 XSS를 생성하거나 캐시를 조작 가능

> > > CRLF를 이용하여 Response를 두개 이상으로 분리하면서 원래의 HTTP Response와 별개로 새로운 HTTP Response를 만들어 조작 가능



### HTTP Vulnerability

1. XSS
2. Proxy and Web server Cache Poisoning
3. Hijacking the client's Session
4. Client Wen browser Poisoning

으로 볼 수 있다. 



---------------------------------

Start the challenge 를 누르면

![challenge01 root-me org web-serveur ch14 username](https://user-images.githubusercontent.com/40850499/44000146-a324e4b6-9e55-11e8-8ac6-df6f1ab97e4a.png)



이런 페이지가 뜨게되는데

Login 과 Password에 각각의 문자를 입력하면 틀린값이라고 authentication log에 뜨게 된다.

우리는 이를 이용하여 문제를 풀어야한다 문제의 이름과 이 취약점이 어디서 일어나는지에 대한 공격을 해야한다.



일단 이 취약점을 보안하려면

>  HTTP Response Header에 입력되는 값을 필터링을 하면 된다.
> HTTP 응답 헤더에 입력되는 값에 대하여 개행문자 %0D 혹은 %0A 를 제거/치환하는 등 입력에 대한유효성 검사를 실시함으로써 헤더가 두 개 이상으로 나뉘어지는것을 방지한다.



```asp
String test = test.replaceAll(“₩r”,””).replaceAll(“₩n”,” ”);
이것을 이용하여 보안해준다.
```



"%0D%0A%20+New_Header+%0D%0A" 와 같은 공격 구문을 사용할 수 있다.

서버 측에서 재전송하는 파라미터에서 새로운 라인을 만들어 헤더를 조작할 수 있게 된다

### Solve

```
우리가 %0D%0A%20+New_Header+%0D%0A 과 같은 공격으로 조작을 할 수 있게된다는걸 알게되면
즉시, 공격 코드를 짤 수 있다.
일단 username을 admin으로 설정하고 authentication log가 참으로 나타야함으로써 
코드를 autheticated로 해야한다.
```

-----------------------

```bash
$ http://challenge01.root-me.org/web-serveur/ch14/?username=admin%20authenticated.%0Aguest&password=admin

$ Result Well done, you can validate challenge with this 
  password : rFSP&G0p&5uAg1%

```



