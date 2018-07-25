# FTZ level 1 & 2 의 문제풀이



7 / 25, 2018 Jinyan 김상민	



FTZ 1번 ~ 2번까지의 문제풀이를 하고자 합니다.

> 환경 : Windows 10
>
> - Red Hat Linux 9.0 (VMware Workstation)
> - putty



#### 1 - 1 문제 풀이

level 1을 로그인하면 디텍토리와 파일들이 어떤것이 있는지 확인해 볼수 있습니다.

<img src=https://user-images.githubusercontent.com/37801676/43205231-e2d6afd2-905d-11e8-931d-2dcf043469fb.PNG>

디텍토리에 문제에 의도를 보내는 메세지인 hint를 볼 수 있습니다.

디텍토리 속의 hint가 파일인 것을 확인하고 들여다 보면

<img src=https://user-images.githubusercontent.com/37801676/43205232-e3c84f86-905d-11e8-825e-c2c0290229c5.PNG>

> level2 권한에 setuid가 걸린 파일을 찾는다.

라는 문구를 볼 수 있습니다. 

먼저 SetUID는 Trainer 10에서 배울 수 있는 내용입니다.



```
SetUID 란

Set + User ID = 유저(본인) ID를 변경한다. 라는 뜻인데요.

SetID가 걸려있는 파일을 조작하면 잠시 동안 권한을 얻을 수 있습니다. 
```



SetUID가 걸린 파일을 찾기 위해서 사용되는 명령어를 사용하면

```
find /-user level2 -perm -4000
= 유저의 권한을 가진 파일을 찾는다. (- 최소한 / -perm 권한을 찾는다 / setuid r,w,x 모두 의미 )
```

<img src=https://user-images.githubusercontent.com/37801676/43205238-e5d58e38-905d-11e8-9da8-68a61fec5f9c.PNG>

Permission denied (허가거부)가 많이 보이지만 그 중 /bin/ExecuteMe를 발견 하게 됩니다.



다시 cd로 bin 디텍터리 경로로 이동 해주고 ExecuteMe 를 확인해주면 파일인 것을 확인 할 수 있습니다.

<img src=https://user-images.githubusercontent.com/37801676/43205239-e604f98e-905d-11e8-9fbd-6e3d67271f8a.PNG>

ExecuteMe 파일을 실행을 시켜 주게 되면

<img src=https://user-images.githubusercontent.com/37801676/43205241-e634383e-905d-11e8-8d81-1cedd6e4b4d9.PNG>

<img src=https://user-images.githubusercontent.com/37801676/43205243-e662f5b6-905d-11e8-87d7-b5d2af71f7cb.PNG>

키를 얻을수 있는 my-pass와 chmod의 명령어는 사용이 불가능하고 그 외의 명령어만 사용이 가능합니다.

그러므로 level2의 권한을 유지하기 위해서 bash셀을 이용해 주게 됩니다.

<img src=https://user-images.githubusercontent.com/37801676/43205244-e6912d46-905d-11e8-9a92-f0e4a766d5b8.PNG>

그렇게 되면 level2의 권한을 그대로 유지하게 되고 

<img src=https://user-images.githubusercontent.com/37801676/43207085-2cf282c2-9062-11e8-8b9e-1ffa7ce79608.jpg>

my-pass 명령어를 사용할 수 있게 됩니다.



- Trainer 10번을 배우고 와서 힘들지않게 바로 답을 찾아낼 수 있었다.





#### 2 - 1 문제풀이

level 2에서도 디텍토리에서 hint라는 파일을 확인 할 수 있었습니다.

<img src=https://user-images.githubusercontent.com/37801676/43208689-f011848a-9065-11e8-8f18-dbe3d76aadff.PNG>

hint의 내용을 들여다 보면 

<img src=https://user-images.githubusercontent.com/37801676/43208690-f040dc3a-9065-11e8-950e-865f5c28f4b4.PNG >

텍스트 파일 편집으로 셀의 명령을 실행하면 되는 문장이 나오게 됩니다.



먼저 level3의 SetUID를 찾아 봅시다.

<img src=https://user-images.githubusercontent.com/37801676/43208692-f06d0eea-9065-11e8-9c21-9ffe0de3b1ca.PNG >

그러면 Permission denied(허가거부) 속에서 /usr/bin/editor를 발견 할 수 있습니다.

그럼 /usr/bin 디텍터리 경로로 이동을 해주고 

 <img src=https://user-images.githubusercontent.com/37801676/43208693-f09caa56-9065-11e8-8cba-d46d741394bf.PNG >

editor를 실행 시켜 줍니다.

그러면 vi Editor가 나오게 되는데 유닉스 시스템에 사용되고 있는 편집기로 bash셀로 넘어가는 명령어를 입력하게 되면

<img src= https://user-images.githubusercontent.com/37801676/43208700-f3b9852e-9065-11e8-8e93-140a41cff192.PNG>

level3 권한으로 넘어올 수 있게 됩니다.

<img src=https://user-images.githubusercontent.com/37801676/43208702-f4233b72-9065-11e8-8b6e-89b700a28700.PNG>

그런 후 my-pass를 입력하여 키를 알아낼 수 있었습니다.



#### 2 - 2 막혔던 곳

텍스트를 이용하는 힌트가 Trainer8의 텍스트 파일 생성 및 작업에 대한 내용이 떠올라 막힘없이 가다가 cat >> editor에서 막히게 되었습니다.

<img src=https://user-images.githubusercontent.com/37801676/43208694-f0cb2e26-9065-11e8-9adf-7de2e800dca4.PNG>

하지만 금방 실행하면 나오는 것을 알게 되었는데

vi editor에서 또 한번 문제가 생겼습니다. vi의 명령어를 모르기 때문에 /bin/bash를 입력하고 실행하였는데,

<img src= https://user-images.githubusercontent.com/37801676/43208699-f38dc7c2-9065-11e8-90bb-211267be808a.PNG>

실행이 되지 않았고, 찾아보고 vi에 대해서 보다보니 유닉스 명령어를 사용할때 실행해주는 !(느낌표)가 필요하다는 것을 알게 되었고, 이 후 해결하게되었습니다.