# Tutorial: Building a High-Performance Trading System

이 튜토리얼은 `DOCS/TASKS.md`에 정의된 마일스톤을 따라 초저지연(Low-Latency) 트레이딩 시스템을 구축하는 과정을 안내합니다. 각 단계별 핵심 개념과 구현 팁을 제공합니다.

## Phase 1: 기본 호가창 엔진 구현 (Order Book Engine)

이 단계에서는 시스템의 가장 핵심인 호가창(Order Book)과 매칭 엔진을 개발합니다. 트레이딩 시스템의 심장을 만드는 과정입니다.

### 1.1 환경 설정
*   **Modern C++**: C++17 또는 C++20을 사용합니다. `constexpr`, `std::optional`, `concepts`(C++20) 등 최신 기능을 활용하여 성능과 안정성을 높입니다.
*   **Build System**: CMake는 업계 표준입니다. 복잡한 의존성을 관리하기 쉽도록 초기에 잘 구성해두세요.

### 1.2 자료구조가 성능을 좌우한다
트레이딩 시스템에서 `std::map`이나 `std::unordered_map`은 사용하지 않는 것이 좋습니다. 동적 메모리 할당과 포인터 역참조로 인한 Cache Miss가 발생하기 때문입니다.
*   **대안**: 미리 할당된 배열(Array)이나 사용자 정의 Flat Map을 사용하세요.
*   **Object Pool**: `Order` 객체를 매번 `new/delete` 하지 마세요. 미리 수만 개를 할당해두고 재사용하는 Object Pool 패턴이 필수적입니다.

### 1.3 매칭 로직 (Price-Time Priority)
*   가장 비싼 매수 호가(Best Bid)와 가장 싼 매도 호가(Best Ask)가 겹칠 때 거래가 체결됩니다.
*   같은 가격이라면 먼저 들어온 주문이 우선됩니다.
*   **Tip**: 매칭 로직은 매우 빈번하게 실행되므로 분기(Branch)를 최소화하고 코드를 간결하게 유지하는 것이 중요합니다.

---

## Phase 2: Market Data Feed 핸들러

거래소에서 쏟아지는 시세 데이터를 지연 없이 수신해야 합니다.

### 2.1 UDP Multicast 이해하기
*   거래소는 TCP가 아닌 UDP Multicast로 시세를 뿌립니다. 연결 오버헤드가 없고 다수에게 동시에 전송할 수 있기 때문입니다.
*   **Socket Option**: `SO_REUSEADDR`, `IP_ADD_MEMBERSHIP` 설정을 확인하세요.

### 2.2 SBE (Simple Binary Encoding)
*   JSON이나 XML은 너무 느립니다. 금융권에서는 고정 길이의 바이너리 포맷이나 SBE를 사용합니다.
*   패딩(Padding) 없이 데이터를 구조체(struct)에 매핑(Casting)하거나, 비트 연산으로 직접 값을 추출하여 파싱 속도를 극대화합니다.

### 2.3 패킷 유실 대응
*   UDP는 신뢰성을 보장하지 않습니다. 패킷 헤더의 Sequence Number를 확인하여 중간에 빠진 패킷이 있는지 감지해야 합니다.
*   실전에서는 유실된 패킷을 재요청(Snapshot/Recovery)하는 로직이 필요하지만, 이 프로젝트에서는 유실 감지 후 로그를 남기는 것부터 시작해 봅시다.

---

## Phase 3: 동시성 제어 및 통합 (Concurrency & Integration)

수신(IO)과 처리(Computation)를 병렬화하여 전체 처리량을 높이는 단계입니다.

### 3.1 Lock-free Queue (SPSC Ring Buffer)
*   `std::mutex`는 Context Switch를 유발하여 레이턴시를 크게 증가시킵니다.
*   Producer(수신 스레드)와 Consumer(매칭 스레드) 사이의 데이터 전달에는 Lock-free Ring Buffer를 사용하세요.
*   `std::atomic`의 `memory_order_acquire`, `memory_order_release`를 사용하여 데이터 일관성을 보장하면서도 성능을 챙겨야 합니다.

### 3.2 핵심: 스레드 분리 및 CPU Pinning
*   **Thread Isolation**: 수신 스레드와 워커 스레드가 같은 CPU 코어를 두고 경쟁하지 않도록 분리합니다.
*   **CPU Pinning (Affinity)**: `pthread_setaffinity_np` 등을 사용하여 스레드를 특정 코어에 고정시키세요. OS 스케줄러가 스레드를 다른 코어로 옮기면서 발생하는 Cache Flushing을 막을 수 있습니다.

---

## Phase 4: 성능 최적화 및 벤치마킹 (Optimization)

이제 마이크로초(μs)를 나노초(ns) 단위로 줄여나가는 극한의 최적화 단계입니다.

### 4.1 측정 없이는 최적화도 없다
*   감으로 최적화하지 마세요. `rdtsc` 명령어 등을 사용하여 코드 구간별 소요 사이클을 정밀하게 측정해야 합니다.
*   Linux `perf` 도구를 사용하여 Cache Miss, Branch Misprediction이 발생하는 Hot Spot을 찾으세요.

### 4.2 하드웨어 친화적 코딩
*   **Cache Line (64 bytes)**: 자주 접근하는 데이터가 같은 캐시 라인에 있도록 배치하거나(Data Locality), 불필요한 데이터 공유로 인한 False Sharing을 피하기 위해 `alignas(64)`로 패딩을 넣으세요.
*   **Branch Prediction**: `if (unlikely(condition))` 힌트를 사용하여 CPU가 분기를 예측하도록 도울 수 있습니다.

### 4.3 Kernel Bypass (Advanced)
*   OS 커널의 네트워크 스택을 타는 것조차 느리다면, DPDK나 Solarflare OpenOnload 같은 기술을 사용하여 NIC에서 유저 공간으로 바로 패킷을 쏘아 올릴 수 있습니다. 이 프로젝트의 최종 보스 단계입니다.

---

## 마무리

이 프로젝트를 완성하면 여러분은 단순히 "개발을 할 줄 아는 사람"이 아니라, "컴퓨터 구조와 시스템을 이해하고 극한의 성능을 제어할 수 있는 엔지니어"임을 증명하게 됩니다. 포기하지 말고 끝까지 완주해 보세요!
