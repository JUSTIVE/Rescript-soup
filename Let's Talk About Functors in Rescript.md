# 리스크립트의 펑터에 대해 이야기해봅시다.

## 개요

[reference](https://dusty.phillips.codes/2021/09/18/lets-talk-about-functors-in-rescript/)

함수형 프로그래밍은 다른 패러다임과 기본적인 문법에서 큰 차이를 보이지 않는 것처럼 느껴집니다.
물론, 데이터와 행위가 구분되고, 이에 따라서 클래스나 객체나 상속을 할 필요가 없지만, 그래도 상대적으로는 똑같게 느껴집니다.
이는 특히 메서드 참조와 거의 비슷하게 생긴(파이썬의 self 객체와 비교했을 때) pipe-first 메서드가 있는 리스크립트에서 사실입니다.

하지만 함수형 프로그래밍에 깊에 들어가게 된다면, `모나드` 나 `펑터` 와 같은 괴이한 용어들을 마주하기 시작합니다.
역시, 순수주의보다 실용주의를 강조하는 리스크립트에서는 덜 사실입니다.
실제로, 리스크립트 문서에서 `모나드`를 검색한다면 빈 결과가 나오지만, `펑터`를 검색한다면 매우 짧은 섹션을 볼 수 있습니다.

펑터는 리스크립트의 핵심 기능이 아니지만, 명확히 설명할 수 있을 정도로 이해하고 싶은 유용한 추상입니다.

## 먼저, 제너릭 타입에 대해 생각해 봅시다.

리스크립트 문서를 읽다 보면, 펑터를 다음과 같이 소개하고 있습니다 : `모듈을 인자로 받고 모듈을 반환하는 함수`.
맨 처음 이것을 읽게 되었을 때 클래스를 인자로 받고 약간 수정된 다른 버전의 클래스를 반환하는 파이썬의 클래스 데코레이터와 유사한 것으로 생각했었습니다.

하지만 깊게 파고들수록, 펑터는 타입에 관련된 것이란 것을 알게됩니다.

따라서 펑터에 대해 생각하기 위해 우선 타입에 대해 생각해봅시다.
리스크립트는 이미 제너릭 타입의 개념을 가지고 있습니다.
우리가 그들을 사용할 때에, 예를 들어 제너릭 콜렉션(`array<string>`,`array<int>`)이 있고, 다음과 같이 우리만의 제너릭 레코드를 만들 수 있습니다.

```re
type keyValuePair<'a,'b> = {
  name:'a,
  value:'b
}
```

이 타입은 `'a` 와 `'b`로 명명된 두 타입에 걸쳐 제너릭합니다.
우리는 이 제너릭 타입으로부터 무한한 구체적인 타입들을 만들 수 있습니다. 
예를 들어, `string`과 `integer`를 받는 버전의 타입을 다음과 같이 인스턴스화 할 수 있습니다.

```re
type hundredsOfApples: keyValuePair<string,int>={
  name:"apple",
  value:480
}
```
또한 이 타입으로부터 다른 구체적인 타입을 만들 수 있습니다.

```re
type twoStrings = keyValuePair<string,string>
```

그리고 기존에 존재하는 제너릭 타입으로부터 새 제너릭 타입을 만들 수 있습니다.
아래는 오직 한 개의 타입 인자를 받습니다.

```re
type stringValuePair = keyValuePair<string,'a>
```

위에서 교육적인 목적으로 `'a`를 남용한 것을 주목하세요.
이 `'a` 는 `type keyValuePair<'a,'b>`의 `'a`와는 완전히 다른 것입니다.
사실, 이 `'a`는 `keyValuePair`의 `'b`에 대입되었습니다!
이 두 사례에서 `'a`는 그냥 `지금 제너릭이기때문에 지금 내가 당장 신경쓰고 싶지 않은 타입`을 의미합니다.

## 함수를 넣고 섞기

제너릭 타입을 함수에 넣고 제너릭 타입을 반환을 받을 수도 있습니다.
예를 들어, 다음은 유효한 리스크립트 함수입니다.
```re
let extractValue: keyValuePair<'a,'b> => 'b = pair => pair.value
```
이것도 마찬가지입니다.
```re
let valueIfName : (keyValuePair<string,'b>,string)=>option<'b> = (pair,nameToMatch)=> {
  if pair.name == nameToMatch {
    Some(pair.value)
  } else {
    None
  }
}
```
위의 예제에서, 반환 값이 새 제너릭 타입인 것에 주목하세요.
그리고 이 값은 반드시 pair에서 `'b`로 정의된 `value` 타입과 같은 타입이어야 합니다.

리스크립트에서는 인자가 제너릭인 제너릭 함수를 다음과 같이 만들 수 있습니다.

```re
let showAndReturn: 'a => 'a = argument => {
  Js.log(argument)
  argument
}
```
이 함수는 __아무 타입__ 이나 인자로 받아도 동작할 것입니다.
반면, 이 함수를 특별화시킬 방법이 없습니다. 
예를 들어, 아래는 유효한 __타입스크립트__ 입니다.

```typescript
//Rescript 가 아니라 Typescript 입니다.
function showAndReturn<atype>(argument:atype):atype{
  console.log(argument)
  return argument
}

let m = showAndReturn<string>("hello")
```
리스크립트보다 타입스크립트의 장점은 다음과 같이 컴파일 타임에 오류가 발생하도록 호출할 때에 타입을 특수화시킬 수 있는 것입니다.
```typescript
//여전히 Typescript입니다. 컴파일되지 않습니다.
let m = showAndReturn<number>("hello");
```
아는 한에서 리스크립트에 이에 대응되는 개념은 없지만, `펑터`(뒤에서 다룰겁니다. 약속!)는 제너릭 타이핑을 제공하는 하나의 방법입니다.

## 데이터베이스에 대해 생각해 봅시다.

