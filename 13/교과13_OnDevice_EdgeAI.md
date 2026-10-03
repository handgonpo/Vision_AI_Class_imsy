# 0. 교과 13은 어떤 수업인가요?

지금까지의 AI 수업에서는 주로 PC에서 데이터를 준비하고 모델을 학습한 뒤, 이미지나 영상을 
입력하여 AI가 제대로 예측하는지 확인했습니다. 

하지만 실제 산업현장에서는 AI 모델을 개발용 PC 안에서만 실행하는 것으로 끝나지 않습니다. 

예를 들어 공장에 설치된 카메라가 제품을 촬영하고 AI가 불량을 판단해야 한다면, 다음과 같은 과정이 필요합니다. 

```
카메라
→ 현장 장치
→ AI 추론
→ 정상 / 불량 판단
→ 결과 기록
→ 화면 · 대시보드 · 경고장치 등에 전달
```

즉, 만들어진 AI 모델을 실제로 사용할 장치나 실행환경으로 옮겨 동작시키는 과정이 필요합니다. 이러한 과정을 일반적으로 배포(Deployment)라고 합니다.
배포는 반드시 Cloud Server에 올리는 것만을 의미하지 않습니다.
AI는 상황에 따라 다음과 같이 여러 곳에서 실행할 수 있습니다.

```
Cloud Server에서 실행

또는

현장의 PC에서 실행

또는

카메라 근처의 Edge 장치에서 실행
```

산업현장에서는 네트워크가 항상 안정적이라고 보장할 수 없고, 카메라 영상을 계속 외부 Server로 보내는 것이 비효율적일 수도 있습니다. 또한 빠른 판단이 필요한 경우에는 Camera 가까이에 있는 장치에서 바로 AI를 실행하는 방식이 유리할 수 있습니다.

이처럼 데이터가 발생하는 현장 가까이에서 AI를 실행하는 방식을 Edge AI 또는 OnDevice AI라고 합니다.

교과 13에서는 이러한 Edge AI의 동작 방식을 이해하기 위해 먼저 **Raspberry Pi**를 사용합니다.

Raspberry Pi는 손바닥 정도 크기의 작은 컴퓨터로, Linux를 실행할 수 있고 Camera, Sensor, LED, Buzzer 같은 실제 장치를 연결할 수 있습니다.

```
일반 PC
→ AI 개발 · 학습

Raspberry Pi
→ 실제 Camera · Sensor 연결
→ AI 모델 실행
→ 결과 판단
→ GPIO · Log 출력
```

따라서 이 수업의 목적은 Raspberry Pi 자체를 배우는 것만이 아닙니다.

PC에서 개발한 프로그램과 AI 모델을 작은 현장 장치로 옮기고,

```
입력 → 처리 → AI 추론 → 판단 → 출력 → 기록 → 성능 측정 → 자동운영
```

까지 연결하는 전체 과정을 직접 경험하는 것이 핵심입니다.

이 경험은 이후 교과 14에서 사용하는 **NVIDIA Jetson Orin Nano Super**와도 연결됩니다.

Jetson은 NVIDIA GPU를 이용하여 AI 모델을 더 빠르게 실행할 수 있도록 설계된 Edge AI 장치입니다. 교과 14에서는 실제 제조 Vision AI 프로젝트를 Jetson에서 실행하게 됩니다.

처음부터 Jetson의 GPU, CUDA, TensorRT와 같은 환경을 모두 다루기보다, 교과 13에서는 먼저 Raspberry Pi를 이용하여 Edge AI의 기본 구조를 익힙니다.

```
교과 13

PC에서 개발
→ Raspberry Pi에 배포
→ Camera · Sensor 입력
→ AI 추론
→ GPIO · Log
→ 성능 측정
→ 자동실행 · 장애 확인

        ↓ 경험 확장

교과 14

PC에서 모델 개발
→ Jetson에 배포
→ 산업용 Camera 입력
→ GPU AI 추론
→ 제조 검사
→ 결과 · 통계 · 운영
```

따라서 교과 13은 단순한 Raspberry Pi 실습이 아닙니다.

> PC에서 개발한 AI를 실제 현장 장치로 가져가 실행하고 운영하는 방법을 배우며, 
> 이후 Jetson 기반 제조 Edge AI 프로젝트를 수행하기 위한 배포와 운영의 기초를 익히는 수업입니다.

이 과정을 위해 13일 동안 하나의 `subject13_edge_ai` 프로젝트를 계속 확장합니다.

![[Pasted image 20260910122243.png]]

```text
Edge 구조 이해
→ Raspberry Pi
→ Camera · Sensor
→ GPIO
→ Rule
→ 작은 AI 모델
→ ONNX 배포
→ 성능 측정
→ 안정화
→ 자동운영
→ Test
→ Jetson 확장
→ 최종 검증
```

핵심은 다음 흐름을 실제 장치에서 끝까지 경험하는 것입니다.

> **입력 → 처리 → 판단 → 출력 → 기록 → 성능 측정 → 안정화 → 운영**

---

# 1. 100시간 · 13일 전체 흐름

교과 13은 총 100시간이며 **1~12일차는 각 8시간, 13일차는 4시간**으로 진행합니다.

| 일차 | 무엇을 배우나요? | 핵심 결과 |
|---:|---|---|
| 1일차 | Edge AI와 Cloud 방식 | 기본 Edge Pipeline 이해 |
| 2일차 | Raspberry Pi · Linux · SSH | 원격 Edge 개발환경 |
| 3일차 | USB Camera · OpenCV · GPIO | 실제 영상 입력과 LED·Buzzer 출력 |
| 4일차 | Sensor · Sampling · Timestamp · CSV | 시간에 따른 데이터 기록 |
| 5일차 | Rule 기반 Edge Warning | Camera → Rule → GPIO → Log |
| 6일차 | Camera Dataset · TinyCNN | PyTorch 분류 모델 |
| 7일차 | PyTorch → ONNX → Raspberry Pi | 실제 Edge AI 추론 |
| 8일차 | Latency · FPS | 실제 처리성능 측정 |
| 9일차 | Frame Skip · Voting · Debounce · Hold | 속도·출력 안정화 |
| 10일차 | Headless · systemd · FastAPI | 자동실행·상태조회·자동재시작 |
| 11일차 | Test · 실패 분석 · Trade-off | 기능·성능·실패 조건 검증 |
| 12일차 | Raspberry Pi → Jetson | GPU Edge 확장 이해 |
| 13일차 | Acceptance Test · 운영 문서화 | 최종 운영 검증과 Handoff |

---

# 2. 1~4일차 — Edge 장치와 실제 입출력을 익힙니다

## 1일차 — Edge AI가 왜 필요한가?

가상의 입력으로 작은 Edge Pipeline을 만들고 Edge 방식과 가상의 Cloud 지연을 비교합니다.  
13일 동안 사용할 프로젝트의 기본 구조도 준비합니다.

### 🔗 [[1일차 — Edge AI가 왜 필요한가]]

---

## 2일차 — Raspberry Pi · Linux · SSH

1일차의 프로젝트를 Raspberry Pi에서 실행합니다.  
SSH, Linux, Network, Architecture와 장치별 Python 환경을 확인합니다.

### 🔗 [[2일차 — RaspberryPi_Linux_SSH_상세실습]]

---

## 3일차 — Camera · OpenCV · GPIO

Raspberry Pi에 연결한 **USB Camera** 영상을 OpenCV로 읽고 GPIO로 LED·Buzzer를 제어합니다.  
실제 입력과 실제 출력을 처음 연결합니다.

### 🔗 [[3일차 — Camera_GPIO_상세실습]]

---

## 4일차 — Sensor · Sampling · Timestamp · CSV

Button과 숫자형 입력값을 읽고 Timestamp와 함께 CSV에 기록합니다.  
시간에 따라 들어오는 데이터를 수집하고 기록하는 방법을 배웁니다.

### 🔗 [[4일차 — Sensor·Sampling·Timestamp·CSV]]

---

# 3. 5~7일차 — Rule에서 AI로 확장합니다

## 5일차 — AI 없이 Rule 기반 Edge Warning System 만들기

Camera Frame의 빨간색 영역을 Rule로 분석하여 NORMAL / WARNING을 판단합니다.  
판단 결과를 GPIO와 Event Log까지 연결하여 첫 End-to-End 시스템을 완성합니다.

### 🔗 [[5일차 — AI 없이 Rule 기반 Edge Warning System 만들기]]

---

## 6일차 — Edge용 작은 AI 모델을 만들기

Camera에서 `normal / warning_red` 이미지를 직접 수집하여 Train·Validation·Test 데이터를 구성합니다.  
PC에서 작은 `TinyCNN`을 학습하여 7일차 배포에 사용할 PyTorch 모델을 만듭니다.

### 🔗 [[6일차 — Edge용 작은 AI 모델을 만들기]]

---

## 7일차 — PyTorch 모델을 ONNX로 바꾸고 Raspberry Pi에 배포하기

6일차 PyTorch 모델을 ONNX로 변환하고 결과를 검증합니다.  
Raspberry Pi에서 ONNX Runtime으로 Camera 실시간 AI 추론과 GPIO·Log를 연결합니다.

### 🔗 [[7일차 — PyTorch 모델을 ONNX로 바꾸고 Raspberry Pi에 배포하기]]

---

# 4. 8~11일차 — 실행되는 AI를 운영 가능한 시스템으로 바꿉니다

## 8일차 — Latency와 FPS를 제대로 측정하기

Raspberry Pi에서 Model-only와 End-to-End 처리성능을 나누어 측정합니다.  
Mean·P50·P95·FPS를 이용해 실제 Edge 성능을 확인합니다.

### 🔗 [[8일차 — Latency와 FPS를 제대로 측정하기]]

---

## 9일차 — 느리고 흔들리는 Edge AI를 개선하기

Frame Skip, Voting, Debounce, Hold 등을 적용합니다.  
속도·반응성·출력 안정성 사이의 Trade-off를 비교하여 운영 설정을 선택합니다.

### 🔗 [[9일차 — 느리고 흔들리는 Edge AI를 개선하기]]

---

## 10일차 — Headless · systemd · FastAPI · 자동실행 · 장애 복구

Raspberry Pi가 부팅되면 Edge Runtime이 자동으로 실행되도록 구성합니다.  
FastAPI로 외부 PC에서 상태를 확인하고 비정상 종료 시 자동 재시작되는 운영 구조를 만듭니다.

### 🔗 [[10일차 — Headless·systemd·FastAPI·자동실행·장애 복구]]

---

## 11일차 — Test · 실패 분석 · 성능 실험 · Trade-off

새로운 기능을 추가하기보다 지금까지 만든 시스템을 검증합니다.  
Unit → Integration → System Test를 진행하고 실패 조건과 최종 운영 설정을 분석합니다.

### 🔗 [[11일차 — Test·실패 분석·성능 실험·Trade-off]]

---

# 5. 12~13일차 — Jetson 확장과 최종 운영 검증

## 12일차 — Raspberry Pi에서 Jetson으로 확장하기

Raspberry Pi의 CPU + ONNX Runtime 구조와 Jetson의 GPU + TensorRT 구조를 비교합니다.  
어떤 코드는 재사용하고 어떤 부분은 장치에 맞게 바꾸는지 이해하며 교과 14로 연결합니다.

### 🔗 [[12일차 — Raspberry Pi에서 Jetson으로 확장하기]]

---

## 13일차 — 최종 통합 · Acceptance Test · 운영 문서화

새로운 기능을 만들지 않습니다.  
재부팅, 자동실행, Camera·AI·GPIO·Log·FastAPI, 장애 복구와 성능 상태를 확인하여 **다른 사람이 운영할 수 있는 상태인지 최종 검증**합니다.

### 🔗 [[13일차 — 최종 통합·Acceptance Test·운영 문서화]]

---

# 6. 교과 14와의 연결

교과 13에서는 Raspberry Pi를 통해 Edge AI의 기본 구조를 경험합니다.

교과 14에서는 이 경험을 **Industrial Camera + Jetson Orin Nano Super + ONNX/TensorRT 기반 제조 Vision AI**로 확장합니다.

```text
교과 13
Raspberry Pi에서 Edge AI 구조 이해
        ↓
교과 14
Jetson 기반 실제 제조 검사로 확장
```

장치와 모델이 달라져도 다음 사고방식은 그대로 이어집니다.

> **입력 → 전처리 → 추론 → 판단 → 기록 → 성능 → 운영**

