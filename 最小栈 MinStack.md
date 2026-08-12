# LeetCode 刷题记录
## 📌 题目信息
- **题目编号**：155
- **题目名称**：最小栈 MinStack
- **题目难度**：简单
- **题目链接**：https://leetcode.cn/problems/min-stack/
- **刷题日期**：2026\08\12
- **标签**：栈 / 辅助栈

## 📝 题目大意
设计一个支持 push ，pop ，top 操作，并能在常数时间内检索到最小元素的栈。
实现 MinStack 类：
1. `MinStack()` 初始化堆栈对象。
2. `void push(int value)` 将元素 value 推入堆栈。
3. `void pop()` 删除堆栈顶部的元素。
4. `int top()` 获取堆栈顶部的元素。
5. `int getMin()` 获取堆栈中的最小元素。

要求：`getMin()` 需要在 **\(O(1)\)** 时间复杂度完成。

> 普通栈只能快速获取栈顶元素，无法直接拿到最小值，因此引入辅助栈保存每一步对应的最小值。

## 💡 解题思路
### 核心思想：双栈同步方案
维护两个栈：
1. `stack`：主栈，正常存放所有入栈元素；
2. `min_stack`：辅助栈，**和主栈保持相同长度**，每一位保存「主栈当前位置及之前所有元素的最小值」。

**步骤拆解**
1. push 操作：
   - value 压入主栈；
   - 若辅助栈为空，最小值就是 value；否则取 `min(value, min_stack[-1])` 压入辅助栈。
2. pop 操作：主栈、辅助栈同时弹出栈顶；
3. top 操作：直接返回主栈栈顶；
4. getMin 操作：直接返回辅助栈栈顶。

示例流程：
依次 push -2，0，-3
stack：[-2, 0, -3]
min_stack：[-2, -2, -3]
getMin() → -3
pop()
stack：[-2, 0]
min_stack：[-2, -2]
getMin() → -2

## ⚙️ 复杂度分析
- **时间复杂度**：push / pop / top / getMin 全部为 $\(O(1)\)$
- **空间复杂度**：$\(O(n)\)$，需要额外开辟辅助栈空间

## 💻 AC 代码
```python
class MinStack:

    def __init__(self):
        self.stack = []
        self.min_stack = []

    def push(self, value: int) -> None:
        self.stack.append(value)
        self.min_stack.append(min(value, value if not self.min_stack else self.min_stack[-1]))
        

    def pop(self) -> None:
        self.stack.pop()
        self.min_stack.pop()

    def top(self) -> int:
        return self.stack[-1]

    def getMin(self) -> int:
        return self.min_stack[-1]

# Your MinStack object will be instantiated and called as such:
# obj = MinStack()
# obj.push(value)
# obj.pop()
# param_3 = obj.top()
# param_4 = obj.getMin()
```

## 🧩 关键知识点 & 技巧
1. 双栈同步写法逻辑直观，面试上手最快，不容易出错；
2. push 内三元表达式作用：判断辅助栈是否为空，空时直接取 value，否则对比当前值与辅助栈栈顶最小值；
3. 辅助栈与主栈长度严格一致，pop 时两个栈一起弹出，无需额外判断；
4. 不能每次调用 getMin 遍历栈查找最小值，会退化到 $\(O(n)\)$，不满足题目要求。

> 拓展优化思路：
> 可以优化辅助栈空间，仅当新元素 ≤ 当前最小值时才入栈，pop 时只有弹出元素等于辅助栈栈顶才弹出辅助栈。节省空间，但代码条件更多。

## ❌ 错题反思 / 踩坑记录
**常见错误**
1. pop 只弹出主栈，忘记同步弹出辅助栈，后续 getMin 结果错乱；
2. push 时辅助栈判断条件写错，空栈场景直接访问 `min_stack[-1]`，触发下标越界；
3. 尝试用一个变量保存最小值，一旦最小值被 pop 弹出，无法快速找到次小值；
4. 混淆方法返回值：题目规定 pop() 无返回值，不要写成 return pop()。

**避坑要点**
- 辅助栈必须记录每一步状态，不能只存全局单一最小值；
- 访问栈顶前要保证栈非空（LeetCode 测试用例保证操作合法，无需额外判空）；
- 三元表达式 `value if not self.min_stack else self.min_stack[-1]` 优先判断空栈，防止索引报错。

## ✅ 总结
本题是栈经典面试题，核心解法为**辅助最小栈**。
双栈同步方案代码简洁、容错率高，优先作为面试标准答案。
本质思路：用额外空间记录历史状态，换取查询最小值的常数时间。

---

# 📎 拓展：空间优化版辅助栈（减少冗余存储）
## 优化思路
原版辅助栈会存储大量重复最小值，造成空间浪费。
优化规则：
1. push：**只有新元素 ≤ 当前最小值**，才压入辅助栈；
2. pop：取出主栈弹出元素，如果该元素 == 辅助栈栈顶，辅助栈才弹出；
3. getMin：依旧直接返回辅助栈栈顶。

> ⚠️ 关键条件必须是 `<=`，不能只写 `<`
> 例：连续 push 两个相同最小值 [-2, -2]，如果只用 `<`，辅助栈只存一个 -2；第一次 pop 后辅助栈被弹出，最小值丢失。

## 💻 空间优化版代码
```python
class MinStack:
    def __init__(self):
        self.stack = []
        self.min_stack = []

    def push(self, value: int) -> None:
        self.stack.append(value)
        # 小于等于当前最小值才入辅助栈
        if not self.min_stack or value <= self.min_stack[-1]:
            self.min_stack.append(value)

    def pop(self) -> None:
        top_val = self.stack.pop()
        # 弹出的值等于当前最小值，辅助栈同步弹出
        if top_val == self.min_stack[-1]:
            self.min_stack.pop()

    def top(self) -> int:
        return self.stack[-1]

    def getMin(self) -> int:
        return self.min_stack[-1]
```

## 优化对比
- 原版同步双栈：辅助栈长度 = 主栈长度，重复值全部保存；编码简单，bug少；
- 空间优化版本：辅助栈长度 ≤ 主栈长度，消除冗余；面试可作为加分拓展回答；
- 时间复杂度不变：所有操作依旧 \(O(1)\)。
