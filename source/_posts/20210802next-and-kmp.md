---
title: next数组与KMP算法
date: 2021-08-02 09:50:45
tags:
- Algorithm
---

## next数组

### 定义

list：任意类型的数组
k-prefix：list的前k个字符
k-suffix：list的后k个字符

next数组与list长度一致
next[i]表示list[0...i]中使k-prefix等于k-suffix的最大k

例: list = abcabcd
list[0...0] = a，next[0] = 0,
list[0...1] = ab，next[1] = 0,
list[0...2] = abc，next[2] = 0,
list[0...3] = abca，1-prefix=a=1-suffix, next[3] = 1,
list[0...4] = abcab，2-prefix=ab=2-suffix, next[4] = 2,
list[0...5] = abcabc，3-prefix=abc=3-suffix, next[5] = 3,
list[0...6] = abcabcd，next[6] = 0,

所以 abcabcd 的next数组为 [0,0,0,1,2,3,0]

### 快速求next数组

例如 abcabcdabcabca
假设当前list为[0...j]，检查的前缀元素为list[i]，当前检查的后缀元素为list[j], 已知next[j-1], 求next[j]

如果list[i]=list[j], 则匹配的前缀长度为i+1, next[j] = i+1, 同时i++，j++

如果list[i]!=list[j]，则我们需要重新选择前缀元素开始匹配，暴力计算的话我们需要把i放到0重新开始匹配前缀，但是如果i前面的部分字符匹配上了，我们可以节约一些时间。

之前检查的前缀为list[0...i]，如果i前面的n个元素和list[0...n]相同，那么说明0-n的元素已经匹配成功了，我们只需要从n+1的元素开始匹配即可。那如果获取n呢？

已知list[0...n]=list[i-n...i-1]，相当于list[0...i-1]的n-prefix=n-suffix，所以n=next[i-1]。

所以当list[i]!=list[j]
如果i > 0，我们选取第n+1个元素list[n]与list[i]匹配，i=n=next[i-1]
如果i = 0, 说明i前面没有元素可以做再次匹配了，我们需要移动j，匹配list[j+1]与list[i]

该算法求next数组的时间复杂度为O(N), N为list长度

代码实现如下
```java
public static int[] getNext(int[] list) {
    int[] next = new int[list.length];
    next[0] = 0;
    int i = 0;
    int j = 1;
    while(j < list.length) {
        if(list[i] == list[j]) {
            i++;
            next[j] = i;
            j++;
        } else if(i > 0){
            i = next[i-1];
        } else {
            j++;
        }
    }

    return next;
}
```

### 通过next数组求list的最小重复子串

已知list的长度为N，N-next[N-1] = X
如果N%X == 0，则最小重复子串为list[0...X-1]
否则最小重复子串为list本身

## KMP字符串匹配

已知list S和list P，可以用KMP算法快速在S中寻找是否存在P或者P的前缀，时间复杂度为O(M+N)，M，N分别为S，P的长度
整体逻辑和求next数组类似

已知P的next数组，假设当前检查位置为S[i]和P[j]

当S[i]=P[j]，则匹配成功，i++，j++，如果j此时等于P的长度N，说明在S中找到P
当S[i]!=P[j]
如果j>0, 向前移动n个元素，j=next[j-1]再次开始匹配
如果j=0, 无法移动j，则移动i，i++

我们可以通过kmp来查找S中P的位置，代码如下
```java
public static void KMPSearch(int[] S, int[] P) {
    int[] next = getNext(P);

    int i = 0;
    int j = 0;
    while(i < S.length) {
        if(S[i] == P[j]) {
            i++;
            j++;
            if(j == P.length) {
                System.out.println("Found pattern at index " + (i - j));
                j = next[j-1];
            }
        } else if(j > 0){
            j = next[j-1];
        } else {
            i++;
        }
    }
}
```

## 其他实现

### next数组head加入-1

next数组的长度为N+1，N为list长度

next[j]表示list[j-1]的最大k值，即原本的next[j-1]

例如：list=abaaba
原本next为[0,0,1,1,2,3]
现在next为[-1,0,0,1,1,2,3]

求next数组时
如果i=next[0]=-1时，表示当前j的前后缀匹配全失败了，所以next[j+1] = i+1 = 0， 同时i=i+1=0, j=j+1重新开始匹配
如果匹配成功，next[j+1]=i+1，同时i=i+1，j=j+1，开始下一个元素的匹配
可见j=-1时的逻辑和匹配成功的逻辑相同，所以两者代码合并

修改之后代码实现看着更简单，但是理解成本更高一点

next数组代码如下
```java
public static int[] getNext(int[] list) {
    int[] next = new int[list.length];
    next[0] = -1;
    int i = -1;
    int j = 0;
    while(j < list.length-1) {
        if(i == -1 || list[i] == list[j]) {
            i++;
            j++;
            next[j] = i;
        } else {
            i = next[i];
        }
    }

    return next;
}
```

同理，用该next数组实现KMP算法时，因为next[j]相当于原来的next[j-1]，所以匹配失败时直接j=next[j]

如果j==-1，表示S[i]和所有P[j]包括P[0]都不匹配，需要j=j+1=0，i++，重新进行匹配
如果匹配成功，则i++，j++
可见两个情况的逻辑相同，所以合并

对应的KMP算法如下
```java
public static void KMPSearch(int[] S, int[] P) {
    int[] next = getNext(P);

    int i = 0;
    int j = 0;
    while(i < S.length) {
        if(j == -1 || S[i] == P[j]) {
            i++;
            j++;
            if(j == P.length) {
                System.out.println("Found pattern at index " + (i - j));
                j = next[j];
            }
        } else {
            j = next[j];
        }
    }

    int a = 0;
}
```
