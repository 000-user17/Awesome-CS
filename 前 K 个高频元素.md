# LeetCode 刷题记录
## 📌 题目信息
- **题目编号**：347
- **题目名称**：前 K 个高频元素
- **题目难度**：中等
- **题目链接**：https://leetcode.cn/problems/top-k-frequent-elements/
- **刷题日期**：2026\08\18
- **标签**：哈希表 / 堆 / 桶排序

## 📝 题目大意
给你一个整数数组 nums 和一个整数 k ，返回其中出现频率前 k 高的元素，返回顺序不作要求。

## 💡 解题思路
提供两种主流可行解法：哈希+小根堆、哈希+桶排序。
### 思路一：哈希统计频率 + 容量为k的小根堆（面试常用）
1. 使用哈希字典统计数组中每个数字出现的频次；
2. 维护一个小根堆，堆内存储元组 `(频次, 数值)`，堆顶为当前堆内频次最小元素；
3. 遍历频次字典，不断将元素入堆；当堆长度超过k时，弹出堆顶频次最小元素；
4. 遍历结束后，堆中剩余k个元素就是频次最高的前k个数字。

### 思路二：哈希统计频率 + 桶排序（理论时间最优）
1. 哈希字典统计各数字出现频次；
2. 创建桶数组，数组下标代表频次，每个桶存放对应频次的数字；
3. 从最大频次位置向前逆序遍历桶，依次收集数字，收集满k个直接返回答案。

**步骤拆解：**
1. 遍历数组，统计每个数字出现次数；
2. 方案一：借助小根堆不断筛选，保留频次最大k个元素；
3. 方案二：利用频次作为下标构建桶，从高频率往低频收集结果；
4. 整理结果返回。

## ⚙️ 复杂度分析
- **方法1（哈希+小根堆）**
时间复杂度：$O(n\log k)$，n为数组长度，堆操作复杂度 $\log k$
空间复杂度：$\(O(n)\)$，哈希表存储所有数字频次

- **方法2（哈希+桶排序）**
时间复杂度：$\(O(n)\)$，线性遍历数组与桶数组
空间复杂度：$\(O(n)\)$，哈希表与桶数组占用空间

## 💻 AC 代码（Python）
### 解法1：哈希 + 小根堆
```python
import heapq
from typing import List
class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        freq = dict()
        for num in nums:
            freq[num] = freq.get(num, 0) + 1

        heap = []
        for num, cnt in freq.items():
            heapq.heappush(heap, (cnt, num))
            if len(heap) > k:
                heapq.heappop(heap)
        return [item[1] for item in heap]
```

### 解法2：哈希 + 桶排序
```python
from typing import List
class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        freq = dict()
        for num in nums:
            freq[num] = freq.get(num, 0) + 1
        
        bucket = [[] for _ in range(len(nums) + 1)]
        for num, cnt in freq.items():
            bucket[cnt].append(num)
        res = []
        for i in range(len(nums), 0, -1):
            for val in bucket[i]:
                res.append(val)
                if len(res) == k:
                    return res
        return res
```

## 🧩 关键知识点 & 技巧
- 技巧1：Python heapq 仅实现小根堆，保存`(频次,数值)`，依靠频次自动排序；
- 技巧2：限制堆最大容量为k，相比全部入堆再弹出，优化时间复杂度；
- 技巧3：桶排序利用频次天然有序特性，无需排序，实现线性时间复杂度；
- 技巧4：字典 `.items()` 同时遍历键值，键为数组元素，值为出现频次。

## ❌ 错题反思 / 踩坑记录
**高频错误：**
1. 堆中存储顺序写反，存成`(num, cnt)`，导致按照数字大小排序而非频次；
2. 堆满条件判断缺失，没有及时pop，堆持续膨胀；
3. 桶数组大小设置过小，造成下标越界；
4. 正向遍历桶，从低频开始收集，结果顺序完全错误。

**修正思路：**
堆元组优先放频次；桶必须逆序从最高频次收集；时刻控制堆长度不超过k。

## ✅ 总结
前K高频元素有两条经典路线：
1. 小根堆方案代码简洁，空间开销可控，面试首选；
2. 桶排序达到理论\(O(n)\)时间复杂度，适合数据规模巨大场景。
两种方案都需要先用哈希表统计频次，区别在于后续筛选前k元素的策略。题目允许任意顺序返回结果，无需额外排序。
