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

*[previous post 1](https://yongqyu.github.io/mldl-mrr-vs-map-vs-ndcg-rank-aware-evaluation-metrics-and-when-to-use-them-1.html)*  
*[previous post 2](https://yongqyu.github.io/mldl-mrr-vs-map-vs-ndcg-rank-aware-evaluation-metrics-and-when-to-use-them-2.html)*

-----------------------------------------------------

## Normalized Discounted Cumulative Gain

MAP 지표의 목표는 NDCG의 것과 비슷하다. 둘 다 더 높은 유사도의 아이템을 리스트의 상위에 놓고싶어한다. 하지만 NDCG는 추천 리스트 평가에 더 맞춰져있다. 이 지표는 어떤 요소가 다른 요소보다 "더" 유사하다는 정보를 사용한다. 매우 유사한 아이템은 적당히 유사한 아이템보다 앞에 오고, 적당히 유사한 아이템은 유사하지 않은 아이템보다 앞에 오게 된다. 다음은 NDCG를 순서대로 계산하는 도표이다.:

![ndcg](https://miro.medium.com/max/700/1*_a_YsiSw6K1whOzTYYq_Ag.png)

우선 Cumulative Gain (CG) 를 구한다. 이는 유사도의 정도를 측적하는데 쓰이는 지표이다. 이 지표는 리스트의 랭크나 위치를 고려하지는 않는다. 랭킹 태스크에서는 리스트에서의 위치에 따른 영향력이 있어야 한다. 기본적인 Discounted Cumulative Gain (DCG) 는 로그 스케일로 위치에 대한 패널티가 추가된다. 게다가 실제 서비스에서는, 연관된 아이템을 탐색에 집중하기 위해 연관 점수를 강화하는 것이 일반적이다. 이런 형태가 industrical DCG 식이다.

사용자는 그때그때 다른 수의 아이템을 추천 받는다. 이는 DCG 측정 방법이 유저별로 공평하지 않게 만든다. 그래서 이 점수를 0과 1 사이로 정규화 할 필요가 있다. 이를 위해 사용자에게 제공할 수 있는 이상적인 랭킹을 정한다. 이를 Ideal Discounted Cumulative Gaine (IDCG) 라고 한다. 이는 깔끔한 정규화 상수로, 우리는 이를 통해 NDCG를 구할 수 있다. NDCG는 사용자 별 지표로, 테스트셋의 모든 유저에 대해 계산해야 한다. 그리고 모든 유저들에 대해 평균을 내 하나의 값을 구한다. 이 평균값이 추천 시스템을 피교하는데 사용된다. 이 과정을 아래 그림에 시각화했다.

![ndcg_ex](https://miro.medium.com/max/700/1*W6cQB2kozFxedqVu9lpSVw.png)

### NDCG Pros
* NDCG의 가장 큰 장점은 유사도 값의 정도를 고려한다는 것이다. 만약 데이터셋이 이 정보를 제공한다면, NDCG가 적절하다.
* MAP와 비교해서 순위 있는 아이템들의 위치를 평가하기에 좋다. binary relevant/non-relevant 시나리오 모두에서 작동한다.
* 부드러운 로그 스케일의 감소 요소는 이론적으로도 훌륭한 근거가 있다. 이 연구의 저자들은 두개의 다른 추천 모델 쌍들에 대해서 NDCG 지표를 통해 일관적으로 더 나은 모델을 찾을 수 있다는 것을 보여줬다.
  
### NDCG Cons
* 부분적 피드백에 대한 이슈가 있다. 이 문제는 평가 데이터가 완전하지 못할때 발생한다. 이는 추천 시스템의 대부분의 경우에 해당한다. 만약 평가 데이터가 다 있다면 모델이 할 일이 없을테니 말이다. 이 경우 시스템 관리자는 없는 평가 데이터를 어떻게 처리할지 결정해야 한다. 연관이 없는 것 처럼 0으로 처리하거나, 사용자의 평균,중간값으로 처리하는 식이다.
* IDCG가 0인 경우에 사용자를 직접 처리해야한다. 이는 사용자의 연관 아이템이 없는 경우인데, 해결법으로는 NDCG를 0으로 처리하는 것이다. 
* 다른 문제는 NDCG@K 처리이다. 추천 시스템으로 부터 반환받는 추천 리스트의 길이가 k보다 작을 수 있다. 이 문제는 고정된 길이의 결과 리스트에 최소한의 점수가 되도록 나머지 부분을 채우는 것이다.
  

