# 1. 오늘 실습의 전체 흐름

교과 13에서는 앞으로 **PC에서 개발한 프로그램과 AI 모델을 Raspberry Pi 같은 Edge 장치로 옮겨 실제 입력을 받고, 판단하고, 결과를 출력하는 과정**을 배웁니다.

처음부터 Raspberry Pi, Camera, GPIO, ONNX 모델을 한꺼번에 연결하면 전체 구조를 이해하기 어렵습니다.

그래서 1일차에는 실제 장치를 연결하기 전에 PC에서 아주 작은 프로그램을 만들어 **Edge 시스템의 기본 흐름**부터 이해합니다.

## 먼저 Edge AI와 Cloud 방식을 간단히 구분합니다

AI가 실행되는 위치에 따라 크게 다음처럼 생각할 수 있습니다.

```text
Cloud 방식

Camera / Sensor
        ↓
Network
        ↓
외부 Server
        ↓
AI 처리
        ↓
결과 전달
```

```text
Edge 방식

Camera / Sensor
        ↓
현장 가까이에 있는 Edge 장치
        ↓
AI 처리
        ↓
결과 출력
```

Cloud 방식은 데이터를 Network를 통해 Server로 보내 처리할 수 있고, Edge 방식은 데이터가 발생하는 현장 가까이의 장치에서 직접 처리할 수 있습니다.

산업현장에서는 빠른 반응이 필요하거나 Network 상태와 관계없이 장치를 계속 운영해야 하는 경우가 있기 때문에 Edge 방식이 사용될 수 있습니다.

> 1일차에서는 Cloud Server를 만들거나 Cloud와 Edge의 속도를 비교하는 실험을 하지 않습니다.

Cloud와 Edge의 차이는 왜 Edge AI를 배우는지 이해하기 위한 개념으로만 확인합니다.

교과 13의 실제 실습은 Edge AI를 중심으로 진행합니다.

![[Pasted image 20260910131601.png]]

---

![[Group 19.png]]

## 오늘은 PC에서 가장 작은 Edge Pipeline을 만들어 봅니다

오늘은 실제 Camera 대신 `0.0 ~ 1.0` 사이의 가상 입력값을 사용합니다.

```text
가상의 입력값
        ↓
입력 처리
        ↓
기준값으로 상태 판단
        ↓
NORMAL / WARNING
        ↓
터미널 출력
        ↓
CSV Log 기록
        ↓
Latency 측정
```

지금 사용하는 가상 입력은 앞으로 수업이 진행되면서 실제 입력과 AI 추론으로 바뀝니다.

```text
1일차
가상의 숫자
        ↓
3~5일차
Camera / Sensor / Rule
        ↓
6~7일차
AI Model / ONNX
        ↓
Raspberry Pi에서 실제 Edge AI 실행
```

따라서 오늘의 목표는 완성된 Edge AI를 만드는 것이 아니라 **앞으로 계속 확장할 프로그램의 가장 작은 뼈대를 먼저 이해하고 실행해 보는 것**입니다.

오늘 실습은 다음 순서로 진행합니다.

![[Pasted image 20260910132835.png]]

```text
프로젝트 폴더 생성
→ Python 가상환경 생성
→ 최소 프로젝트 구조 구성
→ 설정파일 작성
→ Edge Pipeline Simulator 작성
→ 가상 입력 생성
→ NORMAL / WARNING 판단
→ 터미널 출력
→ CSV Log 기록
→ 처리시간 확인
→ Threshold 변경 실험
→ 실습 결과 기록
→ Git Commit
→ 내부 Remote 저장소가 있으면 연결
→ Push
```

1일차가 끝나면 다음 질문에 답할 수 있으면 됩니다.

> **“입력이 들어오고, 프로그램이 처리하고, 상태를 판단하고, 결과를 출력하고 기록하는 Edge Pipeline은 어떤 순서로 동작하는가?”**

2일차부터는 오늘 만든 프로젝트를 **Raspberry Pi라는 실제 Edge 장치에서 실행**하기 시작합니다.

---

## 오늘의 8시간 학습 흐름

오늘은 코드를 많이 만드는 것보다 **Edge Pipeline이 어떻게 연결되어 동작하는지 이해하는 것**이 더 중요합니다.

수업은 다음 8개의 학습 블록으로 진행합니다.

| 학습 블록 | 무엇을 배우나요? | 끝나면 할 수 있어야 하는 것 |
|---|---|---|
| 1 | Edge AI와 Edge Pipeline 이해 | 입력 → 처리 → 판단 → 출력 → 기록의 순서를 말할 수 있다 |
| 2 | 프로젝트 폴더·가상환경·Git 준비 | 내가 어디에서 어떤 Python으로 실행하는지 확인할 수 있다 |
| 3 | `settings.yaml`과 설정 읽기 | 코드와 설정값을 분리하는 이유를 설명할 수 있다 |
| 4 | `pipeline.py` 작성 | 입력 생성·전처리·판단·로그 함수의 역할을 구분할 수 있다 |
| 5 | `run_pipeline.py` 실행 | 여러 파일이 하나의 프로그램으로 연결되는 흐름을 찾을 수 있다 |
| 6 | 실행 결과·CSV Log·Threshold 실험 | 설정값 하나가 결과를 어떻게 바꾸는지 확인할 수 있다 |
| 7 | Mini Challenge | 요구사항을 보고 Pipeline을 직접 수정하고 오류를 찾아볼 수 있다 |
| 8 | 복습·결과 기록·Git 저장 | 오늘 만든 구조를 자신의 말로 설명하고 다음 수업과 연결할 수 있다 |

### 오늘 계속 기억할 다섯 단어

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

오늘 작성하는 모든 코드는 결국 이 다섯 단계 중 하나의 역할을 합니다.

코드를 작성하다가 흐름이 헷갈리면 다음 질문으로 돌아옵니다.

> **“지금 보고 있는 코드는 입력·처리·판단·출력·기록 중 어떤 역할을 하고 있는가?”**

---

# 2. 실습을 시작할 위치 확인

이 교재의 명령은 **WSL2 Ubuntu 또는 Linux 터미널**을 기준으로 작성합니다.

터미널을 열고 현재 위치를 확인합니다.

```bash
pwd
```

현재 사용자를 확인합니다.

```bash
whoami
```

홈 디렉토리의 내용을 확인합니다.

```bash
ls
```

예를 들어 다음과 같이 보일 수 있습니다.

```text
/home/user
```

중요한 것은 출력 결과를 그대로 외우는 것이 아니라 다음을 구분하는 것입니다.

```text
pwd
→ 지금 내가 어느 폴더에서 작업하고 있는가?

whoami
→ 현재 어떤 사용자 계정으로 작업하고 있는가?

ls
→ 현재 폴더 안에 무엇이 있는가?
```

앞으로 파일이 없거나 경로 오류가 발생했을 때 가장 먼저 확인할 정보입니다.

---

# 3. 교과 13 프로젝트 폴더 만들기

먼저 AI 실습을 모아둘 상위 폴더로 이동합니다.

```bash
cd ~/ai_vision
```

교과 13 프로젝트 폴더를 만듭니다.

```bash
mkdir subject13_edge_ai
```

프로젝트 폴더로 이동합니다.

```bash
cd subject13_edge_ai
```

현재 위치를 다시 확인합니다.

```bash
pwd
```

그리고 그 폴더로 이동합니다.

```bash
code -r .
```

앞으로 특별한 설명이 없다면 명령은 이 **프로젝트 루트 폴더**에서 실행합니다.

---

# 4. Git 저장소를 먼저 시작하기

오늘부터 코드가 계속 바뀌기 때문에 프로젝트를 만든 직후 Git 저장소를 시작합니다.

```bash
git init
```

기본 Branch 이름을 `main`으로 통일합니다.

```bash
git branch -M main
```

현재 상태를 확인합니다.

```bash
git status
```

---

# 5. Python 가상환경 만들기

프로젝트마다 필요한 Python Package 버전이 달라질 수 있습니다.

따라서 시스템 Python에 모든 Package를 설치하지 않고 프로젝트 전용 환경을 만듭니다.

먼저 Python 버전을 확인합니다.

```bash
python3 --version
```

Python 위치도 확인합니다.

```bash
which python3
```

가상환경을 생성합니다.

```bash
python3 -m venv .venv
```

생성 여부를 확인합니다.

```bash
ls -a
```

다음 폴더가 보이면 됩니다.

```text
.venv
```

가상환경을 활성화합니다.

```bash
source .venv/bin/activate
```

터미널 앞부분에 다음과 같은 표시가 나타나는지 확인합니다.

```text
(.venv)
```

Python 위치를 다시 확인합니다.

```bash
which python
```

예:

```text
/home/user/ai_vision/subject13_edge_ai/.venv/bin/python
```

가상환경 활성화 전과 후의 차이를 비교합니다.

```text
활성화 전
/usr/bin/python3

활성화 후
subject13_edge_ai/.venv/bin/python
```

즉, 이제부터 설치하는 Python Package는 이 프로젝트의 `.venv` 안에서 관리됩니다.

---

# 6. 필요한 Package 설치

1일차에는 복잡한 AI Package를 설치하지 않습니다.

설정파일을 읽기 위해 `PyYAML` 하나만 사용합니다.

```bash
python -m pip install --upgrade pip
```

```bash
python -m pip install pyyaml
```

설치 여부를 확인합니다.

```bash
python -c "import yaml; print(yaml.__version__)"
```

현재 Package 목록을 `requirements.txt`에 기록합니다.

```bash
python -m pip freeze > requirements.txt
```

확인합니다.

```bash
cat requirements.txt
```

`requirements.txt`는 다른 사람이 같은 프로젝트를 실행할 때 필요한 Python Package를 다시 설치하기 위한 파일입니다.

```text
현재 PC
pip freeze
        ↓
requirements.txt
        ↓
다른 PC
pip install -r requirements.txt
```

---

# 7. 오늘 필요한 최소 폴더만 만들기

13일 뒤의 최종 폴더 구조를 처음부터 모두 만들지 않습니다.

**오늘 실제로 사용하는 폴더만** 만듭니다.

```bash
mkdir -p configs src scripts logs reports
```

현재 구조를 확인합니다.

```bash
find . -maxdepth 2 -type d | sort
```

다음과 같은 구조가 만들어집니다.

```text
subject13_edge_ai/
├── .venv/
├── configs/
├── logs/
├── reports/
├── scripts/
└── src/
```

각 폴더는 다음 역할을 합니다.

| 폴더 | 오늘의 역할 |
|---|---|
| `configs/` | 실행 중 바꿀 수 있는 설정값 저장 |
| `src/` | 프로그램의 핵심 처리 기능 작성 |
| `scripts/` | 사용자가 직접 실행할 시작 프로그램 |
| `logs/` | 실행 결과와 측정값 저장 |
| `reports/` | 실험 결과와 해석 기록 |

앞으로 Camera가 필요해지면 Camera 파일을 추가하고, ONNX가 필요해지면 모델 관련 파일을 추가합니다.

즉, 프로젝트 구조도 수업이 진행되면서 성장합니다.

## 왜 파일과 폴더를 나누어서 만들까요?

하나의 파일에 설정, 실행, 판단, Log 저장 코드를 모두 작성할 수도 있습니다.

하지만 프로그램이 커질수록 다음처럼 역할을 나누는 편이 이해하고 수정하기 쉽습니다.

```text
설정값
→ configs/

핵심 기능
→ src/

실행 시작점
→ scripts/

실행 결과
→ logs/

실험 정리
→ reports/
```

중요한 점은 **파일 이름 자체가 Python의 필수 규칙은 아니라는 것**입니다.

```text
Django 같은 Framework
→ 정해진 구조와 규칙이 동작에 직접 영향을 줄 수 있음

오늘의 Edge Pipeline
→ 우리가 역할을 이해하기 쉽게 나누어 만든 프로젝트 구조
```

따라서 오늘은 파일명을 외우는 것보다 다음 관계를 이해합니다.

> **“설정은 어디에 있고, 핵심 기능은 어디에 있으며, 어떤 파일을 실행하면 전체 기능이 연결되는가?”**

---

# 8. `.gitignore` 만들기

프로젝트 루트에 `.gitignore` 파일을 만듭니다.

```gitignore
# Python virtual environment
.venv/

# Python cache
__pycache__/
*.pyc

# IDE
.vscode/
.idea/

# Local environment / secret
.env

# Runtime logs
logs/

# 앞으로 추가될 데이터
data/
datasets/

# 앞으로 추가될 AI 모델
*.pt
*.pth
*.onnx
*.engine

# 학습 결과
runs/
weights/
checkpoints/
```

저장한 뒤 Git 상태를 확인합니다.

```bash
git status
```

`.venv/`와 `logs/`가 Git 대상에 나타나지 않아야 합니다.

이 단계에서 기억할 것은 하나입니다.

> 실행환경·데이터·모델·로그와 소스코드는 구분해서 관리합니다.

---

# 9. 첫 번째 설정파일 만들기

다음 파일을 만듭니다.

```text
configs/settings.yaml
```

내용:

```yaml
device_name: pc-day01

sample_count: 20
interval_ms: 100

warning_threshold: 0.70

log_path: logs/day01_events.csv
```

각 값의 역할은 실제 실행하면서 확인합니다.

```text
device_name
→ 현재 실행환경을 구분하기 위한 이름

sample_count
→ 몇 개의 입력을 처리할 것인가?

interval_ms
→ 입력이 들어오는 간격

warning_threshold
→ WARNING으로 판단할 기준

log_path
→ 실행 결과를 저장할 위치
```

설정값을 Python 코드 안에 직접 고정하지 않고 별도 파일로 두면 실험할 때 코드를 계속 수정하지 않아도 됩니다.

---

# 10. 설정파일을 읽는 코드 만들기

다음 파일을 만듭니다.

```text
src/config_loader.py
```

코드:

```python
from pathlib import Path

import yaml


def load_config(path: str) -> dict:
    config_path = Path(path)

    if not config_path.exists():
        raise FileNotFoundError(
            f"설정파일을 찾을 수 없습니다: {config_path}"
        )

    with config_path.open("r", encoding="utf-8") as file:
        config = yaml.safe_load(file)

    return config
```

의사코드

```
설정파일 경로를 입력받는다
        ↓
입력받은 문자열 경로를 Path 객체로 변환한다
        ↓
설정파일이 실제로 존재하는지 확인한다
        ↓
파일이 없으면
    "설정파일을 찾을 수 없습니다" 오류를 발생시킨다
        ↓
파일이 있으면
    UTF-8 방식으로 설정파일을 연다
        ↓
YAML 내용을 읽어서
Python Dictionary 형태로 변환한다
        ↓
변환된 설정값을 반환한다
```

이 파일의 역할은 단순합니다.

```text
settings.yaml
        ↓
config_loader.py
        ↓
Python Dictionary
```

핵심 프로그램이 YAML 파일을 직접 읽는 코드까지 모두 가지고 있지 않도록 역할을 분리한 것입니다.

---

# 11. 첫 실행 프로그램 만들기

다음 파일을 만듭니다.

```text
scripts/check_config.py
```

코드:

```python
from src.config_loader import load_config


def main():
    config = load_config("configs/settings.yaml")

    print("=== Day 01 Config Check ===")
    print(f"Device            : {config['device_name']}")
    print(f"Sample Count      : {config['sample_count']}")
    print(f"Warning Threshold : {config['warning_threshold']}")


if __name__ == "__main__":
    main()
```

의사코드

```
config_loader에서
설정파일을 읽는 load_config 함수를 가져온다
        ↓
main 함수를 시작한다
        ↓
configs/settings.yaml 파일을 읽는다
        ↓
읽어온 설정값을 config에 저장한다
        ↓
"Day 01 Config Check" 제목을 출력한다
        ↓
config에서 device_name 값을 꺼내 출력한다
        ↓
config에서 sample_count 값을 꺼내 출력한다
        ↓
config에서 warning_threshold 값을 꺼내 출력한다
        ↓
이 파일이 직접 실행된 경우
main 함수를 실행한다
```

프로젝트 루트에서 실행합니다.

```bash
python -m scripts.check_config
```

예상 결과:

```text
=== Day 01 Config Check ===
Device            : pc-day01
Sample Count      : 20
Warning Threshold : 0.7
```

이 프로그램은 아직 Edge AI 프로그램이 아닙니다.

하지만 다음 연결은 이미 만들어졌습니다.

```text
설정파일
→ Python 프로그램
→ 실행 결과
```

![[Pasted image 20260910133933.png]]

---

# 12. 일부러 설정파일 오류 만들기

정상적으로 실행되었다면 일부러 실패시켜 봅니다.

`configs/settings.yaml`의 파일명을 잠시 다음처럼 변경합니다.

```text
settings.yaml
→ settings_test.yaml
```

다시 실행합니다.

```bash
python -m scripts.check_config
```

오류 메시지에서 다음 부분을 찾아봅니다.

```text
FileNotFoundError
설정파일을 찾을 수 없습니다.
```

파일명을 원래대로 돌립니다.

```text
settings_test.yaml
→ settings.yaml
```

다시 실행합니다.

```bash
python -m scripts.check_config
```

이 실습의 목적은 오류를 피하는 것이 아닙니다.

```text
오류 발생
→ 메시지 확인
→ 문제 위치 확인
→ 수정
→ 재실행
```

이라는 습관을 만드는 것입니다.

---

# 13. 첫 번째 Git Checkpoint 저장하기

현재까지는 다음 기능이 정상입니다.

```text
프로젝트 폴더
→ 가상환경
→ Package
→ 설정파일
→ Python에서 설정 읽기
```

여기까지 정상 상태를 Git에 저장합니다.

```bash
git status
git add .
git status
git commit -m "chore: initialize day01 edge project"
git log --oneline
```

이 Commit은 앞으로 돌아올 수 있는 **첫 번째 정상 Version**입니다.

---

# 14. Edge Pipeline Simulator 만들기

이제 개요에서 본 `입력 → 처리 → 판단 → 출력` 흐름을 실제 프로그램으로 실행합니다.

오늘은 Camera 대신 0.0~1.0 사이의 가상 입력값을 사용합니다.

예:

```text
0.12
0.46
0.81
0.35
0.92
```

기준값이 `0.70`이라면:

```text
0.12 → NORMAL
0.46 → NORMAL
0.81 → WARNING
0.35 → NORMAL
0.92 → WARNING
```

Camera나 AI가 없어도 **입력값을 받아 상태를 판단하고 결과를 기록하는 구조**를 먼저 경험할 수 있습니다.

---

# 15. 핵심 Pipeline 코드 작성하기

## 이 코드는 왜 만들까요? 

앞에서 `settings.yaml`을 읽는 구조를 만들었습니다. 

이제는 설정값만 읽는 것이 아니라, 실제로 입력값을 받아 판단하고 그 결과를 기록하는 기능이 필요합니다. 

이번 코드는 아직 Camera나 AI 모델을 사용하지 않고, 먼저 가상의 숫자를 이용하여 다음 흐름을 연습하기 위해 만듭니다.

즉, 앞으로 Camera와 AI 모델이 연결되기 전에 Edge Pipeline의 가장 기본적인 처리 구조를 먼저 만드는 코드입니다.

다음 파일을 만듭니다.

```text
src/pipeline.py
```

코드:

```python
import csv
import random
import time
from pathlib import Path


def generate_input() -> float:
    return random.random()


def preprocess(value: float) -> float:
    return value


def decide(value: float, threshold: float) -> str:
    if value >= threshold:
        return "WARNING"

    return "NORMAL"


def append_log(
    log_path: str,
    device_name: str,
    value: float,
    state: str,
    latency_ms: float,
) -> None:
    path = Path(log_path)
    path.parent.mkdir(parents=True, exist_ok=True)

    is_new = not path.exists()

    with path.open("a", newline="", encoding="utf-8") as file:
        writer = csv.writer(file)

        if is_new:
            writer.writerow(
                [
                    "timestamp",
                    "device",
                    "input_value",
                    "state",
                    "latency_ms",
                ]
            )

        writer.writerow(
            [
                time.strftime("%Y-%m-%d %H:%M:%S"),
                device_name,
                f"{value:.3f}",
                state,
                f"{latency_ms:.3f}",
            ]
        )
```

의사코드

```
[의사코드]

프로그램에서 사용할 기능을 준비한다

CSV 파일을 다루기 위한 csv를 가져온다
랜덤 숫자를 만들기 위한 random을 가져온다
현재 시간을 기록하기 위한 time을 가져온다
파일 경로를 다루기 위한 Path를 가져온다


1. 가상의 입력값 만들기

0.0 이상 1.0 미만의
랜덤 숫자 하나를 만든다
        ↓
그 값을 반환한다


2. 입력값 전처리하기

입력값을 하나 받는다
        ↓
현재는 별도의 처리를 하지 않는다
        ↓
입력받은 값을 그대로 반환한다


3. NORMAL / WARNING 판단하기

입력값과 기준값 Threshold를 받는다
        ↓
입력값이 Threshold 이상인지 확인한다
        ↓
이상이면
    "WARNING" 반환
        ↓
미만이면
    "NORMAL" 반환


4. 실행 결과를 CSV Log에 기록하기

로그 파일 경로를 입력받는다
장치 이름을 입력받는다
입력값을 입력받는다
판단 결과를 입력받는다
처리시간을 입력받는다
        ↓
문자열 형태의 파일 경로를
Path 객체로 변환한다
        ↓
로그 파일이 저장될 폴더가 없으면
자동으로 폴더를 만든다
        ↓
로그 파일이 이미 있는지 확인한다
        ↓
CSV 파일을 이어쓰기 방식으로 연다
        ↓
파일이 처음 만들어진 경우라면
다음 제목 행을 먼저 기록한다

timestamp
device
input_value
state
latency_ms
        ↓
현재 시간을 구한다
        ↓
입력값을 소수점 3자리로 정리한다
        ↓
처리시간도 소수점 3자리로 정리한다
        ↓
현재 시간
장치 이름
입력값
NORMAL / WARNING
처리시간
을 한 줄로 CSV에 저장한다
```

이 파일에는 서로 다른 역할의 함수가 있습니다.

```text
generate_input()
→ 입력 만들기

preprocess()
→ 입력 처리

decide()
→ 상태 판단

append_log()
→ 실행 결과 기록
```

지금은 매우 단순하지만 앞으로 다음처럼 바뀔 수 있습니다.

```text
generate_input()
가상 숫자 → Camera Frame

preprocess()
그대로 전달 → Resize / Normalize

decide()
Threshold → ONNX AI + 운영 Threshold

append_log()
기본 상태 → Confidence / FPS / Error 추가
```

즉, 오늘 만든 구조가 이후 수업에서 계속 확장됩니다.

---

# 16. Edge Pipeline 실행 프로그램 만들기

다음 파일을 만듭니다.

```text
scripts/run_pipeline.py
```

코드:

```python
import random
import time

from src.config_loader import load_config
from src.pipeline import (
    append_log,
    decide,
    generate_input,
    preprocess,
)


def percentile(values, percent):
    ordered = sorted(values)

    if not ordered:
        return 0.0

    index = int((len(ordered) - 1) * percent)
    return ordered[index]


def main():
    config = load_config("configs/settings.yaml")

    sample_count = int(config["sample_count"])
    interval_sec = float(config["interval_ms"]) / 1000.0
    threshold = float(config["warning_threshold"])

    random.seed(13)

    latencies = []
    warning_count = 0

    print("=== Edge Pipeline Simulator ===")
    print(f"Device    : {config['device_name']}")
    print(f"Threshold : {threshold}")
    print()

    for index in range(1, sample_count + 1):
        started = time.perf_counter()

        value = generate_input()
        processed = preprocess(value)
        state = decide(processed, threshold)

        elapsed_ms = (time.perf_counter() - started) * 1000.0
        latencies.append(elapsed_ms)

        if state == "WARNING":
            warning_count += 1

        append_log(
            log_path=config["log_path"],
            device_name=config["device_name"],
            value=value,
            state=state,
            latency_ms=elapsed_ms,
        )

        print(
            f"{index:02d} | "
            f"value={value:.3f} | "
            f"{state:7s} | "
            f"{elapsed_ms:8.3f} ms"
        )

        time.sleep(interval_sec)

    mean_latency = sum(latencies) / len(latencies)
    p95_latency = percentile(latencies, 0.95)

    effective_fps = (
        1000.0 / mean_latency
        if mean_latency > 0
        else 0.0
    )

    print()
    print("=== Result ===")
    print(f"Samples       : {sample_count}")
    print(f"Warnings      : {warning_count}")
    print(f"Mean Latency  : {mean_latency:.3f} ms")
    print(f"P95 Latency   : {p95_latency:.3f} ms")
    print(f"Approx. FPS   : {effective_fps:.2f}")


if __name__ == "__main__":
    main()
```

### 잠깐 이해하기 — `random.seed(13)`은 왜 사용할까요?

`generate_input()`은 매번 랜덤한 숫자를 만듭니다.

그런데 Threshold를 `0.50`, `0.70`, `0.90`으로 바꾸어 비교할 때 입력값까지 매번 달라지면 **결과가 Threshold 때문에 달라진 것인지, 입력이 달라져서 그런 것인지 구분하기 어렵습니다.**

```text
random.seed(13)
        ↓
같은 순서의 랜덤 입력을 다시 만들 수 있음
        ↓
입력은 최대한 같게 유지
        ↓
Threshold만 변경
        ↓
결과 차이를 더 쉽게 비교
```

즉, 오늘은 `random.seed(13)`을 **공정한 비교 실험을 위해 입력 조건을 반복 가능하게 만드는 장치**라고 이해하면 됩니다.

의사코드

```
[의사코드]

필요한 기능을 불러온다

- 설정파일 읽기
- 가상 입력값 만들기
- 입력 전처리
- NORMAL / WARNING 판단
- CSV Log 저장
        ↓

settings.yaml 파일을 읽는다
        ↓

설정값에서 다음 값을 가져온다

- 몇 번 실행할지
- 입력 간격
- WARNING 기준값
        ↓

랜덤 입력값이 매번 같은 순서로 나오도록
random seed를 고정한다
        ↓

Latency를 저장할 빈 목록을 만든다
WARNING 횟수를 0으로 시작한다
        ↓

프로그램 제목과
현재 장치 이름,
WARNING 기준값을 출력한다
        ↓

설정된 횟수만큼 반복한다
        ↓

현재 시간을 기록한다
        ↓

가상의 입력값을 하나 만든다
        ↓

입력값을 전처리한다
        ↓

Threshold와 비교하여

NORMAL 또는 WARNING을 판단한다
        ↓

처리가 끝난 시간을 측정하여
Latency를 계산한다
        ↓

Latency를 목록에 저장한다
        ↓

결과가 WARNING이면
WARNING 횟수를 1 증가시킨다
        ↓

현재 실행 결과를 CSV Log에 저장한다

- 입력값
- NORMAL / WARNING
- Latency
- 장치 이름
        ↓

현재 결과를 터미널에 출력한다
        ↓

설정한 시간만큼 기다린다
        ↓

다음 입력값을 다시 처리한다
        ↓

모든 반복이 끝나면
전체 결과를 계산한다

- 평균 Latency
- P95 Latency
- 대략적인 FPS
        ↓

최종 결과를 화면에 출력한다

- 전체 Sample 수
- WARNING 횟수
- 평균 Latency
- P95 Latency
- Approx. FPS
```

오늘은 실제 AI 모델 대신 아주 작은 판단 로직을 사용합니다.

중요한 것은 코드의 복잡도가 아니라 다음 구조가 실제로 연결되는지 확인하는 것입니다.

```text
입력 생성
→ 전처리
→ 판단
→ 결과 출력
→ Log 기록
```

---

# 17. 실행 전에 지금까지 만든 파일이 어떻게 연결되는지 확인하기

지금까지 위에서 여러 파일을 만들었습니다.

```text
settings.yaml 
config_loader.py 
check_config.py 
pipeline.py 
run_pipeline.py
```

각 파일은 따로 존재하지만, 실제로는 서로 연결되어 하나의 프로그램으로 동작합니다.

먼저 각 파일의 역할을 간단히 확인합니다.

```
settings.yaml
→ 프로그램에서 사용할 설정값을 저장

config_loader.py
→ settings.yaml을 읽어서 Python에서 사용할 수 있게 변환

check_config.py
→ 설정파일이 정상적으로 읽히는지 확인

pipeline.py
→ 입력 생성, 전처리, 판단, Log 저장 기능을 담당

run_pipeline.py
→ 위의 기능들을 불러와 실제로 반복 실행
```

전체 연결은 다음과 같습니다.

```
settings.yaml
설정값 저장
        ↓
config_loader.py
설정값 읽기
        ↓
check_config.py
설정이 정상인지 먼저 확인
        ↓
run_pipeline.py 실행
        ↓
pipeline.py의 기능 사용

generate_input()
가상 입력 생성
        ↓
preprocess()
입력 처리
        ↓
decide()
NORMAL / WARNING 판단
        ↓
append_log()
CSV Log 저장
        ↓
run_pipeline.py
처리시간 계산 및 결과 출력
        ↓
설정된 횟수만큼 반복
        ↓
최종 결과 정리
```

여기서 중요한 점은 `check_config.py`와 `run_pipeline.py`의 역할이 다르다는 것입니다.

```
check_config.py
→ "설정파일을 제대로 읽을 수 있나?" 확인하는 테스트 프로그램

run_pipeline.py
→ "실제 Pipeline을 실행하자"라는 메인 실행 프로그램
```

따라서 실제 실행 순서는 다음처럼 이해하면 됩니다.

```
1. settings.yaml에 설정값 작성
        ↓
2. check_config.py 실행
   → 설정이 제대로 읽히는지 확인
        ↓
3. 문제가 없다면 run_pipeline.py 실행
        ↓
4. pipeline.py의 기능들이 순서대로 동작
        ↓
5. NORMAL / WARNING 결과 출력
        ↓
6. CSV Log 기록
        ↓
7. 처리시간과 전체 결과 확인
```

즉, 지금까지 만든 것은 여러 개의 독립된 코드가 아니라,

> **설정파일을 읽고 → 입력을 만들고 → 판단하고 → 결과를 출력하고 기록하는 하나의 작은 Edge Pipeline 프로그램**

입니다.

1일차에서는 아직 실제 Camera나 AI 모델을 사용하지 않습니다.

```
현재

가상 숫자
→ 판단
→ 결과
→ Log

앞으로

Camera
→ AI Model
→ 판단
→ GPIO / Log
```

![[Pasted image 20260910161859.png]]

---

# 18. Guided Lab — Edge Pipeline 실행

이제 PC에서 작은 Edge Pipeline을 실행합니다.

```bash
python -m scripts.run_pipeline
```

예상 형태:

```text
=== Edge Pipeline Simulator ===
Device    : pc-day01
Threshold : 0.7

01 | value=0.xxx | NORMAL  |    x.xxx ms
02 | value=0.xxx | WARNING |    x.xxx ms
...
```

마지막에는 다음 값이 출력됩니다.

```text
Samples
Warnings
Mean Latency
P95 Latency
Approx. FPS
```

오늘은 숫자 자체를 성능 기준으로 해석하지 않습니다.

다음 흐름이 실제로 동작하는지만 먼저 확인합니다.

```text
입력이 계속 만들어지는가?
        ↓
각 입력마다 NORMAL / WARNING이 나오는가?
        ↓
처리 시간이 기록되는가?
        ↓
CSV Log가 저장되는가?
```

### Latency와 FPS 숫자를 어떻게 봐야 할까요?

1일차의 입력과 판단은 매우 단순하기 때문에 `0.005 ms`처럼 아주 작은 Latency가 나올 수 있습니다.

또한 현재의 `Approx. FPS`는 **아주 짧은 처리 구간의 평균 Latency만 이용해 계산한 참고값**입니다. 실제 Camera 입력, AI 모델 추론, 파일 저장, 대기시간까지 포함한 장치의 실제 FPS를 뜻하지 않습니다.

```text
1일차
→ "처리시간을 코드로 측정할 수 있다"는 것만 확인

8일차
→ Raspberry Pi + ONNX 실제 Pipeline에서
   Latency / FPS를 본격적으로 측정하고 비교
```

따라서 오늘은 숫자가 크거나 작은지 평가하지 않습니다.

---

## 실행 결과를 코드에서 다시 찾아보기

프로그램이 정상적으로 실행되었다면 이번에는 결과만 보고 넘어가지 않고, **화면에 나온 값이 어느 파일의 어떤 코드에서 만들어졌는지 직접 찾아봅니다.**

지금까지 만든 파일을 다시 열어 다음 항목을 확인합니다.

```
터미널에 보이는 value는 어디에서 만들어질까? 
          ↓ 
NORMAL / WARNING은 어디에서 판단할까? 
          ↓ 
Latency는 어디에서 계산할까? 
          ↓ 
화면 출력은 어느 코드가 담당할까? 
          ↓ 
CSV Log는 어느 코드가 저장할까? 
          ↓ 
최종 Samples / Warnings / Mean Latency는 어디에서 계산하고 출력할까?
```

### 위의 항목을 확인한 후에 아래 부분을 직접 체크합니다.

#### 1. `value`는 어디에서 만들어질까요?

`src/pipeline.py`를 엽니다.

다음 함수를 찾습니다.

```python
def generate_input() -> float:
    return random.random()
```

이 함수가 `0.0 ~ 1.0` 사이의 가상 입력값을 만듭니다.

예를 들어

```python
value=0.734
```

의 `0.734`가 바로 여기에서 만들어진 값입니다.

---

#### 2. NORMAL / WARNING은 어디에서 결정될까요?

같은 `src/pipeline.py`에서 다음 함수를 찾습니다.

```python
def decide(value: float, threshold: float) -> str:
    if value >= threshold:
        return "WARNING"

    return "NORMAL"
```

현재 Threshold가 `0.70`이라면

```python
0.734 >= 0.70
→ WARNING

0.225 < 0.70
→ NORMAL
```

처럼 판단합니다.

---

#### 3. 처리시간 Latency는 어디에서 계산할까요?

`scripts/run_pipeline.py`를 엽니다.

```python
started = time.perf_counter()
```

에서 시작 시간을 기록하고,

```python
elapsed_ms = (time.perf_counter() - started) * 1000.0
```

에서 처리시간을 계산합니다.

그래서 터미널에 다음과 같은 값이 표시됩니다.

```
0.006 ms
0.004 ms
0.005 ms
```

---

#### 4. 한 줄의 실행 결과는 어디에서 출력할까요?

`run_pipeline.py`에서 다음 `print()` 부분을 찾습니다.

```python
print(
    f"{index:02d} | "
    f"value={value:.3f} | "
    f"{state:7s} | "
    f"{elapsed_ms:8.3f} ms"
)
```

이 코드가 다음과 같은 한 줄을 만들어 냅니다.

```
09 | value=0.734 | WARNING | 0.006 ms
```

즉,

```
09
→ 몇 번째 입력인지

value=0.734
→ 생성된 입력값

WARNING
→ 판단 결과

0.006 ms
→ 처리시간
```

입니다.

---

#### 5. CSV Log는 어디에서 저장할까요?

`src/pipeline.py`의 `append_log()` 함수를 찾습니다.

```python
def append_log(...):
```

이 함수가 실행 결과를

```
logs/day01_events.csv
```

파일에 저장합니다.

저장되는 값은 다음과 같습니다.

```
timestamp
device
input_value
state
latency_ms
```

즉, 터미널에서 잠깐 보이는 결과를 파일에 남겨 나중에도 확인할 수 있게 만든 것입니다.

---

#### 6. 마지막 전체 결과는 어디에서 계산할까요?

다시 `scripts/run_pipeline.py`의 반복문 아래를 확인합니다.

```
mean_latency = sum(latencies) / len(latencies)
p95_latency = percentile(latencies, 0.95)
```

그리고

```python
print(f"Samples       : {sample_count}")
print(f"Warnings      : {warning_count}")
print(f"Mean Latency  : {mean_latency:.3f} ms")
print(f"P95 Latency   : {p95_latency:.3f} ms")
print(f"Approx. FPS   : {effective_fps:.2f}")
```

부분에서 최종 결과를 출력합니다.

---

#### 코드 리뷰 체크

코드를 직접 열어 다음 질문에 답해 봅니다.

- `value`를 생성하는 함수는 무엇인가?
- `NORMAL / WARNING`을 결정하는 함수는 무엇인가?
- `warning_threshold` 값은 어느 파일에서 가져오는가?
- Latency는 어느 파일에서 계산하는가?
- 한 줄의 실행 결과는 어느 파일에서 출력하는가?
- CSV 파일을 저장하는 함수는 무엇인가?
- `Warnings` 개수는 어디에서 증가하는가?
- 최종 평균 Latency는 어디에서 계산하는가?

모두 찾았다면 지금 실행한 프로그램이 다음처럼 연결된다는 것을 이해한 것입니다.

```
settings.yaml
        ↓
설정 읽기
        ↓
가상 입력 생성
        ↓
판단
        ↓
처리시간 계산
        ↓
터미널 출력
        ↓
CSV Log 저장
        ↓
최종 결과 계산
```

> 실행 결과를 보는 것에서 끝나는 것이 아니라, **“이 결과가 어느 코드에서 만들어졌는가?”를 다시 찾아보는 것이 이번 코드 리뷰의 목적입니다.**

### 3분 복습 — 파일을 보지 않고 말해보기

잠시 코드를 닫고 다음 문장을 자신의 말로 완성해 봅니다.

```text
settings.yaml은 __________________________ 파일이다.

config_loader.py는 _______________________ 역할이다.

pipeline.py는 ____________________________ 기능을 모아 둔 파일이다.

check_config.py는 ________________________ 확인하는 프로그램이다.

run_pipeline.py는 ________________________ 실행하는 시작점이다.
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
settings.yaml
→ 실행에 사용할 설정값을 저장하는 파일

config_loader.py
→ YAML 설정값을 읽어 Python Dictionary로 바꾸는 역할

pipeline.py
→ 입력 생성, 전처리, 판단, Log 저장 기능을 모아 둔 파일

check_config.py
→ 설정파일이 정상적으로 읽히는지 확인하는 프로그램

run_pipeline.py
→ 설정과 Pipeline 기능을 연결하여 전체 흐름을 실행하는 시작점
```

정답의 문장을 그대로 외울 필요는 없습니다.  
각 파일이 **왜 존재하는지 자신의 말로 설명할 수 있으면 됩니다.**

</details>

---

# 19. 로그 파일 확인하기

다음 명령으로 로그를 확인합니다.

```bash
cat logs/day01_events.csv
```

예:

```csv
timestamp,device,input_value,state,latency_ms
2026-10-01 10:15:01,pc-day01,0.259,NORMAL,0.012
2026-10-01 10:15:01,pc-day01,0.685,NORMAL,0.009
```

화면에 결과를 출력하는 것과 Log를 남기는 것은 목적이 다릅니다.

```text
터미널 출력
→ 지금 실행 결과를 바로 확인

CSV Log
→ 실행이 끝난 뒤 결과 분석
```

현장 장치는 항상 모니터가 연결되어 있지 않을 수 있기 때문에 **결과를 기록하는 습관**이 중요합니다.

---

# 20. Modify Lab — WARNING 기준 변경

이번에는 판단 기준 하나만 바꾸어 결과가 어떻게 달라지는지 확인합니다.

기존:

```yaml
warning_threshold: 0.70
```

변경:

```yaml
warning_threshold: 0.50
```

다시 실행합니다.

```bash
python -m scripts.run_pipeline
```

`Warnings` 개수가 어떻게 달라졌는지 확인합니다.

다시 다음처럼 변경합니다.

```yaml
warning_threshold: 0.90
```

실행합니다.

```bash
python -m scripts.run_pipeline
```

다음 표를 `reports/day01_result.md`에 추가합니다.

```md
## Threshold Experiment

| Threshold | Warning Count | 관찰 |
|---:|---:|---|
| 0.50 |  |  |
| 0.70 |  |  |
| 0.90 |  |  |
```

오늘은 다음 관계만 경험하면 됩니다.

```text
같은 입력 데이터
    +
기준값 변경
        ↓
최종 운영 상태가 달라질 수 있다.
```

실험이 끝나면 기본값으로 되돌립니다.

```yaml
warning_threshold: 0.70
```

---

# 21. 한 번에 한 변수만 바꾸는 이유

앞의 실험에서는 `warning_threshold` 하나만 변경했습니다.

```text
변경
warning_threshold

고정
입력 생성 방식
프로그램 구조
Sample 수
```

여러 조건을 한꺼번에 바꾸면 결과가 달라진 원인을 찾기 어렵습니다.

앞으로 모델 성능과 Edge 시스템의 속도를 비교할 때도 같은 원칙을 사용합니다.

> **비교 실험에서는 가능한 한 한 번에 한 변수만 변경합니다.**

### 같은 입력으로 비교하는 이유 다시 확인하기

이번 실험에서 `random.seed(13)`을 사용했기 때문에 프로그램을 다시 실행해도 같은 순서의 가상 입력을 재현할 수 있습니다.

예를 들어 다음 두 실험을 비교한다고 생각해 봅니다.

```text
실험 A
같은 입력 + Threshold 0.50

실험 B
같은 입력 + Threshold 0.90
```

입력이 같기 때문에 Warning 개수가 달라졌다면 **Threshold 변경의 영향**을 더 분명하게 확인할 수 있습니다.

이 생각은 뒤에서 AI 모델, ONNX, Raspberry Pi의 성능을 비교할 때도 그대로 사용합니다.

---

# 22. Mini Challenge — 작은 Edge Pipeline을 스스로 다시 이해하기

지금까지는 안내된 순서대로 코드를 작성했습니다.

이제부터는 **결과를 먼저 생각하고, 코드에서 원인을 찾고, 필요한 부분만 수정하는 연습**을 합니다.

새로운 어려운 기술을 배우는 시간이 아닙니다.

```text
실행 결과 보기
        ↓
어디에서 만들어졌는지 찾기
        ↓
결과를 먼저 예상하기
        ↓
설정값 하나 바꾸기
        ↓
필요한 기능 하나 추가하기
        ↓
Log로 다시 확인하기
        ↓
오류가 나면 원인을 찾아 수정하기
```

Mini Challenge에서는 정답을 바로 보지 않습니다.

먼저 직접 생각하고 실행한 뒤, 각 Challenge 아래의 **예시 정답 확인해보기**를 열어 자신의 생각과 비교합니다.

---

## Challenge 1 — 실행 결과 한 줄을 코드로 역추적하기

다음 결과가 터미널에 출력되었다고 가정합니다.

```text
09 | value=0.734 | WARNING | 0.006 ms
```

코드를 열어 다음 질문에 답합니다.

1. `09`는 어느 반복문에서 만들어지는가?
2. `0.734`라는 `value`는 어느 함수에서 만들어지는가?
3. `WARNING`은 어느 함수에서 결정되는가?
4. `0.006 ms`는 어느 파일에서 계산되는가?
5. 이 한 줄을 화면에 출력하는 `print()`는 어느 파일에 있는가?
6. 같은 결과를 CSV에 저장하는 함수는 무엇인가?

가능하면 다음 표를 직접 채웁니다.

| 화면에 보인 값 | 담당 파일 | 담당 코드 또는 함수 |
|---|---|---|
| `09` |  |  |
| `value=0.734` |  |  |
| `WARNING` |  |  |
| `0.006 ms` |  |  |
| 터미널 한 줄 출력 |  |  |
| CSV 저장 |  |  |

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

| 화면에 보인 값 | 담당 파일 | 담당 코드 또는 함수 |
|---|---|---|
| `09` | `scripts/run_pipeline.py` | `for index in range(...)`의 `index` |
| `value=0.734` | `src/pipeline.py` | `generate_input()` |
| `WARNING` | `src/pipeline.py` | `decide()` |
| `0.006 ms` | `scripts/run_pipeline.py` | `elapsed_ms` 계산 |
| 터미널 한 줄 출력 | `scripts/run_pipeline.py` | 반복문 안의 `print()` |
| CSV 저장 | `src/pipeline.py` | `append_log()` |

핵심은 결과 한 줄이 한 파일에서 전부 만들어지는 것이 아니라는 점입니다.

```text
pipeline.py에서 값과 상태를 만들고
        +
run_pipeline.py가 실행 순서를 제어하고 출력하며
        +
append_log()가 결과를 파일에 남깁니다.
```

</details>

---

## Challenge 2 — 실행하기 전에 NORMAL / WARNING을 먼저 예상하기

이번에는 프로그램을 실행하지 말고 먼저 판단해 봅니다.

Threshold가 다음과 같다고 가정합니다.

```text
warning_threshold = 0.70
```

입력값:

```text
0.23
0.51
0.69
0.70
0.92
```

다음 표의 `예상 상태`를 먼저 작성합니다.

| Input | 예상 상태 |
|---:|---|
| 0.23 |  |
| 0.51 |  |
| 0.69 |  |
| 0.70 |  |
| 0.92 |  |

특히 `0.70`이 어떤 상태가 되는지 `decide()`의 조건을 다시 읽어봅니다.

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

현재 코드는 다음 조건을 사용합니다.

```python
if value >= threshold:
    return "WARNING"
```

따라서 정답은 다음과 같습니다.

| Input | 상태 | 이유 |
|---:|---|---|
| 0.23 | NORMAL | 0.70보다 작음 |
| 0.51 | NORMAL | 0.70보다 작음 |
| 0.69 | NORMAL | 0.70보다 작음 |
| 0.70 | WARNING | `>=`이므로 같은 값도 WARNING |
| 0.92 | WARNING | 0.70보다 큼 |

여기서 중요한 것은 `>`와 `>=`의 차이입니다.

```text
value > 0.70
→ 0.70은 NORMAL

value >= 0.70
→ 0.70도 WARNING
```

운영 기준을 코드로 만들 때 경계값을 정확하게 정하는 이유입니다.

</details>

---

## Challenge 3 — Python 코드는 그대로 두고 설정값만 바꾸기

이번 Challenge에서는 `src/pipeline.py`와 `scripts/run_pipeline.py`를 수정하지 않습니다.

`configs/settings.yaml`만 다음과 같이 변경합니다.

```yaml
device_name: challenge-pc

sample_count: 5
interval_ms: 500

warning_threshold: 0.80

log_path: logs/day01_challenge.csv
```

실행하기 전에 먼저 예상합니다.

```text
몇 번의 입력이 처리될까?

입력 사이의 대기시간은 약 몇 초일까?

터미널의 Device 이름은 무엇으로 보일까?

Log는 어느 파일에 저장될까?

Python 코드를 수정하지 않았는데도 실행 결과가 바뀔까?
```

이제 실행합니다.

```bash
python -m scripts.run_pipeline
```

실행 결과와 자신의 예상을 비교합니다.

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
Sample 수
→ 5

입력 간격
→ 500 ms = 0.5초

Device
→ challenge-pc

Log
→ logs/day01_challenge.csv
```

Python 코드 자체는 바꾸지 않았지만 프로그램은 `settings.yaml`을 읽기 때문에 실행 조건이 달라집니다.

이것이 설정값을 코드와 분리한 이유입니다.

```text
코드 수정 없이
settings.yaml 변경
        ↓
실행 조건 변경
```

</details>

Challenge가 끝나면 기본 설정으로 되돌립니다.

```yaml
device_name: pc-day01

sample_count: 20
interval_ms: 100

warning_threshold: 0.70

log_path: logs/day01_events.csv
```

---

## Challenge 4 — NORMAL과 WARNING 사이에 CAUTION 추가하기

지금까지 상태는 두 가지였습니다.

```text
NORMAL
WARNING
```

이번에는 요구사항만 보고 중간 상태를 하나 추가합니다.

```text
input < 0.50
→ NORMAL

0.50 <= input < 0.70
→ CAUTION

input >= 0.70
→ WARNING
```

바로 코드를 수정하지 말고 먼저 **어느 파일들이 영향을 받는지** 생각합니다.

```text
새 기준값을 어디에 저장해야 할까?

설정값을 어느 파일이 읽을까?

실제 상태를 판단하는 함수는 무엇일까?

run_pipeline.py는 새 기준값을 decide()에 어떻게 전달해야 할까?

CSV Log는 CAUTION도 그대로 기록할 수 있을까?
```

### 1단계 — 설정값 추가

`configs/settings.yaml`에 다음 기준을 추가합니다.

```yaml
caution_threshold: 0.50
warning_threshold: 0.70
```

### 2단계 — 판단 함수 수정

수정할 핵심 위치는 다음입니다.

```text
src/pipeline.py
→ decide()
```

먼저 직접 수정합니다.

조건의 순서도 생각해 봅니다.

```text
0.80은 WARNING이어야 하는데
CAUTION 조건을 먼저 검사하면 어떻게 될까?
```

### 3단계 — 실행 프로그램 연결

`scripts/run_pipeline.py`에서도 새 설정값을 읽고 `decide()`에 전달해야 합니다.

### 4단계 — 결과 확인

```bash
python -m scripts.run_pipeline
```

다음 세 상태가 모두 나올 수 있는 구조인지 확인합니다.

```text
NORMAL
CAUTION
WARNING
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

`configs/settings.yaml`

```yaml
caution_threshold: 0.50
warning_threshold: 0.70
```

`src/pipeline.py`

```python
def decide(
    value: float,
    caution_threshold: float,
    warning_threshold: float,
) -> str:
    if value >= warning_threshold:
        return "WARNING"

    if value >= caution_threshold:
        return "CAUTION"

    return "NORMAL"
```

`run_pipeline.py`에서 설정값을 읽습니다.

```python
caution_threshold = float(config["caution_threshold"])
warning_threshold = float(config["warning_threshold"])
```

판단 함수를 호출합니다.

```python
state = decide(
    processed,
    caution_threshold,
    warning_threshold,
)
```

조건을 높은 기준부터 확인하는 이유도 중요합니다.

```text
value = 0.80

먼저 WARNING 검사
0.80 >= 0.70
→ WARNING
```

반대로 CAUTION 조건을 너무 넓게 먼저 검사하면 WARNING까지 CAUTION으로 처리할 수 있습니다.

CSV의 `state` 열은 문자열을 그대로 저장하기 때문에 `append_log()`는 별도 수정 없이 `CAUTION`도 기록할 수 있습니다.

</details>

---

## Challenge 5 — CSV Log가 정말 실행 결과와 연결되어 있는지 확인하기

이번에는 화면만 보지 않고 **기록된 결과가 실제 실행과 맞는지 확인**합니다.

Challenge 전용 Log를 새로 시작합니다.

```bash
rm -f logs/day01_challenge.csv
```

설정을 다음처럼 바꿉니다.

```yaml
sample_count: 5
log_path: logs/day01_challenge.csv
```

실행합니다.

```bash
python -m scripts.run_pipeline
```

Log의 앞부분을 확인합니다.

```bash
head -n 6 logs/day01_challenge.csv
```

전체 줄 수도 확인합니다.

```bash
wc -l logs/day01_challenge.csv
```

다음 질문에 답합니다.

```text
Sample을 5개 실행했는데
왜 CSV 파일은 6줄일까?

첫 번째 줄은 무엇일까?

터미널의 value와 CSV의 input_value가 연결되는가?

터미널의 NORMAL / CAUTION / WARNING과
CSV의 state가 연결되는가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

새 Log 파일에서 Sample을 5개 처리했다면 보통 다음처럼 됩니다.

```text
1줄
→ CSV 제목(Header)

5줄
→ 실제 Sample 결과

총 6줄
```

첫 줄:

```csv
timestamp,device,input_value,state,latency_ms
```

그 아래부터 실행 결과가 한 Sample당 한 줄씩 기록됩니다.

즉,

```text
터미널에서 본 현재 결과
        ↓
append_log()
        ↓
CSV에 남은 실행 기록
```

이라는 연결을 확인한 것입니다.

</details>

Challenge가 끝나면 `settings.yaml`의 `sample_count`와 `log_path`를 원래 값으로 되돌립니다.

---

## Challenge 6 — 오류 메시지를 보고 문제 파일 찾기

이번에는 일부러 설정 이름을 잘못 작성합니다.

`configs/settings.yaml`의

```yaml
warning_threshold: 0.70
```

을 잠시 다음처럼 바꿉니다.

```yaml
warning_threshol: 0.70
```

마지막 `d`가 빠졌습니다.

실행합니다.

```bash
python -m scripts.run_pipeline
```

오류가 발생하면 바로 정답을 보지 말고 다음 순서로 읽습니다.

```text
오류의 마지막 줄 확인
        ↓
어떤 이름을 찾지 못했는지 확인
        ↓
run_pipeline.py에서 그 이름을 사용하는 위치 확인
        ↓
settings.yaml의 Key 이름과 비교
        ↓
수정
        ↓
재실행
```

다음 질문에 답합니다.

```text
오류 종류는 무엇인가?

프로그램은 어떤 Key를 찾고 있었는가?

어느 파일의 이름이 잘못되었는가?

고친 뒤 정상 실행되는가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

다음과 비슷한 오류를 볼 수 있습니다.

```text
KeyError: 'warning_threshold'
```

`run_pipeline.py`는 다음 Key를 요구합니다.

```python
config["warning_threshold"]
```

하지만 `settings.yaml`에는 잘못된 이름이 있습니다.

```yaml
warning_threshol: 0.70
```

따라서 설정파일을 다음처럼 다시 고칩니다.

```yaml
warning_threshold: 0.70
```

이 Challenge의 핵심은 오류 문장을 모두 이해하는 것이 아닙니다.

```text
오류가 난 이름 찾기
→ 그 이름을 사용하는 코드 찾기
→ 설정파일과 비교
→ 수정
→ 재실행
```

의 순서를 익히는 것입니다.

</details>

---

## Challenge 7 — 오늘의 Pipeline을 앞으로의 Edge AI와 연결하기

다음 빈칸을 채워 봅니다.

```text
오늘의 generate_input()
가상 숫자 생성
        ↓
앞으로 ________________________

오늘의 preprocess()
값을 그대로 전달
        ↓
앞으로 ________________________

오늘의 decide()
Threshold로 상태 판단
        ↓
앞으로 ________________________

오늘의 append_log()
기본 결과 저장
        ↓
앞으로 ________________________
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
generate_input()
가상 숫자
→ Camera Frame / Sensor 입력

preprocess()
값 그대로 전달
→ Resize / Normalize / 입력 변환

decide()
단순 Threshold
→ AI Model 추론 결과 + 운영 기준으로 판단

append_log()
기본 상태 기록
→ Confidence / FPS / Error / 장치상태 등 추가 기록
```

바뀌는 것은 입력 장치와 처리 기술입니다.

하지만 큰 흐름은 그대로 유지됩니다.

```text
입력
→ 처리
→ 판단
→ 출력
→ 기록
```

이것이 1일차에 작은 Pipeline을 먼저 만드는 이유입니다.

</details>

---

## Challenge 마무리 — 2일차에 이어서 사용할 기본 상태로 되돌리기

Challenge에서는 학습을 위해 여러 설정과 판단 규칙을 바꾸었습니다.

2일차에서는 1일차의 기본 Pipeline을 그대로 이어서 사용하므로 Challenge가 끝나면 **기본 상태로 한 번 정리**합니다.

### 1. `settings.yaml` 기본값 확인

```yaml
device_name: pc-day01

sample_count: 20
interval_ms: 100

warning_threshold: 0.70

log_path: logs/day01_events.csv
```

`caution_threshold`는 Challenge에서만 사용했다면 제거합니다.

### 2. `decide()` 기본 형태 확인

`src/pipeline.py`

```python
def decide(value: float, threshold: float) -> str:
    if value >= threshold:
        return "WARNING"

    return "NORMAL"
```

### 3. `run_pipeline.py`의 기본 연결 확인

```python
threshold = float(config["warning_threshold"])
```

판단 부분:

```python
state = decide(processed, threshold)
```

### 4. 다시 실행하여 정상 상태 확인

```bash
python -m scripts.check_config
python -m scripts.run_pipeline
```

다음 흐름이 다시 정상적으로 동작하면 됩니다.

```text
가상 입력
→ NORMAL / WARNING
→ 터미널 출력
→ CSV Log
```

Challenge에서 만든 `logs/day01_challenge.csv`는 연습용 결과이므로 필요하면 그대로 보관해도 되고 삭제해도 됩니다.

```bash
rm -f logs/day01_challenge.csv
```

이제 2일차에서 이 기본 프로젝트를 Raspberry Pi로 이어서 사용할 준비가 되었습니다.

---

## Mini Challenge 결과 기록

`reports/day01_result.md`에 다음 내용을 추가합니다.

```md
## Mini Challenge Review

### 1. 실행 결과 역추적
내가 찾은 파일과 함수:

### 2. Threshold 경계값
0.70이 WARNING이 되는 이유:

### 3. 설정파일만 변경한 실험
Python 코드를 수정하지 않아도 달라진 결과:

### 4. CAUTION 상태 추가
수정한 파일:
조건 순서에서 주의한 점:

### 5. CSV Log 확인
Sample 수와 CSV 줄 수의 관계:

### 6. 오류 해결
발생한 오류:
원인:
수정한 내용:

### 7. 오늘의 Pipeline 한 문장 설명
내 설명:
```

정답 문장을 복사하는 것이 아니라 **자신이 실제로 실행하고 확인한 결과를 기준으로 작성합니다.**

---

# 23. Challenge가 잘 안 될 때 확인하는 순서

한 번에 전체 코드를 다시 작성하지 않습니다.

다음 순서로 확인합니다.

```text
1. settings.yaml에 값이 있는가?
        ↓
2. config_loader가 설정을 읽는가?
        ↓
3. run_pipeline.py가 값을 가져오는가?
        ↓
4. decide()가 두 Threshold를 받는가?
        ↓
5. 조건 순서가 올바른가?
        ↓
6. 출력 결과가 세 상태로 나오는가?
        ↓
7. CSV에도 세 상태가 기록되는가?
```

문제를 작은 단계로 나누어 확인합니다.

---

# 24. 두 번째 Git Checkpoint 저장하기

Edge Pipeline과 실험이 정상적으로 동작하면 현재 상태를 Git에 저장합니다.

```bash
git status
git diff
git add src scripts configs requirements.txt
git commit -m "feat: add day01 edge pipeline simulator"
git log --oneline
```

기능 단위로 Commit이 분리되어 있는지 확인합니다.

---

# 25. README 만들기

프로젝트 루트에 `README.md`를 만듭니다.

```md
# Subject 13 Edge AI

교과 13 Edge AI 실습 프로젝트입니다.

## Day 01

PC에서 작은 Edge Pipeline Simulator를 실행하여
가상 입력 → 처리 → 판단 → 출력 → Log의 기본 구조를 확인합니다.

## Run

가상환경 활성화:

```bash
source .venv/bin/activate
```

Pipeline 실행:

```bash
python -m scripts.run_pipeline
```

## Current Structure

```text
subject13_edge_ai/
├── configs/
├── reports/
├── scripts/
├── src/
├── .gitignore
├── README.md
└── requirements.txt
```

## Next

Day 02에서는 실제 Raspberry Pi에 SSH로 접속하여
PC에서 작성한 프로그램을 Edge 장치에서 실행할 준비를 합니다.
```

README는 프로젝트 사용설명서입니다.

```text
이 프로젝트는 무엇인가?
어떻게 실행하는가?
현재 어디까지 만들어졌는가?
다음 단계는 무엇인가?
```

를 다른 사람이 확인할 수 있게 작성합니다.

---

# 26. 실습 결과 문서 완성하기

`reports/day01_result.md`에 다음 항목을 작성합니다.

```md
# Day 01 Experiment Result

## 오늘 확인한 것

### 1. 입력 → 처리 → 판단 → 출력

내가 확인한 결과:

### 2. CSV Log

Log에서 확인한 내용:

### 3. Threshold 변경

Threshold를 바꾸었을 때 달라진 점:

### 4. 실패 또는 오류

발생한 문제:

원인:

해결:

### 5. 다음 날 확인하고 싶은 것

PC에서 실행한 이 프로그램을 Raspberry Pi에서 실행하면 무엇이 달라질까?
```

실제 자신이 실행한 결과를 근거로 작성합니다.

---

# 27. Git에 문서까지 저장하기

```bash
git status
git add README.md reports/day01_result.md
git commit -m "docs: record day01 experiment results"
git log --oneline
```

오늘 최소한 다음 세 종류의 Version이 만들어집니다.

```text
프로젝트 초기화
        ↓
Edge Pipeline 기능 구현
        ↓
실험 결과 문서화
```

---

# 28. Remote 저장소를 사용하기 전 보안 확인

교육장에서는 외부 서비스 사용이 제한될 수 있습니다.

따라서 먼저 다음 원칙을 확인합니다.

```text
외부 GitHub 사용이 금지된 환경
→ GitHub에 Push하지 않음

내부 GitLab / Gitea가 제공된 환경
→ 제공된 내부 Repository 사용

Remote 저장소가 아직 없는 환경
→ Local Git Commit까지만 진행
```

Git의 핵심은 Remote 서비스 이름이 아니라 **변경 이력을 안전하게 남기고 정상 Version으로 돌아갈 수 있게 관리하는 것**입니다.

먼저 확인합니다.

```bash
git status
```

다음과 같은 파일이 Git 대상에 들어가 있지 않은지 확인합니다.

```text
기업 제공 데이터
개인정보가 포함된 이미지·영상
비밀키
.env
Password
API Key
대용량 모델
실행 로그
```

현재 `.gitignore`에 의해 다음은 제외되어 있어야 합니다.

```text
.venv/
logs/
data/
datasets/
*.onnx
*.pt
```

---

# 29. Local Git과 Remote Git의 차이 이해하기

현재까지 사용한 Git은 내 PC 안에서 Version을 관리하는 **Local Git**입니다.

```text
내 PC
        ↓
git add
        ↓
git commit
        ↓
Local Repository
```

내부 GitLab이나 Gitea 같은 Remote 저장소가 제공되면 다음 단계가 추가됩니다.

```text
Local Repository
        ↓
git push
        ↓
내부 Remote Repository
```

Remote가 없어도 오늘의 Pipeline 실습과 Git Commit 학습은 정상적으로 진행할 수 있습니다.

---

# 30. 내부 Remote Repository가 제공된 경우 연결하기

교육장에서 내부 GitLab 또는 Gitea Repository 주소를 제공받은 경우에만 진행합니다.

예:

```text
http://INTERNAL_GIT_SERVER/USERNAME/subject13_edge_ai.git
```

프로젝트 터미널에서 연결합니다.

```bash
git remote add origin http://INTERNAL_GIT_SERVER/USERNAME/subject13_edge_ai.git
```

연결 내용을 확인합니다.

```bash
git remote -v
```

이미 `origin`이 존재하면 새로 추가하지 않고 현재 주소를 먼저 확인합니다.

---

# 31. Remote가 있는 경우 첫 Push 하기

내부 Remote Repository가 연결되어 있다면 다음 명령을 실행합니다.

```bash
git push -u origin main
```

Push가 끝나면 Remote Repository에서 다음 파일과 폴더가 보이는지 확인합니다.

```text
configs/
reports/
scripts/
src/
.gitignore
README.md
requirements.txt
```

다음 항목은 보이지 않아야 합니다.

```text
.venv/
logs/
```

Remote Repository가 제공되지 않은 환경에서는 이 단계의 Push를 생략하고 Local Commit 기록만 확인합니다.

---

# 32. Version 기록이 제대로 남았는지 확인하기

터미널에서 다음 명령을 실행합니다.

```bash
git log --oneline
```

오늘 최소한 다음과 같은 흐름의 Commit이 구분되어 있는지 확인합니다.

```text
프로젝트 초기화
        ↓
Edge Pipeline 기능 구현
        ↓
실험 결과 문서화
```

필요한 파일만 Version 관리 대상에 포함되었는지도 확인합니다.

```bash
git status
```

---

# 33. 마지막 Modify — 설정 하나 바꾸고 Version 저장하기

`configs/settings.yaml`에서:

```yaml
device_name: pc-day01
```

을 자신의 장치 이름으로 바꿉니다.

예:

```yaml
device_name: learner01-pc
```

프로그램을 다시 실행합니다.

```bash
python -m scripts.run_pipeline
```

출력에서 Device 이름이 바뀌었는지 확인합니다.

Git 변경사항을 확인합니다.

```bash
git diff
```

Commit합니다.

```bash
git add configs/settings.yaml
git commit -m "chore: set local device name"
```

내부 Remote Repository가 연결되어 있는 경우에만 Push합니다.

```bash
git push
```

핵심은 다음 흐름입니다.

```text
설정 변경
→ 실행하여 확인
→ git diff로 변경 확인
→ Commit
→ 필요한 경우 내부 Remote에 Push
```

---

# 34. 오늘 만든 시스템을 다시 보기

1일차가 끝났을 때 만들어진 것은 완성된 Edge AI가 아닙니다.

현재 상태는 다음과 같습니다.

```text
[PC]

settings.yaml
        ↓
가상 입력 생성
        ↓
Preprocess
        ↓
Decision
        ↓
NORMAL / WARNING
(Challenge 완료 시 CAUTION 포함)
        ↓
Terminal
        ↓
CSV Log
        ↓
처리시간 기록
```

여기서 중요한 것은 오늘 만든 가상 입력과 단순 판단 로직이 앞으로 실제 입력과 AI 추론으로 교체된다는 점입니다.

```text
오늘
가상의 숫자 + Threshold 판단

        ↓

3~5일차
Camera / Sensor + Rule

        ↓

6~7일차
AI Model + ONNX

        ↓

Raspberry Pi에서 실제 Edge AI 실행
```

즉, 오늘은 **전체 Edge 시스템의 뼈대를 PC에서 먼저 실행해 본 것**입니다.

---

# 35. 1일차 최종 프로젝트 구조

```text
subject13_edge_ai/
│
├── .venv/                     # Git 제외
│
├── configs/
│   └── settings.yaml
│
├── logs/                      # 실행 시 생성, Git 제외
│   └── day01_events.csv
│
├── reports/
│   └── day01_result.md
│
├── scripts/
│   ├── check_config.py
│   └── run_pipeline.py
│
├── src/
│   ├── config_loader.py
│   └── pipeline.py
│
├── .gitignore
├── README.md
└── requirements.txt
```

파일의 역할을 다시 확인합니다.

| 파일 | 역할 |
|---|---|
| `configs/settings.yaml` | 실험 조건과 실행 설정 |
| `src/config_loader.py` | 설정파일 읽기 |
| `src/pipeline.py` | 입력·처리·판정·로그 기능 |
| `scripts/check_config.py` | 설정 정상 여부 확인 |
| `scripts/run_pipeline.py` | 프로그램 실행 시작점 |
| `logs/day01_events.csv` | 실제 실행 결과 |
| `reports/day01_result.md` | 결과 비교와 해석 |
| `requirements.txt` | Python Package 재현 |
| `.gitignore` | Git에서 제외할 파일 |
| `README.md` | 프로젝트 실행 안내 |

---

---

# 36. 1일차 최종 복습 — 코드를 보지 않고 설명해 보기

이제 파일을 잠시 닫고 다음 질문에 답해 봅니다.

## 복습 질문

1. 1일차에서 실제 Camera 대신 무엇을 입력으로 사용했나요?
2. `settings.yaml`을 따로 만든 이유는 무엇인가요?
3. YAML 설정을 Python에서 읽는 파일은 무엇인가요?
4. 설정이 정상적으로 읽히는지만 확인하는 실행 파일은 무엇인가요?
5. 가상 입력을 만드는 함수는 무엇인가요?
6. NORMAL / WARNING을 판단하는 함수는 무엇인가요?
7. 실행 결과를 CSV에 저장하는 함수는 무엇인가요?
8. 전체 Pipeline을 반복 실행하는 시작 파일은 무엇인가요?
9. Threshold를 낮추면 일반적으로 WARNING은 늘어날까요, 줄어들까요?
10. `random.seed(13)`을 사용한 이유는 무엇인가요?
11. 터미널 출력과 CSV Log의 목적은 어떻게 다른가요?
12. 오늘의 가상 입력은 앞으로 무엇으로 바뀔 수 있나요?

먼저 자신의 말로 답한 뒤 아래 예시와 비교합니다.

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

1. `0.0 ~ 1.0` 사이의 가상 숫자를 사용했습니다.
2. Python 코드를 수정하지 않고 실행 조건을 바꿀 수 있도록 설정과 코드를 분리했습니다.
3. `src/config_loader.py`입니다.
4. `scripts/check_config.py`입니다.
5. `generate_input()`입니다.
6. `decide()`입니다.
7. `append_log()`입니다.
8. `scripts/run_pipeline.py`입니다.
9. 같은 입력이라면 Threshold가 낮아질수록 WARNING이 나올 가능성이 커집니다.
10. 비교 실험에서 같은 순서의 랜덤 입력을 재현하기 위해 사용했습니다.
11. 터미널은 현재 실행 상태를 바로 보는 용도이고, CSV Log는 실행 후 결과를 다시 확인하고 분석하기 위한 기록입니다.
12. Camera Frame이나 Sensor 입력으로 바뀔 수 있습니다.

</details>

---

# 37. 1일차 핵심을 한 문장으로 설명하기

다음 문장을 그대로 외우지 말고 자신의 말로 다시 설명해 봅니다.

> **1일차에는 PC에서 가상 입력을 만들어 처리하고, 기준값으로 상태를 판단하고, 결과를 화면에 출력하고 CSV에 기록하는 작은 Edge Pipeline을 만들었습니다.**

조금 더 구조적으로 말하면 다음과 같습니다.

```text
settings.yaml
        ↓
설정 읽기
        ↓
입력 생성
        ↓
전처리
        ↓
판단
        ↓
출력
        ↓
Log 기록
```

오늘 작성한 파일이 여러 개였던 이유도 이 흐름의 역할을 나누기 위해서입니다.

---

# 38. 2일차와 연결하기

1일차에는 Pipeline이 **PC에서 정상적으로 움직이는지** 확인했습니다.

2일차에는 프로그램의 목적이 갑자기 바뀌는 것이 아닙니다.

```text
1일차

PC
  ↓
settings.yaml
  ↓
가상 입력
  ↓
판단
  ↓
출력 / Log

        ↓ 같은 프로젝트를 이어서 사용

2일차

Raspberry Pi
  ↓
Linux / SSH
  ↓
프로젝트 실행환경 준비
  ↓
1일차 프로그램 실행 확인
```

즉, 2일차의 핵심 질문은 다음과 같습니다.

> **“PC에서 만든 이 프로그램을 Raspberry Pi라는 실제 Edge 장치로 옮기면 무엇을 준비하고 확인해야 하는가?”**

1일차의 Pipeline 흐름을 설명할 수 있다면 2일차로 넘어갈 준비가 된 것입니다.

---
