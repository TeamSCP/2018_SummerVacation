# Javascript-Obfuscation

2018.7.9 plit00 김두형

이 문서는 root-me.org 에 있는 web-client 문제 중에서 자바스크립트 난독화 2,3을 푼것이다



#### Charcode

이 문제들을 풀기전에 알아야할 지식들이 있다.

- Charcode : 키보드에서 유니코드를 나타내는문자 

- event.keycode : 문자가 아닌 다른 key event를 반환(ASCII를 기반한다)

- event.charcode : 문자 키의 유니코드 값을 반환

  

  - 다음의 소스를 보자

    ```javascript
    function getKey(event)
    {
        event = event || window.event;
        var keyCode = event.which || event.keyCode;
        alert(keyCode);
    }
    function getChar(event)
    {
        event = event || window.event;
        var keyCode = event.which || event.keyCode;
        var typedChar = String.fromCharCode(keyCode);
        alert(typedChar);	
    }
    ```

    

  위의 소스를 이용하면 밑과 같은 것을 만들수 있다.

   

![](https://user-images.githubusercontent.com/13433722/42440970-0a8644e8-83a2-11e8-8e51-d488d43198c8.png)



예를 들어서 18이라는 숫자를 쳤다 

> Kedown.event.keyCode : 56
>
> Keypress.event.keyCode : 56
>
> keyup.event.keyCode : 56

위와 같이 나오는 이유는 옆의 표를 보게되면 18은 1 = blank || 8 = 56 이라는 값을 가지고 있기때문에 

56이 표시되는것입니다.



@Example

```javascript
function keycheck(evt)
{
    var keyCode = evt.which?evt.which:event.keyCode;
}
```

> evt 전달된 값의 which 속성을 본 후 값이 유효하다면 이 값을 사용하도록 합시다.



### Javascript-Obfuscation 2

문제의 소스를 보자

```html
<script type="text/javascript">
	var pass = unescape("unescape%28%22String.fromCharCode%2528104%252C68%252C117%252C102%252C106%252C100%252C107%252C105%252C49%252C53%252C54%2529%22%29");
</script>
```

우리가 봐야할 부분은 'var pass' 이다.

> Unescape는 charstring의 콘텐츠를 포함하는 문자열 값을 반환합니다

```bash
#Solve1

>unescape("unescape%28%22String.fromCharCode%2528104%252C68%252C117%252C102%252C106%252C100%252C107%252C105%252C49%252C53%252C54%2529%22%29")
<-"unescape("String.fromCharCode%28104%2C68%2C117%2C102%2C106%2C100%2C107%2C105%2C49%2C53%2C54%29")"
>unescape("String.fromCharCode%28104%2C68%2C117%2C102%2C106%2C100%2C107%2C105%2C49%2C53%2C54%29")
<- "String.fromCharCode(104,68,117,102,106,100,107,105,49,53,54)"
> String.fromCharCode(104,68,117,102,106,100,107,105,49,53,54)
```

```bash
#Solve2

1. charcode decoder ("String.fromCharCode%2528104%252C68%252C117%252C102%252C106%252C100%252C107%252C105%252C49%252C53%252C54%2529%22%29")

2.codestring eval(pass1) / eval(pass2)
```

> **eval()** 문자로써 표현된 Javascript 코드를 실행하는 메소드이다.
>
> 얻어진 값이 비어있다면 **undefined** 가 반환된다.



### Javascript-Obfuscation 3

이문제는 2번문제와 같은계열의 문제이다.

```javascript
function dechiffre(pass_enc){
        var pass = "70,65,85,88,32,80,65,83,83,87,79,82,68,32,72,65,72,65";
        var tab  = pass_enc.split(',');
                var tab2 = pass.split(',');var i,j,k,l=0,m,n,o,p = "";i = 0;j = tab.length;
                        k = j + (l) + (n=0);
                        n = tab2.length;
                        for(i = (o=0); i < (k = j = n); i++ ){o = tab[i-l];p += String.fromCharCode((o = tab2[i]));
                                if(i == 5)break;}
                        for(i = (o=0); i < (k = j = n); i++ ){
                        o = tab[i-l]; 
                                if(i > 5 && i < k-1)
                                        p += String.fromCharCode((o = tab2[i]));
                        }
        p += String.fromCharCode(tab2[17]);
        pass = p;return pass;
    }
    String["fromCharCode"](dechiffre("\x35\x35\x2c\x35\x36\x2c\x35\x34\x2c\x37\x39\x2c\x31\x31\x35\x2c\x36\x39\x2c\x31\x31\x34\x2c\x31\x31\x36\x2c\x31\x30\x37\x2c\x34\x39\x2c\x35\x30"));
    
    h = window.prompt('Entrez le mot de passe / Enter password');
    alert( dechiffre(h) );
```

> 우리가 봐야할 코드는 var pass와 string["fromcharcode"]_(decjoffre)부분인데 
>
> 마찬가지로 console을 이용해서 보자.

```bash
>String.fromCharCode(70,65,85,88,32,80,65,83,83,87,79,82,68,32,72,65,72,65)
>"FAUX PASSWORD HAHA"
```

? 한번에 안된다…. 자 그럼 밑으로 가보자

![image-20180709185055365](https://user-images.githubusercontent.com/13433722/42443739-460362ec-83a9-11e8-87ad-88a619cc098d.png)

> dechiffre("\x35\x35\x2c\x35\x36\x2c\x35\x34\x2c\x37\x39\x2c\x31\x31\x35\x2c\x36\x39\x2c\x31\x31\x34\x2c\x31\x31\x36\x2c\x31\x30\x37\x2c\x34\x39\x2c\x35\x30") 을 한 결과이다.

dechiffre("~~")에 있는 것을 디코딩해준다면 위와 같이 참값이 나오게된다. 
