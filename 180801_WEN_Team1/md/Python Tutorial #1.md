# Python Tutorial #1

**26 Jul 2018 이승혁**

이번에는 파이썬을 공부하면서 배우게 된 내용들 중 현재까지도 유용하게 사용하고 있는 함수나 모듈을 소개시켜 드리겠습니다. 이미 아시는 분들도 많겠지만 모르는 사람들을 대상으로 하는 문서이기 때문에 모르는 사람이 읽는 것을 전제로 하여 쓰도록 하겠습니다.

------

1. 람다식(Lambda)
2. map
3. reduce
4. filter

------

## 1. 람다식(Lambda)

람다식은 파이썬 외에도 C++, C#, Java등에도 존재하는 형식입니다.

람다는 함수를 간편하게 작성할 수 있어서 다른 함수의 인수로 넣을 때 주로 사용합니다.

익명함수이기 때문에 한번 쓰고 다음으로 넘어가면 힙(Heap) 메모리 영역에서 사라집니다.

파이썬에서는 모든 것이 객체로 관리 되고 각 객체들은 레퍼런스 카운터를 갖게 됩니다. 이 카운터가 0이 되어 누구도 참조하지 않게 된다면 메모리를 환원 하게 됩니다.

람다식은 다음과 같은 방식으로 사용합니다.

> lambda 인자 : 표현식

예를 들어

>  def sum(a, b):
> 	return int(a) + int(b)
>
> print "Sum :", sum(a, b)
> print "Lambda :", (lambda x, y : x + y)(a, b)

아래의 두 줄은 각각 a 와 b의 합연산을 시행하여 결과를 출력합니다. 

<img width="476" alt="screen shot 2018-08-02 at 1 42 49 pm" src="https://user-images.githubusercontent.com/40850499/43563843-e363f4c6-965e-11e8-9534-15aa718f618b.png">

------

### map

map 함수는 리스트의 모든 값에 주어진 함수를 적용시킵니다.

> map(함수, 리스트)

위와 같이 사용됩니다.

> inp = raw_input().split()
>
> inp = map(int, inp)

위 코드의 첫 줄에서 inp에 string 데이터를 입력받게 되면 두번째 줄의 map 함수가 그 string 데이터를 int 형으로 바꿔줍니다.<img width="164" alt="screen shot 2018-08-02 at 1 38 28 pm" src="https://user-images.githubusercontent.com/40850499/43563840-e2ff2c44-965e-11e8-844e-5f090a85a3ec.png">

예를 들어, inp에 1 2 3 4 5 를 입력했다면, map 함수의 이전에는 1 2 3 4 5가 str 형이기 때문에 1 2 3 4 5 의 아스키코드인 49 50 51 52 53 으로 저장되어 있지만 map 함수로 형변환을 시행 한 이후에는 int형 1 2 3 4 5 로 저장됩니다.

------

### reduce

> reduce(함수, 리스트)

reduce 함수는 리스트의 첫번째 값 부터 마지막 값 까지 연속적으로 함수를 적용시킵니다.

> def mulsum(a):
> 	sum = 0
> 	for i in range(len(inp)):
> 		sum += inp[i]
>
> ​	retrun sum
>
> print "Sum :", mulsum(inp)
> print "Reduce :", (reduce(lambda x, y : x + y, inp))

리스트의 값들로 두 인자에 함수를 누적적으로 적용합니다 위의 식에서는 왼쪽 인자 x는 오른쪽 인자인 y와 합한 값이 되고 x의 값은 매 연산마다 갱신됩니다. 주어진 리스트의 값을 전부 이용할 때 까지 진행되며, 위의 x의 값은 리스트의 각 값들과 연속적으로 연산됩니다. reduce 안에도 조건을 위해 람다가 쓰이는 것을 볼 수 있습니다.

<img width="476" alt="screen shot 2018-08-02 at 1 45 29 pm" src="https://user-images.githubusercontent.com/40850499/43563844-e38c60d2-965e-11e8-867b-5a77c216c5c3.png">

------

### filter

> filter(함수, 리스트)

filter 함수는 리스트의 값들을 함수 형식으로 주어진 조건에 적용하여 결과가 참이 되는 것들로 새로운 리스트를 만들어 주는 함수입니다.

> def evenfinder(inp):
> 	res = []
> 	for i in range(len(inp)):
> 		if inp[i] % 2 == 0:
> 			res.append(inp[i])
> 	return res
>
> print "Function :", evenfinder(inp)
> print "Filter :", filter(lambda x : x % 2 == 0, inp)

위 예제는 짝수를 찾아주는 코드입니다. 만들어진 함수로 출력 한 결과와 람다와 filter를 이용한 결과가 동일하지만 코드 길이의 차이는 상당히 많이 나는 것을 볼 수 있습니다. 

<img width="471" alt="screen shot 2018-08-02 at 2 10 08 pm" src="https://user-images.githubusercontent.com/40850499/43563845-e3ca8c9a-965e-11e8-82ad-ce7dfa45fdcf.png">

> print "List Expression :", [i for i in inp if i % 2 == 0]

하지만 위의 방법 처럼 리스트 표현식을 사용하는 것이 속도도 빠르고 알아보기 쉽기 때문에 리스트 표현식으로 사용할 수 있다면 리스트 표현식을 사용하는 것이 좋다고 합니다.



이상입니다.