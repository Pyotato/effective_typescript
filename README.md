# 🕵️ ITEM 56: 정보를 감추는 목적으로 private 사용하지 않기

자바스크립트는 클래스에 비공개 속성을 만들 수 없습니다.
많은 이가 비공개 속성임을 나타내기 위해 언더스코어(_) 접두가로 붙이던 것이 관례로 인정될 뿐이었습니다.

```ts
class Foo{
  _private = 'secret123';
}
```

그러나 속성에 언더스코어를 붙이는 것은 단순히 비공개라고 표시한 것뿐입니다.
따라서 일반적인 속성과 동일하게 클래스 외부로 공개되어 있다는 점을 주의해야 합니다.

```ts
const f = new Foo();
f._private ; // 'secret123';

```

타입스크립트에는 public, protected, private 접근 제어자를 사용해서 공개 규칙을 강제할 수 있는 것으로 오해할 수 있습니다.

```ts
class Diary{
  private secret = 'cheated on my English test';
}
const diary = new Diary();
console.log(diary.secret);
```

<img width="804" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/0c578378-171b-49c5-b5cf-a322fe373693"/>

그러나 public, protected, private 같은 접근 제어자는 타입스크립트 키워드이기 때문에 컴파일 후에는 제거됩니다([아이템 3](https://github.com/Pyotato/effective_typescript/blob/item3/README.md)). 
이 타입스크립트 코드를 컴파일하게 되면 다음 예제의 자바스크립트 코드로 변환됩니다 (target=ES2017이 설정된 상태).

<img width="805" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/484e7ed2-18e3-49f7-8a46-3427b93a18e0"/>

즉, 정보를 감추기 위해 private을 사용하면 안됩니다.

자바스크립트에서 정보를 숨기기 위해 가장 효율적인 방법은 클로저(closure)를 사용하는 것입니다.
다음 코드처럼 생성자에서 클로저를 만들어 낼 수 있습니다.

```ts
declare function hash(text: string): number;

class PasswordChecker{
  checkPassword: (password: string)=>boolean;
  constructor(passwordHash: number){
    this.checkPassword = (password: string) =>{
      return hash(password) === passwordHash;
    }
  }
}

const checker = new PasswordChecker(hash('s3cret'));
checker.checkPassword('s3cret');                      // true
```

앞의 코드를 살펴보면 PasswordChecker의 생성자 외부에서 passwordHash 변수에 접근할 수 없기 때문에 정보를 숨기는 목적을 달성했습니다.
그런데 몇 가지 주의 사항이 있습니다.
passwordHash를 생성자 외부에서 접근할 수 없기 때문에, passwordHash에 접근해야 하는 메서드 역시 생성자 내부에 정의되어야 합니다.
그리고 메서드 정의가 생성자 내부에 존재하게 되면, 인스턴스를 생성할 때마다 각 메서드의 복사본이 생성되기 때문에 메모리를 낭비하게 된다는 것을 기억해야 합니다. 
또한 동일한 클래스로부터 생성된 인스턴스라고 하더라도 서로의 비공개 데이터에 접근하는 것이 불가능하기 떄문에 철저하게 비공개이면서 동시에 불편함이 따릅니다.

> 일반적인 객체지향 언어에서는 동일한 클래스의 개별 인스턴스끼리 private 속성 접근이 가능합니다 (클래스 단위 비공개). <br/>
> 그러나 클로저 방식은 동일 클래스의 개별 인스턴스 간에 속성 접근이 불가능 (인스턴스 단위 비공개)하기 때문에 불편하다는 의미입니다.

또 하나의 선택지로, 현재 표준화가 진행 중인 비공개 필드 기능을 사용할 수도 있습니다. ([2024 기준 등록됨](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Private_properties))
비공개 필드 기능은 접두사로 `#`를 붙여서 타입 체크와 런타임 모두에서 비공개로 만드는 역할을 합니다.

```ts
class PasswordChecker{
  #passwordHash: number;

  constructor(passwordHash: number){
  this.#passwordHash = passwordHash;
  }

  checkPassword(password:string){
    return hash(password) === this.#passwordHash;
  }
}

const checker = new PasswordChecker(hash('s3cret'));
console.log(checker.checkPassword('s3cret'));  //false
console.log(checker.checkPassword('secret'));  //true
```

`#passwordHash` 속성은 클래스 외부에서 접근할 수 없습니다.
그러나 클로저 기법과 다르게 클래스 메서드나 동일한 클래스의 개별 인스턴스끼리는 접근이 가능합니다.
비공개 필드를 지원하지 않는 자바스크립트 버전으로 컴파일하게 되면, WeapMap을 사용한 구현으로 대체됩니다.
어쨋든 구현 방식과 무관하게 데이터는 동일하게 비공개입니다.
2021년 기준으로 비공개 필드는 자바스크립트 표준화 3단계이고, 타입스크립트에서 사용 가능합니다.

> 비공개 필드는 2020년 2월20일에 릴리스된 타입스크립트 3.8부터 지원됩니다.

만약 설계 관점의 캡슐화가 아닌 '보안'에 대해 걱정하고 있다면, 내장된 프로토타입과 함수에 대한 변조 같은 문제를 알고 있어야 합니다.
