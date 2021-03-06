# 함수형 프로그래밍과 ES6+ 예제 코드

## 목차

1. [함수형 자바스크립트 기본기](#함수형-자바스크립트-기본기)
2. [ES6에서의 순회와 이터러블/이터레이터 프로토콜](#es6에서의-순회와-이터러블이터레이터-프로토콜)
3. [제너레이터와 이터레이터](#제너레이터와-이터레이터)
4. [map, filter, reduce](#map-filter-reduce)
5. [코드를 값으로 다루어 표현력 높이기](#코드를-값으로-다루어-표현력-높이기)
6. [장바구니 예제](#장바구니-예제)
7. [지연성 1](#지연성-1)
8. [지연성 2](#지연성-2)
9. [비동기/동시성 프로그래밍 1](#비동기동시성-프로그래밍-1)
10. [비동기/동시성 프로그래밍 2](#비동기동시성-프로그래밍-2)
11. [비동기/동시성 프로그래밍 3](#비동기동시성-프로그래밍-3)

## 함수형 자바스크립트 기본기

### 평가와 일급

**평가**

- 코드가 계산(Evaluation) 되어 값을 만드는 것

**일급**

- 값으로 다룰 수 있다.
- 변수에 담을 수 있다.
- 함수의 인자로 사용될 수 있다.
- 함수의 결과로 사용될 수 있다.

```javascript
const a = 10;
const add10 = a => a + 10;
const r = add10(a);
```

### 일급 함수

- 함수를 값으로 다룰 수 있다.
- 조합성과 추상화의 도구

```javascript
const add5 = a => a + 5;
log(add5);
log(add5(5));

const f1 = () => () => 1;
log(f1());

const f2 = f1();
log(f2);
log(f2());
```

### 고차 함수

함수를 값으로 다루는 함수

두가지 유형의 고차함수가 있음

**함수를 인자로 받아서 실행하는 함수**

```javascript
const apply1 = f => f(1);
const add2 = a => a + 2;
log(apply1(add2));
log(apply1(a => a - 1));
```

```javascript
const times = (f, n) => {
  let i = -1;
  while(++i < n) f(i);
};

times(log, 3);

times(a => log(a + 10), 3);
```

**함수를 만들어 리턴하는 함수(클로저를 만들어 리턴하는 함수)**

```javascript
const addMaker = a => b => a + b;
const add10 = addMaker(10);
log(add10(5));
log(add10(10));
```

**[⬆ 상단으로](#목차)**

## ES6에서의 순회와 이터러블/이터레이터 프로토콜

### 이터러블/이터레이터 프로토콜

- 이터러블: 이터레이터를 리턴하는 [Symbol.iterator]() 를 가진 값
- 이터레이터: { value, done } 객체를 리턴하는 next() 를 가진 값
- 이터러블/이터레이터 프로토콜: 이터러블을 for...of, 전개 연산자 등과 함께 동작하도록한 규약

### 사용자 정의 이터러블

- well-formed iterator는 이터레이터 또한 이터러블하게 구현되어있다.
- 이터레이터는 자기자신(이터레이터)를 반환하는 이터레이터 심볼함수를 갖고있다.

```javascript
 const iterable = {
  [Symbol.iterator]() {
    let i = 3;
    return {
      next() {
        return i == 0 ? { done: true } : { value: i--, done: false };
      },
      [Symbol.iterator]() {
        return this;
      }
    }
  }
};
let iterator = iterable[Symbol.iterator](); // 자신의 이터레이터를 반환

for(const a of iterator) log(a);
```

- 많은 라이브러리와 Web API 또한 이터러블/이터레이터 프로토콜을 따릅니다.
- DOM, Facebook(Immutable JS) 등

### 전개 연산자

- 전개연산자 또한 마찬가지로 이터러블/이터레이터 프로토콜을 따릅니다.

```javascript
const a = [1, 2];
a[Symbol.iterator] = null;
log([...a]); // error
```

**[⬆ 상단으로](#목차)**

## 제너레이터와 이터레이터

### 제너레이터와 이터레이터

- 제너레이터: 이터레이터이자 이터러블을 생성하는 함수
- 제너레이터 함수를 실행하면 이터레이터가 반한됩니다.
- 즉 제너레이터는 well-formed 이터레이터를 쉽게 만들어줌

```javascript
function* gen() {
  yield 1;
  if(false) yield 2;
  yield 3;
};
let iter = gen(); // 이터레이터
```

### 제너레이터의 활용 - odds

```javascript
function* infinity(i = 0) {
  while(true) yield i++;
}

function* limit(l, iter) {
  for(const a of iter) {
    yield a;
    if(a == l) return;
  }
}

function* odds(l) {
  for(const a of limit(l, infinity(1))) {
    if(a % 2) yield a;
  }
}

let iter2 = odds(10);
```

**[⬆ 상단으로](#목차)**

## 이터러블/이터레이터 프로토콜의 활용

### 이터러블 프로토콜을 따른 map의 다형성

- 기존의 Array의 map 메서드는 DOM의 NodeList에서는 동작하지 않습니다.
- 아래 예시 코드로 이터레이터로 동작하게 하여 보다 다형성 높게 map을 사용할 수 있습니다.

**DOM List**

```javascript
  const map = (f, iter) => {
  let res = [];
  for(const a of iter) {
    res.push(f(a));
  }
  return res;
};
```

```javascript
console.log(map(el => el.nodeName, document.querySelectorAll('*')));
```

**제너레이터**

```javascript
function* gen() {
  yield 2;
  if(false) yield 3;
  yield 4;
}

console.log(map(a => a * a, gen()));
```

**Map Object**

```javascript
let m = new Map();
m.set('a', 10);
m.set('b', 20);
console.log(new Map(map(([k, a]) => [k, a * 2], m)));
```

### filter

```javascript
const filter = (f, iter) => {
  let res = [];
  for(const a of iter) {
    if(f(a)) res.push(a);
  }
  return res;
};
```

**기본**

```javascript
const products = [
  { name: '반팔티', price: 15000 },
  { name: '긴팔티', price: 20000 },
  { name: '핸드폰케이스', price: 15000 },
  { name: '후드티', price: 30000 },
  { name: '바지', price: 25000 }
];

console.log(...filter(p => p.price < 20000, products));
console.log(...filter(p => p.price >= 20000, products));
```

**제너레이터**

```javascript
function* gen() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  yield 5;
}

console.log(map(a => a % 2, gen()));
```

### reduce

```javascript
  const reduce = (f, acc, iter) => {
  if(!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  }
  for(const a of iter) {
    acc = f(acc, a);
  }
  return acc;
};

console.log(reduce(add, 0, [1, 2, 3, 4, 5])); //15
```

```javascript
const products = [
  { name: '반팔티', price: 15000 },
  { name: '긴팔티', price: 20000 },
  { name: '핸드폰케이스', price: 15000 },
  { name: '후드티', price: 30000 },
  { name: '바지', price: 25000 }
];

console.log(
  reduce((total_price, product) => total_price + product.price, 0, products)
);
```

### map+filter+reduce 중첩 사용과 함수형 사고

```javascript
const products = [
  { name: '반팔티', price: 15000 },
  { name: '긴팔티', price: 20000 },
  { name: '핸드폰케이스', price: 15000 },
  { name: '후드티', price: 30000 },
  { name: '바지', price: 25000 }
];
const add = (a, b) => a + b;
console.log(reduce(
  add, map(p => p.price, filter(p => p.price < 20000, products))
));

console.log(
  reduce(
    add,
    filter(n => n >= 20000,
      map(p => p.price, products))));
```

**[⬆ 상단으로](#목차)**

## 코드를 값으로 다루어 표현력 높이기

### go

즉시 값을 평가함

```javascript
const go = (...args) => reduce((a, f) => f(a), args);
go(
  1,
  a => a + 10,
  a => a + 100,
  console.log
);
// 111
```

- go 함수는 함수를 전달한 파라미터를 순차적으로 실행한다.
- 첫번째 파라미터는 사용할 값을 전달하며, 두번째 부터는 순차적으로 실행할 함수를 전달한다.
- 내부적으로 reduce 함수를 사용하여 순차적으로 실행하는 함수의 리턴 값을 다음 함수에 전달하며 실행한다.

### pipe

```javascript
const pipe = (f, ...fs) => (...as) => go(f(...as), ...fs);

const sum = pipe(
  (a, b) => a + b,
  a => a + 100,
  a => a + 1000
);

sum(1, 10); // 1111

```

- 파이프 함수는 함수를 리턴하는 함수이다.
- 반환된 함수를 실행하면 go 함수가 실행된다.
- 반환된 함수로 전달하는 파라미터는 pipe 함수의 첫번째 파라미터(함수)의 파라미터로 전달된다.
- 이후 파이프 함수에 2번째 부터 전달한 파라미터가 go 함수로 순차적으로 실행된다.
- 마찬가지로 go함수는 reduce함수이므로 파라미터(함수)가 반환한 값을 다음 파라미터(함수)의 파라미터로 전달한다.

### go를 사용하여 읽기 좋은 코드로 만들기

```javascript
const products = [
  { name: '반팔티', price: 15000 },
  { name: '긴팔티', price: 20000 },
  { name: '핸드폰케이스', price: 15000 },
  { name: '후드티', price: 30000 },
  { name: '바지', price: 25000 }
];
const add = (a, b) => a + b;
console.log(reduce( // 이전 코드
  add, map(p => p.price, filter(p => p.price < 20000, products))
));

const go = (...args) => reduce((a, f) => f(a), fs);

go( // go를 사용한 코드
  products,
  products => map(product => product.price, products),
  products => filter(price => price < 20000, products),
  products => reduce(add, 0, products),
  console.log
)

```

### curry

```javascript
const curry = f => (a, ..._) => _.length ? f(a, ..._) : (..._) => f(a, ..._);

const mult = curry((a, b) => a * b);
mult(2) // (..._) => ((a, b) => a * b)(2, ..._);
mult(2)(3); // 6

const mult3 = mult(3);
mult3(3) // 9;
mult3(4) // 12;
```

- curry 함수는 함수의 인자가 1개만 넘어오면, 함수를 실행하지 않고, 파라미터로 받은 함수를 반환한다.
- 파라미터로 반환한 함수를 재 실행하면 이전에 전달했던 파라미터와 함께 실행한다.

### go+curry를 사용하여 더 읽기 좋은 코드로 만들기

```javascript
const curry = f => (a, ..._) => _.length ? f(a, ..._) : (..._) => f(a, ..._);
const go = (...args) => reduce((a, f) => f(a), args);

const map = curry((f, iter) => {
  const result = [];
  for(const a of iter) {
    result.push(f(a));
  }
  return result;
});

const filter = curry((f, iter) => {
  const result = [];
  for(const a of iter) {
    if(f(a)) result.push(a);
  }
  return result;
});

const reduce = curry((f, acc, iter) => {
  if(!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  }
  for(const a of iter) {
    acc = f(acc, a);
  }
  return acc;
});

// curry 가 적용되기 전
go(
  products,
  products => map(product => product.price, products),
  products => filter(price => price < 20000, products),
  products => reduce(add, 0, products),
  console.log
)

// curry가 적용된 후
go(
  products,
  products => map(product => product.price)(products),
  products => filter(price => price < 20000)(products),
  products => reduce(add)(products),
  console.log
)

// 하나의 인자를 전달하면 다음 인자를 받는 함수를 반환 (최종)
go(
  products,
  map(product => product.price),
  filter(price => price < 20000),
  reduce(add),
  console.log
)
```

### 함수 조합으로 함수 만들기

```javascript
const products = [
  { name: '반팔티', price: 15000 },
  { name: '긴팔티', price: 20000 },
  { name: '핸드폰케이스', price: 15000 },
  { name: '후드티', price: 30000 },
  { name: '바지', price: 25000 }
];
const curry = (f) => (a, ..._) => _.length ? f(a, ..._) : (..._) => f(a, ..._);
const map = curry((f, iter) => {
  const res = [];
  for(const a of iter) {
    res.push(f(a));
  }
  return res;
});

const filter = curry((f, iter) => {
  const res = [];
  for(const a of iter) {
    if(f(a)) {
      res.push(a);
    }
  }
  return res;
});

const reduce = curry((f, acc, iter) => {
  if(!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  }
  for(const a of iter) {
    acc = f(acc, a);
  }
  return acc;
});
const go = (...args) => reduce((a, f) => f(a), args);
const pipe = (f, ...fs) => (...as) => go(f(...as), ...fs);

const add = (a, b) => a + b;
go(
  products,
  filter(product => product.price < 20000),
  map(product => product.price),
  reduce(add),
  console.log
)

// --- 함수 조합
const totalPrice = pipe(
  map(product => product.price),
  reduce(add),
);

go(
  products,
  filter(product => product.price < 20000),
  totalPrice,
  console.log
)

go(
  products,
  filter(product => product.price > 20000),
  totalPrice,
  console.log
)

const baseTotalPrice = predi => pipe(
  filter(predi),
  totalPrice,
)

go(
  products,
  baseTotalPrice(product => product.price < 20000),
  console.log
)

go(
  products,
  baseTotalPrice(product => product.price > 20000),
  console.log
)
```

**[⬆ 상단으로](#목차)**

## 장바구니 예제

### 총 수량, 총 가격

### HTML로 출력하기

**[⬆ 상단으로](#목차)**

## 지연성 1

### range와 느긋한 L.range

### range와 느긋한 L.range 테스트

### take

### 제너레이터/이터레이터 프로토콜로 구현하는 지연 평가

### L.map

### L. filter

### range, map, filter, take, reduce 중첩 사용

### L.range, L.map, L.filter, take 의 평가 순서

### 엄격한 계산과 느긋한 계산의 효율성 비교

### map, filter 계열 함수들이 가지는 결합 법칙

### ES6의 기본 규악을 통해 구현하는 지연 평가의 장점

**[⬆ 상단으로](#목차)**

## 지연성 2

### 결과를 만드는 함수 reduce, take

### queryStr 함수 만들기

### Array.prototype.join 보다 다형성이 높은 join 함수

### take, find

### L.map, L.filter로 map과 filter 만들기

### L.flatten, flatten

### L.flatMap, flatMap

### 2차원 배열 다루기

### 지연성 / 이터러블 중심 프로그래밍 실무적인 코드

**[⬆ 상단으로](#목차)**

## 비동기/동시성 프로그래밍 1

### callback과 Promise

### 비동기를 값으로 만드는 Promise

### 값으로서의 Promise 활용

### 합성 관점에서의 Promise와 모나드

### Kleisli Composition 관점에서의 Promise

### go, pipe, reduce에서 비동기 제어

### promise.then의 중요한 규칙

**[⬆ 상단으로](#목차)**

## 비동기/동시성 프로그래밍 2

### 지연 평가 + Promise - L.map, map, take

### Kleisli Composition - L.filter, filter, nop, take

### reduce에서 nop 지원

### 지연 평가 + Promise의 효율성

### 지연된 함수열을 병렬적으로 평가하기 - C.reduce, C.take 1

### 지연된 함수열을 병렬적으로 평가하기 - C.reduce, C.take 2

### 즉시 병렬적으로 평가하기 - C.map, C.filter

### 즉시, 지연, Promise, 병렬적 조합하기

### 코드 간단히 정리

### Node.js에서 SQL 병렬 평가로 얻은 효율

**[⬆ 상단으로](#목차)**

## 비동기/동시성 프로그래밍 3

### async/await

### Q&A) Array.prototype.map이 있는데 왜 FxJS의 map 함수가 필요한지?

### Q&A) 이제 비동기는 async/await로 제어할 수 있는데 왜 파이프라인이 필요한지?

### Q&A) async/await와 파이프라인을 같이 사용하기도 하는지?

### Q&A) 동기 상황에서 에러 핸들링은 어떻게 해야하는지?

### Q&A) 비동기 상황에서 에러 핸들링은 어떻게 해야하는지?

### Q&A) 동기/비동기 에러 핸들링에서의 파이프라인의 이점은?

**[⬆ 상단으로](#목차)**