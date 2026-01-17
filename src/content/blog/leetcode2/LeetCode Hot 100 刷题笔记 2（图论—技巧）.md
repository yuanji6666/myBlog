---
title: LeetCode Hot 100 刷题笔记 2（图论—技巧）
publishDate: 2025-12-27 08:00:00
description: 实践费曼学习法，记录解题思路，完善中......
heroImage: { src: 'image.png', color: '#B4C6DA' }
tags:
  - algorithm
  - leetcode
language: 中文
---
# 图论

### 200.岛屿数量

简单的dfs

### 994.腐烂的橘子

这一题用广度优先遍历是很自然的，因为腐烂一次是影响四周的橘子

不过自己做得好吃力，关键在一些细节上

**考虑 只有0，只有1，只有2的情况是否被考虑进去了**

```go
var dx [4]int = [4]int{1,-1,0,0}
var dy [4]int = [4]int{0,0,1,-1}
func orangesRotting(grid [][]int) int {
	m := len(grid)
	n := len(grid[0])
	fresh := 0
	minute := 0
	queue := make([][2]int, 0)
	
	//初始化队列
	for i := range grid {
		for j := range grid[i]{
			if grid[i][j] == 2 {
				queue = append(queue, [2]int{i, j})
			}else if grid[i][j] == 1 {
				fresh++
			}
		}
	}
	
	//核心逻辑
	for len(queue) != 0 && fresh != 0 {
		//每分钟
		minute++
		l := len(queue)
		
		//对于每个刚腐烂的
		for i:=0;i<l;i++{
			x := queue[i][0]
			y := queue[i][1]
			
			//每个方向
			for i2 := 0 ; i2 < 4 ;i2++{
				ax := x+dx[i2]
				ay := y+dy[i2]
				
				if ax >= 0 && ax < m && ay >= 0 && ay < n && grid[ax][ay] == 1 {
					grid[ax][ay] = 2
					fresh -- 
					queue = append(queue,[2]int{ax,ay})
				}
			}
			
		}
		
		//更新队列
		queue = queue[l:]
	}
	
	if fresh != 0 {return -1}
	return minute
}
```

### 208.前缀树

**实现数据结构题**

查找操作是很简单的，用哈希表就可以，但是**前缀查找**时间复杂度是很高的，要全部遍历一遍

使用前缀树，可以把前缀查找时间复杂度降为$O(n)$

# 回溯

回溯算法的精髓在于**回退**操作

当我们从深层次的节点回到浅层次的节点时，通过撤销回到原来的状态

这就需要我们设计合适的**状态变量**

**通过状态变量能精准地描述某种状态，通过修改状态变量能完美地回退到某个状态**

==边递归边剪枝？==

### 46.全排列

理解**回溯**经典的一道题

状态变量的设计：
-  递归结束条件是递归深度，所以我们需要depth记录递归深度
- 选过的数字不能再选，所以我们要记录已经选过的数字，即onPath变量

这一题中使用go语言解答要注意：
- 答案添加的时候一定不能直接append path，因为切片是引用类型，每次更改都会改变原来的值，结果中每个切片结果会相等

```go
func permute(nums []int) (ans [][]int){
	n := len(nums)
	path := make([]int, n, n)
	onPath := make([]bool, n)

	var f func(i int)
	f = func(i int){
		if i == n {
			//注意这里不能直接将path添加到ans后面，否则结果中每个切片值相等
			rst := make([]int, n)
			copy(rst, path)
			ans = append(ans, rst)
			return
		}
		for j, on := range onPath {
			if !on {
				path[i] = nums[j] 
				onPath[j] = true 
				f(i+1)
				onPath[j] = false
			}
		}
	}
	f(0)
	return ans
}
```

### 78.子集

求出所给数组的所有子集

状态变量的设计：
- 在固定递归深度结束，维护depth
- 每个结果的长度是不一样的，可以把set本身当作状态变量，撤销选只需要删除path中的末尾，撤销不选什么都不用做，depth已经说明了一切

```go
func subsets(nums []int) (ans [][]int ){
	n := len(nums)
	set := make([]int, 0)
	var dfs func(i int)
	dfs = func(i int){
		if i == n {
			copySet := make([]int, len(set))
			copy(copySet, set)
			ans = append(ans, copySet)
			return
		}
		set = append(set, nums[i])
		dfs(i+1)
		set = set[:len(set)-1]
		dfs(i+1)
	}
	dfs(0)
	return ans
}
```

### 17.电话号码 

这一题也是固定深度的递归，而且各层递归对字母的选择互不干扰，因此记录depth就行了

注意字符串和byte的转化，byte是uint8别名，能表示所有ascii码

```go
func letterCombinations(digits string) (ans []string ){
	digitsBytes := []byte(digits)
	n := len(digits)
	bytes :=make([]byte, 0)
	
	letters := [][]byte{
		{'a','b','c'},
		{'d','e','f'},
		{'g','h','i'},
		{'j','k','l'},
		{'m','n','o'},
		{'p','q','r','s'},
		{'t','u','v'},
		{'w','x','y','z'},
	}

	var dfs func(int)
	dfs =func(i int){
		if i == n {
			ans =append(ans, string(bytes))
			return
		}

		digit := digitsBytes[i]
		digit -='2'
		for _, value := range letters[digit]{
			bytes = append(bytes, value)
			dfs(i+1)
			bytes = bytes[:i]
		}

	}
	dfs(0)
	return
    
}
```

### N皇后

一道固定深度的递归

斜方向的约束怎样设计需要想一想

```go
func solveNQueens(n int) [][]string {
	chosenIndex := make([]int, 0)//记录选择答案中每行的列号，例如[1,3,0,2]
	chosenColumn := make([]bool, n)//记录已经选择过的列号
	ans := make([][]int, 0)//存储所有可行的chosenIndex

	var dfs func(int)
	dfs = func(i int){
		if i == n {
			ans = append(ans, append([]int{},chosenIndex...))
		}

		//分叉，遍历寻找可行分叉
		for j:=0 ; j < n ; j++ {
			//行约束已经满足
			//列约束
			if chosenColumn[j]{
				continue
			}

			//斜方向约束
			flag := true
			for index, value := range chosenIndex{
				//依次检查上方的第index行，选择的第value列
				if j-value == i-index || j-value == index-i {
					flag = false
					break
				}
			} 
			
			//都满足证明是合法尝试
			if flag {
				chosenIndex = append(chosenIndex, j)
				chosenColumn[j] = true
				dfs(i+1)
				chosenColumn[j] = false
				chosenIndex = chosenIndex[:len(chosenIndex)-1]
			}
		}

		
	}

	//递归,最多n层
	dfs(0)

	//转换答案
	finalAns := make([][]string,0)
	for _, v := range ans {
		oneOfTheAns := make([]string, n)
		for i, j := range v{
			oneOfTheAns[i] = strings.Repeat(".", j)+"Q"+strings.Repeat(".",n-j-1)	
		}
		finalAns = append(finalAns, oneOfTheAns)
	}
	return finalAns
	
}
```

# 二分

### 153.寻找旋转排序数组的最小值

经过思考，我发现了**旋转排序**数组与**红蓝染色**法的共性

红蓝染色最终根据**target**将数据分成两半

满足条件：蓝色的左边一定是蓝色，红色的右边一定是红色（数组**有序**）

旋转数组仔细一想其实也是两段，且也满足上面的条件
只不过**这里的蓝色是大于数组最后一个数的，红色是小于最后一个数的**，所以他们的解答如此相似！

```go
func findMin(nums []int) int {
	n := len(nums)
	l, r := -1, n
	for r - l > 1 {
		m := l + (r-l)/2
		if nums[m] <= nums[n-1] {
			r = m
		}else{
			l = m
		} 
	}
	return nums[r]   
}
```

### 33.搜索旋转排序数组

1. 两次二分：化归，通过153找到对应的段，然后正常查找
2. 一次二分：每次移动LR的时候，加一个条件判断是否在同一递增段
# 技巧

### 169.多数元素

