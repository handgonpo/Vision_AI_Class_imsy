
> **오늘의 핵심:** 11일차까지 Raspberry Pi에서 만든 Edge AI 시스템을 Test와 Failure Analysis로 검증했습니다.  
> 12일차에는 그 시스템을 버리고 새로 만드는 것이 아니라, **검증된 구조를 기준으로 Jetson으로 옮길 때 무엇을 그대로 재사용하고 무엇을 장치에 맞게 바꾸어야 하는지 실제 환경·모델·Runtime·성능 결과로 확인**합니다.
>
> 오늘의 목표는 “Jetson 명령어를 많이 외우는 것”이 아닙니다.
>
> ```text
> Raspberry Pi에서 검증한 Edge Pipeline
>         ↓
> 장치 독립 영역 / 장치 종속 영역 구분
>         ↓
> Jetson 환경 확인
>         ↓
> 같은 Source와 ONNX 배포
>         ↓
> TensorRT Engine 생성·Benchmark
>         ↓
> Camera / Inference Contract 정리
>         ↓
> Migration Map 작성
>         ↓
> Raspberry Pi 운영 Baseline 복원
>         ↓
> 13일차 Final Acceptance Test
> ```
>
> 1~11일차에 사용한 `subject13_edge_ai` 프로젝트를 그대로 이어서 사용합니다. **새 프로젝트를 만들지 않습니다.**
>
> Jetson 장비가 준비되어 있지 않은 경우에는 제공된 Jetson 환경 화면·TensorRT 변환 결과·Benchmark 자료를 사용하여 같은 구조 분석과 비교 실습을 진행합니다. 없는 측정값을 임의로 만들지 않습니다.

---

## 11일차와 12일차는 어디가 이어질까요?

11일차에서는 10일차 운영형 Raspberry Pi 시스템을 다음 순서로 검증했습니다.

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
Failure Analysis
        ↓
Trade-off 판단
        ↓
11일차 Tested Baseline
```

12일차는 이 검증 결과를 출발점으로 사용합니다.

```text
11일차

Raspberry Pi
Camera
→ Preprocess
→ ONNX AI
→ Decision
→ Stabilization
→ GPIO / Log
→ Performance
→ systemd
→ FastAPI
→ Test / Failure Analysis

        ↓

12일차

같은 Source 구조
+
11일차에서 실제 채택된 운영 ONNX + Metadata
+
같은 Test 관점
        ↓
Jetson 환경으로 이동
        ↓
재사용할 부분과 교체할 부분 확인
```

오늘 처음부터 Camera 코드, Decision, Stabilizer, Runtime State, Performance 계산, Unit Test를 다시 만들지 않습니다. 이 파일들은 **이전 일차의 파일을 다시 사용**합니다.

11일차 마지막에 `main`이 깨끗하게 정리되어 있어야 하며, 오늘 시작할 때 그 상태를 `day11-rpi-tested-baseline` 기준점으로 남깁니다.

### 먼저 정할 한 가지 — 12일차에서 어떤 ONNX를 Jetson으로 옮길까요?

11일차에서는 `96×96`, `64×64`, Confidence Threshold 같은 후보를 **Validation에서 비교**했습니다.  
하지만 후보를 비교했다는 사실과 그 후보를 **실제 운영 Baseline으로 채택했다는 사실은 다릅니다.**

12일차에서는 파일명만 보고 모델을 추측하지 않습니다.

```text
11일차 실험 후보
→ 비교용일 수 있음

11일차에서 운영 Baseline으로 실제 채택
→ configs/settings.yaml의
   onnx_model_path / onnx_meta_path에 반영
→ System Test 재확인
→ day11-rpi-tested-baseline 기준점

12일차
→ 위 Baseline이 실제 참조하는 ONNX + Metadata를 Jetson으로 이동
```

따라서 11일차에서 별도 채택 결정을 하지 않았다면 기존 10일차 운영 Baseline이 계속 기준이 될 수 있습니다. 이 경우 기본적으로 `day07_tiny_cnn.onnx` 계열을 사용할 수 있습니다.

반대로 11일차에서 `64×64` 또는 다른 검증된 후보를 **공식 운영 Baseline으로 채택**했다면 12일차는 그 모델과 짝이 되는 Metadata를 사용합니다.

> **12일차의 Source of Truth는 특정 파일명이 아니라 현재 `day11-rpi-tested-baseline`의 실제 운영 Config입니다.**

---

## 12일차 결과는 13일차에 어떻게 연결될까요?

12일차의 Jetson 실습은 교과 14를 준비하는 중요한 확장 실습이지만, **바로 다음 13일차의 Final Acceptance Test 대상은 지금까지 운영해 온 Raspberry Pi Edge AI 시스템**입니다.

따라서 12일차를 마칠 때 Raspberry Pi를 실험 중인 상태로 남겨 두면 안 됩니다.

```text
12일차

Raspberry Pi Tested Baseline
        ↓
Jetson Migration / Runtime 비교
        ↓
Jetson 결과·Migration 문서 기록
        ↓
Raspberry Pi 설정·Service 정상 복원
        ↓
Git / README / Report 정리

        ↓

13일차

Raspberry Pi Reboot
        ↓
사람이 Python을 직접 실행하지 않음
        ↓
systemd 자동실행 확인
        ↓
Camera → AI → Decision → Stabilization
        ↓
GPIO / Log
        ↓
FastAPI
        ↓
장애 복구
        ↓
Final Acceptance / Release
```

13일차에는 새로운 기능을 더 만드는 것이 아니라 1~12일차 결과를 최종 운영 기준으로 검증합니다. 따라서 오늘의 마지막 **복원 Gate**가 매우 중요합니다.

교과 14에서는 그 이후에 Jetson·산업용 Camera·제조 Vision AI로 확장합니다.

---

## 오늘의 학습 목표

수업이 끝나면 다음을 할 수 있어야 합니다.

```text
1. Raspberry Pi와 Jetson의 역할 차이를 Edge AI 관점에서 설명한다.
2. 장치가 바뀌어도 재사용하기 쉬운 Module과 바뀌기 쉬운 Module을 구분한다.
3. CPU와 GPU의 역할을 구분한다.
4. JetPack · CUDA · TensorRT의 위치를 설명한다.
5. ONNX와 TensorRT가 같은 개념이 아님을 설명한다.
6. Jetson의 Hostname · IP · Architecture · Jetson Linux 상태를 확인한다.
7. Raspberry Pi의 .venv를 복사하지 않고 Jetson에서 실행환경을 다시 준비한다.
8. 11일차 Unit Test를 Jetson에서 재사용하여 장치 독립 Logic을 검증한다.
9. 11일차 운영 Baseline이 실제 참조하는 ONNX와 Metadata를 Jetson으로 전송하고 Hash로 동일성을 확인한다.
10. 준비된 ONNX Runtime이 있으면 같은 Fixed Image의 Prediction을 확인한다.
11. TensorRT Engine을 대상 Jetson에서 생성해야 하는 이유를 설명한다.
12. trtexec Benchmark와 전체 End-to-End 성능이 같은 값이 아님을 설명한다.
13. Camera Layer와 Inference Layer의 Contract를 문서로 정리한다.
14. Raspberry Pi → Jetson Migration Map을 작성한다.
15. 제공 결과와 직접 측정값을 구분하고 비교 조건 차이를 기록한다.
16. 실행 결과가 어느 파일·명령·Module에서 만들어졌는지 역추적한다.
17. Mini Challenge에서 예측 → 설정 변경 → 기능 수정 → 오류 복구 → 증명을 수행한다.
18. 13일차를 위해 Raspberry Pi 운영 Baseline을 정상 상태로 복원한다.
```

---

## 오늘의 8시간 학습 흐름

| 학습 블록 | 권장 시간 | 무엇을 배우나요? | 끝나면 할 수 있어야 하는 것 |
|---|---:|---|---|
| 1 | 45분 | 11일차 Handoff · Tested Baseline · Migration 관점 | 오늘 무엇을 옮기고 무엇을 비교할지 설명할 수 있다 |
| 2 | 60분 | Module Portability · CPU/GPU · JetPack/CUDA/TensorRT | 재사용 영역과 변경 영역을 구분할 수 있다 |
| 3 | 75분 | Jetson Linux · Network · SSH · Python · 공통 Test | Jetson 실행환경을 확인하고 공통 Logic을 검증할 수 있다 |
| 4 | 85분 | ONNX 전송 · Hash · TensorRT Engine · Benchmark | 현재 운영 Baseline 모델을 Jetson Runtime으로 옮기고 결과를 기록할 수 있다 |
| 5 | 60분 | Camera / Inference Contract · Resource 관찰 | 장치별 구현과 공통 Interface를 구분할 수 있다 |
| 6 | 55분 | Raspberry Pi vs Jetson 비교 · Migration Map | 교과 13 구조가 Jetson에서 어떻게 이어지는지 설명할 수 있다 |
| 7 | 65분 | Mini Challenge | 결과 역추적·예측·설정 변경·기능 수정·오류 복구를 스스로 수행할 수 있다 |
| 8 | 35분 | 복원 · Report · README · Git · 복습 · 13일차 Handoff | Raspberry Pi를 Final Acceptance 가능한 상태로 남길 수 있다 |

총 480분을 기준으로 합니다. Jetson 부팅·Network·Engine Build 시간은 장비 상태에 따라 달라질 수 있으므로 실제 수업에서는 각 단계의 PASS 조건을 확인하면서 진행합니다.

---

## 오늘 사용할 실제 장비와 이전 일차 결과

```text
[Raspberry Pi — 11일차 Tested Baseline]

subject13_edge_ai/
├── configs/
│   ├── settings.yaml
│   └── device.local.yaml        ← 장치에 따라 존재
├── src/
│   ├── ai_preprocess.py
│   ├── camera_input.py
│   ├── decision.py
│   ├── event_logger.py
│   ├── gpio_output.py
│   ├── inference_onnx.py
│   ├── performance.py
│   ├── resource_monitor.py
│   ├── runtime_state.py
│   └── stabilizer.py
├── scripts/
│   ├── camera_index_probe.py
│   ├── camera_test.py
│   ├── predict_onnx.py
│   ├── benchmark_model_only.py
│   └── ...
├── tests/
│   ├── test_decision.py
│   ├── test_stabilizer.py
│   └── test_runtime_state.py
├── models/
│   ├── <현재 운영 Baseline ONNX>
│   └── <현재 운영 Baseline Metadata>
└── reports/
    └── day11_test_analysis.md
```

오늘 새로 추가하거나 완성하는 핵심 파일은 다음입니다.

```text
scripts/
└── jetson_env_check.py

docs/
├── day12_camera_adapter.md
├── day12_inference_contract.md
└── day12_migration_map.md

reports/
├── day12_rpi_jetson.md
└── day12_evidence/
    └── ...                       ← Challenge에서 증거를 남길 때 사용
```

Jetson 장비가 없는 경우에는 제공 자료의 값을 별도 CSV로 기록할 수 있습니다.

```text
reports/
├── day12_jetson_provided.csv
└── day12_rpi_result.csv
```

---

## 오늘 배울 기술 스택

| 기술 | 오늘 어디에 사용하나요? |
|---|---|
| Linux / SSH | Jetson 원격 접속과 환경 확인 |
| Git | 11일차 Tested Baseline과 12일차 변경 이력 구분 |
| Python venv | Jetson 전용 Python 실행환경 구성 |
| pytest | 11일차 장치 독립 Logic Test 재사용 |
| ONNX | 장치 사이에서 옮길 모델 교환 형식 |
| ONNX Runtime | 준비된 경우 Jetson에서 Baseline ONNX Prediction 확인 |
| JetPack | Jetson Software Stack 이해 |
| CUDA | NVIDIA GPU 연산 기반 이해 |
| TensorRT / `trtexec` | ONNX → Engine 변환과 GPU Model-only Benchmark |
| `tegrastats` | Jetson CPU·GPU·Memory·Temperature·Power 상태 관찰 |
| SHA-256 | PC와 Jetson 모델 파일 동일성 확인 |
| Markdown / CSV | Migration·성능·환경·판단 근거 기록 |

---

## 실습 전에 자신의 환경값을 적어두기

교재의 IP·사용자 이름·Port는 예시값으로 고정하지 않습니다.

```text
Raspberry Pi 사용자 이름 : ______________________________
Raspberry Pi Hostname    : ______________________________
Raspberry Pi IPv4        : ______________________________
Raspberry Pi API Port    : ______________________________

Jetson 사용자 이름       : ______________________________
Jetson Hostname          : ______________________________
Jetson IPv4              : ______________________________
Jetson Architecture      : ______________________________

Jetson Project Path      : ______________________________
Jetson Python Path       : ______________________________

JetPack / Jetson Linux   : ______________________________
CUDA 확인 결과           : ______________________________
TensorRT / trtexec       : ______________________________

USB Camera를 사용할 경우 Camera Index : __________________
```

이 교재에서는 다음 표기를 사용합니다.

```text
<RPI_USER>
→ 현재 Raspberry Pi 사용자 이름

<RPI_IP>
→ 현재 Raspberry Pi IPv4

<API_PORT>
→ configs/settings.yaml에서 확인한 API Port

<JETSON_USER>
→ 현재 Jetson 사용자 이름

<JETSON_IP>
→ 현재 Jetson IPv4

<REPOSITORY_URL>
→ 기관에서 허용된 내부 GitLab/Gitea 등 Repository 주소
```

IP가 기억과 다르면 예전 값을 반복해서 사용하지 않습니다.

Raspberry Pi:

```bash
hostname
hostname -I
```

Jetson:

```bash
hostname
hostname -I
```

여러 IPv4가 보이면 실제 PC와 통신하는 Network Interface의 주소를 사용합니다.

---

## 오늘의 전체 실습 흐름

```text
11일차 Tested Baseline 확인
        ↓
PC / Raspberry Pi HEAD 비교
        ↓
day11-rpi-tested-baseline Tag
        ↓
Module Portability 분류
        ↓
CPU / GPU
        ↓
JetPack / CUDA / TensorRT
        ↓

[Jetson 환경]

Power / Boot
        ↓
Hostname / IP / Architecture
        ↓
Jetson Linux / JetPack 확인
        ↓
SSH
        ↓
같은 subject13_edge_ai Source
        ↓
Jetson 전용 .venv
        ↓
11일차 Unit Test 재사용
        ↓
jetson_env_check.py
        ↓

[Model 이동]

11일차 운영 Baseline이 실제 참조하는
ONNX + Metadata
        ↓
SCP 또는 허용된 내부 전달
        ↓
SHA-256 동일성 확인
        ↓
ONNX Runtime 경로가 준비된 경우
Fixed Image Prediction
        ↓
TensorRT 사용 가능
        ↓
ONNX → TensorRT Engine
        ↓
trtexec Model-only Benchmark
        ↓
tegrastats와 자원 상태 관찰
        ↓

[Migration 설계]

Camera Contract
        ↓
Inference Contract
        ↓
재사용 Module / 변경 Module
        ↓
Raspberry Pi ↔ Jetson 비교
        ↓
Migration Map
        ↓
Mini Challenge
        ↓
Challenge 변경 복원
        ↓

[13일차 Handoff]

Raspberry Pi Config 복원
        ↓
Edge Service active + enabled
        ↓
API Service active + enabled
        ↓
/health 정상
        ↓
Git / README / Report
        ↓
13일차 Final Acceptance
```

---

## 오늘의 성공 기준

```text
11일차 Tested Baseline 확인
        ↓
장치 독립 / 장치 종속 Module 구분
        ↓
Jetson Hostname / IP / Architecture 확인
        ↓
Jetson Python 환경 확인
        ↓
장치 독립 Unit Test PASS
        ↓
ONNX + Metadata 전송 확인
        ↓
Hash 동일성 확인
        ↓
TensorRT 사용 가능 여부 확인
        ↓
Engine 생성 또는 제공 자료 분석
        ↓
Benchmark 조건과 결과 기록
        ↓
Camera / Inference Contract 작성
        ↓
Migration Map 완성
        ↓
Mini Challenge 완료
        ↓
Raspberry Pi 운영 Baseline 복원
        ↓
두 Service active + enabled
        ↓
/health 정상
        ↓
Git / README / Report 완료
        ↓
13일차 Handoff PASS
```

---

# PART A. 11일차 Tested Baseline을 기준점으로 고정하기

# 1. 11일차까지의 Raspberry Pi 상태 확인하기

12일차는 Jetson 실습부터 바로 시작하지 않습니다. 먼저 **11일차에서 검증한 Raspberry Pi 상태가 실제로 그대로 남아 있는지** 확인합니다.

PC에서 현재 Raspberry Pi의 주소를 확인한 뒤 접속합니다.

```bash
ssh <RPI_USER>@<RPI_IP>
```

접속이 되지 않으면 IP를 추측하지 않습니다. Raspberry Pi에서 직접 확인할 수 있다면 다음 명령을 사용합니다.

```bash
hostname
hostname -I
```

프로젝트로 이동합니다.

```bash
cd ~/ai_vision/subject13_edge_ai
```

현재 장치와 경로를 확인합니다.

```bash
whoami
hostname
pwd
```

가상환경을 활성화합니다.

```bash
source .venv/bin/activate
```

11일차 Unit Test를 다시 확인합니다.

```bash
python -m pytest tests -v
```

현재 API Port도 고정값으로 외우지 않고 설정에서 확인합니다.

```bash
grep -E "^api_port:"   configs/settings.yaml   configs/device.local.yaml 2>/dev/null
```

두 Service:

```bash
systemctl status \
  subject13-edge.service \
  subject13-api.service \
  --no-pager
```

자동실행 상태:

```bash
systemctl is-enabled subject13-edge.service
systemctl is-enabled subject13-api.service
```

Raspberry Pi 내부에서 API를 확인합니다.

```bash
curl http://127.0.0.1:<API_PORT>/health
```

### 예상 결과와 해석

정확한 출력은 환경에 따라 달라지지만 다음 조건을 확인합니다.

```text
pytest
→ PASS

subject13-edge.service
→ active

subject13-api.service
→ active

두 Service
→ enabled

/health
→ API와 Edge Runtime이 정상
```

하나라도 실패하면 Jetson 실습으로 넘어가기 전에 11일차 Baseline부터 복구합니다.

```text
오류
        ↓
systemctl status
        ↓
journalctl
        ↓
Config / Path / Camera / Model 확인
        ↓
수정
        ↓
재실행
        ↓
11일차 Baseline PASS
```

### Guided Lab — PC와 Raspberry Pi가 같은 Source를 보고 있는지 확인하기

오늘은 PC에서 Source를 관리하고 Raspberry Pi·Jetson으로 배포할 수 있습니다. 먼저 PC와 Raspberry Pi의 Commit이 같은지 확인합니다.

PC 프로젝트에서:

```bash
git rev-parse HEAD
```

Raspberry Pi 프로젝트에서:

```bash
git rev-parse HEAD
```

두 값이 같다면 같은 Commit을 기준으로 비교하기 쉽습니다.

다르다면 무조건 `git pull`부터 하지 않습니다.

```text
Remote가 있는가?
        ↓
있음
→ git status 확인
→ 필요한 변경 Commit
→ 허용된 내부 Remote로 동기화

없음
→ Local Git 이력 확인
→ 수업에서 정한 SCP / 내부 전달 방식으로 Source 동기화
```

장치별 설정인 `configs/device.local.yaml`은 다른 장치의 파일로 덮어쓰지 않습니다.

---

# 1-1. 11일차 최종 운영 Model Contract 확인하기

Jetson으로 무엇을 옮길지 결정하기 전에 Raspberry Pi가 **현재 실제 운영에서 어떤 ONNX와 Metadata를 읽는지** 확인합니다.

파일명을 기억으로 적지 않고 `config_loader.py`가 병합한 실제 Config를 읽습니다.

Raspberry Pi 프로젝트 루트에서:

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
):
    print(f"{key}: {config[key]}")

for key in (
    "onnx_model_path",
    "onnx_meta_path",
):
    path = Path(str(config[key]))

    print(
        f"{key} exists: {path.exists()} "
        f"absolute: {path.is_absolute()}"
    )

meta_path = Path(str(config["onnx_meta_path"]))

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

### 예상 결과와 해석

예를 들어 11일차에서 별도 Model을 채택하지 않았다면 다음과 비슷할 수 있습니다.

```text
onnx_model_path: models/day07_tiny_cnn.onnx
onnx_meta_path: models/day07_tiny_cnn.meta.json
ai_confidence_threshold: <현재 값>
onnx_model_path exists: True absolute: False
onnx_meta_path exists: True absolute: False
```

11일차에서 다른 후보를 공식 Baseline으로 채택했다면 파일명은 달라질 수 있습니다.

```text
onnx_model_path: models/day08_tiny_cnn_64.onnx
onnx_meta_path: models/day08_tiny_cnn_64.meta.json
...
```

둘 중 어느 이름이 보이느냐가 정답이 아닙니다.

핵심은:

```text
현재 Raspberry Pi 운영 Config가 참조하는 Model
+
그 Model과 짝이 되는 Metadata
+
실제 파일 존재
```

가 일치하는 것입니다.

### 11일차 Report와 실제 운영 Config가 같은지 확인하기

`reports/day11_test_analysis.md`의 `Final Decision`에서 실제 채택 여부를 다시 확인합니다.

```bash
grep -n -E \
  "Final Decision|Model:|Input Size:|Confidence Threshold:" \
  reports/day11_test_analysis.md
```

다음 두 경우를 구분합니다.

```text
Case A
11일차에서 후보를 비교했지만
운영 Baseline으로 채택하지 않음
        ↓
기존 10일차 운영 Model 유지
        ↓
정상

Case B
11일차 Report에는 새 Model을
운영 Baseline으로 채택했다고 기록
        ↓
하지만 현재 Config는 예전 Model을 참조
        ↓
Handoff 불일치
        ↓
Jetson 실습 전에 먼저 정리
```

`Case B`라면 어느 파일이 맞는지 추측해서 진행하지 않습니다.

```text
11일차 Final Decision 확인
        ↓
실제 System Test를 통과한 조합 확인
        ↓
onnx_model_path / onnx_meta_path /
ai_confidence_threshold 일치 확인
        ↓
필요한 경우 운영 Config 정리
        ↓
Raspberry Pi Service 재시작
        ↓
/health + System Test 확인
        ↓
그 상태를 Baseline으로 사용
```

### Project-relative 경로를 권장하는 이유

Model 경로가 다음처럼 Raspberry Pi 절대경로로 저장되어 있다면:

```text
/home/어떤사용자/ai_vision/subject13_edge_ai/models/...
```

Jetson에서는 같은 절대경로가 존재하지 않을 수 있습니다.

교과 13의 배포 자산은 가능한 한 다음처럼 **프로젝트 Root 기준 상대경로**를 사용하는 편이 이식하기 쉽습니다.

```text
models/파일명.onnx
models/파일명.meta.json
```

절대경로가 의도적으로 필요하다면 Jetson Config에서 대상 장치 경로를 별도로 맞추고 그 이유를 Report에 기록합니다.

### 오늘 사용할 Baseline 값을 적어두기

위 명령에서 확인한 실제 값을 적습니다.

```text
<BASELINE_ONNX>
→ ___________________________________________

<BASELINE_META>
→ ___________________________________________

<BASELINE_INPUT_SIZE>
→ Metadata의 image_size: ____________________

<BASELINE_THRESHOLD>
→ ___________________________________________
```

이후 교재의 `<BASELINE_ONNX>`, `<BASELINE_META>`, `<BASELINE_INPUT_SIZE>`는 **자신이 실제 확인한 값으로 바꾸어 사용**합니다.

---

# 2. 12일차 시작 전 `day11-rpi-tested-baseline` 만들기

11일차 마지막 정상 상태를 Git Tag로 표시하면 Jetson 확장 중 문제가 생겼을 때 **“어디가 검증된 Raspberry Pi 기준점이었는가?”**를 명확히 찾을 수 있습니다.

먼저 Source 관리 기준으로 사용하는 PC 프로젝트에서 현재 Branch를 확인합니다.

```bash
git branch --show-current
git status
```

11일차에서 `main`으로 정리했다면 다음과 비슷해야 합니다.

```text
main
nothing to commit, working tree clean
```

변경사항이 남아 있다면 의미를 확인하고 먼저 Commit 또는 복원합니다. 내용을 확인하지 않고 `git restore .`를 실행하지 않습니다.

Tag 존재 여부:

```bash
git tag --list day11-rpi-tested-baseline
```

아무것도 출력되지 않고 현재 `main`이 11일차 검증 완료 Commit이라면:

```bash
git tag day11-rpi-tested-baseline
```

확인:

```bash
git show \
  --stat \
  day11-rpi-tested-baseline
```

이 Tag의 의미는 다음과 같습니다.

```text
Raspberry Pi
→ Unit Test 완료
→ Integration / System Test 완료
→ Failure Analysis 완료
→ 운영 설정 선택 근거 기록
→ systemd / FastAPI 정상
```

### 이 Tag는 왜 12일차에 필요할까요?

오늘 Jetson에서 Source·Runtime·환경이 달라져도 다음 기준을 잃지 않기 위해서입니다.

```text
"Jetson에서 안 된다."
        ↓
Jetson만의 문제인가?
        ↓
아니면 원래 Source가 이미 깨져 있었는가?
```

`day11-rpi-tested-baseline`이 있으면 두 상황을 구분하는 기준점이 됩니다.

---

# PART B. 무엇을 재사용하고 무엇을 바꿀지 먼저 이해하기

# 3. 먼저 코드에서 장치에 종속되는 부분 찾기

Jetson을 만지기 전에 현재 프로젝트를 봅니다.

```text
src/
├── ai_preprocess.py
├── camera_input.py
├── decision.py
├── event_logger.py
├── gpio_output.py
├── inference_onnx.py
├── performance.py
├── resource_monitor.py
├── runtime_state.py
├── stabilizer.py
└── ...
```

이 파일들을 다음 두 그룹으로 나눕니다.

```text
A. 장치가 바뀌어도
   그대로 사용하기 쉬운 코드

B. 장치나 Runtime에 따라
   바뀔 가능성이 큰 코드
```

---

# 4. 재사용하기 쉬운 모듈 표시하기

다음 파일부터 확인합니다.

```text
decision.py
stabilizer.py
event_logger.py
runtime_state.py
performance.py
```

이 코드들은 다음과 같은 역할을 합니다.

```text
Decision
→ Class / Confidence를 운영 상태로 변환

Stabilizer
→ Voting / Debounce / Hold

Event Logger
→ 상태 변화 기록

Runtime State
→ 최신 장치 상태 기록

Performance
→ Mean / P50 / P95 계산
```

이 기능들은 Raspberry Pi의 GPIO 번호나 Camera Driver에 직접 의존하지 않습니다.

따라서 Jetson에서도 비교적 그대로 재사용하기 쉽습니다.

---

# 5. 변경 가능성이 큰 모듈 표시하기

다음 파일은 장치에 따라 바뀔 가능성이 큽니다.

```text
camera_input.py
inference_onnx.py
gpio_output.py
resource_monitor.py
```

이유:

```text
camera_input.py
→ Raspberry Pi에서는 일반 USB Camera
→ Jetson에서는 산업용 Camera SDK가 들어갈 수 있음

inference_onnx.py
→ Raspberry Pi에서는 ONNX Runtime CPU
→ Jetson에서는 TensorRT GPU Runtime으로 교체 가능

gpio_output.py
→ 실제 프로젝트의 출력장치 요구에 따라 달라질 수 있음

resource_monitor.py
→ Raspberry Pi와 Jetson의 GPU / 온도 / 전력 확인 방법이 다름
```

---

# 6. 오늘의 Migration Report를 먼저 만들기

## 이 파일은 왜 필요할까요?

12일차에는 환경 확인, Model 전송, Hash, TensorRT, 성능 결과, 재사용 Module, 변경 Module처럼 기록할 항목이 많습니다.

마지막에 기억으로 정리하면 다음 문제가 생길 수 있습니다.

```text
직접 측정한 값인지
제공 자료의 값인지
구분하기 어려움

어떤 명령으로 확인했는지
기억이 나지 않음

Raspberry Pi와 Jetson의
측정 조건이 달랐는데
숫자만 비교할 수 있음
```

그래서 실습을 시작할 때부터 하나의 Report를 만들고 각 단계가 끝날 때 바로 채웁니다.

파일:

```text
reports/day12_rpi_jetson.md
```

처음에는 다음 뼈대만 작성합니다.

````md
# Day 12 Raspberry Pi → Jetson

## Environment

Raspberry Pi Hostname:

Raspberry Pi Architecture:

Raspberry Pi Runtime:

Jetson Hostname:

Jetson Architecture:

Jetson Linux / JetPack:

CUDA:

TensorRT:

## Module Portability

| Module | Raspberry Pi 역할 | Jetson에서 | 이유 |
|---|---|---|---|
| config_loader.py | 설정 읽기 | 재사용 |  |
| decision.py | 운영 판정 | 재사용 |  |
| stabilizer.py | 시간적 안정화 | 재사용 |  |
| event_logger.py | Event 기록 | 재사용 |  |
| runtime_state.py | 상태 공유 | 재사용 |  |
| performance.py | Latency 통계 | 재사용 |  |
| camera_input.py | USB Camera | 변경 가능 |  |
| inference_onnx.py | ONNX Runtime CPU | 변경 가능 |  |
| gpio_output.py | GPIO | 프로젝트별 변경 |  |
| resource_monitor.py | CPU / Temp | 변경 |  |

## Day 11 Baseline Deployment Contract

Adopted ONNX:

Adopted Metadata:

Input Size:

Operational Threshold:

Source:
- `day11-rpi-tested-baseline`
- `configs/settings.yaml` + 필요한 경우 `configs/device.local.yaml`

## Baseline Model Evidence

PC ONNX SHA-256:

Jetson ONNX SHA-256:

PC Metadata SHA-256:

Jetson Metadata SHA-256:

## Runtime / Benchmark

## Camera Contract

## Inference Contract

## Migration Map

## Mini Challenge

## 13일차 Handoff
````

지금 빈칸을 억지로 채우지 않습니다. 실제로 확인한 값만 그때그때 기록합니다.

---

# 7. Raspberry Pi와 Jetson의 역할을 실제 Pipeline으로 비교하기

Raspberry Pi에서 만든 구조:

```text
USB Camera
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
```

Jetson으로 이동하면 다음처럼 바뀔 수 있습니다.

```text
Camera
     ↓
Camera Driver / SDK
     ↓
Preprocess
     ↓
ONNX / TensorRT
     ↓
NVIDIA GPU
     ↓
AI Result
     ↓
Decision
     ↓
Log / API / Output
```

전체 구조가 사라지는 것이 아니라 **입력 계층과 추론 Runtime이 주로 달라집니다.**

---

# 8. CPU와 GPU의 역할을 실행 흐름으로 구분하기

Raspberry Pi 기본 실습:

```text
CPU
→ Camera 처리
→ Preprocess
→ ONNX 추론
→ Decision
```

Jetson:

```text
CPU
→ OS
→ Camera 제어
→ 일부 전처리
→ Program Logic

GPU
→ 병렬 AI 연산 가속
```

모든 코드를 GPU가 실행하는 것은 아닙니다.

Jetson에서도 다음 기능은 CPU에서 동작할 수 있습니다.

```text
Python Logic
Config
Logging
FastAPI
systemd
File I/O
일부 Camera 처리
```

GPU는 특히 AI 연산을 가속하는 데 사용됩니다.

---

# 9. JetPack·CUDA·TensorRT의 위치 보기

Jetson 환경을 다음처럼 생각합니다.

```text
Jetson Hardware
        ↓
Ubuntu / Jetson Linux
        ↓
JetPack
        │
        ├─ CUDA
        ├─ TensorRT
        └─ NVIDIA 관련 Library
```

오늘 외워야 할 세부 버전 목록이 목적은 아닙니다.

역할만 구분합니다.

```text
JetPack
→ Jetson 개발환경을 구성하는 NVIDIA Software Stack

CUDA
→ NVIDIA GPU 연산을 사용할 수 있는 기반

TensorRT
→ 학습된 AI Model을 NVIDIA GPU에서 빠르게 추론하도록 최적화하는 Runtime
```

---

# 10. ONNX와 TensorRT의 역할을 구분하기

6~7일차:

```text
PyTorch
→ ONNX
```

여기서 ONNX는 모델을 다른 Runtime과 장치로 옮기기 쉽게 하는 **교환 형식**으로 사용했습니다.

Jetson에서는 다음 단계가 추가될 수 있습니다.

```text
PyTorch
→ ONNX
→ TensorRT Engine
→ NVIDIA GPU Inference
```

구분:

```text
ONNX
→ 모델 교환 / 배포 형식

TensorRT
→ NVIDIA GPU 추론 최적화 Runtime
```

따라서:

```text
ONNX = TensorRT
```

가 아닙니다.

---

# 11. 왜 Raspberry Pi에서 TensorRT를 사용하지 않았나요?

TensorRT는 NVIDIA GPU를 대상으로 하는 Runtime입니다.

교과 13에서 사용한 Raspberry Pi의 기본 추론 구조는:

```text
Raspberry Pi CPU
→ ONNX Runtime CPU
```

였습니다.

따라서 오늘의 핵심 비교는:

```text
Raspberry Pi
CPU 중심

Jetson
NVIDIA GPU 활용 가능
```

입니다.

---

# PART C. Jetson 환경을 직접 확인하고 같은 Source를 준비하기

# 12. Jetson 장비가 준비되어 있다면 여기부터 실제 실습하기

이 절부터는 **Jetson 장비가 준비된 경우** 진행합니다.

장비가 준비되지 않았다면 `# 32. Jetson 장비가 없는 경우`로 이동합니다.

Jetson에 전원과 Network를 연결하고 부팅합니다.

처음에는 모니터가 연결되어 있어도 괜찮습니다.

목표는 최종적으로 PC에서 SSH로 관리하는 것입니다.

---

# 13. Jetson의 Hostname과 IP 확인하기

Jetson Terminal에서:

```bash
hostname
```

예:

```text
jetson13-01
```

IP:

```bash
hostname -I
```

Network:

```bash
ip addr
```

Python:

```bash
python3 --version
```

Architecture:

```bash
uname -m
```

Jetson에서는 다음과 비슷한 Architecture가 보일 수 있습니다.

```text
aarch64
```

---

# 14. Jetson Linux / JetPack / CUDA / TensorRT 상태 확인하기

오늘은 버전 번호를 외우는 것이 아니라 **현재 장비에 무엇이 준비되어 있는지 직접 확인**합니다.

Jetson Linux Release 파일이 존재하는 환경에서는:

```bash
cat /etc/nv_tegra_release
```

파일이 없다면:

```text
No such file or directory
```

가 나올 수 있습니다. 이 경우 즉시 OS를 다시 설치하지 않고 현재 장비가 실제 Jetson인지, 어떤 Image가 설치되어 있는지 확인합니다.

JetPack Meta Package가 설치된 환경:

```bash
dpkg-query \
  --show \
  nvidia-jetpack
```

Meta Package 방식이 아니라 구성요소가 개별 설치된 환경에서는 원하는 출력이 없을 수도 있습니다.

CUDA Compiler가 PATH에 있다면:

```bash
command -v nvcc
nvcc --version
```

PATH에서 찾지 못하면 일반적인 CUDA 경로도 확인할 수 있습니다.

```bash
ls -l /usr/local/cuda/bin/nvcc
```

TensorRT 실행도구:

```bash
command -v trtexec
```

PATH에 없다면 일반적인 Jetson 설치 위치도 확인합니다.

```bash
ls -l /usr/src/tensorrt/bin/trtexec
```

현재 결과를 `reports/day12_rpi_jetson.md`에 그대로 기록합니다.

```text
Jetson Linux:

JetPack:

CUDA / nvcc:

TensorRT / trtexec:
```

특정 명령이 없다고 해서 임의 Package를 바로 설치하지 않습니다. 수업 장비의 사전 검증된 JetPack 환경과 설치 정책을 기준으로 진행합니다.

---

# 15. Jetson GPU와 자원 상태 확인하기

Jetson에서는 `tegrastats`를 이용하여 자원 상태를 관찰할 수 있습니다.

먼저 위치를 확인합니다.

```bash
command -v tegrastats
```

PATH에 없다면 일반적인 위치를 확인합니다.

```bash
ls -l /usr/bin/tegrastats
```

실제 실행 경로가 확인되면 실행합니다.

PATH에서 실행 가능한 경우:

```bash
tegrastats
```

몇 줄을 확인한 뒤 `Ctrl+C`로 종료합니다.

환경에 따라 다음과 같은 정보를 볼 수 있습니다.

```text
CPU
Memory
GPU 사용상태
Temperature
Power 관련 상태
```

정확한 항목 이름과 출력 형식은 Jetson Software 버전에 따라 달라질 수 있습니다.

오늘은 숫자의 좋고 나쁨을 바로 판단하지 않고 **현재 상태를 읽는 방법**부터 익힙니다.

---

# 16. PC에서 Jetson으로 SSH 접속하기

2일차 Raspberry Pi에서 사용한 원격 관리 구조를 Jetson에서도 다시 사용합니다.

```text
PC
        ↓
Network
        ↓
SSH
        ↓
Jetson
```

먼저 Jetson에서 실제 값을 확인합니다.

```bash
whoami
hostname
hostname -I
```

PC에서 접속합니다.

```bash
ssh <JETSON_USER>@<JETSON_IP>
```

접속 후:

```bash
whoami
hostname
pwd
```

PC Terminal인지 Jetson Terminal인지 헷갈리지 않도록 프롬프트와 `hostname`을 확인합니다.

### SSH가 실패하면 오류 종류부터 구분합니다

#### `Connection timed out`

```text
오류
→ 응답이 오지 않음

확인
→ Jetson Power / Boot
→ 현재 IP
→ PC와 Jetson Network
→ Firewall / 교육장 Network 정책

수정
→ 실제 도달 가능한 IP와 Network 상태 복구

재실행
→ ssh <JETSON_USER>@<JETSON_IP>
```

#### `Connection refused`

```text
오류
→ Host에는 도달했지만 SSH Service 연결 거부 가능

확인
→ Jetson의 SSH Service 상태
→ Port 설정

수정
→ 수업 장비의 SSH 설정 복구

재실행
```

#### `Permission denied`

```text
오류
→ 계정 또는 인증 문제

확인
→ 사용자 이름
→ 인증 방식
→ 비밀번호 / Key 정책

수정
→ 실제 계정과 허용된 인증방식 사용

재실행
```

#### `REMOTE HOST IDENTIFICATION HAS CHANGED!`

Jetson을 재설치하거나 같은 IP에 다른 장치가 연결되면 Host Key가 달라질 수 있습니다.

먼저 **실제로 접속하려는 장치가 맞는지 확인**합니다. 확인된 경우에만 PC의 이전 Host Key 항목을 정리합니다.

```bash
ssh-keygen -R <JETSON_IP>
```

그다음 다시 접속합니다.

SSH 실패 하나만으로 Jetson OS를 다시 설치하지 않습니다.

---

# 17. Jetson에는 새로운 프로젝트를 만들지 않고 같은 Source를 준비하기

Jetson에서도 `subject13_edge_ai`를 사용합니다.

```text
Raspberry Pi에서 검증한 Source 구조
        ↓
Jetson에 같은 Source 준비
        ↓
장치별 Runtime만 다르게 확인
```

상위 폴더:

```bash
mkdir -p ~/ai_vision
cd ~/ai_vision
```

## 방법 A — 허용된 내부 GitLab / Gitea Remote가 있는 경우

Repository가 아직 없다면:

```bash
git clone \
  <REPOSITORY_URL> \
  subject13_edge_ai
```

이미 Clone되어 있다면:

```bash
cd ~/ai_vision/subject13_edge_ai
git status
git pull
```

`git pull`은 실제 Remote가 연결되어 있고 기관 정책상 허용될 때만 사용합니다.

## 방법 B — Remote 없이 PC에서 Source를 전달하는 경우

먼저 Jetson에 프로젝트 폴더를 만듭니다.

```bash
mkdir -p ~/ai_vision/subject13_edge_ai
```

PC의 프로젝트 루트에서 **공통 Source만** 전송합니다.

```bash
scp -r \
  src \
  scripts \
  tests \
  systemd \
  <JETSON_USER>@<JETSON_IP>:~/ai_vision/subject13_edge_ai/
```

공통 설정을 넣을 폴더를 Jetson에 먼저 만듭니다.

```bash
ssh <JETSON_USER>@<JETSON_IP> \
  "mkdir -p ~/ai_vision/subject13_edge_ai/configs"
```

공통 설정:

```bash
scp \
  configs/settings.yaml \
  <JETSON_USER>@<JETSON_IP>:~/ai_vision/subject13_edge_ai/configs/
```

Root 파일:

```bash
scp \
  README.md \
  requirements.txt \
  .gitignore \
  <JETSON_USER>@<JETSON_IP>:~/ai_vision/subject13_edge_ai/
```

### 전송하지 않는 것

```text
.venv/
→ Jetson에서 다시 생성

configs/device.local.yaml
→ 장치마다 다르므로 Raspberry Pi 것을 복사하지 않음

models/*.onnx
→ 뒤에서 Hash 확인과 함께 별도 전송

data/
→ 필요한 Fixed Image만 뒤에서 별도 전송

logs/
→ 기존 장치 Runtime 기록이므로 복사하지 않음
```

### Source 준비 확인

Jetson:

```bash
cd ~/ai_vision/subject13_edge_ai
```

```bash
ls
ls src
ls scripts
ls tests
ls configs/settings.yaml
```

같은 프로젝트 이름을 사용하지만 **장치별 `.venv`, Local Config, Model Runtime 파일은 분리**합니다.

---

# 18. Raspberry Pi의 `.venv`를 Jetson으로 복사하지 않기

다음은 하지 않습니다.

```text
Raspberry Pi .venv
→ SCP
→ Jetson
```

이유:

```text
CPU Architecture
Package Binary
Runtime
OS Library
```

가 다를 수 있기 때문입니다.

Source Code는 공유하지만 가상환경은 장치별로 다시 만듭니다.

```text
Source
→ 공유

.venv
→ 장치별 생성
```

---

# 19. Jetson용 Python 환경 준비하기

## 왜 Raspberry Pi의 `.venv`를 그대로 복사하면 안 될까요?

Source Code는 같은 프로젝트를 사용하지만 가상환경에는 장치·OS·Python에 맞는 Binary Package가 들어갈 수 있습니다.

```text
Raspberry Pi .venv
        ↓
그대로 SCP
        ↓
Jetson
        ↓
Package / Binary / Runtime 충돌 가능
```

따라서 `.venv`는 **장치별로 다시 생성**합니다.

먼저 Jetson 프로젝트로 이동합니다.

```bash
cd ~/ai_vision/subject13_edge_ai
```

Python 버전:

```bash
python3 --version
```

`venv` 기능이 없다면 Ubuntu 계열에서는 다음 Package가 필요할 수 있습니다.

```bash
sudo apt update
sudo apt install -y python3-venv
```

수업 장비가 폐쇄망이거나 Package 설치가 제한되어 있다면 외부 설치를 임의로 시도하지 않습니다. 사전 준비된 Package 또는 내부 Mirror 기준을 사용합니다.

### 경로 A — 일반 가상환경

사전 검증된 Jetson 환경이 일반 Python Package만 필요하다면:

```bash
python3 -m venv .venv
```

### 경로 B — JetPack의 System Python Package를 함께 봐야 하는 경우

JetPack이 설치한 일부 Python Package를 가상환경에서 그대로 확인해야 하는 환경이라면:

```bash
python3 -m venv \
  --system-site-packages \
  .venv
```

`--system-site-packages`가 항상 더 좋은 것은 아닙니다. System Package가 가상환경에 노출되어 Package 충돌 가능성도 있으므로 **사전 검증된 수업 장비 기준**을 사용합니다.

가상환경을 활성화합니다.

```bash
source .venv/bin/activate
```

확인:

```bash
which python
python --version
```

예상 형태:

```text
.../subject13_edge_ai/.venv/bin/python
```

### requirements를 설치하기 전에 생각할 점

Raspberry Pi의 `requirements.txt`에는 Jetson에서 그대로 설치하기 어려운 Package가 포함될 수도 있습니다. 특히 Architecture나 NVIDIA Runtime과 관련된 Package는 환경별 설치방식이 다를 수 있습니다.

따라서 다음 순서로 진행합니다.

```text
Python / venv 확인
        ↓
수업 장비에 이미 준비된 NVIDIA Runtime 확인
        ↓
공통 Python Package 확인
        ↓
필요한 Package만 사전 검증 방식으로 설치
```

준비된 수업환경에서 전체 `requirements.txt`가 검증되어 있다면:

```bash
python -m pip install -r requirements.txt
```

설치 오류가 발생했을 때 무조건 버전을 바꾸거나 임의의 Wheel을 인터넷에서 설치하지 않습니다.

---

# 20. Guided Lab — 장치 독립 Logic을 Jetson에서 먼저 검증하기

GPU 추론부터 시작하지 않습니다.

11일차에 만든 다음 Test 파일을 **그대로 다시 사용**합니다.

```text
tests/test_decision.py
tests/test_stabilizer.py
tests/test_runtime_state.py
```

이 Test들은 Camera나 GPIO를 직접 열지 않고 Logic에 값을 넣어 검증합니다.

실행 전에 연결을 다시 봅니다.

```text
tests/test_decision.py
→ src/decision.py

tests/test_stabilizer.py
→ src/stabilizer.py

tests/test_runtime_state.py
→ src/runtime_state.py
```

Jetson의 가상환경에서:

```bash
python -m pytest \
  tests/test_decision.py \
  tests/test_stabilizer.py \
  tests/test_runtime_state.py \
  -v
```

### 예상 결과와 해석

11일차와 같은 Source가 준비되어 있다면 핵심은 모두 PASS입니다.

```text
Decision
→ PASS

Stabilizer
→ PASS

Runtime State
→ PASS
```

이 결과는 다음을 의미합니다.

```text
Jetson에서도
장치 독립 Logic은
같은 Test로 검증 가능
```

하지만 다음을 의미하지는 않습니다.

```text
Camera PASS
TensorRT PASS
GPIO PASS
GPU PASS
```

### 오류 → 원인 확인 → 수정 → 재실행

`No module named pytest`:

```text
오류
→ pytest가 현재 Python에 없음

원인 확인
→ which python
→ python -m pytest --version

수정
→ 수업에서 준비한 Package / 내부 Mirror / 검증된 설치 방법 사용

재실행
→ 같은 pytest 명령
```

`ModuleNotFoundError: No module named 'src'`:

```text
오류
→ Project Module을 찾지 못함

원인 확인
→ pwd
→ ls
→ 프로젝트 루트에서 실행 중인지 확인

수정
→ cd ~/ai_vision/subject13_edge_ai

재실행
→ python -m pytest ...
```

Test가 실제로 `FAILED`라면 Source Commit이 Raspberry Pi 기준과 같은지도 확인합니다.

```bash
git rev-parse HEAD
```

---

# 21. Guided Lab — Jetson 환경 확인 Script 만들기

## 이 파일은 왜 필요할까요?

앞에서는 `hostname`, `uname -m`, `which trtexec` 같은 명령을 하나씩 실행했습니다. 실제 장치를 비교할 때는 **같은 확인 항목을 반복해서 기록할 수 있는 작은 Script**가 있으면 편리합니다.

이 파일은 AI 추론을 하지 않습니다.

```text
Jetson 환경
        ↓
Hostname
Architecture
Python
Jetson Linux
nvcc
trtexec
tegrastats
JetPack Package
        ↓
한 번에 출력
```

파일:

```text
scripts/jetson_env_check.py
```

## 의사코드

```text
현재 Hostname / Architecture / Python 버전 확인
        ↓
/etc/nv_tegra_release가 있는지 확인
        ↓
있으면 내용 읽기
없으면 "not found"
        ↓
nvcc 실행파일 위치 찾기
        ↓
PATH에 없으면 일반적인 Jetson 경로도 확인
        ↓
trtexec 위치 찾기
        ↓
tegrastats 위치 찾기
        ↓
nvidia-jetpack Package 정보 명령 실행
        ↓
명령이 없거나 실패해도
전체 Script가 중단되지 않도록
오류 내용을 문자열로 기록
```

## 코드

```python
import platform
import shutil
import subprocess
from pathlib import Path


def run_command(
    command: list[str],
) -> str:
    try:
        result = subprocess.run(
            command,
            capture_output=True,
            text=True,
            check=False,
        )

        text = (
            result.stdout.strip()
            or result.stderr.strip()
        )

        return text or "(no output)"

    except Exception as error:
        return (
            f"{type(error).__name__}: "
            f"{error}"
        )


def find_executable(
    name: str,
    fallback_paths: list[str],
) -> str:
    found = shutil.which(name)

    if found:
        return found

    for raw_path in fallback_paths:
        path = Path(raw_path)

        if path.is_file():
            return str(path)

    return "not found"


def main():
    print(
        "=== Jetson Environment Check ==="
    )

    print(
        f"Hostname      : "
        f"{platform.node()}"
    )

    print(
        f"Architecture  : "
        f"{platform.machine()}"
    )

    print(
        f"Python        : "
        f"{platform.python_version()}"
    )

    print()

    release_path = Path(
        "/etc/nv_tegra_release"
    )

    if release_path.exists():
        print("[Jetson Linux]")
        print(
            release_path.read_text(
                encoding="utf-8"
            ).strip()
        )
    else:
        print(
            "[Jetson Linux] "
            "release file not found"
        )

    nvcc_path = find_executable(
        "nvcc",
        [
            "/usr/local/cuda/bin/nvcc",
        ],
    )

    trtexec_path = find_executable(
        "trtexec",
        [
            "/usr/src/tensorrt/bin/trtexec",
        ],
    )

    tegrastats_path = find_executable(
        "tegrastats",
        [
            "/usr/bin/tegrastats",
        ],
    )

    print()
    print(f"nvcc          : {nvcc_path}")
    print(f"trtexec       : {trtexec_path}")
    print(f"tegrastats    : {tegrastats_path}")

    print()
    print("[nvidia-jetpack]")
    print(
        run_command(
            [
                "dpkg-query",
                "--show",
                "nvidia-jetpack",
            ]
        )
    )


if __name__ == "__main__":
    main()
```

## 실행 전에 파일 연결을 확인합니다

```text
scripts/jetson_env_check.py
        ↓
Python 표준 Library
platform / shutil / subprocess / pathlib
        ↓
Jetson OS 파일과 명령
/etc/nv_tegra_release
dpkg-query
nvcc
trtexec
tegrastats
        ↓
터미널 출력
```

이 Script는 `src/`의 AI Pipeline을 호출하지 않습니다. **장치 환경을 확인하는 독립적인 진단 Script**입니다.

## 실행

Jetson 프로젝트 루트에서:

```bash
python -m scripts.jetson_env_check
```

## 예상 결과 형태

실제 버전과 경로는 장비마다 다릅니다.

```text
=== Jetson Environment Check ===
Hostname      : <실제 Hostname>
Architecture  : aarch64
Python        : <실제 Python 버전>

[Jetson Linux]
<실제 release 정보>

nvcc          : <경로 또는 not found>
trtexec       : <경로 또는 not found>
tegrastats    : <경로 또는 not found>

[nvidia-jetpack]
<Package 정보 또는 오류 메시지>
```

`not found`가 하나 보였다고 바로 OS를 다시 설치하지 않습니다.

```text
not found
        ↓
PATH에 없는가?
        ↓
Fallback 경로에도 없는가?
        ↓
수업 장비의 사전 설치 상태 확인
        ↓
필요한 경우에만 준비된 설치 절차 사용
```

## 코드 리뷰 — 출력은 어느 코드에서 만들어졌을까요?

실행 후 `scripts/jetson_env_check.py`를 다시 엽니다.

```text
Hostname
→ platform.node()

Architecture
→ platform.machine()

Python
→ platform.python_version()

Jetson Linux
→ /etc/nv_tegra_release

nvcc / trtexec / tegrastats
→ find_executable()

JetPack Package 출력
→ run_command()
→ dpkg-query
```

다음 질문에 답합니다.

```text
1. PATH에서 명령을 찾는 함수는 무엇인가?
2. trtexec가 PATH에 없을 때 어떤 경로를 추가로 확인하는가?
3. /etc/nv_tegra_release가 없으면 Script가 종료되는가?
4. dpkg-query가 실패해도 전체 프로그램을 계속 실행할 수 있는 이유는?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
1. shutil.which()를 사용하는 find_executable()
2. /usr/src/tensorrt/bin/trtexec
3. 종료되지 않는다. "release file not found"를 출력하고 계속 진행한다.
4. run_command()가 예외를 잡아 문자열로 반환하고,
   subprocess.run()도 check=False로 실행하기 때문이다.
```

</details>

---

# 22. Jetson 환경 결과 기록하기

`reports/day12_rpi_jetson.md`에 추가합니다.

```md
## Jetson Environment

Hostname:

Architecture:

Python:

Jetson Linux:

JetPack:

CUDA:

TensorRT:

tegrastats:

## Raspberry Pi Environment

Architecture:

Runtime:

Inference Device:

## Difference

```

실제 출력값만 적습니다.

---

# PART D. 11일차 운영 Baseline ONNX를 Jetson으로 옮기고 TensorRT까지 연결하기

# 23. Guided Lab — 현재 운영 Baseline ONNX와 Metadata를 Jetson으로 전송하기

## 왜 새 모델을 만들지 않을까요?

12일차의 목적은 모델을 다시 학습하는 것이 아니라 **같은 배포 자산이 장치가 바뀌었을 때 어떻게 이어지는지** 확인하는 것입니다.

12일차에서는 **11일차의 Tested Baseline이 실제 운영에서 참조하는 Model과 Metadata**를 사용합니다.

앞에서 적어 둔 값을 다시 확인합니다.

```text
<BASELINE_ONNX>
<BASELINE_META>
```

예:

```text
별도 Model 채택 없음
→ models/day07_tiny_cnn.onnx
→ models/day07_tiny_cnn.meta.json

11일차에서 64×64 후보를 공식 운영 Baseline으로 채택
→ models/day08_tiny_cnn_64.onnx
→ models/day08_tiny_cnn_64.meta.json
```

예시는 가능한 경로를 보여 주는 것일 뿐입니다. **실제 기준은 현재 운영 Config입니다.**

먼저 PC에서 실제 존재 여부를 확인합니다.

```bash
ls -lh \
  <BASELINE_ONNX> \
  <BASELINE_META>
```

### PC에 Baseline Model 파일이 없다면?

ONNX는 Source Git에서 제외되는 Binary 배포 자산이므로 PC와 Raspberry Pi의 Source Commit이 같아도 Model 파일 자체가 PC에 없을 수 있습니다.

이 경우 **비슷한 이름의 다른 Model로 대신하지 않습니다.**

```text
PC에 Baseline ONNX 없음
        ↓
Raspberry Pi가 실제 운영 중인
<BASELINE_ONNX> / <BASELINE_META> 확인
        ↓
허용된 내부 전송 방식으로
정확한 두 파일을 PC에 가져오거나
Raspberry Pi → Jetson으로 직접 전달
        ↓
SHA-256으로 동일성 확인
```

어느 장치에서 파일을 전달하든 **11일차 Tested Baseline과 같은 Binary인지 Hash로 증명**하는 것이 핵심입니다.

Jetson 프로젝트의 `models/` 폴더가 없다면 먼저 만듭니다.

```bash
ssh <JETSON_USER>@<JETSON_IP> \
  "mkdir -p ~/ai_vision/subject13_edge_ai/models"
```

PC에서 전송합니다.

```bash
scp \
  <BASELINE_ONNX> \
  <BASELINE_META> \
  <JETSON_USER>@<JETSON_IP>:~/ai_vision/subject13_edge_ai/models/
```

고정 Test 이미지도 다시 사용합니다.

```text
data/deploy_samples/normal.jpg
data/deploy_samples/warning_red.jpg
```

먼저 PC에서 실제 파일명을 확인합니다.

```bash
ls -lh data/deploy_samples/
```

폴더를 준비합니다.

```bash
ssh <JETSON_USER>@<JETSON_IP> \
  "mkdir -p ~/ai_vision/subject13_edge_ai/data/deploy_samples"
```

전송:

```bash
scp \
  data/deploy_samples/normal.jpg \
  data/deploy_samples/warning_red.jpg \
  <JETSON_USER>@<JETSON_IP>:~/ai_vision/subject13_edge_ai/data/deploy_samples/
```

파일명이 실제와 다르면 교재의 이름을 그대로 입력하지 않습니다. `ls` 또는 `find`로 실제 파일명을 확인한 뒤 사용합니다.

### Jetson Config도 같은 Baseline을 참조하는지 확인하기

`predict_onnx.py`는 Model 경로를 명령행에서 직접 받는 프로그램이 아니라 `configs/settings.yaml`의 `onnx_model_path`, `onnx_meta_path`를 읽어 사용합니다. 따라서 파일만 복사하고 Config가 다른 Model을 가리키면 **전송한 Model과 실제 실행 Model이 달라질 수 있습니다.** 7일차의 `predict_onnx.py`도 이 두 Config Key를 읽어 `ONNXClassifier`를 생성합니다.

Jetson에서:

```bash
python - <<'PY'
from pathlib import Path

from src.config_loader import load_config

config = load_config("configs/settings.yaml")

for key in (
    "onnx_model_path",
    "onnx_meta_path",
):
    path = Path(str(config[key]))
    print(f"{key}: {path}")
    print(f"exists: {path.exists()}")
PY
```

두 경로가 앞에서 정한 `<BASELINE_ONNX>`, `<BASELINE_META>`와 같은 배포 자산을 가리키고 실제 파일도 존재해야 합니다.

다르다면 먼저 다음을 구분합니다.

```text
공통 settings.yaml이 아직 예전 Model을 가리키는가?
        ↓
Jetson device.local.yaml이 Model 경로를 덮어쓰는가?
        ↓
실제 전송된 파일명이 다른가?
```

Raspberry Pi의 `device.local.yaml`을 Jetson으로 복사하여 문제를 해결하지 않습니다. Model Handoff는 공통 운영 Baseline과 Jetson의 실제 배포 경로를 명시적으로 맞춥니다.

---

# 24. 전송한 모델이 정말 같은 파일인지 Hash로 증명하기

파일명이 같다는 것만으로 내용이 같다고 단정하지 않습니다.

```text
PC의 ONNX
        ↓
SHA-256
        ↓
Jetson의 ONNX
        ↓
SHA-256
        ↓
같은가?
```

PC:

```bash
sha256sum \
  <BASELINE_ONNX> \
  <BASELINE_META>
```

Jetson:

```bash
cd ~/ai_vision/subject13_edge_ai

sha256sum \
  <BASELINE_ONNX> \
  <BASELINE_META>
```

두 장치에서 **각 파일의 Hash가 각각 같아야** 합니다.

Report에 기록합니다.

```md
## Baseline Model Evidence

PC ONNX SHA-256:

Jetson ONNX SHA-256:

PC Metadata SHA-256:

Jetson Metadata SHA-256:

Result:
- SAME / DIFFERENT
```

### Hash가 다르면 어떻게 할까요?

```text
오류
→ PC와 Jetson Hash가 다름

원인 확인
→ 실제 파일 경로가 같은가?
→ 예전 모델이 Jetson에 남아 있는가?
→ 전송이 정상 완료되었는가?

수정
→ 실제 기준 파일을 다시 전송

재확인
→ sha256sum 다시 실행
```

Hash가 다르면 Benchmark를 계속 진행하지 않습니다. **같은 모델을 비교한다는 전제부터 다시 맞춥니다.**

---

# 25. Jetson에서 ONNX Runtime 경로가 준비된 경우 같은 Prediction 확인하기

## 이 실습은 언제 진행할까요?

사전 검증된 Jetson 환경에 `onnxruntime`이 준비되어 있을 때만 진행합니다.

확인:

```bash
python -c \
"import onnxruntime as ort; print(ort.__version__); print(ort.get_available_providers())"
```

정상적으로 Import된다면 7일차의 `scripts/predict_onnx.py`를 **이전 일차의 파일 그대로 다시 사용**합니다.

```bash
python -m scripts.predict_onnx \
  --image data/deploy_samples/normal.jpg
```

```bash
python -m scripts.predict_onnx \
  --image data/deploy_samples/warning_red.jpg
```

### 예상 결과 형태

```text
=== ONNX Prediction ===
Image      : ...
Class      : normal 또는 warning_red
Confidence : <실제 값>
```

정확한 Confidence는 Model·Image·Runtime에 따라 달라질 수 있습니다.

이 단계에서 확인하는 핵심은 다음입니다.

```text
같은 ONNX
+
같은 Metadata
+
같은 전처리 코드
        ↓
Jetson에서도
Class / Confidence를 얻을 수 있는가?
```

### ONNX Runtime이 없다면?

임의로 `pip install onnxruntime`부터 실행하지 않습니다.

Jetson의 Python·JetPack·Architecture 조합에 따라 설치 가능한 Package와 방식이 달라질 수 있기 때문입니다.

```text
Import 실패
        ↓
수업 장비가 ONNX Runtime 경로를 지원하도록
사전 준비되었는지 확인
        ↓
지원됨
→ 준비된 설치 절차 사용

지원하지 않음
→ 이 단계는 건너뛰고
   TensorRT 변환·Benchmark 또는 제공 자료로 진행
```

12일차의 성공 여부가 `onnxruntime` Package를 현장에서 새로 설치하는 데 달려 있지 않도록 합니다.

---

# 26. TensorRT Engine은 대상 Jetson에서 생성하기

## 왜 ONNX를 바로 GPU에서 실행한다고 하지 않을까요?

오늘의 흐름은 다음처럼 구분합니다.

```text
ONNX
→ 모델 교환 / 배포 형식

TensorRT
→ NVIDIA GPU 추론 최적화 Runtime

TensorRT Engine
→ 특정 TensorRT 환경에서 실행하기 위해
   Build된 직렬화 결과
```

따라서 다음 문장은 틀립니다.

```text
ONNX = TensorRT
```

TensorRT를 사용할 수 있는지 먼저 확인합니다.

```bash
command -v trtexec
```

아무것도 나오지 않으면 일반적인 Jetson 경로도 확인합니다.

```bash
ls -l /usr/src/tensorrt/bin/trtexec
```

둘 중 하나에서 실제 실행파일이 확인되면 그 경로를 사용합니다.

Engine 폴더:

```bash
mkdir -p models/tensorrt
```

예를 들어 `trtexec`가 PATH에 있다면:

```bash
trtexec \
  --onnx=<BASELINE_ONNX> \
  --saveEngine=models/tensorrt/day12_tiny_cnn_fp16.engine \
  --fp16
```

PATH에 없고 `/usr/src/tensorrt/bin/trtexec`에 있다면:

```bash
/usr/src/tensorrt/bin/trtexec \
  --onnx=<BASELINE_ONNX> \
  --saveEngine=models/tensorrt/day12_tiny_cnn_fp16.engine \
  --fp16
```

### 예상 결과에서 무엇을 볼까요?

TensorRT 버전에 따라 출력 문구는 달라질 수 있습니다.

다음 네 가지를 확인합니다.

```text
1. ONNX를 읽었는가?
2. Engine Build가 성공했는가?
3. FP16 경로가 사용 가능한가?
4. Engine 파일이 실제로 만들어졌는가?
```

파일 확인:

```bash
ls -lh models/tensorrt/
```

정상이라면 다음 파일이 보여야 합니다.

```text
day12_tiny_cnn_fp16.engine
```

### `--fp16`에서 오류가 난다면?

```text
오류
        ↓
trtexec 출력에서 FP16 / Builder / Model 관련 메시지 확인
        ↓
현재 Jetson과 TensorRT가 FP16을 지원하는지 확인
        ↓
수업 장비의 검증된 설정 확인
        ↓
필요하면 검증된 FP32 Build 명령으로 비교
```

임의로 옵션을 여러 개 동시에 바꾸지 않습니다.

---

# 27. TensorRT Engine을 다른 PC에서 만들어 가져오지 않는 이유

다음 흐름을 기본 경로로 사용하지 않습니다.

```text
다른 PC
→ TensorRT Engine 생성
→ Jetson으로 복사
```

TensorRT Engine은 다음 요소의 영향을 받을 수 있습니다.

```text
GPU Architecture
TensorRT Version
CUDA 환경
Builder 설정
Precision
```

따라서 교과 13에서는 다음 원칙을 사용합니다.

```text
공통 배포 자산
→ ONNX

장치 종속 Build 자산
→ TensorRT Engine

ONNX
→ 대상 Jetson으로 이동
→ 대상 환경에서 Engine 생성
```

이 구분은 12일차의 핵심 Migration 원칙입니다.

---

# 28. TensorRT Build 결과를 코드와 파일에서 다시 찾기

`trtexec`는 Python 파일이 아니라 **TensorRT가 제공하는 실행도구**입니다.

다음 연결을 직접 확인합니다.

```text
입력 파일
→ <BASELINE_ONNX>

Build 명령
→ trtexec --onnx=...

Precision 옵션
→ --fp16

출력 파일
→ --saveEngine=...
→ models/tensorrt/day12_tiny_cnn_fp16.engine
```

다음 질문에 답합니다.

```text
1. ONNX 파일 경로는 어느 옵션에서 지정하는가?
2. Engine 출력 경로는 어느 옵션에서 지정하는가?
3. FP16을 요청하는 옵션은 무엇인가?
4. Engine이 실제 생성되었는지는 어떤 명령으로 증명하는가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
1. --onnx=
2. --saveEngine=
3. --fp16
4. ls -lh models/tensorrt/
```

</details>

---

# 29. Guided Lab — TensorRT Engine Model-only Benchmark 실행하기

Engine이 정상 생성되었다면 TensorRT 자체의 Model-only 성능을 확인합니다.

```bash
trtexec \
  --loadEngine=models/tensorrt/day12_tiny_cnn_fp16.engine \
  --warmUp=200 \
  --duration=10
```

`trtexec`가 PATH에 없다면 앞에서 확인한 실제 경로를 사용합니다.

출력 형식은 TensorRT 버전에 따라 다를 수 있습니다. 다음 종류의 값을 찾아 Report에 적습니다.

```text
Latency 관련 값
Throughput 관련 값
실행 Precision
Input Shape
```

### 중요한 주의 — `trtexec` Benchmark는 실제 Camera Pipeline 전체 속도가 아닙니다

현재 Benchmark에는 다음 단계가 포함되지 않을 수 있습니다.

```text
Camera Capture
Preprocess
Class 이름으로 Postprocess
Operational Decision
Stabilization
GPIO
Event Log
FastAPI
Network
```

따라서 다음처럼 구분합니다.

```text
trtexec
→ TensorRT Engine Model-only Benchmark

교과 8~10에서 만든 운영 Pipeline
→ Camera부터 Log/API까지 포함할 수 있는 End-to-End
```

---

# 30. `trtexec`를 실제 이미지 Prediction 프로그램으로 착각하지 않기

`trtexec`로 Engine을 Build하고 Benchmark했다고 해서 다음이 자동으로 증명되는 것은 아닙니다.

```text
normal.jpg
→ normal

warning_red.jpg
→ warning_red
```

왜냐하면 실제 Class Prediction에는 다음이 필요하기 때문입니다.

```text
Image 읽기
        ↓
Resize / Normalize
        ↓
NCHW Tensor
        ↓
TensorRT Engine
        ↓
Output Tensor
        ↓
Softmax
        ↓
Class Index
        ↓
Metadata Class 이름
        ↓
Confidence
```

오늘은 `TensorRTClassifier` 같은 새로운 추론 Adapter를 갑자기 처음부터 구현하지 않습니다.

따라서 검증 범위를 정확히 나눕니다.

```text
ONNX Runtime이 준비된 Jetson
→ Fixed Image Prediction까지 확인 가능

TensorRT trtexec
→ Engine Build와 Model-only Benchmark 확인

실제 TensorRT Image Prediction
→ 별도 검증된 Runner가 제공된 경우에만 실행
→ 그렇지 않으면 교과 14의 Runtime 구현에서 연결
```

이 구분을 Report에 적으면 **무엇을 실제로 검증했고 무엇은 아직 검증하지 않았는지** 명확해집니다.

---

# 31. Raspberry Pi와 Jetson 성능 숫자를 비교할 때 먼저 측정 범위를 확인하기

Raspberry Pi의 8일차 `benchmark_model_only.py`와 Jetson의 `trtexec`는 서로 다른 Runtime과 측정도구를 사용할 수 있습니다.

따라서 숫자를 표에 나란히 적더라도 기본적으로 다음 중 하나로 표시합니다.

```text
직접 비교 가능
또는
참고 비교
```

다음 조건이 모두 같지 않다면 **참고 비교**로 적는 편이 안전합니다.

```text
Model 구조
Input Shape
Batch
Precision
Warm-up
측정 구간
Runtime
통계 계산 방식
```

예를 들어 Raspberry Pi가 ONNX Runtime FP32이고 Jetson이 TensorRT FP16이라면 장치뿐 아니라 Runtime과 Precision도 함께 바뀌었습니다.

```text
Changed Variable
→ Device 하나만 아님
→ Runtime도 다름
→ Precision도 다를 수 있음
```

따라서 “Jetson이 몇 배 빠르다”라고 단정하기 전에 측정 조건 차이를 먼저 적습니다.

`reports/day12_rpi_jetson.md`에 다음 표를 추가합니다.

```md
## Runtime / Deployment Comparison

| Device | Runtime | Processor | Precision | Input | Mean / Latency | P95 | Throughput / FPS | 비교 수준 |
|---|---|---|---|---|---:|---:|---:|---|
| Raspberry Pi | ONNX Runtime | CPU | FP32 |  |  |  |  | 기준 |
| Jetson | ONNX Runtime | CPU 또는 Provider |  |  |  |  |  | 조건이 맞을 때 직접 비교 |
| Jetson | TensorRT | GPU | FP16 |  |  |  |  | 보통 참고 비교 |

### Interpretation

직접 비교 가능한 행:

참고 비교인 행:

측정 조건이 다른 이유:

내가 실제로 검증한 범위:
```

측정하지 않은 값은 `N/A`로 기록합니다. 숫자를 추측해서 채우지 않습니다.

---

# PART E. 장비 유무와 관계없이 Migration Contract를 완성하기

# 32. Jetson 장비가 없는 경우 — 제공 자료로 같은 실습 진행하기

Jetson 장비가 준비되지 않은 경우에도 단순 이론 강의로 끝내지 않습니다.

제공된 다음 자료를 사용합니다.

```text
Jetson 환경 확인 화면

JetPack / CUDA / TensorRT 확인 화면

ONNX → TensorRT 변환 시연

TensorRT Benchmark 결과

교과 14 Architecture
```

그리고 실제 Raspberry Pi 결과와 비교합니다.

---

# 33. 제공된 Jetson Benchmark CSV 템플릿 만들기

## 이 파일은 왜 필요할까요?

Jetson 장비가 없는 경우 제공 자료의 숫자를 직접 측정값과 섞지 않도록 **제공 결과만 따로 기록**합니다. `source=provided`를 남기면 나중에 결과의 출처를 구분할 수 있습니다.

파일:

```text
reports/day12_jetson_provided.csv
```

형식:

```csv
device,runtime,processor,input_size,mean_latency_ms,p95_ms,fps,source
Jetson,TensorRT,GPU,<PROVIDED_INPUT_SIZE>,,,,provided
```

제공 자료에 있는 실제 값을 입력합니다.

`<PROVIDED_INPUT_SIZE>`도 자신의 Baseline에 맞추어 임의로 바꾸지 않습니다. 제공 자료가 `96`이라면 `96`, `64`라면 `64`처럼 **제공 자료의 실제 조건**을 적습니다.

값이 제공되지 않은 항목은 비워 둡니다.

임의의 숫자를 만들지 않습니다.

---

# 34. Raspberry Pi 결과도 같은 형식으로 정리하기

## 이 파일은 왜 필요할까요?

8일차 Raspberry Pi 측정값을 Jetson 제공 자료와 같은 열 구조로 정리하면 값 자체뿐 아니라 `device`, `runtime`, `processor`, `input_size`, `source`를 함께 비교할 수 있습니다.

파일:

```text
reports/day12_rpi_result.csv
```

예:

```csv
device,runtime,processor,input_size,mean_latency_ms,p95_ms,fps,source
Raspberry Pi,ONNX Runtime,CPU,<RPI_RESULT_INPUT_SIZE>,,,,measured
```

8일차에 측정한 실제 값을 입력합니다.

다만 11일차에서 최종 운영 Baseline을 다른 Input Size로 채택했다면 **그 Baseline과 대응하는 Raspberry Pi 측정 결과**를 사용합니다.

```text
Baseline = 96×96
→ 96×96 측정 결과 사용

Baseline = 64×64
→ 64×64 측정 결과 사용

대응 측정값이 없음
→ 다른 크기의 숫자를 대신 넣지 않음
→ N/A 또는 REFERENCE로 기록
```

---

# 35. 장비가 없어도 비교표를 직접 완성하기

`reports/day12_rpi_jetson.md`:

```md
## Provided Comparison

| Device | Runtime | Processor | Input | Mean | P95 | FPS | 결과 출처 |
|---|---|---|---:|---:|---:|---:|---|
| Raspberry Pi | ONNX Runtime | CPU | <RPI_INPUT_SIZE> |  |  |  | 직접 측정 |
| Jetson | TensorRT | GPU | <JETSON_INPUT_SIZE> |  |  |  | 제공 자료 |

## 확인

같은 점:

다른 점:

직접 측정값인가?

제공된 측정값인가?

측정 조건이 완전히 같은가?

단순 FPS 비교에 주의할 점:
```

`<RPI_INPUT_SIZE>`와 `<JETSON_INPUT_SIZE>`는 실제 기록된 값을 각각 적습니다. 두 값이 다르면 같은 입력 조건이 아니므로 `Same Model` 직접 비교라고 표현하지 않고 **REFERENCE 비교**로 표시합니다.

---

# 36. Raspberry Pi Camera 계층과 Jetson Camera 계층 비교하기

교과 13 Raspberry Pi:

```text
USB Camera
→ Linux Video Device
→ /dev/video*
→ OpenCV VideoCapture
```

교과 14 Jetson에서는 산업용 Camera를 사용할 수 있습니다.

예상 구조:

```text
Industrial Camera
→ Vendor Driver / SDK
→ Camera API
→ Frame
→ OpenCV / AI Preprocess
```

따라서 기존 `camera_input.py`를 그대로 사용할 수도 있고, 산업용 Camera 요구에 맞는 새로운 Adapter가 필요할 수도 있습니다.

---

# 37. Camera가 달라져도 지켜야 할 출력 Contract 정하기

3일차에서 만든 `src/camera_input.py`는 **이전 일차의 파일을 다시 사용**합니다.

현재 `CameraInput.read()`가 반환하는 값은 OpenCV가 사용하는 BGR Frame입니다.

```text
USB Camera
        ↓
CameraInput
        ↓
OpenCV BGR Frame
        ↓
ai_preprocess / 이후 처리
```

교과 14의 산업용 Camera는 Vendor Driver나 SDK를 사용할 수 있습니다.

```text
Industrial Camera
        ↓
Vendor Driver / SDK
        ↓
IndustrialCameraInput
        ↓
BGR Frame
        ↓
ai_preprocess / 이후 처리
```

중요한 것은 내부 구현을 억지로 같게 만드는 것이 아니라 **다음 Module이 기대하는 출력 형태를 맞추는 것**입니다.

예를 들어 산업용 SDK가 RGB, Bayer, Mono 형식으로 Frame을 준다면 Adapter 안에서 필요한 변환을 하고 공통 Pipeline에는 BGR Frame을 넘길 수 있습니다.

---

# 38. Camera Adapter Contract 문서 만들기

## 이 파일은 왜 필요할까요?

오늘 실제 산업용 Camera SDK 코드를 새로 개발하지 않습니다. 하지만 다음 날 또는 교과 14에서 Camera가 바뀔 때 **어떤 Interface를 유지해야 기존 Pipeline을 덜 수정할 수 있는지** 먼저 문서로 남깁니다.

3일차 `CameraInput`의 실제 사용 형태를 기준으로 작성합니다.

파일:

```text
docs/day12_camera_adapter.md
```

내용:

````md
# Camera Adapter Contract

## Raspberry Pi Current Adapter

Implementation:

- `src/camera_input.py`
- `CameraInput`

Create:

```python
camera = CameraInput(
    index=...,
    width=...,
    height=...,
)
```

Read:

```python
frame = camera.read()
```

Release:

```python
camera.release()
```

Output:

- OpenCV BGR Frame
- `numpy.ndarray`
- 일반적인 Color Frame이라면 shape는 `(height, width, 3)`

## Jetson / Course 14 Target Adapter

Input:

- Industrial Camera
- Vendor Driver / SDK

Target Usage:

```python
camera = IndustrialCameraInput(...)
frame = camera.read()
camera.release()
```

Output Goal:

- 이후 Pipeline이 사용할 수 있는 BGR Frame

## Common Contract

1. 생성 시 Camera 연결에 필요한 설정을 받는다.
2. `read()`는 현재 Frame을 반환한다.
3. 실패하면 조용히 잘못된 값을 반환하지 말고 원인을 알 수 있는 오류를 만든다.
4. `release()`로 Camera Resource를 정리한다.
5. 이후 Preprocess가 기대하는 Frame 형식을 문서로 유지한다.

## Reuse After Camera Layer

- `ai_preprocess.py`
- `decision.py`
- `stabilizer.py`
- `event_logger.py`
- `performance.py`
````

### 왜 `open()`을 공통 필수 함수로 넣지 않았을까요?

현재 3일차 `CameraInput`은 별도의 `open()` 호출이 아니라 **객체를 생성할 때 Camera를 엽니다.**

새 Adapter에 갑자기 다른 사용법을 강제하면 호출부까지 불필요하게 달라질 수 있습니다.

오늘은 이미 사용한 Interface를 기준으로 Contract를 정합니다.

---

# 39. Inference 계층도 같은 방식으로 분리해서 보기

7일차에서 만든 `src/inference_onnx.py`의 `ONNXClassifier`는 **이전 일차의 파일을 다시 사용**합니다.

현재 구조:

```text
NCHW float32 Tensor
        ↓
ONNXClassifier.predict()
        ↓
ONNX Runtime
        ↓
Prediction Dictionary
```

Jetson에서 TensorRT용 Adapter를 나중에 만든다면 내부 Runtime은 달라질 수 있습니다.

```text
NCHW float32 Tensor
        ↓
TensorRTClassifier.predict()
        ↓
TensorRT Engine
        ↓
Prediction Dictionary
```

중요한 것은 다음 Module이 필요로 하는 최소 출력입니다.

```python
{
    "class_name": "warning_red",
    "confidence": 0.91,
}
```

기존 `ONNXClassifier.predict()`는 이 두 값 외에도 `class_index`, `probabilities`, `logits` 같은 값을 반환할 수 있습니다. 하지만 `decision.py`가 실제 운영 판정에 꼭 필요한 값은 주로 다음 두 가지입니다.

```text
class_name
confidence
```

---

# 40. Inference Contract 문서 만들기

## 이 파일은 왜 필요할까요?

Runtime이 바뀔 때 `decision.py`, `stabilizer.py`, `event_logger.py`까지 모두 다시 고치지 않으려면 **Inference 결과의 약속**을 먼저 정해야 합니다.

파일:

```text
docs/day12_inference_contract.md
```

내용:

````md
# Inference Contract

## Raspberry Pi Current Runtime

Runtime:

- ONNX Runtime CPU

Input:

- NCHW
- float32
- Model Metadata의 `image_size`에 맞춘 Tensor

Call:

```python
result = classifier.predict(input_tensor)
```

Required Output:

```python
{
    "class_name": "warning_red",
    "confidence": 0.91,
}
```

Optional Existing Output:

- `class_index`
- `probabilities`
- `logits`

## Jetson Target Runtime

Runtime:

- TensorRT 또는 수업에서 검증된 Jetson GPU Runtime

Input Goal:

- Model이 요구하는 Shape / dtype
- Preprocess 규칙 일치

Required Output Goal:

```python
{
    "class_name": "warning_red",
    "confidence": 0.91,
}
```

## Reused Modules After Inference

- `decision.py`
- `stabilizer.py`
- `event_logger.py`
- `runtime_state.py`

## Important

TensorRT Engine Build가 성공했다는 사실만으로
실제 Image → Class / Confidence Contract까지
검증되었다고 기록하지 않는다.

별도 TensorRT Inference Runner가 실제로
Preprocess → Engine → Postprocess를 수행했을 때만
Prediction Contract PASS로 기록한다.
````

### 코드 리뷰 — Contract가 실제 코드와 맞는지 확인하기

문서만 작성하고 끝내지 않습니다.

다음 파일을 직접 엽니다.

```text
src/camera_input.py
src/inference_onnx.py
src/decision.py
```

확인할 연결:

```text
CameraInput.read()
→ 어떤 Frame을 반환하는가?

ONNXClassifier.predict()
→ 어떤 Dictionary Key를 반환하는가?

ai_to_operational_state()
→ Prediction 중 어떤 값을 입력으로 받는가?
```

문서와 실제 코드가 다르면 문서 또는 구현 중 어느 쪽이 기준인지 확인하고 맞춥니다.

---

# 41. 재사용한 Unit Test 결과를 코드 위치와 다시 연결하기

20번 실습에서 이미 Unit Test를 실행했으므로 같은 명령을 반복하지 않습니다.

이번에는 결과를 보고 **왜 이 Test가 Jetson에서도 그대로 재사용될 수 있었는지 코드 수준에서 확인**합니다.

```text
test_decision.py
        ↓
ai_to_operational_state()
        ↓
Camera / GPU 직접 사용 없음

test_stabilizer.py
        ↓
TemporalStabilizer.update()
        ↓
상태 문자열 + 시간값 사용

test_runtime_state.py
        ↓
RuntimeStateStore
        ↓
임시 JSON 파일 사용
```

다음 질문에 답합니다.

```text
1. 세 Test 중 실제 Camera를 여는 Test가 있는가?
2. 세 Test 중 TensorRT Engine을 요구하는 Test가 있는가?
3. 세 Test가 PASS했다면 어떤 계층의 이식성을 확인한 것인가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
1. 없다.
2. 없다.
3. Decision / Stabilization / Runtime State처럼
   Hardware 의존성이 낮은 공통 Logic의 이식성을 확인한 것이다.
```

</details>

이것이 **“장치가 달라져도 Test를 재사용할 수 있다”**는 의미입니다.

---

# 42. 8일차 성능 측정 구조도 재사용하기

Raspberry Pi에서 사용한 측정 구조:

```text
Warm-up
→ 반복
→ Latency 저장
→ Mean
→ P50
→ P95
```

이 측정 원칙은 Jetson에서도 같습니다.

달라지는 것은:

```text
Runtime

ONNX Runtime CPU
        ↓
TensorRT GPU
```

입니다.

즉 `performance.py`의 통계 계산 방식은 그대로 재사용할 수 있습니다.

---

# 43. 9일차 안정화 로직도 그대로 가져가기

Raspberry Pi:

```text
AI Raw Result
→ Voting
→ Debounce
→ Hold
```

Jetson에서도 AI 결과가 시간에 따라 흔들릴 수 있습니다.

따라서:

```text
stabilizer.py
```

는 GPU 여부와 관계없이 재사용할 수 있습니다.

GPU가 빨라졌다고 해서 시간적 안정화 문제가 자동으로 사라지는 것은 아닙니다.

---

# 44. 10일차 systemd도 같은 운영 개념으로 이어집니다

Jetson도 Linux 기반 장치이므로 다음 개념은 그대로 이어질 수 있습니다.

```text
Boot
→ systemd
→ AI Program
→ 자동실행
→ Restart
→ journalctl
```

단, 실제 Service 파일의:

```text
User
WorkingDirectory
Python Path
Environment
ExecStart
```

는 Jetson 환경에 맞게 다시 확인해야 합니다.

Raspberry Pi Service 파일을 아무 수정 없이 복사하지 않습니다.

---

# 45. FastAPI도 장치와 독립적인 계층입니다

Raspberry Pi:

```text
Edge Runtime
→ Runtime State
→ FastAPI
→ PC
```

Jetson에서도 같은 구조를 사용할 수 있습니다.

```text
Jetson AI Runtime
→ Runtime State
→ FastAPI
→ PC / Tablet / Dashboard
```

AI Runtime이:

```text
ONNX Runtime CPU
```

에서:

```text
TensorRT GPU
```

로 바뀌어도 HTTP API의 역할은 크게 달라지지 않습니다.

---

# PART F. Raspberry Pi → Jetson Migration Map과 비교 결과 완성하기

# 46. 교과 14로 연결되는 제조 Vision AI 구조 보기

교과 14에서는 입력과 AI Task가 더 실제 제조환경에 가까워집니다.

```text
Industrial Camera
        ↓
Camera SDK
        ↓
Jetson
        ↓
Preprocess
        ↓
Object Detection Model
        ↓
ONNX / TensorRT
        ↓
Inspection Result
        ↓
Log / Statistics
        ↓
Optional FastAPI
        ↓
PC / Tablet
```

오늘 Tiny CNN Classifier를 그대로 제조검사 모델로 사용한다는 뜻은 아닙니다.

다음 **구조적 경험**을 재사용합니다.

```text
Camera 입력
Model 배포
Runtime
Decision
Performance
Logging
Service
Failure Test
External Status
```

---

# 47. 교과 13과 교과 14의 Module Mapping 작성하기

Report에 추가합니다.

```md
## Course 13 → Course 14 Mapping

| Course 13 | Course 14에서 | 유지 / 변경 | 이유 |
|---|---|---|---|
| Raspberry Pi | Jetson | 변경 |  |
| USB Camera | Industrial Camera | 변경 |  |
| OpenCV VideoCapture | Camera SDK | 변경 가능 |  |
| Tiny CNN | Detection Model | 변경 |  |
| ONNX Runtime CPU | TensorRT GPU | 변경 |  |
| decision.py | Inspection Decision | 일부 재사용 |  |
| stabilizer.py | Temporal Stability | 재사용 가능 |  |
| event_logger.py | Inspection Log | 재사용 가능 |  |
| performance.py | FPS / Latency | 재사용 |  |
| systemd | Auto Start | 재사용 가능 |  |
| FastAPI | Result API | 재사용 가능 |  |
```

---

# 48. Raspberry Pi → Jetson Migration Map 직접 작성하기

## 이 파일은 왜 필요할까요?

지금까지의 비교표를 마지막에 한 장의 Migration 문서로 묶어 **그대로 가져갈 것, 수정할 것, 새로 확인할 것**을 분리합니다. 13일차 Final Release와 이후 교과 14에서 다시 찾을 수 있는 연결 문서입니다.

파일:

```text
docs/day12_migration_map.md
```

내용:

```md
# Raspberry Pi → Jetson Migration Map

## 그대로 가져갈 것

1.

2.

3.

4.

5.

## 수정할 것

1.

2.

3.

4.

## 새로 확인할 것

1. JetPack

2. CUDA

3. TensorRT

4. Camera Driver / SDK

5. GPU / Power / Temperature

## Final Flow

Raspberry Pi:

USB Camera
→ ONNX Runtime CPU
→ Decision
→ GPIO / Log

Jetson:

Industrial Camera
→ Camera SDK
→ ONNX → TensorRT GPU
→ Decision
→ Inspection Result / Log
```

앞에서 확인한 프로젝트를 보면서 직접 채웁니다.

---

# 49. CPU vs GPU 비교표 완성하기

Report:

```md
## CPU vs GPU Edge AI

| 항목 | Raspberry Pi | Jetson |
|---|---|---|
| 주요 AI 추론 장치 | CPU | GPU 사용 가능 |
| 기본 실습 Runtime | ONNX Runtime | TensorRT 등 |
| 병렬 AI 연산 | 제한적 | 강점 |
| 전력·열 관리 | 필요 | 더욱 중요 |
| Camera | USB Camera | 산업용 Camera 가능 |
| Linux / SSH | 사용 | 사용 |
| systemd | 사용 | 사용 가능 |
| FastAPI | 사용 | 사용 가능 |
| Latency / FPS 측정 | 사용 | 그대로 필요 |

## 내가 이해한 핵심

```

---

# 50. ONNX vs TensorRT 비교표 완성하기

```md
## ONNX vs TensorRT

| 항목 | ONNX | TensorRT |
|---|---|---|
| 역할 |  |  |
| 특정 GPU 의존성 |  |  |
| 교과 13에서 |  |  |
| Jetson에서 |  |  |
| 장점 |  |  |
| 주의점 |  |  |

## 설명

ONNX:

TensorRT:

두 기술의 연결:
```

---

# PART G. Mini Challenge — Jetson Migration을 스스로 검증하기

앞의 Guided Lab에서는 안내된 순서대로 Raspberry Pi와 Jetson의 차이를 확인했습니다.

이제부터는 다음 흐름을 스스로 반복합니다.

```text
실행 결과 보기
        ↓
코드 / 명령 위치 찾기
        ↓
실행 전에 결과 예측
        ↓
설정값만 변경
        ↓
기능 일부 수정
        ↓
일부러 오류 발생
        ↓
원인 확인 → 수정 → 재실행
        ↓
Hash / Report / Evidence로 정상 동작 증명
        ↓
전체 Migration Pipeline 설명
```

각 Challenge는 먼저 문제만 읽고 직접 시도합니다.  
그 뒤에 **예시 정답 확인해보기**를 열어 자신의 결과와 비교합니다.

Jetson 장비가 없는 경우에는 제공 자료와 PC/Raspberry Pi에서 실행 가능한 부분을 사용합니다. 없는 Jetson 출력값을 임의로 만들지 않습니다.

---

## Challenge 1 — 실행 결과를 보고 코드 위치 찾기

Jetson에서 다음과 비슷한 결과가 출력되었다고 가정합니다.

```text
=== Jetson Environment Check ===
Hostname      : jetson-...
Architecture  : aarch64
Python        : 3.x.x

[Jetson Linux]
...

nvcc          : /usr/local/cuda/bin/nvcc
trtexec       : /usr/src/tensorrt/bin/trtexec
tegrastats    : /usr/bin/tegrastats
```

`scripts/jetson_env_check.py`를 열어 다음 표를 직접 채웁니다.

| 화면 결과 | 담당 함수·코드 | 외부에서 확인하는 대상 |
|---|---|---|
| Hostname |  |  |
| Architecture |  |  |
| Python |  |  |
| Jetson Linux |  |  |
| `nvcc` 경로 |  |  |
| `trtexec` 경로 |  |  |
| `tegrastats` 경로 |  |  |

추가로 다음 질문에 답합니다.

```text
1. trtexec가 PATH에 없는데도 경로를 찾을 수 있는 코드는 어디인가?
2. JetPack Package 정보는 어떤 함수가 어떤 명령을 실행해 얻는가?
3. 이 Script의 결과는 AI Prediction 결과인가, 환경 진단 결과인가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

| 화면 결과 | 담당 함수·코드 | 외부에서 확인하는 대상 |
|---|---|---|
| Hostname | `platform.node()` | 현재 OS Hostname |
| Architecture | `platform.machine()` | CPU Architecture |
| Python | `platform.python_version()` | 현재 Python |
| Jetson Linux | `Path("/etc/nv_tegra_release")` | Jetson Linux Release 파일 |
| `nvcc` 경로 | `find_executable()` | PATH 또는 `/usr/local/cuda/bin/nvcc` |
| `trtexec` 경로 | `find_executable()` | PATH 또는 `/usr/src/tensorrt/bin/trtexec` |
| `tegrastats` 경로 | `find_executable()` | PATH 또는 `/usr/bin/tegrastats` |

```text
1. find_executable()의 fallback_paths 확인 부분
2. run_command()가 dpkg-query --show nvidia-jetpack 실행
3. 환경 진단 결과
```

왜 이런 결과가 나올까요?

`jetson_env_check.py`는 ONNX나 TensorRT Model을 실행하지 않습니다. OS와 실행파일의 존재 위치를 읽고 출력할 뿐입니다.

</details>

---

## Challenge 2 — 실행 전에 재사용 가능 여부를 예측하기

다음 Module을 Jetson으로 옮긴다고 가정합니다.

```text
src/decision.py
src/stabilizer.py
src/runtime_state.py
src/camera_input.py
src/inference_onnx.py
src/gpio_output.py
src/performance.py
```

아직 Jetson에서 실행하지 말고 먼저 다음 표를 채웁니다.

| Module | 그대로 재사용 예상 | 수정 가능성 높음 | 이유 |
|---|---|---|---|
| decision.py |  |  |  |
| stabilizer.py |  |  |  |
| runtime_state.py |  |  |  |
| camera_input.py |  |  |  |
| inference_onnx.py |  |  |  |
| gpio_output.py |  |  |  |
| performance.py |  |  |  |

그다음 Jetson에서 Hardware 의존성이 낮은 Test를 실행합니다.

```bash
python -m pytest \
  tests/test_decision.py \
  tests/test_stabilizer.py \
  tests/test_runtime_state.py \
  -v
```

실행 전 예상과 실제 결과를 비교합니다.

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

| Module | 예상 | 이유 |
|---|---|---|
| `decision.py` | 재사용 가능성이 높음 | 문자열·Confidence 기반 Logic |
| `stabilizer.py` | 재사용 가능성이 높음 | Voting·Debounce·Hold Logic |
| `runtime_state.py` | 재사용 가능성이 높음 | JSON 파일 기반 상태 공유 |
| `camera_input.py` | 변경 가능성 높음 | Camera Driver / SDK 차이 |
| `inference_onnx.py` | 변경 가능성 높음 | Jetson에서 TensorRT Runtime으로 바뀔 수 있음 |
| `gpio_output.py` | 변경 가능성 있음 | GPIO Library·Pin·출력 요구 차이 |
| `performance.py` | 재사용 가능성이 높음 | Latency 통계 계산 자체는 장치 독립적 |

예상되는 Unit Test 핵심 결과:

```text
Decision Test
→ PASS

Stabilizer Test
→ PASS

Runtime State Test
→ PASS
```

실제 결과가 FAIL이라면 “Jetson이라서 원래 안 된다”고 넘기지 않습니다.

```text
FAIL
→ Import 오류인가?
→ Python / Package 문제인가?
→ Source가 같은 Commit인가?
→ 실제 Logic 오류인가?
```

를 구분합니다.

</details>

---

## Challenge 3 — Python 코드는 그대로 두고 설정값만 바꾸어 결과 비교하기

이번 Challenge의 목적은 **Runtime 코드를 수정하지 않고 운영 Threshold만 바꾸어 결과가 달라지는지** 확인하는 것입니다.

ONNX Runtime이 준비된 Jetson에서는 Jetson에서 수행합니다.  
Jetson에 ONNX Runtime이 준비되지 않았다면 같은 `subject13_edge_ai` Source가 정상 동작하는 PC 또는 Raspberry Pi에서 수행합니다.

먼저 현재 설정을 기록합니다.

```bash
grep -E "^ai_confidence_threshold:" \
  configs/settings.yaml
```

같은 Fixed Image를 사용합니다.

```text
data/deploy_samples/warning_red.jpg
```

실험 A:

```yaml
ai_confidence_threshold: 0.60
```

실행:

```bash
python -m scripts.onnx_decision_test \
  --image data/deploy_samples/warning_red.jpg
```

실험 B:

```yaml
ai_confidence_threshold: 0.90
```

같은 명령을 다시 실행합니다.

### 먼저 예측하기

실행 전 다음 질문에 답합니다.

```text
Prediction Class 자체가 Threshold 때문에 바뀌는가?

Confidence 자체가 Threshold 때문에 바뀌는가?

Operational State는 바뀔 수 있는가?

예를 들어 warning_red Confidence가 0.82라면
Threshold 0.60과 0.90에서 각각 State는?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

같은 Model과 같은 Image를 사용하므로 Threshold 변경은 AI Model의 Class와 Confidence를 직접 바꾸지 않습니다.

```text
Prediction
warning_red 0.82
```

라고 가정하면:

```text
Threshold 0.60
0.82 >= 0.60
→ WARNING

Threshold 0.90
0.82 < 0.90
→ NORMAL
```

왜 그런가요?

```text
ONNXClassifier.predict()
→ Class / Confidence 생성

ai_to_operational_state()
→ Class / Confidence와
   ai_confidence_threshold를 비교
→ 최종 운영 State 생성
```

즉, **AI Prediction과 운영 Decision은 분리**되어 있기 때문입니다.

</details>

### Challenge 3 복원

실험이 끝나면 11일차에서 선택했던 운영 Threshold로 반드시 복원합니다.

현재 Git 기준값과 비교합니다.

```bash
git diff configs/settings.yaml
```

Challenge에서 이 파일의 Threshold만 바꿨고 다른 필요한 변경이 없다면:

```bash
git restore configs/settings.yaml
```

복원값을 다시 확인합니다.

```bash
grep -E "^ai_confidence_threshold:" \
  configs/settings.yaml
```

---

## Challenge 4 — `jetson_env_check.py` 기능 일부 수정하기

이번에는 기존 기능을 조금 확장합니다.

요구사항:

```text
TensorRT Engine 파일이 만들어졌는가?
        ↓
있으면 파일 크기는 몇 Byte인가?
        ↓
환경 진단 결과에 함께 출력
```

대상 파일:

```text
scripts/jetson_env_check.py
```

바로 정답을 보지 말고 다음을 먼저 생각합니다.

```text
1. Engine 경로를 다루려면 이미 Import한 어떤 Class를 재사용할 수 있는가?
2. 파일 존재 여부는 어떤 메서드로 확인할 수 있는가?
3. 파일 크기는 어떤 메서드로 확인할 수 있는가?
4. Engine이 없어도 Script 전체가 실패하지 않게 하려면 어떻게 해야 하는가?
```

직접 코드를 추가한 뒤 실행합니다.

```bash
python -m scripts.jetson_env_check
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

`main()`의 환경 출력 뒤에 다음 코드를 추가할 수 있습니다.

```python
engine_path = Path(
    "models/tensorrt/"
    "day12_tiny_cnn_fp16.engine"
)

print()
print("[TensorRT Engine]")

if engine_path.exists():
    size_bytes = engine_path.stat().st_size

    print(f"Path   : {engine_path}")
    print("Exists : True")
    print(f"Bytes  : {size_bytes}")
else:
    print(f"Path   : {engine_path}")
    print("Exists : False")
```

예상 형태:

```text
[TensorRT Engine]
Path   : models/tensorrt/day12_tiny_cnn_fp16.engine
Exists : True
Bytes  : <실제 크기>
```

Engine이 없다면:

```text
Exists : False
```

가 나와야 하며 Script 자체는 정상 종료됩니다.

왜 이렇게 작성할까요?

`Path.exists()`로 먼저 존재 여부를 확인했기 때문에 파일이 없는데 `stat()`을 바로 호출하여 오류가 나는 것을 피할 수 있습니다.

</details>

이 기능을 최종 Source에 유지할지 여부는 팀의 코드 정책에 따라 결정할 수 있습니다. 유지한다면 Report에 “Challenge Extension”이라고 기록합니다.

---

## Challenge 5 — 일부러 경로 오류를 만들고 원인 찾기

이번에는 Source 파일을 망가뜨리지 않고 **잘못된 경로를 명령에 넣어 오류를 관찰**합니다.

Jetson에서 앞에서 확인한 실제 Baseline Model 경로를 다시 적습니다.

```text
<BASELINE_ONNX>
→ ___________________________________________
```

이번에는 Baseline 파일 자체를 바꾸지 않고 **존재하지 않는 경로**를 일부러 입력합니다.

```bash
sha256sum \
  models/not_existing_day12_model.onnx
```

오류 메시지를 본 뒤 바로 정답을 보지 말고 다음 순서로 해결합니다.

```text
오류 마지막 줄 확인
        ↓
어떤 경로를 찾지 못했는가?
        ↓
현재 위치 pwd
        ↓
models 폴더 ls
        ↓
실제 파일명 확인
        ↓
명령 수정
        ↓
재실행
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

다음과 비슷한 오류를 볼 수 있습니다.

```text
sha256sum: models/not_existing_day12_model.onnx:
No such file or directory
```

확인:

```bash
pwd
ls -lh models/
```

올바른 Baseline 경로가 확인되면:

```bash
sha256sum \
  <BASELINE_ONNX>
```

로 다시 실행합니다.

오류의 원인은 TensorRT나 GPU가 아니라 **단순 Path 오류**입니다.

이런 구분이 중요한 이유는 Jetson에서 문제가 생겼다고 해서 모든 오류를 CUDA·TensorRT 문제로 생각하면 원인을 찾는 시간이 길어지기 때문입니다.

</details>

---

## Challenge 6 — 결과 파일과 Evidence로 정상 동작 증명하기

이번에는 “내가 실행했다”라고 말하는 대신 **다른 사람이 나중에 확인할 수 있는 증거 파일**을 남깁니다.

Jetson 프로젝트에서:

```bash
mkdir -p reports/day12_evidence
```

환경 확인 결과를 저장합니다.

```bash
python -m scripts.jetson_env_check \
  | tee reports/day12_evidence/jetson_env.txt
```

모델 Hash도 저장합니다.

```bash
sha256sum \
  <BASELINE_ONNX> \
  <BASELINE_META> \
  | tee reports/day12_evidence/jetson_model_hash.txt
```

TensorRT Engine이 있다면 파일 정보도 저장합니다.

```bash
ls -lh \
  models/tensorrt/ \
  | tee reports/day12_evidence/tensorrt_files.txt
```

확인:

```bash
find reports/day12_evidence \
  -maxdepth 1 \
  -type f \
  -print
```

다음 질문에 답합니다.

```text
1. 환경 진단 결과는 어느 파일에 남았는가?
2. Model 동일성을 확인할 Hash는 어느 파일에 남았는가?
3. TensorRT Engine 자체를 Git에 올리지 않아도
   Engine 존재 증거를 무엇으로 남길 수 있는가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
1. reports/day12_evidence/jetson_env.txt

2. reports/day12_evidence/jetson_model_hash.txt

3. reports/day12_evidence/tensorrt_files.txt
   + reports/day12_rpi_jetson.md의 Benchmark 기록
```

`tee`는 화면에 출력하면서 같은 내용을 파일에도 저장합니다.

```text
명령 실행
        ↓
터미널에서 즉시 확인
        +
Evidence 파일 저장
```

이 방식은 장비 자체의 Binary Model을 Repository에 넣지 않고도 실행 근거를 남기는 데 도움이 됩니다.

</details>

Jetson 장비가 없는 경우에는 제공 자료의 출처와 값이 들어 있는 `reports/day12_jetson_provided.csv`와 비교 Report가 Evidence 역할을 합니다.

---

## Challenge 7 — 오늘 배운 전체 Pipeline을 자신의 말로 설명하기

다음 빈칸을 채우지 말고, 먼저 문서를 닫고 2~3분 동안 자신의 말로 설명합니다.

질문:

```text
1. 왜 11일차 Tested Baseline을 먼저 확인했는가?

2. 왜 Raspberry Pi의 .venv를 Jetson으로 복사하지 않았는가?

3. Source 중 어떤 부분은 재사용하고 어떤 부분은 바뀌는가?

4. 왜 ONNX와 TensorRT를 같은 것으로 보면 안 되는가?

5. 왜 ONNX는 PC에서 만들어 옮길 수 있지만
   TensorRT Engine은 대상 Jetson에서 만드는가?

6. trtexec Benchmark는 전체 Camera Pipeline FPS인가?

7. Camera가 산업용 Camera로 바뀌어도
   어떤 Contract를 유지하면 이후 Module을 재사용하기 쉬운가?

8. 12일차가 끝난 뒤 왜 다시 Raspberry Pi를 정상 상태로 복원해야 하는가?
```

자신의 답을 `reports/day12_rpi_jetson.md`의 `Mini Challenge` 부분에 작성합니다.

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
11일차에서 정상 검증한 Raspberry Pi 상태를
기준점으로 먼저 고정한다.
        ↓
같은 Source를 Jetson에 준비하되
.venv는 장치별로 새로 만든다.
        ↓
decision / stabilizer / runtime_state /
performance처럼 장치 독립 Logic은 재사용한다.
        ↓
camera_input / inference runtime /
resource monitor처럼 Hardware와 Runtime에
가까운 부분은 Jetson 환경에 맞게 바뀔 수 있다.
        ↓
ONNX와 Metadata를 Jetson으로 옮기고
Hash로 같은 파일인지 증명한다.
        ↓
ONNX Runtime이 준비되어 있으면
같은 Fixed Image Prediction을 확인한다.
        ↓
TensorRT가 준비되어 있으면
대상 Jetson에서 ONNX를 Engine으로 Build한다.
        ↓
trtexec로 Engine의 Model-only 성능을 확인한다.
        ↓
Camera / Inference Contract를 문서화한다.
        ↓
Raspberry Pi → Jetson Migration Map을 만든다.
        ↓
마지막에는 13일차 Final Acceptance를 위해
Raspberry Pi의 설정과 Service를 정상 Baseline으로 복원한다.
```

`trtexec` 결과는 Model-only Benchmark이므로 Camera·Decision·GPIO·Log·API 전체 End-to-End 성능과 같은 값으로 해석하지 않습니다.

</details>

---

# 51. Mini Challenge 완료 기준

다음 항목을 확인합니다.

```text
Challenge 1
→ 결과를 코드 위치로 역추적

Challenge 2
→ 실행 전 재사용 Module 예측

Challenge 3
→ 설정값만 변경하고 결과 비교

Challenge 4
→ jetson_env_check.py 기능 확장

Challenge 5
→ 일부러 Path 오류 생성·복구

Challenge 6
→ Evidence 파일로 정상 동작 증명

Challenge 7
→ 전체 Migration Pipeline 설명
```

Challenge 3에서 변경한 운영 설정은 반드시 복원합니다.

---

# 52. Jetson 장비가 있는 경우 USB Camera Smoke Test

산업용 Camera SDK를 오늘 미리 구현하지 않습니다.

Jetson에 일반 USB Camera가 연결되어 있고 OpenCV 환경이 준비되어 있다면 3일차의 파일을 **그대로 재사용**하여 Camera 입력 계층만 확인할 수 있습니다.

먼저 Camera 번호를 추측하지 않습니다.

```bash
python -m scripts.camera_index_probe
```

정상 index가 확인되면 `configs/device.local.yaml`처럼 장치별 설정에서 실제 Camera index를 사용합니다.

그다음:

```bash
python -m scripts.camera_test
```

이 실습이 확인하는 범위는 다음입니다.

```text
USB Camera
        ↓
Jetson Linux
        ↓
OpenCV VideoCapture
        ↓
BGR Frame
```

이것만으로 TensorRT AI까지 연결된 End-to-End가 검증된 것은 아닙니다.

### Camera 오류가 발생한다면

```text
오류
→ Camera를 열 수 없음

원인 확인
→ /dev/video* 존재 여부
→ camera_index_probe 결과
→ 다른 Process가 Camera를 점유하는가?
→ 권한 문제인가?

수정
→ 실제 Camera index 사용
→ 점유 Process 종료
→ 수업 장비의 권한 정책 확인

재실행
→ camera_index_probe
→ camera_test
```

확인 명령 예:

```bash
ls -l /dev/video*
```

장치명이 환경마다 다를 수 있으므로 `/dev/video0` 하나만 있다고 단정하지 않습니다.

---

# 53. 오늘의 Jetson 최소 성공 기준 정하기

Jetson 실습은 환경에 따라 세 단계로 나눌 수 있습니다.

```text
Level 1
Jetson 환경 확인
+
공통 Unit Test
+
ONNX / Metadata 전송
+
Hash 동일성
        ↓

Level 2
ONNX Runtime이 준비된 경우
Fixed Image
→ ONNX Prediction
        ↓

Level 3
TensorRT가 준비된 경우
ONNX
→ TensorRT Engine Build
→ trtexec Model-only Benchmark
```

USB Camera가 없어도 Level 1~3의 가능한 범위를 수행할 수 있습니다.

중요한 것은 **수행하지 않은 범위를 PASS라고 기록하지 않는 것**입니다.

```text
ONNX Runtime 없음
→ Fixed Image ONNX Prediction = N/A

TensorRT Runner 없음
→ 실제 TensorRT Image Prediction = N/A

trtexec Engine Build 성공
→ TensorRT Build / Benchmark = PASS
```

---

# 54. Guided Lab — Jetson 자원 상태와 Benchmark를 동시에 관찰하기

Jetson 장비가 있다면 두 Terminal을 사용합니다.

## Terminal A — 자원 상태

```bash
tegrastats
```

`tegrastats`가 PATH에 없다면 앞의 `jetson_env_check.py`에서 확인한 실제 경로를 사용합니다.

몇 줄을 관찰합니다.

```text
CPU
Memory
GPU 사용상태
Temperature
Power 관련 값
```

## Terminal B — TensorRT Benchmark

Engine이 있는 경우:

```bash
trtexec \
  --loadEngine=models/tensorrt/day12_tiny_cnn_fp16.engine \
  --warmUp=200 \
  --duration=10
```

Benchmark가 끝난 뒤 Terminal A에서 `Ctrl+C`로 `tegrastats`를 종료합니다.

### 무엇을 기록할까요?

`reports/day12_rpi_jetson.md`에 다음을 추가합니다.

```md
## Jetson Resource Observation

Benchmark:

Runtime:

Input Shape:

GPU 관찰:

CPU 관찰:

Memory 관찰:

Temperature 관찰:

Power 관련 관찰:

특이사항:
```

정확한 숫자를 다른 팀과 억지로 맞추지 않습니다. Power Mode, Cooling, 주변온도, Background Process에 따라 달라질 수 있습니다.

---

# 55. Jetson에서 전력·열을 성능과 함께 보는 이유

GPU 가속은 추론을 빠르게 만들 수 있지만 Edge 장치는 서버실이 아니라 현장 장비입니다.

```text
성능
+
전력
+
온도
+
안정성
```

을 함께 봅니다.

다음처럼 단순화하지 않습니다.

```text
FPS가 가장 높음
        ↓
무조건 최적
```

예를 들어 높은 처리량이 나오더라도 Temperature가 계속 상승하거나 장시간 운영에서 성능이 흔들린다면 운영 조건을 다시 검토해야 합니다.

오늘은 Power Mode를 새로 튜닝하는 수업이 아닙니다. 현재 장비 상태를 **관찰하고 기록**하는 데 집중합니다.

---

# 56. Raspberry Pi와 Jetson 비교 Experiment 설계하기

12일차의 비교는 “무조건 정확한 속도 대결”이 아니라 **측정 조건을 적고 차이를 해석하는 연습**입니다.

Report에 다음 형식을 작성합니다.

```md
## Device Comparison Experiment

### Hypothesis

Jetson의 GPU Runtime은
Raspberry Pi CPU Runtime과 다른
성능 특성을 보일 것이다.

### Raspberry Pi Condition

Device:

Runtime:

Precision:

Input Shape:

Batch:

Warm-up:

Measurement Scope:

Tool:

### Jetson Condition

Device:

Runtime:

Precision:

Input Shape:

Batch:

Warm-up:

Measurement Scope:

Tool:

### Result

| Device | Runtime | Mean / Latency | P95 | FPS / Throughput | 비교 수준 |
|---|---|---:|---:|---:|---|
| Raspberry Pi | ONNX Runtime CPU |  |  |  |  |
| Jetson | TensorRT GPU |  |  |  |  |

### Interpretation

같은 조건:

다른 조건:

직접 비교 가능한가?:

참고 비교인가?:

내가 결론낼 수 있는 범위:
```

### 왜 `Changed Variable = Device` 하나라고 쓰지 않을까요?

현재 비교에서는 다음이 동시에 달라질 수 있습니다.

```text
Device
Runtime
Precision
Benchmark Tool
```

따라서 과학적인 One-Variable A/B Test라고 과장하지 않습니다.

11일차에서 배운 “한 번에 한 변수” 원칙을 기억하고, 오늘 비교가 그 원칙을 완전히 만족하지 않는다면 **그 한계를 Report에 적는 것**이 올바른 분석입니다.

---

# 57. 공정한 비교가 어려운 경우 표시 방법

다음 중 하나라도 다르면 숫자만 보고 순위를 만들지 않습니다.

```text
Runtime이 다름
측정 명령이 다름
Input Batch가 다름
Camera 포함 여부가 다름
Model Precision이 다름
FP32 vs FP16
Warm-up이 다름
통계 계산 방식이 다름
```

Report의 `비교 수준`에 다음 중 하나를 적습니다.

```text
DIRECT
→ 주요 측정 조건을 충분히 맞춤

REFERENCE
→ 조건 차이가 있어 참고용 비교
```

측정하지 않은 값은:

```text
N/A
```

로 기록합니다.

---

# 58. 12일차 최종 Architecture 작성하기

`docs/day12_migration_map.md` 마지막에 다음 구조를 완성합니다.

```text
[Course 13]

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
Decision / Stabilization
     ↓
GPIO / Log
     ↓
systemd / FastAPI


            ↓ 확장


[Course 14]

Industrial Camera
     ↓
Camera Driver / SDK
     ↓
Jetson
     ↓
Preprocess
     ↓
Detection Model
     ↓
ONNX / TensorRT GPU
     ↓
Inspection Decision
     ↓
Log / Statistics
     ↓
systemd / API
```

---

# 59. 무엇이 그대로이고 무엇이 바뀌는지 마지막으로 표시하기

위 Architecture에 다음 표시를 직접 추가합니다.

```text
[REUSE]
→ 그대로 가져갈 개념 / Module

[CHANGE]
→ 장치에 맞게 바꿀 부분

[NEW]
→ 교과 14에서 새로 배울 부분
```

예:

```text
[CHANGE] USB Camera
→ Industrial Camera

[CHANGE] ONNX Runtime CPU
→ TensorRT GPU

[REUSE] Decision

[REUSE] Performance Measurement

[REUSE] systemd

[REUSE] FastAPI 개념
```

---

# 60. 12일차 Report 최종 형식

`reports/day12_rpi_jetson.md`를 다음 항목까지 완성합니다.

```md
# Day 12 Raspberry Pi → Jetson

## 1. Module Portability

## 2. Raspberry Pi Environment

## 3. Jetson Environment

## 4. CPU vs GPU Edge AI

## 5. ONNX vs TensorRT

## 6. Baseline Model / Runtime Comparison

## 7. Camera Layer

## 8. Inference Layer

## 9. Reused Modules

## 10. Changed Modules

## 11. Course 13 → Course 14 Mapping

## 12. Challenge Answers

### 왜 Raspberry Pi에 TensorRT를 적용하지 않았는가?

### Jetson에서 가장 많이 바뀌는 Module은?

### 교과 13에서 그대로 재사용되는 경험은?

## 13. Final Architecture

## 14. 내가 교과 14에서 가장 먼저 확인할 것
```

---

# PART H. 13일차 Final Acceptance를 위해 기본 상태로 복원하고 마무리하기

# 61. Challenge와 실험 변경사항 정리하기

12일차에는 Jetson 환경 확인과 Challenge를 위해 Source나 설정을 임시로 바꿀 수 있었습니다.

먼저 PC와 Jetson에서 각각 확인합니다.

```bash
git status
git diff
```

다음 두 종류를 구분합니다.

```text
유지할 변경
→ jetson_env_check.py 개선
→ docs/
→ reports/
→ README

임시 변경
→ Challenge용 ai_confidence_threshold
→ 잘못된 경로 실험
→ 임시 파일
```

임시 설정이 남아 있으면 기본 상태로 복원합니다.

`configs/settings.yaml`을 Challenge에서만 바꿨다면:

```bash
git restore configs/settings.yaml
```

하지만 `configs/device.local.yaml`은 Git에서 제외될 수 있으므로 `git restore`로 돌아오지 않을 수 있습니다. 실제 장치별 값을 직접 확인합니다.

```bash
cat configs/device.local.yaml
```

Camera index, device_name 등 장치별 값이 원래 의도와 같은지 확인합니다.

---

# 62. Raspberry Pi를 13일차용 운영 Baseline으로 복원하기

바로 다음 13일차는 Jetson이 아니라 **Raspberry Pi Final Acceptance Test**에서 시작합니다.

PC에서 Raspberry Pi에 접속합니다.

```bash
ssh <RPI_USER>@<RPI_IP>
```

프로젝트:

```bash
cd ~/ai_vision/subject13_edge_ai
```

현재 Branch와 변경사항:

```bash
git branch --show-current
git status
```

11일차 Tested Baseline과 `settings.yaml` 차이를 확인합니다.

```bash
git diff \
  day11-rpi-tested-baseline \
  -- configs/settings.yaml
```

의도하지 않은 12일차 실험 변경이 남아 있고 **11일차 운영값으로 돌아가는 것이 맞다는 것을 확인한 경우에만** 다음처럼 복원합니다.

```bash
git restore \
  --source day11-rpi-tested-baseline \
  -- configs/settings.yaml
```

`configs/device.local.yaml`은 장치별 설정이므로 자동으로 덮어쓰지 않습니다.

현재 중요한 값을 확인합니다.

```bash
grep -E \
"camera_index|onnx_model_path|onnx_meta_path|ai_confidence_threshold|inference_every_n_frames|vote_window|vote_warning_min|debounce_required|warning_hold_sec|api_port|ai_event_log_path" \
configs/settings.yaml \
configs/device.local.yaml 2>/dev/null
```

정확한 값은 장치와 11일차 최종 선택에 따라 다릅니다. 교재의 숫자를 복사하지 말고 자신의 Baseline을 확인합니다.

---

## 62-1. Raspberry Pi Service를 정상 상태로 되돌리기

실험 중 Service를 중지했다면 다시 시작합니다.

```bash
sudo systemctl restart \
  subject13-edge.service \
  subject13-api.service
```

상태:

```bash
systemctl is-active subject13-edge.service
systemctl is-active subject13-api.service
```

예상:

```text
active
active
```

자동실행:

```bash
systemctl is-enabled subject13-edge.service
systemctl is-enabled subject13-api.service
```

예상:

```text
enabled
enabled
```

하나라도 다르면 다음 순서로 확인합니다.

```text
오류
        ↓
systemctl status
        ↓
journalctl
        ↓
WorkingDirectory / Python / Config / Camera / Model
        ↓
수정
        ↓
restart
        ↓
is-active / is-enabled 재확인
```

최근 Journal:

```bash
journalctl \
  -u subject13-edge.service \
  -n 30 \
  --no-pager
```

```bash
journalctl \
  -u subject13-api.service \
  -n 30 \
  --no-pager
```

---

## 62-2. 13일차 직전 API와 Runtime State 확인하기

API Port:

```bash
grep -E "^api_port:"   configs/settings.yaml   configs/device.local.yaml 2>/dev/null
```

Raspberry Pi 내부:

```bash
curl http://127.0.0.1:<API_PORT>/health
```

PC:

```bash
curl http://<RPI_IP>:<API_PORT>/health
```

Runtime State:

```bash
ls -lh runtime/edge_state.json
cat runtime/edge_state.json
```

몇 초 뒤 파일 수정시각이 갱신되는지 확인하려면:

```bash
stat runtime/edge_state.json
```

다시 잠시 후:

```bash
stat runtime/edge_state.json
```

### Operational AI Event Log 경로도 확인하기

10~11일차에서 운영 Runtime의 최종 State 변화가 `AIEventLogger`를 통해 Event CSV로 남도록 연결하고 System Test했습니다.

12일차에서는 새로운 Event를 억지로 만들 필요는 없지만, **13일차가 확인할 실제 운영 Log 경로가 어떤 파일인지** 다시 확인해 둡니다.

```bash
python - <<'PY'
from pathlib import Path

from src.config_loader import load_config

config = load_config("configs/settings.yaml")
path = Path(str(config["ai_event_log_path"]))

print(f"ai_event_log_path: {path}")
print(f"exists: {path.exists()}")

if path.exists():
    print(f"size_bytes: {path.stat().st_size}")
PY
```

파일이 존재한다면 최근 기록을 확인할 수 있습니다.

```bash
EVENT_LOG_PATH="$(
  python -c   'from src.config_loader import load_config; print(load_config("configs/settings.yaml")["ai_event_log_path"])'
)"

tail -n 5 "$EVENT_LOG_PATH"
```

상태 변화가 오랫동안 없었다면 마지막 Timestamp가 현재 시각과 같지 않을 수 있습니다. **12일차 Handoff에서는 경로와 파일의 연결을 확인하고, 실제 NORMAL↔WARNING 전환으로 새 Row가 생성되는 최종 증명은 13일차 Acceptance에서 수행합니다.**

13일차에서 가장 먼저 검증할 것은 **재부팅 후 사람이 Python을 직접 실행하지 않아도 이 운영 구조가 자동으로 살아나는가**입니다.

오늘은 그 직전 정상 상태까지만 확인하고, 최종 재부팅 Acceptance는 13일차에 수행합니다.

---

## 62-3. 13일차 Handoff Gate

다음 항목이 모두 PASS인지 체크합니다.

- [ ] Raspberry Pi에 SSH 접속할 수 있다.
- [ ] `subject13-edge.service`가 `active`이다.
- [ ] `subject13-api.service`가 `active`이다.
- [ ] 두 Service가 `enabled`이다.
- [ ] `/health`가 현재 상태를 정상적으로 반환한다.
- [ ] `runtime/edge_state.json`이 갱신된다.
- [ ] 현재 `onnx_model_path` / `onnx_meta_path`가 11일차에서 실제 채택한 운영 Baseline과 일치한다.
- [ ] 두 Model / Metadata 파일이 실제로 존재한다.
- [ ] 11일차 운영 Threshold와 안정화 값이 복원되어 있다.
- [ ] `ai_event_log_path`가 실제 운영 Event CSV를 가리키며 파일 경로를 설명할 수 있다.
- [ ] Camera index 등 `device.local.yaml` 장치값이 올바르다.
- [ ] `python -m pytest tests -v`가 정상이다.
- [ ] Raspberry Pi에 Challenge용 임시 설정이 남아 있지 않다.
- [ ] Jetson 결과는 `reports/day12_rpi_jetson.md`에 기록되어 있다.
- [ ] TensorRT Engine을 Source Repository에 Commit하지 않는다.

이 Gate가 12일차와 13일차를 연결합니다.

---

# 63. 12일차 Source와 문서를 Git Checkpoint로 남기기

Git에 저장하기 전에 먼저 **어느 장치의 프로젝트가 Source 관리 기준인지** 확인합니다.

기본 권장 흐름이 PC 기준이라면 PC 프로젝트에서 진행합니다.

```bash
cd ~/ai_vision/subject13_edge_ai
git status
```

Jetson에서 작성한 Evidence를 PC Repository에도 보관하려면 허용된 방법으로 먼저 가져옵니다.

예를 들어 Network와 보안정책이 허용될 때 PC에서:

```bash
mkdir -p reports/day12_evidence
```

```bash
scp \
  <JETSON_USER>@<JETSON_IP>:~/ai_vision/subject13_edge_ai/reports/day12_evidence/*.txt \
  reports/day12_evidence/
```

파일이 실제로 존재할 때만 실행합니다.

오늘 Git에 남길 수 있는 대표 파일:

```text
scripts/jetson_env_check.py

docs/day12_camera_adapter.md
docs/day12_inference_contract.md
docs/day12_migration_map.md

reports/day12_rpi_jetson.md
reports/day12_evidence/*.txt
```

Jetson 장비가 없어서 제공 자료를 사용했다면 다음 파일도 포함할 수 있습니다.

```text
reports/day12_jetson_provided.csv
reports/day12_rpi_result.csv
```

Stage 전 실제 파일을 확인합니다.

```bash
git status
```

필요한 파일만 Stage합니다.

```bash
git add \
  scripts/jetson_env_check.py \
  docs/day12_camera_adapter.md \
  docs/day12_inference_contract.md \
  docs/day12_migration_map.md \
  reports/day12_rpi_jetson.md
```

Evidence 파일이 있다면:

```bash
git add reports/day12_evidence
```

제공 Benchmark CSV가 있다면:

```bash
git add \
  reports/day12_jetson_provided.csv \
  reports/day12_rpi_result.csv
```

확인:

```bash
git status
```

Commit:

```bash
git commit -m \
  "docs: record raspberry pi to jetson migration"
```

---

# 64. TensorRT Engine과 Model은 Source Repository와 분리하기

TensorRT Engine을 만들었다면 다음과 같은 Binary 파일이 생깁니다.

```text
models/tensorrt/day12_tiny_cnn_fp16.engine
```

ONNX Model도 Source Code가 아닙니다.

```text
<BASELINE_ONNX>
→ 현재 11일차 Tested Baseline이 실제 참조하는 ONNX
```

1일차부터 사용한 `.gitignore` 원칙을 그대로 적용합니다.

```gitignore
*.onnx
*.engine
```

현재 Git 대상인지 확인합니다.

```bash
git status
```

Binary Model과 Engine이 일반 Source Commit에 포함되지 않아야 합니다.

왜 분리할까요?

```text
Source Code
→ Git에서 변경 이력 관리

Model / Engine / Dataset
→ 용량·배포·보안·장치 종속성을 고려해 별도 관리
```

특히 TensorRT Engine은 대상 Hardware와 Runtime 환경의 영향을 받으므로 Source와 같은 방식으로 공유하는 파일로 생각하지 않습니다.

---

# 65. 직접 측정값과 제공 자료를 구분해서 관리하기

12일차에는 다음 두 종류 결과가 섞일 수 있습니다.

```text
measured
→ 내가 실제 장비에서 측정

provided
→ 제공된 화면 / CSV / 시연 결과를 분석
```

CSV나 Report에 `source` 또는 `결과 출처`를 반드시 남깁니다.

예:

```csv
device,runtime,processor,input_size,mean_latency_ms,p95_ms,fps,source
Raspberry Pi,ONNX Runtime,CPU,<RPI_RESULT_INPUT_SIZE>,,,,measured
Jetson,TensorRT,GPU,<PROVIDED_INPUT_SIZE>,,,,provided
```

값이 없으면 빈칸 또는 `N/A`로 둡니다.

---

# 66. README에 12일차 실행·Migration 정보를 추가하기

## README는 왜 수정할까요?

12일차 Report는 자세한 실험 기록입니다. README는 프로젝트를 처음 보는 사람이 **어디서 무엇을 확인해야 하는지 빠르게 찾는 입구** 역할을 합니다.

`README.md`에 다음 정도를 추가합니다.

````md
## Day 12 Raspberry Pi → Jetson

### Goal

11일차까지 검증한 Raspberry Pi Edge Pipeline을 기준으로
Jetson에서 재사용할 Module과 변경할 Module을 구분한다.

### Jetson Environment Check

```bash
python -m scripts.jetson_env_check
```

### Raspberry Pi Baseline

실제 운영 Model은 특정 파일명으로 고정하지 않고
`configs/settings.yaml`의 `onnx_model_path`와 `onnx_meta_path`를 확인한다.
11일차에서 별도 후보를 공식 채택하지 않았다면 기존 Baseline을 유지한다.

```text
USB Camera
→ OpenCV
→ ONNX Runtime CPU
→ Decision
→ Stabilization
→ GPIO / Log
→ systemd / FastAPI
```

### Jetson Expansion

```text
Camera Layer
→ Jetson
→ ONNX 또는 TensorRT
→ GPU Inference
→ Decision / Log / API
```

### Main Reuse

```text
Config 구조
Decision
Stabilization
Runtime State
Event Log
Performance 통계
Test 관점
systemd 개념
FastAPI 개념
```

### Main Change

```text
Camera Driver / SDK
Inference Runtime
GPU / Power / Thermal Monitoring
장치별 Service 경로
```

### Evidence

- `reports/day12_rpi_jetson.md`
- `docs/day12_camera_adapter.md`
- `docs/day12_inference_contract.md`
- `docs/day12_migration_map.md`

### Important

`trtexec` Benchmark는 TensorRT Engine의 Model-only 성능 확인이며,
Camera → Preprocess → Decision → GPIO → Log → API 전체 End-to-End FPS와
같은 값으로 해석하지 않는다.

### Day 13 Handoff

12일차 종료 시 Raspberry Pi의 운영 Config와 두 systemd Service를
11일차 Tested Baseline 기준으로 정상 복원한다.
````

Stage:

```bash
git add README.md
```

Commit:

```bash
git commit -m \
  "docs: add day12 jetson migration guide"
```

---

# 67. Remote 저장소가 있는 경우 동기화하기

기관에서 허용된 내부 GitLab / Gitea Remote가 연결되어 있다면:

```bash
git remote -v
```

현재 Branch와 상태를 확인합니다.

```bash
git branch --show-current
git status
```

허용된 Remote로 Push합니다.

```bash
git push
```

Remote가 없거나 외부 Git 사용이 금지된 환경에서는 Push가 필수가 아닙니다.

```text
Local Git
→ Version 이력 관리

내부 GitLab / Gitea
→ 기관 정책상 허용될 때 사용

외부 GitHub
→ 허용 여부 확인 없이 사용하지 않음
```

Dataset, 기업 제공 Camera Data, Runtime Log, ONNX Model, TensorRT Engine은 보안·용량·정책에 맞게 별도 관리합니다.

---

# 68. 12일차가 끝난 시점의 프로젝트 구조

공통 Source와 문서:

```text
subject13_edge_ai/
│
├── configs/
│   ├── settings.yaml
│   └── device.local.yaml          ← 장치에 따라 존재, 보통 Git 제외
│
├── docs/
│   ├── ...
│   ├── day12_camera_adapter.md
│   ├── day12_inference_contract.md
│   └── day12_migration_map.md
│
├── reports/
│   ├── ...
│   ├── day11_test_analysis.md
│   ├── day12_rpi_jetson.md
│   └── day12_evidence/
│       ├── jetson_env.txt
│       ├── jetson_model_hash.txt
│       └── tensorrt_files.txt
│
├── scripts/
│   ├── ...
│   └── jetson_env_check.py
│
├── src/
│   └── ...
│
├── tests/
│   └── ...
│
├── systemd/
│   └── generated/
│
├── .gitignore
├── README.md
└── requirements.txt
```

Jetson 장비가 없는 경우에는 다음 비교 CSV가 있을 수 있습니다.

```text
reports/
├── day12_jetson_provided.csv
└── day12_rpi_result.csv
```

Jetson 장비에만 존재할 수 있는 배포 Binary:

```text
models/
├── <현재 Baseline ONNX>
├── <현재 Baseline Metadata>
└── tensorrt/
    └── day12_tiny_cnn_fp16.engine
```

---

# 69. 1~12일차 시스템 성장 확인하기

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
systemd
→ 자동실행
→ 장애 복구
→ FastAPI
```

11일차:

```text
Unit Test
→ System Test
→ Failure Analysis
→ Trade-off
```

12일차:

```text
Raspberry Pi
CPU + ONNX Runtime
        ↓
Edge Pipeline 재사용
        ↓
Jetson
GPU + TensorRT
        ↓
교과 14 제조 Vision AI
```

---

# 70. 핵심 복습 문제

문서를 보지 않고 먼저 답합니다.

### 문제 1

Raspberry Pi에서 검증한 Source를 Jetson으로 가져갈 때 `.venv`까지 그대로 복사하면 안 되는 이유는 무엇인가요?

### 문제 2

`decision.py`, `stabilizer.py`, `runtime_state.py`가 Camera Driver보다 재사용하기 쉬운 이유는 무엇인가요?

### 문제 3

ONNX와 TensorRT의 역할 차이는 무엇인가요?

### 문제 4

왜 TensorRT Engine은 대상 Jetson에서 생성하는 것이 기본 원칙인가요?

### 문제 5

`trtexec`로 Engine Build와 Benchmark가 성공하면 `normal.jpg`의 Class Prediction까지 검증되었다고 말할 수 있나요?

### 문제 6

PC와 Jetson의 ONNX 파일명이 같으면 같은 모델이라고 충분히 증명된 것인가요?

### 문제 7

Jetson에서 `trtexec`가 `command -v trtexec`로 나오지 않는다면 바로 TensorRT가 없다고 결론내려도 되나요?

### 문제 8

Raspberry Pi의 Model-only Benchmark와 Jetson의 `trtexec` 결과를 비교할 때 무엇을 함께 기록해야 하나요?

### 문제 9

산업용 Camera Adapter가 다른 SDK를 사용하더라도 이후 Pipeline을 재사용하기 쉽게 만들려면 어떤 출력 Contract를 맞추는 것이 좋을까요?

### 문제 10

Inference Runtime이 ONNX Runtime에서 TensorRT로 바뀌어도 `decision.py`를 재사용하기 위해 최소한 어떤 Prediction 결과를 유지해야 하나요?

### 문제 11

Jetson GPU가 더 빠르면 Voting·Debounce·Hold 같은 안정화 Logic이 필요 없어지나요?

### 문제 12

Jetson에서 `pytest`의 Decision·Stabilizer·Runtime State Test가 모두 PASS했다면 무엇을 증명한 것인가요?

### 문제 13

12일차 마지막에 Raspberry Pi Service와 설정을 다시 확인해야 하는 이유는 무엇인가요?

### 문제 14

13일차에 가장 먼저 확인하는 핵심 운영 질문은 무엇인가요?

---

## 예시 정답 확인해보기

<details>
<summary><strong>복습 문제 예시 정답 열기</strong></summary>

### 정답 1

`.venv`에는 Python 버전, OS, CPU Architecture, Binary Package와 연결된 환경이 들어갈 수 있습니다. Source는 공유하더라도 가상환경은 대상 장치에서 다시 만드는 것이 안전합니다.

### 정답 2

`decision.py`, `stabilizer.py`, `runtime_state.py`는 주로 문자열·숫자·시간·파일 같은 Logic을 다루고 특정 Camera Driver나 GPU Runtime에 직접 의존하지 않기 때문입니다.

### 정답 3

```text
ONNX
→ 모델 교환 / 배포 형식

TensorRT
→ NVIDIA GPU에서 추론을 최적화하는 Runtime
```

둘은 같은 것이 아닙니다.

### 정답 4

TensorRT Engine은 GPU Architecture, TensorRT·CUDA 버전, Builder 옵션, Precision 등의 영향을 받을 수 있으므로 대상 Jetson 환경에서 Build하는 것이 기본 원칙입니다.

### 정답 5

아닙니다.

`trtexec` Build·Benchmark는 Engine 생성과 Runtime 성능을 확인합니다. 실제 이미지 Prediction에는 Image 읽기, Preprocess, Engine 실행, Postprocess, Class Mapping까지 연결된 별도 Runner가 필요합니다.

### 정답 6

아닙니다.

```bash
sha256sum
```

으로 ONNX와 Metadata의 실제 내용이 같은지 비교해야 합니다.

### 정답 7

아닙니다.

PATH에 없을 수 있으므로 예를 들어 다음 위치도 확인할 수 있습니다.

```text
/usr/src/tensorrt/bin/trtexec
```

수업 장비의 실제 설치 위치를 확인해야 합니다.

### 정답 8

최소한 다음을 함께 기록합니다.

```text
Device
Runtime
Precision
Input Shape
Batch
Warm-up
측정 구간
Benchmark Tool
통계 방식
```

조건이 다르면 `REFERENCE` 비교라고 명시합니다.

### 정답 9

이후 OpenCV 기반 Pipeline이 BGR Frame을 기대한다면 Camera Adapter가 최종적으로 **BGR `numpy.ndarray` Frame**을 반환하도록 Contract를 맞추면 재사용하기 쉽습니다.

### 정답 10

현재 운영 판정에 필요한 최소 결과는 다음입니다.

```python
{
    "class_name": "...",
    "confidence": 0.0,
}
```

Runtime 내부 구현이 바뀌어도 이 Contract를 맞추면 `decision.py`를 재사용하기 쉽습니다.

### 정답 11

아닙니다.

GPU가 빨라져도 Frame별 Prediction이 경계 상황에서 흔들릴 수 있으므로 시간적 안정화 문제는 별도로 남을 수 있습니다.

### 정답 12

장치가 바뀌어도 해당 **장치 독립 Logic과 그 Test 구조를 재사용할 수 있음**을 증명한 것입니다. Camera·GPIO·TensorRT까지 모두 정상이라는 뜻은 아닙니다.

### 정답 13

바로 다음 13일차가 Raspberry Pi Final Acceptance Test로 시작하기 때문입니다. Challenge용 Threshold나 중지된 Service가 남아 있으면 최종 Acceptance 결과가 왜곡될 수 있습니다.

### 정답 14

```text
Raspberry Pi를 재부팅한 뒤
사람이 Python을 직접 실행하지 않아도
Edge AI와 FastAPI가 자동으로 정상 운영되는가?
```

입니다.

</details>

---

# 71. 오늘 반드시 남아 있어야 하는 결과

```text
1. day11-rpi-tested-baseline 기준점

2. Raspberry Pi vs Jetson Module Portability 표

3. Raspberry Pi / Jetson 환경 기록

4. scripts/jetson_env_check.py

5. 11일차 Tested Baseline의 실제 ONNX / Metadata / Input Size 기록

6. 같은 Baseline ONNX / Metadata Hash 비교 결과

7. Jetson ONNX Runtime 결과
   또는 N/A 사유

8. TensorRT Engine Build 결과
   또는 제공 자료 분석 / N/A 사유

9. TensorRT Benchmark 결과와 측정 범위

10. Jetson Resource 관찰 결과

11. docs/day12_camera_adapter.md

12. docs/day12_inference_contract.md

13. docs/day12_migration_map.md

14. reports/day12_rpi_jetson.md

15. Mini Challenge 결과

16. Raspberry Pi 13일차 Handoff Gate PASS

17. README Day 12 섹션

18. Git Commit
```

없는 장비나 Runtime의 결과는 억지로 채우지 않습니다.

```text
직접 측정
→ measured

제공 자료
→ provided

수행하지 못함
→ N/A + 이유
```

로 구분합니다.

---

# 72. 자가 체크리스트

오늘 수업을 마치기 전에 직접 체크합니다.

- [ ] 11일차 Tested Baseline이 정상인지 확인했다.
- [ ] `day11-rpi-tested-baseline` Tag의 의미를 설명할 수 있다.
- [ ] Raspberry Pi와 Jetson에서 재사용할 Module과 변경할 Module을 구분할 수 있다.
- [ ] CPU와 GPU의 역할을 구분할 수 있다.
- [ ] JetPack·CUDA·TensorRT의 관계를 설명할 수 있다.
- [ ] ONNX와 TensorRT를 같은 개념으로 설명하지 않는다.
- [ ] Jetson의 Hostname·IP·Architecture를 실제 명령으로 확인했다.
- [ ] Raspberry Pi `.venv`를 Jetson으로 복사하지 않았다.
- [ ] Jetson에서 사용한 Python 위치를 확인했다.
- [ ] 장치 독립 Unit Test를 재사용했다.
- [ ] 특정 파일명을 추측하지 않고 11일차 운영 Config에서 Baseline ONNX / Metadata를 확인했다.
- [ ] 11일차에서 후보를 비교한 것과 실제 운영 Baseline으로 채택한 것을 구분할 수 있다.
- [ ] 현재 운영 Baseline의 ONNX와 Metadata를 Jetson으로 전송했다.
- [ ] ONNX와 Metadata Hash를 비교했다.
- [ ] 비교표의 Input Size는 Baseline이나 제공 자료의 실제 값으로 기록하고, 다른 조건을 같은 값처럼 맞추지 않았다.
- [ ] TensorRT Engine은 대상 Jetson에서 Build해야 하는 이유를 설명할 수 있다.
- [ ] `trtexec` Model-only Benchmark와 End-to-End 성능을 구분할 수 있다.
- [ ] 직접 측정값과 제공 자료의 값을 구분해서 기록했다.
- [ ] Camera Adapter Contract를 작성했다.
- [ ] Inference Contract를 작성했다.
- [ ] Migration Map을 작성했다.
- [ ] Mini Challenge 7단계를 완료했다.
- [ ] Challenge에서 바꾼 운영 설정을 복원했다.
- [ ] Raspberry Pi 두 Service가 `active`이다.
- [ ] Raspberry Pi 두 Service가 `enabled`이다.
- [ ] Raspberry Pi `/health`가 정상이다.
- [ ] `runtime/edge_state.json`이 갱신되는지 확인했다.
- [ ] `ai_event_log_path`가 가리키는 실제 운영 Event Log 경로를 확인했다.
- [ ] README를 정리했다.
- [ ] Git에 Model·TensorRT Engine이 포함되지 않았는지 확인했다.
- [ ] Git Commit과 Report를 완료했다.
- [ ] 13일차가 왜 Raspberry Pi Final Acceptance로 이어지는지 설명할 수 있다.

---

# 73. 다음 날 연결 — 13일차 Final Acceptance Test

12일차까지 새로운 핵심 기술 학습은 대부분 끝났습니다.

오늘은 다음 두 결과를 남겼습니다.

```text
A. Raspberry Pi
→ 11일차 Tested Baseline
→ 실제 채택된 ONNX / Metadata / 운영 Config
→ 13일차 Final Acceptance를 위해 정상 복원

B. Jetson
→ 환경 확인
→ ONNX / TensorRT 확장 경험
→ Migration Map
→ 교과 14 준비
```

바로 다음 13일차에서는 **Jetson 기능을 더 추가하지 않습니다.**

```text
Raspberry Pi 전원 / Reboot
        ↓
사람이 Python을 직접 실행하지 않음
        ↓
systemd 자동실행
        ↓
Camera
        ↓
ONNX AI
        ↓
Decision
        ↓
Stabilization
        ↓
GPIO / Event Log
        ↓
Runtime State
        ↓
FastAPI
        ↓
PC 상태 확인
        ↓
장애 복구
        ↓
Acceptance Test
        ↓
운영 문서
        ↓
Final Release
```

13일차의 핵심 질문은 다음입니다.

> **“이 Edge AI 시스템을 다른 사람이 전원을 켜고 상태를 확인하며, 문제가 생겼을 때 로그를 보고 다시 운영할 수 있는가?”**

따라서 12일차 마지막에 Raspberry Pi를 정상 Baseline으로 복원한 것이 다음 수업의 출발점이 됩니다.

13일차가 끝나면 교과 13에서 배운 Edge AI 운영 경험을 교과 14의 Jetson 기반 제조 Vision AI로 연결합니다.

---

# 오늘의 한 문장 정리

> **12일차는 Raspberry Pi에서 검증한 Edge AI 구조를 Jetson으로 옮기며 “무엇을 재사용하고 무엇을 장치에 맞게 바꿔야 하는가”를 실제 환경·모델·Runtime·성능·Contract로 확인하고, 13일차 Final Acceptance를 위해 Raspberry Pi를 다시 정상 운영 상태로 넘기는 날입니다.**
