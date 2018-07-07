Machine Learing - Tensorflow#1
==
7월 8일, 2018 edithedith404(plit00) 김두형
필자가 Mac OS X를 쓰기에 mac위주의 설명이다.

#— [보기](https://www.tensorflow.org/community/) 이 커뮤니티를 통해 공부한 내용

컴퓨터가 개발스킬 등을 배울수 있도록 만드는 프로그래밍 알고리즘  & 딥러닝

* C

* C++

* Python(C,C++에서 제공하지않는 많은 헬퍼 함수들을 쓸 수 있기때문에 파이썬을 사용해준다.)

  

텐서플로우는 오픈소스 라이브러리로 데이터 흐름도를 이용하여 수치를 해석한다.

- 연산은 graph로 표현
- Graph는 session 에서 실행
- 데이터는 tensor로 표현



install
-----
먼저 pip를 설치해야한다.
#pip는 파이썬 패키지를 설치하고 관리하는 패키지 매니저 프로그램이다.

1.pip 설치하기

```python
$sudo easy_install pip
$sudo easy_install --upgrade six
```

2.Tensorflow 설치

```python
$sudo pip3 install --upgrade $TF version
```

3.Anaconda 설치

아나콘다는 여러 수학,과학 패키지를 기본적으로 포함하고 있는 파이썬 배포판이다.

```python
$ conda create -n tensorflow python=3.5 
```

  

설치테스트

```python
$ source activate tensorflow
#텐서플로우를 사용한 프로그램을 실행한다.

--
(tensorflow)$ source deactivate
```



### 텐서플로우 실행(& Example)

터미널을 실행하여 아래 명령을 실행한다.
우리가 C를 할때 hello world! 를 출력하는것과 같이 hello, tensorfolw!를 출력해보자.

```python
$ python
......
>>> import tensorflow as tf #텐서플로우 기능을 사용하기위해 불러오자.
>>> hello = tf.constant('hello, Tensorflow!')
>>> sess = tf.Session()
>>> print(sess.run(hello))
hello, TensorFlow!
>>> a = tf.constant(16)
>>> b = tf.constant(12)
>>> print(sess.run(a+b))
28
>>> exit()
```



### Graph 만들기

파이썬 라이브러리의 작업 생성 함수는 만들어진 op들의 결과값을 반환하는데, 변한된 작업들의 결과값은 다른 op를 생성할 때 함수의 입력값으로 이용할 수 있다. [Graph class](http://www.graphclasses.org/classes.cgi) 에서 명시적으로 관리하는 방법을 볼 수 있다.

```python
import tensorflow as tf

marix1 = tf.constant([[2.,2.]])
marix2 = ft.constant([[3.,3.]])

product = tf.matmul(marix1,marix2)

```

 Default graph에는 3개의 node가 있고, 2개는 constant op이고 하나는 matmul op입니다. 하지만 행렬을 곱해서 결과값을 얻으려면 sess에 graph를 실행해야한다.

```python
sess = tf.Session()

result = sess.run(product)
print(result)
>>>[[12.]]
# 실행을 완료하였습니다
#session close

sess.close()
```



 연산에 쓰인 시스템을 돌려보내려면 session을 닫아야한다. 시스템을 더 쉽게 관리 하기위해서는#with를 써주면 된다. 각 Session에 with가 있으므로써 구문 끝에서 자동으로 close()가 호출되게 된다.

```python
with tf.Session() as sess:
    result = sess.run([product])
    print(result)
```

```python
with tf.Session() as sess:
    with tf.device("/gpu:1"): #컴퓨터의 2번째 GPU
        matrix1 = tf.constant([[2.,2.]])
        matrix2 = tf.constant([[3.,3.]])
        product = tf.matmul(matrix1,matrix2)
```



#### Interactive Usage

파이썬 예제들은 Session을 실행시키고 Session.run() 메서드를 이용해서 graph의 작업을 처리한다.!

```python
import tensorflow as tf
sess = tf.InteractiveSession()

x = tf.Varible([1.0, 2.0])
y = tf.constant([3.0,3.0])

x.initializer.run()

sub = tf.subtract(x, a)
print(sub.eval())

>>> [-2, -1]

sess.close() #!!!
```



### Tensors

tensorflow 프로그램은 모든 데이터를 tensor 데이터 구조로 나타낸다. 연산 graph에 있는 op간엔tensor만 주고받을 수 있기 때문인데, tensorflow의 tensor를 n 차원의 배열이라고 보면 된다.
Static type,rank,shape 값을 가지고있다.

### Variables

그래프를 실행하여도 variable의 상태가 유지가 된다.

#Count example

```python
state = tf.Variable(0, name="counter")

one = tf.constant(1)
new_value = tf.add(state, one)
update = tf.assign(state, new_value)

init_op = tf.global_variables_initializer()

with tf.Session() as sess:
  sess.run(init_op)
  print(sess.run(state))
  for _ in range(3):
    sess.run(update)
    print(sess.run(state))

>>> 
0
1
2
3
```

호로록 #1 완료

