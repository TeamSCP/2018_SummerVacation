# Macine Learning

Rev1r, 정재훈

team S.C.P

2018.07.05

------

[TOC]

------

## 1. Why should we learn ML?

  머신러닝에 대한 설명에 앞서 먼저 ''왜 우리는 머신러닝을 배워야 할까?'' 라는 의문이 들어야 합니다. 사실 해킹적 관점에서 머신러닝은 그저 개발자의 미래만으로 생각 할 수도 있습니다. 하지만, S/W엔지니어와 해커의 관계가 그렇듯, 그렇게 멀찍이 볼 주제는 아니라고 생각합니다. 또한, 해커라면, 컴퓨터 상에서 정말 뭐든 자유롭게 하는 것이 궁극적인 목표지 않을까 합니다. 그렇게 자유로운 해커가 되기 위해서는 정해진 길이 있다기 보단 다양한 것을 접해보고 다양한 것을 경험해보는 것이 해커로서 가장 중요하다고 생각합니다.

## 2. What is ML : Machine Learning

  머신러닝이 무엇일까요? 머신러닝이 나타나기 전에는 어떤방식으로 인공지능을 구성하려 했었을까요? 머신러닝이 있기 전에는 상황마다 규칙을 적용해서 프로그래밍 합니다. 하지만, 인공지능은 그 상황들을 학습하여 규칙들을 스스로 생성하게 됩니다. 

### 2-1. Explicit Programming

머신러닝을 이해하기 이전에 먼저 Explicit Programming을 이해해야 합니다.  위에서도 얘기했듯이 머신러닝이 있기 전에는 상황마다 규칙 즉, if문이나 switch/case문을 통해 예외를 처리하고 새로 발생하는 문제가 있을 때, 프로그래머는 그 규칙을 계속해서 만들어 나갔습니다. 그것이 바로 Explicit Programming 입니다.

**하지만, 그렇게 되면 같은 상황(input)에 대해 도출되는 결과 값(output)이 언제나 동일하게 됩니다.**

Explicit Program의 예로, Starcraft1의 Computer를 생각할 수 있는데요. 게임이 시작 되자마자 컴퓨터의 넥서스를 한대 치고 도망가게 되면, 컴퓨터의 모든 일꾼이 따라오게 됩니다. 자, 이제 우리는 생각해 볼 수 있는데요. `적을 인식함 -> 공격을 당함 -> 상대를 공격함` 과 같이 프로그래밍 되어있을 것입니다. 자원을 캐고있던 일꾼이 모두 적을 공격하러 가게되면 적의 일꾼은 자원을 잘 캐고 적은 성장을 잘 하게 되는 반면에 컴퓨터의 일꾼은 자원을 캐는것을 일시 중지하고 적을 공격하게 되어서, 성장이 중지되어 결국엔 컴퓨터가 지게되고, 적이 승리하게 될 것입니다. 그 적이 사람이라면, 필승법이라는 것이 생기게 된 것이라고 생각할 수 있겠죠? 

그럼 이제 우리가 블리자드의 프로그래머가 되었다고 생각 해봅시다. `공격을 당함 -> 내가 가진 일꾼이 충분 한가? -yes-> 한 마리 정도만 공격을 저지하러 감` 과 같은 규칙을 추가해주어야 할 것입니다.

하지만, 이런 상황이 정말 많이 발생될 것입니다. 이런 모든 상황에 대해 규칙을 추가해주고 수정하는 것이 가능 할까요? 거의 불가능 할 것입니다. 그래서 나온 것이 바로 Machine Learning 입니다.

### 2-2. Machine Learning

서론이 길었습니다. 이제 머신러닝이 무엇인지 알아봅시다.

앞에서도 어느정도 설명 했듯이, 머신러닝은 Explicit programming 을 넘어서 단순 자동화가 아닌 컴퓨터가 학습을 하여 규칙을 스스로 생성해 나가게 하는 것을 말합니다.

머신러닝에는 여러가지 학습 방법이 있습니다. Supervised/Unsupervised/Reinforcement Learning. 이렇게 세 가지의 학습방법이 있습니다. 이 문서에서는 Supervised Learning 에 관해서 다룰 것이고, 추후의 문서에서 다른 학습방법 또한 다룰 예정입니다.

## 3. Supervised Learning

Supervised Learning에 대해 간략히 설명 드리자면, 학습할 기반 데이터가 존재 할 때 Supervised Learning 이라고 합니다. 즉, 트레이닝 과정을 거칠 수 있으면, Supervised Learning 이라고 하는 것 입니다.

### 3-1. Type of Supervised Learning

Supervised Learning은 세가지 종류로 나뉘게 됩니다. 어떤 것을 모델링 하냐에 따라 나눈 것입니다.

1. Regression Model : 우선순위가 없는, 넓은 범위의 데이터 중 하나를 결과 값으로 도출해 내는 모델
2. Binary Classification Model : 0 과 1/ 참과 거짓/ 가능 불가능 과 같은 것을 결과 값으로 도출해 내는 모델
3. multi-labels Classification Model : 우선순위가 존재 하는, 예를 들어 학점과 같이 A~F 와 같은 데이터를 도출해 내는 모델

위와 같이 세가지의 모델링 방식으로 나뉘게 됩니다. 

## 4. Predict Result via Regression Data

제가 만약 하스스톤을 플레이 했다고 가정을 해볼까요? (돌크리트 ㅈㅅ) 

| X(play times) | 1    | 2    | 3    | 4    | 8    | 9    |
| ------------- | ---- | ---- | ---- | ---- | ---- | ---- |
| Y(wins)       | 4    | 9    | 12   | 17   | 35   | 38   |

다음 데이터는 하스스톤을 한 시간이 있고 그 만큼 플레이 했을 때의 승 수를 내타낸 표입니다.  도출 될 결과값이 광범위한 값이기 때문에  이 표는 regression data라고 할 수 있습니다. 

우리가 이 데이터를 통해 학습을 시켜 궁극적으로 알고싶은 것은 내가 몇시간 만에 몇번을 이길 수 있을까에 대한 것입니다. 이 데이터를 가지고 트레이닝 과정을 거치면 regression model 속에서 어떤 규칙을 만들겁니다. 그럼 이제 그 규칙을 통해 난 5시간을 플레이 할건데 몇번을 이길 수 있을까? 라는 질문을 던지면, 21 정도가 나오게 된다는 것이죠! [^1]

### 4-1. Learning!

이제 이 데이터를 통해 학습을 시킬겁니다. 학습을 하기 전에 사람이 무언가를 학습하고 추론 할 때의 사고 과정을 생각 해 봅시다. 먼저, 위와 같은 데이터들을 바탕으로 가정을 합니다. 정말 의식의 흐름기법을 이용해 생각을 해봅시다.

1시간하면 4번 이기네?

2시간 하면 9번 이기네? 1시간에 4번이라서 8번 이길 줄 알았는데 9번을 이기네?

3시간 하면 12번 이기네? 1시간에 4번이지만 2시간을 플레이하면 9번을 이겼으니까 3시간하면 13 또는 14번 이길 줄 알았는데 그냥 12번만 이길 수 있네? 그럼 1시간에 4번 이길 수 있지만 간헐적으로 한두번 정도 더 이길 수도 있겠구나!

4시간 하면 17번 이기네? 3번에서 예상한 것 처럼 보통 1시간에 4번가량 이기고 간헐적으로 한두번 저도 더 이기는구나!

**그럼 5시간 플레이 하면, 몇 번 이길 수 있을까?**

와 같은 방식으로 기계도 학습을 하고 추론을 하게 됩니다. 먼저 Hypothesis Function에 대해 알아보겠습니다.

### 4-2. Hypothesis Function

Hyptheesis Function 을 직역하면 가설 함수 입니다. 위 데이터를 좌표상에 나타내 보면 다음과 같습니다.

![img](https://user-images.githubusercontent.com/40850499/42409936-aa403598-821c-11e8-9ceb-4c0ee8eed280.png)

이제 우리는 선형의 가설을 세울 것입니다. 선형적이라는 것은 직선적이라는 것인데요, 좌표상에 직선을 몇 개 그어보겠습니다.

![img](https://user-images.githubusercontent.com/40850499/42409937-abdb1148-821c-11e8-9913-33e15df8846a.png)

저는 지금 3개의 가설을 세웠습니다. 123번 중에 어떤 것이 가장 적절한 가설일까요?

맞습니다. 바로 3번 가설이 가장 적절한 가설입니다. 점들과 거리가 가장 가까운 직선이 가장 적절한 가설 이라고 생각하신 것 이겠죠?

![img](https://user-images.githubusercontent.com/40850499/42409939-ad0d1d22-821c-11e8-9add-1eddc7452b93.png)

이 처럼 결과가 선형적일 것이다라고 가정 하고, 데이터를 통해 이런 선형 그래프를 계속 조정해 가면서 가장 적절한 선형 그래프를 찾는 것이 머신러닝의 학습 방법입니다. 이런 적절한 그래프를 찾는 것은 생각보다 간단 합니다. 일단 가설함수가 선형적이기 때문에 일차함수로 나타낼 수 있습니다.

![img](https://user-images.githubusercontent.com/40850499/42409940-ae4c02fc-821c-11e8-8b8f-4d9d311c6e79.png)
$$
H(x)=Wx+b
$$
H는 Hypothesis, W는 weight 그리고 b는 bias 를 의미합니다.

### 4-3. Cost Function (Loss Function)

Cost Function 을 직역하면 비용 함수 입니다. Loss 함수라고도 합니다. 이 코스트 함수는 가설 함수가 얼마나 적절한지를 판단해 주는 함수 입니다. 

글로써 표현하자면, 각 데이터에서 가설함수와의 거리를 구하고 그 거리의 제곱 한 것을 모두 더하고 그 개수만큼 나눈 함수입니다. 식으로 표현하면 다음과 같이 나타낼 수 있습니다.
$$
cost(W,b)=\frac{1}{m} \sum_{i=1}^m(H(x_i)-y_i)^2
$$
위 함수를 W에 대해 미분하고,  (0.1 정도의 값)를 곱해 W에서 빼는 것을 계속 반복을 하게되면 cost함수의 최솟값을 찾을 수 있게 됩니다. 이 알고리즘을 Gradient Descent Algorithm이라고 합니다. 
$$
W <= W-\alpha\frac{\partial}{\partial W}cost(W,b)
$$
위와 같은 알고리즘을 이용해 cost함수의 최솟값을 찾을 수 있습니다. 그 때의 W와 b를 가지고 다시 가설함수를 설정하면 우리는 학습을 시킨 것입니다.

## 5. implementation via Tensorflow

> IT분야가 다 그렇듯 알고리즘은 소수의 [천재](https://namu.wiki/w/%EC%B2%9C%EC%9E%AC)가 만들어내고 대부분의 엔지니어는 그것들을 적재적소에 활용하는 역할을 맡는다. 알려진 알고리즘들은 이미 함수로 구현까지 끝나 있다. 따라서 이 문서를 보고 있을 대부분의 개발자들은 각 알고리즘의 장점과 단점을 외우고 호출방식을 익히는 것이 실용적인 면에서 훨씬 중요하다.
>
> ​	- in 꺼무위키[^2]

네. 맞습니다. 본인이 천재일 거란 생각은 금물이기도 하고, 설령 당신이 천재라면 이 문서를 보고 공부하진 않을거에요.[^3] 결론적으로 위에 것들 수학이고 함수고 다 필요 없고 구현만 할 수 있어도 당신은 인공지능을 구현 할 수 있다~~~~~~~ 이말입니다.

### 5.1 How to use Tensorflow

우선 당신의 컴퓨터에 Python 3.6.x 64bit 와 tensorflow가 설치되어 있는 것을 전제로 설명 하겠습니다. 설치방법은 구글링 하세요. 

```python
import tensorflow as tf

#x and y data
x_train = [1,2,3,4,8,9]
y_train = [4,9,12,17,35,38]

W=tf.Variable(tf.random_normal([1]), name='weight')
b=tf.Variable(tf.random_normal([1]), name='bias')

# Our hypothesis XW + b
hypothesis = x_train * W + b

#cost function
cost = tf.reduce_mean(tf.square(hypothesis - y_train))

#Minimize
optimizer = tf.train.GradientDescentOptimizer(learning_rate=0.01)
train = optimizer.minimize(cost)

sess = tf.Session()
sess.run(tf.global_variables_initializer())

for step in range(10000):
    sess.run(train)
    if step % 200 == 0:
        print(step, sess.run(cost), sess.run(W), sess.run(b))

```

예제 소스코드를 먼저 보여드리면서 코드 설명을 드리겠습니다. 사용법을 아시거나 급하신 분들은 소스코드 복+붙 해서 빠르게 지나가시면 되구요. 그럼 이제부터 소스코드 설명 들어갑니다.

```python
import tensorflow as tf
```

와 같이 `tensorflow` 라이브러리를 `import`[^4] 해줄 수 있습니다. `as tf`를 추가하여 `tensorflow.메서드` 처럼 사용 할 것을 `tf.메서드` 와 같이 사용할 수 있습니다.

```python
x_train = [1,2,3,4,8,9]
y_train = [4,9,12,17,35,38]

W=tf.Variable(tf.random_normal([1]), name='weight')
b=tf.Variable(tf.random_normal([1]), name='bias')

hypothesis = x_train * W + b
```

`Variable` 메서드를 호출하면 텐서플로 내부의 그래프 자료구조에 만들어질 하나의 변수를 정의하게 된다고 이해하면 됩니다. 나중에 이 메서드의 매개변수에 대해 더 자세히 보도록 하겠습니다. 지금은 예제를 완성하기 위해 다음 단계로 넘어가겠습니다.

이 변수들을 이용해서 앞서 얘기했던 대로 실제 값과 H(x)=W*x+b로 계산한 값 사이의 거리를 기반으로 비용함수를 만들겠습니다. 이 거리에 제곱을 하고. 그 합계에 대한 평균을 냅니다. 텐서플로에서는 이 비용함수를 다음과 같이 나타냅니다.

```python
cost = tf.reduce_mean(tf.square(hypothesis - y_train))
```

이 코드는 우리가 이미 알고 있는 값 `y_train`과 입력 데이터 `x_train`에 의해 계산된 `hypothesis`값 사이의 거리를 제곱한 것의 평균을 계산합니다.

이 전에 설명한 Gradient Descent Algorithm 은 다음과 같이 구현되어 있습니다.

```python
optimizer = tf.train.GradientDescentOptimizer(learning_rate=0.01)
train = optimizer.minimize(cost)
```

현재 이 코드에서 알아야 할 것은 두 가지입니다. 하나는 이 코드를 실행함으로써 텐서플로가 내부 자료구조 안에 관련 데이터를 생성했다는 점, 또 하나는 이 구조 안에 train에 의해 호출되는 Optimizer가 구현되었으며, 이 Optimizer는 앞에서 정의한 비용함수에 Gradient Descent Algorithm을 적용한다는 점입니다. 이 예제에서 0.01로 지정한 매개변수는 학습 속도라고 합니다. 즉, 비용 함수의 경사면을 따라 내려가는 속도로 볼 수 있습니다.

이제 알고리즘 실행을 해보겠습니다. 세션을 생성하고 `run`메서드에 `train`매개변수를 넣어 호출해야 합니다. 또한 앞에서 변수들을 선언했으므로 다음과 같이 이들을 먼저 초기화해야 합니다.

```python
sess = tf.Session()
sess.run(tf.global_variables_initializer())
```

이제 반복적인 프로세스를 실행합니다. 

```python
for step in range(10000):
    sess.run(train)
    if step % 200 == 0:
        print(step, sess.run(cost), sess.run(W), sess.run(b))
```

제 소스코드는 10000번 학습시키고 200번마다 상태를 출력시키도록 작성하였습니다.

![1](https://user-images.githubusercontent.com/40850499/42410601-fb050ef4-8226-11e8-869b-48d0aea8852f.png)

### 5.1 Result

결과적으로 1800번 알고리즘을 반복한 이후로 변동이 없는 것을 알 수 있었습니다. 그래프로 나타내면

![image](https://user-images.githubusercontent.com/40850499/42410613-356934f8-8227-11e8-8226-86484713c251.png)

![image](https://user-images.githubusercontent.com/40850499/42410616-4e2d3e26-8227-11e8-98fa-1fcab1217f5f.png)

위와 같고 하스스톤을 5시간 플레이 한다면 약 21번 이길 수 있다는 어마어마하고 굉장하고 엄청난 결과가 나왔습니다. [^5]



[^1]: 물론 하스스톤은 굉장히 운빨x망겜이고, 덱에 따라 승률이 많이 갈립니다만 제한적으로 가정한 것 입니다.
[^2]: 꺼무위키 나라
[^3]: 이 문서는 다소 팩트로 후드리 챱챱하는 경향이 있습니다. 주의하세요.
[^4]: 한국어로 뭐라 하죠? 
[^5]: 짝짝짝~~ 