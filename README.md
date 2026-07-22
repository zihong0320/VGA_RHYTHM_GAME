# 🎵 VGA & Camera-Based Rhythm Game System (VGA_RHYTHM_GAME)

> **대한상공회의소 VGA 미니 프로젝트**  
> FPGA(Verilog HDL), OV7670 카메라 모듈, UART 통신 및 Python UI를 연동한 객체 인식 기반 모션 리듬 게임 시스템

---

## 📌 0. Summary

### Overview
<p align="center">
  <img width="342" alt="Overview" src="https://github.com/user-attachments/assets/db06a132-4072-4ef2-b1da-e522fe1b809e" />
</p>

* **프로젝트명**: VGA_RHYTHM_GAME (RHYTHM BEAT)
* **핵심 기능**:
  * **VGA 디스플레이 출력**: FPGA 기반으로 VGA 화면에 노트 및 게임 UI를 실시간 출력
  * **카메라 모션 인식**: OV7670 카메라 모듈을 통해 특정 빨간색 영역(손/물체)을 감지하여 노트 판정 수행
  * **Python UI 연동**: UART 통신을 통한 PC(Python UI)와 FPGA 간 실시간 데이터 송수신

---

### 🛠 개발 환경 및 Architecture
* **Target Board**: Basys3 (Xilinx Artix-7)
* **Camera Module**: OV7670
* **Development Tools**: Vivado, VS Code
* **Languages**: SystemVerilog, Verilog HDL, Python

---

### 👥 Team Members & Task Allocation

| Name | Sub-module / Role | Key Responsibilities |
| :--- | :--- | :--- |
| **조준호** | **MainController** | 게임 메인 상태 머신(FSM) 설계, 모듈간 동기화 제어 |
| **윤수민** | **SCCB & VGA Controller** | OV7670 레지스터 설정, VGA Timing Generator 및 픽셀 출력 |
| **김수빈** | **GameResult & UVM** | 판정 알고리즘(Perfect/Good/Miss), Fever 모드, UVM 검증 환경 구축 |
| **문태성** | **Line Counter** | 프레임 단위 노트 Y축 위치 이동 관리 (최대 16개 노트) |
| **김지홍** | **UART Sender & FIFO & ROM** | FPGA $\to$ PC 패킷 전송, FIFO 기반 Latency 및 Pop 타이밍 제어, 음악 ROM 파일 생성 |
| **송주연** | **UART Receiver & ROM** | PC $\to$ FPGA 노트 생성 제어 신호 수신 및 디코딩, 음악 ROM 파일 생성 |
| **서어진** | **Python UI** | PySerial 통신, PC 측 음원/노트 동기화 및 대시보드 UI 구현 |

---

## 🛠️ 2. Hardware Architecture & System Design

### 2.1 Overall System Architecture

<p align="center">
  <img width="75%" alt="Overall Block Diagram" src="https://github.com/user-attachments/assets/99622e72-415e-454e-8ad9-e9cb1c533319" />
</p>

---

### 2.2 VGA & Camera Engine

<p align="center">
  <img width="75%" alt="VGAcam Block Diagram" src="https://github.com/user-attachments/assets/8efc4658-da4f-4744-9d25-fcf86b16ff35" />
</p>

#### 2.2-1 SCCB Controller
<p align="center">
  <img width="75%" alt="SCCB Controller Diagram" src="https://github.com/user-attachments/assets/82d84dad-945c-4f93-89f3-7358d3898752" />
</p>

OV7670 카메라 설정 및 제어를 위해 SCCB 프로토콜을 구현하여 레지스터를 초기화

#### 2.2-2 Region Detector (카메라 색상 인식)
<p align="center">
  <img width="70%" alt="Region Detector Diagram" src="https://github.com/user-attachments/assets/dfc156ef-7a4e-49d3-8d34-4e05fd424982" />
</p>

* **RED 픽셀 판정 기준**:
  * $R > 4'b0100 \quad \text{and} \quad G < 4'b0111 \quad \text{and} \quad B < 4'b0111$
  * $R > G + 4'b0010 \quad \text{and} \quad R > B + 4'b0010$
* **REGION 입력 판정 기준**: 지정 영역 내 RED 픽셀 카운트가 **50 Pixel 이상**일 때 유효 감지 판정

---

### 2.3 Main Controller

| Main Controller Block Diagram | Dataflow |
| :---: | :---: |
| <img src="https://github.com/user-attachments/assets/e1ce6cb1-ed76-4db5-8abe-11a43d938cb7" width="100%"/> | <img src="https://github.com/user-attachments/assets/ec58f5db-4001-45ef-a222-832c346d12b5" width="100%"/> |

<p align="center">
  <img width="50%" alt="Main Controller FSM" src="https://github.com/user-attachments/assets/ff023d26-c8e9-4cde-9b88-8c37a9f41a51" /><br>
  <b>[ Main Controller FSM ]</b>
</p>

* 게임 전반의 상태(`IDLE`, `SELECT`, `READY`, `GAME`, `DONE`)를 관리하는 State Machine

---

### 2.4 Python - UART TX Protocol

<p align="center">
  <img width="60%" alt="Python UART TX Concept" src="https://github.com/user-attachments/assets/49135d12-b900-4811-8b18-6248603492bd" />
</p>

* **동작 과정**:
  1. PC에서 ROM 데이터를 읽어 노트 생성 시점 계산
  2. 노트 출발 시점에 UART를 통해 **1 Byte** 데이터 전송
  3. 하위 4비트(`lane_data[3:0]`)에 생성된 노트의 레인 정보 매핑

<p align="center">
  <img width="50%" alt="1Byte TX Protocol" src="https://github.com/user-attachments/assets/96ae248c-9913-49be-90a7-e4a3efc4f71c" /><br>
  <b>[ 1 Byte TX Protocol (Baud Rate : 115,200) ]</b>
</p>

---

### 2.5 Line Counter

<p align="center">
  <img width="90%" alt="Line Counter Diagram" src="https://github.com/user-attachments/assets/80ca07d8-65a2-4648-9381-f9d5123393a3" />
</p>

* **Frame 단위 노트 관리**:
  * `note_start` 및 `lane_data[3:0]` 신호 입력 시, **1 Frame마다 3 Line씩 아래로 하강하는 Line Count 생성**
  * 한 화면당 **최대 16개의 Lane/노트** 실시간 관리

---

### 2.6 GameResult & Score Engine

| GameResult Diagram | Score Engine |
| :---: | :---: |
| <img src="https://github.com/user-attachments/assets/795a947d-91e8-4b21-a602-9d97dd7f309e" width="100%"/> | <img src="https://github.com/user-attachments/assets/f7c66f56-005f-4e7f-87f1-9e0cdffda3cb" width="100%"/> |

* Line Counter의 노트 위치와 카메라/버튼 입력 신호를 비교하여 Perfect, Good, Miss 판정을 수행하고 점수를 업데이트

---

### 2.7 Sender & UART TX (FPGA $\to$ PC)

<p align="center">
  <img width="80%" alt="Sender Block Diagram" src="https://github.com/user-attachments/assets/8f674f68-b651-4287-9af4-edfd51c87eb8" /><br>
  <b>[ Sender Block Diagram ]</b>
</p>

* **기능**: 게임 상태 정보(MISS, GOOD, PERFECT, COMBO, FEVER, SCORE, MAIN STATE) 및 버튼 신호를 총 **7 Byte 패킷**으로 PC에 송신.
* **FIFO 기반 안정화**:
  * FIFO Clock ($100\text{ MHz}$, $10\text{ ns}$) 대비 UART Transmit 속도 ($115,200\text{ bps}$, $1\text{ Byte} \approx 69.44\ \mu\text{s}$) 보완을 위해 **Depth 32 Byte FIFO Buffer** 적용.
  * 7 Byte 패킷을 최대 4개까지 임시 저장 가능하여 데이터 유실 방지.
  * UART전송 시간(약 $0.6\text{ ms}$) 대비 사람의 입력 연타 간격(약 $66\text{ ms}$)이 충분히 길어 Buffer Overflow 방지.

<p align="center">
  <img width="90%" alt="Sender Dataflow" src="https://github.com/user-attachments/assets/25ba40b8-a31d-42d3-a801-3e8cbb0ee16c" /><br>
  <b>[ Sender Dataflow ]</b>
</p>

* **Sender Control Unit FSM**:
  <p align="center">
    <img width="90%" alt="Sender FSM" src="https://github.com/user-attachments/assets/c3221775-2983-467a-a013-a997c850c66d" />
  </p>
  
  * `START` $\to$ `SCORE_THIRD`까지 총 7 Byte를 전송하는 FSM 구조:
    * **Byte 0**: Header Bit (`8'hFF`)
    * **Byte 1**: `{0, main_state[2:0], btn[3:0]}`
    * **Byte 2**: `{0000, Fever, Perfect, Good, Miss}`
    * **Byte 3**: `Combo[7:0]`
    * **Byte 4**: `Score[7:0]` (LSB)
    * **Byte 5**: `Score[15:8]` (MID)
    * **Byte 6**: `Score[23:16]` (MSB)

---

### 2.8 Python - UART RX

<p align="center">
  <img width="90%" alt="Python UART RX Interface" src="https://github.com/user-attachments/assets/bdee63a2-3f17-485a-9da3-5934cc1210d1" />
</p>

* FPGA에서 전송된 7 Byte 패킷을 실시간 파싱하여 Python UI 상의 스코어보드 및 대시보드 화면에 반영합니다.

---

## 🧪 3. Result & Verification

### 3.1 Verification Architecture & UVM Result

| UVM Environment Block Diagram | Verification Result |
| :---: | :---: |
| <img src="https://github.com/user-attachments/assets/cda11cc1-caf3-407f-bca7-f4bdb9ccf126" width="100%"/> | <img src="https://github.com/user-attachments/assets/a68709d2-8fc7-4cc6-b187-b6f7b298268b" width="100%"/> |

* SystemVerilog UVM 구축을 통해 신호 검증 및 기능 테스트 수행 완료.

---

### 3.2 Demo Video

[![Demo Video](https://img.shields.io/badge/Demo_Video-Watch_on_GitHub-blue?style=for-the-badge&logo=github)](https://github.com/user-attachments/assets/a1a9cf91-86d8-4608-8e64-c149ed528b66)

---

## 🚨 4. TroubleShooting & Review

### 🚨 TroubleShooting: Sender FIFO POP Timing Latency 이슈
* **목표**: Main State, 버튼 값, 판정 결과, 점수 데이터 변동 시 즉각 PC로 전송.
* **문제점**: 데이터를 보내는 타이밍이 1-Cycle 밀리는 Latency 이슈 발생했었음.
* **원인 분석**: POP 시점이 FIFO에 데이터가 들어오자마자가 아닌, 다음 데이터가 들어오는 시점에 POP 되도록 제어되어 오차 발생.
* **해결 방법**:
  1. `!empty && ready && !valid` 조건 만족 시 `pop = 1` 트리거
  2. FIFO에 `pop` 신호가 있을 때, 다음 UART의 `valid = 1` 설정
  3. UART의 `ready` 신호 확인 시 다음 `valid = 0` 처리하여 타이밍 밀림 해결

### 💡 프로젝트 고찰
* 팀원과의 명확한 포트, 방식의 정의에 대한 소통 없이 서브 모듈을 각각 구성하니 통합하는 과정에서 두 번의 코드 수정이 필요했음 따라서, 소통의 중요성을 체감함.
* 특히 UART 통신과 FIFO 제어 시 발생한 타이밍 이슈를 분석하고 디버깅하며 하드웨어 간 실시간 통신 시스템 설계의 중요성을 깨달음.
