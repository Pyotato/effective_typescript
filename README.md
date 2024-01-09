# 🤫 ITEM 40 :함수 안으로 단언문 감추기

함수를 작성하다 보면, 외부로 드러난 타입 정의는 간단하지만 내부 로직이 복잡해서 안전한 타입으로 구현하기 어려운 경우가 많습니다. 함수의 모든 부분을 안전한 타입으로 구현하는 것이 이상적이지만, 불필요한 예외 상황까지 고려해 가며 타입 정보를 힘들게 구성할 필요는 없습니다. **함수 내부에는 타입 단언을 사용하고 함수 외부로 드러나는 타입 정의를 정확히 명시하는 정도로 끝내는 것이 낫습니다.** 프로젝트 전반에 위험한 타입 단언문이 드러나 있는 것보다, 제대로 타입이 정의된 함수 안으로 타입 단언문을 감추는 것이 더 좋은 설계입니다.

예를 들어, 어떤 함수가 자신의 마지막 호출을 캐시하도록 만든다고 가정해 보겠습니다.
함수 캐싱은 리액트 같은 라이브러리에서 실행시간이 오래 걸리는 함수호출을 개선하는 일반적인 기법입니다. (리액트를 사용하고 있다면 실제로는 직접 구현하지 말고, 내장 함수 useMemo 훅을 사용해야합니다.) 어떤 함수든 캐싱할 수 있도록 래퍼 함수 cacheLast를 만들어 보겠습니다. cacheLast의 선언은 쉽게 만들 수 있습니다.

```ts
// 선언
// declare function cacheLast<T extends Function>(fn:T):T;

// 구현체
declare function shallowEqual(a: any, b: any): boolean;
function cacheLast<T extends Function>(fn: T): T {
  let lastArgs: any[] | null = null;
  let lastResult: any;
  return function (...args: any[]) {
    // 오류!
    if (!lastArgs || !shallowEqual(lastArgs, args)) {
      lastResult = fn(...args);
      lastArgs = args;
    }
    return lastResult;
  };
}
```

![ts playground](https://github.com/Pyotato/effective_typescript/assets/102423086/81be63b3-a351-431a-9d09-af6690d30ae5)

타입스크립트는 **반환문에 있는 함수**와 **원본 함수 T타입**이 어떤 관련이 있는지 알지 못하기 떄문에 오류가 발생했습니다.
그러나 결과적으로 원본 함수 T타입과 동일한 매개변수로 호출되고 반환값 역시 예상한 결과가 되기 떄문에, 타입 단언문을 추가해서 오류를 제거하는 것이 큰 문제가 되지는 않습니다.

```ts
declare function shallowEqual(a: any, b: any): boolean;
function cacheLast<T extends Function>(fn: T): T {
  let lastArgs: any[] | null = null;
  let lastResult: any;
  return function (...args: any[]) {
    if (!lastArgs || !shallowEqual(lastArgs, args)) {
      lastResult = fn(...args);
      lastAr밖에 없습니다.

```ts
declare function shallowEqual(a: any, b: any): boolean;
function shallowObjectEqual<T extends object>(a: T, b: T): boolean {
  for (const [k, aVal] of Object.entries(a)) {
    if (!(k in b) || aVal !== (b as any)[k]) {
      return false;
    }
  }
  return Object.keys(a).length === Object.keys(b).length;
}
```

b as any 타입 단언은 안전하며 (k in b를 체크했으므로), 결국 정확한 타입으로 정의되고 제대로 구현한 함수가 됩니다.
객체가 같은지 체크하기 위해 객체 순회와 단언문이 코드에 직접들어가는 것보다, 앞의 코드처럼 별도의 함수로 분리해 내는 것이 훨씬 좋은 설계입니다.
