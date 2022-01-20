# Codestates Project 2  
## 기업 협업 프로젝트 - 웰로
 
### 프로젝트 주제  
<big>Context 태깅 모델 & 개인-정책 추천 모델 구축</big>

### 프로젝트 배경  
대한민국에는 수많은 정책이 있음에도, 정책 대상자들은 해당 정책의 존재를 모른다.
기관 입장에서는 정책 대상자를 선별하여 홍보하는 과정이 번거롭다.

### 프로젝트 목적  
정책 공고문 context에서 정책 특성을 자동으로 추출하고,  
이를 토대로 전국의 정책/지원 사업을 유저의 프로필 특성에 맞게 추천해준다.

### 프로젝트 세부사항

#### 데이터 셋
유저 raw데이터: 20,010개 샘플
정책 raw데이터: 88,468개 샘플


#### 모델링 개요  
#### 태깅 모델  
  - 태깅 모델 A:  
    - 유저, 정책데이터에 대한 1차 태그 추출 모델  
    - 태그 관련 필수 키워드 빈도수에 따른   
    
  - 태깅 모델 B:  
    - 모델 A 이후로 진행하는 advanced 모델  
    - 각 키워드를 이용해 개체명 인식기 직접 학습진행  
    - 개체명 인식기를 이용해 문맥 내 다종류의 키워드를 1개의 통일된 태그워드로 치환  
    - 모델 A에서 나온 결과를 라벨로 두고 개체명 인식기를 통과한 문맥데이터를 인풋값으로 BERT모델 학습
  
#### 추천 모델  
  - 추천 모델 A:  
    - 유사도 기반 모델  
    - 베이스라인으로 사용되며 넓은 범위/좁은 범위로 나타냄  
    - 유저*feature(one-hot), feature(one-hot)*정책 matrix를 행렬곱을 취해줌  
    - 만들어진 유저*정책 matrix의 score를 파악해 랭킹이 높은 정책을 추천
    
  - 추천 모델 B:
    - 모델 A를 통해 추려진 정책을 각 유저에 맞는지 여부(적합성)를 라벨로 두고 학습진행  
      - 적합성의 여부1: 유저의 관심정책에 해당하는 정책인가
      - 적합성의 여부2: 유저의 조건에 매칭이 되는 정책인가
    - 딥러닝을 이용하는 모델, 후보군은 Wide & Deep, DeepFM, NCF(완전한 memorization은 불가할 것으로 예상)  
    - 추천 모델 A와 순서를 바꾸어 진행할 수 있음.   
    ㄴ> 먼저 wide & deep으로 추천 리스트를 뽑고 이후 더 높은 score의 정책을 추천  
    
  - 샘플 축소 모델: 샘플 수가 너무 많아 학습 진행에 어려움이 있었고, 이를 해결하기 위한 방법으로 아래 두 가지를 모델 A 이후 진행.  
    - 대표 샘플 추출:  
      - 유저, 정책데이터의 각 feature를 one-hot인코딩 후 score를 합산  
      - score를 정렬하여 기준 score를 군집으로 평가해 군집 내에서 대표 샘플을 추출하는 방식  
    - 클러스터링 추출:  
      - k-prototypes 모듈을 통해 numerical, categorical features에 대한 clustering 진행  
      - 선정한 군집 내에서 n개의 샘플 추출하는 방식  

<br>

---

### 회고(프로젝트 후기):  

#### 딥러닝 모델 환경 구축의 문제
프로젝트 데이터에 맞는 딥러닝 모델을 선정하기 위해 다양한 모델을 공부했으나, 이론적으로 모델에 대해 아는 것과 실제 데이터로 모델을 다루는 것은 많이 다르다는 것을 깨달음. 
모델의 Input 데이터에 맞춰 우리의 데이터를 엔지니어링하는 부분에서 실패와 보완을 반복함.

#### 개인
추천과 자연어처리에 관심을 두고 다양한 구현을 해봤음에도 실무 데이터를 이용해 실제 활용하는 것이 어렵다는 것을 느낄 수 있었다. 
데이터 활용과 모델링의 깊이있는 공부 및 사용 경험이 적어 시간이 부족했고, 그로인해 원했던 딥러닝 모델을 적용하지 못한 것이 아쉽다. 
모델들의 발전을 좀더 깊이있게 공부하면서 이론상에서 놓쳤던 부분을 재숙지하고, 이 모델을 서비스에서 어떻게 응용할 수 있을지 더 많이 고민해야겠다.



