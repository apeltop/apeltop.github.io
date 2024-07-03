---
title: 개행이 안되고 한 줄로 표시되는 문제 해결하기
author: sunshine@ptokos.com
categories: [ css, html, javascript ]
---

## 상황

사용자가 개행을 해서 입력을 했으나 화면에는 한 줄로 표시되는 문제가 발생했다.
실제로는 아래와 같이 입력이 되었다.

> a
>
> b
>
> c

![css-prewrap-1.png](/assets/img/css/css-prewrap-1.png)

로그를 찍어보면 개행으로 출력이 된다.

![css-prewrap-2.png](/assets/img/css/css-prewrap-2.png)

## 해결

CSS 에서 `white-space: pre-wrap;` 속성을 사용하면 개행이 되어 출력된다.

![css-prewrap-3.png](/assets/img/css/css-prewrap-3.png)

## CSS 의 white-space: pre-wrap; 속성 이해하기

### white-space 속성이란?

`white-space` 속성은 요소 내의 공백 문자를 어떻게 처리할지 결정하는 CSS 속성입니다. 이 속성은 여러 가지 값을 가질 수 있는데, 그 중 `pre-wrap`은 특히 유용한 옵션입니다.

### pre-wrap 의 의미

- `pre`: "preserve"의 약자로, 원본 텍스트의 형식을 "보존"한다는 의미입니다.
- `wrap`: 긴 줄은 자동으로 줄바꿈됩니다.

### white-space 속성 종류

| 값          | 설명                               |
|------------|----------------------------------|
| `normal`   | 공백을 축소하고 자동 줄바꿈합니다.              |
| `nowrap`   | 모든 줄바꿈을 무시하고 한 줄로 표시합니다.         |
| `pre`      | 공백과 줄바꿈을 유지하지만, 자동 줄바꿈은 하지 않습니다. |
| `pre-line` | 줄바꿈은 유지하지만, 연속된 공백은 하나로 축소합니다.   |
| `pre-wrap` | 모든 공백과 줄바꿈을 유지하며, 필요시 자동 줄바꿈합니다. |

