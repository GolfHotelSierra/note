[toc]

# 链表

1. 使用**多个指针**
   - [21. 合并两个有序链表](https://leetcode.cn/problems/merge-two-sorted-lists/)
2. 创建**辅助链表**
   - [86. 分隔链表](https://leetcode.cn/problems/partition-list/)
3. 使用头结点 **dummy head**
   - [86. 分隔链表](https://leetcode.cn/problems/partition-list/)

4. 不依赖前继节点，**通过复制值删除当前节点**

   但是**无法删除最后一个节点**

   - [237. 删除链表中的节点](https://leetcode.cn/problems/delete-node-in-a-linked-list/)

5. 有些链表确实一次循环无法得到结果，需要**多次循环**

   - [138. 随机链表的复制](https://leetcode.cn/problems/copy-list-with-random-pointer/)

6. 链表题目**不代表不可以使用其它的数据结构辅助**，比如哈希表
   - [138. 随机链表的复制](https://leetcode.cn/problems/copy-list-with-random-pointer/)





# 栈与队列

- 栈和队列都可以**使用动态列表代替**
  - [20. 有效的括号](https://leetcode.cn/problems/valid-parentheses/)
- 可以在**栈底添加无意义的元素作为边界**
  - [20. 有效的括号](https://leetcode.cn/problems/valid-parentheses/)





# 其它

1. 当**从字典中获取值**时，可以使用 **`get()` 函数**，可以指定**找不到元素时的默认值**，即使不指定也**会返回 `None` 而不是报错**
