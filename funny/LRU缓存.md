# [LRU缓存](https://leetcode-cn.com/problems/lru-cache/)

## 题目描述

运用你所掌握的数据结构，设计和实现一个  LRU (最近最少使用) 缓存机制。它应该支持以下操作： 获取数据 get 和 写入数据 put 。

获取数据 get(key) - 如果密钥 (key) 存在于缓存中，则获取密钥的值（总是正数），否则返回 -1。
写入数据 put(key, value) - 如果密钥不存在，则写入其数据值。当缓存容量达到上限时，它应该在写入新数据之前删除最久未使用的数据值，从而为新的数据值留出空间。

进阶:

你是否可以在 O(1) 时间复杂度内完成这两种操作？

示例:

```go
LRUCache cache = new LRUCache( 2 /* 缓存容量 */ );

cache.put(1, 1);
cache.put(2, 2);
cache.get(1);       // 返回  1
cache.put(3, 3);    // 该操作会使得密钥 2 作废
cache.get(2);       // 返回 -1 (未找到)
cache.put(4, 4);    // 该操作会使得密钥 1 作废
cache.get(1);       // 返回 -1 (未找到)
cache.get(3);       // 返回  3
cache.get(4);       // 返回  4
```

## 解题思路

首先需要了解一下LRU缓存的工作机制：

例如访问序列`7 0 1 2 0 3 0 4`，LRU缓存大小为3，则LRU缓存中数据分布如下：

|      |      | 1    | 2    | 0    | 3    | 0    | 4    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
|      | 0    | 0    | 1    | 2    | 0    | 3    | 0    |
| 7    | 7    | 7    | 0    | 1    | 2    | 2    | 3    |

**LRU缓存特点**：

- 缓存头部存放最近访问的元素；缓存尾部则是缓存中最久没被访问过的元素
- 插入元素在缓存头部；当缓存满时，删除缓存尾部元素
- 查询元素时，则应该将该元素移动到头部

**结构设计：**

- Get操作O(1)复杂度：**HashMap查询key**
- Put操作O(1)复杂度：**链表需要头尾指针 - 双向链表，方便移动**

**因此在本题中，用HashMap来查询key，而在HashMap中key对应的value则是链表中该kv对所在的节点**

```go
type LRUCache struct {
    // 存放kv对的双向链表
    lruList *list.List

    // 存放key与该kv对所在的链表结点
    hashMap map[int](*list.Element)

    // 当前缓存大小
    len int

    // 缓存最大容量
    capacity int
}

// 链表结点
type entry struct {
    key int
    value int
}


func Constructor(capacity int) LRUCache {
    // 构建空的LRU缓存，并设置其最大容量
    return LRUCache {
        lruList: list.New(),
        hashMap: make(map[int](*list.Element)),
        len: 0,
        capacity: capacity,
    }
}


func (this *LRUCache) Get(key int) int {
    if _,ok := this.hashMap[key]; ok {
        // LRU中存在该key

        // 移动该key对应的结点至链表头部
        this.lruList.MoveToFront(this.hashMap[key])

        // 获取value值并返回
        resEntry := this.hashMap[key]
        return resEntry.Value.(*entry).value
    } else {
        // 不存在，返回-1
        return -1
    }
}


func (this *LRUCache) Put(key int, value int)  {
    if _,ok := this.hashMap[key]; ok {
        // LRU中已有，则更新值，并移动到头部
        this.hashMap[key].Value.(*entry).value = value
        this.lruList.MoveToFront(this.hashMap[key])
        return
    } else {
        // LRU中不存在该key，添加到LRU中
        // 首先需要判断当前LRU是否存在空余空间以容纳新的元素
        if this.len == this.capacity {
            // 缓存已满，则需要删除最后即最久未被访问的kv对结点
            keyTail := this.lruList.Back().Value.(*entry).key
            this.lruList.Remove(this.lruList.Back())
            delete(this.hashMap, keyTail)
            this.len--
        }
        
        // 插入新元素
        ele := entry{key, value}
        this.lruList.PushFront(&ele)
        this.len++
        this.hashMap[key] = this.lruList.Front()
    }
}


/**
 * Your LRUCache object will be instantiated and called as such:
 * obj := Constructor(capacity);
 * param_1 := obj.Get(key);
 * obj.Put(key,value);
 */
```



