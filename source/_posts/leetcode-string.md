title: LeetCode字符串总结
date: 2018-12-21 18:24:12
tags: LeetCode
---

> 字符串题目总结
<!-- more -->

# 反转字符串

> 编写一个函数，其作用是将输入的字符串反转过来。

直接从两头往中间走，同时交换两边的字符即可。

```Cpp
string reverseString(string s) {
  int left = 0, right = s.size() - 1;
  while (left < right) {
    char t = s[left];
    s[left] = s[right];
    s[right] = t;
    left++;
    right--;
  }
  return s;
}
```

另外，可使用swap函数帮助翻转

```Cpp
string reverseString(string s) {
  int left = 0, right = s.size() - 1;
  while (left < right) {
    swap(s[left++], s[right--]);
  }
  return s;
}
```

```Python
def reverseString(self, s):
    return s[::-1]
```

```Python
def reverseString(self, s):
    """
    :type s: str
    :rtype: str
    """
    result = ""
    for i, _ in enumerate(s):
        result += s[len(s) - 1 - i]
    return result
```

# 反转字符串II

> 给定一个字符串和一个整数 k，你需要对从字符串开头算起的每个 2k 个字符的前k个字符进行反转。如果剩余少于 k 个字符，则将剩余的所有全部反转。如果有小于 2k 但大于或等于 k 个字符，则反转前 k 个字符，并将剩余的字符保持原样。
> 示例: 输入: s = "abcdefg", k = 2 输出: "bacdfeg"

```Cpp
string reverseStr(string s, int k) {
  int size = s.size();
  int pos = 0;
  while (pos < size) {
    if (pos + k - 1 < size) {
      reverse(s.begin() + pos, s.begin() + pos + k);
    } else {
      reverse(s.begin() + pos, s.end());
    }
    pos += 2 * k;
  }
  return s;
}
```

# 反转整数

> 给出一个 32 位的有符号整数，你需要将这个整数中每位上的数字进行反转。
> 示例 1: 输入: 123 输出: 321
> 示例 2: 输入: -123 输出: -321
> 示例 3: 输入: 120 输出: 21
> 仅能存储32位有符号整数，若反转后溢出则返回0.

翻转数字问题需要注意的就是溢出问题。

```Cpp
int reverse(int x) {
  int res = 0;
  while (x != 0) {
    if (abs(res) > INT_MAX / 10) {
      return 0;
    }
    res = res * 10 + x % 10;
    x /= 10;
  }
  return res;
}
```

```Python
def reverse(self, x):
    """
    :type x: int
    :rtype: int
    """
    flag = (x >= 0)
    x = (x if flag else -x)
    res = 0
    while x != 0:
        if res > ((pow(2,31) - 1) / 10):
            return 0
        res = res * 10 + x % 10
        x = x // 10
    return res if flag else -res
```

# 字符串中的第一个唯一字符

> 给定一个字符串，找到它的第一个不重复的字符，并返回它的索引。如果不存在，则返回 -1。
> 示例：s = "leetcode", 返回0。s = "loveleetcode", 返回2。

使用哈希表建立每个字符与其出现次数的映射，然后按顺序遍历字符串，找到第一个出现次数为1的字符，返回其位置即可。

```Cpp
int firstUniqChar(string s) {
  unordered_map<char, int> m;
  for (char c : s) {
    m[c]++;
  }
  for (int i = 0; i < s.size(); ++i) {
    if (m[s[i]] == 1) {
      return i;
    }
  }
  return -1;
}
```

```Python
def firstUniqChar(self, s):
    """
    :type s: str
    :rtype: int
    """
    dict1 = {}
    for c in s:
        dict1[c] = dict1[c] + 1 if c in dict1 else 1
    for i in range(len(s)):
        if dict1[s[i]] == 1:
            return i
    return -1
```

# 有效的字母异位词

> 给定两个字符串 s 和 t ，编写一个函数来判断 t 是否是 s 的一个字母异位词。
> 示例 1：输入: s = "anagram", t = "nagaram" 输出: true
> 示例 2: 输入: s = "rat", t = "car" 输出: false

使用哈希表映射，用一个数组代替哈希表。

```Cpp
bool isAnagram(string s, string t) {
  if (s.size() != t.size()) {
    return false;
  }
  int m[26] = { 0 };
  for (int i = 0; i < s.size(); ++i) {
    ++m[s[i] - 'a'];
  }
  for (int i = 0; i < t.size(); ++i) {
    if (--m[t[i] - 'a'] < 0) {
      return false;
    }
  }
  return true;
}
```

```Python
def isAnagram(self, s, t):
    """
    :type s: str
    :type t: str
    :rtype: bool
    """
    if len(s) != len(t):
        return False
    m = [0] * 26
    for c in s:
        m[ord(c) - ord('a')] += 1
    for c in t:
        m[ord(c) - ord('a')] -= 1
        if m[ord(c) - ord('a')] < 0:
            return False
    return True
```

# 验证回文字符串

> 给定一个字符串，验证它是否是回文串，只考虑字母和数字字符，可以忽略字母的大小写。
> 示例 1: 输入: "A man, a plan, a canal: Panama" 输出: true
> 示例 2: 输入: "race a car" 输出: false

需要建立两个指针，left和right, 分别从字符的开头和结尾处开始遍历整个字符串，如果遇到非字母数字的字符就跳过，继续往下找，直到找到下一个字母数字或者结束遍历，如果遇到大写字母，就将其转为小写。等左右指针都找到字母数字时，比较这两个字符，若相等，则继续比较下面两个分别找到的字母数字，若不相等，直接返回false。

```Cpp
bool isPalindrome(string s) {
  int left = 0, right = s.size() - 1;
  while (left < right) {
    if (!isAlphaNum(s[left])) {
      ++left;
    } else if (!isAlphaNum(s[right])) {
      --right;
    // 'a' == 97, 'A' = 65
    } else if ((s[left] + 32 - 'a') % 32 != (s[right] + 32 - 'a') % 32) {
      return false;
    } else {
      ++left;
      --right;
    }
  }
  return true;
}

bool isAlphaNum(char &ch) {
  if (ch >= 'a' && ch <= 'z') {
    return true;
  }
  if (ch >= 'A' && ch <= 'Z') {
    return true;
  }
  if (ch >= '0' && ch <= '9') {
    return true;
  }
  return false;
}
```

```Python
def isPalindrome(self, s):
    """
    :type s: str
    :rtype: bool
    """
    left = 0
    right = len(s) - 1
    while left < right:
        if not s[left].isalnum():
            left += 1
        elif not s[right].isalnum():
            right -= 1
        elif s[left].lower() != s[right].lower():
            return False
        else:
            left += 1
            right -= 1
    return True
```

# 字符串转换整数

> 首先，该函数会根据需要丢弃无用的开头空格字符，直到寻找到第一个非空格的字符为止。
当我们寻找到的第一个非空字符为正或者负号时，则将该符号与之后面尽可能多的连续数字组合起来，作为该整数的正负号；假如第一个非空字符是数字，则直接将其与之后连续的数字字符组合起来，形成整数。
该字符串除了有效的整数部分之后也可能会存在多余的字符，这些字符可以被忽略，它们对于函数不应该造成影响。
注意：假如该字符串中的第一个非空格字符不是一个有效整数字符、字符串为空或字符串仅包含空白字符时，则你的函数不需要进行转换。
在任何情况下，若函数不能进行有效的转换时，请返回 0。
仅处理32位大小有符号整数。

1. 若字符串开头是空格，则跳过所有空格，到第一个非空格字符，如果没有，则返回0.
2. 若第一个非空格字符是符号+/-，则标记sign的真假，这道题还有个局限性，那就是在c++里面，+-1和-+1都是认可的，都是-1，而在此题里，则会返回0.
3. 若下一个字符不是数字，则返回0. 完全不考虑小数点和自然数的情况。
4. 如果下一个字符是数字，则转为整形存下来，若接下来再有非数字出现，则返回目前的结果。
5. 还需要考虑边界问题，如果超过了整形数的范围，则用边界值替代当前值。

```Cpp
int myAtoi(string str) {
  if (str.empty()) {
    return 0;
  }
  int sign = 1, base = 0, i = 0, n = str.size();
  while (i < n && str[i] == ' ') {
    ++i;
  }
  if (str[i] == '+' || str[i] == '-') {
    sign = (str[i++] == '+') ? 1 : -1;
  }
  while (i < n && str[i] >= '0' && str[i] <= '9') {
    if (base > INT_MAX / 10 || (base == INT_MAX / 10 && str[i] - '0' > 7)) {
      return (sign == 1) ? INT_MAX : INT_MIN;
    }
    base = 10 * base + (str[i++] - '0');
  }
  return base * sign;
}
```

```Python
def myAtoi(self, str):
    """
    :type str: str
    :rtype: int
    """
    str = str.strip()
    if not str:
      return 0
    sign = 1
    if str[0] == "-":
      sign = -1
    if str[0] in ("+", "-"):
      str = str[1:]

    num = 0
    for c in str:
      if c >= "0" and c <= "9":
          num = num * 10 + ord(c) - ord("0")
      else:
          break
    num = num * sign

    if num > (pow(2,31) - 1):
      num = (pow(2,31) - 1)
    elif num < -pow(2, 31):
      num = -pow(2, 31)
    return num
```

# 实现strStr()

> 给定一个 haystack 字符串和一个 needle 字符串，在 haystack 字符串中找出 needle 字符串出现的第一个位置 (从0开始)。如果不存在，则返回 -1。
> 示例 1: 输入: haystack = "hello", needle = "ll" 输出: 2
> 示例 2: 输入: haystack = "aaaaa", needle = "bba" 输出: -1

1. 如果子字符串为空，则返回0，如果子字符串长度大于母字符串长度，则返回-1。
2. 并不需要遍历整个母字符串，而是遍历到剩下的长度和子字符串相等的位置即可。
3. 对于每一个字符，我们都遍历一遍子字符串，一个一个字符的对应比较，如果对应位置有不等的，则跳出循环，如果一直都没有跳出循环，则说明子字符串出现了，则返回起始位置即可。

```Cpp
int strStr(string haystack, string needle) {
  if (needle.empty()) {
    return 0;
  }
  int m = haystack.size(), n = needle.size();
  if (m < n) {
    return -1;
  }
  for (int i = 0; i <= m - n; ++i) {
    int j = 0;
    for (j = 0; j < n; ++j) {
      if (haystack[i + j] != needle[j]) {
        break;
      }
      if (j == n) {
        return i;
      }
    }
  }
  return -1;
}
```

```Python
def strStr(self, haystack, needle):
    """
    :type haystack: str
    :type needle: str
    :rtype: int
    """
    l = len(needle)
    for i in range(len(haystack) - l + 1):
        if haystack[i:i + l] == needle:
            return i
    return -1
```

# 最长公共前缀

> 编写一个函数来查找字符串数组中的最长公共前缀。如果不存在公共前缀，返回空字符串 ""。
> 示例 1: 输入: ["flower","flow","flight"] 输出: "fl"
> 示例 2: 输入: ["dog","racecar","car"] 输出: ""

直接按一个字符串为单位查找。

```Cpp
string longestCommonPrefix(vector<string> &strs) {
  if (strs.empty()) {
    return "";
  }
  string res = "";
  for (int j = 0; j < strs[0].size(); ++j) {
    char c = strs[0][j];
    for (int i = 1; i < strs.size(); ++i) {
      if (j >= strs[i].size() || strs[i][j] != c) {
        return res;
      }
    }
    res.push_back(c);
  }
  return res;
}
```

将输入的字符串数组进行排序。首尾字符串与其他的公共前缀最少。所以仅需要查看首尾字符串的公共前缀。

```Cpp
string longestCommonPrefix(vector<string> &strs) {
  if (strs.empty()) {
    return "";
  }
  sort(strs.begin(), strs.end());
  int i = 0, len = min(strs[0].size(), strs.back().size());
  while (i < len && strs[0][1] == strs.back()[i]) {
    ++i;
  }
  return strs[0].substr(0, i);
}
```

```Python
def longestCommonPrefix(self, strs):
    """
    :type strs: List[str]
    :rtype: str
    """
    if not strs:
        return ""
    s1 = min(strs)
    s2 = max(strs)
    for i, j in enumerate(s1):
        if j != s2[i]:
            return s1[:i]
    return s1
```