---
layout: post
category: develop
comments_id: 2
---
# [ : Using Graph Learning to Power Recommendations](https://eng.uber.com/uber-eats-graph-learning/)  
Uber Engineering Blog  
Ankit Jain, Isaac Liu, Ankur Sarda, and Piero Molino  
December 4, 2019

이 글은 비공식 번역문 입니다.

*[previous post](https://yongqyu.github.io/gnn-food-discovery-with-uber-eats-1.html)*

-----------------------------------------------------

### Graph learning for dish and restaurant recommendation at Uber

우버 이츠 앱에는 아래와 같이 몇가지 추천 화면이 있다.

![fig3-2](https://1fykyq3mdn5r21tpna3wkdyi-wpengine.netdna-ssl.com/wp-content/uploads/2019/12/image6-1.gif)
![fig3-2](https://1fykyq3mdn5r21tpna3wkdyi-wpengine.netdna-ssl.com/wp-content/uploads/2019/12/image13-3.gif)
![fig3-3](https://1fykyq3mdn5r21tpna3wkdyi-wpengine.netdna-ssl.com/wp-content/uploads/2019/12/image15.gif)  
*Figure 3: 우버 이츠 UI는 사용자의 과거 주문과 취향을 통해 다양한 옵션을 제공한다.*

메인 피드에서 사용자 취향 기반으로 식당과 메뉴의 추천 캐러셀을 보여준다. 식당의 메뉴를 탐색할때도, 식당 내 메뉴를 사용자의 취향에 맞게 추천한다. 이런 제안들은 과거 데이터를 통해 학습된 추천 모델에 의해 생성된다. 우버 이츠의 추천 시스템은 두가지로 나눌 수 있다.: 후보군 생성과 개인화 랭킹.

후보군 생성에서는 적절한 음식과 식당을 연산 가능한 방법으로 추린다. 엄청나게 많은 수의 음식과 식당들을 미리 한번 거르기 위해서는 매우 뛰어난 확정성이 필요하다. 사전 필터링은 물리적 위치 같은 특징을 기반으로 해서, 배달 범위 밖 식당은 추천에서 제외한다. 음식이나 식당의 후보들은 또한 주어진 사용자와 맞아야 한다.

그 다음으로, 개인화 랭킹은 날짜, 시간, 사용자의 현재 위치 등 추가 정보를 사용해 사전 필터 된 후보군에서 순위를 매기는 ML 기반 모델이다. 반복되는 주문을 예로 들면, 모델은 특정한 요일의 음식 종류나 점심,저녁 별 음식 차이 등의 패턴을 찾아낸다.

우버 이츠의 추천 시스템을 개선하기 위해 GNN을 활용하려면, 2개의 bi-partite 그래프를 그릴 수 있다. 하나는 사용자와 음식을 노드로 하고, 평점을 엣지로 하는 그래프. 다른 하나는 사용자와 식당을 노드로 하고, 방문 횟수를 엣지로 하는 그래프이다. 우리는 projection 후 통합 함수(aggregation function)를 max pooling이나 mean pooling으로 하는 특징을 가지는 GraphSAGE[^4] 를 선택했다. 그 이유는, 우리는 강력한 확정성을 중심으로 고려했기 때문이다. 이 GNN 모델에서, 노드 정보와 이웃 정보의 합성은 concatenation을 통해 이루어진다. 게다가, GraphSAGE는 중심 노드에서 1-, 2-hop 거리의 노드들을 정해진 숫자로 샘플링 하는 전략을 적용했는데, 이는 수십억의 노드 그래프가 학습 가능한 스케일로 만들고, 심지어 성능도 좋다.

GraphSAGE를 우리 그래프에 적용하기 위해, 우리는 몇가지 수정을 했다. 먼저, 다른 종류의 노드는 다른 특징들을 가지므로, 추가적인 영사층(projection layer)을 더했다. 이 층은 노드 종류에 상관 없이 같은 사이즈의 벡터가 되도록 만든다. 예를 들어, 음식은 설명글이나 연관 이미지로 표현되고, 식당은 메뉴로 표현되는데 이 피쳐의 크기들이 다 다르므로 영사층이 이를 같은 크기로 만든다. 게다가 graphSAGE는 binary 엣지를 가지는데, 우리 경우는 방문 횟수나, 평점 등 가중치가 있는 엣지가 필요하다. 이를 해결하기 위해, 우리는 엣지에 가중치를 주는 개념을 추가했다. 그 중 가장 영향력 있는 변화는, 아이템 랭킹에 더 적합한 hinge loss를 적용한 것이다. (역주 : 여기서 hinge loss는 triplet loss를 의미하는 것 같다.)

사용자 u가 음식 v를 최소한 한번 주문하면, 가중치 엣지가 둘 사이에 존재하게 된다. 만약 우리가 한 쌍의 노드 사이의 점수를 예측하고 싶다면, 이는 임의로 선택된 음식 n 보다는 높은 점수일 것이다. 그 차이가 마진값보다 높아야 한다. 이 loss의 문제는 높은 가중치의 엣지와 낮은 가중치의 엣지가 바뀔 수 있다는 것이다. 한번 주문한 음식과 열번 주문한 음식으로는 잘 작동하지 않을 수 있다. 이런 이유로, 우리는 낮은 수준의 긍정(low-rank positive)이라는 개념을 제안한다.


![fig4](https://1fykyq3mdn5r21tpna3wkdyi-wpengine.netdna-ssl.com/wp-content/uploads/2019/12/image11-1.png)
*Figure 4: 우버 이츠 추천 시스템의 low rank positive로 향상된 max-margin loss*

그림 4는 어떻게 낮은 수준의 긍정을 활용해 loss를 개선했는지 보여준다. 긍정의 엣지 <u, v>, 낮은 수준의 긍정 <u, l> (여기서 l은 v보다 점수가 낮은 노드)가 주어진다.
우리는 높은 가중치의 엣지가 부정 마진보다는 작은 차이로 높은 순위가 되도록 두번째 항을 추가했다. 로스의 두 항 모두 알파 곱 연산이 있는데, 이는 부정 샘플 부분과 낮은 수준의 긍정 부분의 중요도를 조절하는 하이퍼 파라미터이다. 마지막의 통합 함수와 샘플링에는 weights를 사용한다.

GNN을 학습시켜 노드의 표현을 한번 얻으면, 우리는 노드간의 거리를 그들간의 유사도의 근사값으로 사용할 수 있다. 특히 사용자와 음식들 간의 내적과 코사인 유사도를 추천시스템의 입력값 중 하나로 추가했고, 이를 온라인과 오프라인에서 테스트했다.

임베딩 벡터들이 우리 추천에 얼마나 유용한지 평가하기 위해, 우리는 특정 분할 시점 이전 4달치 데이터로 모델을 학습했다. 그리고 그 기준일 이후 열흘의 음식, 식당 주문 데이터에서 평가했다. 특히, 우리는 유저와 모든 음식들, 그리고 도시 내 식당들간의 코사인 유사도를 계산해서 순위를 매겼다. 실험에서 우리는 기존 모델 대비 MRR, Precision@K, NDCG와 같은 지표에서 최대 20% 성능 향상을 관측할 수 있었다.

그래프를 활용한 학습의 성능 향상을 통해 우버 이츠의 추천 시스템 내 랭킹 모델에도 적용해도 된다는 확신을 얻었다. 그래프를 통해 학습한 개인화 랭킹 모델은 기존 모델 대비 AUC 12% 향상된 성능을 보였다.

게다가 이 특징의 영향력을 분석해보니, 추천 모델 내에서 가장 영향력 있는 특징이었다. 이는 아래 그림 5와 같이, 그래프 학습 모델이 기존 어느 모델보다 더 많은 정보를 추출한다는 확신을 주었다.:

![fig5](https://1fykyq3mdn5r21tpna3wkdyi-wpengine.netdna-ssl.com/wp-content/uploads/2019/12/image18.png)
*Figure 4: 새로운 그래프 학습 특징이 우버 이츠의 음식, 식당 추천 시스템에서 퀄리티와 적절성을 결정하는데 가장 가치 있는 특징임을 증명했다.*

이런 오프라인 결과를 통해, 우리는 새 모델이 온라인 실험에 출시돼도 괜찮겠다 싶었다. 우리는 샌프란시스코에서 A/B 테스트를 구성했고, 실질적인 참여율 및 클릭률 향상을 관측할 수 있었다. 또한, 이 모델이 예측한 음식들이 Uber Eats 사용자들에게 더 많은 관심을 끌었음을 입증했다.

---
{: data-content="footnotes"}

[^4]: William L. Hamilton, Rex Ying and Jure Leskovec: Inductive Representation Learning on Large Graphs. NIPS 2017
