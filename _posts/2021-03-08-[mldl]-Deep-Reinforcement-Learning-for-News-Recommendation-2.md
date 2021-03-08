---
layout: post
category: develop
comments_id: 2
---
[link](https://towardsdatascience.com/deep-reinforcement-learning-for-news-recommendation-part-1-architecture-5741b1a6ed56)  
toward data science, medium blog  
[Mike Watts](https://medium.com/@awarebayes)

Dec 28, 2018

이 글은 비공식 번역문 입니다.

*[previous post 1](https://yongqyu.github.io/mldl-deep-reinforcement-learning-for-news-recommendation-1.html)*  

-----------------------------------------------------

> 아래 글은 이 논문[^1]을 다룬다.

### The State Module

더 좋은 추천 성능을 위해 복잡하고 동적인 사용자-아이템 관계를 명시적으로 모델링하는 상태 표현 모듈을 먼저 거친다. 쉽게 말해, 임베딩을 뉴스나 고객 대신 문장 속 단어의 의미를 찾아내는 것으로 생각할 수 있다. 이것은 두가지 핵심적인 역할을 한다.: 랭킹과 크리틱 네트워크의 입력으로 행동들을 통합한다.

### The Actor Network (Policy)

한 사용자에 대해, 액터 네트워크는 그 사용자의 상태 s를 기반으로 행동을 생성한다. 사용자가 양의 반응을 한 최근 n개의 아이템을 기반으로 임베딩된 사용자 상태가 입력으로 사용된다. 이는 사용자의 상태에 대한 종합적인 표현을 위해 상태 모듈에 입력된다. 그 후, 이 표현은 (-1, 1) 범위의 n개의 벡터 형태인 행동으로 변형된다.

저자는 랭킹 스코어를 위해 소프트맥스를 쓰지 않았다.:

> 우리 모델에서 설명했듯, 아이템의 추천이나 아이템 리스트의 추천과 행동은 대응하지 않는다. 대신, 행동은 연속된 파라미터 벡터이다. 이런 행동을 위해, 파리미터 벡터는 아이템 임베딩과 곱해서 모든 후보 아이템의 랭킹 점수를 구하는데 사용된다.

### The Critic Network (Q-Learning)

현재 상태에 대한 액션: Qπ (s, a) 이 얼마나 그럴듯 했는지를 평가하는데 사용된다. 크리틱 네트워크의 입력은 사용자 상태 표현 모듈에 의해 생성된 사용자 상태 s와 정책 네트워크에 의해 만들어진 행동이고, 출력은 벡터 형태의 Q-값이다. Q-값에 의해, 엑터 네트워크의 파라미터들은 행동 a의 성과를 개선시키는 방향으로 업데이트 된다. 손실 함수로는 순수한 MSE가 사용되는데, 이는 일반적으로 정규화되지 않은 실제 값의 보상을 예측하기 위함이며, 고로 이는 회귀 문제이다.

### The State Module Zoo

![drrp](https://miro.medium.com/max/624/1*ZYqXcmk5UA7kNwLP6KueWw.png)

DRR-p — 아이템들의 쌍 간에 의존성을 활용한다. 원소간의 곱 연산을 통해 n개의 아이템간 상호작용을 계산한다. (사용자-아이템 간의 상호작용은 사용되지 않는다.)

![drrp](https://miro.medium.com/max/700/1*8o6nbuIpZ4LHjh-fwp-U4Q.png)

DRR-u, 사용자 임베딩이 역시나 통합된다. 이이템들간의 지역적인 의존성에 더해, 사용자-아이템간의 상호작용 역시 고려된다.

![drrave](https://miro.medium.com/max/700/1*37BAqsTPbIBPfzj4Krlk3g.png)

만약 상당히 긴 간겨의 뉴스 배치를 활용하면, 문제가 될 위치를 찾기가 어렵다. 하지만 만약 순열 H가 짧은 주기라면, 아이템들의 위치를 기억해 과적합으로 이어진다. Average pooling 층이 적용됐기 때문에, 이 구조를 DRR-ave라고 부른다. 그림에서 보듯, H의 아이템 임베딩들이 먼저 weighted average pooling 층에 의해 변형된다. 그 결과의 벡터는 입력된 유저와의 상호작용을 모델링하기 위해 사용된다. 최종적으로, 유저의 임베딩과 상호작용 벡터, 그리고 아이템들의 average pooling 결과가 하나의 벡터로 연결돼 상태 표현을 나타낸다.
