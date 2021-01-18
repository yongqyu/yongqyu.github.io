---
layout: post
category: develop
comments_id: 2
---
[link](https://medium.com/the-dl/transformers-viewed-as-generalizations-of-gcns-afe3b595c903)  
Medium Blog  
[Frank Odom](https://frank-odom.medium.com/?source=post_page-----afe3b595c903--------------------------------)  
Dec 13, 2020

이 글은 비공식 번역문 입니다.


-----------------------------------------------------

## Introduction

Transformers 와 Graph Convolutional Networks (GCNs) 은 최근 몇년간 엄청나게 성장했다. 처음 볼땐, 이 두 모델이 매우 다른 듯 했다. 이 모델들이 풀고자 하는 문제도 달랐다.:
* Transformers 는 자연어 처리의 어텐션 베이스 모델에서 발전했다.
* GCNs 은 3D 포인트 클라우드 같은 비정형 데이터를 처리한다.

GCNs 은 자연어 처리에서는 크게 집중 받지 못했지만, 두 모델 모두 다양한 분야에서 응용됐다. 이 글에서는, 이 두 모델이 사실은 매우 유사함을 논할것이다. 사실, GCNs 은 Transformer Encoder 의 변형으로 볼 수 있다. 이것이 어떻게 이들의 해석에 영향을 미치는지, 그리고 이에 따른 Transformers의 활용에 대해 얘기해보자.

> Note: 코드 샘플 없이 높은 수준의 토론이 될 것이다. 이 글을 이해하기 위해 Transformers 나 GCNs의 코드를 이해할 필요는 없다.

## Transformer Encoder

Transformer Encoder 의 구조로 시작해보자. N 층의 구성으로, 각 층은 Multi-Head Attention 과 Feed Forward layer들로 구성된다.

![Trm Enc](https://miro.medium.com/max/552/1*_dSdZfMV8CPOFVv6CWKf3g.png)  
*Source: Original Author*

하지만 Multi-Head Attention 대신에 단순한 Single-Head Attention 을 사용해보자. 이것은 ```num_heads```를 1로 설정하는 것과 같다. 그러면 아래 그림과 같이 Attention module 에서 Q, K, V가 모두 ```(batch_size, seq_length, num_features)``` 꼴이 된다.

![Trm Enc Mod](https://miro.medium.com/max/607/1*YKcSZzALwwpIFBaiaR-0fQ.png)  
*Source: Original Author*

### Graph Convolutional Network

수많은 GCN 변형이 존재하지만, 이 글에서는 "Vanilla GCN" 구조를 사용한다. 앞의 Transformer와 같이, GCN도 N개의 동일한 layer들로 구성된다.:

![GCN](https://miro.medium.com/max/700/1*LO-_s4NYysBS4E5RurO9Ow.png)  
*Source: Original Author*

*Spectral Convolution* 은 GCN 모델의 특징인데, 이 layer는 input vector와 미리 정의한 인접 행렬 을 곱한다. 그래프 데이터에서 인접 행렬은 어떤 노드들이 엣지로 연결되어 있는지를 특정한다. 인접 행렬의 곱셈으로 정보가 효과적으로 이웃 노드들에게 전달된다.

### Simplifying the Transformer

우리는 GCNs 이 Transformers의 특정 케이스로 보이길 원한다. 그러기 위해서는, 점차적으로 Transformer의 구조를 단순화 할 것이다. 단순화는 원래 구조에서 일부 수정으로 재구현 가능하다.

***Simplification #1***
 
먼저 weights를 identity matirx로, biases를 0으로 설정한다. 그래서 linear layer를 단순화 한다. 이를 통해 Single-Head Attention을 매우 단순하게 만들 수 있고, 결국 이것은 Scaled Dot-Product Attention과 같아진다.

![id trm](https://miro.medium.com/max/700/1*EtxmdI15EmkLjLtr-V5yNg.png)  
*Source: Original Author*

***Simplification #2***

만약 *Layer(x)* 의 값이 매우 크면, 우리는 *(x + Layer(x)) ≈ Layer(x)* 라고 근사할 수 있습니다. 정규화는 단순히 residual 연산 후 rescale 하는 것 입니다. 그러므로, 값이 매우 크면 non-residual layer은 residual layer와 결과가 같습니다. 그래서 non-residual layer는 residual layer로 단순화 가능하다. 이를 통해, 우리는 Transformer 구조를 다음과 같이 단순화 할 수 있다.

![trm mod](https://miro.medium.com/max/700/1*qsYoEuteSVdiZxhkvaL0_w.png)  
*Source: Original Author*

***Simplification #3***

Feed Forward 는 여러층으로 연결된 linear layer + nonlinear activations 로 구성된다. 정확하게는, 한 층의 linear layer와 activation 은 Feed Forward의 단순화로 볼 수 있다. 이 단순화로 Transformer와 GCN의 다이어그램이 매우 유사해 보인다.:

![trm n gcn](https://miro.medium.com/max/698/1*SxXxP8P7PWdI5BxdpNjQWw.png)  
*Source: Original Author*

### Attention vs Spectral Convolution

이제 Dot-Product Attention 이 Spectral Convolution 로 단순화 된다면, GCNs 은 Transformers 모델들의 일부분으로 볼 수 있다. 하지만 이는 불가능하다. 왜냐하면, Spectral Convolution 연산은 미리 정의된 인접 행렬을 사용하기 때문이다. Dot-Product Attention 은 입력으로 들어오는 모든 행령들을 직접 연산한다. 일반적으로, 우리는 결졍되지 않은 real-value의 입력으로부터 미리 정해진 인접 행렬을 구할수는 없다. 하지만 여전히 이 둘이 유사해보이고, 이들을 더 분석함으로서 추가적인 인사이트를 얻을 수 있다. 먼저 Spectral Convolution을 보자.

인접 행렬(```A[:,i,j]```)의 각 원소는 노드 i,j가 연결돼있다면 0이 아닌 값을 갖는다. 이는 다음과 같은 특징으로 이어진다.
* Static - 입력에 따라 고정된다.
* Square - ```(batch_size, num_nodes, num_nodes)``` 형태의 정사각 행렬이다.
* Sparse - 오직 몇개의 원소만이 0이 아니다.
* Symmetric - 엣지가 양방향이기 때문에,  ```A[:, i, j] == A[:, j, i]``` 이다.

Dot-Product Attention 은 인접 행렬을 사용하지 않는데, 어떻게 Spectral Convolution과 비교할 수 있을까? 중간 결과 행렬 중 하나는 (빨간 표시) 인접 행렬과 비슷해 보인다.:

![adj](https://miro.medium.com/max/614/1*6tWuFALcaoP97tru_NjM4A.png)  
*Source: Original Author*

이것을 *attention matrix* 라고 하겠다. 이 행렬은 입력값 중 어디에 집중해야 할지 대략적으로 알려준다. 이 행려은 인접 행렬과 비교해 아래와 같은 성질을 띤다.
* Dynamic - 입력값을 막 곱했기 때문에 미리 정해지지 않는다.
* Square - 두 ```(batch_size, seq_length, num_features)``` 형태의 행렬을 곱해 ```(batch_size, seq_length, seq_length)``` 형태이다.
* Approximately Sparse - softmax activation은 각 행의 대부분의 값을 0에 가깝게 만든다. 하지만 0에 가까울 뿐 0은 아니다.
* Non-Symmetric - 행 단위의 Softmax 연산은 대칭이지 않다.

인접 행렬의 특징 중 약 2개의 특징을 찾았다. 그리고 softmax activation을 점단위 activation 함수로 바꿔 *attention matrix* 를 대칭으로 만들 수 있다. 그리고 activation 함수를 잘 고르면 (relu, 또는 그 변형들) approximately sparse 특징도 유지할 수 있다. 이제 dynamic 특징만 남았다.


### Interpretations

GCN 은 Transformers 의 완벽한 특이 케이스는 아니지만, 이 둘은 강한 유사성을 가진다. 이것은 우리에게 무엇을 알려줄 수 있는가? 저자가 느낀 점을 아래 적었지만, 놓친 추가적인 인사이트도 있을 거라 예상한다. 

1. Does this change our understanding of Transformers or GCNs, respectively?   
특별한 점은 없을 것이다. Transformers 의 attention head 에 대한 대체적인 설명이 될수도 있다. 하지만 이 모델들이 유도되고, 구현되고, 학습되는 원리에 대한
변화는 없다. 이건 그냥 재밌는 글이었고, 두 모델의 이해를 돕는 실험이었길 바란다.

2. Should we be surprised that GCNs have shown potential on certain NLP tasks?   
전혀 아니다. Transformers 는 모든 자연어 태스크의 SOTA 이고, 이 글은 단지 GCNs 이 Transformers와 유사함을 보일 뿐이다.

3. Should we be surprised that Transformers are taking over the computer vision space?  
그렇게 생각하지 않는다. Spectral Convolution 은 주로 image convolution의 일반화된 형태로 표현된다. (실제로는 Transformer가 GCNs의 일반화된 형태가 아닌 만큼, image conv의 일반화된 형태는 아니다. 컨셉 적으로만 유사할 뿐이다.) 만약 Transformers 가 GCNs 과 연관이 있고, GCNs이 image convolution 관 연관이 있다면, 분명 Transformers 와 Convolution Neural Networks 간의 연결고리가 있을 것이다. 먼 연결이더라도 말이다.

4. Are there any other tasks that Transformers should be applied to, given their relationship to GCNs?  
분명 있다. 나는 Transformers 가 그래프 구조의 데이터에 적용된 더 많은 경우를 보고싶다. 아마 Transformers 는 더 많은 학습 데이터를 필요로 할 것이다. 이 모델이 학습해야할 파라미터도 더 많고, 인접 행렬의 일부 정보를 놓쳤기 때문이다. 우리는 computer vision 분야에서도 비슷한 현상을 목격했다. - Transformers 는 CNNs 보다 더 많은 학습이 필요했지만, 결국 state-of-the-art의 성능을 보여줬다.
