# Day 1 · 哈希与双指针

> **日期：** 2026-07-20
> **学习目标：** 哈希表的使用技巧与双指针的经典模式
> **相关知识页：** [[02-Wiki/专题总结/01-哈希表]] · [[02-Wiki/专题总结/02-双指针与滑动窗口]]

---

## 一、今日模板回顾

### 哈希表
```python
# 模板速记：值 → 索引映射
seen = {}
for i, num in enumerate(nums):
    if target - num in seen:
        return [seen[target - num], i]
    seen[num] = i
```

### 双指针（对撞）
```python
left, right = 0, len(nums) - 1
while left < right:
    if nums[left] + nums[right] == target:
        return [left, right]
    elif nums[left] + nums[right] < target:
        left += 1
    else:
        right -= 1
```

### 双指针（快慢）
```python
slow = fast = 0
while fast < len(nums):
    if nums[fast] != 0:
        nums[slow], nums[fast] = nums[fast], nums[slow]
        slow += 1
    fast += 1
```

---

## 二、做题记录

### 1. 两数之和（Easy）
- **核心思路：** 用哈希表存已遍历的数，一遍扫描找配对数（target - num）
- **代码实现：** 
  ```python
  def twoSum(nums, target):
      seen = {}
      for i, num in enumerate(nums):
          if target - num in seen:
              return [seen[target - num], i]
          seen[num] = i
      return []
  ```
- **复杂度：** O(n) / O(n)
- **掌握程度：** ✅
- **感悟/易错点：** 哈希表查询/插入都是O(1)，所以总时间O(n)；空间用来存储n个元素所以O(n)；不能用双向指针是因为数组可能无序且需要返回原索引

### 2. 字母异位词分组（Medium）
- **核心思路：** 用排序后的字符串作为哈希表key分组，异位词排序后完全相同
- **代码实现：**
  ```python
  from collections import defaultdict
  def groupAnagrams(strs):
      groups = defaultdict(list)
      for s in strs:
          sorted_s = ''.join(sorted(s))
          groups[sorted_s].append(s)
      return list(groups.values())
  ```
- **复杂度：** O(n·k log k) / O(n·k)  （n个字符串，每个长度k）
- **掌握程度：** ✅
- **感悟/易错点：** sorted()返回列表需转字符串；返回value不是key；可用defaultdict避免判断key存在

### 3. 最长连续序列（Medium）
- **核心思路：** 转集合O(1)查询；只从"起点"（num-1不存在）开始计数，避免重复访问
- **代码实现：**
  ```python
  def longestConsecutive(nums):
      nums_set = set(nums)
      max_length = 0
      for num in nums_set:
          if num - 1 not in nums_set:  # 只检查起点
              current = num
              length = 1
              while current + 1 in nums_set:
                  current += 1
                  length += 1
              max_length = max(max_length, length)
      return max_length
  ```
- **复杂度：** O(n) / O(n)
- **掌握程度：** ✅
- **感悟/易错点：** 虽然有两层循环但每元素最多访问一次所以O(n)；关键是只检查起点来剪枝；不能排序（O(n log n)）

### 4. 移动零（Easy）
- **核心思路：** 快慢指针；slow指向下一个非零元素位置；fast遇非零交换，遇零只移动
- **代码实现：**
  ```python
  def moveZeroes(nums):
      slow = 0
      for fast in range(len(nums)):
          if nums[fast] != 0:
              nums[slow], nums[fast] = nums[fast], nums[slow]
              slow += 1
  ```
- **复杂度：** O(n) / O(1)
- **掌握程度：** ✅
- **感悟/易错点：** 原地修改不能用remove/append；必须交换保持顺序；fast扫全数组slow指向非零位置

### 5. 盛最多水的容器（Medium）
- **核心思路：** 对撞指针两端夹逼；面积=宽×min(左高,右高)；每次移动较矮的一侧
- **代码实现：**
  ```python
  def maxArea(self, height):
      left, right = 0, len(height) - 1
      max_area = 0
      while left < right:
          area = min(height[left], height[right]) * (right - left)
          max_area = max(max_area, area)
          if height[left] < height[right]:
              left += 1
          else:
              right -= 1
      return max_area
  ```
- **复杂度：** O(n) / O(1)
- **掌握程度：** ✅
- **感悟/易错点：** 移动较大侧无意义（宽减小且高还被小侧限制）；独立完成

### 6. 三数之和（Medium）
- **核心思路：** 排序后固定第一个数，剩余用对撞指针找两数之和；跳过重复元素避免结果重复
- **代码实现：**
  ```python
  def threeSum(self, nums):
      nums.sort()
      result = []
      for i in range(len(nums) - 2):
          if nums[i] > 0:
              break
          if i > 0 and nums[i] == nums[i - 1]:
              continue
          left, right = i + 1, len(nums) - 1
          while left < right:
              total = nums[i] + nums[left] + nums[right]
              if total == 0:
                  result.append([nums[i], nums[left], nums[right]])
                  while left < right and nums[left] == nums[left + 1]:
                      left += 1
                  while left < right and nums[right] == nums[right - 1]:
                      right -= 1
                  left += 1
                  right -= 1
              elif total < 0:
                  left += 1
              else:
                  right -= 1
      return result
  ```
- **复杂度：** O(n²) / O(1)
- **掌握程度：** ✅
- **感悟/易错点：** 排序是对撞指针的前提；nums[i]>0可直接break（后面不可能为0）；找到解后要跳过重复的left和right

### 7. 接雨水（Hard）
- **核心思路：** 位置i水量 = min(left_max, right_max) - height[i]；对撞指针维护两侧最大值，移动较矮一侧
- **代码实现：**
  ```python
  def trap(self, height):
      left, right = 0, len(height) - 1
      left_max, right_max = 0, 0
      water = 0
      while left < right:
          left_max  = max(left_max,  height[left])
          right_max = max(right_max, height[right])
          if left_max < right_max:
              water += left_max - height[left]
              left += 1
          else:
              water += right_max - height[right]
              right -= 1
      return water
  ```
- **复杂度：** O(n) / O(1)
- **掌握程度：** ✅
- **感悟/易错点：** 水量由较矮的墙决定（min）；哪侧矮就处理哪侧，因为那侧水量已可确定；left_max/right_max要先更新再计算水量

---

## 三、今日总结

**学到的新模板/技巧：**
- 哈希表查找：值→索引映射，O(1)查询替代O(n)扫描
- 哈希表分组：sorted字符串作key，defaultdict(list)分组
- 哈希表去重+剪枝：转set后只检查"起点"实现O(n)
- 快慢指针：slow指向待填位置，fast扫描全数组
- 对撞指针：两端夹逼，移动较矮/较小一侧
- 对撞指针进阶：维护left_max/right_max，按列计算雨水

**遇到的困难：**
- 接雨水：水量公式理解（是min不是sum）
- 三数之和：去重逻辑（遍历时跳过比最后set去重更高效）

**遗留问题（需复习）：**
- 无

**整体感受：** 😊
