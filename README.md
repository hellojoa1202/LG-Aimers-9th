# LG Aimers 9기 — 스포츠 해커톤 제구 성공 확률 예측

<sub>LG Aimers 9기 Phase 2 온라인 해커톤에서 수행한 투구 단위 제구 성공 확률 예측 프로젝트 아카이빙</sub>

| 항목 | 내용 |
|---|---|
| 기간 | 2026.08.03 ~ 2026.09.02 |
| 주관 | LG AI Research |
| 과정 | LG Aimers 9기 Phase 2 온라인 해커톤 |
| 문제 | 투구 단위 `control_success` 확률 예측 |
| 평가 지표 | Brier Skill Score |
| 최종 결과 | Public BSS 703.94 → 1150.38, 최종 86위 |

## 프로젝트

투구 직전까지 확인 가능한 투수·타자·경기 상황·구종 이력과 과거 Trackman 정보를 활용해 각 투구의 제구 성공 확률을 예측했다. 단순한 이진 분류가 아니라 확률의 보정 품질을 평가하는 Brier Skill Score를 기준으로 모델과 실험을 비교했다.

## 핵심 결과

| 모델 | Public BSS | 역할 |
|---|---:|---|
| Adaptive Gate | 1088.5349 | 계층 baseline 위 상황별 residual gate |
| Seed Ensemble + Shift | 1119.2195 | 6-seed 평균 및 전역 calibration |
| F Expert | 1122.2577 | Futures 행 전용 residual 전문가 |
| F Regime | 1126.4544 | F residual 3채널·실패 유형·transition 결합 |
| **F Regime 0.75** | **최종 선택본** | F 보정 강도 0.75 적용 제출 패키지 |

- 전체 실험 기록: `archive/EXPERIMENTS.md`, `archive/experiment_log.csv`

## 모델 설계

- **Hierarchical base**: 투수 기본 제구 확률 추정
- **CatBoost residual model**: 경기 상황과 최근 흐름에 따른 잔차 학습
- **Failure expert**: 제구 실패 유형별 보정
- **R/F league expert**: 리그·시즌 분포 차이와 미래 시즌 변화 반영
- **Adaptive gate**: 기본 확률과 residual 보정 비중 조절
- **Seed ensemble·calibration·Tensor-EB**: 예측 안정성과 소규모 그룹 통계 보완

## 사용 정보 및 피처

- 경기 상황: 볼카운트, 점수, 주자, 아웃, 이닝, leverage index
- 선수 및 이력: 투수·타자 ID, 투구·타석 이력
- 누적 및 최근 정보: 누적 성적, 최근 성적, pitch mix, season delta
- Trackman: 과거 데이터와 Trackman reliability
- 파생 피처: `count_pressure`, `recent_form`, `recent_slope`, `pitchmix_entropy`

## 저장소 구성

```text
├── final/                 # 최종 제출 패키지 및 추론 소스
├── src/                   # 전처리·feature·gate·보조 target
├── evaluation/            # local 평가·LB 환산·forward OOF
├── archive/experiments/   # 전체 실험·학습 코드
├── docs/                  # 모델 구조·재현·데이터 설명
└── requirements.txt
```

## 재현 조건

- Python 3.11
- 필요 파일: `train.csv`, `test.csv`, `trackman_history.csv`, `sample_submission.csv`
- 데이터 경로: `data/`
- 최종 추론 패키지: `final/260818_F_regime075.zip`
- 상세 문서: [docs/REPRODUCE.md](docs/REPRODUCE.md)

대회 원본 데이터와 개인 경로, 외부 학습 자산은 저장소에 포함하지 않는다.

## 검증 원칙

- 2019~`t-1` 데이터로 학습하고 `t` 연도를 예측
- 검증 연도: 2022·2023·2024
- random split 기반 후보 배제
- Test 다른 행·Test 전체 분포·Test 내부 rolling/누적 집계 미사용
- Train snapshot·통계·보조 target만 추론 자산으로 사용
- 행 단독·부분집합·shuffle 입력 일관성 검사

## 자료

실험 기록, 모델 구조, 재현 방법과 최종 추론 코드를 중심으로 정리했으며, 원본 대회 데이터와 제출에 사용할 수 없는 외부 파일은 별도로 보관한다.
