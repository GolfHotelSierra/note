[toc]

# Tokenization

> character: 字符 (e.g. "A") < word: 单词 (e.g. "Apple")

- 因为 nlp 的大多数模型只能处理“数字”而不能处理“文本”所以需要一种映射的方式，称作 **tokenization**，一个“数字”对应的一个单位的文本 (e.g. 一个 character、数个 character、一个 word 等等)，称作 **token**

> tokenization 一般是



## One-Hot

> 准确地说，One-Hot 不是某一种具体的 tokenization 方式，不论 token 是 character、sub-word、word 等，都可以用 One-Hot 作为“文本”和“数字”间的”桥梁“

1. 将不同的 word 对应到一个唯一的 id 上

2. 将这个 id 变成一个二进制编码，编码规则是 `index=id` 那一位设置为“1”，而其他位设置为“0”

   ```
   e.g. [[1,0,0],[0,1,0],[0,0,1]]
   ```



## BPE (Byte Pair Encoding)

- BPE 可以看做是 sub-word level 的分词方式

- 假设对于文本 "aaabdaaabac"

  1. 相邻字节对中 "aa" 最常出现，用一个新字节 "Z" 替换

  2. 对于 "ZabdZabac"，下一个常见的字节对是 "ab"，用 "Y" 替换

  3. 以此类推，为了进一步压缩，还可以递归地将 "ZY" 替换为 "X"

     最终得到 "XdXac"，然后可以以 character-level 做 One-Hot 等

  4. 对于生成出的结果，按照映射关系“解码“即可





# Embedding

- Embedding 的基本目标，
  - 是让如 One-Hot 得到的高维编码 (e.g. token 表长度一般在 $10^3$ 数量级) 进行<u>*降维*</u>
  - 让拥有相同含义的 token 在低维空间中尽量靠近 (e.g. "dog" 和 "cat"、"car" 和 "bus" 等)





# 参考资料

- [知乎上对 BPE 的介绍，其中有一个简单的 BPE 计算的例子](https://www.zhihu.com/search?type=content&q=BPE)