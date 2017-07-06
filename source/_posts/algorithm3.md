---
title: algorithm---3
date: 2017-07-06 23:10:51
categories: algorithm
tags: [algorithm]
---
### 字符串的字典序全排列

输入一个字符串,按字典序打印出该字符串中字符的所有排列。例如输入字符串abc,则打印出由字符a,b,c所能排列出来的所有字符串abc,acb,bac,bca,cab和cba。 （可能有字符重复）

Array.sort方法重新理解：传入的比较函数有点纯函数的意味---compareFunction(a, b) must always return the same value when given a specific pair of elements a and b as its two arguments.

另外sort是根据返回的结果result进行排序，如果result<0，就升序，result>0，就降序，result==0就按默认的规则排序。

所以这一题的思路没什么特殊，就是取全排列然后再排序。

```javascript
function Permutation(str) {
    // write code here
    if (!str) return "";
    let arr = str.split("");
    let cc = new Set();
    // arr.sort((a,b)=> a.charCodeAt(0) - b.charCodeAt(0));
    recurse(cc, arr, 0);
    return [...cc].sort();
}
function recurse(cc, arr, num) {
    if (num == arr.length) {
        cc.add(arr.join(""));
        return;
    }
    for (let i = num; i < arr.length; i++) {
        [arr[i], arr[num]] = [arr[num], arr[i]];
        recurse(cc, arr, num + 1);
        [arr[i], arr[num]] = [arr[num], arr[i]];
    }
}
```

### 最大子段和问题

```javascript
function maxSum(arr) {
    let sum = 0, a = 0;
    for (let i = 0; i < arr.length; i++) {
        if (a > 0) {
            a = a + arr[i];
        } else {
            a = arr[i]; //这里可以记录开始位置
        }
        if (a > sum) {
            sum = a; //这里可以记录结束位置
        }
    }
    return sum;
}
```