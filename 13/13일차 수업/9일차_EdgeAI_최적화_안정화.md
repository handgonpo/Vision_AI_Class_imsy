
> **오늘의 핵심:** 8일차에는 Raspberry Pi Edge AI를 “느린 것 같다”라고 추측하지 않고 `Mean / P50 / P95 / FPS / Stage Profile / Resource 상태`로 측정했습니다.  
> 9일차에는 그 Baseline을 근거로 **Frame Skip · Majority Voting · Debounce · Hold · Resource Monitoring**을 적용하고, 속도·반응성·출력 안정성 사이의 Trade-off를 비교하여 10일차가 사용할 운영 Config를 선택합니다.
>
> 1~8일차에 사용한 `subject13_edge_ai` 프로젝트와 Raspberry Pi의 기존 `.venv`를 그대로 이어서 사용합니다. **새 프로젝트와 새 가상환경을 만들지 않습니다.**
>
> 실제 Raspberry Pi 사용자 이름, IP, Hostname, Camera index, 지원 Camera 해상도, Buzzer 사용 여부, Thermal 정보 경로는 환경마다 달라질 수 있습니다. 교재의 `<RPI_USER>`, `<RPI_IP>` 같은 표시는 자신의 실제 환경값으로 바꾸어 사용합니다.

---

## 오늘 가장 중요한 질문

8일차까지는 다음 질문에 답했습니다.

```text
실제 ONNX Pipeline은 얼마나 빠른가?
        ↓
Model-only Mean / P95는?
        ↓
End-to-End Mean / P95 / FPS는?
        ↓
Camera / Preprocess / Inference / Decision 중
어느 단계가 가장 큰가?
        ↓
CPU / Memory / Temperature 상태는?
```

오늘부터 질문이 바뀝니다.

```text
측정된 Baseline
        ↓
무엇을 바꾸면 계산 부담이 줄어드는가?
        ↓
무엇을 바꾸면 출력 흔들림이 줄어드는가?
        ↓
그 대신 무엇을 잃는가?
        ↓
실제 운영에서는 어떤 조합을 선택할 것인가?
```

따라서 9일차의 핵심 질문은 다음입니다.

> **“8일차 측정 결과를 근거로 한 번에 한 변수씩 바꾸고, 속도·반응성·안정성의 Trade-off를 비교하여 현재 Raspberry Pi에 맞는 운영 설정을 선택할 수 있는가?”**

---

## 8일차와 9일차는 어디가 이어질까요?

8일차 마지막에 다음 Baseline을 만들었습니다.

```text
7일차 ONNX Model
        ↓
Raspberry Pi
        ↓
Model-only Benchmark
        ↓
End-to-End Benchmark
        ↓
Mean / P50 / P95 / FPS
        ↓
Stage Profile
        ↓
CPU / Memory / Temperature
        ↓
reports/day08_performance.md
```

8일차의 목적은 **측정**이었습니다.

9일차는 이 결과를 새로 버리지 않고 그대로 출발점으로 사용합니다.

```text
8일차 Baseline
        ↓
9일차 오늘 Baseline 재확인
        ↓
Frame Skip
        ↓
Voting
        ↓
Debounce
        ↓
Hold
        ↓
Resource Monitoring
        ↓
Candidate A / B / C
        ↓
최종 운영 Config
```

오늘도 `models/day07_tiny_cnn.onnx`, `src/ai_preprocess.py`, `src/inference_onnx.py`, `src/camera_input.py`, `src/decision.py`, `src/gpio_output.py`, `configs/settings.yaml`, `configs/device.local.yaml` 등을 **이전 일차에서 만든 파일로 다시 사용**합니다. 새로 만드는 것처럼 다시 작성하지 않습니다.

---

## 9일차 결과는 10일차에 어떻게 연결될까요?

9일차가 끝나면 Raspberry Pi는 다음 상태가 됩니다.

```text
Camera
→ ONNX AI
→ Operational Decision
→ Frame Skip
→ Majority Voting
→ Debounce
→ Hold
→ GPIO
→ Optimization / Resource Log
```

하지만 아직 사람이 SSH로 들어가 다음 명령을 직접 실행해야 합니다.

```bash
python -m scripts.optimized_ai_warning_system
```

10일차에는 이 **9일차 최종 Pipeline과 운영 Config를 그대로 사용**하면서 실행 방식만 운영형으로 바꿉니다.

```text
9일차 Final Pipeline + Final Config
        ↓
10일차 Headless Edge Runtime
        ↓
systemd
        ↓
Boot 자동실행
        ↓
비정상 종료 시 자동재시작
        ↓
Runtime State
        ↓
FastAPI
        ↓
PC에서 상태 확인
```

9일차에서 선택한 `inference_every_n_frames`, `vote_window`, `vote_warning_min`, `debounce_required`, `warning_hold_sec`가 10일차의 **운영 Baseline**이 됩니다.

---

## 오늘의 학습 목표

수업이 끝나면 다음을 할 수 있어야 합니다.

```text
1. 8일차 Baseline을 다시 확인하고 오늘 실험의 기준값을 기록한다.
2. Frame Skip이 Model Latency를 줄이는 기술이 아니라 추론 호출 빈도를 줄이는 방법임을 설명한다.
3. Majority Voting / Debounce / Hold의 역할을 서로 구분한다.
4. vote_window가 최근 Camera Frame이 아니라 최근 AI 판단 개수라는 점을 설명한다.
5. Stabilizer를 Camera 없이 먼저 검증한다.
6. Frame Skip과 Stabilizer를 실제 Camera ONNX Pipeline에 연결한다.
7. 같은 60초 Protocol에서 각 설정의 결과를 비교한다.
8. Inference/sec와 상태 변화 횟수를 함께 해석한다.
9. 단순 변화 횟수뿐 아니라 100 Inference당 변화율을 확인한다.
10. CPU / Memory / Temperature / Throttling을 장시간 기록한다.
11. Camera 해상도·조명·빠른 Event가 운영 안정성에 미치는 영향을 확인한다.
12. 속도·안정성·반응성 사이의 Trade-off를 설명한다.
13. Candidate A/B/C를 같은 조건으로 비교하고 최종 Config를 선택한다.
14. 실행 결과가 어느 파일의 어느 코드에서 만들어졌는지 역추적한다.
15. 오류를 Config / Camera / Runtime / Resource / Stabilization 영역으로 나누어 복구한다.
16. Mini Challenge 후 10일차가 사용할 깨끗한 Day 09 Final Baseline으로 복원한다.
```

---

## 오늘의 8시간 학습 흐름

| 학습 블록 | 권장 시간 | 무엇을 배우나요? | 끝나면 할 수 있어야 하는 것 |
|---|---:|---|---|
| 1 | 45분 | 8일차 Handoff · 오늘 Baseline 재측정 | 비교 기준을 먼저 고정할 수 있다 |
| 2 | 60분 | Frame Skip · Voting · Debounce · Hold 개념 | 각 최적화가 무엇을 바꾸는지 설명할 수 있다 |
| 3 | 75분 | Stabilizer Guided Lab | Camera 없이 시간적 안정화 로직을 검증할 수 있다 |
| 4 | 90분 | 실제 Camera + ONNX 최적화 실험 | Frame Skip과 안정화를 실제 Pipeline에서 비교할 수 있다 |
| 5 | 60분 | Resource Monitoring · Stress | 장시간 CPU/Memory/Temperature 상태를 기록할 수 있다 |
| 6 | 55분 | Candidate 비교 · 운영 Config 선택 | 속도·안정성·반응성을 근거로 설정을 선택할 수 있다 |
| 7 | 65분 | Mini Challenge · 오류 복구 · Log 증명 | 결과 역추적부터 전체 Pipeline 설명까지 스스로 수행할 수 있다 |
| 8 | 30분 | 복원 · Report · README · Git · 복습 | 10일차에 넘길 Final Baseline을 완성할 수 있다 |

총 480분을 기준으로 합니다. 실제 Camera 연결 상태와 Raspberry Pi 속도에 따라 각 블록의 시간은 조금 달라질 수 있습니다.

---

## 오늘도 계속 기억할 기본 Pipeline

1일차부터 사용한 구조는 그대로입니다.

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

9일차에서는 다음처럼 확장됩니다.

```text
입력
→ Camera Frame

처리
→ Preprocess
→ 필요 Frame에서 ONNX Inference

판단 1
→ Class + Confidence
→ Raw NORMAL / WARNING

판단 2
→ Voting
→ Debounce
→ Hold
→ Final NORMAL / WARNING

출력
→ Green / Red LED
→ 선택 Buzzer

기록
→ Optimization Summary
→ Resource CSV
→ Report
```

코드를 보다가 헷갈리면 다음 두 질문으로 돌아옵니다.

> **“지금 보고 있는 코드는 입력·처리·판단·출력·기록 중 어느 역할인가?”**  
> **“이 설정을 바꾸면 Model 자체가 바뀌는가, 아니면 운영 방식만 바뀌는가?”**

---

## 오늘 사용할 실제 장비와 이전 일차 결과

```text
Raspberry Pi
→ Raspberry Pi OS / Linux
→ USB Camera
→ Green / Red LED
→ 선택 Buzzer

7일차에서 만든 AI
→ models/day07_tiny_cnn.onnx
→ models/day07_tiny_cnn.meta.json

8일차에서 만든 성능 도구
→ src/performance.py
→ scripts/benchmark_model_only.py
→ scripts/benchmark_e2e.py
→ scripts/profile_e2e_stages.py
→ reports/day08_performance.md
```

오늘 새로 추가하는 핵심 파일:

```text
src/stabilizer.py
src/optimization_logger.py
src/resource_monitor.py

scripts/stabilizer_demo.py
scripts/optimized_ai_warning_system.py
scripts/resource_monitor.py

reports/day09_optimization.md
reports/day09_optimization_runs.csv
```

---

## 오늘 배울 기술 스택

| 기술 | 오늘 어디에 사용하나요? |
|---|---|
| ONNX Runtime | 이전 일차 Tiny CNN 실시간 추론 재사용 |
| OpenCV / CameraInput | 실제 Camera Frame 입력 |
| Python `deque` | 최근 AI 상태 Window 저장 |
| Majority Voting | 최근 여러 판단을 하나의 후보 상태로 통합 |
| Debounce | 상태 전환 후보가 일정 횟수 지속되는지 확인 |
| `time.monotonic()` | WARNING Hold 시간 계산 |
| Frame Skip | AI 추론 호출 빈도 조절 |
| `psutil` | CPU / Memory 상태 확인 |
| Linux Thermal 정보 | Temperature 확인 |
| `vcgencmd` | 지원되는 Raspberry Pi 환경의 Throttling 확인 |
| CSV | 최적화 Run / Resource 상태 기록 |
| Git | Source · Config · Report Version 관리 |

---

## 실습 전에 자신의 환경값 적어두기

```text
Raspberry Pi 사용자 이름      : ______________________________
Raspberry Pi Hostname         : ______________________________
Raspberry Pi IPv4             : ______________________________
Raspberry Pi Architecture     : ______________________________
Raspberry Pi Python           : ______________________________
Raspberry Pi ORT Version      : ______________________________

Camera Index                  : ______________________________
Camera 기본 Resolution        : ______________________________
Model Input Size              : ______________________________
ONNX Model                    : ______________________________
use_buzzer                    : true / false

Day 08 Model-only Mean        : ______________________________
Day 08 Model-only P95         : ______________________________
Day 08 End-to-End Mean        : ______________________________
Day 08 End-to-End P95         : ______________________________
Day 08 End-to-End FPS         : ______________________________
Day 08 가장 큰 Stage          : ______________________________
```

교재에서는 다음 표기법을 사용합니다.

```text
<RPI_USER>
→ 자신의 Raspberry Pi 사용자 이름

<RPI_IP>
→ 현재 확인한 Raspberry Pi IPv4
```

IP가 달라졌다면 예전 값을 계속 사용하지 않습니다.

```bash
hostname
hostname -I
```

Camera index도 이전 값을 무조건 믿지 않고 문제가 생기면 다시 확인합니다.

```bash
python -m scripts.camera_index_probe
```

---

## 오늘의 전체 실습 흐름

```text
8일차 Report 확인
        ↓
오늘 Baseline 재측정
        ↓
Day 09 Report 시작
        ↓
Frame Skip 이해
        ↓
Stabilizer 설정 추가
        ↓
TemporalStabilizer
        ↓
Camera 없는 Guided Demo
        ↓
Voting 비교
        ↓
Debounce
        ↓
Hold
        ↓
Optimization Logger
        ↓
실제 Camera + ONNX
        ↓
Frame Skip 1 / 2 / 3
        ↓
Voting 3 / 5 / 7
        ↓
Hold 0 / 1 / 2초
        ↓
조합 실험
        ↓
Resource Monitor
        ↓
5분 Stress
        ↓
Camera / 조명 / 짧은 Event Stress
        ↓
Candidate A / B / C
        ↓
Final Operating Config 선택
        ↓
Guided Failure
        ↓
Mini Challenge
        ↓
Challenge 상태 복원
        ↓
선택 심화: Precision / INT8
        ↓
Report / README / Git
        ↓
10일차 Headless 운영으로 연결
```

---

## 오늘의 성공 기준

```text
8일차 Baseline 확인
        ↓
오늘 Baseline 재측정
        ↓
Frame Skip 효과와 약점 설명
        ↓
Voting / Debounce / Hold 구분
        ↓
Stabilizer Demo 정상
        ↓
실제 Camera + ONNX + Stabilizer 정상
        ↓
Optimization CSV 생성
        ↓
Resource CSV 생성
        ↓
Candidate 비교표 완성
        ↓
Final Config 선택 이유 설명
        ↓
Mini Challenge 완료
        ↓
Challenge 복원
        ↓
최종 Run PASS
        ↓
README / Report / Git
        ↓
10일차 Handoff 준비
```

---

## 오늘은 아직 하지 않는 것

```text
새 AI 모델 학습
→ 6일차에서 완료

ONNX 배포 구조 새로 만들기
→ 7일차에서 완료

정식 Latency / FPS 측정 설계
→ 8일차에서 완료

systemd 자동실행 / 자동재시작
→ 10일차

FastAPI 상태 조회
→ 10일차
```

오늘은 **“잘 실행되는 AI를 실제 운영에서 덜 흔들리고, 필요한 만큼만 계산하도록 조정하는 날”**입니다.

---

# PART A. 8일차 Baseline을 이어받아 오늘의 비교 기준 만들기


# 1. 8일차 결과를 먼저 다시 확인하기

Raspberry Pi에 접속합니다.

```bash
ssh <RPI_USER>@<RPI_IP>
```

사용자 이름과 IP는 자신의 장비에 맞게 변경합니다.

프로젝트로 이동합니다.

```bash
cd ~/ai_vision/subject13_edge_ai
```

최신 Source를 확인하는 방법은 수업 환경에 따라 다릅니다.

### 허용된 내부 GitLab / Gitea Remote가 있는 경우

먼저 Remote가 연결되어 있는지 확인합니다.

```bash
git remote -v
```

Remote가 정상이고 교육장 정책상 사용이 허용되어 있다면:

```bash
git pull
```

### Local Git만 사용하는 경우

Remote가 없다면 `git pull`을 억지로 실행하지 않습니다.  
8일차에서 사용한 Source가 현재 Raspberry Pi에 있는지 `git status`, `git log`와 실제 파일로 확인합니다.

```bash
git status
git log --oneline -10
```

PC에서 수정한 파일을 가져와야 한다면 수업에서 정한 SCP 또는 내부 파일 전달 방식을 사용합니다.

> `configs/device.local.yaml`은 Camera index 같은 **Raspberry Pi 장치 전용 설정**이므로 PC의 공통 파일로 덮어쓰지 않습니다.

가상환경:

```bash
source .venv/bin/activate
```

확인:

```bash
which python
```

8일차 Report:

```bash
cat reports/day08_performance.md
```

8일차 성능 Summary가 있다면 확인합니다.

```bash
find reports/day08 \
  -type f \
  -name "*summary.csv" \
  -maxdepth 1
```

오늘은 새로운 최적화를 시작하기 전에 **현재 Baseline 숫자**를 먼저 적어 둡니다.

```text
Model Input:

Model-only Mean:

Model-only P95:

End-to-End Mean:

End-to-End P95:

End-to-End FPS:

가장 느린 Stage:

CPU / Memory:

Temperature:
```

---

# 2. 오늘의 Baseline을 다시 한 번 측정하기

성능 비교는 이전 날 숫자를 그대로 믿기보다 같은 날 같은 조건에서 Baseline을 한 번 더 측정하는 것이 좋습니다.

기본 설정을 확인하기 전에 8일차 마지막에 복원한 Baseline을 기억합니다.

```yaml
benchmark_warmup: 20
benchmark_repeat: 100
benchmark_include_gpio: false
benchmark_include_log: true

onnx_model_path: models/day07_tiny_cnn.onnx
onnx_meta_path: models/day07_tiny_cnn.meta.json
ai_warning_class: warning_red
ai_confidence_threshold: 0.70
```

Camera 해상도는 8일차에 실제로 검증한 기본값을 사용합니다. 기본 실습이 `640 × 480`이었다면 그 값으로 시작합니다.  
`camera_index`는 `configs/device.local.yaml`에 있는 **현재 Raspberry Pi의 실제 값**을 유지합니다.

기본 설정을 확인합니다.

```bash
grep -E \
"benchmark_warmup|benchmark_repeat|image_width|image_height|ai_confidence_threshold" \
configs/settings.yaml
```

Model-only:

```bash
python -m scripts.benchmark_model_only \
  --image data/deploy_samples/normal.jpg \
  --label day09_baseline
```

End-to-End:

```bash
python -m scripts.benchmark_e2e \
  --label day09_baseline
```

8일차 결과와 오늘 다시 잰 Baseline이 완전히 같을 필요는 없습니다.  
온도, Background Process, Camera 상태가 달라지면 수치도 달라질 수 있습니다.

중요한 것은 오늘의 모든 최적화 실험을 **오늘 다시 측정한 Baseline과 같은 조건으로 비교**하는 것입니다.

Stage Profile:

```bash
python -m scripts.profile_e2e_stages
```

오늘의 Baseline을 `reports/day09_optimization.md`에 기록합니다.

---

# 3. 9일차 Report 파일 먼저 만들기

파일:

```text
reports/day09_optimization.md
```

내용:

```md
# Day 09 Edge AI Optimization

## Baseline

Model:

Model Input:

Camera Resolution:

Confidence Threshold:

Model-only Mean:

Model-only P95:

End-to-End Mean:

End-to-End P95:

End-to-End FPS:

가장 큰 Stage:

CPU:

Memory:

Temperature:

## 오늘 변경할 것

- Frame Skip
- Majority Voting
- Debounce
- Hold
- Resource Monitoring

## 최종 선택

Frame Skip:

Vote Window:

Warning Minimum:

Debounce:

Hold:

선택 이유:
```

이 문서는 실험이 끝날 때까지 계속 채웁니다.

---


# PART B. 계산 빈도와 시간적 안정화의 원리 이해하기


# 4. Frame Skip을 적용하기 전에 의미를 정확히 정하기

오늘의 `Frame Skip`은 다음처럼 정의합니다.

```text
inference_every_n_frames = 1

Frame 1 → AI
Frame 2 → AI
Frame 3 → AI
Frame 4 → AI
```

```text
inference_every_n_frames = 2

Frame 1 → AI
Frame 2 → Skip
Frame 3 → AI
Frame 4 → Skip
```

```text
inference_every_n_frames = 3

Frame 1 → AI
Frame 2 → Skip
Frame 3 → Skip
Frame 4 → AI
```

Camera Frame은 계속 읽지만 AI 추론 횟수를 줄입니다.

---

# 5. Frame Skip에서 생기는 Trade-off 생각하기

장점:

```text
Inference 횟수 감소
→ CPU 부담 감소 가능
→ 처리 여유 증가 가능
```

단점:

```text
짧게 나타난 위험상황
→ Skip된 Frame에만 존재
→ 놓칠 가능성
```

따라서:

```text
Frame Skip을 크게 할수록 무조건 좋다.
X
```

실제 속도와 반응성을 같이 봐야 합니다.

---

# 6. 9일차 최적화 설정 추가하기

`configs/settings.yaml`에 다음 항목을 추가합니다.

```yaml
inference_every_n_frames: 1

vote_window: 5
vote_warning_min: 3

debounce_required: 2
warning_hold_sec: 1.0

optimization_duration_sec: 60

resource_log_path: logs/day09_resource.csv
resource_sample_sec: 2.0
resource_monitor_duration_sec: 300

optimization_result_path: reports/day09_optimization_runs.csv
```

3일차부터 사용한 다음 장치 설정은 새로 만들지 않습니다.

```yaml
camera_index: <실제 장치 값>
use_buzzer: false   # 실제 Buzzer를 사용할 때만 true
```

`camera_index`는 가능하면 `configs/device.local.yaml`의 실제 장치값을 사용하고, 공통 `settings.yaml`로 덮어쓰지 않습니다.

각 값의 의미:

```text
inference_every_n_frames
→ 몇 Frame마다 AI 추론할 것인가?

vote_window
→ 최근 몇 개 AI 결과를 볼 것인가?

vote_warning_min
→ 최근 결과 중 WARNING이 몇 개 이상이면 WARNING 후보인가?

debounce_required
→ 같은 후보 상태가 몇 번 연속 확인되어야 실제 상태를 바꿀 것인가?

warning_hold_sec
→ WARNING이 확정된 뒤 최소 유지 시간

optimization_duration_sec
→ 한 번의 Live 최적화 실험 시간
```

---

# 7. 시간적 안정화 모듈 만들기

## 이 파일은 왜 필요할까요?

7일차 AI는 한 Frame마다 `NORMAL` 또는 `WARNING`을 냅니다. 경계 상황에서는 연속 Frame이 다음처럼 흔들릴 수 있습니다.

```text
NORMAL
WARNING
NORMAL
WARNING
WARNING
```

모델을 다시 학습하지 않아도 최근 여러 AI 판단을 시간 순서로 모아 **운영 출력 상태를 안정화**할 수 있습니다.

`src/stabilizer.py`는 다음 세 역할을 한 곳에 모읍니다.

```text
최근 AI 판단 모으기
→ Majority Voting
→ Debounce
→ WARNING Hold
```

## 의사코드

```text
설정값 검증
        ↓
최근 AI 판단을 저장할 deque 준비
        ↓
새 raw_state가 들어오면 최근 Window에 추가
        ↓
Window 안 WARNING 개수 계산
        ↓
Warning Minimum 이상이면 WARNING 후보
아니면 NORMAL 후보
        ↓
같은 후보가 Debounce 횟수만큼 이어지는지 확인
        ↓
조건을 만족하면 stable_state 변경
        ↓
WARNING이 확정되면 Hold 종료시각 계산
        ↓
Hold 시간 안이면 WARNING 출력 유지
        ↓
raw / voted / stable / output 상태 반환
```

파일:

```text
src/stabilizer.py
```

코드:

```python
from collections import deque


class TemporalStabilizer:
    def __init__(
        self,
        vote_window: int,
        vote_warning_min: int,
        debounce_required: int,
        warning_hold_sec: float,
    ):
        if vote_window < 1:
            raise ValueError(
                "vote_window는 1 이상이어야 합니다."
            )

        if not (
            1
            <= vote_warning_min
            <= vote_window
        ):
            raise ValueError(
                "vote_warning_min은 "
                "1~vote_window 범위여야 합니다."
            )

        if debounce_required < 1:
            raise ValueError(
                "debounce_required는 "
                "1 이상이어야 합니다."
            )

        if warning_hold_sec < 0:
            raise ValueError(
                "warning_hold_sec는 "
                "0 이상이어야 합니다."
            )

        self.votes = deque(
            maxlen=vote_window
        )

        self.vote_warning_min = (
            vote_warning_min
        )

        self.debounce_required = (
            debounce_required
        )

        self.warning_hold_sec = (
            warning_hold_sec
        )

        self.candidate_state = None
        self.candidate_count = 0

        self.stable_state = "NORMAL"
        self.hold_until = 0.0

    def update(
        self,
        raw_state: str,
        now_sec: float,
    ) -> dict:
        self.votes.append(
            raw_state
        )

        warning_count = sum(
            1
            for state in self.votes
            if state == "WARNING"
        )

        if (
            warning_count
            >= self.vote_warning_min
        ):
            voted_state = "WARNING"
        else:
            voted_state = "NORMAL"

        if (
            voted_state
            != self.candidate_state
        ):
            self.candidate_state = (
                voted_state
            )
            self.candidate_count = 1
        else:
            self.candidate_count += 1

        if (
            self.candidate_count
            >= self.debounce_required
        ):
            self.stable_state = (
                self.candidate_state
            )

        if self.stable_state == "WARNING":
            self.hold_until = max(
                self.hold_until,
                now_sec
                + self.warning_hold_sec,
            )

        if now_sec < self.hold_until:
            output_state = "WARNING"
        else:
            output_state = (
                self.stable_state
            )

        return {
            "raw_state": raw_state,
            "warning_count":
                warning_count,
            "vote_count":
                len(self.votes),
            "voted_state":
                voted_state,
            "candidate_state":
                self.candidate_state,
            "candidate_count":
                self.candidate_count,
            "stable_state":
                self.stable_state,
            "output_state":
                output_state,
        }
```

---

### 중요한 해석 — `vote_window`는 Camera Frame 개수가 아닐 수 있습니다

9일차 Live 프로그램에서는 Stabilizer가 **AI 추론을 수행한 시점에만** `update()`됩니다.

예를 들어:

```text
Camera 30 FPS
inference_every_n_frames = 3
```

이라면 Camera Frame은 약 30개/초 들어오더라도 Stabilizer에는 약 10개의 AI 판단/초가 들어갈 수 있습니다.

따라서:

```text
vote_window: 5
```

는 **최근 5개의 AI 판단 결과**라고 이해합니다.

Frame Skip 값을 바꾸면 같은 `vote_window: 5`라도 실제 시간 폭은 달라질 수 있습니다. 이것이 뒤에서 Frame Skip과 안정화를 함께 비교해야 하는 이유입니다.

# 8. 안정화 로직을 Camera 없이 먼저 테스트하기

실제 Camera를 바로 사용하면 **AI Prediction이 흔들린 것인지 Stabilizer가 잘못 동작한 것인지** 구분하기 어렵습니다.

먼저 가상의 상태 배열을 사용하여 Stabilizer만 독립적으로 확인합니다.

## 이 파일은 왜 필요할까요?

`stabilizer_demo.py`는 Camera, ONNX, GPIO를 모두 제외하고 `TemporalStabilizer`의 동작만 눈으로 확인하는 작은 테스트 프로그램입니다.

## 의사코드

```text
미리 정한 NORMAL / WARNING 배열 준비
        ↓
settings.yaml에서 Voting / Debounce / Hold 읽기
        ↓
TemporalStabilizer 생성
        ↓
상태를 하나씩 update()
        ↓
raw / warning_count / vote / stable / output 출력
        ↓
설정값을 하나씩 바꾸어 결과 비교
```

먼저 가상의 상태 배열을 사용합니다.

파일:

```text
scripts/stabilizer_demo.py
```

코드:

```python
import time

from src.config_loader import load_config
from src.stabilizer import (
    TemporalStabilizer,
)


RAW_SEQUENCE = [
    "NORMAL",
    "NORMAL",
    "WARNING",
    "NORMAL",
    "WARNING",
    "WARNING",
    "WARNING",
    "NORMAL",
    "WARNING",
    "NORMAL",
    "NORMAL",
    "NORMAL",
]


def main():
    config = load_config(
        "configs/settings.yaml"
    )

    stabilizer = TemporalStabilizer(
        vote_window=int(
            config["vote_window"]
        ),
        vote_warning_min=int(
            config["vote_warning_min"]
        ),
        debounce_required=int(
            config["debounce_required"]
        ),
        warning_hold_sec=float(
            config["warning_hold_sec"]
        ),
    )

    print("=== Stabilizer Demo ===")
    print(
        f"Vote Window    : "
        f"{config['vote_window']}"
    )
    print(
        f"Warning Min    : "
        f"{config['vote_warning_min']}"
    )
    print(
        f"Debounce       : "
        f"{config['debounce_required']}"
    )
    print(
        f"Hold           : "
        f"{config['warning_hold_sec']} sec"
    )
    print()

    for index, raw_state in enumerate(
        RAW_SEQUENCE,
        start=1,
    ):
        now = time.monotonic()

        result = stabilizer.update(
            raw_state,
            now,
        )

        print(
            f"{index:02d} | "
            f"raw={raw_state:7s} | "
            f"warning="
            f"{result['warning_count']}/"
            f"{result['vote_count']} | "
            f"vote="
            f"{result['voted_state']:7s} | "
            f"stable="
            f"{result['stable_state']:7s} | "
            f"output="
            f"{result['output_state']:7s}"
        )

        time.sleep(0.2)


if __name__ == "__main__":
    main()
```

---

# 9. 최근 5개 중 3개 WARNING 다수결 실행하기

설정:

```yaml
vote_window: 5
vote_warning_min: 3
debounce_required: 1
warning_hold_sec: 0.0
```

실행:

```bash
python -m scripts.stabilizer_demo
```

가상 입력:

```text
NORMAL
NORMAL
WARNING
NORMAL
WARNING
WARNING
WARNING
...
```

각 시점의 다음 값을 봅니다.

```text
raw
warning_count
vote
stable
output
```

최근 5개 중 WARNING이 3개 이상인 순간부터 `voted_state`가 WARNING으로 바뀝니다.

---

# 10. Voting Window를 3 / 5 / 7로 바꾸어 보기

한 번에 `vote_window`만 바꿉니다.

실험 A:

```yaml
vote_window: 3
vote_warning_min: 2
```

실험 B:

```yaml
vote_window: 5
vote_warning_min: 3
```

실험 C:

```yaml
vote_window: 7
vote_warning_min: 4
```

각각 실행합니다.

```bash
python -m scripts.stabilizer_demo
```

관찰:

```text
Window가 작음
→ 빠르게 반응
→ 흔들림에 더 민감할 수 있음

Window가 큼
→ 안정적일 수 있음
→ 반응이 늦어질 수 있음
```

---

# 11. Debounce를 적용해 보기

Voting 결과도 한 번만 바뀌었다고 즉시 출력 상태를 바꾸지 않고 일정 횟수 연속 확인할 수 있습니다.

설정:

```yaml
vote_window: 5
vote_warning_min: 3
debounce_required: 2
warning_hold_sec: 0.0
```

실행:

```bash
python -m scripts.stabilizer_demo
```

`candidate_count`가 `2` 이상이 되어야 `stable_state`가 바뀝니다.

이것이 오늘 사용하는 간단한 Debounce입니다.

---

# 12. Hold를 적용해 보기

설정:

```yaml
vote_window: 5
vote_warning_min: 3
debounce_required: 2
warning_hold_sec: 1.0
```

실행:

```bash
python -m scripts.stabilizer_demo
```

WARNING이 한 번 확정되면 최근 결과가 NORMAL 쪽으로 바뀌더라도 최소 시간 동안 WARNING 출력이 유지될 수 있습니다.

목적:

```text
경계에서
Red LED / Buzzer가
짧게 켜졌다 꺼지는 현상 감소
```

---

# 13. Voting·Debounce·Hold의 역할을 구분하기

```text
Majority Voting
→ 최근 여러 판단을 모아 결정

Debounce
→ 상태 전환 후보가 일정 횟수 계속되어야 변경

Hold
→ WARNING 확정 후 최소 시간 유지
```

세 기능은 비슷해 보이지만 역할이 다릅니다.

---


# PART C. 실제 Camera · ONNX Pipeline에 Frame Skip과 안정화 연결하기


# 14. Live 최적화 결과를 저장할 Logger 만들기

## 이 파일은 왜 필요할까요?

Frame Skip, Voting, Debounce, Hold를 여러 조합으로 실험하면 터미널 출력만으로는 비교하기 어렵습니다.

`src/optimization_logger.py`는 각 실험을 **한 줄의 Summary**로 남겨 나중에 같은 표에서 비교할 수 있게 합니다.

특히 Frame Skip을 바꾸면 전체 추론 횟수가 달라지므로 단순 `Raw Changes` 개수만 비교하면 오해할 수 있습니다. 그래서 **100회 추론당 상태변화 횟수**도 함께 저장합니다.

## 의사코드

```text
실험 이름과 실행 결과를 받는다
        ↓
전체 Frame / Inference 수 계산
        ↓
Camera Loop FPS 계산
        ↓
Inference/sec 계산
        ↓
100 Inference당 Raw 변화율 계산
        ↓
100 Inference당 Output 변화율 계산
        ↓
설정값과 결과를 CSV 한 줄로 저장
```

파일:

```text
src/optimization_logger.py
```

코드:

```python
import csv
from pathlib import Path


class OptimizationRunLogger:
    def __init__(
        self,
        path: str,
    ):
        self.path = Path(path)

        self.path.parent.mkdir(
            parents=True,
            exist_ok=True,
        )

    def append(
        self,
        run_name: str,
        duration_sec: float,
        frames: int,
        inference_count: int,
        raw_changes: int,
        output_changes: int,
        frame_skip: int,
        vote_window: int,
        warning_min: int,
        debounce: int,
        hold_sec: float,
    ):
        is_new = not self.path.exists()

        camera_loop_fps = (
            frames / duration_sec
            if duration_sec > 0
            else 0.0
        )

        inference_per_sec = (
            inference_count / duration_sec
            if duration_sec > 0
            else 0.0
        )

        raw_change_per_100 = (
            raw_changes / inference_count * 100.0
            if inference_count > 0
            else 0.0
        )

        output_change_per_100 = (
            output_changes / inference_count * 100.0
            if inference_count > 0
            else 0.0
        )

        with self.path.open(
            "a",
            newline="",
            encoding="utf-8",
        ) as file:
            writer = csv.writer(file)

            if is_new:
                writer.writerow(
                    [
                        "run_name",
                        "duration_sec",
                        "frames",
                        "inference_count",
                        "camera_loop_fps",
                        "inference_per_sec",
                        "raw_state_changes",
                        "output_state_changes",
                        "raw_changes_per_100_inferences",
                        "output_changes_per_100_inferences",
                        "inference_every_n_frames",
                        "vote_window",
                        "warning_min",
                        "debounce",
                        "hold_sec",
                    ]
                )

            writer.writerow(
                [
                    run_name,
                    f"{duration_sec:.3f}",
                    frames,
                    inference_count,
                    f"{camera_loop_fps:.3f}",
                    f"{inference_per_sec:.3f}",
                    raw_changes,
                    output_changes,
                    f"{raw_change_per_100:.3f}",
                    f"{output_change_per_100:.3f}",
                    frame_skip,
                    vote_window,
                    warning_min,
                    debounce,
                    f"{hold_sec:.3f}",
                ]
            )
```

---

### 왜 `Raw Changes` 개수만 보면 안 될까요?

예를 들어 60초 동안:

```text
Frame Skip 1
→ Inference 1,000회
→ Raw Changes 20회

Frame Skip 3
→ Inference 330회
→ Raw Changes 10회
```

이면 단순히 `20 → 10`만 보고 “AI가 두 배 안정적”이라고 결론 내리면 안 됩니다.  
두 번째 실험은 **AI를 본 횟수 자체가 훨씬 적기 때문**입니다.

그래서 같은 시간·같은 동작 Protocol을 유지하고 다음을 함께 봅니다.

```text
Inference/sec
Raw Changes
Output Changes
Raw Changes / 100 Inferences
Output Changes / 100 Inferences
놓친 짧은 WARNING
```

# 15. Frame Skip + 안정화를 적용한 실시간 프로그램 만들기

## 이 파일은 왜 필요할까요?

7일차의 실시간 AI Warning System에 오늘 배운 **Frame Skip + Temporal Stabilizer + 실험 Summary Log**를 실제로 연결합니다.

이 파일의 목표는 Model 자체를 다시 만드는 것이 아니라 **Model을 언제 호출하고, 여러 Prediction을 어떻게 운영 출력으로 만들 것인지** 실험하는 것입니다.

## 실행 전에 파일 연결 확인하기

```text
configs/settings.yaml
configs/device.local.yaml
        ↓
scripts/optimized_ai_warning_system.py
        │
        ├─ src/camera_input.py          ← 이전 일차 재사용
        ├─ src/ai_preprocess.py         ← 7일차 재사용
        ├─ src/inference_onnx.py        ← 7일차 재사용
        ├─ src/decision.py              ← 7일차 재사용
        ├─ src/gpio_output.py           ← 3일차 재사용
        ├─ src/stabilizer.py            ← 오늘 새로 추가
        └─ src/optimization_logger.py   ← 오늘 새로 추가
        ↓
Camera Loop
→ 선택 Frame에서만 AI
→ Raw NORMAL/WARNING
→ Voting / Debounce / Hold
→ Final Output
→ GPIO
→ reports/day09_optimization_runs.csv
```

## 의사코드

```text
Config와 실행 인자를 읽는다
        ↓
Duration / Frame Skip 설정 검증
        ↓
Camera / ONNX / GPIO 준비
        ↓
TemporalStabilizer 준비
        ↓
실험 Summary Logger 준비
        ↓
Camera Frame 계속 읽기
        ↓
현재 Frame이 AI 추론 차례인가?
        ├─ YES
        │   → Preprocess
        │   → ONNX Prediction
        │   → Operational Decision
        │   → Stabilizer update
        │   → Final State 계산
        │
        └─ NO
            → 직전 Final State 유지
        ↓
GPIO에 Final State 출력
        ↓
일정 Frame마다 상태 출력
        ↓
Duration 종료
        ↓
Camera / GPIO 정리
        ↓
실험 Summary CSV 기록
```

파일:

```text
scripts/optimized_ai_warning_system.py
```

코드:

```python
import argparse
import time

from src.ai_preprocess import (
    bgr_frame_to_nchw,
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
from src.optimization_logger import (
    OptimizationRunLogger,
)
from src.stabilizer import (
    TemporalStabilizer,
)


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--run-name",
        default="default",
    )

    parser.add_argument(
        "--duration",
        type=float,
        default=None,
    )

    return parser.parse_args()


def main():
    args = parse_args()

    config = load_config(
        "configs/settings.yaml"
    )

    duration = (
        args.duration
        if args.duration is not None
        else float(
            config[
                "optimization_duration_sec"
            ]
        )
    )

    if duration <= 0:
        raise ValueError(
            "duration은 0보다 커야 합니다."
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
        index=int(config["camera_index"]),
        width=int(config["image_width"]),
        height=int(config["image_height"]),
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
            config["debounce_required"]
        ),
        warning_hold_sec=float(
            config["warning_hold_sec"]
        ),
    )

    run_logger = OptimizationRunLogger(
        config["optimization_result_path"]
    )

    warning_class = (
        config["ai_warning_class"]
    )

    confidence_threshold = float(
        config["ai_confidence_threshold"]
    )

    frame_count = 0
    inference_count = 0

    raw_changes = 0
    output_changes = 0

    previous_raw = None
    previous_output = None

    last_raw_state = "NORMAL"
    last_prediction = {
        "class_name": "normal",
        "confidence": 0.0,
    }

    started = time.perf_counter()

    try:
        print(
            "=== Optimized AI Warning System ==="
        )
        print(f"Run Name      : {args.run_name}")
        print(f"Duration      : {duration} sec")
        print(f"Inference N   : {every_n}")
        print(
            f"Vote          : "
            f"{config['vote_warning_min']}/"
            f"{config['vote_window']}"
        )
        print(
            f"Debounce      : "
            f"{config['debounce_required']}"
        )
        print(
            f"Hold          : "
            f"{config['warning_hold_sec']} sec"
        )
        print()

        time.sleep(1.0)

        while True:
            now = time.perf_counter()

            if (
                now - started
                >= duration
            ):
                break

            frame = camera.read()
            frame_count += 1

            should_infer = (
                (frame_count - 1)
                % every_n
                == 0
            )

            if should_infer:
                tensor = bgr_frame_to_nchw(
                    frame,
                    classifier.image_size,
                )

                prediction = (
                    classifier.predict(
                        tensor
                    )
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

                last_prediction = prediction
                last_raw_state = raw_state

                inference_count += 1

                if (
                    previous_raw is not None
                    and raw_state
                    != previous_raw
                ):
                    raw_changes += 1

                previous_raw = raw_state

                stabilized = (
                    stabilizer.update(
                        raw_state,
                        time.monotonic(),
                    )
                )

                final_state = (
                    stabilized[
                        "output_state"
                    ]
                )

                if (
                    previous_output is not None
                    and final_state
                    != previous_output
                ):
                    output_changes += 1

                previous_output = (
                    final_state
                )

            else:
                final_state = (
                    previous_output
                    or "NORMAL"
                )

            if final_state == "WARNING":
                output.warning()
            else:
                output.normal()

            if (
                frame_count == 1
                or frame_count % 30 == 0
            ):
                print(
                    f"frame="
                    f"{frame_count:5d} "
                    f"infer="
                    f"{inference_count:5d} "
                    f"class="
                    f"{last_prediction['class_name']:12s} "
                    f"conf="
                    f"{last_prediction['confidence']:.3f} "
                    f"raw="
                    f"{last_raw_state:7s} "
                    f"out="
                    f"{final_state:7s}"
                )

    except KeyboardInterrupt:
        print("\n사용자가 종료했습니다.")

    finally:
        elapsed = (
            time.perf_counter()
            - started
        )

        camera.release()
        output.close()

        run_logger.append(
            run_name=args.run_name,
            duration_sec=elapsed,
            frames=frame_count,
            inference_count=
                inference_count,
            raw_changes=raw_changes,
            output_changes=
                output_changes,
            frame_skip=every_n,
            vote_window=int(
                config["vote_window"]
            ),
            warning_min=int(
                config["vote_warning_min"]
            ),
            debounce=int(
                config["debounce_required"]
            ),
            hold_sec=float(
                config["warning_hold_sec"]
            ),
        )

        print()
        print("=== Run Summary ===")
        print(
            f"Duration      : "
            f"{elapsed:.2f} sec"
        )
        print(
            f"Frames        : "
            f"{frame_count}"
        )
        print(
            f"Inferences    : "
            f"{inference_count}"
        )
        print(
            f"Raw Changes   : "
            f"{raw_changes}"
        )
        print(
            f"Output Changes: "
            f"{output_changes}"
        )


if __name__ == "__main__":
    main()
```

---

### 중요한 해석 — Skip된 Frame에서는 무엇이 일어날까요?

`inference_every_n_frames = 3`이라고 해도 Camera를 3 Frame마다 한 번만 읽는 것이 아닙니다.

```text
Frame 1 → Camera Read → AI → Stabilizer
Frame 2 → Camera Read → AI Skip → 직전 Output 유지
Frame 3 → Camera Read → AI Skip → 직전 Output 유지
Frame 4 → Camera Read → AI → Stabilizer
```

따라서 Frame Skip은 **Camera 입력 자체를 버리는 최적화가 아니라, AI 추론 호출 빈도를 줄이는 실험**입니다.

또한 Buzzer는 이전 일차의 `use_buzzer` 설정을 따릅니다. 실제 Buzzer 사양과 배선이 검증되지 않았다면 `false`로 두고 LED만으로 필수 실습을 진행합니다.

# 16. 먼저 안정화 없이 Baseline Live Run 실행하기

설정:

```yaml
inference_every_n_frames: 1

vote_window: 1
vote_warning_min: 1

debounce_required: 1
warning_hold_sec: 0.0
```

실행:

```bash
python -m scripts.optimized_ai_warning_system \
  --run-name live_baseline \
  --duration 60
```

비교 실험의 공정성을 위해 **60초 동작 Protocol**을 먼저 고정합니다.

가능하면 Camera와 빨간 카드의 위치를 테이프로 표시하고, 조명과 거리를 바꾸지 않습니다. 한 학생은 타이머를 보고 구간을 알려주고 다른 학생은 같은 동작을 반복합니다.

60초 동안 다음 순서를 최대한 비슷하게 수행합니다.

```text
NORMAL 상태 15초

경계 상태 15초
→ 빨간 카드를 작게 / 일부만 보이기

명확한 WARNING 15초

다시 NORMAL 15초
```

결과:

```text
Raw Changes
Output Changes
```

를 기록합니다.

안정화가 없으므로 두 값이 비슷할 가능성이 큽니다.

---

### 관찰값의 성격도 구분합니다

```text
Frames / Inferences / Raw Changes / Output Changes
→ 프로그램이 기록한 정량값

체감 반응 / 놓친 짧은 WARNING
→ 오늘은 사람이 같은 Protocol에서 관찰한 정성값
```

정성 관찰을 `123 ms`처럼 정밀한 수치로 적지 않습니다.  
정확한 Reaction Time을 측정하려면 별도의 이벤트 기준과 계측 설계가 필요합니다.

# 17. Frame Skip 2 적용하기

다른 안정화는 끈 상태를 유지합니다.

```yaml
inference_every_n_frames: 2

vote_window: 1
vote_warning_min: 1

debounce_required: 1
warning_hold_sec: 0.0
```

실행:

```bash
python -m scripts.optimized_ai_warning_system \
  --run-name skip2 \
  --duration 60
```

같은 동작 순서를 반복합니다.

Report:

```md
## Frame Skip

| Every N Frames | Frames | Inferences | Inference/sec | Raw Changes | Output Changes | Raw /100 Inf | Output /100 Inf | 관찰 |
|---:|---:|---:|---:|---:|---:|---:|---:|---|
| 1 |  |  |  |  |  |  |
| 2 |  |  |  |  |  |  |
```

---

`Raw Changes`가 줄어도 Inference 횟수 자체가 줄었을 수 있으므로 CSV의 `raw_changes_per_100_inferences`도 함께 봅니다.

# 18. Frame Skip 3 적용하기

설정:

```yaml
inference_every_n_frames: 3
```

실행:

```bash
python -m scripts.optimized_ai_warning_system \
  --run-name skip3 \
  --duration 60
```

표에 추가합니다.

```md
| 3 |  |  |  |  |  |  |
```

다음 질문에 답합니다.

```text
Inference 횟수는 얼마나 줄었는가?

짧게 보여준 경고 물체를 놓친 적이 있는가?

출력 반응이 늦어진 느낌이 있는가?
```

---

# 19. Frame Skip이 Model 자체를 빠르게 만든 것은 아닙니다

Frame Skip은 ONNX 한 번의 추론시간을 줄이는 기술이 아닙니다.

```text
Model-only Latency
→ 거의 같은 모델

하지만

초당 Inference 횟수
→ 감소
```

즉:

```text
AI Model 자체가 빨라짐
X

AI를 호출하는 횟수를 줄임
O
```

이 차이를 구분합니다.

---

# 20. Majority Voting 3/5 적용하기

Frame Skip을 다시 `1`로 돌리고 Voting만 적용합니다.

```yaml
inference_every_n_frames: 1

vote_window: 5
vote_warning_min: 3

debounce_required: 1
warning_hold_sec: 0.0
```

실행:

```bash
python -m scripts.optimized_ai_warning_system \
  --run-name vote3of5 \
  --duration 60
```

경계 상황에서:

```text
Raw Changes
vs
Output Changes
```

를 비교합니다.

Voting이 효과가 있다면 `Output Changes`가 줄어들 수 있습니다.

---

# 21. Voting Window 3 / 5 / 7 비교하기

실험 A:

```yaml
vote_window: 3
vote_warning_min: 2
```

실험 B:

```yaml
vote_window: 5
vote_warning_min: 3
```

실험 C:

```yaml
vote_window: 7
vote_warning_min: 4
```

각각 60초 동안 같은 패턴으로 실험합니다.

Report:

```md
## Majority Voting

| Window | Warning Min | Raw Changes | Output Changes | 반응 속도 | 관찰 |
|---:|---:|---:|---:|---|---|
| 3 | 2 |  |  |  |  |
| 5 | 3 |  |  |  |  |
| 7 | 4 |  |  |  |  |
```

---

# 22. Debounce 2 적용하기

기본 Voting을:

```yaml
vote_window: 5
vote_warning_min: 3
```

로 두고:

```yaml
debounce_required: 2
warning_hold_sec: 0.0
```

실행합니다.

```bash
python -m scripts.optimized_ai_warning_system \
  --run-name debounce2 \
  --duration 60
```

관찰:

```text
한 번 튄 결과 때문에
바로 상태가 바뀌는 경우가 줄어드는가?

WARNING 확정이 약간 늦어지는가?
```

---

# 23. Hold 1초 적용하기

설정:

```yaml
vote_window: 5
vote_warning_min: 3
debounce_required: 2
warning_hold_sec: 1.0
```

실행:

```bash
python -m scripts.optimized_ai_warning_system \
  --run-name hold1 \
  --duration 60
```

경계에서 빨간색 카드를 넣었다 뺐다 하며 LED / Buzzer의 변화를 확인합니다.

Report:

```md
## Hold

| Hold | Output Changes | 체감 안정성 | 반응 지연 | 관찰 |
|---:|---:|---|---|---|
| 0 sec |  |  |  |  |
| 1 sec |  |  |  |  |
```

---

# 24. Hold 2초도 비교하기

설정:

```yaml
warning_hold_sec: 2.0
```

실행:

```bash
python -m scripts.optimized_ai_warning_system \
  --run-name hold2 \
  --duration 60
```

WARNING이 끝난 뒤에도 Red LED가 오래 유지되는지 봅니다.

안정성은 좋아 보일 수 있지만 **정상 복귀가 느려질 수 있습니다.**

```text
안정성 증가
↔
복귀 반응 지연
```

이것이 Trade-off입니다.

---

# 25. Frame Skip + Voting + Hold를 조합하기

이제 각 기능을 개별적으로 확인했으므로 조합합니다.

예:

```yaml
inference_every_n_frames: 2

vote_window: 5
vote_warning_min: 3

debounce_required: 2
warning_hold_sec: 1.0
```

실행:

```bash
python -m scripts.optimized_ai_warning_system \
  --run-name combined_v1 \
  --duration 60
```

같은 60초 동작 패턴을 사용합니다.

---

# 26. 조합 실험에서 무엇을 봐야 하나요?

다음 네 가지를 함께 봅니다.

```text
Inference/sec
→ 계산량

Raw Changes
→ AI 원시 판단 흔들림

Output Changes
→ 실제 장치 출력 흔들림

반응 체감
→ 위험 물체를 보여준 뒤 WARNING까지 지연
```

예:

```text
Frame Skip 증가
→ Inference/sec 감소

Voting 증가
→ Output Changes 감소 가능

Hold 증가
→ Output 안정
→ NORMAL 복귀 지연 가능
```

---

### 같은 숫자를 다른 의미로 해석하지 않기

Frame Skip을 바꾼 실험끼리는 추론 횟수가 달라집니다.  
따라서 단순 상태 변화 횟수뿐 아니라 `Inference/sec`와 `100 Inferences당 변화율`을 같이 봅니다.

Voting / Debounce / Hold만 비교할 때는 Frame Skip을 고정해야 안정화 효과를 더 분명하게 비교할 수 있습니다.


# 27. Guided Code Review — Live 결과를 어느 코드에서 만들었는지 역추적하기

다음과 같은 출력이 나왔다고 가정합니다.

```text
frame=  120 infer=   60 class=warning_red conf=0.914 raw=WARNING out=WARNING
```

결과만 보고 넘어가지 않고 실제 파일을 열어 한 항목씩 찾습니다.

```text
frame=120
→ scripts/optimized_ai_warning_system.py
→ frame_count += 1

infer=60
→ should_infer가 True일 때
→ inference_count += 1

class=warning_red / conf=0.914
→ src/inference_onnx.py의 ONNXClassifier.predict()

raw=WARNING
→ src/decision.py의 ai_to_operational_state()

out=WARNING
→ src/stabilizer.py의 TemporalStabilizer.update()
→ result["output_state"]

LED 상태
→ src/gpio_output.py
→ output.warning() / output.normal()

최종 Run Summary
→ scripts/optimized_ai_warning_system.py의 finally

CSV 한 줄
→ src/optimization_logger.py
→ OptimizationRunLogger.append()
```

직접 코드에서 다음 질문에 답합니다.

1. `Frame Skip` 여부를 계산하는 식은 어디에 있나요?
2. Skip된 Frame에서도 Camera는 읽나요?
3. Skip된 Frame에서는 Stabilizer의 `update()`가 호출되나요?
4. `raw_state`와 `final_state`는 같은 의미인가요?
5. `Output Changes`는 어느 조건에서 증가하나요?
6. `reports/day09_optimization_runs.csv`의 한 줄은 어느 함수가 저장하나요?

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
1. (frame_count - 1) % every_n == 0
2. 예. camera.read()는 매 Loop에서 호출됩니다.
3. 아니요. 현재 구조에서는 실제 AI 추론을 한 시점에만 update()합니다.
4. 다릅니다.
   raw_state = 단일 AI Prediction을 운영 상태로 바꾼 결과
   final_state = Voting / Debounce / Hold를 지난 실제 출력 상태
5. 이전 final_state와 새 final_state가 다를 때 증가합니다.
6. src/optimization_logger.py의 OptimizationRunLogger.append()입니다.
```

이 차이를 설명할 수 있어야 9일차의 수치를 올바르게 해석할 수 있습니다.

</details>

---


# PART D. 장시간 Resource 상태와 Stress 조건 확인하기


# 28. Resource Monitoring을 위한 Package 확인하기

Raspberry Pi 가상환경에서:

```bash
python -m pip show psutil
```

없다면 설치가 가능한 환경인지 먼저 확인합니다.

일반 인터넷 환경:

```bash
python -m pip install psutil
```

폐쇄망 또는 외부 Package 설치가 제한된 교육장에서는 **검증된 Offline Wheel 또는 허용된 내부 Package 저장소**를 사용합니다.  
인터넷이 되지 않는다고 임의 Architecture의 Wheel을 설치하지 않습니다.

확인:

```bash
python -c \
"import psutil; print(psutil.__version__)"
```

Raspberry Pi의 Package 기록도 갱신합니다.

```bash
python -m pip freeze \
  > reports/day09_pi_environment.txt
```

---

# 29. Resource Monitor 모듈 만들기

## 이 파일은 왜 필요할까요?

8일차에서는 Latency와 FPS를 측정했습니다. 9일차에서는 장시간 실행 중 **CPU, Memory, Temperature, Throttling 상태가 어떻게 변하는지** 함께 확인합니다.

장치 상태를 읽는 코드를 실행 Script 안에 모두 넣지 않고 `src/resource_monitor.py`에 모으면 다른 프로그램에서도 재사용할 수 있습니다.

## 의사코드

```text
CPU 사용률 읽기
        ↓
Memory 사용률 읽기
        ↓
Linux thermal_zone 목록 확인
        ↓
CPU / SoC와 관련된 Temperature 후보 읽기
        ↓
지원되면 vcgencmd get_throttled 실행
        ↓
지원되지 않으면 UNAVAILABLE 반환
```

파일:

```text
src/resource_monitor.py
```

코드:

```python
import subprocess
from pathlib import Path

import psutil


def get_cpu_percent() -> float:
    return float(
        psutil.cpu_percent(
            interval=None
        )
    )


def get_memory_percent() -> float:
    return float(
        psutil.virtual_memory().percent
    )


def _read_zone_temperature(
    zone: Path,
) -> tuple[str, float] | None:
    temp_path = zone / "temp"

    if not temp_path.exists():
        return None

    type_path = zone / "type"

    try:
        zone_type = (
            type_path.read_text(
                encoding="utf-8"
            ).strip()
            if type_path.exists()
            else zone.name
        )

        raw = float(
            temp_path.read_text(
                encoding="utf-8"
            ).strip()
        )
    except (OSError, ValueError):
        return None

    temperature = (
        raw / 1000.0
        if raw > 1000.0
        else raw
    )

    return zone_type, temperature


def get_temperature_c() -> float:
    root = Path(
        "/sys/class/thermal"
    )

    if not root.exists():
        return -1.0

    fallback = []

    for zone in sorted(
        root.glob("thermal_zone*")
    ):
        item = _read_zone_temperature(
            zone
        )

        if item is None:
            continue

        zone_type, temperature = item
        name = zone_type.lower()

        if (
            "cpu" in name
            or "soc" in name
        ):
            return temperature

        fallback.append(temperature)

    return (
        fallback[0]
        if fallback
        else -1.0
    )


def get_throttled_raw() -> str:
    try:
        result = subprocess.run(
            [
                "vcgencmd",
                "get_throttled",
            ],
            capture_output=True,
            text=True,
            check=True,
        )

        return result.stdout.strip()

    except (
        OSError,
        subprocess.CalledProcessError,
    ):
        return "UNAVAILABLE"
```

---

### Temperature 경로를 하나로 단정하지 않는 이유

Linux 장치에서는 Temperature 정보가 노출되는 `thermal_zone` 번호가 환경에 따라 달라질 수 있습니다.

먼저 직접 확인할 수도 있습니다.

```bash
for z in /sys/class/thermal/thermal_zone*; do
  echo "=== $z ==="
  cat "$z/type" 2>/dev/null
  cat "$z/temp" 2>/dev/null
done
```

따라서 `thermal_zone0`이 항상 CPU라고 가정하지 않고, 가능한 Zone의 `type`을 확인합니다.

`get_temperature_c()`가 `-1.0`을 반환하면 **“온도가 -1도”라는 뜻이 아니라 이 방법으로 Temperature를 얻지 못했다는 뜻**입니다.

# 30. Throttling 상태를 직접 확인하기

먼저 명령이 현재 OS에서 제공되는지 확인할 수 있습니다.

```bash
command -v vcgencmd || echo "vcgencmd unavailable"
```

지원되는 환경이라면:

```bash
vcgencmd get_throttled
```

예:

```text
throttled=0x0
```

`0x0`이라면 현재 또는 기록된 전원·Throttling 관련 Flag가 없는 정상 상태로 볼 수 있습니다.

다른 값이 나온다면 단순히 숫자를 지우거나 무시하지 않습니다.

```text
온도
전원
CPU 제한
```

상태를 함께 확인합니다.

사용 중인 Raspberry Pi 모델과 OS의 `vcgencmd` 해석 기준은 해당 장비 문서를 기준으로 확인합니다.

---

`vcgencmd`가 제공되지 않는 환경이라면 무조건 설치부터 하지 않습니다. 해당 Raspberry Pi OS 이미지와 장비에서 제공되는 진단 방법을 확인하고, 오늘 Report에는 `UNAVAILABLE`이라고 기록할 수 있습니다.

# 31. Resource Monitoring 프로그램 만들기

## 이 파일은 왜 필요할까요?

`src/resource_monitor.py`가 **한 번의 장치 상태를 읽는 함수**라면, `scripts/resource_monitor.py`는 일정 시간 동안 반복 측정하여 CSV로 남기는 실행 프로그램입니다.

## 의사코드

```text
Duration 인자와 Config 읽기
        ↓
Duration / Sampling Interval 검증
        ↓
CSV Header 작성
        ↓
CPU 측정 기준점 한 번 준비
        ↓
설정한 시간 동안 반복
        ↓
Timestamp / CPU / Memory / Temperature / Throttled 읽기
        ↓
CSV 한 줄 저장 + flush
        ↓
터미널에 현재 상태 출력
        ↓
Sampling Interval 대기
```

파일:

```text
scripts/resource_monitor.py
```

코드:

```python
import argparse
import csv
import time
from datetime import datetime
from pathlib import Path

from src.config_loader import load_config
from src.resource_monitor import (
    get_cpu_percent,
    get_memory_percent,
    get_temperature_c,
    get_throttled_raw,
)


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--duration",
        type=float,
        default=None,
    )

    return parser.parse_args()


def main():
    args = parse_args()

    config = load_config(
        "configs/settings.yaml"
    )

    duration = (
        args.duration
        if args.duration is not None
        else float(
            config[
                "resource_monitor_duration_sec"
            ]
        )
    )

    interval = float(
        config["resource_sample_sec"]
    )

    if duration <= 0:
        raise ValueError(
            "Resource Monitor duration은 "
            "0보다 커야 합니다."
        )

    if interval <= 0:
        raise ValueError(
            "resource_sample_sec는 "
            "0보다 커야 합니다."
        )

    path = Path(
        config["resource_log_path"]
    )

    path.parent.mkdir(
        parents=True,
        exist_ok=True,
    )

    print("=== Resource Monitor ===")
    print(f"Duration : {duration} sec")
    print(f"Interval : {interval} sec")
    print(f"Log      : {path}")
    print()

    # psutil.cpu_percent(interval=None)의
    # 첫 호출은 기준점을 만드는 용도로 사용합니다.
    get_cpu_percent()

    started = time.perf_counter()

    with path.open(
        "w",
        newline="",
        encoding="utf-8",
    ) as file:
        writer = csv.writer(file)

        writer.writerow(
            [
                "timestamp",
                "elapsed_sec",
                "cpu_percent",
                "memory_percent",
                "temperature_c",
                "throttled",
            ]
        )

        while True:
            elapsed = (
                time.perf_counter()
                - started
            )

            if elapsed >= duration:
                break

            timestamp = (
                datetime.now().isoformat(
                    timespec="seconds"
                )
            )

            cpu = get_cpu_percent()
            memory = get_memory_percent()
            temperature = (
                get_temperature_c()
            )
            throttled = (
                get_throttled_raw()
            )

            writer.writerow(
                [
                    timestamp,
                    f"{elapsed:.3f}",
                    f"{cpu:.2f}",
                    f"{memory:.2f}",
                    f"{temperature:.2f}",
                    throttled,
                ]
            )

            file.flush()

            print(
                f"{elapsed:6.1f}s | "
                f"CPU={cpu:5.1f}% | "
                f"MEM={memory:5.1f}% | "
                f"TEMP={temperature:5.1f}C | "
                f"{throttled}"
            )

            time.sleep(interval)


if __name__ == "__main__":
    main()
```

---

# 32. Terminal을 두 개 열어 장시간 실행하기

## Terminal A — Resource Monitor

PC에서 Raspberry Pi에 SSH 접속합니다.

```bash
ssh <RPI_USER>@<RPI_IP>
```

```bash
cd ~/ai_vision/subject13_edge_ai
source .venv/bin/activate
```

5분 Monitoring:

```bash
python -m scripts.resource_monitor \
  --duration 300
```

---

## Terminal B — AI System

별도의 PC 터미널에서 같은 Raspberry Pi에 접속합니다.

```bash
ssh <RPI_USER>@<RPI_IP>
```

```bash
cd ~/ai_vision/subject13_edge_ai
source .venv/bin/activate
```

5분 실행:

```bash
python -m scripts.optimized_ai_warning_system \
  --run-name stress_5min \
  --duration 300
```

---

두 SSH Terminal이 **같은 Raspberry Pi**에 접속했는지 각 Terminal에서 다음을 확인합니다.

```bash
hostname
hostname -I
pwd
```

Resource Monitor는 GPIO를 제어하지 않으므로 AI System과 동시에 실행해도 됩니다.  
단, Monitoring 자체도 아주 작은 추가 부하를 만들 수 있으므로 Report에는 “Resource Monitor 동시 실행” 조건을 기록합니다.

# 33. 장시간 실행 중 확인할 것

Resource Monitor에서 다음을 봅니다.

```text
CPU %
Memory %
Temperature
throttled
```

AI System에서는:

```text
정상적으로 계속 Frame을 읽는가?
Inference가 멈추지 않는가?
LED / Buzzer 상태가 정상인가?
```

5분이 끝나면:

```bash
tail logs/day09_resource.csv
```

로 마지막 상태를 확인합니다.

---

# 34. 장시간 실행 결과 Report에 기록하기

```md
## 5 Minute Stress Test

Start Temperature:

Max Temperature:

End Temperature:

CPU Range:

Memory Range:

Throttled Start:

Throttled End:

AI Program Error:

Camera Error:

관찰:
```

장비에 문제가 발생하면 무리하게 테스트를 계속하지 않습니다.

---

# 35. Camera 해상도를 높여 Stress 조건 바꾸기

8일차와 마찬가지로 Camera가 지원하는 범위에서 한 변수만 변경합니다.

기본:

```yaml
image_width: 640
image_height: 480
```

변경 후보:

```yaml
image_width: 1280
image_height: 720
```

단, 이 해상도를 Camera가 실제로 지원하는지 먼저 확인합니다. 지원하지 않으면 `1280 × 720`을 억지로 사용하지 않고 **현재 Camera가 지원하는 더 높은 해상도**를 선택합니다.

Model Input Size는 그대로 유지합니다.

```text
96 × 96
```

다시 Resource Monitor와 AI를 실행합니다.

예:

```bash
python -m scripts.resource_monitor \
  --duration 180
```

다른 터미널:

```bash
python -m scripts.optimized_ai_warning_system \
  --run-name stress_camera_720p \
  --duration 180
```

---

# 36. 해상도 증가 전·후 비교하기

Report:

```md
## Camera Resolution Stress

| Camera | CPU | Memory | Max Temp | Inference/sec | Throttled | 관찰 |
|---|---:|---:|---:|---:|---|---|
| 640 × 480 |  |  |  |  |  |  |
| 1280 × 720 |  |  |  |  |  |  |

변경한 변수:
Camera Resolution

고정한 변수:
Model / Model Input / Stabilization
```

실험 후 기본 해상도로 복구합니다.

---

# 37. 조명 변화 Stress Test

Camera Resolution은 기본으로 복구합니다.

```yaml
image_width: 640
image_height: 480
```

이제 장치 부하보다 **판정 안정성**에 Stress를 줍니다.

```text
밝은 환경
→ 보통 조명
→ 어두운 환경
```

같은 물체와 같은 거리를 유지합니다.

다음 값을 비교합니다.

```text
Raw Changes
Output Changes
```

Voting과 Hold가 조명 변화에서 결과 흔들림을 얼마나 줄였는지 확인합니다.

---

# 38. 물체를 빠르게 이동시켜 Frame Skip의 약점 확인하기

설정:

```yaml
inference_every_n_frames: 1
```

빨간색 카드를 Camera 앞에 매우 짧게 보여줍니다.

다음:

```yaml
inference_every_n_frames: 3
```

같은 동작을 반복합니다.

질문:

```text
Frame Skip 3에서
짧은 WARNING을 놓친 경우가 있는가?

Frame Skip 1에서는?
```

Frame Skip은 계산량을 줄이는 대신 짧은 Event를 놓칠 가능성이 있습니다.

---

# 39. 최적화에서 Accuracy와 안정성을 혼동하지 않기

Majority Voting을 적용했다고 AI Model Accuracy 자체가 높아진 것은 아닙니다.

```text
Model Accuracy
→ 이미지 한 장의 분류 성능

Temporal Stability
→ 시간에 따라 이어지는 여러 결과를 어떻게 운영 상태로 만들 것인가
```

예:

```text
Raw AI
NORMAL
WARNING
NORMAL
WARNING
WARNING

        ↓ Voting

Operational Output
WARNING
```

이것은 Model을 재학습한 것이 아니라 **운영 판단을 안정화한 것**입니다.

---

# 40. 최적화에서 FPS와 반응속도도 구분하기

Frame Skip을 적용하면 전체 Loop가 여유로워질 수 있습니다.

하지만 실제 AI 판단은 더 드물게 수행됩니다.

```text
Camera Loop
30 FPS

Inference Every 3 Frames
→ 약 10번/초 추론
```

따라서 다음을 별도로 봅니다.

```text
Camera Loop FPS

Inference/sec

WARNING Reaction
```

---


# PART E. Candidate를 같은 조건에서 비교하고 Final Operating Config 선택하기


# 41. 최적화 조합 후보 3개 만들기

오늘 실험 결과를 보고 세 가지 후보를 만듭니다.

예:

## Candidate A — 빠른 반응

```yaml
inference_every_n_frames: 1
vote_window: 3
vote_warning_min: 2
debounce_required: 1
warning_hold_sec: 0.5
```

## Candidate B — 균형

```yaml
inference_every_n_frames: 2
vote_window: 5
vote_warning_min: 3
debounce_required: 2
warning_hold_sec: 1.0
```

## Candidate C — 안정성 우선

```yaml
inference_every_n_frames: 2
vote_window: 7
vote_warning_min: 4
debounce_required: 2
warning_hold_sec: 2.0
```

이 값은 예시입니다.

자신의 실험 결과에 맞게 바꿉니다.

---

# 42. 세 후보를 같은 조건에서 비교하기

각 후보를 60초씩 실행합니다.

예:

```bash
python -m scripts.optimized_ai_warning_system \
  --run-name candidate_a \
  --duration 60
```

```bash
python -m scripts.optimized_ai_warning_system \
  --run-name candidate_b \
  --duration 60
```

```bash
python -m scripts.optimized_ai_warning_system \
  --run-name candidate_c \
  --duration 60
```

각 실험에서 같은 동작 패턴을 반복합니다.

---

# 43. Candidate 비교표 작성하기

```md
## Optimization Candidates

| Candidate | Frame Skip | Vote | Debounce | Hold | Inference/sec | Raw Changes | Output Changes | Output /100 Inf | 놓친 경고 | 체감 반응 |
|---|---:|---|---:|---:|---:|---:|---:|---:|---:|---|
| A |  |  |  |  |  |  |  |  |  |
| B |  |  |  |  |  |  |  |  |  |
| C |  |  |  |  |  |  |  |  |  |

선택:

선택 이유:
```

---

`Output Changes`가 적다고 무조건 좋은 설정은 아닙니다. **경고를 아예 놓쳐서 상태 변화가 적어진 경우**도 있기 때문에 `놓친 경고`와 반응성을 반드시 같이 봅니다.

# 44. Trade-off를 자신의 결과로 설명하기

다음 문장을 자신의 실험값으로 완성합니다.

```text
Frame Skip을 ___로 늘리니
Inference/sec는 ___에서 ___로 변했다.

하지만
____________________ 문제가 생겼다.
```

```text
Vote Window를 ___로 늘리니
Output Changes는 ___에서 ___로 줄었다.

하지만
____________________ 지연이 생겼다.
```

```text
Hold를 ___초로 설정하니
경고 깜빡임은 ____________________.

하지만
NORMAL 복귀는 ____________________.
```

---

# 45. 최종 운영 Config 고정하기

오늘 선택한 설정을 `configs/settings.yaml`에 적용합니다.

예:

```yaml
inference_every_n_frames: 2

vote_window: 5
vote_warning_min: 3

debounce_required: 2
warning_hold_sec: 1.0
```

이 값은 절대적인 정답이 아닙니다.

오늘의 측정 결과를 근거로 선택한 **현재 Baseline 운영값**입니다.

---

### 다음 날에 넘길 값이므로 여기서 함부로 다시 바꾸지 않습니다

10일차는 오늘 선택한 운영 Config를 그대로 사용하여 Headless Runtime과 systemd에 연결합니다.  
따라서 최종 선택 후에는 `reports/day09_optimization.md`에 값을 기록하고, Mini Challenge가 끝난 뒤 다시 이 값으로 복원합니다.

# 46. Config 선택 이유를 Report에 남기기

`reports/day09_optimization.md`에 다음을 완성합니다.

```md
## Final Configuration

Inference Every N Frames:

Vote Window:

Warning Minimum:

Debounce:

Hold:

## Why

속도:

안정성:

반응성:

CPU / Temperature:

이 설정을 선택한 이유:
```

---


# PART F. Guided Failure Lab — 오류를 영역별로 분류하고 복구하기


# 47. 오류를 최적화 영역별로 구분하기

| 증상 | 먼저 확인할 영역 |
|---|---|
| 추론 횟수가 예상보다 적음 | Frame Skip |
| WARNING이 너무 늦음 | Vote Window / Debounce |
| WARNING이 너무 오래 유지됨 | Hold |
| 결과가 계속 깜빡임 | Voting / Debounce |
| 짧은 위험을 놓침 | Frame Skip |
| CPU 사용률이 높음 | Inference 빈도 / Camera / Runtime |
| 온도가 계속 상승 | Cooling / Load / 장시간 실행 |
| `throttled` 값 변화 | 전원 / 온도 / 장치 상태 |
| INT8 실행 실패 | Runtime / Operator Support |
| INT8은 실행되나 느림 | Hardware / Runtime 특성 |
| Camera가 열리지 않음 | `device.local.yaml`의 Camera index / Camera 연결 |
| Buzzer가 의도치 않게 울림 | `use_buzzer` / 배선 / GPIO 설정 |
| Temperature가 `-1.0` | Thermal Zone 탐색 실패 / 현재 OS 제공 방식 |

---

# 48. Guided Failure Lab — 일부러 실패시키기 — Vote 조건 불가능하게 만들기

예:

```yaml
vote_window: 5
vote_warning_min: 6
```

실행:

```bash
python -m scripts.stabilizer_demo
```

현재 `TemporalStabilizer`는 다음 오류를 발생시켜야 합니다.

```text
vote_warning_min은
1~vote_window 범위여야 합니다.
```

이 오류는 Camera나 AI 문제가 아니라:

```text
Stabilization Config 문제
```

입니다.

복구:

```yaml
vote_window: 5
vote_warning_min: 3
```

---

# 49. Guided Failure Lab — 일부러 실패시키기 — Frame Skip을 0으로 설정하기

```yaml
inference_every_n_frames: 0
```

실행:

```bash
python -m scripts.optimized_ai_warning_system \
  --run-name wrong_skip \
  --duration 10
```

프로그램이 다음 설정 오류를 알려야 합니다.

```text
inference_every_n_frames는
1 이상이어야 합니다.
```

복구:

```yaml
inference_every_n_frames: 2
```

---


# PART G. Mini Challenge — 측정 근거로 최적화 Pipeline을 스스로 설명하고 수정하기

Guided Lab에서는 설정을 하나씩 바꾸며 함께 실험했습니다. 이제부터는 **문제 → 예상 → 코드 위치 → 실행 → 증거 → 복구** 순서를 스스로 수행합니다.

정답을 먼저 열지 않습니다. 먼저 자신의 예상과 실행 결과를 적은 뒤 각 Challenge의 **예시 정답 확인해보기**를 펼칩니다.

Mini Challenge를 시작하기 전에 현재 Final Candidate 설정과 핵심 Source를 임시 백업합니다.

```bash
cp configs/settings.yaml \
  /tmp/day09_settings_before_challenge.yaml

cp src/stabilizer.py \
  /tmp/day09_stabilizer_before_challenge.py

cp scripts/stabilizer_demo.py \
  /tmp/day09_stabilizer_demo_before_challenge.py
```

이 백업은 Challenge 복원을 위한 임시 파일입니다.

---

## Challenge 1 — 실행 결과를 보고 코드 위치 찾기

다음 결과가 보였다고 가정합니다.

```text
frame=   90 infer=   45 class=warning_red conf=0.882 raw=WARNING out=NORMAL
```

다음 표를 코드에서 직접 찾아 채웁니다.

| 결과 | 담당 파일 | 담당 코드/함수 |
|---|---|---|
| `frame=90` |  |  |
| `infer=45` |  |  |
| `warning_red` |  |  |
| `0.882` |  |  |
| `raw=WARNING` |  |  |
| `out=NORMAL` |  |  |
| Run Summary CSV |  |  |

특히 `raw=WARNING`인데 `out=NORMAL`이 가능한 이유를 설명합니다.

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

| 결과 | 담당 파일 | 담당 코드/함수 |
|---|---|---|
| `frame=90` | `scripts/optimized_ai_warning_system.py` | `frame_count` |
| `infer=45` | `scripts/optimized_ai_warning_system.py` | `inference_count` |
| `warning_red` | `src/inference_onnx.py` | `ONNXClassifier.predict()` |
| `0.882` | `src/inference_onnx.py` | Prediction Confidence |
| `raw=WARNING` | `src/decision.py` | `ai_to_operational_state()` |
| `out=NORMAL` | `src/stabilizer.py` | `TemporalStabilizer.update()`의 `output_state` |
| Run Summary CSV | `src/optimization_logger.py` | `OptimizationRunLogger.append()` |

`raw=WARNING` 한 번이 들어와도 최근 Window의 WARNING 수가 부족하거나 Debounce 조건을 아직 만족하지 않았다면 실제 출력은 `NORMAL`로 유지될 수 있습니다.

```text
Single AI Result
→ raw_state

최근 결과 + 운영 안정화
→ output_state
```

</details>

---

## Challenge 2 — 실행 전에 Stabilizer 결과를 예측하기

다음 설정을 사용한다고 가정합니다.

```yaml
vote_window: 5
vote_warning_min: 3
debounce_required: 2
warning_hold_sec: 0.0
```

입력은 `stabilizer_demo.py`의 앞 7개입니다.

```text
1 NORMAL
2 NORMAL
3 WARNING
4 NORMAL
5 WARNING
6 WARNING
7 WARNING
```

실행하기 전에 6번째와 7번째 결과를 예측합니다.

```text
6번째
warning_count:
voted_state:
stable_state:
output_state:

7번째
warning_count:
voted_state:
stable_state:
output_state:
```

이제 실행하여 비교합니다.

```bash
python -m scripts.stabilizer_demo
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

6번째 시점의 최근 5개는 다음과 같습니다.

```text
NORMAL
WARNING
NORMAL
WARNING
WARNING
```

WARNING은 3개이므로:

```text
voted_state = WARNING
candidate_count = 1
```

하지만 Debounce가 2이므로 아직 `stable_state`는 NORMAL입니다.

7번째에는 최근 5개 중 WARNING이 4개가 되고 WARNING 후보가 두 번 연속 이어집니다.

```text
6번째
warning_count = 3
voted_state   = WARNING
stable_state  = NORMAL
output_state  = NORMAL

7번째
warning_count = 4
voted_state   = WARNING
stable_state  = WARNING
output_state  = WARNING
```

핵심은 Voting과 Debounce가 서로 다른 단계라는 점입니다.

</details>

---

## Challenge 3 — Python 코드는 그대로 두고 설정값만 바꾸어 비교하기

이번 Challenge에서는 Python 파일을 수정하지 않습니다.

실험 A:

```yaml
inference_every_n_frames: 1
vote_window: 1
vote_warning_min: 1
debounce_required: 1
warning_hold_sec: 0.0
```

실험 B:

```yaml
inference_every_n_frames: 2
vote_window: 1
vote_warning_min: 1
debounce_required: 1
warning_hold_sec: 0.0
```

각각 같은 30초 Protocol로 실행합니다.

```bash
python -m scripts.optimized_ai_warning_system \
  --run-name challenge_skip1 \
  --duration 30
```

```bash
python -m scripts.optimized_ai_warning_system \
  --run-name challenge_skip2 \
  --duration 30
```

실행 전에 먼저 예상합니다.

```text
어느 실험의 Inference/sec가 더 낮을까?
Model-only Latency 자체가 절반으로 줄어들까?
Raw Changes 개수만으로 안정성이 좋아졌다고 말할 수 있을까?
어떤 추가 열을 함께 봐야 할까?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

보통 `inference_every_n_frames: 2`에서 AI 호출 횟수와 `Inference/sec`가 줄어듭니다.

하지만 Frame Skip은 ONNX 한 번의 계산 자체를 빠르게 만든 것이 아닙니다.

```text
Model-only Latency
→ 같은 ONNX Model이라면 본질적으로 같은 종류의 연산

Inference 호출 빈도
→ 줄어듦
```

또 `Raw Changes`가 줄었다고 바로 안정성이 좋아졌다고 말하면 안 됩니다. 추론 횟수가 줄었기 때문일 수도 있습니다.

따라서 다음을 같이 봅니다.

```text
Inference/sec
Raw Changes
Output Changes
Raw Changes / 100 Inferences
Output Changes / 100 Inferences
짧은 WARNING 누락 여부
```

</details>

---

## Challenge 4 — 기존 기능을 수정하여 Output 전환 여부를 표시하기

요구사항:

> `stabilizer_demo.py`에서 실제 `output_state`가 직전 출력과 달라진 순간에만 `CHANGED`라고 표시하고 싶습니다.

먼저 어느 파일을 수정할지 생각합니다.

이번 Challenge에서는 `stabilizer.py`의 알고리즘 자체를 바꾸지 않고 **Demo 출력 기능만 확장**합니다.

힌트:

```text
previous_output
현재 result["output_state"]
비교
```

직접 구현한 뒤 실행합니다.

```bash
python -m scripts.stabilizer_demo
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

`main()`에서 반복문 전에 다음 변수를 준비할 수 있습니다.

```python
previous_output = None
```

`result = stabilizer.update(...)` 다음에:

```python
output_state = result["output_state"]

changed = (
    previous_output is not None
    and output_state != previous_output
)

change_text = (
    "CHANGED"
    if changed
    else "-"
)

previous_output = output_state
```

기존 `print()` 끝에 다음을 추가할 수 있습니다.

```python
f" | change={change_text}"
```

예상 형태:

```text
06 | raw=WARNING | ... | output=NORMAL  | change=-
07 | raw=WARNING | ... | output=WARNING | change=CHANGED
```

이 기능은 AI 정확도를 높이는 기능이 아니라 **상태 전환을 눈으로 더 쉽게 확인하기 위한 Debug 출력**입니다.

Challenge가 끝난 뒤 원래 Demo Source로 복원합니다.

</details>

---

## Challenge 5 — 일부러 오류를 만들고 원인 찾기

이번에는 기존 Guided Failure와 다른 설정을 잘못 만들어 봅니다.

```yaml
warning_hold_sec: -1.0
```

실행:

```bash
python -m scripts.stabilizer_demo
```

오류가 나면 다음 순서로 확인합니다.

```text
오류 마지막 줄
        ↓
어느 설정 이름이 보이는가?
        ↓
어느 Class가 검증하는가?
        ↓
settings.yaml 값 확인
        ↓
정상 범위로 수정
        ↓
재실행
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

최종 `TemporalStabilizer`는 음수 Hold 시간을 허용하지 않습니다.

```python
if warning_hold_sec < 0:
    raise ValueError(
        "warning_hold_sec는 "
        "0 이상이어야 합니다."
    )
```

따라서 문제 영역은 Camera나 ONNX가 아니라:

```text
Stabilization Config
```

입니다.

복구 예:

```yaml
warning_hold_sec: 1.0
```

단, Challenge를 끝낸 뒤에는 임의의 `1.0`이 아니라 **오늘 자신이 선택한 Final Candidate 값**으로 되돌립니다.

</details>

---

## Challenge 6 — 결과 파일과 Log로 정상 동작을 증명하기

터미널에서 “잘 됐다”고 말하는 것으로 끝내지 않습니다.

30초 실험 하나를 실행합니다.

```bash
python -m scripts.optimized_ai_warning_system \
  --run-name challenge_proof \
  --duration 30
```

Summary CSV에서 해당 Run을 찾습니다.

```bash
tail -n 10 \
  reports/day09_optimization_runs.csv
```

Resource Monitor도 짧게 실행합니다.

```bash
python -m scripts.resource_monitor \
  --duration 20
```

확인:

```bash
head logs/day09_resource.csv
tail logs/day09_resource.csv
```

다음 질문에 답합니다.

```text
challenge_proof Run이 CSV에 실제로 있는가?
frames와 inference_count가 0보다 큰가?
Frame Skip 설정이 CSV와 일치하는가?
Resource CSV에 timestamp가 있는가?
CPU / Memory / Temperature 열이 존재하는가?
Temperature가 -1.0이라면 어떤 의미인가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

정상적인 증거는 다음처럼 연결됩니다.

```text
실행 명령
        ↓
optimized_ai_warning_system.py
        ↓
OptimizationRunLogger.append()
        ↓
reports/day09_optimization_runs.csv
```

Resource Monitoring은:

```text
scripts/resource_monitor.py
        ↓
src/resource_monitor.py
        ↓
logs/day09_resource.csv
```

`frames > 0`, `inference_count > 0`이고 실행한 설정이 CSV 한 줄에 남아 있어야 합니다.

Temperature가 `-1.0`이면 실제 온도가 영하 1도라는 뜻이 아니라 **현재 코드가 사용할 수 있는 Thermal Zone을 찾지 못했다는 표시**입니다.

</details>

---

## Challenge 7 — 오늘 배운 전체 Pipeline을 자신의 말로 설명하기

코드를 닫고 다음 빈칸을 자신의 말로 채웁니다.

```text
8일차에서 우리는 ____________________________ 를 측정했다.

9일차 Frame Skip은 Model 한 번을 빠르게 하는 것이 아니라
_____________________________________________.

Raw AI 결과는 ________________________________ 를 거쳐
실제 Output 상태로 바뀐다.

Voting의 역할은 ______________________________.

Debounce의 역할은 ____________________________.

Hold의 역할은 ________________________________.

Resource Monitor는 ____________________________ 를 확인한다.

최종 Config는 가장 숫자가 큰 설정이 아니라
_____________________________________________ 근거로 선택한다.

10일차에는 이 Final Pipeline을
_____________________________________________ 방식으로 운영한다.
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

예시:

```text
8일차에서 우리는 Model-only와 End-to-End의
Latency / FPS / P95 / 병목과 장치 상태를 측정했다.

Frame Skip은 AI 추론을 호출하는 빈도를 줄인다.

Raw AI 결과는
Voting → Debounce → Hold를 거쳐
실제 Output 상태가 된다.

Voting은 최근 여러 AI 판단을 모아 후보 상태를 정한다.

Debounce는 같은 후보가 일정 횟수 이어져야 상태를 바꾼다.

Hold는 WARNING 확정 후 최소 시간 동안 경고를 유지한다.

Resource Monitor는 CPU / Memory / Temperature / Throttling을 기록한다.

최종 Config는
속도 + 안정성 + 반응성 + 놓친 경고 + 장치 상태의
Trade-off를 근거로 선택한다.

10일차에는 이 Final Pipeline을
Headless Runtime과 systemd 자동실행 구조로 운영한다.
```

문장을 그대로 외울 필요는 없습니다. **같은 의미를 자신의 말로 설명할 수 있으면 됩니다.**

</details>

---


# PART H. Mini Challenge 상태를 복원하고 10일차용 Final Baseline 고정하기

# 50. Challenge에서 변경한 설정과 Source 복원하기

9일차는 Challenge로 끝나지 않습니다. 10일차는 오늘 선택한 **Final Operating Config와 정상 Source**를 그대로 이어서 사용합니다.

임시 백업을 복원합니다.

```bash
cp /tmp/day09_settings_before_challenge.yaml \
  configs/settings.yaml

cp /tmp/day09_stabilizer_before_challenge.py \
  src/stabilizer.py

cp /tmp/day09_stabilizer_demo_before_challenge.py \
  scripts/stabilizer_demo.py
```

설정 확인:

```bash
grep -E \
"inference_every_n_frames|vote_window|vote_warning_min|debounce_required|warning_hold_sec" \
configs/settings.yaml
```

중요한 점:

```text
복원해야 하는 값
→ 오늘 Candidate 비교 후 내가 선택한 Final Config

무조건 예시값 2 / 5 / 3 / 2 / 1.0
→ 아님
```

Camera index도 확인합니다.

```bash
python -m scripts.camera_index_probe
```

`configs/device.local.yaml`의 실제 장치값을 유지합니다.

---

# 51. Final Baseline을 다시 실행해 PASS 확인하기

Stabilizer:

```bash
python -m scripts.stabilizer_demo
```

실제 AI System:

```bash
python -m scripts.optimized_ai_warning_system \
  --run-name final_day09 \
  --duration 60
```

Resource:

```bash
python -m scripts.resource_monitor \
  --duration 60
```

결과 파일:

```bash
tail -n 10 \
  reports/day09_optimization_runs.csv
```

```bash
tail \
  logs/day09_resource.csv
```

다음이 모두 성립해야 합니다.

```text
Camera Read 정상
ONNX Inference 정상
Raw Decision 정상
Stabilizer 정상
GPIO 정상
Optimization Summary 생성
Resource Log 생성
Final Config 복원
```

여기까지가 10일차에 넘기는 **Day 09 Handoff Baseline**입니다.

---


# PART I. 선택 심화 — Precision과 Quantization을 “변환”이 아니라 “측정”으로 보기

아래 Precision / INT8 내용은 **9일차 필수 최적화의 핵심이 아닙니다.**

오늘 반드시 끝내야 하는 핵심은:

```text
Frame Skip
+ Voting
+ Debounce
+ Hold
+ Resource Monitoring
+ Candidate 비교
+ Final Config
```

입니다.

수업 시간이 충분하고 현재 PC/Runtime에서 Quantization 실습이 가능한 경우에만 다음 선택 실습을 진행합니다.

중요한 원칙:

> **“파일이 작아졌다”와 “Raspberry Pi가 빨라졌다”는 같은 말이 아닙니다.**

---

# 52. 모델 크기와 숫자 표현을 먼저 확인하기

6~8일차에 사용한 Tiny CNN은 일반적으로 FP32 Weight를 사용합니다.

FP32:

```text
1개의 값
≈ 4 bytes
```

FP16:

```text
1개의 값
≈ 2 bytes
```

INT8:

```text
1개의 값
≈ 1 byte
```

같은 Parameter 수라고 가정하면 Weight 저장량은 이론적으로 다음처럼 줄어들 수 있습니다.

```text
FP32
4

FP16
2

INT8
1
```

그러나 **파일 크기가 줄었다고 반드시 Raspberry Pi에서 빨라지는 것은 아닙니다.**

속도는 다음 조건의 영향을 함께 받습니다.

```text
CPU가 해당 연산을 얼마나 잘 지원하는가?
Runtime이 해당 정밀도 연산을 지원하는가?
Model이 어떤 연산으로 구성되어 있는가?
변환 과정에서 Cast가 추가되는가?
```

---

# 53. 현재 모델의 Parameter 수와 실제 파일 크기 확인하기

PC에서 학습한 원본 PyTorch 모델의 정보는 6일차 `model_summary.py`로 확인할 수 있습니다.

PC:

```bash
python -m scripts.model_summary
```

ONNX 파일 크기:

```bash
ls -lh models/day07_tiny_cnn.onnx
```

Raspberry Pi에서도:

```bash
ls -lh models/day07_tiny_cnn.onnx
```

같은 모델 파일이라면 크기는 같아야 합니다.

---

# 54. FP32·FP16·INT8의 이론적 Weight 크기 계산하기

파일:

```text
scripts/precision_size_demo.py
```

코드:

```python
from pathlib import Path

import torch

from src.config_loader import load_config
from src.tiny_cnn import TinyCNN


def mb(byte_count: int) -> float:
    return byte_count / (1024 * 1024)


def main():
    config = load_config(
        "configs/settings.yaml"
    )

    checkpoint_path = Path(
        config["ai_model_path"]
    )

    if not checkpoint_path.exists():
        print(
            "이 실습은 PyTorch Model이 있는 "
            "PC에서 실행합니다."
        )
        return

    checkpoint = torch.load(
        checkpoint_path,
        map_location="cpu",
    )

    classes = checkpoint["classes"]

    model = TinyCNN(
        num_classes=len(classes)
    )

    model.load_state_dict(
        checkpoint["model_state"]
    )

    count = sum(
        parameter.numel()
        for parameter in model.parameters()
    )

    fp32_bytes = count * 4
    fp16_bytes = count * 2
    int8_bytes = count * 1

    print("=== Precision Size Demo ===")
    print(f"Parameters : {count:,}")
    print()
    print(
        f"FP32 raw weights : "
        f"{mb(fp32_bytes):.4f} MB"
    )
    print(
        f"FP16 raw weights : "
        f"{mb(fp16_bytes):.4f} MB"
    )
    print(
        f"INT8 raw weights : "
        f"{mb(int8_bytes):.4f} MB"
    )

    print()
    print(
        "실제 ONNX 파일 크기는 "
        "Graph와 Metadata 등의 영향으로 "
        "이 값과 정확히 같지 않습니다."
    )


if __name__ == "__main__":
    main()
```

이 실습은 **PC에서 실행**합니다.

```bash
python -m scripts.precision_size_demo
```

결과를 Report에 적습니다.

---

# 55. Quantization은 오늘의 필수 성능 개선 방법이 아닙니다

오늘의 필수 최적화는 다음입니다.

```text
Frame Skip
Voting
Debounce
Hold
```

FP16·INT8은 개념을 이해하고 **현재 Runtime과 Model에서 실제 적용 가능한 경우에만** 추가 실험합니다.

이유:

```text
FP16 Model
→ Raspberry Pi CPU에서 반드시 빠르다는 보장 없음

INT8 Model
→ Runtime / Operator 지원 필요

Quantization
→ Accuracy 재검증 필요
```

즉 다음처럼 판단합니다.

```text
변환 성공
≠
실제 장치에서 성능 향상
```

반드시 Benchmark 결과로 확인해야 합니다.

---

# 56. 선택 실습 — INT8 Dynamic Quantization 파일 만들어 보기

이 실습은 ONNX Runtime Quantization 기능이 현재 모델에서 정상 동작하는 경우에만 진행합니다.

PC에서 파일을 만듭니다.

```text
scripts/quantize_int8_optional.py
```

코드:

```python
from pathlib import Path

from onnxruntime.quantization import (
    QuantType,
    quantize_dynamic,
)

from src.config_loader import load_config


def main():
    config = load_config(
        "configs/settings.yaml"
    )

    source = Path(
        config["onnx_model_path"]
    )

    target = Path(
        "models/day09_tiny_cnn_int8.onnx"
    )

    if not source.exists():
        raise FileNotFoundError(
            f"ONNX Model이 없습니다: {source}"
        )

    print("=== Optional INT8 Quantization ===")
    print(f"Source : {source}")
    print(f"Target : {target}")

    quantize_dynamic(
        model_input=str(source),
        model_output=str(target),
        weight_type=QuantType.QInt8,
    )

    print()
    print("Quantization completed.")
    print(
        f"FP32 file : "
        f"{source.stat().st_size / 1024:.1f} KB"
    )
    print(
        f"INT8 file : "
        f"{target.stat().st_size / 1024:.1f} KB"
    )


if __name__ == "__main__":
    main()
```

실행:

```bash
python -m scripts.quantize_int8_optional
```

변환이 실패하면 오류를 기록하고 필수 실습으로 돌아갑니다.

---

# 57. INT8이 만들어져도 바로 빠르다고 결론 내리지 않기

INT8 파일이 생성되었다면 다음 순서가 필요합니다.

```text
1. PC에서 ONNX Runtime 실행 가능?

2. Test Accuracy 유지?

3. Raspberry Pi Runtime에서 실행 가능?

4. Model-only Latency 실제로 감소?

5. P95는?

6. 파일 크기는?
```

어느 단계에서 지원되지 않아도 정상적인 학습 결과입니다.

Report에는 다음처럼 기록합니다.

```md
## Optional Quantization

INT8 변환: 성공 / 실패

Model Size:

PC 실행: 성공 / 실패

Raspberry Pi 실행: 성공 / 실패

Accuracy 변화:

Latency 변화:

결론:
```

---


# PART J. 8일차 Baseline과 비교하고 결과를 문서화하기


# 58. 8일차 Baseline과 9일차 최종 상태 비교하기

Report에 다음 표를 추가합니다.

```md
## Day 08 Baseline vs Day 09 Optimized

| 항목 | Day 08 Baseline | Day 09 Selected | 변화 |
|---|---:|---:|---|
| Inference Every N Frames | 1 |  |  |
| Inference/sec |  |  |  |
| Raw State Changes |  |  |  |
| Output State Changes |  |  |  |
| Output Changes / 100 Inferences |  |  |  |
| CPU |  |  |  |
| Temperature |  |  |  |
| 놓친 경고 |  |  |  |
| 체감 반응 |  |  |  |

개선된 점:

나빠진 점:

Trade-off:
```

Model-only Latency가 동일해도 운영 Pipeline은 달라질 수 있습니다.

---

# 59. 9일차 Source Git Checkpoint 만들기

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
src/stabilizer.py \
src/optimization_logger.py \
src/resource_monitor.py \
scripts/stabilizer_demo.py \
scripts/optimized_ai_warning_system.py \
scripts/resource_monitor.py
```

선택 Quantization 실습을 했다면:

```bash
git add \
scripts/precision_size_demo.py \
scripts/quantize_int8_optional.py
```

Commit:

```bash
git commit -m \
"feat: add frame skip and temporal stabilization"
```

---

# 60. 9일차 Report Commit 만들기

실험 결과:

```bash
git add \
reports/day09_optimization.md \
reports/day09_pi_environment.txt
```

`reports/day09_optimization_runs.csv`가 생성되어 있다면 함께 관리할 수 있습니다.

```bash
git add reports/day09_optimization_runs.csv
```

Commit:

```bash
git commit -m \
"docs: record day09 optimization tradeoffs"
```

---

# 61. README에 9일차 실행 방법 추가하기

`README.md`에 다음 내용을 추가합니다.

````md
## Day 09 Optimization

### Stabilizer Demo

```bash
python -m scripts.stabilizer_demo
```

### Optimized AI System

```bash
python -m scripts.optimized_ai_warning_system \
  --run-name test \
  --duration 60
```

### Resource Monitor

```bash
python -m scripts.resource_monitor \
  --duration 300
```

### Optimization Flow

```text
Day 08 Baseline
→ Frame Skip
→ Majority Voting
→ Debounce
→ Hold
→ Resource Monitoring
→ Candidate Comparison
→ Final Operating Config
```

### Day 10 Handoff

```text
scripts/optimized_ai_warning_system.py
+ Day 09 Final Config
→ Headless Runtime / systemd / FastAPI
```
````

README도 Commit합니다.

```bash
git add README.md
git commit -m \
"docs: add day09 optimization commands"
```

# 62. 원격 Repository에 Push하기

허용된 Remote Repository가 연결되어 있고 교육장 보안정책상 Push가 가능한 경우:

```bash
git push
```

기관 보안정책에서 외부 GitHub가 제한되면 Local Git 또는 허용된 내부 저장소를 사용합니다.

Dataset, ONNX Model, Runtime Log는 기존 보안·저장 정책을 유지합니다.

---

# 63. 9일차가 끝난 시점의 프로젝트 구조

공통 Source:

```text
subject13_edge_ai/
│
├── configs/
│   └── settings.yaml
│
├── reports/
│   ├── ...
│   ├── day08_performance.md
│   ├── day09_optimization.md
│   ├── day09_optimization_runs.csv
│   └── day09_pi_environment.txt
│
├── scripts/
│   ├── ...
│   ├── optimized_ai_warning_system.py
│   ├── resource_monitor.py
│   └── stabilizer_demo.py
│
├── src/
│   ├── ...
│   ├── optimization_logger.py
│   ├── resource_monitor.py
│   └── stabilizer.py
│
├── .gitignore
├── README.md
└── requirements.txt
```

선택 실습:

```text
scripts/
├── precision_size_demo.py
└── quantize_int8_optional.py
```

장치에만 존재:

```text
logs/
├── day09_resource.csv
└── ...

models/
├── day07_tiny_cnn.onnx
└── 선택 INT8 Model
```

---

Raspberry Pi 장치 전용 파일은 공통 Source와 구분합니다.

```text
configs/device.local.yaml
→ camera_index 등 장치별 값
→ 다른 장치의 파일로 덮어쓰지 않음

logs/
→ Runtime / Resource Log
→ 기본적으로 Git 제외
```

# 64. 1~9일차 시스템 성장 확인하기

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
→ Mean / P50 / P95
→ 병목 확인
```

9일차:

```text
병목과 흔들림
        ↓
Frame Skip
        ↓
Voting
        ↓
Debounce
        ↓
Hold
        ↓
Resource Monitoring
        ↓
Trade-off 비교
        ↓
운영 Config 선택
```

이제 시스템은 단순히 AI가 실행되는 상태를 넘어 **속도와 출력 안정성을 조정할 수 있는 상태**가 되었습니다.

---

# 65. 수업 종료 전 최종 확인

Raspberry Pi:

```bash
cd ~/ai_vision/subject13_edge_ai
source .venv/bin/activate
```

안정화 로직:

```bash
python -m scripts.stabilizer_demo
```

최종 AI System:

```bash
python -m scripts.optimized_ai_warning_system \
  --run-name final_day09 \
  --duration 60
```

자원 상태:

```bash
python -m scripts.resource_monitor \
  --duration 60
```

Throttling과 온도는 현재 OS에서 `vcgencmd`가 지원되는 경우 확인합니다.

```bash
command -v vcgencmd
```

지원된다면:

```bash
vcgencmd get_throttled
vcgencmd measure_temp
```

지원되지 않으면 `logs/day09_resource.csv`와 현재 OS가 제공하는 Thermal 정보로 확인합니다.

Git:

```bash
git status
git log --oneline -12
```

다음 결과가 있어야 합니다.

```text
Frame Skip 실험

Voting 3 / 5 / 7 비교

Debounce 적용

Hold 비교

Resource CSV

장시간 실행 결과

Candidate 비교표

최종 운영 Config

Trade-off 설명
```

---


# PART K. 최종 복습 · 자가 체크 · 10일차 Handoff

# 66. 9일차 핵심 복습 문제

먼저 답을 보지 않고 자신의 말로 설명합니다.

1. 8일차와 9일차의 가장 큰 차이는 무엇인가요?
2. Frame Skip 2는 무엇을 의미하나요?
3. Frame Skip이 ONNX Model-only Latency 자체를 절반으로 만드는 기술인가요?
4. `vote_window: 5`는 항상 Camera 5 Frame을 의미하나요?
5. Majority Voting의 역할은 무엇인가요?
6. Debounce와 Hold의 차이는 무엇인가요?
7. `raw_state`와 `output_state`는 왜 다를 수 있나요?
8. Frame Skip을 1에서 3으로 바꿨더니 Raw Changes가 줄었습니다. 바로 안정성이 좋아졌다고 결론내리면 안 되는 이유는 무엇인가요?
9. `raw_changes_per_100_inferences` 같은 정규화 값은 왜 도움이 되나요?
10. `Output Changes`가 가장 적은 Candidate가 항상 최선인가요?
11. Camera 해상도 Stress에서 Model Input 96×96을 그대로 두는 이유는 무엇인가요?
12. Resource Monitor에서 확인하는 핵심 값은 무엇인가요?
13. Temperature가 `-1.0`이면 무엇을 의미하나요?
14. `vcgencmd`가 없으면 프로그램 전체가 실패해야 하나요?
15. `warning_hold_sec: -1.0`은 왜 오류인가요?
16. `use_buzzer`를 무조건 `true`로 강제하지 않는 이유는 무엇인가요?
17. 60초 A/B 실험에서 동작 Protocol을 최대한 같게 하는 이유는 무엇인가요?
18. 9일차의 Final Config를 선택할 때 함께 봐야 하는 기준은 무엇인가요?
19. 10일차가 9일차에서 가장 중요하게 넘겨받는 Source와 설정은 무엇인가요?
20. 10일차에서 새로 추가되는 운영 개념은 무엇인가요?

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

1. 8일차는 현재 성능을 측정하고 병목을 찾는 날이고, 9일차는 그 측정 결과를 근거로 운영 방식을 바꾸어 전후를 비교하는 날입니다.
2. Camera Frame은 계속 읽되 대략 두 Frame마다 한 번 AI 추론을 수행하는 설정입니다.
3. 아닙니다. Model 한 번의 연산을 줄이는 것이 아니라 호출 빈도를 줄이는 방식입니다.
4. 아닙니다. 현재 구조에서는 Stabilizer가 AI 추론 시점에만 업데이트되므로 최근 5개의 **AI 판단 결과**입니다.
5. 최근 여러 판단에서 WARNING 개수를 세어 하나의 후보 상태를 만드는 역할입니다.
6. Debounce는 후보 상태가 일정 횟수 이어져야 실제 상태를 변경하고, Hold는 확정된 WARNING을 최소 시간 유지합니다.
7. 단일 AI 결과가 WARNING이어도 Voting/Debounce 조건이 부족하면 실제 Output은 NORMAL일 수 있습니다.
8. Frame Skip이 커지면 추론 횟수 자체가 줄어 Raw 변화 기회도 줄기 때문입니다.
9. 서로 다른 Inference 수를 가진 실험을 더 공정하게 해석하는 데 도움이 됩니다.
10. 아닙니다. 경고를 놓쳐 변화가 적어진 것일 수도 있으므로 놓친 경고와 반응성을 함께 봐야 합니다.
11. Camera 입력 부하만 바꾸는 One Variable 실험으로 만들기 위해서입니다.
12. CPU, Memory, Temperature, Throttling 및 시간에 따른 변화입니다.
13. 현재 코드가 사용할 수 있는 Thermal Zone을 찾지 못했다는 표시입니다.
14. 아닙니다. 지원되지 않으면 `UNAVAILABLE`로 기록하고 다른 장치 상태 정보와 함께 해석할 수 있습니다.
15. 최소 유지 시간은 음수가 될 수 없기 때문입니다.
16. 실제 Buzzer 장비 사양과 배선이 다를 수 있고, 반복 실험에서 불필요한 소음이나 잘못된 출력이 생길 수 있기 때문입니다.
17. 결과 차이가 설정 때문인지 사람이 다른 동작을 했기 때문인지 구분하기 위해서입니다.
18. Inference/sec, 상태 흔들림, 놓친 경고, 반응성, CPU/Memory/Temperature 등을 함께 봅니다.
19. `scripts/optimized_ai_warning_system.py`, `src/stabilizer.py`와 오늘 선택한 Final Operating Config입니다.
20. Headless Runtime, systemd 자동실행/자동재시작, Runtime State와 FastAPI 상태 조회입니다.

</details>

---

# 67. 자가 체크리스트

- [ ] 8일차 `reports/day08_performance.md`와 성능 Summary를 확인했다.
- [ ] 오늘 Baseline을 같은 조건으로 다시 측정했다.
- [ ] 실제 Raspberry Pi Hostname/IP/Camera index를 확인했다.
- [ ] `configs/device.local.yaml`을 공통 설정으로 덮어쓰지 않았다.
- [ ] Frame Skip이 Model 자체를 빠르게 하는 기술이 아님을 설명할 수 있다.
- [ ] Majority Voting / Debounce / Hold를 구분할 수 있다.
- [ ] `vote_window`가 최근 AI 판단 개수임을 이해했다.
- [ ] `stabilizer_demo.py`를 실행하고 결과를 코드로 역추적했다.
- [ ] Frame Skip 1/2/3 중 두 조건 이상을 비교했다.
- [ ] Voting Window 3/5/7 중 두 조건 이상을 비교했다.
- [ ] Debounce와 Hold를 각각 적용해 보았다.
- [ ] `reports/day09_optimization_runs.csv`가 생성되는 것을 확인했다.
- [ ] 상태 변화 횟수와 Inference 수를 함께 해석했다.
- [ ] CPU/Memory/Temperature를 `logs/day09_resource.csv`에 기록했다.
- [ ] `vcgencmd` 지원 여부를 확인하고 결과를 올바르게 기록했다.
- [ ] 5분 Stress Test 또는 수업에서 정한 장시간 Test를 수행했다.
- [ ] 짧은 WARNING에서 Frame Skip의 약점을 확인했다.
- [ ] Candidate A/B/C 또는 이에 준하는 후보를 같은 Protocol로 비교했다.
- [ ] Final Operating Config를 측정 근거로 선택했다.
- [ ] Mini Challenge 7개를 수행했다.
- [ ] Mini Challenge에서 변경한 Source와 설정을 복원했다.
- [ ] Final Day 09 Run이 정상 실행되었다.
- [ ] `reports/day09_optimization.md`를 완성했다.
- [ ] README에 Day 09 실행방법을 기록했다.
- [ ] Source와 Report를 Git에 저장했다.
- [ ] 10일차에 무엇을 이어서 사용할지 설명할 수 있다.

---

# 68. 9일차 핵심을 한 문장으로 설명하기

오늘 한 일을 다음 한 문장으로 설명할 수 있으면 됩니다.

> **“8일차에서 측정한 Raspberry Pi Edge AI Baseline을 기준으로 Frame Skip과 시간적 안정화를 한 변수씩 비교하고, Resource 상태와 반응성까지 함께 확인하여 10일차가 사용할 운영 Config를 선택했다.”**

---


# 69. 다음 날 연결

9일차까지는 프로그램을 사람이 직접 실행했습니다.

```bash
python -m scripts.optimized_ai_warning_system
```

프로그램이 종료되면 사람이 다시 실행해야 합니다.

Raspberry Pi가 재부팅되어도 사람이 SSH로 접속하여 실행해야 합니다.

다음 날에는 이 운영 방식을 바꿉니다.

```text
Raspberry Pi Boot
        ↓
systemd
        ↓
Edge Warning System 자동 실행
        ↓
비정상 종료
        ↓
자동 재시작
```

그리고 외부 PC에서는 Raspberry Pi에 매번 직접 들어가지 않아도 상태를 확인할 수 있도록 다음 연결을 추가합니다.

```text
Edge Warning System
        ↓
상태 정보
        ↓
FastAPI
        ↓
PC Browser / Client
```

즉 10일차부터는:

> **“잘 동작하는 프로그램”**

을

> **“사람이 계속 붙어 있지 않아도 운영되는 Edge 프로그램”**

으로 바꾸기 시작합니다.
