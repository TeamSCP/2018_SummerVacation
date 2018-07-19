## MD5_COMPARE[Wargame.kr]

2018.08.16 Plit00 김두형

#분석-지식</br>
#풀이</br>
#QnA</br>



#### 분석을 먼저 시작해보자

<img width="615" alt="wargame kr 8080 md5_compare view-source 2018-07-16 00-39-15" src="https://user-images.githubusercontent.com/13433722/42923382-34673e46-8b60-11e8-8bcc-3317561c6834.png">

> 문제에 대한 php소스파일이다.
> 우리가 여기서 봐야할 함수는 ctype_alpha와 is_numeric 이다

Ctype_alpha는 v1에 오는 text가 알파벳,즉 영어인지 확인하는 함수이다.
Is_numeric은 v2에 오는 text가 숫자인지 아닌지를 확인하는 함수이기때문에 v1은 영어,v2는 숫자여야 한다.

++ 세번째 if문을 보게되면 > if(md5($v1) != md5($v2)) {$chk = false;} < 
이것은 md5로 해쉬한 값이 같아야만 chk가 true로 남아있을 수 있습니다.

##### 지식

```
지식이란 무엇인가?
이것은 어떤 취약점으로 풀어야하는가! 이것이 관건입니다.

이 문제는 PHP Magic Hash 를 통하여 
String => float && 0=0 true로 만들어주면 flag가 나오게됩니다.
```



<img width="559" alt="php md5 240610708 md5 qnkcdzo hacker news 2018-07-16 01-37-57" src="https://user-images.githubusercontent.com/13433722/42923636-7ca9e52c-8b61-11e8-921b-e36be79cfa3d.png">

Md5('240610708')은 QNKCDZO 와 같다고 합니다.



String => float && 0=0 true 이게 무슨소리인가 하면!

v1의 ctype_alpha 값이 string으로 있다가 비교를 하기 시작하면 float 의 값

```bash
$ echo -n 240610708 | md5sum
    0e462097431906509019562988736854  -
    $ echo -n QNKCDZO | md5sum
    0e830400451993494058024219903391  -
    $ echo -n aabg7XSs | md5sum
    0e087386482136013740957780965295  -
```



문자열이 아닌 0e로 시작하는 값을 서로 비교할때 문자열이 아닌 float형으로 인식하게 되면서

0의 지수형을 나타내는 변수로 바뀌게 됩니다.

비교한 결과가 같다면 0=0 => true!!!!!!

