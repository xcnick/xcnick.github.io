title: LeetCode数组总结
date: 2018-12-27 15:40:43
tags: LeetCode
---

> 数组题目总结
<!-- more -->

# 从排序数组中删除重复项

> 给定一个排序数组，你需要在原地删除重复出现的元素，使得每个元素只出现一次，返回移除后数组的新长度。不要使用额外的数组空间，你必须在原地修改输入数组并在使用 O(1) 额外空间的条件下完成。

使用快慢指针遍历数组
* 如果两个指针指向的数字相同，则快指针向前走一步；
* 如果两个指针指向的数字不同，则两个指针都往前走一步，并且将快指针的值赋给慢指针的下一个。

```Cpp
int removeDuplicates(vector<int> &nums) {
  if (nums.empty()) {
    return 0;
  }
  int j = 0, n = nums.size();
  for (int i = 1; i < n; ++i) {
    if (nums[i] != nums[j]) {
      nums[++j] = nums[i];
    }
  }
  return j + 1;
}
```

```Python
def removeDuplicates(self, nums):
    """
    :type nums: List[int]
    :rtype: int
    """
    if not nums:
        return 0
    length = 0
    for i in range(1, len(nums)):
        if nums[i] != nums[length]:
            length += 1
            nums[length] = nums[i]
    return length + 1
```

# 买卖股票的最佳时机 II

> 给定一个数组，它的第 i 个元素是一支给定股票第 i 天的价格。设计一个算法来计算你所能获取的最大利润。你可以尽可能地完成更多的交易（多次买卖一支股票）。
注意：你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。
> 示例 1: 输入: [7,1,5,3,6,4] 输出: 7
> 示例 2: 输入: [1,2,3,4,5] 输出: 4
> 示例 3: 输入: [7,6,4,3,1] 输出: 0

由于可以无限次买入和卖出。从第二天开始，如果当前价格比之前价格高，则把差值加入利润中。

```Cpp
int maxProfit(vector<int> &prices) {
  int res = 0;
  for (int i = 0; i < prices.size() - 1; ++i) {
    if (prices[i] < prices[i + 1]) {
      res += prices[i + 1] - prices[i];
    }
  }
  return res;
}
```

```Python
def maxProfit(self, prices):
    """
    :type prices: List[int]
    :rtype: int
    """
    res = 0
    for i in range(len(prices) - 1):
        if prices[i] < prices[i + 1]:
            res += prices[i + 1] - prices[i]
    return res
```

# 旋转数组

> 给定一个数组，将数组中的元素向右移动 k 个位置，其中 k 是非负数。
> 示例 1：输入: [1,2,3,4,5,6,7] 和 k = 3 输出: [5,6,7,1,2,3,4]
> 示例 2: 输入: [-1,-100,3,99] 和 k = 2 输出: [3,99,-1,-100]

最直观的方法是使用一种O(n)的空间复杂度方法，复制一个和nums一样的数组，然后通过映射关系i->(i+k)%n来交换数字。

```Cpp
void rotate(vector<int> &nums, int k) {
  vector<int> t = nums;
  for (int i = 0; i < nums.size(); ++i) {
    nums[(i + k) % nums.size()] = t[i];
  }
}
```

```Python
def rotate(self, nums, k):
    """
    :type nums: List[int]
    :type k: int
    :rtype: void Do not return anything, modify nums in-place instead.
    """
    t = nums[:]
    for i in range(len(nums)):
        nums[(i + k) % len(nums)] = t[i]
```

Python的数组切片操作

```Python
def rotate(self, nums, k):
    """
    :type nums: List[int]
    :type k: int
    :rtype: void Do not return anything, modify nums in-place instead.
    """
    i = k % len(nums)
    nums[:] = nums[-i:] + nums[:-i]
```

旋转数组的操作可以看成从数组的末尾取k个数放入数组的开头，可以用stl中的push_back和erase来完成

```Cpp
void rotate(vector<int> &nums, int k) {
  if (nums.empty() || (k %= nums.size()) == 0) {
    return;
  }
  int n = nums.size();
  for (int i = 0; i < n - k; ++i) {
    nums.push_back(nums[0]);
    nums.erase(nums.begin());
  }
}
```

类似反转字符的方法，先把前n-k个数字翻转，再把后k个数字翻转，最后把整个数组翻转

```Cpp
void rotate(vector<int> &nums, int k) {
  if (nums.empty() || (k %= nums.size()) == 0) {
    return;
  }
  int n = nums.size();
  reverse(nums.begin(), nums.begin() + n - k);
  reverse(nums.begin() + n - k, nums.end());
  reverse(nums.begin(), nums.end());
}
```

# 存在重复

> 给定一个整数数组，判断是否存在重复元素。如果任何值在数组中出现至少两次，函数返回 true。如果数组中每个元素都不相同，则返回 false。
> 示例 1: 输入: [1,2,3,1] 输出: true
> 示例 2: 输入: [1,2,3,4] 输出: false

使用哈希表，遍历整个数组，如果表里存在，则返回false，如果不存在，则放入表中。

```Cpp
bool containsDuplicate(vector<int>& nums) {
  unordered_map<int, int> m;
  for (int i = 0; i < nums.size(); ++i) {
    if (m.find(nums[i]) != nums.end()) {
      return false;
    }
    ++m[nums[i]];
  }
}
```

```Python
def containsDuplicate(self, nums):
    """
    :type nums: List[int]
    :rtype: bool
    """
    return len(nums) != len(set(nums))
```

# 只出现一次的数字

> 给定一个非空整数数组，除了某个元素只出现一次以外，其余每个元素均出现两次。找出那个只出现了一次的元素。
> 示例 1: 输入: [2,2,1] 输出: 1
> 示例 2: 输入: [4,1,2,1,2] 输出: 4

算法时间复杂度O(n)，空间复杂度O(1)，不能使用排序或者map。使用异或的方法，相同的数字异或后得到0，将所有的数字异或并加起来，得到的数字就是只出现一次的数字。

```Cpp
int singleNumber(vector<int>& nums) {
  int result = 0;
  for (int i = 0; i < nums.size(); ++i) {
    result ^= nums[i];
  }
  return result;
}
```

```Python
def singleNumber(self, nums):
    """
    :type nums: List[int]
    :rtype: int
    """
    res = 0
    for n in nums:
        res ^= n
    return res
```

# 两个数组的交集

> 给定两个数组，编写一个函数来计算它们的交集。
> 示例 1: 输入: nums1 = [1,2,2,1], nums2 = [2,2] 输出: [2]
> 示例 2: 输入: nums1 = [4,9,5], nums2 = [9,4,9,8,4] 输出: [9,4]
> 说明: 输出结果中的每个元素一定是唯一的。我们可以不考虑输出结果的顺序。

将nums1放入一个set, 然后遍历nums2，如果set中存在，则放入结果set中，最后转换为vector

```Cpp
vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
  unordered_set<int> s(nums1.begin(), nums1.end()), res;
  for (auto n : nums2) {
    if (s.count(n)) {
      res.insert(n);
    }
  }
  return vector<int>(res.begin(), res.end());
}
```

```Python
def intersection(self, nums1, nums2):
    """
    :type nums1: List[int]
    :type nums2: List[int]
    :rtype: List[int]
    """
    s1 = set(nums1)
    res = set()
    for n in nums2:
        if n in s1:
            res.add(n)
    return list(res)
```

# 加一

> 给定一个由整数组成的非空数组所表示的非负整数，在该数的基础上加一。最高位数字存放在数组的首位， 数组中每个元素只存储一个数字。你可以假设除了整数 0 之外，这个整数不会以零开头。
> 示例 1: 输入: [1,2,3] 输出: [1,2,4] 解释: 输入数组表示数字 123。

如果末尾数字是9，则有进位问题，如果前面的数字仍为9，则需要继续进位。依次类推后，如果第一位是9，则需要增加一个1。

```Cpp
vector<int> plusOne(vector<int> &digits) {
  int n = digits.size();
  for (int i = n - 1; i >= 0; --i) {
    if (digits[i] == 9) {
      digits[i] = 0;
    } else {
      digits[i] += 1;
      return digits;
    }
  }
  if (digits.front() == 0) {
    digits.insert(digits.begin(), 1);
  }
  return digits;
}
```

```Python
def plusOne(self, digits):
    """
    :type digits: List[int]
    :rtype: List[int]
    """
    return [int(i) for i in str(int("".join([str(i) for i in digits])) + 1)]
```

# 移动零

> 给定一个数组 nums，编写一个函数将所有 0 移动到数组的末尾，同时保持非零元素的相对顺序。
> 示例: 输入: [0,1,0,3,12] 输出: [1,3,12,0,0]

使用in-place替换来做，需要两个指针，一个向后扫，找到非零位置，然后与前面的指针交换位置。

```Cpp
void moveZeroes(vector<int>& nums) {
  for (int i = 0, j = 0; i < nums.size(); ++i) {
    if (nums[i]) {
      swap(nums[i], nums[j++]);
    }
  }
}
```

```Python
def moveZeroes(self, nums):
    """
    :type nums: List[int]
    :rtype: void Do not return anything, modify nums in-place instead.
    """
    j = 0
    for i in range(len(nums)):
        if nums[i] != 0:
            nums[j], nums[i] = nums[i], nums[j]
            j += 1
```