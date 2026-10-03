
## 오늘의 핵심 질문

> 같은 실제 이미지를 현실적인 범위에서 변형하면 어떤 학습데이터를 만들 수 있고, VAE는 이미지의 특징을 어떻게 압축하고 다시 복원할까?

다음 질문에 답할 수 있도록 데이터를 나누고, 만들고, 확인하고, 기록하는 전체 흐름을 경험하는 것이 목표입니다.

```text
무엇을 학습용으로 사용할 것인가?
        ↓
어떤 변형이 현실적인가?
        ↓
생성한 데이터는 정상적인가?
        ↓
어떤 원본에서 어떻게 만들었는가?
        ↓
평가용 데이터가 학습에 섞이지 않았는가?
```

---

# 오늘의 수업 목표

수업이 끝나면 다음 내용을 설명하고 직접 실행할 수 있어야 합니다.

- 실제 이미지와 Augmented Image의 차이를 설명할 수 있습니다.
- `Apple Red 1` 데이터의 Training과 Test를 구분하여 사용할 수 있습니다.
- Training 데이터에서 Train과 Validation을 다시 나누고, 기존 Test는 평가용으로 유지할 수 있습니다.
- Data Leakage가 무엇인지 설명할 수 있습니다.
- Brightness·Rotation·Blur·Noise가 어떤 촬영 조건을 표현하는지 설명할 수 있습니다.
- 이미지 한 장에 Augmentation을 적용하고 결과를 눈으로 비교할 수 있습니다.
- Train 이미지에만 여러 Augmentation을 적용할 수 있습니다.
- 생성 이미지의 원본·방법·파라미터·Seed를 Metadata로 기록할 수 있습니다.
- VAE의 `Encoder → Latent Space → Decoder → Reconstruction` 흐름을 설명할 수 있습니다.
- VAE를 직접 학습하고 Train Loss와 Validation Loss를 확인할 수 있습니다.
- Test 이미지의 Original과 Reconstruction을 비교할 수 있습니다.
- Latent Space에서 무작위로 Sampling한 이미지가 어떻게 생성되는지 확인할 수 있습니다.
- Automatic QA와 Human QA의 차이를 설명할 수 있습니다.
- Train·Validation·Test 사이의 Leakage를 코드로 확인할 수 있습니다.
- Mini Challenge에서 `Banana 1`을 이용하여 오늘의 전체 Pipeline을 스스로 다시 수행할 수 있습니다.
- Mini Challenge에서 한 번에 하나의 조건만 바꾸고 결과 차이를 설명할 수 있습니다.
- VAE 조건은 Validation으로 비교하고, Test는 선택된 최종 조건의 마지막 확인에만 사용할 수 있습니다.
- 오늘 만든 코드와 실험 결과를 README와 Git에 정리할 수 있습니다.

---

# 오늘 수업의 전체 흐름

![[Pasted image 20260908143606.png]]

오늘은 가지고 있는 이미지를 이용하여 학습 데이터를 더 다양하게 만드는 두 가지 방법을 배웁니다.

첫 번째 방법은 Augmentation(데이터 증강) 입니다.

이미 가지고 있는 사과 이미지를 조금 밝게 하거나, 돌리거나, 흐리게 하거나, Noise를 넣어 
새로운 학습 이미지처럼 만드는 방법입니다.

```
원본 사과 이미지
        ↓
밝기 변경
회전
Blur
Noise
        ↓
조금씩 다른 사과 이미지 생성
```

두 번째 방법은 VAE입니다.

VAE는 이미지를 단순히 돌리거나 밝게 만드는 것이 아니라, 
여러 사과 이미지를 보면서 “사과 이미지에는 어떤 공통적인 특징이 있는가?”를 모델이 학습하도록 하는 방법입니다.

학습이 끝나면 입력한 이미지를 다시 복원해 보거나, 
학습한 특징을 이용하여 새로운 형태의 이미지를 만들어 볼 수 있습니다.

```
여러 사과 이미지
        ↓
VAE가 이미지 특징 학습
        ↓
특징을 압축하여 기억
        ↓
이미지 다시 복원
또는
새로운 이미지 생성
```

따라서 오늘은 같은 사과 데이터를 가지고 서로 다른 두 가지 데이터 생성 방법을 비교합니다.

![[Pasted image 20260924175947.png]]

```
                    Apple Red 1 이미지
                           ↓
                  학습용 Train 준비
                           ↓
             ┌─────────────┴─────────────┐
             │                           │
             ↓                           ↓
       실험 A                       실험 B

   Augmentation                      VAE

원본 이미지를 직접              이미지 특징을
조금씩 변형                     모델이 학습
             │                           │
             ↓                           ↓
   변형 이미지 생성              복원 이미지 확인
                                 생성 이미지 확인
             │                           │
             └─────────────┬─────────────┘
                           ↓
                    결과가 정상인지 확인
                           ↓
                    두 방법 결과 비교
```

여기서 가장 중요한 것은 두 실험을 억지로 하나로 연결하는 것이 아닙니다.

```
Augmentation 결과
        ↓
VAE 입력
```

처럼 사용하는 것이 아니라,

```
같은 원본 Train
        ├─ Augmentation 실험
        └─ VAE 실험
```

처럼 같은 데이터에서 출발하여 서로 다른 방법을 각각 경험해 보는 것입니다.

오늘 수업이 끝나면 다음 차이를 이해하는 것이 가장 중요합니다.

```
Augmentation
→ 사람이 정한 방법으로 기존 이미지를 변형한다.

VAE
→ 모델이 여러 이미지를 보고 특징을 학습한다.
```

처음부터 어려운 생성모델의 수식을 이해하는 것이 목표는 아닙니다.

오늘은 직접 이미지를 만들고 결과를 눈으로 비교하면서

> “기존 이미지를 변형하는 것과 모델이 이미지 특징을 학습하는 것은 어떻게 다른가?”

를 이해하는 것이 첫 번째 목표입니다.

---

# 오늘 사용할 실제 데이터

## 1. 본 실습 — Fruits-360 `Apple Red 1`

오늘의 기본 실습 데이터는 **빨간 사과 이미지**입니다.

현재 준비된 데이터는 약 600장 규모이며, Fruits-360의 원래 구조를 유지하여 사용합니다.

대표적인 구성은 다음과 같습니다.

```text
Apple Red 1

Training
→ 약 492장

Test
→ 약 164장
```

실제 수량은 제공된 파일을 코드로 다시 확인합니다.

오늘은 다음처럼 사용합니다.

```text
기존 Training
        ↓
Train + Validation로 분리

기존 Test
        ↓
그대로 유지
```

예를 들어 Training이 492장이라면:

```text
Training 492장
        ↓
Train 약 419장
Validation 약 73장

Test 164장
→ 그대로 유지
```

> 숫자를 외우는 것이 목적은 아닙니다. 실제 파일 수에 따라 코드가 자동으로 계산합니다.

### 왜 빨간 사과 데이터를 사용하나요?

`Apple Red 1`은 한 종류의 객체가 비슷한 배경과 조건으로 반복되어 있어 1일차 생성모델의 기본 원리를 확인하기 좋습니다.

```text
같은 종류의 객체
        +
조금씩 다른 회전·형태
        ↓
Augmentation 변화가 눈에 잘 보임
        ↓
VAE가 어떤 특징을 유지하고
어떤 세부정보를 잃는지 비교하기 쉬움
```

오늘은 이 데이터가 실제 제조라인 데이터와 똑같다고 가정하지 않습니다.

**통제된 교육 데이터로 생성·보강 원리를 먼저 익히는 것**이 목적입니다.

---

## 2. Mini Challenge — Fruits-360 `Banana 1`

Mini Challenge에서는 `Banana 1`을 사용합니다.

```text
본 실습
Apple Red 1
→ 빨간색
→ 둥근 형태

Mini Challenge
Banana 1
→ 노란색
→ 길고 휘어진 형태
```

파일 형식과 데이터 성격은 비슷하지만 객체의 색과 형태가 확실히 다릅니다.

따라서 본 실습 코드를 그대로 복사하여 끝내는 것이 아니라 다음 내용을 다시 판단해야 합니다.

```text
같은 Augmentation 범위를 사용해도 되는가?

VAE는 사과와 다른 형태의 특징도 복원하는가?

Latent Dimension을 바꾸면
복원 결과가 어떻게 달라지는가?
```

Mini Challenge도 Fruits-360의 Training/Test 구분을 유지합니다.

---

## 3. 오늘 데이터에는 별도 라벨 파일이 필요하지 않습니다

1일차의 핵심은 Classification이나 Detection 학습이 아닙니다.

따라서 다음 파일은 필요하지 않습니다.

```text
Class Label TXT
BBox
YOLO Label
Mask
Segmentation Polygon
```

오늘 필요한 것은 **이미지 자체**입니다.

---

# 오늘 배울 기술 스택

| 기술 | 오늘 하는 일 | 쉽게 말하면 |
|---|---|---|
| Python | 전체 실습 코드 작성 | 데이터를 나누고 만들고 기록 |
| pathlib | 파일·폴더 경로 처리 | 이미지 위치를 안전하게 관리 |
| OpenCV | 이미지 변형·읽기·저장 | 밝기·회전·Blur·Noise 적용 |
| NumPy | Noise 생성·배열 계산 | 픽셀 값을 숫자로 처리 |
| Pandas | Metadata·Loss·QA CSV 저장 | 실험 결과를 표로 기록 |
| Matplotlib | Augmentation 결과 비교 | 여러 이미지를 한 화면에서 확인 |
| Pillow | VAE 입력 이미지 읽기 | RGB 이미지 로드 |
| PyTorch | VAE 구현·학습 | Encoder와 Decoder 학습 |
| TorchVision | Resize·Tensor 변환·결과 저장 | VAE용 이미지 전처리 |
| Train / Validation / Test | 실험 기준 고정 | 학습과 평가 데이터 분리 |
| Data Leakage | 평가 오류 방지 | Test 정보가 Train에 섞이지 않게 관리 |
| Metadata | 생성 이력 기록 | 어떤 원본을 어떻게 바꿨는지 저장 |
| Automatic / Human QA | 합성 이미지 품질검수 | 코드 검사 + 사람이 눈으로 확인 |
| Git | 코드와 실험 기록 저장 | 오늘 작업을 재현 가능한 상태로 보관 |

---

# PART 1. 교과 11의 1일차 프로젝트와 실행환경을 준비합니다

---

# 1. 교과 11에서 1일차가 어디에 해당하는지 확인합니다

교과 11에서는 5일 동안 서로 다른 합성·보강 방법을 경험합니다.

```text
1일차
일반 Augmentation + VAE
        ↓
2일차
GAN + WGAN-GP + Diffusion
        ↓
3일차
Cut-Paste + BBox 자동 생성
        ↓
4일차
3D Rendering + Segmentation
        ↓
5일차
Keypoint Sequence Augmentation
```

오늘은 그 시작입니다.

처음부터 복잡한 생성모델로 들어가지 않고 다음 두 가지를 먼저 구분합니다.

```text
Augmentation
→ 기존 이미지를 직접 변형

VAE
→ 이미지 특징을 압축하고
  다시 복원하는 생성모델
```

오늘 만들어 놓은 데이터 관리 방식은 2~5일차에도 반복해서 사용합니다.

```text
데이터 준비
→ 생성·보강
→ QA
→ Metadata
→ Leakage 방지
→ 결과 비교
```

---

# 2. 교과 11 프로젝트 폴더를 만듭니다

전체 디렉토리 구조

```
subject11_synthetic_data/
│
├─ .venv/                     ← 교과 11 전체 공통 Python 환경
├─ requirements.txt           ← 교과 11 공통 라이브러리 목록
├─ README.md                  ← 교과 11 전체 설명
├─ .gitignore                 ← 교과 11 전체 Git 제외 규칙
│
├─ day01_image_aug_vae/       ← 오늘 작업
├─ day02_...                  ← 2일차
├─ day03_...                  ← 3일차
├─ day04_...                  ← 4일차
└─ day05_...                  ← 5일차
```

오늘은 `day01_image_aug_vae`만 만듭니다.  
2~5일차 폴더는 각 수업을 시작할 때 추가합니다.

WSL 또는 Ubuntu 터미널을 실행합니다.

```bash
cd ~/ai_vision
```

교과 11 프로젝트 폴더를 만듭니다.

```bash
mkdir -p subject11_synthetic_data
cd subject11_synthetic_data
```

현재 위치를 확인합니다.

```bash
pwd
```

다음과 비슷하면 됩니다.

```text
/home/사용자계정/ai_vision/subject11_synthetic_data
```

## 교과 11 전체에서 사용할 공통 파일을 만듭니다

다음 세 파일은 1일차만 사용하는 파일이 아니라 **교과 11 전체에서 공통으로 사용**합니다.

```bash
touch requirements.txt
touch README.md
touch .gitignore
```

각 파일의 역할은 다음과 같습니다.

```
requirements.txt
→ 교과 11에서 사용하는 Python 라이브러리 기록

README.md
→ 교과 11의 전체 실습 흐름과 실행방법 기록

.gitignore
→ 데이터·모델·가상환경 등
  Git에 저장하지 않을 파일 지정
```

## 1일차 작업 폴더를 만듭니다

이제 오늘 사용할 폴더를 만듭니다.

```bash
mkdir -p day01_image_aug_vae
cd day01_image_aug_vae
```

현재 위치를 확인합니다.

```bash
pwd
```

다음과 비슷하면 됩니다.

```
/home/사용자계정/ai_vision/subject11_synthetic_data/day01_image_aug_vae
```

## 1일차 실습에 필요한 폴더를 만듭니다

```bash
mkdir -p \
src \
scripts \
data/apple/source_train \
data/apple/source_test \
data/apple/split/train \
data/apple/split/val \
data/apple/split/test \
data/apple/augmented/train \
data/banana/source_train \
data/banana/source_test \
data/banana/split/train \
data/banana/split/val \
data/banana/split/test \
data/banana/augmented/train \
models/apple \
models/banana \
results/apple/augmentation \
results/apple/vae \
results/banana/augmentation \
results/banana/vae \
reports/apple \
reports/banana
```

1일차에서 작성할 파일도 미리 만듭니다.

```bash
touch day01_notes.md

touch src/day01_common.py

touch scripts/00_check_env.py
touch scripts/01_check_dataset.py
touch scripts/02_prepare_split.py
touch scripts/03_augment_preview.py
touch scripts/04_augment_train.py
touch scripts/05_train_vae.py
touch scripts/06_qa_images.py
touch scripts/07_check_leakage.py
```

## 현재 폴더 구조를 확인합니다

교과 11 Root로 한 단계 이동합니다.

```bash
cd ..
```

현재 위치를 확인합니다.

```bash
pwd
```

```
.../subject11_synthetic_data
```

구조를 확인합니다.

```bash
tree -L 4
```

`tree` 명령이 없다면 설치합니다.

```bash
sudo apt install tree
```

현재는 대략 다음과 같은 구조가 보이면 됩니다.

```
subject11_synthetic_data/
│
├─ requirements.txt
├─ README.md
├─ .gitignore
│
└─ day01_image_aug_vae/
    │
    ├─ data/
    │   ├─ apple/
    │   └─ banana/
    │
    ├─ models/
    │   ├─ apple/
    │   └─ banana/
    │
    ├─ reports/
    │   ├─ apple/
    │   └─ banana/
    │
    ├─ results/
    │   ├─ apple/
    │   └─ banana/
    │
    ├─ scripts/
    ├─ src/
    └─ day01_notes.md
```

### 지금 무엇을 한 것인가요?

```
교과 11 전체 프로젝트 생성
        ↓
subject11_synthetic_data
        ↓
교과 전체 공통 파일 준비
        ↓
1일차 작업 폴더 생성
        ↓
Apple 본 실습
+
Banana Mini Challenge
        ↓
data / models / results / reports 분리
```

`Apple Red 1` 본 실습과 `Banana 1` Mini Challenge의 데이터와 결과를 서로 다른 폴더에 저장하므로 나중에 실험 결과를 구분하고 비교하기 쉽습니다.

---

# 3. Python 가상환경을 만들고 필요한 라이브러리를 설치합니다

# 3. 교과 11 전체에서 사용할 Python 가상환경을 준비합니다

교과 11에서는 **매일 새로운 가상환경을 만들지 않습니다.**

오늘 프로젝트 Root에 `.venv`를 **한 번만 만들고**, 2~5일차에는 이 환경을 다시 활성화하여 사용합니다.

```
교과 11 Root
        ↓
      .venv
        ↓
 ┌──────┼──────┬──────┬──────┐
 ↓      ↓      ↓      ↓      ↓
1일차   2일차   3일차   4일차   5일차
```

새로운 라이브러리가 필요한 날에는 기존 `.venv`에 필요한 패키지만 추가합니다.

---

## 현재 위치를 확인합니다

현재 위치가 교과 11 Root인지 확인합니다.

```bash
pwd
```

다음과 비슷해야 합니다.

```
/home/사용자계정/ai_vision/subject11_synthetic_data
```

> `day01_image_aug_vae` 안에서 가상환경을 만들지 않습니다.

---

## 교과 11 공통 가상환경을 만듭니다

```bash
python3 -m venv .venv
```

구조는 다음과 같이 됩니다.

```
subject11_synthetic_data/
│
├─ .venv/
├─ requirements.txt
├─ README.md
├─ .gitignore
│
└─ day01_image_aug_vae/
```

`.venv`와 `day01_image_aug_vae`가 **같은 단계에 있는 것**이 중요합니다.

---

## 가상환경을 활성화합니다

```bash
source .venv/bin/activate
```

정상적으로 활성화되면 터미널 앞부분에 다음처럼 표시될 수 있습니다.

```bash
(.venv) 사용자계정@PC:...
```

Python이 어느 위치에서 실행되는지도 확인합니다.

```bash
which python
```

다음과 비슷하면 정상입니다.

```
/home/사용자계정/ai_vision/subject11_synthetic_data/.venv/bin/python
```

즉,

```
day01_image_aug_vae/.venv
```

가 아니라

```
subject11_synthetic_data/.venv
```

를 사용하고 있어야 합니다.

---

## 공통 `requirements.txt`를 작성합니다

프로젝트 Root의 `requirements.txt`에 1일차에서 필요한 라이브러리를 작성합니다.

```
numpy
pandas
matplotlib
opencv-python
Pillow
torch
torchvision
```

현재 파일 위치는 다음입니다.

```
subject11_synthetic_data/
└─ requirements.txt
```

1일차에서는 위 라이브러리부터 설치합니다.

2~5일차에서 추가 라이브러리가 필요하면 **이 파일에 추가하면서 같은 환경을 계속 사용**합니다.

---

## 필요한 라이브러리를 설치합니다

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

설치가 끝난 뒤 주요 패키지가 보이는지 간단히 확인할 수 있습니다.

```bash
pip list
```

---

## 현재 환경을 1일차 기록으로 저장합니다

1일차에서 사용한 Python 환경을 나중에 확인할 수 있도록 기록합니다.

```bash
pip freeze > day01_image_aug_vae/reports/environment_freeze.txt
```

파일은 다음 위치에 만들어집니다.

```
day01_image_aug_vae/
└─ reports/
   └─ environment_freeze.txt
```

> `environment_freeze.txt`에는 오늘 실제로 설치되어 있던 패키지와 버전이 기록됩니다.  
> 이후 문제가 생겼을 때 1일차 실행환경을 확인하거나 같은 환경을 다시 구성할 때 참고할 수 있습니다.

## 이제 1일차 실습 폴더로 이동합니다

가상환경은 교과 11 Root에 만들어졌습니다.

이제 실제 1일차 코드를 실행하기 위해 `day01_image_aug_vae` 폴더로 이동합니다.

```bash
cd day01_image_aug_vae
```

현재 위치를 확인합니다.

```bash
pwd
```

다음과 비슷하면 됩니다.

```bash
/home/사용자계정/ai_vision/subject11_synthetic_data/day01_image_aug_vae
```

---

# 4. Python·PyTorch·GPU 환경을 확인하는 코드를 작성합니다

먼저 Python과 주요 라이브러리가 정상적으로 설치되어 있고, PyTorch가 GPU를 사용할 수 있는지 확인합니다.

이 확인을 먼저 해두면 뒤에서 오류가 발생했을 때 **환경 문제인지 코드 문제인지** 구분하기 쉬워집니다.

**파일: `scripts/00_check_env.py`**

```python
import sys

import cv2
import numpy as np
import pandas as pd
import torch
import torchvision
from PIL import Image


def main() -> None:
    print("=" * 60)
    print("Subject 11 - Day 01 Environment Check")
    print("=" * 60)

    print("Python      :", sys.version.split()[0])
    print("OpenCV      :", cv2.__version__)
    print("NumPy       :", np.__version__)
    print("Pandas      :", pd.__version__)
    print("PyTorch     :", torch.__version__)
    print("TorchVision :", torchvision.__version__)
    print("Pillow      :", Image.__version__)

    print("-" * 60)

    cuda_available = torch.cuda.is_available()

    print("CUDA Available :", cuda_available)

    if cuda_available:
        print("GPU            :", torch.cuda.get_device_name(0))
    else:
        print("GPU            : CPU mode")

    print("=" * 60)
    print("Environment check completed.")


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

본격적인 실습을 시작하기 전에 Python과 주요 라이브러리, GPU가 정상적으로 준비되어 있는지 확인합니다.

- Augmentation(데이터 증강)  
  기존 이미지를 밝게 하거나, 회전시키거나, 흐리게 하는 등 조금씩 변형하여 학습 데이터를 
  다양하게 만드는 방법입니다.  
  Augmentation은 대부분 CPU에서도 충분히 실행할 수 있습니다.

- VAE(Variational Autoencoder)  
  여러 이미지를 보면서 공통적인 특징을 학습하고, 그 특징을 이용해 이미지를 다시 복원하거나 새로운 이미지를 만들어 보는 생성모델입니다.  
  VAE 학습은 계산량이 많기 때문에 GPU를 사용하면 훨씬 빠르게 실행할 수 있습니다.

따라서 실습 전에 다음 항목을 확인합니다.

```text
Python이 정상적으로 실행되는가?
        ↓
OpenCV · NumPy · Pandas가 설치되어 있는가?
        ↓
PyTorch · TorchVision이 정상인가?
        ↓
CUDA를 사용할 수 있는가?
        ↓
GPU가 정상적으로 인식되는가?
```

실행 결과는 다음과 같습니다.

```
Python      : 3.12.3
OpenCV      : 5.0.0
NumPy       : 2.5.3
Pandas      : 3.0.6
PyTorch     : 2.14.0+cu130
TorchVision : 0.29.0+cu130
Pillow      : 12.3.0

CUDA Available : True
GPU            : NVIDIA GeForce RTX 4080 SUPER
```

### 결과를 어떻게 해석하나요?

`CUDA Available : True`가 나왔으므로 PyTorch가 NVIDIA GPU를 사용할 수 있는 상태입니다.

또한 GPU가 다음과 같이 정상적으로 인식되었습니다.

```
NVIDIA GeForce RTX 4080 SUPER
```

즉 현재 환경에서는

```
Augmentation
→ CPU 또는 GPU 환경에서 실습 가능

VAE
→ GPU를 이용하여 빠르게 학습 가능
```

한 상태입니다.

> `Environment check completed.`가 출력되면 오늘 실습에 필요한 기본 실행환경이 정상적으로 준비된 것입니다.

### 의사코드로 읽어보기

```text
Python 버전을 확인한다
        ↓
OpenCV·NumPy·Pandas 버전을 확인한다
        ↓
PyTorch와 TorchVision을 확인한다
        ↓
CUDA 사용 가능 여부를 확인한다
        ↓
GPU가 있으면 GPU 이름을 출력한다
        ↓
환경 점검 완료
```

실행합니다.

```bash
python scripts/00_check_env.py
```

예:

```text
CUDA Available : True
GPU            : NVIDIA ...
Environment check completed.
```

CUDA가 `False`여도 오늘 실습 전체가 중단되는 것은 아닙니다.

```text
Augmentation
→ CPU 실행 가능

VAE
→ CPU 실행 가능
→ GPU가 있으면 더 빠름
```

---

# 5. Apple과 Banana에서 함께 사용할 공통 경로 코드를 작성합니다

오늘은 두 종류의 데이터를 사용합니다.

```
본 실습
→ Apple Red 1

Mini Challenge
→ Banana 1
```

두 데이터는 사용하는 폴더만 다르고, 이후 수행하는 작업은 거의 같습니다.

```
데이터 확인
→ Train / Validation / Test 분리
→ Augmentation
→ VAE
→ QA
→ Leakage 확인
```

따라서 Apple용 코드와 Banana용 코드를 따로 만들지 않고, 
데이터 이름만 바꾸면 같은 코드가 다른 폴더를 사용하도록 공통 경로 기능을 먼저 만듭니다.

### 의사코드로 읽어보기

```text
프로젝트 Root 위치를 찾는다
        ↓
사용할 이미지 확장자를 정의한다
        ↓
폴더에서 이미지 파일만 가져오는 함수를 만든다
        ↓
dataset 이름을 받는다
        ↓
data / results / reports / models 경로를 만든다
        ↓
필요한 폴더가 없으면 자동으로 생성한다
```


**파일: `src/day01_common.py`**

```python
from __future__ import annotations

from pathlib import Path


ROOT = Path(__file__).resolve().parents[1]

VALID_EXTENSIONS = {
    ".jpg",
    ".jpeg",
    ".png",
    ".bmp",
}


def list_images(directory: Path) -> list[Path]:
    if not directory.exists():
        return []

    return sorted(
        path
        for path in directory.iterdir()
        if path.is_file()
        and path.suffix.lower() in VALID_EXTENSIONS
    )


def dataset_paths(dataset: str) -> dict[str, Path]:
    data_root = ROOT / "data" / dataset
    result_root = ROOT / "results" / dataset
    report_root = ROOT / "reports" / dataset
    model_root = ROOT / "models" / dataset

    paths = {
        "source_train": data_root / "source_train",
        "source_test": data_root / "source_test",
        "split_train": data_root / "split" / "train",
        "split_val": data_root / "split" / "val",
        "split_test": data_root / "split" / "test",
        "augmented_train": data_root / "augmented" / "train",
        "result_aug": result_root / "augmentation",
        "result_vae": result_root / "vae",
        "report": report_root,
        "model": model_root,
    }

    for path in paths.values():
        path.mkdir(
            parents=True,
            exist_ok=True,
        )

    return paths
```

## 이 파일은 무엇을 하나요?

이 파일은 **실험을 직접 실행하는 파일이 아닙니다.**

Apple과 Banana를 사용할 때 필요한 폴더 위치를 다른 Python 파일들에게 알려주는 **공통 경로 도우미 파일**입니다.

쉽게 보면 다음 역할을 합니다.

```
다른 실습 코드
        ↓
apple 또는 banana라는 이름 전달
        ↓
day01_common.py
        ↓
어느 data / results / reports / models
폴더를 사용해야 하는지 결정
```

## 어떤 코드가 Apple과 Banana를 구분하나요?

가장 중요한 부분은 다음 함수입니다.

```python
def dataset_paths(dataset: str) -> dict[str, Path]:
```

여기서 `dataset`에는 나중에

```
apple
```

또는

```
banana
```

가 들어옵니다.

그리고 다음 코드가 실제 경로를 만듭니다.

```python
data_root = ROOT / "data" / dataset
result_root = ROOT / "results" / dataset
report_root = ROOT / "reports" / dataset
model_root = ROOT / "models" / dataset
```

예를 들어 `dataset`이 `apple`이면 다음처럼 바뀝니다.

```
data/apple
results/apple
reports/apple
models/apple
```

반대로 `dataset`이 `banana`이면 다음처럼 됩니다.

```
data/banana
results/banana
reports/banana
models/banana
```

즉 이 한 줄의 `dataset` 값이 **어느 데이터 폴더를 사용할지 결정하는 스위치 역할**을 합니다. 1일차_AppleRed1_Augmentation_VAE_…

---

## 그러면 `--dataset apple`은 어디에 있나요?

`--dataset apple`은 이 공통 파일 안에 있는 코드가 아닙니다.

뒤에서 실제로 실행하는 스크립트가 받습니다.

예를 들어 `scripts/01_check_dataset.py`에는 다음 코드가 있습니다.

```python
parser.add_argument(
    "--dataset",
    default="apple",
    choices=["apple", "banana"],
)
```

다음처럼 실행하면

```python
python scripts/01_check_dataset.py --dataset apple
```

`apple`이라는 값이 `args.dataset`에 들어갑니다.

그리고 다음 코드가 실행됩니다.

```python
paths = dataset_paths(args.dataset)
```

즉 전체 흐름은 다음과 같습니다.

```
python scripts/01_check_dataset.py --dataset apple
                    ↓
             args.dataset
                    ↓
                 "apple"
                    ↓
       dataset_paths("apple")
                    ↓
       data/apple
       results/apple
       reports/apple
       models/apple
```

Mini Challenge에서는 명령어만 바꿉니다.

```bash
python scripts/01_check_dataset.py --dataset banana
```

그러면:

```
--dataset banana
        ↓
args.dataset = "banana"
        ↓
dataset_paths("banana")
        ↓
data/banana
results/banana
reports/banana
models/banana
```

로 자동으로 바뀝니다. 실제 `01_check_dataset.py`가 `--dataset` 값을 받은 뒤 `dataset_paths(args.dataset)`를 호출하는 구조입니다. 1일차_AppleRed1_Augmentation_VAE_…

---

## `ROOT`는 무엇인가요?

다음 코드도 중요합니다.

```python
ROOT = Path(__file__).resolve().parents[1]
```

현재 파일은 다음 위치에 있습니다.

```
day01_image_aug_vae/
└─ src/
   └─ day01_common.py
```

따라서 `ROOT`는 한 단계 위의

```
day01_image_aug_vae/
```

가 됩니다.

그래서 모든 데이터 경로는 이 폴더를 기준으로 찾습니다.

```
day01_image_aug_vae/
├─ data/
├─ models/
├─ results/
├─ reports/
├─ scripts/
└─ src/
```

---

## `list_images()`는 무엇을 하나요?

다음 함수는 지정된 폴더 안에서 **이미지 파일만 찾아오는 역할**을 합니다.

```python
def list_images(directory: Path) -> list[Path]:
```

사용 가능한 이미지 확장자는 위에서 정해 놓았습니다.

```python
VALID_EXTENSIONS = {
    ".jpg",
    ".jpeg",
    ".png",
    ".bmp",
}
```

즉 폴더 안에 다른 파일이 있어도 이미지 파일만 골라서 가져옵니다.

```
폴더
├─ apple01.jpg   → 사용
├─ apple02.png   → 사용
├─ memo.txt      → 제외
└─ README.md     → 제외
```

---

## 이 공통 파일이 이후 어디에 사용되나요?

오늘 작성하는 여러 스크립트가 이 파일을 가져와 사용합니다.

```python
from src.day01_common import (
    dataset_paths,
    list_images,
)
```

예를 들어 이후 실습에서:

```
01_check_dataset.py
→ 입력 데이터 경로 확인

02_prepare_split.py
→ Train / Validation / Test 경로 결정

03_augment_preview.py
→ Augmentation 결과 저장 위치 결정

04_augment_train.py
→ 보강 이미지와 Metadata 저장 위치 결정

05_train_vae.py
→ VAE 데이터·모델·결과 경로 결정

06_qa_images.py
→ QA 결과 위치 결정

07_check_leakage.py
→ Leakage 검사 대상 경로 결정
```

처럼 반복해서 사용합니다.

### 핵심만 기억하면 됩니다

```
day01_common.py
→ Apple이나 Banana를 직접 학습하는 코드가 아님

dataset_paths(dataset)
→ 사용할 데이터 이름을 받아
  경로를 결정하는 공통 함수

--dataset apple / banana
→ 뒤의 실행 스크립트에서 입력

args.dataset
→ 공통 함수에 전달

결과
→ 같은 코드를 Apple과 Banana에서 재사용
```

---

# PART 2. Apple Red 1 데이터를 확인하고 공정한 실험 기준을 만듭니다

---

# 6. Apple Red 1 이미지를 입력 폴더에 넣습니다

이제 실제로 사용할 `Apple Red 1` 이미지를 준비합니다.

오늘은 Fruits-360에서 제공하는 데이터를 다음과 같이 사용합니다.

```
Fruits-360

Training / Apple Red 1
→ 오늘의 Training 원본

Test / Apple Red 1
→ 오늘의 Test 원본
```

이 두 데이터를 우리가 만든 프로젝트 폴더에 각각 복사합니다.

현재 프로젝트에서는 다음 위치를 사용합니다. 1일차_AppleRed1_Augmentation_VAE_…

```
data/apple/source_train/
→ 원래 Training 이미지

data/apple/source_test/
→ 원래 Test 이미지
```

## 현재 1일차 폴더에 있는지 확인합니다

터미널에서 현재 위치를 확인합니다.

```bash
pwd
```

다음과 비슷하면 됩니다.

```
/home/사용자계정/ai_vision/subject11_synthetic_data/day01_image_aug_vae
```

현재 WSL 폴더를 Windows 탐색기로 엽니다.

```bash
explorer.exe .
```

그러면 현재 작업 중인 `day01_image_aug_vae` 폴더가 Windows 파일 탐색기로 열립니다.

---

## Training 이미지를 넣습니다

Windows 탐색기에서 다음 폴더로 이동합니다.

```
day01_image_aug_vae
└─ data
   └─ apple
      └─ source_train
```

다른 탐색기 창에서 준비해 둔 Fruits-360 데이터의

```
Training
└─ Apple Red 1
```

폴더를 엽니다.

`Apple Red 1` 안에 있는 **이미지 파일 전체를 선택하여** 다음 위치에 복사합니다.

```
data/apple/source_train/
```

즉 결과는 다음과 같아야 합니다.

```
data/apple/source_train/
├─ 0_100.jpg
├─ 1_100.jpg
├─ 2_100.jpg
├─ ...
└─ Apple Red 1 Training 이미지
```

> `source_train/Apple Red 1/이미지`처럼 폴더를 한 단계 더 만들지 않습니다.  
> 이미지 파일들이 `source_train` 바로 아래에 있어야 합니다.

---

## Test 이미지를 넣습니다

이번에는 Fruits-360 데이터의

```
Test
└─ Apple Red 1
```

폴더를 엽니다.

그 안의 이미지 파일 전체를 다음 위치에 복사합니다.

```
data/apple/source_test/
```

결과는 다음과 같아야 합니다.

```
data/apple/source_test/
├─ 0_100.jpg
├─ 1_100.jpg
├─ 2_100.jpg
├─ ...
└─ Apple Red 1 Test 이미지
```

최종 구조는 다음과 같습니다.

```
data/apple/
│
├─ source_train/
│  ├─ 이미지
│  ├─ 이미지
│  └─ ...
│
└─ source_test/
   ├─ 이미지
   ├─ 이미지
   └─ ...
```

Training과 Test를 처음부터 따로 보관하는 이유는 학습에 사용할 이미지와 마지막 평가에 사용할 이미지를 섞지 않기 위해서입니다.

```
원래 Training
        ↓
나중에 Train + Validation으로 분리

원래 Test
        ↓
마지막 평가용으로 그대로 유지
```

> Fruits-360의 기존 Test 이미지는 Training 이미지와 섞지 않습니다.

이제 이미지 복사가 끝났다면 다음 단계의 `01_check_dataset.py`를 실행하여 파일이 실제로 들어갔는지, 정상적으로 읽히는지, 몇 장인지, 이미지 크기가 얼마인지 자동으로 확인합니다.


---

# 7. 입력 이미지가 정상인지 자동으로 확인하는 코드를 작성합니다

현재 흐름을 보면 
`00_check_env.py`는 Python·라이브러리·CUDA·GPU 같은 실행환경을 확인하는 코드이고, 
`01_check_dataset.py`는 Apple 이미지가 실제로 존재하고 읽을 수 있는지, Training/Test가 몇 장인지, 대표 이미지 크기가 얼마인지 확인하는 입력 데이터 점검 코드입니다.

앞에서 `00_check_env.py`를 이용하여 Python과 주요 라이브러리, GPU가 정상적으로 준비되어 있는지 확인했습니다.

```
00_check_env.py
→ 실행환경 확인

Python
라이브러리
PyTorch
CUDA
GPU
```

이제는 컴퓨터 환경이 아니라 **오늘 실제로 사용할 Apple 이미지 데이터**를 확인합니다.

```
01_check_dataset.py
→ 입력 데이터 확인

Training 이미지
Test 이미지
파일 수
읽기 가능 여부
이미지 크기
```

즉 두 파일의 역할은 다음처럼 구분합니다.

```
00번
"내 컴퓨터가 실습할 준비가 되었는가?"

        ↓

01번
"내가 사용할 이미지 데이터가
실습할 준비가 되었는가?"
```

## 왜 데이터 확인을 먼저 하나요?

이후 실습에서는 Apple 이미지를 이용하여 Train·Validation을 만들고, Augmentation과 VAE 학습을 진행합니다.

그런데 입력 데이터 자체에 문제가 있다면 이후 코드가 정상이어도 실습이 제대로 진행되지 않습니다.

따라서 본격적인 실험 전에 다음 네 가지를 먼저 확인합니다.

```
1. 이미지 파일이 실제로 있는가?
        ↓
2. OpenCV가 이미지를 정상적으로 읽을 수 있는가?
        ↓
3. Training과 Test가 각각 몇 장인가?
        ↓
4. 이미지의 Width·Height는 얼마인가?
```

각 확인에는 이유가 있습니다.

```
이미지가 없음
→ 이후 Split이나 학습을 진행할 수 없음

이미지가 깨져 있어 읽히지 않음
→ Augmentation이나 VAE 실행 중 오류 가능

Training / Test 수량 확인
→ 이후 Train / Validation 분리 기준이 됨

이미지 크기 확인
→ 이후 이미지 전처리와 모델 입력 크기를 이해하는 기준이 됨
```

따라서 이 단계의 목적은 모델을 학습하는 것이 아닙니다.

> “지금 준비한 Apple 데이터로 다음 실습을 시작해도 되는가?”를 확인하는 단계입니다.

---

## 정상적으로 확인되면 무엇을 얻게 되나요?

예를 들어 다음과 같은 결과가 나온다면

```
[SOURCE TRAIN]
Count: 492

[SOURCE TEST]
Count: 164

Total: 656

Dataset check: READY
```

다음 내용을 확인한 것입니다.

```
Apple Training 이미지가 존재함
        +
Apple Test 이미지가 존재함
        +
파일이 정상적으로 읽힘
        +
Training / Test 수량 확인 완료
        +
대표 이미지 크기 확인 완료
        ↓
다음 단계 진행 가능
```

`Dataset check: READY`는 입력 데이터의 기본 점검이 완료되었다는 뜻입니다.

다만 이것이 “좋은 학습 데이터라는 것이 완전히 검증되었다”는 뜻은 아닙니다.

현재 단계에서는 파일 존재 여부와 읽기 가능 여부, 수량, 기본 크기만 확인합니다. 

이후에 데이터 분리, Augmentation 결과, QA, Leakage 등을 차례로 다시 확인합니다. 

실제 자료도 뒤에서 Training을 Train/Validation으로 분리하고 Test를 유지하는 단계로 이어집니다. 

---

## 앞에서 만든 공통 경로 파일과 연결됩니다

이제 여기서 처음으로 앞에서 작성한 `day01_common.py`를 실제로 사용합니다.

```
day01_common.py
→ 데이터가 어디 있는지 알려줌

01_check_dataset.py
→ 그 위치의 데이터를 실제로 검사함
```

즉 지금 단계에서 이 정도만 이해하면 충분합니다.

```
day01_common.py
Apple 데이터는 여기 있어

        ↓

01_check_dataset.py
그럼 실제로 파일이 있는지..
읽히는지..
몇 장인지 확인해 볼게
```

**파일: `scripts/01_check_dataset.py`**

현재 `01_check_dataset.py`는 `day01_common.py`의 `dataset_paths()`와 `list_images()`를 가져와서 Apple 데이터의 위치를 찾고 이미지 목록을 읽습니다.

```python
from __future__ import annotations

import argparse
import sys
from pathlib import Path

import cv2


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day01_common import (
    dataset_paths,
    list_images,
)


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--dataset",
        default="apple",
        choices=["apple", "banana"],
    )

    return parser.parse_args()


def check_group(name, files):
    unreadable = []

    print()
    print(f"[{name}]")
    print("Count:", len(files))

    for path in files:
        image = cv2.imread(str(path))

        if image is None:
            unreadable.append(path.name)

    if files:
        first = cv2.imread(str(files[0]))

        if first is not None:
            height, width = first.shape[:2]

            print(
                "First image size:",
                f"{width} x {height}",
            )

    if unreadable:
        print("Unreadable:", len(unreadable))

    return unreadable


def main() -> None:
    args = parse_args()
    paths = dataset_paths(args.dataset)

    train_files = list_images(
        paths["source_train"]
    )

    test_files = list_images(
        paths["source_test"]
    )

    if not train_files:
        raise RuntimeError(
            "source_train 이미지가 없습니다."
        )

    if not test_files:
        raise RuntimeError(
            "source_test 이미지가 없습니다."
        )

    train_bad = check_group(
        "SOURCE TRAIN",
        train_files,
    )

    test_bad = check_group(
        "SOURCE TEST",
        test_files,
    )

    if train_bad or test_bad:
        raise RuntimeError(
            "읽을 수 없는 이미지가 있습니다."
        )

    print()
    print(
        "Total:",
        len(train_files) + len(test_files),
    )

    print("Dataset check: READY")


if __name__ == "__main__":
    main()
```

### 의사코드로 읽어보기

```text
apple 또는 banana를 선택한다
        ↓
source_train 이미지 목록을 가져온다
        ↓
source_test 이미지 목록을 가져온다
        ↓
각 이미지를 OpenCV로 읽어본다
        ↓
읽히지 않는 파일이 있는지 확인한다
        ↓
Training / Test 수량을 출력한다
        ↓
대표 이미지의 크기를 확인한다
        ↓
모두 정상이면 READY 출력
```

실행합니다.

```bash
python scripts/01_check_dataset.py \
  --dataset apple
```

예상되는 형태:

```text
[SOURCE TRAIN]
Count: 492

[SOURCE TEST]
Count: 164

Total: 656

Dataset check: READY
```

실제 숫자는 현재 가지고 있는 파일 수를 기준으로 확인합니다.

---

# 8. Train·Validation·Test의 역할을 구분합니다

앞에서 Apple 원본 데이터를 다음과 같이 준비했습니다.

```
data/apple/source_train/
→ Fruits-360의 원래 Training 이미지

data/apple/source_test/
→ Fruits-360의 원래 Test 이미지
```

이제 원래 Training 이미지를 다시 두 용도로 나눕니다.

```
원래 Training
        ↓
┌───────────────┬───────────────┐
│                               │
↓                               ↓
Train                        Validation
약 85%                       약 15%
│                               │
모델 학습                    학습 상태 확인


원래 Test
        ↓
Test 그대로 유지
        ↓
마지막 결과 확인
```

쉽게 구분하면 다음과 같습니다.

```
Train
→ 모델이 실제로 학습하는 데이터

Validation
→ 학습이 잘 되고 있는지 중간에 확인하는 데이터

Test
→ 모든 조건을 정한 뒤 마지막에 확인하는 데이터
```

따라서 오늘은 Fruits-360의 기존 Test는 그대로 보호하고, 기존 Training 안에서만 Train과 Validation을 나눕니다.

---

# 9. Data Leakage를 이해합니다

데이터를 나누기 전에 먼저 Augmentation을 하면 문제가 생길 수 있습니다.

예를 들어 다음과 같은 상황입니다.

```
원본 apple_A
→ Train

apple_A를 회전한 이미지
→ Test
```

두 이미지는 사실상 같은 사과에서 만들어진 매우 비슷한 데이터입니다.

이런 정보가 Train과 Test에 동시에 들어가면 모델이 이미 비슷한 이미지를 학습했는데도, 처음 보는 데이터를 잘 처리한 것처럼 결과가 나올 수 있습니다.

이처럼 학습에 사용된 정보가 평가 데이터에 섞이는 문제를 `Data Leakage`라고 합니다.

그래서 오늘은 반드시 다음 순서를 지킵니다.

```
먼저 데이터 분리
        ↓
Train 확정
Validation 확정
Test 확정
        ↓
Train에만 Augmentation 적용
```

> 먼저 나누고, 그다음 Train만 보강한다.

이 원칙을 기억하면 됩니다.

---

# 10. Training을 Train·Validation으로 나누고 Test를 고정하는 코드를 작성합니다

이제 앞에서 이해한 데이터 분리 원칙을 실제로 적용합니다.

이번에 작성할 `02_prepare_split.py`는 **Apple 원본 데이터를 실제 실험에 사용할 세 그룹으로 나누어 주는 코드**입니다.

이 코드를 실행하면 다음 작업이 이루어집니다.

```
source_train
        ↓
약 85% → Train
약 15% → Validation

source_test
        ↓
Test로 그대로 유지
```

왜 이 작업이 필요할까요?

앞으로 진행할 Augmentation과 VAE 실습에서 각 데이터의 역할을 섞지 않기 위해서입니다.

```
Train
→ Augmentation
→ VAE 학습

Validation
→ 학습 상태와 조건 확인

Test
→ 마지막 결과 확인
```

또한 같은 실험을 다시 실행했을 때 비슷한 기준으로 데이터를 나눌 수 있도록 **Seed를 고정**하고, 어떤 이미지가 어느 그룹에 들어갔는지도 기록합니다.

즉 이번 코드의 목적은 다음 한 문장으로 정리할 수 있습니다.

> “학습용·중간 확인용·최종 평가용 데이터를 먼저 확실하게 분리하여 이후 실험의 기준을 만드는 코드”입니다.

이제 실제 코드를 작성합니다.

**파일: `scripts/02_prepare_split.py`**

```python
from __future__ import annotations

import argparse
import csv
import random
import shutil
import sys
from pathlib import Path


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day01_common import (
    dataset_paths,
    list_images,
)


SEED = 42
VAL_RATIO = 0.15


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--dataset",
        default="apple",
        choices=["apple", "banana"],
    )

    parser.add_argument(
        "--seed",
        type=int,
        default=SEED,
    )

    parser.add_argument(
        "--val-ratio",
        type=float,
        default=VAL_RATIO,
    )

    return parser.parse_args()


def reset_directory(directory: Path) -> None:
    if directory.exists():
        shutil.rmtree(directory)

    directory.mkdir(
        parents=True,
        exist_ok=True,
    )


def copy_files(
    files,
    target_dir,
    split_name,
    rows,
):
    for source_path in files:
        target_path = (
            target_dir
            / source_path.name
        )

        shutil.copy2(
            source_path,
            target_path,
        )

        rows.append(
            {
                "file": source_path.name,
                "source_group": (
                    source_path.parent.name
                ),
                "split": split_name,
            }
        )


def main() -> None:
    args = parse_args()

    if not 0 < args.val_ratio < 1:
        raise ValueError(
            "--val-ratio는 0보다 크고 1보다 작아야 합니다."
        )

    paths = dataset_paths(
        args.dataset
    )

    source_train = list_images(
        paths["source_train"]
    )

    source_test = list_images(
        paths["source_test"]
    )

    if len(source_train) < 10:
        raise RuntimeError(
            "source_train 이미지가 너무 적습니다."
        )

    if not source_test:
        raise RuntimeError(
            "source_test 이미지가 없습니다."
        )

    rng = random.Random(
        args.seed
    )

    shuffled = source_train.copy()
    rng.shuffle(shuffled)

    val_count = int(
        len(shuffled)
        * args.val_ratio
    )

    val_count = max(
        1,
        min(
            val_count,
            len(shuffled) - 1,
        ),
    )

    val_files = shuffled[:val_count]
    train_files = shuffled[val_count:]
    test_files = source_test

    reset_directory(
        paths["split_train"]
    )

    reset_directory(
        paths["split_val"]
    )

    reset_directory(
        paths["split_test"]
    )

    rows = []

    copy_files(
        train_files,
        paths["split_train"],
        "train",
        rows,
    )

    copy_files(
        val_files,
        paths["split_val"],
        "val",
        rows,
    )

    copy_files(
        test_files,
        paths["split_test"],
        "test",
        rows,
    )

    report_path = (
        paths["report"]
        / "split_manifest.csv"
    )

    with report_path.open(
        "w",
        newline="",
        encoding="utf-8-sig",
    ) as file:
        writer = csv.DictWriter(
            file,
            fieldnames=[
                "file",
                "source_group",
                "split",
            ],
        )

        writer.writeheader()
        writer.writerows(rows)

    print("=" * 60)
    print("Dataset Split Completed")
    print("=" * 60)

    print("Dataset :", args.dataset)
    print("Train   :", len(train_files))
    print("Val     :", len(val_files))
    print("Test    :", len(test_files))
    print("Seed    :", args.seed)

    print()
    print("Manifest:", report_path)


if __name__ == "__main__":
    main()
```

### 의사코드로 읽어보기

```text
Validation 비율이 0과 1 사이인지 확인한다
        ↓
원래 Training 이미지를 읽는다
        ↓
Seed 42로 순서를 섞는다
        ↓
약 15%를 Validation으로 선택한다
        ↓
Validation이 최소 1장,
Train도 최소 1장 남도록 보정한다
        ↓
나머지를 Train으로 사용한다
        ↓
원래 Test는 그대로 Test로 복사한다
        ↓
Train / Val / Test 폴더를 만든다
        ↓
각 파일의 Split 정보를 CSV로 기록한다
```

실행합니다.

```bash
python scripts/02_prepare_split.py \
  --dataset apple
```

실제 실행 결과는 다음과 같습니다.

```text
============================================================
Dataset Split Completed
============================================================
Dataset : apple
Train   : 419
Val     : 73
Test    : 164
Seed    : 42

Manifest: .../reports/apple/split_manifest.csv
```

### 실행 결과를 해석합니다

원래 Apple 데이터는 다음과 같았습니다.

```
Training : 492장
Test     : 164장
```

`02_prepare_split.py`를 실행한 결과 기존 Training 492장이 다음처럼 나뉘었습니다.

```
원래 Training 492장
        ↓
┌─────────────────┬─────────────────┐
│                                   │
↓                                   ↓
Train                            Validation
419장                            73장

원래 Test 164장
        ↓
Test 164장 그대로 유지
```

즉,

```
419 + 73 = 492
```

이므로 기존 Training 이미지가 빠지지 않고 Train과 Validation으로 나누어진 것을 확인할 수 있습니다.

`Test : 164`도 원래 Test 수량과 같으므로 기존 Test를 Training에 섞지 않고 그대로 유지한 상태입니다.

---

## 이 코드는 무엇을 입력받고 무엇을 만들었나요?

이번 코드는 다음 흐름으로 이해하면 됩니다.

```
[입력]

--dataset apple
        +
data/apple/source_train/
492장
        +
data/apple/source_test/
164장

        ↓

[처리]

source_train을 Seed 42로 섞음
        ↓
약 85% → Train
약 15% → Validation
        ↓
기존 source_test는 그대로 Test로 사용

        ↓

[출력]

data/apple/split/train/
→ 419장

data/apple/split/val/
→ 73장

data/apple/split/test/
→ 164장

reports/apple/split_manifest.csv
→ 각 이미지가 어디에 배치되었는지 기록
```

여기서 `Seed : 42`는 같은 데이터로 다시 실행했을 때 같은 방식으로 Train과 Validation을 나눌 수 있도록 사용하는 **재현 기준값**입니다.

---

## Manifest를 확인합니다

```
head reports/apple/split_manifest.csv
```

실제 결과:

```
file,source_group,split
266_100.jpg,source_train,train
264_100.jpg,source_train,train
r_222_100.jpg,source_train,train
288_100.jpg,source_train,train
269_100.jpg,source_train,train
...
```

각 열은 다음 의미입니다.

|항목|의미|
|---|---|
|`file`|이미지 파일명|
|`source_group`|원래 어느 데이터에서 왔는지|
|`split`|최종적으로 Train / Val / Test 중 어디에 들어갔는지|

예를 들어

```
266_100.jpg,source_train,train
```

은

```
266_100.jpg
→ 원래 source_train 이미지
→ 이번 Split에서 train으로 배치
```

되었다는 뜻입니다.

`head` 명령은 CSV의 **맨 앞부분만 보여주기 때문에 현재 출력에 `train`만 보이는 것은 정상**입니다. Validation과 Test가 없는 것이 아닙니다.

---

## 문제가 생기면 무엇을 확인하나요?

이 코드에서는 다음 상황이 발생하면 정상적으로 Split을 진행하지 않습니다.

```
source_train 이미지가 너무 적음
→ Training 데이터 배치를 다시 확인

source_test 이미지가 없음
→ data/apple/source_test/ 확인

Validation 비율이 잘못됨
→ 0보다 크고 1보다 작은 값인지 확인
```

정상 실행되면 반드시 다음 문구가 표시됩니다.

```
Dataset Split Completed
```

그리고 Train·Val·Test 수량과 `split_manifest.csv` 위치가 함께 출력됩니다.

---

### 오늘 Split 결과에서 기억할 핵심

```
원래 Training
→ Train 419
→ Validation 73

원래 Test
→ Test 164 그대로 유지

이후
Train만 Augmentation
Validation은 중간 확인
Test는 마지막 확인
```

> **데이터를 먼저 나누고, 그다음 Train에만 Augmentation을 적용합니다.**

---

# 11. 오늘 사용할 실험 기준을 확정합니다

앞에서 Apple 데이터를 다음과 같이 나누었습니다.

```text
Train      : 419장
Validation : 73장
Test       : 164장
Seed       : 42
```

이제부터는 이 데이터 구성을 오늘 실험의 기준으로 그대로 사용합니다.

즉 이후 Augmentation이나 VAE 조건을 바꾸더라도, 데이터 자체는 다시 나누지 않습니다.

```text
같은 Train
같은 Validation
같은 Test
같은 Seed

        ↓

Augmentation 조건 변경
VAE 조건 변경

        ↓

결과 차이 비교
```

왜 이렇게 할까요?

데이터까지 매번 달라지면 결과가 달라졌을 때

```
데이터가 달라서 그런 것인지
?
설정값이 달라서 그런 것인지
```

구분하기 어렵기 때문입니다.

따라서 오늘은 데이터 구성은 고정하고, 실험 조건만 바꾸면서 결과를 비교합니다.

> 이처럼 비교의 출발점이 되는 기준 상태를 오늘 수업에서는 `Baseline`으로 이해합니다.

오늘의 Baseline은 다음과 같습니다.

```
Apple Red 1
+
Train / Validation / Test 고정
+
Seed 42
+
아직 Augmentation 조건이나 VAE 조건을 변경하기 전 상태
```

이 기준을 잡아두고 다음 실습부터 실제 이미지 변형과 VAE 실험을 시작합니다.

---

# PART 3. Apple 이미지 한 장으로 Augmentation을 이해합니다

앞에서는 Apple 데이터를 Train·Validation·Test로 나누고 오늘 사용할 데이터 기준을 확정했습니다.

이제부터는 그중 **Train 이미지에 실제 변형을 적용해 보는 실습**을 시작합니다.

하지만 처음부터 Train 419장 전체를 한꺼번에 변형하지 않습니다.

먼저 이미지 한 장만 사용하여

```text
원본 이미지
        ↓
Brightness
Rotation
Blur
Noise
        ↓
변형 결과 확인
        ↓
이 정도 변형이 실제 데이터로 사용해도 괜찮은지 판단
```

하는 과정을 먼저 진행합니다.

즉 PART 3의 목적은

> Augmentation을 많이 만드는 것이 아니라, 어떤 변형을 적용했을 때 원래 사과의 의미가 유지되는지 직접 눈으로 확인하는 것입니다.

이 Preview가 괜찮다고 판단되면 뒤에서 Train 전체에 같은 종류의 Augmentation을 적용합니다.

---

# 12. Augmentation이 무엇인지 이해합니다

`Augmentation`은 기존 이미지를 조금씩 변형하여 학습 데이터의 조건을 다양하게 만드는 방법입니다.

예를 들어 실제 촬영 환경에서는 항상 똑같은 사진만 얻어지지 않습니다.

```
조명이 달라질 수 있음
→ Brightness

카메라나 객체의 각도가 달라질 수 있음
→ Rotation

초점이나 움직임 때문에 흐려질 수 있음
→ Blur

센서나 촬영환경에서 잡음이 생길 수 있음
→ Noise
```

오늘은 다음 네 가지 변형을 사용합니다.

|방법|쉽게 말하면|
|---|---|
|`Brightness`|이미지를 조금 밝거나 어둡게 만듦|
|`Rotation`|이미지를 조금 회전시킴|
|`Blur`|이미지를 조금 흐리게 만듦|
|`Noise`|픽셀에 작은 잡음을 추가함|

중요한 것은 강하게 변형하는 것이 목적이 아니라는 점입니다.

```
강한 변형
≠
좋은 Augmentation

실제 촬영에서 일어날 수 있는 변형
=
좋은 Augmentation 후보
```

예를 들어 사과 이미지를 너무 어둡게 만들어 형태가 보이지 않거나, 너무 많이 회전시켜 사과가 잘려 버리면 학습 데이터로 사용하기 어려울 수 있습니다.

그래서 먼저 한 장에서 결과를 확인합니다.

---

# 13. 이미지 한 장으로 Augmentation Preview를 만드는 코드를 작성합니다

이제 실제 Train 이미지 한 장을 가져와 네 가지 변형을 적용해 봅니다.

이번에 작성할 `03_augment_preview.py`는 대량의 데이터를 만드는 코드가 아니라, Augmentation 결과를 미리 시험해 보는 코드입니다.

이 코드는 다음 과정을 수행합니다.

```
Train 이미지 한 장 선택
        ↓
원본 그대로 확인
        ↓
Brightness 적용
Rotation 적용
Blur 적용
Noise 적용
        ↓
각 결과 저장
        ↓
한 화면에서 비교할 수 있는 Grid 저장
```

왜 먼저 한 장만 확인할까요?

만약 변형 조건이 너무 강하거나 부자연스러운데 바로 수백 장을 생성하면, 잘못된 데이터도 
함께 많이 만들어지기 때문입니다.

따라서 순서는 다음과 같습니다.

```
한 장으로 Preview
        ↓
눈으로 확인
        ↓
변형이 자연스러운가?
        ↓
괜찮다면
        ↓
Train 전체 Augmentation
```

즉 `03_augment_preview.py`의 역할은

> “이 Augmentation 조건을 Train 전체에 적용해도 괜찮은지 먼저 시험해 보는 코드”

라고 이해하면 됩니다.

실행이 끝나면 원본과 네 가지 변형 이미지, 그리고 한 화면에서 비교할 수 있는 Grid가 `results/apple/augmentation/`에 저장됩니다. 

이제 실제 코드를 작성합니다.

**파일: `scripts/03_augment_preview.py`**

```python
from __future__ import annotations

import argparse
import sys
from pathlib import Path

import cv2
import matplotlib.pyplot as plt
import numpy as np


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day01_common import (
    dataset_paths,
    list_images,
)


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--dataset",
        default="apple",
        choices=["apple", "banana"],
    )

    parser.add_argument(
        "--rotation-angle",
        type=float,
        default=10.0,
    )

    return parser.parse_args()


def rotate_image(image, angle):
    height, width = image.shape[:2]

    center = (
        width // 2,
        height // 2,
    )

    matrix = cv2.getRotationMatrix2D(
        center,
        angle,
        1.0,
    )

    return cv2.warpAffine(
        image,
        matrix,
        (width, height),
        borderMode=cv2.BORDER_REFLECT_101,
    )


def add_noise(
    image,
    sigma=10,
    seed=42,
):
    rng = np.random.default_rng(
        seed
    )

    noise = rng.normal(
        0,
        sigma,
        image.shape,
    )

    result = (
        image.astype(np.float32)
        + noise
    )

    return np.clip(
        result,
        0,
        255,
    ).astype(np.uint8)


def main() -> None:
    args = parse_args()
    paths = dataset_paths(args.dataset)

    train_files = list_images(
        paths["split_train"]
    )

    if not train_files:
        raise RuntimeError(
            "Train 이미지가 없습니다. "
            "02_prepare_split.py를 먼저 실행하세요."
        )

    source_path = train_files[0]

    image = cv2.imread(
        str(source_path)
    )

    if image is None:
        raise RuntimeError(
            f"이미지를 읽지 못했습니다: {source_path}"
        )

    results = {
        "Original": image.copy(),
        "Brightness": cv2.convertScaleAbs(
            image,
            alpha=0.80,
            beta=0,
        ),
        "Rotation": rotate_image(
            image,
            args.rotation_angle,
        ),
        "Blur": cv2.GaussianBlur(
            image,
            (5, 5),
            0,
        ),
        "Noise": add_noise(
            image,
            sigma=10,
            seed=42,
        ),
    }

    output_dir = paths["result_aug"]

    for name, result in results.items():
        output_path = (
            output_dir
            / f"preview_{name.lower()}.jpg"
        )

        saved = cv2.imwrite(
            str(output_path),
            result,
        )

        if not saved:
            raise RuntimeError(
                f"저장 실패: {output_path}"
            )

    figure = plt.figure(
        figsize=(15, 4)
    )

    for index, (name, result) in enumerate(
        results.items(),
        start=1,
    ):
        axis = figure.add_subplot(
            1,
            len(results),
            index,
        )

        rgb = cv2.cvtColor(
            result,
            cv2.COLOR_BGR2RGB,
        )

        axis.imshow(rgb)
        axis.set_title(name)
        axis.axis("off")

    plt.tight_layout()

    angle_name = str(
        args.rotation_angle
    ).replace(".", "_")

    grid_path = (
        output_dir
        / f"augmentation_grid_rot{angle_name}.jpg"
    )

    plt.savefig(
        grid_path,
        dpi=150,
    )

    plt.close()

    print("Source:", source_path)
    print("Saved :", grid_path)


if __name__ == "__main__":
    main()
```

### 의사코드로 읽어보기

```text
Train 이미지 한 장을 선택한다
        ↓
원본을 그대로 보관한다
        ↓
밝기를 80% 수준으로 낮춘다
        ↓
10° 회전한다
        ↓
Gaussian Blur를 적용한다
        ↓
Gaussian Noise를 추가한다
        ↓
다섯 이미지를 각각 저장한다
        ↓
한 장의 비교 Grid로 저장한다
```

실행합니다.

```bash
python scripts/03_augment_preview.py \
  --dataset apple
```

결과를 확인합니다.

```text
Source: /home/youjung/ai_vision/subject11_Synthetic_Data/day01_image_aug_vae/data/apple/split/train/0_100.jpg
Saved : /home/youjung/ai_vision/subject11_Synthetic_Data/day01_image_aug_vae/results/apple/augmentation/augmentation_grid_rot10_0.jpg
```

### 실행 결과를 해석합니다

`Source`는 이번 Preview에 사용된 원본 Train 이미지입니다.

```
Source
→ data/apple/split/train/0_100.jpg
```

즉 Train 이미지 중 한 장을 가져와 Augmentation을 시험한 것입니다.

`Saved`는 원본과 네 가지 변형을 한 화면에서 비교할 수 있도록 만든 결과 Grid 이미지입니다.

```
Saved
→ augmentation_grid_rot10_0.jpg
```

`rot10_0`은 이번 Preview에서 Rotation을 `10°`로 적용했다는 의미입니다.

실행 후 다음 파일들이 만들어집니다. 

```
results/apple/augmentation/
├─ preview_original.jpg
├─ preview_brightness.jpg
├─ preview_rotation.jpg
├─ preview_blur.jpg
├─ preview_noise.jpg
└─ augmentation_grid_rot10_0.jpg
```

각 파일은 다음 결과를 보여줍니다.

```
Original
→ 원본 이미지

Brightness
→ 밝기를 변경한 이미지

Rotation
→ 10° 회전한 이미지

Blur
→ 조금 흐리게 만든 이미지

Noise
→ 픽셀에 잡음을 추가한 이미지

Grid
→ 위 결과를 한 화면에서 비교
```

### 무엇을 확인하면 되나요?

지금은 어느 방법이 더 좋은지 점수를 계산하는 단계가 아닙니다.

Grid를 직접 보면서 다음 한 가지만 판단합니다.

> 변형된 이미지가 여전히 자연스러운 Apple 이미지로 보이는가?

즉,

```
원본과 비교
        ↓
변형이 너무 강하지 않은가?
        ↓
사과의 색과 형태가 유지되는가?
        ↓
실제 촬영에서도 일어날 수 있는 변화인가?
```

를 확인합니다.

이 Preview가 괜찮다면 다음 단계에서 한 장이 아니라 Train 전체에 Augmentation을 적용합니다.

---

# 14. Augmentation 결과를 눈으로 비교합니다

앞에서 만든 `augmentation_grid_rot10_0.jpg`를 열어 원본과 변형 이미지를 비교합니다.

이번 단계에서는 복잡한 수치를 계산하지 않고, 변형 후에도 원래 사과의 특징이 유지되는지 눈으로 확인합니다.

## Brightness

```text
너무 어둡거나 밝아지지 않았는가?

사과가 조금 어두워졌는가?

여전히 같은 빨간 사과로 볼 수 있는가?
```

## Rotation

```text
회전 후에도 사과가 자연스럽게 보이는가?


10° 정도 회전했을 때
사과의 의미가 유지되는가?

객체 일부가 지나치게 잘리지는 않았는가?
```

## Blur

```text
흐려졌지만 형태를 알아볼 수 있는가?

약간 흐려졌지만
형태를 구분할 수 있는가?
```

## Noise

```text
잡음이 있어도 색과 윤곽이 유지되는가?

Noise가 생겼지만
사과의 색·윤곽이 유지되는가?
```

변형이 눈에 잘 보인다는 이유만으로 좋은 데이터가 되는 것은 아닙니다.

---

# 15. Train 전체에 적용할 Augmentation 범위를 확인합니다

Preview 결과가 괜찮다면 이제 Train 전체에 적용할 기본 범위를 정합니다.

오늘은 다음 값을 사용합니다.

```
Brightness
→ alpha 0.8 ~ 1.2

Rotation
→ -10° ~ +10°

Blur
→ kernel 3 또는 5

Noise
→ sigma 5 / 10 / 15
```

이 값들은 모든 이미지에 항상 맞는 정답은 아닙니다.

중요한 것은 **실제 촬영 환경에서 일어날 수 있는 정도의 변화인지 확인한 뒤 사용하는 것**입니다.

```
변형값 선택
        ↓
한 장으로 Preview
        ↓
결과 확인
        ↓
자연스럽다면
        ↓
Train 전체에 적용
```

> 지금까지는 한 장으로 시험한 단계이고, 
> 다음 PART에서는 이 기준을 이용하여 Train 전체 이미지를 자동으로 보강합니다.
> 
---

# PART 4. Train 전체를 자동으로 보강하고 Metadata를 기록합니다

---

# 16. Metadata가 왜 필요한지 이해합니다

합성 이미지만 저장하면 나중에 다음 질문에 답하기 어렵습니다.

```text
이 이미지는 어떤 원본에서 만들었나?

무슨 방법을 사용했나?

회전각도는 몇 도였나?

어떤 Seed를 사용했나?
```

그래서 생성 이미지와 함께 **생성 이력**을 기록합니다.

오늘 Metadata에는 다음 정보를 남깁니다.

```text
생성 파일명
원본 파일명
원본 파일 Hash
Augmentation 방법
사용한 Parameter
Seed
검수 상태
```

---

# 17. Train 전체를 자동으로 보강하는 코드를 작성합니다

**파일: `scripts/04_augment_train.py`**

```python
from __future__ import annotations

import argparse
import hashlib
import random
import shutil
import sys
from pathlib import Path

import cv2
import numpy as np
import pandas as pd


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day01_common import (
    dataset_paths,
    list_images,
)


SEED = 42


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--dataset",
        default="apple",
        choices=["apple", "banana"],
    )

    parser.add_argument(
        "--aug-per-image",
        type=int,
        default=2,
    )

    parser.add_argument(
        "--rotation-max",
        type=float,
        default=10.0,
    )

    return parser.parse_args()


def sha256_file(path: Path) -> str:
    digest = hashlib.sha256()

    with path.open("rb") as file:
        for chunk in iter(
            lambda: file.read(1024 * 1024),
            b"",
        ):
            digest.update(chunk)

    return digest.hexdigest()


def rotate_image(image, angle):
    height, width = image.shape[:2]

    center = (
        width // 2,
        height // 2,
    )

    matrix = cv2.getRotationMatrix2D(
        center,
        angle,
        1.0,
    )

    return cv2.warpAffine(
        image,
        matrix,
        (width, height),
        borderMode=cv2.BORDER_REFLECT_101,
    )


def apply_augmentation(
    image,
    method,
    py_rng,
    np_rng,
    rotation_max,
):
    if method == "brightness":
        alpha = py_rng.uniform(
            0.8,
            1.2,
        )

        result = cv2.convertScaleAbs(
            image,
            alpha=alpha,
            beta=0,
        )

        parameter = (
            f"alpha={alpha:.3f}"
        )

    elif method == "rotation":
        angle = py_rng.uniform(
            -rotation_max,
            rotation_max,
        )

        result = rotate_image(
            image,
            angle,
        )

        parameter = (
            f"angle={angle:.2f}"
        )

    elif method == "blur":
        kernel = py_rng.choice(
            [3, 5]
        )

        result = cv2.GaussianBlur(
            image,
            (kernel, kernel),
            0,
        )

        parameter = (
            f"kernel={kernel}"
        )

    elif method == "noise":
        sigma = py_rng.choice(
            [5, 10, 15]
        )

        noise = np_rng.normal(
            0,
            sigma,
            image.shape,
        )

        result = (
            image.astype(np.float32)
            + noise
        )

        result = np.clip(
            result,
            0,
            255,
        ).astype(np.uint8)

        parameter = (
            f"sigma={sigma}"
        )

    else:
        raise ValueError(
            f"Unknown method: {method}"
        )

    return result, parameter


def main() -> None:
    args = parse_args()
    paths = dataset_paths(args.dataset)

    train_files = list_images(
        paths["split_train"]
    )

    if not train_files:
        raise RuntimeError(
            "Train 이미지가 없습니다."
        )

    output_dir = (
        paths["augmented_train"]
    )

    if output_dir.exists():
        shutil.rmtree(output_dir)

    output_dir.mkdir(
        parents=True,
        exist_ok=True,
    )

    methods = [
        "brightness",
        "rotation",
        "blur",
        "noise",
    ]

    rows = []

    for source_index, source_path in enumerate(
        train_files
    ):
        image = cv2.imread(
            str(source_path)
        )

        if image is None:
            print(
                "[SKIP]",
                source_path.name,
            )
            continue

        source_hash = sha256_file(
            source_path
        )

        for index in range(
            1,
            args.aug_per_image + 1,
        ):
            sample_seed = (
                SEED
                + source_index
                * args.aug_per_image
                + (index - 1)
            )

            py_rng = random.Random(
                sample_seed
            )

            np_rng = np.random.default_rng(
                sample_seed
            )

            method = py_rng.choice(
                methods
            )

            result, parameter = (
                apply_augmentation(
                    image,
                    method,
                    py_rng,
                    np_rng,
                    args.rotation_max,
                )
            )

            output_name = (
                f"{source_path.stem}"
                f"__{method}"
                f"__{index:02d}.jpg"
            )

            output_path = (
                output_dir
                / output_name
            )

            saved = cv2.imwrite(
                str(output_path),
                result,
            )

            if not saved:
                raise RuntimeError(
                    f"저장 실패: {output_path}"
                )

            rows.append(
                {
                    "file": output_name,
                    "method": method,
                    "source": source_path.name,
                    "source_hash": source_hash,
                    "parameter": parameter,
                    "seed": sample_seed,
                    "approved": "pending",
                }
            )

    metadata_path = (
        paths["report"]
        / "synthetic_metadata.csv"
    )

    pd.DataFrame(
        rows
    ).to_csv(
        metadata_path,
        index=False,
        encoding="utf-8-sig",
    )

    print("=" * 60)
    print("Augmentation Completed")
    print("=" * 60)

    print(
        "Original Train:",
        len(train_files),
    )

    print(
        "Generated     :",
        len(rows),
    )

    print(
        "Rotation Max :",
        args.rotation_max,
    )

    print(
        "Metadata     :",
        metadata_path,
    )


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

Train 데이터 전체에 자동으로 보강을 적용하면서 **생성 이력까지 함께 남기기 위해** 작성합니다.

또한 같은 스크립트를 Mini Challenge에서도 사용할 수 있도록 `--dataset`과 `--rotation-max`를 인자로 받습니다.

이번 코드에서는 Base Seed를 42로 두고 각 생성 이미지마다 별도의 `sample_seed`를 계산합니다. 따라서 Metadata의 한 행만 확인해도 해당 생성 이미지에 사용된 난수 기준을 다시 추적하기 쉽습니다.

### 의사코드로 읽어보기

```text
Train 이미지 목록을 가져온다
        ↓
기존 augmented 폴더를 비우고 다시 만든다
        ↓
Base Seed를 42로 정한다
        ↓
각 원본 이미지마다
두 개의 보강 이미지를 만든다
        ↓
생성 이미지마다 고유한 sample_seed를 계산한다
        ↓
그 Seed로 Augmentation 방법과 Parameter를 결정한다
        ↓
Brightness / Rotation / Blur / Noise 중
한 방법을 적용한다
        ↓
결과 이미지를 저장한다
        ↓
원본 Hash와 생성 방법·Parameter·sample_seed를 기록한다
        ↓
모든 기록을 Metadata CSV로 저장한다
```

실행합니다.

```bash
python scripts/04_augment_train.py \
  --dataset apple
```

Train이 약 419장이고 원본당 2장씩 만들었다면 약 838장의 보강 이미지가 생성됩니다.

실제 수량은 실행 결과를 확인합니다.

---

# 18. 생성 이미지와 Metadata를 확인합니다

생성 파일 일부를 확인합니다.

```bash
ls data/apple/augmented/train | head
```

예:

```text
...__brightness__01.jpg
...__rotation__02.jpg
...__noise__01.jpg
```

Metadata를 확인합니다.

```bash
head reports/apple/synthetic_metadata.csv
```

주요 열은 다음 의미입니다.

| 열 | 의미 |
|---|---|
| `file` | 생성된 이미지 |
| `method` | 사용한 보강 방법 |
| `source` | 원본 파일 |
| `source_hash` | 원본 파일의 SHA-256 |
| `parameter` | 실제 적용한 값 |
| `seed` | 해당 생성 이미지에 사용한 난수 재현 기준 |
| `approved` | 검수 상태 |

현재 `approved`는 다음 상태입니다.

```text
pending
```

아직 자동·사람 검수가 끝나지 않았기 때문입니다.

오늘은 `approved` 값을 코드가 자동으로 `yes`로 바꾸지 않습니다. 사람의 판단이 필요한 항목을 프로그램이 임의로 승인하면 안 되기 때문입니다.

1일차에서는 Human QA 결과를 `day01_notes.md`와 Mini Challenge의 `challenge_notes.md`에 기록합니다. **검수된 Augmented Image를 실제 학습 데이터에 합치는 승인 Workflow는 교과 11 후반의 성능 비교 단계에서 다시 다룹니다.**

---

# PART 5. VAE로 Apple 이미지의 특징을 압축하고 복원합니다

---

## Augmentation과 VAE는 여기서부터 서로 다른 실험으로 봅니다

앞에서 만든 Augmented Image는 `data/apple/augmented/train/`에 그대로 보관합니다.

VAE는 다음 데이터만 사용합니다.

```text
data/apple/split/train
→ VAE 학습

data/apple/split/val
→ Epoch별 Validation Loss 확인

data/apple/split/test
→ 마지막 Reconstruction 확인
```

즉 오늘은 다음 두 실험을 섞지 않습니다.

```text
실험 A
원본 Train
→ Brightness / Rotation / Blur / Noise
→ Augmented Image
→ Metadata + QA

실험 B
원본 Train
→ VAE 학습
→ Test Reconstruction
→ Random Latent Sample
```

이렇게 분리하면 **기존 이미지를 직접 바꾸는 Augmentation**과 **데이터의 특징 분포를 학습하는 VAE**의 역할을 구분하기 쉽습니다.

---

# 19. VAE의 전체 구조를 이해합니다

VAE는 이미지를 다음 순서로 처리합니다.

```text
Input Image
        ↓
Encoder
        ↓
mu + logvar
        ↓
Latent z
        ↓
Decoder
        ↓
Reconstruction
```

쉽게 생각하면 다음과 같습니다.

```text
원본 이미지
        ↓
중요한 특징을 작은 숫자 공간에 표현
        ↓
그 숫자 표현을 이용해
다시 이미지 복원
```

오늘의 목적은 **사진처럼 완벽한 이미지를 생성하는 것**이 아닙니다.

다음 질문에 답할 수 있으면 됩니다.

```text
어떤 특징이 유지되었는가?

어떤 세부정보가 사라졌는가?

Latent Space라는 작은 표현으로
이미지를 다시 만들 수 있는가?
```

---

# 20. Encoder·Latent Space·Decoder를 조금 더 자세히 봅니다

## Encoder

```text
64 × 64 RGB Image
        ↓
Convolution
        ↓
더 작은 Feature
        ↓
mu / logvar
```

Encoder는 이미지를 그대로 저장하는 것이 아니라 특징을 압축해서 표현합니다.

## Latent Space

Latent Space는 이미지의 특징을 작은 벡터로 표현하는 공간입니다.

오늘 기본값은 다음과 같습니다.

```text
latent_dim = 32
```

즉 한 이미지를 그대로 저장하는 대신 **32개의 잠재값으로 표현하는 구조**를 실험합니다.

## Decoder

```text
Latent z
        ↓
Decoder
        ↓
64 × 64 RGB Image
```

Decoder는 Latent 표현에서 다시 이미지를 만듭니다.

## 왜 `mu`와 `logvar`가 필요한가요?

VAE는 Latent 값을 한 점으로만 외우지 않고 **분포 형태로 학습**하려고 합니다.

오늘은 수학식을 외우기보다 다음 흐름을 이해합니다.

```text
Encoder
        ↓
mu
logvar
        ↓
분포에서 z를 Sampling
        ↓
Decoder
```

---

# 21. VAE를 학습하는 코드를 작성합니다

오늘의 VAE는 다음 원칙을 사용합니다.

```text
Train
→ 모델 학습

Validation
→ Epoch마다 Loss 확인

Test
→ 학습이 끝난 뒤
  Original vs Reconstruction 확인
```

**파일: `scripts/05_train_vae.py`**

```python
from __future__ import annotations

import argparse
import random
import sys
from pathlib import Path

import numpy as np
import pandas as pd
import torch
import torch.nn as nn
import torch.nn.functional as F

from PIL import Image

from torch.utils.data import (
    DataLoader,
    Dataset,
)

from torchvision import transforms
from torchvision.utils import save_image


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day01_common import (
    dataset_paths,
    list_images,
)


IMAGE_SIZE = 64
BATCH_SIZE = 32
SEED = 42
BETA = 0.001


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--dataset",
        default="apple",
        choices=["apple", "banana"],
    )

    parser.add_argument(
        "--epochs",
        type=int,
        default=10,
    )

    parser.add_argument(
        "--latent",
        type=int,
        default=32,
    )

    parser.add_argument(
        "--eval-split",
        choices=["val", "test"],
        default="test",
        help=(
            "Reconstruction을 만들 Split. "
            "조건 비교는 val, 최종 확인은 test를 사용합니다."
        ),
    )

    parser.add_argument(
        "--eval-only",
        action="store_true",
        help=(
            "이미 학습된 동일 조건의 Model을 불러와 "
            "선택한 Split의 Reconstruction만 생성합니다."
        ),
    )

    return parser.parse_args()


def set_seed(seed: int) -> None:
    random.seed(seed)
    np.random.seed(seed)
    torch.manual_seed(seed)

    if torch.cuda.is_available():
        torch.cuda.manual_seed_all(seed)


class FlatImageDataset(Dataset):
    def __init__(
        self,
        root_dir,
        image_size=64,
    ):
        self.files = list_images(
            Path(root_dir)
        )

        if not self.files:
            raise RuntimeError(
                f"No images found: {root_dir}"
            )

        self.transform = transforms.Compose(
            [
                transforms.Resize(
                    (image_size, image_size)
                ),
                transforms.ToTensor(),
            ]
        )

    def __len__(self):
        return len(self.files)

    def __getitem__(self, index):
        path = self.files[index]

        image = Image.open(
            path
        ).convert("RGB")

        return self.transform(
            image
        )


class VAE(nn.Module):
    def __init__(
        self,
        latent_dim=32,
    ):
        super().__init__()

        self.encoder = nn.Sequential(
            nn.Conv2d(
                3,
                32,
                4,
                2,
                1,
            ),
            nn.ReLU(),
            nn.Conv2d(
                32,
                64,
                4,
                2,
                1,
            ),
            nn.ReLU(),
            nn.Conv2d(
                64,
                128,
                4,
                2,
                1,
            ),
            nn.ReLU(),
        )

        self.fc_mu = nn.Linear(
            128 * 8 * 8,
            latent_dim,
        )

        self.fc_logvar = nn.Linear(
            128 * 8 * 8,
            latent_dim,
        )

        self.fc_decode = nn.Linear(
            latent_dim,
            128 * 8 * 8,
        )

        self.decoder = nn.Sequential(
            nn.ConvTranspose2d(
                128,
                64,
                4,
                2,
                1,
            ),
            nn.ReLU(),
            nn.ConvTranspose2d(
                64,
                32,
                4,
                2,
                1,
            ),
            nn.ReLU(),
            nn.ConvTranspose2d(
                32,
                3,
                4,
                2,
                1,
            ),
            nn.Sigmoid(),
        )

    def encode(self, x):
        feature = self.encoder(x)

        feature = feature.view(
            feature.size(0),
            -1,
        )

        mu = self.fc_mu(
            feature
        )

        logvar = self.fc_logvar(
            feature
        )

        return mu, logvar

    def reparameterize(
        self,
        mu,
        logvar,
    ):
        std = torch.exp(
            0.5 * logvar
        )

        eps = torch.randn_like(
            std
        )

        return (
            mu
            + eps * std
        )

    def decode(self, z):
        feature = self.fc_decode(
            z
        )

        feature = feature.view(
            -1,
            128,
            8,
            8,
        )

        return self.decoder(
            feature
        )

    def forward(self, x):
        mu, logvar = self.encode(
            x
        )

        z = self.reparameterize(
            mu,
            logvar,
        )

        reconstruction = self.decode(
            z
        )

        return (
            reconstruction,
            mu,
            logvar,
        )


def vae_loss(
    reconstruction,
    original,
    mu,
    logvar,
):
    reconstruction_loss = F.mse_loss(
        reconstruction,
        original,
        reduction="mean",
    )

    kl_loss = -0.5 * torch.mean(
        1
        + logvar
        - mu.pow(2)
        - logvar.exp()
    )

    total_loss = (
        reconstruction_loss
        + BETA * kl_loss
    )

    return (
        total_loss,
        reconstruction_loss,
        kl_loss,
    )


def evaluate(
    model,
    loader,
    device,
):
    model.eval()

    total = 0.0
    reconstruction_total = 0.0
    kl_total = 0.0
    count = 0

    with torch.no_grad():
        for images in loader:
            images = images.to(
                device
            )

            reconstruction, mu, logvar = (
                model(images)
            )

            loss, recon_loss, kl_loss = (
                vae_loss(
                    reconstruction,
                    images,
                    mu,
                    logvar,
                )
            )

            batch_size = images.size(0)

            total += (
                loss.item()
                * batch_size
            )

            reconstruction_total += (
                recon_loss.item()
                * batch_size
            )

            kl_total += (
                kl_loss.item()
                * batch_size
            )

            count += batch_size

    if count == 0:
        raise RuntimeError(
            "Validation 데이터가 없습니다."
        )

    return (
        total / count,
        reconstruction_total / count,
        kl_total / count,
    )


def main() -> None:
    args = parse_args()

    if args.epochs < 1:
        raise ValueError(
            "--epochs는 1 이상이어야 합니다."
        )

    if args.latent < 1:
        raise ValueError(
            "--latent는 1 이상이어야 합니다."
        )

    set_seed(SEED)

    paths = dataset_paths(
        args.dataset
    )

    train_dataset = FlatImageDataset(
        paths["split_train"],
        image_size=IMAGE_SIZE,
    )

    val_dataset = FlatImageDataset(
        paths["split_val"],
        image_size=IMAGE_SIZE,
    )

    test_dataset = FlatImageDataset(
        paths["split_test"],
        image_size=IMAGE_SIZE,
    )

    generator = torch.Generator()
    generator.manual_seed(SEED)

    train_loader = DataLoader(
        train_dataset,
        batch_size=BATCH_SIZE,
        shuffle=True,
        generator=generator,
    )

    val_loader = DataLoader(
        val_dataset,
        batch_size=BATCH_SIZE,
        shuffle=False,
    )

    test_loader = DataLoader(
        test_dataset,
        batch_size=BATCH_SIZE,
        shuffle=False,
    )

    eval_loader = (
        val_loader
        if args.eval_split == "val"
        else test_loader
    )

    device = torch.device(
        "cuda"
        if torch.cuda.is_available()
        else "cpu"
    )

    print("Device     :", device)
    print("Train      :", len(train_dataset))
    print("Val        :", len(val_dataset))
    print("Test       :", len(test_dataset))
    print("Eval Split :", args.eval_split)
    print("Eval Only  :", args.eval_only)

    model = VAE(
        latent_dim=args.latent
    ).to(device)

    model_path = (
        paths["model"]
        / (
            f"vae_latent"
            f"{args.latent}"
            f"_e{args.epochs}.pt"
        )
    )

    loss_path = (
        paths["report"]
        / (
            f"vae_loss_latent"
            f"{args.latent}"
            f"_e{args.epochs}.csv"
        )
    )

    if args.eval_only:
        if not model_path.exists():
            raise FileNotFoundError(
                "먼저 같은 Epoch / Latent 조건으로 "
                f"학습해야 합니다: {model_path}"
            )

        state_dict = torch.load(
            model_path,
            map_location=device,
        )

        model.load_state_dict(
            state_dict
        )

        print(
            "Loaded Model:",
            model_path,
        )

    else:
        optimizer = torch.optim.Adam(
            model.parameters(),
            lr=1e-3,
        )

        history = []

        for epoch in range(
            1,
            args.epochs + 1,
        ):
            model.train()

            train_total = 0.0
            train_reconstruction = 0.0
            train_kl = 0.0
            train_count = 0

            for images in train_loader:
                images = images.to(
                    device
                )

                optimizer.zero_grad()

                reconstruction, mu, logvar = (
                    model(images)
                )

                loss, recon_loss, kl_loss = (
                    vae_loss(
                        reconstruction,
                        images,
                        mu,
                        logvar,
                    )
                )

                loss.backward()
                optimizer.step()

                batch_size = images.size(0)

                train_total += (
                    loss.item()
                    * batch_size
                )

                train_reconstruction += (
                    recon_loss.item()
                    * batch_size
                )

                train_kl += (
                    kl_loss.item()
                    * batch_size
                )

                train_count += batch_size

            train_loss = (
                train_total
                / train_count
            )

            train_recon = (
                train_reconstruction
                / train_count
            )

            train_kl_loss = (
                train_kl
                / train_count
            )

            (
                val_loss,
                val_recon,
                val_kl,
            ) = evaluate(
                model,
                val_loader,
                device,
            )

            history.append(
                {
                    "epoch": epoch,
                    "train_loss": train_loss,
                    "train_recon": train_recon,
                    "train_kl": train_kl_loss,
                    "val_loss": val_loss,
                    "val_recon": val_recon,
                    "val_kl": val_kl,
                }
            )

            print(
                f"Epoch "
                f"{epoch:02d}/"
                f"{args.epochs} "
                f"Train={train_loss:.6f} "
                f"Val={val_loss:.6f}"
            )

        torch.save(
            model.state_dict(),
            model_path,
        )

        pd.DataFrame(
            history
        ).to_csv(
            loss_path,
            index=False,
            encoding="utf-8-sig",
        )

    model.eval()

    with torch.no_grad():
        sample = next(
            iter(eval_loader)
        ).to(device)

        mu, _ = model.encode(
            sample
        )

        reconstruction = model.decode(
            mu
        )

        count = min(
            8,
            sample.size(0),
        )

        comparison = torch.cat(
            [
                sample[:count],
                reconstruction[:count],
            ],
            dim=0,
        )

        reconstruction_path = (
            paths["result_vae"]
            / (
                f"reconstruction_"
                f"{args.eval_split}_"
                f"latent{args.latent}"
                f"_e{args.epochs}.png"
            )
        )

        save_image(
            comparison,
            reconstruction_path,
            nrow=count,
        )

        generated_path = None

        if not args.eval_only:
            random_z = torch.randn(
                16,
                args.latent,
                device=device,
            )

            generated = model.decode(
                random_z
            )

            generated_path = (
                paths["result_vae"]
                / (
                    f"generated_samples_"
                    f"latent{args.latent}"
                    f"_e{args.epochs}.png"
                )
            )

            save_image(
                generated,
                generated_path,
                nrow=4,
            )

    print()
    print("Model          :", model_path)

    if loss_path.exists():
        print("Loss           :", loss_path)

    print(
        "Reconstruction :",
        reconstruction_path,
    )

    if generated_path is not None:
        print(
            "Random Samples :",
            generated_path,
        )

if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

Apple의 **원본 Train 이미지**로 VAE를 학습하고 Validation Loss로 학습 상태를 확인합니다. Reconstruction은 `--eval-split`으로 Validation 또는 Test를 명시하여 만들 수 있고, `--eval-only`를 사용하면 이미 학습된 같은 조건의 모델을 다시 학습하지 않고 최종 Split만 확인할 수 있습니다.

오늘 앞에서 만든 Augmented Image는 이 VAE 학습에 넣지 않습니다. 1일차에서는 Augmentation과 VAE를 각각 독립된 실험으로 실행하여 두 방법의 차이를 먼저 이해합니다.

### 의사코드로 읽어보기

```text
Epoch와 Latent 값이 올바른지 확인한다
        ↓
Train / Val / Test 이미지를
64 × 64 Tensor로 준비한다
        ↓
Train DataLoader의 난수 Seed를 고정한다
        ↓
Encoder로 특징을 압축한다
        ↓
mu / logvar를 만든다
        ↓
Train 단계에서는
Latent z를 Sampling한다
        ↓
Decoder로 이미지를 복원한다
        ↓
원본과 복원 이미지 차이를 계산한다
        +
Latent 분포가 지나치게 흐트러지지 않도록
KL Loss를 계산한다
        ↓
두 Loss를 합쳐 역전파한다
        ↓
Train 데이터로 파라미터를 업데이트한다
        ↓
Validation 데이터로 Loss를 확인한다
        ↓
모든 Epoch가 끝나면 모델과 Loss를 저장한다
        ↓
eval-split으로
Validation 또는 Test를 선택한다
        ↓
선택한 Split의 mu를 Decoder에 넣어
Reconstruction을 저장한다
        ↓
학습 실행이면
표준정규분포에서 Random z를 만든다
        ↓
Decoder에 넣어
VAE Random Sample 16장을 저장한다

eval-only이면
이미 저장된 Model을 불러와
선택한 Split Reconstruction만 만든다
```

---

# 22. VAE를 기본 조건으로 실행합니다

기본 조건입니다.

```text
Epoch = 10
Latent Dimension = 32
```

실행합니다.

```bash
python scripts/05_train_vae.py \
  --dataset apple \
  --epochs 10 \
  --latent 32 \
  --eval-split test
```

학습 중 다음과 비슷하게 출력됩니다.

```text
Device : cuda
Train  : ...
Val    : ...
Test   : ...

Epoch 01/10 Train=... Val=...
Epoch 02/10 Train=... Val=...
...
Epoch 10/10 Train=... Val=...
```

완료되면 다음 결과가 생깁니다.

```text
models/apple/
└─ vae_latent32_e10.pt

reports/apple/
└─ vae_loss_latent32_e10.csv

results/apple/vae/
├─ reconstruction_test_latent32_e10.png
└─ generated_samples_latent32_e10.png
```

---

# 23. VAE Loss를 읽고 해석합니다

Loss CSV를 확인합니다.

```bash
cat reports/apple/vae_loss_latent32_e10.csv
```

오늘은 다음 두 값을 중심으로 봅니다.

```text
train_loss
→ 실제 학습에 사용한 Train의 Loss

val_loss
→ 학습에 사용하지 않은 Validation의 Loss
```

일반적으로 학습이 진행되면서 두 값이 어느 정도 낮아지는지 확인합니다.

하지만 다음처럼 단정하지 않습니다.

```text
Loss가 무조건 작다
=
좋은 생성모델이다
```

반드시 복원 이미지도 함께 봅니다.

---

# 24. Original과 Reconstruction을 비교합니다

`reconstruction_test_latent32_e10.png`는 다음 순서로 저장됩니다.

```text
위쪽 8장
→ Test Original

아래쪽 8장
→ VAE Reconstruction
```

다음 질문에 답합니다.

```text
빨간색 특징은 유지되는가?

둥근 사과 형태는 유지되는가?

세부 윤곽은 얼마나 흐려졌는가?

배경은 어떻게 복원되는가?
```

VAE Reconstruction이 원본보다 흐릿해 보일 수 있습니다.

이것은 오늘 사용하는 간단한 VAE 구조와 Loss의 특성상 자연스럽게 나타날 수 있습니다.

오늘의 핵심은 **사진 품질 경쟁**이 아니라 다음 흐름을 이해하는 것입니다.

```text
이미지
→ 특징 압축
→ Latent
→ 복원
```

## Random Sample도 함께 확인합니다

`generated_samples_latent32_e10.png`는 기존 Test 이미지를 복원한 결과가 아닙니다.

```text
표준정규분포
        ↓
Random z 16개
        ↓
학습된 Decoder
        ↓
새로운 이미지 16장
```

이 결과는 10 Epoch의 간단한 VAE로 만든 교육용 샘플이므로 흐릿하거나 완성도가 낮을 수 있습니다.

여기서는 다음 차이만 분명하게 구분합니다.

```text
Reconstruction
→ 실제 Test 이미지를 Encoder에 넣고 다시 복원

Random Sample
→ 실제 이미지를 넣지 않고
  Latent z에서 Decoder로 생성
```

따라서 1일차의 VAE 실습은 **복원 원리와 생성 원리를 모두 가볍게 확인하는 단계**입니다.

---

# 25. Augmentation과 VAE의 차이를 다시 정리합니다

둘은 같은 일이 아닙니다.

```text
Augmentation

원본 이미지
        ↓
밝기·회전·Blur·Noise
        ↓
변형 이미지
```

```text
VAE

원본 Train 이미지
        ↓
Encoder 학습
        ↓
Latent Space
        ↓
Decoder
        ├─ Test Reconstruction
        └─ Random Latent Sample 생성
```

1일차에서는 VAE를 통해 **생성모델의 압축·복원 원리와 Latent Sampling의 기본 생성 원리**를 익힙니다.

2일차에는 여기서 확장하여 GAN·WGAN-GP·Diffusion으로 새로운 이미지를 만드는 방법을 비교합니다.

---

# PART 6. 생성한 데이터를 QA하고 Leakage를 확인합니다

---

# 26. Automatic QA를 수행하는 코드를 작성합니다

Automatic QA에서는 다음 항목을 확인합니다.

```text
Metadata에 기록된 파일이 실제로 존재하는가?

OpenCV로 정상적으로 읽히는가?

Width·Height가 0보다 큰가?

Metadata의 원본 Hash가
실제 Train 원본과 연결되는가?
```

그리고 사람이 빠르게 확인할 수 있도록 보강 이미지 16장을 한 Grid로 만듭니다.

**파일: `scripts/06_qa_images.py`**

```python
from __future__ import annotations

import argparse
import hashlib
import sys
from pathlib import Path

import cv2
import numpy as np
import pandas as pd


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day01_common import (
    dataset_paths,
    list_images,
)


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--dataset",
        default="apple",
        choices=["apple", "banana"],
    )

    return parser.parse_args()


def sha256_file(path: Path) -> str:
    digest = hashlib.sha256()

    with path.open("rb") as file:
        for chunk in iter(
            lambda: file.read(1024 * 1024),
            b"",
        ):
            digest.update(chunk)

    return digest.hexdigest()


def make_grid(
    image_paths,
    output_path,
    tile_size=128,
    columns=4,
):
    if not image_paths:
        return

    rows = []

    for start in range(
        0,
        len(image_paths),
        columns,
    ):
        batch = image_paths[
            start:start + columns
        ]

        tiles = []

        for path in batch:
            image = cv2.imread(
                str(path)
            )

            if image is None:
                continue

            tile = cv2.resize(
                image,
                (tile_size, tile_size),
            )

            cv2.putText(
                tile,
                path.stem.split("__")[-2],
                (5, 15),
                cv2.FONT_HERSHEY_SIMPLEX,
                0.35,
                (0, 0, 0),
                1,
                cv2.LINE_AA,
            )

            tiles.append(tile)

        while len(tiles) < columns:
            tiles.append(
                np.zeros(
                    (
                        tile_size,
                        tile_size,
                        3,
                    ),
                    dtype=np.uint8,
                )
            )

        rows.append(
            np.hstack(tiles)
        )

    if rows:
        grid = np.vstack(rows)

        saved = cv2.imwrite(
            str(output_path),
            grid,
        )

        if not saved:
            raise RuntimeError(
                f"QA Grid 저장 실패: {output_path}"
            )


def choose_human_qa_samples(
    metadata: pd.DataFrame,
    augmented_dir: Path,
) -> list[Path]:
    selected = []

    for method in [
        "brightness",
        "rotation",
        "blur",
        "noise",
    ]:
        group = metadata[
            metadata["method"] == method
        ]

        if group.empty:
            continue

        group = group.sample(
            n=min(
                4,
                len(group),
            ),
            random_state=42,
        )

        for filename in group["file"]:
            path = (
                augmented_dir
                / filename
            )

            if path.exists():
                selected.append(path)

    return selected


def main() -> None:
    args = parse_args()
    paths = dataset_paths(args.dataset)

    metadata_path = (
        paths["report"]
        / "synthetic_metadata.csv"
    )

    if not metadata_path.exists():
        raise FileNotFoundError(
            metadata_path
        )

    metadata = pd.read_csv(
        metadata_path
    )

    required_columns = {
        "file",
        "method",
        "source_hash",
    }

    missing_columns = (
        required_columns
        - set(metadata.columns)
    )

    if missing_columns:
        raise RuntimeError(
            "Metadata 열이 부족합니다: "
            + ", ".join(
                sorted(missing_columns)
            )
        )

    train_files = list_images(
        paths["split_train"]
    )

    train_hashes = {
        sha256_file(path)
        for path in train_files
    }

    rows = []

    for record in metadata.to_dict(
        orient="records"
    ):
        image_path = (
            paths["augmented_train"]
            / record["file"]
        )

        exists = image_path.exists()
        readable = False
        width = 0
        height = 0

        if exists:
            image = cv2.imread(
                str(image_path)
            )

            if image is not None:
                readable = True
                height, width = (
                    image.shape[:2]
                )

        source_hash_ok = (
            record["source_hash"]
            in train_hashes
        )

        auto_pass = (
            exists
            and readable
            and width > 0
            and height > 0
            and source_hash_ok
        )

        rows.append(
            {
                "file": record["file"],
                "method": record["method"],
                "exists": exists,
                "readable": readable,
                "width": width,
                "height": height,
                "source_hash_ok": source_hash_ok,
                "auto_pass": auto_pass,
            }
        )

    qa_path = (
        paths["report"]
        / "qa_results.csv"
    )

    pd.DataFrame(
        rows
    ).to_csv(
        qa_path,
        index=False,
        encoding="utf-8-sig",
    )

    grid_path = (
        paths["result_aug"]
        / "human_qa_grid.jpg"
    )

    human_qa_samples = (
        choose_human_qa_samples(
            metadata,
            paths["augmented_train"],
        )
    )

    make_grid(
        human_qa_samples,
        grid_path,
    )

    passed = sum(
        row["auto_pass"]
        for row in rows
    )

    print("QA total :", len(rows))
    print("QA pass  :", passed)
    print("QA fail  :", len(rows) - passed)
    print(
        "Human QA samples:",
        len(human_qa_samples),
    )
    print("QA CSV   :", qa_path)
    print("QA Grid  :", grid_path)


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

수백 장의 이미지를 사람이 하나씩 열어보기 전에 **파일 오류와 원본 연결 오류를 자동으로 먼저 걸러내기 위해** 작성합니다.

### 의사코드로 읽어보기

```text
Synthetic Metadata를 읽는다
        ↓
필수 열이 있는지 확인한다
        ↓
Train 원본의 Hash 목록을 만든다
        ↓
각 생성 이미지가 존재하는지 확인한다
        ↓
OpenCV로 읽히는지 확인한다
        ↓
Width·Height가 정상인지 확인한다
        ↓
Metadata의 원본 Hash가
Train에 실제로 있는지 확인한다
        ↓
조건을 모두 통과하면 auto_pass
        ↓
결과를 qa_results.csv로 저장한다
        ↓
Brightness / Rotation / Blur / Noise에서
Seed 42로 각각 최대 4장씩 Sampling한다
        ↓
네 방법을 고르게 볼 수 있는
Human QA Grid를 만든다
```

실행합니다.

```bash
python scripts/06_qa_images.py \
  --dataset apple
```

---

# 27. Human QA를 수행합니다

Automatic QA를 통과했다고 해서 이미지가 반드시 좋은 학습데이터라는 뜻은 아닙니다.

`human_qa_grid.jpg`를 눈으로 확인합니다.

이 Grid에는 Brightness·Rotation·Blur·Noise에서 각각 최대 4장씩, 총 16장의 대표 샘플이 들어 있습니다.

따라서 오늘의 Human QA는 **전체 생성 이미지를 한 장씩 승인하는 절차가 아니라, 네 가지 보강 방법의 대표 결과를 빠르게 확인하는 Sampling Review**입니다.

다음 항목을 봅니다.

- [ ] 사과의 기본 형태가 유지되어 있다.
- [ ] 지나치게 어둡거나 밝아 의미를 잃지 않았다.
- [ ] Rotation으로 객체가 부자연스럽게 잘리지 않았다.
- [ ] Blur가 지나쳐 객체 형태가 사라지지 않았다.
- [ ] Noise가 너무 강해 사과의 주요 특징이 사라지지 않았다.
- [ ] 실제 촬영에서 발생할 수 있는 변화라고 설명할 수 있다.

Human QA에서 중요한 질문은 다음입니다.

> **“보기 좋은 이미지인가?”가 아니라 “오늘의 학습 목적에 맞는 현실적인 데이터인가?”**

---

# 28. Train·Validation·Test Leakage를 확인하는 코드를 작성합니다

이번에는 파일명만 비교하지 않고 **파일 내용의 SHA-256 Hash**를 사용합니다.

또한 Augmented Image의 원본 Hash가 Train에만 존재하는지 확인합니다.

**파일: `scripts/07_check_leakage.py`**

```python
from __future__ import annotations

import argparse
import hashlib
import sys
from pathlib import Path

import pandas as pd


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day01_common import (
    dataset_paths,
    list_images,
)


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--dataset",
        default="apple",
        choices=["apple", "banana"],
    )

    return parser.parse_args()


def sha256_file(path: Path) -> str:
    digest = hashlib.sha256()

    with path.open("rb") as file:
        for chunk in iter(
            lambda: file.read(1024 * 1024),
            b"",
        ):
            digest.update(chunk)

    return digest.hexdigest()


def hash_set(directory: Path) -> set[str]:
    return {
        sha256_file(path)
        for path in list_images(directory)
    }


def main() -> None:
    args = parse_args()
    paths = dataset_paths(args.dataset)

    train_hashes = hash_set(
        paths["split_train"]
    )

    val_hashes = hash_set(
        paths["split_val"]
    )

    test_hashes = hash_set(
        paths["split_test"]
    )

    train_val = (
        train_hashes
        & val_hashes
    )

    train_test = (
        train_hashes
        & test_hashes
    )

    val_test = (
        val_hashes
        & test_hashes
    )

    metadata_path = (
        paths["report"]
        / "synthetic_metadata.csv"
    )

    augmented_source_errors = 0

    if metadata_path.exists():
        metadata = pd.read_csv(
            metadata_path
        )

        if "source_hash" not in metadata.columns:
            raise RuntimeError(
                "Metadata에 source_hash 열이 없습니다."
            )

        for source_hash in metadata[
            "source_hash"
        ]:
            source_is_invalid = (
                source_hash not in train_hashes
                or source_hash in val_hashes
                or source_hash in test_hashes
            )

            if source_is_invalid:
                augmented_source_errors += 1

    leak_count = (
        len(train_val)
        + len(train_test)
        + len(val_test)
        + augmented_source_errors
    )

    report_path = (
        paths["report"]
        / "leakage_check.txt"
    )

    lines = [
        f"dataset={args.dataset}",
        f"train_val_duplicate={len(train_val)}",
        f"train_test_duplicate={len(train_test)}",
        f"val_test_duplicate={len(val_test)}",
        (
            "augmented_source_error="
            f"{augmented_source_errors}"
        ),
        f"leak_count={leak_count}",
    ]

    report_path.write_text(
        "\n".join(lines),
        encoding="utf-8",
    )

    for line in lines:
        print(line)

    if leak_count > 0:
        raise RuntimeError(
            "Data Leakage 후보가 있습니다."
        )

    print("Leakage check: PASS")


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

평가 데이터와 학습 데이터가 정확히 분리되어 있는지 **파일 내용 수준에서 확인**하기 위해 작성합니다.

Augmented Image가 반드시 Train 원본에서만 만들어졌는지도 함께 검사합니다.

### 의사코드로 읽어보기

```text
Train 이미지 Hash를 만든다
        ↓
Validation 이미지 Hash를 만든다
        ↓
Test 이미지 Hash를 만든다
        ↓
서로 같은 Hash가 있는지 비교한다
        ↓
Synthetic Metadata를 읽는다
        ↓
각 Augmented Image의 원본 Hash가
Train에만 있는지 확인한다
        ↓
중복·오류가 하나도 없으면 PASS
        ↓
결과를 leakage_check.txt에 저장한다
```

실행합니다.

```bash
python scripts/07_check_leakage.py \
  --dataset apple
```

정상이라면 마지막에 다음이 표시됩니다.

```text
Leakage check: PASS
```

`PASS`가 의미하는 범위도 정확히 이해합니다.

```text
오늘 코드가 확인하는 것
→ 파일 내용이 완전히 같은 Exact Duplicate
→ Augmented Image가 Train 원본에서만 만들어졌는지

오늘 코드가 자동으로 확인하지 못하는 것
→ 서로 다른 파일이지만 매우 비슷한 Near-Duplicate
→ 같은 촬영 세션·같은 원본 개체에서 연속으로 얻은 유사 이미지
```

따라서 실제 프로젝트에서는 파일 Hash 확인만으로 Leakage 검사가 끝나지 않습니다.

사람·제품·촬영 세션·생산 Lot처럼 서로 연관된 데이터가 있다면 **관련된 데이터 묶음이 Train과 Test에 갈라지지 않도록 Group 기준으로 Split**해야 합니다.

1일차에서는 먼저 Exact Duplicate와 Augmentation 원본 Leakage를 코드로 확인하는 기본기를 익힙니다.

---

# 29. Apple 본 실습 결과를 한 번에 연결합니다

지금까지 한 일을 다시 연결합니다.

```text
Apple Red 1
        ↓
원래 Training / Test 확인
        ↓
Training
→ Train + Validation
        ↓
기존 Test 유지
        ↓
Train 한 장 Preview
        ↓
Brightness
Rotation
Blur
Noise
        ↓
Train 전체 Augmentation
        ↓
Metadata
        ↓
VAE
원본 Train 학습
Validation 확인
Test Reconstruction
Random Latent Sample
        ↓
Automatic QA
        ↓
Human QA
        ↓
Leakage Check
```

여기까지가 1일차 본 실습입니다.

이제 새로운 데이터인 `Banana 1`에 같은 흐름을 스스로 적용합니다.

---

# PART 7. Mini Challenge — Banana 1로 1일차 전체 흐름을 스스로 반복합니다

---

# 30. Mini Challenge의 목표를 확인합니다

Mini Challenge는 새로운 알고리즘을 배우는 시간이 아닙니다.

오늘 Apple 데이터로 함께 진행한 전체 흐름을 **Banana 데이터에 스스로 다시 적용**합니다.

```text
Apple Red 1
→ 함께 실습

        ↓

Banana 1
→ 스스로 반복
```

오늘 작성한 스크립트를 그대로 사용합니다.

```text
01_check_dataset.py
02_prepare_split.py
03_augment_preview.py
04_augment_train.py
05_train_vae.py
06_qa_images.py
07_check_leakage.py
```

새로운 Python 파일을 만드는 것이 핵심이 아닙니다.

다음 질문에 스스로 답하는 것이 핵심입니다.

> **“데이터가 달라져도 오늘의 실험 절차를 다시 적용할 수 있는가?”**

---

# 31. Challenge 요구사항 1 — Banana 1 데이터를 준비하고 정상 여부를 확인합니다

Banana 1의 Training 이미지를 다음 위치에 넣습니다.

```text
data/banana/source_train/
```

Banana 1의 Test 이미지를 다음 위치에 넣습니다.

```text
data/banana/source_test/
```

다음 구조가 되면 됩니다.

```text
data/banana/
├─ source_train/
└─ source_test/
```

입력 데이터를 확인합니다.

```bash
python scripts/01_check_dataset.py \
  --dataset banana
```

먼저 다음을 기록합니다.

```text
Banana Training 수:

Banana Test 수:

전체 수:

이미지 크기:
```

---

# 32. Challenge 요구사항 2 — Banana의 Split을 직접 만듭니다

다음 코드를 실행합니다.

```bash
python scripts/02_prepare_split.py \
  --dataset banana
```

결과를 확인합니다.

```bash
cat reports/banana/split_manifest.csv | head
```

다음 표를 채웁니다.

| Split | 이미지 수 |
|---|---:|
| Train | |
| Validation | |
| Test | |

그리고 한 문장으로 답합니다.

> **왜 Banana Test를 Augmentation 전에 Train에 합치지 않았나요?**

---

# 33. Challenge 요구사항 3 — Banana 한 장에서 Augmentation 범위를 판단합니다

먼저 Apple과 같은 기본 조건을 사용합니다.

```bash
python scripts/03_augment_preview.py \
  --dataset banana \
  --rotation-angle 10
```

결과를 확인합니다.

그다음 Rotation만 바꾸어 봅니다.

예:

```bash
python scripts/03_augment_preview.py \
  --dataset banana \
  --rotation-angle 15
```

두 Grid를 비교합니다.

```text
results/banana/augmentation/augmentation_grid_rot10_0.jpg
VS
results/banana/augmentation/augmentation_grid_rot15_0.jpg
```

`preview_rotation.jpg` 같은 개별 Preview 파일은 두 번째 실행에서 덮어써질 수 있으므로, Rotation 10°와 15° 비교에는 이름이 서로 다른 **두 Grid 파일**을 사용합니다.

다음 질문에 답합니다.

```text
Banana의 길고 휘어진 형태에서
15° 회전도 자연스러운가?

객체가 프레임에서
부자연스럽게 잘리지는 않는가?

10°와 15° 중
오늘 Challenge에 사용할 범위는 무엇인가?
```

> 한 번에 Rotation과 Brightness와 Noise를 모두 바꾸지 않습니다.
>
> 이번 Challenge에서는 **Rotation 조건 하나만 판단**합니다.

---

# 34. Challenge 요구사항 4 — Banana Train을 자동 보강하고 Metadata를 확인합니다

10°를 선택했다면:

```bash
python scripts/04_augment_train.py \
  --dataset banana \
  --rotation-max 10
```

15°를 선택했다면:

```bash
python scripts/04_augment_train.py \
  --dataset banana \
  --rotation-max 15
```

완료 후 다음을 확인합니다.

```bash
head reports/banana/synthetic_metadata.csv
```

다음 항목을 기록합니다.

```text
Banana Train 원본 수:

생성된 Augmented 수:

선택한 Rotation Max:

선택 이유:
```

---

# 35. Challenge 요구사항 5 — Latent 32를 Validation에서 확인합니다

먼저 Apple과 같은 조건을 사용합니다.

```text
Epoch = 10
Latent = 32
Evaluation Split = Validation
```

실행합니다.

```bash
python scripts/05_train_vae.py \
  --dataset banana \
  --epochs 10 \
  --latent 32 \
  --eval-split val
```

결과를 확인합니다.

```text
results/banana/vae/
├─ reconstruction_val_latent32_e10.png
└─ generated_samples_latent32_e10.png
```

이 단계에서는 Test를 보지 않습니다.

Validation Reconstruction을 확인합니다.

```text
Validation Banana의 노란색은 유지되는가?

길고 휘어진 형태는 유지되는가?

세부 윤곽은 어느 정도 흐려지는가?
```

Random Sample은 생성 특성을 이해하기 위한 보조 결과로 확인합니다.

```text
실제 Banana 이미지를 입력하지 않았는데도
Banana와 비슷한 색·형태가 나타나는가?

완성도가 낮거나 흐릿하다면
현재 Epoch와 간단한 VAE 구조의 한계로 설명할 수 있는가?
```

> Latent 조건 선택의 기준은 **Validation Loss와 Validation Reconstruction**입니다. Random Sample은 생성 특성을 관찰하는 보조 결과이며, Test는 아직 사용하지 않습니다.

---

# 36. Challenge 요구사항 6 — Latent 하나만 바꾸고 Validation으로 선택합니다

이번에는 **Latent Dimension 하나만 변경**합니다.

기본:

```text
Latent = 32
```

비교:

```text
Latent = 16
```

두 실험 모두 같은 Train과 Validation, 같은 Epoch를 사용합니다.

Latent 16도 Test가 아니라 Validation에서 확인합니다.

```bash
python scripts/05_train_vae.py \
  --dataset banana \
  --epochs 10 \
  --latent 16 \
  --eval-split val
```

두 Validation Reconstruction과 Validation Loss를 비교합니다.

```text
[Validation Reconstruction]

reconstruction_val_latent32_e10.png
        VS
reconstruction_val_latent16_e10.png
```

```text
[Validation Loss]

vae_loss_latent32_e10.csv
        VS
vae_loss_latent16_e10.csv
```

Random Sample은 생성 특성을 비교하는 보조 자료로 함께 확인할 수 있습니다.

```text
generated_samples_latent32_e10.png
        VS
generated_samples_latent16_e10.png
```

다음 질문에 답합니다.

```text
Latent 32와 16에서
Validation 형태 복원은 어떻게 달랐는가?

Validation Loss는 어떻게 달랐는가?

두 조건 중 어떤 Latent를 선택할 것인가?

선택 근거는 무엇인가?

한 번에 하나의 변수만 바꾼 이유는 무엇인가?
```

> Epoch는 둘 다 10으로 유지합니다.
>
> Latent만 바꾸어야 결과 차이의 원인을 설명하기 쉽습니다.
>
> **Latent 선택에는 Test를 사용하지 않습니다.**

Validation 근거로 Latent를 하나 선택한 뒤 그 조건을 Freeze합니다.

아래 `SELECTED_LATENT` 값은 자신의 Validation 결과에 맞게 `32` 또는 `16`으로 설정합니다.

```bash
SELECTED_LATENT=32

python scripts/05_train_vae.py \
  --dataset banana \
  --epochs 10 \
  --latent "$SELECTED_LATENT" \
  --eval-split test \
  --eval-only
```

이 명령은 선택된 조건의 저장된 Model을 다시 학습하지 않고 불러와 Test Reconstruction만 만듭니다.

```text
Latent 32를 선택했다면
→ reconstruction_test_latent32_e10.png

Latent 16을 선택했다면
→ reconstruction_test_latent16_e10.png
```

최종 Test에서는 다음만 확인합니다.

```text
선택된 조건이
처음 보는 Banana Test에서도
색과 형태를 어느 정도 유지하는가?
```

Test 결과를 본 뒤 Latent를 다시 바꾸지 않습니다.

```text
Validation으로 조건 선택
        ↓
Latent Freeze
        ↓
Test 마지막 확인
```

---

# 37. Challenge 요구사항 7 — QA와 Leakage를 직접 확인합니다

Automatic QA를 실행합니다.

```bash
python scripts/06_qa_images.py \
  --dataset banana
```

`human_qa_grid.jpg`도 확인합니다.

다음 항목을 기록합니다.

```text
Automatic QA total:

Automatic QA pass:

Automatic QA fail:

Human QA에서 발견한 특징:
```

Leakage Check를 실행합니다.

```bash
python scripts/07_check_leakage.py \
  --dataset banana
```

정상이라면:

```text
Leakage check: PASS
```

가 표시됩니다.

---

# 38. Mini Challenge 결과를 `challenge_notes.md`로 정리합니다

**파일: `reports/banana/challenge_notes.md`**

다음 형식을 사용합니다.

```markdown
# Day01 Mini Challenge - Banana 1

## 1. 데이터

- Source Training:
- Source Test:
- Train:
- Validation:
- Test:

## 2. Data Leakage

- Test를 Train에 섞지 않은 이유:
- Leakage Check 결과:

## 3. Augmentation Preview

- 기본 Rotation:
- 비교 Rotation:
- 최종 선택:
- 선택 이유:

## 4. Train Augmentation

- 원본 Train 수:
- 생성 이미지 수:
- 사용한 방법:
- Metadata 확인 결과:

## 5. VAE 기본 실험

- Epoch:
- Latent:
- Train Loss 특징:
- Validation Loss 특징:
- Validation Reconstruction에서 유지된 특징:
- 흐려지거나 사라진 특징:
- Random Sample에서 확인한 색·형태:
- Random Sample의 한계:

## 6. VAE 조건 비교와 최종 선택

- 실험 A: Latent 32
- 실험 B: Latent 16
- 두 실험에서 동일하게 유지한 조건:
- Validation Reconstruction의 차이:
- Validation Loss의 차이:
- 선택한 Latent:
- 선택 근거:
- Test를 조건 선택에 사용하지 않았는가?:

## 7. Final Test

- Final Latent:
- Test Reconstruction 파일:
- Test에서 확인한 특징:
- Test를 본 뒤 Latent를 다시 변경하지 않았는가?:

## 8. QA

- Automatic QA:
- Human QA:
- 학습에 넣기 어렵다고 판단한 이미지가 있었는가?:
- 이유:

## 9. Apple 본 실습과 Banana Challenge 비교

- 공통으로 적용할 수 있었던 과정:
- 데이터가 달라지면서 다시 판단해야 했던 부분:
- Apple Reconstruction 특징:
- Banana Reconstruction 특징:

## 10. 최종 결론

- Augmentation과 VAE의 차이:
- Train / Validation / Test를 먼저 나누는 이유:
- 오늘 가장 중요하다고 생각한 데이터 관리 원칙:
```

복잡한 보고서를 작성하는 것이 목적은 아닙니다.

다음 관계를 자신의 결과로 설명할 수 있으면 됩니다.

```text
Real Data
        ↓
Split
        ↓
Train Augmentation
        ↓
Metadata
        ↓
VAE
        ↓
QA
        ↓
Leakage
        ↓
조건 비교
```

---

# 39. Mini Challenge 완료 조건을 확인합니다

아래 항목을 하나씩 직접 확인합니다.

- [ ] `Banana 1` Training과 Test를 올바른 폴더에 배치했다.
- [ ] 입력 이미지가 정상적으로 읽히는지 확인했다.
- [ ] Training에서 Train과 Validation을 만들었다.
- [ ] 기존 Test는 평가용으로 유지했다.
- [ ] Banana 한 장에서 Augmentation Preview를 확인했다.
- [ ] Rotation 10°와 15° 중 하나를 근거를 가지고 선택했다.
- [ ] Train에만 Augmentation을 적용했다.
- [ ] Metadata에서 원본·방법·Parameter·Seed를 확인했다.
- [ ] VAE Latent 32를 학습하고 Validation Reconstruction을 확인했다.
- [ ] VAE Latent 16을 학습하고 Validation Reconstruction을 확인했다.
- [ ] Epoch는 동일하게 유지하고 Latent만 변경했다.
- [ ] 두 Validation Loss와 Validation Reconstruction을 비교했다.
- [ ] Validation 근거로 최종 Latent를 선택했다.
- [ ] 최종 Latent를 Freeze한 뒤에만 Test Reconstruction을 한 번 확인했다.
- [ ] Test를 보고 Latent를 다시 변경하지 않았다.
- [ ] 두 Random Sample은 생성 특성을 이해하는 보조 결과로 비교했다.
- [ ] Automatic QA를 수행했다.
- [ ] Human QA Grid를 확인했다.
- [ ] Leakage Check를 수행했다.
- [ ] `challenge_notes.md`를 작성했다.
- [ ] Apple과 Banana 결과 차이를 한 문장 이상 설명했다.

---

# 40. Mini Challenge를 약 3시간으로 진행합니다

| 단계 | 권장 시간 |
|---|---:|
| Banana 데이터 확인·Split | 20분 |
| Augmentation Preview·Rotation 조건 선택 | 25분 |
| Train 전체 Augmentation·Metadata 확인 | 30분 |
| VAE Latent 32 · Validation 확인 | 30분 |
| VAE Latent 16 · Validation 확인 | 30분 |
| Validation 비교·Latent Freeze·Final Test | 10분 |
| Automatic QA·Human QA·Leakage | 20분 |
| Apple vs Banana 비교·`challenge_notes.md` 작성 | 15분 |
| **합계** | **180분** |

Mini Challenge에서는 새로운 코드를 만드는 데 시간을 쓰지 않습니다.

```text
새 알고리즘 추가
        X

오늘 만든 코드를
새 데이터에서 다시 실행
        ↓
조건 하나 변경
        ↓
결과를 읽고 설명
        O
```

---

# PART 8. 오늘 만든 결과를 정리하고 버전관리합니다

---

# 41. 오늘의 실험 기록을 `day01_notes.md`에 정리합니다

**파일: `day01_notes.md`**

다음 내용을 정리합니다.

```markdown
# Subject 11 - Day 01

## 1. 본 실습 데이터

- Dataset: Fruits-360
- Class: Apple Red 1
- Source Training:
- Source Test:
- Train:
- Validation:
- Test:

## 2. 사용한 Augmentation

- Brightness:
- Rotation:
- Blur:
- Noise:

## 3. Metadata

- 생성 이미지 수:
- 기록한 주요 항목:
- Seed:

## 4. VAE

- Epoch:
- Latent:
- Train Loss 특징:
- Validation Loss 특징:
- Reconstruction에서 유지된 특징:
- 사라지거나 흐려진 특징:
- Random Sample에서 확인한 특징:

## 5. QA

- Automatic QA 결과:
- Human QA에서 확인한 내용:

## 6. Leakage

- Leakage Check 결과:
- Test를 그대로 유지한 이유:

## 7. Mini Challenge

- Dataset: Banana 1
- 선택한 Rotation:
- Latent 32 Validation 결과:
- Latent 16 Validation 결과:
- 선택한 Latent:
- 선택 근거:
- Final Test Reconstruction:
- Apple과 다른 점:

## 8. 오늘의 결론

- Augmentation:
- VAE:
- 가장 중요한 데이터 관리 원칙:
```

---

# 42. README에 1일차 내용을 정리합니다

`README.md`에 다음과 같은 구조로 정리합니다.

````markdown
# Subject 11 - Synthetic Data

## Day 01 - Augmentation · VAE

### Main Dataset

- Fruits-360 `Apple Red 1`

### Mini Challenge Dataset

- Fruits-360 `Banana 1`

### Day 01 Pipeline

```text
Real Image
→ Train / Validation / Test
        ├─ Train Augmentation → Metadata
        └─ 원본 Train → VAE
                    → Reconstruction
                    → Random Sample
→ Augmented Image QA
→ Leakage Check
```

### Main Scripts

```text
scripts/
├─ 00_check_env.py
├─ 01_check_dataset.py
├─ 02_prepare_split.py
├─ 03_augment_preview.py
├─ 04_augment_train.py
├─ 05_train_vae.py
├─ 06_qa_images.py
└─ 07_check_leakage.py
```

### Key Results

- Augmentation Preview
- Synthetic Metadata
- VAE Model
- VAE Loss
- Validation Reconstruction for Condition Selection
- Final Test Reconstruction
- VAE Random Samples
- QA Result
- Leakage Check
- Banana Mini Challenge Notes

### Next

Day 02에서는 GAN·WGAN-GP·Diffusion을 이용하여
새로운 이미지 생성 방식을 비교합니다.
````

README에는 코드 전체를 복사하지 않습니다.

다음 내용만 빠르게 확인할 수 있으면 됩니다.

```text
무슨 데이터를 사용했는가?

무슨 코드를 실행하는가?

무슨 결과가 만들어지는가?

다음 수업으로 어떻게 이어지는가?
```

---

# 43. `.gitignore`를 작성합니다

원본 이미지·생성 이미지·모델·가상환경은 Git에 올리지 않습니다.

`.gitignore`에 다음 내용을 작성합니다.

```gitignore
# Python
.venv/
__pycache__/
*.pyc

# Input / generated image data
data/
results/

# Model
models/
*.pt
*.pth

# Large or generated reports
reports/*/qa_results.csv
reports/*/synthetic_metadata.csv
reports/*/vae_loss_*.csv

# Environment
reports/environment_freeze.txt

# Editor / OS
.vscode/
.DS_Store
```

다음과 같은 작은 문서 기록은 Git에 남길 수 있습니다.

```text
Python Source
README.md
day01_notes.md
challenge_notes.md
leakage_check.txt
```

기관 정책에 따라 원격 저장소 사용 범위는 달라질 수 있습니다.

원본 데이터는 저장소에 포함하지 않습니다.

---

# 44. Local Git에 1일차 결과를 저장합니다

교과 11은 2~5일차도 같은 프로젝트에 이어서 작업하므로 Git 저장소는 `day01_image_aug_vae` 안이 아니라 **교과 11 프로젝트 Root**에서 시작합니다.

먼저 Root로 이동합니다.

```bash
cd ~/ai_vision/subject11_synthetic_data
```

처음 한 번만 Git을 시작합니다.

```bash
git init
git branch -M main
```

상태를 확인합니다.

```bash
git status
```

1일차 폴더를 등록합니다.

```bash
git add day01_image_aug_vae
```

다시 확인합니다.

```bash
git status
```

다음 항목이 Git에 들어가지 않았는지 확인합니다.

```text
day01_image_aug_vae/data/
day01_image_aug_vae/results/
day01_image_aug_vae/models/
day01_image_aug_vae/.venv/
```

Commit합니다.

```bash
git commit -m "feat: complete subject11 day01 augmentation and VAE"
```

기록을 확인합니다.

```bash
git log --oneline
```

이렇게 시작하면 2~5일차 폴더도 같은 `subject11_synthetic_data` 저장소에 이어서 기록할 수 있습니다.

원격 저장소 사용이 허용된 환경이라면 연결된 저장소에 Push할 수 있습니다.

외부 저장소 사용이 제한된 환경에서는 Local Git 또는 기관 내부 Git만 사용합니다.

---

# 45. 1일차 자가 체크리스트를 확인합니다

오늘 배운 내용을 기준으로 직접 체크합니다.

- [ ] `Apple Red 1` Training과 Test의 역할을 구분할 수 있다.
- [ ] Training에서 Train과 Validation을 나누는 이유를 설명할 수 있다.
- [ ] Test를 학습 데이터 생성에 사용하지 않는 이유를 설명할 수 있다.
- [ ] Data Leakage가 무엇인지 설명할 수 있다.
- [ ] Brightness·Rotation·Blur·Noise의 의미를 설명할 수 있다.
- [ ] Augmentation Preview를 보고 변형 범위가 현실적인지 판단할 수 있다.
- [ ] Train에만 자동 Augmentation을 적용할 수 있다.
- [ ] Metadata에서 원본·방법·Parameter·Seed를 읽을 수 있다.
- [ ] VAE의 Encoder·Latent·Decoder 흐름을 설명할 수 있다.
- [ ] Train Loss와 Validation Loss의 역할을 구분할 수 있다.
- [ ] Test Original과 Reconstruction을 비교할 수 있다.
- [ ] VAE Random Sample과 Reconstruction의 차이를 설명할 수 있다.
- [ ] Automatic QA와 Human QA의 차이를 설명할 수 있다.
- [ ] Leakage Check 결과를 읽을 수 있다.
- [ ] `Banana 1`에서 같은 전체 Pipeline을 스스로 다시 실행했다.
- [ ] Mini Challenge에서 한 번에 하나의 조건만 변경했다.
- [ ] Latent 32와 16은 Validation Loss와 Validation Reconstruction으로 비교했다.
- [ ] Validation 근거로 Latent를 선택하고 Freeze했다.
- [ ] 선택된 Latent의 Test Reconstruction은 마지막에 한 번만 확인했다.
- [ ] Test를 본 뒤 Latent 조건을 다시 바꾸지 않았다.
- [ ] `day01_notes.md`와 `challenge_notes.md`를 작성했다.
- [ ] README를 정리했다.
- [ ] Git Commit을 완료했다.

---

# 46. 1일차에서 반드시 남겨야 할 세 가지 생각

## ① Augmentation

```text
기존 이미지를
현실적인 범위에서 변형하여
학습 조건을 다양하게 만드는 것
```

## ② VAE

```text
이미지를 특징 공간으로 압축하고
그 특징에서 다시 이미지를 복원하며

Latent Space에서 Sampling한 값으로
새로운 이미지를 만들어 보는
생성모델의 기본 구조
```

## ③ 데이터 관리

```text
먼저 Split
        ↓
Train에서만 데이터 생성
        ↓
Validation / Test 보호
        ↓
Metadata 기록
        ↓
QA
        ↓
Leakage 확인
```

> **합성데이터는 많이 만들었다고 끝나는 것이 아닙니다.**
>
> 어떤 원본에서 어떤 조건으로 만들었는지 기록하고, 사용할 수 있는 품질인지 확인하며, 평가 데이터가 학습에 섞이지 않았는지 검증해야 합니다.

---

# 다음 시간

2일차에는 VAE의 압축·복원에서 한 단계 더 나아갑니다.

```text
1일차

기존 이미지
→ Augmentation

이미지
→ Encoder
→ Latent
→ Decoder
→ Reconstruction / Random Sample

        ↓

2일차

Noise
→ GAN / WGAN-GP
→ 새로운 이미지 생성

Prompt / Seed
→ Diffusion
→ 새로운 이미지 생성
```

> **1일차에서 반드시 기억할 질문:**  
> 데이터를 새로 만들었다면, 그 데이터는 왜 만들었고, 어디에서 왔으며, 품질은 괜찮고, 평가 데이터와 분리되어 있는가?
