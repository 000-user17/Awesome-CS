# LeetCode 刷题记录
## 📌 题目信息
- **题目编号**：23
- **题目名称**：合并 K 个升序链表
- **题目难度**：困难
- **题目链接**：https://leetcode.cn/problems/merge-k-sorted-lists/
- **刷题日期**：2026\09\02
- **标签**：链表、分治、优先队列(堆)

## 📝 题目大意
给一个链表数组，数组中每一条链表都是**升序有序**。把全部链表合并成一条升序链表，返回合并后链表头节点。

> 重点难点：
> 1. k条链表，不是简单的2条链表合并；
> 2. 暴力逐个合并时间效率差；
> 3. 两种主流解法：**分治归并**、**最小堆(优先队列)**；
> 4. 需要熟悉基础：合并两个有序链表 mergeTwoLists。

## 💡 解题思路
### 思路1：分治归并（推荐，面试高频）
借鉴归并排序思想：
1. 将k个链表数组，不断两两分组；
2. 每组调用`mergeTwoLists`合并两个升序链表；
3. 合并后的新链表放回数组，继续两两合并；
4. 不断二分缩小规模，最后得到1条链表即为答案。

> 原理：
合并k条链表，等价于归并排序，把链表数组不断对半拆分，递归合并左右两部分。

**子函数 mergeTwoLists( l1, l2 )**
- 虚拟哨兵头结点；
- 双指针依次比较两个链表节点，把较小节点接到结果链表；
- 返回合并完成链表。

**分治递归步骤：**
1. 如果 left > right：返回 None
2. 如果 left == right：返回 lists[left]
3. mid = (left + right) // 2
4. leftPart = 递归合并 [left, mid]
5. rightPart = 递归合并 [mid+1, right]
6. return mergeTwoLists(leftPart, rightPart)

### 思路2：最小堆（优先队列 heapq）
1. 把每条链表的头节点入小顶堆；堆按节点val排序；
2. 新建虚拟头哨兵，tail指针指向结果链表尾部；
3. 循环弹出堆中val最小节点，接到tail后面；
4. 如果弹出的节点还有next后继，把后继节点压入堆；
5. 堆为空结束，返回虚拟头的next。

> 注意python heapq不能直接存节点，需要存 (val, id(node), node) 避免节点比较报错。

### 思路3：暴力（不推荐，仅理解）
循环遍历，依次把链表逐个合并：`res = mergeTwoLists(res, lists[i])`
时间差，k很大时会超时。

## ⚙️ 复杂度分析
- **分治归并**
时间复杂度：$O(N \log k)$，N全部节点总个数，k链表数量；每个节点被logk次合并；
空间复杂度：$O(\log k)$，递归栈开销；迭代版分治可以做到O(1)。

- **最小堆解法**
时间复杂度：$O(N \log k)$，每个节点入堆出堆一次，堆大小最多k；
空间复杂度：$\(O(k)\)$，堆最多存放k个节点。

## 💻 AC 代码：分治递归（推荐）
```python
from typing import List, Optional
# Definition for singly‑linked list.
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

class Solution:
    def mergeKLists(self, lists: List[Optional[ListNode]]) -> Optional[ListNode]:
        def mergeTwo(l1: Optional[ListNode], l2: Optional[ListNode]) -> Optional[ListNode]:
            dummy = ListNode(-1)
            cur = dummy
            while l1 and l2:
                if l1.val < l2.val:
                    cur.next = l1
                    l1 = l1.next
                else:
                    cur.next = l2
                    l2 = l2.next
                cur = cur.next
            cur.next = l1 if l1 else l2
            return dummy.next

        def divide(left: int, right: int) -> Optional[ListNode]:
            if left > right:
                return None
            if left == right:
                return lists[left]
            mid = (left + right) // 2
            l = divide(left, mid)
            r = divide(mid + 1, right)
            return mergeTwo(l, r)

        return divide(0, len(lists)-1)
```

## 💻 AC 代码：最小堆 heapq
```python
import heapq
from typing import List, Optional
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

class Solution:
    def mergeKLists(self, lists: List[Optional[ListNode]]) -> Optional[ListNode]:
        heap = []
        for node in lists:
            if node:
                heapq.heappush(heap, (node.val, id(node), node))
        dummy = ListNode(-1)
        tail = dummy
        while heap:
            _, _, cur = heapq.heappop(heap)
            tail.next = cur
            tail = tail.next
            if cur.next:
                heapq.heappush(heap, (cur.next.val, id(cur.next), cur.next))
        return dummy.next
```

## 🧩 关键知识点 & 技巧
1. **基础：合并两个有序链表是本体子模块**，一定要熟练；
2. 分治：数组对半拆分，递归合并左右，时间最优，面试优先写分治；
3. 堆解法：利用小顶堆每次取k个链表头部最小元素；python heapq不能直接比较对象，存入`(val, id(node), node)`规避报错；
4. 虚拟头(dummy哨兵)：合并链表必用，省去处理空头边界；
5. 边界：lists数组为空；数组内链表为空节点None，要过滤。

## ❌ 错题反思 / 踩坑记录
**高频错误**
1. 直接暴力循环逐个合并链表，时间复杂度退化O(Nk)，大数据样例超时；
2. 分治递归边界写错：left>right返回None；left==right直接返回lists[left]；
3. mergeTwoLists没有用dummy哨兵，处理头节点逻辑繁琐，空链表报错；
4. heapq直接push节点对象，python会尝试比较ListNode实例，抛异常；
5. 入堆时没有过滤None链表，把None压入堆，pop后访问.val空指针；
6. 堆弹出节点后，忘记把node.next压回堆，链表只取头节点。

**避坑要点**
- 优先掌握分治归并，时间优秀，不依赖堆API；
- mergeTwoLists子函数写正确，分治本体只是递归框架；
- 堆解法存入元组`(val, id, node)`，id用来当val相等时的第二比较项；
- 所有链表合并，优先使用dummy虚拟头结点。

## ✅ 总结
合并k个升序链表困难题，两套主流解法：
1. **分治归并**：把链表数组二分，两两合并，$O(N\log k)$，面试首选；
2. **最小堆**：维护k个链表头部，每次取出最小值；适合理解多路归并。

> 拓展：多路归并思想也用于外部排序。

需要我顺带把**合并两个有序链表**也生成一份完整刷题模板吗。
