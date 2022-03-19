---
title: 反转链表
catalog: true
date: 2022-03-13 22:19:09
subtitle: 算法练习
header-img: '/img/article_header/article_header.png'
tags: Arithmetic
---

## 题目

将链表的顺序翻转过来

例：

- 输入：1 -> 2 -> 3 -> 4 -> 5
- 输出：5 -> 4 -> 3 -> 2 -> 1

使用两种方式解题

## 解题

### 方法一：迭代

```js
function reverse(head) {
  let prev = null,
    next;
  let curr = head;

  while (curr) {
    next = curr.next;
    curr.next = prev;
    prev = curr;
    curr = next;
  }

  return prev;
}
```

### 方法二：递归

```js
function reverse(head) {
  let result = null;
  function recursion(node) {
    if (node === null || node.next === null) {
      result = node;
      return node;
    }
    let prevNode = recursion(node.next);

    prevNode.next = node;
    node.next = null;

    return node;
  }
  recursion(head);
  return result;
}
```
