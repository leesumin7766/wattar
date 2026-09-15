agent-README.md - Wattar LLM Executor Guide (v2 - Brightics 1.3 포함)
이 파일은 Antigravity IDE의 Gemini(Executor), Codex(Reviewer), Ollama(Local Checker)가 읽는 기계용 가이드다. 사람은 README.md를 본다.
2026-09-14 16:23 환경 기준, Brightics Studio v1.3 설치 완료 버전

0. Environment Lock - 반드시 지킬 것
Python: 3.10.11 ONLY - C:\Users\tta\AppData\Local\Programs\Python\Python310\python.exe
Launcher: py -3.10로 실행. python 명령어 금지 (다른 버전 꼬임)
venv: C:\Users\tta\wattar-code\.venv\Scripts\Activate.ps1
Node: fnm default v22.23.2 (npm 11.19.0)
Brightics Studio: v1.3 - C:\brightics-studio\start-brightics.cmd -> http://127.0.0.1:3000
다운로드: https://github.com/brightics/studio/releases -> BrighticsStudio-1.3-windows.exe
설치 경로 한글 금지 (Tokenizer 오류)
Brightics Studio는 Brightics AI의 오픈소스 버전으로 2018년 무료 공개됨. v1.3이 최신 OSS.
원칙: 원본/은 G드라이브 심볼릭 링크 - 절대 수정 금지 (읽기 전용). 결과물은 전부 정제/에 저장.
powershell
# PowerShell - wattar-code 루트에서 환경 캡처 (Repo Structure 7번용)
$env:PATH = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
"=== Python ===" > env_versions.txt
py -3.10 --version >> env_versions.txt
pip list >> env_versions.txt
"=== Node / NPM ===" >> env_versions.txt
node --version >> env_versions.txt
npm --version >> env_versions.txt
fnm --version >> env_versions.txt
"=== Git ===" >> env_versions.txt
git --version >> env_versions.txt
"=== LLM Tools ===" >> env_versions.txt
codex --version >> env_versions.txt
ollama list >> env_versions.txt
"=== Brightics ===" >> env_versions.txt
echo "Brightics Studio v1.3 at C:\brightics-studio - http://127.0.0.1:3000" >> env_versions.txt
cat env_versions.txt
1. 프로젝트 핵심 정의
목표식: Q_total = Q_주민 + Q_관광 + Q_산업 + Q_누수
목표: 총량이 아닌 원인 분리 예측. K-water AI(총량), 상수iN2.5(시설)의 한계 극복
분석 환경: Big-데이터 웨이브 Brightics AI에서 08~12번 상관분석 후 결과 반출, 로컬 Python(.venv 3.10.11)에서 모델링
Tech Stack: Python 3.10.11, Prophet, XGBoost, Isolation Forest, SHAP, QGIS, Brightics Studio v1.3, JupyterLab 4.6.3
2. 에이전트 역할 정의
역할	모델	담당	호출 방법
Executor	GEMINI PRO (Antigravity) 0.59.0	코드 생성, API 수집, 전처리 실행	Ctrl+L -> "@codebase agent-README.md Step1 수행"
Reviewer	CODEX (GPT PLUS) 0.153.4	Validation 체크리스트 검수, 실패 시 수정 코드 제안	codex exec "agent-README.md Step1 기준으로 정제/ 검토"
Local Checker	OLLAMA llama3.1:8b 4.9GB	민감 데이터 요약, 로컬 문법 체크	ollama run llama3.1:8b "파일 요약"
Visualizer	JupyterLab 4.6.3	각 Step 시각화 필수	jupyter notebook notebooks/
3. 데이터 인벤토리 (README 4번 - 12개)
ID	데이터명	현재 위치 (원본/)	상태	Brightics 사용
01	배수지별 유량적산차	공공데이터포털/	확인 필요	Load CSV
02	광역취수장 제원	공공데이터포털/수자원_..._API	가이드만 있음	-
03	누수신고 민원	없음	❌ 수집예정	라벨
04	기상청 ASOS	기상청/OBS_ASOS_TIM_2021~2025 10개	✅ 있음	Load CSV
05	낙동강 수질	없음	❌ 수집예정	-
06	관광 숙박/방문객	없음	❌ TourAPI 필요	-
07	구별 인구	행안부_인구/ 중복 2개	✅ 있음 (합쳐야함)	Load CSV
08	블록별 유량/수압/수질	빅데이터웨이브/	확인 필요	Load CSV
09	쇼미The부산	빅데이터웨이브/ 내부 예상	확인 필요	Load CSV
10	격자별 유동인구 50m	데이터오픈랩 - 안심구역	⚠️ 집계만 반출	상관분석 핵심
11	행정동별 생활인구	데이터오픈랩	⚠️ API->CSV 필요	상관분석 핵심
12	관광업종 카드매출	데이터오픈랩 BC카드	⚠️ API->CSV 필요	상관분석 핵심
4. 파이프라인 - LLM이 단계별로 수행
Step 0: Raw Inventory
Objective: 원본/ 폴더 전체 스캔
Executor: "원본/을 rglob로 스캔해서 src/data/inventory.json 생성"
Output: src/data/inventory.json, notebooks/00_inventory.ipynb
Validation: 파일 개수 >=25, 빈 폴더 0개
Step 1: API to CSV (가장 중요)
Objective: 11,12번 + 06 TourAPI + 03 민원 데이터를 로컬 csv로 고정
Input: 원본/공공데이터포털/수자원_.../Colab_API_to_CSV_가이드, 원본/행안부_.../Colab_API_to_CSV_가이드
Output: 정제/생활인구_행정동별_일별.csv, 정제/카드매출_관광업종_월별.csv, 정제/수자원_공급량_일별.csv
Executor Prompt: "Colab 가이드를 로컬 Python 3.10용으로 변환, .env에서 API키 읽어서 정제/에 저장"
Validation (Reviewer):
 csv 0KB 아닌가?
 날짜 YYYYMMDD 형식인가?
 행정동코드 10자리 유지?
Visualization: notebooks/01_api_to_csv.ipynb 일자별 row수 추이
Step 2: 키 통일 / 정제
Input: 정제/.csv + 원본/기상청/.csv + 원본/행안부_인구/*.xlsx
Output: 정제/기상청_일별_부산.csv, 정제/인구_월별_행정동.csv
Executor: "xlsx->csv, 기상청 10개 concat, 결측치 -999 처리"
행안부_인구 중복 폴더 합치기
Step 3: Brightics AI (상관분석 - 붙임4 증빙용) - v1.3 기준
툴: C:\brightics-studio\start-brightics.cmd -> http://127.0.0.1:3000
핵심 산출물은 Brightics 워크플로우 .json이며, 실행하려면 Brightics Studio에 해당 JSON을 import하고 dataset/의 csv를 업로드해 연결한다.
워크플로우:
Data -> Load CSV: 정제/기상청_일별 + 정제/인구_월별 + 정제/생활인구 (오픈랩 집계)
Statistics -> Correlation: 유량 vs 유동인구 vs 기온
Modeling -> K-Means Clustering: 블록 유형 군집
Result -> Report 생성
산출물: docs/brightics/01_유량_vs_유동인구_상관.png, brightics_workflow.json
주의: 경로 한글 금지, 오픈랩 원본 반출 금지 (집계만)
Step 4: Q_주민 (Baseline) - Prophet
Input: 정제/인구 + 정제/기상
Output: 정제/Q_주민_baseline.csv
Step 5: Q_관광 (Tourism) - XGBoost + SHAP
Input: 10,11,12번 집계 + 06 TourAPI + Q_주민
Output: 정제/Q_관광.csv, notebooks/04_tourism_shap.png
핵심: "해운대 12시 수요↑ = 관광 68%"
Step 6: Q_누수 (Leak) - Isolation Forest
Input: 01 배수지 유량 + 08 블록 유량 + 10 유동인구 야간
Output: 정제/Q_누수_탐지.csv
Validation: 03 누수신고 대비 Precision >=0.82
Step 7: 최종 분해 및 대시보드
Q_total = Q_주민+Q_관광+Q_산업+Q_누수 스택 그래프
Output: notebooks/07_final_dashboard.ipynb, docs/부산_3D_지도_Q4분리.pdf
Validation:
 4개 Q 합이 Q_total과 ±5% 이내?
 Brightics 캡처 08~12번 포함?
 1장 PDF에 SHAP, 누수, 정책(전력비 5억) 포함?
5. 실행 규칙
원본/ 읽기 전용. 쓰기는 정제/와 src/data/에만
매 Step 끝에는 notebooks/에 그래프 1개 이상 저장
Reviewer LGTM 전까지 다음 Step 이동 금지
오픈랩 10,11,12 원본 절대 깃 커밋 금지 (.gitignore에 오픈랩원본* 추가)
모든 csv는 utf-8-sig, index=False
Brightics 설치 경로 한글 금지
6. 명령어 예시
powershell
# Step1 실행 (Executor)
# Antigravity Agent 패널: 
@codebase agent-README.md Step1 API to CSV를 Python 3.10으로 수행해줘. 입력은 원본/공공데이터포털/수자원_...가이드 참고

# Step1 검토 (Reviewer)
codex exec "agent-README.md Step1 Validation 기준으로 정제/생활인구_*.csv 검토하고 문제 있으면 수정 코드 제안해줘"

# Brightics 실행
cd C:\brightics-studio; .\start-brightics.cmd

# 시각화 확인
cd C:\Users\tta\wattar-code; .\.venv\Scripts\Activate.ps1; jupyter notebook notebooks/01_api_to_csv.ipynb

# Ollama 로컬 체크
ollama run llama3.1:8b "src/data/01_kwater_api_to_csv.py 문법 체크"
7. 산출물
env_versions.txt
정제/*.csv (01~12 ID 통일)
notebooks/00~07_*.ipynb
docs/brightics/ (Brightics 캡처 + brightics_workflow.json)
docs/부산_3D_지도_Q4분리.pdf (최종 1장)
