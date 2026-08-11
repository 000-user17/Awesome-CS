# LeetCode 刷题记录

## 📌 题目信息

- **题目编号**：208

- **题目名称**：实现 Trie (前缀树)

- **题目难度**：中等

- **题目链接**：https://leetcode.cn/problems/implement-trie-prefix-tree/description/?envType=problem-list-v2&envId=2cktkvj&

- **刷题日期**：2026\08\11

- **标签**：前缀树 / Trie / 哈希表

## 📝 题目大意

Trie（前缀树）是树形结构，用来高效存储、检索字符串。
要求实现 Trie 三个接口：
1. `Trie()`：初始化前缀树
2. `insert(word)`：插入字符串 word
3. `search(word)`：完整匹配单词，存在返回 true
4. `startsWith(prefix)`：判断是否存在以 prefix 为前缀的单词

## 💡 解题思路

**核心思想：**
使用节点+字典构建多叉树；每个节点维护子节点映射，并用标记区分「单词结尾」和普通前缀节点。

**步骤拆解：**
1. 定义 `TrieNode` 节点结构
   - `children`：字典，key=字符，value=子节点
   - `is_end`：布尔标记，代表当前节点是否是某个单词的最后一个字符

2. insert 插入流程
   - 从根节点出发，逐个遍历字符
   - 字符不存在则新建 TrieNode
   - 移动到对应子节点，遍历结束后将当前节点 `is_end=True`

3. search 完整查找流程
   - 从根节点遍历所有字符，路径中断直接返回 False
   - 字符路径全部走完后，**必须判断 is_end**，区分只是前缀还是完整单词

4. startsWith 前缀查找流程
   - 只需要完整走完前缀字符路径即可，不需要判断 is_end

## ⚙️ 复杂度分析

- **时间复杂度**：单次 insert / search / startsWith 均为 O(L)，L为字符串长度
- **空间复杂度**：O(N * L)，N为单词总数，L平均单词长度；所有不重复字符共用树节点

## 💻 AC 代码（Python）
```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word: str) -> None:
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.is_end = True

    def search(self, word: str) -> bool:
        node = self.root
        for ch in word:
            if ch not in node.children:
                return False
            node = node.children[ch]
        return node.is_end

    def startsWith(self, prefix: str) -> bool:
        node = self.root
        for ch in prefix:
            if ch not in node.children:
                return False
            node = node.children[ch]
        return True

# Your Trie object will be instantiated and called as such:
# obj = Trie()
# obj.insert(word)
# param_2 = obj.search(word)
# param_3 = obj.startsWith(prefix)
```

## 🧩 关键知识点 & 技巧

- 技巧1：区分 `search` 和 `startsWith` 的核心就是 `is_end` 标记
- 技巧2：Trie 优势：大量字符串前缀重复时，节省存储空间；适合自动补全、词典检索
- 易错点：
  1. 不要把 `node` 直接当成字典，必须使用 `node.children[ch]`，否则报 item assignment 错误
  2. search 不能只判断路径存在，忘记判断 `is_end`（例如插入 app，search("ap") 应该返回 false）

## ❌ 错题反思 / 踩坑记录

**错误原因：**
新手容易写出 `node[ch]` 直接赋值，混淆 TrieNode 对象和内部 children 字典，触发 TypeError。
部分人实现 search 时遗漏 `is_end` 判断，导致把前缀误判为完整单词。

**修正思路：**
所有子节点操作都访问 `node.children`；牢记：路径走完 ≠ 存在完整单词。

## ✅ 总结
前缀树本质是共享公共前缀的多叉树。
节点只存一个字符，依靠父子关系拼接完整字符串；
`is_end` 是区分「前缀」和「完整单词」的关键标记。
三个方法逻辑高度相似：遍历字符路径，区别只在于最后是否校验单词结束标记。
