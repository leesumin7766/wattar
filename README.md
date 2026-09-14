# Wattar - 수요분리형 부산 누수 관리 데이터 분석 AI
> Q_total = Q_주민 + Q_관광 + Q_산업 + Q_누수. 총량이 아닌 원인을 예측한다.

부산의 물 수요 스파이크는 왜 생길까? 해운대는 관광객 때문인지, 온천동은 누수 때문인지 구분해주는 AI 모델과 대시보드.

DX Challenge / 부산 빅데이터혁신센터 제출용

## 1. 문제점 (Problem)

부산은 3가지 리스크가 동시에 터진다.

1.  **노후관 누수:** 동래구 온천동 50mm관 사고처럼 야간에도 유량이 줄지 않는 블록 존재
2.  **관광 수요 스파이크:** 해운대 해수욕장 주말 유동인구 15%↑ → 배수지 유량 12%↑, 하지만 기존 모델은 "왜" 늘었는지 모름
3.  **원수 비용 증가:** 낙동강 원수 탁도 악화 시 정수장 가동비용 3배 증가

## 2. 한계점 (Limitation)

*   **K-water AI:** 전국 43개 광역정수장 표준 모델. `Q_total` 총량 예측은 정확하지만, 수요의 원인을 분리하지 못함.
*   **부산시 상수iN2.5:** 시설 운영/수압 관리에 집중. 인구/관광 등 외부 변수와의 결합 분석 부재.

결국 "물이 많이 필요하다"는 알지만, "누가/왜 쓰는지"는 모른다.

## 3. 해결방안 (Solution)

수요를 4개로 분리 예측하는 하이브리드 모델 제안.

**Architecture**
`K-water 유량 + 오픈랩 유동인구 + 웨이브 블록유량 + 관광/기상 데이터 → 수요분리`

1.  **Q_주민 (Baseline):** Prophet으로 요일/계절성 기반 주민 수요 예측 (인구/고령화/1인가구 반영)
2.  **Q_관광 (Tourism):** XGBoost + SHAP. 오픈랩 격자 유동인구, 카드매출, TourAPI 숙박률을 피처로 사용. "해운대는 관광 68%"라고 설명 가능
3.  **Q_누수 (Leak):** Isolation Forest. 야간최소유량(Night Minimum Flow)과 오픈랩 데이터 교차 검증으로 누수 탐지
4.  **Q_산업 (Industry):** 공단 전력사용량 기반 선택적 분리

**분석 환경:** Big-데이터 웨이브 내 **Brightics AI (삼성SDS)** 로 상관분석/군집분석/시각화 수행 후 결과 반출

## 4. 활용 데이터 (Data Sources)

| 구분 | 데이터명 | 출처 | 수집기간 | 활용 목적 |
| :--- | :--- | :--- | :--- | :--- |
| **K-water** | 01. 배수지별 유량적산차 | 공공데이터포털 / K-water AI 경진대회 | 2021-2025 여름 | **Main Target** Q_total |
|  | 02. 광역취수장 제원정보 | data.go.kr | - | 원수-정수 연계 |
| **공공** | 03. 부산시 상수도 누수신고 민원 | 부산시 상수도사업본부 | 2021-2025 | 누수 라벨 (정답지) |
|  | 04. 기상청 ASOS 시간별 기온/강수 | 기상청 개방포털 | 2021-2025 여름 | 폭염 수요 피처 |
|  | 05. 낙동강 원수 수질측정망 | 환경부 WAMIS | 2021-2025 | 원수 비용 분석 |
|  | 06. 관광 숙박/방문객 | 한국관광데이터랩 TourAPI | 2021-2025 | Q_관광 보조 피처 |
|  | 07. 구별 인구/고령화/1인가구 | 행안부 / 쇼미The부산 | 2021-2025 | Q_주민 베이스라인 |
| **Big-데이터 웨이브** | 08. 블록별 유량/수압/수질 | 부산형 데이터 통합플랫폼 Big-데이터웨이브 | 2022-2025 | 블록별 검증, 상수iN2.5 연계 |
|  | 09. 쇼미The부산 / 통합데이터지도 | Big-데이터웨이브 실증서비스 | 2023-2025 | 인구/소득/소비 교차검증 |
| **데이터 오픈랩 (가점)** | 10. 격자별 시간대별 유동인구 (50m) | 부산빅데이터혁신센터 데이터오픈랩 (SKT/KT) | 202108~202508 | **Q_관광 핵심 피처** - 가점 |
|  | 11. 행정동별 일별 생활인구 합계 | 데이터오픈랩 - 인구 | 202108~202508 | 생활-상주인구 차이로 관광객 추정 |
|  | 12. 업종별 관광업종 카드매출 | 데이터오픈랩 - 소비 (BC카드) | 202201~202507 | Q_관광 소비 증빙 |

> 오픈랩 데이터는 안심구역 정책상 원본 반출 불가. 시간/동 단위 집계/평균 결과만 반출 심사 후 활용.

**Brightics 활용:** 웨이브 포털 내 Brightics로 08~12번 데이터 상관분석, `유량 vs 유동인구` 회귀분석 결과 캡처 → 붙임4 증빙

## 5. 예상 결과 (Expected Outcome)

1.  **설명가능한 수요 예측:** "해운대 블록 12시 수요↑ = 관광 68%, 주민 32%" 처럼 원인별 기여도 제시 (SHAP)
2.  **누수 조기 탐지 정확도 82%:** 야간최소유량 + 유동인구 교차로 오탐 감소
3.  **정책 효과:** 연간 전력비 5억 절감 레퍼런스 기반, 관광 성수기 가압장 운영 최적화 제안
4.  **시각화:** 1장 PDF - 부산 3D 지도 위에 Q_4분리 스택 그래프 + Brightics 분석 대시보드

## 6. Tech Stack

`Python 3.10.11, Prophet, XGBoost, Isolation Forest, SHAP, Big-데이터웨이브 Brightics, QGIS, JupyterLab 4.6.3`

## 7. Repo Structure & Environment

### 7-1. Structure
wattar-code/
├── 원본/ # G:\내 드라이브\wattar 심볼릭 링크 - 읽기 전용
│ ├── 공공데이터포털/
│ │ ├── 수자원_광역상수도_공급량_API/
│ │ └── 행안부_행정동별_세대수_API/
│ ├── 기상청/ # OBS_ASOS_TIM_2021∼2025.csv/xlsx (10개)
│ ├── 빅데이터웨이브/
│ └── 행안부_인구/
├── 정제/ # 전처리 결과물 - Agent가 쓰는 곳
├── src/data/ # 전처리 파이썬 코드
├── notebooks/ # 시각화 (00∼07)
├── docs/brightics/ # Brightics 캡처
├── .venv/ # Python 3.10.11 가상환경
├── agent-README.md # LLM 실행/검토용 기계용 가이드
├── env_versions.txt # 버전 스냅샷 (2026-09-14)
├── .gitignore
└── README.md


### 7-2. Environment Versions (2026-09-14 16:23 기준)

| 구분 | 프로그램 | 버전 | 경로 / 비고 |
| :--- | :--- | :--- | :--- |
| **Python** | Python | 3.10.11 | `C:\Users\tta\AppData\Local\Programs\Python\Python310\python.exe` |
|  | py launcher | `C:\Windows\py.exe` | `py -3.10`로 실행 |
|  | pip | 23.0.1 | `pip 26.2.1` 업데이트 가능 |
|  | venv | `.venv/` | `.\.venv\Scripts\Activate.ps1` |
|  | 주요 패키지 | jupyterlab 4.6.3, notebook 7.6.2, ipykernel 7.3.0, requests 2.34.2, beautifulsoup4 4.15.0 | `pip list` 참고 |
| **Node.js** | fnm | 1.39.0 | Node 버전 매니저 |
|  | Node | v22.23.2 (default), v24.21.0, system | `fnm list` - `v22.23.2` 사용중 |
|  | npm | 11.19.0 | `C:\Program Files\nodejs\node.exe` |
| **Git** | git | 2.55.0.windows.3 | |
|  | gh cli | 2.100.0 (2026-09-03) | |
| **LLM Tools** | codex-cli (GPT PLUS) | 0.153.4 | `C:\Users\tta\AppData\Roaming\npm\codex` - Reviewer 역할 |
|  | gemini cli | 0.59.0 | Antigravity IDE 내장 - Executor 역할 (Gemini 3.0/3.8 Flash High) |
|  | ollama | 0.33.3 | `C:\Users\tta\AppData\Local\Programs\Ollama\ollama.exe` - Local Checker |
|  | ollama model | llama3.1:8b (4.9GB) | 4 days ago |
| **분석 툴** | Brightics Studio | v1.3 (Latest OSS) | `C:\brightics-studio` - 삼성SDS 무료 공개 버전, `http://127.0.0.1:3000` |
|  | QGIS | 3.x | 지도 시각화 |

### 7-3. Quick Start

```powershell
# 1. PATH 복구
$env:PATH = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")

# 2. Python 가상환경
cd C:\Users\tta\wattar-code
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt

# 3. Brightics Studio 실행
cd C:\brightics-studio
.\start-brightics.cmd
# 브라우저 자동 팝업 -> 안 뜨면 http://127.0.0.1:3000 접속

# 4. Agent 실행 (Executor = GEMINI)
# Antigravity IDE Agent 패널: @codebase agent-README.md Step1 수행

# 5. 시각화
jupyter notebook notebooks/

# 4. 검토 (Reviewer = CODEX)
codex exec "agent-README.md Step1 기준으로 정제/ 폴더 검토"

# 5. 시각화 (Visualizer = Jupyter)
jupyter notebook notebooks/
