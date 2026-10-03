
> **오늘의 핵심:** 3일차에는 USB Camera의 연속 Frame을 실제 입력으로 받아 Raspberry Pi에서 처리하고 GPIO로 출력했습니다.  
> 4일차에는 Button과 숫자형 입력을 사용하여 **시간에 따라 계속 들어오는 값을 일정한 간격으로 읽고(Sampling), 시간정보(Timestamp)를 붙여 CSV로 기록하는 흐름**을 만듭니다.
>
> 1~3일차에 사용한 `subject13_edge_ai` 프로젝트를 그대로 이어서 사용합니다. **새 프로젝트와 새 가상환경을 만들지 않습니다.**
>
> 실제 IP, Hostname, GPIO 배선, CPU Temperature 경로처럼 환경에 따라 달라질 수 있는 값은 교재의 예시를 그대로 외우지 않고 자신의 Raspberry Pi에서 확인합니다.

---

## 오늘 가장 중요한 질문

3일차에는 실제 Camera가 Raspberry Pi로 들어오고 실제 LED가 반응하는 흐름을 만들었습니다.

```text
3일차

USB Camera
    ↓
Raspberry Pi
    ↓
OpenCV
    ↓
Frame
    ↓
상태 확인
    ↓
GPIO
    ↓
LED / 선택 Buzzer
```

오늘은 Camera와 다른 형태의 입력을 다룹니다.

```text
Button
→ PRESSED / RELEASED

숫자형 입력
→ 48.2
→ 48.5
→ 48.7
→ ...
```

숫자가 한 번 들어오는 것만으로는 현장 데이터가 되기 어렵습니다.

다음 질문이 함께 필요합니다.

```text
언제 읽었는가?
        ↓
얼마나 자주 읽었는가?
        ↓
무슨 값이 들어왔는가?
        ↓
어떤 상태로 판단했는가?
        ↓
그 결과를 어디에 기록했는가?
```

따라서 오늘의 가장 중요한 질문은 다음입니다.

> **“시간에 따라 계속 들어오는 입력을 Raspberry Pi가 일정한 간격으로 읽고, 시간정보와 함께 기록하고, 기준값으로 상태를 판단하려면 어떤 흐름으로 프로그램을 구성해야 하는가?”**

오늘은 이 질문에 답할 수 있도록 다음 구조를 실제로 만듭니다.

```text
Input
   ↓
Sampling
   ↓
Timestamp
   ↓
CSV Log
   ↓
Threshold
   ↓
NORMAL / WARNING
   ↓
GPIO
   ↓
Event Log
```

5일차에는 이 구조를 다시 Camera에 연결합니다.

```text
4일차
숫자형 Input
→ Sampling
→ Threshold
→ NORMAL / WARNING
→ GPIO / Log

        ↓

5일차
Camera Frame
→ HSV / Color Mask
→ Area Threshold
→ NORMAL / WARNING
→ GPIO / Event Log
```

즉, 4일차는 **시간에 따라 들어오는 값을 읽고 기록하며 판단하는 구조**를 익히는 날이고, 5일차는 그 판단 대상을 Camera 영상으로 확장하는 날입니다.

---

## 오늘의 학습 목표

수업이 끝나면 다음을 할 수 있어야 합니다.

```text
1. Digital Input과 Numeric Input의 차이를 설명한다.
2. Button의 PRESSED / RELEASED를 Raspberry Pi에서 읽는다.
3. 숫자형 입력을 한 번 읽고 값의 출처를 확인한다.
4. Sampling Interval과 Sampling Count의 의미를 설명한다.
5. Timestamp와 elapsed_sec의 차이를 설명한다.
6. 일정 간격으로 값을 반복해서 읽어 CSV에 저장한다.
7. Sampling Log와 Event Log의 목적을 구분한다.
8. Threshold로 NORMAL / WARNING을 판단한다.
9. 3일차의 GPIOOutput을 다시 사용하여 상태를 실제 LED로 출력한다.
10. 오류가 발생했을 때 Input / Config / Sampling / Log / GPIO 영역으로 나누어 원인을 찾는다.
11. 실행 결과가 어느 파일의 어느 코드에서 만들어졌는지 역추적한다.
12. 오늘 만든 Pipeline을 자신의 말로 설명한다.
```

---

## 오늘의 8시간 학습 흐름

장비 상태와 실습 속도에 따라 실제 시간은 조금 달라질 수 있습니다.

| 학습 블록 | 권장 시간 | 무엇을 배우나요? | 끝나면 할 수 있어야 하는 것 |
|---|---:|---|---|
| 1 | 30분 | 3일차 상태 확인 · 4일차 전체 구조 | Camera/GPIO 이후 왜 Sampling을 배우는지 설명할 수 있다 |
| 2 | 50분 | Digital Input · Button · Event | Button 입력을 안전하게 연결하고 PRESSED/RELEASED를 확인할 수 있다 |
| 3 | 60분 | Numeric Input · CPU Temperature · Simulated Sensor | 숫자형 입력 Source를 바꾸어 같은 코드로 읽을 수 있다 |
| 4 | 90분 | Sampling · Timestamp · CSV Logger | 반복 입력을 시간정보와 함께 CSV에 저장할 수 있다 |
| 5 | 60분 | Sampling Log · Event Log · Threshold | 모든 Sample과 상태 변화 Event의 차이를 설명할 수 있다 |
| 6 | 50분 | 3일차 GPIO 재사용 · Sensor→GPIO 통합 | 숫자형 입력을 NORMAL/WARNING으로 판단해 LED로 출력할 수 있다 |
| 7 | 90분 | Mini Challenge | 결과 예측·코드 역추적·설정 변경·오류 해결·통합을 스스로 수행할 수 있다 |
| 8 | 50분 | 복원 · Report · README · Git · 복습 · 5일차 연결 | 오늘의 정상 상태를 보존하고 다음 날 수업으로 연결할 수 있다 |

총 480분을 기준으로 구성합니다.

---

## 오늘 사용할 실제 장비와 데이터

오늘 별도의 Dataset을 다운로드하지 않습니다.

실제 Raspberry Pi에서 다음 입력을 사용합니다.

```text
Digital Input
→ Push Button
→ PRESSED / RELEASED

Numeric Input
→ Raspberry Pi CPU Temperature
→ 숫자값

재현용 Numeric Input
→ Simulated Sensor
→ 일정 범위의 숫자값
```

사용 장비:

```text
Raspberry Pi
PC + SSH
Push Button
Breadboard
Jumper Wire

3일차에서 사용한 장치
→ Green LED
→ Red LED
→ 선택 Buzzer
```

CPU Temperature는 산업용 외부 센서의 대체품이 아닙니다.

오늘의 목적은 특정 센서 모델을 배우는 것이 아니라,

> **숫자형 입력 → Sampling → Timestamp → CSV → 판단 → 출력**

이라는 공통 구조를 확실하게 이해하는 것입니다.

---

## 오늘 배울 기술 스택

| 기술 | 오늘 어디에 사용하나요? |
|---|---|
| Raspberry Pi OS / Linux | GPIO와 장치 상태를 확인하는 실제 Edge 실행환경 |
| SSH | PC에서 Raspberry Pi Terminal에 접속 |
| Python | 입력·Sampling·Logging·상태 판단 Program 작성 |
| `gpiozero` | Button 입력과 3일차 GPIO 출력 재사용 |
| YAML | Sampling 조건과 장치 설정 관리 |
| CSV | 시간에 따라 발생한 값을 파일에 기록 |
| `datetime` | 실제 발생 시각 Timestamp 생성 |
| `time.perf_counter()` | 실행 시작 후 경과시간 측정 |
| Git | 정상 동작 Source와 실험 결과 기록 |

---

## 오늘도 계속 기억할 다섯 단계

1일차에 배운 구조는 오늘도 같습니다.

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

오늘은 다음처럼 바뀝니다.

```text
입력
→ Button / Numeric Sensor

처리
→ Sampling / Timestamp

판단
→ Threshold

출력
→ Green / Red LED / 선택 Buzzer

기록
→ Sample CSV / Event CSV
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

Button BCM               : ____________________
Green LED BCM            : ____________________
Red LED BCM              : ____________________
Buzzer 사용 여부          : 사용 / 미사용

CPU Temperature 경로     : ____________________
```

이 문서에서는 다음 표기를 사용합니다.

```text
<RPI_USER>
→ 자신의 Raspberry Pi 사용자 이름

<RPI_IP>
→ 자신의 Raspberry Pi IPv4

<ACTUAL_BUTTON_BCM>
→ 실제 배선한 Button BCM 번호

<CPU_TEMP_PATH>
→ 자신의 Raspberry Pi에서 확인한 Temperature 파일 경로
```

교재에 보이는 `GPIO23`, `/sys/class/thermal/thermal_zone0/temp` 같은 값은 **수업용 예시 또는 기본값**입니다.

실제 장비와 다르면 확인한 값을 사용합니다.

---

## 오늘의 성공 기준

수업이 끝났을 때 다음 흐름이 실제로 연결되면 됩니다.

```text
3일차 Raspberry Pi 환경 정상
        ↓
Button 입력 확인
        ↓
숫자형 입력 확인
        ↓
Sampling
        ↓
Timestamp
        ↓
Sample CSV
        ↓
Threshold
        ↓
NORMAL / WARNING
        ↓
3일차 GPIOOutput 재사용
        ↓
Green / Red LED
        ↓
초기 상태 + 상태 변화 Event CSV
        ↓
Report / README / Git
```

---

## GPIO 입력 실습 안전 규칙

오늘 Button 배선을 추가합니다.

배선을 바꿀 때는 다음 순서를 지킵니다.

```text
Raspberry Pi 종료
        ↓
전원 분리
        ↓
배선 변경
        ↓
배선 다시 확인
        ↓
전원 연결
        ↓
부팅 후 SSH 접속
```

Button 실습은 기본적으로 다음 구조를 사용합니다.

```text
GPIO
  ↓
Push Button
  ↓
GND
```

Python의 내부 Pull-up을 사용합니다.

**GPIO Pin에 5V를 직접 입력하지 않습니다.**

---

# 1. 3일차 프로젝트에서 이어서 시작하기

3일차가 끝났다면 같은 `subject13_edge_ai` 프로젝트에 Camera와 GPIO 기능이 이미 있습니다.

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

Raspberry Pi에만 존재하는 항목도 있습니다.

```text
.venv/
configs/device.local.yaml
data/
logs/
```

오늘은 **새 프로젝트를 만들지 않습니다.**

또한 3일차의 `gpio_output.py`를 다시 작성하지 않습니다.

```text
3일차에서 만든 것

camera_input.py
→ Camera 입력

gpio_output.py
→ 실제 GPIO 출력

        ↓ 그대로 재사용

4일차에서 새로 만드는 것

sensor_input.py
→ 숫자형 입력

sensor_logger.py
→ Sample CSV

event_logger.py
→ 상태 변화 Event CSV
```

---

# 2. Raspberry Pi에 접속하고 3일차 정상 상태 확인하기

PC에서 자신의 Raspberry Pi에 접속합니다.

```bash
ssh <RPI_USER>@<RPI_IP>
```

IP를 기억해서 입력하지 말고 현재 할당된 주소를 사용합니다.

`.local` 이름이 교육장 환경에서 정상 동작한다면 다음처럼 사용할 수도 있습니다.

```bash
ssh <RPI_USER>@<RPI_HOSTNAME>.local
```

`.local`이 동작하지 않는다고 Raspberry Pi가 고장난 것은 아닙니다. 실제 IPv4로 다시 확인합니다.

접속 후:

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

현재 Python을 확인합니다.

```bash
which python
```

장치 상태:

```bash
python -m scripts.check_device
```

3일차 Camera와 GPIO 파일이 있는지 확인합니다.

```bash
ls src
ls scripts
```

## Source 최신 상태 확인

2일차부터 사용한 전달 방식에 따라 다릅니다.

### 내부 GitLab/Gitea 같은 Remote를 사용하는 경우

```bash
git status
git pull
git log --oneline -6
```

### SCP 방식으로 Source를 전달한 경우

Raspberry Pi 폴더가 Git Repository가 아닐 수 있습니다.

이 경우 `git pull`을 억지로 실행하지 않습니다.

```bash
ls
ls configs
ls src
ls scripts
```

필요한 파일이 존재하는지 확인합니다.

## 3일차 기능을 한 번 다시 확인하기

GPIO:

```bash
python -m scripts.gpio_test
```

Camera:

```bash
python -m scripts.camera_test
```

두 기능이 정상이라면 4일차로 넘어갑니다.

```text
기존 기능 정상
→ 새 Sensor/Sampling 기능 추가

기존 기능부터 오류
→ 3일차 환경 먼저 복구
→ 그 다음 4일차 진행
```

---

# 3. Camera Frame과 Sensor 값은 무엇이 같을까요?

3일차 Camera는 시간에 따라 Frame이 계속 들어왔습니다.

```text
Frame 1
→ Frame 2
→ Frame 3
→ Frame 4
→ ...
```

Sensor도 같은 방식으로 생각할 수 있습니다.

```text
Value 1
→ Value 2
→ Value 3
→ Value 4
→ ...
```

데이터 모양은 다릅니다.

```text
Camera
→ 이미지 배열

Button
→ PRESSED / RELEASED

Temperature
→ 숫자

Distance
→ 숫자

Light
→ 숫자
```

하지만 프로그램 입장에서는 공통 질문이 생깁니다.

```text
입력이 들어온다
        ↓
언제 읽을까?
        ↓
몇 초 간격으로 읽을까?
        ↓
읽은 값을 어떻게 저장할까?
        ↓
어떤 기준으로 상태를 판단할까?
```

이것이 오늘 배우는 `Sampling`, `Timestamp`, `CSV`, `Threshold`입니다.

---

# 4. Digital Input과 Numeric Input 구분하기

## Digital Input

상태가 두 가지로 구분되는 입력입니다.

```text
0 / 1
OFF / ON
RELEASED / PRESSED
```

오늘은 Push Button을 사용합니다.

```text
Button
→ GPIO
→ Raspberry Pi
→ PRESSED / RELEASED
```

## Numeric Input

숫자값이 계속 변하는 입력입니다.

```text
48.2
48.5
48.4
48.9
...
```

오늘 필수 실습에서는 Raspberry Pi가 실제로 제공하는 CPU Temperature를 숫자형 입력으로 사용합니다.

외부 센서가 없어도 숫자형 데이터를 실제 Raspberry Pi에서 읽을 수 있기 때문입니다.

또한 같은 실험을 반복하기 위한 `SimulatedSensor`도 만듭니다.

```text
실제 입력
→ CPU Temperature

재현 가능한 연습 입력
→ Simulated Sensor
```

---

# 5. GPIO · I2C · SPI · UART는 어디에 사용될까요?

Raspberry Pi의 외부 입력이 모두 같은 방식으로 연결되는 것은 아닙니다.

```text
GPIO
→ 단순 Digital Input / Output

I2C
→ 주소를 이용해 여러 장치와 통신

SPI
→ Clock을 함께 사용하는 비교적 빠른 동기식 통신

UART
→ TX / RX를 이용한 직렬 통신
```

오늘은 이 통신 방식을 깊게 구현하지 않습니다.

현재 장치 파일이 보이는지만 확인할 수 있습니다.

```bash
ls /dev/i2c-* 2>/dev/null
ls /dev/spidev* 2>/dev/null
ls -l /dev/serial* 2>/dev/null
```

결과가 없어도 오늘 필수 실습에는 문제가 없습니다.

```text
오늘 필수
→ GPIO Button
→ CPU Temperature / Simulated Sensor
→ Sampling / Timestamp / CSV
```

외부 I2C·SPI·UART 센서는 모델에 따라 전압, 배선, 주소, Library가 다르므로 **정확한 센서 모델이 확인되지 않은 상태에서 임의로 배선하지 않습니다.**

---

# 6. Button 배선과 설정값 준비하기

3일차 LED/Buzzer 배선에 Button을 추가합니다.

배선을 바꾸기 전에 Raspberry Pi를 종료합니다.

```bash
sudo shutdown -h now
```

완전히 종료된 뒤 전원을 분리합니다.

수업의 기본 예시는 `GPIO23`을 사용합니다.

```text
GPIO23
  │
  └── Push Button ── GND
```

이 값이 교육장 표준 배선과 다르면 지정받은 BCM 번호를 사용합니다.

배선 후 전원을 연결하고 Raspberry Pi가 부팅되면 다시 SSH로 접속합니다.

```bash
ssh <RPI_USER>@<RPI_IP>
cd ~/ai_vision/subject13_edge_ai
source .venv/bin/activate
```

## `configs/settings.yaml`에 4일차 공통 설정 추가

3일차 설정은 삭제하지 않습니다.

기존 내용 아래에 다음 항목을 추가합니다.

```yaml
# Day 04 - Digital / Numeric Input
button_bcm: 23

sampling_interval_sec: 0.5
sampling_count: 20

sensor_source: cpu_temp

sensor_log_path: logs/day04_sensor.csv
event_log_path: logs/day04_events.csv

sensor_warning_threshold: 60.0
```

각 값의 역할:

```text
button_bcm
→ Button을 연결한 BCM GPIO 번호

sampling_interval_sec
→ 몇 초 간격으로 값을 읽을 것인가?

sampling_count
→ 몇 번 읽을 것인가?

sensor_source
→ cpu_temp 또는 simulated 중 어떤 입력을 사용할 것인가?

sensor_log_path
→ 모든 Sample을 기록할 CSV 경로

event_log_path
→ 최초 상태와 상태 변화 Event를 기록할 CSV 경로

sensor_warning_threshold
→ NORMAL / WARNING을 나누는 숫자 기준
```

`button_bcm: 23`은 수업 기본 배선입니다.

자신의 실제 배선이 다르면 그 값을 사용합니다.

---

# 7. Button을 독립적으로 테스트하기

다음 파일을 만듭니다.

```text
scripts/button_test.py
```

## 이 코드는 왜 만들까요?

전체 Sensor Pipeline을 만들기 전에 **Button이라는 Digital Input 하나만 독립적으로 정상인지 확인**하기 위해 만듭니다.

이 단계에서 확인할 것은 단순합니다.

```text
Button 누름
→ PRESSED

Button 뗌
→ RELEASED
```

Button이 여기에서 동작하지 않으면 나중의 통합 Program을 먼저 의심할 필요가 없습니다.

## 의사코드

```text
설정파일을 읽는다
        ↓
button_bcm 값을 가져온다
        ↓
내부 Pull-up을 사용하는 Button 객체를 만든다
        ↓
Button을 누르면 PRESSED를 출력한다
        ↓
Button을 떼면 RELEASED를 출력한다
        ↓
프로그램은 Event를 기다린다
        ↓
Ctrl+C가 들어오면 종료한다
        ↓
Button Resource를 정리한다
```

## 코드

```python
from signal import pause

from gpiozero import Button

from src.config_loader import load_config


def main():
    config = load_config("configs/settings.yaml")

    button = Button(
        int(config["button_bcm"]),
        pull_up=True,
        bounce_time=0.05,
    )

    print("=== Button Test ===")
    print("Button을 눌렀다가 떼어보세요.")
    print("종료: Ctrl+C")

    button.when_pressed = (
        lambda: print("PRESSED")
    )

    button.when_released = (
        lambda: print("RELEASED")
    )

    try:
        pause()

    except KeyboardInterrupt:
        print("\n종료합니다.")

    finally:
        button.close()


if __name__ == "__main__":
    main()
```

## 실행

```bash
python -m scripts.button_test
```

예상 형태:

```text
=== Button Test ===
Button을 눌렀다가 떼어보세요.
종료: Ctrl+C
PRESSED
RELEASED
PRESSED
RELEASED
```

## 결과를 어떻게 해석할까요?

```text
PRESSED / RELEASED가 번갈아 정상 출력
→ Python + GPIO Input + Wiring 기본 정상

프로그램은 실행되지만 아무 반응이 없음
→ Button Wiring / BCM 번호 우선 확인

Import 오류
→ .venv / gpiozero Package 우선 확인
```

## 코드 리뷰 — `PRESSED`는 어디에서 만들어졌을까요?

코드를 다시 열고 다음을 직접 찾습니다.

```text
GPIO 번호는 어디에서 읽는가?
→ configs/settings.yaml
→ config["button_bcm"]

Button 객체를 만드는 곳은?
→ Button(...)

PRESSED를 출력하는 곳은?
→ button.when_pressed

RELEASED를 출력하는 곳은?
→ button.when_released

종료할 때 Resource를 닫는 곳은?
→ finally
→ button.close()
```

> 화면의 결과와 실제 코드를 다시 연결해서 보는 것이 코드 리뷰의 목적입니다.

---

# 8. Button의 Event와 Sampling은 어떻게 다를까요?

방금 `button_test.py`는 상태가 바뀌는 순간 반응했습니다.

```text
Released
        ↓
Button 누름 발생
        ↓
PRESSED 출력

Pressed
        ↓
Button 뗌 발생
        ↓
RELEASED 출력
```

이것이 **Event 방식**입니다.

반대로 Sampling은 일정한 시간 간격으로 현재 상태를 확인합니다.

```text
0.0초 → RELEASED
0.2초 → RELEASED
0.4초 → PRESSED
0.6초 → PRESSED
0.8초 → RELEASED
```

두 방식의 차이는 다음과 같습니다.

| 방식 | 의미 |
|---|---|
| Event | 상태가 바뀌는 순간에 반응 |
| Sampling | 정해진 시간 간격마다 현재 값을 확인 |

어느 하나가 항상 더 좋은 것은 아닙니다.

입력의 성격과 시스템 목적에 따라 선택합니다.

---

# 9. CPU Temperature가 어디에서 들어오는지 먼저 확인하기

숫자형 입력으로 사용할 Temperature 경로가 자신의 Raspberry Pi에서 존재하는지 확인합니다.

먼저 Thermal Zone을 확인합니다.

```bash
ls -d /sys/class/thermal/thermal_zone* 2>/dev/null
```

각 Zone의 종류를 확인할 수 있습니다.

```bash
for zone in /sys/class/thermal/thermal_zone*; do
  echo -n "$zone : "
  cat "$zone/type" 2>/dev/null
done
```

예를 들어 다음처럼 보일 수 있습니다.

```text
/sys/class/thermal/thermal_zone0 : cpu-thermal
```

실제 Temperature 파일을 확인합니다.

```bash
cat /sys/class/thermal/thermal_zone0/temp
```

예:

```text
48700
```

이 값은 보통 milli-degree Celsius 형태입니다.

```text
48700
÷ 1000
= 48.7°C
```

장치나 OS 구성에 따라 Thermal Zone 번호가 다를 수 있습니다.

따라서 자신의 실제 경로를 확인합니다.

```text
내 CPU Temperature 경로:

____________________________________
```

## `configs/device.local.yaml`에 장치 전용 경로 기록

`device.local.yaml`은 2일차부터 사용하던 **현재 Raspberry Pi 전용 설정파일**입니다.

기존 `device_name`, `camera_index` 등을 지우지 않고 아래 항목을 추가합니다.

```yaml
cpu_temp_path: /sys/class/thermal/thermal_zone0/temp
```

위 경로는 예시입니다.

자신의 장치에서 확인한 실제 경로를 사용합니다.

Temperature 경로를 찾을 수 없더라도 오늘 수업을 중단할 필요는 없습니다.

```text
CPU Temperature 사용 가능
→ cpu_temp 사용

CPU Temperature 경로가 다르거나 사용할 수 없음
→ simulated 사용
```

---

# 10. 숫자형 입력 모듈 만들기

다음 파일을 만듭니다.

```text
src/sensor_input.py
```

## 이 코드는 왜 만들까요?

앞으로 실행 Program마다 Temperature 파일을 직접 읽는 코드를 반복해서 쓰지 않기 위해 만듭니다.

또한 실제 CPU Temperature와 재현용 Simulated Sensor를 **같은 `read()` 방식**으로 사용할 수 있게 만듭니다.

```text
CPUTemperatureSensor
→ read()
→ 실제 숫자값

SimulatedSensor
→ read()
→ 재현 가능한 숫자값
```

실행 Program에서는 **“어디에서 값이 왔는가?”보다 `read()`로 값을 받는 것**에 집중할 수 있습니다.

## 의사코드

```text
CPUTemperatureSensor를 만든다
        ↓
Temperature 파일 경로를 저장한다
        ↓
read()가 호출되면
        ↓
경로가 존재하는지 확인한다
        ↓
없으면 오류를 발생시킨다
        ↓
파일의 숫자를 읽는다
        ↓
1000으로 나누어 °C 값으로 바꾼다
        ↓
float 값으로 반환한다


SimulatedSensor를 만든다
        ↓
최솟값 / 최댓값 / Seed를 저장한다
        ↓
read()가 호출되면
        ↓
지정 범위 안의 숫자를 만든다
        ↓
값을 반환한다


create_sensor()를 만든다
        ↓
source가 cpu_temp이면
→ CPUTemperatureSensor 반환

source가 simulated이면
→ SimulatedSensor 반환

둘 다 아니면
→ 잘못된 Source 오류
```

## 코드

```python
import random
from pathlib import Path


class CPUTemperatureSensor:
    def __init__(self, path: str):
        self.path = Path(path)

    def read(self) -> float:
        if not self.path.exists():
            raise RuntimeError(
                "CPU Temperature 경로를 찾을 수 없습니다: "
                f"{self.path}"
            )

        raw = self.path.read_text(
            encoding="utf-8"
        ).strip()

        return float(raw) / 1000.0


class SimulatedSensor:
    def __init__(
        self,
        minimum: float = 40.0,
        maximum: float = 70.0,
        seed: int = 13,
    ):
        self.minimum = minimum
        self.maximum = maximum
        self.random = random.Random(seed)

    def read(self) -> float:
        return self.random.uniform(
            self.minimum,
            self.maximum,
        )


def create_sensor(
    source: str,
    cpu_temp_path: str | None = None,
):
    if source == "cpu_temp":
        if not cpu_temp_path:
            raise ValueError(
                "cpu_temp을 사용하려면 "
                "cpu_temp_path 설정이 필요합니다."
            )

        return CPUTemperatureSensor(
            cpu_temp_path
        )

    if source == "simulated":
        return SimulatedSensor()

    raise ValueError(
        f"지원하지 않는 sensor_source: {source}"
    )
```

## 파일 역할 확인

```text
sensor_input.py
→ 값을 읽는 역할

아직 하지 않는 것
→ CSV 저장
→ NORMAL / WARNING 판단
→ LED 제어
```

역할을 한 파일에 모두 넣지 않습니다.

---

# 11. 숫자형 입력을 한 번만 읽어보기

다음 파일을 만듭니다.

```text
scripts/sensor_once.py
```

## 이 코드는 왜 만들까요?

Sampling을 바로 시작하지 않고 먼저 **입력 Source 하나에서 숫자 한 개를 정상적으로 읽을 수 있는지** 확인하기 위해 만듭니다.

```text
한 번 읽기 성공
        ↓
반복 Sampling으로 확장
```

작은 기능부터 확인하면 오류 범위를 좁히기 쉽습니다.

## 의사코드

```text
설정파일을 읽는다
        ↓
sensor_source를 가져온다
        ↓
cpu_temp_path가 있으면 함께 가져온다
        ↓
create_sensor()로 실제 Sensor 객체를 만든다
        ↓
read()를 한 번 호출한다
        ↓
Source와 Value를 출력한다
```

## 코드

```python
from src.config_loader import load_config
from src.sensor_input import create_sensor


def main():
    config = load_config("configs/settings.yaml")

    source = config["sensor_source"]

    sensor = create_sensor(
        source=source,
        cpu_temp_path=config.get(
            "cpu_temp_path"
        ),
    )

    value = sensor.read()

    print("=== Sensor Once ===")
    print(f"Source : {source}")
    print(f"Value  : {value:.2f}")


if __name__ == "__main__":
    main()
```

## 실행

먼저 실제 CPU Temperature:

```yaml
sensor_source: cpu_temp
```

```bash
python -m scripts.sensor_once
```

예:

```text
=== Sensor Once ===
Source : cpu_temp
Value  : 48.70
```

재현용 입력으로 바꾸어 봅니다.

```yaml
sensor_source: simulated
```

다시 실행합니다.

```bash
python -m scripts.sensor_once
```

예:

```text
=== Sensor Once ===
Source : simulated
Value  : 47.77
```

## 여기서 무엇을 확인했을까요?

Python 실행 Program은 같습니다.

```text
sensor_once.py
→ 그대로

settings.yaml
sensor_source
→ 변경

create_sensor()
→ 다른 입력 객체 선택
```

즉,

> **같은 Program에서 설정값만 바꾸어 입력 Source를 교체할 수 있는 구조**

를 만든 것입니다.

## 코드 리뷰

다음 결과가 나왔다고 가정합니다.

```text
Source : simulated
Value  : 47.77
```

직접 찾습니다.

```text
"simulated" 값은 어디에 있는가?
→ settings.yaml

어떤 Sensor Class를 선택하는가?
→ create_sensor()

숫자를 실제로 만드는 함수는?
→ SimulatedSensor.read()

Value를 화면에 출력하는 파일은?
→ scripts/sensor_once.py
```

---

# 12. Sampling이란 무엇일까요?

한 번만 읽으면 다음과 같습니다.

```text
read()
→ 48.7
```

Sampling은 정해진 간격으로 반복해서 값을 읽는 것입니다.

```text
0.0초
→ 48.7

0.5초
→ 48.8

1.0초
→ 48.8

1.5초
→ 48.9
```

현재 설정:

```yaml
sampling_interval_sec: 0.5
sampling_count: 20
```

의 의미는 다음과 같습니다.

```text
약 0.5초 간격
        ×
20회 읽기
```

오늘은 Sampling을 통해 **시간에 따라 데이터가 어떻게 쌓이는지** 확인합니다.

---

# 13. Sample CSV를 저장하는 Logger 만들기

다음 파일을 만듭니다.

```text
src/sensor_logger.py
```

## 이 코드는 왜 만들까요?

`sensor_input.py`는 값을 읽는 역할만 합니다.

읽은 값을 파일에 저장하는 책임은 별도 Logger로 나눕니다.

```text
sensor_input.py
→ Input

sensor_logger.py
→ Record
```

나중에 Sensor 종류가 달라져도 CSV 저장 코드는 그대로 재사용할 수 있습니다.

## 의사코드

```text
CSV 파일 경로를 받는다
        ↓
부모 폴더가 없으면 만든다
        ↓
append()가 호출되면
        ↓
CSV 파일이 처음 만들어지는지 확인한다
        ↓
이어쓰기 방식으로 파일을 연다
        ↓
새 파일이면 Header를 먼저 기록한다
        ↓
Timestamp
Elapsed Time
Source
Value
를 한 줄로 저장한다
```

## 코드

```python
import csv
from pathlib import Path


class SensorCSVLogger:
    def __init__(self, path: str):
        self.path = Path(path)

        self.path.parent.mkdir(
            parents=True,
            exist_ok=True,
        )

    def append(
        self,
        timestamp: str,
        elapsed_sec: float,
        source: str,
        value: float,
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
                        "elapsed_sec",
                        "source",
                        "value",
                    ]
                )

            writer.writerow(
                [
                    timestamp,
                    f"{elapsed_sec:.3f}",
                    source,
                    f"{value:.3f}",
                ]
            )
```

---

# 14. Timestamp와 elapsed_sec를 구분하기

오늘 CSV에는 시간 정보를 두 종류로 기록합니다.

## Timestamp — 실제 시각

예:

```text
2026-10-06T10:15:30.125
```

질문:

```text
이 Sample은 실제로 언제 발생했는가?
```

를 알 수 있습니다.

## elapsed_sec — 프로그램 시작 후 경과시간

예:

```text
0.000
0.501
1.002
1.503
```

질문:

```text
프로그램이 시작된 뒤 몇 초쯤 지났을 때 읽은 값인가?
```

를 알 수 있습니다.

두 값의 목적은 다릅니다.

```text
timestamp
→ 달력 시각

elapsed_sec
→ 실행 시작점을 0으로 본 상대 시간
```

---

# 15. Sensor Sampling 실행 프로그램 만들기

다음 파일을 만듭니다.

```text
scripts/sensor_sampling.py
```

## 이 코드는 왜 만들까요?

지금까지 만든 다음 기능을 하나의 실행 흐름으로 연결합니다.

```text
Config
        ↓
Sensor 선택
        ↓
일정 횟수 반복
        ↓
Value 읽기
        ↓
Timestamp 만들기
        ↓
Elapsed Time 계산
        ↓
CSV 저장
        ↓
터미널 출력
```

이 파일이 오늘의 첫 번째 핵심 실행 Program입니다.

## 의사코드

```text
설정파일을 읽는다
        ↓
Sensor Source와 CPU Temperature 경로를 가져온다
        ↓
Sensor 객체를 만든다
        ↓
CSV Logger를 만든다
        ↓
Sampling Interval과 Count를 읽는다
        ↓
시작 시간을 저장한다
        ↓
설정된 Count만큼 반복한다
        ↓
Sensor 값을 읽는다
        ↓
시작 후 경과시간을 계산한다
        ↓
현재 실제 시각을 Timestamp로 만든다
        ↓
CSV에 한 줄 기록한다
        ↓
터미널에 현재 Sample을 출력한다
        ↓
설정한 Interval만큼 기다린다
        ↓
다음 Sample로 이동한다
```

## 코드

```python
import time
from datetime import datetime

from src.config_loader import load_config
from src.sensor_input import create_sensor
from src.sensor_logger import SensorCSVLogger


def main():
    config = load_config("configs/settings.yaml")

    source = config["sensor_source"]

    sensor = create_sensor(
        source=source,
        cpu_temp_path=config.get(
            "cpu_temp_path"
        ),
    )

    logger = SensorCSVLogger(
        config["sensor_log_path"]
    )

    interval = float(
        config["sampling_interval_sec"]
    )

    count = int(
        config["sampling_count"]
    )

    print("=== Sensor Sampling ===")
    print(f"Source   : {source}")
    print(f"Interval : {interval} sec")
    print(f"Count    : {count}")
    print()

    started = time.perf_counter()

    for index in range(1, count + 1):
        value = sensor.read()

        elapsed = (
            time.perf_counter() - started
        )

        timestamp = datetime.now().isoformat(
            timespec="milliseconds"
        )

        logger.append(
            timestamp=timestamp,
            elapsed_sec=elapsed,
            source=source,
            value=value,
        )

        print(
            f"{index:02d} | "
            f"{timestamp} | "
            f"{value:.2f}"
        )

        time.sleep(interval)


if __name__ == "__main__":
    main()
```

---

# 16. 실행 전에 여러 파일이 어떻게 연결되는지 먼저 확인하기

바로 실행하지 않고 지금까지 만든 파일의 역할을 확인합니다.

```text
configs/settings.yaml
→ Sampling 조건
→ Sensor Source
→ Log 경로
→ Threshold

configs/device.local.yaml
→ 현재 Raspberry Pi의 Local 값
→ cpu_temp_path

src/config_loader.py
→ 공통 + Local 설정 읽기

src/sensor_input.py
→ 실제 숫자값 생성/읽기

src/sensor_logger.py
→ Sample CSV 저장

scripts/sensor_sampling.py
→ 위 기능을 순서대로 실행
```

실제 실행 흐름:

```text
settings.yaml
device.local.yaml
        ↓
load_config()
        ↓
create_sensor()
        ↓
sensor.read()
        ↓
value
        ↓
datetime.now()
        +
time.perf_counter()
        ↓
timestamp / elapsed_sec
        ↓
SensorCSVLogger.append()
        ↓
logs/day04_sensor.csv
        ↓
터미널 출력
        ↓
sleep(interval)
        ↓
다음 Sample
```

이제 다음 질문에 답해 봅니다.

```text
Sensor 값은 어느 파일이 만드는가?

CSV 저장은 어느 파일이 담당하는가?

몇 초 간격으로 읽을지는 어디에서 정하는가?

실제 Timestamp는 어느 코드가 만드는가?

전체 반복 실행을 시작하는 파일은 무엇인가?
```

답을 확인한 뒤 실행합니다.

---

# 17. Guided Lab — Sensor Sampling 실행하기

실험 결과를 깨끗하게 보기 위해 기존 4일차 Sample Log가 있다면 먼저 지웁니다.

```bash
rm -f logs/day04_sensor.csv
```

기본 설정을 확인합니다.

```yaml
sampling_interval_sec: 0.5
sampling_count: 20
sensor_source: cpu_temp
sensor_log_path: logs/day04_sensor.csv
```

실행합니다.

```bash
python -m scripts.sensor_sampling
```

예상 형태:

```text
=== Sensor Sampling ===
Source   : cpu_temp
Interval : 0.5 sec
Count    : 20

01 | 2026-10-06T10:15:30.125 | 48.70
02 | 2026-10-06T10:15:30.626 | 48.80
03 | 2026-10-06T10:15:31.127 | 48.80
...
```

CPU Temperature를 사용할 수 없는 환경이라면:

```yaml
sensor_source: simulated
```

로 바꾸고 동일한 Program을 실행합니다.

## CSV 확인

```bash
head -n 10 logs/day04_sensor.csv
```

예:

```csv
timestamp,elapsed_sec,source,value
2026-10-06T10:15:30.125,0.000,cpu_temp,48.700
2026-10-06T10:15:30.626,0.501,cpu_temp,48.800
2026-10-06T10:15:31.127,1.002,cpu_temp,48.800
```

줄 수를 확인합니다.

```bash
wc -l logs/day04_sensor.csv
```

새 파일에서 20개의 Sample을 기록했다면 일반적으로:

```text
Header 1줄
+
Sample 20줄
=
총 21줄
```

입니다.

## 실행 결과 해석

```text
20개의 값이 화면에 출력
→ Sampling Count 정상

CSV에 Timestamp가 존재
→ 실제 발생 시각 기록 정상

elapsed_sec가 증가
→ 시간 흐름 기록 정상

source가 cpu_temp 또는 simulated
→ 실제 사용한 Input Source 확인 가능
```

---

# 18. 코드 리뷰 — 터미널 한 줄과 CSV 한 줄을 역추적하기

다음 결과가 보였다고 가정합니다.

```text
03 | 2026-10-06T10:15:31.127 | 48.80
```

CSV:

```csv
2026-10-06T10:15:31.127,1.002,cpu_temp,48.800
```

코드를 직접 열어 다음 위치를 찾습니다.

| 결과 | 확인할 파일/코드 |
|---|---|
| `48.80` | `src/sensor_input.py` → `read()` |
| `cpu_temp` | `configs/settings.yaml` → `sensor_source` |
| Timestamp | `scripts/sensor_sampling.py` → `datetime.now()` |
| `1.002` | `scripts/sensor_sampling.py` → `time.perf_counter()` |
| CSV Header/Row | `src/sensor_logger.py` → `append()` |
| 20회 반복 | `scripts/sensor_sampling.py` → `for index in range(...)` |
| 0.5초 대기 | `scripts/sensor_sampling.py` → `time.sleep(interval)` |

다음 질문에 자신의 말로 답합니다.

```text
화면에 출력되는 값과 CSV에 저장되는 값은
같은 sensor.read() 결과를 사용하는가?

Sampling Count를 10으로 바꾸면
어느 반복문의 횟수가 달라지는가?

Log 경로를 바꾸면
sensor_logger.py 코드를 수정해야 하는가?
```

마지막 질문의 핵심은 다음입니다.

```text
Log 경로
→ Config에서 변경

Logger 코드
→ 그대로 재사용
```

---

# 19. `sleep(0.5)`가 정확히 0.500초 간격을 만들까요?

CSV를 보면 다음처럼 기록될 수 있습니다.

```text
0.000
0.501
1.003
1.504
```

정확히 다음과 같지 않을 수 있습니다.

```text
0.000
0.500
1.000
1.500
```

현재의 단순한 실습 Loop에서는 한 Sample 사이에 다음 작업도 함께 일어납니다.

```text
Sensor 읽기
+
Timestamp 생성
+
CSV 쓰기
+
터미널 출력
+
Python / OS 실행시간
+
sleep(interval)
```

따라서 오늘의 `sampling_interval_sec`는 **각 반복 뒤에 기다리는 기본 간격**으로 이해합니다.

> 4일차에서는 정밀한 실시간 Scheduler를 만드는 것이 목표가 아닙니다.  
> 설정한 간격과 실제 기록된 간격이 조금 다를 수 있다는 사실을 직접 확인하는 것이 중요합니다.

---

# 20. Modify Lab — Sampling Interval만 바꾸어 비교하기

한 번에 한 변수만 바꿉니다.

`source`, `count`, Program 구조는 그대로 두고 `sampling_interval_sec`만 변경합니다.

실험 A:

```yaml
sampling_interval_sec: 0.1
sampling_count: 20
```

실험 B:

```yaml
sampling_interval_sec: 0.5
sampling_count: 20
```

실험 C:

```yaml
sampling_interval_sec: 1.0
sampling_count: 20
```

각 실험은 별도 Log로 보관하면 비교가 쉽습니다.

예:

```yaml
sensor_log_path: logs/day04_sensor_01.csv
```

```yaml
sensor_log_path: logs/day04_sensor_05.csv
```

```yaml
sensor_log_path: logs/day04_sensor_10.csv
```

실행:

```bash
python -m scripts.sensor_sampling
```

실행 전에 예상 소요시간을 적습니다.

| Interval | Count | 대략적인 예상 시간 | 실제 관찰 |
|---:|---:|---:|---|
| 0.1 sec | 20 | 약 2초 이상 | |
| 0.5 sec | 20 | 약 10초 이상 | |
| 1.0 sec | 20 | 약 20초 이상 | |

마지막 반복 뒤에도 `sleep()`이 실행되므로 실제 종료시간은 단순 계산과 조금 다를 수 있습니다.

이 실험의 핵심은 정확히 몇 초인지 외우는 것이 아닙니다.

```text
Interval 짧음
→ 같은 시간에 더 많은 값을 읽을 수 있음
→ 저장량 / 처리량 증가

Interval 김
→ 같은 시간에 읽는 값의 수가 줄어듦
→ 변화가 빠른 현상은 놓칠 수 있음
```

---

# 21. Button도 Sampling 방식으로 읽어보기

앞에서는 Button의 Event를 확인했습니다.

이번에는 같은 Button을 일정한 간격으로 확인해 봅니다.

다음 파일을 만듭니다.

```text
scripts/button_sampling.py
```

## 이 코드는 왜 만들까요?

같은 입력 장치도 **Event 방식과 Sampling 방식으로 다르게 읽을 수 있다는 차이**를 실제로 비교하기 위해 만듭니다.

## 의사코드

```text
설정파일을 읽는다
        ↓
Button 객체를 만든다
        ↓
0.2초 간격으로 50번 반복한다
        ↓
현재 Button 상태를 확인한다
        ↓
PRESSED 또는 RELEASED로 바꾼다
        ↓
Timestamp와 함께 화면에 출력한다
        ↓
0.2초 기다린다
        ↓
반복이 끝나면 Button Resource를 닫는다
```

## 코드

```python
import time
from datetime import datetime

from gpiozero import Button

from src.config_loader import load_config


def main():
    config = load_config("configs/settings.yaml")

    button = Button(
        int(config["button_bcm"]),
        pull_up=True,
        bounce_time=0.05,
    )

    interval = 0.2
    count = 50

    try:
        print("=== Button Sampling ===")

        for index in range(1, count + 1):
            pressed = button.is_pressed

            state = (
                "PRESSED"
                if pressed
                else "RELEASED"
            )

            timestamp = datetime.now().isoformat(
                timespec="milliseconds"
            )

            print(
                f"{index:02d} | "
                f"{timestamp} | "
                f"{state}"
            )

            time.sleep(interval)

    finally:
        button.close()


if __name__ == "__main__":
    main()
```

실행:

```bash
python -m scripts.button_sampling
```

실행 중 Button을 여러 번 눌렀다가 떼어봅니다.

비교:

```text
button_test.py
→ 상태 변화 Event가 발생한 순간 출력

button_sampling.py
→ 0.2초마다 현재 상태 출력
```

Button을 매우 짧게 눌렀다 떼면 Sampling 시점 사이에서 그 변화를 놓칠 수도 있습니다.

이것이 Sampling 주기가 중요한 이유 중 하나입니다.

---

# 22. Sampling Log와 Event Log를 구분하기

지금까지 만든 Sample CSV는 **모든 Sampling 결과**를 저장합니다.

```text
48.1
48.2
48.2
48.3
48.4
...
```

하지만 운영 시스템에서는 상태가 바뀐 순간을 따로 보고 싶을 수 있습니다.

```text
NORMAL
→ WARNING
→ NORMAL
```

따라서 두 종류의 기록을 구분합니다.

```text
Sampling Log
→ 모든 Sample 기록
→ 시간에 따른 값 변화 분석

Event Log
→ 최초 상태 + 이후 상태가 바뀐 순간 기록
→ 운영 상태 변화 추적
```

여기서 중요한 기술적 차이가 있습니다.

`previous_state = None`으로 시작하면 첫 Sample은 비교할 이전 상태가 없으므로 **현재의 최초 상태를 한 번 기록**합니다.

그 이후부터 상태가 바뀔 때만 Event를 추가합니다.

```text
첫 Sample NORMAL
→ 최초 상태이므로 기록

NORMAL
→ NORMAL
→ 기록하지 않음

NORMAL
→ WARNING
→ 상태 변화이므로 기록

WARNING
→ WARNING
→ 기록하지 않음

WARNING
→ NORMAL
→ 상태 변화이므로 기록
```

---

# 23. Event CSV Logger 만들기

다음 파일을 만듭니다.

```text
src/event_logger.py
```

## 이 코드는 왜 만들까요?

Sample CSV와 Event CSV는 목적이 다르기 때문에 별도의 Logger로 구분합니다.

```text
SensorCSVLogger
→ 모든 값 저장

EventCSVLogger
→ Program이 Event라고 결정한 순간만 저장
```

`EventCSVLogger` 자체가 상태 변화를 판단하는 것은 아닙니다.

판단은 실행 Program이 하고, Logger는 전달받은 Event를 파일에 기록합니다.

## 의사코드

```text
Event CSV 경로를 받는다
        ↓
부모 폴더가 없으면 만든다
        ↓
append()가 호출되면
        ↓
새 파일인지 확인한다
        ↓
새 파일이면 Header를 기록한다
        ↓
Timestamp / Source / Value / State를 기록한다
```

## 코드

```python
import csv
from pathlib import Path


class EventCSVLogger:
    def __init__(self, path: str):
        self.path = Path(path)

        self.path.parent.mkdir(
            parents=True,
            exist_ok=True,
        )

    def append(
        self,
        timestamp: str,
        source: str,
        value: float,
        state: str,
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
                        "source",
                        "value",
                        "state",
                    ]
                )

            writer.writerow(
                [
                    timestamp,
                    source,
                    f"{value:.3f}",
                    state,
                ]
            )
```

---

# 24. 숫자값을 NORMAL / WARNING으로 판단하고 Event 기록하기

다음 파일을 만듭니다.

```text
scripts/sensor_event.py
```

## 이 코드는 왜 만들까요?

지금까지는 값을 읽고 저장만 했습니다.

이제 숫자값에 운영 기준인 Threshold를 적용합니다.

```text
value < threshold
→ NORMAL

value >= threshold
→ WARNING
```

그리고 **최초 상태와 이후 상태가 바뀌는 순간만 Event CSV에 기록**합니다.

실제 CPU Temperature는 수업 중 Threshold를 넘지 않을 수 있습니다.

따라서 상태 변화를 확실하게 보기 위해 이 실습에서는 `simulated`를 사용합니다.

```yaml
sensor_source: simulated
sensor_warning_threshold: 60.0
```

## 의사코드

```text
설정을 읽는다
        ↓
Sensor를 만든다
        ↓
Event Logger를 만든다
        ↓
Interval / Count / Threshold를 읽는다
        ↓
previous_state를 None으로 시작한다
        ↓
반복하며 Sensor 값을 읽는다
        ↓
Threshold와 비교해 NORMAL / WARNING을 만든다
        ↓
현재 상태와 previous_state가 다른지 확인한다
        ↓
다르면 changed=True
        ↓
최초 상태이거나 상태가 바뀌었으므로 Event CSV에 기록한다
        ↓
previous_state를 현재 상태로 바꾼다
        ↓
같으면 Event CSV에 기록하지 않는다
        ↓
터미널에는 모든 Sample 상태를 출력한다
```

## 코드

```python
import time
from datetime import datetime

from src.config_loader import load_config
from src.event_logger import EventCSVLogger
from src.sensor_input import create_sensor


def decide_state(
    value: float,
    threshold: float,
) -> str:
    if value >= threshold:
        return "WARNING"

    return "NORMAL"


def main():
    config = load_config("configs/settings.yaml")

    source = config["sensor_source"]

    sensor = create_sensor(
        source=source,
        cpu_temp_path=config.get(
            "cpu_temp_path"
        ),
    )

    logger = EventCSVLogger(
        config["event_log_path"]
    )

    interval = float(
        config["sampling_interval_sec"]
    )

    count = int(
        config["sampling_count"]
    )

    threshold = float(
        config["sensor_warning_threshold"]
    )

    previous_state = None

    print("=== Sensor Event ===")
    print(f"Source    : {source}")
    print(f"Threshold : {threshold}")
    print()

    for index in range(1, count + 1):
        value = sensor.read()

        state = decide_state(
            value,
            threshold,
        )

        changed = (
            state != previous_state
        )

        print(
            f"{index:02d} | "
            f"{value:6.2f} | "
            f"{state:7s} | "
            f"changed={changed}"
        )

        if changed:
            timestamp = datetime.now().isoformat(
                timespec="milliseconds"
            )

            logger.append(
                timestamp=timestamp,
                source=source,
                value=value,
                state=state,
            )

            previous_state = state

        time.sleep(interval)


if __name__ == "__main__":
    main()
```

---

# 25. Guided Lab — Event 실행과 CSV 확인

기존 Event Log를 지웁니다.

```bash
rm -f logs/day04_events.csv
```

설정:

```yaml
sensor_source: simulated
sampling_interval_sec: 0.5
sampling_count: 20
sensor_warning_threshold: 60.0
event_log_path: logs/day04_events.csv
```

실행:

```bash
python -m scripts.sensor_event
```

예상 형태:

```text
01 |  47.77 | NORMAL  | changed=True
02 |  60.xx | WARNING | changed=True
03 |  60.xx | WARNING | changed=False
04 |  55.xx | NORMAL  | changed=True
...
```

실제 숫자는 실행 조건에 따라 달라질 수 있습니다.

Event CSV:

```bash
cat logs/day04_events.csv
```

예:

```csv
timestamp,source,value,state
2026-10-06T14:20:00.125,simulated,47.770,NORMAL
2026-10-06T14:20:01.128,simulated,66.562,WARNING
2026-10-06T14:20:02.631,simulated,45.943,NORMAL
```

Sampling Count가 20이라고 Event Row도 반드시 20개가 되는 것은 아닙니다.

```text
Sample
→ 매번 발생

Event
→ 최초 상태 + 상태가 바뀐 순간만 발생
```

## 코드 리뷰 — `changed=False`인데 왜 CSV가 늘어나지 않을까요?

다음 코드를 찾습니다.

```python
if changed:
    logger.append(...)
    previous_state = state
```

따라서:

```text
changed=True
→ Event CSV 기록

changed=False
→ Event CSV 기록 안 함
```

`EventCSVLogger`가 상태 변화를 스스로 찾는 것이 아니라, `sensor_event.py`가 상태 변화를 판단한 뒤 Logger를 호출한다는 점도 확인합니다.

---

# 26. 3일차 GPIOOutput을 다시 사용하기

오늘 새로운 LED Class를 만들지 않습니다.

3일차에서 이미 다음 기능을 만들었습니다.

```text
src/gpio_output.py

normal()
→ Green ON
→ Red OFF
→ Buzzer OFF

warning()
→ Green OFF
→ Red ON
→ 선택 Buzzer ON

close()
→ 출력 OFF
→ GPIO Resource 정리
```

오늘은 숫자형 Sensor 상태를 이 출력에 연결합니다.

다음 파일을 만듭니다.

```text
scripts/sensor_gpio_demo.py
```

## 이 코드는 왜 만들까요?

4일차에 새로 만든 Sensor 입력과 3일차의 실제 출력을 연결하기 위해 만듭니다.

```text
Sensor
→ Threshold
→ NORMAL / WARNING
→ GPIOOutput
```

즉, 이전 수업의 코드를 버리지 않고 재사용합니다.

## 의사코드

```text
설정을 읽는다
        ↓
Sensor 객체를 만든다
        ↓
3일차 GPIOOutput을 만든다
        ↓
Interval / Count / Threshold를 읽는다
        ↓
반복하며 Sensor 값을 읽는다
        ↓
Threshold로 상태를 판단한다
        ↓
WARNING이면 output.warning()
        ↓
NORMAL이면 output.normal()
        ↓
현재 Value와 State를 출력한다
        ↓
반복 종료 또는 오류 시 GPIO Resource를 정리한다
```

## 코드

```python
import time

from src.config_loader import load_config
from src.gpio_output import GPIOOutput
from src.sensor_input import create_sensor


def decide_state(
    value: float,
    threshold: float,
) -> str:
    if value >= threshold:
        return "WARNING"

    return "NORMAL"


def main():
    config = load_config("configs/settings.yaml")

    source = config["sensor_source"]

    sensor = create_sensor(
        source=source,
        cpu_temp_path=config.get(
            "cpu_temp_path"
        ),
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
            config.get("use_buzzer", False)
        ),
    )

    interval = float(
        config["sampling_interval_sec"]
    )

    count = int(
        config["sampling_count"]
    )

    threshold = float(
        config["sensor_warning_threshold"]
    )

    try:
        print("=== Sensor GPIO Demo ===")

        for index in range(1, count + 1):
            value = sensor.read()

            state = decide_state(
                value,
                threshold,
            )

            if state == "WARNING":
                output.warning()
            else:
                output.normal()

            print(
                f"{index:02d} | "
                f"{value:6.2f} | "
                f"{state}"
            )

            time.sleep(interval)

    finally:
        output.close()


if __name__ == "__main__":
    main()
```

설정:

```yaml
sensor_source: simulated
sensor_warning_threshold: 60.0
sampling_interval_sec: 0.5
sampling_count: 20
```

실행:

```bash
python -m scripts.sensor_gpio_demo
```

동작:

```text
Value < 60
→ NORMAL
→ Green LED

Value >= 60
→ WARNING
→ Red LED
→ Buzzer는 use_buzzer=true일 때만
```

Buzzer 사양이 확인되지 않았다면 계속 다음 값을 유지합니다.

```yaml
use_buzzer: false
```

---

# 27. 지금까지 만든 4일차 전체 Pipeline 확인하기

오늘은 다음 연결을 만들었습니다.

```text
settings.yaml
device.local.yaml
        ↓
config_loader.py
        ↓
sensor_input.py
        ↓
Sensor Value
        ↓
Sampling
        ↓
Timestamp
        ↓
sensor_logger.py
        ↓
Sample CSV

Sensor Value
        ↓
Threshold
        ↓
NORMAL / WARNING
        │
        ├── gpio_output.py
        │       ↓
        │   LED / 선택 Buzzer
        │
        └── 상태 변화 확인
                ↓
          event_logger.py
                ↓
          Event CSV
```

3일차와 비교하면 다음처럼 성장했습니다.

```text
3일차

Camera
→ Frame
→ 입력 성공/실패
→ GPIO

4일차

Numeric Input
→ Sampling
→ Timestamp
→ Threshold
→ NORMAL / WARNING
→ GPIO
→ CSV
```

5일차에는 이 판단 구조를 Camera Frame 내용 분석으로 바꿉니다.

---

# 28. 오류가 나면 전체 코드를 다시 쓰지 않습니다

4일차에는 입력, 설정, Log, GPIO가 함께 있기 때문에 오류를 영역별로 나누어 봅니다.

| 증상 | 먼저 확인할 영역 |
|---|---|
| Button 반응이 없음 | Wiring / BCM / GPIO |
| `gpiozero` Import 오류 | `.venv` / Package |
| CPU Temperature 경로 오류 | `device.local.yaml` / OS Path |
| `sensor_source: unknown` | Config |
| `sampling_interval_sec: abc` | Config Value Type |
| CSV가 생성되지 않음 | Log Path / Permission / Logger |
| CSV Header만 있고 값이 없음 | Sensor read / 실행 중 오류 |
| Sampling은 되는데 Event가 거의 없음 | Threshold / Source / State 변화 |
| LED가 반응하지 않음 | 3일차 GPIO / Wiring / Config |
| Python은 정상인데 실제 Button/LED가 안 됨 | Hardware Wiring 우선 확인 |

문제를 찾는 기본 순서:

```text
1. 지금 어느 장치인가?
        ↓
2. 어느 .venv인가?
        ↓
3. Input Source가 정상인가?
        ↓
4. Config 값이 정상인가?
        ↓
5. 독립 Program이 정상인가?
        ↓
6. CSV가 정상인가?
        ↓
7. Decision이 정상인가?
        ↓
8. GPIO가 정상인가?
        ↓
9. 통합 Program을 확인한다
```

---

# 29. 오류 연습 — 잘못된 Sensor Source

`settings.yaml`을 잠시 다음처럼 변경합니다.

```yaml
sensor_source: unknown
```

실행:

```bash
python -m scripts.sensor_once
```

예:

```text
ValueError: 지원하지 않는 sensor_source: unknown
```

바로 코드를 다시 쓰지 않습니다.

```text
오류 마지막 줄 읽기
        ↓
unknown이라는 값 확인
        ↓
어느 설정에서 왔는지 찾기
        ↓
settings.yaml 확인
        ↓
수정
        ↓
재실행
```

복원:

```yaml
sensor_source: cpu_temp
```

또는 실험 목적에 따라:

```yaml
sensor_source: simulated
```

---

# 30. 오류 연습 — 잘못된 Sampling Interval

잠시 다음처럼 바꿉니다.

```yaml
sampling_interval_sec: abc
```

실행:

```bash
python -m scripts.sensor_sampling
```

`float()` 변환 과정에서 오류가 발생할 수 있습니다.

이 오류는 Sensor 고장이 아닙니다.

```text
"abc"를 숫자로 바꿀 수 없음
→ Config Value Type 문제
```

복원:

```yaml
sampling_interval_sec: 0.5
```

재실행합니다.

```bash
python -m scripts.sensor_sampling
```

---

# 31. 자동 복구 확인 — Log 폴더를 삭제하면?

다음 명령으로 Runtime Log 폴더를 잠시 삭제합니다.

```bash
rm -rf logs
```

다시 실행합니다.

```bash
python -m scripts.sensor_sampling
```

현재 `SensorCSVLogger`는 다음 역할을 합니다.

```python
self.path.parent.mkdir(
    parents=True,
    exist_ok=True,
)
```

따라서 필요한 `logs/` 폴더가 자동으로 다시 생성됩니다.

확인:

```bash
ls logs
```

이 경우는 프로그램이 자동으로 처리할 수 있는 상황입니다.

```text
폴더 없음
→ Logger가 생성
→ Program 계속 실행
```

---

# 32. Mini Challenge — 4일차 Pipeline을 스스로 다시 이해하기

지금까지는 안내된 순서대로 실습했습니다.

이제 다음 순서로 스스로 확인합니다.

```text
실행 결과 보기
        ↓
코드 위치 찾기
        ↓
실행 전에 결과 예측하기
        ↓
설정값만 바꾸어 비교하기
        ↓
기존 기능 일부 수정하기
        ↓
일부러 오류 만들기
        ↓
CSV / 실제 LED로 정상 동작 증명하기
        ↓
전체 Pipeline을 자신의 말로 설명하기
```

각 Challenge에서는 먼저 직접 생각하고 실행합니다.

그 뒤의 **예시 정답 확인해보기**를 열어 자신의 결과와 비교합니다.

---

## Challenge 1 — 실행 결과를 보고 코드 위치 찾기

다음 결과가 나왔다고 가정합니다.

```text
07 |  66.56 | WARNING | changed=True
```

그리고 Event CSV에 다음 행이 추가되었습니다.

```csv
2026-10-06T14:20:01.128,simulated,66.562,WARNING
```

코드를 열어 다음 표를 직접 채웁니다.

| 확인할 것 | 담당 파일 | 코드/함수 |
|---|---|---|
| `66.56` 숫자의 출처 | | |
| `WARNING` 판단 | | |
| `changed=True` 판단 | | |
| Timestamp 생성 | | |
| Event CSV 저장 | | |
| `simulated` Source 선택 | | |

정답을 보기 전에 실제 파일을 열어 찾습니다.

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

| 확인할 것 | 담당 파일 | 코드/함수 |
|---|---|---|
| `66.56` 숫자의 출처 | `src/sensor_input.py` | `SimulatedSensor.read()` |
| `WARNING` 판단 | `scripts/sensor_event.py` | `decide_state()` |
| `changed=True` 판단 | `scripts/sensor_event.py` | `state != previous_state` |
| Timestamp 생성 | `scripts/sensor_event.py` | `datetime.now().isoformat(...)` |
| Event CSV 저장 | `src/event_logger.py` | `EventCSVLogger.append()` |
| `simulated` Source 선택 | `configs/settings.yaml` + `src/sensor_input.py` | `sensor_source` → `create_sensor()` |

한 줄의 결과가 하나의 파일에서 전부 만들어지는 것이 아닙니다.

```text
Config
+
Input Module
+
Decision
+
Event 판단
+
Logger
=
최종 결과
```

</details>

---

## Challenge 2 — 실행 전에 결과를 먼저 예측하기

다음 조건을 가정합니다.

```text
Threshold = 60.0

입력 순서
55.0
62.0
65.0
58.0
57.0
```

아직 실행하지 말고 먼저 표를 채웁니다.

| 순서 | Value | State | changed | Event 기록 여부 |
|---:|---:|---|---|---|
| 1 | 55.0 | | | |
| 2 | 62.0 | | | |
| 3 | 65.0 | | | |
| 4 | 58.0 | | | |
| 5 | 57.0 | | | |

`previous_state`가 처음에는 `None`이라는 점을 생각합니다.

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

| 순서 | Value | State | changed | Event 기록 여부 |
|---:|---:|---|---|---|
| 1 | 55.0 | NORMAL | True | O — 최초 상태 |
| 2 | 62.0 | WARNING | True | O |
| 3 | 65.0 | WARNING | False | X |
| 4 | 58.0 | NORMAL | True | O |
| 5 | 57.0 | NORMAL | False | X |

핵심:

```text
모든 Sample
→ State는 계산

상태가 그대로
→ Event는 중복 기록하지 않음

첫 Sample
→ 비교할 이전 상태가 없으므로 최초 상태로 기록
```

</details>

---

## Challenge 3 — Python 코드는 그대로 두고 설정값만 바꾸기

이번 Challenge에서는 다음 Python 파일을 수정하지 않습니다.

```text
sensor_input.py
sensor_logger.py
sensor_sampling.py
```

`configs/settings.yaml`의 값만 다음처럼 바꿉니다.

```yaml
sensor_source: simulated
sampling_interval_sec: 0.25
sampling_count: 8
sensor_log_path: logs/day04_challenge.csv
```

실행 전에 예상합니다.

```text
몇 개의 Sample이 출력될까?

Sample 사이의 기본 대기시간은?

어떤 Source가 표시될까?

어느 CSV 파일이 만들어질까?

새 CSV의 전체 줄 수는 몇 줄일까?
```

기존 Challenge CSV가 있다면 먼저 삭제합니다.

```bash
rm -f logs/day04_challenge.csv
```

실행:

```bash
python -m scripts.sensor_sampling
```

확인:

```bash
wc -l logs/day04_challenge.csv
head logs/day04_challenge.csv
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

예상:

```text
Sample
→ 8개

기본 대기
→ 0.25초

Source
→ simulated

Log
→ logs/day04_challenge.csv

새 파일 전체 줄 수
→ Header 1 + Sample 8 = 9줄
```

Python 코드를 수정하지 않아도 Config를 통해 실행 조건이 바뀝니다.

```text
같은 Program
+
다른 Config
=
다른 실행 조건
```

</details>

---

## Challenge 4 — 기존 모듈을 조합하여 `input_monitor.py` 완성하기

이번에는 안내 코드를 먼저 보지 않습니다.

다음 파일을 직접 만듭니다.

```text
scripts/input_monitor.py
```

요구사항:

```text
1. 설정파일에서 Sensor Source를 읽는다.
2. 설정된 Interval과 Count를 사용한다.
3. 모든 Sample을 Timestamp와 함께 Sample CSV에 저장한다.
4. Threshold로 NORMAL / WARNING을 판정한다.
5. 최초 상태와 상태가 바뀌는 순간만 Event CSV에 저장한다.
6. NORMAL이면 Green LED를 켠다.
7. WARNING이면 Red LED를 켠다.
8. Buzzer는 use_buzzer 설정을 따른다.
9. 종료 시 GPIO Resource를 정리한다.
```

새 기능을 처음부터 다시 만들지 않습니다.

다음 기존 모듈을 사용합니다.

```text
src/config_loader.py
src/sensor_input.py
src/sensor_logger.py
src/event_logger.py
src/gpio_output.py
```

먼저 흐름을 종이에 적습니다.

```text
Config
  ↓
Sensor
  ↓
read()
  ↓
Timestamp
  ↓
Sample CSV
  ↓
Threshold
  ↓
NORMAL / WARNING
  │
  ├─ GPIO
  │
  └─ changed?
       ↓ YES
     Event CSV
```

직접 작성하고 실행한 뒤 예시 정답을 확인합니다.

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```python
import time
from datetime import datetime

from src.config_loader import load_config
from src.event_logger import EventCSVLogger
from src.gpio_output import GPIOOutput
from src.sensor_input import create_sensor
from src.sensor_logger import SensorCSVLogger


def decide_state(
    value: float,
    threshold: float,
) -> str:
    if value >= threshold:
        return "WARNING"

    return "NORMAL"


def main():
    config = load_config("configs/settings.yaml")

    source = config["sensor_source"]

    sensor = create_sensor(
        source=source,
        cpu_temp_path=config.get(
            "cpu_temp_path"
        ),
    )

    sample_logger = SensorCSVLogger(
        config["sensor_log_path"]
    )

    event_logger = EventCSVLogger(
        config["event_log_path"]
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
            config.get("use_buzzer", False)
        ),
    )

    interval = float(
        config["sampling_interval_sec"]
    )

    count = int(
        config["sampling_count"]
    )

    threshold = float(
        config["sensor_warning_threshold"]
    )

    previous_state = None
    started = time.perf_counter()

    try:
        print("=== Day 04 Input Monitor ===")

        for index in range(1, count + 1):
            value = sensor.read()

            elapsed = (
                time.perf_counter() - started
            )

            timestamp = datetime.now().isoformat(
                timespec="milliseconds"
            )

            state = decide_state(
                value,
                threshold,
            )

            sample_logger.append(
                timestamp=timestamp,
                elapsed_sec=elapsed,
                source=source,
                value=value,
            )

            if state == "WARNING":
                output.warning()
            else:
                output.normal()

            changed = (
                state != previous_state
            )

            if changed:
                event_logger.append(
                    timestamp=timestamp,
                    source=source,
                    value=value,
                    state=state,
                )

                previous_state = state

            print(
                f"{index:02d} | "
                f"{value:6.2f} | "
                f"{state:7s} | "
                f"changed={changed}"
            )

            time.sleep(interval)

    finally:
        output.close()


if __name__ == "__main__":
    main()
```

핵심은 새 알고리즘이 아닙니다.

기존에 만든 역할별 모듈을 연결하여 하나의 Pipeline을 완성한 것입니다.

</details>

---

## Challenge 5 — 일부러 오류를 만들고 원인 찾기

`settings.yaml`에서 `sampling_count`의 이름을 잠시 틀리게 작성합니다.

정상:

```yaml
sampling_count: 20
```

오류:

```yaml
sampling_coun: 20
```

실행:

```bash
python -m scripts.input_monitor
```

바로 정답을 보지 말고 다음 순서로 확인합니다.

```text
오류의 마지막 줄
        ↓
어떤 Key를 찾는가?
        ↓
그 Key를 사용하는 Python 파일
        ↓
settings.yaml의 실제 Key
        ↓
비교
        ↓
수정
        ↓
재실행
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

다음과 비슷한 오류가 발생할 수 있습니다.

```text
KeyError: 'sampling_count'
```

`input_monitor.py`는 다음 값을 찾습니다.

```python
config["sampling_count"]
```

하지만 설정파일에는 다음처럼 오타가 있습니다.

```yaml
sampling_coun: 20
```

따라서 다시:

```yaml
sampling_count: 20
```

으로 수정합니다.

이 오류는 Sensor Hardware 고장이 아닙니다.

```text
Config Key 불일치
→ KeyError
```

수정 후 같은 Program을 다시 실행하여 정상 동작을 확인합니다.

</details>

---

## Challenge 6 — Log와 실제 장치로 정상 동작을 증명하기

이번 Challenge에서는 **“실행됐어요”라고 말하는 것으로 끝내지 않습니다.**

다음 조건으로 실행합니다.

```yaml
sensor_source: simulated
sampling_interval_sec: 0.2
sampling_count: 10
sensor_warning_threshold: 60.0

sensor_log_path: logs/day04_monitor_samples.csv
event_log_path: logs/day04_monitor_events.csv
```

기존 파일을 삭제합니다.

```bash
rm -f logs/day04_monitor_samples.csv
rm -f logs/day04_monitor_events.csv
```

실행:

```bash
python -m scripts.input_monitor
```

다음 네 가지 증거를 확인합니다.

```text
1. 터미널
→ 10개 Sample 출력

2. Sample CSV
→ Header + 10개 Sample

3. Event CSV
→ 최초 상태 + 상태 변화만 기록

4. 실제 GPIO
→ NORMAL / WARNING에 따라 LED 변화
```

명령:

```bash
wc -l logs/day04_monitor_samples.csv
cat logs/day04_monitor_events.csv
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

Sample CSV가 새 파일이라면:

```text
Header 1
+
Sample 10
=
총 11줄
```

Event CSV 줄 수는 고정되지 않습니다.

입력값에 따라 상태 변화 횟수가 달라지기 때문입니다.

```text
Sampling Log
→ 모든 10개 값이 있어야 함

Event Log
→ 최초 상태 + 변화 Event만 있어야 함
```

또한 LED가 실제 상태에 따라 바뀌어야 합니다.

```text
Software 출력만 정상
+
실제 LED 반응 없음
→ GPIO / Wiring을 별도로 확인
```

즉, 파일과 실제 Hardware를 함께 확인해야 End-to-End 동작을 증명할 수 있습니다.

</details>

---

## Challenge 7 — 오늘의 전체 Pipeline을 자신의 말로 설명하기

코드를 닫고 다음 빈칸을 자신의 말로 채웁니다.

```text
1. sensor_source는 __________________________________

2. sensor_input.py는 _________________________________

3. Sampling은 ______________________________________

4. Timestamp는 _____________________________________

5. sensor_logger.py는 _______________________________

6. Threshold는 _____________________________________

7. Event Log는 _____________________________________

8. gpio_output.py는 _________________________________
```

마지막에는 다음 흐름을 1분 안에 설명해 봅니다.

```text
Input
→ Sampling
→ Timestamp
→ Sample Log
→ Threshold
→ NORMAL / WARNING
→ GPIO
→ Event Log
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

예시:

```text
sensor_source
→ 어떤 숫자형 입력을 사용할지 정하는 설정값

sensor_input.py
→ 실제 또는 simulated 숫자값을 읽는 모듈

Sampling
→ 정해진 간격으로 반복해서 입력을 읽는 것

Timestamp
→ Sample이 발생한 실제 시각

sensor_logger.py
→ 모든 Sample을 CSV로 저장

Threshold
→ 숫자값을 운영 상태로 바꾸는 기준

Event Log
→ 최초 상태와 이후 상태가 바뀐 순간을 기록

gpio_output.py
→ NORMAL/WARNING 상태를 실제 LED/Buzzer 출력으로 바꾸는 3일차 모듈
```

정답 문장을 그대로 외우는 것이 아니라 **각 파일이 왜 필요한지 자신의 말로 설명할 수 있으면 됩니다.**

</details>

---

# 33. Mini Challenge가 끝나면 5일차용 기본 상태로 복원하기

Challenge에서는 여러 설정값과 Log 경로를 바꾸었습니다.

5일차에서 혼동하지 않도록 기본 상태를 정리합니다.

## `configs/settings.yaml`

4일차 기본값을 다음처럼 확인합니다.

```yaml
button_bcm: 23

sampling_interval_sec: 0.5
sampling_count: 20

sensor_source: cpu_temp

sensor_log_path: logs/day04_sensor.csv
event_log_path: logs/day04_events.csv

sensor_warning_threshold: 60.0
```

`button_bcm`은 실제 수업 배선이 다른 경우 자신의 정상값을 유지합니다.

CPU Temperature를 사용할 수 없는 장비라면 다음 상태를 기본값으로 유지해도 됩니다.

```yaml
sensor_source: simulated
```

3일차 설정도 정상 상태인지 확인합니다.

```yaml
image_width: 640
image_height: 480
capture_dir: data/captures

green_led_bcm: 17
red_led_bcm: 27
buzzer_bcm: 22
use_buzzer: false
```

실제 장비의 표준 배선이 다르면 확인된 정상값을 사용합니다.

## `configs/device.local.yaml`

다음 값은 자신의 실제 환경값을 유지합니다.

```yaml
device_name: <내 Raspberry Pi 이름>
camera_index: <실제 Camera index>
cpu_temp_path: <실제 CPU Temperature 경로>
```

`device.local.yaml`은 Git 대상이 아닙니다.

## 기본 실행 확인

```bash
python -m scripts.sensor_once
python -m scripts.sensor_sampling
python -m scripts.sensor_event
python -m scripts.sensor_gpio_demo
```

`input_monitor.py`까지 완성했다면:

```bash
python -m scripts.input_monitor
```

각 Program이 정상 실행되는지 확인합니다.

---

# 34. 4일차 실습 결과 기록하기

다음 파일을 완성합니다.

```text
reports/day04_sensor_sampling.md
```

내용:

```md
# Day 04 Sensor / Sampling / Timestamp / CSV

## 1. Digital Input

Button BCM:

PRESSED 확인:

RELEASED 확인:

Event와 Sampling의 차이:

## 2. Numeric Input

기본 Sensor Source:

CPU Temperature Path:

첫 측정값:

Simulated Sensor 실행 결과:

## 3. Sampling Experiment

| Interval | Count | 예상 소요시간 | 실제 관찰 | 특징 |
|---:|---:|---:|---:|---|
| 0.1 sec | 20 | | | |
| 0.5 sec | 20 | | | |
| 1.0 sec | 20 | | | |

## 4. Timestamp

timestamp의 의미:

elapsed_sec의 의미:

실제 간격이 설정값과 정확히 같지 않을 수 있는 이유:

## 5. Sampling Log

파일:

Sample 수:

CSV 전체 줄 수:

확인한 점:

## 6. Event Log

Threshold:

최초 상태:

상태 변화 Event 수:

Sampling Log보다 행이 적을 수 있는 이유:

## 7. Sensor → GPIO

NORMAL:

WARNING:

Buzzer 사용 여부:

## 8. Failure Test

### Case 1

증상:

오류 영역:

원인:

수정:

재실행 결과:

### Case 2

증상:

오류 영역:

원인:

수정:

재실행 결과:

## 9. Mini Challenge

input_monitor.py 실행 결과:

Sample CSV 증거:

Event CSV 증거:

실제 GPIO 증거:

## 10. 오늘의 Pipeline

Input
→ Sampling
→ Timestamp
→ Sample CSV
→ Threshold
→ NORMAL / WARNING
→ GPIO
→ Event CSV

내 설명:
```

실제 실행 결과를 기준으로 작성합니다.

---

# 35. README에 Day 04 실행 방법 추가하기

`README.md`에 다음 내용을 추가합니다.

````md
## Day 04 Sensor / Sampling

Button Event Test:

```bash
python -m scripts.button_test
```

Button Sampling:

```bash
python -m scripts.button_sampling
```

Sensor 1회 읽기:

```bash
python -m scripts.sensor_once
```

Sensor Sampling:

```bash
python -m scripts.sensor_sampling
```

Sensor Event:

```bash
python -m scripts.sensor_event
```

Sensor + GPIO:

```bash
python -m scripts.sensor_gpio_demo
```

Mini Challenge:

```bash
python -m scripts.input_monitor
```

현재 Pipeline:

```text
Numeric Input
→ Sampling
→ Timestamp
→ Sample CSV
→ Threshold
→ NORMAL / WARNING
→ GPIO
→ Event CSV
```
````

---

# 36. Git에 올리기 전에 Source와 Runtime 결과를 구분하기

상태를 확인합니다.

```bash
git status
```

다음 Runtime 파일은 Git에 올리지 않습니다.

```text
logs/day04_sensor.csv
logs/day04_events.csv
logs/day04_challenge.csv
logs/day04_monitor_samples.csv
logs/day04_monitor_events.csv
```

1일차부터 `logs/`가 `.gitignore`에 포함되어 있어야 합니다.

장치 전용 설정도 Git에 올리지 않습니다.

```text
configs/device.local.yaml
```

반면 다음은 Source 또는 결과 문서이므로 Git으로 관리합니다.

```text
configs/settings.yaml

src/sensor_input.py
src/sensor_logger.py
src/event_logger.py

scripts/button_test.py
scripts/button_sampling.py
scripts/sensor_once.py
scripts/sensor_sampling.py
scripts/sensor_event.py
scripts/sensor_gpio_demo.py
scripts/input_monitor.py

reports/day04_sensor_sampling.md
README.md
```

---

# 37. Git Checkpoint 만들기

Raspberry Pi 자체가 Git Repository인 경우:

```bash
git status
git diff
```

Source Stage:

```bash
git add \
  configs/settings.yaml \
  src/sensor_input.py \
  src/sensor_logger.py \
  src/event_logger.py \
  scripts/button_test.py \
  scripts/button_sampling.py \
  scripts/sensor_once.py \
  scripts/sensor_sampling.py \
  scripts/sensor_event.py \
  scripts/sensor_gpio_demo.py \
  scripts/input_monitor.py
```

Commit:

```bash
git commit -m "feat: add sensor sampling and event pipeline"
```

Report와 README:

```bash
git add README.md reports/day04_sensor_sampling.md
git commit -m "docs: record day04 sampling experiments"
```

확인:

```bash
git log --oneline -8
```

허용된 내부 Remote가 있다면:

```bash
git push
```

외부 GitHub 사용이 제한된 환경에서는 **Local Git 또는 허용된 내부 GitLab/Gitea**를 사용합니다.

## SCP 방식으로 Source를 관리하는 경우

2일차부터 Raspberry Pi에는 실행용 Source만 SCP로 전달하고 PC의 Local Git을 기준으로 관리했다면, Raspberry Pi에서 억지로 새 Git Repository를 만들지 않습니다.

수업에서 사용한 편집 방식에 따라 **정상 Source를 PC 프로젝트로 되돌린 뒤 PC Local Git에서 Commit**합니다.

중요한 원칙은 다음입니다.

```text
Source Code / Report
→ Version 관리

Runtime Log / 장치 Local 설정
→ Git 제외
```

---

# 38. 4일차가 끝난 시점의 프로젝트 구조

공통 Source는 다음과 비슷합니다.

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
│   ├── input_monitor.py
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

---

# 39. 오늘 만든 기능을 한 번에 다시 확인하기

Raspberry Pi에서:

```bash
cd ~/ai_vision/subject13_edge_ai
source .venv/bin/activate
```

장치:

```bash
python -m scripts.check_device
```

Button:

```bash
python -m scripts.button_test
```

Sensor 한 번:

```bash
python -m scripts.sensor_once
```

Sampling:

```bash
python -m scripts.sensor_sampling
```

Event:

```bash
python -m scripts.sensor_event
```

GPIO:

```bash
python -m scripts.sensor_gpio_demo
```

Mini Challenge:

```bash
python -m scripts.input_monitor
```

Log:

```bash
ls logs
head logs/day04_sensor.csv
cat logs/day04_events.csv
```

Git Repository를 사용하는 경우:

```bash
git status
git log --oneline -8
```

---

# 40. 핵심 복습 문제

정답을 보기 전에 먼저 자신의 말로 답합니다.

1. 3일차 Camera Frame과 4일차 Sensor Value의 공통점은 무엇인가요?
2. Digital Input과 Numeric Input의 차이는 무엇인가요?
3. Button Event와 Button Sampling은 어떻게 다른가요?
4. `sensor_source`는 어느 파일에 있으며 무엇을 결정하나요?
5. `cpu_temp_path`는 왜 `device.local.yaml`에 두었나요?
6. `sensor_input.py`의 역할은 무엇인가요?
7. `sensor_logger.py`와 `event_logger.py`는 어떻게 다른가요?
8. Sampling Interval이 짧아지면 어떤 변화가 생길 수 있나요?
9. `timestamp`와 `elapsed_sec`는 어떻게 다른가요?
10. `time.sleep(0.5)`를 사용했는데 실제 간격이 정확히 0.500초가 아닐 수 있는 이유는 무엇인가요?
11. Sample 20개를 기록한 새 CSV가 21줄이 될 수 있는 이유는 무엇인가요?
12. Event CSV의 행 수가 Sampling 수보다 적을 수 있는 이유는 무엇인가요?
13. 첫 Sample도 Event CSV에 기록되는 이유는 무엇인가요?
14. 4일차에서 3일차의 어떤 파일을 다시 사용했나요?
15. `sensor_source: unknown` 오류는 어느 영역을 먼저 확인해야 하나요?
16. `sampling_interval_sec: abc`는 Hardware 오류인가요?
17. Buzzer 사양이 확인되지 않았다면 어떤 설정을 사용하나요?
18. `input_monitor.py`가 사용하는 다섯 개의 핵심 모듈은 무엇인가요?
19. Runtime CSV와 Report를 Git에서 다르게 관리하는 이유는 무엇인가요?
20. 5일차에서는 오늘 만든 구조 중 무엇을 다시 사용하게 되나요?

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

1. 둘 다 **시간에 따라 계속 들어오는 입력 데이터**입니다.
2. Digital Input은 PRESSED/RELEASED처럼 이산적인 상태이고 Numeric Input은 Temperature처럼 연속적인 숫자값입니다.
3. Event는 상태가 바뀌는 순간 반응하고, Sampling은 정해진 시간마다 현재 상태를 확인합니다.
4. `configs/settings.yaml`에 있으며 `cpu_temp` 또는 `simulated` 중 어떤 입력을 사용할지 정합니다.
5. Raspberry Pi와 OS 환경에 따라 실제 Thermal Zone 경로가 달라질 수 있는 장치 전용 값이기 때문입니다.
6. 실제 또는 simulated 숫자형 입력을 `read()`로 제공하는 역할입니다.
7. `sensor_logger.py`는 모든 Sample을 기록하고 `event_logger.py`는 실행 Program이 Event라고 판단한 순간만 기록합니다.
8. 같은 시간에 더 많은 값을 얻을 수 있지만 저장량과 처리량이 증가할 수 있습니다.
9. `timestamp`는 실제 시각이고 `elapsed_sec`는 Program 시작 이후 경과시간입니다.
10. Sensor 읽기, CSV 쓰기, 출력, Python 실행, OS Scheduling 시간 등이 함께 포함되기 때문입니다.
11. Header 1줄 + 실제 Sample 20줄이기 때문입니다.
12. Event는 모든 Sample이 아니라 최초 상태와 상태 변화가 있을 때만 기록하기 때문입니다.
13. `previous_state`가 처음에는 `None`이라 현재 첫 상태와 다르므로 최초 상태를 한 번 기록합니다.
14. `src/gpio_output.py`를 그대로 재사용했습니다.
15. `settings.yaml`의 Config 값을 먼저 확인합니다.
16. 아닙니다. 문자열 `abc`를 `float`로 변환할 수 없는 Config Value Type 문제입니다.
17. `use_buzzer: false`를 유지합니다.
18. `config_loader.py`, `sensor_input.py`, `sensor_logger.py`, `event_logger.py`, `gpio_output.py`입니다.
19. Runtime Log는 실행할 때 계속 생성되는 데이터이고 Report는 실험의 해석과 결과를 남기는 문서이기 때문입니다.
20. `입력 → 판단 → GPIO → Event Log` 구조를 재사용하고, 숫자형 Sensor 대신 Camera Frame에서 측정한 `red_area`를 판단값으로 사용하게 됩니다.

</details>

---

# 41. 자가 체크리스트

다음 항목을 직접 확인합니다.

- [ ] 3일차 `subject13_edge_ai` 프로젝트를 그대로 이어서 사용했다.
- [ ] 새 `.venv`를 만들지 않고 Raspberry Pi의 기존 `.venv`를 사용했다.
- [ ] Button 배선을 변경할 때 Raspberry Pi 전원을 안전하게 끄고 작업했다.
- [ ] `button_test.py`에서 PRESSED / RELEASED를 확인했다.
- [ ] 실제 CPU Temperature 경로를 확인하거나 `simulated` 대체 방법을 사용할 수 있다.
- [ ] `sensor_once.py`에서 숫자값 한 개를 정상적으로 읽었다.
- [ ] `sensor_sampling.py`에서 설정된 횟수만큼 Sample을 읽었다.
- [ ] Timestamp와 elapsed_sec를 구분할 수 있다.
- [ ] Sample CSV에 Header와 실제 값이 기록되는 것을 확인했다.
- [ ] Sampling Interval을 한 번에 하나씩 변경해 비교했다.
- [ ] Event 방식과 Sampling 방식의 차이를 설명할 수 있다.
- [ ] Event CSV가 최초 상태와 상태 변화만 기록한다는 것을 확인했다.
- [ ] 3일차 `GPIOOutput`을 새로 만들지 않고 재사용했다.
- [ ] `sensor_gpio_demo.py`에서 NORMAL / WARNING에 따라 LED가 반응했다.
- [ ] Buzzer 사양이 확인되지 않았다면 `use_buzzer: false`를 유지했다.
- [ ] 오류가 났을 때 Hardware / Config / Python / Log / Integration 영역을 구분해 확인했다.
- [ ] Mini Challenge에서 실행 결과의 코드 위치를 직접 찾았다.
- [ ] Mini Challenge에서 실행 전에 결과를 예측했다.
- [ ] Mini Challenge에서 Config만 바꾸어 결과를 비교했다.
- [ ] `input_monitor.py`를 기존 모듈을 조합하여 완성했다.
- [ ] 일부러 오류를 만들고 원인을 찾은 뒤 정상 상태로 복원했다.
- [ ] Sample CSV와 Event CSV를 이용해 정상 동작을 증명했다.
- [ ] 오늘의 전체 Pipeline을 자신의 말로 설명할 수 있다.
- [ ] Challenge 설정을 5일차에 사용할 기본 상태로 복원했다.
- [ ] `reports/day04_sensor_sampling.md`를 완성했다.
- [ ] README에 Day 04 실행 방법을 기록했다.
- [ ] Runtime Log와 `device.local.yaml`이 Git 대상이 아닌지 확인했다.
- [ ] 정상 Source와 Report를 Git 또는 교육장에서 허용된 Version 관리 방식으로 보존했다.

---

# 42. 1~4일차 시스템이 어떻게 성장했는지 확인하기

1일차:

```text
가상 숫자
→ Decision
→ Log
```

2일차:

```text
PC
→ Raspberry Pi OS / Network / SSH
→ 같은 Project를 Raspberry Pi에서 실행
```

3일차:

```text
USB Camera
→ Raspberry Pi
→ OpenCV
→ GPIO
→ 실제 LED
```

4일차:

```text
Button / Numeric Input
→ Sampling
→ Timestamp
→ CSV
→ Threshold
→ NORMAL / WARNING
→ GPIO
→ Event Log
```

이제 프로젝트는 다음 기본 요소를 실제로 갖게 되었습니다.

```text
Input
→ Time
→ Processing
→ Decision
→ Output
→ Record
```

---

# 43. 5일차와 연결하기

4일차까지는 숫자형 값을 기준으로 상태를 판단했습니다.

```text
Numeric Value
     ↓
Threshold
     ↓
NORMAL / WARNING
     ↓
GPIO
     ↓
Event Log
```

5일차에서는 3일차 Camera와 4일차 판단·기록 구조가 합쳐집니다.

```text
USB Camera
     ↓
CameraInput
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
     │
     └─ Event Log
```

4일차에서 배운 핵심은 그대로 유지됩니다.

```text
4일차

Sensor Value
→ Threshold
→ State

5일차

Red Area
→ Threshold
→ State
```

판단에 사용하는 **측정값의 종류만 달라지는 것**입니다.

따라서 5일차를 시작하기 전에 다음 질문에 자신의 말로 답할 수 있으면 됩니다.

> **“시간에 따라 들어오는 입력값을 Sampling하고 Timestamp와 함께 기록한 뒤, Threshold로 운영 상태를 판단하고 GPIO와 Event Log로 연결하는 과정은 어떤 순서로 동작하는가?”**

이 질문에 답할 수 있다면 5일차의 Rule 기반 Edge Warning System으로 넘어갈 준비가 된 것입니다.
