
> **오늘의 핵심:** 4일차에는 숫자형 입력을 Sampling하고 Threshold로 `NORMAL / WARNING`을 판단한 뒤 GPIO와 Event Log로 연결했습니다.  
> 5일차에는 같은 판단 구조를 **USB Camera의 실제 Frame**에 적용하여, 사람이 직접 만든 Rule로 첫 번째 완성형 Edge Warning System을 만듭니다.
>
> 1~4일차에 사용한 `subject13_edge_ai` 프로젝트를 그대로 이어서 사용합니다. **새 프로젝트와 새 가상환경을 만들지 않습니다.**
>
> 실제 IP, Hostname, Camera index, 조명, Camera 해상도, HSV 범위, 면적 Threshold는 환경마다 달라질 수 있습니다. 교재의 예시값을 정답처럼 외우지 않고 자신의 장비에서 확인한 값을 사용합니다.
>
> 비밀번호, Wi-Fi 비밀번호, 인증키 같은 민감한 정보는 README·Report·Git에 기록하지 않습니다.

---

## 오늘 가장 중요한 질문

1일차부터 계속 사용한 Edge Pipeline의 큰 구조는 같습니다.

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

4일차에는 숫자형 입력을 사용했습니다.

```text
4일차

Sensor Value
     ↓
Sampling / Timestamp
     ↓
Threshold
     ↓
NORMAL / WARNING
     ↓
GPIO
     ↓
Event Log
```

오늘은 **판단에 사용하는 측정값**을 Camera Frame에서 만듭니다.

```text
5일차

USB Camera
     ↓
OpenCV Frame
     ↓
BGR → HSV
     ↓
Red Color Mask
     ↓
Red Area
     ↓
Area Threshold
     ↓
NORMAL / WARNING
     │
     ├─ GPIO
     │   ├─ Green LED
     │   └─ Red LED / 선택 Buzzer
     │
     └─ Event Log
         └─ WARNING Image
```

따라서 오늘 가장 중요한 질문은 다음입니다.

> **“Camera Frame에서 사람이 정한 Rule로 측정값을 만들고, 그 값을 운영 기준과 비교하여 실제 출력과 기록까지 연결하려면 어떤 파일과 코드가 어떤 순서로 동작해야 하는가?”**

오늘은 아직 AI 모델을 사용하지 않습니다.

```text
5일차
사람이 직접 만든 Rule
HSV + Red Area + Threshold
        ↓
6일차
같은 문제를 이미지 Dataset으로 만들고
작은 Tiny CNN을 학습
        ↓
7일차
PyTorch Model → ONNX
→ Raspberry Pi로 다시 배포
```

6일차의 기본 Class는 다음 두 개를 사용합니다.

```text
normal
warning_red
```

5일차와 **같은 문제**를 사용해야 나중에 Rule과 AI의 판단 방식을 직접 비교할 수 있습니다.

---

## 오늘의 학습 목표

수업이 끝나면 다음을 할 수 있어야 합니다.

```text
1. 4일차의 Threshold 판단 구조와 5일차의 Camera Rule 구조를 연결해 설명한다.
2. OpenCV Frame이 BGR이라는 것을 설명하고 HSV로 변환하는 이유를 이해한다.
3. 빨간색을 HSV 두 Hue 구간으로 Masking하는 이유를 설명한다.
4. Mask에서 Red Area를 계산하고 Threshold로 NORMAL / WARNING을 판단한다.
5. 감으로 Threshold를 정하지 않고 실제 관찰값을 보고 기준을 정한다.
6. 저장 이미지 → 실시간 Camera 순서로 작은 성공을 확인한다.
7. 3일차 CameraInput과 GPIOOutput을 새로 만들지 않고 재사용한다.
8. 4일차 Event Log 개념을 Camera Rule에 다시 적용한다.
9. WARNING으로 바뀐 순간의 대표 Frame을 저장한다.
10. 실행 결과가 어느 파일의 어느 코드에서 만들어졌는지 역추적한다.
11. 조명·거리·배경 변화에서 Rule의 실패를 코드 오류와 구분한다.
12. 오늘 만든 Rule Baseline이 6일차 AI Baseline과 어떤 관계인지 설명한다.
```

---

## 오늘의 8시간 학습 흐름

장비 상태와 실습 속도에 따라 실제 시간은 조금 달라질 수 있습니다.

| 학습 블록 | 권장 시간 | 무엇을 배우나요? | 끝나면 할 수 있어야 하는 것 |
|---|---:|---|---|
| 1 | 40분 | 4일차 상태 확인 · 오늘 전체 Pipeline | Sensor Threshold와 Camera Rule의 공통 구조를 설명할 수 있다 |
| 2 | 55분 | BGR · HSV · 저장 Frame 확보 | Camera Frame을 HSV 관점에서 이해하고 테스트 이미지를 준비할 수 있다 |
| 3 | 75분 | `RedAreaRule` · 저장 이미지 Guided Lab | 한 장의 이미지에서 Mask·Area·State를 만들 수 있다 |
| 4 | 65분 | 실시간 Camera Probe · Threshold 결정 | 실제 Camera 값으로 Threshold 후보를 비교할 수 있다 |
| 5 | 75분 | Event Log · WARNING Image · GPIO 통합 | Camera→Rule→GPIO→Log를 End-to-End로 실행할 수 있다 |
| 6 | 60분 | 실패 조건 실험 · Rule 한계 분석 | 조명·거리·배경 변화의 영향을 원인별로 설명할 수 있다 |
| 7 | 70분 | Mini Challenge | 결과 역추적·예측·설정 변경·기능 수정·오류 해결을 스스로 수행할 수 있다 |
| 8 | 40분 | 기본 상태 복원 · Report · README · Git · 복습 · 6일차 연결 | 정상 Baseline을 보존하고 다음 수업으로 연결할 수 있다 |

총 480분을 기준으로 구성합니다.

---

## 오늘 사용할 실제 장비와 데이터

오늘은 별도의 공개 Dataset을 다운로드하지 않습니다.

입력 데이터는 **Raspberry Pi에 연결된 USB Camera가 직접 촬영한 Frame**입니다.

```text
실제 장비
→ Raspberry Pi
→ USB Camera
→ 빨간색 카드 또는 빨간색 물체
→ Green / Red LED
→ 선택 Buzzer
```

오늘 최소 두 가지 장면을 준비합니다.

```text
NORMAL 장면
→ 빨간색 경고 대상이 없음

WARNING 후보 장면
→ 빨간색 카드가 화면에 충분히 크게 들어옴
```

빨간색 대상은 **6일차 `warning_red` Class의 실제 의미**와 연결됩니다.

중요한 점은 다음입니다.

```text
6일차의 정답 Label
≠ 5일차 Rule이 출력한 결과를 그대로 복사

6일차의 정답 Label
= 실제 장면의 의미
  normal / warning_red
```

5일차 Rule이 잘못 판단한 장면도 6일차에서는 실제 장면 의미에 맞게 Label해야 합니다.

---

## 오늘 배울 기술 스택

| 기술 | 오늘 어디에 사용하나요? |
|---|---|
| Raspberry Pi OS / Linux | 실제 Edge 실행환경 |
| SSH | PC에서 Raspberry Pi Terminal 접속 |
| OpenCV | Camera Frame 읽기, BGR→HSV 변환, Mask, 이미지 저장 |
| NumPy | HSV 범위 배열 만들기 |
| Python | Rule 모듈과 통합 실행 Program 작성 |
| YAML | HSV 범위·Threshold·Log 경로 같은 설정값 관리 |
| `gpiozero` | 3일차의 LED/Buzzer 출력 모듈 재사용 |
| CSV | 상태 변화 Event 기록 |
| Git | 정상 Rule Baseline과 Report 보존 |

---

## 오늘도 계속 기억할 다섯 단계

```text
입력
  ↓
USB Camera Frame

처리
  ↓
BGR → HSV → Red Mask → Red Area

판단
  ↓
Area Threshold
→ NORMAL / WARNING

출력
  ↓
Green / Red LED / 선택 Buzzer

기록
  ↓
Event CSV / WARNING Image / Report
```

코드를 보다가 헷갈리면 다음 질문으로 돌아옵니다.

> **“지금 보고 있는 코드는 입력·처리·판단·출력·기록 중 어느 역할을 하는가?”**

---

## 실습 전에 자신의 환경값 적어두기

다음 값은 학생마다 또는 교육장 환경에 따라 달라질 수 있습니다.

```text
Raspberry Pi 사용자 이름 : ____________________
Raspberry Pi Hostname    : ____________________
Raspberry Pi IPv4        : ____________________

Project Path             : ~/ai_vision/subject13_edge_ai

Camera /dev/video 장치   : ____________________
OpenCV Camera Index      : ____________________

Green LED BCM            : ____________________
Red LED BCM              : ____________________
Buzzer 사용 여부          : 사용 / 미사용

오늘 요청 해상도          : ______ × ______
오늘 실제 Frame 해상도    : ______ × ______

최종 Red Area Threshold  : ____________________
```

이 교재에서는 환경에 따라 달라지는 값을 다음처럼 표시합니다.

```text
<RPI_USER>
→ 자신의 Raspberry Pi 사용자 이름

<RPI_IP>
→ 자신의 Raspberry Pi IPv4

<ACTUAL_CAMERA_INDEX>
→ camera_index_probe.py에서 실제 Frame을 읽은 번호
```

Camera index는 `configs/device.local.yaml`에 저장된 장치 전용 값을 사용합니다.

---

## 오늘의 성공 기준

수업이 끝났을 때 다음 흐름이 실제로 연결되면 됩니다.

```text
4일차 프로젝트 정상
        ↓
Camera / GPIO 독립 Test 정상
        ↓
NORMAL / WARNING 테스트 Frame 준비
        ↓
BGR → HSV
        ↓
Red Mask
        ↓
Red Area
        ↓
Threshold
        ↓
NORMAL / WARNING
        ↓
GPIO
        ↓
최초 상태 + 상태 변화 Event Log
        ↓
WARNING 전환 시 대표 이미지 저장
        ↓
실패 조건 분석
        ↓
Rule Baseline Report
        ↓
6일차 AI Dataset 준비로 연결
```

---

## 오늘 새로 설치하는 Package가 있나요?

5일차 핵심 실습은 3일차부터 사용한 `opencv-python-headless`, `numpy`, `gpiozero`와 기존 `PyYAML` 환경을 그대로 사용합니다.

따라서 정상적인 4일차 `.venv`라면 먼저 새 Package를 설치하지 않습니다.

다음 Import가 정상인지 확인할 수 있습니다.

```bash
python -c "import cv2, numpy, yaml; print('Day05 imports OK')"
```

GPIO를 사용하는 Raspberry Pi에서는:

```bash
python -c "import gpiozero; print('gpiozero OK')"
```

가 정상인지도 확인합니다.

Import 오류가 있을 때만 기존 `requirements.txt`와 `.venv` 상태를 확인합니다.

새 Package를 무조건 다시 설치하면 기존 정상 환경이 달라질 수 있으므로 먼저 현재 상태를 확인합니다.

---

# 1. 4일차 프로젝트에서 이어서 시작하기

4일차가 끝났다면 공통 Repository는 다음과 비슷합니다.

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
│   └── day04_sensor_sampling.md
│
├── scripts/
│   ├── button_sampling.py
│   ├── button_test.py
│   ├── camera_gpio_demo.py
│   ├── camera_index_probe.py
│   ├── camera_test.py
│   ├── capture_series.py
│   ├── check_config.py
│   ├── check_device.py
│   ├── gpio_test.py
│   ├── led_single_test.py
│   ├── run_pipeline.py
│   ├── sensor_event.py
│   ├── sensor_gpio_demo.py
│   ├── sensor_once.py
│   └── sensor_sampling.py
│
├── src/
│   ├── camera_input.py
│   ├── config_loader.py
│   ├── device_info.py
│   ├── event_logger.py
│   ├── gpio_output.py
│   ├── pipeline.py
│   ├── sensor_input.py
│   └── sensor_logger.py
│
├── .gitignore
├── README.md
└── requirements.txt
```

Raspberry Pi에만 존재하는 항목:

```text
.venv/
configs/device.local.yaml
data/
logs/
```

4일차까지의 연결은 다음과 같습니다.

```text
Button / Sensor
→ Sampling
→ Timestamp
→ Threshold
→ NORMAL / WARNING
→ GPIO
→ CSV
```

오늘은 숫자형 Sensor 대신 **Camera Frame 자체를 분석하여 판단값을 만들 것**입니다.

```text
USB Camera
→ OpenCV Frame
→ 색상 분석
→ Red Area
→ Threshold
→ NORMAL / WARNING
→ GPIO
→ Event Log
→ WARNING Image
```

---

# 2. Raspberry Pi에 접속하고 4일차 정상 상태 확인하기

오늘은 새 프로젝트를 만들지 않습니다.

PC에서 자신의 Raspberry Pi에 접속합니다.

```bash
ssh <RPI_USER>@<RPI_IP>
```

`.local` 이름이 교육장 Network에서 정상 동작한다면 다음처럼 사용할 수도 있습니다.

```bash
ssh <RPI_USER>@<RPI_HOSTNAME>.local
```

`.local`이 되지 않아도 Raspberry Pi가 고장난 것은 아닙니다. 현재 IPv4를 다시 확인하여 접속합니다.

접속 후 현재 장치와 위치를 확인합니다.

```bash
whoami
hostname
hostname -I
pwd
```

프로젝트로 이동합니다.

```bash
cd ~/ai_vision/subject13_edge_ai
```

가상환경을 활성화합니다.

```bash
source .venv/bin/activate
```

현재 Python이 Raspberry Pi 프로젝트의 `.venv`인지 확인합니다.

```bash
which python
```

장치 상태를 확인합니다.

```bash
python -m scripts.check_device
```

## Source 최신 상태는 전달 방식에 따라 확인합니다

2일차부터 Source를 어떤 방식으로 전달했는지에 따라 확인 방법이 다릅니다.

### 내부 GitLab/Gitea 같은 Remote를 사용하는 경우

```bash
git status
git pull
git log --oneline -7
```

4일차 Commit이 보이는지 확인합니다.

### SCP로 Source를 전달한 경우

Raspberry Pi 폴더가 Git Repository가 아닐 수 있습니다.

이 경우 `git pull`을 억지로 실행하지 않습니다.

```bash
ls
ls configs
ls src
ls scripts
```

4일차에서 사용한 파일이 존재하는지 확인합니다.

## 4일차까지의 기존 기능을 먼저 다시 확인합니다

Camera:

```bash
python -m scripts.camera_index_probe
python -m scripts.camera_test
```

GPIO:

```bash
python -m scripts.gpio_test
```

4일차의 숫자형 Pipeline도 짧게 확인할 수 있습니다.

```bash
python -m scripts.sensor_once
```

다음처럼 판단합니다.

```text
기존 Camera / GPIO / Sensor 기능 정상
→ 5일차 Rule 기능 추가

기존 기능부터 오류
→ 4일차까지의 환경을 먼저 복구
→ 정상 상태 확인
→ 그 다음 5일차 진행
```

새 기능을 추가하기 전에 기존 기능이 정상이라는 것을 확인하면, 이후 오류가 **기존 환경 문제인지 5일차 Rule 문제인지** 구분하기 쉬워집니다.

---

# 3. 오늘 사용할 실제 대상 준비하기

오늘의 기본 예제는 **빨간색 카드 또는 빨간색 물체**입니다.

가능하면 다음 조건을 준비합니다.

```text
빨간색 카드 1개
배경은 빨간색과 확실히 다른 색상
고정된 Camera 위치
가능하면 조명이 크게 흔들리지 않는 장소
```

처음부터 복잡한 물체를 사용하지 않습니다.

첫 성공 기준은 다음 정도면 충분합니다.

```text
빨간색 카드가 화면에 충분히 크게 들어오면
→ WARNING

빨간색 카드가 없거나 작으면
→ NORMAL
```

---

# 4. Camera Frame 한 장을 먼저 확보하기

Rule을 만들기 전에 **입력 이미지가 정상인지 먼저 확인**합니다.

## Camera 설정값 확인

공통 설정과 장치 전용 설정을 함께 확인합니다.

```bash
grep -E "image_width|image_height|capture_dir" \
  configs/settings.yaml

grep -E "camera_index" \
  configs/device.local.yaml
```

`camera_index`는 장치마다 달라질 수 있으므로 3일차부터 `device.local.yaml`에 저장해 사용합니다.

필요하면 다시 Probe합니다.

```bash
python -m scripts.camera_index_probe
```

Camera 테스트를 실행합니다.

```bash
python -m scripts.camera_test
```

정상이라면 실제 Frame 크기와 저장 경로가 출력되고 JPG 파일이 생성됩니다.

저장된 이미지를 확인합니다.

```bash
find data/captures -maxdepth 1 -type f | tail
```

## 오늘 Rule 테스트용 이미지 두 장 준비하기

Rule을 실시간 Camera에 바로 적용하지 않습니다.

먼저 **결과를 반복해서 확인할 수 있는 고정 이미지** 두 장을 준비합니다.

```text
NORMAL 이미지
→ 빨간색 경고 대상이 없음

WARNING 후보 이미지
→ 빨간색 카드가 화면에 충분히 크게 들어옴
```

예:

```text
data/captures/day05_normal.jpg
data/captures/day05_red.jpg
```

3일차의 `camera_test.py` 또는 `capture_series.py`로 촬영한 파일 중 원하는 장면을 골라 복사하거나 이름을 정리해도 됩니다.

예:

```bash
cp data/captures/<NORMAL_CAPTURE>.jpg \
   data/captures/day05_normal.jpg

cp data/captures/<RED_CAPTURE>.jpg \
   data/captures/day05_red.jpg
```

`<NORMAL_CAPTURE>`와 `<RED_CAPTURE>`에는 자신의 실제 파일명을 넣습니다.

확인합니다.

```bash
ls -lh \
  data/captures/day05_normal.jpg \
  data/captures/day05_red.jpg
```

두 파일이 모두 존재해야 다음 단계로 넘어갑니다.

---

# 5. BGR과 HSV를 실제 값으로 확인하기

OpenCV가 읽은 Camera Frame은 기본적으로 BGR 순서입니다.

```text
B
G
R
```

색상을 Rule로 다룰 때는 HSV가 더 편리한 경우가 많습니다.

```text
H
→ Hue, 색상 종류

S
→ Saturation, 색의 선명도

V
→ Value, 밝기
```

오늘은 이론을 길게 외우지 않습니다.

실제 Camera Frame을 HSV로 변환하고 결과를 비교합니다.

---

# 6. 첫 HSV 변환 프로그램 만들기

다음 파일을 만듭니다.

```text
scripts/hsv_preview.py
```

## 이 코드는 왜 만들까요?

`CameraInput`으로 받은 Frame은 BGR 형태입니다.

오늘의 Rule은 색상을 기준으로 하므로 BGR 값을 그대로 비교하기보다 HSV로 바꾸어 다음 세 정보를 분리해서 보는 것이 편리합니다.

```text
H
→ 색상 종류

S
→ 색의 선명도

V
→ 밝기
```

HSV 배열 자체를 일반 사진처럼 보면 의미를 이해하기 어렵습니다.

그래서 오늘은 다음을 저장합니다.

```text
원본 BGR 이미지
+
H Channel 확인 이미지
+
S Channel 확인 이미지
+
V Channel 확인 이미지
+
화면 중앙 Pixel의 BGR / HSV 숫자
```

## 의사코드

```text
공통 + Local 설정을 읽는다
        ↓
Camera를 연다
        ↓
Frame 한 장을 읽는다
        ↓
BGR Frame을 HSV로 변환한다
        ↓
H / S / V Channel을 각각 분리한다
        ↓
확인용 이미지로 저장한다
        ↓
화면 중앙 Pixel의
BGR 값과 HSV 값을 출력한다
        ↓
마지막에 Camera Resource를 닫는다
```

## 코드

```python
from pathlib import Path

import cv2

from src.camera_input import CameraInput
from src.config_loader import load_config


def main():
    config = load_config(
        "configs/settings.yaml"
    )

    camera = None

    output_dir = Path(
        "data/day05_debug"
    )

    output_dir.mkdir(
        parents=True,
        exist_ok=True,
    )

    try:
        camera = CameraInput(
            index=int(config["camera_index"]),
            width=int(config["image_width"]),
            height=int(config["image_height"]),
        )

        frame = camera.read()

        hsv = cv2.cvtColor(
            frame,
            cv2.COLOR_BGR2HSV,
        )

        h_channel, s_channel, v_channel = (
            cv2.split(hsv)
        )

        # H는 0~179 범위를 사용하므로
        # 사람이 보기 쉽게 0~255로 펼친 확인용 이미지를 만듭니다.
        h_view = cv2.normalize(
            h_channel,
            None,
            0,
            255,
            cv2.NORM_MINMAX,
        )

        cv2.imwrite(
            str(output_dir / "bgr.jpg"),
            frame,
        )

        cv2.imwrite(
            str(output_dir / "h_channel.jpg"),
            h_view,
        )

        cv2.imwrite(
            str(output_dir / "s_channel.jpg"),
            s_channel,
        )

        cv2.imwrite(
            str(output_dir / "v_channel.jpg"),
            v_channel,
        )

        height, width = frame.shape[:2]
        center_x = width // 2
        center_y = height // 2

        center_bgr = frame[
            center_y,
            center_x,
        ].tolist()

        center_hsv = hsv[
            center_y,
            center_x,
        ].tolist()

        print("=== HSV Preview ===")
        print(f"Frame Shape : {frame.shape}")
        print(
            f"Center BGR  : {center_bgr}"
        )
        print(
            f"Center HSV  : {center_hsv}"
        )
        print()

        print("Saved:")
        print(output_dir / "bgr.jpg")
        print(output_dir / "h_channel.jpg")
        print(output_dir / "s_channel.jpg")
        print(output_dir / "v_channel.jpg")

    finally:
        if camera is not None:
            camera.release()


if __name__ == "__main__":
    main()
```

## 실행

```bash
python -m scripts.hsv_preview
```

예상 형태:

```text
=== HSV Preview ===
Frame Shape : (480, 640, 3)
Center BGR  : [xx, xx, xx]
Center HSV  : [xx, xx, xx]

Saved:
data/day05_debug/bgr.jpg
data/day05_debug/h_channel.jpg
data/day05_debug/s_channel.jpg
data/day05_debug/v_channel.jpg
```

실제 숫자는 Camera가 보고 있는 장면에 따라 달라집니다.

파일을 확인합니다.

```bash
find data/day05_debug -maxdepth 1 -type f | sort
```

## 결과를 어떻게 해석할까요?

`frame.shape`와 `hsv.shape`는 보통 같은 높이·너비·Channel 수를 갖습니다.

```text
BGR
→ 같은 Pixel을 B / G / R 값으로 표현

HSV
→ 같은 Pixel을 H / S / V 값으로 표현
```

`h_channel.jpg`는 한 Frame 안에서 Hue 분포를 보기 쉽게 펼친 **확인용 이미지**입니다. 픽셀의 정확한 Hue 숫자는 터미널 출력이나 배열 값으로 확인합니다.

변환한다고 이미지의 위치 정보가 사라지는 것은 아닙니다.

같은 `x, y` 위치의 Pixel을 **다른 색상 표현 방식으로 다시 나타낸 것**입니다.

## 코드 리뷰 — `Center HSV`는 어디에서 만들어졌을까요?

코드를 다시 열어 다음 순서를 직접 찾습니다.

```text
camera.read()
        ↓
frame
        ↓
cv2.cvtColor(..., COLOR_BGR2HSV)
        ↓
hsv
        ↓
hsv[center_y, center_x]
        ↓
Center HSV 출력
```

다음 질문에 답해 봅니다.

```text
Camera를 실제로 읽는 파일은 무엇인가?
→ src/camera_input.py

BGR을 HSV로 바꾸는 파일은 무엇인가?
→ scripts/hsv_preview.py

중앙 Pixel 좌표를 만드는 코드는 어디인가?
→ center_x / center_y

H Channel 확인 이미지를 만드는 이유는 무엇인가?
→ Hue 배열을 사진처럼 직접 보기 어렵기 때문에
```

---

# 7. 빨간색은 HSV에서 한 구간만 보면 안 되는 이유

OpenCV의 Hue 범위는 일반적으로 다음과 같습니다.

```text
0 ~ 179
```

빨간색은 Hue 범위의 양 끝 부분에 걸쳐 나타날 수 있습니다.

따라서 빨간색 Rule은 두 구간을 합쳐 사용하는 경우가 많습니다.

예:

```text
Red Range 1
H: 0 ~ 10

Red Range 2
H: 170 ~ 179
```

S와 V 값은 조명과 물체에 따라 달라질 수 있으므로 처음에는 비교적 넓게 시작합니다.

예:

```text
S >= 100
V >= 70
```

이 값은 절대적인 정답이 아닙니다.

실제 Camera와 조명에서 테스트하여 조정합니다.

---

# 8. 5일차 Rule 설정값 추가하기

오늘의 Rule은 Python 코드 안에 숫자를 직접 고정하지 않고 `configs/settings.yaml`에서 읽습니다.

## 왜 설정파일에 따로 둘까요?

오늘은 실제 조명과 Camera 거리 때문에 HSV 범위와 면적 Threshold를 여러 번 조정할 수 있습니다.

코드 안에 숫자를 직접 쓰면 실험할 때마다 Python 파일을 수정해야 합니다.

```text
Python 코드
→ 판단 방법

settings.yaml
→ 판단에 사용할 실험값
```

이렇게 역할을 나누면 **같은 Rule 코드에 다른 기준값**을 적용하기 쉽습니다.

## 기존 설정은 지우지 않습니다

1~4일차의 설정값은 그대로 두고 5일차 항목을 아래에 추가합니다.

```yaml
# Day 05 - HSV Red Area Rule
red_h1_min: 0
red_h1_max: 10

red_h2_min: 170
red_h2_max: 179

red_s_min: 100
red_v_min: 70

red_area_threshold: 10000

rule_event_log_path: logs/day05_rule_events.csv
warning_image_dir: data/warning_images
```

`use_buzzer`는 3일차부터 사용하던 값을 그대로 사용합니다.

```yaml
use_buzzer: false
```

Buzzer 사양이 확인된 환경에서만 `true`를 사용합니다.

## 각 설정값의 역할

```text
red_h1_min / red_h1_max
→ 빨간색 첫 번째 Hue 구간

red_h2_min / red_h2_max
→ 빨간색 두 번째 Hue 구간

red_s_min
→ 너무 회색에 가까운 Pixel을 제외하기 위한 최소 Saturation

red_v_min
→ 너무 어두운 Pixel을 제외하기 위한 최소 Brightness

red_area_threshold
→ 빨간색으로 판단된 Pixel 수가 몇 개 이상이면 WARNING으로 볼 것인가?

rule_event_log_path
→ 최초 상태와 상태 변화 Event를 저장할 CSV 경로

warning_image_dir
→ WARNING으로 바뀐 순간의 대표 Frame 저장 폴더
```

## 지금은 `10000`을 정답으로 생각하지 않습니다

`red_area_threshold: 10000`은 **첫 실행을 위한 시작값**입니다.

실제 기준은 다음 순서로 정합니다.

```text
NORMAL 장면 측정
        ↓
WARNING 후보 장면 측정
        ↓
분포 또는 범위 비교
        ↓
Threshold 후보 선택
        ↓
실제 Camera에서 재검증
```

해상도를 바꾸면 Pixel 수 자체가 달라지므로 동일한 Threshold를 그대로 사용할 수 없을 수 있습니다.

예를 들어:

```text
640 × 480
→ 총 307,200 Pixel

1280 × 720
→ 총 921,600 Pixel
```

같은 물체도 해상도와 거리 조건이 달라지면 `red_area`가 달라질 수 있습니다.

따라서 **Threshold는 Camera 조건과 함께 기록**합니다.

---

# 9. Rule을 Camera 코드와 분리하기

새 파일:

```text
src/rule_engine.py
```

## 이 파일은 왜 필요할까요?

3일차의 `camera_input.py`는 **Frame을 읽는 역할**입니다.

5일차에는 Frame 안의 빨간색 영역을 분석하여 상태를 판단하는 **새로운 처리·판단 역할**이 필요합니다.

Camera를 여는 코드와 Rule을 한 파일에 섞지 않습니다.

```text
camera_input.py
→ 입력

rule_engine.py
→ 처리 + 판단
```

이렇게 나누면 6~7일차에 Rule 대신 AI Model을 연결하더라도 Camera 입력과 GPIO 출력 구조를 그대로 재사용하기 쉽습니다.

## 의사코드

```text
Frame을 입력받는다
        ↓
BGR을 HSV로 변환한다
        ↓
빨간색 Hue 구간 1의 범위를 만든다
        ↓
빨간색 Hue 구간 2의 범위를 만든다
        ↓
각 범위에 해당하는 Pixel을 Mask로 만든다
        ↓
두 Mask를 OR로 합친다
        ↓
Mask에서 0이 아닌 Pixel 수를 센다
        ↓
red_area를 Threshold와 비교한다
        ↓
Threshold 이상이면 WARNING
미만이면 NORMAL
        ↓
red_area / state / mask를 하나의 결과로 반환한다
```

## 코드:

```python
from dataclasses import dataclass

import cv2
import numpy as np


@dataclass
class RuleResult:
    red_area: int
    state: str
    mask: object


class RedAreaRule:
    def __init__(
        self,
        h1_min: int,
        h1_max: int,
        h2_min: int,
        h2_max: int,
        s_min: int,
        v_min: int,
        area_threshold: int,
    ):
        self.h1_min = h1_min
        self.h1_max = h1_max

        self.h2_min = h2_min
        self.h2_max = h2_max

        self.s_min = s_min
        self.v_min = v_min

        self.area_threshold = area_threshold

    def analyze(self, frame) -> RuleResult:
        hsv = cv2.cvtColor(
            frame,
            cv2.COLOR_BGR2HSV,
        )

        lower1 = np.array(
            [
                self.h1_min,
                self.s_min,
                self.v_min,
            ],
            dtype=np.uint8,
        )

        upper1 = np.array(
            [
                self.h1_max,
                255,
                255,
            ],
            dtype=np.uint8,
        )

        lower2 = np.array(
            [
                self.h2_min,
                self.s_min,
                self.v_min,
            ],
            dtype=np.uint8,
        )

        upper2 = np.array(
            [
                self.h2_max,
                255,
                255,
            ],
            dtype=np.uint8,
        )

        mask1 = cv2.inRange(
            hsv,
            lower1,
            upper1,
        )

        mask2 = cv2.inRange(
            hsv,
            lower2,
            upper2,
        )

        mask = cv2.bitwise_or(
            mask1,
            mask2,
        )

        red_area = int(
            cv2.countNonZero(mask)
        )

        if red_area >= self.area_threshold:
            state = "WARNING"
        else:
            state = "NORMAL"

        return RuleResult(
            red_area=red_area,
            state=state,
            mask=mask,
        )
```

이 모듈은 Camera를 열지 않습니다.

입력으로 이미 만들어진 `frame`을 받아서 다음만 수행합니다.

```text
Frame
→ HSV
→ Red Mask
→ Red Area
→ Threshold
→ NORMAL / WARNING
```

이렇게 Camera와 판단 로직을 분리해 두면 나중에 Rule만 AI로 바꾸기 쉬워집니다.

---

# 10. 저장 이미지 한 장으로 Rule 먼저 테스트하기

실시간 Camera에 바로 연결하지 않습니다.

먼저 4번에서 준비한 고정 이미지 한 장으로 Rule을 반복해서 확인합니다.

새 파일:

```text
scripts/rule_image_test.py
```

## 이 파일은 왜 필요할까요?

실시간 Camera는 장면이 계속 바뀝니다.

Rule 코드를 처음 만들 때는 입력까지 계속 바뀌면 오류 원인을 찾기 어렵습니다.

그래서 다음 순서로 진행합니다.

```text
저장 이미지 1장
→ Rule만 확인
→ Mask 확인
→ Threshold 확인
→ 정상
        ↓
실시간 Camera
```

또한 Python 파일 안의 이미지 경로를 매번 수정하지 않도록 `--image` 인자로 테스트할 파일을 전달합니다.

## 의사코드

```text
실행할 때 --image 경로를 받는다
        ↓
settings.yaml을 읽는다
        ↓
입력 이미지 파일이 존재하는지 확인한다
        ↓
OpenCV로 이미지를 읽는다
        ↓
RedAreaRule을 설정값으로 만든다
        ↓
rule.analyze()를 실행한다
        ↓
red_area / threshold / state를 출력한다
        ↓
Mask 이미지를 저장한다
        ↓
원본 위에 State와 Area를 표시한 Overlay를 저장한다
```

## 코드

```python
import argparse
from pathlib import Path

import cv2

from src.config_loader import load_config
from src.rule_engine import RedAreaRule


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--image",
        required=True,
        help="테스트할 이미지 경로",
    )

    return parser.parse_args()


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
    args = parse_args()

    config = load_config(
        "configs/settings.yaml"
    )

    image_path = Path(args.image)

    if not image_path.exists():
        raise FileNotFoundError(
            f"이미지가 없습니다: {image_path}"
        )

    frame = cv2.imread(
        str(image_path)
    )

    if frame is None:
        raise RuntimeError(
            "이미지를 읽지 못했습니다."
        )

    rule = build_rule(config)

    result = rule.analyze(frame)

    print("=== Rule Image Test ===")
    print(f"Image     : {image_path}")
    print(f"Red Area  : {result.red_area}")
    print(
        f"Threshold : "
        f"{config['red_area_threshold']}"
    )
    print(f"State     : {result.state}")

    output_dir = Path(
        "data/day05_debug"
    )

    output_dir.mkdir(
        parents=True,
        exist_ok=True,
    )

    stem = image_path.stem

    mask_path = (
        output_dir
        / f"{stem}_red_mask.jpg"
    )

    overlay_path = (
        output_dir
        / f"{stem}_rule_overlay.jpg"
    )

    cv2.imwrite(
        str(mask_path),
        result.mask,
    )

    overlay = frame.copy()

    cv2.putText(
        overlay,
        (
            f"{result.state} "
            f"area={result.red_area}"
        ),
        (20, 40),
        cv2.FONT_HERSHEY_SIMPLEX,
        1.0,
        (0, 0, 255)
        if result.state == "WARNING"
        else (0, 255, 0),
        2,
    )

    cv2.imwrite(
        str(overlay_path),
        overlay,
    )

    print("Debug images saved:")
    print(mask_path)
    print(overlay_path)


if __name__ == "__main__":
    main()
```

---

# 11. Guided Lab — WARNING 후보 이미지를 먼저 실행하기

빨간색 카드가 있는 이미지를 사용합니다.

```bash
python -m scripts.rule_image_test \
  --image data/captures/day05_red.jpg
```

예상 형태:

```text
=== Rule Image Test ===
Image     : data/captures/day05_red.jpg
Red Area  : 18432
Threshold : 10000
State     : WARNING
```

숫자는 실제 이미지에 따라 달라집니다.

중요한 것은 다음 관계입니다.

```text
Red Area >= Threshold
→ WARNING
```

Debug 파일을 확인합니다.

```bash
find data/day05_debug \
  -maxdepth 1 \
  -type f | sort
```

`day05_red_red_mask.jpg`에서 **흰색으로 보이는 Pixel**이 현재 Rule이 빨간색이라고 판단한 영역입니다.

원본 카드와 Mask가 비슷한 위치를 잡고 있는지 확인합니다.

---

## 코드 리뷰 — `State : WARNING`은 어디에서 만들어졌을까요?

다음 한 줄이 출력되었다고 가정합니다.

```text
State : WARNING
```

파일을 다시 열어 아래 흐름을 직접 찾습니다.

```text
settings.yaml
red_area_threshold
        ↓
rule_image_test.py
build_rule()
        ↓
rule_engine.py
RedAreaRule.analyze()
        ↓
cv2.cvtColor()
        ↓
cv2.inRange()
        ↓
cv2.bitwise_or()
        ↓
cv2.countNonZero()
        ↓
red_area
        ↓
if red_area >= area_threshold
        ↓
WARNING
        ↓
rule_image_test.py
print()
```

다음 질문에 답합니다.

```text
실제 빨간색 Pixel 수를 세는 함수는 무엇인가?
Threshold는 어느 파일의 값인가?
NORMAL / WARNING을 최종 결정하는 코드는 어느 파일인가?
Mask를 JPG로 저장하는 코드는 어느 파일인가?
```

---

# 12. NORMAL 이미지도 같은 Rule로 확인하기

이번에는 Python 코드를 수정하지 않고 `--image` 값만 바꿉니다.

```bash
python -m scripts.rule_image_test \
  --image data/captures/day05_normal.jpg
```

예상 형태:

```text
=== Rule Image Test ===
Image     : data/captures/day05_normal.jpg
Red Area  : 523
Threshold : 10000
State     : NORMAL
```

같은 Rule에서 다음 두 장면이 구분되어야 합니다.

```text
빨간 카드 없음
→ 작은 red_area
→ NORMAL

빨간 카드 크게 있음
→ 큰 red_area
→ WARNING
```

두 Debug Mask도 함께 비교합니다.

```text
day05_normal_red_mask.jpg
day05_red_red_mask.jpg
```

### 여기서 잠깐 확인합니다

Mask가 실제 빨간색 카드와 전혀 다른 영역을 잡는다면 Area Threshold부터 조정하지 않습니다.

먼저 다음을 확인합니다.

```text
HSV 범위
→ red_h1_min / max
→ red_h2_min / max
→ red_s_min
→ red_v_min
```

반대로 Mask는 대체로 맞는데 NORMAL/WARNING 경계만 원하는 것과 다르다면:

```text
red_area_threshold
```

을 조정합니다.

즉, 다음 두 문제를 구분합니다.

```text
어떤 Pixel이 빨간색인가?
→ HSV 범위 문제

얼마나 많은 빨간 Pixel이 있어야 WARNING인가?
→ Area Threshold 문제
```

---

# 13. Camera Frame에서 빨간색 면적을 계속 출력하기

실시간 Camera에서 `red_area`가 어떻게 변하는지 먼저 확인합니다.

파일:

```text
scripts/rule_camera_probe.py
```

## 이 파일은 왜 필요할까요?

저장 이미지에서는 Rule이 동작했습니다.

이제 같은 Rule을 실제 Camera의 연속 Frame에 적용하여 **장면 변화에 따라 `red_area`가 어떻게 움직이는지** 확인합니다.

아직 GPIO나 Event Log를 연결하지 않습니다.

```text
Camera
→ Rule
→ red_area / state 출력
```

만 먼저 확인합니다.

기능을 한 단계씩 연결하면 문제가 생겼을 때 원인을 좁히기 쉽습니다.

## 의사코드

```text
설정을 읽는다
        ↓
Camera를 연다
        ↓
RedAreaRule을 만든다
        ↓
반복한다
        ↓
Frame을 한 장 읽는다
        ↓
Rule로 분석한다
        ↓
red_area와 state를 출력한다
        ↓
짧게 기다린 뒤 다음 Frame으로 이동한다
        ↓
Ctrl+C가 들어오면 반복을 끝낸다
        ↓
Camera를 닫는다
```

## 코드:

```python
import time

from src.camera_input import CameraInput
from src.config_loader import load_config
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

    camera = CameraInput(
        index=int(config["camera_index"]),
        width=int(config["image_width"]),
        height=int(config["image_height"]),
    )

    rule = build_rule(config)

    try:
        print("=== Rule Camera Probe ===")
        print("종료: Ctrl+C")
        print()

        time.sleep(1.0)

        while True:
            frame = camera.read()

            result = rule.analyze(frame)

            print(
                f"red_area="
                f"{result.red_area:8d} | "
                f"{result.state}"
            )

            time.sleep(0.1)

    except KeyboardInterrupt:
        print("\n종료합니다.")

    finally:
        camera.release()


if __name__ == "__main__":
    main()
```

실행:

```bash
python -m scripts.rule_camera_probe
```

---

# 14. 빨간색 카드를 움직이며 값 관찰하기

실행 중 다음 순서로 직접 움직입니다.

```text
1. 빨간색 카드 없음
2. 화면 가장자리에 조금 보이기
3. 화면 중앙에 작게 보이기
4. Camera 가까이 가져오기
5. 화면 대부분을 채우기
```

터미널의 `red_area` 값이 어떻게 달라지는지 봅니다.

예:

```text
카드 없음
red_area=     410

작게 보임
red_area=    3820

중간
red_area=    9140

크게 보임
red_area=   24180
```

Threshold가 `10000`이면:

```text
9140
→ NORMAL

24180
→ WARNING
```

이제 `10000`이라는 숫자가 단순한 예제가 아니라 **실제 Camera에서 관찰한 값과 비교해야 하는 기준값**이라는 것을 알 수 있습니다.

---

## 코드 리뷰 — 실시간 `red_area` 한 줄을 역추적하기

다음 결과가 보였다고 가정합니다.

```text
red_area=   24180 | WARNING
```

화면 결과만 보지 않고 파일을 다시 열어 다음을 찾습니다.

| 결과 | 파일 / 코드 |
|---|---|
| Camera index | `configs/device.local.yaml` |
| Frame 읽기 | `src/camera_input.py` → `read()` |
| HSV/Mask 계산 | `src/rule_engine.py` → `analyze()` |
| `24180` | `cv2.countNonZero(mask)` |
| `WARNING` | `red_area >= area_threshold` |
| 한 줄 출력 | `scripts/rule_camera_probe.py` → `print()` |
| 반복 실행 | `while True` |

### 3분 복습 — 코드를 닫고 말해보기

```text
Camera Frame은 _____________________ 파일이 읽는다.

빨간색 Mask는 ______________________ 파일이 만든다.

red_area는 __________________________ 함수로 계산한다.

Threshold는 _________________________ 파일에서 읽는다.

실시간 반복의 시작점은 ______________ 파일이다.
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
Camera Frame
→ src/camera_input.py

빨간색 Mask
→ src/rule_engine.py

red_area
→ cv2.countNonZero()

Threshold
→ configs/settings.yaml

실시간 반복 시작점
→ scripts/rule_camera_probe.py
```

</details>

---

# 15. Threshold를 정할 때 먼저 데이터를 봅니다

처음부터 감으로 Threshold를 정하지 않습니다.

간단하게 다음 값을 기록합니다.

```text
NORMAL 1:
NORMAL 2:
NORMAL 3:

WARNING 후보 1:
WARNING 후보 2:
WARNING 후보 3:
```

예:

```text
NORMAL
420
610
720

WARNING 후보
13200
18600
25400
```

이 경우 다음 사이에서 기준을 정할 수 있습니다.

```text
NORMAL 최대값
720

WARNING 최소값
13200
```

예:

```text
10000
```

현장 조건이 바뀌면 이 값도 달라질 수 있습니다.

---

# 16. Modify Lab 1 — 면적 Threshold 비교하기

다음 세 값을 비교합니다.

```yaml
red_area_threshold: 5000
```

```yaml
red_area_threshold: 10000
```

```yaml
red_area_threshold: 20000
```

각 설정에서:

```bash
python -m scripts.rule_camera_probe
```

를 실행합니다.

같은 거리와 같은 카드 크기를 유지해 비교합니다.

Report에 다음 표를 작성합니다.

```md
## Area Threshold Experiment

| Threshold | 카드 없음 | 카드 작게 | 카드 크게 | 관찰 |
|---:|---|---|---|---|
| 5000 |  |  |  |  |
| 10000 |  |  |  |  |
| 20000 |  |  |  |  |
```

한 번에 Threshold만 바꿉니다.

---

# 17. 4일차 Event Logger를 Vision Rule까지 확장하기

4일차에서 이미 다음 파일을 만들었습니다.

```text
src/event_logger.py
```

이 파일에는 Sensor 상태를 기록하는 `EventCSVLogger`가 있습니다.

오늘은 **새 파일을 만들지 않고**, 기존 `event_logger.py` 아래에 Vision Rule용 Class를 추가합니다.

## 왜 기존 파일을 다시 사용할까요?

4일차와 5일차는 입력 형태는 다르지만 둘 다 **운영 상태 변화 기록**이라는 같은 목적을 가지고 있습니다.

```text
4일차
Sensor Value
→ Threshold
→ State
→ Event CSV

5일차
Red Area
→ Threshold
→ State
→ Event CSV
```

다만 5일차에는 추가로 다음 정보가 필요합니다.

```text
측정값 이름
→ red_area

기준값
→ threshold

WARNING 대표 이미지
→ image_path
```

그래서 기존 Sensor Logger는 그대로 두고 새로운 `VisionRuleEventLogger` Class만 추가합니다.

## 의사코드

```text
CSV 경로를 받는다
        ↓
부모 폴더가 없으면 만든다
        ↓
append()가 호출된다
        ↓
새 파일인지 확인한다
        ↓
새 파일이면 Header를 기록한다
        ↓
Timestamp
Mode
Metric Name
Metric Value
Threshold
State
Image Path
를 한 줄로 저장한다
```

## `src/event_logger.py` 아래에 추가할 코드

> `csv`와 `Path`는 4일차 파일의 위쪽에서 이미 Import되어 있습니다. 같은 Import를 다시 작성하지 않아도 됩니다.

```python
class VisionRuleEventLogger:
    def __init__(self, path: str):
        self.path = Path(path)

        self.path.parent.mkdir(
            parents=True,
            exist_ok=True,
        )

    def append(
        self,
        timestamp: str,
        mode: str,
        metric_name: str,
        metric_value: float,
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
                        "metric_name",
                        "metric_value",
                        "threshold",
                        "state",
                        "image_path",
                    ]
                )

            writer.writerow(
                [
                    timestamp,
                    mode,
                    metric_name,
                    f"{metric_value:.3f}",
                    f"{threshold:.3f}",
                    state,
                    image_path,
                ]
            )
```

이 Logger는 다음을 기록합니다.

```text
언제?
→ timestamp

어떤 판단 방식?
→ mode=rule

무엇을 측정?
→ metric_name=red_area

실제 측정값?
→ metric_value

어떤 기준과 비교?
→ threshold

운영 결과?
→ NORMAL / WARNING

WARNING 대표 이미지?
→ image_path
```

`VisionRuleEventLogger` 자체가 상태 변화를 판단하는 것은 아닙니다.

```text
상태 변화 판단
→ 실행 Program

CSV에 기록
→ Logger
```

역할을 구분합니다.

---

# 18. WARNING 대표 이미지를 저장하는 함수 추가하기

`src/rule_engine.py`를 다시 사용합니다.

새 파일을 만들지 않고 **WARNING으로 바뀐 순간의 Frame을 저장하는 기능**을 같은 Rule 관련 모듈에 추가합니다.

## 왜 모든 WARNING Frame을 저장하지 않을까요?

Camera는 초당 여러 Frame이 들어옵니다.

WARNING이 10초 동안 유지되는 동안 매 Frame을 저장하면 파일이 지나치게 많아질 수 있습니다.

오늘은 다음 방식으로 시작합니다.

```text
NORMAL → WARNING
→ 대표 Frame 1장 저장

WARNING → WARNING
→ 추가 저장하지 않음

WARNING → NORMAL
→ 이미지 저장하지 않음
```

즉, **상태가 WARNING으로 전환된 순간의 증거 이미지**만 남깁니다.

## `src/rule_engine.py` 상단 Import에 추가

```python
from datetime import datetime
from pathlib import Path
```

기존 `cv2`, `numpy` Import는 그대로 유지합니다.

## 의사코드

```text
Frame과 저장 폴더, red_area를 받는다
        ↓
저장 폴더가 없으면 만든다
        ↓
현재 시각으로 파일명용 Timestamp를 만든다
        ↓
파일명에 Timestamp와 red_area를 넣는다
        ↓
cv2.imwrite()로 JPG를 저장한다
        ↓
저장 실패면 오류를 발생시킨다
        ↓
성공하면 저장 경로를 문자열로 반환한다
```

## `src/rule_engine.py` 아래에 추가할 코드

```python
def save_warning_image(
    frame,
    output_dir: str,
    red_area: int,
) -> str:
    directory = Path(output_dir)

    directory.mkdir(
        parents=True,
        exist_ok=True,
    )

    timestamp = datetime.now().strftime(
        "%Y%m%d_%H%M%S_%f"
    )

    path = directory / (
        f"warning_{timestamp}"
        f"_area{red_area}.jpg"
    )

    ok = cv2.imwrite(
        str(path),
        frame,
    )

    if not ok:
        raise RuntimeError(
            f"WARNING 이미지 저장 실패: {path}"
        )

    return str(path)
```

WARNING 이미지 파일명에는 다음 두 정보가 들어갑니다.

```text
발생 시각
+
red_area
```

예:

```text
warning_20261007_141530_123456_area18420.jpg
```

파일명만 보아도 **언제 발생했고 당시 면적이 어느 정도였는지** 일부 정보를 확인할 수 있습니다.

---

# 19. 상태 변화만 기록해야 하는 이유

Camera는 초당 여러 Frame이 들어옵니다.

WARNING 상태가 10초 동안 유지된다고 해서 모든 Frame마다 Event를 기록하면 다음처럼 됩니다.

```text
WARNING
WARNING
WARNING
WARNING
WARNING
...
```

Event Log가 지나치게 많아집니다.

오늘은 우선 다음처럼 **상태가 바뀌는 순간**을 기록합니다.

```text
NORMAL
→ WARNING
기록

WARNING
→ WARNING
기록하지 않음

WARNING
→ NORMAL
기록
```

이 방식은 4일차 Sensor Event에서 사용한 방식과 같습니다.

---

# 20. Rule 기반 Edge Warning System v1 만들기

파일:

```text
scripts/rule_warning_system.py
```

## 이 파일은 왜 필요할까요?

지금까지는 기능을 일부러 나누어 확인했습니다.

```text
Camera 입력
→ camera_input.py

Rule 판단
→ rule_engine.py

GPIO 출력
→ gpio_output.py

Event 기록
→ event_logger.py
```

이제 `rule_warning_system.py`가 이 모듈들을 순서대로 연결하는 **메인 실행 시작점**이 됩니다.

새 기능을 다시 작성하는 것이 아니라 이미 검증한 기능을 조합합니다.

## 실행 전에 전체 파일 연결 보기

```text
configs/settings.yaml
configs/device.local.yaml
        ↓
src/config_loader.py
        ↓
scripts/rule_warning_system.py
        │
        ├─ src/camera_input.py
        │      ↓
        │    Frame
        │
        ├─ src/rule_engine.py
        │      ↓
        │    red_area / state / mask
        │
        ├─ src/gpio_output.py
        │      ↓
        │    Green / Red / 선택 Buzzer
        │
        └─ src/event_logger.py
               ↓
            Event CSV
               +
        WARNING 전환이면
        save_warning_image()
```

## 의사코드

```text
설정파일을 읽는다
        ↓
Rule 객체를 만든다
        ↓
GPIOOutput을 만든다
        ↓
Vision Rule Event Logger를 만든다
        ↓
previous_state = None으로 시작한다
        ↓
Camera를 연다
        ↓
반복한다
        ↓
Frame을 읽는다
        ↓
Rule로 red_area와 state를 계산한다
        ↓
WARNING이면 경고 출력
NORMAL이면 정상 출력
        ↓
현재 state와 previous_state를 비교한다
        ↓
상태가 처음이거나 바뀌었다면
        ↓
Timestamp를 만든다
        ↓
WARNING 전환이면 대표 이미지를 저장한다
        ↓
Event CSV에 기록한다
        ↓
previous_state를 현재 state로 바꾼다
        ↓
터미널에 red_area / state / changed를 출력한다
        ↓
Ctrl+C 또는 오류가 발생하면
Camera와 GPIO Resource를 정리한다
```

## 코드:

```python
import time
from datetime import datetime

from src.camera_input import CameraInput
from src.config_loader import load_config
from src.event_logger import VisionRuleEventLogger
from src.gpio_output import GPIOOutput
from src.rule_engine import (
    RedAreaRule,
    save_warning_image,
)


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

    rule = build_rule(config)

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
            config.get("use_buzzer", False)
        ),
    )

    logger = VisionRuleEventLogger(
        config["rule_event_log_path"]
    )

    previous_state = None
    camera = None

    try:
        camera = CameraInput(
            index=int(config["camera_index"]),
            width=int(config["image_width"]),
            height=int(config["image_height"]),
        )

        print(
            "=== Rule Edge Warning System v1 ==="
        )
        print("종료: Ctrl+C")
        print()

        time.sleep(1.0)

        while True:
            frame = camera.read()

            result = rule.analyze(frame)

            if result.state == "WARNING":
                output.warning()
            else:
                output.normal()

            changed = (
                result.state != previous_state
            )

            image_path = ""

            if changed:
                timestamp = (
                    datetime.now().isoformat(
                        timespec="milliseconds"
                    )
                )

                if result.state == "WARNING":
                    image_path = save_warning_image(
                        frame,
                        config[
                            "warning_image_dir"
                        ],
                        result.red_area,
                    )

                logger.append(
                    timestamp=timestamp,
                    mode="rule",
                    metric_name="red_area",
                    metric_value=result.red_area,
                    threshold=float(
                        config[
                            "red_area_threshold"
                        ]
                    ),
                    state=result.state,
                    image_path=image_path,
                )

                previous_state = result.state

            print(
                f"red_area="
                f"{result.red_area:8d} | "
                f"{result.state:7s} | "
                f"changed={changed}"
            )

            # 이 값은 화면 갱신 간격을 위한 작은 대기입니다.
            # 실제 Pipeline FPS는 8일차에서 별도로 측정합니다.
            time.sleep(0.05)

    except KeyboardInterrupt:
        print("\n종료합니다.")

    except Exception as error:
        print(f"실행 오류: {error}")
        output.warning()
        time.sleep(1.0)
        raise

    finally:
        if camera is not None:
            camera.release()

        output.close()


if __name__ == "__main__":
    main()
```

---

# 21. Guided Lab — 첫 번째 완성형 Rule 시스템 실행하기

실행:

```bash
python -m scripts.rule_warning_system
```

빨간색 카드가 없을 때:

```text
NORMAL
→ Green LED
```

빨간색 카드를 충분히 크게 보여주면:

```text
red_area >= threshold
→ WARNING
→ Red LED
→ 선택 Buzzer (`use_buzzer: true`일 때)
→ WARNING Image 저장
→ Event CSV 기록
```

카드를 다시 치우면:

```text
WARNING
→ NORMAL
→ Event CSV 기록
→ Green LED
```

종료:

```text
Ctrl+C
```

---

# 22. Event Log 확인하기

```bash
cat logs/day05_rule_events.csv
```

예:

```csv
timestamp,mode,metric_name,metric_value,threshold,state,image_path
2026-10-07T14:15:20.123,rule,red_area,420.000,10000.000,NORMAL,
2026-10-07T14:15:23.451,rule,red_area,18420.000,10000.000,WARNING,data/warning_images/warning_...
2026-10-07T14:15:27.104,rule,red_area,810.000,10000.000,NORMAL,
```

다음 흐름을 읽을 수 있어야 합니다.

```text
14:15:20
NORMAL

14:15:23
WARNING 발생
+
이미지 저장

14:15:27
NORMAL 복귀
```

---

# 23. WARNING 이미지 확인하기

```bash
find data/warning_images -type f
```

파일 수:

```bash
find data/warning_images -type f | wc -l
```

필요한 이미지를 PC로 가져와 확인할 수 있습니다.

PC에서:

```bash
scp \
<RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/data/warning_images/*.jpg \
.
```

모든 WARNING Frame을 계속 저장하는 것이 아니라 **상태가 WARNING으로 바뀐 순간의 대표 이미지**를 남기도록 했습니다.

---

## 코드 리뷰 — `WARNING` 한 번이 실제로 어디까지 연결될까요?

다음 결과가 보였다고 가정합니다.

```text
red_area=   18420 | WARNING | changed=True
```

동시에 Red LED가 켜지고, Buzzer를 사용하는 장비라면 Buzzer가 동작하며, Event CSV와 WARNING Image가 만들어졌다고 가정합니다.

이 한 번의 결과를 코드에서 역추적합니다.

```text
Camera Frame
→ camera_input.py
→ CameraInput.read()

red_area / WARNING
→ rule_engine.py
→ RedAreaRule.analyze()

Red LED / 선택 Buzzer
→ gpio_output.py
→ output.warning()

changed=True
→ rule_warning_system.py
→ state != previous_state

WARNING 이미지
→ rule_engine.py
→ save_warning_image()

Event CSV
→ event_logger.py
→ VisionRuleEventLogger.append()
```

다음 표를 직접 채워 봅니다.

| 실행 결과 | 담당 파일 | 담당 함수/코드 |
|---|---|---|
| Camera Frame |  |  |
| `red_area=18420` |  |  |
| `WARNING` |  |  |
| Red LED |  |  |
| `changed=True` |  |  |
| WARNING JPG |  |  |
| Event CSV 한 줄 |  |  |

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

| 실행 결과 | 담당 파일 | 담당 함수/코드 |
|---|---|---|
| Camera Frame | `src/camera_input.py` | `CameraInput.read()` |
| `red_area=18420` | `src/rule_engine.py` | `cv2.countNonZero(mask)` |
| `WARNING` | `src/rule_engine.py` | `red_area >= area_threshold` |
| Red LED | `src/gpio_output.py` | `GPIOOutput.warning()` |
| `changed=True` | `scripts/rule_warning_system.py` | `state != previous_state` |
| WARNING JPG | `src/rule_engine.py` | `save_warning_image()` |
| Event CSV 한 줄 | `src/event_logger.py` | `VisionRuleEventLogger.append()` |

한 번의 화면 결과가 한 파일에서 전부 만들어지는 것이 아니라 여러 모듈이 역할을 나누어 만든다는 점이 핵심입니다.

</details>

---

# 24. 지금 완성된 전체 Pipeline 확인하기

```text
USB Camera
     ↓
CameraInput
     ↓
OpenCV Frame
     ↓
BGR → HSV
     ↓
Red Mask
     ↓
Red Area
     ↓
Area Threshold
     ↓
NORMAL / WARNING
     │
     ├─ GPIO
     │   ├─ Green LED
     │   ├─ Red LED
     │   └─ Buzzer
     │
     └─ Event Log
         └─ WARNING Image
```

이것이 교과 13의 첫 번째 **End-to-End Edge Warning System**입니다.

아직 AI 모델은 없습니다.

---

# 25. Rule과 운영 상태를 구분해서 보기

현재 판단은 두 단계로 볼 수 있습니다.

```text
측정값
red_area = 18420

        ↓

운영 기준
threshold = 10000

        ↓

운영 상태
WARNING
```

중요한 것은 다음입니다.

```text
red_area
≠ WARNING 그 자체

red_area를
Threshold와 비교한 결과
→ WARNING
```

7일차의 AI에서도 같은 구조가 등장합니다.

```text
AI Prediction
Class + Confidence

        ↓

Operational Threshold

        ↓

NORMAL / WARNING
```

---

# 26. Modify Lab 2 — 빨간색에서 파란색으로 바꾸기

이번에는 판단 대상 색상만 바꿉니다.

기존 Red Rule을 그대로 수정해도 되지만, 기존 정상 Version을 보존하기 위해 별도의 `BlueAreaRule`을 만들어 볼 수 있습니다.

OpenCV HSV에서 파란색은 환경에 따라 대략 다음 범위에서 시작해 볼 수 있습니다.

```text
H: 100 ~ 130
S: 100 ~ 255
V: 70 ~ 255
```

이 값도 정답이 아닙니다.

실제 Camera 환경에서 조정합니다.

다음 질문을 기준으로 직접 변경합니다.

```text
어떤 Hue 범위를 사용할 것인가?
S 최소값은?
V 최소값은?
Area Threshold는 같은 값을 사용해도 되는가?
```

파란색 카드로 다음이 되는지 확인합니다.

```text
Blue Card 없음
→ NORMAL

Blue Card 크게 있음
→ WARNING
```

---

# 27. Modify Lab 3 — 중앙 ROI만 검사하기

화면 전체가 아니라 중앙 영역만 검사할 수 있습니다.

예:

```text
전체 Frame
640 × 480

중앙 ROI
가로 25% ~ 75%
세로 25% ~ 75%
```

`rule_engine.py`의 `analyze()`에서 Frame을 자른 뒤 HSV 변환을 적용해 볼 수 있습니다.

이 실습은 선택 확장입니다. 기본 Rule Baseline이 정상인 경우에만 진행합니다.

## 의사코드

```text
Frame의 높이와 너비를 구한다
        ↓
중앙 50% 영역의 x1 / x2 / y1 / y2를 계산한다
        ↓
Frame에서 ROI를 잘라낸다
        ↓
전체 Frame 대신 ROI를 HSV로 변환한다
        ↓
ROI 안의 Red Area를 계산한다
        ↓
같은 Threshold로 결과를 확인한다
        ↓
전체 Frame 방식과 차이를 비교한다
```

힌트:

```python
height, width = frame.shape[:2]

x1 = int(width * 0.25)
x2 = int(width * 0.75)

y1 = int(height * 0.25)
y2 = int(height * 0.75)

roi = frame[y1:y2, x1:x2]
```

그다음:

```text
frame
대신
roi
```

를 분석합니다.

같은 빨간색 카드가 화면 가장자리에 있을 때와 중앙에 있을 때 결과가 어떻게 달라지는지 확인합니다.

---

# 28. Modify Lab 4 — 밝기 조건으로 바꾸어 보기

이번에는 색상 대신 밝기만으로 WARNING을 만들 수 있습니다.

예:

```text
화면 평균 밝기 < 특정 값
→ WARNING
```

이 실습도 선택 확장입니다. 색상 Rule과 다른 판단값을 만들어도 Camera/GPIO/Log 구조를 재사용할 수 있는지 확인합니다.

## 의사코드

```text
Frame을 입력받는다
        ↓
BGR Frame을 Grayscale로 변환한다
        ↓
전체 Pixel의 평균 밝기를 계산한다
        ↓
평균 밝기가 기준보다 낮은지 비교한다
        ↓
낮으면 WARNING
그렇지 않으면 NORMAL
        ↓
기존 GPIO / Event Log 구조에 연결할 수 있음을 확인한다
```

힌트:

```python
gray = cv2.cvtColor(
    frame,
    cv2.COLOR_BGR2GRAY,
)

mean_brightness = gray.mean()
```

예:

```text
mean_brightness < 60
→ 너무 어두움
→ WARNING
```

이 실습의 목적은 밝기 Rule이 우수하다는 것을 보이는 것이 아닙니다.

```text
판단 모듈만 바꾸어도
나머지 Camera / GPIO / Log 구조는
그대로 재사용할 수 있다.
```

는 것을 확인하는 것입니다.

---

# 29. WARNING이 너무 짧게 깜빡이는 문제 보기

Threshold 근처에서 `red_area`가 다음처럼 움직일 수 있습니다.

```text
9800
10200
9900
10150
9700
```

Threshold:

```text
10000
```

결과:

```text
NORMAL
WARNING
NORMAL
WARNING
NORMAL
```

LED와 Buzzer가 계속 흔들릴 수 있습니다.

오늘은 이 현상을 **Rule의 경계 흔들림**으로 관찰만 합니다.

9일차에서는 Voting, Hold, Debounce 같은 방법으로 본격적으로 안정화합니다.

5일차에는 간단한 Hold 정도만 선택적으로 실험합니다.

---

# 30. Preview Lab — 경계 흔들림을 관찰하고 9일차 과제로 남기기

Threshold 근처에서 `red_area`가 오르내리면 상태가 빠르게 바뀔 수 있습니다.

```text
9800
→ NORMAL

10200
→ WARNING

9900
→ NORMAL

10150
→ WARNING
```

오늘은 이 현상을 **직접 관찰하고 기록만** 합니다.

아직 Hold, Voting, Debounce 같은 안정화 로직을 핵심 Program에 넣지 않습니다.

이유는 다음과 같습니다.

```text
5일차 목표
→ Rule이 어떻게 판단되는지 명확하게 이해

9일차 목표
→ 흔들리는 Edge AI / Rule 결과를 안정화
```

현재의 단순한 Baseline을 먼저 보존해야 나중에 안정화 전·후를 비교할 수 있습니다.

Report에 다음만 기록합니다.

```text
Threshold 근처에서 상태가 흔들렸는가?:

어떤 장면에서 흔들렸는가?:

추후 안정화가 필요한 이유:
```

---

# 31. 일부러 실패시키기 1 — 조명을 어둡게 만들기

같은 빨간색 카드를 같은 위치에 둡니다.

조명만 바꿉니다.

```text
밝은 환경
→ 실행

어두운 환경
→ 실행
```

다른 조건은 바꾸지 않습니다.

확인:

```text
red_area가 어떻게 달라지는가?
WARNING 결과가 유지되는가?
```

HSV의 `V` 조건 때문에 어두운 빨간색 영역이 Mask에서 빠질 수 있습니다.

Report에 기록합니다.

```text
변경 변수
→ 조명

고정 변수
→ 카드
→ 거리
→ Threshold
→ Camera 위치
```

---

# 32. 일부러 실패시키기 2 — 카드를 멀리 이동하기

조명은 그대로 둡니다.

거리만 바꿉니다.

```text
가까이
→ red_area 큼

멀리
→ red_area 작음
```

같은 빨간색 카드인데도 면적 Threshold 방식에서는 거리에 따라 결과가 바뀔 수 있습니다.

예:

```text
가까이
18420
→ WARNING

멀리
6200
→ NORMAL
```

이것은 코드 오류가 아닙니다.

현재 Rule의 특성입니다.

---

# 33. 일부러 실패시키기 3 — 빨간색 배경 사용하기

빨간색 카드가 없어도 배경에 큰 빨간색 영역이 있으면:

```text
Red Mask
→ 큰 면적

Threshold 통과
→ WARNING
```

이 경우 시스템은 잘못 경고할 수 있습니다.

분류:

```text
Camera 고장
X

GPIO 고장
X

Rule 설계 한계
O
```

Rule은 사람이 정한 단순한 기준이기 때문에 환경 변화에 민감할 수 있습니다.

---

# 34. 일부러 실패시키기 4 — 설정값을 지나치게 낮추기

예:

```yaml
red_area_threshold: 100
```

실행:

```bash
python -m scripts.rule_warning_system
```

작은 Noise나 배경의 붉은 영역만으로도 WARNING이 많이 발생할 수 있습니다.

다음과 같이 볼 수 있습니다.

```text
Threshold 너무 낮음
→ 민감함
→ False Warning 증가 가능

Threshold 너무 높음
→ 둔감함
→ 실제 대상 놓칠 가능성
```

---

# 35. 실패 사례를 이미지로 남기기

다음 최소 세 종류의 실패 사례를 확보합니다.

```text
1. 어두운 조명

2. 대상이 너무 멀리 있음

3. 빨간색 배경
```

각 사례마다 다음을 기록합니다.

```text
조건:
예상:
실제:
red_area:
state:
왜 실패했다고 생각하는가?
무엇을 바꾸면 개선될까?
```

가능하면 해당 Frame도 저장합니다.

예:

```text
data/day05_failures/
├── dark.jpg
├── far.jpg
└── red_background.jpg
```

이 폴더는 실행 데이터이므로 Git에는 올리지 않습니다.

---

# 36. Rule의 한계를 코드 오류와 구분하기

다음 상황을 구분합니다.

## 코드 오류

```text
Camera Frame을 읽지 못함
파일 경로 오류
Import 오류
잘못된 Config Key
```

프로그램이 정상적으로 실행되지 않습니다.

## Rule 한계

```text
프로그램은 정상 실행
하지만 조명 변화에서 오판정
거리 변화에서 오판정
배경색 때문에 오판정
```

프로그램은 정상 동작하지만 **판단 기준이 현실 조건을 충분히 표현하지 못한 것**입니다.

이 차이를 구분해야 다음 개선 방향을 정할 수 있습니다.

---

# 37. Mini Challenge — Rule Edge Warning System을 스스로 다시 이해하기

지금까지는 안내된 순서대로 Rule 시스템을 만들었습니다.

이제부터는 단순히 코드를 다시 입력하는 것이 아니라 다음 순서로 문제를 해결합니다.

```text
실행 결과 보기
        ↓
어느 파일에서 만들어졌는지 찾기
        ↓
실행 전에 결과 예상하기
        ↓
설정값만 바꾸어 비교하기
        ↓
기존 기능 일부 수정하기
        ↓
일부러 오류를 만들고 원인 찾기
        ↓
Log / 이미지로 정상 동작 증명하기
        ↓
전체 Pipeline을 자신의 말로 설명하기
```

각 Challenge는 먼저 직접 생각하고 실행합니다.

그 다음에만 **예시 정답 확인해보기**를 열어 자신의 결과와 비교합니다.

---

## Challenge 1 — 실행 결과를 보고 코드 위치 찾기

다음 결과가 터미널에 출력되었다고 가정합니다.

```text
red_area=   18420 | WARNING | changed=True
```

그리고 동시에 다음 파일이 생겼다고 가정합니다.

```text
logs/day05_rule_events.csv

data/warning_images/
warning_..._area18420.jpg
```

다음 표를 직접 채웁니다.

| 결과 | 담당 파일 | 담당 코드 또는 함수 |
|---|---|---|
| Camera Frame |  |  |
| `red_area=18420` |  |  |
| `WARNING` |  |  |
| `changed=True` |  |  |
| Red LED |  |  |
| Event CSV |  |  |
| WARNING JPG |  |  |

바로 정답을 보지 말고 실제 파일을 열어 찾습니다.

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

| 결과 | 담당 파일 | 담당 코드 또는 함수 |
|---|---|---|
| Camera Frame | `src/camera_input.py` | `CameraInput.read()` |
| `red_area=18420` | `src/rule_engine.py` | `cv2.countNonZero(mask)` |
| `WARNING` | `src/rule_engine.py` | `red_area >= area_threshold` |
| `changed=True` | `scripts/rule_warning_system.py` | `state != previous_state` |
| Red LED | `src/gpio_output.py` | `GPIOOutput.warning()` |
| Event CSV | `src/event_logger.py` | `VisionRuleEventLogger.append()` |
| WARNING JPG | `src/rule_engine.py` | `save_warning_image()` |

핵심은 한 줄의 결과가 한 파일에서 모두 만들어지는 것이 아니라는 점입니다.

```text
Camera 입력
+
Rule 계산
+
운영 상태 판단
+
GPIO 출력
+
Event 기록
```

이 역할이 서로 다른 파일로 나뉘어 있습니다.

</details>

---

## Challenge 2 — 실행하기 전에 결과를 먼저 예상하기

현재 설정이 다음과 같다고 가정합니다.

```yaml
red_area_threshold: 10000
```

다음 `red_area`에 대해 프로그램을 실행하지 않고 먼저 상태를 예상합니다.

| Red Area | 예상 State |
|---:|---|
| 420 | |
| 9,999 | |
| 10,000 | |
| 10,001 | |
| 24,180 | |

특히 `10,000`이 어떤 상태가 되는지 `rule_engine.py`의 비교 연산자를 확인합니다.

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

현재 판단 코드는 다음과 같습니다.

```python
if red_area >= self.area_threshold:
    state = "WARNING"
else:
    state = "NORMAL"
```

따라서:

| Red Area | State | 이유 |
|---:|---|---|
| 420 | NORMAL | 10,000 미만 |
| 9,999 | NORMAL | 10,000 미만 |
| 10,000 | WARNING | `>=`이므로 같은 값도 포함 |
| 10,001 | WARNING | 10,000보다 큼 |
| 24,180 | WARNING | 10,000보다 큼 |

`>`와 `>=`는 경계값에서 결과가 다릅니다.

```text
red_area > 10000
→ 10000은 NORMAL

red_area >= 10000
→ 10000도 WARNING
```

운영 Rule에서 경계조건을 정확히 적어야 하는 이유입니다.

</details>

---

## Challenge 3 — Python 코드는 그대로 두고 Threshold만 바꾸기

이번 Challenge에서는 다음 파일을 수정하지 않습니다.

```text
src/rule_engine.py
scripts/rule_image_test.py
scripts/rule_camera_probe.py
```

같은 `day05_red.jpg`를 사용하고 `settings.yaml`의 Threshold만 바꿉니다.

실험 A:

```yaml
red_area_threshold: 5000
```

실험 B:

```yaml
red_area_threshold: 10000
```

실험 C:

```yaml
red_area_threshold: 20000
```

각 설정마다 다음을 실행합니다.

```bash
python -m scripts.rule_image_test \
  --image data/captures/day05_red.jpg
```

실행하기 전에 먼저 예측합니다.

```text
같은 이미지의 red_area는 크게 달라질까?

Threshold가 높아질수록 WARNING은 쉬워질까 어려워질까?

Python 코드를 수정하지 않았는데 State가 달라질 수 있을까?
```

다음 표를 채웁니다.

| Threshold | Red Area | State | 관찰 |
|---:|---:|---|---|
| 5000 | | | |
| 10000 | | | |
| 20000 | | | |

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

같은 저장 이미지를 사용하면 `red_area`는 같은 HSV 설정에서는 거의 같은 값으로 계산됩니다.

예를 들어 `red_area=18420`이라면:

```text
Threshold 5000
18420 >= 5000
→ WARNING

Threshold 10000
18420 >= 10000
→ WARNING

Threshold 20000
18420 < 20000
→ NORMAL
```

바뀐 것은 입력 이미지나 Rule 코드가 아니라 **운영 기준값**입니다.

```text
같은 측정값
+
다른 Threshold
        ↓
다른 운영 상태
```

이것은 1일차와 4일차에서 반복해서 확인했던 원리와 같습니다.

</details>

Challenge가 끝나면 자신이 Guided Lab에서 선택한 기본 Threshold로 되돌립니다.

---

## Challenge 4 — 기존 Rule 기능에 `red_ratio`를 추가하기

현재 `red_area`는 빨간색으로 판단된 Pixel의 **개수**입니다.

이번에는 전체 Frame 중 빨간 Pixel이 차지하는 비율도 함께 계산합니다.

요구사항:

```text
red_ratio
=
red_area / 전체 Pixel 수
```

예를 들어:

```text
Frame 640 × 480
→ 전체 307,200 Pixel

red_area = 30,720
→ red_ratio = 0.10
→ 약 10%
```

먼저 어느 파일을 수정해야 하는지 생각합니다.

```text
Rule 결과 구조는 어디에 있는가?

Frame 크기는 어느 코드에서 알 수 있는가?

red_area를 계산한 직후 무엇을 추가하면 되는가?

터미널 출력은 어느 파일에서 수정해야 하는가?
```

`RuleResult`에 `red_ratio`를 추가하고 `rule_camera_probe.py`에서 함께 출력해 봅니다.

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

`src/rule_engine.py`의 결과 구조에 값을 추가할 수 있습니다.

```python
@dataclass
class RuleResult:
    red_area: int
    red_ratio: float
    state: str
    mask: object
```

`analyze()`에서 전체 Pixel 수와 비율을 계산합니다.

```python
height, width = frame.shape[:2]

total_pixels = height * width

red_ratio = (
    red_area / total_pixels
    if total_pixels > 0
    else 0.0
)
```

반환값에도 추가합니다.

```python
return RuleResult(
    red_area=red_area,
    red_ratio=red_ratio,
    state=state,
    mask=mask,
)
```

`rule_camera_probe.py`의 출력에는 다음처럼 추가할 수 있습니다.

```python
print(
    f"red_area="
    f"{result.red_area:8d} | "
    f"ratio="
    f"{result.red_ratio:6.3f} | "
    f"{result.state}"
)
```

예:

```text
red_area=   24180 | ratio= 0.079 | WARNING
```

이 실습의 핵심은 `red_ratio`가 더 좋은 Rule이라는 결론을 내리는 것이 아닙니다.

```text
요구사항 추가
→ 어떤 데이터가 더 필요한가?
→ 어느 파일이 책임지는가?
→ 어느 결과 구조를 바꿔야 하는가?
→ 어느 출력이 영향을 받는가?
```

를 추적하는 연습입니다.

</details>

Challenge가 끝나면 6일차 연결을 단순하게 유지하기 위해 `red_ratio` 변경은 원래 코드로 복원합니다.

---

## Challenge 5 — 일부러 Config 오류를 만들고 원인 찾기

`configs/settings.yaml`에서 다음 Key를 잠시 잘못 작성합니다.

정상:

```yaml
red_area_threshold: 10000
```

오류:

```yaml
red_area_threshol: 10000
```

마지막 `d`가 빠졌습니다.

실행합니다.

```bash
python -m scripts.rule_camera_probe
```

오류가 발생하면 바로 정답을 보지 않고 다음 순서로 확인합니다.

```text
오류의 마지막 줄 확인
        ↓
어떤 Key를 찾지 못했는가?
        ↓
그 Key를 사용하는 Python 파일 찾기
        ↓
settings.yaml의 실제 Key와 비교
        ↓
오타 수정
        ↓
재실행
```

다음 질문에 답합니다.

```text
오류 종류는 무엇인가?

Camera Hardware 문제인가?

어느 파일을 먼저 수정해야 하는가?

수정 후 같은 명령이 정상 실행되는가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

다음과 비슷한 오류가 발생할 수 있습니다.

```text
KeyError: 'red_area_threshold'
```

`build_rule()`은 다음 Key를 찾습니다.

```python
config["red_area_threshold"]
```

하지만 YAML에는 다음처럼 오타가 있습니다.

```yaml
red_area_threshol: 10000
```

따라서 Hardware나 OpenCV를 수정하는 문제가 아닙니다.

```text
Config Key 문제
```

입니다.

원래 이름으로 복구합니다.

```yaml
red_area_threshold: 10000
```

다시 실행합니다.

```bash
python -m scripts.rule_camera_probe
```

정상 실행되어야 합니다.

</details>

---

## Challenge 6 — Log와 이미지로 정상 동작을 증명하기

이번 Challenge에서는 터미널 화면만 보고 성공이라고 판단하지 않습니다.

기존 Challenge용 결과가 섞이지 않도록 현재 Log와 WARNING 이미지를 잠시 정리합니다.

필요한 결과를 보존해야 한다면 먼저 다른 폴더로 복사합니다.

새 실험으로 시작할 때:

```bash
rm -f logs/day05_rule_events.csv

rm -f data/warning_images/*.jpg
```

통합 Program을 실행합니다.

```bash
python -m scripts.rule_warning_system
```

다음 순서로 장면을 만듭니다.

```text
1. 빨간 카드 없음
→ NORMAL

2. 빨간 카드 크게 보여주기
→ WARNING

3. 카드를 치우기
→ NORMAL

4. 다시 빨간 카드 보여주기
→ WARNING

5. 종료
```

이제 Event Log를 확인합니다.

```bash
cat logs/day05_rule_events.csv
```

WARNING 이미지도 확인합니다.

```bash
find data/warning_images \
  -type f \
  -name "*.jpg" | sort
```

다음 질문에 답합니다.

```text
왜 첫 NORMAL도 Event에 기록될 수 있는가?

WARNING 상태가 여러 Frame 지속되어도
왜 WARNING 이미지가 매 Frame마다 생기지 않는가?

두 번 WARNING으로 전환했다면
WARNING 대표 이미지는 보통 몇 장 기대하는가?

Event CSV의 WARNING 행과 이미지 경로가 연결되는가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

`previous_state = None`으로 시작하기 때문에 첫 상태도 다음 비교에서 변화로 간주됩니다.

```text
None
→ NORMAL
→ changed=True
→ 최초 상태 기록
```

그 뒤에는 같은 상태가 계속되면:

```text
WARNING
→ WARNING
→ changed=False
→ Event 추가 기록 없음
→ WARNING 대표 이미지 추가 저장 없음
```

따라서 장면을:

```text
NORMAL
→ WARNING
→ NORMAL
→ WARNING
```

으로 만들었다면 Event CSV에는 대략 네 개의 상태 기록이 생길 수 있고, WARNING 전환은 두 번이므로 대표 WARNING 이미지는 보통 두 장을 기대할 수 있습니다.

실제 파일 수는 프로그램이 정상적으로 전환을 인식한 횟수와 실험 과정에 따라 달라질 수 있습니다.

핵심은 **터미널 결과와 영구 기록이 같은 상태 변화를 설명하는지 확인하는 것**입니다.

</details>

---

## Challenge 7 — 오늘의 전체 Pipeline을 자신의 말로 설명하기

코드를 닫고 다음 빈칸을 채웁니다.

```text
Camera Frame을 읽는 파일은 __________________________ 이다.

Frame을 HSV로 바꾸고 Red Mask를 만드는 파일은
__________________________ 이다.

Red Area를 계산하는 함수는 __________________________ 이다.

WARNING 기준값은 __________________________ 에 있다.

NORMAL / WARNING의 실제 LED 출력은
__________________________ 이 담당한다.

상태 변화 CSV는 __________________________ 이 기록한다.

WARNING 대표 Frame은 __________________________ 함수가 저장한다.
```

그 다음 아래 흐름을 **파일명을 보지 않고 자신의 말로 설명**합니다.

```text
USB Camera
→ Frame
→ HSV
→ Red Mask
→ Red Area
→ Threshold
→ NORMAL / WARNING
→ GPIO
→ Event Log
→ WARNING Image
```

마지막으로 6일차와의 차이도 설명합니다.

```text
5일차
사람이 정한 HSV / Area Rule

6일차
실제 이미지 Dataset으로 학습한 Tiny CNN
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
Camera Frame
→ src/camera_input.py

HSV / Red Mask
→ src/rule_engine.py

Red Area 계산
→ cv2.countNonZero()

WARNING 기준값
→ configs/settings.yaml의 red_area_threshold

LED / Buzzer 출력
→ src/gpio_output.py

상태 변화 CSV
→ src/event_logger.py의 VisionRuleEventLogger

WARNING 대표 이미지
→ save_warning_image()
```

한 문장으로 설명하면 다음과 같습니다.

> USB Camera에서 들어온 Frame을 HSV 색상공간으로 바꾸고 빨간색 Mask의 Pixel 수를 계산한 뒤, 사람이 정한 Area Threshold와 비교하여 NORMAL 또는 WARNING을 결정하고, 그 결과를 GPIO와 Event Log 및 WARNING 이미지로 남기는 Rule 기반 Edge Pipeline입니다.

6일차에는 입력 Camera 자체가 바뀌는 것이 아니라 **판단 방법을 데이터 기반 AI로 바꾸기 위한 Dataset과 Tiny CNN 모델**을 준비합니다.

</details>

---

# 38. Mini Challenge 마무리 — 6일차에 사용할 기본 상태로 복원하기

Challenge에서는 학습을 위해 설정과 코드를 바꾸었습니다.

6일차는 5일차 Rule Baseline이 정상 동작하는 상태에서 시작하므로 핵심 파일을 기본 상태로 되돌립니다.

## 1. `red_area_threshold` 복원

Guided Lab에서 실제 환경을 보고 선택한 **최종 Threshold**로 되돌립니다.

예를 들어 최종 선택값이 `10000`이었다면:

```yaml
red_area_threshold: 10000
```

`5000`, `20000`, `100` 같은 실험값이 남아 있지 않은지 확인합니다.

## 2. Config Key 이름 확인

```yaml
red_area_threshold:
```

마지막 글자까지 정확한지 확인합니다.

## 3. `rule_engine.py` 복원

Challenge 4에서 `red_ratio`를 추가했다면 기본 `RuleResult`로 되돌립니다.

```python
@dataclass
class RuleResult:
    red_area: int
    state: str
    mask: object
```

반환값도 다음 기본 구조를 사용합니다.

```python
return RuleResult(
    red_area=red_area,
    state=state,
    mask=mask,
)
```

## 4. Buzzer 안전 설정 확인

사양을 확인하지 않은 장비라면:

```yaml
use_buzzer: false
```

를 유지합니다.

## 5. 기본 실행 재확인

```bash
python -m scripts.rule_image_test \
  --image data/captures/day05_normal.jpg
```

```bash
python -m scripts.rule_image_test \
  --image data/captures/day05_red.jpg
```

실시간:

```bash
python -m scripts.rule_camera_probe
```

통합:

```bash
python -m scripts.rule_warning_system
```

다음 상태가 모두 확인되면 복원이 완료되었습니다.

```text
NORMAL 장면
→ NORMAL
→ Green

WARNING 장면
→ WARNING
→ Red
→ 선택 Buzzer
→ Event Log
→ WARNING Image
```

이 상태를 **Day 05 Rule Baseline**으로 보존합니다.

---

# 39. 5일차 실습 결과를 Report로 정리하기

다음 파일을 만듭니다.

```text
reports/day05_rule_warning.md
```

이 Report는 단순히 “성공했다”라고 적는 문서가 아닙니다.

오늘 실제로 사용한 환경값, Rule 기준, 정상/실패 조건, 최종 Threshold를 기록하여 **왜 이 Rule이 그렇게 동작했는지 다시 설명할 수 있게 만드는 문서**입니다.

다음 틀을 사용합니다.

````md
# Day 05 Rule Edge Warning System

## 1. Environment

Raspberry Pi Hostname:

Camera Index:

Requested Resolution:

Actual Resolution:

Buzzer Used: yes / no

## 2. Rule

Target Color:

HSV Range 1:

HSV Range 2:

S Min:

V Min:

Area Threshold:

## 3. Fixed Image Test

### Normal Image

Image:

Red Area:

Threshold:

State:

Mask 확인:

### Warning Image

Image:

Red Area:

Threshold:

State:

Mask 확인:

## 4. Live Camera Probe

카드 없음 Red Area 범위:

카드 작게 Red Area 범위:

카드 크게 Red Area 범위:

## 5. Area Threshold Experiment

| Threshold | 카드 없음 | 카드 작게 | 카드 크게 | 관찰 |
|---:|---|---|---|---|
| 5000 |  |  |  |  |
| 10000 |  |  |  |  |
| 20000 |  |  |  |  |

## 6. 내가 선택한 최종 Threshold

값:

선택 이유:

## 7. End-to-End

Camera:

Rule:

Decision:

GPIO:

Event Log:

Warning Image:

## 8. Event Evidence

첫 상태:

WARNING 전환 횟수:

NORMAL 복귀 횟수:

Event CSV 행 수:

WARNING Image 수:

Log와 Image가 연결되었는가?:

## 9. Failure Cases

### Case 1 — 조명 변화

조건:

예상:

실제:

Red Area:

State:

원인:

개선 아이디어:

### Case 2 — 거리 변화

조건:

예상:

실제:

Red Area:

State:

원인:

개선 아이디어:

### Case 3 — 배경 변화

조건:

예상:

실제:

Red Area:

State:

원인:

개선 아이디어:

## 10. Code Error와 Rule Limit 구분

내가 경험한 Code/Config 오류:

원인:

수정:

재실행 결과:

내가 경험한 Rule 한계:

왜 코드 오류와 다른가?:

## 11. Mini Challenge

### 결과 역추적

내가 찾은 핵심 파일 연결:

### 실행 전 예측

예측과 실제 결과가 달랐던 부분:

### Config 변경

Threshold 변경 결과:

### 기능 수정

수정한 기능:

영향받은 파일:

### 일부러 만든 오류

오류 메시지:

원인:

복구:

### 정상 동작 증명

Event Log:

WARNING Image:

## 12. Day 05 Rule Baseline 한 문장

내 설명:

## 13. 6일차 연결

5일차 Rule 방식:

6일차 AI 방식:

6일차 Class:
- normal
- warning_red

5일차 Rule이 오판정한 장면도
6일차에서는 실제 장면 의미로 Label해야 하는 이유:

## 14. Rule 방식에서 확인한 한계

- 
- 
- 
````

Report에는 비밀번호, Wi-Fi 비밀번호, 인증키 같은 민감한 정보를 적지 않습니다.

실제 IP도 제출 정책상 필요하지 않다면 굳이 기록하지 않아도 됩니다. 장치를 구분할 때는 Hostname이나 장치 번호 정도로 충분할 수 있습니다.

---

# 40. README에 5일차 실행 방법 추가하기

`README.md`에 다음 내용을 추가합니다.

````md
## Day 05 Rule Edge Warning System

### 저장 이미지 Rule Test

```bash
python -m scripts.rule_image_test \
  --image data/captures/day05_red.jpg
```

### Camera Rule Probe

```bash
python -m scripts.rule_camera_probe
```

### Rule Warning System

```bash
python -m scripts.rule_warning_system
```

### 현재 Pipeline

```text
USB Camera
→ OpenCV Frame
→ BGR → HSV
→ Red Color Mask
→ Red Area
→ Area Threshold
→ NORMAL / WARNING
→ GPIO
→ Event Log
→ WARNING Image
```

### Runtime 결과

```text
logs/day05_rule_events.csv
data/warning_images/
```

Runtime 결과는 Git에 올리지 않고, 실험 해석은 `reports/day05_rule_warning.md`에 기록합니다.
````

README에 명령을 적은 뒤 실제 파일명과 실행 명령이 맞는지 다시 확인합니다.

---

# 41. Git에 올리기 전에 Runtime 데이터와 Source를 구분하기

오늘은 Camera 이미지와 Event Log가 많이 생깁니다.

이 파일들은 실행할 때마다 달라지는 Runtime 데이터입니다.

```text
data/captures/
data/day05_debug/
data/day05_failures/
data/warning_images/

logs/day05_rule_events.csv
```

1일차부터 `data/`와 `logs/`를 `.gitignore`로 제외했다면 Git 대상에 나타나지 않아야 합니다.

반면 다음은 Source 또는 학습 결과 문서입니다.

```text
configs/settings.yaml

src/rule_engine.py
src/event_logger.py

scripts/hsv_preview.py
scripts/rule_image_test.py
scripts/rule_camera_probe.py
scripts/rule_warning_system.py

reports/day05_rule_warning.md
README.md
```

`configs/device.local.yaml`도 장치 전용 설정이므로 Git에 올리지 않습니다.

중요한 구분:

```text
Runtime 원본 데이터
→ Git 제외

학생이 작성한 Source
→ Git 관리

실험 결과를 해석한 Report
→ Git 관리

장치별 Local 설정 / 민감정보
→ Git 제외
```

---

# 42. Version 관리 방법은 현재 Source 전달 방식에 맞춥니다

2일차부터 사용한 방식에 따라 다릅니다.

## 방법 A — Raspberry Pi가 Git Repository인 경우

현재 상태를 확인합니다.

```bash
git status
git diff
```

Rule 모듈과 실행 Program이 정상이라면 Stage합니다.

```bash
git add \
  configs/settings.yaml \
  src/rule_engine.py \
  src/event_logger.py \
  scripts/hsv_preview.py \
  scripts/rule_image_test.py \
  scripts/rule_camera_probe.py \
  scripts/rule_warning_system.py
```

Commit:

```bash
git commit -m \
  "feat: add hsv red-area rule pipeline"
```

확인:

```bash
git log --oneline -7
```

Report와 README까지 완성한 뒤:

```bash
git add \
  README.md \
  reports/day05_rule_warning.md
```

```bash
git commit -m \
  "docs: record day05 rule baseline results"
```

## 방법 B — Raspberry Pi에 SCP로 Source만 전달한 경우

Raspberry Pi 폴더가 Git Repository가 아니라면 `git commit`을 억지로 실행하지 않습니다.

현재 정상 Source를 PC로 가져와 PC의 Local Git에서 Version을 관리합니다.

예를 들어 PC에서 필요한 파일을 가져옵니다.

```bash
scp <RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/src/rule_engine.py \
  src/

scp <RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/src/event_logger.py \
  src/
```

실행 Program도 가져옵니다.

```bash
scp <RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/scripts/hsv_preview.py \
  scripts/

scp <RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/scripts/rule_image_test.py \
  scripts/

scp <RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/scripts/rule_camera_probe.py \
  scripts/

scp <RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/scripts/rule_warning_system.py \
  scripts/
```

공통 설정과 Report도 필요한 경우 가져옵니다.

```bash
scp <RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/configs/settings.yaml \
  configs/

scp <RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/reports/day05_rule_warning.md \
  reports/
```

PC Local Git에서:

```bash
git status
git diff
git add configs src scripts reports README.md
git commit -m "feat: complete day05 rule baseline"
```

---

# 43. Remote 저장소는 교육장 보안정책에 맞게 사용하기

외부 GitHub 사용이 제한된 교육장에서는 외부 Repository로 Push하지 않습니다.

```text
허용된 내부 GitLab / Gitea가 있음
→ 내부 Remote 사용 가능

외부 Git 금지
→ Local Git Commit으로 Version 관리

SCP 방식
→ Raspberry Pi에서 실행
→ PC로 Source 회수
→ PC Local Git Commit
```

허용된 Remote가 이미 연결되어 있을 때만 다음을 사용합니다.

```bash
git remote -v
git push
```

촬영 이미지, 실행 Log, 장치별 Local 설정, 비밀번호 같은 정보는 외부 Repository에 올리지 않습니다.

---

# 44. README에 5일차 실행 순서가 실제 코드와 맞는지 확인하기

README에 적은 명령이 실제 파일명과 일치하는지 직접 실행해 봅니다.

저장 이미지 Rule:

```bash
python -m scripts.rule_image_test \
  --image data/captures/day05_red.jpg
```

실시간 Rule Probe:

```bash
python -m scripts.rule_camera_probe
```

통합 Rule System:

```bash
python -m scripts.rule_warning_system
```

README는 코드와 다른 설명을 적는 문서가 아니라 **현재 정상 Version을 다시 실행하기 위한 안내서**입니다.

---

# 45. PC와 Raspberry Pi의 Source가 같은 Version인지 확인하기

내부 Remote를 사용하는 경우:

```text
Raspberry Pi
→ Commit / Push
        ↓
PC
→ Pull
```

PC에서:

```bash
cd ~/ai_vision/subject13_edge_ai
git pull
git log --oneline -7
```

SCP 방식이라면 이미 필요한 Source를 PC로 가져왔으므로 PC Local Git의 최신 Commit을 확인합니다.

```bash
git log --oneline -7
```

두 환경에서 중요한 것은 파일을 무조건 같은 방식으로 전달하는 것이 아니라 다음입니다.

```text
Raspberry Pi에서 정상 실행한 Source
        ↓
PC에도 동일한 Source 보존
        ↓
6일차에서 이어서 사용
```

---

# 46. 5일차가 끝난 시점의 프로젝트 구조

공통 Source:

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
│   └── day05_rule_warning.md
│
├── scripts/
│   ├── button_sampling.py
│   ├── button_test.py
│   ├── camera_gpio_demo.py
│   ├── camera_index_probe.py
│   ├── camera_test.py
│   ├── capture_series.py
│   ├── check_config.py
│   ├── check_device.py
│   ├── gpio_test.py
│   ├── hsv_preview.py
│   ├── led_single_test.py
│   ├── rule_camera_probe.py
│   ├── rule_image_test.py
│   ├── rule_warning_system.py
│   ├── run_pipeline.py
│   ├── sensor_event.py
│   ├── sensor_gpio_demo.py
│   ├── sensor_once.py
│   └── sensor_sampling.py
│
├── src/
│   ├── camera_input.py
│   ├── config_loader.py
│   ├── device_info.py
│   ├── event_logger.py
│   ├── gpio_output.py
│   ├── pipeline.py
│   ├── rule_engine.py
│   ├── sensor_input.py
│   └── sensor_logger.py
│
├── .gitignore
├── README.md
└── requirements.txt
```

장치에만 존재:

```text
.venv/

configs/device.local.yaml

data/
├── captures/
├── day05_debug/
├── day05_failures/
└── warning_images/

logs/
└── day05_rule_events.csv
```

---

# 47. 5일차 최종 실행 확인

Raspberry Pi에 접속한 상태에서:

```bash
cd ~/ai_vision/subject13_edge_ai
source .venv/bin/activate
```

기본 설정을 확인합니다.

```bash
grep -E \
  "red_h|red_s_min|red_v_min|red_area_threshold|use_buzzer" \
  configs/settings.yaml
```

Camera:

```bash
python -m scripts.camera_test
```

GPIO:

```bash
python -m scripts.gpio_test
```

저장 이미지 NORMAL:

```bash
python -m scripts.rule_image_test \
  --image data/captures/day05_normal.jpg
```

저장 이미지 WARNING:

```bash
python -m scripts.rule_image_test \
  --image data/captures/day05_red.jpg
```

실시간 Rule 값:

```bash
python -m scripts.rule_camera_probe
```

통합:

```bash
python -m scripts.rule_warning_system
```

종료 후 Event Log:

```bash
cat logs/day05_rule_events.csv
```

WARNING 이미지:

```bash
find data/warning_images \
  -type f \
  -name "*.jpg" | sort
```

현재 폴더가 Git Repository라면:

```bash
git status
git log --oneline -7
```

SCP 방식이라면 PC Local Git에서 마지막 Commit을 확인합니다.

이 모든 확인이 끝나면 5일차의 Rule Baseline은 6일차에 연결할 수 있는 정상 상태입니다.

---

# 47-1. 5일차 핵심 복습 문제

교재와 코드를 잠시 닫고 먼저 답해 봅니다.

1. 4일차의 `Sensor Value → Threshold → State`와 5일차의 구조는 어떻게 대응되나요?
2. OpenCV Camera Frame이 기본적으로 사용하는 Channel 순서는 무엇인가요?
3. HSV의 H, S, V는 각각 무엇을 의미하나요?
4. 빨간색 Hue를 두 구간으로 나누어 사용하는 이유는 무엇인가요?
5. `cv2.inRange()`는 어떤 결과를 만드나요?
6. `cv2.bitwise_or()`는 오늘 어디에 사용했나요?
7. `cv2.countNonZero(mask)`가 만드는 값은 무엇인가요?
8. `red_area` 자체가 WARNING을 의미하나요?
9. `red_area_threshold`는 어느 파일에 있나요?
10. Camera index는 왜 `device.local.yaml`에 두나요?
11. `rule_engine.py`는 Camera를 직접 여나요?
12. 저장 이미지 한 장으로 먼저 Rule을 확인한 이유는 무엇인가요?
13. `rule_camera_probe.py`와 `rule_warning_system.py`의 역할은 어떻게 다른가요?
14. 첫 상태가 Event Log에 기록될 수 있는 이유는 무엇인가요?
15. WARNING이 계속 유지될 때 매 Frame을 Event로 기록하지 않는 이유는 무엇인가요?
16. `use_buzzer: false`가 필요한 상황은 언제인가요?
17. 조명이 어두워져 오판정하면 무조건 Python 코드 오류인가요?
18. 같은 빨간 카드가 멀어졌을 때 NORMAL로 바뀔 수 있는 이유는 무엇인가요?
19. 5일차 Rule Baseline을 먼저 만드는 이유는 무엇인가요?
20. 6일차에는 5일차의 어떤 부분이 AI 방식으로 바뀌나요?

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

1. 4일차의 Sensor Value 대신 5일차에서는 Camera Frame에서 계산한 `red_area`가 측정값 역할을 합니다.
2. BGR입니다.
3. H는 색상 종류, S는 채도, V는 밝기를 나타냅니다.
4. OpenCV Hue 범위에서 빨간색이 0 부근과 179 부근 양쪽에 걸쳐 나타날 수 있기 때문입니다.
5. 지정한 HSV 범위에 해당하는 Pixel을 흰색, 나머지를 검은색으로 표현하는 Binary Mask를 만듭니다.
6. 빨간색 Hue 구간 1 Mask와 구간 2 Mask를 하나로 합칠 때 사용했습니다.
7. Mask에서 0이 아닌 Pixel 수, 즉 현재 Rule이 빨간색이라고 본 면적을 만듭니다.
8. 아닙니다. `red_area`는 측정값이고 Threshold와 비교한 결과가 NORMAL/WARNING입니다.
9. `configs/settings.yaml`입니다.
10. USB 연결 상태와 장치 환경에 따라 실제 Camera index가 달라질 수 있는 장치 전용 값이기 때문입니다.
11. 아닙니다. 이미 받은 Frame을 분석하는 역할만 합니다.
12. 입력을 고정하여 Rule 자체의 문제를 먼저 확인하기 위해서입니다.
13. `rule_camera_probe.py`는 실시간 측정값과 상태를 관찰하는 점검 Program이고, `rule_warning_system.py`는 GPIO·Event Log·WARNING Image까지 연결하는 통합 Program입니다.
14. `previous_state=None`으로 시작하므로 첫 상태는 이전 값과 다르다고 판단되기 때문입니다.
15. 같은 WARNING을 매 Frame 저장하면 Event Log와 이미지가 지나치게 많아지기 때문입니다.
16. Buzzer 사양을 확인하지 못했거나 Buzzer를 사용하지 않는 장비에서 사용합니다.
17. 아닙니다. 프로그램은 정상 실행되지만 현재 HSV Rule이 조명 변화에 민감한 Rule 설계 한계일 수 있습니다.
18. 면적 기반 Rule에서는 Camera에서 멀어지면 대상이 차지하는 Pixel 수가 줄어들기 때문입니다.
19. 이후 AI 방식과 같은 문제에서 성공·실패 조건을 비교할 기준점이 필요하기 때문입니다.
20. Camera 입력·GPIO·Log의 큰 구조는 유지하고, 사람이 만든 HSV/Area Rule 대신 이미지 Dataset으로 학습한 Tiny CNN 판단을 준비합니다.

</details>

---

# 47-2. 자가 체크리스트

다음 항목을 실제로 확인합니다.

- [ ] 4일차 `subject13_edge_ai` 프로젝트를 그대로 이어서 사용했다.
- [ ] 새 `.venv`를 만들지 않고 기존 Raspberry Pi 환경을 사용했다.
- [ ] Camera와 GPIO를 Rule 통합 전에 각각 독립적으로 확인했다.
- [ ] 실제 Camera index는 추측하지 않고 확인한 값을 사용했다.
- [ ] `day05_normal.jpg`와 `day05_red.jpg`를 준비했다.
- [ ] BGR과 HSV의 차이를 자신의 말로 설명할 수 있다.
- [ ] `hsv_preview.py`에서 실제 Frame을 HSV로 변환했다.
- [ ] `rule_engine.py`가 Camera 입력과 분리된 판단 모듈이라는 것을 이해한다.
- [ ] 저장 이미지에서 Red Mask를 직접 확인했다.
- [ ] NORMAL 이미지와 WARNING 후보 이미지의 `red_area`를 비교했다.
- [ ] Threshold를 감으로만 정하지 않고 실제 측정값을 보고 선택했다.
- [ ] 해상도·거리 변화가 `red_area`에 영향을 줄 수 있음을 이해한다.
- [ ] `rule_camera_probe.py`에서 실시간 `red_area` 변화를 확인했다.
- [ ] 4일차의 Event 개념을 Vision Rule에 다시 연결했다.
- [ ] WARNING 전환 시 대표 이미지가 저장되는 것을 확인했다.
- [ ] `rule_warning_system.py`에서 Camera→Rule→GPIO→Log가 연결되었다.
- [ ] Buzzer 사양이 확인되지 않았다면 `use_buzzer: false`를 유지했다.
- [ ] 조명·거리·배경 변화에서 최소 한 가지 실패 조건을 직접 확인했다.
- [ ] 코드 오류와 Rule 설계 한계를 구분할 수 있다.
- [ ] Mini Challenge에서 실행 결과를 코드 위치로 역추적했다.
- [ ] Mini Challenge에서 실행 전에 결과를 예측했다.
- [ ] Mini Challenge에서 Python 코드를 바꾸지 않고 Config만 변경했다.
- [ ] Mini Challenge에서 기존 Rule 기능 일부를 수정해 보았다.
- [ ] Mini Challenge에서 일부러 Config 오류를 만들고 원인을 찾아 복구했다.
- [ ] Event CSV와 WARNING Image로 정상 동작을 증명했다.
- [ ] Challenge가 끝난 뒤 Day 05 Rule Baseline 상태로 복원했다.
- [ ] `reports/day05_rule_warning.md`에 결과와 실패 사례를 기록했다.
- [ ] README에 Day 05 실행 방법을 기록했다.
- [ ] Runtime 이미지·Log와 Source/Report를 구분하여 Git 관리했다.
- [ ] 오늘의 전체 Pipeline을 자신의 말로 설명할 수 있다.
- [ ] 6일차의 `normal / warning_red` Dataset이 5일차 Rule 출력값이 아니라 실제 장면 의미로 Label되어야 함을 이해한다.

---

# 48. 1~5일차 시스템 성장 확인하기

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
→ 같은 Project 실행
```

3일차:

```text
Camera
→ Raspberry Pi
→ GPIO
```

4일차:

```text
Sensor
→ Sampling
→ Timestamp
→ CSV
→ GPIO
```

5일차:

```text
USB Camera
→ OpenCV
→ HSV
→ Rule
→ NORMAL / WARNING
→ GPIO
→ Event Log
→ WARNING Image
```

5일차가 끝나면 처음으로 다음 구조가 하나로 연결됩니다.

```text
실제 입력
→ 처리
→ 판단
→ 운영 상태
→ 실제 출력
→ 기록
```

---

# 49. 지금 만든 Rule 시스템을 왜 Baseline이라고 할까요?

현재 시스템은 단순하지만 매우 중요합니다.

```text
Camera
→ HSV
→ Red Area
→ Threshold
→ WARNING
```

이 시스템은 이후 AI 방식과 비교할 수 있는 첫 기준점이 됩니다.

예를 들어 다음 질문이 가능해집니다.

```text
Rule은 어떤 조건에서 잘 동작했는가?

Rule은 어떤 조건에서 실패했는가?

AI를 사용하면 어떤 실패를 줄일 수 있을까?

AI를 사용했을 때 오히려 새로 생기는 문제는 무엇일까?
```

즉 Rule을 먼저 만들어 두어야 AI를 사용한 이유를 실제 결과로 설명할 수 있습니다.

---

# 50. 다음 날 연결 — Rule Baseline에서 AI Baseline으로

5일차에서는 사람이 직접 정한 기준으로 Camera Frame을 판단했습니다.

```text
USB Camera
→ BGR
→ HSV
→ Red Mask
→ Red Area
→ 사람이 정한 Threshold
→ NORMAL / WARNING
```

6일차에는 **같은 장면 의미**를 AI가 구분하도록 작은 이미지 Dataset을 직접 만듭니다.

기본 Class는 두 개입니다.

```text
normal
→ 빨간색 경고 대상이 없는 장면

warning_red
→ 빨간색 경고 대상이 있는 장면
```

중요한 점은 6일차 Label을 5일차 Rule 출력으로 자동 결정하지 않는 것입니다.

예를 들어 실제로 빨간 카드가 있는데 5일차 Rule이 조명 문제로 `NORMAL`이라고 잘못 판단했다면:

```text
실제 장면
→ warning_red

5일차 Rule 결과
→ NORMAL
```

이어도 6일차 정답 Label은:

```text
warning_red
```

입니다.

그래야 AI가 단순히 기존 Rule의 실수를 복사하는 것이 아니라 **실제 장면의 의미를 학습**할 수 있습니다.

6일차에서는 Camera로 데이터를 Session 단위로 새로 촬영합니다.

```text
Session 01
→ Train

Session 02
→ Validation

Session 03
→ Test
```

연속 Frame을 무작위로 섞어 Train/Test에 나누지 않고 Session을 구분하는 이유는 서로 거의 같은 장면이 여러 Split에 섞이는 것을 줄이기 위해서입니다.

6일차의 기본 흐름은 다음과 같습니다.

```text
Raspberry Pi Camera
        ↓
normal / warning_red 촬영
        ↓
Session 01 / 02 / 03
        ↓
Train / Validation / Test
        ↓
PC로 Dataset 준비
        ↓
96 × 96 RGB
        ↓
Tiny CNN
        ↓
PyTorch Training
        ↓
Validation으로 Model 확인
        ↓
Test로 최종 Baseline 평가
        ↓
models/day06_tiny_cnn.pt
```

즉 5일차와 6일차의 차이는 다음과 같습니다.

```text
5일차
사람이 특징과 기준을 직접 정의
HSV / Red Area / Threshold

6일차
이미지와 Label을 이용해
작은 CNN이 판단 기준을 학습
```

하지만 큰 Edge 시스템 구조는 그대로 이어집니다.

```text
입력
→ 처리
→ 판단
→ 출력
→ 기록
```

7일차에는 6일차의 PyTorch 모델을 ONNX로 변환하고, PC에서 결과 일치 여부를 확인한 뒤 Raspberry Pi로 다시 배포합니다.

```text
5일차
Rule Baseline
        ↓
6일차
PyTorch Tiny CNN Baseline
        ↓
7일차
ONNX
        ↓
Raspberry Pi Inference
```

5일차가 끝났을 때 다음 문장을 자신의 말로 설명할 수 있으면 6일차로 넘어갈 준비가 된 것입니다.

> **“5일차에서는 사람이 정한 HSV·면적 Rule로 판단했고, 6일차에서는 같은 `normal / warning_red` 문제를 실제 이미지 Dataset으로 만들어 작은 AI 모델이 판단 기준을 학습하도록 바꾼다.”**
