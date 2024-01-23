#  🤙 ITEM 49 : 콜백에서 this에 대한 타입 제공하기

자바스크립트에서 this 키워드는 매우 혼란스러운 기능입니다.
let이나 const로 선언된 변수가 렉시컬 스코프 (lexical scope)인 반면, this는 다이나민 스크프(dynamic scope)입니다.
다이나믹 스코프의 값은 `'정의된`' 방식이 아니라 `'호출된 방식'`에 따라 달라집니다.

> 이 책에서 자바스크립트 언어 기초를 다루지 않지만, 기초 서적이나 검색을 통해 배울 수 있습니다. 변수 스코프 관련된 특징을 반드시 숙지하길 바랍니다. <br/>
> 👉[스코프](https://developer.mozilla.org/ko/docs/Glossary/Scope)

this는 전형적으로 객체의 현재 인스턴스를 참조하는 클래스에서 가장 많이 쓰입니다. 

```ts
class C{
  vals = [1, 2, 3];
  logSquares(){
      for(const val of this.vals){
        console.log(val * val);
      }
  }
}
const c = new C();
c.logSquares();
```

<img width="851" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/c3896b9e-d80b-4ed3-9818-2f32b4025507"/>


이제 logSquares룰 외부 변수에 넣고 호출하면 어떻게 되는지 보겠습니다.

```ts
const c = new C();
const method = c.logSquares;
method();
```

이 코드는 런타임에 다음과 같은 오류가 발생합니다.

<img width="1298" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/bbd0e335-c797-4ab1-9f13-52512f079dba"/>

c.logSquares()기 실제로는 두 가지 작업을 수행하기 때문에 문제가 발생합니다.
C.prototype.logSquares를 호출하고, 또한 this의 값을 c로 바인딩합니다.
앞의 코드에서는 logSquares의 참조 변수를 사용함으로써 두 가지 작업을 분리했고, this의 값은 undefined로 설정됩니다.

자바스크립트에는 this바인딩을 온전히 제어할 수 있는 방법이 있습니다.
call을 사용하면 명시적으로 this를 바인딩하여 문제를 해결할 수 있습니다.

```ts
const c = new C();
const method = c.logSquares;
method.call(c);
```

<img width="833" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/3906c0e9-c4b6-44df-9476-bc838c944a56"/>

this가 반드시 C의 인스턴스에 바인딩되어야 하는 것은 아니며, 어떤 것이든 바인딩할 수 있습니다.
그러므로 라이브러리들은 API의 일부에서 this의 값을 사용할 수 있게 합니다.
심지어 DOM에서도 this를 바인딩할 수 있습니다.
이벤트 핸들러를 예로 들 수 있습니다.

```js
document.querySelector('input')!.addEventListener('change', function(e){
  console.log(this);      //   이벤트가 발생한 input 엘리먼트를 출력
});
```

this 바인딩은 종종 콜백 함수에서 쓰입니다.
에를 들어, 클래스 내에 onClick 핸들러를 정의한다면 다음처럼 할 수 있습니다.

```ts
class ResetButton{
  render(){
    return makeButton({text:'Reset',onClick: this.onClick});
  }
  onClick(){
    alert(`Reset ${this}`);
  }
}
```

그러나 ResetButton에서 onClick을 호출하면, this 바인딩 문제로 인해 "Reset이 정의되지 않습니다"라는 경고가 뜹니다.
일반적인 해결책은 생성자에서 메서드에 this를 바인딩시키는 것입니다.

```ts
class ResetButton{
  constructor(){
    this.onClick = this.onClick.bind(this);
  }
  render(){
    return makeButton({text:'Reset',onClick: this.onClick});
  }
  onClick(){
    alert(`Reset ${this}`);
  }
}
```

`onClick(){ ... }`은 ResetButton.prototype의 속성을 정의합니다.

> 자바스크립트가 프로토타입 기반 언어라는 것을 잊으면 안 됩니다.
> 또한 전형적인 객체 지향 언어들의 클래스와 다르게 자바스크립트의 클래스는 단지 간편 문법(syntactic sugar)에 해당한다는 것도 주의해야 합니다.

그러므로 ResetButton의 모든 인스턴스에서 공유됩니댜.
그러나 생성자에서 `this.onClick = ...`으로 바인딩하면, onClick 속성에 this가 바인딩되어 해당 인스턴스에 생성됩니다.
속성 탐색 순서 (lookup sequence)에서 onClick 인스턴스 속성은 onClick 프로토타입 (prototype) 속성보다 앞에 놓이므로, render() 메서드의 this.onClick은 바인딩된 함수를 참조하게 됩니다.

조금 더 간단한 방법으로 바인딩을 해결할 수도 있습니다.

```ts
class ResetButton{
  render(){
    return makeButton({text:'Reset',onClick: this.onClick});
  }
  onClick = () => {
    alert(`Reset ${this}`); // "this"가 항상 인스턴스를 참조합니다.
  }
}
```

onClick을 화살표 함수로 바꿨습니다.
화살표 함수로 바꾸면, ResetButton이 생성될 때마다 제대로 바인딩된 this를 가지는 새 함수를 생성하게 됩니다.
이해를 돕기 위해 자바스크립트가 실제로 생성한 코드를 살펴보겠습니다. 

```ts
class ResetButton{
  constructor(){
    var _this = this;
    this.onClick = function(){
      alert("Reset"+_this);
    };
  }
  render(){
    return makeButton({text:'Reset',onClick: this.onClick});
  }
}
```

this 바인딩은 자바스크립트의 동작이기 때문에, 타입스크립트 역시 this 바인딩을 그대로 모델링하게 됩니다.
만약 작성 중인 라이브러리에 this를 사용하는 콜백 함수가 있다면, this 바인딩 문제를 고려해야 합니다.

이 문제는 콜백 함수의 매개변수에 this를 추가하고, 콜백 함수를 call로 호출해서 해결할 수 있습니다.

```ts
function addKeyListener(
  el: HTMLElement,
  fn: (this: HTMLElement, e: KeyboardEvent) => void
){
  el.addEventListener('keydown',e =>{
    fn.call(el,e);
  });
}
```

콜백 함수의 첫 번째 매개변수에 있는 this는 특별하게 처리됩니다.
다음 예제처럼 call을 제거하고 fn을 두개의 매개변수로 호출해 보면 알 수 있습니다. 

```ts
function addKeyListener(
  el: HTMLElement,
  fn: (this: HTMLElement, e: KeyboardEvent) => void
){
  el.addEventListener('keydown',e =>{
    fn(el,e);
  });
}
```

<img width="1172" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/b9114248-f12f-4a99-be31-9f8b7a76cacd"/>

콜백 함수의 매개변수에 this를 추가하면 this 바인딩이 체크되기 때문에 실수를 방지할 수 있습니다. 

```ts
function addKeyListener(
  el: HTMLElement,
  fn: (this: HTMLElement, e: KeyboardEvent) => void
){
  el.addEventListener('keydown',e =>{
    fn(e);
  });
}
```

<img width="1493" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/96810395-5989-4ca2-b329-f68ae92ab73f"/>

또한 라이브러리 사용자가 콜백 함수에서 this를 참조할 수 있고 완전한 타입 안전성도 얻을 수 있습니다.

```ts
declare let el : HTMLElement;
addKeyListener(el, function(e){
  this.innerHTML;  // 정상, 'this'는 HTMLElement 타입
})

```

만약 라이브러리 사용자가 콜백을 화살표 함수로 작성하고 this를 참조하려고 하면 타입스크립트가 문제를 잡아냅니다.

```ts
class Foo{
  registerHandler(el: HTMLElement){
    addKeyListener(el, e => {
      this.innerHTML;
    })
  }
}
```

<img width="1172" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/9e2e6846-5a0d-4f5b-9f84-9cd4bbd18878"/>

this의 사용법은 반드시 기억해야 합니다.
콜백 함수에서 this값을 사용해야 한다면 this는 API의 일부가 되는 것이기 때문에 반드시 타입 선언에 포함해야합니다.


















