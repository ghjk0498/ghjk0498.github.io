# Synchronization: Mutex, Semaphore, Monitor

## 📌 핵심 요약
- **Mutex**: 상호 배제를 위한 이진 락, 한 번에 하나의 쓰레드만 임계 영역 접근 허용
- **Semaphore**: 카운팅 기반 동기화 도구, 제한된 수의 쓰레드가 자원에 동시 접근 가능
- **Monitor**: 고수준 동기화 구조체, 데이터와 동기화 코드를 하나로 캡슐화
- 모두 Race Condition과 동시성 문제 해결을 위한 동기화 메커니즘

## 🎯 주요 개념

### 동기화가 필요한 이유
```
문제 상황: Race Condition
쓰레드 A: balance = 1000, balance = balance + 100 → balance = 1100
쓰레드 B: balance = 1000, balance = balance - 50  → balance = 950

결과: 마지막에 실행된 연산만 반영됨 (데이터 불일치)
해결: 임계 영역(Critical Section)에 대한 상호 배제 필요
```

### 임계 영역 (Critical Section)
**정의**: 공유 자원에 접근하는 코드 부분으로, 한 번에 하나의 쓰레드만 실행되어야 하는 영역

**조건**:
- **상호 배제(Mutual Exclusion)**: 한 번에 하나의 프로세스만 실행
- **진행(Progress)**: 임계영역이 비어있으면 진입 허용
- **유한 대기(Bounded Waiting)**: 무한 대기 방지

## 📊 Mutex (Mutual Exclusion)

### 기본 개념
**정의**: 이진 세마포어의 특수한 형태로, 락(Lock)을 소유한 쓰레드만이 공유 자원에 접근 가능

```
상태: Locked(1) 또는 Unlocked(0)
소유권: 락을 획득한 쓰레드만이 해제 가능
용도: 임계 영역 보호
```

### 동작 원리
```cpp
// Mutex 기본 동작
mutex.lock();     // 임계영역 진입
// Critical Section (공유 자원 접근)
mutex.unlock();   // 임계영역 탈출
```

### 구현 예시

**C++ 예시**
```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx;
int shared_counter = 0;

void increment() {
    for (int i = 0; i < 100000; ++i) {
        mtx.lock();           // 뮤텍스 획득
        shared_counter++;     // 임계 영역
        mtx.unlock();         // 뮤텍스 해제
    }
}

// RAII 방식 (권장)
void safe_increment() {
    for (int i = 0; i < 100000; ++i) {
        std::lock_guard<std::mutex> lock(mtx);  // 자동 해제
        shared_counter++;
    }
}
```

**Python 예시**
```python
import threading
import time

mutex = threading.Lock()
shared_data = 0

def worker():
    global shared_data
    for _ in range(100000):
        with mutex:  # 컨텍스트 매니저 사용
            shared_data += 1

# 또는 명시적 사용
def worker_explicit():
    global shared_data
    for _ in range(100000):
        mutex.acquire()
        try:
            shared_data += 1
        finally:
            mutex.release()
```

> 💡 **핵심 포인트**: Mutex는 소유권 개념이 있어 락을 획득한 쓰레드만이 해제 가능

## 📊 Semaphore

### 기본 개념
**정의**: 정수 카운터를 사용하여 동시에 자원에 접근할 수 있는 쓰레드 수를 제한하는 동기화 도구

```
카운터: 사용 가능한 자원의 개수
P연산(wait): 카운터 감소, 0이면 대기
V연산(signal): 카운터 증가, 대기 중인 쓰레드 깨움
```

### 세마포어 종류

**이진 세마포어 (Binary Semaphore)**
```
카운터 값: 0 또는 1
용도: Mutex와 유사하지만 소유권 개념 없음
```

**카운팅 세마포어 (Counting Semaphore)**
```
카운터 값: 0 이상의 정수
용도: 제한된 자원 풀 관리 (DB 연결, 프린터 등)
```

### 구현 예시

**C++ 예시**
```cpp
#include <semaphore>
#include <thread>
#include <iostream>

// C++20 표준 세마포어
std::counting_semaphore<3> pool_semaphore(3);  // 최대 3개 동시 접근

void worker(int id) {
    pool_semaphore.acquire();  // P연산 (카운터 감소)
    
    std::cout << "작업자 " << id << " 자원 사용 시작\n";
    std::this_thread::sleep_for(std::chrono::seconds(2));
    std::cout << "작업자 " << id << " 자원 사용 완료\n";
    
    pool_semaphore.release();  // V연산 (카운터 증가)
}
```

**Python 예시**
```python
import threading
import time

# 최대 3개의 쓰레드만 동시 실행 허용
semaphore = threading.Semaphore(3)

def worker(worker_id):
    semaphore.acquire()  # P연산
    try:
        print(f"작업자 {worker_id} 작업 시작")
        time.sleep(2)  # 작업 시뮬레이션
        print(f"작업자 {worker_id} 작업 완료")
    finally:
        semaphore.release()  # V연산

# 10개 쓰레드 생성하지만 3개씩만 동시 실행
threads = []
for i in range(10):
    t = threading.Thread(target=worker, args=(i,))
    threads.append(t)
    t.start()
```

### 실제 활용 사례
```python
# 데이터베이스 연결 풀 관리
class DatabasePool:
    def __init__(self, max_connections=5):
        self.semaphore = threading.Semaphore(max_connections)
        self.connections = [create_connection() for _ in range(max_connections)]
    
    def get_connection(self):
        self.semaphore.acquire()
        return self.connections.pop()
    
    def return_connection(self, conn):
        self.connections.append(conn)
        self.semaphore.release()
```

## 📊 Monitor

### 기본 개념
**정의**: 공유 데이터와 그 데이터를 조작하는 함수들을 하나의 모듈로 캡슐화한 고수준 동기화 구조

```
구성 요소:
- 공유 데이터
- 데이터 접근 함수들
- 초기화 코드
- 조건 변수 (Condition Variable)
```

### 특징
- **상호 배제**: 한 번에 하나의 프로세스만 모니터 내부 실행
- **조건 동기화**: wait/signal을 통한 조건부 대기
- **캡슐화**: 데이터와 동기화 로직을 함께 관리

### 조건 변수 (Condition Variable)

**기본 연산**
```cpp
condition.wait(lock);     // 조건이 만족될 때까지 대기
condition.notify_one();   // 대기 중인 쓰레드 하나 깨움
condition.notify_all();   // 대기 중인 모든 쓰레드 깨움
```

### 구현 예시

**C++ Monitor 패턴**
```cpp
#include <mutex>
#include <condition_variable>
#include <queue>

class BoundedBuffer {
private:
    std::queue<int> buffer;
    std::mutex mtx;
    std::condition_variable not_full;
    std::condition_variable not_empty;
    size_t max_size;

public:
    BoundedBuffer(size_t size) : max_size(size) {}
    
    void put(int item) {
        std::unique_lock<std::mutex> lock(mtx);
        
        // 버퍼가 가득 찰 때까지 대기
        not_full.wait(lock, [this] { return buffer.size() < max_size; });
        
        buffer.push(item);
        std::cout << "아이템 " << item << " 추가됨\n";
        
        not_empty.notify_one();  // 소비자에게 알림
    }
    
    int get() {
        std::unique_lock<std::mutex> lock(mtx);
        
        // 버퍼가 비어있지 않을 때까지 대기
        not_empty.wait(lock, [this] { return !buffer.empty(); });
        
        int item = buffer.front();
        buffer.pop();
        std::cout << "아이템 " << item << " 제거됨\n";
        
        not_full.notify_one();  // 생산자에게 알림
        
        return item;
    }
};
```

**Python Monitor 패턴**
```python
import threading
import time
from collections import deque

class BoundedBuffer:
    def __init__(self, max_size):
        self.buffer = deque()
        self.max_size = max_size
        self.mutex = threading.Lock()
        self.not_full = threading.Condition(self.mutex)
        self.not_empty = threading.Condition(self.mutex)
    
    def put(self, item):
        with self.not_full:
            while len(self.buffer) >= self.max_size:
                self.not_full.wait()  # 버퍼가 가득 참
            
            self.buffer.append(item)
            print(f"아이템 {item} 추가됨")
            
            self.not_empty.notify()  # 소비자 깨움
    
    def get(self):
        with self.not_empty:
            while len(self.buffer) == 0:
                self.not_empty.wait()  # 버퍼가 비어있음
            
            item = self.buffer.popleft()
            print(f"아이템 {item} 제거됨")
            
            self.not_full.notify()  # 생산자 깨움
            
            return item

# 생산자-소비자 문제 해결
def producer(buffer):
    for i in range(5):
        buffer.put(i)
        time.sleep(1)

def consumer(buffer):
    for _ in range(5):
        item = buffer.get()
        time.sleep(1.5)
```

## 💡 비교 분석

### 복잡도와 사용성
| 동기화 도구 | 복잡도 | 사용 편의성 | 표현력 |
|-------------|---------|-------------|---------|
| **Mutex** | 낮음 | 높음 | 낮음 |
| **Semaphore** | 중간 | 중간 | 중간 |
| **Monitor** | 높음 | 낮음 | 높음 |

### 적용 상황

**Mutex 사용**
```
✅ 단순한 임계 영역 보호
✅ 빠른 성능이 필요한 경우
✅ 소유권이 중요한 경우
❌ 복잡한 동기화 로직
```

**Semaphore 사용**
```
✅ 자원 풀 관리 (DB 연결, 프린터 등)
✅ 생산자-소비자 문제
✅ 동시 접근 수 제한
❌ 복잡한 조건부 동기화
```

**Monitor 사용**
```
✅ 복잡한 동기화 로직
✅ 조건부 대기가 필요한 경우
✅ 데이터와 동기화를 함께 관리
❌ 단순한 락킹만 필요한 경우
```

## ⚠️ 주의사항과 문제점

### 데드락 (Deadlock)
```cpp
// 데드락 발생 예시
std::mutex mutex1, mutex2;

void thread1() {
    mutex1.lock();
    mutex2.lock();  // thread2가 이미 획득
    // 작업
    mutex2.unlock();
    mutex1.unlock();
}

void thread2() {
    mutex2.lock();
    mutex1.lock();  // thread1이 이미 획득
    // 작업
    mutex1.unlock();
    mutex2.unlock();
}
```

**해결 방법**
```cpp
// 락 순서 통일
void safe_thread1() {
    std::lock(mutex1, mutex2);  // 동시 획득
    std::lock_guard<std::mutex> lg1(mutex1, std::adopt_lock);
    std::lock_guard<std::mutex> lg2(mutex2, std::adopt_lock);
    // 작업
}
```

### 우선순위 역전 (Priority Inversion)
> ⚠️ **주의사항**: 높은 우선순위 쓰레드가 낮은 우선순위 쓰레드를 기다리는 현상

### 기아 상태 (Starvation)
```python
# 해결: 타임아웃 설정
if mutex.acquire(timeout=5.0):
    try:
        # 임계 영역
        pass
    finally:
        mutex.release()
else:
    print("락 획득 실패 - 타임아웃")
```

## 🔗 관련 개념

### 선행 지식
- 프로세스와 쓰레드 개념
- 임계 영역과 Race Condition
- 원자적 연산(Atomic Operation)

### 연관 기법
- **스핀락 (Spinlock)**: CPU를 계속 사용하며 대기
- **읽기-쓰기 락**: 읽기는 동시 허용, 쓰기는 배타적
- **무잠금 프로그래밍 (Lock-free)**: CAS 연산 활용

### 심화 학습
- **메모리 모델**: Sequential Consistency, Memory Barrier
- **원자적 연산**: Compare-and-Swap, Load-Link/Store-Conditional
- **비동기 프로그래밍**: async/await, Future/Promise

## 📚 실제 문제 해결 예시

### 생산자-소비자 문제
```python
import threading
import queue
import time

# Python의 Queue는 내부적으로 Monitor 패턴 사용
buffer = queue.Queue(maxsize=5)

def producer():
    for i in range(10):
        buffer.put(f"아이템-{i}")
        print(f"생산: 아이템-{i}")
        time.sleep(0.5)

def consumer():
    while True:
        try:
            item = buffer.get(timeout=2)
            print(f"소비: {item}")
            buffer.task_done()
            time.sleep(1)
        except queue.Empty:
            break
```

### 읽기-쓰기 문제
```python
import threading

class ReadWriteLock:
    def __init__(self):
        self.read_ready = threading.Condition(threading.RLock())
        self.readers = 0
    
    def acquire_read(self):
        self.read_ready.acquire()
        try:
            self.readers += 1
        finally:
            self.read_ready.release()
    
    def release_read(self):
        self.read_ready.acquire()
        try:
            self.readers -= 1
            if self.readers == 0:
                self.read_ready.notifyAll()
        finally:
            self.read_ready.release()
    
    def acquire_write(self):
        self.read_ready.acquire()
        while self.readers > 0:
            self.read_ready.wait()
    
    def release_write(self):
        self.read_ready.release()
```

## 🎯 선택 가이드

### 언제 무엇을 사용할까?

**간단한 보호가 필요할 때 → Mutex**
```python
# 카운터, 플래그 변수 보호
counter_lock = threading.Lock()
with counter_lock:
    counter += 1
```

**자원 수 제한이 필요할 때 → Semaphore**
```python
# 동시 다운로드 수 제한
download_semaphore = threading.Semaphore(3)
with download_semaphore:
    download_file()
```

**복잡한 조건 동기화가 필요할 때 → Monitor**
```python
# 생산자-소비자, 읽기-쓰기 문제
class SafeBuffer:
    def __init__(self):
        self.condition = threading.Condition()
        # 복잡한 동기화 로직
```

## 🏷️ 태그
#동기화 #동시성 #병렬프로그래밍 #뮤텍스 #세마포어 #모니터 #임계영역 #중급
