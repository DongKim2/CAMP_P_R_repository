---
marp: true
theme: gaia
class: lead
paginate: true
backgroundColor: #fff
backgroundImage: url('https://marp.app/assets/hero-background.svg')
style: |
  h1 {
    color: #000;
  }
  h2 {
    color: #333;
  }
  strong {
    color: #0056b3;
  }
---

# High-Performance Trading System
## Prototype for Real-time Trading Programmer

**극한의 성능을 향한 도전**

---

## 1. 프로젝트 개요 (Overview)

**목표**: Renaissance Technologies 등의 'Real-time Trading Programmer' 포지션 요구사항 충족

*   **초저지연 (Ultra-Low Latency)**
*   **고신뢰성 (High-Reliability)**
*   **하드웨어/소프트웨어 최적화**

> "단순한 코딩을 넘어, 극한의 엔지니어링 역량을 증명한다."

---

## 2. 주요 목표 (Goals)

*   **Performance**
    *   마이크로초(μs) 단위 레이턴시
    *   초당 수백만 건 처리
*   **Technology**
    *   Modern C++ (C++17/20)
    *   Low-level Linux System Programming
*   **Engineering**
    *   Memory Optimization
    *   Race Condition 해결

---

## 3. 핵심 기능 (Features) - 1

### 📈 초고속 호가창 엔진 (Order Book)
*   **주문 관리**: 생성(Add), 수정(Modify), 취소(Cancel)
*   **매칭 엔진**: Price-Time Priority 알고리즘
*   **자료구조**: `std::map` 미사용 -> Custom Flat Map / Array

---

## 3-1. Deep Dive: Memory Optimization

**"메모리 할당은 가장 비싼 작업이다"**

*   **문제**: 프로그램 실행 중 `new`로 메모리를 할당하면 OS에게 공간을 요청하느라 시간이 지체됩니다. (가게에서 주문할 때마다 지갑을 가지러 집에 다녀오는 꼴)
*   **해결**: **Object Pool** 기법 사용.
    *   시작할 때 메모리를 왕창 빌려둡니다. (미리 가져온 지갑)
    *   필요할 때 쓰고, 다 쓰면 반납만 합니다.
    *   **결과**: 할당 시간 **Zero**.

---

## 3-2. Deep Dive: Race Condition

**"두 사람이 동시에 하나의 펜을 집으려 한다면?"**

*   **문제**: 여러 스레드가 동시에 같은 데이터를 건드리면 데이터가 깨지거나 순서가 꼬입니다. 이를 막고자 `Lock(잠금)`을 걸면 다른 스레드는 멈춰서 기다려야 합니다. (성능 저하)
*   **해결**: **Lock-free 알고리즘 (Atomic)**
    *   잠금 없이도 줄을 서서 펜을 집을 수 있게 만드는 마법 같은 기술.
    *   `std::atomic`을 사용하여 하드웨어 레벨에서 순서를 보장합니다.
    *   **결과**: 기다림 없는 초고속 병렬 처리.

---

## 3. 핵심 기능 (Features) - 2

### 📡 Market Data Feed 핸들러
*   **UDP Multicast**: 거래소 시세 데이터 고속 수신
*   **Zero Copy / SBE**: 역직렬화(Deserialization) 비용 최소화
*   **패킷 손실 감지**: Sequence Number Tracking

---

## 3-3. Deep Dive: UDP Multicast

**"1:1 전화 통화 vs 라디오 방송"**

*   **TCP (전화)**: "여보세요? 들리니? 응 들려." (연결 확인 필수, 느림)
*   **UDP Multicast (라디오)**: 방송국(거래소)이 **"지금 삼성전자 7만원!"** 하고 쏘면, 라디오(트레이딩 봇)들은 그냥 듣기만 하면 됩니다.
*   **장점**: 연결 절차가 없고, 수천 명에게 동시에 데이터를 보낼 수 있어 압도적으로 빠릅니다.

---

## 3-4. Deep Dive: Zero Copy & SBE

**"번역하지 말고 원문 그대로 읽어라"**

*   **기존 방식 (JSON/XML)**:
    `{ "price": 100 }` -> 글자를 읽고, 숫자로 변환하고, 메모리에 복사... (느림)
*   **해결 (SBE, Simple Binary Encoding)**:
    *   데이터를 **010101...** 바이너리(기계어) 형태로 그대로 보냅니다.
    *   **Zero Copy**: 받은 데이터를 복사하지 않고, 메모리 주소만 가리켜서 바로 읽습니다.
    *   **결과**: 눈으로 보는 순간 뇌에 입력되는 수준의 속도.

---

## 3-5. Deep Dive: Packet Loss Detection

**"빠진 페이지 찾기"**

*   **문제**: UDP는 데이터를 막 던지기 때문에 중간에 배달 사고(유실)가 나도 모릅니다. "가, 나, 다, ... 마" (라? '라' 어디 갔어?)
*   **해결**: **Sequence Number (일련번호)**
    *   모든 메시지에 번호를 매깁니다. `Msg 1`, `Msg 2`, `Msg 3`...
    *   `1`번 받고 `3`번이 오면? -> **"2번 빠졌다! 비상!"**
    *   즉시 감지하고 로그를 남기거나, 재요청 로직(Recovery)을 수행합니다.

---

## 4. 비기능 요구사항 (Non-Functionals)

| 구분 | 목표치 / 내용 |
| :--- | :--- |
| **Latency** | 평균 **10μs** 미만 (Inbound -> Matching) |
| **Throughput** | **> 1,000,000** updates/sec |
| **Reliability** | **Zero Data Loss**, Zero Deadlock |
| **Scalability** | 멀티 종목(Symbol) 처리 구조 |

---

## 5. 기술 스택 (Tech Stack)

*   **Language**: Modern C++ (Template Meta-programming)
*   **OS**: Linux (Kernel Bypass, CPU Isolation)
*   **Network**: UDP Multicast, Raw Sockets
*   **Concurrency**: Lock-free Queues (SPSC), Atomic Operations
*   **Tools**: Perf, Valgrind, GDB

---

## 6. 아키텍처 전략 (Architecture)

1.  **자료구조 최적화**
    *   Cache-friendly Memory Layout
    *   Object Pool / Memory Pool (No `new/delete`)
2.  **Concurrency**
    *   **Lock-free**: Critical Path에서 Mutex 제거
    *   **Thread Pinning**: CPU Core 독점 할당
3.  **Network**
    *   Kernel Bypass (Experimental)
    *   Simple Binary Encoding (SBE)

---

## 7. 마일스톤 (Milestones)

1.  **Phase 1**: Order Book & Matching Engine (Single Thread)
2.  **Phase 2**: UDP Multicast Reciever & Parser
3.  **Phase 3**: Lock-free Integration (Producer-Consumer)
4.  **Phase 4**: Optimization & Benchmarking

---

# Q & A

**Thank You**
