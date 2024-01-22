# ✈️ ITEM 47 : 공개 API에 등장하는 모든 타입을 익스포트하기

타입스크립트를 사용하다 보면, 언젠가는 서드파티의 모둘에서 익스포트되지 않은 타입 정보가 필요한 경우가 생깁니다.
다행히 타입 간의 매핑을 해주는 도구가 많이 있으며, 웬만하면 필요한 타입을 참조하는 방법을 찾을 수 있습니다.
다른 관점으로 생각해 보면, 라이브러리 제작자는 프로젝트 초기에 타입 익스포트부터 작성해야 한다는 의미입니다.
만약 함수의 선언에 이미 타입 정보가 있다면 제대로 익스포트 되고 있는 것이며, 타입 정보가 없다면 타입을 명시적으로 작성해야 합니다.

만약 어떤 타입을 숨기고 싶어서 익스포트하지 않았다고 가정해 보겠습니다. 

```ts
interface SecretName {
  first: string;
  last: string;
}
interface SecretSanta {
  name: SecretName;
  gift: string;
}

export function getGift(name: SecretName. gift: string):SecretSanta{ /** */ }
```

해당 라이브러리 사용자는 SecretName 또는 SecretSanta를 직접 임포트할 수 없고, getGift만 임포트 가능합니다.
그러나 타입들은 익스포트된 함수 시그니처에 등장하기 때문에 추출해 낼 수 있습니다.
추출하는 한가지 방법은 Parameters와 ReturnType 제너릭 타입을 사용하는 것입니다.

```ts
type MySanta = ReturnType<typeof getGift>;    // SecretSanta
type MyName = Parameters<typeof getGift>;     // SecretName
```
만약 프로젝트의 융통성을 위해 타입들을 일부러 익스포트하지 않았던 것이라면, 쓸데없는 작업을 한 셈입니다.
공개 API 매개변수에 놓이는 순간 타입은 노출되기 때문입니다.
그러므로 굳이 숨기려 하지 말고 라이브러리 사용자를 위해 명시적으로 익스포트하는 것이 좋습니다.
