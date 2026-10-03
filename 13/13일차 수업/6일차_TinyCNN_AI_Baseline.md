
> **오늘의 핵심:** 5일차에는 사람이 직접 정한 `HSV + Red Area + Threshold` Rule로 Camera 장면을 `NORMAL / WARNING`으로 판단했습니다.  
> 6일차에는 **같은 `normal / warning_red` 문제를 이미지 Dataset으로 만들고, 작은 CNN이 판단 기준을 학습하도록 바꿉니다.**
>
> 1~5일차에 사용한 `subject13_edge_ai` 프로젝트를 그대로 이어서 사용합니다. **새 프로젝트와 새 가상환경을 만들지 않습니다.**
>
> Camera 데이터 수집은 Raspberry Pi에서 진행하고, Model Training·Validation·Test는 PC에서 진행합니다.
>
> 실제 IP, Hostname, Camera index, PC의 GPU 사용 여부, Package 설치 방법은 환경마다 달라질 수 있습니다. 교재의 예시를 고정값처럼 외우지 않고 자신의 환경에서 확인합니다.
>
> 비밀번호, Wi-Fi 비밀번호, 인증키 같은 민감한 정보는 README·Report·Git에 기록하지 않습니다.

---

## 오늘 가장 중요한 질문

5일차까지의 Edge Pipeline은 다음과 같았습니다.

```text
USB Camera
    ↓
OpenCV Frame
    ↓
HSV / Red Mask / Red Area
    ↓
사람이 정한 Threshold
    ↓
NORMAL / WARNING
    ↓
GPIO / Event Log / WARNING Image
```

오늘은 Camera와 출력 장치를 바꾸는 날이 아닙니다.  
**판단 기준을 만드는 방법**을 바꾸는 날입니다.

```text
5일차 — Rule

Image
  ↓
사람이 특징을 직접 선택
HSV / Red Area
  ↓
사람이 Threshold를 직접 정함
  ↓
NORMAL / WARNING


6일차 — AI

Image + 정답 Label
  ↓
Tiny CNN Training
  ↓
Model이 Weight를 학습
  ↓
normal / warning_red
```

따라서 오늘 가장 중요한 질문은 다음입니다.

> **“5일차와 같은 문제를 공정한 이미지 Dataset으로 만들고, Train·Validation·Test를 분리한 뒤, 작은 CNN을 학습하여 7일차 ONNX 배포에 사용할 PyTorch Baseline Model을 어떻게 고정할 것인가?”**

오늘 만든 Model은 아직 Raspberry Pi에서 실시간 추론하지 않습니다.

```text
6일차
Raspberry Pi Camera
→ Session Dataset 수집
→ PC로 이동
→ Train / Validation / Test
→ Tiny CNN Training
→ Best Validation Model
→ Test 최종 평가
→ models/day06_tiny_cnn.pt

        ↓

7일차
PyTorch Model
→ ONNX Export
→ PC 결과 일치 확인
→ Raspberry Pi 전송
→ ONNX Runtime
→ Camera 실시간 추론
```

---

## 5일차와 6일차의 Label 관계를 먼저 정확히 이해합니다

6일차의 정답 Label은 **5일차 Rule이 출력한 결과를 그대로 복사하면 안 됩니다.**

예를 들어 실제 장면에 빨간 경고 카드가 있지만, 어두운 조명 때문에 5일차 Rule이 잘못 판단했다고 생각해 봅니다.

```text
실제 장면 의미
→ warning_red

5일차 Rule 출력
→ NORMAL
```

6일차의 정답 Label은 다음입니다.

```text
warning_red
```

왜냐하면 AI가 배워야 할 것은 **기존 Rule의 실수**가 아니라 **실제 장면의 의미**이기 때문입니다.

오늘의 기본 Class는 다음 두 개입니다.

```text
normal
→ 빨간색 경고 대상이 없는 장면

warning_red
→ 빨간색 경고 대상이 존재하는 장면
```

---

## 오늘의 학습 목표

수업이 끝나면 다음을 할 수 있어야 합니다.

```text
1. 5일차 Rule 방식과 6일차 AI 방식의 차이를 설명한다.
2. normal / warning_red Label을 실제 장면 의미로 정의한다.
3. 연속 Camera Frame을 무작위로 Train/Test에 섞으면 왜 위험한지 설명한다.
4. Session 01 / 02 / 03을 Train / Validation / Test로 구분한다.
5. Raspberry Pi에서 Class별 이미지를 Session 단위로 수집한다.
6. Source Code와 Dataset의 이동 방법을 구분한다.
7. Dataset 폴더 구조와 Class 수를 확인한다.
8. SHA-256으로 Split 사이의 완전 동일 파일 Leakage를 확인한다.
9. 96×96 RGB 입력을 사용하는 Tiny CNN 구조를 설명한다.
10. Epoch / Batch Size / Learning Rate의 기본 의미를 설명한다.
11. Train과 Validation의 역할을 구분하고 Best Validation Model을 저장한다.
12. Test Dataset은 Model 선택이 끝난 뒤 최종 평가에 사용한다.
13. Accuracy와 Confusion Matrix를 함께 읽는다.
14. 한 장의 이미지에서 Class와 Confidence를 확인한다.
15. 실행 결과가 어느 파일의 어느 코드에서 만들어졌는지 역추적한다.
16. 오늘 만든 PyTorch Checkpoint가 7일차 ONNX Export에 어떤 정보로 연결되는지 설명한다.
```

---

## 오늘의 8시간 학습 흐름

| 학습 블록 | 권장 시간 | 무엇을 배우나요? | 끝나면 할 수 있어야 하는 것 |
|---|---:|---|---|
| 1 | 40분 | 5일차 연결 · Label · Session Split | Rule과 AI의 차이, Train/Val/Test 역할을 말할 수 있다 |
| 2 | 80분 | Raspberry Pi Session 촬영 | `normal / warning_red`를 Session별로 안전하게 촬영할 수 있다 |
| 3 | 45분 | PC 전송 · Split · Dataset QA · Leakage | Dataset 구조와 Leakage를 점검할 수 있다 |
| 4 | 55분 | Tiny CNN · 입력 Tensor · Parameter | 작은 CNN이 Image를 2 Class로 분류하는 구조를 설명할 수 있다 |
| 5 | 90분 | Training · Validation · History · Best Model | 학습 결과를 읽고 Validation 기준 Best Model을 고정할 수 있다 |
| 6 | 80분 | Mini Challenge · 오류 분석 · 기본 상태 복원 | 결과 역추적·예측·설정 실험·코드 수정·오류 복구를 스스로 수행할 수 있다 |
| 7 | 55분 | Test · Confusion Matrix · 단일 이미지 Prediction | 고정된 Model을 Test에서 최종 평가할 수 있다 |
| 8 | 35분 | Report · README · Git · 복습 · 7일차 연결 | 오늘 결과를 재현 가능한 상태로 남기고 다음 날로 연결할 수 있다 |

총 480분을 기준으로 구성합니다.

---

## 오늘도 계속 기억할 다섯 단계

1일차부터 사용한 큰 구조는 바뀌지 않습니다.

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

오늘은 다음처럼 대응됩니다.

```text
입력
→ Raspberry Pi Camera로 촬영한 JPG

처리
→ RGB 변환 / Resize / Tensor

판단
→ Tiny CNN
→ normal / warning_red

출력
→ PC 터미널의 Class / Accuracy / Confidence

기록
→ Training History / Test Metrics / Confusion Matrix / Model Checkpoint
```

7일차에는 오늘의 판단 Model을 Raspberry Pi의 실제 Edge Pipeline에 다시 넣습니다.

---

## 오늘 사용할 실제 장비와 데이터

```text
Raspberry Pi
USB Camera
빨간색 카드 또는 5일차에서 사용한 동일한 경고 대상

PC
Python
PyTorch
Torchvision
Pillow
```

오늘 별도의 공개 Dataset을 받는 것이 아니라 **직접 Camera로 수집한 작은 Dataset**을 사용합니다.

기본 수량은 다음과 같습니다.

```text
2 Classes
×
3 Sessions
×
Class당 60장
=
총 360장
```

정확히 360장을 채우는 것보다 다음이 더 중요합니다.

```text
Label이 맞는가?
Session 조건이 구분되는가?
같은 장면을 복사하듯 찍지 않았는가?
심한 Blur / Black Frame이 없는가?
Train / Val / Test 사이에 같은 파일이 섞이지 않았는가?
```

---

## 오늘 배울 기술 스택

| 기술 | 오늘 어디에 사용하나요? |
|---|---|
| Raspberry Pi OS / Linux | Camera Dataset 수집 |
| SSH / SCP | PC↔Raspberry Pi 접속과 Dataset 복사 |
| OpenCV | 3일차부터 사용한 Camera 입력 재사용 |
| Python | Dataset 준비·검증·학습·평가 |
| PyTorch | Tiny CNN 정의와 Training |
| Torchvision `ImageFolder` | 폴더 이름을 Class Label로 읽기 |
| Pillow | 단일 이미지 Prediction |
| CSV | Training History·Test Metrics·Confusion Matrix 저장 |
| SHA-256 | Split 사이의 완전 동일 이미지 확인 |
| Git | Source·설정·Report Version 관리 |

---

## 실습 전에 자신의 환경값 적어두기

```text
Raspberry Pi 사용자 이름 : ____________________
Raspberry Pi Hostname    : ____________________
Raspberry Pi IPv4        : ____________________

Raspberry Pi Project Path: ~/ai_vision/subject13_edge_ai

PC Project Path          : ____________________

Camera /dev/video 장치   : ____________________
OpenCV Camera Index      : ____________________

PC Python                : ____________________
PyTorch Version          : ____________________
Torchvision Version      : ____________________
CUDA 사용 가능 여부      : True / False

Session 01 조건          : ____________________
Session 02 변경 조건     : ____________________
Session 03 변경 조건     : ____________________
```

이 문서에서는 환경에 따라 달라지는 값을 다음처럼 표시합니다.

```text
<RPI_USER>
→ 자신의 Raspberry Pi 사용자 이름

<RPI_IP>
→ 현재 확인한 Raspberry Pi IPv4

<PC_PROJECT_PATH>
→ 자신의 PC에서 subject13_edge_ai가 있는 실제 경로
```

---

## 오늘의 성공 기준

오늘이 끝났을 때 다음 연결이 실제로 성립하면 됩니다.

```text
5일차 Rule Baseline 정상
        ↓
normal / warning_red 의미 확정
        ↓
Session 01 / 02 / 03 촬영
        ↓
PC로 Dataset 이동
        ↓
session01 → train
session02 → val
session03 → test
        ↓
Dataset Summary
        ↓
Exact Duplicate Leakage Check
        ↓
Tiny CNN
        ↓
Training / Validation
        ↓
Best Validation Model
        ↓
Mini Challenge 후 Baseline 복원
        ↓
Test 최종 평가
        ↓
Confusion Matrix
        ↓
Single Image Prediction
        ↓
models/day06_tiny_cnn.pt
        ↓
Report / README / Git
        ↓
7일차 ONNX Export 준비
```

---

# PART A. 5일차 정상 상태에서 시작하기

# 1. 새 프로젝트를 만들지 않습니다

5일차에서 사용한 같은 프로젝트를 이어서 사용합니다.

```text
subject13_edge_ai
```

새로 만들지 않는 것:

```text
새 Project
새 Raspberry Pi .venv
새 Git Repository
새 Camera Input 모듈
```

오늘 다시 사용하는 5일차 이전 파일:

```text
src/config_loader.py
→ 공통 설정과 장치별 Local 설정을 읽는 기존 기능

src/camera_input.py
→ Raspberry Pi Camera Frame 읽기

scripts/camera_index_probe.py
→ 실제 Camera Index 확인

scripts/camera_test.py
→ Camera 단독 점검

scripts/rule_warning_system.py
→ 5일차 Rule Baseline 최종 점검
```

즉, Camera 기능을 다시 작성하지 않습니다.

---

# 2. 5일차 Rule Baseline을 한 번만 다시 확인합니다

PC에서 Raspberry Pi에 접속합니다.

```bash
ssh <RPI_USER>@<RPI_IP>
```

현재 IP를 모른다면 Raspberry Pi에서 직접 다음 명령으로 확인합니다.

```bash
hostname -I
```

`.local` 이름이 교육장 네트워크에서 정상 동작하고 Hostname을 알고 있다면 다음 방식도 사용할 수 있습니다.

```bash
ssh <RPI_USER>@<RPI_HOSTNAME>.local
```

`.local`이 안 된다고 장치가 고장난 것은 아닙니다. 현재 IPv4를 확인해 다시 접속합니다.

프로젝트로 이동합니다.

```bash
cd ~/ai_vision/subject13_edge_ai
```

기존 Raspberry Pi 가상환경을 활성화합니다.

```bash
source .venv/bin/activate
```

확인합니다.

```bash
whoami
hostname
hostname -I
pwd
which python
```

Camera Index를 추측하지 않습니다.

```bash
python -m scripts.camera_index_probe
```

Camera 단독 Test:

```bash
python -m scripts.camera_test
```

5일차 Rule Baseline:

```bash
python -m scripts.rule_warning_system
```

빨간 경고 대상을 넣고 뺐을 때 `NORMAL / WARNING`이 바뀌는지 짧게 확인한 뒤 `Ctrl+C`로 종료합니다.

```text
기존 기능 정상
→ 6일차 Dataset 수집 시작

기존 기능부터 오류
→ 5일차 환경부터 복구
→ Camera / Rule 정상 확인
→ 그 다음 6일차 진행
```

새 기능을 추가하기 전에 기존 기능이 정상임을 확인하면 오류 범위를 크게 줄일 수 있습니다.

---

# 3. Source 최신 상태 확인 방법은 교육장 방식에 맞춥니다

외부 GitHub 사용을 전제로 하지 않습니다.

### 허용된 내부 GitLab / Gitea Remote를 사용하는 경우

```bash
git status
git remote -v
git pull
git log --oneline -8
```

### Raspberry Pi에는 SCP로 Source만 전달한 경우

Raspberry Pi 폴더가 Git Repository가 아닐 수 있습니다. 이 경우 `git pull`을 억지로 실행하지 않습니다.

```bash
ls
ls configs
ls src
ls scripts
```

필요한 파일이 존재하는지 확인합니다.

```text
핵심 목표
→ 5일차에서 정상 실행한 Source와
   6일차에서 사용할 Source가 같은 Version이어야 한다.
```

---

# PART B. AI 문제와 Dataset을 설계하기

# 4. 5일차와 같은 문제를 유지하는 이유

오늘 기본 Class:

```text
normal
warning_red
```

5일차와 동일한 문제를 사용하면 다음 비교가 가능해집니다.

```text
같은 장면
        ↓
┌──────────────────┬──────────────────┐
│ 5일차 Rule       │ 6~7일차 AI       │
├──────────────────┼──────────────────┤
│ HSV              │ CNN Feature      │
│ Red Area         │ Learned Weight   │
│ Threshold        │ Class Probability│
└──────────────────┴──────────────────┘
        ↓
NORMAL / WARNING 비교
```

오늘의 목적은 AI가 Rule보다 무조건 좋다는 것을 증명하는 것이 아닙니다.

```text
Rule이 잘하는 조건
Rule이 실패하는 조건
AI가 잘하는 조건
AI가 실패하는 조건
```

을 같은 문제에서 비교할 수 있는 Baseline을 만드는 것이 목적입니다.

---

# 5. Train / Validation / Test는 왜 세 부분으로 나눌까요?

초보자가 가장 먼저 구분해야 할 역할은 다음입니다.

```text
Train
→ Weight를 실제로 학습하는 데이터

Validation
→ Training 중 어떤 Model 상태가 더 나은지 선택하는 데이터

Test
→ Model 선택이 끝난 뒤 마지막으로 한 번 평가하는 데이터
```

중요한 원칙:

> **Test 결과를 보고 Epoch, Image Size, Learning Rate를 계속 바꾸지 않습니다.**

그렇게 하면 Test도 사실상 Model 선택에 사용한 데이터가 되어 최종 평가의 의미가 약해집니다.

오늘은 다음 순서를 지킵니다.

```text
Train / Validation으로 Baseline 결정
        ↓
Mini Challenge도 Train / Validation 중심으로 수행
        ↓
기본 설정 복원
        ↓
Official Baseline 고정
        ↓
Test 최종 평가
```

---

# 6. 연속 Frame을 무작위로 섞지 않는 이유

Camera가 0.2초 간격으로 연속 촬영하면 장면은 매우 비슷할 수 있습니다.

```text
10:00:00.000
10:00:00.200
10:00:00.400
10:00:00.600
```

이 이미지들을 무작위로 나누면 거의 같은 장면이 Train과 Test에 동시에 들어갈 수 있습니다.

```text
거의 같은 Frame A
→ Train

거의 같은 Frame B
→ Test
```

이 경우 Test Accuracy가 높아도 새로운 조건에서 잘 동작한다고 보기 어렵습니다.

그래서 **촬영 Session 자체를 먼저 분리**합니다.

```text
Session 01
→ Train

Session 02
→ Validation

Session 03
→ Test
```

SHA-256 검사는 완전히 같은 파일을 찾는 보조 점검입니다.  
거의 비슷하지만 파일 내용이 조금 다른 연속 Frame은 Hash가 달라질 수 있으므로 Session 분리가 더 중요합니다.

---

# 7. 세 Session의 촬영 조건을 먼저 적습니다

## Session 01 — Train

```text
배경 A
조명 A
기본 Camera 위치

촬영 중 변화
→ 카드 위치
→ 카드 각도
→ 카드 거리
→ 화면에서 차지하는 크기
```

## Session 02 — Validation

Session 01과 완전히 같게 복사하지 않습니다.

```text
예:
조명 밝기 약간 변경
또는
Camera와 대상 거리 약간 변경
```

한 번에 너무 많은 조건을 바꾸지 않습니다.

## Session 03 — Test

Train과 구분되는 조건을 하나 정도 사용합니다.

```text
예:
배경 B
또는
Camera 위치 변경
```

Test Session은 이후 Training에 섞지 않습니다.

---

# 8. 권장 촬영 수량

기본 실습:

```text
Session 01
normal       60
warning_red  60

Session 02
normal       60
warning_red  60

Session 03
normal       60
warning_red  60
```

총 360장입니다.

60이라는 숫자 자체가 품질을 보장하지 않습니다.

```text
60장의 거의 같은 사진
<
위치·거리·각도에 변화가 있는 60장
```

---

# 9. 6일차 설정값을 기존 `settings.yaml`에 추가합니다

기존 1~5일차 설정을 삭제하지 않습니다. 아래 항목을 추가합니다.

```yaml
# Day 06 - Tiny CNN Baseline
class_names:
  - normal
  - warning_red

capture_count: 60
capture_interval_sec: 0.20

session_root: data/sessions
dataset_dir: data/dataset

ai_image_size: 96
ai_batch_size: 32
ai_epochs: 15
ai_learning_rate: 0.001

ai_model_path: models/day06_tiny_cnn.pt
training_history_path: reports/day06_training_history.csv
```

각 Key의 역할:

```text
class_names
→ AI가 구분할 Class 이름과 순서

capture_count
→ 한 번 실행할 때 촬영할 이미지 수

capture_interval_sec
→ Camera Frame 저장 간격

session_root
→ 원본 Session 저장 폴더

dataset_dir
→ Train / Val / Test Dataset 폴더

ai_image_size
→ 학습 전에 Resize할 이미지 크기

ai_batch_size
→ 한 번의 Weight Update에 묶어 처리할 이미지 수

ai_epochs
→ Train Dataset 전체를 반복해서 학습할 횟수

ai_learning_rate
→ Weight를 업데이트하는 기본 크기

ai_model_path
→ Validation 기준으로 가장 좋은 PyTorch Checkpoint 저장 위치

training_history_path
→ Epoch별 Train / Validation 결과 CSV
```

`class_names`의 순서는 매우 중요합니다.

```text
Class 0 → normal
Class 1 → warning_red
```

7일차 ONNX Metadata도 이 Class 순서를 이어서 사용합니다.

---

# PART C. Raspberry Pi에서 Session Dataset 수집하기

# 10. Session 촬영 Program을 만들기 전에 역할을 확인합니다

새 파일:

```text
scripts/collect_session.py
```

이 파일은 3일차의 `CameraInput`을 다시 사용합니다.

```text
새로 만들지 않는 것
→ Camera를 여는 Class
→ Frame 저장 함수

다시 사용하는 것
→ src/camera_input.py
→ CameraInput
→ save_frame()

오늘 새로 만드는 것
→ 어느 Session인가?
→ 어느 Class인가?
→ 몇 장 촬영하는가?
→ Metadata를 어떻게 남기는가?
```

또 하나의 안전장치를 넣습니다.

같은 `session/class` 폴더에 이미 JPG가 있다면 자동으로 추가 촬영하지 않고 오류를 냅니다.

왜냐하면 실수로 같은 명령을 두 번 실행해:

```text
원래 60장
→ 다시 실행
→ 120장
```

이 되는 것을 막기 위해서입니다.

---

## `collect_session.py` 의사코드

```text
--session과 --class-name을 입력받는다
        ↓
settings.yaml을 읽는다
        ↓
class-name이 허용된 Class인지 확인한다
        ↓
Session / Class 저장 폴더를 계산한다
        ↓
이미 JPG가 있으면
→ 중복 촬영을 막기 위해 오류를 발생시킨다
        ↓
Camera를 기존 CameraInput으로 연다
        ↓
capture_count와 interval을 읽는다
        ↓
3초 후 촬영 시작
        ↓
설정된 횟수만큼 반복
        ↓
Frame 읽기
        ↓
JPG 저장
        ↓
Timestamp / Session / Class / Path / Width / Height
Metadata CSV에 기록
        ↓
설정된 간격만큼 대기
        ↓
반복 종료 후 Camera Resource 정리
```

## 코드

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
        "--session",
        required=True,
        help="예: session01",
    )

    parser.add_argument(
        "--class-name",
        required=True,
        help="예: normal",
    )

    return parser.parse_args()


def append_metadata(
    metadata_path: Path,
    timestamp: str,
    session: str,
    class_name: str,
    image_path: Path,
    width: int,
    height: int,
):
    is_new = not metadata_path.exists()

    with metadata_path.open(
        "a",
        newline="",
        encoding="utf-8",
    ) as file:
        writer = csv.writer(file)

        if is_new:
            writer.writerow(
                [
                    "timestamp",
                    "session",
                    "class_name",
                    "image_path",
                    "width",
                    "height",
                ]
            )

        writer.writerow(
            [
                timestamp,
                session,
                class_name,
                str(image_path),
                width,
                height,
            ]
        )


def main():
    args = parse_args()

    config = load_config(
        "configs/settings.yaml"
    )

    class_names = list(
        config["class_names"]
    )

    if args.class_name not in class_names:
        raise ValueError(
            f"지원하지 않는 Class: "
            f"{args.class_name}\n"
            f"가능한 Class: {class_names}"
        )

    root = Path(
        config["session_root"]
    )

    output_dir = (
        root
        / args.session
        / args.class_name
    )

    if output_dir.exists():
        existing_images = list(
            output_dir.glob("*.jpg")
        )

        if existing_images:
            raise RuntimeError(
                "이미 촬영된 이미지가 있습니다: "
                f"{output_dir}\n"
                "같은 Class를 다시 촬영하려면 "
                "기존 Session을 먼저 백업/정리하거나 "
                "새 Session 이름을 사용하세요."
            )

    output_dir.mkdir(
        parents=True,
        exist_ok=True,
    )

    metadata_path = (
        root
        / args.session
        / "metadata.csv"
    )

    camera = CameraInput(
        index=int(config["camera_index"]),
        width=int(config["image_width"]),
        height=int(config["image_height"]),
    )

    count = int(
        config["capture_count"]
    )

    interval = float(
        config["capture_interval_sec"]
    )

    try:
        print("=== Session Capture ===")
        print(f"Session : {args.session}")
        print(f"Class   : {args.class_name}")
        print(f"Count   : {count}")
        print()
        print("3초 후 촬영을 시작합니다.")

        time.sleep(3.0)

        for index in range(1, count + 1):
            frame = camera.read()

            height, width = frame.shape[:2]

            path = save_frame(
                frame,
                str(output_dir),
                prefix=args.class_name,
            )

            timestamp = (
                datetime.now().isoformat(
                    timespec="milliseconds"
                )
            )

            append_metadata(
                metadata_path=metadata_path,
                timestamp=timestamp,
                session=args.session,
                class_name=args.class_name,
                image_path=path,
                width=width,
                height=height,
            )

            print(
                f"{index:03d}/{count} "
                f"{path.name}"
            )

            time.sleep(interval)

    finally:
        camera.release()


if __name__ == "__main__":
    main()

```

---

# 11. 실행 전에 촬영 파일이 어떻게 연결되는지 봅니다

```text
configs/settings.yaml
→ class_names
→ capture_count
→ capture_interval_sec
→ session_root

configs/device.local.yaml
→ 현재 Raspberry Pi의 camera_index

        ↓

src/config_loader.py
→ 설정값 읽기

        ↓

scripts/collect_session.py
        ↓
src/camera_input.py
→ CameraInput.read()
→ save_frame()

        ↓

data/sessions/
├── session01/
├── session02/
└── session03/

각 Session
→ JPG
→ metadata.csv
```

`camera_index`는 실제 장치 값이므로 추측하지 않고 `camera_index_probe.py`에서 확인한 값을 사용합니다.

---

# 12. Guided Lab — Session 01 Train 데이터 촬영

먼저 `normal`입니다.

촬영 조건:

```text
빨간색 경고 대상이 없음
```

실행 전에 확인합니다.

```bash
grep -E   "capture_count|capture_interval_sec|session_root"   configs/settings.yaml
```

실행:

```bash
python -m scripts.collect_session   --session session01   --class-name normal
```

예상 형태:

```text
=== Session Capture ===
Session : session01
Class   : normal
Count   : 60

3초 후 촬영을 시작합니다.
001/60 normal_....jpg
002/60 normal_....jpg
...
060/60 normal_....jpg
```

이번에는 `warning_red`입니다.

```bash
python -m scripts.collect_session   --session session01   --class-name warning_red
```

촬영 중 카드의 위치·각도·거리·크기를 조금씩 바꿉니다.

---

## 실행 결과를 코드에서 다시 찾아보기

다음 한 줄이 보였다고 가정합니다.

```text
017/60 warning_red_....jpg
```

직접 파일을 열어 찾습니다.

```text
60
→ settings.yaml의 capture_count

warning_red
→ 실행 인자 --class-name
→ args.class_name

JPG 파일
→ src/camera_input.py의 save_frame()

017
→ collect_session.py의 for index in range(...)

저장 폴더
→ session_root / session / class_name

metadata.csv
→ append_metadata()
```

결과 한 줄이 한 곳에서 모두 만들어지는 것이 아니라, Config·실행 인자·기존 Camera 모듈이 연결되어 만들어진다는 점을 확인합니다.

---

# 13. Session 01 파일과 Metadata를 확인합니다

```bash
find data/sessions/session01/normal   -type f -name "*.jpg" | wc -l
```

```bash
find data/sessions/session01/warning_red   -type f -name "*.jpg" | wc -l
```

각각 기본값은 약 60장입니다.

Metadata:

```bash
head -n 5 data/sessions/session01/metadata.csv
```

예상 Header:

```csv
timestamp,session,class_name,image_path,width,height
```

촬영 120장이 정상 완료되었다면 Metadata에는 Header 1줄과 Sample 120줄이 기록됩니다.

```bash
wc -l data/sessions/session01/metadata.csv
```

기본적으로:

```text
1 Header
+
120 Samples
=
121 lines
```

중간에 촬영을 취소했다면 수량이 다를 수 있습니다. 숫자를 억지로 맞추지 말고 원인을 기록합니다.

---

# 14. Session 02 Validation 촬영

Session 01과 비교하여 조건 하나를 바꿉니다.

예:

```text
변경
→ Camera와 대상 거리

고정
→ 기본 배경
→ 같은 빨간 카드
```

`normal`:

```bash
python -m scripts.collect_session   --session session02   --class-name normal
```

`warning_red`:

```bash
python -m scripts.collect_session   --session session02   --class-name warning_red
```

Session 01 이미지를 복사하지 않습니다. 새로 촬영합니다.

---

# 15. Session 03 Test 촬영

Test는 최종 평가용 Session입니다.

예:

```text
Session 03
→ 배경을 변경
→ 나머지 조건은 가능한 한 기록
```

`normal`:

```bash
python -m scripts.collect_session   --session session03   --class-name normal
```

`warning_red`:

```bash
python -m scripts.collect_session   --session session03   --class-name warning_red
```

이 Session을 나중에 Train으로 옮기지 않습니다.

---

# 16. 촬영 중 자주 만나는 오류

## 오류 A — SSH 접속 실패

```text
ssh: connect to host ... port 22 ...
```

확인 순서:

```text
Raspberry Pi 전원
→ 같은 Network인지 확인
→ hostname -I로 현재 IPv4 확인
→ <RPI_IP> 갱신
→ SSH 재실행
```

IP가 예전과 달라졌을 수 있습니다.

## 오류 B — Camera가 열리지 않음

```text
Camera open failed
또는 Frame read failed
```

확인 순서:

```text
ls /dev/video*
→ 실제 Video 장치 확인
→ camera_index_probe.py 실행
→ configs/device.local.yaml 확인
→ camera_test.py 재실행
→ collect_session.py 재실행
```

Camera 번호를 `0`으로 단정하지 않습니다.

## 오류 C — 이미 촬영된 이미지가 있다는 오류

이번 최종 코드에서는 같은 Session/Class에 기존 JPG가 있으면 중복 촬영을 막습니다.

```text
오류
→ 기존 파일이 있는지 확인
→ 실수로 재실행한 것인지 확인
→ 기존 Session을 유지할지 결정
→ 필요하면 전체 Session을 백업 후 정리
→ 다시 촬영
```

Class 하나만 무작정 지우면 기존 `metadata.csv`와 실제 파일이 어긋날 수 있습니다. 초보 실습에서는 재촬영이 필요하면 **Session 전체를 백업한 뒤 두 Class를 함께 다시 촬영**하는 편이 안전합니다.

---

# 17. Raspberry Pi에서 촬영 품질을 먼저 확인합니다

폴더 구조:

```bash
find data/sessions   -maxdepth 3   -type d | sort
```

예상:

```text
data/sessions/
├── session01/
│   ├── normal/
│   ├── warning_red/
│   └── metadata.csv
├── session02/
│   ├── normal/
│   ├── warning_red/
│   └── metadata.csv
└── session03/
    ├── normal/
    ├── warning_red/
    └── metadata.csv
```

파일 개수만 보고 끝내지 않습니다.

직접 몇 장을 확인합니다.

```text
normal 폴더
→ 빨간 경고 대상이 잘못 들어간 사진이 없는가?

warning_red 폴더
→ 빨간 대상이 실제로 보이는가?

전체
→ 너무 어둡거나 완전히 검은 Frame은 없는가?
→ 심하게 흔들린 Frame은 없는가?
→ 모든 사진이 거의 똑같지는 않은가?
```

---

# PART D. Source와 Dataset을 PC로 옮기기

# 18. Source와 Dataset은 같은 방식으로 관리하지 않습니다

```text
Source Code / Config / Report
→ Git Version 관리

Raw Dataset / Captured Image
→ Git 제외
→ SCP 또는 교육장 허용 파일 전송

장치별 device.local.yaml
→ Git 제외
```

5일차까지의 `.gitignore`에서 최소 다음 항목이 있어야 합니다.

```gitignore
data/
*.pt
.venv/
logs/
configs/device.local.yaml
```

---

# 19. Raspberry Pi에서 Source를 저장하는 방법

### 내부 Remote를 사용하는 경우

```bash
git status
git diff
git add configs/settings.yaml scripts/collect_session.py
git commit -m "feat: add session based image collection"
git push
```

### Raspberry Pi가 Git Repository가 아닌 SCP 방식인 경우

Raspberry Pi에서 `git commit`을 억지로 하지 않습니다.

PC의 프로젝트에서 필요한 Source를 가져옵니다.

```bash
scp <RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/scripts/collect_session.py   scripts/

scp <RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/configs/settings.yaml   configs/
```

PC Local Git에서 Version을 관리합니다.

---

# 20. Dataset을 PC로 복사합니다

Raspberry Pi SSH 세션에서 나옵니다.

```bash
exit
```

PC 프로젝트로 이동합니다.

```bash
cd <PC_PROJECT_PATH>
```

실제 경로가 `~/ai_vision/subject13_edge_ai`라면 다음처럼 사용할 수 있습니다.

```bash
cd ~/ai_vision/subject13_edge_ai
```

PC에 `data/`를 준비합니다.

```bash
mkdir -p data
```

Raspberry Pi Dataset을 복사합니다.

```bash
scp -r   <RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/data/sessions   ./data/
```

확인:

```bash
find data/sessions   -maxdepth 3   -type d | sort
```

SCP 실패 시 확인 순서:

```text
Raspberry Pi IP가 현재 값인가?
→ SSH 접속 자체는 되는가?
→ 원격 경로가 실제로 존재하는가?
→ PC에서 명령을 실행하고 있는가?
→ 대상 PC 폴더에 쓰기 권한이 있는가?
→ 수정 후 SCP 재실행
```

---

# 21. PC Python 환경을 확인합니다

1일차에 만든 PC `.venv`를 다시 사용합니다.

```bash
source .venv/bin/activate
```

확인:

```bash
which python
python --version
```

PyTorch / Torchvision / Pillow:

```bash
python -c "import torch, torchvision, PIL; print('torch:', torch.__version__); print('torchvision:', torchvision.__version__); print('CUDA:', torch.cuda.is_available())"
```

`CUDA: True`가 반드시 정답은 아닙니다.

```text
CUDA True
→ GPU 사용 가능

CUDA False
→ CPU Training
→ 오늘의 작은 Tiny CNN은 CPU에서도 실습 가능
```

Package가 없다면 교육장 환경에 맞는 설치 방법을 사용합니다.

```text
인터넷 허용 환경
→ 검증된 설치 명령 또는 공식 설치 방식 사용

폐쇄망
→ 교육장 Offline Wheel / 내부 Package 저장소 사용
```

인터넷 연결을 전제로 무조건 설치하지 않습니다.

현재 PC 환경을 기록합니다.

```bash
python -m pip freeze   > reports/day06_pc_environment.txt
```

이 파일은 PC Training 환경 기록입니다. Raspberry Pi용 Package 목록과 같을 필요가 없습니다.

---

# PART E. Session을 Train / Val / Test Dataset으로 만들기

# 22. Dataset Split을 만들기 전에 고정 규칙을 다시 확인합니다

오늘은 Random Split을 사용하지 않습니다.

```text
session01 → train
session02 → val
session03 → test
```

목표 구조:

```text
data/dataset/
├── train/
│   ├── normal/
│   └── warning_red/
├── val/
│   ├── normal/
│   └── warning_red/
└── test/
    ├── normal/
    └── warning_red/
```

`torchvision.datasets.ImageFolder`는 **폴더 이름을 Class 이름으로 읽습니다.**

따라서 다음 오타는 단순한 폴더 이름 실수가 아니라 Label 문제입니다.

```text
warning_red
≠
warning-red
≠
warning
```

---

# 23. Dataset Split 생성 Program 만들기

새 파일:

```text
scripts/build_dataset_split.py
```

## 이 파일은 왜 필요할까요?

원본 Session 구조를 직접 학습 코드에서 복잡하게 읽지 않고, 학습에서 널리 쓰는 `train/val/test/class` 구조로 변환하기 위해 만듭니다.

원본 Session은 그대로 두고 `copy2()`로 Dataset을 만듭니다.

```text
data/sessions
→ 원본 보존

data/dataset
→ 학습용 복사본
```

기존 `data/dataset`이 있으면 다시 만드는 과정에서 삭제 후 재생성됩니다. 따라서 `data/dataset` 안에 별도의 수작업 파일을 보관하지 않습니다.

## 의사코드

```text
settings.yaml을 읽는다
        ↓
session_root와 dataset_dir을 읽는다
        ↓
Class 이름을 읽는다
        ↓
기존 dataset_dir이 있으면 제거한다
        ↓
session01 / session02 / session03 반복
        ↓
train / val / test로 대응시킨다
        ↓
각 Class 폴더가 실제로 있는지 확인한다
        ↓
JPG가 한 장 이상 있는지 확인한다
        ↓
대상 train/val/test Class 폴더를 만든다
        ↓
이미지를 복사한다
        ↓
Session / Split / Class / Count를 출력한다
        ↓
전체 이미지 수를 출력한다
```

## 코드

```python

import shutil
from pathlib import Path

from src.config_loader import load_config


SESSION_TO_SPLIT = {
    "session01": "train",
    "session02": "val",
    "session03": "test",
}


def main():
    config = load_config(
        "configs/settings.yaml"
    )

    source_root = Path(
        config["session_root"]
    )

    target_root = Path(
        config["dataset_dir"]
    )

    class_names = list(
        config["class_names"]
    )

    if target_root.exists():
        print(
            f"기존 Dataset 폴더 삭제: "
            f"{target_root}"
        )

        shutil.rmtree(target_root)

    total = 0

    for session, split in (
        SESSION_TO_SPLIT.items()
    ):
        for class_name in class_names:
            source_dir = (
                source_root
                / session
                / class_name
            )

            if not source_dir.exists():
                raise FileNotFoundError(
                    f"폴더가 없습니다: "
                    f"{source_dir}"
                )

            target_dir = (
                target_root
                / split
                / class_name
            )

            target_dir.mkdir(
                parents=True,
                exist_ok=True,
            )

            images = sorted(
                source_dir.glob("*.jpg")
            )

            if not images:
                raise RuntimeError(
                    f"이미지가 없습니다: {source_dir}"
                )

            for image_path in images:
                destination = (
                    target_dir
                    / image_path.name
                )

                shutil.copy2(
                    image_path,
                    destination,
                )

                total += 1

            print(
                f"{session:9s} "
                f"→ {split:5s} "
                f"{class_name:12s} "
                f"{len(images):3d}"
            )

    print()
    print(f"Total Images: {total}")
    print(f"Dataset     : {target_root}")


if __name__ == "__main__":
    main()

```

---

# 24. Guided Lab — Dataset Split 생성

실행 전에 예상합니다.

기본 촬영이 모두 60장이라면:

```text
train
normal 60 + warning_red 60
→ 120

val
normal 60 + warning_red 60
→ 120

test
normal 60 + warning_red 60
→ 120

Grand Total
→ 360
```

실행:

```bash
python -m scripts.build_dataset_split
```

예상 형태:

```text
session01 → train normal         60
session01 → train warning_red    60
session02 → val   normal         60
session02 → val   warning_red    60
session03 → test  normal         60
session03 → test  warning_red    60

Total Images: 360
Dataset     : data/dataset
```

실제 촬영 수가 다르면 자신의 실제 숫자가 출력됩니다.

---

# 25. Dataset Summary Program 만들기

새 파일:

```text
scripts/dataset_summary.py
```

## 이 파일은 왜 필요할까요?

학습 전에 다음 질문을 자동으로 확인하기 위해 만듭니다.

```text
train 폴더가 실제로 있는가?
val / test 폴더가 있는가?
각 Class 폴더가 있는가?
각 Split에 몇 장이 있는가?
한 Class가 실수로 비어 있지 않은가?
```

## 의사코드

```text
dataset_dir과 class_names를 읽는다
        ↓
Dataset Root가 없으면 오류
        ↓
train / val / test 반복
        ↓
각 Class 폴더 존재 확인
        ↓
JPG 수 계산
        ↓
Class별 Count 출력
        ↓
Split Total 계산
        ↓
Grand Total 출력
```

## 코드

```python

from pathlib import Path

from src.config_loader import load_config


def main():
    config = load_config(
        "configs/settings.yaml"
    )

    root = Path(
        config["dataset_dir"]
    )

    class_names = list(
        config["class_names"]
    )

    if not root.exists():
        raise FileNotFoundError(
            f"Dataset 폴더가 없습니다: {root}"
        )

    print("=== Dataset Summary ===")

    grand_total = 0

    for split in [
        "train",
        "val",
        "test",
    ]:
        split_total = 0

        print()
        print(f"[{split}]")

        for class_name in class_names:
            directory = (
                root
                / split
                / class_name
            )

            if not directory.exists():
                raise FileNotFoundError(
                    f"Class 폴더가 없습니다: {directory}"
                )

            count = len(
                list(
                    directory.glob("*.jpg")
                )
            )

            print(
                f"{class_name:12s}: "
                f"{count}"
            )

            split_total += count

        print(
            f"split total : {split_total}"
        )

        grand_total += split_total

    print()
    print(f"Grand Total : {grand_total}")


if __name__ == "__main__":
    main()

```

실행:

```bash
python -m scripts.dataset_summary
```

예상 형태:

```text
=== Dataset Summary ===

[train]
normal      : 60
warning_red : 60
split total : 120

[val]
normal      : 60
warning_red : 60
split total : 120

[test]
normal      : 60
warning_red : 60
split total : 120

Grand Total : 360
```

---

# 26. 완전히 같은 파일 Leakage를 확인하는 Program 만들기

새 파일:

```text
scripts/check_split_leakage.py
```

## 이 파일은 왜 필요할까요?

Session을 분리했더라도 복사 실수로 똑같은 JPG가 Train과 Test에 들어갈 수 있습니다.

파일 내용을 SHA-256으로 계산하여 **완전히 동일한 파일**을 찾습니다.

이 검사는 다음을 보장하지는 않습니다.

```text
거의 비슷한 연속 Frame
→ Hash는 다를 수 있음
```

그래서 Session 분리와 육안 확인을 함께 사용합니다.

## 의사코드

```text
Dataset Root를 읽는다
        ↓
train / val / test의 모든 JPG를 찾는다
        ↓
각 파일의 SHA-256 Hash를 계산한다
        ↓
Split별 Hash 목록을 만든다
        ↓
train vs val
train vs test
val vs test
공통 Hash를 찾는다
        ↓
공통 Hash가 있으면 FAIL
        ↓
없으면 NO EXACT DUPLICATES
```

## 코드

```python

import hashlib
from pathlib import Path

from src.config_loader import load_config


def file_hash(path: Path) -> str:
    sha = hashlib.sha256()

    with path.open("rb") as file:
        while True:
            block = file.read(1024 * 1024)

            if not block:
                break

            sha.update(block)

    return sha.hexdigest()


def main():
    config = load_config(
        "configs/settings.yaml"
    )

    root = Path(
        config["dataset_dir"]
    )

    split_hashes = {}

    for split in [
        "train",
        "val",
        "test",
    ]:
        hashes = {}

        for path in (
            root / split
        ).rglob("*.jpg"):
            digest = file_hash(path)

            hashes.setdefault(
                digest,
                [],
            ).append(path)

        split_hashes[split] = hashes

    pairs = [
        ("train", "val"),
        ("train", "test"),
        ("val", "test"),
    ]

    found = False

    for left, right in pairs:
        common = (
            set(split_hashes[left])
            & set(split_hashes[right])
        )

        print(
            f"{left} vs {right}: "
            f"{len(common)} duplicate hash"
        )

        if common:
            found = True

            for digest in common:
                print(
                    "  ",
                    split_hashes[left][digest],
                )
                print(
                    "  ",
                    split_hashes[right][digest],
                )

    print()

    if found:
        print("RESULT: LEAKAGE CHECK FAIL")
    else:
        print("RESULT: NO EXACT DUPLICATES")


if __name__ == "__main__":
    main()

```

실행:

```bash
python -m scripts.check_split_leakage
```

정상 예:

```text
train vs val: 0 duplicate hash
train vs test: 0 duplicate hash
val vs test: 0 duplicate hash

RESULT: NO EXACT DUPLICATES
```

---

# 27. Dataset 결과를 코드에서 역추적합니다

다음 출력이 보였다고 가정합니다.

```text
session02 → val warning_red 60
```

직접 찾습니다.

```text
session02 → val
→ build_dataset_split.py의 SESSION_TO_SPLIT

warning_red
→ settings.yaml의 class_names
→ source_dir / target_dir 구성

60
→ source_dir.glob("*.jpg")
→ len(images)

실제 복사
→ shutil.copy2()
```

다음 출력:

```text
train vs test: 0 duplicate hash
```

위치는:

```text
check_split_leakage.py
→ file_hash()
→ SHA-256
→ set(...) & set(...)
→ 공통 Hash 개수 출력
```

---

# 28. Dataset 오류를 읽는 기본 순서

```text
오류 발생
        ↓
마지막 오류 줄 확인
        ↓
어떤 경로 / Class 이름인지 확인
        ↓
settings.yaml 확인
        ↓
data/sessions 원본 확인
        ↓
build_dataset_split 재실행
        ↓
dataset_summary 재실행
        ↓
leakage check 재실행
```

학습부터 다시 실행하기 전에 Dataset 구조가 먼저 정상이어야 합니다.

---

# PART F. Tiny CNN 구조 이해하기

# 29. 오늘은 왜 작은 CNN을 사용할까요?

오늘의 목표는 큰 Model을 만드는 것이 아닙니다.

```text
작은 Dataset
→ 작은 CNN
→ Training
→ Validation
→ Test
→ Checkpoint
→ 7일차 ONNX
```

이 전체 수명주기를 하루 안에 이해하는 것이 목표입니다.

입력:

```text
96 × 96 RGB
```

구조:

```text
Image
  ↓
Conv 3→16
  ↓
ReLU
  ↓
MaxPool
  ↓
Conv 16→32
  ↓
ReLU
  ↓
MaxPool
  ↓
Conv 32→64
  ↓
ReLU
  ↓
MaxPool
  ↓
Adaptive Average Pool
  ↓
64 Features
  ↓
Linear
  ↓
2 Logits
  ↓
normal / warning_red
```

---

# 30. CNN 용어를 아주 짧게 이해합니다

```text
Conv
→ 작은 Filter를 이동시키며 이미지의 패턴을 학습

ReLU
→ 음수 값을 0으로 바꾸는 간단한 비선형 함수

MaxPool
→ 공간 크기를 줄이면서 강한 특징을 남김

AdaptiveAvgPool2d(1,1)
→ 마지막 Feature Map을 Channel당 값 하나로 요약

Linear
→ 최종 Class 점수를 만듦

Logit
→ Softmax 이전의 원시 Class 점수

Softmax
→ Logit을 Class별 확률처럼 해석할 수 있는 값으로 변환
```

CNN이 실제로 무엇을 특징으로 선택했는지는 5일차 Rule처럼 사람이 `HSV`라고 직접 지정하지 않습니다.

---

# 31. Tiny CNN 파일 만들기

새 파일:

```text
src/tiny_cnn.py
```

## 이 파일은 왜 필요할까요?

Training, Test, 단일 이미지 Prediction, 7일차 ONNX Export가 **같은 Model 구조**를 사용해야 하기 때문입니다.

Model 구조를 여러 실행 파일에 복사하지 않고 하나의 파일에 둡니다.

```text
src/tiny_cnn.py
→ Model 구조

scripts/train_tiny_cnn.py
→ 학습

scripts/evaluate_tiny_cnn.py
→ 평가

scripts/predict_tiny_cnn.py
→ 한 장 예측

7일차 export_onnx.py
→ 같은 TinyCNN 재사용
```

## 의사코드

```text
TinyCNN Class를 만든다
        ↓
Feature Extractor를 만든다
        ↓
Conv / ReLU / Pool을 3번 구성한다
        ↓
Adaptive Average Pool로 1×1로 요약한다
        ↓
64 Feature를 num_classes 개수로 바꾸는 Linear Layer를 만든다
        ↓
forward()에서
Feature 추출
→ Flatten
→ Linear
→ Logit 반환
```

## 코드

```python

import torch.nn as nn


class TinyCNN(nn.Module):
    def __init__(
        self,
        num_classes: int,
    ):
        super().__init__()

        self.features = nn.Sequential(
            nn.Conv2d(
                3,
                16,
                kernel_size=3,
                padding=1,
            ),
            nn.ReLU(),
            nn.MaxPool2d(2),

            nn.Conv2d(
                16,
                32,
                kernel_size=3,
                padding=1,
            ),
            nn.ReLU(),
            nn.MaxPool2d(2),

            nn.Conv2d(
                32,
                64,
                kernel_size=3,
                padding=1,
            ),
            nn.ReLU(),
            nn.MaxPool2d(2),

            nn.AdaptiveAvgPool2d(
                (1, 1)
            ),
        )

        self.classifier = nn.Linear(
            64,
            num_classes,
        )

    def forward(self, x):
        x = self.features(x)
        x = x.flatten(1)
        x = self.classifier(x)

        return x

```

---

# 32. Parameter 수를 확인하는 Program 만들기

새 파일:

```text
scripts/model_summary.py
```

## 이 파일은 왜 필요할까요?

Model을 학습하기 전에 **우리가 만든 Model의 Class 수와 Parameter 수가 예상과 맞는지** 확인합니다.

## 의사코드

```text
settings.yaml을 읽는다
        ↓
class_names를 가져온다
        ↓
Class 수만큼 출력 Node를 가진 TinyCNN을 만든다
        ↓
모든 Parameter 개수를 더한다
        ↓
학습 가능한 Parameter 개수를 더한다
        ↓
Class / Parameter / Trainable을 출력한다
```

## 코드

```python

from src.config_loader import load_config
from src.tiny_cnn import TinyCNN


def main():
    config = load_config(
        "configs/settings.yaml"
    )

    class_names = list(
        config["class_names"]
    )

    model = TinyCNN(
        num_classes=len(class_names)
    )

    total = sum(
        parameter.numel()
        for parameter in model.parameters()
    )

    trainable = sum(
        parameter.numel()
        for parameter in model.parameters()
        if parameter.requires_grad
    )

    print("=== Tiny CNN ===")
    print(f"Classes    : {class_names}")
    print(f"Parameters : {total:,}")
    print(f"Trainable  : {trainable:,}")


if __name__ == "__main__":
    main()

```

실행:

```bash
python -m scripts.model_summary
```

기본 2 Class라면 Parameter 수는 다음 값이 예상됩니다.

```text
23,714
```

왜 이 값이 나오는지 모두 암산할 필요는 없습니다.  
중요한 것은 Class 수나 Model 구조가 바뀌면 Parameter 수가 달라질 수 있다는 것입니다.

---

# 33. Training 전에 Image가 Tensor로 바뀌는 흐름을 이해합니다

오늘 Training 입력:

```text
JPG
  ↓
PIL Image
  ↓
RGB
  ↓
Resize 96 × 96
  ↓
ToTensor()
  ↓
Shape [3, 96, 96]
  ↓
값 범위 대략 0.0 ~ 1.0
  ↓
Batch
  ↓
[Batch, 3, 96, 96]
```

복잡한 Augmentation을 오늘 기본 Baseline에 추가하지 않습니다.

7일차 ONNX에서도 **같은 Input Size와 같은 기본 값 범위**를 재현해야 하기 때문입니다.

---

# 34. Epoch / Batch Size / Learning Rate를 구분합니다

```text
Epoch
→ Train Dataset 전체를 몇 번 반복할 것인가?

Batch Size
→ 한 번의 계산에 몇 장을 묶을 것인가?

Learning Rate
→ Weight를 어느 정도 크기로 업데이트할 것인가?
```

기본값:

```yaml
ai_batch_size: 32
ai_epochs: 15
ai_learning_rate: 0.001
```

이 값들은 절대적인 정답이 아닙니다. 오늘은 Baseline 실습을 위한 시작값입니다.

---

# PART G. Training과 Validation

# 35. Training Program을 만들기 전에 전체 연결을 봅니다

새 파일:

```text
scripts/train_tiny_cnn.py
```

이 파일은 여러 기능을 연결하는 오늘의 핵심 실행 Program입니다.

```text
configs/settings.yaml
        ↓
data/dataset/train
data/dataset/val
        ↓
ImageFolder
        ↓
Resize / ToTensor
        ↓
DataLoader
        ↓
src/tiny_cnn.py
        ↓
CrossEntropyLoss
        ↓
Adam Optimizer
        ↓
Epoch 반복
        ↓
Train Loss / Accuracy
Val Loss / Accuracy
        ↓
reports/day06_training_history.csv
        ↓
Best Validation Accuracy 갱신 시
models/day06_tiny_cnn.pt 저장
```

Test 폴더는 이 Program에서 사용하지 않습니다.

---

# 36. Training Program 만들기

## 이 파일은 왜 필요할까요?

단순히 `model.fit()` 같은 한 줄로 숨기지 않고, 초보자가 다음 흐름을 직접 볼 수 있도록 작성합니다.

```text
Batch 읽기
→ Forward
→ Loss
→ Backward
→ Optimizer Step
→ Accuracy 누적
→ Validation
→ Best Model 저장
```

## 의사코드

```text
Random Seed를 고정한다
        ↓
settings.yaml을 읽는다
        ↓
Dataset / Image Size / Batch / Epoch / LR / 저장 경로를 읽는다
        ↓
Resize + ToTensor 전처리를 만든다
        ↓
Train ImageFolder와 Validation ImageFolder를 만든다
        ↓
Config Class와 Dataset Class 순서가 같은지 확인한다
        ↓
Train DataLoader는 shuffle=True
Validation DataLoader는 shuffle=False
        ↓
CUDA가 가능하면 cuda, 아니면 cpu 선택
        ↓
TinyCNN 생성
        ↓
Loss = CrossEntropyLoss
Optimizer = Adam
        ↓
각 Epoch 반복
        ↓
Train
→ gradient 계산
→ Weight Update
        ↓
Validation
→ gradient 없이 성능 확인
        ↓
History CSV 기록
        ↓
현재 Validation Accuracy가 최고이면
Model Checkpoint 저장
        ↓
최고 Validation Accuracy와 저장 경로 출력
```

## 코드

```python

import csv
import random
from pathlib import Path

import torch
from torch import nn
from torch.utils.data import DataLoader
from torchvision import datasets, transforms

from src.config_loader import load_config
from src.tiny_cnn import TinyCNN


def set_seed(seed: int = 13):
    random.seed(seed)
    torch.manual_seed(seed)

    if torch.cuda.is_available():
        torch.cuda.manual_seed_all(seed)


def accuracy_from_logits(
    logits,
    targets,
):
    predictions = logits.argmax(
        dim=1
    )

    correct = (
        predictions == targets
    ).sum().item()

    return correct


def run_epoch(
    model,
    loader,
    criterion,
    device,
    optimizer=None,
):
    training = optimizer is not None

    if training:
        model.train()
    else:
        model.eval()

    total_loss = 0.0
    total_correct = 0
    total_samples = 0

    for images, targets in loader:
        images = images.to(device)
        targets = targets.to(device)

        if training:
            optimizer.zero_grad()

        with torch.set_grad_enabled(
            training
        ):
            logits = model(images)

            loss = criterion(
                logits,
                targets,
            )

            if training:
                loss.backward()
                optimizer.step()

        batch_size = targets.size(0)

        total_loss += (
            loss.item()
            * batch_size
        )

        total_correct += (
            accuracy_from_logits(
                logits,
                targets,
            )
        )

        total_samples += batch_size

    mean_loss = (
        total_loss / total_samples
    )

    accuracy = (
        total_correct / total_samples
    )

    return mean_loss, accuracy


def append_history(
    path: Path,
    epoch: int,
    train_loss: float,
    train_acc: float,
    val_loss: float,
    val_acc: float,
):
    path.parent.mkdir(
        parents=True,
        exist_ok=True,
    )

    is_new = not path.exists()

    with path.open(
        "a",
        newline="",
        encoding="utf-8",
    ) as file:
        writer = csv.writer(file)

        if is_new:
            writer.writerow(
                [
                    "epoch",
                    "train_loss",
                    "train_accuracy",
                    "val_loss",
                    "val_accuracy",
                ]
            )

        writer.writerow(
            [
                epoch,
                f"{train_loss:.6f}",
                f"{train_acc:.6f}",
                f"{val_loss:.6f}",
                f"{val_acc:.6f}",
            ]
        )


def main():
    set_seed(13)

    config = load_config(
        "configs/settings.yaml"
    )

    dataset_dir = Path(
        config["dataset_dir"]
    )

    image_size = int(
        config["ai_image_size"]
    )

    batch_size = int(
        config["ai_batch_size"]
    )

    epochs = int(
        config["ai_epochs"]
    )

    learning_rate = float(
        config["ai_learning_rate"]
    )

    model_path = Path(
        config["ai_model_path"]
    )

    history_path = Path(
        config["training_history_path"]
    )

    if history_path.exists():
        history_path.unlink()

    transform = transforms.Compose(
        [
            transforms.Resize(
                (image_size, image_size)
            ),
            transforms.ToTensor(),
        ]
    )

    train_dataset = (
        datasets.ImageFolder(
            dataset_dir / "train",
            transform=transform,
        )
    )

    val_dataset = (
        datasets.ImageFolder(
            dataset_dir / "val",
            transform=transform,
        )
    )

    expected_classes = list(
        config["class_names"]
    )

    if train_dataset.classes != expected_classes:
        raise RuntimeError(
            "Config의 class_names와 Train Dataset의 "
            "Class 순서가 다릅니다.\n"
            f"Config: {expected_classes}\n"
            f"Train : {train_dataset.classes}"
        )

    if (
        train_dataset.class_to_idx
        != val_dataset.class_to_idx
    ):
        raise RuntimeError(
            "Train과 Validation의 "
            "Class 순서가 다릅니다."
        )

    train_loader = DataLoader(
        train_dataset,
        batch_size=batch_size,
        shuffle=True,
        num_workers=0,
    )

    val_loader = DataLoader(
        val_dataset,
        batch_size=batch_size,
        shuffle=False,
        num_workers=0,
    )

    device = torch.device(
        "cuda"
        if torch.cuda.is_available()
        else "cpu"
    )

    model = TinyCNN(
        num_classes=len(
            train_dataset.classes
        )
    ).to(device)

    criterion = nn.CrossEntropyLoss()

    optimizer = torch.optim.Adam(
        model.parameters(),
        lr=learning_rate,
    )

    best_val_acc = -1.0

    print("=== Training ===")
    print(f"Device      : {device}")
    print(
        f"Classes     : "
        f"{train_dataset.classes}"
    )
    print(
        f"Train       : "
        f"{len(train_dataset)}"
    )
    print(
        f"Validation  : "
        f"{len(val_dataset)}"
    )
    print(f"Image Size  : {image_size}")
    print(f"Epochs      : {epochs}")
    print()

    for epoch in range(
        1,
        epochs + 1,
    ):
        train_loss, train_acc = (
            run_epoch(
                model,
                train_loader,
                criterion,
                device,
                optimizer=optimizer,
            )
        )

        with torch.no_grad():
            val_loss, val_acc = (
                run_epoch(
                    model,
                    val_loader,
                    criterion,
                    device,
                )
            )

        append_history(
            history_path,
            epoch,
            train_loss,
            train_acc,
            val_loss,
            val_acc,
        )

        print(
            f"Epoch "
            f"{epoch:02d}/{epochs} | "
            f"train_loss="
            f"{train_loss:.4f} | "
            f"train_acc="
            f"{train_acc:.4f} | "
            f"val_loss="
            f"{val_loss:.4f} | "
            f"val_acc="
            f"{val_acc:.4f}"
        )

        if val_acc > best_val_acc:
            best_val_acc = val_acc

            model_path.parent.mkdir(
                parents=True,
                exist_ok=True,
            )

            torch.save(
                {
                    "model_state":
                    model.state_dict(),

                    "class_to_idx":
                    train_dataset.class_to_idx,

                    "classes":
                    train_dataset.classes,

                    "image_size":
                    image_size,

                    "best_val_accuracy":
                    best_val_acc,
                },
                model_path,
            )

            print(
                f"  → BEST MODEL SAVED "
                f"({best_val_acc:.4f})"
            )

    print()
    print(
        f"Best Validation Accuracy: "
        f"{best_val_acc:.4f}"
    )
    print(f"Model: {model_path}")
    print(f"History: {history_path}")


if __name__ == "__main__":
    main()

```

---

# 37. Training 코드를 실행하기 전에 문법과 파일을 점검합니다

프로젝트 루트에서:

```bash
python -m py_compile   src/tiny_cnn.py   scripts/build_dataset_split.py   scripts/dataset_summary.py   scripts/check_split_leakage.py   scripts/model_summary.py   scripts/train_tiny_cnn.py
```

아무 메시지 없이 종료되면 Python 문법 검사는 통과한 것입니다.

Dataset:

```bash
python -m scripts.dataset_summary
```

Leakage:

```bash
python -m scripts.check_split_leakage
```

Model:

```bash
python -m scripts.model_summary
```

이 세 가지가 정상인 뒤 Training으로 넘어갑니다.

---

# 38. Guided Lab — Tiny CNN Training 실행

Baseline 파일을 덮어쓰기 전에 현재 설정을 확인합니다.

```bash
grep -E   "ai_image_size|ai_batch_size|ai_epochs|ai_learning_rate|ai_model_path|training_history_path"   configs/settings.yaml
```

기본값:

```text
Image Size    96
Batch Size    32
Epochs        15
Learning Rate 0.001

Model
models/day06_tiny_cnn.pt

History
reports/day06_training_history.csv
```

실행:

```bash
python -m scripts.train_tiny_cnn
```

예상 시작 형태:

```text
=== Training ===
Device      : cuda
또는 cpu

Classes     : ['normal', 'warning_red']
Train       : 120
Validation  : 120
Image Size  : 96
Epochs      : 15
```

Epoch별:

```text
Epoch 01/15 | train_loss=... | train_acc=... | val_loss=... | val_acc=...
  → BEST MODEL SAVED (...)

Epoch 02/15 | ...
...
```

실제 Accuracy는 자신의 Dataset에 따라 달라집니다.

---

# 39. Training 결과를 어떻게 해석할까요?

초보자가 먼저 볼 값은 네 가지입니다.

```text
train_loss
train_acc
val_loss
val_acc
```

대표적인 패턴:

```text
Train과 Val Accuracy가 함께 올라감
→ 학습이 진행되는 정상적인 모습일 수 있음

Train Accuracy는 매우 높은데
Val Accuracy가 낮음
→ Train 장면에 과하게 맞았을 가능성

Train과 Val 모두 낮음
→ 데이터 품질 / Label / 모델 학습 조건을 점검

Val Accuracy가 Epoch마다 크게 흔들림
→ Dataset이 작거나 조건 차이가 큰지 확인
```

Accuracy 하나만 보고 원인을 확정하지 않습니다.

---

# 40. `BEST MODEL SAVED`는 어느 코드에서 만들어졌을까요?

터미널에 다음이 나왔다고 가정합니다.

```text
→ BEST MODEL SAVED (0.9417)
```

직접 `scripts/train_tiny_cnn.py`를 열어 찾습니다.

```text
val_acc 계산
        ↓
if val_acc > best_val_acc
        ↓
best_val_acc 갱신
        ↓
torch.save(...)
        ↓
models/day06_tiny_cnn.pt
        ↓
print("BEST MODEL SAVED")
```

Checkpoint에는 다음 정보가 들어갑니다.

```text
model_state
→ 학습된 Weight

class_to_idx
→ Class 이름과 숫자 Index 관계

classes
→ ['normal', 'warning_red']

image_size
→ 96

best_val_accuracy
→ 저장 당시 최고 Validation Accuracy
```

7일차 ONNX Export에서 이 정보 중 `model_state`, `classes`, `class_to_idx`, `image_size`를 다시 사용합니다.

---

# 41. Training History CSV를 확인합니다

```bash
head -n 6 reports/day06_training_history.csv
```

Header:

```csv
epoch,train_loss,train_accuracy,val_loss,val_accuracy
```

15 Epoch를 정상 완료했다면:

```bash
wc -l reports/day06_training_history.csv
```

기본적으로:

```text
Header 1
+
Epoch 15
=
16 lines
```

실행을 다시 하면 현재 Training Program은 기존 History를 지우고 새 실험 결과를 기록합니다. 따라서 실험 비교가 필요하면 **Model Path와 History Path를 별도로 지정**합니다.

---

# 42. Training 오류를 구분해서 해결합니다

## 오류 A — `ModuleNotFoundError: No module named 'torch'`

```text
오류
→ which python 확인
→ PC .venv인지 확인
→ PyTorch 설치 여부 확인
→ 교육장 허용 설치 방식으로 설치
→ Import 확인
→ Training 재실행
```

## 오류 B — Dataset Class 순서가 다름

예:

```text
Config:
['normal', 'warning_red']

Train:
['normal', 'warning']
```

확인:

```bash
find data/dataset/train   -maxdepth 1 -type d | sort
```

원본 Session Class 이름을 확인하고 Split을 다시 만듭니다.

```bash
python -m scripts.build_dataset_split
python -m scripts.dataset_summary
```

## 오류 C — Model이 저장되지 않음

확인:

```bash
ls -lh models
```

Training이 끝까지 실행되었는지, `ai_model_path`가 올바른지 확인합니다.

## 오류 D — CUDA 관련 오류

`torch.cuda.is_available()`이 `False`이면 오늘은 CPU로 실행해도 됩니다.

```bash
python -c "import torch; print(torch.cuda.is_available())"
```

강제로 `cuda` 문자열을 고정하지 않습니다. 현재 코드는 사용 가능 여부를 확인해 자동 선택합니다.

---

# PART H. Mini Challenge — Test를 열기 전에 스스로 이해하기

Mini Challenge는 **Test 결과를 Model 선택에 사용하지 않도록 Test 평가 전에 수행**합니다.

정답을 바로 열지 않습니다. 먼저 자신의 예상과 해결 과정을 적은 뒤 `예시 정답 확인해보기`를 확인합니다.

---

## Challenge 1 — Training 출력 한 줄을 코드로 역추적하기

다음 결과를 봅니다.

```text
Epoch 04/15 | train_loss=0.3210 | train_acc=0.9000 | val_loss=0.4100 | val_acc=0.8500
  → BEST MODEL SAVED (0.8500)
```

다음을 채웁니다.

| 화면 값 | 담당 파일 | 담당 함수/코드 |
|---|---|---|
| `04/15` |  |  |
| `train_loss` |  |  |
| `train_acc` |  |  |
| `val_acc` |  |  |
| `BEST MODEL SAVED` |  |  |
| 실제 `.pt` 저장 |  |  |
| Epoch별 CSV 저장 |  |  |

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

| 화면 값 | 담당 파일 | 담당 함수/코드 |
|---|---|---|
| `04/15` | `scripts/train_tiny_cnn.py` | `for epoch in range(...)` |
| `train_loss` | `scripts/train_tiny_cnn.py` | `run_epoch()` 반환값 |
| `train_acc` | `scripts/train_tiny_cnn.py` | `accuracy_from_logits()`를 누적한 결과 |
| `val_acc` | `scripts/train_tiny_cnn.py` | Validation `run_epoch()` 결과 |
| `BEST MODEL SAVED` | `scripts/train_tiny_cnn.py` | `if val_acc > best_val_acc:` |
| `.pt` 저장 | `scripts/train_tiny_cnn.py` | `torch.save(...)` |
| History CSV | `scripts/train_tiny_cnn.py` | `append_history()` |

핵심 흐름:

```text
Dataset
→ run_epoch()
→ Loss / Accuracy
→ Validation 비교
→ torch.save()
→ History CSV
```

</details>

---

## Challenge 2 — 실행 전에 Parameter 수를 예측하기

기본 Tiny CNN은 다음 구조입니다.

```text
Conv 3→16
Conv 16→32
Conv 32→64
Linear 64→2
```

`python -m scripts.model_summary`를 실행하기 전에 다음 질문에 답합니다.

```text
Class 수는 몇 개인가?

마지막 Linear의 출력 개수는 몇 개인가?

Parameter 수는 대략 수천 개인가?
수만 개인가?
수백만 개인가?
```

가능하면 정확한 예상값도 적습니다.

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

기본 2 Class에서는 다음 값이 예상됩니다.

```text
Classes
→ ['normal', 'warning_red']

Final Output
→ 2

Parameters
→ 23,714
```

계산 예:

```text
Conv1
3 × 16 × 3 × 3 + 16
= 448

Conv2
16 × 32 × 3 × 3 + 32
= 4,640

Conv3
32 × 64 × 3 × 3 + 64
= 18,496

Linear
64 × 2 + 2
= 130

Total
448 + 4,640 + 18,496 + 130
= 23,714
```

Model이 작은 편이라는 것을 숫자로 확인할 수 있습니다.

</details>

---

## Challenge 3 — Python 코드는 그대로 두고 설정값만 바꾸어 Validation 비교하기

공식 Baseline을 덮어쓰지 않도록 Challenge 전용 경로를 사용합니다.

`configs/settings.yaml`의 Day 06 부분만 잠시 다음처럼 변경합니다.

```yaml
ai_image_size: 96
ai_batch_size: 32
ai_epochs: 3
ai_learning_rate: 0.001

ai_model_path: models/day06_challenge_epochs3.pt
training_history_path: reports/day06_challenge_epochs3.csv
```

실행하기 전에 예상합니다.

```text
몇 Epoch가 출력될까?

History CSV는 총 몇 줄일까?

공식 day06_tiny_cnn.pt가 덮어써질까?

Model 구조가 같다면 Parameter 수가 달라질까?
```

실행:

```bash
python -m scripts.train_tiny_cnn
```

확인:

```bash
wc -l reports/day06_challenge_epochs3.csv
ls -lh models/day06_challenge_epochs3.pt
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
Epoch
→ 3회

History
→ Header 1 + Epoch 3
→ 총 4줄

공식 Model
→ ai_model_path를 Challenge 전용 경로로 바꾸었으므로
   models/day06_tiny_cnn.pt를 덮어쓰지 않음

Parameter 수
→ Epoch 수만 바꿨으므로 Model 구조는 동일
→ 23,714
```

결과가 좋아지거나 나빠지는 방향은 Dataset에 따라 다릅니다.

비교 기준은 Test가 아니라 **Validation 결과**입니다.

</details>

---

## Challenge 4 — 기존 `model_summary.py` 기능을 조금 확장하기

현재 `model_summary.py`는 Class와 Parameter 수를 출력합니다.

이번에는 Dummy Tensor 한 개를 Model에 넣어 **Output Shape도 확인**하도록 잠시 기능을 추가합니다.

요구사항:

```text
Input Shape
→ [1, 3, 96, 96]

TinyCNN 실행
        ↓
Output Shape
→ [1, 2]
```

어느 Import와 어느 코드가 추가되어야 하는지 먼저 생각합니다.

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

`model_summary.py` 위쪽에:

```python
import torch
```

`TinyCNN`을 만든 뒤 다음을 추가할 수 있습니다.

```python
image_size = int(
    config["ai_image_size"]
)

dummy = torch.zeros(
    1,
    3,
    image_size,
    image_size,
)

logits = model(dummy)

print(f"Input Shape : {list(dummy.shape)}")
print(f"Output Shape: {list(logits.shape)}")
```

예상:

```text
Input Shape : [1, 3, 96, 96]
Output Shape: [1, 2]
```

`[1, 2]`의 의미:

```text
1
→ 이미지 한 장

2
→ normal / warning_red 두 Class의 Logit
```

이 확인은 Model 구조를 바꾸는 것이 아니라 입력과 출력 형태를 직접 보는 Smoke Test입니다.

</details>

---

## Challenge 5 — 일부러 Class 설정 오류를 만들고 원인 찾기

실제 Dataset 폴더를 망가뜨리지 않고 Config만 잠시 잘못 작성합니다.

기존:

```yaml
class_names:
  - normal
  - warning_red
```

잠시 다음 순서로 바꿉니다.

```yaml
class_names:
  - warning_red
  - normal
```

`python -m scripts.train_tiny_cnn`을 실행합니다.

바로 정답을 보지 말고 다음 순서로 확인합니다.

```text
오류 마지막 줄
        ↓
Config Class 출력
        ↓
Train Dataset Class 출력
        ↓
ImageFolder가 읽은 Class 순서 확인
        ↓
settings.yaml 복구
        ↓
재실행
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

최종 Training 코드에는 Config와 Dataset Class 순서를 확인하는 방어 코드가 있습니다.

비슷한 오류:

```text
RuntimeError:
Config의 class_names와 Train Dataset의 Class 순서가 다릅니다.
Config: ['warning_red', 'normal']
Train : ['normal', 'warning_red']
```

원인:

```text
settings.yaml의 Class 순서를 바꾸었지만
ImageFolder는 실제 폴더 이름을 기준으로
['normal', 'warning_red'] 순서를 사용
```

복구:

```yaml
class_names:
  - normal
  - warning_red
```

그리고 다시 실행합니다.

```bash
python -m scripts.model_summary
```

Challenge의 목적은 오류를 피하는 것이 아니라 **Model·Dataset·Metadata가 같은 Class Contract를 가져야 한다는 것**을 확인하는 것입니다.

</details>

---

## Challenge 6 — 결과 파일로 정상 동작을 증명하기

화면에서 “학습됐다”고 말하는 것만으로 끝내지 않습니다.

아래 항목을 직접 확인합니다.

```bash
ls -lh models/day06_tiny_cnn.pt
```

```bash
head -n 5 reports/day06_training_history.csv
```

```bash
wc -l reports/day06_training_history.csv
```

```bash
python -m scripts.check_split_leakage
```

다음 표를 채웁니다.

| 증거 | 실제 결과 | 무엇을 증명하나요? |
|---|---|---|
| Model 파일 존재 |  |  |
| History Header |  |  |
| History 줄 수 |  |  |
| Leakage Check |  |  |
| Dataset Summary |  |  |

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

기본 15 Epoch Baseline이라면 예를 들어:

```text
models/day06_tiny_cnn.pt 존재
→ Validation 기준으로 저장된 Checkpoint가 있음

History Header
epoch,train_loss,train_accuracy,val_loss,val_accuracy
→ 어떤 Metric을 기록했는지 확인

History 16줄
→ Header 1 + Epoch 15

RESULT: NO EXACT DUPLICATES
→ Split 사이 완전 동일 JPG를 발견하지 못함

Dataset Summary
train/val/test의 두 Class Count 확인
→ 학습 입력 구조가 준비됨
```

이 결과 파일들이 **실행이 실제로 일어났다는 증거**입니다.

</details>

---

## Challenge 7 — 오늘의 전체 Pipeline을 자신의 말로 설명하기

코드를 닫고 아래 빈칸을 채웁니다.

```text
5일차에서는
________________________________________
방식으로 판단했다.

6일차에서는
Raspberry Pi Camera로 __________________
를 수집했다.

연속 Frame Leakage를 줄이기 위해
________________________________________
단위로 Train / Validation / Test를 나눴다.

PC에서는
________________________________________
를 이용해 2 Class Model을 학습했다.

가장 좋은 Model은
________________________________________
기준으로 저장했다.

Test는
________________________________________
이 끝난 뒤 최종 평가에 사용한다.

6일차의 최종 Model은
________________________________________
이다.

7일차에서는 이 Model을
________________________________________
로 변환한다.
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
5일차
→ HSV + Red Area + Threshold Rule

6일차 데이터
→ normal / warning_red 이미지

Split
→ Session 단위

PC Model
→ Tiny CNN

Best Model
→ Validation Accuracy가 이전 최고값보다 높을 때 저장

Test
→ Model 선택이 끝난 뒤 사용

최종 Model
→ models/day06_tiny_cnn.pt

7일차
→ ONNX로 변환
```

정답 문장을 외우는 것이 아니라 **왜 이런 순서가 필요한지 자신의 말로 설명**할 수 있어야 합니다.

</details>

---

# 43. Mini Challenge 마무리 — 공식 Baseline 상태로 복원합니다

Challenge의 설정과 임시 코드를 다음 날까지 남기지 않습니다.

## 1. Day 06 기본 Config 복원

```yaml
class_names:
  - normal
  - warning_red

capture_count: 60
capture_interval_sec: 0.20

session_root: data/sessions
dataset_dir: data/dataset

ai_image_size: 96
ai_batch_size: 32
ai_epochs: 15
ai_learning_rate: 0.001

ai_model_path: models/day06_tiny_cnn.pt
training_history_path: reports/day06_training_history.csv
```

## 2. `model_summary.py` Challenge 수정 복원

Challenge 4의 Dummy Tensor 출력은 학습 이해용 실험입니다. 7일차 연결을 단순하게 유지하려면 원래 Baseline 코드로 되돌립니다.

```bash
git diff scripts/model_summary.py
```

원래 코드로 복원한 뒤:

```bash
python -m scripts.model_summary
```

## 3. 공식 Baseline History와 Model 확인

Challenge 전용 파일과 공식 파일을 구분합니다.

```text
공식
models/day06_tiny_cnn.pt
reports/day06_training_history.csv

Challenge
models/day06_challenge_epochs3.pt
reports/day06_challenge_epochs3.csv
```

Challenge 파일은 7일차 입력이 아닙니다.

## 4. Dataset Contract 다시 확인

```bash
python -m scripts.dataset_summary
python -m scripts.check_split_leakage
```

## 5. 공식 Baseline을 마지막으로 다시 확인

공식 Config로 이미 15 Epoch Baseline을 정상 학습했고 파일이 보존되어 있다면 불필요하게 다시 학습할 필요는 없습니다.

파일이 Challenge 과정에서 덮어써졌거나 확신이 없다면 다시 실행합니다.

```bash
python -m scripts.train_tiny_cnn
```

최종적으로 다음 파일이 있어야 합니다.

```text
models/day06_tiny_cnn.pt
reports/day06_training_history.csv
```

이제부터 **공식 Baseline 설정은 고정**합니다.

Test 결과를 본 뒤 이 설정을 다시 선택하지 않습니다.

---

# PART I. 고정된 Baseline을 Test에서 최종 평가하기

# 44. Test 평가 Program을 만듭니다

새 파일:

```text
scripts/evaluate_tiny_cnn.py
```

## 이 파일은 왜 필요할까요?

Training Program은 Train과 Validation만 사용했습니다.

이제 **선택이 끝난 Checkpoint**를 Test Dataset에서 한 번 평가합니다.

기록:

```text
Test Accuracy
Parameter Count
Model File Size
Confusion Matrix
```

Test Accuracy가 기대보다 낮더라도 Test 이미지를 Train으로 옮겨 다시 학습하지 않습니다.

## 의사코드

```text
settings.yaml을 읽는다
        ↓
Model 파일이 존재하는지 확인한다
        ↓
Checkpoint를 CPU로 읽는다
        ↓
Checkpoint의 classes / image_size를 읽는다
        ↓
Test Dataset을 같은 Resize / ToTensor로 준비한다
        ↓
Test Class 순서가 Checkpoint Class와 같은지 확인한다
        ↓
TinyCNN을 만들고 Weight를 읽는다
        ↓
model.eval()
        ↓
Gradient 없이 Test 전체 반복
        ↓
Prediction = 가장 큰 Logit의 Index
        ↓
Correct / Total 계산
        ↓
Confusion Matrix 누적
        ↓
Accuracy 계산
        ↓
Parameter Count와 Model 파일 크기 계산
        ↓
터미널 출력
        ↓
day06_test_metrics.csv 저장
        ↓
day06_confusion_matrix.csv 저장
```

## 코드

```python

import csv
from pathlib import Path

import torch
from torch.utils.data import DataLoader
from torchvision import datasets, transforms

from src.config_loader import load_config
from src.tiny_cnn import TinyCNN


def main():
    config = load_config(
        "configs/settings.yaml"
    )

    dataset_dir = Path(
        config["dataset_dir"]
    )

    model_path = Path(
        config["ai_model_path"]
    )

    if not model_path.exists():
        raise FileNotFoundError(
            f"Model 파일이 없습니다: {model_path}"
        )

    checkpoint = torch.load(
        model_path,
        map_location="cpu",
    )

    classes = list(
        checkpoint["classes"]
    )

    image_size = int(
        checkpoint["image_size"]
    )

    transform = transforms.Compose(
        [
            transforms.Resize(
                (image_size, image_size)
            ),
            transforms.ToTensor(),
        ]
    )

    test_dataset = datasets.ImageFolder(
        dataset_dir / "test",
        transform=transform,
    )

    if test_dataset.classes != classes:
        raise RuntimeError(
            "Model과 Test Dataset의 "
            "Class 순서가 다릅니다."
        )

    test_loader = DataLoader(
        test_dataset,
        batch_size=int(config["ai_batch_size"]),
        shuffle=False,
        num_workers=0,
    )

    model = TinyCNN(
        num_classes=len(classes)
    )

    model.load_state_dict(
        checkpoint["model_state"]
    )

    model.eval()

    matrix = [
        [0 for _ in classes]
        for _ in classes
    ]

    correct = 0
    total = 0

    with torch.no_grad():
        for images, targets in test_loader:
            logits = model(images)

            predictions = logits.argmax(
                dim=1
            )

            for true, pred in zip(
                targets.tolist(),
                predictions.tolist(),
            ):
                matrix[true][pred] += 1

                if true == pred:
                    correct += 1

                total += 1

    if total == 0:
        raise RuntimeError(
            "Test Dataset에 이미지가 없습니다."
        )

    accuracy = correct / total

    parameter_count = sum(
        parameter.numel()
        for parameter in model.parameters()
    )

    model_size_mb = (
        model_path.stat().st_size
        / (1024 * 1024)
    )

    print("=== Test Result ===")
    print(f"Classes     : {classes}")
    print(f"Samples     : {total}")
    print(f"Accuracy    : {accuracy:.4f}")
    print(
        f"Parameters  : "
        f"{parameter_count:,}"
    )
    print(
        f"Model Size  : "
        f"{model_size_mb:.3f} MB"
    )

    print()
    print("Confusion Matrix")
    print("Rows=True, Cols=Pred")

    for class_name, row in zip(
        classes,
        matrix,
    ):
        print(
            f"{class_name:12s} {row}"
        )

    metrics_path = Path(
        "reports/day06_test_metrics.csv"
    )

    metrics_path.parent.mkdir(
        parents=True,
        exist_ok=True,
    )

    with metrics_path.open(
        "w",
        newline="",
        encoding="utf-8",
    ) as file:
        writer = csv.writer(file)

        writer.writerow(
            ["metric", "value"]
        )

        writer.writerow(
            ["test_accuracy", accuracy]
        )

        writer.writerow(
            [
                "parameter_count",
                parameter_count,
            ]
        )

        writer.writerow(
            [
                "model_size_mb",
                f"{model_size_mb:.6f}",
            ]
        )

    matrix_path = Path(
        "reports/"
        "day06_confusion_matrix.csv"
    )

    with matrix_path.open(
        "w",
        newline="",
        encoding="utf-8",
    ) as file:
        writer = csv.writer(file)

        writer.writerow(
            ["true/pred", *classes]
        )

        for class_name, row in zip(
            classes,
            matrix,
        ):
            writer.writerow(
                [class_name, *row]
            )

    print()
    print(f"Metrics: {metrics_path}")
    print(f"Matrix : {matrix_path}")


if __name__ == "__main__":
    main()

```

---

# 45. 실행 전에 Training과 Test 파일 연결을 확인합니다

```text
models/day06_tiny_cnn.pt
        ↓
torch.load()
        ↓
Checkpoint
├── model_state
├── classes
└── image_size

        ↓

src/tiny_cnn.py
→ 같은 Model 구조 생성
        ↓
load_state_dict()
        ↓

data/dataset/test
        ↓
ImageFolder
        ↓
Prediction
        ↓
Accuracy / Confusion Matrix
        ↓
reports/day06_test_metrics.csv
reports/day06_confusion_matrix.csv
```

---

# 46. Guided Lab — Test를 최종 평가합니다

실행:

```bash
python -m scripts.evaluate_tiny_cnn
```

예상 형태:

```text
=== Test Result ===
Classes     : ['normal', 'warning_red']
Samples     : 120
Accuracy    : ...
Parameters  : 23,714
Model Size  : ... MB

Confusion Matrix
Rows=True, Cols=Pred
normal       [...]
warning_red  [...]

Metrics: reports/day06_test_metrics.csv
Matrix : reports/day06_confusion_matrix.csv
```

숫자는 자신의 Dataset 결과를 사용합니다.

Test가 낮게 나왔다면 다음을 기록합니다.

```text
Train Accuracy는 어땠나?
Validation Accuracy는 어땠나?
Session 03은 무엇이 달랐나?
어느 Class를 더 자주 틀렸나?
```

그 결과는 **다음 개선 실험의 가설**이 될 수 있지만, 오늘의 Test를 다시 학습 데이터로 바꾸지는 않습니다.

---

# 47. Confusion Matrix를 읽습니다

예:

```text
Rows=True
Cols=Prediction

                 Pred
              normal  warning_red

True normal      55       5
True warning      8      52
```

해석:

```text
실제 normal 60장
→ 55장 normal
→ 5장 warning_red 오분류

실제 warning_red 60장
→ 52장 warning_red
→ 8장 normal 오분류
```

Accuracy가 같아도 어떤 Class를 틀렸는지에 따라 운영 의미는 다를 수 있습니다.

산업 경고 시스템에서는 `warning_red`를 `normal`로 놓치는 오류가 어떤 의미인지도 생각해 봅니다.

---

# 48. Test 결과를 코드에서 역추적합니다

다음 결과가 보였다고 가정합니다.

```text
Accuracy : 0.8917
```

코드 위치:

```text
evaluate_tiny_cnn.py
→ correct
→ total
→ accuracy = correct / total
```

Confusion Matrix의 한 숫자:

```text
matrix[true][pred] += 1
```

Model Size:

```text
model_path.stat().st_size
→ Byte
→ 1024 × 1024로 나눔
→ MB
```

CSV 저장:

```text
day06_test_metrics.csv
→ Test Accuracy / Parameter Count / Model Size

day06_confusion_matrix.csv
→ True Class별 Prediction Count
```

---

# 49. 단일 이미지 Prediction Program 만들기

새 파일:

```text
scripts/predict_tiny_cnn.py
```

## 이 파일은 왜 필요할까요?

전체 Test Accuracy만 보면 한 장의 실제 이미지가 어떻게 예측되었는지 감이 잘 오지 않습니다.

그리고 7일차에는 **동일한 Test 이미지**를 PyTorch와 ONNX에 넣어 결과를 비교합니다.

따라서 오늘 한 장 Prediction의 기준 결과를 남깁니다.

## 의사코드

```text
--image 경로를 입력받는다
        ↓
settings.yaml에서 Model 경로를 읽는다
        ↓
Model과 Image가 실제로 있는지 확인한다
        ↓
Checkpoint를 읽는다
        ↓
classes / image_size 읽기
        ↓
TinyCNN 생성 + Weight 로드
        ↓
이미지 RGB 변환
        ↓
Resize + ToTensor
        ↓
Batch 차원 추가
        ↓
Gradient 없이 Model 실행
        ↓
Softmax로 Class Probability 계산
        ↓
가장 큰 Probability의 Class 선택
        ↓
Class / Confidence / 모든 Class Probability 출력
```

## 코드

```python

import argparse
from pathlib import Path

import torch
from PIL import Image
from torchvision import transforms

from src.config_loader import load_config
from src.tiny_cnn import TinyCNN


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

    model_path = Path(
        config["ai_model_path"]
    )

    image_path = Path(args.image)

    if not model_path.exists():
        raise FileNotFoundError(
            f"Model 파일이 없습니다: {model_path}"
        )

    if not image_path.exists():
        raise FileNotFoundError(
            f"이미지 파일이 없습니다: {image_path}"
        )

    checkpoint = torch.load(
        model_path,
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

    transform = transforms.Compose(
        [
            transforms.Resize(
                (image_size, image_size)
            ),
            transforms.ToTensor(),
        ]
    )

    image = Image.open(
        image_path
    ).convert("RGB")

    tensor = transform(
        image
    ).unsqueeze(0)

    with torch.no_grad():
        logits = model(tensor)

        probabilities = torch.softmax(
            logits,
            dim=1,
        )[0]

    index = int(
        probabilities.argmax().item()
    )

    class_name = classes[index]

    confidence = float(
        probabilities[index].item()
    )

    print("=== Prediction ===")
    print(f"Image      : {image_path}")
    print(f"Class      : {class_name}")
    print(
        f"Confidence : "
        f"{confidence:.4f}"
    )

    print()
    print("All probabilities")

    for name, probability in zip(
        classes,
        probabilities.tolist(),
    ):
        print(
            f"{name:12s}: "
            f"{probability:.4f}"
        )


if __name__ == "__main__":
    main()

```

---

# 50. Guided Lab — normal / warning_red 한 장씩 예측

먼저 실제 파일명을 찾습니다.

```bash
find data/dataset/test/normal   -type f -name "*.jpg" | head
```

한 장 선택:

```bash
python -m scripts.predict_tiny_cnn   --image data/dataset/test/normal/<ACTUAL_FILE>.jpg
```

다음으로:

```bash
find data/dataset/test/warning_red   -type f -name "*.jpg" | head
```

```bash
python -m scripts.predict_tiny_cnn   --image data/dataset/test/warning_red/<ACTUAL_FILE>.jpg
```

예상 형태:

```text
=== Prediction ===
Image      : ...
Class      : normal
Confidence : 0.xxxx

All probabilities
normal      : 0.xxxx
warning_red : 0.xxxx
```

Confidence가 높다고 항상 정답이라는 뜻은 아닙니다.

```text
Prediction
→ Model이 가장 높게 선택한 Class

Confidence
→ 현재 Model 출력에서 선택 Class의 Softmax 값

정답 여부
→ 실제 Folder Label과 비교해야 판단
```

7일차에서 같은 이미지 파일명을 기록해 둡니다.

---

# 51. Model Size와 7일차 준비 정보를 기록합니다

```bash
ls -lh models/day06_tiny_cnn.pt
```

그리고 Checkpoint의 핵심은 다음입니다.

```text
Model Type
→ TinyCNN

Input Size
→ 96 × 96

Classes
→ normal / warning_red

Parameter Count
→ 23,714

Checkpoint
→ models/day06_tiny_cnn.pt
```

오늘은 Raspberry Pi Latency/FPS를 측정하지 않습니다.

```text
6일차
→ PC에서 AI Baseline의 Dataset / Accuracy / Model 확인

7일차
→ ONNX 변환 및 Raspberry Pi 추론

8일차
→ 실제 Latency / FPS 측정과 비교
```

---

# 52. 5일차 Rule 실패 사례를 AI와 비교할 수 있습니다

5일차에서 `data/day05_failures/`를 만들었다면 해당 파일은 Raspberry Pi의 Runtime 데이터이므로 PC에 없을 수 있습니다.

필요한 경우 PC에서 별도로 가져옵니다.

```bash
scp -r   <RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/data/day05_failures   ./data/
```

파일이 실제로 있을 때만 실행합니다.

```bash
find data/day05_failures -type f
```

예:

```bash
python -m scripts.predict_tiny_cnn   --image data/day05_failures/<ACTUAL_FILE>.jpg
```

Report에 비교합니다.

```text
조건
→ Rule 결과
→ AI Prediction
→ Confidence
→ 어느 쪽이 맞았는가?
→ 왜 그랬을 가능성이 있는가?
```

AI가 Rule보다 항상 좋아야 하는 것은 아닙니다.

---

# PART J. 결과 문서화와 Version 관리

# 53. Day 06 Report를 작성합니다

새 파일:

```text
reports/day06_ai_baseline.md
```

다음 템플릿을 사용합니다.

````md
# Day 06 AI Baseline

## 1. Problem

5일차 Rule 문제:
- normal:
- warning_red:

6일차 AI 문제:
- normal:
- warning_red:

Label은 5일차 Rule 출력이 아니라 실제 장면 의미로 결정했는가?
- yes / no

## 2. Environment

### Raspberry Pi Capture

Hostname:
Camera Index:
Requested Resolution:
Actual Resolution:

### PC Training

Python:
PyTorch:
Torchvision:
CUDA Available:
Training Device: cpu / cuda

## 3. Session Design

### Session 01 — Train

환경:
normal 이미지 수:
warning_red 이미지 수:

### Session 02 — Validation

변경한 조건:
normal 이미지 수:
warning_red 이미지 수:

### Session 03 — Test

변경한 조건:
normal 이미지 수:
warning_red 이미지 수:

## 4. Dataset Split

session01 → train  
session02 → val  
session03 → test

Dataset Summary:

## 5. Leakage Check

Train vs Val:
Train vs Test:
Val vs Test:
Result:

주의:
SHA-256은 완전히 동일한 파일만 확인한다.

## 6. Tiny CNN

Classes:
Input Size:
Parameter Count:
Epochs:
Batch Size:
Learning Rate:

## 7. Training / Validation

Best Validation Accuracy:
Best Model Path:

Training History에서 관찰한 점:

## 8. Test

Test Accuracy:

Confusion Matrix:

Class별로 확인한 점:

Test를 본 뒤 Baseline 설정을 다시 선택하지 않았는가?
- yes / no

## 9. Single Image Prediction

### normal

Image:
Prediction:
Confidence:

### warning_red

Image:
Prediction:
Confidence:

## 10. Rule vs AI

5일차 Rule이 잘한 조건:

5일차 Rule이 실패한 조건:

같은/유사 조건에서 AI 결과:

AI도 실패한 조건:

## 11. Day 07 Handoff

PyTorch Model:
models/day06_tiny_cnn.pt

Classes:
- normal
- warning_red

Input Size:
96

다음 단계:
PyTorch
→ ONNX
→ PC 결과 일치 검증
→ Raspberry Pi 배포
````

실제 수치를 기록합니다.

---

# 54. README에 Day 06 실행 순서를 추가합니다

`README.md`에 아래 내용을 추가합니다.

````md
## Day 06 — Tiny CNN AI Baseline

### Raspberry Pi — Session Capture

```bash
python -m scripts.collect_session   --session session01   --class-name normal
```

`normal / warning_red`, `session01 / 02 / 03`을 실제 촬영 계획에 맞게 실행합니다.

### PC — Dataset Build

```bash
python -m scripts.build_dataset_split
python -m scripts.dataset_summary
python -m scripts.check_split_leakage
```

### Model Check

```bash
python -m scripts.model_summary
```

### Training

```bash
python -m scripts.train_tiny_cnn
```

### Test

```bash
python -m scripts.evaluate_tiny_cnn
```

### Single Image Prediction

```bash
python -m scripts.predict_tiny_cnn   --image <ACTUAL_IMAGE_PATH>
```

### Current AI Flow

```text
Raspberry Pi Camera
→ Session Dataset
→ PC
→ Train / Val / Test
→ Tiny CNN
→ Best Validation Model
→ Test
→ models/day06_tiny_cnn.pt
```
````

README에 있는 명령은 실제 파일명과 맞아야 합니다.

---

# 55. Git에 넣을 것과 넣지 않을 것을 다시 구분합니다

Git으로 관리:

```text
configs/settings.yaml
src/tiny_cnn.py
scripts/*.py
README.md
reports/day06_ai_baseline.md
reports/day06_training_history.csv
reports/day06_test_metrics.csv
reports/day06_confusion_matrix.csv
reports/day06_pc_environment.txt
```

Git에서 제외:

```text
data/
models/*.pt
.venv/
logs/
configs/device.local.yaml
```

Model을 Git에서 제외해도 7일차에 필요하지 않은 것이 아닙니다.

```text
Git 제외
≠
삭제

models/day06_tiny_cnn.pt
→ PC Local Project에 보존
→ 7일차 ONNX Export 입력
```

---

# 56. Source와 결과를 Commit합니다

먼저:

```bash
git status
git diff
```

Source:

```bash
git add   configs/settings.yaml   src/tiny_cnn.py   scripts/collect_session.py   scripts/build_dataset_split.py   scripts/dataset_summary.py   scripts/check_split_leakage.py   scripts/model_summary.py   scripts/train_tiny_cnn.py   scripts/evaluate_tiny_cnn.py   scripts/predict_tiny_cnn.py
```

Commit:

```bash
git commit -m   "feat: add day06 tiny cnn baseline pipeline"
```

문서/결과:

```bash
git add   README.md   reports/day06_ai_baseline.md   reports/day06_training_history.csv   reports/day06_test_metrics.csv   reports/day06_confusion_matrix.csv   reports/day06_pc_environment.txt
```

```bash
git commit -m   "docs: record day06 ai baseline results"
```

확인:

```bash
git log --oneline -10
git status
```

---

# 57. Remote 저장소는 보안정책에 맞게 사용합니다

허용된 내부 GitLab/Gitea가 있는 경우:

```bash
git remote -v
git push
```

외부 Git 사용이 금지된 환경이라면 Local Git Commit까지만 수행합니다.

```text
Local Git
→ 정상 Version 보존

내부 Remote 허용
→ Push 가능

외부 Git 금지
→ 외부 Repository로 전송하지 않음
```

Raw Dataset, 장치별 Local 설정, 비밀번호, Wi-Fi 정보는 Remote에 올리지 않습니다.

---

# 58. 6일차가 끝난 시점의 프로젝트 구조

공통 Source와 Report:

```text
subject13_edge_ai/
│
├── configs/
│   └── settings.yaml
│
├── reports/
│   ├── day01_result.md
│   ├── day02_environment.md
│   ├── day03_camera_gpio.md
│   ├── day04_sensor_sampling.md
│   ├── day05_rule_warning.md
│   ├── day06_ai_baseline.md
│   ├── day06_training_history.csv
│   ├── day06_test_metrics.csv
│   ├── day06_confusion_matrix.csv
│   └── day06_pc_environment.txt
│
├── scripts/
│   ├── ...
│   ├── build_dataset_split.py
│   ├── check_split_leakage.py
│   ├── collect_session.py
│   ├── dataset_summary.py
│   ├── evaluate_tiny_cnn.py
│   ├── model_summary.py
│   ├── predict_tiny_cnn.py
│   └── train_tiny_cnn.py
│
├── src/
│   ├── ...
│   └── tiny_cnn.py
│
├── .gitignore
├── README.md
└── requirements.txt
```

PC Local Runtime:

```text
data/
├── sessions/
└── dataset/

models/
├── day06_tiny_cnn.pt
└── Challenge 파일이 남아 있다면 별도 구분

.venv/
```

Raspberry Pi Local Runtime:

```text
configs/device.local.yaml
data/sessions/
.venv/
```

---

# 59. 수업 종료 전 최종 Smoke Check

PC 프로젝트 루트에서 차례로 확인합니다.

```bash
pwd
which python
```

Dataset:

```bash
python -m scripts.dataset_summary
```

Leakage:

```bash
python -m scripts.check_split_leakage
```

Model 구조:

```bash
python -m scripts.model_summary
```

Model 파일:

```bash
ls -lh models/day06_tiny_cnn.pt
```

Training History:

```bash
head -n 3 reports/day06_training_history.csv
```

Test:

```bash
cat reports/day06_test_metrics.csv
cat reports/day06_confusion_matrix.csv
```

Git:

```bash
git status
git log --oneline -10
```

다음이 모두 준비되어 있어야 합니다.

```text
[ ] Session 01 / 02 / 03
[ ] normal / warning_red Label
[ ] Train / Val / Test Dataset
[ ] Dataset Summary
[ ] Exact Duplicate Leakage Check
[ ] TinyCNN Source
[ ] Best Validation Model
[ ] Training History
[ ] Test Accuracy
[ ] Confusion Matrix
[ ] Single Image Prediction
[ ] day06_ai_baseline.md
[ ] README Day 06
[ ] Git Commit
```

---

# 60. 핵심 복습 문제

교재와 코드를 잠시 닫고 먼저 답합니다.

1. 5일차 Rule과 6일차 AI의 가장 큰 차이는 무엇인가요?
2. 6일차의 `warning_red` Label은 5일차 Rule 출력으로 결정하나요?
3. 연속 Frame을 무작위로 Train/Test에 나누면 왜 문제가 될 수 있나요?
4. Session 01, 02, 03은 각각 어디에 사용하나요?
5. Train과 Validation의 역할은 어떻게 다른가요?
6. Test는 언제 사용해야 하나요?
7. `ImageFolder`에서 폴더 이름은 어떤 의미를 가지나요?
8. `build_dataset_split.py`는 원본 Session을 이동하나요, 복사하나요?
9. SHA-256 Leakage Check가 찾는 것은 무엇인가요?
10. SHA-256이 0 duplicate라고 하면 거의 같은 Frame도 없다는 뜻인가요?
11. 오늘 Tiny CNN의 기본 입력 크기는 얼마인가요?
12. 기본 Class가 2개일 때 Tiny CNN의 최종 출력 개수는 몇 개인가요?
13. Epoch는 무엇인가요?
14. Batch Size는 무엇인가요?
15. Learning Rate는 무엇인가요?
16. `train_loss`와 `val_loss`는 각각 어느 데이터에서 계산되나요?
17. Best Model은 어떤 기준으로 저장하나요?
18. `models/day06_tiny_cnn.pt` 안에는 Weight 외에 어떤 정보가 들어가나요?
19. Confusion Matrix의 Row와 Column은 오늘 어떻게 정의했나요?
20. 7일차는 6일차의 어떤 파일을 가장 먼저 사용하나요?

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

1. 5일차는 사람이 HSV·Red Area·Threshold를 직접 정하고, 6일차는 이미지와 Label을 이용해 CNN Weight를 학습합니다.
2. 아닙니다. 실제 장면 의미로 `normal / warning_red`를 정합니다.
3. 거의 같은 장면이 Train과 Test에 함께 들어가 성능이 과도하게 좋아 보일 수 있기 때문입니다.
4. `session01 → train`, `session02 → val`, `session03 → test`입니다.
5. Train은 Weight 학습, Validation은 Training 중 Model 선택에 사용합니다.
6. Model 선택과 설정 결정이 끝난 뒤 최종 평가에 사용합니다.
7. Class Label 이름으로 사용됩니다.
8. `shutil.copy2()`로 복사하여 `data/dataset`을 만듭니다.
9. Split 사이에 완전히 동일한 파일이 들어갔는지 확인합니다.
10. 아닙니다. 거의 같은 이미지라도 파일 내용이 다르면 Hash가 다를 수 있습니다.
11. `96 × 96 RGB`입니다.
12. 2개입니다.
13. Train Dataset 전체를 한 번 학습하는 단위입니다.
14. 한 번의 학습 계산에 묶어서 처리할 Sample 수입니다.
15. Weight Update의 기본 크기입니다.
16. `train_loss`는 Train, `val_loss`는 Validation 데이터에서 계산됩니다.
17. 현재 코드에서는 Validation Accuracy가 이전 최고값보다 높을 때 저장합니다.
18. `class_to_idx`, `classes`, `image_size`, `best_val_accuracy`도 함께 저장합니다.
19. `Rows=True`, `Cols=Prediction`입니다.
20. PC의 `models/day06_tiny_cnn.pt`입니다.

</details>

---

# 61. 자가 체크리스트

- [ ] 5일차 `subject13_edge_ai` 프로젝트를 그대로 이어서 사용했다.
- [ ] 새 Raspberry Pi 가상환경을 만들지 않았다.
- [ ] 실제 Raspberry Pi IP를 확인하고 접속했다.
- [ ] Camera Index를 추측하지 않고 Probe했다.
- [ ] 5일차 Rule Baseline을 먼저 재확인했다.
- [ ] `normal / warning_red` Label을 실제 장면 의미로 정의했다.
- [ ] Session 01/02/03의 역할을 설명할 수 있다.
- [ ] Session별로 두 Class를 새로 촬영했다.
- [ ] 같은 명령을 중복 실행해 이미지가 의도치 않게 늘어나지 않았는지 확인했다.
- [ ] Metadata CSV와 실제 촬영 수를 확인했다.
- [ ] Source Code와 Raw Dataset을 다른 방식으로 관리했다.
- [ ] Dataset을 PC로 복사했다.
- [ ] PC `.venv`에서 PyTorch/Torchvision을 확인했다.
- [ ] `session01→train`, `session02→val`, `session03→test` 규칙을 유지했다.
- [ ] `dataset_summary.py`로 Class 수를 확인했다.
- [ ] `check_split_leakage.py`로 Exact Duplicate를 확인했다.
- [ ] SHA-256 검사의 한계를 설명할 수 있다.
- [ ] `tiny_cnn.py`가 Model 구조만 담당한다는 것을 이해한다.
- [ ] 기본 Parameter 수 23,714를 확인했다.
- [ ] 96×96 RGB가 Tensor로 바뀌는 흐름을 설명할 수 있다.
- [ ] Train과 Validation의 역할을 구분한다.
- [ ] Best Validation Model이 저장되는 코드를 찾았다.
- [ ] Training History CSV를 확인했다.
- [ ] Mini Challenge에서 결과를 코드 위치로 역추적했다.
- [ ] Mini Challenge에서 실행 전에 결과를 예측했다.
- [ ] Mini Challenge에서 Python 코드를 바꾸지 않고 Config만 변경했다.
- [ ] Mini Challenge에서 기존 기능을 수정해 보았다.
- [ ] Mini Challenge에서 일부러 오류를 만들고 원인을 찾아 복구했다.
- [ ] 결과 파일로 정상 동작을 증명했다.
- [ ] Challenge 후 Day 06 공식 Baseline으로 복원했다.
- [ ] Test는 Baseline 고정 후 최종 평가에 사용했다.
- [ ] Confusion Matrix를 읽었다.
- [ ] normal / warning_red 한 장씩 Prediction했다.
- [ ] `models/day06_tiny_cnn.pt`를 보존했다.
- [ ] `reports/day06_ai_baseline.md`를 작성했다.
- [ ] README에 실제 실행 명령을 기록했다.
- [ ] Runtime Dataset/Model과 Source/Report의 Git 관리 범위를 구분했다.
- [ ] 오늘의 전체 Pipeline을 자신의 말로 설명할 수 있다.
- [ ] 7일차에 `.pt → ONNX`로 이어진다는 것을 이해한다.

---

# 62. 1~6일차 시스템 성장 확인하기

```text
1일차
가상 입력
→ 판단
→ Log

2일차
PC
→ SSH
→ Raspberry Pi
→ 같은 Project

3일차
Camera
→ Raspberry Pi
→ GPIO

4일차
Sensor
→ Sampling
→ Timestamp
→ CSV
→ GPIO

5일차
Camera
→ HSV / Red Area Rule
→ NORMAL / WARNING
→ GPIO
→ Event Log
→ WARNING Image

6일차
Camera
→ Session Dataset
→ Train / Validation / Test
→ Tiny CNN
→ Best Validation Model
→ Test
→ PyTorch Checkpoint
```

가장 큰 변화:

```text
5일차
사람이 판단 기준을 코드로 작성

6일차
데이터와 Label을 이용해
Model이 판단 기준을 Weight로 학습
```

하지만 큰 Pipeline 사고방식은 같습니다.

```text
입력
→ 처리
→ 판단
→ 출력
→ 기록
```

---

# 63. 다음 날 연결 — PyTorch Baseline에서 ONNX Edge AI로

6일차 종료 시 PC에 반드시 있어야 하는 핵심 파일:

```text
models/day06_tiny_cnn.pt
```

그리고 이 Model이 어떤 입력을 기대하는지 알고 있어야 합니다.

```text
Classes
→ ['normal', 'warning_red']

Input
→ RGB
→ 96 × 96
→ Tensor
→ 0.0 ~ 1.0

Model
→ TinyCNN

Checkpoint
→ model_state
→ class_to_idx
→ classes
→ image_size
```

7일차는 다음 순서로 진행합니다.

```text
models/day06_tiny_cnn.pt
        ↓
PyTorch Prediction 재확인
        ↓
ONNX Export
        ↓
ONNX Model Check
        ↓
Class / Input Size Metadata 저장
        ↓
같은 Tensor로
PyTorch vs ONNX 결과 비교
        ↓
Raspberry Pi 전송
        ↓
ONNX Runtime CPU Inference
        ↓
Camera
        ↓
AI Class + Confidence
        ↓
NORMAL / WARNING
        ↓
GPIO / Event Log
```

5일차와 비교하면 다음 한 부분이 교체됩니다.

```text
5일차

Camera
→ HSV / Red Area Rule
→ Decision
→ GPIO / Log


7일차

Camera
→ ONNX Tiny CNN
→ Decision
→ GPIO / Log
```

6일차가 끝났을 때 다음 문장을 자신의 말로 설명할 수 있으면 다음 수업으로 넘어갈 준비가 된 것입니다.

> **“5일차에서는 사람이 HSV·면적 기준을 직접 정해 판단했고, 6일차에서는 같은 `normal / warning_red` 문제를 Session 단위 Dataset으로 만들고 Tiny CNN을 학습해 PyTorch Baseline을 고정했다. 7일차에는 이 Checkpoint의 Class 순서와 96×96 입력 조건을 유지한 채 ONNX로 변환해 Raspberry Pi에 배포한다.”**
