---
title:  hashmap 1.8 源码理解
date: 2019-03-31 09:19:58
tags:
---

###  hashmap 1.8 源码理解

<!--more-->

关于 hashmap 1.8 源码理解

1. hashmap 1.7 使用 数组+链表     1.8 使用： 数组+链表+红黑树

​       为了解决哈希碰撞后，链表过长从而导致索引效率慢的问题 时间复杂度从O（N）-O（logn）

1. 当链表长度大于8的时候，将链表转换为红黑树，当resize时，红黑树不足6，转换为链表，插入结点后，判断是否hashmap大小>容量* 加载因子
2. 一般来说， 容量（capacity）：必须是2的幂，并且最大容量为（2的30次方）

​      加载因子（Load factor)：HashMap在其容量自动增加前可达到多满的一种尺度

​     扩容阙值（threshold）：当哈希表的大小 大于等于扩容阙值时，就会扩容hashmap, 从而使哈希表将大约具有两倍桶数   扩容阙值= 容量 x 加载因子

1. 关于hash(key)  ：扰动：加大哈希码低位的随机性，使得分布更均匀，从而提高对应数组存储下标位置的随机性 & 均匀性，最终减少Hash冲突

​    1.7：进行hashCode()+4次为运算+5次异或

​    1.8： hashCode（）+1次位运算+1次异或    h^(h>>>16)高位参与低位运算

​      因为直接hashcode() 的映射是 -（2^31）~2^31-1 而hashmap的容量范围最小为16-2^30,而且一般不会使用最大值，不需要这么多的存储空间以及难以提供这么多空间。

然后使用 h&(length-1)：将哈希码与数组的长度取模，如果为奇数，那么会浪费一般空间，所以要求数组必须为2的幂，而且求余速度慢，所以使用了& 运算

1. 1.7：扩容时，与put一样，每个数据都需要重新h&(length-1)  （头插法）

​      1.8:  原位置或者 原位置+就容量    （尾插法）

 1.8扩容机制： 扩容后容量=旧容量*2  若hash值新增参与位运算的位=0，那么扩容后还是0.则为原来位置，若为1，则扩容后为原始位置+旧容量

1. get 数据： 数组-红黑树-链表（put操作 红黑树也优先于链表）

2. 关于String、Interger这样的包装类适合作为Key，因为这些包装类的特性是final类型，则具有不可变性，保证了key的不可更改性，不会出现哈希码不同的状态，内部重写了equals  hashcode方法，不容易出现hash值的计算错误

   ​    https://blog.csdn.net/carson_ho/article/details/79373134







关于Morris算法解析：遍历二叉树，空间O(1)

根据中序排序的思想：找到当前结点的前序结点，则当前结点左子树的最右结点如果左孩子没有右子树，则左孩子为前序结点，如果没有左孩子也没有右孩子，则为第一个结点

Morris遍历算法的步骤如下：

1， 根据当前节点，找到其前序节点，如果前序节点的右孩子是空，那么把前序节点的右孩子指向当前节点，然后进入当前节点的左孩子。

2， 如果当前节点的左孩子为空，打印当前节点，然后进入右孩子。

3，如果当前节点的前序节点其右孩子指向了它本身，那么把前序节点的右孩子设置为空，打印当前节点，然后进入右孩子。

```java
public class MorrisTraval {
    private TreeNode root = null;
    public MorrisTraval(TreeNode r) {
        this.root = r;
    }
    
    public void travel() {
        TreeNode n = this.root;
        //如果当前左孩子为空，打印，如果前继等于当前，打印
        while (n != null) {
            if (n.left == null) {
                System.out.print(n.vaule + " ");
                n = n.right;
            } else {
                TreeNode pre = getPredecessor(n);
                
                if (pre.right == null) {
                    pre.right = n;
                    n = n.left;
                }else if (pre.right == n) {
                    pre.right = null;
                    System.out.print(n.vaule + " ");
                    n = n.right;
                }
                
            }
        }
    }
    //查询前序结点
    private TreeNode getPredecessor(TreeNode n) {
        TreeNode pre = n;
        if (n.left != null) {
            pre = pre.left;
            while (pre.right != null && pre.right != n) {
                pre = pre.right;
            }
        }
        
        return pre;
    }
    
}

```



