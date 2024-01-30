# 🛕 ITEM 55: DOM 계층 구조 이해하기

이 책의 내용은 대부분 타입스크립트의 실행 환경 (웹브라우저, 서버, 모바일)과 무관하지만, 여기서 다룰 내영은 브라우저와 관련되어 있습니다.
브라우저에서 동작하는 프로젝트를 다룰 게 아니라면 이번 아이템은 건너뛰어도 됩니다.

DOM 계층은 웹브라우저에서 자바스크립트를 실행할 때 어디에서나 존재합니다.
엘리먼트를 얻기 위해 `document.getElementById`를 사용할 때나 엘리먼트를 생성하기 위해 `document.createElement`를 사용할 때, 두 개의 차이가 무엇인지 모르더라도 결과가 어떠한 엘리먼트라는 것은 분명합니다.
그리고 많은 부분에서 엘리먼트의 DOM과 관련된 메서드를 사용하고 엘리먼트의 속성을 사용하게 됩니다.

타입스크립트에서는 DOM 엘리먼트의 게층 구조를 파악하기 용이합니다.
Element와 EventTarget에 달려 있는 Node의 구체적인 타입을 안다면 타입 오류를 디버깅할 수 있고, 언제 타입 단언을 사용해야 할지 알 수 있습니다.
그리고 대다수의 브라우저 API가 DOM을 기반으로 하기 때문에, 리액트나 d3같은 프레임워크도 DOM이 관련되어 있습니다.

`<div>`의 경계를 넘어서 마우스르 움직이는 경우를 추적하고 싶다고 가정해 보겠습니다.
다음 예제는 언뜻 보기에 문제가 없는 자바스크립트 코드입니다.

```ts
function handleDrag(eDown: Event) {
  const targetEL = eDown.currentTarget;
  targetEl.classList.add("dragging");
  const dragStart = [eDown.clientX, eDown.clientY];
  const handlerUp = (eUp: Event) => {
    taretEl.classList.remove("dragging");
    targetEl.removeEventListener("mouseup", handleUp);
    const dragEnd = [eUp.clientX, eUp.clientY];
    console.log(
      "dx, dy =",
      [0, 1].map((i) => dragEnd[i] - dragStart[i])
    );
  };
  targetEl.addEventListener("mouseup", handleUp);
}

const div = document.getElementById("surface");
div.addEventListener("mousedown", handleDrag);
```

그러나 타입스크립트에서는 수많은 오류가 표시됩니다.

<img width="1125" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/44ec4f88-0eb4-4122-8e75-1ea9263139f3"/>

먼저 EventTarget타입 관련된 오류를 살펴보겠습니다.

EventTarget오류를 이해하려면 DOM계층 구조를 자세히 살펴봐야 합니다.
다음 HTML 코드를 보겠습니다.

```html
<p id="quote">and<i>yet</i>it moves</p>
```

브라우저에서 자바스크립트 콘솔을 열고 p 엘리먼트의 참조를 얻어보면, HTMLParagraphElement 타입이라는 것을 알 수 있습니다.

```ts
const p = document.getElemntsByTagName("p");
// p instance of HTMLParagraphElement // true
```

HTMLParagraphElement는 HTMLElement의 서브타입이고, HTMLElement는 Element의 서브타입입니다.
또한 Element는 Node의 서브타입이고, Node는 EventTarget의 서브타입입니다.

<div id="table">다음은 계층 구조에 따른 타입의 몇 가지 예시입니다.</div>

| 타입              | 예시                         |
| :---------------- | :--------------------------- |
| EventTarget       | window, XMLHttpRequest       |
| Node              | document, Text, Comment      |
| Element           | HTMLElement, SVGElement 포함 |
| HTMLElement       | `<i>`, `<b>`                     |
| HTMLButtonElement | `<button>`                     |

게층 구조별로 타입을 좀 더 자세히 알아보겠습니다.

> 첫 번쨰, EventTarget은 DOM 타입 중 가장 추상화된 타입입니다.

이벤트 리스너를 추가하거나 제거하고, 이벤트를 보내는 것밖에 할 수 없습니다.
오류가 발생한 부분을 다시 살펴보겠습니다.

<img width="770" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/c1a17408-505f-4e05-bfe6-7cfa5ac83ba9"/>

Event의 currentTarget속성은 `EventTarget | null`입니다.
그렇기 때문에 null 가능성이 오류로 표시되었고, 또한 EventTarget 타입에 classList 속성이 없기 때문에 오류가 되었습니다.
한편, eDown.currentTarget은 실제로 HTMLElement 타입이지만, 타입 관점에서는 window나 XMLHTTPRequest가 될 수도 있다는 점을 주의하기 바랍니다.

> 두 번째, Node 타입을 알아보겠습니다.

Element가 아닌 Node인 경우를 몇 가지 예로 들어 텍스트 조각과 주석이 있습니다.
예를 들어, 다음 HTML 코드를 예로 들어 보겠습니다.

```html
<p>
  And <i>yet</i> it moves
  <!--quote from Galileo-->
</p>
```

가장 바깥쪽의 엘리먼트는 HTMLParagraphElement입니다.
그리고 children과 childNodes 속성을 가지고 있습니다.

> p.children <br/>
> HTMLCOllection [i]<br/>
> p.childNodes <br/>
> NodeList(5) [text, i, text, comment, text]

children은 자식 엘리먼트 (`<i>yet</i>`)를 포함하는 배열과 유사한 구조인 HTMLCollection 입니다.
반면 childNodes는 배열과 유사한 Node의 컬렉션인 NodeList입니다.
childNodes는 엘리먼트(`<i>yet</i>`)뿐만 아니라 텍스트 조각 (`"And"`와 `"it moves"`)과 주석 (`<!--quote from Galileo-->`)까지도 포함하고 있습니다.

> 세 번째, Elements와 HTMLElement를 알아보겠습니다.

HTMLxxxElement 형태의 특정 엘리먼트들은 자신만의 고유한 속성을 가지고 있습니다.
예를 들어, HTMLImageElement에는 src 속성이 있고, HTMLInputElement에는 value 속성이 있습니다.
이런 속성에 접근하려면, 타입 정보 역시 실제 엘리먼트 타입이어야 하므로 상당히 구체적으로 타입을 지정해야 합니다.

보통은 HTML 태그 값에 해당하는 'button'같은 리터럴 값을 사용하여 DOM에 대한 정확한 타입을 얻을 수 있습니다.
예를 들자면, 다음과 같습니다.

```js
document.getElementsByTagName("p")[0]; // HTMLParagraphElement
documetn.createElement("button"); // HTMLButtonElement
document.querySelector("div"); // HTMLDivElement
```

그러나 항상 정확한 타입을 얻을 수 이쓴ㄴ 것은 아닙니다.
특히 `document.getElementsById`에서 문제가 발샐하게 됩니다.

```js
document.getElementById("my-div"); // HTMLElement
```

일반적으로 타입 단언문은 지양해야 하지만 ([아이템 9](https://github.com/Pyotato/effective_typescript/tree/item9)), DOM 관련해서는 타입스크립트보다 우리가 더 정확히 알고 있는 경우이므로 단언문을 사용해도 좋습니다.
`#my-div`가 div 태그라는 것을 알고 있으므로 문제가 되지 않습니다.

```ts
document.getElementById("my-div") as HTMLDivElement;
```

`strictNullChecks`가 설정된 상태라면, `document.getElementById`가 null인 경우를 체크해야 합니다.
실제 코드에서 `document.getElementById`가 null일 가능성이 있다면 if 분기문을 추가해야 합니다.

<img width="955" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/719c205c-a3be-47d2-b70b-6953accce715"/>

[표](#table)에서 살펴봤던 `EventTarget` 타입의 계층 구조뿐 아니라, `Event`타입에도 별도의 계층 구조가 있습니다.
Mozilla 문서에는 52개 이상의 Event 종류가 나열되어 있습니다.

Event는 가장 추상화된 이벤트입니다.
더 구체적인 타입들은 다음과 같습니다.

- UIEvent: 모든 종류의 사용자 인터페이스 이벤트
- MouseEvent: 클릭처럼 마우스로부터 발생되는 이벤트
- TouchEvent: 모바일 기기의 터치 이벤트
- WheelEvent: 스크롤 휠을 돌려서 발생되는 이벤트
- KeyboardEvent: 키 누름 이벤트

clientX와 clientY에서 발생한 오류의 원인은, handleDrag 함수의 매개변수는 `Event`로 선언된 반면 clientX와 clientY는 보다 구체적인 `MouseEvent`타입이기 때문입니다.

그렇다면 오류를 어떻게 고칠 수 있을까요?
DOM에 대한 타입 추론은 문맥의 정보를 폭넓게 확용합니다 ([아이템 26](https://github.com/Pyotato/effective_typescript/blob/item26/README.md)).
`'mousedown'`이벤트 핸들러를 인라인 함수로 만들면 타입스크립트는 더 많은 문맥 정보를 사용하게 되고, 대부분의 오류를 제거할 수 있습니다.
또한 매개변수 타입을 Event 대신 `MouseEvenet`로 선언할 수 있습니다.
다음 예제는 방금 언급한 인라인 함수와 이벤트 타입 변경을 적용해서 오류를 제거한 코드입니다.

```ts
function addDraghandler(el: HTMLElement) {
  el.addEventListener("mousedown", (eDown) => {
    const dragStart = [eDown.clientX, eDown.clientY];
    const handleUp = (eUp: MouseEvent) => {
      el.classList.remove("dragging");
      el.removeEventListener("mouseup", handleUp);
      const dragEnd = [eUp.clientX, eUp.clientY];
      console.log(
        "dx, dy =",
        [0, 1].map((i) => dragEnd[i] - dragStart[i])
      );
    };
    el.addEventListener("mouseup", handleUp);
  });
}

const div = document.getElementById("surface");
if (div) addDraghandler(div);
```

코드 마지막의 if구뭄은 `#surface` 엘리먼트가 없는 경우를 체크합니다.
만약 해당 엘리먼트가 반드시 존재한다는 것을 알고 있다면, if구문 대신 단언문을 사용할 수도 있습니다(`addDraghandler(div!)`).
