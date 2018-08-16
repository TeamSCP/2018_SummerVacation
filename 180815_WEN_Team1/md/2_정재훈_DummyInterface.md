# Dummy Interface

2018/08/16

정재훈

------

네트워크 프로그래밍을 할 때 생기는 예외 상황을 더 수월하게 처리하기 위해서 디버깅을 할 때 필수적으로 더미 인터페이스를 이용해야 한다. 그 이유는, 예를들어 정말 특수한 상황이 발생됐을 경우를 처리해야할 때 그 상황이 얼마 지나지 않아 다시 일어나는 상황이 아니지만 꼭 처리가 필요한 부분이라면 보통 그 상황을 덤프시켜놓고 덤프파일을 분석하게 될것이다. 하지만 그 상황을 만들기 위해 노력하는 것 보다 단지 그 덤프시켜 놓은 파일을 그대로 전송하는 방법이 있다. 바로 그것이 더미 인터페이스를 이용하는 방법이다.

또한, 보통 네트워크 프로그래밍을 하게 되면 와이어 샤크와 동시에 켜놓고 실행하게 된다. 하지만 매번 와이어샤크와 프로그램을 동시에 시작해서 동시에 끄는 일은 불가능 & 귀찮 .... 고로 더미 인터페이스를 이용하면, 한번 잡아놓은 패킷으로 수만번 돌려볼 수 있다.( 와이어 샤크를 같이 돌리는 일은 필요 없다는 말이다. )

자, 해보자

<img src="https://user-images.githubusercontent.com/40850499/44177889-cd4eec00-a12a-11e8-96b4-abe22e475621.png" width="80%" align="center"></img>

먼저 와이어샤크와 당신이 프로그래밍 한 프로그램은 있어야 한다. (그 프로그램을 디버깅 하기 위한 거니깐^^)

그리고 다음과 같은 명령어를 통해 네트워크 더미 인터페이스를 만들 수 있다.

> \# modprobe dummy
>
> \# ip link add dum0 type dummy

먼저 확인을 해보면 아직 더미 인터페이스가 보이지 않는다.

<img src="https://user-images.githubusercontent.com/40850499/44178018-6251e500-a12b-11e8-89a7-0bae490cfbd7.png" width="80%" align="center"></img>

이제 `ifconfig`를 이용해 up 시켜주자, 그러면

<img src="https://user-images.githubusercontent.com/40850499/44178058-8ad9df00-a12b-11e8-9e50-6ce9db10eff7.png" width="80%" align="center"></img>

위와 같이 더미 인터페이스가 생성된다.

이제 패킷을 생성하고 덤프 떠서 저장한걸 더미로 쏴주기만 하면 된다.

<img src="https://user-images.githubusercontent.com/40850499/44178089-ae048e80-a12b-11e8-9650-8f060b994f9a.png" width="80%" align="center"></img>

성☆공★

thanks to gilgil