# JavaScript Event Loop

## 📌 핵심 요약
- JavaScript의 Event Loop는 비동기 작업을 처리하는 핵심 메커니즘
- 단일 스레드 언어인 JavaScript가 비동기 작업을 효율적으로 처리할 수 있게 해주는 시스템
- Call Stack, Web APIs, Callback Queue, Microtask Queue의 상호작용으로 구성
- 논블로킹(Non-blocking) 동작을 가능하게 하여 응답성 있는 애플리케이션 구현 지원

## 🎯 주요 개념

### 기본 정의
Event Loop는 JavaScript 런타임의 동시성 모델을 구현하는 메커니즘으로, 실행 컨텍스트 스택이 비어있을 때 태스크 큐에서 태스크를 가져와 실행하는 무한 루프입니다.

### 핵심 원리
1. **단일 스레드 실행**: JavaScript는 한 번에 하나의 작업만 처리
2. **비동기 처리**: Web APIs를 통해 시간이 걸리는 작업을 별도로 처리
3. **우선순위 기반 실행**: Microtask가 Macrotask보다 우선 실행

### 주요 특징
- 논블로킹 I/O 처리
- 이벤트 기반 프로그래밍 지원
- 동시성 처리 (실제 병렬 처리는 아님)
- 예측 가능한 실행 순서

## 📊 상세 내용

### 구성 요소

#### 1. Call Stack (호출 스택)
- 현재 실행 중인 함수들이 쌓이는 스택
- LIFO (Last In First Out) 구조
- 함수 호출 시 push, 완료 시 pop

#### 2. Web APIs (브라우저 API)
- setTimeout, setInterval, fetch 등
- DOM 이벤트, AJAX 요청 처리
- 비동기 작업을 별도 스레드에서 처리

#### 3. Callback Queue (Task Queue)
- Web APIs에서 완료된 콜백이 대기하는 큐
- FIFO (First In First Out) 구조
- setTimeout, setInterval 콜백 등이 여기에 추가

#### 4. Microtask Queue
- Promise, async/await, queueMicrotask() 등의 콜백이 대기
- Callback Queue보다 높은 우선순위
- 각 Macrotask 실행 후 모든 Microtask 처리

### 작동 원리

```
1. Call Stack에서 동기 코드 실행
2. 비동기 작업 발견 시 Web APIs로 위임
3. Web APIs에서 작업 완료 시 콜백을 Queue에 추가
4. Event Loop가 Call Stack이 비었는지 확인
5. 비어있다면 Queue에서 태스크를 가져와 실행
   - 먼저 모든 Microtask 처리
   - 그 다음 하나의 Macrotask 처리
6. 반복
```

### 실행 우선순위
1. **동기 코드** (Call Stack)
2. **Microtask** (Promise, queueMicrotask)
3. **Macrotask** (setTimeout, setInterval, I/O)
4. **requestAnimationFrame** (브라우저 리페인트 전)
5. **requestIdleCallback** (브라우저 유휴 시간)

## 💡 실제 활용

### 적용 사례

#### 비동기 데이터 로딩
```javascript
// API 호출 중에도 UI가 멈추지 않음
fetch('/api/data')
  .then(response => response.json())
  .then(data => updateUI(data));
```

#### 사용자 인터랙션 처리
```javascript
// 클릭 이벤트는 Event Loop를 통해 처리
button.addEventListener('click', handleClick);
```

#### 애니메이션 최적화
```javascript
// 브라우저 리페인트와 동기화
requestAnimationFrame(animate);
```

### 장단점 분석

**장점**
- 단순한 프로그래밍 모델
- 효율적인 리소스 사용
- 응답성 있는 UI 구현 가능
- 동시성 문제 회피 (race condition 최소화)

**단점**
- CPU 집약적 작업에 부적합
- 실제 병렬 처리 불가
- 콜백 지옥(Callback Hell) 가능성
- 복잡한 비동기 흐름 디버깅 어려움

### 모범 사례

#### 1. 긴 작업 분할
```javascript
// 나쁜 예: 메인 스레드 블로킹
for(let i = 0; i < 1000000; i++) {
  processItem(i);
}

// 좋은 예: 청크 단위로 분할
function processInChunks(items, chunkSize = 100) {
  let index = 0;
  
  function processChunk() {
    const end = Math.min(index + chunkSize, items.length);
    
    for(let i = index; i < end; i++) {
      processItem(items[i]);
    }
    
    index = end;
    
    if(index < items.length) {
      setTimeout(processChunk, 0);
    }
  }
  
  processChunk();
}
```

#### 2. Microtask vs Macrotask 이해
```javascript
// 실행 순서 예측
console.log('1');

setTimeout(() => console.log('2'), 0);

Promise.resolve().then(() => console.log('3'));

console.log('4');

// 출력: 1, 4, 3, 2
```

#### 3. 비동기 에러 처리
```javascript
// Promise 에러 처리
fetchData()
  .then(processData)
  .catch(handleError);

// async/await 에러 처리
async function loadData() {
  try {
    const data = await fetchData();
    return processData(data);
  } catch (error) {
    handleError(error);
  }
}
```

## 🔗 관련 주제

### 선행 지식
- JavaScript 기본 문법
- 함수와 스코프
- 콜백 함수 개념
- 동기/비동기 프로그래밍 기초

### 연관 개념
- Promise와 async/await
- Web Workers (진짜 멀티스레딩)
- Node.js Event Loop (libuv)
- 브라우저 렌더링 과정

### 심화 학습 경로
1. Promise 패턴 마스터
2. async/await 심화
3. RxJS와 반응형 프로그래밍
4. Web Workers와 Service Workers
5. Node.js의 Event Loop 차이점

## 📚 참고 자료

### 핵심 문헌
- MDN Web Docs - Event Loop
- JavaScript.info - Event Loop
- Philip Roberts - "What the heck is the event loop anyway?"

### 추천 리소스
- [Loupe](http://latentflip.com/loupe/) - Event Loop 시각화 도구
- Jake Archibald의 Event Loop 퀴즈
- V8 블로그의 Event Loop 관련 포스트

### 추가 학습 자료
- "You Don't Know JS" 시리즈 - Async & Performance
- Node.js 공식 문서 - Event Loop 가이드
- Chrome DevTools의 Performance 탭 활용법

## 🏷️ 태그
#JavaScript #EventLoop #비동기프로그래밍 #중급 #브라우저동작원리 #성능최적화

---

> 💡 **핵심 포인트**: Event Loop는 JavaScript의 비동기 처리를 가능하게 하는 핵심 메커니즘입니다. Call Stack이 비었을 때 Queue에서 태스크를 가져와 실행하는 단순한 원리이지만, 이를 잘 이해하면 효율적인 비동기 코드를 작성할 수 있습니다.

> ⚠️ **주의사항**: setTimeout(fn, 0)이 즉시 실행되지 않는다는 점을 기억하세요. 최소한 현재 실행 중인 코드와 모든 Microtask가 완료된 후에 실행됩니다.

> 📌 **기억할 점**: Microtask(Promise)는 항상 Macrotask(setTimeout)보다 먼저 실행됩니다. 이는 비동기 코드의 실행 순서를 예측할 때 매우 중요합니다.

> 🔍 **심화 학습**: Event Loop의 작동 원리를 완전히 이해하려면 실제로 다양한 비동기 패턴을 실험해보고, 브라우저 개발자 도구를 활용해 Performance를 분석해보는 것을 추천합니다.
