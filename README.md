#  🚥 ITEM 11 : 잉여 속성 체크의 한계 인지하기

타입이 명시된 객체 리터럴을 할당할 때 타입스크립트는 해당 타입이 속성이 있는지, 그리고 '그 외의 속성은 없는지' 확인합니다.

```ts
interface Room{
  numDoors :number;
  ceilingHeightFt: number;
}

const r: Room = {
  numDoors :1,
  ceilingHeightFt: 10,
  elephant: 'present'
}

```

<img width="1097" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/810837d8-bd0d-4e48-b7e8-57955a0361c8"/>

Room 타입에 생뚱맞게 elephant 속성이 있는 것이 어색하긴 하지만, 구조적 타이핑 관점 ([아이템 4](https://github.com/Pyotato/effective_typescript/blob/item4/README.md))으로 생각해보면 오류가 발생하지 않아야 합니다. 
임시 변수를 도입해 보면 알 수 았는데, obj 객체는 Room 타입에 할당이 가능합니다.

```ts
const obj = {
  numDoors :1,
  ceilingHeightFt: 10,
  elephant: 'present',
};

const r: Room = obj;  //  정상

```
obj의 타입은 `{numDoors: number; ceilingHeightFt: number; elephant: string;}`으로 추론됩니다.
obj타입은 Room 타입의 부분 집합을 포함하므로, Room에 할당 가능하며 타입 체커도 통과합니다. [(아이템 7)](https://github.com/Pyotato/effective_typescript/blob/item7/README.md)

앞 두 예제의 차이점을 살펴보겠습니다. 첫 번째 예제에서는, 구조적 타입 시스템에서 발생할 수 있는 중요한 종류의 오류를 잡을 수 있도록 '**잉여 속성 체크**'라는 과정이 수행되었습니다.
그러나 잉여 속성 체크 역시 조건에 따라 동작하지 않는다는 한계가 있고, 통상적인 할당 가능 검사와 함께 쓰이면 구조적 타이핑이 무엇인지 혼란스러워질 수 있습니다. 
잉여 속성 체크가 할당 가능 검사와는 별도의 과정이라는 것을 알야야 타입스크립트 *타입 시스템에 대한 개념*을 정확히 잡을 수 있습니다.

> 💡 원서에는 *개념* 대신 심성 모델 (mental model)이라는 용어를 사용하고 있습니다. <br/>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;심성 모형이란 각자가 특정 대상을 바라보는 마음 속의 모형을 의미합니다. <br/>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;이 책에서는 낯선 표현인 심성 모형 대신 '*개념*'이라는 단어를 사용했습니다.

```ts
interface Options{
  title : string;
  darkMode?: boolean;
}

function setDarkMode(){}

function createWindow(options: Options){
  if(options.darkMode){
    setDarkMode();
  }
  // ...
}

createWindow({
  title: 'Spider Solitaire',
  darkmode: true
})

```

<img width="1591" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/d889ceaa-01dc-4f6e-8ecb-a847e62950cb"/>

앞의 코드를 실행하면 런타임에 어떠한 종류의 오류도 발생하지 않습니다.
그러나 타입스크립트가 알려주느 오류 메시지처럼 의도한 대로 동작하지 않을 수 있습니다. 오류가 발생한 부분은 `darkmode`가 아닌 `darkMode`이어야합니다.

Options 타입은 범위가 매우 넓기 때문에, 순수한 구조적 타입 체커는 이런 종류의 오류를 찾아내지 못합니다.
`darkMode` 속성에 boolean 타입이 아닌 다른 타입의 값이 지정된 경우를 제외하면, string 타입인 title속성과 '또 다른 어떤 속성'을 가지는 모든 객체는 Options 타입의 범위에 속합니다.

타입스크립트 타입은 범위가 아주 넓어질 수 있습니다. 여기 Options에 할당할 수 있는 몇 가지 값이 더 있습니다.

```ts
const o1: Options = document; // 정상
const o2 : Options = new HTMLAnchorElement; // 정상
```

`document`과 `HTMLAnchorElement`의 인스턴스 모두 `string` 타입인 `title` 속성을 가지고 있기 때문에 할당문은 정상입니다.
`Options`은 정말 넓은 타입이라는 것을 알 수 있습니다.

잉여 속성 체크를 이용하면 기본적으로 타입 시스템의 구조와 본질을 해치지 않으면서도 객체 리터럴에 알 수 없는 속성을 허용하지 않음으로써, 
앞에서 다룬 `Room`이나 `Options` 예제 같은 문제점을 방지할 수 있습니다.
(그래서 '엄격한 객체 리터럴 체크'라고도 불립니다.)
`document`나 `new HTMLAnchorElement`는 객체 리터럴이 아니기 때문에 잉여 속성 체크가 되지 않습니다.
그러나 `{title, darkmmode}` 객체는 체크가 됩니다.

```ts
const o: Options = {title: 'Ski Free', darkmode: true};

```

<img width="1591" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/8206e255-1641-407a-b277-601c7917af94"/>

오류가 사라지는 이유를 알아내기 위해 타입 구문 없는 임시 변수를 사용해 보겠습니다.

```ts
const o: Options = {title: 'Ski Free', darkmode: true};
const o : Options = intermediate; // 정상
```

첫번째 줄의 오른쪽은 객체 리터럴이지만, 두번째 줄의 오른쪽 (intermidate)은 객체 리터럴이 아닙니다.<br/>
따라서 잉여 속성 체크가 적용되지 않고 오류는 사라집니다.

잉여 속성 체크는 타입 단언문을 사용할 때에도 적용되지 않습니다.

```ts
const o = intermediate as Options; // 정상
```

이 예제가 단언문보다 선언문을 사용하는 단적인 이유 중 하나입니다. ([아이템 9](https://github.com/Pyotato/effective_typescript/blob/item9/README.md))

잉여 속성 체크를 원치 않다면, 인덱스 시그니처를 사용해서 타입스크립트가 추가적인 속성을 예상하도록 할 수 있습니다.

```ts
interface Options{
  darkMode?: boolean;
  [otherOptions: string] : unknown;
}

const o : Options = {darkmode: true};  // 정상

```

이런 방법이 데이터를 모델링하는 데 적절한지 아닌지에 대해서는 [아이템 15](https://github.com/Pyotato/effective_typescript/blob/item15/README.md)에서 다룹니다.

선택적 속성만 가지는 '약한(weak)' 타입에도 비슷한 체크가 동작합니다.

```ts
interface LineChartOptions{
  logscale?: boolean;
  invertedYAxis?: boolean;
  areaChart?: boolean;
}

const opts : Options = {logScale: true};
const o: LineChartOptions = opts;

```

<img width="1253" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/6f6de0f3-ab24-4fd6-93a7-b1a0961cdcdc"/>

구조적 관점에서 `LineChartOptions` 타입은 모든 속성이 선택적이므로 모든 객체를 포함할 수 있습니다. 
이런 약한 타입에 대해서 타입스크립트는 값 타입과 선언 타입에 공통된 속성이 있는지 확인하는 별도의 체크를 수행합니다.
공통 속성 체크는 잉여 속성 체크와 마찬가지로 오타를 잡는데 효과적이며 구조적으로 엄격하지 않습니다.
그러나 잉여 속성 체크와 다르게 약한 타입과 관련된 할당문마다 수행됩니다.
임시 변수를 제거하더라도 공통 속성 체크는 여전히 동작합니다.

잉여 속성 체크는 구조적 타이핑 시스템에서 허용되는 속성 이름의 오타 같은 실수를 잡는 데 효과적인 방법입니다.
선택적 필드를 포함하는 `Options`같은 타입에 특히 유용한 반면, 적용 범위도 매우 제한적이며 오직 객체 리터럴에만 적용됩니다.
이러한 한계점을 인지하고 잉여 속성 체크와 일반적으로 타입 체크를 구분한다면, 두 가지 모두의 개념을 잡는 데에 도움이 될 것입니다.

잉여 속성 체크가 어떻게 버그르 잡고, 새로운 설계 가능성을 보여주는지에 대한 구체적인 예제는 [아이템 18](https://github.com/Pyotato/effective_typescript/blob/item18/README.md)에서 다룹니다.
또한 임시 상수를 도입함으로써 잉여 속성 체크 문제를 해결하지만, 문맥 관점의 오류를 발생시키는 예제는 [아이템26](https://github.com/Pyotato/effective_typescript/blob/item26/README.md))에서 다룹니다.
