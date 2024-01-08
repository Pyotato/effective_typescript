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
      lastArgs = args;
    }
    return lastResult;
  } as unknown as T;
}
```

실제로 함수를 실행해 보면 잘 동작합니다.
함수 내부에는 any가 꽤 많아 보이지만 타입 정의에는 any가 없기 때문에, cacheLast를 호출하는 쪽에서는 any가 사용됐는지 알지 못합니다.

> 📌 앞의 코드에는 사실 두 가지 문제가 있습니다. **함수를 연속으로 호출하는 경우에 this의 값이 동일한지 체크하지 않습니다.**
>
> 또한 **원본 함수가 객체처럼 속성 값을 가지고 있었다면 래퍼 함수에는 속성값이 없기 떄문에 타입이 달라집니다.**
>
> 그러나 일부 예외적인 상황을 제외한다면 cacheLast 함수는 괜찮게 구현되었습니다.
>
> 여기서는 예를 들기 위해 간단한 구현체를 보인것이고, 실제로는 안전한 타입으로 cacheLast를 구현할 수 있습니다.

한편, 앞 예제에 나온 shallowEqual의 두 개의 배열을 매개변수로 받아서 비교하는 함수이며 타입 정의와 구현이 간단합니다.
그러나 **객체를 매개변수**로 하는 shallowObjectEqual은 타입 정의는 간단하지만 구현이 조금 복잡한니다.

먼저 shallowObjectEqual의 타입 정의를 보겠습니다.

```ts
declare function shallowObjectEqual<T extends object>(a: any, b: any): boolean;
```

객체 매개변수 a와 b가 동일한 키를 가진다는 보장이 없기 때문에 구현할 때는 주의해야합니다. ([아이템 54](https://github.com/Pyotato/effective_typescript/tree/item54))

```ts
declare function shallowEqual(a: any, b: any): boolean;
function shallowObjectEqual<T extends object>(a: T, b: T): boolean {
  for (const [k, aVal] of Object.entries(a)) {
    if (!(k in b) || aVal !== b[k]) {
      // 오류!
      return false;
    }
  }
  return Object.keys(a).length === Object.keys(b).length;
}
```

![image](https://github.com/Pyotato/effective_typescript/assets/102423086/5066cd9e-7e55-4050-88be-17b8d235c239)

if구문의 k in b 체크로 b객체에 k속성이 있다는 것을 확인했지만 b[k]부분에서 오류가 발생하는 것이 이상합니다. (타입스크립틔 문맥 활용 능력이 부족한 것으로 보입니다.)
어쨋든 실제 오류가 아니라는 것을 알고 있기 때문에 any로 단언할 수 박ㅆ에 없습니다.

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
