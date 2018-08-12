## XorDDoS Analysis

### 요약
해당악성코드는 공격자에 의해 감염 후 , 원격지와 통신을 수행하고 이통신을 통해 공격하고자 하는 대상의 ip 주소를 얻고,
그후 대량의 Packet을 대상 주소로 전송하는 악성코드이다.

<img width="636" alt="2018-08-13 12 59 21" src="https://user-images.githubusercontent.com/40850499/44005445-908a2e8e-9eae-11e8-8be5-494920317839.png">

(Kali-KM 사진참조)

<img width="795" alt="2018-08-13 4 14 59" src="https://user-images.githubusercontent.com/40850499/44005512-9b460c8e-9eaf-11e8-834c-6f1b5cd9ad1c.png">

## 분석 

<img width="797" alt="2018-08-13 4 17 44" src="https://user-images.githubusercontent.com/40850499/44005538-e591ce22-9eaf-11e8-8e61-35dc2d14e200.png">

/bin/&Random = 임의의 문자열 

<img width="793" alt="2018-08-13 4 18 50" src="https://user-images.githubusercontent.com/40850499/44005548-086cd536-9eb0-11e8-9300-1146b0b920ed.png">

/etc/cron.hourly/:
gcc.sh: 지속성 유지
$Random:이 파일은 매번 삭제 될 때마다 재생성되며, / bin에있는 Random 파일도 똑같은기능을한다.

<img width="790" alt="2018-08-13 4 18 59" src="https://user-images.githubusercontent.com/40850499/44005551-1742bb98-9eb0-11e8-8fe6-82d2f85d8677.png">

/ lib /
libudev.so : "백업"용도

### 지속성 유지 
추가적으로 악성 샘플을 다시 실행시키기 위해 '/etc/crontab' 파일을 변조한다. crontab 파일은 Windows 의 작업 스케쥴러와 같은 역할을 하며 계속실행시킨다.
### 코드부분을 좀더 분석한후 상세설명
내부 네트워크를 만들고나서 실행 해야하는데 못만들고있어서 자세한분석이 미약한부분 설명추가

md파일을 쓰기위한 준비과정 네트워크쪽을 공부해 내부 서버망을 만들어 맥북을 공격서버 데탑을 목표서버로 만든뒤에 실행할 계획에있지만 아직까지 성과가 없습니다..
