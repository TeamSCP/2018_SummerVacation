# 수요일까지 다못올린 문제올리겠습니다
# CODE GATE - RedVelvet

2018.07.23 Pvgodd 김성준

###Radre 2

###Angr

## Radre 2 

을 이용해 func 을 분석하기 위해 pdf @ main 명령어를 사용한다.

<img width="790" alt="2018-07-23 2 23 51" src="https://user-images.githubusercontent.com/40850499/43048286-104ae2b6-8e20-11e8-8cec-5b036ade8e3f.png">

func1~func15가 메인 프로세스이며, 마지막 비교 구문을 통해 플래그를 출력해줍니다. func1~func15를 살펴보면 이전 함수의 두번째 파라미터가 다음 함수의 첫번째 파라미터로 들어가는 것으로 보아 한글자 한글자씩 비교하는 형태로 보이며

<img width="1680" alt="2018-07-23 2 51 56" src="https://user-images.githubusercontent.com/40850499/43048490-7e2174b4-8e23-11e8-8f38-c9efe759d070.png">

모두 성공했을때 플래그를 보여주는 박스가보인다.

## angr-Solve

```python
import angr

proj = angr.Project("./RedVelvet")
simgr = proj.factory.simgr()

simgr.explore(find=lambda s: "HAPPINESS:)\n"*15 in s.posix.dumps(1))

s = simgr.found[0]
print s.posix.dumps(1)
flag = s.posix.dumps(0)[:26]
print flag
```

을 이용해 flag 값을 구할수있다.











