---
title: algorithm---1
date: 2017-05-07 15:32:51
categories: algorithm
tags: [algorithm]
---
###  主定理（MasterTheorem）

设a≥1，b>1为常数，f(n) 为函数，T(n) 为非负整数，且T(n) = aT(n/b) + f(n) 则有以下结果：
![主定理](http://omoi0oliz.bkt.clouddn.com/algorithm.png)

###  找出n个数中第K大的数

采用类似快速排序算法的思想求解，从数组中随机选择一个数x，将数组分为两部分，一部分都小于x，另一部分都大于等于x.这个数x所在的位置正好是n-K的话，则返回这个数x，不然就继续找。

以下是java代码
``` java
import java.util.*;

public class Finder {
    public int findKth(int[] a, int n, int K) {
        // write code here
        int i = 0, j = n - 1;
        int re = findHelp(a, i, j);
        int temp = a[re];
        a[re] = a[i];
        a[i] = temp;
        while ((n - re) != K) {
            if (K > (n - re)) {
                j = re - 1;
                re = findHelp(a, i, j);
                temp = a[re];
                a[re] = a[i];
                a[i] = temp;
            } else {
                i = re + 1;
                re = findHelp(a, i, j);
                temp = a[re];
                a[re] = a[i];
                a[i] = temp;
            }
        }
        return a[re];
    }

    public int findHelp(int[] a, int i, int j) {
        if (i >= j) return j;
        int cc = a[i];
        while (i < j) {
            while (i < j && a[j] > cc) {
                j--;
            }
            while (i < j && a[i] <= cc) {
                i++;
            }
            int temp = a[i];
            a[i] = a[j];
            a[j] = temp;
        }
        return j;
    }
}

```

以下是javascript代码
``` javascript
function findKth(array, K) {
  let n = array.length;
  let i = 0,
    j = n - 1;
  let re = findHelp(array, i, j);
  [array[i], array[re]] = [array[re], array[i]];
  while ((n - re) != K) {
    if (K > (n - re)) {
      j = re - 1;
      re = findHelp(array, i, j);
      [array[i], array[re]] = [array[re], array[i]];
    } else {
      i = re + 1;
      re = findHelp(array, i, j);
      [array[i], array[re]] = [array[re], array[i]];
    }

  }
  return array[re];

  function findHelp(arr, i, j) {
    if (i >= j) return j;
    let cc = arr[i];
    while (i < j) {
      while (i < j && arr[j] > cc) {
        j--;
      }
      while (i < j && arr[i] <= cc) {
        i++;
      }
      [arr[i], arr[j]] = [arr[j], arr[i]];
    }
    return j;
  }
}
```

###  时间复杂度

根据这个算法的过程可以得到如下等式

>T(n) = T(n/2) + O(n)

根据主定理可知，T(n) = O(n).

同样也可以利用堆排序来做，虽然时间复杂度会是O(nKlogK);
```javascript
function findKth(array, K) {
  let cc = array.slice(0, K);
  cc.sort((a, b) => a - b);
  for (var i = K; i < array.length; i++) {
    if (array[i] > cc[0]) {
      cc[0] = array[i];
      for (var j = cc.length - 1; j > 0; j--) {
        cc = adjustHeap(cc, j);
        [cc[j], cc[0]] = [cc[0], cc[j]];
      }
    }
  }
  console.log(cc[0]);

  function adjustHeap(arr, j) {
    if (j == 0) return arr;
    let end = parseInt((j - 1) / 2);
    for (let i = end; i >= 0; i--) {
      if (arr[2 * i + 1] && arr[2 * i + 1] > arr[i]) {
        [arr[2 * i + 1], arr[i]] = [arr[i], arr[2 * i + 1]];
      }
      if ((2 * i + 2) <= j && arr[2 * i + 2] && arr[2 * i + 2] > arr[i]) {
        [arr[2 * i + 2], arr[i]] = [arr[i], arr[2 * i + 2]];
      }
    }
    return arr;
  }

}
```