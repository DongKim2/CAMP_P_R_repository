# TASKS: High-Performance Trading System

## Phase 1: 기본 호가창 엔진 구현 (Order Book Engine)
단일 스레드 환경에서 동작하는 핵심 호가창 로직을 구현하고 검증합니다.

- [ ] **환경 설정 및 프로젝트 초기화**
    - [ ] CMake 빌드 시스템 구성
    - [ ] Git 저장소 초기화 및 `.gitignore` 설정 (C++, VScode, Build output)
    - [ ] Google Test (gtest) 프레임워크 연동
- [ ] **자료구조 설계 및 구현**
    - [ ] `Order` 구조체 설계 (ID, Price, Quantity, Side, Timestamp)
    - [ ] `OrderBook` 클래스 설계 (Bid/Ask 관리)
    - [ ] `std::map` 대체용 Fixed-size Array 또는 Flat Map 프로토타입 구현
    - [ ] Memory Pool (Object Pool) 구현: `Order` 객체 재사용
- [ ] **매칭 엔진 로직 구현**
    - [ ] `addOrder()`: 지정가 주문(Limit Order) 추가 및 즉시 체결 로직
    - [ ] `cancelOrder()`: 주문 취소 로직
    - [ ] `match()`: Price-Time Priority 기반 매칭 알고리즘 구현
- [ ] **단위 테스트 (Unit Test)**
    - [ ] 기본 주문 추가/취소 테스트
    - [ ] 복합 매칭 시나리오 테스트 (부분 체결, 전량 체결)
    - [ ] Memory Leak 및 안정성 테스트

## Phase 2: Market Data Feed 핸들러
UDP 멀티캐스트를 통해 시세 데이터를 수신하고 파싱하는 모듈을 개발합니다.

- [ ] **네트워크 수신부 구현**
    - [ ] Linux Socket API 활용 UDP Multicast Receiver 구현
    - [ ] Non-blocking I/O 설정 및 `epoll` 연동 (선택 사항)
    - [ ] Packet Capture를 위한 pcap 테스트 도구 준비
- [ ] **데이터 파싱 및 프로토콜 설계**
    - [ ] 더미 거래소(Exchange Simulator) 패킷 생성기 구현
    - [ ] 바이너리 프로토콜(SBE 스타일) 정의 및 파서 구현
    - [ ] Endianness 변환 및 데이터 무결성 검증
- [ ] **통합 테스트**
    - [ ] 송신부(Simulator) -> 수신부(Receiver) 데이터 연동 테스트
    - [ ] 패킷 유실(Sequence Gap) 탐지 로직 검증

## Phase 3: 동시성 제어 및 통합 (Concurrency & Integration)
수신 스레드와 처리 스레드를 분리하고 Lock-free 큐를 통해 연결합니다.

- [ ] **Lock-free Queue 구현**
    - [ ] SPSC (Single Producer Single Consumer) Ring Buffer 구현
    - [ ] `std::atomic` 활용 Memory Order 최적화
    - [ ] Queue Full/Empty 상황에 대한 예외 처리
- [ ] **스레드 분리 및 최적화**
    - [ ] Market Data Consumer Thread (수신) 분리
    - [ ] Order Book Engine Worker Thread (처리) 분리
    - [ ] Thread Affinity (CPU Pinning) 설정 코드 작성
- [ ] **시스템 통합**
    - [ ] Receiver -> Queue -> Engine 파이프라인 연결
    - [ ] 전체 데이터 흐름에 대한 Stress Test 수행

## Phase 4: 성능 최적화 및 벤치마킹 (Optimization)
극한의 레이턴시를 달성하기 위한 미세 조정 및 성능 측정을 수행합니다.

- [ ] **벤치마킹 환경 구축**
    - [ ] Latency 측정용 타임스탬프 로깅 (TSC 활용 등)
    - [ ] Google Benchmark 연동 또는 커스텀 벤치마크 툴 작성
- [ ] **프로파일링 및 병목 제거**
    - [ ] `perf` 툴을 이용한 CPU Cache Miss, Branch Miss 분석
    - [ ] Hot Path 최적화 (Inlining, Branch Hinting `likely/unlikely`)
    - [ ] False Sharing 방지를 위한 Data Alignment (`alignas`) 적용
- [ ] **Kernel Bypass (Optional/Advanced)**
    - [ ] DPDK 또는 Solarflare OpenOnload 환경 조사 및 적용 가능성 테스트
    - [ ] 유저 레벨 네트워크 스택 실험

## Documentation & Wrap-up
- [ ] `README.md` 작성: 빌드 방법, 실행부, 아키텍처 설명
- [ ] 최종 성능 레포트(`DOCS/PERFORMANCE.md`) 작성
