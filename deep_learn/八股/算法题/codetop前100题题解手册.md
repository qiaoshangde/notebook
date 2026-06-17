# CodeTop 前 100 题题解手册

数据来源：`https://codetop.cc/home` 前 5 页，每页 20 题，共 100 题。抓取时间：2026-06-17。

使用方式：

- 先看题名，先问自己：这题是什么类型？
- 再看“核心结构”：状态、指针、栈、队列、哈希表、递归返回值是什么？
- 最后背“记忆卡片”：面试时先说这个，再展开代码。

说明：

- 只有稳定题型才给模板；模拟题、数学概率题、特定技巧题不强行套模板。
- 代码默认 Python，偏 LeetCode 写法；需要 `List`、`Optional`、`deque`、`defaultdict`、`heapq` 等时，默认从标准库导入。

## 题型模板

### 滑动窗口模板

适用信号：连续子串、连续子数组、最长/最短、窗口内满足某条件。

核心：

- `left/right` 两个指针维护窗口 `[left, right]`。
- `cnt/window` 记录窗口内字符或数字状态。
- 右指针扩张，窗口不合法时左指针收缩。

```python
left = 0
ans = 0
window = {}
for right, x in enumerate(nums_or_s):
    # 加入 x，更新 window
    while 窗口不合法:
        # 移除 nums_or_s[left]
        left += 1
    ans = max(ans, right - left + 1)
```

### 双指针模板

适用信号：有序数组、两端夹逼、原地移动、链表快慢指针。

核心：

- 数组常见：`l/r` 从两端向中间走。
- 链表常见：`slow/fast` 制造距离差或判断环。

```python
l, r = 0, len(nums) - 1
while l < r:
    if 需要左移:
        l += 1
    else:
        r -= 1
```

### 链表哑节点模板

适用信号：删除、插入、反转局部、合并链表。

核心：

- `dummy` 处理头节点变化。
- `pre/cur/nxt` 控制断链和接链。

```python
dummy = ListNode(0, head)
pre = dummy
cur = head
while cur:
    nxt = cur.next
    # 改指针
    cur = nxt
return dummy.next
```

### 二分模板

适用信号：有序、答案单调、第一个/最后一个满足条件、O(log n)。

核心：

- 用左闭右闭或左闭右开都行，但全文统一左闭右闭。

```python
l, r = 0, len(nums) - 1
while l <= r:
    mid = (l + r) // 2
    if nums[mid] == target:
        return mid
    elif nums[mid] < target:
        l = mid + 1
    else:
        r = mid - 1
return -1
```

找左边界：

```python
l, r = 0, len(nums)
while l < r:
    mid = (l + r) // 2
    if nums[mid] >= target:
        r = mid
    else:
        l = mid + 1
```

### 树 DFS 模板

适用信号：二叉树深度、路径、最近公共祖先、最大路径和、验证 BST。

核心：

- 先定义递归函数返回什么。
- 树题最重要的是“当前节点向父节点贡献什么”。

```python
def dfs(root):
    if not root:
        return 空节点返回值
    left = dfs(root.left)
    right = dfs(root.right)
    return 当前节点返回值
```

### 树 BFS 模板

适用信号：层序遍历、右视图、宽度、最短层数。

```python
q = deque([root])
while q:
    size = len(q)
    level = []
    for _ in range(size):
        node = q.popleft()
        level.append(node.val)
        if node.left:
            q.append(node.left)
        if node.right:
            q.append(node.right)
```

### 回溯模板

适用信号：所有排列、所有组合、所有路径、括号生成、IP 复原。

核心：

- `path` 是当前选择。
- `start` 控制组合不回头，`used` 控制排列不重复。
- 做选择、递归、撤销选择。

```python
def backtrack(start):
    if 到达答案:
        ans.append(path[:])
        return
    for i in range(start, len(nums)):
        path.append(nums[i])
        backtrack(i + 1)
        path.pop()
```

### 动态规划模板

适用信号：最值、方案数、能否、子序列、路径。

核心：

- `dp[i]` 或 `dp[i][j]` 的含义必须先定死。
- 转移来自“最后一步怎么来”。

```python
dp = [初始值] * n
for i in range(n):
    for j in range(i):
        dp[i] = max/min/或(dp[i], dp[j] + ...)
```

### 单调栈/单调队列模板

适用信号：最近更大/更小、窗口最大值、接雨水。

单调栈：

```python
stack = []
for i, x in enumerate(nums):
    while stack and nums[stack[-1]] < x:
        j = stack.pop()
        # i 是 j 右边第一个更大值
    stack.append(i)
```

单调队列：

```python
q = deque()
for i, x in enumerate(nums):
    while q and nums[q[-1]] <= x:
        q.pop()
    q.append(i)
    if q[0] <= i - k:
        q.popleft()
    if i >= k - 1:
        ans.append(nums[q[0]])
```

### 图 BFS / 拓扑排序模板

适用信号：课程依赖、先后关系、是否能完成、有向图环。

```python
graph = [[] for _ in range(n)]
indeg = [0] * n
for a, b in prerequisites:
    graph[b].append(a)
    indeg[a] += 1
q = deque(i for i in range(n) if indeg[i] == 0)
seen = 0
while q:
    u = q.popleft()
    seen += 1
    for v in graph[u]:
        indeg[v] -= 1
        if indeg[v] == 0:
            q.append(v)
return seen == n
```

## 前 100 题题解卡片

### 1. 无重复字符的最长子串

- 类型：滑动窗口 + 哈希表。
- 分析：题目要求“最长子串”，子串必须连续，所以不能用子序列 DP。窗口里不能有重复字符，右指针扩张遇到重复时，左指针必须跳到重复字符上次位置之后。
- 思路：用 `last` 记录字符上次出现位置；遍历 `right`，若 `s[right]` 在窗口内出现过，则更新 `left=max(left,last[c]+1)`；每轮更新最长长度。
- 核心结构：`left` 窗口左边界，`last` 字符最后位置。
- 坑：`left` 只能右移，不能被旧重复字符拉回去；空串返回 0。
- 相似题：最小覆盖子串、长度最小的子数组、至多 K 个不同字符。
- 记忆卡片：连续且不重复，窗口；重复就让左边界跳过上一个它。

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        last = {}
        left = ans = 0
        for right, c in enumerate(s):
            if c in last and last[c] >= left:
                left = last[c] + 1
            last[c] = right
            ans = max(ans, right - left + 1)
        return ans
```


#### 详细分析与小例子
题目关键词是“子串”和“不重复”。子串必须连续，所以可以想象拿一个尺子框住字符串中的一段连续区域；右边界负责向右扩，左边界负责在窗口不合法时收缩。只要窗口里没有重复字符，就可以用当前窗口长度更新答案。

例子：`s = "abcabcbb"`。窗口先扩成 `"abc"`，长度为 3；再遇到第二个 `a` 时，`"abca"` 出现重复，左边界跳到第一个 `a` 后面，窗口变成 `"bca"`。整个过程里最大窗口长度是 3。

为什么不是 DP：这题的限制只和“当前窗口里有没有重复字符”有关，窗口左右移动就能维护，不需要记录复杂状态。

### 2. LRU 缓存机制

- 类型：哈希表 + 双向链表设计。
- 分析：`get/put` 都要 O(1)。哈希表能 O(1) 找节点，但不能维护最近使用顺序；双向链表能 O(1) 删除、插入节点。组合起来：哈希表定位节点，链表头表示最近使用，链表尾表示最久未使用。
- 思路：设置 dummy head/tail。`get` 找到节点后移动到头部；`put` 已存在则更新并移动，不存在则插入头部，超容量删除尾部前一个节点。
- 核心结构：`cache: key -> node`，双向链表。
- 坑：容量满时要同时删链表节点和哈希表；更新已有 key 不增加 size。
- 相似题：LFU 缓存、设计浏览器历史。
- 记忆卡片：LRU = 字典找人，链表排队；用过就搬到队头，淘汰队尾。

```python
class Node:
    def __init__(self, key=0, val=0):
        self.key = key
        self.val = val
        self.prev = None
        self.next = None

class LRUCache:
    def __init__(self, capacity: int):
        self.cap = capacity
        self.cache = {}
        self.head = Node()
        self.tail = Node()
        self.head.next = self.tail
        self.tail.prev = self.head

    def _remove(self, node):
        node.prev.next = node.next
        node.next.prev = node.prev

    def _add_first(self, node):
        node.next = self.head.next
        node.prev = self.head
        self.head.next.prev = node
        self.head.next = node

    def get(self, key: int) -> int:
        if key not in self.cache:
            return -1
        node = self.cache[key]
        self._remove(node)
        self._add_first(node)
        return node.val

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            node = self.cache[key]
            node.val = value
            self._remove(node)
            self._add_first(node)
            return
        node = Node(key, value)
        self.cache[key] = node
        self._add_first(node)
        if len(self.cache) > self.cap:
            old = self.tail.prev
            self._remove(old)
            del self.cache[old.key]
```


#### 详细分析与小例子
LRU 可以理解成一个“最近使用队列”：刚被访问过的数据放到队头，最久没访问的数据沉到队尾。`get` 和 `put` 都要求 O(1)，所以单用列表不行，因为移动元素和删除队尾附近元素可能不是 O(1)。

例子：容量为 2。执行 `put(1,1), put(2,2)` 后，最近使用顺序是 `[2,1]`。执行 `get(1)`，1 被访问，顺序变成 `[1,2]`。再执行 `put(3,3)`，容量超了，淘汰队尾 2，剩 `[3,1]`。

哈希表负责 O(1) 找到 key 对应的节点，双向链表负责 O(1) 删除节点和移动到队头。双向链表比单链表合适，因为删除当前节点时需要立刻找到前驱。

### 3. 反转链表

- 类型：链表指针反转。
- 分析：单链表不能向前走，所以遍历时必须保存下一个节点，再把当前节点指向前一个节点。反转的本质是把每条 `cur.next` 改成 `prev`。
- 思路：`prev=None, cur=head`；循环中保存 `nxt=cur.next`，执行 `cur.next=prev`，然后 `prev=cur, cur=nxt`。
- 核心结构：`prev/cur/nxt` 三指针。
- 坑：先保存 `nxt` 再断链；空链表返回 `None`。
- 相似题：反转链表 II、K 个一组翻转链表、回文链表。
- 记忆卡片：反转链表三步：存后、改向、前进。

```python
class Solution:
    def reverseList(self, head):
        prev, cur = None, head
        while cur:
            nxt = cur.next
            cur.next = prev
            prev = cur
            cur = nxt
        return prev
```


#### 详细分析与小例子
反转链表的本质是把每个节点的 `next` 指针反过来。因为单链表不能回头，一旦你把 `cur.next` 改掉，就会丢失后面的链表，所以必须先保存 `nxt = cur.next`。

例子：`1 -> 2 -> 3`。先让 `1.next = None`，反转部分是 `1 -> None`；再让 `2.next = 1`，反转部分是 `2 -> 1 -> None`；最后让 `3.next = 2`，得到 `3 -> 2 -> 1`。

记住三步顺序：保存后继、改变指向、三个指针整体前进。顺序错了很容易断链。

### 4. 数组中的第 K 个最大元素

- 类型：堆 / 快速选择。
- 分析：排序能做但 O(nlogn)，面试更常问 O(n) 平均的快速选择。第 K 大等价于升序下标 `n-k`。每次 partition 后，基准左边都小于它，右边都大于它，只递归目标所在一侧。
- 思路：随机选 pivot，分区后得到位置 `p`；若 `p==n-k` 返回；小了搜右边，大了搜左边。
- 核心结构：partition 的左右指针。
- 坑：题目要第 K 大，不是第 K 个不同元素；重复元素正常参与排名。
- 相似题：TopK 高频元素、最小的 K 个数、排序数组。
- 记忆卡片：第 K 大先转下标：`target = n - k`。

```python
class Solution:
    def findKthLargest(self, nums, k: int) -> int:
        import random
        target = len(nums) - k

        def partition(l, r):
            p = random.randint(l, r)
            nums[p], nums[r] = nums[r], nums[p]
            pivot = nums[r]
            i = l
            for j in range(l, r):
                if nums[j] <= pivot:
                    nums[i], nums[j] = nums[j], nums[i]
                    i += 1
            nums[i], nums[r] = nums[r], nums[i]
            return i

        l, r = 0, len(nums) - 1
        while True:
            p = partition(l, r)
            if p == target:
                return nums[p]
            if p < target:
                l = p + 1
            else:
                r = p - 1
```


#### 详细分析与小例子
题目只问第 K 大，不要求整个数组排好序。完整排序当然可以，但做了多余工作。快速选择每次 partition 后，可以确定一个 pivot 的最终排名，只需要继续处理目标排名所在的一边。

例子：`nums=[3,2,1,5,6,4], k=2`。第 2 大等价于升序排序后的下标 `len(nums)-k=4`。如果某次 partition 后 `5` 正好位于下标 4，那么它左边都不大于它，右边都不小于它，答案就是 5。

坑点是“第 K 大”不是“第 K 个不同的数”。数组里有重复数字时，重复数字也要按出现次数参与排名。

### 5. K 个一组翻转链表

- 类型：链表分组反转。
- 分析：本题不是反转值，而是改节点指针。每次先检查后面是否够 k 个，不够则保持原样。够 k 个后反转 `[group_start, group_end]`，再接回前后链表。
- 思路：用 `dummy` 和 `pre` 指向每组前一个节点；找到 `end`；保存 `nxt=end.next`；断开后反转这一段；接回；`pre` 移到原组头。
- 核心结构：`pre/end/start/nxt`。
- 坑：最后不足 k 个不能反转；反转后原 `start` 变成组尾。
- 相似题：反转链表、反转链表 II、两两交换链表节点。
- 记忆卡片：先数够 k，再断开反转，最后接回。

```python
class Solution:
    def reverseKGroup(self, head, k: int):
        def reverse(node):
            prev, cur = None, node
            while cur:
                nxt = cur.next
                cur.next = prev
                prev = cur
                cur = nxt
            return prev

        dummy = ListNode(0, head)
        pre = dummy
        while True:
            end = pre
            for _ in range(k):
                end = end.next
                if not end:
                    return dummy.next
            start = pre.next
            nxt = end.next
            end.next = None
            pre.next = reverse(start)
            start.next = nxt
            pre = start
```


#### 详细分析与小例子
这题难点不是反转，而是“分组”和“接回去”。每组反转前必须先确认后面至少还有 k 个节点；如果不够 k 个，最后那段必须原样保留。

例子：`1->2->3->4->5, k=2`。第一组 `1,2` 反转为 `2,1`；第二组 `3,4` 反转为 `4,3`；最后 `5` 不够 2 个，不动，结果是 `2->1->4->3->5`。

指针关系要特别小心：反转前的组头反转后会变成组尾，所以它要接到下一组的开头。通常用 `pre/end/start/nxt` 四个变量把边界固定住。

### 6. 三数之和

- 类型：排序 + 双指针。
- 分析：三元组去重是关键。先排序，固定第一个数 `i`，剩下问题变成两数之和。由于数组有序，和小了左指针右移，和大了右指针左移。
- 思路：遍历 `i`，跳过重复 `nums[i]`；用 `l=i+1,r=n-1` 找 `nums[l]+nums[r] == -nums[i]`；命中后左右都跳过重复值。
- 核心结构：固定指针 `i` + 双指针 `l/r`。
- 坑：不能返回重复三元组；`nums[i] > 0` 后可以提前结束。
- 相似题：两数之和、四数之和、最接近的三数之和。
- 记忆卡片：三数之和先排序，固定一个，双指针找另外两个。

```python
class Solution:
    def threeSum(self, nums):
        nums.sort()
        ans = []
        n = len(nums)
        for i in range(n - 2):
            if nums[i] > 0:
                break
            if i > 0 and nums[i] == nums[i - 1]:
                continue
            l, r = i + 1, n - 1
            while l < r:
                s = nums[i] + nums[l] + nums[r]
                if s == 0:
                    ans.append([nums[i], nums[l], nums[r]])
                    l += 1
                    r -= 1
                    while l < r and nums[l] == nums[l - 1]:
                        l += 1
                    while l < r and nums[r] == nums[r + 1]:
                        r -= 1
                elif s < 0:
                    l += 1
                else:
                    r -= 1
        return ans
```


#### 详细分析与小例子
三数之和最大的麻烦是去重。如果不排序，既难去重，也很难快速决定指针怎么移动。排序后，固定第一个数，剩下两个数就变成有序数组里的两数之和。

例子：`[-1,0,1,2,-1,-4]` 排序为 `[-4,-1,-1,0,1,2]`。固定第一个 `-1` 时，目标是找两个数和为 `1`。左指针在 `-1`，右指针在 `2`，正好得到 `[-1,-1,2]`。

去重有三处：固定的 `i` 要跳过重复值；找到答案后，左指针跳过重复值；右指针也跳过重复值。否则同一个三元组会出现多次。

### 7. 最大子数组和

- 类型：动态规划 / Kadane。
- 分析：连续子数组的最大和，只关心“以当前位置结尾”的最优值。如果前面的和是负数，接上它只会变差，不如从当前数重新开始。
- 思路：`cur` 表示以当前元素结尾的最大和；`cur=max(x,cur+x)`；`ans=max(ans,cur)`。
- 核心结构：`cur` 和 `ans`。
- 坑：全负数时答案是最大负数，所以初始化要用第一个元素，不能用 0。
- 相似题：乘积最大子数组、最大子矩阵、股票最佳时机。
- 记忆卡片：前缀为负就丢掉，从当前重新开始。

```python
class Solution:
    def maxSubArray(self, nums):
        cur = ans = nums[0]
        for x in nums[1:]:
            cur = max(x, cur + x)
            ans = max(ans, cur)
        return ans
```


#### 详细分析与小例子
连续子数组的关键是“以当前数字结尾”。如果前面累积出来的和已经是负数，那么把它接到当前数字前面只会拖累当前数字，不如从当前数字重新开始。

例子：`[-2,1,-3,4,-1,2,1,-5,4]`。走到 `4` 时，前面的连续和已经没价值，于是从 `4` 重新开始，后面得到 `4 + (-1) + 2 + 1 = 6`。

全负数是常见坑。答案不能初始化成 0，因为题目要求子数组至少包含一个元素；`[-3,-1,-2]` 的答案应该是 `-1`。

### 8. 手撕快速排序

- 类型：排序 / 快排。
- 分析：快排核心是 partition：选一个基准，把小于基准的放左边，大于基准的放右边，再递归两边。为了避免有序数组退化，基准要随机选。
- 思路：随机 pivot 放到末尾；扫描区间，把 `<= pivot` 的元素交换到左侧；pivot 放到最终位置；递归左右区间。
- 核心结构：`partition(l,r)`。
- 坑：不随机会在有序数据上退化；递归边界是 `l >= r`。
- 相似题：数组排序、第 K 大、颜色分类。
- 记忆卡片：快排 = 随机基准 + 一次分区 + 递归两边。

```python
class Solution:
    def sortArray(self, nums):
        import random

        def quicksort(l, r):
            if l >= r:
                return
            p = random.randint(l, r)
            nums[p], nums[r] = nums[r], nums[p]
            pivot = nums[r]
            i = l
            for j in range(l, r):
                if nums[j] <= pivot:
                    nums[i], nums[j] = nums[j], nums[i]
                    i += 1
            nums[i], nums[r] = nums[r], nums[i]
            quicksort(l, i - 1)
            quicksort(i + 1, r)

        quicksort(0, len(nums) - 1)
        return nums
```


#### 详细分析与小例子
快排的核心是 partition：选一个基准值，把小于等于它的数放左边，大于它的数放右边。partition 完成后，基准值的位置已经是最终排序后的位置。

例子：`[5,2,3,1]`，假设 pivot 是 3。一次分区后可能变成 `[2,1,3,5]`。此时 3 已经就位，后面只需要递归排序左边 `[2,1]` 和右边 `[5]`。

为什么随机 pivot：如果总选固定位置，在已经有序或接近有序的数据上容易退化成 O(n²)。随机选择能降低这种风险。

### 9. 最长回文子串

- 类型：中心扩展 / 字符串。
- 分析：回文串以中心向两边对称。中心可能是一个字符，也可能是两个字符之间，所以每个位置要扩展奇数和偶数两种中心。
- 思路：写 `expand(l,r)` 返回以该中心扩展出的最长边界；遍历每个 `i`，比较 `(i,i)` 和 `(i,i+1)`。
- 核心结构：中心左右指针 `l/r`。
- 坑：偶数长度回文不能漏；返回切片时右边界要 `r + 1`。
- 相似题：回文子串个数、最长回文子序列、回文链表。
- 记忆卡片：回文看中心，中心有单双两种。

```python
class Solution:
    def longestPalindrome(self, s: str) -> str:
        def expand(l, r):
            while l >= 0 and r < len(s) and s[l] == s[r]:
                l -= 1
                r += 1
            return l + 1, r - 1

        start = end = 0
        for i in range(len(s)):
            for l, r in (expand(i, i), expand(i, i + 1)):
                if r - l > end - start:
                    start, end = l, r
        return s[start:end + 1]
```


#### 详细分析与小例子
回文串的特点是左右对称。与其枚举所有子串再判断，不如枚举“中心”，然后向两边扩展。中心有两种：单个字符中心和两个字符之间的中心。

例子：`s="babad"`。以中间的 `a` 为中心，可以扩出 `"bab"` 或 `"aba"` 这样的长度 3 回文。`s="cbbd"` 的答案 `"bb"` 是偶数长度，中心在两个 `b` 之间。

所以每个位置都要尝试两次：`expand(i,i)` 处理奇数长度，`expand(i,i+1)` 处理偶数长度。

### 10. 合并两个有序链表

- 类型：链表双指针。
- 分析：两个链表已经有序，每次取较小节点接到结果链表后面即可。使用 dummy 可以避免单独处理头节点。
- 思路：`cur` 指向结果链表尾部；比较 `l1.val` 和 `l2.val`，接小的并前进；最后接剩余链表。
- 核心结构：`dummy/cur/l1/l2`。
- 坑：最后不要忘了接上未遍历完的一条链表。
- 相似题：合并 K 个排序链表、合并两个有序数组。
- 记忆卡片：两个有序合并，每次接小的，剩下整段接上。

```python
class Solution:
    def mergeTwoLists(self, list1, list2):
        dummy = ListNode()
        cur = dummy
        while list1 and list2:
            if list1.val <= list2.val:
                cur.next = list1
                list1 = list1.next
            else:
                cur.next = list2
                list2 = list2.next
            cur = cur.next
        cur.next = list1 or list2
        return dummy.next
```


#### 详细分析与小例子
两个链表已经有序，所以每次只需要比较两个链表当前头节点，谁小就把谁接到结果后面。`dummy` 节点的作用是避免单独处理结果链表的第一个节点。

例子：`1->2->4` 和 `1->3->4`。先接第一个 1，再接第二个 1，然后接 2、3、4、4，结果是 `1->1->2->3->4->4`。

当其中一条链表走完时，另一条链表剩下部分已经有序，可以整段接到结果后面，不需要继续逐个比较。

### 11. 岛屿数量

- 类型：网格 DFS/BFS。
- 分析：一个岛屿就是一片连通的 `'1'`。遍历网格，遇到未访问陆地就计数一次，并把整座岛淹掉，避免重复计数。
- 思路：双层循环找 `'1'`；`dfs(i,j)` 越界或非陆地返回；否则改成 `'0'`，递归四个方向。
- 核心结构：网格 DFS，方向数组。
- 坑：字符是 `'1'/'0'`，不是整数；访问后要标记。
- 相似题：岛屿最大面积、被围绕的区域、腐烂橘子。
- 记忆卡片：遇到一块陆地，答案加一，然后整座岛淹掉。

```python
class Solution:
    def numIslands(self, grid):
        m, n = len(grid), len(grid[0])

        def dfs(i, j):
            if i < 0 or i >= m or j < 0 or j >= n or grid[i][j] != '1':
                return
            grid[i][j] = '0'
            dfs(i + 1, j)
            dfs(i - 1, j)
            dfs(i, j + 1)
            dfs(i, j - 1)

        ans = 0
        for i in range(m):
            for j in range(n):
                if grid[i][j] == '1':
                    ans += 1
                    dfs(i, j)
        return ans
```


#### 详细分析与小例子
一个岛屿就是上下左右连通的一整片陆地。遍历网格时，第一次遇到某座岛的 `1`，答案加 1，然后用 DFS/BFS 把这座岛全部标记为访问过，避免后面重复计数。

例子：左上角有一片连在一起的 `1`。扫到第一个 `1` 时答案加 1，DFS 会把它上下左右能连到的所有 `1` 都改成 `0`。后面再扫到这些格子时，就不会再加答案。

网格 DFS 的固定写法是：越界返回、不是目标值返回、标记当前格子、递归四个方向。

### 12. 二叉树的层序遍历

- 类型：树 BFS。
- 分析：层序遍历天然用队列。每次处理当前队列长度个节点，这些节点正好属于同一层。
- 思路：根节点入队；循环中记录 `size=len(q)`；弹出 `size` 次，收集当前层并加入左右孩子。
- 核心结构：队列 `q` 和每层 `level`。
- 坑：空树返回 `[]`；每层长度必须在进入该层时固定。
- 相似题：锯齿形层序、右视图、最大宽度。
- 记忆卡片：BFS 分层靠 `size`，一层处理 `size` 个。

```python
class Solution:
    def levelOrder(self, root):
        if not root:
            return []
        from collections import deque
        q = deque([root])
        ans = []
        while q:
            level = []
            for _ in range(len(q)):
                node = q.popleft()
                level.append(node.val)
                if node.left:
                    q.append(node.left)
                if node.right:
                    q.append(node.right)
            ans.append(level)
        return ans
```


#### 详细分析与小例子
层序遍历就是一层一层访问。队列先进先出，正好适合 BFS。为了分层，每轮开始先记录队列长度 `size`，这 `size` 个节点就是当前层的全部节点。

例子：树 `[3,9,20,null,null,15,7]`。队列先是 `[3]`，处理后得到第一层 `[3]`，并把 `9,20` 入队。下一轮队列长度是 2，处理得到 `[9,20]`。

注意不能在 for 循环里动态使用变化后的队列长度，否则下一层节点会混进当前层。

### 13. 搜索旋转排序数组

- 类型：变形二分。
- 分析：旋转数组整体不有序，但每次二分后，左半或右半至少有一边是有序的。先判断哪边有序，再判断 target 是否落在这边。
- 思路：若 `nums[l] <= nums[mid]`，左边有序；若 `nums[l] <= target < nums[mid]` 搜左，否则搜右。右边同理。
- 核心结构：`l/mid/r`。
- 坑：题目元素互不相同；边界用 `<=` 判断左边有序。
- 相似题：寻找旋转数组最小值、搜索旋转数组 II。
- 记忆卡片：旋转二分先找有序半边，再看目标在不在里面。

```python
class Solution:
    def search(self, nums, target: int) -> int:
        l, r = 0, len(nums) - 1
        while l <= r:
            mid = (l + r) // 2
            if nums[mid] == target:
                return mid
            if nums[l] <= nums[mid]:
                if nums[l] <= target < nums[mid]:
                    r = mid - 1
                else:
                    l = mid + 1
            else:
                if nums[mid] < target <= nums[r]:
                    l = mid + 1
                else:
                    r = mid - 1
        return -1
```


#### 详细分析与小例子
旋转数组整体不是有序的，但二分切一刀后，左半或右半至少有一边是有序的。我们先判断哪边有序，再判断 target 是否落在有序那边，从而排除另一半。

例子：`[4,5,6,7,0,1,2]`，中点是 7。左半 `[4,5,6,7]` 有序。target 是 0，不在 4 到 7 之间，所以去右半 `[0,1,2]` 搜。

如果元素有重复，判断会更麻烦；本题说明互不相同，所以 `nums[l] <= nums[mid]` 可以可靠判断左半有序。

### 14. 两数之和

- 类型：哈希表。
- 分析：暴力枚举两数是 O(n²)。遍历到当前数 `x` 时，只需要知道之前是否出现过 `target-x`，所以用哈希表记录已遍历数字的位置。
- 思路：遍历 `nums`；若 `target-x` 在字典里，返回对应下标和当前下标；否则记录 `x`。
- 核心结构：`seen: 数值 -> 下标`。
- 坑：不能同一个元素用两次，所以先查再存。
- 相似题：三数之和、四数之和、两数之和 II。
- 记忆卡片：一边遍历一边问：之前有没有人能和我凑 target？

```python
class Solution:
    def twoSum(self, nums, target: int):
        seen = {}
        for i, x in enumerate(nums):
            if target - x in seen:
                return [seen[target - x], i]
            seen[x] = i
```


#### 详细分析与小例子
遍历到当前数字 `x` 时，只需要知道前面是否出现过 `target - x`。这个“是否出现过”和“出现在哪里”用哈希表最合适。

例子：`nums=[2,7,11,15], target=9`。看到 2 时，问前面有没有 7，没有，于是存下 `2 -> 0`。看到 7 时，问前面有没有 2，有，直接返回 `[0,1]`。

必须先查再存，避免同一个元素被使用两次。比如当前数字是 3，target 是 6，不能拿当前这个 3 和自己配对。

### 15. 全排列

- 类型：回溯。
- 分析：排列问题每个位置都可以选择还没用过的数。需要 `used` 标记哪些数已经放入当前路径。
- 思路：当 `path` 长度等于 `nums` 长度时加入答案；否则枚举每个未使用数字，选择、递归、撤销。
- 核心结构：`path` 和 `used`。
- 坑：加入答案要拷贝 `path[:]`；本题无重复数字，不需要去重。
- 相似题：全排列 II、子集、组合总和。
- 记忆卡片：排列用 used，组合用 start。

```python
class Solution:
    def permute(self, nums):
        ans, path = [], []
        used = [False] * len(nums)

        def backtrack():
            if len(path) == len(nums):
                ans.append(path[:])
                return
            for i, x in enumerate(nums):
                if used[i]:
                    continue
                used[i] = True
                path.append(x)
                backtrack()
                path.pop()
                used[i] = False

        backtrack()
        return ans
```


#### 详细分析与小例子
排列关心顺序，`[1,2]` 和 `[2,1]` 是不同答案。因此每一层都可以从所有数字里选择，只是已经在当前路径里的数字不能再用，所以需要 `used` 数组。

例子：`[1,2,3]`。第一位选 1 后，第二位可以选 2 或 3；如果第二位选 2，第三位只能选 3，得到 `[1,2,3]`。回退后第二位选 3，得到 `[1,3,2]`。

排列用 `used`，组合通常用 `start`。这是区分排列题和组合题的一个重要信号。

### 16. 有效的括号

- 类型：栈。
- 分析：括号匹配满足后进先出，最后出现的左括号必须最先被匹配，所以用栈。
- 思路：遇到左括号入栈；遇到右括号，栈空或栈顶不匹配则失败；最后栈必须为空。
- 核心结构：栈保存等待匹配的左括号。
- 坑：最后有剩余左括号也是 false；遇到右括号时栈可能为空。
- 相似题：最长有效括号、括号生成、字符串解码。
- 记忆卡片：括号题先想栈；右括号只看栈顶。

```python
class Solution:
    def isValid(self, s: str) -> bool:
        mp = {')': '(', ']': '[', '}': '{'}
        st = []
        for c in s:
            if c in mp:
                if not st or st[-1] != mp[c]:
                    return False
                st.pop()
            else:
                st.append(c)
        return not st
```


#### 详细分析与小例子
括号匹配符合“后出现的左括号先被匹配”，这就是栈的后进先出。遇到左括号就入栈，遇到右括号就检查栈顶是不是对应的左括号。

例子：`"([])"`。读到 `(` 入栈，读到 `[` 入栈；读到 `]` 时栈顶必须是 `[`；读到 `)` 时栈顶必须是 `(`。最后栈空，说明全部匹配。

反例：`"([)]"`。读到 `)` 时，栈顶是 `[`，不是 `(`，所以不合法。

### 17. 合并两个有序数组

- 类型：数组双指针 / 原地合并。
- 分析：`nums1` 后面有空位。如果从前往后合并，会覆盖还没处理的 `nums1` 元素；所以从后往前填最大值。
- 思路：`i=m-1,j=n-1,k=m+n-1`；每次把较大者放到 `nums1[k]`；最后若 `nums2` 剩余，继续填入。
- 核心结构：三个尾指针。
- 坑：只需要处理 `nums2` 剩余；`nums1` 剩余本来就在原位。
- 相似题：合并两个有序链表、归并排序。
- 记忆卡片：有空位在尾部，就从尾部放最大值。

```python
class Solution:
    def merge(self, nums1, m: int, nums2, n: int) -> None:
        i, j, k = m - 1, n - 1, m + n - 1
        while j >= 0:
            if i >= 0 and nums1[i] > nums2[j]:
                nums1[k] = nums1[i]
                i -= 1
            else:
                nums1[k] = nums2[j]
                j -= 1
            k -= 1
```


#### 详细分析与小例子
`nums1` 后面预留了空位。如果从前往后合并，会覆盖 `nums1` 里还没处理的有效元素。从后往前放最大值，就能安全使用这些空位。

例子：`nums1=[1,2,3,0,0,0], nums2=[2,5,6]`。从尾部比较 3 和 6，放 6；比较 3 和 5，放 5；比较 3 和 2，放 3；最后得到 `[1,2,2,3,5,6]`。

如果 `nums1` 前面还有剩余，不需要动，因为它已经在正确位置；如果 `nums2` 还有剩余，就要继续填入。

### 18. 买卖股票的最佳时机

- 类型：贪心 / 一次遍历。
- 分析：只能买卖一次。遍历到某天卖出时，最优买入价一定是之前最低价格，所以维护历史最低价即可。
- 思路：遍历价格；更新 `min_price`；用 `price-min_price` 更新最大利润。
- 核心结构：历史最小值 `min_price`。
- 坑：不能先卖后买；利润最小为 0。
- 相似题：买卖股票 II、含冷冻期股票、最大子数组和。
- 记忆卡片：今天卖赚多少？看之前最低买入价。

```python
class Solution:
    def maxProfit(self, prices):
        min_price = float('inf')
        ans = 0
        for p in prices:
            min_price = min(min_price, p)
            ans = max(ans, p - min_price)
        return ans
```


#### 详细分析与小例子
只能交易一次，所以站在每一天想：如果今天卖出，最好的买入价一定是今天之前见过的最低价。因此遍历时维护一个历史最低价即可。

例子：`[7,1,5,3,6,4]`。看到 1，最低价变成 1；看到 5，利润是 4；看到 6，利润是 5，更新答案。最终最大利润是 5。

注意不能先卖后买，所以最低价只能来自当前天之前或当前天。

### 19. 反转链表 II

- 类型：链表局部反转。
- 分析：只反转 `[left,right]`，需要先走到反转段前一个节点 `pre`。局部反转常用头插法：每次把 `cur.next` 拿出来插到 `pre.next` 前面。
- 思路：`pre` 到 left 前；`cur=pre.next`；循环 `right-left` 次，把 `nxt=cur.next` 摘下插到 `pre` 后。
- 核心结构：`pre/cur/nxt`。
- 坑：使用 dummy 处理 `left=1`；循环次数是 `right-left`。
- 相似题：反转链表、K 个一组翻转链表。
- 记忆卡片：局部反转用头插，`cur` 留在组尾不动。

```python
class Solution:
    def reverseBetween(self, head, left: int, right: int):
        dummy = ListNode(0, head)
        pre = dummy
        for _ in range(left - 1):
            pre = pre.next
        cur = pre.next
        for _ in range(right - left):
            nxt = cur.next
            cur.next = nxt.next
            nxt.next = pre.next
            pre.next = nxt
        return dummy.next
```


#### 详细分析与小例子
局部反转可以用头插法。`pre` 固定在反转区间前一个节点，`cur` 固定为反转区间的尾部候选；每次把 `cur.next` 摘下来，插到 `pre.next` 前面。

例子：`1->2->3->4->5, left=2, right=4`。`pre=1, cur=2`。先把 3 摘出来插到 1 后面，变成 `1->3->2->4->5`；再把 4 摘出来插到 1 后面，变成 `1->4->3->2->5`。

循环次数是 `right-left`，因为第一个节点已经在反转段里，不需要额外移动。

### 20. 二叉树的锯齿形层序遍历

- 类型：树 BFS。
- 分析：仍然是层序遍历，只是偶数层从左到右，奇数层从右到左。可以每层收集后按方向反转。
- 思路：BFS 分层；用 `left_to_right` 标记方向；当前层结束后必要时 `level[::-1]`。
- 核心结构：队列 + 方向布尔值。
- 坑：不要在队列入队顺序上复杂化，统一左子右子入队即可。
- 相似题：层序遍历、右视图、最大宽度。
- 记忆卡片：锯齿只是层结果反向，不是遍历结构变了。

```python
class Solution:
    def zigzagLevelOrder(self, root):
        if not root:
            return []
        from collections import deque
        q = deque([root])
        ans = []
        left_to_right = True
        while q:
            level = []
            for _ in range(len(q)):
                node = q.popleft()
                level.append(node.val)
                if node.left:
                    q.append(node.left)
                if node.right:
                    q.append(node.right)
            ans.append(level if left_to_right else level[::-1])
            left_to_right = not left_to_right
        return ans
```


#### 详细分析与小例子
锯齿形遍历本质仍然是层序遍历，只是每一层输出方向交替变化。队列入队顺序不用改，仍然左孩子再右孩子，输出时按方向决定是否反转当前层。

例子：层序结果是 `[3]`, `[9,20]`, `[15,7]`。锯齿形输出第一层 `[3]`，第二层反向 `[20,9]`，第三层正向 `[15,7]`。

不要为了锯齿而改变 BFS 的基本结构，否则容易把层顺序弄乱。

### 21. 最长上升子序列

- 类型：动态规划 / 贪心二分。
- 分析：普通 DP 是 `dp[i]` 表示以 `nums[i]` 结尾的 LIS 长度，O(n²)。更优解维护 `tails[len]`：长度为 `len+1` 的上升子序列的最小结尾。结尾越小，后续越容易接。
- 思路：遍历 `x`，在 `tails` 中找第一个 `>=x` 的位置替换；若不存在则追加。
- 核心结构：`tails` + 二分。
- 坑：替换不是改变真实序列，而是维护更优结尾；严格上升用 `bisect_left`。
- 相似题：俄罗斯套娃信封、最长递增子序列个数。
- 记忆卡片：LIS 优化记 `tails`：同长度下结尾越小越好。

```python
class Solution:
    def lengthOfLIS(self, nums):
        import bisect
        tails = []
        for x in nums:
            i = bisect.bisect_left(tails, x)
            if i == len(tails):
                tails.append(x)
            else:
                tails[i] = x
        return len(tails)
```


#### 详细分析与小例子
`tails[i]` 表示“长度为 i+1 的上升子序列，最小可能结尾是多少”。它不一定是真实答案序列，但它能帮助我们判断当前数字可以接在哪个长度后面。

例子：`[10,9,2,5,3,7,101,18]`。读到 2，`tails=[2]`；读到 5，变成 `[2,5]`；读到 3，长度为 2 的序列结尾从 5 优化为 3，`tails=[2,3]`；读到 7，变成 `[2,3,7]`。

结尾越小，后面越容易接上更大的数字。因为 `tails` 是递增的，所以可以用二分找替换位置。

### 22. 二叉树的最近公共祖先

- 类型：树 DFS。
- 分析：如果当前节点是 `p/q`，它本身可能就是最近公共祖先。左右子树分别递归寻找 `p/q`，若左右都找到了，当前节点就是答案；若只一边找到，就把那边结果向上返回。
- 思路：空节点返回空；命中 `p/q` 返回当前节点；递归左右；左右都非空返回 root，否则返回非空一侧。
- 核心结构：递归返回“当前子树里找到的目标节点或祖先”。
- 坑：不要额外判断父指针；`p` 是 `q` 祖先时直接返回 `p`。
- 相似题：BST 最近公共祖先、二叉树最大路径和。
- 记忆卡片：左右各找到一个，当前就是 LCA；只找到一个，继续往上交。

```python
class Solution:
    def lowestCommonAncestor(self, root, p, q):
        if not root or root == p or root == q:
            return root
        left = self.lowestCommonAncestor(root.left, p, q)
        right = self.lowestCommonAncestor(root.right, p, q)
        if left and right:
            return root
        return left or right
```


#### 详细分析与小例子
递归函数可以理解为：在当前子树里寻找 p 或 q，如果找到了就返回找到的节点；如果左右两边都找到了，说明当前节点就是最近公共祖先。

例子：如果 p 在 root 的左子树，q 在 root 的右子树，那么左递归返回 p，右递归返回 q，当前 root 同时收到左右非空，所以 root 是答案。

如果 p 本身就是 q 的祖先，那么递归走到 p 时直接返回 p，后面会一路把 p 往上返回。

### 23. 合并 K 个排序链表

- 类型：堆 / 分治合并。
- 分析：K 条链表都已排序。每次从 K 个头节点里取最小值即可，最小堆能把取最小降到 O(log k)。Python 堆遇到值相同的节点不能直接比较节点，所以要加一个唯一序号。
- 思路：所有非空头节点入堆 `(val, idx, node)`；弹出最小节点接到结果；若该节点有 next，继续入堆。
- 核心结构：最小堆。
- 坑：堆元素要带 `idx` 防止节点比较报错；空链表要跳过。
- 相似题：合并两个有序链表、数组中第 K 大。
- 记忆卡片：K 路归并，用堆管住 K 个当前最小头。

```python
class Solution:
    def mergeKLists(self, lists):
        import heapq
        heap = []
        for i, node in enumerate(lists):
            if node:
                heapq.heappush(heap, (node.val, i, node))
        dummy = ListNode()
        cur = dummy
        idx = len(lists)
        while heap:
            _, _, node = heapq.heappop(heap)
            cur.next = node
            cur = cur.next
            if node.next:
                idx += 1
                heapq.heappush(heap, (node.next.val, idx, node.next))
        return dummy.next
```


#### 详细分析与小例子
合并两个链表时，每次从两个头里选小的。合并 K 个链表时，就是每次从 K 个当前头节点里选最小。最小堆非常适合反复取最小值。

例子：三个链表当前头分别是 1、1、2。堆先弹出一个 1，把它接到结果后，再把这个节点的 next 放入堆。堆始终保存每条链当前能选择的最小节点。

Python 堆在值相同的时候会继续比较节点对象，可能报错，所以堆元素里要加一个唯一序号。

### 24. 螺旋矩阵

- 类型：矩阵模拟。
- 分析：按层收缩边界。每一圈依次走上边、右边、下边、左边。走完一圈后上下左右边界向内缩。
- 思路：维护 `top,bottom,left,right`；每次按四个方向遍历；每走完一条边就收缩；循环条件是 `top<=bottom and left<=right`。
- 核心结构：四个边界。
- 坑：单行或单列时，下边/左边遍历前要重新检查边界。
- 相似题：旋转图像、矩阵置零。
- 记忆卡片：螺旋矩阵不是 DFS，是四条边界一圈圈收。

```python
class Solution:
    def spiralOrder(self, matrix):
        ans = []
        top, bottom = 0, len(matrix) - 1
        left, right = 0, len(matrix[0]) - 1
        while top <= bottom and left <= right:
            for j in range(left, right + 1):
                ans.append(matrix[top][j])
            top += 1
            for i in range(top, bottom + 1):
                ans.append(matrix[i][right])
            right -= 1
            if top <= bottom:
                for j in range(right, left - 1, -1):
                    ans.append(matrix[bottom][j])
                bottom -= 1
            if left <= right:
                for i in range(bottom, top - 1, -1):
                    ans.append(matrix[i][left])
                left += 1
        return ans
```


#### 详细分析与小例子
螺旋遍历可以看成一圈一圈剥掉矩阵外壳。每一圈按上边、右边、下边、左边的顺序走，走完一圈后四个边界向内收缩。

例子：`[[1,2,3],[4,5,6],[7,8,9]]`。先走上边 `1,2,3`，右边 `6,9`，下边反向 `8,7`，左边向上 `4`，最后剩中心 `5`。

最容易错的是单行或单列。走下边和左边前，要检查当前上下边界、左右边界是否仍然有效。

### 25. 重排链表

- 类型：链表中点 + 反转 + 合并。
- 分析：目标是 `L0→Ln→L1→Ln-1...`。先用快慢指针找到中点，把后半段反转，再把前半段和反转后的后半段交替合并。
- 思路：找中点 `slow`；断开后半段并反转；两个链表交替插入。
- 核心结构：快慢指针、反转链表、交替合并。
- 坑：必须先断开 `slow.next=None`；奇数长度中点留在前半段。
- 相似题：回文链表、反转链表、两两交换节点。
- 记忆卡片：重排链表三件套：找中点、反后半、交替合。

```python
class Solution:
    def reorderList(self, head) -> None:
        if not head or not head.next:
            return
        slow, fast = head, head.next
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
        second = slow.next
        slow.next = None

        prev = None
        while second:
            nxt = second.next
            second.next = prev
            prev = second
            second = nxt

        first, second = head, prev
        while second:
            n1, n2 = first.next, second.next
            first.next = second
            second.next = n1
            first, second = n1, n2
```


#### 详细分析与小例子
目标顺序是 `L0 -> Ln -> L1 -> Ln-1 ...`，后半段需要倒着插入前半段。所以可以拆成三步：找中点、反转后半段、前后两段交替合并。

例子：`1->2->3->4->5`。先拆成前半 `1->2->3` 和后半 `4->5`；反转后半得到 `5->4`；交替合并得到 `1->5->2->4->3`。

拆分时要把前半段尾部断开，否则合并时链表可能连回旧节点形成环。

### 26. 环形链表

- 类型：链表快慢指针。
- 分析：如果链表有环，快指针每次走两步，慢指针每次走一步，快指针最终会追上慢指针；如果无环，快指针会先到空。
- 思路：`slow=head, fast=head`；循环条件 `fast and fast.next`；相遇返回 True，否则 False。
- 核心结构：`slow/fast`。
- 坑：循环条件必须检查 `fast.next`；空链表无环。
- 相似题：环形链表 II、快乐数。
- 记忆卡片：有环就像操场跑圈，快的一定追上慢的。

```python
class Solution:
    def hasCycle(self, head) -> bool:
        slow = fast = head
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
            if slow == fast:
                return True
        return False
```


#### 详细分析与小例子
如果链表有环，快指针每次走两步，慢指针每次走一步。慢指针进入环后，快指针相当于在环形跑道上追慢指针，每轮距离缩短 1，最终一定相遇。

例子：链表尾部连回第二个节点。slow 和 fast 都进入环后，fast 不会遇到空，而是在环里绕，最后追上 slow。

如果没有环，fast 或 fast.next 会先变成空，这时就能确定无环。

### 27. 字符串相加

- 类型：字符串模拟 / 大数加法。
- 分析：不能直接转整数时，就按小学加法从低位到高位相加，维护进位。
- 思路：两个指针从字符串尾部开始；每轮取当前位，没有则取 0；计算 `s=val1+val2+carry`，保存个位，更新进位。
- 核心结构：尾指针和 `carry`。
- 坑：最后进位不能漏；结果是反向生成，最后要反转。
- 相似题：两数相加、字符串相乘、二进制求和。
- 记忆卡片：字符串加法就是倒着扫，带进位。

```python
class Solution:
    def addStrings(self, num1: str, num2: str) -> str:
        i, j = len(num1) - 1, len(num2) - 1
        carry = 0
        ans = []
        while i >= 0 or j >= 0 or carry:
            a = ord(num1[i]) - ord('0') if i >= 0 else 0
            b = ord(num2[j]) - ord('0') if j >= 0 else 0
            s = a + b + carry
            ans.append(str(s % 10))
            carry = s // 10
            i -= 1
            j -= 1
        return ''.join(reversed(ans))
```


#### 详细分析与小例子
字符串里的个位在最后面，数字加法又必须从个位开始，所以两个指针从末尾向前扫，模拟小学竖式加法。

例子：`"456" + "77"`。先算 `6+7=13`，写 3 进 1；再算 `5+7+1=13`，写 3 进 1；最后算 `4+0+1=5`，结果反转后是 `"533"`。

最后如果还有进位不能忘，比如 `"99" + "1"` 最后要补出最高位 1。

### 28. 合并区间

- 类型：排序 + 区间合并。
- 分析：区间合并的前提是按左端点排序。遍历时只需要看当前区间是否和答案最后一个区间重叠。
- 思路：排序；若答案为空或当前左端点大于上个右端点，新增区间；否则更新上个右端点为最大值。
- 核心结构：排序后的区间列表和 `ans[-1]`。
- 坑：边界相等算重叠，比如 `[1,4]` 和 `[4,5]` 要合并。
- 相似题：插入区间、无重叠区间、会议室。
- 记忆卡片：区间题先按左端点排序，再只盯上一个右端点。

```python
class Solution:
    def merge(self, intervals):
        intervals.sort(key=lambda x: x[0])
        ans = []
        for l, r in intervals:
            if not ans or l > ans[-1][1]:
                ans.append([l, r])
            else:
                ans[-1][1] = max(ans[-1][1], r)
        return ans
```


#### 详细分析与小例子
区间合并必须先排序。按左端点排序后，当前区间只需要和结果里的最后一个区间比较：如果当前左端点不超过最后区间右端点，就说明有重叠。

例子：`[[1,3],[2,6],[8,10]]`。先放 `[1,3]`；看到 `[2,6]`，2 <= 3，合并成 `[1,6]`；看到 `[8,10]`，8 > 6，新开一个区间。

边界相等也算重叠：`[1,4]` 和 `[4,5]` 要合并为 `[1,5]`。

### 29. 编辑距离

- 类型：二维动态规划。
- 分析：`dp[i][j]` 表示 `word1` 前 i 个字符变成 `word2` 前 j 个字符的最少操作数。最后一步要么字符相同不操作，要么插入、删除、替换三选一。
- 思路：初始化空串变换；若 `word1[i-1]==word2[j-1]`，继承 `dp[i-1][j-1]`；否则取三种操作最小 +1。
- 核心结构：`dp[i][j]`。
- 坑：下标和长度错位，字符取 `i-1/j-1`；空串初始化不能漏。
- 相似题：最长公共子序列、单词拆分。
- 记忆卡片：编辑距离三操作：删、插、改；二维 DP 看最后一个字符。

```python
class Solution:
    def minDistance(self, word1: str, word2: str) -> int:
        m, n = len(word1), len(word2)
        dp = [[0] * (n + 1) for _ in range(m + 1)]
        for i in range(m + 1):
            dp[i][0] = i
        for j in range(n + 1):
            dp[0][j] = j
        for i in range(1, m + 1):
            for j in range(1, n + 1):
                if word1[i - 1] == word2[j - 1]:
                    dp[i][j] = dp[i - 1][j - 1]
                else:
                    dp[i][j] = 1 + min(dp[i - 1][j], dp[i][j - 1], dp[i - 1][j - 1])
        return dp[m][n]
```


#### 详细分析与小例子
`dp[i][j]` 表示 word1 前 i 个字符变成 word2 前 j 个字符的最少操作。看最后一个字符：相同就不用动它；不同就从插入、删除、替换三种操作里选最小。

例子：把 `"horse"` 变成 `"ros"`。当最后字符不同，我们可以删除 word1 最后一个字符，也可以插入 word2 需要的字符，也可以把当前字符替换掉。DP 表格就是把这些选择都记下来。

初始化也很直观：一个长度为 i 的字符串变成空串，需要删除 i 次，所以 `dp[i][0]=i`。

### 30. 相交链表

- 类型：链表双指针。
- 分析：两条链表长度可能不同。让两个指针分别走 A+B 和 B+A，总路程相同；若相交，会在交点相遇；若不相交，会同时到 `None`。
- 思路：`p=headA,q=headB`；每次前进，走到空就切换到另一条链表头；直到 `p==q`。
- 核心结构：双指针换路。
- 坑：比较的是节点对象，不是值。
- 相似题：环形链表、删除倒数第 N 个节点。
- 记忆卡片：你走我的路，我走你的路，路程补齐后交点见。

```python
class Solution:
    def getIntersectionNode(self, headA, headB):
        p, q = headA, headB
        while p != q:
            p = p.next if p else headB
            q = q.next if q else headA
        return p
```


#### 详细分析与小例子
两条链表可能长度不同，所以两个指针同时从头走不一定能同时到交点。让 A 指针走完 A 后去走 B，让 B 指针走完 B 后去走 A，它们走过的总长度就相同了。

例子：A 独有 2 个节点，B 独有 3 个节点，后面共享 4 个节点。A 指针走 `2+共享+3`，B 指针走 `3+共享+2`，长度差被抵消，最终会在共享部分的第一个节点相遇。

判断相交比较的是节点对象是否相同，不是节点值相同。

### 31. 接雨水

- 类型：双指针 / 单调栈。
- 分析：每个位置能接多少水，取决于左边最高和右边最高的较小值。双指针维护左右最大值，哪边最大值更小，哪边就可以确定水量。
- 思路：`l/r` 从两端走；维护 `left_max/right_max`；若左最大较小，处理左边，否则处理右边。
- 核心结构：双指针 + 两侧最高墙。
- 坑：水量不能为负；移动较低侧。
- 相似题：柱状图最大矩形、盛最多水的容器。
- 记忆卡片：水位由短板决定，哪边短先结算哪边。

```python
class Solution:
    def trap(self, height):
        l, r = 0, len(height) - 1
        left_max = right_max = 0
        ans = 0
        while l < r:
            left_max = max(left_max, height[l])
            right_max = max(right_max, height[r])
            if left_max < right_max:
                ans += left_max - height[l]
                l += 1
            else:
                ans += right_max - height[r]
                r -= 1
        return ans
```


#### 详细分析与小例子
每个位置能接多少水，取决于左边最高墙和右边最高墙的较小值。双指针维护两边最高值，哪边最高值更小，就说明哪边当前位置的水量已经可以确定。

例子：高度 `[0,1,0,2]`。第三个位置高度是 0，左边最高是 1，右边最高至少是 2，所以它能接 `1-0=1` 格水。

这题也能用单调栈，但双指针更适合记忆：水由短板决定，短板那边先结算。

### 32. 最长公共子序列

- 类型：二维动态规划。
- 分析：子序列可以不连续。`dp[i][j]` 表示 text1 前 i 个和 text2 前 j 个的 LCS 长度。若最后字符相等，则答案来自 `dp[i-1][j-1]+1`；否则丢掉任意一边最后字符取最大。
- 思路：双循环填表。
- 核心结构：`dp[i][j]`。
- 坑：子序列不是子串；相等时只从左上角 +1。
- 相似题：编辑距离、最长重复子数组。
- 记忆卡片：LCS 看最后字符：相等左上 +1，不等取上/左最大。

```python
class Solution:
    def longestCommonSubsequence(self, text1: str, text2: str) -> int:
        m, n = len(text1), len(text2)
        dp = [[0] * (n + 1) for _ in range(m + 1)]
        for i in range(1, m + 1):
            for j in range(1, n + 1):
                if text1[i - 1] == text2[j - 1]:
                    dp[i][j] = dp[i - 1][j - 1] + 1
                else:
                    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
        return dp[m][n]
```


#### 详细分析与小例子
子序列不要求连续，所以两个字符串最后字符不相等时，不是直接清零，而是尝试丢掉其中一个字符串的最后字符，看哪种更优。

例子：`"abcde"` 和 `"ace"`。`a`、`c`、`e` 可以匹配，中间的 `b`、`d` 可以跳过，所以最长公共子序列是 `"ace"`，长度 3。

和“最长公共子数组/子串”区别很大：子串要求连续，不相等时会断；子序列不连续，可以跳过字符。

### 33. 删除排序链表中的重复元素 II

- 类型：链表删除 + 哑节点。
- 分析：要删除所有重复值，只保留没有重复出现过的节点。因为头节点也可能被删，所以必须用 dummy。
- 思路：`pre` 指向结果尾；`cur` 扫描链表。若 `cur.next` 同值，记录该值并跳过所有同值节点；否则 `pre` 前进。
- 核心结构：`pre/cur`。
- 坑：重复值要全部删除，不是保留一个；处理头部重复要靠 dummy。
- 相似题：删除排序链表重复元素、移除元素。
- 记忆卡片：发现重复，整段同值全部跳过。

```python
class Solution:
    def deleteDuplicates(self, head):
        dummy = ListNode(0, head)
        pre = dummy
        cur = head
        while cur:
            if cur.next and cur.val == cur.next.val:
                val = cur.val
                while cur and cur.val == val:
                    cur = cur.next
                pre.next = cur
            else:
                pre = cur
                cur = cur.next
        return dummy.next
```


#### 详细分析与小例子
这题不是普通去重，而是“出现重复的值全部删除”。因为链表已排序，重复值一定连续，所以发现 `cur.val == cur.next.val` 后，可以整段跳过这个值。

例子：`1->2->3->3->4->4->5`。3 出现重复，两个 3 都删；4 也重复，两个 4 都删；最后剩 `1->2->5`。

头节点也可能属于重复段，比如 `1->1->2`，所以必须用 dummy 统一处理删除头节点的情况。

### 34. 复原 IP 地址

- 类型：回溯。
- 分析：IP 地址由 4 段组成，每段长度 1-3，数值 0-255。枚举每一段的切分位置即可。
- 思路：回溯参数为当前位置 `idx` 和已选段 `path`；段数为 4 且刚好用完字符串时加入答案；剪枝剩余长度必须能被剩余段数容纳。
- 核心结构：`idx/path`。
- 坑：前导零只能是单独 `"0"`；段值不能超过 255。
- 相似题：括号生成、组合总和、子集。
- 记忆卡片：IP 回溯只切 4 段，每段 1 到 3 位，零不能带头。

```python
class Solution:
    def restoreIpAddresses(self, s: str):
        ans, path = [], []

        def backtrack(idx):
            remain = 4 - len(path)
            if len(s) - idx < remain or len(s) - idx > remain * 3:
                return
            if len(path) == 4:
                if idx == len(s):
                    ans.append('.'.join(path))
                return
            for end in range(idx + 1, min(idx + 4, len(s) + 1)):
                part = s[idx:end]
                if len(part) > 1 and part[0] == '0':
                    break
                if int(part) <= 255:
                    path.append(part)
                    backtrack(end)
                    path.pop()

        backtrack(0)
        return ans
```


#### 详细分析与小例子
复原 IP 本质是在字符串中切出 4 段。每段长度只能是 1 到 3，数值必须在 0 到 255，因此适合回溯枚举每段切多长。

例子：`"25525511135"`。第一段可以试 `"2"`、`"25"`、`"255"`。如果第一段选 `"255"`，继续切第二段，最终能得到 `"255.255.11.135"` 和 `"255.255.111.35"`。

前导零是常见坑：`"0"` 合法，但 `"01"`、`"001"` 不合法。

### 35. 二叉树中的最大路径和

- 类型：树 DFS。
- 分析：路径可以从任意节点开始和结束，但向父节点只能贡献一条边方向的最大收益。当前节点作为拐点时，可以连接左右两边更新全局答案。
- 思路：`dfs(node)` 返回从 node 往下走的最大单边贡献；左右贡献小于 0 就丢弃；全局答案用 `node.val+left+right` 更新。
- 核心结构：递归返回单边贡献，全局 `ans` 记录拐点最大值。
- 坑：全负数不能返回 0 作为最终答案，`ans` 初始化负无穷。
- 相似题：二叉树直径、路径总和。
- 记忆卡片：给父亲只能交一条路，自己做答案可以吃左右两条路。

```python
class Solution:
    def maxPathSum(self, root) -> int:
        self.ans = float('-inf')

        def dfs(node):
            if not node:
                return 0
            left = max(dfs(node.left), 0)
            right = max(dfs(node.right), 0)
            self.ans = max(self.ans, node.val + left + right)
            return node.val + max(left, right)

        dfs(root)
        return self.ans
```


#### 详细分析与小例子
路径可以在某个节点处从左子树拐到右子树，但如果这条路径还要继续往父节点走，就不能同时带上左右两边，因为路径不能分叉。

例子：当前节点值是 10，左边最大贡献是 2，右边最大贡献是 10。以当前节点为最高拐点时，总路径和是 `2+10+10=22`；但返回给父节点时，只能返回 `10+max(2,10)=20`。

如果某边贡献是负数，接上它只会让路径变小，所以负贡献要当成 0 丢掉。

### 36. 删除链表的倒数第 N 个节点

- 类型：链表快慢指针。
- 分析：倒数第 n 个不好直接定位，可以让快指针先走 n 步，制造 n 的距离差。快指针到尾时，慢指针正好在待删节点前。
- 思路：用 dummy；`fast` 先走 n 步；然后 `fast/slow` 同步走到 `fast.next` 为空；删除 `slow.next`。
- 核心结构：距离差为 n 的快慢指针。
- 坑：删除头节点要用 dummy；循环停在待删节点前一个。
- 相似题：链表中倒数第 k 个节点、相交链表。
- 记忆卡片：倒数第 n，先让快指针领先 n。

```python
class Solution:
    def removeNthFromEnd(self, head, n: int):
        dummy = ListNode(0, head)
        fast = slow = dummy
        for _ in range(n):
            fast = fast.next
        while fast.next:
            fast = fast.next
            slow = slow.next
        slow.next = slow.next.next
        return dummy.next
```


#### 详细分析与小例子
倒数第 N 个节点可以通过快慢指针制造距离差。让 fast 先走 N 步，然后 slow 和 fast 一起走；当 fast 到尾部时，slow 正好在待删除节点前面。

例子：`1->2->3->4->5, n=2`。fast 先领先 2 步，然后一起走。fast 到 5 时，slow 在 3，删除 `slow.next`，也就是 4。

删除头节点是坑，所以用 dummy。比如删除 `1->2` 的倒数第 2 个，没有 dummy 会很别扭。

### 37. 寻找两个正序数组的中位数

- 类型：二分切分。
- 分析：目标是把两个数组切成左右两半，左半所有元素小于等于右半。只需要在较短数组上二分切分位置 `i`，另一个数组切分位置由总左半长度确定。
- 思路：设 `i` 为 A 左半个数，`j=(m+n+1)//2-i`；满足 `A[i-1]<=B[j]` 且 `B[j-1]<=A[i]` 时找到切分。
- 核心结构：两个数组的切分线。
- 坑：始终在短数组二分；边界用正负无穷处理。
- 相似题：第 K 小元素、二分答案。
- 记忆卡片：中位数就是找一刀，让左半最大小于右半最小。

```python
class Solution:
    def findMedianSortedArrays(self, nums1, nums2) -> float:
        A, B = nums1, nums2
        if len(A) > len(B):
            A, B = B, A
        m, n = len(A), len(B)
        l, r = 0, m
        half = (m + n + 1) // 2
        while l <= r:
            i = (l + r) // 2
            j = half - i
            Aleft = A[i - 1] if i > 0 else float('-inf')
            Aright = A[i] if i < m else float('inf')
            Bleft = B[j - 1] if j > 0 else float('-inf')
            Bright = B[j] if j < n else float('inf')
            if Aleft <= Bright and Bleft <= Aright:
                if (m + n) % 2:
                    return max(Aleft, Bleft)
                return (max(Aleft, Bleft) + min(Aright, Bright)) / 2
            if Aleft > Bright:
                r = i - 1
            else:
                l = i + 1
```


#### 详细分析与小例子
中位数的本质是把两个有序数组合起来后切成左右两半，左边数量等于右边或多一个，并且左边所有数都不大于右边所有数。我们不真的合并，而是在两个数组上各切一刀。

例子：`A=[1,3], B=[2]`。总长度 3，左半需要 2 个数。切成左边 `[1] + [2]`，右边 `[3]`，左半最大是 2，所以中位数是 2。

为了让二分更稳，只在短数组上切。边界位置没有数时，用正负无穷当哨兵处理。

### 38. 环形链表 II

- 类型：快慢指针找环入口。
- 分析：快慢指针相遇后，从头节点和相遇点各出发一个指针，每次走一步，会在环入口相遇。这来自距离公式：头到入口距离等于相遇点到入口的剩余环距离。
- 思路：先用快慢指针判断是否相遇；无环返回 None；有环则 `p=head`，`slow` 留在相遇点，同步前进找入口。
- 核心结构：快慢指针 + 二次相遇。
- 坑：先判断无环；返回入口节点，不是布尔值。
- 相似题：环形链表、寻找重复数。
- 记忆卡片：第一次相遇证明有环，第二次相遇就是入口。

```python
class Solution:
    def detectCycle(self, head):
        slow = fast = head
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
            if slow == fast:
                p = head
                while p != slow:
                    p = p.next
                    slow = slow.next
                return p
        return None
```


#### 详细分析与小例子
第一次快慢指针相遇只能说明有环。相遇后，把一个指针放回头节点，另一个留在相遇点，两者每次走一步，它们会在环入口相遇。

例子：`1->2->3->4->5`，5 指回 3。第一次相遇可能在 4 或 5。然后一个从 1 出发，一个从相遇点出发，同步走，最终会在 3 相遇。

面试时不用死背公式，可以说这是由快指针比慢指针多走的环长倍数推出的。

### 39. 比较版本号

- 类型：字符串模拟。
- 分析：版本号按点分段比较，每段按整数比较，缺失段视为 0。不能直接字符串比较，因为 `"10"` 大于 `"2"`。
- 思路：`split('.')` 后转整数；双指针遍历两个列表，缺失补 0；不同则返回 1 或 -1。
- 核心结构：分段数组。
- 坑：前导零要忽略；长度不同的尾部全 0 则相等。
- 相似题：字符串相加、atoi。
- 记忆卡片：版本号按段转整数比，缺段补 0。

```python
class Solution:
    def compareVersion(self, version1: str, version2: str) -> int:
        a = version1.split('.')
        b = version2.split('.')
        n = max(len(a), len(b))
        for i in range(n):
            x = int(a[i]) if i < len(a) else 0
            y = int(b[i]) if i < len(b) else 0
            if x < y:
                return -1
            if x > y:
                return 1
        return 0
```


#### 详细分析与小例子
版本号不能直接按字符串比较，因为每一段要按整数比较。字符串里 `"10"` 按字典序可能小于 `"2"`，但版本号里 10 明显大于 2。

例子：`"1.01"` 和 `"1.001"`。分段转整数后都是 `[1,1]`，所以相等。`"1.0"` 和 `"1.0.0"`，缺失段补 0，也相等。

核心就是按点切分、逐段转整数、短的版本后面补 0。

### 40. 二叉树的右视图

- 类型：树 BFS。
- 分析：右视图就是每一层最右边的节点。层序遍历时，每层最后弹出的节点就是右视图节点。
- 思路：BFS 分层；遍历当前层时，如果是该层最后一个节点，把值加入答案。
- 核心结构：队列分层。
- 坑：不是只沿右指针走，左子树深处也可能出现在右视图。
- 相似题：层序遍历、锯齿形遍历。
- 记忆卡片：右视图 = 每层最后一个被看到的节点。

```python
class Solution:
    def rightSideView(self, root):
        if not root:
            return []
        from collections import deque
        q = deque([root])
        ans = []
        while q:
            size = len(q)
            for i in range(size):
                node = q.popleft()
                if node.left:
                    q.append(node.left)
                if node.right:
                    q.append(node.right)
                if i == size - 1:
                    ans.append(node.val)
        return ans
```


#### 详细分析与小例子
右视图不是只沿着右孩子一直走，而是每一层从右边能看到的最后一个节点。有些层没有右孩子时，左子树的节点也可能被看到。

例子：根 1 只有左孩子 2，2 有右孩子 5。右视图是 `[1,2,5]`，如果只走右指针就会漏掉 2 和 5。

所以最稳的做法是 BFS 分层，每层取最后一个节点值。

### 41. 滑动窗口最大值

- 类型：单调队列。
- 分析：每个窗口都求最大值，如果每次扫描窗口是 O(nk)。单调队列保存下标，并保持对应值从大到小，队头就是当前窗口最大值。
- 思路：遍历下标 i；队尾小于等于当前值就弹出；加入 i；队头过期则弹出；窗口形成后记录 `nums[q[0]]`。
- 核心结构：递减队列保存下标。
- 坑：队列存下标才能判断过期；相等时弹出旧值可减少无用元素。
- 相似题：接雨水、每日温度、队列最大值。
- 记忆卡片：窗口最大值用单调队列，队头永远是最大。

```python
class Solution:
    def maxSlidingWindow(self, nums, k: int):
        from collections import deque
        q = deque()
        ans = []
        for i, x in enumerate(nums):
            while q and nums[q[-1]] <= x:
                q.pop()
            q.append(i)
            if q[0] <= i - k:
                q.popleft()
            if i >= k - 1:
                ans.append(nums[q[0]])
        return ans
```


#### 详细分析与小例子
每个窗口都求最大值，如果每次都扫描 k 个元素会很慢。单调队列保存下标，并让队列中下标对应的值从大到小排列，队头就是当前窗口最大值。

例子：`nums=[1,3,-1,-3,5], k=3`。窗口 `[1,3,-1]` 的最大值是 3。后来 5 进入窗口时，队列里比 5 小的元素以后都不可能成为最大值，可以从队尾弹掉。

队列里必须存下标，因为要判断队头是否已经滑出窗口。

### 42. 二分查找

- 类型：基础二分。
- 分析：数组有序，比较中点和 target 后可以排除一半区间。
- 思路：左闭右闭写法；`l<=r` 循环；命中返回下标；否则按大小移动边界。
- 核心结构：`l/r/mid`。
- 坑：`mid` 每轮重算；没找到返回 -1。
- 相似题：搜索插入位置、搜索旋转排序数组。
- 记忆卡片：二分每次排除一半，边界写法要统一。

```python
class Solution:
    def search(self, nums, target: int) -> int:
        l, r = 0, len(nums) - 1
        while l <= r:
            mid = (l + r) // 2
            if nums[mid] == target:
                return mid
            if nums[mid] < target:
                l = mid + 1
            else:
                r = mid - 1
        return -1
```


#### 详细分析与小例子
数组有序，所以看中点就能排除一半。如果 `nums[mid] < target`，中点左边都更小，不可能是答案；如果 `nums[mid] > target`，中点右边都更大，也不可能是答案。

例子：`[-1,0,3,5,9,12]` 找 9。中点 3 太小，丢左半；新的中点是 9，找到。

二分最重要的是统一边界写法。这里用左闭右闭，所以循环条件是 `l <= r`。

### 43. 括号生成

- 类型：回溯。
- 分析：合法括号序列的约束是：左括号数量不超过 n，右括号数量不能超过左括号数量。按这个约束生成即可。
- 思路：递归参数为当前字符串、左括号数、右括号数；可放左括号时放左；可放右括号时放右；长度到 `2n` 加入答案。
- 核心结构：`left/right/path`。
- 坑：右括号只能在 `right < left` 时放。
- 相似题：有效括号、最长有效括号、复原 IP。
- 记忆卡片：左括号别超 n，右括号别超过左括号。

```python
class Solution:
    def generateParenthesis(self, n: int):
        ans = []

        def backtrack(path, left, right):
            if len(path) == 2 * n:
                ans.append(path)
                return
            if left < n:
                backtrack(path + '(', left + 1, right)
            if right < left:
                backtrack(path + ')', left, right + 1)

        backtrack('', 0, 0)
        return ans
```


#### 详细分析与小例子
合法括号串的任意前缀中，右括号数量都不能超过左括号数量，否则前面已经无法匹配。回溯时只要遵守这个规则，就不会生成非法结果。

例子：`n=2`。一开始只能放 `(`。当当前串是 `"("` 时，可以继续放 `(` 得到 `"(("`，也可以放 `)` 得到 `"()"`。最终得到 `"(())"` 和 `"()()"`。

判断能不能放：左括号数量小于 n 才能放左括号；右括号数量小于左括号数量才能放右括号。

### 44. 最长有效括号

- 类型：栈 / 动态规划。
- 分析：栈保存未匹配的下标。遇到右括号时弹出一个左括号下标；弹完后若栈不空，当前有效长度是 `i-stack[-1]`；若栈空，当前右括号作为新的边界。
- 思路：栈初始化 `[-1]` 作为哨兵；遍历字符，左括号入栈，右括号弹栈并更新答案。
- 核心结构：栈保存边界下标。
- 坑：初始化 `-1` 很重要；栈空时要压入当前右括号下标。
- 相似题：有效括号、括号生成。
- 记忆卡片：有效括号长度 = 当前下标 - 上一个无法匹配的位置。

```python
class Solution:
    def longestValidParentheses(self, s: str) -> int:
        st = [-1]
        ans = 0
        for i, c in enumerate(s):
            if c == '(':
                st.append(i)
            else:
                st.pop()
                if not st:
                    st.append(i)
                else:
                    ans = max(ans, i - st[-1])
        return ans
```


#### 详细分析与小例子
这题要的是最长长度，所以栈里要存下标。栈保存尚未匹配的左括号下标，以及最近一个无法匹配的位置作为边界。

例子：`s=")()())"`。第一个 `)` 无法匹配，它的下标 0 成为边界。后面到下标 4 时，合法片段是 `"()()"`，长度是 `4-0=4`。

初始化栈为 `[-1]` 是为了处理从开头就合法的情况，比如 `"()"` 的长度是 `1 - (-1) = 2`。

### 45. 排序链表

- 类型：链表归并排序。
- 分析：链表适合归并排序，因为合并两个有序链表可以 O(1) 额外空间改指针。先用快慢指针找中点并断开，再递归排序左右两段，最后合并。
- 思路：找到中点前驱 `slow`；断链；递归排序 `head` 和 `mid`；合并两个有序链表。
- 核心结构：快慢指针切分 + 合并。
- 坑：断链 `slow.next=None`；递归终止是空或单节点。
- 相似题：合并两个有序链表、数组排序。
- 记忆卡片：链表排序优先归并：切两半，排两半，合起来。

```python
class Solution:
    def sortList(self, head):
        if not head or not head.next:
            return head

        slow, fast = head, head.next
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
        mid = slow.next
        slow.next = None

        left = self.sortList(head)
        right = self.sortList(mid)
        return self.merge(left, right)

    def merge(self, a, b):
        dummy = ListNode()
        cur = dummy
        while a and b:
            if a.val <= b.val:
                cur.next = a
                a = a.next
            else:
                cur.next = b
                b = b.next
            cur = cur.next
        cur.next = a or b
        return dummy.next
```


#### 详细分析与小例子
链表不适合按下标随机访问，所以数组里的快排思路不太适合链表。链表更适合归并排序：快慢指针切成两半，分别排好，再合并两个有序链表。

例子：`4->2->1->3`。先切成 `4->2` 和 `1->3`，分别排序为 `2->4` 和 `1->3`，最后合并成 `1->2->3->4`。

切开链表时一定要断链，否则递归时左右两段没有真正分离，可能无限递归。

### 46. x 的平方根

- 类型：二分答案。
- 分析：要求整数平方根，即最大 `mid` 满足 `mid*mid <= x`。满足条件具有单调性，所以二分。
- 思路：在 `[0,x]` 中二分；若 `mid*mid <= x`，记录答案并向右找；否则向左。
- 核心结构：答案型二分。
- 坑：返回向下取整；`x=0/1` 也能被统一处理。
- 相似题：有效完全平方数、寻找峰值。
- 记忆卡片：找最大合法值，用二分保留 ans。

```python
class Solution:
    def mySqrt(self, x: int) -> int:
        l, r = 0, x
        ans = 0
        while l <= r:
            mid = (l + r) // 2
            if mid * mid <= x:
                ans = mid
                l = mid + 1
            else:
                r = mid - 1
        return ans
```


#### 详细分析与小例子
整数平方根是向下取整，也就是找最大的整数 a，使得 `a*a <= x`。这个条件从小到大先满足，超过某个点后不满足，天然适合二分。

例子：`x=8`。`2*2=4` 满足，`3*3=9` 不满足，所以答案是 2。

这类题叫“找最大合法值”。当 mid 合法时，记录答案，并继续往右试试有没有更大的合法值。

### 47. 二叉树的中序遍历

- 类型：树 DFS / 栈。
- 分析：中序遍历顺序是左、根、右。递归最直观；面试也常要求迭代写法，用栈模拟递归一路向左。
- 思路：当前节点不断压栈并左移；左边走到底后弹出访问，再转向右子树。
- 核心结构：栈和当前指针。
- 坑：循环条件是 `cur or st`。
- 相似题：前序遍历、验证 BST。
- 记忆卡片：中序 = 左根右；迭代就是一路压左。

```python
class Solution:
    def inorderTraversal(self, root):
        ans, st = [], []
        cur = root
        while cur or st:
            while cur:
                st.append(cur)
                cur = cur.left
            cur = st.pop()
            ans.append(cur.val)
            cur = cur.right
        return ans
```


#### 详细分析与小例子
中序遍历顺序是左、根、右。递归很好写，迭代版就是用栈模拟“先一路走到最左边”的过程。

例子：树为 2，左孩子 1，右孩子 3。先把 2 和 1 依次压栈；左边到底后弹出 1 访问，再弹出 2 访问，然后转到右孩子 3。

外层循环条件必须是当前节点不为空，或者栈里还有节点。否则右子树可能还没处理就结束了。

### 48. 用栈实现队列

- 类型：数据结构设计。
- 分析：队列先进先出，栈后进先出。用两个栈：`in_stack` 负责入队，`out_stack` 负责出队。当 `out_stack` 空时，把 `in_stack` 全部倒过去，顺序就反过来了。
- 思路：`push` 入 `in_stack`；`pop/peek` 前先调用 `_move` 保证 `out_stack` 有元素。
- 核心结构：输入栈、输出栈。
- 坑：只有 `out_stack` 为空时才倒，不能每次倒。
- 相似题：用队列实现栈、最小栈。
- 记忆卡片：入栈存新货，出栈倒一次，倒完就是队列顺序。

```python
class MyQueue:
    def __init__(self):
        self.in_stack = []
        self.out_stack = []

    def push(self, x: int) -> None:
        self.in_stack.append(x)

    def _move(self):
        if not self.out_stack:
            while self.in_stack:
                self.out_stack.append(self.in_stack.pop())

    def pop(self) -> int:
        self._move()
        return self.out_stack.pop()

    def peek(self) -> int:
        self._move()
        return self.out_stack[-1]

    def empty(self) -> bool:
        return not self.in_stack and not self.out_stack
```


#### 详细分析与小例子
一个栈会把顺序反过来，两个栈反两次就能恢复队列顺序。入队全部进入 `in_stack`；出队时如果 `out_stack` 为空，就把 `in_stack` 全倒过去。

例子：push 1、2、3 后，`in_stack=[1,2,3]`。第一次 pop 前倒到 `out_stack=[3,2,1]`，弹出的是 1，符合队列先进先出。

只有 `out_stack` 空时才倒。如果每次 pop 都倒，会破坏已经排好的出队顺序。

### 49. 下一个排列

- 类型：数组模拟 / 字典序。
- 分析：下一个排列要尽量小地变大。从右往左找第一个下降位置 `i`，即 `nums[i] < nums[i+1]`，说明后缀是降序的。再从右找第一个大于 `nums[i]` 的数交换，最后把后缀反转成升序。
- 思路：找 `i`；若存在，找 `j` 交换；反转 `i+1` 到末尾。
- 核心结构：下降点和后缀反转。
- 坑：完全降序时直接反转成最小排列。
- 相似题：全排列、最大数。
- 记忆卡片：从右找破坏降序的位置，换一个刚大的，再把后缀变最小。

```python
class Solution:
    def nextPermutation(self, nums) -> None:
        i = len(nums) - 2
        while i >= 0 and nums[i] >= nums[i + 1]:
            i -= 1
        if i >= 0:
            j = len(nums) - 1
            while nums[j] <= nums[i]:
                j -= 1
            nums[i], nums[j] = nums[j], nums[i]
        l, r = i + 1, len(nums) - 1
        while l < r:
            nums[l], nums[r] = nums[r], nums[l]
            l += 1
            r -= 1
```


#### 详细分析与小例子
下一个排列要求“刚好比当前排列大一点”。所以应该尽量从右边改，因为右边位权小。从右往左找第一个 `nums[i] < nums[i+1]` 的位置，它右侧是降序，已经是最大后缀。

例子：`[1,2,3]`。从右看，2 < 3，所以把 2 换成右侧刚好比它大的 3，再把后缀反转，得到 `[1,3,2]`。

如果整个数组是 `[3,2,1]`，说明已经是最大排列，下一个排列就是最小的 `[1,2,3]`。

### 50. 最小覆盖子串

- 类型：滑动窗口。
- 分析：要找包含 t 所有字符的最短子串。右指针扩张直到窗口满足条件，然后左指针尽量收缩，收缩过程中更新最短答案。
- 思路：`need` 统计 t；`window` 统计窗口；`valid` 表示有多少字符满足数量要求；当 `valid==len(need)` 时收缩。
- 核心结构：`need/window/valid/left/right`。
- 坑：字符有重复，不能只看是否存在；更新 `valid` 的时机要在数量刚好达到或刚好跌破时。
- 相似题：无重复字符最长子串、字符串排列。
- 记忆卡片：覆盖类窗口：先扩到满足，再缩到不能再缩。

```python
class Solution:
    def minWindow(self, s: str, t: str) -> str:
        from collections import Counter, defaultdict
        need = Counter(t)
        window = defaultdict(int)
        left = valid = 0
        start, length = 0, float('inf')
        for right, c in enumerate(s):
            if c in need:
                window[c] += 1
                if window[c] == need[c]:
                    valid += 1
            while valid == len(need):
                if right - left + 1 < length:
                    start, length = left, right - left + 1
                d = s[left]
                left += 1
                if d in need:
                    if window[d] == need[d]:
                        valid -= 1
                    window[d] -= 1
        return '' if length == float('inf') else s[start:start + length]
```


#### 详细分析与小例子
最小覆盖子串要求窗口包含 t 的所有字符。右指针先扩张，直到窗口满足条件；一旦满足，就移动左指针尽量缩短窗口，缩短过程中更新答案。

例子：`s="ADOBECODEBANC", t="ABC"`。窗口扩到 `"ADOBEC"` 时第一次包含 A、B、C，记录长度 6。后面继续滑动，最终找到更短的 `"BANC"`。

字符可能重复，比如 t 是 `"AABC"` 时需要两个 A，所以必须统计每个字符数量，而不是只看是否出现。

### 51. 两数相加

- 类型：链表模拟 / 大数加法。
- 分析：链表逆序存储数字，正好从低位开始加。逐位相加并维护进位，生成新链表。
- 思路：遍历 `l1/l2`，当前位值为空则 0；`sum = a+b+carry`；新建个位节点；更新进位。
- 核心结构：链表指针和 `carry`。
- 坑：最后 carry 可能产生新节点；两条链长度不同。
- 相似题：字符串相加、字符串相乘。
- 记忆卡片：逆序链表加法就是小学竖式，从头加到尾。

```python
class Solution:
    def addTwoNumbers(self, l1, l2):
        dummy = ListNode()
        cur = dummy
        carry = 0
        while l1 or l2 or carry:
            a = l1.val if l1 else 0
            b = l2.val if l2 else 0
            s = a + b + carry
            cur.next = ListNode(s % 10)
            carry = s // 10
            cur = cur.next
            l1 = l1.next if l1 else None
            l2 = l2.next if l2 else None
        return dummy.next
```


#### 详细分析与小例子
链表里的数字是逆序存储的，所以头节点就是个位，正好可以从头开始模拟竖式加法。每一位相加后产生个位数字和进位。

例子：`342 + 465` 在链表里是 `2->4->3` 和 `5->6->4`。先算 `2+5=7`，再算 `4+6=10` 写 0 进 1，再算 `3+4+1=8`，结果链表是 `7->0->8`。

两条链表长度可能不同，短的那边当作 0。最后如果还有进位，要新建一个节点。

### 52. 零钱兑换

- 类型：完全背包动态规划。
- 分析：每个硬币可以无限使用，要求凑出金额的最少硬币数。`dp[x]` 表示凑出金额 x 的最少硬币数。
- 思路：初始化 `dp[0]=0`，其他为无穷；对每个金额 x，枚举 coin，若 `x>=coin`，用 `dp[x-coin]+1` 更新。
- 核心结构：一维 DP。
- 坑：不可达返回 -1；初始化不能用 0。
- 相似题：组合总和 IV、完全平方数、单词拆分。
- 记忆卡片：最少硬币 = 当前金额减一个硬币后的最优 +1。

```python
class Solution:
    def coinChange(self, coins, amount: int) -> int:
        dp = [float('inf')] * (amount + 1)
        dp[0] = 0
        for x in range(1, amount + 1):
            for c in coins:
                if x >= c:
                    dp[x] = min(dp[x], dp[x - c] + 1)
        return -1 if dp[amount] == float('inf') else dp[amount]
```


#### 详细分析与小例子
每种硬币可以无限次使用，目标是凑出 amount 的最少硬币数。`dp[x]` 表示凑出金额 x 最少需要多少枚硬币。最后一步一定是用了某个 coin，所以来源是 `dp[x-coin]+1`。

例子：coins=`[1,2,5]`, amount=11。凑 11 可以看成先凑 6 再加 5，或者先凑 9 再加 2，或者先凑 10 再加 1。最优是 `5+5+1`，答案 3。

不可达要返回 -1，所以初始值要设成无穷大，而不是 0。

### 53. 字符串相乘

- 类型：字符串模拟 / 大数乘法。
- 分析：不能直接转整数。按乘法竖式，`num1[i] * num2[j]` 的结果会贡献到数组位置 `i+j+1`，进位到 `i+j`。
- 思路：创建长度 `m+n` 的数组；倒序双循环相乘并累加；最后去掉前导零。
- 核心结构：结果数组 `res`。
- 坑：任一数为 `"0"` 直接返回 `"0"`；处理前导零。
- 相似题：字符串相加、两数相加。
- 记忆卡片：乘法位置记公式：个位落在 `i+j+1`。

```python
class Solution:
    def multiply(self, num1: str, num2: str) -> str:
        if num1 == '0' or num2 == '0':
            return '0'
        m, n = len(num1), len(num2)
        res = [0] * (m + n)
        for i in range(m - 1, -1, -1):
            for j in range(n - 1, -1, -1):
                mul = (ord(num1[i]) - 48) * (ord(num2[j]) - 48)
                s = mul + res[i + j + 1]
                res[i + j + 1] = s % 10
                res[i + j] += s // 10
        return ''.join(map(str, res)).lstrip('0')
```


#### 详细分析与小例子
字符串乘法就是模拟竖式乘法。`num1[i]` 和 `num2[j]` 相乘后，个位会落在结果数组的 `i+j+1`，进位落在 `i+j`。

例子：`"12" * "34"`。2*4 贡献到最后一位，2*3 和 1*4 贡献到中间位置，1*3 贡献到前面位置。最后整理进位得到 `"408"`。

结果数组长度最多是 `m+n`。最后要去掉前导零；如果任一输入是 `"0"`，直接返回 `"0"`。

### 54. 字符串转换整数 atoi

- 类型：字符串模拟。
- 分析：按规则解析：跳过空格，读取符号，再读连续数字，遇到其他字符停止，最后按 32 位范围截断。
- 思路：指针 `i` 跳空格；判断正负号；循环累积数字；每步或最后做边界截断。
- 核心结构：扫描指针、符号、结果。
- 坑：符号后必须是数字才继续；范围是 `[-2^31, 2^31-1]`。
- 相似题：比较版本号、有效数字。
- 记忆卡片：atoi 四步：空格、符号、数字、截断。

```python
class Solution:
    def myAtoi(self, s: str) -> int:
        i, n = 0, len(s)
        while i < n and s[i] == ' ':
            i += 1
        sign = 1
        if i < n and s[i] in '+-':
            sign = -1 if s[i] == '-' else 1
            i += 1
        ans = 0
        while i < n and s[i].isdigit():
            ans = ans * 10 + ord(s[i]) - 48
            i += 1
        ans *= sign
        return max(-2**31, min(2**31 - 1, ans))
```


#### 详细分析与小例子
atoi 是规则模拟题，按顺序处理：先跳过前导空格，再看正负号，然后读取连续数字，遇到非数字停止，最后做范围截断。

例子：`"   -42abc"`。跳过空格，读到负号，继续读数字 42，遇到 a 停止，结果是 -42。

坑点是边界：结果必须限制在 32 位有符号整数范围内，也就是 `[-2147483648, 2147483647]`。

### 55. 爬楼梯

- 类型：动态规划 / 斐波那契。
- 分析：到第 n 阶的最后一步只能来自第 n-1 阶走 1 步，或第 n-2 阶走 2 步。
- 思路：`dp[n]=dp[n-1]+dp[n-2]`；用两个变量滚动即可。
- 核心结构：前两项状态。
- 坑：`n=1` 返回 1；`n=2` 返回 2。
- 相似题：打家劫舍、不同路径。
- 记忆卡片：最后一步拆来源：从前一阶来，或从前两阶来。

```python
class Solution:
    def climbStairs(self, n: int) -> int:
        a, b = 1, 1
        for _ in range(n):
            a, b = b, a + b
        return a
```


#### 详细分析与小例子
到第 n 阶的最后一步只有两种可能：从第 n-1 阶走 1 步上来，或者从第 n-2 阶走 2 步上来。所以方法数等于前两项之和。

例子：n=3。走法有 `1+1+1`、`1+2`、`2+1`，共 3 种。它等于到第 2 阶的 2 种，加上到第 1 阶的 1 种。

这就是斐波那契型 DP，用两个变量滚动保存前两项即可。

### 56. 缺失的第一个正数

- 类型：原地哈希。
- 分析：答案一定在 `[1,n+1]`。把每个合法数字 x 放到下标 `x-1` 的位置上，最后第一个位置不匹配的下标 i，对应答案就是 `i+1`。
- 思路：遍历数组，当前数在 `[1,n]` 且不在正确位置时，不断交换到正确位置；再扫描找第一个 `nums[i]!=i+1`。
- 核心结构：数组自身作为哈希表。
- 坑：交换条件要避免死循环，尤其重复数字；忽略非正数和大于 n 的数。
- 相似题：寻找重复数、数组中重复的数字。
- 记忆卡片：数字 x 应该回到坑位 x-1。

```python
class Solution:
    def firstMissingPositive(self, nums) -> int:
        n = len(nums)
        for i in range(n):
            while 1 <= nums[i] <= n and nums[nums[i] - 1] != nums[i]:
                j = nums[i] - 1
                nums[i], nums[j] = nums[j], nums[i]
        for i, x in enumerate(nums):
            if x != i + 1:
                return i + 1
        return n + 1
```


#### 详细分析与小例子
长度为 n 的数组，缺失的第一个正数一定在 `1` 到 `n+1` 之间。我们希望数字 x 放在下标 `x-1` 的位置上，数组本身就变成一个哈希表。

例子：`[3,4,-1,1]`。3 应该去下标 2，1 应该去下标 0。调整后，扫描发现下标 1 上不是 2，所以答案是 2。

交换时要防止死循环：如果目标位置已经是同样的数，说明有重复，不能继续换。

### 57. 从前序与中序遍历序列构造二叉树

- 类型：树递归。
- 分析：前序第一个元素是根；中序中根左边是左子树，右边是右子树。用哈希表快速找到根在中序的位置。
- 思路：递归函数接收前序区间和中序区间；建立根；计算左子树大小；递归构造左右子树。
- 核心结构：区间递归 + 中序位置表。
- 坑：区间边界容易错；不要在递归中反复 `index`。
- 相似题：中序后序构造树、二叉树序列化。
- 记忆卡片：前序拿根，中序切左右。

```python
class Solution:
    def buildTree(self, preorder, inorder):
        pos = {v: i for i, v in enumerate(inorder)}

        def build(pl, pr, il, ir):
            if pl > pr:
                return None
            root_val = preorder[pl]
            root = TreeNode(root_val)
            k = pos[root_val]
            left_size = k - il
            root.left = build(pl + 1, pl + left_size, il, k - 1)
            root.right = build(pl + left_size + 1, pr, k + 1, ir)
            return root

        return build(0, len(preorder) - 1, 0, len(inorder) - 1)
```


#### 详细分析与小例子
前序遍历第一个元素一定是根节点。中序遍历中，根节点左边全是左子树，右边全是右子树。利用这个规律可以递归构造整棵树。

例子：preorder=`[3,9,20,15,7]`，inorder=`[9,3,15,20,7]`。前序第一个 3 是根；中序里 3 左边 `[9]` 是左子树，右边 `[15,20,7]` 是右子树。

为了快速找到根在中序中的位置，要提前用哈希表记录值到下标的映射。

### 58. 子集

- 类型：回溯。
- 分析：每个元素都有选或不选两种可能。也可以看作组合枚举：从当前位置开始，每加入一个元素就形成一个新子集。
- 思路：递归一开始先把当前 `path` 加入答案；然后从 `start` 到末尾枚举下一个选择。
- 核心结构：`start/path`。
- 坑：每个中间状态都是答案；加入时要拷贝。
- 相似题：全排列、组合总和。
- 记忆卡片：子集题，每个节点都是答案。

```python
class Solution:
    def subsets(self, nums):
        ans, path = [], []

        def backtrack(start):
            ans.append(path[:])
            for i in range(start, len(nums)):
                path.append(nums[i])
                backtrack(i + 1)
                path.pop()

        backtrack(0)
        return ans
```


#### 详细分析与小例子
子集就是每个元素选或不选。回溯树上的每个节点都是一个合法子集，所以每次进入递归时都可以先把当前 path 放入答案。

例子：`[1,2]`。一开始 path 是 `[]`；选 1 得到 `[1]`；在 `[1]` 基础上选 2 得到 `[1,2]`；回退后单独选 2 得到 `[2]`。答案是 `[],[1],[1,2],[2]`。

因为子集不关心顺序，所以用 `start` 保证后面只选当前位置之后的元素，避免重复。

### 59. 翻转字符串里的单词

- 类型：字符串处理。
- 分析：题目要去掉多余空格并反转单词顺序。Python 中 `split()` 默认按任意空白切分并去除多余空格，正好符合要求。
- 思路：`s.split()` 得到单词数组；反转后用单空格连接。
- 核心结构：单词列表。
- 坑：多个空格、前后空格都要去掉；不是反转每个单词字符。
- 相似题：左旋转字符串、反转字符串。
- 记忆卡片：反转单词顺序 = split 清洗 + reverse + join。

```python
class Solution:
    def reverseWords(self, s: str) -> str:
        return ' '.join(reversed(s.split()))
```


#### 详细分析与小例子
题目要反转单词顺序，并去掉多余空格。单词内部字符不需要反转。最简单的思路是先把字符串按空白切成单词，再反转单词列表。

例子：`"  hello   world  "`。切分后得到 `['hello','world']`，反转后是 `['world','hello']`，再用一个空格连接，得到 `"world hello"`。

Python 的 `split()` 默认会处理多个空格和首尾空格，所以这题可以写得很短。

### 60. 在排序数组中查找元素的第一个和最后一个位置

- 类型：二分边界。
- 分析：排序数组中找范围，本质是找左边界和右边界。右边界可以找“第一个大于 target 的位置”再减 1。
- 思路：写 `lower_bound(x)` 返回第一个 `>=x` 的位置；左边界是 `lower_bound(target)`，右边界是 `lower_bound(target+1)-1`。
- 核心结构：左边界二分。
- 坑：target 不存在时返回 `[-1,-1]`；空数组也要处理。
- 相似题：二分查找、搜索插入位置。
- 记忆卡片：范围 = 第一个 >= target，到第一个 > target 前一位。

```python
class Solution:
    def searchRange(self, nums, target: int):
        def lower_bound(x):
            l, r = 0, len(nums)
            while l < r:
                mid = (l + r) // 2
                if nums[mid] >= x:
                    r = mid
                else:
                    l = mid + 1
            return l

        left = lower_bound(target)
        if left == len(nums) or nums[left] != target:
            return [-1, -1]
        right = lower_bound(target + 1) - 1
        return [left, right]
```


#### 详细分析与小例子
排序数组里找某个值的范围，本质是找两个边界：第一个大于等于 target 的位置，以及第一个大于 target 的位置。右边界就是第二个位置减 1。

例子：`nums=[5,7,7,8,8,10], target=8`。第一个大于等于 8 的位置是 3；第一个大于 8 的位置是 5；所以范围是 `[3,4]`。

如果第一个大于等于 target 的位置越界，或者那个位置不是 target，说明 target 不存在。

### 61. 链表中倒数第 k 个节点

- 类型：链表快慢指针。
- 分析：倒数第 k 个节点可以通过距离差定位。让 fast 先走 k 步，然后 slow 和 fast 同步走，fast 到空时 slow 就在倒数第 k 个。
- 思路：`fast=head` 先走 k；`slow=head`；两者同步走到 fast 为空。
- 核心结构：领先 k 步的双指针。
- 坑：本题返回节点，不删除节点；k 通常保证合法。
- 相似题：删除链表倒数第 N 个节点。
- 记忆卡片：找倒数第 k，就让快指针先走 k。

```python
class Solution:
    def getKthFromEnd(self, head, k: int):
        fast = slow = head
        for _ in range(k):
            fast = fast.next
        while fast:
            fast = fast.next
            slow = slow.next
        return slow
```


#### 详细分析与小例子
倒数第 k 个节点可以用快慢指针定位。让 fast 先走 k 步，然后 slow 和 fast 同时走；fast 走到空时，slow 就刚好在倒数第 k 个节点。

例子：`1->2->3->4->5, k=2`。fast 先走到 3，slow 在 1。两个一起走，fast 到空时，slow 在 4，4 就是倒数第 2 个节点。

这题只是返回节点，不需要删除，所以不一定需要 dummy。

### 62. 字符串解码

- 类型：栈 / 字符串模拟。
- 分析：遇到嵌套括号时，天然用栈保存之前的字符串和重复次数。遇到 `[` 时把当前上下文入栈，开始新片段；遇到 `]` 时弹出上下文并拼接。
- 思路：扫描字符；数字累积成 `num`；字母加入 `cur`；`[` 入栈 `(cur,num)` 并重置；`]` 弹出并组合。
- 核心结构：栈保存外层上下文。
- 坑：数字可能多位；括号可嵌套。
- 相似题：有效括号、基本计算器。
- 记忆卡片：左括号保存现场，右括号恢复现场并重复当前串。

```python
class Solution:
    def decodeString(self, s: str) -> str:
        st = []
        cur = ''
        num = 0
        for c in s:
            if c.isdigit():
                num = num * 10 + int(c)
            elif c == '[':
                st.append((cur, num))
                cur, num = '', 0
            elif c == ']':
                prev, k = st.pop()
                cur = prev + cur * k
            else:
                cur += c
        return cur
```


#### 详细分析与小例子
字符串解码有嵌套结构，遇到 `[` 时要保存外层已经构造好的字符串和重复次数，等遇到 `]` 时再恢复外层并拼接。栈正好用来保存这种“现场”。

例子：`"3[a2[c]]"`。先看到 `3[`，保存重复 3；里面是 `a2[c]`，`2[c]` 解成 `cc`，所以里面变成 `acc`；最后重复 3 次得到 `accaccacc`。

数字可能是多位数，比如 `12[a]`，所以读数字时要累积，而不是只读一个字符。

### 63. 求根到叶子节点数字之和

- 类型：树 DFS。
- 分析：从根到叶子形成一个数字，每往下一层，当前值就乘 10 再加节点值。到叶子时返回这条路径的数字。
- 思路：DFS 参数带当前数字 `cur`；更新为 `cur*10+node.val`；叶子返回 cur；非叶子返回左右子树之和。
- 核心结构：路径累积值。
- 坑：只有叶子节点才能结算；空节点返回 0。
- 相似题：路径总和、路径总和 II。
- 记忆卡片：树上数字题，向下一层就是 `cur*10+val`。

```python
class Solution:
    def sumNumbers(self, root) -> int:
        def dfs(node, cur):
            if not node:
                return 0
            cur = cur * 10 + node.val
            if not node.left and not node.right:
                return cur
            return dfs(node.left, cur) + dfs(node.right, cur)
        return dfs(root, 0)
```


#### 详细分析与小例子
从根到叶子每条路径代表一个数字。往下一层时，原来的数字要左移一位，也就是乘 10，再加上当前节点值。

例子：路径 `1 -> 2 -> 3` 表示数字 123。计算过程是 `0*10+1=1`，`1*10+2=12`，`12*10+3=123`。

只有走到叶子节点时才能把当前数字加入答案；中间节点不能提前结算。

### 64. 最小栈

- 类型：数据结构设计。
- 分析：普通栈能 O(1) push/pop/top，但不能 O(1) 得到最小值。额外维护一个辅助栈，栈顶始终是当前最小值。
- 思路：主栈正常存值；辅助栈在新值小于等于当前最小值时入栈；pop 时如果弹出的是当前最小值，辅助栈也弹。
- 核心结构：主栈 + 最小值栈。
- 坑：相同最小值也要入辅助栈，否则弹出一个后最小值会错。
- 相似题：用栈实现队列、队列最大值。
- 记忆卡片：最小栈用影子栈，记录每一层的最小值。

```python
class MinStack:
    def __init__(self):
        self.st = []
        self.min_st = []

    def push(self, val: int) -> None:
        self.st.append(val)
        if not self.min_st or val <= self.min_st[-1]:
            self.min_st.append(val)

    def pop(self) -> None:
        x = self.st.pop()
        if x == self.min_st[-1]:
            self.min_st.pop()

    def top(self) -> int:
        return self.st[-1]

    def getMin(self) -> int:
        return self.min_st[-1]
```


#### 详细分析与小例子
普通栈能快速得到栈顶，但不能快速知道最小值。辅助栈保存“当前层级下的最小值”，这样 `getMin` 直接看辅助栈栈顶。

例子：依次 push `3, 5, 2, 2`。最小栈会记录 `3,2,2`。弹出一个 2 后，最小值仍然是 2；再弹出一个 2 后，最小值回到 3。

相同最小值也要入最小栈，否则弹出其中一个后，最小值会被错误更新。

### 65. 组合总和

- 类型：回溯。
- 分析：每个数字可以重复使用，所以递归下一层仍然可以从当前下标开始，而不是 `i+1`。先排序可以在当前和超过 target 时剪枝。
- 思路：`backtrack(start, remain)`；若 remain 为 0 收集答案；枚举 `i` 从 start 开始，选择 candidates[i] 后递归 `i`。
- 核心结构：`start/remain/path`。
- 坑：可重复使用是递归 `i`，不是 `i+1`；超过 target 直接 break。
- 相似题：子集、全排列、零钱兑换。
- 记忆卡片：组合可复用，下一层还从 i 开始。

```python
class Solution:
    def combinationSum(self, candidates, target: int):
        candidates.sort()
        ans, path = [], []

        def backtrack(start, remain):
            if remain == 0:
                ans.append(path[:])
                return
            for i in range(start, len(candidates)):
                x = candidates[i]
                if x > remain:
                    break
                path.append(x)
                backtrack(i, remain - x)
                path.pop()

        backtrack(0, target)
        return ans
```


#### 详细分析与小例子
组合总和里每个数字可以重复使用，所以选择了 `candidates[i]` 后，下一层仍然可以从 i 开始，而不是从 i+1 开始。

例子：`candidates=[2,3,6,7], target=7`。可以选 `2` 后继续选 `2`，得到路径 `2,2,3`；也可以直接选 `7`。所以答案有 `[2,2,3]` 和 `[7]`。

排序后，如果当前数字已经大于剩余目标，就可以 break，后面更大的数字也不用试。

### 66. 对称二叉树

- 类型：树 DFS。
- 分析：判断整棵树是否镜像，本质是比较左子树和右子树是否互为镜像。镜像条件：两个节点值相等，左的左等于右的右，左的右等于右的左。
- 思路：写 `same(a,b)` 比较两棵树是否镜像。
- 核心结构：双节点递归。
- 坑：一个为空一个不为空时 false；不是比较左右子树是否完全相同，而是镜像。
- 相似题：翻转二叉树、相同的树。
- 记忆卡片：对称比较交叉：外侧对外侧，内侧对内侧。

```python
class Solution:
    def isSymmetric(self, root) -> bool:
        def mirror(a, b):
            if not a or not b:
                return a == b
            return a.val == b.val and mirror(a.left, b.right) and mirror(a.right, b.left)
        return mirror(root.left, root.right) if root else True
```


#### 详细分析与小例子
对称二叉树不是左右子树完全相同，而是互为镜像。镜像意味着左边的左孩子要对应右边的右孩子，左边的右孩子要对应右边的左孩子。

例子：根节点两边都是 2，左子树是 `3,4`，右子树必须是 `4,3` 才对称。

递归时每次比较两个节点：值要相等，外侧要相等，内侧也要相等。

### 67. 最小路径和

- 类型：网格动态规划。
- 分析：只能向右或向下，走到 `(i,j)` 的上一步只能来自上方或左方。`dp[i][j]` 是到该格子的最小路径和。
- 思路：可原地修改 grid；第一行只能从左来，第一列只能从上来；其余取上和左最小加当前值。
- 核心结构：二维 DP。
- 坑：第一行第一列要单独初始化。
- 相似题：不同路径、最大正方形。
- 记忆卡片：网格路径 DP，看上面和左边。

```python
class Solution:
    def minPathSum(self, grid):
        m, n = len(grid), len(grid[0])
        for i in range(m):
            for j in range(n):
                if i == 0 and j == 0:
                    continue
                up = grid[i - 1][j] if i > 0 else float('inf')
                left = grid[i][j - 1] if j > 0 else float('inf')
                grid[i][j] += min(up, left)
        return grid[-1][-1]
```


#### 详细分析与小例子
只能向右或向下，所以走到某个格子 `(i,j)` 的上一步只能来自上方 `(i-1,j)` 或左方 `(i,j-1)`。要路径和最小，就取这两者里较小的。

例子：网格 `[[1,3,1],[1,5,1],[4,2,1]]`。到右下角的最小路径是 `1->3->1->1->1`，和为 7。

第一行只能从左边走来，第一列只能从上边走来，所以边界要单独处理或用无穷大简化。

### 68. 用 Rand7() 实现 Rand10()

- 类型：概率 / 拒绝采样。
- 分析：`rand7()` 能生成 1-7。两次调用可等概率生成 1-49。取其中 1-40，因为 40 能被 10 整除，再映射到 1-10；如果落在 41-49 就重采样。
- 思路：`x=(rand7()-1)*7+rand7()`；若 `x<=40` 返回 `(x-1)%10+1`。
- 核心结构：拒绝采样保证均匀。
- 坑：不能用 `rand7()+rand7()`，分布不均匀；拒绝区间必须丢弃。
- 相似题：随机数生成、蓄水池抽样。
- 记忆卡片：先造等概率大区间，再丢掉不能整除的尾巴。

```python
class Solution:
    def rand10(self):
        while True:
            x = (rand7() - 1) * 7 + rand7()
            if x <= 40:
                return (x - 1) % 10 + 1
```


#### 详细分析与小例子
`rand7()` 只能均匀生成 1 到 7。调用两次可以均匀生成 1 到 49。为了得到 1 到 10 的均匀分布，只取 1 到 40，因为 40 能被 10 整除；41 到 49 丢弃重来。

例子：如果生成的 x 是 1 到 40，就用 `(x-1)%10+1` 映射到 1 到 10。每个结果都对应 4 个 x，所以概率相同。

不能用 `rand7()+rand7()`，因为和为 2 的情况只有一种，和为 8 的情况有多种，分布不均匀。

### 69. 岛屿的最大面积

- 类型：网格 DFS。
- 分析：和岛屿数量类似，只是每遇到一个岛屿时计算它的面积，并更新最大值。
- 思路：遍历网格；遇到 1 调用 DFS，DFS 返回当前连通块面积；访问过改成 0。
- 核心结构：DFS 返回面积。
- 坑：网格值是整数 0/1；访问后标记防止重复。
- 相似题：岛屿数量、被围绕区域。
- 记忆卡片：岛屿面积 = DFS 每踩一格贡献 1。

```python
class Solution:
    def maxAreaOfIsland(self, grid):
        m, n = len(grid), len(grid[0])

        def dfs(i, j):
            if i < 0 or i >= m or j < 0 or j >= n or grid[i][j] == 0:
                return 0
            grid[i][j] = 0
            return 1 + dfs(i + 1, j) + dfs(i - 1, j) + dfs(i, j + 1) + dfs(i, j - 1)

        ans = 0
        for i in range(m):
            for j in range(n):
                if grid[i][j] == 1:
                    ans = max(ans, dfs(i, j))
        return ans
```


#### 详细分析与小例子
这题和岛屿数量类似，只是遇到一座岛时不只是计数，而是要算这座岛有多少格。DFS 每访问一个陆地格子，就贡献面积 1。

例子：某个岛由 5 个相连的 1 组成。第一次扫到它时，DFS 会递归走完整片陆地，返回面积 5，用它更新最大面积。

访问过的陆地要改成 0，防止后面重复计算同一座岛。

### 70. 最长连续序列

- 类型：哈希集合。
- 分析：要求 O(n)，不能排序。把所有数放进集合。只有当 `x-1` 不存在时，x 才可能是一段连续序列的起点，然后向后数长度。
- 思路：遍历集合；若 `x-1 not in set`，从 x 开始 while 查 `cur+1`。
- 核心结构：哈希集合。
- 坑：只从起点开始数，否则会退化；重复元素用 set 去重。
- 相似题：缺失的第一个正数、并查集连续区间。
- 记忆卡片：连续序列只从没有前驱的数开始数。

```python
class Solution:
    def longestConsecutive(self, nums) -> int:
        s = set(nums)
        ans = 0
        for x in s:
            if x - 1 not in s:
                cur = x
                while cur + 1 in s:
                    cur += 1
                ans = max(ans, cur - x + 1)
        return ans
```


#### 详细分析与小例子
要 O(n)，不能排序。把数字放进集合后，只有当 `x-1` 不存在时，x 才是一段连续序列的起点。只从起点开始向后数，能避免重复扫描。

例子：`[100,4,200,1,3,2]`。集合里 1 没有前驱 0，所以从 1 开始数到 4，长度是 4。2、3、4 都有前驱，不再作为起点重复数。

如果不判断起点，每个数字都往后数，可能退化成 O(n²)。

### 71. 买卖股票的最佳时机 II

- 类型：贪心。
- 分析：可以多次交易，但不能同时持有多股。只要明天比今天贵，就把这段上涨收益吃掉。所有连续上涨段的收益相加就是最大利润。
- 思路：遍历相邻两天，若 `prices[i] > prices[i-1]`，累加差值。
- 核心结构：累加正收益。
- 坑：可以当天买当天卖等价于不操作；只累加正差值。
- 相似题：买卖股票 I、含手续费股票。
- 记忆卡片：多次交易，把所有上涨坡都吃掉。

```python
class Solution:
    def maxProfit(self, prices):
        ans = 0
        for i in range(1, len(prices)):
            if prices[i] > prices[i - 1]:
                ans += prices[i] - prices[i - 1]
        return ans
```


#### 详细分析与小例子
可以多次交易时，只要价格从今天到明天上涨，就可以把这段利润吃掉。连续上涨的大坡可以拆成每天的小坡，收益总和一样。

例子：`[7,1,5,3,6,4]`。1 到 5 上涨赚 4，3 到 6 上涨赚 3，总利润 7。

这题和只能交易一次不同，不需要找全局最低买入点，只累加所有正差值即可。

### 72. 二叉树的最大深度

- 类型：树 DFS。
- 分析：树的最大深度等于左右子树最大深度的较大值加 1。
- 思路：空节点深度为 0；递归计算左右深度。
- 核心结构：递归返回深度。
- 坑：空树返回 0。
- 相似题：平衡二叉树、二叉树直径。
- 记忆卡片：树深度 = max(左深度, 右深度) + 1。

```python
class Solution:
    def maxDepth(self, root) -> int:
        if not root:
            return 0
        return max(self.maxDepth(root.left), self.maxDepth(root.right)) + 1
```


#### 详细分析与小例子
二叉树最大深度就是从根到最远叶子的节点数。对任意节点来说，它的深度等于左右子树深度的最大值加 1。

例子：根 3 的左子树深度是 1，右子树深度是 2，那么整棵树深度是 `max(1,2)+1=3`。

空节点深度定义为 0，这样叶子节点深度自然是 1。

### 73. 最大正方形

- 类型：二维动态规划。
- 分析：`dp[i][j]` 表示以 `(i,j)` 为右下角的最大正方形边长。若当前位置是 1，则边长由上、左、左上三个位置的最小值决定。
- 思路：遍历矩阵；若为 `'1'`，`dp[i][j]=min(up,left,diag)+1`；更新最大边长。
- 核心结构：右下角 DP。
- 坑：矩阵字符是 `'1'`；返回面积，不是边长。
- 相似题：最小路径和、 maximal rectangle。
- 记忆卡片：正方形能长多大，看上、左、左上最短板。

```python
class Solution:
    def maximalSquare(self, matrix):
        m, n = len(matrix), len(matrix[0])
        dp = [[0] * (n + 1) for _ in range(m + 1)]
        best = 0
        for i in range(1, m + 1):
            for j in range(1, n + 1):
                if matrix[i - 1][j - 1] == '1':
                    dp[i][j] = min(dp[i - 1][j], dp[i][j - 1], dp[i - 1][j - 1]) + 1
                    best = max(best, dp[i][j])
        return best * best
```


#### 详细分析与小例子
以某个格子为右下角的正方形，能扩多大，取决于它的上方、左方、左上方三个位置能形成的正方形边长。三者中最小的那个限制了当前正方形。

例子：如果上方能形成边长 2，左方边长 2，左上边长 1，那么当前位置最多只能形成边长 2 的正方形，因为左上角那块不够大。

注意题目返回面积，所以最后要用最大边长平方。

### 74. 平衡二叉树

- 类型：树 DFS。
- 分析：平衡要求每个节点左右子树高度差不超过 1。递归返回高度，如果某棵子树已经不平衡，就返回 -1 向上传播失败。
- 思路：`height(node)` 返回高度或 -1；左右任一为 -1 或高度差大于 1，则返回 -1。
- 核心结构：带失败标记的高度递归。
- 坑：不是只判断根节点，要每个节点都平衡。
- 相似题：最大深度、二叉树直径。
- 记忆卡片：查平衡顺便算高度，不平衡用 -1 传上去。

```python
class Solution:
    def isBalanced(self, root) -> bool:
        def height(node):
            if not node:
                return 0
            left = height(node.left)
            right = height(node.right)
            if left == -1 or right == -1 or abs(left - right) > 1:
                return -1
            return max(left, right) + 1
        return height(root) != -1
```


#### 详细分析与小例子
平衡二叉树要求每个节点的左右子树高度差都不超过 1。不能只检查根节点，因为某个深层子树也可能不平衡。

例子：根节点左右高度差看起来是 1，但左子树内部可能是一条很长的链，那个子树本身已经不平衡。

递归时顺便计算高度。如果发现某棵子树不平衡，返回 -1 作为失败标记，向上层快速传播。

### 75. 回文链表

- 类型：链表中点 + 反转。
- 分析：判断链表是否回文，可以找到中点，反转后半段，然后从两头向中间比较。
- 思路：快慢指针找中点；反转 slow 之后的链表；前半和后半逐个比较。
- 核心结构：中点、反转后半段。
- 坑：奇数长度时 slow 在中间，反转从 slow 开始也能比较；只比较后半段长度。
- 相似题：重排链表、反转链表。
- 记忆卡片：链表回文不能倒着走，就把后半段反过来。

```python
class Solution:
    def isPalindrome(self, head) -> bool:
        slow = fast = head
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
        prev = None
        cur = slow
        while cur:
            nxt = cur.next
            cur.next = prev
            prev = cur
            cur = nxt
        p, q = head, prev
        while q:
            if p.val != q.val:
                return False
            p = p.next
            q = q.next
        return True
```


#### 详细分析与小例子
单链表不能从尾往前走，所以判断回文时，可以先找到中点，把后半段反转，然后从头和反转后的后半段一起往后比较。

例子：`1->2->2->1`。后半段 `2->1` 反转为 `1->2`，再和前半段 `1->2` 逐个比较，完全相同，所以是回文。

奇数长度时，中间节点不影响结果，只需要比较后半段长度即可。

### 76. 乘积最大子数组

- 类型：动态规划。
- 分析：乘法有负负得正，所以只维护最大值不够，还要维护以当前位置结尾的最小乘积。遇到负数时，最大和最小会互换角色。
- 思路：遍历 x；若 x<0 交换 `max_prod/min_prod`；更新二者；答案取最大。
- 核心结构：当前位置最大/最小乘积。
- 坑：有负数和 0；初始化用第一个元素。
- 相似题：最大子数组和。
- 记忆卡片：乘积题要同时记最大和最小，因为负数会翻身。

```python
class Solution:
    def maxProduct(self, nums):
        max_p = min_p = ans = nums[0]
        for x in nums[1:]:
            if x < 0:
                max_p, min_p = min_p, max_p
            max_p = max(x, max_p * x)
            min_p = min(x, min_p * x)
            ans = max(ans, max_p)
        return ans
```


#### 详细分析与小例子
乘积和加法不同，负数会让最大值和最小值互换。一个很小的负数乘上当前负数，可能变成很大的正数。所以要同时维护以当前位置结尾的最大乘积和最小乘积。

例子：`[2,3,-2,4]`。到 3 时最大乘积是 6；遇到 -2 后，最大变成 -2，最小变成 -12；后面再遇到负数时，最小值可能翻成最大值。

遇到 0 时，相当于重新开始，公式里的 `max(x, max_p*x)` 会自然处理。

### 77. 最长公共前缀

- 类型：字符串扫描。
- 分析：最长公共前缀必须是第一个字符串的前缀。不断缩短候选前缀，直到所有字符串都以它开头。
- 思路：初始化 `prefix=strs[0]`；遍历其他字符串，若不匹配就去掉 prefix 最后一位。
- 核心结构：候选前缀。
- 坑：空数组返回空串；某个字符串为空时结果为空。
- 相似题：Trie 前缀树、最长公共子序列。
- 记忆卡片：公共前缀先拿第一个当候选，不匹配就剪短。

```python
class Solution:
    def longestCommonPrefix(self, strs) -> str:
        if not strs:
            return ''
        prefix = strs[0]
        for s in strs[1:]:
            while not s.startswith(prefix):
                prefix = prefix[:-1]
                if not prefix:
                    return ''
        return prefix
```


#### 详细分析与小例子
最长公共前缀一定是第一个字符串的某个前缀。可以先把第一个字符串当候选前缀，然后拿它去和其他字符串比较，不匹配就逐步缩短。

例子：`["flower","flow","flight"]`。初始前缀是 `flower`，和 `flow` 比缩短为 `flow`，再和 `flight` 比，最终缩短为 `fl`。

如果某次前缀缩短为空，说明没有公共前缀，可以直接返回空串。

### 78. 搜索二维矩阵 II

- 类型：矩阵搜索 / 双指针。
- 分析：矩阵每行升序、每列升序。从右上角开始，当前值太大就左移，太小就下移，每次排除一行或一列。
- 思路：`i=0,j=n-1`；比较 `matrix[i][j]` 与 target；直到越界。
- 核心结构：右上角指针。
- 坑：从左上角无法决定排除方向；右上或左下才有单调性。
- 相似题：搜索二维矩阵、二分查找。
- 记忆卡片：矩阵行列都有序，从右上角走楼梯。

```python
class Solution:
    def searchMatrix(self, matrix, target: int) -> bool:
        m, n = len(matrix), len(matrix[0])
        i, j = 0, n - 1
        while i < m and j >= 0:
            if matrix[i][j] == target:
                return True
            if matrix[i][j] > target:
                j -= 1
            else:
                i += 1
        return False
```


#### 详细分析与小例子
矩阵每行从左到右递增，每列从上到下递增。从右上角开始最方便：当前值太大就左移，太小就下移，每一步都能排除一整行或一整列。

例子：右上角是 15，target 是 5，15 太大，向左走；如果当前值变成 3，3 太小，向下走。

从左上角不行，因为太小的时候，向右和向下都会变大，无法确定排除哪边。

### 79. 验证二叉搜索树

- 类型：树 DFS。
- 分析：BST 要求当前节点值在合法区间内，左子树所有节点都小于当前值，右子树所有节点都大于当前值。不能只比较直接左右孩子。
- 思路：递归传入上下界 `(low, high)`；当前值必须满足 `low < val < high`；左子树更新上界，右子树更新下界。
- 核心结构：上下界递归。
- 坑：严格小于/大于，不能等于；全局约束不是局部约束。
- 相似题：二叉树中序遍历、BST 最近公共祖先。
- 记忆卡片：验证 BST 传范围，不只看孩子。

```python
class Solution:
    def isValidBST(self, root) -> bool:
        def dfs(node, low, high):
            if not node:
                return True
            if not (low < node.val < high):
                return False
            return dfs(node.left, low, node.val) and dfs(node.right, node.val, high)
        return dfs(root, float('-inf'), float('inf'))
```


#### 详细分析与小例子
验证 BST 不能只看当前节点和左右孩子。左子树的所有节点都必须小于当前节点，右子树的所有节点都必须大于当前节点，这是全局范围约束。

例子：根 5，右孩子 7，但 7 的左孩子是 4。4 虽然小于 7，但它在 5 的右子树里，必须大于 5，所以这不是 BST。

递归时给每个节点传合法范围 `(low, high)`，当前值必须严格落在范围内。

### 80. 二叉树的前序遍历

- 类型：树 DFS / 栈。
- 分析：前序顺序是根、左、右。迭代时用栈，先压右孩子再压左孩子，保证左孩子先出栈。
- 思路：根入栈；弹出访问；按右、左顺序压栈。
- 核心结构：栈。
- 坑：压栈顺序和访问顺序相反。
- 相似题：中序遍历、后序遍历。
- 记忆卡片：前序根左右；栈里先压右再压左。

```python
class Solution:
    def preorderTraversal(self, root):
        if not root:
            return []
        st, ans = [root], []
        while st:
            node = st.pop()
            ans.append(node.val)
            if node.right:
                st.append(node.right)
            if node.left:
                st.append(node.left)
        return ans
```


#### 详细分析与小例子
前序遍历顺序是根、左、右。用栈迭代时，因为栈后进先出，要先压右孩子，再压左孩子，这样左孩子会先被弹出访问。

例子：根 1，左 2，右 3。先访问 1，再把 3 压栈、2 压栈；弹出时先弹 2，再弹 3，顺序就是 1、2、3。

如果压栈顺序反了，结果会变成根、右、左。

### 81. 最大数

- 类型：自定义排序。
- 分析：要让拼接后的数最大，两个数字字符串 a、b 的顺序取决于 `a+b` 和 `b+a` 哪个更大。
- 思路：转字符串；按比较器排序；拼接；若结果以 0 开头说明全是 0。
- 核心结构：拼接比较器。
- 坑：不能按数值大小排序；全 0 返回 `"0"`。
- 相似题：下一个排列、排序数组。
- 记忆卡片：谁在前看拼接：`ab > ba`，a 就在 b 前。

```python
class Solution:
    def largestNumber(self, nums) -> str:
        from functools import cmp_to_key
        arr = list(map(str, nums))

        def cmp(a, b):
            if a + b > b + a:
                return -1
            if a + b < b + a:
                return 1
            return 0

        arr.sort(key=cmp_to_key(cmp))
        ans = ''.join(arr)
        return '0' if ans[0] == '0' else ans
```


#### 详细分析与小例子
要让拼接后的数字最大，两个数字 a 和 b 谁在前，不看它们本身大小，而看 `a+b` 和 `b+a` 哪个更大。

例子：`3` 和 `30`。`"330"` 大于 `"303"`，所以 3 应该放在 30 前面。`9` 和 `34`，`"934"` 大于 `"349"`，所以 9 在前。

如果排序后第一个字符是 0，说明所有数都是 0，答案应该是 `"0"`，不是 `"000"`。

### 82. 二叉树最大宽度

- 类型：树 BFS + 编号。
- 分析：把二叉树按完全二叉树编号，某层宽度就是最右编号减最左编号加 1。即使中间有空节点，也能被编号差体现。
- 思路：队列保存 `(node,index)`；每层记录首尾 index；孩子编号为 `2*i`、`2*i+1`。为防数字过大，每层可减去本层起始编号。
- 核心结构：层序遍历 + 节点编号。
- 坑：宽度包含中间空位；不是该层节点数量。
- 相似题：层序遍历、右视图。
- 记忆卡片：宽度不是节点数，是完全二叉树编号的跨度。

```python
class Solution:
    def widthOfBinaryTree(self, root) -> int:
        from collections import deque
        q = deque([(root, 0)])
        ans = 0
        while q:
            size = len(q)
            base = q[0][1]
            first = last = 0
            for i in range(size):
                node, idx = q.popleft()
                idx -= base
                if i == 0:
                    first = idx
                if i == size - 1:
                    last = idx
                if node.left:
                    q.append((node.left, idx * 2))
                if node.right:
                    q.append((node.right, idx * 2 + 1))
            ans = max(ans, last - first + 1)
        return ans
```


#### 详细分析与小例子
最大宽度包含中间空节点，所以不能只数每层真实节点数量。把树按完全二叉树编号，每层宽度就是最右编号减最左编号加 1。

例子：某层真实节点在编号 4 和 7，中间 5、6 位置为空，这层宽度仍然是 `7-4+1=4`。

编号可能很大，所以每层可以减去本层最左编号，只保留相对位置。

### 83. 旋转图像

- 类型：矩阵原地变换。
- 分析：顺时针旋转 90 度可以拆成两步：先沿主对角线转置，再每一行反转。
- 思路：双循环交换 `matrix[i][j]` 和 `matrix[j][i]`；然后每行 `reverse`。
- 核心结构：转置 + 行反转。
- 坑：必须原地；转置时只遍历上三角，避免换两次。
- 相似题：螺旋矩阵。
- 记忆卡片：顺时针 90 度 = 转置后每行反转。

```python
class Solution:
    def rotate(self, matrix) -> None:
        n = len(matrix)
        for i in range(n):
            for j in range(i + 1, n):
                matrix[i][j], matrix[j][i] = matrix[j][i], matrix[i][j]
        for row in matrix:
            row.reverse()
```


#### 详细分析与小例子
顺时针旋转 90 度可以拆成两步：先沿主对角线转置，再反转每一行。这样能原地完成，不需要额外矩阵。

例子：`[[1,2],[3,4]]`。转置后变成 `[[1,3],[2,4]]`；每行反转后变成 `[[3,1],[4,2]]`，这就是顺时针旋转结果。

转置时只交换上三角和下三角对应位置，避免同一对元素交换两次。

### 84. 二叉树的直径

- 类型：树 DFS。
- 分析：直径可能经过某个节点，长度等于左子树最大深度 + 右子树最大深度。DFS 返回深度，同时用全局变量更新直径。
- 思路：`depth(node)` 返回节点向下最大边数对应的节点深度；每个节点更新 `ans=left+right`。
- 核心结构：深度返回值 + 全局最大直径。
- 坑：题目直径按边数，不是节点数；空节点深度为 0 时 `left+right` 正好是边数。
- 相似题：最大路径和、最大深度。
- 记忆卡片：直径在某节点拐弯，等于左深度加右深度。

```python
class Solution:
    def diameterOfBinaryTree(self, root) -> int:
        self.ans = 0

        def depth(node):
            if not node:
                return 0
            left = depth(node.left)
            right = depth(node.right)
            self.ans = max(self.ans, left + right)
            return max(left, right) + 1

        depth(root)
        return self.ans
```


#### 详细分析与小例子
二叉树直径可能经过某个节点，路径长度等于这个节点左子树深度加右子树深度。DFS 计算深度的同时，用每个节点尝试更新答案。

例子：某节点左边最长向下路径有 2 条边，右边有 3 条边，那么经过这个节点的直径候选是 5 条边。

题目里的直径按边数算，不是节点数。用空节点深度 0、叶子返回 1 时，`left+right` 正好是边数。

### 85. 寻找峰值

- 类型：二分。
- 分析：如果 `nums[mid] < nums[mid+1]`，右边一定存在峰值，因为走势向上；否则左边包含 mid 在内一定存在峰值。
- 思路：二分区间 `[l,r]`；比较 `mid` 和 `mid+1`；向更高的一侧收缩。
- 核心结构：坡度判断。
- 坑：循环用 `l < r`，因为会访问 `mid+1`。
- 相似题：山脉数组峰顶、二分答案。
- 记忆卡片：找峰值就往高处走，高处那边一定有峰。

```python
class Solution:
    def findPeakElement(self, nums) -> int:
        l, r = 0, len(nums) - 1
        while l < r:
            mid = (l + r) // 2
            if nums[mid] < nums[mid + 1]:
                l = mid + 1
            else:
                r = mid
        return l
```


#### 详细分析与小例子
如果 `nums[mid] < nums[mid+1]`，说明 mid 右边是上坡方向，右侧一定存在峰值；否则左侧包含 mid 的区域一定存在峰值。这样每次能排除一半。

例子：`[1,2,3,1]`。中点附近如果看到 2 到 3 是上升，就往右找，最后找到峰值 3。

循环条件用 `l < r`，因为代码会访问 `mid+1`，最终 l 和 r 相遇的位置就是峰值。

### 86. 不同路径

- 类型：网格动态规划。
- 分析：只能向右或向下，走到一个格子的路径数等于从上方来的路径数加从左方来的路径数。
- 思路：一维 DP，`dp[j]` 表示当前行到第 j 列的路径数；每格更新 `dp[j]+=dp[j-1]`。
- 核心结构：一维滚动数组。
- 坑：第一行和第一列都只有一种走法。
- 相似题：最小路径和、爬楼梯。
- 记忆卡片：路径数看来源：上面 + 左边。

```python
class Solution:
    def uniquePaths(self, m: int, n: int) -> int:
        dp = [1] * n
        for _ in range(1, m):
            for j in range(1, n):
                dp[j] += dp[j - 1]
        return dp[-1]
```


#### 详细分析与小例子
只能向右或向下，所以到某个格子的路径数等于到上方格子的路径数加到左方格子的路径数。这是典型网格 DP。

例子：`m=3,n=2`。从左上到右下，一共要走 2 次向下和 1 次向右，可能路径有 3 条。DP 也会算出 3。

第一行和第一列都只有一种走法，所以一维 dp 可以初始化为全 1。

### 87. 和为 K 的子数组

- 类型：前缀和 + 哈希表。
- 分析：子数组和为 k 等价于 `prefix[j] - prefix[i] = k`，即之前出现过 `prefix[j]-k`。用哈希表统计每个前缀和出现次数。
- 思路：遍历 nums 累加前缀和 `pre`；答案加 `cnt[pre-k]`；再把当前 `pre` 计数加 1。
- 核心结构：前缀和频次表。
- 坑：初始化 `cnt[0]=1`，表示从下标 0 开始的子数组；有负数不能用滑动窗口。
- 相似题：两数之和、连续子数组和。
- 记忆卡片：子数组和 K：当前前缀问以前有没有 `pre-k`。

```python
class Solution:
    def subarraySum(self, nums, k: int) -> int:
        from collections import defaultdict
        cnt = defaultdict(int)
        cnt[0] = 1
        pre = ans = 0
        for x in nums:
            pre += x
            ans += cnt[pre - k]
            cnt[pre] += 1
        return ans
```


#### 详细分析与小例子
子数组和可以用前缀和表示。若当前前缀和是 `pre`，想找和为 k 的子数组，就要看之前有没有前缀和等于 `pre-k`。

例子：`nums=[1,1,1], k=2`。走到第二个 1 时，当前前缀和是 2，之前有前缀和 0，所以 `[1,1]` 是答案；走到第三个 1 时，当前前缀和是 3，之前有前缀和 1，又找到一个 `[1,1]`。

因为数组可能有负数，不能用滑动窗口；前缀和 + 哈希表更通用。

### 88. 路径总和 II

- 类型：树 DFS + 回溯。
- 分析：需要返回所有从根到叶子的路径，路径和等于 target。DFS 过程中维护当前路径和剩余目标，到叶子时判断。
- 思路：进入节点时加入 path，减少 remain；若叶子且 remain 等于节点值，收集路径；递归结束后弹出。
- 核心结构：`path/remain`。
- 坑：必须是根到叶子；加入答案要拷贝 path。
- 相似题：路径总和、求根到叶数字之和。
- 记忆卡片：路径题 DFS 带 path，叶子才结算。

```python
class Solution:
    def pathSum(self, root, targetSum: int):
        ans, path = [], []

        def dfs(node, remain):
            if not node:
                return
            path.append(node.val)
            if not node.left and not node.right and remain == node.val:
                ans.append(path[:])
            dfs(node.left, remain - node.val)
            dfs(node.right, remain - node.val)
            path.pop()

        dfs(root, targetSum)
        return ans
```


#### 详细分析与小例子
这题要所有根到叶子的路径，所以 DFS 过程中要维护当前 path。每走到一个节点，就把它加入 path；离开这个节点时，再把它弹出，恢复现场。

例子：target=22，路径 `5->4->11->2` 的和正好是 22，且 2 是叶子，所以这条路径加入答案。如果中途和等于 22 但还没到叶子，也不能算。

加入答案时要拷贝 `path[:]`，否则后续回溯修改 path 会影响已经保存的结果。

### 89. 两两交换链表中的节点

- 类型：链表局部交换。
- 分析：每次交换相邻两个节点，需要改变三条指针。用 dummy 和 `pre` 指向待交换二元组前一个节点。
- 思路：令 `a=pre.next,b=a.next`；执行 `a.next=b.next; b.next=a; pre.next=b`；`pre` 移到 a。
- 核心结构：`pre/a/b`。
- 坑：不能只交换值；最后不足两个节点保持原样。
- 相似题：K 个一组翻转链表、反转链表 II。
- 记忆卡片：两两交换就是 k=2 的分组反转。

```python
class Solution:
    def swapPairs(self, head):
        dummy = ListNode(0, head)
        pre = dummy
        while pre.next and pre.next.next:
            a = pre.next
            b = a.next
            a.next = b.next
            b.next = a
            pre.next = b
            pre = a
        return dummy.next
```


#### 详细分析与小例子
两两交换就是每次处理两个节点。用 `pre` 指向这一对节点前面的位置，`a=pre.next`，`b=a.next`，然后把 `b` 放到 `a` 前面。

例子：`1->2->3->4`。第一轮把 2 放到 1 前面，得到 `2->1->3->4`；第二轮把 4 放到 3 前面，得到 `2->1->4->3`。

最后如果只剩一个节点，没有成对，就保持原样。

### 90. 长度最小的子数组

- 类型：滑动窗口。
- 分析：数组元素为正数，窗口和随右指针扩张只会增大，随左指针收缩只会减小，因此适合滑动窗口。
- 思路：右指针累加；当窗口和 `>= target` 时，不断收缩左边并更新最短长度。
- 核心结构：`left/right/window_sum`。
- 坑：只有正数时才能这样做；无答案返回 0。
- 相似题：最小覆盖子串、无重复字符最长子串。
- 记忆卡片：正数数组求最短连续满足和，扩到够，再缩到不够。

```python
class Solution:
    def minSubArrayLen(self, target: int, nums) -> int:
        left = total = 0
        ans = float('inf')
        for right, x in enumerate(nums):
            total += x
            while total >= target:
                ans = min(ans, right - left + 1)
                total -= nums[left]
                left += 1
        return 0 if ans == float('inf') else ans
```


#### 详细分析与小例子
数组元素都是正数，所以窗口和随着右指针右移只会增大，随着左指针右移只会减小。这种单调性使滑动窗口成立。

例子：target=7，`nums=[2,3,1,2,4,3]`。窗口扩到 `[2,3,1,2]` 和为 8，满足后开始缩；后面找到 `[4,3]`，长度 2，是最短。

如果数组有负数，这种窗口和单调性就不存在，不能直接用这个模板。

### 91. 打家劫舍

- 类型：动态规划。
- 分析：每间房只有偷或不偷。偷第 i 间，则不能偷 i-1，收益是 `dp[i-2]+nums[i]`；不偷第 i 间，收益是 `dp[i-1]`。
- 思路：滚动变量 `prev2/prev1` 表示前两项；每轮更新 `cur=max(prev1,prev2+x)`。
- 核心结构：相邻不能同时选的 DP。
- 坑：空数组时返回 0；只需要 O(1) 空间。
- 相似题：打家劫舍 II、买卖股票、爬楼梯。
- 记忆卡片：偷当前就隔一家，不偷当前继承上一家。

```python
class Solution:
    def rob(self, nums) -> int:
        prev2 = prev1 = 0
        for x in nums:
            prev2, prev1 = prev1, max(prev1, prev2 + x)
        return prev1
```


#### 详细分析与小例子
每间房只有偷或不偷。偷当前房子，就不能偷上一间，收益是“前前间最优 + 当前金额”；不偷当前房子，收益就是“上一间最优”。两者取最大。

例子：`[1,2,3,1]`。到金额 3 时，可以偷 1 和 3，总收益 4；也可以不偷 3，保持前面收益 2。取 4。最后答案是 4。

这是“相邻不能同时选”的经典 DP。

### 92. 路径总和

- 类型：树 DFS。
- 分析：判断是否存在根到叶子的路径和等于 target。递归时不断减去当前节点值，到叶子时检查剩余值是否等于叶子值。
- 思路：空节点 false；叶子节点判断 `targetSum == root.val`；否则递归左右子树。
- 核心结构：剩余目标值。
- 坑：必须到叶子，不能中途满足就返回 true。
- 相似题：路径总和 II、求根到叶数字之和。
- 记忆卡片：路径和题，剩余值一路往下减，叶子结算。

```python
class Solution:
    def hasPathSum(self, root, targetSum: int) -> bool:
        if not root:
            return False
        if not root.left and not root.right:
            return targetSum == root.val
        return self.hasPathSum(root.left, targetSum - root.val) or self.hasPathSum(root.right, targetSum - root.val)
```


#### 详细分析与小例子
路径必须从根到叶子。递归时把 target 不断减去当前节点值，走到叶子时检查剩余值是否正好等于叶子值。

例子：target=22，路径 `5->4->11->2`。剩余值依次变成 17、13、2，到叶子 2 时正好匹配，返回 true。

不能在中间节点提前返回 true，因为题目要求必须到叶子。

### 93. 删除排序链表中的重复元素

- 类型：链表扫描。
- 分析：链表已排序，重复节点一定相邻。只要当前节点和下一个节点值相同，就跳过下一个节点。
- 思路：`cur=head`；若 `cur.val==cur.next.val`，令 `cur.next=cur.next.next`；否则 cur 前进。
- 核心结构：当前指针。
- 坑：本题保留一个重复值；和 II 的“全部删除”不同。
- 相似题：删除排序链表重复元素 II。
- 记忆卡片：排序链表去重，只和隔壁比。

```python
class Solution:
    def deleteDuplicates(self, head):
        cur = head
        while cur and cur.next:
            if cur.val == cur.next.val:
                cur.next = cur.next.next
            else:
                cur = cur.next
        return head
```


#### 详细分析与小例子
链表已经排序，所以重复值一定相邻。只要当前节点和下一个节点值相同，就跳过下一个节点；不同才移动当前指针。

例子：`1->1->2->3->3`。先发现两个 1 相邻，跳过第二个 1；后面发现两个 3 相邻，跳过第二个 3，结果是 `1->2->3`。

这题和“删除重复元素 II”不同：这里重复值保留一个，II 是全部删除。

### 94. 单词拆分

- 类型：动态规划。
- 分析：`dp[i]` 表示 `s[:i]` 能否被字典拆分。若存在 `j<i`，使 `dp[j]` 为真且 `s[j:i]` 在字典里，则 `dp[i]` 为真。
- 思路：初始化 `dp[0]=True`；遍历 i，枚举 j；命中后 break。
- 核心结构：布尔 DP + 单词集合。
- 坑：字典用 set；`dp[0]` 表示空串可拆。
- 相似题：零钱兑换、完全背包、编辑距离。
- 记忆卡片：能否拆到 i，看前面某个 j 能拆，且 j 到 i 是单词。

```python
class Solution:
    def wordBreak(self, s: str, wordDict) -> bool:
        words = set(wordDict)
        dp = [False] * (len(s) + 1)
        dp[0] = True
        for i in range(1, len(s) + 1):
            for j in range(i):
                if dp[j] and s[j:i] in words:
                    dp[i] = True
                    break
        return dp[-1]
```


#### 详细分析与小例子
`dp[i]` 表示字符串前 i 个字符能不能被字典拆分。若存在某个 j，使得 `dp[j]` 为真，并且 `s[j:i]` 是字典单词，那么 `dp[i]` 也为真。

例子：`s="leetcode"`, 字典有 `"leet"` 和 `"code"`。`dp[4]` 因为 `"leet"` 为真；之后 `dp[8]` 因为 `dp[4]` 为真且 `"code"` 在字典里，所以整体可拆。

`dp[0]=True` 表示空串可拆，这是后续单词从开头开始匹配的基础。

### 95. 基本计算器 II

- 类型：栈 / 表达式模拟。
- 分析：只有 `+ - * /` 和非负整数。乘除优先级高，可以在读到一个新操作符时，把上一个操作符和数字结算：加减入栈，乘除直接和栈顶合并。
- 思路：扫描并累积数字；遇到操作符或末尾时，根据前一个 `sign` 处理当前数字；最后栈求和。
- 核心结构：栈保存已经处理过的项。
- 坑：整数除法向 0 截断，Python `//` 对负数是向下取整，要用 `int(a / b)`；末尾数字也要处理。
- 相似题：字符串解码、逆波兰表达式。
- 记忆卡片：加减先入栈，乘除立刻和栈顶算。

```python
class Solution:
    def calculate(self, s: str) -> int:
        st = []
        num = 0
        sign = '+'
        for i, c in enumerate(s):
            if c.isdigit():
                num = num * 10 + int(c)
            if (not c.isdigit() and c != ' ') or i == len(s) - 1:
                if sign == '+':
                    st.append(num)
                elif sign == '-':
                    st.append(-num)
                elif sign == '*':
                    st.append(st.pop() * num)
                elif sign == '/':
                    st.append(int(st.pop() / num))
                sign = c
                num = 0
        return sum(st)
```


#### 详细分析与小例子
表达式里乘除优先级高于加减。可以用栈保存已经处理好的项：遇到加减就把数字正负入栈；遇到乘除就立刻和栈顶合并。

例子：`"3+2*2"`。先把 3 入栈；遇到 `+` 后把 2 入栈；遇到 `*` 时，下一数字 2 要和栈顶 2 相乘，变成 4。最后栈是 `[3,4]`，和为 7。

Python 里负数整除要小心，题目要求向 0 截断，所以用 `int(a / b)`。

### 96. 最长重复子数组

- 类型：二维动态规划。
- 分析：重复子数组要求连续，和 LCS 不同。`dp[i][j]` 表示以 `nums1[i-1]` 和 `nums2[j-1]` 结尾的最长公共后缀长度。若当前元素相等，就从左上角加 1，否则为 0。
- 思路：双循环填表并更新最大值。
- 核心结构：公共后缀 DP。
- 坑：连续，所以不相等时必须归 0。
- 相似题：最长公共子序列、最长公共子串。
- 记忆卡片：连续匹配看后缀，相等左上 +1，不等清零。

```python
class Solution:
    def findLength(self, nums1, nums2) -> int:
        m, n = len(nums1), len(nums2)
        dp = [[0] * (n + 1) for _ in range(m + 1)]
        ans = 0
        for i in range(1, m + 1):
            for j in range(1, n + 1):
                if nums1[i - 1] == nums2[j - 1]:
                    dp[i][j] = dp[i - 1][j - 1] + 1
                    ans = max(ans, dp[i][j])
        return ans
```


#### 详细分析与小例子
重复子数组要求连续，所以 `dp[i][j]` 表示以 `nums1[i-1]` 和 `nums2[j-1]` 结尾的最长公共连续长度。如果两个元素相等，就接在前一个公共后缀后面；不相等就断掉。

例子：`nums1=[1,2,3,2,1]`, `nums2=[3,2,1,4,7]`。公共连续子数组是 `[3,2,1]`，长度 3。

和最长公共子序列不同，不连续不行，所以不相等时必须清零。

### 97. 翻转二叉树

- 类型：树 DFS。
- 分析：翻转二叉树就是每个节点左右孩子交换。对当前节点交换后，再递归翻转左右子树。
- 思路：空节点返回；交换 `root.left/root.right`；递归左右；返回 root。
- 核心结构：递归交换。
- 坑：交换后递归左右都可以，因为左右引用已换好。
- 相似题：对称二叉树。
- 记忆卡片：翻转树，每个节点都交换左右孩子。

```python
class Solution:
    def invertTree(self, root):
        if not root:
            return None
        root.left, root.right = root.right, root.left
        self.invertTree(root.left)
        self.invertTree(root.right)
        return root
```


#### 详细分析与小例子
翻转二叉树就是每个节点的左右孩子交换。交换当前节点后，再递归处理左右子树即可。

例子：根 4，左子树 2，右子树 7。翻转后，7 到左边，2 到右边；然后 2 和 7 的子节点也分别交换。

这题不需要复杂状态，核心就是对每个节点执行同一个动作：交换左右孩子。

### 98. 多数元素

- 类型：摩尔投票。
- 分析：多数元素出现次数超过一半。把多数元素和其他元素两两抵消，最后剩下的一定是多数元素。
- 思路：维护候选 `cand` 和计数 `cnt`；计数为 0 时换候选；遇到候选加 1，否则减 1。
- 核心结构：候选值和票数。
- 坑：本题保证多数元素存在；若不保证，需要最后再验证。
- 相似题：出现次数超过 n/3 的元素。
- 记忆卡片：多数元素超过半数，和别人抵消后还能活下来。

```python
class Solution:
    def majorityElement(self, nums):
        cand = None
        cnt = 0
        for x in nums:
            if cnt == 0:
                cand = x
            cnt += 1 if x == cand else -1
        return cand
```


#### 详细分析与小例子
多数元素出现次数超过一半。把多数元素和其他元素两两抵消，最后剩下的候选一定还是多数元素。摩尔投票就是模拟这个抵消过程。

例子：`[2,2,1,1,1,2,2]`。2 和其他数抵消若干次后，因为 2 总次数超过一半，最后候选仍然会回到 2。

本题保证多数元素存在。如果不保证，投票结束后还要再数一遍候选是否真的超过一半。

### 99. 移动零

- 类型：双指针 / 原地数组。
- 分析：保持非零元素相对顺序，把它们依次写到数组前面，剩下位置自然填 0。也可以边遍历边交换。
- 思路：`slow` 指向下一个非零应该放的位置；遍历 `fast`，遇到非零就交换 `nums[slow]` 和 `nums[fast]`，`slow++`。
- 核心结构：慢指针维护非零区尾部。
- 坑：要求原地；非零元素相对顺序不能变。
- 相似题：移除元素、颜色分类。
- 记忆卡片：slow 前面全是整理好的非零数。

```python
class Solution:
    def moveZeroes(self, nums) -> None:
        slow = 0
        for fast in range(len(nums)):
            if nums[fast] != 0:
                nums[slow], nums[fast] = nums[fast], nums[slow]
                slow += 1
```


#### 详细分析与小例子
目标是把非零元素按原相对顺序放到前面，零移动到后面。`slow` 指向下一个非零元素应该放的位置，`fast` 负责扫描数组。

例子：`[0,1,0,3,12]`。fast 扫到 1，把它换到 slow=0 的位置；扫到 3，把它换到 slow=1 的位置；扫到 12，把它换到 slow=2 的位置。结果是 `[1,3,12,0,0]`。

这个过程保持了非零元素的相对顺序，因为 fast 是从左到右扫描的。

### 100. 课程表

- 类型：图 / 拓扑排序。
- 分析：课程依赖构成有向图。若图中有环，则环上的课程互相等待，无法完成。拓扑排序通过不断移除入度为 0 的课程来判断是否能学完。
- 思路：建图 `b -> a` 表示学 a 前要先学 b；统计入度；所有入度 0 入队；弹出课程并降低后继入度；最终看处理数量是否为 numCourses。
- 核心结构：邻接表 + 入度数组 + 队列。
- 坑：边方向别反；判断的是是否能完成所有课程。
- 相似题：课程表 II、任务调度器、拓扑排序模板题。
- 记忆卡片：依赖题看有向环；入度为 0 的先学，学完删边。

```python
class Solution:
    def canFinish(self, numCourses: int, prerequisites) -> bool:
        from collections import deque
        graph = [[] for _ in range(numCourses)]
        indeg = [0] * numCourses
        for a, b in prerequisites:
            graph[b].append(a)
            indeg[a] += 1
        q = deque(i for i in range(numCourses) if indeg[i] == 0)
        seen = 0
        while q:
            u = q.popleft()
            seen += 1
            for v in graph[u]:
                indeg[v] -= 1
                if indeg[v] == 0:
                    q.append(v)
        return seen == numCourses
```


#### 详细分析与小例子
课程依赖可以看成有向图。`[a,b]` 表示学 a 之前必须先学 b，也就是有一条边 `b -> a`。如果图里有环，就会出现互相等待，无法完成所有课程。

例子：课程 0 依赖 1，课程 1 又依赖 0。两门课都等对方先学，队列里没有入度为 0 的课程，所以无法完成。

拓扑排序的做法是：先学所有入度为 0 的课程；学完一门课，就把它指向的后续课程入度减 1。最后如果学过的课程数等于总数，说明没有环。

