
> **오늘의 핵심:** 10일차에는 Raspberry Pi의 Edge AI를 `systemd`가 자동으로 실행하고, `Runtime State`와 `FastAPI`로 상태를 확인하며, 장애가 발생하면 다시 시작할 수 있는 **운영형 Edge 시스템**으로 만들었습니다.
>
> 11일차에는 새로운 기능을 계속 추가하는 것이 핵심이 아닙니다. 지금까지 만든 시스템을 **작은 기능 → 연결 기능 → 전체 시스템** 순서로 검증하고, 실패 조건을 모으고, 한 번에 한 변수만 바꾸어 결과를 비교한 뒤 **왜 이 설정을 선택했는지 근거를 남기는 것**이 핵심입니다.
>
> 1~10일차에 사용한 `subject13_edge_ai` 프로젝트와 기존 `.venv`를 그대로 이어서 사용합니다. **새 프로젝트와 새 가상환경을 만들지 않습니다.**

---

## 오늘 가장 중요한 질문

10일차까지 다음 구조를 만들었습니다.

```text
Raspberry Pi Boot
        ↓
systemd
        ↓
subject13-edge.service
        ↓
Edge Runtime
        ↓
Camera
→ Preprocess
→ ONNX AI
→ Operational Decision
→ Frame Skip
→ Voting / Debounce / Hold
→ GPIO
        ├─→ Operational AI Event Log
        │    → 첫 운영 State + 이후 최종 State 변화 기록
        └─→ runtime/edge_state.json
        ↓
subject13-api.service
        ↓
FastAPI
        ↓
/health /status /metrics
        ↓
PC
```

오늘은 이 시스템을 보면서 세 가지 질문에 실제 결과로 답합니다.

```text
1. 기능은 정상적으로 동작하는가?

2. 어떤 조건에서 실패하는가?

3. Accuracy · False Warning · Missed Warning ·
   Latency · FPS · Stability 사이에서
   무엇을 우선할 것인가?
```

따라서 11일차의 핵심 질문은 다음입니다.

> **“지금까지 만든 Edge AI 시스템을 Test와 실험으로 검증하고, 실패 원인을 구분하며, 운영 설정을 선택한 근거를 재현 가능한 기록으로 남길 수 있는가?”**

---

## 10일차와 11일차는 어디가 이어질까요?

10일차 마지막에 다음 상태가 정상이어야 했습니다.

```text
subject13-edge.service
→ enabled + active

subject13-api.service
→ enabled + active

runtime/edge_state.json
→ 계속 갱신

/health
→ api: OK
→ edge: OK
→ stale: false

Camera
→ AI
→ Stabilization
→ GPIO
→ Operational AI Event Log 새 Row 기록
→ 정상 동작
```

11일차에서는 이 상태를 **운영 Baseline**으로 사용합니다.

```text
10일차 운영 Baseline
        ↓
Unit Test
        ↓
Integration Test
        ↓
System Test
        ↓
Controlled Experiment
        ↓
Failure Case 수집
        ↓
Root Cause 분류
        ↓
Trade-off 판단
```

오늘 처음부터 Camera나 Model을 다시 만드는 것이 아닙니다.

`src/decision.py`, `src/stabilizer.py`, `src/runtime_state.py`, `src/event_logger.py`, `src/ai_preprocess.py`, `src/inference_onnx.py`, `scripts/edge_runtime.py`, `scripts/status_api.py`, `scripts/optimized_ai_warning_system.py` 등은 **이전 일차에서 만든 파일을 다시 사용**합니다.

특히 `src/event_logger.py`의 `AIEventLogger`는 7일차에서 만든 Class를 10일차 운영 Runtime이 다시 사용한 것이므로, 11일차에서는 새 Logger를 만들지 않고 **실제 운영 Runtime이 이 Logger를 정상적으로 호출하는지 Test**합니다.

---

## 11일차 결과는 12일차에 어떻게 연결될까요?

12일차에는 Raspberry Pi에서 완전히 다른 프로젝트를 새로 시작하지 않습니다.

11일차가 끝나면 다음 상태가 준비됩니다.

```text
Raspberry Pi Edge Pipeline
        ↓
Unit Test PASS
        ↓
Integration Test PASS
        ↓
System Test PASS
        ↓
Failure Analysis 기록
        ↓
성능·정확성·안정성 Trade-off 기록
        ↓
11일차 Tested Baseline
```

12일차에서는 이 Baseline을 기준으로 Raspberry Pi와 Jetson의 차이를 비교합니다.

```text
11일차
Raspberry Pi
CPU 중심 Edge AI
        ↓
검증된 공통 구조 확인
        ↓
12일차
Jetson Orin Nano Super
GPU 가속 Edge AI
```

12일차에서 특히 다시 보게 될 파일은 다음과 같습니다.

```text
장치가 바뀌어도 재사용하기 쉬운 부분
→ config_loader.py
→ decision.py
→ stabilizer.py
→ runtime_state.py
→ performance.py
→ Test 코드와 Report

장치에 따라 달라질 수 있는 부분
→ camera_input.py
→ inference Runtime
→ gpio_output.py
→ resource_monitor.py
```

즉, 오늘 Test와 Failure Analysis를 제대로 남겨 두어야 내일 **“무엇을 그대로 가져가고 무엇을 장치에 맞게 바꿔야 하는가?”**를 근거 있게 비교할 수 있습니다.

---

## 오늘의 학습 목표

수업이 끝나면 다음을 할 수 있어야 합니다.

```text
1. Unit / Integration / System Test의 차이를 설명한다.
2. Hardware 없이 검증할 수 있는 Logic과 Hardware가 필요한 기능을 구분한다.
3. pytest로 Decision / Stabilizer / Runtime State를 검증한다.
4. 경계값 정책을 Test로 고정하는 이유를 설명한다.
5. 저장 이미지를 이용하여 Preprocess → ONNX → Decision 연결을 확인한다.
6. FastAPI 응답 구조를 작은 Smoke Test 프로그램으로 확인한다.
7. 실제 Camera → AI → GPIO → Operational AI Event Log → systemd → FastAPI 전체 흐름을 System Test한다.
8. 10일차 운영 Runtime이 `ai_event_log_path`에 첫 운영 State와 이후 **최종 output_state 변화**를 실제로 기록하는지 증명한다.
9. 실험에서 한 번에 한 변수만 바꾸는 이유를 설명한다.
10. Validation Dataset과 Test Dataset의 역할을 구분한다.
11. Confidence Threshold의 False Warning / Missed Warning Trade-off를 비교한다.
12. Day 08의 96×96 / 64×64 ONNX Variant를 재사용하여 Accuracy와 Latency를 비교한다.
13. Hold 시간을 바꾸어 출력 안정성과 NORMAL 복귀 지연을 비교한다.
14. 수동 Camera 실험 전에 systemd Edge Service와 Camera 점유 관계를 확인한다.
15. 실패 이미지를 수집하고 Manifest와 분석 CSV로 연결한다.
16. Input / Data-Model / Runtime / Performance / Operation 문제를 구분한다.
17. 실행 결과가 어느 파일의 어느 코드에서 만들어졌는지 역추적한다.
18. Mini Challenge에서 예측 → 수정 → 오류 복구 → 증명까지 스스로 수행한다.
19. Challenge와 실험이 끝난 뒤 12일차용 정상 Baseline으로 복원한다.
```

---

## 오늘의 8시간 학습 흐름

| 학습 블록 | 권장 시간 | 무엇을 배우나요? | 끝나면 할 수 있어야 하는 것 |
|---|---:|---|---|
| 1 | 45분 | 10일차 Handoff · Test 전략 · Baseline 고정 | 무엇을 어떤 순서로 검증할지 설명할 수 있다 |
| 2 | 80분 | Unit Test · pytest | Decision·Stabilizer·Runtime State를 자동 검증할 수 있다 |
| 3 | 60분 | Integration Test · API Smoke Test | 저장 Image와 API 연결을 작은 범위에서 검증할 수 있다 |
| 4 | 55분 | System Test | Camera·AI·GPIO·Operational Event Log·systemd·FastAPI 전체 흐름을 확인할 수 있다 |
| 5 | 85분 | Controlled Experiment · Trade-off | 한 변수 실험과 Validation/Test 분리를 적용할 수 있다 |
| 6 | 65분 | Failure Case · Root Cause | 실패를 수집하고 원인 영역을 분류할 수 있다 |
| 7 | 60분 | Mini Challenge | 결과 역추적·예측·설정 변경·기능 수정·오류 복구를 스스로 수행할 수 있다 |
| 8 | 30분 | 복원 · Report · README · Git · 복습 | 12일차가 사용할 Tested Baseline을 남길 수 있다 |

총 480분을 기준으로 합니다. Raspberry Pi 재부팅, Camera 상태, 교육장 Network 속도에 따라 실제 소요시간은 조금 달라질 수 있습니다.

---

## 오늘도 계속 기억할 기본 Pipeline

1일차부터 사용한 큰 구조는 그대로입니다.

```text
입력
  ↓
처리
  ↓
판단
  ↓
출력
  ↓
기록
```

11일차에서는 여기에 **검증**이 추가됩니다.

```text
입력
→ Camera / 저장 Image

처리
→ Preprocess / ONNX

판단
→ Class + Confidence
→ Operational Decision
→ Stabilization

출력
→ GPIO

운영
→ systemd
→ FastAPI

기록
→ Operational AI Event Log
→ Runtime State
→ Journal
→ CSV / Report

검증
→ Unit Test
→ Integration Test
→ System Test
→ Failure Analysis
→ Trade-off
```

코드를 보다가 헷갈리면 다음 세 질문으로 돌아옵니다.

> **“지금 검증하는 범위는 한 함수인가, 여러 모듈의 연결인가, 전체 장치인가?”**  
> **“이 결과가 달라졌다면 방금 바꾼 변수 하나 때문이라고 말할 수 있는가?”**  
> **“이 값은 Validation에서 선택한 값인가, 마지막 Test에서 확인한 값인가?”**

---

## 오늘 사용할 실제 장비와 이전 일차 결과

```text
Raspberry Pi
→ Raspberry Pi OS / Linux
→ USB Camera
→ Green / Red LED
→ 선택 Buzzer

10일차 운영 구조
→ subject13-edge.service
→ subject13-api.service
→ src/event_logger.py의 AIEventLogger 재사용
→ configs/settings.yaml의 ai_event_log_path
→ 실제 운영 Event CSV
→ runtime/edge_state.json
→ /health
→ /status
→ /metrics

7~9일차 AI / 안정화
→ models/day07_tiny_cnn.onnx
→ models/day07_tiny_cnn.meta.json
→ src/ai_preprocess.py
→ src/inference_onnx.py
→ src/decision.py
→ src/stabilizer.py
→ scripts/optimized_ai_warning_system.py

8일차 성능 도구
→ scripts/benchmark_model_only.py
→ scripts/evaluate_onnx.py
→ models/day08_tiny_cnn_96.onnx
→ models/day08_tiny_cnn_96.meta.json
→ models/day08_tiny_cnn_64.onnx
→ models/day08_tiny_cnn_64.meta.json
```

오늘 새로 추가하는 핵심 파일:

```text
tests/
├── test_decision.py
├── test_stabilizer.py
└── test_runtime_state.py

scripts/
├── api_smoke_test.py
├── evaluate_operational_threshold.py
├── capture_failure_case.py
└── analyze_failure_cases.py

reports/
├── day11_test_analysis.md
└── day11_failure_analysis.csv

data/day11_failures/
├── images/
└── manifest.csv
```

`data/day11_failures/`는 실행 중 만들어지는 Failure Dataset입니다. 소스코드가 아니므로 기존 `.gitignore`의 `data/` 제외 규칙을 그대로 사용합니다.

---

## 오늘 배울 기술 스택

| 기술 | 오늘 어디에 사용하나요? |
|---|---|
| `pytest` | 작은 Python Logic 자동 검증 |
| `assert` | 실제 결과와 기대 결과 비교 |
| Fixed Image | Camera 없이 ONNX 연결 검증 |
| ONNX Runtime | 이전 일차 Model 추론 재사용 |
| `urllib.request` | 외부 Package 없이 HTTP API Smoke Test |
| `systemctl` | 두 Service 상태와 수동 중지·재시작 |
| `journalctl` | System Failure 이력 확인 |
| Validation Dataset | Threshold·Input Size 같은 선택에 사용 |
| Test Dataset | 선택이 끝난 뒤 마지막 확인에 사용 |
| CSV | Failure Manifest와 분석 결과 기록 |
| Git Tag / Branch | Baseline과 실험 작업 구분 |
| Report | 결과·가설·해석·선택 근거 기록 |

---

## 실습 전에 자신의 환경값 적어두기

실제 값은 장비와 교육장 Network에 따라 다릅니다.

```text
Raspberry Pi 사용자 이름     : ______________________________
Raspberry Pi Hostname        : ______________________________
Raspberry Pi IPv4            : ______________________________
Raspberry Pi Project Path    : ______________________________
Raspberry Pi Python Path     : ______________________________

Camera Index                 : ______________________________
Camera Resolution            : ______________________________
use_buzzer                   : true / false

API Port                     : ______________________________

현재 ai_confidence_threshold : ______________________________
현재 inference_every_n_frames: ______________________________
현재 vote_window             : ______________________________
현재 vote_warning_min        : ______________________________
현재 debounce_required       : ______________________________
현재 warning_hold_sec        : ______________________________

Day 08 96×96 Model           : 존재 / 없음
Day 08 64×64 Model           : 존재 / 없음
Validation Dataset           : 존재 / 없음
Test Dataset                 : 존재 / 없음
```

교재에서는 다음 표기를 사용합니다.

```text
<RPI_USER>
→ 자신의 Raspberry Pi 사용자 이름

<RPI_IP>
→ 현재 확인한 Raspberry Pi IPv4

<API_PORT>
→ configs/settings.yaml의 api_port
```

IP가 달라졌다면 예전 값을 반복해서 사용하지 않습니다.

Raspberry Pi에서:

```bash
hostname
hostname -I
```

API Port:

```bash
grep -E "^api_port:" configs/settings.yaml
```

Camera index에 문제가 있다면:

```bash
python -m scripts.camera_index_probe
```

---

## 오늘의 전체 실습 흐름

```text
10일차 두 Service 확인
        ↓
/health /status /metrics 확인
        ↓
현재 Git 상태 확인
        ↓
day10-operational-baseline Tag
        ↓
Day 11 Report 시작
        ↓

[Unit Test]

Decision
→ Stabilizer
→ Runtime State
→ 전체 pytest
→ 일부러 Test 실패
→ 원인 읽기
→ 복구

        ↓

[Integration Test]

Fixed Image
→ Preprocess
→ ONNX
→ Decision

PyTorch ↔ ONNX Parity 재확인

FastAPI
→ /health /status /metrics
→ Smoke Test

        ↓

[System Test]

Camera
→ Edge Runtime
→ ONNX
→ Stabilization
→ GPIO
→ Operational AI Event Log
→ Runtime State
→ FastAPI
→ PC

        ↓

[Controlled Experiment]

Validation Dataset
→ Threshold 비교

Day 08 Variant
→ 96 / 64 Accuracy 비교

Raspberry Pi
→ Model-only Benchmark

Manual Camera Experiment
→ Edge Service Stop
→ Hold 0 / 1 / 2 비교
→ Config 복원
→ Edge Service Restart

        ↓

[Failure Analysis]

Edge Service Stop
→ Camera Failure Case 촬영
→ Manifest
→ AI 재분석
→ Failure CSV
→ Root Cause 분류
→ Edge Service Restart

        ↓

Mini Challenge
        ↓
Challenge 복원
        ↓
전체 Test 재실행
        ↓
두 Service active 확인
        ↓
README / Report / Git
        ↓
12일차 Raspberry Pi → Jetson 연결
```

---

## 오늘의 성공 기준

```text
10일차 Baseline 확인
        ↓
Decision Test PASS
        ↓
Stabilizer Test PASS
        ↓
Runtime State Test PASS
        ↓
Fixed Image Integration PASS
        ↓
API Smoke Test PASS
        ↓
System Test PASS
        ↓
Operational AI Event Log 새 Row 확인 PASS
        ↓
한 변수 실험 3종 기록
        ↓
Failure Case 5개 이상
        ↓
Failure Analysis CSV 생성
        ↓
Trade-off 설명
        ↓
Mini Challenge 완료
        ↓
Challenge 변경 복원
        ↓
전체 pytest PASS
        ↓
subject13-edge.service active
        ↓
subject13-api.service active
        ↓
/health 정상
        ↓
Report / README / Git 정리
        ↓
12일차 Handoff 준비
```

---

## 오늘은 새로 하지 않는 것

```text
새 AI 모델 학습
→ 6일차에서 완료

ONNX Export 구조 새로 만들기
→ 7일차에서 완료

Latency / FPS 측정 도구 새로 설계
→ 8일차에서 완료

Frame Skip / Voting / Debounce / Hold 새로 학습
→ 9일차에서 완료

systemd / FastAPI 운영 구조 새로 만들기
→ 10일차에서 완료

Jetson / TensorRT 실습
→ 12일차
```

오늘은 **“기능을 더 많이 만드는 날”이 아니라 “지금 가진 기능이 어디까지 믿을 수 있는지 확인하는 날”**입니다.

---

# PART A. 10일차 운영 Baseline을 확인하고 Test 기준 고정하기

# 1. Raspberry Pi에 접속하기

PC에서 현재 Raspberry Pi 주소를 확인한 뒤 접속합니다.

```bash
ssh <RPI_USER>@<RPI_IP>
```

프로젝트 루트로 이동합니다.

```bash
cd ~/ai_vision/subject13_edge_ai
```

현재 위치와 장치를 확인합니다.

```bash
pwd
whoami
hostname
hostname -I
```

여기서 보이는 값은 교재 예시와 같을 필요가 없습니다.

---

# 2. 10일차 두 Service 상태 확인하기

```bash
systemctl status \
  subject13-edge.service \
  subject13-api.service \
  --no-pager
```

정상 Baseline이라면 두 Service에서 다음 핵심을 찾습니다.

```text
Active: active (running)
```

자동실행도 확인합니다.

```bash
systemctl is-enabled subject13-edge.service
systemctl is-enabled subject13-api.service
```

예상:

```text
enabled
enabled
```

둘 중 하나라도 실패하면 바로 Test를 진행하지 않습니다.

```text
Service FAIL
        ↓
journalctl 확인
        ↓
10일차 운영 문제 먼저 복구
        ↓
Service PASS
        ↓
11일차 Test 시작
```

---

# 3. Runtime State와 API Baseline 확인하기

현재 API Port를 먼저 확인합니다.

```bash
grep -E "^api_port:" configs/settings.yaml
```

Raspberry Pi 안에서:

```bash
curl http://127.0.0.1:<API_PORT>/health
```

```bash
curl http://127.0.0.1:<API_PORT>/status
```

```bash
curl http://127.0.0.1:<API_PORT>/metrics
```

Runtime State:

```bash
cat runtime/edge_state.json
```

### 예상 결과와 해석

정확한 숫자는 장비마다 다릅니다.

`/health`에서는 다음 의미가 중요합니다.

```text
api: OK
→ FastAPI Process가 요청에 응답함

edge: OK
→ 최근 Edge Runtime State가 정상

stale: false
→ State가 너무 오래되지 않음
```

`/status`에서는 다음을 봅니다.

```text
pred_class
confidence
raw_state
state
```

`/metrics`에서는 다음처럼 장비 상태를 봅니다.

```text
loop_fps
inference_per_sec
mean_inference_ms
p95_inference_ms
cpu_percent
memory_percent
temperature_c
```

### Operational AI Event Log Baseline도 확인하기

10일차의 운영 `scripts/edge_runtime.py`는 7일차의 `AIEventLogger`를 다시 사용하여 **첫 운영 State와 이후 최종 `output_state` 변화**를 Event CSV에 기록합니다. 이 Log는 `Runtime State`와 역할이 다릅니다.

```text
Runtime State
→ 지금 이 순간의 최신 상태

Operational AI Event Log
→ 실제 운영 State가 언제 어떻게 바뀌었는지 남긴 이력
```

Event Log 경로를 교재 예시로 고정하지 않고 현재 Config에서 읽습니다.

```bash
AI_EVENT_LOG_PATH="$(
  python -c 'from src.config_loader import load_config; print(load_config("configs/settings.yaml")["ai_event_log_path"])'
)"
```

확인:

```bash
echo "$AI_EVENT_LOG_PATH"
```

파일이 실제로 존재하는지 확인합니다.

```bash
test -f "$AI_EVENT_LOG_PATH"   && echo "EVENT LOG EXISTS"   || echo "EVENT LOG NOT FOUND"
```

최근 Row:

```bash
tail -n 5 "$AI_EVENT_LOG_PATH"
```

현재 줄 수도 기록합니다.

```bash
wc -l "$AI_EVENT_LOG_PATH"
```

정상 10일차 Baseline이라면 Header와 최소 한 개 이상의 운영 Event Row를 확인할 수 있어야 합니다. 10일차 Runtime은 시작 후 첫 운영 State를 한 번 기록하고, 이후에는 **최종 State가 바뀔 때만** 새 Row를 추가합니다.

이 단계에서 파일이 없으면 바로 System Test로 넘어가지 않습니다.

```text
Event Log 없음
        ↓
ai_event_log_path 실제 값 확인
        ↓
subject13-edge.service 실행 확인
        ↓
journalctl 확인
        ↓
10일차 edge_runtime.py에 AIEventLogger 연결 확인
        ↓
정상 Event Row 생성 확인
        ↓
11일차 Test 시작
```

`reports/day11_test_analysis.md`의 `10일차 Baseline`에도 다음을 기록합니다.

```md
Operational Event Log Path:

Operational Event Log Last Row:

Operational Event Log Baseline:
PASS / FAIL
```

---

# 4. PC에서도 API에 접근되는지 확인하기

PC에서:

```bash
curl http://<RPI_IP>:<API_PORT>/health
```

Raspberry Pi 안에서는 되는데 PC에서 안 된다면 다음 순서로 확인합니다.

```text
현재 RPI IP 확인
        ↓
api_port 확인
        ↓
Raspberry Pi Listen 상태 확인
        ↓
PC와 Pi가 서로 도달 가능한 Network인지 확인
        ↓
교육장 Firewall / Network 정책 확인
```

Listen Port:

```bash
ss -ltnp | grep ":<API_PORT>"
```

예시 Port `8000`을 무조건 사용하지 않습니다. 자신의 `configs/settings.yaml` 값을 사용합니다.

---

# 5. Source 상태 확인하기

현재 Git 상태를 봅니다.

```bash
git status
git log --oneline -10
git remote -v
```

### 내부 GitLab / Gitea Remote가 연결된 경우

기관 정책상 사용이 허용되고 Remote가 정상일 때만 동기화합니다.

```bash
git pull
```

### Local Git만 사용하는 경우

Remote가 없으면 `git pull`을 억지로 실행하지 않습니다.

현재 Raspberry Pi에 10일차 Source가 있는지 다음으로 확인합니다.

```bash
git status
git log --oneline -10
ls scripts/edge_runtime.py
ls scripts/status_api.py
ls src/runtime_state.py
```

`configs/device.local.yaml`은 Camera index처럼 **장치별 값**을 저장하므로 다른 PC의 파일로 덮어쓰지 않습니다.

---

# 6. 10일차 Baseline을 Git Tag로 표시하기

오늘 실험을 시작하기 전에 돌아갈 위치를 표시합니다.

먼저 변경사항이 없는지 확인합니다.

```bash
git status
```

Tag가 이미 있는지 확인합니다.

```bash
git tag --list day10-operational-baseline
```

아무것도 출력되지 않았다면 다음 Tag를 만듭니다.

```bash
git tag day10-operational-baseline
```

이미 Tag가 보인다면 같은 이름을 다시 만들지 않습니다.

확인:

```bash
git show \
  --stat \
  day10-operational-baseline
```

이 Tag의 의미:

```text
day10-operational-baseline
→ 11일차 실험을 시작하기 전
→ 두 Service와 API가 정상인 기준점
```

---

# 7. 11일차 실험 Branch 준비하기

오늘은 **PC와 Raspberry Pi 두 곳의 프로젝트를 모두 사용할 수 있습니다.** 그래서 먼저 어느 쪽을 Source 편집의 기준으로 사용할지 정합니다.

기본 흐름은 다음처럼 **PC를 주 작업 사본**으로 사용하는 것입니다.

```text
PC
→ Test Source 작성 / 수정
→ Local Git Commit
→ 내부 Remote 또는 수업에서 정한 파일 전달 방식
→ Raspberry Pi 동기화
→ Camera / GPIO / systemd가 필요한 실습 실행
```

이렇게 하면 같은 파일을 PC와 Raspberry Pi에서 서로 다르게 수정하여 버전이 갈리는 일을 줄일 수 있습니다.

PC 프로젝트로 이동합니다.

```bash
cd ~/ai_vision/subject13_edge_ai
```

현재 Branch를 확인합니다.

```bash
git branch --show-current
```

11일차 실험 Branch가 아직 없다면 만듭니다.

```bash
git switch -c experiment/day11
```

이미 만들어 둔 Branch라면 이동합니다.

```bash
git switch experiment/day11
```

확인합니다.

```bash
git branch
```

예:

```text
* experiment/day11
  main
```

수업 환경에서 VS Code Remote SSH로 **Raspberry Pi 자체를 주 작업 사본으로 사용하도록 정했다면**, 같은 원칙을 Raspberry Pi 프로젝트에 적용할 수 있습니다. 중요한 것은 한 가지입니다.

> **PC와 Raspberry Pi에서 같은 Source 파일을 동시에 서로 다르게 수정하지 않습니다. 어느 사본이 오늘의 기준인지 먼저 정하고, Hardware 실습 전에는 두 환경을 동기화합니다.**

외부 GitHub Push는 필수가 아닙니다.

```text
Local Git
→ 실험 이력 관리

내부 GitLab / Gitea Remote
→ 기관 정책과 수업 환경이 허용될 때만 사용

Remote가 없는 환경
→ SCP 또는 수업에서 정한 내부 파일 전달 방식 사용
```

`configs/device.local.yaml`은 Raspberry Pi마다 Camera index 등 장치값이 다를 수 있으므로 PC의 파일로 덮어쓰지 않습니다.

---

# 8. Day 11 Report를 먼저 만들기

오늘은 결과를 마지막에 기억으로 적지 않습니다.

실험을 수행할 때마다 바로 기록합니다.

파일:

```text
reports/day11_test_analysis.md
```

다음 뼈대로 시작합니다.

````md
# Day 11 Test · Failure Analysis · Trade-off

## Environment

PC:

Raspberry Pi Hostname:

Raspberry Pi IP:

API Port:

Camera Index:

ONNX Model:

Operational Threshold:

Frame Skip:

Vote Window:

Warning Minimum:

Debounce:

Hold:

## 10일차 Baseline

Edge Service:

API Service:

Edge Enabled:

API Enabled:

/health:

Runtime State:

Operational Event Log Path:

Operational Event Log Last Row:

Operational Event Log Baseline:
PASS / FAIL

## Unit Test

## Integration Test

## System Test

## Experiment 01 — Confidence Threshold

## Experiment 02 — Input Size

## Experiment 03 — Warning Hold

## Failure Analysis

## Overall Trade-off

## Final Decision

## 12일차 Handoff
````

이 파일은 오늘 하루 동안 계속 채웁니다.

---

# 9. Unit · Integration · System Test의 차이 이해하기

하나의 큰 프로그램만 실행하고 “된다 / 안 된다”라고 하지 않습니다.

```text
Unit Test

작은 함수 / Class
→ 입력을 직접 만들어 넣음
→ 기대 결과와 비교
→ Hardware 없이도 가능한 경우가 많음


Integration Test

여러 모듈 연결
→ 저장 Image
→ Preprocess
→ ONNX
→ Decision
→ 연결이 맞는지 확인


System Test

실제 전체 환경
→ Camera
→ Edge Runtime
→ GPIO
→ Operational AI Event Log
→ systemd
→ Runtime State
→ FastAPI
→ PC
```

문제가 생겼을 때 Test 범위가 나뉘어 있으면 원인을 좁히기 쉽습니다.

예:

```text
Decision Unit Test PASS
+
ONNX Fixed Image Integration PASS
+
실제 Camera System Test FAIL
        ↓
Decision Logic보다는
Camera / Runtime / Device 쪽을 먼저 의심
```

---

# 10. Hardware가 필요한 코드와 필요 없는 코드 구분하기

Hardware가 필요한 대표 코드:

```text
src/camera_input.py
src/gpio_output.py
scripts/edge_runtime.py의 실제 Camera/GPIO 구간
```

Hardware 없이도 작은 입력값으로 Test 가능한 코드:

```text
src/decision.py
src/stabilizer.py
src/runtime_state.py
일부 ai_preprocess 로직
```

오늘 Unit Test는 이 분리를 이용합니다.

---

# PART B. pytest로 작은 Logic부터 자동 검증하기

# 11. pytest를 왜 사용할까요?

지금까지는 코드를 실행하고 사람이 결과를 눈으로 확인했습니다.

예:

```text
warning_red + 0.95
→ WARNING
```

하지만 같은 조건을 반복해서 확인하려면 자동 Test가 편리합니다.

```text
입력
        ↓
함수 실행
        ↓
실제 결과
        ↓
assert
        ↓
기대 결과와 같은가?
```

`pytest`는 이런 작은 검증 코드를 모아 한 번에 실행하는 도구입니다.

---

# 12. pytest 설치 여부 확인하기

Unit Test는 먼저 PC Linux/WSL2 또는 현재 개발환경에서 진행합니다.

프로젝트 루트:

```bash
cd ~/ai_vision/subject13_edge_ai
```

가상환경:

```bash
source .venv/bin/activate
```

확인:

```bash
python -m pytest --version
```

설치되어 있지 않다면:

```bash
python -m pip install pytest
```

폐쇄망에서 Package 설치가 되지 않는다면 무조건 외부 인터넷을 시도하지 않습니다.

```text
내부 Package Mirror
또는
사전 제공 Wheel
또는
교육장에 준비된 Package
```

가 있는지 먼저 확인합니다.

설치 후:

```bash
python -m pytest --version
```

12일차에서 Raspberry Pi에서도 같은 Test를 다시 실행할 수 있으므로 Pi 환경에도 `pytest`가 필요할 수 있습니다.

---

# 13. Test 폴더 만들기

새 파일을 만들기 전에 역할부터 확인합니다.

```text
src/
→ 실제 프로그램 Logic

scripts/
→ 사용자가 실행하는 프로그램

tests/
→ 실제 프로그램이 기대대로 동작하는지 확인하는 검증 코드
```

프로젝트 루트에서:

```bash
mkdir -p tests
```

구조:

```text
subject13_edge_ai/
└── tests/
```

---

# 14. Decision Unit Test 만들기

## 이 파일은 왜 필요할까요?

7일차의 `src/decision.py`는 다음 운영 규칙을 담당합니다.

```text
predicted_class == warning_class
그리고
confidence >= confidence_threshold
        ↓
WARNING

그 외
        ↓
NORMAL
```

오늘 새로 Decision Logic을 만드는 것이 아닙니다.

**이전 일차의 Decision이 앞으로 수정되어도 현재 정책이 유지되는지 자동으로 확인하는 Test**를 만듭니다.

## 의사코드

```text
ai_to_operational_state() 가져오기
        ↓
경고 Class 이름 준비
        ↓
여러 입력 조합 만들기
        ↓
함수 실행
        ↓
실제 State와 기대 State 비교
        ↓
경계값 0.80도 확인
        ↓
하나라도 다르면 Test FAIL
```

파일:

```text
tests/test_decision.py
```

코드:

```python
from src.decision import (
    ai_to_operational_state,
)


WARNING_CLASS = "warning_red"


def test_warning_high_confidence():
    state = ai_to_operational_state(
        predicted_class="warning_red",
        confidence=0.95,
        warning_class=WARNING_CLASS,
        confidence_threshold=0.80,
    )

    assert state == "WARNING"


def test_warning_low_confidence():
    state = ai_to_operational_state(
        predicted_class="warning_red",
        confidence=0.50,
        warning_class=WARNING_CLASS,
        confidence_threshold=0.80,
    )

    assert state == "NORMAL"


def test_normal_class_high_confidence():
    state = ai_to_operational_state(
        predicted_class="normal",
        confidence=0.99,
        warning_class=WARNING_CLASS,
        confidence_threshold=0.80,
    )

    assert state == "NORMAL"


def test_below_boundary():
    state = ai_to_operational_state(
        predicted_class="warning_red",
        confidence=0.79,
        warning_class=WARNING_CLASS,
        confidence_threshold=0.80,
    )

    assert state == "NORMAL"


def test_equal_boundary():
    state = ai_to_operational_state(
        predicted_class="warning_red",
        confidence=0.80,
        warning_class=WARNING_CLASS,
        confidence_threshold=0.80,
    )

    assert state == "WARNING"
```

---

# 15. Decision Test 실행하기

```bash
python -m pytest \
  tests/test_decision.py \
  -v
```

예상:

```text
test_warning_high_confidence PASSED
test_warning_low_confidence PASSED
test_normal_class_high_confidence PASSED
test_below_boundary PASSED
test_equal_boundary PASSED
```

정확한 출력 줄 형식은 pytest 버전에 따라 조금 달라질 수 있습니다.

핵심은:

```text
5 passed
```

입니다.

### 결과 해석

마지막 Test가 PASS했다는 것은 현재 정책이 다음임을 의미합니다.

```text
confidence >= threshold
```

따라서:

```text
0.79 < 0.80
→ NORMAL

0.80 >= 0.80
→ WARNING
```

입니다.

### 코드 리뷰 — 이 결과는 어디에서 만들어졌을까요?

`tests/test_decision.py`는 결과를 직접 만들어 내는 운영 코드가 아닙니다.

연결:

```text
Test Input
→ tests/test_decision.py
        ↓
src/decision.py
→ ai_to_operational_state()
        ↓
NORMAL / WARNING
        ↓
assert
        ↓
PASS / FAIL
```

다음 질문에 답해 봅니다.

```text
1. 실제 상태를 결정하는 코드는 어느 파일인가?
2. 기대 결과를 적는 코드는 어느 파일인가?
3. 0.80이 WARNING인 이유는 어느 연산자 때문인가?
4. Test가 FAIL하면 실제 코드가 무조건 고장난 것인가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
1. src/decision.py의 ai_to_operational_state()
2. tests/test_decision.py의 assert
3. >= 연산자
4. 아니다.
   실제 코드가 잘못되었을 수도 있고,
   Test의 기대값이 잘못되었을 수도 있으므로
   둘을 비교해야 한다.
```

</details>

---

# 16. Stabilizer Unit Test 만들기

## 이 파일은 왜 필요할까요?

9일차에는 `TemporalStabilizer`로 다음 세 기능을 연결했습니다.

```text
Majority Voting
→ Debounce
→ WARNING Hold
```

실제 Camera를 연결하면 AI Prediction까지 함께 변하기 때문에 안정화 Logic만 따로 보기 어렵습니다.

그래서 가상의 `NORMAL / WARNING` 배열과 가상의 시간을 직접 넣어 Test합니다.

## 의사코드

```text
TemporalStabilizer 준비
        ↓
가상의 상태 Sequence 입력
        ↓
Voting 결과 확인
        ↓
Debounce 전환 횟수 확인
        ↓
Hold 시간 안과 밖 비교
        ↓
기대 output_state와 assert 비교
```

파일:

```text
tests/test_stabilizer.py
```

코드:

```python
from src.stabilizer import (
    TemporalStabilizer,
)


def test_majority_vote_3_of_5():
    stabilizer = TemporalStabilizer(
        vote_window=5,
        vote_warning_min=3,
        debounce_required=1,
        warning_hold_sec=0.0,
    )

    sequence = [
        "NORMAL",
        "WARNING",
        "WARNING",
        "NORMAL",
        "WARNING",
    ]

    result = None

    for index, raw_state in enumerate(sequence):
        result = stabilizer.update(
            raw_state=raw_state,
            now_sec=float(index),
        )

    assert result is not None
    assert result["warning_count"] == 3
    assert result["output_state"] == "WARNING"


def test_two_of_five_is_normal():
    stabilizer = TemporalStabilizer(
        vote_window=5,
        vote_warning_min=3,
        debounce_required=1,
        warning_hold_sec=0.0,
    )

    sequence = [
        "NORMAL",
        "WARNING",
        "NORMAL",
        "NORMAL",
        "WARNING",
    ]

    result = None

    for index, raw_state in enumerate(sequence):
        result = stabilizer.update(
            raw_state=raw_state,
            now_sec=float(index),
        )

    assert result is not None
    assert result["warning_count"] == 2
    assert result["output_state"] == "NORMAL"


def test_debounce_requires_two_candidates():
    stabilizer = TemporalStabilizer(
        vote_window=1,
        vote_warning_min=1,
        debounce_required=2,
        warning_hold_sec=0.0,
    )

    first = stabilizer.update(
        raw_state="WARNING",
        now_sec=0.0,
    )

    second = stabilizer.update(
        raw_state="WARNING",
        now_sec=1.0,
    )

    assert first["stable_state"] == "NORMAL"
    assert second["stable_state"] == "WARNING"


def test_warning_hold_keeps_output_temporarily():
    stabilizer = TemporalStabilizer(
        vote_window=1,
        vote_warning_min=1,
        debounce_required=1,
        warning_hold_sec=2.0,
    )

    warning = stabilizer.update(
        raw_state="WARNING",
        now_sec=0.0,
    )

    normal_inside_hold = stabilizer.update(
        raw_state="NORMAL",
        now_sec=1.0,
    )

    normal_after_hold = stabilizer.update(
        raw_state="NORMAL",
        now_sec=2.1,
    )

    assert warning["output_state"] == "WARNING"
    assert normal_inside_hold["stable_state"] == "NORMAL"
    assert normal_inside_hold["output_state"] == "WARNING"
    assert normal_after_hold["output_state"] == "NORMAL"
```

---

# 17. Stabilizer Test 실행하기

```bash
python -m pytest \
  tests/test_stabilizer.py \
  -v
```

예상 핵심:

```text
4 passed
```

### 결과 해석

```text
3 of 5
→ WARNING

2 of 5
→ NORMAL

Debounce 2
→ 첫 후보 1회만으로 바로 stable_state 변경하지 않음

Hold 2초
→ stable_state가 NORMAL로 돌아와도
   Hold 시간 안에서는 output_state WARNING 유지 가능
```

### 코드 리뷰

다음 연결을 직접 찾습니다.

```text
가상 Sequence
→ test_stabilizer.py
        ↓
TemporalStabilizer.update()
→ src/stabilizer.py
        ↓
warning_count
→ voted_state
→ candidate_count
→ stable_state
→ output_state
        ↓
assert
```

`warning_hold_sec`가 Model Confidence를 바꾸는 값이 아니라 **최종 운영 출력의 시간 동작을 바꾸는 값**이라는 점도 다시 확인합니다.

---

# 18. Runtime State Unit Test 만들기

## 이 파일은 왜 필요할까요?

10일차의 Edge Runtime과 FastAPI는 다음 파일을 사이에 두고 정보를 공유합니다.

```text
Edge Runtime
→ runtime/edge_state.json
→ FastAPI
```

실제 Runtime 파일을 Test 중에 망가뜨리지 않도록 `pytest`가 제공하는 `tmp_path` 임시 폴더를 사용합니다.

이번 Test는 세 가지를 확인합니다.

```text
정상 Write / Read

파일이 없을 때
→ {}

잘못된 JSON일 때
→ {}
```

## 의사코드

```text
pytest 임시 폴더 받기
        ↓
임시 edge_state.json 경로 만들기
        ↓
RuntimeStateStore.write()
        ↓
RuntimeStateStore.read()
        ↓
값과 timestamp 확인

다른 임시 경로
        ↓
파일이 없음
        ↓
read()
        ↓
{} 확인

잘못된 JSON 작성
        ↓
read()
        ↓
{} 확인
```

파일:

```text
tests/test_runtime_state.py
```

코드:

```python
from src.runtime_state import (
    RuntimeStateStore,
)


def test_runtime_state_write_and_read(
    tmp_path,
):
    path = tmp_path / "edge_state.json"

    store = RuntimeStateStore(
        str(path)
    )

    store.write(
        {
            "service": "edge",
            "status": "RUNNING",
            "state": "NORMAL",
        }
    )

    data = store.read()

    assert data["service"] == "edge"
    assert data["status"] == "RUNNING"
    assert data["state"] == "NORMAL"
    assert "timestamp" in data


def test_runtime_state_missing_file_returns_empty(
    tmp_path,
):
    path = tmp_path / "missing_state.json"

    store = RuntimeStateStore(
        str(path)
    )

    assert store.read() == {}


def test_runtime_state_invalid_json_returns_empty(
    tmp_path,
):
    path = tmp_path / "broken_state.json"

    path.write_text(
        "{broken-json",
        encoding="utf-8",
    )

    store = RuntimeStateStore(
        str(path)
    )

    assert store.read() == {}
```

---

# 19. Runtime State Test 실행하기

```bash
python -m pytest \
  tests/test_runtime_state.py \
  -v
```

예상 핵심:

```text
3 passed
```

### 코드 리뷰

```text
tmp_path
→ pytest가 만든 임시 폴더

store.write()
→ src/runtime_state.py

timestamp 추가
→ RuntimeStateStore.write()

JSON 다시 읽기
→ RuntimeStateStore.read()

파일 없음 / JSON 오류
→ {}
```

실제 `runtime/edge_state.json`을 사용하지 않았다는 점이 중요합니다.

---

# 20. 전체 Unit Test 실행하기

```bash
python -m pytest \
  tests \
  -v
```

현재 교재의 세 Test 파일만 있다면 예상 Test 수는 다음과 같습니다.

```text
Decision
→ 5

Stabilizer
→ 4

Runtime State
→ 3

합계
→ 12 tests
```

정상이라면:

```text
12 passed
```

기존 프로젝트에 다른 Test가 이미 있다면 전체 Test 수는 더 많을 수 있습니다.

---

# 21. Guided Error — 일부러 Test 하나를 실패시켜 보기

정상 Test가 먼저 PASS한 뒤 진행합니다.

실제 `src/decision.py`는 수정하지 않습니다.

`tests/test_decision.py`의 마지막 기대값만 잠시 다음처럼 바꿉니다.

기존:

```python
assert state == "WARNING"
```

임시 변경:

```python
assert state == "NORMAL"
```

실행:

```bash
python -m pytest \
  tests/test_decision.py \
  -v
```

### 오류 → 원인 확인 → 수정 → 재실행

```text
오류
→ AssertionError / FAILED

원인 확인
→ 실제 state와 기대 state가 다름

코드 확인
→ src/decision.py는 >= 정책
→ Test 기대값만 임시로 NORMAL로 바뀜

수정
→ assert state == "WARNING"

재실행
→ pytest PASS
```

복구 후:

```bash
python -m pytest \
  tests/test_decision.py \
  -v
```

이 실습에서 기억할 점:

```text
Test FAIL
≠
무조건 실제 프로그램 버그
```

Test 자체의 기대값이 틀렸을 수도 있습니다.

---

# 22. Unit Test 결과를 Report에 기록하기

`reports/day11_test_analysis.md`의 `Unit Test` 부분을 채웁니다.

````md
## Unit Test

| Test File | Test Count | Passed | Failed | Hardware 필요 |
|---|---:|---:|---:|---|
| test_decision.py | 5 |  |  | 아니오 |
| test_stabilizer.py | 4 |  |  | 아니오 |
| test_runtime_state.py | 3 |  |  | 아니오 |

### Boundary Policy

현재 정책:

`confidence >= confidence_threshold`

### Guided Failure

일부러 바꾼 기대값:

실제 오류:

원인:

복구 후 결과:
````

Unit Test가 끝났다면 다음 질문에 답할 수 있어야 합니다.

> **“Camera가 없어도 어떤 부분까지 자동으로 검증할 수 있는가?”**

---

# PART C. 저장 Image와 API로 Integration Test하기

# 23. Integration Test는 무엇을 확인할까요?

Unit Test에서는 작은 함수에 직접 값을 넣었습니다.

이제는 여러 모듈이 연결될 때도 같은 결과가 나오는지 확인합니다.

오늘 첫 번째 Integration Test:

```text
저장 Image
        ↓
Preprocess
        ↓
ONNX Runtime
        ↓
Class + Confidence
        ↓
Operational Decision
```

실제 Camera는 아직 사용하지 않습니다.

이렇게 하면 Camera 상태와 무관하게 **전처리·Model·Decision 연결**을 확인할 수 있습니다.

---

# 24. 7일차 배포 Sample 다시 확인하기

이전 일차에서 만든 파일을 다시 사용합니다.

```text
data/deploy_samples/
├── normal.jpg
└── warning_red.jpg
```

확인:

```bash
ls -lh data/deploy_samples/
```

파일이 없다면 새 이름을 임의로 만들지 않습니다.

7일차에서 사용한 실제 Sample 경로를 찾습니다.

```bash
find data \
  -maxdepth 3 \
  -type f \
  \( -name "*.jpg" -o -name "*.jpeg" -o -name "*.png" \) \
  | sort
```

---

# 25. normal Image로 ONNX → Decision 연결 확인하기

`onnx_decision_test.py`는 **7일차에서 만든 파일을 다시 사용**합니다.

새로 작성하지 않습니다.

현재 운영 Threshold를 먼저 확인합니다.

```bash
grep -E "^ai_confidence_threshold:" configs/settings.yaml
```

실행:

```bash
python -m scripts.onnx_decision_test \
  --image data/deploy_samples/normal.jpg
```

필요하면 특정 Threshold를 명령행에서 지정할 수 있습니다.

```bash
python -m scripts.onnx_decision_test \
  --image data/deploy_samples/normal.jpg \
  --threshold 0.70
```

### 예상 결과 형태

```text
Prediction : normal
Confidence : <실제 값>
Threshold  : 0.70
State      : NORMAL
```

정확한 Confidence는 Model과 Image에 따라 달라질 수 있습니다.

`normal.jpg`가 반드시 `NORMAL`이 나와야 한다고 눈으로 단정하지 말고 실제 Label과 결과를 확인합니다. 실패하면 그 자체가 11일차 Failure Analysis의 단서가 됩니다.

---

# 26. warning_red Image로 ONNX → Decision 연결 확인하기

```bash
python -m scripts.onnx_decision_test \
  --image data/deploy_samples/warning_red.jpg
```

예상 형태:

```text
Prediction : warning_red
Confidence : <실제 값>
Threshold  : <현재 설정값>
State      : WARNING
```

다음 두 결과를 구분합니다.

```text
Prediction
→ AI가 예측한 Class

State
→ Class + Confidence + 운영 Threshold로 만든 운영 결과
```

예를 들어:

```text
Prediction = warning_red
Confidence = 0.62
Threshold = 0.70
        ↓
State = NORMAL
```

이 될 수도 있습니다.

AI 예측과 운영 판정은 같은 개념이 아닙니다.

---

# 27. Integration 결과를 코드에서 역추적하기

실행 결과만 보고 넘어가지 않습니다.

다음 파일을 직접 엽니다.

```text
scripts/onnx_decision_test.py
src/ai_preprocess.py
src/inference_onnx.py
src/decision.py
configs/settings.yaml
```

연결:

```text
이미지 경로
→ onnx_decision_test.py
        ↓
image_path_to_nchw()
→ ai_preprocess.py
        ↓
ONNXClassifier.predict()
→ inference_onnx.py
        ↓
class_name / confidence
        ↓
ai_to_operational_state()
→ decision.py
        ↓
NORMAL / WARNING
```

다음 질문에 답합니다.

```text
1. Image를 Tensor로 바꾸는 함수는 어디에 있는가?
2. ONNX Session을 실제로 호출하는 Class는 무엇인가?
3. confidence가 만들어지는 곳은 어디인가?
4. Threshold는 어디에서 가져오는가?
5. 최종 State는 어느 함수에서 결정되는가?
```

이 다섯 위치를 찾았다면 Integration 결과를 코드로 설명할 수 있습니다.

---

# 28. PyTorch와 ONNX Parity를 다시 확인하기

이 파일도 **7일차에서 만든 `scripts/compare_pytorch_onnx.py`를 다시 사용**합니다.

PyTorch Checkpoint가 있는 PC에서 진행합니다.

```bash
python -m scripts.compare_pytorch_onnx \
  --image data/deploy_samples/normal.jpg
```

```bash
python -m scripts.compare_pytorch_onnx \
  --image data/deploy_samples/warning_red.jpg
```

확인:

```text
Class Match
→ True인지 확인

Max Prob Diff
→ 두 Runtime의 확률 차이 확인
```

동일한 입력과 같은 Weight를 사용해도 부동소수점 연산 차이로 확률이 완전히 같은 문자열이 되지 않을 수 있습니다.

오늘은 다음을 봅니다.

```text
Class가 같은가?

Probability 차이가 비정상적으로 크지 않은가?
```

Parity가 깨졌다면 Threshold 실험 전에 Export·Metadata·Preprocess 문제를 먼저 해결합니다.

---

# 29. FastAPI Smoke Test 프로그램을 만들기 전에 역할 이해하기

10일차에는 Browser와 `curl`로 API를 확인했습니다.

11일차에는 같은 확인을 작은 프로그램으로 반복합니다.

Smoke Test의 목적:

```text
API가 응답하는가?
        ↓
필수 Endpoint가 있는가?
        ↓
필수 JSON Key가 있는가?
```

오늘 Smoke Test가 Model Accuracy까지 검증하지는 않습니다.

```text
API Contract 확인
≠
AI 성능 평가
```

---

# 30. FastAPI Smoke Test 프로그램 만들기

## 이 파일은 왜 필요할까요?

PC에서 다음 세 Endpoint를 자동으로 호출합니다.

```text
/health
/status
/metrics
```

그리고 최소한 필요한 JSON Key가 있는지 확인합니다.

IP와 Port는 환경마다 다르므로 코드에 `192.168...` 또는 `8000`을 고정하지 않습니다.

명령행 인자로 받습니다.

## 의사코드

```text
--host / --port / --timeout 받기
        ↓
Base URL 만들기
        ↓
/health 요청
/status 요청
/metrics 요청
        ↓
JSON으로 변환
        ↓
각 Endpoint에 필요한 Key 확인
        ↓
health의 api 값도 확인
        ↓
모두 만족
→ PASS
        ↓
연결 실패 / Key 없음
→ 오류 출력
```

파일:

```text
scripts/api_smoke_test.py
```

코드:

```python
import argparse
import json
from urllib.request import urlopen


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--host",
        required=True,
        help="Raspberry Pi의 현재 IPv4 또는 Hostname",
    )

    parser.add_argument(
        "--port",
        type=int,
        required=True,
        help="configs/settings.yaml의 api_port",
    )

    parser.add_argument(
        "--timeout",
        type=float,
        default=3.0,
    )

    return parser.parse_args()


def get_json(
    url: str,
    timeout: float,
) -> dict:
    with urlopen(
        url,
        timeout=timeout,
    ) as response:
        body = response.read().decode(
            "utf-8"
        )

    return json.loads(body)


def check_keys(
    name: str,
    data: dict,
    required: list[str],
) -> None:
    missing = [
        key
        for key in required
        if key not in data
    ]

    if missing:
        raise AssertionError(
            f"{name} missing keys: {missing}"
        )

    print(
        f"{name:10s}: PASS"
    )


def main():
    args = parse_args()

    if not (
        1 <= args.port <= 65535
    ):
        raise ValueError(
            "port는 1~65535 범위여야 합니다."
        )

    if args.timeout <= 0:
        raise ValueError(
            "timeout은 0보다 커야 합니다."
        )

    base = (
        f"http://{args.host}:{args.port}"
    )

    health = get_json(
        base + "/health",
        args.timeout,
    )

    status = get_json(
        base + "/status",
        args.timeout,
    )

    metrics = get_json(
        base + "/metrics",
        args.timeout,
    )

    check_keys(
        "/health",
        health,
        [
            "api",
            "edge",
            "edge_status",
            "state_age_sec",
            "stale",
        ],
    )

    check_keys(
        "/status",
        status,
        [
            "status",
            "state",
            "pred_class",
            "confidence",
            "stale",
        ],
    )

    check_keys(
        "/metrics",
        metrics,
        [
            "loop_fps",
            "inference_per_sec",
            "mean_inference_ms",
            "p95_inference_ms",
            "cpu_percent",
            "memory_percent",
            "temperature_c",
            "stale",
        ],
    )

    if health["api"] != "OK":
        raise AssertionError(
            "FastAPI가 api=OK를 반환하지 않았습니다."
        )

    print()
    print("=== API Smoke Test PASS ===")
    print(
        f"Edge Health : {health['edge']}"
    )
    print(
        f"Stale       : {health['stale']}"
    )


if __name__ == "__main__":
    main()
```

---

# 31. API Smoke Test 실행하기

먼저 Raspberry Pi의 현재 값을 확인합니다.

```bash
hostname -I
grep -E "^api_port:" configs/settings.yaml
```

PC에서:

```bash
python -m scripts.api_smoke_test \
  --host <RPI_IP> \
  --port <API_PORT>
```

예상 형태:

```text
/health   : PASS
/status   : PASS
/metrics  : PASS

=== API Smoke Test PASS ===
Edge Health : OK
Stale       : False
```

`Edge Health`가 `DEGRADED`인데 Key 검사는 PASS할 수도 있습니다.

이 경우 의미는 다음과 같습니다.

```text
API 형식
→ 정상

하지만

Edge Runtime 현재 상태
→ 별도 문제 가능
```

따라서 Smoke Test의 PASS와 System 상태의 PASS를 같은 것으로 생각하지 않습니다.

---

# 32. API Smoke Test 결과를 코드에서 역추적하기

```text
<RPI_IP> / <API_PORT>
→ parse_args()

/health /status /metrics URL
→ main()

HTTP 요청
→ get_json()

JSON Key 확인
→ check_keys()

PASS 출력
→ check_keys()

Edge Health / Stale 출력
→ main()
```

FastAPI 쪽에서는 10일차 파일을 다시 확인합니다.

```text
scripts/status_api.py
```

다음 연결을 찾습니다.

```text
RuntimeStateStore.read()
        ↓
state_age_sec()
        ↓
/health
/status
/metrics
```

---

# 33. API Smoke Test에서 자주 만나는 오류

### 오류 A — `ConnectionRefusedError`

```text
오류
→ 해당 Host:Port에 연결할 수 없음

원인 확인
→ API Service 상태
→ 실제 Listen Port
→ 입력한 Port

확인
→ systemctl status subject13-api.service
→ ss -ltnp | grep ":<API_PORT>"

수정
→ 실제 Port 사용
→ 필요하면 API Service 복구

재실행
→ api_smoke_test.py
```

### 오류 B — `timed out`

```text
오류
→ 요청 시간 안에 연결 또는 응답이 끝나지 않음

원인 확인
→ 현재 RPI IP
→ PC ↔ Pi Network
→ 교육장 Firewall
→ API Process

수정
→ 올바른 IP
→ Network 복구
→ 필요하면 timeout을 약간 늘려 재확인

재실행
```

### 오류 C — `missing keys`

```text
오류
→ Endpoint는 응답했지만 예상 JSON 구조와 다름

원인 확인
→ Browser / curl로 실제 JSON 확인
→ scripts/status_api.py 확인

수정
→ API와 Smoke Test 중 어느 쪽 계약이 맞는지 확인

재실행
```

---

# 34. Integration Test 결과를 Report에 기록하기

````md
## Integration Test

### Fixed Image

| Image | Prediction | Confidence | Operational State | PASS / FAIL |
|---|---|---:|---|---|
| normal.jpg |  |  |  |  |
| warning_red.jpg |  |  |  |  |

### PyTorch ↔ ONNX

| Image | Class Match | Max Prob Diff | 해석 |
|---|---|---:|---|
| normal.jpg |  |  |  |
| warning_red.jpg |  |  |  |

### API Smoke Test

Host:

Port:

/health:

/status:

/metrics:

Edge Health:

Stale:
````

---

# PART D. 실제 Raspberry Pi 전체 흐름을 System Test하기

# 35. System Test를 시작하기 전에 확인할 것

System Test는 실제 Hardware와 운영환경을 모두 사용합니다.

```text
Camera
→ Edge Runtime
→ ONNX
→ Decision
→ Stabilization
→ GPIO
→ Operational AI Event Log
→ Runtime State
→ FastAPI
→ PC
```

따라서 Unit Test보다 실패 지점이 많습니다.

System Test에서는 “결과가 다르다”만 기록하지 말고 **어느 계층까지 정상인지** 확인합니다.

---

# 36. System Test Checklist 만들기

`reports/day11_test_analysis.md`에 추가합니다.

````md
## System Test

| 기능 | 확인 방법 | 결과 | Evidence / 메모 |
|---|---|---|---|
| Edge Service | systemctl status | PASS / FAIL |  |
| API Service | systemctl status | PASS / FAIL |  |
| Edge Enabled | systemctl is-enabled | PASS / FAIL |  |
| API Enabled | systemctl is-enabled | PASS / FAIL |  |
| Runtime State | edge_state.json timestamp | PASS / FAIL |  |
| Camera | Runtime State / 실제 입력 | PASS / FAIL |  |
| ONNX Prediction | /status | PASS / FAIL |  |
| NORMAL GPIO | Green LED | PASS / FAIL |  |
| WARNING GPIO | Red LED / 선택 Buzzer | PASS / FAIL |  |
| Operational Event Log Path | `ai_event_log_path` 확인 | PASS / FAIL |  |
| Operational Event Log New Row | State 전환 전·후 줄 수 / 최근 Timestamp 비교 | PASS / FAIL |  |
| Operational Event Log State | CSV `state`가 최종 `/status state`와 일치 | PASS / FAIL |  |
| /health | curl | PASS / FAIL |  |
| /status | curl | PASS / FAIL |  |
| /metrics | curl | PASS / FAIL |  |
| Auto Restart | 통제된 Process Kill | PASS / FAIL |  |
| Journal | journalctl | PASS / FAIL |  |
````

---

# 37. NORMAL 상태 System Test

Camera 앞에서 경고 대상을 제거합니다.

PC:

```bash
curl http://<RPI_IP>:<API_PORT>/status
```

실제 결과에서 확인합니다.

```text
state
→ NORMAL

GPIO
→ Green LED
```

Model Prediction과 State가 다를 수 있으므로 둘 다 기록합니다.

```text
pred_class
confidence
raw_state
state
```

---

# 38. WARNING 상태 System Test

경고 대상을 Camera 앞에 보여줍니다.

PC:

```bash
curl http://<RPI_IP>:<API_PORT>/status
```

확인:

```text
pred_class
→ warning_red가 예상되는 상황

state
→ WARNING이 예상되는 상황

GPIO
→ Red LED
→ use_buzzer=true이고 배선이 검증된 경우 Buzzer
```

실제 결과가 다르면 억지로 PASS로 기록하지 않습니다.

`FAIL`을 남기고 뒤의 Failure Analysis에서 원인을 분류합니다.

### Operational AI Event Log System Test — 현재 실행에서 새 Row가 생기는가?

10일차에서 Event Log가 존재하는지만 확인했다면, 11일차에서는 **지금 수행한 System Test 때문에 실제 새 Row가 추가되었는지** 증명합니다. 과거 7일차에 남아 있던 CSV를 보는 것만으로는 PASS가 아닙니다.

먼저 Config에서 실제 경로를 읽습니다.

```bash
AI_EVENT_LOG_PATH="$(
  python -c 'from src.config_loader import load_config; print(load_config("configs/settings.yaml")["ai_event_log_path"])'
)"
```

경로:

```bash
echo "$AI_EVENT_LOG_PATH"
```

현재 줄 수를 저장합니다.

```bash
BEFORE_ROWS="$(wc -l < "$AI_EVENT_LOG_PATH")"
echo "Before rows: $BEFORE_ROWS"
```

현재 `/status`의 최종 운영 State도 확인합니다.

```bash
curl http://127.0.0.1:<API_PORT>/status
```

이제 Camera 앞 장면을 바꾸어 **최종 `state`가 실제로 반대 상태로 전환될 때까지** 기다립니다.

예:

```text
현재 state = NORMAL
        ↓
경고 대상 제시
        ↓
Voting / Debounce / Hold 통과
        ↓
/status state = WARNING 확인
```

또는 반대로:

```text
현재 state = WARNING
        ↓
경고 대상 제거
        ↓
Hold 시간이 끝날 때까지 기다림
        ↓
/status state = NORMAL 확인
```

전환 후 줄 수를 다시 확인합니다.

```bash
AFTER_ROWS="$(wc -l < "$AI_EVENT_LOG_PATH")"
echo "After rows : $AFTER_ROWS"
```

최근 Event:

```bash
tail -n 5 "$AI_EVENT_LOG_PATH"
```

PASS 기준은 다음 세 가지입니다.

```text
1. AFTER_ROWS > BEFORE_ROWS

2. 마지막 Event의 timestamp가
   이번 System Test 이후의 시각이다.

3. 마지막 Event의 state가
   실제 최종 /status state와 일치한다.
```

중요한 점은 `raw_state`가 잠깐 바뀌었다고 무조건 Event Row가 생기는 것이 아니라는 것입니다.

```text
raw_state 순간 변화
        ↓
Voting / Debounce / Hold에서 흡수
        ↓
최종 output_state 변화 없음
        ↓
새 Event Row 없음
```

반대로:

```text
최종 output_state 변화
        ↓
GPIO 출력 변화
        ↓
AIEventLogger.append()
        ↓
새 Event CSV Row
```

가 정상입니다.

#### 코드 리뷰 — 새 Event Row는 어디에서 만들어졌을까요?

다음 Producer → Consumer 연결을 직접 찾습니다.

```text
scripts/edge_runtime.py
        ↓
last_state
= TemporalStabilizer.update()["output_state"]
        ↓
last_state != previous_logged_state
        ↓
src/event_logger.py
AIEventLogger.append()
        ↓
configs/settings.yaml
ai_event_log_path
        ↓
Operational Event CSV
```

다음 질문에 답해 봅니다.

```text
1. Event Log의 state는 raw_state인가, 최종 last_state인가?
2. 모든 Camera Frame마다 CSV 한 줄을 추가하는가?
3. 현재 System Test가 새 Row를 만들었다는 가장 간단한 증거는 무엇인가?
4. 파일이 존재하지만 줄 수가 늘지 않았다면 무엇부터 확인해야 하는가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
1. Voting / Debounce / Hold가 반영된 최종 last_state이다.

2. 아니다.
   첫 운영 State와 이후 최종 State가 바뀌는 순간만 기록한다.

3. System Test 전후 줄 수 비교 +
   마지막 timestamp +
   마지막 state 확인이다.

4. /status의 최종 state가 실제로 전환되었는지,
   ai_event_log_path가 실제 Writer 경로와 같은지,
   subject13-edge.service가 최신 10일차 Source를 실행하는지,
   journalctl에 오류가 없는지 순서대로 확인한다.
```

</details>

#### Event Log가 증가하지 않을 때 — 오류 → 원인 확인 → 수정 → 재실행

```text
오류
→ State를 바꿨다고 생각했지만 Event CSV 줄 수가 그대로임

원인 확인 1
→ curl /status
→ 실제 최종 state가 바뀌었는가?

원인 확인 2
→ echo "$AI_EVENT_LOG_PATH"
→ settings.yaml의 실제 ai_event_log_path와 같은가?

원인 확인 3
→ systemctl status subject13-edge.service
→ 최신 Runtime이 실행 중인가?

원인 확인 4
→ journalctl -u subject13-edge.service -n 50 --no-pager
→ Logger 또는 파일 쓰기 오류가 있는가?

수정
→ 잘못된 경로 / 오래된 Source / Service 상태를 정상화

재실행
→ 최종 state 전환
→ wc -l 전후 비교
→ tail -n 5 확인
```

System Test Checklist의 `Operational Event Log` 세 항목에도 결과와 Evidence를 기록합니다.

---

# 39. Metrics System Test

PC:

```bash
curl http://<RPI_IP>:<API_PORT>/metrics
```

다음 값이 존재하고 시간이 지나며 갱신되는지 확인합니다.

```text
loop_fps
inference_per_sec
mean_inference_ms
p95_inference_ms
cpu_percent
memory_percent
temperature_c
state_age_sec
stale
```

숫자 자체는 장비마다 다릅니다.

오늘 System Test의 첫 목표는:

```text
값이 존재하는가?
        ↓
계속 갱신되는가?
        ↓
비정상적인 0 / None / stale가 아닌가?
```

입니다.

---

# 40. 자동재시작 System Test

10일차에서 배운 내용을 **System Test 항목으로 재확인**합니다.

먼저 Service:

```bash
systemctl status \
  subject13-edge.service \
  --no-pager
```

통제된 강제 종료:

```bash
sudo systemctl kill \
  -s SIGKILL \
  subject13-edge.service
```

잠시 기다립니다.

```bash
sleep 5
```

상태:

```bash
systemctl status \
  subject13-edge.service \
  --no-pager
```

Journal:

```bash
journalctl \
  -u subject13-edge.service \
  -n 50 \
  --no-pager
```

예상 흐름:

```text
Process 비정상 종료
        ↓
systemd 실패 감지
        ↓
Restart=on-failure
        ↓
새 Process 시작
        ↓
Runtime State 다시 갱신
```

`RestartSec`가 짧으면 `/health`에서 `DEGRADED`가 눈에 보이기 전에 빠르게 정상 복귀할 수도 있습니다.

따라서 **DEGRADED 화면을 꼭 봐야 PASS인 것은 아닙니다.**

Journal과 Process 재시작 이력까지 함께 봅니다.

---

# 41. System Test 결과는 한 위치만 보고 판정하지 않습니다

```text
systemctl
→ 현재 Process 상태

journalctl
→ 과거 오류 / Restart 이력

runtime/edge_state.json
→ Edge Runtime이 기록한 최신 상태

/health
→ API가 해석한 Edge 생존 상태

/status
→ Prediction / Operational State

/metrics
→ 성능 / 자원 상태

GPIO
→ 실제 물리 출력

Operational AI Event Log
→ 최종 운영 State 변화의 시간 이력
→ 현재 System Test에서 새 Row가 생겼는지 확인
```

한 종류의 Evidence만 보고 전체 시스템을 PASS로 판정하지 않습니다.

---

# 42. System Test 오류를 찾는 기본 순서

```text
1. Power / 장치
        ↓
2. Network / SSH
        ↓
3. Service
        ↓
4. Journal
        ↓
5. Runtime State
        ↓
6. Camera / Model / Config
        ↓
7. Operational Event Log
        ↓
8. API
        ↓
9. GPIO
```

예를 들어 `/status`가 안 열린다고 바로 AI Model을 다시 학습하지 않습니다.

먼저 API Service와 Network부터 확인합니다.

---

# PART E. 한 번에 한 변수만 바꾸어 Trade-off 검증하기

# 43. 실험을 시작하기 전에 가장 중요한 규칙

오늘 실험은 다음 형식을 사용합니다.

```text
가설
        ↓
변경 변수
→ 딱 1개
        ↓
고정 변수
→ 나머지 조건
        ↓
측정 지표
        ↓
결과
        ↓
해석
```

잘못된 예:

```text
Threshold 변경
+
Input Size 변경
+
Frame Skip 변경
        ↓
결과 변화
        ↓
무엇 때문인지 알기 어려움
```

오늘은 가능한 한 **한 번에 한 변수만** 바꿉니다.

---

# 44. Validation과 Test를 구분하기

이번 날 가장 중요한 실험 원칙 중 하나입니다.

```text
Train
→ Model Weight 학습

Validation
→ Threshold / Input Size / 운영 설정 후보 비교

Test
→ 선택이 끝난 뒤 마지막 확인
```

Threshold `0.50 / 0.70 / 0.90`을 Test Dataset에서 여러 번 비교하고 가장 좋은 값을 고르면, Test Dataset을 사실상 설정 선택에 사용한 것이 됩니다.

따라서 오늘의 **후보 선택은 Validation Dataset**으로 진행합니다.

```text
Validation
→ 여러 후보 비교
→ 선택

그 뒤

Test
→ 선택한 조건을 한 번 최종 확인
```

Test 결과를 보고 다시 Threshold를 계속 바꾸기 시작하면 Test가 다시 Validation 역할을 하게 됩니다.

---

# 45. Validation / Test Dataset 경로 확인하기

PC에서:

```bash
find data/dataset \
  -maxdepth 2 \
  -type d \
  | sort
```

예상 구조:

```text
data/dataset/
├── train/
├── val/
└── test/
```

Class:

```bash
find data/dataset/val \
  -maxdepth 1 \
  -mindepth 1 \
  -type d \
  | sort
```

Dataset이 없다면 오늘 코드를 임의의 빈 폴더로 실행하지 않습니다.

6일차 Dataset 위치를 먼저 확인합니다.

---

# 46. Experiment 01 — Confidence Threshold만 비교하기

변경 변수:

```text
Confidence Threshold
```

후보:

```text
0.50
0.70
0.90
```

고정:

```text
같은 ONNX Model
같은 Validation Dataset
같은 Input Size
같은 Preprocess
같은 warning_class
```

측정:

```text
Operational Accuracy
False Warning
Missed Warning
```

---

# 47. Operational Threshold 평가 프로그램 만들기

## 이 파일은 왜 필요할까요?

`evaluate_onnx.py`는 Class Accuracy를 확인합니다.

하지만 운영에서는 다음처럼 Class Prediction에 Threshold 정책이 추가됩니다.

```text
Class
+
Confidence
+
Threshold
        ↓
NORMAL / WARNING
```

따라서 Threshold별로 **운영 State 기준** 결과를 비교하는 프로그램이 필요합니다.

오늘 후보 선택의 기본 Dataset은 `data/dataset/val`입니다.

## 의사코드

```text
Model / Metadata / Dataset / Threshold 받기
        ↓
Dataset 존재 확인
        ↓
ONNXClassifier 생성
        ↓
Metadata의 Class 순서 확인
        ↓
각 Class Folder의 이미지 순회
        ↓
Image → Tensor
        ↓
ONNX Prediction
        ↓
Operational Decision
        ↓
Expected State와 비교
        ↓
Accuracy
False Warning
Missed Warning
계산
        ↓
결과 출력
```

파일:

```text
scripts/evaluate_operational_threshold.py
```

코드:

```python
import argparse
from pathlib import Path

from PIL import Image

from src.ai_preprocess import (
    pil_to_nchw_float32,
)
from src.decision import (
    ai_to_operational_state,
)
from src.inference_onnx import (
    ONNXClassifier,
)


IMAGE_SUFFIXES = {
    ".jpg",
    ".jpeg",
    ".png",
}


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--model",
        required=True,
    )

    parser.add_argument(
        "--meta",
        required=True,
    )

    parser.add_argument(
        "--dataset",
        default="data/dataset/val",
    )

    parser.add_argument(
        "--warning-class",
        default="warning_red",
    )

    parser.add_argument(
        "--threshold",
        type=float,
        required=True,
    )

    return parser.parse_args()


def iter_images(
    directory: Path,
):
    for path in sorted(
        directory.iterdir()
    ):
        if (
            path.is_file()
            and path.suffix.lower()
            in IMAGE_SUFFIXES
        ):
            yield path


def main():
    args = parse_args()

    if not (
        0.0 <= args.threshold <= 1.0
    ):
        raise ValueError(
            "threshold는 0.0~1.0 범위여야 합니다."
        )

    root = Path(
        args.dataset
    )

    if not root.exists():
        raise FileNotFoundError(
            f"Dataset이 없습니다: {root}"
        )

    classifier = ONNXClassifier(
        args.model,
        args.meta,
    )

    if (
        args.warning_class
        not in classifier.classes
    ):
        raise ValueError(
            "warning_class가 Model Class에 없습니다: "
            f"{args.warning_class}"
        )

    total = 0
    correct = 0

    false_warning = 0
    missed_warning = 0

    for class_name in classifier.classes:
        class_dir = (
            root / class_name
        )

        if not class_dir.exists():
            raise FileNotFoundError(
                "Class Folder가 없습니다: "
                f"{class_dir}"
            )

        expected_state = (
            "WARNING"
            if class_name
            == args.warning_class
            else "NORMAL"
        )

        for image_path in iter_images(
            class_dir
        ):
            with Image.open(
                image_path
            ) as image:
                tensor = (
                    pil_to_nchw_float32(
                        image,
                        classifier.image_size,
                    )
                )

            prediction = (
                classifier.predict(
                    tensor
                )
            )

            actual_state = (
                ai_to_operational_state(
                    predicted_class=
                        prediction[
                            "class_name"
                        ],
                    confidence=
                        prediction[
                            "confidence"
                        ],
                    warning_class=
                        args.warning_class,
                    confidence_threshold=
                        args.threshold,
                )
            )

            total += 1

            if (
                actual_state
                == expected_state
            ):
                correct += 1

            if (
                expected_state == "NORMAL"
                and actual_state
                == "WARNING"
            ):
                false_warning += 1

            if (
                expected_state == "WARNING"
                and actual_state
                == "NORMAL"
            ):
                missed_warning += 1

    if total == 0:
        raise ValueError(
            f"평가할 이미지가 없습니다: {root}"
        )

    accuracy = (
        correct / total
    )

    print(
        "=== Operational Threshold ==="
    )
    print(
        f"Dataset         : {root}"
    )
    print(
        f"Input Size      : "
        f"{classifier.image_size}"
    )
    print(
        f"Threshold       : "
        f"{args.threshold:.2f}"
    )
    print(
        f"Samples         : {total}"
    )
    print(
        f"Accuracy        : "
        f"{accuracy:.4f}"
    )
    print(
        f"False Warning   : "
        f"{false_warning}"
    )
    print(
        f"Missed Warning  : "
        f"{missed_warning}"
    )


if __name__ == "__main__":
    main()
```

---

# 48. Threshold 0.50 / 0.70 / 0.90 비교하기

PC에서 **같은 Validation Dataset**을 사용합니다.

```bash
python -m scripts.evaluate_operational_threshold \
  --model models/day07_tiny_cnn.onnx \
  --meta models/day07_tiny_cnn.meta.json \
  --dataset data/dataset/val \
  --threshold 0.50
```

```bash
python -m scripts.evaluate_operational_threshold \
  --model models/day07_tiny_cnn.onnx \
  --meta models/day07_tiny_cnn.meta.json \
  --dataset data/dataset/val \
  --threshold 0.70
```

```bash
python -m scripts.evaluate_operational_threshold \
  --model models/day07_tiny_cnn.onnx \
  --meta models/day07_tiny_cnn.meta.json \
  --dataset data/dataset/val \
  --threshold 0.90
```

Report:

````md
## Experiment 01 — Confidence Threshold

### Hypothesis

Threshold를 높이면 낮은 Confidence의 WARNING이
NORMAL로 처리되는 경우가 늘 수 있다.

### Changed Variable

Confidence Threshold

### Fixed

Model:

Input Size:

Dataset: data/dataset/val

Preprocess:

Warning Class:

### Result

| Threshold | Val Operational Accuracy | False Warning | Missed Warning |
|---:|---:|---:|---:|
| 0.50 |  |  |  |
| 0.70 |  |  |  |
| 0.90 |  |  |  |

### Interpretation

False Warning이 가장 적은 값:

Missed Warning이 가장 적은 값:

Validation 기준 후보:

이유:
````

### Trade-off

Threshold를 높이면:

```text
애매한 WARNING
→ NORMAL로 처리되는 경우 증가 가능
        ↓
False Warning 감소 가능
하지만
Missed Warning 증가 가능
```

Threshold를 낮추면 반대 경향이 나타날 수 있습니다.

실제 Dataset 결과로 판단합니다.

---

# 49. Experiment 01 결과를 코드에서 역추적하기

예를 들어 다음이 출력되었다고 가정합니다.

```text
Threshold       : 0.70
Samples         : 40
Accuracy        : 0.9000
False Warning   : 2
Missed Warning  : 2
```

각 값의 위치를 찾습니다.

```text
Threshold
→ args.threshold

Samples
→ total

Accuracy
→ correct / total

False Warning
→ expected NORMAL + actual WARNING

Missed Warning
→ expected WARNING + actual NORMAL
```

숫자가 어디서 만들어졌는지 설명할 수 있어야 실험 결과를 믿고 사용할 수 있습니다.

---

# 50. Experiment 02 — Input Size 비교는 8일차 결과를 재사용합니다

8일차에서 다음 ONNX Variant를 이미 만들었습니다.

```text
models/day08_tiny_cnn_96.onnx
models/day08_tiny_cnn_96.meta.json

models/day08_tiny_cnn_64.onnx
models/day08_tiny_cnn_64.meta.json
```

두 Variant는 8일차 설계에서 **같은 Tiny CNN Weight를 사용하고 입력 Shape를 다르게 Export한 결과**입니다.

11일차에는 다시 Export하지 않습니다.

오늘은 기존 파일을 이용해:

```text
Validation Accuracy
+
Raspberry Pi Model-only Latency
```

를 한 표에서 다시 확인합니다.

---

# 51. 96×96 / 64×64 Validation Accuracy 확인하기

이전 일차의 `scripts/evaluate_onnx.py`를 다시 사용합니다.

96:

```bash
python -m scripts.evaluate_onnx \
  --model models/day08_tiny_cnn_96.onnx \
  --meta models/day08_tiny_cnn_96.meta.json \
  --dataset data/dataset/val
```

64:

```bash
python -m scripts.evaluate_onnx \
  --model models/day08_tiny_cnn_64.onnx \
  --meta models/day08_tiny_cnn_64.meta.json \
  --dataset data/dataset/val
```

정확한 Accuracy는 Dataset에 따라 달라집니다.

---

# 52. Raspberry Pi Model-only Benchmark 전에 Service 영향 제거하기

여기서 중요한 운영 문제가 하나 있습니다.

10일차의 `subject13-edge.service`가 계속 Camera와 ONNX 추론을 수행 중이면 Raspberry Pi CPU를 사용합니다.

그 상태에서 Model-only Benchmark를 실행하면 비교 조건에 불필요한 Background 부하가 들어갈 수 있습니다.

따라서 이번 **성능 비교 구간에서만** Edge Service를 의도적으로 중지합니다.

```bash
sudo systemctl stop \
  subject13-edge.service
```

상태:

```bash
systemctl status \
  subject13-edge.service \
  --no-pager
```

의도적인 `stop`은 장애가 아니며 `Restart=on-failure`가 다시 시작해야 하는 상황도 아닙니다.

API Service가 켜져 있다면 Edge State가 시간이 지나 `stale`이 될 수 있습니다. 이것도 현재 실험에서는 예상 가능한 현상입니다.

---

# 53. Raspberry Pi에서 96×96 / 64×64 Model-only Benchmark

8일차의 기존 도구를 재사용합니다.

96:

```bash
python -m scripts.benchmark_model_only \
  --image data/deploy_samples/normal.jpg \
  --model models/day08_tiny_cnn_96.onnx \
  --meta models/day08_tiny_cnn_96.meta.json \
  --label day11_pi_96
```

64:

```bash
python -m scripts.benchmark_model_only \
  --image data/deploy_samples/normal.jpg \
  --model models/day08_tiny_cnn_64.onnx \
  --meta models/day08_tiny_cnn_64.meta.json \
  --label day11_pi_64
```

다음 값을 기록합니다.

```text
Mean
P50
P95
FPS*
```

`FPS*`는 Model-only Latency로 환산한 추론 처리율 성격의 값입니다. Camera End-to-End FPS와 같은 값으로 해석하지 않습니다.

Report:

````md
## Experiment 02 — Input Size

### Changed Variable

ONNX Input Size

### Fixed

Weight:

Validation Dataset:

Raspberry Pi:

Benchmark Image:

Warm-up:

Repeat:

### Result

| Input | Val Accuracy | Mean ms | P95 ms | Model-only FPS* |
|---:|---:|---:|---:|---:|
| 96 × 96 |  |  |  |  |
| 64 × 64 |  |  |  |  |

### Interpretation

가장 빠른 설정:

Validation Accuracy가 높은 설정:

운영 후보:

이유:
````

---

# 54. Experiment 02가 끝나면 Edge Service 다시 시작하기

```bash
sudo systemctl start \
  subject13-edge.service
```

확인:

```bash
systemctl status \
  subject13-edge.service \
  --no-pager
```

API:

```bash
curl \
  http://127.0.0.1:<API_PORT>/health
```

`stale: false`와 정상 Heartbeat로 돌아왔는지 확인합니다.

---

# 55. Experiment 03 — Hold 시간만 비교하기

세 번째 실험은 출력 안정성을 봅니다.

변경 변수:

```text
warning_hold_sec
```

후보:

```text
0.0
1.0
2.0
```

고정:

```text
같은 Model
같은 Confidence Threshold
같은 Frame Skip
같은 Vote Window
같은 Warning Minimum
같은 Debounce
같은 Camera 위치
같은 조명
같은 60초 Protocol
```

측정:

```text
Output State Changes
체감 WARNING 안정성
NORMAL 복귀 지연
```

---

# 56. Camera를 직접 여는 실험 전에 Edge Service를 중지하기

이 단계는 매우 중요합니다.

10일차 `subject13-edge.service`도 Camera를 열고 있습니다.

동시에 `optimized_ai_warning_system.py`를 실행하면 두 Process가 같은 Camera를 점유하려고 하여 다음 문제가 생길 수 있습니다.

```text
Camera Open Fail

Frame Read Fail

두 Process가 서로 다른 상태로 GPIO 제어
```

따라서 수동 Camera 프로그램을 실행할 때는 먼저 Edge Service를 의도적으로 중지합니다.

```bash
sudo systemctl stop \
  subject13-edge.service
```

확인:

```bash
systemctl status \
  subject13-edge.service \
  --no-pager
```

---

# 57. Hold 실험 전 Baseline Config 백업하기

실험 후 정확히 복원하기 위해 현재 파일을 백업합니다.

```bash
cp \
  configs/settings.yaml \
  /tmp/day11_settings_before_hold.yaml
```

현재 고정 조건을 기록합니다.

```bash
grep -E \
"inference_every_n_frames|vote_window|vote_warning_min|debounce_required|warning_hold_sec|ai_confidence_threshold" \
configs/settings.yaml
```

---

# 58. Hold 0초 실험하기

`configs/settings.yaml`에서 **이 값만** 변경합니다.

```yaml
warning_hold_sec: 0.0
```

실행:

```bash
python -m scripts.optimized_ai_warning_system \
  --run-name day11_hold0 \
  --duration 60
```

60초 Protocol:

```text
NORMAL 15초
        ↓
경계 상태 15초
        ↓
명확한 WARNING 15초
        ↓
NORMAL 복귀 15초
```

Run Summary의 `Output Changes`를 기록합니다.

---

# 59. Hold 1초 실험하기

설정:

```yaml
warning_hold_sec: 1.0
```

다른 값은 바꾸지 않습니다.

실행:

```bash
python -m scripts.optimized_ai_warning_system \
  --run-name day11_hold1 \
  --duration 60
```

같은 60초 Protocol을 최대한 반복합니다.

---

# 60. Hold 2초 실험하기

설정:

```yaml
warning_hold_sec: 2.0
```

실행:

```bash
python -m scripts.optimized_ai_warning_system \
  --run-name day11_hold2 \
  --duration 60
```

---

# 61. Hold 결과표 작성하기

````md
## Experiment 03 — Warning Hold

### Hypothesis

Hold가 길어지면 WARNING 출력 흔들림은 줄어들 수 있지만
NORMAL 복귀는 늦어질 수 있다.

### Changed Variable

warning_hold_sec

### Fixed

Frame Skip:

Vote Window:

Warning Minimum:

Debounce:

Confidence Threshold:

Camera 위치:

### Result

| Hold | Output Changes | WARNING 안정성 | NORMAL 복귀 지연 | 관찰 |
|---:|---:|---|---|---|
| 0 sec |  |  |  |  |
| 1 sec |  |  |  |  |
| 2 sec |  |  |  |  |

### Interpretation

가장 안정적인 값:

가장 빠르게 복귀한 값:

운영 후보:

이유:
````

사람이 느낀 반응을 `137 ms`처럼 정밀한 계측값으로 적지 않습니다.

정밀한 반응시간을 측정하려면 별도의 이벤트 기준과 측정 설계가 필요합니다.

---

# 62. Hold 실험 후 Config와 Service 복원하기

백업한 Config를 복원합니다.

```bash
cp \
  /tmp/day11_settings_before_hold.yaml \
  configs/settings.yaml
```

값 확인:

```bash
grep -E \
"inference_every_n_frames|vote_window|vote_warning_min|debounce_required|warning_hold_sec|ai_confidence_threshold" \
configs/settings.yaml
```

Edge Service:

```bash
sudo systemctl start \
  subject13-edge.service
```

확인:

```bash
systemctl status \
  subject13-edge.service \
  --no-pager
```

API:

```bash
curl \
  http://127.0.0.1:<API_PORT>/health
```

---

# 63. Validation으로 선택한 후보를 Test Dataset에서 마지막 확인하기

지금까지 Threshold와 Input Size 같은 **후보 선택은 Validation Dataset으로 수행했습니다.** 이제 Validation 결과를 보고 최종 후보를 하나 선택한 뒤, 그 조합만 Test Dataset에서 한 번 확인합니다.

먼저 실제 선택값을 적습니다.

```text
<SELECTED_MODEL>
→ Validation 비교에서 선택한 ONNX 모델 경로

<SELECTED_META>
→ 위 모델과 짝이 되는 Metadata 경로

<SELECTED_THRESHOLD>
→ Validation 비교에서 선택한 운영 Threshold
```

예를 들어 `64 × 64` 모델이 빠르다는 이유만으로 무조건 선택하거나, 교재 예시인 `0.70`을 그대로 정답으로 사용하지 않습니다. **앞에서 기록한 Validation 결과를 근거로 자신의 최종 조합을 정합니다.**

최종 운영 상태를 Test Dataset에서 확인합니다.

```bash
python -m scripts.evaluate_operational_threshold \
  --model <SELECTED_MODEL> \
  --meta <SELECTED_META> \
  --dataset data/dataset/test \
  --threshold <SELECTED_THRESHOLD>
```

같은 선택 모델의 Class Accuracy와 Confusion Matrix도 확인하려면 다음처럼 실행합니다.

```bash
python -m scripts.evaluate_onnx \
  --model <SELECTED_MODEL> \
  --meta <SELECTED_META> \
  --dataset data/dataset/test
```

실제 명령을 입력할 때 `<SELECTED_...>` 문자열을 그대로 쓰지 않고 자신이 선택한 경로와 값으로 바꿉니다.

예를 들어 Validation에서 다음 조합을 선택했다고 가정합니다.

```text
Model     : models/day08_tiny_cnn_64.onnx
Metadata  : models/day08_tiny_cnn_64.meta.json
Threshold : 0.70
```

그때만 실제 명령은 다음처럼 됩니다.

```bash
python -m scripts.evaluate_operational_threshold \
  --model models/day08_tiny_cnn_64.onnx \
  --meta models/day08_tiny_cnn_64.meta.json \
  --dataset data/dataset/test \
  --threshold 0.70
```

이 예시는 **명령 형식을 보여주기 위한 예**입니다. `64`나 `0.70`이 모든 학생의 정답이라는 뜻이 아닙니다.

### 왜 Test 결과를 보고 다시 설정을 고르면 안 될까요?

```text
Train
→ 모델 학습

Validation
→ Threshold / Input Size 같은 후보 선택

Test
→ 선택이 끝난 뒤 마지막 일반화 확인
```

Test 결과를 본 뒤 다시 Threshold를 고르고, 다시 Test하고, 또 고르는 과정을 반복하면 Test Dataset이 사실상 설정 선택에 사용됩니다. 그러면 마지막 성능을 처음 보는 데이터에 대한 공정한 평가라고 보기 어려워집니다.

따라서 오늘은 다음 순서를 지킵니다.

```text
Validation에서 후보 선택
        ↓
선택값 고정
        ↓
Test 1회 최종 확인
        ↓
결과 기록
        ↓
Test 결과를 보고 다시 후보 탐색하지 않음
```

Test 결과가 기대보다 낮더라도 숨기지 않습니다. 그 자체가 다음 개선에서 사용할 중요한 근거입니다.

---

# 64. Overall Trade-off 표 작성하기

`reports/day11_test_analysis.md`에 오늘 결과를 한 표로 모읍니다.

````md
## Overall Trade-off

| Candidate | Validation Accuracy | Test Accuracy* | False Warning | Missed Warning | Mean Latency | P95 | FPS* | Output Stability |
|---|---:|---:|---:|---:|---:|---:|---:|---|
| Day10 Baseline |  | N/A |  |  |  |  |  |  |
| Threshold 후보 |  |  |  |  | N/A | N/A | N/A | N/A |
| Input Size 후보 |  |  | N/A | N/A |  |  |  | N/A |
| Hold 후보 | N/A | N/A | N/A | N/A | N/A | N/A | N/A |  |
````

`*`는 실제로 최종 Test를 수행한 후보에만 값을 적습니다.

측정하지 않은 값은:

```text
N/A
```

로 기록합니다.

값을 추측해서 채우지 않습니다.

---

# 65. 좋은 Edge AI를 한 숫자로 판단하지 않기

예:

```text
Candidate A
→ Accuracy 높음
→ Latency 느림

Candidate B
→ Accuracy 조금 낮음
→ Latency 빠름

Candidate C
→ WARNING 출력 안정적
→ NORMAL 복귀 늦음
```

어느 하나가 항상 정답은 아닙니다.

현장 요구가:

```text
WARNING을 놓치면 안 됨
```

이라면 `Missed Warning`을 더 중요하게 볼 수 있습니다.

반대로:

```text
잘못된 WARNING이 발생할 때마다
라인이 멈춤
```

이라면 `False Warning`도 매우 중요합니다.

오늘은 숫자 하나가 아니라 **요구조건과 함께 선택 이유를 설명**합니다.

---

# PART F. 실패 조건을 수집하고 Root Cause를 분류하기

# 66. Failure Analysis에서는 무엇을 하나요?

System Test에서 실패가 보이면 바로 Model을 다시 학습하지 않습니다.

먼저 실패를 다음 영역으로 나눕니다.

```text
Input
→ 조명 / 배경 / 거리 / Blur / Camera

Data / Model
→ 학습 분포 / Class 혼동 / Confidence 경계

Runtime
→ Model Path / Metadata / Shape / Preprocess

Performance
→ CPU / Memory / Temperature / Latency

Operation
→ systemd / FastAPI / 권한 / Port / Restart / Network
```

같은 `WARNING을 놓침`이라는 현상도 원인이 다를 수 있습니다.

```text
너무 어두움
→ Input

먼 거리 Data 부족
→ Data

Confidence 0.69
Threshold 0.70
→ Operational Policy

잘못된 Model Path
→ Runtime

CPU 과부하
→ Performance
```

먼저 영역을 나눈 뒤 수정 대상을 정합니다.

---

# 67. Failure Image를 저장할 위치 준비하기

실패 이미지는 Source Code가 아닙니다.

기존 `data/` 아래에 저장합니다.

```text
data/day11_failures/
├── images/
└── manifest.csv
```

`data/`는 기존 `.gitignore`에서 제외되어 있어야 합니다.

확인:

```bash
git check-ignore \
  data/day11_failures/test.jpg
```

정상이라면 해당 경로가 Ignore 대상이라는 사실을 확인할 수 있습니다.

---

# 68. Failure Case 촬영 전에 Edge Service를 중지하기

이 단계도 Hold 실험과 같은 이유로 중요합니다.

`subject13-edge.service`가 Camera를 계속 사용하고 있는 상태에서 `capture_failure_case.py`가 같은 Camera를 열면 충돌할 수 있습니다.

따라서 Failure Image 수집 동안에는 Edge Service를 의도적으로 중지합니다.

```bash
sudo systemctl stop \
  subject13-edge.service
```

확인:

```bash
systemctl status \
  subject13-edge.service \
  --no-pager
```

API Service가 살아 있다면 `/health`는 시간이 지나 `DEGRADED` 또는 `stale: true`가 될 수 있습니다.

현재는 **의도적으로 Edge를 중지한 수집 작업**이므로 예상 가능한 상태입니다.

---

## Failure Capture를 어디에서 작성하고 실행할까요?

`capture_failure_case.py`와 뒤의 `analyze_failure_cases.py`는 Source 파일이므로 **주 작업 사본에서 작성한 뒤 Raspberry Pi와 동기화**할 수 있습니다. 하지만 실제 Failure Image 촬영은 Raspberry Pi의 Camera를 사용하므로 Raspberry Pi에서 실행합니다.

```text
PC 주 작업 사본
→ Failure Script 작성
→ Local Git / 내부 전달로 Pi 동기화
        ↓
Raspberry Pi
→ Camera를 사용하는 Failure Capture 실행
→ data/day11_failures/ 생성
```

VS Code Remote SSH로 Raspberry Pi를 주 작업 사본으로 사용하는 수업이라면 Pi에서 바로 작성해도 됩니다. 어느 방식을 사용하든 같은 Source 파일을 PC와 Pi에서 따로 수정하여 서로 다른 버전을 만들지 않습니다.

# 69. Failure Case 수집 프로그램 만들기

## 이 파일은 왜 필요할까요?

실패 상황을 말로만 적으면 나중에 같은 조건을 다시 보기 어렵습니다.

따라서 다음 두 가지를 함께 저장합니다.

```text
실제 Image
+
그 Image가 어떤 조건에서 만들어졌는지 Manifest
```

Manifest에는 최소한 다음 정보를 남깁니다.

```text
timestamp
category
case_name
expected_state
image_path
```

## 의사코드

```text
category / case / expected / delay 받기
        ↓
설정파일 읽기
        ↓
Camera index / width / height 확인
        ↓
Camera 열기
        ↓
정해진 시간 대기
        ↓
Frame 한 장 읽기
        ↓
조건 이름이 들어간 파일명으로 저장
        ↓
manifest.csv가 처음이면 Header 작성
        ↓
촬영 정보 한 줄 추가
        ↓
Camera 정리
```

파일:

```text
scripts/capture_failure_case.py
```

코드:

```python
import argparse
import csv
import time
from datetime import datetime
from pathlib import Path

from src.camera_input import (
    CameraInput,
    save_frame,
)
from src.config_loader import load_config


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--category",
        required=True,
        choices=[
            "input",
            "data_model",
            "runtime",
            "performance",
            "operation",
        ],
    )

    parser.add_argument(
        "--case",
        required=True,
    )

    parser.add_argument(
        "--expected",
        required=True,
        choices=[
            "NORMAL",
            "WARNING",
        ],
    )

    parser.add_argument(
        "--delay",
        type=float,
        default=3.0,
    )

    return parser.parse_args()


def main():
    args = parse_args()

    if args.delay < 0:
        raise ValueError(
            "delay는 0 이상이어야 합니다."
        )

    config = load_config(
        "configs/settings.yaml"
    )

    root = Path(
        "data/day11_failures"
    )

    image_dir = (
        root / "images"
    )

    manifest_path = (
        root / "manifest.csv"
    )

    camera = CameraInput(
        index=int(
            config["camera_index"]
        ),
        width=int(
            config["image_width"]
        ),
        height=int(
            config["image_height"]
        ),
    )

    try:
        print(
            f"{args.delay:.1f}초 후 "
            "Failure Case를 촬영합니다."
        )

        time.sleep(
            args.delay
        )

        frame = camera.read()

        prefix = (
            f"{args.category}"
            f"__{args.case}"
            f"__{args.expected.lower()}"
        )

        image_path = save_frame(
            frame,
            str(image_dir),
            prefix=prefix,
        )

        is_new = (
            not manifest_path.exists()
        )

        manifest_path.parent.mkdir(
            parents=True,
            exist_ok=True,
        )

        with manifest_path.open(
            "a",
            newline="",
            encoding="utf-8",
        ) as file:
            writer = csv.writer(file)

            if is_new:
                writer.writerow(
                    [
                        "timestamp",
                        "category",
                        "case_name",
                        "expected_state",
                        "image_path",
                    ]
                )

            writer.writerow(
                [
                    datetime.now().isoformat(
                        timespec="milliseconds"
                    ),
                    args.category,
                    args.case,
                    args.expected,
                    str(image_path),
                ]
            )

        print(
            f"Saved: {image_path}"
        )
        print(
            f"Manifest: {manifest_path}"
        )

    finally:
        camera.release()


if __name__ == "__main__":
    main()
```

---

# 70. Failure Capture 코드를 실행하기 전에 연결 확인하기

```text
configs/settings.yaml
configs/device.local.yaml
        ↓
config_loader.py
        ↓
capture_failure_case.py
        ↓
CameraInput
        ↓
Frame
        ↓
save_frame()
        ↓
data/day11_failures/images/*.jpg

동시에

capture_failure_case.py
        ↓
CSV Writer
        ↓
data/day11_failures/manifest.csv
```

실제 Camera index를 코드에 직접 넣지 않고 기존 Config를 읽습니다.

---

# 71. Failure Case 1 — 어두운 조명

조건:

```text
경고 대상 있음
+
조명이 어두움
```

Expected:

```text
WARNING
```

실행:

```bash
python -m scripts.capture_failure_case \
  --category input \
  --case dark_lighting \
  --expected WARNING
```

촬영 직전 실제 조건을 만들고 기다립니다.

---

# 72. Failure Case 2 — 빨간색 배경

조건:

```text
경고 대상 없음
+
배경에 큰 빨간 영역
```

Expected:

```text
NORMAL
```

실행:

```bash
python -m scripts.capture_failure_case \
  --category input \
  --case red_background \
  --expected NORMAL
```

---

# 73. Failure Case 3 — 경고 물체가 너무 멀리 있음

조건:

```text
경고 물체 있음
+
Camera에서 매우 작게 보임
```

Expected:

```text
WARNING
```

실행:

```bash
python -m scripts.capture_failure_case \
  --category input \
  --case far_object \
  --expected WARNING
```

---

# 74. Failure Case 4 — 빠른 움직임과 Blur

경고 물체를 빠르게 움직여 흐릿한 순간을 만듭니다.

```bash
python -m scripts.capture_failure_case \
  --category input \
  --case motion_blur \
  --expected WARNING
```

한 번에 원하는 Blur가 잡히지 않을 수 있습니다.

필요하면 같은 조건을 여러 번 촬영하고 나중에 대표 이미지를 선택합니다.

---

# 75. Failure Case 5 — Confidence 경계 상황

경고 물체를 일부만 보여 주거나 거리·각도를 조절합니다.

```bash
python -m scripts.capture_failure_case \
  --category data_model \
  --case confidence_boundary \
  --expected WARNING
```

목표는 **운영 Threshold 근처의 Confidence가 나오는 어려운 사례**를 확보하는 것입니다.

정확히 경계값을 맞추기 어렵다면 가장 가까운 사례를 사용합니다.

---

# 76. Failure Manifest와 Image 확인하기

Manifest:

```bash
cat \
  data/day11_failures/manifest.csv
```

Image:

```bash
find \
  data/day11_failures/images \
  -type f \
  -name "*.jpg" \
  | sort
```

최소 다섯 Case가 있는지 확인합니다.

### 코드 리뷰 — Saved 경로는 어디서 만들어졌을까요?

```text
case 이름
→ parse_args()

prefix
→ capture_failure_case.py

실제 파일명
→ save_frame()

image_path
→ save_frame() 반환값

Manifest의 image_path
→ writer.writerow()
```

한 장의 Failure Image와 Manifest 한 줄이 같은 경로로 연결되는지 확인합니다.

---

# 77. Failure Image를 AI로 다시 분석하는 프로그램 만들기

## 이 파일은 왜 필요할까요?

촬영 시점에는 Expected State만 적었습니다.

이제 저장된 Image를 현재 ONNX Model과 운영 Threshold로 다시 실행합니다.

```text
Manifest
        ↓
Image Path
        ↓
Image
        ↓
Preprocess
        ↓
ONNX Prediction
        ↓
Operational Decision
        ↓
Expected vs Actual
        ↓
correct
        ↓
reports/day11_failure_analysis.csv
```

## 의사코드

```text
Config 읽기
        ↓
Manifest 존재 확인
        ↓
ONNX Model / Metadata 준비
        ↓
Threshold / Warning Class 읽기
        ↓
Manifest 한 줄씩 읽기
        ↓
Image Path 존재 확인
        ↓
Image → Tensor
        ↓
ONNX Prediction
        ↓
Operational State
        ↓
Expected와 비교
        ↓
Result 목록에 저장
        ↓
Analysis CSV 작성
        ↓
터미널에도 한 줄씩 출력
```

파일:

```text
scripts/analyze_failure_cases.py
```

코드:

```python
import csv
from pathlib import Path

from PIL import Image

from src.ai_preprocess import (
    pil_to_nchw_float32,
)
from src.config_loader import load_config
from src.decision import (
    ai_to_operational_state,
)
from src.inference_onnx import (
    ONNXClassifier,
)


def main():
    config = load_config(
        "configs/settings.yaml"
    )

    root = Path(
        "data/day11_failures"
    )

    manifest_path = (
        root / "manifest.csv"
    )

    output_path = Path(
        "reports/"
        "day11_failure_analysis.csv"
    )

    if not manifest_path.exists():
        raise FileNotFoundError(
            "Failure Manifest가 없습니다: "
            f"{manifest_path}"
        )

    classifier = ONNXClassifier(
        config["onnx_model_path"],
        config["onnx_meta_path"],
    )

    threshold = float(
        config[
            "ai_confidence_threshold"
        ]
    )

    warning_class = (
        config["ai_warning_class"]
    )

    results = []

    with manifest_path.open(
        "r",
        newline="",
        encoding="utf-8",
    ) as file:
        reader = csv.DictReader(
            file
        )

        for row in reader:
            image_path = Path(
                row["image_path"]
            )

            if not image_path.exists():
                raise FileNotFoundError(
                    "Failure Image가 없습니다: "
                    f"{image_path}"
                )

            with Image.open(
                image_path
            ) as image:
                tensor = (
                    pil_to_nchw_float32(
                        image,
                        classifier.image_size,
                    )
                )

            prediction = (
                classifier.predict(
                    tensor
                )
            )

            actual_state = (
                ai_to_operational_state(
                    predicted_class=
                        prediction[
                            "class_name"
                        ],
                    confidence=
                        prediction[
                            "confidence"
                        ],
                    warning_class=
                        warning_class,
                    confidence_threshold=
                        threshold,
                )
            )

            expected_state = (
                row["expected_state"]
            )

            correct = (
                actual_state
                == expected_state
            )

            results.append(
                {
                    **row,
                    "pred_class":
                        prediction[
                            "class_name"
                        ],
                    "confidence":
                        float(
                            prediction[
                                "confidence"
                            ]
                        ),
                    "actual_state":
                        actual_state,
                    "correct":
                        correct,
                }
            )

    if not results:
        raise ValueError(
            "분석할 Failure Case가 없습니다."
        )

    output_path.parent.mkdir(
        parents=True,
        exist_ok=True,
    )

    fieldnames = [
        "timestamp",
        "category",
        "case_name",
        "expected_state",
        "pred_class",
        "confidence",
        "actual_state",
        "correct",
        "image_path",
    ]

    with output_path.open(
        "w",
        newline="",
        encoding="utf-8",
    ) as file:
        writer = csv.DictWriter(
            file,
            fieldnames=fieldnames,
        )

        writer.writeheader()

        for row in results:
            writer.writerow(
                {
                    **row,
                    "confidence":
                        f"{row['confidence']:.6f}",
                }
            )

    print(
        "=== Failure Analysis ==="
    )

    for row in results:
        print(
            f"{row['case_name']:22s} "
            f"expected="
            f"{row['expected_state']:7s} "
            f"pred="
            f"{row['pred_class']:12s} "
            f"conf="
            f"{row['confidence']:.3f} "
            f"state="
            f"{row['actual_state']:7s} "
            f"correct="
            f"{row['correct']}"
        )

    print()
    print(
        f"Saved: {output_path}"
    )


if __name__ == "__main__":
    main()
```

---

# 78. Failure Analysis 실행하기

Raspberry Pi 또는 동일한 Model과 Failure Dataset이 있는 PC에서:

```bash
python -m scripts.analyze_failure_cases
```

예상 형태:

```text
=== Failure Analysis ===
dark_lighting          expected=WARNING pred=... conf=... state=... correct=...
red_background         expected=NORMAL  pred=... conf=... state=... correct=...
...

Saved: reports/day11_failure_analysis.csv
```

정확한 Prediction은 실제 Image에 따라 달라집니다.

CSV:

```bash
cat \
  reports/day11_failure_analysis.csv
```

---

# 79. Failure Analysis 결과를 코드에서 역추적하기

CSV 한 줄을 예로 봅니다.

```text
case_name
expected_state
pred_class
confidence
actual_state
correct
image_path
```

각 값의 출처:

```text
case_name / expected_state / image_path
→ manifest.csv

pred_class / confidence
→ ONNXClassifier.predict()

actual_state
→ ai_to_operational_state()

correct
→ actual_state == expected_state

CSV 저장
→ csv.DictWriter
```

즉, `correct=False` 한 줄은 단순한 문자열이 아니라:

```text
실제 촬영조건
+
정답 의도
+
AI Prediction
+
운영 Decision
```

이 연결의 결과입니다.

---

# 80. Failure Root Cause 표 작성하기

`reports/day11_test_analysis.md`에 추가합니다.

````md
## Failure Analysis

### Image Failure Cases

| Case | Expected | Actual | Confidence | Primary Area | 원인 가설 | 개선 아이디어 |
|---|---|---|---:|---|---|---|
| dark_lighting |  |  |  | Input |  |  |
| red_background |  |  |  | Input |  |  |
| far_object |  |  |  | Input / Data |  |  |
| motion_blur |  |  |  | Input / Data |  |  |
| confidence_boundary |  |  |  | Data / Policy |  |  |

### Operation / Runtime Failure

| Failure | Area | 확인 위치 | 원인 | 복구 |
|---|---|---|---|---|
| Camera 없음 | Runtime / Input | journalctl |  |  |
| Model Path 오류 | Runtime | journalctl |  |  |
| Edge Process SIGKILL | Operation | systemd / journal |  |  |
| API Process SIGKILL | Operation | systemd / journal |  |  |
| API Port 오류 | Operation / Network | curl / ss |  |  |
````

10일차에서 이미 만든 장애 기록도 오늘 Root Cause 표에 재사용합니다.

같은 장애를 불필요하게 반복해서 만들 필요는 없습니다.

---

# 81. 실패 원인을 바로 Model 탓으로 돌리지 않기

예를 들어 `warning_red`를 놓쳤습니다.

가능한 원인:

```text
Image 자체가 너무 어두움
→ Input

학습 Data에 어두운 조건이 부족함
→ Data

Model이 다른 Class로 예측
→ Model

warning_red로 예측했지만 Confidence가 Threshold 아래
→ Operational Policy

잘못된 ONNX / Metadata 조합
→ Runtime

CPU 부하로 반응이 늦음
→ Performance
```

먼저 Evidence를 보고 영역을 정합니다.

---

# 82. 선택 복습 — Rule과 AI의 실패 조건 비교하기

5일차에서 Rule 기반 판단을 이미 배웠다면 다음 질문을 **비교 관점으로만** 다시 생각할 수 있습니다.

```text
빨간 배경
→ Rule은 무엇을 볼까?
→ AI는 무엇을 볼까?

어두운 조명
→ Rule Threshold는 어떻게 영향을 받을까?
→ AI Confidence는 어떻게 영향을 받을까?

먼 거리
→ Rule Feature는 충분할까?
→ AI가 학습한 분포에 있었을까?
```

오늘 새로운 Rule 프로그램을 만들지는 않습니다.

11일차의 핵심은 **현재 AI 시스템의 Test와 Failure Analysis**입니다.

---

# 83. Failure 수집이 끝나면 Edge Service 복원하기

```bash
sudo systemctl start \
  subject13-edge.service
```

상태:

```bash
systemctl status \
  subject13-edge.service \
  --no-pager
```

API:

```bash
curl \
  http://127.0.0.1:<API_PORT>/health
```

최종적으로 다시:

```text
edge: OK
stale: false
```

가 되는지 확인합니다.

---

# PART G. Mini Challenge — Test 결과를 스스로 해석하고 복구하기

# 84. Mini Challenge 시작 전 백업하기

Challenge에서 설정과 Smoke Test 코드를 바꿀 수 있으므로 먼저 현재 정상 상태를 백업합니다.

```bash
cp \
  configs/settings.yaml \
  /tmp/day11_settings_before_challenge.yaml
```

```bash
cp \
  scripts/api_smoke_test.py \
  /tmp/day11_api_smoke_before_challenge.py
```

현재 Unit Test도 확인합니다.

```bash
python -m pytest \
  tests \
  -q
```

Challenge에서는 정답을 먼저 열지 않습니다.

문제를 직접 생각하고 실행한 뒤 **예시 정답 확인해보기**를 열어 비교합니다.

---

## Challenge 1 — 실행 결과를 보고 코드 위치 찾기

다음 두 결과를 봅니다.

결과 A:

```text
/health   : PASS
/status   : PASS
/metrics  : PASS
```

결과 B:

```text
far_object
expected=WARNING
pred=normal
conf=0.611
state=NORMAL
correct=False
```

다음을 직접 찾습니다.

| 화면에 보인 값 | 담당 파일 | 담당 코드 / 함수 |
|---|---|---|
| `/health : PASS` |  |  |
| `pred=normal` |  |  |
| `conf=0.611` |  |  |
| `state=NORMAL` |  |  |
| `correct=False` |  |  |
| Failure CSV 저장 |  |  |

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

| 화면에 보인 값 | 담당 파일 | 담당 코드 / 함수 |
|---|---|---|
| `/health : PASS` | `scripts/api_smoke_test.py` | `check_keys()`의 `print()` |
| `pred=normal` | `src/inference_onnx.py` + `analyze_failure_cases.py` | `ONNXClassifier.predict()` 결과 |
| `conf=0.611` | `src/inference_onnx.py` | `prediction["confidence"]` |
| `state=NORMAL` | `src/decision.py` | `ai_to_operational_state()` |
| `correct=False` | `scripts/analyze_failure_cases.py` | `actual_state == expected_state` |
| Failure CSV 저장 | `scripts/analyze_failure_cases.py` | `csv.DictWriter` |

한 줄의 결과가 한 파일에서 모두 만들어지는 것이 아니라 여러 모듈을 지나 만들어집니다.

```text
Image
→ ONNX
→ Prediction
→ Decision
→ Expected와 비교
→ CSV
```

</details>

---

## Challenge 2 — 실행 전에 결과 예측하기

다음 Unit Test 조건을 실행하기 전에 결과를 예상합니다.

```text
warning_class = warning_red
confidence_threshold = 0.80
```

Case:

```text
A
predicted_class = warning_red
confidence = 0.79

B
predicted_class = warning_red
confidence = 0.80

C
predicted_class = normal
confidence = 0.99
```

그리고 Stabilizer:

```text
vote_window = 1
vote_warning_min = 1
debounce_required = 2
warning_hold_sec = 0

입력
WARNING
WARNING
```

질문:

```text
1. A의 State는?
2. B의 State는?
3. C의 State는?
4. Stabilizer 첫 WARNING 뒤 stable_state는?
5. 두 번째 WARNING 뒤 stable_state는?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
A
0.79 < 0.80
→ NORMAL

B
0.80 >= 0.80
→ WARNING

C
Class가 warning_red가 아님
→ NORMAL

Stabilizer 첫 WARNING
→ 후보 횟수 1
→ debounce_required=2 미충족
→ stable_state NORMAL

두 번째 WARNING
→ 후보 횟수 2
→ Debounce 충족
→ stable_state WARNING
```

Decision과 Stabilizer가 서로 다른 역할이라는 점이 핵심입니다.

</details>

---

## Challenge 3 — Python 코드는 바꾸지 않고 설정값만 비교하기

오늘 수집한 `confidence_boundary` Failure Image 중 하나를 선택합니다.

먼저 파일을 찾습니다.

```bash
find \
  data/day11_failures/images \
  -type f \
  -name "*confidence_boundary*.jpg" \
  | sort
```

선택한 경로를 `<BOUNDARY_IMAGE>`라고 합니다.

현재 설정:

```bash
grep -E \
  "^ai_confidence_threshold:" \
  configs/settings.yaml
```

### 문제

Python Source는 수정하지 않습니다.

`configs/settings.yaml`의 `ai_confidence_threshold`만 두 값으로 비교합니다.

예:

```text
0.60
0.85
```

각 값에서 다음을 실행합니다.

```bash
python -m scripts.onnx_decision_test \
  --image <BOUNDARY_IMAGE>
```

실행하기 전에 먼저 예측합니다.

```text
이 Image의 Confidence가 두 Threshold 사이에 있다면
State는 어떻게 달라질까?

Prediction Class 자체도 Threshold 때문에 바뀔까?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

예를 들어 같은 Image의 Prediction이:

```text
pred_class = warning_red
confidence = 0.72
```

라면:

```text
Threshold 0.60
0.72 >= 0.60
→ WARNING

Threshold 0.85
0.72 < 0.85
→ NORMAL
```

하지만 `pred_class`와 원래 Model Confidence는 Threshold 때문에 다시 학습되거나 바뀌는 값이 아닙니다.

```text
Model Prediction
→ 같은 Image / 같은 Model이면 기본적으로 같은 추론 결과

Operational State
→ Threshold에 따라 달라질 수 있음
```

실제 Confidence가 두 Threshold 밖에 있다면 State가 같을 수도 있습니다. 그 결과도 정상적인 실험 결과입니다.

</details>

Challenge가 끝나면 아직 복원하지 않아도 됩니다. 마지막 복원 절에서 한 번에 되돌립니다.

---

## Challenge 4 — 기존 API Smoke Test 기능을 조금 확장하기

현재 Smoke Test는 API 구조가 존재하는지 확인합니다.

이번에는 선택 기능을 하나 추가합니다.

요구사항:

```text
--require-edge-ok를 지정하지 않음
→ 기존처럼 API Key 구조만 확인

--require-edge-ok를 지정함
→ /health의 edge가 OK인지 확인
→ stale이 false인지 확인
→ 아니면 Test 실패
```

바로 정답을 보지 말고 먼저 어느 부분을 수정해야 하는지 생각합니다.

```text
argparse?
main()?
health Dictionary?
Assertion?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

`parse_args()`에 다음을 추가할 수 있습니다.

```python
parser.add_argument(
    "--require-edge-ok",
    action="store_true",
)
```

`main()`의 `/health` 확인 뒤에 다음을 추가할 수 있습니다.

```python
if args.require_edge_ok:
    if health["edge"] != "OK":
        raise AssertionError(
            "Edge Health가 OK가 아닙니다."
        )

    if bool(health["stale"]):
        raise AssertionError(
            "Runtime State가 stale입니다."
        )
```

실행:

```bash
python -m scripts.api_smoke_test \
  --host <RPI_IP> \
  --port <API_PORT> \
  --require-edge-ok
```

정상 Baseline이라면 기존 PASS 뒤에 오류 없이 종료되어야 합니다.

이 기능은 API 형식뿐 아니라 **현재 Edge Health까지 엄격하게 확인하고 싶을 때** 사용하는 선택 조건입니다.

</details>

---

## Challenge 5 — 일부러 잘못된 Port로 실패시키고 원인 찾기

실제 API Port를 먼저 확인합니다.

```bash
grep -E "^api_port:" configs/settings.yaml
```

그 값과 다른 사용하지 않는 Port 하나를 `<WRONG_PORT>`로 정합니다.

실행:

```bash
python -m scripts.api_smoke_test \
  --host <RPI_IP> \
  --port <WRONG_PORT>
```

질문:

```text
1. 어떤 종류의 오류가 보이는가?
2. Model 문제인가?
3. Camera 문제인가?
4. 실제 Listen Port는 어떻게 확인할 수 있는가?
5. 수정 후 어떤 명령으로 재실행할 것인가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

잘못된 Port에 아무 Server도 없다면 환경에 따라 다음과 비슷한 연결 오류가 발생할 수 있습니다.

```text
ConnectionRefusedError
URLError
timed out
```

이 문제는 우선 AI Model 문제가 아닙니다.

확인:

```bash
ss -ltnp | grep ":<API_PORT>"
```

API Service:

```bash
systemctl status \
  subject13-api.service \
  --no-pager
```

수정:

```text
잘못된 Port
→ configs/settings.yaml에서 확인한 실제 Port
```

재실행:

```bash
python -m scripts.api_smoke_test \
  --host <RPI_IP> \
  --port <API_PORT>
```

정상 PASS가 다시 나오면 복구를 증명한 것입니다.

</details>

---

## Challenge 6 — Result File과 Log로 정상 동작 증명하기

이번에는 “화면에서 됐다”라고 말하지 않고 파일을 남깁니다.

Evidence 폴더:

```bash
mkdir -p \
  reports/day11_evidence
```

API Smoke 결과 저장:

```bash
python -m scripts.api_smoke_test \
  --host <RPI_IP> \
  --port <API_PORT> \
  | tee \
  reports/day11_evidence/api_smoke.txt
```

Unit Test 결과 저장:

```bash
python -m pytest \
  tests \
  -q \
  | tee \
  reports/day11_evidence/pytest.txt
```

Failure Analysis CSV:

```bash
head \
  reports/day11_failure_analysis.csv
```

Operational AI Event Log의 최근 Row도 현재 실행 증거로 남깁니다.

먼저 실제 경로를 Config에서 읽습니다.

```bash
AI_EVENT_LOG_PATH="$(
  python -c 'from src.config_loader import load_config; print(load_config("configs/settings.yaml")["ai_event_log_path"])'
)"
```

최근 Row를 Evidence 파일로 저장합니다.

```bash
tail -n 10 "$AI_EVENT_LOG_PATH" \
  > reports/day11_evidence/operational_event_log_tail.csv
```

내용:

```bash
cat reports/day11_evidence/operational_event_log_tail.csv
```

여기서는 단순히 파일이 있다는 것만 보지 않습니다. **System Test에서 만든 최근 State 전환 Row와 Timestamp가 포함되어 있는지** 확인합니다.

질문:

```text
1. API 정상 Evidence 파일은 무엇인가?
2. Unit Test 정상 Evidence 파일은 무엇인가?
3. Failure Prediction 결과는 어느 CSV에 있는가?
4. 현재 운영 State 변화 Evidence는 어느 파일에 저장했는가?
5. operational_event_log_tail.csv에서 무엇을 확인해야 하는가?
6. systemd 과거 장애는 어느 명령으로 확인하는가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
API Evidence
→ reports/day11_evidence/api_smoke.txt

pytest Evidence
→ reports/day11_evidence/pytest.txt

Failure Analysis
→ reports/day11_failure_analysis.csv

현재 운영 State 변화 Evidence
→ reports/day11_evidence/operational_event_log_tail.csv

operational_event_log_tail.csv 확인
→ 이번 System Test 이후의 timestamp
→ 최종 state 변화 Row
→ /status의 최종 state와 일치 여부

systemd Process / Restart 이력
→ journalctl -u <service>
```

예:

```bash
journalctl \
  -u subject13-edge.service \
  -n 50 \
  --no-pager
```

정상 동작의 증거는 한 종류만 있는 것이 아닙니다.

```text
자동 Test
+
API 결과
+
Operational Event Log
+
CSV 분석
+
Journal
+
Report
```

를 목적에 따라 사용합니다.

</details>

---

## Challenge 7 — 오늘 배운 전체 Pipeline을 자신의 말로 설명하기

코드를 닫고 다음 빈칸을 채웁니다.

```text
10일차 운영 Baseline
        ↓
________________ Test
→ 작은 Logic 검증
        ↓
________________ Test
→ Fixed Image + ONNX + Decision
        ↓
________________ Test
→ Camera + GPIO + Operational Event Log + systemd + FastAPI
        ↓
한 번에 ______ 변수 변경
        ↓
Validation에서 후보 비교
        ↓
________________ Dataset에서 마지막 확인
        ↓
Failure Case 촬영
        ↓
________________.csv
        ↓
AI 재분석
        ↓
reports/________________________.csv
        ↓
Root Cause 분류
        ↓
Accuracy / Latency / FPS / Stability
________________ 판단
        ↓
12일차 Raspberry Pi → Jetson
```

그리고 다음 상황을 말로 설명합니다.

```text
A. Unit Test는 PASS인데 System Test의 Camera가 FAIL했다.
→ 어디부터 볼 것인가?

B. /health API는 응답하지만 edge=DEGRADED다.
→ FastAPI가 죽은 것인가?

C. Threshold를 높였더니 False Warning은 줄고 Missed Warning은 늘었다.
→ 무조건 좋은가?

D. 64×64가 빠르지만 Validation Accuracy가 낮아졌다.
→ 어떤 판단이 필요한가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
10일차 운영 Baseline
        ↓
Unit Test
        ↓
Integration Test
        ↓
System Test
        ↓
한 번에 한 변수 변경
        ↓
Validation에서 후보 비교
        ↓
Test Dataset에서 마지막 확인
        ↓
Failure Case 촬영
        ↓
manifest.csv
        ↓
AI 재분석
        ↓
reports/day11_failure_analysis.csv
        ↓
Root Cause 분류
        ↓
Trade-off 판단
        ↓
12일차 Raspberry Pi → Jetson
```

상황 설명:

```text
A.
Decision / Stabilizer 같은 작은 Logic이 PASS라면
Camera / Runtime / 장치 연결부터 좁혀 본다.

B.
FastAPI가 응답했다는 뜻일 수 있다.
하지만 Edge Runtime State가 없거나 오래되었거나 ERROR일 수 있다.

C.
아니다.
False Warning과 Missed Warning의 비용을 현장 요구조건과 함께 봐야 한다.

D.
속도와 Accuracy의 Trade-off를 비교해야 한다.
빠르다는 이유만으로 선택하면 안 된다.
```

정답 문장을 그대로 외울 필요는 없습니다.

**Test 범위, Evidence, 한 변수 실험, Failure 영역, Trade-off를 연결해서 설명할 수 있으면 됩니다.**

</details>

---

# 85. Mini Challenge 종료 — 12일차를 위해 기본 상태로 복원하기

Challenge에서 바꾼 Config를 복원합니다.

```bash
cp \
  /tmp/day11_settings_before_challenge.yaml \
  configs/settings.yaml
```

Smoke Test 파일도 원래 상태로 복원합니다.

```bash
cp \
  /tmp/day11_api_smoke_before_challenge.py \
  scripts/api_smoke_test.py
```

현재 Config:

```bash
grep -E \
"ai_confidence_threshold|inference_every_n_frames|vote_window|vote_warning_min|debounce_required|warning_hold_sec|api_port" \
configs/settings.yaml
```

Edge Service를 정상 상태로 돌립니다.

```bash
sudo systemctl restart \
  subject13-edge.service
```

API Service:

```bash
sudo systemctl restart \
  subject13-api.service
```

최종 상태:

```bash
systemctl status \
  subject13-edge.service \
  subject13-api.service \
  --no-pager
```

API:

```bash
python -m scripts.api_smoke_test \
  --host <RPI_IP> \
  --port <API_PORT>
```

Unit Test:

```bash
python -m pytest \
  tests \
  -q
```

Operational AI Event Log도 최신 운영 Runtime에서 계속 기록 가능한 상태인지 확인합니다.

```bash
AI_EVENT_LOG_PATH="$(
  python -c 'from src.config_loader import load_config; print(load_config("configs/settings.yaml")["ai_event_log_path"])'
)"
```

```bash
tail -n 5 "$AI_EVENT_LOG_PATH"
```

마지막으로 Camera 앞에서 최종 State를 한 번 전환한 뒤 새 Row가 추가되는지 확인합니다. 이 확인까지 끝나야 12일차가 **10일차 운영 Event Log가 포함된 Tested Baseline**을 전달받습니다.

이제 Challenge용 변경이 12일차로 넘어가지 않아야 합니다.

---

# PART H. 결과 기록 · README · Git · 최종 복습

# 86. Final Decision을 Report에 작성하기

오늘 실험 결과를 보고 한 문단으로 정리합니다.

`reports/day11_test_analysis.md`:

````md
## Final Decision

### 현재 운영 Baseline

Model:

Input Size:

Confidence Threshold:

Frame Skip:

Vote Window:

Warning Minimum:

Debounce:

Hold:

Operational Event Log Path:

Operational Event Log System Test:
PASS / FAIL

Operational Event Log Evidence:
reports/day11_evidence/operational_event_log_tail.csv

### Validation에서 확인한 Trade-off

Accuracy:

False Warning:

Missed Warning:

Latency:

P95:

FPS:

Output Stability:

### Test Dataset 최종 확인

수행 여부:

결과:

### 가장 중요한 Failure 조건

1.

2.

3.

### 현재 설정을 유지하거나 바꾸는 이유

### 12일차로 넘길 주의사항
````

Test 결과가 좋지 않았다고 숨기지 않습니다.

실패 조건이 명확하게 기록되어 있으면 12일차와 이후 프로젝트에서 중요한 개선 근거가 됩니다.

---

# 87. 실제 운영 Config를 바꿀지 결정하는 규칙

11일차 실험에서 후보가 좋아 보였다고 바로 운영 Service 설정을 바꾸지 않습니다.

기본 원칙:

```text
Validation에서 후보 선택
        ↓
Test에서 마지막 확인
        ↓
System Test 재확인 가능
        ↓
변경 이유 Report 기록
        ↓
그 뒤 운영 Baseline 채택
```

수업에서 별도 채택 결정을 하지 않았다면 **10일차 정상 운영 Config를 유지**합니다.

이렇게 하면 12일차는 알려진 정상 Baseline에서 시작할 수 있습니다.

---

# 88. Test Source와 Report가 서로 어디에 있는지 확인하기

현재 핵심 Source:

```text
tests/
├── test_decision.py
├── test_stabilizer.py
└── test_runtime_state.py

scripts/
├── api_smoke_test.py
├── evaluate_operational_threshold.py
├── capture_failure_case.py
└── analyze_failure_cases.py
```

결과:

```text
reports/
├── day11_test_analysis.md
├── day11_failure_analysis.csv
└── day11_evidence/
```

Git 제외 Data:

```text
data/day11_failures/
├── images/
└── manifest.csv
```

---

# 89. README에 11일차 실행 방법 추가하기

`README.md`에 다음 내용을 추가합니다.

````md
## Day 11 Test · Failure Analysis · Trade-off

### Unit Test

```bash
python -m pytest tests -v
```

### Fixed Image Integration

```bash
python -m scripts.onnx_decision_test \
  --image data/deploy_samples/normal.jpg
```

### API Smoke Test

```bash
python -m scripts.api_smoke_test \
  --host <RPI_IP> \
  --port <API_PORT>
```

### Operational Threshold

후보 선택은 Validation Dataset을 사용합니다.

```bash
python -m scripts.evaluate_operational_threshold \
  --model models/day07_tiny_cnn.onnx \
  --meta models/day07_tiny_cnn.meta.json \
  --dataset data/dataset/val \
  --threshold 0.70
```

### Operational Event Log

실제 경로는 `configs/settings.yaml`의 `ai_event_log_path`를 기준으로 확인합니다.

```bash
AI_EVENT_LOG_PATH="$(
  python -c 'from src.config_loader import load_config; print(load_config("configs/settings.yaml")["ai_event_log_path"])'
)"
tail -n 10 "$AI_EVENT_LOG_PATH"
```

System Test에서는 파일 존재만 확인하지 않고 NORMAL ↔ WARNING 최종 State 전환 뒤 **새 Timestamp Row가 추가되는지** 확인합니다.

### Failure Analysis

```bash
python -m scripts.analyze_failure_cases
```

### Test Flow

```text
Unit Test
→ Integration Test
→ System Test
  → Operational Event Log 새 Row 확인
→ Controlled Experiment
→ Failure Collection
→ Root Cause Classification
→ Trade-off
```

### Important

수동 Camera 프로그램을 실행할 때는
`subject13-edge.service`가 Camera를 점유하고 있지 않은지 먼저 확인합니다.

Validation에서 후보를 선택하고
Test Dataset은 마지막 확인에 사용합니다.
````

---

# 90. Git에 올리기 전에 제외 대상 확인하기

```bash
git status
```

다음은 Git 관리 대상입니다.

```text
tests/
새 scripts/
reports/day11_test_analysis.md
reports/day11_failure_analysis.csv
reports/day11_evidence/
README.md
```

다음은 기존 정책대로 Git에서 제외합니다.

```text
data/day11_failures/
Dataset
Model Weight / ONNX
runtime/
raw logs/
.venv/
device.local.yaml
```

`configs/device.local.yaml`의 관리 정책은 기존 프로젝트 규칙을 그대로 따릅니다.

---

# 91. 11일차 Source Commit 만들기

실험 Branch에서:

```bash
git add \
  tests \
  scripts/api_smoke_test.py \
  scripts/evaluate_operational_threshold.py \
  scripts/capture_failure_case.py \
  scripts/analyze_failure_cases.py
```

Commit:

```bash
git commit -m \
  "test: add day11 edge validation and failure analysis"
```

새 파일을 `git commit -am`만으로 저장하려 하지 않습니다.

새 파일은 먼저 `git add`가 필요합니다.

---

# 92. Report와 README Commit 만들기

```bash
git add \
  reports/day11_test_analysis.md \
  reports/day11_failure_analysis.csv \
  README.md
```

Evidence 폴더를 Git으로 관리하기로 했다면:

```bash
if [ -d reports/day11_evidence ]; then
  git add reports/day11_evidence
fi
```

Commit:

```bash
git commit -m \
  "docs: record day11 tests tradeoff and failures"
```

---

# 93. 실험 Branch를 Main에 반영하기

먼저 현재 변경사항이 깨끗한지 확인합니다.

```bash
git status
```

예상:

```text
nothing to commit, working tree clean
```

Main으로 이동합니다.

```bash
git switch main
```

11일차 결과를 Main에 반영합니다.

```bash
git merge \
  experiment/day11
```

확인:

```bash
git log \
  --oneline \
  --decorate \
  -12
```

Merge Conflict가 발생하면 무작정 파일을 덮어쓰지 않습니다.

```text
충돌 파일 확인
        ↓
어느 Version을 유지할지 내용 비교
        ↓
충돌 표시 제거
        ↓
Test 재실행
        ↓
Commit
```

---

# 94. 내부 Remote가 있는 경우에만 Push하기

현재 Remote:

```bash
git remote -v
```

허용된 내부 GitLab / Gitea가 연결되어 있고 기관 정책상 허용된 경우:

```bash
git push
```

Remote가 없거나 외부 Git 사용이 제한된 폐쇄망에서는 Push를 억지로 시도하지 않습니다.

**Local Git Commit만으로도 오늘 Version 기록은 완성됩니다.**

---

# 95. PC와 Raspberry Pi Source를 최종적으로 맞추기

오늘 PC에서 Test Source를 작성했다면 Raspberry Pi에도 다음 날 필요한 Source가 있어야 합니다.

수업 환경에 따라 둘 중 하나를 사용합니다.

### 내부 Git Remote가 있는 경우

PC에서 Push 후 Raspberry Pi에서:

```bash
cd ~/ai_vision/subject13_edge_ai
git switch main
git pull
```

### Local Git + 파일 전달 환경

수업에서 정한 SCP 또는 내부 파일 전달 방식으로 **Source와 Test 파일만** 맞춥니다.

장치 전용 설정을 덮어쓰지 않습니다.

```text
전송 가능
→ tests/
→ 새 scripts/
→ README / reports

덮어쓰기 주의
→ configs/device.local.yaml

전송하지 않음
→ .venv/
→ runtime/
→ raw logs/
→ 장치별 임시 파일
```

Source를 맞춘 뒤 Raspberry Pi에서:

```bash
python -m pytest \
  tests \
  -q
```

Pi에 pytest가 없다면 오늘 사용한 Package 설치 정책에 따라 준비합니다.

---

# 96. 11일차가 끝난 시점의 프로젝트 구조

```text
subject13_edge_ai/
│
├── configs/
│   └── settings.yaml
│
├── reports/
│   ├── ...
│   ├── day10_fault_recovery.md
│   ├── day11_test_analysis.md
│   ├── day11_failure_analysis.csv
│   └── day11_evidence/
│       └── operational_event_log_tail.csv
│
├── scripts/
│   ├── ...
│   ├── api_smoke_test.py
│   ├── evaluate_operational_threshold.py
│   ├── capture_failure_case.py
│   └── analyze_failure_cases.py
│
├── src/
│   ├── ...
│   ├── decision.py
│   ├── event_logger.py
│   ├── stabilizer.py
│   └── runtime_state.py
│
├── tests/
│   ├── test_decision.py
│   ├── test_stabilizer.py
│   └── test_runtime_state.py
│
├── systemd/
│   └── generated/
│
├── .gitignore
├── README.md
└── requirements.txt
```

장치/실행 Data:

```text
data/
├── dataset/
├── deploy_samples/
└── day11_failures/
    ├── images/
    └── manifest.csv

models/
└── ...

runtime/
└── edge_state.json

logs/
└── <configs/settings.yaml의 ai_event_log_path>
    → 10일차 운영 Edge Runtime이 계속 갱신하는 Event CSV
```

---

# 97. 오늘 만든 Test 체계 한 번에 다시 보기

```text
[Unit Test]

Decision
Stabilizer
Runtime State
        ↓

[Integration Test]

Fixed Image
→ Preprocess
→ ONNX
→ Decision

FastAPI
→ JSON Contract
        ↓

[System Test]

Camera
→ Edge Runtime
→ ONNX / Decision / Stabilization
→ GPIO
→ Operational AI Event Log
→ systemd
→ Runtime State
→ FastAPI
→ PC
        ↓

[Controlled Experiment]

한 변수만 변경
→ Validation에서 후보 비교
→ Test에서 마지막 확인
        ↓

[Failure Analysis]

실패 조건 촬영
→ Manifest
→ ONNX 재분석
→ Failure CSV
→ Root Cause
        ↓

[Trade-off]

Accuracy
False Warning
Missed Warning
Latency
FPS
Stability
→ 선택 이유
```

---

# 98. 1~11일차에서 시스템이 어떻게 성장했는지 확인하기

```text
1일차
가상 입력
→ Decision
→ Log

2일차
PC
→ SSH
→ Raspberry Pi

3일차
Camera
→ GPIO

4일차
Sensor
→ Sampling
→ Timestamp
→ CSV

5일차
Camera
→ Rule
→ Decision
→ GPIO / Log

6일차
Camera
→ Dataset
→ Tiny CNN
→ PyTorch Model

7일차
PyTorch
→ ONNX
→ Raspberry Pi Inference

8일차
Latency / FPS
→ 병목 측정

9일차
Frame Skip
→ Voting
→ Debounce
→ Hold
→ 안정화

10일차
systemd
→ 자동실행
→ 장애 복구
→ Operational AI Event Log
→ Runtime State
→ FastAPI

11일차
Unit Test
→ Integration Test
→ System Test
→ Controlled Experiment
→ Failure Analysis
→ Trade-off
```

이제 시스템은 단순히 “동작한다”에서 끝나지 않습니다.

```text
어떤 기능이 정상인가?
어디에서 실패하는가?
어떤 설정을 왜 선택했는가?
```

를 Evidence로 설명할 수 있어야 합니다.

---

# 99. 핵심 복습 문제 — 교재를 보지 않고 먼저 답하기

### 문제 1

Unit Test와 System Test의 가장 큰 차이는 무엇인가요?

### 문제 2

Decision Unit Test에서 `confidence == threshold`를 따로 Test한 이유는 무엇인가요?

### 문제 3

Stabilizer를 Camera 없이 Test할 수 있는 이유는 무엇인가요?

### 문제 4

`tmp_path`를 Runtime State Test에 사용한 이유는 무엇인가요?

### 문제 5

Integration Test에서 저장 Image를 사용하는 장점은 무엇인가요?

### 문제 6

API Smoke Test가 PASS했는데도 전체 Edge 시스템이 FAIL일 수 있는 이유는 무엇인가요?

### 문제 7

한 번에 여러 설정을 바꾸면 왜 실험 해석이 어려워지나요?

### 문제 8

Threshold 후보 선택에 Test Dataset을 반복해서 사용하면 왜 문제가 되나요?

### 문제 9

Confidence Threshold를 높일 때 일반적으로 어떤 Trade-off를 관찰할 수 있나요?

### 문제 10

Model-only Benchmark 전에 Edge Service를 중지한 이유는 무엇인가요?

### 문제 11

수동 Camera 프로그램 전에 Edge Service를 중지해야 하는 이유는 무엇인가요?

### 문제 12

Failure Manifest에는 왜 `expected_state`와 `image_path`를 함께 기록하나요?

### 문제 13

`correct=False`가 나오면 바로 Model을 다시 학습해야 하나요?

### 문제 14

`Input / Data-Model / Runtime / Performance / Operation` 분류가 왜 필요한가요?

### 문제 15

11일차 마지막에 어떤 상태를 남겨야 12일차를 시작할 수 있나요?

### 문제 16

Operational AI Event Log를 System Test에서 단순히 `파일 존재`로만 확인하면 부족한 이유는 무엇인가요?

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

### 1

Unit Test는 작은 함수나 Class의 Logic을 제한된 입력으로 검증합니다. System Test는 실제 Camera·AI·GPIO·Operational AI Event Log·systemd·Runtime State·FastAPI까지 전체 환경을 검증합니다.

### 2

현재 운영 정책이 `>=`인지 `>`인지 경계에서 동작이 달라지기 때문입니다. 경계값 정책을 Test로 고정하면 의도하지 않은 변경을 발견할 수 있습니다.

### 3

`TemporalStabilizer.update()`는 `NORMAL / WARNING`과 시간값을 입력받아 동작하므로 가상의 상태와 시간을 직접 넣어 Logic만 확인할 수 있습니다.

### 4

실제 `runtime/edge_state.json`을 Test 중에 덮어쓰거나 망가뜨리지 않고 독립된 임시 파일에서 Write/Read를 확인하기 위해서입니다.

### 5

Camera 조건을 고정하고 같은 Image를 반복 입력할 수 있으므로 Preprocess → ONNX → Decision 연결 문제를 더 쉽게 분리할 수 있습니다.

### 6

Smoke Test는 Endpoint와 JSON 구조를 확인합니다. FastAPI는 응답해도 Edge Runtime State가 stale이거나 Camera·GPIO가 실패했을 수 있습니다.

### 7

결과가 변했을 때 어떤 변경이 원인이었는지 구분하기 어렵기 때문입니다.

### 8

여러 후보를 Test 결과로 선택하면 Test가 사실상 Validation처럼 사용되어 최종 성능을 독립적으로 평가하기 어려워집니다.

### 9

Threshold를 높이면 False Warning은 줄 수 있지만 Missed Warning이 늘 수 있습니다. 실제 결과와 현장 비용을 함께 봐야 합니다.

### 10

Edge Runtime이 동시에 ONNX 추론을 수행하면 CPU 부하가 Benchmark에 섞여 96/64 비교 조건이 달라질 수 있기 때문입니다.

### 11

두 Process가 같은 Camera를 동시에 열려고 하거나 같은 GPIO를 서로 다르게 제어할 수 있기 때문입니다.

### 12

어떤 조건에서 촬영한 어떤 Image가 어떤 정답이어야 했는지 나중에 재현하고 분석하기 위해서입니다.

### 13

아닙니다. Input, Data, Model, Threshold Policy, Runtime, Performance, Operation 중 어느 영역 문제인지 먼저 Evidence로 분류합니다.

### 14

같은 증상도 수정해야 할 위치가 다르기 때문입니다. 원인 영역을 좁히면 불필요한 재학습이나 코드 변경을 줄일 수 있습니다.

### 15

최소 다음 상태가 필요합니다.

```text
전체 Unit Test
→ PASS

Integration / System Test
→ 결과 기록

Operational AI Event Log
→ 현재 실행에서 새 Row 생성 확인
→ Evidence 저장

Failure Analysis
→ CSV + Root Cause 기록

Trade-off
→ 선택 이유 기록

subject13-edge.service
→ active

subject13-api.service
→ active

/health
→ 정상

Challenge용 변경
→ 복원

Git main
→ clean

README / Report
→ 완료
```

### 16

과거 7일차나 이전 실행에서 만들어진 Event CSV가 그대로 남아 있을 수 있기 때문입니다. **현재 System Test에서 최종 운영 State를 실제로 전환한 뒤, 줄 수 증가·새 Timestamp·최종 `state` 일치**를 확인해야 지금 실행 중인 `edge_runtime.py`가 Event Log를 쓰고 있다는 증거가 됩니다.

</details>

---

# 100. 자가 체크리스트

수업을 마치기 전에 직접 체크합니다.

- [ ] 10일차 운영 Baseline에서 시작했다.
- [ ] 새 프로젝트나 새 `.venv`를 만들지 않았다.
- [ ] `<RPI_IP>`와 `<API_PORT>`를 실제 환경에서 확인했다.
- [ ] `day10-operational-baseline` Tag를 확인하거나 만들었다.
- [ ] Unit / Integration / System Test의 차이를 설명할 수 있다.
- [ ] `tests/test_decision.py`를 실행했다.
- [ ] `tests/test_stabilizer.py`를 실행했다.
- [ ] `tests/test_runtime_state.py`를 실행했다.
- [ ] 전체 pytest가 PASS한다.
- [ ] Test를 일부러 실패시킨 뒤 원인을 읽고 복구했다.
- [ ] 저장 Image → Preprocess → ONNX → Decision 연결을 확인했다.
- [ ] PyTorch와 ONNX Parity를 확인했다.
- [ ] API Smoke Test를 실제 IP와 Port로 실행했다.
- [ ] NORMAL / WARNING System Test를 수행했다.
- [ ] `ai_event_log_path`의 실제 운영 Event CSV 경로를 Config에서 확인했다.
- [ ] System Test 전후 Event CSV 줄 수를 비교했다.
- [ ] 최종 State 전환 뒤 새로운 Timestamp Row가 추가되는 것을 확인했다.
- [ ] Event CSV의 `state`가 `raw_state`가 아니라 Voting·Debounce·Hold가 반영된 최종 운영 State라는 것을 설명할 수 있다.
- [ ] `reports/day11_evidence/operational_event_log_tail.csv`에 최근 운영 Event를 남겼다.
- [ ] `/metrics`가 갱신되는 것을 확인했다.
- [ ] 자동재시작을 System Test 항목으로 확인했다.
- [ ] 후보 선택은 Validation Dataset으로 수행했다.
- [ ] Test Dataset은 선택 후 마지막 확인에 사용했다.
- [ ] Threshold 실험에서 False Warning과 Missed Warning을 비교했다.
- [ ] 96×96 / 64×64 Variant를 새로 만들지 않고 8일차 결과를 재사용했다.
- [ ] Model-only Benchmark 전에 Edge Service 부하를 제거했다.
- [ ] Hold 실험 전에 Camera를 사용하는 Edge Service를 중지했다.
- [ ] Hold 실험 후 Config와 Service를 복원했다.
- [ ] Failure Image를 최소 5개 수집했다.
- [ ] `manifest.csv`와 실제 Image 경로가 연결된다.
- [ ] `day11_failure_analysis.csv`를 생성했다.
- [ ] Failure 원인을 영역별로 분류했다.
- [ ] Mini Challenge 7단계를 수행했다.
- [ ] Challenge에서 바꾼 Config와 Source를 복원했다.
- [ ] `reports/day11_test_analysis.md`를 완성했다.
- [ ] README에 11일차 실행 흐름을 정리했다.
- [ ] Test Source와 Report를 Git Commit했다.
- [ ] 필요하면 PC와 Pi Source를 안전하게 동기화했다.
- [ ] `configs/device.local.yaml`을 다른 장치 값으로 덮어쓰지 않았다.
- [ ] 수업 종료 시 Edge/API Service가 정상이다.
- [ ] 12일차에는 Raspberry Pi Tested Baseline을 Jetson과 비교한다는 것을 설명할 수 있다.

---

# 101. 수업 종료 전 최종 확인

Raspberry Pi에서:

## Service

```bash
systemctl status \
  subject13-edge.service \
  subject13-api.service \
  --no-pager
```

## Enabled

```bash
systemctl is-enabled \
  subject13-edge.service
```

```bash
systemctl is-enabled \
  subject13-api.service
```

## Runtime State

```bash
cat \
  runtime/edge_state.json
```

## Operational AI Event Log

실제 경로를 다시 읽습니다.

```bash
AI_EVENT_LOG_PATH="$(
  python -c 'from src.config_loader import load_config; print(load_config("configs/settings.yaml")["ai_event_log_path"])'
)"
```

```bash
echo "$AI_EVENT_LOG_PATH"
tail -n 5 "$AI_EVENT_LOG_PATH"
```

최근 System Test에서 만든 State 전환 Row와 Timestamp가 남아 있는지 확인합니다.

## API

```bash
curl \
  http://127.0.0.1:<API_PORT>/health
```

PC에서:

```bash
python -m scripts.api_smoke_test \
  --host <RPI_IP> \
  --port <API_PORT>
```

Unit Test:

```bash
python -m pytest \
  tests \
  -q
```

Git:

```bash
git status
```

최종적으로 `main` Branch에 있다면:

```bash
git branch --show-current
```

예상:

```text
main
```

---

# 102. 오늘 반드시 남아 있어야 하는 결과

```text
Unit Test 결과
        ↓
Decision / Stabilizer / Runtime State

Integration Test 결과
        ↓
Fixed Image / ONNX / Decision
API Smoke Test

System Test Checklist
        ↓
Camera / GPIO / Operational AI Event Log / systemd / FastAPI

Operational Event Log Evidence
        ↓
reports/day11_evidence/operational_event_log_tail.csv

Experiment 01
→ Confidence Threshold

Experiment 02
→ Input Size

Experiment 03
→ Warning Hold

Failure Image 5개 이상
+
manifest.csv

reports/day11_failure_analysis.csv

Root Cause 분류표

Accuracy
False Warning
Missed Warning
Latency
FPS
Stability
Trade-off 설명

Mini Challenge Evidence

README

Git Commit

정상 Service / API 상태
```

---

# 103. 다음 날 연결 — Raspberry Pi Tested Baseline을 Jetson으로 확장하기

11일차까지 Raspberry Pi에서 다음 전체 흐름을 만들고 검증했습니다.

```text
실제 Camera
        ↓
Preprocess
        ↓
ONNX AI
        ↓
Operational Decision
        ↓
Stabilization
        ↓
GPIO / Operational AI Event Log
        ↓
Performance
        ↓
systemd
        ↓
Runtime State
        ↓
FastAPI
        ↓
Unit / Integration / System Test
        ↓
Failure Analysis
        ↓
Trade-off
```

12일차에는 이 구조를 버리고 Jetson에서 처음부터 다시 만드는 것이 아닙니다.

```text
11일차 Raspberry Pi Tested Baseline
        ↓
공통 구조와 장치 종속 부분 분리
        ↓
Raspberry Pi
CPU + ONNX Runtime
        ↓
Jetson Orin Nano Super
GPU + ONNX / TensorRT
```

12일차에서 다시 묻습니다.

```text
어떤 Source는 그대로 재사용할 수 있는가?

Camera 계층은 무엇이 달라질 수 있는가?

ONNX Runtime CPU와
TensorRT GPU는 어디가 다른가?

Latency / FPS 측정 생각은
그대로 가져갈 수 있는가?

systemd / Runtime State / FastAPI / Test 구조는
Jetson에서도 어떻게 재사용할 수 있는가?
```

12일차 시작에서는 오늘의 깨끗한 `main` 상태를 확인하고 **`day11-rpi-tested-baseline`** 기준점을 만들 수 있습니다.

따라서 오늘은 Challenge용 Config, 잘못된 Port, 중지된 Service, 깨진 Test를 남겨 두지 않습니다.

> **11일차의 마지막 한 문장:**  
> **“동작하는 Edge AI를 Test하고, 실패를 분류하고, 한 변수 실험으로 Trade-off를 설명할 수 있는 상태까지 만들었다.”**
