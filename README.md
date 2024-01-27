# 👶 ITEM 38 : any 타입은 가능한 한 좁은 범위에서만 사용하기

먼저, 함수와 관련된 any의 사용법을 살펴보겠습니다.

```ts
function processBar(b: Bar){/** */}
function f(){
  const x = expressionReturningFoo();
  processBar(x);
}
```

<img width="725" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/a056abcb-4cce-4cc7-907e-c9ab8af69ced"/>

문맥상으로 x라는 변수가 동시에 Foo타입과 Bar타입에 할당 가능하다면, 오류를 제거하는 방법은 두 가지 입니다.

```ts
// 방법 1 : 이렇게 하지 맙시다!
function f1(){
  const x : any= expressionReturningFoo();
  processBar(x);
}

// 방법 2 : 이게 낫습니다!
function f2(){
  const x = expressionReturningFoo();
  processBar(x as any);
}

```

두 가지 해결책 중에서 f1에서 사용된 `x: any`보다 f2에서 사용된 `x as any` 형태가 권장됩니다.
그 이유는 타입이 processBar 함수의 매개변수에만 사용된 표현식이므로 다른 코드에는 영향을 미치지 않기 때문입니다.
f1에서는 함수의 마지막까지 x타입이 any인 반면, f2는 processBar 호출 이후에 x가 그대로 Foo 타입이기 때문입니다.

만일 f1 함수가 x를 반환한다면 문제가 커집니다.
예를 들어 보겠습니다.

```ts
function f1(){
  const x : any= expressionReturningFoo();
  processBar(x);
  return x;
}
function g(){
  const foo = f1();   // any 타입
  foo.fooMethod();    // 이 함수 호출은 체크되지 않습니다.
}
```

g 함수 내에서 f1이 사용되므로 f1의 반환 타입인 any 타입이 foo의 타입에 영향을 미칩니다.
이렇게 함수에서 any를 반환하면 그 영향력은 프로젝트 전반에 전염병처럼 퍼지게 됩니다.
반면 any의 사용 범위를 좁게 제한하는 f2를 사용한다면 any 타입이 함수 바깥으로 영향을 미치지 않습니다.

비슷한 관점에서, 타입스크립트가 함수의 반환 타입을 추론할 수 있는 경우에도 함수의 반환타입을 명시하는 것이 좋습니다.
함수의 반환 타입을 명시하면 any 타입이 함수 바깥으로 영향을 미치는 것을 방지할 수 있습니다.
함수의 반환타입과 관련된 내용은 [아이템 19](https://github.com/Pyotato/effective_typescript/blob/item19/README.md)에서 자세히 다룹니다.

f1과 f2함수를 다시 한번 살펴보겠습니다.
f1은 오류를 제거하기 위해 x를 any타입으로 선언했습니다.
한편 f2는 오류를 제거하기 위해 x가 사용되는 곳에 `as any` 단언문을 사용습니다.
여기서 `@ts-ignore`를 사용하면 any를 사용하지 않고 오류를 제거할 수 있습니다.

```ts
function f1(){
  const x =  expressionReturningFoo();
  // @ts-ignore
  processBar(x);
  return x;
}
```

`@ts-ignore`를 사용한 다음 줄의 오류가 무시됩니다.
그러나 근본적인 원인을 해결한 것이 아니기 때문에 다른 곳에서 더 큰 문제가 발생할 수도 있습니다.
타입 체커가 알려주는 오류는 문제가 될 가능성이 높은 부분이므로 근본적인 원인을 찾아 적극적으로 대처하는 것이 바람직합니다.

이번에는 객체와 관련한 any의 사용법을 살펴보겠습니다.
어떤 큰 객체 안의 한 개 속성이 타입 오류를 가지는 상황을 예로 들어 보겠습니다.

```ts
const config: Config = {
  a: 1,
  b: 2, 
  c: [
    key: value
// ~~~~~ : 'foo' 속성이 'Foo' 타입에 필요하지만 'Bar' 타입에는 없습니다.
  }
}
```

단순히 생각하면 config 전체를 `as any`로 선언해서 오류를 제거할 수 있습니다.

```ts
const config: Config = {
  a: 1,
  b: 2, 
  c: [
    key: value
  }
} as any; // 이렇게 하지 맙시다!
```

객체 전체를 any로 단언하면 다른 속성들(a와 b) 역시 타입 체크가 되지 않는 부작용이 생깁니다.
그러므로 다음 코드처럼 최소한의 범위에만 any를 사용하는 것이 좋습니다.

```ts
const config: Config = {
  a: 1,
  b: 2, 
  c: [
    key: value as any;
  }
}
```
