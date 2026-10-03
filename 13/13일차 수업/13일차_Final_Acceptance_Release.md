
## 12일차와 13일차는 어디가 이어질까요?

12일차에는 Raspberry Pi에서 만든 Edge AI 구조를 Jetson으로 확장하며 다음을 확인했습니다.

```text
Raspberry Pi Tested Baseline
        ↓
11일차에서 실제 채택한 운영 Model / Metadata 확인
        ↓
장치 독립 / 장치 종속 Module 구분
        ↓
Jetson Linux / JetPack / CUDA / TensorRT 확인
        ↓
같은 Source + 현재 운영 Baseline ONNX / Metadata 이동
        ↓
Hash 동일성 확인
        ↓
TensorRT Engine Build / Model-only Benchmark
        ↓
Camera Contract
        ↓
Inference Contract
        ↓
Migration Map
        ↓
교과 14 준비
```

하지만 12일차의 마지막에는 Jetson 실험 상태로 끝내지 않았습니다.

다음 13일차 Final Acceptance를 위해 **Raspberry Pi 운영 Baseline을 다시 정상 상태로 복원**했습니다.

```text
12일차 마지막

Raspberry Pi Config 복원
        ↓
subject13-edge.service
active + enabled
        ↓
subject13-api.service
active + enabled
        ↓
/health 정상
        ↓
runtime/edge_state.json 갱신
        ↓
현재 onnx_model_path / onnx_meta_path가
11일차 Tested Baseline과 일치
        ↓
ai_event_log_path가 실제 운영 Event CSV를 가리킴
        ↓
13일차 Handoff Gate PASS
```

따라서 오늘은 Camera, ONNX, Decision, Stabilizer, GPIO, systemd, FastAPI를 새로 만드는 것이 아닙니다.

**이전 일차에서 만든 파일과 서비스를 그대로 다시 사용하여 최종 검증**합니다.

---

## 13일차가 끝나면 무엇이 달라질까요?

13일차가 끝나면 교과 13의 Guided Lab 중심 수업은 종료됩니다.

그다음 교과 14에서는 학생이 팀과 함께 직접 문제를 정의하고, 데이터를 확인하고, 모델과 Runtime을 선택하고, 결과를 검증하는 **실무형 포트폴리오 프로젝트**로 넘어갑니다.

```text
교과 13
Guided Edge AI 학습

Raspberry Pi
USB Camera
Tiny CNN
ONNX Runtime CPU
Decision / Stabilization
GPIO / Log
systemd / FastAPI
Test / Failure Analysis
        ↓
12일차 Jetson Migration 경험
        ↓
13일차 Final Acceptance / Release
        ↓

교과 14
Student-led Manufacturing Vision AI Project

Jetson Orin Nano Super
Industrial Camera
Object Detection
ONNX / TensorRT GPU
Inspection Result
Log / Statistics
Performance
Project Documentation
Portfolio
```

오늘 교과 14의 프로젝트를 미리 구현하지는 않습니다.

대신 **교과 13에서 무엇을 검증했고, 무엇을 그대로 가져갈 수 있으며, 교과 14에서 무엇이 새로 바뀌는지** 명확하게 정리하고 끝냅니다.

---

# 오늘의 학습 목표

수업이 끝나면 다음을 할 수 있어야 합니다.

```text
1. 12일차 Handoff 상태에서 Raspberry Pi Final Acceptance를 시작할 수 있다.

2. Raspberry Pi 재부팅 후 사람이 Python을 직접 실행하지 않아도
   Edge Runtime과 FastAPI가 자동 실행되는지 확인할 수 있다.

3. Camera → AI → Decision → Stabilization → GPIO → Log → API의
   전체 동작을 한 번의 Acceptance 흐름으로 확인할 수 있다.

4. AI Prediction과 Operational Decision을 구분해서 설명할 수 있다.

5. 긴 Benchmark를 다시 하지 않고
   기존 성능 Report와 현재 /metrics를 이용해 최종 성능을 정리할 수 있다.

6. 비정상 종료 후 systemd의 자동재시작이 동작하는지 확인하고
   journalctl에서 복구 근거를 찾을 수 있다.

7. 다른 학습자가 README만 보고 장치를 운영할 수 있는지 확인할 수 있다.

8. final_check.py의 출력이 어느 설정·파일·Runtime State에서 만들어지는지
   코드로 역추적할 수 있다.

9. 최종 Acceptance 결과를 ACCEPT / CONDITIONAL ACCEPT / NOT ACCEPT로 기록할 수 있다.

10. README · Report · Git Commit · Final Tag까지 최종 Release를 정리할 수 있다.

11. 교과 13의 Raspberry Pi Edge 경험과 12일차 Jetson Migration 결과를
    교과 14 제조 Vision AI 프로젝트의 시작점으로 연결할 수 있다.
```

---

# 오늘의 4시간 학습 흐름

13일차는 **실제 운영 확인과 최종 인수시험 자체를 오늘의 실습**으로 진행합니다.

| 학습 블록 | 권장 시간 | 무엇을 하나요? | 완료 기준 |
|---|---:|---|---|
| 1 | 55분 | Reboot · systemd · API Baseline Acceptance | 사람이 Python을 실행하지 않아도 두 Service와 API가 정상이다 |
| 2 | 65분 | Camera · AI · Decision · GPIO · Log · Performance Acceptance | NORMAL/WARNING 전체 기능과 기록 근거를 확인한다 |
| 3 | 60분 | Failure Recovery · Peer Operation · `final_check.py` | 장애가 복구되고 다른 사람도 운영할 수 있다 |
| 4 | 60분 | README · Final Acceptance · Git Release · Course 14 Handoff · 복습 | 교과 13을 Release하고 교과 14 시작점을 설명할 수 있다 |

**총 240분**을 기준으로 합니다.

재부팅 시간, Camera 상태, 교육장 Network 상태에 따라 각 단계 시간은 조금 달라질 수 있습니다.

오늘은 시간이 부족하다고 기능 검증을 생략하고 문서만 작성하지 않습니다.

반대로 이미 8~12일차에서 충분히 수행한 Benchmark나 반복 실험을 다시 길게 수행하지도 않습니다.

---

# 오늘 사용할 기존 결과와 새로 만들 파일

## 이전 일차의 파일을 다시 사용합니다

오늘 새로 만드는 프로젝트는 없습니다.

다음은 **이미 이전 일차에서 만든 파일**입니다.

```text
subject13_edge_ai/
│
├── configs/
│   ├── settings.yaml
│   └── device.local.yaml          ← 장치에 따라 존재, 보통 Git 제외
│
├── src/
│   ├── ai_preprocess.py
│   ├── camera_input.py
│   ├── config_loader.py
│   ├── decision.py
│   ├── event_logger.py
│   ├── gpio_output.py
│   ├── inference_onnx.py
│   ├── performance.py
│   ├── resource_monitor.py
│   ├── runtime_state.py
│   └── stabilizer.py
│
├── scripts/
│   ├── edge_runtime.py
│   ├── status_api.py
│   └── ...
│
├── models/
│   ├── <현재 운영 Baseline ONNX>
│   └── <현재 운영 Baseline Metadata>
│
├── systemd/
├── tests/
├── reports/
├── docs/
├── README.md
└── requirements.txt
```

오늘 이 파일들을 새로 만드는 것처럼 다시 작성하지 않습니다.

특히 Model 파일명은 `day07_tiny_cnn.onnx`처럼 기억으로 고정하지 않습니다.  
12일차에서 확인한 것처럼 **현재 병합된 Config의 `onnx_model_path`와 `onnx_meta_path`가 가리키는 11일차 Tested Baseline**을 최종 기준으로 사용합니다.

각 파일이 연결되어 정상 운영되는지만 확인합니다.

## 오늘 새로 만들거나 완성할 핵심 파일

```text
reports/day13_acceptance_test.md
→ 오늘의 모든 Acceptance 결과를 한곳에 기록

scripts/final_check.py
→ 현재 Model / Metadata / Operational Event Log / Runtime State를 한 번에 확인하는 작은 최종 점검 Script

docs/course14_handoff.md
→ 교과 13에서 교과 14로 가져갈 구조와 변경점을 정리

README.md
→ 기존 파일을 새로 만들지 않고 최종 운영 정보만 보완
```

---

# 오늘 사용할 환경값을 먼저 확인합니다

IP, Hostname, Port, Camera 번호는 장비와 Network에 따라 달라질 수 있습니다.

교재 예시값을 그대로 외워서 사용하지 않습니다.

먼저 다음 빈칸을 자신의 환경값으로 채웁니다.

```text
Raspberry Pi 사용자 이름 : ______________________________
Raspberry Pi Hostname    : ______________________________
Raspberry Pi IPv4        : ______________________________
Project Path             : ______________________________

API Port                 : ______________________________
Camera Index             : ______________________________

AI Event Log Path        : ______________________________
Warning Image Directory  : ______________________________

현재 ONNX Model          : ______________________________
현재 ONNX Metadata       : ______________________________
현재 Model Input Size    : ______________________________
현재 Confidence Threshold: ______________________________
```

이 교재에서는 다음 표기를 사용합니다.

```text
<RPI_USER>
→ 자신의 Raspberry Pi 사용자 이름

<RPI_HOSTNAME>
→ 현재 Raspberry Pi Hostname

<RPI_IP>
→ 현재 Raspberry Pi IPv4

<API_PORT>
→ 현재 설정에 실제로 사용되는 API Port

<AI_EVENT_LOG_PATH>
→ 현재 설정의 AI Event Log 파일 경로

<AI_WARNING_IMAGE_DIR>
→ WARNING 이미지를 저장하도록 설정했다면 해당 폴더

<CURRENT_ONNX>
→ 현재 병합된 Config의 `onnx_model_path`

<CURRENT_META>
→ 현재 병합된 Config의 `onnx_meta_path`
```

---

## IP가 예전과 다르면 어떻게 할까요?

DHCP 환경에서는 재부팅 뒤 IPv4가 달라질 수 있습니다.

예전에 사용했던 IP가 기억난다는 이유만으로 계속 같은 주소를 입력하지 않습니다.

가능한 경우 Raspberry Pi에서 다음을 확인합니다.

```bash
hostname
hostname -I
```

mDNS가 정상인 환경이라면 PC에서 Hostname으로 접속할 수도 있습니다.

```bash
ssh <RPI_USER>@<RPI_HOSTNAME>.local
```

환경에서 `.local` 이름 해석이 되지 않는다면 현재 IPv4를 확인한 뒤 사용합니다.

```bash
ssh <RPI_USER>@<RPI_IP>
```

---

## API Port는 어떻게 확인할까요?

프로젝트 루트에서 다음을 확인합니다.

```bash
grep -E "^api_port:" \
  configs/settings.yaml \
  configs/device.local.yaml \
  2>/dev/null
```

`device.local.yaml`에서 같은 Key를 덮어쓰도록 구성했다면 **실제 장치별 값이 최종값**이 될 수 있습니다.

두 파일이 모두 출력된다면 어떤 값이 실제로 적용되는지 이전 일차의 `config_loader.py` 병합 규칙을 다시 확인합니다.

오늘은 `8000` 같은 특정 Port를 무조건 사용하지 않습니다.

---

# 오늘의 전체 Pipeline을 먼저 확인합니다

오늘 검증할 시스템은 다음입니다.

```text
Raspberry Pi Power ON
        ↓
Boot
        ↓
systemd
        ↓
subject13-edge.service
        ↓
scripts/edge_runtime.py
        ↓
CameraInput
        ↓
Preprocess
        ↓
ONNXClassifier
        ↓
Class + Confidence
        ↓
Operational Decision
        ↓
Voting / Debounce / Hold
        ↓
NORMAL / WARNING
        ↓
GPIO
Green LED / Red LED / Buzzer
        ↓
Operational AI Event Log
(첫 운영 State + 최종 output_state 변화)
        ↓
runtime/edge_state.json
        ↓
subject13-api.service
        ↓
scripts/status_api.py
        ↓
/health /status /metrics
        ↓
PC
```

동시에 다음 정보도 확인합니다.

```text
Latency / FPS
CPU / Memory / Temperature
journalctl
Auto Restart
```

13일차에는 이 구조를 더 복잡하게 만들지 않습니다.

**끝까지 실제로 연결되어 있는지 확인하고 Release합니다.**

---

# 오늘의 성공 기준

```text
Reboot
        ↓
사람이 Python을 직접 실행하지 않음
        ↓
Edge Service active + enabled
        ↓
API Service active + enabled
        ↓
/health 정상
        ↓
현재 ONNX / Metadata가
11일차 Tested Baseline과 일치
        ↓
NORMAL 확인
        ↓
WARNING 확인
        ↓
GPIO 확인
        ↓
현재 Acceptance 실행으로
Operational Event Log 새 Row 생성 확인
        ↓
/metrics 확인
        ↓
대표 장애 1개 자동 복구
        ↓
Peer Operation 확인
        ↓
final_check PASS
        ↓
Final Acceptance 판정
        ↓
README / Report / Git / Tag
        ↓
Course 14 Handoff
```

---

# PART A. Final Boot · Service · API Acceptance — 55분

# 1. 왜 재부팅부터 시작할까요?

10일차 이후 `systemd`를 사용하기 시작하면서 프로그램의 운영 방식이 바뀌었습니다.

개발 중에는 다음처럼 직접 실행했습니다.

```text
사람
→ Terminal 열기
→ python -m scripts....
→ 프로그램 실행
```

하지만 운영형 Edge 장치는 사람이 매번 Python 명령을 입력해야 한다면 자동 운영이라고 보기 어렵습니다.

최종 Acceptance에서는 다음을 확인합니다.

```text
Power / Reboot
        ↓
Linux Boot
        ↓
systemd
        ↓
Edge Runtime 자동실행
        ↓
FastAPI 자동실행
```

따라서 오늘의 첫 번째 시험은 **“재부팅 뒤 내가 Python을 직접 실행하지 않아도 시스템이 살아나는가?”**입니다.

---

# 2. 재부팅 전 12일차 Handoff Gate를 짧게 확인합니다

Raspberry Pi에 접속합니다.

```bash
ssh <RPI_USER>@<RPI_IP>
```

프로젝트로 이동합니다.

```bash
cd ~/ai_vision/subject13_edge_ai
```

프로젝트 경로를 다른 위치에 만들었다면 자신의 실제 경로로 이동합니다.

현재 상태를 확인합니다.

```bash
pwd
whoami
hostname
hostname -I
```

재부팅 전에 **12일차에서 넘겨받은 현재 운영 Model / Metadata / Event Log Contract**도 실제 병합 Config에서 확인합니다.

```bash
python - <<'PY'
import json
from pathlib import Path

from src.config_loader import load_config

config = load_config("configs/settings.yaml")

for key in (
    "onnx_model_path",
    "onnx_meta_path",
    "ai_confidence_threshold",
    "ai_event_log_path",
    "runtime_state_path",
    "api_port",
):
    print(f"{key}: {config[key]}")

model_path = Path(str(config["onnx_model_path"]))
meta_path = Path(str(config["onnx_meta_path"]))
event_path = Path(str(config["ai_event_log_path"]))

print(f"model exists: {model_path.exists()}")
print(f"metadata exists: {meta_path.exists()}")
print(f"event log exists: {event_path.exists()}")

if meta_path.exists():
    metadata = json.loads(
        meta_path.read_text(encoding="utf-8")
    )
    print(
        "metadata image_size:",
        metadata.get("image_size"),
    )
    print(
        "metadata classes:",
        metadata.get("classes"),
    )
PY
```

여기서 중요한 것은 특정 파일명이 나오는 것이 아닙니다.

```text
현재 Config가 가리키는 ONNX
+
그 ONNX와 짝이 되는 Metadata
+
실제 파일 존재
+
11일차 Tested Baseline과 같은 운영 조합
+
실제 ai_event_log_path
```

가 일치해야 합니다.

11일차에서 별도 Model을 공식 채택하지 않았다면 Day7의 96×96 Model일 수 있고, 11일차에서 다른 후보를 실제 운영 Baseline으로 채택했다면 파일명과 Input Size가 달라질 수 있습니다. **13일차에서는 현재 Tested Baseline을 그대로 Acceptance합니다.**

두 Service가 현재 정상인지 확인합니다.

```bash
systemctl is-active subject13-edge.service
systemctl is-active subject13-api.service
```

정상이라면 각각 다음과 같이 보입니다.

```text
active
active
```

자동실행 여부도 확인합니다.

```bash
systemctl is-enabled subject13-edge.service
systemctl is-enabled subject13-api.service
```

정상이라면 각각 다음과 같습니다.

```text
enabled
enabled
```

이 단계가 실패한다면 재부팅 Acceptance를 시작하기 전에 12일차 Handoff 상태부터 복구합니다.

---

# 3. Raspberry Pi를 완전히 재부팅합니다

Raspberry Pi에서 실행합니다.

```bash
sudo reboot
```

SSH 연결이 끊기는 것은 정상입니다.

다음 흐름으로 이해합니다.

```text
sudo reboot
        ↓
SSH Session 종료
        ↓
Raspberry Pi Reboot
        ↓
Network 다시 연결
        ↓
systemd Service 시작
```

부팅이 끝날 때까지 기다립니다.

그다음 PC에서 다시 접속합니다.

Hostname 접속이 되는 환경:

```bash
ssh <RPI_USER>@<RPI_HOSTNAME>.local
```

IPv4를 사용하는 환경:

```bash
ssh <RPI_USER>@<RPI_IP>
```

예전 IP로 접속되지 않는다면 먼저 현재 IP가 달라졌는지 확인합니다.

---

# 4. 중요한 규칙 — 아직 Python을 직접 실행하지 않습니다

프로젝트로 이동합니다.

```bash
cd ~/ai_vision/subject13_edge_ai
```

여기서 다음 명령을 실행하지 않습니다.

```text
python -m scripts.edge_runtime
X

python -m scripts.status_api
X
```

오늘의 첫 번째 Acceptance는 systemd 자동실행을 보는 시험이기 때문입니다.

내가 직접 Python을 실행해 버리면 다음을 구분하기 어렵습니다.

```text
systemd가 자동으로 실행했는가?

또는

내가 Terminal에서 실행했는가?
```

---

# 5. Edge Service 자동실행 확인

```bash
systemctl status \
  subject13-edge.service \
  --no-pager
```

다음 핵심을 찾습니다.

```text
Active: active (running)
```

자동실행 설정도 확인합니다.

```bash
systemctl is-enabled subject13-edge.service
```

정상:

```text
enabled
```

### 이 결과는 어디에서 만들어졌을까요?

`active (running)`은 Python의 `print()` 결과가 아닙니다.

다음 연결을 봅니다.

```text
systemd
        ↓
subject13-edge.service
        ↓
ExecStart
        ↓
프로젝트 .venv/bin/python
        ↓
-m scripts.edge_runtime
        ↓
Process 실행 중
        ↓
systemctl status
active (running)
```

10일차에서 만든 Service Unit의 `ExecStart`, `WorkingDirectory`, `User`가 올바르기 때문에 이 Process가 실행됩니다.

---

# 6. FastAPI Service 자동실행 확인

```bash
systemctl status \
  subject13-api.service \
  --no-pager
```

정상 핵심:

```text
Active: active (running)
```

자동실행:

```bash
systemctl is-enabled subject13-api.service
```

정상:

```text
enabled
```

두 Service는 역할이 다릅니다.

```text
subject13-edge.service
→ Camera / AI / Decision / GPIO / Runtime State

subject13-api.service
→ Runtime State를 읽어서 HTTP JSON으로 제공
```

API Service가 Edge AI를 대신 실행하는 것이 아닙니다.

---

# 7. 현재 API Port 확인

Port를 고정값으로 가정하지 않습니다.

```bash
grep -E "^api_port:" \
  configs/settings.yaml \
  configs/device.local.yaml \
  2>/dev/null
```

확인한 실제 값을 `<API_PORT>`에 사용합니다.

---

# 8. Raspberry Pi 내부에서 `/health` 확인

```bash
curl \
  http://127.0.0.1:<API_PORT>/health
```

정상 형태의 예시는 다음과 비슷합니다.

```json
{
  "api": "OK",
  "edge": "OK",
  "edge_status": "RUNNING",
  "state_age_sec": 0.8,
  "stale": false
}
```

실제 숫자와 JSON의 Key 순서는 환경에 따라 달라질 수 있습니다.

### 결과 해석

```text
api = OK
→ FastAPI가 요청에 응답하고 있음

edge = OK
→ 최근 Runtime State가 운영 기준상 정상

edge_status = RUNNING
→ Edge Runtime이 실행 상태를 기록 중

stale = false
→ Runtime State가 너무 오래되지 않음
```

---

# 9. PC에서도 API가 보이는지 확인

PC에서 실행합니다.

```bash
curl \
  http://<RPI_IP>:<API_PORT>/health
```

Raspberry Pi 내부에서는 되지만 PC에서 되지 않는다면 다음을 구분합니다.

```text
Raspberry Pi 내부 curl PASS
+
PC curl FAIL
        ↓
Edge Runtime 자체보다는
Network / Listen Address / Firewall / IP / Port 쪽을 먼저 확인
```

확인 순서:

```text
<RPI_IP>가 현재 값인가?
        ↓
<API_PORT>가 현재 설정과 같은가?
        ↓
API Service가 active인가?
        ↓
어느 주소와 Port에서 Listen 중인가?
        ↓
PC와 Pi가 서로 도달 가능한 Network인가?
```

Listen 상태 예:

```bash
ss -ltnp | grep ":<API_PORT>"
```

---

# 10. `/status`와 `/metrics` 확인

PC 또는 Raspberry Pi에서 실제 주소를 사용합니다.

```bash
curl \
  http://<RPI_IP>:<API_PORT>/status
```

확인할 대표 값:

```text
status
state
raw_state
pred_class
confidence
confidence_threshold
stale
```

성능과 Resource 상태:

```bash
curl \
  http://<RPI_IP>:<API_PORT>/metrics
```

확인할 대표 값:

```text
loop_fps
inference_per_sec
mean_inference_ms
p95_inference_ms
cpu_percent
memory_percent
temperature_c
stale
```

정확한 숫자는 장치 상태에 따라 달라질 수 있습니다.

오늘은 숫자를 서로 맞추는 것이 아니라 **정상적으로 최신 상태가 전달되는지** 확인합니다.

---

# 11. 실행 결과를 코드로 역추적합니다

이번에는 `curl` 결과만 보고 넘어가지 않습니다.

다음 연결을 직접 확인합니다.

```text
scripts/edge_runtime.py
        ↓ write
runtime/edge_state.json
        ↓ read
src/runtime_state.py
        ↓
scripts/status_api.py
        ↓
/health /status /metrics
        ↓
curl 결과
```

`runtime/edge_state.json`을 확인합니다.

```bash
cat runtime/edge_state.json
```

그다음 `scripts/status_api.py`에서 다음 위치를 찾습니다.

```text
/health
→ health()

/status
→ status()

/metrics
→ metrics()
```

다음 질문에 직접 답합니다.

```text
1. pred_class는 FastAPI가 새로 추론해서 만든 값인가?

2. confidence는 어느 파일에 먼저 기록되어 있는가?

3. stale은 어떤 정보로 계산하는가?

4. loop_fps는 API가 Benchmark를 새로 실행해서 만든 값인가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
1. 아니다.
   Edge Runtime이 AI 추론 결과를 Runtime State에 기록하고,
   FastAPI는 그 상태를 읽어 전달한다.

2. runtime/edge_state.json에 Edge Runtime이 기록한다.

3. Runtime State의 timestamp와 현재 시각 차이를
   runtime_stale_sec 기준과 비교한다.

4. 아니다.
   Edge Runtime이 기록한 성능 값을 FastAPI가 읽어 전달한다.
```

</details>

---

# 12. 오늘의 Acceptance Report를 만듭니다

## 이 파일은 왜 필요할까요?

오늘은 여러 기능을 확인합니다.

마지막에 기억으로 “다 됐다”고 적으면 어떤 항목을 실제로 확인했는지 알기 어렵습니다.

그래서 **검증할 때마다 바로 결과를 기록**합니다.

새 파일:

```text
reports/day13_acceptance_test.md
```

처음에는 다음 뼈대만 작성합니다.

````md
# Day 13 Final Acceptance Test

## Environment

Raspberry Pi Hostname:

Raspberry Pi IPv4:

Project Path:

API Port:

Camera Index:

ONNX Model:

ONNX Metadata:

Model Input Size:

Confidence Threshold:

Operational AI Event Log Path:

## 1. Boot & Service Acceptance

재부팅 후 사람이 직접 Python 실행:

- 하지 않음 / 실행함

Edge Service Active:

- PASS / FAIL

Edge Service Enabled:

- PASS / FAIL

API Service Active:

- PASS / FAIL

API Service Enabled:

- PASS / FAIL

/health:

- PASS / FAIL

/status:

- PASS / FAIL

/metrics:

- PASS / FAIL

Runtime State Fresh:

- PASS / FAIL

## 2. Functional Acceptance

## 3. Final Performance Summary

## 4. Recovery Acceptance

## 5. Peer Operation Test

## 6. Final Check

## 7. Final Acceptance

## 8. Course 14 Handoff
````

지금 모든 빈칸을 채우지 않습니다.

각 실습이 끝날 때 해당 부분만 작성합니다.

---

# 13. PART A에서 자주 만나는 오류

## 오류 A — 재부팅 뒤 SSH 접속 실패

```text
오류
→ SSH 연결 실패

원인 확인
→ Raspberry Pi가 부팅 중인가?
→ 현재 IP가 달라졌는가?
→ Hostname 해석이 되는가?
→ PC와 Pi가 같은 교육장 Network에서 통신 가능한가?

수정
→ 실제 Hostname / IP 확인
→ Network 상태 복구

재실행
→ ssh <RPI_USER>@<RPI_IP>
```

## 오류 B — Service가 `failed`

먼저 상태를 봅니다.

```bash
systemctl status \
  subject13-edge.service \
  --no-pager
```

최근 로그:

```bash
journalctl \
  -u subject13-edge.service \
  -n 50 \
  --no-pager
```

확인 순서:

```text
WorkingDirectory
→ Python Path
→ Config
→ Model Path
→ Camera
→ Permission
```

원인을 수정한 뒤 재시작합니다.

```bash
sudo systemctl restart subject13-edge.service
```

다시 확인합니다.

```bash
systemctl is-active subject13-edge.service
```

## 오류 C — `/health`가 `DEGRADED`

재부팅 직후에는 Edge Runtime이 첫 Runtime State를 기록하기 전에 API가 먼저 응답하여 잠시 `DEGRADED`가 보일 수 있습니다. 몇 초 기다린 뒤 한 번 더 확인합니다. 계속 `DEGRADED`라면 아래 순서로 원인을 확인합니다.

API가 응답한다는 사실과 Edge Runtime이 정상이라는 사실은 다릅니다.

```text
api = OK
+
edge = DEGRADED
        ↓
FastAPI Process는 살아 있음
하지만
Edge Runtime State가 오래되었거나 RUNNING이 아님
```

확인:

```bash
cat runtime/edge_state.json
```

```bash
journalctl \
  -u subject13-edge.service \
  -n 50 \
  --no-pager
```

원인을 수정한 뒤 다시 `/health`를 확인합니다.

---

# PART B. Functional · Log · Performance Acceptance — 65분

# 14. 왜 기능을 한 번 더 확인할까요?

Service가 `active`라고 해서 모든 기능이 정상이라는 뜻은 아닙니다.

예를 들어 다음과 같은 상황도 가능합니다.

```text
Process는 실행 중
하지만 Camera Frame 실패
```

또는:

```text
AI Prediction은 정상
하지만 GPIO 출력 실패
```

따라서 다음 전체 연결을 실제 입력으로 확인합니다.

```text
Camera
        ↓
Preprocess
        ↓
ONNX AI
        ↓
Prediction
        ↓
Operational Decision
        ↓
Stabilization
        ↓
GPIO
        ↓
Operational Event Log / Runtime State
        ↓
API
```

오늘은 반복 성능 실험이 아니라 **대표 NORMAL 1회 + 대표 WARNING 1회**를 중심으로 확인합니다.

---

# 14-1. 현재 Acceptance 실행 전 Operational Event Log 기준점 기록

10~11일차에서 운영 `edge_runtime.py`는 7일차의 `AIEventLogger`를 재사용하여 **첫 운영 State와 이후 최종 `output_state`가 바뀌는 순간**을 Event CSV에 기록하도록 연결했습니다.

13일차에서는 단순히 Log 파일이 존재하는지만 보면 안 됩니다. 과거 7일차나 10~11일차에 만들어진 Row를 보고 현재 Acceptance가 정상이라고 오판할 수 있기 때문입니다.

따라서 NORMAL / WARNING 입력을 보여주기 **전에** 현재 Row 수와 마지막 Row를 증거로 남깁니다.

먼저 실제 운영 Log 경로를 병합 Config에서 읽습니다.

```bash
EVENT_LOG_PATH="$(
  python -c   'from src.config_loader import load_config; print(load_config("configs/settings.yaml")["ai_event_log_path"])'
)"

echo "$EVENT_LOG_PATH"
```

Evidence 폴더:

```bash
mkdir -p reports/day13_evidence
```

파일이 실제로 있는지 확인합니다.

```bash
ls -lh "$EVENT_LOG_PATH"
```

현재 Row 수:

```bash
wc -l "$EVENT_LOG_PATH"   | tee reports/day13_evidence/event_log_before_rows.txt
```

현재 마지막 Row:

```bash
tail -n 1 "$EVENT_LOG_PATH"   | tee reports/day13_evidence/event_log_before_last.txt
```

이 값은 **“현재 Acceptance 입력을 넣기 전 상태”**의 기준점입니다.

> Event Logger는 모든 Frame을 기록하지 않습니다.  
> 따라서 시간이 흘렀다는 이유만으로 Row가 계속 늘어나는 것이 정상은 아닙니다. **최종 운영 State가 실제로 바뀌어야 새 Row가 추가**됩니다.

---

# 15. NORMAL 상태 Acceptance

Camera 앞에서 경고 대상을 제거합니다.

현재 상태를 확인합니다.

```bash
curl \
  http://<RPI_IP>:<API_PORT>/status
```

정상 입력에서 최종 운영 상태가 다음과 같은지 확인합니다.

```text
state
→ NORMAL
```

Raspberry Pi의 출력도 확인합니다.

```text
Green LED
→ 정상 상태 표시
```

장치 배선이나 최종 출력정책이 다른 경우에는 자신의 프로젝트에서 정의한 NORMAL 출력을 확인합니다.

Report에 기록합니다.

```md
### NORMAL

Input Condition:

Prediction:

Confidence:

Operational State:

Green LED 또는 정상 출력:

- PASS / FAIL
```

---

# 16. WARNING 상태 Acceptance

6~7일차부터 사용했던 경고 대상 또는 최종 검증에 사용한 동일한 조건을 Camera 앞에 보여줍니다.

새로운 임의 대상을 갑자기 기준으로 사용하지 않습니다.

상태를 확인합니다.

```bash
curl \
  http://<RPI_IP>:<API_PORT>/status
```

대표적으로 다음을 봅니다.

```text
pred_class
→ warning class인지 확인

confidence
→ 실제 값 확인

state
→ WARNING인지 확인
```

GPIO 출력도 확인합니다.

```text
Red LED
→ WARNING 출력

Buzzer
→ 최종 설정에서 사용하는 경우 확인
```

Report:

```md
### WARNING

Input Condition:

Prediction:

Confidence:

Confidence Threshold:

Operational State:

Red LED:

- PASS / FAIL

Buzzer:

- PASS / FAIL / N/A
```

WARNING 확인이 끝나면 경고 대상을 제거하고 최종 상태가 다시 `NORMAL`로 복귀하는지도 한 번 확인합니다.

```bash
curl   http://<RPI_IP>:<API_PORT>/status
```

이 과정은 단순 반복이 아닙니다. **NORMAL → WARNING → NORMAL**의 실제 운영 State 변화가 발생하면 뒤에서 현재 Acceptance가 만든 Event Row를 더 명확하게 증명할 수 있습니다.

---

# 17. Prediction과 Operational Decision을 마지막으로 구분합니다

다음 두 값은 같은 것이 아닙니다.

```text
AI Prediction
→ Model이 만든 Class + Confidence
```

예:

```text
warning_red
Confidence 0.86
```

운영 판정은 그 결과에 운영 기준을 적용합니다.

```text
Prediction
+
Confidence
+
Operational Threshold
        ↓
NORMAL / WARNING
```

예를 들어 Threshold가 `0.90`이라고 가정하고 실제 Confidence가 `0.86`이라면:

```text
Prediction
→ warning_red 0.86

Operational Decision
→ 0.86 < 0.90
→ NORMAL 가능
```

따라서 **“AI가 warning_red라고 예측했다”와 “최종 장치가 WARNING을 출력했다”는 별도의 단계**입니다.

오늘은 자신의 실제 설정값과 실제 결과를 기록합니다.

```md
### Prediction vs Operational Decision

AI Class:

Confidence:

Operational Threshold:

Final State:

Prediction과 운영 판정을 분리해서 확인했는가?

- PASS / FAIL
```

---

# 18. 기능 결과를 코드에서 역추적합니다

이번에는 화면과 LED만 보고 끝내지 않습니다.

다음 파일을 **이전 일차의 파일 그대로 다시 열어** 연결을 확인합니다.

```text
src/camera_input.py
→ Camera Frame

src/ai_preprocess.py
→ Resize / Normalize / Tensor 변환

src/inference_onnx.py
→ Class + Confidence

src/decision.py
→ Operational Decision

src/stabilizer.py
→ Voting / Debounce / Hold

src/gpio_output.py
→ LED / Buzzer

src/event_logger.py
→ Event 기록

src/runtime_state.py
→ 최신 Runtime 상태 저장

scripts/status_api.py
→ HTTP JSON 제공
```

전체 코드 흐름:

```text
CameraInput.read()
        ↓
Preprocess
        ↓
ONNXClassifier.predict()
        ↓
class_name / confidence
        ↓
ai_to_operational_state()
        ↓
raw_state
        ↓
TemporalStabilizer.update()
        ↓
output_state
        ↓
GPIOOutput
        ↓
Event Logger
        ↓
RuntimeStateStore.write()
        ↓
/status
```

다음 질문에 직접 답합니다.

```text
1. Camera Frame을 실제로 읽는 파일은?

2. Class와 Confidence를 만드는 파일은?

3. Confidence Threshold를 이용해 운영 상태를 만드는 파일은?

4. 한 Frame의 WARNING이 바로 최종 출력으로 가지 않도록 하는 파일은?

5. LED / Buzzer를 제어하는 파일은?

6. /status가 직접 ONNX 추론을 수행하는가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
1. src/camera_input.py

2. src/inference_onnx.py

3. src/decision.py

4. src/stabilizer.py

5. src/gpio_output.py

6. 아니다.
   Edge Runtime이 만든 Runtime State를 status_api.py가 읽어서 반환한다.
```

</details>

---

# 19. 현재 Acceptance 실행으로 Operational Event Log 새 Row가 생성되었는지 증명합니다

운영용 Log 파일명을 임의로 추측하지 않습니다. 앞에서 사용한 같은 Terminal이라면 `EVENT_LOG_PATH`를 그대로 사용합니다.

새 Terminal이라면 다시 실제 병합 Config에서 경로를 읽습니다.

```bash
EVENT_LOG_PATH="$(
  python -c \
  'from src.config_loader import load_config; print(load_config("configs/settings.yaml")["ai_event_log_path"])'
)"
```

먼저 현재 Log의 마지막 부분을 확인합니다.

```bash
tail -n 5 "$EVENT_LOG_PATH"
```

10일차 운영 `edge_runtime.py`가 사용하는 `AIEventLogger`의 핵심 열은 다음과 같습니다.

```text
timestamp
pred_class
confidence
threshold
state
image_path
```

여기서 `state`는 한 Frame의 `raw_state`가 아니라 **Voting / Debounce / Hold가 반영된 최종 운영 State**를 기록합니다.

## 19-1. 실행 전·후 Row 수 비교

현재 Row 수를 저장합니다.

```bash
wc -l "$EVENT_LOG_PATH" \
  | tee reports/day13_evidence/event_log_after_rows.txt
```

앞에서 저장한 Before 값과 비교합니다.

```bash
cat reports/day13_evidence/event_log_before_rows.txt
cat reports/day13_evidence/event_log_after_rows.txt
```

숫자만 비교하고 싶다면 다음처럼 확인할 수 있습니다.

```bash
BEFORE_ROWS="$(
  awk '{print $1}' \
  reports/day13_evidence/event_log_before_rows.txt
)"

AFTER_ROWS="$(
  wc -l < "$EVENT_LOG_PATH"
)"

echo "Before rows: $BEFORE_ROWS"
echo "After rows : $AFTER_ROWS"
```

NORMAL → WARNING → NORMAL의 최종 State 변화가 실제로 발생했다면 일반적으로 `AFTER_ROWS`가 `BEFORE_ROWS`보다 커야 합니다.

하지만 단순히 숫자가 커졌다는 사실만으로 끝내지 않습니다.

## 19-2. 새 Timestamp Row 확인

현재 마지막 Row를 Evidence로 저장합니다.

```bash
tail -n 3 "$EVENT_LOG_PATH" \
  | tee reports/day13_evidence/event_log_after_tail.txt
```

다음 세 가지를 확인합니다.

```text
1. Before 이후 새로운 Timestamp Row가 존재하는가?
2. 새 Row의 state가 실제 Acceptance에서 본 최종 State 변화와 맞는가?
3. 파일 경로가 현재 Config의 ai_event_log_path와 같은가?
```

이 세 가지가 맞아야 **“현재 13일차 Acceptance 실행이 Operational Event Log를 실제로 갱신했다”**고 판단합니다.

### 중요한 구분

다음은 PASS 근거가 아닙니다.

```text
과거 day07_ai_events.csv가 존재함
        ↓
현재 Runtime Log도 정상이라고 판단
X
```

다음이 PASS 근거입니다.

```text
현재 Config의 ai_event_log_path 확인
        ↓
Acceptance 전 Row 수 / 마지막 Row 기록
        ↓
NORMAL → WARNING → NORMAL 실제 상태 변화
        ↓
Acceptance 후 Row 수 증가
        ↓
새 Timestamp / state 확인
        ↓
현재 Runtime Event Log PASS
```

## 19-3. 결과를 Report에 기록합니다

```md
### Operational AI Event Log

Event Log File:

Before Rows:

After Rows:

New Row Created:

- PASS / FAIL

New Timestamp Confirmed:

- PASS / FAIL

Logged Final State:

Current `/status` State:

State Meaning Consistent:

- PASS / FAIL

Evidence:

- `reports/day13_evidence/event_log_before_rows.txt`
- `reports/day13_evidence/event_log_before_last.txt`
- `reports/day13_evidence/event_log_after_rows.txt`
- `reports/day13_evidence/event_log_after_tail.txt`
```

### 오류 → 원인 확인 → 수정 → 재실행

#### 오류 A — Log 파일이 없음

```text
오류
→ ai_event_log_path 위치에 파일이 없음

원인 확인
→ 병합 Config의 실제 ai_event_log_path 확인
→ subject13-edge.service Journal 확인
→ 10일차 edge_runtime.py가 AIEventLogger를 생성하는지 확인

수정
→ 현재 운영 Baseline의 Config / Source 연결 복구

재실행
→ Service Restart
→ 실제 State 변화
→ Log 재확인
```

#### 오류 B — 파일은 있지만 Row가 증가하지 않음

```text
오류
→ Before Rows == After Rows

원인 확인
→ 실제 최종 /status state가 변했는가?
→ WARNING Prediction만 바뀌고 output_state는 그대로였는가?
→ Voting / Debounce / Hold 때문에 아직 최종 State가 안 바뀌었는가?
→ edge_runtime.py의 AIEventLogger.append() 조건 확인

수정
→ 실제 NORMAL ↔ WARNING 최종 State 전환을 충분히 유지
→ 필요한 경우 Service / Camera / Decision 원인 복구

재실행
→ Before 기준 다시 기록
→ State 전환
→ After Row / Timestamp 재확인
```

#### 오류 C — 새 Row는 있지만 state 의미가 다름

```text
오류
→ CSV state와 실제 운영 상태 해석이 맞지 않음

원인 확인
→ raw_state와 output_state를 혼동했는가?
→ 현재 edge_runtime.py가 어떤 값을 Logger에 넘기는가?

정상 기준
→ 운영 Event Log는 최종 output_state 변화 기록

수정
→ 10일차 최종 Runtime Source와 현재 배포 Source가 같은지 확인

재실행
→ State 전환
→ CSV와 /status 다시 비교
```

---

# 20. WARNING 이미지 저장 기능은 사용하는 경우만 확인합니다

최종 설정에서 WARNING 이미지 저장 기능을 사용한다면 경로를 확인합니다.

```bash
grep -E \
  "ai_warning_image_dir|warning_image" \
  configs/settings.yaml \
  configs/device.local.yaml \
  2>/dev/null
```

설정에 실제 저장 경로가 있고 기능을 사용 중이라면:

```bash
find <AI_WARNING_IMAGE_DIR> \
  -type f \
  -name "*.jpg" \
  | tail
```

PNG를 사용했다면 실제 확장자를 사용합니다.

기능을 사용하지 않는다면 13일차에 새로 구현하지 않습니다.

```md
### Warning Image

운영 설정에서 사용:

- YES / NO

저장 확인:

- PASS / FAIL / N/A
```

---

# 21. 성능은 긴 Benchmark를 다시 하지 않습니다

8일차부터 이미 Latency와 FPS를 측정했고, 9일차에서는 안정화와 처리 전략을 다뤘으며, 11일차에서는 Trade-off를 검증했습니다.

오늘 다시 여러 조건을 바꿔 장시간 Benchmark를 하면 4시간 Final Acceptance의 목적에서 벗어납니다.

오늘은 두 가지를 사용합니다.

```text
현재 운영 상태
→ /metrics

이전 검증 결과
→ Day 08 / Day 09 / Day 11 Report
```

현재 값:

```bash
curl \
  http://<RPI_IP>:<API_PORT>/metrics
```

이전 Report가 실제로 있는지 확인합니다.

```bash
ls reports/
```

예를 들어 다음 파일을 이전 일차에서 사용했다면 내용을 확인합니다.

```bash
cat reports/day08_performance.md
```

```bash
cat reports/day09_optimization.md
```

```bash
cat reports/day11_test_analysis.md
```

파일명이 실제 프로젝트와 다르면 `ls reports/`에서 확인한 실제 파일명을 사용합니다.

---

# 22. 최종 성능을 한곳에 정리합니다

`reports/day13_acceptance_test.md`의 `Final Performance Summary`를 채웁니다.

```md
## 3. Final Performance Summary

Model:

Model Input:

Camera Resolution:

Confidence Threshold:

Frame Skip:

Vote Window:

Warning Minimum:

Debounce:

Hold:

### Raspberry Pi

Test Accuracy 또는 최종 평가 지표:

Model-only Mean Latency:

Model-only P95:

현재 End-to-End / Loop FPS:

### Final Interpretation

현재 운영 설정을 선택한 이유:

Accuracy / Latency / FPS / Stability 사이에서 고려한 점:

수치의 출처가 되는 Report:
```

`Model`, `Model Input`, `Confidence Threshold`는 기억으로 적지 않습니다. PART A에서 확인한 현재 병합 Config와 Metadata 값을 사용합니다.

### 중요한 규칙

측정하지 않은 숫자는 만들지 않습니다.

```text
직접 측정함
→ 실제 값 기록

이전 Report에 있음
→ Report에서 가져오고 출처 기록

측정하지 않음
→ N/A + 이유
```

12일차의 Jetson `trtexec` Model-only 결과를 Raspberry Pi Camera End-to-End FPS와 같은 범위라고 적지 않습니다.

---

# 23. Functional Acceptance 표를 완성합니다

Report에 추가합니다.

```md
## 2. Functional Acceptance

| 항목 | 확인 방법 | 결과 |
|---|---|---|
| Camera Input | 실제 입력에서 상태 갱신 | PASS / FAIL |
| AI Prediction | `/status`의 pred_class | PASS / FAIL |
| Class / Confidence | `/status` | PASS / FAIL |
| Operational Decision | NORMAL / WARNING | PASS / FAIL |
| Stabilization | 최종 output_state 동작 | PASS / FAIL |
| NORMAL Output | Green LED 또는 정의된 출력 | PASS / FAIL |
| WARNING Output | Red LED / Buzzer | PASS / FAIL / N/A |
| Operational Event Log | 현재 Acceptance 전·후 Row / Timestamp / state 비교 | PASS / FAIL |
| Warning Image | 사용하는 경우 저장 확인 | PASS / FAIL / N/A |
| Runtime State | 최신 JSON 갱신 | PASS / FAIL |
| Performance Metrics | `/metrics` | PASS / FAIL |
```

---

# 24. PART B에서 자주 만나는 오류

## 오류 A — Camera 상태가 갱신되지 않음

먼저 Camera 번호를 추측하지 않습니다.

```text
오류
→ Runtime은 살아 있지만 Camera 입력이 정상적이지 않음

원인 확인
→ Camera 장치가 연결되어 있는가?
→ 어떤 /dev/video* 장치가 있는가?
→ 현재 Camera index가 맞는가?
→ 다른 Process가 Camera를 점유하는가?

수정
→ 실제 장치와 index 확인
→ 점유 문제 해결

재실행
→ Service Restart
→ /health /status 재확인
```

장치 확인 예:

```bash
ls -l /dev/video*
```

이전 일차의 Probe Script가 있다면 다시 사용합니다.

```bash
python -m scripts.camera_index_probe
```

단, systemd Edge Service가 Camera를 이미 점유하고 있다면 수동 Probe를 실행하기 전에 Service 점유 관계를 먼저 확인합니다.

## 오류 B — Prediction은 나오는데 GPIO가 바뀌지 않음

```text
Prediction PASS
+
State PASS
+
GPIO FAIL
        ↓
Model보다 Output 계층을 먼저 확인
```

확인:

```text
src/gpio_output.py
Pin 설정
장치별 Local Config
배선
LED / Buzzer 사용 여부
```

문제를 수정한 뒤 정상/경고 입력을 다시 한 번 확인합니다.

## 오류 C — Operational Event Log 검증 실패

고정 파일명을 추측하지 않습니다.

```text
오류
→ Log가 없거나 현재 Acceptance의 새 Row가 확인되지 않음

원인 확인
→ 병합 Config에서 실제 ai_event_log_path 확인
→ 10일차 운영 edge_runtime.py가 AIEventLogger를 사용 중인지 확인
→ /status의 최종 state가 실제로 NORMAL ↔ WARNING으로 변했는지 확인
→ Voting / Debounce / Hold 때문에 최종 State가 아직 바뀌지 않았는지 확인
→ journalctl에서 Runtime 오류 확인

수정
→ 현재 운영 Baseline Source / Config 복구
→ 실제 최종 State 변화가 일어나도록 입력 유지

재실행
→ Before Row 기준 다시 저장
→ NORMAL → WARNING → NORMAL
→ After Row 수와 새 Timestamp 다시 확인
```

---

# PART C. Failure Recovery · Peer Operation · Final Check — 60분

# 25. 왜 대표 장애를 다시 확인할까요?

10일차와 11일차에서 이미 여러 장애를 실습했습니다.

오늘은 모든 실패를 다시 재현하지 않습니다.

최종 Release 전에 다음 하나만 다시 확인합니다.

```text
Edge Process 비정상 종료
        ↓
systemd가 실패 감지
        ↓
Restart=on-failure
        ↓
Process 자동 재시작
        ↓
Runtime State 다시 갱신
        ↓
/health 정상 복귀
```

이 흐름은 “프로그램이 한 번 실행되는가?”가 아니라 **“운영 중 장애가 나도 다시 살아나는가?”**를 확인하는 시험입니다.

---

# 26. 장애 발생 전 정상 상태를 기록합니다

```bash
systemctl status \
  subject13-edge.service \
  --no-pager
```

현재 API:

```bash
curl \
  http://<RPI_IP>:<API_PORT>/health
```

정상 기준:

```text
Edge Service
→ active

/health
→ edge = OK
→ stale = false
```

이 상태를 확인한 뒤에만 장애 시험을 시작합니다.

---

# 27. Edge Process를 비정상 종료합니다

10일차에서 `Restart=on-failure`를 설정했기 때문에 **의도적인 `systemctl stop`이 아니라 실패 상태**를 만들어야 자동재시작을 확인할 수 있습니다.

실행:

```bash
sudo systemctl kill \
  -s SIGKILL \
  subject13-edge.service
```

잠시 기다립니다.

```bash
sleep 5
```

다시 상태를 확인합니다.

```bash
systemctl status \
  subject13-edge.service \
  --no-pager
```

`Restart=on-failure`가 정상이라면 다시 다음 상태가 되어야 합니다.

```text
active (running)
```

---

# 28. 자동재시작 근거를 Journal에서 확인합니다

```bash
journalctl \
  -u subject13-edge.service \
  -n 80 \
  --no-pager
```

확인할 흐름:

```text
Process 비정상 종료
        ↓
systemd가 실패 감지
        ↓
Restart 예약
        ↓
새 Process 시작
        ↓
Service active
```

정확한 문구는 systemd 버전과 종료 상황에 따라 조금 다를 수 있습니다.

그다음 API가 다시 정상인지 확인합니다.

```bash
curl \
  http://<RPI_IP>:<API_PORT>/health
```

Camera / AI / GPIO도 다시 동작하는지 한 번 확인합니다.

---

# 29. 장애 복구 결과를 Report에 기록합니다

```md
## 4. Recovery Acceptance

Failure:

- Edge Process SIGKILL

Failure Detected:

- PASS / FAIL

journalctl Evidence:

- PASS / FAIL

systemd Auto Restart:

- PASS / FAIL

복구 후 Edge Service:

- PASS / FAIL

복구 후 /health:

- PASS / FAIL

복구 후 Camera / AI / GPIO:

- PASS / FAIL

복구 과정에서 확인한 점:
```

---

# 30. 장애 복구 결과를 코드와 설정에서 역추적합니다

자동재시작은 `edge_runtime.py` 안의 반복문이 스스로 Process를 다시 만든 것이 아닙니다.

다음 구조입니다.

```text
Edge Process SIGKILL
        ↓
Process 종료
        ↓
systemd
        ↓
subject13-edge.service
        ↓
Restart=on-failure
        ↓
RestartSec
        ↓
ExecStart 다시 실행
        ↓
scripts.edge_runtime
```

다음 명령으로 실제 Unit을 확인할 수 있습니다.

```bash
systemctl cat subject13-edge.service
```

다음 항목을 찾습니다.

```text
WorkingDirectory
ExecStart
Restart=on-failure
RestartSec
```

### 확인 문제

```text
1. `systemctl stop`으로 중지한 경우에도 무조건 자동재시작하는가?

2. `SIGKILL` 뒤 다시 시작되는 주체는 Python 코드인가 systemd인가?

3. 다시 시작할 Python 경로는 어디에 정의되어 있는가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
1. 아니다.
   Restart=on-failure는 사람이 의도적으로 stop한 상황과
   비정상 종료를 구분한다.

2. systemd가 Service 실패를 감지하여 다시 시작한다.

3. subject13-edge.service의 ExecStart에 정의되어 있다.
```

</details>

---

# 31. Peer Operation Test — 다른 사람이 README만 보고 운영할 수 있는가?

13일차에는 **운영 인수인계 시험**도 짧게 수행합니다.

가능하면 2명 또는 2개 팀이 서로 장치를 바꾸어 확인합니다.

목적은 상대방의 코드를 수정하는 것이 아닙니다.

다음 문서만 보고 운영합니다.

```text
README.md
```

필요하면 장애 복구 기록을 참고할 수 있습니다.

```text
reports/day10_fault_recovery.md
또는 실제 프로젝트의 10일차 장애 복구 Report
```

---

# 32. 상대 장치에서 수행할 작업

상대방에게 실제 접속 정보를 전달받습니다.

```text
USER
HOSTNAME 또는 IP
API Port
```

비밀번호나 민감한 인증정보를 문서에 공개적으로 기록하지 않습니다.

다음 다섯 항목만 확인합니다.

```text
1. SSH 접속

2. 두 Service 상태 확인

3. /health 확인

4. WARNING 상태 확인

5. 최근 Log 위치 확인
```

예:

```bash
ssh <OTHER_RPI_USER>@<OTHER_RPI_IP>
```

Service:

```bash
systemctl status \
  subject13-edge.service \
  subject13-api.service \
  --no-pager
```

상대 장치의 실제 API Port를 README 또는 Config에서 확인한 뒤:

```bash
curl \
  http://127.0.0.1:<OTHER_API_PORT>/health
```

경고 입력을 보여준 뒤:

```bash
curl \
  http://127.0.0.1:<OTHER_API_PORT>/status
```

README에 안내된 실제 Log 위치를 확인합니다.

---

# 33. Peer Operation에서 하면 안 되는 것

```text
상대 Source Code 수정
X

상대 Config 임의 변경
X

Model 교체
X

Package 설치
X

Service Unit 수정
X
```

목적은 이것입니다.

```text
README만 보고
다른 사람이
시스템의 상태를 확인하고
기본 운영을 수행할 수 있는가?
```

---

# 34. Peer Operation 결과 기록

```md
## 5. Peer Operation Test

확인한 장치:

README만 보고 SSH 접속:

- PASS / FAIL

Service 확인:

- PASS / FAIL

/health 확인:

- PASS / FAIL

WARNING 확인:

- PASS / FAIL

Log 위치 확인:

- PASS / FAIL

문서에서 찾기 어려웠던 설명:

1.

2.

README에 보완할 항목:
```

이 시험에서 FAIL이 나왔다고 프로그램 자체가 반드시 고장난 것은 아닙니다.

다음처럼 구분합니다.

```text
기능은 정상
+
다른 사람이 운영하기 어려움
        ↓
Documentation 문제
```

이 경우 4교시에서 README를 보완합니다.

---

# 35. 최종 점검 Script를 만들기 전에 역할을 이해합니다

13일차에는 큰 기능을 새로 개발하지 않습니다.

하지만 마지막 Release 시점에는 다음을 한 번에 확인하면 편리합니다.

```text
ONNX Model 파일이 존재하는가?

ONNX Metadata가 존재하는가?

현재 Config의 Operational AI Event Log 파일이 존재하는가?

Config가 존재하는가?

README가 존재하는가?

Runtime State가 실제로 들어오는가?

Runtime State가 너무 오래되지 않았는가?
```

이 확인을 위해 작은 Script 하나만 추가합니다.

새 파일:

```text
scripts/final_check.py
```

이 Script는 Camera를 새로 열지 않습니다.

ONNX 추론을 새로 실행하지도 않습니다.

이미 운영 중인 시스템의 **Release 필수 파일과 Runtime State를 확인하는 진단 Script**입니다.

---

# 36. `final_check.py` 의사코드

```text
settings.yaml을 읽는다
        ↓
필수 Config Key가 있는지 확인한다
        ↓
없으면 어떤 Key가 없는지 출력한다
        ↓
ONNX Model 경로 확인
        ↓
ONNX Metadata 경로 확인
        ↓
ai_event_log_path 확인
        ↓
운영 Event Log 파일 존재 확인
        ↓
settings.yaml 확인
        ↓
README.md 확인
        ↓
RuntimeStateStore 준비
        ↓
현재 Runtime State 읽기
        ↓
state_age_sec()로 State 나이 계산
        ↓
status가 RUNNING인가?
        ↓
State가 runtime_stale_sec보다 오래되지 않았는가?
        ↓
현재 State / Prediction / Confidence / Loop FPS 출력
        ↓
모든 필수 검사 PASS
        ↓
FINAL CHECK: PASS
```

---

# 37. `scripts/final_check.py` 작성

```python
from pathlib import Path

from src.config_loader import load_config
from src.runtime_state import (
    RuntimeStateStore,
    state_age_sec,
)


REQUIRED_CONFIG_KEYS = [
    "onnx_model_path",
    "onnx_meta_path",
    "ai_event_log_path",
    "runtime_state_path",
    "runtime_stale_sec",
]


def check_file(
    label: str,
    path: str,
) -> bool:
    exists = Path(path).is_file()

    print(
        f"{label:20s}: "
        f"{'PASS' if exists else 'FAIL'} "
        f"{path}"
    )

    return exists


def main() -> int:
    config = load_config(
        "configs/settings.yaml"
    )

    print(
        "=== Subject 13 Final Check ==="
    )
    print()

    missing_keys = [
        key
        for key in REQUIRED_CONFIG_KEYS
        if key not in config
    ]

    if missing_keys:
        print(
            "Config Keys         : FAIL "
            f"missing={missing_keys}"
        )
        print()
        print("FINAL CHECK: FAIL")
        return 1

    print("Config Keys         : PASS")

    results = []

    results.append(
        check_file(
            "ONNX Model",
            str(config["onnx_model_path"]),
        )
    )

    results.append(
        check_file(
            "ONNX Metadata",
            str(config["onnx_meta_path"]),
        )
    )

    results.append(
        check_file(
            "AI Event Log",
            str(config["ai_event_log_path"]),
        )
    )

    results.append(
        check_file(
            "Config",
            "configs/settings.yaml",
        )
    )

    results.append(
        check_file(
            "README",
            "README.md",
        )
    )

    store = RuntimeStateStore(
        str(config["runtime_state_path"])
    )

    state = store.read()
    age = state_age_sec(state)

    runtime_ok = (
        bool(state)
        and state.get("status") == "RUNNING"
        and age is not None
        and age
        <= float(config["runtime_stale_sec"])
    )

    print(
        f"{'Runtime State':20s}: "
        f"{'PASS' if runtime_ok else 'FAIL'}"
    )

    print(
        f"{'State Age Sec':20s}: "
        f"{age if age is not None else 'N/A'}"
    )

    print(
        f"{'Current State':20s}: "
        f"{state.get('state', 'UNKNOWN')}"
    )

    print(
        f"{'Prediction':20s}: "
        f"{state.get('pred_class', 'UNKNOWN')}"
    )

    print(
        f"{'Confidence':20s}: "
        f"{state.get('confidence', 0.0)}"
    )

    print(
        f"{'Loop FPS':20s}: "
        f"{state.get('loop_fps', 0.0)}"
    )

    results.append(runtime_ok)

    print()

    if all(results):
        print("FINAL CHECK: PASS")
        return 0

    print("FINAL CHECK: FAIL")
    return 1


if __name__ == "__main__":
    raise SystemExit(main())
```

---

# 38. 실행 전에 여러 파일이 어떻게 연결되는지 확인합니다

```text
scripts/final_check.py
        ↓
src/config_loader.py
        ↓
configs/settings.yaml
(+ 장치별 Local Config가 병합되는 구조라면 Local 값)
        ↓
onnx_model_path
onnx_meta_path
ai_event_log_path
runtime_state_path
runtime_stale_sec
        ↓
Path.is_file()
        ↓
Model / Metadata / Operational Event Log / Config / README 존재 확인

그리고

runtime_state_path
        ↓
RuntimeStateStore.read()
        ↓
runtime/edge_state.json
        ↓
state_age_sec()
        ↓
RUNNING + Fresh 여부
        ↓
FINAL CHECK: PASS / FAIL
```

`final_check.py`는 `systemctl`을 대신하지 않습니다.

Service의 `active / enabled`는 이미 PART A에서 별도로 확인했습니다.

이 Script의 범위는 **배포 파일 + Operational Event Log 파일 존재 + Runtime State 진단**입니다.

`AI Event Log : PASS`는 파일 존재를 확인하는 항목입니다. **현재 Acceptance 실행으로 새 Row가 생겼다는 증명은 PART B의 Before/After Row·Timestamp 비교가 담당**합니다.

---

# 39. `final_check.py` 실행

Raspberry Pi 프로젝트 루트에서 가상환경을 활성화합니다.

```bash
cd ~/ai_vision/subject13_edge_ai
```

```bash
source .venv/bin/activate
```

실행:

```bash
python -m scripts.final_check
```

정상 형태의 예:

```text
=== Subject 13 Final Check ===

Config Keys         : PASS
ONNX Model          : PASS <현재 onnx_model_path>
ONNX Metadata       : PASS <현재 onnx_meta_path>
AI Event Log        : PASS <현재 ai_event_log_path>
Config              : PASS configs/settings.yaml
README              : PASS README.md
Runtime State       : PASS
State Age Sec       : <실제 값>
Current State       : NORMAL 또는 WARNING
Prediction          : <실제 Class>
Confidence          : <실제 값>
Loop FPS            : <실제 값>

FINAL CHECK: PASS
```

`Current State`가 NORMAL이 아니어도 현재 Camera 입력이 WARNING 조건이라면 그 자체가 오류는 아닙니다.

핵심은 **현재 입력과 출력이 일치하고 Runtime State가 최신인지**입니다.

---

# 40. Final Check 결과를 코드에서 역추적합니다

실행 결과를 보고 다음 위치를 직접 찾습니다.

| 화면에 보인 결과 | 담당 코드 |
|---|---|
| `Config Keys : PASS` | `REQUIRED_CONFIG_KEYS`와 `missing_keys` |
| `ONNX Model : PASS` | `check_file()` + `onnx_model_path` |
| `ONNX Metadata : PASS` | `check_file()` + `onnx_meta_path` |
| `AI Event Log : PASS` | `check_file()` + `ai_event_log_path` |
| `Runtime State : PASS` | `runtime_ok` 조건 |
| `State Age Sec` | `state_age_sec(state)` |
| `Current State` | `state.get("state")` |
| `Prediction` | `state.get("pred_class")` |
| `Confidence` | `state.get("confidence")` |
| `Loop FPS` | `state.get("loop_fps")` |
| `FINAL CHECK: PASS` | `all(results)` |

다음 질문에 답합니다.

```text
1. ONNX Model 경로를 final_check.py에 고정해서 썼는가?

2. runtime_stale_sec는 어디에서 가져오는가?

3. Runtime State가 비어 있으면 PASS가 되는가?

4. status가 RUNNING이어도 State가 너무 오래되면 PASS가 되는가?

5. `AI Event Log : PASS`만으로 현재 Acceptance가 새 Row를 만들었다고 증명할 수 있는가?

6. final_check.py가 직접 Camera를 열거나 AI 추론을 새로 수행하는가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
1. 아니다.
   configs/settings.yaml에서 onnx_model_path를 읽는다.

2. Config에서 읽는다.

3. 아니다.
   bool(state)가 False이므로 runtime_ok가 False가 된다.

4. 아니다.
   age <= runtime_stale_sec 조건도 만족해야 한다.

5. 아니다.
   이 항목은 현재 Config의 Event Log 파일 존재만 확인한다.
   현재 실행의 새 Row 생성은 PART B의 Before/After Row와 Timestamp로 증명한다.

6. 아니다.
   이미 운영 중인 Runtime State와 배포 파일을 점검한다.
```

</details>

---

# 41. Final Check 오류 처리

## 오류 A — `KeyError` 대신 `Config Keys : FAIL`

예:

```text
Config Keys         : FAIL missing=['onnx_meta_path']
FINAL CHECK: FAIL
```

확인:

```bash
grep -E \
  "onnx_model_path|onnx_meta_path|ai_event_log_path|runtime_state_path|runtime_stale_sec" \
  configs/settings.yaml \
  configs/device.local.yaml \
  2>/dev/null
```

```text
오류
→ 필수 Config Key 없음

원인 확인
→ 설정파일 누락 / 잘못된 Key 이름 / 잘못된 Version

수정
→ 11일차 Tested Baseline과 12일차 Handoff의 실제 Key와 맞춤

재실행
→ python -m scripts.final_check
```

## 오류 B — ONNX Model `FAIL`

```text
오류
→ Config의 Model 경로에 실제 파일이 없음

원인 확인
→ pwd
→ Config 경로 확인
→ ls models/

수정
→ 올바른 배포 Model 복원

재실행
→ final_check.py
```

## 오류 C — Runtime State `FAIL`

```text
오류
→ State가 없거나
→ status가 RUNNING이 아니거나
→ State가 stale 기준보다 오래됨

원인 확인
→ systemctl status subject13-edge.service
→ journalctl
→ cat runtime/edge_state.json

수정
→ Edge Runtime 원인 복구

재실행
→ /health
→ final_check.py
```

---

# 42. Final Check 결과를 Report에 기록합니다

```md
## 6. Final Check

Config Keys:

- PASS / FAIL

ONNX Model:

- PASS / FAIL

ONNX Metadata:

- PASS / FAIL

AI Event Log File:

- PASS / FAIL

Current Acceptance New Event Row:

- PASS / FAIL

Runtime State:

- PASS / FAIL

State Age:

Current State:

Prediction:

Confidence:

Loop FPS:

Final Check:

- PASS / FAIL
```

---

# PART D. Final Release · Course 14 Handoff · 복습 — 60분

# 43. 마지막 교시는 무엇을 하나요?

새 기능 개발을 멈춥니다.

다음 네 가지를 끝냅니다.

```text
README 보완
        ↓
Final Acceptance 판정
        ↓
Git Release / Final Tag
        ↓
Course 14 Handoff
```

13일차의 마지막 Commit이 **교과 13의 종료점이자 교과 14로 넘어가기 위한 기준점**이 됩니다.

---

# 44. README는 왜 마지막에 다시 확인할까요?

Peer Operation Test에서 다른 사람이 막힌 부분이 있었다면 그 내용은 코드 문제가 아니라 문서 문제일 수 있습니다.

README는 1~13일차의 모든 설명을 다시 붙이는 파일이 아닙니다.

최종 운영자가 가장 먼저 볼 **입구 문서**입니다.

다음 내용이 있는지만 확인합니다.

```text
1. 이 프로젝트는 무엇인가?

2. Raspberry Pi에 어떻게 접속하는가?

3. 현재 Hostname / IP는 어떻게 확인하는가?

4. API Port는 어디에서 확인하는가?

5. Edge / API Service 상태는 어떻게 확인하는가?

6. /health /status /metrics는 어떻게 보는가?

7. 현재 운영 Model / Metadata 경로는 어디에서 확인하는가?

8. Operational Event Log 경로와 최근 기록은 어떻게 확인하는가?

9. Service는 어떻게 Restart하는가?

10. 장애가 나면 어떤 순서로 확인하는가?

11. 12일차 Jetson Migration 자료는 어디에 있는가?

12. 교과 14 Handoff 문서는 어디에 있는가?
```

없는 내용만 추가합니다.

---

# 45. README 최종 운영 섹션 예시

아래 내용은 그대로 복사해야 하는 정답이 아니라 **최소 운영정보의 예시**입니다.

실제 프로젝트의 사용자명, 경로, Port를 코드에 고정하지 않습니다.

````md
## Final Operation

### 1. Connect

현재 Raspberry Pi Hostname 또는 IPv4를 확인한 뒤 접속합니다.

```bash
ssh <RPI_USER>@<RPI_IP>
```

### 2. Service Status

```bash
systemctl status \
  subject13-edge.service \
  subject13-api.service \
  --no-pager
```

### 3. API Port

```bash
grep -E "^api_port:" \
  configs/settings.yaml \
  configs/device.local.yaml \
  2>/dev/null
```

### 4. API

```bash
curl http://<RPI_IP>:<API_PORT>/health
curl http://<RPI_IP>:<API_PORT>/status
curl http://<RPI_IP>:<API_PORT>/metrics
```

### 5. Restart

```bash
sudo systemctl restart subject13-edge.service
sudo systemctl restart subject13-api.service
```

### 6. Journal

```bash
journalctl \
  -u subject13-edge.service \
  -n 50 \
  --no-pager
```

### 7. Operational AI Event Log

현재 병합 Config에서 경로를 확인합니다.

```bash
EVENT_LOG_PATH="$(
  python -c   'from src.config_loader import load_config; print(load_config("configs/settings.yaml")["ai_event_log_path"])'
)"

tail -n 20 "$EVENT_LOG_PATH"
```

운영 Log는 모든 Frame이 아니라 첫 운영 State와 이후 최종 `output_state` 변화 기준으로 기록됩니다.

### 8. Runtime State

```bash
cat runtime/edge_state.json
```

### 9. Final Check

```bash
source .venv/bin/activate
python -m scripts.final_check
```

### 10. Troubleshooting Order

```text
Power / Boot
→ Network / SSH
→ systemctl
→ journalctl
→ Runtime State
→ Config / Model / Camera
→ GPIO
→ Operational Event Log
→ API
```

### 11. Raspberry Pi → Jetson Migration

- `reports/day12_rpi_jetson.md`
- `docs/day12_camera_adapter.md`
- `docs/day12_inference_contract.md`
- `docs/day12_migration_map.md`

### 12. Course 14 Handoff

- `docs/course14_handoff.md`
````

---

# 46. 최종 Release에 포함되는 것과 포함되지 않는 것

Source와 문서:

```text
configs/
docs/
reports/
scripts/
src/
systemd/
tests/
README.md
requirements.txt
```

장치에서 별도로 필요한 배포 자산:

```text
models/
→ ONNX Model / Metadata

data/
→ 필요한 Test Sample

logs/
→ 운영 중 생성

runtime/
→ 운영 중 생성
```

일반 Source Commit에 포함하지 않는 대상은 기존 `.gitignore` 정책을 유지합니다.

대표 예:

```text
.venv/
Raw Dataset
기업 제공 데이터
대용량 Runtime Log
Runtime State
Model / Engine Binary
```

특히 12일차의 TensorRT Engine은 대상 Hardware와 Runtime에 영향을 받는 배포 Binary이므로 일반 Source Code처럼 취급하지 않습니다.

---

# 47. 교과 14 Handoff 문서를 만들기 전에 12일차 자료를 다시 봅니다

교과 14 Handoff를 기억에 의존해 새로 만들지 않습니다.

12일차에 이미 다음 연결자료를 만들었습니다.

```text
reports/day12_rpi_jetson.md
→ Raspberry Pi와 Jetson 환경 / Runtime / Benchmark 비교

docs/day12_camera_adapter.md
→ USB Camera에서 Industrial Camera로 바뀔 때의 Camera Contract

docs/day12_inference_contract.md
→ ONNX Runtime에서 TensorRT로 바뀔 때 유지할 Prediction Contract

docs/day12_migration_map.md
→ 무엇을 재사용하고 무엇을 바꿀지 정리한 Migration Map
```

파일이 실제로 있는지 확인합니다.

```bash
ls -lh \
  reports/day12_rpi_jetson.md \
  docs/day12_camera_adapter.md \
  docs/day12_inference_contract.md \
  docs/day12_migration_map.md
```

파일이 없다면 이름을 새로 만들어 내용이 있는 것처럼 꾸미지 않습니다.

12일차에서 사용한 실제 파일명을 확인합니다.

---

# 48. 교과 14에서 무엇이 바뀌고 무엇이 이어질까요?

교과 13의 최종 시스템:

```text
USB Camera
        ↓
Raspberry Pi
        ↓
OpenCV
        ↓
ONNX Runtime CPU
        ↓
Class + Confidence
        ↓
Decision
        ↓
Stabilization
        ↓
GPIO / Log
        ↓
Performance
        ↓
systemd / FastAPI
        ↓
Test / Failure Analysis
```

교과 14의 제조 Vision AI 프로젝트에서는 장비와 AI Task가 바뀝니다.

```text
Industrial Camera
        ↓
Camera Driver / SDK
        ↓
Jetson Orin Nano Super
        ↓
Preprocess
        ↓
Object Detection Model
        ↓
ONNX / TensorRT GPU
        ↓
Inspection Result
        ↓
Log / Statistics
        ↓
Performance / Operation
        ↓
Project Output
```

교과 13의 Tiny CNN을 그대로 제조검사 모델로 사용한다는 뜻이 아닙니다.

재사용하는 것은 **문제를 푸는 구조와 운영 경험**입니다.

---

# 49. 교과 14에서 그대로 가져갈 경험

```text
Linux / SSH

환경값을 Config와 분리하는 방법

Camera Input 계층을 다른 처리와 분리하는 방법

학습된 Model을 배포 Runtime으로 옮기는 관점

Preprocess 일치 확인

Model Prediction과 운영 Decision 분리

Latency / FPS 측정

시간적 안정화가 필요한 이유

Log / Evidence 남기기

systemd 운영 개념

FastAPI 상태 제공 개념

Unit / Integration / System Test 구분

Failure Analysis

Acceptance Test

Git / README / Release 문서화
```

---

# 50. 교과 14에서 주로 바뀌는 부분

```text
장치
Raspberry Pi
→ Jetson Orin Nano Super

Camera
USB Camera
→ Industrial Camera

Camera 연결
OpenCV VideoCapture 중심
→ Vendor Driver / SDK 가능

AI Task
Tiny CNN Classification
→ Manufacturing Object Detection

Inference Runtime
ONNX Runtime CPU
→ ONNX / TensorRT GPU

출력
LED / Buzzer Warning
→ Inspection Result / Log / Statistics / 프로젝트 요구기능
```

12일차에서 만든 Contract가 중요한 이유가 여기에 있습니다.

```text
Camera 내부 구현이 바뀌어도
→ 이후 Pipeline이 기대하는 Frame 형식을 맞춘다.

Inference Runtime이 바뀌어도
→ 이후 Decision이 기대하는 Prediction 형식을 맞춘다.
```

---

# 51. 교과 14는 Guided Lab에서 Student-led Project로 바뀝니다

교과 13에서는 대부분의 실습 흐름이 제공되었습니다.

```text
무엇을 확인할지 안내
        ↓
파일 역할 설명
        ↓
의사코드
        ↓
코드 작성
        ↓
실행
        ↓
결과 해석
        ↓
오류 수정
```

교과 14에서는 학생이 더 많은 판단을 직접 해야 합니다.

```text
프로젝트 요구사항 확인
        ↓
문제 정의
        ↓
데이터 확인 / 수집 / 라벨링
        ↓
Baseline 결정
        ↓
Model 학습 / 평가
        ↓
Jetson 배포 구조 결정
        ↓
Camera 연결
        ↓
검사 결과 / Log / Statistics 설계
        ↓
Latency / FPS / AI 성능 검증
        ↓
Failure 분석
        ↓
최종 시연 / 문서 / Portfolio
```

교과 13에서 배운 질문을 그대로 가져갑니다.

```text
입력은 정상인가?

Model과 Data가 맞는가?

Preprocess가 학습 때와 같은가?

Prediction은 어떻게 운영 결과가 되는가?

Latency와 FPS는 충분한가?

실패 원인은 어느 계층인가?

장치는 장시간 운영 가능한가?

결과를 Log와 문서로 증명할 수 있는가?
```

---

# 52. `docs/course14_handoff.md`를 만듭니다

## 이 파일은 왜 필요할까요?

13일차가 끝난 뒤 바로 교과 14 프로젝트가 시작되면, 다음과 같은 일이 생기기 쉽습니다.

```text
“Jetson이니까 모든 코드를 새로 만들어야 하나?”

“Camera SDK가 바뀌면 AI 코드도 전부 바꿔야 하나?”

“TensorRT가 빨랐으니 그 FPS를 전체 시스템 FPS라고 써도 되나?”

“교과 13에서 만든 Test와 Log는 버려도 되나?”
```

이 혼란을 줄이기 위해 12일차 Migration 결과와 13일차 Acceptance 결과를 한 장으로 연결합니다.

새 파일:

```text
docs/course14_handoff.md
```

다음 형식으로 작성합니다.

````md
# Course 13 → Course 14 Handoff

## 1. Course 13 Final Accepted Structure

USB Camera
→ Raspberry Pi
→ Preprocess
→ ONNX Runtime CPU
→ Class + Confidence
→ Decision
→ Stabilization
→ GPIO / Log
→ Performance
→ systemd
→ FastAPI
→ Test / Failure Analysis

Final Acceptance Result:

- ACCEPT / CONDITIONAL ACCEPT / NOT ACCEPT

## 2. Day 12 Jetson Migration Evidence

Environment / Benchmark:

- `reports/day12_rpi_jetson.md`

Camera Contract:

- `docs/day12_camera_adapter.md`

Inference Contract:

- `docs/day12_inference_contract.md`

Migration Map:

- `docs/day12_migration_map.md`

## 3. Course 14에서 바뀌는 핵심

Raspberry Pi
→ Jetson Orin Nano Super

USB Camera
→ Industrial Camera

Tiny CNN Classification
→ Manufacturing Object Detection

ONNX Runtime CPU
→ ONNX / TensorRT GPU

GPIO Warning
→ Inspection Result / Log / Statistics / Project Output

## 4. 그대로 재사용할 경험

- Linux / SSH
- Config 분리
- Camera Input 계층 분리
- Model Deployment 관점
- Preprocess 일치 확인
- Prediction과 Operational Decision 분리
- Latency / FPS 측정
- Failure Analysis
- Logging / Evidence
- systemd 운영 개념
- FastAPI 상태 제공 개념
- Unit / Integration / System Test 관점
- Acceptance Test
- Git / README / Release

## 5. Course 14 첫 번째 Technical Gate

1. Jetson OS / JetPack 상태 확인
2. Industrial Camera Driver / SDK 확인
3. 실제 Camera Frame 1장 획득
4. 데이터와 Label 구조 확인
5. Detection Baseline Model 확인
6. ONNX / TensorRT Runtime 경로 확인
7. 성능 측정 범위 정의
8. Log / Result 구조 정의

## 6. Day 12에서 확인했지만 과장하면 안 되는 것

- TensorRT `trtexec` Model-only Benchmark는 Camera 포함 End-to-End FPS가 아니다.
- TensorRT Engine Build 성공만으로 실제 Image → Class/Detection 결과가 검증된 것은 아니다.
- Raspberry Pi와 Jetson의 Runtime·Precision·측정도구가 다르면 단순 속도배수로 결론내리지 않는다.

## 7. Course 14 Project에서 우리가 먼저 결정할 것

Project Problem:

Input Data:

Camera:

Detection Classes:

Model Baseline:

Jetson Runtime:

Performance Metric:

Log / Statistics:

Team Role:

First PASS Gate:
````

이 문서는 교과 14 프로젝트 전체 WBS를 대신하지 않습니다.

**교과 13의 검증된 경험을 교과 14의 첫 기술 의사결정으로 넘기는 Handoff 문서**입니다.

---

# 53. Handoff 문서를 12일차와 연결해서 코드 리뷰합니다

새 문서를 작성한 뒤 다음을 직접 대조합니다.

```text
course14_handoff.md의 Camera 변경
        ↓
day12_camera_adapter.md에서 확인

course14_handoff.md의 Inference Runtime 변경
        ↓
day12_inference_contract.md에서 확인

course14_handoff.md의 Raspberry Pi ↔ Jetson 비교
        ↓
day12_rpi_jetson.md에서 확인

course14_handoff.md의 REUSE / CHANGE
        ↓
day12_migration_map.md에서 확인
```

즉, Handoff 문서는 새 아이디어를 갑자기 추가하는 문서가 아닙니다.

**12일차에서 실제로 확인한 Migration 결과를 교과 14 시작점으로 요약**합니다.

---

# 54. Final Acceptance를 판정합니다

`reports/day13_acceptance_test.md`의 마지막 표를 완성합니다.

```md
## 7. Final Acceptance

| 영역 | 항목 | 결과 |
|---|---|---|
| Boot | 재부팅 후 자동실행 | PASS / FAIL |
| Service | Edge Service active / enabled | PASS / FAIL |
| Service | API Service active / enabled | PASS / FAIL |
| Input | Camera | PASS / FAIL |
| Deployment | 현재 ONNX / Metadata = 11일차 Tested Baseline | PASS / FAIL |
| AI | Prediction | PASS / FAIL |
| Decision | NORMAL / WARNING | PASS / FAIL |
| Stability | Voting / Debounce / Hold 결과 | PASS / FAIL |
| Output | LED / Buzzer | PASS / FAIL / N/A |
| Log | 현재 Acceptance에서 Operational Event Log 새 Row / Timestamp / state 확인 | PASS / FAIL |
| Runtime | Runtime State Fresh | PASS / FAIL |
| Performance | 기존 결과 + 현재 Metrics 확인 | PASS / FAIL |
| Recovery | 비정상 종료 후 자동재시작 | PASS / FAIL |
| API | /health /status /metrics | PASS / FAIL |
| Documentation | Peer Operation | PASS / FAIL |
| Final Check | scripts/final_check.py | PASS / FAIL |
| Handoff | Course 14 연결 설명 | PASS / FAIL |

## Final Result

- ACCEPT
- CONDITIONAL ACCEPT
- NOT ACCEPT

## 남아 있는 문제

1.

2.

## 교과 14로 넘어갈 때 주의할 점

1.

2.
```

---

# 55. ACCEPT / CONDITIONAL ACCEPT / NOT ACCEPT의 의미

## ACCEPT

```text
최종 필수 기능 정상
+
운영 / Recovery 정상
+
문서화 가능
+
다른 사람이 기본 운영 가능
```

## CONDITIONAL ACCEPT

핵심 Pipeline은 동작하지만 해결해야 할 문제가 남아 있습니다.

예:

```text
WARNING 이미지 저장 기능은 사용하지 않음

특정 Network에서 외부 API 접근 제한

선택 Buzzer 미사용

교과 14에서 다시 확인할 Jetson Runtime 조건 존재
```

문제를 숨기지 않고 조건을 적습니다.

## NOT ACCEPT

다음처럼 핵심 운영경로가 정상적으로 검증되지 않은 경우입니다.

```text
재부팅 후 Edge Runtime이 시작되지 않음

Camera / AI Pipeline이 동작하지 않음

WARNING Decision이 동작하지 않음

Runtime State / API가 운영 상태를 반영하지 못함

복구가 되지 않고 원인도 정리되지 않음
```

Final Result를 좋게 보이게 하기 위해 FAIL을 임의로 PASS로 바꾸지 않습니다.

Acceptance의 목적은 점수를 예쁘게 만드는 것이 아니라 **현재 시스템 상태를 정확히 남기는 것**입니다.

---

# 56. README와 Handoff를 최종 확인합니다

다음 파일이 존재하는지 확인합니다.

```bash
ls -lh \
  README.md \
  reports/day13_acceptance_test.md \
  docs/course14_handoff.md \
  scripts/final_check.py
```

12일차 Handoff 자료도 확인합니다.

```bash
ls -lh \
  reports/day12_rpi_jetson.md \
  docs/day12_camera_adapter.md \
  docs/day12_inference_contract.md \
  docs/day12_migration_map.md
```

파일명이 실제 프로젝트와 다르면 실제 이름을 확인합니다.

---

# 57. Git에 올리기 전에 무엇이 바뀌었는지 확인합니다

```bash
git status
```

```bash
git diff
```

오늘의 대표 변경:

```text
scripts/final_check.py

docs/course14_handoff.md

reports/day13_acceptance_test.md

reports/day13_evidence/
→ 현재 Acceptance Event Log 증거

README.md 보완
```

Runtime Log나 장치별 Local Config가 의도치 않게 Stage되지 않는지 확인합니다.

---

# 58. Final Release Commit 만들기

필요한 파일만 Stage합니다.

```bash
git add \
  scripts/final_check.py \
  docs/course14_handoff.md \
  reports/day13_acceptance_test.md \
  reports/day13_evidence \
  README.md
```

상태를 다시 확인합니다.

```bash
git status
```

Commit:

```bash
git commit -m \
  "release: finalize subject13 edge ai"
```

---

# 59. Final Tag 만들기

현재 Commit을 확인합니다.

```bash
git log \
  --oneline \
  -5
```

기존에 같은 Tag가 있는지 먼저 확인합니다.

```bash
git tag --list subject13-vfinal
```

아무것도 출력되지 않고 현재 Commit이 최종 Release가 맞다면:

```bash
git tag subject13-vfinal
```

확인:

```bash
git show \
  --stat \
  subject13-vfinal
```

이 Tag의 의미:

```text
subject13-vfinal
→ 교과 13 Final Acceptance와 Release 문서화가 끝난 기준점
```

이미 같은 Tag가 있다면 내용을 확인하지 않고 강제로 덮어쓰지 않습니다.

---

# 60. Remote 저장소가 있는 경우에만 Push합니다

기관에서 허용된 내부 GitLab / Gitea 등이 연결되어 있는지 확인합니다.

```bash
git remote -v
```

허용된 Remote가 있다면:

```bash
git push
```

Tag도 공유해야 한다면:

```bash
git push \
  origin \
  subject13-vfinal
```

외부 Git 사용이 제한된 환경에서는 Local Git 또는 기관이 허용한 내부 저장소를 사용합니다.

Dataset, 기업 제공 데이터, Runtime Log, ONNX/TensorRT Binary의 외부 업로드는 기관 정책을 따릅니다.

---

# 61. 최종 Git 상태 확인

```bash
git status
```

정상적으로 모든 필요한 내용을 Commit했다면 다음과 비슷하게 보입니다.

```text
nothing to commit, working tree clean
```

필요한 결과파일이 `.gitignore` 대상이라면 working tree에 보이지 않는 것이 정상일 수 있습니다.

중요한 것은 **왜 Git에 포함하거나 제외했는지 설명할 수 있는 것**입니다.

---

# 62. 교과 13 최종 시스템을 한 번에 다시 봅니다

```text
[입력]
USB Camera
        ↓

[처리]
OpenCV / Preprocess
        ↓

[AI]
ONNX Runtime CPU
        ↓
Class + Confidence
        ↓

[판단]
Operational Decision
        ↓
Voting / Debounce / Hold
        ↓
NORMAL / WARNING
        ↓

[출력]
LED / Buzzer
        ↓

[기록]
Operational AI Event Log
Runtime State
        ↓

[운영]
systemd
Auto Restart
        ↓

[외부 상태]
FastAPI
/health /status /metrics
        ↓
PC

[검증]
Latency / FPS
Unit / Integration / System Test
Failure Analysis
Acceptance Test
```

---

# 63. 1~13일차 시스템 성장 최종 확인

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
→ Raspberry Pi
→ AI Inference

8일차
Latency / FPS
→ 성능 측정

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
→ FastAPI

11일차
Unit / Integration / System Test
→ Failure Analysis
→ Trade-off

12일차
Raspberry Pi Tested Baseline
→ Jetson
→ ONNX / TensorRT
→ Migration Contract

13일차
Reboot
→ Final Acceptance
→ Recovery
→ Peer Operation
→ Release
→ Course 14 Handoff
```

---

# 64. 핵심 복습 문제

문서를 보지 않고 먼저 답합니다.

## 문제 1

13일차 Final Acceptance를 재부팅부터 시작하는 이유는 무엇인가요?

## 문제 2

재부팅 직후 왜 `python -m scripts.edge_runtime`을 직접 실행하면 안 되나요?

## 문제 3

`subject13-edge.service`와 `subject13-api.service`의 역할 차이는 무엇인가요?

## 문제 4

`/health`에서 `api=OK`, `edge=DEGRADED`가 동시에 나올 수 있는 이유는 무엇인가요?

## 문제 5

`/status`의 `pred_class`와 최종 `state`는 왜 같은 개념이 아닌가요?

## 문제 6

왜 과거 `logs/day07_ai_events.csv`가 존재한다는 사실만으로 13일차 Operational Event Log를 PASS 처리하면 안 되나요?

## 문제 7

13일차에서 긴 Benchmark를 다시 하지 않는 이유는 무엇인가요?

## 문제 8

`systemctl kill -s SIGKILL` 이후 Service가 다시 시작되는 주체는 무엇인가요?

## 문제 9

Peer Operation Test에서 Source Code를 수정하지 않는 이유는 무엇인가요?

## 문제 10

`final_check.py`의 `Runtime State : PASS`가 되기 위한 핵심 조건은 무엇인가요?

## 문제 11

12일차에서 만든 `day12_camera_adapter.md`와 `day12_inference_contract.md`는 교과 14에서 왜 중요할까요?

## 문제 12

교과 13에서 교과 14로 넘어갈 때 주로 바뀌는 네 가지는 무엇인가요?

## 문제 13

교과 14에서 그대로 재사용할 수 있는 교과 13의 경험을 네 가지 이상 말해 보세요.

## 문제 14

`ACCEPT`, `CONDITIONAL ACCEPT`, `NOT ACCEPT` 중 문제가 남아 있는데도 숨기지 않고 조건을 명시하는 상태는 무엇인가요?

---

## 예시 정답 확인해보기

<details>
<summary><strong>복습 문제 예시 정답 열기</strong></summary>

### 정답 1

운영형 Edge 장치는 사람이 매번 Python을 실행하지 않아도 Boot 후 systemd를 통해 자동으로 동작해야 하기 때문입니다.

### 정답 2

직접 Python을 실행하면 systemd가 자동으로 실행한 것인지 사람이 실행한 것인지 구분할 수 없기 때문입니다.

### 정답 3

```text
subject13-edge.service
→ Camera / AI / Decision / GPIO / Runtime State를 수행하는 Edge Runtime

subject13-api.service
→ 이미 만들어진 Runtime State를 읽어서 HTTP JSON으로 제공하는 API
```

### 정답 4

FastAPI Process 자체는 살아 있지만 Edge Runtime State가 오래되었거나 `RUNNING` 상태가 아니면 API는 응답하면서도 Edge 상태는 `DEGRADED`일 수 있습니다.

### 정답 5

`pred_class`는 AI Model의 예측이고, 최종 `state`는 Class·Confidence·Operational Threshold와 안정화 Logic을 반영한 운영 결과이기 때문입니다.

### 정답 6

현재 운영 Runtime이 사용하는 Log는 병합 Config의 `ai_event_log_path`가 기준이며, 과거 파일 존재만으로는 **현재 13일차 Acceptance가 새 Event를 기록했다는 사실을 증명할 수 없기 때문**입니다. Acceptance 전·후 Row 수와 새 Timestamp를 함께 확인해야 합니다.

### 정답 7

8~11일차에서 이미 성능 측정과 Trade-off 검증을 수행했기 때문입니다. 13일차는 새로운 최적화 실험이 아니라 Final Acceptance와 Release가 목적입니다.

### 정답 8

Python 코드 자체가 아니라 `systemd`가 `Restart=on-failure` 정책에 따라 Process를 다시 시작합니다.

### 정답 9

Peer Operation의 목적은 다른 사람의 코드를 고치는 것이 아니라 **README만으로 기존 시스템을 운영할 수 있는지** 확인하는 것이기 때문입니다.

### 정답 10

대표적으로 다음을 만족해야 합니다.

```text
Runtime State가 존재
+
status == RUNNING
+
state_age_sec를 계산할 수 있음
+
State 나이가 runtime_stale_sec 이하
```

### 정답 11

교과 14에서 Camera Driver와 Inference Runtime이 바뀌더라도 이후 Pipeline이 기대하는 입력·출력 형식을 유지하면 기존 Logic을 더 많이 재사용할 수 있기 때문입니다.

### 정답 12

대표적으로 다음이 바뀝니다.

```text
Raspberry Pi → Jetson Orin Nano Super
USB Camera → Industrial Camera
Tiny CNN Classification → Object Detection
ONNX Runtime CPU → ONNX / TensorRT GPU
```

### 정답 13

예:

```text
Linux / SSH
Config 분리
Camera 계층 분리
Model Deployment 관점
Preprocess 일치
Decision 분리
Latency / FPS 측정
Failure Analysis
Logging
systemd
FastAPI
Test / Acceptance
Git / README
```

### 정답 14

`CONDITIONAL ACCEPT`입니다.

핵심 기능은 동작하지만 남은 문제와 조건을 명확하게 기록하는 상태입니다.

</details>

---

# 65. 오늘 반드시 남아 있어야 하는 결과

```text
1. reports/day13_acceptance_test.md
   → Boot / Functional / Performance / Recovery / Peer / Final Acceptance

2. reports/day13_evidence/event_log_*
   → 현재 Acceptance 실행 전·후 Operational Event Log 증거

3. scripts/final_check.py
   → 현재 Model / Metadata / Operational Event Log 파일과 Runtime State 점검

4. docs/course14_handoff.md
   → Course 13 → Course 14 연결

5. README.md 최종 운영 섹션

6. Final Release Commit

7. subject13-vfinal Tag

8. 12일차 Migration 문서와의 연결

9. ACCEPT / CONDITIONAL ACCEPT / NOT ACCEPT 최종 판정
```

---

# 66. 자가 체크리스트

수업을 마치기 전에 직접 체크합니다.

- [ ] 12일차 Handoff Gate에서 시작했다.
- [ ] Raspberry Pi를 실제로 재부팅했다.
- [ ] 재부팅 후 `python -m ...`을 직접 실행하지 않고 Service 상태를 확인했다.
- [ ] `subject13-edge.service`가 `active`인지 확인했다.
- [ ] `subject13-edge.service`가 `enabled`인지 확인했다.
- [ ] `subject13-api.service`가 `active`인지 확인했다.
- [ ] `subject13-api.service`가 `enabled`인지 확인했다.
- [ ] 실제 API Port를 Config에서 확인했다.
- [ ] `/health`를 확인했다.
- [ ] `/status`를 확인했다.
- [ ] `/metrics`를 확인했다.
- [ ] Runtime State가 최신인지 확인했다.
- [ ] NORMAL 입력과 출력의 연결을 확인했다.
- [ ] WARNING 입력과 출력의 연결을 확인했다.
- [ ] Prediction과 Operational Decision의 차이를 설명할 수 있다.
- [ ] 병합 Config의 `ai_event_log_path`에서 실제 Operational Event Log 경로를 확인했다.
- [ ] Functional Acceptance 전에 Event Log Row 수와 마지막 Row를 Evidence로 저장했다.
- [ ] NORMAL → WARNING → NORMAL 최종 State 변화를 확인했다.
- [ ] Functional Acceptance 후 Event Log에 새로운 Timestamp Row가 추가된 것을 확인했다.
- [ ] 새 Event CSV의 `state`가 실제 최종 운영 State 의미와 일치하는지 확인했다.
- [ ] 현재 `onnx_model_path` / `onnx_meta_path`가 11일차 Tested Baseline과 일치하는지 확인했다.
- [ ] 현재 운영 성능과 이전 성능 Report를 연결해 기록했다.
- [ ] 측정하지 않은 숫자를 임의로 만들지 않았다.
- [ ] Edge Process 비정상 종료 후 자동재시작을 확인했다.
- [ ] `journalctl`에서 Recovery 근거를 확인했다.
- [ ] 다른 학습자가 README만으로 기본 운영을 수행해 보았다.
- [ ] Peer Operation에서 상대 Source를 수정하지 않았다.
- [ ] `scripts/final_check.py`를 작성하고 실행했다.
- [ ] Final Check 출력을 실제 코드 위치로 역추적했다.
- [ ] `reports/day13_acceptance_test.md`를 완성했다.
- [ ] Final Result를 ACCEPT / CONDITIONAL ACCEPT / NOT ACCEPT 중 하나로 기록했다.
- [ ] README에 최종 운영정보를 보완했다.
- [ ] 12일차 Jetson Migration 자료를 다시 확인했다.
- [ ] `docs/course14_handoff.md`를 작성했다.
- [ ] 교과 13에서 재사용할 부분과 교과 14에서 바뀔 부분을 설명할 수 있다.
- [ ] Git Final Release Commit을 만들었다.
- [ ] `subject13-vfinal` Tag를 확인했다.
- [ ] 기관 정책상 허용되는 저장소만 사용했다.
- [ ] Dataset / Model / Engine / Runtime Log의 관리정책을 지켰다.

---

# 67. 교과 13을 마치며

교과 13에서 중요한 것은 Raspberry Pi 명령어를 많이 외운 것이 아닙니다.

13일 동안 반복해서 배운 구조는 다음입니다.

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
        ↓
성능 측정
        ↓
안정화
        ↓
운영
        ↓
Test
        ↓
Failure Analysis
        ↓
Acceptance
        ↓
Release
```

교과 14에서는 더 이상 동일한 Guided Lab을 그대로 따라가는 것이 중심이 아닙니다.

학생이 팀과 함께 직접 다음을 결정해야 합니다.

```text
어떤 불량 / 이물을 찾을 것인가?

어떤 데이터를 사용할 것인가?

Label은 올바른가?

어떤 Detection Model을 Baseline으로 사용할 것인가?

Jetson에서 어떤 Runtime으로 실행할 것인가?

Industrial Camera Frame은 어떻게 받을 것인가?

무엇을 성능지표로 볼 것인가?

어떤 Log와 통계를 남길 것인가?

어떤 실패가 발생했고 어떻게 고칠 것인가?

최종 결과를 어떻게 시연하고 Portfolio로 설명할 것인가?
```

장비와 AI Task는 바뀌지만 문제를 해결하는 기본 질문은 그대로입니다.

```text
입력은 정상인가?

데이터와 Model은 맞는가?

Model은 올바르게 배포되었는가?

Runtime은 현재 Hardware와 맞는가?

Prediction을 어떻게 실제 검사 결과로 바꿀 것인가?

Latency와 FPS는 충분한가?

장시간 운영해도 안정적인가?

문제가 생기면 어느 계층부터 확인할 것인가?

결과를 Log와 Report로 증명할 수 있는가?
```

이 질문들을 가지고 교과 14의 Jetson 기반 제조 Vision AI 프로젝트로 넘어갑니다.

---

# 오늘의 한 문장 정리

> **13일차는 1~12일차에 만든 Raspberry Pi Edge AI를 재부팅부터 기능·로그·성능·장애복구·운영문서까지 최종 Acceptance하고 Release한 뒤, 12일차의 Jetson Migration 경험을 교과 14의 학생 주도 제조 Vision AI 프로젝트 시작점으로 넘기는 날입니다.**
