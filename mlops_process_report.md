# Git, DVC, MLflow 기반 AI 수요예측 모델 관리 보고서

## 1. 실습 목적

본 실습의 목적은 AI 채용 공고 데이터를 활용하여 고연봉 여부를 예측하는 모델을 학습하고, Git, DVC, MLflow를 이용해 코드, 데이터, 모델 실험 결과를 체계적으로 관리하는 것이다.

이번 실습에서는 단순히 모델을 학습하는 데서 끝내지 않고, 데이터셋을 DVC로 관리하고, 4개의 머신러닝 모델을 MLflow에 기록한 뒤, 성능 비교를 통해 최종 모델을 선정하였다.

## 2. 전체 진행 과정

| 단계 | 사용 도구 | 수행 내용 | 결과 |
|---|---|---|---|
| 1 | Git | 프로젝트 코드 버전 관리 설정 | 모델링 파일 및 MLflow 스크립트 커밋 |
| 2 | DVC | 데이터셋 추적 설정 | CSV 원본은 DVC로 관리, `.dvc` 파일은 Git에 저장 |
| 3 | Google Drive | DVC remote storage 연결 | 데이터 원본을 Google Drive에 저장 |
| 4 | MLflow | 4개 모델 실험 기록 | 모델별 run, metric, parameter, artifact 저장 |
| 5 | MLflow Registry | 최고 모델 등록 | LightGBM을 `salary_classifier@production`으로 등록 |

## 3. Git 기반 코드 관리 결과

Git은 모델링 코드와 MLflow 학습 스크립트의 변경 이력을 관리하는 데 사용하였다.

Git에 포함한 주요 파일과 역할은 다음과 같다.

| 파일 | 역할 |
|---|---|
| `logistic_regression_salary_cross_validation.ipynb` | Logistic Regression 모델링 및 교차검증 과정을 정리한 노트북 |
| `random_forest_salary_random_split.ipynb` | Random Forest 모델 학습 및 평가 과정을 정리한 노트북 |
| `lightgbm_salary_random_split.ipynb` | LightGBM 모델 학습 및 평가 과정을 정리한 노트북 |
| `catboost_salary_random_split.ipynb` | CatBoost 모델 학습 및 평가 과정을 정리한 노트북 |
| `train_salary_models_with_mlflow.py` | 4개 모델을 한 번에 학습하고 MLflow에 실험 결과를 기록하는 통합 학습 스크립트 |
| `data/salary_prediction_monthly_dataset.csv.dvc` | DVC가 데이터셋 원본 파일을 추적하기 위해 생성한 메타데이터 파일 |
| `.gitignore` | Git에 올리지 않을 파일과 폴더를 지정하는 설정 파일 |

반대로 데이터 CSV 파일, MLflow 로컬 산출물, 가상환경 폴더는 Git에 직접 업로드하지 않도록 `.gitignore`에 등록하였다.

| 제외 대상 | 제외 이유 |
|---|---|
| `venv/` | Python 가상환경 폴더로, 용량이 크고 사용자 환경마다 달라 Git에 올리지 않음 |
| `mlruns/` | MLflow 로컬 실험 기록 폴더로, 로컬 실행 산출물이므로 Git 관리 대상에서 제외 |
| `mlartifacts/` | MLflow 모델 artifact 저장 폴더로, 학습 결과 산출물이므로 Git에 직접 올리지 않음 |
| `mlflow.db` | 로컬 MLflow Tracking Server가 사용하는 SQLite DB 파일로, 로컬 실행 결과이므로 제외 |
| `.dvc/cache/` | DVC가 실제 데이터 파일을 캐싱하는 내부 폴더로, 용량이 커질 수 있어 Git에 올리지 않음 |
| `.dvc/tmp/` | DVC 실행 중 생성되는 임시 파일 폴더로, 버전 관리 대상이 아님 |

### 시각 자료 추천

- GitHub 저장소 파일 목록 화면
- `train_salary_models_with_mlflow.py`가 GitHub에 올라간 화면
- `data/salary_prediction_monthly_dataset.csv.dvc` 파일이 GitHub에 올라간 화면

## 4. DVC 기반 데이터 관리 결과

데이터셋은 Git에 직접 올리지 않고 DVC로 추적하였다. 이는 데이터 파일이 커질 경우 Git 저장소가 무거워지는 문제를 방지하기 위한 것이다.

관리 대상 데이터셋은 다음 파일이다.

```text
data/salary_prediction_monthly_dataset.csv
```

DVC 추적 파일은 다음과 같다.

```text
data/salary_prediction_monthly_dataset.csv.dvc
```

DVC remote storage는 Google Drive로 설정하였다.

```text
remote name: mlops_dvc_storage
remote type: Google Drive
```

이 구조를 통해 Git에는 데이터의 메타데이터만 저장하고, 실제 데이터 원본은 Google Drive에 저장하였다.

### 시각 자료 추천

- Google Drive에 DVC 저장소 폴더가 생성된 화면
- GitHub에서 `.dvc` 파일만 올라가 있고 CSV 원본은 제외된 화면
- 터미널에서 `dvc status` 또는 `dvc push` 성공 화면

## 5. MLflow 실험 관리 과정

MLflow에서는 4개 모델을 하나의 Experiment 안에 각각의 run으로 기록하였다.

Experiment 이름은 다음과 같다.

```text
salary_prediction_models
```

실험에 사용한 모델은 다음 4개이다.

| 모델 | MLflow run 이름 |
|---|---|
| Logistic Regression | `LogisticRegression` |
| Random Forest | `RandomForest` |
| LightGBM | `LightGBM` |
| CatBoost | `CatBoost` |

각 run에는 다음 항목을 기록하였다.

| 기록 항목 | 내용 |
|---|---|
| Parameters | 모델별 주요 하이퍼파라미터 |
| Metrics | accuracy, precision, recall, f1, roc_auc, pr_auc, log_loss |
| Model artifact | 학습 완료된 모델 파일 |
| Source | `train_salary_models_with_mlflow.py` |

### 시각 자료 추천

- MLflow Home 화면에서 `salary_prediction_models` Experiment가 보이는 화면
- MLflow Runs 화면에서 4개 모델 run이 보이는 화면
- MLflow Chart View에서 metric 그래프가 보이는 화면

## 6. 모델 성능 비교 결과

4개 모델을 동일한 데이터셋과 동일한 train/test split 기준으로 학습한 뒤 MLflow에 성능 지표를 기록하였다.

주요 비교 기준은 `roc_auc`로 설정하였다. `roc_auc`는 분류 모델이 양성과 음성을 얼마나 잘 구분하는지 평가하는 지표이며, 값이 높을수록 성능이 좋다.

| 순위 | 모델 | ROC-AUC | F1 | Accuracy |
|---:|---|---:|---:|---:|
| 1 | Logistic Regression | 0.9826 | 0.8604 | 0.9310 |
| 2 | CatBoost | 0.9819 | 0.8745 | 0.9303 |
| 3 | LightGBM | 0.9806 | 0.8734 | 0.9310 |
| 4 | Random Forest | 0.9792 | 0.8789 | 0.9370 |

ROC-AUC 기준으로는 Logistic Regression이 0.9826으로 가장 높았고, LightGBM은 0.9806으로 매우 근접한 성능을 보였다.

프로젝트 회의에서는 단일 지표 1위 여부만이 아니라 여러 성능 지표의 균형, 모델 운용 가능성, 트리 기반 모델의 해석 및 활용성을 함께 고려하였다. 그 결과 LightGBM을 최종 모델로 선정하였다.

### 시각 자료 추천

- MLflow Chart View의 `roc_auc` 비교 그래프
- MLflow Chart View의 `f1` 비교 그래프
- MLflow Chart View의 `accuracy` 비교 그래프
- MLflow Chart View의 `log_loss` 비교 그래프

## 7. 최종 모델 선정 결과

최종 모델은 프로젝트 회의에서 성능 지표의 균형과 모델 운용 가능성을 종합적으로 고려하여 LightGBM으로 선정하였다.

```text
최종 선정 모델: LightGBM
선정 기준: 프로젝트 회의 기반 종합 판단
ROC-AUC: 0.9806
F1: 0.8734
Accuracy: 0.9310
```

선정된 LightGBM 모델은 MLflow Model Registry에 다음 이름으로 등록하였다.

```text
salary_classifier
```

등록된 모델 버전은 다음과 같다.

```text
Version: 4
Alias: production
Source Run: LightGBM
```

즉 현재 배포 가능한 대표 모델은 다음과 같이 표현할 수 있다.

```text
salary_classifier@production
```

이 모델은 코드에서 다음 방식으로 불러올 수 있다.

```python
mlflow.sklearn.load_model("models:/salary_classifier@production")
```

### 시각 자료 추천

- MLflow Models 화면에서 `salary_classifier`가 보이는 화면
- `salary_classifier` Version 4 상세 화면
- Version 4 화면에서 `Source Run: LightGBM`이 보이는 부분
- Version 4 화면에서 `Aliases: @production`이 보이는 부분

## 8. 최종 결과 요약

이번 실습을 통해 Git, DVC, MLflow를 연계하여 모델 관리 흐름을 구성하였다.

| 관리 대상 | 사용 도구 | 결과 |
|---|---|---|
| 코드 | Git | 모델링 노트북과 MLflow 학습 스크립트 버전 관리 |
| 데이터 | DVC + Google Drive | 데이터셋 원본은 원격 저장소에 저장, Git에는 `.dvc` 파일만 저장 |
| 실험 | MLflow Runs | 4개 모델의 성능 지표와 파라미터 기록 |
| 최종 모델 | MLflow Registry | LightGBM을 `salary_classifier@production`으로 등록 |

최종적으로 LightGBM 모델이 여러 성능 지표의 균형과 프로젝트 회의 결과를 바탕으로 대표 모델로 선정되었다. MLflow Registry에 등록된 `salary_classifier@production`을 통해 이후 예측 API나 배포 코드에서 동일한 모델을 불러올 수 있다.

## 9. 보고서에 넣으면 좋은 이미지 구성

보고서에는 다음 순서로 이미지를 배치하면 과정과 결과가 자연스럽게 보인다.

| 그림 번호 | 추천 이미지 | 강조할 내용 |
|---|---|---|
| 그림 1 | GitHub 저장소 파일 목록 | 코드와 `.dvc` 파일이 Git에 올라간 결과 |
| 그림 2 | Google Drive DVC 저장 폴더 | 데이터 원본이 Git이 아닌 원격 저장소에 저장됨 |
| 그림 3 | MLflow Home 화면 | `salary_prediction_models` Experiment 생성 확인 |
| 그림 4 | MLflow Runs 목록 | 4개 모델 run이 각각 기록됨 |
| 그림 5 | MLflow Chart View | 모델별 metric 비교 결과 |
| 그림 6 | ROC-AUC 차트 | LightGBM을 최종 선정 모델로 등록 |
| 그림 7 | MLflow Models 목록 | `salary_classifier` 등록 확인 |
| 그림 8 | Version 4 상세 화면 | `Source Run: LightGBM`, `Alias: production` 확인 |

이미지를 많이 넣는다면 본문 설명은 길게 쓰기보다, 각 이미지 아래에 한 줄 해석을 붙이는 방식이 좋다.

예시는 다음과 같다.

```text
그림 4. MLflow Runs 화면에서 4개 모델이 각각의 run으로 기록된 것을 확인할 수 있다.
그림 6. 성능 지표 비교 결과 LightGBM은 ROC-AUC, F1, Accuracy에서 안정적인 성능을 보였다.
그림 8. 최종 모델인 LightGBM이 salary_classifier Version 4로 등록되었고 production alias가 부여되었다.
```
