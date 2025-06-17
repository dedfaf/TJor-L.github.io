---
title: Segment Tree
summary: A **segment tree** is a binary tree built over a contiguous array segment.
date: 2018-09-21
tags:
    - algorithms
    - oi
image:
  caption: 'Image credit: [**Unsplash**](https://unsplash.com)'
---

If you’re still asking this question the day before the Advanced division contest, you’ll miss out on a first prize; and if you’re chasing a first prize in the Standard division, this blog will waste 5–20 precious minutes of your life.

Those statements make it clear: the segment tree is a crucial data-structure stepping stone from novice to serious OI competitor. It’s powerful but not trivial—I personally had to study it about five times before I dared write this post.

For a seasoned OI contestant, the segment tree isn’t an algorithm, it’s a tool: it can reduce interval (“segment”) updates and queries from **O(N)** down to **O(log N)**.

Without further ado, this blog is divided into four parts:

1. **Concept Introduction**
2. **Simple Segment Tree (no push-down)**
3. **Interval +/− Updates and Queries**
4. **Interval ×/÷ Updates and Queries**

---

## Part 1: Concept Introduction

A **segment tree** is a binary tree built over a contiguous array segment.
For example, for an array of length 4, we visualize:

```
           [1…4]
          /     \
      [1…2]     [3…4]
      /   \     /   \
    [1]  [2] [3]   [4]
```

If we store **range sums**, then:

* The root’s value = sum of elements **1…4**
* Its left child = sum of **1…2**, right child = sum of **3…4**
* And so on.

A key property is:

> **node\[i].sum = node\[2*i].sum + node\[2*i+1].sum**

We represent each tree node in a struct:

```cpp
struct Node {
    int l, r;        // covered interval [l, r]
    long long sum;   // sum over that interval
};
Node tree[4 * MAXN];
int input[MAXN];     // the original array
```

Since in a binary heap representation children of index `i` are `2*i` and `2*i+1`, we can build the tree recursively:

```cpp
inline void build(int i, int l, int r) {
    tree[i].l = l;
    tree[i].r = r;
    if (l == r) {                // leaf node
        tree[i].sum = input[l];
        return;
    }
    int mid = (l + r) >> 1;
    build(i*2,     l,   mid);    // build left subtree
    build(i*2 + 1, mid+1, r);    // build right subtree
    tree[i].sum = tree[i*2].sum + tree[i*2+1].sum;
}
```

That’s how you construct a segment tree. You might ask: why allocate several times the size of the original array? Because soon we’ll put this “big” array to work—let’s move on.

---

## Part 2: Simple Segment Tree (no push-down)

### 2.1 Single-Point Update & Range Query

Now we get to the real power of segment trees: efficiently maintaining an array so you can query the sum over any sub-range, e.g. **4…67** in **1…100**.

A naive loop

```cpp
long long sum = 0;
for (int i = 4; i <= 67; i++)
    sum += a[i];
```

is too slow if you do it repeatedly. Instead, build a segment tree over **1…4** with values `[1,2,3,4]`:

```
           (1+2+3+4)=10
           /          \
       (1+2)=3      (3+4)=7
       /    \        /    \
      [1]   [2]    [3]    [4]
```

To query **1…3**, we:

1. Start at root `[1…4]`. Its left child `[1…2]` intersects our query, so recurse there.
2. `[1…2]` is fully contained in **1…3** → return 3.
3. Back at `[1…4]`, the right child `[3…4]` also intersects → recurse.
4. `[3…4]` isn’t fully contained, but its left child `[3…3]` is → return 3.
5. Sum = 3 + 3 = 6.

This takes **O(log N)** steps instead of **O(N)**.

#### Code: range‐sum query

```cpp
// Query sum over [l, r]
inline long long query(int i, int l, int r) {
    // completely inside
    if (tree[i].l >= l && tree[i].r <= r)
        return tree[i].sum;
    // completely outside
    if (tree[i].r < l || tree[i].l > r)
        return 0;
    long long s = 0;
    // left child intersects?
    if (tree[i*2].r >= l)
        s += query(i*2, l, r);
    // right child intersects?
    if (tree[i*2+1].l <= r)
        s += query(i*2+1, l, r);
    return s;
}
```

#### Code: single-point update

To add `k` at position `pos`, we walk down the tree to the leaf, update, then pull sums back up:

```cpp
inline void update(int i, int pos, int k) {
    if (tree[i].l == tree[i].r) {  // leaf
        tree[i].sum += k;
        return;
    }
    // descend to correct child
    if (pos <= tree[i*2].r)
        update(i*2, pos, k);
    else
        update(i*2+1, pos, k);
    // pull up
    tree[i].sum = tree[i*2].sum + tree[i*2+1].sum;
}
```

---

### 2.2 Range-Update & Point Query

If you want to add `k` over an entire range `[l…r]` but only query single points, you can “tag” whole intervals and accumulate tags on the path down:

```cpp
// add k to every element in [l, r]
inline void range_add(int i, int l, int r, int k) {
    if (tree[i].l >= l && tree[i].r <= r) {
        // tag this node
        tree[i].sum += k;  
        return;
    }
    if (tree[i*2].r >= l)
        range_add(i*2, l, r, k);
    if (tree[i*2+1].l <= r)
        range_add(i*2+1, l, r, k);
}
```

Then to query a single position `pos`, we walk down, summing all tags:

```cpp
long long ans = 0;
void point_query(int i, int pos) {
    ans += tree[i].sum;  // accumulate tags
    if (tree[i].l == tree[i].r) return;
    if (pos <= tree[i*2].r)
        point_query(i*2, pos);
    else
        point_query(i*2+1, pos);
}
```

---

By now we’ve covered the **basic** segment tree. You can also use it for range‐min, range‐max, interval coloring, etc. But its true elegance comes when you combine **range updates** with **range queries**—that’s when *lazy propagation* (push-down) enters the picture. If you’ve mastered this chapter and can write Fenwick-tree templates 1 & 2 from memory, proceed to Part 3!

---

## Part 3: Advanced Segment Tree (Range Updates & Range Queries)

If you simply merge the “range update, point query” and “point update, range query” versions, you’ll run into correctness issues. For example, on $1…4$, if you add 1 to $1…3$, you’d tag $1…2$ and $3…3$. Later querying $2…4$ would miss the tag on $3…4$.

The solution is **lazy propagation** (“push-down”): whenever you visit a node partially, you push its pending tag to its children so that every relevant segment sees the update.

### 3.1 Node Definition

```cpp
struct Node {
    int l, r;
    long long sum;  // interval sum
    long long lazy; // pending increment to push down
};
Node tree[4 * MAXN];
```

### 3.2 push\_down()

```cpp
void push_down(int i) {
    if (tree[i].lazy != 0) {
        long long d = tree[i].lazy;
        int lc = i<<1, rc = i<<1|1;
        int mid = (tree[i].l + tree[i].r) >> 1;

        // Left child covers [l … mid]
        tree[lc].sum  += d * (mid - tree[lc].l + 1);
        tree[lc].lazy += d;

        // Right child covers [mid+1 … r]
        tree[rc].sum  += d * (tree[rc].r - mid);
        tree[rc].lazy += d;

        tree[i].lazy = 0;
    }
}
```

### 3.3 Range Add

```cpp
// Add k to every element in [L, R]
void range_add(int i, int L, int R, int k) {
    if (L <= tree[i].l && tree[i].r <= R) {
        tree[i].sum  += 1LL * k * (tree[i].r - tree[i].l + 1);
        tree[i].lazy += k;
        return;
    }
    push_down(i);
    int mid = (tree[i].l + tree[i].r) >> 1;
    if (L <= mid)      range_add(i<<1,     L, R, k);
    if (R    >  mid)   range_add(i<<1|1,   L, R, k);
    tree[i].sum = tree[i<<1].sum + tree[i<<1|1].sum;
}
```

### 3.4 Range Query

```cpp
long long range_query(int i, int L, int R) {
    if (L <= tree[i].l && tree[i].r <= R)
        return tree[i].sum;
    push_down(i);
    long long ans = 0;
    int mid = (tree[i].l + tree[i].r) >> 1;
    if (L <= mid)    ans += range_query(i<<1,     L, R);
    if (R    >  mid) ans += range_query(i<<1|1,   L, R);
    return ans;
}
```

---

## Part 4: Multiplicative & √ (Sqrt) Segment Tree

### 4.1 Multiplicative Updates

When you need both **multiply** and **add** updates, you maintain two lazy tags:

* `mlz` for multiplicative factor
* `plz` for additive increment

On push-down you apply **multiply first**, then **add**:

```cpp
struct Node {
    int l, r;
    long long sum;
    long long mlz = 1, plz = 0;
};

void push_down(int i) {
    long long m = tree[i].mlz, p = tree[i].plz;
    int lc = i<<1, rc = i<<1|1;
    int mid = (tree[i].l + tree[i].r) >> 1;

    // Apply to left child
    tree[lc].sum  = (tree[lc].sum * m + p * (mid - tree[lc].l + 1)) % P;
    tree[lc].mlz  = (tree[lc].mlz * m) % P;
    tree[lc].plz  = (tree[lc].plz * m + p) % P;

    // Apply to right child
    tree[rc].sum  = (tree[rc].sum * m + p * (tree[rc].r - mid)) % P;
    tree[rc].mlz  = (tree[rc].mlz * m) % P;
    tree[rc].plz  = (tree[rc].plz * m + p) % P;

    tree[i].mlz = 1;
    tree[i].plz = 0;
}
```

The range‐multiply and range‐add functions follow the same skeleton as `range_add` above, just updating `mlz` or `plz` appropriately.

### 4.2 √ (Square‐Root) Updates

For operations like integer division or square‐root—where

$$
\bigl\lfloor(a+b)/k\bigr\rfloor \neq \lfloor a/k\rfloor + \lfloor b/k\rfloor
\quad\text{and}\quad
\sqrt{a} + \sqrt{b} \neq \sqrt{a + b},
$$

you cannot simply tag a whole segment. Instead:

1. **Maintain** `minv` and `maxv` in each node.
2. If for a node covering $l…r$,

   $$
     \bigl\lfloor\sqrt{\minv}\bigr\rfloor
     = \bigl\lfloor\sqrt{\maxv}\bigr\rfloor,
   $$

   then every element in that segment will floor‐sqrt to the same decrement. You can apply it **in one shot**.
3. Otherwise, **recurse**.

```cpp
void range_sqrt(int i, int L, int R) {
    if (tree[i].l >= L && tree[i].r <= R &&
        (long long)(sqrt(tree[i].maxv)) == (long long)(sqrt(tree[i].minv))) {
        long long d = tree[i].maxv - (long long)sqrt(tree[i].maxv);
        tree[i].sum  -= d * (tree[i].r - tree[i].l + 1);
        tree[i].minv -= d;
        tree[i].maxv -= d;
        tree[i].lazy += d;  // reuse lazy as a decrement tag
        return;
    }
    if (tree[i].r < L || tree[i].l > R) return;
    push_down(i);
    range_sqrt(i<<1,     L, R);
    range_sqrt(i<<1|1,   L, R);
    tree[i].sum  = tree[i<<1].sum  + tree[i<<1|1].sum;
    tree[i].minv = min(tree[i<<1].minv, tree[i<<1|1].minv);
    tree[i].maxv = max(tree[i<<1].maxv, tree[i<<1|1].maxv);
}
```

---

## Summary

* A **segment tree** turns many linear‐time interval operations into **O(log N)**.
* **Core ideas**:

  1. `build` → `point update`/`range update` → `point query`/`range query`.
  2. If you do **range updates + range queries**, add a **lazy** tag and implement `push_down`.
  3. For non‐linear updates (division, sqrt), use segment min/max to identify uniform segments.
* Node indexing: left child `i<<1`, right child `i<<1|1`.

OI is a long journey with endless learning—may segment trees accompany you all the way, and when you return you’ll still be as eager as ever.
