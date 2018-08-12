기술문서 6주차
=========
root me 워게임 풀이
------
>1번 문제 <br>

<img width="299" alt="6 1" src="https://user-images.githubusercontent.com/40850499/44002271-1d4a189a-9e7b-11e8-8931-b300c1c798b5.PNG">
<br>버튼과 입력창이 모두 잠겨있는 상태임
<br><img width="383" alt="6" src="https://user-images.githubusercontent.com/40850499/44002287-447b3c50-9e7b-11e8-8733-0a44df58ca7f.PNG">
<br>그래서 소스를 확인해보자 auth-login과 Member access가 모두 diabled로 되어있는것을 확인함
<br><img width="405" alt="6 1" src="https://user-images.githubusercontent.com/40850499/44002302-7a3957be-9e7b-11e8-89d2-2b1f6b5d0506.PNG">
<br>disabled → abled로 바꿔줌
<br>![6](https://user-images.githubusercontent.com/40850499/44002339-f569db20-9e7b-11e8-8b7d-f7f87a7f40fe.PNG)
<br>1번 해결! flag값이 나옴

<br>
>2번 문제

<img width="325" alt="6 2" src="https://user-images.githubusercontent.com/40850499/44002348-2451e860-9e7c-11e8-86b7-eee26178841f.PNG">
<br>Username과 Password를 입력해서 푸는 문제
<br>먼저 Username에 Duni0107을 Password 1234를 넣어봄
<br><img width="545" alt="root me 2" src="https://user-images.githubusercontent.com/40850499/44002368-637b65b6-9e7c-11e8-89a0-16d4e1f81d77.PNG">
<br> login.js가 존재. 열어보자
<br><img width="925" alt="root me 2" src="https://user-images.githubusercontent.com/40850499/44002391-a6c46886-9e7c-11e8-9fd8-1114e05bc004.PNG">
<br>이런 소스가 나타남. 잘 읽어보면 username과 password 값이 존재
<br>if문이 성립할 때에 password값이 이 문제의 flag값임을 알 수 있음
<br><img width="259" alt="root me 2" src="https://user-images.githubusercontent.com/40850499/44002412-df5a5034-9e7c-11e8-978e-98039744b284.PNG">
<br>값을 넣어줌
<br><img width="284" alt="root me 2" src="https://user-images.githubusercontent.com/40850499/44002417-ec1791c4-9e7c-11e8-8542-61c5dd4695db.PNG">
<br>해결!!!!!!!!

        여기서 잠깐!!!!!!!!!!!!!

        소스를 보면 username과 password를 정의하는데 둘 다 toLowerCase()라는게 있음
        이것이 무엇일까 구글에게 물어봄

          → 문자열에 있는 모든 영문자를 소문자로 변환해주는 것


<br>
>3번 문제

<img width="330" alt="root me 3" src="https://user-images.githubusercontent.com/40850499/44002445-84b0d300-9e7d-11e8-8aa4-2e7100d20989.PNG">
<br>마찬가지로 3번도 password 값을 찾는 문제
<br>바로 소스를 읽어보자
<br><img width="726" alt="root me 3" src="https://user-images.githubusercontent.com/40850499/44002464-e969b974-9e7d-11e8-8d30-fd6416dae915.PNG">
<br>if문이 성립하게 해주면 이 password값이 flag 값이 되어줄것임
<br><img width="243" alt="root me 3" src="https://user-images.githubusercontent.com/40850499/44002475-173d5be4-9e7e-11e8-90d1-d8c4f8e7a129.PNG">
<br>알아낸 password 값을 입력해보자
<br><img width="284" alt="root me 2" src="https://user-images.githubusercontent.com/40850499/44002480-34564f74-9e7e-11e8-98ab-663a42bab0aa.PNG">
<br>해결 !!!!!!

<br>
>4번 문제

<img width="219" alt="root me 4" src="https://user-images.githubusercontent.com/40850499/44002486-520a3346-9e7e-11e8-8f33-a726a7ae0c5e.PNG">
<br>4번 문제는 저런 모습임
<br><img width="870" alt="root me 4" src="https://user-images.githubusercontent.com/40850499/44002491-66c0b79c-9e7e-11e8-9340-8286ad0600dd.PNG">
<br>소스를 보니 login.js가 존재함. 이동해보자
<br><img width="914" alt="root me 4" src="https://user-images.githubusercontent.com/40850499/44002517-c327cb2e-9e7e-11e8-8f7e-bfd8bb53b770.PNG">
<br>이런 소스가 나타남 읽어보자
<br>TheLists라는 리스트를 split 함수로 쪼개서 각각 Username, Password 변수에 넣어줬다는 것을 알 수 있음
<br><img width="330" alt="root me 4 god" src="https://user-images.githubusercontent.com/40850499/44002530-1632e63c-9e7f-11e8-8fa6-b448aceb2fa4.PNG">
<br>알아낸 Username 값을 입력
<br><img width="329" alt="hidden" src="https://user-images.githubusercontent.com/40850499/44002537-3b8abb08-9e7f-11e8-847e-e002d22c3954.PNG">
<br>알아낸 Password 값을 입력
<br><img width="331" alt="root me 4" src="https://user-images.githubusercontent.com/40850499/44002549-67d7f7b6-9e7f-11e8-9eac-df0c44be223d.PNG">
<br>마찬가지로 이 Password 값이 문제의 flag값이라는 문구가 뜸
<br><img width="213" alt="root me 4" src="https://user-images.githubusercontent.com/40850499/44002552-751dd896-9e7f-11e8-821d-6806a3d1f938.PNG">
<br>flag값을 입력
<br><img width="195" alt="root me 4" src="https://user-images.githubusercontent.com/40850499/44002557-886b2f52-9e7f-11e8-9549-57daf93eb592.PNG">
<br>문제 해결 10 포인트를 얻음!!!!!!!
