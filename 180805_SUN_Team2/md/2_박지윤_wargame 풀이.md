기술문서 5주차
=======
Javascript
----

조건문 else - if만으로는 좀 더 복잡한 상황을 정리하기가 부족

        if(true) {
          alert(1);
        } else {
          alert(2);
        }
        → 결과 : 1

        if(false) {
          alert(1);
        } else {
          alert(2);
        }
        → 결과 : 2
        → 설명 : if문의 조건이 true이면 if의 중괄호 구간이 실행됨.
                false이면 else 이후의 중괄호 구간이 실행됨. else는 주어진 조건이 거짓일 때 실행할 구간을 정의하는 것

else if - 조건문을 좀 더 풍부하게 할 수 있음

        if(false) {
          alert(1);
        } else if(true) {
          alert(2);
        } else if (true) {
          alert(3);
        } else {
          alert(4);
        }
        → 결과 : 2

        if(false) {
          alert(1);
        } else if(false) {
          alert(2);
        } else if (true) {
          alert(3);
        } else {
          alert(4);
        }
        → 결과 : 3

        if(false) {
          alert(1);
        } else if(false) {
          alert(2);
        } else if (false) {
          alert(3);
        } else {
          alert(4);
        }

        → 결과 : 4
        → 설명 : else if의 특징은 if나 else와는 다르게 여러개가 올 수 있음. else if의 조건이 모두 false이면 else가 실행


변수와 비교연산자
<br><br>
<img width="324" alt="default" src="https://user-images.githubusercontent.com/40850499/43385119-dee5a318-941a-11e8-8f6d-0fcb02204afc.PNG">
<br>

        아이디를 coding으로 설정해주고 맞으면 '아이디가 일치합니다' 틀리면 '아이디가 일치하지 않습니다'가 나오게 설정
<br>
<img width="331" alt="id" src="https://user-images.githubusercontent.com/40850499/43385187-078b039e-941b-11e8-9aee-a2a7dcd571ca.PNG">
<br> 잘못된 아이디를 입력해줌<br>
<img width="332" alt="id x" src="https://user-images.githubusercontent.com/40850499/43676707-504aeb20-9831-11e8-9d2f-b7542dd85e41.PNG">
<br>아이디가 일치하지 않습니다 문구가 뜨는 것이 확인됨
<br><img width="329" alt="id" src="https://user-images.githubusercontent.com/40850499/43385291-52249190-941b-11e8-99c5-b3e9eb13ad3f.PNG">
<br>이번엔 제대로 된 아이디를 입력해줌<br>
<img width="330" alt="id" src="https://user-images.githubusercontent.com/40850499/43385308-60d464ae-941b-11e8-9e71-d8c1228b8fe0.PNG">
<br>아이디가 일치합니다 문구가 뜨는 것이 확인됨
<br>

          → 위의 내용에서 promrt() 구문은 사용자가 입력한 값을 가져와서 id 변수의 값으로 대입함. 이를 함수라고 함

조건문의 중첩
<br>
<img width="404" alt="default" src="https://user-images.githubusercontent.com/40850499/43385918-0522db5c-941d-11e8-8250-4a6b4dcb34e1.PNG">
<br>

        if문 안에 if문 등장! 사용자가 입력한 값과 아이디의 값이 일치한지 확인 후 일치한다면 비밀번호가 일치하는지 확인
        조건문은 조건문 안에 중첩해서 사용될 수 있음

<br>
<img width="329" alt="id" src="https://user-images.githubusercontent.com/40850499/43385291-52249190-941b-11e8-99c5-b3e9eb13ad3f.PNG">
<br><br><img width="328" alt="default" src="https://user-images.githubusercontent.com/40850499/43386030-5ac4a702-941d-11e8-9689-c93dc4fcbc8a.PNG">
<br>
올바른 아이디를 입력해주면 아이디가 일치한다는 문구 뜸
<img width="330" alt="0000" src="https://user-images.githubusercontent.com/40850499/43386108-9d9cc3a2-941d-11e8-8ac7-8166b7e82f4f.PNG">
<br>
잘못된 비밀번호 0000을 입력<br>
<img width="328" alt="default" src="https://user-images.githubusercontent.com/40850499/43386132-ac9b349c-941d-11e8-8e87-f925137d0665.PNG">
<br>인증 실패가 뜸
<br>
<img width="330" alt="0107" src="https://user-images.githubusercontent.com/40850499/43386153-bd0695c4-941d-11e8-9e97-28225dde8042.PNG">
<br>제대로 된 비밀번호 0107을 입력<br>
<img width="328" alt="default" src="https://user-images.githubusercontent.com/40850499/43386167-ca8e0466-941d-11e8-94fc-e73e31f53ca5.PNG">
<br>인증 확인 <br>


Wargame.kr을 풀어보자
-------

<img width="600" alt="fly me to the moon" src="https://user-images.githubusercontent.com/40850499/43677649-62ccfd2e-9840-11e8-8a59-340abd8961a2.PNG">
푸는데 거의 5일이 걸린 엄청난 삽질을 하게 만들었던 이 문제를 풀어보자

<br>
자바 스크립트 문제라는 말을 확인하고 스타트 버튼을 눌러보면 게임이 나옴
<img width="449" alt="fly me to the moon" src="https://user-images.githubusercontent.com/40850499/43677653-824539c8-9840-11e8-91c9-95436849adc5.PNG">
<br>배가 벽과 부딪히지 않아야하는 게임임. 벽에 부딪히면 게임 오버<br>

      추측

      1. 벽을 없애자
      2. 배를 없애자
      3. 벽과 배가 부딪혀도 게임이 끝나지 않게 하자
      4. 점수를 조작하자


<br>![default](https://user-images.githubusercontent.com/40850499/43687031-c8c35eb0-9909-11e8-9b28-d6fdf0cd0cd6.PNG)
<br>소스를 보고 배의 위치를 지워서 재설정해줌
<br>

![default](https://user-images.githubusercontent.com/40850499/43687011-8f68148a-9909-11e8-881e-a4196ecb0f82.jpg)
<br>배와 벽이 부딪혀서 게임이 끝남
<br>그래서 문제에서 얘기했던 자바스크립트를 해석해서 풀기로 함

<br>
<img width="413" alt="default" src="https://user-images.githubusercontent.com/40850499/43677672-f43fe8de-9840-11e8-9b10-52bdb03420c4.PNG">
<br>기존에 코드는 이렇게 전혀 알아볼 수 없게 되어있음. 그래서 보기 좋게 정리해봄
<br>
<img width="342" alt="default" src="https://user-images.githubusercontent.com/40850499/43677679-14e0b29e-9841-11e8-9ea8-be2698b3df94.PNG">
<br>확실히 알아 볼 수 있게 정리가됨. 이제 벽에 부딪히면 죽는 부분을 찾아봄
<br><img width="421" alt="default" src="https://user-images.githubusercontent.com/40850499/43677687-333c933e-9841-11e8-8618-20e2cc5d8d33.PNG">
<br>벽에 부딪히면 죽는 부분을 확인함. 이제 ||-or 부분을 &&-and 로 바꿔줌
<br>

![default](https://user-images.githubusercontent.com/40850499/43687057-1532ac06-990a-11e8-9753-61fa82e5cac2.png)
<br>이런식으로 &&로 바꿔준 뒤 내용을 콘솔에 추가

<br>
![image](https://user-images.githubusercontent.com/40850499/43686647-5eb7206a-9904-11e8-9a37-7ab6c350c0fd.png)
<br>벽에 부딪혀도 죽지 않는 무적의 배가 완성됨
<br>그런데 31337점을 넘겨도 죽지를 않아서 게임이 끝나지를 않음...
<br>그래서 점수를 조작하여 문제를 해결해야겠다고 생각함
<br>

![image](https://user-images.githubusercontent.com/40850499/43686662-902d58da-9904-11e8-93e8-d587382430e0.png)
<br>이 부분이 점수를 정의하는 부분임. 이제 점수 조작을 해보자

![31337](https://user-images.githubusercontent.com/40850499/43687066-4783e788-990a-11e8-8e83-941d10ff12ad.PNG)
<br>원래 점수를 정의하던 부분을 지우고 31337점으로 조작한 후 콘솔에 추가

<br>![fly](https://user-images.githubusercontent.com/40850499/43687077-61e385b6-990a-11e8-8ed6-224f5dfeac99.PNG)
<br>실행시켜보면 바로 게임이 해결되고 key 값이 나타남
>다 풀고보니 '그렇게 어렵지 않았네..'라는 생각이 들었지만 많은 시행착오를 겪고 풀어낸 문제였음
자바스크립트를 정확히 해석할 줄 알고 콘솔에 추가할줄 안다면 어렵지 않게 풀 수 있음ㅜㅡㅜ

끝!
