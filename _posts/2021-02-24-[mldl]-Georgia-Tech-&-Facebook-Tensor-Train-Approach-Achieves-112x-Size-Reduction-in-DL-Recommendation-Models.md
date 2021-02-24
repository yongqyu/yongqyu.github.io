---
layout: post
category: develop
comments_id: 2
---
[link](https://syncedreview.com/2021/02/02/georgia-tech-facebook-tensor-train-approach-achieves-112x-size-reduction-in-dl-recommendation-models/)  
Synced     
Feb 2, 2021

이 글은 비공식 번역문 입니다.

-----------------------------------------------------

# A new study by the Georgia Institute of Technology and Facebook AI introduces TT-Rec, a way to drastically compress the size of memory-intensive Deep Learning Recommendation Models (DLRM) and make them easier to deploy at scale.

Georgia Institute of Technology와 Facebook AI가 새로운 연구에서 메모리 집중적인 딥러닝 추천 모델 (DLRM)의 크기를 극단적으로 압축되도록 디자인되고 적절한 크기로 쉽게 사용할 수 있는 TT-Rec (Tensor-Train for DLRM)을 소개했다. 제안된 방법의 핵심 혁신요소는 DLRM의 거대한 임베딩 테이블을 텐서 학습 분해를 이용한 매트릭스 곱의 연속으로 대체한 것이다.

제품 환경에서 추천 시스템을 구축하는 신경망 기반의 개인화 및 추천 모델은 넷플릭스나 유투브같은 컨텐츠 플렛폼이나 페이스북 같은 거대 테크 회사들의 필수적인 도구가 되었다. 이런 구조화된 DLRM들은 크게 두가지 요소로 가진다.: MultiLayer Perceptron (MLP) 계층 모듈과 임베딩 테이블 (Embs) 이다. MLP 계층은 사용자의 나이와 같은 연속적인 값을 다룬다면, EMBs는 밀도가 낮은 고차원의 입력값을 밀도가 높은 벡터 표현으로 인코딩하는 방식으로 카테고리 피쳐를 다룬다.

![tt-rec](https://i2.wp.com/syncedreview.com/wp-content/uploads/2021/02/Facebook-AI-DLRM.gif?resize=720%2C389&ssl=1)

논문의 TT-Rec: 딥러닝 추천 모델을 위한 텐서 학습 압축은 산업에 있는 DLRM의 임베딩 테이블 메모리 크기가 기가바이트에서 테라파이트로 증가하는 사실에 집중한다. 예를 들어, 페이스북 데이터 센터의 자원을 많이 먹는 DLRM은 전체 추천 모델 용량의 99% 이상을 구성하는 거대한 임베딩 테이블로 인해, 학습 시간의 50% 그리고 AI 추론 사이클의 80% 이상을 소비한다. 임베딩 테이블에는 수천만개의 행이 존재하고, 그 숫자는 새로운 추천 모델을 거듭할수록 지수적으로 상승해 TB 스케일의 메모리를 필요로 한다.

점점 더 큰 DLRM의 적용이 늘어나면서, 인프라구조의 요구 용량이 지수적으로 증가하지 않는 빠르고 효과적인 DLRM의 개발이 매우 필요해졌다.

![fig1](https://i0.wp.com/syncedreview.com/wp-content/uploads/2021/02/image-11.png?strip=info&w=688&ssl=1)
![fig2](https://i0.wp.com/syncedreview.com/wp-content/uploads/2021/02/image-10.png?strip=info&w=670&ssl=1)

이런 목표를 이루기 위해, 연구진들은 DLRM의 거대한 임베딩 테이블을 대체할 매트릭스 곱의 연속인 TT-Rec의 압축 기술을 디자인했다. 텐서-학습 분할은 텐서를 작은 다차원 데이터로 나눔으로써 매트릭스 분할을 할 수 있다. 연구진들은 그들이 제안한 방법을 메모리 저장공간과 대역폭의 트레이드 오프를 위한 룩업 테이블로 비유한다. 그 결과, 임베딩 벡터들은 3차원의 텐서(TT-core 곱)로 대체된다. 연구진들은 또한 DLRM에서의 특징적인 밀도 낮은 피처를 탐색하는 TT-Rec의 캐시 구조도 소개했다. 이는 실험적으로도 성능 향상에 기여했다.

그들의 접근법을 평가하기 위해, 연구진들은 Criteo의 Kaggle과 Terabyte 데이터셋으로 MLPerf-DLRM을 학습시켰다. 이 실험들에서 제안된 TT-Rec은 베이스라인 만큼의 모델 성능을 유지하면서 학습 시간을 단지 13.9% 증가시키고 전체 모델 메모리 용량 요구사항을 112배까지 줄였다.

![fig5](https://i2.wp.com/syncedreview.com/wp-content/uploads/2021/02/image-12.png?w=837&ssl=1)

연구진들은 TT-Rec의 메모리 용량 감소가 학습 시간의 적은 상승에 비해 상당하며, 이 기술이 실시간 추천 모델 학습을 더 효과적이고 실용적으로 만들것이라고 믿는다.
