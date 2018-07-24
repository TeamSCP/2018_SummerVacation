# CTF ZONE - Validator3000

2018.07.23 Pvgodd 김성준

#IDA 으로 분석

#dump를이용한 플래그값 추출

## IDA 

<img width="1290" alt="2018-07-25 1 09 17" src="https://user-images.githubusercontent.com/40850499/43152449-83193650-8fa9-11e8-9ce6-d2840ef32a45.png">

분석을 해본결과 JZ분기문을 이용해 Checker module is not available 으로 점프하는걸 볼수있다 생각을 해본결과 Validator 에 뜻을생각하에 dump 파일로 분석하기로했다.



## dump 

작업 관리자 덤프파일 만들 파일 선택 덤프파일 만들기 를 눌른다

AppData\Local\Temp\Validator3000.DMP

에파일 위치가있다.

파일을 확장자 .TXT로 바꿔주고 C T F Z O N E 을 쳐보면 플래그가나온다.














