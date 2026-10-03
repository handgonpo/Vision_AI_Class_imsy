## 오늘의 핵심 질문

> **3D 공간에서 객체·카메라·조명의 조건을 프로그램이 알고 있다면, 한 번의 Rendering으로 RGB 이미지뿐 아니라 BBox·Segmentation·Depth 같은 정답 데이터까지 함께 만들 수 있을까?**

3일차에는 2D Background와 투명 Object PNG를 합성하여 Detection용 데이터를 만들었습니다.

```text
3일차

Background JPG
+
RGBA Object PNG
        ↓
Position / Scale / Rotation
        ↓
Cut-Paste
        ↓
Synthetic RGB
+
YOLO BBox
        ↓
QA
```

4일차에는 이 생각을 **3D 공간**으로 확장합니다.

```text
4일차

3D Object
+
3D Position / Rotation
+
Camera
+
Light
        ↓
BlenderProc
        ↓
Rendering
        ↓
RGB
+
COCO BBox
+
Semantic Mask
+
Instance Mask
+
Depth
        ↓
QA
        ↓
Real Reference와 비교
```

수업 전에 준비된 **YCB-V 3D Object와 실제 촬영 Reference Image**를 사용합니다.

본 실습에서는 다음 두 Object를 함께 사용합니다.

```text
cracker_box
   +
mustard_bottle
```

하루 마지막 Mini Challenge에서는 본 실습에 사용하지 않은 다음 Object를 사용합니다.

```text
power_drill
```

오늘도 1~3일차와 같은 패턴으로 진행합니다.

```text
준비된 Source 확인
        ↓
한 번 실행
        ↓
결과 확인
        ↓
여러 조건 자동 생성
        ↓
Metadata
        ↓
Automatic QA
        ↓
Human QA
        ↓
Real vs Synthetic 비교
        ↓
Mini Challenge
        ↓
한 변수만 변경
        ↓
결과 해석
```

오늘의 목적은 3D Rendering 이미지를 많이 만드는 것이 아닙니다.

**3D Synthetic Data가 어떻게 만들어지고, Ground Truth가 왜 자동으로 생성될 수 있으며, 실제 사진과 얼마나 다른지를 확인하는 전체 과정**을 경험하는 것이 목표입니다.

---

# 오늘의 수업 목표

수업이 끝나면 다음 내용을 설명하고 직접 실행할 수 있어야 합니다.

- 3일차의 2D Cut-Paste와 4일차의 3D Rendering 차이를 설명할 수 있습니다.
- 1~3일차와 같은 교과 11 프로젝트를 이어서 사용할 수 있습니다.
- 1일차에서 만든 `.venv`를 그대로 재사용할 수 있습니다.
- 4일차에 제공된 `day04_student_data`의 `main`과 `challenge` 역할을 구분할 수 있습니다.
- YCB-V의 3D Model과 실제 촬영 Reference Image를 구분할 수 있습니다.
- BOP Object ID와 수업용 Class ID가 서로 다른 값이라는 점을 설명할 수 있습니다.
- `cracker_box`, `mustard_bottle`, `power_drill`의 고정 Class ID를 사용할 수 있습니다.
- Blender와 BlenderProc의 역할 차이를 설명할 수 있습니다.
- BlenderProc Script는 일반 `python`이 아니라 `blenderproc run`으로 실행해야 한다는 점을 이해할 수 있습니다.
- BOP 형식의 `.ply` 3D Object를 BlenderProc로 불러올 수 있습니다.
- 3D Object의 위치·회전과 Camera·Light의 역할을 설명할 수 있습니다.
- 하나의 3D Scene을 Rendering할 수 있습니다.
- 한 번의 Scene에서 RGB·Depth·Semantic Segmentation·Instance Segmentation을 생성할 수 있습니다.
- COCO Annotation에서 BBox가 자동 생성되는 흐름을 설명할 수 있습니다.
- Random Seed를 고정하고 Object·Camera·Light 조건을 바꾸어 여러 Synthetic Sample을 생성할 수 있습니다.
- 생성 조건을 Metadata JSON과 CSV로 기록할 수 있습니다.
- HDF5에 저장된 여러 Ground Truth를 Preview Image로 확인할 수 있습니다.
- COCO BBox와 HDF5 결과를 Automatic QA로 검사할 수 있습니다.
- Human QA가 Automatic QA와 다른 문제를 확인한다는 점을 설명할 수 있습니다.
- YCB-V 실제 Reference Image와 Synthetic Render를 비교하여 Domain Gap을 설명할 수 있습니다.
- Mini Challenge에서 `power_drill`로 같은 Pipeline을 스스로 반복할 수 있습니다.
- Mini Challenge에서 Camera Distance 하나만 변경하고 다른 Random 조건이 동일했는지 검증한 뒤 BBox 크기와 Domain Gap 변화를 비교할 수 있습니다.
- 오늘 만든 코드·실험 조건·결과를 README와 Git에 정리할 수 있습니다.

---

# 오늘 수업의 전체 흐름

![[Pasted image 20260908150044.png]]

4일차에는 준비된 **3D 모델·카메라·조명**을 이용해 BlenderProc로 한 장의 장면을 먼저 렌더링하고, RGB·BBox·Mask·Depth 같은 정답 데이터가 함께 만들어지는 원리를 이해합니다. 

그다음 위치·회전·카메라·조명 조건을 바꾸어 여러 장의 합성데이터를 자동 생성하고, 생성 조건을 Metadata로 기록합니다.  

마지막에는 생성 결과를 **QA로 검수하고 실제 이미지와 비교해 Domain Gap을 확인한 뒤**, Mini Challenge에서 다른 3D 객체로 같은 흐름을 스스로 반복합니다.

```text
3일차 프로젝트 확인
        ↓
교과 11 Root 이동
        ↓
4일차 폴더 생성
        ↓
1일차 .venv 재사용
        ↓
BlenderProc 2.7 설치 상태 확인
        ↓
blenderproc --help
        ↓
blenderproc quickstart
        ↓
Quickstart HDF5 확인
        ↓
제공된 day04_student_data 배치
        ↓

[본 실습 Source 확인]

main
├─ cracker_box 3D Model
├─ mustard_bottle 3D Model
├─ cracker_box Real 10장
└─ mustard_bottle Real 10장
        ↓
Object ID / Class ID 확인
        ↓
YCB-V / BOP Model 경로 확인
        ↓
입력 데이터 자동 검사
        ↓
Real Reference Preview
        ↓

[실험 A — Single 3D Scene]

cracker_box
+
mustard_bottle
+
Plane
+
Camera
+
Light
        ↓
BlenderProc
        ↓
RGB
Depth
Semantic Mask
Instance Mask
COCO BBox
        ↓
HDF5 Preview
        ↓
COCO Annotation 확인
        ↓

[실험 B — Randomized Dataset]

Object Position / Rotation
        +
Camera Distance / Position
        +
Light Position / Energy
        ↓
Seed 고정
        ↓
Random Sample 1장 Smoke Test
        ↓
여러 Scene 자동 Rendering
        ↓
HDF5
COCO
Metadata
        ↓

[QA · Domain Gap]

Automatic QA
        ↓
Human QA
        ↓
Real Reference
VS
Synthetic Render
        ↓
Domain Gap 분석
        ↓

[Mini Challenge]

challenge
└─ power_drill
        ↓
Single Render
        ↓
Baseline Camera Distance
        ↓
Far Camera Distance
        ↓
다른 조건은 유지
        ↓
BBox 크기 비교
        ↓
QA
        ↓
Real vs Synthetic 비교
        ↓
결론
        ↓

README · Git 정리
        ↓
자가 체크
```

> **오늘의 데이터 운영 원칙**
>
> 본 실습에서는 `main` 데이터만 사용합니다.
>
> Mini Challenge에서는 `challenge` 데이터만 사용합니다.
>
> 원본 YCB-V 전체 데이터는 다시 다운로드하거나 다시 분리하지 않습니다.
>
> 제공된 `day04_student_data`는 수업 전에 필요한 3D Object와 Real Reference만 추출한 데이터입니다.
>
> 오늘 생성한 Synthetic Render는 자동으로 학습 승인하지 않습니다.
>
> `Rendering 성공 → QA → Real과 비교 → 사용 여부 판단` 순서를 지킵니다.

---

# 오늘 사용할 실제 데이터

## 1. 제공되는 4일차 학생용 데이터

```text
day04_student_data/
│
├─ main/
│  ├─ bop/
│  │  └─ ycbv/
│  │     ├─ camera_uw.json
│  │     ├─ camera_cmu.json
│  │     ├─ dataset_info.md
│  │     └─ models/
│  │        ├─ models_info.json
│  │        ├─ obj_000002.ply
│  │        └─ obj_000005.ply
│  ├─ model_previews/
│  │  ├─ cracker_box_model.png
│  │  └─ mustard_bottle_model.png
│  ├─ real_reference/
│  │  ├─ cracker_box/
│  │  │  └─ cracker_box_real_001.png ~ 010
│  │  └─ mustard_bottle/
│  │     └─ mustard_bottle_real_001.png ~ 010
│  └─ data_config.json
│
├─ challenge/
│  ├─ bop/
│  │  └─ ycbv/
│  │     ├─ camera_uw.json
│  │     ├─ camera_cmu.json
│  │     ├─ dataset_info.md
│  │     └─ models/
│  │        ├─ models_info.json
│  │        └─ obj_000015.ply
│  ├─ model_previews/
│  │  └─ power_drill_model.png
│  ├─ real_reference/
│  │  └─ power_drill/
│  │     └─ power_drill_real_001.png ~ 010
│  └─ data_config.json
│
├─ classes.txt
├─ object_map.json
└─ README_DATA.txt
```

원본 YCB-V 전체 `test`, 전체 `models`, 데이터 추출용 Python Script는 오늘 학생 실습에서 사용하지 않습니다.

## 2. 본 실습 `main`

```text
cracker_box
→ YCB: 003_cracker_box
→ BOP Object ID: 2
→ Lesson Class ID: 1
→ Real Reference: 10장

mustard_bottle
→ YCB: 006_mustard_bottle
→ BOP Object ID: 5
→ Lesson Class ID: 2
→ Real Reference: 10장
```

3D Model 파일명은 BOP 구조를 유지합니다.

```text
obj_000002.ply
obj_000005.ply
```

학생이 임의로 파일명을 변경하지 않습니다.

## 3. Mini Challenge `challenge`

```text
power_drill
→ YCB: 035_power_drill
→ BOP Object ID: 15
→ Lesson Class ID: 3
→ Real Reference: 10장
```

## 4. Class ID는 수업 전체에서 고정합니다

```text
0 → background / floor
1 → cracker_box
2 → mustard_bottle
3 → power_drill
```

BOP Object ID와 Lesson Class ID는 목적이 다릅니다.

```text
BOP ID
→ 어떤 3D Model을 불러올지 찾는 번호

Lesson Class ID
→ 오늘 생성하는 정답 데이터에서 사용할 Class 번호
```

---

# 오늘 배울 기술 스택

| 기술 | 오늘 하는 일 | 쉽게 말하면 |
|---|---|---|
| Python | 경로·검사·Metadata·QA | 4일차 전체 자동화 |
| pathlib | 폴더·파일 경로 관리 | Main과 Challenge 연결 |
| BlenderProc | 3D Scene 자동 Rendering | Blender를 Python으로 제어 |
| BOP Loader | YCB-V `.ply` Model Loading | ID로 3D 모델 불러오기 |
| YCB-V | 3D Model + 실제 촬영 Reference | Real과 Synthetic 비교 |
| Camera | 촬영 위치·거리·시점 | 화면 구성 제어 |
| Light | 위치·밝기 | 조명 조건 제어 |
| 3D Rotation | 객체 방향 변경 | 여러 방향의 Object 생성 |
| RGB | 모델 입력 이미지 | 가상 Camera 결과 |
| Depth | Camera와 Pixel 거리 | 거리 정보 |
| Semantic Segmentation | Class별 Mask | 종류별 영역 |
| Instance Segmentation | Object별 Mask | 객체별 영역 |
| COCO Annotation | BBox·Mask 정보 | Detection/Segmentation 정답 |
| HDF5 | 여러 Rendering 결과 저장 | 결과 묶음 |
| Randomization | Camera·Light·Object 변화 | 다양한 Synthetic Data 생성 |
| Metadata | Seed·Camera·Light·Object 기록 | 생성 조건 추적 |
| Automatic QA | 파일·BBox·Ground Truth 검사 | 구조 오류 자동 확인 |
| Human QA | 현실성·Domain Gap 판단 | 사람이 품질 확인 |
| Git | 코드·설명 기록 | 재현 가능한 결과 관리 |

---

# 오늘의 8시간 운영 구성

| 단계 | 권장 시간 | 내용 |
|---|---:|---|
| 본 실습 1 | 45분 | 3일차 연결·환경·데이터 검사 |
| 본 실습 2 | 40분 | YCB-V·BOP ID·3D Ground Truth 이해 |
| 본 실습 3 | 60분 | Single Rendering·HDF5·COCO 확인 |
| 본 실습 4 | 70분 | Randomized Rendering·Batch·Metadata |
| 본 실습 5 | 55분 | Automatic QA·Human QA·Domain Gap |
| 본 실습 정리 | 30분 | 3일차 비교·실패 조건 추적 |
| Mini Challenge | 180분 | power_drill Camera Distance 한 변수 실험 |
| **합계** | **480분** | **8시간** |

Rendering 시간은 PC 성능에 따라 달라질 수 있으므로 `Single Render → Random Smoke Test → Batch Rendering` 순서를 지킵니다.

---

# PART 1. 3일차 프로젝트에서 4일차를 이어서 시작합니다

# 1. 1~4일차가 어떻게 연결되는지 확인합니다

```text
1일차
Real Image → Augmentation / VAE

2일차
Real Train → DCGAN / WGAN-GP
Pretrained Diffusion → Synthetic Image

3일차
2D Background + RGBA Object
→ Cut-Paste
→ RGB + BBox

4일차
3D Object + Camera + Light
→ Rendering
→ RGB + BBox + Mask + Depth
```

3일차와 4일차의 공통점은 다음입니다.

```text
프로그램이 Object의 위치를 알고 있음
        ↓
정답 Label 자동 생성 가능
```

차이는:

```text
3일차 → 2D Pixel 공간
4일차 → 3D World 공간
```

입니다.

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
```

4일차 폴더를 만듭니다.

```bash
mkdir -p day04_3d_segmentation
cd day04_3d_segmentation
```

# 3. 4일차 폴더와 파일을 준비합니다

```bash
mkdir -p \
src \
scripts \
data \
results/main \
results/challenge \
reports/main \
reports/challenge
```

```bash
touch requirements_day04.txt
touch src/day04_common.py

touch scripts/00_check_day04.py
touch scripts/01_preview_real_reference.py
touch scripts/02_render_single_scene.py
touch scripts/03_export_hdf5_preview.py
touch scripts/04_render_random_sample.py
touch scripts/05_generate_dataset.py
touch scripts/06_collect_metadata.py
touch scripts/07_qa_3d_dataset.py
touch scripts/08_compare_real_synthetic.py
touch scripts/09_compare_experiments.py

touch day04_notes.md
touch README.md
touch .gitignore
```

```bash
tree -L 2
```

예상:

```text
day04_3d_segmentation/
├─ data/
├─ reports/
├─ results/
├─ scripts/
├─ src/
├─ requirements_day04.txt
├─ day04_notes.md
├─ README.md
└─ .gitignore
```

### 지금 무엇을 한 것인가요?

```text
3일차 결과는 그대로 보관
        ↓
4일차 전용 폴더 생성
        ↓
Source / Rendering / QA / Reports 분리
```

# 4. 제공된 `day04_student_data`를 배치합니다

수업에서 제공된 `day04_student_data`를 다음 위치에 복사합니다.

```text
day04_3d_segmentation/
└─ data/
   └─ day04_student_data/
```

확인합니다.

```bash
tree data/day04_student_data -L 5
```

# 5. 1일차 가상환경을 그대로 사용합니다

```bash
source ../day01_image_aug_vae/.venv/bin/activate
```

```bash
which python
```

새로운 가상환경을 만들지 않습니다.

# 6. Blender와 BlenderProc의 역할을 구분합니다

```text
Blender
→ 3D Scene을 만들고 볼 수 있는 프로그램

BlenderProc
→ Blender를 Python으로 자동 제어해
  AI 학습용 Synthetic Data를 만드는 도구
```

오늘은 복잡한 3D 모델링을 하지 않습니다.

이미 준비된 YCB-V 3D Model을 불러옵니다.

실행 방식도 구분합니다.

```text
일반 Python Script
→ python scripts/00_check_day04.py

BlenderProc Script
→ blenderproc run scripts/02_render_single_scene.py
```

# 7. 4일차 추가 라이브러리를 설치합니다

`requirements_day04.txt`:

```text
blenderproc==2.7.0
h5py
matplotlib
numpy
pandas
Pillow
```

설치:

```bash
pip install -r requirements_day04.txt
```

먼저 일반 Python 환경에서 필요한 패키지가 보이는지 확인합니다.

```bash
python -c "import h5py, matplotlib, numpy, pandas, PIL; print('Day04 Python libraries OK')"
```

그다음 BlenderProc CLI가 정상인지 확인합니다.

```bash
blenderproc --help
```

이 명령이 정상적으로 표시되기 전에는 Rendering Script로 넘어가지 않습니다.

환경 기록:

```bash
pip freeze > reports/environment_freeze_day04.txt
```

# 8. BlenderProc Quickstart로 Rendering 환경을 확인합니다

```bash
blenderproc quickstart
```

결과 확인:

```bash
find output -maxdepth 2 -type f
```

HDF5 시각화:

```bash
blenderproc vis hdf5 output/0.hdf5
```

정상 확인 후:

```bash
rm -rf output
```

오늘 Rendering 코드는 BlenderProc 2.7 API를 기준으로 작성합니다. 수업 중 임의로 BlenderProc 버전을 바꾸지 않습니다.

수업 중 외부에서 BOP Toolkit이나 YCB-V 같은 대용량 Asset을 새로 내려받는 흐름에 의존하지 않습니다. 인터넷 사용이 제한된 환경에서는 수업 전에 준비된 BlenderProc 환경과 `day04_student_data`를 그대로 사용합니다.

4일차는 다음 순서로 실행환경을 확인합니다.

```text
일반 Python 라이브러리 확인
        ↓
blenderproc --help
        ↓
blenderproc quickstart
        ↓
Quickstart 0.hdf5 확인
        ↓
day04_student_data 확인
        ↓
YCB-V / BOP Model 경로 확인
        ↓
Main Single Render
        ↓
HDF5 / COCO 확인
        ↓
Random Sample 1장 Smoke Test
        ↓
Batch Rendering
```

```text
Quickstart 성공
→ BlenderProc 기본 Rendering 확인

첫 BOP Single Render 성공
→ YCB-V Model Loader와 Ground Truth 저장 확인

Random Smoke Test 성공
→ Randomization과 Metadata 저장 확인

그 다음
→ Batch Rendering
```

# 9. Main과 Challenge의 공통 경로 코드를 작성합니다

**파일: `src/day04_common.py`**

```python
from __future__ import annotations

import json
from pathlib import Path


ROOT = Path(__file__).resolve().parents[1]

DATA_ROOT = (
    ROOT
    / "data"
    / "day04_student_data"
)


def load_json(path: Path):
    if not path.exists():
        raise FileNotFoundError(path)

    with path.open(
        "r",
        encoding="utf-8",
    ) as file:
        return json.load(file)


def asset_root(asset_set: str) -> Path:
    if asset_set not in {
        "main",
        "challenge",
    }:
        raise ValueError(
            f"지원하지 않는 asset_set: {asset_set}"
        )

    return DATA_ROOT / asset_set


def data_config(asset_set: str) -> dict:
    return load_json(
        asset_root(asset_set)
        / "data_config.json"
    )


def bop_dataset_path(asset_set: str) -> Path:
    return (
        asset_root(asset_set)
        / "bop"
        / "ycbv"
    )


def object_map() -> dict:
    return load_json(
        DATA_ROOT
        / "object_map.json"
    )


def lesson_objects(
    asset_set: str,
) -> list[dict]:
    mapping = object_map()

    return [
        item
        for item in mapping["objects"]
        if item["usage"] == asset_set
    ]


def real_reference_root(
    asset_set: str,
) -> Path:
    return (
        asset_root(asset_set)
        / "real_reference"
    )


def result_root(
    asset_set: str,
    tag: str,
) -> Path:
    path = (
        ROOT
        / "results"
        / asset_set
        / tag
    )

    path.mkdir(
        parents=True,
        exist_ok=True,
    )

    return path


def report_root(
    asset_set: str,
) -> Path:
    path = (
        ROOT
        / "reports"
        / asset_set
    )

    path.mkdir(
        parents=True,
        exist_ok=True,
    )

    return path
```

### 이 코드는 왜 작성하나요?

본 실습과 Mini Challenge에서 같은 Script를 재사용하기 위해 작성합니다.

```text
--asset-set main
→ cracker_box + mustard_bottle

--asset-set challenge
→ power_drill
```

### 의사코드로 읽어보기

```text
4일차 Root를 찾는다
        ↓
day04_student_data 위치를 찾는다
        ↓
main 또는 challenge를 선택한다
        ↓
data_config.json을 읽는다
        ↓
BOP ycbv 경로를 만든다
        ↓
object_map.json을 읽는다
        ↓
해당 Set의 Object 정보를 찾는다
        ↓
Real / results / reports 경로를 만든다
```

# 10. 실제 수업 데이터가 정확한지 자동으로 검사합니다

**파일: `scripts/00_check_day04.py`**

```python
from __future__ import annotations

import argparse
import sys
from pathlib import Path

from PIL import Image


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day04_common import (
    asset_root,
    bop_dataset_path,
    data_config,
    lesson_objects,
    real_reference_root,
)


EXPECTED = {
    "main": {
        "object_count": 2,
        "real_per_class": 10,
    },
    "challenge": {
        "object_count": 1,
        "real_per_class": 10,
    },
}


def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--asset-set",
        default="main",
        choices=["main", "challenge"],
    )
    return parser.parse_args()


def image_files(directory: Path) -> list[Path]:
    valid = {".png", ".jpg", ".jpeg"}

    if not directory.exists():
        return []

    return sorted(
        path
        for path in directory.iterdir()
        if path.is_file()
        and path.suffix.lower() in valid
    )


def main() -> None:
    args = parse_args()

    expected = EXPECTED[args.asset_set]
    source_root = asset_root(args.asset_set)
    bop_root = bop_dataset_path(args.asset_set)
    config = data_config(args.asset_set)
    objects = lesson_objects(args.asset_set)

    print("=" * 64)
    print("Subject 11 - Day 04 Data Check")
    print("=" * 64)
    print("Asset Set:", args.asset_set)
    print("Source   :", source_root)

    if len(objects) != expected["object_count"]:
        raise RuntimeError(
            "Object Mapping 수가 수업 기준과 다릅니다."
        )

    if not (
        bop_root
        / "models"
        / "models_info.json"
    ).exists():
        raise FileNotFoundError(
            "models_info.json이 없습니다."
        )

    config_ids = [
        int(value)
        for value in config["bop_obj_ids"]
    ]

    reference_root = real_reference_root(
        args.asset_set
    )

    for item in objects:
        bop_id = int(item["bop_obj_id"])
        class_name = item["class_name"]
        lesson_id = int(
            item["lesson_class_id"]
        )

        if bop_id not in config_ids:
            raise RuntimeError(
                f"{class_name}의 BOP ID가 "
                "data_config.json에 없습니다."
            )

        model_path = (
            bop_root
            / "models"
            / f"obj_{bop_id:06d}.ply"
        )

        if not model_path.exists():
            raise FileNotFoundError(model_path)

        real_dir = (
            reference_root
            / class_name
        )

        real_files = image_files(real_dir)

        if len(real_files) != (
            expected["real_per_class"]
        ):
            raise RuntimeError(
                f"{class_name} Real Reference 수가 "
                "수업 기준과 다릅니다.\n"
                f"기대: {expected['real_per_class']}\n"
                f"현재: {len(real_files)}"
            )

        unreadable = []

        for path in real_files:
            try:
                with Image.open(path) as image:
                    image.verify()
            except Exception:
                unreadable.append(path.name)

        if unreadable:
            raise RuntimeError(
                f"{class_name}에서 읽을 수 없는 "
                "이미지가 있습니다: "
                + ", ".join(unreadable)
            )

        print("-" * 64)
        print("Class Name      :", class_name)
        print("Lesson Class ID :", lesson_id)
        print("BOP Object ID   :", bop_id)
        print("3D Model        :", model_path.name)
        print("Real Reference  :", len(real_files))

    print("=" * 64)
    print("Day 04 data check: READY")


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

Rendering 전에 `3D Model 누락`, `Object ID 오류`, `Real Reference 수량 오류`, `main/challenge 혼합`을 먼저 찾기 위해 작성합니다.

### 의사코드로 읽어보기

```text
main 또는 challenge를 선택한다
        ↓
Object Mapping을 읽는다
        ↓
BOP Model 파일을 확인한다
        ↓
Class별 Real Reference를 확인한다
        ↓
실제 사진을 읽어본다
        ↓
Object ID / Class ID를 출력한다
        ↓
정상이면 READY
```

실행:

```bash
python scripts/00_check_day04.py \
  --asset-set main
```

---

# PART 2. 3D Object와 Ground Truth 구조를 이해합니다

# 11. 실제 촬영 Reference Image를 확인합니다

**파일: `scripts/01_preview_real_reference.py`**

```python
from __future__ import annotations

import argparse
import sys
from pathlib import Path

from PIL import Image, ImageDraw, ImageOps


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day04_common import (
    lesson_objects,
    real_reference_root,
    result_root,
)


TILE_W = 220
TILE_H = 180
LABEL_H = 36


def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--asset-set",
        default="main",
        choices=["main", "challenge"],
    )
    return parser.parse_args()


def list_images(directory: Path):
    return sorted(
        path
        for path in directory.iterdir()
        if path.is_file()
        and path.suffix.lower()
        in {".png", ".jpg", ".jpeg"}
    )


def main() -> None:
    args = parse_args()

    objects = lesson_objects(args.asset_set)
    real_root = real_reference_root(
        args.asset_set
    )

    rows = []

    for item in objects:
        class_name = item["class_name"]

        rows.append(
            (
                class_name,
                list_images(
                    real_root / class_name
                ),
            )
        )

    columns = 5

    total_rows = sum(
        (len(files) + columns - 1)
        // columns
        for _, files in rows
    )

    canvas = Image.new(
        "RGB",
        (
            TILE_W * columns,
            total_rows
            * (TILE_H + LABEL_H),
        ),
        "white",
    )

    draw = ImageDraw.Draw(canvas)
    row_offset = 0

    for class_name, files in rows:
        class_rows = (
            len(files) + columns - 1
        ) // columns

        for index, path in enumerate(files):
            row = (
                row_offset
                + index // columns
            )

            column = index % columns

            x = column * TILE_W
            y = row * (
                TILE_H + LABEL_H
            )

            with Image.open(path) as image:
                image = (
                    ImageOps
                    .exif_transpose(image)
                    .convert("RGB")
                )

                image.thumbnail(
                    (
                        TILE_W - 12,
                        TILE_H - 12,
                    ),
                    Image.Resampling.LANCZOS,
                )

                px = (
                    x
                    + (TILE_W - image.width)
                    // 2
                )

                py = (
                    y
                    + (TILE_H - image.height)
                    // 2
                )

                canvas.paste(
                    image,
                    (px, py),
                )

            draw.text(
                (
                    x + 5,
                    y + TILE_H + 7,
                ),
                f"{class_name}: {path.name}",
                fill="black",
            )

        row_offset += class_rows

    output_dir = result_root(
        args.asset_set,
        "source"
    )

    output_path = (
        output_dir
        / "real_reference_grid.jpg"
    )

    canvas.save(
        output_path,
        quality=92,
    )

    print("Saved:", output_path)


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

Synthetic Render를 만들기 전에 실제 YCB-V 촬영 장면을 기준으로 확보하기 위해 작성합니다.

### 의사코드로 읽어보기

```text
해당 Set의 Class를 찾는다
        ↓
Class별 Real Reference를 읽는다
        ↓
Preview 크기로 줄인다
        ↓
한 화면에 Grid로 배치한다
        ↓
파일명과 Class를 표시한다
        ↓
real_reference_grid.jpg로 저장한다
```

실행:

```bash
python scripts/01_preview_real_reference.py \
  --asset-set main
```

# 12. `.ply`와 BOP Object ID를 이해합니다

```text
obj_000002.ply
→ cracker_box

obj_000005.ply
→ mustard_bottle

obj_000015.ply
→ power_drill
```

`.ply`는 3D Mesh 형식입니다.

오늘은 이 파일을 직접 수정하지 않습니다.

# 13. 3D Scene의 구성요소를 이해합니다

```text
Object
→ 촬영 대상

Plane
→ 바닥

Camera
→ 촬영 위치와 시점

Light
→ 빛의 위치와 세기
```

3일차:

```text
2D x, y
```

4일차:

```text
3D x, y, z
```

입니다.

## 오늘의 Camera는 어떤 Camera인가요?

제공 데이터에는 YCB-V의 `camera_uw.json`, `camera_cmu.json`도 포함되어 있습니다.

하지만 오늘은 실제 YCB-V 촬영 장면을 그대로 복제하는 Camera Calibration 실습이 아니라 **3D Object·Camera·Light를 직접 구성하고 Ground Truth가 자동 생성되는 구조를 이해하는 실습**입니다.

따라서 Rendering은 BlenderProc의 가상 Camera를 사용하고 해상도를 `512 × 512`로 고정합니다.

```text
YCB-V 3D Model
+
직접 구성한 가상 Camera / Light
→ Synthetic Scene

YCB-V Real Reference
→ Domain Gap 비교 기준
```

즉 Camera JSON은 원본 데이터의 Camera 정보를 확인하기 위해 보관하지만 오늘 Rendering Camera에 직접 적용하지 않습니다. 이 차이도 Domain Gap 원인 중 하나가 될 수 있습니다.

# 14. 오늘 생성하는 Ground Truth를 구분합니다

```text
RGB
→ 가상 Camera 이미지

COCO BBox
→ [x, y, width, height]

Semantic Mask
→ Pixel마다 Class ID

Instance Mask
→ Object Instance별 ID

Depth
→ Camera와 Pixel 사이 거리
```

# 15. 왜 3D에서는 Ground Truth를 자동 생성할 수 있나요?

3D Scene에는 이미:

```text
Object Mesh
Object 위치
Object 회전
Camera 위치
Camera 방향
Object Class
```

가 있습니다.

그래서 Camera에 투영할 때 정답을 함께 계산할 수 있습니다.

---

# PART 3. Main 데이터로 첫 번째 3D Scene을 Rendering합니다

# 16. 첫 Rendering에서는 두 Object를 모두 사용합니다

```text
               Light
                 ↓

cracker_box   mustard_bottle
      \          /
         Plane
           ↑
         Camera
```

첫 실행은 조건을 고정합니다.

# 17. Single Scene Rendering 코드를 작성합니다

BlenderProc Rendering Script에서는 **`import blenderproc as bproc`를 실제 첫 import로 두는 실행 패턴**을 유지합니다.

```text
import blenderproc as bproc
        ↓
나머지 라이브러리 import
        ↓
bproc.init()
        ↓
Scene 구성
```

따라서 일반 Python Script에서 자주 사용하는 `from __future__ import annotations`도 BlenderProc Rendering Script 앞에는 두지 않습니다.

**파일: `scripts/02_render_single_scene.py`**

```python
import blenderproc as bproc

import argparse
import json
import sys
from pathlib import Path

import numpy as np


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day04_common import (
    bop_dataset_path,
    data_config,
    lesson_objects,
    result_root,
)


def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--asset-set",
        default="main",
        choices=["main", "challenge"],
    )
    parser.add_argument(
        "--tag",
        default="single",
    )
    parser.add_argument(
        "--seed",
        type=int,
        default=42,
    )
    return parser.parse_args()


def main() -> None:
    args = parse_args()

    np.random.seed(args.seed)
    bproc.init()

    config = data_config(
        args.asset_set
    )

    object_info = {
        int(item["bop_obj_id"]): item
        for item
        in lesson_objects(args.asset_set)
    }

    bop_path = bop_dataset_path(
        args.asset_set
    )

    bop_ids = [
        int(value)
        for value
        in config["bop_obj_ids"]
    ]

    objects = bproc.loader.load_bop_objs(
        bop_dataset_path=str(bop_path),
        obj_ids=bop_ids,
        object_model_unit="mm",
        move_origin_to_x_y_plane=True,
    )

    if len(objects) != len(bop_ids):
        raise RuntimeError(
            "요청한 BOP Object 수와 "
            "불러온 Object 수가 다릅니다."
        )

    for obj in objects:
        bop_id = int(
            obj.get_cp("category_id")
        )

        item = object_info[bop_id]

        obj.set_cp(
            "bop_obj_id",
            bop_id,
        )

        obj.set_cp(
            "category_id",
            int(item["lesson_class_id"]),
        )

        obj.set_cp(
            "supercategory",
            "coco_annotations",
        )

        obj.set_name(
            item["class_name"]
        )

        obj.set_shading_mode("auto")

    if len(objects) == 2:
        x_positions = [-0.13, 0.13]
    else:
        x_positions = [0.0]

    object_rows = []

    for index, obj in enumerate(objects):
        rotation_z = (
            -0.20
            if index == 0
            else 0.20
        )

        if len(objects) == 1:
            rotation_z = 0.15

        obj.set_location(
            [
                x_positions[index],
                0.0,
                0.0,
            ]
        )

        obj.set_rotation_euler(
            [0.0, 0.0, rotation_z]
        )

        object_rows.append(
            {
                "name": obj.get_name(),
                "lesson_class_id": int(
                    obj.get_cp("category_id")
                ),
                "bop_obj_id": int(
                    obj.get_cp("bop_obj_id")
                ),
                "location": (
                    obj.get_location().tolist()
                ),
                "rotation": (
                    obj.get_rotation_euler()
                    .tolist()
                ),
            }
        )

    floor = bproc.object.create_primitive(
        "PLANE",
        scale=[0.55, 0.55, 1.0],
    )

    floor.set_name("floor")
    floor.set_location(
        [0.0, 0.0, -0.001]
    )
    floor.set_cp("category_id", 0)
    floor.set_cp(
        "supercategory",
        "background",
    )

    light = bproc.types.Light()
    light.set_type("POINT")

    light_location = [
        0.25,
        -0.35,
        0.85,
    ]

    light_energy = 500.0

    light.set_location(
        light_location
    )
    light.set_energy(
        light_energy
    )

    camera_location = np.array(
        [0.0, -0.75, 0.35]
    )

    poi = bproc.object.compute_poi(
        objects
    )

    rotation_matrix = (
        bproc.camera
        .rotation_from_forward_vec(
            poi - camera_location
        )
    )

    camera_pose = (
        bproc.math
        .build_transformation_mat(
            camera_location,
            rotation_matrix,
        )
    )

    bproc.camera.add_camera_pose(
        camera_pose
    )

    bproc.camera.set_resolution(
        512,
        512,
    )

    bproc.renderer.enable_depth_output(
        activate_antialiasing=False
    )

    render_data = (
        bproc.renderer.render()
    )

    seg_data = (
        bproc.renderer.render_segmap(
            map_by=[
                "instance",
                "class",
                "name",
            ],
            default_values={
                "category_id": 0,
            },
        )
    )

    render_data.update(seg_data)

    output_dir = result_root(
        args.asset_set,
        args.tag,
    )

    bproc.writer.write_hdf5(
        str(output_dir),
        render_data,
    )

    coco_dir = (
        output_dir
        / "coco_data"
    )

    bproc.writer.write_coco_annotations(
        str(coco_dir),
        instance_segmaps=(
            seg_data["instance_segmaps"]
        ),
        instance_attribute_maps=(
            seg_data[
                "instance_attribute_maps"
            ]
        ),
        colors=render_data["colors"],
        color_file_format="JPEG",
        supercategory="coco_annotations",
        append_to_existing_output=False,
    )

    metadata = {
        "asset_set": args.asset_set,
        "tag": args.tag,
        "seed": args.seed,
        "resolution": [512, 512],
        "objects": object_rows,
        "camera_location": (
            camera_location.tolist()
        ),
        "light_location": light_location,
        "light_energy": light_energy,
        "approved": "pending",
    }

    with (
        output_dir
        / "sample_metadata.json"
    ).open(
        "w",
        encoding="utf-8",
    ) as file:
        json.dump(
            metadata,
            file,
            ensure_ascii=False,
            indent=2,
        )

    print("=" * 64)
    print("Single 3D Scene Completed")
    print("=" * 64)
    print("Asset Set:", args.asset_set)
    print(
        "Objects  :",
        [
            row["name"]
            for row in object_rows
        ],
    )
    print("Output   :", output_dir)


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

여러 장을 만들기 전에 **YCB-V Model Loading부터 RGB·Depth·Segmentation·COCO BBox까지 한 장으로 검증**하기 위해 작성합니다.

### 의사코드로 읽어보기

```text
Set을 선택한다
        ↓
BOP ID를 읽는다
        ↓
3D Model을 불러온다
        ↓
BOP ID를 Lesson Class ID로 연결한다
        ↓
Object를 바닥 위에 배치한다
        ↓
Plane / Light / Camera를 만든다
        ↓
RGB + Depth를 Rendering한다
        ↓
Semantic + Instance를 Rendering한다
        ↓
HDF5 / COCO / Metadata를 저장한다
```

# 18. Main Single Scene을 실행합니다

```bash
blenderproc run \
  scripts/02_render_single_scene.py \
  --asset-set main \
  --tag single \
  --seed 42
```

예상 구조:

```text
results/main/single/
├─ 0.hdf5
├─ sample_metadata.json
└─ coco_data/
   ├─ coco_annotations.json
   └─ images/
      └─ 000000.jpg
```

# 19. RGB와 COCO BBox를 확인합니다

```bash
python -m json.tool \
  results/main/single/coco_data/coco_annotations.json \
  | head -100
```

Main의 객체 Class ID:

```text
1 → cracker_box
2 → mustard_bottle
```

COCO BBox 형식:

```text
[x, y, width, height]
```

# 20. HDF5를 BlenderProc Viewer로 확인합니다

```bash
blenderproc vis hdf5 \
  results/main/single/0.hdf5
```

# 21. HDF5 결과를 PNG Preview로 저장하는 코드를 작성합니다

**파일: `scripts/03_export_hdf5_preview.py`**

```python
from __future__ import annotations

import argparse
from pathlib import Path

import h5py
import matplotlib.pyplot as plt
import numpy as np
from PIL import Image


def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--input",
        required=True,
    )
    parser.add_argument(
        "--output",
        required=True,
    )
    return parser.parse_args()


def find_key(keys, words):
    for key in keys:
        lower = key.lower()

        if all(
            word in lower
            for word in words
        ):
            return key

    return None


def main() -> None:
    args = parse_args()

    input_path = Path(args.input)
    output_dir = Path(args.output)

    output_dir.mkdir(
        parents=True,
        exist_ok=True,
    )

    if not input_path.exists():
        raise FileNotFoundError(input_path)

    with h5py.File(
        input_path,
        "r",
    ) as file:
        keys = list(file.keys())

        print("HDF5 Keys")
        for key in keys:
            print(" -", key)

        if "colors" in file:
            rgb = np.array(
                file["colors"]
            )

            if (
                rgb.ndim == 3
                and rgb.shape[-1] >= 3
            ):
                rgb = rgb[..., :3]

            if rgb.dtype != np.uint8:
                if rgb.max() <= 1.0:
                    rgb = rgb * 255.0

                rgb = np.clip(
                    rgb,
                    0,
                    255,
                ).astype(np.uint8)

            Image.fromarray(rgb).save(
                output_dir
                / "rgb.png"
            )

        depth_key = None

        for candidate in [
            "depth",
            "distance",
        ]:
            if candidate in file:
                depth_key = candidate
                break

        if depth_key:
            depth = np.array(
                file[depth_key]
            ).astype(np.float32)

            valid = (
                np.isfinite(depth)
                & (depth > 0)
            )

            normalized = np.zeros_like(
                depth,
                dtype=np.float32,
            )

            if valid.any():
                values = depth[valid]
                low = np.percentile(
                    values,
                    2,
                )
                high = np.percentile(
                    values,
                    98,
                )

                if high > low:
                    normalized[valid] = (
                        np.clip(
                            (
                                depth[valid]
                                - low
                            )
                            / (
                                high - low
                            ),
                            0,
                            1,
                        )
                    )

            plt.imsave(
                output_dir
                / "depth.png",
                normalized,
                cmap="gray",
            )

        class_key = find_key(
            keys,
            ["class", "seg"],
        )

        if class_key:
            plt.imsave(
                output_dir
                / "semantic_mask.png",
                np.array(
                    file[class_key]
                ),
            )

        instance_key = find_key(
            keys,
            ["instance", "seg"],
        )

        if instance_key:
            plt.imsave(
                output_dir
                / "instance_mask.png",
                np.array(
                    file[instance_key]
                ),
            )

    print(
        "Preview Saved:",
        output_dir,
    )


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

HDF5의 RGB·Depth·Semantic·Instance를 PNG로 남겨 비교하기 위해 작성합니다.

### 의사코드로 읽어보기

```text
HDF5를 연다
        ↓
Key 목록을 출력한다
        ↓
RGB를 저장한다
        ↓
Depth를 보기 좋게 정규화한다
        ↓
Class Segmentation을 저장한다
        ↓
Instance Segmentation을 저장한다
```

실행:

```bash
python scripts/03_export_hdf5_preview.py \
  --input results/main/single/0.hdf5 \
  --output results/main/single/previews
```

# 22. Single 결과를 직접 비교합니다

```text
RGB
Semantic Mask
Instance Mask
Depth
COCO BBox
```

다음을 확인합니다.

```text
Object 형태가 정상인가?

BBox가 Object를 감싸는가?

Semantic이 Class를 구분하는가?

Instance가 Object를 구분하는가?

Depth에 거리 차이가 보이는가?
```


# PART 4. Object·Camera·Light를 바꾸어 여러 Synthetic Scene을 생성합니다

# 23. Randomization이 필요한 이유를 이해합니다

Single Scene 하나만 계속 복사하면 Dataset 다양성이 없습니다.

```text
같은 Object 위치

같은 Camera

같은 Light

같은 방향
        ↓
거의 같은 이미지 반복
```

오늘은 다음 조건을 바꿉니다.

```text
Object X / Y 위치

Object Z축 회전

Camera X 위치

Camera Distance

Camera Height

Light X / Y / Z

Light Energy
```

하지만 Random 범위가 넓을수록 무조건 좋은 것은 아닙니다.

```text
다양성 증가
        ↓
현실성 유지?
        ↓
QA 필요
```

---

# 24. 한 개의 Randomized Sample을 생성하는 코드를 작성합니다

**파일: `scripts/04_render_random_sample.py`**

```python
import blenderproc as bproc

import argparse
import json
import random
import sys
from pathlib import Path

import numpy as np


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day04_common import (
    bop_dataset_path,
    data_config,
    lesson_objects,
)


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
        "--seed",
        type=int,
        required=True,
    )

    parser.add_argument(
        "--output",
        required=True,
    )

    parser.add_argument(
        "--camera-distance-min",
        type=float,
        default=0.65,
    )

    parser.add_argument(
        "--camera-distance-max",
        type=float,
        default=0.85,
    )

    parser.add_argument(
        "--light-min",
        type=float,
        default=350.0,
    )

    parser.add_argument(
        "--light-max",
        type=float,
        default=800.0,
    )

    parser.add_argument(
        "--rotation-max-deg",
        type=float,
        default=25.0,
    )

    return parser.parse_args()


def main() -> None:
    args = parse_args()

    if not (
        0.2
        <= args.camera_distance_min
        <= args.camera_distance_max
    ):
        raise ValueError(
            "Camera Distance 범위를 확인하세요."
        )

    if not (
        1.0
        <= args.light_min
        <= args.light_max
    ):
        raise ValueError(
            "Light 범위를 확인하세요."
        )

    if not (
        0
        <= args.rotation_max_deg
        <= 180
    ):
        raise ValueError(
            "Rotation 범위를 확인하세요."
        )

    rng = random.Random(args.seed)
    np.random.seed(args.seed)

    output_dir = Path(args.output)
    output_dir.mkdir(
        parents=True,
        exist_ok=True,
    )

    bproc.init()

    config = data_config(
        args.asset_set
    )

    object_info = {
        int(item["bop_obj_id"]): item
        for item
        in lesson_objects(args.asset_set)
    }

    bop_path = bop_dataset_path(
        args.asset_set
    )

    bop_ids = [
        int(value)
        for value
        in config["bop_obj_ids"]
    ]

    objects = bproc.loader.load_bop_objs(
        bop_dataset_path=str(bop_path),
        obj_ids=bop_ids,
        object_model_unit="mm",
        move_origin_to_x_y_plane=True,
    )

    if len(objects) != len(bop_ids):
        raise RuntimeError(
            "BOP Object Loading 수가 다릅니다."
        )

    for obj in objects:
        bop_id = int(
            obj.get_cp("category_id")
        )

        item = object_info[bop_id]

        obj.set_cp(
            "bop_obj_id",
            bop_id,
        )

        obj.set_cp(
            "category_id",
            int(item["lesson_class_id"]),
        )

        obj.set_cp(
            "supercategory",
            "coco_annotations",
        )

        obj.set_name(
            item["class_name"]
        )

        obj.set_shading_mode("auto")

    if len(objects) == 2:
        base_x = [-0.13, 0.13]
    else:
        base_x = [0.0]

    object_rows = []

    rotation_max = np.deg2rad(
        args.rotation_max_deg
    )

    for index, obj in enumerate(objects):
        x = (
            base_x[index]
            + rng.uniform(
                -0.035,
                0.035,
            )
        )

        y = rng.uniform(
            -0.04,
            0.04,
        )

        rotation_z = rng.uniform(
            -rotation_max,
            rotation_max,
        )

        obj.set_location(
            [x, y, 0.0]
        )

        obj.set_rotation_euler(
            [0.0, 0.0, rotation_z]
        )

        object_rows.append(
            {
                "name": obj.get_name(),
                "lesson_class_id": int(
                    obj.get_cp("category_id")
                ),
                "bop_obj_id": int(
                    obj.get_cp("bop_obj_id")
                ),
                "location": [
                    x,
                    y,
                    0.0,
                ],
                "rotation_z": rotation_z,
            }
        )

    floor = bproc.object.create_primitive(
        "PLANE",
        scale=[0.60, 0.60, 1.0],
    )

    floor.set_name("floor")
    floor.set_location(
        [0.0, 0.0, -0.001]
    )
    floor.set_cp("category_id", 0)
    floor.set_cp(
        "supercategory",
        "background",
    )

    light_location = [
        rng.uniform(-0.45, 0.45),
        rng.uniform(-0.45, 0.15),
        rng.uniform(0.65, 1.15),
    ]

    light_energy = rng.uniform(
        args.light_min,
        args.light_max,
    )

    light = bproc.types.Light()
    light.set_type("POINT")
    light.set_location(
        light_location
    )
    light.set_energy(
        light_energy
    )

    camera_distance = rng.uniform(
        args.camera_distance_min,
        args.camera_distance_max,
    )

    camera_location = np.array(
        [
            rng.uniform(-0.13, 0.13),
            -camera_distance,
            rng.uniform(0.26, 0.48),
        ]
    )

    poi = bproc.object.compute_poi(
        objects
    )

    rotation_matrix = (
        bproc.camera
        .rotation_from_forward_vec(
            poi - camera_location
        )
    )

    camera_pose = (
        bproc.math
        .build_transformation_mat(
            camera_location,
            rotation_matrix,
        )
    )

    bproc.camera.add_camera_pose(
        camera_pose
    )

    bproc.camera.set_resolution(
        512,
        512,
    )

    bproc.renderer.enable_depth_output(
        activate_antialiasing=False
    )

    render_data = (
        bproc.renderer.render()
    )

    seg_data = (
        bproc.renderer.render_segmap(
            map_by=[
                "instance",
                "class",
                "name",
            ],
            default_values={
                "category_id": 0,
            },
        )
    )

    render_data.update(seg_data)

    bproc.writer.write_hdf5(
        str(output_dir),
        render_data,
    )

    coco_dir = (
        output_dir
        / "coco_data"
    )

    bproc.writer.write_coco_annotations(
        str(coco_dir),
        instance_segmaps=(
            seg_data["instance_segmaps"]
        ),
        instance_attribute_maps=(
            seg_data[
                "instance_attribute_maps"
            ]
        ),
        colors=render_data["colors"],
        color_file_format="JPEG",
        supercategory="coco_annotations",
        append_to_existing_output=False,
    )

    metadata = {
        "asset_set": args.asset_set,
        "seed": args.seed,
        "objects": object_rows,
        "camera_distance": camera_distance,
        "camera_location": (
            camera_location.tolist()
        ),
        "light_location": light_location,
        "light_energy": light_energy,
        "rotation_max_deg": (
            args.rotation_max_deg
        ),
        "camera_distance_min": (
            args.camera_distance_min
        ),
        "camera_distance_max": (
            args.camera_distance_max
        ),
        "light_min": args.light_min,
        "light_max": args.light_max,
        "approved": "pending",
    }

    with (
        output_dir
        / "sample_metadata.json"
    ).open(
        "w",
        encoding="utf-8",
    ) as file:
        json.dump(
            metadata,
            file,
            ensure_ascii=False,
            indent=2,
        )

    print(
        "Completed Seed:",
        args.seed,
    )

    print(
        "Output        :",
        output_dir,
    )


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

한 장면의 Camera·Light·Object 조건을 Seed에 따라 바꾸고, **같은 코드·같은 Seed·같은 Parameter 범위에서 Random 조건을 다시 구성하고 Metadata로 추적할 수 있는 Randomized Sample**을 만들기 위해 작성합니다.

Seed는 Random 조건을 추적하기 위한 기준입니다. 다른 Blender / BlenderProc / GPU / Driver / Library 버전까지 달라진 환경에서 픽셀 단위로 완전히 동일한 Render를 보장한다는 의미는 아닙니다.

### 의사코드로 읽어보기

```text
Set과 Seed를 받는다
        ↓
Randomization 범위를 받는다
        ↓
3D Object를 불러온다
        ↓
Object 위치와 회전을 Random하게 정한다
        ↓
Light 위치와 밝기를 Random하게 정한다
        ↓
Camera 거리와 위치를 Random하게 정한다
        ↓
RGB / Depth / Segmentation을 Rendering한다
        ↓
HDF5와 COCO를 저장한다
        ↓
실제 사용한 값을 Metadata에 기록한다
```

# 25. 한 장으로 Smoke Test를 수행합니다

```bash
blenderproc run \
  scripts/04_render_random_sample.py \
  --asset-set main \
  --seed 100 \
  --output results/main/smoke/sample_0100 \
  --camera-distance-min 0.65 \
  --camera-distance-max 0.85 \
  --light-min 350 \
  --light-max 800 \
  --rotation-max-deg 25
```

확인:

```bash
find results/main/smoke/sample_0100 \
  -maxdepth 3 \
  -type f
```

```bash
blenderproc vis hdf5 \
  results/main/smoke/sample_0100/0.hdf5
```

한 장이 정상일 때만 Batch 생성으로 넘어갑니다.

---

# 26. 여러 Scene을 자동 생성하는 Launcher를 작성합니다

**파일: `scripts/05_generate_dataset.py`**

```python
from __future__ import annotations

import argparse
import csv
import shutil
import subprocess
from pathlib import Path


ROOT = Path(__file__).resolve().parents[1]


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
        "--count",
        type=int,
        default=8,
    )

    parser.add_argument(
        "--seed-start",
        type=int,
        default=200,
    )

    parser.add_argument(
        "--camera-distance-min",
        type=float,
        default=0.65,
    )

    parser.add_argument(
        "--camera-distance-max",
        type=float,
        default=0.85,
    )

    parser.add_argument(
        "--light-min",
        type=float,
        default=350.0,
    )

    parser.add_argument(
        "--light-max",
        type=float,
        default=800.0,
    )

    parser.add_argument(
        "--rotation-max-deg",
        type=float,
        default=25.0,
    )

    parser.add_argument(
        "--overwrite",
        action="store_true",
    )

    return parser.parse_args()


def main() -> None:
    args = parse_args()

    if args.count < 1:
        raise ValueError(
            "--count는 1 이상이어야 합니다."
        )

    blenderproc = shutil.which(
        "blenderproc"
    )

    if blenderproc is None:
        raise RuntimeError(
            "blenderproc 명령을 찾을 수 없습니다."
        )

    output_root = (
        ROOT
        / "results"
        / args.asset_set
        / args.tag
    )

    if output_root.exists():
        if not args.overwrite:
            raise RuntimeError(
                f"이미 결과 폴더가 있습니다:\n"
                f"{output_root}\n"
                "다시 만들려면 --overwrite를 추가하세요."
            )

        shutil.rmtree(output_root)

    output_root.mkdir(
        parents=True,
        exist_ok=True,
    )

    rows = []

    render_script = (
        ROOT
        / "scripts"
        / "04_render_random_sample.py"
    )

    for index in range(args.count):
        seed = (
            args.seed_start
            + index
        )

        sample_dir = (
            output_root
            / f"sample_{seed:04d}"
        )

        command = [
            blenderproc,
            "run",
            str(render_script),
            "--asset-set",
            args.asset_set,
            "--seed",
            str(seed),
            "--output",
            str(sample_dir),
            "--camera-distance-min",
            str(
                args.camera_distance_min
            ),
            "--camera-distance-max",
            str(
                args.camera_distance_max
            ),
            "--light-min",
            str(args.light_min),
            "--light-max",
            str(args.light_max),
            "--rotation-max-deg",
            str(
                args.rotation_max_deg
            ),
        ]

        print("=" * 64)
        print(
            f"Generating "
            f"{index + 1}/{args.count}"
        )
        print("Seed:", seed)
        print("=" * 64)

        subprocess.run(
            command,
            check=True,
            cwd=ROOT,
        )

        rows.append(
            {
                "sample": sample_dir.name,
                "seed": seed,
                "camera_distance_min": (
                    args.camera_distance_min
                ),
                "camera_distance_max": (
                    args.camera_distance_max
                ),
                "light_min": args.light_min,
                "light_max": args.light_max,
                "rotation_max_deg": (
                    args.rotation_max_deg
                ),
            }
        )

    plan_path = (
        output_root
        / "generation_plan.csv"
    )

    with plan_path.open(
        "w",
        newline="",
        encoding="utf-8-sig",
    ) as file:
        writer = csv.DictWriter(
            file,
            fieldnames=list(
                rows[0].keys()
            ),
        )

        writer.writeheader()
        writer.writerows(rows)

    print(
        "Generation Completed:",
        output_root,
    )


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

학생이 `blenderproc run`을 여러 번 직접 입력하지 않고 **Seed만 증가시키며 여러 Scene을 자동 생성**하기 위해 작성합니다.

같은 `tag`를 실수로 덮어쓰지 않도록 기존 결과가 있으면 중단합니다.

### 의사코드로 읽어보기

```text
Set / Tag / Count를 받는다
        ↓
blenderproc 실행파일을 찾는다
        ↓
결과 폴더가 이미 있으면 중단한다
        ↓
Seed를 하나씩 증가시킨다
        ↓
각 Seed마다 Rendering Script를 실행한다
        ↓
sample_XXXX로 저장한다
        ↓
전체 조건을 generation_plan.csv에 기록한다
```

# 27. Main Dataset을 8장 생성합니다

```bash
python scripts/05_generate_dataset.py \
  --asset-set main \
  --tag baseline \
  --count 8 \
  --seed-start 200 \
  --camera-distance-min 0.65 \
  --camera-distance-max 0.85 \
  --light-min 350 \
  --light-max 800 \
  --rotation-max-deg 25
```

결과:

```text
results/main/baseline/
├─ generation_plan.csv
├─ sample_0200/
├─ sample_0201/
├─ ...
└─ sample_0207/
```

각 Sample에는:

```text
0.hdf5
sample_metadata.json
coco_data/
```

가 있습니다.

오늘은 수백 장 생성보다 **자동 생성 구조를 정확히 이해하는 것**이 먼저입니다.

---

# 28. Sample별 Metadata를 하나의 CSV로 모읍니다

**파일: `scripts/06_collect_metadata.py`**

```python
from __future__ import annotations

import argparse
import json
from pathlib import Path

import pandas as pd


ROOT = Path(__file__).resolve().parents[1]


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

    return parser.parse_args()


def main() -> None:
    args = parse_args()

    source_root = (
        ROOT
        / "results"
        / args.asset_set
        / args.tag
    )

    rows = []

    for sample_dir in sorted(
        source_root.glob("sample_*")
    ):
        metadata_path = (
            sample_dir
            / "sample_metadata.json"
        )

        if not metadata_path.exists():
            continue

        with metadata_path.open(
            "r",
            encoding="utf-8",
        ) as file:
            data = json.load(file)

        row = {
            "sample": sample_dir.name,
            "asset_set": (
                data["asset_set"]
            ),
            "seed": data["seed"],
            "camera_distance": (
                data["camera_distance"]
            ),
            "camera_x": (
                data["camera_location"][0]
            ),
            "camera_y": (
                data["camera_location"][1]
            ),
            "camera_z": (
                data["camera_location"][2]
            ),
            "light_x": (
                data["light_location"][0]
            ),
            "light_y": (
                data["light_location"][1]
            ),
            "light_z": (
                data["light_location"][2]
            ),
            "light_energy": (
                data["light_energy"]
            ),
            "rotation_max_deg": (
                data["rotation_max_deg"]
            ),
            "approved": (
                data["approved"]
            ),
        }

        for object_index, item in enumerate(
            data["objects"],
            start=1,
        ):
            prefix = f"object{object_index}"

            row[
                f"{prefix}_name"
            ] = item["name"]

            row[
                f"{prefix}_class_id"
            ] = (
                item["lesson_class_id"]
            )

            row[
                f"{prefix}_bop_id"
            ] = item["bop_obj_id"]

            row[
                f"{prefix}_x"
            ] = item["location"][0]

            row[
                f"{prefix}_y"
            ] = item["location"][1]

            row[
                f"{prefix}_rotation_z"
            ] = item["rotation_z"]

        rows.append(row)

    if not rows:
        raise RuntimeError(
            "수집할 Metadata가 없습니다."
        )

    dataframe = pd.DataFrame(rows)

    report_dir = (
        ROOT
        / "reports"
        / args.asset_set
    )

    report_dir.mkdir(
        parents=True,
        exist_ok=True,
    )

    output_path = (
        report_dir
        / f"{args.tag}_metadata.csv"
    )

    dataframe.to_csv(
        output_path,
        index=False,
        encoding="utf-8-sig",
    )

    print(
        "Samples:",
        len(dataframe),
    )

    print("Saved:", output_path)


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

Sample별 JSON을 한 표로 모아 **Camera·Light·Object 조건을 한 번에 비교하고 실패 Sample을 다시 추적**하기 위해 작성합니다.

### 의사코드로 읽어보기

```text
sample_* 폴더를 찾는다
        ↓
Metadata JSON을 읽는다
        ↓
Seed / Camera / Light를 기록한다
        ↓
Object 이름·Class·위치·회전을 기록한다
        ↓
모든 Sample을 DataFrame으로 만든다
        ↓
CSV로 저장한다
```

실행:

```bash
python scripts/06_collect_metadata.py \
  --asset-set main \
  --tag baseline
```

결과:

```text
reports/main/baseline_metadata.csv
```

---

# PART 5. 3D Synthetic Dataset을 QA하고 Real Reference와 비교합니다

# 29. Automatic QA에서 무엇을 확인할지 정합니다

```text
HDF5가 있는가?

Metadata가 있는가?

COCO JSON이 있는가?

RGB가 있는가?

Depth가 있는가?

Semantic이 있는가?

Instance가 있는가?

BBox가 있는가?

BBox Width / Height가 정상인가?

BBox가 이미지 안에 있는가?
```

# 30. Automatic QA 코드를 작성합니다

**파일: `scripts/07_qa_3d_dataset.py`**

```python
from __future__ import annotations

import argparse
import json
from pathlib import Path

import h5py
import pandas as pd


ROOT = Path(__file__).resolve().parents[1]


EXPECTED_OBJECTS = {
    "main": 2,
    "challenge": 1,
}

EXPECTED_CLASS_IDS = {
    "main": {1, 2},
    "challenge": {3},
}


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
        "--overwrite",
        action="store_true",
    )

    return parser.parse_args()


def has_key(keys, words) -> bool:
    for key in keys:
        lower = key.lower()

        if all(
            word in lower
            for word in words
        ):
            return True

    return False


def main() -> None:
    args = parse_args()

    report_dir = (
        ROOT
        / "reports"
        / args.asset_set
    )

    report_dir.mkdir(
        parents=True,
        exist_ok=True,
    )

    output_path = (
        report_dir
        / f"{args.tag}_qa.csv"
    )

    if (
        output_path.exists()
        and not args.overwrite
    ):
        raise RuntimeError(
            "이미 QA CSV가 있습니다.\n"
            f"{output_path}\n"
            "Human QA 기록을 보호하기 위해 "
            "자동으로 덮어쓰지 않습니다.\n"
            "처음부터 다시 만들려면 "
            "--overwrite를 추가하세요."
        )

    source_root = (
        ROOT
        / "results"
        / args.asset_set
        / args.tag
    )

    expected_objects = (
        EXPECTED_OBJECTS[
            args.asset_set
        ]
    )

    expected_class_ids = (
        EXPECTED_CLASS_IDS[
            args.asset_set
        ]
    )

    rows = []

    for sample_dir in sorted(
        source_root.glob("sample_*")
    ):
        reasons = []

        hdf5_path = (
            sample_dir / "0.hdf5"
        )

        metadata_path = (
            sample_dir
            / "sample_metadata.json"
        )

        coco_path = (
            sample_dir
            / "coco_data"
            / "coco_annotations.json"
        )

        if not hdf5_path.exists():
            reasons.append(
                "missing_hdf5"
            )

        if not metadata_path.exists():
            reasons.append(
                "missing_metadata"
            )

        if not coco_path.exists():
            reasons.append(
                "missing_coco"
            )

        hdf5_keys = []

        if hdf5_path.exists():
            try:
                with h5py.File(
                    hdf5_path,
                    "r",
                ) as file:
                    hdf5_keys = list(
                        file.keys()
                    )
            except Exception:
                reasons.append(
                    "unreadable_hdf5"
                )

        if hdf5_keys:
            if "colors" not in hdf5_keys:
                reasons.append(
                    "missing_rgb"
                )

            if not (
                "depth" in hdf5_keys
                or "distance" in hdf5_keys
            ):
                reasons.append(
                    "missing_depth"
                )

            if not has_key(
                hdf5_keys,
                ["class", "seg"],
            ):
                reasons.append(
                    "missing_semantic"
                )

            if not has_key(
                hdf5_keys,
                ["instance", "seg"],
            ):
                reasons.append(
                    "missing_instance"
                )

        annotation_count = 0
        invalid_bbox_count = 0
        category_ids = set()

        if coco_path.exists():
            try:
                with coco_path.open(
                    "r",
                    encoding="utf-8",
                ) as file:
                    coco = json.load(file)

                images = {
                    int(item["id"]): item
                    for item
                    in coco.get(
                        "images",
                        [],
                    )
                }

                annotations = coco.get(
                    "annotations",
                    [],
                )

                annotation_count = len(
                    annotations
                )

                if annotation_count < (
                    expected_objects
                ):
                    reasons.append(
                        "missing_annotation"
                    )

                for annotation in annotations:
                    try:
                        category_ids.add(
                            int(
                                annotation[
                                    "category_id"
                                ]
                            )
                        )
                    except (
                        KeyError,
                        TypeError,
                        ValueError,
                    ):
                        reasons.append(
                            "invalid_category_id"
                        )

                    bbox = annotation.get(
                        "bbox"
                    )

                    if (
                        not bbox
                        or len(bbox) != 4
                    ):
                        invalid_bbox_count += 1
                        continue

                    x, y, width, height = bbox

                    image_info = images.get(
                        int(
                            annotation[
                                "image_id"
                            ]
                        )
                    )

                    if image_info is None:
                        invalid_bbox_count += 1
                        continue

                    image_width = float(
                        image_info["width"]
                    )

                    image_height = float(
                        image_info["height"]
                    )

                    if (
                        width <= 1
                        or height <= 1
                        or x < 0
                        or y < 0
                        or x + width
                        > image_width
                        or y + height
                        > image_height
                    ):
                        invalid_bbox_count += 1

                if (
                    category_ids
                    != expected_class_ids
                ):
                    reasons.append(
                        "class_id_mismatch"
                    )

                if invalid_bbox_count:
                    reasons.append(
                        "invalid_bbox"
                    )

            except Exception:
                reasons.append(
                    "unreadable_coco"
                )

        rows.append(
            {
                "sample": sample_dir.name,
                "annotation_count": (
                    annotation_count
                ),
                "category_ids": (
                    ",".join(
                        str(value)
                        for value
                        in sorted(category_ids)
                    )
                ),
                "invalid_bbox_count": (
                    invalid_bbox_count
                ),
                "auto_pass": (
                    len(reasons) == 0
                ),
                "reason": (
                    "|".join(reasons)
                ),
                "human_decision": "",
                "human_note": "",
            }
        )

    if not rows:
        raise RuntimeError(
            "검사할 Sample이 없습니다."
        )

    dataframe = pd.DataFrame(rows)

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

    failed = (
        len(dataframe) - passed
    )

    print("=" * 64)
    print(
        "3D Synthetic Automatic QA"
    )
    print("=" * 64)
    print("Total:", len(dataframe))
    print("PASS :", passed)
    print("FAIL :", failed)
    print("Saved:", output_path)


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

여러 Sample을 사람이 하나씩 열기 전에 **파일 누락·Ground Truth 누락·Lesson Class ID 오류·BBox 범위 오류를 자동으로 먼저 찾기 위해** 작성합니다.

또한 Human QA를 작성한 CSV를 실수로 덮어쓰지 않도록 기존 QA CSV가 있으면 자동으로 중단합니다.

### 의사코드로 읽어보기

```text
각 Sample을 찾는다
        ↓
HDF5 / Metadata / COCO를 확인한다
        ↓
HDF5 Key를 읽는다
        ↓
RGB / Depth / Semantic / Instance를 확인한다
        ↓
COCO Annotation 수를 확인한다
        ↓
Lesson Class ID가 예상값인지 확인한다
        ↓
BBox 범위를 확인한다
        ↓
문제가 없으면 auto_pass
        ↓
QA CSV로 저장한다
```

실행:

```bash
python scripts/07_qa_3d_dataset.py \
  --asset-set main \
  --tag baseline
```

QA CSV를 처음부터 다시 만들어야 할 때만 `--overwrite`를 사용합니다.

```bash
python scripts/07_qa_3d_dataset.py \
  --asset-set main \
  --tag baseline \
  --overwrite
```

정상적인 구조라면:

```text
Total: 8
PASS : 8
FAIL : 0
```

처럼 나올 수 있습니다.

`PASS`는 **구조와 Label이 정상이라는 뜻**이지 현실성이 완벽하다는 뜻은 아닙니다.

---

# 31. Real과 Synthetic을 한 화면에서 비교하는 코드를 작성합니다

**파일: `scripts/08_compare_real_synthetic.py`**

```python
from __future__ import annotations

import argparse
import sys
from pathlib import Path

from PIL import (
    Image,
    ImageDraw,
    ImageOps,
)


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day04_common import (
    lesson_objects,
    real_reference_root,
)


TILE_W = 240
TILE_H = 190
LABEL_H = 36


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

    return parser.parse_args()


def image_files(
    directory: Path,
) -> list[Path]:
    if not directory.exists():
        return []

    return sorted(
        path
        for path in directory.rglob("*")
        if path.is_file()
        and path.suffix.lower()
        in {".png", ".jpg", ".jpeg"}
    )


def make_row(
    files,
    label_prefix,
    columns,
):
    selected = files[:columns]

    canvas = Image.new(
        "RGB",
        (
            TILE_W * columns,
            TILE_H + LABEL_H,
        ),
        "white",
    )

    draw = ImageDraw.Draw(canvas)

    for column, path in enumerate(
        selected
    ):
        x = column * TILE_W

        with Image.open(path) as image:
            image = (
                ImageOps
                .exif_transpose(image)
                .convert("RGB")
            )

            image.thumbnail(
                (
                    TILE_W - 12,
                    TILE_H - 12,
                ),
                Image.Resampling.LANCZOS,
            )

            px = (
                x
                + (TILE_W - image.width)
                // 2
            )

            py = (
                (TILE_H - image.height)
                // 2
            )

            canvas.paste(
                image,
                (px, py),
            )

        draw.text(
            (
                x + 5,
                TILE_H + 7,
            ),
            f"{label_prefix} {column + 1}",
            fill="black",
        )

    return canvas


def main() -> None:
    args = parse_args()

    real_root = real_reference_root(
        args.asset_set
    )

    objects = lesson_objects(
        args.asset_set
    )

    real_files = []

    per_class = max(
        1,
        6 // len(objects),
    )

    for item in objects:
        class_name = item[
            "class_name"
        ]

        class_files = image_files(
            real_root
            / class_name
        )

        real_files.extend(
            class_files[:per_class]
        )

    synthetic_root = (
        ROOT
        / "results"
        / args.asset_set
        / args.tag
    )

    synthetic_files = sorted(
        synthetic_root.glob(
            "sample_*/"
            "coco_data/"
            "images/*"
        )
    )

    if not real_files:
        raise RuntimeError(
            "Real Reference가 없습니다."
        )

    if not synthetic_files:
        raise RuntimeError(
            "Synthetic RGB가 없습니다."
        )

    columns = min(
        len(real_files),
        len(synthetic_files),
        6,
    )

    real_row = make_row(
        real_files,
        "REAL",
        columns,
    )

    synthetic_row = make_row(
        synthetic_files,
        "SYNTHETIC",
        columns,
    )

    canvas = Image.new(
        "RGB",
        (
            max(
                real_row.width,
                synthetic_row.width,
            ),
            (
                real_row.height
                + synthetic_row.height
            ),
        ),
        "white",
    )

    canvas.paste(
        real_row,
        (0, 0),
    )

    canvas.paste(
        synthetic_row,
        (0, real_row.height),
    )

    output_path = (
        synthetic_root
        / "domain_gap_grid.jpg"
    )

    canvas.save(
        output_path,
        quality=92,
    )

    print("Saved:", output_path)


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

실제 Reference와 Synthetic Render를 같은 화면에서 비교하여 **Domain Gap을 눈으로 확인**하기 위해 작성합니다. Main에서는 `cracker_box`와 `mustard_bottle` 중 한 Class에만 실제 사진이 몰리지 않도록 Class별 Reference를 나누어 선택합니다.

### 의사코드로 읽어보기

```text
Set의 Class별 Real Reference를 찾는다
        ↓
Class별 Reference를 균형 있게 선택한다
        ↓
Synthetic COCO RGB를 찾는다
        ↓
Real 일부를 한 줄에 배치한다
        ↓
Synthetic 일부를 다음 줄에 배치한다
        ↓
한 장의 비교 Grid로 저장한다
```

실행:

```bash
python scripts/08_compare_real_synthetic.py \
  --asset-set main \
  --tag baseline
```

결과:

```text
results/main/baseline/domain_gap_grid.jpg
```

# 32. Human QA와 Domain Gap을 직접 확인합니다

다음 항목을 확인합니다.

| 항목 | 확인 질문 |
|---|---|
| Shape | 실제 Reference와 형태가 비슷한가 |
| Size | Object가 너무 크거나 작지 않은가 |
| Camera | 실제 촬영에서 가능한 시점인가 |
| Light | 지나치게 밝거나 어둡지 않은가 |
| Material | 실제 재질과 너무 다르지 않은가 |
| Background | 실제보다 지나치게 단순하지 않은가 |
| Occlusion | 실제에는 가림이 많은데 Synthetic에는 없는가 |
| Noise / Blur | Synthetic이 지나치게 깨끗하지 않은가 |
| BBox | Object를 정상적으로 감싸는가 |
| Mask | Object 경계를 따라가는가 |
| Depth | Camera 거리와 일관되는가 |

`reports/main/baseline_qa.csv`의 `human_decision`:

```text
approved
needs_review
rejected
```

이 CSV에 Human QA를 작성한 뒤에는 같은 QA 명령을 다시 실행하지 않습니다. 자동 QA를 정말 다시 만들어야 할 때만 기존 기록을 따로 보관한 뒤 `--overwrite`를 사용합니다.

예:

```text
needs_review

BBox와 Mask는 정상이나
실제 YCB-V 장면보다
배경과 조명이 너무 단순하다.
```

---

# PART 6. 3일차와 4일차를 연결하고 Metadata를 해석합니다

# 33. 3일차 Cut-Paste와 4일차 Rendering을 비교합니다

| 비교 | 3일차 Cut-Paste | 4일차 3D Rendering |
|---|---|---|
| Source | JPG + RGBA PNG | 3D Model |
| 공간 | 2D Pixel | 3D World |
| 위치 | x, y | x, y, z |
| 회전 | 2D Rotation | 3D Rotation |
| Camera | 원본에 고정 | 직접 제어 |
| Light | 원본에 포함 | 직접 제어 |
| BBox | 붙인 Pixel 영역 | Camera Projection |
| Semantic | 별도 제작 필요 | 자동 생성 가능 |
| Instance | 별도 제작 필요 | 자동 생성 가능 |
| Depth | 만들기 어려움 | 자동 생성 가능 |
| 장점 | 빠르고 단순 | 다양한 Ground Truth |
| 주요 한계 | 합성 느낌 | Domain Gap 관리 필요 |

# 34. Metadata에서 한 Sample의 생성 조건을 다시 추적합니다

```bash
python - <<'PY'
import pandas as pd

df = pd.read_csv(
    "reports/main/baseline_metadata.csv"
)

print(
    df[
        df["sample"]
        == "sample_0203"
    ].T
)
PY
```

확인:

```text
Seed
Camera Distance
Camera X / Y / Z
Light X / Y / Z
Light Energy
Object 위치
Object Rotation
```

이것이 오늘의 Data Lineage입니다.

같은 Seed를 사용한 실험을 다시 비교할 때는 `reports/environment_freeze_day04.txt`도 함께 보관합니다. Random 조건은 Seed와 Metadata로 추적하고, 세부 Render 차이는 Blender / BlenderProc / GPU 환경 차이의 영향을 받을 수 있습니다.

# 35. Main 본 실습의 전체 흐름을 연결합니다

```text
Prepared YCB-V Main
        ↓
cracker_box
mustard_bottle
        ↓
Real Reference 20장
        ↓
Single Render
        ↓
RGB / Depth / Semantic / Instance / COCO
        ↓
Randomized Render 8장
        ↓
Metadata
        ↓
Automatic QA
        ↓
Human QA
        ↓
Real vs Synthetic
        ↓
Domain Gap
```

다음을 자신의 결과로 작성합니다.

```text
3D Rendering에서 BBox를 자동 생성할 수 있는 이유:

Semantic과 Instance의 가장 큰 차이:

가장 눈에 띈 Domain Gap:

Metadata로 다시 찾은 실패 조건:
```

---

# PART 7. Mini Challenge — `power_drill`로 4일차 전체 흐름을 스스로 반복합니다

# 36. Mini Challenge의 목표를 확인합니다

본 실습:

```text
main
→ cracker_box + mustard_bottle
```

Mini Challenge:

```text
challenge
→ power_drill
```

새 알고리즘을 만드는 문제가 아닙니다.

오늘 만든 같은 Pipeline을 새 Object에 적용합니다.

```text
Input Check
        ↓
Real Preview
        ↓
Single Render
        ↓
Baseline Camera
        ↓
Far Camera
        ↓
다른 조건 유지
        ↓
BBox 크기 비교
        ↓
QA
        ↓
Real vs Synthetic
        ↓
결론
```

핵심 질문:

> **Camera를 멀리 두는 조건 하나만 바꾸면 화면 속 Object와 BBox 크기, 실제 Reference와의 차이는 어떻게 달라질까?**

# 37. Challenge 요구사항 1 — 입력 데이터를 확인합니다

```bash
python scripts/00_check_day04.py \
  --asset-set challenge
```

확인:

```text
Class Name      : power_drill
Lesson Class ID : 3
BOP Object ID   : 15
3D Model        : obj_000015.ply
Real Reference  : 10
```

# 38. Challenge 요구사항 2 — 실제 power_drill을 확인합니다

```bash
python scripts/01_preview_real_reference.py \
  --asset-set challenge
```

결과:

```text
results/challenge/source/real_reference_grid.jpg
```

# 39. Challenge 요구사항 3 — Single Render를 수행합니다

```bash
blenderproc run \
  scripts/02_render_single_scene.py \
  --asset-set challenge \
  --tag single \
  --seed 42
```

```bash
blenderproc vis hdf5 \
  results/challenge/single/0.hdf5
```

```bash
python scripts/03_export_hdf5_preview.py \
  --input results/challenge/single/0.hdf5 \
  --output results/challenge/single/previews
```

다음을 확인합니다.

```text
RGB
Depth
Semantic
Instance
COCO BBox
```

Challenge는 Object가 하나이므로 Semantic과 Instance가 화면상 비슷하게 보일 수 있지만 의미는 다릅니다.

# 40. Challenge 요구사항 4 — 기준 Camera Distance로 6장을 생성합니다

```text
Seed
500 ~ 505

Camera Distance
0.65 ~ 0.85

Light
350 ~ 800

Rotation
±25°
```

```bash
python scripts/05_generate_dataset.py \
  --asset-set challenge \
  --tag camera_baseline \
  --count 6 \
  --seed-start 500 \
  --camera-distance-min 0.65 \
  --camera-distance-max 0.85 \
  --light-min 350 \
  --light-max 800 \
  --rotation-max-deg 25
```

```bash
python scripts/06_collect_metadata.py \
  --asset-set challenge \
  --tag camera_baseline
```

```bash
python scripts/07_qa_3d_dataset.py \
  --asset-set challenge \
  --tag camera_baseline
```

# 41. Challenge 요구사항 5 — Camera Distance만 변경합니다

다른 조건은 모두 유지합니다.

같은 Script와 같은 Seed를 사용하므로 Object 위치·회전, Light, Camera X·Z에 사용되는 Random 값의 순서는 동일하게 유지됩니다. 두 실험에서 Camera Distance 범위만 다르게 지정합니다.

Camera가 이동하면 Object를 계속 바라보도록 Camera 방향은 자동으로 다시 계산됩니다. 이것은 Camera Distance 변경에 따라 생기는 종속 결과이며, 새로운 Random 변수를 추가한 것이 아닙니다.

따라서 비교 순서는 다음과 같습니다.

```text
Metadata 비교
        ↓
Camera Distance 이외 조건 SAME 확인
        ↓
BBox Area Ratio 비교
        ↓
Real vs Synthetic 비교
```

```text
Count
→ 6 그대로

Seed
→ 500 ~ 505 그대로

Light
→ 350 ~ 800 그대로

Rotation
→ ±25° 그대로

변경
→ Camera Distance만
```

```text
Baseline
0.65 ~ 0.85

Far
0.95 ~ 1.15
```

실행:

```bash
python scripts/05_generate_dataset.py \
  --asset-set challenge \
  --tag camera_far \
  --count 6 \
  --seed-start 500 \
  --camera-distance-min 0.95 \
  --camera-distance-max 1.15 \
  --light-min 350 \
  --light-max 800 \
  --rotation-max-deg 25
```

```bash
python scripts/06_collect_metadata.py \
  --asset-set challenge \
  --tag camera_far
```

```bash
python scripts/07_qa_3d_dataset.py \
  --asset-set challenge \
  --tag camera_far
```

# 42. 두 실험의 BBox 크기를 비교하는 코드를 작성합니다

**파일: `scripts/09_compare_experiments.py`**

```python
from __future__ import annotations

import argparse
import json
from pathlib import Path

import numpy as np
import pandas as pd


ROOT = Path(__file__).resolve().parents[1]


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
        "--tag-a",
        required=True,
    )

    parser.add_argument(
        "--tag-b",
        required=True,
    )

    return parser.parse_args()


def collect(
    asset_set,
    tag,
):
    source_root = (
        ROOT
        / "results"
        / asset_set
        / tag
    )

    rows = []

    for sample_dir in sorted(
        source_root.glob("sample_*")
    ):
        coco_path = (
            sample_dir
            / "coco_data"
            / "coco_annotations.json"
        )

        if not coco_path.exists():
            continue

        with coco_path.open(
            "r",
            encoding="utf-8",
        ) as file:
            coco = json.load(file)

        images = {
            int(item["id"]): item
            for item
            in coco.get("images", [])
        }

        for annotation in coco.get(
            "annotations",
            [],
        ):
            image_info = images.get(
                int(
                    annotation["image_id"]
                )
            )

            if image_info is None:
                continue

            _, _, width, height = (
                annotation["bbox"]
            )

            image_area = (
                float(image_info["width"])
                * float(
                    image_info["height"]
                )
            )

            bbox_area_ratio = (
                width
                * height
                / image_area
            )

            rows.append(
                {
                    "tag": tag,
                    "sample": (
                        sample_dir.name
                    ),
                    "category_id": (
                        annotation[
                            "category_id"
                        ]
                    ),
                    "bbox_area_ratio": (
                        bbox_area_ratio
                    ),
                }
            )

    return rows


def load_metadata(
    asset_set,
    tag,
):
    path = (
        ROOT
        / "reports"
        / asset_set
        / f"{tag}_metadata.csv"
    )

    if not path.exists():
        raise FileNotFoundError(
            path
        )

    return (
        pd.read_csv(path)
        .sort_values("sample")
        .reset_index(drop=True)
    )


def verify_non_distance_conditions(
    asset_set,
    tag_a,
    tag_b,
):
    first = load_metadata(
        asset_set,
        tag_a,
    )

    second = load_metadata(
        asset_set,
        tag_b,
    )

    if not first["sample"].equals(
        second["sample"]
    ):
        raise RuntimeError(
            "두 실험의 Sample 구성이 다릅니다."
        )

    ignore_columns = {
        "camera_distance",
        "camera_y",
        "camera_distance_min",
        "camera_distance_max",
        "approved",
    }

    compare_columns = [
        column
        for column in first.columns
        if (
            column in second.columns
            and column not in ignore_columns
        )
    ]

    mismatches = []

    for column in compare_columns:
        left = first[column]
        right = second[column]

        if (
            pd.api.types.is_numeric_dtype(left)
            and pd.api.types.is_numeric_dtype(right)
        ):
            same = np.allclose(
                left.to_numpy(),
                right.to_numpy(),
                rtol=1e-9,
                atol=1e-9,
                equal_nan=True,
            )
        else:
            same = (
                left.astype(str)
                .equals(
                    right.astype(str)
                )
            )

        if not same:
            mismatches.append(
                column
            )

    return mismatches


def main() -> None:
    args = parse_args()

    mismatches = (
        verify_non_distance_conditions(
            args.asset_set,
            args.tag_a,
            args.tag_b,
        )
    )

    if mismatches:
        raise RuntimeError(
            "Camera Distance 외 조건이 "
            "달라졌습니다: "
            + ", ".join(mismatches)
        )

    print(
        "Non-distance conditions: SAME"
    )

    rows = []

    rows.extend(
        collect(
            args.asset_set,
            args.tag_a,
        )
    )

    rows.extend(
        collect(
            args.asset_set,
            args.tag_b,
        )
    )

    dataframe = pd.DataFrame(rows)

    if dataframe.empty:
        raise RuntimeError(
            "비교할 Annotation이 없습니다."
        )

    summary = (
        dataframe
        .groupby("tag")[
            "bbox_area_ratio"
        ]
        .agg(
            [
                "count",
                "mean",
                "median",
                "min",
                "max",
            ]
        )
        .reset_index()
    )

    report_dir = (
        ROOT
        / "reports"
        / args.asset_set
    )

    report_dir.mkdir(
        parents=True,
        exist_ok=True,
    )

    detail_path = (
        report_dir
        / (
            f"{args.tag_a}"
            f"_vs_"
            f"{args.tag_b}"
            f"_bbox_detail.csv"
        )
    )

    summary_path = (
        report_dir
        / (
            f"{args.tag_a}"
            f"_vs_"
            f"{args.tag_b}"
            f"_bbox_summary.csv"
        )
    )

    dataframe.to_csv(
        detail_path,
        index=False,
        encoding="utf-8-sig",
    )

    summary.to_csv(
        summary_path,
        index=False,
        encoding="utf-8-sig",
    )

    print(
        summary.to_string(
            index=False
        )
    )

    print("Detail :", detail_path)
    print("Summary:", summary_path)


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

먼저 두 실험의 Metadata를 비교하여 **Camera Distance 외 조건이 실제로 같았는지 확인**합니다.

그 다음 “멀어져서 작아 보인다”를 눈으로만 판단하지 않고 **COCO BBox가 이미지 전체에서 차지하는 비율을 숫자로 비교**합니다.

### 의사코드로 읽어보기

```text
Baseline / Far Metadata를 읽는다
        ↓
같은 Sample끼리 정렬한다
        ↓
Camera Distance 관련 열을 제외한 조건이 같은지 확인한다
        ↓
다르면 한 변수 실험이 아니므로 중단한다
        ↓
두 실험의 COCO를 읽는다
        ↓
BBox Width × Height를 계산한다
        ↓
전체 Image Area로 나눈다
        ↓
BBox Area Ratio를 만든다
        ↓
실험별 Mean / Median / Min / Max를 계산한다
        ↓
CSV로 저장한다
```

실행:

```bash
python scripts/09_compare_experiments.py \
  --asset-set challenge \
  --tag-a camera_baseline \
  --tag-b camera_far
```

정상이라면 BBox 통계보다 먼저 다음 문장이 출력되어야 합니다.

```text
Non-distance conditions: SAME
```

이 문장이 확인되어야 `camera_baseline`과 `camera_far`를 한 변수 실험으로 비교할 수 있습니다.

일반적으로 Camera가 멀어지면 Object가 작게 보이고 BBox Area Ratio도 작아질 가능성이 있습니다.

하지만 **실제 실행 결과를 보고 결론**을 작성합니다.

# 43. Challenge 요구사항 6 — Real vs Synthetic을 비교합니다

```bash
python scripts/08_compare_real_synthetic.py \
  --asset-set challenge \
  --tag camera_baseline
```

```bash
python scripts/08_compare_real_synthetic.py \
  --asset-set challenge \
  --tag camera_far
```

비교:

```text
results/challenge/camera_baseline/domain_gap_grid.jpg

results/challenge/camera_far/domain_gap_grid.jpg
```

| 비교 항목 | Camera Baseline | Camera Far |
|---|---|---|
| Object 화면 크기 | | |
| 평균 BBox Area Ratio | | |
| 실제 Reference와 크기 유사성 | | |
| 형태 확인 난이도 | | |
| Domain Gap | | |
| 더 적절한 조건 | | |

# 44. Challenge 요구사항 7 — `challenge_notes.md`를 작성합니다

**파일: `reports/challenge/challenge_notes.md`**

```markdown
# Day 04 Mini Challenge - power_drill

## 1. Challenge Source

- 3D Model:
- BOP Object ID:
- Lesson Class ID:
- Real Reference 수:

## 2. Single Render

- RGB:
- Depth:
- Semantic:
- Instance:
- COCO BBox:

## 3. Camera Baseline

- Seed:
- Count:
- Camera Distance:
- Light:
- Rotation:
- Automatic QA:
- Human QA:

## 4. Camera Far

- Seed:
- Count:
- Camera Distance:
- Light:
- Rotation:
- Automatic QA:
- Human QA:

## 5. 한 변수 비교

- 유지한 조건:
- 변경한 조건:
- Non-distance conditions SAME 여부:
- Baseline 평균 BBox Area Ratio:
- Far 평균 BBox Area Ratio:
- 가장 큰 차이:

## 6. Domain Gap

- 실제와 더 비슷한 조건:
- 가장 큰 차이:
- 실제 환경에 부족한 요소:

## 7. 실패 사례

- Sample:
- 문제:
- Metadata에서 확인한 조건:
- 원인 가설:
- 수정 방법:

## 8. 최종 선택

- 선택한 Camera Distance:
- 선택한 이유:

## 9. 오늘의 결론

- 3D에서 Ground Truth가 자동 생성되는 이유:
- Automatic QA만으로 부족한 이유:
- Domain Gap을 줄이기 위해 추가하고 싶은 조건:
```

# 45. Mini Challenge 완료 조건을 확인합니다

- [ ] `power_drill` 3D Model을 확인했다.
- [ ] BOP ID 15와 Lesson Class ID 3을 구분했다.
- [ ] Real Reference 10장을 확인했다.
- [ ] Single Render를 성공했다.
- [ ] RGB·Depth·Semantic·Instance·COCO BBox를 확인했다.
- [ ] `camera_baseline` 6장을 생성했다.
- [ ] `camera_far` 6장을 생성했다.
- [ ] 두 실험에서 같은 Seed를 사용했다.
- [ ] Count·Light·Rotation을 동일하게 유지했다.
- [ ] Camera Distance만 변경했다.
- [ ] Metadata를 수집했다.
- [ ] `Non-distance conditions: SAME`을 확인했다.
- [ ] Automatic QA를 수행했다.
- [ ] BBox Area Ratio를 비교했다.
- [ ] Real vs Synthetic Grid를 만들었다.
- [ ] Human QA를 수행했다.
- [ ] 실패 사례를 한 개 이상 찾았다.
- [ ] Metadata에서 실패 원인을 다시 추적했다.
- [ ] 더 적절한 Camera Distance를 근거와 함께 선택했다.
- [ ] `challenge_notes.md`를 작성했다.

# 46. Mini Challenge를 약 3시간으로 진행합니다

| 단계 | 권장 시간 |
|---|---:|
| Challenge 데이터 확인·Real Preview | 20분 |
| Single Render·Ground Truth 확인 | 30분 |
| Camera Baseline 6장 생성 | 30분 |
| Camera Far 6장 생성 | 30분 |
| Metadata·Automatic QA | 20분 |
| BBox Area Ratio 비교 | 20분 |
| Real vs Synthetic·Human QA | 20분 |
| `challenge_notes.md` 작성 | 10분 |
| **합계** | **180분** |

---

# PART 8. 오늘 만든 결과를 정리하고 버전관리합니다

# 47. `day04_notes.md`를 작성합니다

**파일: `day04_notes.md`**

```markdown
# Subject 11 - Day 04

## 1. 오늘의 목적

YCB-V 3D Object를 BlenderProc로 Rendering하고
RGB·BBox·Semantic·Instance·Depth를 함께 생성한다.

## 2. Main Source

- cracker_box
  - BOP ID:
  - Class ID:
  - Real Reference:

- mustard_bottle
  - BOP ID:
  - Class ID:
  - Real Reference:

## 3. Single Render

- RGB:
- Depth:
- Semantic:
- Instance:
- COCO BBox:

## 4. Main Randomized Dataset

- Tag:
- Count:
- Seed:
- Camera Distance:
- Light:
- Rotation:

## 5. Metadata

- 추적한 Sample:
- Camera:
- Light:
- Object Position:
- Object Rotation:

## 6. Automatic QA

- PASS:
- FAIL:
- 주요 오류:

## 7. Human QA

- Approved:
- Needs Review:
- Rejected:
- 주요 이유:

## 8. Domain Gap

- Real과 가장 큰 차이:
- Synthetic이 지나치게 단순한 요소:
- 추가하면 좋을 조건:

## 9. 3일차와 4일차 비교

- Cut-Paste 장점:
- 3D Rendering 장점:
- BBox 생성 방식 차이:

## 10. Mini Challenge

- power_drill Baseline:
- power_drill Far:
- BBox 차이:
- 더 적절한 조건:
- 선택 이유:

## 11. 오늘의 결론

-
```

# 48. README에 4일차 내용을 정리합니다

````markdown
# Subject 11 - Day 04
## 3D Rendering · Segmentation · BlenderProc

### Main Data

```text
cracker_box
BOP ID 2
Lesson Class 1
Real Reference 10

mustard_bottle
BOP ID 5
Lesson Class 2
Real Reference 10
```

### Mini Challenge

```text
power_drill
BOP ID 15
Lesson Class 3
Real Reference 10
```

### Pipeline

```text
YCB-V 3D Model
        ↓
BlenderProc BOP Loader
        ↓
Object / Camera / Light
        ↓
RGB / Depth / Semantic / Instance / COCO BBox
        ↓
Metadata
        ↓
Automatic QA
        ↓
Human QA
        ↓
Real vs Synthetic
```

### Scripts

```text
00_check_day04.py
01_preview_real_reference.py
02_render_single_scene.py
03_export_hdf5_preview.py
04_render_random_sample.py
05_generate_dataset.py
06_collect_metadata.py
07_qa_3d_dataset.py
08_compare_real_synthetic.py
09_compare_experiments.py
```

### Key Rule

```text
Rendering 성공
≠
학습데이터 승인

Rendering
→ Ground Truth 확인
→ Automatic QA
→ Human QA
→ Real Reference 비교
→ 사용 여부 판단
```

### Next

Day 05에서는 사람 움직임을
Keypoint Sequence로 표현하고
Sequence Augmentation과 QA를 수행한다.
````

# 49. `.gitignore`를 작성합니다

```gitignore
# Python
__pycache__/
*.pyc

# Editor
.vscode/

# Student Data
data/day04_student_data/

# Generated Rendering Results
results/

# Heavy 3D / Rendering Files
*.ply
*.hdf5
*.exr

# OS
.DS_Store
Thumbs.db
```

# 50. 기존 Git 저장소를 그대로 사용합니다

교과 11 Root로 이동합니다.

```bash
cd ~/ai_vision/subject11_synthetic_data
```

```bash
git status
```

```bash
git add day04_3d_segmentation
```

다시 확인합니다.

```bash
git status
```

다음 대용량 파일이 Stage에 포함되지 않았는지 확인합니다.

```text
data/day04_student_data/
results/
*.ply
*.hdf5
```

Commit:

```bash
git commit -m "feat: add day04 blenderproc 3d synthetic lab"
```

기존 Remote가 있고 Push가 허용된 환경이면:

```bash
git push
```

새 Git 저장소를 다시 만들지 않습니다.

# 51. 하루가 끝났을 때 자가 체크합니다

| 확인사항 | 완료 |
|---|:---:|
| 1~3일차와 같은 프로젝트를 이어서 사용했다 | □ |
| 1일차 `.venv`를 재사용했다 | □ |
| 일반 Python 라이브러리를 확인했다 | □ |
| `blenderproc --help`를 확인했다 | □ |
| BlenderProc Quickstart를 실행하고 `0.hdf5`를 확인했다 | □ |
| `day04_student_data`를 올바르게 배치했다 | □ |
| Main과 Challenge를 구분했다 | □ |
| BOP ID와 Lesson Class ID를 구분했다 | □ |
| Main Real Reference 20장을 확인했다 | □ |
| Main 3D Model 2개를 확인했다 | □ |
| Single Scene Rendering에 성공했다 | □ |
| RGB를 확인했다 | □ |
| Depth를 확인했다 | □ |
| Semantic Mask를 확인했다 | □ |
| Instance Mask를 확인했다 | □ |
| COCO BBox를 확인했다 | □ |
| HDF5 Preview를 만들었다 | □ |
| Random Smoke Test를 수행했다 | □ |
| Main Randomized Sample 8장을 만들었다 | □ |
| Metadata를 CSV로 수집했다 | □ |
| Automatic QA를 수행했다 | □ |
| Human QA를 수행했다 | □ |
| Real vs Synthetic Grid를 만들었다 | □ |
| 실패 Sample의 Metadata를 추적했다 | □ |
| 3일차와 4일차 BBox 생성 차이를 설명할 수 있다 | □ |
| Mini Challenge에서 power_drill을 사용했다 | □ |
| Camera Distance 하나만 변경했다 | □ |
| `Non-distance conditions: SAME`을 확인했다 | □ |
| Seed는 Random 조건 추적 기준이며 다른 환경의 픽셀 동일성을 보장하지 않는다는 점을 이해했다 | □ |
| BBox Area Ratio를 비교했다 | □ |
| Mini Challenge Domain Gap을 비교했다 | □ |
| `challenge_notes.md`를 작성했다 | □ |
| `day04_notes.md`를 작성했다 | □ |
| README를 작성했다 | □ |
| Git Commit을 완료했다 | □ |

# 52. 4일차 최종 정리

오늘은 단순히 3D 이미지를 Rendering한 것이 아닙니다.

```text
Prepared YCB-V Data
        ↓
3D Model + Real Reference
        ↓
BOP Object ID
        ↓
BlenderProc
        ↓
Object Position / Rotation
Camera
Light
        ↓
Rendering
        ↓
RGB
Depth
Semantic Mask
Instance Mask
COCO BBox
        ↓
Metadata
        ↓
Automatic QA
        ↓
Human QA
        ↓
Real vs Synthetic
        ↓
Domain Gap
```

오늘 반드시 기억해야 할 세 가지는 다음입니다.

## ① 3D Scene은 Ground Truth를 알고 있습니다

```text
Object의 위치·형태·Class를 알고 있음
        ↓
Camera Projection
        ↓
BBox / Mask / Depth
자동 생성 가능
```

## ② Randomization은 다양성과 현실성을 함께 봅니다

```text
Random 범위 증가
        ↓
다양성 증가 가능

하지만

비현실적 Camera
비현실적 Light
비현실적 Object 위치
        ↓
학습에 도움이 되지 않을 수 있음
```

따라서:

```text
Randomization
→ QA
→ 범위 조정
```

이 필요합니다.

## ③ Synthetic과 Real의 차이를 반드시 확인합니다

```text
Synthetic이 깨끗하고 예쁨
        ↓
좋은 학습데이터
X

Synthetic
        ↓
실제 Camera 환경과 비교
        ↓
Domain Gap 확인
        ↓
부족 조건 보완
O
```

# 다음 시간 연결

4일차에서는:

```text
3D Scene
→ RGB / BBox / Mask / Depth
```

를 다뤘습니다.

5일차에서는:

```text
사람 움직임
→ Keypoint
→ Frame Feature
→ Sequence
→ Sequence Augmentation
→ QA
```

로 데이터 형태가 달라집니다.

교과 11은 다음처럼 서로 다른 데이터 형태에 합성·보강 방법을 적용합니다.

```text
Image
        ↓
Generative Model
        ↓
Detection
        ↓
3D Rendering
        ↓
Motion Sequence
```
