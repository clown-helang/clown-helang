---
title: 删除数组中的重复项
catalog: true
date: 2022-03-20 21:17:22
subtitle: 算法练习
header-img: '/img/article_header/article_header.png'
tags: Arithmetic
---

## 题目

一个有序数组 nums，原地删除重复出现的元素，使每个元素只出现一次，返回删除后数组的新长度。不能使用额外的数组空间，必须在原地的修改输入数组并在使用 O(1)额外空间的条件下完成。

例：

​ 输入：[0,1,2,2,3,3,4]

​ 输出：5

## 解法：双指针算法

数组完成排序后，我们可以放置两个指针`i`和`j`，其中`i`使慢指针，而`j`是快指针。只要`nums[i]=nums[j]`，我们就增加`j`以跳过重复项。

当遇到`nums[j]!= nums[i]`时，跳过重复项的运行已经结束，必须把`nums[j]`的值复制到`nums[i+1]`。然后递增`i`，接着将再次重复相同的过程，直到`j`到达数组的末尾为止。代码如下所示：

```js
function removeDuplicates(nums) {
  if (!nums.length) return 0;
  let i = 0;
  for (let j = 1; j < nums.length; j++) {
    if (nums[j] !== nums[i]) {
      i++;
      nums[i] = nums[j];
    }
  }
  return i + 1;
}
```
