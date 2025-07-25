# Process vs Thread

## 📌 핵심 요약
- **프로세스(Process)**: 운영체제에서 실행 중인 프로그램의 인스턴스로, 독립적인 메모리 공간을 가짐
- **쓰레드(Thread)**: 프로세스 내에서 실제 작업을 수행하는 실행 단위로, 같은 프로세스 내 다른 쓰레드와 메모리 공간을 공유
- 프로세스는 "집", 쓰레드는 "집 안의 거주자"로 비유 가능

## 🎯 주요 개념

### 프로세스 (Process)
**정의**: 메모리에 올라와 실행 중인 프로그램의 인스턴스
- 독립적인 메모리 주소 공간 할당
- 운영체제로부터 자원(CPU 시간, 메모리 등) 할당받음
- 최소 하나 이상의 쓰레드를 포함

### 쓰레드 (Thread)
**정의**: 프로세스 내에서 실제 작업을 수행하는 실행 흐름의 단위
- Light Weight Process(LWP)라고도 불림
- 같은 프로세스 내 쓰레드들과 메모리 공간 공유
- CPU 스케줄링의 기본 단위

## 📊 상세 비교

### 메모리 구조
| 구분 | 프로세스 | 쓰레드 |
|------|----------|---------|
| **메모리 공간** | 독립적 (격리됨) | 공유 (힙, 데이터, 코드 영역) |
| **스택** | 독립적 스택 | 각자 독립적 스택 |
| **힙** | 독립적 힙 | 프로세스 내 모든 쓰레드가 공유 |
| **전역 변수** | 독립적 | 공유 |

### 생성 및 전환 비용
```
프로세스:
- 생성 비용: 높음 (메모리 할당, 초기화 등)
- 컨텍스트 스위칭: 무거움 (메모리 맵 교체 필요)
- 통신 방법: IPC (Inter-Process Communication)

쓰레드:
- 생성 비용: 낮음 (스택만 할당)
- 컨텍스트 스위칭: 가벼움 (레지스터 상태만 저장/복원)
- 통신 방법: 공유 메모리 직접 접근
```

### 독립성과 안정성
> 💡 **핵심 포인트**: 프로세스는 높은 안정성, 쓰레드는 높은 효율성

**프로세스**
- ✅ 한 프로세스 오류가 다른 프로세스에 영향 없음
- ✅ 강력한 보안과 격리
- ❌ 생성/전환 비용이 높음
- ❌ 프로세스 간 통신이 복잡함

**쓰레드**
- ✅ 빠른 생성과 전환
- ✅ 효율적인 메모리 사용
- ❌ 한 쓰레드 오류가 전체 프로세스에 영향
- ❌ 동기화 문제 (Race Condition, Deadlock)

## 💡 실제 활용

### 프로세스 적합한 경우
```python
# 예시: 독립적인 작업들
- 웹 브라우저의 각 탭
- 텍스트 에디터, 미디어 플레이어 등 별도 애플리케이션
- 보안이 중요한 서비스 분리
- 안정성이 최우선인 시스템
```

### 쓰레드 적합한 경우
```python
# 예시: 협력적인 작업들
- 웹 서버의 동시 요청 처리
- 게임의 렌더링, 물리연산, AI 등
- 파일 다운로드 중 UI 응답성 유지
- 대용량 데이터 병렬 처리
```

### 멀티프로세싱 vs 멀티쓰레딩

**멀티프로세싱**
```
장점:
- 한 프로세스 다운되어도 시스템 안정성 유지
- 진정한 병렬 처리 (멀티코어에서)
- 보안성 높음

단점:
- 메모리 사용량 많음
- 프로세스 간 통신 복잡
- 생성/전환 오버헤드 큼
```

**멀티쓰레딩**
```
장점:
- 메모리 효율적
- 빠른 통신과 데이터 공유
- 적은 생성/전환 비용

단점:
- 동기화 복잡성
- 디버깅 어려움
- 한 쓰레드 오류시 전체 영향
```

## 🔗 관련 주제

### 선행 지식
- 운영체제 기본 개념
- 메모리 구조 (스택, 힙, 데이터, 코드 영역)
- CPU 스케줄링

### 연관 개념
- **동기화**: Mutex, Semaphore, Monitor
- **IPC**: 파이프, 메시지 큐, 공유 메모리, 소켓
- **컨텍스트 스위칭**: PCB, TCB
- **동시성 vs 병렬성**: Concurrency vs Parallelism

### 심화 학습 경로
- 프로세스 스케줄링 알고리즘
- 쓰레드 동기화 메커니즘
- 데드락과 해결 방법
- 코루틴(Coroutine)과 비동기 프로그래밍

## 📚 실제 구현 예시

### 프로세스 생성 (Python)
```python
import multiprocessing
import os

def worker_process(name):
    print(f"프로세스 {name}: PID = {os.getpid()}")
    
if __name__ == "__main__":
    # 독립적인 프로세스들 생성
    processes = []
    for i in range(3):
        p = multiprocessing.Process(target=worker_process, args=(f"작업자-{i}",))
        processes.append(p)
        p.start()
    
    for p in processes:
        p.join()
```

### 쓰레드 생성 (Python)
```python
import threading
import time

shared_data = 0  # 모든 쓰레드가 공유하는 데이터

def worker_thread(name):
    global shared_data
    for i in range(5):
        shared_data += 1
        print(f"쓰레드 {name}: shared_data = {shared_data}")
        time.sleep(0.1)

# 같은 프로세스 내에서 쓰레드들 실행
threads = []
for i in range(2):
    t = threading.Thread(target=worker_thread, args=(f"작업자-{i}",))
    threads.append(t)
    t.start()

for t in threads:
    t.join()
```

## ⚠️ 주의사항

### 흔한 오해
- **오해**: "쓰레드가 항상 더 빠르다"
- **실제**: 작업 특성에 따라 다름. I/O 집약적 vs CPU 집약적 작업에 따라 선택

### 동기화 문제
> 🔍 **심화 학습**: Race Condition을 방지하기 위한 동기화 기법 필수

```python
# 문제가 있는 코드 (Race Condition)
import threading

counter = 0

def increment():
    global counter
    for _ in range(100000):
        counter += 1  # 원자적 연산이 아님!

# 해결된 코드 (Lock 사용)
import threading

counter = 0
lock = threading.Lock()

def safe_increment():
    global counter
    for _ in range(100000):
        with lock:  # 동기화
            counter += 1
```

## 🎯 선택 가이드

### 프로세스를 선택해야 할 때
- ✅ 안정성과 보안이 최우선
- ✅ 독립적인 작업들
- ✅ CPU 집약적 작업의 병렬 처리
- ✅ 오류 격리가 중요한 시스템

### 쓰레드를 선택해야 할 때
- ✅ 빠른 응답성이 필요
- ✅ 메모리 효율성이 중요
- ✅ 자주 데이터를 공유해야 함
- ✅ I/O 대기 시간을 활용하고 싶을 때

## 🏷️ 태그
#운영체제 #프로세스 #쓰레드 #동시성 #병렬프로그래밍 #시스템프로그래밍 #중급
