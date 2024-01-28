# 🐒 ITEM 43 : 몽키패치보다는 안전한 타입 사용하기

자바스크립트의 가장 유명한 특징 중 하나는, 객체와 클래스에 임의의 속성을 추가할 수 있을 만틈 유연하다는 것입니다.
객체에 속성을 추가할 수 있는 기능은 종종 웹 페이지에서 `window`나 `document`에 값을 할당하여 전역 변수를 만드는 데 사용합니다.

```ts
window.monkey = 'Tamarin';
window.monkey = 'Howler';
```

또는 DOM 엘리먼트에 데이터를 추가하기 위해서도 사용됩니다.

```ts
const el = document.getElementById('colobus');
el.home = 'tree';
```

객체에 속성을 추가하는 코드 스타일은 특히 제이쿼리(jQuery)를 사용하는 코드에서 흔히 볼 수 있습니다.

심지어 내장 기능의 프로토타입에도 속성을 추가할 수 있습니다.
그런데 이상한 결과를 보일 때가 있습니다.

```ts
Regex.prototype.mokey = 'Capuchin'
//"Capuchin"
/123/.monkey;
//"Capuchin"
```

정규식`(/123/)`에 monkey라는 속성을 추가한 적이 없는데 "Capuchin"이라는 값이 들어 있습니다.
사실 객체에 임의의 속성을 추가하는 것은 일반적으로 좋은 설계가 아닙니다.
예를 들어 window 또는 DOM 노드에 데이터를 추가한다고 가정해 보겠습니다.
그러면 그 데이터는 기본적으로 전역 변수가 됩니다.
전역 변수를 사용하면 은연중에 프로그램 내에서 서로 멀리 떨어진 부분들 간에 의존성을 만들게 됩니다.
그러면 함수를 호출할 때마다 부작용 (side effect)을 고려해야만 합니다.

타입스크립트까지 더하면 또 다른 문제가 발생합니다.
타입 체커는 Document와 HTMLElement의 내장 속성에 대해서는 알고 있지만, 임의로 추가한 속성에 대해서는 알지 못합니다.

```ts
document.monkey = 'Tamarin';
```

<img width="789" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/ff4a1960-3e24-43ef-bae7-e04d2520e146"/>

이 오류를 해결하는 가장 간단한 방법은 any 단언문을 사용하는 것입니다.

```ts
(document as any).monkey = 'Tamarin';
```

타입 체커는 통과하지만 단점이 있습니다.
any를 사용함으로써 타입 안전성을 상실하고, 언어 서비스를 사용할 수 없게 된다는 것입니다.

```ts
(document as any).monky = 'Tamarin';   // 오타지만 정상
(document as any).monkey = 'Tamarin';  // 타입이 잘못됐지만 정상
```

최선의 해결책은 docmeent 또는 DOM으로부터 데이터를 분리하는 것입니다.
분리할 수 없는 경우 (객체와 데이터가 붙어 있어야만 하는 라이브러리를 사용중이거나 자바스크립트 애플리케이션을 마이그레이션하는 과정 중이라면), 두 가지 차선책이 존재합니다.

```ts
interface Document {
  /** 몽키 패치의 속(genus) 또는 종 (species) */
  monkey : string;
}
document.monkey = 'Tamarin';  // 정상
```

보강을 사용하는 방법이 `any`보다 나은 점은 다음과 같습니다.

- 타입이 더 안전합니다. 타입 체커는 오타나 잘못된 타이브이 할당을 오류로 표시합니다.
- 속성에 주석을 붙일 수 있습니다 ([아이템 48](https://github.com/Pyotato/effective_typescript/blob/item48/README.md)).
- 속성에 자동완성을 사용할 수 있습니다.
- 몽키 패치가 어떤 부분에 적용되었는지 정확한 기록이 남습니다.

그리고 모듈의 관점에서 (타입스크립트 파일이 `import/export`를 사용하는 경우), 제대로 동작하게 하려면 `global` 선언을 추가해야 합니다.

```ts
import [};
declare global {
  interface Document {
    /** 몽키 패치의 속(genus) 또는 종 (species) */
    monkey : string;
  }
}

document.monkey = 'Tamarin';  // 정상
```

보강을 사용할 때 주의해야 할 점은 모듈 영역(scope)과 관련이 있습니다.
보강은 전역적으로 적용되기 때문에, 코드의 다른 부분이나 라이브러리로부터 분리할 수 없습니다.
그리고 애플리케이션이 실행되는 동안 속성을 할당하면 실행 시점에서 보강을 적용할 방법이 없습니다.
특히 웹 페이지 내의 HTML 엘리먼트를 조작할 때, 어떤 엘리먼트는 속성이 있고 어떤 엘리먼트는 속성이 없는 경우 문제가 됩니다.
이러한 이유로 속성을 `string|undefined`로 선언할 수도 있습니다.
이렇게 선언하면 더 정확할 수 잇지만 다루기에는 더 불편해집니다.

두 번째, 더 구체적인 타입 단언문을 사용하는 것입니다.

```ts
interface MonkeyDocument extends Document{
    /** 몽키 패치의 속(genus) 또는 종 (species) */
    monkey : string;
  }
}

(document as MonkeyDocument).monkey = 'Tamarin';  // 정상
```

MonkeyDocument는 Document를 확장하기 때문에 ([아이템 9](https://github.com/Pyotato/effective_typescript/blob/item9/README.md)) 타입 단언문은 정상이며 할당문의 타입은 안전합니다.
또한 Document 타입을 건드리지 않고 별도로 확장하는 새로운 타입을 도입했기 때문에 모듈 영역 문제도 해결할 수 있습니다 (import하는 곳의 영역에만 해당됨). 
따라서 몽키 패치된 속성을 참조하는 경우에만 단언문을 사용하거나 새로운 변수를 도입하면 됩니다.
그러나 몽키 패치를 남용해서는 안 되며 궁극적으로 더 잘 설계된 구조로 리팩터링하는 것이 좋습니다.
