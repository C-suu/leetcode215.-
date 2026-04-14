# leetcode215.-

<img width="1102" height="646" alt="image" src="https://github.com/user-attachments/assets/487f8ba5-b8ef-4fb6-8dcd-8ed2e5d2f8c6" />

### （1）题目意思及生僻概念解释

**题目大意：**
给定一堆毫无顺序的整数，要求找出这堆数字中排名第 `k` 的那个最大的数字。

**生僻不易理解的概念解释：**

  * **“排序后的第 k 个最大的元素，而不是第 k 个不同的元素”：** 这句话的意思是，如果数组中有重复的数字，名次会并列计算，但坑位各自独立。例如数组 `[5, 5, 4]`，排在第一大的是 `5`，排在第二大的还是 `5`，排在第三大的才是 `4`。找“第2大”的元素就是 `5`。
  * **“时间复杂度 O(n)”：** 这是一个衡量算法运行速度的概念。`O(n)` 意味着算法执行所需的时间，与给定的数字总个数（即 `n`）成正比。如果使用普通的完全排序方法（如Python自带的排序），其时间复杂度通常是 `O(n log n)`，耗时比题目要求的要长。因此，必须采用一种不需要对所有数字进行彻底排序就能找到答案的特殊方法（如快速选择算法）。

-----

### （2）代码解题思路解析

从这段代码的角度来看，采用的是\*\*“快速选择算法（Quick Select）”\*\*的思路，结合了“三路划分”的技巧。其核心思想是“不断缩小搜索范围”，具体思路如下：

1.  **随机设立标尺：** 在当前数组中随机抽取一个数字作为“标尺”（基准值 `pivot`）。
2.  **分装三个桶：** 将数组里的所有数字与这个标尺进行比较，按大小分配到三个不同的桶中：
      * `big` 桶：装所有比标尺**大**的数字。
      * `equal` 桶：装所有与标尺**相等**的数字。
      * `small` 桶：装所有比标尺**小**的数字。
3.  **按数量判定去向（核心逻辑）：**
      * 要想找第 `k` 大的数，先看 `big` 桶。如果 `big` 桶里的数字个数大于或等于 `k`，说明第 `k` 大的数字绝对藏在 `big` 桶里。此时可以直接丢弃另外两个桶，只盯着 `big` 桶继续寻找第 `k` 大即可。
      * 如果 `big` 桶和 `equal` 桶里的数字总数加起来都不到 `k` 个，说明前排的名次不够，第 `k` 大的数字必然落在了最后面的 `small` 桶里。此时进入 `small` 桶寻找，但需要更新名次：要找的目标在 `small` 桶里排名第 `k - (大桶人数 + 等于桶人数)`。
      * 如果既不在大桶，也不在小桶，那必定正好落在 `equal` 桶中。因为 `equal` 桶里的数字全都是一样的（等于基准值），此时直接返回基准值即为答案。
4.  **递归循环：** 重复上述过程，每次都会抛弃掉一部分不可能的数字，直到最终锁定目标。

-----

### （3）附带注释的代码

```python
import random
from typing import List

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
            if len(big) + len(equal) < k:
                # 目标绝对在小数桶里。
                # 此时需要扣除已经被确认比目标大的那些数 (即大数桶和相等数桶的总数)
                return quick_select(small, k - (len(big) + len(equal)))
            # 判定阶段 3：目标既不在大数桶，也不在小数桶
            # 那目标必定落在相等数桶里，直接返回基准值即可
            return pivot
        # 启动第一次快速选择搜索
        return quick_select(nums, k)
```

-----

### （4）每一行代码的详细解释

  * `class Solution:`
      * 定义一个名为 `Solution` 的类，符合 LeetCode 等刷题平台的标准格式，用于封装解题方法。
  * `def findKthLargest(self, nums: List[int], k: int) -> int:`
      * 定义主函数，接收一个整数列表 `nums` 和一个代表目标名次的整数 `k`，最终返回一个整数作为答案。
  * `def quick_select(nums, k):`
      * 在主函数内部定义一个辅助函数（闭包函数），实现具体的递归查找逻辑，接收当前被划分的数组和当前需要寻找的名次。
  * `pivot = random.choice(nums)`
      * 利用 `random` 模块提供的 `choice` 方法，从当前的 `nums` 数组中随机挑出一个元素作为比较的基准值。随机化是为了防止在面对极端数据（如已有序的数组）时算法效率退化。
  * `big, equal, small = [], [], []`
      * 初始化三个空的列表，作为承载划分结果的“桶”。
  * `for num in nums:`
      * 开启一个循环，依次取出当前数组 `nums` 中的每一个数字进行检查。
  * `if num > pivot:`
      * 判断当前取出的数字是否严格大于基准值。
  * `big.append(num)`
      * 如果大于基准值，则将该数字装入 `big`（大数）桶中。
  * `elif num < pivot:`
      * 如果前面的条件不满足，继续判断该数字是否严格小于基准值。
  * `small.append(num)`
      * 如果小于基准值，则将该数字装入 `small`（小数）桶中。
  * `else:`
      * 如果既不大于也不小于基准值，说明等于基准值。
  * `equal.append(num)`
      * 将与基准值相等的数字装入 `equal` 桶中。
  * `if k <= len(big):`
      * 分桶完毕后，检查 `big` 桶中的数字总个数是否大于或等于需要寻找的名次 `k`。
  * `return quick_select(big, k)`
      * 如果上一个条件成立，说明第 `k` 大的数一定在 `big` 桶中。因此再次调用 `quick_select` 函数，把 `big` 桶作为新的数组传进去，继续找第 `k` 大。
  * `if len(big) + len(equal) < k:`
      * 如果目标不在 `big` 桶中，继续检查 `big` 桶和 `equal` 桶的元素总和是否小于 `k`。如果小于，说明比目标大或者与目标相等的数字加起来都不够占满前 `k` 个名额。
  * `return quick_select(small, k - (len(big) + len(equal)))`
      * 如果上一个条件成立，说明目标在 `small` 桶中。此时去 `small` 桶里寻找，但必须将名次 `k` 减掉前面排在前面的所有元素数量（即 `big` 和 `equal` 桶的总长度）。
  * `return pivot`
      * 如果程序能执行到这一行，说明前两个 `if` 条件都不满足，即目标既不在 `big` 桶也不在 `small` 桶。那么目标必然落在 `equal` 桶内。因为 `equal` 桶里的数字全部等于 `pivot`，直接返回 `pivot` 即可得到正确答案。
  * `return quick_select(nums, k)`
      * 这是 `findKthLargest` 主函数的最后一步，将最初始的原始数组和原始的 `k` 值传入刚才定义好的内部函数，启动整个查找流程并返回最终结果。

-----

### （5）具体数值算例表格

以题目中的示例 `nums = [3, 2, 1, 5, 6, 4]`，`k = 2` 为例。假设随机选择的基准值情况如下表所示：

| 执行阶段 | 变量 / 状态变化 | 对应代码行 |
| :--- | :--- | :--- |
| **启动调用** | 初始输入：`nums = [3, 2, 1, 5, 6, 4]`, `k = 2` | `return quick_select(nums, k)` |
| **第1轮：选基准** | 随机选取基准值 `pivot = 4` （假设随机抽到了4） | `pivot = random.choice(nums)` |
| **第1轮：分桶** | 将原数组拆分进入三个桶：<br>➔ `big = [5, 6]`<br>➔ `equal = [4]`<br>➔ `small = [3, 2, 1]` | `for num in nums:` 以及内部判断等几行代码 |
| **第1轮：判定 1** | 判断 `k <= len(big)`<br>➔ 此时 `2 <= 2`，条件成立 | `if k <= len(big):` |
| **第1轮：行动** | 确定目标在 `big` 桶内，丢弃其他部分。准备进入下一轮。 | `return quick_select(big, k)` |
| **第2轮：选基准** | 此时输入：`nums = [5, 6]`, `k = 2`<br>随机选取基准值 `pivot = 6` （假设随机抽到了6） | `pivot = random.choice(nums)` |
| **第2轮：分桶** | 将当前数组拆分进入三个桶：<br>➔ `big = []` (5和6没有比6大的)<br>➔ `equal = [6]`<br>➔ `small = [5]` | `for num in nums:` 以及内部判断等几行代码 |
| **第2轮：判定 1** | 判断 `k <= len(big)`<br>➔ 此时 `2 <= 0`，条件不成立 | `if k <= len(big):` |
| **第2轮：判定 2** | 判断 `len(big) + len(equal) < k`<br>➔ 此时 `0 + 1 < 2` (即 `1 < 2`)，条件成立 | `if len(big) + len(equal) < k:` |
| **第2轮：行动** | 确定目标在 `small` 桶内。<br>计算新名次：`k - (len(big) + len(equal)) = 2 - 1 = 1`<br>准备进入下一轮。 | `return quick_select(small, k - (len(big) + len(equal)))` |
| **第3轮：选基准** | 此时输入：`nums = [5]`, `k = 1`<br>只有一个元素，基准值必然是 `pivot = 5` | `pivot = random.choice(nums)` |
| **第3轮：分桶** | 将当前数组拆分进入三个桶：<br>➔ `big = []`<br>➔ `equal = [5]`<br>➔ `small = []` | `for num in nums:` 以及内部判断等几行代码 |
| **第3轮：判定 1** | 判断 `k <= len(big)`<br>➔ 此时 `1 <= 0`，条件不成立 | `if k <= len(big):` |
| **第3轮：判定 2** | 判断 `len(big) + len(equal) < k`<br>➔ 此时 `0 + 1 < 1` (即 `1 < 1`)，条件不成立 | `if len(big) + len(equal) < k:` |
| **第3轮：行动** | 既然既不在 `big` 也不在 `small`，必定在 `equal` 中。<br>直接输出答案 `5`。 | `return pivot` |
