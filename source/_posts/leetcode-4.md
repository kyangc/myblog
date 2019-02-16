title: 4# Median of Two Sorted Arrays
date: 2015-01-04 12:00:00
categories: 算法
tags: [二分,Kth,算法] 
description: 弱鸡进城纪实
---

>弱鸡进城纪实。

<!--more-->

昨晚睡前开搞的一道LeetCode，[#4 Median of Two Sorted Arrays][1]，大意是，给定两个有序数组，找出这两个数组的中位数，时间复杂度在O(logN)内。

OK，拿到这道题的第一思路是并归排序之后取中值。但是因为复杂度为O(M+N)，并不符合题意，于是另作他想。

昨晚因为太晚没有细想，就是模模糊糊觉得对数时间复杂度这个应该和二分搜索有关，具体的又没什么头绪。

今天继续攻略这道题，未果，死马当活马医的写了一个并归算法提交上去没想到居然AC，囧rz……

但是怎么想怎么不爽，这明显他喵的不对也能过？得过且过明显不是我的忍道，于是上网找了些资料，才知道这道题应该做如是处理：

求两个有序数组的中位数，其实可以作为“**求两个有序数组的第K个最小值**”的特殊情况来看待。

而我们可以轻易得到这样三个结论：

*	**有序数组A的第K/2个值如果比有序数组B的第K/2个值小，那么有序数组A的前K/2个值中不可能会有有序数组A+B的第K个值。**

*	**有序数组B的第K/2个值如果比有序数组A的第K/2个值小，那么有序数组B*	的前K/2个值中不可能会有有序数组A+B的第K个值。**

*	**有序数组A的第K/2个值如果与有序数组B的第K/2个值相等，那么该值即为A+B的第K个值**

进一步的，我们可以得出一个这样的结论：**如果A数组中第Ka个数比B数组中得第K-Ka个数小，那么A+B的第K个数肯定不在A数组的前Ka个数中。**

由此我们可以写出一个递归的函数，其作用为返回有序数列A和B组合之后的第K个数：

```
double findKth(int A[], int m, int B[], int n, int k){
		 //默认人为A的长度比B小
		 if(m>n) return findKth(B,n,A,m,k);
		 //如果A的长度为0，那么返回B[K-1]
		 if(m == 0) return B[k-1];
		 //如果K=1（求第一个数），那么可以直接得到
		 if(k == 1) return min(A[0],B[0]);
		 
		 //将K分为前两个K/2，如果A
		 int ka = min(k/2,m);
		 int kb = k - ka;
		 
		 //开始递归
		 if(A[ka -1]>B[kb-1])
		 		//如果A数列中第Ka个数比较大，那么可以舍弃B数列中的前Kb个数
		 		return findKth(A,m,B+kb,n-kb,k-kb);
		 else if(A[ka -1]<B[kb-1])
		 		//如果B数列中第Kb个数比较大，那么可以舍弃A数列中的前Ka个数
		 		return findKth(A+ka,m-ka,B,n,k-ka);
		 else
		 		//如果A数列中的第Ka个数和B数列中的第Kb个数相同，那么这就是A+B的第K个数
		 		return A[ka-];
		 }
```

OK，到这里基本就结束了，不过这道题在处理两个数列个数之和为偶数的时候是取中间两个中位数的均值作为最终的中位数的，所以这里在外边需要对于奇偶性进行分开讨论：

```
double findMedianSortedArrays(int A[], int m, int B[], int n) {
    	int length = m+n;
    	if(length %2==1){
    		return findKth(A,m,B,n,length/2+1);
    	}else{
    		return (findKth(A,m,B,n,length/2)+findKth(A,m,B,n,length/2+1))/2;
    	}
    }
```

OK，到这里这道题就结束了，整个复杂度是O(log(M+N))，最后运行的结果也很漂亮，108ms，基本是最靠前的了。

这里必须吐槽一句，同样的写法，我用Java提交，用时将近一秒，用CPP提交，用时108ms，不得不感叹一下，Java真的很不适合做这种数组类型的题……

（不，主要是我LOW）

[1]:https://oj.leetcode.com/problems/median-of-two-sorted-arrays/