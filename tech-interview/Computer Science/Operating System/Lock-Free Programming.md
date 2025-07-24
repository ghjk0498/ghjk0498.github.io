# 락 프리(Lock-Free) 프로그래밍

## ❓ 면접 질문
"락 프리(Lock-Free) 프로그래밍이 무엇인지 설명하고, 일반적인 락 기반 동기화와 어떤 차이점이 있는지 설명해주세요."

## ✅ 핵심 답변

### 개념 정리
락 프리 프로그래밍은 **뮤텍스나 세마포어 같은 락(Lock)을 사용하지 않고도 멀티스레드 환경에서 데이터의 일관성을 보장하는 프로그래밍 기법**입니다. 주로 **원자적 연산(Atomic Operations)**과 **Compare-and-Swap(CAS)** 같은 하드웨어 지원 명령어를 활용합니다.

### 상세 설명

#### 락 기반 vs 락 프리 방식

| 구분 | 락 기반(Lock-Based) | 락 프리(Lock-Free) |
|------|-------------------|-------------------|
| 동기화 방법 | 뮤텍스, 세마포어 등 | 원자적 연산, CAS |
| 블로킹 여부 | 블로킹 발생 가능 | 논블로킹 |
| 데드락 위험 | 있음 | 없음 |
| 성능 | 락 경합 시 성능 저하 | 높은 동시성 |
| 구현 복잡도 | 상대적으로 단순 | 복잡함 |

#### 락 프리의 핵심 메커니즘

**1. Compare-and-Swap (CAS)**
```c
// CAS 의사코드
bool compare_and_swap(int* ptr, int expected, int new_value) {
    if (*ptr == expected) {
        *ptr = new_value;
        return true;
    }
    return false;
}
```

**2. ABA 문제와 해결**
```text
Thread A: 값 읽기 (A) → 다른 작업 수행
Thread B: A → B → A로 변경
Thread A: CAS 수행 (A로 보이지만 실제로는 변경됨)
```

### 면접 답변 템플릿
"락 프리 프로그래밍은 전통적인 락 메커니즘 없이 멀티스레드 환경에서 데이터 일관성을 보장하는 기법입니다. 

주요 차이점은 세 가지입니다. 첫째, **블로킹이 발생하지 않아** 스레드가 대기하지 않습니다. 둘째, **데드락이 원천적으로 불가능**합니다. 셋째, **높은 동시성**을 제공하여 성능상 이점이 있습니다.

핵심 메커니즘은 Compare-and-Swap 같은 원자적 연산을 활용하는 것이며, 이를 통해 여러 스레드가 동시에 접근해도 데이터 무결성을 보장할 수 있습니다."

## 📚 참고할만한 정보

### 심화 개념

#### 락 프리 자료구조 예시

**1. 락 프리 스택 (Treiber Stack)**
```c
typedef struct Node {
    int data;
    struct Node* next;
} Node;

typedef struct {
    Node* top;
} LockFreeStack;

void push(LockFreeStack* stack, int data) {
    Node* newNode = malloc(sizeof(Node));
    newNode->data = data;
    
    Node* oldTop;
    do {
        oldTop = stack->top;
        newNode->next = oldTop;
    } while (!CAS(&stack->top, oldTop, newNode));
}

int pop(LockFreeStack* stack) {
    Node* oldTop;
    Node* newTop;
    
    do {
        oldTop = stack->top;
        if (oldTop == NULL) return -1; // 스택이 비어있음
        newTop = oldTop->next;
    } while (!CAS(&stack->top, oldTop, newTop));
    
    int data = oldTop->data;
    free(oldTop);
    return data;
}
```

**2. 락 프리 큐 (Michael & Scott Queue)**
```c
typedef struct QNode {
    int data;
    struct QNode* next;
} QNode;

typedef struct {
    QNode* head;
    QNode* tail;
} LockFreeQueue;

void enqueue(LockFreeQueue* queue, int data) {
    QNode* newNode = malloc(sizeof(QNode));
    newNode->data = data;
    newNode->next = NULL;
    
    QNode* tail;
    QNode* next;
    
    while (true) {
        tail = queue->tail;
        next = tail->next;
        
        if (tail == queue->tail) { // tail이 변경되지 않았는지 확인
            if (next == NULL) {
                if (CAS(&tail->next, NULL, newNode)) {
                    break; // 성공적으로 노드 추가
                }
            } else {
                CAS(&queue->tail, tail, next); // tail 포인터 이동
            }
        }
    }
    CAS(&queue->tail, tail, newNode); // tail 포인터 업데이트
}
```

#### 메모리 순서 (Memory Ordering)

**메모리 배리어의 종류**
- **Acquire**: 읽기 연산 이후의 메모리 접근이 재배열되지 않음
- **Release**: 쓰기 연산 이전의 메모리 접근이 재배열되지 않음
- **Sequential Consistency**: 모든 연산이 순차적으로 실행되는 것처럼 보임

```cpp
// C++11 atomic 사용 예시
#include <atomic>

std::atomic<int> counter{0};

// memory_order_relaxed: 순서 보장 없음
counter.fetch_add(1, std::memory_order_relaxed);

// memory_order_acquire/release: 동기화 보장
counter.store(42, std::memory_order_release);
int value = counter.load(std::memory_order_acquire);
```

### 실무 예시

#### Java의 락 프리 구현
```java
// java.util.concurrent.atomic 패키지
AtomicInteger counter = new AtomicInteger(0);

// 락 프리 증가
int oldValue, newValue;
do {
    oldValue = counter.get();
    newValue = oldValue + 1;
} while (!counter.compareAndSet(oldValue, newValue));

// 또는 간단하게
counter.incrementAndGet();
```

#### 실제 활용 사례
- **데이터베이스**: 인덱스 구조, 트랜잭션 로그
- **웹 서버**: 연결 풀, 요청 큐
- **게임 엔진**: 물리 시뮬레이션, 렌더링 파이프라인
- **금융 시스템**: 고빈도 거래, 시장 데이터 처리

### 관련 기술

#### Wait-Free vs Lock-Free
- **Lock-Free**: 시스템 전체적으로 진행이 보장됨
- **Wait-Free**: 모든 스레드의 진행이 보장됨 (더 강한 조건)

#### Hazard Pointer
```c
// 메모리 해제 문제 해결을 위한 Hazard Pointer
typedef struct HazardPointer {
    void* pointer;
    struct HazardPointer* next;
} HazardPointer;

void protect_pointer(void* ptr) {
    HazardPointer* hp = get_hazard_pointer();
    hp->pointer = ptr;
}

void safe_delete(void* ptr) {
    add_to_retire_list(ptr);
    if (retire_list_size() > THRESHOLD) {
        scan_and_cleanup();
    }
}
```

## 💡 면접 팁

### 자주 나오는 꼬리 질문들
1. **"ABA 문제가 무엇인가요?"**
   - 포인터 값은 같지만 실제 객체가 변경된 상황
   - Tagged Pointer나 Hazard Pointer로 해결

2. **"락 프리가 항상 더 빠른가요?"**
   - 경합이 적을 때는 락이 더 효율적일 수 있음
   - 구현 복잡도와 메모리 사용량 고려 필요

3. **"메모리 재사용 문제는 어떻게 해결하나요?"**
   - Hazard Pointer, RCU(Read-Copy-Update) 등의 기법 활용

### ⚠️ 주의사항
- 락 프리 ≠ 락 사용 금지 (필요시 락 사용 가능)
- 모든 상황에서 락 프리가 최선은 아님
- 정확성 검증이 매우 어려움
- 플랫폼별 원자적 연산 지원 차이 고려

## 🏷️ 태그
`동시성` `멀티스레딩` `원자적연산` `CAS` `논블로킹` `성능최적화` `자료구조` `메모리모델`
