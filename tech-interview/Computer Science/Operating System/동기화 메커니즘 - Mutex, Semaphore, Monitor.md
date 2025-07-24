# 동기화 메커니즘 - Mutex, Semaphore, Monitor

## ❓ 면접 질문
"Mutex, Semaphore, Monitor의 차이점과 각각의 사용 목적에 대해 설명해주세요."

## ✅ 핵심 답변

### 개념 정리
동기화 메커니즘은 여러 프로세스나 스레드가 공유 자원에 접근할 때 발생할 수 있는 **경쟁 상태(Race Condition)**를 방지하기 위한 도구들입니다.

### 상세 설명

#### 1. Mutex (Mutual Exclusion)
- **목적**: 한 번에 하나의 스레드만 공유 자원에 접근하도록 보장
- **특징**: Binary 형태 (잠금/해제)
- **동작 원리**: 
  - `lock()`: 자원 사용 전 잠금 획득
  - `unlock()`: 자원 사용 후 잠금 해제
- **사용 사례**: 파일 쓰기, 데이터베이스 업데이트 등

```c
// Mutex 사용 예시
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

void critical_section() {
    pthread_mutex_lock(&mutex);  // 잠금 획득
    // 임계 영역 - 공유 자원 접근
    shared_data++;
    pthread_mutex_unlock(&mutex);  // 잠금 해제
}
```

#### 2. Semaphore (세마포어)
- **목적**: 정해진 개수의 스레드만 동시에 자원에 접근하도록 제한
- **특징**: 카운터 형태 (0 이상의 정수값)
- **동작 원리**:
  - `wait()` 또는 `P()`: 카운터 감소, 0이면 대기
  - `signal()` 또는 `V()`: 카운터 증가, 대기 중인 프로세스 깨움
- **사용 사례**: 연결 풀, 프린터 큐, 주차장 관리 등

```c
// Semaphore 사용 예시
sem_t semaphore;
sem_init(&semaphore, 0, 3);  // 최대 3개 스레드 허용

void access_resource() {
    sem_wait(&semaphore);  // 세마포어 획득
    // 자원 사용 (최대 3개 스레드 동시 접근 가능)
    use_shared_resource();
    sem_post(&semaphore);  // 세마포어 반환
}
```

#### 3. Monitor (모니터)
- **목적**: 고수준 동기화 구조체로 프로시저와 데이터를 캡슐화
- **특징**: 
  - 한 번에 하나의 프로세스만 모니터 내부에서 활성화
  - 조건 변수(Condition Variable)를 통한 대기/신호
- **구성 요소**:
  - 공유 데이터
  - 이 데이터에 접근하는 프로시저들
  - 조건 변수와 wait/signal 연산
- **사용 사례**: 생산자-소비자 문제, 리더-라이터 문제 등

```java
// Monitor 사용 예시 (Java synchronized)
public class BankAccount {
    private int balance = 0;
    
    public synchronized void deposit(int amount) {  // 모니터 진입
        balance += amount;
        notifyAll();  // 대기 중인 스레드들에게 신호
    }
    
    public synchronized void withdraw(int amount) throws InterruptedException {
        while (balance < amount) {
            wait();  // 조건이 만족될 때까지 대기
        }
        balance -= amount;
    }
}
```

### 면접 답변 템플릿
"동기화 메커니즘은 멀티스레딩 환경에서 공유 자원의 안전한 접근을 보장하는 도구입니다.

**Mutex**는 상호 배제를 위한 가장 기본적인 메커니즘으로, 한 번에 하나의 스레드만 임계 영역에 접근할 수 있도록 합니다. 이진 세마포어와 유사하지만, 잠금을 획득한 스레드만이 해제할 수 있다는 소유권 개념이 있습니다.

**Semaphore**는 정수 카운터를 사용하여 정해진 개수의 스레드가 동시에 자원에 접근할 수 있도록 제어합니다. 예를 들어, 데이터베이스 연결 풀에서 최대 10개의 연결만 허용하고 싶을 때 카운터를 10으로 설정한 세마포어를 사용할 수 있습니다.

**Monitor**는 가장 고수준의 동기화 구조로, 공유 데이터와 이를 조작하는 프로시저들을 하나의 모듈로 캡슐화합니다. Java의 synchronized 키워드나 C#의 lock 문이 모니터의 구현 예시입니다."

## 📚 참고할만한 정보

### 심화 개념

#### 동기화 메커니즘 비교표
| 특징 | Mutex | Semaphore | Monitor |
|------|-------|-----------|---------|
| 접근 제한 | 1개 스레드 | N개 스레드 | 1개 스레드 |
| 구현 복잡도 | 단순 | 중간 | 복잡 |
| 소유권 개념 | 있음 | 없음 | 있음 |
| 조건 대기 | 불가능 | 제한적 | 가능 |
| 언어 지원 | 라이브러리 | 라이브러리 | 언어 내장 |

#### 데드락 방지
- **Mutex**: 락 순서 정하기, 타임아웃 설정
- **Semaphore**: 자원 할당 순서 관리
- **Monitor**: 조건 변수 적절한 사용

### 실무 예시

#### 1. 웹 서버 요청 처리
```python
# 연결 풀 관리 (Semaphore 활용)
import threading

connection_pool = threading.Semaphore(10)  # 최대 10개 연결

def handle_request():
    connection_pool.acquire()  # 연결 획득
    try:
        # 데이터베이스 작업 수행
        process_database_query()
    finally:
        connection_pool.release()  # 연결 반환
```

#### 2. 캐시 시스템
```java
// 캐시 동기화 (Monitor 패턴)
public class Cache {
    private Map<String, Object> cache = new HashMap<>();
    
    public synchronized Object get(String key) {
        return cache.get(key);
    }
    
    public synchronized void put(String key, Object value) {
        cache.put(key, value);
        notifyAll();  // 캐시 업데이트 알림
    }
}
```

### 관련 기술
- **읽기-쓰기 락 (Reader-Writer Lock)**: 읽기 작업은 동시 허용, 쓰기 작업은 배타적 처리
- **스핀락 (Spinlock)**: 락을 얻을 때까지 계속 확인하는 바쁜 대기 방식
- **조건 변수 (Condition Variable)**: 특정 조건이 만족될 때까지 대기하는 메커니즘

## 💡 면접 팁
- **성능 비교**: "Mutex는 오버헤드가 적지만, Semaphore는 더 유연한 제어가 가능합니다"
- **실무 경험**: 실제 프로젝트에서 사용한 동기화 패턴 사례 준비
- **문제 해결**: 데드락이나 기아 상태 경험과 해결 방법 설명

## ⚠️ 주의사항
- **데드락**: 여러 락을 동시에 사용할 때 순서 주의
- **기아 상태**: 특정 스레드가 계속 자원을 못 얻는 상황 방지
- **성능**: 과도한 동기화는 성능 저하 원인

## 🏷️ 태그
`#운영체제` `#동기화` `#멀티스레딩` `#Mutex` `#Semaphore` `#Monitor` `#임계영역` `#경쟁상태` `#데드락`

## 🔗 관련 질문
- [프로세스와 스레드 차이점](./프로세스와스레드_차이점.md)
- [데드락 개념과 해결방법](./데드락_개념과해결방법.md)
- [멀티프로세싱 vs 멀티스레딩](./멀티프로세싱vs멀티스레딩.md)
