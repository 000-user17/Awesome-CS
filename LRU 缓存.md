# LeetCode 刷题记录
## 📌 题目信息
- **题目编号**：146
- **题目名称**：LRU 缓存 LRU Cache
- **题目难度**：中等
- **题目链接**：https://leetcode.cn/problems/lru-cache/
- **刷题日期**：2026\08\13
- **标签**：双向链表 / 哈希表 / 设计题

## 📝 题目大意
请你设计并实现一个满足 LRU (最近最少使用) 缓存约束的数据结构。
实现 `LRUCache` 类：
1. `LRUCache(int capacity)`：以正整数作为容量初始化 LRU 缓存
2. `int get(int key)`：关键字存在则返回值，不存在返回 -1
3. `void put(int key, int value)`
   - key已存在：更新数值，并标记为最近使用
   - key不存在：插入键值对；若超出容量，逐出**最久未使用**的数据
要求：`get`、`put` 平均时间复杂度 $\(O(1)\)$。

> 分析：
> - 哈希表：可以 $\(O(1)\)$ 根据 key 查找节点，但无法维护使用时序；
> - 普通单向链表：无法 $\(O(1)\)$ 删除中间节点；
> - **双向链表 + 哈希表** 组合：
>   双向链表维护访问顺序；哈希表建立 `key → 链表节点` 的映射。
> - 约定：靠近虚拟头结点 = 最近使用；靠近虚拟尾结点 = 最久未使用。

## 💡 解题思路
### 核心架构
1. **双向链表（带虚拟头 head、虚拟尾 tail 哨兵）**
   哨兵节点消除大量空链表边界判断；
2. **哈希字典 `self.dict`**
   存储 key 到 ListNode 的映射，快速定位节点；
3. 基础操作封装：
   - `moveToHead(node)`：将已存在节点移动到链表头部（标记为最近使用）
   - `removeFromTail()`：删除尾部前面节点（淘汰最久未使用，返回被删节点）

### 执行逻辑
**get(key)**
1. key不在字典 → 返回 -1
2. key存在 → 取出节点，移动到链表头部，返回节点值

**put(key, value)**
1. key已存在：更新节点值，移动到头部，直接返回
2. key不存在：
   - 缓存已满：删除尾节点，同时字典 pop 对应 key
   - 新建节点，存入字典，插入链表头部

**关键细节**
- ListNode 必须同时保存 `key` 和 `val`；淘汰尾节点时，需要节点内的 key 删除字典元素；
- 移动节点前，先把节点从原有链表位置断开，再头插；
- 虚拟头尾永不删除，只操作中间业务节点。

## ⚙️ 复杂度分析
- **时间复杂度**：$\(O(1)\)$
  get、put 内所有操作：哈希查找、双向链表指针修改均为常数时间
- **空间复杂度**：$\(O(capacity)\)$
  最多存放 capacity 个节点，哈希表与链表占用等量空间

## 💻 AC 代码
```python
class ListNode:
    def __init__(self, key, val, next = None, pre = None):
        self.val = val
        self.key = key
        self.next = next
        self.pre = pre

class LRUCache:

    def __init__(self, capacity: int):
        self.capacity = capacity
        self.dict = {}
        # 虚拟头、虚拟尾哨兵
        self.head = ListNode(-1, -1)
        self.tail = ListNode(-1, -1)
        self.head.next = self.tail
        self.tail.pre = self.head

    def get(self, key: int) -> int:
        if key not in self.dict:
            return -1
        node = self.dict[key]
        self.moveToHead(node)
        return node.val

    def put(self, key: int, value: int) -> None:
        if key in self.dict:
            node = self.dict[key]
            self.moveToHead(node)
            node.val = value
            return
        # 容量已满，淘汰最久未使用
        if len(self.dict) == self.capacity:
            node = self.removeFromTail()
            if node:
                self.dict.pop(node.key)
        # 创建新节点插入
        node = ListNode(key, value)
        self.dict[key] = node
        self.moveToHead(node)

    def moveToHead(self, node: ListNode):
        # 先将节点从原位置断开
        if node.pre:
            node.pre.next = node.next
        if node.next:
            node.next.pre = node.pre
        # 头插，放到head之后
        node.next = self.head.next
        node.pre = self.head
        self.head.next.pre = node
        self.head.next = node

    def removeFromTail(self):
        # 链表只有哨兵，无数据节点
        if self.tail.pre == self.head:
            return None
        node = self.tail.pre
        self.tail.pre = node.pre
        node.pre.next = self.tail
        return node

    def printDict(self):
        """调试：打印字典内key与对应value"""
        for k, v in self.dict.items():
            print(" ".join([str(k), str(v.val)]))

# Your LRUCache object will be instantiated and called as such:
# obj = LRUCache(capacity)
# param_1 = obj.get(key)
# obj.put(key,value)
```

## 🧩 关键知识点 & 技巧
1. **设计题标准套路：哈希表 + 双向链表**，LFU、LRU 系列高频组合；
2. 虚拟头尾哨兵极大简化边界，不用反复判断 `pre`/`next` 是否为空；
3. 节点内必须存储 key：链表淘汰节点时，无法反向从 value 找到哈希表的 key；
4. 操作顺序：移动节点「先断开、再插入」，顺序不能颠倒；
5. Python 内置 `OrderedDict` 可以一行实现，但面试要求手写双向链表时不能使用。

## ❌ 错题反思 / 踩坑记录
**高频错误（你最初写代码遇到的问题）**
1. 变量混淆：`self.dict[key]` 错写成 `dict[key]`，`dict` 是内置类型，直接触发 AttributeError；
2. ListNode 只存 val，没有保存 key，淘汰尾部节点后无法删除字典内键；
3. put 函数未判断 key 重复，重复 key 不断新建节点，字典与链表数据不一致；
4. 淘汰尾节点时，忘记执行 `self.dict.pop()`，字典长度持续膨胀；
5. `moveToHead` 没有先断开节点原有前后指针，造成链表指针混乱、出现环；
6. 调试打印函数直接遍历 `self.dict`，没有使用 `.items()`，类型报错。
7. 未考虑put进来已有key的情况

**避坑要点**
- 类成员字典尽量不要命名 `self.dict`，和内置 dict 重名，建议改为 `self.cache`；
- 任何节点移动逻辑：**先脱离原链表，再挂载到新位置**；
- put 操作分支顺序：优先判断 key 是否存在，再处理容量溢出；
- 双向链表删除节点后，记得同步维护哈希表。

## ✅ 总结
LRU 是后端、算法面试超级高频**设计题**，不只是链表知识点，重点考察：
1. 数据结构组合思想；
2. 双向链表指针操作的细心程度；
3. 哈希表与链表双向数据一致性维护。
