---
title: 二叉树的遍历 - 非递归版Java实现
date: 2018-04-30 20:07:29
categories: 
	- 算法
tags: 
    - 数据结构与算法
    - Java
---
## TreeNode 的定义

``` java
    public class TreeNode {
        int val;
        TreeNode left;
        TreeNode right;
        public TreeNode(int val) {
            this.val = val;
        }
    }

```

<!--more-->


二叉树的后序遍历
-----------------------

> 主要思想：首先遍历root根节点的所有左节点，并依次入栈。对出栈的元素，如果没有右儿子或者虽然有右儿子但右儿子已完成遍历，即可完成出栈；否则，再次入栈，并把右儿子入栈，遍历右儿子的所有左儿子。


### 版本一


``` java

import java.util.ArrayList;
import java.util.Stack;
public class Solution {
    public ArrayList<Integer> postorderTraversal(TreeNode root) {
        Stack<TreeNode> stack = new Stack<TreeNode>();
        TreeNode cur = root;
        /* 用来记录最新出栈的节点，
         * 如果当前节点的右儿子与flag相同，说明当前节点右子树已完成遍历
         */
        TreeNode flag = null;
        ArrayList<Integer> ans = new ArrayList<Integer>(20);
        while (cur != null) {
            stack.push(cur);
            cur = cur.left;
        }
        while (!stack.isEmpty()) {
            cur = stack.pop();
            if (cur.right == null || cur.right == flag) {
                ans.add(cur.val);
                flag = cur;
            }
            else {
                stack.push(cur);
                cur = cur.right;
                while(cur != null) {
                    stack.push(cur);
                    cur = cur.left;
                }
            }
        }
        return ans;
    }
}
```
### 版本二
``` java

import java.util.ArrayList;
import java.util.Stack;
public class Solution {
    public ArrayList<Integer> postorderTraversal(TreeNode root) {
        Stack<TreeNode> stack = new Stack<TreeNode>();
        TreeNode cur = root;
        TreeNode prior = null;
        ArrayList<Integer> ans = new ArrayList<Integer>(20);
        while (cur != null || !stack.isEmpty()) {
            if (cur != null) {
                stack.push(cur);
                cur = cur.left;
            } else {
                cur = stack.pop();
                if (cur.right == null || cur.right == prior) {
                    ans.add(cur.val);
                    prior = cur;
                    cur = null;
                }
                else {
                    stack.push(cur);
                    cur = cur.right;
                    stack.push(cur);
                    cur = cur.left;
                }
            }
        }
        return ans;
    }
}
```

二叉树的先序遍历
-----------------------

> 先对root根节点入栈，然后根节点出栈，将根节点的右儿子先入栈，再将根节点的左儿子入栈。

``` java
import java.util.ArrayList;
import java.util.Stack;
public class Solution {
    public ArrayList<Integer> preorderTraversal(TreeNode root) {

        Stack<TreeNode> stack = new Stack<TreeNode>();
        ArrayList<Integer> ans = new ArrayList<Integer>(20);
        if (root == null) {
            return ans;
        }
        TreeNode cur = root;
        stack.push(cur);
        while (!stack.isEmpty()) {
            cur = stack.pop();
            ans.add(cur.val);
            if (cur.right != null) {
                stack.push(cur.right);
            }
            if (cur.left != null) {
                stack.push(cur.left);
            }
        }
        return ans;
    }
}
```

二叉树的中序遍历
-----------------------

> cur == null 说明没有左子树（叶子节点）或者左子树已经完成遍历，可以开始遍历右子树了。
> 在"二叉树的后序遍历"所给出的版本二中有类似的用法。

``` java
    public ArrayList<Integer> inorderTraversal(TreeNode root) {
        TreeNode cur = root;
        Stack<TreeNode> stack = new Stack<TreeNode>();
        ArrayList<Integer> ans = new ArrayList<Integer>(20);
        while (cur != null || !stack.isEmpty()) {
            // 相当于一个开关
            if (cur != null) {
                stack.push(cur);
                cur = cur.left;
            }
            else {
                cur = stack.pop();
                ans.add(cur.val);
                if (cur.right == null) {
                    cur = null;
                }
                else {
                    cur = cur.right;
                    stack.push(cur);
                    cur = cur.left;
                }
            }
        }
        return ans;
    }
```

二叉树的层序遍历
------------------------

>基于队列（Queue）实现的自上而下、从左到右按层遍历二叉树
>
>记录每一层节点的个数n，并出列n次，出列的同时把子结点压入队列。
>这样就可以遍历完一层，同时读入下一层。
``` java
   public ArrayList<Integer> levelTraversal(TreeNode root) {
        if (root == null) {
            return null;
        }
        // 使用双向链表（LinkedList）来表示队列（Queue）
        Queue<TreeNode> queue = new LinkedList<TreeNode>();
        ArrayList<Integer> ans = new ArrayList<Integer>(20);
        TreeNode cur = root;
        queue.offer(cur);
        while(!queue.isEmpty()) {
            // size 记录当前层的节点数，用于完整遍历当前层节点
            for (int i = 0, size = queue.size(); i < size; i++){
                cur = queue.poll();
                ans.add(cur.val);
                // 先入栈left，就是自上而下，从左到右遍历
                // 先入栈right，就是从右到左遍历
                if (cur.left != null) {
                    queue.offer(cur.left);
                }
                if (cur.right != null) {
                    queue.offer(cur.right);
                }
            }
        }
        return ans;
    }
```



