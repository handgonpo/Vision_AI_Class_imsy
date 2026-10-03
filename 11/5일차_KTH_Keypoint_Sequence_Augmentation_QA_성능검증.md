## 오늘의 핵심 질문

> **사람의 움직임을 영상이 아니라 Keypoint Sequence로 표현한 뒤 위치·크기·좌표 오차·동작 속도를 보강하면, 같은 Validation/Test에서 실제 Action Recognition 성능이 달라질까?**

1~4일차에는 이미지와 3D 장면을 중심으로 합성·보강 데이터를 만들었습니다.

```text
1일차
Image Augmentation + VAE

2일차
DCGAN + WGAN-GP + Diffusion

3일차
Cut-Paste + YOLO BBox

4일차
3D Rendering
→ RGB + BBox + Mask + Depth
```

5일차에는 데이터 형태가 **사람의 움직임을 나타내는 시계열 데이터**로 바뀝니다.

![[Pasted image 20260908150456.png]]

5일차에는 사람의 동작 영상을 **관절 좌표가 시간순으로 이어진 Keypoint Sequence**로 바꾸고, 위치·크기·노이즈·속도 변화를 적용해 새로운 학습데이터를 만듭니다.  

만든 보강 데이터는 **Automatic QA와 Human QA로 정상 여부를 확인한 뒤**, 통과한 데이터만 원본 Train에 추가합니다.  

마지막에는 **원본만 학습한 모델과 보강 데이터를 추가한 모델을 같은 Validation/Test로 비교**하여 실제로 성능이 좋아졌는지 확인합니다.

```text
KTH Action Video
        ↓
YOLO Pose
        ↓
Frame별 COCO17 Keypoint
        ↓
Keypoint Sequence
        ↓
Translation
Scale
Jitter
Time Warp
        ↓
Sequence QA
        ↓
Original Train
VS
Original + Approved Augmented Train
        ↓
같은 Validation / Test
        ↓
성능 비교
```

오늘의 목적은 Keypoint JSON을 많이 만드는 것이 아닙니다.

다음 질문에 답할 수 있도록 **원본 영상 → Keypoint → Sequence Augmentation → QA → 성능검증**의 전체 흐름을 직접 실행하는 것이 목표입니다.

```text
동작 영상은 어떤 Sequence로 바뀌는가?
        ↓
어떤 변형이 동작 의미를 유지하는가?
        ↓
어떤 보강 데이터가 정상적인가?
        ↓
Train에만 보강을 적용했는가?
        ↓
같은 Validation/Test에서 성능이 달라졌는가?
        ↓
보강 데이터가 실제로 도움이 되었는가?
```

---

# 오늘의 수업 목표

수업이 끝나면 다음 내용을 설명하고 직접 실행할 수 있어야 합니다.

- 1~4일차와 같은 교과 11 프로젝트와 1일차 `.venv`를 그대로 이어서 사용할 수 있습니다.
- 제공된 `day05_student_data`의 `main`과 `challenge` 역할을 구분할 수 있습니다.
- KTH 수업용 파일명 `S11_boxing_T01.avi`에서 Subject·Action·조건 ID를 읽을 수 있습니다.
- `boxing`, `handwaving`, `handclapping`의 세 Action을 구분할 수 있습니다.
- Train·Validation·Test가 이미 Subject 단위로 분리된 이유를 설명할 수 있습니다.
- 같은 Subject가 Train과 Validation/Test에 섞이면 안 되는 이유를 설명할 수 있습니다.
- KTH AVI 영상을 OpenCV로 읽고 기본 정보를 확인할 수 있습니다.
- YOLO Pose로 한 영상의 사람을 추론하고 COCO 17 Keypoint를 추출할 수 있습니다.
- Pixel Keypoint를 0~1 정규화 좌표로 저장할 수 있습니다.
- 여러 Frame의 Keypoint가 하나의 Sequence가 되는 구조를 설명할 수 있습니다.
- 원본 영상과 Keypoint JSON을 다시 연결하여 Skeleton을 확인할 수 있습니다.
- Translation·Scale·Jitter·Time Warp의 의미를 설명할 수 있습니다.
- Train Sequence에만 보강을 적용할 수 있습니다.
- 생성된 Sequence의 Source·Method·Parameter·Seed를 Metadata로 기록할 수 있습니다.
- JSON 구조·NaN·좌표 범위·Keypoint 수·유효 Frame 비율을 Automatic QA로 확인할 수 있습니다.
- Human QA에서 동작 의미가 유지되는지 직접 판단할 수 있습니다.
- Variable-length Sequence를 고정 길이 통계 Feature로 바꾸는 이유를 설명할 수 있습니다.
- Original Train으로 Baseline 모델을 만들 수 있습니다.
- Original Train + Approved Augmented Train으로 Augmented 모델을 만들 수 있습니다.
- 두 모델을 동일한 Validation/Test에서 비교할 수 있습니다.
- Accuracy와 Confusion Matrix를 보고 보강 효과를 해석할 수 있습니다.
- Mini Challenge에서 본 실습과 겹치지 않는 Subject 데이터로 같은 Pipeline을 스스로 반복할 수 있습니다.
- Mini Challenge에서 Jitter 강도 하나만 변경하고, Validation으로 더 적절한 조건을 선택한 뒤 선택된 Final 조건만 Test에서 평가할 수 있습니다.
- 1~5일차의 합성데이터 방법을 하나의 흐름으로 정리할 수 있습니다.
- 오늘 만든 코드·결과·실패 사례를 README와 Git에 정리할 수 있습니다.

---

# 오늘 수업의 전체 흐름

![[Pasted image 20260908150902.png]]

5일차에는 사람의 동작 영상을 **YOLO Pose로 관절 좌표 시계열 데이터로 변환**하고, 위치·크기·노이즈·속도 변화를 적용해 보강 데이터를 만듭니다.  

그다음 원본과 보강 데이터를 **QA로 검수하여 정상 데이터만 선택**하고, 원본만 학습한 모델과 보강 데이터를 추가한 모델을 같은 Validation/Test에서 비교합니다.  

마지막에는 Mini Challenge에서 **Jitter 값 하나만 바꾸어 성능 차이를 확인**하며, 어떤 보강이 실제로 도움이 되었는지 해석합니다.

```text
4일차 프로젝트 확인
        ↓
교과 11 Root 이동
        ↓
5일차 폴더 생성
        ↓
1일차 .venv 재사용
        ↓
day05_student_data 배치
        ↓

[본 실습 Source 확인]

main/train
24 Videos
S11 S12 S13 S14

main/val
12 Videos
S19 S20

main/test
12 Videos
S02 S03

Action
boxing
handwaving
handclapping
        ↓
입력 데이터 자동 검사
        ↓
Subject Leakage 확인
        ↓

[실험 A — Video → Keypoint]

대표 3개 영상
        ↓
YOLO Pose
        ↓
COCO17 Keypoint JSON
        ↓
Skeleton Preview
        ↓
Main 48개 전체 Keypoint 추출
        ↓

[실험 B — Sequence Augmentation]

Main Train 24 Sequence
        ↓
Translation
Scale
Jitter
Time Warp Fast
Time Warp Slow
        ↓
120 Augmented Sequence
        ↓
Metadata
        ↓

[QA]

Original + Augmented
        ↓
Automatic QA
        ↓
Skeleton Preview
        ↓
Human QA
        ↓
Approved / Needs Review / Rejected
        ↓

[성능검증]

Original Train
        ↓
Baseline

Original Train
+
Approved Augmented Train
        ↓
Augmented

        ↓
Same Validation
Same Test
        ↓
Accuracy
Confusion Matrix
        ↓
보강 효과 해석
        ↓

[Mini Challenge]

challenge
Train 12 / Val 6 / Test 6
        ↓
Keypoint 추출
        ↓
Jitter 0.005
VS
Jitter 0.015
        ↓
다른 조건 동일
        ↓
Automatic QA + Human QA
        ↓
같은 Validation에서 Candidate 비교
        ↓
Validation Accuracy + Confusion Matrix
        ↓
더 적절한 Jitter 선택
        ↓
Final Jitter Freeze
        ↓
선택된 조건만 Test S05 최종 평가
        ↓
Final Test Accuracy + Confusion Matrix
        ↓
한 변수 실험 해석
        ↓

README · Git
        ↓
교과 11 최종 정리
```

> **오늘의 데이터 운영 원칙**
>
> `main`은 본 실습에만 사용합니다.
>
> `challenge`는 Mini Challenge에만 사용합니다.
>
> 제공된 Train·Validation·Test는 다시 Random Split하지 않습니다.
>
> **Augmentation은 Train Sequence에만 적용합니다.**
>
> Validation과 Test는 Original Sequence를 그대로 유지합니다.
>
> Baseline과 Augmented 모델은 같은 Original Validation/Test를 사용합니다.
>
> Main의 사전 정의된 Pipeline은 최종 보고 단계에서 Validation과 Test를 함께 확인하되, Test 결과를 보고 Augmentation Parameter를 다시 조정하지 않습니다.
>
> Mini Challenge의 Jitter 0.005 vs 0.015 선택은 **Validation으로만** 수행하고, Test는 선택된 Final Jitter를 Freeze한 뒤 마지막에 한 번 평가합니다.

---

# 오늘 사용할 실제 데이터

## 1. 제공되는 5일차 학생용 데이터

수업 시작 전에 다음 폴더가 제공됩니다.

```text
day05_student_data/
│
├─ main/
│  ├─ train/
│  │  ├─ S11_boxing_T01.avi
│  │  ├─ S11_boxing_T02.avi
│  │  ├─ S11_handwaving_T01.avi
│  │  ├─ S11_handwaving_T02.avi
│  │  ├─ S11_handclapping_T01.avi
│  │  ├─ S11_handclapping_T02.avi
│  │  ├─ ...
│  │  └─ S14_handclapping_T02.avi
│  │
│  ├─ val/
│  │  ├─ S19_...
│  │  └─ S20_...
│  │
│  └─ test/
│     ├─ S02_...
│     └─ S03_...
│
├─ challenge/
│  ├─ train/
│  │  ├─ S15_...
│  │  └─ S16_...
│  ├─ val/
│  │  └─ S21_...
│  └─ test/
│     └─ S05_...
│
├─ classes.txt
├─ README_DATA.txt
├─ split_summary.csv
└─ video_manifest.csv
```

오늘은 원본 KTH 전체 폴더를 다시 다운로드하거나 다시 선별하지 않습니다.

이미 수업용으로 정리된 `day05_student_data`만 사용합니다.

## 2. 본 실습 `main`

```text
Train
S11 S12 S13 S14
4 Subjects × 3 Actions × 2 조건
= 24 Videos

Validation
S19 S20
2 Subjects × 3 Actions × 2 조건
= 12 Videos

Test
S02 S03
2 Subjects × 3 Actions × 2 조건
= 12 Videos
```

총 48개 영상입니다.

## 3. Mini Challenge `challenge`

```text
Train
S15 S16
= 12 Videos

Validation
S21
= 6 Videos

Test
S05
= 6 Videos

Total
= 24 Videos
```

본 실습과 다른 Subject를 사용하므로 같은 코드를 다시 사용해도 실제 입력 데이터는 달라집니다.

## 4. Action Class는 세 가지입니다

`classes.txt`의 수업 기준은 다음과 같습니다.

```text
0 boxing
1 handwaving
2 handclapping
```

오늘의 Class ID는 중간에 바꾸지 않습니다.

## 5. 파일명을 읽는 방법

```text
S11_boxing_T01.avi
```

```text
S11
→ Subject 11

boxing
→ Action

T01
→ 수업용 조건 ID 01
```

`T01`, `T02`는 원본 KTH의 `d1`, `d2`를 수업용 파일명에서 구분하기 위해 사용한 ID입니다.

## 6. 영상 특성과 오늘의 보강을 연결합니다

현재 제공된 영상은 흑백이며 영상에 따라 사람의 화면 크기와 촬영 조건이 다르게 보일 수 있습니다.

```text
사람 위치 차이
→ Translation

사람 크기 차이
→ Scale

Pose 좌표의 작은 오차
→ Jitter

동작 속도 차이
→ Time Warp
```

실제 영상에서 변화가 보인다는 이유만으로 보강 범위를 크게 잡지는 않습니다.

**동작 의미가 유지되는 범위**에서만 변형합니다.

---

# 오늘 배울 기술 스택

| 기술 | 오늘 하는 일 | 쉽게 말하면 |
|---|---|---|
| Python | 전체 Sequence Pipeline 작성 | 영상부터 성능비교까지 실행 |
| pathlib | Main·Challenge·Split 경로 관리 | 데이터 위치를 안전하게 연결 |
| OpenCV | AVI 읽기·Skeleton Video 저장 | 영상 입출력 |
| Ultralytics YOLO Pose | 사람 Pose 추출 | Frame에서 COCO17 관절 찾기 |
| COCO17 Keypoint | 17개 관절 표현 | 사람 자세를 좌표로 저장 |
| NumPy | Sequence·Interpolation·Augmentation | 시계열 좌표 계산 |
| Pandas | Metadata·QA·Feature CSV | 결과를 표로 저장 |
| Translation | 전체 관절 위치 이동 | 화면 위치 변화 |
| Scale | 관절 크기 변화 | 사람 크기·거리 변화 |
| Jitter | 좌표에 작은 Noise | Pose 오차·미세 자세 변화 |
| Time Warp | Frame 길이 변화 | 빠른/느린 동작 |
| Subject-wise Split | 사람 단위 분리 | 같은 사람의 정보 누수 방지 |
| Automatic QA | 구조·범위 자동 검사 | 잘못된 JSON 선별 |
| Human QA | 동작 의미 검수 | 사람이 현실성 확인 |
| Statistical Sequence Feature | 가변 길이 Sequence를 고정 Feature로 변환 | 모델 입력 만들기 |
| Random Forest | 교육용 Action Classifier | Baseline과 Augmented 비교 |
| Accuracy / Confusion Matrix | 보강 효과 확인 | 어떤 Action이 맞고 틀렸는지 비교 |
| Metadata | Source·Method·Parameter·Seed 기록 | Data Lineage |
| Git | 코드와 실험 기록 관리 | 재현 가능한 수업 결과 저장 |

---

# 오늘의 8시간 운영 구성

| 단계 | 권장 시간 | 내용 |
|---|---:|---|
| 본 실습 1 | 45분 | 4일차 연결·폴더·환경·데이터 검사 |
| 본 실습 2 | 50분 | 대표 영상 3개 → Pose → Keypoint → Preview |
| 본 실습 3 | 60분 | Main 48개 전체 Keypoint 추출 |
| 본 실습 4 | 60분 | Train Sequence Augmentation·Metadata |
| 본 실습 5 | 45분 | Automatic QA·Human QA |
| 본 실습 6 | 40분 | Feature 구성·Baseline vs Augmented |
| Mini Challenge | 180분 | Challenge 데이터로 한 변수 실험 |
| **합계** | **480분** | **8시간** |

---

# PART 1. 4일차 프로젝트에서 5일차를 이어서 시작합니다

# 1. 교과 11의 5일을 연결합니다

```text
Day 01
Image → 일반 Augmentation + VAE

Day 02
Image → GAN / WGAN-GP / Diffusion

Day 03
Detection → Cut-Paste + BBox

Day 04
3D → Rendering + Mask + Depth

Day 05
Motion → Keypoint Sequence Augmentation
```

5일차에는 5일 동안 반복한 다음 원칙을 실제 성능비교까지 연결합니다.

```text
Real Data
→ 부족한 조건 찾기
→ Synthetic / Augmented Data
→ QA
→ Metadata
→ Train에 추가
→ Same Validation / Test
→ 성능 비교
```

# 2. 교과 11 프로젝트 Root로 이동합니다

```bash
cd ~/ai_vision/subject11_synthetic_data
```

확인합니다.

```bash
ls
```

```text
day01_image_aug_vae
day02_generative_models
day03_detection_cutpaste
day04_3d_segmentation
```

# 3. 5일차 폴더를 만듭니다

```bash
mkdir -p day05_keypoint_sequence
cd day05_keypoint_sequence
```

```bash
mkdir -p \
src \
scripts \
data \
data/keypoints \
data/features \
models \
results/main \
results/challenge \
reports/main \
reports/challenge
```

```bash
touch requirements_day05.txt
touch src/day05_common.py

touch scripts/00_check_day05.py
touch scripts/01_extract_keypoints.py
touch scripts/02_visualize_keypoints.py
touch scripts/03_augment_keypoints.py
touch scripts/04_qa_keypoints.py
touch scripts/05_build_features.py
touch scripts/06_compare_baseline_augmented.py

touch day05_notes.md
touch final_reflection.md
touch README.md
touch .gitignore
```

```bash
tree -L 2
```

예상:

```text
day05_keypoint_sequence/
├─ data/
│  ├─ features/
│  └─ keypoints/
├─ models/
├─ reports/
├─ results/
├─ scripts/
├─ src/
├─ requirements_day05.txt
├─ day05_notes.md
├─ final_reflection.md
├─ README.md
└─ .gitignore
```

# 4. 제공된 `day05_student_data`를 배치합니다

현재 준비한 `day05_student_data` 폴더 전체를 다음 위치에 복사합니다.

```text
day05_keypoint_sequence/
└─ data/
   └─ day05_student_data/
```

확인:

```bash
tree data/day05_student_data -L 3
```

오늘은 `boxing`, `handwaving`, `handclapping` 원본 전체 폴더를 사용하지 않습니다.

# 5. 1일차 가상환경을 그대로 사용합니다

```bash
source ../day01_image_aug_vae/.venv/bin/activate
```

```bash
which python
```

# 6. 5일차에 필요한 라이브러리를 확인합니다

`requirements_day05.txt`:

```text
ultralytics
opencv-python
numpy
pandas
scikit-learn
joblib
```

```bash
pip install -r requirements_day05.txt
```

```bash
python -c "import cv2, ultralytics, sklearn, joblib; print('Day05 libraries OK')"
```

```bash
pip freeze > reports/environment_freeze_day05.txt
```

# 7. YOLO Pose 모델을 준비합니다

교과 10에서 사용한 모델 또는 수업과 함께 제공된 모델을 다음 위치에 둡니다.

```text
models/yolo11n-pose.pt
```

확인:

```bash
ls -lh models/yolo11n-pose.pt
```

---

# 8. Main과 Challenge의 공통 경로 코드를 작성합니다

코드를 작성하기 전에 경로를 먼저 정리합니다.

```text
Input Video
data/day05_student_data/{main|challenge}/{train|val|test}

Original Keypoint
data/keypoints/{main|challenge}/original/{train|val|test}

Augmented Keypoint
data/keypoints/{main|challenge}/augmented/{tag}

Feature
data/features/{main|challenge}/{tag}

Report
reports/{main|challenge}
```

**파일: `src/day05_common.py`**

```python
from __future__ import annotations

import re
from pathlib import Path


ROOT = Path(__file__).resolve().parents[1]

DATA_ROOT = ROOT / "data" / "day05_student_data"
KEYPOINT_ROOT = ROOT / "data" / "keypoints"
FEATURE_ROOT = ROOT / "data" / "features"
REPORT_ROOT = ROOT / "reports"
RESULT_ROOT = ROOT / "results"
MODEL_ROOT = ROOT / "models"


VIDEO_PATTERN = re.compile(
    r"^(?P<subject>S\d{2})_"
    r"(?P<action>[A-Za-z0-9\-]+)_"
    r"(?P<take>T\d{2})\.avi$"
)


def read_classes() -> dict[int, str]:
    path = DATA_ROOT / "classes.txt"

    if not path.exists():
        raise FileNotFoundError(path)

    mapping = {}

    for index, raw_line in enumerate(
        path.read_text(encoding="utf-8").splitlines()
    ):
        line = raw_line.strip()

        if not line:
            continue

        parts = line.split(maxsplit=1)

        if len(parts) == 2 and parts[0].isdigit():
            class_id = int(parts[0])
            class_name = parts[1].strip()
        else:
            class_id = index
            class_name = line

        mapping[class_id] = class_name

    if not mapping:
        raise RuntimeError("classes.txt가 비어 있습니다.")

    return mapping


def parse_video_name(path: Path) -> dict[str, str]:
    match = VIDEO_PATTERN.match(path.name)

    if match is None:
        raise ValueError(
            f"수업용 파일명 형식이 아닙니다: {path.name}"
        )

    return match.groupdict()


def video_dir(asset_set: str, split: str) -> Path:
    return DATA_ROOT / asset_set / split


def list_videos(asset_set: str, split: str) -> list[Path]:
    directory = video_dir(asset_set, split)

    if not directory.exists():
        return []

    return sorted(directory.glob("*.avi"))


def original_keypoint_dir(
    asset_set: str,
    split: str,
) -> Path:
    path = KEYPOINT_ROOT / asset_set / "original" / split
    path.mkdir(parents=True, exist_ok=True)
    return path


def augmented_keypoint_dir(
    asset_set: str,
    tag: str,
) -> Path:
    path = KEYPOINT_ROOT / asset_set / "augmented" / tag
    path.mkdir(parents=True, exist_ok=True)
    return path


def feature_dir(asset_set: str, tag: str) -> Path:
    path = FEATURE_ROOT / asset_set / tag
    path.mkdir(parents=True, exist_ok=True)
    return path


def report_dir(asset_set: str) -> Path:
    path = REPORT_ROOT / asset_set
    path.mkdir(parents=True, exist_ok=True)
    return path


def result_dir(asset_set: str) -> Path:
    path = RESULT_ROOT / asset_set
    path.mkdir(parents=True, exist_ok=True)
    return path
```

### 이 코드는 왜 작성하나요?

본 실습과 Mini Challenge에서 같은 Script를 재사용하기 위해 작성합니다.

### 의사코드로 읽어보기

```text
5일차 Root를 찾는다
→ day05_student_data 위치를 만든다
→ Keypoint / Feature / Report / Result 경로를 만든다
→ classes.txt를 읽는다
→ 파일명에서 Subject / Action / T ID를 읽는다
→ main/challenge와 split별 경로 함수를 만든다
```


# 9. 실제 수업 데이터가 정확한지 자동으로 검사합니다

코드를 작성하기 전에 오늘 확인해야 할 조건을 정합니다.

```text
Main
Train 24
Val 12
Test 12

Challenge
Train 12
Val 6
Test 6

Class
boxing
handwaving
handclapping

Train / Val / Test
Subject 중복 없음

AVI
OpenCV로 열 수 있음
FPS > 0
Frame > 0
Width / Height > 0
```

**파일: `scripts/00_check_day05.py`**

```python
from __future__ import annotations

import argparse
import sys
from pathlib import Path

import cv2


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day05_common import (
    list_videos,
    parse_video_name,
    read_classes,
)


EXPECTED = {
    "main": {
        "train": {
            "count": 24,
            "subjects": {"S11", "S12", "S13", "S14"},
        },
        "val": {
            "count": 12,
            "subjects": {"S19", "S20"},
        },
        "test": {
            "count": 12,
            "subjects": {"S02", "S03"},
        },
    },
    "challenge": {
        "train": {
            "count": 12,
            "subjects": {"S15", "S16"},
        },
        "val": {
            "count": 6,
            "subjects": {"S21"},
        },
        "test": {
            "count": 6,
            "subjects": {"S05"},
        },
    },
}


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--asset-set",
        default="all",
        choices=["main", "challenge", "all"],
    )

    return parser.parse_args()


def check_video(
    path: Path,
) -> tuple[float, int, int, int]:
    capture = cv2.VideoCapture(str(path))

    if not capture.isOpened():
        raise RuntimeError(
            f"영상을 열 수 없습니다: {path}"
        )

    fps = float(
        capture.get(cv2.CAP_PROP_FPS)
    )

    frames = int(
        capture.get(cv2.CAP_PROP_FRAME_COUNT)
    )

    width = int(
        capture.get(cv2.CAP_PROP_FRAME_WIDTH)
    )

    height = int(
        capture.get(cv2.CAP_PROP_FRAME_HEIGHT)
    )

    capture.release()

    if (
        fps <= 0
        or frames <= 0
        or width <= 0
        or height <= 0
    ):
        raise RuntimeError(
            f"영상 정보가 비정상입니다: {path.name}"
        )

    return fps, frames, width, height


def check_asset_set(
    asset_set: str,
) -> None:
    classes = read_classes()

    expected_classes = {
        0: "boxing",
        1: "handwaving",
        2: "handclapping",
    }

    if classes != expected_classes:
        raise RuntimeError(
            "classes.txt가 수업 기준과 다릅니다.\n"
            f"현재: {classes}\n"
            f"기준: {expected_classes}"
        )

    split_subjects = {}

    print("=" * 68)
    print(f"Day05 Data Check - {asset_set}")
    print("=" * 68)

    for split in ["train", "val", "test"]:
        videos = list_videos(
            asset_set,
            split,
        )

        expected = EXPECTED[
            asset_set
        ][split]

        if len(videos) != expected["count"]:
            raise RuntimeError(
                f"{asset_set}/{split} 수가 "
                "수업 기준과 다릅니다.\n"
                f"기대: {expected['count']}\n"
                f"현재: {len(videos)}"
            )

        subjects = set()
        actions = set()
        takes = set()

        for path in videos:
            info = parse_video_name(path)

            subjects.add(
                info["subject"]
            )

            actions.add(
                info["action"]
            )

            takes.add(
                info["take"]
            )

            check_video(path)

        if subjects != expected["subjects"]:
            raise RuntimeError(
                f"{asset_set}/{split} Subject가 "
                "수업 기준과 다릅니다.\n"
                f"현재: {sorted(subjects)}"
            )

        if actions != {
            "boxing",
            "handwaving",
            "handclapping",
        }:
            raise RuntimeError(
                f"{asset_set}/{split} Action 구성이 "
                "수업 기준과 다릅니다."
            )

        if takes != {"T01", "T02"}:
            raise RuntimeError(
                f"{asset_set}/{split} T ID가 "
                "수업 기준과 다릅니다."
            )

        split_subjects[split] = subjects

        fps, _, width, height = (
            check_video(videos[0])
        )

        print(
            f"{split:5s}: "
            f"{len(videos):2d} videos / "
            f"Subjects {sorted(subjects)} / "
            f"FPS {fps:.2f} / "
            f"Size {width}x{height}"
        )

    if (
        split_subjects["train"] & split_subjects["val"]
        or split_subjects["train"] & split_subjects["test"]
        or split_subjects["val"] & split_subjects["test"]
    ):
        raise RuntimeError(
            f"{asset_set}에서 Subject Leakage가 있습니다."
        )

    print("Subject Leakage: NONE")


def main() -> None:
    args = parse_args()

    asset_sets = (
        ["main", "challenge"]
        if args.asset_set == "all"
        else [args.asset_set]
    )

    for asset_set in asset_sets:
        check_asset_set(asset_set)

    print("=" * 68)
    print("Day05 data check: READY")


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

5일차에서는 데이터 분할 자체가 성능검증의 핵심입니다.

영상 파일만 존재하는지 확인하는 것으로 끝내지 않고, **Subject가 Split 사이에 섞이지 않았는지까지 확인**합니다.

### 의사코드로 읽어보기

```text
main / challenge를 선택한다
→ classes.txt를 확인한다
→ Train / Val / Test 영상 수를 확인한다
→ 파일명에서 Subject / Action / T ID를 읽는다
→ AVI를 OpenCV로 열어본다
→ FPS / Frame / Size가 정상인지 확인한다
→ Split별 Subject를 모은다
→ Train / Val / Test Subject가 겹치는지 확인한다
→ 모두 정상이면 READY
```

실행합니다.

```bash
python scripts/00_check_day05.py \
  --asset-set all
```

---

# PART 2. KTH 영상을 Keypoint Sequence로 변환합니다

# 10. 영상과 Keypoint Sequence의 차이를 이해합니다

한 Frame의 입력은 이미지입니다.

```text
Frame
→ Width × Height × RGB
```

Pose 모델을 통과하면 다음처럼 바뀝니다.

```text
Frame
        ↓
YOLO Pose
        ↓
17 Keypoints
        ↓
[x, y, confidence]
```

영상은 여러 Frame으로 구성되므로:

```text
Frame 000 → 17 Keypoints
Frame 001 → 17 Keypoints
Frame 002 → 17 Keypoints
...
```

가 하나의 동작 Sequence가 됩니다.

# 11. COCO 17 Keypoint를 확인합니다

```text
0  nose
1  left_eye
2  right_eye
3  left_ear
4  right_ear
5  left_shoulder
6  right_shoulder
7  left_elbow
8  right_elbow
9  left_wrist
10 right_wrist
11 left_hip
12 right_hip
13 left_knee
14 right_knee
15 left_ankle
16 right_ankle
```

각 관절은 다음 세 값으로 저장합니다.

```text
[x_normalized, y_normalized, confidence]
```

예:

```text
[0.5124, 0.2851, 0.8732]
```

`x`, `y`를 0~1로 정규화하면 영상 해상도가 달라도 같은 좌표 범위에서 처리할 수 있습니다.

# 12. 처음부터 48개를 처리하지 않고 대표 영상 3개를 먼저 확인합니다

본 실습의 대표 영상:

```text
S11_boxing_T01.avi
S11_handwaving_T01.avi
S11_handclapping_T01.avi
```

```text
대표 3개
→ Pose 추출
→ JSON 확인
→ Skeleton Preview
→ 정상 확인
→ Main 전체 48개 처리
```

# 13. Video → Keypoint JSON 코드를 작성합니다

코드를 작성하기 전에 처리 순서를 먼저 읽습니다.

```text
AVI 열기
→ Frame 하나 읽기
→ YOLO Pose
→ 사람이 없으면 17개 0값
→ 사람이 있으면 가장 신뢰도가 높은 사람 선택
→ 17개 Keypoint 추출
→ x / width, y / height
→ JSON Frame에 추가
→ 영상 끝까지 반복
```

**파일: `scripts/01_extract_keypoints.py`**

```python
from __future__ import annotations

import argparse
import json
import sys
from pathlib import Path

import cv2
import numpy as np
from ultralytics import YOLO


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day05_common import (
    list_videos,
    original_keypoint_dir,
    parse_video_name,
)


MODEL_PATH = (
    ROOT
    / "models"
    / "yolo11n-pose.pt"
)

COCO17_NAMES = [
    "nose",
    "left_eye",
    "right_eye",
    "left_ear",
    "right_ear",
    "left_shoulder",
    "right_shoulder",
    "left_elbow",
    "right_elbow",
    "left_wrist",
    "right_wrist",
    "left_hip",
    "right_hip",
    "left_knee",
    "right_knee",
    "left_ankle",
    "right_ankle",
]


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--asset-set",
        default="main",
        choices=["main", "challenge"],
    )

    parser.add_argument(
        "--split",
        default="all",
        choices=["train", "val", "test", "all"],
    )

    parser.add_argument(
        "--demo",
        action="store_true",
    )

    parser.add_argument(
        "--conf",
        type=float,
        default=0.15,
    )

    parser.add_argument(
        "--imgsz",
        type=int,
        default=640,
    )

    parser.add_argument(
        "--overwrite",
        action="store_true",
    )

    return parser.parse_args()


def choose_person(result) -> int | None:
    if (
        result.keypoints is None
        or result.keypoints.xy is None
        or len(result.keypoints.xy) == 0
    ):
        return None

    if (
        result.boxes is not None
        and result.boxes.conf is not None
        and len(result.boxes.conf) > 0
    ):
        return int(
            result.boxes.conf.argmax().item()
        )

    return 0


def demo_videos() -> list[Path]:
    names = [
        "S11_boxing_T01.avi",
        "S11_handwaving_T01.avi",
        "S11_handclapping_T01.avi",
    ]

    train_videos = {
        path.name: path
        for path in list_videos(
            "main",
            "train",
        )
    }

    missing = [
        name
        for name in names
        if name not in train_videos
    ]

    if missing:
        raise RuntimeError(
            "Demo 영상이 없습니다: "
            + ", ".join(missing)
        )

    return [
        train_videos[name]
        for name in names
    ]


def extract_video(
    model,
    video_path: Path,
    output_path: Path,
    conf: float,
    imgsz: int,
) -> None:
    capture = cv2.VideoCapture(
        str(video_path)
    )

    if not capture.isOpened():
        raise RuntimeError(
            f"영상을 열 수 없습니다: {video_path}"
        )

    fps = float(
        capture.get(cv2.CAP_PROP_FPS)
    )

    width = int(
        capture.get(cv2.CAP_PROP_FRAME_WIDTH)
    )

    height = int(
        capture.get(cv2.CAP_PROP_FRAME_HEIGHT)
    )

    info = parse_video_name(
        video_path
    )

    frames = []
    frame_index = 0
    detected_frames = 0
    confidence_values = []

    while True:
        success, frame = capture.read()

        if not success:
            break

        result = model.predict(
            source=frame,
            conf=conf,
            imgsz=imgsz,
            verbose=False,
        )[0]

        person_index = choose_person(
            result
        )

        if person_index is None:
            keypoints = [
                [0.0, 0.0, 0.0]
                for _ in range(17)
            ]

            detected = False

        else:
            xy = (
                result.keypoints.xy[
                    person_index
                ]
                .detach()
                .cpu()
                .numpy()
            )

            if len(xy) != 17:
                raise RuntimeError(
                    "COCO17 Keypoint 수가 "
                    f"17이 아닙니다: {len(xy)}"
                )

            if (
                result.keypoints.conf
                is not None
            ):
                scores = (
                    result.keypoints.conf[
                        person_index
                    ]
                    .detach()
                    .cpu()
                    .numpy()
                )
            else:
                scores = np.ones(
                    17,
                    dtype=np.float32,
                )

            keypoints = []

            for joint_index in range(17):
                x = float(
                    xy[joint_index, 0]
                )

                y = float(
                    xy[joint_index, 1]
                )

                score = float(
                    scores[joint_index]
                )

                nx = (
                    x / width
                    if width > 0
                    else 0.0
                )

                ny = (
                    y / height
                    if height > 0
                    else 0.0
                )

                keypoints.append(
                    [
                        round(nx, 8),
                        round(ny, 8),
                        round(score, 6),
                    ]
                )

                if score > 0:
                    confidence_values.append(
                        score
                    )

            detected = True
            detected_frames += 1

        frames.append(
            {
                "frame_index": frame_index,
                "detected": detected,
                "keypoints": keypoints,
            }
        )

        frame_index += 1

    capture.release()

    if frame_index == 0:
        raise RuntimeError(
            f"Frame이 없습니다: {video_path.name}"
        )

    mean_conf = (
        float(
            np.mean(confidence_values)
        )
        if confidence_values
        else 0.0
    )

    payload = {
        "source_video": video_path.name,
        "subject": info["subject"],
        "action": info["action"],
        "take": info["take"],
        "fps": fps,
        "width": width,
        "height": height,
        "total_frames": frame_index,
        "detected_frames": detected_frames,
        "detection_rate": (
            detected_frames / frame_index
        ),
        "mean_keypoint_confidence": mean_conf,
        "keypoint_format": (
            "COCO17_normalized_xy_conf"
        ),
        "joint_names": COCO17_NAMES,
        "frames": frames,
    }

    output_path.parent.mkdir(
        parents=True,
        exist_ok=True,
    )

    output_path.write_text(
        json.dumps(
            payload,
            ensure_ascii=False,
            indent=2,
        ),
        encoding="utf-8",
    )

    print(
        video_path.name,
        "→",
        output_path.name,
        f"Detection={payload['detection_rate']:.3f}",
    )


def main() -> None:
    args = parse_args()

    if not MODEL_PATH.exists():
        raise FileNotFoundError(
            "YOLO Pose 모델이 없습니다:\n"
            f"{MODEL_PATH}"
        )

    if args.demo:
        if (
            args.asset_set != "main"
            or args.split not in {"all", "train"}
        ):
            raise ValueError(
                "--demo는 main/train에서 사용합니다."
            )

        work_items = [
            (
                path,
                original_keypoint_dir(
                    "main",
                    "train",
                ),
            )
            for path in demo_videos()
        ]

    else:
        splits = (
            ["train", "val", "test"]
            if args.split == "all"
            else [args.split]
        )

        work_items = []

        for split in splits:
            output_dir = (
                original_keypoint_dir(
                    args.asset_set,
                    split,
                )
            )

            for path in list_videos(
                args.asset_set,
                split,
            ):
                work_items.append(
                    (
                        path,
                        output_dir,
                    )
                )

    model = YOLO(
        str(MODEL_PATH)
    )

    for video_path, output_dir in work_items:
        output_path = (
            output_dir
            / f"{video_path.stem}.json"
        )

        if (
            output_path.exists()
            and not args.overwrite
        ):
            print(
                "[SKIP]",
                output_path.name,
            )
            continue

        extract_video(
            model=model,
            video_path=video_path,
            output_path=output_path,
            conf=args.conf,
            imgsz=args.imgsz,
        )


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

영상 자체를 직접 변형하기 전에 사람의 동작을 **관절 좌표 시계열**로 바꾸기 위해 작성합니다.

### 의사코드로 읽어보기

```text
YOLO Pose 모델을 연다
→ AVI 영상을 연다
→ Frame을 하나씩 읽는다
→ Pose를 추론한다
→ 사람이 없으면 17개 0값을 넣는다
→ 사람이 있으면 COCO17 좌표를 읽는다
→ x와 y를 영상 크기로 나눈다
→ Frame별 Keypoint를 저장한다
→ 영상 끝까지 반복한다
→ Detection Rate를 계산한다
→ JSON으로 저장한다
```

# 14. 대표 3개 영상으로 먼저 실행합니다

```bash
python scripts/01_extract_keypoints.py \
  --asset-set main \
  --split train \
  --demo
```

결과:

```text
data/keypoints/main/original/train/
├─ S11_boxing_T01.json
├─ S11_handwaving_T01.json
└─ S11_handclapping_T01.json
```

JSON 일부를 확인합니다.

```bash
head -50 \
  data/keypoints/main/original/train/S11_boxing_T01.json
```

# 15. Keypoint를 원본 영상 위에 다시 그립니다

숫자만 보고 Pose가 정상인지 판단하기 어렵습니다.

원본 영상과 JSON을 다시 연결하여 Skeleton을 그립니다.

Augmented Sequence는 원본 Video와 Frame 수가 달라질 수 있으므로 `--video`를 생략하면 빈 Canvas 위에 Skeleton만 그리도록 구성합니다.

**파일: `scripts/02_visualize_keypoints.py`**

```python
from __future__ import annotations

import argparse
import json
from pathlib import Path

import cv2
import numpy as np


SKELETON = [
    (5, 6),
    (5, 7),
    (7, 9),
    (6, 8),
    (8, 10),
    (5, 11),
    (6, 12),
    (11, 12),
    (11, 13),
    (13, 15),
    (12, 14),
    (14, 16),
]


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--json",
        required=True,
    )

    parser.add_argument(
        "--video",
        default="",
    )

    parser.add_argument(
        "--output",
        required=True,
    )

    parser.add_argument(
        "--confidence",
        type=float,
        default=0.15,
    )

    parser.add_argument(
        "--width",
        type=int,
        default=640,
    )

    parser.add_argument(
        "--height",
        type=int,
        default=480,
    )

    return parser.parse_args()


def main() -> None:
    args = parse_args()

    data = json.loads(
        Path(args.json).read_text(
            encoding="utf-8"
        )
    )

    capture = None

    if args.video:
        capture = cv2.VideoCapture(
            args.video
        )

        if not capture.isOpened():
            raise RuntimeError(
                "원본 영상을 열 수 없습니다."
            )

        width = int(
            capture.get(
                cv2.CAP_PROP_FRAME_WIDTH
            )
        )

        height = int(
            capture.get(
                cv2.CAP_PROP_FRAME_HEIGHT
            )
        )

        fps = float(
            capture.get(
                cv2.CAP_PROP_FPS
            )
        )

    else:
        width = args.width
        height = args.height

        fps = float(
            data.get(
                "fps",
                25.0,
            )
        )

        if fps <= 0:
            fps = 25.0

    output_path = Path(
        args.output
    )

    output_path.parent.mkdir(
        parents=True,
        exist_ok=True,
    )

    writer = cv2.VideoWriter(
        str(output_path),
        cv2.VideoWriter_fourcc(
            *"mp4v"
        ),
        fps,
        (width, height),
    )

    if not writer.isOpened():
        raise RuntimeError(
            "출력 VideoWriter를 열 수 없습니다."
        )

    for frame_index, item in enumerate(
        data["frames"]
    ):
        if capture is not None:
            success, frame = capture.read()

            if not success:
                break
        else:
            frame = np.full(
                (
                    height,
                    width,
                    3,
                ),
                32,
                dtype=np.uint8,
            )

        points = []

        for x, y, score in item["keypoints"]:
            px = int(x * width)
            py = int(y * height)

            points.append(
                (
                    px,
                    py,
                    score,
                )
            )

        if len(points) != 17:
            raise RuntimeError(
                "Keypoint 수가 17이 아닙니다."
            )

        for start, end in SKELETON:
            x1, y1, s1 = points[start]
            x2, y2, s2 = points[end]

            if (
                s1 >= args.confidence
                and s2 >= args.confidence
            ):
                cv2.line(
                    frame,
                    (x1, y1),
                    (x2, y2),
                    (255, 255, 255),
                    2,
                )

        for x, y, score in points:
            if score >= args.confidence:
                cv2.circle(
                    frame,
                    (x, y),
                    3,
                    (255, 255, 255),
                    -1,
                )

        cv2.putText(
            frame,
            f"{data['action']} frame={frame_index}",
            (12, 24),
            cv2.FONT_HERSHEY_SIMPLEX,
            0.55,
            (255, 255, 255),
            1,
            cv2.LINE_AA,
        )

        writer.write(frame)

    if capture is not None:
        capture.release()

    writer.release()

    print(
        "Saved:",
        output_path,
    )


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

Keypoint JSON의 숫자가 실제 사람의 관절을 따라가는지 **영상으로 다시 확인**하기 위해 작성합니다.

### 의사코드로 읽어보기

```text
Keypoint JSON을 읽는다
→ 원본 Video가 있으면 Frame을 읽는다
→ 없으면 빈 Canvas를 만든다
→ 정규화 x,y를 Pixel 좌표로 되돌린다
→ 관절 사이에 Skeleton Line을 그린다
→ Keypoint 점을 그린다
→ Frame을 Video로 저장한다
```

실행합니다.

```bash
python scripts/02_visualize_keypoints.py \
  --json data/keypoints/main/original/train/S11_boxing_T01.json \
  --video data/day05_student_data/main/train/S11_boxing_T01.avi \
  --output results/main/S11_boxing_T01_pose.mp4
```

다음 항목을 확인합니다.

```text
사람을 따라가는가?

손목·팔꿈치가 동작을 따라가는가?

사람이 작아졌을 때 검출이 흔들리는가?

일부 Frame에서 관절이 사라지는가?
```

# 16. 대표 결과가 정상이라면 Main 48개 전체를 추출합니다

```bash
python scripts/01_extract_keypoints.py \
  --asset-set main \
  --split all
```

Demo에서 이미 만든 세 JSON은 자동으로 `[SKIP]` 처리됩니다.

예상:

```text
data/keypoints/main/original/
├─ train/ 24 JSON
├─ val/   12 JSON
└─ test/  12 JSON
```


# PART 3. Main Train Sequence를 보강하고 Metadata를 기록합니다

# 17. 왜 Train만 보강하는지 다시 확인합니다

잘못된 구조:

```text
Train
Val
Test
        ↓
모두 Augmentation
```

올바른 구조:

```text
Train
→ Augmentation O

Validation
→ Original only

Test
→ Original only
```

Validation과 Test까지 보강하면 평가 조건 자체가 바뀌므로 Baseline과 공정하게 비교하기 어렵습니다.

# 18. 네 가지 Sequence Augmentation을 이해합니다

오늘 사용하는 기본 방법은 다음입니다.

```text
Translation
→ 모든 관절 위치를 조금 이동

Scale
→ 사람 중심 기준으로 관절 크기 변화

Jitter
→ 좌표에 작은 Gaussian Noise

Time Warp Fast
→ Frame 수 감소

Time Warp Slow
→ Frame 수 증가
```

오늘 기본값:

```text
Translation
dx,dy 약 ±0.04 이내

Scale
0.90 ~ 1.10

Jitter
sigma = 0.005

Time Warp Fast
0.85

Time Warp Slow
1.15
```

이 값은 절대 기준이 아닙니다.

실제 환경과 동작 의미를 보면서 조절합니다.

# 19. Sequence Augmentation 코드를 작성합니다

코드를 작성하기 전에 오늘의 두 가지 사용 방법을 구분합니다.

```text
본 실습
→ 다섯 방법을 모두 사용
→ Translation / Scale / Jitter / Time Warp Fast / Slow

Mini Challenge
→ Jitter만 사용
→ sigma 0.005와 0.015를 비교
```

같은 스크립트를 두 경우에 모두 사용할 수 있도록 `--methods` 옵션을 넣습니다.

또한 Time Warp처럼 Frame 수가 바뀌는 경우에는 `total_frames`, `detected_frames`, `detection_rate`도 새 Sequence를 기준으로 다시 계산합니다. 원본 JSON의 값을 그대로 복사하면 실제 Frame 수와 Metadata가 서로 달라질 수 있기 때문입니다.

Main Train은 24개입니다. 다섯 종류를 모두 만들면:

```text
24 Original
×
5 Augmentation
=
120 Augmented Sequence
```

가 됩니다.

**파일: `scripts/03_augment_keypoints.py`**

```python
from __future__ import annotations

import argparse
import json
import random
import shutil
import sys
from pathlib import Path

import numpy as np
import pandas as pd


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day05_common import (
    augmented_keypoint_dir,
    original_keypoint_dir,
    report_dir,
)


BASE_SEED = 42

SUPPORTED_METHODS = [
    "translate",
    "scale",
    "jitter",
    "timewarp_fast",
    "timewarp_slow",
]


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--asset-set",
        required=True,
        choices=[
            "main",
            "challenge",
        ],
    )

    parser.add_argument(
        "--tag",
        required=True,
    )

    parser.add_argument(
        "--translate-max",
        type=float,
        default=0.04,
    )

    parser.add_argument(
        "--scale-min",
        type=float,
        default=0.90,
    )

    parser.add_argument(
        "--scale-max",
        type=float,
        default=1.10,
    )

    parser.add_argument(
        "--jitter-sigma",
        type=float,
        default=0.005,
    )

    parser.add_argument(
        "--time-fast",
        type=float,
        default=0.85,
    )

    parser.add_argument(
        "--time-slow",
        type=float,
        default=1.15,
    )

    parser.add_argument(
        "--methods",
        nargs="+",
        choices=SUPPORTED_METHODS,
        default=SUPPORTED_METHODS,
    )

    parser.add_argument(
        "--overwrite",
        action="store_true",
    )

    return parser.parse_args()


def to_array(
    data: dict,
) -> np.ndarray:
    return np.asarray(
        [
            item["keypoints"]
            for item in data["frames"]
        ],
        dtype=np.float32,
    )


def make_output(
    original: dict,
    array: np.ndarray,
) -> dict:
    output = dict(original)
    output_frames = []

    detected_frames = 0
    confidence_values = []

    for frame_index in range(
        len(array)
    ):
        frame = array[
            frame_index
        ]

        detected = bool(
            np.any(
                frame[:, 2] > 0
            )
        )

        if detected:
            detected_frames += 1

        positive_scores = (
            frame[
                frame[:, 2] > 0,
                2,
            ]
        )

        confidence_values.extend(
            positive_scores.tolist()
        )

        output_frames.append(
            {
                "frame_index": frame_index,
                "detected": detected,
                "keypoints": frame.tolist(),
            }
        )

    output["frames"] = output_frames
    output["total_frames"] = len(
        output_frames
    )
    output["detected_frames"] = (
        detected_frames
    )
    output["detection_rate"] = (
        detected_frames
        / len(output_frames)
        if output_frames
        else 0.0
    )
    output[
        "mean_keypoint_confidence"
    ] = (
        float(
            np.mean(
                confidence_values
            )
        )
        if confidence_values
        else 0.0
    )

    return output


def translate(
    array: np.ndarray,
    dx: float,
    dy: float,
) -> np.ndarray:
    result = array.copy()

    valid = (
        result[:, :, 2] > 0
    )

    x = result[:, :, 0]
    y = result[:, :, 1]

    x[valid] = x[valid] + dx
    y[valid] = y[valid] + dy

    result[:, :, 0] = np.clip(
        x,
        0,
        1,
    )

    result[:, :, 1] = np.clip(
        y,
        0,
        1,
    )

    return result


def scale_body(
    array: np.ndarray,
    factor: float,
) -> np.ndarray:
    result = array.copy()

    for frame_index in range(
        len(result)
    ):
        valid = (
            result[
                frame_index,
                :,
                2,
            ] > 0
        )

        if valid.sum() < 2:
            continue

        center = (
            result[
                frame_index,
                valid,
                :2,
            ].mean(
                axis=0
            )
        )

        result[
            frame_index,
            valid,
            :2,
        ] = (
            center
            + (
                result[
                    frame_index,
                    valid,
                    :2,
                ]
                - center
            )
            * factor
        )

    result[:, :, :2] = np.clip(
        result[:, :, :2],
        0,
        1,
    )

    return result


def jitter(
    array: np.ndarray,
    sigma: float,
    rng,
) -> np.ndarray:
    result = array.copy()

    valid = (
        result[:, :, 2] > 0
    )

    noise = rng.normal(
        0,
        sigma,
        size=result[:, :, :2].shape,
    )

    xy = result[:, :, :2]
    xy[valid] = (
        xy[valid]
        + noise[valid]
    )

    result[:, :, :2] = np.clip(
        xy,
        0,
        1,
    )

    return result


def time_warp(
    array: np.ndarray,
    factor: float,
) -> np.ndarray:
    old_length = len(array)

    new_length = max(
        2,
        int(
            round(
                old_length
                * factor
            )
        ),
    )

    old_position = np.linspace(
        0,
        old_length - 1,
        old_length,
    )

    new_position = np.linspace(
        0,
        old_length - 1,
        new_length,
    )

    output = np.zeros(
        (
            new_length,
            array.shape[1],
            array.shape[2],
        ),
        dtype=np.float32,
    )

    for joint in range(
        array.shape[1]
    ):
        for channel in range(
            array.shape[2]
        ):
            output[
                :,
                joint,
                channel,
            ] = np.interp(
                new_position,
                old_position,
                array[
                    :,
                    joint,
                    channel,
                ],
            )

    output[:, :, :2] = np.clip(
        output[:, :, :2],
        0,
        1,
    )

    output[:, :, 2] = np.clip(
        output[:, :, 2],
        0,
        1,
    )

    return output


def save_variant(
    output_dir: Path,
    source_path: Path,
    original: dict,
    array: np.ndarray,
    method: str,
    parameter: str,
    seed: int,
    rows: list[dict],
) -> None:
    output = make_output(
        original,
        array,
    )

    output[
        "augmentation"
    ] = {
        "method": method,
        "parameter": parameter,
        "source": source_path.name,
        "seed": seed,
    }

    output_name = (
        f"{source_path.stem}"
        f"__{method}.json"
    )

    output_path = (
        output_dir
        / output_name
    )

    output_path.write_text(
        json.dumps(
            output,
            ensure_ascii=False,
            indent=2,
        ),
        encoding="utf-8",
    )

    rows.append(
        {
            "file": output_name,
            "source": source_path.name,
            "subject": original["subject"],
            "action": original["action"],
            "method": method,
            "parameter": parameter,
            "seed": seed,
            "approved": "pending",
        }
    )


def main() -> None:
    args = parse_args()

    if not (
        0
        <= args.translate_max
        <= 0.20
    ):
        raise ValueError(
            "--translate-max 범위를 확인하세요."
        )

    if not (
        0.5
        <= args.scale_min
        <= args.scale_max
        <= 1.5
    ):
        raise ValueError(
            "Scale 범위를 확인하세요."
        )

    if not (
        0
        <= args.jitter_sigma
        <= 0.10
    ):
        raise ValueError(
            "--jitter-sigma 범위를 확인하세요."
        )

    if not (
        0.50
        <= args.time_fast
        < 1.0
    ):
        raise ValueError(
            "--time-fast는 0.50 이상 1.0 미만이어야 합니다."
        )

    if not (
        1.0
        < args.time_slow
        <= 1.50
    ):
        raise ValueError(
            "--time-slow는 1.0 초과 1.50 이하여야 합니다."
        )

    source_dir = (
        original_keypoint_dir(
            args.asset_set,
            "train",
        )
    )

    source_files = sorted(
        source_dir.glob(
            "*.json"
        )
    )

    if not source_files:
        raise RuntimeError(
            "Train Original Keypoint JSON이 없습니다."
        )

    output_dir = (
        augmented_keypoint_dir(
            args.asset_set,
            args.tag,
        )
    )

    if any(
        output_dir.iterdir()
    ):
        if not args.overwrite:
            raise RuntimeError(
                "이미 Augmented 결과가 있습니다.\n"
                "다시 만들려면 --overwrite를 사용하세요."
            )

        shutil.rmtree(
            output_dir
        )

        output_dir.mkdir(
            parents=True,
            exist_ok=True,
        )

    rows = []

    for index, source_path in enumerate(
        source_files
    ):
        seed = (
            BASE_SEED
            + index
        )

        py_rng = random.Random(
            seed
        )

        np_rng = np.random.default_rng(
            seed
        )

        original = json.loads(
            source_path.read_text(
                encoding="utf-8"
            )
        )

        array = to_array(
            original
        )

        if (
            array.ndim != 3
            or array.shape[1:] != (
                17,
                3,
            )
        ):
            raise RuntimeError(
                f"Keypoint Shape 오류: "
                f"{source_path.name} "
                f"{array.shape}"
            )

        if "translate" in args.methods:
            dx = py_rng.uniform(
                -args.translate_max,
                args.translate_max,
            )

            dy = py_rng.uniform(
                -args.translate_max,
                args.translate_max,
            )

            save_variant(
                output_dir,
                source_path,
                original,
                translate(
                    array,
                    dx,
                    dy,
                ),
                "translate",
                (
                    f"dx={dx:.5f},"
                    f"dy={dy:.5f}"
                ),
                seed,
                rows,
            )

        if "scale" in args.methods:
            scale_factor = (
                py_rng.uniform(
                    args.scale_min,
                    args.scale_max,
                )
            )

            save_variant(
                output_dir,
                source_path,
                original,
                scale_body(
                    array,
                    scale_factor,
                ),
                "scale",
                (
                    f"factor="
                    f"{scale_factor:.5f}"
                ),
                seed,
                rows,
            )

        if "jitter" in args.methods:
            save_variant(
                output_dir,
                source_path,
                original,
                jitter(
                    array,
                    args.jitter_sigma,
                    np_rng,
                ),
                "jitter",
                (
                    f"sigma="
                    f"{args.jitter_sigma}"
                ),
                seed,
                rows,
            )

        if "timewarp_fast" in args.methods:
            save_variant(
                output_dir,
                source_path,
                original,
                time_warp(
                    array,
                    args.time_fast,
                ),
                "timewarp_fast",
                (
                    f"factor="
                    f"{args.time_fast}"
                ),
                seed,
                rows,
            )

        if "timewarp_slow" in args.methods:
            save_variant(
                output_dir,
                source_path,
                original,
                time_warp(
                    array,
                    args.time_slow,
                ),
                "timewarp_slow",
                (
                    f"factor="
                    f"{args.time_slow}"
                ),
                seed,
                rows,
            )

    metadata_path = (
        report_dir(
            args.asset_set
        )
        / f"{args.tag}_metadata.csv"
    )

    pd.DataFrame(
        rows
    ).to_csv(
        metadata_path,
        index=False,
        encoding="utf-8-sig",
    )

    print("=" * 68)
    print("Sequence Augmentation Completed")
    print("=" * 68)
    print(
        "Methods       :",
        ", ".join(args.methods),
    )
    print(
        "Original Train:",
        len(source_files),
    )
    print(
        "Augmented     :",
        len(rows),
    )
    print(
        "Output        :",
        output_dir,
    )
    print(
        "Metadata      :",
        metadata_path,
    )


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

원본 Train Sequence에서 동작 의미를 유지하는 작은 변화를 만들어 **위치·크기·좌표 오차·속도 조건을 보강**하기 위해 작성합니다.

`--methods`를 사용하면 본 실습에서는 다섯 방법을 모두 만들고, Mini Challenge에서는 Jitter만 선택할 수 있습니다. 이렇게 하면 Challenge에서 Jitter 강도만 바꾼 **정확한 한 변수 실험**을 만들 수 있습니다.

### 의사코드로 읽어보기

```text
Train Original JSON을 찾는다
        ↓
사용할 Augmentation 방법을 확인한다
        ↓
한 Sequence를 NumPy 배열로 바꾼다
        ↓
선택한 방법만 적용한다
        ↓
새 Sequence의 Frame 수와 Detection 정보를 다시 계산한다
        ↓
Augmented JSON으로 저장한다
        ↓
Source / Method / Parameter / Seed를 Metadata에 기록한다
        ↓
모든 Train Sequence에 반복한다
```

# 20. Main Train을 다섯 방법으로 보강합니다

Main 본 실습에서는 다섯 방법을 모두 사용합니다. `--methods`를 생략하면 기본값으로 다섯 방법이 모두 선택됩니다.

```bash
python scripts/03_augment_keypoints.py \
  --asset-set main \
  --tag main_aug \
  --jitter-sigma 0.005
```

예상:

```text
Methods        : translate, scale, jitter, timewarp_fast, timewarp_slow
Original Train : 24
Augmented      : 120
```

결과:

```text
data/keypoints/main/augmented/main_aug/
├─ S11_boxing_T01__translate.json
├─ S11_boxing_T01__scale.json
├─ S11_boxing_T01__jitter.json
├─ S11_boxing_T01__timewarp_fast.json
├─ S11_boxing_T01__timewarp_slow.json
└─ ...
```

Metadata:

```text
reports/main/main_aug_metadata.csv
```

Metadata의 `approved`는 아직 `pending`입니다.

```text
생성 완료
≠
학습 사용 승인
```

다음 PART에서 Automatic QA와 Human QA를 수행한 뒤 사용할 방법을 결정합니다.

# 21. Augmented Sequence를 눈으로 확인합니다

한 방법만 보고 전체를 판단하지 않습니다.

대표 `S11_boxing_T01`에서 다섯 방법을 Skeleton Video로 만들어 비교합니다.

```bash
for METHOD in translate scale jitter timewarp_fast timewarp_slow
do
  python scripts/02_visualize_keypoints.py \
    --json data/keypoints/main/augmented/main_aug/S11_boxing_T01__${METHOD}.json \
    --output results/main/S11_boxing_T01_${METHOD}.mp4
done
```

생성되는 파일:

```text
results/main/
├─ S11_boxing_T01_translate.mp4
├─ S11_boxing_T01_scale.mp4
├─ S11_boxing_T01_jitter.mp4
├─ S11_boxing_T01_timewarp_fast.mp4
└─ S11_boxing_T01_timewarp_slow.mp4
```

다음 질문에 답하면서 확인합니다.

```text
Translation 후에도 boxing 동작이 유지되는가?

Scale 후 관절의 전체 형태가 자연스러운가?

Jitter 때문에 손목·팔꿈치가 과도하게 흔들리지 않는가?

Time Warp Fast가 지나치게 빠르지 않은가?

Time Warp Slow가 지나치게 느리지 않은가?
```

이 Preview는 뒤의 Human QA에서 어떤 방법을 승인할지 판단하는 근거가 됩니다.

# PART 4. Original과 Augmented Sequence를 QA합니다

# 22. Automatic QA와 Human QA의 역할을 구분합니다

Automatic QA:

```text
JSON 읽기
Frame 존재
17 Keypoint
NaN / Infinity
좌표 범위
유효 Frame 비율
```

Human QA:

```text
같은 Action 의미 유지
Jitter 현실성
Scale 현실성
Time Warp 속도
Skeleton 연속성
```

두 검사는 서로 대체하지 않습니다.

# 23. Automatic QA 코드를 작성합니다

이번 실습에서는 한 Frame에서 Confidence `0.10` 이상인 관절이 8개 이상일 때 유효 Frame으로 계산합니다.

`valid_frame_ratio < 0.30`이면 자동 Review 대상으로 둡니다.

이 값은 현업의 절대 품질 기준이 아니라 **오늘의 저해상도 동작영상에서 Pose 결과를 점검하기 위한 교육용 기준**입니다.

또한 `mean_joint_step`, `max_joint_step`을 함께 계산합니다.

```text
mean_joint_step
→ Frame 사이 관절 이동량의 평균

max_joint_step
→ 가장 크게 튄 관절 이동량
```

이 두 값은 자동 합격/불합격 기준으로 바로 사용하지 않습니다. Action마다 움직임 크기가 다르기 때문입니다. 대신 Human QA에서 Jitter가 과도한지 비교할 때 참고합니다.

**파일: `scripts/04_qa_keypoints.py`**

```python
from __future__ import annotations

import argparse
import json
import math
import sys
from pathlib import Path

import numpy as np
import pandas as pd


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day05_common import (
    augmented_keypoint_dir,
    original_keypoint_dir,
    report_dir,
)


MIN_VALID_FRAME_RATIO = 0.30
JOINT_CONFIDENCE = 0.10


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--asset-set",
        required=True,
        choices=["main", "challenge"],
    )

    parser.add_argument(
        "--tag",
        required=True,
    )

    return parser.parse_args()


def motion_statistics(
    frames: list[dict],
) -> tuple[float, float]:
    steps = []

    previous = None

    for frame in frames:
        keypoints = frame.get(
            "keypoints",
            []
        )

        if len(keypoints) != 17:
            previous = None
            continue

        current = np.asarray(
            keypoints,
            dtype=np.float32,
        )

        if previous is not None:
            valid = (
                (previous[:, 2] >= JOINT_CONFIDENCE)
                & (current[:, 2] >= JOINT_CONFIDENCE)
            )

            if valid.any():
                delta = (
                    current[
                        valid,
                        :2,
                    ]
                    - previous[
                        valid,
                        :2,
                    ]
                )

                distance = np.linalg.norm(
                    delta,
                    axis=1,
                )

                steps.extend(
                    distance.tolist()
                )

        previous = current

    if not steps:
        return 0.0, 0.0

    return (
        float(np.mean(steps)),
        float(np.max(steps)),
    )


def check_file(
    path: Path,
    split: str,
    source_type: str,
) -> dict:
    reasons = []

    try:
        data = json.loads(
            path.read_text(
                encoding="utf-8"
            )
        )
    except Exception:
        return {
            "file": path.name,
            "split": split,
            "source_type": source_type,
            "subject": "",
            "action": "",
            "method": "",
            "frames": 0,
            "valid_frame_ratio": 0.0,
            "mean_joint_step": 0.0,
            "max_joint_step": 0.0,
            "invalid_value": 1,
            "out_of_range": 0,
            "wrong_joint_count": 0,
            "auto_pass": False,
            "reason": "json_read_error",
        }

    frames = data.get(
        "frames",
        []
    )

    if not frames:
        reasons.append(
            "no_frames"
        )

    valid_frames = 0
    invalid_value = 0
    out_of_range = 0
    wrong_joint_count = 0

    for frame in frames:
        keypoints = frame.get(
            "keypoints",
            []
        )

        if len(keypoints) != 17:
            wrong_joint_count += 1
            continue

        valid_joint_count = 0

        for point in keypoints:
            if len(point) != 3:
                invalid_value += 1
                continue

            x, y, score = point

            values = [
                x,
                y,
                score,
            ]

            if not all(
                isinstance(
                    value,
                    (int, float),
                )
                and math.isfinite(
                    value
                )
                for value in values
            ):
                invalid_value += 1
                continue

            if score < 0 or score > 1:
                invalid_value += 1
                continue

            if score > 0 and not (
                0 <= x <= 1
                and 0 <= y <= 1
            ):
                out_of_range += 1

            if score >= JOINT_CONFIDENCE:
                valid_joint_count += 1

        if valid_joint_count >= 8:
            valid_frames += 1

    frame_count = len(
        frames
    )

    valid_ratio = (
        valid_frames / frame_count
        if frame_count > 0
        else 0.0
    )

    if (
        valid_ratio
        < MIN_VALID_FRAME_RATIO
    ):
        reasons.append(
            "low_valid_frame_ratio"
        )

    if invalid_value > 0:
        reasons.append(
            "invalid_value"
        )

    if out_of_range > 0:
        reasons.append(
            "out_of_range"
        )

    if wrong_joint_count > 0:
        reasons.append(
            "wrong_joint_count"
        )

    (
        mean_joint_step,
        max_joint_step,
    ) = motion_statistics(frames)

    augmentation = data.get(
        "augmentation",
        {},
    )

    return {
        "file": path.name,
        "split": split,
        "source_type": source_type,
        "subject": data.get(
            "subject",
            "",
        ),
        "action": data.get(
            "action",
            "",
        ),
        "method": (
            augmentation.get(
                "method",
                "original",
            )
        ),
        "frames": frame_count,
        "valid_frame_ratio": round(
            valid_ratio,
            4,
        ),
        "mean_joint_step": round(
            mean_joint_step,
            6,
        ),
        "max_joint_step": round(
            max_joint_step,
            6,
        ),
        "invalid_value": invalid_value,
        "out_of_range": out_of_range,
        "wrong_joint_count": (
            wrong_joint_count
        ),
        "auto_pass": (
            len(reasons) == 0
        ),
        "reason": "|".join(
            reasons
        ),
    }


def main() -> None:
    args = parse_args()

    rows = []

    for split in [
        "train",
        "val",
        "test",
    ]:
        directory = (
            original_keypoint_dir(
                args.asset_set,
                split,
            )
        )

        for path in sorted(
            directory.glob(
                "*.json"
            )
        ):
            rows.append(
                check_file(
                    path,
                    split,
                    "original",
                )
            )

    aug_dir = (
        augmented_keypoint_dir(
            args.asset_set,
            args.tag,
        )
    )

    for path in sorted(
        aug_dir.glob(
            "*.json"
        )
    ):
        rows.append(
            check_file(
                path,
                "train",
                "augmented",
            )
        )

    if not rows:
        raise RuntimeError(
            "QA할 Keypoint JSON이 없습니다."
        )

    dataframe = pd.DataFrame(
        rows
    )

    output_path = (
        report_dir(
            args.asset_set
        )
        / f"{args.tag}_qa.csv"
    )

    dataframe.to_csv(
        output_path,
        index=False,
        encoding="utf-8-sig",
    )

    passed = int(
        dataframe[
            "auto_pass"
        ].sum()
    )

    print("=" * 68)
    print("Keypoint Sequence QA")
    print("=" * 68)
    print(
        "Total :",
        len(dataframe),
    )
    print(
        "PASS  :",
        passed,
    )
    print(
        "REVIEW:",
        len(dataframe) - passed,
    )
    print(
        "Saved:",
        output_path,
    )


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

보강 데이터가 생성되었다는 사실만으로 Train에 추가하지 않고, **JSON 구조·좌표 범위·유효 Frame 비율을 자동 검사**하기 위해 작성합니다.

Sequence에서는 이미지 한 장과 달리 시간 방향의 움직임도 중요하므로 관절 이동량 통계도 함께 기록합니다.

### 의사코드로 읽어보기

```text
Original과 Augmented JSON을 찾는다
        ↓
JSON을 읽는다
        ↓
Frame 수와 17개 Keypoint 구조를 확인한다
        ↓
NaN / Infinity / Confidence 범위를 확인한다
        ↓
x,y가 0~1 범위인지 확인한다
        ↓
유효 Frame 비율을 계산한다
        ↓
Frame 사이 관절 이동량을 계산한다
        ↓
구조 문제가 없으면 auto_pass=True
        ↓
모든 결과를 QA CSV로 저장한다
```

# 24. Main Automatic QA를 실행합니다

```bash
python scripts/04_qa_keypoints.py \
  --asset-set main \
  --tag main_aug
```

결과:

```text
reports/main/main_aug_qa.csv
```

주요 열을 확인합니다.

```text
valid_frame_ratio
mean_joint_step
max_joint_step
auto_pass
reason
```

`auto_pass=False`인 파일은 먼저 `reason`을 확인하고 해당 Skeleton을 직접 봅니다.

# 25. Human QA를 수행하고 사용할 방법을 명시적으로 결정합니다

Automatic QA를 통과했다고 해서 동작 의미가 반드시 정상이라는 뜻은 아닙니다.

본 실습에서는 120개를 모두 처음부터 끝까지 사람이 보는 대신 다음 방식으로 진행합니다.

```text
Automatic QA
→ 모든 Sequence 검사

Human QA
→ 방법별 대표 Sample 검사
→ Automatic QA Review 대상 추가 검사

최종
→ 방법 단위로 approved / needs_review / rejected
```

먼저 PART 3에서 만든 다섯 Preview를 확인합니다.

가능하면 각 방법에서 Subject·Action이 다른 Sample도 1개씩 더 확인합니다.

다음 항목을 봅니다.

```text
같은 Action Label이라고 말할 수 있는가?

관절이 갑자기 비정상적으로 튀지 않는가?

사람의 크기와 위치가 지나치지 않은가?

빠르기·느리기가 현실적인 범위인가?

Automatic QA에서 max_joint_step이 큰 Sample은
실제로도 부자연스러운가?
```

확인이 끝나면 다음 파일을 직접 만듭니다.

**파일: `reports/main/main_aug_human_qa.csv`**

```csv
method,decision,note
translate,approved,동작 의미가 유지됨
scale,approved,크기 변화가 자연스러움
jitter,approved,관절 흔들림이 허용 범위임
timewarp_fast,approved,빠른 동작으로 볼 수 있음
timewarp_slow,approved,느린 동작으로 볼 수 있음
```

`decision`에는 다음 세 값 중 하나를 사용합니다.

```text
approved
→ 성능비교 Train 후보로 사용

needs_review
→ 추가 확인 전에는 사용하지 않음

rejected
→ 사용하지 않음
```

위 CSV의 내용은 예시입니다. 실제 Preview 결과를 보고 직접 결정합니다.

오늘은 **방법 단위 Human QA**를 사용합니다. 모든 Sequence를 하나씩 수동 승인하는 대신 전체 파일은 Automatic QA로 검사하고, 대표 Sample과 Review 대상을 사람이 확인한 뒤 방법 단위로 사용 여부를 결정합니다.

이후 Feature 생성 코드는 `auto_pass=True`이면서 Human QA에서 `approved`된 방법의 Augmented Sequence만 사용합니다.

# PART 5. Sequence Feature를 만들고 Baseline vs Augmented 성능을 비교합니다

# 26. 길이가 다른 Sequence를 어떻게 같은 모델에 넣을지 생각합니다

영상마다 Frame 수가 다를 수 있고 Time Warp를 적용하면 길이가 더 달라집니다.

```text
Original 120 Frames
Fast     102 Frames
Slow     138 Frames
```

Random Forest는 입력 Feature 수가 같아야 합니다.

오늘은 각 관절의 움직임을 **고정 길이 통계 Feature**로 요약합니다.

각 관절의 `x`, `y`에 대해:

```text
mean
std
min
max
mean absolute velocity
max absolute velocity
```

를 계산합니다.

```text
17 Joints
×
2 Coordinates
×
6 Statistics
=
204 Features
```

이 방법은 최종 산업용 Action Recognition 구조가 아니라 **보강 전후를 같은 조건에서 빠르게 비교하기 위한 교육용 Baseline**입니다.

# 27. Missing Keypoint를 보정하는 방법을 이해합니다

```text
Confidence >= 0.10
→ 유효 좌표

중간 Missing Frame
→ 앞뒤 유효 좌표로 선형보간

처음/끝 Missing
→ 가장 가까운 유효 좌표로 보정

유효 좌표가 전혀 없는 관절
→ 0 유지
```

이 보간은 Random Forest에 넣을 고정 Feature를 계산하기 위해 Sequence의 빈 좌표를 연결하는 **추정 처리**입니다.

```text
보간 좌표
≠
실제로 측정된 Ground Truth
```

Pose가 불안정했던 구간을 완전히 복원했다는 의미가 아니므로, Automatic QA의 유효 Frame 비율과 Human QA의 Skeleton 확인을 함께 봅니다.

# 28. Sequence Feature Dataset을 만드는 코드를 작성합니다

이 단계에서는 **Automatic QA와 Human QA가 모두 끝난 뒤** Feature를 만듭니다.

사용 규칙은 다음과 같습니다.

```text
Original Train / Val / Test
→ Automatic QA 통과 파일 사용

Augmented Train
→ Automatic QA 통과
+
Human QA에서 approved된 방법
→ 둘 다 만족한 Sequence만 사용
```

따라서 `main_aug_human_qa.csv`가 없으면 코드는 중단됩니다. Human QA를 건너뛴 채 Augmented Data를 Train에 넣지 않기 위한 장치입니다.

**파일: `scripts/05_build_features.py`**

```python
from __future__ import annotations

import argparse
import json
import sys
from pathlib import Path

import numpy as np
import pandas as pd


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day05_common import (
    augmented_keypoint_dir,
    feature_dir,
    original_keypoint_dir,
    report_dir,
)


STAT_NAMES = [
    "mean",
    "std",
    "min",
    "max",
    "mean_abs_velocity",
    "max_abs_velocity",
]


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--asset-set",
        required=True,
        choices=["main", "challenge"],
    )

    parser.add_argument(
        "--tag",
        required=True,
    )

    return parser.parse_args()


def load_qa(
    asset_set: str,
    tag: str,
) -> pd.DataFrame:
    path = (
        report_dir(
            asset_set
        )
        / f"{tag}_qa.csv"
    )

    if not path.exists():
        raise FileNotFoundError(
            f"QA Report가 없습니다: {path}"
        )

    return pd.read_csv(
        path
    )


def boolean_mask(
    series: pd.Series,
) -> pd.Series:
    return (
        series.astype(str)
        .str.strip()
        .str.lower()
        .isin(
            [
                "true",
                "1",
                "yes",
            ]
        )
    )


def load_approved_methods(
    asset_set: str,
    tag: str,
) -> set[str]:
    path = (
        report_dir(
            asset_set
        )
        / f"{tag}_human_qa.csv"
    )

    if not path.exists():
        raise FileNotFoundError(
            "Human QA 파일이 없습니다.\n"
            f"{path}\n"
            "Human QA를 먼저 완료하세요."
        )

    dataframe = pd.read_csv(
        path
    )

    required = {
        "method",
        "decision",
    }

    missing = (
        required
        - set(
            dataframe.columns
        )
    )

    if missing:
        raise RuntimeError(
            "Human QA CSV에 필요한 열이 없습니다: "
            + ", ".join(
                sorted(missing)
            )
        )

    decision = (
        dataframe[
            "decision"
        ]
        .fillna("")
        .astype(str)
        .str.strip()
        .str.lower()
    )

    approved = set(
        dataframe.loc[
            decision == "approved",
            "method",
        ]
        .astype(str)
        .str.strip()
        .tolist()
    )

    if not approved:
        raise RuntimeError(
            "approved로 결정된 Augmentation 방법이 없습니다."
        )

    return approved


def approved_original_files(
    qa: pd.DataFrame,
    split: str,
) -> set[str]:
    auto_pass = boolean_mask(
        qa["auto_pass"]
    )

    subset = qa[
        (
            qa["source_type"]
            == "original"
        )
        & (
            qa["split"]
            == split
        )
        & auto_pass
    ]

    return set(
        subset["file"].tolist()
    )


def approved_augmented_files(
    qa: pd.DataFrame,
    approved_methods: set[str],
) -> set[str]:
    auto_pass = boolean_mask(
        qa["auto_pass"]
    )

    subset = qa[
        (
            qa["source_type"]
            == "augmented"
        )
        & (
            qa["split"]
            == "train"
        )
        & auto_pass
        & (
            qa["method"].isin(
                approved_methods
            )
        )
    ]

    return set(
        subset["file"].tolist()
    )


def interpolate_xy(
    array: np.ndarray,
) -> np.ndarray:
    result = array.copy()

    length = len(result)
    positions = np.arange(
        length
    )

    for joint in range(17):
        confidence = (
            result[:, joint, 2]
        )

        valid = (
            confidence >= 0.10
        )

        valid_indices = (
            positions[valid]
        )

        if len(
            valid_indices
        ) == 0:
            result[
                :,
                joint,
                :2,
            ] = 0.0
            continue

        for coordinate in [
            0,
            1,
        ]:
            values = (
                result[
                    valid,
                    joint,
                    coordinate,
                ]
            )

            result[
                :,
                joint,
                coordinate,
            ] = np.interp(
                positions,
                valid_indices,
                values,
            )

    return result


def feature_dict(
    data: dict,
) -> dict[str, float]:
    array = np.asarray(
        [
            item["keypoints"]
            for item in data["frames"]
        ],
        dtype=np.float32,
    )

    if (
        array.ndim != 3
        or array.shape[1:] != (
            17,
            3,
        )
    ):
        raise RuntimeError(
            "Keypoint Shape이 올바르지 않습니다."
        )

    array = interpolate_xy(
        array
    )

    features = {}

    for joint in range(17):
        for coordinate, name in [
            (0, "x"),
            (1, "y"),
        ]:
            values = array[
                :,
                joint,
                coordinate,
            ]

            velocity = np.diff(
                values
            )

            if len(
                velocity
            ) == 0:
                velocity = np.array(
                    [0.0],
                    dtype=np.float32,
                )

            statistics = {
                "mean": float(
                    np.mean(values)
                ),
                "std": float(
                    np.std(values)
                ),
                "min": float(
                    np.min(values)
                ),
                "max": float(
                    np.max(values)
                ),
                "mean_abs_velocity": (
                    float(
                        np.mean(
                            np.abs(
                                velocity
                            )
                        )
                    )
                ),
                "max_abs_velocity": (
                    float(
                        np.max(
                            np.abs(
                                velocity
                            )
                        )
                    )
                ),
            }

            for stat_name in (
                STAT_NAMES
            ):
                key = (
                    f"j{joint:02d}_"
                    f"{name}_"
                    f"{stat_name}"
                )

                features[key] = (
                    statistics[
                        stat_name
                    ]
                )

    return features


def rows_from_directory(
    directory: Path,
    allowed: set[str],
    source_type: str,
) -> list[dict]:
    rows = []

    for path in sorted(
        directory.glob(
            "*.json"
        )
    ):
        if path.name not in allowed:
            continue

        data = json.loads(
            path.read_text(
                encoding="utf-8"
            )
        )

        row = {
            "file": path.name,
            "subject": data["subject"],
            "action": data["action"],
            "source_type": source_type,
        }

        row.update(
            feature_dict(data)
        )

        rows.append(row)

    return rows


def save_rows(
    rows: list[dict],
    path: Path,
) -> None:
    if not rows:
        raise RuntimeError(
            f"저장할 Feature Row가 없습니다: "
            f"{path.name}"
        )

    pd.DataFrame(
        rows
    ).to_csv(
        path,
        index=False,
        encoding="utf-8-sig",
    )


def main() -> None:
    args = parse_args()

    qa = load_qa(
        args.asset_set,
        args.tag,
    )

    approved_methods = (
        load_approved_methods(
            args.asset_set,
            args.tag,
        )
    )

    output_dir = feature_dir(
        args.asset_set,
        args.tag,
    )

    original_rows = {}

    for split in [
        "train",
        "val",
        "test",
    ]:
        allowed = (
            approved_original_files(
                qa,
                split,
            )
        )

        original_rows[
            split
        ] = rows_from_directory(
            original_keypoint_dir(
                args.asset_set,
                split,
            ),
            allowed,
            "original",
        )

    aug_allowed = (
        approved_augmented_files(
            qa,
            approved_methods,
        )
    )

    aug_rows = rows_from_directory(
        augmented_keypoint_dir(
            args.asset_set,
            args.tag,
        ),
        aug_allowed,
        "augmented",
    )

    train_original = (
        original_rows["train"]
    )

    train_with_aug = (
        train_original
        + aug_rows
    )

    save_rows(
        train_original,
        output_dir
        / "train_original.csv",
    )

    save_rows(
        train_with_aug,
        output_dir
        / "train_with_aug.csv",
    )

    save_rows(
        original_rows["val"],
        output_dir
        / "val_original.csv",
    )

    save_rows(
        original_rows["test"],
        output_dir
        / "test_original.csv",
    )

    print("=" * 68)
    print("Sequence Feature Dataset")
    print("=" * 68)
    print(
        "Approved Methods:",
        sorted(
            approved_methods
        ),
    )
    print(
        "Train Original :",
        len(train_original),
    )
    print(
        "Approved Aug   :",
        len(aug_rows),
    )
    print(
        "Train + Aug    :",
        len(train_with_aug),
    )
    print(
        "Validation     :",
        len(
            original_rows["val"]
        ),
    )
    print(
        "Test           :",
        len(
            original_rows["test"]
        ),
    )
    print(
        "Feature Dir    :",
        output_dir,
    )


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

길이가 서로 다른 Original·Time Warp Sequence를 같은 크기의 모델 입력으로 바꾸고, **QA를 통과하고 Human QA에서 승인된 Augmented Data만 성능비교에 사용**하기 위해 작성합니다.

### 의사코드로 읽어보기

```text
Automatic QA CSV를 읽는다
        ↓
Human QA CSV를 읽는다
        ↓
approved된 Augmentation 방법을 찾는다
        ↓
Original은 auto_pass 파일을 선택한다
        ↓
Augmented는 auto_pass + approved 방법만 선택한다
        ↓
Missing Keypoint를 관절별로 보간한다
        ↓
각 관절 x,y의 통계와 속도 Feature를 계산한다
        ↓
204개 Feature를 만든다
        ↓
Train Original CSV를 만든다
        ↓
Train Original + Approved Augmented CSV를 만든다
        ↓
Validation / Test는 Original CSV만 만든다
```

# 29. Main Feature Dataset을 만듭니다

먼저 다음 두 파일이 존재하는지 확인합니다.

```text
reports/main/main_aug_qa.csv
reports/main/main_aug_human_qa.csv
```

그다음 실행합니다.

```bash
python scripts/05_build_features.py \
  --asset-set main \
  --tag main_aug
```

다섯 방법을 모두 `approved`했고 모든 Original이 Automatic QA를 통과했다면:

```text
Approved Methods : 5개
Train Original   : 24
Approved Aug     : 120
Train + Aug      : 144
Validation       : 12
Test             : 12
```

처럼 표시됩니다.

한 방법을 `rejected` 또는 `needs_review`로 두었다면 해당 방법의 Augmented Sequence는 `Train + Aug`에서 제외됩니다.

# 30. Baseline과 Augmented의 조건을 정확히 구분합니다

Baseline:

```text
Original Train
→ Random Forest
→ Same Validation
→ Same Test
```

Augmented:

```text
Original Train
+
Approved Augmented Train
→ 같은 Random Forest
→ Same Validation
→ Same Test
```

바뀌는 것은 **Train Data 구성**입니다.

Main 실험의 Augmentation Pipeline은 이 단계에 오기 전에 이미 정해져 있습니다. 따라서 Test 결과는 최종 보강 효과를 보고하는 데 사용하며, Test Accuracy를 본 뒤 Jitter·Scale·Time Warp 등의 Parameter를 다시 바꾸지 않습니다.

# 31. Baseline vs Augmented 비교 코드를 작성합니다

**파일: `scripts/06_compare_baseline_augmented.py`**

```python
from __future__ import annotations

import argparse
import json
import sys
from pathlib import Path

import joblib
import pandas as pd

from sklearn.ensemble import (
    RandomForestClassifier,
)
from sklearn.metrics import (
    accuracy_score,
    classification_report,
    confusion_matrix,
)


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day05_common import (
    feature_dir,
    read_classes,
    report_dir,
)


SEED = 42


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--asset-set",
        required=True,
        choices=["main", "challenge"],
    )

    parser.add_argument(
        "--tag",
        required=True,
    )

    parser.add_argument(
        "--eval-split",
        choices=["val", "test", "both"],
        default="both",
        help=(
            "Candidate 선택은 val, "
            "Freeze 후 최종 평가는 test, "
            "Main 최종 보고는 both를 사용합니다."
        ),
    )

    return parser.parse_args()


def load_xy(path: Path):
    dataframe = pd.read_csv(path)

    feature_columns = [
        column
        for column in dataframe.columns
        if column.startswith("j")
    ]

    if not feature_columns:
        raise RuntimeError(
            f"Feature가 없습니다: {path}"
        )

    x = dataframe[
        feature_columns
    ].to_numpy()

    y = dataframe[
        "action"
    ].astype(str).to_numpy()

    return (
        dataframe,
        x,
        y,
        feature_columns,
    )


def make_model():
    return RandomForestClassifier(
        n_estimators=300,
        random_state=SEED,
        class_weight="balanced",
        n_jobs=-1,
    )


def evaluate(model, x, y):
    prediction = model.predict(x)

    accuracy = accuracy_score(
        y,
        prediction,
    )

    return (
        accuracy,
        prediction.tolist(),
    )


def evaluate_split(
    split: str,
    input_dir: Path,
    feature_columns: list[str],
    baseline,
    augmented,
    labels: list[str],
) -> dict:
    (
        dataframe,
        x_eval,
        y_eval,
        eval_columns,
    ) = load_xy(
        input_dir
        / f"{split}_original.csv"
    )

    if feature_columns != eval_columns:
        raise RuntimeError(
            f"{split} Feature Column이 Train과 다릅니다."
        )

    if set(y_eval) != set(labels):
        raise RuntimeError(
            f"{split}에 모든 Action이 포함되지 않았습니다."
        )

    (
        baseline_accuracy,
        baseline_prediction,
    ) = evaluate(
        baseline,
        x_eval,
        y_eval,
    )

    (
        augmented_accuracy,
        augmented_prediction,
    ) = evaluate(
        augmented,
        x_eval,
        y_eval,
    )

    return {
        "dataframe": dataframe,
        "y": y_eval,
        "baseline_accuracy": baseline_accuracy,
        "baseline_prediction": baseline_prediction,
        "augmented_accuracy": augmented_accuracy,
        "augmented_prediction": augmented_prediction,
    }


def save_confusion(
    output_dir: Path,
    tag: str,
    model_name: str,
    split: str,
    y_true,
    prediction,
    labels: list[str],
    legacy_test_name: bool = False,
) -> Path:
    matrix = confusion_matrix(
        y_true,
        prediction,
        labels=labels,
    )

    if legacy_test_name:
        filename = (
            f"{tag}_{model_name}"
            "_confusion.csv"
        )
    else:
        filename = (
            f"{tag}_{model_name}_"
            f"{split}_confusion.csv"
        )

    path = output_dir / filename

    pd.DataFrame(
        matrix,
        index=labels,
        columns=labels,
    ).to_csv(
        path,
        encoding="utf-8-sig",
    )

    return path


def main() -> None:
    args = parse_args()

    input_dir = feature_dir(
        args.asset_set,
        args.tag,
    )

    (
        train_original_df,
        x_train_original,
        y_train_original,
        feature_columns,
    ) = load_xy(
        input_dir
        / "train_original.csv"
    )

    (
        train_aug_df,
        x_train_aug,
        y_train_aug,
        aug_columns,
    ) = load_xy(
        input_dir
        / "train_with_aug.csv"
    )

    if feature_columns != aug_columns:
        raise RuntimeError(
            "Baseline Train과 Augmented Train의 "
            "Feature Column이 서로 다릅니다."
        )

    labels = [
        "boxing",
        "handwaving",
        "handclapping",
    ]

    expected_classes = set(
        read_classes().values()
    )

    if expected_classes != set(labels):
        raise RuntimeError(
            "classes.txt와 평가 Label 구성이 다릅니다."
        )

    if set(y_train_original) != expected_classes:
        raise RuntimeError(
            "Baseline Train에 모든 Action이 "
            "포함되지 않았습니다."
        )

    baseline = make_model()
    baseline.fit(
        x_train_original,
        y_train_original,
    )

    augmented = make_model()
    augmented.fit(
        x_train_aug,
        y_train_aug,
    )

    requested_splits = (
        ["val", "test"]
        if args.eval_split == "both"
        else [args.eval_split]
    )

    results = {}

    for split in requested_splits:
        results[split] = evaluate_split(
            split=split,
            input_dir=input_dir,
            feature_columns=feature_columns,
            baseline=baseline,
            augmented=augmented,
            labels=labels,
        )

    output_dir = report_dir(
        args.asset_set
    )

    if args.eval_split == "both":
        summary = pd.DataFrame(
            [
                {
                    "model": "baseline",
                    "train_count": len(
                        train_original_df
                    ),
                    "val_accuracy": (
                        results["val"][
                            "baseline_accuracy"
                        ]
                    ),
                    "test_accuracy": (
                        results["test"][
                            "baseline_accuracy"
                        ]
                    ),
                },
                {
                    "model": "augmented",
                    "train_count": len(
                        train_aug_df
                    ),
                    "val_accuracy": (
                        results["val"][
                            "augmented_accuracy"
                        ]
                    ),
                    "test_accuracy": (
                        results["test"][
                            "augmented_accuracy"
                        ]
                    ),
                },
            ]
        )

        summary_path = (
            output_dir
            / f"{args.tag}_performance.csv"
        )
    else:
        split = args.eval_split

        summary = pd.DataFrame(
            [
                {
                    "model": "baseline",
                    "train_count": len(
                        train_original_df
                    ),
                    "eval_split": split,
                    "accuracy": (
                        results[split][
                            "baseline_accuracy"
                        ]
                    ),
                },
                {
                    "model": "augmented",
                    "train_count": len(
                        train_aug_df
                    ),
                    "eval_split": split,
                    "accuracy": (
                        results[split][
                            "augmented_accuracy"
                        ]
                    ),
                },
            ]
        )

        summary_path = (
            output_dir
            / (
                f"{args.tag}_"
                f"performance_{split}.csv"
            )
        )

    summary.to_csv(
        summary_path,
        index=False,
        encoding="utf-8-sig",
    )

    report = {}

    for split in requested_splits:
        result = results[split]

        save_confusion(
            output_dir=output_dir,
            tag=args.tag,
            model_name="baseline",
            split=split,
            y_true=result["y"],
            prediction=result[
                "baseline_prediction"
            ],
            labels=labels,
            legacy_test_name=(
                args.eval_split == "both"
                and split == "test"
            ),
        )

        save_confusion(
            output_dir=output_dir,
            tag=args.tag,
            model_name="augmented",
            split=split,
            y_true=result["y"],
            prediction=result[
                "augmented_prediction"
            ],
            labels=labels,
            legacy_test_name=(
                args.eval_split == "both"
                and split == "test"
            ),
        )

        report[
            f"baseline_{split}"
        ] = classification_report(
            result["y"],
            result["baseline_prediction"],
            labels=labels,
            output_dict=True,
            zero_division=0,
        )

        report[
            f"augmented_{split}"
        ] = classification_report(
            result["y"],
            result["augmented_prediction"],
            labels=labels,
            output_dict=True,
            zero_division=0,
        )

    if args.eval_split == "both":
        report_path = (
            output_dir
            / (
                f"{args.tag}"
                "_classification_report.json"
            )
        )
    else:
        report_path = (
            output_dir
            / (
                f"{args.tag}_"
                f"{args.eval_split}_"
                "classification_report.json"
            )
        )

    report_path.write_text(
        json.dumps(
            report,
            ensure_ascii=False,
            indent=2,
        ),
        encoding="utf-8",
    )

    model_dir = (
        ROOT
        / "models"
        / args.asset_set
        / args.tag
    )

    model_dir.mkdir(
        parents=True,
        exist_ok=True,
    )

    joblib.dump(
        baseline,
        model_dir
        / "baseline.joblib",
    )

    joblib.dump(
        augmented,
        model_dir
        / "augmented.joblib",
    )

    print("=" * 68)
    print("Baseline vs Augmented")
    print("=" * 68)
    print("Eval Split:", args.eval_split)

    for split in requested_splits:
        result = results[split]

        delta = (
            result["augmented_accuracy"]
            - result["baseline_accuracy"]
        )

        print(
            f"{split.upper():4s} "
            f"Baseline={result['baseline_accuracy']:.4f} "
            f"Augmented={result['augmented_accuracy']:.4f} "
            f"Delta={delta:.4f}"
        )

    print(
        "Saved:",
        summary_path,
    )


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

보강 데이터를 많이 만들었다는 사실이 아니라 **같은 평가 데이터에서 실제 성능 변화가 있었는지 확인**하기 위해 작성합니다.

`--eval-split`은 평가 단계의 역할을 분리하기 위한 옵션입니다.

```text
--eval-split val
→ Candidate 비교·선택

--eval-split test
→ Final 조건 Freeze 후 마지막 평가

--eval-split both
→ 이미 조건이 확정된 Main 실험의 최종 보고
```

Candidate 단계에서 `val`을 선택하면 이 Script는 `test_original.csv`를 읽지 않습니다.

### 의사코드로 읽어보기

```text
Original Train Feature를 읽는다
→ Original + Augmented Train Feature를 읽는다
→ 같은 설정의 Random Forest 두 개를 만든다
→ Baseline은 Original Train으로 학습한다
→ Augmented는 Original + Augmented Train으로 학습한다
→ eval-split을 확인한다
→ val이면 Validation만 읽고 평가한다
→ test이면 Test만 읽고 평가한다
→ both이면 Validation과 Test를 최종 보고한다
→ Accuracy와 Confusion Matrix를 저장한다
→ Model을 저장한다
```

# 32. Main 성능검증을 실행합니다

```bash
python scripts/06_compare_baseline_augmented.py \
  --asset-set main \
  --tag main_aug \
  --eval-split both
```

결과:

```text
reports/main/
├─ main_aug_performance.csv
├─ main_aug_baseline_val_confusion.csv
├─ main_aug_augmented_val_confusion.csv
├─ main_aug_baseline_confusion.csv
├─ main_aug_augmented_confusion.csv
└─ main_aug_classification_report.json
```

모델:

```text
models/main/main_aug/
├─ baseline.joblib
└─ augmented.joblib
```

# 33. 결과를 해석합니다

```text
Augmented Test Accuracy
>
Baseline Test Accuracy
→ 보강이 도움이 되었을 가능성
```

```text
Augmented Test Accuracy
≈
Baseline Test Accuracy
→ 보강 효과가 작거나
  Original만으로도 충분할 가능성
```

```text
Augmented Test Accuracy
<
Baseline Test Accuracy
→ 보강 강도가 부적절하거나
  비현실적인 Sequence가 섞였거나
  작은 데이터셋 변동 가능성
```

오늘 데이터는 교육용 소규모 데이터입니다.

```text
Accuracy
+
Confusion Matrix
+
QA
+
Human Review
+
Metadata
```

를 함께 보고 판단합니다.


---

# PART 6. 5일차 본 실습과 교과 11 전체 흐름을 연결합니다

# 34. Main 본 실습의 산출물을 확인합니다

여기서는 앞의 Pipeline을 다시 길게 반복하지 않고, 실제로 남아 있어야 하는 결과를 확인합니다.

```text
Original Keypoint
→ train / val / test JSON

Augmented Keypoint
→ Train에서 생성한 JSON

Metadata
→ main_aug_metadata.csv

Automatic QA
→ main_aug_qa.csv

Human QA
→ main_aug_human_qa.csv

Feature
→ train_original.csv
→ train_with_aug.csv
→ val_original.csv
→ test_original.csv

Performance
→ main_aug_performance.csv
→ Baseline / Augmented Confusion Matrix
```

이 파일들이 서로 연결되어 있어야 다음 질문에 답할 수 있습니다.

```text
어떤 원본에서 만들었는가?
→ Metadata

구조적으로 정상인가?
→ Automatic QA

동작 의미가 유지되는가?
→ Human QA

실제로 성능에 도움이 되었는가?
→ Performance
```

# 35. 1~5일차의 Metadata를 비교합니다

```text
1일차
Source Image
Method
Brightness / Rotation / Blur / Noise
Seed

2일차
Model
Epoch
Prompt
Seed

3일차
Background
Object
Position
Scale
Rotation
Class

4일차
3D Object
Camera
Light
Rotation
Seed

5일차
Source Video
Subject
Action
Method
Parameter
Seed
```

데이터 형태는 달라졌지만 Metadata의 목적은 같습니다.

```text
이 데이터가
어디에서 왔고
어떻게 만들어졌는지
다시 추적
```

# 36. 5일 동안 반복한 QA를 비교합니다

```text
Image QA
→ 깨짐·밝기·형태

GAN / Diffusion QA
→ 생성 실패·반복·Domain Gap

Detection QA
→ BBox·Class·Label 범위

3D QA
→ RGB·Mask·Depth·BBox

Sequence QA
→ Frame·Keypoint·좌표·동작 의미
```

핵심은 다음입니다.

```text
생성 가능
≠
학습 사용 가능

생성
→ QA
→ 승인
→ Train
```

# 37. 교과 11의 성능비교 원칙을 정리합니다

```text
Before

Real Train
        ↓
Baseline


After

Real Train
+
Approved Synthetic / Augmented
        ↓
Augmented Model


조건 판단

Same Validation
        ↓
Candidate 비교·선택


최종 평가

Final 조건 Freeze
        ↓
Same Test
```

Validation/Test 자체는 실험 사이에서 바꾸지 않습니다. 다만 **조건을 고르는 역할은 Validation**, 최종 성능을 보고하는 역할은 **Test**로 구분합니다.

Main처럼 Augmentation Pipeline이 이미 정해진 최종 비교에서는 같은 Validation/Test를 함께 보고할 수 있지만, Test를 확인한 뒤 Parameter를 다시 조정하지 않습니다.

# 38. 실패 사례를 한 개 이상 기록합니다

`reports/main/failure_cases.md`:

```markdown
# Day 05 Failure Cases

## Case 1

- File:
- Subject:
- Action:
- Augmentation:
- Parameter:
- Automatic QA:
- Human QA:

### Problem

### Possible Cause

### Fix

### Result
```

---

# PART 7. Mini Challenge — `challenge` 데이터로 5일차 전체 흐름을 스스로 반복합니다

# 39. Mini Challenge의 목표를 확인합니다

본 실습:

```text
main

Train
S11 S12 S13 S14

Val
S19 S20

Test
S02 S03
```

Mini Challenge:

```text
challenge

Train
S15 S16

Val
S21

Test
S05
```

본 실습과 다른 Subject를 사용합니다.

새로운 알고리즘을 만드는 문제가 아닙니다.

오늘 작성한 Script를 다시 사용하여 다음 질문에 답합니다.

> **Jitter의 강도만 0.005에서 0.015로 높이면 QA와 Action Recognition 성능은 어떻게 달라질까?**

# 40. Challenge 데이터부터 검사합니다

```bash
python scripts/00_check_day05.py \
  --asset-set challenge
```

확인:

```text
Train 12
Val    6
Test   6
Subject Leakage NONE
```

# 41. Challenge 24개 영상을 Keypoint로 변환합니다

```bash
python scripts/01_extract_keypoints.py \
  --asset-set challenge \
  --split all
```

결과:

```text
data/keypoints/challenge/original/
├─ train/ 12
├─ val/    6
└─ test/   6
```

대표 Sample을 확인합니다.

```bash
python scripts/02_visualize_keypoints.py \
  --json data/keypoints/challenge/original/train/S15_boxing_T01.json \
  --video data/day05_student_data/challenge/train/S15_boxing_T01.avi \
  --output results/challenge/S15_boxing_T01_pose.mp4
```

# 42. 실험 A — Jitter 0.005만 생성합니다

Challenge의 목적은 다섯 보강 방법을 다시 모두 비교하는 것이 아닙니다.

오늘 배운 Pipeline은 그대로 유지하되 **Jitter 하나만 선택**합니다.

```text
Original Challenge Train
        ↓
Jitter 0.005만 생성
        ↓
Automatic QA + Human QA
        ↓
Original + Approved Jitter
        ↓
같은 Validation에서 Candidate 성능 비교
```

실행:

```bash
python scripts/03_augment_keypoints.py \
  --asset-set challenge \
  --tag jitter005 \
  --jitter-sigma 0.005 \
  --methods jitter
```

Challenge Train이 12개이므로 다음과 같이 생성됩니다.

```text
Original Train : 12
Augmented      : 12
Method         : jitter
```

Automatic QA:

```bash
python scripts/04_qa_keypoints.py \
  --asset-set challenge \
  --tag jitter005
```

대표 Jitter Sequence를 확인합니다.

```bash
python scripts/02_visualize_keypoints.py \
  --json data/keypoints/challenge/augmented/jitter005/S15_boxing_T01__jitter.json \
  --output results/challenge/S15_boxing_jitter005.mp4
```

확인 후 Human QA 파일을 만듭니다.

**파일: `reports/challenge/jitter005_human_qa.csv`**

```csv
method,decision,note
jitter,approved,Jitter 0.005에서 동작 의미가 유지됨
```

실제 결과가 부자연스럽다면 `approved` 대신 `needs_review` 또는 `rejected`를 기록합니다.

Feature:

```bash
python scripts/05_build_features.py \
  --asset-set challenge \
  --tag jitter005
```

성능:

```bash
python scripts/06_compare_baseline_augmented.py \
  --asset-set challenge \
  --tag jitter005 \
  --eval-split val
```

# 43. 실험 B — 같은 Noise 패턴에서 Jitter만 0.015로 변경합니다

실험 B도 Jitter만 생성합니다.

다음 조건은 그대로 유지합니다.

```text
같은 Challenge Original Train
같은 Validation
같은 Random Forest
같은 204-D Feature
같은 Seed
같은 Jitter Noise 방향
```

Test S05는 두 Candidate를 고르는 동안 평가에 사용하지 않습니다. 두 조건의 Validation 비교가 끝나고 Final Jitter를 Freeze한 뒤 선택된 한 조건에서만 Test를 실행합니다.

변경하는 값은 하나입니다.

```text
sigma

0.005
→
0.015
```

실행:

```bash
python scripts/03_augment_keypoints.py \
  --asset-set challenge \
  --tag jitter015 \
  --jitter-sigma 0.015 \
  --methods jitter
```

Automatic QA:

```bash
python scripts/04_qa_keypoints.py \
  --asset-set challenge \
  --tag jitter015
```

대표 Sequence:

```bash
python scripts/02_visualize_keypoints.py \
  --json data/keypoints/challenge/augmented/jitter015/S15_boxing_T01__jitter.json \
  --output results/challenge/S15_boxing_jitter015.mp4
```

Human QA 파일:

**파일: `reports/challenge/jitter015_human_qa.csv`**

```csv
method,decision,note
jitter,approved,Jitter 0.015 결과를 직접 확인한 뒤 작성
```

Feature:

```bash
python scripts/05_build_features.py \
  --asset-set challenge \
  --tag jitter015
```

성능:

```bash
python scripts/06_compare_baseline_augmented.py \
  --asset-set challenge \
  --tag jitter015 \
  --eval-split val
```

# 44. Jitter 두 조건을 직접 비교합니다

비교할 파일:

```text
reports/challenge/jitter005_qa.csv
reports/challenge/jitter015_qa.csv

reports/challenge/jitter005_human_qa.csv
reports/challenge/jitter015_human_qa.csv

reports/challenge/jitter005_performance_val.csv
reports/challenge/jitter015_performance_val.csv

reports/challenge/jitter005_baseline_val_confusion.csv
reports/challenge/jitter005_augmented_val_confusion.csv
reports/challenge/jitter015_baseline_val_confusion.csv
reports/challenge/jitter015_augmented_val_confusion.csv
```

다음 표를 작성합니다.

| 비교 | Jitter 0.005 | Jitter 0.015 |
|---|---:|---:|
| Augmented Sequence 수 | | |
| Automatic QA PASS | | |
| `mean_joint_step` 특징 | | |
| `max_joint_step` 특징 | | |
| Human QA 판단 | | |
| Baseline Validation Accuracy | | |
| Augmented Validation Accuracy | | |
| Validation Delta | | |
| Validation Confusion Matrix 특징 | | |
| 더 적절한 Jitter | | |

`Baseline Validation Accuracy`는 두 실험에서 같아야 정상입니다.

왜냐하면:

```text
같은 Original Train
+
같은 Validation
+
같은 Random Forest 설정
+
같은 204-D Feature
```

을 사용하기 때문입니다.

더 적절한 Jitter는 Validation Accuracy 하나만 보고 기계적으로 선택하지 않습니다.

```text
Validation Accuracy
+
Validation Confusion Matrix
+
Automatic QA
+
Human QA
```

를 함께 보고 선택합니다. 이 단계에서는 Test 결과를 보지 않습니다.

# 45. Mini Challenge의 한 변수 원칙을 확인합니다

이번 Challenge에서는 두 실험 모두 `--methods jitter`를 사용합니다.

따라서 비교 구조가 분명합니다.

```text
실험 A
Original Train + Jitter 0.005

실험 B
Original Train + Jitter 0.015
```

다른 Augmentation을 함께 넣지 않으므로 성능 차이를 Jitter 강도와 더 직접적으로 연결하여 해석할 수 있습니다.

## Final Jitter를 선택하고 Freeze한 뒤 Test를 한 번 실행합니다

Validation과 QA를 근거로 `jitter005` 또는 `jitter015` 중 하나를 선택합니다.

예를 들어 Validation 결과를 근거로 `jitter005`를 선택했다면:

```bash
SELECTED_TAG=jitter005

python scripts/06_compare_baseline_augmented.py \
  --asset-set challenge \
  --tag "$SELECTED_TAG" \
  --eval-split test
```

`jitter015`를 선택했다면 `SELECTED_TAG=jitter015`로 바꿉니다.

중요한 점은 **두 Candidate 모두에서 Test를 실행하지 않는 것**입니다.

```text
Jitter 0.005
        \
         → Validation + QA 비교
        /
Jitter 0.015
        ↓
Final Jitter 선택
        ↓
Freeze
        ↓
선택된 Tag만 Test S05
        ↓
Final Test Accuracy
+
Final Test Confusion Matrix
```

선택된 조건의 최종 결과는 다음 형식으로 저장됩니다.

```text
reports/challenge/<선택된태그>_performance_test.csv
reports/challenge/<선택된태그>_baseline_test_confusion.csv
reports/challenge/<선택된태그>_augmented_test_confusion.csv
reports/challenge/<선택된태그>_test_classification_report.json
```

Test 결과를 확인한 뒤 Jitter를 다시 바꾸면 Test가 선택 데이터로 변하므로, 같은 Test를 보면서 조건을 재조정하지 않습니다.

# 46. `challenge_notes.md`를 작성합니다

**파일: `reports/challenge/challenge_notes.md`**

```markdown
# Day 05 Mini Challenge

## 1. Challenge Data

- Train Subjects:
- Validation Subject:
- Test Subject:
- Action:
- Train Video:
- Val Video:
- Test Video:

## 2. Keypoint Extraction

- JSON 수:
- 가장 낮은 Detection Rate:
- 가장 확인이 필요했던 영상:

## 3. Candidate A

- Tag: jitter005
- Jitter sigma: 0.005
- Augmented Sequence:
- Automatic QA PASS:
- mean/max joint step 특징:
- Human QA:
- Baseline Validation Accuracy:
- Augmented Validation Accuracy:
- Validation Delta:
- Validation Confusion Matrix 특징:

## 4. Candidate B

- Tag: jitter015
- Jitter sigma: 0.015
- Augmented Sequence:
- Automatic QA PASS:
- mean/max joint step 특징:
- Human QA:
- Baseline Validation Accuracy:
- Augmented Validation Accuracy:
- Validation Delta:
- Validation Confusion Matrix 특징:

## 5. 한 변수 비교와 Final 선택

- 동일하게 유지한 조건:
- 변경한 조건:
- Skeleton에서 가장 큰 차이:
- QA에서 가장 큰 차이:
- Validation 성능에서 가장 큰 차이:
- 선택한 Jitter:
- 선택 근거:
- Final Jitter Freeze 여부:

## 6. Final Test

- Selected Tag:
- Test Subject:
- Baseline Test Accuracy:
- Augmented Test Accuracy:
- Test Delta:
- Final Test Confusion Matrix 특징:
- Test를 본 뒤 Jitter를 다시 변경하지 않았는가?:

## 7. 실패 사례

- File:
- Problem:
- Metadata:
- 원인 가설:
- 수정 방법:

## 8. 최종 해석

- 선택한 Jitter가 Validation에서 더 적절했던 이유:
- Final Test에서 확인된 결과:
- 다음 실험에서 보완할 점:

## 9. 오늘의 결론

- Sequence Augmentation에서 가장 중요한 점:
- QA가 필요한 이유:
- Candidate 선택에 Validation을 사용하는 이유:
- Final Test를 마지막 한 번만 사용하는 이유:
```

# 47. Mini Challenge 완료 조건을 확인합니다

- [ ] `challenge` Train·Val·Test 수량을 확인했다.
- [ ] Subject가 Split 사이에 겹치지 않는 것을 확인했다.
- [ ] 24개 Challenge Video를 Keypoint JSON으로 변환했다.
- [ ] 대표 Original Skeleton을 확인했다.
- [ ] Jitter 0.005만 사용한 보강 데이터를 만들었다.
- [ ] Jitter 0.015만 사용한 보강 데이터를 만들었다.
- [ ] 두 Candidate의 Original Train·Validation·Seed·모델·204-D Feature 조건을 같게 유지했다.
- [ ] 두 Candidate의 Automatic QA를 수행했다.
- [ ] `mean_joint_step`과 `max_joint_step`을 비교했다.
- [ ] 두 Jitter Skeleton을 직접 비교했다.
- [ ] 각 Candidate의 Human QA 파일을 작성했다.
- [ ] approved된 Jitter만 Feature에 포함했다.
- [ ] 두 Candidate는 `--eval-split val`로 Validation에서만 비교했다.
- [ ] 두 Candidate의 Baseline Validation Accuracy가 같은지 확인했다.
- [ ] Validation Accuracy만 보지 않고 QA와 Validation Confusion Matrix도 확인했다.
- [ ] 더 적절한 Jitter를 Validation 근거로 선택했다.
- [ ] Final Jitter를 Freeze했다.
- [ ] 선택된 한 Tag만 `--eval-split test`로 Test S05에서 최종 평가했다.
- [ ] Test 결과를 본 뒤 Jitter를 다시 변경하지 않았다.
- [ ] 한 개 이상의 실패 사례를 기록했다.
- [ ] `challenge_notes.md`를 작성했다.

# 48. Mini Challenge를 약 3시간으로 진행합니다

| 단계 | 권장 시간 |
|---|---:|
| Challenge 데이터 확인·Keypoint 추출 | 35분 |
| Original Skeleton 확인 | 15분 |
| Jitter 0.005 생성·Automatic/Human QA | 30분 |
| Jitter 0.015 생성·Automatic/Human QA | 30분 |
| 두 Feature·Validation Candidate 평가 | 25분 |
| QA·Validation Confusion Matrix 비교·Final Jitter 선택 | 20분 |
| 선택된 Final Jitter Test·실패 사례 정리 | 5분 |
| `challenge_notes.md` 작성 | 20분 |
| **합계** | **180분** |

# PART 8. 오늘 만든 결과와 교과 11 전체를 정리하고 버전관리합니다

# 49. `day05_notes.md`를 작성합니다

**파일: `day05_notes.md`**

```markdown
# Subject 11 - Day 05

## 1. Main Data

- Train:
- Validation:
- Test:
- Actions:
- Train Subjects:
- Validation Subjects:
- Test Subjects:

## 2. Keypoint Extraction

- Original JSON:
- Detection Rate가 가장 낮은 Sample:
- 원인:

## 3. Main Augmentation

- Translation:
- Scale:
- Jitter:
- Time Warp Fast:
- Time Warp Slow:
- Augmented Sequence 수:

## 4. QA

- Automatic PASS:
- Review:
- Human Rejected Methods:
- 대표 실패 원인:

## 5. Feature

- Feature 수:
- Missing Keypoint 처리 방식:

## 6. Baseline

- Train 수:
- Validation Accuracy:
- Test Accuracy:

## 7. Augmented

- Train 수:
- Validation Accuracy:
- Test Accuracy:

## 8. Performance Delta

- Test Delta:
- 향상 / 유지 / 하락:
- 원인 가설:

## 9. Mini Challenge

- Jitter 0.005 Validation:
- Jitter 0.015 Validation:
- 선택한 Jitter:
- 선택 근거:
- Final Test Accuracy:
- Final Test Confusion Matrix 특징:
- Test 후 Parameter 재조정 여부: 하지 않음

## 10. 오늘의 핵심

-
```

# 50. 교과 11 최종 Method Selection Matrix를 작성합니다

`reports/method_selection_matrix.md`:

```markdown
# Subject 11 - Method Selection Matrix

| 상황 | 우선 고려 방법 | 이유 | 주요 QA |
|---|---|---|---|
| 실제 이미지의 작은 조건 변화가 부족함 | 일반 Augmentation | 기존 Label 유지가 쉬움 | 변형 현실성 |
| 이미지 특징 압축·복원 구조를 확인 | VAE | Latent 표현 학습 | Reconstruction |
| 새로운 이미지 생성 | GAN / WGAN-GP | Data Distribution 학습 | 다양성·Mode Collapse |
| Prompt 기반 이미지 생성 | Diffusion | 조건 생성이 쉬움 | Domain Gap |
| Detection 객체·배경 조합 부족 | Cut-Paste | BBox 자동 생성 가능 | 합성경계·BBox |
| 3D 시점·조명·Mask·Depth 필요 | 3D Rendering | Ground Truth 자동 생성 | Domain Gap |
| Action Sequence 위치·크기·속도 부족 | Keypoint Augmentation | 영상 재촬영 없이 시계열 보강 | Label 의미·Sequence QA |
```

# 51. `final_reflection.md`를 작성합니다

```markdown
# Subject 11 Final Reflection

## 1. 1~5일차에서 사용한 Synthetic / Augmentation 방법은 무엇인가?

## 2. 기존 데이터를 직접 변형하는 방법과 생성모델의 차이는 무엇인가?

## 3. Detection 데이터에서는 Image만 생성하면 부족한 이유는 무엇인가?

## 4. 3D Rendering이 BBox·Mask·Depth를 함께 만들 수 있는 이유는 무엇인가?

## 5. Keypoint Sequence에서 Subject-wise Split이 중요한 이유는 무엇인가?

## 6. 합성데이터를 많이 만들면 무조건 좋은가?

## 7. Automatic QA와 Human QA가 모두 필요한 이유는 무엇인가?

## 8. Metadata가 필요한 이유는 무엇인가?

## 9. 왜 Candidate 조건은 Validation으로 선택하고 Test는 Final 조건을 Freeze한 뒤 마지막에 평가해야 하는가?

## 10. 보강 후 성능이 하락했다면 가장 먼저 무엇을 확인할 것인가?

## 11. 제조 Vision AI에서 사용할 수 있는 교과 11 방법은 무엇인가?

## 12. 산업안전 Action AI에서 사용할 수 있는 교과 11 방법은 무엇인가?

## 13. 앞으로 합성데이터를 만들 때 가장 먼저 확인할 것은 무엇인가?
```

# 52. README를 작성합니다

````markdown
# Subject 11 - Day 05
## KTH Keypoint Sequence Augmentation · QA · Evaluation

### Main Data

```text
Train 24
S11 S12 S13 S14

Validation 12
S19 S20

Test 12
S02 S03

Actions
boxing
handwaving
handclapping
```

### Mini Challenge

```text
Train 12
S15 S16

Validation 6
S21

Test 6
S05
```

### Pipeline

```text
KTH Video
→ YOLO Pose
→ COCO17 Keypoint
→ Sequence Augmentation
→ QA
→ Feature
→ Baseline vs Augmented
→ Validation으로 Candidate 판단
→ Final 조건 Freeze
→ Test 최종 평가
```

### Scripts

```text
00_check_day05.py
01_extract_keypoints.py
02_visualize_keypoints.py
03_augment_keypoints.py
04_qa_keypoints.py
05_build_features.py
06_compare_baseline_augmented.py
```

### Main Command Order

```bash
python scripts/00_check_day05.py --asset-set all

python scripts/01_extract_keypoints.py \
  --asset-set main \
  --split train \
  --demo

python scripts/01_extract_keypoints.py \
  --asset-set main \
  --split all

python scripts/03_augment_keypoints.py \
  --asset-set main \
  --tag main_aug \
  --jitter-sigma 0.005

python scripts/04_qa_keypoints.py \
  --asset-set main \
  --tag main_aug

# Skeleton을 확인한 뒤
# reports/main/main_aug_human_qa.csv 작성

python scripts/05_build_features.py \
  --asset-set main \
  --tag main_aug

python scripts/06_compare_baseline_augmented.py \
  --asset-set main \
  --tag main_aug \
  --eval-split both
```

### 가장 중요한 원칙

```text
Synthetic / Augmented 생성
→ QA
→ Approved Data
→ Train에만 추가
→ Validation으로 조건 판단
→ Final 조건 Freeze
→ Test는 마지막 최종 평가
```
````

# 53. `.gitignore`를 작성합니다

```gitignore
# Python
__pycache__/
*.pyc

# Editor
.vscode/

# Environment
.venv/

# Provided Videos
data/day05_student_data/
*.avi
*.mp4
*.mov

# Generated Keypoints / Features
data/keypoints/
data/features/

# Pose / Trained Models
models/
*.pt
*.pth
*.joblib

# Generated Results
results/

# OS
.DS_Store
Thumbs.db
```

영상·Keypoint·모델 같은 대용량 결과는 Git에 올리지 않고 코드와 Report 중심으로 기록합니다.

# 54. 기존 Git 저장소를 그대로 사용합니다

교과 11 Root로 이동합니다.

```bash
cd ~/ai_vision/subject11_synthetic_data
```

```bash
git status
```

```bash
git add day05_keypoint_sequence
```

```bash
git status
```

다음 항목이 Stage에 들어가지 않았는지 확인합니다.

```text
day05_student_data
AVI
Keypoint JSON
Feature CSV
YOLO Pose Model
joblib Model
Generated MP4
```

Commit:

```bash
git commit -m "feat: add day05 keypoint sequence augmentation lab"
```

기관에서 허용된 원격 저장소가 연결되어 있을 때만 Push합니다.

```bash
git push
```

# 55. 하루가 끝났을 때 자가 체크합니다

| 확인사항 | 완료 |
|---|:---:|
| 1~4일차 프로젝트를 이어서 사용했다 | □ |
| 1일차 `.venv`를 재사용했다 | □ |
| `day05_student_data`를 올바르게 배치했다 | □ |
| Main과 Challenge를 구분했다 | □ |
| Main 24/12/12 수량을 확인했다 | □ |
| Challenge 12/6/6 수량을 확인했다 | □ |
| Train·Val·Test Subject가 겹치지 않음을 확인했다 | □ |
| `boxing`, `handwaving`, `handclapping` Class를 확인했다 | □ |
| 대표 영상 3개에서 COCO17 Keypoint를 추출했다 | □ |
| Skeleton Preview를 확인했다 | □ |
| Main 48개 Keypoint JSON을 만들었다 | □ |
| Augmentation은 Main Train에만 적용했다 | □ |
| Translation을 이해했다 | □ |
| Scale을 이해했다 | □ |
| Jitter를 이해했다 | □ |
| Time Warp를 이해했다 | □ |
| Main Train에서 5종 보강을 생성했다 | □ |
| Metadata를 기록했다 | □ |
| Automatic QA를 수행했다 | □ |
| Human QA를 수행했다 | □ |
| rejected Sequence를 Train 후보에서 제외했다 | □ |
| Missing Keypoint를 보간하는 이유를 이해했다 | □ |
| 204-D Feature를 만들었다 | □ |
| Baseline 모델을 학습했다 | □ |
| Augmented 모델을 학습했다 | □ |
| Main 최종 비교에서 동일한 Original Validation/Test를 사용했다 | □ |
| Main Test 결과를 보고 Augmentation Parameter를 다시 조정하지 않았다 | □ |
| Accuracy를 비교했다 | □ |
| Confusion Matrix를 확인했다 | □ |
| Mini Challenge에서 Jitter 하나만 변경했다 | □ |
| Mini Challenge Candidate는 Validation으로만 비교했다 | □ |
| Final Jitter를 Freeze한 뒤 선택된 조건만 Test에서 평가했다 | □ |
| Challenge QA와 성능을 함께 비교했다 | □ |
| 실패 사례를 기록했다 | □ |
| Method Selection Matrix를 작성했다 | □ |
| Final Reflection을 작성했다 | □ |
| README를 작성했다 | □ |
| Git Commit을 완료했다 | □ |

# 56. 5일차와 교과 11 전체를 마무리합니다

5일차의 핵심은 사람 동작 영상을 Pose로 바꾸는 것 자체가 아닙니다.

```text
KTH Video
→ COCO17 Keypoint
→ Train Sequence Augmentation
→ Automatic QA
→ Human QA
→ Approved Augmented Train
→ Same Validation / Test
→ Performance Comparison
```

여기에서 반드시 기억할 원칙은 세 가지입니다.

## ① 사람 동작 데이터는 Subject 단위로 분리합니다

```text
Train Subject
→ Original + Augmented

Validation Subject
→ Original only

Test Subject
→ Original only
```

같은 사람의 원본과 보강 결과가 Train과 Test에 나뉘면 공정한 평가가 어려워집니다.

## ② Sequence Augmentation은 Action 의미를 유지해야 합니다

```text
좌표를 바꿀 수 있음
≠
같은 Action을 유지함
```

그래서 모든 Sequence의 구조는 Automatic QA로 검사하고, 대표 Sample과 Review 대상은 Human QA로 확인합니다.

## ③ 조건 선택은 Validation, 최종 평가는 Test로 구분합니다

```text
Baseline / Augmented
        ↓
Same Validation
        ↓
조건 비교·선택
        ↓
Final 조건 Freeze
        ↓
Same Test
        ↓
최종 성능 보고
```

Test 결과를 본 뒤 같은 Test에 맞추어 Parameter를 다시 바꾸지 않습니다.

보강 데이터의 수가 많아졌다는 사실만으로 좋은 Dataset이라고 판단하지 않습니다.

마지막으로 1~5일차를 연결하면 다음과 같습니다.

```text
Day 01
Image Augmentation + VAE

Day 02
GAN + WGAN-GP + Diffusion

Day 03
Cut-Paste + BBox

Day 04
3D Rendering + BBox / Mask / Depth

Day 05
Keypoint Sequence Augmentation + 성능검증
```

데이터 형식은 달라도 교과 11에서 반복한 기준은 같습니다.

```text
부족한 조건 확인
        ↓
Synthetic / Augmented Data 생성
        ↓
QA
        ↓
Metadata
        ↓
사용할 데이터 승인
        ↓
Train에 추가
        ↓
Validation으로 조건 판단
        ↓
Final 조건 Freeze
        ↓
Test로 최종 효과 검증
```

이 흐름을 이해하면 새로운 프로젝트에서도 “어떤 합성기법을 사용할 것인가?”보다 먼저 **“어떤 데이터가 부족하며, 만든 데이터가 실제 성능에 도움이 되는가?”**를 질문할 수 있습니다.

# 다음 교과와의 연결

교과 11에서 배운 방법은 이후 프로젝트의 데이터 부족 조건에 따라 선택적으로 사용할 수 있습니다.

```text
제조 Vision AI
→ 이미지·Detection 조건 부족
→ Image Augmentation / Cut-Paste / 3D Rendering 검토

산업안전 Action AI
→ 사람·거리·속도·Pose 조건 부족
→ Keypoint Sequence Augmentation 검토
```
