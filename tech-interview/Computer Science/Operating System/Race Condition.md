# 레이스 컨디션 (Race Condition)

## ❓ 면접 질문
"레이스 컨디션이 무엇인지 설명하고, 어떤 상황에서 발생하는지 예시와 함께 설명해주세요. 그리고 이를 해결하는 방법들을 말씀해주세요."

## ✅ 핵심 답변

### 개념 정리
레이스 컨디션은 **둘 이상의 프로세스나 스레드가 공유 자원에 동시에 접근할 때, 실행 순서에 따라 결과가 달라지는 상황**을 말합니다. 마치 두 사람이 동시에 같은 문을 통과하려다가 충돌하는 것과 같은 개념입니다.

### 상세 설명

#### 발생 조건
1. **공유 자원 존재**: 여러 프로세스/스레드가 접근하는 데이터나 자원
2. **동시 접근**: 최소 하나의 프로세스가 쓰기(write) 작업 수행
3. **동기화 부재**: 접근 순서를 제어하는 메커니즘이 없음

#### 발생 예시
```c
// 전역 변수 (공유 자원)
int counter = 0;

// 스레드 A의 작업
void threadA() {
    for(int i = 0; i < 1000; i++) {
        counter++;  // 1. counter 읽기 2. +1 연산 3. counter에 쓰기
    }
}

// 스레드 B의 작업  
void threadB() {
    for(int i = 0; i < 1000; i++) {
        counter++;  // 1. counter 읽기 2. +1 연산 3. counter에 쓰기
    }
}
```

**예상 결과**: 2000
**실제 결과**: 1000~2000 사이의 불규칙한 값

**발생 시나리오**:
```
시간 → | 스레드 A      | 스레드 B      | counter 값
1      | counter 읽기(0)| -            | 0
2      | +1 연산(1)    | counter 읽기(0)| 0  
3      | counter 쓰기  | +1 연산(1)    | 1
4      | -            | counter 쓰기  | 1 (손실!)
```

### 면접 답변 템플릿
"레이스 컨디션은 여러 프로세스나 스레드가 공유 자원에 동시에 접근할 때 실행 순서에 따라 결과가 달라지는 동시성 문제입니다. 

예를 들어, 두 스레드가 같은 변수를 동시에 증가시키는 경우를 생각해보면, 각 스레드가 '읽기-연산-쓰기' 과정을 거치는데, 이 과정이 원자적(atomic)이지 않아서 중간에 다른 스레드가 끼어들면 예상과 다른 결과가 나올 수 있습니다.

이를 해결하기 위해서는 뮤텍스, 세마포어 같은 동기화 기법을 사용하거나, 원자적 연산을 활용할 수 있습니다."

## 📚 참고할만한 정보

### 해결 방법들

#### 1. 뮤텍스(Mutex) 사용
```c
#include <pthread.h>

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
int counter = 0;

void threadA() {
    for(int i = 0; i < 1000; i++) {
        pthread_mutex_lock(&mutex);
        counter++;  // 임계 구역
        pthread_mutex_unlock(&mutex);
    }
}
```

#### 2. 세마포어(Semaphore) 사용
```c
#include <semaphore.h>

sem_t semaphore;
int counter = 0;

void init() {
    sem_init(&semaphore, 0, 1);  // 초기값 1 (바이너리 세마포어)
}

void threadA() {
    for(int i = 0; i < 1000; i++) {
        sem_wait(&semaphore);    // P 연산
        counter++;               // 임계 구역
        sem_post(&semaphore);    // V 연산
    }
}
```

#### 3. 원자적 연산(Atomic Operations) 사용
```c
#include <stdatomic.h>

atomic_int counter = 0;

void threadA() {
    for(int i = 0; i < 1000; i++) {
        atomic_fetch_add(&counter, 1);  // 원자적 증가
    }
}
```

#### 4. 임계 구역(Critical Section) 설정
```c
// Windows API 예시
CRITICAL_SECTION cs;
int counter = 0;

void init() {
    InitializeCriticalSection(&cs);
}

void threadA() {
    for(int i = 0; i < 1000; i++) {
        EnterCriticalSection(&cs);
        counter++;
        LeaveCriticalSection(&cs);
    }
}
```

### 실무에서의 레이스 컨디션 예시

#### 1. 웹 서버에서의 동시 요청 처리
```java
public class UserService {
    private int userCount = 0;
    
    // 레이스 컨디션 발생 가능
    public void addUser() {
        userCount++;  // 여러 스레드가 동시에 실행하면 문제
    }
    
    // 해결책: synchronized 키워드 사용
    public synchronized void addUserSafe() {
        userCount++;
    }
}
```

#### 2. 데이터베이스 동시 업데이트
```sql
-- 레이스 컨디션 발생 가능한 상황
-- 스레드 1: SELECT balance FROM account WHERE id = 1; (잔액: 1000)
-- 스레드 2: SELECT balance FROM account WHERE id = 1; (잔액: 1000)
-- 스레드 1: UPDATE account SET balance = 800 WHERE id = 1; (200 출금)
-- 스레드 2: UPDATE account SET balance = 500 WHERE id = 1; (500 출금)
-- 결과: 잔액 500 (실제로는 300이어야 함)

-- 해결책: 트랜잭션과 락 사용
BEGIN TRANSACTION;
SELECT balance FROM account WHERE id = 1 FOR UPDATE;
UPDATE account SET balance = balance - 200 WHERE id = 1;
COMMIT;
```

#### 3. 파일 시스템에서의 동시 접근
```python
import threading
import time

file_lock = threading.Lock()

def write_to_file(data):
    with file_lock:  # 락 획득
        with open('shared_file.txt', 'a') as f:
            f.write(data + '\n')
        # 락 자동 해제
```

### 레이스 컨디션의 종류

#### 1. 데이터 레이스 (Data Race)
- 동일한 메모리 위치에 동시 접근
- 최소 하나가 쓰기 작업
- 동기화 없음

#### 2. 일반적인 레이스 컨디션
- 공유 자원에 대한 접근 순서 문제
- 프로그램 로직의 정확성에 영향

### 감지 및 디버깅 방법

#### 1. 정적 분석 도구
- **ThreadSanitizer**: 컴파일 타임에 레이스 컨디션 감지
- **Helgrind**: Valgrind의 스레드 에러 감지 도구

#### 2. 테스팅 기법
```c
// 스트레스 테스트로 레이스 컨디션 노출
void stress_test() {
    pthread_t threads[100];
    
    for(int i = 0; i < 100; i++) {
        pthread_create(&threads[i], NULL, worker_thread, NULL);
    }
    
    for(int i = 0; i < 100; i++) {
        pthread_join(threads[i], NULL);
    }
    
    printf("Expected: %d, Actual: %d\n", EXPECTED_VALUE, actual_value);
}
```

#### 3. 로깅과 모니터링
```java
public class RaceConditionDetector {
    private static final AtomicLong accessCount = new AtomicLong(0);
    
    public void logAccess(String threadName) {
        long count = accessCount.incrementAndGet();
        System.out.println("Thread " + threadName + " access #" + count);
    }
}
```

### 예방 원칙

#### 1. 불변 객체 사용
```java
public final class ImmutableCounter {
    private final int value;
    
    public ImmutableCounter(int value) {
        this.value = value;
    }
    
    public ImmutableCounter increment() {
        return new ImmutableCounter(value + 1);  // 새 객체 반환
    }
    
    public int getValue() { return value; }
}
```

#### 2. 스레드 로컬 변수 활용
```java
public class ThreadSafeCounter {
    private ThreadLocal<Integer> counter = ThreadLocal.withInitial(() -> 0);
    
    public void increment() {
        counter.set(counter.get() + 1);
    }
    
    public int getValue() {
        return counter.get();
    }
}
```

#### 3. 락-프리(Lock-Free) 자료구조 사용
```java
import java.util.concurrent.atomic.AtomicReference;

public class LockFreeStack<T> {
    private AtomicReference<Node<T>> head = new AtomicReference<>();
    
    public void push(T item) {
        Node<T> newNode = new Node<>(item);
        Node<T> currentHead;
        do {
            currentHead = head.get();
            newNode.next = currentHead;
        } while (!head.compareAndSet(currentHead, newNode));
    }
}
```

## 💡 면접 팁

### 면접관이 원하는 키워드
- **동시성 문제**, **공유 자원**, **원자적 연산**
- **임계 구역**, **상호 배제**, **동기화**
- **뮤텍스**, **세마포어**, **데드락 방지**

### 추가 질문 대비
- "데드락과 레이스 컨디션의 차이점은?"
- "락-프리 프로그래밍에 대해 알고 있나요?"
- "실제 프로젝트에서 레이스 컨디션을 경험한 적이 있나요?"

### 실무 경험 어필 포인트
- 멀티스레드 환경에서의 개발 경험
- 성능 최적화를 위한 동시성 제어 경험
- 디버깅 도구 사용 경험

## 🏷️ 태그
`운영체제` `동시성` `멀티스레딩` `동기화` `뮤텍스` `세마포어` `임계구역` `원자적연산` `데이터레이스` `스레드안전성`
