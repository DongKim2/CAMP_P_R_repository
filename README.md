# High-Performance Trading System Prototype

**Ultral-Low Latency Trading System in Modern C++**

본 프로젝트는 초저지연(Low-Latency) 및 고신뢰성(High-Reliability)을 목표로 하는 트레이딩 시스템의 핵심 모듈을 구현한 프로토타입입니다. Renaissance Technologies와 같은 최상위 퀀트 펀드의 기술적 요구사항을 충족시키기 위해 설계되었습니다.

## 🚀 Key Features

*   **초고속 호가창 엔진 (Limit Order Book)**: `std::map`을 사용하지 않는 자료구조 최적화와 Object Pool을 통한 메모리 관리로 극한의 매칭 성능 달성.
*   **UDP Multicast Market Data Handler**: 거래소 시세 데이터를 지연 없이 수신하기 위한 효율적인 네트워크 처르 및 SBE(Simple Binary Encoding) 파싱.
*   **Lock-free Concurrency**: SPSC(Single Producer Single Consumer) 큐를 활용한 Lock-free 아키텍처로 스레드 간 경합 최소화.
*   **Kernel Bypass Ready**: 극한의 성능을 위한 DPDK/Solarflare 활용 구조 설계 (Experimental).

## 🛠 Technology Stack

*   **Language**: Modern C++ (C++17/20)
*   **Build System**: CMake
*   **Platform**: Linux (Optimized for Kernel Bypass & CPU Pinning)
*   **Tools**: Git, GDB, Valgrind, Perf, Google Test, Google Benchmark

## 📂 Project Structure

```
CAMP_P_R_repository/
├── DOCS/
│   ├── Ideation.md    # 초기 아이디어 및 프로젝트 방향성
│   ├── PRD.md         # 제품 요구사항 정의서 (Product Requirements Document)
│   ├── TASKS.md       # 개발 작업 목록 및 마일스톤
│   └── Tutorial.md    # 핵심 기술 개념 및 구현 가이드
├── src/               # 소스 코드 (예정)
├── include/           # 헤더 파일 (예정)
├── tests/             # 단위 테스트 (예정)
└── README.md          # 프로젝트 메인 설명 파일
```

## 📖 Documentation

프로젝트에 대한 자세한 문서는 `DOCS` 폴더에 위치해 있습니다.

1.  **[PRD.md](DOCS/PRD.md)**: 프로젝트의 상세 목표와 기능 명세를 확인하세요.
2.  **[TASKS.md](DOCS/TASKS.md)**: 현재 진행 상황과 앞으로의 개발 계획(Roadmap)을 확인하세요.
3.  **[Tutorial.md](DOCS/Tutorial.md)**: 프로젝트에 사용된 핵심 기술에 대한 튜토리얼을 제공합니다.

## 🚦 Getting Started

### Prerequisites
*   C++ Compiler (GCC 9+ or Clang 10+)
*   CMake 3.10+

### Build
```bash
mkdir build
cd build
cmake ..
make
```

### Run Tests
```bash
./bin/trading_system_test
```
