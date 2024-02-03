# ⚙️ ITEM 2 : 타입스크립트 설정 이해하기

다음 코드가 오류 없이 타입 체커를 통과할 수 있을지 생각해 보겠습니다.

```ts
function add(a,b){
  return a+b;
}
add(10,null);
```

설정이 어떻게 되어 있는지 모른다면 대답할 수 없는 질문입니다.
타입스크립트 컴파일러는 매우 많은 설정을 가지고 있습니다.
햔재 시점에는 설정이 거의 100개에 이릅니다.

이 설정들은 커맨드 라인에서 사용할 수 있습니다.

> $ tsc --noImplicitAny program.ts

<i>tsconfig.json</i>설정 파일을 통해서도 가능합니다.

```json
{
  "compileOptions": {
    "noImplicitAny": true
  }
}
```

가급적 설정  파일을 사용하는 것이 좋습니다.
그래야만 타입스크립트를 어떻게 사용할 계획인지 동료들이나 도구들이 알 수 있습니다.
설정 파일은 `ts --init`만 실행하면 간단히 생성됩니다.

타입스크립트의 설정들은 어디서 소스 파일을 찾을지, 어떤 종류의 출력을 생성할지 제어하는 내용이 대부분입니다.
그런데 언어 자체의 핵심 요소들을 제어하는 설정도 있습니다. 
대부분의 언어에서는 허용하지 않는 고수준의 설계의 설정입니다.
타입스크립트는 어떻게 설정하느냐에 따라 완전히 다른 언어처럼 느껴질 수 있습니다.
설정을 제대로 사용하려면, `noImplicitAny`와 `strictNullChecks`를 이해해야 합니다.

`noImplicitAny`는 변수들이 미리 정의된 타입을 가져야 하는지 여부를 제어합니다.
다음 코드는 `noImplicitAny`가 해제되어 있을 때 유효합니다.

```ts
function add(a,b){
  return a+b;
}
``` 

편집기에서 add부분에 마우스를 올려보면, 타입스크립트가 추론한 함수의 타입을 알 수 있습니다.

```ts
function add(a: any,b: any): any
``` 

any타입을 매개변수에 사용하면 타입 체커는 속절없이 무력해집니다. any타입은 유용하지만 매우 주의해서 사용해야 합니다.
[아이템 5](https://github.com/Pyotato/effective_typescript/blob/item5/README.md)과 3장에서 any에 대해 더 자세히 다룹니다.

다음 예제를 보겠습니다.

<img width="486" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/c3cf3256-0a8d-4820-a078-ec77b2c12ac3"/>

any를 코드에 넣지 않았지만, any 타입으로 간주되기 때문에 이를 '암시적 any'라고 부릅니다.

> 한글판 컴파일러는 '명시적(explicit)'의 반대말로 암시적(implicit)이라는 단어를 사용합니다. '암묵적으로 합의된' 정도의 의미로 생각하면 됩니다.

그런데 같은 코드임에도 `noImplicitAny`가 설정되었다면 오류가 됩니다.
이 오류들은 명시적으로 `: any`라고 선언해 주거나 더 분명하나 타입을 사용하면 해결할 수 있습니다. 

```ts
function add(a: number,b: number){
  return a+b;
}
``` 

타입스크립트는 타입 정보를 가질 때 가장 효과적이기 때문에, 되도록이면 `noImplicitAny`를 설정해야 합니다.
모든 변수에 타입을 명시하는 것에 익숙해지면, `noImplicitAny`가 없는 채로 작성된 타입스크립트는 완전히 다른 언어처럼 느껴질 겁니다.

새 프로젝트를 시작한다면 처음부터 `noImplicitAny`를 설정하여 코드를 작성할 때마다 타입을 명시하도록 해야 합니다.
그러면 타입스크립트가 문제를 발견하기 수월해지고, 코드의 가독성이 좋아지며, 개발자의 생산성이 향상됩니다 ([아이템 6](https://github.com/Pyotato/effective_typescript/blob/item6/README.md) 참고). 
`noImplicitAny` 설정 해제는, 자바스크립트로 되어 있는 기존 프로젝트를 타입스크립트로 전환하는 상황에만 필요합니다 (8징 참고).

다음은 `strictNullChecks`가 해제되었을 때 유효한 코드입니다.

```ts
const x : number = null; // 정상, null은 유효한 값입니다.
``` 

그러나 `strictNullChecks`를 설정하면 오류가 됩니다.

<img width="684" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/3e7e8889-fd30-408c-851f-1f93dbb53c55"/>

null 대신 undefined를 써도 같은 오류가 납니다. 만약 null을 허용하려고 한다면, 의도를 명시적으로 드러냄으로써 오류를 고칠 수 있습니다.

```ts
const x : number | null = null;
``` 

만약 null을 허용하지 않으려면, 이 값이 어디서부터 왔는지 찾아야 하고, null을 체크하는 코드나 단언문(assertion)을 추가해야 합니다.

```ts
const el = document.getElementById('status');
el.textContent = 'Ready';

if(el){
el.textContent = 'Ready';
}
el!.textContent = 'Ready';
```

<img width="684" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/4a12a369-a17f-45b3-8b26-3e3f0c550f5f"/>

`strictNullChecks`는 null과 undefined관련된 오류를 잡아 내는 데 많은 도움이 되지만, 코드 작성을 어렵게 합니다.
새 프로젝트를 시작한다면 가급적 `strictNullChecks`를 설정하는 것이 좋지만, 타입스크립트가 처음이거나 자바스크립트 코드를 마이그레이션 중이라면 설정하지 않아도 괜찮습니다.
`strictNullChecks`를 설정하려면 `noImplicitAny`를 먼저 설정해야 합니다.

`strictNullChecks` 설정 없이 개발하기로 선택했다면, "undefined는 개체가 아닙니다"라는 끔찍한 런타임 오류를 주의하기 바랍니다. 
결국은 이 오류 때문에 엄격한 체크를 설정할 수밖에 없을 겁니다.
프로젝트가 거대해질수록 설정 변경은 어려워질 것이므로, 가능한 한 초반에 설정하는게 좋습니다.

언어에 의미적으로 영향을 미치는 설정들(예를들어, `noImplicitThis` `strictFunctionTypes`)이 많지만, `noImplicitAny`와 `strictNullChecks`만큼 중요한 것은 없습니다.
이 모든 체크를 설정하고 싶다면 strict 설정을 하면 됩니다.
타입스크립트에 strict 설정을 하면 대부분의 오류를 잡아냅니다.

공동 프로젝트를 진행하는 도중 동료에게 타입스크립트 예제를 공유했는데 오류가 재현되지 않는다면, 컴파일러 설정이 동일한지 먼저 확인해 봐야 합니다.






































