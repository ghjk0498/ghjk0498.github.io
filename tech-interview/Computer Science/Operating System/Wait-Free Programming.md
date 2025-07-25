# Wait-Free 프로그래밍

## ❓ 면접 질문
"Wait-Free 프로그래밍이 무엇인지 설명하고, Lock-Free와 어떤 차이점이 있는지 설명해주세요."

## ✅ 핵심 답변

### 개념 정리
Wait-Free 프로그래밍은 **모든 스레드가 유한한 단계 내에서 작업을 완료할 수 있음을 보장하는 최강의 논블로킹 프로그래밍 기법**입니다. 어떤 스레드도 다른 스레드의 상태나 속도에 관계없이 독립적으로 진행할 수 있습니다.

### 상세 설명

#### 동시성 프로그래밍 단계별 비교

```text
강도 순서: Wait-Free > Lock-Free > Obstruction-Free > Blocking

┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│ Wait-Free   │  │ Lock-Free   │  │Obstruction- │  │  Blocking   │
│             │  │             │  │    Free     │  │             │
│모든 스레드가 │  │시스템 전체가 │  │격리된 상태에 │  │락으로 인한   │
│진행 보장     │  │진행 보장    │  │서만 진행     │  │대기 발생     │
└─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘
```

#### Wait-Free vs Lock-Free 차이점

| 구분 | Wait-Free | Lock-Free |
|------|-----------|-----------|
| **진행 보장** | 모든 스레드가 개별적으로 진행 | 시스템 전체적으로만 진행 보장 |
| **완료 시간** | 유한한 단계 내 완료 보장 | 일부 스레드는 무한정 지연 가능 |
| **기아 현상** | 없음 | 발생 가능 |
| **구현 복잡도** | 매우 높음 | 높음 |
| **성능 오버헤드** | 높음 | 중간 |
| **실용성** | 제한적 | 널리 사용됨 |

#### Wait-Free의 핵심 특성

**1. 개별적 진행 보장 (Individual Progress)**
```text
Wait-Free 시나리오:
Thread A: 단계1 → 단계2 → 완료 (3단계 내 보장)
Thread B: 단계1 → 단계2 → 완료 (3단계 내 보장)  
Thread C: 단계1 → 단계2 → 완료 (3단계 내 보장)

Lock-Free 시나리오:
Thread A: 단계1 → 단계2 → 완료
Thread B: 단계1 → 재시도 → 재시도 → ... (무한정 지연 가능)
Thread C: 단계1 → 단계2 → 완료
```

**2. 도움 메커니즘 (Helping Mechanism)**
느린 스레드를 빠른 스레드가 도와주는 방식으로 구현

### 면접 답변 템플릿
"Wait-Free는 동시성 프로그래밍에서 가장 강력한 보장을 제공하는 기법입니다.

Lock-Free와의 핵심 차이점은 **진행 보장의 범위**입니다. Lock-Free는 시스템 전체적으로는 진행이 보장되지만 개별 스레드는 무한정 지연될 수 있는 반면, Wait-Free는 **모든 스레드가 유한한 단계 내에서 작업을 완료**할 수 있습니다.

이는 '도움 메커니즘'을 통해 구현되는데, 빠른 스레드가 느린 스레드의 작업을 대신 완료해주는 방식입니다. 따라서 기아 현상이 원천적으로 불가능하며, 실시간 시스템에서 예측 가능한 성능을 보장할 수 있습니다."

## 📚 참고할만한 정보

### 심화 개념

#### Wait-Free 구현 패턴

**1. 단일 연산 Wait-Free (Trivial Wait-Free)**
```c
// 원자적 연산 하나로 완료되는 경우
typedef struct {
    atomic_int value;
} WaitFreeCounter;

int increment(WaitFreeCounter* counter) {
    // 단일 원자적 연산으로 완료 - Wait-Free!
    return atomic_fetch_add(&counter->value, 1);
}
```

**2. 도움 기반 Wait-Free (Help-Based Wait-Free)**
```c
typedef struct Operation {
    enum { PUSH, POP } type;
    int value;
    atomic_bool completed;
    struct Operation* next;
} Operation;

typedef struct {
    atomic_ptr top;
    atomic_ptr pending_ops; // 도움이 필요한 연산들
} WaitFreeStack;

void push(WaitFreeStack* stack, int value) {
    Operation* op = create_operation(PUSH, value);
    
    // 1단계: 연산을 pending 리스트에 추가
    add_to_pending(stack, op);
    
    // 2단계: 다른 스레드들의 pending 연산들을 도움
    help_others(stack);
    
    // 3단계: 자신의 연산이 완료될 때까지 도움 제공
    while (!op->completed) {
        help_others(stack);
    }
}

void help_others(WaitFreeStack* stack) {
    Operation* op = get_oldest_pending(stack);
    if (op != NULL) {
        execute_operation(stack, op);
        op->completed = true;
    }
}
```

**3. 범용 Wait-Free 변환 (Universal Construction)**
```c
// Herlihy의 범용 구조체 개념
typedef struct Node {
    State state;           // 현재 상태
    Operation* operation;  // 적용할 연산
    int sequence_number;   // 순서 번호
    struct Node* next;
} Node;

typedef struct {
    atomic_ptr head;       // 현재 상태를 가리키는 포인터
    atomic_int counter;    // 전역 카운터
} UniversalObject;

Result apply_operation(UniversalObject* obj, Operation* op) {
    int seq_num = atomic_fetch_add(&obj->counter, 1);
    Node* new_node = create_node(op, seq_num);
    
    Node* current;
    do {
        current = atomic_load(&obj->head);
        State new_state = apply_op_to_state(current->state, op);
        new_node->state = new_state;
    } while (!atomic_compare_exchange(&obj->head, current, new_node));
    
    return extract_result(new_node->state, op);
}
```

#### Wait-Free 알고리즘 예시

**1. Wait-Free 큐 (Kogan & Petrank Algorithm)**
```c
typedef struct QNode {
    atomic_ptr data;
    atomic_ptr next;
    atomic_int enq_tid;  // enqueue한 스레드 ID
    atomic_int deq_tid;  // dequeue한 스레드 ID
} QNode;

typedef struct {
    atomic_ptr head;
    atomic_ptr tail;
    State* state[MAX_THREADS]; // 각 스레드의 상태
} WaitFreeQueue;

void enqueue(WaitFreeQueue* queue, void* data, int tid) {
    State* my_state = &queue->state[tid];
    
    // Phase 1: 연산 공표
    my_state->operation = ENQUEUE;
    my_state->arg = data;
    my_state->phase = 1;
    
    // Phase 2: 다른 스레드들 도움
    help_enqueue(queue, tid);
    
    // Phase 3: 완료 대기
    while (my_state->phase != 3) {
        help_all(queue);
    }
}

void help_enqueue(WaitFreeQueue* queue, int tid) {
    State* state = &queue->state[tid];
    if (state->operation != ENQUEUE || state->phase != 1) 
        return;
    
    QNode* new_node = create_node(state->arg);
    QNode* tail = atomic_load(&queue->tail);
    
    if (atomic_compare_exchange(&tail->next, NULL, new_node)) {
        atomic_compare_exchange(&queue->tail, tail, new_node);
        state->phase = 3; // 완료 표시
    }
}
```

**2. Wait-Free 해시 테이블**
```c
typedef struct Bucket {
    atomic_ptr key;
    atomic_ptr value;
    atomic_int version;
} Bucket;

typedef struct {
    Bucket* buckets;
    int size;
    OpDesc* pending[MAX_THREADS];
} WaitFreeHashTable;

bool insert(WaitFreeHashTable* table, void* key, void* value, int tid) {
    OpDesc* op = create_op_desc(INSERT, key, value);
    table->pending[tid] = op;
    
    // 다른 스레드들 도움
    for (int i = 0; i < MAX_THREADS; i++) {
        help_insert(table, i);
    }
    
    // 자신의 연산 완료 확인
    return op->completed;
}
```

### 실무 예시

#### Wait-Free의 실제 활용 분야

**1. 실시간 시스템**
```c
// 항공기 제어 시스템에서의 센서 데이터 읽기
typedef struct {
    atomic_long timestamp;
    atomic_int altitude;
    atomic_int speed;
    atomic_int heading;
} FlightData;

FlightData* get_flight_data(FlightDataSystem* system) {
    // Wait-Free로 구현되어 어떤 스레드도 블록되지 않음
    // 실시간 제약조건 만족 보장
    return atomic_load(&system->current_data);
}
```

**2. 고빈도 거래 시스템**
```c
// 주문 처리에서 Wait-Free 보장
typedef struct {
    atomic_ptr order_book;
    WaitFreeQueue* incoming_orders;
} TradingSystem;

bool place_order(TradingSystem* system, Order* order) {
    // 모든 거래자가 공평하게 처리됨
    // 기아 현상 없이 유한 시간 내 완료 보장
    return wait_free_enqueue(system->incoming_orders, order);
}
```

**3. 메모리 할당자**
```java
// Java의 일부 가비지 컬렉터에서 사용
public class WaitFreeAllocator {
    private final AtomicReference<Block>[] freelists;
    
    public Object allocate(int size) {
        int tid = Thread.currentThread().getId();
        // 각 스레드가 독립적으로 할당 가능
        // 다른 스레드의 영향을 받지 않음
        return allocateFromPrivateList(tid, size);
    }
}
```

### 관련 기술

#### Wait-Free 설계 원칙

**1. 도움 메커니즘 (Helping Mechanism)**
- 빠른 스레드가 느린 스레드를 도움
- 연산의 멱등성(Idempotency) 보장 필요
- 상태 기반 협력 구조

**2. 연산 선형화 (Operation Linearization)**
```c
// 연산의 효과가 원자적으로 보이는 지점 정의
typedef struct {
    atomic_long linearization_point;
    OperationResult result;
} LinearizableOp;
```

**3. 메모리 관리**
```c
// Hazard Pointer와 결합한 안전한 메모리 해제
typedef struct {
    atomic_ptr pointer;
    atomic_int reference_count;
    HazardPointer* hazards[MAX_THREADS];
} SafePointer;
```

#### 성능 분석

**시간 복잡도 분석**
```text
Wait-Free 연산의 시간 복잡도: O(1) 또는 O(P) (P = 프로세서 수)
- 개별 스레드: 최대 k단계 내 완료 (k는 상수)
- 전체 시스템: 모든 스레드가 동시에 진행

Lock-Free 연산의 시간 복잡도: 
- 개별 스레드: O(∞) (기아 현상 가능)
- 전체 시스템: O(1) 평균
```

**공간 복잡도**
- 각 스레드별 상태 저장 필요: O(P)
- 도움 메커니즘을 위한 추가 메모리: O(P × k)

## 💡 면접 팁

### 자주 나오는 꼬리 질문들

1. **"Wait-Free가 항상 Lock-Free보다 좋은가요?"**
   - 구현 복잡도가 매우 높음
   - 메모리 오버헤드가 큼 (스레드 수에 비례)
   - 실제 성능은 경합 수준에 따라 다름

2. **"도움 메커니즘은 어떻게 동작하나요?"**
   - 빠른 스레드가 느린 스레드의 연산을 대신 수행
   - 멱등성 보장이 핵심 (같은 연산을 여러 번 해도 결과 동일)
   - 상태 기반으로 누가 도움이 필요한지 판단

3. **"범용 Wait-Free 구조체란 무엇인가요?"**
   - 임의의 순차 자료구조를 Wait-Free로 변환하는 일반적 방법
   - Herlihy의 범용 구조체 (Universal Construction)
   - 상태 기계와 합의 알고리즘 활용

4. **"실제로 어디에 사용되나요?"**
   - 실시간 시스템 (항공기, 의료기기)
   - 고빈도 거래 시스템
   - 일부 가비지 컬렉터
   - 커널 레벨 동기화

### ⚠️ 주의사항

**구현시 고려사항**
- **ABA 문제**: Tagged pointer나 epoch-based 해결책 필요
- **메모리 순서**: 적절한 memory barrier 사용
- **스케일링**: 스레드 수 증가시 성능 저하 가능
- **디버깅**: 매우 복잡한 상호작용으로 디버깅 어려움

**성능 특성**
- 낮은 경합: Lock-Free가 더 효율적일 수 있음
- 높은 경합: Wait-Free의 장점 극대화
- 메모리 사용량: 스레드 수에 비례하여 증가

**적용 기준**
- 실시간 보장이 절대적으로 필요한 경우
- 기아 현상이 치명적인 시스템
- 높은 구현/유지보수 비용 감수 가능한 경우

## 🔗 관련 문서 링크
- [락 프리(Lock-Free) 프로그래밍](./락프리_프로그래밍.md)
- [원자적 연산과 메모리 모델](./원자적연산_메모리모델.md)
- [합의 알고리즘과 FLP 불가능성](./합의알고리즘_FLP정리.md)

## 🏷️ 태그
`동시성` `논블로킹` `실시간시스템` `도움메커니즘` `범용구조체` `선형화가능성` `합의알고리즘` `성능최적화`
