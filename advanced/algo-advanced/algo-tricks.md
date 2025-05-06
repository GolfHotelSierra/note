[toc]

# 链表

- 使用**多个指针**
  - [21. 合并两个有序链表](https://leetcode.cn/problems/merge-two-sorted-lists/)
- 创建**辅助链表**
  - [86. 分隔链表](https://leetcode.cn/problems/partition-list/)

- 使用头结点 **dummy head**
  - [86. 分隔链表](https://leetcode.cn/problems/partition-list/)

- 不依赖前继节点，**通过复制值删除当前节点**

  但是**无法删除最后一个节点**

  - [237. 删除链表中的节点](https://leetcode.cn/problems/delete-node-in-a-linked-list/)

- 有些链表确实一次循环无法得到结果，需要**多次循环**
  - [138. 随机链表的复制](https://leetcode.cn/problems/copy-list-with-random-pointer/)

- 链表题目**不代表不可以使用其它的数据结构辅助**，比如哈希表
  - [138. 随机链表的复制](https://leetcode.cn/problems/copy-list-with-random-pointer/)

- **将数组看做链表，将 nums[index] = value，看做 index -> value** + **抽象链表成环**
  
  - [287. 寻找重复数](https://leetcode.cn/problems/find-the-duplicate-number/)
  
- 涉及**反转问题**，尝试**多指针**或**递归**
  - [LCR 141. 训练计划 III](https://leetcode.cn/problems/fan-zhuan-lian-biao-lcof/)





# 栈与队列

- 栈和队列都可以**使用动态列表代替**
  - [20. 有效的括号](https://leetcode.cn/problems/valid-parentheses/)
- 可以在**栈底添加无意义的元素作为边界**
  - [20. 有效的括号](https://leetcode.cn/problems/valid-parentheses/)

- 当只关心最大、最小值时，可以尝试**堆**
  - [295. 数据流的中位数](https://leetcode.cn/problems/find-median-from-data-stream/)





# 哈希表

- 似乎**字符串**问题很喜欢用**哈希表**
  - [409. 最长回文串](https://leetcode.cn/problems/longest-palindrome/)
  - [205. 同构字符串](https://leetcode.cn/problems/isomorphic-strings/)





# 双指针

- **快慢指针**
  - [876. 链表的中间结点](https://leetcode.cn/problems/middle-of-the-linked-list/)
- **循环指针**
  - [160. 相交链表](https://leetcode.cn/problems/intersection-of-two-linked-lists/)
- **双指针交替移动**
  - [167. 两数之和 II - 输入有序数组](https://leetcode.cn/problems/two-sum-ii-input-array-is-sorted/)
- **快慢指针 + 循环指针 + 替代未知值 (本质是让快慢指针的路程相等)**
  - [142. 环形链表 II](https://leetcode.cn/problems/linked-list-cycle-ii/)
- **滑动窗口 + 排序  + 三指针 (去重)**
  - [15. 三数之和](https://leetcode.cn/problems/3sum/)
- **递减队列 (滑动窗口最大值) + 滑动窗口**
  - [239. 滑动窗口最大值](https://leetcode.cn/problems/sliding-window-maximum/)





# 模拟

- **循环序列 -> 子序列是否在`原序列 * n`中**
  - [796. 旋转字符串](https://leetcode.cn/problems/rotate-string/)
- **指定形状的字符串 -> `res[i]` 表示第 i 行的字符串 + 定期从 +step 转为 -step**
  - [6. Z 字形变换](https://leetcode.cn/problems/zigzag-conversion/)

- **线性路径 + 二维数组 -> 边界 + 移动**
  - [54. 螺旋矩阵](https://leetcode.cn/problems/spiral-matrix/)
- **旋转 + 二维数组 -> 顺时针旋转 90 度，`matrix[i][j]` 移动到 `matrix[j][n-1-i]`**
  - [48. 旋转图像](https://leetcode.cn/problems/rotate-image/)
- **字符串转数字 -> 一位数字 `x = ascii(num) - ascii('0')` + 整个数字 `res = res*10 + x`**
  - [8. 字符串转换整数 (atoi)](https://leetcode.cn/problems/string-to-integer-atoi/)





# 查找

- 可以优先考虑**二分查找 (包括部分有序)**

  但如果**序列完全无序**，大概率不适合二分查找

  - [154. 寻找旋转排序数组中的最小值 II](https://leetcode.cn/problems/find-minimum-in-rotated-sorted-array-ii/)

- **找到数组中的重复整数 <==> 找到链表的环入口**

  - [287. 寻找重复数](https://leetcode.cn/problems/find-the-duplicate-number/)
  
- **通过 i-1 和 i+1 防止二分查找在某个 if-else 判断中死循环**

  - [154. 寻找旋转排序数组中的最小值 II](https://leetcode.cn/problems/find-minimum-in-rotated-sorted-array-ii/)






# 搜索

- 常见的是**二叉树的搜索**
  - **BFS**
  - **DFS**
- 尝试**分情况讨论**

- **使用返回值返回左右子树的信息** (用 `None` 表示无信息)
  - [236. 二叉树的最近公共祖先](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree/)
- **使用返回值作为提前返回的标志**
  - [236. 二叉树的最近公共祖先](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree/)
  - [230. 二叉搜索树中第 K 小的元素](https://leetcode.cn/problems/kth-smallest-element-in-a-bst/)
- **二叉搜索树先考虑中序遍历**
  - [230. 二叉搜索树中第 K 小的元素](https://leetcode.cn/problems/kth-smallest-element-in-a-bst/)
- **第 K 大的元素 -> 长度为 K 的小顶堆，堆顶为答案**
  - [215. 数组中的第K个最大元素](https://leetcode.cn/problems/kth-largest-element-in-an-array/)





# 回溯

- **排列问题 -> (排序) + 回溯问题**
  - **`selected` 数组**
  - **`duplicated` 数组 (排序后，也可以用跳过重复元素替代)**
- **使用返回值返回左右子树的信息**
  - [104. 二叉树的最大深度](https://leetcode.cn/problems/maximum-depth-of-binary-tree/)





# 分治

- **树结构可以考虑分治算法**





# 动态规划

- **状态转移方程的核心是转移**，不是方程
  - [121. 买卖股票的最佳时机](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/)
- **动态循环结束后，通过额外操作 (e.g. 求最大) 才得到答案**
  - [53. 最大子数组和](https://leetcode.cn/problems/maximum-subarray/)
- **多次动态循环**
  - [213. 打家劫舍 II](https://leetcode.cn/problems/house-robber-ii/)
- **多层循环 + 动态规划**
  - [300. 最长递增子序列](https://leetcode.cn/problems/longest-increasing-subsequence/)
- **三指针 (去重) + 动态规划**
  - [264. 丑数 II](https://leetcode.cn/problems/ugly-number-ii/)





# 贪心

- **二维数组入口选取 + 行列删除 + 贪心**
  - [240. 搜索二维矩阵 II](https://leetcode.cn/problems/search-a-2d-matrix-ii/)
- **正向遍历 + 反向遍历**
  - [135. 分发糖果](https://leetcode.cn/problems/candy/)
- **递增队列**
  - [768. 最多能完成排序的块 II](https://leetcode.cn/problems/max-chunks-to-make-sorted-ii/)





# 位运算

- **`n//2 <==> n>>1`；`n*2 <==> n<<1`**
- **`奇数 <==> n&1=1`;`偶数 <==> n&1=0`**
- **`n为2的幂 <==> n & (n - 1) == 0`**

- **两个值是否相等 -> 异或运算**





# 数学







# 其它

1. **读题！！！**
2. 当**从字典中获取值**时，可以使用 **`get()` 函数**，可以指定**找不到元素时的默认值**，即使不指定也**会返回 `None` 而不是报错**
3. 使用 **`defaultdict()` (python) 指定默认值**
4. 使用**方向矩阵 `directs = [(0, 1), (0, -1), (1, 0), (-1, 0)]`**
5. **层序遍历可以更好地保存树的信息**
6. **滑动窗口**
