
## 📌 Project Summary | 프로젝트 요약

- **🛠 Tool | 도구**:  
  - R 
  - Libraries: mgcv, glmnet, lme4, gtsummary, ggpubr, pROC 등  
- **🧪 Design | 설계**:  
  - Binary classification problem (이진 분류 문제)  
  - 다양한 날씨 변수로 다음날 강수 여부(RainTomorrow) 예측  
  - 모델 비교: Logistic Regression, LASSO, GLMM, GAM  
- **🎯 Goal | 목표**:  
  - Predict whether it will rain tomorrow based on weather factors  
  - 기상 요소들을 기반으로 **다음날 비가 올지 예측**하는 모델 개발

---

## 📊 Variables | 변수 설명

| 변수 (Variable) | 설명 (Description) |
|------------------|--------------------|
| RainTomorrow     | 다음날 비 여부 (Yes/No) |
| MinTemp, MaxTemp | 최저/최고 기온 (°C) |
| Rainfall         | 강수량 (mm) |
| Humidity9am/3pm  | 오전/오후 습도 (%) |
| Pressure9am/3pm  | 오전/오후 기압 (hPa) |
| WindGustDir, WindDir9am/3pm | 풍향 (16방위) |
| WindGustSpeed    | 순간 최대 풍속 (km/h) |
| Temp9am/3pm      | 시간대별 기온 (°C) |

---

## 🔍 Methodology | 분석 방법

- **Data Cleaning & Imputation**  
  - 결측치는 평균/최빈값으로 보간  
  - 풍향은 숫자 방향으로 변환해 방향성 고려

- **Modeling Approaches | 모델링 방법**
  - Logistic Regression (기본 베이스라인)
  - LASSO (변수 선택 포함)
  - GAM (비선형 관계 반영)
  - GLMM (지역별 랜덤효과 포함)

- **Model Evaluation | 모델 평가**
  - Accuracy, AUC, Confusion Matrix로 비교

---

## 📈 Key Findings | 주요 결과

- GAM 및 GLMM 모델이 일반 로지스틱 회귀보다 성능이 우수
- LASSO는 변수 선택이 가능해 해석력이 뛰어남
- 일부 기온/습도/풍속 변수들이 예측에 큰 기여

---

## 🧠 Interpretation | 해석 (강조)

- ☔ **Humidity and wind-related features are strong predictors of rainfall**  
- ☔ **습도와 풍속 관련 변수는 강수 예측에서 중요한 역할을 함**  
- 🌡️ **Temperature variation alone is insufficient**, but in combination, it improves model performance  
- 🌡️ **기온만으로는 한계가 있지만 다른 변수와 결합할 경우 예측력 향상에 기여**

---

## 🧾 Conclusion | 결론 (강조)

- ✅ 다양한 날씨 요인을 활용한 통계 모델로 **다음날 강수 여부를 유의미하게 예측 가능**  
- ✅ GLMM과 GAM은 데이터의 구조적 특성을 반영해 예측 성능을 향상시킴  
- 💡 본 프로젝트는 **기상 데이터를 활용한 머신러닝/통계 모델링 예시**로 활용 가능  
- 💡 날씨 기반 의사결정(농업, 물류 등)에 실질적 도움을 줄 수 있음
