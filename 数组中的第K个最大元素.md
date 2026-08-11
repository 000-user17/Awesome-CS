# LeetCode 刷题记录
## 📌 题目信息
- **题目编号**：215
- **题目名称**：数组中的第K个最大元素
- **题目难度**：中等
- **题目链接**：https://leetcode.cn/problems/kth-largest-element-in-an-array/
- **刷题日期**：2026\08\11
- **标签**：快速选择、分治、快速排序

## 📝 题目大意
给定整数数组 `nums` 和整数 `k`，返回数组中第 `k` 个最大的元素。
注意：是排序后的第 k 个最大元素，不是第 k 个不同元素。
要求实现平均时间复杂度 $O(n)$ 的算法。

示例：
输入：`[3,2,1,5,6,4]`, k = 2
输出：5

## 💡 解题思路
**核心思想：快速选择 QuickSelect**
借鉴快速排序的分区（partition），选定基准 `pivot` 将数组划分；
不需要完整排序，只向目标位置一侧搜索，平均时间复杂度 $O(n)$。
本题目标：降序排列后，下标 `k-1` 的元素就是第k大。

区间采用**左闭右开 `[l, r)`**
- 分区后得到分界位置 `pos`
- `pos == k-1`：找到答案
- `pos > k-1`：目标在左区间 `[l, pos)`
- `pos < k-1`：目标在右区间 `[pos+1, r)`

两套双向指针模板区别：
1. ✅ AC霍尔分区：交换元素后 `i +=1,j -=1`，持续收缩窗口，等值元素分散两侧，避免大数据量重复元素超时。
2. ❌ 超时写法：交换后不移动指针，搭配 `>=` 判断，大量重复数字时分区失衡，复杂度退化 $O(n^2)$。

## ⚙️ 复杂度分析
- **时间复杂度**：平均 $O(n)$；最坏 $O(n^2)$。随机选取pivot大幅降低最坏情况概率
- **空间复杂度**：$O(1)$，迭代原地划分

## 💻 AC 代码（Python）
```python
import random
class Solution(object):
    def findKthLargest(self, nums, k):
        """
        :type nums: List[int]
        :type k: int
        :rtype: int
        """
        l = 0
        r = len(nums)

        def partition_random(nums, l, r):
            if l == r - 1:
                return l
            
            pivot_idx = random.randint(l, r - 1)
            pivot = nums[pivot_idx]
            nums[pivot_idx], nums[l] = nums[l], nums[pivot_idx]

            i = l + 1
            j = r - 1
            while i <= j:
                while i <= j and nums[j] < pivot:
                    j -= 1
                while i <= j and nums[i] > pivot:
                    i += 1
                if i >= j:
                    break
                nums[i], nums[j] = nums[j], nums[i]
                i += 1
                j -= 1

            nums[l], nums[j] = nums[j], nums[l]
            return j

        while l < r:
            pos = partition_random(nums, l, r)
            if pos == k - 1:
                return nums[pos]
            elif pos > k - 1:
                r = pos
            else:
                l = pos + 1
```

## 💻 AC 代码（Python 大根堆）
```python
class Solution:
    def findKthLargest(self, nums: List[int], k: int) -> int:
        n = len(nums)

        def buildHeap(nums, n):
            n = len(nums)
            for i in range(n // 2 - 1, -1, -1):
                heapify(nums, n, i)
        
        def heapify(nums, n, i):
            largest = i
            left = 2 * i + 1
            right = 2 * i + 2

            if left < n and nums[left] > nums[largest]:
                largest = left
            
            if right < n and nums[right] > nums[largest]:
                largest = right
            
            if i != largest:
                nums[i], nums[largest] = nums[largest], nums[i]
                heapify(nums, n, largest)
            
        buildHeap(nums, n)

        for i in range(k):
            nums[0], nums[n - i - 1] = nums[n - i - 1], nums[0]
            heapify(nums, n - i - 1, 0)
        
        return nums[n - k]
```

## ❌ 超时问题代码（Python）
```python
import random
class Solution(object):
    def findKthLargest(self, nums, k):
        """
        :type nums: List[int]
        :type k: int
        :rtype: int
        """
        l = 0
        r = len(nums)

        def partition_random(nums, l, r):
            if l == r - 1:
                return l
            
            pivot_idx = random.randint(l, r - 1)
            pivot = nums[pivot_idx]
            nums[pivot_idx], nums[l] = nums[l], nums[pivot_idx]

            i = l
            j = r - 1
            while i < j:
                while i < j and nums[j] < pivot:
                    j -= 1
                while i < j and nums[i] >= pivot:
                    i += 1
                nums[i], nums[j] = nums[j], nums[i]

            nums[l], nums[i] = nums[i], nums[l]
            return i

        while l < r:
            pos = partition_random(nums, l, r)
            if pos == k - 1:
                return nums[pos]
            elif pos > k - 1:
                r = pos
            else:
                l = pos + 1
```

## 🧩 关键知识点 \& 技巧

- 技巧1：快速选择不要完整排序，只搜索单侧区间，实现平均 O (n)；

- 技巧2：随机选取 pivot，规避有序数组导致的最坏时间复杂度；

- 易错点：双向指针是成套模板，扫描条件、循环边界、交换后是否移动指针、最后交换用 i 还是 j 四者必须配套，不能随意混搭。

1. 双向指针模板混用：交换完成后不执行 i +=1,j -=1；
2. 扫描条件使用 >= 搭配 <，遇到大量重复数字时等值元素全部聚集一侧；
3. 循环终止后，混淆用 i 还是 j 和头部 pivot 交换；
4. 区间定义混淆：左闭右开 [l,r) 和闭区间 [left,right] 两套逻辑不能混用；
5. 目标下标混淆：升序目标 n-k、降序目标 k-1，必须和分区方向匹配。

## ❌ 错题反思 / 踩坑记录

**错误原因：**

1. 两套双向指针模板随意拼接；错误使用「交换之后不推进 i、j」的写法；
2. 内层循环条件 nums[i] >= pivot，所有等于 pivot 的元素全部向左侧堆积；
3. 测试用例包含大量重复数字（如 [2,2,2,2]）时，分区极度不平衡，每次只能剔除一个元素，算法退化到 \(O(n^2)\)，触发 LeetCode 超时。

**修正思路：**

1. 使用霍尔分区范式：交换成功后执行 i += 1; j -= 1，持续收缩搜索窗口；
2. 修改扫描条件为严格大于 / 小于 nums[i] > pivot、nums[j] < pivot，等值元素参与交换，分散到两侧；
3. 循环结束后使用 j 和最左侧 pivot 交换，返回 j。


## ✅ 总结
快速选择的核心难点不在思路，而在于partition 双向指针模板的一致性。
不同写法的指针初始位置、循环条件、交换后是否移动指针、最后交换使用 i/j，是成套绑定关系，不能零散复制粘贴。
遇到大量重复元素测试用例极易超时，优先选择交换后主动收缩窗口的霍尔分区写法。如果你需要，我可以再补充基于堆的 TopK 解法，并标注堆解法 \(O(n\log k)\) 不满足本题严格 O (n) 要求。
