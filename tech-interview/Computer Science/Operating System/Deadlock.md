# 데드락 (Deadlock)

## ❓ 면접 질문
"데드락이 무엇인지 설명하고, 데드락이 발생하는 4가지 조건을 말씀해주세요. 그리고 데드락을 예방하고 해결하는 방법들을 설명해주세요."

## ✅ 핵심 답변

### 개념 정리
데드락은 **둘 이상의 프로세스나 스레드가 서로가 가진 자원을 기다리며 무한정 대기하는 상황**입니다. 마치 좁은 길에서 두 차가 서로 마주보고 서서 둘 다 움직일 수 없는 상황과 같습니다.

### 상세 설명

#### 데드락 발생의 4가지 필요조건 (Coffman 조건)

1. **상호 배제 (Mutual Exclusion)**
   - 한 번에 하나의 프로세스만 자원을 사용할 수 있음
   - 자원을 공유할 수 없는 상태

2. **점유와 대기 (Hold and Wait)**  
   - 프로세스가 최소한 하나의 자원을 보유한 채로 다른 자원을 기다림
   - 이미 할당받은 자원을 놓지 않고 추가 자원을 요청

3. **비선점 (No Preemption)**
   - 다른 프로세스가 사용 중인 자원을 강제로 빼앗을 수 없음
   - 자원을 자발적으로 반납할 때까지 기다려야 함

4. **순환 대기 (Circular Wait)**
   - 프로세스들이 원형으로 서로의 자원을 기다리는 상황
   - P1 → P2 → P3 → P1 형태의 대기 사이클 형성

#### 데드락 발생 예시
```c
// 두 개의 뮤텍스
pthread_mutex_t mutex1 = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_t mutex2 = PTHREAD_MUTEX_INITIALIZER;

// 스레드 A
void* threadA(void* arg) {
    pthread_mutex_lock(&mutex1);    // mutex1 획득
    printf("Thread A: mutex1 획득\n");
    
    sleep(1);  // 의도적 지연
    
    pthread_mutex_lock(&mutex2);    // mutex2 대기 (데드락!)
    printf("Thread A: mutex2 획득\n");
    
    pthread_mutex_unlock(&mutex2);
    pthread_mutex_unlock(&mutex1);
    return NULL;
}

// 스레드 B  
void* threadB(void* arg) {
    pthread_mutex_lock(&mutex2);    // mutex2 획득
    printf("Thread B: mutex2 획득\n");
    
    sleep(1);  // 의도적 지연
    
    pthread_mutex_lock(&mutex1);    // mutex1 대기 (데드락!)
    printf("Thread B: mutex1 획득\n");
    
    pthread_mutex_unlock(&mutex1);
    pthread_mutex_unlock(&mutex2);
    return NULL;
}
```

**데드락 시나리오:**
```
시간 → | 스레드 A        | 스레드 B        | 상태
1      | mutex1 획득     | -              | A: mutex1 보유
2      | -              | mutex2 획득     | A: mutex1, B: mutex2 보유  
3      | mutex2 대기     | mutex1 대기     | 데드락 발생!
```

### 면접 답변 템플릿
"데드락은 두 개 이상의 프로세스가 서로가 점유한 자원을 기다리며 무한정 대기하는 상황입니다.

데드락이 발생하려면 네 가지 조건이 모두 만족되어야 합니다: 상호배제, 점유와 대기, 비선점, 그리고 순환대기입니다.

예를 들어 스레드 A가 자원1을 가지고 자원2를 기다리고, 스레드 B가 자원2를 가지고 자원1을 기다리는 상황에서 데드락이 발생합니다.

해결 방법으로는 예방 기법으로 자원 할당 순서를 정하거나, 회피 기법으로 안전 상태를 유지하거나, 탐지 후 회복하는 방법이 있습니다."

## 📚 참고할만한 정보

### 데드락 해결 방법

#### 1. 예방 (Prevention)
4가지 필요조건 중 하나를 제거하여 데드락을 원천 차단

##### 상호 배제 조건 제거
```c
// 읽기 전용 자원은 공유 가능하게 설계
// 예: 읽기-쓰기 락 사용
pthread_rwlock_t rwlock = PTHREAD_RWLOCK_INITIALIZER;

void reader() {
    pthread_rwlock_rdlock(&rwlock);  // 여러 리더 동시 접근 가능
    // 읽기 작업
    pthread_rwlock_unlock(&rwlock);
}

void writer() {
    pthread_rwlock_wrlock(&rwlock);  // 독점 접근
    // 쓰기 작업  
    pthread_rwlock_unlock(&rwlock);
}
```

##### 점유와 대기 조건 제거
```c
// 방법 1: 모든 자원을 한 번에 요청
int acquire_all_resources() {
    if (try_lock_all(&mutex1, &mutex2, &mutex3)) {
        return SUCCESS;
    }
    return FAILURE;
}

// 방법 2: 자원을 요청하기 전에 보유 자원 모두 해제
void atomic_resource_swap() {
    release_all_resources();
    acquire_needed_resources();
}
```

##### 비선점 조건 제거
```c
// 타임아웃을 이용한 강제 해제
int timed_mutex_lock(pthread_mutex_t* mutex, int timeout_ms) {
    struct timespec ts;
    clock_gettime(CLOCK_REALTIME, &ts);
    ts.tv_sec += timeout_ms / 1000;
    
    int result = pthread_mutex_timedlock(mutex, &ts);
    if (result == ETIMEDOUT) {
        // 타임아웃 시 다른 자원들도 해제
        release_held_resources();
        return TIMEOUT;
    }
    return SUCCESS;
}
```

##### 순환 대기 조건 제거 (가장 일반적)
```c
// 자원 순서 지정으로 데드락 방지
typedef enum {
    RESOURCE_A = 1,
    RESOURCE_B = 2, 
    RESOURCE_C = 3
} resource_id_t;

void acquire_resources_ordered(resource_id_t first, resource_id_t second) {
    if (first < second) {
        lock_resource(first);
        lock_resource(second);
    } else {
        lock_resource(second);
        lock_resource(first);
    }
}

// 사용 예시
void threadA() {
    acquire_resources_ordered(RESOURCE_A, RESOURCE_B);
    // 작업 수행
    unlock_resource(RESOURCE_B);
    unlock_resource(RESOURCE_A);
}

void threadB() {
    acquire_resources_ordered(RESOURCE_B, RESOURCE_A);  // 동일한 순서로 획득
    // 작업 수행  
    unlock_resource(RESOURCE_B);
    unlock_resource(RESOURCE_A);
}
```

#### 2. 회피 (Avoidance)
시스템이 항상 안전 상태를 유지하도록 자원 할당을 제어

##### 은행원 알고리즘 (Banker's Algorithm)
```c
typedef struct {
    int allocation[MAX_PROCESS][MAX_RESOURCE];  // 현재 할당된 자원
    int max[MAX_PROCESS][MAX_RESOURCE];         // 최대 필요 자원
    int available[MAX_RESOURCE];                // 사용 가능한 자원
    int need[MAX_PROCESS][MAX_RESOURCE];        // 추가 필요 자원
} banker_state_t;

bool is_safe_state(banker_state_t* state) {
    bool finish[MAX_PROCESS] = {false};
    int work[MAX_RESOURCE];
    
    // available 자원을 work에 복사
    for (int i = 0; i < MAX_RESOURCE; i++) {
        work[i] = state->available[i];
    }
    
    // 안전 순서 찾기
    int safe_sequence[MAX_PROCESS];
    int count = 0;
    
    while (count < MAX_PROCESS) {
        bool found = false;
        
        for (int p = 0; p < MAX_PROCESS; p++) {
            if (!finish[p]) {
                bool can_allocate = true;
                
                // need <= work 확인
                for (int r = 0; r < MAX_RESOURCE; r++) {
                    if (state->need[p][r] > work[r]) {
                        can_allocate = false;
                        break;
                    }
                }
                
                if (can_allocate) {
                    // 자원 반환 시뮬레이션
                    for (int r = 0; r < MAX_RESOURCE; r++) {
                        work[r] += state->allocation[p][r];
                    }
                    finish[p] = true;
                    safe_sequence[count++] = p;
                    found = true;
                }
            }
        }
        
        if (!found) {
            return false;  // 안전하지 않은 상태
        }
    }
    
    return true;  // 안전한 상태
}

bool request_resources(int process_id, int request[]) {
    banker_state_t temp_state = current_state;
    
    // 임시로 자원 할당
    for (int r = 0; r < MAX_RESOURCE; r++) {
        temp_state.available[r] -= request[r];
        temp_state.allocation[process_id][r] += request[r];
        temp_state.need[process_id][r] -= request[r];
    }
    
    // 안전 상태인지 확인
    if (is_safe_state(&temp_state)) {
        current_state = temp_state;  // 실제 할당
        return true;
    }
    
    return false;  // 요청 거부
}
```

#### 3. 탐지 및 회복 (Detection and Recovery)

##### 데드락 탐지 알고리즘
```c
typedef struct {
    int resource_count;
    int process_count;
    int allocation[MAX_PROCESS][MAX_RESOURCE];
    int request[MAX_PROCESS][MAX_RESOURCE];
    int available[MAX_RESOURCE];
} detection_state_t;

bool detect_deadlock(detection_state_t* state, int deadlocked_processes[]) {
    bool finish[MAX_PROCESS] = {false};
    int work[MAX_RESOURCE];
    int deadlock_count = 0;
    
    // available 자원 복사
    for (int r = 0; r < state->resource_count; r++) {
        work[r] = state->available[r];
    }
    
    // 자원을 할당받지 않은 프로세스는 완료로 표시
    for (int p = 0; p < state->process_count; p++) {
        bool has_allocation = false;
        for (int r = 0; r < state->resource_count; r++) {
            if (state->allocation[p][r] != 0) {
                has_allocation = true;
                break;
            }
        }
        if (!has_allocation) {
            finish[p] = true;
        }
    }
    
    bool progress = true;
    while (progress) {
        progress = false;
        
        for (int p = 0; p < state->process_count; p++) {
            if (!finish[p]) {
                bool can_satisfy = true;
                
                // request <= work 확인
                for (int r = 0; r < state->resource_count; r++) {
                    if (state->request[p][r] > work[r]) {
                        can_satisfy = false;
                        break;
                    }
                }
                
                if (can_satisfy) {
                    // 자원 회수
                    for (int r = 0; r < state->resource_count; r++) {
                        work[r] += state->allocation[p][r];
                    }
                    finish[p] = true;
                    progress = true;
                }
            }
        }
    }
    
    // 완료되지 않은 프로세스는 데드락 상태
    for (int p = 0; p < state->process_count; p++) {
        if (!finish[p]) {
            deadlocked_processes[deadlock_count++] = p;
        }
    }
    
    return deadlock_count > 0;
}
```

##### 데드락 회복 방법
```c
// 1. 프로세스 종료
void recover_by_process_termination(int deadlocked_processes[], int count) {
    // 방법 1: 모든 데드락 프로세스 종료
    for (int i = 0; i < count; i++) {
        terminate_process(deadlocked_processes[i]);
    }
    
    // 방법 2: 하나씩 종료하며 데드락 해소 확인
    for (int i = 0; i < count; i++) {
        terminate_process(deadlocked_processes[i]);
        if (!detect_deadlock(&current_state, NULL)) {
            break;  // 데드락 해소됨
        }
    }
}

// 2. 자원 선점
void recover_by_resource_preemption() {
    // 희생자 프로세스 선택 (비용 최소화)
    int victim = select_victim_process();
    
    // 자원 선점
    preempt_resources(victim);
    
    // 프로세스 롤백
    rollback_process(victim);
    
    // 기아 상태 방지를 위한 카운터 관리
    increment_preemption_count(victim);
}
```

### 실무에서의 데드락 예시

#### 1. 데이터베이스 데드락
```sql
-- 트랜잭션 1
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- 다른 트랜잭션이 실행되기를 기다림
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;

-- 트랜잭션 2 (동시 실행)
BEGIN TRANSACTION;  
UPDATE accounts SET balance = balance - 50 WHERE id = 2;
-- 트랜잭션 1이 id=1 락을 해제하기를 기다림
UPDATE accounts SET balance = balance + 50 WHERE id = 1;
COMMIT;

-- 해결책: 항상 같은 순서로 테이블/행에 접근
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = LEAST(1, 2);
UPDATE accounts SET balance = balance + 100 WHERE id = GREATEST(1, 2);
COMMIT;
```

#### 2. Java에서의 데드락 예시
```java
public class DeadlockExample {
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();
    
    // 데드락 발생 가능
    public void method1() {
        synchronized (lock1) {
            System.out.println("Method1: lock1 획득");
            
            try { Thread.sleep(100); } catch (InterruptedException e) {}
            
            synchronized (lock2) {  // 데드락 위험!
                System.out.println("Method1: lock2 획득");
            }
        }
    }
    
    public void method2() {
        synchronized (lock2) {
            System.out.println("Method2: lock2 획득");
            
            try { Thread.sleep(100); } catch (InterruptedException e) {}
            
            synchronized (lock1) {  // 데드락 위험!
                System.out.println("Method2: lock1 획득");
            }
        }
    }
    
    // 해결책: 락 순서 통일
    public void safeMethod1() {
        synchronized (lock1) {
            synchronized (lock2) {
                // 작업 수행
            }
        }
    }
    
    public void safeMethod2() {
        synchronized (lock1) {  // 동일한 순서
            synchronized (lock2) {
                // 작업 수행
            }
        }
    }
}
```

#### 3. 네트워크 소켓 데드락
```java
public class NetworkDeadlock {
    private final Object sendLock = new Object();
    private final Object receiveLock = new Object();
    
    // 데드락 위험
    public void sendAndReceive(Socket socket) {
        synchronized (sendLock) {
            // 데이터 전송
            synchronized (receiveLock) {
                // 응답 수신 대기 - 상대방도 같은 순서라면 데드락!
            }
        }
    }
    
    // 해결책: 타임아웃 설정
    public void safeNetworkOperation(Socket socket) throws IOException {
        socket.setSoTimeout(5000);  // 5초 타임아웃
        
        try {
            // 네트워크 작업 수행
        } catch (SocketTimeoutException e) {
            // 타임아웃 처리
            handleTimeout();
        }
    }
}
```

### 데드락 디버깅 도구

#### 1. Java 데드락 감지
```java
import java.lang.management.ManagementFactory;
import java.lang.management.ThreadMXBean;

public class DeadlockDetector {
    private final ThreadMXBean threadBean = 
        ManagementFactory.getThreadMXBean();
    
    public void detectDeadlock() {
        long[] deadlockedThreads = threadBean.findDeadlockedThreads();
        
        if (deadlockedThreads != null) {
            System.out.println("데드락 감지됨!");
            for (long threadId : deadlockedThreads) {
                System.out.println("데드락된 스레드 ID: " + threadId);
            }
        }
    }
}
```

#### 2. Linux 시스템 데드락 모니터링
```bash
# 프로세스 간 락 상태 확인
sudo lsof -p [PID]

# 스레드 상태 확인  
ps -eLf | grep [PROCESS_NAME]

# 시스템 락 정보
cat /proc/locks

# gdb를 이용한 데드락 분석
gdb -p [PID]
(gdb) info threads
(gdb) thread apply all bt
```

### 성능 고려사항

#### 락 경합 최소화
```c
// 잘못된 예: 큰 임계 구역
void bad_example() {
    pthread_mutex_lock(&mutex);
    
    expensive_computation();     // 비용이 큰 연산
    file_io_operation();        // I/O 작업
    network_communication();    // 네트워크 통신
    
    shared_data++;              // 실제 공유 데이터 접근
    
    pthread_mutex_unlock(&mutex);
}

// 개선된 예: 최소 임계 구역
void good_example() {
    // 임계 구역 밖에서 준비 작업
    int result = expensive_computation();
    char* data = prepare_data();
    
    // 최소한의 임계 구역
    pthread_mutex_lock(&mutex);
    shared_data += result;
    update_shared_structure(data);
    pthread_mutex_unlock(&mutex);
    
    // 후처리 작업
    cleanup_data(data);
}
```

#### 락-프리 자료구조 활용
```c
#include <stdatomic.h>

typedef struct node {
    atomic_intptr_t next;
    int data;
} node_t;

typedef struct {
    atomic_intptr_t head;
} lockfree_stack_t;

void push(lockfree_stack_t* stack, node_t* node) {
    node_t* head;
    do {
        head = (node_t*)atomic_load(&stack->head);
        atomic_store(&node->next, (intptr_t)head);
    } while (!atomic_compare_exchange_weak(&stack->head, 
                                         (intptr_t*)&head, 
                                         (intptr_t)node));
}
```

## 💡 면접 팁

### 면접관이 원하는 키워드
- **4가지 필요조건**, **상호배제**, **점유와대기**, **비선점**, **순환대기**
- **예방**, **회피**, **탐지**, **회복**
- **은행원알고리즘**, **자원할당순서**, **타임아웃**

### 꼬리 질문 대비
- "라이브락과 데드락의 차이점은?"
- "기아 상태(Starvation)와 데드락의 관계는?"
- "분산 시스템에서의 데드락은 어떻게 처리하나요?"
- "데이터베이스에서 데드락을 어떻게 감지하고 해결하나요?"

### 실무 경험 어필 포인트
- 멀티스레드 프로그래밍에서의 데드락 경험
- 데이터베이스 트랜잭션 최적화 경험  
- 시스템 성능 개선을 위한 락 최적화 경험

## ⚠️ 주의사항

### 일반적인 실수들
1. **락 순서 불일치**: 다른 스레드에서 다른 순서로 락 획득
2. **중첩 락**: 같은 스레드에서 동일한 뮤텍스를 중복 잠금
3. **예외 처리 누락**: 예외 발생 시 락이 해제되지 않음
4. **타임아웃 미설정**: 무한 대기 상황 방치

### 베스트 프랙티스
1. **일관된 락 순서 유지**
2. **최소 임계 구역 원칙**
3. **RAII 패턴 활용** (C++의 lock_guard 등)
4. **타임아웃과 재시도 로직 구현**
5. **데드락 감지 도구 활용**

## 🏷️ 태그
`운영체제` `동시성` `데드락` `상호배제` `동기화` `뮤텍스` `자원관리` `은행원알고리즘` `임계구역` `프로세스동기화`
