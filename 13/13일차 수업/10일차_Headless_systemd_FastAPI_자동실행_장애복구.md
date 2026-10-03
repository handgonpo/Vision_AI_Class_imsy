
> **오늘의 핵심:** 9일차에는 Raspberry Pi에서 `Frame Skip · Majority Voting · Debounce · Hold`를 적용하고, 속도·반응성·안정성의 Trade-off를 비교하여 **운영 Config**를 선택했습니다.
>
> 10일차에는 AI 모델을 다시 학습하거나 새로운 판단 알고리즘을 만드는 것이 아닙니다. **9일차의 최종 Edge AI Pipeline을 사람이 매번 SSH로 실행하지 않아도 되는 운영형 프로그램으로 바꾸는 것**이 핵심입니다.
>
> ```text
> 9일차
> Camera
> → ONNX AI
> → Operational Decision
> → Stabilization
> → GPIO
> → 사람이 직접 실행
>
>            ↓
>
> 10일차
> Raspberry Pi Boot
> → systemd
> → Edge Runtime 자동 실행
> → AI Event Log
> → Runtime State
> → FastAPI
> → PC에서 상태 확인
> → 비정상 종료 시 자동 재시작
> ```
>
> 1~9일차에 사용한 `subject13_edge_ai` 프로젝트와 Raspberry Pi의 기존 `.venv`를 그대로 이어서 사용합니다. **새 프로젝트와 새 가상환경을 만들지 않습니다.**

---

## 오늘 가장 중요한 질문

9일차까지는 다음 질문에 답했습니다.

```text
AI가 Raspberry Pi에서 실행되는가?
        ↓
속도와 흔들림을 측정할 수 있는가?
        ↓
Frame Skip / Voting / Debounce / Hold를 조정할 수 있는가?
```

10일차에는 질문이 달라집니다.

```text
전원을 켠 뒤
사람이 Python 명령을 직접 입력하지 않아도 실행되는가?
        ↓
프로그램이 죽으면 다시 살아나는가?
        ↓
모니터가 없어도 현재 상태를 확인할 수 있는가?
        ↓
API가 고장나도 Edge AI 자체는 계속 동작하는가?
        ↓
장애 원인을 Journal과 Runtime State에서 찾을 수 있는가?
```

따라서 오늘의 핵심 질문은 다음입니다.

> **“9일차의 Edge AI를 부팅 후 자동으로 실행하고, 장애 시 복구하며, 외부 PC에서 상태를 확인할 수 있는 독립적인 운영형 Edge 시스템으로 만들 수 있는가?”**

---

## 9일차와 10일차는 어디가 이어질까요?

9일차 마지막 Raspberry Pi에는 다음 흐름이 준비되어 있습니다.

```text
Camera
→ Preprocess
→ ONNX Runtime
→ Class + Confidence
→ Operational Decision
→ Frame Skip
→ Majority Voting
→ Debounce
→ Hold
→ GPIO
```

그리고 다음 운영값을 실제 실험 결과로 선택했습니다.

```yaml
inference_every_n_frames: <9일차에서 선택한 값>
vote_window: <9일차에서 선택한 값>
vote_warning_min: <9일차에서 선택한 값>
debounce_required: <9일차에서 선택한 값>
warning_hold_sec: <9일차에서 선택한 값>
```

오늘은 이 값을 임의로 초기화하지 않습니다.

```text
9일차 Final Config
        ↓
10일차 Edge Runtime에서 그대로 사용
```

즉, 오늘의 새 학습은 **AI 판단을 바꾸는 것**이 아니라 **실행과 운영 방식을 바꾸는 것**입니다.

---

## 10일차 결과는 11일차에 어떻게 연결될까요?

10일차가 끝나면 Raspberry Pi는 다음 상태가 됩니다.

```text
Raspberry Pi Boot
        ↓
systemd
        ├─ subject13-edge.service
        │      ↓
        │   Edge Runtime
        │      ↓
        │   Camera → ONNX → Stabilization → GPIO
        │      ├─→ AI Event Log
        │      ↓
        │   runtime/edge_state.json
        │
        └─ subject13-api.service
               ↓
            FastAPI
               ↓
        /health /status /metrics
               ↓
              PC
```

11일차에는 새로운 운영 기능을 계속 추가하는 것이 아니라 **이 상태를 Test 대상으로 사용**합니다.

```text
10일차 운영 Baseline
        ↓
11일차 Unit Test
        ↓
Integration Test
        ↓
System Test
        ↓
Failure Analysis
        ↓
Accuracy / Latency / FPS / Stability
Trade-off 검증
```

따라서 오늘 수업 종료 시에는 두 Service가 다시 정상 상태이고, API가 응답하며, 장애 복구 기록이 남아 있어야 합니다.

---

## 오늘의 학습 목표

수업이 끝나면 다음을 할 수 있어야 합니다.

```text
1. Headless 운영이 무엇인지 설명한다.
2. 9일차 Edge Pipeline과 10일차 운영 계층의 차이를 설명한다.
3. Runtime State를 JSON으로 안전하게 저장하고 읽는다.
4. Heartbeat와 stale 상태의 의미를 설명한다.
5. 9일차 Pipeline을 Headless Edge Runtime으로 실행한다.
6. systemd Service가 Python 프로그램을 어떻게 실행하는지 설명한다.
7. start / stop / restart / enable의 차이를 구분한다.
8. journalctl로 Service 오류를 찾는다.
9. 재부팅 후 Edge Runtime이 자동 실행되는지 확인한다.
10. 비정상 종료 후 Restart=on-failure가 동작하는지 확인한다.
11. FastAPI를 Edge Runtime과 분리하는 이유를 설명한다.
12. /health /status /metrics의 역할을 구분한다.
13. Raspberry Pi 내부와 외부 PC에서 API를 확인한다.
14. API가 중지되어도 Edge AI가 계속 동작하는지 검증한다.
15. 환경마다 달라지는 IP / Hostname / Port / Camera 값을 직접 확인한다.
16. 실행 결과를 systemd Unit / Python 코드 / Runtime State까지 역추적한다.
17. 장애를 Network / systemd / Runtime / Model / Camera / API 영역으로 분류한다.
18. 7일차의 `AIEventLogger`를 운영 Runtime에 재사용하여 **최종 운영 State 변화**를 Event CSV로 기록한다.
19. Mini Challenge 후 11일차가 사용할 정상 운영 Baseline으로 복원한다.
```

---

## 오늘의 8시간 학습 흐름

| 학습 블록 | 권장 시간 | 무엇을 배우나요? | 끝나면 할 수 있어야 하는 것 |
|---|---:|---|---|
| 1 | 40분 | 9일차 Handoff · Headless 구조 | 오늘 새로 추가되는 운영 계층을 설명할 수 있다 |
| 2 | 70분 | Runtime State · Heartbeat · Edge Runtime | Terminal 없이도 상태를 파일로 남길 수 있다 |
| 3 | 90분 | systemd · Service · Journal · Boot | 자동실행과 자동재시작을 검증할 수 있다 |
| 4 | 85분 | FastAPI · 상태 API · PC 조회 | Edge Runtime과 API를 분리하여 상태를 제공할 수 있다 |
| 5 | 55분 | 장애 복구 Guided Lab | Crash·Camera·Model·Config 오류를 관찰하고 복구할 수 있다 |
| 6 | 45분 | 운영 진단 · Remote Client · 작은 확장 | Headless 장치의 상태를 순서대로 진단할 수 있다 |
| 7 | 65분 | Mini Challenge | 결과 역추적·예측·설정 변경·기능 수정·오류 복구를 스스로 수행할 수 있다 |
| 8 | 30분 | Baseline 복원 · Report · README · Git · 복습 | 11일차 Test 시작 상태를 완성할 수 있다 |

총 480분을 기준으로 합니다. Raspberry Pi 재부팅 시간, Camera 초기화, 교육장 Network 상태에 따라 실제 시간은 조금 달라질 수 있습니다.

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

10일차에서는 여기에 **운영**과 **상태 조회**가 추가됩니다.

```text
입력
→ Camera

처리
→ Preprocess
→ ONNX

판단
→ Class + Confidence
→ Stabilization

출력
→ GPIO

운영
→ systemd
→ Boot 자동실행
→ 비정상 종료 시 자동재시작

기록
→ AI Event CSV
→ Runtime State
→ systemd Journal

상태 조회
→ FastAPI
→ PC
```

코드를 보다가 헷갈리면 다음 세 질문으로 돌아옵니다.

> **“이 코드는 AI 판단을 담당하는가, 운영을 담당하는가, 상태 조회를 담당하는가?”**  
> **“이 결과가 나오기 전에 어떤 파일과 Service가 연결되었는가?”**  
> **“이 장애가 발생해도 Edge AI 자체는 계속 동작해야 하는가?”**

---

## 오늘 사용할 실제 장비와 이전 일차 결과

```text
Raspberry Pi
→ Raspberry Pi OS / Linux
→ SSH
→ USB Camera
→ Green / Red LED
→ 선택 Buzzer

7일차부터 재사용
→ models/day07_tiny_cnn.onnx
→ models/day07_tiny_cnn.meta.json
→ src/ai_preprocess.py
→ src/inference_onnx.py
→ src/decision.py
→ src/event_logger.py
→ configs/settings.yaml의 `ai_event_log_path`

9일차에서 재사용
→ src/stabilizer.py
→ src/resource_monitor.py
→ configs/settings.yaml
→ configs/device.local.yaml
```

오늘 새로 추가하는 핵심 파일은 다음입니다.

```text
src/runtime_state.py

scripts/runtime_state_test.py
scripts/edge_runtime.py
scripts/render_systemd_units.py
scripts/status_api.py
scripts/check_remote_status.py

systemd/generated/
→ 장치에서 생성되는 Service Unit

runtime/edge_state.json
→ 실행 중 계속 갱신되는 장치 상태

reports/day10_fault_recovery.md
reports/day10_pi_environment.txt
```

---

## 오늘 배울 기술 스택

| 기술 | 오늘 어디에 사용하나요? |
|---|---|
| Python `json` | Runtime State 저장 |
| `os.replace()` | 완성된 임시 파일을 State 파일로 교체 |
| `datetime` / UTC | Heartbeat 시각 기록 |
| Linux `systemd` | Edge Runtime·FastAPI Process 관리 |
| `systemctl` | Start·Stop·Restart·Enable·Status |
| `journalctl` | Service 표준출력·Traceback 확인 |
| FastAPI | 상태 조회용 HTTP API |
| Uvicorn | FastAPI 실행 Server |
| `curl` | API Endpoint 확인 |
| `ss` | 실제 Listen Port 확인 |
| Git | Source·Config·Report Version 관리 |

---

## 실습 전에 자신의 환경값 적어두기

환경마다 달라지는 값을 교재 예시 그대로 사용하지 않습니다.

```text
Raspberry Pi 사용자 이름     : ______________________________
Raspberry Pi Hostname        : ______________________________
Raspberry Pi IPv4            : ______________________________
Raspberry Pi Project Path    : ______________________________
Raspberry Pi Python Path     : ______________________________

Camera Index                 : ______________________________
Camera Resolution            : ______________________________
use_buzzer                   : true / false

API Host                     : ______________________________
API Port                     : ______________________________

9일차 Frame Skip             : ______________________________
9일차 Vote Window            : ______________________________
9일차 Warning Minimum        : ______________________________
9일차 Debounce               : ______________________________
9일차 Hold                   : ______________________________
```

교재에서는 다음 표기법을 사용합니다.

```text
<RPI_USER>
→ 자신의 Raspberry Pi 사용자 이름

<RPI_HOSTNAME>
→ 자신의 Raspberry Pi Hostname

<RPI_IP>
→ 현재 확인한 Raspberry Pi IPv4

<API_PORT>
→ configs/settings.yaml의 api_port 값
```

IP는 DHCP 환경에서 재부팅 뒤 달라질 수 있습니다. 연결이 안 된다고 이전 IP를 계속 반복하지 말고 Hostname, 공유기/교육장 장비 목록, 직접 화면의 `hostname -I` 등 현재 환경에서 가능한 방법으로 다시 확인합니다.

---

## 오늘의 전체 실습 흐름

```text
9일차 Final Config 확인
        ↓
9일차 Edge Pipeline 짧게 Precheck
        ↓
Headless 개념
        ↓
Runtime State 설계
        ↓
RuntimeStateStore
        ↓
State 단독 Test
        ↓
Headless Edge Runtime
        ↓
운영 State 변화 → AI Event Log 기록
        ↓
Terminal에서 먼저 검증
        ↓
systemd Unit 생성
        ↓
Edge Service 시작
        ↓
Journal 확인
        ↓
Boot 자동실행
        ↓
Crash 자동재시작
        ↓
FastAPI 설치·확인
        ↓
/health /status /metrics
        ↓
Raspberry Pi 내부 curl
        ↓
PC에서 curl / Browser
        ↓
API Service 등록
        ↓
두 Service Boot 자동실행
        ↓
장애 복구 실험
        ↓
운영 진단
        ↓
Mini Challenge
        ↓
정상 Baseline 복원
        ↓
Report / README / Git
        ↓
11일차 Test
```

---

## 오늘의 성공 기준

```text
9일차 Final Pipeline PASS
        ↓
runtime/edge_state.json 갱신 PASS
        ↓
Headless Edge Runtime PASS
        ↓
운영 Event Log 새 Row 생성 PASS
        ↓
subject13-edge.service active
        ↓
재부팅 후 자동실행 PASS
        ↓
SIGKILL 후 자동재시작 PASS
        ↓
subject13-api.service active
        ↓
/health /status /metrics PASS
        ↓
PC에서 API 접근 PASS
        ↓
API 장애 중 Edge Runtime 독립 동작 PASS
        ↓
장애 기록 / 복구 PASS
        ↓
Mini Challenge PASS
        ↓
두 Service enabled + active
        ↓
11일차 Handoff 준비
```

---

## 오늘은 아직 하지 않는 것

```text
새 AI 모델 학습
→ 6일차에서 완료

ONNX 변환과 Raspberry Pi 배포
→ 7일차에서 완료

정식 Latency / FPS Benchmark
→ 8일차에서 완료

Frame Skip / Voting / Debounce / Hold 선택
→ 9일차에서 완료

pytest 기반 Unit Test 체계
Integration Test
System Test
Failure Dataset 분석
→ 11일차

Jetson 확장
→ 12일차
```

오늘은 **“실행되는 AI”를 “운영할 수 있는 AI”로 바꾸는 날**입니다.

---

# PART A. 9일차 Final Pipeline을 이어받아 Headless 운영 구조 이해하기

# 1. 9일차 최종 상태에서 이어서 시작하기

9일차가 끝난 Raspberry Pi에는 다음 기능이 준비되어 있습니다.

```text
Camera Input
→ ONNX Runtime
→ Class + Confidence
→ Confidence Threshold
→ Frame Skip
→ Voting
→ Debounce
→ Hold
→ GPIO
```

9일차의 대표 실행 프로그램:

```text
scripts/optimized_ai_warning_system.py
```

9일차에서 선택한 운영 설정도 `configs/settings.yaml`에 있습니다.

예:

```yaml
inference_every_n_frames: 2

vote_window: 5
vote_warning_min: 3

debounce_required: 2
warning_hold_sec: 1.0
```

10일차에서는 이 값을 그대로 사용합니다.

---

# 2. Raspberry Pi에 접속하고 현재 상태 확인하기

PC에서 Raspberry Pi에 접속합니다.

```bash
ssh <RPI_USER>@<RPI_IP>
```

사용자 이름과 IP 주소는 자신의 장비에 맞게 변경합니다.

프로젝트로 이동합니다.

```bash
cd ~/ai_vision/subject13_edge_ai
```

9일차 최종 Source가 현재 Raspberry Pi에 있는지 먼저 확인합니다.

```bash
git status
git log --oneline -10
```

수업 환경에 따라 Source 동기화 방식이 다릅니다.

### 허용된 내부 GitLab / Gitea Remote가 있는 경우

```bash
git remote -v
```

Remote가 정상이고 기관 정책상 사용이 허용되어 있을 때만 다음을 실행합니다.

```bash
git pull
```

### Local Git만 사용하는 경우

Remote가 없다면 `git pull`을 억지로 실행하지 않습니다. 9일차에서 사용한 Source가 현재 Raspberry Pi에 있는지 실제 파일과 Commit으로 확인합니다. PC에서 수정한 파일을 가져와야 한다면 수업에서 정한 SCP 또는 내부 파일 전달 방식을 사용합니다.

> `configs/device.local.yaml`은 Camera index처럼 Raspberry Pi마다 달라지는 장치 전용 설정이므로 다른 PC의 파일로 덮어쓰지 않습니다.

가상환경을 활성화합니다.

```bash
source .venv/bin/activate
```

현재 Python:

```bash
which python
```

장치 확인:

```bash
python -m scripts.check_device
```

ONNX:

```bash
python -m scripts.inspect_onnx_runtime
```

Camera:

```bash
python -m scripts.camera_index_probe
```

GPIO:

```bash
python -m scripts.gpio_test
```

9일차 최적화 프로그램도 짧게 실행합니다.

```bash
python -m scripts.optimized_ai_warning_system \
  --run-name day10_precheck \
  --duration 20
```

정상이라면 10일차 운영 구조를 추가합니다.

---

# 3. Headless 실행에서 무엇이 달라질까요?

지금까지는 사람이 SSH Terminal을 보고 있었습니다.

```text
SSH 접속
→ Python 직접 실행
→ Terminal 출력 확인
```

현장 장치에서는 모니터와 키보드가 없을 수 있습니다.

이런 상태를 **Headless 운영**이라고 생각할 수 있습니다.

Headless에서는 다음에 의존하지 않습니다.

```text
cv2.imshow()
GUI 창
사람이 계속 보고 있는 Terminal
```

대신 다음 정보로 상태를 확인합니다.

```text
systemctl status
journalctl
Runtime State
GPIO 상태
FastAPI /health
FastAPI /status
FastAPI /metrics
```

---

# 4. 오늘 만들 운영 구조 먼저 보기

10일차가 끝나면 두 개의 독립 프로그램이 동작합니다.

```text
                 [Raspberry Pi]

USB Camera
     ↓
Edge Runtime
     ↓
ONNX AI
     ↓
Stabilization
     ↓
GPIO
     │
     ├────→ AI Event Log
     │
     └────→ runtime/edge_state.json
                       │
                       ▼
                    FastAPI
                  ┌────┼─────┐
                  ↓    ↓     ↓
              /health /status /metrics
                       │
                       ▼
                       PC
```

그리고 두 프로그램을 각각 systemd가 관리합니다.

```text
subject13-edge.service
→ Edge Runtime

subject13-api.service
→ FastAPI
```

두 Service를 분리하는 이유는 간단합니다.

```text
FastAPI 장애
→ Edge AI는 계속 동작

Network 장애
→ Edge AI는 계속 동작

Edge AI 장애
→ FastAPI는 장애 상태를 보여줄 수 있음
```

---

# 5. 10일차 운영 설정 추가하기

`configs/settings.yaml`에 다음 항목을 추가합니다.

```yaml
runtime_state_path: runtime/edge_state.json
runtime_heartbeat_sec: 1.0
runtime_stale_sec: 5.0

api_host: 0.0.0.0
api_port: 8000

service_restart_sec: 3
```

각 값의 역할:

```text
runtime_state_path
→ Edge Runtime의 최신 상태 파일

runtime_heartbeat_sec
→ 상태파일을 갱신하는 간격

runtime_stale_sec
→ 몇 초 이상 갱신되지 않으면 오래된 상태로 볼 것인가?

api_host
→ 외부 PC에서 접근할 수 있도록 0.0.0.0 사용

api_port
→ FastAPI Port

service_restart_sec
→ systemd 자동 재시작 대기시간
```

### 설정값 사이의 관계도 확인합니다

예시값은 출발점일 뿐입니다.

```text
runtime_heartbeat_sec
→ 0보다 커야 함

runtime_stale_sec
→ 보통 heartbeat보다 충분히 크게 설정
→ Heartbeat가 한 번 늦었다고 바로 장애로 오판하지 않도록 함

api_port
→ 1~65535 범위의 TCP Port
→ 일반 사용자 Service에서는 1024 이상의 비특권 Port를 권장
→ 현재 다른 Process가 사용 중이지 않은 Port 사용
→ 교육장 방화벽 / Network 정책에 따라 외부 접근 허용 여부가 달라질 수 있음

api_host = 0.0.0.0
→ 같은 내부 Network의 다른 PC에서도 접근할 수 있게 Listen

api_host = 127.0.0.1
→ Raspberry Pi 자기 자신에서만 접근
```

오늘 기본 예시로 `api_port: 8000`을 사용하지만, 충돌하거나 교육장 정책이 다르면 다른 Port를 선택할 수 있습니다. 중요한 것은 `settings.yaml`, Uvicorn 실행 명령, systemd Unit, PC Client가 **같은 Port**를 사용하도록 맞추는 것입니다.

---

# 6. Runtime 파일은 Git에서 제외하기

실행 중 계속 바뀌는 상태파일은 Source Code가 아닙니다.

`.gitignore`에 다음을 추가합니다.

```gitignore
# Runtime state
runtime/

# Device-local generated systemd units
systemd/generated/
```

구분:

```text
Source Code
→ Git 관리

runtime/edge_state.json
→ 장치 실행 중 생성
→ Git 제외

systemd/generated/*.service
→ 현재 사용자 / 절대경로 / .venv Python이 들어가는 장치별 생성물
→ Git 제외
→ 필요할 때 render_systemd_units.py로 다시 생성
```

---

# 7. Edge Runtime과 FastAPI가 정보를 공유하는 방법

두 프로그램을 하나의 Python Process에 억지로 넣지 않습니다.

Edge Runtime은 최신 상태를 JSON 파일에 기록합니다.

예:

```json
{
  "service": "edge",
  "status": "RUNNING",
  "hostname": "rpi13-01",
  "timestamp": "2026-10-12T10:15:31.125+00:00",
  "state": "NORMAL",
  "raw_state": "NORMAL",
  "pred_class": "normal",
  "confidence": 0.943,
  "loop_fps": 18.2,
  "inference_per_sec": 9.1,
  "mean_inference_ms": 12.4,
  "p95_inference_ms": 15.8,
  "cpu_percent": 42.0,
  "memory_percent": 23.1,
  "temperature_c": 51.7
}
```

FastAPI는 이 파일을 읽어서 PC에 전달합니다.

```text
Edge Runtime
→ JSON State Write

FastAPI
→ JSON State Read
→ HTTP Response
```

---

# PART B. Runtime State와 Headless Edge Runtime 만들기

# 8. Runtime State 저장 모듈 만들기

## 이 파일은 왜 필요할까요?

9일차까지는 사람이 Terminal을 보면서 현재 상태를 확인할 수 있었습니다. 그러나 10일차부터는 프로그램이 `systemd` 아래에서 Headless로 실행됩니다. 따라서 **현재 Edge Runtime이 살아 있는지, 최근 AI 결과가 무엇인지, 성능·자원 상태가 어떤지 다른 프로그램이 읽을 수 있는 형태로 남겨야 합니다.**

`src/runtime_state.py`는 이 역할만 담당합니다.

```text
Edge Runtime
        ↓
현재 상태 Dictionary
        ↓
RuntimeStateStore.write()
        ↓
runtime/edge_state.json

FastAPI
        ↓
RuntimeStateStore.read()
        ↓
JSON 상태 읽기
```

이 파일은 Camera를 열거나 AI를 실행하지 않습니다. **상태를 안전하게 저장하고 읽는 공통 기능**만 담당합니다.

## 의사코드

```text
현재 UTC 시각을 ISO 문자열로 만든다
        ↓
Runtime State를 저장할 경로를 준비한다
        ↓
상태 Dictionary를 복사한다
        ↓
timestamp를 추가한다
        ↓
임시 .tmp 파일에 완성된 JSON을 먼저 쓴다
        ↓
os.replace()로 실제 State 파일을 교체한다

읽을 때
        ↓
파일이 없으면 빈 Dictionary 반환
        ↓
JSON이 정상이라면 Dictionary 반환
        ↓
읽기 중 오류가 나면 빈 Dictionary 반환
```

파일:

```text
src/runtime_state.py
```

코드:

```python
import json
import os
from datetime import datetime, timezone
from pathlib import Path


def utc_now_iso() -> str:
    return datetime.now(
        timezone.utc
    ).isoformat(
        timespec="milliseconds"
    )


class RuntimeStateStore:
    def __init__(
        self,
        path: str,
    ):
        self.path = Path(path)

        self.path.parent.mkdir(
            parents=True,
            exist_ok=True,
        )

    def write(
        self,
        data: dict,
    ) -> None:
        payload = dict(data)

        payload["timestamp"] = (
            utc_now_iso()
        )

        temp_path = self.path.with_suffix(
            self.path.suffix + ".tmp"
        )

        temp_path.write_text(
            json.dumps(
                payload,
                ensure_ascii=False,
                indent=2,
            ),
            encoding="utf-8",
        )

        os.replace(
            temp_path,
            self.path,
        )

    def read(self) -> dict:
        if not self.path.exists():
            return {}

        try:
            return json.loads(
                self.path.read_text(
                    encoding="utf-8"
                )
            )

        except Exception:
            return {}
```

---

# 9. 왜 임시 파일을 만든 뒤 교체하나요?

다음처럼 직접 파일에 쓰는 중:

```text
FastAPI가 읽기 시작
```

하면 아직 JSON이 완성되지 않은 순간을 읽을 수 있습니다.

오늘 코드는:

```text
edge_state.json.tmp
→ 완성된 JSON 작성
→ os.replace()
→ edge_state.json 교체
```

순서로 기록합니다.

이렇게 하면 FastAPI가 **중간까지 작성된 JSON을 읽는 가능성**을 줄일 수 있습니다.

---

# 10. Runtime State 모듈만 먼저 테스트하기

## 이 파일은 왜 필요할까요?

Camera·ONNX·GPIO·systemd를 한꺼번에 연결하기 전에 방금 만든 `RuntimeStateStore`가 **혼자서도 정상적으로 쓰고 읽을 수 있는지** 먼저 확인합니다. 작은 기능을 먼저 검증하면 이후 오류가 생겼을 때 원인을 좁히기 쉽습니다.

```text
가짜 상태 Dictionary
        ↓
RuntimeStateStore.write()
        ↓
runtime/edge_state.json
        ↓
RuntimeStateStore.read()
        ↓
Terminal 출력
```

## 의사코드

```text
settings.yaml을 읽는다
        ↓
runtime_state_path를 가져온다
        ↓
RuntimeStateStore를 만든다
        ↓
TEST 상태를 저장한다
        ↓
같은 파일을 다시 읽는다
        ↓
Key와 값을 화면에 출력한다
        ↓
cat 명령으로 실제 JSON 파일도 확인한다
```

파일:

```text
scripts/runtime_state_test.py
```

코드:

```python
from src.config_loader import load_config
from src.runtime_state import (
    RuntimeStateStore,
)


def main():
    config = load_config(
        "configs/settings.yaml"
    )

    store = RuntimeStateStore(
        config["runtime_state_path"]
    )

    store.write(
        {
            "service": "edge",
            "status": "TEST",
            "state": "NORMAL",
            "pred_class": "normal",
            "confidence": 0.95,
            "loop_fps": 20.0,
            "inference_per_sec": 10.0,
            "mean_inference_ms": 10.0,
            "p95_inference_ms": 12.0,
        }
    )

    data = store.read()

    print("=== Runtime State ===")

    for key, value in data.items():
        print(
            f"{key:16s}: {value}"
        )


if __name__ == "__main__":
    main()
```

실행:

```bash
python -m scripts.runtime_state_test
```

파일 확인:

```bash
cat runtime/edge_state.json
```


### 예상 실행 결과와 해석

터미널에는 다음과 비슷한 형태가 보입니다.

```text
=== Runtime State ===
service         : edge
status          : TEST
state           : NORMAL
pred_class      : normal
confidence      : 0.95
loop_fps        : 20.0
inference_per_sec: 10.0
mean_inference_ms: 10.0
p95_inference_ms: 12.0
timestamp       : 2026-10-xxTxx:xx:xx.xxx+00:00
```

정확한 `timestamp`는 실행 시각마다 달라집니다.

`cat runtime/edge_state.json`에서 같은 값이 JSON으로 보이면 다음 연결이 정상입니다.

```text
Python Dictionary
→ RuntimeStateStore.write()
→ JSON 파일
→ RuntimeStateStore.read()
→ Python Dictionary
```

### 코드 리뷰 — 이 결과는 어디에서 만들어졌을까요?

코드를 직접 열어 다음을 찾습니다.

| 결과 | 담당 위치 |
|---|---|
| `status: TEST` | `scripts/runtime_state_test.py`의 `store.write({...})` |
| `timestamp` | `src/runtime_state.py`의 `utc_now_iso()`와 `write()` |
| `.tmp` 파일 작성 | `RuntimeStateStore.write()` |
| 실제 파일 교체 | `os.replace()` |
| JSON 다시 읽기 | `RuntimeStateStore.read()` |
| 터미널 한 줄 출력 | `scripts/runtime_state_test.py`의 `for key, value ...` |

다음 질문에 답해 봅니다.

```text
1. runtime_state_test.py가 직접 JSON 문자열을 조립하는가?
2. timestamp를 넣는 파일은 어느 파일인가?
3. State 파일이 없을 때 read()는 무엇을 반환하는가?
4. 잘못된 JSON을 읽으면 현재 구현은 무엇을 반환하는가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
1. 아니다. Dictionary를 RuntimeStateStore.write()에 전달한다.
2. src/runtime_state.py이다.
3. 빈 Dictionary {}를 반환한다.
4. 읽기 오류를 잡고 빈 Dictionary {}를 반환한다.
```

FastAPI는 뒤에서 빈 Dictionary를 보고 Edge 상태를 정상이라고 오판하지 않도록 `DEGRADED`로 해석합니다.

</details>

### Guided Error — State 경로가 잘못되었는지 확인하기

오류를 무작정 만들기보다 먼저 정상 상태를 확인한 뒤 한 가지 조건만 바꿉니다.

1. `configs/settings.yaml`의 현재 `runtime_state_path`를 메모합니다.
2. 임시로 다음처럼 바꿉니다.

```yaml
runtime_state_path: runtime_test/day10_state.json
```

3. 다시 실행합니다.

```bash
python -m scripts.runtime_state_test
```

4. 새 경로에 파일이 생겼는지 확인합니다.

```bash
find runtime runtime_test -maxdepth 2 -type f -print 2>/dev/null
```

이 실습은 오류가 아니라 **설정값이 실제 출력 경로를 바꾼다는 것**을 확인하는 Guided Lab입니다.

확인이 끝나면 반드시 원래 값으로 복원합니다.

```yaml
runtime_state_path: runtime/edge_state.json
```


---

# 11. Headless용 Edge Runtime 만들기

9일차의 `scripts/optimized_ai_warning_system.py`를 삭제하지 않습니다. 그 파일은 **60초 같은 정해진 실험시간 동안 설정을 비교하고 Summary를 남기는 실험용 프로그램**입니다.

10일차에는 목적이 다릅니다.

```text
9일차 optimized_ai_warning_system.py
→ 실험용
→ 정해진 Duration
→ 설정 조합 비교

10일차 edge_runtime.py
→ 운영용
→ 종료시간 없음
→ 계속 Camera / AI / GPIO 실행
→ 일정 간격으로 Runtime State 갱신
→ systemd가 Process 관리
```

즉, 이전 코드를 억지로 덮어쓰지 않고 역할이 다른 운영용 시작점을 추가합니다.

## 이 파일은 왜 필요할까요?

`systemd`는 결국 **어떤 Python 프로그램을 계속 실행할 것인지** 알아야 합니다. `scripts/edge_runtime.py`가 그 실행 대상입니다.

오늘 새로 만드는 운영 흐름은 다음과 같습니다.

```text
configs/settings.yaml
configs/device.local.yaml
        ↓
src/config_loader.py
        ↓
scripts/edge_runtime.py
        │
        ├─ src/camera_input.py
        ├─ src/ai_preprocess.py
        ├─ src/inference_onnx.py
        ├─ src/decision.py
        ├─ src/stabilizer.py
        ├─ src/gpio_output.py
        ├─ src/event_logger.py
        ├─ src/performance.py
        ├─ src/resource_monitor.py
        └─ src/runtime_state.py
        ↓
Camera → ONNX → Stabilization → GPIO
        ├─→ AI Event CSV
        └─→ runtime/edge_state.json
```

위의 대부분 파일은 이전 일차에서 만든 파일을 **다시 사용**합니다. 오늘 새 핵심은 `edge_runtime.py`가 이들을 운영용으로 연결하고 `RuntimeStateStore`를 주기적으로 갱신한다는 점입니다.

## 의사코드

```text
Config를 읽는다
        ↓
Runtime State Store 준비
        ↓
STARTING 상태 기록
        ↓
9일차 Final Config 읽기
Frame Skip / Voting / Debounce / Hold
        ↓
Camera / ONNX / GPIO 준비
        ↓
7일차 AIEventLogger를 기존 `ai_event_log_path`로 준비
        ↓
무한 반복
        ↓
Camera Frame 읽기
        ↓
이번 Frame이 추론 차례인가?
        ├─ YES → Preprocess → ONNX → Decision → Stabilizer
        └─ NO  → 직전 최종 상태 유지
        ↓
GPIO 출력
        ↓
최종 운영 State가 이전 기록 State와 달라졌는가?
        ├─ YES → AIEventLogger.append()로 Event CSV 한 줄 기록
        └─ NO  → 중복 Event를 기록하지 않음
        ↓
Heartbeat 시간이 되었는가?
        ├─ YES
        │   → FPS / Inference/sec 계산
        │   → 최근 Latency 통계 계산
        │   → CPU / Memory / Temperature 읽기
        │   → runtime/edge_state.json 갱신
        └─ NO
            → 다음 Frame
        ↓
오류 발생
        → ERROR 상태 기록
        → 오류를 다시 발생시켜 systemd가 실패를 감지하게 함
        ↓
종료 시 Camera / GPIO 정리
```

파일:

```text
scripts/edge_runtime.py
```

코드:

```python
import platform
import time
from collections import deque
from datetime import datetime

from src.ai_preprocess import (
    bgr_frame_to_nchw,
)
from src.camera_input import CameraInput
from src.config_loader import load_config
from src.decision import (
    ai_to_operational_state,
)
from src.event_logger import AIEventLogger
from src.gpio_output import GPIOOutput
from src.inference_onnx import (
    ONNXClassifier,
)
from src.performance import (
    summarize_latency,
)
from src.resource_monitor import (
    get_cpu_percent,
    get_memory_percent,
    get_temperature_c,
)
from src.runtime_state import (
    RuntimeStateStore,
)
from src.stabilizer import (
    TemporalStabilizer,
)


def main():
    config = load_config(
        "configs/settings.yaml"
    )

    store = RuntimeStateStore(
        config["runtime_state_path"]
    )

    camera = None
    output = None

    try:
        store.write(
            {
                "service": "edge",
                "status": "STARTING",
                "hostname":
                    platform.node(),
            }
        )

        every_n = int(
            config[
                "inference_every_n_frames"
            ]
        )

        if every_n < 1:
            raise ValueError(
                "inference_every_n_frames는 "
                "1 이상이어야 합니다."
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

        classifier = ONNXClassifier(
            config["onnx_model_path"],
            config["onnx_meta_path"],
        )

        output = GPIOOutput(
            green_pin=int(
                config["green_led_bcm"]
            ),
            red_pin=int(
                config["red_led_bcm"]
            ),
            buzzer_pin=int(
                config["buzzer_bcm"]
            ),
            use_buzzer=bool(
                config.get(
                    "use_buzzer",
                    False,
                )
            ),
        )

        stabilizer = TemporalStabilizer(
            vote_window=int(
                config["vote_window"]
            ),
            vote_warning_min=int(
                config["vote_warning_min"]
            ),
            debounce_required=int(
                config[
                    "debounce_required"
                ]
            ),
            warning_hold_sec=float(
                config["warning_hold_sec"]
            ),
        )

        warning_class = (
            config["ai_warning_class"]
        )

        confidence_threshold = float(
            config[
                "ai_confidence_threshold"
            ]
        )

        event_logger = AIEventLogger(
            config["ai_event_log_path"]
        )

        heartbeat_sec = float(
            config[
                "runtime_heartbeat_sec"
            ]
        )

        if heartbeat_sec <= 0:
            raise ValueError(
                "runtime_heartbeat_sec는 "
                "0보다 커야 합니다."
            )

        frame_count = 0
        inference_count = 0

        last_state = "NORMAL"
        last_raw_state = "NORMAL"

        last_prediction = {
            "class_name": "normal",
            "confidence": 0.0,
        }

        # Event Log는 첫 운영 State와 이후 State 변화만 기록합니다.
        previous_logged_state = None

        latency_window = deque(
            maxlen=100
        )

        loop_started = (
            time.perf_counter()
        )

        last_heartbeat = 0.0

        print(
            "Edge Runtime started.",
            flush=True,
        )

        while True:
            frame = camera.read()

            frame_count += 1

            should_infer = (
                (frame_count - 1)
                % every_n
                == 0
            )

            if should_infer:
                tensor = (
                    bgr_frame_to_nchw(
                        frame,
                        classifier.image_size,
                    )
                )

                infer_started = (
                    time.perf_counter()
                )

                prediction = (
                    classifier.predict(
                        tensor
                    )
                )

                inference_ms = (
                    time.perf_counter()
                    - infer_started
                ) * 1000.0

                latency_window.append(
                    inference_ms
                )

                raw_state = (
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
                            confidence_threshold,
                    )
                )

                stabilized = (
                    stabilizer.update(
                        raw_state,
                        time.monotonic(),
                    )
                )

                last_state = (
                    stabilized[
                        "output_state"
                    ]
                )

                last_raw_state = (
                    raw_state
                )

                last_prediction = (
                    prediction
                )

                inference_count += 1

            if last_state == "WARNING":
                output.warning()
            else:
                output.normal()

            # Day 07의 AIEventLogger를 운영 Runtime에서도 재사용합니다.
            # raw_state가 아니라 Voting / Debounce / Hold가 반영된
            # 실제 최종 출력 상태(last_state)의 변화만 기록합니다.
            if last_state != previous_logged_state:
                timestamp = datetime.now().isoformat(
                    timespec="milliseconds"
                )

                event_logger.append(
                    timestamp=timestamp,
                    pred_class=str(
                        last_prediction[
                            "class_name"
                        ]
                    ),
                    confidence=float(
                        last_prediction[
                            "confidence"
                        ]
                    ),
                    threshold=confidence_threshold,
                    state=last_state,
                    image_path="",
                )

                previous_logged_state = last_state

            now = time.perf_counter()

            if (
                now - last_heartbeat
                >= heartbeat_sec
            ):
                elapsed = (
                    now - loop_started
                )

                loop_fps = (
                    frame_count / elapsed
                    if elapsed > 0
                    else 0.0
                )

                inference_per_sec = (
                    inference_count / elapsed
                    if elapsed > 0
                    else 0.0
                )

                if latency_window:
                    stats = (
                        summarize_latency(
                            list(
                                latency_window
                            )
                        )
                    )

                    mean_latency = (
                        stats.mean_ms
                    )

                    p95_latency = (
                        stats.p95_ms
                    )
                else:
                    mean_latency = 0.0
                    p95_latency = 0.0

                store.write(
                    {
                        "service": "edge",
                        "status": "RUNNING",
                        "hostname":
                            platform.node(),
                        "state":
                            last_state,
                        "raw_state":
                            last_raw_state,
                        "pred_class":
                            last_prediction[
                                "class_name"
                            ],
                        "confidence":
                            round(
                                float(
                                    last_prediction[
                                        "confidence"
                                    ]
                                ),
                                6,
                            ),
                        "confidence_threshold":
                            confidence_threshold,
                        "frame_count":
                            frame_count,
                        "inference_count":
                            inference_count,
                        "loop_fps":
                            round(
                                loop_fps,
                                3,
                            ),
                        "inference_per_sec":
                            round(
                                inference_per_sec,
                                3,
                            ),
                        "mean_inference_ms":
                            round(
                                mean_latency,
                                3,
                            ),
                        "p95_inference_ms":
                            round(
                                p95_latency,
                                3,
                            ),
                        "cpu_percent":
                            get_cpu_percent(),
                        "memory_percent":
                            get_memory_percent(),
                        "temperature_c":
                            get_temperature_c(),
                    }
                )

                last_heartbeat = now

    except KeyboardInterrupt:
        print(
            "Edge Runtime stopped.",
            flush=True,
        )

    except Exception as error:
        store.write(
            {
                "service": "edge",
                "status": "ERROR",
                "hostname":
                    platform.node(),
                "error_type":
                    type(error).__name__,
                "error_message":
                    str(error),
            }
        )

        print(
            f"Edge Runtime ERROR: "
            f"{type(error).__name__}: "
            f"{error}",
            flush=True,
        )

        raise

    finally:
        if camera is not None:
            camera.release()

        if output is not None:
            output.close()


if __name__ == "__main__":
    main()
```

### 운영 Event Log는 어떤 State를 기록하나요?

10일차에서는 7일차에 이미 만든 `AIEventLogger`를 **새로 만들지 않고 다시 사용**합니다.

```text
ONNX Prediction
        ↓
raw_state
        ↓
Voting / Debounce / Hold
        ↓
last_state = 최종 운영 State
        ↓
last_state가 이전 기록 State와 달라졌는가?
        ├─ YES → AIEventLogger.append()
        └─ NO  → 기록하지 않음
```

여기서 중요한 기준은 **`raw_state`가 아니라 실제 GPIO에 전달되는 최종 `last_state`를 Event Log의 `state`로 기록한다는 것**입니다.

예를 들어 AI의 Raw Prediction이 짧게 흔들렸지만 Voting·Debounce 때문에 최종 출력이 계속 `NORMAL`이라면 Event CSV에는 불필요한 WARNING Event를 추가하지 않습니다.

또한 10일차에서는 새로운 Log 설정 Key를 만들지 않습니다. 7일차부터 사용하던 다음 설정을 그대로 재사용합니다.

```yaml
ai_event_log_path: <현재 configs/settings.yaml에 기록된 실제 경로>
```

실제 값은 다음처럼 확인합니다.

```bash
grep -E "^ai_event_log_path:" \
configs/settings.yaml \
configs/device.local.yaml \
2>/dev/null
```

`AIEventLogger`의 `image_path`는 기본값처럼 빈 문자열로 기록합니다. 10일차 핵심은 **운영 State Event가 실제 Runtime에서 계속 기록되는지** 확인하는 것입니다. WARNING 대표 이미지 저장은 기존 기능을 사용하는 경우에만 별도로 유지합니다.

---

# 12. Headless Edge Runtime을 Terminal에서 먼저 실행하기

systemd에 등록하기 전에 직접 실행합니다.

```bash
python -m scripts.edge_runtime
```

화면에 Camera 창은 나타나지 않습니다.

정상이라면 Terminal에는 시작 메시지만 보이고 프로그램은 계속 실행됩니다.

다른 SSH Terminal을 하나 더 열어 상태파일을 확인합니다.

```bash
cd ~/ai_vision/subject13_edge_ai
```

```bash
cat runtime/edge_state.json
```

약 1초 간격으로 값이 갱신되는지 확인합니다.

운영 Event Log 경로도 확인합니다.

```bash
grep -E "^ai_event_log_path:" \
configs/settings.yaml \
configs/device.local.yaml \
2>/dev/null
```

출력된 실제 경로를 `<AI_EVENT_LOG_PATH>`에 넣어 최근 Event를 확인합니다.

```bash
tail -n 10 <AI_EVENT_LOG_PATH>
```

정상이라면 첫 운영 State가 한 줄 기록되고, 이후 최종 State가 바뀔 때 새 Row가 추가됩니다.

---

# 13. Runtime State가 실제로 변하는지 확인하기

다음 명령을 여러 번 실행합니다.

```bash
cat runtime/edge_state.json
```

빨간색 카드를 Camera 앞에 보여줍니다.

다음 값이 변하는지 확인합니다.

```text
pred_class
confidence
raw_state
state
```

예:

```text
normal
→ warning_red

NORMAL
→ WARNING
```

이제 같은 변화가 **현재 운영 Event Log에도 새 Row로 남았는지** 확인합니다.

변화시키기 전과 후에 다음을 실행합니다.

```bash
tail -n 10 <AI_EVENT_LOG_PATH>
```

예상 형태:

```csv
timestamp,mode,pred_class,confidence,threshold,state,image_path
...,ai,normal,0.94,0.700,NORMAL,
...,ai,warning_red,0.86,0.700,WARNING,
```

정확한 Confidence와 Timestamp는 실행할 때마다 달라집니다.

여기서 확인할 것은 다음입니다.

```text
Runtime State의 최종 state 변화
        ↓
GPIO 변화
        ↓
AI Event CSV에 같은 최종 State의 새 Row
```

과거 7일차에 만들어 둔 CSV가 단순히 존재하는지만 보지 않습니다. **10일차 `edge_runtime.py`를 실행한 뒤 새로운 Timestamp가 실제로 추가되었는지** 확인해야 합니다.

---

# 14. Edge Runtime 종료하기

직접 실행한 Terminal에서:

```text
Ctrl+C
```

로 종료합니다.

GPIO가 종료 상태로 정리되는지 확인합니다.


## 실행 전에 여러 파일의 연결을 다시 확인합니다

```text
configs/settings.yaml
configs/device.local.yaml
        ↓
config_loader.py
        ↓
edge_runtime.py
        │
        ├─ CameraInput
        ├─ ONNXClassifier
        ├─ ai_to_operational_state()
        ├─ TemporalStabilizer
        ├─ GPIOOutput
        ├─ AIEventLogger
        ├─ summarize_latency()
        ├─ resource_monitor
        └─ RuntimeStateStore
        ↓
        ├─ AI Event CSV
        └─ runtime/edge_state.json
```

`edge_runtime.py`에서 새로운 AI를 학습하지 않습니다. 이전 일차 기능을 운영용으로 연결합니다.

## 예상 실행 결과

직접 실행하면 화면에 GUI Camera 창이 뜨지 않아야 합니다.

```text
Edge Runtime started.
```

프로그램은 계속 실행됩니다. 별도 Terminal에서 `cat runtime/edge_state.json`을 실행했을 때 다음과 비슷한 Key가 보여야 합니다.

```json
{
  "service": "edge",
  "status": "RUNNING",
  "hostname": "rpi13-xx",
  "state": "NORMAL",
  "raw_state": "NORMAL",
  "pred_class": "normal",
  "confidence": 0.93,
  "frame_count": 123,
  "inference_count": 62,
  "loop_fps": 20.1,
  "inference_per_sec": 10.1,
  "mean_inference_ms": 14.2,
  "p95_inference_ms": 17.6,
  "cpu_percent": 40.0,
  "memory_percent": 22.0,
  "temperature_c": 52.0,
  "timestamp": "..."
}
```

숫자는 장비마다 달라집니다. **Key가 존재하고 timestamp가 계속 바뀌는지**가 먼저 중요합니다.

운영 State가 바뀌었다면 Event CSV에도 새 Row가 추가되어야 합니다.

```csv
timestamp,mode,pred_class,confidence,threshold,state,image_path
...,ai,normal,0.940000,0.700,NORMAL,
...,ai,warning_red,0.860000,0.700,WARNING,
```

`AIEventLogger`는 모든 Frame을 기록하는 Logger가 아닙니다. **첫 운영 State와 이후 최종 운영 State가 바뀌는 순간만 기록**합니다.

### `frame_count`와 `inference_count`가 다른 이유

9일차 Final Config가 예를 들어 다음이라면:

```yaml
inference_every_n_frames: 2
```

모든 Camera Frame을 읽지만 약 두 Frame마다 한 번 AI 추론을 수행합니다.

```text
frame_count
→ Camera에서 읽은 Frame 수

inference_count
→ 실제 ONNX 추론을 수행한 횟수
```

따라서 두 값이 항상 같을 필요는 없습니다.

## 코드 리뷰 — Runtime State 한 줄을 역추적합니다

다음 항목이 어느 파일에서 만들어지는지 직접 찾습니다.

| Runtime State Key | 만들어지는 위치 |
|---|---|
| `pred_class` | `ONNXClassifier.predict()` 결과를 `edge_runtime.py`가 저장 |
| `raw_state` | `ai_to_operational_state()` 결과 |
| `state` | `TemporalStabilizer.update()`의 `output_state` |
| `loop_fps` | `edge_runtime.py`의 `frame_count / elapsed` |
| `mean_inference_ms` | `performance.py`의 `summarize_latency()` 결과 |
| `cpu_percent` | `resource_monitor.py` |
| Event CSV의 `state` | `edge_runtime.py`의 `last_state != previous_logged_state` → `AIEventLogger.append()` |
| Event CSV의 `pred_class` / `confidence` | 마지막 `ONNXClassifier.predict()` 결과 |
| `timestamp` | Runtime State는 `RuntimeStateStore.write()`, Event CSV는 `edge_runtime.py`의 `datetime.now()` |

코드를 보고 다음 질문에 답합니다.

```text
1. Frame Skip 판단은 어느 파일에서 수행되는가?
2. Stabilizer는 어느 시점에 update()되는가?
3. Skip된 Frame에서는 마지막 State가 어떻게 처리되는가?
4. Runtime State는 매 Frame마다 쓰는가?
5. Event Log는 매 Frame마다 쓰는가?
6. Event Log의 `state`에는 `raw_state`와 `last_state` 중 무엇을 기록하는가?
7. 오류가 발생하면 왜 다시 raise 하는가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
1. scripts/edge_runtime.py의 should_infer 계산이다.
2. 실제 AI 추론을 수행한 Frame에서 raw_state를 만든 뒤 update()한다.
3. 직전 last_state를 유지한다.
4. 아니다. runtime_heartbeat_sec 간격을 기준으로 갱신한다.
5. 아니다. 첫 운영 State와 이후 최종 State가 바뀌는 순간만 기록한다.
6. `last_state`를 기록한다. Voting / Debounce / Hold까지 반영되어 실제 GPIO에 전달되는 운영 상태이기 때문이다.
7. ERROR State를 기록한 뒤 Process를 실패 상태로 종료하여
   뒤에서 systemd의 Restart=on-failure가 실패를 감지하게 하기 위해서이다.
```

</details>

## 초보자가 자주 만나는 오류

### 오류 A — `KeyError: 'runtime_heartbeat_sec'`

```text
오류
→ settings.yaml에 10일차 Key가 없음

원인 확인
→ grep runtime_heartbeat_sec configs/settings.yaml

수정
→ Key 추가

재실행
→ python -m scripts.edge_runtime
```

### 오류 B — Camera를 열 수 없음

```text
오류
→ Camera 초기화 실패

원인 확인
→ Camera 연결 / Camera index / 다른 Process 점유 확인

수정
→ scripts.camera_index_probe 재확인
→ 필요하면 configs/device.local.yaml의 실제 Camera index 수정

재실행
→ python -m scripts.edge_runtime
```

### 오류 C — ONNX 파일 없음

```text
오류
→ onnx_model_path가 실제 파일과 다름

원인 확인
→ grep onnx_model_path configs/settings.yaml
→ ls -lh models/

수정
→ 경로 또는 파일 복구

재실행
→ python -m scripts.edge_runtime
```


이제 사람이 직접 실행하는 상태에서 systemd 관리로 넘어갑니다.

---

# PART C. systemd로 Edge Runtime을 자동 실행하고 복구하기

# 15. systemd가 무엇을 관리하게 되나요?

지금까지:

```text
SSH
→ source .venv/bin/activate
→ python -m scripts.edge_runtime
```

을 사람이 직접 실행했습니다.

systemd를 사용하면:

```text
Raspberry Pi Boot
        ↓
systemd
        ↓
정해진 Python 실행
        ↓
Edge Runtime
```

으로 바뀝니다.

중요한 점:

> systemd에서는 사람이 `source .venv/bin/activate`를 실행하지 않습니다.

대신 `.venv/bin/python`의 **절대 경로**를 직접 지정합니다.

---

# 16. systemd 파일을 저장할 프로젝트 폴더 만들기

프로젝트 안에 다음 폴더를 추가합니다.

```bash
mkdir -p systemd/generated
```

구조:

```text
subject13_edge_ai/
└── systemd/
    └── generated/
```

여기에 생성한 Service 파일을 먼저 확인한 뒤 `/etc/systemd/system/`으로 복사합니다.

---

# 17. 현재 사용자와 절대 경로 확인하기

다음 값을 확인합니다.

```bash
whoami
```

```bash
pwd
```

```bash
which python
```

예:

```text
User
edgepi

Project
/home/edgepi/ai_vision/subject13_edge_ai

Python
/home/edgepi/ai_vision/subject13_edge_ai/.venv/bin/python
```

systemd에서는 이 절대 경로가 중요합니다.

---

# 18. Service 파일을 자동으로 만들어 주는 Script 작성하기

장비마다 사용자 이름, 프로젝트 절대경로, 가상환경 Python 경로가 다를 수 있습니다. 따라서 교재의 `/home/어떤사용자/...`를 그대로 복사한 Service 파일을 사용하지 않고 **현재 Raspberry Pi 환경을 읽어 Unit 파일을 생성**합니다.

## 이 파일은 왜 필요할까요?

`systemd`는 SSH Terminal의 현재 상태를 모릅니다. 따라서 다음 값을 Unit 파일 안에 명확히 적어야 합니다.

```text
User=
→ 어느 Linux 사용자 권한으로 실행할 것인가?

WorkingDirectory=
→ 상대경로 configs/... models/... 를 어디 기준으로 찾을 것인가?

ExecStart=
→ 어떤 Python으로 어떤 Module을 실행할 것인가?
```

특히 `.venv/bin/python`을 사용해야 합니다. `sys.executable`이 가상환경 Python을 가리킬 때 **그 경로 자체를 보존**합니다. `Path(...).resolve()`로 심볼릭 링크를 끝까지 따라가면 환경에 따라 `/usr/bin/python...`으로 바뀌어 가상환경 Package를 놓칠 수 있으므로 오늘 생성기에서는 사용하지 않습니다.

## 의사코드

```text
settings.yaml 읽기
        ↓
현재 Project Root 확인
        ↓
현재 sys.executable 확인
        ↓
현재 Linux 사용자 확인
        ↓
Restart 대기시간 읽기
        ↓
Edge Unit 문자열 만들기
        ↓
API Unit 문자열 만들기
        ↓
systemd/generated/ 아래에 저장
        ↓
생성된 User / WorkingDirectory / ExecStart를 눈으로 검토
```

파일:

```text
scripts/render_systemd_units.py
```

코드:

```python
import getpass
import sys
from pathlib import Path

from src.config_loader import load_config


def main():
    config = load_config(
        "configs/settings.yaml"
    )

    project_root = (
        Path.cwd().resolve()
    )

    # sys.executable은 활성화된 .venv의 Python 경로입니다.
    # resolve()로 심볼릭 링크를 끝까지 따라가면
    # /usr/bin/python... 으로 바뀔 수 있으므로
    # venv 경로 자체를 그대로 사용합니다.
    python_path = Path(sys.executable).absolute()

    user = getpass.getuser()

    output_dir = (
        project_root
        / "systemd"
        / "generated"
    )

    output_dir.mkdir(
        parents=True,
        exist_ok=True,
    )

    restart_sec = int(
        config["service_restart_sec"]
    )

    edge_unit = f"""[Unit]
Description=Subject13 Edge AI Runtime
After=local-fs.target

[Service]
Type=simple
User={user}
WorkingDirectory={project_root}
ExecStart={python_path} -m scripts.edge_runtime

Restart=on-failure
RestartSec={restart_sec}

# systemctl stop 때 Python의 finally가 실행될 수 있도록
# SIGINT를 사용합니다.
KillSignal=SIGINT
Environment=PYTHONUNBUFFERED=1

[Install]
WantedBy=multi-user.target
"""

    api_unit = f"""[Unit]
Description=Subject13 Edge AI Status API
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User={user}
WorkingDirectory={project_root}
ExecStart={python_path} -m uvicorn scripts.status_api:app --host {config["api_host"]} --port {config["api_port"]}

Restart=on-failure
RestartSec={restart_sec}

Environment=PYTHONUNBUFFERED=1

[Install]
WantedBy=multi-user.target
"""

    edge_path = (
        output_dir
        / "subject13-edge.service"
    )

    api_path = (
        output_dir
        / "subject13-api.service"
    )

    edge_path.write_text(
        edge_unit,
        encoding="utf-8",
    )

    api_path.write_text(
        api_unit,
        encoding="utf-8",
    )

    print("=== systemd Units ===")
    print(f"User    : {user}")
    print(f"Project : {project_root}")
    print(f"Python  : {python_path}")
    print()
    print(edge_path)
    print(api_path)


if __name__ == "__main__":
    main()
```

FastAPI 파일은 아직 만들지 않았기 때문에 우선 Edge Service만 설치합니다.


## Guided Lab — 생성된 Unit을 설치하기 전에 읽어보기

`cat` 결과에서 다음 줄을 직접 찾습니다.

```text
User=<현재 Raspberry Pi 사용자>

WorkingDirectory=<현재 subject13_edge_ai 절대경로>

ExecStart=<현재 프로젝트의 .venv/bin/python> -m scripts.edge_runtime

Restart=on-failure

RestartSec=<settings.yaml 값>

KillSignal=SIGINT
```

특히 `ExecStart`가 `/usr/bin/python...`이 아니라 **프로젝트의 `.venv/bin/python`을 가리키는지** 확인합니다.

```bash
which python
```

결과와 Unit의 Python 경로를 비교합니다.

### 왜 `source .venv/bin/activate`가 Unit에 없을까요?

SSH Terminal에서는 사람이 Shell 환경을 활성화했습니다.

```text
source .venv/bin/activate
→ PATH가 바뀜
→ python이 .venv Python을 가리킴
```

systemd에서는 Shell을 사람이 준비하지 않습니다.

```text
ExecStart=/.../subject13_edge_ai/.venv/bin/python -m scripts.edge_runtime
```

처럼 **사용할 Python 자체를 정확하게 지정**합니다.

### 선택 확인 — Unit 문법 검사

환경에서 `systemd-analyze`를 사용할 수 있다면 다음처럼 생성된 Unit의 기본 문법을 확인할 수 있습니다.

```bash
systemd-analyze verify \
  systemd/generated/subject13-edge.service
```

경고가 나오면 무조건 무시하지 말고 Unit 파일의 경로·Directive 이름을 먼저 확인합니다.


---

# 19. Edge Service 파일 생성하기

가상환경이 활성화된 상태에서 실행합니다.

> 이 생성 Script 자체는 `sudo`로 실행하지 않습니다. `sudo python ...`으로 실행하면 `User=root` 또는 다른 Python 경로가 들어갈 수 있습니다.

```bash
python -m scripts.render_systemd_units
```

파일 확인:

```bash
cat systemd/generated/subject13-edge.service
```

다음 세 값이 자신의 Raspberry Pi와 맞는지 반드시 확인합니다.

```text
User=

WorkingDirectory=

ExecStart=
```

예:

```text
User=edgepi

WorkingDirectory=/home/edgepi/ai_vision/subject13_edge_ai

ExecStart=/home/edgepi/ai_vision/subject13_edge_ai/.venv/bin/python -m scripts.edge_runtime
```

---

# 20. Edge Service를 systemd에 설치하기

파일을 복사합니다.

```bash
sudo cp \
systemd/generated/subject13-edge.service \
/etc/systemd/system/
```

systemd에게 새 Unit 파일을 다시 읽게 합니다.

```bash
sudo systemctl daemon-reload
```

Service를 시작합니다.

```bash
sudo systemctl start \
subject13-edge.service
```

---

# 21. Service 상태 확인하기

```bash
systemctl status \
subject13-edge.service \
--no-pager
```

정상이면 다음과 비슷한 상태를 찾습니다.

```text
Active: active (running)
```

프로그램을 사람이 직접 실행하지 않았는데 Edge Runtime이 동작하고 있는 것입니다.

Runtime State도 확인합니다.

```bash
cat runtime/edge_state.json
```

## 코드 리뷰 — `systemctl start`가 실제로 무엇을 실행했을까요?

다음 연결을 직접 역추적합니다.

```text
sudo systemctl start subject13-edge.service
        ↓
/etc/systemd/system/subject13-edge.service
        ↓
ExecStart=
<PROJECT>/.venv/bin/python -m scripts.edge_runtime
        ↓
scripts/edge_runtime.py
        ↓
Camera / ONNX / Stabilizer / GPIO
        ├─→ AI Event Log
        └─→ runtime/edge_state.json
```

다음 명령으로 **systemd가 실제로 읽고 있는 Unit**도 확인합니다.

```bash
systemctl cat subject13-edge.service
```

프로젝트 안의 `systemd/generated/...`만 보고 끝내지 않습니다. 설치 후에는 `/etc/systemd/system/`의 Unit이 실제 실행 기준입니다.

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

`systemctl start`가 Python 코드를 자동으로 찾는 것이 아닙니다. Unit 파일의 `ExecStart`가 실행 명령을 지정하고, `WorkingDirectory`가 상대경로의 기준을 지정하며, `User`가 권한을 결정합니다.

</details>


---

# 22. Start·Stop·Restart를 직접 해보기

중지:

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

다시 시작:

```bash
sudo systemctl start \
subject13-edge.service
```

재시작:

```bash
sudo systemctl restart \
subject13-edge.service
```

상태:

```bash
systemctl status \
subject13-edge.service \
--no-pager
```

이 네 명령의 역할을 구분합니다.

```text
start
→ 시작

stop
→ 의도적으로 중지

restart
→ 중지 후 다시 시작

status
→ 현재 상태 확인
```

---

# 23. journalctl에서 Edge Runtime 로그 보기

최근 50줄:

```bash
journalctl \
-u subject13-edge.service \
-n 50 \
--no-pager
```

실시간:

```bash
journalctl \
-u subject13-edge.service \
-f
```

종료:

```text
Ctrl+C
```

`print()`로 출력한 내용과 Python Error Traceback이 systemd Journal에 남습니다.

---

# 24. 부팅 시 자동실행 활성화하기

현재는 Service를 직접 Start했습니다.

부팅할 때 자동으로 실행되도록 설정합니다.

```bash
sudo systemctl enable \
subject13-edge.service
```

확인:

```bash
systemctl is-enabled \
subject13-edge.service
```

정상:

```text
enabled
```

---

# 25. 재부팅 전에 현재 상태 기록하기

```bash
systemctl status \
subject13-edge.service \
--no-pager
```

```bash
cat runtime/edge_state.json
```

그다음 Raspberry Pi를 재부팅합니다.

```bash
sudo reboot
```

SSH 연결이 끊기는 것은 정상입니다.

---

# 26. 재부팅 후 자동실행 확인하기

부팅이 완료될 때까지 기다립니다. 재부팅 뒤 DHCP 환경에서는 IPv4가 달라질 수 있으므로 이전 주소가 실패하면 무조건 같은 IP를 계속 사용하지 않습니다.

확인 가능한 방법은 환경에 따라 다릅니다.

```text
<RPI_HOSTNAME>.local
→ mDNS가 동작하는 환경에서 사용 가능

공유기 / 교육장 장비 목록
→ 현재 IPv4 재확인

직접 연결된 화면
→ hostname -I
```

현재 주소를 확인한 뒤 PC에서 다시 접속합니다.

```bash
ssh <RPI_USER>@<RPI_IP>
```

프로젝트:

```bash
cd ~/ai_vision/subject13_edge_ai
```

사람이 Python을 실행하지 않은 상태에서:

```bash
systemctl status \
subject13-edge.service \
--no-pager
```

확인합니다.

```text
Active: active (running)
```

Runtime State:

```bash
cat runtime/edge_state.json
```

Camera 앞에 경고 대상을 보여 GPIO가 반응하는지도 확인합니다.

---

# 27. 자동재시작과 의도적인 Stop은 다릅니다

Service 설정:

```text
Restart=on-failure
```

의 의미는:

```text
프로그램 비정상 종료
→ 자동 재시작
```

입니다.

하지만:

```bash
sudo systemctl stop \
subject13-edge.service
```

은 사람이 의도적으로 중지한 것이므로 systemd가 다시 시작시키지 않습니다.

또한 현재 `edge_runtime.py`는 의도적인 Stop 때 마지막 Runtime State를 즉시 삭제하지 않습니다. 따라서 `edge_state.json`에 마지막 `RUNNING` 값이 잠시 남아 있을 수 있습니다.

```text
Service Stop
→ 새 Heartbeat가 더 이상 기록되지 않음
→ state_age_sec 증가
→ runtime_stale_sec를 넘으면 API가 stale로 판단
```

즉, **파일이 남아 있다는 것과 Process가 살아 있다는 것은 다릅니다.** 이것이 Heartbeat와 stale 판정을 사용하는 이유입니다.

자동재시작을 확인하려면 **Crash 상황**을 만들어야 합니다.

---

# 28. 일부러 Program을 강제 종료하기

Service가 실행 중인지 확인합니다.

```bash
systemctl status \
subject13-edge.service \
--no-pager
```

강제 종료:

```bash
sudo systemctl kill \
-s SIGKILL \
subject13-edge.service
```

3~5초 기다립니다.

```bash
sleep 5
```

다시 확인합니다.

```bash
systemctl status \
subject13-edge.service \
--no-pager
```

`Restart=on-failure`가 동작했다면 다시:

```text
active (running)
```

상태가 되어야 합니다.

---

# 29. 자동재시작 기록을 journalctl에서 확인하기

```bash
journalctl \
-u subject13-edge.service \
-n 80 \
--no-pager
```

확인할 내용:

```text
Process 종료
→ Service 실패
→ Restart
→ 새로운 Process 시작
```

장애가 발생했다는 사실과 복구 과정이 Journal에 남습니다.

---

# 30. Camera 초기화 오류를 Headless 상태에서 확인하기

이 실습은 USB Camera를 무리하게 실행 중 반복 탈착하지 않습니다.

순서:

```text
Service Stop
→ Camera 분리
→ Service Start
→ 오류 확인
```

먼저:

```bash
sudo systemctl stop \
subject13-edge.service
```

Camera를 안전하게 분리합니다.

다시 시작:

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

Journal:

```bash
journalctl \
-u subject13-edge.service \
-n 50 \
--no-pager
```

Runtime State:

```bash
cat runtime/edge_state.json
```

예상되는 상태:

```text
status: ERROR
error_type: RuntimeError
Camera를 열 수 없습니다...
```

---

# 31. Camera 오류에서 systemd Restart도 확인하기

Camera가 계속 없으면 프로그램은 다음처럼 반복될 수 있습니다.

```text
Start
→ Camera Init Fail
→ Exit
→ systemd Restart
→ Camera Init Fail
→ ...
```

Journal에서 한두 번 정도 재시작되는 것을 확인합니다.

이 상태를 오래 방치하지 않습니다. Camera가 계속 없는 동안 재시작 Loop가 계속될 수 있으므로 먼저 Service를 중지합니다.

```bash
sudo systemctl stop \
  subject13-edge.service
```

Camera를 다시 연결합니다.

그다음:

```bash
sudo systemctl restart \
subject13-edge.service
```

정상 복구를 확인합니다.

---

# 32. Model 파일 오류도 확인하기

Service를 중지합니다.

```bash
sudo systemctl stop \
subject13-edge.service
```

Model 파일 이름을 임시로 바꿉니다.

```bash
mv \
models/day07_tiny_cnn.onnx \
models/day07_tiny_cnn.onnx.bak
```

Service 시작:

```bash
sudo systemctl start \
subject13-edge.service
```

확인:

```bash
journalctl \
-u subject13-edge.service \
-n 50 \
--no-pager
```

Runtime State:

```bash
cat runtime/edge_state.json
```

문제 영역:

```text
Model Path / Deployment File
```

재시작 Loop를 오래 두지 않도록 먼저 Service를 중지합니다.

```bash
sudo systemctl stop \
  subject13-edge.service
```

원래대로 복구합니다.

```bash
sudo systemctl stop \
subject13-edge.service
```

```bash
mv \
models/day07_tiny_cnn.onnx.bak \
models/day07_tiny_cnn.onnx
```

```bash
sudo systemctl start \
subject13-edge.service
```

---

# 33. Config 오류도 확인하기

Service를 중지합니다.

```bash
sudo systemctl stop \
subject13-edge.service
```

`configs/settings.yaml`에서 임시로:

```yaml
inference_every_n_frames: 0
```

로 변경합니다.

Service 시작:

```bash
sudo systemctl start \
subject13-edge.service
```

Journal:

```bash
journalctl \
-u subject13-edge.service \
-n 50 \
--no-pager
```

Runtime State:

```bash
cat runtime/edge_state.json
```

예상:

```text
ValueError
inference_every_n_frames는 1 이상...
```

재시작 Loop를 오래 두지 않도록 먼저 중지합니다.

```bash
sudo systemctl stop \
  subject13-edge.service
```

복구:

```yaml
inference_every_n_frames: 2
```

다시:

```bash
sudo systemctl restart \
subject13-edge.service
```

---

# PART D. FastAPI로 Edge Runtime 상태를 외부에 제공하기

# 34. FastAPI 설치하기

Edge Runtime이 다시 정상인지 확인합니다.

```bash
systemctl status \
subject13-edge.service \
--no-pager
```

가상환경을 활성화합니다.

```bash
source .venv/bin/activate
```

먼저 FastAPI와 Uvicorn이 이미 설치되어 있는지 확인합니다.

```bash
python -m pip show fastapi uvicorn
```

이미 설치되어 있다면 다시 설치하지 않아도 됩니다.

설치가 필요하고 인터넷 사용이 허용된 환경이라면 다음처럼 설치할 수 있습니다.

```bash
python -m pip install \
  fastapi \
  uvicorn
```

폐쇄망에서는 외부 PyPI에 접속할 수 없을 수 있습니다. 이 경우 교육장에서 제공한 **내부 Package Mirror 또는 미리 준비된 Wheel 파일**을 사용합니다. 정확한 Mirror 주소나 Wheel 경로는 교육장 안내값을 사용하며 교재에서 임의의 주소를 고정하지 않습니다.

확인:

```bash
python -c \
"import fastapi, uvicorn; print('FastAPI:', fastapi.__version__); print('Uvicorn:', uvicorn.__version__)"
```

Raspberry Pi Package 기록도 갱신합니다.

```bash
python -m pip freeze \
  > reports/day10_pi_environment.txt
```

10일차부터 FastAPI와 Uvicorn은 운영 Source의 실제 의존성이므로 현재 Raspberry Pi 실행환경을 기준으로 `requirements.txt`도 갱신합니다.

```bash
python -m pip freeze \
  > requirements.txt
```

확인합니다.

```bash
grep -E "^(fastapi|uvicorn)==" requirements.txt
```

> 교육장에서 `requirements.txt`를 PC용과 Raspberry Pi용으로 별도 관리하는 정책을 이미 사용하고 있다면 그 정책을 우선합니다. 오늘 새로 임의의 관리방식을 만들지 않습니다.

---

# 35. FastAPI가 Edge Runtime을 직접 실행하지 않게 만들기

잘못된 구조:

```text
FastAPI Request
→ Camera 열기
→ AI 실행
→ GPIO
```

이렇게 만들면 API Request가 없을 때 Edge AI가 멈추게 됩니다.

오늘의 구조:

```text
Edge Runtime
→ 항상 독립 실행
→ Camera / AI / GPIO 수행
→ Runtime State 기록

FastAPI
→ Runtime State만 읽음
→ 외부 PC에 전달
```

FastAPI는 **모니터링 통신 계층**입니다.

---

# 36. Runtime State가 오래되었는지 판정하는 함수 만들기

## 이 코드는 왜 필요할까요?

Runtime State 파일이 존재한다는 사실만으로 Edge Runtime이 지금도 살아 있다고 판단할 수는 없습니다. Process가 멈추면 마지막 JSON 파일은 디스크에 그대로 남을 수 있기 때문입니다.

```text
State 파일 존재
≠
현재 Edge Runtime 정상
```

따라서 마지막 `timestamp`가 현재 시각에서 얼마나 오래되었는지 계산하여 Heartbeat가 끊겼는지 확인합니다.

## 의사코드

```text
State Dictionary에서 timestamp를 찾는다
        ↓
timestamp가 없으면 None 반환
        ↓
ISO 문자열을 datetime으로 변환
        ↓
현재 UTC 시각 구하기
        ↓
현재 시각 - State 시각
        ↓
초 단위 age 반환
        ↓
형식이 잘못되었으면 None 반환
```

`src/runtime_state.py`에는 이미 `datetime`, `timezone`을 import하고 있습니다. 같은 import를 다시 쓰지 않고 파일 아래에 다음 함수만 추가합니다.

```python
def state_age_sec(
    data: dict,
) -> float | None:
    timestamp = data.get(
        "timestamp"
    )

    if not timestamp:
        return None

    try:
        state_time = (
            datetime.fromisoformat(
                timestamp
            )
        )

        now = datetime.now(
            timezone.utc
        )

        return (
            now - state_time
        ).total_seconds()

    except Exception:
        return None
```

이제 상태파일이 최근에 갱신되었는지 계산할 수 있습니다.

---

# 37. FastAPI Status API 만들기

## 이 파일은 왜 필요할까요?

Edge Runtime은 이미 독립적으로 Camera·AI·GPIO를 실행하고 `runtime/edge_state.json`을 갱신합니다. FastAPI는 그 AI를 다시 실행하지 않습니다. **이미 만들어진 최신 상태를 읽어 HTTP JSON으로 제공하는 모니터링 계층**만 담당합니다.

세 Endpoint의 역할을 먼저 구분합니다.

```text
/health
→ API가 응답하는가?
→ Edge Runtime State가 최근 것인가?
→ Edge status가 RUNNING인가?

/status
→ 현재 AI Prediction과 운영 State는 무엇인가?

/metrics
→ FPS / Latency / CPU / Memory / Temperature는 어떤가?
```

## 실행 전에 파일 연결 확인하기

```text
scripts/edge_runtime.py
        ↓ write
runtime/edge_state.json
        ↑ read
src/runtime_state.py
        ↑
scripts/status_api.py
        ↓
FastAPI / Uvicorn
        ↓
/health /status /metrics
        ↓
Raspberry Pi 내부 curl
        ↓
PC curl / Browser
```

## 의사코드

```text
Config를 읽는다
        ↓
RuntimeStateStore 준비
        ↓
stale 기준시간을 읽는다
        ↓
FastAPI App 생성
        ↓
공통 get_state()
    State 파일 읽기
    마지막 timestamp의 나이 계산
    stale 여부 계산
        ↓
/health
    API / Edge / stale 상태 반환
        ↓
/status
    AI Prediction / Raw State / Final State 반환
        ↓
/metrics
    FPS / Latency / Resource 값 반환
```

파일:

```text
scripts/status_api.py
```

코드:

```python
from fastapi import FastAPI

from src.config_loader import load_config
from src.runtime_state import (
    RuntimeStateStore,
    state_age_sec,
)


config = load_config(
    "configs/settings.yaml"
)

store = RuntimeStateStore(
    config["runtime_state_path"]
)

STALE_SEC = float(
    config["runtime_stale_sec"]
)

app = FastAPI(
    title="Subject 13 Edge Status API",
    version="1.0.0",
)


def get_state():
    state = store.read()

    age = state_age_sec(
        state
    )

    stale = (
        age is None
        or age > STALE_SEC
    )

    return state, age, stale


@app.get("/health")
def health():
    state, age, stale = (
        get_state()
    )

    edge_status = (
        state.get("status")
        if state
        else "UNKNOWN"
    )

    edge_ok = (
        bool(state)
        and not stale
        and edge_status
        == "RUNNING"
    )

    return {
        "api": "OK",
        "edge": (
            "OK"
            if edge_ok
            else "DEGRADED"
        ),
        "edge_status":
            edge_status,
        "state_age_sec":
            None
            if age is None
            else round(age, 3),
        "stale": stale,
    }


@app.get("/status")
def status():
    state, age, stale = (
        get_state()
    )

    return {
        "status":
            state.get(
                "status",
                "UNKNOWN",
            ),
        "state":
            state.get(
                "state",
                "UNKNOWN",
            ),
        "raw_state":
            state.get(
                "raw_state",
                "UNKNOWN",
            ),
        "pred_class":
            state.get(
                "pred_class",
                "UNKNOWN",
            ),
        "confidence":
            state.get(
                "confidence",
                0.0,
            ),
        "confidence_threshold":
            state.get(
                "confidence_threshold",
                0.0,
            ),
        "timestamp":
            state.get(
                "timestamp",
            ),
        "state_age_sec":
            None
            if age is None
            else round(age, 3),
        "stale":
            stale,
        "error_type":
            state.get(
                "error_type",
            ),
        "error_message":
            state.get(
                "error_message",
            ),
    }


@app.get("/metrics")
def metrics():
    state, age, stale = (
        get_state()
    )

    return {
        "loop_fps":
            state.get(
                "loop_fps",
                0.0,
            ),
        "inference_per_sec":
            state.get(
                "inference_per_sec",
                0.0,
            ),
        "mean_inference_ms":
            state.get(
                "mean_inference_ms",
                0.0,
            ),
        "p95_inference_ms":
            state.get(
                "p95_inference_ms",
                0.0,
            ),
        "cpu_percent":
            state.get(
                "cpu_percent",
                0.0,
            ),
        "memory_percent":
            state.get(
                "memory_percent",
                0.0,
            ),
        "temperature_c":
            state.get(
                "temperature_c",
                -1.0,
            ),
        "frame_count":
            state.get(
                "frame_count",
                0,
            ),
        "inference_count":
            state.get(
                "inference_count",
                0,
            ),
        "state_age_sec":
            None
            if age is None
            else round(age, 3),
        "stale":
            stale,
    }
```

---

# 38. FastAPI를 systemd 전에 직접 실행하기

Edge Service가 실행 중인지 확인합니다.

```bash
systemctl status \
subject13-edge.service \
--no-pager
```

프로젝트 가상환경:

```bash
source .venv/bin/activate
```

FastAPI 실행:

```bash
python -m uvicorn \
scripts.status_api:app \
--host 0.0.0.0 \
--port <API_PORT>
```

정상이라면 다음과 비슷한 메시지가 나타납니다.

```text
Uvicorn running on http://0.0.0.0:<API_PORT>
```

---

# 39. Raspberry Pi 안에서 API 먼저 확인하기

다른 SSH Terminal에서:

```bash
curl \
http://127.0.0.1:<API_PORT>/health
```

```bash
curl \
http://127.0.0.1:<API_PORT>/status
```

```bash
curl \
http://127.0.0.1:<API_PORT>/metrics
```

JSON이 반환되는지 확인합니다.

---

# 40. `/health`에서 확인할 내용

예:

```json
{
  "api": "OK",
  "edge": "OK",
  "edge_status": "RUNNING",
  "state_age_sec": 0.421,
  "stale": false
}
```

의미:

```text
api: OK
→ FastAPI Process가 응답함

edge: OK
→ 최근 Edge Runtime State가 정상

state_age_sec
→ 마지막 Heartbeat가 몇 초 전인지

stale: false
→ 오래된 상태가 아님
```

---

# 41. `/status`에서 확인할 내용

예:

```json
{
  "status": "RUNNING",
  "state": "WARNING",
  "raw_state": "WARNING",
  "pred_class": "warning_red",
  "confidence": 0.91,
  "confidence_threshold": 0.7
}
```

구조:

```text
AI Prediction
→ pred_class / confidence

운영 결과
→ state
```

둘을 한 API에서 함께 볼 수 있습니다.

---

# 42. `/metrics`에서 확인할 내용

예:

```json
{
  "loop_fps": 22.4,
  "inference_per_sec": 11.2,
  "mean_inference_ms": 14.8,
  "p95_inference_ms": 18.7,
  "cpu_percent": 47.1,
  "memory_percent": 21.8,
  "temperature_c": 54.2
}
```

8~9일차에서 Terminal과 CSV로 확인했던 값 일부가 이제 HTTP를 통해 외부로 제공됩니다.

---

# 43. PC에서 Raspberry Pi API 확인하기

Raspberry Pi IP를 확인합니다.

```bash
hostname -I
```

예:

```text
<RPI_IP>
```

PC Browser에서 다음 주소를 엽니다.

```text
http://<RPI_IP>:<API_PORT>/health
```

```text
http://<RPI_IP>:<API_PORT>/status
```

```text
http://<RPI_IP>:<API_PORT>/metrics
```

PC Terminal에서도 확인할 수 있습니다.

```bash
curl \
http://<RPI_IP>:<API_PORT>/health
```

---

# 44. FastAPI 자동 문서도 확인하기

PC Browser:

```text
http://<RPI_IP>:<API_PORT>/docs
```

다음 Endpoint가 보이는지 확인합니다.

```text
GET /health
GET /status
GET /metrics
```

오늘은 API 문서 기능을 깊게 다루지 않습니다.

Endpoint를 직접 실행하여 JSON이 반환되는지만 확인합니다.

---

# 45. Camera 앞에서 상태를 바꾸고 PC에서 확인하기

Camera 앞에 경고 대상이 없을 때:

```text
/status
→ state: NORMAL
```

빨간색 경고 대상을 보여줍니다.

잠시 후 PC Browser를 새로고침합니다.

```text
/status
→ state: WARNING
→ pred_class: warning_red
→ confidence: ...
```

즉:

```text
Camera 입력
→ Raspberry Pi 내부 AI
→ Runtime State
→ FastAPI
→ PC
```

전체 연결이 완성되었습니다.

---

# 46. FastAPI를 종료해도 Edge AI가 계속 동작하는지 확인하기

직접 실행한 FastAPI Terminal에서:

```text
Ctrl+C
```

PC에서 API를 호출하면 더 이상 응답하지 않습니다.

하지만 Raspberry Pi의 Edge Service는 확인합니다.

```bash
systemctl status \
subject13-edge.service \
--no-pager
```

Camera 앞에 경고 물체를 보여 GPIO도 확인합니다.

정상 구조:

```text
FastAPI DOWN
        ↓
PC 조회 실패

하지만

Edge Runtime
→ Camera
→ AI
→ GPIO
계속 동작
```

이것이 두 프로그램을 분리한 이유입니다.


## 코드 리뷰 — `/health` JSON은 어디에서 만들어졌을까요?

예를 들어 PC에서 다음 결과를 받았다고 가정합니다.

```json
{
  "api": "OK",
  "edge": "OK",
  "edge_status": "RUNNING",
  "state_age_sec": 0.421,
  "stale": false
}
```

각 값의 출처를 역추적합니다.

```text
"api": "OK"
→ scripts/status_api.py의 health()

"edge_status": "RUNNING"
→ runtime/edge_state.json의 status
→ scripts/edge_runtime.py가 기록

"state_age_sec"
→ src/runtime_state.py의 state_age_sec()

"stale"
→ status_api.py의 get_state()
→ age와 runtime_stale_sec 비교
```

여기서 중요한 점은 `/health`가 Camera를 새로 열거나 ONNX를 다시 추론하지 않는다는 것입니다.

```text
Camera / AI / GPIO
→ Edge Runtime

HTTP 요청 처리
→ FastAPI

둘 사이의 공유 정보
→ runtime/edge_state.json
```

## 초보자가 자주 겪는 API 오류

### 오류 A — Raspberry Pi 안에서도 `Connection refused`

```text
오류
curl http://127.0.0.1:<API_PORT>/health
→ Connection refused

원인 확인
→ API Process가 실행 중인가?
→ 실제 Port가 맞는가?

확인
→ systemctl status subject13-api.service
→ sudo ss -ltnp
→ grep api_port configs/settings.yaml

수정
→ API Service 시작 / Port 일치

재실행
→ curl ...
```

### 오류 B — Raspberry Pi 안에서는 되는데 PC에서는 안 됨

```text
가능한 원인
→ 잘못된 RPI IP
→ PC와 Raspberry Pi가 다른 Network
→ api_host가 127.0.0.1
→ 방화벽 / 교육장 Network 정책
```

확인 순서:

```text
1. Raspberry Pi에서 127.0.0.1:<API_PORT> 성공?
2. hostname -I로 현재 IPv4 확인
3. settings.yaml의 api_host 확인
4. ss에서 0.0.0.0:<API_PORT> 또는 해당 Interface Listen 확인
5. PC에서 Raspberry Pi Network 도달 여부 확인
6. 교육장 방화벽 정책 확인
```

### 오류 C — API는 응답하지만 `"edge": "DEGRADED"`

이 경우 FastAPI 자체는 살아 있습니다.

```text
API Process
→ 정상

하지만

Runtime State
→ 없거나
→ 오래되었거나
→ status가 RUNNING이 아님
```

다음 순서로 확인합니다.

```bash
systemctl status \
  subject13-edge.service \
  --no-pager
```

```bash
cat runtime/edge_state.json
```

```bash
journalctl \
  -u subject13-edge.service \
  -n 50 \
  --no-pager
```

### 오류 D — `/heath`처럼 주소를 잘못 입력하여 404

FastAPI가 실행 중이어도 Endpoint 이름이 틀리면 `404 Not Found`가 나올 수 있습니다.

```text
/health
O

/heath
X
```

오류 메시지를 보고 “Server가 죽었다”고 바로 결론 내리지 않습니다. **HTTP 응답은 왔지만 요청한 경로가 없는 경우**일 수 있습니다.



---

# 47. FastAPI도 systemd Service로 등록하기

FastAPI 파일을 만든 뒤 Service Unit을 다시 생성합니다.

```bash
source .venv/bin/activate
```

```bash
python -m scripts.render_systemd_units
```

API Unit 확인:

```bash
cat systemd/generated/subject13-api.service
```

설치:

```bash
sudo cp \
systemd/generated/subject13-api.service \
/etc/systemd/system/
```

systemd 갱신:

```bash
sudo systemctl daemon-reload
```

---

# 48. FastAPI Service 시작하기

```bash
sudo systemctl start \
subject13-api.service
```

상태:

```bash
systemctl status \
subject13-api.service \
--no-pager
```

정상:

```text
Active: active (running)
```

---

# 49. FastAPI Service Journal 확인하기

```bash
journalctl \
-u subject13-api.service \
-n 50 \
--no-pager
```

Uvicorn 시작 메시지가 보이는지 확인합니다.

PC에서도:

```bash
curl \
http://<RPI_IP>:<API_PORT>/health
```

응답을 확인합니다.

---

# 50. FastAPI도 부팅 자동실행 활성화하기

```bash
sudo systemctl enable \
subject13-api.service
```

확인:

```bash
systemctl is-enabled \
subject13-api.service
```

이제 두 Service가 모두 Enabled 상태여야 합니다.

```bash
systemctl is-enabled \
subject13-edge.service
```

```bash
systemctl is-enabled \
subject13-api.service
```

---

# 51. 두 Service를 한 번에 상태 확인하기

```bash
systemctl status \
subject13-edge.service \
subject13-api.service \
--no-pager
```

역할:

```text
subject13-edge
→ Camera / AI / GPIO

subject13-api
→ HTTP / JSON
```

---

# 52. Raspberry Pi 재부팅 후 두 Service 자동실행 확인하기

재부팅합니다.

```bash
sudo reboot
```

잠시 기다린 뒤 PC에서 다시 SSH 접속합니다.

```bash
ssh <RPI_USER>@<RPI_IP>
```

확인:

```bash
systemctl status \
subject13-edge.service \
subject13-api.service \
--no-pager
```

PC Browser 또는 curl:

```bash
curl \
http://<RPI_IP>:<API_PORT>/health
```

사람이 Python을 직접 실행하지 않아도 두 프로그램이 자동으로 동작하면 됩니다.

---

# PART E. 장애를 통제된 조건에서 발생시키고 복구하기

# 53. 장애 실험 1 — Edge Runtime 강제 종료

현재:

```text
Edge Service ON
API Service ON
```

Edge Runtime만 강제 종료합니다.

```bash
sudo systemctl kill \
-s SIGKILL \
subject13-edge.service
```

바로 PC에서:

```bash
curl \
http://<RPI_IP>:<API_PORT>/health
```

강제 종료 직후에는 `state_age_sec`가 증가합니다.

중요한 점은 **반드시 `edge: DEGRADED`가 보이는 것은 아니라는 것**입니다. 예를 들어:

```text
RestartSec = 3초
runtime_stale_sec = 5초
```

라면 systemd가 3초 안팎에 Edge Runtime을 다시 시작하여 새 Heartbeat를 쓰기 때문에, stale 기준 5초를 넘기기 전에 복구될 수 있습니다. 이 경우 `/health`는 계속 `edge: OK`로 보이거나 아주 짧은 변화만 보일 수 있습니다.

```text
강제 종료
→ State Age 증가
→ RestartSec 뒤 Process 재시작
→ 새 Heartbeat 기록
→ State Age 다시 작아짐
```

반대로 재시작이 늦거나 `runtime_stale_sec`를 넘기면 `edge: DEGRADED`가 보일 수 있습니다.

따라서 이 실습의 핵심은 특정 문자열을 억지로 보는 것이 아니라 **Journal에서 실제 Crash와 Restart가 있었는지, Runtime State의 Heartbeat가 다시 갱신되는지 확인하는 것**입니다.

---

# 54. 장애 실험 1 기록 확인하기

Edge Journal:

```bash
journalctl \
-u subject13-edge.service \
-n 80 \
--no-pager
```

API는 계속 실행 중인지 확인합니다.

```bash
systemctl status \
subject13-api.service \
--no-pager
```

구조:

```text
Edge Crash
→ API는 살아 있음
→ State Age 증가
→ stale 기준을 넘기면 DEGRADED 가능
→ systemd Edge Restart
→ 새 Heartbeat
→ API 정상 상태 유지 또는 복귀
```

---

# 55. 장애 실험 2 — FastAPI 강제 종료

이번에는 Edge Runtime은 그대로 두고 API만 강제 종료합니다.

```bash
sudo systemctl kill \
-s SIGKILL \
subject13-api.service
```

잠시 후:

```bash
systemctl status \
subject13-api.service \
--no-pager
```

`Restart=on-failure`에 의해 다시 실행되는지 확인합니다.

그동안 Edge Runtime도 확인합니다.

```bash
systemctl status \
subject13-edge.service \
--no-pager
```

GPIO 판단은 계속 동작해야 합니다.

---

# 56. 장애 실험 3 — Model 파일 문제

이 실험은 32절에서 이미 한 번 수행했다면 다시 하지 않아도 됩니다.

확인할 항목:

```text
Edge Runtime
→ ERROR

API
→ Edge 상태를 DEGRADED 또는 ERROR로 표시

journalctl
→ Model Path 오류

Model 복구
→ Service Restart
→ 정상 복귀
```

---

# 57. 장애 실험 4 — Network 연결 해제는 선택 실습

SSH만 사용 중인 상태에서 Wi-Fi를 끄면 접속 자체가 끊길 수 있습니다.

따라서 Network 장애 실험은 다음 조건에서만 진행합니다.

```text
Raspberry Pi에 직접 모니터·키보드가 있거나

유선 Network와 무선 Network를
별도로 확인할 수 있는 환경
```

핵심 예상 결과는 다음입니다.

```text
Network DOWN
→ PC FastAPI 접근 실패

하지만
Edge Runtime
→ Camera
→ AI
→ GPIO
계속 동작
```

Network 복구 후 API 접근이 다시 가능해야 합니다.

---

# 58. 장애 실험 기록 문서 만들기

파일:

```text
reports/day10_fault_recovery.md
```

내용:

```md
# Day 10 Fault Recovery

## Service

Edge Service:

API Service:

Edge Enabled:

API Enabled:

## Boot Test

재부팅 전:

재부팅 후 Edge:

재부팅 후 API:

사람이 직접 Python 실행:

- 필요 / 불필요

## Failure 1

장애:

발생 시각:

Edge Service 상태:

API 상태:

GPIO 상태:

Runtime State:

journalctl:

자동재시작:

복구 방법:

## Failure 2

장애:

발생 시각:

Edge Service 상태:

API 상태:

GPIO 상태:

Runtime State:

journalctl:

자동재시작:

복구 방법:

## Operational Event Log

Event Log Path:

10일차 실행 전 마지막 Timestamp:

10일차 실행 후 새 Timestamp:

NORMAL → WARNING 또는 WARNING → NORMAL 전환 기록:

- 확인 / 미확인

현재 Runtime에서 새 Row 생성:

- PASS / FAIL

## FastAPI

### /health

확인한 값:

### /status

확인한 값:

### /metrics

확인한 값:

## Edge와 API의 역할 차이

Edge Runtime:

FastAPI:

FastAPI가 중지되었을 때 Edge AI:

## 최종 운영 상태

```

최소 두 가지 장애를 실제로 발생시키고 기록합니다.

---

# PART F. 운영 진단과 작은 확장 실습

# 59. Guided Modify Lab — `/metrics`에 Throttling 상태도 연결하기

9일차에는 Raspberry Pi에서 지원되는 경우 `get_throttled_raw()`로 전원·Throttling 관련 상태를 확인했습니다. 오늘은 이미 배운 이 값을 **Runtime State → FastAPI** 경로에 한 번 연결해 봅니다.

이 실습은 새 기술을 배우는 것이 아니라, 기존 데이터 하나가 여러 파일을 지나 API까지 전달되는 흐름을 확인하는 작은 수정 실습입니다.

## 먼저 어느 파일이 바뀌어야 할지 생각합니다

```text
src/resource_monitor.py
get_throttled_raw()
        ↓
scripts/edge_runtime.py
Runtime State에 저장
        ↓
runtime/edge_state.json
        ↓
scripts/status_api.py
/metrics에서 반환
        ↓
PC
```

바로 수정하지 말고 다음 질문에 답합니다.

```text
1. get_throttled_raw()는 이미 어느 파일에 있는가?
2. edge_runtime.py는 현재 이 함수를 import하고 있는가?
3. Runtime State에 어떤 Key 이름으로 넣을 것인가?
4. /metrics는 그 Key를 어떻게 읽어야 하는가?
5. vcgencmd를 지원하지 않는 환경에서는 어떤 값을 보여주는 것이 안전한가?
```

### 1단계 — `edge_runtime.py` import 추가

기존:

```python
from src.resource_monitor import (
    get_cpu_percent,
    get_memory_percent,
    get_temperature_c,
)
```

수정:

```python
from src.resource_monitor import (
    get_cpu_percent,
    get_memory_percent,
    get_temperature_c,
    get_throttled_raw,
)
```

### 2단계 — Runtime State에 값 추가

`store.write({...})`에 다음 Key를 추가합니다.

```python
{
    "throttled": get_throttled_raw(),
}
```

### 3단계 — `/metrics` 응답에 값 추가

`scripts/status_api.py`의 `/metrics` 반환 Dictionary에 다음을 추가합니다.

```python
{
    "throttled": state.get(
        "throttled",
        "UNAVAILABLE",
    ),
}
```

### 4단계 — Source를 바꿨으므로 Service 재시작

Unit 파일 자체를 바꾼 것은 아니므로 `daemon-reload`는 필요하지 않습니다.

```bash
sudo systemctl restart \
  subject13-edge.service
```

```bash
sudo systemctl restart \
  subject13-api.service
```

### 5단계 — 결과 확인

```bash
curl \
http://127.0.0.1:<API_PORT>/metrics
```

예상 형태:

```json
{
  "loop_fps": 20.1,
  "temperature_c": 52.4,
  "throttled": "throttled=0x0"
}
```

환경에서 `vcgencmd`를 사용할 수 없다면 다음처럼 나올 수 있습니다.

```json
{
  "throttled": "UNAVAILABLE"
}
```

`UNAVAILABLE`은 무조건 장애라는 뜻이 아닙니다. **현재 장비·OS에서 해당 확인 방법을 사용할 수 없다는 뜻**으로 해석합니다.

### 코드 리뷰

```text
화면의 "throttled"
→ status_api.py /metrics
→ runtime/edge_state.json의 throttled
→ edge_runtime.py의 store.write()
→ resource_monitor.py의 get_throttled_raw()
```

이 흐름을 직접 찾았다면 “한 값이 어느 파일을 거쳐 외부로 전달되는가?”를 이해한 것입니다.

> 이 Guided Modify Lab의 결과는 유용하므로 그대로 유지해도 됩니다. 단, `get_throttled_raw()`가 없는 이전 9일차 Source를 사용 중인 학생은 억지로 추가하지 않고 핵심 실습만 진행합니다.

---

# 60. FastAPI는 Dashboard가 아닙니다

현재 PC에서 보는 것은 JSON입니다.

```text
/health
/status
/metrics
```

FastAPI의 역할:

```text
Raspberry Pi 상태
→ HTTP / JSON으로 제공
```

Dashboard의 역할:

```text
HTTP / JSON
→ 사람이 보기 좋은 화면
```

즉:

```text
FastAPI
≠ Dashboard
```

입니다.

교과 13에서는 Browser JSON이나 간단한 Client 확인만으로 충분합니다.

---

# 61. PC에서 간단한 상태 Client 만들기

Browser와 `curl`만으로도 충분히 확인할 수 있지만, 마지막으로 **PC 프로그램에서 세 Endpoint를 순서대로 읽는 작은 Client**를 만들어 봅니다. 이 Client는 Raspberry Pi 내부 AI를 실행하지 않고 API 결과만 읽습니다.

## 이 파일은 왜 필요할까요?

11일차에서는 API Smoke Test와 System Test로 이어집니다. 오늘의 Client는 “외부 프로그램이 Edge 장치 상태를 읽을 수 있다”는 가장 작은 예제입니다.

## 의사코드

```text
--host와 --port를 받는다
        ↓
http://host:port Base URL 생성
        ↓
/health 요청
→ JSON Decode
→ 출력
        ↓
/status 요청
→ JSON Decode
→ 출력
        ↓
/metrics 요청
→ JSON Decode
→ 출력
```

파일:

```text
scripts/check_remote_status.py
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
        help="Raspberry Pi IPv4 또는 Hostname",
    )

    parser.add_argument(
        "--port",
        type=int,
        default=8000,
        help="configs/settings.yaml의 api_port와 같은 값",
    )

    return parser.parse_args()


def get_json(url: str):
    with urlopen(
        url,
        timeout=3,
    ) as response:
        return json.loads(
            response.read().decode(
                "utf-8"
            )
        )


def main():
    args = parse_args()

    base = (
        f"http://{args.host}:{args.port}"
    )

    for endpoint in [
        "/health",
        "/status",
        "/metrics",
    ]:
        print()
        print(endpoint)

        data = get_json(
            base + endpoint
        )

        print(
            json.dumps(
                data,
                ensure_ascii=False,
                indent=2,
            )
        )


if __name__ == "__main__":
    main()
```

PC에서 실행:

```bash
python -m scripts.check_remote_status \
  --host <RPI_IP> \
  --port <API_PORT>
```

이 프로그램은 Raspberry Pi 내부 AI를 실행하지 않습니다.

API를 통해 결과만 읽습니다.

---

# 62. Headless 운영 확인 순서 정리하기

모니터가 없는 Raspberry Pi가 정상인지 확인할 때 다음 순서를 사용합니다.

```text
1. Network
SSH 또는 API 접근 가능한가?

        ↓

2. systemd
Edge / API Service가 running인가?

        ↓

3. Runtime Heartbeat
edge_state.json이 최근 값인가?

        ↓

4. /health
Edge가 OK인가?

        ↓

5. /status
현재 Prediction / State는?

        ↓

6. /metrics
Latency / FPS / CPU / Temp는?

        ↓

7. journalctl
오류가 남아 있는가?

        ↓

8. GPIO / Event Log
실제 출력과 기록은 정상인가?
```

---

# 63. 장애 원인을 운영 계층별로 구분하기

| 증상 | 먼저 확인할 영역 |
|---|---|
| SSH 접속도 안 됨 | Network / Power |
| Edge Service inactive | systemd / Program |
| Edge Service 반복 재시작 | Camera / Model / Config / GPIO |
| API만 안 됨 | FastAPI / Port / API Service |
| API는 되는데 `edge: DEGRADED` | Edge Runtime / Heartbeat |
| `/status` 값이 오래됨 | Runtime State / Edge Process |
| Prediction 정상, GPIO만 이상 | GPIO |
| `/metrics` 값 없음 | Runtime State 작성 부분 |
| 재부팅 후 안 켜짐 | `systemctl enable` |
| 오류 원인을 모름 | `journalctl -u ...` |

---

# 64. systemd Service를 수정한 뒤 반드시 해야 하는 것

Service 파일을 수정했다면 `/etc/systemd/system/`에 다시 복사합니다.

예:

```bash
sudo cp \
systemd/generated/subject13-edge.service \
/etc/systemd/system/
```

그다음:

```bash
sudo systemctl daemon-reload
```

그리고:

```bash
sudo systemctl restart \
subject13-edge.service
```

Unit 파일을 수정했는데 `daemon-reload`를 하지 않으면 systemd가 이전 설정을 사용할 수 있습니다.

---

# 65. Service의 현재 설정 확인하기

실제로 systemd가 읽고 있는 Unit을 확인합니다.

```bash
systemctl cat \
subject13-edge.service
```

API:

```bash
systemctl cat \
subject13-api.service
```

프로젝트 안의 파일만 보지 말고 **systemd가 실제로 사용 중인 설정**도 확인합니다.

---

# 66. 자동실행을 잠시 해제하는 방법도 확인하기

자동실행 해제:

```bash
sudo systemctl disable \
subject13-edge.service
```

다시 활성화:

```bash
sudo systemctl enable \
subject13-edge.service
```

API도 같은 방식입니다.

```bash
sudo systemctl disable \
subject13-api.service
```

```bash
sudo systemctl enable \
subject13-api.service
```

수업 종료 시에는 두 Service를 다시 `enabled` 상태로 둡니다.

---

# 67. FastAPI Port가 실제로 열려 있는지 확인하기

Raspberry Pi에서:

```bash
sudo ss -ltnp | grep ":<API_PORT>"
```

정상이라면 `configs/settings.yaml`의 `api_port`에 지정한 Port를 Uvicorn이 Listen하고 있는 것을 확인할 수 있습니다.

API Process:

```bash
ps aux | grep uvicorn
```

systemd가 실행한 Process인지 확인합니다.

---

# 68. 외부 공개용 API가 아니라 실습용 내부 API입니다

현재 API는:

```text
0.0.0.0:<API_PORT>
```

으로 같은 Network의 PC에서 접근할 수 있게 했습니다.

오늘 실습에는 인증 기능을 추가하지 않습니다.

따라서 다음 원칙을 지킵니다.

```text
인터넷에 Port Forwarding하지 않기

공용 서버에 그대로 노출하지 않기

기업 데이터나 개인정보를 API Response에 넣지 않기

실습망 / 허용된 내부 Network에서 사용
```

---


# PART G. Mini Challenge — Headless 운영 흐름을 스스로 진단하고 수정하기

지금까지는 안내된 순서대로 Runtime State, systemd, FastAPI를 연결했습니다. 이제는 **결과를 보고 원인을 찾고, 설정과 코드를 직접 수정하며, 장애를 복구하는 연습**을 합니다.

Mini Challenge는 다음 순서로 진행합니다.

```text
1. 실행 결과 → 코드 위치 찾기
2. 실행 전 결과 예측하기
3. 설정값만 바꾸어 비교하기
4. 기존 기능 일부 수정하기
5. 일부러 오류 만들기 → 원인 찾기 → 복구
6. 결과 파일 / Journal로 정상 동작 증명하기
7. 전체 Pipeline을 자신의 말로 설명하기
```

정답을 먼저 열지 않습니다. 직접 생각하고 실행한 뒤 **예시 정답 확인해보기**를 열어 비교합니다.

---

## Mini Challenge 시작 전 — 정상 Baseline을 백업합니다

Challenge에서 Config와 API Source를 일부 수정합니다. 11일차에 영향을 남기지 않도록 현재 정상 상태를 임시 백업합니다.

먼저 두 Service가 정상인지 확인합니다.

```bash
systemctl status \
  subject13-edge.service \
  subject13-api.service \
  --no-pager
```

API:

```bash
curl \
http://127.0.0.1:<API_PORT>/health
```

정상이라면 다음 파일을 임시 백업합니다.

```bash
cp \
configs/settings.yaml \
/tmp/day10_settings_before_challenge.yaml
```

```bash
cp \
scripts/status_api.py \
/tmp/day10_status_api_before_challenge.py
```

현재 Unit도 다시 생성해 둡니다.

```bash
source .venv/bin/activate
python -m scripts.render_systemd_units
```

> `/tmp`는 임시 영역입니다. Challenge 도중에는 Raspberry Pi를 재부팅하지 않습니다. 마지막 복원 단계에서 원래 Source와 Config로 되돌립니다.

---

## Challenge 1 — `/health` 결과를 코드까지 역추적하기

PC 또는 Raspberry Pi에서 다음과 비슷한 결과가 보였다고 가정합니다.

```json
{
  "api": "OK",
  "edge": "OK",
  "edge_status": "RUNNING",
  "state_age_sec": 0.382,
  "stale": false
}
```

다음 표를 코드와 파일을 직접 열어 채웁니다.

| 화면에 보인 값 | 최초 데이터가 만들어지는 위치 | API로 전달하는 위치 |
|---|---|---|
| `api: OK` |  |  |
| `edge_status: RUNNING` |  |  |
| `state_age_sec` |  |  |
| `stale` |  |  |
| `timestamp`의 원본 |  |  |

추가 질문:

```text
1. /health 요청이 들어올 때 ONNX 추론을 새로 수행하는가?
2. RUNNING은 FastAPI가 임의로 만든 문자열인가?
3. state_age_sec는 어떤 두 시각의 차이인가?
4. Runtime State가 비어 있으면 edge는 OK가 될 수 있는가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

| 값 | 최초 데이터 / 계산 위치 | API 위치 |
|---|---|---|
| `api: OK` | FastAPI가 자기 응답 상태로 직접 만듦 | `scripts/status_api.py`의 `health()` |
| `edge_status: RUNNING` | `scripts/edge_runtime.py`가 Runtime State에 기록 | `health()`가 State의 `status`를 읽음 |
| `state_age_sec` | `src/runtime_state.py`의 `state_age_sec()` | `health()`가 반환 |
| `stale` | `status_api.py`의 `get_state()` | `age > STALE_SEC` 비교 |
| `timestamp` | `RuntimeStateStore.write()`의 `utc_now_iso()` | `state_age_sec()`가 읽음 |

```text
1. 아니다. /health는 Runtime State만 읽는다.
2. 아니다. Edge Runtime이 기록한 status를 읽는다.
3. 현재 UTC 시각 - Runtime State timestamp이다.
4. 아니다. State가 없으면 edge는 DEGRADED로 해석된다.
```

핵심 연결:

```text
edge_runtime.py
→ runtime_state.py
→ edge_state.json
→ status_api.py
→ /health
```

</details>

---

## Challenge 2 — 실행 전에 장애 결과를 예측하기

현재 상태:

```text
subject13-edge.service
→ active

subject13-api.service
→ active

/health
→ edge: OK
```

이제 **API Service만** 다음처럼 강제 종료한다고 가정합니다.

```bash
sudo systemctl kill \
  -s SIGKILL \
  subject13-api.service
```

실행하기 전에 다음을 먼저 예상합니다.

```text
1. Edge Service는 같이 종료될까?
2. Camera → AI → GPIO는 멈출까?
3. PC의 /health 호출은 잠시 어떻게 될까?
4. Restart=on-failure가 있으면 API Service는 어떻게 될까?
5. 복구 후 /health는 다시 응답할까?
```

예상을 적은 뒤 실제로 실행합니다.

```bash
sudo systemctl kill \
  -s SIGKILL \
  subject13-api.service
```

잠시 후:

```bash
systemctl status \
  subject13-api.service \
  --no-pager
```

Edge도 확인합니다.

```bash
systemctl status \
  subject13-edge.service \
  --no-pager
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
1. Edge Service는 별도 Unit이므로 함께 종료되지 않아야 한다.
2. Edge Runtime이 정상이라면 Camera → AI → GPIO는 계속 동작한다.
3. API Process가 내려간 순간에는 Connection refused / 연결 실패가 날 수 있다.
4. Restart=on-failure가 SIGKILL 실패를 감지하면 RestartSec 뒤 다시 시작한다.
5. API가 재시작되면 /health가 다시 응답한다.
```

이 Challenge의 핵심은 **FastAPI 장애와 Edge AI 장애를 같은 것으로 보지 않는 것**입니다.

```text
API DOWN
≠
Edge AI DOWN
```

</details>

---

## Challenge 3 — Python 코드는 그대로 두고 `runtime_stale_sec`만 비교하기

이번 Challenge에서는 Python Source를 수정하지 않습니다.

Heartbeat는 예를 들어 다음처럼 유지합니다.

```yaml
runtime_heartbeat_sec: 1.0
```

### 실험 A — 빠르게 stale 판정

`configs/settings.yaml`:

```yaml
runtime_stale_sec: 2.0
```

`status_api.py`는 시작할 때 Config를 읽으므로 API Service를 재시작합니다.

```bash
sudo systemctl restart \
  subject13-api.service
```

Edge Service가 정상인지 확인한 뒤:

```bash
sudo systemctl restart \
  subject13-edge.service
```

```bash
curl \
http://127.0.0.1:<API_PORT>/health
```

정상 State가 갱신된 것을 확인한 뒤 Edge를 **의도적으로 Stop**합니다.

```bash
sudo systemctl stop \
  subject13-edge.service
```

3초 정도 기다린 뒤:

```bash
curl \
http://127.0.0.1:<API_PORT>/health
```

결과를 기록합니다.

### 실험 B — 느리게 stale 판정

Edge를 다시 시작합니다.

```bash
sudo systemctl start \
  subject13-edge.service
```

Config:

```yaml
runtime_stale_sec: 10.0
```

API에 새 설정을 적용합니다.

```bash
sudo systemctl restart \
  subject13-api.service
```

Edge의 새 Heartbeat가 생긴 것을 확인한 뒤 다시 중지합니다.

```bash
sudo systemctl stop \
  subject13-edge.service
```

3초 후 `/health`를 확인하고, 다시 8초 이상 기다린 뒤 한 번 더 확인합니다.

다음 표를 작성합니다.

| stale 기준 | Edge Stop 후 3초 | 충분히 지난 뒤 | 해석 |
|---:|---|---|---|
| 2초 |  |  |  |
| 10초 |  |  |  |

질문:

```text
stale 시간을 크게 하면 무조건 좋은가?
작게 하면 무조건 좋은가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

대체로 다음 경향을 예상할 수 있습니다.

| stale 기준 | Edge Stop 후 3초 | 충분히 지난 뒤 | 해석 |
|---:|---|---|---|
| 2초 | `DEGRADED`, `stale: true` 가능 | 계속 DEGRADED | 장애 감지가 빠르지만 일시적인 Heartbeat 지연에도 민감 |
| 10초 | 아직 `OK`로 보일 수 있음 | 10초를 넘으면 DEGRADED | 오판에는 여유가 있지만 실제 장애 감지가 늦음 |

```text
작은 stale_sec
→ 빠른 장애 감지
→ 일시적인 지연을 장애로 오판할 위험 증가

큰 stale_sec
→ 순간 지연에 여유
→ 실제 Process 정지를 늦게 감지
```

정답은 하나가 아닙니다. `runtime_heartbeat_sec`, 장치 부하, 운영 요구조건을 함께 보고 정합니다.

실험 후 Edge를 다시 시작합니다.

```bash
sudo systemctl start \
  subject13-edge.service
```

</details>

---

## Challenge 4 — `/status`에 Frame 수와 Inference 수 추가하기

현재 `edge_runtime.py`는 Runtime State에 다음 값을 이미 기록합니다.

```text
frame_count
inference_count
```

하지만 `/status`에서는 이 값을 보여주지 않습니다.

요구사항:

> `/status` 응답에 `frame_count`, `inference_count` 두 값을 추가하고, 새로고침할 때 값이 증가하는지 확인합니다.

먼저 어느 파일만 수정하면 되는지 생각합니다.

```text
edge_runtime.py를 수정해야 할까?
runtime_state.py를 수정해야 할까?
status_api.py만 수정하면 될까?
```

직접 수정한 뒤 API Service만 재시작합니다.

```bash
sudo systemctl restart \
  subject13-api.service
```

확인:

```bash
curl \
http://127.0.0.1:<API_PORT>/status
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

`edge_runtime.py`는 이미 두 값을 Runtime State에 기록하고 있으므로 오늘 요구사항에서는 `scripts/status_api.py`만 수정하면 됩니다.

`/status`의 반환 Dictionary에 다음을 추가할 수 있습니다.

```python
{
    "frame_count": state.get(
        "frame_count",
        0,
    ),
    "inference_count": state.get(
        "inference_count",
        0,
    ),
}
```

예상 형태:

```json
{
  "status": "RUNNING",
  "state": "NORMAL",
  "frame_count": 1240,
  "inference_count": 620
}
```

잠시 뒤 다시 호출하면 값이 증가할 수 있습니다.

```text
Runtime State에는 이미 값이 있음
        ↓
FastAPI가 어떤 Key를 외부에 보여줄지만 수정
```

즉, 데이터가 이미 생성되고 있다면 모든 파일을 수정할 필요가 없습니다.

</details>

---

## Challenge 5 — 일부러 API Unit의 실행 Module을 틀리게 만들고 복구하기

이번에는 `systemd/generated/subject13-api.service`의 **생성된 로컬 Unit**만 잠시 수정합니다.

정상 Unit을 먼저 확인합니다.

```bash
grep '^ExecStart=' \
systemd/generated/subject13-api.service
```

정상 형태:

```text
... -m uvicorn scripts.status_api:app ...
```

임시로 다음처럼 잘못 바꿉니다.

```text
scripts.status_api_typo:app
```

그 뒤 실제 systemd 위치에 복사합니다.

```bash
sudo cp \
systemd/generated/subject13-api.service \
/etc/systemd/system/
```

Unit이 바뀌었으므로:

```bash
sudo systemctl daemon-reload
```

API를 재시작합니다.

```bash
sudo systemctl restart \
  subject13-api.service
```

바로 정답을 보지 말고 다음 순서로 원인을 찾습니다.

```text
오류 발생
        ↓
systemctl status
        ↓
journalctl
        ↓
ExecStart 확인
        ↓
Module 이름 확인
        ↓
원래 Unit 재생성
        ↓
daemon-reload
        ↓
restart
        ↓
/health 재확인
```

확인 명령:

```bash
systemctl status \
  subject13-api.service \
  --no-pager
```

```bash
journalctl \
  -u subject13-api.service \
  -n 40 \
  --no-pager
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

예상되는 핵심 원인은 다음입니다.

```text
systemd 자체가 FastAPI 코드를 이해하지 못한 것
X

ExecStart가 존재하지 않는 Python Module을 실행
O
```

복구는 직접 손으로 다시 맞추기보다 **생성기를 다시 실행하여 정상 Unit을 만드는 것**이 안전합니다.

```bash
source .venv/bin/activate
python -m scripts.render_systemd_units
```

```bash
sudo cp \
systemd/generated/subject13-api.service \
/etc/systemd/system/
```

```bash
sudo systemctl daemon-reload
sudo systemctl restart \
  subject13-api.service
```

확인:

```bash
curl \
http://127.0.0.1:<API_PORT>/health
```

정상 응답이 돌아오면 복구 완료입니다.

</details>

---

## Challenge 6 — 파일과 Journal로 “정상 동작했다”는 증거 남기기

화면에서 한 번 `OK`를 본 것만으로 끝내지 않습니다. 11일차 Test에서 다시 사용할 수 있도록 **증거 파일**을 남깁니다.

폴더:

```bash
mkdir -p \
reports/day10_evidence
```

현재 `/health` 응답 저장:

```bash
curl -s \
http://127.0.0.1:<API_PORT>/health \
> reports/day10_evidence/health.json
```

`/status`:

```bash
curl -s \
http://127.0.0.1:<API_PORT>/status \
> reports/day10_evidence/status.json
```

운영 Event Log의 최근 Row도 증거로 남깁니다. 먼저 `ai_event_log_path`의 실제 값을 확인한 뒤 `<AI_EVENT_LOG_PATH>`를 바꿉니다.

```bash
tail -n 20 <AI_EVENT_LOG_PATH> \
> reports/day10_evidence/ai_event_log_tail.csv
```

Edge Journal:

```bash
journalctl \
  -u subject13-edge.service \
  -n 50 \
  --no-pager \
> reports/day10_evidence/edge_journal.txt
```

API Journal:

```bash
journalctl \
  -u subject13-api.service \
  -n 50 \
  --no-pager \
> reports/day10_evidence/api_journal.txt
```

파일 목록:

```bash
find reports/day10_evidence \
  -maxdepth 1 \
  -type f \
  -print
```

내용도 직접 확인합니다.

```bash
cat \
reports/day10_evidence/health.json
```

질문:

```text
1. health.json만 있으면 Edge Runtime의 과거 오류 원인도 알 수 있을까?
2. journal 파일은 무엇을 보완해 주는가?
3. status.json은 health.json과 무엇이 다른가?
4. ai_event_log_tail.csv는 무엇을 증명하는가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
1. 아니다.
   health.json은 저장한 순간의 상태를 보여준다.

2. journal은 Process 시작·종료·Traceback·Restart 기록을 남겨
   과거 장애 흐름을 확인하는 데 도움이 된다.

3. health는 살아 있는지와 stale 여부 중심이고,
   status는 현재 Prediction과 운영 State를 확인하는 데 사용한다.

4. ai_event_log_tail.csv는 현재 운영 Runtime에서 최종 State 변화가 CSV Event로 실제 기록되었는지 확인하는 증거이다.
   단, 과거 Row만 복사한 것이 아닌지 Timestamp도 함께 확인해야 한다.
```

따라서 운영 증거는 한 종류만 보는 것보다 다음처럼 역할을 구분하는 것이 좋습니다.

```text
현재 상태
→ Runtime State / API

운영 상태 변화 이력
→ AI Event CSV

Process 이력과 오류
→ journalctl

실험·수업 결과
→ Report
```

</details>

---

## Challenge 7 — 오늘의 전체 Pipeline을 자신의 말로 설명하기

코드를 보지 않고 다음 빈칸을 채워 봅니다.

```text
Raspberry Pi Boot
        ↓
____________________
        ↓
subject13-edge.service
        ↓
____________________
        ↓
Camera
        ↓
ONNX AI
        ↓
____________________
        ↓
GPIO
        ↓
운영 State 변화 → ____________________
        ↓
runtime/____________________
        ↓
subject13-api.service
        ↓
____________________
        ↓
/health /status /metrics
        ↓
____________________
```

그리고 다음 장애 상황을 말로 설명합니다.

```text
A. FastAPI가 죽었다.
→ Edge AI는 어떻게 되는가?
→ 누가 API를 다시 시작하는가?

B. Edge Runtime이 죽었다.
→ Runtime State는 어떻게 보이는가?
→ /health는 어떻게 변할 수 있는가?
→ 누가 Edge Runtime을 다시 시작하는가?

C. Network가 끊겼다.
→ PC 조회는?
→ Raspberry Pi 내부 Camera / AI / GPIO는?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

전체 흐름의 한 예:

```text
Raspberry Pi Boot
        ↓
systemd
        ↓
subject13-edge.service
        ↓
scripts/edge_runtime.py
        ↓
Camera
        ↓
ONNX AI
        ↓
Stabilization
        ↓
GPIO
        ↓
운영 State 변화 → AI Event Log
        ↓
runtime/edge_state.json
        ↓
subject13-api.service
        ↓
FastAPI
        ↓
/health /status /metrics
        ↓
PC
```

장애 설명:

```text
A. API Process 장애
→ Edge Service는 독립적으로 계속 동작
→ systemd가 API Service를 Restart
→ 복구 중 PC API 요청은 잠시 실패할 수 있음

B. Edge Runtime 장애
→ 마지막 State가 더 이상 갱신되지 않거나 ERROR State가 기록될 수 있음
→ /health는 stale 또는 ERROR를 보고 DEGRADED
→ systemd가 Edge Service를 Restart

C. Network 장애
→ PC의 HTTP / SSH 접근은 실패할 수 있음
→ Edge Runtime은 Network와 분리되어 있다면
   Raspberry Pi 내부 Camera / AI / GPIO를 계속 수행
```

정답 문장을 그대로 외우는 것이 목적이 아닙니다. **어떤 Process가 무엇을 책임지는지 구분하여 설명할 수 있으면 됩니다.**

</details>

---

## Mini Challenge 종료 — 11일차를 위해 정상 상태로 복원하기

Challenge에서 변경한 `settings.yaml`과 `status_api.py`를 백업본으로 복원합니다.

```bash
cp \
/tmp/day10_settings_before_challenge.yaml \
configs/settings.yaml
```

```bash
cp \
/tmp/day10_status_api_before_challenge.py \
scripts/status_api.py
```

Challenge 시작 전에 만든 백업은 Part F까지 완료한 상태입니다. 따라서 Guided Modify Lab에서 `throttled`를 추가했다면 그 상태가 백업에 포함되어 있으며, Challenge에서 바꾼 내용만 되돌아갑니다.

현재 Config를 기준으로 Unit을 다시 생성합니다.

```bash
source .venv/bin/activate
python -m scripts.render_systemd_units
```

두 Unit을 실제 systemd 위치에 다시 복사합니다.

```bash
sudo cp \
systemd/generated/subject13-edge.service \
/etc/systemd/system/
```

```bash
sudo cp \
systemd/generated/subject13-api.service \
/etc/systemd/system/
```

반드시 Unit을 다시 읽힙니다.

```bash
sudo systemctl daemon-reload
```

두 Service를 활성화하고 재시작합니다.

```bash
sudo systemctl enable \
  subject13-edge.service
```

```bash
sudo systemctl enable \
  subject13-api.service
```

```bash
sudo systemctl restart \
  subject13-edge.service
```

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
curl \
http://127.0.0.1:<API_PORT>/health
```

예상 핵심:

```text
Edge Service
→ active (running)

API Service
→ active (running)

Edge Enabled
→ enabled

API Enabled
→ enabled

/health
→ api: OK
→ edge: OK
→ stale: false
```

마지막으로 Camera 앞에서 NORMAL / WARNING을 바꾸어 `/status`와 GPIO가 함께 변하는지 확인합니다.

운영 Event Log도 함께 확인합니다.

```bash
tail -n 10 <AI_EVENT_LOG_PATH>
```

Challenge 복원 뒤에도 현재 `edge_runtime.py`가 새 State 변화 Row를 기록해야 합니다.

```text
Challenge용 변경
→ 제거 또는 의도한 최종 기능만 유지
        ↓
10일차 정상 운영 Baseline
        ↓
11일차 Test 시작
```

---

# PART H. 결과 기록 · Git · README · 최종 복습

# 69. 10일차 실행 결과 Report 작성하기

`reports/day10_fault_recovery.md`에 다음 표도 추가합니다.

```md
## Service Test

| Test | Edge | API | 자동복구 | 확인 위치 |
|---|---|---|---|---|
| 정상 시작 |  |  | - | systemctl |
| Edge SIGKILL |  |  |  | journalctl |
| API SIGKILL |  |  |  | journalctl |
| Camera 오류 |  |  |  | journalctl / health |
| Model 오류 |  |  |  | journalctl / status |

## Operational Event Log Test

| 확인 항목 | 결과 | Evidence |
|---|---|---|
| 첫 운영 State 기록 | PASS / FAIL | Event CSV |
| 최종 State 변화 새 Row 기록 | PASS / FAIL | Event CSV Timestamp |
| Raw 흔들림만 있고 최종 State가 같을 때 중복 Event 없음 | PASS / FAIL | Event CSV |
| `ai_event_log_path`와 실제 파일 경로 일치 | PASS / FAIL | Config + 파일 |

## API Test

| Endpoint | 목적 | 결과 |
|---|---|---|
| /health | Process·Heartbeat 상태 |  |
| /status | AI·운영 상태 |  |
| /metrics | 성능·자원 상태 |  |
```

---

# 70. 10일차 Source Git Checkpoint 만들기

Git 상태를 확인합니다.

```bash
git status
```

Runtime 파일은 나타나지 않아야 합니다.

Stage:

```bash
git add \
configs \
src/runtime_state.py \
scripts/runtime_state_test.py \
scripts/edge_runtime.py \
scripts/render_systemd_units.py \
scripts/status_api.py \
requirements.txt \
.gitignore
```

PC Client를 만들었다면:

```bash
git add \
scripts/check_remote_status.py
```

Commit:

```bash
git commit -m \
"feat: add headless systemd runtime and status api"
```

---

# 71. 10일차 Report와 환경 기록 Commit하기

Stage:

```bash
git add \
reports/day10_fault_recovery.md \
reports/day10_pi_environment.txt

if [ -d reports/day10_evidence ]; then
  git add reports/day10_evidence
fi
```

Commit:

```bash
git commit -m \
"docs: record day10 service and recovery tests"
```

---

# 72. README에 10일차 운영 명령 추가하기

`README.md`에 다음 내용을 추가합니다.

````md
## Day 10 Headless Operation

### Edge Service

```bash
systemctl status subject13-edge.service
```

```bash
journalctl \
-u subject13-edge.service \
-n 50 \
--no-pager
```

### API Service

```bash
systemctl status subject13-api.service
```

### API

```text
GET /health
GET /status
GET /metrics
```

Example:

```bash
curl http://<RPI_IP>:<API_PORT>/health
```

### Operation Flow

```text
Raspberry Pi Boot
→ systemd
→ Edge Runtime
→ Camera / AI / Stabilization / GPIO
→ AI Event Log
→ Runtime State
→ FastAPI
→ PC
```

### Important

FastAPI가 중지되어도
Edge Runtime은 독립적으로 동작하도록 구성합니다.
````

Commit:

```bash
git add README.md
git commit -m \
"docs: add day10 headless operation guide"
```

---

# 73. 원격 Repository에 Push하기

현재 교육장에 **허용된 내부 GitLab / Gitea Remote**가 연결되어 있다면 다음처럼 Push할 수 있습니다.

```bash
git remote -v
git push
```

Remote가 없거나 외부 Git 사용이 제한된 폐쇄망에서는 Push를 억지로 시도하지 않습니다. **Local Git Commit만으로도 오늘의 Version 기록은 완성**됩니다.

다음 파일은 기존 원칙대로 Remote 저장소에 올리지 않습니다.

```text
Dataset
Model
Runtime State
Raw Log
기업 제공 데이터
```

---

# 74. 10일차가 끝난 시점의 프로젝트 구조

공통 Source:

```text
subject13_edge_ai/
│
├── configs/
│   └── settings.yaml
│
├── reports/
│   ├── ...
│   ├── day09_optimization.md
│   ├── day10_fault_recovery.md
│   └── day10_pi_environment.txt
│
├── scripts/
│   ├── ...
│   ├── check_remote_status.py
│   ├── edge_runtime.py
│   ├── render_systemd_units.py
│   ├── runtime_state_test.py
│   └── status_api.py
│
├── src/
│   ├── ...
│   └── runtime_state.py
│
├── systemd/
│   └── generated/          ← 장치에서 생성, Git 제외
│       ├── subject13-edge.service
│       └── subject13-api.service
│
├── .gitignore
├── README.md
└── requirements.txt
```

Raspberry Pi 실행 중에만 존재:

```text
runtime/
└── edge_state.json

logs/
└── <configs/settings.yaml의 ai_event_log_path>
    → 7일차 AIEventLogger를 10일차 운영 Runtime이 계속 갱신

models/
└── day07_tiny_cnn.onnx
```

systemd 실제 설치 위치:

```text
/etc/systemd/system/
├── subject13-edge.service
└── subject13-api.service
```

---

# 75. 1~10일차 시스템 성장 확인하기

1일차:

```text
가상 입력
→ Decision
→ Log
```

2일차:

```text
PC
→ SSH
→ Raspberry Pi
```

3일차:

```text
Camera
→ GPIO
```

4일차:

```text
Sensor
→ Sampling
→ Timestamp
→ CSV
```

5일차:

```text
Camera
→ Rule
→ Decision
→ GPIO
→ Log
```

6일차:

```text
Camera
→ Dataset
→ Tiny CNN
→ PyTorch Model
```

7일차:

```text
PyTorch
→ ONNX
→ Raspberry Pi
→ AI Inference
```

8일차:

```text
Latency / FPS
→ 병목 측정
```

9일차:

```text
Frame Skip
→ Voting
→ Debounce
→ Hold
→ 안정화
```

10일차:

```text
Raspberry Pi Boot
        ↓
systemd
        ↓
Edge Runtime
        ↓
Camera
→ ONNX AI
→ Stabilization
→ GPIO
        ↓
AI Event Log
        ↓
Runtime State
        ↓
FastAPI
        ↓
PC
```

이제 프로그램은 **사람이 SSH Terminal에서 직접 실행해야만 동작하는 실습 코드**에서 **부팅 후 자동으로 실행되고 외부에서 상태를 확인할 수 있는 운영형 Edge 프로그램**으로 성장했습니다.

---


# 76. 핵심 복습 문제 — 10일차를 마무리하기

교재를 바로 다시 보지 말고 먼저 자신의 말로 답합니다.

### 문제 1

9일차와 10일차의 가장 큰 차이는 무엇인가요?

### 문제 2

Headless 운영에서 `cv2.imshow()` 같은 GUI 창에 의존하면 안 되는 이유는 무엇인가요?

### 문제 3

`runtime/edge_state.json`은 왜 필요한가요?

### 문제 4

`RuntimeStateStore.write()`가 `.tmp` 파일을 먼저 만든 뒤 `os.replace()`를 사용하는 이유는 무엇인가요?

### 문제 5

`runtime_heartbeat_sec`와 `runtime_stale_sec`는 각각 무엇인가요?

### 문제 6

systemd Unit의 `User`, `WorkingDirectory`, `ExecStart`는 각각 어떤 역할인가요?

### 문제 7

systemd에서 `source .venv/bin/activate` 대신 `.venv/bin/python` 절대경로를 사용하는 이유는 무엇인가요?

### 문제 8

`Restart=on-failure`와 `systemctl stop`은 어떤 차이가 있나요?

### 문제 9

Unit 파일을 수정한 뒤 `daemon-reload`가 필요한 이유는 무엇인가요?

### 문제 10

`/health`, `/status`, `/metrics`는 각각 무엇을 확인하기 위한 Endpoint인가요?

### 문제 11

FastAPI가 Camera와 ONNX 추론을 직접 실행하지 않도록 분리한 이유는 무엇인가요?

### 문제 12

Raspberry Pi 내부의 `127.0.0.1:<API_PORT>` 호출은 되는데 PC의 `<RPI_IP>:<API_PORT>` 호출이 안 되면 무엇부터 확인해야 하나요?

### 문제 13

`/health`가 `"edge": "DEGRADED"`를 반환하지만 `"api": "OK"`라면 어떤 의미인가요?

### 문제 14

`systemctl status`와 `journalctl`은 무엇이 다른가요?

### 문제 15

10일차 마지막에 어떤 상태를 남겨야 11일차 Test를 시작할 수 있나요?

### 문제 16

10일차 운영 Event Log는 왜 `raw_state`가 아니라 Voting·Debounce·Hold가 반영된 최종 `state` 변화 기준으로 기록하나요?

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

### 1

9일차는 AI의 **운영 설정을 최적화**하는 날이고, 10일차는 그 Final Pipeline을 **자동실행·자동재시작·상태조회가 가능한 운영형 구조**로 만드는 날입니다.

### 2

Headless 장치는 모니터·키보드·GUI 세션이 없을 수 있습니다. 따라서 상태 확인은 Service, Journal, Runtime State, API 같은 방법으로 해야 합니다.

### 3

Edge Runtime의 최근 상태를 다른 Process인 FastAPI가 읽을 수 있게 해 주며, Headless 상태에서도 현재 Prediction·운영상태·성능·자원을 확인하는 기준점이 됩니다.

### 4

파일을 직접 덮어쓰는 순간 FastAPI가 아직 완성되지 않은 JSON을 읽을 가능성을 줄이기 위해서입니다. 완성된 임시 파일을 만든 뒤 같은 경로의 최종 파일로 교체합니다.

### 5

```text
runtime_heartbeat_sec
→ Edge Runtime이 State를 갱신하는 간격

runtime_stale_sec
→ 마지막 State가 이 시간보다 오래되면
   현재 상태로 신뢰하지 않는 기준
```

### 6

```text
User
→ Process를 실행할 Linux 사용자

WorkingDirectory
→ 상대경로의 기준 폴더

ExecStart
→ 실제 실행할 명령과 Python
```

### 7

systemd는 사용자의 활성화된 Shell 환경을 자동으로 재현하지 않습니다. 따라서 사용할 Python을 Unit에서 직접 지정해야 합니다.

### 8

`Restart=on-failure`는 비정상 종료를 자동 복구하기 위한 설정입니다. `systemctl stop`은 관리자가 의도적으로 중지하는 명령이므로 실패 복구와 같은 의미가 아닙니다.

### 9

Unit 파일을 디스크에서 수정해도 systemd Manager가 이전 정의를 메모리에 가지고 있을 수 있으므로 새 Unit 정의를 다시 읽게 해야 합니다.

### 10

```text
/health
→ API와 Edge Runtime의 생존 / stale 상태

/status
→ 현재 Prediction과 Operational State

/metrics
→ FPS / Latency / CPU / Memory / Temperature 등
```

### 11

API Request가 없거나 Network가 끊겨도 현장의 Camera·AI·GPIO 판단은 계속 동작해야 하기 때문입니다. 또한 한 Process의 장애가 다른 기능까지 같이 죽는 것을 줄일 수 있습니다.

### 12

현재 Raspberry Pi IP, `api_host`, 실제 Listen Port, PC와 Pi의 Network 도달성, 교육장 방화벽 정책을 순서대로 확인합니다.

### 13

FastAPI Process는 정상적으로 응답하지만 Edge Runtime State가 없거나 오래되었거나 `RUNNING`이 아닌 상태일 수 있다는 뜻입니다.

### 14

```text
systemctl status
→ 현재 Service 상태와 최근 일부 메시지

journalctl
→ 시간 순서의 Process 출력 / 오류 / Restart 이력
```

### 15

최소 다음 상태가 필요합니다.

```text
subject13-edge.service
→ enabled + active

subject13-api.service
→ enabled + active

/health
→ api OK / edge OK / stale false

Camera → AI → Stabilization → GPIO
→ 정상

Runtime State
→ 갱신 중

Operational Event Log
→ 현재 Runtime에서 State 변화 Row 생성 확인

Day10 Report / Git
→ 정리 완료
```

이 상태가 11일차의 **운영 Baseline**입니다.

### 16

운영 Event Log는 실제 장치가 외부에 출력한 상태 이력을 남기는 기록입니다. 따라서 순간적인 `raw_state` 흔들림보다 Voting·Debounce·Hold가 반영되어 실제 GPIO에 전달된 최종 `state`를 기준으로 기록해야 합니다.

```text
raw_state 흔들림
→ Stabilizer가 흡수
→ 최종 state 변화 없음
→ Event Log 추가 없음

최종 state 변화
→ GPIO 변화
→ Event Log 새 Row
```

이렇게 해야 11일차 System Test와 13일차 Final Acceptance에서 **실제 운영 출력의 변화 이력**을 증거로 사용할 수 있습니다.

</details>

---

# 77. 자가 체크리스트

수업을 마치기 전에 직접 체크합니다.

- [ ] 9일차 Final Config를 그대로 이어받았는지 확인했다.
- [ ] 새 프로젝트나 새 `.venv`를 만들지 않았다.
- [ ] `runtime_state_path`, Heartbeat, stale 설정의 의미를 설명할 수 있다.
- [ ] `runtime_state_test.py`로 JSON 쓰기·읽기를 확인했다.
- [ ] `edge_runtime.py`를 systemd 전에 직접 실행해 보았다.
- [ ] Runtime State의 `timestamp`가 갱신되는 것을 확인했다.
- [ ] Camera 앞 상황에 따라 `pred_class`, `raw_state`, `state`가 변하는 것을 확인했다.
- [ ] 7일차의 `AIEventLogger`를 새로 만들지 않고 운영 `edge_runtime.py`에서 재사용했다.
- [ ] `ai_event_log_path`의 실제 경로를 확인했다.
- [ ] 첫 운영 State와 이후 최종 State 변화가 Event CSV에 새 Timestamp로 기록되는 것을 확인했다.
- [ ] Raw State만 흔들리고 최종 State가 유지되는 경우 불필요한 중복 Event가 추가되지 않는 것을 확인했다.
- [ ] 생성된 Edge Unit의 `User`, `WorkingDirectory`, `ExecStart`를 직접 확인했다.
- [ ] `ExecStart`가 현재 프로젝트 `.venv/bin/python`을 사용한다.
- [ ] Edge Service의 `start`, `stop`, `restart`, `status` 차이를 안다.
- [ ] `journalctl`에서 Edge Runtime 로그와 오류를 확인했다.
- [ ] Edge Service를 `enable`하고 재부팅 후 자동실행을 확인했다.
- [ ] SIGKILL 후 `Restart=on-failure` 자동복구를 확인했다.
- [ ] Camera / Model / Config 오류 중 최소 하나를 직접 복구했다.
- [ ] FastAPI와 Uvicorn이 현재 Raspberry Pi `.venv`에서 실행된다.
- [ ] `/health`, `/status`, `/metrics`의 역할을 구분한다.
- [ ] Raspberry Pi 내부에서 API를 확인했다.
- [ ] PC에서 현재 `<RPI_IP>`와 `<API_PORT>`로 API를 확인했다.
- [ ] API가 중지되어도 Edge AI가 독립적으로 동작하는 것을 확인했다.
- [ ] API Service도 `enable`하고 재부팅 후 자동실행을 확인했다.
- [ ] `systemd/generated/`이 장치별 생성물이라는 것을 이해한다.
- [ ] Mini Challenge 뒤 Config와 Source를 정상 상태로 복원했다.
- [ ] `reports/day10_fault_recovery.md`를 완성했다.
- [ ] 필요한 Evidence와 환경 기록을 남겼다.
- [ ] README에 10일차 운영 명령을 정리했다.
- [ ] Git Commit으로 10일차 정상 상태를 저장했다.
- [ ] 수업 종료 시 두 Service가 `enabled + active` 상태이다.
- [ ] `/health`가 최종적으로 `edge: OK`, `stale: false`를 반환한다.
- [ ] 11일차에는 이 상태를 Unit / Integration / System Test 대상으로 사용한다는 것을 설명할 수 있다.

---

# 78. 수업 종료 전 최종 확인

Raspberry Pi에서 다음을 순서대로 확인합니다.

## Edge Service

```bash
systemctl is-enabled \
subject13-edge.service
```

```bash
systemctl status \
subject13-edge.service \
--no-pager
```

## API Service

```bash
systemctl is-enabled \
subject13-api.service
```

```bash
systemctl status \
subject13-api.service \
--no-pager
```

## Runtime State

```bash
cat runtime/edge_state.json
```

## Operational Event Log

먼저 실제 경로를 확인합니다.

```bash
grep -E "^ai_event_log_path:" \
configs/settings.yaml \
configs/device.local.yaml \
2>/dev/null
```

그다음 현재 설정의 경로를 사용합니다.

```bash
tail -n 10 <AI_EVENT_LOG_PATH>
```

Camera 앞 상황을 한 번 바꾸고 다시 `tail`했을 때 **새 Timestamp Row가 추가되는지** 확인합니다.

## Edge Journal

```bash
journalctl \
-u subject13-edge.service \
-n 30 \
--no-pager
```

## API Journal

```bash
journalctl \
-u subject13-api.service \
-n 30 \
--no-pager
```

## API

```bash
curl \
http://127.0.0.1:<API_PORT>/health
```

```bash
curl \
http://127.0.0.1:<API_PORT>/status
```

```bash
curl \
http://127.0.0.1:<API_PORT>/metrics
```

PC에서도:

```bash
curl \
http://<RPI_IP>:<API_PORT>/health
```

을 확인합니다.

---

# 79. 최종 장애 복구 확인

오늘 최소 두 가지 장애를 다시 확인합니다.

예:

```text
Case 1
Edge Process SIGKILL

Case 2
FastAPI Process SIGKILL
```

확인할 항목:

```text
어떤 Service가 실패했는가?

다른 Service는 계속 살아 있었는가?

systemd가 다시 시작했는가?

journalctl에 무엇이 기록되었는가?

/health는 어떻게 변했는가?

복구 후 다시 OK가 되었는가?
```

---

# 80. 다음 날 연결

10일차까지는 다음 운영 구조를 만들었습니다.

```text
자동실행
+
자동재시작
+
상태 기록
+
HTTP 상태 조회
```

이제 11일차에는 새로운 기능을 계속 추가하기보다 **현재 시스템을 실제 Test 대상으로 사용**합니다.

```text
10일차 운영 Baseline
        ↓
Unit Test
→ Decision / Stabilizer / Runtime State
        ↓
Integration Test
→ Fixed Image → Preprocess → ONNX → Decision
        ↓
System Test
→ Camera → Edge Runtime → GPIO → Event Log → systemd → Runtime State → FastAPI
        ↓
한 변수 실험
→ Accuracy / Latency / FPS / Stability 비교
        ↓
Failure Case 수집
        ↓
원인 영역 분류
        ↓
Trade-off 판단
```

즉 11일차에는:

> **“기능이 하나씩 동작한다.”**

에서 더 나아가,

> **“전체 시스템이 요구한 조건을 만족하는가?”**

를 Test와 기록을 통해 확인합니다.

11일차는 먼저 오늘 남긴 두 Service와 API가 정상인지 다시 확인한 뒤, 이 상태를 **`day10-operational-baseline` 기준점**으로 고정하고 Unit Test → Integration Test → System Test로 진행합니다.

따라서 오늘 수업을 마칠 때 임의의 Challenge 설정이나 고장난 Unit을 남겨 두지 않습니다.