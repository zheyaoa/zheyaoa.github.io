---
title: 数组中出现次数超过一半的数字(快排解法)
date: 2019-03-25 16:31:43
tags: quickSort
categories: [Algorithm,sort]
---

最近一直在刷算法，在牛客上刷到了一题[数组中出现次数超过一半的数字](https://www.nowcoder.com/ta/coding-interviews/question-ranking?uuid=e8a1b01a2df14cb2b228b30ee6a92163&rp=2),题目大意如下。

**数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。例如输入一个长度为9的数组{1,2,3,2,2,2,5,4,2}。由于数字2在数组中出现了5次，超过数组长度的一半，因此输出2。如果不存在则输出0。**

本文章使用到了**快速排序**，对于快速排序这里不做过多讲解，如不了解可以查看对于[快速排序](https://blog.csdn.net/MoreWindows/article/details/6684558)的详细讲解。

<!--more-->

### 思路分析

​	作为一个小菜鸡，第一想法是因为该数字出现次数超过数组长度的一半,所以排序后数组的中位数一定会是该数字。当然我们也不需要完全进行排序(事实上我们只需要知道在中位数上的值就行了)。自然而然想到了快排，每一次Partition排序返回的次序(**后用index表示**)后，如果需要寻找的中位值(**后用target表示**)==**index**,则直接返回arr[index],否则在对应区间Partion直到找出结果。

​	当然还有更好的思路，但这并不是本篇博客的重点。本篇博客的侧重点在于快速排序的解法。

### 代码实现

```java
@params array 传入的数组
public static int Solution(int[] array){
  int target = array.length/2;
  int start = 0,end = array.length-1;
  return getTarget(array,start,end,target);
}
public static int getTarget(int[] array,int start,int end,int target){
	int index = partion(array,start,end);
  if(target == index){
     	return array[target];
   }else if(target < index){
   		return getTarget(array,start,index-1,target);
   }else{
   		return getTarget(array,index+1,end,target);
   }
}
public static void swap(int[] array,int i,int j){
	int tmp = array[i];
  array[i] = array[j];
  array[j] = tmp;
}
private static int partion(int[] array,int start,int end){
	int left =start;
	int root = array[start];
  while(start<end){
  	while(array[end]>=root&&start<end){
    	end -- ;
  	}
 	  while(array[start]<=root&&start<end){
    	start++;
 	  }
 		swap(array,start,end);
  }
  swap(array,)
  return start;
}
```

​	这里着重分析的是`getTarget`函数，在思路分析中我们已经知道了我们只是需要找到`target = (start+end)/2`即可,利用`partion`函数每次可得到一个值index,所有小于`arr[index]`的值均位于index左部，大于`arr[index]`的值位于右部,`arr[index]`在`arr`数组完全排序后次数为`index`

​	在这里将分情况讨论，如果获取的值`index`恰好等于`target`，则表明`arr[index]`正是我们要找的值。否则如果`target<index`将在`index`的左部继续寻找，否则在右部寻找，具体代码如下

```java
if(target == index){
  return array[target];
}else if(target < index){
  return getTarget(array,start,index-1,target);
}else{
  return getTarget(array,index+1,end,target);
}
```

以上就是对于数组中出现次数超过一半的数字快速排序的解题思路。

