# smartphone-addiction-prediction - Logistic regression

## **1. transformer**

gender에 대해서는 one hot encoding

impact 에 대해서는 binary encoding을 적용하였다.

나머지 수치형 데이터에 대한 전처리는 바꿔가며 실험을 진행하였다.

Logistic Regression의 주요 조정 변수는 다음과 같다.

| 조정 변수 | 실험 값 |
| --- | --- |
| Scaler | StandardScaler, MinMaxScaler, RobustScaler, None |
| C | 0.01, 0.1, 1, 10, 100 |
| class_weight | None, balanced |
| penalty | L2 |

여기서 `C`는 규제 강도를 의미한다. `C`가 작을수록 규제가 강하고, `C`가 클수록 규제가 약하다. 본 실험에서는 전체 feature를 안정적으로 줄이는 **L2 규제**를 사용하였다.

## 2. 기본 모델 실험 결과

초기 실험 결과, 상위 10개 모델은 모두 `class_weight=None`을 사용한 모델이었다. 이는 `class_weight='balanced'`를 적용한 모델보다 클래스 가중치를 적용하지 않은 모델이 더 높은 weighted F1-score를 보였다는 것을 의미한다.

이는 최종 평가 지표로 `average='weighted'`를 사용했기 때문으로 해석할 수 있다. Weighted average는 클래스별 성능을 계산한 뒤, 각 클래스의 데이터 수에 비례하여 평균을 낸다. 따라서 `class_weight='balanced'`가 소수 클래스를 더 맞추도록 학습을 조정했더라도, 다수 클래스의 성능이 낮아지면 전체 weighted score가 오히려 감소할 수 있다.

기본 threshold인 0.5 기준에서 가장 높은 F1_weighted는 **0.890672**였다. 이 점수는 다음 세 모델에서 동일하게 나타났다.

| Rank | Scaler | C | class_weight | Accuracy | Precision_weighted | Recall_weighted | F1_weighted |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | StandardScaler | 10.00 | None | 0.891600 | 0.890292 | 0.891600 | 0.890672 |
| 2 | RobustScaler | 100.00 | None | 0.891600 | 0.890292 | 0.891600 | 0.890672 |
| 3 | StandardScaler | 100.00 | None | 0.891600 | 0.890292 | 0.891600 | 0.890672 |
| 4 | MinMaxScaler | 10.00 | None | 0.891600 | 0.890265 | 0.891600 | 0.890640 |
| 5 | None | 100.00 | None | 0.891467 | 0.890164 | 0.891467 | 0.890546 |
| 6 | MinMaxScaler | 100.00 | None | 0.891467 | 0.890150 | 0.891467 | 0.890530 |
| 7 | StandardScaler | 1.00 | None | 0.891333 | 0.890035 | 0.891333 | 0.890420 |
| 8 | RobustScaler | 10.00 | None | 0.891333 | 0.890021 | 0.891333 | 0.890403 |
| 9 | None | 10.00 | None | 0.891067 | 0.889722 | 0.891067 | 0.890102 |
| 10 | None | 1.00 | None | 0.891333 | 0.889801 | 0.891333 | 0.890090 |

## 3. Threshold 조정 실험

Logistic Regression은 각 샘플에 대해 클래스 1에 속할 확률을 출력한다. 기본적으로 threshold는 `0.5`이며, 예측 확률이 0.5 이상이면 1, 미만이면 0으로 분류한다.

하지만 threshold 0.5가 항상 최적은 아니므로, 기본 실험에서 1위를 차지한 모델을 기준으로 threshold를 조정하였다. 먼저 넓은 범위에서 threshold를 테스트한 결과, **0.45~0.50 사이에서 높은 성능이 나타나는 것**을 확인하였다.

1위 모델에 대한 threshold tuning 결과는 다음과 같다.

| Rank | Threshold | Accuracy | Precision_weighted | Recall_weighted | F1_weighted |
| --- | --- | --- | --- | --- | --- |
| 1 | 0.465 | 0.893733 | 0.892140 | 0.893733 | 0.892031 |
| 2 | 0.464 | 0.893600 | 0.892002 | 0.893600 | 0.891869 |
| 3 | 0.468 | 0.893467 | 0.891867 | 0.893467 | 0.891829 |
| 4 | 0.466 | 0.893467 | 0.891864 | 0.893467 | 0.891777 |
| 5 | 0.463 | 0.893467 | 0.891864 | 0.893467 | 0.891725 |

기본 threshold 0.5에서의 최고 F1_weighted가 **0.890672**였던 것과 비교하면, threshold를 0.465로 조정했을 때 **0.892031**까지 상승하였다. 따라서 threshold 조정은 모델 성능을 미세하게 개선하는 데 효과가 있었다.

---

## 4. Top 10 모델 대상 Threshold 조정

1위 모델에서 threshold 0.45~0.50 구간의 성능이 높게 나타났기 때문에, 동일한 threshold tuning을 초기 Top 10 모델 전체에 적용하였다. 각 모델에 대해 threshold를 0.45부터 0.50까지 0.001 간격으로 조정하고, weighted F1-score가 가장 높은 threshold를 선택하였다.

최종 결과는 다음과 같다.

| New Rank | Original Rank (threshold tuning 전 순위) | Scaler | C ( 규제 강도) | class_weight | Threshold | Accuracy | Precision_weighted | Recall_weighted | F1_weighted |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 9 | None | 10.0 | None | 0.488 | 0.893600 | 0.892145 | 0.893600 | 0.892424 |
| 2 | 8 | RobustScaler | 10.0 | None | 0.467 | 0.893867 | 0.892279 | 0.893867 | 0.892209 |
| 3 | 5 | None | 100.0 | None | 0.488 | 0.893333 | 0.891881 | 0.893333 | 0.892171 |
| 4 | 2 | RobustScaler | 100.0 | None | 0.487 | 0.893333 | 0.891863 | 0.893333 | 0.892138 |
| 5 | 7 | StandardScaler | 1.0 | None | 0.467 | 0.893733 | 0.892141 | 0.893733 | 0.892082 |
| 6 | 6 | MinMaxScaler | 100.0 | None | 0.468 | 0.893733 | 0.892141 | 0.893733 | 0.892082 |
| 7 | 1 | StandardScaler | 10.0 | None | 0.467 | 0.893733 | 0.892141 | 0.893733 | 0.892065 |
| 8 | 10 | None | 1.0 | None | 0.485 | 0.893333 | 0.891823 | 0.893333 | 0.892056 |
| 9 | 4 | MinMaxScaler | 10.0 | None | 0.465 | 0.893733 | 0.892140 | 0.893733 | 0.892031 |
| 10 | 3 | StandardScaler | 100.0 | None | 0.489 | 0.893067 | 0.891618 | 0.893067 | 0.891917 |

Threshold tuning 이후 최종 1위 모델은 기존 9위였던 **Scaler=None, C=10.0, class_weight=None, threshold=0.488** 조합이었다. 이 모델의 F1_weighted는 **0.892424**로, 초기 최고 점수인 **0.890672**보다 상승하였다.

---

## 5. 최종 모델 선정

최종 모델은 scaler를 적용하지 않은 상태에서 가장 높은 weighted F1-score를 기록하였다. 이는 해당 데이터의 수치형 feature들이 Logistic Regression 학습에 큰 스케일 문제를 일으키지 않았거나, 스케일링을 적용했을 때보다 원래 값의 분포가 더 적합했을 가능성을 보여준다.

---

## 6. 결론

이번 실험에서는 Logistic Regression 모델의 성능을 개선하기 위해 수치형 feature의 scaler, 규제 강도 `C`, class weight, threshold를 비교하였다.

초기 실험에서는 `StandardScaler + C=10 + class_weight=None` 조합이 가장 높은 성능을 보였지만, 상위 모델들의 성능 차이는 매우 작았다. 이후 threshold tuning을 적용한 결과, 기본 threshold 0.5보다 약간 낮은 0.45~0.50 구간에서 더 높은 성능이 나타났다.

최종적으로 **Scaler=None, C=10.0, class_weight=None, threshold=0.488** 조합이 가장 높은 weighted F1-score를 기록하였다. 따라서 본 실험에서는 하이퍼파라미터 조정보다 threshold 조정이 성능 개선에 더 직접적인 영향을 준 것으로 볼 수 있다.

