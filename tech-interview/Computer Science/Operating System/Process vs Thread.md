# Process vs Thread

## 📌 핵심 요약
- **Process(프로세스)**: 실행 중인 프로그램의 독립적인 인스턴스로, 자체 메모리 공간을 가짐
- **Thread(스레드)**: 프로세스 내에서 실행되는 가장 작은 실행 단위로, 프로세스의 자원을 공유
- 멀티태스킹과 동시성 프로그래밍의 핵심 개념으로, 시스템 자원 활용과 성능 최적화에 필수적

## 🎯 주요 개념

### 기본 정의
- **프로세스**: 운영체제로부터 자원을 할당받는 작업의 단위
- **스레드**: 프로세스가 할당받은 자원을 이용하는 실행의 단위

### 핵심 원리
```
프로세스 구조:
┌─────────────────────────┐
│      Process A          │
│  ┌─────┬─────┬─────┐   │
│  │Code │Data │Heap │   │
│  └─────┴─────┴─────┘   │
│  ┌─────────────────┐   │
│  │     Stack       │   │
│  └─────────────────┘   │
└─────────────────────────┘

스레드 구조:
┌─────────────────────────────────┐
│           Process               │
│  ┌─────┬─────┬─────────────┐   │
│  │Code │Data │    Heap     │   │ ← 공유 영역
│  └─────┴─────┴─────────────┘   │
│  ┌──────┐ ┌──────┐ ┌──────┐   │
│  │Stack1│ │Stack2│ │Stack3│   │ ← 각 스레드별 독립
│  └──────┘ └──────┘ └──────┘   │
│   Thread1  Thread2  Thread3    │
└─────────────────────────────────┘
```

### 주요 특징

| 특성 | Process | Thread |
|------|---------|---------|
| 메모리 공간 | 독립적 | 공유 (Stack 제외) |
| 생성/종료 비용 | 높음 | 낮음 |
| 통신 방식 | IPC (Inter-Process Communication) | 공유 메모리 |
| 컨텍스트 스위칭 | 오버헤드 큼 | 오버헤드 작음 |
| 안정성 | 다른 프로세스에 영향 없음 | 하나가 죽으면 전체 영향 |

## 📊 상세 내용

### 이론적 배경

#### 프로세스의 메모리 구조
```
┌─────────────────┐ High Address
│      Stack      │ ← 지역 변수, 함수 매개변수
│        ↓        │   (아래로 증가)
│                 │
│        ↑        │   (위로 증가)
│      Heap       │ ← 동적 할당 메모리
├─────────────────┤
│      Data       │ ← 전역 변수, 정적 변수
├─────────────────┤
│      Text       │ ← 프로그램 코드
└─────────────────┘ Low Address
```

#### 스레드의 공유/독립 자원
- **공유 자원**: Code, Data, Heap, 열린 파일, 시그널
- **독립 자원**: Stack, CPU 레지스터, Thread ID, PC(Program Counter)

### 구성 요소

#### Process Control Block (PCB)
```python
class PCB:
    def __init__(self):
        self.process_id = None      # PID
        self.process_state = None   # New, Ready, Running, Waiting, Terminated
        self.program_counter = None # 다음 실행할 명령어 주소
        self.cpu_registers = {}     # CPU 레지스터 정보
        self.memory_info = {}       # 메모리 관리 정보
        self.io_status = []         # I/O 상태 정보
        self.accounting_info = {}   # CPU 사용시간, 시간제한 등
```

#### Thread Control Block (TCB)
```python
class TCB:
    def __init__(self):
        self.thread_id = None       # TID
        self.thread_state = None    # 스레드 상태
        self.program_counter = None # PC
        self.register_set = {}      # 레지스터 집합
        self.stack_pointer = None   # 스택 포인터
        self.parent_process = None  # 부모 프로세스 참조
```

### 작동 원리

#### 프로세스 생성 (Fork)
```python
import os

def create_process_example():
    pid = os.fork()
    
    if pid == 0:
        # 자식 프로세스
        print(f"자식 프로세스 PID: {os.getpid()}")
        print(f"부모 프로세스 PID: {os.getppid()}")
    else:
        # 부모 프로세스
        print(f"부모 프로세스 PID: {os.getpid()}")
        print(f"생성된 자식 프로세스 PID: {pid}")
```

#### 스레드 생성
```python
import threading

def worker_thread(name):
    print(f"스레드 {name} 시작")
    # 작업 수행
    print(f"스레드 {name} 종료")

# Python 예제
threads = []
for i in range(3):
    t = threading.Thread(target=worker_thread, args=(f"Thread-{i}",))
    threads.append(t)
    t.start()

for t in threads:
    t.join()
```

```java
// Java 예제
class WorkerThread extends Thread {
    private String name;
    
    public WorkerThread(String name) {
        this.name = name;
    }
    
    @Override
    public void run() {
        System.out.println("스레드 " + name + " 시작");
        // 작업 수행
        System.out.println("스레드 " + name + " 종료");
    }
}

// 사용
for (int i = 0; i < 3; i++) {
    WorkerThread thread = new WorkerThread("Thread-" + i);
    thread.start();
}
```

## 💡 실제 활용

### 적용 사례

#### 멀티프로세스 적합 상황
```python
import multiprocessing

def cpu_intensive_task(n):
    # CPU 집약적 작업 (예: 대규모 계산)
    result = sum(i * i for i in range(n))
    return result

# 멀티프로세스로 병렬 처리
if __name__ == '__main__':
    with multiprocessing.Pool() as pool:
        tasks = [1000000, 1000000, 1000000, 1000000]
        results = pool.map(cpu_intensive_task, tasks)
        print(results)
```

#### 멀티스레드 적합 상황
```python
import threading
import requests
import time

def fetch_url(url):
    # I/O 집약적 작업 (예: 네트워크 요청)
    response = requests.get(url)
    print(f"{url}: {response.status_code}")

# 멀티스레드로 동시 처리
urls = ['http://example1.com', 'http://example2.com', 'http://example3.com']
threads = []

start_time = time.time()
for url in urls:
    t = threading.Thread(target=fetch_url, args=(url,))
    threads.append(t)
    t.start()

for t in threads:
    t.join()

print(f"실행 시간: {time.time() - start_time}초")
```

### 장단점 분석

#### 프로세스
> 💡 **장점**
> - 독립적 메모리 공간으로 안정성 높음
> - 한 프로세스 문제가 다른 프로세스에 영향 없음
> - 병렬 처리로 진정한 동시성 구현 가능

> ⚠️ **단점**
> - 생성/종료 오버헤드 큼
> - 프로세스 간 통신(IPC) 복잡하고 비용 높음
> - 메모리 사용량 많음

#### 스레드
> 💡 **장점**
> - 생성/종료 오버헤드 작음
> - 메모리 공유로 통신 간단하고 빠름
> - 컨텍스트 스위칭 비용 낮음

> ⚠️ **단점**
> - 동기화 문제 (Race Condition, Deadlock)
> - 한 스레드 오류가 전체 프로세스 영향
> - 디버깅 어려움

### 모범 사례

#### 동기화 처리
```python
import threading

# 공유 자원
counter = 0
lock = threading.Lock()

def increment():
    global counter
    for _ in range(100000):
        # 동기화 없이는 Race Condition 발생
        with lock:
            counter += 1

# 여러 스레드에서 동시 실행
threads = []
for _ in range(5):
    t = threading.Thread(target=increment)
    threads.append(t)
    t.start()

for t in threads:
    t.join()

print(f"최종 카운터 값: {counter}")  # 500000이 보장됨
```

```java
// Java에서의 동기화
public class Counter {
    private int count = 0;
    
    // synchronized 키워드로 동기화
    public synchronized void increment() {
        count++;
    }
    
    public synchronized int getCount() {
        return count;
    }
}
```

## 🔗 관련 주제

### 선행 지식
- 운영체제 기본 개념
- 메모리 구조와 관리
- CPU 스케줄링

### 연관 개념
- **동시성(Concurrency) vs 병렬성(Parallelism)**
- **동기화 기법**: Mutex, Semaphore, Monitor
- **교착상태(Deadlock)**: 발생 조건과 해결 방법
- **IPC 기법**: Pipe, Message Queue, Shared Memory, Socket

### 심화 학습 경로
1. **스레드 풀(Thread Pool)**: 효율적인 스레드 관리
2. **비동기 프로그래밍**: async/await, Future/Promise
3. **Actor 모델**: 동시성 프로그래밍의 대안
4. **코루틴(Coroutine)**: 경량 동시성 구현

## 📚 참고 자료

### 핵심 문헌
- "Operating System Concepts" - Silberschatz, Galvin, Gagne
- "Modern Operating Systems" - Andrew S. Tanenbaum
- "Java Concurrency in Practice" - Brian Goetz

### 추천 리소스
- [Linux 프로세스 관리 문서](https://www.kernel.org/doc/html/latest/admin-guide/mm/index.html)
- [Python threading 공식 문서](https://docs.python.org/3/library/threading.html)
- [Java Concurrency Tutorial](https://docs.oracle.com/javase/tutorial/essential/concurrency/)

### 추가 학습 자료
- 온라인 강의: CS 운영체제 강의 (MIT OpenCourseWare)
- 실습 환경: Docker를 활용한 프로세스 격리 실습
- 성능 분석 도구: htop, ps, jstack, jconsole

## 🏷️ 태그
#운영체제 #동시성프로그래밍 #시스템프로그래밍 #중급

---

> 📌 **기억할 점**: Process는 독립적이고 안전하지만 무겁고, Thread는 가볍고 빠르지만 동기화가 필요합니다. 상황에 맞는 선택이 중요합니다!
