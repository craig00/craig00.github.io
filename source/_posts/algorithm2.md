---
title: algorithm---2
date: 2017-06-27 14:32:51
categories: algorithm
tags: [algorithm]
---
#### 复杂链表的复制

输入一个复杂链表（每个节点中有节点值，以及两个指针，一个指向下一个节点，另一个特殊指针指向任意一个节点），返回结果为复制后复杂链表的head。（注意，输出结果中请不要返回参数中的节点引用，否则判题程序会直接返回空）

题目里的特殊指针指向的任意节点，依然是这条链上的节点。这一点居然没搞明白！

所以思路就是先把所有节点复制一遍，并插入源节点之后，形成链表，然后处理random节点（需要结合前面说的），最后拆分一下，拆出一条也是OK滴。

```javascript
/*function RandomListNode(x){
    this.label = x;
    this.next = null;
    this.random = null;
}*/
function Clone(pHead) {
    // write code here
    if (!pHead) return null;
    let current = pHead;
    while (current) {
        let node = new RandomListNode(current.label);
        node.next = current.next;
        current.next = node;
        current = node.next;
    }
    current = pHead;
    while (current) {
        node = current.next;
        if (current.random) {
            node.random = current.random.next;
        }
        current = node.next;
    }
    let copyHead = pHead.next;
    let temp;
    current = pHead;
    while (current.next) {
        temp = current.next;
        current.next = temp.next;
        current = temp;
    }
    return copyHead;
}
```

#### 二叉搜索树与双向链表

输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的双向链表。要求不能创建任何新的结点，只能调整树中结点指针的指向。

核心思路是利用中序遍历，中序遍历正好符合排序要求。

```javascript
/* function TreeNode(x) {
    this.val = x;
    this.left = null;
    this.right = null;
} */
//非递归版
function Convert(pRootOfTree)
{
    // write code here
    if(!pRootOfTree) return null;
    let pre;
    let node = pRootOfTree;
    let stack = [];
    let start = 0;
    let root;
    while(node != null || stack.length != 0){
        while(node != null){
            stack.push(node);
            node = node.left;
        }
        let pop = stack.pop();
        if (start == 0) {
            start++;
            root = pop;
            pre = pop;
        }else{
            pre.right = pop;
            pop.left = pre;
            pre = pop;
        }
        node = pop.right;
    }
    return root;
}
//递归版
function Convert(pRootOfTree) {
    // write code here
    if (!pRootOfTree) return null;
    //不依靠外部变量
    if (pRootOfTree.left == null && pRootOfTree.right == null)
        return pRootOfTree;
    let left = Convert(pRootOfTree.left); //中序遍历
    let p = left;
    while (p && p.right != null) {//取到左子树的最大值
        p = p.right;
    }
    if (left != null) {
        p.right = pRootOfTree;
        pRootOfTree.left = p;
    }
    let right = Convert(pRootOfTree.right);
    if (right != null) {
        right.left = pRootOfTree;
        pRootOfTree.right = right;
    }
    return left != null ? left : pRootOfTree;
}
//简单递归版
//依赖外部变量
let leftN = null,rightN = null;
function Convert(pRootOfTree) {
    // write code here
    if (!pRootOfTree) return null;
    Convert(pRootOfTree.left);
    if (rightN == null) {
        leftN = rightN = pRootOfTree;
    }else{
        rightN.right = pRootOfTree;
        pRootOfTree.left = rightN;
        rightN = pRootOfTree;
    }
    Convert(pRootOfTree.right);
    return leftN;
}
```