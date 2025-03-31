[208. 实现 Trie (前缀树)](https://leetcode.cn/problems/implement-trie-prefix-tree/)

> **[Trie](https://baike.baidu.com/item/字典树/9825209?fr=aladdin)**（发音类似 "try"）或者说 **前缀树** 是一种树形数据结构，用于高效地存储和检索字符串数据集中的键。这一数据结构有相当多的应用情景，例如自动补全和拼写检查。
>
> 请你实现 Trie 类：
>
> - `Trie()` 初始化前缀树对象。
> - `void insert(String word)` 向前缀树中插入字符串 `word` 。
> - `boolean search(String word)` 如果字符串 `word` 在前缀树中，返回 `true`（即，在检索之前已经插入）；否则，返回 `false` 。
> - `boolean startsWith(String prefix)` 如果之前已经插入的字符串 `word` 的前缀之一为 `prefix` ，返回 `true` ；否则，返回 `false` 。

---

简单暴力，直接开撕

```go
type Trie struct {
    Children [26]*Trie
    IsEnd bool
}


func Constructor() Trie {
    return Trie{}
}


func (t *Trie) Insert(word string) {
    for i := 0; i < len(word); i ++ {
        if t.Children[int(word[i] - 'a')] == nil {
            t.Children[int(word[i] - 'a')] = &Trie{
        }}
        t = t.Children[int(word[i] - 'a')]
    }
    t.IsEnd = true
}


func (t *Trie) Search(word string) bool {
    for i := 0; i < len(word); i ++ {
        t = t.Children[int(word[i] - 'a')]
        if t == nil {
            return false
        }
    }
    return t.IsEnd
}


func (t *Trie) StartsWith(prefix string) bool {
    for i := 0; i < len(prefix); i ++ {
        t = t.Children[int(prefix[i] - 'a')]
        if t == nil {
            return false
        }
    }
    return true
}
```

