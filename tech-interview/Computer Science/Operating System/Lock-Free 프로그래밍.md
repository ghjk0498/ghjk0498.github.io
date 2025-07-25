# Lock-Free 프로그래밍

## 📌 핵심 요약
- Lock-Free는 스레드가 잠금(lock) 없이도 동시에 공유 데이터에 안전하게 접근할 수 있는 동시성 프로그래밍 기법
- 최소 하나의 스레드가 항상 진행을 보장하여 교착상태(deadlock)를 방지
- 성능 향상과 확장성을 제공하지만 구현의 복잡도가 높음

## 🎯 주요 개념

### 기본 정의
**Lock-Free**: 여러 스레드가 공유 자원에 접근할 때 상호 배제(mutual exclusion) 없이도 데이터 일관성을 보장하는 알고리즘

### 핵심 원리
- **진행 보장(Progress Guarantee)**: 시스템 전체적으로 최소 하나의 스레드는 항상 작업을 완료할 수 있음
- **원자적 연산(Atomic Operations)**: CAS(Compare-And-Swap) 같은 하드웨어 수준의 원자적 명령어 활용
- **메모리 순서 보장**: 메모리 배리어를 통한 연산 순서 제어

### 주요 특징
- 교착상태 불가능
- 우선순위 역전 문제 없음
- 대기 시간 예측 가능
- 높은 동시성 수준 지원

## 📊 상세 내용

### 이론적 배경
Lock-Free는 1970년대 Leslie Lamport의 연구에서 시작되어, 현대 멀티코어 시스템에서 중요성이 증가했습니다. 

**동시성 수준 계층구조**:
1. Wait-Free (가장 강함)
2. **Lock-Free** ← 우리가 다루는 주제
3. Obstruction-Free
4. Lock-Based (가장 약함)

### 구성 요소

**1. 원자적 연산**
- Compare-And-Swap (CAS)
- Load-Link/Store-Conditional (LL/SC)
- Fetch-And-Add (FAA)

**2. 메모리 모델**
- Sequential Consistency
- Acquire-Release Semantics
- Relaxed Ordering

**3. ABA 문제 해결**
- 버전 번호 사용
- 해저드 포인터(Hazard Pointers)
- 에포크 기반 회수(Epoch-Based Reclamation)

### 작동 원리

Lock-Free 알고리즘의 기본 패턴:
1. 현재 상태를 읽음
2. 새로운 상태를 계산
3. CAS로 상태 업데이트 시도
4. 실패 시 1번부터 재시도

> 💡 **핵심 포인트**: Lock-Free는 "낙관적 동시성 제어"를 사용합니다. 충돌이 드물다고 가정하고, 충돌 시에만 재시도합니다.

## 💡 실제 활용

### 적용 사례

**1. Lock-Free 큐**
- Michael & Scott Queue
- 고성능 메시지 전달 시스템

**2. Lock-Free 스택**
- Treiber Stack
- 메모리 할당자 구현

**3. Lock-Free 해시 테이블**
- Split-Ordered Lists
- 고성능 캐시 시스템

### 장단점 분석

**장점**:
- ✅ 교착상태 없음
- ✅ 높은 처리량(throughput)
- ✅ 예측 가능한 지연시간
- ✅ 뛰어난 확장성

**단점**:
- ❌ 구현 복잡도 높음
- ❌ 디버깅 어려움
- ❌ 플랫폼 의존성
- ❌ 메모리 관리 복잡

### 모범 사례

> ⚠️ **주의사항**: Lock-Free 구현 시 다음 사항을 반드시 고려하세요:
> - 메모리 순서 지정 정확히 이해
> - ABA 문제 대비책 마련
> - 철저한 테스트와 검증
> - 실제로 필요한 경우에만 사용

**구현 가이드라인**:
1. 기존 검증된 라이브러리 우선 사용
2. 단순한 구조부터 시작
3. 성능 측정을 통한 검증
4. 스트레스 테스트 필수

## 🔗 관련 주제

### 선행 지식
- [동시성 프로그래밍 기초](./동시성프로그래밍기초.md)
- [원자적 연산과 메모리 모델](./원자적연산.md)
- [캐시 일관성 프로토콜](./캐시일관성.md)

### 연관 개념
- [Wait-Free 알고리즘](./wait-free.md) - 더 강한 진행 보장
- [Hazard Pointers](./hazard-pointers.md) - 메모리 안전성
- [RCU (Read-Copy-Update)](./rcu.md) - 읽기 최적화 기법

### 심화 학습 경로
1. Lock-Free 자료구조 구현 → 2. 메모리 회수 기법 → 3. 고급 동시성 패턴

> 🔍 **심화 학습**: Lock-Free 프로그래밍을 마스터하려면 하드웨어 아키텍처와 메모리 모델에 대한 깊은 이해가 필요합니다.

## 📚 참고 자료

### 핵심 문헌
- "The Art of Multiprocessor Programming" - Maurice Herlihy, Nir Shavit
- "C++ Concurrency in Action" - Anthony Williams
- "Is Parallel Programming Hard?" - Paul McKenney

### 추천 리소스
- [Preshing on Programming](https://preshing.com/) - Lock-Free 알고리즘 설명
- [1024cores.net](http://www.1024cores.net/) - Dmitry Vyukov의 동시성 블로그
- [Concurrency Freaks](https://concurrencyfreaks.blogspot.com/) - 최신 연구 동향

### 추가 학습 자료
- Lock-Free 프로그래밍 온라인 코스
- 오픈소스 Lock-Free 라이브러리 코드 분석
- 학술 논문: Michael & Scott Queue, Hazard Pointers

## 🏷️ 태그
#Lock-Free #동시성프로그래밍 #원자적연산 #고성능컴퓨팅 #멀티스레딩 #중급난이도

---

> 📌 **기억할 점**: Lock-Free는 강력하지만 복잡한 기술입니다. 대부분의 경우 잘 설계된 lock 기반 솔루션이 충분하며, Lock-Free는 정말 필요한 경우에만 사용하세요.
