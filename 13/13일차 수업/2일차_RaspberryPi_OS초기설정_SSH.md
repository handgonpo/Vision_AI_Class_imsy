
> **오늘의 핵심:** 처음 받은 Raspberry Pi에 운영체제를 직접 설치하고, Network와 SSH를 준비한 뒤, 1일차에 만든 Edge Pipeline을 실제 Raspberry Pi에서 실행합니다.
>
> 실제 Wi-Fi 비밀번호, Raspberry Pi 비밀번호, 관리자 비밀번호, 인증키 등 민감한 정보는 모두 `xxxx`로 표시하며 README·보고서·Git에 기록하지 않습니다.

---

## 오늘 가장 중요한 질문

1일차에는 다음 흐름을 PC에서 만들었습니다.

```text
가상 입력
   ↓
Python 처리
   ↓
NORMAL / WARNING 판단
   ↓
터미널 출력
   ↓
CSV Log 기록
```

오늘은 이 프로그램의 기능을 바꾸는 날이 아닙니다.

먼저 학생이 직접 Raspberry Pi를 처음 사용할 수 있는 상태로 준비합니다.

```text
microSD 준비
→ Raspberry Pi OS 설치
→ Hostname / 사용자 / Wi-Fi / SSH 설정
→ 첫 부팅
→ Network 확인
```

그 다음 **1일차에 PC에서 만든 같은 프로그램을 Raspberry Pi에서 실행하려면 무엇을 확인하고 준비해야 하는지**를 배웁니다.

즉, 2일차는 단순히 SSH 명령 하나를 배우는 날이 아니라 **Raspberry Pi를 처음 세팅하고, PC에서 만든 프로젝트를 실제 Edge 장치에서 실행할 수 있는 상태까지 연결하는 날**입니다.

```text
1일차

[PC]
subject13_edge_ai
        ↓
가상 입력 → 판단 → 출력 → Log

                ↓ 실행 장소 이동

2일차

[Windows PC]
        ↓
Raspberry Pi Imager
        ↓
microSD에 Raspberry Pi OS 설치
        ↓
Hostname / 사용자 / Wi-Fi / SSH 설정
        ↓
[Raspberry Pi 첫 부팅]
        ↓
Network
        ↓
SSH
        ↓
[Raspberry Pi Linux]
        ↓
Python 가상환경
        ↓
subject13_edge_ai
        ↓
같은 Pipeline 실행
```

즉, 오늘의 핵심은 다음 한 문장으로 정리할 수 있습니다.

> **Raspberry Pi OS를 직접 설치하고 Network와 SSH를 준비한 뒤, PC에서 만든 프로그램을 실제 Edge 장치로 옮겨 같은 Pipeline을 실행할 수 있는 상태를 만드는 것**

---

## 오늘의 8시간 학습 흐름

아래 시간은 실습 속도에 따라 조금 달라질 수 있습니다.  
중요한 것은 모든 학생이 **OS 설치 → 첫 부팅 → SSH → 프로젝트 실행**의 전체 연결을 한 번 직접 경험하는 것입니다.

| 학습 블록 | 권장 시간 | 무엇을 배우나요? | 끝나면 할 수 있어야 하는 것 |
|---|---:|---|---|
| 1 | 30분 | 1일차 프로젝트와 2일차 연결 | 오늘 새 프로젝트를 만드는 것이 아니라는 점을 설명할 수 있다 |
| 2 | 60분 | microSD · Raspberry Pi Imager · OS 선택 | Raspberry Pi에 운영체제를 설치하는 이유와 과정을 설명할 수 있다 |
| 3 | 60분 | Hostname · 사용자 · Wi-Fi · SSH 설정 · 첫 부팅 | 자신의 Pi를 구분하고 Headless 접속 준비를 할 수 있다 |
| 4 | 75분 | Linux · Network · IP · SSH · 오류 확인 | PC에서 자신의 Raspberry Pi에 접속하고 실패 원인을 1차 분류할 수 있다 |
| 5 | 60분 | Architecture · Source 전달 | Source와 `.venv`를 구분하고 프로젝트를 Pi로 전달할 수 있다 |
| 6 | 60분 | Pi 전용 `.venv` · requirements · 1일차 Pipeline 실행 | PC에서 만든 Pipeline을 Raspberry Pi에서 실행할 수 있다 |
| 7 | 75분 | 장치 정보 코드 · Local 설정 · SCP · Mini Challenge | 장치 상태를 코드로 확인하고 환경 차이를 구분할 수 있다 |
| 8 | 60분 | 오류 추적 · 결과 기록 · Git · 복습 · 3일차 연결 | 3일차 Camera/GPIO 실습 준비 상태인지 스스로 확인할 수 있다 |

---

## 오늘 계속 구분해야 할 네 가지

```text
PC
→ 코드를 작성하고 관리하는 개발 환경

Raspberry Pi
→ 코드를 실제로 실행할 Edge 장치

SSH
→ PC에서 Raspberry Pi의 터미널을 원격으로 사용하는 방법

SCP
→ SSH 연결을 이용해 PC와 Raspberry Pi 사이에서 파일을 복사하는 방법
```

오늘 명령어가 많아 보여도 전부 외울 필요는 없습니다.

헷갈릴 때는 다음 질문부터 확인합니다.

```text
지금 나는 어느 장치에 있는가?
        ↓
어느 사용자로 접속했는가?
        ↓
어느 폴더에 있는가?
        ↓
어느 Python을 사용하고 있는가?
```

이 네 가지를 확인하는 습관이 2일차의 가장 중요한 기초입니다.

---

## 실습 전에 자신의 환경값 적어두기

아래 값은 학생마다 또는 교육장마다 다를 수 있습니다.

교재의 예시 값을 그대로 입력하지 말고 **자신에게 실제로 할당된 값**을 확인합니다.

```text
Raspberry Pi 사용자 이름 : ____________________

Raspberry Pi Hostname    : ____________________

Raspberry Pi IPv4        : ____________________

Wi-Fi SSID               : 수업 중 확인 / 공유 문서에는 `xxxx`

Wi-Fi 비밀번호           : 공유 문서·Git에 기록하지 않음 (`xxxx`)

Raspberry Pi 비밀번호    : 공유 문서·Git에 기록하지 않음 (`xxxx`)

PC 프로젝트 경로         : ____________________

Raspberry Pi 프로젝트 경로: ____________________
```

이 문서에서는 환경에 따라 달라지는 값을 다음처럼 표시합니다.

```text
<RPI_USER>
→ Raspberry Pi 사용자 이름

<RPI_HOSTNAME>
→ Raspberry Pi 장치 이름

<RPI_IP>
→ Raspberry Pi의 실제 IPv4 주소

<INTERNAL_REPO_URL>
→ 교육장에서 제공된 내부 GitLab/Gitea Repository 주소
```

예를 들어 실제 값이 다음과 같다면,

```text
사용자  : edgepi
Hostname: rpi13-01
IPv4    : 192.168.x.x
```

다음 명령의

```bash
ssh <RPI_USER>@<RPI_IP>
```

`<RPI_USER>`와 `<RPI_IP>`를 자신의 값으로 바꾸어 실행합니다.

---

## 오늘의 성공 기준

수업이 끝났을 때 다음 흐름이 실제로 연결되면 됩니다.

```text
microSD 준비
 ↓
Raspberry Pi Imager로 OS 설치
 ↓
Hostname / 사용자 / Wi-Fi / SSH 설정
 ↓
Raspberry Pi 첫 부팅
 ↓
Raspberry Pi의 IP 확인
 ↓
PC에서 SSH 접속
 ↓
프로젝트 전달
 ↓
Raspberry Pi 전용 .venv 생성
 ↓
requirements.txt 설치
 ↓
1일차 check_config.py 실행
 ↓
1일차 run_pipeline.py 실행
 ↓
Raspberry Pi 장치 정보 확인
 ↓
device.local.yaml 적용
 ↓
결과와 오류 기록
```

3일차에는 이 상태 위에 **USB Camera와 GPIO**를 연결합니다.

# 1. 1일차 프로젝트에서 이어서 시작하기

1일차가 끝났다면 PC에는 다음 프로젝트가 있습니다.

```text
~/ai_vision/subject13_edge_ai/
├── configs/
│   └── settings.yaml
├── reports/
│   └── day01_result.md
├── scripts/
│   ├── check_config.py
│   └── run_pipeline.py
├── src/
│   ├── config_loader.py
│   └── pipeline.py
├── .gitignore
├── README.md
└── requirements.txt
```

1일차에는 이 코드를 PC에서 실행했습니다.

```text
[PC]
가상 입력
→ Python
→ Decision
→ NORMAL / WARNING
→ CSV Log
```

2일차에는 **같은 프로젝트의 실행 장소만 Raspberry Pi로 옮깁니다.**

```text
[PC]
   │
   │ SSH
   ▼
[Raspberry Pi]
   │
   ├─ Linux
   ├─ Python
   └─ subject13_edge_ai 실행
```

## 1일차에서 반드시 남아 있어야 하는 것

2일차를 시작하기 전에 1일차 프로젝트가 기본 상태로 돌아와 있어야 합니다.

특히 Mini Challenge에서 `CAUTION`을 추가했다면 1일차 마지막에 다시 다음 상태로 복원했습니다.

```text
settings.yaml
→ warning_threshold 하나 사용

pipeline.py
→ NORMAL / WARNING 두 상태 판단

run_pipeline.py
→ 1일차 기본 Pipeline 실행
```

먼저 PC에서 다음 두 명령이 정상 실행되는지 확인합니다.

```bash
cd ~/ai_vision/subject13_edge_ai
source .venv/bin/activate

python -m scripts.check_config
python -m scripts.run_pipeline
```

정상이라면 2일차를 시작합니다.

이 단계에서 오류가 난다면 Raspberry Pi로 옮기기 전에 먼저 **PC의 1일차 상태를 정상으로 복구**합니다.

```text
PC에서 정상 실행
        ↓
Raspberry Pi로 이동

PC에서 이미 오류
        ↓
먼저 PC 문제 해결
        ↓
그 다음 Raspberry Pi로 이동
```

이렇게 해야 Raspberry Pi에서 오류가 났을 때

```text
원래 코드 문제인가?
Raspberry Pi 환경 문제인가?
```

를 구분하기 쉬워집니다.

---


# 2. Raspberry Pi를 처음 받았을 때 무엇부터 준비할까요?

오늘은 Raspberry Pi가 이미 완성된 상태라고 가정하지 않습니다.

직접 다음 과정을 경험합니다.

```text
Raspberry Pi 본체 확인
        ↓
microSD 준비
        ↓
Windows PC에 microSD 연결
        ↓
Raspberry Pi Imager 준비
        ↓
Raspberry Pi OS 설치
        ↓
초기 설정 입력
        ↓
Raspberry Pi 첫 부팅
```

이 과정이 필요한 이유는 Raspberry Pi도 하나의 컴퓨터이기 때문입니다.

일반 PC가 운영체제 없이 바로 프로그램을 실행할 수 없는 것처럼 Raspberry Pi도 먼저 운영체제가 필요합니다.

오늘 사용할 운영체제는 **Raspberry Pi OS**입니다.

## 준비물 확인

실습을 시작하기 전에 다음을 확인합니다.

```text
Raspberry Pi
microSD 카드
microSD 카드 리더기
Raspberry Pi 전원 어댑터
Windows PC
교육장에서 사용할 Network 정보
```

아직 Camera와 GPIO 부품은 사용하지 않습니다.

```text
2일차
→ Raspberry Pi 자체를 실행 가능한 Edge 컴퓨터로 준비

3일차
→ 준비된 Raspberry Pi에 Camera / GPIO 연결
```

## microSD는 무슨 역할을 할까요?

오늘 사용하는 Raspberry Pi에서는 microSD에 운영체제를 설치합니다.

```text
microSD
   ↓
Raspberry Pi OS
   ↓
Raspberry Pi 부팅
   ↓
Linux 실행
   ↓
Python 프로그램 실행
```

따라서 microSD는 단순히 사진을 저장하는 메모리 카드로만 사용하는 것이 아닙니다.

오늘 실습에서는 **Raspberry Pi가 부팅할 운영체제가 들어가는 저장장치**가 됩니다.

## 기존 microSD를 다시 사용할 때 주의하기

Raspberry Pi Imager로 새 OS를 기록하면 선택한 저장장치의 기존 내용이 지워집니다.

기존 자료가 필요한 경우 먼저 백업합니다.

```text
기존 자료 필요
→ 먼저 백업

백업 완료
→ Raspberry Pi Imager로 새 OS 기록
```

Windows 탐색기에서 microSD의 파일을 하나씩 삭제한 뒤 설치할 필요는 없습니다.

Raspberry Pi Imager가 선택한 저장장치를 Raspberry Pi OS에 맞게 다시 구성합니다.

> **가장 중요한 확인:** OS를 기록하기 전에 선택한 저장장치가 정말 실습용 microSD인지 다시 확인합니다.

---

# 3. Raspberry Pi Imager로 운영체제 준비하기

## Raspberry Pi Imager는 무엇인가요?

Raspberry Pi Imager는 microSD에 Raspberry Pi OS를 설치하기 위한 도구입니다.

흐름은 다음과 같습니다.

```text
Windows PC
   ↓
Raspberry Pi Imager
   ↓
Raspberry Pi OS 선택
   ↓
microSD 선택
   ↓
초기 설정 입력
   ↓
OS 기록
```

## 1. microSD를 Windows PC에 연결하기

```text
microSD
→ 카드 리더기
→ Windows PC USB
```

Windows에서 새로운 저장장치가 보이는지 확인합니다.

드라이브 문자는 PC마다 다를 수 있으므로 `G:` 같은 예시를 그대로 외우지 않습니다.

## 2. Raspberry Pi Imager 설치하기

Raspberry Pi 공식 사이트에서 Raspberry Pi Imager를 설치합니다.

설치 후 실행합니다.

Imager의 화면 구성이나 버튼 위치는 버전에 따라 달라질 수 있습니다.

따라서 화면 모양을 그대로 외우기보다 다음 세 항목을 찾습니다.

```text
장치(Device)
운영체제(OS)
저장장치(Storage)
```

## 3. 사용할 Raspberry Pi 모델 선택하기

수업에서 지급받은 실제 Raspberry Pi 모델을 선택합니다.

예를 들어 실습 장비가 Raspberry Pi 4라면 Raspberry Pi 4를 선택합니다.

```text
내가 받은 Raspberry Pi 모델 확인
        ↓
Imager에서 같은 장치 선택
```

모델을 모르면 본체를 보고 임의로 고르지 말고 장비 정보를 먼저 확인합니다.

## 4. 운영체제 선택하기

본 실습에서는 Headless 방식으로 SSH 접속하여 작업하므로 교육 자료 기준으로 다음 OS를 사용합니다.

```text
Raspberry Pi OS Lite (64-bit)
```

Imager 버전에 따라 다음과 같이 들어갈 수 있습니다.

```text
Raspberry Pi OS (other)
        ↓
Raspberry Pi OS Lite (64-bit)
```

`Lite`는 데스크톱 GUI보다 터미널 중심으로 사용하는 환경입니다.

오늘부터 대부분의 작업은 PC에서 SSH로 접속하여 진행합니다.

## 5. 저장장치 선택하기

microSD가 연결된 저장장치를 선택합니다.

이 단계에서는 특히 주의합니다.

```text
내 PC의 다른 USB 저장장치
≠
실습용 microSD
```

잘못된 저장장치를 선택하면 그 저장장치의 내용이 지워질 수 있습니다.

선택 후 용량과 장치 이름을 다시 확인합니다.

---

# 4. Hostname · 사용자 · Wi-Fi · SSH를 처음부터 설정하기

운영체제를 기록하기 전에 Raspberry Pi가 첫 부팅 후 사용할 기본 정보를 설정합니다.

이 설정을 미리 넣어두면 모니터와 키보드를 Raspberry Pi에 직접 연결하지 않고도 Network와 SSH를 통해 접속할 수 있습니다.

```text
Imager에서 초기 설정
        ↓
microSD에 함께 저장
        ↓
Raspberry Pi 첫 부팅
        ↓
설정 적용
        ↓
Wi-Fi 연결
        ↓
SSH 접속 가능
```

## 1. Hostname 설정하기

Hostname은 Raspberry Pi의 **장치 이름**입니다.

학생이 여러 명이면 모든 Raspberry Pi에 같은 Hostname을 사용하지 않습니다.

예:

```text
학생 01 → rpi13-01
학생 02 → rpi13-02
학생 03 → rpi13-03
...
학생 30 → rpi13-30
```

자신에게 지정된 번호를 사용합니다.

```text
내 Hostname: ____________________
```

왜 서로 다르게 만들까요?

```text
같은 수업실
   ↓
여러 Raspberry Pi
   ↓
각 장치를 구분해야 함
   ↓
서로 다른 Hostname 사용
```

## 2. 사용자 이름과 비밀번호 설정하기

Raspberry Pi에서 사용할 사용자 계정을 만듭니다.

이 문서에서는 실제 값을 공개하지 않고 다음처럼 표시합니다.

```text
사용자 이름 : <RPI_USER>
비밀번호    : xxxx
```

비밀번호는 화면 캡처, README, Git Commit, 보고서에 기록하지 않습니다.

> 비밀번호를 입력하는 화면이나 SSH 로그인 중에는 입력한 문자가 화면에 보이지 않을 수 있습니다.

## 3. 지역과 시간대 설정하기

지역, 시간대, 키보드 설정은 교육장 환경에 맞게 선택합니다.

예를 들어 대한민국에서 수업한다면 서울 시간대를 기준으로 설정할 수 있습니다.

중요한 것은 메뉴 위치를 외우는 것이 아니라 **현재 수업 지역에 맞는 설정인지 확인하는 것**입니다.

## 4. Wi-Fi 설정하기

교육장에서 제공한 Wi-Fi 정보를 입력합니다.

교재에는 실제 값을 적지 않습니다.

```text
SSID     : xxxx
비밀번호 : xxxx
```

학생 개인 보고서나 Git에도 Wi-Fi 비밀번호를 기록하지 않습니다.

교육장에서 유선 LAN을 사용하도록 제공된 경우에는 현장 안내에 따릅니다.

## 5. SSH 활성화하기

오늘 수업의 핵심은 PC에서 Raspberry Pi로 원격 접속하는 것입니다.

따라서 Imager 초기 설정에서 SSH를 활성화합니다.

초기 실습에서는 교육 환경에서 안내한 인증 방식을 사용합니다.

```text
SSH 사용
→ 활성화

인증 정보
→ 교육장에서 안내한 방식 사용

비밀번호 예시
→ xxxx
```

SSH가 활성화되어야 다음 연결을 만들 수 있습니다.

```text
PC
 ↓
Network
 ↓
SSH
 ↓
Raspberry Pi Linux Terminal
```

## 설정을 완료하기 전에 다시 확인하기

```text
[ ] Raspberry Pi 모델이 맞다
[ ] Raspberry Pi OS Lite (64-bit)를 선택했다
[ ] 저장장치가 실습용 microSD가 맞다
[ ] Hostname이 내 번호와 맞다
[ ] 사용자 이름을 확인했다
[ ] 비밀번호는 기억하고 있지만 문서에는 기록하지 않았다
[ ] Wi-Fi 정보가 교육장 정보와 맞다
[ ] SSH를 활성화했다
```

---

# 5. microSD에 OS를 기록하고 Raspberry Pi를 처음 부팅하기

## 1. OS 기록 시작하기

설정을 다시 확인한 뒤 Imager에서 기록을 시작합니다.

기록 전에 저장장치의 데이터가 삭제된다는 경고가 나오면 **선택한 장치가 microSD인지 다시 확인**합니다.

```text
장치 확인
        ↓
기록 시작
        ↓
OS 파일 쓰기
        ↓
검증
        ↓
완료
```

기록이 끝날 때까지 microSD를 임의로 분리하지 않습니다.

## 2. microSD를 안전하게 제거하기

기록과 검증이 끝난 뒤 Windows에서 저장장치를 안전하게 제거합니다.

그 다음 카드 리더기에서 microSD를 빼냅니다.

## 3. Raspberry Pi에 microSD 장착하기

전원이 연결되지 않은 상태에서 microSD를 Raspberry Pi의 microSD 슬롯에 장착합니다.

```text
전원 OFF
   ↓
microSD 장착
   ↓
전원 연결
```

## 4. Raspberry Pi 첫 전원 켜기

전원을 연결합니다.

LED가 켜지고 저장장치를 읽기 시작하면 부팅 과정이 시작된 것입니다.

처음 부팅은 설정 적용 때문에 시간이 걸릴 수 있습니다.

바로 SSH 접속을 반복하기보다 먼저 몇 분 기다립니다.

```text
microSD 장착
        ↓
전원 ON
        ↓
Raspberry Pi OS 부팅
        ↓
Hostname / 사용자 설정 적용
        ↓
Wi-Fi 설정 적용
        ↓
Network에서 IP 할당
        ↓
SSH 서비스 시작
```

이 순서를 이해하면 전원을 켠 직후 SSH가 되지 않는다고 바로 OS를 다시 설치하지 않게 됩니다.

## 5. 첫 부팅에서 기억할 것

```text
전원 ON
≠
즉시 SSH 가능
```

운영체제가 부팅되고 Network에 연결되고 SSH 서비스가 시작될 시간이 필요합니다.

약 3~5분 정도 기다린 뒤 연결을 확인합니다.

환경과 장치 상태에 따라 실제 시간은 달라질 수 있습니다.

## 처음부터 다시 설치해야 할까요?

SSH가 한 번 실패했다고 바로 microSD를 다시 기록하지 않습니다.

먼저 다음 순서로 확인합니다.

```text
전원
 ↓
부팅 대기
 ↓
Network
 ↓
IP
 ↓
SSH
```

OS 재설치는 이 기본 확인 이후에도 초기 설정 자체가 잘못된 것이 확인되었을 때 고려합니다.

---

## 초기 설정 복습 — 순서를 스스로 다시 맞춰보기

아래 항목을 올바른 순서로 적어 봅니다.

```text
A. SSH로 PC에서 접속한다
B. microSD에 Raspberry Pi OS를 기록한다
C. Raspberry Pi Imager를 실행한다
D. Raspberry Pi에 microSD를 장착한다
E. Hostname / 사용자 / Wi-Fi / SSH를 설정한다
F. Raspberry Pi에 전원을 연결한다
G. Network에서 IP가 할당될 때까지 기다린다
```

먼저 정답을 보지 않고 직접 순서를 적습니다.

```text
내가 생각한 순서:

___ → ___ → ___ → ___ → ___ → ___ → ___
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
C. Raspberry Pi Imager 실행
        ↓
E. Hostname / 사용자 / Wi-Fi / SSH 설정
        ↓
B. microSD에 Raspberry Pi OS 기록
        ↓
D. Raspberry Pi에 microSD 장착
        ↓
F. Raspberry Pi 전원 ON
        ↓
G. 부팅 / Network / IP 할당 대기
        ↓
A. PC에서 SSH 접속
```

가장 중요한 생각은 다음입니다.

```text
SSH는 맨 처음 하는 일이 아니다.

운영체제
→ Network
→ SSH 서비스

가 먼저 준비되어야 한다.
```

</details>

## 초기 설정 복습 — 학생 30명이 같은 Hostname을 사용하면?

다음 상황을 생각해 봅니다.

```text
학생 01 → rpi13
학생 02 → rpi13
학생 03 → rpi13
...
```

왜 문제가 될 수 있을까요?

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

같은 Network에 여러 Raspberry Pi가 존재하므로 장치를 이름으로 구분하기 어려워집니다.

따라서 다음처럼 학생별로 고유한 Hostname을 사용하는 편이 관리하기 쉽습니다.

```text
rpi13-01
rpi13-02
rpi13-03
...
```

Hostname은 **“내가 지금 어느 Raspberry Pi에 접속하려는가?”**를 구분하는 장치 이름입니다.

</details>

---

# 6. 지금 어느 장치에서 작업 중인지 확인하기

오늘부터 PC 터미널과 Raspberry Pi 터미널을 오가게 됩니다.

명령을 실행하기 전에 다음 세 가지를 자주 확인합니다.

```bash
whoami
hostname
pwd
```

예:

```text
learner@DESKTOP:~/ai_vision/subject13_edge_ai$
→ PC

edgepi@rpi13-01:~$
→ Raspberry Pi
```

각 명령의 의미:

```text
whoami
→ 현재 사용자

hostname
→ 현재 장치 이름

pwd
→ 현재 작업 폴더
```

명령이 예상대로 동작하지 않을 때는 먼저 **어느 장치의 어느 폴더에 있는지** 확인합니다.

## 잠깐 연습 — 프롬프트만 보고 어느 장치인지 구분하기

다음 두 줄을 봅니다.

```text
learner@DESKTOP:~/ai_vision/subject13_edge_ai$
edgepi@rpi13-01:~/ai_vision/subject13_edge_ai$
```

첫 번째는 보통 PC/WSL2 환경이고, 두 번째는 Raspberry Pi에 SSH 접속한 상태입니다.

하지만 프롬프트 모양만 외우지 않습니다.

확실하지 않으면 바로 다음 명령으로 확인합니다.

```bash
whoami
hostname
pwd
```

> **명령 실행 전에 “어느 장치인지” 확인하는 습관은 이후 Camera, GPIO, ONNX 실습에서 매우 중요합니다.**


---

# 7. Raspberry Pi 기본 상태 확인하기

앞 절에서 직접 Raspberry Pi OS를 설치하고 첫 부팅까지 진행했습니다.

이제 다음 상태가 실제로 만들어졌는지 확인합니다.

```text
Raspberry Pi OS 설치 완료
→ 사용자 계정 준비
→ Hostname 적용
→ Wi-Fi 또는 LAN 연결
→ IP 할당
→ SSH 서비스 준비
```

아직 PC에서 SSH로 접속하지 못했다면 아래 Network/SSH 확인 절차를 따라갑니다.

이미 SSH 접속에 성공했다면 Raspberry Pi 터미널에서 다음 명령을 실행합니다.

```bash
hostname
```

예:

```text
rpi13-01
```

IP 주소:

```bash
hostname -I
```

운영체제:

```bash
cat /etc/os-release
```

하드웨어 모델:

```bash
cat /proc/device-tree/model
```

CPU Architecture:

```bash
uname -m
```

Python:

```bash
python3 --version
```

현재 사용자와 경로:

```bash
whoami
pwd
```

다음처럼 구분해서 이해합니다.

| 명령 | 확인 내용 |
|---|---|
| `hostname` | 장치 이름 |
| `hostname -I` | IP 주소 |
| `cat /etc/os-release` | Linux 배포판 |
| `cat /proc/device-tree/model` | Raspberry Pi 모델 |
| `uname -m` | CPU Architecture |
| `python3 --version` | Python 버전 |
| `whoami` | 사용자 |
| `pwd` | 현재 위치 |

## 왜 이 정보를 먼저 확인할까요?

프로그램이 실행되지 않을 때 원인이 항상 Python 코드인 것은 아닙니다.

예를 들어 다음과 같은 문제가 있을 수 있습니다.

```text
잘못된 Raspberry Pi에 접속함
→ Hostname 확인

IP가 할당되지 않음
→ hostname -I 확인

32-bit / 64-bit 또는 Architecture 차이
→ uname -m 확인

Python이 예상과 다름
→ python3 --version 확인

잘못된 사용자·경로
→ whoami / pwd 확인
```

따라서 코드를 실행하기 전에 **장치의 신원과 실행환경을 먼저 확인**합니다.


---

# 8. Network 상태 확인하기

IPv4 주소를 확인합니다.

```bash
ip -4 addr
```

기본 Gateway를 확인합니다.

```bash
ip route
```

NetworkManager 환경이라면 다음도 사용할 수 있습니다.

```bash
nmcli device status
```

`nmcli`가 없는 환경에서는 다음 세 명령만으로도 기본 확인이 가능합니다.

```bash
hostname -I
ip -4 addr
ip route
```

## `hostname -I`에 주소가 여러 개 보이면

환경에 따라 IPv4와 IPv6가 함께 표시될 수 있습니다.

예:

```text
192.168.x.x  2406:xxxx:xxxx:....
```

오늘 SSH 실습에서는 교육장에서 안내한 **Raspberry Pi의 IPv4 주소**를 우선 사용합니다.

Wi-Fi 인터페이스가 `wlan0`인 경우 IPv4를 더 자세히 보려면 다음처럼 확인할 수 있습니다.

```bash
ip -4 addr show wlan0
```

`wlan0` 이름이 다르거나 보이지 않으면 먼저 다음으로 실제 인터페이스 이름을 확인합니다.

```bash
ip -4 addr
```

> IP는 고정된 교재 예시를 외우는 값이 아니라 **현재 Network가 Raspberry Pi에 할당한 주소를 확인해서 사용하는 값**입니다.


---

# 9. Hostname과 IP 구분하기

예:

```text
Hostname
rpi13-01

IP
<RPI_IP>
```

Hostname은 장치를 구분하기 위한 이름이고, IP는 Network에서 장치를 찾아가기 위한 주소입니다.

PC에서 Raspberry Pi에 접속할 때 IP를 직접 사용할 수 있습니다.

```bash
ssh <RPI_USER>@<RPI_IP>
```

환경에 따라 `.local` 이름이 정상적으로 동작하면 다음처럼 접속할 수도 있습니다.

```bash
ssh <RPI_USER>@<RPI_HOSTNAME>.local
```

`.local` 방식이 동작하지 않으면 IP를 사용합니다.

## Hostname과 IP 중 무엇을 먼저 사용할까요?

교육장에서는 다음 순서가 가장 단순합니다.

```text
1. 실제 IPv4를 확인할 수 있다
→ IP로 SSH 접속

2. Hostname/.local 이름 해석이 정상이다
→ Hostname으로 접속해도 됨

3. .local이 안 된다
→ Raspberry Pi 고장이라고 판단하지 않음
→ IP로 다시 접속
```

WSL2, Windows, 공유기, mDNS 설정에 따라 `.local` 이름이 어떤 환경에서는 되고 다른 환경에서는 안 될 수 있습니다.

따라서 **Hostname 접속 실패와 Raspberry Pi 자체 실패를 같은 문제로 보지 않습니다.**


---

# 10. PC에서 Raspberry Pi 연결 확인하기

이제 PC에서 Raspberry Pi까지 Network 경로가 열려 있는지 확인합니다.

이 단계의 목적은 단순합니다.

```text
PC
 ↓
Network
 ↓
Raspberry Pi
```

가 연결되는지 확인한 뒤,

```text
PC
 ↓
SSH
 ↓
Raspberry Pi Terminal
```

까지 들어가는 것입니다.

먼저 **PC의 WSL2 또는 Linux 터미널**에서 실행합니다.

```bash
ping -c 4 <RPI_IP>
```

응답이 오면 같은 Network에서 Raspberry Pi가 보인다는 뜻입니다.

하지만 `ping`이 실패했다고 바로 Raspberry Pi가 고장난 것은 아닙니다.

교육장 Network 정책에서 ICMP 응답을 막을 수도 있기 때문에 **최종 확인은 SSH로 합니다.**

```bash
ssh <RPI_USER>@<RPI_IP>
```

처음 접속하면 다음과 비슷한 메시지가 나타날 수 있습니다.

```text
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

자신이 접속하려는 Raspberry Pi의 IP가 맞는지 확인한 뒤 다음을 입력합니다.

```text
yes
```

비밀번호 인증을 사용하는 환경이라면 비밀번호를 입력합니다.

비밀번호를 입력할 때 화면에 글자가 표시되지 않는 것은 정상입니다.

접속에 성공하면 프롬프트가 다음처럼 바뀔 수 있습니다.

```text
edgepi@rpi13-01:~$
```

## SSH 오류 메시지는 무엇을 뜻할까요?

SSH가 안 될 때는 메시지의 마지막 부분을 먼저 봅니다.

| 대표 증상 | 먼저 의심할 영역 |
|---|---|
| `Connection timed out` | IP, Wi-Fi/LAN, Network 경로, 방화벽 |
| `No route to host` | Network 경로 또는 잘못된 IP |
| `Connection refused` | SSH 서비스가 실행되지 않거나 Port가 열리지 않음 |
| `Permission denied` | 사용자 이름, 비밀번호, 인증키 |
| `Could not resolve hostname` | Hostname 오타 또는 이름 해석 문제 |
| `REMOTE HOST IDENTIFICATION HAS CHANGED!` | 같은 IP/Hostname의 SSH Host Key가 이전과 달라짐 |

### Raspberry Pi OS를 다시 설치한 뒤 Host Key 경고가 나온 경우

Raspberry Pi를 재이미징하면 SSH Host Key가 바뀔 수 있습니다.

다음 경고가 보일 수 있습니다.

```text
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
```

먼저 **지금 접속하려는 장치가 정말 자신의 Raspberry Pi인지 IP와 Hostname을 다시 확인**합니다.

장치를 재설치한 것이 확실하고 주소도 맞다면 PC에서 기존 기록을 제거한 뒤 다시 접속할 수 있습니다.

```bash
ssh-keygen -R <RPI_IP>
```

그 다음 다시:

```bash
ssh <RPI_USER>@<RPI_IP>
```

> Host Key 경고를 이유 없이 무조건 삭제하지 않습니다. 장치가 맞는지 먼저 확인합니다.

## SSH가 안 될 때 바로 OS부터 다시 설치하지 않습니다

다음 순서로 범위를 좁힙니다.

```text
Raspberry Pi 전원 확인
        ↓
부팅 시간 충분히 기다림
        ↓
Raspberry Pi의 실제 IP 확인
        ↓
PC와 같은 Network인지 확인
        ↓
SSH 사용자 이름 확인
        ↓
SSH 재시도
        ↓
오류 메시지 분류
```

이 순서가 2일차 전체 Troubleshooting의 기본입니다.

---

# 11. SSH 접속 후 네 가지 확인

접속 직후 실행합니다.

```bash
whoami
hostname
hostname -I
pwd
```

예:

```text
edgepi
rpi13-01
<RPI_IP>
/home/edgepi
```

SSH 상태에서는 PC 화면에서 키보드를 사용하고 있지만 실제 명령은 Raspberry Pi에서 실행됩니다.

```text
[PC]
키보드 입력
   ↓
SSH
   ↓
[Raspberry Pi]
Linux 명령 실행
   ↓
결과 반환
   ↓
[PC Terminal]
```

## 접속한 뒤 가장 먼저 해야 하는 이유

SSH 연결에 성공했다는 사실만으로는 충분하지 않습니다.

예를 들어 실수로 옆 학생의 Raspberry Pi에 접속했을 수도 있습니다.

따라서 접속 직후 다음을 확인합니다.

```text
whoami
→ 내가 사용할 계정이 맞는가?

hostname
→ 내가 사용할 Raspberry Pi가 맞는가?

hostname -I
→ 현재 장치의 Network 주소는 무엇인가?

pwd
→ 지금 어느 폴더에서 작업을 시작하는가?
```

**접속 성공 → 장치 확인**까지 해야 SSH 확인이 끝난 것입니다.


---

# 12. PC와 Raspberry Pi의 Architecture 비교

Raspberry Pi에서:

```bash
uname -m
```

64-bit Raspberry Pi OS라면 예를 들어:

```text
aarch64
```

SSH에서 나옵니다.

```bash
exit
```

PC에서:

```bash
uname -m
```

WSL2 PC라면 예를 들어:

```text
x86_64
```

즉:

```text
PC
x86_64

Raspberry Pi
aarch64
```

처럼 CPU Architecture가 다를 수 있습니다.

이 때문에 **PC에서 만든 `.venv` 폴더를 Raspberry Pi로 그대로 복사하지 않습니다.**

```text
공유
→ Source Code
→ requirements.txt

각 장치에서 새로 생성
→ .venv
```

## `.venv`를 복사하면 안 되는 이유를 조금 더 쉽게 이해하기

가상환경에는 단순한 Python 소스코드만 들어 있는 것이 아닙니다.

설치된 Package와 실행 경로, 장치/운영체제에 영향을 받는 파일들이 들어갈 수 있습니다.

따라서 다음처럼 생각합니다.

```text
PC에서 공유해도 되는 것
→ .py 소스코드
→ .yaml 공통 설정
→ requirements.txt
→ README / reports

PC에서 Raspberry Pi로 그대로 복사하지 않는 것
→ .venv
```

즉,

```text
requirements.txt
= "필요한 Package 목록"

.venv
= "그 장치에서 실제로 만들어진 실행환경"
```

입니다.

PC와 Raspberry Pi의 Architecture가 다를 수 있으므로 **각 장치에서 자기 `.venv`를 새로 만듭니다.**


---

# 13. 1일차 프로젝트를 Raspberry Pi로 보내기 전에 PC 상태 확인하기

Raspberry Pi로 프로젝트를 보내기 전에 PC의 프로젝트가 정상 상태인지 확인합니다.

PC에서:

```bash
cd ~/ai_vision/subject13_edge_ai
git status
git log --oneline -5
```

`git status`에서 수정 중인 파일이 있다면 무엇이 바뀌었는지 먼저 확인합니다.

```bash
git diff
```

필요한 변경이라면 Commit합니다.

```bash
git add .
git commit -m "chore: prepare day02 raspberry pi deployment"
```

단, `.gitignore`에 의해 다음 항목은 Git 대상에서 제외되어 있어야 합니다.

```text
.venv/
logs/
configs/*.local.yaml   # 2일차에서 추가
```

## 프로젝트 전달 방법은 교육장 환경에 따라 선택합니다

2일차에는 **두 방법을 모두 실행하는 것이 아닙니다.**

교육장 환경에 맞는 한 가지 방법을 선택합니다.

```text
방법 A
내부 GitLab / Gitea 같은 Remote Repository 사용 가능
        ↓
PC에서 Push
        ↓
Raspberry Pi에서 Clone

방법 B
Remote Repository를 사용할 수 없음
        ↓
SSH 연결 확인
        ↓
SCP로 필요한 Source 파일 전달
```

외부 GitHub 사용이 제한된 환경에서는 GitHub 주소를 임의로 사용하지 않습니다.

오늘의 핵심은 특정 서비스가 아니라 다음입니다.

> **PC에서 만든 Source Code와 설정 구조를 Raspberry Pi로 전달한다.**

---

## 방법 A를 사용하는 경우 — 내부 Remote 최신 상태 확인

내부 GitLab/Gitea Repository가 연결되어 있다면 확인합니다.

```bash
git remote -v
```

Remote 주소가 교육장에서 제공된 주소인지 확인한 뒤 Push합니다.

```bash
git push
```

이제 흐름은 다음과 같습니다.

```text
PC Source
   ↓
Local Git Commit
   ↓
내부 Remote Repository
   ↓
Raspberry Pi Clone
```

Remote가 없는 환경에서는 이 Push를 생략하고 다음 절의 **방법 B — SCP**를 사용합니다.

---

# 14. 같은 1일차 프로젝트를 Raspberry Pi에 준비하기

새로운 `subject13_edge_ai` 프로젝트를 다시 만들지 않습니다.

**1일차에 만든 같은 프로젝트를 Raspberry Pi에 가져옵니다.**

먼저 Raspberry Pi에 접속합니다.

```bash
ssh <RPI_USER>@<RPI_IP>
```

작업용 상위 폴더를 준비합니다.

```bash
mkdir -p ~/ai_vision
cd ~/ai_vision
```

## 방법 A — 내부 Remote Repository가 있는 경우

Git이 설치되어 있는지 확인합니다.

```bash
git --version
```

Git이 없다면 교육장 Package Repository 또는 인터넷 사용이 가능한 환경에서 설치합니다.

```bash
sudo apt update
sudo apt install -y git
```

> 교육장 Network가 외부 인터넷을 차단하는 경우 `apt`가 실패할 수 있습니다. 이 경우 코드를 잘못 작성한 문제가 아니라 Package 설치 경로 문제이므로 교육장에서 제공한 내부 Mirror 또는 사전 설치 환경을 사용합니다.

내부 Repository를 Clone합니다.

```bash
git clone <INTERNAL_REPO_URL>
```

예를 들어 Repository 이름이 `subject13_edge_ai`라면 다음 폴더가 생깁니다.

```text
~/ai_vision/subject13_edge_ai
```

이동합니다.

```bash
cd ~/ai_vision/subject13_edge_ai
```

확인합니다.

```bash
pwd
git log --oneline -5
git remote -v
```

PC에서 보았던 Commit 기록이 보이면 같은 프로젝트를 가져온 것입니다.

---

## 방법 B — Remote Repository가 없는 경우 SCP로 Source 전달

이 방법에서는 먼저 Raspberry Pi에 빈 프로젝트 폴더만 준비합니다.

Raspberry Pi에서:

```bash
mkdir -p ~/ai_vision/subject13_edge_ai
exit
```

이제 **PC의 프로젝트 루트**로 이동합니다.

```bash
cd ~/ai_vision/subject13_edge_ai
```

필요한 Source와 문서만 Raspberry Pi로 보냅니다.

```bash
scp -r \
  configs \
  reports \
  scripts \
  src \
  .gitignore \
  README.md \
  requirements.txt \
  <RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/
```

다시 Raspberry Pi에 접속합니다.

```bash
ssh <RPI_USER>@<RPI_IP>
cd ~/ai_vision/subject13_edge_ai
```

확인합니다.

```bash
ls
```

다음 항목이 보이는지 확인합니다.

```text
configs
reports
scripts
src
.gitignore
README.md
requirements.txt
```

이 방법에서는 PC의 `.venv`와 `logs/`를 보내지 않습니다.

또한 `.git` 저장소 자체도 SCP 대상에 넣지 않았으므로 **Version 관리는 PC의 Local Git을 기준으로 유지**합니다.

## 두 방법의 차이

```text
방법 A — 내부 Remote
PC Commit → Push → Raspberry Pi Clone
→ Git 이력까지 Raspberry Pi에 존재

방법 B — SCP
PC Source → SCP → Raspberry Pi
→ 실행에 필요한 파일만 전달
→ Version 관리는 PC Local Git 중심
```

어느 방법을 사용해도 오늘의 핵심 실습인 **Raspberry Pi에서 1일차 프로그램 실행**은 동일하게 진행할 수 있습니다.

---

# 15. Raspberry Pi 전용 `.venv` 만들기

Clone 후 다음을 확인합니다.

```bash
ls -a
```

`.venv`가 없는 것이 정상입니다.

Raspberry Pi에서 새로 만듭니다.

필요한 경우:

```bash
sudo apt install -y python3-venv
```

가상환경 생성:

```bash
python3 -m venv .venv
```

활성화:

```bash
source .venv/bin/activate
```

확인:

```bash
which python
```

예:

```text
/home/edgepi/ai_vision/subject13_edge_ai/.venv/bin/python
```

이제 `requirements.txt`로 Package를 재현합니다.

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

PyYAML 확인:

```bash
python -c "import yaml; print(yaml.__version__)"
```

핵심:

```text
PC .venv 복사
X

Raspberry Pi에서
requirements.txt로 새 .venv 구성
O
```

## 이 단계는 왜 필요할까요?

지금 Raspberry Pi에는 Source Code는 있지만 아직 **이 프로젝트 전용 Python 실행환경**이 없습니다.

```text
Source Code 준비
        ↓
Raspberry Pi용 .venv 생성
        ↓
requirements.txt를 보고 Package 설치
        ↓
프로그램 실행 가능
```

### 명령 흐름을 의사코드로 읽기

```text
Raspberry Pi 프로젝트 폴더로 이동한다
        ↓
python3가 설치되어 있는지 확인한다
        ↓
.venv를 새로 만든다
        ↓
.venv를 활성화한다
        ↓
현재 python 경로를 확인한다
        ↓
requirements.txt를 읽어 필요한 Package를 설치한다
        ↓
PyYAML이 정상 import되는지 확인한다
```

## `pip install`이 실패한다면

오류를 먼저 구분합니다.

```text
"No matching distribution"
→ Package/Version/Architecture 문제 가능성

"Temporary failure in name resolution"
→ DNS / Network 문제 가능성

"Connection timed out"
→ Network 또는 Package Repository 접근 문제 가능성

"externally-managed-environment"
→ 시스템 Python에 직접 설치하려 했는지 확인
→ .venv 활성화 여부 확인
```

교육장에서 인터넷이 제한된 경우에는 외부 PyPI 접속을 반복 시도하지 않습니다.

교육장에서 제공한 내부 Package Mirror 또는 사전에 준비된 wheel 파일이 있다면 그 방법을 사용합니다.

오늘은 1일차 `requirements.txt`의 Package가 많지 않기 때문에 **환경 재현 원리를 이해하는 것**에 집중합니다.


---

# 16. 1일차 프로그램을 Raspberry Pi에서 실행하기

설정 확인:

```bash
python -m scripts.check_config
```

1일차 Pipeline 실행:

```bash
python -m scripts.run_pipeline
```

다음과 같은 결과가 다시 나오는지 확인합니다.

```text
value=...
NORMAL / WARNING
Mean Latency
P95 Latency
Approx. FPS
```

1일차와 프로그램은 같지만 실행 장치가 달라졌습니다.

```text
1일차
PC에서 실행

2일차
Raspberry Pi에서 실행
```

## 여기서 무엇을 확인한 것일까요?

중요한 점은 새로운 프로그램을 만든 것이 아니라는 것입니다.

```text
같은 Source Code
        +
같은 공통 설정
        ↓

PC에서도 실행
Raspberry Pi에서도 실행
```

즉, 1일차에 만든 코드가 특정 PC 한 대에만 묶여 있지 않고 **다른 Linux 장치에서도 실행 가능한 형태로 구성되었는지** 확인한 것입니다.

### PC와 Pi 결과를 비교할 때 무엇을 보나요?

오늘은 Latency 숫자의 크기를 성능 비교에 사용하지 않습니다.

다음만 확인합니다.

```text
설정파일을 읽는가?
        ↓
가상 입력이 생성되는가?
        ↓
NORMAL / WARNING이 나오는가?
        ↓
CSV Log가 생성되는가?
        ↓
프로그램이 끝까지 종료되는가?
```

실제 Pi + ONNX 성능 비교는 뒤의 성능 측정 수업에서 진행합니다.

## 코드 리뷰 — Raspberry Pi에서 실행했지만 어느 코드를 사용했을까요?

프로그램 실행 후 다음 파일을 다시 열어 확인합니다.

```text
scripts/run_pipeline.py
→ 전체 실행 시작점

src/pipeline.py
→ 입력 생성 / 전처리 / 판단 / Log

src/config_loader.py
→ 설정파일 읽기

configs/settings.yaml
→ 공통 설정값
```

질문:

```text
Raspberry Pi용 run_pipeline.py를 새로 만들었는가?
→ 아니다.

같은 run_pipeline.py를 사용했는가?
→ 그렇다.

달라진 것은 무엇인가?
→ 실행 장치와 Python 환경
```

이 구분이 오늘의 핵심입니다.


---

# 17. Raspberry Pi 장치 정보를 Python에서 읽기

다음 파일을 추가합니다.

```text
src/device_info.py
```

코드:

```python
import platform
import socket
import subprocess
from pathlib import Path


def get_hostname() -> str:
    return socket.gethostname()


def get_ip_addresses() -> str:
    try:
        result = subprocess.run(
            ["hostname", "-I"],
            capture_output=True,
            text=True,
            check=True,
        )
        return result.stdout.strip()
    except Exception:
        return "UNKNOWN"


def get_os_name() -> str:
    return platform.platform()


def get_architecture() -> str:
    return platform.machine()


def get_python_version() -> str:
    return platform.python_version()


def get_current_path() -> str:
    return str(Path.cwd())


def config_exists() -> bool:
    return Path("configs/settings.yaml").exists()
```

이 파일은 Linux 장치의 상태를 Python 프로그램 안에서 읽기 위한 모듈입니다.

## 이 코드는 왜 만들까요?

터미널 명령으로 장치 정보를 확인할 수 있지만, 이후 프로그램이 커지면 **Python 코드 안에서도 현재 장치의 상태를 확인해야 하는 경우**가 생깁니다.

예를 들어 앞으로 다음 정보를 Log나 상태 화면에 사용할 수 있습니다.

```text
어느 장치에서 실행 중인가?
어떤 Architecture인가?
어떤 Python을 사용하는가?
설정파일이 존재하는가?
```

그래서 Linux 명령과 Python 프로그램 사이를 연결하는 작은 모듈을 만듭니다.

## 의사코드

```text
장치의 Hostname을 읽는 함수를 만든다
        ↓
장치의 IP 주소를 읽는 함수를 만든다
        ↓
운영체제 이름을 읽는 함수를 만든다
        ↓
CPU Architecture를 읽는 함수를 만든다
        ↓
Python 버전을 읽는 함수를 만든다
        ↓
현재 실행 폴더를 읽는 함수를 만든다
        ↓
configs/settings.yaml 파일이 존재하는지 확인한다
        ↓
각 함수는 확인한 값을 호출한 코드에 반환한다
```

`device_info.py`는 화면에 직접 출력하는 파일이 아닙니다.

```text
device_info.py
→ 정보를 수집하는 기능

check_device.py
→ 그 기능을 호출하고 결과를 화면에 보여주는 실행 프로그램
```

역할을 분리해서 이해합니다.


---

# 18. `check_device.py` 만들기

다음 파일을 만듭니다.

```text
scripts/check_device.py
```

코드:

```python
from src.device_info import (
    config_exists,
    get_architecture,
    get_current_path,
    get_hostname,
    get_ip_addresses,
    get_os_name,
    get_python_version,
)


def main():
    print("=== Raspberry Pi Device Check ===")
    print(f"Hostname      : {get_hostname()}")
    print(f"IP            : {get_ip_addresses()}")
    print(f"OS            : {get_os_name()}")
    print(f"Architecture  : {get_architecture()}")
    print(f"Python        : {get_python_version()}")
    print(f"Current Path  : {get_current_path()}")
    print(f"Config Exists : {config_exists()}")


if __name__ == "__main__":
    main()
```

실행:

```bash
python -m scripts.check_device
```

예:

```text
=== Raspberry Pi Device Check ===
Hostname      : rpi13-01
IP            : <RPI_IP>
OS            : Linux-...
Architecture  : aarch64
Python        : 3.x.x
Current Path  : /home/edgepi/ai_vision/subject13_edge_ai
Config Exists : True
```

터미널 명령으로 보던 정보를 프로그램 내부에서도 읽을 수 있게 되었습니다.

## 이 코드는 왜 만들까요?

앞에서 만든 `device_info.py`는 정보를 **수집하는 함수 모음**입니다.

이제 실제로 함수를 호출하여 한 화면에서 장치 상태를 확인할 실행 프로그램이 필요합니다.

```text
device_info.py
장치 정보 수집
        ↓
check_device.py
함수 호출
        ↓
터미널에 결과 출력
```

## 의사코드

```text
device_info.py에서 필요한 함수를 가져온다
        ↓
main 함수를 시작한다
        ↓
"Raspberry Pi Device Check" 제목을 출력한다
        ↓
Hostname을 출력한다
        ↓
IP 주소를 출력한다
        ↓
OS를 출력한다
        ↓
Architecture를 출력한다
        ↓
Python 버전을 출력한다
        ↓
현재 프로젝트 경로를 출력한다
        ↓
설정파일 존재 여부를 출력한다
        ↓
이 파일을 직접 실행했다면 main 함수를 실행한다
```

## 실행 결과를 코드에서 다시 찾아보기

실행 결과가 다음처럼 나왔다고 가정합니다.

```text
Hostname      : rpi13-01
IP            : 192.168.x.x
Architecture  : aarch64
Config Exists : True
```

각 값이 어디에서 만들어지는지 코드에서 찾습니다.

| 화면의 결과 | `check_device.py`에서 호출 | 실제 정보를 얻는 위치 |
|---|---|---|
| Hostname | `get_hostname()` | `socket.gethostname()` |
| IP | `get_ip_addresses()` | `hostname -I` 명령 |
| Architecture | `get_architecture()` | `platform.machine()` |
| Config Exists | `config_exists()` | `Path(...).exists()` |

> 화면 결과를 보고 **어느 함수가 그 값을 만들었는지 다시 찾아보는 것**이 코드 리뷰의 목적입니다.


---

# 19. Raspberry Pi 자원 상태 확인하기

CPU:

```bash
lscpu
```

Memory:

```bash
free -h
```

Disk:

```bash
df -h
```

현재 Load:

```bash
uptime
```

Raspberry Pi OS에서 지원한다면 온도:

```bash
vcgencmd measure_temp
```

오늘은 성능을 분석하지 않습니다.

다만 Raspberry Pi도 다음 자원을 가진 Linux 컴퓨터라는 것을 직접 확인합니다.

```text
CPU
Memory
Storage
Temperature
Network
Process
```

이 값들은 8~9일차 성능 실험에서 다시 사용합니다.

## 오늘 이 숫자들을 성능 평가에 사용하지 않습니다

`free -h`, `df -h`, `uptime`, 온도 값은 오늘 다음 사실만 확인하기 위한 자료입니다.

```text
Raspberry Pi도
CPU / Memory / Disk / Network를 가진
하나의 Linux 컴퓨터이다.
```

8~9일차에는 이 정보가 실제 Latency/FPS와 어떤 관계가 있는지 다시 살펴봅니다.

오늘은 장치가 비정상 상태인지 확인하는 **기초 상태 확인 명령**으로 기억합니다.


---

# 20. 장치별 설정을 공통 설정과 분리하기

1일차 `configs/settings.yaml`에는 PC에서 사용하던 장치 이름이 들어 있을 수 있습니다.

예:

```yaml
device_name: learner01-pc
```

Raspberry Pi에서는 실제 Hostname을 사용하고 싶습니다.

그러나 공통 `settings.yaml`을 매번 장치에 맞춰 Commit하면 PC와 Raspberry Pi가 계속 서로의 값을 덮어쓰게 됩니다.

그래서 Raspberry Pi 전용 설정파일을 하나 만듭니다.

```text
configs/device.local.yaml
```

내용:

```yaml
device_name: rpi13-01
```

자신의 실제 Hostname에 맞게 작성합니다.

## 왜 `settings.yaml` 하나만 계속 바꾸면 안 될까요?

같은 Repository를 PC와 Raspberry Pi가 함께 사용한다고 생각해 봅니다.

```text
PC
device_name = learner01-pc

Raspberry Pi
device_name = rpi13-01
```

두 장치가 같은 `settings.yaml`을 계속 수정하고 Commit하면 장치 이름 하나 때문에 불필요한 변경이 반복될 수 있습니다.

그래서 다음처럼 나눕니다.

```text
모든 장치가 함께 사용할 값
→ settings.yaml

그 장치에서만 사용할 값
→ device.local.yaml
```

이것은 앞으로 Camera 번호, Port, 장치 경로처럼 **환경마다 달라질 수 있는 값**을 관리할 때도 사용할 수 있는 방식입니다.


---

# 21. Local 설정파일을 Git에서 제외하기

`.gitignore`에 다음을 추가합니다.

```gitignore
# Device specific settings
configs/*.local.yaml
```

확인:

```bash
git status
```

`configs/device.local.yaml`이 Git 대상에 나타나지 않아야 합니다.

구조:

```text
공통 설정
configs/settings.yaml
→ Git 관리

장치별 설정
configs/device.local.yaml
→ Git 제외
```

## Local 설정을 Git에서 제외하는 이유

`device.local.yaml`은 장치마다 다른 값입니다.

따라서 Repository에 하나의 값으로 고정하지 않습니다.

```text
settings.yaml
→ 공통 기준
→ Git 관리

device.local.yaml
→ 내 장치 전용
→ Git 제외
```

오늘은 단순히 `device_name`만 넣지만 이후에는 환경마다 다른 설정이 추가될 수 있습니다.

비밀번호나 비밀키 같은 민감한 값은 별도의 보안 정책에 따라 관리하며, 교육용 Repository에 그대로 기록하지 않습니다.


---

# 22. 공통 설정 + Local 설정 합치기

`src/config_loader.py`를 다음처럼 수정합니다.

```python
from pathlib import Path

import yaml


def read_yaml(path: Path) -> dict:
    if not path.exists():
        return {}

    with path.open("r", encoding="utf-8") as file:
        data = yaml.safe_load(file)

    return data or {}


def load_config(
    path: str,
    local_path: str = "configs/device.local.yaml",
) -> dict:
    base_config = read_yaml(Path(path))
    local_config = read_yaml(Path(local_path))

    if not base_config:
        raise FileNotFoundError(
            f"기본 설정파일을 찾을 수 없습니다: {path}"
        )

    base_config.update(local_config)
    return base_config
```

동작:

```text
settings.yaml
공통값
        ↓
device.local.yaml
장치별 값
        ↓
같은 Key가 있으면
Local 값 우선
```

확인:

```bash
python -m scripts.check_config
```

예:

```text
Device : rpi13-01
```

Pipeline 실행:

```bash
python -m scripts.run_pipeline
```

로그에서 장치 이름 확인:

```bash
tail -n 5 logs/day01_events.csv
```

## 이 코드는 왜 수정할까요?

1일차의 `load_config()`는 하나의 YAML 파일만 읽었습니다.

2일차부터는 다음 두 종류의 설정이 생겼습니다.

```text
settings.yaml
→ 공통 설정

device.local.yaml
→ 현재 장치 전용 설정
```

따라서 두 파일을 읽고 하나의 Dictionary로 합치는 기능이 필요합니다.

## 의사코드

```text
기본 settings.yaml을 읽는다
        ↓
파일이 없거나 비어 있으면 빈 Dictionary를 반환한다
        ↓
device.local.yaml을 읽는다
        ↓
Local 파일이 없으면 빈 Dictionary를 사용한다
        ↓
기본 설정이 아예 없으면 오류를 발생시킨다
        ↓
기본 설정 Dictionary에 Local 설정을 덮어쓴다
        ↓
같은 Key가 있으면 Local 값이 최종값이 된다
        ↓
합쳐진 설정 Dictionary를 반환한다
```

예를 들어:

```yaml
# settings.yaml
device_name: pc-day01
sample_count: 20
warning_threshold: 0.70
```

```yaml
# device.local.yaml
device_name: rpi13-01
```

최종 Python 설정은 다음처럼 이해할 수 있습니다.

```text
device_name       → rpi13-01
sample_count      → 20
warning_threshold → 0.70
```

`device_name`만 Local 값으로 바뀌고 나머지 공통값은 그대로 유지됩니다.

## 코드 리뷰 — 어떤 코드가 Local 값을 우선하게 만들까요?

다음 한 줄을 찾습니다.

```python
base_config.update(local_config)
```

이 코드가 같은 Key가 있을 때 Local 값을 최종값으로 만드는 부분입니다.

따라서 `check_config.py`와 `run_pipeline.py`는 별도로 두 YAML을 읽을 필요가 없습니다.

```text
check_config.py / run_pipeline.py
        ↓
load_config()
        ↓
settings.yaml + device.local.yaml
        ↓
합쳐진 config 사용
```

이것이 설정 로더를 따로 만든 장점입니다.


---

# 23. SSH와 SCP의 역할 비교하기

SSH는 원격 장치에서 명령을 실행합니다.

```text
PC
→ SSH
→ Raspberry Pi Terminal
```

SCP는 SSH 연결을 이용해 파일을 전송합니다.

테스트를 위해 Raspberry Pi SSH 세션을 종료합니다.

```bash
exit
```

PC에서 파일 생성:

```bash
echo "hello from pc" > /tmp/pc_to_pi.txt
```

Raspberry Pi로 전송:

```bash
scp /tmp/pc_to_pi.txt <RPI_USER>@<RPI_IP>:/home/<RPI_USER>/
```

다시 접속:

```bash
ssh <RPI_USER>@<RPI_IP>
```

확인:

```bash
cat ~/pc_to_pi.txt
```

정상이라면:

```text
hello from pc
```

삭제:

```bash
rm ~/pc_to_pi.txt
```

정리:

```text
Git
→ 프로젝트 버전 관리

SSH
→ 원격 명령 실행

SCP
→ 파일 직접 전송
```

## 실행 전에 먼저 예상하기

다음 명령을 실행한다고 생각합니다.

```bash
scp /tmp/pc_to_pi.txt <RPI_USER>@<RPI_IP>:/home/<RPI_USER>/
```

실행 전에 답해 봅니다.

```text
파일은 어디에서 출발하는가?
→ PC

어디로 가는가?
→ Raspberry Pi의 /home/<RPI_USER>/

Raspberry Pi에서 어떤 명령으로 내용을 확인할 수 있는가?
→ cat ~/pc_to_pi.txt
```

즉, SSH와 SCP는 목적이 다릅니다.

```text
SSH
→ 원격 장치에서 명령 실행

SCP
→ 원격 장치와 파일 복사
```

3일차 이후 Camera 설정파일이나 작은 테스트 파일을 옮길 때도 같은 개념을 다시 사용할 수 있습니다.


---

# 24. Modify Lab — 장치 정보 하나 더 추가하기

지금까지 `check_device.py`에서는 다음 정보를 확인했습니다.

```text
Hostname
IP
OS
Architecture
Python
Current Path
Config Exists
```

이번에는 안내된 코드를 그대로 복사하지 않고 **정보 하나를 직접 추가**해 봅니다.

다음 중 하나를 선택합니다.

```text
CPU 개수
현재 시간
전체 Memory 용량
Disk 사용량
```

초보 단계에서는 `CPU 개수`를 권장합니다.

## 먼저 어느 파일을 수정해야 하는지 생각하기

바로 코드를 작성하지 말고 역할부터 찾습니다.

```text
장치 정보를 실제로 수집하는 파일
→ src/device_info.py

수집한 정보를 화면에 출력하는 파일
→ scripts/check_device.py
```

따라서 수정 흐름은 다음과 같습니다.

```text
device_info.py
정보를 가져오는 함수 추가
        ↓
check_device.py
그 함수를 import
        ↓
함수를 호출하여 출력
        ↓
실행
        ↓
결과 확인
```

## 직접 해보기

`src/device_info.py`에서 CPU 개수를 반환하는 함수를 만들어 봅니다.

힌트:

```python
import os

os.cpu_count()
```

그 다음 `scripts/check_device.py`에서 그 함수를 불러와 출력합니다.

실행:

```bash
python -m scripts.check_device
```

결과에 다음과 같은 줄이 추가되는지 확인합니다.

```text
CPU Count     : ...
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

`src/device_info.py` 위쪽에 다음 import를 추가할 수 있습니다.

```python
import os
```

함수:

```python
def get_cpu_count() -> int:
    return os.cpu_count() or 0
```

`check_device.py`의 import 목록에 추가합니다.

```python
get_cpu_count,
```

그리고 `main()` 안에서 출력합니다.

```python
print(f"CPU Count     : {get_cpu_count()}")
```

실제 숫자는 Raspberry Pi 모델과 환경에 따라 달라질 수 있습니다.

중요한 것은 숫자를 외우는 것이 아니라 다음 연결입니다.

```text
device_info.py
→ 정보 수집

check_device.py
→ 함수 호출

터미널
→ 결과 확인
```

</details>

---

# 25. Mini Challenge — Raspberry Pi 실행환경을 스스로 다시 확인하기

지금까지는 안내된 순서대로 Raspberry Pi를 준비했습니다.

이제부터는 **실행 결과를 보고 원인을 찾고, 실행 전에 결과를 예상하고, 필요한 부분만 직접 수정하는 연습**을 합니다.

새로운 어려운 기술을 배우는 시간이 아닙니다.

오늘 배운 내용을 다시 연결하는 시간입니다.

```text
장치 확인
   ↓
Network 확인
   ↓
SSH
   ↓
Path
   ↓
Python / .venv
   ↓
Config
   ↓
1일차 Pipeline
   ↓
Log
```

각 Challenge에서는 먼저 직접 생각하고 실행합니다.

그 다음 **예시 정답 확인해보기**를 열어 자신의 생각과 비교합니다.

---

## Challenge 1 — 결과를 보고 어느 코드에서 만들었는지 찾기

다음 실행 결과가 나왔다고 가정합니다.

```text
=== Raspberry Pi Device Check ===
Hostname      : rpi13-01
IP            : 192.168.x.x
Architecture  : aarch64
Python        : 3.x.x
Current Path  : /home/edgepi/ai_vision/subject13_edge_ai
Config Exists : True
```

코드를 열어 다음 표를 채웁니다.

| 출력 결과 | 어느 파일의 어떤 함수가 값을 얻는가? |
|---|---|
| Hostname | |
| IP | |
| Architecture | |
| Python | |
| Current Path | |
| Config Exists | |

또 다음 질문에 답합니다.

```text
화면에 print()하는 파일은 무엇인가?
장치 정보를 실제로 수집하는 파일은 무엇인가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

| 출력 결과 | 파일 / 함수 |
|---|---|
| Hostname | `src/device_info.py` → `get_hostname()` |
| IP | `src/device_info.py` → `get_ip_addresses()` |
| Architecture | `src/device_info.py` → `get_architecture()` |
| Python | `src/device_info.py` → `get_python_version()` |
| Current Path | `src/device_info.py` → `get_current_path()` |
| Config Exists | `src/device_info.py` → `config_exists()` |

```text
화면 출력
→ scripts/check_device.py

정보 수집
→ src/device_info.py
```

한 파일이 모든 일을 하는 것이 아니라 **수집 기능과 실행/출력 기능을 분리**했습니다.

</details>

---

## Challenge 2 — 명령을 실행하기 전에 어느 장치인지 예상하기

다음 프롬프트를 봅니다.

```text
A.
learner@DESKTOP:~/ai_vision/subject13_edge_ai$

B.
edgepi@rpi13-01:~/ai_vision/subject13_edge_ai$
```

다음 질문에 먼저 답합니다.

1. A에서 `hostname`을 실행하면 PC 이름이 나올까요, Pi 이름이 나올까요?
2. B에서 `uname -m`을 실행하면 어느 장치의 Architecture가 나올까요?
3. B에서 `python -m scripts.run_pipeline`을 실행하면 코드는 어디에서 실행될까요?
4. 확실하지 않을 때 어떤 세 명령으로 다시 확인할 수 있을까요?

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
A
→ PC/WSL2 Terminal

B
→ Raspberry Pi SSH Terminal
```

따라서:

```text
A의 hostname
→ PC 장치 이름

B의 uname -m
→ Raspberry Pi Architecture

B의 run_pipeline.py
→ Raspberry Pi에서 실행
```

확실하지 않으면:

```bash
whoami
hostname
pwd
```

를 실행합니다.

</details>

---

## Challenge 3 — Python 코드는 그대로 두고 장치별 설정만 바꾸기

현재 공통 설정:

```yaml
# configs/settings.yaml
device_name: pc-day01
sample_count: 20
interval_ms: 100
warning_threshold: 0.70
log_path: logs/day01_events.csv
```

Raspberry Pi Local 설정:

```yaml
# configs/device.local.yaml
device_name: rpi13-01
```

실행하기 전에 예상합니다.

```text
최종 device_name은 무엇인가?
sample_count는 몇 개인가?
warning_threshold는 얼마인가?
log_path는 어디인가?
```

그 다음 실행합니다.

```bash
python -m scripts.check_config
python -m scripts.run_pipeline
```

Log도 확인합니다.

```bash
tail -n 5 logs/day01_events.csv
```

질문:

```text
왜 device_name만 Raspberry Pi 값으로 바뀌었는가?
어느 코드가 두 설정을 합쳤는가?
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

최종 설정은 다음처럼 됩니다.

```text
device_name       → rpi13-01
sample_count      → 20
interval_ms       → 100
warning_threshold → 0.70
log_path          → logs/day01_events.csv
```

`device.local.yaml`에는 `device_name`만 있으므로 그 Key만 Local 값으로 덮어씁니다.

핵심 코드:

```python
base_config.update(local_config)
```

따라서 Python 실행 코드를 수정하지 않고도 장치별 이름을 다르게 사용할 수 있습니다.

</details>

---

## Challenge 4 — Raspberry Pi가 READY인지 프로그램으로 판정하기

이번에는 장치 정보 프로그램을 조금 확장합니다.

다음 두 조건을 모두 만족하면 `READY`, 아니면 `NOT READY`가 나오도록 만들어 봅니다.

```text
조건 1
configs/settings.yaml이 존재한다.

조건 2
현재 프로젝트의 .venv 안에서 실행 중이다.
```

가상환경 여부는 Python에서 다음 두 값을 비교할 수 있습니다.

```python
import sys

sys.prefix
sys.base_prefix
```

일반적인 venv에서는 두 값이 다릅니다.

먼저 직접 다음 함수를 `src/device_info.py`에 만들어 봅니다.

```text
is_venv_active()
→ 현재 가상환경인지 True / False 반환
```

그 다음 `scripts/check_device.py`에서 다음처럼 보이게 합니다.

```text
Config Exists : True
Venv Active   : True

[Result]
READY
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

`src/device_info.py`

```python
import sys


def is_venv_active() -> bool:
    return sys.prefix != sys.base_prefix
```

`check_device.py` import 목록에 추가합니다.

```python
is_venv_active,
```

`main()` 안에서 값을 먼저 구합니다.

```python
config_ok = config_exists()
venv_ok = is_venv_active()
ready = config_ok and venv_ok
```

출력:

```python
print(f"Config Exists : {config_ok}")
print(f"Venv Active   : {venv_ok}")

print()
print("[Result]")
print("READY" if ready else "NOT READY")
```

의사코드로 다시 보면:

```text
설정파일이 있는지 확인한다
        ↓
가상환경인지 확인한다
        ↓
둘 다 True인가?
   ┌────┴────┐
  예         아니오
  ↓            ↓
READY      NOT READY
```

이 Challenge의 목적은 복잡한 진단 프로그램을 만드는 것이 아니라 **장치가 실습 가능한 상태인지 조건으로 판단하는 연습**입니다.

</details>

---

## Challenge 5 — 일부러 오류를 만들고 원인 영역 찾기

### 문제 A — 잘못된 프로젝트 경로

Raspberry Pi에서 일부러 다음처럼 입력합니다.

```bash
cd ~/ai_vision/subject13_edge_ai_typo
```

예상되는 오류를 먼저 적어 봅니다.

```text
예상 오류:
문제 영역:
확인할 명령:
```

실행 후 오류를 확인합니다.

복구할 때는:

```bash
pwd
ls ~/ai_vision
cd ~/ai_vision/subject13_edge_ai
```

### 문제 B — 가상환경 비활성화

가상환경을 종료합니다.

```bash
deactivate
```

현재 Python을 확인합니다.

```bash
which python3
```

`check_device.py`의 READY/NOT READY 결과도 확인합니다.

다시 복구합니다.

```bash
source .venv/bin/activate
python -m scripts.check_device
```

### 문제 C — Local 설정파일을 잠시 숨기기

```bash
mv configs/device.local.yaml configs/device.local.bak
```

실행:

```bash
python -m scripts.check_config
```

어떤 `device_name`이 사용되는지 확인합니다.

다시 복구합니다.

```bash
mv configs/device.local.bak configs/device.local.yaml
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

### 문제 A

```text
대표 오류
→ No such file or directory

문제 영역
→ Path

확인
→ pwd
→ ls ~/ai_vision
```

### 문제 B

가상환경을 빠져나오면 `is_venv_active()`가 일반적으로 `False`가 됩니다.

```text
Config Exists : True
Venv Active   : False

[Result]
NOT READY
```

문제 영역:

```text
Python / venv
```

### 문제 C

`device.local.yaml`이 없어도 `settings.yaml`이 존재하므로 프로그램 자체는 실행될 수 있습니다.

이 경우 공통 설정의 `device_name`이 사용됩니다.

```text
프로그램 실행 성공
≠
장치별 설정까지 올바름
```

따라서 **실행되었다는 사실과 설정이 맞다는 사실을 구분**해야 합니다.

</details>

---

## Challenge 6 — PC에서 보낸 파일이 정말 Raspberry Pi에 도착했는지 증명하기

PC에서 SSH 세션을 종료합니다.

```bash
exit
```

PC에서 테스트 파일을 만듭니다.

```bash
echo "day02 scp challenge" > /tmp/day02_scp_check.txt
```

Raspberry Pi로 보냅니다.

```bash
scp /tmp/day02_scp_check.txt <RPI_USER>@<RPI_IP>:/home/<RPI_USER>/
```

다시 접속합니다.

```bash
ssh <RPI_USER>@<RPI_IP>
```

실행하기 전에 예상합니다.

```text
파일 위치:
파일 내용:
확인 명령:
```

그 다음 확인합니다.

```bash
cat ~/day02_scp_check.txt
```

정상이라면 다음 내용이 보입니다.

```text
day02 scp challenge
```

마지막으로 삭제합니다.

```bash
rm ~/day02_scp_check.txt
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
파일 출발
→ PC의 /tmp/day02_scp_check.txt

전송
→ SCP

파일 도착
→ Raspberry Pi의 /home/<RPI_USER>/

증명
→ Raspberry Pi에서 cat으로 실제 내용 확인
```

즉, 단순히 `scp` 명령이 끝난 것을 보고 성공이라고 하는 것이 아니라 **도착한 파일을 실제 장치에서 다시 확인**했습니다.

</details>

---

## Challenge 7 — 오늘의 전체 연결을 자신의 말로 설명하기

아래 빈칸을 먼저 채웁니다.

```text
1일차의 Source Code는 __________에 있었습니다.

Raspberry Pi에 원격 접속할 때 __________를 사용했습니다.

프로젝트 Source를 전달할 때
내부 Remote가 있으면 __________를 사용할 수 있고,
없으면 __________로 전달할 수 있습니다.

PC의 .venv는 Raspberry Pi에 __________ 않습니다.

Raspberry Pi에서는 __________를 새로 만들고
requirements.txt로 Package를 __________ 합니다.

공통 설정은 __________에 두고,
장치 전용 설정은 __________에 둡니다.

마지막에는 1일차의 __________를
Raspberry Pi에서 다시 실행합니다.
```

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

```text
1일차의 Source Code는 PC에 있었습니다.

Raspberry Pi에 원격 접속할 때 SSH를 사용했습니다.

프로젝트 Source를 전달할 때
내부 Remote가 있으면 Git Clone을 사용할 수 있고,
없으면 SCP로 전달할 수 있습니다.

PC의 .venv는 Raspberry Pi에 그대로 복사하지 않습니다.

Raspberry Pi에서는 .venv를 새로 만들고
requirements.txt로 Package를 다시 설치합니다.

공통 설정은 settings.yaml에 두고,
장치 전용 설정은 device.local.yaml에 둡니다.

마지막에는 1일차의 run_pipeline.py를
Raspberry Pi에서 다시 실행합니다.
```

핵심 흐름:

```text
PC Source
  ↓
Network
  ↓
SSH / Source 전달
  ↓
Raspberry Pi
  ↓
Pi 전용 .venv
  ↓
공통 설정 + Local 설정
  ↓
1일차 Pipeline 실행
  ↓
Log 확인
```

</details>

---

# 26. Mini Challenge 마무리 — 3일차에 사용할 정상 상태로 복원하기

Challenge 중 일부러 환경을 바꾸었다면 다음 상태로 복원합니다.

## 1. Raspberry Pi 프로젝트 위치

```bash
cd ~/ai_vision/subject13_edge_ai
pwd
```

## 2. 가상환경 활성화

```bash
source .venv/bin/activate
which python
```

`which python` 결과가 프로젝트의 `.venv/bin/python`을 가리키는지 확인합니다.

## 3. Local 설정파일 복구

```bash
ls configs
```

다음 두 파일이 있어야 합니다.

```text
settings.yaml
device.local.yaml
```

`device.local.yaml`에는 자신의 실제 Raspberry Pi Hostname 또는 장치 식별 이름을 사용합니다.

```yaml
device_name: rpi13-01
```

## 4. 전체 확인

```bash
python -m scripts.check_config
python -m scripts.check_device
python -m scripts.run_pipeline
```

다음이 모두 정상이어야 합니다.

```text
설정 읽기
→ 정상

장치 정보
→ 정상

Venv Active
→ True   # Challenge 4를 적용한 경우

Pipeline
→ NORMAL / WARNING 출력

CSV Log
→ 생성/추가 기록
```

## 5. 연습용 임시 파일 정리

```bash
rm -f ~/day02_scp_check.txt
```

Challenge 때문에 잘못 만든 파일이나 `.bak` 파일이 남아 있지 않은지 확인합니다.

```bash
find configs -maxdepth 1 -type f -print
```

이제 3일차 Camera/GPIO 실습에서 이어서 사용할 Raspberry Pi 상태가 준비되었습니다.

---

# 27. 문제가 생겼을 때 먼저 어느 영역인지 분류하기

2일차부터는 오류가 발생했을 때 바로 Python 코드부터 고치지 않습니다.

먼저 **어느 단계까지 성공했는지** 확인합니다.

```text
Power
  ↓
Network
  ↓
SSH
  ↓
Identity
  ↓
Path
  ↓
Python / venv
  ↓
Package
  ↓
Config
  ↓
Code
```

| 증상 | 먼저 확인할 영역 | 첫 확인 |
|---|---|---|
| Raspberry Pi가 전혀 보이지 않음 | Power / Network | 전원, 부팅, IP |
| SSH `timeout` | Network | 실제 IP, 같은 Network |
| SSH `Permission denied` | 계정/인증 | 사용자 이름, 인증 방식 |
| `.local` 이름만 실패 | Hostname/mDNS | IP로 재시도 |
| `cd` 실패 | Path | `pwd`, `ls` |
| `ModuleNotFoundError` | Python/Package | `which python`, venv, requirements |
| 장치 이름이 PC 이름으로 나옴 | Config | `device.local.yaml` |
| `SyntaxError` | Code | 오류가 표시한 파일/줄 |

## 문제를 말하는 방법도 연습합니다

다음 문장보다:

```text
"안 돼요."
```

다음처럼 말하는 것이 좋습니다.

```text
"Raspberry Pi에는 SSH 접속이 됩니다.
프로젝트 폴더에도 들어왔습니다.
그런데 .venv에서 실행했을 때 yaml Package를 찾지 못합니다."
```

이 문장에는 이미 다음 정보가 들어 있습니다.

```text
Network
→ 성공

SSH
→ 성공

Path
→ 성공

문제 범위
→ Python / Package
```

문제를 작게 나누면 해결도 빨라집니다.

---

# 28. 2일차 작업을 Version으로 저장하기

오늘 수정한 핵심 파일은 다음과 같습니다.

```text
src/device_info.py
scripts/check_device.py
src/config_loader.py
.gitignore
```

Challenge에서 READY 판정 기능을 완성했다면 그 코드도 포함됩니다.

`configs/device.local.yaml`은 장치별 설정이므로 Git 대상에서 제외되어야 합니다.

## 방법 A — Raspberry Pi가 내부 Remote Repository를 Clone한 경우

Raspberry Pi에서 확인합니다.

```bash
git status
git diff
```

`device.local.yaml`이 Git 대상에 보이지 않는지 확인합니다.

Stage:

```bash
git add .gitignore src scripts
```

Commit:

```bash
git commit -m "feat: add raspberry pi device environment check"
```

확인:

```bash
git log --oneline -5
```

내부 Remote가 연결되어 있고 Push가 허용된 경우:

```bash
git push
```

---

## 방법 B — SCP로 Source만 전달한 경우

이 경우 Raspberry Pi 폴더에는 Git Repository가 없을 수 있습니다.

따라서 **Pi에서 `git status`가 안 된다고 실습 실패가 아닙니다.**

먼저 Raspberry Pi에서 수정한 공통 Source와 보고서를 PC로 다시 가져옵니다.

PC에서 필요한 파일을 하나씩 가져올 수 있습니다.

예:

```bash
scp <RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/src/device_info.py src/
scp <RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/src/config_loader.py src/
scp <RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/scripts/check_device.py scripts/
scp <RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/.gitignore ./
```

그 다음 PC의 Local Git에서:

```bash
git status
git diff
git add .gitignore src scripts
git commit -m "feat: add raspberry pi device environment check"
```

`device.local.yaml`은 가져오거나 Commit하지 않습니다.

> Version 관리 위치는 환경에 따라 달라질 수 있지만 **장치별 설정과 공통 Source를 분리한다는 원칙은 동일**합니다.

---

# 29. 환경 확인 결과 기록하기

오늘은 코드만 만든 것이 아니라 **어떤 Raspberry Pi 환경에서 실행했는지** 기록하는 것도 중요합니다.

다음 파일을 만듭니다.

```text
reports/day02_environment.md
```

다음 형식으로 작성합니다.

```md
# Day 02 Raspberry Pi Environment

## Initial Setup

Raspberry Pi 모델:

microSD OS 기록: 성공 / 실패

선택한 OS:

Hostname 설정:

Wi-Fi 연결: 성공 / 실패

SSH 활성화: 성공 / 실패

첫 부팅 중 발생한 문제:

해결:

> 비밀번호, Wi-Fi 비밀번호, 인증키 등 민감한 정보는 이 보고서에 기록하지 않습니다.

## Device

Hostname:

IP:

Model:

OS:

Architecture:

Python:

Project Path:

## SSH

PC에서 SSH 접속: 성공 / 실패

사용한 접속 방식: IP / Hostname

접속 중 발생한 문제:

해결:

## Python Environment

Virtual Environment:

Python Path:

requirements.txt 설치: 성공 / 실패

## Project Test

check_config.py: 성공 / 실패

run_pipeline.py: 성공 / 실패

check_device.py: 성공 / 실패

## Config

공통 설정파일:

Local 설정파일:

실행 시 사용된 device_name:

## File Transfer

사용한 방법: 내부 Remote / SCP

확인한 내용:

## Failure Test

### Case 1

증상:

분류: Network / SSH / Path / Python / Package / Config / Code

원인:

해결:

### Case 2

증상:

분류:

원인:

해결:

## Mini Challenge Review

### 1. 실행 결과 역추적

내가 찾은 함수와 파일:

### 2. 실행 위치 구분

PC와 Raspberry Pi를 구분한 방법:

### 3. Local 설정

공통 설정과 Local 설정이 합쳐진 결과:

### 4. READY 판정

내가 사용한 조건:

### 5. SCP 확인

전송한 파일과 확인 방법:

### 6. 오늘의 전체 흐름

PC에서 만든 프로그램을 Raspberry Pi에서 실행하기까지의 흐름을 내 말로 설명:

## PC와 Raspberry Pi에서 달랐던 점

## 다음 확인

Camera Frame을 Raspberry Pi에서 직접 읽을 수 있는가?
```

정답 문장을 복사하지 않습니다.

**자신이 실제로 사용한 IP, Hostname, 오류와 결과**를 기준으로 기록합니다.

> 환경 정보에는 수업에 필요한 기술 정보만 기록하고 비밀번호나 비밀키는 작성하지 않습니다.

---

# 30. README에 2일차 실행 방법 추가하기

1일차 README에는 PC에서 Pipeline을 실행하는 방법을 기록했습니다.

이제 Raspberry Pi 실행 방법도 추가합니다.

프로젝트 루트의 `README.md`에 다음 내용을 이어서 작성합니다.

````md
## Day 02 — Raspberry Pi / SSH

1일차에 만든 같은 프로젝트를 Raspberry Pi에서 실행합니다.

### Raspberry Pi

SSH 접속:

```bash
ssh <RPI_USER>@<RPI_IP>
```

프로젝트 이동:

```bash
cd ~/ai_vision/subject13_edge_ai
```

가상환경 활성화:

```bash
source .venv/bin/activate
```

장치 확인:

```bash
python -m scripts.check_device
```

Pipeline 실행:

```bash
python -m scripts.run_pipeline
```

### Device Local Config

Raspberry Pi마다 다음 Local 설정을 사용합니다.

```text
configs/device.local.yaml
```

이 파일은 Git에서 제외합니다.

### Next

Day 03에서는 Raspberry Pi에 USB Camera와 GPIO 출력 장치를 연결합니다.
````

README는 명령어를 모두 외우지 않아도 **다음에 다시 프로젝트를 실행할 수 있게 만드는 사용설명서**입니다.

---

# 31. 문서까지 Version으로 저장하기

`reports/day02_environment.md`와 `README.md` 작성이 끝났다면 Version에 남깁니다.

## 내부 Remote Repository 방식

Raspberry Pi 또는 프로젝트의 Git 작업 장치에서:

```bash
git status
git add README.md reports/day02_environment.md
git commit -m "docs: record day02 raspberry pi environment"
```

내부 Remote에 Push가 허용된 경우:

```bash
git push
```

PC에서도 같은 Remote를 사용하는 경우 최신 Commit을 가져옵니다.

```bash
git pull
git log --oneline -5
```

## SCP 방식

Raspberry Pi에서 보고서를 작성했다면 PC로 가져옵니다.

PC에서:

```bash
scp <RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/reports/day02_environment.md reports/
```

README도 Pi에서 수정했다면 가져옵니다.

```bash
scp <RPI_USER>@<RPI_IP>:~/ai_vision/subject13_edge_ai/README.md ./
```

그 다음 PC Local Git에서:

```bash
git status
git add README.md reports/day02_environment.md
git commit -m "docs: record day02 raspberry pi environment"
```

오늘의 Version 흐름을 확인합니다.

```text
Day 01
프로젝트 / Pipeline
        ↓
Day 02
Raspberry Pi 실행환경
        ↓
장치 정보 / Local Config
        ↓
환경 기록
```

---

# 32. PC와 Raspberry Pi의 공통 파일과 Local 파일 다시 구분하기

2일차가 끝날 때 가장 헷갈리기 쉬운 부분입니다.

다음 구조를 다시 확인합니다.

```text
                  [공통 Source]
               settings.yaml
               src/
               scripts/
               README
               requirements.txt
                     │
              ┌──────┴──────┐
              ↓             ↓
            [PC]       [Raspberry Pi]
              │             │
          각자 .venv     각자 .venv
              │             │
       device.local    device.local
       learner01-pc      rpi13-01
```

## 공통으로 관리하는 것

```text
Python Source
공통 settings.yaml
requirements.txt
README
reports
```

## 장치마다 따로 만드는 것

```text
.venv/
configs/device.local.yaml
runtime logs/
```

이 원칙은 뒤에서 Camera 번호, Port, 실제 장치 경로가 달라질 때도 도움이 됩니다.

---

# 33. 2일차 최종 프로젝트 구조

공통 프로젝트 구조는 다음과 같습니다.

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

각 장치에서만 존재하는 파일:

```text
.venv/
configs/device.local.yaml
logs/
```

파일의 역할을 다시 확인합니다.

| 파일 | 역할 |
|---|---|
| `configs/settings.yaml` | 모든 장치가 공유하는 공통 설정 |
| `configs/device.local.yaml` | 현재 장치에만 적용할 설정 |
| `src/config_loader.py` | 공통 + Local 설정 읽기 |
| `src/device_info.py` | Linux/Raspberry Pi 장치 정보 수집 |
| `scripts/check_device.py` | 장치 정보와 준비 상태 출력 |
| `scripts/run_pipeline.py` | 1일차 Pipeline 실행 |
| `reports/day02_environment.md` | 실제 Pi 환경과 문제 해결 기록 |
| `requirements.txt` | Pi에서 Python Package 환경 재현 |
| `.gitignore` | `.venv`, Local 설정, Log 등을 Version 대상에서 제외 |

---

# 34. 수업 종료 전 Raspberry Pi에서 최종 확인

Raspberry Pi에 접속합니다.

```bash
ssh <RPI_USER>@<RPI_IP>
```

프로젝트로 이동합니다.

```bash
cd ~/ai_vision/subject13_edge_ai
```

가상환경을 활성화합니다.

```bash
source .venv/bin/activate
```

## 1. 내가 어느 장치인지 확인

```bash
whoami
hostname
hostname -I
uname -m
pwd
which python
```

## 2. 설정 확인

```bash
python -m scripts.check_config
```

## 3. 장치 확인

```bash
python -m scripts.check_device
```

Challenge 4의 READY 판정을 추가했다면:

```text
[Result]
READY
```

인지 확인합니다.

## 4. 1일차 Pipeline 다시 실행

```bash
python -m scripts.run_pipeline
```

## 5. Log 확인

```bash
tail -n 5 logs/day01_events.csv
```

Log의 `device` 값이 현재 Raspberry Pi의 Local 설정을 반영하는지 확인합니다.

## 오늘의 최종 PASS 기준

다음 항목이 모두 맞으면 2일차 핵심 실습은 완료입니다.

```text
[ ] Raspberry Pi Imager로 microSD에 OS 기록 완료
[ ] 학생별 고유 Hostname 설정 완료
[ ] Wi-Fi 또는 LAN 연결 완료
[ ] SSH 활성화 및 첫 부팅 완료
[ ] PC에서 Raspberry Pi로 SSH 접속 가능
[ ] 올바른 Hostname/IP 확인
[ ] Raspberry Pi 전용 .venv 사용
[ ] requirements 설치 완료
[ ] check_config.py 정상
[ ] check_device.py 정상
[ ] run_pipeline.py 정상
[ ] CSV Log 생성
[ ] device.local.yaml 적용
```

---

# 35. 수업 종료 전 PC에서 최종 확인

Raspberry Pi SSH 세션에서 나옵니다.

```bash
exit
```

PC 프로젝트로 이동합니다.

```bash
cd ~/ai_vision/subject13_edge_ai
```

PC의 Python 환경이 필요하다면 다시 활성화합니다.

```bash
source .venv/bin/activate
```

Git 상태를 확인합니다.

```bash
git status
git log --oneline -5
```

내부 Remote 방식을 사용한 경우 필요한 최신 Commit을 가져옵니다.

```bash
git pull
```

SCP 방식을 사용한 경우 Pi에서 수정한 공통 Source와 보고서가 PC에 반영되었는지 확인합니다.

최종적으로 다음 관계를 설명할 수 있어야 합니다.

```text
PC와 Raspberry Pi
→ 같은 Source를 사용

PC와 Raspberry Pi
→ .venv는 각각 따로 생성

PC와 Raspberry Pi
→ device.local.yaml도 각각 따로 사용 가능

공통 코드
→ Git 또는 허용된 방식으로 관리
```

---

# 36. 2일차 최종 복습 — 처음 받은 Raspberry Pi가 실행 장치가 되기까지

오늘 배운 명령을 모두 외우기보다 **어떤 준비 단계가 어떤 역할을 하는지** 연결해서 이해합니다.

## 복습 문제

먼저 답을 보지 않고 자신의 말로 답합니다.

1. microSD에 Raspberry Pi OS를 기록할 때 사용한 도구는 무엇인가요?
2. OS 기록 전에 저장장치를 다시 확인해야 하는 이유는 무엇인가요?
3. 학생마다 Hostname을 다르게 만드는 이유는 무엇인가요?
4. Wi-Fi 비밀번호나 Raspberry Pi 비밀번호를 README와 Git에 기록하면 안 되는 이유는 무엇인가요?
5. Raspberry Pi에 전원을 연결한 직후 SSH가 안 된다고 바로 OS를 다시 설치하면 안 되는 이유는 무엇인가요?
6. 첫 부팅에서 `OS 부팅 → Network → IP → SSH`는 어떤 순서로 준비되나요?
7. 1일차와 2일차의 가장 큰 차이는 무엇인가요?
8. SSH는 무엇을 하기 위해 사용했나요?
9. SCP는 SSH와 어떤 차이가 있나요?
10. Hostname과 IP는 어떤 차이가 있나요?
11. `.local` Hostname 접속이 안 되면 무엇으로 다시 시도할 수 있나요?
12. PC의 `.venv`를 Raspberry Pi로 그대로 복사하지 않는 이유는 무엇인가요?
13. Raspberry Pi에서는 어떤 파일을 이용해 Python Package를 다시 설치했나요?
14. `device_info.py`와 `check_device.py`의 역할은 어떻게 다른가요?
15. `settings.yaml`과 `device.local.yaml`을 왜 나누었나요?
16. Local 설정이 공통 설정보다 우선되게 하는 핵심 코드는 무엇인가요?
17. `ModuleNotFoundError`가 발생하면 코드보다 먼저 무엇을 확인할 수 있나요?
18. `cd`에서 `No such file or directory`가 나오면 어느 문제 영역인가요?
19. SSH는 되는데 `device_name`이 PC 이름으로 보인다면 어느 설정을 먼저 확인하나요?
20. 3일차에는 오늘 준비한 Raspberry Pi에 무엇을 연결하나요?

<details>
<summary><strong>예시 정답 확인해보기</strong></summary>

1. Raspberry Pi Imager를 사용했습니다.
2. 선택한 저장장치의 기존 내용이 지워질 수 있기 때문에 실습용 microSD가 맞는지 확인해야 합니다.
3. 같은 Network에서 여러 Raspberry Pi를 장치 이름으로 구분하기 위해서입니다.
4. 비밀번호는 민감한 정보이며 Repository와 공유 문서에 남기면 안 되기 때문입니다.
5. 첫 부팅에는 OS 부팅, 설정 적용, Network 연결, IP 할당, SSH 서비스 시작 시간이 필요하기 때문입니다.
6. 운영체제가 먼저 부팅되고, Network 설정이 적용된 뒤 IP가 할당되고 SSH 서비스가 사용할 수 있는 상태가 됩니다.
7. 1일차에는 PC에서 Pipeline 구조를 만들었고, 2일차에는 Raspberry Pi 자체를 세팅한 뒤 같은 프로젝트를 Pi에서 실행했습니다.
8. PC에서 Raspberry Pi의 Linux 터미널에 원격 접속하고 명령을 실행하기 위해 사용했습니다.
9. SSH는 원격 명령 실행, SCP는 SSH 연결을 이용한 파일 복사입니다.
10. Hostname은 장치 이름이고 IP는 Network에서 장치를 찾아가기 위한 주소입니다.
11. 실제 IPv4 주소를 확인하여 `ssh <RPI_USER>@<RPI_IP>` 방식으로 접속할 수 있습니다.
12. PC와 Pi는 Architecture와 실행환경이 다를 수 있고 `.venv`에는 장치 환경에 영향을 받는 Package와 경로가 포함될 수 있기 때문입니다.
13. `requirements.txt`를 사용했습니다.
14. `device_info.py`는 정보를 수집하고 `check_device.py`는 그 함수를 호출해 화면에 보여줍니다.
15. 모든 장치가 공유할 설정과 장치마다 달라지는 Local 설정을 분리하기 위해서입니다.
16. `base_config.update(local_config)`입니다.
17. `which python`, 가상환경 활성화 여부, Package 설치 여부를 먼저 확인할 수 있습니다.
18. Path 문제입니다.
19. `configs/device.local.yaml`과 설정 병합 결과를 먼저 확인합니다.
20. USB Camera 입력과 GPIO 출력 장치를 연결합니다.

</details>

---

# 37. 자가 체크리스트

수업을 마치기 전에 직접 체크합니다.

- [ ] 1일차 프로젝트를 새로 만들지 않고 그대로 이어서 사용했다.
- [ ] Raspberry Pi Imager의 역할을 설명할 수 있다.
- [ ] 실습용 microSD를 정확히 선택해 Raspberry Pi OS를 기록했다.
- [ ] 학생별로 고유한 Hostname을 설정했다.
- [ ] Wi-Fi/Network 설정과 SSH 활성화를 직접 확인했다.
- [ ] 비밀번호와 Wi-Fi 비밀번호를 README, Git, 보고서에 기록하지 않았다.
- [ ] microSD를 장착하고 Raspberry Pi의 첫 부팅 흐름을 설명할 수 있다.
- [ ] PC와 Raspberry Pi 터미널을 구분할 수 있다.
- [ ] `whoami`, `hostname`, `pwd`의 목적을 설명할 수 있다.
- [ ] Raspberry Pi의 실제 IPv4를 확인할 수 있다.
- [ ] PC에서 Raspberry Pi로 SSH 접속할 수 있다.
- [ ] SSH 오류를 Network/인증/Hostname 문제로 1차 분류할 수 있다.
- [ ] PC와 Pi의 Architecture가 다를 수 있음을 확인했다.
- [ ] PC의 `.venv`를 Pi로 복사하지 않는 이유를 설명할 수 있다.
- [ ] Raspberry Pi에서 `.venv`를 새로 만들었다.
- [ ] `requirements.txt`로 필요한 Package를 설치했다.
- [ ] Raspberry Pi에서 1일차 `run_pipeline.py`를 실행했다.
- [ ] `device_info.py`와 `check_device.py`의 역할을 구분할 수 있다.
- [ ] `settings.yaml`과 `device.local.yaml`의 차이를 설명할 수 있다.
- [ ] Local 설정이 Git 대상에서 제외되는지 확인했다.
- [ ] SCP로 파일 전송 후 실제 도착 내용을 확인할 수 있다.
- [ ] 일부러 만든 Path/venv/Config 오류를 복구했다.
- [ ] `reports/day02_environment.md`에 실제 환경 결과를 기록했다.
- [ ] README에 Raspberry Pi 실행 방법을 기록했다.
- [ ] 3일차 시작 전에 Raspberry Pi가 정상 실행 상태인지 확인했다.

체크되지 않는 항목이 있다면 전체를 처음부터 다시 하지 않습니다.

해당 항목과 연결된 단계만 다시 확인합니다.

---

# 38. 2일차 핵심을 한 문장으로 설명하기

다음 문장을 그대로 외우지 말고 자신의 말로 다시 설명해 봅니다.

> **2일차에는 Raspberry Pi OS를 microSD에 직접 설치하고 Wi-Fi/SSH를 준비한 뒤, PC에서 만든 1일차 프로젝트를 Raspberry Pi로 가져와 Pi 전용 Python 환경에서 같은 Edge Pipeline을 실행했습니다.**

조금 더 구조적으로 표현하면:

```text
Raspberry Pi Imager
   ↓
microSD + Raspberry Pi OS
   ↓
Hostname / 사용자 / Wi-Fi / SSH 설정
   ↓
Raspberry Pi 첫 부팅
   ↓
Network / IP
   ↓
PC에서 SSH 접속
   ↓
PC Source 전달
   ↓
Raspberry Pi Linux
   ↓
Pi 전용 .venv
   ↓
requirements.txt
   ↓
공통 설정 + Local 설정
   ↓
1일차 Pipeline 실행
   ↓
장치 상태 / Log 확인
```

오늘의 목적은 Linux 명령어를 많이 외우는 것이 아닙니다.

> **“PC에서 만든 프로그램이 실제 Edge 장치에서 실행되기까지 어떤 준비 단계가 필요한가?”**

를 이해하는 것이 핵심입니다.

---

# 39. 3일차와 연결하기

오늘까지는 Raspberry Pi에서 입력으로 여전히 **가상의 숫자**를 사용했습니다.

```text
2일차

PC
   ↓ SSH
Raspberry Pi
   ↓
Python
   ↓
가상 입력
   ↓
NORMAL / WARNING
   ↓
CSV Log
```

3일차부터 실제 입력과 실제 출력을 연결합니다.

```text
3일차 입력

USB Camera
    ↓
Raspberry Pi
    ↓
OpenCV
    ↓
Frame 확인
```

그리고 출력 쪽에서는:

```text
Raspberry Pi
    ↓
GPIO
    ↓
LED / Buzzer
```

즉, 1일차와 2일차에서 만든 구조는 버려지는 것이 아닙니다.

```text
1일차
Pipeline의 뼈대 이해
        ↓
2일차
Raspberry Pi OS 설치
→ Network / SSH
→ 실행환경 준비
→ 1일차 Pipeline을 Pi에서 실행
        ↓
3일차
실제 Camera / GPIO 연결
```

3일차를 시작하기 전에 다음 질문에 답할 수 있으면 됩니다.

> **“처음 받은 Raspberry Pi에 OS를 설치하고, Network와 SSH를 준비한 뒤, PC에서 만든 Python 프로젝트를 Pi에서 실행하려면 어떤 순서로 확인해야 하는가?”**

이 질문에 자신의 말로 답할 수 있다면 다음 수업으로 넘어갈 준비가 된 것입니다.
