# Product Requirements Document (PRD): High-Performance Trading System Prototype

## 1. 개요 (Overview)
본 프로젝트는 세계 최고의 퀀트 헤지펀드(예: Renaissance Technologies)의 'Real-time Trading Programmer' 포지션에 부합하는 **초저지연(Ultra-Low Latency) 및 고신뢰성(High-Reliability) 트레이딩 시스템 핵심 모듈**을 구현하는 것을 목표로 합니다. 단순한 기능 구현을 넘어, 하드웨어와 소프트웨어의 경계에서 극한의 성능을 이끌어내는 기술적 역량을 입증하기 위한 포트폴리오 프로젝트입니다.

## 2. 목표 (Goals)
*   **극한의 성능 구현**: 마이크로초(μs) 단위의 레이턴시와 초당 수백만 건의 처리를 지원하는 시스템 구축.
*   **기술적 깊이 증명**: Modern C++, Low-level Linux 시스템 프로그래밍, Lock-free 알고리즘 등 고급 기술 스택 활용 능력 입증.
*   **컴퓨터 과학 근본 문제 해결**: 메모리 최적화, 경쟁 상태(Race Condition) 해결, 네트워크 오버헤드 최소화 등 난이도 높은 엔지니어링 챌린지 해결.

## 3. 핵심 기능 (Functional Requirements)

### 3.1. 초고속 호가창 엔진 (Limit Order Book Engine)
*   **주문 관리**: 매수(Bid) 및 매도(Ask) 주문의 생성, 수정, 취소 기능.
*   **주문 매칭**: 가격 및 시간 우선 순위(Price-Time Priority)에 따른 실시간 주문 체결.
*   **호가창 관리**: 가격 레벨(Price Level) 별 수량 집계 및 호가창 상태 유지.
*   **스냅샷 및 데이터 복구**: 시스템 재시작 시 상태 복구를 위한 데이터 지속성(Persistence) 지원(선택 사항).

### 3.2. Market Data Feed 핸들러
*   **UDP Multicast 수신**: 거래소 시세 데이터(UDP 패킷)의 고속 수신.
*   **패킷 파싱**: 수신된 바이너리 데이터의 역직렬화(Deserialization) 및 내부 데이터 구조 변환.
*   **패킷 손실 감지**: 시퀀스 번호 확인을 통한 패킷 유실 탐지 및 로깅.

### 3.3. 성능 측정 및 모니터링
*   **레이턴시 측정**: Inbound 패킷 수신부터 매칭 완료까지의 구간별 소요 시간 정밀 측정.
*   **지터(Jitter) 분석**: 처리 시간의 편차 측정 및 시각화 데이터 제공.

## 4. 비기능 요구사항 (Non-Functional Requirements)

### 4.1. 성능 (Performance)
*   **Latency**: 주문 처리 및 매칭 로직의 평균 레이턴시 10μs 미만 목표.
*   **Throughput**: 초당 100만 건 이상의 주문 업데이트 처리.
*   **Optimization**: Cache line 최적화, Branch prediction 최적화 등을 통한 CPU 사이클 낭비 최소화.

### 4.2. 신뢰성 (Reliability)
*   **Zero Data Loss**: 네트워크 패킷 수신 및 데이터 처리 과정에서의 데이터 유실 방지.
*   **Concurrency**: 멀티 스레드 환경에서의 Deadlock 및 Race Condition 완전 배제.

### 4.3. 확장성 (Scalability)
*   다양한 종목(Symbol) 처리를 위한 유연한 아키텍처 설계.

## 5. 기술 스택 (Tech Stack)

| 구분 | 상세 기술 | 비고 |
| :--- | :--- | :--- |
| **언어** | **Modern C++ (C++17/20)** | Template Meta-programming, Concept, STL Customizing |
| **OS** | **Linux** | Kernel Bypass, CPU Pinning, Isolation |
| **Network** | **TCP/UDP/Multicast** | Raw Sockets, Zero-copy, DPDK/Solarflare (Optional) |
| **Concurrency** | **Lock-free Structures** | SPSC/MPMC Queues, Atomic Operations, Memory Barriers |
| **Tools** | **GDB, Perf, Valgrind** | 프로파일링 및 병목 분석 도구 활용 |

## 6. 아키텍처 및 구현 전략 (Architecture & Implementation Strategy)

### 6.1. 자료구조 최적화
*   STL(`std::map`, `std::set`) 대신 Cache-friendly한 커스텀 자료구조(예: Flat Map, Fixed-size Array based Pool) 사용.
*   동적 메모리 할당 최소화를 위한 Object Pool 및 Memory Pool 도입.

### 6.2. 병렬 처리 및 동기화
*   Critical Path 내 Lock 사용 배제 (Spinlock 또는 Lock-free 알고리즘 적용).
*   Double Buffering 기법을 통한 데이터 읽기/쓰기 경합 해소.

### 6.3. 네트워크 최적화
*   Kernel Bypass 기법(DPDK 또는 Solarflare OpenOnload) 검토 및 적용 실험.
*   SBE(Simple Binary Encoding) 또는 유사한 바이너리 프로토콜을 이용한 고속 파싱 구현.

## 7. 마일스톤 (Milestones)
1.  **Phase 1**: 기본 호가창(Order Book) 자료구조 설계 및 단일 스레드 매칭 엔진 구현.
2.  **Phase 2**: UDP Multicast 수신부 구현 및 더미 데이터(Dummy Data) 피드 연동.
3.  **Phase 3**: Lock-free 큐를 이용한 수신부(Producer)와 처리부(Consumer) 스레드 분리 및 통합.
4.  **Phase 4**: 성능 벤치마킹(Latency/Throughput) 및 프로파일링을 통한 병목 지점 최적화.
