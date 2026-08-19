# LeetCode 刷题记录
## 📌 题目信息
- **题目编号**：337
- **题目名称**：打家劫舍 III
- **题目难度**：中等
- **题目链接**：https://leetcode.cn/problems/house-robber-iii/
- **刷题日期**：2026\08\19
- **标签**：二叉树 / 树形动态规划 / DFS

## 📝 题目大意
房屋按照二叉树结构排布，父子节点为直接相连房屋。**不能同时打劫直接相连的两个节点**，求小偷可以盗取的最高金额。
> 系列对比：
> 打家劫舍Ⅰ：普通一维数组；
> 打家劫舍Ⅱ：环形数组；
> 打家劫舍Ⅲ：二叉树树形结构。
> 难点：树结构，每个节点有两种选择：打劫 / 不打劫；子树的最优解依赖父节点是否被选择。

## 💡 解题思路
### 思路1：记忆化搜索（DFS + 哈希缓存）
#### 核心思想
递归函数增加参数 `isSelect` 标记**当前节点是否被打劫**。
- `isSelect=True`：打劫当前节点，则左右子节点一定不能打劫；
- `isSelect=False`：不打劫当前节点，则左右子节点可以自由选择打劫或者不打劫，取各自最大值。

因为同一 `(node, isSelect)` 组合会被重复递归调用，使用字典 memo 缓存已经计算过的结果，避免重复计算。

**步骤拆解**
1. 边界：空节点收益为0；
2. 缓存命中：如果`(node,isSelect)`已经计算，直接返回缓存值；
3. 如果打劫当前节点：当前值 + 左子树不选 + 右子树不选；
4. 如果不打劫当前节点：左子树选/不选取最大 + 右子树选/不选取最大；
5. 将结果存入memo缓存；
6. 最终答案取根节点两种状态的最大值。

### 思路2：树形DP（元组返回，最优解法，面试首选）
#### 核心思想
后序遍历，一次递归同时返回子树的**两种状态结果**，打包成元组返回：
- `sel`：打劫当前节点，该子树最大收益
- `not_sel`：不打劫当前节点，该子树最大收益

状态转移公式：
$$
sel = node.val + left_{not\_sel} + right_{not\_sel}
$$
> 打劫当前节点 → 左右孩子都不能打劫，只能取孩子不选的收益

$$
not\_sel = \max(left_{sel}, left_{not\_sel}) + \max(right_{sel}, right_{not\_sel})
$$
> 不打劫当前节点 → 左右孩子不受限制，各自取打劫/不打劫的较大值相加

**步骤拆解**
1. 边界：空节点返回 `(0, 0)`；
2. 后序递归，先拿到左、右子树的 `(sel, not_sel)`；
3. 根据公式计算当前节点的 sel、not_sel；
4. 返回元组 `(sel, not_sel)` 向上传递；
5. 根节点返回的元组取最大值即为答案。

> 关键区别：记忆化搜索是**多次递归同一节点，靠缓存消除重复**；树形DP是**每个节点仅遍历一次，一次性算出全部状态向上返回，无重复子问题**。

## ⚙️ 复杂度分析
### 记忆化搜索版本
- **时间复杂度**：$\(O(n)\)$，每个节点两种状态，一共 $2n$ 个子问题，每个问题计算 $\(O(1)\)$
- **空间复杂度**：$\(O(n+h)\)$，$n$ 为memo哈希缓存；$h$ 二叉树递归栈深度；平衡树 $h=\log n$，链式树 $\(h=n\)$

### 树形DP元组返回（最优）
- **时间复杂度**：$\(O(n)\)$，每个节点仅访问1次，无重复计算
- **空间复杂度**：$\(O(h)\)$，仅递归栈开销，无额外哈希缓存空间

## 💻 AC 代码
### ① 记忆化搜索版本
```python
# Definition for a binary tree node.
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

from typing import Optional
class Solution:
    def rob(self, root: Optional[TreeNode]) -> int:
        memo = {}

        def dfs(node, isSelect: bool):
            if not node:
                return 0
            key = (node, isSelect)
            if key in memo:
                return memo[key]
            
            if isSelect:
                # 选当前节点，子节点不能选
                res = node.val + dfs(node.left, False) + dfs(node.right, False)
            else:
                # 不选当前节点：子节点自由选最优
                res = max(dfs(node.left, True), dfs(node.left, False)) + max(dfs(node.right, True), dfs(node.right, False))
            
            memo[key] = res
            return res
        
        return max(dfs(root, True), dfs(root, False))
```

### ② 树形DP元组返回（最优面试版）
```python
# Definition for a binary tree node.
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

from typing import Optional
class Solution:
    def rob(self, root: Optional[TreeNode]) -> int:
        def dfs(node):
            # 返回 (选当前节点收益, 不选当前节点收益)
            if not node:
                return (0, 0)
            sel_left, not_sel_left = dfs(node.left)
            sel_right, not_sel_right = dfs(node.right)

            sel = node.val + not_sel_left + not_sel_right
            not_sel = max(sel_left, not_sel_left) + max(sel_right, not_sel_right)
            return (sel, not_sel)
        
        return max(dfs(root))
```

## 🧩 关键知识点 & 技巧
1. 树形DP经典套路：节点存在多组状态，**优先一次递归把全部状态算出打包返回**，而不是多次递归同一个节点；
2. 记忆化搜索：适合容易写出暴力递归的场景，靠缓存解决重复子问题；代价是额外哈希表开销；
3. 状态定义区分：
   - `isSelect=False` 不代表子节点必须选！子节点可以选或不选，各自取最大值；
4. 空节点返回 `(0,0)`：选空、不选空收益都是0，保证叶子节点计算逻辑统一；
5. 最终答案一定取根节点两种状态的最大值，不能只返回打劫根节点的值。

## ❌ 错题反思 / 踩坑记录
### 原始错误暴力写法（超时+逻辑错误）
```python
# ❌ 错误示例
def dfs(node, isSelect):
    if not node:
        return 0
    if not isSelect:
        return dfs(node.left, True) + dfs(node.right, True) # bug：强制子节点必须选
    val1 = node.val + dfs(node.left, False) + dfs(node.right, False)
    val2 = dfs(node.left, True) + dfs(node.right, True)
    return max(val1, val2)
```
**问题**
1. 逻辑错误：不选当前节点时强制子节点必须选，没有取子树max；
2. 指数级时间复杂度：`dfs(node,True)`、`dfs(node,False)`两套独立递归，子树被反复遍历，$O(2^n)$，大数据直接超时。

**其他常见坑**
1. 记忆化搜索不要直接用 `lru_cache`，TreeNode对象不能被缓存装饰器处理，要用手动字典memo；
2. 树形DP不要漏掉 `max()`，不选当前节点时子节点是自由选择；
3. 直接返回sel，忘记取`max(sel, not_sel)`，根节点不打劫的时候直接错解。

**两种方案对比总结**
|方案|时间复杂度|额外空间|优缺点|
|---|---|---|---|
|暴力无记忆递归|$O(2^n)$|$\(O(h)\)$|逻辑简单，指数爆炸，大数据超时|
|DFS+memo记忆化搜索|$\(O(n)\)$|$\(O(n+h)\)$|可以AC；需要维护哈希缓存，代码更长|
|✅树形DP元组返回|$\(O(n)\)$|$\(O(h)\)$|理论最优；无缓存，每个节点遍历一次；面试推荐|

## ✅ 总结
打家劫舍 III 是树形DP入门标杆题目。
- 记忆化搜索属于**自上而下**思路：暴力递归+缓存消除重复计算，容易从暴力改写出来，但是会带来额外哈希开销。
- 元组返回树形DP属于**自底向上后序DP**，一次递归同时输出节点全部状态，没有重复子问题，时间空间达到理论下界，是面试最优解。

> 通用迁移技巧：二叉树每个节点有多种状态时，dfs返回元组承载多状态，后序遍历由子节点推导父节点，大量树形DP题目可以复用这套模板。
