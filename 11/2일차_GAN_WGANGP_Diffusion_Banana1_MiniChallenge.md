## 오늘의 핵심 질문

> **1일차에 사용한 같은 실제 이미지의 특징을 생성모델이 학습하면, 기존 이미지를 직접 변형하지 않고도 새로운 이미지를 만들 수 있을까? 그리고 그 이미지를 실제 AI 학습데이터로 사용해도 될까?**

1일차에는 Fruits-360의 `Apple Red 1`을 이용하여 다음 두 가지 방법을 경험했습니다.

```text
실험 A
원본 Train
→ Brightness / Rotation / Blur / Noise
→ Augmented Image

실험 B
원본 Train
→ VAE
→ Latent Space
→ Reconstruction
→ Random Sample
```

2일차에는 **새로운 실제 데이터를 다시 수집하지 않습니다.**

1일차에서 이미 분리해 둔 `Apple Red 1`의 **Train 이미지**를 그대로 이어서 사용합니다.

다음 질문에 답할 수 있도록 **생성 방식과 생성 결과를 비교하고, 실제 학습데이터로 사용할 수 있는지 판단하는 과정**을 경험합니다.

```text
어떤 방법으로 이미지를 만들었는가?
        ↓
생성 결과는 얼마나 다양한가?
        ↓
실패하거나 이상한 이미지는 없는가?
        ↓
실제 Apple과 얼마나 다른가?
        ↓
학습데이터 후보로 사용할 수 있는가?
        ↓
어떤 조건으로 만들었는지 기록했는가?
```

---

# 오늘의 수업 목표

수업이 끝나면 다음 내용을 설명하고 직접 실행할 수 있어야 합니다.

- 1일차 프로젝트와 `.venv`를 그대로 이어서 사용할 수 있습니다.
- 1일차 `Apple Red 1`의 Train·Validation·Test 역할을 다시 구분할 수 있습니다.
- 2일차 GAN 학습에는 **1일차 Apple Train만 사용**해야 하는 이유를 설명할 수 있습니다.
- Augmentation·VAE·GAN·WGAN-GP·Diffusion의 차이를 설명할 수 있습니다.
- GAN의 Generator와 Discriminator 역할을 설명할 수 있습니다.
- Random Noise에서 DCGAN이 새로운 이미지를 만드는 흐름을 설명할 수 있습니다.
- DCGAN의 Epoch별 생성 결과와 Loss를 확인할 수 있습니다.
- Mode Collapse가 어떤 현상인지 생성 결과를 보고 설명할 수 있습니다.
- WGAN-GP의 Critic과 Gradient Penalty가 어떤 역할을 하는지 큰 흐름으로 설명할 수 있습니다.
- 같은 Apple Train으로 DCGAN과 WGAN-GP를 학습하고 결과를 비교할 수 있습니다.
- 사전학습 Diffusion 모델에 Prompt·Seed를 입력하여 Apple 이미지를 생성할 수 있습니다.
- Prompt를 고정하고 Seed만 바꾸는 실험을 할 수 있습니다.
- Seed를 고정하고 Prompt만 바꾸는 실험을 할 수 있습니다.
- Real Apple과 Synthetic Apple 사이의 Domain Gap을 확인할 수 있습니다.
- 생성 이미지를 Automatic QA와 Human QA로 확인할 수 있습니다.
- 생성 조건을 Metadata로 기록하는 이유를 설명할 수 있습니다.
- Mini Challenge에서 `Banana 1`을 사용하여 2일차의 핵심 생성·비교·QA 흐름을 스스로 다시 수행할 수 있습니다.
- 오늘 만든 코드와 결과를 README와 Git에 정리할 수 있습니다.

---

# 오늘 수업의 전체 흐름

![[Pasted image 20260908145106.png]]

2일차에는 1일차의 Apple Train 데이터를 그대로 사용하여 **DCGAN과 WGAN-GP가 새로운 이미지를 만드는 방식을 각각 실습하고 결과를 비교**합니다.  

이후 사전학습된 **Diffusion 모델로 Apple 이미지를 생성**해 보고, Real 이미지와 세 가지 생성 결과의 차이를 함께 확인합니다.  

마지막에는 생성된 이미지를 바로 사용하지 않고 **Domain Gap, Automatic QA, Human QA로 검수한 뒤 Banana 데이터로 같은 흐름을 스스로 반복**합니다.


```text
1일차 프로젝트 확인
        ↓
1일차 .venv 재사용
        ↓
2일차 폴더 생성
        ↓
Diffusion용 라이브러리·Local Model 확인
        ↓
1일차 Apple Train 경로 확인
        ↓
Environment Preflight
        ↓
GAN / Diffusion Smoke Test
        ↓
본 실습
        ↓

[실험 A — DCGAN]

Apple Train
        ↓
Random Noise
        ↓
Generator
        ↓
Fake Apple
        ↓
Discriminator
        ↓
Epoch별 생성 결과
        ↓
Final Samples
        ↓
Loss 확인
        ↓

[실험 B — WGAN-GP]

같은 Apple Train
        ↓
Random Noise
        ↓
Generator
        ↓
Fake Apple
        ↓
Critic
        ↓
Gradient Penalty
        ↓
Epoch별 생성 결과
        ↓
Final Samples
        ↓
Loss 확인
        ↓

DCGAN vs WGAN-GP
        ↓
형태 · 다양성 · 반복 패턴 비교
        ↓

[실험 C — Diffusion]

사전학습 Diffusion Model
        ↓
Apple Prompt
        ↓
Seed
        ↓
Apple Synthetic Image
        ↓
Prompt 고정 + Seed 변경
        ↓
Seed 고정 + Prompt 변경
        ↓

[생성데이터 검수]

Real Apple
VS
DCGAN Apple
WGAN-GP Apple
Diffusion Apple
        ↓
Domain Gap Review
        ↓
Automatic QA
        ↓
Human QA
        ↓
Metadata 확인
        ↓

Mini Challenge
Banana 1
        ↓
짧은 DCGAN / WGAN-GP 비교
        ↓
Banana Diffusion
        ↓
Real Banana vs Synthetic Banana
        ↓
QA · Domain Gap · 결과 해석
        ↓

README · Git 정리
        ↓
자가 체크
```

> **오늘의 데이터 운영 원칙**
>
> `Apple Red 1`의 **1일차 Train만** DCGAN과 WGAN-GP의 학습에 사용합니다.
>
> Validation과 Test는 생성모델 학습에 섞지 않습니다.
>
> Diffusion은 1일차 Apple로 처음부터 학습하는 것이 아니라, **사전에 준비된 Pretrained Diffusion Model**을 사용하여 Apple 조건의 이미지를 생성합니다.
>
> 오늘 생성한 이미지는 바로 학습데이터로 승인하지 않습니다. 먼저 Real과 비교하고 QA를 수행합니다.

---

# 오늘 사용할 실제 데이터

## 1. 본 실습 — 1일차 `Apple Red 1` Train

1일차에서 이미 다음 구조를 만들었습니다.

```text
day01_image_aug_vae/
└─ data/
   └─ apple/
      ├─ source_train/
      ├─ source_test/
      │
      └─ split/
         ├─ train/
         ├─ val/
         └─ test/
```

2일차의 DCGAN과 WGAN-GP가 실제로 사용하는 폴더는 다음입니다.

```text
../day01_image_aug_vae/data/apple/split/train/
```

예를 들어 1일차에서 `Apple Red 1` Training 492장을 사용했다면 약 15%를 Validation으로 나누었으므로 Train은 약 419장이 됩니다.

실제 수량은 오늘 코드로 다시 확인합니다.

```text
Apple Train
→ DCGAN 학습
→ WGAN-GP 학습

Apple Validation
→ 오늘 GAN 학습에 사용하지 않음

Apple Test
→ 오늘 GAN 학습에 사용하지 않음
```

### 왜 같은 Apple Train을 계속 사용하나요?

DCGAN과 WGAN-GP는 **같은 Apple Train을 사용해야 학습 방식의 차이를 비교하기 쉽습니다.**

1일차 VAE도 같은 Apple Train을 사용했으므로, VAE·DCGAN·WGAN-GP는 동일한 실제 데이터에서 서로 다른 생성모델의 특징을 비교할 수 있습니다.

Diffusion은 조금 다릅니다. 오늘 사용하는 Diffusion은 Apple Train으로 새로 학습하지 않고 **사전학습 모델에 Prompt와 Seed를 입력하여 이미지를 생성**합니다. 따라서 Apple Train은 Diffusion의 학습 입력이 아니라 **생성 결과와 비교할 Real Reference** 역할을 합니다.

```text
Apple Train
    ├─ VAE 학습
    ├─ DCGAN 학습
    └─ WGAN-GP 학습

Pretrained Diffusion
    └─ Prompt + Seed로 생성

        ↓

모든 결과를
같은 Real Apple 기준으로 비교
```

이 구분을 해 두면 “같은 데이터로 학습한 모델 비교”와 “사전학습 모델의 생성 결과 비교”를 혼동하지 않습니다.

---

## 2. Diffusion — Apple을 표현하는 Prompt

Diffusion에서는 실제 Apple Train 이미지를 모델 학습에 넣지 않습니다.

대신 사전학습 모델에 다음과 같은 조건을 입력합니다.

```text
a single red apple on a clean white background,
centered object,
realistic product photography,
neutral lighting
```

오늘은 Prompt와 Seed를 바꾸면서 결과를 비교합니다.

```text
같은 Prompt + 다른 Seed
→ 결과 다양성 확인

같은 Seed + 다른 Prompt
→ 조건 변화 확인
```

---

## 3. Mini Challenge — 1일차 `Banana 1`

Mini Challenge에서도 새로운 데이터를 수집하지 않습니다.

1일차 Mini Challenge에서 준비한 `Banana 1`을 그대로 사용합니다.

```text
../day01_image_aug_vae/data/banana/split/train/
```

2일차 Challenge에서는 Banana Train만 생성모델 학습에 사용합니다.

> 1일차 Banana Mini Challenge를 완료하지 못했다면 2일차 Challenge를 시작하기 전에 1일차의 Banana 데이터 확인과 Split 단계까지 먼저 완료합니다.

---

# 오늘 배울 기술 스택

| 기술 | 오늘 하는 일 | 쉽게 말하면 |
|---|---|---|
| Python | 생성모델 학습·생성·검수 코드 작성 | 오늘 실험 전체를 실행 |
| pathlib | 1일차와 2일차 경로 연결 | 같은 데이터를 안전하게 재사용 |
| PyTorch | DCGAN·WGAN-GP 구현 | 생성모델 학습 |
| TorchVision | 이미지 전처리·Grid 저장 | 64×64 학습 이미지와 결과 저장 |
| Pillow | 이미지 읽기·비교 이미지 생성 | 결과 확인용 이미지 처리 |
| NumPy | Seed·배열 계산 | 재현 가능한 실험 보조 |
| Pandas | Loss·Metadata·QA CSV 저장 | 실험 결과 기록 |
| DCGAN | Noise에서 새로운 이미지 생성 | Generator와 Discriminator 경쟁 |
| WGAN-GP | GAN 학습 안정성 비교 | Critic + Gradient Penalty |
| Diffusers | Pretrained Diffusion 실행 | Prompt·Seed로 이미지 생성 |
| Prompt | 생성 조건 지정 | 어떤 이미지를 만들지 설명 |
| Seed | Random 결과 재현 | 같은 조건을 다시 만들기 |
| Domain Gap | Real과 Synthetic 차이 확인 | 실제 환경과 얼마나 다른지 판단 |
| Automatic / Human QA | 생성 이미지 검수 | 파일 검사 + 사람이 의미 확인 |
| Metadata | 생성 조건 기록 | 어떤 모델·Seed·Prompt로 만들었는지 저장 |
| Git | 2일차 코드와 기록 관리 | 실험을 다시 확인할 수 있게 저장 |

---

# 오늘의 8시간 운영 구성

2일차는 본 실습 약 5시간과 Mini Challenge 약 3시간으로 진행합니다.

| 단계 | 권장 시간 | 내용 |
|---|---:|---|
| 프로젝트·환경·데이터 확인 | 30분 | 1일차 연결, `.venv`, Apple Train, Diffusion 준비 확인 |
| DCGAN | 55분 | 구조 이해, Smoke Test, 본 학습, Loss·Mode Collapse 확인 |
| WGAN-GP | 70분 | Critic·Gradient Penalty 이해, Smoke Test, 본 학습 |
| GAN 결과 비교 | 20분 | DCGAN vs WGAN-GP Grid와 실패 패턴 비교 |
| Diffusion | 50분 | Pretrained Model, Prompt·Seed 실험, Metadata |
| Domain Gap·QA | 55분 | Real vs Synthetic, Automatic QA, Human QA |
| 본 실습 정리 | 20분 | 산출물과 핵심 결과 기록 |
| Mini Challenge | 180분 | Banana로 같은 생성·비교·QA 흐름 반복 |
| **합계** | **480분** | **8시간** |

> GAN과 Diffusion의 실행시간은 GPU 성능에 따라 달라질 수 있습니다. Smoke Test가 정상인지 먼저 확인한 뒤 본 학습을 실행하며, 시간이 크게 초과되면 Epoch를 줄이고 실제 사용한 값을 기록합니다.

---

# PART 1. 1일차 프로젝트에서 2일차를 이어서 시작합니다

---

# 1. 1일차와 2일차가 어떻게 연결되는지 확인합니다

```text
1일차

Augmentation
→ 기존 이미지 직접 변형

VAE
→ Encoder
→ Latent
→ Decoder
→ Reconstruction / Random Sample


2일차

DCGAN
→ Noise
→ Generator
→ 새로운 이미지

WGAN-GP
→ Noise
→ Generator
→ 새로운 이미지
→ Critic + Gradient Penalty

Diffusion
→ Prompt + Seed
→ Denoising
→ 새로운 이미지
```

오늘도 1일차에서 배운 데이터 관리 원칙은 그대로 유지됩니다.

```text
Train만 생성모델 학습에 사용
        ↓
생성 조건 기록
        ↓
QA
        ↓
Real과 비교
        ↓
사용 여부 판단
```

---

# 2. 교과 11 프로젝트 Root로 이동합니다

```bash
cd ~/ai_vision/subject11_synthetic_data
```

1일차 폴더를 확인합니다.

```bash
ls
```

다음 폴더가 보여야 합니다.

```text
day01_image_aug_vae
```

---

# 3. 2일차 폴더를 만듭니다

```bash
mkdir -p day02_generative_models
cd day02_generative_models
```

오늘 필요한 폴더를 만듭니다.

```bash
mkdir -p \
src \
scripts \
results/apple/dcgan/epochs \
results/apple/dcgan/final \
results/apple/wgan_gp/epochs \
results/apple/wgan_gp/final \
results/apple/diffusion \
results/apple/compare \
results/banana/dcgan/epochs \
results/banana/dcgan/final \
results/banana/wgan_gp/epochs \
results/banana/wgan_gp/final \
results/banana/diffusion \
results/banana/compare \
models/apple \
models/banana \
reports/apple \
reports/banana
```

파일을 준비합니다.

```bash
touch src/day02_common.py
touch scripts/00_check_day02.py
touch scripts/01_train_dcgan.py
touch scripts/02_train_wgan_gp.py
touch scripts/03_compare_gan_results.py
touch scripts/04_diffusion_generate.py
touch scripts/05_domain_gap_review.py
touch scripts/06_synthetic_qa.py
touch requirements_day02.txt
touch day02_notes.md
touch README.md
touch .gitignore
```

### 지금 무엇을 한 것인가요?

```text
1일차
→ Real Data와 VAE 결과 보관

2일차
→ 생성모델 코드와 결과를 별도 관리

apple
→ 본 실습

banana
→ Mini Challenge
```

---

# 4. 1일차 가상환경을 그대로 사용합니다

```bash
source ../day01_image_aug_vae/.venv/bin/activate
```

Python 위치를 확인합니다.

```bash
which python
```

PyTorch와 CUDA도 확인합니다.

```bash
python -c "import torch; print(torch.__version__); print('CUDA:', torch.cuda.is_available())"
```

---

# 5. Diffusion 실행환경과 Local Model을 먼저 확인합니다

DCGAN과 WGAN-GP는 1일차의 PyTorch 환경을 그대로 사용합니다.

Diffusion은 모델 파일과 추가 라이브러리가 준비되어 있어야 하므로, 본 실습보다 먼저 **환경과 Model 위치를 확인**합니다.

```text
1일차 .venv
        ↓
Diffusers · Transformers · Accelerate 확인
        ↓
DIFFUSION_MODEL_ID 확인
        ↓
Local Model이면 model_index.json 확인
        ↓
Environment Check
        ↓
Diffusion 1장 Smoke Test
        ↓
Prompt / Seed 본 실험
```

Diffusion 실습에 필요한 패키지는 별도 파일에 기록합니다.

**파일: `requirements_day02.txt`**

```text
diffusers
transformers
accelerate
safetensors
```

필요한 패키지가 아직 없다면 준비된 설치 경로에서 설치합니다.

```bash
pip install -r requirements_day02.txt
```

인터넷 사용이 제한된 환경에서는 수업 중 외부에서 대용량 패키지나 Model을 새로 받는 방식에 의존하지 않습니다. 제공된 Local Package, 사전 설치 환경 또는 기관 내부 저장소를 사용합니다.

설치 여부를 확인합니다.

```bash
python -c "import diffusers, transformers, accelerate; print('Diffusion libraries OK')"
```

오늘 Diffusion 실습은 **SD-Turbo 계열의 Diffusers 형식 모델**을 기준으로 진행합니다.

로컬 모델을 제공받았다면 환경변수에 모델 폴더를 지정합니다.

예:

```bash
export DIFFUSION_MODEL_ID="$HOME/ai_vision/models/sd-turbo"
```

Local Model을 사용하는 경우 다음 파일이 있는지 확인합니다.

```bash
ls "$DIFFUSION_MODEL_ID/model_index.json"
```

`model_index.json`이 보이면 오늘 코드가 Diffusers의 `from_pretrained()`로 읽을 수 있는 기본 구조가 준비된 것입니다. 이 단계에서는 아직 이미지를 생성하지 않고 **Model 위치와 형식만 확인**합니다.

인터넷 사용이 허용되고 모델 다운로드가 가능한 환경에서는 다음처럼 Model ID를 지정할 수도 있습니다.

```bash
export DIFFUSION_MODEL_ID="stabilityai/sd-turbo"
```

현재 설정값을 확인합니다.

```bash
echo "$DIFFUSION_MODEL_ID"
```

> 인터넷 접속이 제한된 환경에서는 패키지와 모델을 수업 전에 제공된 로컬 파일 또는 기관 내부 저장소에서 준비해야 합니다.
>
> 오늘 사용하는 `guidance_scale=0.0`, 적은 Inference Step 설정은 **SD-Turbo 기준**입니다. 다른 Diffusion 모델을 사용하면 권장 설정이 달라질 수 있습니다.

2일차 설치 상태도 기록합니다.

```bash
pip freeze > reports/environment_freeze_day02.txt
```

---

# 6. Apple과 Banana에서 함께 사용할 공통 경로 코드를 작성합니다

**파일: `src/day02_common.py`**

```python
from __future__ import annotations

from pathlib import Path


ROOT = Path(__file__).resolve().parents[1]
SUBJECT_ROOT = ROOT.parent
DAY01_ROOT = SUBJECT_ROOT / "day01_image_aug_vae"

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


def day01_train_dir(dataset: str) -> Path:
    if dataset not in {"apple", "banana"}:
        raise ValueError(
            f"지원하지 않는 dataset: {dataset}"
        )

    return (
        DAY01_ROOT
        / "data"
        / dataset
        / "split"
        / "train"
    )


def day02_paths(dataset: str) -> dict[str, Path]:
    result_root = ROOT / "results" / dataset
    model_root = ROOT / "models" / dataset
    report_root = ROOT / "reports" / dataset

    paths = {
        "train": day01_train_dir(dataset),
        "dcgan_epochs": result_root / "dcgan" / "epochs",
        "dcgan_final": result_root / "dcgan" / "final",
        "wgan_epochs": result_root / "wgan_gp" / "epochs",
        "wgan_final": result_root / "wgan_gp" / "final",
        "diffusion": result_root / "diffusion",
        "compare": result_root / "compare",
        "model": model_root,
        "report": report_root,
    }

    for key, path in paths.items():
        if key == "train":
            continue

        path.mkdir(parents=True, exist_ok=True)

    return paths
```

### 이 코드는 왜 작성하나요?

본 실습에서는 Apple, Mini Challenge에서는 Banana를 사용하지만 생성모델 코드는 같습니다.

```text
--dataset apple
→ 1일차 Apple Train 사용

--dataset banana
→ 1일차 Banana Train 사용
```

### 의사코드로 읽어보기

```text
2일차 Root를 찾는다
        ↓
1일차 Root를 찾는다
        ↓
dataset이 apple / banana인지 확인한다
        ↓
1일차 해당 Train 경로를 만든다
        ↓
2일차 결과·모델·리포트 경로를 만든다
        ↓
필요한 폴더를 자동 생성한다
```

---

# 7. 환경과 1일차 Train 데이터를 한 번에 확인하는 코드를 작성합니다

**파일: `scripts/00_check_day02.py`**

```python
from __future__ import annotations

import argparse
import os
import sys
from pathlib import Path

import accelerate
import cv2
import diffusers
import safetensors
import torch
import torchvision
import transformers


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day02_common import (
    day02_paths,
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


def main() -> None:
    args = parse_args()
    paths = day02_paths(args.dataset)

    print("=" * 60)
    print("Subject 11 - Day 02 Check")
    print("=" * 60)

    print("PyTorch      :", torch.__version__)
    print("TorchVision  :", torchvision.__version__)
    print("Diffusers    :", diffusers.__version__)
    print("Transformers :", transformers.__version__)
    print("Accelerate   :", accelerate.__version__)
    print("Safetensors  :", safetensors.__version__)

    cuda_available = torch.cuda.is_available()

    print("CUDA Available :", cuda_available)

    if cuda_available:
        print(
            "GPU            :",
            torch.cuda.get_device_name(0),
        )
    else:
        print("GPU            : CPU mode")

    print("-" * 60)

    model_id = os.environ.get(
        "DIFFUSION_MODEL_ID",
        "",
    )

    if not model_id:
        raise RuntimeError(
            "DIFFUSION_MODEL_ID가 설정되지 않았습니다. "
            "제공된 Local Model 경로 또는 사용할 Model ID를 "
            "먼저 지정하세요."
        )

    print(
        "Diffusion Model:",
        model_id,
    )

    local_model = Path(
        model_id
    ).expanduser()

    looks_like_local_path = (
        model_id.startswith("/")
        or model_id.startswith(".")
        or model_id.startswith("~")
    )

    if local_model.exists():
        if not local_model.is_dir():
            raise RuntimeError(
                "DIFFUSION_MODEL_ID가 Local 경로이지만 "
                f"폴더가 아닙니다: {local_model}"
            )

        model_index = (
            local_model
            / "model_index.json"
        )

        if not model_index.exists():
            raise RuntimeError(
                "Local Diffusion Model에 "
                f"model_index.json이 없습니다: {model_index}"
            )

        print(
            "Diffusion Source:",
            "LOCAL MODEL",
        )

        print(
            "model_index.json:",
            model_index,
        )

    elif looks_like_local_path:
        raise RuntimeError(
            "지정한 Local Diffusion Model 경로가 없습니다: "
            f"{local_model}"
        )

    else:
        print(
            "Diffusion Source:",
            "MODEL ID / CACHE",
        )

        print(
            "[INFO] 인터넷 제한 환경에서는 "
            "이 Model이 Local Cache 또는 기관 내부 환경에 "
            "미리 준비되어 있어야 합니다."
        )

    print("-" * 60)

    train_dir = paths["train"]

    if not train_dir.exists():
        raise RuntimeError(
            f"1일차 Train 폴더가 없습니다: {train_dir}"
        )

    image_files = list_images(train_dir)

    if not image_files:
        raise RuntimeError(
            f"1일차 Train 이미지가 없습니다: {train_dir}"
        )

    unreadable = []

    for path in image_files:
        image = cv2.imread(str(path))

        if image is None:
            unreadable.append(path.name)

    if unreadable:
        raise RuntimeError(
            "읽을 수 없는 이미지가 있습니다: "
            + ", ".join(unreadable[:5])
        )

    first = cv2.imread(
        str(image_files[0])
    )

    height, width = first.shape[:2]

    print("Dataset      :", args.dataset)
    print("Train Dir    :", train_dir)
    print("Train Images :", len(image_files))
    print("First Size   :", f"{width} x {height}")

    if len(image_files) < 100:
        print(
            "[WARNING] 이미지 수가 적어 "
            "GAN 생성 품질이 불안정할 수 있습니다."
        )

    print("=" * 60)
    print("Day 02 preflight: READY")


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

2일차에서는 본 학습을 시작하기 전에 **1일차 Train 연결과 생성환경이 모두 준비되어 있는지** 확인합니다. GAN은 Train 경로와 PyTorch를, Diffusion은 추가 라이브러리와 Model 위치까지 먼저 확인합니다.

### 의사코드로 읽어보기

```text
PyTorch / TorchVision을 확인한다
        ↓
Diffusers / Transformers / Accelerate / Safetensors를 확인한다
        ↓
CUDA 사용 가능 여부를 확인한다
        ↓
DIFFUSION_MODEL_ID를 확인한다
        ↓
Local Model이면 model_index.json을 확인한다
        ↓
1일차 Train 폴더를 찾는다
        ↓
Train 이미지와 대표 크기를 확인한다
        ↓
모두 정상이면 Preflight READY
```

본 실습에서는 Apple을 확인합니다.

```bash
python scripts/00_check_day02.py --dataset apple
```

`Day 02 preflight: READY`가 확인되면 긴 실행으로 바로 들어가지 않습니다.

```text
Environment Preflight
        ↓
DCGAN 1 Epoch Smoke Test
        ↓
WGAN-GP 1 Epoch Smoke Test
        ↓
Diffusion 1장 Smoke Test
        ↓
본 학습 / 본 생성
```

Smoke Test의 목적은 좋은 결과를 얻는 것이 아니라 **경로·GPU·Model·저장 과정이 실제로 동작하는지 빠르게 확인하는 것**입니다.

---

# PART 2. DCGAN으로 Apple의 새로운 이미지를 생성합니다

---

# 8. GAN의 Generator와 Discriminator를 이해합니다

2일차에서 GAN 이론을 깊게 증명하지 않습니다. **두 모델이 서로 다른 역할을 하며 경쟁한다는 흐름**을 이해하면 됩니다.

```text
Random Noise
        ↓
Generator
        ↓
Fake Apple
        ↓
Discriminator
        ↑
Real Apple
```

`Generator`는 Real Apple과 비슷한 이미지를 만들려고 하고, `Discriminator`는 Real과 Fake를 구분하려고 합니다.

---

# 9. DCGAN의 이미지 크기 흐름을 확인합니다

```text
Generator

Noise
100 × 1 × 1
        ↓
4 × 4
        ↓
8 × 8
        ↓
16 × 16
        ↓
32 × 32
        ↓
64 × 64 RGB
```

원본은 100×100이지만 오늘 GAN은 64×64로 학습합니다.

---

# 10. DCGAN 학습 코드를 작성합니다

**파일: `scripts/01_train_dcgan.py`**

```python
from __future__ import annotations

import argparse
import csv
import random
import shutil
import sys
from pathlib import Path

import numpy as np
import pandas as pd
import torch
import torch.nn as nn

from PIL import Image
from torch.utils.data import DataLoader, Dataset
from torchvision import transforms
from torchvision.utils import save_image


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day02_common import day02_paths, list_images


IMAGE_SIZE = 64
CHANNELS = 3
NOISE_DIM = 100
BATCH_SIZE = 32
SEED = 42
VISUAL_SEED = 2026


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
        default=15,
    )

    return parser.parse_args()


def set_seed(seed: int) -> None:
    random.seed(seed)
    np.random.seed(seed)
    torch.manual_seed(seed)

    if torch.cuda.is_available():
        torch.cuda.manual_seed_all(seed)


class FlatImageDataset(Dataset):
    def __init__(self, root_dir: Path):
        self.files = list_images(root_dir)

        if not self.files:
            raise RuntimeError(
                f"No images found: {root_dir}"
            )

        self.transform = transforms.Compose(
            [
                transforms.Resize(
                    (IMAGE_SIZE, IMAGE_SIZE)
                ),
                transforms.ToTensor(),
                transforms.Normalize(
                    (0.5, 0.5, 0.5),
                    (0.5, 0.5, 0.5),
                ),
            ]
        )

    def __len__(self):
        return len(self.files)

    def __getitem__(self, index):
        image = Image.open(
            self.files[index]
        ).convert("RGB")

        return self.transform(image)


class Generator(nn.Module):
    def __init__(self):
        super().__init__()

        self.model = nn.Sequential(
            nn.ConvTranspose2d(
                NOISE_DIM, 512, 4, 1, 0, bias=False
            ),
            nn.BatchNorm2d(512),
            nn.ReLU(True),

            nn.ConvTranspose2d(
                512, 256, 4, 2, 1, bias=False
            ),
            nn.BatchNorm2d(256),
            nn.ReLU(True),

            nn.ConvTranspose2d(
                256, 128, 4, 2, 1, bias=False
            ),
            nn.BatchNorm2d(128),
            nn.ReLU(True),

            nn.ConvTranspose2d(
                128, 64, 4, 2, 1, bias=False
            ),
            nn.BatchNorm2d(64),
            nn.ReLU(True),

            nn.ConvTranspose2d(
                64, CHANNELS, 4, 2, 1, bias=False
            ),
            nn.Tanh(),
        )

    def forward(self, noise):
        return self.model(noise)


class Discriminator(nn.Module):
    def __init__(self):
        super().__init__()

        self.model = nn.Sequential(
            nn.Conv2d(
                CHANNELS, 64, 4, 2, 1, bias=False
            ),
            nn.LeakyReLU(0.2, inplace=True),

            nn.Conv2d(
                64, 128, 4, 2, 1, bias=False
            ),
            nn.BatchNorm2d(128),
            nn.LeakyReLU(0.2, inplace=True),

            nn.Conv2d(
                128, 256, 4, 2, 1, bias=False
            ),
            nn.BatchNorm2d(256),
            nn.LeakyReLU(0.2, inplace=True),

            nn.Conv2d(
                256, 512, 4, 2, 1, bias=False
            ),
            nn.BatchNorm2d(512),
            nn.LeakyReLU(0.2, inplace=True),

            nn.Conv2d(
                512, 1, 4, 1, 0, bias=False
            ),
            nn.Sigmoid(),
        )

    def forward(self, image):
        return self.model(image).view(-1)


def reset_directory(directory: Path) -> None:
    if directory.exists():
        shutil.rmtree(directory)

    directory.mkdir(
        parents=True,
        exist_ok=True,
    )


def make_noise(
    count: int,
    device,
    seed: int,
):
    noise_generator = torch.Generator(
        device=device
    ).manual_seed(seed)

    return torch.randn(
        count,
        NOISE_DIM,
        1,
        1,
        generator=noise_generator,
        device=device,
    )


def save_final_samples(
    generator,
    output_dir,
    device,
) -> None:
    generator.eval()

    final_noise = make_noise(
        16,
        device,
        VISUAL_SEED + 1,
    )

    with torch.no_grad():
        samples = generator(final_noise)
        samples = (samples + 1) / 2

    for index, image in enumerate(
        samples,
        start=1,
    ):
        save_image(
            image,
            output_dir / f"sample_{index:02d}.png",
        )


def main() -> None:
    args = parse_args()

    if args.epochs < 1:
        raise ValueError(
            "--epochs는 1 이상이어야 합니다."
        )

    set_seed(SEED)
    paths = day02_paths(args.dataset)

    reset_directory(
        paths["dcgan_epochs"]
    )

    reset_directory(
        paths["dcgan_final"]
    )

    device = torch.device(
        "cuda"
        if torch.cuda.is_available()
        else "cpu"
    )

    print("Device :", device)
    print("Dataset:", args.dataset)

    dataset = FlatImageDataset(
        paths["train"]
    )

    loader_generator = torch.Generator()
    loader_generator.manual_seed(SEED)

    loader = DataLoader(
        dataset,
        batch_size=BATCH_SIZE,
        shuffle=True,
        generator=loader_generator,
        drop_last=False,
    )

    generator = Generator().to(device)
    discriminator = Discriminator().to(
        device
    )

    criterion = nn.BCELoss()

    optimizer_g = torch.optim.Adam(
        generator.parameters(),
        lr=0.0002,
        betas=(0.5, 0.999),
    )

    optimizer_d = torch.optim.Adam(
        discriminator.parameters(),
        lr=0.0002,
        betas=(0.5, 0.999),
    )

    fixed_noise = make_noise(
        64,
        device,
        VISUAL_SEED,
    )

    history = []

    for epoch in range(
        1,
        args.epochs + 1,
    ):
        generator.train()
        discriminator.train()

        epoch_d = 0.0
        epoch_g = 0.0
        image_count = 0

        for real_images in loader:
            real_images = real_images.to(
                device
            )

            batch_size = real_images.size(
                0
            )

            real_labels = torch.ones(
                batch_size,
                device=device,
            )

            fake_labels = torch.zeros(
                batch_size,
                device=device,
            )

            optimizer_d.zero_grad()

            real_output = discriminator(
                real_images
            )

            loss_real = criterion(
                real_output,
                real_labels,
            )

            noise = torch.randn(
                batch_size,
                NOISE_DIM,
                1,
                1,
                device=device,
            )

            fake_images = generator(
                noise
            )

            fake_output = discriminator(
                fake_images.detach()
            )

            loss_fake = criterion(
                fake_output,
                fake_labels,
            )

            loss_d = (
                loss_real
                + loss_fake
            )

            loss_d.backward()
            optimizer_d.step()

            optimizer_g.zero_grad()

            fake_output = discriminator(
                fake_images
            )

            loss_g = criterion(
                fake_output,
                real_labels,
            )

            loss_g.backward()
            optimizer_g.step()

            epoch_d += (
                loss_d.item()
                * batch_size
            )

            epoch_g += (
                loss_g.item()
                * batch_size
            )

            image_count += batch_size

        avg_d = epoch_d / image_count
        avg_g = epoch_g / image_count

        history.append(
            {
                "epoch": epoch,
                "d_loss": avg_d,
                "g_loss": avg_g,
            }
        )

        print(
            f"Epoch {epoch:02d}/"
            f"{args.epochs} "
            f"D={avg_d:.4f} "
            f"G={avg_g:.4f}"
        )

        generator.eval()

        with torch.no_grad():
            samples = generator(
                fixed_noise
            )

            samples = (
                samples + 1
            ) / 2

            save_image(
                samples,
                paths["dcgan_epochs"]
                / f"epoch_{epoch:03d}.png",
                nrow=8,
            )

    model_g = (
        paths["model"]
        / "dcgan_generator.pt"
    )

    model_d = (
        paths["model"]
        / "dcgan_discriminator.pt"
    )

    torch.save(
        generator.state_dict(),
        model_g,
    )

    torch.save(
        discriminator.state_dict(),
        model_d,
    )

    loss_path = (
        paths["report"]
        / "dcgan_loss.csv"
    )

    pd.DataFrame(
        history
    ).to_csv(
        loss_path,
        index=False,
        encoding="utf-8-sig",
    )

    save_final_samples(
        generator,
        paths["dcgan_final"],
        device,
    )

    metadata_path = (
        paths["report"]
        / "dcgan_metadata.csv"
    )

    with metadata_path.open(
        "w",
        newline="",
        encoding="utf-8-sig",
    ) as file:
        writer = csv.DictWriter(
            file,
            fieldnames=[
                "method",
                "dataset",
                "epochs",
                "batch_size",
                "noise_dim",
                "seed",
                "epoch_visual_seed",
                "final_sample_seed",
                "learning_rate",
                "model_generator",
                "model_discriminator",
            ],
        )

        writer.writeheader()

        writer.writerow(
            {
                "method": "dcgan",
                "dataset": args.dataset,
                "epochs": args.epochs,
                "batch_size": BATCH_SIZE,
                "noise_dim": NOISE_DIM,
                "seed": SEED,
                "epoch_visual_seed": VISUAL_SEED,
                "final_sample_seed": VISUAL_SEED + 1,
                "learning_rate": 0.0002,
                "model_generator": str(
                    model_g
                ),
                "model_discriminator": str(
                    model_d
                ),
            }
        )

    print("DCGAN Training Completed")
    print("Loss    :", loss_path)
    print(
        "Samples :",
        paths["dcgan_final"],
    )


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

1일차 Apple Train의 특징을 Generator가 학습하고, Epoch가 진행되면서 생성 결과가 어떻게 달라지는지 보기 위해 작성합니다.

### 의사코드로 읽어보기

```text
Apple Train을 64×64로 준비한다
        ↓
Generator와 Discriminator를 만든다
        ↓
Noise로 Fake Apple을 만든다
        ↓
Discriminator가 Real / Fake를 구분하도록 학습한다
        ↓
Generator가 Fake를 Real처럼 만들도록 학습한다
        ↓
Epoch마다 같은 fixed_noise로 결과를 저장한다
        ↓
학습이 끝나면 모델과 Loss를 저장한다
        ↓
새 Noise로 Final Image 16장을 만든다
        ↓
학습 조건을 Metadata로 기록한다
```

### 코드에서 꼭 확인할 세 부분

`transforms.Normalize((0.5, ...), (0.5, ...))`는 Real Image의 픽셀 범위를 약 `-1 ~ 1`로 바꿉니다. Generator의 마지막 Layer가 `Tanh()`이기 때문에 Real과 Fake의 값 범위를 맞추기 위한 처리입니다.

`fake_images.detach()`는 Discriminator를 학습하는 순간에는 Generator까지 함께 업데이트하지 않도록 계산 그래프를 끊는 역할을 합니다. 이후 Generator를 학습할 때는 `detach()`하지 않은 Fake Image를 다시 사용합니다.

`fixed_noise`는 Epoch마다 같은 Noise를 넣어 **같은 입력에서 Generator가 어떻게 바뀌는지** 보기 위한 기준입니다.

오늘 코드는 두 Seed를 구분합니다.

```text
epoch_visual_seed = 2026
→ Epoch별 변화 비교용 fixed_noise

final_sample_seed = 2027
→ 최종 16장 생성용 Noise
```

DCGAN과 WGAN-GP 모두 같은 두 Seed를 사용하므로 **Epoch 비교와 Final Sample 비교의 Noise 조건을 각각 맞출 수 있습니다.** Metadata에도 두 값을 따로 기록합니다.

---

# 11. DCGAN을 실행합니다

먼저 Smoke Test를 합니다.

```bash
python scripts/01_train_dcgan.py \
  --dataset apple \
  --epochs 1
```

정상이라면 본 실습을 실행합니다.

```bash
python scripts/01_train_dcgan.py \
  --dataset apple \
  --epochs 15
```

> Smoke Test와 본 실습은 각각 새 학습으로 시작합니다. 본 실습을 실행하면 `results/apple/dcgan/`의 Smoke Test 결과는 비우고 본 학습 결과를 다시 저장합니다.
>
> GPU 상황 때문에 `15 Epoch`를 줄였다면 실제 사용한 Epoch를 `dcgan_metadata.csv`와 `day02_notes.md`에서 확인할 수 있도록 그대로 기록합니다.

---

# 12. DCGAN 결과를 관찰합니다

```text
results/apple/dcgan/epochs/

epoch_001.png
epoch_005.png
epoch_010.png
epoch_015.png
```

다음 질문에 답합니다.

```text
빨간색 특징이 나타나는가?

Apple의 둥근 형태가 나타나는가?

이미지들이 서로 다양한가?

비슷한 결과 몇 종류만 반복되는가?

Epoch가 증가하면서 결과가 항상 좋아지는가?
```

---

## DCGAN Loss도 함께 확인합니다

```bash
tail reports/apple/dcgan_loss.csv
```

`d_loss`와 `g_loss`는 서로 경쟁하는 두 모델의 Loss입니다.

```text
d_loss
→ Discriminator가 Real / Fake를 구분하는 과정의 Loss

g_loss
→ Generator가 Fake를 Real처럼 보이게 만드는 과정의 Loss
```

GAN에서는 Loss가 단순히 작을수록 무조건 좋은 모델이라고 판단하지 않습니다.

```text
Loss 변화
+
Epoch별 생성 이미지
+
다양성
+
실패 패턴
```

을 함께 봅니다.

---

# 13. Mode Collapse를 이해합니다

```text
서로 다른 Noise
        ↓
거의 같은 이미지가 반복
        ↓
다양성이 부족
        ↓
Mode Collapse 가능성
```

생성 이미지가 그럴듯해 보여도 다양성이 너무 낮으면 데이터 보강 목적에는 문제가 될 수 있습니다.

---

# PART 3. WGAN-GP로 학습 안정성을 비교합니다

---

# 14. WGAN-GP가 무엇을 바꾸는지 이해합니다

WGAN-GP에서는 DCGAN의 Discriminator 대신 **Critic**을 사용하고, 학습을 안정시키기 위한 **Gradient Penalty**를 추가합니다.

```text
DCGAN
Real / Fake 구분
→ Discriminator

WGAN-GP
Real / Fake에 점수
→ Critic
→ Gradient Penalty
```

오늘은 Wasserstein Distance나 Gradient Penalty의 수식을 유도하지 않습니다. 코드에서 `Critic`과 `Gradient Penalty`가 어디에 사용되는지 확인하면 충분합니다.

> 짧은 교육용 실험에서 WGAN-GP가 반드시 DCGAN보다 더 좋은 이미지를 만든다고 단정하지 않습니다. 실제 실행 결과를 비교합니다.

---

# 15. WGAN-GP 학습 코드를 작성합니다

**파일: `scripts/02_train_wgan_gp.py`**

```python
from __future__ import annotations

import argparse
import csv
import random
import shutil
import sys
from pathlib import Path

import numpy as np
import pandas as pd
import torch
import torch.nn as nn

from PIL import Image
from torch.utils.data import DataLoader, Dataset
from torchvision import transforms
from torchvision.utils import save_image


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day02_common import day02_paths, list_images


IMAGE_SIZE = 64
CHANNELS = 3
NOISE_DIM = 100
BATCH_SIZE = 32
LAMBDA_GP = 10.0
N_CRITIC = 5
SEED = 42
VISUAL_SEED = 2026


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
        default=15,
    )

    return parser.parse_args()


def set_seed(seed: int) -> None:
    random.seed(seed)
    np.random.seed(seed)
    torch.manual_seed(seed)

    if torch.cuda.is_available():
        torch.cuda.manual_seed_all(seed)


class FlatImageDataset(Dataset):
    def __init__(self, root_dir: Path):
        self.files = list_images(root_dir)

        if not self.files:
            raise RuntimeError(
                f"No images found: {root_dir}"
            )

        self.transform = transforms.Compose(
            [
                transforms.Resize(
                    (IMAGE_SIZE, IMAGE_SIZE)
                ),
                transforms.ToTensor(),
                transforms.Normalize(
                    (0.5, 0.5, 0.5),
                    (0.5, 0.5, 0.5),
                ),
            ]
        )

    def __len__(self):
        return len(self.files)

    def __getitem__(self, index):
        image = Image.open(
            self.files[index]
        ).convert("RGB")

        return self.transform(image)


class Generator(nn.Module):
    def __init__(self):
        super().__init__()

        self.model = nn.Sequential(
            nn.ConvTranspose2d(
                NOISE_DIM, 512, 4, 1, 0, bias=False
            ),
            nn.BatchNorm2d(512),
            nn.ReLU(True),

            nn.ConvTranspose2d(
                512, 256, 4, 2, 1, bias=False
            ),
            nn.BatchNorm2d(256),
            nn.ReLU(True),

            nn.ConvTranspose2d(
                256, 128, 4, 2, 1, bias=False
            ),
            nn.BatchNorm2d(128),
            nn.ReLU(True),

            nn.ConvTranspose2d(
                128, 64, 4, 2, 1, bias=False
            ),
            nn.BatchNorm2d(64),
            nn.ReLU(True),

            nn.ConvTranspose2d(
                64, CHANNELS, 4, 2, 1, bias=False
            ),
            nn.Tanh(),
        )

    def forward(self, noise):
        return self.model(noise)


class Critic(nn.Module):
    def __init__(self):
        super().__init__()

        self.model = nn.Sequential(
            nn.Conv2d(
                CHANNELS, 64, 4, 2, 1
            ),
            nn.LeakyReLU(0.2, inplace=True),

            nn.Conv2d(
                64, 128, 4, 2, 1
            ),
            nn.LeakyReLU(0.2, inplace=True),

            nn.Conv2d(
                128, 256, 4, 2, 1
            ),
            nn.LeakyReLU(0.2, inplace=True),

            nn.Conv2d(
                256, 512, 4, 2, 1
            ),
            nn.LeakyReLU(0.2, inplace=True),

            nn.Conv2d(
                512, 1, 4, 1, 0
            ),
        )

    def forward(self, image):
        return self.model(image).view(-1)


def gradient_penalty(
    critic,
    real,
    fake,
    device,
):
    batch_size = real.size(0)

    alpha = torch.rand(
        batch_size,
        1,
        1,
        1,
        device=device,
    )

    interpolated = (
        alpha * real
        + (1 - alpha) * fake
    )

    interpolated.requires_grad_(True)

    scores = critic(interpolated)

    gradients = torch.autograd.grad(
        outputs=scores,
        inputs=interpolated,
        grad_outputs=torch.ones_like(scores),
        create_graph=True,
        retain_graph=True,
        only_inputs=True,
    )[0]

    gradients = gradients.view(
        batch_size,
        -1,
    )

    penalty = (
        (
            gradients.norm(
                2,
                dim=1,
            )
            - 1
        )
        ** 2
    ).mean()

    return penalty


def reset_directory(directory: Path) -> None:
    if directory.exists():
        shutil.rmtree(directory)

    directory.mkdir(
        parents=True,
        exist_ok=True,
    )


def make_noise(
    count: int,
    device,
    seed: int,
):
    noise_generator = torch.Generator(
        device=device
    ).manual_seed(seed)

    return torch.randn(
        count,
        NOISE_DIM,
        1,
        1,
        generator=noise_generator,
        device=device,
    )


def save_final_samples(
    generator,
    output_dir,
    device,
) -> None:
    generator.eval()

    final_noise = make_noise(
        16,
        device,
        VISUAL_SEED + 1,
    )

    with torch.no_grad():
        samples = generator(final_noise)
        samples = (samples + 1) / 2

    for index, image in enumerate(
        samples,
        start=1,
    ):
        save_image(
            image,
            output_dir / f"sample_{index:02d}.png",
        )


def main() -> None:
    args = parse_args()

    if args.epochs < 1:
        raise ValueError(
            "--epochs는 1 이상이어야 합니다."
        )

    set_seed(SEED)
    paths = day02_paths(args.dataset)

    reset_directory(
        paths["wgan_epochs"]
    )

    reset_directory(
        paths["wgan_final"]
    )

    device = torch.device(
        "cuda"
        if torch.cuda.is_available()
        else "cpu"
    )

    print("Device :", device)
    print("Dataset:", args.dataset)

    dataset = FlatImageDataset(
        paths["train"]
    )

    loader_generator = torch.Generator()
    loader_generator.manual_seed(SEED)

    loader = DataLoader(
        dataset,
        batch_size=BATCH_SIZE,
        shuffle=True,
        generator=loader_generator,
        drop_last=False,
    )

    generator = Generator().to(device)
    critic = Critic().to(device)

    optimizer_g = torch.optim.Adam(
        generator.parameters(),
        lr=1e-4,
        betas=(0.0, 0.9),
    )

    optimizer_c = torch.optim.Adam(
        critic.parameters(),
        lr=1e-4,
        betas=(0.0, 0.9),
    )

    fixed_noise = make_noise(
        64,
        device,
        VISUAL_SEED,
    )

    history = []

    for epoch in range(
        1,
        args.epochs + 1,
    ):
        generator.train()
        critic.train()

        epoch_c = 0.0
        epoch_g = 0.0
        critic_update_count = 0
        batch_counter = 0

        for real_images in loader:
            real_images = real_images.to(
                device
            )

            batch_size = real_images.size(
                0
            )

            for _ in range(N_CRITIC):
                noise = torch.randn(
                    batch_size,
                    NOISE_DIM,
                    1,
                    1,
                    device=device,
                )

                fake_images = generator(
                    noise
                )

                critic_real = critic(
                    real_images
                ).mean()

                critic_fake = critic(
                    fake_images.detach()
                ).mean()

                gp = gradient_penalty(
                    critic,
                    real_images,
                    fake_images.detach(),
                    device,
                )

                loss_c = (
                    critic_fake
                    - critic_real
                    + LAMBDA_GP * gp
                )

                optimizer_c.zero_grad()
                loss_c.backward()
                optimizer_c.step()

                epoch_c += loss_c.item()
                critic_update_count += 1

            noise = torch.randn(
                batch_size,
                NOISE_DIM,
                1,
                1,
                device=device,
            )

            fake_images = generator(
                noise
            )

            loss_g = -critic(
                fake_images
            ).mean()

            optimizer_g.zero_grad()
            loss_g.backward()
            optimizer_g.step()

            epoch_g += loss_g.item()
            batch_counter += 1

        avg_c = (
            epoch_c
            / critic_update_count
        )

        avg_g = (
            epoch_g
            / batch_counter
        )

        history.append(
            {
                "epoch": epoch,
                "critic_loss": avg_c,
                "generator_loss": avg_g,
            }
        )

        print(
            f"Epoch {epoch:02d}/"
            f"{args.epochs} "
            f"C={avg_c:.4f} "
            f"G={avg_g:.4f}"
        )

        generator.eval()

        with torch.no_grad():
            samples = generator(
                fixed_noise
            )

            samples = (
                samples + 1
            ) / 2

            save_image(
                samples,
                paths["wgan_epochs"]
                / f"epoch_{epoch:03d}.png",
                nrow=8,
            )

    model_g = (
        paths["model"]
        / "wgan_gp_generator.pt"
    )

    model_c = (
        paths["model"]
        / "wgan_gp_critic.pt"
    )

    torch.save(
        generator.state_dict(),
        model_g,
    )

    torch.save(
        critic.state_dict(),
        model_c,
    )

    loss_path = (
        paths["report"]
        / "wgan_gp_loss.csv"
    )

    pd.DataFrame(
        history
    ).to_csv(
        loss_path,
        index=False,
        encoding="utf-8-sig",
    )

    save_final_samples(
        generator,
        paths["wgan_final"],
        device,
    )

    metadata_path = (
        paths["report"]
        / "wgan_gp_metadata.csv"
    )

    with metadata_path.open(
        "w",
        newline="",
        encoding="utf-8-sig",
    ) as file:
        writer = csv.DictWriter(
            file,
            fieldnames=[
                "method",
                "dataset",
                "epochs",
                "batch_size",
                "noise_dim",
                "seed",
                "epoch_visual_seed",
                "final_sample_seed",
                "learning_rate",
                "n_critic",
                "lambda_gp",
                "model_generator",
                "model_critic",
            ],
        )

        writer.writeheader()

        writer.writerow(
            {
                "method": "wgan_gp",
                "dataset": args.dataset,
                "epochs": args.epochs,
                "batch_size": BATCH_SIZE,
                "noise_dim": NOISE_DIM,
                "seed": SEED,
                "epoch_visual_seed": VISUAL_SEED,
                "final_sample_seed": VISUAL_SEED + 1,
                "learning_rate": 1e-4,
                "n_critic": N_CRITIC,
                "lambda_gp": LAMBDA_GP,
                "model_generator": str(
                    model_g
                ),
                "model_critic": str(
                    model_c
                ),
            }
        )

    print(
        "WGAN-GP Training Completed"
    )

    print("Loss    :", loss_path)
    print(
        "Samples :",
        paths["wgan_final"],
    )


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

같은 Apple Train을 사용하면서 DCGAN과 다른 학습 방식으로 생성모델을 만들고, 생성 결과와 학습 안정성이 실제로 어떻게 달라지는지 비교하기 위해 작성합니다.

### 의사코드로 읽어보기

```text
Apple Train을 준비한다
        ↓
Generator와 Critic을 만든다
        ↓
Noise로 Fake Image를 만든다
        ↓
Real과 Fake를 Critic에 넣어 점수를 계산한다
        ↓
Real과 Fake 중간 이미지를 만든다
        ↓
Gradient Penalty를 계산한다
        ↓
Critic을 여러 번 학습한다
        ↓
Generator를 한 번 학습한다
        ↓
Epoch마다 같은 fixed_noise로 결과를 저장한다
        ↓
모델·Loss·Final Sample·Metadata를 저장한다
```

### 코드에서 꼭 확인할 세 부분

WGAN-GP의 Critic 마지막에는 `Sigmoid()`가 없습니다. Real/Fake 확률을 만드는 대신 **제한되지 않은 점수**를 출력하기 때문입니다.

`N_CRITIC = 5`는 Generator를 한 번 업데이트하기 전에 Critic을 여러 번 업데이트한다는 뜻입니다. 오늘은 학습 흐름을 확인하기 위한 기준값으로 사용합니다.

`gradient_penalty()`는 Real과 Fake 사이의 중간 이미지를 만들고 그 지점의 Gradient 크기를 확인합니다. 이 Penalty가 Critic Loss에 더해져 학습이 지나치게 불안정해지는 것을 줄이는 역할을 합니다.

---

# 16. WGAN-GP를 실행합니다

먼저 Smoke Test를 합니다.

```bash
python scripts/02_train_wgan_gp.py \
  --dataset apple \
  --epochs 1
```

정상이라면 본 실습을 실행합니다.

```bash
python scripts/02_train_wgan_gp.py \
  --dataset apple \
  --epochs 15
```

> Smoke Test와 본 실습은 각각 새 학습으로 시작합니다. 본 실습을 실행하면 `results/apple/wgan_gp/`의 Smoke Test 결과는 비우고 본 학습 결과를 다시 저장합니다.
>
> GPU 상황 때문에 Epoch를 줄였다면 실제 사용한 값을 `wgan_gp_metadata.csv`와 `day02_notes.md`에 그대로 남깁니다.

WGAN-GP는 Critic을 여러 번 학습하므로 DCGAN보다 오래 걸릴 수 있습니다.

---

## WGAN-GP Loss를 확인할 때 주의합니다

```bash
tail reports/apple/wgan_gp_loss.csv
```

WGAN-GP의 `critic_loss`와 `generator_loss`는 DCGAN의 BCE Loss와 정의가 다릅니다.

따라서 다음처럼 숫자 크기를 직접 비교하지 않습니다.

```text
DCGAN g_loss = 2.0
WGAN-GP generator_loss = 0.5

→ WGAN-GP가 더 좋다
X
```

각 모델 안에서 Loss가 어떻게 변하는지 보고, 최종 판단은 생성 이미지와 함께 합니다.

---

# 17. DCGAN과 WGAN-GP 결과를 한 화면에서 비교하는 코드를 작성합니다

**파일: `scripts/03_compare_gan_results.py`**

```python
from __future__ import annotations

import argparse
import sys
from pathlib import Path

from PIL import Image, ImageDraw


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day02_common import (
    day02_paths,
    list_images,
)


TILE_SIZE = 128
GRID_COLUMNS = 4


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--dataset",
        default="apple",
        choices=["apple", "banana"],
    )

    return parser.parse_args()


def make_grid(
    image_paths,
    title,
):
    selected = image_paths[:16]

    if not selected:
        raise RuntimeError(
            f"{title} 이미지가 없습니다."
        )

    rows = (
        len(selected)
        + GRID_COLUMNS - 1
    ) // GRID_COLUMNS

    header = 40

    canvas = Image.new(
        "RGB",
        (
            TILE_SIZE * GRID_COLUMNS,
            header + TILE_SIZE * rows,
        ),
        "white",
    )

    draw = ImageDraw.Draw(canvas)

    draw.text(
        (10, 12),
        title,
        fill="black",
    )

    for index, path in enumerate(
        selected
    ):
        image = Image.open(
            path
        ).convert("RGB")

        image = image.resize(
            (TILE_SIZE, TILE_SIZE)
        )

        column = index % GRID_COLUMNS
        row = index // GRID_COLUMNS

        x = column * TILE_SIZE
        y = header + row * TILE_SIZE

        canvas.paste(
            image,
            (x, y),
        )

    return canvas


def main() -> None:
    args = parse_args()
    paths = day02_paths(args.dataset)

    dcgan_files = list_images(
        paths["dcgan_final"]
    )

    wgan_files = list_images(
        paths["wgan_final"]
    )

    dcgan_grid = make_grid(
        dcgan_files,
        "DCGAN Final Samples",
    )

    wgan_grid = make_grid(
        wgan_files,
        "WGAN-GP Final Samples",
    )

    width = (
        dcgan_grid.width
        + wgan_grid.width
    )

    height = max(
        dcgan_grid.height,
        wgan_grid.height,
    )

    canvas = Image.new(
        "RGB",
        (width, height),
        "white",
    )

    canvas.paste(
        dcgan_grid,
        (0, 0),
    )

    canvas.paste(
        wgan_grid,
        (dcgan_grid.width, 0),
    )

    output_path = (
        paths["compare"]
        / "dcgan_vs_wgan_gp.jpg"
    )

    canvas.save(output_path)

    print(
        "Saved:",
        output_path,
    )


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

두 모델의 최종 이미지를 한 화면에서 비교하면 형태·색·다양성·반복 패턴을 더 쉽게 확인할 수 있습니다.

### 의사코드로 읽어보기

```text
DCGAN Final Image를 최대 16장 읽는다
        ↓
4열 Grid를 만든다
        ↓
WGAN-GP Final Image를 최대 16장 읽는다
        ↓
같은 크기의 Grid를 만든다
        ↓
두 Grid를 좌우에 배치한다
        ↓
비교 이미지를 저장한다
```

실행합니다.

```bash
python scripts/03_compare_gan_results.py \
  --dataset apple
```

---

# 18. DCGAN과 WGAN-GP 결과를 해석합니다

| 비교 항목 | DCGAN | WGAN-GP |
|---|---|---|
| 빨간색 특징 | | |
| Apple 형태 | | |
| 이미지 다양성 | | |
| 비슷한 결과 반복 | | |
| 심한 Artifact | | |
| 가장 안정적으로 보인 Epoch | | |

실행 후에는 자신의 결과를 근거로 기록합니다. 아래 문장은 **작성 형식의 예시일 뿐 실제 측정 결과가 아닙니다.**

```text
현재 15 Epoch 실험에서는
WGAN-GP가 형태를 조금 더 안정적으로 유지했지만
두 방식 모두 충분히 Real Apple처럼 보이지 않았다.
```

또는:

```text
이번 실험에서는
두 방식의 품질 차이가 크지 않았다.
```

실제 결과가 다르면 관찰한 내용 그대로 기록합니다.

---

# PART 4. Pretrained Diffusion으로 Apple 이미지를 생성합니다

---

# 19. Diffusion은 오늘 처음부터 학습하지 않습니다

오늘은 대규모 Diffusion Model을 새로 학습하지 않고 **준비된 Pretrained Model**을 사용합니다.

```text
Noise
        ↓
Prompt 조건
        ↓
Denoising
        ↓
새로운 이미지
```

직접 비교할 조건은 `Prompt`, `Seed`, `Inference Steps`, `Guidance`입니다. 이 값들의 복잡한 수학적 의미보다 **어떤 조건을 바꾸었을 때 결과가 어떻게 달라지는지**에 집중합니다.

---

# 20. Prompt와 Seed의 역할을 구분합니다

```text
Prompt
→ 어떤 이미지를 만들지 설명

Seed
→ Random Noise의 시작값
→ 같은 주요 조건을 다시 재현할 때 사용
```

오늘은 두 실험을 분리합니다.

```text
실험 A
Prompt 고정
Seed 변경

실험 B
Seed 고정
Prompt 변경
```

---

# 21. Diffusion 생성 코드를 작성합니다

**파일: `scripts/04_diffusion_generate.py`**

```python
from __future__ import annotations

import argparse
import csv
import os
import sys
from pathlib import Path

import torch

from diffusers import (
    AutoPipelineForText2Image,
)


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day02_common import day02_paths


IMAGE_WIDTH = 512
IMAGE_HEIGHT = 512


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--dataset",
        default="apple",
        choices=["apple", "banana"],
    )

    parser.add_argument(
        "--prompt",
        required=True,
    )

    parser.add_argument(
        "--seed",
        type=int,
        default=42,
    )

    parser.add_argument(
        "--steps",
        type=int,
        default=4,
    )

    parser.add_argument(
        "--guidance",
        type=float,
        default=0.0,
    )

    parser.add_argument(
        "--count",
        type=int,
        default=1,
    )

    parser.add_argument(
        "--tag",
        default="sample",
    )

    return parser.parse_args()


def load_pipeline(
    model_id,
    device,
):
    dtype = (
        torch.float16
        if device == "cuda"
        else torch.float32
    )

    pipeline = (
        AutoPipelineForText2Image
        .from_pretrained(
            model_id,
            torch_dtype=dtype,
        )
    )

    return pipeline.to(device)


def main() -> None:
    args = parse_args()

    if args.steps < 1:
        raise ValueError(
            "--steps는 1 이상이어야 합니다."
        )

    if args.count < 1:
        raise ValueError(
            "--count는 1 이상이어야 합니다."
        )

    model_id = os.environ.get(
        "DIFFUSION_MODEL_ID"
    )

    if not model_id:
        raise RuntimeError(
            "DIFFUSION_MODEL_ID "
            "환경변수를 먼저 설정하세요."
        )

    paths = day02_paths(args.dataset)

    device = (
        "cuda"
        if torch.cuda.is_available()
        else "cpu"
    )

    print("Device :", device)
    print("Model  :", model_id)

    if device == "cpu":
        print(
            "[WARNING] CPU에서는 "
            "Diffusion 생성이 오래 걸릴 수 있습니다."
        )

    pipeline = load_pipeline(
        model_id,
        device,
    )

    metadata_path = (
        paths["report"]
        / "diffusion_metadata.csv"
    )

    rows = []

    for index in range(args.count):
        current_seed = (
            args.seed + index
        )

        generator = (
            torch.Generator(
                device=device
            )
            .manual_seed(
                current_seed
            )
        )

        result = pipeline(
            prompt=args.prompt,
            num_inference_steps=(
                args.steps
            ),
            guidance_scale=(
                args.guidance
            ),
            width=IMAGE_WIDTH,
            height=IMAGE_HEIGHT,
            generator=generator,
        )

        image = result.images[0]

        file_name = (
            f"{args.tag}"
            f"_seed{current_seed}"
            f"_step{args.steps}.png"
        )

        output_path = (
            paths["diffusion"]
            / file_name
        )

        image.save(output_path)

        rows.append(
            {
                "file": file_name,
                "method": "diffusion",
                "dataset": args.dataset,
                "model": model_id,
                "prompt": args.prompt,
                "seed": current_seed,
                "steps": args.steps,
                "guidance": args.guidance,
                "width": IMAGE_WIDTH,
                "height": IMAGE_HEIGHT,
                "approved": "pending",
            }
        )

        print(
            "Saved:",
            output_path,
        )

    fieldnames = [
        "file",
        "method",
        "dataset",
        "model",
        "prompt",
        "seed",
        "steps",
        "guidance",
        "width",
        "height",
        "approved",
    ]

    existing_rows = []

    if (
        metadata_path.exists()
        and metadata_path.stat().st_size > 0
    ):
        with metadata_path.open(
            "r",
            newline="",
            encoding="utf-8-sig",
        ) as file:
            existing_rows = list(
                csv.DictReader(file)
            )

    new_files = {
        row["file"]
        for row in rows
    }

    existing_rows = [
        row
        for row in existing_rows
        if row.get("file") not in new_files
    ]

    with metadata_path.open(
        "w",
        newline="",
        encoding="utf-8-sig",
    ) as file:
        writer = csv.DictWriter(
            file,
            fieldnames=fieldnames,
        )

        writer.writeheader()

        writer.writerows(
            existing_rows + rows
        )

    print(
        "Metadata:",
        metadata_path,
    )


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

사전학습 Diffusion Model에 Prompt와 Seed를 주어 새로운 이미지를 만들고, 어떤 조건으로 만들었는지 Metadata까지 함께 기록하기 위해 작성합니다.

`--dataset`은 Diffusion을 Apple이나 Banana로 다시 학습한다는 뜻이 아니라 **결과 폴더와 비교할 Real 데이터 종류를 구분하는 이름**입니다.

### 의사코드로 읽어보기

```text
Prompt·Seed·Steps·생성 수를 입력받는다
        ↓
DIFFUSION_MODEL_ID를 확인한다
        ↓
Pretrained Model을 불러온다
        ↓
Seed로 Random Generator를 만든다
        ↓
Prompt와 생성 조건으로 이미지를 만든다
        ↓
이미지를 저장한다
        ↓
Model·Prompt·Seed·Steps를 Metadata에 기록한다
```

---

# 22. Diffusion Smoke Test — 기준 Apple Prompt로 한 장 생성합니다

```bash
export DIFFUSION_MODEL_ID="$HOME/ai_vision/models/sd-turbo"
```

먼저 **한 장만 생성하는 Smoke Test**를 실행합니다. 여기서 중요한 것은 품질 평가가 아니라 Model Load → 생성 → 파일 저장 → Metadata 기록이 정상적으로 이어지는지 확인하는 것입니다.

```bash
python scripts/04_diffusion_generate.py \
  --dataset apple \
  --prompt "a single red apple on a clean white background, centered object, realistic product photography, neutral lighting" \
  --seed 42 \
  --steps 4 \
  --guidance 0.0 \
  --count 1 \
  --tag base
```

이미지와 `reports/apple/diffusion_metadata.csv`가 정상적으로 생성되면 Diffusion 실행환경이 준비된 것입니다.

이후 Seed와 Prompt를 바꾸는 본 실험으로 진행합니다.

다음 질문에 답합니다.

```text
한 개의 빨간 사과가 보이는가?

흰 배경이 유지되는가?

Real Fruits-360 Apple과
형태·배경·조명이 비슷한가?
```

---

# 23. Prompt를 고정하고 Seed만 바꿉니다

```bash
python scripts/04_diffusion_generate.py \
  --dataset apple \
  --prompt "a single red apple on a clean white background, centered object, realistic product photography, neutral lighting" \
  --seed 100 \
  --steps 4 \
  --guidance 0.0 \
  --count 4 \
  --tag same_prompt
```

확인합니다.

```text
Prompt는 같은데
사과 모양이 어떻게 달라졌는가?

배경과 조명은 어느 정도 유지되는가?
```

---

# 24. Seed를 고정하고 Prompt만 바꿉니다

## 기준 조명

```bash
python scripts/04_diffusion_generate.py \
  --dataset apple \
  --prompt "a single red apple on a clean white background, centered object, realistic product photography, neutral lighting" \
  --seed 200 \
  --steps 4 \
  --guidance 0.0 \
  --count 1 \
  --tag neutral_light
```

## 어두운 조명

```bash
python scripts/04_diffusion_generate.py \
  --dataset apple \
  --prompt "a single red apple on a clean white background, centered object, realistic product photography, dim lighting" \
  --seed 200 \
  --steps 4 \
  --guidance 0.0 \
  --count 1 \
  --tag dim_light
```

## 강한 조명

```bash
python scripts/04_diffusion_generate.py \
  --dataset apple \
  --prompt "a single red apple on a clean white background, centered object, realistic product photography, strong overhead lighting and reflections" \
  --seed 200 \
  --steps 4 \
  --guidance 0.0 \
  --count 1 \
  --tag strong_light
```

이제 다음을 비교합니다.

```text
Seed는 같음
        ↓
Prompt의 조명 표현만 변경
        ↓
밝기·반사·그림자는
어떻게 달라졌는가?
```

---

# 25. Diffusion Metadata를 확인합니다

```bash
cat reports/apple/diffusion_metadata.csv
```

| 열 | 의미 |
|---|---|
| `file` | 생성된 이미지 |
| `model` | 사용한 Diffusion Model |
| `prompt` | 생성 조건 |
| `seed` | Random 시작값 |
| `steps` | Denoising 반복 조건 |
| `guidance` | Prompt 반영 관련 생성 조건 |
| `width` / `height` | 생성 이미지 크기 |
| `approved` | 검수 전 상태 |

같은 `tag + seed + steps` 조합을 다시 실행하면 같은 파일명이 다시 생성됩니다. 이 경우 코드는 기존 파일을 덮어쓰고 Metadata에서도 해당 파일의 기존 행을 새 조건으로 교체하여 한 파일에 한 기록만 남깁니다.

---

# PART 5. Real Apple과 Synthetic Apple의 Domain Gap을 확인합니다

---

# 26. Domain Gap을 이해합니다

생성 이미지가 보기 좋다고 해서 바로 Real 데이터와 같은 역할을 하는 것은 아닙니다.

예를 들어 Diffusion Apple이 매우 사실적으로 보여도 다음 차이가 있을 수 있습니다.

```text
Real Fruits-360 Apple
→ 일정한 흰 배경
→ 실제 촬영된 사과
→ 특정 카메라·조명 조건

Synthetic Apple
→ 모델이 만들어낸 형태
→ 다른 표면 질감
→ 다른 그림자
→ 다른 배경 특성
```

이처럼 **실제 데이터 분포와 합성 데이터 분포 사이의 차이**를 Domain Gap이라고 합니다.

오늘은 정교한 수치 하나로 Domain Gap을 확정하지 않습니다.

먼저 다음 항목을 사람이 비교합니다.

```text
형태
색
배경
조명
Artifact
다양성
```

---

# 27. Real과 세 가지 Synthetic 결과를 검수표로 만드는 코드를 작성합니다

이 코드는 다음 데이터를 한 CSV에 모읍니다.

```text
Real Apple 10장
DCGAN Final 16장
WGAN-GP Final 16장
Diffusion 생성 이미지
```

**파일: `scripts/05_domain_gap_review.py`**

```python
from __future__ import annotations

import argparse
import random
import sys
from pathlib import Path

import pandas as pd


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day02_common import (
    day02_paths,
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
        "--overwrite",
        action="store_true",
        help=(
            "기존 domain_gap_review.csv를 "
            "의도적으로 다시 만들 때 사용합니다."
        ),
    )

    return parser.parse_args()


def add_rows(
    rows,
    files,
    method,
):
    for path in files:
        rows.append(
            {
                "file": str(path),
                "method": method,
                "realistic_shape": "",
                "realistic_color": "",
                "realistic_background": "",
                "realistic_light": "",
                "artifact": "",
                "diversity": "",
                "domain_gap_level": "",
                "decision": "",
                "note": "",
            }
        )


def main() -> None:
    args = parse_args()
    paths = day02_paths(
        args.dataset
    )

    rows = []

    train_files = list_images(
        paths["train"]
    )

    if not train_files:
        raise RuntimeError(
            "Real Train 이미지가 없습니다."
        )

    rng = random.Random(SEED)

    real_count = min(
        10,
        len(train_files),
    )

    real_files = rng.sample(
        train_files,
        real_count,
    )

    dcgan_files = list_images(
        paths["dcgan_final"]
    )

    wgan_files = list_images(
        paths["wgan_final"]
    )

    diffusion_files = list_images(
        paths["diffusion"]
    )

    if not dcgan_files:
        raise RuntimeError(
            "DCGAN Final 이미지가 없습니다."
        )

    if not wgan_files:
        raise RuntimeError(
            "WGAN-GP Final 이미지가 없습니다."
        )

    if not diffusion_files:
        raise RuntimeError(
            "Diffusion 이미지가 없습니다."
        )

    add_rows(
        rows,
        real_files,
        "real_reference",
    )

    add_rows(
        rows,
        dcgan_files,
        "dcgan",
    )

    add_rows(
        rows,
        wgan_files,
        "wgan_gp",
    )

    add_rows(
        rows,
        diffusion_files,
        "diffusion",
    )

    output_path = (
        paths["report"]
        / "domain_gap_review.csv"
    )

    if (
        output_path.exists()
        and not args.overwrite
    ):
        raise RuntimeError(
            "domain_gap_review.csv가 이미 있습니다. "
            "작성한 검수 내용을 보호하기 위해 덮어쓰지 않습니다. "
            "정말 새로 만들려면 --overwrite를 사용하세요."
        )

    pd.DataFrame(
        rows
    ).to_csv(
        output_path,
        index=False,
        encoding="utf-8-sig",
    )

    print(
        "Saved:",
        output_path,
    )

    print(
        "Real Reference:",
        len(real_files),
    )

    print(
        "DCGAN:",
        len(dcgan_files),
    )

    print(
        "WGAN-GP:",
        len(wgan_files),
    )

    print(
        "Diffusion:",
        len(diffusion_files),
    )


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

Real과 Synthetic 결과를 같은 기준으로 검수하기 위해 작성합니다.

생성 방법마다 확인 기준이 달라지면 비교가 어려우므로 한 표에서 같은 항목을 기록합니다.

### 의사코드로 읽어보기

```text
1일차 Real Train에서
Seed 42로 대표 이미지 최대 10장을 고른다
        ↓
DCGAN Final Image를 가져온다
        ↓
WGAN-GP Final Image를 가져온다
        ↓
Diffusion Image를 가져온다
        ↓
모든 이미지에 같은 검수 항목을 만든다
        ↓
형태·색·배경·조명·Artifact·다양성을
기록할 수 있는 CSV로 저장한다
```

DCGAN·WGAN-GP·Diffusion 생성이 모두 끝난 뒤 실행합니다.

```bash
python scripts/05_domain_gap_review.py \
  --dataset apple
```

한 번 생성한 뒤 사람이 검수 내용을 작성하면 같은 명령으로는 덮어쓰지 않습니다.

```text
기존 domain_gap_review.csv 있음
→ 실행 중단
→ 사람이 작성한 검수 내용 보호
```

실험을 처음부터 다시 하여 검수표 자체를 새로 만들 필요가 있을 때만 다음처럼 의도적으로 실행합니다.

```bash
python scripts/05_domain_gap_review.py \
  --dataset apple \
  --overwrite
```

결과:

```text
reports/apple/domain_gap_review.csv
```

---

# 28. Domain Gap 검수표를 직접 작성합니다

`domain_gap_review.csv`를 열어 다음 기준으로 작성합니다.

```text
realistic_shape
→ high / medium / low

realistic_color
→ high / medium / low

realistic_background
→ high / medium / low

realistic_light
→ high / medium / low

artifact
→ none / small / severe

diversity
→ high / medium / low

domain_gap_level
→ low / medium / high

decision
→ approved / needs_review / rejected
```

이 값은 정답표가 아닙니다.

자신이 실제 결과를 보고 근거를 남기는 것이 목적입니다.

---

# 29. 생성 이미지의 Automatic QA를 수행하는 코드를 작성합니다

이번 Automatic QA에서는 다음 항목을 확인합니다.

```text
파일이 실제로 존재하는가?

OpenCV로 읽을 수 있는가?

Width·Height가 정상인가?

밝기 평균과 밝기 표준편차는 얼마인가?

거의 한 가지 밝기만 가진
비정상적으로 균일한 이미지인가?

같은 파일 내용이
완전히 중복되어 있지는 않은가?
```

평균 밝기 자체에는 정답 Threshold를 두지 않습니다. 흰 배경이 많은 Apple·Banana 이미지는 정상이어도 평균 밝기가 높을 수 있기 때문입니다.

대신 밝기 표준편차가 지나치게 작은 이미지를 `near_uniform` 후보로 표시합니다. 이것도 최종 품질 판정이 아니라 **파일 이상을 빠르게 찾기 위한 간단한 규칙**입니다.

또한 사람이 빠르게 확인할 수 있도록 DCGAN·WGAN-GP·Diffusion에서 **Seed 42로 방법별 최대 4장을 고정 Sampling**하여 Grid를 만듭니다. 항상 앞의 몇 장만 보는 것보다 대표 Sample을 일정한 기준으로 다시 확인하기 쉽습니다.

**파일: `scripts/06_synthetic_qa.py`**

```python
from __future__ import annotations

import argparse
import hashlib
import random
import sys
from pathlib import Path

import cv2
import numpy as np
import pandas as pd


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day02_common import (
    day02_paths,
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

    return parser.parse_args()


def sha256_file(path: Path) -> str:
    digest = hashlib.sha256()

    with path.open("rb") as file:
        for chunk in iter(
            lambda: file.read(
                1024 * 1024
            ),
            b"",
        ):
            digest.update(chunk)

    return digest.hexdigest()


def collect_method_files(paths):
    return {
        "dcgan": list_images(
            paths["dcgan_final"]
        ),
        "wgan_gp": list_images(
            paths["wgan_final"]
        ),
        "diffusion": list_images(
            paths["diffusion"]
        ),
    }


def make_human_grid(
    method_files,
    output_path,
):
    selected = []
    rng = random.Random(SEED)

    for method in [
        "dcgan",
        "wgan_gp",
        "diffusion",
    ]:
        files = method_files[
            method
        ]

        sample_count = min(
            4,
            len(files),
        )

        sampled = (
            rng.sample(
                files,
                sample_count,
            )
            if sample_count > 0
            else []
        )

        for path in sampled:
            selected.append(
                (method, path)
            )

    if not selected:
        return

    tile_size = 160
    columns = 4
    header = 24

    tiles = []

    for method, path in selected:
        image = cv2.imread(
            str(path)
        )

        if image is None:
            continue

        image = cv2.resize(
            image,
            (
                tile_size,
                tile_size,
            ),
        )

        canvas = np.full(
            (
                tile_size + header,
                tile_size,
                3,
            ),
            255,
            dtype=np.uint8,
        )

        canvas[
            header:,
            :,
        ] = image

        cv2.putText(
            canvas,
            method,
            (5, 17),
            cv2.FONT_HERSHEY_SIMPLEX,
            0.45,
            (0, 0, 0),
            1,
            cv2.LINE_AA,
        )

        tiles.append(canvas)

    if not tiles:
        return

    while len(tiles) % columns != 0:
        tiles.append(
            np.zeros_like(
                tiles[0]
            )
        )

    rows = []

    for start in range(
        0,
        len(tiles),
        columns,
    ):
        rows.append(
            np.hstack(
                tiles[
                    start:
                    start + columns
                ]
            )
        )

    grid = np.vstack(rows)

    saved = cv2.imwrite(
        str(output_path),
        grid,
    )

    if not saved:
        raise RuntimeError(
            f"QA Grid 저장 실패: {output_path}"
        )


def main() -> None:
    args = parse_args()
    paths = day02_paths(
        args.dataset
    )

    method_files = (
        collect_method_files(
            paths
        )
    )

    all_items = []

    for method, files in (
        method_files.items()
    ):
        for path in files:
            all_items.append(
                (method, path)
            )

    if not all_items:
        raise RuntimeError(
            "QA할 생성 이미지가 없습니다."
        )

    hash_groups = {}

    for method, path in all_items:
        file_hash = sha256_file(
            path
        )

        hash_groups.setdefault(
            file_hash,
            [],
        ).append(
            str(path)
        )

    duplicate_hashes = {
        file_hash
        for file_hash, names
        in hash_groups.items()
        if len(names) > 1
    }

    rows = []

    for method, path in all_items:
        image = cv2.imread(
            str(path)
        )

        readable = (
            image is not None
        )

        width = 0
        height = 0
        mean_brightness = 0.0
        std_brightness = 0.0

        if readable:
            height, width = (
                image.shape[:2]
            )

            gray = cv2.cvtColor(
                image,
                cv2.COLOR_BGR2GRAY,
            )

            mean_brightness = float(
                gray.mean()
            )

            std_brightness = float(
                gray.std()
            )

        file_hash = sha256_file(
            path
        )

        exact_duplicate = (
            file_hash
            in duplicate_hashes
        )

        near_uniform = (
            std_brightness < 2.0
            if readable
            else True
        )

        auto_pass = (
            readable
            and width > 0
            and height > 0
            and not exact_duplicate
            and not near_uniform
        )

        rows.append(
            {
                "file": str(path),
                "method": method,
                "readable": readable,
                "width": width,
                "height": height,
                "mean_brightness": round(
                    mean_brightness,
                    2,
                ),
                "std_brightness": round(
                    std_brightness,
                    2,
                ),
                "near_uniform": near_uniform,
                "exact_duplicate": (
                    exact_duplicate
                ),
                "auto_pass": auto_pass,
            }
        )

    qa_path = (
        paths["report"]
        / "synthetic_qa.csv"
    )

    pd.DataFrame(
        rows
    ).to_csv(
        qa_path,
        index=False,
        encoding="utf-8-sig",
    )

    grid_path = (
        paths["compare"]
        / "human_qa_grid.jpg"
    )

    make_human_grid(
        method_files,
        grid_path,
    )

    passed = sum(
        row["auto_pass"]
        for row in rows
    )

    print(
        "QA total:",
        len(rows),
    )

    print(
        "QA pass :",
        passed,
    )

    print(
        "QA fail :",
        len(rows) - passed,
    )

    print(
        "QA CSV  :",
        qa_path,
    )

    print(
        "QA Grid :",
        grid_path,
    )


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

생성 이미지 수가 많아지면 사람이 하나씩 파일을 열기 전에 프로그램이 먼저 확인할 수 있는 오류를 걸러내는 것이 효율적입니다.

다만 Automatic QA가 통과했다고 “좋은 Apple 데이터”라고 확정되는 것은 아닙니다.

### 의사코드로 읽어보기

```text
DCGAN Final Image를 모은다
        ↓
WGAN-GP Final Image를 모은다
        ↓
Diffusion Image를 모은다
        ↓
각 파일의 Hash를 만든다
        ↓
Exact Duplicate를 확인한다
        ↓
OpenCV로 읽는다
        ↓
Width·Height·평균 밝기·밝기 표준편차를 계산한다
        ↓
거의 균일한 이미지인지 확인한다
        ↓
기본 파일 조건을 통과하면 auto_pass
        ↓
Seed 42로 방법별 최대 4장을 고른다
        ↓
Human QA Grid를 만든다
```

실행합니다.

```bash
python scripts/06_synthetic_qa.py \
  --dataset apple
```

---

# 30. Human QA에서는 이미지의 의미를 확인합니다

`human_qa_grid.jpg`를 확인합니다.

```text
results/apple/compare/
└─ human_qa_grid.jpg
```

Automatic QA와 Human QA의 역할은 다릅니다.

```text
Automatic QA

파일이 열리는가?
크기가 정상인가?
거의 균일한 이미지인가?
Exact Duplicate인가?


Human QA

정말 Apple처럼 보이는가?
형태가 비정상적으로 찌그러졌는가?
배경이 현실적인가?
표면 질감이 이상하지 않은가?
같은 패턴만 반복되는가?
학습에 넣을 의미가 있는가?
```

오늘 Human QA는 다음 세 가지 결론 중 하나로 기록합니다.

```text
approved
→ 현재 목적에서 사용할 후보

needs_review
→ 추가 확인 필요

rejected
→ 현재 목적에는 사용하기 어려움
```

> `approved`는 현업 학습데이터로 최종 확정되었다는 뜻이 아닙니다. 2일차에서는 생성 결과를 비교하고 후보를 분류하는 연습입니다.

---

# 31. 1일차와 2일차의 생성 방식을 한 번에 정리합니다

| 방법 | 입력 | 핵심 |
|---|---|---|
| Augmentation | Real Image | 기존 이미지를 직접 변형 |
| VAE | Real Train | 압축·복원 + Latent Sample |
| DCGAN | Noise + Real Train 학습 | Generator·Discriminator 경쟁 |
| WGAN-GP | Noise + Real Train 학습 | Critic + Gradient Penalty |
| Diffusion | Prompt + Seed + Pretrained Model | 조건을 주어 이미지 생성 |

하나의 기술이 모든 문제의 정답은 아닙니다.

---

# PART 6. 2일차 본 실습 결과를 정리합니다

---

# 32. Apple 본 실습에서 남겨야 할 결과를 확인합니다

```text
results/apple/
├─ dcgan/
│  ├─ epochs/
│  └─ final/
├─ wgan_gp/
│  ├─ epochs/
│  └─ final/
├─ diffusion/
└─ compare/
   ├─ dcgan_vs_wgan_gp.jpg
   └─ human_qa_grid.jpg

models/apple/
├─ dcgan_generator.pt
├─ dcgan_discriminator.pt
├─ wgan_gp_generator.pt
└─ wgan_gp_critic.pt

reports/apple/
├─ dcgan_loss.csv
├─ dcgan_metadata.csv
├─ wgan_gp_loss.csv
├─ wgan_gp_metadata.csv
├─ diffusion_metadata.csv
├─ domain_gap_review.csv
└─ synthetic_qa.csv
```

다음 네 종류의 결과가 남아 있으면 됩니다.

```text
생성 이미지
학습 Loss
생성 Metadata
QA / Domain Gap 기록
```

---

# 33. Apple 본 실습의 결과를 한 번 연결합니다

여기까지의 결과를 다음 흐름으로 다시 확인합니다.

```text
같은 Apple Train
        ↓

DCGAN
→ Noise에서 Apple 생성
        ↓

WGAN-GP
→ Critic + Gradient Penalty
→ Apple 생성
        ↓

DCGAN vs WGAN-GP
→ 형태·색·다양성·반복 패턴 비교
        ↓

Pretrained SD-Turbo
→ Prompt + Seed
→ Apple 생성
        ↓

Real Apple
VS
Synthetic Apple
        ↓
Domain Gap
        ↓
Automatic QA
        ↓
Human QA
```

다음 세 문장을 자신의 실행 결과에 맞게 간단히 적어 둡니다.

```text
DCGAN에서 가장 눈에 띈 특징:

WGAN-GP가 DCGAN과 달랐던 점:

Diffusion과 Real Apple 사이에서
가장 크게 느껴진 Domain Gap:
```

전체 실험 기록은 Mini Challenge가 끝난 뒤 `day02_notes.md`에 한 번만 정리합니다.

---

# PART 7. Mini Challenge — Banana 1로 2일차 생성·비교 흐름을 스스로 반복합니다

---

# 34. Mini Challenge의 목표를 확인합니다

1일차 Mini Challenge에서는 Banana 데이터를 이용하여 다음 과정을 스스로 수행했습니다.

```text
Split
→ Augmentation
→ VAE
→ QA
→ Leakage
```

2일차에는 **같은 Banana Train을 이어서 사용**합니다.

새로운 데이터를 다운로드하거나 촬영하지 않습니다.

```text
1일차 Banana Train
        ↓

2일차 Mini Challenge

DCGAN
        +
WGAN-GP
        ↓
짧은 학습 결과 비교
        ↓
Pretrained Diffusion
        ↓
Banana Prompt / Seed
        ↓
Real Banana
VS
Synthetic Banana
        ↓
Domain Gap
        ↓
QA
        ↓
결과 설명
```

Mini Challenge의 핵심 질문은 다음입니다.

> **Apple에서 배운 생성모델 비교 방법을 Banana에서도 스스로 다시 적용할 수 있는가?**

새로운 Python 코드를 작성하는 문제가 아닙니다.

오늘 만든 스크립트에 `--dataset banana`를 사용하여 전체 흐름을 다시 수행합니다.

---

# 35. Challenge 요구사항 1 — Banana Train을 확인합니다

```bash
python scripts/00_check_day02.py \
  --dataset banana
```

다음 내용을 기록합니다.

```text
Banana Train 경로:

Banana Train 수:

대표 이미지 크기:

CUDA 사용 여부:
```

Train 폴더가 없다는 오류가 나오면 1일차 Banana Split이 완료되지 않은 것입니다.

---

# 36. Challenge 요구사항 2 — Banana DCGAN을 짧게 학습합니다

복습이 목적이므로 **5 Epoch**를 기본으로 합니다.

```bash
python scripts/01_train_dcgan.py \
  --dataset banana \
  --epochs 5
```

다음 질문에 답합니다.

```text
노란색이 나타나는가?

길고 휘어진 형태가 나타나는가?

비슷한 결과만 반복되는가?
```

5 Epoch 결과가 좋지 않아도 실패한 Challenge가 아닙니다. 이 결과는 **생성 방식의 차이를 관찰하기 위한 짧은 교육용 실험**이며, 고품질 생성모델의 성능을 증명하는 결과로 해석하지 않습니다.

---

# 37. Challenge 요구사항 3 — Banana WGAN-GP를 짧게 학습합니다

```bash
python scripts/02_train_wgan_gp.py \
  --dataset banana \
  --epochs 5
```

확인합니다.

```text
DCGAN보다 형태가 더 잘 보이는가?

색이 더 안정적인가?

결과 다양성이 달라졌는가?

짧은 학습에서는
두 방법 모두 불안정한가?
```

---

# 38. Challenge 요구사항 4 — 두 GAN 결과를 비교합니다

```bash
python scripts/03_compare_gan_results.py \
  --dataset banana
```

| 비교 항목 | Banana DCGAN | Banana WGAN-GP |
|---|---|---|
| 노란색 특징 | | |
| 휘어진 형태 | | |
| 다양성 | | |
| 반복 패턴 | | |
| Artifact | | |
| 현재 더 안정적으로 보이는 방법 | | |

결론에는 반드시 **현재 5 Epoch 실험 기준**이라는 표현을 포함합니다. DCGAN이나 WGAN-GP 중 하나가 항상 더 우수하다는 일반적인 결론으로 확대하지 않습니다.

---

# 39. Challenge 요구사항 5 — Diffusion으로 Banana를 생성합니다

Apple 본 실습에서 Pretrained Diffusion Model의 Load와 1장 Smoke Test를 이미 확인했으므로, Mini Challenge에서는 같은 Model을 사용하여 Banana의 Prompt·Seed 변화에 집중합니다.

기준 Prompt:

```text
a single yellow banana on a clean white background,
centered object,
realistic product photography,
neutral lighting
```

같은 Prompt에서 Seed를 바꾸어 4장을 만듭니다.

```bash
python scripts/04_diffusion_generate.py \
  --dataset banana \
  --prompt "a single yellow banana on a clean white background, centered object, realistic product photography, neutral lighting" \
  --seed 300 \
  --steps 4 \
  --guidance 0.0 \
  --count 4 \
  --tag banana_seed
```

Seed 400을 고정하고 Prompt만 바꿉니다.

### 기준 조명

```bash
python scripts/04_diffusion_generate.py \
  --dataset banana \
  --prompt "a single yellow banana on a clean white background, centered object, realistic product photography, neutral lighting" \
  --seed 400 \
  --steps 4 \
  --guidance 0.0 \
  --count 1 \
  --tag banana_neutral
```

### 어두운 조명

```bash
python scripts/04_diffusion_generate.py \
  --dataset banana \
  --prompt "a single yellow banana on a clean white background, centered object, realistic product photography, dim lighting" \
  --seed 400 \
  --steps 4 \
  --guidance 0.0 \
  --count 1 \
  --tag banana_dim
```

다음 질문에 답합니다.

```text
Seed가 달라지면 Banana 모양이 어떻게 달라졌는가?

Prompt의 조명 조건을 바꾸었을 때
실제 결과도 어두워졌는가?

Banana 형태가 비정상적인 이미지는 없는가?
```

---

# 40. Challenge 요구사항 6 — Domain Gap을 확인합니다

```bash
python scripts/05_domain_gap_review.py \
  --dataset banana
```

이미 작성한 Banana 검수표가 있다면 프로그램이 덮어쓰지 않고 중단합니다. 새 검수표가 정말 필요한 경우에만 `--overwrite`를 사용합니다.

결과:

```text
reports/banana/domain_gap_review.csv
```

최소 다음 내용을 기록합니다.

```text
DCGAN Banana
→ Real과 가장 크게 다른 점

WGAN-GP Banana
→ Real과 가장 크게 다른 점

Diffusion Banana
→ Real과 가장 크게 다른 점
```

---

# 41. Challenge 요구사항 7 — Automatic QA와 Human QA를 수행합니다

```bash
python scripts/06_synthetic_qa.py \
  --dataset banana
```

결과:

```text
reports/banana/synthetic_qa.csv

results/banana/compare/
└─ human_qa_grid.jpg
```

Human QA에서는 다음을 확인합니다.

- [ ] Banana의 노란색이 의미 있게 유지되어 있다.
- [ ] 길고 휘어진 형태가 크게 깨지지 않았다.
- [ ] 한 객체가 여러 개로 비정상적으로 분리되지 않았다.
- [ ] 배경이 Real Reference와 지나치게 다르지 않다.
- [ ] 심한 Artifact가 없다.
- [ ] 같은 결과만 반복 생성되지 않는다.
- [ ] `approved / needs_review / rejected` 중 하나로 이유를 설명할 수 있다.

---

# 42. Mini Challenge 결과를 `challenge_notes.md`로 정리합니다

**파일: `reports/banana/challenge_notes.md`**

```markdown
# Day02 Mini Challenge - Banana 1

## 1. 데이터

- 1일차 Banana Train 경로:
- Train 수:
- 새로운 실제 데이터를 추가했는가?:

## 2. DCGAN

- Epoch:
- 노란색 특징:
- Banana 형태:
- 다양성:
- Mode Collapse 의심 여부:
- 대표 실패 사례:

## 3. WGAN-GP

- Epoch:
- 노란색 특징:
- Banana 형태:
- 다양성:
- DCGAN과 달라진 점:
- 대표 실패 사례:

## 4. DCGAN vs WGAN-GP

- 현재 더 안정적으로 보인 방법:
- 그 이유:
- 5 Epoch 결과만으로 일반화할 수 없는 이유:

## 5. Diffusion

- Model:
- 기준 Prompt:
- Seed 변경 결과:
- Prompt 변경 결과:
- 가장 현실적인 결과:
- 가장 비현실적인 결과:

## 6. Domain Gap

- DCGAN:
- WGAN-GP:
- Diffusion:
- Real Banana와 가장 크게 달랐던 부분:

## 7. QA

- Automatic QA:
- Human QA:
- Approved:
- Needs Review:
- Rejected:
- 가장 많이 발견된 Reject 이유:

## 8. Apple 본 실습과 Banana Challenge 비교

- 공통으로 적용할 수 있었던 흐름:
- Apple과 Banana에서 GAN 결과가 달랐던 점:
- Diffusion Prompt에서 바뀐 조건:
- 데이터가 달라도 유지해야 하는 QA 원칙:

## 9. 최종 결론

- GAN과 WGAN-GP의 차이:
- GAN과 Diffusion의 차이:
- 보기 좋은 이미지가 곧 좋은 학습데이터가 아닌 이유:
- 오늘 가장 중요하다고 생각한 생성데이터 관리 원칙:
```

---

# 43. Mini Challenge 완료 조건을 확인합니다

- [ ] 1일차 `Banana 1` Train을 그대로 이어서 사용했다.
- [ ] 새로운 실제 데이터를 별도로 수집하지 않았다.
- [ ] Banana Train 경로와 이미지 수를 확인했다.
- [ ] Banana DCGAN을 5 Epoch 실행했다.
- [ ] Banana WGAN-GP를 5 Epoch 실행했다.
- [ ] DCGAN과 WGAN-GP Final Image를 비교했다.
- [ ] 비교 결과를 “현재 5 Epoch 기준”으로 해석했다.
- [ ] Diffusion으로 Banana 이미지를 생성했다.
- [ ] Prompt를 고정하고 Seed를 변경했다.
- [ ] Seed를 고정하고 Prompt를 변경했다.
- [ ] Diffusion Metadata를 확인했다.
- [ ] Real Banana와 Synthetic Banana의 Domain Gap을 비교했다.
- [ ] Automatic QA를 수행했다.
- [ ] Human QA Grid를 확인했다.
- [ ] `approved / needs_review / rejected`를 근거와 함께 기록했다.
- [ ] `challenge_notes.md`를 작성했다.
- [ ] Apple 본 실습과 Banana Challenge 결과를 비교했다.

---

# 44. Mini Challenge를 약 3시간으로 진행합니다

| 단계 | 권장 시간 |
|---|---:|
| Banana Train 확인·Challenge 계획 | 10분 |
| DCGAN 5 Epoch 실행·결과 확인 | 35분 |
| WGAN-GP 5 Epoch 실행·결과 확인 | 45분 |
| DCGAN vs WGAN-GP 비교 | 15분 |
| Banana Diffusion Seed·Prompt 실험 | 30분 |
| Domain Gap Review | 15분 |
| Automatic QA + Human QA | 15분 |
| `challenge_notes.md` 작성·결론 | 15분 |
| **합계** | **180분** |

GPU 성능에 따라 GAN 학습시간이 달라질 수 있습니다.

WGAN-GP가 예상보다 오래 걸리는 경우에는 Epoch를 줄여도 됩니다.

단, 실제 사용한 Epoch는 반드시 기록합니다.

---

# PART 8. 오늘 만든 결과를 정리하고 버전관리합니다

---

# 45. `day02_notes.md`를 완성합니다

Apple 본 실습과 Banana Mini Challenge를 한 문서에 정리합니다.

```markdown
# Subject 11 - Day 02

## 1. Apple 본 실습

- Train 경로:
- Train 수:

## 2. Apple DCGAN

- Epoch:
- 생성 특징:
- Mode Collapse:
- 실패 사례:

## 3. Apple WGAN-GP

- Epoch:
- 생성 특징:
- DCGAN과 차이:
- 실패 사례:

## 4. Apple Diffusion

- Model:
- 기준 Prompt:
- Seed 변경 결과:
- Prompt 변경 결과:

## 5. Apple Domain Gap / QA

- DCGAN:
- WGAN-GP:
- Diffusion:
- Approved:
- Needs Review:
- Rejected:

## 6. Banana Mini Challenge

- Train 수:
- DCGAN Epoch:
- WGAN-GP Epoch:
- 현재 더 안정적으로 보인 방법:
- Diffusion 결과:
- Domain Gap:
- QA:

## 7. 기술 비교

- Augmentation:
- VAE:
- DCGAN:
- WGAN-GP:
- Diffusion:

## 8. 오늘 발견한 실패 사례

### 실패 1
- 방법:
- 현상:
- 원인 가설:
- 다음에 바꾸어 볼 조건:

### 실패 2
- 방법:
- 현상:
- 원인 가설:
- 다음에 바꾸어 볼 조건:

## 9. 오늘의 결론

- 보기 좋은 이미지와 좋은 학습데이터의 차이:
- Metadata가 필요한 이유:
- Domain Gap을 확인해야 하는 이유:
- 가장 중요하다고 생각한 생성데이터 관리 원칙:
```

---

# 46. README에 2일차 내용을 정리합니다

````markdown
# Subject 11 - Day 02
## GAN · WGAN-GP · Diffusion

### Main Dataset

- Day 01 `Apple Red 1` Train 재사용

```text
../day01_image_aug_vae/data/apple/split/train
```

### Mini Challenge Dataset

- Day 01 `Banana 1` Train 재사용

```text
../day01_image_aug_vae/data/banana/split/train
```

### Day 02 Pipeline

```text
Real Train
    ├─ DCGAN
    │   → Generated Image
    │
    └─ WGAN-GP
        → Generated Image

Pretrained Diffusion
    → Prompt + Seed
    → Generated Image

Real
VS
Synthetic
    → Domain Gap
    → Automatic QA
    → Human QA
    → Metadata
```

### Main Scripts

```text
scripts/
├─ 00_check_day02.py
├─ 01_train_dcgan.py
├─ 02_train_wgan_gp.py
├─ 03_compare_gan_results.py
├─ 04_diffusion_generate.py
├─ 05_domain_gap_review.py
└─ 06_synthetic_qa.py
```

### Main Results

- DCGAN Epoch Samples
- DCGAN Final Samples
- WGAN-GP Epoch Samples
- WGAN-GP Final Samples
- DCGAN vs WGAN-GP
- Diffusion Prompt / Seed Samples
- Domain Gap Review
- Automatic QA
- Human QA
- Experiment Metadata
- Banana Mini Challenge

### Next

Day 03에서는
Background Image와 Object PNG를 합성하면서
Detection 학습에 필요한 BBox까지 자동으로 만듭니다.
````

---

# 47. `.gitignore`를 작성합니다

```gitignore
# Python
__pycache__/
*.pyc

# Model weights
models/
*.pt
*.pth
*.bin
*.safetensors

# Generated images
results/

# Model cache
.cache/
huggingface/

# Editor
.vscode/

# OS
.DS_Store
```

원본 Apple·Banana 데이터와 2일차 생성 이미지는 Git에 올리지 않습니다.

`reports/`의 CSV는 크기가 작고 오늘 실험의 조건·Loss·QA 판단을 다시 확인하는 데 필요하므로 기록으로 남깁니다.

---

# 48. 기존 교과 11 Git 저장소에 2일차를 추가합니다

1일차에서 이미 Git을 시작했으므로 `git init`을 다시 하지 않습니다.

```bash
cd ~/ai_vision/subject11_synthetic_data
```

상태를 확인합니다.

```bash
git status
```

2일차 코드를 추가합니다.

```bash
git add day02_generative_models
```

Commit합니다.

```bash
git commit -m "feat: add subject11 day02 generative model labs"
```

최근 기록을 확인합니다.

```bash
git log --oneline --decorate -5
```

원격 저장소 사용이 허용된 환경이라면:

```bash
git remote -v
git push
```

외부 저장소 사용이 제한된 환경에서는 Local Git 또는 기관 내부 Git을 사용합니다.

---

# 49. 2일차 자가 체크리스트를 확인합니다

- [ ] 1일차 Apple Train을 2일차에서 그대로 재사용할 수 있다.
- [ ] 2일차에서 새로운 실제 Apple 데이터를 다시 수집할 필요가 없다는 것을 이해했다.
- [ ] Validation과 Test를 GAN 학습에 사용하지 않았다.
- [ ] Diffusion에 필요한 라이브러리와 `DIFFUSION_MODEL_ID`를 확인했다.
- [ ] Local Diffusion Model을 사용한다면 `model_index.json`을 확인했다.
- [ ] DCGAN과 WGAN-GP는 1 Epoch Smoke Test 후 본 학습을 실행했다.
- [ ] Diffusion은 1장 Smoke Test 후 Prompt·Seed 본 실험을 진행했다.
- [ ] Augmentation·VAE·GAN·WGAN-GP·Diffusion의 차이를 설명할 수 있다.
- [ ] Generator와 Discriminator의 역할을 설명할 수 있다.
- [ ] Random Noise에서 DCGAN이 이미지를 만드는 흐름을 설명할 수 있다.
- [ ] DCGAN Epoch별 생성 결과를 확인했다.
- [ ] Mode Collapse가 무엇인지 설명할 수 있다.
- [ ] WGAN-GP의 Critic과 Gradient Penalty 역할을 큰 흐름으로 설명할 수 있다.
- [ ] WGAN-GP Epoch별 생성 결과를 확인했다.
- [ ] DCGAN과 WGAN-GP를 같은 데이터에서 비교했다.
- [ ] WGAN-GP가 항상 더 좋다고 단정하면 안 되는 이유를 설명할 수 있다.
- [ ] Pretrained Diffusion Model을 불러올 수 있다.
- [ ] Apple Prompt로 이미지를 생성했다.
- [ ] Prompt를 고정하고 Seed를 변경했다.
- [ ] Seed를 고정하고 Prompt를 변경했다.
- [ ] Diffusion Metadata를 확인했다.
- [ ] Real Apple과 Synthetic Apple의 Domain Gap을 설명할 수 있다.
- [ ] Automatic QA와 Human QA의 차이를 설명할 수 있다.
- [ ] `approved / needs_review / rejected`를 근거와 함께 구분할 수 있다.
- [ ] 1일차 Banana Train을 사용하여 2일차 Mini Challenge를 수행했다.
- [ ] Banana에서 DCGAN과 WGAN-GP를 짧게 다시 비교했다.
- [ ] Banana Diffusion Prompt·Seed 실험을 수행했다.
- [ ] `challenge_notes.md`를 작성했다.
- [ ] README를 정리했다.
- [ ] Git Commit을 완료했다.

---

# 50. 2일차에서 반드시 남겨야 할 세 가지 생각

## ① 학습 데이터와 비교 기준을 구분합니다

```text
같은 Apple Train
        ├─ VAE
        ├─ DCGAN
        └─ WGAN-GP

Pretrained Diffusion
        └─ Prompt + Seed

        ↓

모든 생성 결과를
같은 Real Apple Reference와 비교
```

VAE·DCGAN·WGAN-GP는 Apple Train에서 특징을 학습하지만, 오늘의 Diffusion은 Apple Train으로 다시 학습하지 않습니다. **학습 방식과 비교 기준을 구분해서 해석하는 것**이 중요합니다.

## ② 생성 성공과 학습데이터 승인은 다른 문제입니다

```text
이미지 생성 성공
        ↓
파일이 정상인가?
        ↓
Real과 비슷한가?
        ↓
Artifact는 없는가?
        ↓
다양한가?
        ↓
Domain Gap은 어느 정도인가?
        ↓
QA
        ↓
사용 후보 판단
```

## ③ 생성 조건을 반드시 기록합니다

```text
GAN

Dataset
Epoch
Batch
Noise Dim
Seed
Learning Rate
Model


Diffusion

Model
Prompt
Seed
Steps
Guidance
```

> **같은 결과를 다시 만들거나 실패 원인을 비교하려면 생성 이력이 필요합니다.**

---

# 다음 시간

1일차와 2일차에서는 주로 **이미지 자체를 생성하고 품질을 확인**했습니다.

```text
1일차
Augmentation
VAE

2일차
DCGAN
WGAN-GP
Diffusion
```

3일차에는 질문이 달라집니다.

객체검출 학습데이터에는 이미지뿐 아니라 정답 위치 정보도 필요합니다.

```text
Image
+
Class
+
Bounding Box
```

다음 시간에는 Background와 투명 Object PNG를 합성하면서 **이미지와 BBox를 동시에 만드는 방법**으로 넘어갑니다.

```text
Background
        +
Object PNG
        ↓
Cut-Paste
        ↓
Synthetic Image
        +
자동 계산된 BBox
        ↓
YOLO Label
```

> **2일차에서 반드시 기억할 질문:**  
> 생성 이미지가 보기 좋다는 이유만으로 학습데이터로 사용해도 되는가?  
> 답은 **아니며, Real과 비교하고 Domain Gap과 QA를 확인한 뒤 판단해야 한다**입니다.
