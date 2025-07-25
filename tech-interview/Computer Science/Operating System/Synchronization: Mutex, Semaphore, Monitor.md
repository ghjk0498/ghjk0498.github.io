# Synchronization: Mutex, Semaphore, Monitor

## 📌 핵심 요약
- **동기화(Synchronization)**는 여러 프로세스나 스레드가 공유 자원에 동시 접근할 때 발생하는 문제를 해결하는 메커니즘
- Mutex, Semaphore, Monitor는 대표적인 동기화 도구로, 각각 다른 특성과 사용 목적을 가짐
- 임계 구역(Critical Section) 보호와 경쟁 상태(Race Condition) 방지가 주요 목적

## 🎯 주요 개념

### 기본 정의
- **Mutex (Mutual Exclusion)**: 상호 배제를 보장하는 이진 세마포어의 특별한 형태
- **Semaphore**: 정수 값을 가지는 동기화 도구로, 여러 자원의 개수를 관리
- **Monitor**: 상호 배제와 조건 동기화를 캡슐화한 고수준 동기화 구조

### 핵심 원리
> 💡 **핵심 포인트**: 모든 동기화 메커니즘은 "한 번에 하나의 프로세스/스레드만 임계 구역에 진입"이라는 원칙을 구현

### 주요 특징
| 구분 | Mutex | Semaphore | Monitor |
|------|-------|-----------|---------|
| 값의 범위 | 0 또는 1 | 0 이상의 정수 | N/A (내부적으로 관리) |
| 소유권 | 있음 | 없음 | 있음 |
| 사용 난이도 | 중간 | 높음 | 낮음 |
| 주요 용도 | 상호 배제 | 자원 개수 제한 | 복잡한 동기화 |

## 📊 상세 내용

### 1. Mutex (Mutual Exclusion)

#### 작동 원리
- Lock과 Unlock 두 가지 연산만 존재
- 한 스레드가 Lock을 획득하면, 다른 스레드는 대기
- Lock을 획득한 스레드만 Unlock 가능 (소유권 존재)

#### 특징
- **재진입 가능(Reentrant)**: 같은 스레드가 여러 번 Lock 가능
- **우선순위 역전 문제**: 낮은 우선순위 스레드가 Lock을 가지고 있을 때 발생
- **Deadlock 위험**: 여러 Mutex를 잘못된 순서로 획득할 때

> ⚠️ **주의사항**: Mutex는 반드시 Lock을 획득한 스레드가 Unlock해야 함

### 2. Semaphore

#### 작동 원리
- P(wait) 연산: 세마포어 값 감소, 0이면 대기
- V(signal) 연산: 세마포어 값 증가, 대기 중인 프로세스 깨움
- 초기값에 따라 동시 접근 가능한 프로세스 수 결정

#### 종류
- **Binary Semaphore**: 0과 1만 가지는 세마포어 (Mutex와 유사)
- **Counting Semaphore**: 여러 개의 자원을 관리

#### 특징
- 소유권 개념이 없어 어떤 프로세스든 signal 가능
- 순서 제어에 유용 (프로세스 간 실행 순서 조정)
- 생산자-소비자 문제 해결에 적합

### 3. Monitor

#### 작동 원리
- 공유 데이터와 이를 접근하는 프로시저를 하나의 모듈로 묶음
- 한 시점에 하나의 프로세스만 모니터 내부에서 활동
- 조건 변수(Condition Variable)를 통한 대기와 신호

#### 구성 요소
- **진입 큐(Entry Queue)**: 모니터 진입 대기
- **조건 변수(Condition Variable)**: wait()와 signal() 연산
- **상호 배제**: 자동으로 보장됨

#### 특징
- 프로그래머가 직접 동기화 코드를 작성할 필요 없음
- 객체 지향 프로그래밍과 잘 어울림
- Java의 synchronized 키워드가 대표적 예시

> 📌 **기억할 점**: Monitor는 고수준 동기화 도구로, Mutex와 Condition Variable을 캡슐화한 것

## 💡 실제 활용

### 적용 사례

#### Mutex 사용 예시
- 파일 시스템의 파일 접근 제어
- 데이터베이스 레코드 수정
- 공유 메모리 영역 보호

#### Semaphore 사용 예시
- 프린터 스풀러 (제한된 프린터 자원)
- 데이터베이스 연결 풀 관리
- 네트워크 연결 수 제한

#### Monitor 사용 예시
- Java의 synchronized 메서드
- 생산자-소비자 버퍼 관리
- 읽기-쓰기 문제 해결

### 장단점 분석

| 도구 | 장점 | 단점 |
|------|------|------|
| **Mutex** | • 구현이 직관적<br>• 소유권으로 안전성 보장 | • Deadlock 가능성<br>• 우선순위 역전 문제 |
| **Semaphore** | • 유연한 자원 관리<br>• 프로세스 간 신호 전달 | • 프로그래밍 실수 가능성 높음<br>• 디버깅 어려움 |
| **Monitor** | • 사용이 간편<br>• 자동 상호 배제 | • 언어 수준 지원 필요<br>• 세밀한 제어 어려움 |

### 선택 기준
> 🔍 **심화 학습**: 각 상황에 맞는 동기화 도구 선택이 중요

1. **단순 상호 배제**: Mutex
2. **여러 자원 관리**: Counting Semaphore
3. **복잡한 동기화 로직**: Monitor
4. **프로세스 간 동기화**: Semaphore (IPC 지원)

## 🔗 관련 주제

### 선행 지식
- 프로세스와 스레드의 개념
- 임계 구역(Critical Section)
- 경쟁 상태(Race Condition)
- Deadlock의 4가지 조건

### 연관 개념
- 조건 변수(Condition Variable)
- 읽기-쓰기 락(Reader-Writer Lock)
- 스핀락(Spinlock)
- 락프리(Lock-free) 프로그래밍

### 심화 학습 경로
1. 동기화 기초 → **현재 주제** → 고급 동기화 패턴
2. 운영체제 동기화 → 분산 시스템 동기화
3. 언어별 동기화 구현 (Java, C++, Python)

## 📚 참고 자료

### 핵심 문헌
- "Operating System Concepts" - Silberschatz, Galvin, Gagne
- "The Art of Multiprocessor Programming" - Maurice Herlihy, Nir Shavit
- "Modern Operating Systems" - Andrew S. Tanenbaum

### 추천 리소스
- [Little Book of Semaphores](https://greenteapress.com/wp/semaphores/)
- POSIX Threads Programming Guide
- Java Concurrency in Practice

### 추가 학습 자료
- 운영체제 강의 노트 (동기화 부분)
- 멀티스레드 프로그래밍 실습 예제
- 동시성 제어 디자인 패턴

## 🏷️ 태그
#동기화 #운영체제 #멀티스레딩 #동시성제어 #중급
