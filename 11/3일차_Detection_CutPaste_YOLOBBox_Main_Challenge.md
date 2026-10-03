## 오늘의 핵심 질문

> **배경 이미지와 투명 Object PNG를 합성할 때, 프로그램이 객체를 붙인 위치를 이용하여 Detection 학습용 이미지와 정답 BBox를 함께 자동 생성할 수 있을까?**

1일차와 2일차에서는 주로 **이미지 자체를 변형하거나 새로 생성하는 방법**을 실습했습니다.

```text
1일차
Apple Red 1
→ Augmentation
→ VAE
→ Reconstruction / Random Sample

2일차
같은 Apple Train
→ DCGAN
→ WGAN-GP

Pretrained Diffusion
→ Prompt + Seed
→ 새로운 이미지
```

3일차에는 여기에서 한 단계 더 나아갑니다.

객체검출 모델이 학습하려면 이미지 한 장만 있어서는 충분하지 않습니다.

```text
Detection 학습데이터

Image
  +
Class
  +
Bounding Box
```

오늘은 수업 전에 준비된 작은 Source Data를 사용합니다.

```text
Background JPG
        +
투명 Object PNG
        ↓
Position / Scale / Rotation
        ↓
Cut-Paste
        ↓
Synthetic Detection Image
        +
Pixel BBox
        +
YOLO Label
        ↓
BBox 재시각화
        ↓
Automatic QA
        ↓
Human QA
        ↓
Metadata
```

본 실습에서는 `main` 데이터를 함께 사용합니다.

하루의 마지막 Mini Challenge에서는 본 실습과 겹치지 않도록 별도로 준비된 `challenge` 데이터를 사용하여 오늘의 전체 흐름을 스스로 다시 수행합니다.

```text
본 실습
main
→ Background 20장
→ Object 3 Class × 5장
→ Cut-Paste + YOLO BBox
→ QA

==============================

Mini Challenge
challenge
→ Background 10장
→ Object 3 Class × 3장
→ 같은 Pipeline을 스스로 반복
→ Rotation 조건 하나만 변경
→ 결과 비교·설명
```

오늘의 목적은 이미지를 많이 만드는 것이 아닙니다.

다음 질문에 답할 수 있도록 **합성 이미지와 정답 Label을 함께 만들고, 다시 검수하고, 생성 조건을 추적하는 전체 과정**을 경험하는 것이 목표입니다.

```text
어떤 Source를 사용했는가?
        ↓
어떤 위치·크기·회전으로 붙였는가?
        ↓
BBox는 정확한가?
        ↓
YOLO Label은 정상인가?
        ↓
합성 장면은 현실적인가?
        ↓
어떤 조건으로 만든 데이터인지 다시 찾을 수 있는가?
```

---

# 오늘의 수업 목표

수업이 끝나면 다음 내용을 설명하고 직접 실행할 수 있어야 합니다.

- 1~2일차와 같은 교과 11 프로젝트와 1일차 `.venv`를 그대로 이어서 사용할 수 있습니다.
- 3일차에 새로운 Source Data가 필요한 이유를 설명할 수 있습니다.
- `main`과 `challenge` 데이터의 역할을 구분할 수 있습니다.
- Background JPG와 투명 RGBA Object PNG의 차이를 설명할 수 있습니다.
- `drill`, `fire_extinguisher`, `wet_floor_sign`의 Class ID를 고정하여 사용할 수 있습니다.
- PNG의 Alpha Channel이 Cut-Paste에서 어떤 역할을 하는지 설명할 수 있습니다.
- 한 개의 객체를 Background에 합성할 수 있습니다.
- 합성 위치를 이용하여 Pixel BBox를 계산할 수 있습니다.
- Pixel BBox를 YOLO 정규화 좌표로 변환할 수 있습니다.
- 위치·크기·회전을 바꾸어 여러 Synthetic Detection Image를 자동 생성할 수 있습니다.
- 생성 이미지와 YOLO TXT를 같은 파일명으로 저장할 수 있습니다.
- 생성 조건을 Metadata CSV에 기록할 수 있습니다.
- YOLO Label을 다시 이미지 위에 그려 BBox를 확인할 수 있습니다.
- 모든 Label의 Class ID·좌표 범위·파일 대응을 Automatic QA로 확인할 수 있습니다.
- Automatic QA와 Human QA의 차이를 설명할 수 있습니다.
- 합성 가능한 데이터와 실제 학습에 사용할 수 있는 데이터가 같지 않다는 점을 설명할 수 있습니다.
- Mini Challenge에서 `challenge` Source Data로 오늘의 전체 흐름을 스스로 다시 수행할 수 있습니다.
- Mini Challenge에서 한 번에 하나의 조건만 변경하고 결과 차이를 설명할 수 있습니다.
- 오늘 만든 코드와 결과를 README와 Git에 정리할 수 있습니다.

---

# 오늘 수업의 전체 흐름

![[Pasted image 20260908145548.png]]

3일차에는 먼저 제공된 **배경 이미지와 투명 객체 PNG가 정상적으로 준비되었는지 확인**한 뒤 실습을 시작합니다.  

실험 A에서는 배경 한 장에 객체 한 개를 직접 붙여 보면서 **Cut-Paste → BBox 계산 → YOLO Label 생성** 과정을 이해하고, 실험 B에서는 같은 원리를 여러 이미지에 자동으로 반복하여 합성 Detection 데이터셋을 만듭니다.  

마지막에는 생성된 BBox와 Label이 올바른지 **Automatic QA와 Human QA로 검수**하고, Mini Challenge에서는 다른 데이터로 같은 과정을 반복하면서 Rotation 조건을 바꾸어 결과 차이를 스스로 분석합니다.

```text
2일차 프로젝트 확인
        ↓
1일차 .venv 재사용
        ↓
3일차 폴더 생성
        ↓
제공된 day03_student_data 배치
        ↓

[본 실습 Source 확인]

main/backgrounds
20장

main/objects
drill              5장
fire_extinguisher  5장
wet_floor_sign     5장
        ↓
classes.txt 확인
        ↓
입력 데이터 자동 검사
        ↓
Asset Preview
        ↓

[실험 A — Single Cut-Paste]

bg_001.jpg
+
drill_001.png
        ↓
Scale
Rotation
Position
        ↓
Cut-Paste
        ↓
Pixel BBox
        ↓
YOLO Label
        ↓
BBox Visualization
        ↓

[실험 B — Batch Dataset]

main Source Data
        ↓
Class 균형 구성
        ↓
Background Random
Object View Random
Position Random
Scale Random
Rotation Random
        ↓
Synthetic Image
+
YOLO Label
        ↓
Metadata
        ↓

[BBox와 QA]

YOLO Label
        ↓
Pixel BBox로 다시 변환
        ↓
이미지 위에 재시각화
        ↓
Automatic QA
        ↓
Human QA
        ↓
Approved / Needs Review / Rejected
        ↓

[Mini Challenge]

challenge/backgrounds
10장

challenge/objects
3 Class × 3장
        ↓
같은 Pipeline 스스로 반복
        ↓
Rotation 15° 기준 실험
        ↓
Rotation 30° 비교 실험
        ↓
한 변수 실험 결과 해석
        ↓

README · Git 정리
        ↓
자가 체크
```

> **오늘의 데이터 운영 원칙**
>
> `main`과 `challenge`는 이미 수업 전에 서로 다른 Source Image로 분리되어 있습니다.
>
> 본 실습 중에는 `main`만 사용합니다.
>
> Mini Challenge에서는 `challenge`만 사용합니다.
>
> 같은 `bg_001.jpg` 또는 `drill_001.png`라는 파일명이 있어도 `main`과 `challenge`는 서로 다른 폴더의 다른 Source입니다.
>
> 오늘 생성한 Synthetic Image는 자동으로 Train 승인하지 않습니다. BBox 재시각화와 QA를 거친 뒤 사용 여부를 판단합니다.

---

# 오늘 사용할 실제 데이터

## 1. 제공되는 3일차 학생용 Source Data

오늘은 새로운 이미지를 직접 촬영하거나 웹에서 다시 다운로드하지 않습니다.

수업 시작 전에 제공된 `day03_student_data`를 사용합니다.

```text
day03_student_data/
│
├─ main/
│  ├─ backgrounds/
│  │  ├─ bg_001.jpg
│  │  ├─ bg_002.jpg
│  │  ├─ ...
│  │  └─ bg_020.jpg
│  │
│  └─ objects/
│     ├─ drill/
│     │  ├─ drill_001.png
│     │  ├─ ...
│     │  └─ drill_005.png
│     │
│     ├─ fire_extinguisher/
│     │  ├─ fire_extinguisher_001.png
│     │  ├─ ...
│     │  └─ fire_extinguisher_005.png
│     │
│     └─ wet_floor_sign/
│        ├─ wet_floor_sign_001.png
│        ├─ ...
│        └─ wet_floor_sign_005.png
│
├─ challenge/
│  ├─ backgrounds/
│  │  ├─ bg_001.jpg
│  │  ├─ ...
│  │  └─ bg_010.jpg
│  │
│  └─ objects/
│     ├─ drill/
│     │  ├─ drill_001.png
│     │  ├─ drill_002.png
│     │  └─ drill_003.png
│     │
│     ├─ fire_extinguisher/
│     │  ├─ fire_extinguisher_001.png
│     │  ├─ fire_extinguisher_002.png
│     │  └─ fire_extinguisher_003.png
│     │
│     └─ wet_floor_sign/
│        ├─ wet_floor_sign_001.png
│        ├─ wet_floor_sign_002.png
│        └─ wet_floor_sign_003.png
│
└─ classes.txt
```

## 2. 본 실습 `main`

```text
Background
20장

Object
drill               5장
fire_extinguisher   5장
wet_floor_sign      5장

총 Object
15장
```

Object PNG는 객체 주변이 투명하도록 미리 정리되어 있습니다.

오늘은 Source Image와 원본 Mask를 다시 전처리하지 않습니다.

학생이 사용하는 단계에서는 이미 다음 상태입니다.

```text
RGB Object
+
Alpha Channel
        ↓
RGBA PNG
        ↓
Cut-Paste에 바로 사용
```

## 3. Mini Challenge `challenge`

```text
Background
10장

Object
drill               3장
fire_extinguisher   3장
wet_floor_sign      3장

총 Object
9장
```

`challenge` 데이터는 본 실습과 다른 Background와 다른 Object View를 사용합니다.

따라서 코드는 같아도 실제 입력은 달라집니다.

## 4. Class 순서는 고정합니다

`classes.txt`의 내용은 다음과 같습니다.

```text
drill
fire_extinguisher
wet_floor_sign
```

YOLO Class ID는 줄 순서로 결정합니다.

```text
0 → drill
1 → fire_extinguisher
2 → wet_floor_sign
```

오늘은 중간에 이 순서를 바꾸지 않습니다.

---

# 오늘 배울 기술 스택

| 기술 | 오늘 하는 일 | 쉽게 말하면 |
|---|---|---|
| Python | Cut-Paste·BBox·QA 코드 작성 | 오늘 실습 전체 실행 |
| pathlib | Source·결과 폴더 관리 | 경로를 안전하게 연결 |
| Pillow | RGBA Object 합성 | 투명 객체를 Background에 붙임 |
| Alpha Channel | 투명 영역 표현 | 객체만 보이게 합성 |
| OpenCV | BBox 재시각화·파일 검사 | Label을 다시 이미지에 그림 |
| Pandas | Metadata·QA CSV 처리 | 생성 조건과 검수 결과 기록 |
| Cut-Paste | 2D 합성데이터 생성 | Background에 객체 배치 |
| Position Randomization | 위치 변경 | 객체가 한 위치에만 몰리지 않게 함 |
| Scale Randomization | 크기 변경 | 여러 크기의 객체 생성 |
| Rotation Randomization | 방향 변경 | 여러 각도의 객체 생성 |
| Pixel BBox | 객체 위치 기록 | `x1, y1, x2, y2` |
| YOLO Label | 정규화 BBox 저장 | `class xc yc w h` |
| Automatic QA | Label 자동 검사 | 형식·범위·파일 오류 확인 |
| Human QA | 시각적 검수 | 합성 장면이 현실적인지 판단 |
| Metadata | 생성 이력 기록 | 어떤 Source와 조건으로 만들었는지 추적 |
| Git | 코드·실험 기록 관리 | 재현 가능한 실습 상태 저장 |

---

# 오늘의 8시간 운영 구성

| 단계 | 권장 시간 | 내용 |
|---|---:|---|
| 본 실습 1 | 40분 | 프로젝트 연결·환경·입력 데이터 검사 |
| 본 실습 2 | 40분 | Detection Label·Alpha·BBox 개념과 Source Preview |
| 본 실습 3 | 55분 | Single Cut-Paste·Pixel BBox·YOLO Label 확인 |
| 본 실습 4 | 55분 | Batch Dataset 생성·Metadata |
| 본 실습 5 | 70분 | BBox 재시각화·Automatic QA·Human QA |
| 본 실습 6 | 40분 | 편향·Metadata·Data Lineage 해석 |
| Mini Challenge | 180분 | Challenge Source로 Rotation 한 변수 실험 |
| **합계** | **480분** | **8시간** |

실제 파일 생성 시간은 PC 성능과 Background 해상도에 따라 달라질 수 있습니다.

오늘도 한 번에 대량 생성하지 않습니다.

```text
Input Check
        ↓
Single Sample
        ↓
Smoke Test
        ↓
Batch Dataset
        ↓
QA
```

의 순서로 진행합니다.

---

# PART 1. 2일차 프로젝트에서 3일차를 이어서 시작합니다

---

# 1. 1~3일차가 어떻게 연결되는지 확인합니다

```text
1일차

Real Image
→ Augmentation

Real Image
→ VAE
→ Reconstruction / Random Sample


2일차

Real Train
→ DCGAN / WGAN-GP

Pretrained Diffusion
→ Prompt + Seed

        ↓

새로운 이미지 생성


3일차

Background
+
Transparent Object PNG
        ↓
Cut-Paste
        ↓
새로운 이미지
+
정답 BBox 자동 생성
```

3일차에서 가장 큰 변화는 **이미지와 Label을 동시에 만든다**는 점입니다.

```text
1~2일차
Image 중심

3일차
Image + Class + BBox
```

---

# 2. 교과 11 프로젝트 Root로 이동합니다

```bash
cd ~/ai_vision/subject11_synthetic_data
```

현재까지의 폴더를 확인합니다.

```bash
ls
```

다음 폴더가 보여야 합니다.

```text
day01_image_aug_vae
day02_generative_models
```

3일차 폴더를 만듭니다.

```bash
mkdir -p day03_detection_cutpaste
cd day03_detection_cutpaste
```

---

# 3. 3일차 폴더와 파일을 준비합니다

다음 구조를 만듭니다.

```bash
mkdir -p \
src \
scripts \
data \
dataset/main \
dataset/challenge \
results/main \
results/challenge \
reports/main \
reports/challenge
```

파일을 준비합니다.

```bash
touch src/day03_common.py

touch scripts/00_check_day03.py
touch scripts/01_preview_assets.py
touch scripts/02_cutpaste_single.py
touch scripts/03_generate_cutpaste_dataset.py
touch scripts/04_visualize_yolo_bbox.py
touch scripts/05_validate_yolo_labels.py
touch scripts/06_make_qa_report.py

touch day03_notes.md
touch README.md
touch .gitignore
```

현재 구조를 확인합니다.

```bash
tree -L 2
```

다음과 비슷하면 됩니다.

```text
day03_detection_cutpaste/
├─ data/
├─ dataset/
│  ├─ main/
│  └─ challenge/
├─ reports/
│  ├─ main/
│  └─ challenge/
├─ results/
│  ├─ main/
│  └─ challenge/
├─ scripts/
├─ src/
├─ day03_notes.md
├─ README.md
└─ .gitignore
```

---

# 4. 제공된 `day03_student_data`를 `data/` 아래에 배치합니다

수업에서 제공된 `day03_student_data` 폴더를 다음 위치에 복사합니다.

```text
day03_detection_cutpaste/
└─ data/
   └─ day03_student_data/
```

복사한 뒤 확인합니다.

```bash
tree data/day03_student_data -L 4
```

다음 구조가 보여야 합니다.

```text
data/day03_student_data/
├─ main/
│  ├─ backgrounds/
│  └─ objects/
│     ├─ drill/
│     ├─ fire_extinguisher/
│     └─ wet_floor_sign/
├─ challenge/
│  ├─ backgrounds/
│  └─ objects/
│     ├─ drill/
│     ├─ fire_extinguisher/
│     └─ wet_floor_sign/
└─ classes.txt
```

오늘 수업에서는 원본 BG-20K 전체 폴더나 Object의 원본 `images/`, `masks/`를 사용하지 않습니다.

이미 수업용으로 정리된 `day03_student_data`만 사용합니다.

---

# 5. 1일차 가상환경을 그대로 사용합니다

현재 위치는 다음입니다.

```text
~/ai_vision/subject11_synthetic_data/day03_detection_cutpaste
```

1일차에서 만든 `.venv`를 활성화합니다.

```bash
source ../day01_image_aug_vae/.venv/bin/activate
```

Python 위치를 확인합니다.

```bash
which python
```

라이브러리를 확인합니다.

```bash
python -c "import cv2, PIL, pandas; print('Day03 libraries OK')"
```

정상이라면:

```text
Day03 libraries OK
```

가 출력됩니다.

3일차를 위해 새로운 가상환경을 만들지 않습니다.

---

# 6. `main`과 `challenge`를 함께 처리할 공통 경로 코드를 작성합니다

본 실습과 Mini Challenge의 코드를 따로 복사하지 않습니다.

`--asset-set main` 또는 `--asset-set challenge`로 입력 Source만 바꾸도록 구성합니다.

**파일: `src/day03_common.py`**

```python
from __future__ import annotations

import shutil
from pathlib import Path


ROOT = Path(__file__).resolve().parents[1]

DATA_ROOT = (
    ROOT
    / "data"
    / "day03_student_data"
)

CLASS_FILE = DATA_ROOT / "classes.txt"

VALID_EXTENSIONS = {
    ".jpg",
    ".jpeg",
    ".png",
    ".bmp",
}


def list_images(
    directory: Path,
) -> list[Path]:
    if not directory.exists():
        return []

    return sorted(
        path
        for path in directory.iterdir()
        if path.is_file()
        and path.suffix.lower()
        in VALID_EXTENSIONS
    )


def read_classes() -> list[str]:
    if not CLASS_FILE.exists():
        raise FileNotFoundError(
            CLASS_FILE
        )

    classes = [
        line.strip()
        for line
        in CLASS_FILE.read_text(
            encoding="utf-8"
        ).splitlines()
        if line.strip()
    ]

    if not classes:
        raise RuntimeError(
            "classes.txt가 비어 있습니다."
        )

    return classes


def asset_paths(
    asset_set: str,
) -> dict[str, Path]:
    if asset_set not in {
        "main",
        "challenge",
    }:
        raise ValueError(
            f"지원하지 않는 asset_set: "
            f"{asset_set}"
        )

    root = DATA_ROOT / asset_set

    return {
        "root": root,
        "backgrounds": (
            root / "backgrounds"
        ),
        "objects": (
            root / "objects"
        ),
    }


def experiment_paths(
    asset_set: str,
    tag: str,
) -> dict[str, Path]:
    if not tag.strip():
        raise ValueError(
            "tag는 비어 있을 수 없습니다."
        )

    dataset_root = (
        ROOT
        / "dataset"
        / asset_set
        / tag
    )

    result_root = (
        ROOT
        / "results"
        / asset_set
        / tag
    )

    report_root = (
        ROOT
        / "reports"
        / asset_set
    )

    paths = {
        "images": (
            dataset_root
            / "images"
        ),
        "labels": (
            dataset_root
            / "labels"
        ),
        "visualized": (
            result_root
            / "visualized"
        ),
        "report": report_root,
    }

    report_root.mkdir(
        parents=True,
        exist_ok=True,
    )

    return paths


def reset_directory(
    directory: Path,
) -> None:
    if directory.exists():
        shutil.rmtree(
            directory
        )

    directory.mkdir(
        parents=True,
        exist_ok=True,
    )
```

### 이 코드는 왜 작성하나요?

본 실습과 Mini Challenge에서 같은 Python 파일을 재사용하기 위해 작성합니다.

```text
--asset-set main
→ 본 실습 Source

--asset-set challenge
→ Mini Challenge Source
```

또한 `--tag`를 이용하여 서로 다른 실험 결과가 덮어쓰이지 않도록 결과 폴더를 분리합니다.

### 의사코드로 읽어보기

```text
3일차 Root를 찾는다
        ↓
day03_student_data 경로를 만든다
        ↓
classes.txt를 읽는 함수를 만든다
        ↓
main 또는 challenge Source 경로를 만든다
        ↓
실험 tag별 dataset / results / reports 경로를 만든다
        ↓
필요하면 결과 폴더를 초기화하는 함수를 만든다
```

---

# 7. 입력 데이터가 정확한지 자동으로 검사합니다

오늘은 모든 학생이 같은 Source Data를 사용합니다.

따라서 본 실습에서 다음 수량이 맞는지 확인합니다.

```text
main

Background
20장

Object
drill               5장
fire_extinguisher   5장
wet_floor_sign      5장
```

Mini Challenge는 다음입니다.

```text
challenge

Background
10장

Object
각 Class 3장
```

또한 Object PNG가 실제 Alpha Channel을 가지고 있는지도 확인합니다.

**파일: `scripts/00_check_day03.py`**

```python
from __future__ import annotations

import argparse
import sys
from pathlib import Path

import cv2
from PIL import Image


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day03_common import (
    asset_paths,
    list_images,
    read_classes,
)


EXPECTED_CLASSES = [
    "drill",
    "fire_extinguisher",
    "wet_floor_sign",
]

EXPECTED_COUNTS = {
    "main": {
        "backgrounds": 20,
        "objects_per_class": 5,
    },
    "challenge": {
        "backgrounds": 10,
        "objects_per_class": 3,
    },
}


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--asset-set",
        default="main",
        choices=[
            "main",
            "challenge",
        ],
    )

    return parser.parse_args()


def has_alpha(
    path: Path,
) -> bool:
    with Image.open(path) as image:
        rgba = image.convert(
            "RGBA"
        )

        alpha = rgba.getchannel(
            "A"
        )

        alpha_min, alpha_max = (
            alpha.getextrema()
        )

    return (
        alpha_min < 255
        and alpha_max > 0
    )


def main() -> None:
    args = parse_args()

    classes = read_classes()

    if classes != EXPECTED_CLASSES:
        raise RuntimeError(
            "classes.txt 순서가 "
            "수업 기준과 다릅니다.\n"
            f"현재: {classes}\n"
            f"기준: {EXPECTED_CLASSES}"
        )

    paths = asset_paths(
        args.asset_set
    )

    expected = (
        EXPECTED_COUNTS[
            args.asset_set
        ]
    )

    backgrounds = list_images(
        paths["backgrounds"]
    )

    if len(backgrounds) != (
        expected["backgrounds"]
    ):
        raise RuntimeError(
            "Background 수가 다릅니다.\n"
            f"기대: "
            f"{expected['backgrounds']}\n"
            f"현재: {len(backgrounds)}"
        )

    unreadable = []

    for path in backgrounds:
        image = cv2.imread(
            str(path)
        )

        if image is None:
            unreadable.append(
                path.name
            )

    if unreadable:
        raise RuntimeError(
            "읽을 수 없는 Background: "
            + ", ".join(unreadable)
        )

    print("=" * 60)
    print(
        "Subject 11 - Day 03 Check"
    )
    print("=" * 60)

    print(
        "Asset Set   :",
        args.asset_set,
    )

    print(
        "Backgrounds :",
        len(backgrounds),
    )

    for class_id, class_name in enumerate(
        classes
    ):
        class_dir = (
            paths["objects"]
            / class_name
        )

        object_files = list_images(
            class_dir
        )

        if len(object_files) != (
            expected[
                "objects_per_class"
            ]
        ):
            raise RuntimeError(
                f"{class_name} 수가 "
                "수업 기준과 다릅니다.\n"
                f"기대: "
                f"{expected['objects_per_class']}\n"
                f"현재: {len(object_files)}"
            )

        alpha_bad = [
            path.name
            for path in object_files
            if not has_alpha(path)
        ]

        if alpha_bad:
            raise RuntimeError(
                f"{class_name}에서 "
                "Alpha 문제가 있습니다: "
                + ", ".join(
                    alpha_bad
                )
            )

        print(
            f"Class {class_id} "
            f"{class_name}: "
            f"{len(object_files)} "
            f"/ Alpha OK"
        )

    print("=" * 60)
    print("Day 03 check: READY")


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

Cut-Paste 코드를 실행하기 전에 **학생마다 Source Data가 다른 문제와 Alpha가 없는 PNG 문제를 먼저 차단**하기 위해 작성합니다.

### 의사코드로 읽어보기

```text
main 또는 challenge를 선택한다
        ↓
classes.txt 순서를 확인한다
        ↓
Background 수를 확인한다
        ↓
Background를 OpenCV로 읽어본다
        ↓
Class별 Object 수를 확인한다
        ↓
Object PNG의 Alpha Channel을 확인한다
        ↓
모두 기준과 같으면 READY
```

본 실습에서는 다음을 실행합니다.

```bash
python scripts/00_check_day03.py \
  --asset-set main
```

정상이라면 다음과 비슷하게 표시됩니다.

```text
Asset Set   : main
Backgrounds : 20
Class 0 drill: 5 / Alpha OK
Class 1 fire_extinguisher: 5 / Alpha OK
Class 2 wet_floor_sign: 5 / Alpha OK
Day 03 check: READY
```

---

# PART 2. Detection 데이터와 3일차 Source를 이해합니다

---

# 8. `main` Source Data를 직접 확인합니다

본 실습 Source는 다음과 같습니다.

```text
data/day03_student_data/main/
├─ backgrounds/
│  ├─ bg_001.jpg
│  ├─ ...
│  └─ bg_020.jpg
└─ objects/
   ├─ drill/
   │  ├─ drill_001.png
   │  └─ ...005.png
   ├─ fire_extinguisher/
   │  └─ ...005.png
   └─ wet_floor_sign/
      └─ ...005.png
```

Background와 Object의 역할은 다릅니다.

```text
Background JPG
→ 최종 합성 이미지의 바탕

Object RGBA PNG
→ Background 위에 붙일 객체
→ Alpha Channel로 객체 외부를 투명하게 처리
```

---

# 9. Class ID와 Label Contract를 고정합니다

오늘의 Class ID는 다음과 같습니다.

```text
0 → drill
1 → fire_extinguisher
2 → wet_floor_sign
```

예를 들어 `drill`이 합성되었다면 YOLO TXT 첫 번째 값은 항상 `0`입니다.

```text
0 x_center y_center width height
```

`fire_extinguisher`는:

```text
1 x_center y_center width height
```

`wet_floor_sign`은:

```text
2 x_center y_center width height
```

입니다.

이 규칙은 본 실습과 Mini Challenge에서 바뀌지 않습니다.

---

# 10. Alpha Channel을 이해합니다

일반 RGB 이미지는 다음 세 값을 사용합니다.

```text
R
G
B
```

오늘 사용하는 Object PNG는 한 값이 더 있습니다.

```text
R
G
B
A
```

`A`는 Alpha Channel입니다.

```text
Alpha = 0
→ 완전 투명

Alpha = 255
→ 완전 보임

중간값
→ 반투명
```

그래서 다음처럼 합성할 수 있습니다.

```text
Background
+
RGBA Object
        ↓
객체 부분만 보임
        ↓
객체 주변 사각형 배경은 보이지 않음
```

오늘 Source PNG는 이미 이 단계가 완료된 상태입니다.

원본 Mask를 다시 만드는 실습은 하지 않습니다.

---

# 11. Pixel BBox를 이해합니다

객체의 위치를 다음 네 값으로 표현할 수 있습니다.

```text
(x1, y1) ┌──────────────┐
         │    Object    │
         └──────────────┘ (x2, y2)
```

```text
x1
→ 왼쪽

y1
→ 위쪽

x2
→ 오른쪽

y2
→ 아래쪽
```

Cut-Paste에서는 프로그램이 객체를 붙이는 위치와 크기를 알고 있습니다.

따라서 BBox를 사람이 다시 마우스로 그리지 않아도 됩니다.

오늘 코드에서 BBox는 **원본 PNG의 바깥 사각형을 그대로 사용하지 않습니다.**

```text
RGBA Object
        ↓
Alpha가 실제로 보이는 영역만 Crop
        ↓
Scale 적용
        ↓
Rotation 적용
        ↓
회전 뒤 생긴 투명 여백을 다시 Alpha 기준으로 Crop
        ↓
최종 Object Width / Height 확인
        ↓
Background에 배치
        ↓
최종 Pixel BBox 계산
```

즉 회전하기 전 Object의 사각형을 그대로 BBox로 사용하는 것이 아니라, **Scale과 Rotation이 모두 적용된 뒤 실제로 보이는 Alpha 영역을 감싸는 최종 축 정렬 BBox**를 사용합니다.

```text
객체를 x1, y1에 붙임
        ↓
최종 Object의 Width / Height를 알고 있음
        ↓
x2 = x1 + width
y2 = y1 + height
        ↓
Pixel BBox 자동 계산
```

여기서 BBox는 회전된 객체 모양 자체를 따라가는 Polygon이나 Mask가 아니라, 객체 전체를 포함하는 **축에 평행한 사각형(Axis-Aligned BBox)** 입니다.

---

# 12. YOLO Label 형식을 이해합니다

YOLO Detection Label은 다음 순서입니다.

```text
class_id x_center y_center width height
```

좌표는 Pixel 숫자를 그대로 쓰지 않고 이미지 전체 크기를 기준으로 `0~1` 사이로 정규화합니다.

```text
x_center =
((x1 + x2) / 2)
/
image_width


y_center =
((y1 + y2) / 2)
/
image_height


bbox_width =
(x2 - x1)
/
image_width


bbox_height =
(y2 - y1)
/
image_height
```

예를 들어:

```text
0 0.542000 0.431000 0.158000 0.213000
```

이라면:

```text
0
→ drill

0.542000
→ BBox 중심 X

0.431000
→ BBox 중심 Y

0.158000
→ BBox Width

0.213000
→ BBox Height
```

를 뜻합니다.

---

# 13. Source Data를 한 화면에서 확인하는 코드를 작성합니다

Background 3장과 각 Class의 대표 Object를 한 화면에서 확인합니다.

**파일: `scripts/01_preview_assets.py`**

```python
from __future__ import annotations

import argparse
import sys
from pathlib import Path

from PIL import (
    Image,
    ImageDraw,
)


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day03_common import (
    asset_paths,
    list_images,
    read_classes,
)


TILE_SIZE = 220


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--asset-set",
        default="main",
        choices=[
            "main",
            "challenge",
        ],
    )

    return parser.parse_args()


def checkerboard(
    size: int,
    cell: int = 20,
) -> Image.Image:
    image = Image.new(
        "RGB",
        (size, size),
        (245, 245, 245),
    )

    draw = ImageDraw.Draw(
        image
    )

    for y in range(
        0,
        size,
        cell,
    ):
        for x in range(
            0,
            size,
            cell,
        ):
            if (
                (x // cell)
                + (y // cell)
            ) % 2:
                draw.rectangle(
                    [
                        x,
                        y,
                        x + cell - 1,
                        y + cell - 1,
                    ],
                    fill=(
                        218,
                        218,
                        218,
                    ),
                )

    return image


def make_tile(
    path: Path,
    transparent: bool,
) -> Image.Image:
    tile = checkerboard(
        TILE_SIZE
    )

    with Image.open(path) as image:
        if transparent:
            obj = image.convert(
                "RGBA"
            )

            obj.thumbnail(
                (
                    TILE_SIZE - 20,
                    TILE_SIZE - 20,
                ),
                Image.Resampling.LANCZOS,
            )

            tile_rgba = tile.convert(
                "RGBA"
            )

            x = (
                TILE_SIZE
                - obj.width
            ) // 2

            y = (
                TILE_SIZE
                - obj.height
            ) // 2

            tile_rgba.alpha_composite(
                obj,
                dest=(x, y),
            )

            return tile_rgba.convert(
                "RGB"
            )

        image = image.convert("RGB")

        image.thumbnail(
            (
                TILE_SIZE - 20,
                TILE_SIZE - 20,
            ),
            Image.Resampling.LANCZOS,
        )

        x = (
            TILE_SIZE
            - image.width
        ) // 2

        y = (
            TILE_SIZE
            - image.height
        ) // 2

        tile.paste(
            image,
            (x, y),
        )

        return tile


def main() -> None:
    args = parse_args()

    paths = asset_paths(
        args.asset_set
    )

    classes = read_classes()

    backgrounds = list_images(
        paths["backgrounds"]
    )[:3]

    items = [
        (
            path,
            False,
            path.name,
        )
        for path in backgrounds
    ]

    for class_name in classes:
        files = list_images(
            paths["objects"]
            / class_name
        )

        if files:
            items.append(
                (
                    files[0],
                    True,
                    class_name,
                )
            )

    columns = 3
    rows = (
        len(items)
        + columns - 1
    ) // columns

    header = 35

    canvas = Image.new(
        "RGB",
        (
            columns * TILE_SIZE,
            rows
            * (
                TILE_SIZE
                + header
            ),
        ),
        "white",
    )

    draw = ImageDraw.Draw(
        canvas
    )

    for index, (
        path,
        transparent,
        label,
    ) in enumerate(items):
        row = index // columns
        column = index % columns

        x = column * TILE_SIZE

        y = row * (
            TILE_SIZE
            + header
        )

        tile = make_tile(
            path,
            transparent,
        )

        canvas.paste(
            tile,
            (x, y),
        )

        draw.text(
            (
                x + 5,
                y + TILE_SIZE + 8,
            ),
            label,
            fill="black",
        )

    output_dir = (
        ROOT
        / "results"
        / args.asset_set
    )

    output_dir.mkdir(
        parents=True,
        exist_ok=True,
    )

    output_path = (
        output_dir
        / "asset_preview.jpg"
    )

    canvas.save(
        output_path,
        quality=92,
    )

    print(
        "Saved:",
        output_path,
    )


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

대량 합성을 시작하기 전에 **Background와 Object Source가 수업 의도대로 보이는지 한 번에 확인**하기 위해 작성합니다.

### 의사코드로 읽어보기

```text
main 또는 challenge를 선택한다
        ↓
Background 3장을 고른다
        ↓
각 Class의 첫 Object를 고른다
        ↓
Object는 체크무늬 위에 합성한다
        ↓
Background와 Object를 한 Grid에 배치한다
        ↓
asset_preview.jpg로 저장한다
```

본 실습에서는 다음을 실행합니다.

```bash
python scripts/01_preview_assets.py \
  --asset-set main
```

결과:

```text
results/main/asset_preview.jpg
```

다음 항목을 확인합니다.

```text
Background가 정상적으로 보이는가?

Object 주변이 투명한가?

세 Class를 눈으로 구분할 수 있는가?

Object에 불필요한 사각 배경이 남아 있지 않은가?
```

---

# PART 3. 객체 한 개를 합성하고 BBox를 직접 확인합니다

---

# 14. 처음부터 여러 장을 만들지 않습니다

오늘의 첫 번째 합성은 다음 한 쌍으로 시작합니다.

```text
main/backgrounds/bg_001.jpg
+
main/objects/drill/drill_001.png
```

먼저 한 장에서 다음 전체 흐름을 확인합니다.

```text
Background 읽기
        ↓
RGBA Object 읽기
        ↓
Alpha 영역 Crop
        ↓
Scale 적용
        ↓
Rotation 적용
        ↓
회전 후 Alpha 영역 다시 Crop
        ↓
최종 Object 크기 확인
        ↓
Position 결정
        ↓
Cut-Paste
        ↓
최종 Object 기준 Pixel BBox
        ↓
YOLO Label
        ↓
BBox Visualization
```

한 장이 정확하지 않으면 100장을 자동 생성해도 의미가 없습니다.

---

# 15. Single Cut-Paste 코드를 작성합니다

**파일: `scripts/02_cutpaste_single.py`**

```python
from __future__ import annotations

import argparse
import sys
from pathlib import Path

from PIL import (
    Image,
    ImageDraw,
)


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day03_common import (
    asset_paths,
    read_classes,
)


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--asset-set",
        default="main",
        choices=[
            "main",
            "challenge",
        ],
    )

    parser.add_argument(
        "--background",
        default="bg_001.jpg",
    )

    parser.add_argument(
        "--class-name",
        default="drill",
    )

    parser.add_argument(
        "--object",
        default="drill_001.png",
    )

    parser.add_argument(
        "--scale",
        type=float,
        default=0.18,
    )

    parser.add_argument(
        "--rotation",
        type=float,
        default=10.0,
    )

    parser.add_argument(
        "--x-ratio",
        type=float,
        default=0.50,
    )

    parser.add_argument(
        "--y-ratio",
        type=float,
        default=0.50,
    )

    return parser.parse_args()


def crop_to_alpha(
    image: Image.Image,
) -> Image.Image:
    image = image.convert(
        "RGBA"
    )

    alpha = image.getchannel(
        "A"
    )

    bbox = alpha.getbbox()

    if bbox is None:
        raise RuntimeError(
            "보이는 Object Pixel이 없습니다."
        )

    return image.crop(
        bbox
    )


def to_yolo(
    x1,
    y1,
    x2,
    y2,
    image_width,
    image_height,
):
    x_center = (
        (x1 + x2) / 2
    ) / image_width

    y_center = (
        (y1 + y2) / 2
    ) / image_height

    width = (
        x2 - x1
    ) / image_width

    height = (
        y2 - y1
    ) / image_height

    return (
        x_center,
        y_center,
        width,
        height,
    )


def main() -> None:
    args = parse_args()

    if not 0 < args.scale < 1:
        raise ValueError(
            "--scale은 0과 1 사이여야 합니다."
        )

    if not 0 <= args.x_ratio <= 1:
        raise ValueError(
            "--x-ratio는 0~1이어야 합니다."
        )

    if not 0 <= args.y_ratio <= 1:
        raise ValueError(
            "--y-ratio는 0~1이어야 합니다."
        )

    paths = asset_paths(
        args.asset_set
    )

    classes = read_classes()

    if args.class_name not in classes:
        raise ValueError(
            f"알 수 없는 Class: "
            f"{args.class_name}"
        )

    class_id = classes.index(
        args.class_name
    )

    background_path = (
        paths["backgrounds"]
        / args.background
    )

    object_path = (
        paths["objects"]
        / args.class_name
        / args.object
    )

    if not background_path.exists():
        raise FileNotFoundError(
            background_path
        )

    if not object_path.exists():
        raise FileNotFoundError(
            object_path
        )

    background = Image.open(
        background_path
    ).convert("RGBA")

    obj = Image.open(
        object_path
    ).convert("RGBA")

    obj = crop_to_alpha(obj)

    bg_width, bg_height = (
        background.size
    )

    obj_width, obj_height = (
        obj.size
    )

    target_width = max(
        8,
        int(
            bg_width
            * args.scale
        ),
    )

    ratio = (
        target_width
        / obj_width
    )

    target_height = max(
        8,
        int(
            obj_height
            * ratio
        ),
    )

    obj = obj.resize(
        (
            target_width,
            target_height,
        ),
        Image.Resampling.LANCZOS,
    )

    obj = obj.rotate(
        args.rotation,
        expand=True,
        resample=(
            Image.Resampling.BICUBIC
        ),
    )

    obj = crop_to_alpha(obj)

    obj_width, obj_height = (
        obj.size
    )

    if (
        obj_width >= bg_width
        or obj_height >= bg_height
    ):
        raise RuntimeError(
            "Object가 Background보다 큽니다."
        )

    max_x = (
        bg_width
        - obj_width
    )

    max_y = (
        bg_height
        - obj_height
    )

    x1 = int(
        max_x
        * args.x_ratio
    )

    y1 = int(
        max_y
        * args.y_ratio
    )

    x2 = x1 + obj_width
    y2 = y1 + obj_height

    composite = background.copy()

    composite.alpha_composite(
        obj,
        dest=(x1, y1),
    )

    output_dir = (
        ROOT
        / "results"
        / args.asset_set
        / "single"
    )

    output_dir.mkdir(
        parents=True,
        exist_ok=True,
    )

    image_path = (
        output_dir
        / "single_cutpaste.jpg"
    )

    label_path = (
        output_dir
        / "single_cutpaste.txt"
    )

    bbox_path = (
        output_dir
        / "single_cutpaste_bbox.jpg"
    )

    composite.convert(
        "RGB"
    ).save(
        image_path,
        quality=95,
    )

    (
        x_center,
        y_center,
        bbox_width,
        bbox_height,
    ) = to_yolo(
        x1,
        y1,
        x2,
        y2,
        bg_width,
        bg_height,
    )

    label_text = (
        f"{class_id} "
        f"{x_center:.6f} "
        f"{y_center:.6f} "
        f"{bbox_width:.6f} "
        f"{bbox_height:.6f}"
    )

    label_path.write_text(
        label_text + "\n",
        encoding="utf-8",
    )

    visualized = composite.copy()

    draw = ImageDraw.Draw(
        visualized
    )

    draw.rectangle(
        [
            x1,
            y1,
            x2,
            y2,
        ],
        outline="red",
        width=4,
    )

    draw.text(
        (
            x1,
            max(
                0,
                y1 - 18,
            ),
        ),
        args.class_name,
        fill="red",
    )

    visualized.convert(
        "RGB"
    ).save(
        bbox_path,
        quality=95,
    )

    print(
        "Background:",
        background_path.name,
    )

    print(
        "Object    :",
        object_path.name,
    )

    print(
        "Class ID  :",
        class_id,
    )

    print(
        "Pixel BBox:",
        [
            x1,
            y1,
            x2,
            y2,
        ],
    )

    print(
        "YOLO Label:",
        label_text,
    )

    print(
        "Saved:",
        output_dir,
    )


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

대량 생성 전에 **Cut-Paste와 BBox 계산이 정확히 연결되는지 한 장으로 검증**하기 위해 작성합니다.

이 코드의 `crop_to_alpha()`는 Object의 투명 여백을 제거합니다. Scale을 적용한 뒤 회전하면 다시 투명 여백이 생길 수 있으므로, 회전 후에도 한 번 더 Alpha 영역을 Crop합니다. 그 다음의 `obj_width`, `obj_height`가 최종 BBox 크기가 됩니다.

따라서 Single 실습에서 저장되는 BBox는 **회전 전 원본 PNG 사각형이 아니라 최종 합성 Object 기준 BBox**입니다.

### 의사코드로 읽어보기

```text
main Source를 선택한다
        ↓
bg_001.jpg를 읽는다
        ↓
drill_001.png를 RGBA로 읽는다
        ↓
투명 영역을 제외한 객체 영역을 Crop한다
        ↓
Background Width 기준으로 크기를 조절한다
        ↓
객체를 회전한다
        ↓
회전 후 다시 Alpha 영역을 Crop한다
        ↓
x-ratio / y-ratio로 위치를 정한다
        ↓
Background에 객체를 합성한다
        ↓
x1, y1, x2, y2를 계산한다
        ↓
YOLO 좌표로 변환한다
        ↓
이미지·TXT·BBox 확인 이미지를 저장한다
```

---

# 16. 본 실습 Single Cut-Paste를 실행합니다

```bash
python scripts/02_cutpaste_single.py \
  --asset-set main \
  --background bg_001.jpg \
  --class-name drill \
  --object drill_001.png \
  --scale 0.18 \
  --rotation 10 \
  --x-ratio 0.50 \
  --y-ratio 0.50
```

결과:

```text
results/main/single/
├─ single_cutpaste.jpg
├─ single_cutpaste.txt
└─ single_cutpaste_bbox.jpg
```

---

# 17. Single 결과를 반드시 직접 확인합니다

`single_cutpaste_bbox.jpg`를 열어 다음을 확인합니다.

```text
drill이 Background 안에 있는가?

객체 주변에 사각형 배경이 남아 있지 않은가?

BBox가 drill 전체를 감싸는가?

BBox가 지나치게 넓지 않은가?

TXT의 첫 값이 0인가?
```

TXT도 확인합니다.

```bash
cat results/main/single/single_cutpaste.txt
```

첫 값이 다음이어야 합니다.

```text
0
```

왜냐하면:

```text
Class 0
→ drill
```

이기 때문입니다.

한 장이 정상인 것을 확인한 뒤에만 여러 장 생성으로 넘어갑니다.

---

# PART 4. 여러 Synthetic Detection Image와 YOLO Label을 자동 생성합니다

---

# 18. Batch 생성에서 무엇을 Randomize하는지 확인합니다

오늘은 다음 조건을 바꿉니다.

```text
Background
Object View
Position X / Y
Scale
Rotation
```

Class는 무작정 Random하게만 고르지 않습니다.

120장을 생성한다면 세 Class를 각각 40장씩 만들고, 그 순서를 Seed로 섞습니다.

```text
drill               40
fire_extinguisher   40
wet_floor_sign      40
```

이렇게 하면 Source가 작더라도 Class 수가 한쪽으로 심하게 치우치는 문제를 줄일 수 있습니다.

---

# 19. Metadata에 무엇을 기록할지 확인합니다

각 Synthetic Image에는 다음 정보가 함께 기록됩니다.

```text
생성 파일명
Asset Set
실험 Tag
Background
Object
Class ID
Class Name
Scale
Rotation Limit
Rotation Unit
실제 Rotation
x_ratio / y_ratio
Pixel BBox
YOLO BBox
Global Seed
Sample Seed
Approved 상태
```

`global_seed`는 전체 실험의 기준 Seed이고, `sample_seed`는 각 Synthetic Image의 Source·Scale·상대 위치 조건을 다시 만들 수 있도록 Sample마다 고정한 Seed입니다.

같은 `asset-set`, `count`, `seed`, `scale` 조건을 유지하고 Rotation 범위만 바꾸면 각 Sample의 **Class·Background·Object View·Scale·x_ratio·y_ratio는 동일하게 유지**됩니다. 이 구조는 Mini Challenge에서 Rotation의 영향만 비교하기 위해 필요합니다.

예:

```text
synthetic_00023.jpg

background:
bg_014.jpg

object:
fire_extinguisher_003.png

class:
1 / fire_extinguisher

scale:
0.143

rotation:
-7.2

bbox:
x1, y1, x2, y2

global_seed:
42

sample_seed:
Sample마다 고정된 값

approved:
pending
```

나중에 이상한 이미지를 발견하면 Metadata를 이용해 **어떤 Source와 생성 조건에서 만들어졌는지 다시 추적**할 수 있습니다.

---

# 20. Batch Cut-Paste 생성 코드를 작성합니다

**파일: `scripts/03_generate_cutpaste_dataset.py`**

```python
from __future__ import annotations

import argparse
import csv
import random
import sys
from pathlib import Path

from PIL import Image


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day03_common import (
    asset_paths,
    experiment_paths,
    list_images,
    read_classes,
    reset_directory,
)


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--asset-set",
        default="main",
        choices=[
            "main",
            "challenge",
        ],
    )

    parser.add_argument(
        "--tag",
        default="baseline",
    )

    parser.add_argument(
        "--count",
        type=int,
        default=120,
    )

    parser.add_argument(
        "--seed",
        type=int,
        default=42,
    )

    parser.add_argument(
        "--scale-min",
        type=float,
        default=0.08,
    )

    parser.add_argument(
        "--scale-max",
        type=float,
        default=0.22,
    )

    parser.add_argument(
        "--rotation",
        type=float,
        default=15.0,
    )

    return parser.parse_args()


def crop_to_alpha(
    image: Image.Image,
):
    image = image.convert(
        "RGBA"
    )

    alpha = image.getchannel(
        "A"
    )

    bbox = alpha.getbbox()

    if bbox is None:
        return None

    return image.crop(
        bbox
    )


def to_yolo(
    x1,
    y1,
    x2,
    y2,
    image_width,
    image_height,
):
    x_center = (
        (x1 + x2) / 2
    ) / image_width

    y_center = (
        (y1 + y2) / 2
    ) / image_height

    width = (
        x2 - x1
    ) / image_width

    height = (
        y2 - y1
    ) / image_height

    return (
        x_center,
        y_center,
        width,
        height,
    )


def main() -> None:
    args = parse_args()

    if args.count < 1:
        raise ValueError(
            "--count는 1 이상이어야 합니다."
        )

    if not (
        0 < args.scale_min
        <= args.scale_max
        < 1
    ):
        raise ValueError(
            "Scale 범위를 확인하세요."
        )

    if args.rotation < 0:
        raise ValueError(
            "--rotation은 0 이상이어야 합니다."
        )

    source = asset_paths(
        args.asset_set
    )

    output = experiment_paths(
        args.asset_set,
        args.tag,
    )

    reset_directory(
        output["images"]
    )

    reset_directory(
        output["labels"]
    )

    classes = read_classes()

    backgrounds = list_images(
        source["backgrounds"]
    )

    if not backgrounds:
        raise RuntimeError(
            "Background가 없습니다."
        )

    object_map = {}

    for class_name in classes:
        files = list_images(
            source["objects"]
            / class_name
        )

        if not files:
            raise RuntimeError(
                f"{class_name} Object가 "
                "없습니다."
            )

        object_map[
            class_name
        ] = files

    schedule_rng = random.Random(
        args.seed
    )

    class_schedule = [
        classes[
            index
            % len(classes)
        ]
        for index in range(
            args.count
        )
    ]

    schedule_rng.shuffle(
        class_schedule
    )

    rows = []

    for sample_index in range(
        args.count
    ):
        class_name = (
            class_schedule[
                sample_index
            ]
        )

        class_id = classes.index(
            class_name
        )

        sample_seed = (
            args.seed * 100_000
            + sample_index
        )

        sample_rng = random.Random(
            sample_seed
        )

        background_path = (
            sample_rng.choice(
                backgrounds
            )
        )

        object_path = (
            sample_rng.choice(
                object_map[
                    class_name
                ]
            )
        )

        scale = sample_rng.uniform(
            args.scale_min,
            args.scale_max,
        )

        rotation_unit = (
            sample_rng.uniform(
                -1.0,
                1.0,
            )
        )

        x_ratio = sample_rng.random()
        y_ratio = sample_rng.random()

        rotation = (
            rotation_unit
            * args.rotation
        )

        with Image.open(
            background_path
        ) as image:
            background = image.convert(
                "RGBA"
            )

        with Image.open(
            object_path
        ) as image:
            obj = image.convert(
                "RGBA"
            )

        obj = crop_to_alpha(
            obj
        )

        if obj is None:
            raise RuntimeError(
                "보이는 Object Pixel이 없습니다: "
                f"{object_path}"
            )

        bg_width, bg_height = (
            background.size
        )

        obj_width, obj_height = (
            obj.size
        )

        target_width = max(
            8,
            int(
                bg_width
                * scale
            ),
        )

        ratio = (
            target_width
            / obj_width
        )

        target_height = max(
            8,
            int(
                obj_height
                * ratio
            ),
        )

        if (
            target_width >= bg_width
            or target_height >= bg_height
        ):
            raise RuntimeError(
                "Scale 범위가 너무 커서 "
                "Object가 Background 안에 들어가지 않습니다.\n"
                f"Sample: {sample_index + 1}\n"
                f"Object: {object_path.name}\n"
                f"Scale : {scale:.5f}"
            )

        obj = obj.resize(
            (
                target_width,
                target_height,
            ),
            Image.Resampling.LANCZOS,
        )

        obj = obj.rotate(
            rotation,
            expand=True,
            resample=(
                Image.Resampling.BICUBIC
            ),
        )

        obj = crop_to_alpha(
            obj
        )

        if obj is None:
            raise RuntimeError(
                "회전 후 보이는 Object Pixel이 없습니다: "
                f"{object_path}"
            )

        obj_width, obj_height = (
            obj.size
        )

        if (
            obj_width >= bg_width
            or obj_height >= bg_height
        ):
            raise RuntimeError(
                "Rotation 후 Object가 "
                "Background보다 커졌습니다.\n"
                f"Sample: {sample_index + 1}\n"
                f"Object: {object_path.name}\n"
                "scale-max 또는 rotation 범위를 "
                "줄여주세요."
            )

        max_x = (
            bg_width
            - obj_width
        )

        max_y = (
            bg_height
            - obj_height
        )

        x1 = int(
            round(
                max_x
                * x_ratio
            )
        )

        y1 = int(
            round(
                max_y
                * y_ratio
            )
        )

        x2 = x1 + obj_width
        y2 = y1 + obj_height

        composite = (
            background.copy()
        )

        composite.alpha_composite(
            obj,
            dest=(x1, y1),
        )

        generated = (
            sample_index + 1
        )

        stem = (
            f"synthetic_"
            f"{generated:05d}"
        )

        image_path = (
            output["images"]
            / f"{stem}.jpg"
        )

        label_path = (
            output["labels"]
            / f"{stem}.txt"
        )

        composite.convert(
            "RGB"
        ).save(
            image_path,
            quality=95,
        )

        (
            x_center,
            y_center,
            bbox_width,
            bbox_height,
        ) = to_yolo(
            x1,
            y1,
            x2,
            y2,
            bg_width,
            bg_height,
        )

        label_text = (
            f"{class_id} "
            f"{x_center:.6f} "
            f"{y_center:.6f} "
            f"{bbox_width:.6f} "
            f"{bbox_height:.6f}"
        )

        label_path.write_text(
            label_text + "\n",
            encoding="utf-8",
        )

        rows.append(
            {
                "file": image_path.name,
                "asset_set": (
                    args.asset_set
                ),
                "tag": args.tag,
                "background": (
                    background_path.name
                ),
                "object": (
                    object_path.name
                ),
                "class_id": class_id,
                "class_name": (
                    class_name
                ),
                "scale": round(
                    scale,
                    5,
                ),
                "rotation_limit": (
                    args.rotation
                ),
                "rotation_unit": round(
                    rotation_unit,
                    6,
                ),
                "rotation": round(
                    rotation,
                    3,
                ),
                "x_ratio": round(
                    x_ratio,
                    6,
                ),
                "y_ratio": round(
                    y_ratio,
                    6,
                ),
                "x1": x1,
                "y1": y1,
                "x2": x2,
                "y2": y2,
                "yolo_x_center": round(
                    x_center,
                    6,
                ),
                "yolo_y_center": round(
                    y_center,
                    6,
                ),
                "yolo_width": round(
                    bbox_width,
                    6,
                ),
                "yolo_height": round(
                    bbox_height,
                    6,
                ),
                "global_seed": (
                    args.seed
                ),
                "sample_seed": (
                    sample_seed
                ),
                "approved": "pending",
            }
        )

    metadata_path = (
        output["report"]
        / (
            f"{args.tag}"
            f"_metadata.csv"
        )
    )

    fieldnames = list(
        rows[0].keys()
    )

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
            rows
        )

    print("=" * 60)
    print(
        "Cut-Paste Dataset Generated"
    )
    print("=" * 60)

    print(
        "Asset Set :",
        args.asset_set,
    )

    print(
        "Tag       :",
        args.tag,
    )

    print(
        "Generated :",
        len(rows),
    )

    print(
        "Images    :",
        output["images"],
    )

    print(
        "Labels    :",
        output["labels"],
    )

    print(
        "Metadata  :",
        metadata_path,
    )


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

작은 Source Data에서 **위치·크기·회전 조건을 바꾸면서 Detection 학습 형식의 이미지와 Label을 자동으로 여러 장 만들기 위해** 작성합니다.

Single 실습과 같은 BBox 계약을 그대로 사용합니다.

```text
Alpha Crop
→ Scale
→ Rotation
→ 회전 후 Alpha 재Crop
→ 최종 Object 크기
→ Pixel BBox
→ YOLO BBox
```

따라서 Batch Dataset에서도 회전 전 PNG 크기가 아니라 **최종 합성 Object의 보이는 영역을 감싸는 BBox**가 Label에 저장됩니다.

또한 Sample마다 별도의 `sample_seed`와 `x_ratio / y_ratio`를 기록하여, 같은 Seed에서 Rotation 범위만 바꾸는 Mini Challenge가 실제로 한 변수 실험이 되도록 합니다.

### 의사코드로 읽어보기

```text
main 또는 challenge Source를 선택한다
        ↓
실험 tag를 정한다
        ↓
생성 수·Global Seed·Scale·Rotation 범위를 입력받는다
        ↓
Class 수가 균형을 이루도록 Schedule을 만든다
        ↓
각 Sample마다 고정 Sample Seed를 만든다
        ↓
그 Seed로 Background·Object·Scale을 선택한다
        ↓
Rotation Unit과 x_ratio / y_ratio를 만든다
        ↓
Rotation Unit × Rotation 범위로 실제 각도를 만든다
        ↓
x_ratio / y_ratio로 상대적 배치 위치를 정한다
        ↓
Cut-Paste한다
        ↓
Pixel BBox와 YOLO BBox를 계산한다
        ↓
같은 이름의 JPG와 TXT를 저장한다
        ↓
생성 조건을 Metadata CSV에 기록한다
```

---

# 21. 먼저 6장 Smoke Test를 실행합니다

세 Class가 최소 두 장씩 포함될 수 있도록 6장으로 확인합니다.

```bash
python scripts/03_generate_cutpaste_dataset.py \
  --asset-set main \
  --tag smoke \
  --count 6 \
  --seed 42 \
  --scale-min 0.08 \
  --scale-max 0.22 \
  --rotation 15
```

확인합니다.

```bash
ls dataset/main/smoke/images
```

```bash
ls dataset/main/smoke/labels
```

같은 Stem의 JPG와 TXT가 존재해야 합니다.

```text
synthetic_00001.jpg
synthetic_00001.txt
```

Smoke Test는 `smoke` 폴더에 따로 저장되므로 본 실습 결과를 덮어쓰지 않습니다.

---

# 22. 본 실습 Dataset 120장을 생성합니다

```bash
python scripts/03_generate_cutpaste_dataset.py \
  --asset-set main \
  --tag baseline \
  --count 120 \
  --seed 42 \
  --scale-min 0.08 \
  --scale-max 0.22 \
  --rotation 15
```

정상이라면 다음 구조가 생성됩니다.

```text
dataset/main/baseline/
├─ images/
│  ├─ synthetic_00001.jpg
│  ├─ ...
│  └─ synthetic_00120.jpg
└─ labels/
   ├─ synthetic_00001.txt
   ├─ ...
   └─ synthetic_00120.txt
```

Metadata:

```text
reports/main/baseline_metadata.csv
```

---

# 23. Label과 Metadata를 직접 확인합니다

첫 Label을 확인합니다.

```bash
cat dataset/main/baseline/labels/synthetic_00001.txt
```

Metadata 앞부분을 확인합니다.

```bash
head reports/main/baseline_metadata.csv
```

다음 두 가지가 다른 역할을 한다는 점을 구분합니다.

```text
YOLO TXT
→ Detection Model이 사용하는 정답

Metadata CSV
→ 사람이 생성 조건과 이력을 추적하는 기록
```

---

# PART 5. BBox를 다시 그리고 Automatic QA와 Human QA를 수행합니다

---

# 24. YOLO Label을 다시 이미지 위에 그리는 코드를 작성합니다

자동 생성한 Label도 다시 확인해야 합니다.

**파일: `scripts/04_visualize_yolo_bbox.py`**

```python
from __future__ import annotations

import argparse
import sys
from pathlib import Path

import cv2


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day03_common import (
    experiment_paths,
    read_classes,
    reset_directory,
)


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--asset-set",
        default="main",
        choices=[
            "main",
            "challenge",
        ],
    )

    parser.add_argument(
        "--tag",
        default="baseline",
    )

    parser.add_argument(
        "--count",
        type=int,
        default=20,
    )

    return parser.parse_args()


def yolo_to_pixel(
    x_center,
    y_center,
    bbox_width,
    bbox_height,
    image_width,
    image_height,
):
    x_center *= image_width
    y_center *= image_height
    bbox_width *= image_width
    bbox_height *= image_height

    x1 = int(
        round(
            x_center
            - bbox_width / 2
        )
    )

    y1 = int(
        round(
            y_center
            - bbox_height / 2
        )
    )

    x2 = int(
        round(
            x_center
            + bbox_width / 2
        )
    )

    y2 = int(
        round(
            y_center
            + bbox_height / 2
        )
    )

    return (
        x1,
        y1,
        x2,
        y2,
    )


def main() -> None:
    args = parse_args()

    paths = experiment_paths(
        args.asset_set,
        args.tag,
    )

    classes = read_classes()

    reset_directory(
        paths["visualized"]
    )

    image_files = sorted(
        paths["images"].glob(
            "*.jpg"
        )
    )[: args.count]

    if not image_files:
        raise RuntimeError(
            "시각화할 이미지가 없습니다."
        )

    saved_count = 0

    for image_path in image_files:
        label_path = (
            paths["labels"]
            / (
                image_path.stem
                + ".txt"
            )
        )

        if not label_path.exists():
            continue

        image = cv2.imread(
            str(image_path)
        )

        if image is None:
            continue

        height, width = (
            image.shape[:2]
        )

        lines = (
            label_path
            .read_text(
                encoding="utf-8"
            )
            .splitlines()
        )

        for line in lines:
            values = line.split()

            if len(values) != 5:
                continue

            class_id = int(
                values[0]
            )

            x_center = float(
                values[1]
            )

            y_center = float(
                values[2]
            )

            bbox_width = float(
                values[3]
            )

            bbox_height = float(
                values[4]
            )

            (
                x1,
                y1,
                x2,
                y2,
            ) = yolo_to_pixel(
                x_center,
                y_center,
                bbox_width,
                bbox_height,
                width,
                height,
            )

            cv2.rectangle(
                image,
                (x1, y1),
                (x2, y2),
                (0, 0, 255),
                2,
            )

            class_name = (
                classes[class_id]
                if (
                    0
                    <= class_id
                    < len(classes)
                )
                else (
                    f"class_"
                    f"{class_id}"
                )
            )

            cv2.putText(
                image,
                class_name,
                (
                    x1,
                    max(
                        20,
                        y1 - 5,
                    ),
                ),
                (
                    cv2
                    .FONT_HERSHEY_SIMPLEX
                ),
                0.65,
                (0, 0, 255),
                2,
                cv2.LINE_AA,
            )

        output_path = (
            paths["visualized"]
            / image_path.name
        )

        cv2.imwrite(
            str(output_path),
            image,
        )

        saved_count += 1

    print(
        "Visualized:",
        saved_count,
    )

    print(
        "Output:",
        paths["visualized"],
    )


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

TXT의 숫자만 보고 BBox가 정확하다고 판단할 수 없기 때문에 **YOLO Label을 다시 Pixel 좌표로 바꾸어 실제 이미지 위에 그려보기 위해** 작성합니다.

### 의사코드로 읽어보기

```text
실험 tag의 이미지와 Label을 찾는다
        ↓
YOLO 좌표를 읽는다
        ↓
0~1 좌표에 이미지 Width / Height를 곱한다
        ↓
Pixel BBox로 되돌린다
        ↓
이미지 위에 사각형과 Class 이름을 그린다
        ↓
최대 지정한 수만큼 저장한다
```

본 실습 결과 20장을 확인합니다.

```bash
python scripts/04_visualize_yolo_bbox.py \
  --asset-set main \
  --tag baseline \
  --count 20
```

결과:

```text
results/main/baseline/visualized/
```

---

# 25. BBox 재시각화 결과를 확인합니다

최소 20장을 직접 확인합니다.

```text
BBox가 Object 전체를 감싸는가?

Class 이름이 맞는가?

회전된 Object에서도 BBox가 정상인가?

작은 Object도 BBox 안에 들어가는가?

BBox가 이미지 밖으로 나가지 않는가?
```

프로그램이 자동으로 만든 Label도 잘못될 수 있습니다.

```text
자동 생성
≠
자동 정답 보장
```

따라서 **재시각화가 QA의 중요한 단계**입니다.

---

# 26. 모든 YOLO Label을 자동 검사하는 코드를 작성합니다

이번에는 전체 120장을 자동 검사합니다.

YOLO TXT는 소수점 6자리로 저장하므로, 이미지 경계에 정확히 닿는 BBox는 반올림 때문에 `0`보다 아주 조금 작거나 `1`보다 아주 조금 크게 복원될 수 있습니다. 검사용 코드에서는 `1e-6`의 작은 허용오차를 사용하여 **반올림 오차를 실제 Label 오류로 잘못 판정하지 않도록** 합니다.

**파일: `scripts/05_validate_yolo_labels.py`**

```python
from __future__ import annotations

import argparse
import math
import sys
from pathlib import Path

import cv2
import pandas as pd


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day03_common import (
    experiment_paths,
    read_classes,
)


EPSILON = 1e-6


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--asset-set",
        default="main",
        choices=[
            "main",
            "challenge",
        ],
    )

    parser.add_argument(
        "--tag",
        default="baseline",
    )

    return parser.parse_args()


def main() -> None:
    args = parse_args()

    paths = experiment_paths(
        args.asset_set,
        args.tag,
    )

    classes = read_classes()

    image_files = sorted(
        paths["images"].glob(
            "*.jpg"
        )
    )

    label_files = sorted(
        paths["labels"].glob(
            "*.txt"
        )
    )

    image_stems = {
        path.stem
        for path in image_files
    }

    label_stems = {
        path.stem
        for path in label_files
    }

    all_stems = sorted(
        image_stems
        | label_stems
    )

    rows = []

    for stem in all_stems:
        image_path = (
            paths["images"]
            / f"{stem}.jpg"
        )

        label_path = (
            paths["labels"]
            / f"{stem}.txt"
        )

        reasons = []

        if not image_path.exists():
            reasons.append(
                "missing_image"
            )

        if not label_path.exists():
            reasons.append(
                "missing_label"
            )

        if image_path.exists():
            image = cv2.imread(
                str(image_path)
            )

            if image is None:
                reasons.append(
                    "unreadable_image"
                )

        if label_path.exists():
            lines = (
                label_path
                .read_text(
                    encoding="utf-8"
                )
                .splitlines()
            )

            if not lines:
                reasons.append(
                    "empty_label"
                )

            for line_index, line in enumerate(
                lines,
                start=1,
            ):
                values = line.split()

                if len(values) != 5:
                    reasons.append(
                        f"line_{line_index}:"
                        "field_count"
                    )

                    continue

                try:
                    class_id = int(
                        values[0]
                    )

                    x_center = float(
                        values[1]
                    )

                    y_center = float(
                        values[2]
                    )

                    width = float(
                        values[3]
                    )

                    height = float(
                        values[4]
                    )

                except ValueError:
                    reasons.append(
                        f"line_{line_index}:"
                        "parse_error"
                    )

                    continue

                numeric_values = [
                    x_center,
                    y_center,
                    width,
                    height,
                ]

                if not all(
                    math.isfinite(value)
                    for value in numeric_values
                ):
                    reasons.append(
                        f"line_{line_index}:"
                        "non_finite"
                    )
                    continue

                if not (
                    0
                    <= class_id
                    < len(classes)
                ):
                    reasons.append(
                        f"line_{line_index}:"
                        "class_id"
                    )

                for name, value in [
                    (
                        "x_center",
                        x_center,
                    ),
                    (
                        "y_center",
                        y_center,
                    ),
                    (
                        "width",
                        width,
                    ),
                    (
                        "height",
                        height,
                    ),
                ]:
                    if not (
                        -EPSILON
                        <= value
                        <= 1.0 + EPSILON
                    ):
                        reasons.append(
                            f"line_"
                            f"{line_index}:"
                            f"{name}_range"
                        )

                if width <= 0:
                    reasons.append(
                        f"line_{line_index}:"
                        "width_zero"
                    )

                if height <= 0:
                    reasons.append(
                        f"line_{line_index}:"
                        "height_zero"
                    )

                x1 = (
                    x_center
                    - width / 2
                )

                y1 = (
                    y_center
                    - height / 2
                )

                x2 = (
                    x_center
                    + width / 2
                )

                y2 = (
                    y_center
                    + height / 2
                )

                if (
                    x1 < -EPSILON
                    or y1 < -EPSILON
                    or x2 > 1.0 + EPSILON
                    or y2 > 1.0 + EPSILON
                ):
                    reasons.append(
                        f"line_{line_index}:"
                        "bbox_outside"
                    )

        rows.append(
            {
                "file": (
                    stem + ".jpg"
                ),
                "auto_pass": (
                    len(reasons) == 0
                ),
                "reason": "|".join(
                    reasons
                ),
            }
        )

    if not rows:
        raise RuntimeError(
            "검사할 Image/Label이 없습니다."
        )

    dataframe = pd.DataFrame(
        rows
    )

    report_path = (
        paths["report"]
        / (
            f"{args.tag}"
            f"_validation.csv"
        )
    )

    dataframe.to_csv(
        report_path,
        index=False,
        encoding="utf-8-sig",
    )

    passed = int(
        dataframe[
            "auto_pass"
        ].sum()
    )

    failed = (
        len(dataframe)
        - passed
    )

    print("=" * 60)
    print(
        "YOLO Validation Completed"
    )
    print("=" * 60)

    print(
        "Total:",
        len(dataframe),
    )

    print(
        "PASS :",
        passed,
    )

    print(
        "FAIL :",
        failed,
    )

    print(
        "Report:",
        report_path,
    )


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

사람이 모든 TXT 숫자를 직접 계산하지 않고 **파일 대응·Class ID·좌표 범위·BBox 범위를 전체 Dataset에서 자동 확인**하기 위해 작성합니다.

### 의사코드로 읽어보기

```text
Image Stem 목록을 만든다
        ↓
Label Stem 목록을 만든다
        ↓
Image와 Label이 서로 짝이 맞는지 확인한다
        ↓
이미지가 읽히는지 확인한다
        ↓
Label 한 줄이 5개 값인지 확인한다
        ↓
Class ID 범위를 확인한다
        ↓
NaN / Infinity 같은 비정상 숫자가 없는지 확인한다
        ↓
작은 반올림 허용오차를 적용해 좌표 범위를 확인한다
        ↓
Width / Height가 0보다 큰지 확인한다
        ↓
BBox가 이미지 밖으로 나가지 않는지 확인한다
        ↓
문제가 없으면 auto_pass
        ↓
CSV로 저장한다
```

본 실습 Dataset을 검사합니다.

```bash
python scripts/05_validate_yolo_labels.py \
  --asset-set main \
  --tag baseline
```

정상적인 실행 결과는 다음 형태입니다.

```text
Total: 120
PASS : 120
FAIL : 0
```

`PASS 120`은 **Label 형식과 좌표가 정상이라는 뜻**입니다.

합성 장면이 현실적이라는 뜻은 아닙니다.

---

# 27. Automatic QA와 Human QA의 역할을 구분합니다

Automatic QA가 잘하는 일:

```text
파일이 있는가?
좌표를 읽을 수 있는가?
Class ID가 정상인가?
0~1 범위를 지키는가?
BBox가 이미지 안에 있는가?
```

사람이 확인해야 하는 일:

```text
객체 크기가 현실적인가?
위치가 자연스러운가?
회전 범위가 적절한가?
객체가 공중에 떠 보이지 않는가?
배경과 객체의 조명이 지나치게 다른가?
BBox가 실제 객체 경계와 시각적으로 맞는가?
```

따라서:

```text
Automatic QA
+
Human QA
```

가 함께 필요합니다.

---

# 28. Human QA 기록용 CSV를 만드는 코드를 작성합니다

Metadata와 Automatic Validation 결과를 연결합니다.

이 CSV에는 이후 사람이 직접 `human_decision`, `human_note`를 입력하므로 **한 번 작성한 검수 결과를 실수로 덮어쓰지 않는 것**도 중요합니다. 같은 Human QA 파일이 이미 있으면 기본적으로 중단하도록 작성합니다.

**파일: `scripts/06_make_qa_report.py`**

```python
from __future__ import annotations

import argparse
import sys
from pathlib import Path

import pandas as pd


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day03_common import (
    experiment_paths,
)


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--asset-set",
        default="main",
        choices=[
            "main",
            "challenge",
        ],
    )

    parser.add_argument(
        "--tag",
        default="baseline",
    )

    parser.add_argument(
        "--overwrite",
        action="store_true",
    )

    return parser.parse_args()


def main() -> None:
    args = parse_args()

    paths = experiment_paths(
        args.asset_set,
        args.tag,
    )

    metadata_path = (
        paths["report"]
        / (
            f"{args.tag}"
            f"_metadata.csv"
        )
    )

    validation_path = (
        paths["report"]
        / (
            f"{args.tag}"
            f"_validation.csv"
        )
    )

    if not metadata_path.exists():
        raise FileNotFoundError(
            metadata_path
        )

    if not validation_path.exists():
        raise FileNotFoundError(
            validation_path
        )

    output_path = (
        paths["report"]
        / (
            f"{args.tag}"
            f"_human_qa.csv"
        )
    )

    if (
        output_path.exists()
        and not args.overwrite
    ):
        raise RuntimeError(
            "기존 Human QA 파일이 있습니다.\n"
            f"{output_path}\n"
            "이미 작성한 검수 결과를 보호하기 위해 "
            "덮어쓰지 않습니다.\n"
            "정말 새로 만들 때만 --overwrite를 사용하세요."
        )

    metadata = pd.read_csv(
        metadata_path
    )

    validation = pd.read_csv(
        validation_path
    )

    merged = metadata.merge(
        validation,
        on="file",
        how="left",
        validate="one_to_one",
    )

    if merged[
        "auto_pass"
    ].isna().any():
        raise RuntimeError(
            "Metadata와 Validation의 file 대응이 "
            "완전하지 않습니다."
        )

    merged[
        "object_size_realistic"
    ] = ""

    merged[
        "position_realistic"
    ] = ""

    merged[
        "rotation_realistic"
    ] = ""

    merged[
        "edge_natural"
    ] = ""

    merged[
        "bbox_visual_ok"
    ] = ""

    merged[
        "human_decision"
    ] = ""

    merged[
        "human_note"
    ] = ""

    merged.to_csv(
        output_path,
        index=False,
        encoding="utf-8-sig",
    )

    print(
        "Saved:",
        output_path,
    )

    print(
        "Rows:",
        len(merged),
    )


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

Automatic QA 결과와 생성 Metadata를 같은 행에 연결하여 **어떤 조건에서 만들어진 이미지가 사람 검수에서 문제가 되었는지 다시 추적하기 위해** 작성합니다.

### 의사코드로 읽어보기

```text
Metadata CSV와 Validation CSV가 있는지 확인한다
        ↓
기존 Human QA CSV가 있으면 기본적으로 중단한다
        ↓
두 CSV를 읽는다
        ↓
file 이름으로 1:1 연결한다
        ↓
누락된 Validation 결과가 없는지 확인한다
        ↓
사람이 작성할 검수 열을 추가한다
        ↓
Human QA CSV로 저장한다
```

실행합니다.

```bash
python scripts/06_make_qa_report.py \
  --asset-set main \
  --tag baseline
```

결과:

```text
reports/main/baseline_human_qa.csv
```

Human QA 내용을 작성한 다음 같은 명령을 다시 실행하면 기존 기록을 보호하기 위해 중단됩니다.

정말 빈 검수표를 다시 만들 때만 다음처럼 실행합니다.

```bash
python scripts/06_make_qa_report.py \
  --asset-set main \
  --tag baseline \
  --overwrite
```

---

# 29. Human QA를 직접 수행합니다

먼저 `results/main/baseline/visualized/`의 20장을 확인합니다.

그리고 `baseline_human_qa.csv`의 해당 행에 다음 기준으로 기록합니다.

```text
object_size_realistic
→ yes / no

position_realistic
→ yes / no

rotation_realistic
→ yes / no

edge_natural
→ yes / no

bbox_visual_ok
→ yes / no

human_decision
→ approved / needs_review / rejected

human_note
→ 판단 근거
```

다음 예처럼 작성할 수 있습니다.

```text
human_decision:
needs_review

human_note:
드릴 크기는 적절하지만
실내 바닥 위에서 공중에 떠 있는 것처럼 보여
장면 현실성이 낮다.
```

오늘은 `approved` 숫자를 많이 만드는 것이 목표가 아닙니다.

**왜 승인했고 왜 거절했는지를 설명하는 것**이 더 중요합니다.

---

# PART 6. 본 실습 결과를 연결하고 데이터 편향과 Metadata를 해석합니다

---

# 30. Class별 생성 개수를 확인합니다

본 실습에서는 120장을 생성했으므로 세 Class가 각각 40장이어야 합니다.

아래 짧은 코드는 Metadata CSV를 읽고 `class_name`별 행 수를 세는 확인 코드입니다.

```bash
python - <<'PY'
import pandas as pd

df = pd.read_csv(
    "reports/main/baseline_metadata.csv"
)

print(
    df["class_name"].value_counts()
)
PY
```

의사코드로 읽으면 다음과 같습니다.

```text
Metadata CSV를 읽는다
→ class_name별 개수를 센다
→ 세 Class가 같은 수인지 확인한다
```

예상:

```text
drill               40
fire_extinguisher   40
wet_floor_sign      40
```

Class 균형을 맞추었다고 해서 모든 편향이 사라지는 것은 아닙니다.

---

# 31. Position과 Scale도 편향될 수 있습니다

합성 객체가 항상 중앙에만 나타나면 모델은 다음처럼 잘못 배울 수 있습니다.

```text
Synthetic Train
→ Object가 대부분 중앙

        ↓

Model
→ 중앙 위치와 Object 존재를
  함께 학습할 가능성
```

오늘 코드는 Background 안에서 가능한 위치를 Random하게 선택합니다.

하지만 Random이라고 해서 언제나 현실적인 것은 아닙니다.

```text
Random Position
→ 다양성 O

실제 장면에서 절대 나올 수 없는 위치
→ 현실성 X
```

따라서 Human QA에서 위치도 확인합니다.

Scale도 마찬가지입니다.

```text
너무 작음
→ 학습 신호가 약할 수 있음

현실적으로 가능한 작은 객체
→ 부족 조건 보완 가능

너무 큼
→ 실제 현장과 다른 Distribution
```

---

# 32. Metadata에서 한 장의 생성 과정을 다시 추적합니다

예를 들어 `synthetic_00023.jpg`가 이상하다고 가정합니다.

아래 코드는 Metadata에서 해당 파일 한 행만 찾아 세로 방향으로 출력하는 확인 코드입니다.

```bash
python - <<'PY'
import pandas as pd

df = pd.read_csv(
    "reports/main/baseline_metadata.csv"
)

print(
    df[
        df["file"]
        == "synthetic_00023.jpg"
    ].T
)
PY
```

의사코드로 읽으면 다음과 같습니다.

```text
Metadata CSV를 읽는다
→ file이 synthetic_00023.jpg인 행을 찾는다
→ Source와 생성 조건을 출력한다
```

다음 정보를 다시 찾을 수 있습니다.

```text
어떤 Background였는가?

어떤 Object View였는가?

Class는 무엇인가?

Scale은 얼마였는가?

Rotation은 얼마였는가?

어디에 배치되었는가?

Global Seed와 Sample Seed는 무엇인가?

같은 Sample의 x_ratio / y_ratio는 무엇인가?
```

이것이 오늘 경험하는 **Data Lineage**입니다.

---

# 33. 본 실습의 전체 흐름을 한 번 연결합니다

```text
main Source Data
        ↓
20 Background
15 RGBA Object
3 Class
        ↓
Input Check
        ↓
Asset Preview
        ↓
Single Cut-Paste
        ↓
Pixel BBox
        ↓
YOLO Label
        ↓
120장 Batch 생성
        ↓
Metadata
        ↓
BBox 재시각화
        ↓
Automatic QA
        ↓
Human QA
        ↓
Approved
Needs Review
Rejected
```

다음 문장을 자신의 실행 결과에 맞게 적어 둡니다.

```text
BBox 자동 생성이 가능한 이유:

본 실습에서 가장 부자연스러웠던 합성 조건:

Automatic QA로 찾을 수 있는 문제:

Human QA가 추가로 필요한 이유:
```

전체 기록은 Mini Challenge가 끝난 뒤 `day03_notes.md`에 한 번 정리합니다.

---

# PART 7. Mini Challenge — `challenge` 데이터로 3일차 전체 흐름을 스스로 반복합니다

---

# 34. Mini Challenge의 목표를 확인합니다

본 실습에서는 `main` 데이터를 함께 사용했습니다.

Mini Challenge에서는 본 실습과 겹치지 않는 `challenge` 데이터를 사용합니다.

```text
본 실습

main
→ Background 20
→ Object 3 Class × 5
→ 함께 실습


Mini Challenge

challenge
→ Background 10
→ Object 3 Class × 3
→ 스스로 반복
```

새로운 Python 파일을 작성하는 문제가 아닙니다.

오늘 만든 스크립트의 `--asset-set`과 `--tag`를 바꾸어 전체 Pipeline을 다시 수행합니다.

```text
입력 검사
        ↓
Asset Preview
        ↓
Single Cut-Paste
        ↓
Batch 생성
        ↓
YOLO Label
        ↓
BBox 재시각화
        ↓
Automatic QA
        ↓
Human QA
        ↓
한 변수 실험
        ↓
결론
```

Mini Challenge의 핵심 질문은 다음입니다.

> **Source Data가 달라져도 오늘의 Detection 합성데이터 생성·검수 절차를 스스로 다시 적용할 수 있는가?**

---

# 35. Challenge 요구사항 1 — 입력 데이터를 확인합니다

```bash
python scripts/00_check_day03.py \
  --asset-set challenge
```

다음 결과를 확인합니다.

```text
Backgrounds : 10

drill               : 3 / Alpha OK
fire_extinguisher   : 3 / Alpha OK
wet_floor_sign      : 3 / Alpha OK
```

다음 내용을 기록합니다.

```text
Challenge Background 수:

Challenge Object 총 수:

Class 수:

Class ID 순서:
```

---

# 36. Challenge 요구사항 2 — Source Preview를 확인합니다

```bash
python scripts/01_preview_assets.py \
  --asset-set challenge
```

결과:

```text
results/challenge/asset_preview.jpg
```

본 실습 `main`과 다른 View가 사용되고 있는지 확인합니다.

---

# 37. Challenge 요구사항 3 — 다른 Class로 Single Cut-Paste를 수행합니다

본 실습에서는 `drill`을 사용했습니다.

Challenge에서는 `wet_floor_sign`을 사용해 봅니다.

```bash
python scripts/02_cutpaste_single.py \
  --asset-set challenge \
  --background bg_001.jpg \
  --class-name wet_floor_sign \
  --object wet_floor_sign_001.png \
  --scale 0.18 \
  --rotation 15 \
  --x-ratio 0.30 \
  --y-ratio 0.65
```

다음 결과를 직접 확인합니다.

```text
results/challenge/single/
├─ single_cutpaste.jpg
├─ single_cutpaste.txt
└─ single_cutpaste_bbox.jpg
```

질문:

```text
Class ID가 2인가?

BBox가 wet_floor_sign 전체를 감싸는가?

Object의 투명 영역이 자연스럽게 처리되었는가?
```

---

# 38. Challenge 요구사항 4 — 기준 조건으로 90장을 생성합니다

본 실습과 같은 Scale·Rotation 범위를 사용하고 Source만 `challenge`로 바꿉니다.

```bash
python scripts/03_generate_cutpaste_dataset.py \
  --asset-set challenge \
  --tag rot15 \
  --count 90 \
  --seed 2026 \
  --scale-min 0.08 \
  --scale-max 0.22 \
  --rotation 15
```

세 Class가 각각 30장씩 생성되는지 확인합니다.

아래 코드는 `rot15_metadata.csv`를 읽고 Class별 생성 수를 세는 확인 코드입니다.

```bash
python - <<'PY'
import pandas as pd

df = pd.read_csv(
    "reports/challenge/rot15_metadata.csv"
)

print(
    df["class_name"].value_counts()
)
PY
```

의사코드로 읽으면 다음과 같습니다.

```text
rot15 Metadata를 읽는다
→ class_name별 개수를 센다
→ 30 / 30 / 30인지 확인한다
```


---

# 39. Challenge 요구사항 5 — Rotation 범위만 변경합니다

이번에는 다른 조건은 그대로 유지합니다.

```text
Asset Set
→ challenge 그대로

Count
→ 90 그대로

Global Seed
→ 2026 그대로

Scale
→ 0.08 ~ 0.22 그대로

Class / Background / Object / x_ratio / y_ratio
→ 같은 Sample 번호에서 동일

변경하는 것
→ Rotation 범위만 ±15° → ±30°
```

수정된 Batch 생성 코드는 같은 `seed=2026`을 사용하면 Sample별 `sample_seed`도 같아집니다.

따라서 `synthetic_00001`을 예로 들면 두 실험에서 다음 값은 같습니다.

```text
Class
Background
Object View
Scale
Rotation Unit
x_ratio
y_ratio
Sample Seed
```

실제 회전각만 다음처럼 달라집니다.

```text
rot15
rotation = rotation_unit × 15°

rot30
rotation = rotation_unit × 30°
```

회전하면 Object의 축 정렬 BBox 크기가 달라지므로 `x1, y1, x2, y2` Pixel 값까지 같아야 하는 것은 아닙니다. 대신 같은 `x_ratio / y_ratio`를 사용하여 Background 안의 **상대적 배치 조건**을 유지합니다.

실행합니다.

```bash
python scripts/03_generate_cutpaste_dataset.py \
  --asset-set challenge \
  --tag rot30 \
  --count 90 \
  --seed 2026 \
  --scale-min 0.08 \
  --scale-max 0.22 \
  --rotation 30
```

두 Metadata에서 Rotation 이외의 핵심 생성 조건이 같은지 확인합니다.

아래 코드는 두 CSV에서 다음 값이 모두 같은지 검사합니다.

```text
file
asset_set
class_id / class_name
background
object
scale
rotation_unit
x_ratio / y_ratio
global_seed
sample_seed
```

`tag`, `rotation_limit`, 실제 `rotation`, 그리고 회전 결과에 따라 달라질 수 있는 BBox 값은 비교 대상에서 제외합니다.

```bash
python - <<'PY'
import pandas as pd

a = pd.read_csv(
    "reports/challenge/rot15_metadata.csv"
)

b = pd.read_csv(
    "reports/challenge/rot30_metadata.csv"
)

columns = [
    "file",
    "asset_set",
    "class_id",
    "class_name",
    "background",
    "object",
    "scale",
    "rotation_unit",
    "x_ratio",
    "y_ratio",
    "global_seed",
    "sample_seed",
]

same = a[columns].equals(
    b[columns]
)

print(
    "Non-rotation conditions same:",
    same,
)
PY
```

의사코드로 읽으면 다음과 같습니다.

```text
rot15 Metadata를 읽는다
→ rot30 Metadata를 읽는다
→ tag / rotation_limit / 실제 rotation / BBox를 제외한다
→ Source·Class·Scale·rotation_unit·Position·Seed를 비교한다
→ 두 표의 값이 모두 같은지 확인한다
→ True이면 Rotation 범위만 달라진 한 변수 실험 조건이 유지된 것이다
```

정상이라면:

```text
Non-rotation conditions same: True
```

가 출력되어야 합니다.

이 실험의 목적은 다음과 같습니다.

```text
여러 조건을 동시에 변경
X

Rotation 범위 하나만 변경
O

        ↓

Rotation 변화가
합성 장면의 현실성과
BBox 크기에 어떤 영향을 주는지 비교
```

---

# 40. Challenge 요구사항 6 — 두 실험의 BBox와 Label을 검수합니다

먼저 `rot15`를 확인합니다.

```bash
python scripts/04_visualize_yolo_bbox.py \
  --asset-set challenge \
  --tag rot15 \
  --count 15
```

```bash
python scripts/05_validate_yolo_labels.py \
  --asset-set challenge \
  --tag rot15
```

```bash
python scripts/06_make_qa_report.py \
  --asset-set challenge \
  --tag rot15
```

그다음 `rot30`을 확인합니다.

```bash
python scripts/04_visualize_yolo_bbox.py \
  --asset-set challenge \
  --tag rot30 \
  --count 15
```

```bash
python scripts/05_validate_yolo_labels.py \
  --asset-set challenge \
  --tag rot30
```

```bash
python scripts/06_make_qa_report.py \
  --asset-set challenge \
  --tag rot30
```

두 결과를 비교합니다.

| 비교 항목 | Rotation ±15° | Rotation ±30° |
|---|---|---|
| Label Automatic QA | | |
| BBox 시각적 정확성 | | |
| 객체 방향 다양성 | | |
| 부자연스러운 장면 | | |
| 실제 수업 데이터로 더 적절한 범위 | | |

---

# 41. Challenge 요구사항 7 — `challenge_notes.md`를 작성합니다

**파일: `reports/challenge/challenge_notes.md`**

```markdown
# Day 03 Mini Challenge

## 1. Challenge Source

- Background:
- Object:
- Classes:

## 2. Single Cut-Paste

- 사용한 Background:
- 사용한 Object:
- Class ID:
- Pixel BBox:
- YOLO Label:
- 시각적 확인 결과:

## 3. Rotation ±15°

- Count:
- Seed:
- Scale:
- Automatic QA:
- Human QA에서 보인 특징:

## 4. Rotation ±30°

- Count:
- Seed:
- Scale:
- Automatic QA:
- Human QA에서 보인 특징:

## 5. 한 변수 비교

- 유지한 조건:
- 변경한 조건:
- 가장 큰 차이:

## 6. 최종 선택

- 선택한 Rotation 범위:
- 선택한 이유:

## 7. 실패 사례

- 파일:
- 문제:
- Metadata에서 찾은 조건:
- 원인 가설:

## 8. 오늘의 결론

- Cut-Paste에서 BBox를 자동 생성할 수 있는 이유:
- Automatic QA만으로 부족한 이유:
- 실제 학습데이터로 승인하기 전에 필요한 것:
```

---

# 42. Mini Challenge 완료 조건을 확인합니다

- [ ] `challenge` Background 10장을 확인했다.
- [ ] `challenge` Object 3 Class × 3장을 확인했다.
- [ ] Class ID `0 / 1 / 2` 순서를 유지했다.
- [ ] Challenge Asset Preview를 만들었다.
- [ ] `wet_floor_sign`으로 Single Cut-Paste를 수행했다.
- [ ] Pixel BBox와 YOLO Label을 확인했다.
- [ ] Rotation ±15°로 90장을 생성했다.
- [ ] Rotation ±30°로 90장을 생성했다.
- [ ] 두 실험에서 Count·Global Seed·Scale·Source·Class·Position 조건을 동일하게 유지했다.
- [ ] Metadata 비교 결과 `Non-rotation conditions same: True`를 확인했다.
- [ ] Rotation 범위만 변경했다.
- [ ] Class별 생성 수를 확인했다.
- [ ] 두 실험의 BBox를 재시각화했다.
- [ ] 두 실험의 Automatic QA를 수행했다.
- [ ] Human QA CSV를 생성했다.
- [ ] 부자연스러운 합성 사례를 한 장 이상 찾았다.
- [ ] Metadata에서 실패 사례의 생성 조건을 다시 찾았다.
- [ ] 더 적절하다고 판단한 Rotation 범위를 근거와 함께 선택했다.
- [ ] `challenge_notes.md`를 작성했다.

---

# 43. Mini Challenge를 약 3시간으로 진행합니다

| 단계 | 권장 시간 |
|---|---:|
| Challenge 입력 검사·Preview | 20분 |
| `wet_floor_sign` Single Cut-Paste | 20분 |
| Rotation ±15° Dataset 생성 | 25분 |
| Rotation ±30° Dataset 생성 | 25분 |
| 두 실험 BBox 재시각화 | 25분 |
| Automatic QA + Human QA | 30분 |
| Metadata 실패 사례 추적 | 20분 |
| 비교표·`challenge_notes.md` 작성 | 15분 |
| **합계** | **180분** |

Mini Challenge에서는 새로운 알고리즘을 추가하지 않습니다.

```text
오늘 만든 코드
        ↓
새 Source Data
        ↓
같은 Pipeline
        ↓
한 변수만 변경
        ↓
결과를 읽고 설명
```

---

# PART 8. 오늘 만든 결과를 정리하고 버전관리합니다

---

# 44. 오늘의 실험 기록을 `day03_notes.md`에 정리합니다

**파일: `day03_notes.md`**

```markdown
# Subject 11 - Day 03

## 1. 오늘의 목적

Background와 RGBA Object를 Cut-Paste하여
Detection Synthetic Image와
YOLO BBox를 함께 생성한다.

## 2. Main Source

- Background:
- drill:
- fire_extinguisher:
- wet_floor_sign:
- Class ID:

## 3. Single Cut-Paste

- Background:
- Object:
- Scale:
- Rotation:
- Position:
- Pixel BBox:
- YOLO Label:
- BBox 시각화 결과:

## 4. Main Batch

- Tag:
- Count:
- Seed:
- Scale:
- Rotation:
- Class별 생성 수:

## 5. Automatic QA

- PASS:
- FAIL:
- 주요 오류:

## 6. Human QA

- Approved:
- Needs Review:
- Rejected:
- 주요 Reject 이유:

## 7. Metadata / Data Lineage

- 추적한 파일:
- Background:
- Object:
- Scale:
- Rotation:
- Position:
- Seed:

## 8. Mini Challenge

- Rotation ±15° 결과:
- Rotation ±30° 결과:
- 더 적절한 조건:
- 선택 이유:

## 9. 오늘 이해한 핵심

- Cut-Paste:
- BBox 자동 생성:
- YOLO Label:
- Automatic QA:
- Human QA:
- Data Lineage:

## 10. 오늘의 결론

-
```

---

# 45. README에 3일차 내용을 정리합니다

`README.md`를 작성합니다.

````markdown
# Subject 11 - Day 03
## Detection Data · Cut-Paste · YOLO BBox

### Main Source

```text
day03_student_data/main

Backgrounds
20

Objects
drill               5
fire_extinguisher   5
wet_floor_sign      5
```

### Mini Challenge Source

```text
day03_student_data/challenge

Backgrounds
10

Objects
3 Classes × 3
```

### Class ID

```text
0 drill
1 fire_extinguisher
2 wet_floor_sign
```

### Day 03 Pipeline

```text
Background
+
RGBA Object
        ↓
Cut-Paste
        ↓
Position / Scale / Rotation
        ↓
Pixel BBox
        ↓
YOLO Label
        ↓
BBox Visualization
        ↓
Automatic QA
        ↓
Human QA
        ↓
Metadata
```

### Main Scripts

```text
scripts/
├─ 00_check_day03.py
├─ 01_preview_assets.py
├─ 02_cutpaste_single.py
├─ 03_generate_cutpaste_dataset.py
├─ 04_visualize_yolo_bbox.py
├─ 05_validate_yolo_labels.py
└─ 06_make_qa_report.py
```

### Main Run

```bash
python scripts/00_check_day03.py --asset-set main

python scripts/01_preview_assets.py --asset-set main

python scripts/02_cutpaste_single.py \
  --asset-set main \
  --class-name drill \
  --object drill_001.png

python scripts/03_generate_cutpaste_dataset.py \
  --asset-set main \
  --tag baseline \
  --count 120 \
  --seed 42 \
  --scale-min 0.08 \
  --scale-max 0.22 \
  --rotation 15

python scripts/04_visualize_yolo_bbox.py \
  --asset-set main \
  --tag baseline \
  --count 20

python scripts/05_validate_yolo_labels.py \
  --asset-set main \
  --tag baseline

python scripts/06_make_qa_report.py \
  --asset-set main \
  --tag baseline
```

### 핵심 원칙

```text
생성됨
≠
학습 승인

생성
→ Label 재시각화
→ Automatic QA
→ Human QA
→ Approved 판단
```

### Next

Day 04에서는 2D Cut-Paste에서 확장하여
3D Scene의 Camera·Light·Object 조건을 바꾸고
RGB·BBox·Mask·Depth 같은 Ground Truth를
자동 생성하는 방법을 실습합니다.
````

README에는 코드 전체를 다시 복사하지 않습니다.

실행 순서와 핵심 결과만 빠르게 확인할 수 있도록 정리합니다.

---

# 46. Git에 포함할 파일과 제외할 파일을 구분합니다

오늘은 Source Image와 대량 생성 Dataset을 Git에 올리지 않는 것을 기본으로 합니다.

`.gitignore`를 작성합니다.

```gitignore
# Python
__pycache__/
*.pyc

# Editor
.vscode/

# Day03 Source Data
data/day03_student_data/

# Generated Images / Labels
dataset/

# Large Visual Results
results/

# OS
.DS_Store
Thumbs.db
```

`reports/`의 작은 CSV와 Markdown 기록은 교육용 데이터만 사용했다면 Commit할 수 있습니다.

기업 데이터나 반출 제한 정보가 포함되면 `reports/`도 저장소 정책에 맞게 제외합니다.

---

# 47. Git 저장소는 다시 만들지 않습니다

교과 11 Root로 이동합니다.

```bash
cd ~/ai_vision/subject11_synthetic_data
```

상태를 확인합니다.

```bash
git status
```

3일차 코드와 문서를 추가합니다.

```bash
git add day03_detection_cutpaste
```

다시 확인합니다.

```bash
git status
```

다음 대용량 폴더가 Stage에 들어가지 않았는지 확인합니다.

```text
data/day03_student_data/
dataset/
results/
```

Commit합니다.

```bash
git commit -m "feat: add day03 cut-paste detection lab"
```

기록을 확인합니다.

```bash
git log --oneline --decorate -5
```

이미 Remote가 연결되어 있고 Push가 허용되는 환경이라면 기존 Remote를 그대로 사용합니다.

```bash
git remote -v
```

```bash
git push
```

새 Git 저장소를 다시 만들거나 새로운 Remote를 중복 추가하지 않습니다.

---

# 48. 하루가 끝났을 때 자가 체크합니다

| 확인사항 | 완료 |
|---|:---:|
| 1~2일차와 같은 교과 11 프로젝트를 이어서 사용했다 | □ |
| 1일차 `.venv`를 재사용했다 | □ |
| `day03_student_data`를 올바른 위치에 배치했다 | □ |
| `main`과 `challenge` 역할을 구분했다 | □ |
| 본 실습 Background 20장을 확인했다 | □ |
| 본 실습 Object 3 Class × 5장을 확인했다 | □ |
| Class ID `0 / 1 / 2`를 고정했다 | □ |
| Object PNG의 Alpha Channel을 확인했다 | □ |
| Asset Preview를 만들었다 | □ |
| Single Cut-Paste를 성공했다 | □ |
| Pixel BBox를 확인했다 | □ |
| YOLO Label을 생성했다 | □ |
| 6장 Smoke Test를 수행했다 | □ |
| Main Synthetic Dataset 120장을 생성했다 | □ |
| JPG와 TXT 파일명이 일치한다 | □ |
| Metadata를 생성했다 | □ |
| Class별 생성 수를 확인했다 | □ |
| BBox를 다시 이미지에 그려 확인했다 | □ |
| 전체 Label Automatic QA를 수행했다 | □ |
| Human QA CSV를 만들고 결과를 확인했다 | □ |
| 실패 사례를 Metadata에서 다시 추적했다 | □ |
| Mini Challenge에서 Challenge Source만 사용했다 | □ |
| Rotation 하나만 변경하여 비교했다 | □ |
| `challenge_notes.md`를 작성했다 | □ |
| `day03_notes.md`를 작성했다 | □ |
| README를 작성했다 | □ |
| Git Commit을 완료했다 | □ |

---

# 49. 3일차 최종 정리

오늘은 단순히 PNG를 Background 위에 붙여본 것이 아닙니다.

다음 전체 과정을 경험했습니다.

```text
Prepared Source Data

Background
+
Transparent RGBA Object
        ↓
Class ID 고정
        ↓
Alpha 확인
        ↓
Single Cut-Paste
        ↓
Position
Scale
Rotation
        ↓
Batch Synthetic Image
        ↓
Pixel BBox 자동 계산
        ↓
YOLO Label 자동 생성
        ↓
Metadata
        ↓
BBox 재시각화
        ↓
Automatic QA
        ↓
Human QA
        ↓
Approved
Needs Review
Rejected
```

오늘 반드시 기억해야 할 세 가지는 다음입니다.

## ① 합성과 Label은 함께 만들 수 있습니다

```text
프로그램이
객체를 어디에 붙였는지 알고 있다
        ↓
정답 BBox도 자동 계산할 수 있다
```

## ② 자동 생성된 Label도 검수해야 합니다

```text
코드가 자동 생성했다
        ↓
무조건 정답
X

코드가 자동 생성했다
        ↓
다시 시각화
        ↓
좌표 자동 검사
        ↓
사람 확인
O
```

## ③ 생성 가능한 데이터와 사용할 데이터는 다릅니다

```text
Synthetic Image 생성 성공
        ↓
학습에 바로 사용
X

Synthetic Image
        ↓
QA
        ↓
현실성 판단
        ↓
Approved 데이터만
학습 후보
```

---

# 다음 시간 연결

3일차에서는 이미 준비된 2D Background와 Object PNG를 사용했습니다.

```text
2D Background
+
2D Object
        ↓
Cut-Paste
        ↓
Image + BBox
```

4일차에서는 한 단계 더 확장합니다.

```text
3D Scene
        ↓
Camera
Light
Object Position
Object Rotation
        ↓
Rendering
        ↓
RGB
BBox
Mask
Depth
```

즉 오늘은:

```text
2D Source를 합성하여
Detection Label까지 자동 생성
```

했다면 다음 시간에는:

```text
3D 가상환경 자체를 바꾸면서
여러 종류의 Ground Truth를 자동 생성
```

하는 방법으로 확장합니다.
