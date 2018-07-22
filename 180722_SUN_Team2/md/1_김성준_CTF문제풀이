CODE GATE - RedVelvet

2018.07.23 Pvgodd 김성준

###Radre 2

###Angr

Radre 2 

을 이용해 func 을 분석하기 위해 pdf @ main 명령어를 사용한다.



func1~func15가 메인 프로세스이며, 마지막 비교 구문을 통해 플래그를 출력해줍니다. func1~func15를 살펴보면 이전 함수의 두번째 파라미터가 다음 함수의 첫번째 파라미터로 들어가는 것으로 보아 한글자 한글자씩 비교하는 형태로 보이며



모두 성공했을때 플래그를 보여주는 박스가보인다.

angr-Solve

    import angr
    
    proj = angr.Project("./RedVelvet")
    simgr = proj.factory.simgr()
    
    simgr.explore(find=lambda s: "HAPPINESS:)\n"*15 in s.posix.dumps(1))
    
    s = simgr.found[0]
    print s.posix.dumps(1)
    flag = s.posix.dumps(0)[:26]
    print flag

을 이용해 flag 값을 구할수있다.






