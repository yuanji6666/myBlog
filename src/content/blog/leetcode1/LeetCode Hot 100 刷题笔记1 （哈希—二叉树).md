---
title: LeetCode Hot 100 刷题笔记1 （哈希—二叉树)
publishDate: 2025-12-26 08:00:00
description: '实践费曼学习法，记录解题思路，完善中......'
heroImage: { src: 'image.png', color: '#B4C6DA' }
tags:
  - algorithm
  - leetcode
language: '中文'
---

# 哈希
# 双指针
# 滑动窗口
### 78.最小覆盖子串
这一题归于子串问题，其实本质上是**最短型**滑动窗口

如果我们从暴力情况考虑，能意识到滑动窗口的本质是**剪枝**，如果一个子串不满足题意，那么它包含的子串肯定不满足

枚举右端点，维护左端点就可以

子串的比较函数可以**维护两个哈希表**实现，key为字符ascII码，value为个数（惯用技巧）
# 子串
# 数组
### 41. 缺失的第一个整数
这一题很容易想到用一个长度为n的哈希表记录每个整数是否存在

但是难点在于要用常数级别的额外空间，所以我们想怎样在原数组的基础上标记

我们可以将对应元素移动到对应位置上去，但这样有点复杂

怎样在保证原有信息的情况下标记？

我们可以通过位置上元素的正负解决，因为负数在题中是无用信息

依循这个思路，首先我们要排除无关元素的干扰，都把它们赋为特定正值

然后遍历数组，把存在的对应位置上的元素标记为负
```go
for _, v := range nums{ 
	if v != 1000000 && v != -1000000{ 
		if nums[abs(v)-1]>0{ 
			nums[abs(v)-1] = 0 - nums[abs(v)-1]} } }
```
最后遍历一次得到答案
# 矩阵

# 链表

### 23.合并k个升序链表

**最直观的想法**，每次遍历所有头节点，取出最小的，更新头节点切片，递归得到
时间复杂度$O(Lm)$
空间复杂度$O(L)$  递归来到了恐怖的L

这一题有$k<=10^4$是很影响时间复杂度的
考虑**分治法**，k个链表合并最终化为2个链表合并
时间复杂度$O(Llog(m))$
空间复杂度$O(log(m))$

###146.LRU缓存

一道设计数据结构的题

首先要深刻理解题意，什么叫最久未使用，其实**使用**包括get，put操作

存储的是键值对，要求操作是O(1)的，我的想法是用切片存键值对？再用切片顺序储存最近使用？

不行的，因为用切片记录使用顺序无法实现O(1)的操作，因为要抽出的可能是切片中间的某个值

使用一个双向链表，自然地实现最近使用的逻辑，
只用把最近使用的放在链表开头，末尾就自然是最久未使用的

```go
type LRUCache struct {
    Capacity int
    Data *list.List//双向链表
    Hash map[int]*list.Element //key映射到节点
}
```

**注意各种操作的逻辑实现**

对于这样的自定义数据结构，我们需要**关注每一个字段在逻辑处理的时候是是否需要需要维护**

这样才能让逻辑严谨

```go
func (this *LRUCache) Get(key int) int {
    node := this.Hash[key] //取得element
	if node == nil {
		return -1
	}
	this.Data.MoveToFront(node)
	//不需要改变Hash表，因为移动节点其地址是不变的
	return node.Value.(pair).Value//注意类型断言 node.Value type is any
}
```


```go
func (this *LRUCache) Put(key int, value int)  {
	//已经存在的情况
    node := this.Hash[key]
    if node != nil{
        this.Data.MoveToFront(node)
		this.Data.Front().Value = pair{key, value}
		return
    }
	//不存在的情况
	this.Data.PushFront(pair{key,value})
	this.Hash[key] = this.Data.Front()
	//逐出未使用的
	if len(this.Hash) > this.Capacity {
		this.Hash[this.Data.Back().Value.(pair).Key] = nil
        this.Data.Remove(this.Data.Back())
	}
}

```
# 二叉树

### 230.二叉搜索树中第k小的元素

中序遍历同时维护一个count记录当前遍历到的是第几个节点就可以，等于k时更新ans

甚至可以让k递减，k等于0时即为ans

### 199.二叉树的右视图

很直观想到**层序遍历**，记录每层的最后一个即可

### 114.二叉树展开为链表

第一遍自己想了一种**后序遍历+子树重构**（deepseek总结），首先递归左右都展开为链表，然后把右子树加到左子树的最下面的节点

但是寻找这个节点的过程又浪费了不少时间

所以考虑正常的递归遍历，遍历**右左中**，采用**头插法**完美解决


### 437.路径总和

题面：统计祖先到孩子的路径上节点**总和**为target的个数

这道题和560的思路是一样的，当二叉树退化为一个**链表**时，和560一样

而560其实就是第一题的变式，**枚举右维护左+hash**就可以，只不过枚举的是前缀和数组

==*注意*== ：有一点要注意，左子树比右子树更早遍历，左子树对哈希表的更新可能会影响到右子树
所以我们在左子树结束时要==撤销对hash表的更新==，这一点区别是由于二叉树和数组**结构不同**导致的


```go
dfs = func(node *TreeNode, s int){
	//s is prefixSum of node's parent node
	
	//process border
	if node == nil {return}
	
	//renew prefixSum
	s += node.Val

	//find in hash
	ans += hash[s-targetSum]
	
	//renew hash
	hash[s]++

	//dfs
	dfs(node.Left, s)
	dfs(node.Right, s)

	//undo hash renew
	hash[s]--
}
```


### 236.二叉树最近公共祖先lowest common ancestor（LCA）

自己想了一种非常直观的做法，直接从root开始向下遍历，找到**左右都不是CA的节点**，即是LCA，否则向下递归

==怎样优化？==

需要找一些规律，人脑想的越多，代码复杂度就越低，越是直观的逻辑，代码复杂度就越高

对灵神代码的理解：

```go
func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
	/*先忽略名字，直观来看函数体，此函数要从root找到的是:
	  p或q或左右子树含p，q的那个节点（三种可能的情况）
	  其实仔细分析：LCA只可能有以下两种情况：
	  1. p, q 是祖先后代关系，lca是pq中的一个。
	  2. p, q 不是祖先后代关系，lca是那个左右都含p, q的。
	    
	  这样我们就明白了，该函数能满足上述的所有情况
	*/
	
	if root == nil || root == p || root == q {
		return root
	}
	
	left := lowestCommonAncestor(root.Left,p,q)
	right := lowestCommonAncestor(root.Right,p,q)
	
	//case 1
	if left != nil && right != nil {
		return root
	}
	
	//case 2
	if left != nil {
		return left
	}

	return right
}
```

### 124.二叉树最大路径和

想用递归，但是发现单纯去**把maxPathSum作为递归返回值**是不恰当的
因为子结构是拐弯的，再往上会分叉，明显不合题意

仔细思考，这个最大路径其实是所谓“直径”，它有一个拐点，只用算出
这个拐点加上左右两边的最长链就可以了

所以我们就把问题转化成了，**求某个节点向下的最长链**，递归过程中**顺带更新答案**

```go
func maxPathSum(root *TreeNode) int {
	ans := -100000000
	var dfs func(node *TreeNode) int
	dfs = func(node *TreeNode) int{ 
		//返回以node为最高节点的最长链长
		
		//边界处理
		if node == nil {
			return 0
		}

		//递归得左右最长链长
		left := dfs(node.Left)
		right := dfs(node.Right)

		//顺带处理最大路径和逻辑
		ans = max(ans, node.Val+left+right)

		//返回结果，如果最长链是负数，这个信息没有保存的必要，返回0
		return max(max(left, right) + node.Val, 0)
	}

	dfs(root)
	return ans	
}
```