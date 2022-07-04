---
title: 논리와 명제
layout: doc
subtitle: 이산수학 논리와 명제
author: sunshine@ptokos.com
tags: [math]
---

이산수학 각 파트별 개념들을 간단히 요약해서 정리해보려합니다.

책은 4차 산업혁명 시대의 이산수학을 보고 공부했습니다.
![Alt text](/assets/img/discrete-mathematics/1.png)

## 명제 (proposition)
> 어떤 사고를 나타내는 문장 중에서 참이나 거짓을 객관적이고 명확하게 구분할 수 있는 문장이나 수학적 식을 말한다.

## 명제 논리 (propositional logic)
> 주어와 술어를 구분하지 않고 전체를 하나의 식으로 처리하여 참 또는 거짓을 판별하는 법칙을 다룬다.

## 술어 논리 (predicate logic)
> 술어 논리는 주어와 술어로 구분하여 참 또는 거짓에 관한 법칙을 다룬다.

## 단순 명제 (simple proposition)
> 하나의 문장이나 식으로 구성되어 있는 명제

## 합성 명제 (composition proposition)
> 여러 개의 단순 명제들이 논리 연산자들로 연결되어 만들어진 명제

## 논리 연산자 (logical operators)
> $\bigvee$, $\bigwedge$, $\sim$ 와 같은 연산자

## 부정 (negation)
> 임의의 명제 $p$ 가 주어졌을 때 그 명제에 대한 부정은 명제 $p$ 의 반대되는 진리값을 가진다. 

## 논리곱 (conjunction)
> 임의의 두 명제 $p, q$ 가 그리고(AND) 로 연결되어 있을 때 명제 $p, q$ 의 논리곱은 $p \bigwedge q$ 로 표시한다.

## 논리합 (disjunction)
> 임의의 두 명제 $p,q$ 가 또는(OR) 으로 연결되어 있을 때 명제 $p,q$ 의 논리합은 $p \bigvee q$ 로 표시한다.

## 배타적 논리합 (Exclusive OR)
> 임의의 두 명제 $p,q$의 배타적 논리합은 $ \bigoplus $ 로 표시한다. 
> 배타적 논리합의 진리값은 p 와 q 두 명제 중에서 하나의 명제가 참이고 하나의 명제가 거짓일 때 참의 진리값을 가지고 그렇지 않으면 거짓의 진리값을 가진다.

## 조건
> 조건 연산자를 함축(implication)이라고도 한다.
> 임의의 명제 $p,q$ 의 조건 연산자는 $p \to q$ 로 표시한다.

조건 연산자 $ \to $ 는 다양한 표현으로 나타내어진다.
1. $ p $ 이면 $ q $이다
2. $ p $ 는 $ q $의 충분조건이다
3. $ q $ 는 $ p $의 필요조건이다
4. $ p $ 는 $ q $를 함축한다

## 쌍방 조건
> 임의의 명제 $p,q$ 의 쌍방 조건은 $p \leftrightarrow q$ 로 표시한다.
> 쌍방 조건의 진리값은 $p,q$ 가 모두 참이거나 거짓일 때 참의 값을 가지고 그 외에는 거짓을 가진다.

쌍방 조건은 다양한 표헌으로 나타내어진다.
1. $p$ 이면 $q$ 이고, $q$ 이면 $p$ 이다.
2. $p$ 는 $q$ 의 필요충분조건이다.

## 항진 명제 (tautology)
> 합성 명제에서 그 명제를 구성하는 단순 명제들의 진리값에 관계없이 그 합성 명제의 진리 값이 항상 참의 값을 가지는 명제를 의미한다.

## 모순 명제 (contradiction)
> 합성 명제에서 그 명제를 구성하는 단순 명제들의 진리값에 관계없이 그 합성 명제의 진리 값이 항상 거짓을 가지는 명제를 의미한다.

## 논리적 동치 (logical equivalence)
> 두 개의 명제 $p,q$ 의 쌍방 조건 $p \leftrightarrow q$ 가 항진 명제이면,
> 두 명제 $p,q$ 는 논리적 동치라고 하고, $p \equiv q$ 또는 $p \Leftrightarrow q $ 라고 표시한다.
> 즉 명제 $p$와 $q$는 같은 논리값을 가진다는 의미이다.

## 논리적 동치 관계의 기본 법칙
### 멱등 법칙 (idempotent law)
$p \vee p \Leftrightarrow p $

$p \wedge p \Leftrightarrow p $

### 항등 법칙 (identity law)
$p \vee T \Leftrightarrow T $

$p \vee F \Leftrightarrow p $

$p \wedge T \Leftrightarrow p $

$p \wedge F \Leftrightarrow F $

### 부정 법칙 (negation law)
$ \sim T \Leftrightarrow F $

$ \sim F \Leftrightarrow T $

$p \vee (\sim p) \Leftrightarrow T $

$p \wedge (\sim p) \Leftrightarrow F $

### 이중 부정 법칙 (double negation law)
$ \sim(\sim p) \Leftrightarrow p$

### 교환 법칙 (commutative law)
$ p \vee q \Leftrightarrow q \vee p$

$ p \wedge q \Leftrightarrow q \wedge p $

$ p \leftrightarrow q \Leftrightarrow q \leftrightarrow p $

### 결합 법칙 (associative law)
$(p \vee q) \vee r \Leftrightarrow p \vee (q \vee r) $


$(p \wedge q) \wedge r \Leftrightarrow p \wedge (q \wedge r) $

### 분배 법칙 (distributive law)
$ p \vee (q \wedge r) \Leftrightarrow (p \vee q) \wedge (p \vee r) $

$ p \wedge (q \vee r) \Leftrightarrow (p \wedge q) \vee (p \wedge r) $

### 흡수 법칙 (absorption law)
$ p \vee (p \wedge q) \Leftrightarrow p $

$ p \wedge (p \vee q) \Leftrightarrow p $

### 드 모르간 법칙 (De Morgan's law)
$ \sim(p \vee q) \Leftrightarrow (\sim p) \wedge (\sim q) $

$ \sim(p \wedge q) \Leftrightarrow (\sim p) \vee (\sim q) $

### 조건 법칙
$ p \to q \Leftrightarrow \sim p \vee q $

### 대우 법칙
$ p \to q \Leftrightarrow \sim q \to \sim p $

## 추론 (argument)
> 주어진 명제가 참인 것을 바탕으로 새로운 명제가 참이 되는 것을 유도해내는 방법
> 전제를 $p_1 \dotsm p_N$ 이라고 하고 결론을 $q$ 라고 할 때 추론에 대한 것을 수학적 식으로 표시하면 $p_1,p_2,\dotsc , p_n \vdash q$ 라고 표시한다.

### 유효 추론 (valid argument)
> 주어진 전제가 참이고 결론도 참인 추론

### 허위 추론 (fallacious argument)
> 주어진 전제가 참이고 결론이 거짓인 추론





