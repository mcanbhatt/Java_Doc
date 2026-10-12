# DSA Interview Questions & Answers — Java Full Stack Developers

> **30 most important DSA topics** covering Arrays, Strings, LinkedList, Trees, Graphs, DP, and more.
> Each question includes a Java solution with explanation and time/space complexity.

---

## Table of Contents

1. [Arrays & Two Pointer](#1-arrays--two-pointer)
2. [Sliding Window](#2-sliding-window)
3. [Sorting & Searching](#3-sorting--searching)
4. [Strings](#4-strings)
5. [LinkedList](#5-linkedlist)
6. [Stack & Queue](#6-stack--queue)
7. [Trees & BST](#7-trees--bst)
8. [Graphs](#8-graphs)
9. [Dynamic Programming](#9-dynamic-programming)
10. [Recursion & Backtracking](#10-recursion--backtracking)

---

## 1. Arrays & Two Pointer

---

### Q1. Find two numbers that add up to a target (Two Sum)

**Problem:** Given an array of integers and a target, return indices of two numbers that add up to target.

**Approach:** Use a HashMap to store complement → index. O(n) time.

```java
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (map.containsKey(complement)) {
            return new int[]{map.get(complement), i};
        }
        map.put(nums[i], i);
    }
    return new int[]{};
}
```

- **Time:** O(n)
- **Space:** O(n)
- **Interview tip:** Always clarify — can array have duplicates? Can we use same element twice?

---

### Q2. Container with most water (Two Pointer)

**Problem:** Given heights array, find two lines that together with x-axis forms a container holding most water.

```java
public int maxArea(int[] height) {
    int left = 0, right = height.length - 1, max = 0;
    while (left < right) {
        int area = Math.min(height[left], height[right]) * (right - left);
        max = Math.max(max, area);
        if (height[left] < height[right]) left++;
        else right--;
    }
    return max;
}
```

- **Time:** O(n)
- **Space:** O(1)
- **Key insight:** Always move the pointer with the smaller height inward.

---

### Q3. Find the maximum subarray sum (Kadane's Algorithm)

**Problem:** Given an integer array, find the contiguous subarray with the largest sum.

```java
public int maxSubArray(int[] nums) {
    int maxSum = nums[0], currentSum = nums[0];
    for (int i = 1; i < nums.length; i++) {
        currentSum = Math.max(nums[i], currentSum + nums[i]);
        maxSum = Math.max(maxSum, currentSum);
    }
    return maxSum;
}
```

- **Time:** O(n)
- **Space:** O(1)
- **Common mistake:** Forgetting to initialise with `nums[0]` instead of 0 (handles all-negative arrays).

---

### Q4. Move all zeros to end (Two Pointer)

**Problem:** Move all zeros to end of array while maintaining order of non-zero elements.

```java
public void moveZeroes(int[] nums) {
    int insertPos = 0;
    for (int num : nums) {
        if (num != 0) nums[insertPos++] = num;
    }
    while (insertPos < nums.length) {
        nums[insertPos++] = 0;
    }
}
```

- **Time:** O(n)
- **Space:** O(1)

---

## 2. Sliding Window

---

### Q5. Longest substring without repeating characters

**Problem:** Find the length of the longest substring without repeating characters.

```java
public int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> map = new HashMap<>();
    int max = 0, left = 0;
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        if (map.containsKey(c)) {
            left = Math.max(left, map.get(c) + 1);
        }
        map.put(c, right);
        max = Math.max(max, right - left + 1);
    }
    return max;
}
```

- **Time:** O(n)
- **Space:** O(min(m, n)) where m = charset size
- **Interview tip:** Very common — practice explaining the window shrink logic clearly.

---

### Q6. Maximum sum subarray of size K

**Problem:** Find max sum of any contiguous subarray of size k.

```java
public int maxSumSubarray(int[] arr, int k) {
    int windowSum = 0, maxSum = 0;
    for (int i = 0; i < k; i++) windowSum += arr[i];
    maxSum = windowSum;
    for (int i = k; i < arr.length; i++) {
        windowSum += arr[i] - arr[i - k];
        maxSum = Math.max(maxSum, windowSum);
    }
    return maxSum;
}
```

- **Time:** O(n)
- **Space:** O(1)

---

## 3. Sorting & Searching

---

### Q7. Binary Search

**Problem:** Search for a target in a sorted array. Return index or -1.

```java
public int binarySearch(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2; // avoids integer overflow
        if (nums[mid] == target) return mid;
        else if (nums[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}
```

- **Time:** O(log n)
- **Space:** O(1)
- **Common mistake:** Using `(left + right) / 2` causes overflow for large arrays.

---

### Q8. Find first and last position of element in sorted array

**Problem:** Given sorted array, find starting and ending position of a given target.

```java
public int[] searchRange(int[] nums, int target) {
    return new int[]{findFirst(nums, target), findLast(nums, target)};
}

private int findFirst(int[] nums, int target) {
    int idx = -1, left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) { idx = mid; right = mid - 1; }
        else if (nums[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return idx;
}

private int findLast(int[] nums, int target) {
    int idx = -1, left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) { idx = mid; left = mid + 1; }
        else if (nums[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return idx;
}
```

- **Time:** O(log n)
- **Space:** O(1)

---

### Q9. Merge sort (important for understanding divide & conquer)

```java
public void mergeSort(int[] arr, int left, int right) {
    if (left < right) {
        int mid = left + (right - left) / 2;
        mergeSort(arr, left, mid);
        mergeSort(arr, mid + 1, right);
        merge(arr, left, mid, right);
    }
}

private void merge(int[] arr, int left, int mid, int right) {
    int n1 = mid - left + 1, n2 = right - mid;
    int[] L = Arrays.copyOfRange(arr, left, mid + 1);
    int[] R = Arrays.copyOfRange(arr, mid + 1, right + 1);
    int i = 0, j = 0, k = left;
    while (i < n1 && j < n2) arr[k++] = (L[i] <= R[j]) ? L[i++] : R[j++];
    while (i < n1) arr[k++] = L[i++];
    while (j < n2) arr[k++] = R[j++];
}
```

- **Time:** O(n log n)
- **Space:** O(n)

---

## 4. Strings

---

### Q10. Check if two strings are anagrams

**Problem:** Given two strings s and t, return true if t is an anagram of s.

```java
public boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;
    int[] count = new int[26];
    for (char c : s.toCharArray()) count[c - 'a']++;
    for (char c : t.toCharArray()) {
        if (--count[c - 'a'] < 0) return false;
    }
    return true;
}
```

- **Time:** O(n)
- **Space:** O(1) — fixed 26-char array

---

### Q11. Reverse words in a string

**Problem:** Given a string, reverse the order of the words.

```java
public String reverseWords(String s) {
    String[] words = s.trim().split("\\s+");
    StringBuilder sb = new StringBuilder();
    for (int i = words.length - 1; i >= 0; i--) {
        sb.append(words[i]);
        if (i > 0) sb.append(" ");
    }
    return sb.toString();
}
```

- **Time:** O(n)
- **Space:** O(n)

---

### Q12. Valid parentheses

**Problem:** Given string containing `()[]{}`, determine if the input string is valid.

```java
public boolean isValid(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    for (char c : s.toCharArray()) {
        if (c == '(' || c == '[' || c == '{') {
            stack.push(c);
        } else {
            if (stack.isEmpty()) return false;
            char top = stack.pop();
            if (c == ')' && top != '(') return false;
            if (c == ']' && top != '[') return false;
            if (c == '}' && top != '{') return false;
        }
    }
    return stack.isEmpty();
}
```

- **Time:** O(n)
- **Space:** O(n)

---

## 5. LinkedList

---

### Q13. Reverse a linked list

```java
public ListNode reverseList(ListNode head) {
    ListNode prev = null, curr = head;
    while (curr != null) {
        ListNode next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
    }
    return prev;
}
```

- **Time:** O(n)
- **Space:** O(1)
- **Interview tip:** Also know the recursive version.

---

### Q14. Detect cycle in linked list (Floyd's algorithm)

```java
public boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}
```

- **Time:** O(n)
- **Space:** O(1)
- **Key insight:** Slow moves 1 step, fast moves 2 — they meet only if there is a cycle.

---

### Q15. Merge two sorted linked lists

```java
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0), curr = dummy;
    while (l1 != null && l2 != null) {
        if (l1.val <= l2.val) { curr.next = l1; l1 = l1.next; }
        else { curr.next = l2; l2 = l2.next; }
        curr = curr.next;
    }
    curr.next = (l1 != null) ? l1 : l2;
    return dummy.next;
}
```

- **Time:** O(m + n)
- **Space:** O(1)

---

### Q16. Find middle of linked list

```java
public ListNode middleNode(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow;
}
```

- **Time:** O(n)
- **Space:** O(1)

---

## 6. Stack & Queue

---

### Q17. Next greater element

**Problem:** For each element in array, find the next greater element to its right.

```java
public int[] nextGreaterElement(int[] nums) {
    int n = nums.length;
    int[] result = new int[n];
    Arrays.fill(result, -1);
    Deque<Integer> stack = new ArrayDeque<>(); // stores indices
    for (int i = 0; i < n; i++) {
        while (!stack.isEmpty() && nums[i] > nums[stack.peek()]) {
            result[stack.pop()] = nums[i];
        }
        stack.push(i);
    }
    return result;
}
```

- **Time:** O(n)
- **Space:** O(n)

---

### Q18. Implement queue using two stacks

```java
class MyQueue {
    Deque<Integer> inStack = new ArrayDeque<>();
    Deque<Integer> outStack = new ArrayDeque<>();

    public void push(int x) { inStack.push(x); }

    public int pop() {
        if (outStack.isEmpty()) transfer();
        return outStack.pop();
    }

    public int peek() {
        if (outStack.isEmpty()) transfer();
        return outStack.peek();
    }

    private void transfer() {
        while (!inStack.isEmpty()) outStack.push(inStack.pop());
    }

    public boolean empty() { return inStack.isEmpty() && outStack.isEmpty(); }
}
```

- **Time:** O(1) amortized for all operations
- **Space:** O(n)

---

## 7. Trees & BST

---

### Q19. Level order traversal (BFS)

```java
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    while (!queue.isEmpty()) {
        int size = queue.size();
        List<Integer> level = new ArrayList<>();
        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();
            level.add(node.val);
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        result.add(level);
    }
    return result;
}
```

- **Time:** O(n)
- **Space:** O(n)

---

### Q20. Maximum depth of binary tree

```java
public int maxDepth(TreeNode root) {
    if (root == null) return 0;
    return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}
```

- **Time:** O(n)
- **Space:** O(h) — h = height of tree

---

### Q21. Validate Binary Search Tree

```java
public boolean isValidBST(TreeNode root) {
    return validate(root, Long.MIN_VALUE, Long.MAX_VALUE);
}

private boolean validate(TreeNode node, long min, long max) {
    if (node == null) return true;
    if (node.val <= min || node.val >= max) return false;
    return validate(node.left, min, node.val) &&
           validate(node.right, node.val, max);
}
```

- **Time:** O(n)
- **Space:** O(h)
- **Common mistake:** Comparing only with parent — must carry min/max bounds down.

---

### Q22. Lowest Common Ancestor (LCA) of BST

```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    while (root != null) {
        if (p.val < root.val && q.val < root.val) root = root.left;
        else if (p.val > root.val && q.val > root.val) root = root.right;
        else return root;
    }
    return null;
}
```

- **Time:** O(h)
- **Space:** O(1)

---

## 8. Graphs

---

### Q23. BFS traversal of graph

```java
public void bfs(int start, Map<Integer, List<Integer>> graph) {
    Set<Integer> visited = new HashSet<>();
    Queue<Integer> queue = new LinkedList<>();
    queue.offer(start);
    visited.add(start);
    while (!queue.isEmpty()) {
        int node = queue.poll();
        System.out.print(node + " ");
        for (int neighbor : graph.getOrDefault(node, new ArrayList<>())) {
            if (!visited.contains(neighbor)) {
                visited.add(neighbor);
                queue.offer(neighbor);
            }
        }
    }
}
```

- **Time:** O(V + E)
- **Space:** O(V)

---

### Q24. Number of islands (DFS on grid)

**Problem:** Given a 2D grid of '1' (land) and '0' (water), count the number of islands.

```java
public int numIslands(char[][] grid) {
    int count = 0;
    for (int i = 0; i < grid.length; i++) {
        for (int j = 0; j < grid[0].length; j++) {
            if (grid[i][j] == '1') {
                dfs(grid, i, j);
                count++;
            }
        }
    }
    return count;
}

private void dfs(char[][] grid, int i, int j) {
    if (i < 0 || i >= grid.length || j < 0 || j >= grid[0].length || grid[i][j] != '1') return;
    grid[i][j] = '0'; // mark visited
    dfs(grid, i+1, j); dfs(grid, i-1, j);
    dfs(grid, i, j+1); dfs(grid, i, j-1);
}
```

- **Time:** O(m × n)
- **Space:** O(m × n) — recursion stack

---

### Q25. Detect cycle in directed graph (DFS + recursion stack)

```java
public boolean hasCycle(int V, List<List<Integer>> adj) {
    boolean[] visited = new boolean[V];
    boolean[] recStack = new boolean[V];
    for (int i = 0; i < V; i++) {
        if (!visited[i] && dfs(i, adj, visited, recStack)) return true;
    }
    return false;
}

private boolean dfs(int node, List<List<Integer>> adj, boolean[] visited, boolean[] recStack) {
    visited[node] = recStack[node] = true;
    for (int neighbor : adj.get(node)) {
        if (!visited[neighbor] && dfs(neighbor, adj, visited, recStack)) return true;
        else if (recStack[neighbor]) return true;
    }
    recStack[node] = false;
    return false;
}
```

- **Time:** O(V + E)
- **Space:** O(V)

---

## 9. Dynamic Programming

---

### Q26. Fibonacci with memoization

```java
public int fib(int n) {
    int[] memo = new int[n + 1];
    Arrays.fill(memo, -1);
    return helper(n, memo);
}

private int helper(int n, int[] memo) {
    if (n <= 1) return n;
    if (memo[n] != -1) return memo[n];
    return memo[n] = helper(n-1, memo) + helper(n-2, memo);
}
```

- **Time:** O(n)
- **Space:** O(n)

---

### Q27. 0/1 Knapsack problem

**Problem:** Given weights and values of n items, find max value within weight capacity W.

```java
public int knapsack(int[] weights, int[] values, int W) {
    int n = weights.length;
    int[][] dp = new int[n + 1][W + 1];
    for (int i = 1; i <= n; i++) {
        for (int w = 0; w <= W; w++) {
            dp[i][w] = dp[i-1][w];
            if (weights[i-1] <= w) {
                dp[i][w] = Math.max(dp[i][w], values[i-1] + dp[i-1][w - weights[i-1]]);
            }
        }
    }
    return dp[n][W];
}
```

- **Time:** O(n × W)
- **Space:** O(n × W)

---

### Q28. Longest Common Subsequence (LCS)

```java
public int lcs(String s1, String s2) {
    int m = s1.length(), n = s2.length();
    int[][] dp = new int[m + 1][n + 1];
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (s1.charAt(i-1) == s2.charAt(j-1)) dp[i][j] = 1 + dp[i-1][j-1];
            else dp[i][j] = Math.max(dp[i-1][j], dp[i][j-1]);
        }
    }
    return dp[m][n];
}
```

- **Time:** O(m × n)
- **Space:** O(m × n)

---

### Q29. Coin change (minimum coins)

**Problem:** Given coins of different denominations and amount, find minimum number of coins needed.

```java
public int coinChange(int[] coins, int amount) {
    int[] dp = new int[amount + 1];
    Arrays.fill(dp, amount + 1);
    dp[0] = 0;
    for (int i = 1; i <= amount; i++) {
        for (int coin : coins) {
            if (coin <= i) dp[i] = Math.min(dp[i], dp[i - coin] + 1);
        }
    }
    return dp[amount] > amount ? -1 : dp[amount];
}
```

- **Time:** O(amount × coins)
- **Space:** O(amount)
- **Interview tip:** Initialise dp array with `amount + 1` as "infinity", not `Integer.MAX_VALUE` (to avoid overflow on +1).

---

## 10. Recursion & Backtracking

---

### Q30. Generate all subsets (Power set)

```java
public List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(nums, 0, new ArrayList<>(), result);
    return result;
}

private void backtrack(int[] nums, int start, List<Integer> current, List<List<Integer>> result) {
    result.add(new ArrayList<>(current));
    for (int i = start; i < nums.length; i++) {
        current.add(nums[i]);
        backtrack(nums, i + 1, current, result);
        current.remove(current.size() - 1); // backtrack
    }
}
```

- **Time:** O(2^n × n)
- **Space:** O(n) — recursion depth

---

## Quick Reference — Time & Space Complexity

| Algorithm | Time | Space |
|---|---|---|
| Two Sum (HashMap) | O(n) | O(n) |
| Binary Search | O(log n) | O(1) |
| Merge Sort | O(n log n) | O(n) |
| BFS / DFS (Graph) | O(V + E) | O(V) |
| DP — Knapsack | O(n × W) | O(n × W) |
| DP — LCS | O(m × n) | O(m × n) |
| DP — Coin Change | O(amount × coins) | O(amount) |
| Tree traversal | O(n) | O(h) |
| Kadane's (max subarray) | O(n) | O(1) |
| Sliding window | O(n) | O(1) |

---

## Top Interview Tips for Java Developers

1. **Always clarify constraints** before coding — array size, negative numbers, duplicates.
2. **Start with brute force**, state complexity, then optimise.
3. **Know Java built-ins** — `Arrays.sort()`, `Collections.sort()`, `PriorityQueue`, `Deque`.
4. **Use `Deque` instead of `Stack`** in Java — `ArrayDeque` is faster.
5. **Avoid integer overflow** — use `long` or `left + (right - left) / 2` for binary search mid.
6. **For graph problems** — clarify if directed/undirected, weighted/unweighted.
7. **DP pattern** — identify state, recurrence relation, base case. Always draw the dp table.
8. **Practice explaining** your thought process out loud — interviewers value communication as much as correctness.

---

*Prepared for Java bench developers — client interview preparation*
