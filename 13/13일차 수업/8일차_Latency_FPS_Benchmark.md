
> **오늘의 핵심:** 7일차에는 PC에서 학습한 Tiny CNN을 ONNX로 변환하고, 같은 전처리와 Class 의미를 유지한 채 Raspberry Pi에서 실제 `Camera → AI → Decision → GPIO → Log` Pipeline까지 연결했습니다.
>
> 8일차에는 새 AI 모델을 만드는 것이 아니라, **이미 정상 동작하는 7일차 Pipeline이 실제 Raspberry Pi에서 얼마나 빠르고 얼마나 일정하게 동작하는지 측정하는 방법**을 배웁니다.
>
> 1~7일차에 사용한 `subject13_edge_ai` 프로젝트와 Raspberry Pi의 기존 `.venv`를 그대로 이어서 사용합니다. **새 프로젝트와 새 가상환경을 만들지 않습니다.**
>
> 오늘의 성능 숫자는 정답을 외우기 위한 값이 아닙니다. 장치·Camera·Runtime·입력 크기·Log 여부가 바뀌면 결과도 달라질 수 있으므로, **측정 조건을 고정하고 자신의 실제 결과를 기록하는 것**이 더 중요합니다.

---

## 오늘 가장 중요한 질문

7일차까지는 다음 질문에 답했습니다.

```text
ONNX Model이 Raspberry Pi에서 실행되는가?
        ↓
Camera Frame을 받을 수 있는가?
        ↓
Class + Confidence가 나오는가?
        ↓
NORMAL / WARNING으로 판단되는가?
        ↓
GPIO / Log까지 연결되는가?
```

오늘부터 질문이 달라집니다.

```text
실행은 된다.
        ↓
그런데 Model 자체는 몇 ms가 걸리는가?
        ↓
Camera까지 포함하면 몇 ms가 걸리는가?
        ↓
평균만 빠른가, 대부분의 Frame도 안정적으로 빠른가?
        ↓
어느 단계가 가장 느린가?
        ↓
조건 하나를 바꾸면 속도와 Accuracy는 어떻게 달라지는가?
```

따라서 오늘 가장 중요한 질문은 다음입니다.

> **“같은 조건에서 반복 측정한 근거를 이용하여 Raspberry Pi Edge AI의 Model-only 성능과 실제 End-to-End 성능을 구분하고, 현재 병목을 설명할 수 있는가?”**

---

## 7일차와 8일차는 어디가 이어질까요?

7일차의 최종 Pipeline은 다음과 같습니다.

```text
USB Camera
        ↓
BGR → RGB
Resize / Normalize
HWC → CHW → NCHW
        ↓
ONNX Runtime
        ↓
Class + Confidence
        ↓
Operational Decision
        ↓
NORMAL / WARNING
        ↓
GPIO
        ↓
Event Log / WARNING Image
```

8일차에서는 이 기능을 새로 만들지 않습니다. 다음 파일과 설정을 **그대로 재사용**합니다.

```text
models/day07_tiny_cnn.onnx
models/day07_tiny_cnn.meta.json

src/ai_preprocess.py
src/inference_onnx.py
src/camera_input.py
src/decision.py
src/gpio_output.py
src/config_loader.py

configs/settings.yaml
configs/device.local.yaml
```

오늘 새로 추가되는 핵심은 다음입니다.

```text
Latency 통계 계산
        ↓
Warm-up
        ↓
반복 Benchmark
        ↓
Mean / P50 / P95
        ↓
Model-only 측정
        ↓
End-to-End 측정
        ↓
단계별 Profile
        ↓
한 변수씩 비교 실험
        ↓
속도 + Accuracy 함께 해석
```

---

## 오늘의 학습 목표

수업이 끝나면 다음을 할 수 있어야 합니다.

```text
1. Latency와 FPS의 차이를 자신의 말로 설명한다.
2. 한 번 측정한 값보다 반복 측정이 필요한 이유를 설명한다.
3. Warm-up이 무엇이며 왜 본 측정과 구분하는지 설명한다.
4. Mean / P50 / P95가 각각 무엇을 보여주는지 설명한다.
5. Model-only와 End-to-End 측정 범위를 구분한다.
6. Raspberry Pi에서 ONNX Model-only Latency를 반복 측정한다.
7. PC와 Raspberry Pi를 같은 조건으로 비교한다.
8. 실제 Camera Pipeline의 End-to-End Latency와 FPS를 측정한다.
9. Camera / Preprocess / Inference / Decision 단계의 시간을 분리해 본다.
10. CPU / Memory / Temperature를 측정 조건과 함께 기록한다.
11. Log 또는 GPIO 포함 여부를 한 변수씩 바꾸어 비교한다.
12. 입력 크기 96과 64의 속도 차이와 Accuracy 차이를 함께 본다.
13. Camera Resolution과 Model Input Size가 서로 다른 값임을 설명한다.
14. 실행 결과가 어느 파일의 어느 코드에서 만들어졌는지 역추적한다.
15. Benchmark 결과 CSV로 자신의 측정이 실제 수행되었음을 증명한다.
16. 9일차에서 최적화할 병목 후보와 현재 Baseline을 남긴다.
```

---

## 오늘의 8시간 학습 흐름

| 학습 블록 | 권장 시간 | 무엇을 배우나요? | 끝나면 할 수 있어야 하는 것 |
|---|---:|---|---|
| 1 | 45분 | 7일차 Handoff · 환경 확인 · Benchmark 조건 | 무엇을 그대로 쓰고 무엇을 측정할지 설명할 수 있다 |
| 2 | 55분 | Latency · FPS · Warm-up · Mean/P50/P95 | 성능 숫자의 의미를 구분할 수 있다 |
| 3 | 70분 | Model-only Benchmark | ONNX 추론 자체의 반복 Latency를 측정할 수 있다 |
| 4 | 80분 | End-to-End Benchmark | Camera부터 Decision/선택 출력까지 실제 Pipeline 시간을 측정할 수 있다 |
| 5 | 55분 | Stage Profile · Resource 상태 | 병목 후보와 장치 상태를 찾을 수 있다 |
| 6 | 60분 | One Variable A/B · Input Size/Accuracy | 한 번에 한 변수만 바꾸어 비교할 수 있다 |
| 7 | 75분 | Mini Challenge | 결과 역추적·예측·설정 변경·코드 수정·오류 복구를 스스로 수행할 수 있다 |
| 8 | 40분 | Baseline 복원 · Report · README · Git · 복습 | 9일차가 사용할 측정 Baseline을 남길 수 있다 |

총 480분을 기준으로 구성합니다. 실제 Camera 연결이나 Raspberry Pi 상태에 따라 각 블록의 시간은 조금 달라질 수 있습니다.

---

## 오늘도 계속 기억할 다섯 단계

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

8일차에서는 여기에 **측정**이라는 관점이 추가됩니다.

```text
입력
Camera
  ↓
처리
Preprocess + ONNX
  ↓
판단
Class + Confidence → NORMAL / WARNING
  ↓
출력
선택 GPIO
  ↓
기록
선택 Benchmark Log
  ↓
측정
Latency / FPS / P50 / P95 / Stage Profile
```

코드를 보다가 헷갈리면 두 가지를 질문합니다.

> **“이 코드는 입력·처리·판단·출력·기록 중 어느 역할인가?”**  
> **“지금 재고 있는 시간에는 어디부터 어디까지가 포함되는가?”**

---

## 오늘 사용할 실제 장비와 데이터

```text
PC
→ 7일차와 같은 ONNX Model 비교 실행
→ 96 / 64 Variant Export와 Accuracy 평가

Raspberry Pi
→ Raspberry Pi OS / Linux
→ ONNX Runtime CPUExecutionProvider
→ USB Camera
→ 선택 Green / Red LED

공통 Model
→ models/day07_tiny_cnn.onnx
→ models/day07_tiny_cnn.meta.json

공통 고정 이미지
→ data/deploy_samples/normal.jpg

입력 크기 비교용 Validation Dataset
→ data/dataset/val/normal/
→ data/dataset/val/warning_red/
```

`normal.jpg`는 Model-only 성능을 PC와 Raspberry Pi에서 같은 입력 조건으로 비교하기 위한 고정 Sample입니다. 입력 크기 96/64 같은 **배포 조건 후보 비교에는 Validation Dataset**을 사용합니다. 6일차에서 Test는 최종 평가용으로 이미 사용했으므로, 8일차의 반복적인 조건 선택에 Test를 다시 사용하지 않습니다.

---

## 오늘 배울 기술 스택

| 기술 | 오늘 어디에 사용하나요? |
|---|---|
| Python `time.perf_counter()` | 짧은 실행 구간의 경과시간 측정 |
| NumPy | Mean·Percentile 계산과 배열 처리 |
| ONNX Runtime | Model-only 및 End-to-End 추론 |
| OpenCV | Raspberry Pi Camera Frame 입력 |
| CSV | 반복 Latency와 Summary 결과 저장 |
| PyYAML | Benchmark 설정값 관리 |
| PyTorch / ONNX | 96·64 입력 Variant Export에 사용 |
| Linux 명령 | CPU·Memory·Load·Temperature 상태 확인 |
| SSH / SCP 또는 내부 Git | PC↔Raspberry Pi Source/Model 동기화 |
| Git | Benchmark Source·Report·Summary Version 관리 |

---

## 실습 전에 자신의 환경값 적어두기

환경마다 달라질 수 있는 값은 교재의 예시를 그대로 외우지 않습니다.

```text
PC Project Path             : ______________________________
PC Hostname                 : ______________________________
PC Architecture             : ______________________________
PC ONNX Runtime Version     : ______________________________

Raspberry Pi 사용자 이름    : ______________________________
Raspberry Pi Hostname       : ______________________________
Raspberry Pi IPv4           : ______________________________
Raspberry Pi Architecture   : ______________________________
Raspberry Pi Python         : ______________________________
Raspberry Pi ORT Version    : ______________________________

Camera Index                : ______________________________
Camera Resolution           : ______________________________
Model Input Size            : ______________________________
Execution Provider          : ______________________________
Cooling / Power Condition   : ______________________________
```

교재에서는 다음 표기법을 사용합니다.

```text
<RPI_USER>
→ 자신의 Raspberry Pi 사용자 이름

<RPI_IP>
→ 현재 확인한 Raspberry Pi IPv4

<PC_PROJECT_PATH>
→ PC의 subject13_edge_ai 실제 경로
```

Raspberry Pi IP가 바뀌었다면 이전 값을 계속 사용하지 않습니다. Raspberry Pi에서 직접 확인할 수 있다면 다음 명령을 사용합니다.

```bash
hostname
hostname -I
```

Camera index는 3일차와 7일차에서 사용한 `scripts.camera_index_probe`로 다시 확인할 수 있습니다.

---

## 오늘의 전체 실습 흐름

```text
7일차 ONNX Camera Pipeline 확인
        ↓
측정 조건 기록
        ↓
Benchmark Config 추가
        ↓
Latency / FPS / Warm-up 이해
        ↓
performance.py
통계 함수 작성
        ↓
Model-only Benchmark
        ↓
Raspberry Pi 결과 저장
        ↓
PC에서 같은 조건 측정
        ↓
PC vs Pi 비교
        ↓
End-to-End Benchmark
Camera → Preprocess → ONNX → Decision → 선택 GPIO/Log
        ↓
Stage Profile
        ↓
Resource 상태 기록
        ↓
Log ON/OFF 비교
        ↓
GPIO ON/OFF 비교
        ↓
Input 96 vs 64
속도 + Accuracy 비교
        ↓
Camera Resolution 비교
        ↓
Warm-up / Repeat 조건 실험
        ↓
Mini Challenge
        ↓
Day 08 Baseline 복원
        ↓
Report / README / Git
        ↓
9일차 최적화 준비
```

---

## 오늘의 성공 기준

8일차가 끝났을 때 다음 연결이 실제로 성립하면 됩니다.

```text
7일차 ONNX Baseline 정상
        ↓
Model-only 반복 측정 성공
        ↓
Mean / P50 / P95 저장
        ↓
PC와 Pi 같은 조건 비교
        ↓
End-to-End 반복 측정 성공
        ↓
Camera / Preprocess / Inference / Decision Profile
        ↓
CPU / Memory / Temperature 기록
        ↓
한 변수 A/B 비교
        ↓
Input Size 속도 + Accuracy 비교
        ↓
병목 후보 설명
        ↓
reports/day08_performance.md 완성
        ↓
9일차용 Baseline 복원
```

---

## 오늘은 아직 하지 않는 것

```text
Frame Skip / Voting / Debounce / Hold 최적화
→ 9일차

장시간 운영 안정화와 Recovery 정리
→ 9일차 이후

systemd 자동실행 / 자동재시작
→ 10일차

FastAPI 운영 상태 제공
→ 10일차
```

오늘은 **“어떻게 고칠까?”보다 “어디가 얼마나 느린가?”를 먼저 측정하는 날**입니다.

---

# PART A. 7일차 Baseline을 다시 확인하고 측정 환경을 준비하기

# 1. 7일차 결과에서 이어서 시작하기

7일차가 끝난 뒤 Raspberry Pi에는 최소 다음 파일이 있어야 합니다.

```text
models/
├── day07_tiny_cnn.onnx
└── day07_tiny_cnn.meta.json
```

그리고 다음 프로그램이 실행되어야 합니다.

```bash
python -m scripts.predict_onnx \
  --image data/deploy_samples/normal.jpg
```

```bash
python -m scripts.onnx_camera_probe
```

```bash
python -m scripts.ai_warning_system
```

오늘은 이 ONNX 모델을 그대로 사용합니다.

---

# 2. Raspberry Pi에 접속하고 7일차 Baseline을 확인하기

8일차는 새 프로젝트를 만드는 날이 아닙니다. 먼저 **7일차 최종 상태가 그대로 살아 있는지** 확인합니다.

PC에서 Raspberry Pi에 접속합니다.

```bash
ssh <RPI_USER>@<RPI_IP>
```

`<RPI_USER>`와 `<RPI_IP>`는 자신의 실제 값으로 바꾸어 사용합니다.

접속이 되지 않으면 바로 OS를 다시 설치하지 않습니다.

```text
전원 / Boot 확인
        ↓
Wi-Fi 연결 확인
        ↓
Raspberry Pi IP 재확인
        ↓
ping 또는 Network 도달 확인
        ↓
SSH 재시도
```

Raspberry Pi에서 현재 Hostname과 IP를 확인할 수 있다면:

```bash
hostname
hostname -I
```

프로젝트로 이동합니다.

```bash
cd ~/ai_vision/subject13_edge_ai
```

현재 Git 상태를 확인합니다.

```bash
git status
git log --oneline -8
```

### 내부 GitLab / Gitea 같은 허용된 Remote가 있는 경우

Remote가 있는지 먼저 확인합니다.

```bash
git remote -v
```

정상 Remote가 연결되어 있고 수업 정책상 허용되어 있다면:

```bash
git pull
```

### Local Git만 사용하는 경우

Remote가 없다면 `git pull`을 억지로 실행하지 않습니다. PC에서 수정한 8일차 Source는 수업에서 정한 SCP 또는 파일 전달 방식으로 Raspberry Pi에 전달합니다.

### Raspberry Pi 기존 `.venv` 활성화

```bash
source .venv/bin/activate
```

현재 Python을 확인합니다.

```bash
which python
python --version
```

장치 상태:

```bash
python -m scripts.check_device
```

ONNX Runtime:

```bash
python -m scripts.inspect_onnx_runtime
```

Camera:

```bash
python -m scripts.camera_index_probe
```

7일차 저장 이미지 추론도 한 번 확인합니다.

```bash
python -m scripts.predict_onnx \
  --image data/deploy_samples/normal.jpg
```

실시간 Pipeline도 짧게 확인합니다.

```bash
python -m scripts.onnx_camera_probe
```

필요하다면 `Ctrl+C`로 종료합니다.

```text
ONNX Model 있음
+ Metadata 있음
+ 저장 이미지 추론 정상
+ Camera 정상
+ ONNX Runtime 정상
        ↓
8일차 Benchmark 시작 가능
```

> 7일차 기능 자체가 실패하는 상태에서 성능 숫자를 재면 “느린 것”과 “고장난 것”을 구분할 수 없습니다. 먼저 기능 Baseline을 정상 상태로 만든 뒤 측정합니다.

---

# PART B. 성능 측정의 기준과 통계값 이해하기

# 3. 성능 측정 전에 조건부터 고정하기

성능 숫자는 **어떤 조건에서 측정했는지**가 함께 있어야 의미가 있습니다.

예를 들어 다음 두 결과를 단순 비교하면 안 됩니다.

```text
A
96 × 96
Camera 없음
Log 없음
100회 반복

B
640 × 480 Camera
96 × 96 전처리
GPIO 있음
Log 있음
20회 반복
```

둘은 측정 범위가 다릅니다.

따라서 오늘부터 성능 결과에는 다음 항목을 함께 기록합니다.

```text
Device
OS / Architecture
Model File
Runtime
Execution Provider
Model Input Size
Camera Resolution
Batch
Warm-up Count
Repeat Count
Camera 포함 여부
Preprocess 포함 여부
Decision 포함 여부
GPIO 포함 여부
Log 포함 여부
```

---

# 4. 8일차 Benchmark 설정 추가하기

`configs/settings.yaml`에 다음 항목을 추가합니다.

```yaml
benchmark_warmup: 20
benchmark_repeat: 100

benchmark_result_dir: reports/day08

benchmark_include_gpio: false
benchmark_include_log: true

benchmark_log_path: logs/day08_e2e_events.csv
```

각 값의 의미:

```text
benchmark_warmup
→ 본 측정 전에 버리는 실행 횟수

benchmark_repeat
→ 실제 통계에 사용할 반복 횟수

benchmark_result_dir
→ 성능 결과 CSV 저장 위치

benchmark_include_gpio
→ End-to-End 측정에 GPIO를 포함할지 여부

benchmark_include_log
→ End-to-End 측정에 Log를 포함할지 여부
```

GPIO를 포함하면 LED·부저가 반복 동작할 수 있으므로 기본값은 `false`로 둡니다.

필요하면 별도 실험에서 `true`로 바꿉니다.

---

# 5. 왜 Warm-up을 하나요?

프로그램을 처음 실행한 직후에는 다음 작업이 섞일 수 있습니다.

```text
ONNX Runtime Session 초기화
Memory 준비
CPU Cache
Library 초기 작업
첫 실행 Overhead
```

첫 번째 추론 시간이:

```text
20 ms
```

인데 이후에는:

```text
6 ms
6 ms
7 ms
6 ms
```

처럼 나올 수 있습니다.

첫 실행 하나를 평균에 그대로 넣으면 평소 실행속도를 잘못 해석할 수 있습니다.

그래서 오늘은 다음처럼 측정합니다.

```text
Warm-up 20회
→ 결과 버림

그 다음 100회
→ 실제 Latency 저장
```

---

# 6. Mean·P50·P95를 계산하는 공통 모듈 만들기

## 이 파일은 왜 필요할까요?

앞으로 여러 Benchmark 프로그램에서 똑같이 `Mean`, `P50`, `P95`, `Min`, `Max`를 계산합니다. 계산 코드를 각 Script에 복사하면 한 파일만 수정되거나 계산 방식이 달라질 수 있습니다.

그래서 성능 통계를 계산하는 기능을 `src/performance.py` 하나로 모읍니다.

```text
여러 Latency 값
        ↓
performance.py
        ↓
Count / Mean / P50 / P95 / Min / Max
        ↓
Mean Latency 기반 이론적 처리량(FPS*)
```

`FPS*`는 **Model-only에서 실제 Camera FPS가 아니라 Mean Latency의 역수로 계산한 이론적 처리량**이라는 점을 계속 기억합니다.

## 의사코드

```text
Latency 값 목록을 받는다
        ↓
목록이 비어 있으면 오류를 발생시킨다
        ↓
float64 NumPy 배열로 바꾼다
        ↓
Mean 계산
        ↓
50 Percentile 계산 → P50
        ↓
95 Percentile 계산 → P95
        ↓
Min / Max 계산
        ↓
Mean이 0보다 크면
1000 / Mean(ms)로 이론적 처리량 계산
        ↓
모든 값을 LatencyStats로 묶어 반환
```

파일:

```text
src/performance.py
```

코드:

```python
from dataclasses import dataclass

import numpy as np


@dataclass
class LatencyStats:
    count: int
    mean_ms: float
    p50_ms: float
    p95_ms: float
    min_ms: float
    max_ms: float
    fps_from_mean: float


def summarize_latency(
    values_ms: list[float],
) -> LatencyStats:
    if not values_ms:
        raise ValueError(
            "Latency 값이 없습니다."
        )

    values = np.asarray(
        values_ms,
        dtype=np.float64,
    )

    mean_ms = float(
        np.mean(values)
    )

    p50_ms = float(
        np.percentile(values, 50)
    )

    p95_ms = float(
        np.percentile(values, 95)
    )

    min_ms = float(
        np.min(values)
    )

    max_ms = float(
        np.max(values)
    )

    fps = (
        1000.0 / mean_ms
        if mean_ms > 0
        else 0.0
    )

    return LatencyStats(
        count=len(values_ms),
        mean_ms=mean_ms,
        p50_ms=p50_ms,
        p95_ms=p95_ms,
        min_ms=min_ms,
        max_ms=max_ms,
        fps_from_mean=fps,
    )
```


### Guided Lab — 통계 함수부터 작은 숫자로 확인하기

실제 Camera Benchmark 전에 함수 하나만 독립적으로 확인합니다.

```bash
python - <<'PY'
from src.performance import summarize_latency

values = [5, 5, 6, 6, 7, 8, 10, 30]
stats = summarize_latency(values)

print("Count:", stats.count)
print("Mean :", round(stats.mean_ms, 3))
print("P50  :", round(stats.p50_ms, 3))
print("P95  :", round(stats.p95_ms, 3))
print("Min  :", round(stats.min_ms, 3))
print("Max  :", round(stats.max_ms, 3))
PY
```

예상되는 핵심은 숫자를 외우는 것이 아니라 다음입니다.

```text
Count
→ 입력한 Latency 개수

Mean
→ 전체 평균

P50
→ 중앙 수준

P95
→ 대부분의 실행이 들어오는 상위 지연 수준
```

이 작은 테스트가 정상이라면 이후 Benchmark Script에서 같은 통계 함수를 재사용합니다.

---

# 7. Mean·P50·P95를 숫자로 먼저 이해하기

예를 들어 Latency가 다음과 같다고 가정합니다.

```text
5
5
6
6
6
7
7
8
10
30 ms
```

대부분은 5~10 ms인데 한 번 30 ms가 발생했습니다.

## Mean

전체를 평균냅니다.

```text
평균적인 처리시간
```

## P50

전체 측정값의 중간 수준입니다.

```text
절반의 요청은
P50 이하 시간 안에 처리
```

## P95

측정값의 95%가 이 값 이하에 들어옵니다.

```text
대부분의 실행이
어느 정도 안에서 끝나는가?
```

Edge 시스템에서는 평균만 빠르고 가끔 매우 느린 Frame이 발생할 수도 있으므로 P95도 함께 확인합니다.

---

# PART C. Model-only Latency를 정확히 측정하기

# 8. Model-only 측정 범위 먼저 고정하기

오늘 Model-only는 다음 범위만 측정합니다.

```text
이미 전처리된
NCHW float32 Tensor
        ↓
ONNX Runtime session.run()
        ↓
Logits
```

포함하지 않는 것:

```text
Camera 읽기
Resize
RGB 변환
Decision
GPIO
CSV
화면 출력
```

따라서 Model-only Latency는 **ONNX Runtime 추론 자체의 처리시간에 가까운 값**입니다.

---

# 9. ONNX Session을 직접 호출하는 Benchmark 만들기

## 이 파일은 왜 필요할까요?

7일차의 `predict_onnx.py`는 이미지 한 장의 **Class와 Confidence가 맞는지** 확인하는 프로그램입니다. 8일차의 목적은 다릅니다.

`benchmark_model_only.py`는 이미 전처리된 Tensor를 준비한 뒤 `session.run()` 구간만 반복 측정하여 **ONNX 추론 자체의 Latency에 가까운 값**을 얻습니다.

```text
고정 이미지 1장
        ↓
전처리 1회
        ↓
NCHW Tensor 준비 완료
        ↓
Warm-up
        ↓
session.run()만 반복 측정
        ↓
Mean / P50 / P95
        ↓
Detail CSV + Summary CSV
```

## 의사코드

```text
--image / --model / --meta / --label 인자를 받는다
        ↓
settings.yaml을 읽는다
        ↓
Model과 Metadata 경로를 결정한다
        ↓
Metadata에서 image_size를 읽는다
        ↓
고정 이미지를 NCHW Tensor로 한 번 전처리한다
        ↓
ONNX Runtime Session을 만든다
        ↓
Warm-up 횟수만큼 session.run() 실행
→ 시간은 저장하지 않는다
        ↓
Repeat 횟수만큼
    시작 시간 기록
    session.run()
    종료 시간 기록
    Latency 저장
        ↓
performance.py로 통계 계산
        ↓
터미널 출력
        ↓
Detail CSV 저장
        ↓
Summary CSV 저장
```

파일:

```text
scripts/benchmark_model_only.py
```

코드:

```python
import argparse
import csv
import json
import platform
import time
from pathlib import Path

import numpy as np
import onnxruntime as ort

from src.ai_preprocess import (
    image_path_to_nchw,
)
from src.config_loader import load_config
from src.performance import (
    summarize_latency,
)


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--image",
        required=True,
    )

    parser.add_argument(
        "--model",
        default=None,
    )

    parser.add_argument(
        "--meta",
        default=None,
    )

    parser.add_argument(
        "--label",
        default="default",
    )

    return parser.parse_args()


def main():
    args = parse_args()

    config = load_config(
        "configs/settings.yaml"
    )

    model_path = (
        args.model
        or config["onnx_model_path"]
    )

    meta_path = (
        args.meta
        or config["onnx_meta_path"]
    )

    metadata = json.loads(
        Path(meta_path).read_text(
            encoding="utf-8"
        )
    )

    image_size = int(
        metadata["image_size"]
    )

    input_tensor = image_path_to_nchw(
        args.image,
        image_size,
    )

    session = ort.InferenceSession(
        model_path,
        providers=[
            "CPUExecutionProvider"
        ],
    )

    input_name = (
        session.get_inputs()[0].name
    )

    output_name = (
        session.get_outputs()[0].name
    )

    warmup = int(
        config["benchmark_warmup"]
    )

    repeat = int(
        config["benchmark_repeat"]
    )

    print("=== Model-only Benchmark ===")
    print(f"Label      : {args.label}")
    print(f"Device     : {platform.node()}")
    print(
        f"Architecture: "
        f"{platform.machine()}"
    )
    print(f"Model      : {model_path}")
    print(f"Input      : {input_tensor.shape}")
    print(f"Warm-up    : {warmup}")
    print(f"Repeat     : {repeat}")
    print()

    for _ in range(warmup):
        session.run(
            [output_name],
            {
                input_name:
                    input_tensor
            },
        )

    latencies = []

    for _ in range(repeat):
        started = time.perf_counter()

        session.run(
            [output_name],
            {
                input_name:
                    input_tensor
            },
        )

        elapsed_ms = (
            time.perf_counter()
            - started
        ) * 1000.0

        latencies.append(
            elapsed_ms
        )

    stats = summarize_latency(
        latencies
    )

    print("=== Result ===")
    print(f"Count : {stats.count}")
    print(
        f"Mean  : "
        f"{stats.mean_ms:.3f} ms"
    )
    print(
        f"P50   : "
        f"{stats.p50_ms:.3f} ms"
    )
    print(
        f"P95   : "
        f"{stats.p95_ms:.3f} ms"
    )
    print(
        f"Min   : "
        f"{stats.min_ms:.3f} ms"
    )
    print(
        f"Max   : "
        f"{stats.max_ms:.3f} ms"
    )
    print(
        f"FPS*  : "
        f"{stats.fps_from_mean:.2f}"
    )

    result_dir = Path(
        config["benchmark_result_dir"]
    )

    result_dir.mkdir(
        parents=True,
        exist_ok=True,
    )

    detail_path = (
        result_dir
        / (
            f"model_only_"
            f"{args.label}_detail.csv"
        )
    )

    with detail_path.open(
        "w",
        newline="",
        encoding="utf-8",
    ) as file:
        writer = csv.writer(file)

        writer.writerow(
            [
                "iteration",
                "latency_ms",
            ]
        )

        for index, latency in enumerate(
            latencies,
            start=1,
        ):
            writer.writerow(
                [
                    index,
                    f"{latency:.6f}",
                ]
            )

    summary_path = (
        result_dir
        / (
            f"model_only_"
            f"{args.label}_summary.csv"
        )
    )

    with summary_path.open(
        "w",
        newline="",
        encoding="utf-8",
    ) as file:
        writer = csv.writer(file)

        writer.writerow(
            [
                "label",
                "device",
                "architecture",
                "model",
                "input_size",
                "warmup",
                "repeat",
                "mean_ms",
                "p50_ms",
                "p95_ms",
                "min_ms",
                "max_ms",
                "fps_from_mean",
            ]
        )

        writer.writerow(
            [
                args.label,
                platform.node(),
                platform.machine(),
                model_path,
                image_size,
                warmup,
                repeat,
                f"{stats.mean_ms:.6f}",
                f"{stats.p50_ms:.6f}",
                f"{stats.p95_ms:.6f}",
                f"{stats.min_ms:.6f}",
                f"{stats.max_ms:.6f}",
                f"{stats.fps_from_mean:.6f}",
            ]
        )

    print()
    print(f"Detail  : {detail_path}")
    print(f"Summary : {summary_path}")
    print()
    print(
        "* FPS는 Mean Model-only Latency의 "
        "역수로 계산한 이론적 처리량입니다."
    )


if __name__ == "__main__":
    main()
```

---

# 10. Raspberry Pi Model-only Benchmark 실행하기

같은 고정 이미지로 측정합니다.

```bash
python -m scripts.benchmark_model_only \
  --image data/deploy_samples/normal.jpg \
  --label pi_96
```

출력 예시는 장비마다 다릅니다.

```text
Mean
P50
P95
Min
Max
FPS
```

자신의 실제 값을 사용합니다.


## 코드 리뷰 — `Mean`, `P95`, `FPS*`는 어디에서 만들어졌을까요?

실행 결과만 보고 넘어가지 않습니다. `scripts/benchmark_model_only.py`와 `src/performance.py`를 열어 다음 흐름을 직접 찾습니다.

```text
benchmark_warmup / benchmark_repeat
→ configs/settings.yaml

session.run()
→ scripts/benchmark_model_only.py

elapsed_ms
→ time.perf_counter() 차이

Mean / P50 / P95
→ src/performance.py의 summarize_latency()

model_only_pi_96_detail.csv
→ 반복별 원시 Latency

model_only_pi_96_summary.csv
→ 통계 Summary
```

다음 질문에 답합니다.

```text
1. 이미지 전처리는 반복 측정 안에 들어 있는가?
2. 실제로 시간을 재는 시작점은 어느 줄인가?
3. Warm-up 결과는 latencies 목록에 들어가는가?
4. FPS*는 어느 값으로부터 계산되는가?
5. Detail CSV와 Summary CSV의 역할은 어떻게 다른가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
1. 아니다. Model-only에서는 입력 Tensor를 먼저 한 번 준비한다.
2. started = time.perf_counter() 직후부터 session.run() 종료까지이다.
3. 들어가지 않는다. Warm-up은 결과를 버린다.
4. Mean Latency(ms)의 역수인 1000 / mean_ms이다.
5. Detail은 반복별 Latency, Summary는 Mean/P50/P95 같은 요약값이다.
```

</details>

---

# 11. 한 번만 실행한 값과 100회 결과를 비교하기

다음처럼 생각하지 않습니다.

```text
한 번 7ms
→ 이 모델은 항상 7ms
```

오늘은 100회 결과를 저장했습니다.

예:

```text
Mean 7.2 ms
P50  6.9 ms
P95  9.8 ms
Max  15.4 ms
```

이렇게 보면:

```text
보통은 약 7ms 수준이지만
일부 실행은 더 느릴 수 있다.
```

는 것을 알 수 있습니다.

---

# 12. PC에서도 같은 Model-only 조건으로 측정하기

Raspberry Pi에서 생성한 Source를 Commit하기 전에 PC에도 같은 `performance.py`, `benchmark_model_only.py`가 있어야 합니다.

Source를 Git에 Push한 뒤 PC에서 Pull하거나, 아직 실습 중이라면 먼저 PC와 Pi의 Source를 동기화합니다.

PC 프로젝트:

```bash
cd ~/ai_vision/subject13_edge_ai
```

가상환경:

```bash
source .venv/bin/activate
```

같은 ONNX 모델:

```bash
ls -lh models/day07_tiny_cnn.onnx
```

같은 이미지:

```bash
ls -lh data/deploy_samples/normal.jpg
```

실행:

```bash
python -m scripts.benchmark_model_only \
  --image data/deploy_samples/normal.jpg \
  --label pc_96
```

---

# 13. PC와 Raspberry Pi 비교에서 반드시 같은 조건 사용하기

다음이 같아야 합니다.

```text
ONNX Model
Input Image
Input Size
Batch
Warm-up
Repeat
Execution Provider 조건
```

장치만 다릅니다.

```text
PC
vs
Raspberry Pi
```

이렇게 해야 장치 차이를 해석할 수 있습니다.

---

# 14. PC vs Raspberry Pi Model-only 비교표 만들기

파일:

```text
reports/day08_performance.md
```

다음부터 작성합니다.

```md
# Day 08 Performance

## Model-only Measurement Condition

Model:

Input Image:

Input Size:

Batch: 1

Warm-up:

Repeat:

Runtime:

Provider:

## PC vs Raspberry Pi

| Device | Mean Latency | P50 | P95 | Min | Max | FPS* |
|---|---:|---:|---:|---:|---:|---:|
| PC |  |  |  |  |  |  |
| Raspberry Pi |  |  |  |  |  |  |

\* Model-only Mean Latency 기반 이론적 처리량
```

---

# 15. Model-only FPS와 실제 Camera FPS를 같은 것으로 보지 않기

Model-only에서 계산한:

```text
1000 / Mean Latency
```

는 Model이 연속해서 처리할 수 있는 이론적인 처리량에 가깝습니다.

하지만 실제 시스템은 다음 시간이 더 필요합니다.

```text
Camera
Preprocess
Decision
GPIO
Log
```

따라서:

```text
Model-only FPS
≠
실제 End-to-End FPS
```

입니다.

이 차이를 지금부터 직접 측정합니다.

---

# PART D. 실제 Camera Pipeline의 End-to-End 성능 측정하기

# 16. End-to-End 측정 범위 정하기

오늘 End-to-End 기본 범위는 다음입니다.

```text
Camera Frame read
        ↓
BGR → RGB
        ↓
Resize / Normalize / NCHW
        ↓
ONNX Runtime
        ↓
Class + Confidence
        ↓
Operational Decision
        ↓
선택적으로 GPIO
        ↓
선택적으로 CSV Log
```

측정에서 제외:

```text
터미널에 매 Frame print
```

`print()` 자체가 성능에 영향을 줄 수 있기 때문입니다.

중간 진행상황만 가끔 출력합니다.

---

# 17. End-to-End 성능 Log 모듈 만들기

## 이 파일은 왜 필요할까요?

End-to-End Benchmark에서는 `Log OFF`와 `Log ON`을 비교합니다. 따라서 Benchmark 전용 Logger가 필요합니다.

다만 CSV의 **파일 생성과 Header 작성 시간**까지 첫 번째 측정 Frame에 우연히 섞이면 비교가 불공정해질 수 있습니다. 그래서 이 Logger는 객체를 만들 때 파일과 Header를 먼저 준비하고, 실제 측정 구간에서는 한 줄의 결과만 추가합니다.

또한 서로 다른 실험 결과가 섞이지 않도록 각 `--label`마다 별도 Log 파일을 사용합니다.

## 의사코드

```text
저장할 CSV 경로를 받는다
        ↓
부모 폴더가 없으면 만든다
        ↓
reset=True이면 기존 같은 이름 파일을 지운다
        ↓
새 파일을 만들고 Header를 먼저 쓴다
        ↓
Benchmark 반복 중 append() 호출
        ↓
frame / state / confidence를 한 줄씩 추가한다
```

파일:

```text
src/benchmark_logger.py
```

코드:

```python
import csv
from pathlib import Path


class BenchmarkCSVLogger:
    def __init__(
        self,
        path: str,
        reset: bool = True,
    ):
        self.path = Path(path)

        self.path.parent.mkdir(
            parents=True,
            exist_ok=True,
        )

        if reset and self.path.exists():
            self.path.unlink()

        with self.path.open(
            "w",
            newline="",
            encoding="utf-8",
        ) as file:
            writer = csv.writer(file)
            writer.writerow(
                [
                    "frame",
                    "state",
                    "confidence",
                ]
            )

    def append(
        self,
        frame_index: int,
        state: str,
        confidence: float,
    ) -> None:
        with self.path.open(
            "a",
            newline="",
            encoding="utf-8",
        ) as file:
            writer = csv.writer(file)
            writer.writerow(
                [
                    frame_index,
                    state,
                    f"{confidence:.6f}",
                ]
            )
```

이 Logger는 Benchmark의 Log Overhead를 보기 위한 작은 기록 모듈입니다. 7일차의 운영 Event Logger와 목적이 다릅니다.

---

# 18. End-to-End Benchmark 프로그램 만들기

## 이 파일은 왜 필요할까요?

Model-only는 `session.run()`만 측정했습니다. 하지만 실제 Edge AI는 Camera를 읽고, 전처리하고, AI 추론하고, 운영 상태를 판단하며 필요하면 GPIO와 Log까지 처리합니다.

`benchmark_e2e.py`는 이 **실제 처리 경로 전체**를 같은 기준으로 반복 측정합니다.

또한 `--label`을 받아 실험마다 결과 파일을 따로 저장합니다. `log_off`, `log_on`, `gpio_on`, `camera_720p`처럼 Label을 다르게 쓰면 앞의 결과를 덮어쓰지 않습니다.

## 실행 전에 파일 연결 확인하기

```text
configs/settings.yaml
configs/device.local.yaml
        ↓
src/config_loader.py
        ↓
scripts/benchmark_e2e.py
        │
        ├─ src/camera_input.py
        ├─ src/ai_preprocess.py
        ├─ src/inference_onnx.py
        ├─ src/decision.py
        ├─ src/gpio_output.py      ← 선택
        ├─ src/benchmark_logger.py ← 선택
        └─ src/performance.py
        ↓
reports/day08/e2e_<label>_detail.csv
reports/day08/e2e_<label>_summary.csv
```

## 의사코드

```text
--label을 받는다
        ↓
Config를 읽는다
        ↓
Warm-up / Repeat / GPIO / Log 설정을 읽는다
        ↓
Camera와 ONNX Classifier를 준비한다
        ↓
GPIO가 켜져 있으면 GPIOOutput을 준비한다
        ↓
Log가 켜져 있으면 Label 전용 Benchmark CSV를 미리 만든다
        ↓
Camera + Preprocess + ONNX Warm-up
→ 결과 시간은 버린다
        ↓
Repeat 횟수만큼
    시작 시간 기록
    Camera Frame 읽기
    Preprocess
    ONNX Prediction
    Operational Decision
    선택 GPIO
    선택 Log
    종료 시간 기록
    Latency 저장
        ↓
Mean / P50 / P95 / FPS 계산
        ↓
Label 전용 Detail / Summary CSV 저장
        ↓
Camera / GPIO Resource 정리
```

파일:

```text
scripts/benchmark_e2e.py
```

코드:

```python
import argparse
import csv
import platform
import time
from pathlib import Path

from src.ai_preprocess import (
    bgr_frame_to_nchw,
)
from src.benchmark_logger import (
    BenchmarkCSVLogger,
)
from src.camera_input import CameraInput
from src.config_loader import load_config
from src.decision import (
    ai_to_operational_state,
)
from src.gpio_output import GPIOOutput
from src.inference_onnx import (
    ONNXClassifier,
)
from src.performance import (
    summarize_latency,
)


def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--label",
        default="default",
    )
    return parser.parse_args()


def safe_label(value: str) -> str:
    cleaned = "".join(
        ch if ch.isalnum() or ch in "-_" else "_"
        for ch in value
    ).strip("_")

    return cleaned or "default"


def main():
    args = parse_args()
    label = safe_label(args.label)

    config = load_config(
        "configs/settings.yaml"
    )

    warmup = int(
        config["benchmark_warmup"]
    )
    repeat = int(
        config["benchmark_repeat"]
    )

    if warmup < 0:
        raise ValueError(
            "benchmark_warmup은 0 이상이어야 합니다."
        )

    if repeat <= 0:
        raise ValueError(
            "benchmark_repeat는 1 이상이어야 합니다."
        )

    include_gpio = bool(
        config["benchmark_include_gpio"]
    )
    include_log = bool(
        config["benchmark_include_log"]
    )

    camera = CameraInput(
        index=int(config["camera_index"]),
        width=int(config["image_width"]),
        height=int(config["image_height"]),
    )

    classifier = ONNXClassifier(
        config["onnx_model_path"],
        config["onnx_meta_path"],
    )

    output = None
    if include_gpio:
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
            use_buzzer=False,
        )

    logger = None
    benchmark_event_log = None
    if include_log:
        base_log_path = Path(
            config["benchmark_log_path"]
        )
        benchmark_event_log = (
            base_log_path.parent
            / f"{base_log_path.stem}_{label}{base_log_path.suffix}"
        )
        logger = BenchmarkCSVLogger(
            str(benchmark_event_log),
            reset=True,
        )

    threshold = float(
        config["ai_confidence_threshold"]
    )
    warning_class = (
        config["ai_warning_class"]
    )

    latencies = []

    try:
        print("=== End-to-End Benchmark ===")
        print(f"Label        : {label}")
        print(f"Device       : {platform.node()}")
        print(
            f"Architecture : {platform.machine()}"
        )
        print(
            f"Camera       : "
            f"{config['image_width']}x"
            f"{config['image_height']}"
        )
        print(
            f"Model Input  : "
            f"{classifier.image_size}"
        )
        print(f"Warm-up      : {warmup}")
        print(f"Repeat       : {repeat}")
        print(f"GPIO         : {include_gpio}")
        print(f"Log          : {include_log}")
        if benchmark_event_log is not None:
            print(f"Event Log    : {benchmark_event_log}")
        print()

        time.sleep(1.0)

        for _ in range(warmup):
            frame = camera.read()
            tensor = bgr_frame_to_nchw(
                frame,
                classifier.image_size,
            )
            classifier.predict(tensor)

        for index in range(
            1,
            repeat + 1,
        ):
            started = time.perf_counter()

            frame = camera.read()

            tensor = bgr_frame_to_nchw(
                frame,
                classifier.image_size,
            )

            prediction = classifier.predict(
                tensor
            )

            state = ai_to_operational_state(
                predicted_class=
                    prediction["class_name"],
                confidence=
                    prediction["confidence"],
                warning_class=warning_class,
                confidence_threshold=threshold,
            )

            if output is not None:
                if state == "WARNING":
                    output.warning()
                else:
                    output.normal()

            if logger is not None:
                logger.append(
                    frame_index=index,
                    state=state,
                    confidence=
                        prediction["confidence"],
                )

            elapsed_ms = (
                time.perf_counter()
                - started
            ) * 1000.0

            latencies.append(elapsed_ms)

            if (
                index == 1
                or index % 25 == 0
                or index == repeat
            ):
                print(f"{index:03d}/{repeat}")

        stats = summarize_latency(latencies)

        print()
        print("=== Result ===")
        print(
            f"Mean : {stats.mean_ms:.3f} ms"
        )
        print(
            f"P50  : {stats.p50_ms:.3f} ms"
        )
        print(
            f"P95  : {stats.p95_ms:.3f} ms"
        )
        print(
            f"FPS  : {stats.fps_from_mean:.2f}"
        )

        result_dir = Path(
            config["benchmark_result_dir"]
        )
        result_dir.mkdir(
            parents=True,
            exist_ok=True,
        )

        detail_path = (
            result_dir
            / f"e2e_{label}_detail.csv"
        )

        with detail_path.open(
            "w",
            newline="",
            encoding="utf-8",
        ) as file:
            writer = csv.writer(file)
            writer.writerow(
                [
                    "frame",
                    "latency_ms",
                ]
            )

            for index, latency in enumerate(
                latencies,
                start=1,
            ):
                writer.writerow(
                    [
                        index,
                        f"{latency:.6f}",
                    ]
                )

        summary_path = (
            result_dir
            / f"e2e_{label}_summary.csv"
        )

        with summary_path.open(
            "w",
            newline="",
            encoding="utf-8",
        ) as file:
            writer = csv.writer(file)
            writer.writerow(
                [
                    "label",
                    "device",
                    "architecture",
                    "camera_width",
                    "camera_height",
                    "model_input",
                    "warmup",
                    "repeat",
                    "gpio",
                    "log",
                    "mean_ms",
                    "p50_ms",
                    "p95_ms",
                    "fps",
                ]
            )
            writer.writerow(
                [
                    label,
                    platform.node(),
                    platform.machine(),
                    config["image_width"],
                    config["image_height"],
                    classifier.image_size,
                    warmup,
                    repeat,
                    include_gpio,
                    include_log,
                    f"{stats.mean_ms:.6f}",
                    f"{stats.p50_ms:.6f}",
                    f"{stats.p95_ms:.6f}",
                    f"{stats.fps_from_mean:.6f}",
                ]
            )

        print()
        print(f"Detail  : {detail_path}")
        print(f"Summary : {summary_path}")

    finally:
        camera.release()

        if output is not None:
            output.close()


if __name__ == "__main__":
    main()
```

`use_buzzer=False`인 이유는 Benchmark 반복 중 Buzzer가 계속 울리는 것을 막기 위한 것입니다. 오늘 GPIO 비교의 필수 출력은 LED만으로 충분합니다.

---

# 19. Raspberry Pi End-to-End Benchmark 실행하기

기본 설정:

```yaml
benchmark_include_gpio: false
benchmark_include_log: true
```

실행:

```bash
python -m scripts.benchmark_e2e \
  --label pi_baseline
```

측정 중에는 Camera 앞의 장면을 크게 바꾸지 않습니다.

100 Frame 측정이 끝나면 다음 값을 기록합니다.

```text
Mean
P50
P95
FPS
```

---

# 20. Model-only와 End-to-End 결과 비교하기

Report에 추가합니다.

```md
## Raspberry Pi: Model-only vs End-to-End

| 범위 | Mean Latency | P50 | P95 | FPS |
|---|---:|---:|---:|---:|
| Model-only |  |  |  |  |
| End-to-End |  |  |  |  |
```

보통 End-to-End에는 더 많은 작업이 들어가므로 Model-only와 다른 결과가 나옵니다.

```text
차이
=
Camera
+
Preprocess
+
Decision
+
선택 GPIO
+
선택 Log
```

---

# 21. Log가 성능에 영향을 주는지 한 변수만 바꾸기

실험 A:

```yaml
benchmark_include_log: false
benchmark_include_gpio: false
```

실행:

```bash
python -m scripts.benchmark_e2e \
  --label log_off
```

결과를 별도로 기록합니다.

실험 B:

```yaml
benchmark_include_log: true
benchmark_include_gpio: false
```

다시 실행합니다.

```bash
python -m scripts.benchmark_e2e \
  --label log_on
```

두 실험은 `reports/day08/e2e_log_off_summary.csv`와 `reports/day08/e2e_log_on_summary.csv`로 따로 남습니다.

다른 조건은 바꾸지 않습니다.

Report:

```md
## Log Overhead

| Log | Mean | P95 | FPS |
|---|---:|---:|---:|
| OFF |  |  |  |
| ON |  |  |  |

관찰:
```

---

# 22. GPIO 포함 여부도 별도로 비교하기

Buzzer가 반복해서 울리지 않도록 Benchmark 코드에서는 `use_buzzer=False`로 했습니다.

실험 A:

```yaml
benchmark_include_gpio: false
```

```bash
python -m scripts.benchmark_e2e \
  --label gpio_off
```

실험 B:

```yaml
benchmark_include_gpio: true
```

```bash
python -m scripts.benchmark_e2e \
  --label gpio_on
```

두 조건을 비교합니다.

```md
## GPIO Overhead

| GPIO | Mean | P95 | FPS |
|---|---:|---:|---:|
| OFF |  |  |  |
| ON |  |  |  |

관찰:
```

---

# 23. End-to-End 시간을 단계별로 나누어 측정하기

전체가 느리다는 것만 알아서는 원인을 찾기 어렵습니다. 이번 Profile은 **핵심 계산 경로**를 다음 단계로 나누어 봅니다.

```text
Camera Read
Preprocess
Inference
Decision
Total
```

GPIO와 Log의 영향은 앞의 `benchmark_include_gpio`, `benchmark_include_log` A/B 실험에서 별도로 확인합니다. 따라서 아래 Profile 코드가 `Output / Log` 시간을 따로 출력하지 않는 것은 의도된 구조입니다.

## 이 파일은 왜 필요할까요?

End-to-End Mean이 30ms라고 해도 어디에서 30ms가 발생했는지 모르면 무엇을 개선할지 결정할 수 없습니다. `profile_e2e_stages.py`는 같은 Pipeline을 여러 구간으로 잘라 **병목 후보를 찾는 진단용 프로그램**입니다.

## 의사코드

```text
Config에서 Repeat / Camera / Model 설정을 읽는다
        ↓
Camera와 ONNX Classifier를 준비한다
        ↓
Warm-up을 수행한다
        ↓
각 반복에서
    Camera 전후 시간 기록
    Preprocess 전후 시간 기록
    Inference 전후 시간 기록
    Decision 전후 시간 기록
    전체 전후 시간 기록
        ↓
각 단계의 Mean / P95 계산
        ↓
가장 큰 구간을 병목 후보로 본다
```

파일:

```text
scripts/profile_e2e_stages.py
```

코드:

```python
import time

from src.ai_preprocess import (
    bgr_frame_to_nchw,
)
from src.camera_input import CameraInput
from src.config_loader import load_config
from src.decision import (
    ai_to_operational_state,
)
from src.inference_onnx import (
    ONNXClassifier,
)
from src.performance import (
    summarize_latency,
)


def main():
    config = load_config(
        "configs/settings.yaml"
    )

    repeat = int(
        config["benchmark_repeat"]
    )

    camera = CameraInput(
        index=int(config["camera_index"]),
        width=int(config["image_width"]),
        height=int(config["image_height"]),
    )

    classifier = ONNXClassifier(
        config["onnx_model_path"],
        config["onnx_meta_path"],
    )

    threshold = float(
        config["ai_confidence_threshold"]
    )

    warning_class = (
        config["ai_warning_class"]
    )

    camera_times = []
    preprocess_times = []
    inference_times = []
    decision_times = []
    total_times = []

    try:
        time.sleep(1.0)

        for _ in range(20):
            frame = camera.read()

            tensor = bgr_frame_to_nchw(
                frame,
                classifier.image_size,
            )

            classifier.predict(tensor)

        for _ in range(repeat):
            total_start = (
                time.perf_counter()
            )

            t0 = time.perf_counter()

            frame = camera.read()

            t1 = time.perf_counter()

            tensor = bgr_frame_to_nchw(
                frame,
                classifier.image_size,
            )

            t2 = time.perf_counter()

            prediction = (
                classifier.predict(
                    tensor
                )
            )

            t3 = time.perf_counter()

            ai_to_operational_state(
                predicted_class=
                    prediction["class_name"],
                confidence=
                    prediction["confidence"],
                warning_class=
                    warning_class,
                confidence_threshold=
                    threshold,
            )

            t4 = time.perf_counter()

            camera_times.append(
                (t1 - t0) * 1000.0
            )

            preprocess_times.append(
                (t2 - t1) * 1000.0
            )

            inference_times.append(
                (t3 - t2) * 1000.0
            )

            decision_times.append(
                (t4 - t3) * 1000.0
            )

            total_times.append(
                (
                    time.perf_counter()
                    - total_start
                )
                * 1000.0
            )

        print(
            "=== Stage Profile ==="
        )

        for name, values in [
            (
                "Camera",
                camera_times,
            ),
            (
                "Preprocess",
                preprocess_times,
            ),
            (
                "Inference",
                inference_times,
            ),
            (
                "Decision",
                decision_times,
            ),
            (
                "Total",
                total_times,
            ),
        ]:
            stats = summarize_latency(
                values
            )

            print(
                f"{name:10s} "
                f"mean={stats.mean_ms:8.3f} "
                f"p95={stats.p95_ms:8.3f}"
            )

    finally:
        camera.release()


if __name__ == "__main__":
    main()
```

---

# 24. 단계별 Profile 실행하기

```bash
python -m scripts.profile_e2e_stages
```

예시 형태:

```text
Camera      mean=...
Preprocess  mean=...
Inference   mean=...
Decision    mean=...
Total       mean=...
```

가장 큰 값을 확인합니다.

이 값이 현재 Pipeline의 **병목 후보**입니다.

Report:

```md
## Stage Profile

| Stage | Mean | P95 | 관찰 |
|---|---:|---:|---|
| Camera |  |  |  |
| Preprocess |  |  |  |
| Inference |  |  |  |
| Decision |  |  |  |
| Total |  |  |  |

현재 가장 큰 구간:
```


## 코드 리뷰 — Stage Profile의 숫자를 역추적하기

예를 들어 다음 결과가 나왔다고 가정합니다.

```text
Camera      mean=12.100 p95=15.300
Preprocess  mean= 1.800 p95= 2.300
Inference   mean= 7.200 p95= 9.500
Decision    mean= 0.030 p95= 0.050
Total       mean=21.300 p95=26.900
```

직접 코드에서 다음을 찾습니다.

```text
Camera
→ frame = camera.read()

Preprocess
→ bgr_frame_to_nchw()

Inference
→ classifier.predict()

Decision
→ ai_to_operational_state()

Total
→ total_start부터 현재 반복의 마지막 측정지점까지
```

여기서 가장 큰 값이 곧바로 “반드시 고쳐야 하는 오류”라는 뜻은 아닙니다. **9일차에서 무엇을 먼저 실험할지 정하는 병목 후보**입니다.

---

# 25. Resource 상태도 측정 조건으로 기록하기

성능 측정 중 장치 상태도 결과에 영향을 줄 수 있습니다.

Raspberry Pi에서:

```bash
free -h
```

```bash
uptime
```

```bash
vcgencmd measure_temp
```

지원되는 경우 CPU Clock:

```bash
vcgencmd measure_clock arm
```

측정 전과 후의 온도를 기록합니다.

Report:

```md
## Raspberry Pi Resource Condition

Before Temperature:

After Temperature:

Memory:

Load Average:

전원/냉각 조건:
```

CPU Temperature가 크게 올라가는 환경에서는 반복 측정 결과가 달라질 수 있습니다.

---

# PART E. 한 번에 한 변수만 바꾸어 성능을 비교하기

# 26. 입력 크기 실험은 현재 모델에 맞게 진행하기

개요에서 `224 vs 160` 같은 예시를 사용할 수 있지만, 6일차 Tiny CNN은 기본적으로 `96 × 96` 입력으로 학습했습니다.

따라서 오늘 기본 비교는 다음처럼 진행합니다.

```text
96 × 96
vs
64 × 64
```

모델 구조에 `AdaptiveAvgPool2d`가 있기 때문에 같은 Weight를 다른 공간 크기로 Export해 실험할 수 있습니다.

단, **입력 크기를 바꾸면 Accuracy도 달라질 수 있으므로 속도와 Accuracy를 함께 봅니다.**

---

# 27. 64 × 64 ONNX Variant 만들기

7일차 `export_onnx.py`는 기본 96 × 96으로 Export했습니다. 오늘은 **같은 Weight를 유지하면서 ONNX 입력 크기만 바꾸었을 때** 속도와 Accuracy가 어떻게 달라지는지 보기 위한 Benchmark Variant를 만듭니다.

> 이 실험은 “64가 더 좋은 모델”을 새로 학습하는 과정이 아닙니다. 같은 Tiny CNN Weight를 다른 입력 Shape로 Export하여 추론 조건을 바꾸는 실험입니다.

## 이 파일은 왜 필요할까요?

기존 `day07_tiny_cnn.onnx`를 덮어쓰면 7일차 Baseline을 잃습니다. 그래서 Variant는 별도 이름으로 저장합니다.

```text
96 Training Weight
        ↓
96 ONNX Variant
64 ONNX Variant
        ↓
같은 Validation Dataset Accuracy 비교
        +
같은 장치 Latency 비교
```

## 의사코드

```text
--size를 입력받는다
        ↓
6일차 PyTorch Checkpoint를 읽는다
        ↓
같은 TinyCNN 구조와 Weight를 복원한다
        ↓
입력 크기에 맞는 Dummy Tensor를 만든다
        ↓
별도 이름의 ONNX로 Export한다
        ↓
ONNX Checker로 구조를 확인한다
        ↓
training_image_size와 실제 inference image_size를 Metadata에 함께 기록한다
```

오늘은 Benchmark용 Variant를 별도로 만듭니다.

파일:

```text
scripts/export_onnx_variant.py
```

코드:

```python
import argparse
import json
from pathlib import Path

import onnx
import torch

from src.config_loader import load_config
from src.tiny_cnn import TinyCNN


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--size",
        type=int,
        required=True,
    )

    return parser.parse_args()


def main():
    args = parse_args()

    config = load_config(
        "configs/settings.yaml"
    )

    checkpoint = torch.load(
        config["ai_model_path"],
        map_location="cpu",
    )

    classes = list(
        checkpoint["classes"]
    )

    training_size = int(
        checkpoint["image_size"]
    )

    model = TinyCNN(
        num_classes=len(classes)
    )

    model.load_state_dict(
        checkpoint["model_state"]
    )

    model.eval()

    size = int(args.size)

    output_dir = Path("models")
    output_dir.mkdir(
        parents=True,
        exist_ok=True,
    )

    onnx_path = (
        output_dir
        / f"day08_tiny_cnn_{size}.onnx"
    )

    meta_path = (
        output_dir
        / f"day08_tiny_cnn_{size}.meta.json"
    )

    dummy = torch.randn(
        1,
        3,
        size,
        size,
        dtype=torch.float32,
    )

    torch.onnx.export(
        model,
        dummy,
        str(onnx_path),
        export_params=True,
        opset_version=17,
        do_constant_folding=True,
        input_names=["input"],
        output_names=["logits"],
        dynamo=False,
    )

    onnx_model = onnx.load(
        str(onnx_path)
    )

    onnx.checker.check_model(
        onnx_model
    )

    metadata = {
        "model_type": "TinyCNN",
        "classes": classes,
        "class_to_idx":
            checkpoint["class_to_idx"],
        "training_image_size":
            training_size,
        "image_size": size,
        "input_shape": [
            1,
            3,
            size,
            size,
        ],
        "input_name": "input",
        "output_name": "logits",
        "onnx_opset": 17,
    }

    meta_path.write_text(
        json.dumps(
            metadata,
            ensure_ascii=False,
            indent=2,
        ),
        encoding="utf-8",
    )

    print("=== Variant Export ===")
    print(
        f"Training Size : "
        f"{training_size}"
    )
    print(
        f"Inference Size: {size}"
    )
    print(f"ONNX : {onnx_path}")
    print(f"Meta : {meta_path}")


if __name__ == "__main__":
    main()
```

---

# 28. 96과 64 Variant Export하기

PC에서 실행합니다.

96:

```bash
python -m scripts.export_onnx_variant \
  --size 96
```

64:

```bash
python -m scripts.export_onnx_variant \
  --size 64
```

확인:

```bash
ls -lh models/day08_tiny_cnn_*
```

두 ONNX는 **같은 Weight를 사용하지만 입력 Shape가 다릅니다.**

---

# 29. Validation Dataset으로 입력 크기별 Accuracy 평가 프로그램 만들기

속도만 빨라졌다고 운영에 더 좋은 조건이라고 결론 내릴 수 없습니다. 입력 크기를 줄이면 이미지 정보량과 분포가 달라지므로 **같은 Validation Dataset에서 Accuracy도 다시 확인**합니다. 6일차의 Test Dataset은 최종 평가 성격을 유지하기 위해 반복적인 조건 선택에는 사용하지 않습니다.

## 이 파일은 왜 필요할까요?

`benchmark_model_only.py`는 속도를 측정하지만 정답 Label을 사용하지 않습니다. `evaluate_onnx.py`는 Validation Dataset의 Class Folder를 기준으로 Prediction을 비교하여 Accuracy와 Confusion Matrix를 확인합니다.

## 의사코드

```text
--model / --meta / --dataset을 받는다
        ↓
ONNXClassifier를 만든다
        ↓
Metadata의 Class 순서대로 Folder를 확인한다
        ↓
각 이미지 파일을 읽는다
        ↓
해당 Variant의 image_size로 전처리한다
        ↓
ONNX 추론한다
        ↓
정답 Class Index와 Prediction Index를 비교한다
        ↓
Correct / Total을 누적한다
        ↓
Confusion Matrix를 누적한다
        ↓
Accuracy와 Matrix를 출력한다
```

파일:

```text
scripts/evaluate_onnx.py
```

코드:

```python
import argparse
from pathlib import Path

import numpy as np
from PIL import Image

from src.ai_preprocess import (
    pil_to_nchw_float32,
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
    return parser.parse_args()


def iter_images(directory: Path):
    for path in sorted(directory.iterdir()):
        if (
            path.is_file()
            and path.suffix.lower() in IMAGE_SUFFIXES
        ):
            yield path


def main():
    args = parse_args()

    classifier = ONNXClassifier(
        args.model,
        args.meta,
    )

    root = Path(args.dataset)
    if not root.exists():
        raise FileNotFoundError(
            f"Validation Dataset이 없습니다: {root}"
        )

    total = 0
    correct = 0

    matrix = np.zeros(
        (
            len(classifier.classes),
            len(classifier.classes),
        ),
        dtype=np.int64,
    )

    for true_index, class_name in enumerate(
        classifier.classes
    ):
        class_dir = root / class_name

        if not class_dir.exists():
            raise FileNotFoundError(
                f"Class Folder가 없습니다: {class_dir}"
            )

        for image_path in iter_images(class_dir):
            with Image.open(image_path) as image:
                tensor = pil_to_nchw_float32(
                    image,
                    classifier.image_size,
                )

            result = classifier.predict(tensor)
            pred_index = int(
                result["class_index"]
            )

            matrix[
                true_index,
                pred_index,
            ] += 1

            total += 1
            if pred_index == true_index:
                correct += 1

    if total == 0:
        raise ValueError(
            f"평가할 이미지가 없습니다: {root}"
        )

    accuracy = correct / total

    print("=== ONNX Test ===")
    print(
        f"Input Size : {classifier.image_size}"
    )
    print(f"Samples    : {total}")
    print(f"Correct    : {correct}")
    print(f"Accuracy   : {accuracy:.4f}")
    print()
    print("Confusion Matrix")
    print("Rows=True, Cols=Prediction")

    for name, row in zip(
        classifier.classes,
        matrix.tolist(),
    ):
        print(f"{name:12s} {row}")


if __name__ == "__main__":
    main()
```

예상 결과의 숫자는 Dataset과 Model에 따라 달라집니다. 중요한 것은 **96과 64를 같은 Validation Dataset으로 평가했다는 것**입니다.

---

# 30. 96 × 96 Validation Accuracy 확인하기

PC:

```bash
python -m scripts.evaluate_onnx \
  --model models/day08_tiny_cnn_96.onnx \
  --meta models/day08_tiny_cnn_96.meta.json
```

결과 기록:

```text
Accuracy:
```

---

# 31. 64 × 64 Validation Accuracy 확인하기

```bash
python -m scripts.evaluate_onnx \
  --model models/day08_tiny_cnn_64.onnx \
  --meta models/day08_tiny_cnn_64.meta.json
```

결과 기록:

```text
Accuracy:
```

두 결과를 비교합니다.

```text
96
→ Accuracy ?

64
→ Accuracy ?
```

입력 크기가 작아지면 항상 Accuracy가 떨어진다고 단정하지 않습니다.

자신의 Validation 결과를 사용합니다.

---

# 32. Raspberry Pi로 Benchmark Variant 전송하기

PC에서:

```bash
scp \
models/day08_tiny_cnn_96.onnx \
models/day08_tiny_cnn_96.meta.json \
models/day08_tiny_cnn_64.onnx \
models/day08_tiny_cnn_64.meta.json \
<RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/models/
```

Raspberry Pi에서 확인:

```bash
ls -lh models/day08_tiny_cnn_*
```

---

# 33. Raspberry Pi에서 96 × 96 Model-only Benchmark

```bash
python -m scripts.benchmark_model_only \
  --image data/deploy_samples/normal.jpg \
  --model models/day08_tiny_cnn_96.onnx \
  --meta models/day08_tiny_cnn_96.meta.json \
  --label pi_96
```

결과를 기록합니다.

---

# 34. Raspberry Pi에서 64 × 64 Model-only Benchmark

```bash
python -m scripts.benchmark_model_only \
  --image data/deploy_samples/normal.jpg \
  --model models/day08_tiny_cnn_64.onnx \
  --meta models/day08_tiny_cnn_64.meta.json \
  --label pi_64
```

다른 조건은 동일해야 합니다.

```text
Device
Image
Warm-up
Repeat
Runtime
Batch
```

변경한 것은:

```text
Input Size
```

뿐입니다.

---

# 35. PC에서도 96 / 64를 같은 조건으로 측정하기

PC:

```bash
python -m scripts.benchmark_model_only \
  --image data/deploy_samples/normal.jpg \
  --model models/day08_tiny_cnn_96.onnx \
  --meta models/day08_tiny_cnn_96.meta.json \
  --label pc_96
```

```bash
python -m scripts.benchmark_model_only \
  --image data/deploy_samples/normal.jpg \
  --model models/day08_tiny_cnn_64.onnx \
  --meta models/day08_tiny_cnn_64.meta.json \
  --label pc_64
```

---

# 36. 입력 크기 비교표 작성하기

Report에 추가합니다.

```md
## Input Size Experiment

### Accuracy

| Input | Validation Accuracy |
|---:|---:|
| 96 × 96 |  |
| 64 × 64 |  |

### Model-only Performance

| Device | Input | Mean | P50 | P95 | FPS* |
|---|---:|---:|---:|---:|---:|
| PC | 96 |  |  |  |  |
| PC | 64 |  |  |  |  |
| Raspberry Pi | 96 |  |  |  |  |
| Raspberry Pi | 64 |  |  |  |  |

가장 빠른 조건:

Accuracy가 가장 높은 조건:

속도와 Accuracy를 함께 고려한 임시 선택:
```

---

# 37. Accuracy는 장치 성능 지표와 구분하기

같은 ONNX 모델, 같은 전처리, 같은 Validation Dataset을 사용하면 Accuracy는 기본적으로 **모델과 입력 조건의 특성**입니다.

따라서 다음처럼 구분합니다.

```text
Accuracy
→ 모델 / 데이터 / 입력 크기 관점

Latency / FPS
→ 장치 / Runtime / 실행조건 관점
```

그래서 다음 표처럼 보는 것이 더 정확합니다.

```text
Input 96
→ Accuracy A
→ PC Latency
→ Pi Latency

Input 64
→ Accuracy B
→ PC Latency
→ Pi Latency
```

---

# 38. Camera Resolution과 Model Input Size도 구분하기

현재 Camera 설정이:

```yaml
image_width: 640
image_height: 480
```

이라고 해서 AI가 `640 × 480`을 그대로 입력받는 것은 아닙니다.

구조:

```text
Camera
640 × 480
        ↓
Preprocess
        ↓
96 × 96
        ↓
ONNX
```

또는:

```text
Camera
640 × 480
        ↓
Preprocess
        ↓
64 × 64
        ↓
ONNX
```

따라서 다음 둘을 별도로 기록합니다.

```text
Camera Resolution
Model Input Size
```

---

# 39. Camera Resolution만 바꾸어 End-to-End 비교하기

이번에는 Model Input은 96으로 고정하고 Camera Resolution만 바꿉니다.

실험 A:

```yaml
image_width: 640
image_height: 480
```

실험 B:

Camera가 지원한다면:

```yaml
image_width: 1280
image_height: 720
```

각각 Label을 다르게 하여 실행합니다.

640 × 480:

```bash
python -m scripts.benchmark_e2e \
  --label camera_640x480
```

1280 × 720이 실제 Camera에서 지원되는 경우:

```bash
python -m scripts.benchmark_e2e \
  --label camera_1280x720
```

실험 전에 `scripts.camera_index_probe` 또는 `v4l2-ctl --list-formats-ext`로 지원 해상도를 확인합니다. 지원하지 않는 해상도를 억지로 사용하지 않습니다.

주의:

```text
Model Input
→ 96으로 고정

변경
→ Camera Resolution만
```

Report:

```md
## Camera Resolution Experiment

| Camera | Model Input | Mean | P95 | FPS |
|---|---:|---:|---:|---:|
| 640 × 480 | 96 |  |  |  |
| 1280 × 720 | 96 |  |  |  |

관찰:
```

실험 후 수업 기본값으로 복구합니다.

---

# 40. 반복 횟수가 너무 적을 때 생기는 문제 보기

설정:

```yaml
benchmark_repeat: 5
```

측정:

```bash
python -m scripts.benchmark_model_only \
  --image data/deploy_samples/normal.jpg \
  --label pi_repeat5
```

그다음:

```yaml
benchmark_repeat: 100
```

다시 측정합니다.

비교:

```text
5회 결과
→ 우연한 값의 영향을 크게 받을 수 있음

100회 결과
→ 분포를 조금 더 안정적으로 볼 수 있음
```

실험 후:

```yaml
benchmark_repeat: 100
```

으로 복구합니다.

---

# 41. Warm-up을 0으로 만들어 비교해 보기

실험 A:

```yaml
benchmark_warmup: 0
```

실행합니다.

실험 B:

```yaml
benchmark_warmup: 20
```

다시 실행합니다.

다른 조건은 그대로 둡니다.

Report:

```md
## Warm-up Experiment

| Warm-up | Mean | P50 | P95 | Max |
|---:|---:|---:|---:|---:|
| 0 |  |  |  |  |
| 20 |  |  |  |  |

첫 실행의 영향:
```

환경에 따라 차이가 크지 않을 수도 있습니다.

그 결과도 그대로 기록합니다.

---

# 42. 일부러 성능 측정을 망가뜨리기 — 매 Frame print

`benchmark_e2e.py`는 매 Frame마다 결과를 출력하지 않습니다.

별도 실험에서 매 Frame 다음처럼 출력해 봅니다.

```python
print(
    index,
    state,
    prediction["confidence"],
)
```

성능을 다시 측정합니다.

다른 조건은 유지합니다.

질문:

```text
FPS가 달라졌는가?

P95가 달라졌는가?

왜 화면 출력도 측정 조건에 포함해야 하는가?
```

실험 후 원래 상태로 복구합니다.

---

# 43. 일부러 성능 측정을 망가뜨리기 — 다른 프로그램 동시에 실행

Benchmark 실행 중 다른 무거운 작업을 실행하면 CPU 자원을 경쟁할 수 있습니다.

무리하게 시스템을 과부하시키는 실험 대신 현재 Load를 확인합니다.

```bash
uptime
```

다른 프로그램이 이미 실행 중이면:

```bash
ps aux --sort=-%cpu | head
```

로 확인합니다.

성능 비교 전에는 가능한 한 불필요한 프로그램을 종료합니다.

이것이 **측정 조건 고정**의 일부입니다.

---

# 44. P95가 Mean보다 중요한 상황 생각하기

예:

```text
Mean
20 ms

P95
65 ms
```

평균은 빠르지만 일부 Frame이 크게 느립니다.

현장 경고 시스템에서는 다음 질문이 중요할 수 있습니다.

```text
평균적으로 빠른가?
+
대부분의 Frame도 안정적으로 빠른가?
```

따라서 다음처럼 함께 기록합니다.

```text
Mean
P50
P95
```

---

# 45. Guided Lab 마무리 — 최종 성능 비교표 만들기

오늘 실험 결과로 다음 표를 완성합니다.

```md
## Final Comparison

| Device | Input | Validation Accuracy | Model Mean | Model P95 | Model FPS* |
|---|---:|---:|---:|---:|---:|
| PC | 96 |  |  |  |  |
| PC | 64 |  |  |  |  |
| Raspberry Pi | 96 |  |  |  |  |
| Raspberry Pi | 64 |  |  |  |  |

## Raspberry Pi End-to-End

| Camera | Model Input | Log | GPIO | Mean | P95 | FPS |
|---|---:|---|---|---:|---:|---:|
| 640 × 480 | 96 | OFF | OFF |  |  |  |
| 640 × 480 | 96 | ON | OFF |  |  |  |
| 640 × 480 | 96 | ON | ON |  |  |  |

## 선택

가장 빠른 Model Input:

Accuracy가 가장 높은 Model Input:

현재 병목:

현장에 사용할 임시 설정:

선택 이유:
```

---

# 46. Guided Lab 결과를 자신의 측정값으로 해석하기

결과 숫자를 채운 뒤 자신의 데이터로 답합니다.

```text
1. PC와 Raspberry Pi 중
   Model-only 추론이 더 빠른 장치는?

2. Raspberry Pi에서
   96과 64 중 더 빠른 Input은?

3. Input Size를 줄였을 때
   Accuracy는 어떻게 변했는가?

4. Model-only와 End-to-End의
   차이는 얼마나 되는가?

5. End-to-End에서
   가장 시간이 많이 걸리는 단계는?

6. Log를 켰을 때
   성능은 얼마나 달라졌는가?

7. Mean과 P95의 차이는 큰가?

8. 현재 설정을 현장에 사용한다면
   어떤 조합을 선택할 것인가?
```

정답은 하나가 아닙니다.

자신의 측정 결과가 근거가 되어야 합니다.


---

# PART F. Mini Challenge — 측정 결과를 보고 스스로 원인을 찾고 증명하기

지금까지는 안내된 순서대로 Benchmark를 만들고 실행했습니다. 이제부터는 결과를 보고 코드와 설정의 원인을 직접 찾습니다.

Mini Challenge에서는 정답을 먼저 열지 않습니다.

```text
결과 확인
        ↓
코드 위치 찾기
        ↓
실행 전 결과 예상
        ↓
설정만 변경
        ↓
기능 일부 수정
        ↓
일부러 오류 만들기
        ↓
CSV / Log로 증명
        ↓
전체 Pipeline 설명
```

각 Challenge를 먼저 수행한 뒤 **예시 정답 확인해보기**를 펼쳐 자신의 결과와 비교합니다.

---

## Challenge 1 — Summary 결과를 코드로 역추적하기

다음과 같은 결과가 나왔다고 가정합니다.

```text
=== Result ===
Mean : 18.420 ms
P50  : 17.910 ms
P95  : 24.880 ms
FPS  : 54.29
```

다음 표를 직접 채웁니다.

| 화면에 보인 값 | 담당 파일 | 담당 코드/함수 |
|---|---|---|
| Mean |  |  |
| P50 |  |  |
| P95 |  |  |
| FPS |  |  |
| 반복별 Latency |  |  |
| Summary CSV |  |  |

그리고 다음 질문에 답합니다.

```text
1. 시간을 재는 시작점은 어디인가?
2. End-to-End에서 Camera read는 측정 범위에 들어가는가?
3. Mean/P50/P95 계산은 어느 함수가 담당하는가?
4. Summary 파일명에 --label이 들어가는 이유는 무엇인가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

| 값 | 담당 파일 | 코드/함수 |
|---|---|---|
| Mean/P50/P95/FPS | `src/performance.py` | `summarize_latency()` |
| 측정 시작/종료 | `scripts/benchmark_e2e.py` | `time.perf_counter()` |
| 반복별 Latency | `scripts/benchmark_e2e.py` | `latencies.append(...)` |
| Summary CSV | `scripts/benchmark_e2e.py` | `e2e_<label>_summary.csv` 작성 부분 |

End-to-End에서는 `started` 뒤에 `camera.read()`가 있으므로 Camera read가 측정 범위에 포함됩니다.

`--label`은 `log_off`, `log_on`, `gpio_on` 같은 실험 결과가 같은 `e2e_summary.csv` 하나에 계속 덮어써지는 문제를 막습니다.

</details>

---

## Challenge 2 — 실행 전에 P95 변화를 예상하기

현재 조건:

```yaml
benchmark_warmup: 20
benchmark_repeat: 100
benchmark_include_gpio: false
benchmark_include_log: false
```

다음 두 상황을 실행하기 전에 먼저 예상합니다.

```text
A. 같은 조건에서 Log만 true로 변경
B. 매 Frame print()를 측정 구간 안에 추가
```

다음 문장을 먼저 완성합니다.

```text
A에서 Mean은 __________________ 가능성이 있다.
A에서 P95는 __________________ 가능성이 있다.

B에서 측정 결과는 __________________ 가능성이 있다.
그 이유는 ____________________________________________.
```

그 뒤 실제 실험 결과와 비교합니다. 결과가 예상과 다르면 **예상이 틀린 것도 정상적인 학습 결과**입니다.

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

파일 I/O를 매 반복 수행하면 Mean 또는 P95가 증가할 가능성이 있습니다. 하지만 실제 증가량은 Storage, OS Cache, 현재 장치 부하에 따라 달라지므로 반드시 자신의 측정값으로 확인합니다.

`print()`도 터미널 I/O이므로 측정 구간 안에 넣으면 실제 계산 Pipeline 이외의 시간이 섞일 수 있습니다. 그래서 기본 Benchmark는 매 Frame 출력을 측정 범위 밖으로 둡니다.

</details>

---

## Challenge 3 — Python 코드는 바꾸지 않고 설정값만 비교하기

이번 Challenge에서는 Python 파일을 수정하지 않습니다.

실험 A:

```yaml
benchmark_warmup: 0
benchmark_repeat: 30
benchmark_include_gpio: false
benchmark_include_log: false
```

실험 B:

```yaml
benchmark_warmup: 20
benchmark_repeat: 100
benchmark_include_gpio: false
benchmark_include_log: false
```

각각 Label을 다르게 실행합니다.

```bash
python -m scripts.benchmark_e2e \
  --label challenge_a
```

```bash
python -m scripts.benchmark_e2e \
  --label challenge_b
```

비교합니다.

```text
Mean
P50
P95
Max
결과의 흔들림
```

질문:

```text
Python 코드를 바꾸지 않았는데 왜 결과 조건이 달라졌는가?
두 Summary CSV의 warmup / repeat 값은 실제 설정과 일치하는가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

`benchmark_e2e.py`는 `configs/settings.yaml`에서 Warm-up과 Repeat 값을 읽기 때문에 Python Source를 수정하지 않아도 측정 조건을 바꿀 수 있습니다.

```text
settings.yaml
        ↓
load_config()
        ↓
benchmark_warmup / benchmark_repeat
        ↓
실제 반복 횟수 변경
```

Summary CSV의 `warmup`, `repeat` 열도 설정과 같아야 합니다. 다르면 실행한 Source 또는 설정파일이 다른지 확인합니다.

</details>

---

## Challenge 4 — 기존 기능을 수정하여 “느린 Frame 개수” 추가하기

요구사항:

> P95보다 느린 Frame이 몇 개인지 Summary와 터미널에서 보고 싶습니다.

바로 정답을 보지 말고 어느 파일을 수정해야 하는지 먼저 생각합니다.

힌트:

```text
모든 Latency 원본
→ latencies

P95
→ stats.p95_ms
```

먼저 `scripts/benchmark_e2e.py`에서 직접 구현합니다.

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

통계를 계산한 뒤 다음처럼 추가할 수 있습니다.

```python
slow_count = sum(
    latency > stats.p95_ms
    for latency in latencies
)

print(
    f"Above P95 Frames : {slow_count}"
)
```

Summary CSV Header에도 새 열을 추가합니다.

```python
"above_p95_frames",
```

Data Row에는 다음 값을 추가합니다.

```python
slow_count,
```

예상 결과는 보간 방식과 값의 분포에 따라 정확히 5개가 아닐 수도 있습니다. 중요한 것은 **P95보다 큰 실제 측정값을 코드로 다시 세었다는 것**입니다.

이 Challenge는 기능 수정 연습이므로 종료 후 기본 Source로 복원합니다.

</details>

---

## Challenge 5 — 일부러 설정 오류를 만들고 원인을 찾기

`configs/settings.yaml`을 잠시 다음처럼 변경합니다.

```yaml
benchmark_repeat: 0
```

실행합니다.

```bash
python -m scripts.benchmark_e2e \
  --label wrong_repeat
```

오류가 발생하면 다음 순서로 확인합니다.

```text
오류 마지막 줄 읽기
        ↓
어떤 설정값이 잘못되었는지 찾기
        ↓
settings.yaml 확인
        ↓
코드의 Validation 위치 확인
        ↓
정상값으로 수정
        ↓
재실행
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

최종본의 `benchmark_e2e.py`는 다음 검사를 합니다.

```python
if repeat <= 0:
    raise ValueError(
        "benchmark_repeat는 1 이상이어야 합니다."
    )
```

따라서 오류는 Camera나 ONNX 문제가 아니라 **Benchmark Config 오류**입니다.

복구:

```yaml
benchmark_repeat: 100
```

다시 실행하여 정상 동작하는지 확인합니다.

</details>

---

## Challenge 6 — CSV로 실제 Benchmark 수행을 증명하기

설정을 다음 Baseline으로 맞춥니다.

```yaml
benchmark_warmup: 20
benchmark_repeat: 100
benchmark_include_gpio: false
benchmark_include_log: true
```

실행합니다.

```bash
python -m scripts.benchmark_e2e \
  --label challenge_proof
```

다음 파일을 확인합니다.

```bash
head reports/day08/e2e_challenge_proof_detail.csv
```

```bash
cat reports/day08/e2e_challenge_proof_summary.csv
```

반복 결과 줄 수도 확인합니다.

```bash
wc -l reports/day08/e2e_challenge_proof_detail.csv
```

질문:

```text
Repeat가 100이면 Detail CSV는 왜 101줄인가?
Summary CSV에는 왜 2줄만 있는가?
Event Log가 켜졌다면 어떤 별도 파일이 만들어지는가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
Detail CSV
Header 1줄 + 측정 100줄
→ 총 101줄

Summary CSV
Header 1줄 + Summary 1줄
→ 총 2줄
```

Log가 켜져 있고 Label이 `challenge_proof`라면 설정한 기본 Log 이름을 바탕으로 다음과 같은 Label 전용 파일이 만들어집니다.

```text
logs/day08_e2e_events_challenge_proof.csv
```

이 파일은 각 반복의 `frame / state / confidence`를 기록합니다.

즉 화면에 보이는 결과만 믿는 것이 아니라 **파일로 실행 증거를 확인**한 것입니다.

</details>

---

## Challenge 7 — 오늘의 전체 Performance Pipeline을 자신의 말로 설명하기

코드를 닫고 다음 빈칸을 자신의 말로 채웁니다.

```text
7일차에는 ____________________________________________.

8일차 Model-only에서는 ________________________________.

8일차 End-to-End에서는 ________________________________.

Warm-up은 ____________________________________________.

Mean은 _______________________________________________.

P95는 ________________________________________________.

Stage Profile은 ________________________________________.

Input Size 실험에서는 속도뿐 아니라 __________________도 본다.

9일차에는 오늘 찾은 __________________를 근거로 최적화를 시도한다.
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
7일차에는 ONNX 모델을 Raspberry Pi 실제 Camera AI Pipeline에 배포했다.

Model-only에서는 이미 준비된 Tensor를 ONNX Runtime에 넣는 추론 구간을 반복 측정한다.

End-to-End에서는 Camera부터 Preprocess, ONNX, Decision, 선택 GPIO/Log까지 측정한다.

Warm-up은 초기 실행 Overhead의 영향을 줄이기 위해 본 측정 전에 수행하고 버리는 실행이다.

Mean은 반복 Latency의 평균이다.

P95는 측정값의 약 95%가 그 이하에서 처리되는 지연 수준이다.

Stage Profile은 전체 시간을 Camera/Preprocess/Inference/Decision으로 나누어 병목 후보를 찾는 과정이다.

Input Size 실험에서는 속도뿐 아니라 Validation Accuracy도 함께 본다.

9일차에는 오늘 찾은 병목과 출력 흔들림 문제를 근거로 최적화를 시도한다.
```

문장을 그대로 외울 필요는 없습니다. **같은 의미를 자신의 말로 설명할 수 있으면 됩니다.**

</details>

---

# PART G. Challenge 상태를 복원하고 9일차용 Baseline을 고정하기

# 47. Mini Challenge에서 변경한 상태를 기본값으로 복원하기

Challenge에서 설정과 코드를 여러 번 바꾸었습니다. 9일차는 8일차 측정 Baseline에서 시작하므로 기본 상태를 다시 고정합니다.

`configs/settings.yaml`에서 Benchmark 기본값을 확인합니다.

```yaml
benchmark_warmup: 20
benchmark_repeat: 100
benchmark_result_dir: reports/day08
benchmark_include_gpio: false
benchmark_include_log: true
benchmark_log_path: logs/day08_e2e_events.csv
```

Camera 기본값은 수업에서 사용한 실제 지원 해상도로 복원합니다. 기본 실습이 640 × 480이었다면:

```yaml
image_width: 640
image_height: 480
```

7일차 ONNX Baseline도 그대로 유지합니다.

```yaml
onnx_model_path: models/day07_tiny_cnn.onnx
onnx_meta_path: models/day07_tiny_cnn.meta.json
ai_warning_class: warning_red
ai_confidence_threshold: 0.70
```

`configs/device.local.yaml`의 `camera_index`는 **자신의 실제 Raspberry Pi 값**을 유지합니다. PC의 예시값으로 덮어쓰지 않습니다.

Challenge 4에서 `Above P95 Frames` 기능을 추가했다면 수업 Baseline Source로 되돌립니다. Challenge 2에서 매 Frame `print()`를 넣었다면 반드시 제거합니다.

최종 Model-only 확인:

```bash
python -m scripts.benchmark_model_only \
  --image data/deploy_samples/normal.jpg \
  --label day08_final_baseline
```

최종 End-to-End 확인:

```bash
python -m scripts.benchmark_e2e \
  --label day08_final_baseline
```

최종 Profile:

```bash
python -m scripts.profile_e2e_stages
```

이 세 결과가 9일차의 출발점입니다.

---

# 48. 8일차 성능 CSV를 한 곳에 모으기

현재 결과는 다음 폴더에 쌓입니다.

```text
reports/day08/
```

예:

```text
reports/day08/
├── model_only_pc_96_detail.csv
├── model_only_pc_96_summary.csv
├── model_only_pc_64_detail.csv
├── model_only_pc_64_summary.csv
├── model_only_pi_96_detail.csv
├── model_only_pi_96_summary.csv
├── model_only_pi_64_detail.csv
├── model_only_pi_64_summary.csv
├── e2e_*_detail.csv
└── e2e_*_summary.csv
```

실험을 여러 번 반복한다면 파일명이 덮어써지지 않도록 Label이나 파일명을 구분합니다.

---

# 49. 결과를 Git에 올릴 것과 제외할 것 구분하기

성능 Summary와 Report는 학습 결과이므로 Git으로 관리할 수 있습니다.

```text
reports/day08_performance.md
reports/day08/*_summary.csv
```

반면 아주 큰 Raw Runtime Log나 반복 생성 파일은 수업 정책에 따라 제외할 수 있습니다.

현재 100개 정도의 Latency CSV는 크기가 작으므로 학습용으로 관리해도 괜찮습니다.

단:

```text
Camera 원본
Dataset
ONNX Model
```

은 기존 원칙대로 Git에서 제외합니다.

---

# 50. 8일차 Source 첫 Commit 만들기

변경사항:

```bash
git status
```

차이:

```bash
git diff
```

Stage:

```bash
git add \
configs/settings.yaml \
src/performance.py \
src/benchmark_logger.py \
scripts/benchmark_model_only.py \
scripts/benchmark_e2e.py \
scripts/profile_e2e_stages.py \
scripts/export_onnx_variant.py \
scripts/evaluate_onnx.py
```

Commit:

```bash
git commit -m \
"feat: add edge performance benchmark tools"
```

---

# 51. Report와 결과 Commit 만들기

`reports/day08_performance.md`를 완성합니다.

Stage:

```bash
git add \
reports/day08_performance.md \
reports/day08
```

Commit:

```bash
git commit -m \
"docs: record day08 latency and fps results"
```

---

# 52. README에 8일차 실행 방법 추가하기

`README.md`에 다음 내용을 추가합니다.

```md
## Day 08 Performance

Model-only:

```bash
python -m scripts.benchmark_model_only \
  --image data/deploy_samples/normal.jpg \
  --label pi_96
```

End-to-End:

```bash
python -m scripts.benchmark_e2e \
  --label pi_baseline
```

Stage Profile:

```bash
python -m scripts.profile_e2e_stages
```

ONNX Input Variant:

```bash
python -m scripts.export_onnx_variant \
  --size 64
```

ONNX Accuracy:

```bash
python -m scripts.evaluate_onnx \
  --model models/day08_tiny_cnn_64.onnx \
  --meta models/day08_tiny_cnn_64.meta.json
```

Performance Flow:

```text
고정 조건
→ Warm-up
→ 반복 측정
→ Mean / P50 / P95
→ Model-only
→ End-to-End
→ 병목 확인
```
```

README도 Commit합니다.

```bash
git add README.md
git commit -m \
"docs: add day08 benchmark commands"
```

---

# 53. 원격 Repository에 Push하기

외부 Repository 사용이 허용된 환경이라면:

```bash
git push
```

기관 보안정책에서 외부 GitHub가 제한되면 Local Git 또는 허용된 내부 저장소를 사용합니다.

ONNX Variant 파일은 Model이므로 Git에 올리지 않습니다.

---

# 54. PC와 Raspberry Pi의 Source를 다시 맞추기

PC와 Raspberry Pi가 같은 Benchmark Source를 사용해야 비교가 의미가 있습니다. 다만 `git pull`은 **허용된 내부 GitLab/Gitea 같은 Remote가 연결되어 있을 때만** 사용합니다.

내부 Remote를 사용하는 경우, Source를 Commit/Push한 장치의 반대편에서 다음처럼 확인합니다.

```bash
git remote -v
git pull
git log --oneline -8
```

Local Git만 사용하는 경우에는 `git pull`을 억지로 실행하지 않습니다. 수업에서 정한 SCP 또는 파일 전달 방식으로 `src/`, `scripts/`, `configs/settings.yaml`의 Day 08 변경분을 전달한 뒤 다음을 확인합니다.

```bash
git status
python -m scripts.benchmark_model_only --help
python -m scripts.benchmark_e2e --help
```

`configs/device.local.yaml`은 Raspberry Pi의 Camera index 같은 장치 전용 값이므로 공통 Source 동기화 과정에서 덮어쓰지 않습니다.

---

# 55. 8일차가 끝난 시점의 프로젝트 구조

공통 Source:

```text
subject13_edge_ai/
│
├── configs/
│   └── settings.yaml
│
├── reports/
│   ├── ...
│   ├── day07_onnx_deployment.md
│   ├── day08_performance.md
│   └── day08/
│       ├── model_only_*_summary.csv
│       ├── model_only_*_detail.csv
│       ├── e2e_*_summary.csv
│       └── e2e_*_detail.csv
│
├── scripts/
│   ├── ...
│   ├── benchmark_e2e.py
│   ├── benchmark_model_only.py
│   ├── evaluate_onnx.py
│   ├── export_onnx_variant.py
│   └── profile_e2e_stages.py
│
├── src/
│   ├── ...
│   ├── benchmark_logger.py
│   └── performance.py
│
├── .gitignore
├── README.md
└── requirements.txt
```

PC와 Raspberry Pi에 별도로 존재하는 Model:

```text
models/
├── day07_tiny_cnn.onnx
├── day07_tiny_cnn.meta.json
├── day08_tiny_cnn_96.onnx
├── day08_tiny_cnn_96.meta.json
├── day08_tiny_cnn_64.onnx
└── day08_tiny_cnn_64.meta.json
```

---

# 56. 1~8일차 시스템 성장 확인하기

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
ONNX Inference
→ Warm-up
→ 반복 측정
→ Mean / P50 / P95
→ Model-only
→ End-to-End
→ 병목 확인
```

이제 시스템이 단순히 **동작하는 단계**에서 **측정할 수 있는 단계**로 바뀌었습니다.

---

# 57. 수업 종료 전 최종 확인

## Raspberry Pi

```bash
cd ~/ai_vision/subject13_edge_ai
source .venv/bin/activate
```

Model-only:

```bash
python -m scripts.benchmark_model_only \
  --image data/deploy_samples/normal.jpg \
  --label pi_96
```

End-to-End:

```bash
python -m scripts.benchmark_e2e \
  --label day08_final_baseline
```

Stage Profile:

```bash
python -m scripts.profile_e2e_stages
```

Resource:

```bash
free -h
uptime
vcgencmd measure_temp
```

Git:

```bash
git status
git log --oneline -10
```

---


---

# PART H. 최종 복습 · 결과 기록 · Git · 다음 날 연결

# 58. 핵심 복습 문제

아래 문제를 먼저 스스로 답한 뒤 예시 정답을 확인합니다.

1. Model-only와 End-to-End의 가장 큰 차이는 무엇인가요?
2. Warm-up 결과를 본 측정 통계에서 빼는 이유는 무엇인가요?
3. Mean이 10ms이고 P95가 40ms라면 어떤 점을 추가로 확인해야 하나요?
4. Model-only의 `1000 / Mean Latency`를 실제 Camera FPS와 같다고 보면 안 되는 이유는 무엇인가요?
5. PC와 Raspberry Pi 성능을 비교할 때 반드시 같게 유지해야 하는 조건은 무엇인가요?
6. Camera Resolution이 640×480이면 ONNX Model Input도 반드시 640×480인가요?
7. 입력을 96에서 64로 줄였을 때 속도만 확인하면 안 되는 이유는 무엇인가요?
8. `profile_e2e_stages.py`에서 Inference가 가장 크다면 무엇을 의미하나요?
9. Benchmark 결과 파일에 `--label`을 사용하는 이유는 무엇인가요?
10. 9일차에서 오늘의 어떤 결과를 가장 먼저 사용하게 되나요?
11. `benchmark_repeat: 0`은 왜 오류로 처리해야 하나요?
12. `configs/device.local.yaml`의 Camera index를 공통 설정처럼 덮어쓰면 안 되는 이유는 무엇인가요?

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

1. Model-only는 준비된 Tensor의 ONNX 추론 구간 중심이고, End-to-End는 Camera부터 전처리·추론·판단·선택 출력/기록까지 포함합니다.
2. 첫 실행 초기화와 Cache 등 평소 반복 실행과 다른 초기 Overhead의 영향을 줄이기 위해서입니다.
3. 평균은 빠르지만 일부 Frame이 많이 늦을 수 있으므로 Detail CSV, P95/Max, 장치 부하와 온도를 확인합니다.
4. 실제 Camera Pipeline에는 Camera read, Preprocess, Decision, GPIO/Log 같은 시간이 추가되기 때문입니다.
5. Model, 입력 이미지, Input Size, Batch, Warm-up, Repeat, Runtime/Provider 등입니다.
6. 아닙니다. Camera 640×480 Frame을 Preprocess에서 96×96 또는 다른 Model Input으로 바꿀 수 있습니다.
7. 입력 크기 변경이 분류 Accuracy에 영향을 줄 수 있으므로 같은 Validation Dataset으로 성능을 다시 확인해야 합니다. Test를 반복적인 설정 선택에 사용하지 않는 것도 중요합니다.
8. 현재 측정 조건에서 ONNX 추론 구간이 가장 큰 병목 후보라는 뜻입니다. 반드시 오류라는 뜻은 아닙니다.
9. 각 A/B 실험의 Detail/Summary가 서로 덮어써지지 않도록 구분하기 위해서입니다.
10. Day08 Baseline의 Model-only/End-to-End Mean·P95·FPS와 가장 느린 Stage를 사용합니다.
11. 측정 Sample이 하나도 없으면 통계 자체를 계산할 수 없고 실험 조건으로도 의미가 없기 때문입니다.
12. Camera index는 장치마다 다를 수 있는 Local 값이기 때문입니다.

</details>

---

# 59. 자가 체크리스트

- [ ] 7일차 ONNX Baseline이 정상 실행되는 것을 확인했다.
- [ ] Raspberry Pi의 실제 Hostname/IP/Camera index를 확인했다.
- [ ] `benchmark_warmup`과 `benchmark_repeat`의 의미를 설명할 수 있다.
- [ ] Mean/P50/P95를 구분할 수 있다.
- [ ] Model-only Benchmark를 실행하고 Detail/Summary CSV를 확인했다.
- [ ] PC와 Raspberry Pi를 같은 Model-only 조건으로 비교했다.
- [ ] End-to-End Benchmark를 실행했다.
- [ ] `--label`별로 결과 파일이 따로 저장되는 것을 확인했다.
- [ ] Log ON/OFF 또는 GPIO ON/OFF 중 하나 이상을 A/B 비교했다.
- [ ] Stage Profile에서 가장 큰 병목 후보를 찾았다.
- [ ] CPU/Memory/Temperature를 성능 조건과 함께 기록했다.
- [ ] 96/64 입력 실험에서 속도와 Accuracy를 함께 확인했다.
- [ ] Mini Challenge의 오류를 스스로 복구했다.
- [ ] Challenge에서 변경한 설정과 코드를 Baseline으로 복원했다.
- [ ] `reports/day08_performance.md`를 완성했다.
- [ ] README에 8일차 실행 명령을 남겼다.
- [ ] Source와 Report를 Git으로 저장했다.
- [ ] 9일차에서 사용할 Baseline과 병목 후보를 설명할 수 있다.

---

# 60. 다음 날 연결

오늘은 다음 질문에 숫자로 답할 수 있게 되었습니다.

```text
Model 자체는 얼마나 빠른가?

실제 Camera Pipeline은 얼마나 빠른가?

평균 Latency는?

P95는?

어느 단계가 가장 느린가?

Input Size를 바꾸면
속도와 Accuracy는 어떻게 달라지는가?
```

다음 날에는 이 측정 결과를 이용하여 실제 개선을 시도합니다.

```text
현재 Pipeline
        ↓
병목 확인
        ↓
Frame Skip
        ↓
Voting
        ↓
Hold / Stabilization
        ↓
속도와 판정 흔들림 비교
```

즉 9일차에는 단순히 코드를 빠르게 만드는 것이 아니라,

> **“측정 결과를 근거로 무엇을 바꿀 것인가?”**

를 실험하게 됩니다.
---

## 8일차 결과가 9일차에서 어떻게 사용될까요?

9일차는 오늘의 숫자를 다시 처음부터 만드는 날이 아닙니다. 오늘 남긴 Baseline을 출발점으로 개선 전·후를 비교합니다.

```text
8일차 End-to-End FPS가 부족함
        ↓
9일차 Frame Skip 후보 실험

8일차 P95가 Mean보다 많이 큼
        ↓
장치 부하 / 장시간 안정성 / 흔들림 원인 확인

8일차 Prediction State가 자주 바뀜
        ↓
9일차 Majority Voting / Debounce / Hold

8일차 CPU / Temperature 부담이 큼
        ↓
9일차 Resource Monitoring과 Trade-off 비교
```

즉 오늘의 마지막 문장은 다음과 같습니다.

> **“8일차에는 빠르다고 추측하지 않고 측정했다. 9일차에는 그 측정 근거를 이용해 한 가지씩 개선한다.”**
