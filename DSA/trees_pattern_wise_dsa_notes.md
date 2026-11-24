# 🌳 DSA Handbook: Binary Trees & Binary Search Trees

### Pattern-wise Java Interview Notes

> **Goal:** Learn to recognize tree problem patterns, choose the right
> traversal/state, and implement reliable Java solutions---not just
> memorize individual answers.

**Language:** Java\
**Platforms:** GeeksforGeeks (GFG) and LeetCode\
**Notation:** `Node.data` is used for GFG-style examples; replace it
with `TreeNode.val` on LeetCode.

------------------------------------------------------------------------

# 🧭 How to Use These Notes

For every problem, follow this sequence:

1.  **Recognize the clue** in the question.
2.  Decide whether the problem needs **BFS, DFS, BST ordering, or
    construction**.
3.  Identify the information each recursive call must return.
4.  Decide whether state belongs in a return value, a parameter, or a
    shared structure.
5.  Dry-run a small tree before coding.
6.  Check null, single-node, skewed-tree, duplicate-value, and overflow
    cases.

Do not jump directly to code. In an interview, explain the approach
first.

------------------------------------------------------------------------

# 🗺️ Complete Trees Roadmap

  -------------------------------------------------------------------------
  \#                Pattern           Main technique    Typical problems
  ----------------- ----------------- ----------------- -------------------
  1                 Recursive         DFS, recursion    Height, node count,
                    foundations       contract          sum

  2                 Traversals        DFS orders, stack Inorder, preorder,
                                                        postorder

  3                 Level-wise        BFS, queue        Level order, zigzag
                    processing                          

  4                 Views and         BFS + horizontal  Left, right, top,
                    coordinates       distance          bottom, vertical

  5                 Tree structure    Postorder         Diameter, balanced
                                      information       tree, width

  6                 Root-to-node      DFS +             Paths, path sum,
                    paths             backtracking      ancestors

  7                 Downward path     Prefix sum +      Count paths with
                    sums              frequency map     sum K

  8                 Ancestors and     LCA, parent map,  LCA, distance K,
                    distance          BFS               burning tree

  9                 Build and         Traversal         Build tree,
                    transform         indices, pointers flatten, DLL

  10                BST operations    Ordering          Search, insert,
                                      invariant         delete

  11                BST order         Inorder, pruning  Kth
                    statistics                          smallest/largest,
                                                        floor/ceil

  12                Advanced tree     Multiple          Largest BST,
                    patterns          techniques        recover BST,
                                                        iterator
  -------------------------------------------------------------------------

------------------------------------------------------------------------

# 1. 🌱 Recursive Foundations

## 1.1 The recursive model

A binary tree is naturally recursive:

-   A node contains a value.
-   It may have a left subtree.
-   It may have a right subtree.
-   Each subtree is itself a binary tree.

``` text
             10
            /  \
           5    15
          / \     \
         2   7     20

       Tree(10)
       ├── Tree(5)
       │   ├── Tree(2)
       │   └── Tree(7)
       └── Tree(15)
           └── Tree(20)
```

A standard Java node:

``` java
class Node {
    int data;
    Node left, right;

    Node(int data) {
        this.data = data;
    }
}
```

LeetCode commonly uses:

``` java
class TreeNode {
    int val;
    TreeNode left, right;
}
```

## 1.2 The three questions to ask before writing recursion

For every recursive tree problem, ask:

1.  **What does one call represent?** Example: solve the subtree rooted
    at `root`.
2.  **What should the call return?** Example: subtree height, count,
    sum, or a boolean.
3.  **What is the base case?** Usually `root == null`, but the returned
    value depends on the problem.

### Example: Maximum height

**Definition:** Height measured in number of nodes on the longest
root-to-leaf path.

``` text
        1
       / \
      2   3
     /
    4

Height = 3 nodes
```

``` java
int height(Node root) {
    if (root == null) return 0;

    int leftHeight = height(root.left);
    int rightHeight = height(root.right);

    return 1 + Math.max(leftHeight, rightHeight);
}
```

**Thinking:** The height of a node is one (the current node) plus the
greater height returned by its children.

-   Time: `O(N)` --- each node is visited once.
-   Auxiliary space: `O(H)` --- recursion stack, where `H` is tree
    height.
-   A skewed tree can make `H = N`; a balanced tree has `H = O(log N)`.

> ⚠️ Height conventions differ. Some platforms count edges instead of
> nodes. Check the problem statement and adjust the base case
> accordingly.

## 1.3 Recursion return-value patterns

  -----------------------------------------------------------------------
  Required result                     Typical recursive return
  ----------------------------------- -----------------------------------
  Is a property true?                 `boolean`

  Height/depth                        `int`

  Number of nodes                     `int`

  Sum of values                       `long` or `int`

  Best candidate/subtree              Custom object or value

  Need to collect answers             Shared result list, with controlled
                                      mutation

  Need path-specific state            Parameter + backtracking
  -----------------------------------------------------------------------

### A critical Java concept: primitive parameters are copied

This does **not** update the caller's `count`:

``` java
void dfs(Node root, int count) {
    if (root == null) return;
    count++; // only changes this call's local copy
    dfs(root.left, count);
}
```

Better options:

**Option A --- return the count (usually clearest):**

``` java
int countNodes(Node root) {
    if (root == null) return 0;
    return 1 + countNodes(root.left) + countNodes(root.right);
}
```

**Option B --- mutable holder when shared accumulation is appropriate:**

``` java
void dfs(Node root, int[] count) {
    if (root == null) return;
    count[0]++;
    dfs(root.left, count);
    dfs(root.right, count);
}
```

**Option C --- instance field:** reset it in the public method before
every run. Be careful with repeated test cases and concurrent calls.

## 1.4 Preorder, inorder, postorder: where does the work happen?

``` text
             N
            / \
           L   R

Preorder:   N → L → R   (process before children)
Inorder:    L → N → R   (process between children)
Postorder:  L → R → N   (process after children)
```

A useful recognition rule:

-   Need to act on a node before exploring children → preorder often
    fits.
-   Need sorted values from a BST → inorder.
-   Need information calculated from both children → postorder often
    fits.

------------------------------------------------------------------------

# 2. 🔁 Traversal Patterns

Use this sample tree for the traversal outputs:

``` text
             1
           /   \
          2     3
         / \   / \
        4   5 6   7
```

  Traversal     Output
  ------------- -----------------
  Preorder      `1 2 4 5 3 6 7`
  Inorder       `4 2 5 1 6 3 7`
  Postorder     `4 5 2 6 7 3 1`
  Level order   `1 2 3 4 5 6 7`

## 2.1 Recursive DFS template

``` java
void preorder(Node root) {
    if (root == null) return;

    visit(root);
    preorder(root.left);
    preorder(root.right);
}

void inorder(Node root) {
    if (root == null) return;

    inorder(root.left);
    visit(root);
    inorder(root.right);
}

void postorder(Node root) {
    if (root == null) return;

    postorder(root.left);
    postorder(root.right);
    visit(root);
}
```

Replace `visit(root)` with the required operation, such as
`list.add(root.data)`.

## 2.2 Iterative inorder

**Pattern clue:** Simulate recursive left descent with a stack.

``` text
        1
       / \
      2   3
     /
    4

Push left chain: 1, 2, 4
Pop 4 → visit
Pop 2 → visit; move to its right
Pop 1 → visit; move to its right
```

``` java
ArrayList<Integer> inorder(Node root) {
    ArrayList<Integer> ans = new ArrayList<>();
    Stack<Node> st = new Stack<>();
    Node curr = root;

    while (curr != null || !st.isEmpty()) {
        while (curr != null) {
            st.push(curr);
            curr = curr.left;
        }

        curr = st.pop();
        ans.add(curr.data);
        curr = curr.right;
    }

    return ans;
}
```

## 2.3 Iterative preorder

Push right before left because a stack is LIFO: left must be processed
first.

``` java
ArrayList<Integer> preorder(Node root) {
    ArrayList<Integer> ans = new ArrayList<>();
    if (root == null) return ans;

    Stack<Node> st = new Stack<>();
    st.push(root);

    while (!st.isEmpty()) {
        Node curr = st.pop();
        ans.add(curr.data);

        if (curr.right != null) st.push(curr.right);
        if (curr.left != null) st.push(curr.left);
    }

    return ans;
}
```

## 2.4 Iterative postorder

One clear method uses two stacks:

``` java
ArrayList<Integer> postorder(Node root) {
    ArrayList<Integer> ans = new ArrayList<>();
    if (root == null) return ans;

    Stack<Node> s1 = new Stack<>();
    Stack<Node> s2 = new Stack<>();

    s1.push(root);

    while (!s1.isEmpty()) {
        Node curr = s1.pop();
        s2.push(curr);

        if (curr.left != null) s1.push(curr.left);
        if (curr.right != null) s1.push(curr.right);
    }

    while (!s2.isEmpty()) {
        ans.add(s2.pop().data);
    }

    return ans;
}
```

## 2.5 Traversal complexity

  Method              Time               Auxiliary space
  --------------- -------- -----------------------------
  Recursive DFS     `O(N)`                        `O(H)`
  Iterative DFS     `O(N)`             `O(H)` worst case
  BFS               `O(N)`   `O(W)` where W is max width

------------------------------------------------------------------------

# 3. 📶 Level Order / BFS Patterns

**Pattern clue:** The problem mentions levels, minimum number of moves
by level, nearest node, or processing nodes level by level.

## 3.1 Level order traversal

``` text
             1          Level 0
           /   \
          2     3       Level 1
         / \     \
        4   5     6      Level 2

Output: [1, 2, 3, 4, 5, 6]
```

``` java
ArrayList<Integer> levelOrder(Node root) {
    ArrayList<Integer> ans = new ArrayList<>();
    if (root == null) return ans;

    Queue<Node> q = new LinkedList<>();
    q.offer(root);

    while (!q.isEmpty()) {
        Node curr = q.poll();
        ans.add(curr.data);

        if (curr.left != null) q.offer(curr.left);
        if (curr.right != null) q.offer(curr.right);
    }

    return ans;
}
```

## 3.2 Level-by-level template

Use this when the output must preserve level boundaries.

``` java
ArrayList<ArrayList<Integer>> levelOrderByLevel(Node root) {
    ArrayList<ArrayList<Integer>> ans = new ArrayList<>();
    if (root == null) return ans;

    Queue<Node> q = new LinkedList<>();
    q.offer(root);

    while (!q.isEmpty()) {
        int size = q.size();
        ArrayList<Integer> level = new ArrayList<>();

        for (int i = 0; i < size; i++) {
            Node curr = q.poll();
            level.add(curr.data);

            if (curr.left != null) q.offer(curr.left);
            if (curr.right != null) q.offer(curr.right);
        }

        ans.add(level);
    }

    return ans;
}
```

**Important:** Capture `size = q.size()` before processing the current
level. Nodes added during this loop belong to the next level.

## 3.3 Left view and right view

At every level: - Left view = first node processed. - Right view = last
node processed.

``` java
ArrayList<Integer> leftView(Node root) {
    ArrayList<Integer> ans = new ArrayList<>();
    if (root == null) return ans;

    Queue<Node> q = new LinkedList<>();
    q.offer(root);

    while (!q.isEmpty()) {
        int size = q.size();

        for (int i = 0; i < size; i++) {
            Node curr = q.poll();

            if (i == 0) ans.add(curr.data);

            if (curr.left != null) q.offer(curr.left);
            if (curr.right != null) q.offer(curr.right);
        }
    }
    return ans;
}
```

For the right view, change `if (i == 0)` to `if (i == size - 1)`.

### DFS alternative for left view

``` java
void leftViewDfs(Node root, int level, ArrayList<Integer> ans) {
    if (root == null) return;

    if (level == ans.size()) {
        ans.add(root.data);
    }

    leftViewDfs(root.left, level + 1, ans);
    leftViewDfs(root.right, level + 1, ans);
}
```

Visit left before right. The first node reached at a depth is the
leftmost visible node under this traversal order.

## 3.4 Zigzag traversal

Alternate the direction of values at each level. BFS still discovers
nodes in normal left-to-right order; reverse the placement for alternate
levels.

``` java
ArrayList<ArrayList<Integer>> zigzag(Node root) {
    ArrayList<ArrayList<Integer>> ans = new ArrayList<>();
    if (root == null) return ans;

    Queue<Node> q = new LinkedList<>();
    q.offer(root);
    boolean leftToRight = true;

    while (!q.isEmpty()) {
        int size = q.size();
        ArrayList<Integer> level = new ArrayList<>(
            Collections.nCopies(size, 0)
        );

        for (int i = 0; i < size; i++) {
            Node curr = q.poll();
            int index = leftToRight ? i : size - 1 - i;
            level.set(index, curr.data);

            if (curr.left != null) q.offer(curr.left);
            if (curr.right != null) q.offer(curr.right);
        }

        ans.add(level);
        leftToRight = !leftToRight;
    }

    return ans;
}
```

------------------------------------------------------------------------

# 4. 👀 Tree Views and Coordinate Patterns

## 4.1 Horizontal distance (HD)

Assign the root HD `0`:

-   Left child: `HD - 1`
-   Right child: `HD + 1`

``` text
                 1 (0)
                /     \
           2 (-1)     3 (+1)
             \        /
            4 (0)   5 (0)

Columns:       -1      0      +1
```

This coordinate model powers top view, bottom view, and vertical order
traversal.

## 4.2 Top view

**Rule:** At each HD, keep the first node encountered in BFS.

``` java
class Pair {
    Node node;
    int hd;

    Pair(Node node, int hd) {
        this.node = node;
        this.hd = hd;
    }
}

ArrayList<Integer> topView(Node root) {
    ArrayList<Integer> ans = new ArrayList<>();
    if (root == null) return ans;

    TreeMap<Integer, Integer> map = new TreeMap<>();
    Queue<Pair> q = new LinkedList<>();
    q.offer(new Pair(root, 0));

    while (!q.isEmpty()) {
        Pair p = q.poll();

        map.putIfAbsent(p.hd, p.node.data);

        if (p.node.left != null)
            q.offer(new Pair(p.node.left, p.hd - 1));

        if (p.node.right != null)
            q.offer(new Pair(p.node.right, p.hd + 1));
    }

    ans.addAll(map.values());
    return ans;
}
```

**Why BFS?** It visits shallower nodes before deeper nodes. Therefore
the first node at a horizontal distance is visible from the top.

## 4.3 Bottom view

**Rule:** At each HD, overwrite the stored node as BFS visits nodes. The
last visited node at that HD becomes the bottom-view candidate.

``` java
ArrayList<Integer> bottomView(Node root) {
    ArrayList<Integer> ans = new ArrayList<>();
    if (root == null) return ans;

    TreeMap<Integer, Integer> map = new TreeMap<>();
    Queue<Pair> q = new LinkedList<>();
    q.offer(new Pair(root, 0));

    while (!q.isEmpty()) {
        Pair p = q.poll();
        map.put(p.hd, p.node.data);

        if (p.node.left != null)
            q.offer(new Pair(p.node.left, p.hd - 1));

        if (p.node.right != null)
            q.offer(new Pair(p.node.right, p.hd + 1));
    }

    ans.addAll(map.values());
    return ans;
}
```

**Tie warning:** Some problem statements specify how to resolve nodes at
equal depth and HD. Follow that exact rule.

## 4.4 Vertical order traversal

Store every node in its HD column. `TreeMap` sorts columns from left to
right.

``` java
ArrayList<Integer> verticalOrder(Node root) {
    ArrayList<Integer> ans = new ArrayList<>();
    if (root == null) return ans;

    TreeMap<Integer, ArrayList<Integer>> map = new TreeMap<>();
    Queue<Pair> q = new LinkedList<>();
    q.offer(new Pair(root, 0));

    while (!q.isEmpty()) {
        Pair p = q.poll();

        map.putIfAbsent(p.hd, new ArrayList<>());
        map.get(p.hd).add(p.node.data);

        if (p.node.left != null)
            q.offer(new Pair(p.node.left, p.hd - 1));

        if (p.node.right != null)
            q.offer(new Pair(p.node.right, p.hd + 1));
    }

    for (ArrayList<Integer> column : map.values()) {
        ans.addAll(column);
    }

    return ans;
}
```

### GFG vs LeetCode distinction

Do not assume all "Vertical Order Traversal" questions have identical
tie-breaking:

-   Classic GFG vertical order commonly uses BFS order within each
    column.
-   LeetCode 987 orders by column, then row, then value when row and
    column tie.

Read the exact specification before selecting the data structure and
sorting rules.

------------------------------------------------------------------------

# 5. 🧱 Height and Structural Patterns

These problems often use **postorder DFS**: solve left and right
subtrees first, then calculate the current node's result.

## 5.1 Diameter of a binary tree

Diameter = maximum number of edges on a path between any two nodes.

At each node:

`candidate diameter = left height + right height`

``` text
          1
         / \
        2   3
       / \
      4   5

Longest path: 4 → 2 → 1 → 3
Diameter = 3 edges
```

Efficient one-pass implementation:

``` java
int diameter = 0;

int height(Node root) {
    if (root == null) return 0;

    int lh = height(root.left);
    int rh = height(root.right);

    diameter = Math.max(diameter, lh + rh);
    return 1 + Math.max(lh, rh);
}
```

Reset `diameter = 0` before each public invocation. This version returns
diameter in edges. If the platform asks for nodes, use `lh + rh + 1`.

-   Time: `O(N)`
-   Stack: `O(H)`

## 5.2 Balanced binary tree

A tree is height-balanced if, at every node, the left and right subtree
heights differ by at most one.

Avoid recalculating height separately at every node; that can lead to
`O(N²)`.

``` java
int checkHeight(Node root) {
    if (root == null) return 0;

    int lh = checkHeight(root.left);
    if (lh == -1) return -1;

    int rh = checkHeight(root.right);
    if (rh == -1) return -1;

    if (Math.abs(lh - rh) > 1) return -1;

    return 1 + Math.max(lh, rh);
}

boolean isBalanced(Node root) {
    return checkHeight(root) != -1;
}
```

**Pattern:** Return height when valid; return `-1` as an early-failure
signal.

## 5.3 Maximum width

At each level, count the number of nodes. For variants that count gaps
between positions (as in LeetCode's maximum width), assign positional
indices to nodes and normalize indices at each level to limit overflow.

Do not confuse: - **Number of nodes at the widest level** - **Width
including gaps between the leftmost and rightmost non-null positions**

These are different problem definitions.

## 5.4 Same tree

``` java
boolean isSameTree(Node a, Node b) {
    if (a == null && b == null) return true;
    if (a == null || b == null) return false;
    if (a.data != b.data) return false;

    return isSameTree(a.left, b.left)
        && isSameTree(a.right, b.right);
}
```

## 5.5 Symmetric tree

Compare mirrored children:

``` java
boolean mirror(Node left, Node right) {
    if (left == null && right == null) return true;
    if (left == null || right == null) return false;
    if (left.data != right.data) return false;

    return mirror(left.left, right.right)
        && mirror(left.right, right.left);
}

boolean isSymmetric(Node root) {
    if (root == null) return true;
    return mirror(root.left, root.right);
}
```

## 5.6 Invert binary tree

Swap the actual child references, then recurse.

``` java
Node invert(Node root) {
    if (root == null) return null;

    Node temp = root.left;
    root.left = root.right;
    root.right = temp;

    invert(root.left);
    invert(root.right);

    return root;
}
```

------------------------------------------------------------------------

# 6. 🛤️ Root-to-Node and Path Patterns

## 6.1 Root-to-leaf paths

**Pattern:** DFS + path list + backtracking.

``` text
          1
         / \
        2   3
       / \
      4   5

Paths:
1 → 2 → 4
1 → 2 → 5
1 → 3
```

``` java
void collectPaths(Node root,
                  ArrayList<Integer> path,
                  ArrayList<ArrayList<Integer>> ans) {
    if (root == null) return;

    path.add(root.data);

    if (root.left == null && root.right == null) {
        ans.add(new ArrayList<>(path)); // copy current path
    } else {
        collectPaths(root.left, path, ans);
        collectPaths(root.right, path, ans);
    }

    path.remove(path.size() - 1); // backtrack
}
```

**Why copy?** `ans.add(path)` stores the same mutable list reference.
Later backtracking changes it. Store `new ArrayList<>(path)`.

## 6.2 Path sum

For a root-to-leaf target sum, carry the remaining target:

``` java
boolean hasPathSum(Node root, int target) {
    if (root == null) return false;

    if (root.left == null && root.right == null) {
        return target == root.data;
    }

    int remaining = target - root.data;

    return hasPathSum(root.left, remaining)
        || hasPathSum(root.right, remaining);
}
```

The leaf check matters: reaching the target at an internal node does not
necessarily complete a root-to-leaf path.

## 6.3 Sum of left leaves

A leaf is a left leaf only if it is the left child of its parent.

``` java
int sumOfLeftLeaves(Node root) {
    return sumLeft(root, false);
}

int sumLeft(Node root, boolean isLeft) {
    if (root == null) return 0;

    if (root.left == null && root.right == null) {
        return isLeft ? root.data : 0;
    }

    return sumLeft(root.left, true)
         + sumLeft(root.right, false);
}
```

## 6.4 Longest root-to-leaf path; tie-break by sum

At a leaf, compare: 1. Greater path length. 2. If lengths tie, greater
sum.

Use a fresh path state for each branch, or return a result object. Avoid
passing one mutable object to both children and letting one branch
corrupt the other's state.

## 6.5 Maximum path sum

This differs from root-to-leaf path sum: the best path can start and end
anywhere, but must be continuous.

At each node: - Best one-sided gain returned to parent = node value +
max(0, left gain, right gain). - Best path passing through current node
= node value + max(0, left gain) + max(0, right gain). - Update a global
maximum with the through-node value.

Use `long` if constraints can exceed the `int` range. Reset the global
maximum to a suitably small value for each test case.

------------------------------------------------------------------------

# 7. 🧮 Downward Path Sum K: Prefix Sum Pattern

**Problem clue:** Count all downward paths whose sum equals K. A path
may start and end anywhere, but it must move from parent to child.

A root-to-current path can contain multiple valid suffixes.

``` text
             10
            /  \
           5   -3
          / \    \
         3   2    11
        / \   \
       3  -2   1
```

### Prefix sum idea

If current root-to-node prefix sum is `S`, a previous prefix sum of
`S - K` means the path between those positions sums to K.

``` java
int countPaths(Node root, int k) {
    Map<Long, Integer> freq = new HashMap<>();
    freq.put(0L, 1);
    return dfsCount(root, 0L, k, freq);
}

int dfsCount(Node root, long sum, int k,
             Map<Long, Integer> freq) {
    if (root == null) return 0;

    sum += root.data;

    int count = freq.getOrDefault(sum - k, 0);

    freq.put(sum, freq.getOrDefault(sum, 0) + 1);

    count += dfsCount(root.left, sum, k, freq);
    count += dfsCount(root.right, sum, k, freq);

    // Backtrack: remove this prefix before exploring a sibling.
    freq.put(sum, freq.get(sum) - 1);

    return count;
}
```

**Why backtrack the map?** A prefix from the left branch must not be
visible while exploring the right branch. The map represents only
prefixes on the current root-to-node path.

-   Average time: `O(N)`
-   Auxiliary space: `O(H)` active prefixes, up to `O(N)` in a skewed
    tree.

------------------------------------------------------------------------

# 8. 🧑‍🤝‍🧑 LCA, Ancestors and Distance

## 8.1 Lowest Common Ancestor in a binary tree

LCA is the lowest node that is an ancestor of both target nodes.

``` java
Node lca(Node root, Node p, Node q) {
    if (root == null || root == p || root == q) return root;

    Node left = lca(root.left, p, q);
    Node right = lca(root.right, p, q);

    if (left != null && right != null) return root;

    return left != null ? left : right;
}
```

This standard form assumes the problem's existence guarantees are
satisfied. If either target may be absent, additionally verify that both
targets were found.

## 8.2 LCA in a BST

Use the BST ordering property:

-   Both values smaller → go left.
-   Both values greater → go right.
-   Otherwise current node is the split point / one target.

``` java
Node lcaBST(Node root, int a, int b) {
    while (root != null) {
        if (a < root.data && b < root.data) {
            root = root.left;
        } else if (a > root.data && b > root.data) {
            root = root.right;
        } else {
            return root;
        }
    }
    return null;
}
```

## 8.3 Nodes at distance K / burning tree

A binary tree has child pointers but no parent pointers. To move in all
directions, first build a parent map.

1.  BFS/DFS from root to record `parent[child] = node`.
2.  Start BFS from the target.
3.  Explore left, right, and parent.
4.  Track visited nodes to avoid revisiting.
5.  After K BFS layers, collect the nodes.

**Pattern:** Convert the tree into an undirected graph using parent
references, then perform BFS.

------------------------------------------------------------------------

# 9. 🏗️ Tree Construction and Transformation

## 9.1 Build from preorder + inorder

Key facts: - Preorder's first unconsumed element is the subtree root. -
Inorder elements left of the root belong to the left subtree. - Inorder
elements right of the root belong to the right subtree.

``` text
Preorder: [1, 2, 4, 5, 3]
Inorder:  [4, 2, 5, 1, 3]

                 1
               /   \
              2     3
             / \
            4   5
```

Use a map for inorder positions to avoid repeatedly scanning the inorder
array.

``` java
int preIndex;
Map<Integer, Integer> position;

Node build(int[] preorder, int inLeft, int inRight) {
    if (inLeft > inRight) return null;

    int value = preorder[preIndex++];
    Node root = new Node(value);

    int mid = position.get(value);

    root.left = build(preorder, inLeft, mid - 1);
    root.right = build(preorder, mid + 1, inRight);

    return root;
}
```

Initialize `preIndex = 0` and populate `position` before calling
`build`. Reset state for every new construction. This standard method
assumes unique values.

-   Time: `O(N)` with the index map.
-   Space: `O(N)` for the map and recursion in the worst case.

## 9.2 Build from inorder + postorder

Postorder's last unconsumed value is the root. When consuming postorder
backwards, build **right first**, then left.

``` text
Postorder: [4, 5, 2, 3, 1]
Inorder:   [4, 2, 5, 1, 3]
```

Common bug: using `postIndex++` or building the left subtree first when
reading from the end.

## 9.3 Flatten binary tree to linked list

Target shape: preorder sequence, using only right pointers.

``` text
Before:             After:

       1                 1
      / \                 \
     2   5                 2
    / \   \                 \
   3   4   6                 3
                             \
                              4
                               \
                                5
                                 \
                                  6
```

A reverse-preorder recursion can keep a `prev` pointer: 1. Process right
subtree. 2. Process left subtree. 3. Set `root.right = prev`,
`root.left = null`. 4. Update `prev = root`.

Reset `prev` for each new tree.

------------------------------------------------------------------------

# 10. 🌲 Binary Search Tree (BST) Patterns

## 10.1 BST invariant

For a strict BST:

``` text
       8
      / \
     3   10
    / \    \
   1   6    14
      / \   /
     4   7 13
```

Every value in a node's left subtree is smaller; every value in its
right subtree is greater. This must hold for the **entire subtree**, not
merely the immediate children.

Duplicates require an explicit convention. Some problems disallow them;
others consistently place duplicates on one side.

## 10.2 Search

``` java
Node search(Node root, int key) {
    while (root != null && root.data != key) {
        root = key < root.data ? root.left : root.right;
    }
    return root;
}
```

-   Average balanced height: `O(log N)`
-   Worst-case skewed tree: `O(N)`

## 10.3 Insert

``` java
Node insert(Node root, int key) {
    if (root == null) return new Node(key);

    if (key < root.data) {
        root.left = insert(root.left, key);
    } else if (key > root.data) {
        root.right = insert(root.right, key);
    }

    return root;
}
```

This implementation ignores duplicates. Change the condition if the
problem defines another duplicate policy.

## 10.4 Delete

Three cases:

1.  **Leaf:** return `null`.
2.  **One child:** return the non-null child.
3.  **Two children:** replace value with inorder successor (minimum in
    right subtree), then delete that successor from the right subtree.

``` java
Node delete(Node root, int key) {
    if (root == null) return null;

    if (key < root.data) {
        root.left = delete(root.left, key);
    } else if (key > root.data) {
        root.right = delete(root.right, key);
    } else {
        if (root.left == null) return root.right;
        if (root.right == null) return root.left;

        Node successor = root.right;
        while (successor.left != null) {
            successor = successor.left;
        }

        root.data = successor.data;
        root.right = delete(root.right, successor.data);
    }

    return root;
}
```

**Common bug:** Calling `delete(root.left, key)` without assigning the
result back to `root.left`. The parent link must be updated.

## 10.5 Validate BST

Use a valid range for every node, not just parent-child comparisons.

``` java
boolean isValidBST(Node root) {
    return valid(root, Long.MIN_VALUE, Long.MAX_VALUE);
}

boolean valid(Node root, long low, long high) {
    if (root == null) return true;

    if (root.data <= low || root.data >= high) return false;

    return valid(root.left, low, root.data)
        && valid(root.right, root.data, high);
}
```

Strict inequalities enforce no duplicates. Adapt bounds if duplicates
are allowed.

## 10.6 Inorder gives sorted order

For a BST, inorder traversal returns values in ascending order (strictly
ascending when keys are unique).

This unlocks: - Kth smallest / largest - Predecessor / successor - Floor
/ ceil - Two-sum variants - Recover swapped BST

## 10.7 Kth smallest

Use inorder and stop when the visit count reaches K.

``` java
int count;
int answer;

void kth(Node root, int k) {
    if (root == null || answer != -1) return;

    kth(root.left, k);

    if (answer != -1) return;

    count++;
    if (count == k) {
        answer = root.data;
        return;
    }

    kth(root.right, k);
}

int kthSmallest(Node root, int k) {
    count = 0;
    answer = -1;
    kth(root, k);
    return answer;
}
```

This example assumes `-1` is a safe "not found" sentinel. Prefer a
separate boolean or optional result if `-1` can be a valid key.

## 10.8 Predecessor and successor

For a key: - Predecessor = greatest value smaller than key. - Successor
= smallest value greater than key.

During search: - If current value is greater than key, it may be the
successor; move left. - If current value is smaller than key, it may be
the predecessor; move right.

If key exists: - Predecessor is rightmost node of left subtree. -
Successor is leftmost node of right subtree.

## 10.9 Floor and ceil

-   Floor = greatest value `<= key`.
-   Ceil = smallest value `>= key`.

Track the best candidate while moving down the BST. If current equals
key, both are the key.

## 10.10 Two Sum in BST

Two common strategies:

-   Inorder into sorted array + two pointers: `O(N)` time and `O(N)`
    space.
-   BST iterator pair (ascending and descending): `O(N)` worst-case time
    and `O(H)` stack space.

------------------------------------------------------------------------

# 11. 🧠 Pattern Recognition Cheat Sheet

  -----------------------------------------------------------------------------
  Problem clue          Pattern           Data structure /  Key idea
                                          state             
  --------------------- ----------------- ----------------- -------------------
  Visit every node in a DFS traversal     Recursion / stack Choose preorder,
  chosen order                                              inorder, postorder

  Process one depth at  BFS               Queue             Capture queue size
  a time                                                    per level

  First or last node    BFS view          Queue             Pick first/last in
  per level                                                 each level

  First node at each    DFS view          Level + result    Visit preferred
  depth                                   size              side first

  Top/bottom/vertical   Coordinate BFS    TreeMap + queue   Horizontal distance
  columns                                                   

  Need both subtree     Postorder DFS     Return height     Combine left and
  heights                                                   right

  Root-to-leaf sequence DFS +             Mutable path      Add, recurse,
                        backtracking                        remove

  Count any downward    Prefix sum        HashMap frequency Find
  path sum                                                  `prefix - target`

  Need parent movement  Parent mapping    HashMap + BFS     Treat tree as
                                                            undirected graph

  Need sorted BST       Inorder           Recursion / stack BST inorder is
  values                                                    sorted

  Delete BST node       BST ordering      Recursive links   Handle 0, 1, 2
                                                            children

  Rebuild from          Traversal indices HashMap + index   Identify root and
  traversals                                                split inorder

  Need subtree validity Postorder summary Custom result     Return multiple
  and size                                object            facts together
  -----------------------------------------------------------------------------

------------------------------------------------------------------------

# 12. ⚠️ Common Java Tree Bugs

  --------------------------------------------------------------------------------
  Mistake                 Why it fails            Better approach
  ----------------------- ----------------------- --------------------------------
  Forgetting              Null pointer exception  Define base case first
  `root == null`                                  

  Returning before        Children never get      Combine recursive results
  recursive calls         processed               

  Not assigning recursive Parent's child          `root.left = solve(root.left)`
  result                  reference stays         
                          unchanged               

  Incrementing primitive  Java passes a copy      Return count or use a mutable
  count parameter                                 holder

  Sharing one mutable     Sibling state leaks     Backtrack after recursion
  path across branches                            
  without undoing                                 

  Adding path list        All answers may         Add `new ArrayList<>(path)`
  directly to answers     reference same list     

  Reusing instance fields Old result leaks into   Reset fields in public method
  across test cases       new call                

  Building postorder tree Traversal index is      Build right before left
  left-first from the end consumed in wrong order 

  Checking only immediate Deeper descendants may  Use min/max bounds
  BST children            violate range           

  Using `HashMap` when    Output order is not     Use `TreeMap`
  ordered columns are     guaranteed              
  required                                        

  Confusing height in     Off-by-one answer       Follow platform definition
  nodes vs edges                                  

  Using `int` for large   Integer overflow        Use `long` when constraints
  path sums                                       require
  --------------------------------------------------------------------------------

------------------------------------------------------------------------

# 13. 🎯 Suggested Interview Question Practice Order

Use this as a progression, not a claim about any company's exact
interview frequency.

## Foundation

1.  Inorder Traversal
2.  Preorder Traversal
3.  Postorder Traversal
4.  Level Order Traversal
5.  Maximum Depth
6.  Same Tree
7.  Symmetric Tree
8.  Invert Binary Tree

## BFS and views

9.  Zigzag Level Order
10. Left View
11. Right View
12. Top View
13. Bottom View
14. Vertical Order Traversal
15. Boundary Traversal

## Structural and path problems

16. Diameter of Binary Tree
17. Balanced Binary Tree
18. Maximum Width
19. Root-to-Leaf Paths
20. Path Sum
21. Sum of Left Leaves
22. Longest Root-to-Leaf Path
23. Count Downward Paths With Sum K
24. Maximum Path Sum

## LCA and construction

25. Lowest Common Ancestor
26. Kth Ancestor
27. Nodes at Distance K
28. Burning Tree
29. Build from Inorder + Preorder
30. Build from Inorder + Postorder
31. Serialize and Deserialize
32. Flatten Binary Tree

## BST

33. Search in BST
34. Insert into BST
35. Delete Node in BST
36. Validate BST
37. Kth Smallest
38. Predecessor and Successor
39. Floor and Ceil
40. Two Sum in BST
41. Recover BST
42. Largest BST in Binary Tree
43. BST Iterator

------------------------------------------------------------------------

# 14. 📝 Interview Explanation Framework

When asked to solve a tree problem, communicate in this order:

**1. Clarify the requirement** - Are values unique? - Does the path need
to start at root? - Is the path required to end at a leaf? - Does height
count nodes or edges? - What should happen if a target node is missing?

**2. State the pattern** - "This is a level-wise problem, so BFS is
suitable." - "I need information from both children, so I will use
postorder DFS." - "Since this is a BST, I can use its ordering property
to eliminate one subtree."

**3. Define the recursive contract** - "For a node, this function
returns the height of its subtree." - "This function returns whether the
target exists below this node." - "This function returns the best
one-sided path sum that can be extended by the parent."

**4. Explain complexity** Mention both time and auxiliary space,
including recursion stack and data structures.

**5. Test edge cases** - Empty tree - Single node - Left-skewed tree -
Right-skewed tree - Duplicate values - Negative values - Missing
target - Very large values

------------------------------------------------------------------------

# 15. ✅ Final Revision Checklist

Before considering Trees complete, make sure you can:

-   [ ] Write recursive and iterative inorder, preorder, and postorder.
-   [ ] Implement BFS and level-by-level traversal without size
    mistakes.
-   [ ] Explain why BFS works for top view and level-based views.
-   [ ] Implement left/right views using both BFS and DFS.
-   [ ] Explain horizontal distance for top, bottom, and vertical views.
-   [ ] Solve diameter and balanced-tree problems in one postorder
    traversal.
-   [ ] Use DFS + backtracking for root-to-leaf paths.
-   [ ] Explain why mutable path results must be copied.
-   [ ] Count downward path sums using prefix sums and backtracking.
-   [ ] Solve LCA in both a general binary tree and a BST.
-   [ ] Build a tree from inorder + preorder and inorder + postorder.
-   [ ] Explain the BST invariant and validate using ranges.
-   [ ] Delete a BST node in all three structural cases.
-   [ ] Use inorder for sorted BST operations.
-   [ ] Reset state between independent test cases.
-   [ ] Dry-run recursive calls and explain the return value at every
    node.

------------------------------------------------------------------------

## Final Principle 🌟

**A tree question is rarely just a tree question. It is usually a
recognizable combination of a few patterns:**

-   DFS or BFS
-   Return information from subtrees
-   Track a path or prefix
-   Maintain parent/level/coordinate information
-   Exploit BST ordering
-   Reconstruct structure from traversal order

Master those patterns, and many seemingly different questions become
variations of a familiar solution.
