# leetcode215.-

<img width="1102" height="646" alt="image" src="https://github.com/user-attachments/assets/487f8ba5-b8ef-4fb6-8dcd-8ed2e5d2f8c6" />


### （1）题目白话文解释

题目的核心诉求是：在一个无序的数字列表中，找到按大小倒序排列后，排在第 $k$ 位的那个数字。

**生僻/关键概念解释：**

  * **第 $k$ 个最大元素**：指的是按数值绝对大小排名的第 $k$ 个位置。如果列表中有重复数字，这些重复数字都会占据排名。例如，在 `[5, 5, 3]` 中，第 1 大是 5，第 2 大还是 5，第 3 大才是 3。这与“第 $k$ 个**不同**的元素”有本质区别。
  * **$O(n)$ 时间复杂度约束**：如果直接使用系统自带的排序函数（如 `sort()`），将整个数组排好序再取第 $k$ 个，虽然代码极其简单，但完全排序的时间复杂度是 $O(n \log n)$，不符合题目 $O(n)$ 的严格要求。因此，必须寻找一种“只排序必要部分”的聪明算法。

-----

### （2）代码解题思路来源

这段代码采用的是\*\*快速选择（Quick Select）**算法，它是经典排序算法**快速排序（Quick Sort）\*\*的变体。

解题思路建立在“分治法”和“丢弃无用数据”的基础上：

1.  **随机选取基准（Pivot）**：从数组中随便挑一个数字作为标尺。
2.  **三路划分（Partition）**：把数组里的数字全拿来和标尺比身高。比标尺高的放进 `big` 组，跟标尺一样高的放进 `equal` 组，比标尺矮的放进 `small` 组。
3.  **精准定位（核心剪枝逻辑）**：
      * 如果 `big` 组的人数大于等于 $k$，说明“第 $k$ 大的人”一定藏在 `big` 组里。此时，`equal` 组和 `small` 组的数据就彻底没用了，直接丢弃，仅对 `big` 组继续进行下一轮寻找。
      * 如果 `big` 组和 `equal` 组的人数加起来还不到 $k$ 个，说明标尺定得太高了，前 $k$ 名中有一部分落在了 `small` 组里。此时，丢弃 `big` 和 `equal` 组，去 `small` 组里找。注意，由于前面已经丢掉了一批大个子，在 `small` 组里找的目标名次需要减去那些被丢掉的人数。
      * 如果正好卡在中间，说明标尺本身就是第 $k$ 大的数字，直接返回即可。

这种方法每次平均能丢弃一半的数据，不需要将全部数据彻底排好序，从而将时间复杂度压低到了 $O(n)$。

-----

### （3）带有注释的代码

```python
import random

class Solution:
    def findKthLargest(self, nums: List[int], k: int) -> int:
        # 定义内部递归函数，专门用于在给定数组 nums 中寻找第 k 大的数
        def quick_select(nums, k):
            # 从当前数组中随机抽取一个数字作为基准值标尺
            pivot = random.choice(nums)
            # 准备三个空桶，分别装 大于、等于、小于 基准值的数字
            big, equal, small = [], [], []
            # 遍历当前数组，将所有数字分发到对应的桶里
            for num in nums:
                if num > pivot:
                    big.append(num)
                elif num < pivot:
                    small.append(num)
                else:
                    equal.append(num)   
            # 判定阶段 1：如果大数桶里的元素数量已经足够多（大于等于 k）
            if k <= len(big):
                # 目标绝对在大数桶里，丢弃其他桶，递归向大数桶里继续寻找第 k 大
                return quick_select(big, k)
            # 判定阶段 2：如果总数减去小数桶的数量，都比 k 小
            # 等价于 len(big) + len(equal) < k，说明大数和相等数加起来都不够 k 个
            if len(nums) - len(small) < k:
                # 目标绝对在小数桶里。
                # 此时需要扣除已经被确认比目标大的那些数 (即大数桶和相等数桶的总数)
                return quick_select(small, k - (len(nums) - len(small)))
            # 判定阶段 3：目标既不在大数桶，也不在小数桶
            # 那目标必定落在相等数桶里，直接返回基准值即可
            return pivot
        # 启动第一次快速选择搜索
        return quick_select(nums, k)
```

-----

### （4）每一行代码详细解释

1.  `class Solution:`：定义执行算法的类。
2.  `def findKthLargest(self, nums, k):`：定义核心公开方法，接收目标数组 `nums` 和名次 `k`。
3.  `def quick_select(nums, k):`：在内部定义一个辅助的递归函数，这种写法方便直接利用外部方法的上下文，同时避免将 `import random` 等操作暴露在外。
4.  `pivot = random.choice(nums)`：利用 Python 的随机模块，从当前数组中随机挑选一个元素。**随机化是避免算法在极端情况（如数组已经有序）下退化为 $O(n^2)$ 时间复杂度的关键保障**。
5.  `big, equal, small = [], [], []`：初始化三个列表（桶），这被称为“三路划分”。
6.  `for num in nums:`：启动循环，逐一考察当前数组中的数字。
7.  `if num > pivot:` 到 `else: equal.append(num)`：比较逻辑。将数字按大小关系分别装入三个不同的列表。
8.  `if k <= len(big):`：核心分支一。如果 `big` 列表的长度大于等于 $k$，意味着全数组排名前 $k$ 大的数字全都在 `big` 里。
9.  `return quick_select(big, k)`：直接将 `big` 列表作为新的搜索空间进行递归，目标名次依然是 $k$。
10. `if len(nums) - len(small) < k:`：核心分支二。`len(nums) - len(small)` 在数学上刚好等于 `len(big) + len(equal)`。如果这个数量小于 $k$，说明哪怕把大于和等于基准值的数字全算上，名额也没凑满 $k$ 个。这证明第 $k$ 大的元素必定沦落到了 `small` 列表中。
11. `return quick_select(small, k - len(nums) + len(small))`：将 `small` 列表作为新搜索空间。重点在于更新名次要求：因为前面已经有 `len(nums) - len(small)` 个数字排在前面了，所以在剩余的 `small` 列表中，只需要寻找排在第 `k - 之前的数字个数` 位的元素。
12. `return pivot`：如果上述两个 `if` 都没有命中，说明第 $k$ 个元素正好落在了 `equal` 列表所在的区间。因为 `equal` 列表里所有的数字都和 `pivot` 相等，所以无论具体是哪一个，返回 `pivot` 就是最终的正确答案。
13. `return quick_select(nums, k)`：在主方法最后，调用并返回内部递归函数的最初执行结果。

-----

### （5）具体数值算例及变量变化表格

设定输入数组 `nums = [3, 2, 1, 5, 6, 4]`，寻找第 `k = 2` 大的元素。
假设由于是随机选取，基准值 `pivot` 在每一轮的取值如下方表格所示。

| 递归层级 | 当前搜索数组 `nums` | 当前目标名次 `k` | 随机基准 `pivot` | 划分结果 (big / equal / small) | 判定条件走向 | 对应执行代码行 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **第一层** | `[3, 2, 1, 5, 6, 4]`| `2` | 假设取 `4` | `big = [5, 6]` <br> `equal = [4]` <br> `small = [3, 2, 1]` | `k(2) <= len(big)(2)` 成立。目标在 big 中。 | `if k <= len(big):`<br> `return quick_select(big, k)` |
| **第二层** | `[5, 6]` | `2` | 假设取 `6` | `big = []` <br> `equal = [6]` <br> `small = [5]` | `k(2) <= 0` 假。<br> `2 - 1 < 2` 成立。目标在 small 中。 | `if len(nums) - len(small) < k:`<br>`return quick_select(...)` |
| **计算新 k**| - | - | - | 扣除已经被排在前面的数量 (`len(big)+len(equal)` = 1) | 新 `k` = `2 - 1 = 1` | `k - len(...) + len(...)` |
| **第三层** | `[5]` | `1` | 只能取 `5` | `big = []` <br> `equal = [5]` <br> `small = []` | `k(1) <= 0` 假。<br> `1 - 0 < 1` 假。<br> 命中 equal 区间。 | `return pivot` |
| **最终返回**| - | - | - | 逐层向上返回，最终结果为 `5` | 无 | `return pivot` |

*(推演结论：数组排序后为 `[6, 5, 4, 3, 2, 1]`，第 2 大的元素确实是 5，算法逻辑严密)*
