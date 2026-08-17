# LeetCode 刷题记录
## 📌 题目信息
- **题目编号**：437
- **题目名称**：路径总和 III Path Sum III
- **题目难度**：中等
- **题目链接**：https://leetcode.cn/problems/path-sum-iii/
- **刷题日期**：2026\08\17
- **标签**：二叉树 / DFS / 前缀和 / 树形回溯

## 📝 题目大意
给定二叉树根节点 `root` 和整数 `targetSum`，统计满足条件的向下路径总数：
1. 路径方向只能\(curSum_B - curSum_{A\(curSum_{A父节点} = curSum_B - targetSum\)} = targetSum\) → 子节点（自上而下）
2. **不需要从根节点出发，也不需要终止在叶子节点**
3. 一条路径由若干连续节点组成，节点数值之和等于 `targetSum`

> 主流思路：
> 方案1：双重DFS暴力枚举每个起点，最坏 $O(n^2)$，倾斜树容易超时；
> 方案2：**前缀和 + DFS + 回溯（当前实现）**，时间复杂度 $\(O(n)\)$，最优解法。

## 💡 解题思路
借鉴数组前缀和思想迁移到二叉树纵向路径：
定义：`curSum` = 从**根节点走到当前节点**的路径总和。
若存在一段中间路径 `[A→B]` 和等于 `targetSum`：
$$curSum_B - curSum_{A\(curSum_B - curSum_{A\(curSum_{A父节点} = curSum_B - targetSum\)} = targetSum\)} = targetSum$$
等价变形：
$$curSum_{A\(curSum_B - curSum_{A\(curSum_{A父节点} = curSum_B - targetSum\)} = targetSum\)} = curSum_B - targetSum$$

执行流程：
1. 哈希字典 `prefixs` 记录同一条路径上各个前缀和出现次数；
2. 初始化 `prefixs = {0:1}`：虚拟前缀和，处理路径直接从根开始就满足条件的场景；
3. DFS递归遍历：
   - 更新当前前缀和 `curSum`
   - 查询 `curSum - targetSum` 在字典中出现次数，累加为当前合法路径数量
   - 将当前前缀和存入字典计数
   - 递归遍历左、右子树
4. **回溯**：左右子树全部遍历完成后，撤销当前前缀和记录，防止左右分支互相干扰。

> 关键特性：二叉树是分支结构，左右子树属于两条独立路径，必须回溯清理状态。

## ⚙️ 复杂度分析
- **时间复杂度**：$\(O(n)\)$
  每个节点仅访问一次，字典操作为 $\(O(1)\)$
- **空间复杂度**：$\(O(h)\)$
  $h$ 为二叉树深度；哈希表最多保存一条从上到下路径上的前缀和；最坏链状树 $\(h=n\)$

## 💻 AC 代码
```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def pathSum(self, root: Optional[TreeNode], targetSum: int) -> int:
        prefixs = {0 : 1}

        def dfs(node, curSum) -> int:
            if not node:
                return 0

            curSum += node.val
            # 查询满足条件的前缀和数量
            res = prefixs.get(curSum - targetSum, 0)
            # 记录当前前缀和
            prefixs[curSum] = prefixs.get(curSum, 0) + 1

            res += dfs(node.left, curSum)
            res += dfs(node.right, curSum)
            
            # 回溯：撤销当前节点前缀和记录
            prefixs[curSum] -= 1
            if prefixs[curSum] == 0:
                del prefixs[curSum]

            return res
        
        return dfs(root, 0)
```

## 🧩 关键知识点 & 技巧
1. 数组前缀和 → 二叉树纵向路径前缀和，核心思想通用；
2. 初始 `{0:1}` 极易遗漏，对应起点为根节点的合法路径；
3. `curSum` 通过函数参数传递，每层独立副本，无需手动 `curSum -= node.val`，规避回溯时序错乱；
4. 字典取值优先使用 `.get(key, 0)`，避免未知key直接访问触发 `KeyError`；
5. 回溯操作放在左右递归之后，操作的 `curSum` 必须和存入字典的值保持一致。

## ❌ 错题反思 / 踩坑记录
1. **回溯顺序错误（重大坑）**
   先修改curSum，再操作字典，导致删除的key和存入key不一致，持续报`KeyError`；
2. dfs存在返回值时，空节点只写`return`，返回`None`，执行`res += dfs()`触发`TypeError`；
3. 使用 `prefixs[key]` 直接访问，不写默认值，不存在的键抛出`KeyError`；
4. 尝试在外层共用`curSum`、`ret`，`+=`触发Python嵌套函数`UnboundLocalError`；
5. 不执行回溯：右子树读取到左子树路径的前缀和，统计结果偏大；
6. 混淆三道路径总和题目：
   - 路径总和I：是否存在根→叶子路径；
   - 路径总和II：收集全部根→叶子路径；
   - 路径总和III：任意向下连续路径计数（本题）。

> 补充Python语法理解：
> 外层字典属于可变对象，内层可以直接修改内部元素；但`int`不可变，`+=`会触发变量重绑定，尽量通过参数传值规避`nonlocal`。

## ✅ 总结
树形前缀和+回溯是高频中等难度模板。
核心口诀：**先查询、再记录、递归左右、最后回溯清理；参数传递维护累加和，杜绝共用变量时序bug。**

如果你需要，我可以再补充【双重DFS暴力版本】作为拓展写到笔记里。
