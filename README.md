# <sub>LG Aimers 9기 — 스포츠 해커톤 제구 성공 확률 예측</sub>

<p><sub>LG Aimers 9기 Phase 2 온라인 해커톤에서 수행한 투구 단위 제구 성공 확률 예측 프로젝트 아카이빙</sub></p>

<table>
  <tr>
    <th><sub>항목</sub></th>
    <th><sub>내용</sub></th>
  </tr>
  <tr><td><sub>기간</sub></td><td><sub>2026.08.03 ~ 2026.09.02</sub></td></tr>
  <tr><td><sub>주관</sub></td><td><sub>LG AI Research</sub></td></tr>
  <tr><td><sub>과정</sub></td><td><sub>LG Aimers 9기 Phase 2 온라인 해커톤</sub></td></tr>
  <tr><td><sub>문제</sub></td><td><sub>투구 단위 <code>control_success</code> 확률 예측</sub></td></tr>
  <tr><td><sub>평가 지표</sub></td><td><sub>Brier Skill Score</sub></td></tr>
  <tr><td><sub>최종 결과</sub></td><td><sub>Public BSS 703.94 → 1150.38, 최종 86위</sub></td></tr>
</table>

## <sub>프로젝트</sub>

<p><sub>투구 직전까지 확인 가능한 투수·타자·경기 상황·구종 이력과 과거 Trackman 정보를 활용해 각 투구의 제구 성공 확률을 예측했다. 단순한 이진 분류가 아니라 확률의 보정 품질을 평가하는 Brier Skill Score를 기준으로 모델과 실험을 비교했다.</sub></p>

## <sub>핵심 결과</sub>

<table>
  <tr>
    <th><sub>모델</sub></th>
    <th><sub>Public BSS</sub></th>
    <th><sub>역할</sub></th>
  </tr>
  <tr><td><sub>Adaptive Gate</sub></td><td><sub>1088.5349</sub></td><td><sub>계층 baseline 위 상황별 residual gate</sub></td></tr>
  <tr><td><sub>Seed Ensemble + Shift</sub></td><td><sub>1119.2195</sub></td><td><sub>6-seed 평균 및 전역 calibration</sub></td></tr>
  <tr><td><sub>F Expert</sub></td><td><sub>1122.2577</sub></td><td><sub>Futures 행 전용 residual 전문가</sub></td></tr>
  <tr><td><sub>F Regime</sub></td><td><sub>1126.4544</sub></td><td><sub>F residual 3채널·실패 유형·transition 결합</sub></td></tr>
  <tr><td><strong><sub>F Regime 0.75</sub></strong></td><td><strong><sub>최종 선택본</sub></strong></td><td><sub>F 보정 강도 0.75 적용 제출 패키지</sub></td></tr>
</table>

<p><sub>전체 실험 기록: <code>archive/EXPERIMENTS.md</code>, <code>archive/experiment_log.csv</code></sub></p>

## <sub>모델 설계</sub>

<ul>
  <li><sub><strong>Hierarchical base</strong>: 투수 기본 제구 확률 추정</sub></li>
  <li><sub><strong>CatBoost residual model</strong>: 경기 상황과 최근 흐름에 따른 잔차 학습</sub></li>
  <li><sub><strong>Failure expert</strong>: 제구 실패 유형별 보정</sub></li>
  <li><sub><strong>R/F league expert</strong>: 리그·시즌 분포 차이와 미래 시즌 변화 반영</sub></li>
  <li><sub><strong>Adaptive gate</strong>: 기본 확률과 residual 보정 비중 조절</sub></li>
  <li><sub><strong>Seed ensemble·calibration·Tensor-EB</strong>: 예측 안정성과 소규모 그룹 통계 보완</sub></li>
</ul>

## <sub>사용 정보 및 피처</sub>

<ul>
  <li><sub>경기 상황: 볼카운트, 점수, 주자, 아웃, 이닝, leverage index</sub></li>
  <li><sub>선수 및 이력: 투수·타자 ID, 투구·타석 이력</sub></li>
  <li><sub>누적 및 최근 정보: 누적 성적, 최근 성적, pitch mix, season delta</sub></li>
  <li><sub>Trackman: 과거 데이터와 Trackman reliability</sub></li>
  <li><sub>파생 피처: <code>count_pressure</code>, <code>recent_form</code>, <code>recent_slope</code>, <code>pitchmix_entropy</code></sub></li>
</ul>

## <sub>저장소 구성</sub>

<pre><sub>├── final/                 # 최종 제출 패키지 및 추론 소스
├── src/                   # 전처리·feature·gate·보조 target
├── evaluation/            # local 평가·LB 환산·forward OOF
├── archive/experiments/   # 전체 실험·학습 코드
├── docs/                  # 모델 구조·재현·데이터 설명
└── requirements.txt</sub></pre>

## <sub>재현 조건</sub>

<ul>
  <li><sub>Python 3.11</sub></li>
  <li><sub>필요 파일: <code>train.csv</code>, <code>test.csv</code>, <code>trackman_history.csv</code>, <code>sample_submission.csv</code></sub></li>
  <li><sub>데이터 경로: <code>data/</code></sub></li>
  <li><sub>최종 추론 패키지: <code>final/260818_F_regime075.zip</code></sub></li>
  <li><sub>상세 문서: <a href="docs/REPRODUCE.md">docs/REPRODUCE.md</a></sub></li>
</ul>

<p><sub>대회 원본 데이터와 개인 경로, 외부 학습 자산은 저장소에 포함하지 않는다.</sub></p>

## <sub>검증 원칙</sub>

<ul>
  <li><sub>2019~<code>t-1</code> 데이터로 학습하고 <code>t</code> 연도를 예측</sub></li>
  <li><sub>검증 연도: 2022·2023·2024</sub></li>
  <li><sub>random split 기반 후보 배제</sub></li>
  <li><sub>Test 다른 행·Test 전체 분포·Test 내부 rolling/누적 집계 미사용</sub></li>
  <li><sub>Train snapshot·통계·보조 target만 추론 자산으로 사용</sub></li>
  <li><sub>행 단독·부분집합·shuffle 입력 일관성 검사</sub></li>
</ul>

## <sub>자료</sub>

<p><sub>실험 기록, 모델 구조, 재현 방법과 최종 추론 코드를 중심으로 정리했으며, 원본 대회 데이터와 제출에 사용할 수 없는 외부 파일은 별도로 보관한다.</sub></p>
