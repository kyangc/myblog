title: 剑指Offer 读书笔记
date: 2015-03-18 20:00:00
categories: 技术
tags: [面试,算法] 
description: 老夫的读书笔记不可能这么萌
---

>老夫的读书笔记不可能这么萌

<!--more-->

-	Q4：**替换空格**
	-	从后向前进行，首先计算空格个数得到替换之后的String长度，使用两个指针，一个指向旧String的尾部，一个指向新String的尾部，按照前指针的内容进行替换，当两个指针相遇时停止。
-	Q5：**反向打印链表**
	-	使用栈
	-	使用递归（当递归层级过深可能导致函数调用栈溢出）
-	Q10：**数字中1的位数**
	-	直接统计可能会造成死循环（负数情况）
	-	使用对于1进行移位求与，当1移位N次之后为0则停止循环。
	-	使用结论：`把一个整数减去1再与原数求与会把该整数的最后一位1置0`，不断的对待考察数进行该操作，知道被操作数归零结束。该方法的复杂度会降低。
-	Q10.1：**判断一个数是不是2的整数次方**
	-	由上面的结论，2的整数次方数其二进制中只有1个1，只需要对该数减一之后与原数求与，若结果为零则是整数次幂。
-	Q11：**求一个数的N次幂**
	-	底数是不是0
	-	指数的正负
	-	double的equal方法
	-	使用位运算来高效的进行2的幂次运算、使用递归来对幂次进行二分，考虑奇偶性。
-	Q12：**打印1到最大的N位数**
	-	大数陷阱：当N大到无法用常用基本数据类型承载的时候，如何打印：
		-	使用Char数组，模拟整数加法，注意结束条件（复杂），打印时不能打印出最前面的0.
		-	使用Char数组，对每一位取10种可能，递归到结束。
-	Q13：**在O(1)复杂度时间内删除链表节点，给定该节点的指针和链表头指针。**
	-	【狸猫换太子】
		-	若该节点不是独立节点或尾节点，那么将该节点的下一个节点的内容复制到该节点，并将该节点的下一节点指针指向该节点的下一节点的下一节点。
		-	若该节点是尾节点，只能遍历链表找到前序节点，将其next指针置空，释放尾节点空间。
		-	若该节点是头结点，那么将头指针置空，释放节点空间。
		-	虽然在尾节点的时候会需要时间去遍历，但是平均时间复杂度还是维持在O(1)上的。
-	Q15：**链表中的倒数第K个节点**
	-	双指针，前指针距后指针K-1的距离。
	-	考虑以下负面情况：
		-	头指针为空
	-	K为0
	-	链表长度小于K
-	Q16：**链表反转**
	-	三指针，pre，cur，next，遍历一次即可。
	-	注意以下情况：
		-	头指针为空
		-	单节点情况
		-	返回值为尾指针
		-	链表断裂
-	Q17：**合并两个排序的链表**
	-	采用递归的方式合并，考虑头指针为空的情况。
		
		
		```
			mergeLinkedList(head1,head2)
			Node* mergedHead
			if(head1->value < head2->value)
				mergedHead = head1;
				mergedHead->next = mergeLinkedList(head1->next,head2);
			else
				mergedHead = head2;
				mergedHead->next = mergeLinkedList(head1,head2->next);
			
		```
			
-	Q18：给定两个二叉树AB，输出B是否是A的子树
	-	使用两个递归函数进行：
			
			
		```
		bool HasSubTree(head1,head2){
			bool hasSubTree = false;
				
			if(head1!=null && head2!=null){
				
				if(head1->value == head2->value)//从根节点是否包含
					hasSubTree = DoesTree1HasTree2(head1,head2);
				
				if(!hasSubTree)//根节点不包含的话，左子结点是否包含
					hasSubTree = DoesTree1HasTree2(head1->left,head2);
					
				if(!hasSubTree)//左子结点不包含的话，右子节点是否包含
					hasSubTree = DoesTree1HasTree2(head1->right,head2);
			}
				
			return hasSubTree;
		}
			
		bool DoesTree1HasTree2(head1,head2){//判断是否包含的核心函数
			
			if(head2 == null) return true;//树2为空，肯定包含
			if(head1 == null) return false;//树1为空，肯定是不不含的
			if(head1->value != head2->value) return false;//根节点的值不一样的话，肯定也是不包含的
				
			return DoesTree1HasTree2(head1->left,head2->left) && DoesTree1HasTree2(head1->right,head2->right)//根节点相同的情况下，每个节点都包含那么就包含。
			
		}
		```
			
			
-	Q19：**二叉树的镜像**
	-	前序遍历二叉树，若节点有子节点，那么交换两个子节点，递归完成。
-	Q20：**顺时针打印矩阵**
	-	将问题分解：
		-	打印一个矩阵可以看做是对矩阵的几个环的打印，每个环的起点都是（startX，startX），当columns>StartX\*2&&rows>startX\*2时循环可以继续。
		-	打印一个环可以看做在4个方向上遍历矩阵，但是不同的情况下需要遍历的边不一样：
			-	从左向右的第一步是肯定存在的，从startX打印到Columns-1-startX
			-	从上到下的第二步，只有在结束行号比开始行号大的时候才存在，从startY打印到Rows-1-startY
			-	从左到右的第三部，只有在结束行号比开始行号大并且结束结束列号比开始咧号大时才有。
			-	从下到上的第四步，只有在结束行号比开始行号大2时才会有。
-	Q21：包含min的栈
	-	实现一个栈，调用min、push、pop的时间复杂度都是O(1)
		-	使用辅助栈保存每次入栈时的最小值，每次出入栈操作都是对两个栈同时操作。这样在另一个栈中，栈顶元素总是原始栈中相同高度的最小值。
-	Q22：**栈的压入、弹出序列**
	-	给定两个序列，判断后一个是不是前一个的弹出序列。
	-	建立一个辅助栈，将前一个序列按照顺序依次入栈，直到遇到弹出序列的第一个元素，将这个元素入栈再出栈，若此时的栈顶元素不等于出栈序列的下一个数，那么继续按照入栈序列入栈直到遇到出栈序列的下一个数，如果最后所有入栈序列的数都入栈了还没找到下一个元素，那么该给出的出栈序列不是该入栈序列的出栈序列。
-	Q25：**二叉树中和为某值得路径**
	-	计算和的方法很简单，利用递归，如果加上本节点之后和达不到，则以新的目标遍历两个子节点，如果达到了，并且该节点是叶子节点，那么将其输出。
	-	本题要求输出路径，那么利用栈储存每次经历的节点，在考察完毕后pop即可。
			
			
		```
			FindSum(Node* root, int expectedSum, int curSum, Vector Path){
				//便利到此节点时的和
				curSum+=root->value;
				
				//将该节点加入路径
				path.push(root->value);
				
				//打印路径
				bool isLeaf = root->left==null && root->right->right==null;
				if(curSum==expectedSum && isLeaf){
					//print path
				}
				
				//不是叶子节点，遍历他的两个叶子节点
				if(root->left!=null) FindSum(root->left,expectedSum,curSum,path);
				if(root->right!=null) FindSum(root->right,expectedSum,curSum,path);

				//返回父节点之前先删除当前节点的路径
				path.pop();
			}
		```
			
			
-	Q26：**复制复杂链表**
	-	首先用next完成链表的初始复制，与此同时，将每个节点的源节点和复制后的节点的对应用hashmap存起来，之后在对sibling进行赋值的时候可以找到对应的地址。【空间换时间】
	-	将新链表复制到每个节点的后面：a->b->c => a->a'->b->b'->c->c'，将新的sibling复制为旧sibling的首尾均+1位置，最后将基数为和偶数位分别连起来就成了两个链表了。
-	Q28：**字符串的排序**
	-	将每一位与后面的位进行交换之后对剩下的位进行递归的相同运算。
	-	Q28.1：N皇后问题
		-	将A[N]设置为0~N-1，对应了第N行中皇后放在那一列。对这个数组进行全排列，对每个排列只需要考察其是否满足对角线上没有皇后这一条件即可。（肯定不在同一行同一列的）
		-	判断是否在对角线上是看是否满足i-j==A[i]-A[j]或j-i==A[i]-A[j]
-	Q33：**把数组排成最小的数**
	-	将一个数组按一定顺序拼接起来，得到的数最小
	-	不能全排列之后找最大，太多
	-	以某种规则将该数组排序是最好的方式:
		-	设元素一为m，元素二为n，拼接起来为mn或nm，若mn>nm，那么n应该在m前面
		-	写一个compare函数，用qsort对数组排序，整个复杂度在O(NlogN)内
		-	当然这个理论需要**证明**就是了……
-	Q38：**二叉树的深度**
	-	采用递归的方式可以方便的得到：
		
			
		```
			int treeDepth(Node*	tree){
				if(tree == null) return 0;
				
				int leftDepth = treeDepth(tree->left);
				int rightDepth = treeDepth(tree->right);
				
				return (leftDepth > rightDepth)?leftDepth+1:rightDepth+1;
			}
		```
			
			
-	Q38.1：**判断一个二叉树是不是平衡二叉树（任意节点的左右子树深度相差不超过1）**
	-	使用后序遍历的方式遍历整个二叉树，在遍历某节点的左右子节点之后可以根据其深度判断是不是平衡的并且更新当前根节点的深度。
				
				
			
		```
					bool IsBalance(Node* tree, int* depth){
					
						if(tree == null)	{ 
							*depth = 0;
							return true;
						}
						
						int left,right;
						if(IsBalance(tree->left,&left) && IsBalance(tree->right,&right)){
							int diff = left - right;
							if(diff >= -1 && diff <= 1){
								*depth = 1+(left>right?left:right);
								return true;
							}
						}								
						return false;
					
					}
					
					bool IsBalance(Node* tree){
						int depth = 0;
						return    1IsBalance(tree,depth);
					}
					
		```
					
					
					
-	Q43：**N个骰子的点数**
	-	N个骰子抛掷点数和为S的概率。
	-	依据：**和为S时N个骰子的情况个数等于N-1个骰子时，和为S-1、S-2、S-3、S-4、S-5、S-6的情况和。**
	-	由此，可以由两个数组很方便的得到结果。
-	Q44：**判断五张牌是不是顺子**
	-	J=11，Q=12，K=13，A=1，大小Joker可以代表任何数，Joker=0
		-	首先对5张牌排序，然后统计0的个数、是否有连续两个非零的数、连续两个不同得数之间差了多少，能不能和0的个数相同。
-	Q47：不使用加减乘除做加法
	-	使用位运算：
		-	第一步：按位异或，得到数A（模拟相加不进位）
		-	第二步：按位与，左移一位，得到数B（得到进位的数）
		-	第三部：使用AB重复前两步