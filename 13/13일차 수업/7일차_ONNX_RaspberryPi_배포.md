
> **오늘의 핵심:** 6일차에는 Raspberry Pi Camera로 직접 수집한 `normal / warning_red` Dataset을 PC로 가져와 Tiny CNN을 학습하고, 최종 PyTorch Baseline인 `models/day06_tiny_cnn.pt`를 만들었습니다.  
> 7일차에는 이 모델을 **ONNX로 변환하고, PC와 Raspberry Pi에서 결과가 일치하는지 확인한 뒤, 실제 Camera → AI → Decision → GPIO → Log Pipeline에 연결**합니다.
>
> 1~6일차에 사용한 `subject13_edge_ai` 프로젝트를 그대로 이어서 사용합니다. **새 프로젝트와 새 가상환경을 만들지 않습니다.**
>
> 7일차의 목표는 “ONNX 파일 하나 만들기”가 아닙니다. **Model · Metadata · Preprocess · Class 순서 · 입력 Shape · 결과가 환경을 바꾸어도 같은 의미를 유지하는지 확인하는 배포 과정**을 경험하는 것이 핵심입니다.
>
> 실제 Raspberry Pi 사용자 이름, Hostname, IP, Camera index, Python 버전, ONNX Runtime 설치 방법은 환경마다 달라질 수 있습니다. 교재의 `<RPI_USER>`, `<RPI_IP>` 같은 표시는 자신의 실제 환경값으로 바꾸어 사용합니다.
>
> 비밀번호, Wi-Fi 비밀번호, 인증키 같은 민감한 정보는 README·Report·Git에 기록하지 않습니다.

---

## 오늘 가장 중요한 질문

6일차까지 AI 모델은 PC 안에 있었습니다.

```text
Raspberry Pi Camera
        ↓
Session Dataset
        ↓
PC
        ↓
Tiny CNN Training
        ↓
models/day06_tiny_cnn.pt
```

오늘은 이 모델을 실제 Edge 장치로 옮깁니다.

```text
6일차 PyTorch Model
        ↓
ONNX Export
        ↓
ONNX 구조 확인
        ↓
PyTorch vs ONNX 결과 비교
        ↓
Model + Metadata + 동일 Test Image
        ↓
Raspberry Pi 전송
        ↓
ONNX Runtime CPU 추론
        ↓
USB Camera
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

따라서 오늘 가장 중요한 질문은 다음입니다.

> **“PC에서 학습한 Tiny CNN을 ONNX로 바꾼 뒤, 입력과 전처리와 Class 의미를 그대로 유지하여 Raspberry Pi에서 같은 결과를 내고 실제 Edge Pipeline에 연결하려면 무엇을 확인해야 하는가?”**

---

## 6일차와 7일차는 어디가 이어질까요?

6일차에서 이미 다음 항목을 고정했습니다.

```text
Class
→ normal / warning_red

Input
→ 96 × 96 RGB

Model
→ TinyCNN

Best Model
→ models/day06_tiny_cnn.pt

Model 선택
→ Validation

최종 평가
→ Test

단일 이미지 Prediction
→ scripts/predict_tiny_cnn.py
```

7일차에서는 이것을 새로 만들지 않습니다.

```text
재사용

src/tiny_cnn.py
→ 6일차와 같은 Model 구조

models/day06_tiny_cnn.pt
→ 6일차에서 고정한 Weight

data/dataset/test/
→ 비교에 사용할 Test Image

scripts/predict_tiny_cnn.py
→ PyTorch 기준 Prediction
```

오늘 새로 추가되는 핵심은 다음입니다.

```text
ONNX Export
공통 AI Preprocess
ONNX Runtime
Metadata Contract
Operational Decision
Raspberry Pi 배포
실시간 Camera AI
AI Event Log
```

---

## 오늘의 학습 목표

수업이 끝나면 다음을 할 수 있어야 합니다.

```text
1. 6일차 PyTorch Checkpoint와 7일차 ONNX Model의 관계를 설명한다.
2. ONNX를 사용하는 이유를 Edge 배포 관점에서 설명한다.
3. ONNX Model과 Metadata JSON을 함께 관리하는 이유를 설명한다.
4. [1, 3, 96, 96] NCHW Input Shape의 의미를 설명한다.
5. 6일차와 동일한 RGB / Resize / float32 / 0~1 전처리를 재현한다.
6. PyTorch와 ONNX에 같은 Tensor를 넣어 Result Parity를 확인한다.
7. ONNX Runtime의 실제 Input / Output Shape와 Provider를 확인한다.
8. ONNX Model과 동일 Test Image를 Raspberry Pi로 안전하게 전송한다.
9. Raspberry Pi에서 같은 이미지의 ONNX 결과를 PC와 비교한다.
10. AI Prediction과 운영 NORMAL / WARNING Decision을 구분한다.
11. Camera Frame을 ONNX 입력으로 바꾸어 실시간 추론한다.
12. 3일차 GPIO와 5일차 Event Log 구조를 AI Pipeline에 재사용한다.
13. WARNING 상태 전환 시 대표 이미지를 저장한다.
14. 실행 결과가 어느 파일의 어느 코드에서 만들어졌는지 역추적한다.
15. Input Shape / Preprocess / Metadata / Model Path 오류를 구분한다.
16. 8일차 Latency / FPS 측정을 위한 안정된 Day 07 Baseline을 남긴다.
```

---

## 오늘의 8시간 학습 흐름

| 학습 블록 | 권장 시간 | 무엇을 배우나요? | 끝나면 할 수 있어야 하는 것 |
|---|---:|---|---|
| 1 | 45분 | 6일차 Handoff · ONNX 개념 · 환경 확인 | 오늘 무엇을 옮기고 무엇을 유지하는지 설명할 수 있다 |
| 2 | 70분 | ONNX Export · Metadata · Input/Output Contract | PyTorch Model을 ONNX로 내보내고 구조를 확인할 수 있다 |
| 3 | 60분 | 공통 Preprocess · PC ONNX · PyTorch/ONNX Parity | 같은 입력에서 두 Runtime 결과를 비교할 수 있다 |
| 4 | 65분 | Raspberry Pi 전송 · ONNX Runtime · 동일 이미지 검증 | PC의 배포 결과를 Pi에서 재현할 수 있다 |
| 5 | 85분 | Operational Decision · Camera AI · GPIO · Log | 실제 Camera를 AI Warning System으로 연결할 수 있다 |
| 6 | 70분 | Mini Challenge | 역추적·예측·Config 변경·기능 수정·오류 복구를 스스로 수행할 수 있다 |
| 7 | 50분 | Baseline 복원 · End-to-End 확인 | 8일차가 사용할 안정된 Day 07 상태를 만들 수 있다 |
| 8 | 35분 | Report · README · Git · 복습 · 8일차 연결 | 오늘 결과를 재현 가능한 형태로 남길 수 있다 |

총 480분을 기준으로 구성합니다.

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

7일차에서는 다음처럼 대응됩니다.

```text
입력
→ USB Camera Frame

처리
→ BGR → RGB
→ Resize
→ float32 / 255
→ HWC → CHW
→ Batch 추가

판단 1
→ ONNX Runtime
→ Class + Confidence

판단 2
→ warning_red인가?
→ Confidence가 운영 Threshold 이상인가?
→ NORMAL / WARNING

출력
→ Green / Red LED
→ 선택 Buzzer

기록
→ AI Event CSV
→ WARNING 대표 이미지
→ 배포 Report
```

코드를 보다가 헷갈리면 다음 질문으로 돌아옵니다.

> **“지금 보고 있는 코드는 입력·처리·판단·출력·기록 중 어느 역할을 하는가?”**

---

## 오늘 사용할 실제 장비와 데이터

오늘은 6일차 결과를 그대로 사용합니다.

```text
PC
→ 6일차 Tiny CNN PyTorch Model
→ ONNX Export
→ ONNX 검증

Raspberry Pi
→ Raspberry Pi OS / Linux
→ USB Camera
→ Green / Red LED
→ 선택 Buzzer

공통 Test Data
→ data/dataset/test/normal/...
→ data/dataset/test/warning_red/...
```

배포 비교를 위해 Test 이미지 두 장을 따로 고정합니다.

```text
data/deploy_samples/normal.jpg
data/deploy_samples/warning_red.jpg
```

이 두 파일은 **PC와 Raspberry Pi에서 같은 입력을 사용했다는 것을 증명하기 위한 Sample**입니다.

---

## 오늘 배울 기술 스택

| 기술 | 오늘 어디에 사용하나요? |
|---|---|
| PyTorch | 6일차 Checkpoint 로드와 기준 Prediction |
| ONNX | Framework와 실행환경 사이의 교환 가능한 Model 형식 |
| ONNX Runtime | PC와 Raspberry Pi에서 ONNX 추론 |
| NumPy | NCHW float32 Tensor와 Softmax 계산 |
| Pillow | 저장 이미지 RGB/Resize 전처리 |
| OpenCV | Raspberry Pi Camera BGR Frame 입력과 WARNING 이미지 저장 |
| JSON | ONNX와 함께 Class 순서·입력 크기 Metadata 저장 |
| SSH / SCP | PC↔Raspberry Pi 접속과 Model/Test Image 전송 |
| gpiozero | 3일차 GPIOOutput 재사용 |
| CSV | AI 상태 변화 Event 기록 |
| Git | Source·설정·Metadata·Report Version 관리 |

---

## 실습 전에 자신의 환경값 적어두기

```text
PC Project Path           : ______________________________

PC Python                 : ______________________________
PyTorch Version           : ______________________________
ONNX Version              : ______________________________
ONNX Runtime Version      : ______________________________

Raspberry Pi 사용자 이름  : ______________________________
Raspberry Pi Hostname     : ______________________________
Raspberry Pi IPv4         : ______________________________
Raspberry Pi Architecture : ______________________________
Raspberry Pi Python       : ______________________________
Raspberry Pi ORT Version  : ______________________________

OpenCV Camera Index       : ______________________________
Green LED BCM             : ______________________________
Red LED BCM               : ______________________________
Buzzer 사용 여부          : 사용 / 미사용
```

환경에 따라 달라지는 값은 다음처럼 표시합니다.

```text
<RPI_USER>
→ 자신의 Raspberry Pi 사용자 이름

<RPI_IP>
→ 현재 확인한 Raspberry Pi IPv4

<RPI_HOSTNAME>
→ 자신의 Raspberry Pi Hostname

<PC_PROJECT_PATH>
→ PC에서 subject13_edge_ai가 있는 실제 경로

<TEST_NORMAL_IMAGE>
→ 실제 Test normal JPG 경로

<TEST_WARNING_IMAGE>
→ 실제 Test warning_red JPG 경로
```

IP가 바뀌었다면 이전 값을 계속 사용하지 않습니다.

Raspberry Pi에서 직접 확인할 수 있다면:

```bash
hostname
hostname -I
```

을 사용합니다.

---

## 오늘의 성공 기준

7일차가 끝났을 때 다음 연결이 실제로 성립하면 됩니다.

```text
6일차 day06_tiny_cnn.pt 확인
        ↓
PyTorch 단일 이미지 Prediction 정상
        ↓
ONNX Export
        ↓
ONNX Checker PASS
        ↓
Metadata 확인
        ↓
NCHW float32 입력 확인
        ↓
PC ONNX Prediction
        ↓
PyTorch vs ONNX Class Match
        ↓
동일 Test Image 고정 + Hash 확인
        ↓
ONNX + Metadata + Test Image
Raspberry Pi 전송
        ↓
Pi ONNX Runtime 정상
        ↓
PC ONNX vs Pi ONNX Class Match
        ↓
Prediction + Confidence
        ↓
Operational Decision
        ↓
실시간 Camera
        ↓
GPIO
        ↓
AI Event Log
        ↓
WARNING Image
        ↓
Mini Challenge
        ↓
Day 07 Baseline 복원
        ↓
Report / README / Git
        ↓
8일차 Latency / FPS 측정 준비
```

---

## 오늘은 아직 하지 않는 것

```text
새 모델 재학습
→ 6일차에서 끝냄

정식 Latency / FPS Benchmark
→ 8일차

장시간 안정화 / Recovery
→ 9일차

systemd / Headless 운영
→ 10일차
```

오늘은 **배포가 정확하게 연결되는지**에 집중합니다.

---

# PART A. 6일차 Baseline과 ONNX 배포 준비


# 1. 6일차 결과에서 그대로 시작하기

6일차가 끝난 PC 프로젝트에는 다음 핵심 파일이 있어야 합니다.

```text
subject13_edge_ai/
│
├── configs/
│   └── settings.yaml
│
├── models/
│   └── day06_tiny_cnn.pt
│
├── reports/
│   ├── day06_ai_baseline.md
│   ├── day06_training_history.csv
│   ├── day06_test_metrics.csv
│   └── day06_confusion_matrix.csv
│
├── scripts/
│   ├── evaluate_tiny_cnn.py
│   ├── predict_tiny_cnn.py
│   └── train_tiny_cnn.py
│
├── src/
│   └── tiny_cnn.py
│
└── ...
```

6일차의 흐름은 여기까지였습니다.

```text
Camera
→ Session Dataset
→ Train / Val / Test
→ Tiny CNN
→ Test
→ day06_tiny_cnn.pt
```

오늘은 이 모델을 Raspberry Pi가 사용할 수 있는 ONNX 형식으로 바꿉니다.

---

# 2. PC에서 6일차 최종 상태 확인하기

7일차는 **PC의 6일차 Baseline Model**에서 시작합니다.

PC의 WSL2 또는 Linux 터미널에서 프로젝트로 이동합니다.

```bash
cd <PC_PROJECT_PATH>
```

예를 들어 실제 경로가 다음과 같다면:

```bash
cd ~/ai_vision/subject13_edge_ai
```

기존 PC 가상환경을 활성화합니다.

```bash
source .venv/bin/activate
```

현재 위치와 Python을 확인합니다.

```bash
pwd
which python
python --version
```

Git 상태를 확인합니다.

```bash
git status
git log --oneline -8
```

Source를 갱신하는 방법은 교육장 전달 방식에 따라 다릅니다.

### 내부 GitLab / Gitea 같은 허용된 Remote를 사용하는 경우

```bash
git remote -v
git pull
```

### Local Git만 사용하는 경우

`git pull`을 실행할 Remote가 없으므로 억지로 실행하지 않습니다.

```bash
git status
git log --oneline -8
```

로 6일차 마지막 Commit을 확인합니다.

6일차 Model이 존재하는지 확인합니다.

```bash
ls -lh models/day06_tiny_cnn.pt
```

다음 6일차 결과도 확인합니다.

```bash
ls -lh \
  reports/day06_ai_baseline.md \
  reports/day06_training_history.csv \
  reports/day06_test_metrics.csv \
  reports/day06_confusion_matrix.csv
```

```text
모두 준비됨
→ 7일차 시작

day06_tiny_cnn.pt 없음
→ 6일차 Training / Baseline 복구
→ Model이 준비된 뒤 7일차 시작
```

7일차에서 임의로 새 Tiny CNN을 다시 학습하여 대신하지 않습니다.

---

# 3. 6일차 PyTorch Prediction이 정상인지 다시 확인하기

ONNX로 변환하기 전에 원본 모델부터 다시 확인합니다.

Test 이미지 하나를 찾습니다.

```bash
find data/dataset/test/normal \
  -type f -name "*.jpg" | head
```

예시 이미지 한 장을 선택합니다.

```bash
python -m scripts.predict_tiny_cnn \
  --image data/dataset/test/normal/파일명.jpg
```

다음 값을 기록합니다.

```text
Class:
Confidence:
```

`warning_red` 이미지도 한 장 확인합니다.

```bash
find data/dataset/test/warning_red \
  -type f -name "*.jpg" | head
```

```bash
python -m scripts.predict_tiny_cnn \
  --image data/dataset/test/warning_red/파일명.jpg
```

오늘 ONNX 결과는 이 PyTorch 결과와 비교합니다.

---

# 4. 오늘부터 PC와 Raspberry Pi의 Package 목록을 분리해서 보기

6일차까지는 프로젝트가 비교적 단순했지만 이제 역할이 분명하게 나뉩니다.

```text
PC
→ PyTorch Training
→ ONNX Export
→ ONNX 검증

Raspberry Pi
→ ONNX Runtime
→ Camera
→ GPIO
```

따라서 필요한 Package도 완전히 같지 않습니다.

예:

```text
PC
torch
torchvision
onnx
onnxruntime

Raspberry Pi
opencv-python-headless
gpiozero
onnxruntime
PyYAML
Pillow
numpy
```

앞으로는 **PC와 Raspberry Pi의 실제 설치 Package를 각각 기록**합니다.

---

# 5. PC용 ONNX 환경 확인하고 기록하기

오늘은 6일차 PC `.venv`를 그대로 사용합니다.

먼저 현재 Package를 기록합니다.

```bash
mkdir -p reports
python -m pip freeze \
  > reports/day07_pc_environment_before.txt
```

기존에 `onnx`와 `onnxruntime`이 설치되어 있는지 먼저 확인합니다.

```bash
python -c \
"import onnx, onnxruntime; print('ONNX:', onnx.__version__); print('ORT:', onnxruntime.__version__)"
```

정상이라면 다시 설치할 필요가 없습니다.

### Package가 없는 경우

인터넷 사용이 허용된 일반 환경:

```bash
python -m pip install \
  onnx \
  onnxruntime
```

폐쇄망 또는 외부 Package 설치가 제한된 교육장:

```text
교육장에서 검증된 Offline Wheel
또는
허용된 내부 Package 저장소
```

를 사용합니다.

임의의 Wheel 파일이나 다른 Architecture용 Package를 설치하지 않습니다.

설치 후 다시 확인합니다.

```bash
python -c \
"import onnx, onnxruntime; print('ONNX:', onnx.__version__); print('ORT:', onnxruntime.__version__)"
```

최종 PC 환경을 기록합니다.

```bash
python -m pip freeze \
  > reports/day07_pc_environment.txt
```

이 파일은 **PC Export/검증 환경**의 기록입니다. Raspberry Pi 환경과 같을 필요는 없습니다.

---

# 6. 7일차 모델 경로 설정 추가하기

`configs/settings.yaml`에 다음 항목을 추가합니다.

```yaml
onnx_model_path: models/day07_tiny_cnn.onnx
onnx_meta_path: models/day07_tiny_cnn.meta.json

ai_warning_class: warning_red
ai_confidence_threshold: 0.70

ai_event_log_path: logs/day07_ai_events.csv
ai_warning_image_dir: data/ai_warning_images
```

각 항목의 역할:

```text
onnx_model_path
→ Raspberry Pi에서 실행할 ONNX 모델

onnx_meta_path
→ Class 순서와 입력 크기 정보

ai_warning_class
→ 경고로 취급할 AI Class

ai_confidence_threshold
→ AI 예측을 실제 WARNING으로 사용할 최소 Confidence

ai_event_log_path
→ AI 상태 변화 기록

ai_warning_image_dir
→ AI WARNING 대표 이미지 저장
```

---

# 7. ONNX로 무엇을 옮겨야 하는지 먼저 확인하기

PyTorch Checkpoint에는 다음 정보가 들어 있습니다.

```text
model_state
classes
class_to_idx
image_size
best_val_accuracy
```

하지만 ONNX 파일은 주로 **계산 Graph와 Weight**를 담습니다.

따라서 Class 이름과 입력 크기는 별도 Metadata로 저장합니다.

오늘 배포 파일은 두 개입니다.

```text
day07_tiny_cnn.onnx
→ 실제 AI 계산 모델

day07_tiny_cnn.meta.json
→ Class 순서와 Input Size
```

둘을 함께 관리합니다.

---

# PART B. PyTorch → ONNX Export와 PC 검증


# 8. ONNX Export 프로그램 만들기

## 이 파일은 왜 필요할까요?

6일차의 `models/day06_tiny_cnn.pt`는 PyTorch가 읽는 Checkpoint입니다. Raspberry Pi에서 PyTorch 전체 환경을 그대로 가져가는 대신, 7일차에서는 **추론에 필요한 계산 Graph와 Weight를 ONNX로 내보내는 단계**를 경험합니다.

`export_onnx.py`는 단순히 확장자를 바꾸는 프로그램이 아닙니다.

```text
6일차 Checkpoint
→ TinyCNN 구조 다시 생성
→ Weight 로드
→ eval() 모드
→ Dummy Input으로 계산 경로 정의
→ ONNX Export
→ ONNX Checker
→ Metadata JSON 저장
```

ONNX 모델과 Metadata를 함께 만드는 이유는 뒤에서 Class 순서와 입력 크기를 잃지 않기 위해서입니다.

## 의사코드

```text
settings.yaml을 읽는다
        ↓
PyTorch / ONNX / Metadata 경로를 가져온다
        ↓
6일차 PyTorch Model이 실제로 있는지 확인한다
        ↓
Checkpoint를 CPU에서 읽는다
        ↓
classes / image_size / class_to_idx를 꺼낸다
        ↓
같은 TinyCNN 구조를 만든다
        ↓
Checkpoint Weight를 로드한다
        ↓
model.eval()로 추론 모드로 바꾼다
        ↓
[1, 3, image_size, image_size] Dummy Input을 만든다
        ↓
torch.onnx.export()로 ONNX를 만든다
        ↓
onnx.checker로 구조를 검사한다
        ↓
Class 순서와 Input 정보를 Metadata JSON으로 저장한다
        ↓
생성된 두 파일 경로를 출력한다
```


파일:

```text
scripts/export_onnx.py
```

코드:

```python
import json
from pathlib import Path

import onnx
import torch

from src.config_loader import load_config
from src.tiny_cnn import TinyCNN


def main():
    config = load_config(
        "configs/settings.yaml"
    )

    pytorch_path = Path(
        config["ai_model_path"]
    )

    onnx_path = Path(
        config["onnx_model_path"]
    )

    meta_path = Path(
        config["onnx_meta_path"]
    )

    if not pytorch_path.exists():
        raise FileNotFoundError(
            f"PyTorch Model이 없습니다: "
            f"{pytorch_path}"
        )

    checkpoint = torch.load(
        pytorch_path,
        map_location="cpu",
    )

    classes = list(
        checkpoint["classes"]
    )

    image_size = int(
        checkpoint["image_size"]
    )

    model = TinyCNN(
        num_classes=len(classes)
    )

    model.load_state_dict(
        checkpoint["model_state"]
    )

    model.eval()

    dummy_input = torch.randn(
        1,
        3,
        image_size,
        image_size,
        dtype=torch.float32,
    )

    onnx_path.parent.mkdir(
        parents=True,
        exist_ok=True,
    )

    print("=== ONNX Export ===")
    print(f"PyTorch : {pytorch_path}")
    print(f"ONNX    : {onnx_path}")
    print(
        f"Input   : "
        f"[1, 3, {image_size}, {image_size}]"
    )
    print(f"Classes : {classes}")

    torch.onnx.export(
        model,
        dummy_input,
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
        "input_name": "input",
        "output_name": "logits",
        "input_shape": [
            1,
            3,
            image_size,
            image_size,
        ],
        "image_size": image_size,
        "classes": classes,
        "class_to_idx":
            checkpoint["class_to_idx"],
        "pytorch_source":
            str(pytorch_path),
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

    print()
    print("ONNX CHECK: OK")
    print(f"Metadata : {meta_path}")


if __name__ == "__main__":
    main()
```

---

# 9. ONNX Export 실행하기

## 코드 리뷰 — `ONNX CHECK: OK`는 어디에서 만들어졌을까요?

실행 결과만 보고 넘어가지 않습니다.

```text
models/day06_tiny_cnn.pt
        ↓
export_onnx.py
        ↓
TinyCNN 생성
        ↓
load_state_dict()
        ↓
torch.onnx.export()
        ↓
models/day07_tiny_cnn.onnx
        ↓
onnx.load()
        ↓
onnx.checker.check_model()
        ↓
ONNX CHECK: OK
```

직접 코드에서 다음을 찾습니다.

```text
PyTorch Model 경로
→ config["ai_model_path"]

ONNX 저장 경로
→ config["onnx_model_path"]

Input Shape의 96
→ checkpoint["image_size"]

Class 순서
→ checkpoint["classes"]

ONNX 구조 검사
→ onnx.checker.check_model()
```

`ONNX CHECK: OK`는 **Prediction이 정확하다는 뜻이 아닙니다.** ONNX 구조가 검사에 통과했다는 뜻입니다. 결과 일치는 뒤의 Parity Test에서 따로 확인합니다.


PC에서 실행합니다.

```bash
python -m scripts.export_onnx
```

정상 예:

```text
=== ONNX Export ===
PyTorch : models/day06_tiny_cnn.pt
ONNX    : models/day07_tiny_cnn.onnx
Input   : [1, 3, 96, 96]
Classes : ['normal', 'warning_red']

ONNX CHECK: OK
Metadata : models/day07_tiny_cnn.meta.json
```

파일 확인:

```bash
ls -lh \
  models/day07_tiny_cnn.onnx \
  models/day07_tiny_cnn.meta.json
```

---

# 10. Metadata 파일 직접 확인하기

```bash
cat models/day07_tiny_cnn.meta.json
```

예:

```json
{
  "model_type": "TinyCNN",
  "input_name": "input",
  "output_name": "logits",
  "input_shape": [1, 3, 96, 96],
  "image_size": 96,
  "classes": [
    "normal",
    "warning_red"
  ]
}
```

다음 세 가지는 반드시 기억합니다.

```text
Input Shape
[1, 3, 96, 96]

Class 0
normal

Class 1
warning_red
```

Class 순서를 바꾸면 같은 출력 숫자도 의미가 달라집니다.

---

# 11. NCHW를 실제 배열 구조로 보기

ONNX Input:

```text
[1, 3, 96, 96]
```

의 의미:

```text
N = Batch
1장

C = Channel
RGB 3개

H = Height
96

W = Width
96
```

즉:

```text
NCHW
=
1 × 3 × 96 × 96
```

일반 이미지 배열은 보통:

```text
HWC
=
96 × 96 × 3
```

형태일 수 있습니다.

따라서 추론 전에 다음 변환이 필요합니다.

```text
HWC
→ CHW
→ Batch 차원 추가
→ NCHW
```

---

# 12. PyTorch와 ONNX가 함께 사용할 전처리 모듈 만들기

## 이 파일은 왜 필요할까요?

6일차 Training은 `Resize → ToTensor` 흐름을 사용했습니다. 7일차에는 PyTorch와 ONNX Runtime, 그리고 Raspberry Pi Camera가 **같은 의미의 입력 Tensor**를 만들어야 합니다.

전처리 코드를 여러 파일에 복사하면 한쪽만 수정되어 결과가 달라질 수 있습니다. 그래서 이미지 파일과 Camera Frame이 모두 같은 함수를 거쳐 `float32 NCHW`가 되도록 공통 모듈로 분리합니다.

## 의사코드

```text
PIL Image를 받는다
        ↓
RGB로 통일한다
        ↓
96×96 같은 지정 크기로 Resize한다
        ↓
NumPy float32 배열로 바꾼다
        ↓
0~255 값을 0~1 범위로 바꾼다
        ↓
HWC → CHW로 축 순서를 바꾼다
        ↓
Batch 차원을 앞에 추가한다
        ↓
연속 메모리 float32 NCHW 배열을 반환한다

이미지 파일 경로를 받은 경우
→ 파일 존재 확인
→ PIL로 읽기
→ 위 공통 함수 사용

OpenCV Camera Frame을 받은 경우
→ BGR → RGB
→ PIL Image
→ 위 공통 함수 사용
```


전처리가 다르면 같은 모델이어도 결과가 달라질 수 있습니다.

파일:

```text
src/ai_preprocess.py
```

코드:

```python
from pathlib import Path

import numpy as np
from PIL import Image


def pil_to_nchw_float32(
    image: Image.Image,
    image_size: int,
) -> np.ndarray:
    image = image.convert("RGB")

    image = image.resize(
        (image_size, image_size),
        Image.Resampling.BILINEAR,
    )

    array = np.asarray(
        image,
        dtype=np.float32,
    )

    array = array / 255.0

    array = np.transpose(
        array,
        (2, 0, 1),
    )

    array = np.expand_dims(
        array,
        axis=0,
    )

    return np.ascontiguousarray(
        array,
        dtype=np.float32,
    )


def image_path_to_nchw(
    path: str,
    image_size: int,
) -> np.ndarray:
    image_path = Path(path)

    if not image_path.exists():
        raise FileNotFoundError(
            f"이미지가 없습니다: "
            f"{image_path}"
        )

    image = Image.open(
        image_path
    )

    return pil_to_nchw_float32(
        image,
        image_size,
    )


def bgr_frame_to_nchw(
    frame,
    image_size: int,
) -> np.ndarray:
    # OpenCV Frame은 BGR입니다.
    # PIL / 학습 입력은 RGB를 사용합니다.
    rgb = frame[:, :, ::-1]

    image = Image.fromarray(
        rgb
    )

    return pil_to_nchw_float32(
        image,
        image_size,
    )
```

---

# 13. 전처리 결과 Shape 확인하기

## 이 파일은 왜 필요할까요?

전처리 코드를 작성했다고 해서 바로 ONNX 추론으로 넘어가지 않습니다. 먼저 **Shape·자료형·값 범위**를 출력하여 모델이 기대하는 입력 Contract와 맞는지 확인합니다.

## 의사코드

```text
--image 경로를 입력받는다
        ↓
settings.yaml에서 Metadata 경로를 읽는다
        ↓
Metadata의 image_size를 확인한다
        ↓
이미지를 공통 전처리 함수로 NCHW Tensor로 바꾼다
        ↓
Shape / dtype / min / max를 출력한다
        ↓
(1, 3, 96, 96), float32, 0~1 범위인지 확인한다
```


간단한 확인 파일을 만듭니다.

파일:

```text
scripts/check_ai_input.py
```

코드:

```python
import argparse
import json
from pathlib import Path

from src.ai_preprocess import (
    image_path_to_nchw,
)
from src.config_loader import load_config


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--image",
        required=True,
    )

    return parser.parse_args()


def main():
    args = parse_args()

    config = load_config(
        "configs/settings.yaml"
    )

    metadata = json.loads(
        Path(
            config["onnx_meta_path"]
        ).read_text(
            encoding="utf-8"
        )
    )

    tensor = image_path_to_nchw(
        args.image,
        int(metadata["image_size"]),
    )

    print("=== AI Input ===")
    print(f"Shape : {tensor.shape}")
    print(f"Dtype : {tensor.dtype}")
    print(f"Min   : {tensor.min():.4f}")
    print(f"Max   : {tensor.max():.4f}")


if __name__ == "__main__":
    main()
```

실행:

```bash
python -m scripts.check_ai_input \
  --image data/dataset/test/normal/파일명.jpg
```

정상:

```text
Shape : (1, 3, 96, 96)
Dtype : float32
Min   : 0....
Max   : 1....
```

---

# 14. ONNX Runtime 추론 모듈 만들기

## 이 파일은 왜 필요할까요?

ONNX Runtime을 직접 호출하는 코드를 모든 실행 파일에 반복하지 않기 위해 **모델 로드와 추론을 담당하는 하나의 Class**로 분리합니다.

이 모듈의 역할은 다음과 같습니다.

```text
ONNX Model + Metadata
        ↓
InferenceSession 생성
        ↓
Input / Output 이름 확인
        ↓
NCHW Tensor 입력
        ↓
Logit 출력
        ↓
Softmax
        ↓
Class / Confidence 반환
```

## 의사코드

```text
Model 경로와 Metadata 경로를 받는다
        ↓
두 파일이 실제로 있는지 확인한다
        ↓
Metadata JSON을 읽는다
        ↓
classes와 image_size를 저장한다
        ↓
CPUExecutionProvider로 InferenceSession을 만든다
        ↓
실제 Input / Output 이름을 Session에서 읽는다

predict() 호출
        ↓
입력 Tensor를 Session에 넣는다
        ↓
Logit을 받는다
        ↓
수치적으로 안정적인 Softmax를 계산한다
        ↓
가장 큰 Probability의 Index를 찾는다
        ↓
Metadata의 Class 순서로 Class 이름을 해석한다
        ↓
Class / Confidence / 전체 Probability / Logit을 반환한다
```


파일:

```text
src/inference_onnx.py
```

코드:

```python
import json
from pathlib import Path

import numpy as np
import onnxruntime as ort


def softmax(
    logits: np.ndarray,
) -> np.ndarray:
    shifted = (
        logits - np.max(
            logits,
            axis=1,
            keepdims=True,
        )
    )

    exp = np.exp(shifted)

    return (
        exp
        / np.sum(
            exp,
            axis=1,
            keepdims=True,
        )
    )


class ONNXClassifier:
    def __init__(
        self,
        model_path: str,
        metadata_path: str,
    ):
        self.model_path = Path(
            model_path
        )

        self.metadata_path = Path(
            metadata_path
        )

        if not self.model_path.exists():
            raise FileNotFoundError(
                f"ONNX Model이 없습니다: "
                f"{self.model_path}"
            )

        if not self.metadata_path.exists():
            raise FileNotFoundError(
                f"Metadata가 없습니다: "
                f"{self.metadata_path}"
            )

        self.metadata = json.loads(
            self.metadata_path.read_text(
                encoding="utf-8"
            )
        )

        self.classes = list(
            self.metadata["classes"]
        )

        self.image_size = int(
            self.metadata["image_size"]
        )

        self.session = (
            ort.InferenceSession(
                str(self.model_path),
                providers=[
                    "CPUExecutionProvider"
                ],
            )
        )

        self.input_name = (
            self.session
            .get_inputs()[0]
            .name
        )

        self.output_name = (
            self.session
            .get_outputs()[0]
            .name
        )

    def predict(
        self,
        input_tensor: np.ndarray,
    ) -> dict:
        outputs = self.session.run(
            [self.output_name],
            {
                self.input_name:
                    input_tensor
            },
        )

        logits = outputs[0]

        probabilities = softmax(
            logits
        )[0]

        index = int(
            np.argmax(probabilities)
        )

        return {
            "class_index": index,
            "class_name":
                self.classes[index],
            "confidence": float(
                probabilities[index]
            ),
            "probabilities":
                probabilities.tolist(),
            "logits":
                logits[0].tolist(),
        }
```

---

# 15. PC에서 ONNX 이미지 한 장 추론하기

## 이 파일은 왜 필요할까요?

Export가 성공했다는 사실과 **실제 추론이 성공한다는 사실은 다릅니다.** 이 프로그램은 저장 이미지 한 장을 ONNX Runtime에 넣어 Class와 Confidence를 확인하는 가장 작은 추론 테스트입니다.

## 의사코드

```text
--image 경로를 받는다
        ↓
Config에서 ONNX Model / Metadata 경로를 읽는다
        ↓
ONNXClassifier를 만든다
        ↓
이미지를 공통 전처리 함수로 바꾼다
        ↓
classifier.predict() 실행
        ↓
최종 Class / Confidence 출력
        ↓
모든 Class Probability 출력
```


파일:

```text
scripts/predict_onnx.py
```

코드:

```python
import argparse

from src.ai_preprocess import (
    image_path_to_nchw,
)
from src.config_loader import load_config
from src.inference_onnx import (
    ONNXClassifier,
)


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--image",
        required=True,
    )

    return parser.parse_args()


def main():
    args = parse_args()

    config = load_config(
        "configs/settings.yaml"
    )

    classifier = ONNXClassifier(
        model_path=
            config["onnx_model_path"],
        metadata_path=
            config["onnx_meta_path"],
    )

    input_tensor = image_path_to_nchw(
        args.image,
        classifier.image_size,
    )

    result = classifier.predict(
        input_tensor
    )

    print("=== ONNX Prediction ===")
    print(f"Image      : {args.image}")
    print(
        f"Class      : "
        f"{result['class_name']}"
    )
    print(
        f"Confidence : "
        f"{result['confidence']:.6f}"
    )

    print()
    print("All probabilities")

    for class_name, probability in zip(
        classifier.classes,
        result["probabilities"],
    ):
        print(
            f"{class_name:12s}: "
            f"{probability:.6f}"
        )


if __name__ == "__main__":
    main()
```

실행:

```bash
python -m scripts.predict_onnx \
  --image data/dataset/test/normal/파일명.jpg
```

`warning_red` 이미지도 실행합니다.

---

# 16. PyTorch와 ONNX를 같은 Tensor로 비교하기

## 이 파일은 왜 필요할까요?

PyTorch 프로그램과 ONNX 프로그램을 따로 실행해 결과만 눈으로 비교하면 전처리까지 같은지 확신하기 어렵습니다. 이 파일에서는 **하나의 NumPy NCHW 배열을 PyTorch와 ONNX에 동시에 넣어** 변환 전후 결과의 일치 여부를 확인합니다.

## 의사코드

```text
이미지 한 장을 공통 전처리한다
        ↓
같은 NCHW 배열을 만든다
        ↓
PyTorch TinyCNN에 같은 배열을 Tensor로 넣는다
        ↓
PyTorch Probability 계산
        ↓
ONNXClassifier에도 같은 배열을 넣는다
        ↓
ONNX Probability 계산
        ↓
두 예측 Class를 비교한다
        ↓
Class Match를 출력한다
        ↓
Class별 Probability 차이 중 최대값을 계산한다
        ↓
Max Prob Diff를 출력한다
```


단순히 두 프로그램을 따로 실행하지 않고 **완전히 같은 전처리 Tensor**를 넣어 비교합니다.

파일:

```text
scripts/compare_pytorch_onnx.py
```

코드:

```python
import argparse
from pathlib import Path

import numpy as np
import torch

from src.ai_preprocess import (
    image_path_to_nchw,
)
from src.config_loader import load_config
from src.inference_onnx import (
    ONNXClassifier,
)
from src.tiny_cnn import TinyCNN


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--image",
        required=True,
    )

    return parser.parse_args()


def torch_softmax(
    logits: torch.Tensor,
):
    return torch.softmax(
        logits,
        dim=1,
    )


def main():
    args = parse_args()

    config = load_config(
        "configs/settings.yaml"
    )

    checkpoint = torch.load(
        Path(config["ai_model_path"]),
        map_location="cpu",
    )

    classes = list(
        checkpoint["classes"]
    )

    image_size = int(
        checkpoint["image_size"]
    )

    model = TinyCNN(
        num_classes=len(classes)
    )

    model.load_state_dict(
        checkpoint["model_state"]
    )

    model.eval()

    input_array = image_path_to_nchw(
        args.image,
        image_size,
    )

    torch_input = torch.from_numpy(
        input_array
    )

    with torch.no_grad():
        torch_logits = model(
            torch_input
        )

        torch_probs = torch_softmax(
            torch_logits
        ).numpy()[0]

    classifier = ONNXClassifier(
        model_path=
            config["onnx_model_path"],
        metadata_path=
            config["onnx_meta_path"],
    )

    onnx_result = classifier.predict(
        input_array
    )

    onnx_probs = np.asarray(
        onnx_result["probabilities"],
        dtype=np.float32,
    )

    torch_index = int(
        np.argmax(torch_probs)
    )

    onnx_index = int(
        np.argmax(onnx_probs)
    )

    max_diff = float(
        np.max(
            np.abs(
                torch_probs
                - onnx_probs
            )
        )
    )

    print("=== PyTorch vs ONNX ===")
    print(f"Image: {args.image}")
    print()

    print(
        "PyTorch : "
        f"{classes[torch_index]} "
        f"{torch_probs[torch_index]:.6f}"
    )

    print(
        "ONNX    : "
        f"{classes[onnx_index]} "
        f"{onnx_probs[onnx_index]:.6f}"
    )

    print()
    print(
        f"Class Match : "
        f"{torch_index == onnx_index}"
    )

    print(
        f"Max Prob Diff : "
        f"{max_diff:.8f}"
    )


if __name__ == "__main__":
    main()
```

---

# 17. Result Parity 확인하기

## 코드 리뷰 — `Class Match`와 `Max Prob Diff` 역추적하기

다음 결과가 나왔다고 가정합니다.

```text
Class Match : True
Max Prob Diff : 0.00000012
```

직접 다음 연결을 찾습니다.

```text
같은 이미지
        ↓
image_path_to_nchw()
        ↓
input_array 하나 생성
        ├─ torch.from_numpy()
        │      ↓
        │   TinyCNN
        │
        └─ ONNXClassifier.predict()
               ↓
두 Probability 배열
        ↓
np.argmax()
→ Class Index 비교
        ↓
Class Match

np.abs(torch_probs - onnx_probs)
        ↓
np.max()
        ↓
Max Prob Diff
```

핵심은 **같은 입력 배열**을 두 Runtime에 넣었다는 점입니다.


normal 이미지:

```bash
python -m scripts.compare_pytorch_onnx \
  --image data/dataset/test/normal/파일명.jpg
```

warning_red 이미지:

```bash
python -m scripts.compare_pytorch_onnx \
  --image data/dataset/test/warning_red/파일명.jpg
```

확인:

```text
Class Match
→ True인가?

Confidence
→ 크게 다르지 않은가?

Max Prob Diff
→ 작은 값인가?
```

ONNX와 PyTorch의 부동소수점 계산은 아주 미세하게 다를 수 있습니다.

중요한 것은 **같은 입력에서 예측 Class가 일치하고 Probability가 크게 어긋나지 않는지**입니다.

---

# 18. ONNX의 실제 Input / Output 정보 확인하기

## 이 파일은 왜 필요할까요?

Metadata에 적어 둔 Shape와 실제 ONNX Runtime이 읽은 Shape가 같은지 확인해야 합니다. `inspect_onnx_runtime.py`는 **모델 파일 자체가 가진 Input/Output Contract**를 직접 보여주는 점검 프로그램입니다.

## 의사코드

```text
settings.yaml을 읽는다
        ↓
ONNX Runtime Session을 만든다
        ↓
모든 Input의 Name / Shape / Type을 출력한다
        ↓
모든 Output의 Name / Shape / Type을 출력한다
        ↓
현재 사용 가능한 Execution Provider를 출력한다
```


파일:

```text
scripts/inspect_onnx_runtime.py
```

코드:

```python
import onnxruntime as ort

from src.config_loader import load_config


def main():
    config = load_config(
        "configs/settings.yaml"
    )

    session = ort.InferenceSession(
        config["onnx_model_path"],
        providers=[
            "CPUExecutionProvider"
        ],
    )

    print("=== ONNX Runtime Inspect ===")

    for item in session.get_inputs():
        print()
        print("[Input]")
        print(f"Name  : {item.name}")
        print(f"Shape : {item.shape}")
        print(f"Type  : {item.type}")

    for item in session.get_outputs():
        print()
        print("[Output]")
        print(f"Name  : {item.name}")
        print(f"Shape : {item.shape}")
        print(f"Type  : {item.type}")

    print()
    print("Providers:")
    print(session.get_providers())


if __name__ == "__main__":
    main()
```

실행:

```bash
python -m scripts.inspect_onnx_runtime
```

예:

```text
Input
Name  : input
Shape : [1, 3, 96, 96]
Type  : tensor(float)

Output
Name  : logits
Shape : [1, 2]
Type  : tensor(float)
```

---

# 19. 배포 전 네 가지 Checkpoint 확인하기

## 1. Input Shape

```text
[1, 3, 96, 96]
```

확인했습니다.

## 2. Preprocess

```text
RGB
→ Resize 96 × 96
→ float32
→ /255
→ HWC → CHW
→ Batch 추가
```

확인했습니다.

## 3. Output

```text
[1, 2]
→ normal
→ warning_red
```

확인했습니다.

## 4. Result Parity

```text
PyTorch vs ONNX
→ Class 일치
→ Probability 크게 어긋나지 않음
```

확인했습니다.

네 가지가 모두 통과하면 Raspberry Pi로 모델을 보냅니다.

---

# 20. PC에서 ONNX 배포 Report 시작하기

파일:

```text
reports/day07_onnx_deployment.md
```

먼저 다음 부분을 작성합니다.

```md
# Day 07 ONNX Deployment

## Export

PyTorch Model:

ONNX Model:

Opset:

Input Shape:

Output Shape:

Classes:

## Preprocess

Color Order:

Resize:

Dtype:

Value Range:

Layout:

## PyTorch vs ONNX

### Normal Sample

PyTorch Class:

PyTorch Confidence:

ONNX Class:

ONNX Confidence:

Class Match:

Max Probability Difference:

### Warning Sample

PyTorch Class:

PyTorch Confidence:

ONNX Class:

ONNX Confidence:

Class Match:

Max Probability Difference:
```

---

# PART C. ONNX Artifact를 Raspberry Pi로 배포하기


# 21. 배포용 테스트 이미지 하나를 고정하기

PC와 Raspberry Pi에서 **완전히 같은 이미지**로 비교하기 위해 파일 하나를 고정합니다.

폴더:

```bash
mkdir -p data/deploy_samples
```

normal Test 이미지 하나를 복사합니다.

```bash
cp \
data/dataset/test/normal/파일명.jpg \
data/deploy_samples/normal.jpg
```

warning_red도 복사합니다.

```bash
cp \
data/dataset/test/warning_red/파일명.jpg \
data/deploy_samples/warning_red.jpg
```

Hash 확인:

```bash
sha256sum \
  data/deploy_samples/normal.jpg \
  data/deploy_samples/warning_red.jpg
```

이 Hash 값을 Report에 기록해도 좋습니다.

---

# 22. Raspberry Pi로 ONNX 파일과 Metadata 전송하기

Raspberry Pi가 켜져 있는지 확인합니다.

PC에서 모델 폴더를 Raspberry Pi에 준비합니다.

```bash
ssh <RPI_USER>@<RPI_IP> \
"mkdir -p ~/ai_vision/subject13_edge_ai/models"
```

ONNX:

```bash
scp \
models/day07_tiny_cnn.onnx \
<RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/models/
```

Metadata:

```bash
scp \
models/day07_tiny_cnn.meta.json \
<RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/models/
```

동일 입력 이미지도 보냅니다.

```bash
ssh <RPI_USER>@<RPI_IP> \
"mkdir -p ~/ai_vision/subject13_edge_ai/data/deploy_samples"
```

```bash
scp \
data/deploy_samples/normal.jpg \
data/deploy_samples/warning_red.jpg \
<RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/data/deploy_samples/
```

---

# 23. Raspberry Pi에서 파일이 정확히 전달되었는지 확인하기

Raspberry Pi에 접속합니다.

```bash
ssh <RPI_USER>@<RPI_IP>
```

프로젝트:

```bash
cd ~/ai_vision/subject13_edge_ai
```

확인:

```bash
ls -lh models
```

```bash
ls -lh data/deploy_samples
```

Hash:

```bash
sha256sum \
  data/deploy_samples/normal.jpg \
  data/deploy_samples/warning_red.jpg
```

PC에서 본 Hash와 같아야 합니다.

같은 이미지로 비교하기 위한 확인입니다.

---

# 24. Raspberry Pi에 7일차 Source를 전달하기

ONNX Model과 Test Image만 전송되어도 **새로 만든 Python Source가 Raspberry Pi에 없으면 실행할 수 없습니다.**

Source 전달 방식은 교육장 환경에 따라 두 가지로 나눕니다.

### 방법 A — 허용된 내부 GitLab / Gitea Remote를 사용하는 경우

PC에서 먼저 7일차 Source를 Commit하고 Push한 뒤 Raspberry Pi에서:

```bash
cd ~/ai_vision/subject13_edge_ai
git status
git pull
git log --oneline -8
```

를 실행합니다.

### 방법 B — Raspberry Pi에는 SCP로 Source를 전달하는 경우

PC 프로젝트 루트에서 필요한 파일을 직접 전송합니다.

먼저 폴더가 있는지 확인합니다.

```bash
ssh <RPI_USER>@<RPI_IP> \
"mkdir -p ~/ai_vision/subject13_edge_ai/src ~/ai_vision/subject13_edge_ai/scripts ~/ai_vision/subject13_edge_ai/configs"
```

핵심 Source:

```bash
scp \
  src/ai_preprocess.py \
  src/inference_onnx.py \
  src/decision.py \
  src/ai_warning.py \
  src/event_logger.py \
  <RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/src/
```

실행 Program:

```bash
scp \
  scripts/check_ai_input.py \
  scripts/predict_onnx.py \
  scripts/inspect_onnx_runtime.py \
  scripts/onnx_decision_test.py \
  scripts/onnx_camera_probe.py \
  scripts/ai_warning_system.py \
  <RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/scripts/
```

오늘 공통 설정도 전달합니다.

```bash
scp \
  configs/settings.yaml \
  <RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/configs/
```

`configs/device.local.yaml`은 Raspberry Pi 장치 전용 값이므로 PC 파일로 덮어쓰지 않습니다.

```text
Source
→ 내부 Git 또는 SCP

ONNX Model
→ SCP 또는 허용된 Model 저장소

Metadata
→ Model과 함께 SCP
→ 작은 Contract 파일이므로 Git에도 보존 가능

Test Image
→ SCP

device.local.yaml
→ Raspberry Pi Local 유지
```

이 구분을 기억합니다.

---

# 25. Raspberry Pi 가상환경 활성화

```bash
source .venv/bin/activate
```

확인:

```bash
which python
```

Architecture:

```bash
uname -m
```

예:

```text
aarch64
```

---

# 26. Raspberry Pi에서 ONNX Runtime 준비하기

Raspberry Pi의 기존 `.venv`를 활성화한 상태에서 먼저 현재 설치 여부를 확인합니다.

```bash
python -c \
"import onnxruntime as ort; print(ort.__version__); print(ort.get_available_providers())"
```

정상이라면 다시 설치하지 않습니다.

### Package가 없는 경우

인터넷 사용이 허용되고 현재 Raspberry Pi OS / Python / Architecture와 호환되는 Wheel이 제공되는 환경:

```bash
python -m pip install \
  onnxruntime \
  Pillow \
  numpy
```

폐쇄망 또는 외부 설치 제한 환경에서는 **사전에 검증된 Raspberry Pi용 Wheel 또는 내부 Package 저장소**를 사용합니다.

설치 전에는 다음 정보를 확인해 둡니다.

```bash
uname -m
python --version
```

예를 들어 `aarch64` 장치에 다른 Architecture용 Wheel을 설치하면 안 됩니다.

설치 후 확인합니다.

```bash
python -c \
"import onnxruntime as ort; print('ORT:', ort.__version__); print('Providers:', ort.get_available_providers())"
```

CPU 기반 실습에서는 다음 Provider가 포함되는지 확인합니다.

```text
CPUExecutionProvider
```

> `onnxruntime` 설치 실패는 Python 코드 오류와 다를 수 있습니다. OS·Python 버전·CPU Architecture와 Wheel 호환성을 먼저 확인합니다.

---

# 27. Raspberry Pi 환경 Package 기록하기

Raspberry Pi에서:

```bash
python -m pip freeze \
  > reports/day07_pi_environment.txt
```

확인:

```bash
head reports/day07_pi_environment.txt
```

PC 환경과 Raspberry Pi 환경은 다를 수 있습니다.

```text
PC
→ Training 환경

Pi
→ Inference 환경
```

둘이 완전히 같은 Package 목록일 필요는 없습니다.

---

# 28. Raspberry Pi에서 ONNX Runtime 구조 확인하기

실행:

```bash
python -m scripts.inspect_onnx_runtime
```

다음을 확인합니다.

```text
Input Shape
[1, 3, 96, 96]

Output Shape
[1, 2]

Provider
CPUExecutionProvider
```

PC에서 확인한 구조와 같은지 봅니다.

---

# 29. Raspberry Pi에서 동일 normal 이미지 추론하기

```bash
python -m scripts.predict_onnx \
  --image data/deploy_samples/normal.jpg
```

결과를 기록합니다.

```text
Class:
Confidence:
```

PC ONNX 결과와 비교합니다.

---

# 30. Raspberry Pi에서 동일 warning_red 이미지 추론하기

```bash
python -m scripts.predict_onnx \
  --image data/deploy_samples/warning_red.jpg
```

결과:

```text
Class:
Confidence:
```

PC와 같은 이미지이므로 Class가 일치하는지 확인합니다.

---

# 31. PC ONNX와 Raspberry Pi ONNX 결과 비교하기

## 결과를 코드와 배포 파일에서 다시 찾아보기

PC와 Pi에서 Class가 같다고 해서 막연히 “배포 성공”이라고 하지 않습니다.

다음 네 가지가 연결되었는지 확인합니다.

```text
같은 ONNX 파일인가?
→ models/day07_tiny_cnn.onnx

같은 Metadata인가?
→ models/day07_tiny_cnn.meta.json

같은 입력 이미지인가?
→ SHA-256 비교

같은 전처리 Source인가?
→ src/ai_preprocess.py
```

이 네 조건을 맞춘 뒤 결과를 비교해야 Runtime 환경 차이를 더 공정하게 볼 수 있습니다.


Report에 다음 표를 추가합니다.

```md
## PC ONNX vs Raspberry Pi ONNX

| Image | PC Class | PC Confidence | Pi Class | Pi Confidence | Match |
|---|---|---:|---|---:|---|
| normal.jpg |  |  |  |  |  |
| warning_red.jpg |  |  |  |  |  |
```

동일 모델과 동일 입력이라도 실행환경 차이로 아주 작은 부동소수점 차이는 생길 수 있습니다.

다음이 중요합니다.

```text
Class가 같은가?

Confidence가 크게 어긋나지 않는가?
```

---

# PART D. Raspberry Pi Camera AI Warning System 완성하기


# 32. AI Prediction과 운영 Decision을 분리하기

AI 결과는 다음처럼 나옵니다.

```text
Prediction

Class
warning_red

Confidence
0.82
```

하지만 실제 장치를 WARNING으로 만들지는 별도의 운영 기준이 결정합니다.

예:

```text
warning_red
+
Confidence >= 0.70
        ↓
WARNING
```

반면:

```text
warning_red
+
Confidence = 0.58
+
Threshold = 0.70
        ↓
NORMAL
```

오늘은 이 두 단계를 코드에서도 분리합니다.

---

# 33. 운영 Decision 모듈 만들기

## 이 파일은 왜 필요할까요?

AI가 `warning_red 0.82`를 출력하는 것은 **AI 예측**입니다. 실제 장치가 Red LED를 켤지는 운영 규칙이 결정합니다.

이 두 역할을 분리하면 모델을 바꾸지 않고도 운영 Threshold를 바꾸어 실험할 수 있습니다.

> 이 수업의 2-Class Baseline에서는 `warning_red`이면서 Confidence가 기준 이상일 때만 `WARNING`, 나머지는 `NORMAL`로 단순화합니다. 실제 산업 안전 시스템에서는 낮은 Confidence를 별도 `UNKNOWN/REVIEW` 상태로 처리하는 등 별도 정책이 필요할 수 있습니다. 오늘은 기존 `NORMAL/WARNING` 구조를 유지합니다.

## 의사코드

```text
예측 Class를 받는다
Confidence를 받는다
경고 Class 이름을 받는다
운영 Confidence Threshold를 받는다
        ↓
예측 Class가 warning_class와 같은가?
        ↓
Confidence가 Threshold 이상인가?
        ↓
두 조건이 모두 True
→ WARNING
        ↓
그 외
→ NORMAL
```


파일:

```text
src/decision.py
```

코드:

```python
def ai_to_operational_state(
    predicted_class: str,
    confidence: float,
    warning_class: str,
    confidence_threshold: float,
) -> str:
    is_warning_class = (
        predicted_class
        == warning_class
    )

    confidence_ok = (
        confidence
        >= confidence_threshold
    )

    if (
        is_warning_class
        and confidence_ok
    ):
        return "WARNING"

    return "NORMAL"
```

이 함수는 AI 모델을 실행하지 않습니다.

다음 정보만 받아 운영 상태를 결정합니다.

```text
predicted_class
confidence
warning_class
confidence_threshold
```

---

# 34. 저장 이미지로 운영 Decision 확인하기

## 이 파일은 왜 필요할까요?

Camera와 GPIO를 다시 붙이기 전에 저장 이미지 한 장으로 **Prediction과 Operational Decision을 분리해서 확인**합니다. `--threshold` 인자를 사용하면 Python 코드를 수정하지 않고 기준만 바꾸어 비교할 수 있습니다.

## 의사코드

```text
--image와 선택적 --threshold를 받는다
        ↓
Config를 읽는다
        ↓
명령행 Threshold가 있으면 그것을 사용한다
없으면 settings.yaml의 기본값을 사용한다
        ↓
ONNX 추론으로 Class / Confidence를 얻는다
        ↓
decision.py에 Class / Confidence / 기준을 전달한다
        ↓
Prediction / Confidence / Threshold / State를 출력한다
```


파일:

```text
scripts/onnx_decision_test.py
```

코드:

```python
import argparse

from src.ai_preprocess import (
    image_path_to_nchw,
)
from src.config_loader import load_config
from src.decision import (
    ai_to_operational_state,
)
from src.inference_onnx import (
    ONNXClassifier,
)


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--image",
        required=True,
    )

    parser.add_argument(
        "--threshold",
        type=float,
        default=None,
    )

    return parser.parse_args()


def main():
    args = parse_args()

    config = load_config(
        "configs/settings.yaml"
    )

    threshold = (
        args.threshold
        if args.threshold is not None
        else float(
            config[
                "ai_confidence_threshold"
            ]
        )
    )

    classifier = ONNXClassifier(
        config["onnx_model_path"],
        config["onnx_meta_path"],
    )

    input_tensor = image_path_to_nchw(
        args.image,
        classifier.image_size,
    )

    prediction = classifier.predict(
        input_tensor
    )

    state = ai_to_operational_state(
        predicted_class=
            prediction["class_name"],
        confidence=
            prediction["confidence"],
        warning_class=
            config["ai_warning_class"],
        confidence_threshold=
            threshold,
    )

    print("=== AI Decision ===")
    print(
        f"Prediction : "
        f"{prediction['class_name']}"
    )
    print(
        f"Confidence : "
        f"{prediction['confidence']:.4f}"
    )
    print(
        f"Threshold  : "
        f"{threshold:.2f}"
    )
    print(f"State      : {state}")


if __name__ == "__main__":
    main()
```

---

# 35. Confidence Threshold 0.50 / 0.70 / 0.90 비교하기

같은 warning 이미지에 세 Threshold를 적용합니다.

```bash
python -m scripts.onnx_decision_test \
  --image data/deploy_samples/warning_red.jpg \
  --threshold 0.50
```

```bash
python -m scripts.onnx_decision_test \
  --image data/deploy_samples/warning_red.jpg \
  --threshold 0.70
```

```bash
python -m scripts.onnx_decision_test \
  --image data/deploy_samples/warning_red.jpg \
  --threshold 0.90
```

예를 들어 Confidence가 `0.82`라면:

```text
Threshold 0.50
→ WARNING

Threshold 0.70
→ WARNING

Threshold 0.90
→ NORMAL
```

AI Prediction은 동일하지만 **운영 Decision이 달라질 수 있습니다.**

---

# 36. Threshold 비교표 작성하기

Report에 추가합니다.

```md
## Confidence Threshold Experiment

Image:

Prediction Class:

Prediction Confidence:

| Threshold | Operational State | 관찰 |
|---:|---|---|
| 0.50 |  |  |
| 0.70 |  |  |
| 0.90 |  |  |

Prediction과 Decision의 차이:
```

---

# 37. Camera Frame을 ONNX 입력으로 바꾸기

저장 이미지 추론이 정상이라면 실시간 Camera로 넘어갑니다.

3일차의:

```text
CameraInput
```

을 그대로 사용합니다.

전처리는:

```text
bgr_frame_to_nchw()
```

를 사용합니다.

구조:

```text
USB Camera
→ BGR Frame
→ BGR → RGB
→ Resize
→ float32 / 255
→ HWC → CHW
→ Batch
→ ONNX
```

---

# 38. Raspberry Pi Camera ONNX Probe 만들기

## 이 파일은 왜 필요할까요?

저장 이미지에서 ONNX 추론과 운영 Decision이 정상인 것을 확인했으므로 이제 **실제 Camera의 연속 Frame**에 같은 구조를 적용합니다. 아직 GPIO와 Event Log는 붙이지 않고 Camera → AI → Decision까지만 확인합니다.

## 의사코드

```text
Config를 읽는다
        ↓
기존 CameraInput으로 Camera를 연다
        ↓
ONNXClassifier를 만든다
        ↓
Confidence Threshold와 warning_class를 읽는다
        ↓
반복 시작
        ↓
Frame 한 장 읽기
        ↓
BGR Frame을 NCHW float32로 전처리
        ↓
ONNX 추론
        ↓
Class + Confidence
        ↓
운영 Decision
        ↓
Class / Confidence / State 출력
        ↓
Ctrl+C
        ↓
Camera Resource 정리
```


파일:

```text
scripts/onnx_camera_probe.py
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


def main():
    config = load_config(
        "configs/settings.yaml"
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

    try:
        print("=== ONNX Camera Probe ===")
        print("종료: Ctrl+C")
        print()

        time.sleep(1.0)

        while True:
            frame = camera.read()

            input_tensor = (
                bgr_frame_to_nchw(
                    frame,
                    classifier.image_size,
                )
            )

            prediction = (
                classifier.predict(
                    input_tensor
                )
            )

            state = (
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

            print(
                f"class="
                f"{prediction['class_name']:12s} "
                f"conf="
                f"{prediction['confidence']:.3f} "
                f"state={state}"
            )

    except KeyboardInterrupt:
        print("\n종료합니다.")

    finally:
        camera.release()


if __name__ == "__main__":
    main()
```

---

# 39. Camera ONNX 추론 실행하기

```bash
python -m scripts.onnx_camera_probe
```

Camera 앞에 아무 경고 대상이 없을 때:

```text
normal
```

이 나오는지 확인합니다.

빨간색 카드를 보여줍니다.

```text
warning_red
```

가 나오는지 확인합니다.

Class뿐 아니라 Confidence도 계속 확인합니다.

예:

```text
class=normal       conf=0.942 state=NORMAL
class=warning_red  conf=0.863 state=WARNING
```

---

# 40. 오늘은 FPS를 정식으로 측정하지 않습니다

Camera 추론을 보면 빠르거나 느리다는 느낌은 받을 수 있습니다.

하지만 오늘은 다음 숫자를 성능 결론으로 사용하지 않습니다.

```text
한 번 실행한 시간
눈으로 본 속도
대충 계산한 FPS
```

8일차에서:

```text
Warm-up
반복 횟수
Mean
P50
P95
End-to-End FPS
```

조건을 맞춰 정식으로 측정합니다.

오늘은 **실시간 Camera에서 ONNX 추론 자체가 안정적으로 실행되는지**만 확인합니다.

---

# 41. AI Event Logger 만들기

## 이 Class는 왜 추가할까요?

5일차의 Event Logger는 Rule의 `red_area`를 기록했습니다. 7일차에는 AI가 만든 **예측 Class와 Confidence**를 기록해야 하므로 같은 `src/event_logger.py`에 AI용 Logger를 추가합니다.

기존 Logger를 삭제하지 않습니다. Rule과 AI의 기록 형식이 서로 다르기 때문에 각각의 Class를 유지합니다.

## 의사코드

```text
CSV 경로를 받는다
        ↓
부모 폴더가 없으면 만든다
        ↓
append() 호출
        ↓
새 파일이면 Header 기록
        ↓
timestamp / mode=ai / pred_class
confidence / threshold / state / image_path
를 한 줄 저장
```


5일차 Rule Event Log 구조를 참고하여 AI 결과용 Logger를 추가합니다.

`src/event_logger.py` 아래에 다음 Class를 추가합니다.

```python
class AIEventLogger:
    def __init__(self, path: str):
        self.path = Path(path)

        self.path.parent.mkdir(
            parents=True,
            exist_ok=True,
        )

    def append(
        self,
        timestamp: str,
        pred_class: str,
        confidence: float,
        threshold: float,
        state: str,
        image_path: str = "",
    ) -> None:
        is_new = not self.path.exists()

        with self.path.open(
            "a",
            newline="",
            encoding="utf-8",
        ) as file:
            writer = csv.writer(file)

            if is_new:
                writer.writerow(
                    [
                        "timestamp",
                        "mode",
                        "pred_class",
                        "confidence",
                        "threshold",
                        "state",
                        "image_path",
                    ]
                )

            writer.writerow(
                [
                    timestamp,
                    "ai",
                    pred_class,
                    f"{confidence:.6f}",
                    f"{threshold:.3f}",
                    state,
                    image_path,
                ]
            )
```

`event_logger.py`의 상단에 이미 다음 Import가 있다면 다시 추가하지 않습니다.

```python
import csv
from pathlib import Path
```

---

# 42. AI WARNING 이미지 저장 함수 만들기

## 이 파일은 왜 필요할까요?

WARNING이 발생했을 때 터미널과 CSV만 남기면 당시 Camera 장면을 다시 확인하기 어렵습니다. 상태가 `WARNING`으로 **전환되는 순간의 대표 Frame**을 한 장 저장하여 판단 근거를 남깁니다.

## 의사코드

```text
Frame / 저장 폴더 / Class / Confidence를 받는다
        ↓
저장 폴더가 없으면 만든다
        ↓
현재 시각을 파일명용 문자열로 만든다
        ↓
Class 이름에서 경로에 위험한 '/'를 '_'로 바꾼다
        ↓
Timestamp + Class + Confidence로 파일명을 만든다
        ↓
cv2.imwrite()로 저장한다
        ↓
저장 실패면 오류
        ↓
저장 경로 반환
```


파일:

```text
src/ai_warning.py
```

코드:

```python
from datetime import datetime
from pathlib import Path

import cv2


def save_ai_warning_image(
    frame,
    output_dir: str,
    class_name: str,
    confidence: float,
) -> str:
    directory = Path(output_dir)

    directory.mkdir(
        parents=True,
        exist_ok=True,
    )

    timestamp = datetime.now().strftime(
        "%Y%m%d_%H%M%S_%f"
    )

    safe_class = class_name.replace(
        "/",
        "_",
    )

    path = directory / (
        f"warning_{timestamp}_"
        f"{safe_class}_"
        f"{confidence:.3f}.jpg"
    )

    ok = cv2.imwrite(
        str(path),
        frame,
    )

    if not ok:
        raise RuntimeError(
            f"AI WARNING 이미지 저장 실패: "
            f"{path}"
        )

    return str(path)
```

---

# 43. AI Warning System 만들기

## 이 파일은 왜 필요할까요?

지금까지 검증한 기능을 하나의 **실제 Edge AI 실행 시작점**으로 연결합니다.

```text
Camera Input
→ AI Preprocess
→ ONNX Runtime
→ Class + Confidence
→ Operational Decision
→ GPIO
→ Event Log
→ WARNING 전환 시 대표 이미지
```

새로운 알고리즘을 다시 만드는 것이 아니라 3일차 Camera/GPIO, 5일차 Event 개념, 6일차 Model, 오늘 만든 ONNX 모듈을 조합합니다.

## 실행 전에 파일 연결 보기

```text
configs/settings.yaml
configs/device.local.yaml
        ↓
src/config_loader.py
        ↓
scripts/ai_warning_system.py
        │
        ├─ src/camera_input.py
        ├─ src/ai_preprocess.py
        ├─ src/inference_onnx.py
        ├─ src/decision.py
        ├─ src/gpio_output.py
        ├─ src/event_logger.py
        └─ src/ai_warning.py
```

## 의사코드

```text
Config를 읽는다
        ↓
Camera / ONNX Classifier / GPIO / Logger를 준비한다
        ↓
Threshold와 warning_class를 읽는다
        ↓
previous_state = None
        ↓
반복
        ↓
Frame 읽기
→ 전처리
→ ONNX 추론
→ 운영 State 결정
        ↓
State에 따라 GPIO 출력
        ↓
이전 State와 달라졌는지 확인
        ↓
상태가 바뀌었으면 Event 기록
        ↓
새 State가 WARNING이면 대표 이미지 1장 저장
        ↓
현재 Class / Confidence / State / changed 출력
        ↓
Ctrl+C
        ↓
Camera와 GPIO Resource 정리
```

> `use_buzzer`는 장비 설정을 그대로 따릅니다. Buzzer를 사용하지 않거나 사양 확인이 끝나지 않았다면 `use_buzzer: false`를 유지합니다.


파일:

```text
scripts/ai_warning_system.py
```

코드:

```python
import time
from datetime import datetime

from src.ai_preprocess import (
    bgr_frame_to_nchw,
)
from src.ai_warning import (
    save_ai_warning_image,
)
from src.camera_input import CameraInput
from src.config_loader import load_config
from src.decision import (
    ai_to_operational_state,
)
from src.event_logger import (
    AIEventLogger,
)
from src.gpio_output import GPIOOutput
from src.inference_onnx import (
    ONNXClassifier,
)


def main():
    config = load_config(
        "configs/settings.yaml"
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
        use_buzzer=bool(config.get("use_buzzer", False)),
    )

    logger = AIEventLogger(
        config["ai_event_log_path"]
    )

    threshold = float(
        config["ai_confidence_threshold"]
    )

    warning_class = (
        config["ai_warning_class"]
    )

    previous_state = None

    try:
        print(
            "=== AI Edge Warning System ==="
        )
        print("종료: Ctrl+C")
        print()

        time.sleep(1.0)

        while True:
            frame = camera.read()

            input_tensor = (
                bgr_frame_to_nchw(
                    frame,
                    classifier.image_size,
                )
            )

            prediction = (
                classifier.predict(
                    input_tensor
                )
            )

            state = (
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

            if state == "WARNING":
                output.warning()
            else:
                output.normal()

            changed = (
                state != previous_state
            )

            if changed:
                timestamp = (
                    datetime.now().isoformat(
                        timespec="milliseconds"
                    )
                )

                image_path = ""

                if state == "WARNING":
                    image_path = (
                        save_ai_warning_image(
                            frame,
                            config[
                                "ai_warning_image_dir"
                            ],
                            prediction[
                                "class_name"
                            ],
                            prediction[
                                "confidence"
                            ],
                        )
                    )

                logger.append(
                    timestamp=timestamp,
                    pred_class=
                        prediction[
                            "class_name"
                        ],
                    confidence=
                        prediction[
                            "confidence"
                        ],
                    threshold=threshold,
                    state=state,
                    image_path=image_path,
                )

                previous_state = state

            print(
                f"class="
                f"{prediction['class_name']:12s} "
                f"conf="
                f"{prediction['confidence']:.3f} "
                f"state={state:7s} "
                f"changed={changed}"
            )

    except KeyboardInterrupt:
        print("\n종료합니다.")

    finally:
        camera.release()
        output.close()


if __name__ == "__main__":
    main()
```

---

# 44. Raspberry Pi에서 AI Warning System 실행하기

```bash
python -m scripts.ai_warning_system
```

기본 상태:

```text
normal
→ NORMAL
→ Green LED
```

빨간색 카드:

```text
warning_red
+
Confidence >= Threshold
→ WARNING
→ Red LED
→ 선택 Buzzer (`use_buzzer: true`인 경우)
→ Event Log
→ Warning Image
```

종료:

```text
Ctrl+C
```

---

# 45. AI Event Log 확인하기

## 코드 리뷰 — CSV 한 줄은 어떻게 만들어졌을까요?

WARNING Event 한 줄을 골라 역추적합니다.

```text
Camera Frame
        ↓
ONNXClassifier.predict()
        ↓
pred_class / confidence
        ↓
ai_to_operational_state()
        ↓
WARNING
        ↓
state != previous_state
        ↓
save_ai_warning_image()
        ↓
image_path
        ↓
AIEventLogger.append()
        ↓
logs/day07_ai_events.csv
```

CSV의 `pred_class`, `confidence`, `threshold`, `state`, `image_path`가 각각 어느 변수에서 왔는지 `ai_warning_system.py`에서 직접 찾아봅니다.


```bash
cat logs/day07_ai_events.csv
```

예:

```csv
timestamp,mode,pred_class,confidence,threshold,state,image_path
2026-10-09T14:10:01.125,ai,normal,0.944201,0.700,NORMAL,
2026-10-09T14:10:05.642,ai,warning_red,0.862314,0.700,WARNING,data/ai_warning_images/...
2026-10-09T14:10:09.981,ai,normal,0.913208,0.700,NORMAL,
```

이제 Log에는 5일차와 다른 정보가 들어 있습니다.

```text
Rule

red_area
threshold
state
```

vs

```text
AI

pred_class
confidence
threshold
state
```

---

# 46. Rule과 AI의 판단 위치 비교하기

5일차:

```text
Camera
→ HSV
→ Red Area
→ Area Threshold
→ State
```

7일차:

```text
Camera
→ AI Preprocess
→ ONNX
→ Class + Confidence
→ Confidence Threshold
→ State
```

달라진 부분은 판단 모듈입니다.

```text
입력 Camera
→ 그대로

GPIO
→ 그대로

Event Log
→ 구조 확장

판단
→ Rule에서 AI로 교체
```

---

# PART E. Mini Challenge — 배포 결과를 읽고 수정하고 복구하기


# 47. Mini Challenge — ONNX 배포 흐름을 스스로 다시 이해하기

지금까지는 안내된 순서대로 다음 흐름을 완성했습니다.

```text
PyTorch
→ ONNX
→ Preprocess
→ PC ONNX
→ Result Parity
→ Raspberry Pi
→ ONNX Runtime
→ Camera
→ Decision
→ GPIO
→ Event Log
```

이제부터는 새로운 기술을 더 배우는 시간이 아니라 **오늘 만든 Pipeline을 스스로 읽고 수정하고 복구하는 시간**입니다.

Mini Challenge에서는 다음 순서를 반복합니다.

```text
결과를 본다
        ↓
결과가 만들어진 코드 위치를 찾는다
        ↓
실행 전에 결과를 예측한다
        ↓
설정값 하나만 바꾼다
        ↓
기존 기능을 조합해 기능 하나를 완성한다
        ↓
일부러 오류를 만들고 원인을 찾는다
        ↓
Log / Hash / 저장 이미지로 정상 동작을 증명한다
        ↓
전체 Pipeline을 자신의 말로 설명한다
```

각 Challenge는 먼저 직접 수행한 뒤 **예시 정답 확인해보기**를 엽니다.

---

# 48. Challenge 1 — ONNX 결과 한 줄을 코드로 역추적하기

Raspberry Pi에서 다음 결과가 출력되었다고 가정합니다.

```text
class=warning_red  conf=0.863 state=WARNING
```

코드를 열어 다음 표를 먼저 채웁니다.

| 화면 값 | 담당 파일 | 담당 함수 또는 코드 |
|---|---|---|
| Camera Frame |  |  |
| `warning_red` |  |  |
| `0.863` |  |  |
| `WARNING` |  |  |
| 한 줄 출력 |  |  |
| Threshold 값 |  |  |

추가 질문:

```text
1. Camera Frame은 어느 파일이 읽는가?
2. BGR → RGB는 어디에서 이루어지는가?
3. ONNX Runtime을 실제 실행하는 Class는 무엇인가?
4. Class 이름은 ONNX 숫자만으로 알 수 있는가?
5. Confidence를 운영 State로 바꾸는 함수는 무엇인가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

| 화면 값 | 담당 파일 | 담당 함수 또는 코드 |
|---|---|---|
| Camera Frame | `src/camera_input.py` | `CameraInput.read()` |
| `warning_red` | `src/inference_onnx.py` | `ONNXClassifier.predict()` + Metadata `classes` |
| `0.863` | `src/inference_onnx.py` | Softmax 후 선택된 Class Probability |
| `WARNING` | `src/decision.py` | `ai_to_operational_state()` |
| 한 줄 출력 | `scripts/onnx_camera_probe.py` | 반복문 안의 `print()` |
| Threshold 값 | `configs/settings.yaml` | `ai_confidence_threshold` |

전체 흐름은 다음과 같습니다.

```text
camera_input.py
→ Frame
        ↓
ai_preprocess.py
→ NCHW float32
        ↓
inference_onnx.py
→ Class + Confidence
        ↓
decision.py
→ NORMAL / WARNING
        ↓
onnx_camera_probe.py
→ 화면 출력
```

ONNX 계산 결과만으로 `warning_red`라는 문자열이 자동으로 생기는 것은 아닙니다. Metadata의 Class 순서가 함께 있어야 Output Index를 올바른 Class 이름으로 해석할 수 있습니다.

</details>

---

# 49. Challenge 2 — 실행 전에 운영 State를 예측하기

현재 설정:

```text
warning_class = warning_red
confidence_threshold = 0.70
```

다음 네 결과의 운영 State를 실행 전에 먼저 작성합니다.

| Prediction | Confidence | 예상 State |
|---|---:|---|
| warning_red | 0.82 |  |
| warning_red | 0.58 |  |
| normal | 0.99 |  |
| normal | 0.45 |  |

특히 `normal 0.45`가 왜 WARNING이 아닌지 `decision.py`의 조건을 읽고 설명합니다.

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

현재 Baseline 함수는 다음 두 조건을 모두 만족할 때만 WARNING입니다.

```text
예측 Class == warning_red
AND
Confidence >= 0.70
```

따라서:

| Prediction | Confidence | State | 이유 |
|---|---:|---|---|
| warning_red | 0.82 | WARNING | 경고 Class이며 0.70 이상 |
| warning_red | 0.58 | NORMAL | 경고 Class지만 기준 미만 |
| normal | 0.99 | NORMAL | 경고 Class가 아님 |
| normal | 0.45 | NORMAL | 경고 Class가 아님 |

이 실습의 2-State Baseline은 낮은 Confidence를 별도 상태로 분리하지 않습니다.

```text
오늘
→ NORMAL / WARNING만 사용

실제 운영 시스템
→ UNKNOWN / REVIEW 같은 정책을 별도로 설계할 수 있음
```

즉 **Model Prediction과 운영 정책은 같은 것이 아닙니다.**

</details>

---

# 50. Challenge 3 — Python 코드는 바꾸지 않고 Config만 변경하기

이번 Challenge에서는 Python 파일을 수정하지 않습니다.

같은 `warning_red.jpg`를 사용하고 `configs/settings.yaml`의 다음 값만 바꿉니다.

실험 A:

```yaml
ai_confidence_threshold: 0.50
```

실행:

```bash
python -m scripts.onnx_decision_test \
  --image data/deploy_samples/warning_red.jpg
```

실험 B:

```yaml
ai_confidence_threshold: 0.70
```

같은 명령을 실행합니다.

실험 C:

```yaml
ai_confidence_threshold: 0.90
```

같은 명령을 실행합니다.

실행 전에 다음을 예상합니다.

```text
ONNX Prediction Class 자체가 바뀔까?

Confidence 자체가 바뀔까?

운영 State는 바뀔 수 있을까?

왜 같은 이미지와 같은 모델을 사용해야 할까?
```

다음 표를 작성합니다.

| Threshold | Prediction | Confidence | State | 관찰 |
|---:|---|---:|---|---|
| 0.50 |  |  |  |  |
| 0.70 |  |  |  |  |
| 0.90 |  |  |  |  |

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

같은 이미지와 같은 ONNX Model을 사용했다면 보통 **Prediction과 Confidence는 그대로**이고 운영 State만 Threshold에 따라 달라질 수 있습니다.

예를 들어 Confidence가 `0.82`라면:

```text
0.50
→ 0.82 >= 0.50
→ WARNING

0.70
→ 0.82 >= 0.70
→ WARNING

0.90
→ 0.82 < 0.90
→ NORMAL
```

변경된 것은 Model Weight가 아니라 운영 기준입니다.

```text
ONNX
→ Prediction + Confidence
        ↓
Config Threshold
        ↓
Operational State
```

한 번에 한 변수만 변경했기 때문에 결과 차이의 원인을 Threshold로 설명하기 쉽습니다.

</details>

Challenge가 끝나면 기본값을 다시 사용합니다.

```yaml
ai_confidence_threshold: 0.70
```

---

# 51. Challenge 4 — 기존 모듈을 조합해 Rule / AI Mode 전환 기능 만들기

5일차에는 Rule로 판단했고 오늘은 AI로 판단했습니다.

이번에는 Camera와 GPIO 코드를 다시 만들지 않고 **판단 방식만 Config로 선택**하도록 연결합니다.

`configs/settings.yaml`에 다음 항목을 추가합니다.

```yaml
decision_mode: ai
```

허용할 값:

```text
rule
ai
```

목표:

```text
decision_mode: rule
→ 5일차 RedAreaRule

decision_mode: ai
→ 7일차 ONNXClassifier

두 경우 모두
→ 같은 CameraInput
→ 같은 GPIOOutput
```

새 파일:

```text
scripts/run_edge_mode.py
```

바로 정답 코드를 보지 말고 다음 Gate를 순서대로 해결합니다.

```text
Gate 1
→ decision_mode 읽기

Gate 2
→ rule / ai가 아니면 오류

Gate 3
→ rule이면 RedAreaRule 준비

Gate 4
→ ai이면 ONNXClassifier 준비

Gate 5
→ Camera Frame에서 Mode별 State 만들기

Gate 6
→ 같은 GPIOOutput으로 NORMAL / WARNING 출력

Gate 7
→ 현재 Mode와 결과를 터미널에 출력
```

### 의사코드

직접 먼저 작성합니다.

```text
Config를 읽는다
        ↓
decision_mode를 읽는다
        ↓
rule / ai인지 확인한다
        ↓
Camera와 GPIO를 만든다
        ↓
rule이면 RedAreaRule 준비
ai이면 ONNXClassifier 준비
        ↓
반복
        ↓
Frame 읽기
        ↓
rule
→ Red Area → State

ai
→ Preprocess → ONNX → Class / Confidence → State
        ↓
State를 GPIO에 전달
        ↓
Mode별 결과 출력
        ↓
Ctrl+C
        ↓
Camera / GPIO 정리
```

먼저 직접 구현한 뒤 아래 예시와 비교합니다.

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

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
from src.gpio_output import GPIOOutput
from src.inference_onnx import (
    ONNXClassifier,
)
from src.rule_engine import RedAreaRule


def build_rule(config):
    return RedAreaRule(
        h1_min=int(config["red_h1_min"]),
        h1_max=int(config["red_h1_max"]),
        h2_min=int(config["red_h2_min"]),
        h2_max=int(config["red_h2_max"]),
        s_min=int(config["red_s_min"]),
        v_min=int(config["red_v_min"]),
        area_threshold=int(
            config["red_area_threshold"]
        ),
    )


def main():
    config = load_config(
        "configs/settings.yaml"
    )

    mode = str(
        config.get(
            "decision_mode",
            "ai",
        )
    ).strip().lower()

    if mode not in {
        "rule",
        "ai",
    }:
        raise ValueError(
            "decision_mode는 "
            "'rule' 또는 'ai'여야 합니다."
        )

    camera = CameraInput(
        index=int(config["camera_index"]),
        width=int(config["image_width"]),
        height=int(config["image_height"]),
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

    rule = None
    classifier = None

    if mode == "rule":
        rule = build_rule(config)

    if mode == "ai":
        classifier = ONNXClassifier(
            config["onnx_model_path"],
            config["onnx_meta_path"],
        )

    try:
        print("=== Edge Decision Mode ===")
        print(f"Mode: {mode}")
        print("종료: Ctrl+C")
        print()

        time.sleep(1.0)

        while True:
            frame = camera.read()

            if mode == "rule":
                result = rule.analyze(frame)
                state = result.state

                detail = (
                    f"red_area="
                    f"{result.red_area}"
                )

            else:
                input_tensor = (
                    bgr_frame_to_nchw(
                        frame,
                        classifier.image_size,
                    )
                )

                prediction = (
                    classifier.predict(
                        input_tensor
                    )
                )

                state = (
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
                            config[
                                "ai_warning_class"
                            ],
                        confidence_threshold=
                            float(
                                config[
                                    "ai_confidence_threshold"
                                ]
                            ),
                    )
                )

                detail = (
                    f"class="
                    f"{prediction['class_name']} "
                    f"conf="
                    f"{prediction['confidence']:.3f}"
                )

            if state == "WARNING":
                output.warning()
            else:
                output.normal()

            print(
                f"mode={mode:4s} | "
                f"{detail} | "
                f"state={state}"
            )

    except KeyboardInterrupt:
        print("\n종료합니다.")

    finally:
        camera.release()
        output.close()


if __name__ == "__main__":
    main()
```

Rule Mode:

```yaml
decision_mode: rule
```

```bash
python -m scripts.run_edge_mode
```

AI Mode:

```yaml
decision_mode: ai
```

```bash
python -m scripts.run_edge_mode
```

핵심은 Mode가 바뀌어도 다음 모듈은 그대로라는 점입니다.

```text
CameraInput
→ 재사용

GPIOOutput
→ 재사용

바뀌는 것
→ 판단 모듈
```

</details>

Challenge가 끝나면 다음 기본값을 사용합니다.

```yaml
decision_mode: ai
```

---

# 52. Challenge 5 — 일부러 Model Path 오류를 만들고 원인 찾기

이번에는 실제 ONNX 파일을 삭제하지 않습니다.

`configs/settings.yaml`의 Model 경로를 잠시 잘못 작성합니다.

정상:

```yaml
onnx_model_path: models/day07_tiny_cnn.onnx
```

임시 오류:

```yaml
onnx_model_path: models/day07_tiny_cnn_missing.onnx
```

실행:

```bash
python -m scripts.predict_onnx \
  --image data/deploy_samples/normal.jpg
```

정답을 바로 보지 말고 오류의 **마지막 줄부터** 읽습니다.

다음 질문에 답합니다.

```text
오류 종류는 무엇인가?

어느 파일이 없다고 하는가?

그 경로는 어디에서 읽었는가?

ONNX Runtime 자체가 고장난 것인가?

Config를 어떻게 복구해야 하는가?
```

수정 후 다시 실행합니다.

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

`src/inference_onnx.py`의 `ONNXClassifier`는 Model 파일 존재 여부를 먼저 확인합니다.

따라서 예상되는 문제 영역은 다음입니다.

```text
Model Path / Deployment Artifact
```

흐름:

```text
settings.yaml
→ 잘못된 onnx_model_path
        ↓
predict_onnx.py
        ↓
ONNXClassifier(...)
        ↓
Path.exists() == False
        ↓
FileNotFoundError
```

이 경우 다음부터 확인합니다.

```bash
grep "onnx_model_path" configs/settings.yaml
ls -lh models
```

정상 경로로 복구합니다.

```yaml
onnx_model_path: models/day07_tiny_cnn.onnx
```

다시 실행합니다.

```bash
python -m scripts.predict_onnx \
  --image data/deploy_samples/normal.jpg
```

정상 Prediction이 다시 나오면 복구 완료입니다.

</details>

---

# 53. Challenge 6 — Log와 저장 파일로 정상 동작을 증명하기

화면에서 LED가 켜졌다는 기억만으로 완료하지 않습니다.

먼저 AI Warning System을 실행합니다.

```bash
python -m scripts.ai_warning_system
```

Camera 앞에서 다음 순서를 한 번 만듭니다.

```text
경고 대상 없음
→ NORMAL

빨간 경고 대상 제시
→ WARNING

다시 제거
→ NORMAL
```

종료:

```text
Ctrl+C
```

이제 다음 증거를 확인합니다.

Event Log:

```bash
tail -n 10 logs/day07_ai_events.csv
```

WARNING Image:

```bash
find data/ai_warning_images \
  -type f \
  -name "*.jpg" \
  | sort \
  | tail
```

배포 Sample Hash:

```bash
sha256sum \
  data/deploy_samples/normal.jpg \
  data/deploy_samples/warning_red.jpg
```

다음 질문에 답합니다.

```text
Event CSV에 NORMAL → WARNING → NORMAL 변화가 남았는가?

WARNING 행에 image_path가 기록되었는가?

그 image_path의 JPG가 실제로 존재하는가?

PC와 Pi에서 비교한 deploy sample은 같은 파일이었는가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

정상적인 Event Log 예시는 다음 형태입니다.

```csv
timestamp,mode,pred_class,confidence,threshold,state,image_path
...,ai,normal,0.94,0.700,NORMAL,
...,ai,warning_red,0.86,0.700,WARNING,data/ai_warning_images/...
...,ai,normal,0.91,0.700,NORMAL,
```

확인할 연결:

```text
AI Prediction
        ↓
Operational Decision
        ↓
State가 이전과 달라짐
        ↓
AIEventLogger.append()
        ↓
CSV Event

새 State가 WARNING
        ↓
save_ai_warning_image()
        ↓
JPG 저장
        ↓
CSV image_path 기록
```

PC와 Raspberry Pi의 `normal.jpg`, `warning_red.jpg` SHA-256이 각각 같다면 **같은 입력 파일을 비교했다는 증거**가 됩니다.

</details>

---

# 54. Challenge 7 — 오늘의 전체 Pipeline을 자신의 말로 설명하기

코드를 닫고 아래 빈칸을 먼저 채웁니다.

```text
6일차 PyTorch Model
→ ______________________
→ ONNX Model

저장 이미지
→ ______________________
→ NCHW float32

NCHW Tensor
→ ______________________
→ Class + Confidence

Class + Confidence
→ ______________________
→ NORMAL / WARNING

Camera
→ Preprocess
→ ONNX
→ Decision
→ ______________________
→ Event Log
```

그다음 오늘의 전체 흐름을 **세 문장 이내**로 설명합니다.

조건:

```text
1. 6일차와의 연결을 포함한다.
2. PC와 Raspberry Pi의 역할을 포함한다.
3. 8일차에 무엇을 측정할지 포함한다.
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

빈칸 예시:

```text
PyTorch Model
→ export_onnx.py
→ ONNX Model

저장 이미지
→ ai_preprocess.py
→ NCHW float32

NCHW Tensor
→ ONNXClassifier
→ Class + Confidence

Class + Confidence
→ decision.py
→ NORMAL / WARNING

Camera
→ Preprocess
→ ONNX
→ Decision
→ GPIO
→ Event Log
```

세 문장 예시:

> 6일차에서 학습한 Tiny CNN PyTorch Model을 7일차에서 ONNX로 변환하고 같은 전처리와 Class 순서를 유지한 채 PC와 Raspberry Pi에서 결과를 비교했습니다. Raspberry Pi에서는 Camera Frame을 ONNX Runtime으로 추론한 뒤 Confidence 기준으로 NORMAL/WARNING을 만들고 GPIO와 Event Log에 연결했습니다. 8일차에서는 이 정상 Baseline을 바꾸지 않고 Model-only와 End-to-End Latency/FPS를 측정합니다.

문장을 그대로 외울 필요는 없습니다. **Model → Runtime → 운영 Decision → 장치 출력**의 연결을 설명할 수 있으면 됩니다.

</details>

---

# 55. 추가 Failure Lab — 배포는 실행 성공만으로 끝나지 않습니다

시간이 허용되면 다음 세 가지를 확인합니다.

### Case A — Input Shape가 틀림

원본 전처리 파일을 수정하지 않고 별도 배열을 만듭니다.

```python
wrong = np.zeros(
    (1, 3, 64, 64),
    dtype=np.float32,
)
```

96×96 고정 Input Model에 이 배열을 넣으면 Dimension 관련 오류가 날 수 있습니다.

```text
오류 영역
→ Input Contract
```

### Case B — BGR을 RGB로 바꾸지 않음

Shape와 dtype이 맞아도 Channel 의미가 다르면 프로그램은 실행되면서 Prediction이 달라질 수 있습니다.

```text
실행 성공
≠
전처리 의미가 올바름
```

### Case C — Metadata Class 순서가 뒤집힘

```text
정상
0 → normal
1 → warning_red

잘못된 해석
0 → warning_red
1 → normal
```

Model Output 값 자체는 같아도 Class 이름 해석이 반대로 바뀔 수 있습니다.

이 세 가지는 각각 다른 문제입니다.

```text
Shape
→ Tensor 구조

BGR / RGB
→ Preprocess 의미

Class 순서
→ Metadata Contract
```

원본 Baseline 파일은 수정하지 않고 **복사본이나 별도 Test 코드**로만 실험합니다.

---

# 56. Mini Challenge 결과를 Report에 기록하기

`reports/day07_onnx_deployment.md`에 다음 내용을 추가합니다.

```md
## Mini Challenge

### 1. Output Trace

Camera Frame:

Class / Confidence:

Operational State:

Terminal Output:

### 2. Prediction Before Run

내가 예상한 State:

실제 결과:

### 3. Config-only Threshold

| Threshold | Prediction | Confidence | State |
|---:|---|---:|---|
| 0.50 |  |  |  |
| 0.70 |  |  |  |
| 0.90 |  |  |  |

### 4. Rule / AI Mode

Rule Mode 결과:

AI Mode 결과:

같은 Camera / GPIO를 재사용했다는 근거:

### 5. Deliberate Error

오류:

원인:

수정:

재실행 결과:

### 6. Runtime Evidence

Event Log:

WARNING Image:

Deploy Sample Hash:

### 7. Pipeline

내 설명:
```

---

# 57. Mini Challenge Review

교재를 보지 않고 다음 질문에 답합니다.

```text
1. ONNX 파일과 Metadata가 둘 다 필요한 이유는 무엇인가?
2. NCHW의 N/C/H/W는 무엇인가?
3. BGR → RGB는 어느 입력에서 특히 필요한가?
4. PyTorch와 ONNX를 비교할 때 같은 Tensor를 쓰는 이유는 무엇인가?
5. Prediction과 Operational State의 차이는 무엇인가?
6. Config Threshold만 바꾸면 Model Weight도 바뀌는가?
7. Model 경로 오류와 Input Shape 오류는 어떻게 구분할 수 있는가?
8. Event Log는 매 Frame을 모두 기록하는가?
9. `decision_mode`를 바꾸어도 재사용하는 모듈은 무엇인가?
10. 8일차에는 오늘 무엇을 그대로 사용해야 하는가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
1. ONNX에는 계산 Graph/Weight가 있고, Metadata는 Class 순서와 Input Size 같은 해석 정보를 보존하기 때문이다.

2. N=Batch, C=Channel, H=Height, W=Width이다.

3. OpenCV Camera Frame이 BGR이므로 AI 입력의 RGB 의미와 맞출 때 필요하다.

4. 전처리 차이를 제거하고 Runtime 변환 자체의 결과 일치 여부를 보기 위해서이다.

5. Prediction은 Model의 Class/Confidence이고, Operational State는 이를 운영 기준과 결합한 NORMAL/WARNING이다.

6. 바뀌지 않는다. 운영 Decision 기준만 바뀐다.

7. Model 경로 오류는 파일 존재/경로에서, Shape 오류는 ONNX Runtime 입력 Dimension에서 확인한다.

8. 현재 코드는 첫 State와 State가 바뀌는 순간만 Event로 기록한다.

9. CameraInput과 GPIOOutput 등 입력·출력 구조는 그대로 재사용한다.

10. day07 ONNX, Metadata, 공통 Preprocess, 동일 Test Sample, 정상 Source와 Config를 유지한다.
```

</details>

---

# 58. 오류가 발생했을 때 확인하는 순서

7일차는 PC·Network·Raspberry Pi·Model·Camera·GPIO가 연결되므로 오류를 한꺼번에 추측하지 않습니다.

```text
1. 현재 장치가 PC인가 Pi인가?
        ↓
2. 현재 Project Path가 맞는가?
        ↓
3. 현재 .venv Python이 맞는가?
        ↓
4. 필요한 Source 파일이 있는가?
        ↓
5. Model / Metadata 파일이 있는가?
        ↓
6. ONNX Runtime Import가 되는가?
        ↓
7. Input Shape가 맞는가?
        ↓
8. Preprocess가 RGB / float32 / 0~1 / NCHW인가?
        ↓
9. 저장 이미지 Prediction이 정상인가?
        ↓
10. Camera가 단독으로 정상인가?
        ↓
11. Camera ONNX Prediction이 정상인가?
        ↓
12. Operational Decision이 정상인가?
        ↓
13. GPIO가 단독으로 정상인가?
        ↓
14. Event Log / Image 저장 경로가 정상인가?
```

증상별 첫 확인 위치:

| 증상 | 먼저 확인할 영역 |
|---|---|
| ONNX 파일이 생성되지 않음 | Export / Package / Checkpoint |
| `FileNotFoundError` Model | `onnx_model_path` / SCP |
| Metadata 없음 | `onnx_meta_path` / 배포 Artifact |
| Input Dimension 오류 | NCHW / `image_size` |
| 실행은 되지만 Class 이상 | RGB/BGR / Scale / Class 순서 |
| Pi에서 `import onnxruntime` 실패 | `.venv` / Wheel / Architecture |
| PC와 Pi Class가 다름 | 동일 Model / 동일 Image Hash / Preprocess |
| Prediction 정상, State 이상 | `decision.py` / Confidence Threshold |
| State 정상, LED 이상 | GPIO 배선 / BCM / `GPIOOutput` |
| WARNING인데 이미지 없음 | `ai_warning_image_dir` / 저장 권한 |
| Log 없음 | `AIEventLogger` / Log 경로 / 상태 변화 |

이 순서를 사용하면 “AI가 안 된다”처럼 너무 넓게 보지 않고 실패 지점을 좁힐 수 있습니다.

---

# PART F. Day 07 Baseline 복원 · Report · Git · README


# 59. Mini Challenge가 끝난 뒤 Day 07 Baseline으로 복원하기

Challenge는 실험을 위한 변경이므로 8일차 전에 기본 상태를 다시 고정합니다.

`configs/settings.yaml`의 Day 07 항목을 확인합니다.

```yaml
onnx_model_path: models/day07_tiny_cnn.onnx
onnx_meta_path: models/day07_tiny_cnn.meta.json

ai_warning_class: warning_red
ai_confidence_threshold: 0.70

ai_event_log_path: logs/day07_ai_events.csv
ai_warning_image_dir: data/ai_warning_images

decision_mode: ai
```

5일차의 기존 `use_buzzer` 설정은 자신의 장비 상태를 유지합니다.

```yaml
use_buzzer: false
```

또는 Buzzer 사용이 검증된 장비라면:

```yaml
use_buzzer: true
```

Challenge에서 Model 이름을 바꾸었다면 원래 파일이 존재하는지 확인합니다.

```bash
ls -lh \
  models/day07_tiny_cnn.onnx \
  models/day07_tiny_cnn.meta.json
```

PC에서 배포 Sample도 확인합니다.

```bash
ls -lh \
  data/deploy_samples/normal.jpg \
  data/deploy_samples/warning_red.jpg
```

Raspberry Pi에서도 같은 두 Model Artifact가 있는지 확인합니다.

```bash
ssh <RPI_USER>@<RPI_IP> \
"cd ~/ai_vision/subject13_edge_ai && ls -lh models/day07_tiny_cnn.onnx models/day07_tiny_cnn.meta.json"
```

이 상태가 8일차 성능 측정의 출발점입니다.

---

# 60. 7일차 Source와 Metadata Git Checkpoint 만들기

먼저 Git 상태를 확인합니다.

```bash
git status
git diff
```

`data/`, `logs/`, `.venv/`, `*.pt`, `*.onnx`, `configs/device.local.yaml`은 Git 대상에 나타나지 않는 것이 기본입니다.

다음 Source와 설정을 Stage합니다.

```bash
git add \
  configs/settings.yaml \
  src/ai_preprocess.py \
  src/inference_onnx.py \
  src/decision.py \
  src/ai_warning.py \
  src/event_logger.py \
  scripts/export_onnx.py \
  scripts/check_ai_input.py \
  scripts/predict_onnx.py \
  scripts/compare_pytorch_onnx.py \
  scripts/inspect_onnx_runtime.py \
  scripts/onnx_decision_test.py \
  scripts/onnx_camera_probe.py \
  scripts/ai_warning_system.py \
  scripts/run_edge_mode.py \
  models/day07_tiny_cnn.meta.json
```

Metadata JSON은 작은 파일이며 Class 순서와 Input Size를 보존하는 **배포 Contract**이므로 이 수업에서는 Git으로 관리합니다.

Binary Model은 제외합니다.

```text
models/day06_tiny_cnn.pt
models/day07_tiny_cnn.onnx
```

Commit:

```bash
git commit -m \
"feat: deploy tiny cnn with onnx runtime"
```

---

# 61. 7일차 Report 완성하기

`reports/day07_onnx_deployment.md`에 다음 항목까지 추가합니다.

```md
## Raspberry Pi

Architecture:

ONNX Runtime Version:

Execution Provider:

Input Shape:

Output Shape:

## PC ONNX vs Raspberry Pi ONNX

| Image | PC Class | PC Confidence | Pi Class | Pi Confidence | Match |
|---|---|---:|---|---:|---|
| normal.jpg |  |  |  |  |  |
| warning_red.jpg |  |  |  |  |  |

## Confidence Threshold

| Threshold | Operational State | 관찰 |
|---:|---|---|
| 0.50 |  |  |
| 0.70 |  |  |
| 0.90 |  |  |

## Live Camera

normal 상태:

warning_red 상태:

GPIO:

Event Log:

Warning Image:

## Rule vs AI

같은 점:

달라진 점:

Rule이 더 잘 동작한 조건:

AI가 더 잘 동작한 조건:

## Failure Test

### Case 1

문제 영역:

증상:

원인:

해결:

### Case 2

문제 영역:

증상:

원인:

해결:

## Day 08 Handoff

ONNX Model:

Metadata:

Deploy Sample Hash:

Baseline Confidence Threshold:

Baseline Decision Mode:

Raspberry Pi Provider:

8일차에서 그대로 사용할 항목:
```

---

# 62. 환경 기록과 Report를 Git에 저장하기

Raspberry Pi 환경 기록을 PC로 가져옵니다.

```bash
scp \
  <RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/reports/day07_pi_environment.txt \
  reports/
```

현재 파일을 확인합니다.

```bash
ls -lh \
  reports/day07_onnx_deployment.md \
  reports/day07_pc_environment.txt \
  reports/day07_pi_environment.txt
```

Stage:

```bash
git add \
  reports/day07_onnx_deployment.md \
  reports/day07_pc_environment.txt \
  reports/day07_pi_environment.txt \
  README.md
```

Commit:

```bash
git commit -m \
"docs: record day07 onnx deployment results"
```

환경 기록에는 Package 버전만 남기고 비밀번호·토큰·Wi-Fi 정보 같은 민감정보는 넣지 않습니다.

---

# 63. README에 7일차 실행 방법 추가하기

README는 설명을 길게 복사하는 문서가 아니라 **현재 정상 Version을 다시 실행하기 위한 짧은 안내서**로 사용합니다.

다음 내용을 `README.md`에 추가합니다.

````md
## Day 07 — ONNX Deployment

### PC — Export

```bash
python -m scripts.export_onnx
```

### PC — ONNX Inspect

```bash
python -m scripts.inspect_onnx_runtime
```

### PC — PyTorch vs ONNX

```bash
python -m scripts.compare_pytorch_onnx \
  --image <TEST_IMAGE>
```

### PC / Raspberry Pi — ONNX Prediction

```bash
python -m scripts.predict_onnx \
  --image <TEST_IMAGE>
```

### Raspberry Pi — Camera ONNX Probe

```bash
python -m scripts.onnx_camera_probe
```

### Raspberry Pi — AI Warning System

```bash
python -m scripts.ai_warning_system
```

### Rule / AI Mode

```bash
python -m scripts.run_edge_mode
```

### Current AI Pipeline

```text
Camera
→ Preprocess
→ ONNX Runtime
→ Class + Confidence
→ Operational Decision
→ NORMAL / WARNING
→ GPIO
→ Event Log
```

### Day 08 Handoff

```text
models/day07_tiny_cnn.onnx
models/day07_tiny_cnn.meta.json
data/deploy_samples/normal.jpg
data/deploy_samples/warning_red.jpg
```
````

README에 적은 명령은 실제 파일명과 일치하는지 직접 확인합니다.

---

# 64. Remote 저장소 사용은 교육장 보안정책에 맞춥니다

외부 GitHub 사용을 전제로 하지 않습니다.

```text
허용된 내부 GitLab / Gitea가 있음
→ Commit 후 내부 Remote에 Push 가능

외부 Git 금지
→ PC Local Git Commit으로 Version 관리

Raspberry Pi가 Git Repository가 아님
→ PC에서 Local Git 관리
→ 필요한 Source는 SCP로 Pi에 전달
```

허용된 Remote가 연결되어 있는 경우에만 확인합니다.

```bash
git remote -v
git push
```

다음 항목은 외부 Remote에 올리지 않습니다.

```text
Raw Dataset
Camera Image
Runtime Log
*.pt
*.onnx
device.local.yaml
비밀번호 / Token / Wi-Fi 정보
```

---

# 65. PC와 Raspberry Pi의 Source Version을 마지막으로 맞추기

8일차 Benchmark는 두 환경이 같은 코드와 같은 Model을 사용해야 의미가 있습니다.

### 내부 Remote를 사용하는 경우

PC:

```bash
git log --oneline -8
```

Raspberry Pi:

```bash
ssh <RPI_USER>@<RPI_IP>
cd ~/ai_vision/subject13_edge_ai
git pull
git log --oneline -8
```

### SCP 방식인 경우

PC의 현재 정상 Source를 Raspberry Pi로 다시 전달합니다.

```bash
scp \
  src/ai_preprocess.py \
  src/inference_onnx.py \
  src/decision.py \
  src/ai_warning.py \
  src/event_logger.py \
  <RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/src/
```

```bash
scp \
  scripts/predict_onnx.py \
  scripts/inspect_onnx_runtime.py \
  scripts/onnx_decision_test.py \
  scripts/onnx_camera_probe.py \
  scripts/ai_warning_system.py \
  scripts/run_edge_mode.py \
  <RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/scripts/
```

Raspberry Pi의 `configs/device.local.yaml`은 덮어쓰지 않습니다.

Model은 별도로 확인합니다.

```bash
ssh <RPI_USER>@<RPI_IP> \
"cd ~/ai_vision/subject13_edge_ai && ls -lh models/day07_tiny_cnn.onnx models/day07_tiny_cnn.meta.json"
```

---

# 66. 7일차가 끝난 시점의 프로젝트 구조

공통 Source:

```text
subject13_edge_ai/
│
├── configs/
│   └── settings.yaml
│
├── reports/
│   ├── ...
│   ├── day06_ai_baseline.md
│   ├── day07_onnx_deployment.md
│   ├── day07_pc_environment.txt
│   └── day07_pi_environment.txt
│
├── scripts/
│   ├── ...
│   ├── ai_warning_system.py
│   ├── check_ai_input.py
│   ├── compare_pytorch_onnx.py
│   ├── export_onnx.py
│   ├── inspect_onnx_runtime.py
│   ├── onnx_camera_probe.py
│   ├── onnx_decision_test.py
│   ├── predict_onnx.py
│   └── run_edge_mode.py
│
├── src/
│   ├── ...
│   ├── ai_preprocess.py
│   ├── ai_warning.py
│   ├── decision.py
│   └── inference_onnx.py
│
├── models/
│   └── day07_tiny_cnn.meta.json
│
├── .gitignore
├── README.md
└── requirements.txt
```

PC에 존재:

```text
models/
├── day06_tiny_cnn.pt
├── day07_tiny_cnn.onnx
└── day07_tiny_cnn.meta.json

data/
└── dataset/
```

Raspberry Pi에 존재:

```text
models/
├── day07_tiny_cnn.onnx
└── day07_tiny_cnn.meta.json

data/
├── deploy_samples/
└── ai_warning_images/

logs/
└── day07_ai_events.csv
```


---

# 67. 7일차 최종 실행 확인

## PC

```bash
cd ~/ai_vision/subject13_edge_ai
source .venv/bin/activate
```

ONNX:

```bash
python -m scripts.inspect_onnx_runtime
```

Parity:

```bash
python -m scripts.compare_pytorch_onnx \
  --image data/deploy_samples/normal.jpg
```

Git:

```bash
git status
git log --oneline -9
```

---

## Raspberry Pi

```bash
ssh <RPI_USER>@<RPI_IP>
```

```bash
cd ~/ai_vision/subject13_edge_ai
source .venv/bin/activate
```

ONNX:

```bash
python -m scripts.inspect_onnx_runtime
```

저장 이미지:

```bash
python -m scripts.predict_onnx \
  --image data/deploy_samples/normal.jpg
```

Camera:

```bash
python -m scripts.onnx_camera_probe
```

AI Warning:

```bash
python -m scripts.ai_warning_system
```

Log:

```bash
tail logs/day07_ai_events.csv
```

---

# PART G. 최종 복습 · 자가 점검 · 8일차 연결


# 68. 7일차 핵심 복습 문제

교재와 코드를 잠시 닫고 먼저 답해 봅니다.

1. 6일차에서 만든 `models/day06_tiny_cnn.pt`는 7일차에서 어떤 역할을 하나요?
2. ONNX Export는 단순히 파일 확장자를 바꾸는 작업인가요?
3. `day07_tiny_cnn.onnx`와 `day07_tiny_cnn.meta.json`은 각각 무엇을 보존하나요?
4. `[1, 3, 96, 96]`의 네 숫자는 무엇을 의미하나요?
5. OpenCV Camera Frame이 BGR이라는 점이 왜 중요할까요?
6. 6일차 Training과 7일차 추론의 전처리가 같아야 하는 이유는 무엇인가요?
7. `float32`와 `0~1` 값 범위는 어디에서 만들어지나요?
8. PyTorch와 ONNX의 결과를 비교할 때 왜 같은 NCHW Tensor를 사용하나요?
9. `Class Match: True`만 확인하고 Probability 차이는 전혀 볼 필요가 없나요?
10. `CPUExecutionProvider`는 무엇을 의미하나요?
11. PC와 Raspberry Pi에서 같은 Test Image를 사용했다는 것을 어떻게 확인할 수 있나요?
12. `warning_red 0.82`와 `WARNING`은 같은 종류의 정보인가요?
13. `ai_confidence_threshold`를 바꾸면 Tiny CNN Weight도 바뀌나요?
14. `onnx_camera_probe.py`와 `ai_warning_system.py`의 역할은 어떻게 다른가요?
15. 3일차에서 만든 어떤 기능을 7일차에서 다시 사용하나요?
16. 5일차에서 배운 어떤 운영 개념을 7일차에서 다시 사용하나요?
17. WARNING 상태가 계속 유지될 때 모든 Frame을 Event Log로 저장하지 않는 이유는 무엇인가요?
18. `device.local.yaml`을 PC 설정으로 덮어쓰면 안 되는 이유는 무엇인가요?
19. ONNX Runtime 설치 실패와 Python 코드 오류를 어떻게 구분할 수 있나요?
20. 8일차에서 오늘 만든 어떤 결과를 그대로 사용하나요?

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

1. 6일차에서 Validation 기준으로 고정한 PyTorch Baseline Weight와 Class/Input 정보를 읽어 ONNX로 Export하는 원본입니다.
2. 아닙니다. 같은 Model 구조를 다시 만들고 Weight를 로드한 뒤 계산 Graph를 ONNX로 내보내고 구조와 결과를 검증하는 과정입니다.
3. ONNX는 주로 계산 Graph와 Weight를 보존하고, Metadata는 Class 순서와 Input Size 같은 해석 정보를 보존합니다.
4. `N=1`, `C=3`, `H=96`, `W=96`입니다.
5. Training 입력은 RGB 의미를 사용하므로 OpenCV BGR Frame을 그대로 넣으면 Channel 의미가 달라질 수 있습니다.
6. 같은 Weight라도 전처리가 달라지면 실제 입력 숫자가 달라져 Prediction이 달라질 수 있기 때문입니다.
7. `src/ai_preprocess.py`의 공통 전처리 함수에서 만듭니다.
8. Runtime 차이만 비교하고 전처리 차이를 비교 결과에서 제거하기 위해서입니다.
9. Class Match가 가장 중요하지만 Probability가 비정상적으로 크게 달라지지 않는지도 함께 확인합니다.
10. ONNX Runtime이 CPU에서 Model을 실행하는 Execution Provider입니다.
11. PC와 Pi에서 같은 파일의 SHA-256 Hash를 비교합니다.
12. 아닙니다. 전자는 Model Prediction이고 후자는 운영 기준까지 적용한 Operational State입니다.
13. 바뀌지 않습니다. 운영 Decision 기준만 바뀝니다.
14. `onnx_camera_probe.py`는 Camera→AI→Decision을 관찰하고, `ai_warning_system.py`는 GPIO·Event Log·WARNING Image까지 통합합니다.
15. `CameraInput`, `GPIOOutput`을 재사용합니다.
16. 상태 변화 Event 기록과 WARNING 대표 이미지 같은 운영 기록 개념을 다시 사용합니다.
17. 같은 상태를 매 Frame 저장하면 Log와 이미지가 지나치게 많아져 실제 상태 변화 Event를 보기 어렵기 때문입니다.
18. Camera index 같은 장치 전용 값이 Raspberry Pi마다 다를 수 있기 때문입니다.
19. `.venv`, Python 버전, Architecture, Package Import와 Wheel 호환성을 먼저 확인한 뒤 코드 실행 단계와 분리해서 판단합니다.
20. `day07_tiny_cnn.onnx`, Metadata, 공통 Preprocess, 배포 Sample, 정상 Source/Config를 그대로 사용하여 Latency/FPS를 측정합니다.

</details>

---

# 69. 자가 체크리스트

다음 항목은 읽기만 하지 말고 실제로 확인합니다.

- [ ] 6일차의 `models/day06_tiny_cnn.pt`를 새로 학습하지 않고 그대로 사용했다.
- [ ] 6일차 Test 이미지에서 PyTorch Prediction을 먼저 확인했다.
- [ ] PC의 현재 `.venv`와 Python 위치를 확인했다.
- [ ] `onnx`와 `onnxruntime` 설치 여부를 먼저 확인한 뒤 필요한 경우에만 설치했다.
- [ ] ONNX Export 후 Checker가 통과했다.
- [ ] `day07_tiny_cnn.onnx`와 `day07_tiny_cnn.meta.json`을 둘 다 확인했다.
- [ ] Metadata의 Class 순서가 `normal`, `warning_red`인지 확인했다.
- [ ] Input Shape가 `(1, 3, 96, 96)`인지 확인했다.
- [ ] Input dtype이 `float32`인지 확인했다.
- [ ] 전처리 값 범위가 0~1인지 확인했다.
- [ ] OpenCV Camera 입력에서 BGR→RGB 변환이 필요한 이유를 설명할 수 있다.
- [ ] PyTorch와 ONNX를 같은 Tensor로 비교했다.
- [ ] `Class Match`와 `Max Prob Diff`를 확인했다.
- [ ] 배포 Sample 두 장을 고정했다.
- [ ] PC에서 배포 Sample의 SHA-256을 기록했다.
- [ ] ONNX와 Metadata를 Raspberry Pi로 전송했다.
- [ ] Raspberry Pi의 실제 IP를 확인하고 고정 예시값에 의존하지 않았다.
- [ ] Raspberry Pi의 Architecture와 Python 버전을 확인했다.
- [ ] Raspberry Pi `.venv`에서 ONNX Runtime Import를 확인했다.
- [ ] Pi에서 `CPUExecutionProvider`를 확인했다.
- [ ] PC와 Pi의 동일 Sample Hash가 같은지 확인했다.
- [ ] PC ONNX와 Pi ONNX의 Class 결과를 비교했다.
- [ ] AI Prediction과 Operational State의 차이를 설명할 수 있다.
- [ ] Confidence Threshold를 한 변수만 바꾸어 비교했다.
- [ ] 저장 이미지 기반 Decision을 먼저 확인한 뒤 Camera로 넘어갔다.
- [ ] `onnx_camera_probe.py`에서 실시간 Class/Confidence/State를 확인했다.
- [ ] `ai_warning_system.py`에서 Camera→AI→Decision→GPIO→Log가 연결되었다.
- [ ] Buzzer가 검증되지 않은 경우 `use_buzzer: false`를 유지했다.
- [ ] Event CSV에 상태 변화가 기록되는 것을 확인했다.
- [ ] WARNING 전환 시 대표 이미지가 저장되는 것을 확인했다.
- [ ] Mini Challenge에서 실행 결과를 코드 위치로 역추적했다.
- [ ] Mini Challenge에서 실행 전에 결과를 예측했다.
- [ ] Mini Challenge에서 Config만 변경하여 결과를 비교했다.
- [ ] Mini Challenge에서 Rule/AI Mode 기능을 기존 모듈로 조합했다.
- [ ] Mini Challenge에서 일부러 Model Path 오류를 만들고 복구했다.
- [ ] Log·Hash·저장 이미지로 정상 동작을 증명했다.
- [ ] Challenge가 끝난 뒤 Day 07 Baseline 설정으로 복원했다.
- [ ] `decision_mode: ai` 상태를 확인했다.
- [ ] `ai_confidence_threshold: 0.70` Baseline을 확인했다.
- [ ] Binary Model과 Raw Data를 Git에서 제외했다.
- [ ] Metadata·Source·Report는 교육장 정책에 맞게 Version 관리했다.
- [ ] 외부 Git 사용이 제한된 경우 Local Git 또는 내부 GitLab/Gitea/SCP 흐름을 사용했다.
- [ ] `reports/day07_onnx_deployment.md`를 실제 결과로 채웠다.
- [ ] README의 실행 명령이 실제 파일명과 일치하는지 확인했다.
- [ ] 8일차에서 정식 Latency/FPS를 측정한다는 것을 이해했다.

---

# 70. 1~7일차 시스템 성장 확인하기

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
→ Tiny CNN Training
→ PyTorch Model
```

7일차:

```text
PyTorch
→ ONNX
→ PC 검증
→ Raspberry Pi
→ ONNX Runtime
→ Camera
→ Class + Confidence
→ Decision
→ GPIO
→ Log
```

이제 판단 부분을 다음처럼 교체할 수 있습니다.

```text
Rule
↕
AI
```

나머지 Edge Pipeline은 최대한 그대로 재사용합니다.

---

# 71. 다음 날 연결

오늘은 Raspberry Pi에서 AI가 **실행되는 것**을 확인했습니다.

하지만 다음 질문에는 아직 정확히 답하지 않았습니다.

```text
한 번 추론하는 데 몇 ms 걸리는가?

평균만 보면 충분한가?

느린 Frame은 얼마나 느린가?

Camera부터 GPIO / Log까지 포함하면 FPS는 얼마인가?

PC와 Raspberry Pi는 얼마나 차이가 나는가?
```

8일차에는 실행 성공 여부를 넘어 **성능을 숫자로 측정**합니다.

```text
Raspberry Pi ONNX Inference
        ↓
Warm-up
        ↓
반복 측정
        ↓
Mean Latency
P50
P95
        ↓
Model-only FPS
        ↓
Camera + Preprocess + AI + Decision + Log
        ↓
End-to-End FPS
```

즉 다음 날부터는:

> **“AI가 Raspberry Pi에서 실행된다.”**

에서 한 단계 더 나아가,

> **“이 AI가 실제 Edge 장치에서 얼마나 빠르게 동작하는가?”**

를 실험 결과로 설명하게 됩니다.
