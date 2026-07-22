# VGA_RHYTHM_GAME

## 0. Summary

#### Overview
- VGA를 이용한 추억의 리듬 게임 시스템 구현(FPGA)
- OV7670(Cam)을 이용하여 빨간 영역(손)을 감지하여 노트 데이터와 일치 or 불일치 판정
- PYTHON ui를 통한 게임 구
<br><br>

#### 개발 환경 및 ARCHITECTURE
- Cam : OV7670
- Tool : Vivado, Vscode
- Language : Systemverilog, Python
- FPGA Board : Basys3
<br><br>

## 1. Instruction & Background
### 1.1 VGA



## 2. Hardware Architecture & System Design
### 2.1 Overall System Architecture


### 2.2 VGA


#### 2.2-1 SCCB Controller

#### 2.2-2 Region Detector


### 2.3 Main Controller

### 2.4 Python - UART TX

### 2.5 Receiver

### 2.6 Line Counter

### 2.6 GameResult

### 2.7 Score

### 2.8 Sender
<img width="899" height="446" alt="image" src="https://github.com/user-attachments/assets/8f674f68-b651-4287-9af4-edfd51c87eb8" />

<Sender Blockdiagram> 

- 게임 데이터 정보(MISS, GOOD, PERFECT, COMBO, FEVER, SCORE, MAIN STATE) 및 버튼 신호를 PC로 송신
- 위의 신호들을 총 7byte로 보냄(1패킷의 크기 = 7byte)
- FIFO를 통해 안정적인 신호 송신
- FIFO(1byte 전송) - 1/100MHz = 10ns, UART(1byte 전송) - 1/115200 x 8 = 69.44us
- FIFO의 Depth는 32byte로 7byte 패킷이 4개까지 들어갈 수 있음
  - 데이터가 유실되지 않는 3가지 이유
    - ① 사람의 손가락 속도 한계 - UART가 7바이트(1패킷)를 PC로 전부 다 전송하는 데 걸리는 시간은 약 0.6ms, 사람의 버튼 연타 간격: 66ms
      -> 사람이 두 번째 버튼을 누르기도 전에(66ms) UART는 이미 첫 번째 패킷(0.6ms)을 PC로 다 보내고 FIFO를 싹 비워진 상태   
    - ③ FIFO에 PUSH 하는 조건
        .push (fifo_push & !fifo_full)
    결론 : FIFO 크기보다 큰 데이터가 들어오기 전에 UART가 전송됨


<img width="1752" height="738" alt="image" src="https://github.com/user-attachments/assets/25ba40b8-a31d-42d3-a801-3e8cbb0ee16c" />

<Sender Dataflow>


<img width="1712" height="757" alt="image" src="https://github.com/user-attachments/assets/c3221775-2983-467a-a013-a997c850c66d" />

<Sender Control Unit FSM>


### 2.9 Python - UART RX
<img width="1759" height="764" alt="image" src="https://github.com/user-attachments/assets/bdee63a2-3f17-485a-9da3-5934cc1210d1" />



## 3. Result
### 3.1 UVM

#### 3.1-1 Blockdiagram
<img width="1719" height="767" alt="image" src="https://github.com/user-attachments/assets/cda11cc1-caf3-407f-bca7-f4bdb9ccf126" />


#### 3.1-2 Result
<img width="2147" height="898" alt="image" src="https://github.com/user-attachments/assets/a68709d2-8fc7-4cc6-b187-b6f7b298268b" />


### 3.2 Demo Video
https://github.com/user-attachments/assets/a1a9cf91-86d8-4608-8e64-c149ed528b66

## 4. TroubleShooting & 고찰
- TroubleShooting
  - Sender에서 FIFO의 POP 시점 수정
    
    <img width="438" height="222" alt="image" src="https://github.com/user-attachments/assets/95e2142d-0a28-4236-8e31-63569ee80e6d" />
    
    - 목표 : Main State, 버튼 값, 판정 결과, 점수 등 데이터의 변동이 있을 때마다, 바로 PC로 전송
    - 문제점 : 데이터 보내는 타이밍이 한 cycle이 밀리는 이슈 발생했었음
    - 그 문제가 발생한 원인 분석 : POP 시점이 FIFO로 데이터가 들어오자마자가 아닌, 다음 데이터가 들어올 때마다 POP 하도록 수정
    - 해결한 방법
        1. !empty && ready && !valid 일 때, pop = 1
        2. FIFO에 pop 신호가 있을 때, 다음 Uart의 valid 신호 = 1
        3. Uart의 ready 신호가 있을 때, 다음 valid 신호 = 0
  - 고찰
    - 팀원들과의 정확한 소통없이, 간단한 모듈을 구성해 완성하는 것조차 쉽지 않았음
    - UART, FIFO 타이밍 이슈를 해결하며 통신이 쉽지 않음을 느낌...
