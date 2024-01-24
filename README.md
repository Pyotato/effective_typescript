# 📖 ITEM 17 : 변경 관련된 오류 방지를 위해 readonly 사용하기 

다음은 삼각수(triangular nunber, 1, 1+2, 1+2+3...)를 출력하는 코드입니다.

```ts
function arraySum(arr:number[]){
    let sum = 0,num;
    while((num=arr.pop())!==undefined){
        sum += num;
    }
    return sum;
}

function printTriangles(n: number){
  const nums = [];
  for(let i=0; i<n; i++){
    nums.push(i);
    console.log(arraySum(nums));
  }
}

printTriangles(5)
```

코드는 간단합니다. 그러나 실행해 해보면 문제가 발생합니다.

<img width="822" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/fe69d40f-6280-42d6-9fa7-af2c2e808da2"/>

arraySum 이 nums을 변경하지 않는다고 간주해서 문제가 발생했습니다.
이 문제는 다음 코드와 같이 해결할 수 있습니다.

이 함수는 배열 안의 숫자들을 모두 합칩니다.
그런데 계산이 끄타면 원래 배열이 전부 비게 됩니다.
자바스크립트 배열은 내용을 변경할 수 있기 때문에, 타입스크립트에서도 역시 오류 없이 통과하게 됩니다.

오류의 범위를 좁히기 위해 `arraySum`이 배열을 변경하지 않는다는 선언을 해보겠습니다.
`readonly` 접근 제어자를 사용하면 됩니다.

```ts
function arraySum(arr: readonly number[]){
    let sum = 0,num;
    while((num=arr.pop())!==undefined){
        sum += num;
    }
    return sum;
}
```

<img width="625" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/51b52c3d-acbb-4866-b56d-6f07957ab230"/>

이 오류 메시지를 자세히 살펴보겠습니다. 
`readonly number[]`는 '타입'이고, `number[]`와 구분되는 몇 가지 특징이 있습니다. 

- 배열의 요소를 읽을 수 있지만, 쓸 수는 없습니다.
- length를 읽을 수 있지만, 바꿀 수는 없습니다 (배열을 변경함).
- 배열을 변경하는 pop을 비롯한 다른 메서드를 호출할 수 없습니다.

`number[]`는 `readonly number[]`보다 기능이 많기 때문에, `readonly number[]`의 서브타입입니다. 
([아이템 7](https://github.com/Pyotato/effective_typescript/blob/item7/README.md)의 내용을 떠올려 보면 쉽게 이해할 수 있습니다.)
따라서 변경 가능한 배열을 `readonly` 배열에 할당할 수 있습니다. 하지만 그 반대는 불가능합니다. 

```ts
const a: number[] = [1,2,3];
const b: readonly number[] = a;
const c: number[] = b;
```

<img width="1095" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/0600496f-4d80-460c-987b-a07e123a2778"/>

타입 단언문 없이 `readonly` 접근제어자를 제거할 수 있다면 `readonly`는 쓸모 없을 것이므로 여기서 오류가 발생하는 게 이치에 맞습니다.

매개뱐수 `readonly`로 선언하면 다음과 같은 일이 생깁니다.

- 타입스크립트는 매개변수가 함수 내에서 변경이 일어나는지 체크합니다.
- 호출하는 쪽에서는 함수가 매개변수를 변경하지 않는다는 보장을 받게 됩니다.
- 호출하는 쪽에서 함수에 `readonly` 배열을 매개변수로 넣을 수도 있습니다.

자바스크립트에서는 (타입스크립트에서도 마찬가지) 명시적으로 언급하지 않는 한, 함수가 매개변수를 변경하지 않는다고 가정합니다.
그러나 이러한 암묵적인 방법은 타입 체크에 문제를 일으킬 수 있스빈다 (자세한 내용은 [야이템 30](https://github.com/Pyotato/effective_typescript/blob/item30/README.md)과 [아이템 31](https://github.com/Pyotato/effective_typescript/blob/item31/README.md)에서 다룹니다).
명시적인 방법을 사용하는 것이 컴파일러와 사람 모두에게 좋습니다.

앞 예제의 arraySum을 고치는 방법은 간단합니다.
배열을 변경하지 않으면 됩니다.

```ts
function arraySum(arr: readonly number[]){
    let sum = 0;
    for(const num of arr){
        sum += num;
    }
    return sum;
}
```

이제 printTriangles이 제대로 동작합니다.

<img width="668" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/cce95166-bdda-4107-8de6-fe135d6aede6"/>

만약 함수가 매개변수를 변경하지 않는다면, `readonly`로 선언해야 합니다.
더 넓은 타입으로 호출할 수 있고 ([아이템 29](https://github.com/Pyotato/effective_typescript/blob/item29/README.md)), 의도치 않은 변경은 방지될 것입니다.
이로 인한 단점은 상대적으로 적습니다.

굳이 찾아보자면 매개변수가 `readonly`로 선언되지 않은 함수를 호출해야 할 경우도 있다는 것입니다.
만약 함수가 매개변수를 변경하지 않고도 제어가 가능하다면 `readonly`로 선언하면 됩니다.
그런데 어떤 함수를 `readonly`로 만들면, 그 함수를 호출하는 다른 함수 모두 `readonly`로 만들어야 합니다.
그러면 인터페이스를 명확히 하고 타입 안전성을 높일 수 있기 때문에 꼭 단점이라고 볼 순 없습니다.
그러나 다른 라이브러리에 있는 함수를 호출하는 경우라면, 타입 선언을 바꿀 수 없으므로 타입 단언문 (`as number[]`)을 사용해야 합니다.

`readonly`를 사용하면 지역 변수와 관련된 모든 종류의 변경 오류를 방지할 수 있습니다. 
예를 들어 소설(novel)에 등장하는 처리를 하는 프로그램을 만든다고 가정해 보겠습니다.
연속된 행을 가져와서 빈 줄을 기준으로 구분되는 단락으로 나누는 기능을 하는 프로그램입니다.

> Frankenstein; or, The Modern Prometheus <br/>
> by Mary Shelley <br/>
> You will rejoice to hear that no diaster has accompanied the commencement of an enterprise which you have regarded with such evil forebodings. I arrived here yesterdat, and my first task is to assure my dear sister of my welfare and increasing confidence in the success of my undertaking.<br/><br/>
> I am already far north of London, and as I walk in the streets of Petersburgh, I feel a cold northern breeze play upon my cheeks, which braces my nerves and fills me with delight.

다음처럼 구현해 볼 수 있습니다.

> 실제로는 `lines.join('\n').split(\/n/n+/)`로만 작성하면 됩니다.

```ts
function parseTaggedText(lines: string[]): string[][]{
  const paragraphs : string[][] = [];
  const currPara: string[] = [];

  const addParagraph = () => {
    if(currPara.length){
      paragraphs.push(currPara);
      currPara.length =  0;   // 배열을 비움
    }
  };

  for(const line of lines){
    if(!line){
      addParagraph();
    } else {
      currPara.push(line);
    }
  }
  addParagraph();
  return paragraphs;
}
```

앞의 소설을 입력을 넣고 실행하면, 다음과 같이 출력됩니다.

`[[],[],[]]`

완전히 잘못되었습니다.

이 코드의 문제점은 별칭([아이템 24](https://github.com/Pyotato/effective_typescript/blob/item24/README.md))과 변경을 동시에 사용해 발생했습니다.
별칭은 다음 행에서 발생합니다.

`paragraphs.push(currPara)`

currPara의 내용이 삽입되지 않고 배열의 참조가 삽입되었습니다.
currPara에 새 값을 채우거나 지운다면 동일한 객체를 참조하고 있는 paragraphs요소에도 변경이 반영됩니다.

결국 다음 코드가 문제의 핵심입니다.

```ts
paragraphs.push(currPara);
currPara.length = 0; // 배열을 비움
```

이 코드는 새 단락을 paragraphs에 삽입하고 바로 지워 버립니다.
문제는 currPara.length를 수정하고 currPara.push를 호출하면 둘 다 currPara 배열을 변경한다는 점입니다.
currPara를 `readonly`로 선언하여 이런 동작을 방지할 수 있습니다.
선언을 바꾸는 즉시 코드 내에서 몇 가지 오류가 발생하게 됩니다.

```ts
function parseTaggedText(lines: string[]): string[][]{
  const currPara: readonly string [] = [];
  const paragraphs : string[][] = [];

  const addParagraph = () => {
    if(currPara.length){
     paragraphs.push(currPara);
    }
    currPara.length = 0;
  };

  for(const line of lines){
    if(!line){
      addParagraph();
    } else {
      currPara.push(line);
    }
  }
  addParagraph();
  return paragraphs;
}
```

<img width="1375" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/45963ab3-9ad2-471e-97cd-d7edd7224577"/>

currPara를 let으로 선언하고 변환이 없는 메서드를 사용함으로써 두 개의 오류를 고칠 수 있습니다.

```ts
function parseTaggedText(lines: string[]): string[][]{
  let currPara: readonly string [] = [];
  const paragraphs : string[][] = [];

  const addParagraph = () => {
    if(currPara.length){
     paragraphs.push(currPara);
    }
   currPara = []; //  배열을 비움
  };

  for(const line of lines){
    if(!line){
      addParagraph();    
    } else {
      currPara = currPara.concat([line]);
    }
  }
  addParagraph();
  return paragraphs;
}
```
<img width="1253" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/1639c81c-f5dd-4151-a21d-5911a42cb300"/>

push와 달리 concat은 원본을 수정하지 않고 새 배열을 반환합니다.
선언부를 const에서 let으로 바꾸고 `readonly`를 추가함으로써 한쪽이 변경 가능성을 또 다른 쪽으로 옮긴 것입니다.
currPara 변수는 이제 가리키는 배열을 자유롭게 변경할 수 잇지만, 그 배열 자체는 변경하지 못하게 됩니다.

여전히 paragraphs에 대한 오류는 남아 있습니다.
이 오류를 바로 잡는 방법은 세 가지 입니다.

첫 번째, currPara의 복사본을 만드는 방법입니다.

```ts
function parseTaggedText(lines: string[]): string[][]{
  let currPara: readonly string [] = [];
  const paragraphs : string[][] = [];

  const addParagraph = () => {
    if(currPara.length){
     paragraphs.push([...currPara]); // 🌟 수정
    }
   currPara = []; //  배열을 비움
  };

  for(const line of lines){
    if(!line){
      addParagraph();    
    } else {
      currPara = currPara.concat([line]);
    }
  }
  addParagraph();
  return paragraphs;
}

```

currPara는 `readonly`로 유지되지만, 복사본은 원하는 대로 변경이 가능하기 때문에 오류는 사라집니다.

두 번째, paragraphs(그리고 함수의 반환 타입)를 `readonly String[]`의 배열로 변경하는 방법입니다.

```ts
function parseTaggedText(lines: string[]): (readonly string[])[] { // 🌟 수정
  let currPara: readonly string [] = [];
  const paragraphs : (readonly string[])[] = []; // 🌟 수정

  const addParagraph = () => {
    if(currPara.length){
     paragraphs.push(currPara);
    }
   currPara = []; //  배열을 비움
  };

  for(const line of lines){
    if(!line){
      addParagraph();    
    } else {
      currPara = currPara.concat([line]);
    }
  }
  addParagraph();
  return paragraphs;
}
```

(여기서 괄호가 중요한데, `readonly string[][]`은 `readonly` 배열의 변경 가능한 배열이 아니라 변경 가능한 배열의 readonly 배열이기 때문입니다.)

앞의 코드는 동작하지만 parseTaggedText의 사용자에게는 조금 불친절하게 느껴질 겁니다.
이미 함수가 반환한 값에 대해 영향을 끼치는 것이 맞는 방법인지 고민해 봐야 합니다.

세 번째, 배열의 `readonly` 속성을 제거하기 위해 단언문을 쓰는 방법입니다.

```ts
function parseTaggedText(lines: string[]): string[][] { // 🌟 수정
  let currPara: readonly string [] = [];
  const paragraphs : string[][] = [];

  const addParagraph = () => {
    if(currPara.length){
     paragraphs.push(currPara as string[]);  // 🌟 수정
    }
   currPara = []; //  배열을 비움
  };

  for(const line of lines){
    if(!line){
      addParagraph();    
    } else {
      currPara = currPara.concat([line]);
    }
  }
  addParagraph();
  return paragraphs;
}

```

바로 다음 문장에서 currPara를 새 배열에 할당하므로, 매우 공격적인 단언문처럼 보이지는 않습니다.

`readonly`는 얕게(shallow) 동작한다는 것에 유의하며 사용해야 합니다.
앞에서 이미 `readonly string[][]`을 봤습니다.
만약 객체의 `readonly` 배열이 있다면, 그 객체 자체는 `readonly`가 아닙니다.

```ts
const dates: readonly Date[] = [new Date()];
dates.push(new Date());

dates[0].setFullYear(2037);  // 정상
```

<img width="1028" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/81afbbfe-e0df-492f-8558-b857fe34e4ce"/>

비슷한 경우가 `readonly`의 사촌 격이자 객체에 사용되는 `Readonly` 제너릭에도 해당됩니다.

```ts
interface Outer{
  inner: {
    x:  number;
  }
}

const o: Readonly <Outer> = {inner: {x: 0}};
o.inner = {x: 1};
o.inner.x = 1;
```

<img width="1047" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/77846a17-baaa-4992-8283-42c161812674"/>

타입 별칭을 만든 다음에 정확히 무슨 일이 일어나는지 편집기에서 살펴볼 수 있습니다.

```ts
type T = Readonly<Outer>;
// Type T = {
//   readonly inner: {
//     x: number;
//   }
// }
```

중요한 점은 `readonly` 접근제어자는 inner에 적용되는 것이지 x는 아니라는 것입니다. 
현재 시점에는 깊은 (deep) readonly 타입이 기본으로 지원되지 않지만, 제너릭을 만들면 깊은 `readonly`타입을 사용할 수 있습니다. 
그러나 제너릭은 만들기 까다롭기 때문에 라이브러리를 사용하는 게 낫습니다.
예를 들어 ts-essentials에 있는 DeepReadonly 제너릭을 사용하면 됩니다.

```ts
let obj: {readonly [k: string] : number} = {};
// 또는 Readonly<{[k: string]: number}>
obj.hi = 45;
obj = {...obj,hi: 12};
obj = {...obj,bye: 34};
```

 <img width="1222" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/78a52638-46bc-41b7-babe-ec42890fceed"/>

이 코드처럼 인덱스 시그니처에 `readonly`를 사용하면 객체의 속성이 변경되는 것을 방지할 수 있습니다.

