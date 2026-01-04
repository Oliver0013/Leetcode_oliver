# 对撞双指针 vs 滑动窗口

## 核心区别一览

| **特性**     | **对撞双指针 (Collision Pointers)**                        | **滑动窗口 (Sliding Window)**                                |
| ------------ | ---------------------------------------------------------- | ------------------------------------------------------------ |
| **物理形态** | **相向而行** ($\rightarrow \dots \leftarrow$)              | **同向而行** ($\rightarrow \dots \rightarrow$)               |
| **初始位置** | `left = 0`, `right = n-1`                                  | `left = 0`, `right = 0`                                      |
| **决策逻辑** | **博弈/谈判 (Either-Or)** 根据现状，决定动左边还是动右边。 | **主从/追逐 (Action-Reaction)** 右边无脑冲，左边视情况被动收缩。 |
| **核心用途** | 找两个点 (Pair)，通常利用有序性。                          | 找一个范围 (Range/Subarray)。                                |
| **典型例题** | 两数/三数之和、接雨水、盛水容器。                          | 无重复字符子串、长度最小子数组。                             |

------

## 一、对撞双指针 (Collision Pointers)

核心心法：“找平衡”。

通常用于有序数组或两端向中间逼近的问题。每一步通过比较当前状态与目标状态，利用单调性排除掉一半的搜索空间。

### 适用场景

1. **有序数组求和**：如 Two Sum II, 3Sum。
2. **两端限制**：如盛最多水的容器（木桶效应）。
3. **字符串操作**：如反转字符串、回文判断。

### 🚀 C++ 标准模板

```c++
void collisionTwoPointers(vector<int>& nums) {
    // 1. 定义左右指针
    int left = 0;
    int right = nums.size() - 1;

    // 2. 主循环：只要没相遇就继续
    while (left < right) {
        // 计算当前状态（如数值之和、面积等）
        int val = calculate(nums[left], nums[right]);

        if (val == target) {
            // 找到目标，处理结果
            return; 
        } 
        else if (val < target) {
            // 值太小了？需要变大 -> 左指针右移 (依赖数组有序)
            left++;
        } 
        else { // val > target
            // 值太大了？需要变小 -> 右指针左移
            right--;
        }
    }
}
```

------

## 二、滑动窗口 (Sliding Window)

核心心法：“先吃再吐” (先扩张，后收缩)。

用于处理连续子数组或子串问题。右指针负责探索未知区域，左指针负责维护窗口的合法性。

### 适用场景

1. **定长窗口**：求长度为 K 的子数组的平均值/最大值。
2. **不定长窗口（求最长）**：无重复字符的最长子串、最大连续1的个数。
3. **不定长窗口（求最短）**：长度最小的子数组、最小覆盖子串。

### 🚀 C++ 标准模板 (万能版)

不要去预判 `right+1`，永远处理当前的 `right`。

```c++
int slidingWindow(string s) {
    // 1. 定义窗口数据结构 (哈希表/数组/计数器)
    // vector<int> window(128, 0); 或 unordered_map<char, int> map;
    
    int left = 0, right = 0;
    int ans = 0; // 记录结果 (最大或最小)

    // 2. 主循环：右指针主动扩张 (老大带着跑)
    for (right = 0; right < s.size(); right++) {
        // 【A. 进】先把右边的元素加入窗口
        char c = s[right];
        window.add(c);

        // 【B. 吐】只要窗口"坏了" (不满足条件)，左指针就被动收缩
        // 例如：有重复字符、和超过target、窗口长度超限等
        while (/* window needs shrink */) {
            char d = s[left];
            window.remove(d); // 移除左边元素
            left++;           // 左边界右移
        }

        // 【C. 算】此时窗口一定是合法的，更新结果
        // 求最长：在 while 出来后更新 (因为此时是合法的最大状态)
        // ans = max(ans, right - left + 1);
        
        // *求最短：通常在 while 循环内部，remove 之前更新 (一旦满足条件就尝试更短)
    }
    return ans;
}
```

### 💡 避坑指南 (Debug Checklist)

1. **不要预判**：千万别写 `if (right + 1 < n)`，这会让逻辑变得极其复杂。让 `for` 循环自然推进 `right`。
2. **左指针防越界**：在 `while` 收缩时，通常隐含了 `left <= right` 的条件（因为如果窗口空了，约束条件自然消失，`while` 会停止），所以大多数情况不需要显式判断 `left < right`。
3. **结果更新位置**：
   - 找**最长**：在 `while` 收缩**之后**更新（保证窗口合法且尽可能长）。
   - 找**最短**：在 `while` 循环**内部**更新（只要满足条件，就记录一次，然后缩一下试试）。

