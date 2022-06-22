---
title: JavaScript 원하는 길이만큼 난수 반환하기
author: sunshine@ptokos.com
categories: [javascript]
---

JavaScript 에서 원하는 길이만큼 난수를 반환하는 함수를 만들어 보려합니다.

우선 난수를 만들 수 있어야하는데 해당 코드는 [mozilla doc](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Math/random) 에서 가져왔습니다.

```javascript
function getRandomInt(min, max) {
  min = Math.ceil(min);
  max = Math.floor(max);
  return Math.floor(Math.random() * (max - min)) + min; //최댓값은 제외, 최솟값은 포함
}
```
 
위 함수를 사용하면 난수 하나를 반환하는데 만들고자 하는 상황은 예를 들어 난수 10개가 합쳐진 문자열을 반환하게 됩니다.

테스트 케이스에서 전화번호를 입력해야하는데 무작위 0~10 난수로 이루어진 숫자로 테스트 케이스를 작성이 적합하다고 판단되었습니다.

그리하여 반환될 길이를 의미하는 outputLength 라는 인자를 추가하였습니다.
### 최종 코드
```javascript
function getRandomInt(min, max, outputLength = 1) {
  min = Math.ceil(min);
  max = Math.floor(max);
  return [...Array(outputLength)].map((_) => Math.floor(Math.random() * (max - min)) + min).join('');
}
```

### 시행 착오
간단하지만 약간의 시행착오를 겪으면서 알게된 것이 있어 포스팅을 하게되었습니다.  
처음에 작성했던 코드는 아래와 같습니다.
```javascript
return new Array(outputLength).map((_) => Math.floor(Math.random() * (max - min)) + min);
```

위 코드를 실행시키면 undefined 로 구성된 배열이 반환됩니다. 
이해가 가지 않아 찾아본 바로는 map 함수는 각 요소를 순서대로 콜백 함수를 호출하여 그 결과를 반환하고 새 배열을 구성하는데 삭제 또는 값이 할당된 적 없는 인덱스는 호출이 되지 않는 것 같습니다.

재할당 되지 않은 배열은 map 이 콜백 함수를 호출하지 않음을 코드로 확인해볼 수 있습니다.
```javascript
console.log(new Array(10).map((_) => 'a'));
console.log([...Array(10)].map((_) => 'a'));

/*
[ <10 empty items> ]
[
  'a', 'a', 'a', 'a',
  'a', 'a', 'a', 'a',
  'a', 'a'
]

 */
```

해결 방법으로는 apply 를 통해 배열을 넘겨주어 할당하는 것입니다.
```javascript
console.log(Array.apply(null, Array(10)).map((_) => 'a'));
/*
[
  'a', 'a', 'a', 'a',
  'a', 'a', 'a', 'a',
  'a', 'a'
]
 */
```

원리는 같지만 더 나은 표현으로는 spread 연산자를 사용하는 것입니다.
```javascript
console.log([...Array(10)].map((_) => 'a'));
/*
[
  'a', 'a', 'a', 'a',
  'a', 'a', 'a', 'a',
  'a', 'a'
]
 */
```
 
