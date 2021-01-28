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

-----------------------------------------------------

## The ML Metrics Trap

부적절한 지표에서의 작은 개선은 잘 알려진 기계학습 (ML) 오류이다. ML 지표의 장단점을 이해하는 것은 ML 종사자로서 개인적 신뢰도를 쌓는데 도움이 된다. 이는 성공이라고 미리 판단하는 것을 피하기 위해 필요하다. ML 종사자들은 프로토타입을 연구 수준에서 제품으로 만들기 위해 많은 예산을 투자한다. 최종 목표는 예측 시스템으로부터 값을 추출하는 것이다. 오프라인 평가 지표는 새 모델을 제품화 하는데 중요한 기준이 된다.   
이 글에서는 3가지 랭킹 지표를 볼 것이다. 랭킹은 근본적인 태스크이다. 이는 기계 학습의 추천 시스템과 정보 탐색 시스템에서 볼 수 있다. 일반적인 대부분의 추천 시스템들은 두가지 일을 한다. 아이템과 사용자 간의 점수를 예측하거나, 사용자에 맞는 순서 나열된 추천 리스트를 생성한다.

등수를 알 때 추천 시스템 평가에 쓸 수 있는 가장 유명한 3가지 평가 지표를 알아보자. :  
* **MRR**: Mean Reciprocal Rank  
* **MAP**: Mean Average Precision  
* **NDCG**: Normalized Discounted Cumulative Gain  

## Flat and “Rank-less” Evaluation Metrics

### Accuracy metrics

랭킹 업무를 다룰 때, 예측 정확도 (prediction auccuracy) 와 결정 지원 지표 (decision support metrics) 로는 부족하다. **예측 정확도 지표**는 mean absolute error (**MAE**), root mean square error (**RMSE**) 를 포함한다. 이들은 실제 점수와 예측 점수를 비교하는데 초점이 맞쳐있다. 이들은 각 점수를 예측하는 수준에서 수행된다. 만약 한 사용자가 어떤 아이템에 4.5점을 줬다면, 이 지표들은 이 점수와 예측 결과가 얼마나 멀리 떨어졌는지를 말해준다.

### Decision support metrics

**결정 지원** 지표는 **Precision, Recall, F1** 점수를 포함한다. 이들은 사용자가 좋은 결정을 내리도록 추천이 얼마나 도움이 됐는지에 집중한다. 이들은 사용자가 'good' 아이템을 고르고, 'bad' 아이템을 피하도록 돕는다. 이런 형태의 지표들은 추천 시스템에서 무엇이 중요한지 강조하기 시작한다. 사용자에게 100개의 아이템을 추천할 때, 가장 중요한 것은 첫 5개 또는 10, 20개에 위치하는 아이템들이다. Precision은 선택된 요소들의 비율이다. 이 지표는 가장 유용한 물건을 추천하는데 집중한다. Recall은 시스템이 선택한 관련 아이템 비율이다. 이 지표는 유용한 물건을 놓치지 않는데 집중한다. F1 점수는 두 지표의 조합이다. F1 조화 평균은 precision과 recall의 균형을 위한 하나의 지표이다.  
랭킹 업무에서는, 이 지표들은 하나의 중요한 약점을 갖는다. 이 결정 지원 지표들은 상위 N개의 추천을 대상으로 하지 않고, **전체 데이터를 커버한다.** Precision과 Recall 모두 결과 전체에 대한 지표이다. 그래서 이런 지표들을 쓸 때는 주로 상위 N개에만 적용한다. 이를 **Precision@N, Recall@N** 형태로 나타낸다. (F1@N 같은 지표는 보지 못했다.) 수정된 Precision@N 지표는 "상위 N개에서" 좋은 아이템의 비율이다. 이는 상위 추천 아이템들에만 집중 했으니, 상위 N개 평가를 어느 정도 포함한다. 하지만 이 값들은 여전히 기존 precision, recall, F1 평가와 유사하다. *이 지표들은 모두 **찾기**를 잘하는 것만 고려한다. 우리는 **찾기 그리고 랭킹 매기기**를 잘 하는데 특화된 지표가 필요하다. 이제 랭킹-인식 평가 지표가 어떻게 도움이 되는지 봐보자.*

## Rank-Aware Evaluation Metrics

추천 시스템은 매우 특징적이고 중요한 고려사항이 있다. 연관된 아이템들을 추천 리스트의 상위에 둘 수 있어야 한다. 아마도 사용자들은 가장은 가자 좋아하는 차 브랜드를 찾기 위해 200개의 아이템을 훑지 않을 것이다. 랭킹-인식 지표는 다음 두가지 목표를 갖는다.:  
1) 추천 아이템을 **어느 위치**에 놓을 것인가?
2) 연관된 **선호도 측면**에서 얼마나 좋은 모델인가?

다음과 같은 지표들이 여기에 도움이 될 수 있다.:  
> MRR: Mean Reciprocal Rank  
> MAP: Mean Average Precision  
> NDCG: Normalized Discounted Cumulative Gain  

위 세 지표들은 두 지표 그룹에서 파생했다. 하나는 binary relevance 기반의 지표로 구성된다. 이 지표들은 아이템이 좋은지 나쁜지를 바이너리로 알기 위해 사용된다. 두번째는 utility 기반의 지표들로 구성된다. 이들은 좋고 나쁨을 절대적 또는 상대적인 수치로 표현한다. 