# 모델별 비교

상태: 진행 중

# AI 직무 스킬 수요 분석 모델 정리

## 1. 프로젝트 개요

### 프로젝트 주제

AI 직무 스킬 수요 분석 및 취준생 커리어/기술 스택 제안

### 프로젝트 목표

- AI 직무 채용 데이터에서 직무별 중요 스킬을 확인한다.
- 기업이 요구하는 스킬과 `high_salary`와 관련 있는 요인을 분석한다.
- 취준생이 희망 직무와 보유 기술 스택을 입력했을 때, 데이터셋 기준으로 우선 확인할 기술 스택을 제안한다.
- 여러 분류 모델을 학습하고 2025년 4월 테스트 데이터 기준 성능을 검증한다.

---

## 2. 사용 데이터셋

| 모델 | 사용 데이터셋 | 비고 |
| --- | --- | --- |
| Logistic Regression | `jswBad_1.csv` | 로지스틱 회귀용 스케일링/원-핫 데이터 |
| RandomForestClassifier | `jswBad_2.csv` | 트리 기반 모델용 숫자 인코딩/원-핫 데이터 |
| GradientBoostingClassifier | `jswBad_2.csv` | 트리 기반 모델용 숫자 인코딩/원-핫 데이터 |
| XGBoost | `jswBad_2.csv` | 트리 기반 모델용 숫자 인코딩/원-핫 데이터 |
| LightGBM | `jswBad_2.csv` | 트리 기반 모델용 숫자 인코딩/원-핫 데이터 |

### 공통 데이터 사용 원칙

- 외부 데이터는 사용하지 않았다.
- 원본 라벨 복원은 하지 않았다.
- `posting_month`, `quarter`, `years_experience`, `job_title`, `company_location`, `industry` 등은 데이터셋에 들어 있는 값 그대로 사용했다.
- 스킬 수요는 별도 예측 모델로 만들지 않았다.
- 스킬 분석은 `skill_*` 컬럼의 실제 등장률만 사용했다.
- 테스트 예측 결과는 각 모델의 `predicted_high_salary`, `predicted_high_salary_probability`로 확인했다.

---

## 3. 학습/테스트 분리

| 구분 | 내용 |
| --- | --- |
| 학습 데이터 | 2024년 1분기 ~ 2025년 1분기 |
| 테스트 데이터 | 2025년 4월 |
| 학습 데이터 수 | 14,105개 |
| 테스트 데이터 수 | 895개 |
| 테스트셋 실제 `high_salary=1` 건수 | 222개 |
| 테스트셋 실제 `high_salary=1` 비율 | 24.80% |

---

## 4. 모델별 검증 성능

### 성능 지표 용어 설명

| 용어 | 쉬운 설명 | 이 프로젝트에서의 의미 |
| --- | --- | --- |
| Accuracy | 전체 정답률 | 테스트 데이터 전체에서 모델이 `high_salary`를 맞힌 비율 |
| Precision | 정밀도 | 모델이 `high_salary=1`이라고 예측한 공고 중 실제로도 `high_salary=1`인 비율 |
| Recall | 재현율 | 실제 `high_salary=1`인 공고 중 모델이 놓치지 않고 찾아낸 비율 |
| F1-score | Precision과 Recall의 균형 점수 | `high_salary=1`을 너무 많이 틀리게 잡지도 않고, 너무 많이 놓치지도 않는 균형 지표 |
| ROC-AUC | 분류 구분 능력 | `high_salary=1`과 `high_salary=0`을 얼마나 잘 구분하는지 나타내는 점수 |

### 지표를 해석하는 방법

- `Accuracy`가 높다: 전체적으로 맞힌 비율이 높다.
- `Precision`이 높다: 모델이 `high_salary=1`이라고 한 예측을 더 믿을 수 있다.
- `Recall`이 높다: 실제 `high_salary=1`인 공고를 덜 놓친다.
- `F1-score`가 높다: Precision과 Recall의 균형이 좋다.
- `ROC-AUC`가 높다: 모델이 두 클래스를 구분하는 능력이 좋다.

이 프로젝트에서는 `high_salary=1` 데이터가 전체보다 적기 때문에 Accuracy만 보면 부족하다. 그래서 Precision, Recall, F1-score, ROC-AUC를 함께 확인했다.

| 모델 | Accuracy | Precision | Recall | F1-score | ROC-AUC |
| --- | --- | --- | --- | --- | --- |
| Logistic Regression | 0.9229 | 0.7762 | 0.9685 | 0.8617 | 0.9847 |
| RandomForestClassifier | 0.9050 | 0.7751 | 0.8694 | 0.8195 | 0.9678 |
| GradientBoostingClassifier | 0.9307 | 0.8008 | 0.9595 | 0.8730 | 0.9844 |
| XGBoost | 0.9296 | 0.7978 | 0.9595 | 0.8712 | 0.9853 |
| LightGBM | 0.9330 | 0.8115 | 0.9505 | 0.8755 | 0.9852 |

### 성능 해석

- `Accuracy`는 LightGBM이 가장 높다. 즉, 전체 테스트 데이터에서 가장 많이 맞혔다.
- `Precision`도 LightGBM이 가장 높다. 즉, LightGBM이 `high_salary=1`이라고 예측한 결과의 신뢰도가 가장 높았다.
- `Recall`은 Logistic Regression이 가장 높다. 즉, 실제 `high_salary=1`인 공고를 가장 덜 놓쳤다.
- `F1-score`는 LightGBM이 가장 높다. 즉, `high_salary=1` 예측에서 정밀도와 재현율의 균형이 가장 좋았다.
- `ROC-AUC`는 XGBoost가 가장 높지만, LightGBM과 차이가 매우 작다. 즉, 두 모델 모두 `high_salary=1`과 `0`을 잘 구분한다.

---

## 5. 2025년 4월 테스트 예측치 요약

### 예측치 관련 용어 설명

| 용어 | 쉬운 설명 |
| --- | --- |
| 예측 `high_salary=0` | 모델이 해당 공고를 높은 연봉 그룹이 아니라고 분류한 것 |
| 예측 `high_salary=1` | 모델이 해당 공고를 높은 연봉 그룹이라고 분류한 것 |
| 예측 확률 | 모델이 `high_salary=1`일 가능성을 몇 %로 봤는지 나타낸 값 |
| 평균 예측 확률 | 테스트 데이터 전체에 대해 모델이 평균적으로 `high_salary=1` 가능성을 얼마나 줬는지 |

예를 들어 평균 예측 확률이 30%라는 것은 모델이 테스트 공고들을 평균적으로 `high_salary=1`일 가능성 30% 정도로 봤다는 뜻이다.

| 모델 | 예측 `high_salary=0` 건수 | 예측 `high_salary=0` 비율 | 예측 `high_salary=1` 건수 | 예측 `high_salary=1` 비율 | 평균 `high_salary=1` 예측 확률 |
| --- | --- | --- | --- | --- | --- |
| Logistic Regression | 618 | 69.05% | 277 | 30.95% | 30.31% |
| RandomForestClassifier | 646 | 72.18% | 249 | 27.82% | 26.94% |
| GradientBoostingClassifier | 629 | 70.28% | 266 | 29.72% | 30.06% |
| XGBoost | 628 | 70.17% | 267 | 29.83% | 29.11% |
| LightGBM | 635 | 70.95% | 260 | 29.05% | 27.75% |

### 예측치 해석

- 실제 테스트셋의 `high_salary=1` 비율은 24.80%이다.
- 모든 모델이 실제 비율보다 `high_salary=1`을 다소 많이 예측했다.
- Logistic Regression은 `high_salary=1` 예측 비율이 30.95%로 가장 높다. 즉, 높은 연봉 그룹이라고 분류한 공고가 가장 많았다.
- RandomForestClassifier는 `high_salary=1` 예측 비율이 27.82%로 실제 비율에 가장 가깝다. 즉, 예측한 높은 연봉 그룹의 양은 실제 분포와 가장 비슷했다.
- LightGBM은 높은 성능을 유지하면서 예측 비율도 과도하게 높지 않은 편이다.

---

## 6. 모델별 장단점

### Logistic Regression

용어 설명

`Logistic Regression`은 입력 변수와 타깃 사이의 관계를 계수로 표현하는 분류 모델이다. 계수가 양수이면 `high_salary=1`과 양의 방향, 음수이면 음의 방향으로 해석할 수 있다.

장점

- 모델 구조가 단순하고 해석이 쉽다.
- 회귀 계수로 `high_salary=1`과 양의 방향/음의 방향인 변수를 확인할 수 있다.
- Recall이 가장 높아 `high_salary=1`을 놓치지 않는 데 강하다.

단점

- Precision이 낮은 편이라 `high_salary=1`로 예측한 것 중 오탐이 비교적 많을 수 있다.
- 비선형 관계나 변수 간 복잡한 상호작용을 잘 반영하기 어렵다.

적합한 경우

- 해석 가능성이 중요할 때
- 발표에서 모델 원리를 간단히 설명해야 할 때
- 변수 방향성을 설명해야 할 때

### RandomForestClassifier

용어 설명

`RandomForestClassifier`는 여러 개의 결정트리를 만들고, 여러 트리의 판단을 종합해서 최종 분류를 하는 모델이다. 하나의 트리보다 안정적인 결과를 얻기 쉽다.

장점

- 트리 기반 모델이라 비선형 관계를 반영할 수 있다.
- 변수 중요도를 확인할 수 있다.
- 예측 `high_salary=1` 비율이 실제 비율에 가장 가까운 편이다.

단점

- 전체 성능이 5개 모델 중 가장 낮다.
- Recall과 F1-score가 다른 부스팅 모델보다 낮다.

적합한 경우

- 기본적인 트리 앙상블 기준 모델이 필요할 때
- 변수 중요도를 간단히 확인하고 싶을 때
- 과하게 복잡하지 않은 비교 기준 모델로 사용할 때

### GradientBoostingClassifier

용어 설명

`GradientBoostingClassifier`는 작은 결정트리들을 순서대로 학습시키면서 이전 모델이 틀린 부분을 보완하는 부스팅 모델이다.

장점

- Accuracy, F1-score, ROC-AUC가 모두 높은 편이다.
- RandomForest보다 성능이 좋다.
- sklearn 기본 모델이라 별도 외부 부스팅 라이브러리 없이 사용할 수 있다.

단점

- XGBoost, LightGBM보다 대규모 데이터나 튜닝 관점에서 확장성이 낮을 수 있다.
- 모델 해석은 로지스틱 회귀보다 어렵다.

적합한 경우

- sklearn 기반으로 안정적인 부스팅 모델을 사용하고 싶을 때
- 외부 라이브러리 설치 없이 부스팅 모델을 구현해야 할 때

### XGBoost

용어 설명

`XGBoost`는 Gradient Boosting을 더 강력하고 효율적으로 만든 부스팅 모델이다. 성능이 좋아 많이 사용되지만, 하이퍼파라미터가 많아 설정이 중요하다.

장점

- ROC-AUC가 가장 높다.
- F1-score와 Accuracy도 매우 높다.
- 부스팅 모델 중 강력한 성능을 보인다.

단점

- LightGBM보다 Precision과 F1-score가 아주 약간 낮다.
- 외부 패키지 설치가 필요하다.
- 하이퍼파라미터가 많아 튜닝 난이도가 있다.

적합한 경우

- 순위화 성능이나 ROC-AUC를 중요하게 볼 때
- 강력한 부스팅 모델을 사용하고 싶을 때
- 추후 하이퍼파라미터 튜닝을 계획할 때

### LightGBM

용어 설명

`LightGBM`은 빠르고 효율적인 부스팅 모델이다. 데이터가 많거나 변수가 많을 때도 학습 속도가 빠르고 성능이 좋은 편이다.

장점

- Accuracy가 가장 높다.
- Precision이 가장 높다.
- F1-score가 가장 높다.
- ROC-AUC도 XGBoost와 거의 차이 없이 높다.
- 전체 성능 균형이 가장 좋다.

단점

- 외부 패키지 설치가 필요하다.
- 모델 해석은 로지스틱 회귀보다 어렵다.

적합한 경우

- 최종 모델 후보를 하나 고를 때
- Accuracy, Precision, F1-score를 균형 있게 보고 싶을 때
- 성능 중심의 모델 선택이 필요할 때

---

## 7. 최종 모델 추천

### 1순위: LightGBM

LightGBM을 최종 모델 1순위로 선택하는 것이 가장 적합하다.

선택 이유

- Accuracy: 0.9330으로 가장 높다.
- Precision: 0.8115로 가장 높다.
- F1-score: 0.8755로 가장 높다.
- ROC-AUC: 0.9852로 XGBoost와 거의 차이가 없다.
- 테스트 예측 `high_salary=1` 비율도 29.05%로 과도하게 높지 않다.

즉, LightGBM은 전체적인 분류 성능과 예측 안정성이 가장 균형 잡힌 모델이다.

### 2순위: XGBoost

XGBoost는 ROC-AUC가 0.9853으로 가장 높다.

따라서 `high_salary=1`과 `high_salary=0`을 구분하는 순위화 능력을 중요하게 본다면 XGBoost도 매우 좋은 후보이다.

다만 Accuracy, Precision, F1-score는 LightGBM이 약간 더 높기 때문에 최종 선택에서는 LightGBM이 조금 더 적합하다.

### 해석용 모델: Logistic Regression

최종 성능 모델로는 LightGBM이 적합하지만, 설명과 해석을 위해 Logistic Regression도 함께 활용할 수 있다.

Logistic Regression은 회귀 계수를 통해 어떤 스킬이나 직무 특성이 `high_salary=1`과 양의 방향인지 설명하기 쉽다.

---

## 8. 최종 결론

본 프로젝트에서는 5개 모델을 학습하고 2025년 4월 테스트 데이터로 검증했다.

검증 결과, `LightGBM`이 Accuracy, Precision, F1-score에서 가장 높은 성능을 보였고 ROC-AUC도 매우 높았다.

따라서 최종 예측 모델로는 `LightGBM`이 가장 적합하다.

단, 발표나 보고서에서 변수 영향 방향을 설명할 때는 `Logistic Regression`의 회귀 계수 분석을 함께 사용하는 것이 좋다.

최종적으로는 다음과 같이 역할을 나눌 수 있다.

| 역할 | 추천 모델 |
| --- | --- |
| 최종 성능 모델 | LightGBM |
| ROC-AUC 중심 대안 모델 | XGBoost |
| 해석 및 설명용 모델 | Logistic Regression |
| 기준 비교 모델 | RandomForestClassifier |
| sklearn 기반 부스팅 모델 | GradientBoostingClassifier |

---

## 9. 산출 파일

| 파일 | 설명 |
| --- | --- |
| `LogisticRegression.ipynb` | 로지스틱 회귀 모델 |
| `RandomForestClassifier.ipynb` | 랜덤포레스트 모델 |
| `GradientBoostingClassifier.ipynb` | sklearn 그래디언트 부스팅 모델 |
| `XGBoost.ipynb` | XGBoost 모델 |
| `LightGBM.ipynb` | LightGBM 모델 |

[LogisticRegression.ipynb](LogisticRegression.ipynb)

[RandomForestClassifier.ipynb](RandomForestClassifier.ipynb)

[GradientBoostingClassifier.ipynb](GradientBoostingClassifier.ipynb)

[XGBoost.ipynb](XGBoost.ipynb)

[LightGBM.ipynb](LightGBM.ipynb)