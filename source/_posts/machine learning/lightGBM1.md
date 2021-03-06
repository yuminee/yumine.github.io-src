---
title: Light GBM① - 이해하기
date: 2020-11-26 23:05:36
tags: [lightGBM, Boosting]
category: [머신러닝]
---


# Light GBM① - 이해하기

### 서론

데이터 분석 방법을 크게 두가지로 나눠본다면,

1. 음성, 이미지, 텍스트 처럼 딱 형태로 떨어지는 데이터가 아니여서 중요한 feature를 추출하는 작업이 필요할때

2. 기업 DB의 테이블 처럼 tabular 형태로 저장되어 있는 정형화된 데이터 형태. 하나 하나 칼럼이 정확한 의미를 가지고 있음.

 

첫번째의 경우는 딥러닝이다. 이미지 인식과 같은 경우는 훈련을 시키려면 feature가 있어야 하는데, 예전 같으면 HOG, SIFT와 같이 그림의 움직임이나 특성에 대해 뭔가를 뽑아내는 작업을 통했다면, 지금은 그냥 컨볼루션 신경망이 알아서 feature를 뽑아내기 때문에 힘든 작업이 필요없다.

<br>

<br>

두번째의 경우는 winning 알고리즘은 부스터이다. 부스팅이 항상 winning하는 알고리즘은 절대 아니다. 다만 각각의 상황에 따라 데이터가 칼럼으로 저장되어 있고 이를 기반으로 뭔가를 분류해야하는 (ex, 고객이탈, 퇴직 예측)의 경우에는 부스팅이 빠르고 정확하게 높은 예측률을 확보 할 수 있다.

<br>

<br>

부스팅중에 높은 예측력을 자랑하는 lightGBM을 이해해보자.

### 부스팅 (Boosting)

약한 학습기를 여러 개 연결하여 강한 학습기를 만드는 앙상블 방법

부스팅은 여러개의 트리(혹은 다른 모델)을 만드는데, 랜덤 포레스트나 배깅과 같은 기법과는 다르게 기존에 있는 예측기를 조금씩 발전시켜서 이를 합한다는 개념. 랜덤 포레스트는 병렬로 무지막지하게 많은 결정트리를 동시에 만든다면 부스팅은 점진적으로 디시전 트리를 발전시킨뒤에 이를 통합하는 과정을 거친다고 보면 된다. 부스팅은 보통 두가지 방향이 있는데,

- adaboost와 같이 중요한 데이터에 대해 weight를 주는 방식 (중요한 데이터를 순간적으로 weight 만큼 뻥튀기 시킨다고 보면 됨)
- GBDT와 같이 딥러닝의 loss function 마냥 정답지와 오답지간의 차이를 훈련에 다시 투입하여 gradient를 적극 이용해서 모델을 개선하는 방식. XGboost나 lightGBM이 여기에 속함.

아래는 gbdt의 예로https://xgboost.readthedocs.io/en/latest/tutorials/model.html에서 가져옴.

![xgboost](https://user-images.githubusercontent.com/33755241/100420230-6bcba500-30c9-11eb-9a8e-9106f8d6c707.png)

<br>

<br>

### Gradient Boosting Decision Tree

GBDT는 딥러닝에서 쉽게 볼 수 있는 개념인 gradient를 이용해서 트리를 만들어 낸다.

<br>

<br>

### Light GBM

Light GBM은 Gradient Boosting 프레임워크로 Tree 기반 학습 알고리즘.
<br>

<br>

### 기존의 다른 Tree기반 알고리즘과 차이점

Light GBM은 Tree가 수직적으로 확장되는 반면, 다른 알고리즘은 Tree가 수평적으로 확장된다. 즉, Light GBM은 leaf-wise 인 반면 다른 알고리즘은 level-wise 이다. 확장하기 위해서 max delta loss 를 가진 leaf를 선택하게 되는 것이다. 동일한  leaf를 확장할 때, leat-wise 알고리즘은 level-wise 알고리즘보다 더 많은 loss, 손실을 줄일 수 있다.

<br>

<br>

아래 다이어그램은 lightGBM와 다른 Boosting 부스팅 알고리즘의 구현을 나타내고 있다.

![Light GBM 작동 방식](https://user-images.githubusercontent.com/33755241/100421350-90288100-30cb-11eb-945c-3fd900b98c59.png)

Light GBM 작동 방식

![다른 Boosting 알고리즘 작동 방식](https://user-images.githubusercontent.com/33755241/100421358-928adb00-30cb-11eb-9e4b-c6cba642e275.png)

다른 Boosting 알고리즘 작동 방식

<br>

<br>

### Light GBM이 인기가 좋은 이유?

데이터는 날이 가면 갈수록 증가하고 기존 알고리즘으로 빠른 결과를 얻기는 점점 힘들어 진다. LightGBM은 **빠른 속도**때문에 말 그대로  'Light' 하다.  Light GBM은 큰 사이즈의 데이터를 다룰 수 있으며, 돌리기 위해 적은 메모리를 사용한다.  Light GBM이 인기가 좋은 또 다른 이유는 결과의 정확도에 초점이 맞춰져있기 때문이다.  LGBM은 GPU를 지원하기 때문에 데이터 사이언티스트들이 데이터 분석 어플리케이션을 개발할때 LGBM을 폭넓게 사용하고 있다.
<br>

<br>

### Light GBM을 어디서든 사용 할 수 있을까?

Light GBM을 작은 데이터셋에 사용하는 것은 추천하지 않는다. Light GBM은 과접합에 민감하고 적은 데이터에서는 과적합 되기가 쉽다. 
<br>

<br>

### Light GBM 구현은 어떻게?

Light GBM 구현은 쉽다. 복잡한건 파라미터 튜닝(parameter tuning)이다. LightGBM은 100개 이상의 파라미터들을 가지고 있다. 하지만 전부를 알아야 할 필요는 없으니 걱정하지 마라.

<br>

<br>

### Parameters

#### Control Parameters
  - **max_depth** : maximum depth of tree. 즉, 트리의 최대 깊이. 이 파라미터는 과적합(over fitting)을 조절한다. 모델이 과적합 된것 같으면, max_depth를 줄여야 함
  -**min_data_in_leaf** :  minimum number of the records a left may have.  즉, left가 가지고 있는 최소한의 레코드. 디폴트 값은 20이며, 최적값(optimum value)이다.  이 파라미터 또한 과적합(over fitting)을 해결 할때 사용한다. 
  - **feature_fraction** : used when your boosting is random forest. 즉,  랜덤 포레스트 일 경우에 사용. 0.8 feature fraction은 LightGBM이 Tree를 만들 때 각각 iteration 반복에서 파라미터 중에서 80%를 랜덤으로 선택.
  - **bagging_fraction** : 각각 iteration 에 사용되는 데이터의 일부를 지정하는데,  일반적으로 훈련 속도를 높이거나, 과적합을 피할때 사용한다.
  - **early_stopping_round** : This parameter can help you speed up your analysis. 즉, 분석 속도를 높이는것을 도와주는 파라미터이다.  지난 early_stopping_round 라운드에서 one metric of one validation data가 향상되지 않으면 멈추게 된다. 지나친 iterations을 줄이는데 도움을 줌
  - **lambda** : lambda specifies regularization. 즉 람다의 값은 regularization을 정규화.  일반적인 값의 범위는 0에서 1사이 이다.
  - **min_gain_to_split** : his parameter will describe the minimum gain to make a split. 즉, 분기 하기 위한 최소한의 gain. tree에서 유용한 분기의 수를 조절함.
  - **max_cat_group** : 카테고리의 수가 클때, 분기 포인트를 찾음(과적합되기 쉽기 때문에). 그래서 Light GBM은 그것들을 'max_cat_group' 그룹으로 병합하고 그룹 경계에서 분기 포인트를 찾습니다. 디폴트는 64.
  <br>

<br>

#### Core Parameters
  - **Task** :  데이터로 수행하고자 하는것을 구체화 한다. Train 혹은 Pedfict
  - **Application** : 가장 중요한 파라미터. 모델의 어플리 케이션이 regression 문제인지 classification 문제인지 정함. 디폴트는 회귀모델이다.
    - regression : for regression
    - binary :for binary classification
    - multiclass : for multiclass classification problem
  - **boosting** : 돌리고 싶은 알고리즘의 타입을 정의. 디폴트는 gdbt
    - gbdt: traditional Gradient Boosting Decision Tree
    - rf: random forest
    - dart: Dropouts meet Multiple Additive Regression Trees
    - goss: Gradient-based One-Side Sampling
  - **num_boost_round** : 부스팅 이터레이션의 갯수. 주로 100개 이상
  - **learning_rate** : 각각 tree의 마지막 outcome 에 영향. GBM은 초기의 추정값에서 시작하여 각각의Tree 결과를 사용하여 추정값을 업데이트. 학습 파라미터들은 이러한 추정에서 발생하는 변화를 조절한다. 일반적인 값으로는 0.1, 0.001, 0.003... 등이 있음
  - **num_leaves** : 전체 트리의 leaves의 수
  - **device** : 디폴트: cpu, can also pass gpu

  <br>

<br>

#### Metric parameter
  - **metric** : 모델 빌딩을 위한 손실을 정함. 아래는  regression과 classification 을 위한 일반적인 손실 값
    - mae: mean absolute error
    - mse: mean squared error
    - binary_logloss: loss for binary classification
    - multi_logloss: loss for multi classification

    <br>

<br>

#### IO parameter
  - **max_bin** :  feature 값의 최대 bin 수
  - **categorical_feature** : 범주형 feature의 인덱스.  만약 categorical_features 가 0,1,2 이면 column 0, column 1, column 2가 categorical variables.
  - **ignore_column** : categorical_features와 동일한 것인데 범주형 feature로써 특정 칼럼을 고려하지 않는 것. 간단히 말해 그 변수들을 무시함.
  - **save_binary** : 데이터 파일의 메모리 사이즈를 해결해야 할때는 이 파라미터를 'True'로 설정. Ture로 하면 데이터셋을 Binary file로 저장함. 바이너리 파일은 다음에 데이터를 읽을때 속도를 올려 줄 것 임.



<br>

<br>

다음 포스트에서는 위의 파라미터들을 이용하여 LightGBM을 구현해 볼 것 이다. 

- 참고한 자료들
    - https://medium.com/@pushkarmandot/https-medium-com-pushkarmandot-what-is-lightgbm-how-to-implement-it-how-to-fine-tune-the-parameters-60347819b7fc
    - https://www.youtube.com/watch?v=1dSfWpFnvP0

