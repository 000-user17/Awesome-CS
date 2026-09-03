# LeetCode 刷题记录
## 📌 题目信息
- **题目编号**：19
- **题目名称**：删除链表的倒数第 N 个结点
- **题目难度**：中等
- **题目链接**：https://leetcode.cn/problems/remove-nth-node-from-end-of-list/
- **刷题日期**：2026\09\03
- **标签**：链表、双指针、虚拟头节点

## 📝 题目大意
给你一个链表，删除链表的倒数第 `n` 个结点，并返回链表的头结点。
> 进阶：要求一趟扫描完成，不允许先遍历统计链表长度再第二趟删除。
> 边界情况：有可能删除的是链表第一个节点。

## 💡 解题思路
**核心思想：虚拟头节点 + 快慢双指针（一次遍历）**
1. 使用`dummy`虚拟哨兵节点，解决要删除头节点的边界，统一所有节点删除逻辑。
2. `fast`快指针先向前走 `n` 步。
3. 之后`fast`、`slow`、`pre`三者同步向后移动，直到`fast`走到链表末尾（fast为None）。
此时`slow`正好指向**待删除节点**，`pre`指向待删除节点的前一个节点。
4. 修改 `pre.next = slow.next`，跳过slow节点，完成删除。
5. 返回`dummy.next`作为新链表头。

> 快慢指针原理：快指针与慢指针保持n的间距；当快指针走到末尾，慢指针就定位到倒数第n个节点。

**步骤拆解：**
1. 创建虚拟头节点`dummy`，`dummy.next = head`；`pre`初始指向dummy，快慢指针slow、fast初始等于head。
2. fast先走n步。
3. fast不为空时循环同步后移：pre=slow；slow=slow.next；fast=fast.next。
4. 循环结束 slow 就是待删节点；`pre.next = slow.next`跳过待删节点。
5. 返回`dummy.next`。

> 为什么需要pre：链表删除节点，必须拿到待删节点的前驱节点，才能修改next指针。

## ⚙️ 复杂度分析
- **时间复杂度**：$\(O(L)\)$，L链表长度，仅遍历链表一遍
- **空间复杂度**：$\(O(1)\)$，只使用若干指针，常数额外空间

## 💻 AC 代码（Python）
```python
# Definition for singly-linked list.
from typing import Optional
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

class Solution:
    def removeNthFromEnd(self, head: Optional[ListNode], n: int) -> Optional[ListNode]:
        slow, fast = head, head
        dummy = ListNode(-1)
        pre = dummy
        dummy.next = head
        # fast 先向前走 n 步
        for i in range(n):
            if not fast:
                return None
            fast = fast.next
        # fast走到末尾，slow落在倒数第n个节点
        while fast:
            pre = slow
            slow = slow.next
            fast = fast.next
        # 删除slow节点
        pre.next = slow.next
        return dummy.next
```

## 🧩 关键知识点 & 技巧
- 技巧1：**虚拟头节点dummy**，专门处理删除头节点的边界，不用单独写if判断头节点特殊逻辑。
- 技巧2：快慢指针，一次遍历定位倒数第n个节点，满足题目进阶要求。
- 技巧3：必须保存前驱节点`pre`；单向链表无法回退，没有前驱就无法删除当前节点。

- 易错点：
  1. 不使用dummy，直接操作head，当删除第一个节点时逻辑出错。
  2. fast先走n步之后，忘记同步维护pre指针，拿不到待删节点的前一个节点。
  3. 循环结束直接修改slow.next，这只是修改slow本身，没有修改前驱节点的next，链表没有真正删掉节点。
  4. 返回head而不是`dummy.next`，当head被删掉会返回旧的无效头。

## ❌ 错题反思 / 踩坑记录
**错误原因：**
1. 不设置虚拟头节点，遇到删除头节点的case处理失败。
2. 只设置快慢指针，没有pre前驱指针，找不到待删节点前面的节点，无法完成删除。
3. 错误写`slow.next = slow.next.next`，slow本身还挂在链表上，实际并未脱离链表。
4. fast移动步数写错，快慢指针间隔不对，定位节点偏移。

**修正思路：**
1. 涉及链表删除，优先考虑dummy哨兵，统一边界。
2. 单向链表删除节点，一定要拿到前驱节点，修改`pre.next`。
3. fast先跑n步，再一起跑；最终slow是待删点，pre是它前面节点。
4. 返回`dummy.next`，不要返回原来head。

## ✅ 总结
19 删除链表倒数第N个结点，快慢指针经典题。
> dummy哨兵处理删头；fast先走n步；fast到末尾时slow指向待删节点；pre保存前驱；`pre.next = slow.next`完成删除，返回`dummy.next`。

> 补充另一种常规思路：先遍历统计链表长度L，再正向走到L‑n位置做删除；该方法需要两次遍历，不能满足题目进阶要求。
