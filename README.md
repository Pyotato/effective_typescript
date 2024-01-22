#  🫡 ITEM 26 : 타입 추론에 문맥이 어떻게 사용되는지 이해하기

타입스크립트는 타입을 추론할 때 단순히 값만 고려하지는 않습니다.
값이 존재하는 곳의 문맥까지도 살핍니다. 
그런데 문맥을 고려해 타입을 추론하면 가끔 이상한 결과가 나옵니다.
이때 타입 추론에 문맥이 어떻게 사용되는지 이해하고 있다면 제대로 대처할 수 있습니다.

자바스크립트는 코드의 동작과 실행 순서를 바꾸지 않으면서 표현식을 상수로 분리해 낼 수 있습니다.
예를 들어, 다음 두 문장은 동일합니다.

```ts
setLanguage('JavaScript');    // 인라인 형태

let language = 'JavaScript'
setLanguage(language);        // 참조 형태
```

타입스크립트에서는 다음 리팩터링이 여전히 동작합니다.

```ts
function setLanguage(language: string){/** */}

setLanguage('JavaScript');

let language = 'JavaScript'
setLanguage(language); 

```

이제 문자열 타입을 더 특정해서 문자열 리터럴 타입의 유니온으로 바꾼다고 가정해 보겠습니다 ([아이템 33](https://github.com/Pyotato/effective_typescript/blob/item33/README.md)에서 자세히 다룹니다).

```ts
type Language  = 'JavaScript' | 'TypeScript' | 'Python';
function setLanguage(language: Language){/** */}

setLanguage('JavaScript');

let language = 'JavaScript'
setLanguage(language); 

```

<img width="1366" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/bd122aff-954e-43bc-9c38-642ab2c2b22c"/>

인라인(inline) 형태에서 타입스크립트는 함수 선언을 통해 매개변수가 Language 타입이어야 한다는 것을 알고 있습니다. 
해당 타입에 문자열 리터럴 'JavaScript'는 할당 가능하므로 정상입니다.
그러나 이 값을 변수로 분리해 내면, 타입스크립트는 할당 시점에 타입을 추론합니다.
이번 경우는 string으로 추론했고, Language 타입으로 할당이 불가능하므로 오류가 발생했습니다.

> 어떤 언어는 변수의 최종 사용처에 기반하여 타입을 추론하기도 합니다. <br/>
> 그러나 이런 바업 역시 혼란스럽습니다. <br/>
> 타입스크립트 창시자인 아네르스 하일스베르는 이를 '먼 곳의 소름끼치는 일 (spooky action at a distance)'이라고 했습니다. <br/>
> 타입스크립트는 일반적으로 값이 처음 등장할 때 타입을 결정합니다. <br/>
> [아이템 41](https://github.com/Pyotato/effective_typescript/blob/item41/README.md)에서 이 규칙에 대한 중요한 예외 상황을 다룹니다.  <br/>


이러한 문제를 해결하는 두 가지 방법이 있습니다.

첫 번째 해법은 타입 선언에서 language의 가능한 값을 제한하는 것입니다. 

```ts
let language : Language= 'JavaScript'
setLanguage(language); 
```

만약 language의 값에 'Typescript' (대문자 'S'여야 합니다) 같은 오타가 잇었다면 오류를 표시해 주는 장점도 있습니다.

두 번째 해법은 language를 상수로 만드는 것입니다.

```ts
const language = 'JavaScript'
setLanguage(language); 
```

const를 사용하여 타입 체커에게 language는 변경할 수 없다고 알려 줍니다.
따라서 타입스크립트는 language에 대해서 더 정확한 타입인 문자열 리터럴 'JavaScript'로 추론할 수 있습니다.
'JavaScript'는 Language에 할당할 수 있으므로 타입 체크를 통과합니다. 물론 language를 재할당해야 한다면 타입 선언이 필요합니다 ([아이템 21](https://github.com/Pyotato/effective_typescript/blob/item21/README.md)에서 자세히 다룹니다).

그런데 이 과정에서 사용되는 문맥으로부터 값을 분리했습니다.
문맥과 값을 분리하면 추후에 근본적인 문제를 발생시킬 수 있습니다.
이제부터 이러한 문맥의 소실로 인해 오류가 발생하는 몇 가지 경우와, 이를 어떻게 해결하는지 하나하나 살펴보겠습니다.

## 튜플 사용 시 주의점

문자열 리터럴 타입과 마찬가지로 튜플 타입에서도 문제가 발생합니다.
이동이 가능한 지도를 보여 주는 프로그램을 작성한다고 생각해 보겠습니다.

```ts
// 매개변수는 (latitude, longitude) 쌍입니다.
function panTo(where: [number, number]){/** */}

panTo([10, 20]);

const loc = [10,20];
panTo(loc);
```

<img width="1438" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/50e1fb79-9b84-44cd-8811-12c68e190529"/>

이전 예제처럼 여기서도 문맥과 값을 분리했습니다.
첫 번째 경우는 [10, 20]이 튜풀 타입 [number, number]에 할당 가능합니다.
두 번째 경우는 타입스크립트가 loc의 타입을 number[]로 추론합니다 (즉, 길이를 알 수 없는 숫자의 배열).
많은 배열이 이와 맞지 않는 수의 요소를 가지므로 튜플 타입에 할당할 수 없습니다.

그러면 any를 사용하지 않고 오류를 고칠 수 있느 방법을 생각해 보겠습니다.
any대신 const로 선언하면 된다는 답이 떠오를 수도 있겠지만 loc은 이미 const로 선언한 상태입니다.
그보다는 타입스크립트가 의도를 정확히 파악할 수 있도록 타입 선언을 제공하는 방법을 시도해 보겠습니다.

```ts
const loc :[number, number] = [10,20];
panTo(loc);   // 정상
```

any를 사용하지 않고 오류를 고칠 수 있는 또 다른 방법은 '상수 문맥'을 제공하는 것입니다.
const는 단지 값이 가리키는 참조가 변하지 않는 얕은(shallow)상수인 반면, as const는 그 값이 내부까지 (deeply) 상수라는 것을 타입스크립트에게 알려줍니다.

```ts
const loc  = [10,20] as const;
panTo(loc);   
```

<img width="1608" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/2e276bde-385a-4de8-add0-a5c7c2c9c1cf"/>

편집기에서 loc에 마우스를 올려보면, 타입은 이제 number[]가 아니라 readonly [10,20]으로 추론됨을 알 수 있습니다.
그런데 안타깝게도 이 추론은 '너무 과하게' 정확합니다.
panTo의 타입 시그니처는 where의 내용이 불변이라고 보장하지 않습니다.
즉, loc 매개변수가 readonly 타입이므로 동작하지 않습니다.

따라서 any를 사용하지 않고 오류를 고칠 수 있는 최선의 해결책은 panTo 함수에 readonly 구문을 추가하는 것입니다.

```ts
function panTo(where: readonly [number, number]){/** */}
const loc  = [10,20] as const;
panTo(loc);   
```

타입 시그니처를 수정할 수 없는 경우라면 타입 구문을 사용해야 합니다.

as const는 문맥 손실과 관련한 문제를 깔끔하게 해결할 수 있지만, 한 가지 단점을 가지고 있습니다.
만약 타입 정의에 실수가 있다면 (에를 들어, 튜플에 세 번째 요소를 추가한다면) 오류는 타입 정의가 아니라 호출되는 곳에서 발생한다는 것입니다.
특히 여러 겹 중첩된 객체에서 오류가 발생한다면 근본적인 원인을 파악하기 어렵습니다.

```ts
const loc = [10,20,30] as const;  // 실제 오류는 여기서 발생합니다.
panTo(loc);
```

<img width="1608" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/dc04567e-d8bf-4f5f-bd20-a372f2ce5a45"/>


## 객체 사용 시 주의점

문맥에서 값을 분리하는 문제는 문자열 리터럴이나 튜플을 포함하는 큰 객체에서 상수를 뽑아낼 때도 발생합니다.

```ts
type Language =  'JavaScript' | 'TypeScript' | 'Python';
interface GovernedLanguage{
  language: Language;
  organization: string;
}

function complain(language: GovernedLanguage){/** */}
complain({language: 'TypeScript', organization: 'Microsoft'})

const ts = {
  language: 'TypeScript',
  organization: 'Microsoft'
}

complain(ts);
```

<img width="1566" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/195b3bc4-1207-4992-b318-d11a0a718427"/>

ts객체에서 language의 타입은 string으로 추론됩니다. 이 문제는 타입 선언을 추가하거나 (const ts: GovernedLanguage = ...) 상수 단언(as const)을 사용해 해결합니다 ([이이템 9](https://github.com/Pyotato/effective_typescript/blob/item9/README.md)). 


## 콜백 사용 시 주의점

콜백을 다른 함수로 전달할 때, 타입스크립트는 콜백의 매개변수 타입을 추론하기 위해 문맥을 사용합니다.

```ts
function callWithRandomNumbers(fn: (n1: number, n2: number)=> void){
  fn(Math.random(), Math.random());
}

callWithRandomNumbers((a, b)=>{
  a; // 타입이 number
  b; // 타입이 number
  console.log(a+b);
})
```

`callWithRandomNumbers`의 타입 선언으로 인해 a와 b의 타입이 number로 추론됩니다.
콜백을 상수로 뽑아내면 문맥이 소실되고 `noImplicitAny` 오류가 발생하게 됩니다.

```ts

const fn = (a,b) =>{
    console.log(a+b);
}
callWithRandomNumbers(fn);
```

<img width="1100" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/8f5ef105-331a-4283-a40a-adc4d71a44bd"/>

이런 경우는 매개변수에 타입 구문을 추가해서 해결할 수 있습니다.

```ts

const fn = (a: number,b: number) =>{
    console.log(a+b);
}
callWithRandomNumbers(fn);
```

또는 가능할 경우 전체 함수 표현식에 타입 선언을 적용하는 것입니다 (자세한 내용은 [아이템 12](https://github.com/Pyotato/effective_typescript/blob/item12/README.md)에서 다뤘습니다).















