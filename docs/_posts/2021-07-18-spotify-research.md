* 원문 링크
    - [contextual and sequential](https://research.atspotify.com/contextual-and-sequential-user-embeddings-for-music-recommendation/)
    - [automated online experiments](https://research.atspotify.com/model-selection-for-production-system-via-automated-online-experiments/)
    - [Long-Tail Content](https://research.atspotify.com/query-understanding-for-surfacing-long-tail-music-content/)
    - [Evolving User Preferences](https://research.atspotify.com/finding-structure-in-users-evolving-listening-preferences/)



### 1. 관심 요소 및 모델
* 각 주제에 포함


### 2. 비즈 인사이트

# Context and Sequence (사용자 모델링)

* Problem formulation
    - session: 짧게 이어지는 track consumption, 한 곡이 끝나고 20 분이 지나면, 다른 세션으로 간주
        - 콘텐츠가 짧아서, 하나의 session 에서 여러 콘텐츠들이 함께 소비됨
    - context: 시간(아침,저녁 등), 단말(모바일, PC, 스피커, 태블릿 등)
        - 음악 추천에서는 context 가 중요함 (아이들나라와 비슷할지도?)
    - 문제: session 시작점에서 그 session 동안 들을 음악 예측
        - 단, 사용자의 global taste 와 context 도 알 수 있음을 가정       
    - 모델: track 을 word2vec bag-of-word model 학습 (latent semantic space)
        - session 을 track embedding 의 average 로 봄
* Contextual Preferences and Skip Rate
* Contextual and sequential model with recurrent network (CoSeRNN)


# Long-Tail Content (검색 모델링)
* Approach
    - 검색이나 추천에서 잘 노출되지 않는 콘텐츠를 long-tail content 라 함
        * long-tail content 여부는 domain expertise 로 판별
    - 이런 콘텐츠를 잘 추천해주면, 콘텐츠도 사용자도 윈윈
        - *어떤 영화를 보겠다는 목적성이 뚜렷하지 않은 고객을 붙잡는 효과가 있을지도!*
        - 다만, 지금 처럼 지면 상하 순서를 정렬하지 않고, 고정해야 할것 같음
        - 그래야 고객이 어디에서 어떤 콘텐츠를 확인할 수 있는지 알 수 있음
    - Casual Music 과 Niche Genres 가 효과적인 long-tail content 이 될 수 있을것 같음
* Focused and non-Focused queries
    - Focused queries: 특정 정보와 연관되는 질의(예. Katy Perry fireworks)
    - Non-Focused queries: 범위가 넓고 open ended information 질의(예. relaxing music)
        * 이러한 queries 에 long-tail content 를 노출할만할듯!
* Query features
    - 검색 질의들을 잘 분석하면 Non-Focused queries 를 분류할 수 있을듯함
    - Standalone features: 사용자에게 제공된, 사용자가 사용한 long-tail content 개수 같은 표면적인 것
    - Reference dependent features: 검색 결과에 long-tail content 가 포함되었던 질의와 비교
        - 질의들의 average embedding distance 가 reference 와의 차이를 가장 명확하게 구분했음
    - Interaction features: 사용자가 검색 결과를 어떻게 활용하는지 파악하여, 질의가 얼마나 일반적인지 정량적인 지표(예. 엔트로피)화 


# Evolving User Preferences (사용자 모델링)

*Recent findings suggest that users’ preferences change over time, and that diversifying the users’ consumption might be important for long-term satisfaction.*

* Evolving Preferences of Users
    - 2016~2020 의 사용 이력으로 total variation distance 측정
    - 17 분기 4 천장르로 요약
    - 시간이 지날수록 total variation 평균적으로 늘어남을 확인
        - 사용자의 선호가 시간에 따라 변화함을 의미
* Discovery Correlates with Conversion
    - 다양한 음악을 듣는 사용자일 수록 유료 서비스 전환율이 높았음
        - 스트리밍 서비스라서 그런듯...
* Preference Transition Model(PTM)
    - PTM 의 핵심은 transition matrix
    - 시간에 따른 사용자 선호 변화를 포착
    - time step 은 1 분기로, time step 별 probability vector 는 4000 차원으로
    - t-1 데이터로 t 가 되는 선형회귀 모델 학습
    - transition matrix A 의 aij 가 의미하는 바
        - t-1 의 장르 i 가 t 의 장르 j 가 될 확률 
* Structure in Long-Term Preferences
    - transition matrix 는 직관적인 설명이 가능함
    - A > B > C 라는 사용자 선호 경로가 있다고 가정
    - 사용자가 C 콘텐츠를 사용할 수 있도록, 연결고리인 B 콘텐츠를 먼저 제공함
        - 예. 클래식을 듣는 사람은 late romantic era 를 듣고 20 세기 음악으로 넘어간다.



### 3. 시스템 구현 힌트

# Automated Online Experiments
* Model selection problem
    - ML 요소가 전체 서비스에서 일부분일 뿐이기 때문에, 전통적인 모델 선택 방법을 사용할 수 없음
    - 실무에서는 A/B 테스트를 많이 활용하지만, 자칫 나쁜 모델이 운영 환경에 배포될 위험이 있음
    - AOE 는 효율적으로 최고의 모델을 모델 pool 에서 선택함
    - 선택은 Bayesian surrogate model 이용
    - 사용자의 immediate feedback 을 이용해서 metric 을 예측
        - Click-through Rate, frequency of track completion
    - acquisition function 이 metric 를 이용하여 모델 랭킹
    - 모델 수렴 혹은 할당한 자원 내에서 반복  
* Bayesian surrogate model
    - accumulated metric(immediate feedback 기대값) 최대화를 목적으로 함
    - immediate feedback 의 분포는 불명이므로, Gaussian Process(GP) surrogate model 을 사용하여 모델링함
    - 위 모델은 user decision과 user feature 만을 인풋으로 사용함 (후보 모델들은 인풋되지 않음)
        - 선택 대상 모델이 무엇이든 GP 를 사용할 수 있는 이유
    - 다만, 데이터 포인트가 많을 경우, GP 업데이트는 일반적으로 expensive 하기 때문에, variational sparse GP approximations 를 inference 에 사용함
    - GP surrogate model 을 사용하여 immediate feedback 의 분포와 나아가 후보 모델의 accumulated metric 의 분포도 구할 수 있음
    - 후보 모델별 예측 accumulated metric 확률을 이용해서 exploring and exploiting 과정을 효율화할 수 있음
* Choosing the next online experiment
    - acquisition function 을 이용해 다음 online experiment 를 정함
        - GP mean 을 최대화하고 GP variance 를 최소화함 (exploration-exploitation trade-off)
    - acquisition function 고려 대상 (Bayesian Optimization 에서 자주 사용하는 것으로 선택)
        - expected improvement
        - probability of improvement
        - upper confidence bound
    - 일반적인 BO 와의 차이
        - space of choices 가 다름
            - 후보 모델이 surrogate model 의 input space(예. user decision, user feature)와 다름
            - 때문에 acquisition function 으로 entropy search 를 사용할 수 없음
            - 대신 surrogate model 을 user decision, user feature 의 space로 정의할 수 있음
                - 일반적으로 structured domain 이기 때문에 정의하기가 비교적 용이함


* airbnb 는 어떨까?

