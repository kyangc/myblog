title: 编程算法艺术（July-Github版）学习笔记
date: 2015-03-16 20:00:00
categories: 技术
tags: [面试,算法] 
description: July的干货还是不少的……

---

>July的干货还是不少的……

<!--more-->

## 数据结构
### **一、字符串**
 - 旋转
 
 	三步反转法，O(N)复杂度。
 
 - 字符串包含
 
 	这里的问题是字符串是否包含某些字母，这里可以用bit-map实现。
 
 - 字符串转数字（AtoI）
 
 	考虑问题需要全面：
 
 	-	开头的正负号，是否只有正负号
 	-	字符串是否为空
 	-	字符串中包含非数字字符
 	-	溢出：
 		-	如 2147483650, result > MAX/10
		-	如 2147483649, result == MAX/10 && digit > MAX%10
 
 - 回文（palindrome）
 	-	判断：从中间往两边或者是从两边往中间，两个指针扫描一次即可。
 	-	最长回文子串（LPS）：
 		-	Manacher算法，O（N）复杂度。
 -	全排列
 	-	没有重复元素的全排列
 		
 		这个情况下的全排列可以使用递归完成。每次排列都是首字母依次与包含首字母在内的元素进行交换，交换后从次字母开始递归的进行算法。
 		
 		如：abc，首先a与a交换位置，求bc的全排列，b与b交换位置，求c的全排列，c的位置是最后一个，输出此时的字符串abc，退回到之前的状态栈，b与c交换位置，求b的全排列，b是最后一个，输出此时的字符串acb，退回到之前的状态栈……
 		
 	-	有重复元素的全排列
 	
 		这个情况下的全排列依然是上面的思路，但是这里需要注意的是，在元素i与元素j进行交换的时候，要求[i,j)中没有与第j个元素相等的元素。即被交换的元素不能曾经被交换过。
 		
 		如：abb，a与a交换，对bb进行递归，不满足交换条件，对b进行递归，到达底部无法交换，输出abb；a与b交换，满足条件可以交换，对ab进行递归，对a进行交换，得到ab，对b进行递归，发现到达底部无法交换，输出bab；对b进行交换，满足交换条件，得到ba，对a进行递归，发现到达底部无法交换，输出bba；a与b进行交换，发现不满足条件，略过，输出结束。
 		
 	-	全组合
 	
 		使用bit-map解决。
 		
 -	问题集【待补完】
 	-	第一个只出现一次的字符
 	-	在字符串中删除特定的字符
 	-	字符串匹配
 	-	字符个数统计
 		-	Hash
 	-	字符串的集合
 	-	最长重复子串（LCS）
 	-	字符串压缩
 	-	均分01
 		-	给定一个字符串，长度不超过100，其中只包含字符0和1,并且字符0和1出现得次数都是偶数。你可以把字符串任意切分，把切分后得字符串任意分给两个人，让两个人得到的0的总个数相等，得到的1的总个数也相等。
 		-	将该串字符串看做一个环，由于01均为偶数且切分后字符串可任意组合，那么必然存在一个直径将所有的1分为两个部分，同时也将0分为了两个部分，这两个部分的01个数相同。由此我们可以得到结论是，在该字符串中切至多两刀，我们就能找到一个窗，其中的01个数与其外的01个数相同。
 		-	代码实现可以以n/2的间隔建立两个窗边界，遍历一次即可知道该窗应放置在哪个位置了。
 
 
### **二、链表**
 - 在O(1)时间删除链表节点
 
 	用下个节点的数据覆盖前一个。注意链尾的情况。
 
 -	单链表的转置
 
 	使用三个工作指针，不断的将每个节点的指向改变。
 
 -	第k个节点
 
 	使用两个指针，前一个距离后一个K的距离，遍历至末尾。
 
 -	中间节点
 	
 	使用两个指针遍历，前一个的移动速度时候一个的两倍，遍历至末尾。
 	
 -	环
 	-	判断是否成环
 	
 		使用两个指针，前一个的移动速度是后一个的两倍，遍历至两者相遇，则说明存在环。
 	
 	-	寻找入口
 	
 		接上，在两个指针相遇之后，后指针回到链表起点，前指针以与后指针相同的速度从相遇点开始遍历链表，两者相遇节点就是环的开始节点。
 	
 -	判断是否相交（无环）
 
 	若相交，则两链表从相交处往后一定是相同的，那么只需要考察两链表最终是否指向同一节点。
 	
 -	有环情况下（有环）
 
 	若相交，在有环情况下，其必然共有一个环，只需要遍历其中一个链表，找到一个环中元素，再在另一个链表中寻找该元素即可。
 	
 	-	相交的第一个公共节点（无环）
 	
 		由对其的思想可知，在无环情况下，其公共部分是一样的，那么，只需要找到两个链表长度的差值，长的链表从该差值开始，与短链表一起同时遍历，遇到的第一个公共节点就是两个链表的第一个公共节点。
 		
 		
### **三、数组**
 - K-th
 
 	求一个数组中第K大的数，可以有以下集中思路：
 	
 	-	排序，取第K个。复杂度最好是O(NlogN)。
 	-	维护一个大小为K的堆，以最大堆为例，首先以O(K)的时间用前K个数建立一个最大堆，然后遍历后n-k个数，如果比对顶的元素小，那么将堆顶元素出堆，将该数加入堆中，出堆元素回到新入堆元素的位置上。
 	
 		由于堆操作的时间复杂度为O(logK)，复杂度可以简化至O(K+(N-K)logK)=O(NlogK)
 	-	使用QuickSelect算法。平均情况下O(N)复杂度。
 		-	使用快速排序思想，选取一个锚点pivot将数组分为两个部分Sa和Sb，若Sa的元素（不含pivot）个数大于等于K，那么第K大得数在Sa里，继续递归的对Sa进行考察；若Sa的元素等于K-1（即含pivot在内共有k个元素），那么该pivot即为待求的第K个值；若Sa的元素少于K个，那么说明第K大得元素在Sb中，应对这个序列应用算法。
 		-	在求pivot的时候使用median3算法，其做法是将给定序列的首中尾按照大小顺序排列，并把pivot放置在倒数第二个位置。
 		
 -	两个有序数列和的K-th问题
 	
 	使用最小堆进行处理。其思想在于从小到大依次找到前K个组合，由于A[0]+B[0]必然是第一个元素，紧跟其后仅比这个组合小的元素是A[0]+B[1]或A[1]+B[0]，将A[0]+B[0]出堆之后，将这两个元素压入堆中，作为下一次出堆的备选。推而广之的，设每次出堆的组合为A[i]+B[j]，那么需要同时将A[i+1]+B[j]和A[i]+B[j+1]压入堆中，直到完成K次出堆。由于建堆的时间复杂度为O(K)，堆操作时间为O(logK)，共有K*O(logK)次堆操作，总的时间复杂度为O(NlogK)。
 
 - K-Sum
 	-	2Sum：
 		-	排序后二分搜索，O(NlogN)/O(1)
 		-	排序后从头尾双向扫描，O(NlogN)/O(1)
 		-	构造X-S[i]数组或Hash，O(N)/O(N)
 	-	K-Sum:
 		-	3Sum可以转化为2Sum；4Sum可转化为3Sum然后转化为2Sum……
 		-	KSum的递归Sum(A,Sum,i)函数实现：（数组A，和为Sum，还有i个值）
 			-	若取第i个数，则问题转化为Sum(A,Sum-A[i],i-1)
 			-	若不取第i个数，则问题转化成Sum(A,Sum,i-1)
 			-	终止条件为Sum<=0或n<=0
 			-	输出结果条件为Sum==A[i]
 			
 -	背包问题（经典的动态规划模型）【待补完】
 	-	01背包
 	
 		N件物品，V个背包，放入第i件物品的费用是C[i]，价值为W[i]，求解放入哪些物品使得背包价值最大。
 		
 		-	解法：设f[i][v]代表了前i件商品恰好放入一个容量为v的背包中的最大价值那么可以得到这么一个等式：
 			
 			`f[i][v]=max{f[i-1][v],f[i-1][v-c[i]]+w[i]}`
 			
 			其代表的意义是，f[i][v]的最大值是“不放第i件商品得到的最大价值”和“放入第i件商品得到的最大价值”的大者。
 			
 - 最大连续子数组和
 	
 	输入一个整形数组，数组里有正数也有负数。数组中连续的一个或多个整数组成一个子数组，每个子数组都有一个和。 求所有子数组的和的最大值，要求时间复杂度为O(n)。
 	
 	-	解法：由于最大子数组的首尾肯定不能包含和为负的子数组，由此我们可以从头开始遍历该数组，得到从上一个非负位置开始的累加和，若累加和为负则清零并重新记录起点。并且在整个过程中维护两个变量：CurSum和MaxSum。由以下的公式来维护CurSum：
 		
 		`currSum = (a[j] > currSum + a[j]) ? a[j] : currSum + a[j];`
 		
 -	跳台阶问题	
 	
 	一个台阶总共有n级，如果一次可以跳1级，也可以跳2级。是斐波那契数列的一个变体。写出通项即可。
 
 -	换硬币问题
 
 	给定几个面额的硬币和一个价值和，问有多少种换法。
 	-	递归解法：设换法有F(Value,Kind)种，硬币面额数组为value[Kind]那么有:
 	
 		`F(v,k)=F(v,k-1)+F(v-value[K-1],k)`
 		
 		意义为，使用K种硬币的换法等于不使用第k种硬币的换法加上使用第k种硬币的换法。
 	-	非递归解法：使用DP。【待补完】
 	
 - 荷兰国旗问题
 	
 	现有n个红白蓝三种不同颜色的小球，乱序排列在一起，请通过两两交换任意两个球，使得从左至右，依次是一些红球、一些白球、一些蓝球。
 	
 	-	解法：类似于快排中的partition过程，中间的白球就是锚点，设置三个指针，Begin、Current、End，以以下原则进行移动、交换：
 		-	开始时Begin和Current在起点，End在终点。
 		-	若Current指向白球，则移动至下一个（Current++）。
 		-	若Current指向红球，则与Begin交换，并且Begin指向下一个，Current指向下一个（Begin++，Current++）
 		-	若Current指向蓝秋，则与End交换，Current不动，End指向前一个（End--）
 		-	运行至End遇到Current，结束算法。
 -	问题集
 	-	找出数组中的唯一出现过两次的元素，其余数都只出现过一次。
 		
 		（1-1000放在含有1001个元素的数组中，找到重复的那一个。）
 		
 		-	对数组求和，减去1-1000的和，就是重复的数字。这个方法可能因为相乘的结果过大而溢出。
 		-	利用异或的性质：`a^b^a=a^a^b=0^b=b(0与0异或为0，1与0异或为1，相当于不会对原数造成变化)`，将该1001个数组元素与0-1000进行异或，那么最终得到的结果就是重复的一个数。
 		-	Bit-map或Hash，将这1001个元素映射到每一位上，重复的那个数位置上为0。但是这种方法会带来额外的空间开销。时间复杂度都是线性的。
 	-	找出数组中唯一一个只出现一次的元素，其余元素均出现过两次。
 		-	利用异或的性质将求整个数组的异或和，结果即为唯一一个不重复的元素。
 	-	找出数组中唯一一个只出现一次的数，其余元素均出现过三次。
 		-	统计这个数组中的所有数（int）在各个位上的1出现的次数，单个的那个数如果在这个位上是1，那么肯定无法被3整除，反之可以，通过统计这个可以知道该单个的数在哪些位上是1，从而得到这个数。
 	-	找出唯一的K个数，其余的数都出现过两次，有几个数只出现过一次。
 		-	K=2
 			-	其思路在于，这两个只出现过一次的数肯定是不一样的，在某些位上肯定是不相同的，我们通过将整个数组进行异或和计算得到哪一位是不同的，根据该位是0还是1将原数组分为两个子数组，这两个子数组必然分别包含了这两个数以及其他重复的数，我们对这两个数组分别进行异或和的计算就可以分别得到这两个非重复的数了。
 		-	K=3
 			-	[待补完]
 	-	寻找数组中的逆序数
 		
 		给定一整型数组，若数组中某个下标值大的元素值小于某个下标值比它小的元素值，称这是一个反序。
 		-	利用并归排序算法的过程中自然产生的对于逆序数的访问，累计逆序数的个数。在并归排序每一次的变序选择中，可以记录下所有的逆序个数。
 	-	【待补完】
 	
### **四、树**

 -	二叉查找树
 	-	性质：
 		-	若任意结点的左子树不空，则左子树上所有结点的值均小于它的根结点的值；
		-	若任意结点的右子树不空，则右子树上所有结点的值均大于它的根结点的值；
		-	任意结点的左、右子树也分别为二叉查找树。
		-	没有键值相等的结点
		-	查找、插入、删除的时间复杂度均为O(logN)，但有可能退化至O(N)
 -	红黑树
 	-	本质上来说就是一棵二叉查找树，但它在二叉查找树的基础上增加了着色和相关的性质使得红黑树相对平衡，从而保证了红黑树的查找、插入、删除的时间复杂度最坏为O(log n)。
 	-	性质：
 		-	每个结点要么是红的，要么是黑的。  
		-	根结点是黑的。  
		-	每个叶结点（叶结点即指树尾端NIL指针或NULL结点）是黑的。  
		-	如果一个结点是红的，那么它的俩个儿子都是黑的。  
		-	对于任一结点而言，其到叶结点树尾端NIL指针的每一条路径都包含相同数目的黑结点。  
			以上的性质保证了红黑树的高度始终为logN，进而保证了各项操作的时间复杂度始终为O(logN)
 -	【待补完】
 
### **五、算法心得**
 -	有序数组的查找
 	-	二分搜索【要会写】
 -	杨氏矩阵的搜索
 	-	首先直接定位到最右上角的元素，配以二分查找，如果当前元素比待查数大则往左走，比待查数小则往下走。
 -	出现次数超过一半的数字
 	-	数组中有一个数字出现的次数超过了数组长度的一半，找出这个数字。
 	-	解法：
 		-	排序，O(NlogN)/O(1)
 		-	Hash，O(N)/O(N)
 		-	使用time/candidate
 			-	在遍历数组的时候保存两个值：一个candidate，用来保存数组中遍历到的某个数字；一个nTimes，表示当前数字的出现次数，其中，nTimes初始化为1。当我们遍历到数组中下一个数字的时候：
 				-	如果下一个数字与之前candidate保存的数字相同，则nTimes加1；
 				-	如果下一个数字与之前candidate保存的数字不同，则nTimes减1；
 				-	每当出现次数nTimes变为0后，用candidate保存下一个数字，并把nTimes重新设为1。 直到遍历完数组中的所有数字为止。