# 🗺️ ITEM 57: 소스맵을 사용하여 타입스크립트 디버깅하기

타입스크립트 코드를 실행한다는 것은, 엄밀히 말하자면 타입스크립트 컴파일러가 생성한 자바스크립트 코드를 실행한다는 것입니다.
사실 컴파일러뿐 아니라 압축기(minifier)나 전처리기(preprocessor)처럼, 기존 코드를 다른 형태의 코드로 변환하는 도구들에도 모두 해당됩니다.
이러한 변환 과정들이 투명하고 직관적이라면 이상적일 것입니다.
자바스크립트 코드를 살펴볼 필요 없이 마치 타입스크립트 코드가 직접 실행되는 것처럼 느껴진다면 말입니다.
하지만 현실은 그렇지 못합니다.

여러분은 디버깅이 필요한 시점에 비로소 타입스크립트가 직접 실행되는 것이 아니라는 사실을 깨닫게 될 겁니다.
디버거는 런타임에 동작하며, 현재 동작하는 코드가 어떤 과정을 거쳐서 만들어진 것인지 알지 못합니다.
디버깅을 하면 보게 되는 코드는 전처리기, 컴파일러, 압축기를 거친 자바스크립트 코드일 겁니다.
이렇게 변환된 자바스크립트 코드는 복잡해 디버깅하기 매우 어렵습니다.

디버깅 문제를 해결하기 위해 브라우저 제조사들ㅇ느 서로 협력하여 소스맵(source map)이라는 해결책을 내놓았습니다.
소스맵은 변환된 코드의 위치와 심벌들을 워논 코드의 원래 위치와 심벌들로 매핑합니다.
대부부의 브라우저와 많은 IDE가 소스맵을 지원합니다.

클릭할 때마다 숫자를 증가시키는 버튼을 HTML 페이지에 추가하는 간간한 스크립트로 작성한다고 가정해 보겠습니다.

```ts
function addCounter(el: HTMLElement){
  let clickCount = 0;
  const button = document.createElement('button');
  button.textContent = 'Click me';
  button.addEventListener('click', ()=>{
    clickCount++;
    button.textContent = `Click me (${clickCount})`;
  });
  el.appendChild(button);
}
addCounter(document.body);
```

이 코드를 브라우저에서 로드하고 디버거를 열면, 다음처럼 자바스크립트로 변환된 코드를 볼 수 있습니다. 
변환된 코드는 원본 코드와 거의 비슷하기 떄문에 디버깅은 크게 어렵지 않습니다.

<img width="962" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/5c204219-72f0-4d78-bdef-56f5183f2cd8"/>


<i>numbersapi.com</i>으로부터 각 숫자에 대한 흥미러운 설명들을 로드한 후 페이지를 더 재미있게 꾸며 보겠습니다.

```ts
function addCounter(el: HTMLElement) {
  let clickCount = 0;
  const triviaEl = document.createElement("p");
  const button = document.createElement("button");
  button.textContent = "Click me";
  button.addEventListener("click", async () => {
    clickCount++;
    const response = await fetch(`http://numbersapi.com/${clickCount}`);
    const trivia = await response.text();
    triviaEl.textContent = trivia;
  });
  el.appendChild(button);
}

addCounter(document.body);
```

브라우저에서 디버거를 열어 보면, 변환된 코드가 엄청나게 복잡해진 것을 확인할 수 있습니다.

오래된 브라우저에서 async와 await를 지원하기 위해, 타입스크립트는 이벤트 핸들러를 상태 머신 (state machine)으로 재작성합니다.
재작성된 코드는 원본 코드와 동일하게 동작하지만, 코드의 형태는 매우 다른 모습을 띠게 됩니다.

코드가 복잡하게 변환된다면 소스맵이 필요합니다.
타입스크립트가 소스맵을 생성할 수 있도록 <i>tsconfig.json</i>에서 `sourceMap` 옵션을 설정해 보겠습니다.

```json
{
  "compilerOptions":{
    "sourceMap" :true
  }
}
```

이제 컴파일을 실행하면 각 <i>.ts</i>에 대해서  <i>.js</i>와 <i>.js.map</i> 두 개의 파일을 생성합니다. 
<i>.js.map</i>이 바로 소스맵입니다. 

소스맵이 <i>.js</i>파일과 같이 있으면, 브라우저의 디버거에서 새로운 <i>index.ts</i>과 일이 나타납니다.
이제 원하는 대로 브레이크포인트를 설정할 수 있고 변수를 조사할 수 있습니다.

디버거 좌측의 파일 목록에서 <i>index.ts</i>가 기울임(이탤릭) 글꼴로 나오는 것은 확인할 수 있습니다.
기울임 글꼴은 웹 페이지에 포함된 '실제' 파일이 아니라는 것을 뜻합니다.
실제로는 소스맵을 통해 타입스크립트처럼 보이는 것뿐입니다.
그리고 <i>index.js.map</i> 파일이 <i>index.ts</i> 파일의 참조를 포함 (브라우저가 네트워크를 통해 로드)하거나, 파일의 내용을 인라인으로 포함 (별도 요청이 불필요)하도록 설정할 수 있습니다. 

소스맵에 대해 알아야 할 몇 가지 사항을 알아보고 이 아이템을 마무리하겠습니다.

- 타입스크립트와 함께 번들러(bundler)나 압축기(minifier)를 사용하고 있다면, 번들러나 압축기가 각자의 소스맵을 생성하게 됩니다.
이상적인 디버깅 환경이 되려면 생성된 자바스크립트가 아닌 원본 타입스크립트 소스로 매핑되도록 해야 합니다. 
번들러가 기본적으로 타입스크립트를 지원한다면 별도 설정 없이 잘 동작해야 합니다.
그렇지 않다면 번들러가 소스맵을 인식할 수 있도록 추가적인 설정이 필요합니다.
- 상용 환경에 소스맵이 유출되고 잇는지 확인해야 합니다. 
디버거를 열지 않는 이상은 소스맵이 로드되지 않으므로, 실제 사용자에게 성능 저하는 발생하지 않습니다.
그러나 소스맵에 원본 코드의 인라인 복사본이 포함되어 있다면 공개해서는 안 될 내용이 들어 있을 수 있습니다.
저질 주석이나 내부 버그 추적을 위한 URL을 공개할 필요는 없습니다.

NodeJS 프로그램의 디버깅에도 소스맵을 사용할 수 있습니다.
보통 편집기가 자동 인식하거나 NodeJS 프로세스를 브라우저 디버거와 연결하면 됩니다.
자세한 내용은 NodeJS 문서를 참고하길 바랍니다.

타입 체커가 코드를 실행하기 전에 많은 오류를 잡을 수 있지만, 디버거를 대체할 수는 없습니다.
소스맵을 사용해서 제대로 된 타입스크립트 디버깅 환경을 구축하길 바랍니다.
