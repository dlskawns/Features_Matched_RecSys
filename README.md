# Codestates Project 2  
## 기업 협업 프로젝트 - Context 태깅 모델 & 개인-정책 추천 모델 구축(With 웰로)
 
### 프로젝트 주제 및 목적
<big>Context 태깅 모델 & 개인-정책 추천 모델 구축</big>

대한민국에는 수많은 정책이 있음에도, 정책 대상자들은 해당 정책의 존재를 모르는 경우가 많아 개인 유저들에게 조건이 부합하는 정책을 추천하고자 한다.  
가입한 유저들의 프로필을 바탕으로 태그를 추출하고, 정책 공고문 context에서 정책 태그를 자동으로 추출하여 이를 토대로 전국의 정책/지원 사업을 유저의 프로필 특성에 맞게 추천해준다.

### 프로젝트 세부사항

#### 데이터 셋
유저 raw데이터: 20,010개 샘플
정책 raw데이터: 88,468개 샘플

#### 태깅 모델  
  - 태깅 모델 A:  
    - 유저, 정책데이터에 대한 1차 태그 추출 모델  
    - 태그 관련 필수 키워드의 유무에 따른 추출 진행
```
# 성별 관련 태그(남성, 여성 무관) 예시
남성 태그 추출
context: ...의료급여수급자 중 50~70세 남자\n- 전립선 검진을 필요로 하는..
keywords: '전립선', 남성', '남자', '의경','장병' 등

여성 태그 추출
context:  ...경제활동을 한 적이 없는 여성 중에서 취업을 희망 하는...
keywords: '여성', 여자', '임산부', '출산', '생리', '미혼모' 등

무관 태그: 두 가지가 모두 추출되거나 없는 경우 무관 태그 생성
```

  - 태깅 모델 B:  
    - 모델 A 이후로 진행하는 advanced 모델  
    - 각 키워드를 이용해 개체명 인식기 직접 학습진행  
    - 개체명 인식기를 이용해 문맥 내 다종류의 키워드를 1개의 통일된 태그워드로 치환  
    - 모델 A에서 나온 결과를 라벨로 두고 개체명 인식기를 통과한 문맥데이터를 인풋값으로 BERT모델 학습


개체명 인식기 예시 - 신청절차 태그(무관: A10:00, 온라인: A10-01, 오프라인: A10-02)  
* 학습을 위한 데이터 전처리  
![image](https://user-images.githubusercontent.com/84971151/154800420-41704cbe-86fd-4aba-8b3a-f64b10d38a74.png)  
* 1차 개체명 인식 학습 결과 -> 최종 모델링 실패  
![image](https://user-images.githubusercontent.com/84971151/154800427-9997449c-fa66-40c2-967c-03dff6e8994c.png)  
* 2차 개체명 인식 학습 결과   
![image](https://user-images.githubusercontent.com/84971151/154800550-9fbdfdf3-0804-4cb4-9436-4bf0de819085.png)  


  
#### 추천 모델  
  - 추천 모델 A:  
    - 유저태그 및 정책태그의 ONE-HOT VECTOR로의 변형 후 만들어진 Matrix 간의 행렬곱을 통해 Score를 추출
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



