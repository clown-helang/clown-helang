---
title: 素数个数统计
catalog: true
date: 2022-03-19 22:50:41
subtitle: 算法练习
header-img: '/img/article_header/article_header.png'
tags: Arithmetic
---

## 题目

统计 N 以内的素数个数（素数：只能被 1 和自身整除的自然数，0、1 除外）

## 方法一：暴力解法

再不考虑时间复杂度的情况下，可以使用逐一判断的方式暴力计算素数个数，具体代码如下所示：

```js
/*
 * 计数
 */
function count(n) {
  let result = 0;

  for (let i = 2; i < n; i++) {
    result += isPrime(i) ? 1 : 0;
  }

  return result;
}

/*
 * 判断是否是素数
 */
function isPrime(n) {
  for (let i = 2; i * i <= n; i++) {
    if (n % i == 0) {
      return false;
    }
  }
  return true;
}
```

## 方法二：埃氏筛选

统计素数的个数，换个角度思考，其实也就是统计合数（非素数）的个数。所以可以将代码优化如下：

> 合数：合数就是除了 1 和本身之外至少还有一个约数的数字。

```js
function count(n) {
  const isPrime = new Array(n);

  let count = 0;

  for (let i = 2; i < n; i++) {
    if (!isPrime[i]) {
      count++;
      for (let j = i * i; j < n; j += i) {
        isPrime[j] = true;
      }
    }
  }

  return count;
}
```
