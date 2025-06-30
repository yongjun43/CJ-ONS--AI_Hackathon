# 🎬 Movie Import Success Predictor

> **해외영화 수입 시 흥행(전국 관객수·매출) 가능성을 예측**  
> 25,000 여 편의 영화 메타데이터 + 시놉시스 + 외부 공휴일 API를 결합해  
> **분류(등급별 관객 범위) · 회귀(관객 수/매출 값)** 두 가지 모델을 제공합니다.

---

## 📌 프로젝트 선정 동기
- 최근 해외영화 투자・배급 시장이 커졌지만 **손익분기점 초과 가능성**을 정량적으로 판단하기 어렵습니다.  
- 1971 ~ 2023년 **KOFIC 통합전산망** 데이터를 토대로 “수입 전 손쉽게 흥행 잠재력을 점검”할 수 있는 분석 도구를 만들고자 했습니다.

---

## 🗺️ 데이터 구성

| 출처 | 설명 | 샘플 수 |
|------|------|---------|
| **KOFIC 영화관 입장권 통합전산망** | 1971-2023 국내 상영 영화 메타데이터 | 25,008 |
| **크롤링 데이터** | 시놉시스, 주·조연 배우 | 17,000+ |
| **공휴일 API** | 개봉일 ±7 일 내 공휴일 여부 | 19,000+ |
| **파생 변수** | 감독·배우 **역대 평균 관객수/매출**, 키워드, 휴일 플래그 등 | – |

> 주요 결측치 정제 후 **최종 18,847편** 사용

---

## 🛠️ 전처리 & EDA 하이라이트
1. **결측치 삭제/치환**   
   - 매출·관객수 완전 누락 행 제거  
   - 범주형 결측 → `"기타"` 로 치환  
2. **범주형 인코딩** (Label Encoding > One-Hot > Target)  
3. **수치형 스케일링** (StandardScaler > MinMax)  
4. **텍스트 처리**  
   - 명사추출(`cleaned_nouns`), KeyBERT 키워드, 원문 비교 → **원문 시놉시스가 가장 우수**  
   - KoBERT vs Sentence-Transformer 임베딩 → **Sentence-Transformer + PCA(3D)** 승  
5. **차원 축소** (PCA vs LLE) → **PCA** 선택

---

## 🤖 모델링

| 문제 | 모델 | 핵심 설정 | Best Metric |
|------|------|-----------|-------------|
| **Classification**<br>관객수 등급(100 만·200 만 단위) | XGBoost / LogisticRegression / KNN | 언더·오버샘플링, Soft NMS | **Accuracy 1.00** *(XGB / 200 만 등급)* |
| **Regression**<br>관객수 예측 | XGBoostRegressor / RandomForest | KoBERT & ST 임베딩, GridSearch | **MAE ↓, R² ↑  (ST + PCA 3D + RF)** |

> 📊 **SHAP Value**로 변수 중요도 해석 & 피처 셀렉션 수행

---

## 🚀 사용 방법

```bash
# 1) 설치
git clone https://github.com/<YOUR_ID>/movie-import-success-pred.git
cd movie-import-success-pred
pip install -r requirements.txt

# 2) 학습 (회귀)
python src/train_regression.py --config configs/regression.yaml

# 3) 예측
python src/infer.py --title "Oppenheimer" --release_date 2023-08-15 --distribution "UPI Korea" ...
```

---

## 📂 폴더 구조

```
movie-import-success-pred/
├── README.md
├── requirements.txt
├── configs/
│   ├── classification.yaml
│   └── regression.yaml
├── data/
│   ├── raw/               # 원본 CSV & 외부 데이터
│   ├── processed/         # 전처리 결과
│   └── README.md          # 데이터 설명
├── notebooks/             # EDA & 실험 노트북
├── src/
│   ├── preprocessing.py
│   ├── feature_engineering.py
│   ├── train_classification.py
│   ├── train_regression.py
│   ├── infer.py
│   └── utils.py
├── assets/                # 결과 그래프·모델 비교 이미지
└── LICENSE
```

---

## 🖼️ 결과 예시

| 실제 vs 예측 (Random 40) | 관객수 구간별 예측 오차 |
|--------------------------|--------------------------|
| ![rand40](assets/prediction_40.png) | ![segments](assets/prediction_segments.png) |

---

## 🗝️ 주요 인사이트
- **휴일 플래그** 삽입 → 최대 **+7 % MAE 개선**  
- **감독/배우 과거 실적**을 로그 스케일링 후 투입 → 모델 안정성 ↑  
- 키워드 추출보다는 **시놉시스 전체 임베딩**이 문맥 보존에 유리

---

## ⚖️ 라이선스
MIT License – 자유롭게 사용하되, 출처 표기를 부탁드립니다.

---

## 👥 기여
| 역할 | 담당 |
|------|------|
| 데이터 수집 & 전처리 | **용준** |
| 모델링 & 실험 | 용준 |
| 결과 해석 & 시각화 | 용준 |
| 발표 | 용준 |

---

> **“데이터로 영화 흥행 가능성을 읽다” – 투자・배급 의사결정을 위한 실전 AI 툴킷**
