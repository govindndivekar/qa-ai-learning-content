# Interview Preparation — DS&A, System Design, Behavioral, QA & AI Interviews

> Subject #8 of the Master Learning Plan.  
> Owner: Govind Divekar. Prerequisites: Python (file 04) or TypeScript (file 02) for coding interviews. Estimated time: ~80 hours over Months 4–6.  
> Target outcome: Pass technical screens and on-sites at a QA/SDET or junior dev level with structured frameworks and a strong portfolio story.

---

## 1. Overview & Learning Outcomes

After 3 months building Python/TypeScript and QA foundations, you now prepare for live interviews end-to-end. This subject weaves five pillars:

1. **DS&A (Data Structures & Algorithms):** 45-minute whiteboard rounds; 150 curated problems by pattern.
2. **System Design:** 45-minute architecture drills; how to talk about scale, trade-offs, and testability.
3. **Behavioral:** STAR stories, company research, leadership principles alignment.
4. **QA-Specific:** Test case design, automation frameworks, bug reports, metrics, live coding.
5. **AI-Era:** GenAI product testing, prompt engineering, portfolio proof.

**Learning outcomes:**
- Solve 2 medium LeetCode problems per week with clean code and strong verbal narration.
- Write 1-page system design docs for 8 canonical systems by Month 6.
- Polish 7 STAR stories to ≤ 2 minutes each; articulate "why this company."
- Design test suites for 5 common products (ATM, chat, feed, ride-share, search).
- Conduct and record 6 mock interviews; achieve >80% pass rate on behavioral alignment.
- Build 1 capstone portfolio project showcasing QA + GenAI (from file 01 or new).

---

## 2. Detailed Syllabus

### Months 4–6: 80 hours over 12 weeks

| Week | Pillar Focus | Hours | Deliverable |
|------|-------------|-------|-------------|
| 13 | DS&A Intro + Big-O + Arrays | 8 | 8 easy problems; Big-O cheat sheet |
| 14 | Hashing + Sliding Window | 8 | 6 problems (two-pointer, hash); 1 STAR story |
| 15 | Stacks, Queues, Linked Lists | 8 | 6 problems; LC pattern guide |
| 16 | Trees + Graph Basics | 8 | 8 problems (DFS, BFS, topo sort) |
| 17 | Heaps, Bit, Binary Search | 8 | 6 problems; framework doc |
| 18 | DP Fundamentals | 8 | 8 DP problems; state-transition writeup |
| 19 | System Design 1: URL Shortener, Rate Limiter | 8 | 2 × 1-page designs |
| 20 | System Design 2: Chat, Feed Notification | 8 | 2 × 1-page designs |
| 21 | Behavioral + QA Design (Test Case, Bug Reports) | 8 | 5 STAR stories; 3 test-case designs |
| 22 | Automation Frameworks + Metrics + Live Coding | 8 | Framework writeup; 2 test code examples |
| 23 | AI-Era (GenAI Testing, Prompt Engineering) + Polish | 8 | Portfolio project + demo; prompt log |
| 24 | **4-Week Interview Sprint Capstone** | 8 | 6 mock interviews; post-mortem |

**Total: 96 hours** (slightly over 80 to buffer review/slack).

---

## 3. Topics — Explanations with Examples

### Pillar 1: DS&A (Data Structures & Algorithms)

#### 1. Big-O & Complexity Analysis

**Concept:**  
Big-O describes worst-case time/space as input grows. Common complexities: O(1), O(log n), O(n), O(n log n), O(n²), O(2^n), O(n!).  
Amortized analysis (e.g., dynamic array append) spreads cost over many operations.

**Example (Python):**
```python
# O(n) linear scan
def contains(arr, target):
    for x in arr:
        if x == target:
            return True
    return False

# O(log n) binary search (requires sorted)
def binary_search(arr, target):
    left, right = 0, len(arr) - 1
    while left <= right:
        mid = (left + right) // 2
        if arr[mid] == target:
            return True
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return False

# O(1) amortized append (dynamic array)
class DynamicArray:
    def __init__(self):
        self.items = []
    def append(self, x):
        # Resize cost amortized: 1 + 2 + 4 + 8 + ... = 2n
        self.items.append(x)
```

**Pitfall:** Confusing "looks fast" code with actual complexity. A nested loop is O(n²). String concatenation in a loop is O(n²) in some languages.

---

#### 2. Arrays & Strings

**Concept:**  
Contiguous memory; fast index access O(1). Key patterns: sliding window (two pointers tracking a subarray), prefix sums (cumulative), Kadane's (max subarray).

**Example:**
```python
# Sliding window: max sum of k consecutive elements
def max_k_sum(arr, k):
    if k > len(arr):
        return 0
    window_sum = sum(arr[:k])
    max_sum = window_sum
    for i in range(k, len(arr)):
        window_sum = window_sum - arr[i - k] + arr[i]
        max_sum = max(max_sum, window_sum)
    return max_sum

# Two pointers: sorted array, find pair summing to target
def two_sum_sorted(arr, target):
    left, right = 0, len(arr) - 1
    while left < right:
        s = arr[left] + arr[right]
        if s == target:
            return (arr[left], arr[right])
        elif s < target:
            left += 1
        else:
            right -= 1
    return None

# Prefix sum: count subarrays with sum = k
def subarray_sum(arr, k):
    count = 0
    prefix = 0
    seen = {0: 1}
    for x in arr:
        prefix += x
        if prefix - k in seen:
            count += seen[prefix - k]
        seen[prefix] = seen.get(prefix, 0) + 1
    return count
```

**Pitfall:** Off-by-one errors with pointers; forgetting to handle edge cases (empty array, k=0).

---

#### 3. Hashing

**Concept:**  
Hash maps (O(1) avg lookup) for frequency counts, grouping, cache.

**Example:**
```python
# Frequency count: most frequent elements
from collections import Counter, defaultdict

def top_k_frequent(arr, k):
    count = Counter(arr)
    return [x for x, _ in count.most_common(k)]

# Group by key: anagrams
def group_anagrams(words):
    groups = defaultdict(list)
    for word in words:
        key = ''.join(sorted(word))
        groups[key].append(word)
    return list(groups.values())

# Two-sum via hash
def two_sum(arr, target):
    seen = {}
    for i, x in enumerate(arr):
        if target - x in seen:
            return [seen[target - x], i]
        seen[x] = i
    return None
```

**Pitfall:** Hash collision handling (handled by language). In interviews, assume O(1) lookup unless told otherwise.

---

#### 4. Linked Lists

**Concept:**  
Nodes with value + next pointer. Key: reverse, detect cycle (Floyd's tortoise-hare), merge.

**Example:**
```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

# Reverse a linked list
def reverse_list(head):
    prev = None
    while head:
        next_temp = head.next
        head.next = prev
        prev = head
        head = next_temp
    return prev

# Detect cycle (Floyd's algorithm)
def has_cycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            return True
    return False

# Merge two sorted lists
def merge_sorted_lists(l1, l2):
    dummy = ListNode(0)
    curr = dummy
    while l1 and l2:
        if l1.val <= l2.val:
            curr.next = l1
            l1 = l1.next
        else:
            curr.next = l2
            l2 = l2.next
        curr = curr.next
    curr.next = l1 if l1 else l2
    return dummy.next
```

**Pitfall:** Null pointer dereference; not tracking prev/next carefully during reversal.

---

#### 5. Stacks & Queues

**Concept:**  
LIFO (stack) and FIFO (queue). Monotonic stack for "next greater element" patterns. Deque for efficient front/back ops.

**Example:**
```python
# Next greater element (monotonic stack)
def next_greater(arr):
    stack = []
    result = [-1] * len(arr)
    for i in range(len(arr) - 1, -1, -1):
        while stack and stack[-1] <= arr[i]:
            stack.pop()
        if stack:
            result[i] = stack[-1]
        stack.append(arr[i])
    return result

# Valid parentheses (stack)
def is_valid(s):
    stack = []
    pairs = {'(': ')', '[': ']', '{': '}'}
    for c in s:
        if c in pairs:
            stack.append(c)
        elif not stack or pairs[stack.pop()] != c:
            return False
    return len(stack) == 0

# Queue simulation (list + popleft, or collections.deque)
from collections import deque
q = deque()
q.append(1)  # enqueue
q.popleft()  # dequeue O(1)
```

**Pitfall:** Using list.pop(0) is O(n); use deque for O(1) operations.

---

#### 6. Trees

**Concept:**  
Hierarchical structure. DFS (pre/in/post-order), BFS (level-order). BST property: left < node < right.

**Example:**
```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

# Preorder (root → left → right)
def preorder(root):
    if not root:
        return []
    return [root.val] + preorder(root.left) + preorder(root.right)

# Level-order BFS
def level_order(root):
    if not root:
        return []
    result = []
    q = deque([root])
    while q:
        level = []
        for _ in range(len(q)):
            node = q.popleft()
            level.append(node.val)
            if node.left:
                q.append(node.left)
            if node.right:
                q.append(node.right)
        result.append(level)
    return result

# Inorder BST validation
def is_valid_bst(root, min_val=float('-inf'), max_val=float('inf')):
    if not root:
        return True
    if root.val <= min_val or root.val >= max_val:
        return False
    return (is_valid_bst(root.left, min_val, root.val) and
            is_valid_bst(root.right, root.val, max_val))
```

**Pitfall:** Confusing traversal orders; not handling edge cases (single node, null).

---

#### 7. Heaps & Priority Queues

**Concept:**  
Min/max heap for top-K, streaming median. Heap insertion/removal O(log n).

**Example:**
```python
import heapq

# Top K frequent (min-heap of size k)
def top_k_frequent(arr, k):
    count = Counter(arr)
    heap = []
    for num, freq in count.items():
        heapq.heappush(heap, (freq, num))
        if len(heap) > k:
            heapq.heappop(heap)
    return [x[1] for x in heap]

# Streaming median
class MedianFinder:
    def __init__(self):
        self.small = []  # max heap (negate for Python min-heap)
        self.large = []  # min heap
    
    def addNum(self, num):
        heapq.heappush(self.small, -num)
        if self.small and self.large and (-self.small[0] > self.large[0]):
            val = -heapq.heappop(self.small)
            heapq.heappush(self.large, val)
        if len(self.small) > len(self.large) + 1:
            val = -heapq.heappop(self.small)
            heapq.heappush(self.large, val)
    
    def findMedian(self):
        if len(self.small) > len(self.large):
            return float(-self.small[0])
        return (-self.small[0] + self.large[0]) / 2.0
```

**Pitfall:** Python heapq is min-heap; negate for max-heap. Heap property maintained after pop/push.

---

#### 8. Graphs

**Concept:**  
Representations: adjacency list (sparse), adjacency matrix (dense). BFS, DFS, topological sort, Dijkstra, Union-Find.

**Example:**
```python
# Adjacency list + BFS
def bfs(graph, start):
    visited = set()
    q = deque([start])
    visited.add(start)
    while q:
        node = q.popleft()
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                q.append(neighbor)
    return visited

# DFS (iterative)
def dfs(graph, start):
    visited = set()
    stack = [start]
    while stack:
        node = stack.pop()
        if node not in visited:
            visited.add(node)
            for neighbor in graph[node]:
                if neighbor not in visited:
                    stack.append(neighbor)
    return visited

# Topological sort (Kahn's algorithm, BFS)
def topo_sort(graph, in_degree):
    q = deque([node for node in graph if in_degree[node] == 0])
    result = []
    while q:
        node = q.popleft()
        result.append(node)
        for neighbor in graph[node]:
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0:
                q.append(neighbor)
    return result if len(result) == len(graph) else []

# Union-Find
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
    
    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
    
    def union(self, x, y):
        px, py = self.find(x), self.find(y)
        if px == py:
            return False
        if self.rank[px] < self.rank[py]:
            px, py = py, px
        self.parent[py] = px
        if self.rank[px] == self.rank[py]:
            self.rank[px] += 1
        return True
```

**Pitfall:** Graph as adjacency list—off-by-one in node indexing. Dijkstra requires non-negative weights.

---

#### 9. Recursion & Backtracking

**Concept:**  
Call stack + base case. Backtracking explores all branches; prune early.

**Example:**
```python
# Subsets (backtrack all combinations)
def subsets(nums):
    result = []
    def backtrack(start, path):
        result.append(path[:])
        for i in range(start, len(nums)):
            path.append(nums[i])
            backtrack(i + 1, path)
            path.pop()
    backtrack(0, [])
    return result

# Permutations
def permutations(nums):
    result = []
    def backtrack(path, remaining):
        if not remaining:
            result.append(path)
        for i in range(len(remaining)):
            backtrack(path + [remaining[i]], remaining[:i] + remaining[i+1:])
    backtrack([], nums)
    return result

# N-Queens
def solve_n_queens(n):
    result = []
    def is_safe(board, row, col):
        for i in range(row):
            if board[i] == col or abs(board[i] - col) == abs(i - row):
                return False
        return True
    
    def backtrack(row, board):
        if row == n:
            result.append(board[:])
            return
        for col in range(n):
            if is_safe(board, row, col):
                board.append(col)
                backtrack(row + 1, board)
                board.pop()
    
    backtrack(0, [])
    return result
```

**Pitfall:** Not backtracking (not undoing state); exponential time—know when to prune.

---

#### 10. Dynamic Programming

**Concept:**  
Optimal substructure + overlapping subproblems. Memoization (top-down) or tabulation (bottom-up).

**Example:**
```python
# Fibonacci (memoization)
def fib(n, memo=None):
    if memo is None:
        memo = {}
    if n in memo:
        return memo[n]
    if n <= 1:
        return n
    memo[n] = fib(n - 1, memo) + fib(n - 2, memo)
    return memo[n]

# Knapsack (0/1, tabulation)
def knapsack(weights, values, capacity):
    n = len(weights)
    dp = [[0] * (capacity + 1) for _ in range(n + 1)]
    for i in range(1, n + 1):
        for w in range(capacity + 1):
            if weights[i-1] <= w:
                dp[i][w] = max(values[i-1] + dp[i-1][w-weights[i-1]], dp[i-1][w])
            else:
                dp[i][w] = dp[i-1][w]
    return dp[n][capacity]

# Longest increasing subsequence (LIS)
def lis(arr):
    n = len(arr)
    dp = [1] * n
    for i in range(1, n):
        for j in range(i):
            if arr[j] < arr[i]:
                dp[i] = max(dp[i], dp[j] + 1)
    return max(dp) if dp else 0

# Edit distance (Levenshtein)
def edit_distance(s1, s2):
    m, n = len(s1), len(s2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    for i in range(m + 1):
        dp[i][0] = i
    for j in range(n + 1):
        dp[0][j] = j
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if s1[i-1] == s2[j-1]:
                dp[i][j] = dp[i-1][j-1]
            else:
                dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])
    return dp[m][n]
```

**Pitfall:** Not recognizing DP problem; inefficient memoization (unhashable state); off-by-one in transitions.

---

#### 11. Binary Search & Parametric Search

**Concept:**  
Search in O(log n). Parametric: guess answer, check feasibility, binary search on answer space.

**Example:**
```python
# Classic binary search
def binary_search(arr, target):
    left, right = 0, len(arr) - 1
    while left <= right:
        mid = (left + right) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return -1

# Search in rotated sorted array
def search_rotated(arr, target):
    left, right = 0, len(arr) - 1
    while left <= right:
        mid = (left + right) // 2
        if arr[mid] == target:
            return mid
        if arr[left] <= arr[mid]:  # left half sorted
            if arr[left] <= target < arr[mid]:
                right = mid - 1
            else:
                left = mid + 1
        else:  # right half sorted
            if arr[mid] < target <= arr[right]:
                left = mid + 1
            else:
                right = mid - 1
    return -1

# Parametric: minimum speed to eat k bananas in h hours
def min_eating_speed(piles, h):
    def can_finish(speed):
        return sum((p + speed - 1) // speed for p in piles) <= h
    
    left, right = 1, max(piles)
    while left < right:
        mid = (left + right) // 2
        if can_finish(mid):
            right = mid
        else:
            left = mid + 1
    return left
```

**Pitfall:** Infinite loops (left = mid or right = mid without +1/-1). Off-by-one in answer range.

---

#### 12. Problem-Solving Framework

**Flow:**
1. **Clarify:** Ask about constraints, edge cases, input format.
2. **Examples:** Small and medium; trace through mentally.
3. **Brute force:** Correct but slow; state complexity.
4. **Optimize:** Spot pattern (two pointers, hash, DP, binary search). Reduce time/space.
5. **Code:** Write clean, handle edge cases.
6. **Test:** Trace examples; test edge cases, null, duplicates.
7. **Complexity:** State O(time) and O(space).

**Interview Narrative:**
"I'll clarify the problem, think through a brute-force approach, identify a pattern I can optimize, code it cleanly, and verify with examples."

---

#### 13. LeetCode Strategy

**Curated 150 (Blind 75 + extras):**
- **Two Pointers (6):** Two Sum II, Valid Palindrome, Container With Most Water, Trap Rain, 3Sum, 4Sum.
- **Hash (5):** Valid Anagram, Group Anagrams, Top K Frequent, Product Array Except Self, Valid Sudoku.
- **Stack (4):** Valid Parentheses, Daily Temperatures, Largest Rectangle in Histogram, Trapping Rain Water II.
- **Linked List (6):** Reverse List, Merge Two, Add Two Numbers, Cycle Detection, Reorder, Copy Random Pointer.
- **Trees (15):** Inorder/Preorder/Postorder, Level Order, Right View, Lowest Common Ancestor, Path Sum, Serialize/Deserialize, Binary Search Tree Validity, Balanced Tree, Flatten.
- **Graphs (8):** Number of Islands, Clone Graph, Course Schedule (topological), Alien Dictionary, Minimum Height Trees, Network Delay, Reconstruct Itinerary, Redundant Connection.
- **Heap (3):** Top K, Median, IPO.
- **Binary Search (6):** Search Sorted, Search Rotated, Median Sorted, Find First/Last, Search Insert Position.
- **DP (20):** Climb Stairs, House Robber, Best Time Buy Sell Stock, Longest Substring Without Repeats, Longest Increasing Subsequence, Coin Change, Word Break, Edit Distance, Regular Expression, Palindromic Substrings, Burst Balloons.
- **Backtracking (8):** Subsets, Permutations, Combinations, Word Search, N-Queens, Sudoku.
- **Math (4):** Majority Element, Missing Number, Counting Bits, Happy Number.
- **String (8):** Longest Palindrome, Substring Without Repeats, Minimum Window, Valid Palindrome, Anagrams, Wildcard Matching.
- **Other (12):** Union-Find, Interval Scheduling, Merge Intervals, Insert Interval, Meeting Rooms, Gas Station, Rotate Array, Trapping Rain Water.

**Pacing:** Week 13–18, solve 25 problems/week. Target 2 days per problem: understand pattern, code, test.

---

### Pillar 2: System Design

#### 17. 45-Minute Interview Flow

**Time allocation:**
- **0–5 min:** Requirements gathering. "Design a URL shortener." Ask: scale (QPS, storage)? Functional vs non-functional? Consistency vs availability trade-off?
- **5–15 min:** Back-of-envelope: QPS → requests/sec; storage: bytes/URL × URLs/day; bandwidth.
- **15–30 min:** High-level design. Draw: client → API → service → DB, cache, queue.
- **30–40 min:** Deep dive into bottleneck. If cache: write-through vs write-behind? If DB: sharding strategy? If queue: Kafka partitions?
- **40–45 min:** Trade-offs & closing. "What if QPS 10x? We'd shard DB, add more cache nodes, horizontal scaling."

**Narrative:** "Let me clarify requirements first, then estimate scale, sketch a high-level design, deep dive into the hard parts, and discuss trade-offs."

---

#### 18. Fundamentals

**Load Balancer:** Distribute traffic (round-robin, least connections). Health checks.

**DNS:** Maps domain → IP. TTL caching.

**CDN:** Edge servers cache content near users. Good for static assets, video streaming.

**Cache:** In-memory (Redis, Memcached).
- Write-through: write to DB, then cache. Slow but consistent.
- Write-behind (write-back): write to cache, async to DB. Fast but risk data loss.

**Database:**
- **SQL (ACID):** Relational, transactions, ACID guarantees. Vertical scaling easier.
- **NoSQL (BASE):** Horizontal scaling, eventual consistency, flexible schema.

**Queues:** Decouple services. Kafka (partitioned, durable), SQS (managed).

**Pub/Sub:** Topics + subscribers. Event-driven.

---

#### 19. SQL vs NoSQL & Sharding

**SQL choice:** Relationships, ACID, fixed schema. Twitter users table: users(id, email, created_at).

**NoSQL choice:** Massive scale, flexible schema, horizontal. Tweets: {id, user_id, text, timestamp}.

**Sharding:** Partition data by shard key (user_id, geo). Each shard holds subset.
- **Hash sharding:** hash(user_id) % num_shards. Even distribution, hard to rebalance.
- **Range sharding:** user_id 0–1M → shard 0, 1M–2M → shard 1. Easy rebalance, hot shards possible.

**Replication:** Master-slave (read replicas), peer-to-peer (Cassandra). Trade-off: consistency vs availability.

---

#### 20. Consistency Models & Idempotency

**Strong (Linearizability):** Every read sees latest write. Expensive (requires synchronous replication).

**Eventual:** Reads may lag writes; converge over time. Fast, scales horizontally.

**Idempotency:** Same request twice = same result. Use request ID, check DB before processing. Essential for retries.

**Retry strategy:** Exponential backoff, jitter. Avoid thundering herd.

---

#### 21. Resilience Patterns

**Rate limiting:** Token bucket, sliding window. Protect against abuse.

**Circuit breaker:** Fail-open when downstream is down, recover gradually.

**Bulkhead:** Isolate pools (threads, connections). One slow service doesn't drag all.

**Timeout:** Fail fast. Set reasonable timeout per SLA.

---

#### 22. Observability

**Metrics:** Latency (p50, p95, p99), throughput, error rate, saturation. Dashboards (Prometheus, Datadog).

**Logs:** Structured logs (JSON), centralized (ELK, Splunk). Helps debugging.

**Traces:** Request journey across services (Jaeger, Zipkin). Identify bottleneck.

**Golden signals:** Latency, traffic, errors, saturation. Monitor these.

---

#### 23. Case Studies (1-page writeup format)

**URL Shortener:**
```
Requirements: Shorten URL (short), redirect (long). 100K QPS, 1B URLs.
Capacity: 1B × 100B = 100GB. 100K QPS = 1M req/min.
Design:
  - API: POST /shorten (long_url → short_code), GET /:code → redirect.
  - Service: Generate short code (Base62, collision detection). Store in DB.
  - DB: <code, long_url, created_at>. Primary: code.
  - Cache: Redis, hot URLs. TTL 24h.
  - Sharding: hash(code) % shards. ~100M URLs/shard.
Scale: 10x QPS? Add cache nodes, read replicas, horizontal DB shards.
Trade-off: Collision detection (code generation) vs cache hit rate. Eventual consistency acceptable.
```

**News Feed (Social):**
```
Requirements: Post + fetch feed (recent posts from friends).
Fanout strategy: Push (write-time) vs Pull (read-time).
Push: On post, write to all followers' feed caches. Fast reads, expensive writes for popular users.
Pull: On feed read, query posts from friends. Slower reads, scalable writes.
Hybrid: Pull + cache. Fetch top posts, cache for 1h.
DB: Posts, follower_edges (user→friends), feed_cache (user→recent posts).
Sharding: user_id. Replicate for high-profile users.
```

---

#### 24. API Design

**REST:** HTTP methods (GET, POST, PUT, DELETE). Stateless, cacheable.

**GraphQL:** Client specifies fields. Reduces over-fetching.

**gRPC:** RPC (binary, multiplexing). Low latency.

**Versioning:** /v1/users, Accept: application/vnd.company.v2+json. Backward compatible.

**Pagination:** offset/limit or cursor. Cursor handles insertions mid-page.

**Idempotency key:** Retry-safe. POST with X-Idempotency-Key header.

---

#### 25. Storage Math (Back-of-Envelope)

**QPS:** Requests/sec. 100K QPS = 100,000 req/sec = 8.64B req/day.

**Storage:** Data/day. If avg 1KB per request, 8.64B × 1KB = 8.64TB/day. 1-year retention = 3PB.

**Bandwidth:** QPS × data/req. 100K × 1KB = 100MB/s = 0.1Gbps.

---

#### 26. Design for Testability (QA Lens)

In system design, think like a QA:
- **Hooks for testing:** Admin endpoints (clear DB, reset state), feature flags (canary rollout).
- **Metrics:** Expose internal metrics (request latency histogram, queue depth) for assertions.
- **Chaos engineering:** Can you survive shard failure? DB failover?
- **Load testing:** Can you sustain SLA under peak load?
- **Monitoring:** Can you detect issues in < 1min (MTTD)?

---

### Pillar 3: Behavioral

#### 27. STAR Framework

**STAR:** Situation, Task, Action, Result. Polish 7 stories:

1. **Biggest success:** Project outcome, metrics (30% faster, 99.9% uptime). Role & impact.
2. **Biggest failure:** What went wrong, what you learned, how you recovered.
3. **Conflict:** Disagreed with PM on feature scope. How resolved (data, empathy, compromise)?
4. **Leadership:** Mentored junior, led initiative, drove change.
5. **Ambiguity:** No clear spec or ownership. How did you define path?
6. **Tight deadline:** Prioritized, cut scope, communicated risk.
7. **Cross-team:** Collaborated with infra/product/design. How smoothed friction?

**Delivery:** ≤ 2 min each. Specific names (people, systems), numbers (impact), emotion (felt challenged, proud).

---

#### 28. Amazon Leadership Principles (if target is AWS/AMZN)

Common QA interviews use:
- **Customer Obsession:** Advocate for users; test edge cases they'd hit.
- **Ownership:** Drive quality end-to-end; don't blame others.
- **Invent & Simplify:** Propose new test strategy; simplify flaky test triage.
- **Are Right, A Lot:** Data-driven decisions (test coverage, bug trends).
- **Learn & Be Curious:** GenAI, new tools, competitor products.
- **Hire & Develop:** Mentor testing guild; raise bar.
- **High Standards:** "No, this feature isn't production-ready." Push back on low quality.
- **Think Big:** Test strategy scales to 10x users.
- **Bias for Action:** Ship, iterate, learn (vs. perfect planning).

---

#### 29. "Why This Company / Role" Template

Research:
- Company mission, recent funding, products, competitors.
- Role: team size, impact, tech stack (Python? Rust? Kubernetes?).
- Skill match: your QA/DP/Python background → their stack.

Narrative: "I'm drawn to [Company] because [mission/product]. The [Role] appeals because [impact]. I have [skill] that directly applies to [problem]. I see opportunity to [grow/contribute]."

Example: "Amazon's mission to delight customers motivates me. The SDET role on AWS CLI testing combines my automation background and Python skills. I'd focus on reducing test flakiness, which I've reduced by 60% at my last role."

---

#### 30. QA-Specific Behavioral

**Scenario:** "Tell about a time you argued against shipping a feature."

**Answer:** "When manager wanted to ship a payment feature without integration testing, I raised concerns: 'If this fails in prod, we lose customer trust and revenue. Can we delay 1 week to automate critical flows?' I showed test gaps. We compromised: shipped with feature flag, rolled out to 5% first. Caught 3 bugs in canary. That disciplined approach prevented a costly incident."

**Scenario:** "You inherit a codebase with 0% test coverage."

**Answer:** "I'd prioritize based on risk: highest-impact flows first (payments, auth, critical paths). Start with 20% coverage in 1 month (unit + integration). Propose test-review gates (new code must have tests). Socialize benefits with dev team (catch regressions early, fewer hotfixes). Show metrics: defect escape rate dropped 40%. Gradually raise bar."

---

#### 31. Closing Questions to Ask

- "How does the QA team fit into the dev cycle? Are there test gates?"
- "What's the current test coverage and automation health?"
- "How do you measure quality? What's success for this role in 6 months?"
- "What's your most critical product feature from a testing perspective?"
- "How do you handle flaky tests?"

---

### Pillar 4: QA / SDET-Specific Rounds

#### 32. Test-Case Design Techniques

**Equivalence Partitioning:** Input domain into classes; test 1 per class + boundary.
- Example: Age field (0–150). Classes: <0 (invalid), 0–150 (valid), >150 (invalid). Test −1, 0, 75, 150, 151.

**Boundary Value:** Test at & around boundaries (off-by-one).
- ATM withdrawal: min=0, max=500. Test: 0, 1, 499, 500, 501.

**State Transition:** States (e.g., order: New → Paid → Shipped → Delivered). Test transitions & invalid transitions.

**Decision Table:** Conditions & outcomes. ATM: account_exists? funds_sufficient? Test all 4 combinations (TT, TF, FT, FF).

**Error Guessing:** Experience-based (crashes, security, off-by-one, null pointers).

**Exploratory:** Unscripted testing. Try user workflows; find surprises.

---

#### 33. Test Strategy Questions

**Risk-Based Testing:** Prioritize tests on product risk. High-risk features get more coverage.

**Shift-Left:** Test earlier (unit, API, contract). Catch bugs in dev, not production.

**Test Quadrants (Agile Testing):**
- Q1: Unit, component tests (supports development).
- Q2: API, integration (supports product).
- Q3: Exploratory, usability (supports product).
- Q4: Performance, security (supports release).

**Test Pyramid:** Unit (many, fast, cheap) > Integration (fewer, slower) > E2E (few, slowest, expensive).

**Test Trophy:** Unit ≈ Integration > E2E. Shift focus to integration layer.

---

#### 34. Automation Framework Design

**Page Object Model (POM):**
```python
# page_objects/login_page.py
class LoginPage:
    def __init__(self, driver):
        self.driver = driver
    
    def enter_email(self, email):
        self.driver.find_element("id", "email").send_keys(email)
    
    def enter_password(self, pwd):
        self.driver.find_element("id", "password").send_keys(pwd)
    
    def click_login(self):
        self.driver.find_element("id", "login_btn").click()

# tests/test_login.py
def test_valid_login():
    page = LoginPage(driver)
    page.enter_email("user@example.com")
    page.enter_password("pwd123")
    page.click_login()
    assert "Dashboard" in driver.title
```

**Layered architecture:**
- **UI layer:** POM, xpaths, selectors.
- **Business layer:** Test flows (login, checkout, payment).
- **Data layer:** Test data, fixtures, DB setup.

**CI/CD integration:** pytest → GitHub Actions → HTML report + metrics.

**Flaky test policy:** Retry failed tests 2×. Log failures. Alert on >5% flakiness. Investigate root cause.

---

#### 35. Bug Reports

**Good bug report contains:**
- Title: "Login button disabled after 3 failed attempts."
- Steps: 1. Open app. 2. Enter wrong password 3 times. 3. Observe button state.
- Expected: "Retry button enabled after 5 min."
- Actual: "Button remains disabled."
- Evidence: screenshot, logs, browser console.
- Environment: OS, browser, app version.
- Severity: Critical (prod down) > High (feature broken) > Medium (workaround) > Low.
- Priority: When to fix (release blocker vs. nice-to-have).

---

#### 36. Performance, Security, Accessibility Coverage

**Performance:** Load time, memory, CPU. Tools: Lighthouse, JMeter, Chrome DevTools.
- Test SLA: "Page load < 2s on 4G, p95."

**Security:** OWASP Top 10 (injection, XSS, broken auth, CSRF). Tools: OWASP ZAP, Burp Suite.
- Test: SQL injection (`' OR '1'='1`), XSS (`<script>alert(1)</script>`), CORS misconfig.

**Accessibility:** WCAG 2.1. Screen reader compat, keyboard nav, color contrast. Tools: Axe, WAVE.
- Test: Tab through UI, use screen reader, check contrast (4.5:1 min).

---

#### 37. Metrics & KPIs

**MTTD (Mean Time To Detect):** Avg time from bug intro to detection. < 1 day is good.

**MTTR (Mean Time To Resolve):** Avg time from detection to fix. < 1 week for prod bugs.

**Defect density:** Bugs / 1000 LOC. Trending down = improving quality.

**Escape rate:** % of prod bugs missed in test. < 10% is strong.

**Test coverage:** % of code executed by tests. Aim: 70%+ for critical paths, 50%+ overall.

**Flakiness rate:** % of tests that fail randomly. Target: < 2%.

**Pitfall:** Vanity metrics (test count). Useful: escape rate, MTTD, MTTR.

---

#### 38. Live Coding (Test Authoring)

**Prompt:** "Write a test for a login function."

**Function signature:**
```python
def login(email: str, password: str) -> dict:
    # Returns {"success": bool, "user_id": int, "error": str}
    pass
```

**Answer:**
```python
import pytest

class TestLogin:
    def test_valid_login(self):
        result = login("user@example.com", "password123")
        assert result["success"] is True
        assert result["user_id"] is not None
    
    def test_invalid_password(self):
        result = login("user@example.com", "wrongpwd")
        assert result["success"] is False
        assert "incorrect" in result["error"].lower()
    
    def test_nonexistent_email(self):
        result = login("nonexistent@example.com", "anypassword")
        assert result["success"] is False
        assert "not found" in result["error"].lower()
    
    @pytest.mark.parametrize("email,pwd", [
        ("user@example.com", ""),
        ("", "password123"),
        ("invalid-email", "password123"),
    ])
    def test_invalid_input(self, email, pwd):
        result = login(email, pwd)
        assert result["success"] is False
```

**Key:** Test happy path, errors, edge cases, invalid inputs. Use parameterization. Clear assertions.

---

#### 39. Debugging Scenarios

**Intermittent failure:** "Test passes sometimes, fails randomly."
- Cause: Race condition, flaky timing, hidden dependency.
- Debug: Add logging, run 100 times, check logs, isolate. Use `pytest -v -s` to see output.

**Env-specific bug:** "Works on my machine, fails in CI."
- Cause: OS difference, env var, data, timing.
- Debug: Run exact CI command locally, check env vars, examine data setup.

**Slow test:** "Tests take 10 min to run."
- Debug: Profile (pytest-benchmark). Mock external calls. Parallelize.

---

### Pillar 5: AI-Era Interviews

#### 40. GenAI in Day-to-Day Work

**Example prompt wins:**
- "Generate 50 test cases for a login form based on risk matrix." → 30 min → 5 min.
- "Explain this bug in simpler terms for non-tech stakeholder." → Rephrased in seconds.
- "What edge cases am I missing for a file upload feature?" → Brainstorm, filter.

**Answer framework:** "I use Claude for ideation, test-case brainstorming, code generation drafts. I always review, adapt, and verify. Example: I prompted for test cases for payment flows, got 80% useful, manually refined and validated against business rules."

**Guardrails:** Don't trust GenAI 100%. Verify logic, security, compliance. Prompt for explanations, debate outputs.

---

#### 41. Testing GenAI Products

**Hallucination:** LLM generates plausible but false info. Test by fact-checking.

**Safety:** Jailbreaks, toxic output. Test with adversarial prompts.

**Evals (evaluation frameworks):** BLEU (language quality), ROUGE (summary quality), custom business metrics.

**Drift:** Model output degrades over time. Monitor quality metrics, retrain.

**Latency:** GenAI is slow. Test timeout behavior, fallback strategies.

**Cost:** GenAI API calls are expensive. Monitor token usage, cost/request.

**Testing approach:**
- Unit: isolated LLM calls (correct schema, non-null).
- Integration: LLM → app logic (relevant output, safety filter applied).
- E2E: user queries → LLM → UI (latency SLA, cost budget).
- Eval: semantic accuracy (did it answer the question?), safety (no jailbreak output).

---

#### 42. Portfolio Project (AI Showcase)

**From file 01 (Build a Capstone Project):** Create or enhance a project showing:
- **Problem:** Real world, user-facing.
- **GenAI integration:** LLM, embedding, retrieval, fine-tuning, or evals.
- **QA angle:** Test suite, metrics, failure modes documented.
- **Demo:** GitHub, live link, video walkthrough.

**Example:** "QA Assistant Bot"
- Use Claude API to generate test cases from requirements.
- UI: paste requirements, click "Generate," get test cases.
- QA: Test coverage (all risk levels), eval (are cases relevant?), latency (<2s), cost tracking.
- GitHub: Code, README, test suite, example outputs.

---

#### 43. "Will AI Take Your QA Job?" — Strong Answer

"AI will change QA, not eliminate it. Routine test case generation? AI handles in seconds. But testing is 80% judgment, design, risk assessment, uncovering unknowns. I'd use AI to amplify: generate cases, I refine & prioritize. What I won't do: blindly trust AI output. I'll stay valuable by understanding *why* tests matter, building judgment, and adapting to new tools. Companies need QA who can teach AI testers what quality means."

---

#### 44. Prompt Engineering in Interviews

**Live scenario:** "Engineer a prompt to generate security test cases for a web app."

**Answer:**
```
Prompt v1 (weak): "Generate security test cases for a web app."

Prompt v2 (better):
"You are a security test expert. Generate 10 security test cases 
for a web app login feature, covering OWASP Top 10. 
Format as: {test_case, attack_vector, expected_result, severity}.
Focus on: SQL injection, XSS, CSRF, broken auth, rate limiting."

(Then iterate based on response quality.)
```

**Key:** Context (role), specificity (login feature, OWASP), format (JSON), constraints (10 cases).

---

### Cross-Cutting: Resume, LinkedIn, Mocks, Offer Negotiation

#### 45. Resume That Passes Screens

**Rules:**
- 1 page (< 10 YOE).
- STAR bullets: "Led automation framework redesign, reducing flaky tests by 60% and cutting release cycle from 2 weeks to 5 days" vs. "Wrote tests."
- Keywords (ATS): automation, Python, Selenium, CI/CD, test strategy, regression, load testing.
- Metrics: 30% faster, 99.9% uptime, 50+ test cases, $100K saved.
- No typos, clean formatting.

**Example:**
```
EXPERIENCE

QA Engineer, TechCorp (2021–2023)
- Designed and led POM-based test automation framework (Python, Selenium); 
  reduced flaky tests by 60% and cut release time from 2 weeks to 5 days.
- Built CI/CD pipeline integration (GitHub Actions); enabled 10 releases/week 
  vs. 1 release/month, increasing team velocity.
- Drove shift-left testing initiative; 40% of bugs caught at unit/API layer, 
  reducing production escapes from 12% to 3%.
```

---

#### 46. LinkedIn & GitHub Portfolio Hygiene

**LinkedIn:**
- Headline: "QA Engineer | Python | Automation | Test Strategy | Hiring to grow your team."
- Summary: 3–4 lines, mission-driven. "I build test systems that catch bugs early, empower dev teams, and scale to millions of users."
- Featured: Pinned GitHub repos, Medium posts, projects.
- Endorsements: Python, Selenium, Automation, Testing.

**GitHub:**
- 2–3 pinned repos: capstone project, test framework, blog.
- README: problem, solution, how to run, metrics.
- Clean code: linting, docstrings.
- Recent commits (shows active).

---

#### 47. Mock Interview Routine

**Monthly goal:** 6 mocks (3 DS&A, 2 system design, 2 behavioral, 1 QA design).

**Using Pramp / interviewing.io / Exponent:**
- Book with peer or mentor. 45-min timed session.
- **Record yourself:** Review gaps (spoke too fast? Missed edge case?).
- **Rate yourself:** 1–5 on communication, problem-solving, code quality.
- **Iterate:** Next mock, apply one improvement (e.g., "Ask clarifying questions upfront").

**Self-recording workflow:**
1. Use Zoom or OBS to record screen + audio.
2. After mock, watch 10 min clip. Note: where did I falter?
3. Redo that section next week.

---

#### 48. Offer Negotiation Basics

**Research:** Levels.fyi, Blind, Comparably. Know market rate for role/level/location.

**Negotiation points:**
- Base salary (15–20% is reasonable ask).
- Bonus (% of base, target vs. minimum).
- Equity (4-year vest, 1-year cliff typical). Don't accept < 20% of your comp in equity.
- Signing bonus (negotiate if switching jobs, cover stock cliff).
- PTO, benefits, WFH policy.

**Script:** "I'm excited to join. My research shows $X is market rate for this role in this market. Can we align on base salary? I'm flexible on bonus/equity split."

---

## 4. Exercises

### Beginner Phase (Weeks 13–15): Foundation

**LeetCode (30 easy, 1 per day):**
- Two Sum, Valid Palindrome, Two Sum II, Container With Most Water.
- Valid Anagram, Valid Parentheses, Majority Element, Missing Number.
- Reverse List, Merge Lists, Climbing Stairs, House Robber.
- Inorder Traversal, Level Order, Same Tree, Symmetric Tree.
- Number of Islands, Clone Graph, Course Schedule, Word Ladder.
- Top K Frequent, Merge K Sorted Lists, Median Finder.
- Longest Palindrome, Longest Substring Without Repeats, Group Anagrams.
- ... (30 total)

**STAR Stories (1 per week, refine to ≤ 2 min):**
- Week 1: Biggest success. Record, replay, tighten.
- Week 2: Biggest failure. Vulnerability + recovery.
- Week 3: Conflict or tight deadline.

---

### Intermediate Phase (Weeks 16–18): Pattern Mastery

**LeetCode (40 medium, 2–3 per week):**
- **Trees:** Binary Tree Paths, Lowest Common Ancestor, Path Sum, Serialize/Deserialize.
- **Graphs:** Number of Islands, Clone Graph, Course Schedule, Alien Dictionary.
- **DP:** Longest Increasing Subsequence, Coin Change, House Robber II, Edit Distance.
- **Strings:** Longest Substring Without Repeats, Minimum Window, Word Break, Palindromic Substrings.
- **Other:** Kth Largest Element, Merge K Sorted Lists, Trapping Rain Water, Rotate Array.
- ... (40 total)

**System Design Drills (2 cases, 1-page writeup each, Week 19):**
1. **URL Shortener:** Requirements, capacity math, design (code generation, collision, sharding), trade-offs.
2. **Rate Limiter:** Requirements, token bucket, distributed rate limiting, edge cases.

**QA Test-Case Design (5 apps, Week 20):**
1. ATM: withdrawal, balance, PIN retry logic.
2. Elevator: floor requests, door opening/closing, weight limits.
3. Messaging: send, receive, read receipts, group chat edge cases.
4. Ride-share: matching, pricing, safety features, cancellation.
5. Search: autocomplete, filters, pagination, no results.

---

### Advanced Phase (Weeks 21–23): Mastery & Mocks

**LeetCode (30 hard):**
- Burst Balloons, Word Ladder II, Largest Rectangle in Histogram, Trapping Rain Water II.
- Merge K Sorted Lists, Regular Expression Matching, Wildcard Matching, Reconstruct Itinerary.
- ... (30 total)

**System Design Drills (4 cases, 1-page each):**
1. **Chat System:** 1-on-1 messages, groups, notifications, read status.
2. **News Feed:** fanout strategy (push vs. pull), caching, ranking.
3. **Notifications:** multi-channel (email, SMS, push), batching, retry.
4. **Ride-Share Matching:** availability, matching algorithm, real-time location, SLA.

**Mock Interviews (minimum 6 total):**
- 3 × DS&A (45 min each): Mix of medium & hard. Record & review.
- 2 × System Design (60 min each): Full scope (requirements → deep dive). Write 1-page summary post-mortem.
- 2 × Behavioral (30 min each): Company-specific scenarios (e.g., Amazon LP, Google culture).
- 1 × QA Design (30 min): Test-case design for a given system.

**Post-mortem template:**
```
Interviewer: [Name]
Date: [Date]
Topic: [DS&A / System Design / Behavioral]
Score: 1–5 (1=fail, 5=perfect)

Strengths:
- Asked clarifying questions upfront.
- Optimized from O(n²) to O(n log n).

Gaps:
- Forgot to handle edge case (empty array).
- Spent 20 min on sub-problem, rushed at end.

Next focus:
- Template: clarify, example, brute force, optimize, code, test.
```

---

### Capstone: 4-Week Interview Sprint (Weeks 21–24)

**Week 1 (DS&A + Behavioral Polish):**
- Solve 10 medium/hard problems (under 45-min timed).
- Polish 2 STAR stories to perfection. Practice delivery.
- Time: 20 hours.

**Week 2 (System Design + Automation Framework Deep-Dive):**
- Design 2 full systems (URL shortener, rate limiter). Time 45 min each, write 1-page summary.
- Read *Designing Data-Intensive Applications* Ch 1–3 (storage, replication, consensus).
- Build or enhance test automation framework (POM, CI integration, metrics).
- Time: 20 hours.

**Week 3 (Mock Interview Marathon):**
- Conduct 6 mocks (3 DS&A, 2 system design, 1 behavioral). Use Pramp or peer.
- Record each; review 10-min highlight.
- Post-mortem each. Iterate.
- Time: 20 hours.

**Week 4 (Final Polish + Company-Specific Prep):**
- Polish resume (1 page, STAR bullets, keywords).
- LinkedIn & GitHub audit (readability, pinned projects).
- Research target companies (3–5): mission, engineering culture, interview process.
- Practice 2 final mocks on company-specific scenarios.
- Write post-mortem & action plan for next rounds.
- Time: 20 hours.

**Capstone Deliverable:**
A 2–3 page interview preparation post-mortem documenting:
- Mock performance (scores, trends).
- Gaps closed vs. remaining.
- Confidence level per pillar (1–5).
- Top 3 things to improve before real interviews.
- Resources used, time invested, ROI.

---

## 5. Additional Reading & Resources

**Books:**
- *Cracking the Coding Interview* (McDowell) — gold standard.
- *Elements of Programming Interviews in Python* (Aziz et al.) — rigorous, many problems.
- *System Design Interview* Vol 1 & 2 (Alex Xu) — canonical case studies.
- *Designing Data-Intensive Applications* (Kleppmann) — deep foundations (storage, consistency, networks).
- *The Pragmatic Programmer* — mindset & practices.

**Online:**
- **NeetCode.io:** Curated 150 list (Blind 75 + extras), video walkthroughs, categorized by pattern.
- **The Tech Interview Handbook** (yangshun/Garrick): Checklists, frameworks, templates.
- **Gayle McDowell's YouTube:** Code walkthrough, interview tips.
- **Pramp, interviewing.io, Exponent:** Live mock interviews with peers/experts.

**QA-Specific:**
- **ISTQB CTFL (Certified Tester Foundation Level) + CT-AI:** Exam prep, test design, automation.
- **Ministry of Testing:** Podcast, essays, community.
- **TestingWithSagar, Tricentis blog, Reena's newsletters:** Modern testing perspectives.
- **Your capstone project (file 01):** Real-world GenAI + QA showcase.

---

## 6. Index

| Topic | Section | Time |
|-------|---------|------|
| Big-O Complexity | 1. Overview, 3.1 | 2h |
| Arrays & Strings | 3.2 | 4h |
| Hashing | 3.3 | 3h |
| Linked Lists | 3.4 | 3h |
| Stacks & Queues | 3.5 | 3h |
| Trees | 3.6 | 4h |
| Heaps & Priority Queues | 3.7 | 3h |
| Graphs (BFS, DFS, Topo, Union-Find) | 3.8 | 5h |
| Recursion & Backtracking | 3.9 | 4h |
| Dynamic Programming | 3.10 | 6h |
| Binary Search | 3.11 | 3h |
| Problem-Solving Framework | 3.12 | 2h |
| LeetCode Strategy & 150 List | 3.13 | 1h |
| System Design Intro (45-min flow) | 3.17 | 2h |
| Fundamentals (LB, DNS, CDN, cache, DB, queues) | 3.18 | 3h |
| SQL vs NoSQL, Sharding | 3.19 | 2h |
| Consistency, Idempotency, Retries | 3.20 | 2h |
| Resilience (Rate Limit, Circuit Breaker, Bulkhead) | 3.21 | 2h |
| Observability (Metrics, Logs, Traces) | 3.22 | 2h |
| Case Studies (URL Shortener, Feed, Chat, etc.) | 3.23 | 6h |
| API Design | 3.24 | 2h |
| Storage Math & Back-of-Envelope | 3.25 | 1h |
| Design for Testability (QA lens) | 3.26 | 1h |
| STAR Framework & Stories | 3.27 | 3h |
| Amazon Leadership Principles | 3.28 | 1h |
| "Why This Company" Prep | 3.29 | 1h |
| QA-Specific Behavioral | 3.30 | 1h |
| Closing Questions | 3.31 | 0.5h |
| Test-Case Design (partitioning, boundary, state, decision table) | 3.32 | 3h |
| Test Strategy (risk-based, shift-left, quadrants, pyramid) | 3.33 | 2h |
| Automation Framework (POM, layered, CI, flaky policy) | 3.34 | 4h |
| Bug Reports | 3.35 | 1h |
| Performance, Security, Accessibility Coverage | 3.36 | 3h |
| Metrics & KPIs | 3.37 | 1h |
| Live Coding (Test Authoring) | 3.38 | 2h |
| Debugging Scenarios | 3.39 | 2h |
| GenAI in Day-to-Day | 3.40 | 1h |
| Testing GenAI Products | 3.41 | 2h |
| Portfolio Project (AI Showcase) | 3.42 | 10h |
| "Will AI Take Your Job?" Answer | 3.43 | 1h |
| Prompt Engineering | 3.44 | 2h |
| Resume & LinkedIn | 3.45–46 | 2h |
| Mock Interview Routine | 3.47 | 3h |
| Offer Negotiation | 3.48 | 1h |

---

## 7. References

- McDowell, G. L. (2015). *Cracking the Coding Interview* (6th ed.). CareerCup.
- Aziz, A., Lee, T.-H., & Prakash, A. (2016). *Elements of Programming Interviews in Python*. EPI.
- Xu, A. (2020–2021). *System Design Interview* Vols. 1 & 2. Independently published.
- Kleppmann, M. (2017). *Designing Data-Intensive Applications*. O'Reilly.
- Hunt, A. & Thomas, D. (1999). *The Pragmatic Programmer*. Addison-Wesley.
- ISTQB. (2023). *Certified Tester Foundation Level Syllabus*. International Software Testing Qualifications Board.
- ISTQB. (2024). *Certified Tester AI Essentials Syllabus*. International Software Testing Qualifications Board.
- Yang, S., Garrick, T., & Contributors. (2023). *The Tech Interview Handbook*. Open-source. https://www.techinterviewhandbook.org/
- NeetCode. (2023). *NeetCode 150*. https://neetcode.io/
- McDowell, G. L. (YouTube). Cracking the Coding Interview videos. https://www.youtube.com/user/gaylemcd
- Pramp. Live coding interview platform. https://www.pramp.com/
- Interviewing.io. Live technical interview platform. https://interviewing.io/
- Exponent. Interview prep platform (formerly "technoracle"). https://www.exponent.com/

---

**Final Note for Govind:**

You've built a solid foundation in Python and QA (files 01–07). This subject stitches it all together: DS&A gets you past the whiteboard, system design shows you can architect at scale, behavioral + QA-specific drills land you the offer, and the AI-era content future-proofs you.

The 4-week capstone sprint (Weeks 21–24) is your final intensive. Mock interviews are your rehearsal stage—record, iterate, improve. By Month 6, you'll walk into real interviews with confidence, a portfolio story, and clean code.

Good luck. You've got this.

