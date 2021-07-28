* 원문링크: [pinterest 블로그](https://medium.com/pinterest-engineering/)


### pinterest metric
[원문링크](https://medium.com/pinterest-engineering/pinvestigator-a-tool-for-exploring-experiment-related-changes-a58f143f83c)
* 주요 metric 과 질문
    - active 고객 수 (pinterest 의 가장 중요한 지표)
    - 기존 고객들은 행복한가?
    - 가입 고객 수는 늘어나고 있는가?
    - 신규 고객들은 행복한가?


   
### anomalous metrics
[원문링크](https://medium.com/pinterest-engineering/learning-about-your-business-from-anomalous-metrics-7d2b7ac0bea9)
* 서비스가 제대로 돌아가고 있는지 확인
    - 최근 4~6 주간의 일일 변화량을 정규 분포로 가정
    - 오늘 발생한 변화량이 위의 분포에서 얼마나 벗어나는지 확인
    - p-value threshold 는 0.001 로 함


   
### Trapped in the Present
[원문링크](https://medium.com/pinterest-engineering/trapped-in-the-present-how-engagement-bias-in-short-run-experiments-can-blind-you-to-long-run-58b55ad3bda0)

* Engagement bias
    - user 분류
        + Core: 1주에 몇 번씩 서비스를 사용하고, 주기적으로 pin 을 저장함
        + Casual: 주에 한 번씩은 서비스를 사용하고, 이따금씩 pin 을 저장함
        + Marginal: 가끔씩 서비스를 사용하고, pin 저장은 자주하지 않음
    - 실험 기간을 짧게할 경우, core user 들이 많이 참여해서, engagement bias 를 만들 수 있음
    - 하지만 장기적으로 봤을 때, marginal user 들이 서비스에 미치는 영향을 무시할 수 없음
    - core user 가 언제까지고 core user 일 수도 없음
    - 어떻게 이 문제를 해결할까?
        + 실험기간을 길게 가져감
        + user 분류에 따라 실험을 나눔
        + 서비스가 안정화되어있다면, Bayesian method 사용하여 engagement level 별로 다른 prior 적용


   
### multitask learning (2020 년)
  **사용자 feedback 의 경중을 loss function 에 반영**

[원문링크](https://medium.com/pinterest-engineering/multi-task-learning-for-related-products-recommendations-at-pinterest-62684f631c12)
1. Background
    - engagement 의 종류가 여러개: close-up, save, clickthrough(관련 링크로 리디렉트), long-click
    - 기존에는 engagement 가 있으면 1, 없으면 0 으로 봤음
    - engagement 의 biz value 를 loss function 에 반영
        + long-click 은 높은 weight
        + ![loss function](https://miro.medium.com/max/1400/1*EiNzp6gI_oZ06PoDeZGZfA.png)
    - 이 경우의 문제
        + 한 사용자의 한 아이템에 대한 engagement 가 여러개일 경우, 하나는 없애야 함
        + importance weight 가 작위적이라 모델의 스코어를 해석하기 어려움
        + 모델이 biz value 와 연결되어, importance weight 가 변경되면, 모델을 다시 학습 해야함
2. Multi-task learning Model
    - 아웃풋 뉴런을 하나에서 4 개(engagement 개수)로 변경
    - ![네트워크 구조](https://miro.medium.com/max/1400/0*d8ACfc7_s5t_IlJk)
    - loss function 에는 task 별 iteration 이 추가됨
    - 기존 loss function 에 비해 각각의 engagement 정보가 보존됨
    - 4 개의 output head 는 각각의 engagement 로 부터 지식을 빌려올 수 있음
    - 하나의 모델이 하나의 engagement 를 예측하는 것에 비해 overfitting 을 해소하는 효과도 있음
3. Calibration and interpretablility
    - 학습셋과 검증셋의 engagement label distribution 이 다르므로, 이를 조정해주어야함
4. Results
    - output head 에 utility function 을 두어서, biz value 와 모델을 decouple
    - online A/B test 결과, 모든 engagement 에서 성능이 향상되었음
    - 여러가지 task 를 추가해볼 수 있는 기회가 열렸음
        + *이를테면, 상세조회 들어갔다가 다시 되돌아 나가는 것 예측?*
5. Utility function and Bayesian optimization
    - utility function 에는 biz value 를 감안한 importance weight 가 들어감
    - 이를 Bayesian optimization 으로 해결하려했음
    - 근데 성능이 좋지 못했음. offline 테스트라서 그런것 같음. online 으로 해봐야겠음


   
### pinability (2017 년)
  **피쳐가 아주 많고, 모델은 비교적 단순**

[원문링크](https://medium.com/pinterest-engineering/how-machine-learning-significantly-improves-engagement-abroad-98c6ca937f9f)
* Pinability 요약
    - linear model -> SVM -> GBDT (gradient boosted decision trees)
    - GBDT 로 home feed 한 이후 pin 을 저장하는 비율이 10 % 증가함
    - hive 와 cascading 을 학습 데이터 생성에 사용
    - 오프라인으로 xgboost 모델 학습
    - dense feature 는 700 여개, sparse feature 는 350 여개
    - adaptive training process 개발하여, instance 마다 다른 weight 적용
        + saved Pin 의 weight 를 clickthrough 보다 2 배 크게
    - weight 를 적절히 조정하는것이 engagement 를 높여주었음


   
### pixie
![pixie image](https://miro.medium.com/max/1400/0*hJbZbrsZS6ycEZ-u)

