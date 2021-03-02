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

-----------------------------------------------------

> 이 글은 이 논문[^1]을 다룬다.

대부분의 기존 추천 시스템은 뉴스 추천을 위해 정적인 접근법을 사용한다. 예를 들어, 이 모델들은 사용자의 행동이 연속적임을 고려하지 않는다. 두번째 문제점은 기존 방법들이 주로 단편적이라는 것인데, 이는 오래 걸리는 보상에 집중하지 못한다는 의미이다. 이 문제는 감마 값을 높게 설정해 쉽게 해결할 수 있지만, 그러는 경우가 드물다.

### Embeddings Recap

임베딩은 어떤 대상이 다른 대상들과의 유사도를 알아내기 위한 해석의 도구로 사용된다. 톨스토이의 책들은 서로 매우 유사하지만, 히치하이커의 책과는 매우 다르다. 대상들은 벡터로 표현되며 유사도는 그들의 내적으로 계산된다.

```
books = ["War and Peace", "Anna Karenina", 
          "The Hitchhiker's Guide to the Galaxy"]
books_encoded_ideal = [[0.53,  0.85],
                       [0.60,  0.80],
                       [-0.78, -0.62]]
Similarity (dot product) between First and Second = 0.99
Similarity (dot product) between Second and Third = -0.94
Similarity (dot product) between First and Third = -0.97
```

### Reinforcement Learning Recap:

![rl](https://miro.medium.com/max/700/1*q1shfEiQDYZ1DaFmIjlHBA.png)

강화학습에 대한 대략적인 소개: 환경(environment)에서 행동(act)하는 주체(agent)가 있다. 행동은 그에 따라 점수 형태의 보상(reward)가 주어진다. 환경은 주체에게 상태(state)를 제공하며 행동을 받고, 다음 단계로 넘어간다. 이 모든 것들이 마르코프 결정 과정 (Markov Decision Process, MDP) 이다. 기본적으로, 여기에는 두가지 예측 방법이 있다.: 정책(policy)과 Q-학습(Q-Learning). Q-학습은 신경망이 다음과 그 이후를 위해 최고의 보상을 예측하도록 학습한다 (참고로 보상은 행동에 따라 다르다.). 하지만, 정책 학습은 다음 행동의 학률을 예측하도록 학습한다.

### Markov Representation:
* 상태 S. 상태 s는 사용자의 긍정적인 상호작용 히스토리를 나타낸다.
* 행동 A. 행동 a는 랭킹 스코어들의 벡터이다. 이 행동은 사용자가 읽고싶어 하는 이상적인 뉴스이다.
* 상태변화 P. 상태는 사용자의 긍정적인 상호작용 히스토리의 표현으로 모델링된다. 그래서, 사용자의 피드백이 수집되면, 상태 변화가 정의된다.
* 보상 R. 행동 a와 상태s에 의해 추천이 되면, 사용자는 (클릭을 하거나 안하거나, 평점을 매기는 등의) 피드백을 한다. 추천 모델은 즉시 사용자의 피드벡에 따른 보상 R(s, a)를 받는다.
* 할인율 γ. γ ∈ [0, 1] 는 long-term 보상을 측정하기 위한 지수이다.

### Previous methods comparison

Q-학습의 문제점은 전반적인 행동에 대한 평가로 인해 행동 범위(action space)가 넓으면 매우 비효율적이다. 정책 학습은 넓은 행동 범위에서도 수렴하는 반면, 연속적인 특징들을 이해할수는 없다.

### How it works

사용자의 행동을 관찰하고 그들이 본 뉴스를 수집한다고 가정하자. 이것은 다음에 무엇을 읽을지 결정하는 액터(Actor) 네트워크로 들어간다. 이 네트워크는 이상적인 뉴스 임베딩을 생성한다. 또한 다른 뉴스 임베딩과 비교해 유사도를 구할 수 있다. 그 중 가장 잘 맞는 하나를 사용자에게 추천할 것이다. 크리틱(Critic)은 액터가 판단하는 것을 돕고, 무엇이 틀렸는지 찾는다. 이는 또한 미래의 행동을 예측하는데 도움이 된다. 예를 들어, 추천 모델이 사용자에게 책을 사는 것을 제안했다. 사용자는 책을 샀고 즉각적으로 100의 보상을 받았다. 하지만 사용자가 미래에 책을 환불해 -500의 벌점을 받을수도 있다. 모든 미래의 행동들도 고려대상이어야 한다. 그래서 Q-네트워크 (크리틱) 처럼, 정책 학습은 주로 근시안적이고 미래의 보상을 예측할 수 없다.

### Architecture
![drr](https://miro.medium.com/max/700/1*EdfGWMhXskA1XR97bG-VKg.png)

위 그림에서 같이, 이 네트워크는 두개의 층으로 구성된다.: 액터와 크리틱. 각각은 서로 다른 학습 유형을 따른다. 액터는 정책 (어떤 행동을 다음에 고를지에 대한 확률) 을 학습하고, 크리틱은 보상에 집중한다 (Q-학습). 우선, 뉴스 임베딩들이 액터의 상태 표현 모듈에 입력돼 인코딩 된다. 그러면 결정(행동)이 벡터 형태로 생성된다. 뉴스 임베딩들과 함께, 이 행동은 크리틱 모듈에 입력돼 보상이 얼마나 좋을지 예측하는데 쓰인다. 


---
{: data-content="footnotes"}
[^1]: [Deep Reinforcement Learning based Recommendation with Explicit User-Item Interactions Modeling](https://arxiv.org/pdf/1810.12027.pdf)