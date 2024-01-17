# 💗 ITEM 21 : 타입 넓히기

[아이템 7](https://github.com/Pyotato/effective_typescript/blob/item7/README.md)에서 설명한 것처럼 런타임에 모든 변수는 유일한 값을 가집니다.
그러나 타입스크립트가 작성된 코드를 체크하는 정적 분석 시점에, 변수는 '가능한' 값들의 집합인 타입을 가집니다.
상수를 사용해서 변수를 초기화할 때 타입을 명시하지 않으면 타입 체커는 타입을 결정해야 합니다. 
이 말은 지정된 탄일 값을 갖고 할당 가능한 값들의 집합을 유추해야 한다는 뜻입니다. 
타입스크립트에서는 이러한 과정을 '넓히기(widening)'이라고 부릅니다.
넓히기의 과정을 이해한다면 오류의 원인을 파악하고 타입 구문을 더 효과적으로 사용할 수 있을 겁니다.

벡터를 다루는 라이브러리를 작성한다고 가정해 보겠습니다. 3D 벡터에 대한 타입과 그 요소들의 값을 얻는 함수를 작성합니다.

```ts
interface Vector3{x: number, y : number, z: number;}
function getComponent(vector: Vector3, axis: 'x'| 'y' | 'z'){
  return vector[axis];
}
```

Vector3 함수를 사용한 다음 코드는 런타임에 오류 없이 실행되지만, 편집기에서는 오류가 표시됩니다.

```ts
let x = 'x';
let vec = {x: 10, y: 20, z: 30};
getComponent(vec,x);
```

<img width="1234" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/edcf6985-316b-4425-b2cc-6708c33b510e"/>

실행은 잘 되지만 편집기에서는 오류가 발생합니다.

getComponent 함수는 두 번째 매개변수에 `'x'| 'y' | 'z'` 타입을 기대했지만, x의 탑은 할당 시점에 넓히기가 동작해서 string으로 추론되었습니다.
string 타입은 `'x'| 'y' | 'z'`타입에 할당이 불가능하므로, 오류가 된 것입니다.

타입 넓히기가 진행될 때, 주어진 값으로 추론 가능한 타입이 여러 개이기 때문에 과정이 상당히 모호합니다. 다음 코드를 예로 들어 보곘습니다.

```ts
const mixed = ['x', 1];
```

<p id="mixed">mixed의 타입이 어떻게 추론되는지 살펴보겠습니다. 다음은 mixed의 타입이 될 수 있는 후보들입니다. 후보가 상당히 많은 걸 알 수 있습니다.</p>

- `('x' | 1 )[]`
- `['x' | 1 ]`
- `[string, number]`
- `readonly [string, number]`
- `(string | number)[]`
- `readonly (string, number)[]`
- `[any, any]`
- `any[]`

정보가 충분하지 않다면 mixed가 어떤 타입으로 추론되어야 하는지 알 수 없습니다.
그러므로 타입스크립트는 작성자의 의도를 추측합니다. (이 경우에는 `(string | number)[]`으로 추측됩니다.)
그러나 타입스크립트가 아무리 영리하더라도 사람의 마음까지 읽을 수는 없고 따라서 추측한 답이 항상 옳을 수도 없습니다.

처음의 예제에서 타입스크립트는 다음 예제와 같은 코드를 예상했기 때문에 x의 타입을 string으로 추론했습니다.

```ts
let x = 'x';
x = 'a';
x = 'Four score and seven years ago';
```

자바스크립트에서는 다음처럼 작성해도 유효합니다.

```ts
let x = 'x';
x = /x|y|z/;
x = ['x', 'y', 'z'];
```
타입스크립트는 x의 타입을 string으로 추론할 때, 명확성과 유연성 사이의 균형을 유지하려고 합니다.
일반적인 규칙은 변수가 선언된 후로는 타입이 바뀌지 않아야 하므로 ([이이템 20](https://github.com/Pyotato/effective_typescript/blob/item20/README.md)), `string|RegExp`나 `string|string[]`이나 any보다는 string을 사용하는 게 낫습니다.

타입스크립트는 넓히기의 과정을 제어할 수 있도록 몇 가지 방법을 제공합니다. 
넓히기 과정을 제어할 수 있는 첫번째 방법은 **const**입니다. 만약 let 대신 const로 변수를 선언하면 더 좁은 타입이 됩니다.
실제로 const를 사용하면 앞에서 발생한 오류가 해결됩니다.

```ts
const x = 'x';
let vec = {x: 10, y: 20, z: 30};
getComponent(vec,x);
```

<img width="457" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/292e6e2b-dd87-47ac-9c89-ec726d640278"/>

이제는 x는 재할당될 수 없으므로 타입스크릡트느 의심의 여지 없이 더 좁은 타입인 ('x')으로 추론할 수 있습니다.
그리고 문자 리터럴 타입 'x'는 `'x'| 'y' | 'z'에 할당 가능하므로 코드가 타입 체커를 통과합니다.

그러나 const는 만능이 아닙니다. 
객체와 배열의 경우에는 여전히 문제가 있습니다. 아이템 초반에 있는 [mixed예제](https://github.com/Pyotato/effective_typescript/blob/item21/README.md#mixed)는 배열에 대한 문제를 보여 줍니다.
튜풀 타입을 추론해야할지, 요소들은 어떤 타입으로 추론해야 할지 알 수 없습니다. 비슷한 문제가 객체에서도 발생합니다.
다음 코드는 자바스크립트에서 정상입니다.

```js
const v = {
  x: 1,
}
v.x = 3;
v.x = '3';
v.y = 4;
v.name = 'Pythagoras';
```

v의 타입은 구체적인 정도에 따라 다양한 모습으로 추론될 수 있습니다. 가장 구체적인 경우라면 `readonly { x: 1}`입니다.
조금 추상적으로는 `{x: number}`입니다. 가장 추상적이라면 `{[key: string]: number}` 또는 object가 될 것입니다.
객체의 경우 타입스크립트의 넓히기 알고리즘은 각 요소를 let으로 할당된 것처럼 다룹니다.
그래서 v의 타입은 `{x: number}`가 됩니다. 덕분에 v.x를 다른 숫자로 재할당할 수 있게 되지만 string으로는 안됩니다.
그리고 다른 속성을 추가하지도 못합니다. (따라서 객체를 한번에 만들어야 합니다. 자세한 내용은 [아이템 23](https://github.com/Pyotato/effective_typescript/blob/item23/README.md)에서 다룹니다.) 

따라서 다음 코드는 마지막 세 문장에서 오류가 발생합니다.

```ts
const v = {
  x: 1,
}
v.x = 3;
v.x = '3';
v.y = 4;
v.name = 'Pythagoras';
```

<img width="1022" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/6c126660-9f0e-4630-915c-0cd6225daa32"/>

앞에서 언급했듯이 타입스크립트는 명확성과 유연성 사이의 균형을 유지하려고 합니다.
오류를 잡기 위해서는 충분히 구체적으로 타입을 추론해야하지만, 잘못된 추론 (false positive)을 할 정도로 구체적으로 수행하지는 않습니다.
예를 들어, 1과 같은 값으로 초기화되는 속성을 적당히 number의 타입으로 추론합니다.

타입 추론의 가도를 직접 제어하려면 타입스크립트의 기본 동작을 재정의해야 합니다. 타입스크립트의 기본 동작을 재정의하는 세 가지 방법이 있습니다.

**첫번째, 명시적 타입 구문을 제공하는 것입니다.**

```ts
const v : {x: 1 | 3| 5} = {
  x: 1,
};
```

<img width="211" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/70572909-566a-41c4-b8cf-d1718d5beecc"/>

**두번째, 타입 체커에 추가적인 문맥을 제공하는 것입니다.** (예를 들어, 함수의 매개변수로 값을 전달). [아이템 26](https://github.com/Pyotato/effective_typescript/blob/item26/README.md)은 타입 추론 과정에서 문맥의 역할에 대한 자세한 내용응 다룹니다.

**세번째, const 단언문을 사용하는 것입니다.** const단언문과 변수 선언에 쓰이는 let이나 const와 혼동해서는 안됩니다. const 단언문은 온전히 타입 공간의 기법입니다. 
다음 예제를 통해 각 변수에 추론된 타입의 차이점을 살펴보겠습니다.

```ts
const v1  = {
  x: 1,
  y: 2,
};

const v2  = {
  x: 1 as const,
  y: 2,
};

const v3  = {
  x: 1 ,
  y: 2,
} as const;
```

<img width="211" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/b8882215-cbef-4502-b111-6763d81d3906"/>
<img width="211" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/6afab255-f3d7-446b-8be3-28e25b844837"/>
<img width="211" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/e6dfc0b0-6b43-4c0b-bebf-429705c79bd2"/>

값 뒤에 as const를 작성하면, 타입스크립트는 최대한 좁은 타입으로 추론합니다.
v3에는 넓히기가 동작하지 않았습니다. v3이 진짜 상수라면, 주석에 보이는 추론된 타입이 실제로 원하는 형태일 것입니다. 또한 배열을 튜플 타입으으로 추론할 때에도 as const를 사용할 수 있습니다.

```ts
const a1  = [1, 2, 3];
const a2  = [1, 2, 3] as const;
```

<img width="276" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/24aaa81c-1cd6-4d76-bf58-c6a6dd824cfa"/>
<img width="276" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/02e76bf3-053b-4126-b983-5fc44387a7cb"/>

넓히기로 인해 오류가 발생한다고 생각되면, 명시적 타입 구문 또는 const 단언문을 추가하는 것을 고려해야 합니다.
단언문으로 인해 추론이 어ㄸ허게 변화하는지 편집기에서 주기적으로 타입을 살펴보기 바랍니다 ([아이템 6](https://github.com/Pyotato/effective_typescript/blob/item6/README.md)).




















