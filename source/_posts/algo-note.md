---
title: algo-note
date: 2024-04-08 10:00:00
tags:
  - 算法
  - 数据结构
---

一、数据结构模板数组问题41. 缺失的第一个正整数给你一个未排序的整数数组 `nums` ，请你找出其中没有出现的最小的正整数。

请你实现时间复杂度为 `O(n)` 并且只使用常数级别额外空间的解决方案。

示例 1：

```
输入：nums = [1,2,0]
输出：3
```

示例 2：

```
输入：nums = [3,4,-1,1]
输出：2
```

示例 3：

```
输入：nums = [7,8,9,11,12]
输出：1
```

```
class Solution &#123;
    public int firstMissingPositive(int[] nums) &#123;
        //最小的正整数，遍历
        int len = nums.length;
        for (int i = 0; i < len; i++) &#123;
            while (nums[i]>0 && nums[i]<=len && nums[i]!=nums[nums[i]-1])&#123;
                //  1 2 3 0 4
                //  - - - ! -
                // 4应该在下标3的位置
                int j = nums[i] - 1;
                int temp = nums[i];
                nums[i] = nums[j];
                nums[j] = temp;
            &#125;
        &#125;
        for (int i = 0; i < len; i++) &#123;
            if (nums[i]!=i+1)&#123;
                return i+1;//如果当前位置不存在对应的合理的值，就返回
            &#125;
        &#125;
        return len+1;
    &#125;
&#125;

```

链表问题146. LRU 缓存请你设计并实现一个满足 LRU (最近最少使用) 缓存 约束的数据结构。

实现 `LRUCache` 类：

- `LRUCache(int capacity)` 以 正整数 作为容量 `capacity` 初始化 LRU 缓存
- `int get(int key)` 如果关键字 `key` 存在于缓存中，则返回关键字的值，否则返回 `-1` 。
- `void put(int key, int value)` 如果关键字 `key` 已经存在，则变更其数据值 `value` ；如果不存在，则向缓存中插入该组 `key-value` 。如果插入操作导致关键字数量超过 `capacity` ，则应该 逐出 最久未使用的关键字。

函数 `get` 和 `put` 必须以 `O(1)` 的平均时间复杂度运行。

示例：

```
输入
["LRUCache", "put", "put", "get", "put", "get", "put", "get", "get", "get"]
[[2], [1, 1], [2, 2], [1], [3, 3], [2], [4, 4], [1], [3], [4]]
输出
[null, null, null, 1, null, -1, null, -1, 3, 4]

解释
LRUCache lRUCache = new LRUCache(2);
lRUCache.put(1, 1); // 缓存是 &#123;1=1&#125;
lRUCache.put(2, 2); // 缓存是 &#123;1=1, 2=2&#125;
lRUCache.get(1);    // 返回 1
lRUCache.put(3, 3); // 该操作会使得关键字 2 作废，缓存是 &#123;1=1, 3=3&#125;
lRUCache.get(2);    // 返回 -1 (未找到)
lRUCache.put(4, 4); // 该操作会使得关键字 1 作废，缓存是 &#123;4=4, 3=3&#125;
lRUCache.get(1);    // 返回 -1 (未找到)
lRUCache.get(3);    // 返回 3
lRUCache.get(4);    // 返回 4
```

要让 put 和 get 方法的时间复杂度为 O(1)，我们可以总结出 cache 这个数据结构必要的条件：

1、显然 cache 中的元素必须有时序，以区分最近使用的和久未使用的数据，当容量满了之后要删除最久未使用的那个元素腾位置。

2、我们要在 cache 中快速找某个 key 是否已存在并得到对应的 val；

3、每次访问 cache 中的某个 key，需要将这个元素变为最近使用的，也就是说 cache 要支持在任意位置快速插入和删除元素。

那么，什么数据结构同时符合上述条件呢？哈希表查找快，但是数据无固定顺序；链表有顺序之分，插入删除快，但是查找慢。所以结合一下，形成一种新的数据结构：`哈希链表 LinkedHashMap`。

```
class LRUCache &#123;
    int cap;
    LinkedHashMap<Integer,Integer> cache = new LinkedHashMap<>();
    public LRUCache(int capacity) &#123;
        this.cap = capacity;
    &#125;
    
    public int get(int key) &#123;
        if(!cache.containsKey(key))&#123;
            return -1;
        &#125;
        int val = cache.get(key);
        cache.remove(key);
        cache.put(key,val);//再次插入就实现了最新的使用的标记
        return cache.get(key);
    &#125;
    
    public void put(int key, int value) &#123;
        if(cache.containsKey(key))&#123;
            cache.put(key,value);
            int val = cache.get(key);
            cache.remove(key);
            cache.put(key,val);//再次插入
            return;
        &#125;
        if(cache.size()>=cap)&#123;
            //使用iterator()方法获取迭代器，可以遍历集合中的元素。next()方法用于返回下一个元素，即获取最早的键。
            int oldestKey = cache.keySet().iterator().next();
            cache.remove(oldestKey);
        &#125;
        cache.put(key,value);
        return;
    &#125;
&#125;
```

用 LinkedHashMap 可以很容易实现 LRU 缓存，不过面试的时候估计这样不好，还是尽量自己实现数据结构吧🤣

主要想法是使用 JDK 提供的 HashMap，然后自己写一个 Node 节点类，用来保存 value ，并且通过这个 Node 里面的 prev、next 指针将各个值串联起来，这样就维护了顺序。

很重要的一个细节是，Node 里面还要加上 key （尽管 HashMap 本来就存了一份）。

原因是当缓存达到容量上限时，就要先移除尾部的节点，这个时候如果只移除链表的 tail 节点，忽略了 HasmMap 也要 remove ，后面再访问这个被移除的 key 就会造成空指针异常！！！

所以我们在 Node 节点里面加上 key ，删掉 tail 之前先 remove 掉 HasmMap 的这个 key，就很方便了。

不适用LinkedList:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40

... [OUTPUT TRUNCATED - 801698 chars omitted out of 851698 total] ...

span>32. 最长有效括号-栈存储哈希算法290. 单词规律36. 有效的数独1. 两数之和四数之和II哈希法求解不含重复字串249. 移位字符串分组单调栈天气：滑动窗口局部最大值剑指 Offer II 039. 直方图最大矩形面积剑指 Offer II 040. 矩阵中最大的矩形优先级队列PriorityQueue的作用剑指 Offer II 059. 数据流的第 K 大数值剑指 Offer II 060. 出现频率最高的 k 个数字373. 查找和最小的 K 对数字TreeSetTreeSet Vs. HashSet使用剑指 Offer II 057. 值和下标之差都在给定的范围内Methodfirst()和last()方法ceiling()，floor()，higher()和lower()方法图论399. 除法求值207. 课程表210. 课程表 IIPRIM算法-最小生成树多叉树208. 实现 Trie (前缀树)211. 添加与搜索单词 - 数据结构设计212. 单词搜索 II堆&#x2F;优先队列方法摘要215. 数组中的第K个最大元素二、算法模板排序算法排序算法的稳定性选择排序快速排序插入排序堆排序归并排序链表排序LCR 170. 交易逆序对的总数280. 摆动排序二分查找法162. 寻找峰值33. 搜索旋转排序数组二分查找KMP二叉树基础知识129. 求根节点到叶节点数字之和226. 翻转二叉树序列化与反序列化二叉树：非递归的方式 前序中序后序遍历前序遍历中序遍历后序遍历：236. 二叉树的最近公共祖先114. 二叉树展开为链表纠正二叉搜索树深度优先遍历（递归）剑指 Offer II 049. 从根节点到叶节点的路径数字之和广度优先遍历（队列）剑指 Offer II 044. 二叉树每层的最大值-层序遍历剑指 Offer II 045. 二叉树最底层最左边的值-层序遍历剑指 Offer II 046. 二叉树的右侧视图117. 填充每个节点的下一个右侧节点指针 II二叉树深度二叉树节点数量919. 完全二叉树插入器98. 验证二叉搜索树回溯算法40. 组合总和 II90. 子集 II47.全排列 II77. 组合17. 电话号码的字母组合剑指 Offer II 084. 含有重复元素集合的全排列 93. 复原 IP 地址437. 路径总和 III:star2: 112. 路径总和113. 路径总和 II剑指 Offer II 086. 分割回文子字符串79.单词搜索513. 找树左下角的值22. 括号生成79. 单词搜索动态规划64. 最小路径和97. 交错字符串II 019回文字符647. 回文子串5. 最长回文子串剑指 Offer II 093. 最长斐波那契数列剑指 Offer II 092. 翻转字符44. 通配符匹配72. 编辑距离718. 最长重复子数组198.打家劫舍139. 单词拆分300. 最长递增子序列120. 三角形最小路径和221. 最大正方形123. 买卖股票的最佳时机 III188. 买卖股票的最佳时机 IV背包算法:baggage_claim:背包递推公式遍历顺序01背包416. 分割等和子集1049. 最后一块石头的重量 II494. 目标和完全背包剑指 Offer II 104. 排列的数目剑指 Offer II 103. 最少的硬币数目Kadane’s Algorithm &#x2F; Kadene算法并查集滑动窗口双指针680. 验证回文串 II26. 删除有序数组中的重复项统计连续子数组个数offer || 014 是否包含 s1 的某个变位词。滑动窗口统计变位词前缀和算法和大于等于target的连续子数组 和下面一样的题offer010 和为k的子数组个数剑指 Offer II 011. 0 和 1 个数相同的子数组304. 二维区域和检索 - 矩阵不可变-二维前缀和算法算数运算进位与非进位不用除法的除法：倍增求解剑指 Offer II 003. 前 n 个数字二进制中 1 的个数剑指 Offer II 004. 只出现一次的数字 剑指 Offer II 005. 单词长度的最大乘积-位运算剑指 Offer II 070. 排序数组中只出现一次的数字43. 字符串相乘实现两个字符串相加50. Pow(x, n)66.加一求解最大公约数GCD