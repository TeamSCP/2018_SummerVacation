Stack overflow attack
======

Buffer Overflow 공격의 세 가지 종류는<br>
1. Stack Overflows Attack<br>
2. Heap Overflows Attack <br>
3. Format String Attack <br>

등이 있다.<br>
이중에서 Stack Overflows Attack에 대해서 알아볼것이다.<br>
Stack Overflows Attack에 대해서 알아보기전에, Stack Frame이라는것을 먼저 알아볼 필요가 있다.<br>
<br>
새로운 프로그램이나 함수가 실행되면, stack frame이라는 메모리 단위가 stack에 할당된다.<br>
Stack frame에는 해당 함수나 프로그램이 종료된 후에 돌아가야할 주소값이 저장되어있다.
<br>
![](http://img1.daumcdn.net/thumb/R1920x0/?fname=http%3A%2F%2Fcfile21.uf.tistory.com%2Fimage%2F226BB336595CE56C25AD7A)<br>
<br>[사진 출처](http://sizzf.tistory.com/12)<br>
Stack overflow란 허용된 buffer의 크기를 초과하여 return값도 채우고 ebp가 저장된 공간도
넘어가고 return값에 문자열의 마지막인 null이 들어가게 되므로, 이 부분에 악성코드를 삽입하는 공격기법이다.
<br><br>
스택오버플로우 사용 이유
1. 프로그램의 흐름의 오류를 발생시켜서 시스템 동작 방해<br>
2. 변조되는 주소값을 시스템의 특정 코드(실행함수)가 위치하는 주소로 변조하여 임의의 명령을 실행<br>
3. 공격자가 실행을 원하는 악성코드를 파일 또는 시스템의 메모리 상에 위치시키고 그 주소값을 EIP 값에 덮어씌우고 실행하는 방식으로 악성코드를 실행
<hr>
다음부터는 실습 내용이다.(중간부터 막혀서 다음주에 마저 완성하도록 하겠습니다.)
<br>![](https://user-images.githubusercontent.com/37801624/43686538-c6d41fec-9902-11e8-9546-1fea19869f5a.PNG)<br>

공격대상은 Easy RM to MP# Converter입니다.

<br>![](https://user-images.githubusercontent.com/37801624/43686539-c6fddc60-9902-11e8-8f63-012e2b3b0109.PNG)<br>
perl을 이용해서 x41(=A) 10000개 입력 후 Crasg.m3u라는 파일로 저장

<br>![](https://user-images.githubusercontent.com/37801624/43686540-c7280760-9902-11e8-8530-3aac9ad603d0.PNG)<br>

 다음과 같은 명령어어를 입력 후 F5버튼을 누르면, 저장되면서 해당 코드가 실행됩니다.(저장 명은 test1.pl로 했습니다.)
 
<br>![](https://user-images.githubusercontent.com/37801624/43686541-c7538cc8-9902-11e8-9129-bdc1a8c54a43.PNG)<br>

<br>![](https://user-images.githubusercontent.com/37801624/43686543-c7a6c186-9902-11e8-841d-66cc617c5f68.PNG)<br>
crash.m3u라는 파일을 Easy Rm to MP3 Converter로 실행시켜보면 아직은 프로그램이 터지지 않은 것을 볼 수 있습니다.
<br>![](https://user-images.githubusercontent.com/37801624/43686544-c7cf6ea6-9902-11e8-87f9-4577b4e65039.PNG)<br>
같은 방법으로 A를 20000번 입력 후 같은 이름으로 다시 저장
<br>![](https://user-images.githubusercontent.com/37801624/43686545-c7f9a3e2-9902-11e8-9c89-37323d5d9327.PNG)<br>
아직도 터지지 않은 것을 볼 수 있습니다.
<br>![](https://user-images.githubusercontent.com/37801624/43686681-e1ad2b68-9904-11e8-85d1-820f77b7a2b4.PNG)<br>
30000번으로 하고 다시 저장하면
<br>![](https://user-images.githubusercontent.com/37801624/43686546-c836df96-9902-11e8-8105-884c6ee58c17.PNG)<br>
드디어 프로그램이 터진 것을 볼 수 있습니다.

이후 실습 진도가 나가지 않아서 다음 md파일에 올리도록 하겠습니다...............




참고 내용
- [안랩코코넛 SECU-LETTER 10월호<가 기재되어있는 카페글](http://cafe345.daum.net/_c21_/bbs_search_read?grpid=RYQP&fldid=359b&contentval=0006ozzzzzzzzzzzzzzzzzzzzzzzzz&nenc=&fenc=&q=%C5%F8%B9%D9%B1%B8%BC%BA%BF%E4%BC%D2&nil_profile=cafetop&nil_menu=sch_updw)<br>
- [corelan 팀의 [Exploit] Exploit writing tutorial part 1 : Stack Based Overflows](https://www.corelan.be/index.php/2009/07/19/exploit-writing-tutorial-part-1-stack-based-overflows/)
- [http://sizzf.tistory.com/12](http://sizzf.tistory.com/12)
