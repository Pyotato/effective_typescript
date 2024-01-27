# 🧪 ITEM 36 : 해당 분야의 용어로 타입 이름 짓기

> 컴퓨터 과학에서 어려운 일은 단 두 가지뿐이다. 캐시 무효화와 이름 짓기.
>  <br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - 필 카튼

지금까지 타입의 형태와 갑스이 집합에 대해 수없이 다루었지만, 타입의 이름을 짓는 것에 대해서는 거의 언급하지 않았습니다.
그러나 이름 짓기 역시 타입 설계에서 중요한 부분입니다.
엄선된 타입, 속성, 변수의 이름은 의도를 명확히 하고 코드와 타이브이 추상화 수준을 높여 줍니다.
잘못 선택한 타입 이름은 코드의 의도를 왜곡하고 잘못된 개념을 심어 주게 됩니다.

동물들의 데이터베이스를 구축한다고 가정해 보겠습니다.
이를 표현하기 위한 인터페이스는 다음과 같습니다.

```ts
interface Animal {
  name: string;
  endangered : boolean;
  habitat: string;
}
const leopard : Animal = {
  name: 'Snow Leopard',
  endangered : false,
  habitat: 'tundra',
}
```

이 코드에는 네 가지 문제가 있습니다.

- `name`은 매우 일반적인 용어입니다. 동물의 학명인지 일반적인 명칭인지 알 수 없습니다.
- `endangered` 속성이 멸종 위기로 표현하기 위해 boolean 타입을 사용한 것이 이상합니다. 이미 멸종된 동물을 true로 해야 하는지 판단할 수 없습니다. `endangered` 속성의 의도를 '멸종 위기 또는 멸종'으로 생각한 것일지도 모릅니다.
- 서식지를 나타내는 `habitat`속성은 너무 범위가 넓은 string 타입 ([아이템 33](https://github.com/Pyotato/effective_typescript/blob/item33/README.md))일뿐만 아니라 서식지라는 뜻 자체도 불분명하기 때문에 다른 속성들보다도 훨씬 모호합니다.
- 객체의 변수명이 leopard이지만, name 속성의 값은 'Snow Leopard'입니다. 객체의 이름과 속성의 name이 다른 의도로 사용된 것인지 불분명합니다.

이 예제의 문제를 해결하려면, 속성에 대한 정보가 모호하기 때문에 해당 속성을 작성한 사람을 찾아서 의도를 물어봐야 합니다.
그러나 해당 속성을 작성한 사람은 아마도 회사에 이미 없거나 코들르 기억하지 못할 겁니다.
또는 속성을 작성한 사람을 찾기 위해 기록을 뒤졌을 떄, 작성한 사람이 바로 본인이라는 것을 알게 되는 최악의 상황이 펼쳐질 수도 있습니다.

반면, 다음 코드의 타입 선언은 의미가 분명합니다.

```ts
interface Animal {
  commonName: string;
  genus: string;
  species: string;
  status : ConservationStatus;
  climates : KoppenClimate[];
}
type ConservationStatus = 'EX' | 'EW' | 'CR' | 'EN' | 'VU' | 'NT' | 'LC';
type KoppenClimate =
                    'Af' | 'Am' | 'As' | 'Aw' |
                    'BSh' | 'BSk' | 'BWh' | 'BWk' |
                    'Cfa' | 'Cfb' | 'Csa' | 'Csb' | 'Csc' | 'Cwc' | 'Cwa' | 'Cwb' | 'Cwc' |
                    'Dfa' | 'Dfb' | 'Dfc' | 'Dfd' |
                    'Dsa' | 'Dsb' | 'Dsc' | 'Dws' | 'Dwb' | 'Dwc' | 'Dwd' |
                    'ET'  | 'EF' ;

const leopard : Animal = {
  commonName: 'Snow Leopard',
  genus: 'Panthera',
  species: 'Uncia',
  status : 'VU',                  // 취약종 (vulnerable)
  climates : ['ET','EF','Dfd'],   // 고산대 (alpine) 또는 아고산대 (subalpine)
}
```

이 코드는 다음 세 가지를 개선했습니다.

- name은 commonName, genus, species 등 더 구체적인 용어로 대체했습니다.
- endangered는 동물 보호 등급에 대한 IUCN의 표준 체계인 ConservationsStatus 타입의 status로 변경되었습니다.
- habitat은 기후를 뜻하는 climates로 변경되었으며, 쾨펜 기후 분류를 사용했습니다.

이번 예제는 데이터를 훨씬 명확하게 표현하고 있습니다.
그리고 정보를 찾기 위해 사람에 의존할 필요가 없습니다.
쾨펜 기후 분류 체계를 공부하거나 동물 보호 상태의 구체적 의미를 파악하려면, 온라인에서 무수히 많은 정보를 찾을 수 있을 겁니다.

코드로 표현하고자 하는 모든 분야에는 주제를 설명하기 위한 전문 요어들이 있습니다.
자체적으로 용어를 만들어 내려고 하지 말고, 해당 분야에 이미 존재하는 용어를 사용해야 합니다.
이런 용어들은 수년, 수십 년, 수 세기에 걸쳐 다듬어져 왔으며 현장에서 실제로 사용되고 있을 겁니다.
이런 용어들은 사용하면 사용자와 소통에 유리하며 타입의 명확성을 올릴 수 있습니다.

전문 분야의 용어는 정확하게 사용해야 합니다. 
특정 용어를 다른 의미로 잘못 쓰게 되면, 직접 만들어 낸 용어보다 더 혼란을 주게 됩니다.

타입, 속성, 변수에 이름을 붙일 때 명심해야 할 세가지 규칙이 있습니다.

- 동일한 의미를 나타낼 때는 같은 용어를 사용해야 합니다.
글을 쓸 때나, 말을 할 때, 같은 단어를 반복해서 사용하면 지루할 수 있기 때문에 동의어 (의미가 같지만 다른 단어)를 사용합니다.
동의어를 사용하면 글을 읽을 떄는 좋을 수 있지만, 코드에서는 좋지 않습니다.
정말로 의미적으로 구분이 되어야 하는 경우에만 다른 용어를 사용해야 합니다.
- data, info, thing, item, object, entity 같은 모호하고 의미 없는 이름은 피해야 합니다.
만약 entity라는 용어가 해당 분야에서 특별한 의미를 가진다면 괜찮습니다.
그러나 귀찮다고 무심코 의미 없는 이름을 붙여서는 안됩니다.
- 이름을 지을 때는 포함된 내용이나 계산 방식이 아니라 데이터 자체가 무엇인지를 고려해야 합니다. 예를 들어, `INodeList`보다는 `Directory`가 더 의미 있는 이름입니다. `Directory`는 구현의 측면이 아니라 개념적인 측면에서 디렉터리를 생각하게 합니다.
좋은 이름은 추상화의 수준을 높이고 의도치 않은 출동의 위험성을 줄여 줍니다.
