
> **오늘의 핵심:** 2일차에 준비한 Raspberry Pi에 실제 USB Camera와 GPIO 출력 장치를 연결하여, 처음으로 **현실의 입력 → Raspberry Pi 처리 → 현실의 출력** 흐름을 만듭니다.
>
> 1일차와 2일차에 사용한 `subject13_edge_ai` 프로젝트를 그대로 이어서 사용합니다. 새 프로젝트를 만들지 않습니다.
>
> 실제 IP, 사용자 이름, Camera 번호 등 환경마다 달라질 수 있는 값은 자신의 장비에서 확인합니다. 비밀번호와 Wi-Fi 비밀번호 같은 민감한 정보는 README·보고서·Git에 기록하지 않습니다.

---

## 오늘 가장 중요한 질문

1일차에는 PC에서 작은 Edge Pipeline의 뼈대를 만들었습니다.

```text
가상 입력
   ↓
Python 처리
   ↓
NORMAL / WARNING
   ↓
터미널 출력
   ↓
CSV Log
```

2일차에는 Raspberry Pi OS를 직접 설치하고 Network와 SSH를 준비한 뒤, 같은 프로젝트를 Raspberry Pi에서 실행했습니다.

```text
PC Source
   ↓
Raspberry Pi
   ↓
Linux / .venv
   ↓
같은 Pipeline 실행
```

오늘은 여기에서 한 단계 더 나아갑니다.

```text
[실제 입력]
USB Camera
    ↓
Raspberry Pi
    ↓
OpenCV
    ↓
Frame 읽기 / 저장
    ↓
상태 확인
    ↓
GPIO
    ↓
[실제 출력]
Green LED / Red LED / 선택 Buzzer
```

즉, 오늘의 가장 중요한 질문은 다음입니다.

> **“Raspberry Pi가 실제 Camera 입력을 받고, Python이 그 결과를 처리한 뒤, GPIO를 통해 실제 장치에 결과를 출력하려면 어떤 순서로 연결해야 하는가?”**

아직 Camera 영상의 내용을 AI가 판단하지는 않습니다.

오늘은 먼저 다음을 확실하게 연결합니다.

```text
현실 입력
→ Linux Device
→ Python
→ 상태
→ 현실 출력
```

이 연결이 안정적으로 되어야 뒤에서 Rule이나 AI Model을 붙일 수 있습니다.

---

## 오늘의 학습 목표

수업이 끝나면 다음을 할 수 있어야 합니다.

```text
1. USB Camera가 Raspberry Pi에서 인식되는지 확인한다.
2. 실제 Camera index를 찾는다.
3. OpenCV로 Frame을 읽고 이미지로 저장한다.
4. 요청 해상도와 실제 Frame 해상도를 구분한다.
5. GPIO의 BCM 번호와 물리 Pin 번호를 구분한다.
6. LED를 안전하게 배선하고 Python으로 제어한다.
7. 선택적으로 Active Buzzer Module을 제어한다.
8. Camera와 GPIO를 각각 독립적으로 Test한다.
9. Camera + GPIO를 하나의 작은 입·출력 Pipeline으로 연결한다.
10. 실패 원인을 Hardware / Linux Device / Config / Python / Integration으로 나누어 찾는다.
```

---

## 오늘의 8시간 학습 흐름

아래 시간은 장비 인식과 배선 속도에 따라 조금 달라질 수 있습니다.

| 학습 블록 | 권장 시간 | 무엇을 배우나요? | 끝나면 할 수 있어야 하는 것 |
|---|---:|---|---|
| 1 | 30분 | 2일차 상태 확인 · 오늘 전체 구조 | Raspberry Pi가 3일차 실습 준비 상태인지 확인할 수 있다 |
| 2 | 60분 | USB Camera · Linux Device · V4L2 | Camera가 Linux에서 인식되는지 확인할 수 있다 |
| 3 | 80분 | OpenCV · Camera index · Frame 저장 | 실제 Frame을 읽고 JPG로 저장할 수 있다 |
| 4 | 50분 | 해상도 · 연속 Capture · 설정 실험 | 설정과 실제 Frame 결과를 비교할 수 있다 |
| 5 | 70분 | GPIO · BCM · 안전 배선 · LED | LED를 안전하게 배선하고 제어할 수 있다 |
| 6 | 60분 | GPIO 모듈 · 선택 Buzzer · 독립 Test | NORMAL/WARNING 출력을 독립적으로 확인할 수 있다 |
| 7 | 60분 | Camera + GPIO 통합 · 실패 분석 | 실제 입력과 출력을 하나의 Program으로 연결할 수 있다 |
| 8 | 70분 | Mini Challenge · 복원 · Report · Git · 복습 | 전체 Pipeline을 자신의 말로 설명하고 4일차로 연결할 수 있다 |

총 480분을 기준으로 구성합니다.

---

## 오늘 사용할 실제 장비와 데이터

오늘의 입력 데이터는 별도로 다운로드하는 Dataset이 아닙니다.

```text
USB Camera가 현재 촬영하는 실제 영상
        ↓
Frame
        ↓
JPG 이미지
```

사용 장비:

```text
Raspberry Pi
USB Camera
microSD / Raspberry Pi OS
PC + SSH
Green LED
Red LED
220~330Ω 저항
Breadboard / Jumper Wire
선택: 3.3V GPIO Signal로 제어 가능한 Active Buzzer Module
```

Buzzer의 사양이 확인되지 않으면 오늘 필수 실습은 **LED까지만** 진행합니다.

---

## 오늘 배울 기술 스택

| 기술 | 오늘 어디에 사용하나요? |
|---|---|
| Raspberry Pi OS / Linux | USB Camera와 GPIO가 연결된 실제 Edge 실행환경 |
| SSH | PC에서 Raspberry Pi Terminal에 원격 접속 |
| V4L2 | Linux가 Camera를 어떻게 인식했는지 확인 |
| OpenCV | Camera Frame 읽기와 이미지 저장 |
| Python | Camera/GPIO 모듈과 실행 Program 작성 |
| `gpiozero` | GPIO 출력 제어 |
| YAML | Camera와 GPIO 설정값 관리 |
| Git | 정상 동작 Version과 문서 기록 |

---

## 오늘 계속 기억할 다섯 단계

1일차에서 배운 다섯 단어를 오늘 실제 장치에 연결합니다.

```text
입력
→ USB Camera Frame

처리
→ OpenCV / Python

판단
→ Capture 성공 / 실패

출력
→ LED / 선택 Buzzer

기록
→ 저장된 JPG / Report / Git
```

오늘 작성하는 코드가 헷갈리면 다음 질문으로 돌아옵니다.

> **“지금 보는 코드는 입력·처리·판단·출력·기록 중 어느 역할을 하고 있는가?”**

---

## 실습 전에 자신의 환경값 적어두기

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
```

이 문서에서는 환경에 따라 달라지는 값에 다음 표기를 사용합니다.

```text
<RPI_USER>
<RPI_HOSTNAME>
<RPI_IP>
<ACTUAL_CAMERA_INDEX>
```

---

## 오늘의 성공 기준

수업이 끝났을 때 다음 연결이 실제로 동작하면 됩니다.

```text
2일차 Raspberry Pi 환경 정상
        ↓
USB Camera 인식
        ↓
OpenCV Frame 읽기
        ↓
JPG 저장
        ↓
GPIO LED Test
        ↓
Camera / GPIO 독립 Test
        ↓
Camera + GPIO 통합
        ↓
정상 입력 → Green
실패 입력 → Red / 선택 Buzzer
        ↓
결과 기록
```

4일차에는 오늘 경험한 **시간에 따라 들어오는 입력**이라는 개념을 Sensor/Input 값으로 확장하여 Sampling · Timestamp · CSV 기록을 다룹니다.

---

## GPIO 실습 안전 규칙

오늘은 실제 전기 신호를 다룹니다.

다음 원칙을 지킵니다.

```text
배선을 바꿀 때
→ Raspberry Pi 종료
→ 전원 분리
→ 배선 변경
→ 다시 확인
→ 전원 연결
```

LED에는 220~330Ω 정도의 저항을 사용합니다.

GPIO Pin에 5V를 직접 입력하지 않습니다.

Buzzer는 제품 사양을 확인한 경우에만 사용합니다.

---

# 1. 2일차 프로젝트에서 이어서 시작하기

2일차가 끝났다면 공통 Repository는 다음과 비슷합니다.

```text
subject13_edge_ai/
│
├── configs/
│   └── settings.yaml
│
├── reports/
│   ├── day01_result.md
│   └── day02_environment.md
│
├── scripts/
│   ├── check_config.py
│   ├── check_device.py
│   └── run_pipeline.py
│
├── src/
│   ├── config_loader.py
│   ├── device_info.py
│   └── pipeline.py
│
├── .gitignore
├── README.md
└── requirements.txt
```

Raspberry Pi에는 Git으로 공유하지 않는 다음 항목도 있습니다.

```text
.venv/
configs/device.local.yaml
logs/
```

2일차까지의 실행 흐름은 다음과 같았습니다.

```text
PC
   │
   │ SSH
   ▼
Raspberry Pi
   │
   ├─ Linux
   ├─ Python .venv
   └─ 기존 Pipeline 실행
```

오늘은 여기에 실제 입·출력 장치를 연결합니다.

```text
USB Camera
     ↓
Raspberry Pi
     ↓
Python / OpenCV
     ↓
실행 결과
     ↓
GPIO
  ┌──┴──┐
 LED   Buzzer
```

---

# 2. Raspberry Pi에 접속하고 2일차 상태 확인하기

오늘은 새 프로젝트를 만들지 않습니다.

2일차에서 준비한 같은 Raspberry Pi와 같은 `subject13_edge_ai` 프로젝트를 이어서 사용합니다.

먼저 PC에서 접속합니다.

```bash
ssh <RPI_USER>@<RPI_IP>
```

접속 직후 다음 네 가지를 확인합니다.

```bash
whoami
hostname
hostname -I
pwd
```

확인 목적:

```text
whoami
→ 지금 어느 사용자로 실행 중인가?

hostname
→ 내가 접속한 Raspberry Pi가 맞는가?

hostname -I
→ 현재 IP는 무엇인가?

pwd
→ 지금 어느 폴더에 있는가?
```

프로젝트 폴더로 이동합니다.

```bash
cd ~/ai_vision/subject13_edge_ai
```

가상환경을 활성화합니다.

```bash
source .venv/bin/activate
```

현재 Python을 확인합니다.

```bash
which python
```

예:

```text
/home/<RPI_USER>/ai_vision/subject13_edge_ai/.venv/bin/python
```

장치 상태도 확인합니다.

```bash
python -m scripts.check_device
```

2일차에 만든 READY 판정을 사용했다면 `READY`인지 확인합니다.

## 프로젝트 Source는 어떻게 최신 상태를 확인할까요?

2일차에서 Source를 Raspberry Pi로 전달한 방법에 따라 다릅니다.

### 방법 A — 내부 GitLab/Gitea 같은 Remote를 Clone한 경우

```bash
git status
git pull
git log --oneline -5
```

2일차 Commit이 보이는지 확인합니다.

### 방법 B — SCP로 Source만 전달한 경우

이 경우 Raspberry Pi 폴더가 Git Repository가 아닐 수 있습니다.

`git pull`을 억지로 실행하지 않습니다.

다음으로 필요한 파일이 있는지 확인합니다.

```bash
ls
ls configs
ls src
ls scripts
```

다음이 보이면 됩니다.

```text
configs/settings.yaml
configs/device.local.yaml
src/config_loader.py
src/device_info.py
src/pipeline.py
scripts/check_config.py
scripts/check_device.py
scripts/run_pipeline.py
requirements.txt
```

## 2일차 Pipeline을 한 번 다시 실행합니다

Camera를 연결하기 전에 기존 Program이 정상인지 확인합니다.

```bash
python -m scripts.check_config
python -m scripts.run_pipeline
```

이 단계가 정상이어야 합니다.

```text
2일차 기존 Pipeline 정상
        ↓
3일차 Camera / GPIO 추가
```

만약 여기서 이미 오류가 난다면 Camera나 GPIO부터 연결하지 않습니다.

```text
기존 Pipeline 오류
→ 먼저 2일차 환경 복구

기존 Pipeline 정상
→ 3일차 Hardware 연결
```

이렇게 해야 이후 오류가 **기존 환경 문제인지, 새 Camera/GPIO 문제인지** 구분하기 쉽습니다.

---

# 3. USB Camera를 Raspberry Pi에 연결하기

오늘 사용하는 Camera는 **PC에 연결하는 것이 아니라 Raspberry Pi의 USB 포트에 직접 연결**합니다.

구조:

```text
USB Camera
     │
     │ USB
     ▼
Raspberry Pi
     │
     │ SSH
     ▼
PC에서 상태 확인
```

PC는 Raspberry Pi 화면을 대신 보는 원격 터미널 역할을 합니다.

실제 Camera Frame을 읽고 처리하는 주체는 Raspberry Pi입니다.

---

# 4. Linux가 Camera를 인식하는지 확인하기

USB 장치 목록을 확인합니다.

```bash
lsusb
```

Camera 제조사나 `Camera`, `Webcam`, `Video`와 관련된 항목이 보이는지 확인합니다.

다음으로 Video Device를 확인합니다.

```bash
ls -l /dev/video*
```

정상적으로 Camera가 인식되었다면 다음과 같은 장치가 보일 수 있습니다.

```text
/dev/video0
/dev/video1
```

한 대의 USB Camera가 여러 `/dev/video*` 장치를 만들 수도 있습니다.

따라서 `/dev/video0`이 보인다고 해서 무조건 OpenCV의 Camera index가 `0`이라고 단정하지 않습니다.

---

# 5. Camera 정보를 더 자세히 확인하기

Linux의 Video4Linux 도구를 설치합니다.

```bash
sudo apt update
sudo apt install -y v4l-utils
```

Camera 목록을 확인합니다.

```bash
v4l2-ctl --list-devices
```

예:

```text
USB Camera:
    /dev/video0
    /dev/video1
```

`/dev/video0`의 지원 정보를 확인할 수 있습니다.

```bash
v4l2-ctl --device=/dev/video0 --all
```

지원 해상도와 Pixel Format을 확인하려면 다음 명령도 사용할 수 있습니다.

```bash
v4l2-ctl --device=/dev/video0 --list-formats-ext
```

출력이 길어도 모두 외울 필요는 없습니다.

다음 정도만 찾으면 됩니다.

```text
Camera가 Linux에서 보이는가?
어떤 /dev/video*가 존재하는가?
어떤 해상도를 지원하는가?
```

---

# 6. 오늘 필요한 Python Package 설치하기

오늘부터 OpenCV와 GPIO를 사용합니다.

현재 가상환경이 활성화되어 있는지 확인합니다.

```bash
which python
```

OpenCV는 SSH 기반 실습이므로 GUI 기능이 없는 Headless Package를 사용합니다.

```bash
python -m pip install opencv-python-headless
```

GPIO 제어를 위해 `gpiozero`를 설치합니다.

```bash
python -m pip install gpiozero
```

설치를 확인합니다.

```bash
python -c "import cv2; print(cv2.__version__)"
```

```bash
python -c "import gpiozero; print('gpiozero import OK')"
```

현재 환경의 Package 목록을 다시 기록합니다.

```bash
python -m pip freeze > requirements.txt
```

확인합니다.

```bash
cat requirements.txt
```

> GPIO Backend 오류가 발생하면 Raspberry Pi OS와 Python 환경에 따라 추가 GPIO Backend가 필요할 수 있습니다.  
> 이 경우 오류 메시지를 먼저 기록한 뒤 `lgpio` 또는 수업 장비에서 사용하는 GPIO Backend를 확인합니다.  
> 정상적으로 동작하는 상태를 확인한 뒤 Package 버전을 고정합니다.

---

# 7. Camera와 GPIO 설정값 추가하기

2일차에는 설정을 두 종류로 나누었습니다.

```text
configs/settings.yaml
→ 모든 장치가 공유하는 공통 설정

configs/device.local.yaml
→ 현재 Raspberry Pi에서만 달라질 수 있는 Local 설정
```

Camera index는 USB 연결 상태에 따라 장치마다 달라질 수 있으므로 **Local 설정**으로 관리합니다.

반면 수업에서 공통으로 사용할 요청 해상도와 표준 GPIO 배선 번호는 공통 설정으로 둡니다.

## `configs/settings.yaml`에 공통 설정 추가

기존 내용은 그대로 두고 아래 항목을 추가합니다.

```yaml
image_width: 640
image_height: 480
capture_dir: data/captures

green_led_bcm: 17
red_led_bcm: 27
buzzer_bcm: 22
use_buzzer: false
```

각 값의 의미:

```text
image_width / image_height
→ Camera에 요청할 Frame 크기

capture_dir
→ 촬영한 JPG를 저장할 폴더

green_led_bcm
→ Green LED GPIO 번호

red_led_bcm
→ Red LED GPIO 번호

buzzer_bcm
→ Buzzer Signal GPIO 번호

use_buzzer
→ Buzzer를 실제로 사용할지 결정
```

Buzzer 사양을 확인하기 전에는 기본값을 `false`로 둡니다.

`true`와 `false`는 따옴표 없이 YAML Boolean 값으로 작성합니다.

## `configs/device.local.yaml`에는 Camera index 추가

현재 파일에 다음 값을 추가합니다.

```yaml
camera_index: 0
```

여기서 `0`은 아직 **예시값**입니다.

곧 `camera_index_probe.py`로 자신의 실제 Camera index를 확인한 뒤 수정합니다.

```text
Camera index를 추측
X

직접 Probe
→ 정상 Frame이 나온 index 확인
→ device.local.yaml에 기록
O
```

이 교재의 GPIO 번호는 **BCM 번호**를 기준으로 합니다.

> `device.local.yaml`은 Git에 올리지 않습니다. 2일차에서 이미 `.gitignore`의 `configs/*.local.yaml` 규칙으로 제외했습니다.

---

# 8. Camera index를 찾는 작은 프로그램 만들기

먼저 Camera가 몇 번 index로 열리는지 확인합니다.

다음 파일을 만듭니다.

```text
scripts/camera_index_probe.py
```

## 이 코드는 왜 만들까요?

Linux에 `/dev/video0`이 보인다고 해서 OpenCV Camera index가 반드시 `0`인 것은 아닙니다.

따라서 `0, 1, 2, 3`을 차례로 열어 실제 Frame까지 읽히는 번호를 찾습니다.

```text
index 0 시도
→ 열림?
→ Frame 읽힘?

index 1 시도
→ 열림?
→ Frame 읽힘?

...
→ 실제 사용할 index 결정
```

## 의사코드

```text
0부터 3까지 Camera index를 하나씩 확인한다
        ↓
해당 index로 Camera를 연다
        ↓
열리지 않으면 OPEN FAIL을 출력한다
        ↓
열렸다면 Frame 한 장을 읽는다
        ↓
Frame이 정상이라면 Width / Height와 함께 OK를 출력한다
        ↓
Frame을 읽지 못하면 FRAME FAIL을 출력한다
        ↓
Camera Resource를 닫는다
        ↓
다음 index를 확인한다
```

코드:

```python
import cv2


def main():
    print("=== Camera Index Probe ===")

    for index in range(4):
        cap = cv2.VideoCapture(index)

        if not cap.isOpened():
            print(f"Camera {index}: OPEN FAIL")
            cap.release()
            continue

        ok, frame = cap.read()

        if ok and frame is not None:
            height, width = frame.shape[:2]

            print(
                f"Camera {index}: OK "
                f"({width} x {height})"
            )
        else:
            print(f"Camera {index}: OPENED BUT FRAME FAIL")

        cap.release()


if __name__ == "__main__":
    main()
```

실행합니다.

```bash
python -m scripts.camera_index_probe
```

예:

```text
=== Camera Index Probe ===
Camera 0: OK (640 x 480)
Camera 1: OPEN FAIL
Camera 2: OPEN FAIL
Camera 3: OPEN FAIL
```

이 경우 기본 Camera index는 `0`으로 사용할 수 있습니다.

0~3에서 모두 실패했는데 `/dev/video*` 장치가 실제로 존재한다면 바로 Camera 고장이라고 단정하지 않습니다.

```text
/dev/video* 다시 확인
→ v4l2-ctl 결과 확인
→ 필요하면 Probe 범위를 0~5 또는 실제 장치 수에 맞게 확장
```

Camera를 USB에서 다시 연결했다면 index가 달라질 수도 있으므로 다시 Probe합니다.

만약 `Camera 1`만 정상이라면 `configs/device.local.yaml`의 값을 다음처럼 변경합니다.

```yaml
camera_index: 1
```

---

# 9. Camera 입력을 모듈로 분리하기

Camera 코드를 여러 실습에서 반복하지 않도록 모듈로 만듭니다.

파일:

```text
src/camera_input.py
```

## 이 코드는 왜 만들까요?

앞의 Probe Program은 Camera 번호를 찾기 위한 점검용 Program입니다.

이제부터 여러 실습에서 Camera를 계속 사용하므로 다음 기능을 한 파일로 모읍니다.

```text
Camera 열기
→ Frame 읽기
→ Camera 닫기
→ Frame 저장
```

`camera_test.py`, `capture_series.py`, 이후 통합 Program이 이 기능을 다시 사용할 수 있습니다.

## 의사코드

```text
Camera index와 요청 해상도를 받는다
        ↓
OpenCV로 Camera를 연다
        ↓
Camera가 열리지 않으면 오류를 발생시킨다
        ↓
요청 Width / Height를 설정한다
        ↓
read()가 호출되면 Frame 한 장을 읽는다
        ↓
실패하면 오류를 발생시킨다
        ↓
성공하면 Frame을 반환한다

save_frame()이 호출되면
        ↓
저장 폴더가 없으면 만든다
        ↓
현재 시간을 이용해 파일명을 만든다
        ↓
JPG로 저장한다
        ↓
실패하면 오류를 발생시킨다
        ↓
성공하면 저장 경로를 반환한다
```

코드:

```python
from datetime import datetime
from pathlib import Path

import cv2


class CameraInput:
    def __init__(
        self,
        index: int,
        width: int,
        height: int,
    ):
        self.index = index
        self.width = width
        self.height = height

        self.cap = cv2.VideoCapture(index)

        if not self.cap.isOpened():
            raise RuntimeError(
                f"Camera를 열 수 없습니다. index={index}"
            )

        self.cap.set(
            cv2.CAP_PROP_FRAME_WIDTH,
            width,
        )

        self.cap.set(
            cv2.CAP_PROP_FRAME_HEIGHT,
            height,
        )

    def read(self):
        ok, frame = self.cap.read()

        if not ok or frame is None:
            raise RuntimeError(
                "Camera Frame을 읽지 못했습니다."
            )

        return frame

    def release(self):
        if self.cap is not None:
            self.cap.release()


def save_frame(
    frame,
    output_dir: str,
    prefix: str = "capture",
) -> Path:
    directory = Path(output_dir)
    directory.mkdir(
        parents=True,
        exist_ok=True,
    )

    timestamp = datetime.now().strftime(
        "%Y%m%d_%H%M%S_%f"
    )

    path = directory / f"{prefix}_{timestamp}.jpg"

    ok = cv2.imwrite(
        str(path),
        frame,
    )

    if not ok:
        raise RuntimeError(
            f"이미지 저장에 실패했습니다: {path}"
        )

    return path
```

이 파일에는 두 가지 역할이 있습니다.

```text
CameraInput
→ Camera를 열고 Frame을 읽는 역할

save_frame()
→ Frame을 이미지 파일로 저장하는 역할
```

앞으로 AI를 연결하더라도 Camera를 읽는 방법은 이 모듈을 재사용할 수 있습니다.

---

# 10. 첫 Camera Frame 읽기

다음 파일을 만듭니다.

```text
scripts/camera_test.py
```

## 이 코드는 왜 만들까요?

`camera_input.py`에 기능을 만들었지만 아직 실제 Raspberry Pi에서 정상 작동하는지 확인하지 않았습니다.

이 Program은 다음 네 가지를 한 번에 확인합니다.

```text
설정 읽기
→ Camera 열기
→ Frame 한 장 읽고 저장
→ 여러 Frame을 연속해서 읽을 수 있는지 확인
```

오늘의 FPS는 Camera 읽기 상태를 확인하기 위한 참고값입니다. AI 추론 성능을 의미하지 않습니다.

## 의사코드

```text
공통 + Local 설정을 읽는다
        ↓
Camera index / Width / Height로 Camera를 연다
        ↓
Camera가 안정화될 시간을 잠시 기다린다
        ↓
Frame 한 장을 읽는다
        ↓
실제 Width / Height / Channel을 확인한다
        ↓
JPG로 저장한다
        ↓
100개의 Frame을 연속으로 읽는다
        ↓
성공한 Frame 수와 걸린 시간을 계산한다
        ↓
참고용 Read FPS를 출력한다
        ↓
성공/실패와 관계없이 Camera Resource를 닫는다
```

코드:

```python
import time

from src.camera_input import (
    CameraInput,
    save_frame,
)
from src.config_loader import load_config


def main():
    config = load_config(
        "configs/settings.yaml"
    )

    camera = CameraInput(
        index=int(config["camera_index"]),
        width=int(config["image_width"]),
        height=int(config["image_height"]),
    )

    try:
        print("=== Camera Test ===")

        # Camera가 안정화될 시간을 조금 줍니다.
        time.sleep(1.0)

        frame = camera.read()

        height, width = frame.shape[:2]

        print(f"Frame Width  : {width}")
        print(f"Frame Height : {height}")
        print(f"Channels     : {frame.shape[2]}")

        saved_path = save_frame(
            frame,
            config["capture_dir"],
            prefix="day03",
        )

        print(f"Saved        : {saved_path}")

        success_count = 0
        total_frames = 100

        started = time.perf_counter()

        for _ in range(total_frames):
            frame = camera.read()

            if frame is not None:
                success_count += 1

        elapsed = time.perf_counter() - started

        print()
        print("=== 100 Frame Read Test ===")
        print(f"Success : {success_count}/{total_frames}")
        print(f"Elapsed : {elapsed:.3f} sec")

        if elapsed > 0:
            print(
                f"Read FPS : "
                f"{success_count / elapsed:.2f}"
            )

    finally:
        camera.release()


if __name__ == "__main__":
    main()
```

실행:

```bash
python -m scripts.camera_test
```

확인할 결과:

```text
Frame Width
Frame Height
Channels
Saved Path
100 Frame Success
Read FPS
```


## 코드 리뷰 — Camera Test 결과는 어디에서 만들어졌을까요?

실행 결과만 보지 말고 코드를 다시 엽니다.

| 결과 | 어느 파일에서 확인할까요? |
|---|---|
| Camera를 실제로 여는 코드 | `src/camera_input.py` → `CameraInput.__init__()` |
| Frame 한 장 읽기 | `src/camera_input.py` → `read()` |
| JPG 저장 | `src/camera_input.py` → `save_frame()` |
| Camera index 값 | `configs/device.local.yaml` |
| Width / Height 값 | `configs/settings.yaml` |
| Frame Width / Height 출력 | `scripts/camera_test.py` |
| 100 Frame 반복 | `scripts/camera_test.py` |
| Camera 종료 | `finally` → `camera.release()` |

다음 질문에 직접 답합니다.

```text
Camera 0/1 번호는 어디에서 최종 결정되는가?
Frame을 읽지 못하면 어느 함수에서 오류가 발생하는가?
저장 폴더가 없으면 어느 코드가 자동으로 만드는가?
프로그램 중간에 오류가 나도 Camera를 닫는 코드는 어디에 있는가?
```

> **실행 결과와 코드 위치를 다시 연결하는 것이 오늘의 첫 번째 코드 리뷰입니다.**


---

# 11. 저장된 이미지를 확인하기

저장 폴더를 확인합니다.

```bash
find data/captures -maxdepth 1 -type f
```

예:

```text
data/captures/day03_20261003_101530_123456.jpg
```

파일 정보를 확인합니다.

```bash
file data/captures/*.jpg
```

PC에서 이미지를 직접 보고 싶다면 SCP로 가져올 수 있습니다.

자신의 Raspberry Pi IP를 확인한 뒤 PC에서:

```bash
scp <RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/data/captures/*.jpg .
```

또는 VS Code Remote SSH 환경을 사용한다면 Raspberry Pi 폴더를 원격으로 열어 파일을 확인할 수도 있습니다.

오늘은 SSH 기반 실습이므로 `cv2.imshow()` 창을 반드시 사용하지 않아도 됩니다.

```text
Camera Frame
→ Raspberry Pi가 읽음
→ JPG 저장
→ PC에서 필요할 때 확인
```

---

# 12. 실제 요청 해상도와 실제 Frame 크기를 비교하기

`settings.yaml`에는 다음 값이 있습니다.

```yaml
image_width: 640
image_height: 480
```

이 값은 Camera에 요청하는 해상도입니다.

실제 Frame은 Camera가 지원하는 해상도에 따라 다르게 반환될 수도 있습니다.

따라서 항상 다음 값을 실제로 확인합니다.

```python
height, width = frame.shape[:2]
```

즉:

```text
설정값
→ 원하는 해상도

frame.shape
→ 실제 받은 해상도
```

두 값이 같은지 직접 확인합니다.

---

# 13. Modify Lab 1 — Camera 해상도 바꾸기

`settings.yaml`에서 다음 값을 바꿉니다.

실험 A:

```yaml
image_width: 640
image_height: 480
```

실행:

```bash
python -m scripts.camera_test
```

결과를 기록합니다.

그다음 Camera가 지원한다면 실험 B:

```yaml
image_width: 1280
image_height: 720
```

다시 실행합니다.

```bash
python -m scripts.camera_test
```

다음 표를 메모합니다.

```text
설정 1
요청 해상도:
실제 해상도:
100 Frame 시간:
Read FPS:

설정 2
요청 해상도:
실제 해상도:
100 Frame 시간:
Read FPS:
```

Camera가 해당 해상도를 지원하지 않으면 다른 크기로 반환될 수 있습니다.

그 결과도 정상적인 실험 결과입니다.

---

# 14. 연속 이미지 10장 저장하기

파일:

```text
scripts/capture_series.py
```

## 이 코드는 왜 만들까요?

Camera는 한 장의 이미지가 아니라 **시간에 따라 Frame이 계속 들어오는 장치**입니다.

따라서 한 장 저장에서 끝내지 않고 Frame을 반복해서 읽고 각각 다른 파일로 저장해 봅니다.

```text
Frame 1 → 저장
Frame 2 → 저장
...
Frame 10 → 저장
```

이 경험이 4일차의 **Sampling** 개념으로 이어집니다.

## 의사코드

```text
설정을 읽는다
        ↓
Camera를 연다
        ↓
1초 기다린다
        ↓
10번 반복한다
        ↓
Frame 한 장을 읽는다
        ↓
서로 다른 Timestamp 파일명으로 저장한다
        ↓
저장 경로를 출력한다
        ↓
0.5초 기다린다
        ↓
반복이 끝나면 Camera를 닫는다
```

코드:

```python
import time

from src.camera_input import (
    CameraInput,
    save_frame,
)
from src.config_loader import load_config


def main():
    config = load_config(
        "configs/settings.yaml"
    )

    camera = CameraInput(
        index=int(config["camera_index"]),
        width=int(config["image_width"]),
        height=int(config["image_height"]),
    )

    try:
        print("=== Capture Series ===")

        time.sleep(1.0)

        for index in range(1, 11):
            frame = camera.read()

            path = save_frame(
                frame,
                config["capture_dir"],
                prefix=f"series_{index:02d}",
            )

            print(
                f"{index:02d}/10 saved: {path}"
            )

            time.sleep(0.5)

    finally:
        camera.release()


if __name__ == "__main__":
    main()
```

실행하기 전에 이전 `series_` 결과만 정리할 수 있습니다.

```bash
rm -f data/captures/series_*.jpg
```

실행합니다.

```bash
python -m scripts.capture_series
```

이번 실습에서 생성한 `series_` 파일 개수를 확인합니다.

```bash
find data/captures -maxdepth 1 -type f -name 'series_*.jpg' | wc -l
```

정상이라면 이번 실행 기준으로 `10`이 나옵니다.

파일명이 서로 겹치지 않는지도 확인합니다.

Timestamp가 들어 있기 때문에 같은 이름으로 덮어쓰는 문제를 줄일 수 있습니다.

---

# 15. Camera 실습 결과에서 확인할 핵심

지금까지 다음 흐름이 실제로 동작했습니다.

```text
USB Camera
     ↓
Linux /dev/video*
     ↓
OpenCV VideoCapture
     ↓
Frame
     ↓
Python NumPy 배열
     ↓
JPG 저장
```

`Frame`은 영상 한 장면입니다.

동영상은 Frame이 시간 순서대로 계속 들어오는 구조입니다.

```text
Frame 1
→ Frame 2
→ Frame 3
→ Frame 4
→ ...
```

3일차에서는 이 연속 Frame을 읽을 수 있다는 것까지 직접 확인합니다.

---

# 16. GPIO 실습 전 전원을 끄고 배선하기

GPIO 배선을 변경하기 전에 Raspberry Pi를 안전하게 종료합니다.

```bash
sudo shutdown -h now
```

Raspberry Pi가 완전히 종료된 뒤 전원을 분리하고 배선합니다.

## 기본 배선 예시

이 교재에서는 다음 BCM 번호를 사용합니다.

| 장치 | BCM GPIO | 물리 Pin 예시 |
|---|---:|---:|
| Green LED | GPIO17 | Pin 11 |
| Red LED | GPIO27 | Pin 13 |
| Buzzer Signal | GPIO22 | Pin 15 |
| GND | - | Pin 6 등 |

LED는 다음처럼 연결합니다.

```text
GPIO17
  ↓
220~330Ω 저항
  ↓
Green LED
  ↓
GND
```

Red LED도 같은 방식입니다.

```text
GPIO27
  ↓
220~330Ω 저항
  ↓
Red LED
  ↓
GND
```

LED의 긴 다리는 일반적으로 Anode(+), 짧은 다리는 Cathode(-)입니다.

방향이 반대면 LED가 켜지지 않을 수 있습니다.

---

# 17. Buzzer 연결 시 주의하기

Buzzer는 제품마다 전압과 구동 방식이 다릅니다.

이 교재의 코드는 **3.3V GPIO Signal로 제어할 수 있는 Active Buzzer Module**을 기준으로 설명합니다.

```text
GPIO22
→ Buzzer Module Signal

GND
→ Buzzer Module GND
```

전원은 사용 중인 Module 사양에 맞춰 연결합니다.

다음 경우에는 GPIO에 바로 연결하지 않습니다.

```text
정격을 모르는 Raw Buzzer
5V 구동이 필요한 Buzzer
GPIO 허용 전류를 넘길 수 있는 장치
```

이 경우 Transistor나 적절한 Driver 회로가 필요할 수 있습니다.

Buzzer 사양을 확인할 수 없다면 **3일차 필수 실습은 LED까지만 진행하고 Buzzer는 제외**합니다.

GPIO 핀에는 5V를 직접 입력하지 않습니다.

---

# 18. 배선 후 다시 부팅하고 GPIO 번호 확인하기

전원을 다시 연결하고 Raspberry Pi가 부팅되면 SSH로 접속합니다.

```bash
ssh <RPI_USER>@<RPI_IP>
```

프로젝트 이동:

```bash
cd ~/ai_vision/subject13_edge_ai
```

가상환경:

```bash
source .venv/bin/activate
```

설정값을 다시 확인합니다.

```bash
grep -E "green_led|red_led|buzzer" configs/settings.yaml
```

예:

```text
green_led_bcm: 17
red_led_bcm: 27
buzzer_bcm: 22
```

**물리 Pin 번호와 BCM 번호를 혼동하지 않습니다.**

---

# 19. Green LED 하나만 먼저 테스트하기

전체 회로를 한 번에 실행하지 않습니다.

파일:

```text
scripts/led_single_test.py
```

코드:

```python
import time

from gpiozero import LED


def main():
    led = LED(17)

    try:
        print("Green LED ON")
        led.on()
        time.sleep(1.0)

        print("Green LED OFF")
        led.off()

    finally:
        led.close()


if __name__ == "__main__":
    main()
```

실행:

```bash
python -m scripts.led_single_test
```

Green LED가 약 1초 동안 켜졌다가 꺼지는지 확인합니다.

정상 동작하지 않으면 다음을 먼저 확인합니다.

```text
LED 방향
저항 연결
GND
GPIO17 배선
BCM 번호
```

---

# 20. LED 점멸 간격 바꾸기

`led_single_test.py`를 잠시 변경하여 반복합니다.

예:

```python
for _ in range(5):
    led.on()
    time.sleep(0.5)

    led.off()
    time.sleep(0.5)
```

정상 동작 후 다음 값으로 바꿔봅니다.

```text
0.5초
→ 0.2초
```

그리고 반복 횟수도 바꿉니다.

```text
5회
→ 10회
```

이 실습에서는 코드를 크게 바꾸지 않고 **한 조건씩 변경하여 실제 장치의 반응을 확인**합니다.

---

# 21. GPIO 출력 모듈 만들기

이제 Green LED, Red LED, Buzzer를 하나의 출력 모듈로 분리합니다.

파일:

```text
src/gpio_output.py
```

## 이 코드는 왜 만들까요?

LED와 Buzzer 제어 코드를 여러 실행 Program에 반복해서 작성하면 배선 번호나 ON/OFF 규칙을 바꿀 때 여러 파일을 수정해야 합니다.

그래서 출력 장치의 동작을 한 모듈로 모읍니다.

```text
normal()
→ 정상 상태 출력

warning()
→ 경고 상태 출력

off()
→ 모든 출력 끄기

close()
→ Resource 정리
```

나중에 Rule이나 AI Model이 붙어도 판단 결과가 `NORMAL`이면 `normal()`, `WARNING`이면 `warning()`을 호출하면 됩니다.

## 의사코드

```text
Green / Red / Buzzer GPIO 번호를 받는다
        ↓
Green LED와 Red LED 객체를 만든다
        ↓
Buzzer 사용 여부를 확인한다
        ↓
사용하면 Buzzer 객체를 만든다
        ↓
처음에는 모든 출력을 OFF로 만든다

normal() 호출
→ Green ON / Red OFF / Buzzer OFF

warning() 호출
→ Green OFF / Red ON / Buzzer ON(사용할 때만)

off() 호출
→ 모두 OFF

close() 호출
→ OFF 후 GPIO Resource 정리
```

코드:

```python
from gpiozero import Buzzer, LED


class GPIOOutput:
    def __init__(
        self,
        green_pin: int,
        red_pin: int,
        buzzer_pin: int,
        use_buzzer: bool = False,
    ):
        self.green = LED(green_pin)
        self.red = LED(red_pin)

        self.use_buzzer = use_buzzer

        if use_buzzer:
            self.buzzer = Buzzer(buzzer_pin)
        else:
            self.buzzer = None

        self.off()

    def normal(self):
        self.green.on()
        self.red.off()

        if self.buzzer is not None:
            self.buzzer.off()

    def warning(self):
        self.green.off()
        self.red.on()

        if self.buzzer is not None:
            self.buzzer.on()

    def off(self):
        self.green.off()
        self.red.off()

        if self.buzzer is not None:
            self.buzzer.off()

    def close(self):
        self.off()

        self.green.close()
        self.red.close()

        if self.buzzer is not None:
            self.buzzer.close()
```

역할:

```text
normal()
→ Green ON
→ Red OFF
→ Buzzer OFF

warning()
→ Green OFF
→ Red ON
→ Buzzer ON

off()
→ 모두 OFF
```

앞으로 AI가 어떤 결과를 내더라도 출력 쪽에서는 `normal()` 또는 `warning()`만 호출하도록 만들 수 있습니다.

---

# 22. GPIO 전체 Test 만들기

파일:

```text
scripts/gpio_test.py
```

## 이 코드는 왜 만들까요?

Camera와 합치기 전에 GPIO만 독립적으로 정상인지 확인하기 위한 Test Program입니다.

Camera가 없는 상태에서도 다음이 실제로 동작해야 합니다.

```text
NORMAL
→ Green

OFF
→ 모두 OFF

WARNING
→ Red + 선택 Buzzer
```

## 의사코드

```text
설정파일을 읽는다
        ↓
GPIO 번호와 Buzzer 사용 여부를 가져온다
        ↓
GPIOOutput을 만든다
        ↓
NORMAL 상태를 출력한다
        ↓
OFF 상태로 만든다
        ↓
WARNING 상태를 출력한다
        ↓
마지막에 다시 OFF
        ↓
오류가 있어도 GPIO Resource를 정리한다
```

코드:

```python
import time

from src.config_loader import load_config
from src.gpio_output import GPIOOutput


def main():
    config = load_config(
        "configs/settings.yaml"
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

    try:
        print("NORMAL")
        output.normal()
        time.sleep(2.0)

        print("OFF")
        output.off()
        time.sleep(1.0)

        print("WARNING")
        output.warning()
        time.sleep(1.0)

        print("OFF")
        output.off()

    finally:
        output.close()


if __name__ == "__main__":
    main()
```

실행:

```bash
python -m scripts.gpio_test
```

확인:

```text
NORMAL
→ Green LED

WARNING
→ Red LED + Buzzer

OFF
→ 모두 꺼짐
```

Buzzer를 사용하지 않는 경우에는 Python 코드를 직접 바꾸지 않고 `configs/settings.yaml`을 확인합니다.

```yaml
use_buzzer: false
```

사양이 확인된 Active Buzzer Module을 사용하는 경우에만 다음처럼 바꿉니다.

```yaml
use_buzzer: true
```

이렇게 하면 **장치 사용 여부를 설정파일로 관리**할 수 있습니다.


## 코드 리뷰 — `NORMAL`을 실행하면 실제로 어디까지 연결될까요?

`python -m scripts.gpio_test`의 `NORMAL` 한 줄을 역추적합니다.

```text
gpio_test.py
output.normal()
        ↓
gpio_output.py
normal()
        ↓
self.green.on()
self.red.off()
        ↓
gpiozero
        ↓
Raspberry Pi GPIO
        ↓
실제 Green LED
```

다음 항목을 직접 찾습니다.

```text
Green GPIO 번호는 어느 YAML에 있는가?
그 값을 Python이 읽는 코드는 어디인가?
NORMAL일 때 Green LED를 켜는 함수는 무엇인가?
프로그램이 끝날 때 LED를 끄고 Resource를 정리하는 코드는 어디인가?
Buzzer를 끄고 싶을 때 Python Code가 아니라 어느 설정을 바꾸는가?
```


---

# 23. Camera와 GPIO를 독립적으로 다시 확인하기

전체 기능을 연결하기 전에 각각 독립적으로 성공해야 합니다.

Camera:

```bash
python -m scripts.camera_test
```

GPIO:

```bash
python -m scripts.gpio_test
```

다음처럼 생각합니다.

```text
Camera 실패
+
GPIO 성공
→ Camera 영역 문제

Camera 성공
+
GPIO 실패
→ GPIO 영역 문제

둘 다 성공
→ 통합 단계로 이동
```

기능을 독립적으로 확인하면 통합 후 문제가 생겼을 때 원인을 찾기 쉬워집니다.

---

# 24. 첫 번째 정상 Checkpoint 남기기

현재까지 다음 기능이 독립적으로 정상이어야 합니다.

```text
USB Camera 인식
→ OpenCV Frame 읽기
→ JPG 저장

GPIO
→ Green LED
→ Red LED
→ 선택 Buzzer
```

이 상태를 Version으로 남깁니다.

## 방법 A — Raspberry Pi가 Git Repository인 경우

```bash
git status
git diff
```

`data/captures/`와 `configs/device.local.yaml`이 Git 대상에 보이지 않는지 확인합니다.

```bash
git add configs src scripts requirements.txt .gitignore
git commit -m "feat: add camera and gpio device modules"
git log --oneline -5
```

## 방법 B — 2일차에 SCP 방식으로 Source를 전달한 경우

Raspberry Pi 폴더가 Git Repository가 아니라면 여기서 억지로 Git을 새로 만들지 않습니다.

현재 정상 Source를 유지하고, 수업 마지막에 공통 Source를 PC로 가져온 뒤 PC Local Git에서 Commit합니다.

오늘 중요한 것은 **정상 상태로 돌아올 수 있는 Checkpoint를 남기는 것**입니다.

---

# 25. Camera + GPIO 작은 통합 프로그램 만들기

이제 두 기능을 처음 연결합니다.

요구사항:

```text
Camera에서 Frame 한 장 읽기
        ↓
이미지 저장 성공
        ↓
Green LED 1초
        ↓
종료

Camera 또는 저장 실패
        ↓
Red LED + Buzzer 1초
        ↓
오류 출력
```

파일:

```text
scripts/camera_gpio_demo.py
```

## 이 코드는 왜 만들까요?

지금까지 Camera와 GPIO를 각각 따로 확인했습니다.

이제 처음으로 다음 두 모듈을 하나의 실행 Program에서 연결합니다.

```text
camera_input.py
→ 실제 입력 담당

gpio_output.py
→ 실제 출력 담당

camera_gpio_demo.py
→ 둘을 연결하는 실행 시작점
```

오늘은 영상 내용을 분석하지 않습니다.

판단 기준은 아주 단순합니다.

```text
Frame 읽기 + 저장 성공
→ NORMAL 출력

Camera 열기 / Frame 읽기 / 저장 실패
→ WARNING 출력
```

## 의사코드

```text
설정을 읽는다
        ↓
GPIOOutput을 만든다
        ↓
Camera를 열 준비를 한다
        ↓
Camera를 연다
        ↓
Frame 한 장을 읽는다
        ↓
JPG로 저장한다
        ↓
성공하면 Green LED
        ↓
오류가 발생하면 오류 내용을 출력하고 Red LED / 선택 Buzzer
        ↓
마지막에는 Camera와 GPIO Resource를 모두 정리한다
```

코드:

```python
import time

from src.camera_input import (
    CameraInput,
    save_frame,
)
from src.config_loader import load_config
from src.gpio_output import GPIOOutput


def main():
    config = load_config(
        "configs/settings.yaml"
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

    camera = None

    try:
        camera = CameraInput(
            index=int(
                config["camera_index"]
            ),
            width=int(
                config["image_width"]
            ),
            height=int(
                config["image_height"]
            ),
        )

        time.sleep(1.0)

        frame = camera.read()

        path = save_frame(
            frame,
            config["capture_dir"],
            prefix="camera_gpio",
        )

        print(f"Capture Success: {path}")

        output.normal()
        time.sleep(1.0)

    except Exception as error:
        print(f"Capture Failed: {error}")

        output.warning()
        time.sleep(1.0)

    finally:
        if camera is not None:
            camera.release()

        output.close()


if __name__ == "__main__":
    main()
```

실행합니다.

```bash
python -m scripts.camera_gpio_demo
```

정상이라면:

```text
Camera Frame 읽기
→ JPG 저장
→ Green LED
```

가 이루어집니다.


## 코드 리뷰 — `Capture Success` 한 줄은 어떤 파일들이 함께 만들까요?

정상 결과가 다음처럼 나왔다고 가정합니다.

```text
Capture Success: data/captures/camera_gpio_....jpg
```

한 줄의 결과 뒤에는 여러 파일이 연결되어 있습니다.

```text
configs/settings.yaml
configs/device.local.yaml
        ↓
config_loader.py
        ↓
camera_gpio_demo.py
        ↓
camera_input.py
        ↓
save_frame()
        ↓
gpio_output.py
        ↓
Green LED
```

다음 질문을 코드에서 직접 찾습니다.

1. `camera_index`는 어느 설정파일에서 최종값을 가져오는가?
2. Camera를 실제로 여는 코드는 어느 파일에 있는가?
3. JPG 이름을 만드는 코드는 어느 함수에 있는가?
4. 성공 시 Green LED를 켜는 코드는 어느 함수인가?
5. Camera 오류를 잡는 `except`는 어느 파일에 있는가?
6. `finally`에서 정리하는 Resource는 무엇인가?

> 여러 파일이 하나의 결과를 만든다는 점이 3일차 Pipeline 이해의 핵심입니다.


---

# 26. 지금 연결된 구조 확인하기

3일차 처음으로 다음 흐름이 실제 하드웨어에서 동작합니다.

```text
[입력]

USB Camera
     ↓
Raspberry Pi
     ↓
OpenCV
     ↓
Frame 저장 성공 여부
     ↓
[상태 결정]

SUCCESS / FAILURE
     ↓
[출력]

Green LED
또는
Red LED + Buzzer
```

아직 AI는 없습니다.

그럼에도 다음 Edge 시스템의 최소 구조는 이미 경험했습니다.

```text
현실 입력
→ 프로그램 처리
→ 상태 결정
→ 현실 출력
```

---

# 27. Modify Lab 2 — NORMAL / WARNING 출력만 직접 바꾸기

`gpio_test.py`를 이용해 다음 순서를 직접 만들어 봅니다.

```text
NORMAL 3초
→ OFF 1초
→ WARNING 0.5초
→ OFF
```

그다음 다음 조건으로 바꿉니다.

```text
NORMAL 1초
→ WARNING 2초
→ OFF
```

변경할 것은 시간값뿐입니다.

```text
출력 장치
코드 구조
GPIO 번호
```

는 그대로 둡니다.

한 번에 한 변수만 바꾸는 연습입니다.

---

# 28. Modify Lab 3 — Camera 10장 + 상태 표시

`capture_series.py`를 수정하여 다음 동작을 만들어 봅니다.

```text
Frame 저장 성공
→ Green LED 짧게 점등

저장 실패
→ Red LED 점등
```

힌트:

```text
CameraInput
+
GPIOOutput
```

두 모듈을 함께 Import합니다.

한 장 저장할 때마다 Green LED를 너무 길게 켜면 촬영 속도가 느려질 수 있습니다.

예:

```text
0.1초 점등
```

정도로 시작해 직접 조절해 봅니다.

---

# 29. 일부러 실패시키기 1 — 잘못된 Camera index

`configs/device.local.yaml`의 Camera index를 임시로 변경합니다.

```yaml
camera_index: 99
```

실행:

```bash
python -m scripts.camera_test
```

오류 메시지를 확인합니다.

다음 영역으로 분류합니다.

```text
GPIO 문제
X

Python 문법 문제
X

Camera Input 문제
O
```

다시 자신의 실제 Camera index로 복구합니다.

```yaml
camera_index: <ACTUAL_CAMERA_INDEX>
```

`<ACTUAL_CAMERA_INDEX>`에는 `camera_index_probe.py`에서 실제 Frame을 읽은 번호를 넣습니다.

---

# 30. 일부러 실패시키기 2 — Camera 분리

실행 중 USB Camera를 무리하게 반복 탈착하지 않습니다.

Camera를 분리하는 실험은 다음처럼 안전하게 진행합니다.

```text
프로그램 종료
→ USB Camera 분리
→ 프로그램 실행
→ 오류 확인
→ USB Camera 다시 연결
→ Camera index 확인
→ 재실행
```

실행:

```bash
python -m scripts.camera_gpio_demo
```

Camera가 열리지 않으면 통합 프로그램은 예외를 처리하고 WARNING 출력을 수행해야 합니다.

Camera를 다시 연결한 뒤:

```bash
python -m scripts.camera_index_probe
```

로 index를 다시 확인합니다.

USB 장치를 재연결하면 index가 달라질 수도 있습니다.

---

# 31. 일부러 실패시키기 3 — 저장 폴더 삭제

촬영 폴더를 삭제합니다.

```bash
rm -rf data/captures
```

다시 실행합니다.

```bash
python -m scripts.camera_test
```

현재 `save_frame()`은 다음 코드를 가지고 있습니다.

```python
directory.mkdir(
    parents=True,
    exist_ok=True,
)
```

따라서 폴더가 없어도 자동으로 다시 만들어집니다.

확인:

```bash
ls data/captures
```

이것은 실패가 아니라 **자동 복구가 가능한 구조**입니다.

이번에는 실제 저장 오류를 경험하려면 권한이 없는 위치를 임시로 지정할 수 있습니다.

예를 들어 자신의 환경에서 쓰기 권한이 없는 경로를 사용하면 저장이 실패할 수 있습니다.

실험 후 반드시 원래 값으로 복구합니다.

```yaml
capture_dir: data/captures
```

---

# 32. 일부러 실패시키기 4 — LED 배선 하나 분리

이 실험은 **전원을 끈 상태에서만** 배선을 변경합니다.

```text
Raspberry Pi 종료
→ 전원 분리
→ Green LED 선 하나 분리
→ 전원 연결
→ 부팅
→ gpio_test.py 실행
```

프로그램에는 오류가 없는데 LED가 켜지지 않을 수 있습니다.

이 경우:

```text
Python 실행
→ 정상

GPIO Code
→ 정상

실제 출력
→ 실패
```

입니다.

즉 소프트웨어 결과만 보고 실제 하드웨어가 정상이라고 판단하면 안 됩니다.

실험 후 다시 전원을 끄고 배선을 원래대로 복구합니다.

---

# 33. 실패 원인을 영역별로 분류하기

| 증상 | 먼저 확인할 영역 |
|---|---|
| `/dev/video*`가 없음 | USB / Linux Device |
| Camera index 오류 | Camera Config |
| Frame을 읽지 못함 | Camera / Driver / 연결 |
| 이미지 저장 실패 | Path / Permission |
| LED가 안 켜짐 | Wiring / GPIO / BCM 번호 |
| Buzzer만 안 울림 | Buzzer 종류 / Wiring / GPIO |
| Python Import 오류 | venv / Package |
| Camera와 GPIO 각각 성공하지만 통합 실패 | Integration Code |

문제를 확인하는 순서:

```text
1. Hardware 연결
        ↓
2. Linux Device 확인
        ↓
3. 독립 Test
Camera / GPIO
        ↓
4. Config
        ↓
5. Python Module
        ↓
6. 통합 Program
```

---

# 34. Mini Challenge — 실제 입·출력 Pipeline을 스스로 다시 연결하기

지금까지는 안내된 순서대로 Camera와 GPIO를 연결했습니다.

이제부터는 **결과를 먼저 생각하고, 코드 위치를 찾고, 설정을 바꾸고, 일부 기능을 직접 수정하고, 실패 원인을 추적하는 방식**으로 오늘 내용을 다시 학습합니다.

새로운 기술을 배우는 시간이 아닙니다.

```text
실행 결과 역추적
        ↓
실행 전에 결과 예측
        ↓
설정만 변경
        ↓
기능 직접 조합
        ↓
일부러 오류 발생
        ↓
실제 파일 / Hardware로 성공 증명
        ↓
전체 Pipeline을 자신의 말로 설명
```

각 Challenge는 먼저 직접 해결합니다.

그 다음 **예시 정답 확인해보기**를 열어 자신의 생각과 비교합니다.

---

## Challenge 1 — 실행 결과를 보고 코드 위치 찾기

다음 결과가 나왔다고 가정합니다.

```text
=== Camera Test ===
Frame Width  : 640
Frame Height : 480
Channels     : 3
Saved        : data/captures/day03_....
```

코드를 열어 다음 표를 채웁니다.

| 결과 | 담당 파일 / 함수 |
|---|---|
| Camera index 읽기 | |
| Camera 열기 | |
| Frame 읽기 | |
| Width / Height 확인 | |
| 저장 파일명 생성 | |
| JPG 저장 | |
| Camera Resource 종료 | |

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

| 결과 | 담당 파일 / 함수 |
|---|---|
| Camera index 읽기 | `config_loader.py`로 병합된 config → `device.local.yaml` |
| Camera 열기 | `src/camera_input.py` → `CameraInput.__init__()` |
| Frame 읽기 | `src/camera_input.py` → `read()` |
| Width / Height 확인 | `scripts/camera_test.py` → `frame.shape[:2]` |
| 저장 파일명 생성 | `src/camera_input.py` → `save_frame()`의 Timestamp |
| JPG 저장 | `src/camera_input.py` → `cv2.imwrite()` |
| Camera Resource 종료 | `scripts/camera_test.py`의 `finally` → `camera.release()` |

결과 한 화면은 한 파일에서 전부 만들어지는 것이 아닙니다.

```text
설정
+ Camera Module
+ 실행 Program
= 최종 결과
```

</details>

---

## Challenge 2 — 실행 전에 결과 예측하기

현재 설정이 다음과 같다고 가정합니다.

```yaml
# settings.yaml
image_width: 640
image_height: 480
capture_dir: data/captures
green_led_bcm: 17
red_led_bcm: 27
use_buzzer: false
```

```yaml
# device.local.yaml
camera_index: 0
```

다음 질문에 실행 전에 답합니다.

1. `camera_gpio_demo.py`가 정상 Capture에 성공하면 어떤 LED가 켜질까요?
2. `use_buzzer: false`라면 WARNING에서도 Buzzer가 울릴까요?
3. `capture_dir`을 바꾸면 Python Source를 수정해야 할까요?
4. `camera_index`를 장치마다 다르게 만들고 싶다면 어느 파일을 바꾸는 것이 맞을까요?

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

1. 정상 Capture이면 `output.normal()`이 호출되므로 Green LED가 켜집니다.
2. `use_buzzer: false`이면 Buzzer 객체를 만들지 않으므로 울리지 않습니다.
3. `capture_dir`은 설정값이므로 Source를 수정하지 않아도 됩니다.
4. 장치마다 달라질 수 있는 `camera_index`는 `configs/device.local.yaml`에 둡니다.

</details>

---

## Challenge 3 — Python 코드는 그대로 두고 저장 위치만 바꾸기

Python 파일은 수정하지 않습니다.

`configs/settings.yaml`의 다음 값만 임시로 변경합니다.

```yaml
capture_dir: data/day03_challenge
```

실행 전에 예상합니다.

```text
JPG는 어느 폴더에 저장될까?
camera_input.py를 수정해야 할까?
폴더가 미리 없어도 저장될까?
```

실행합니다.

```bash
python -m scripts.camera_test
```

확인합니다.

```bash
find data/day03_challenge -maxdepth 1 -type f
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

JPG는 `data/day03_challenge` 아래에 저장됩니다.

`save_frame()` 내부에는 다음 동작이 있습니다.

```python
directory.mkdir(
    parents=True,
    exist_ok=True,
)
```

따라서 폴더가 없으면 자동으로 만듭니다.

```text
Source Code 변경 없음
        +
설정값만 변경
        ↓
실행 위치 변경
```

이것이 설정과 코드를 분리한 장점입니다.

</details>

Challenge가 끝나면 다시 복원합니다.

```yaml
capture_dir: data/captures
```

---

## Challenge 4 — 기존 모듈만 조합하여 `capture_status.py` 완성하기

다음 파일을 직접 만듭니다.

```text
scripts/capture_status.py
```

요구사항:

```text
1. 설정을 읽는다.
2. Camera Frame을 한 장 읽는다.
3. Timestamp 파일명으로 저장한다.
4. 성공하면 Green LED를 1초 켠다.
5. 실패하면 Red LED를 켠다.
6. Buzzer 사용 설정이 true인 경우에만 실패 시 Buzzer가 켜진다.
7. 프로그램이 끝날 때 Camera와 GPIO Resource를 정리한다.
```

새로운 Camera/GPIO 코드를 다시 만들지 않습니다.

다음 기존 모듈을 조합합니다.

```text
src/config_loader.py
src/camera_input.py
src/gpio_output.py
```

먼저 흐름을 의사코드로 적어 봅니다.

```text
Config
   ↓
GPIOOutput
   ↓
CameraInput
   ↓
read()
   ↓
save_frame()
   ↓
성공?
 ┌─┴─┐
YES NO
 ↓   ↓
Green Red / 선택 Buzzer
   ↓
Resource 정리
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```python
import time

from src.camera_input import CameraInput, save_frame
from src.config_loader import load_config
from src.gpio_output import GPIOOutput


def main():
    config = load_config("configs/settings.yaml")

    output = GPIOOutput(
        green_pin=int(config["green_led_bcm"]),
        red_pin=int(config["red_led_bcm"]),
        buzzer_pin=int(config["buzzer_bcm"]),
        use_buzzer=bool(config.get("use_buzzer", False)),
    )

    camera = None

    try:
        camera = CameraInput(
            index=int(config["camera_index"]),
            width=int(config["image_width"]),
            height=int(config["image_height"]),
        )

        time.sleep(1.0)

        frame = camera.read()

        path = save_frame(
            frame,
            config["capture_dir"],
            prefix="challenge",
        )

        print(f"SUCCESS: {path}")

        output.normal()
        time.sleep(1.0)

    except Exception as error:
        print(f"FAIL: {error}")

        output.warning()
        time.sleep(0.5)

    finally:
        if camera is not None:
            camera.release()

        output.close()


if __name__ == "__main__":
    main()
```

정상 Camera에서는 다음 흐름을 기대합니다.

```text
SUCCESS: ...
→ JPG 생성
→ Green LED
```

실패 상태에서는 다음 흐름을 기대합니다.

```text
FAIL: ...
→ Red LED
→ use_buzzer=true인 장비에서만 Buzzer
```

</details>

---

## Challenge 5 — 일부러 Camera 오류를 만들고 원인 찾기

현재 정상 Camera index를 먼저 기록합니다.

```text
정상 Camera index: __________
```

`configs/device.local.yaml`을 임시로 변경합니다.

```yaml
camera_index: 99
```

실행 전에 예상합니다.

```text
어느 단계에서 실패할까?
Camera Module 문제인가?
GPIO Wiring 문제인가?
어떤 출력 상태가 나타날까?
```

실행합니다.

```bash
python -m scripts.capture_status
```

오류의 마지막 부분을 읽고 다음 형식으로 적습니다.

```text
증상:
문제 영역:
확인한 파일:
복구 방법:
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

대표적으로 Camera를 열 수 없다는 오류가 발생할 수 있습니다.

```text
문제 영역
→ Camera Config / Camera Input

확인
→ configs/device.local.yaml
→ camera_index
→ camera_index_probe.py
```

통합 Program이 오류를 처리했다면:

```text
Camera 실패
→ except
→ output.warning()
→ Red LED
→ 선택 Buzzer
```

GPIO 자체가 문제인 것은 아닙니다.

</details>

실험 후 반드시 자신의 정상 index로 복구합니다.

```yaml
camera_index: <ACTUAL_CAMERA_INDEX>
```

그리고 다시 실행하여 정상 상태를 확인합니다.

---

## Challenge 6 — 프로그램 성공을 실제 결과로 증명하기

단순히 터미널에 `SUCCESS`가 보였다고 끝내지 않습니다.

정상 상태에서 다음을 실행합니다.

```bash
python -m scripts.capture_status
```

그리고 세 가지를 확인합니다.

### 1. 파일이 실제로 만들어졌는가?

```bash
find data/captures -maxdepth 1 -type f -name 'challenge_*.jpg' | tail -n 3
```

### 2. JPG 파일로 인식되는가?

방금 생성한 실제 파일 하나를 선택하여 확인합니다.

```bash
file data/captures/<실제_파일명>.jpg
```

### 3. 실제 출력 장치가 반응했는가?

```text
정상 Capture
→ Green LED 확인

실패 Test
→ Red LED 확인
→ Buzzer 사용 장비만 Buzzer 확인
```

보고서에 실제 결과를 한 문장으로 기록합니다.

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

예:

```text
정상 Camera index 0에서 capture_status.py를 실행했다.
challenge_...jpg 파일이 생성되었고 file 명령에서 JPEG image data로 확인했다.
동시에 Green LED가 켜지는 것을 실제 장치에서 확인했다.
```

중요한 것은 예시 문장을 복사하는 것이 아니라 **자신의 실제 장치 결과로 성공을 증명하는 것**입니다.

</details>

---

## Challenge 7 — 오늘의 전체 Pipeline을 자신의 말로 설명하기

다음 빈칸을 먼저 채웁니다.

```text
USB Camera는 Raspberry Pi에서 __________ 입력을 만든다.

Linux에서는 Camera가 보통 __________ 형태로 보일 수 있다.

OpenCV에서는 __________ 를 이용해 Camera를 연다.

실제 한 장의 영상 데이터는 __________ 라고 부른다.

Camera index는 장치마다 달라질 수 있으므로
우리 프로젝트에서는 __________ 에 저장한다.

정상 상태에서는 __________ LED를 켠다.

오류 상태에서는 __________ LED와
설정에 따라 __________ 를 사용할 수 있다.

Camera와 GPIO는 통합 전에 먼저 __________ Test를 한다.
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
USB Camera는 Raspberry Pi에서 실제 영상 입력을 만든다.

Linux에서는 Camera가 /dev/video* 형태로 보일 수 있다.

OpenCV에서는 VideoCapture를 이용해 Camera를 연다.

실제 한 장의 영상 데이터는 Frame이라고 부른다.

Camera index는 장치마다 달라질 수 있으므로
device.local.yaml에 저장한다.

정상 상태에서는 Green LED를 켠다.

오류 상태에서는 Red LED와
설정에 따라 Buzzer를 사용할 수 있다.

Camera와 GPIO는 통합 전에 먼저 독립 Test를 한다.
```

최종 흐름:

```text
USB Camera
   ↓
Linux Device
   ↓
OpenCV
   ↓
Frame
   ↓
Python Program
   ↓
SUCCESS / FAILURE
   ↓
GPIO
   ↓
LED / 선택 Buzzer
```

</details>

---

# 35. Mini Challenge 마무리 — 4일차에 사용할 정상 상태로 복원하기

Challenge에서는 학습을 위해 설정과 Camera 상태를 변경했습니다.

4일차에 영향을 주지 않도록 기본 상태로 정리합니다.

## 1. Camera index 복원

```yaml
# configs/device.local.yaml
camera_index: <ACTUAL_CAMERA_INDEX>
```

## 2. 저장 폴더 복원

```yaml
# configs/settings.yaml
capture_dir: data/captures
```

## 3. Buzzer 설정 확인

사양이 확인되지 않았거나 사용하지 않는 장비:

```yaml
use_buzzer: false
```

확인된 Active Buzzer Module을 사용하는 장비만:

```yaml
use_buzzer: true
```

## 4. 배선 복원

배선을 변경했다면 Raspberry Pi를 종료하고 전원을 분리한 뒤 원래대로 연결합니다.

## 5. 독립 Test 다시 확인

```bash
python -m scripts.camera_index_probe
python -m scripts.camera_test
python -m scripts.gpio_test
```

## 6. 통합 Test 확인

```bash
python -m scripts.camera_gpio_demo
python -m scripts.capture_status
```

다음이 모두 맞아야 합니다.

```text
Camera Frame
→ 정상

JPG 저장
→ 정상

Green / Red LED
→ 정상

Buzzer
→ 설정과 장비 사양에 맞게 동작

통합 Program
→ 정상
```

이제 4일차에서 사용할 Raspberry Pi가 정상 상태로 돌아왔습니다.

---

# 36. 3일차 실행 기록 만들기

파일:

```text
reports/day03_camera_gpio.md
```

다음 형식으로 작성합니다.

```md
# Day 03 Camera & GPIO

## Camera

Linux Video Device:

Camera Index:

요청 해상도:

실제 Frame 해상도:

100 Frame Read 결과:

저장 이미지:

## GPIO

Green LED BCM:

Red LED BCM:

Buzzer BCM:

NORMAL 출력 결과:

WARNING 출력 결과:

## Camera + GPIO

정상 Capture 결과:

실패 Capture 결과:

## Modify

변경한 조건:

예상:

실제 결과:

## Failure Test

### Case 1

증상:

영역:
Camera / Path / GPIO / Python / Integration

원인:

해결:

### Case 2

증상:

영역:

원인:

해결:

## 오늘 확인한 연결

Camera
→ Raspberry Pi
→ Python
→ GPIO
```

다음 항목도 이어서 기록합니다.

```md
## Mini Challenge Review

### 1. 결과 역추적
Camera Test 결과를 만든 파일과 함수:

### 2. 실행 전 예상
내 예상과 실제 결과가 같았던 부분:

### 3. 설정만 변경
변경한 설정:
예상:
실제 결과:

### 4. capture_status.py
정상 상태 결과:
실패 상태 결과:

### 5. Failure Test
증상:
문제 영역:
원인:
복구:

### 6. 성공 증명
생성된 이미지 파일:
`file` 확인 결과:
실제 LED/Buzzer 반응:

### 7. 오늘의 Pipeline 한 문장 설명
내 설명:
```

실제 결과를 직접 작성합니다.

---

# 37. README에 Day 03 실행 방법 추가하기

`README.md`에 다음 내용을 추가합니다.

```md
## Day 03 Camera & GPIO

Camera index 확인:

```bash
python -m scripts.camera_index_probe
```

Camera Test:

```bash
python -m scripts.camera_test
```

GPIO Test:

```bash
python -m scripts.gpio_test
```

Camera + GPIO:

```bash
python -m scripts.camera_gpio_demo
```

Mini Challenge:

```bash
python -m scripts.capture_status
```

Device Local Camera Index:

```text
configs/device.local.yaml
→ camera_index
→ 장치마다 실제 Probe 결과를 사용
```

현재 연결:

```text
USB Camera
→ Raspberry Pi
→ OpenCV
→ Status
→ LED / Buzzer
```
```

---

# 38. Git에 올리기 전에 촬영 데이터 확인

Git 상태를 확인합니다.

```bash
git status
```

`data/captures/`의 촬영 이미지가 나타나지 않아야 합니다.

만약 나타난다면 `.gitignore`에 다음이 있는지 확인합니다.

```gitignore
data/
```

촬영 이미지와 실습 Source Code는 구분합니다.

```text
Source Code
→ Git 관리

Camera Image
→ Git 제외

장치별 Local Config
→ Git 제외
```

---

# 39. 3일차 최종 Version 저장하기

오늘 변경한 공통 Source와 문서를 확인합니다.

```text
src/camera_input.py
src/gpio_output.py
scripts/camera_index_probe.py
scripts/camera_test.py
scripts/capture_series.py
scripts/led_single_test.py
scripts/gpio_test.py
scripts/camera_gpio_demo.py
scripts/capture_status.py
configs/settings.yaml
requirements.txt
reports/day03_camera_gpio.md
README.md
```

다음 항목은 공통 Repository에 넣지 않습니다.

```text
.venv/
configs/device.local.yaml
data/captures/
logs/
```

## 방법 A — Raspberry Pi가 내부 Remote와 연결된 Git Repository인 경우

```bash
git status
git diff
```

Stage:

```bash
git add configs src scripts reports README.md requirements.txt .gitignore
```

`configs/device.local.yaml`과 `data/`가 Stage되지 않았는지 확인합니다.

```bash
git status
```

Commit:

```bash
git commit -m "feat: connect camera input and gpio output"
git log --oneline -6
```

내부 Remote 사용이 허용된 경우:

```bash
git push
```

PC에서도 같은 Remote를 사용하는 경우:

```bash
exit
cd ~/ai_vision/subject13_edge_ai
git pull
git log --oneline -6
```

## 방법 B — Raspberry Pi에 Source만 SCP로 전달한 경우

Pi에서 수정한 공통 Source와 보고서를 PC로 가져옵니다.

파일 수가 많으므로 교육장 운영 방식에 따라 `src/`, `scripts/`, `reports/`, `README.md`, `requirements.txt`, 공통 `configs/settings.yaml`을 PC 프로젝트에 동기화합니다.

예를 들어 필요한 파일을 개별적으로 가져올 수 있습니다.

```bash
scp <RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/src/camera_input.py src/
scp <RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/src/gpio_output.py src/
scp <RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/reports/day03_camera_gpio.md reports/
```

그 다음 PC Local Git에서:

```bash
git status
git diff
git add configs src scripts reports README.md requirements.txt .gitignore
git commit -m "feat: connect camera input and gpio output"
git log --oneline -6
```

`configs/device.local.yaml`과 촬영 이미지는 가져와 Commit하지 않습니다.

> 오늘의 핵심은 특정 Git 서비스가 아니라 **공통 Source와 장치별 Local 데이터/설정을 분리하여 Version을 관리하는 것**입니다.

---

# 40. 3일차가 끝난 시점의 프로젝트 구조

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
│   └── day03_camera_gpio.md
│
├── scripts/
│   ├── camera_gpio_demo.py
│   ├── camera_index_probe.py
│   ├── camera_test.py
│   ├── capture_series.py
│   ├── check_config.py
│   ├── check_device.py
│   ├── gpio_test.py
│   ├── led_single_test.py
│   └── run_pipeline.py
│
├── src/
│   ├── camera_input.py
│   ├── config_loader.py
│   ├── device_info.py
│   ├── gpio_output.py
│   └── pipeline.py
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
└── captures/

logs/
```

Mini Challenge를 완료했다면 다음 파일도 존재합니다.

```text
scripts/capture_status.py
```

`configs/device.local.yaml`에는 자신의 실제 Camera index가 남아 있어야 합니다.

---

# 41. 오늘 만든 기능을 한 번에 다시 실행하기

Raspberry Pi에 접속한 상태에서:

```bash
cd ~/ai_vision/subject13_edge_ai
```

```bash
source .venv/bin/activate
```

Camera index:

```bash
python -m scripts.camera_index_probe
```

Camera:

```bash
python -m scripts.camera_test
```

GPIO:

```bash
python -m scripts.gpio_test
```

통합:

```bash
python -m scripts.camera_gpio_demo
```

Mini Challenge 최종 확인:

```bash
python -m scripts.capture_status
```

Git:

```bash
git status
git log --oneline -6
```

---

# 42. 3일차 최종 복습 — 실제 입력과 출력의 연결을 설명하기

먼저 답을 보지 않고 자신의 말로 답합니다.

1. 2일차와 3일차의 가장 큰 차이는 무엇인가요?
2. USB Camera는 PC가 아니라 어디에 연결했나요?
3. `/dev/video0`이 보인다고 OpenCV index `0`이라고 바로 단정하면 안 되는 이유는 무엇인가요?
4. `camera_index_probe.py`는 무엇을 확인하는 Program인가요?
5. `CameraInput`과 `camera_test.py`의 역할은 어떻게 다른가요?
6. `settings.yaml`의 Width/Height와 실제 `frame.shape`가 다를 수도 있는 이유는 무엇인가요?
7. Camera index는 왜 `device.local.yaml`에 두었나요?
8. GPIO의 BCM 번호와 물리 Pin 번호는 같은 숫자인가요?
9. LED에 저항을 사용하는 이유는 무엇인가요?
10. 배선을 변경할 때 왜 전원을 먼저 끄나요?
11. `gpio_output.py`를 따로 만든 이유는 무엇인가요?
12. Camera와 GPIO를 통합하기 전에 각각 독립 Test를 하는 이유는 무엇인가요?
13. `camera_gpio_demo.py`에서 Capture가 실패하면 어떤 흐름으로 이동하나요?
14. 촬영 이미지가 Git에 올라가지 않아야 하는 이유는 무엇인가요?
15. 4일차에는 오늘의 어떤 개념을 확장하나요?

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

1. 2일차는 Raspberry Pi 실행환경을 준비했고, 3일차는 실제 Camera 입력과 GPIO 출력을 연결했습니다.
2. USB Camera는 Raspberry Pi의 USB Port에 직접 연결합니다.
3. 한 Camera가 여러 `/dev/video*` Node를 만들 수 있고 OpenCV에서 실제 Frame을 읽을 수 있는 index가 다를 수 있기 때문입니다.
4. 여러 Camera index를 열어 실제 Frame까지 읽히는 번호를 찾습니다.
5. `CameraInput`은 재사용할 Camera 기능이고, `camera_test.py`는 그 기능을 실제로 실행해 정상 여부를 확인하는 Test Program입니다.
6. 요청한 해상도를 Camera가 그대로 지원하지 않거나 Driver가 다른 크기를 반환할 수 있기 때문입니다.
7. USB 연결 상태와 장치 환경에 따라 index가 달라질 수 있는 장치별 값이기 때문입니다.
8. 아닙니다. 오늘 코드는 BCM 번호를 기준으로 합니다.
9. LED에 과도한 전류가 흐르는 것을 줄이기 위해 사용합니다.
10. 전원이 연결된 상태에서 잘못된 Pin 접촉이나 배선 변경으로 장치에 문제가 생기는 위험을 줄이기 위해서입니다.
11. NORMAL/WARNING 출력 규칙을 한 곳에서 관리하고 여러 Program에서 재사용하기 위해서입니다.
12. 통합 후 오류가 생겼을 때 Camera 문제인지 GPIO 문제인지 범위를 빠르게 나누기 위해서입니다.
13. `except`에서 오류를 출력하고 `output.warning()`으로 이동한 뒤 마지막에 Resource를 정리합니다.
14. 촬영 데이터는 실행 중 생성되는 데이터이며 용량·보안·Version 관리 측면에서 Source와 분리해야 하기 때문입니다.
15. 시간에 따라 반복해서 들어오는 입력을 Sensor/Input 값으로 확장하고 Sampling · Timestamp · CSV 기록을 학습합니다.

</details>

---

# 43. 자가 체크리스트

수업을 마치기 전에 직접 확인합니다.

- [ ] 2일차 Raspberry Pi 환경이 정상인지 먼저 확인했다.
- [ ] PC와 Raspberry Pi Terminal을 구분할 수 있다.
- [ ] USB Camera를 Raspberry Pi에 직접 연결했다.
- [ ] `lsusb`와 `/dev/video*`로 Linux Camera 인식을 확인했다.
- [ ] `camera_index_probe.py`로 실제 Camera index를 찾았다.
- [ ] Camera index를 `device.local.yaml`에 기록했다.
- [ ] OpenCV로 Frame을 읽고 JPG를 저장했다.
- [ ] 요청 해상도와 실제 Frame 해상도를 구분할 수 있다.
- [ ] 연속 Frame을 여러 장 저장해 보았다.
- [ ] BCM 번호와 물리 Pin 번호를 구분한다.
- [ ] GPIO 배선 변경 시 전원을 끄고 작업했다.
- [ ] Green/Red LED를 독립적으로 Test했다.
- [ ] Buzzer는 사양이 확인된 장비에서만 사용했다.
- [ ] `camera_input.py`와 `gpio_output.py`의 역할을 설명할 수 있다.
- [ ] Camera와 GPIO를 각각 독립 Test한 뒤 통합했다.
- [ ] 잘못된 Camera index 오류를 만들고 다시 복구했다.
- [ ] `capture_status.py` Mini Challenge를 완료했다.
- [ ] 저장된 JPG와 실제 LED 반응으로 성공을 확인했다.
- [ ] `reports/day03_camera_gpio.md`에 실제 결과를 기록했다.
- [ ] 촬영 이미지와 Local 설정이 Git 대상에서 제외되는지 확인했다.
- [ ] 4일차 시작 전에 기본 Camera/GPIO 상태로 복원했다.

체크되지 않은 항목이 있다면 전체를 처음부터 다시 하지 않습니다.

해당 단계만 다시 확인합니다.

---

# 44. 지금까지 프로젝트가 어떻게 성장했는지 확인하기

1일차:

```text
가상 입력
→ Decision
→ Log
```

2일차:

```text
Raspberry Pi OS / Network / SSH
→ Pi 실행환경
→ 같은 Project 실행
```

3일차:

```text
USB Camera
→ Raspberry Pi
→ OpenCV
→ 실제 Frame

Raspberry Pi
→ GPIO
→ LED / Buzzer
```

따라서 지금까지 다음 구조가 실제로 만들어졌습니다.

```text
현실 입력
→ Edge Device
→ Python 처리
→ 현실 출력
```

아직 Camera 영상의 내용을 판단하지는 않습니다.

현재 판단은 다음 정도입니다.

```text
Camera 입력 성공?
→ NORMAL

Camera 입력 실패?
→ WARNING
```

5일차부터는 Camera Frame의 내용을 실제 Rule로 분석하게 됩니다.

---

# 45. 4일차와 연결하기

4일차에는 Camera 외에 **시간에 따라 반복해서 들어오는 Sensor/Input 값**을 다룹니다.

3일차에서 이미 다음을 경험했습니다.

```text
Camera
→ Frame 1
→ Frame 2
→ Frame 3
→ ...
```

즉, 실제 장치의 입력은 한 번만 들어오는 것이 아니라 시간에 따라 계속 들어옵니다.

4일차에는 이 생각을 숫자형 Sensor/Input으로 확장합니다.

```text
Sensor / Input
        ↓
일정한 간격으로 Sampling
        ↓
각 값에 Timestamp
        ↓
CSV 기록
```

연결해서 보면:

```text
1일차
가상 입력 Pipeline 이해
        ↓
2일차
Raspberry Pi 실행환경
        ↓
3일차
Camera / GPIO 실제 입·출력
        ↓
4일차
Sensor / Sampling / Timestamp / CSV
```

3일차가 끝났을 때 다음 질문에 자신의 말로 답할 수 있으면 됩니다.

> **“USB Camera의 실제 Frame이 Raspberry Pi에 들어와 Python에서 처리되고, 그 결과가 GPIO의 실제 출력으로 전달되기까지 어떤 파일과 장치가 연결되는가?”**

이 질문에 답할 수 있다면 4일차로 넘어갈 준비가 된 것입니다.
