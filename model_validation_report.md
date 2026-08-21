# 수요예측 모델 검증 요약

검증일: 2026-05-31

## 검증 대상

- `LogisticRegression.ipynb`
- `RandomForestClassifier.ipynb`
- `XGBoost.ipynb`
- `LightGBM.ipynb`

네 모델 모두 `high_salary`를 예측하는 이진 분류 모델이며, 노트북 기준으로 마지막 `posting_month` 값, 즉 2025년 4월 테스트셋을 이용해 성능을 평가한다.

## 실행 가능성 점검

현재 작업 폴더에는 노트북 4개만 있고 원천 CSV 파일은 없다.

- `LogisticRegression.ipynb`: `C:\Users\rdp-user\Downloads\jswBad_1.csv`
- 나머지 3개 노트북: `C:\Users\rdp-user\Downloads\jswBad_2.csv`

현재 PC에서는 위 두 CSV 경로가 존재하지 않아 노트북을 즉시 재실행할 수 없다. 아래 성능 비교는 각 노트북에 저장된 기존 실행 결과 기준이다.

## 성능 비교

| 순위 | 모델 | Accuracy | Precision | Recall | F1 | ROC-AUC | 테스트 건수 |
|---:|---|---:|---:|---:|---:|---:|---:|
| 1 | LightGBM | 0.9330 | 0.8115 | 0.9505 | 0.8755 | 0.9852 | 895 |
| 2 | XGBoost | 0.9296 | 0.7978 | 0.9595 | 0.8712 | 0.9853 | 895 |
| 3 | LogisticRegression | 0.9229 | 0.7762 | 0.9685 | 0.8617 | 0.9847 | 895 |
| 4 | RandomForestClassifier | 0.9050 | 0.7751 | 0.8694 | 0.8195 | 0.9678 | 895 |

테스트셋 클래스 분포는 `0: 673건`, `1: 222건`이다. `high_salary=1`을 놓치지 않는 것이 중요하면 Recall이 높은 LogisticRegression 또는 XGBoost가 유리하다. Precision과 F1까지 균형 있게 보면 LightGBM이 가장 안정적이다.

## 1차 결론

현재 저장된 결과만 보면 최종 후보는 LightGBM이다.

- F1이 가장 높다.
- Precision이 가장 높다.
- Recall도 0.9505로 충분히 높다.
- ROC-AUC는 XGBoost가 0.9853으로 아주 근소하게 높지만, 차이가 0.0001 수준이라 실무적으로는 동률에 가깝다.

따라서 기본 추천은 LightGBM, `high_salary=1`을 최대한 많이 잡는 것이 최우선이면 LogisticRegression 또는 XGBoost를 보조 후보로 둔다.

## 검증상 주의사항

현재 비교는 완전히 공정한 모델 비교라고 보기 어렵다.

1. LogisticRegression은 `jswBad_1.csv`를 사용하고, 나머지 모델은 `jswBad_2.csv`를 사용한다.
2. LogisticRegression 데이터는 86개 컬럼이고, 나머지 모델 데이터는 34개 컬럼이다.
3. 모든 모델이 `high_salary`를 제외한 모든 컬럼을 입력으로 사용한다. `posting_month`, `quarter`도 입력에 포함된다.
4. 원천 CSV가 현재 경로에 없어 재실행 검증, 임계값 튜닝, 교차검증, 반복 실험을 아직 수행할 수 없다.

## 다음 검증 권장안

최종 모델을 확정하려면 아래 순서로 재검증하는 것이 좋다.

1. CSV 파일을 현재 작업 폴더에 복사하거나 노트북의 `DATA_PATH`를 실제 존재하는 경로로 수정한다.
2. 4개 모델 모두 동일한 데이터셋과 동일한 입력 컬럼으로 다시 학습한다.
3. 2025년 4월 단일 테스트뿐 아니라 시간순 롤링 검증을 추가한다.
4. 0.5 고정 임계값 외에 F1 최대, Recall 우선, Precision 우선 기준의 임계값을 비교한다.
5. 최종 보고 지표는 Accuracy보다 F1, Recall, Precision, ROC-AUC를 우선한다.

## 운영 선택 기준

- 균형형 운영: LightGBM
- 양성 탐지 우선: LogisticRegression 또는 XGBoost
- 설명 가능성 우선: LogisticRegression
- 안정적 성능과 변수 중요도 확인: LightGBM
