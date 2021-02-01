---
layout: post
category: develop
comments_id: 2
---
[link](https://medium.com/swlh/rank-aware-recsys-evaluation-metrics-5191bba16832)  
Medium Blog *The Startup*  
[Moussa Taifi, Ph.D.](https://farmi.medium.com)     
Nov 25, 2019

이 글은 비공식 번역문 입니다.

*[previous post](https://yongqyu.github.io/mldl-mrr-vs-map-vs-ndcg-rank-aware-evaluation-metrics-and-when-to-use-them-1.html)*

-----------------------------------------------------

## MRR: Mean Reciprocal Rank

이 지표는 세 지표 중 가장 단순하다. 이 지표는 가장 유사한 아이템이 어디에 위치하는지를 측정한다. 이는 binary relevance 그룹과 매우 연관돼있다. 알고리즘은 아래와 같다.:

![mrr](https://miro.medium.com/max/700/1*3vI82IYrTiN7fX0ht6mscw.png)

세명의 사용자에게 각각 세개의 추천 리스트가 주워졌다고 보자. 우리는 각 리스트에서 가장 연관성 높은 아이템의 순위를 통해 각 유저의 reciprocal rank를 계산할 수 있다. 그런 후 단순히 모든 유저에 대해 평균을 내면 된다.

![mrr_ex](https://miro.medium.com/max/643/1*dR24Drmb9J5BLZp8ffjOGA.png)

### MRR Pros
* 계산이 단순하고 이해하기 쉽다.
* 이 지표는 가장 연관성 높은 아이템에 매우 집중한다. 그래서 "초고의 아이템" 과 같은 질의에 적합하다.
* 탐색형 질의나 팩트 탐색 같은 정해진 항목 검색에 적합하다.

### MRR Cons
* 추천 리스트의 나머지에 대해서는 평가하지 않는다. 오직 하나의 아이템만 평가한다.
* 이는 집중하고 있는 하나의 연관 아이템이 들어있는 리스트를 만들어낸다. 평가의 목적이 그것이라면 괜찮다.
* 탐색을 위해 연관 아이템 리스트를 보고 싶은 사용자를 위한 지표로는 적절하지 않다. 이런 유저들은 다양한 아이템들을 비교하고 싶어하기 때문이다.

## MAP: Average Precision and Mean Average Precision

만약 binary relevance 데이터가 있다고 해보자. 그리고 특정 N 보다 상위 등수의 추천 리스트 전체를 평가하려고 한다. 이런 구분은 Precision@N 지표를 통해 통합됐었다. P@N decision support 지표는 n개의 추천 아이템이 좋은지를 계산했다. 단점으로는 추천 리스트를 순서 있는 리스트로 취급하지 않는 다는 것이었다. P@N은 전체 리스트를 하나의 집합으로 보고, 모든 에러를 동등하게 취급한다. 우리의 목표는 리스트의 초반에 있는 아이템의 에러와 후반에 있는 아이템의 에러를 구분하는 것이다. 이를 위해, 에러에 따라 다르게 가중치를 주는 평가 지표가 필요하다. 그러면 리스트의 후반으로 갈수록 에러도 감소할 것이다.  
Average Prediction (AP) 는 이런 가중치 이동 변화를 근사하는 평가 지표이다. 이는 이어지는 부분 리스트에 대한 precision의 조합과 이 부분 리스트의 recall 변화량을 합쳐서 사용한다. 계산식은 다음고 같다.:


![map](https://miro.medium.com/max/700/1*wh9Hmb1H1tKxUs-pMXBlvw.png)
![map_ex](https://miro.medium.com/max/700/1*0xdZ-NWJLlf3m4oyjh0K5g.png)

위의 그림에서, AP 지표는 하나의 추천 리스트에 존재한다. 연관된 아이템이 나올때마다 그 부분 리스트에서 새로 precision을 계산한다. 이를 추천 리스트 끝까지 반복한다. 이렇게 얻어진 precision 집합을 평균냄으로써 각 사용자에 대한 average precision을 구한다. 모든 유저에 대해 AP를 구하고 나면 mean average precision을 구할 수 있다.  
이는 AP 지표의 원래 목표를 근사하는 것이다. AP 지표는 precision-recall 곡선 아래의 영역을 의미한다. precision-recall 곡선은 recall 값에 따른 precision을 구함으로 얻을 수 있다. 전반적인 과정은 모든 유저에게 추천된 리스트들에 대해 PR 곡선을 생성한 후, 보간된 PR 곡선을 생성하고, 최종적으로 보간된 PR 곡선을 평균낸다 ([참고](https://www.youtube.com/watch?v=yjCMEjoc_ZI)). 시각적으로는 다음과 같다.:

![ap_curve](https://miro.medium.com/max/700/1*aOhHhjThnTpC0pXZbu5fGw.png)

시스템간의 비교를 위해 PR 곡선 아래 영역을 보면 된다. 위 예를 보면, 시스템 A가 시스템 C보다 모든 recall 영역에 대해 우수함을 알 수 있다. 하지만 시스템 A와 B는 교차하는데, 이런 경우는 어느 시스템이 전반적으로 우수한지 판단하기 어렵다. 그래프는 하나의 지표보다 이해하기가 어렵다. 연구자들이 AP를 근사하는 하나의 평가 지표를 만든 이유이다. 이 과정을 설명하는 [위키피디아](https://en.wikipedia.org/wiki/Evaluation_measures_%28information_retrieval%29#Average_precision)에 추가 설명을 첨부한다.:  
![ap_approx](https://miro.medium.com/max/700/1*Qr55R4r0XoQWwNknfIFkyg.png)

마지막 포인트는 우리가 평균 내는 것이 무엇인지 아는 것이다. 아래 그래프는 다양한 사용자들 사이에 존재하는 노이즈이다. 이런 생각은 MAP 점수를 이해하는데 도움이 된다. 아래 그래프의 밝은 빨간선은 평균 PR 곡선이다. 다른 곡선들은 N명의 각 유저들이다. 실제 효과는 질의에 따라 크게 다를 수 있다. MAP의 평균 연산은 분명히 실제 결과 성능에 영향을 미친다. 이런 샘플 곡선들이 MAP 지표의 퀄리티를 평가하는데 도움이 될 것이다.

![avg_map](https://miro.medium.com/max/700/1*dbIUWWJGK6VT4NgR1AEI4w.png)

([참고1](https://acutecaretesting.org/en/articles/precision-recall-curves-what-are-they-and-how-are-they-used), [참고2](https://medium.com/@jonathan_hui/map-mean-average-precision-for-object-detection-45c121a31173))

### MAP Pros
* 복잡한 Precision-Recall 곡선 아래 영역에 대한 하나의 지표를 제공한다. 이는 각 리스트에 대해 average precision을 제공한다.
* 자연스럽게 추천 **리스트의 랭킹**을 다룬다. 이는 탐색한 아이템들을 하나의 집합으로 보는 것과는 대조적이다.
* 이 지표는 추천 리스트의 상위에서 발생한 에러에 더 큰 가중치를 줄 수 있다. 이는 가능한 추천 리스트의 상위권에 많은 연관 아이템을 위치시키고 싶은 요구와 맞는다.

### MAP Cons
* 이 지표는 binary 평점 데이터에 적절하다. 하지만, numerical 평점에는 그렇지 않다. MAP는 이런 데이터에서는 에러를 계산할 수 없다.
* 1점부터 5점까지 같은 평점 데이터에서는, binary화 하기 위해 기준점을 정하는 것이 먼저다. 예를 들어 4점 이상의 평점만 고려하는 것이다. 이는 인위적인 기준점으로 편향이 생길 수 있다. 그리고 잘 분포된 점수 정보를 버리게 된다. 이런 정보에는 4점과 5점의 차이 같은 것이 있다. 

이런 문제를 해결하기 위해 추천 시스템 커뮤니티에는 다른 더 최신의 지표가 있다. 이 지표는 평점과 같은 잘 분포된 정보를 고려한다. Normalized Discounted Cumulative Gain (NDCG) 에 대해 알아보자.
