# 🎵 VGA & Camera-Based Rhythm Game Accelerator (RHYTHM BEAT)

> **대한상공회의소 VGA 미니 프로젝트**  
> FPGA(Verilog HDL), OV7670 카메라 모듈, UART 인터페이스 및 Python UI를 연동한 객체 인식 기반 모션 리듬 게임 시스템 설계 및 검증

---

## 📌 1. Project Overview

본 프로젝트는 FPGA 기반의 **실시간 영상 처리(OV7670 Camera)**, **VGA Display Controller**, **Main FSM Controller**, **Game Logic Engine**, 그리고 **PC Interface(UART)**를 결합한 통합 하드웨어 가속/제어 시스템입니다.

* **주요 목표**: OV7670 카메라의 특정 색상(RED) 영역 감지를 통해 사용자 입력 신호를 생성하고, 이를 기반으로 VGA 화면상에 실시간 노트 Drop & 판정 시스템을 구동.
* **주요 특징**:
  * **Real-time Color Detection**: RGB565 임계값 조절을 통한 실시간 레인(4-Lane) 영역 선택 및 객체 인식
  * **System Control FSM**: IDLE, SELECT, READY, GAME, DONE의 상태 기반 메인 제어
  * **Low-Latency Event Trigger Sync**: PC-FPGA 간 클럭 불일치 문제를 해결하기 위한 1-Byte Event-Driven UART Protocol 설계
  * **Full Verification Closure**: SystemVerilog UVM 구축을 통한 100% Coverage 검증

---

## 🛠️ 2. System Architecture

```text
+-----------------------------------------------------------------------------------+
|                                FPGA Top Module                                    |
|                                                                                   |
|  +------------------+    +-------------------+    +----------------------------+  |
|  |  OV7670 Camera   |--->|  Region Detector  |--->|       Main Controller      |  |
|  | & SCCB Control   |    | (RED Pixel Count) |    |       (System FSM)         |  |
|  +------------------+    +-------------------+    +--------------+-------------+  |
|                                                                  |                |
|  +------------------+    +-------------------+                   v                |
|  |  UART Receiver   |--->|   Line Counter    |-------> +------------------+     |
|  |   (PC -> FPGA)   |    |  (Note Manager)   |         |    GameResult    |     |
|  +------------------+    +-------------------+         |  (Score/Verdict) |     |
|                                                        +---------+--------+     |
|  +------------------+    +-------------------+                   |              |
|  |   UART Sender    |<---|  FIFO Buffer Sync |<------------------+              |
|  |   (FPGA -> PC)   |    +-------------------+                   |              |
|  +------------------+                                            v              |
|                                                   +------------------------+    |
|                                                   | VGA Display Controller |    |
|                                                   +-----------+------------+    |
+---------------------------------------------------------------+-------------------+
                                                                |
                                                                v
                                                         [ VGA Monitor ]
🛠️ 3. Tech Stack & ToolsCategoryTechnologies & ToolsHDL / LanguageVerilog HDL, SystemVerilog (UVM), PythonFPGA TargetXilinx Artix-7 / Spartan Series (Vivado Design Suite)PeripheralsOV7670 Camera Module, VGA Monitor, UART (USB-to-Serial)VerificationVivado Simulator / Questasim, UVM Methodology👥 4. Team Members & Task AllocationNameRole / Sub-moduleKey Responsibilities조준호MainController게임 메인 상태 머신(FSM) 설계, 모듈간 동기화 제어  윤수민SCCB & VGA ControllerOV7670 레지스터 설정, VGA Timing Generator 및 픽셀 출력  김수빈GameResult & UVM판정 알고리즘(Perfect/Good/Miss), Fever 모드, UVM 검증 환경 구축  문태성Line CounterVSYNC 프레임 단위 노트 Y축 위치 이동 관리 (최대 16개 노트)  김지홍UART Sender & FIFOFPGA $\to$ PC 패킷 전송, FIFO 기반 Latency 및 Pop 타이밍 제어  송주연UART Receiver & ROMPC $\to$ FPGA 노트 생성 제어 신호 수신 및 디코딩  서어진Python UIPySerial 통신, PC 측 음원/노트 동기화 및 대시보드 UI 구현  📑 5. Block Specification & Features5.1. Region Detector (카메라 객체 인식)RED Color Thresholding:$R > 4'b0100$, $G < 4'b0111$, $B < 4'b0111$$R > G + 2 \quad \text{and} \quad R > B + 2$  Region Trigger: 레인별 Y 영역 내 빨간색 픽셀 수 $\ge 50$ 개 감지 시 유효 입력으로 판정.  5.2. Main Controller FSMStates: IDLE $\to$ SELECT $\to$ READY $\to$ GAME $\to$ DONE  Control: 버튼 입력 및 UART 수신 명령에 따른 상태 전환 제어.  5.3. Game Logic & Score Engine (GameResult)Judgment Zone (Y-axis):Perfect: $405 \le Y \le 445$ (+100점)  Good: $385 \le Y < 405$ 또는 $445 < Y \le 465$ (+50점)  Miss: 그 외 영역 (Combo Reset)  Fever Mode: Combo $\ge 3$ 달성 시 발동 (점수 2배 가산: Perfect +200, Good +100).  📡 6. Interface & Communication ProtocolBaud Rate: 115,200 bps  6.1. TX Protocol (Python $\to$ FPGA : 1 Byte)Plaintext  [7:4] Reserved / Command        [3:0] Lane Data Trigger
+-------------------------------+-------------------------------+
|   0x0 (Normal) / 0xF (End)    |  Lane 3 | Lane 2 | Lane 1 | Lane 0 |
+-------------------------------+-------------------------------+
6.2. RX Protocol (FPGA $\to$ Python : 7 Bytes Frame)ByteData NameBit DescriptionByte 0Header0xFF (Start Indicator)  Byte 1System State[7]=0, [6:4]=MainState[2:0], [3:0]=BtnInput[3:0]  Byte 2Verdict[7:4]=0000, [3]=Fever, [2]=Perfect, [1]=Good, [0]=Miss  Byte 3ComboCombo Count [7:0]  Byte 4Score LSBScore [7:0]  Byte 5Score MIDScore [15:8]  Byte 6Score MSBScore [23:16]  🧪 7. Verification & Coverage (UVM)SystemVerilog UVM(Universal Verification Methodology)을 적극 도입하여 시스템 안정성을 검증했습니다.  Plaintext+-----------------------------------------------------------------------+
|                              UVM Environment                          |
|                                                                       |
|  +--------------------+    +------------------+    +---------------+  |
|  |   UVM Generator    |--->|    UVM Driver    |--->|   DUT Top     |  |
|  |  (Random Pattern)  |    +------------------+    |  (FPGA Core)  |  |
|  +--------------------+                            +-------+-------+  |
|                                                            |          |
|  +--------------------+    +------------------+            v          |
|  |   UVM Scoreboard   |<---|   UVM Monitor    |<-----------+          |
|  | (Expected vs Act)  |    +------------------+                       |
|  +--------------------+                                               |
+-----------------------------------------------------------------------+
Coverage Target: 레인별 노트 생성, 판정 영역, Fever 전환 등 다변화 조건 검증.  Result: cp_lane, cp_region, cp_verdict, cross_lane 항목에 대한 Functional Coverage 100% Closure 달성.  🚨 8. Troubleshooting & Optimization8.1. Timing Violation (Critical Path & Negative Slack 해결)[cite: 1]Problem: Score 및 Combo 누적 연산 시 복잡한 조합 논리 경로(Combinational Path)로 인한 Timing Slack 미달 ($WNS = -0.561\text{ ns}$)[cite: 1].Solution: 연산 구간 사이에 파이프라인 레지스터(combo_reg)를 추가 설계하여 연산 경로를 2-Stage로 분할[cite: 1].Result: Timing Violation 완전 해결 ($WNS = +4.100\text{ ns}$ 달성)[cite: 1].8.2. Sender FIFO POP Timing Latency Correction[cite: 1]Problem: UART 전송 FIFO POP 타이밍이 차회 데이터 유입 타이밍과 맞물려 1-Cycle 데이터 밀림 문제 발생[cite: 1].Solution: POP 신호 제어 조건(!empty && ready && !valid)을 재조정하여 Push 즉시 Pop 스케줄링 수행[cite: 1].8.3. PC-FPGA Frame Sync Latency Removal[cite: 1]  Problem: 양방향 실시간 Continuous Sync 방식에서 UART 통신 지연으로 인한 화면-음원 싱크 오차 누적[cite: 1].Solution: PC는 음원 재생 및 노트 타임스탬프 관리, FPGA는 게임 연산 및 Graphic Rendering에 집중하며, 노트 발생 시에만 1-Byte Event-Trigger 방식으로 전환하여 오차 제거
