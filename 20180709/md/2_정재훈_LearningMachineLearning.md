# Learning Machine Learning

Rev1r, 정재훈

team S.C.P

2018.07.12

------

[TOC]

------

<img src="" width="50%" align="center"></img>

## 1. Review

이 전의 문서를 대충 훑고 넘어오셨다면, 반드시 보셔야 하는 부분이옵니다.

<img src="https://user-images.githubusercontent.com/40850499/42629033-f8347efa-860c-11e8-9276-48c9f36a3bad.png" width="40%" align="center"></img> 

Machine Learning 은 세 가지 학습(Supervised/Unsupervised/Reinforcement Learning) 으로 나뉩니다.

<img src="https://user-images.githubusercontent.com/40850499/42629610-d7bc7f0e-860e-11e8-9637-bbaa2e51009e.png" width="90%" align="center"></img>

그 중 Supervised Learning 은 세 가지 모델을 생성 할 수 있습니다. 저번에도 하스스톤으로 예를 들었듯이[^1] 이번에도 하스스톤을 예로 들 것입니다. 

<img src="https://user-images.githubusercontent.com/40850499/42629671-221ee0fa-860f-11e8-8433-e7ad5f29b1e3.png" width="90%" align="center"></img>

예를 들어 플레이 한 시간(x1)이 있고 덱 파워(x2)가 있다고 하고 위와 같은 데이터가 존재한다고 하면, 승 수(Wins) 같은 경우에는 정말 무한대의 값 중 하나의 값을 갖게 됩니다;Logistic Classification. 만약 한 사람이 전설인지 아닌지에 대한 데이터를 추출 한다면, 0과 1 로 표현이 가능할 것입니다;Binary Classification. 그리고 등급에 따라 나뉘면, 0이 전설이라고 할 때, 25급~1급 그리고 전설(0)로 표현이 가능 합니다;Multi-label Classification.

## 2. Binary Classification 

### 2-1. Binary Classification via Linear Hypothesis Function

이제 Binary Classification Model 을 생성할 것이다. Binary Classification Model 역시 Regression Model 처럼 선형의 가설함수로 데이터를 분리할 수 있습니다.

아래와 같은 데이터들이 있다고 하자면,

<img src="https://user-images.githubusercontent.com/40850499/42630921-8f5aa2ae-8613-11e8-9c27-9bd316086111.png" width="50%" align="center"></img>

아까 전에 하스스톤에 대한 표에서 가로 축은 하스스톤을 플레이한 시간(단위 : 년) 이라고 하고, 세로 축은 전설을 달성했는가 아닌가에 대한 자료라고 합시다. 그럼 이 자료들은 Regression Model에서 처럼 H(x)=Wx+b 와 같이 가설함수를 세우고 cost함수를 이용해 최적의 함수를 찾을 수 있습니다.

<img src="https://user-images.githubusercontent.com/40850499/42630760-ed402bc4-8612-11e8-91b1-f76525e07de4.png" width="50%" align="center"></img>

그 최적의 선형 가설함수를 찾았을때 0.5를 기준으로 왼쪽에 있으면 0 오른쪽에 있으면 1로 생각할 수 있다. 하지만, 만약에 50년을 플레이한 사람이 있다고 해봅시다. (물론 하스스톤이 나온지 50년은 커녕 10년도 되지 않았지만 가정이니까 그냥 넘어가자.) 그럼 그래프는 다음과 같이 그려질 것입니다.

<img src="https://user-images.githubusercontent.com/40850499/42719667-a982d2a4-8754-11e8-9083-e46ca3955c41.png" width="50%" align="center"></img>

이렇게 되면 5,6,7,8은 1임에도 불구하고 0이라는 값으로 예측 될 수 있다. 즉, 너무 동떨어져 있는 값 때문에 모델이 제대로 형성되지 않은 것입니다.

### 2-2. Sigmoid Function

그래서 Sigmoid 함수를 사용하여 모델이 잘못 형성되는 것을 막을 수 있습니다. sigmoid 함수를 이용해 다음과 같이 수정할 수 있습니다.

<img src="https://user-images.githubusercontent.com/40850499/42721659-70c8dccc-8779-11e8-85e7-c469e738931e.png" width="50%" align="center"></img>

이런식으로 sigmoid 함수를 이용하면 더 적합한 가설함수를 설계할 수 있습니다. sigmoid 함수를
$$
g(z)=\frac{1}{1+e^z}, z=H(x)=Wx+b
$$
이렇게 표현할 수 있습니다. Binary Classification이 아닌 Logistic Classification일 때는 행렬로 나타낼 수 있기 때문에 H(x)를 가설 함수를 다음과 같이 설계할 수 있습니다. 
$$
H(X)=\frac{1}{1+e^{W^tX}}
$$
sigmoid 함수를 이용하게 되면 무조건 **0 에서 1 사이의 값**으로 H(x) 값이 나오게 됩니다. 따라서 이를 이용해 모델링이 이상해지는 것을 막을 수 있게 되는 것입니다.

### 2-3. Cost Function

자, 이제 sigmoid에 대한 cost 함수를 설계해야 합니다. 이전 cost함수를 그냥 사용했다가는 험한 꼴 납니다.

식을 도출하는 것은 생각보다 어렵지 않습니다. H(x)값(예측 값)이 1일때 y 값(실제 값)이 1이면 cost함수가 0

0이면 잘못 예측이 된것이므로, cost함수가 &infin;값을 갖도록 설계하면 됩니다. 식으로 표현해봅시다.

<img src="https://user-images.githubusercontent.com/40850499/42731162-78f7c5ec-8842-11e8-8de4-9832417e90e9.png" width="50%" align="center"></img>

위와 같이 전개해서 볼 수 있는데요. 미분과정은 wolfram alpha 추천드립니다.

## 3. Softmax classification:Multinormial classification

여러개의 클래스가 있을 때, 그것을 예측하는 Multinormial classification[^2]. 그 중에서도 가장 많이 사용되는 Softmax classification에 대해 알아보겠습니다.

결론적으로 우리가 해야할 것에 대해 새롭게(?) 생각을 해 봅시다. 하스스톤으로 예를 들었듯이 x_1(play time)과 x_2(deck power)가 있고 이에 대해 전설을 달고 못달고에 대한걸 좌표평면상에 나타내 보면 두 군집으로 나눠질 것입니다.

<img src="https://user-images.githubusercontent.com/40850499/42731255-4226a3a6-8844-11e8-872d-fabf6ec53b9a.png" width="40%" align="center"></img>

예를 들어 아무리 하스스톤을 많이 해도 덱이 쓰레기이면 전설을 달지 못할 수 도 있다라는 결론이 나오는 것이죠.

결론적으로 저 두 군집을 나누는 선이 필요한 것입니다. 그것이 바로 Binary Classification 이었던 것입니다.

<img src="https://user-images.githubusercontent.com/40850499/42731281-c530b200-8844-11e8-8b93-7959a492172a.png" width="40%" align="center"></img>

<img src="https://user-images.githubusercontent.com/40850499/42731428-22f79b1c-8848-11e8-943f-950fec8511af.png" width="20%" align="center"></img>

그걸 위와 같이 나타낼 수 있습니다. x데이터가 w(웨이트 값)와 곱해져 그것이 sigmoid 함수를 통해 결과값으로 도출이 되는 것입니다.

### 3-1. Multinormial Classification

여러개에 데이터를 나눌때에도 똑같이 Binary Classification을 이용할 수 있는데요. 만약 A, B, C에 대한 데이터가 있다면, A or Not, B or Not, C or Not 과 같이 구분하는 선으로 나타낼 수 있는겁니다.

<img src="https://user-images.githubusercontent.com/40850499/42731476-e602c3b6-8848-11e8-8581-02926d352100.png" width="80%" align="center"></img>

<img src="https://user-images.githubusercontent.com/40850499/42731489-106268b4-8849-11e8-93a2-44091b9e8fae.png" width="60%" align="center"></img>

<img src="https://user-images.githubusercontent.com/40850499/42731505-42c44886-8849-11e8-89f5-f67e70b8bda2.png" width="50%" align="center"></img>

그렇게 구분하는 선이 여러개면, 그걸 행렬곱으로 표현할 수 있고 그걸 하나로 나타낼 수 있습니다. 그리고 그렇게 했을 때 도출 되는 값을 sigmoid 함수에 넣어 0에서 1사이의 값으로 바꿔줍니다.

<img src="https://user-images.githubusercontent.com/40850499/42731516-79ecefac-8849-11e8-87d0-7741af963e03.png" width="40%" hight="80%" align="center"></img>

그럼 최종적으로 가장 가중치가 높은 값이 최종적으로 예측된 값입니다.

## 4. NN : Nural Network

위에서 공부한 일련의 과정들이 모두모여 첫 input값이 어떤 값으로 도출되고 또 그것이 다음 Layer의 input값이 되면, 그것이 신경망처럼 작용하게 됩니다. 그것을 활용한 것이 인공신경망이고, 머신러닝이 이루어지는 과정입니다.

<img src="https://user-images.githubusercontent.com/40850499/42731542-3050bf08-884a-11e8-8dab-0ddc0ef0a6a4.png" width="50%" align="center"></img>

------

[^1]: 하스스톤은 굉장히 운빨ㅈ망겜이라고 하지만 실력갓흥겜일 때가 있다. 바로, 1티어 덱을 사용하면 된다. 하지만 당신이 아만보라면 아쉽지만 전설을 달 수 없다.
[^2]: 1장 보면 Multi-label Classification 이라고 되어있는데 3장에서 왜 Multinormial Classification으로 되어있냐고 물어본다면, 나도 공부하는 입장입니다만 아마 ML분야가 아직 정의가 제대로 되어있지 않은 것 같다. 이리저리 말해도 찰떡같이 알아듣는 당신이길 바란다.