---
emoji: β²οΈ
title: setTimeout(), setImmedidate(), process.nextTick()
date: '2022-02-06 17:00:00'
author: reviday
tags: blog javscript node.js
categories: javascript node.js
---

# π― λͺ©ν
λμμ μ§μ°μν€λ κ²μΌλ‘λ§ μκ³  μμλ μ΄ μΈ κ°μ§ ν¨μ ```setTimeout()```, ```setImmedidate()```, ```process.nextTick()```μ λν΄ μμλ³΄κ³  μ€ν μμ μ λν μ°¨μ΄λ₯Ό κ°λ¨νκ²λλ§ μμλ³΄κ³ μ νλ€.

<br />

# π κ°λ
[Node.js κ³΅μ λ¬Έμ](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)λ₯Ό νμΈνλ©΄ μ μλ μλμ κ°κ³ , λλ¦λλ‘ κ°λμ μ λ¦¬(<span style="color:grey">~~μ§μ§κΈ°~~</span>)ν΄λ³΄μλ€.

<br />

## 1. setTimeout()
> ```setTimeout()```Β schedules a script to be run after a minimum threshold in ms has elapsed.
- <strong style="color:rgba(217, 115, 13, 1);">μ§μ λ μκ°(ms) μ΄ν</strong>μ ν¨μ λλ μ§μ λ μ½λλ₯Ό μ€ννλ νμ΄λ¨Έλ₯Ό μ€μ νλ€.

<br />

## 2. setImmedidate()
> ```setImmediate()```Β is designed to execute a script once the currentΒ pollΒ phase completes.
- <strong style="color:rgba(217, 115, 13, 1);">νμ¬ ν΄λ§</strong>Β λ¨κ³κ° μλ£λλ©΄Β μ€νλλ€.
- μμμ λΉλκΈ°μ μΌλ‘ μ€ννλ€λ λ©΄μμλ process.nextTick()κ³Ό μ μ¬νμ§λ§, setImmedidate()λ μ΄λ―Έ νμ μλ <strong style="color:rgba(217, 115, 13, 1);">I/O μ΄λ²€νΈλ€μ λ€μ λκΈ°</strong>νκ² λλ€λ μ°¨μ΄κ° μλ€.

<br />

## 3. process.nextTick()
> ```process.nextTick()```Β fires immediately on the same phase.
- νμ¬ μ§ν μ€μΈ μμμ μλ£ μμ  λ€λ‘ ν¨μμ μ€νμ μ§μ°μν¨λ€(μ¦, κ°μ phase λ΄μμ μ¦μ μ€νλλ€).
- ν΄λΉ ν¨μλ‘ μ§μ°λ μ½λ°±μ <strong style="color:rgba(144, 101, 176, 1);">λ§μ΄ν¬λ‘ νμ€ν¬</strong>λΌκ³  λΆλ¦¬λ©°, μ΄λ―Έ νμ μλ <strong style="color:rgba(217, 115, 13, 1);">I/O μ΄λ²€νΈλ€ μμ μμΉ</strong>νκ² λμ΄ νμ¬μ μμμ΄ μλ£λ νμ λ°λ‘ μ€νλλ€.

<br />

# β³ κ²°λ‘ λΆν° κ°λ¨νκ²
1. μΌλ°μ μΌλ‘ ```setTimeout(cb, 0)``` < ```setImmediate()``` < ```process.nextTick()``` μμΌλ‘ λ¨Όμ  μ€νλλ€.
2. ```setTimeout(cb, 0)```, ```setImmediate()```μ μ€νμμλ <strong style="color:rgba(217, 115, 13, 1);">I/O μ¬μ΄ν΄ λ° νλ‘μΈμ€ μ±λ₯</strong>μ λ°λΌ λ³ν  μ μλ€.  
3. ```process.nextTick()```λ₯Ό μ¬μ©νλ©΄ νμ¬ μμ λ°λ‘ λ€λ‘ μ§μ°λλ©°, ν΄λΉ ν¨μλ‘ μ§μ°λ μ½λ°±μ <strong style="color:rgba(144, 101, 176, 1);">λ§μ΄ν¬λ‘ νμ€ν¬</strong>λΌκ³  λΆλ¦°λ€. 

<br />

# π§π»βπ» μ½λλ‘ νμΈν΄λ³΄μ

<br />

## 1οΈβ£ setTimeout(func, delay)μμ delayκ° 0msλΌκ³  0ms νμ μ€νλμ§ μλλ€.
Node.js μμ€μ [node/lib/timers.js](https://github.com/nodejs/node/blob/master/lib/timers.js#L140)λ₯Ό μ΄ν΄λ³΄λ©΄ ```setTimeout()``` ν¨μλ₯Ό νμΈν  μ μκ³ ,
```javascript
function setTimeout(callback, after, arg1, arg2, arg3) {

	...

  const timeout = new Timeout(callback, after, args, false, true);
  insert(timeout, timeout._idleTimeout);

  return timeout;
}
```
μ΄κ³³μμ μ¬μ©λ  [Timeout()](https://github.com/nodejs/node/blob/b2edcfee46097fe8e0510a455b97d5c6d0cac5ec/lib/internal/timers.js#L167)μ μ μλ₯Ό νμΈν΄λ³΄λ©΄ λ€μκ³Ό κ°μ΄ μ μλμ΄ μλ€.
```javascript
function Timeout(callback, after, args, isRepeat, isRefed) {
  after *= 1; // Coalesce to number or NaN
  if (!(after >= 1 && after <= TIMEOUT_MAX)) {
    if (after > TIMEOUT_MAX) {
      process.emitWarning(`${after} does not fit into` +
                          ' a 32-bit signed integer.' +
                          '\nTimeout duration was set to 1.',
                          'TimeoutOverflowWarning');
    }
    after = 1; // Schedule on next tick, follows browser behavior
  }

...

}
```
μ¦, <strong style="color:rgba(217, 115, 13, 1);">delayλ₯Ό 0msμΌλ‘ μ€μ νμ¬λ 1msλ‘ μ€μ λμ΄ λμ</strong>νλ€.

<br />

## 2οΈβ£ setTimeout() vs setImmedidate() 
<strong style="color:rgba(217, 115, 13, 1);">I/O μ¬μ΄ν΄ λ΄μ μμ§ μμ</strong> μλ μ€ν¬λ¦½νΈλ₯Ό μ€ννκ² λλ©΄ λ νμ΄λ¨Έκ° μ€νλλ μμλ <strong style="color:rgba(217, 115, 13, 1);">νλ‘μΈμ€μ μ±λ₯</strong>μ μν΄ μ νλκΈ° λλ¬Έμ <strong style="color:rgba(217, 115, 13, 1);">κ²°κ³Όκ° μΌμ νμ§ μλ€.</strong> κ·Έ μ΄μ λ μμμ μ€λͺν λ΄μ©(<a href="#-κ°λ" style="color:grey">**κ°λ**</a>)μ μλ€. 

```javascript
setTimeout(() => {
  console.log('timeout');
}, 0);

setImmediate(() => {
  console.log('immediate');
});

// Output 
timeout
immediate

// Output 
immediate
timeout
```
νμ§λ§ <strong style="color:rgba(217, 115, 13, 1);">I/O μ¬μ΄ν΄ λ΄</strong>μ μμΉνκ² λλ©΄ <strong style="color:rgba(217, 115, 13, 1);">ν­μ setImmediate()κ° λ¨Όμ  μ€ν</strong>λλ€.
```javascript
const fs = require('fs');

fs.readFile(__filename, () => {
  setTimeout(() => {
    console.log('timeout');
  }, 0);
  setImmediate(() => {
    console.log('immediate');
  });
});

// Output 
immediate
timeout
```

<br />

## 3οΈβ£ setTimeout() vs setImmedidate() 
μ΄λ―Έ κ°λ μ€λͺ λΆλΆμμ μ°¨μ΄λ₯Ό μ€λͺνμκΈ°μ λκ° λ λΉ λ₯Έ λμμ λ³΄μΌμ§ μλͺνμ§λ§, μ½λλ‘ νμΈν΄λ³΄κΈ° μνμ¬ [μ€νμ€λ²νλ‘μ°](https://stackoverflow.com/questions/17502948/nexttick-vs-setimmediate-visual-explanation)μμ κ°μ Έμ¨ μμλ₯Ό λ€μ΄ μ΄ν΄λ³΄λλ‘ νμ.

### setImmediate() 
```javascript
setImmediate(function A() {
  setImmediate(function B() {
    console.log(1);
    setImmediate(function D() { console.log(2); });
    setImmediate(function E() { console.log(3); });
  });
  setImmediate(function C() {
    console.log(4);
    setImmediate(function F() { console.log(5); });
    setImmediate(function G() { console.log(6); });
  });
});

setTimeout(function timeout() {
  console.log('TIMEOUT FIRED');
}, 0);

/* μμμ μΈκΈνλ€ μνΌ, I/O μ¬μ΄ν΄ λ΄μ μμ§ μμΌλ―λ‘ μ€νν  λλ§λ€ κ²°κ³Όκ° λ¬λΌμ§ κ²μ΄λ€. */
// TIMEOUT FIRED 1 4 2 3 5 6
// OR
// 1 4 TIMEOUT FIRED 2 3 5 6
```

### process.nextTick()
```nextTick()``` μ½λ°±μ ν­μ νμ¬ μ½λ μ€νμ΄ μλ£λ μ§νμ μ΄λ²€νΈ λ£¨νλ‘ λμκ°κΈ° μ μ μ€νλκΈ° λλ¬Έμ μ΄λ²€νΈ λ£¨ν μ΄νμ μ€νλλ ```setTimeout()```μ μ½λ°±μ ν­μ λ§μ§λ§μ μΆλ ₯λλ€. κ°λ Ή, ```setTimeout()``` μ½λμ μμΉλ₯Ό μλ‘ μ¬λ¦°λ€κ³  νμ¬λ κ²°κ³Όλ ν­μ κ°λ€.
```javascript
process.nextTick(function A() {
  process.nextTick(function B() {
    console.log(1);
    process.nextTick(function D() { console.log(2); });
    process.nextTick(function E() { console.log(3); });
  });
  process.nextTick(function C() {
    console.log(4);
    process.nextTick(function F() { console.log(5); });
    process.nextTick(function G() { console.log(6); });
  });
});

setTimeout(function timeout() {
  console.log('TIMEOUT FIRED');
}, 0);

/* ν­μ nextTick()μ κ²°κ³Όκ° λ¨Όμ  μΆλ ₯λλ€. */
// 1 4 2 3 5 6 TIMEOUT FIRED
```

μ λΉκ΅λ‘ μ μ μλ€ μνΌ, ```setTimeout()``` κΈ°μ€μΌλ‘ ν­μ κ·Έ μμμ μ€νλλ ```process.nextTick()```μ ```setImmediate()``` λ³΄λ€ λ¨Όμ  μ€νλλ€λ κ²μ μ μ μλ€.


<br />

# μ°Έκ³ ν μ¬μ΄νΈ
1. [Node.js κ³΅μ λ¬Έμ](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)
2. [StackOverFlow](https://stackoverflow.com/questions/17502948/nexttick-vs-setimmediate-visual-explanation)
3. [[NodeJS] Event-Loop Part 2 : setTimeout() vs setImmediate() vs process.nextTick()](https://medium.com/@rpf5573/nodejs-event-loop-part-2-settimeout-vs-setimmediate-vs-process-nexttick-70ba2a9f0895) - **Byeongin Yoon**

<p style="color:grey">κ·Έ μΈ μ°Έκ³ ν λΈλ‘κ·Έλ€μ΄ λ μλλ°, λΈμμ μ λ¦¬νλ λΆλΆμ λΈλ‘κ·Έ κΈλ‘ μμ±ν΄λ³Έ κ²μ΄μ¬μ μΆμ²κ° λ¨μμμ§ μλ€μ. π­π­π­ </p>