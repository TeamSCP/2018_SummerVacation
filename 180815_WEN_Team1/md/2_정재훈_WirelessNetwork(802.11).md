# Wireless Network

2018/08/16

정재훈

------

## 사건의 전말...

어느날 평범히 학교를 다니던 날 정**씨는 문득 와이파이 비밀번호가 기억이 나질 않는다. 뭐지뭐지뭐지뭐지뭐지

그래서 찾아보았다. 와이파이의 모든 것!

# 1. 쉽게 찾자

> Windows 설정 -> 네트워크 및 인터넷 -> 네트워크 및 공유 센터 -> 현재 연결중인 와이파이를 선택! -> 무선 속성 -> 문자표시(관리자 권한!!!!!)

<img src="https://user-images.githubusercontent.com/40850499/44184339-c8e5fb80-a149-11e8-9291-4c9852e3cf18.png" width="80%" align="center"></img>

끝났다.

뭔가 아쉽다.

이렇게 쉬울 리 없다. 좀더 꽁꽁 숨겨놔야하지 않을까

# 2. 좀더 간지나게 찾아보자

우선 관리자권한으루 명령창을 켜봅시더

명령어 뚝딱

Windows

> \> netsh wlan show profiles
>
> \> netsh wlan show profile name=“AP_name” key=clear 

MAC

> \> security find-generic-password -wa AP_name

따-란

근데 리눅스는 왜 없냐고?

# 3. 그래서 찾아보았다

리눅스에서 네트워크를 관리하는 것이 바로 NetworkManager 이다. `NetworkManager` 디렉터리 안에 `system-connection`이라는 디렉터리가 있는데, 그 안에 여태껏 연결 했었던 AP의 이름들이 좌아아아아악 나열되어 있다. 들여다 보면 이렇다.

<img src="https://user-images.githubusercontent.com/40850499/44178471-831b3a00-a12d-11e8-86cc-45570dfafd38.png" width="80%" align="center"></img>

호오옹 여기에 있었다. (잡았다 요놈!)

# 4. 802.11 연결 방식

802.11은 두 가지 연결 방법이 존재한다. (이제부터 진짜다. 앞에껀 다 사쿠라여)

우선 연결방법을 알기 전에 위의 결과로 생각을 해보자. 애초애 와이파이 라는 것이 인증되지 않은 기기와 연결을 시도해야하며, 연결이 된 적 있는 장치는 능동적으로 인식하므로서 연결 될 것이다.

먼저 "802.11 type subtype"을 구글에 쳐보자. 좌아아아악 나오는 것들이 있는데 그 중 여기서 알아야 할 것은 Beacon Frame과 Probe Request/Response 이다.

## 4-1. Passive

Passive 방식은 이전에 연결한 기록이 없을 경우 이루어지는 결합 과정이다.

<img src="https://mblogthumb-phinf.pstatic.net/20140508_201/shj1126zzang_13995321381547wLkh_PNG/1.png?type=w2" width="80%" align="center"></img>

1) AP는 자신을 알리기 위해 Broadcast로 BeaconFrame을 1초에 10번정도 날린다고 한다. 이 Beacon Frame에는 SSID, 채널, 암호화 방식, 전송률 등이 포함된다.

2) 두번째로 스테이션이 AP에 접속하기 위한 인증을 실시한다. AP는 응답메세지를 보낸다.

3) 인증이 완효되면, AP에 대한 접근 권한을 얻기 위해 Association Request 메세지를 보내고 응답을 받은 후 AP와 결합하게 된다.

## 4-2. Active

Active 방식은 Profile 이 있는 경우, 즉 아까와 같이 와이파이의 이름과 비밀번호 등의 연결 정보가 저장되어 있는 파일이 존재하는 경우의 결합과정이다.

<img src="https://mblogthumb-phinf.pstatic.net/20140508_144/shj1126zzang_1399532587518CdL6W_PNG/2.png?type=w2" width="80%" align="center"></img>

1) AP가 지속적으로 Beacon Frame을 쏘는 것과 다르게 Station에서 Probe Request를 보내면서 시작된다. Probe Request를 통해 이전에 접속했던 AP가 맞는지 확인하고 응답을 받는다.

2) 인증을 할 때가 패시브한 연결과정과 다르게 Profile에 기록이 있는 AP라면 바로 Authentication Request를 보내 인증을 시작한다.

3) 패시브와 동일

# 5. 패킷 수집 꿀팁

무조건 많은 패킷을 수집하기 보다, 유효한 정보만을 추리기 위해서 채널을 고정하는 방법이다.

먼저 모니터 모드로 랜카드의 모드를 수정하자

<img src="https://user-images.githubusercontent.com/40850499/44184100-d5b61f80-a148-11e8-92d0-c31b8344f7be.png" width="80%" align="center"></img>

<img src="https://user-images.githubusercontent.com/40850499/44184118-e8305900-a148-11e8-8124-ac163aefcf33.png" width="80%" align="center"></img>

요렇게 모니터모드로 바꿀 수 있다. 그리고 airodump-ng 를 이용해 주변 ap와 station을 살펴보면

<img src="https://user-images.githubusercontent.com/40850499/44184156-0ac27200-a149-11e8-84af-398f9cd3264d.png" width="80%" align="center"></img>

요기는 없지만 만약 opn모드인 것이 있다면 그 AP와 통신하는 Station의 패킷은 평문으로 보이게 된다. 만약 그 오픈모드인 AP의 채널이 5라고 가정하고 채널을 5로 고정 하고 와이어샤크로 패킷을 잡으면 된다.

<img src="https://user-images.githubusercontent.com/40850499/44184245-5ffe8380-a149-11e8-9cfc-e569c6dd1471.png" width="80%" align="center"></img>

고로면 특정 채널의 유효한 패킷만을 추려낼 수 있다.