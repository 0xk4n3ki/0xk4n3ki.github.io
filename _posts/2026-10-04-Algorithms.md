---
title : "Algorithms - GATE CSE"
description: Quick look blog for GATE CSE algorithms course
author : k4n3ki
date : 2026-10-04 1:00:00 +530
categories: [GATE-CSE]
tags: [GATE, CSE, Algorithms]
image:
    path: # /assets/img/dockerTutorial/docker.png
    alt: Algorithms
---

> These are the notes of Course by Dr. Khaleel Ur Rahman Khan.

# Algorithms Design Strategies:
- Divide & Conquer (DandC)
- Greedy Method 
- Dynamic Programming (DP)


| Aspect | Divide & Conquer | Greedy Method | Dynamic Programming |
| ------- | -------- | -------- | ------------ |
| **Core Idea** | Split the problem into independent subproblems, solve each recursively, then combine the results | Build the solution step by step, always taking the locally best choice and never reconsidering it | Solve subproblems from smaller to larger, store the results, and build the optimal solution from them |
| **Subproblems** | Independent, with no overlap | Only one subproblem remains after each choice | Overlapping: the same subproblem is needed many times | 
| **Decisions** | Not-decision-based; it splits and combines | One irreversible decision per step | Tries all choices at each step and picks the best using stored results |
| **Key Property needed** | Problem can be divided and the answers combined | Greedy-choice property and optimal substructure | Optimal Substructure (principle of optimality) and overlapping subproblems |
| **Correctness** | Correct by construction | Correct only if the greedy choice is proven safe; otherwise it can give a wrong answer | Always optimal, since all possibilities are covered implicitly |
| **Direction** | Top-Down recursion | Iterative, usually with sorting or a heap | Bottom-up table or top-down with memoization |
| **Extra memory** | Recursion stack | Little | A table of subproblem results |
| **Typical time** | Often O(nlogn) | Often O(nlogn) | Polynomial, such as O($n^2$) or O(nW) |
| **Examples** | Merge sort, quick sort, binary sort, Strassen's matrix multiplication | Fractional Knapsack, job sequencing, Huffman coding, Optimal merge, Prim's, Kruskal's, Dijkstra | 0/1 knapsack, LCS, matrix chain multiplication, Floyd-Warshall, Bellman-Ford, OBST, TSP |
| **How to Identify** | The problem splits cleanly into Independent pieces of the same type, and the answers combine easily | A simple rule (largest ratio, smallest weight, earliest deadline) seems to work, and you can argue it never needs undoing | The problem asks for min/max/count over many choices, subproblems repeat, and the greedy rule fails on a counterexample | 



## A. Divide & Conquer (DandC)

The core idea of Divide and Conquer is to break a complex problem into smaller, independent subproblems of the same type, solve each subproblem recursively, and then combine their results to form the final solution.

recurrance equation for time complexity:
- Symmetric case:&emsp;&emsp; &emsp;   $T(n) = a*T(\frac{n}{b}) + g(n)$&emsp;&emsp;&emsp;&emsp; ; ${a>=1, b>1, g(n)>0}$
- Asymmetric case I:&emsp; &emsp; $T(n) = T(\alpha*n) + T((1-\alpha)*n) + g(n)$&emsp;&emsp;&emsp;&emsp; ; ${0<\alpha<1}$
- Asymmetric case II: e.g: $T(n) = T(\frac{n}{2}) + T(\frac{n}{4}) + g(n)$

> In DandC strategy, Divide is Mandatory but combine is optional.

Applications of DandC:

### 1. MaxMin

#### Problem: Procedure to find the maximum and minimum simulatneously in and array of size 'n'

<details>
<summary>C++ code</summary>

```cpp
pair<int, int> minmax(const vector<int>& arr, int low, int high){
    if(low == high){
        return {arr[low], arr[low]};
    }

    if(high == low+1){
        if(arr[low] < arr[high]){
            return {arr[low], arr[high]};
        }else{
            return {arr[high], arr[low]};
        }
    }

    int mid = low + (high-low)/2;
    pair<int, int> left = minmax(arr, low, mid);
    pair<int, int> right = minmax(arr, mid+1, high);

    return {min(left.first, right.first), max(left.second, right.second)};
}
```

</details>

performance of maxmin:
- Time Complexity: $T(n) = \frac{3n}{2} - 2$
- Space Complexity: $O(\log_2 n)$


### 2. Binary Search

#### Problem: The primary requirement is that the list of n elements must be in sorted order

<details>
<summary>C++ code for Iterative Binary Search</summary>

```cpp
int binarySearchIterative(const vector<int>&arr, int key){
    int low = 0, high = (int)arr.size()-1;

    while(low <= high){
        int mid = low + (high - low)/2;

        if(arr[mid] == key)
            return mid;
        else if(arr[mid] < key)
            low = mid+1;
        else:
            high = mid-1;
    }
    return -1;
}
```

</details>

<details>
<summary>C++ code for Recursive Binary Search</summary>

```cpp
int binarySearchRecursive(const vector<int>& arr, int low, int high, int key){
    if(low > high)
        return -1;

    int mid = low + (high - low)/2;

    if(arr[mid] == key)
        return mid;
    else if(arr[mid] < key)
        return binarySearchRecursive(arr, mid+1, high, key);
    else:
        return binarySearchRecursive(arr, low, mid-1, key);
}
```

</details>

Performance of Binary Search:
- Time Complexity: $T(n) = T(\frac{n}{2}) + a; => T(n) = O(\log_2 n)$
- Space Complexity: $O(\log_2 n)$


> In Binary Seach, there is no combine operation.

### 3. Merge Sort

#### Problem: Given two sorted lists $L_1(m)$ and $L_2(n)$, where m<=n, it is required to merge them into a single sorted array having (m+n) elements, using 2-way merging.

<details>
<summary>C++ code</summary>

```cpp
void merge(vector<int>& arr, int low, int mid, int high){
    vector<int> temp;
    temp.reserve(high-low+1);

    int i = low, j = mid + 1;

    while(i <= mid && j <=high){
        if(arr[i] <= arr[j])
            temp.push_back(arr[i++]);
        else
            temp.push_back(arr[j++]);
    }

    while(i <= mid) temp.push_back(arr[i++]);
    while(j <= high) temp.push_back(arr[j++]);

    for(int k=0; k < (int)temp.size(); k++)
        arr[low + k] = temp[k];
}

void mergeSort(vector<int>& arr, int low, int high){
    if(low >= high)
    return;

    int mid = low + (high - low)/2;
    mergeSort(arr, low, mid);
    mergeSort(arr, mid+1, high);
}
```

</details>


Performance of merge sort:
- Time Complexity: $T(n) = 2*T(\frac{n}{2}) + b*n; => T(n) = O(n*\log_2 n)$
- Space Complexity: O(n)

> The number of comparisons required to merge two sorted lists $L_1(n)$, $L_2(m)$ and n<=m, lies between n and (n-1+m).

The number of comparisons required in 2-way merge sort:

| n | min | max |
|-------|------|--------|
| $= 2^k$ | $\frac{n}{2}log_2 n$ | $nlog_2 n - n + 1$ |
| $!= 2^k$ | $\frac{n}{2}log_2 n$ | $n\lceil log_2 n \rceil - 2^{\lceil log_2 n \rceil} + 1$ |


### 4. Quick Sort (Partition exchange sort)

<details>
<summary>C++ code</summary>

```cpp
int partitionArr(vector<int> &arr, int low, int high){
    int pivot = arr[high];
    int i = low-1;

    for(int j=low; j<=high-1; j++){
        if(arr[j] < pivot){
            i++;
            swap(arr[i], arr[j]);
        }
    }
    swap(arr[i+1], arr[high]);
    return i+1;
}

void quickSort(vector<int>& arr, int low, int high){
    if(low >= high)
        return;

    int p = partition(arr, low, high);
    quickSort(arr, low, p-1);
    quickSort(arr, p+1, high);
}
```

</details>


Performance of Quick Sort:
- Time Complexity: 
    - Best Case: $T(n) = 2T(\frac{n}{2}) + O(n) => T(n) = O(nlog_2 n)$
    - Worst Case: $T(n) = T(n-1) + O(n) => T(n) = O(n^2)$
    - Average Case: $T(n) = O(nlog_2 n)$
- Space Complexity:
    - Best Case: $O(log_2 n)$
    - Worst Case: $O(n)$


### 5. Matrix Multiplication

#### Problem: multiply 2-square matrices of order n*n

##### a. Standard Divide and Conquer Approach
- Split matrices A and B into four blocks of size $\frac{n}{2}*\frac{n}{2}$. Requires 8 recursive multiplications of submatrices and 4 matrix additions of size $\frac{n}{2}*\frac{n}{2}$ (which take O($n^2$) time).
- Recurrence Relation: T(n) = 8T($\frac{n}{2}$) + O($n^2$)
- Time Complexity: O($n^3$) (same as naive algorithm)

<details>
<summary>C++ code</summary>

```cpp
typedef vector<vector<int>> Matrix;

Matrix add(const Matrix& A, const Matrix& B){
    int n = A.size();
    Matrix C(n, vector<int>(n));
    
    for(int i=0; i<n; i++){
        for(int j=0; j<n; j++){
            c[i][j] = A[i][j] + B[i][j];
        }
    }
    return C;
}

Matrix multiply(const Matrix& A, const Matrix& B){
    int n = A.size();

    if(n==1)
        return Matrix(1, vector<int>(1, A[0][0]*B[0][0]));

    int h = n/2;

    Matrix A11(h, vector<int>(h)), A12(h, vector<int>(h)),
            A21(h, vector<int>(h)), A22(h, vector<int>(h));
    Matrix B11(h, vector<int>(h)), B12(h, vector<int>(h)),
            B21(h, vector<int>(h)), B22(h, vector<int>(h));

    for(int i=0; i<h; i++){
        for(int j=0; j<h; j++){
            A11[i][j] = A[i][j];
            A12[i][j] = A[i][j+h];
            A21[i][j] = A[i+h][j];
            A22[i][j] = A[i+h][j+h];

            B11[i][j] = B[i][j];
            B12[i][j] = B[i][j+h];
            B21[i][j] = B[i+h][j];
            B22[i][j] = B[i+h][j+h];
        }
    }

    Matrix C11 = add(multiply(A11, B11), multiply(A12, B21));
    Matrix C12 = add(multiply(A11, B12), multiply(A12, B22));
    Matrix C21 = add(multiply(A21, B11), multiply(A22, B21));
    Matrix C22 = add(multiply(A21, B12), multiply(A22, B22));

    Matrix C(n, vector<int>(n));
    for(int i=0; i<h; i++){
        for(int j=0; j<h; j++){
            C[i][j] = C11[i][j];
            C[i][j+h] = C12[i][j];
            C[i+j][j] = C21[i][j];
            C[i+h][j+h] = C22[i][j];
        }
    }
    return C;
}
```

</details>

##### b. Strassen's Matrix Multiplication
- An improvement over the standard Divide and Conquer method designed to reduce the number of recursive multiplication calls. 
- Reduces the number of recursive multiplication from 8 down to 7 by cleverly calculating 7 intermediate products ($P_1 to P_7$) using additions and subtractions.
- Recurrence relation: T(n) = 7T($\frac{n}{2}$) + O($n^2$)
- Time Complexity: O($n^{log_2 7}$) = O($n^{2.8074}$), which is faster than O($n^3$).

<details>
<summary>C++ code</summary>

```cpp
typedef vector<vector<int>> Matrix;

Matrix add(const Matrix& A, const Matrix& B){
    int n = A.size();
    Matrix C(n, vector<int>(n));
    for(int i=0; i<n; i++){
        for(int j=0; j<n; j++){
            C[i][j] = A[i][j] + B[i][j];
        }
    }
    return C;
}

Matrix sub(const Matrix& A, const Matrix& B){
    int n = A.size();
    Matrix C(n, vector<int>(n));
    for(int i=0; i<n; i++){
        for(int j=0; j<n; j++){
            C[i][j] = A[i][j] - B[i][j];
        }
    }
    return C;
}

Matrix strassen(const Matrix& A, const Matrix& B){
    int n = A.size();

    if(n==1)
        return Matrix(1, vector<int>(1, A[0][0]*B[0][0]));

    int h = n/2;
    Matrix A11(h, vector<int>(h)), A12(h, vector<int>(h)),
            A21(h, vector<int>(h)), A22(h, vector<int>(h));
    Matrix B11(h, vector<int>(h)), B12(h, vector<int>(h)),
            B21(h, vector<int>(h)), B22(h, vector<int>(h));

    for(int i=0; i<h; i++){
        for(int j=0; j<h; j++){
            A11[i][j] = A[i][j];
            A12[i][j] = A[i][j+h];
            A21[i][j] = A[i+h][j];
            A22[i][j] = A[i+h][j+h];

            B11[i][j] = B[i][j];
            B12[i][j] = B[i][j+h];
            B21[i][j] = B[i+h][j];
            B22[i][j] = B[i+h][j+h];
        }
    }

    Matrix P1 = strassen(A11, sub(B12, B22));
    Matrix P2 = strassen(add(A11, A12), B22);
    Matrix P3 = strassen(add(A21, A22), B11);
    Matrix P4 = strassen(A22, sub(B21, B11));
    Matrix P5 = strassen(add(A11, A22), add(B11, B22));
    Matrix P6 = strassen(sub(A12, A22), add(B21, B22));
    Matrix P7 = strassen(sub(A11, A21), add(B11, B12));

    Matrix C11 = add(sub(add(P5, P4), P2), P6);
    Matrix C12 = add(P1, P2);
    Matrix C21 = add(P3, P4);
    Matrix C22 = sub(sub(add(P5, P1), P3), P7);

    Matrix C(n, vector<int>(n));
    for(int i=0; i<h; i++){
        for(int j=0; j<h; j++){
            C[i][j] = C11[i][j];
            C[i][j+h] = C12[i][j];
            C[i+h][j] = C21[i][j];
            C[i+h][j+h] = C22[i][j];
        }
    }
    return C;
}
```

</details>


### 6. Long Integer Multiplication

#### To efficiently compute the product of two extremely large, arbitrary-precision integers containing n digits

##### a. Standard Divide and Conquer Approach
- Divide two n-digit numbers, X and Y into two halves of size $\frac{n}{2}$.
    - $X = X_L * 10^{\frac{n}{2}} + X_R$
    - $Y = Y_L * 10^{\frac{n}{2}} + Y_R$
- Requires 4 multiplications of size $\frac{n}{2}$ along with linear time additions and shifts
- Recurrence Relation: T(n) = 4T($\frac{n}{2}$) + O(n)
- Time Complexity: O($n^2$) (no improvement over the naive method)

##### b. Karatsuba's Algorithm
- Reduces the number of recursive multiplications from 4 down to 3 using basic algebraic manipulation.
- Instead of calculating $X_LY_R$ and $X_RY_L$ separately, it computes their sum using a single multiplication: $(X_L+X_R)(Y_L+Y_R) - X_LY_L - X_RY_R
- Recurrence Relation: T(n) = 3T($\frac{n}{2}$) + O(n)
- Time Complexity: O($n^{log_2 3}$) = O($n^{1.585}$), whch is significantly faster than O($n^2$).

For k-way split:
| DandC | Anatoly Karatsuba | Toom & Cook |
|-------|-------|------|
| T(n) = $k^2T(\frac{n}{k}) + O(n)$ | T(n) = $(k^2-1)T(\frac{n}{k}) + O(n)$ | T(n) = (2k-1)T(\frac{n}{k}) + O(n)$ |



## B. Greedy Method (GM)

GM is an algorithmic strategy that solves optimization problems by building a solution through a sequence of step-by-step decisions. At each stage, it permanently selects the locally optimal option that satisfies a specific immediate creterion, working under the assumption that these local choices will lead to globally optimal solution.

### 1. KnapSack Problem (KNAP)

#### Problem: Given a Knapsack of capacity 'M' and 'n' objects, each has weight ($w_i$) & Profit($P_i$). We need to maximize the profit with the condition that total weight of selected objects doesn't exceed capacity.


<details>
<summary>C++ code</summary>

```cpp
struct Item{
    double value, weight;
};

bool cmp(const Item &a, const Item &b){
    return (a.value/a.weight) > (b.value/b.weight);
}

double fractionalKnapsack(vector<Item> &item, double capacity){
    sort(items.being(), items.end(), cmp);

    double totalValue = 0;
    for(const auto &it: items){
        if(capacity <= 0) break;

        if(it.weight <= capacity){
            totalValue += it.value;
            capacity -= it.weight;
        }else{
            totalValue += it.value * (capacity/it.weight);
            capacity = 0;
        }
    }
    return totalValue;
}
```

</details>

- Time complexity: O(n) or O($nlog_2 n$) {if sorting is considered}

### 2. Job-Sequencing with Deadline / CPU - Scheduling Problem

#### Problem: Select a subset of 'n' given jobs, such that the jobs in the subset are completable within their Deadlines and generate maximum profit.

For each Job $J_i$, we have:
- Arrival Time (AT), by default = 0
- Brust Time (BT), by default = 1 unit
- Deadline ($d_i$)
- Profit ($P_i$)

<details>
<summary>C++ code</summary>

```cpp
struct Job{
    int id, deadline, profit;
};

bool cmp(const Job &a, const Job &b){
    return a.profit > b.profit;
}

pair<int, int> jobSequencing(vector<Job> &jobs, vector<int> &slot){
    int n = jobs.size();
    sort(jobs.begin(), jobs.end(), cmp);

    int maxSlots = n;
    slot.assign(maxSlots+1, -1);

    int count = 0, totalProfit = 0;
    for(int i=0; i<n; i++){
        for(int t=min(jobs[i].deadline, maxSlots); t>=1; t--){
            if(slot[t] == -1){
                slot[t] = jobs[i].id;
                totalProfit += jobs[i].profit;
                count++;
                break;
            }
        }
    }
    return {count, totalProfit};
}
```

</details>

- Time complexity: O($n^2$)


### 3. Optimal Merge Patterns

#### Problem: Given n sorted files $f_1,f_2,...,f_n$ containing $r_1,r_2,...,r_n$ records, merge them into a single sorted file using 2-way merging, where only two files are merged at a time. Merging files of size x and y produces a file of size x+y and costs x+y record movements. Find the merge order that minimuzes the total number of record movements.

<details>
<summary>C++ code</summary>

```cpp
long long optimalMerge(vector<int> &sizes, vector<tuple<long long, long long, long long>> &steps) {
    // Min-heap of file sizes
    priority_queue<long long, vector<long long>, greater<long long>> pq(sizes.begin(), sizes.end());

    long long totalCost = 0;

    while(pq.size() > 1){
        long long first = pq.top();
        pq.pop();
        long long second = pq.top();
        pq.pop();

        long long merged = first + second;
        totalCost += merged;
        steps.push_back({first, second, merged});

        pq.push(merged);
    }
    return totalCost;
}
```

</details>

- The number of record moves = $\sum_{i=1}^{n} d_i*q_i$
        - $d_i$ = distance from root to $F_i$
        - $q_i$ = Freq/Size of $F_i$

- Time Complexity:
    - with linked list: O($n^2$)
    - with Heap: O($nlog_2 n$)

- Space Complexity: O(n)


#### 3.1 Huffman Coding

#### Problem statement: Given n distinct characters $c_1,c_2,...,c_n$ with frequencies $f_1,f_2,...,f_n$ assign each character a binary code such that:
1. No code is a prefix of another code, so decoding is unambihuous
2. The total bits needed to encode the data, $\sum_{}^{} (f_i*$ length of code for $c_i)$, is as small as possible.

<details>
<summary>C++ code</summary>

```cpp
struct Node {
    char ch;
    int freq;
    Node *left, *right;
    Node(char c, int f, Node *l = nullptr, Node *r = nullptr) : ch(c), freq(f), left(l), right(r) {}
};

struct Compare {
    bool operator() (Node *a, Node *b) { return a->freq > b->freq; }
};

Node *buildHuffmanTree(const vector<char> &chars, const vector<int> &freq) {
    priority_queue<Node *, vector<Node *> Compare> pq;

    for(size_t i=0; i<chars.size(); i++){
        pq.push(new Node(chars[i], freq[i]));
    }

    while(pq.size() > 1){
        Node *l = pq.top();
        pq.pop();
        Node *r = pq.top();
        pq.pop();
        pq.push(new Node('\0', l->freq+r->freq, l, r));
    }
    return pq.top();
}

void generateCodes(Node *root, string code, map<char, string> &codes){
    if(!root) return;
    if(!root->left && !root->right) {
        codes[root->ch] = code.empty() ? "0" : code;
        return;
    }

    generateCodes(root->left, code+"0", codes);
    generateCodes(root->right, code+"1", codes);
}

void freeTree(Node *root){
    if(!root) return;
    freeTree(root->left);
    freeTree(root->right);
    delete root;
}
```

</details>

> Weekness of Huffman coding : In network transmission, misplacement of 1 bit might corrupt whole data.

> avg number of bits: $\sum_{i=1}^{n}d_i*q_i$

### 4. Minimum Cost Spanning Tree (MCST)

#### Problem: Given a connected, undirected, weighted graph G=(V,E) with n vertices and m edges, find a spanning tree T of G with minimum total edge weight. A spanning tree is a set of exactly n-1 edges that connects all vertices without forming a cycle

<details>
<summary>Prim's Algo</summary>

```cpp
long long prim(int n, const vector<vector<pair<int, int>>> &adj, vector<Edge> &mst) {
    vector<bool> visited(n, false);
    priority_queue<tuple<int, int, int>, vector<tuple<int, int, int>>, greater<>> pq;
    pq.push({0, 0, -1});

    long long cost = 0;
    int count = 0;

    while(!pq.empty()) {
        auto [w, u, parent] = pq.top();
        pq.pop();

        if(visited[u]) continue;
        visited[u] = true;
        cost += w;
        count++;
        if(parent != -1) mst.push_back({parent, u, w});

        for(auto [v, wt] : adj[u]){
            if(!visited[v]) pq.push({wt, v, u});
        }
    }
    return count == n ? cost : -1;
}
```

</details>


<details>
<summary>Kruskal's algo</summary>

```cpp
struct Edge {
    int u, v, w;
}

struct DSU {
    vector<int> parent, rnk;
    DSU(int n) : parent(n), rnk(n, 0) {iota(parent.begin(), parent.end(), 0); }
    int find(int x){
        return parent[x] == x ? x : parent[x] = find(parent[x]);
    }

    bool unite(int x, int y) {
        x = find(x);
        y = find(y);
        if(x == y) return false;
        if(rnk[x] < rnk[y]) swap(x, y);
        parent[y] = x;
        if(rnk[x] == rnk[y]) rnk[x]++;
        return true;
    }
};

long long kruskal(int n, vector<Edge> edges, vector<Edge> &mst) {
    stable_sort(edges.begin(), edges.end(), [](const Edge &a, const Edge &b){ return a.w < b.w; });

    DSU dsu(n);
    long long cost = 0;

    for(const Edge &e : edges) {
        if(dsu.unite(e.u, e.v)) {
            mst.push_back(e);
            cost += e.w;
            if((int)mst.size() == n-1) break;
        }
    }
    return (int)mst.size() == n-1 ? cost : -1;
}
```

</details>

|   | Time Complexity | Space Complexity |
|----|--------|------------|
| Prim's Algo | with adjacency matrix: O($V^2$) | O(V + E) |
|        | with adjacency list: O((V+E)log V) | |
| Kruskal's Algo | O(Elog E) | O(V+E) |

### 5. Single Source Shortest Path (SSSP)

#### Problem: Given a weighted graph G=(V,E) with n vertices, m edges, on-negative, m edges, non-negative edge weights and a source vertex s, find the shortest path (minimum total weight) from s to every other vertex.

<details>
<summary>Dijkstra's Algo</summary>

```cpp
const long long INF = LLONG_MAX/4;

vector<long long> dijkstra(int n, const vector<vector<pair<int, int>>> &adj, int src, vector<int> &parent) {
    vector<long long> dist(n, INF);
    parent.assign(n, -1);

    priority_queue<pair<long long, int>, vector<pair<long long, int>>, greater<> pq>;

    dist[src] = 0;
    pq.push({0, src});

    while(!pq.empty()) {
        auto [d, u] = pq.top();
        pq.pop();

        if(d > dist[u]) continue;

        for(auto [v, w] : adj[u]){
            if(dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                parent[v] = u;
                pq.push({dist[v], v});
            }
        }
    }
    return dist;
}

vector<int> getPath(Const vector<int> &parent, int v){
    vector<int> path;
    for(; v != -1; v = parent[v]) path.push_back(v);
    reverse(path.begin(), path.end());
    return path;
}
```

</details>

- Time complexity: O((V+E)log V)



## C. Dynamic Programming

Dynamic Programming (DP) is an algorithm design paradigm used to solve problems whose solution can be viewed as the result of making a sequence of decisions. Instead of commiting to one decision at each step, as the greedy method does, DP considers all possible choices and relies on the Principle of Optimality: whatever the first decision is, the remaining decisions must form an optimal sequence for the state that decision leads to. Because the same subproblems occure again and again, DP solves each one only once and stores its result in a table (memoization top-down, or tabulation bottom-up), so it can be reused instead of recomputed. This turns an exponential brute-force search into a polynomial-time solution.

DP implementation: 
1. Memoization (Top-down Approach) : Recursive
2. Tabulation (Bottom-up) : Iterative

### 1. Multi-stage Graph (Single source shortest path)

#### Problem: A multistage graph G=(V,E) is a directed, weighted graph whose vertices are split into k>=2 disjoint stages $V_1,V_2,...,V_k$, where $V_i={s}$(source) and $V_k={t}$(target). Every edge (u,v) goes from a vertex in stage $V_i$ to a vertex in stage $V_{i+1}$. Find the minimum-cost path from s to t.

Let Cost(i, j) represent cost of the path from vertex 'j' present in stage 'i', to reach destination vertex 't'.

1. `Cost(i, j) = min{c(j, k) + cost(i+1, k)}` - (1)   {k is some intermediate vertex}

2. `Cost(l-1, j) = c(j, t)` - (2)  {l is last stage}

- D(i, j) = 'k' that minimizes eq (1)

> Time complexity : O(V+E)

### 2. Travelling salesperson Problem (TSP)

#### Problem: Start from home city, visit all (n-1) cities once and come back home. Minimizes the distance (cost of tour).

Let g(i, S) represent cost of the Tour of TSP from vertex 'i' & visiting all vertices in the set 'S' exactly once and terminating the tour at $V_o$.

1. `g(i, S) = min(c(i, k), g(k, S-{k}))`; $k\in S$, (i, k)$\in$ E 
2. g(i, $\phi$) = c(i, $V_o$); i & $V_o$ don't have any intermediate vertex
- J(i, S) = value of 'k' that minimizes equation (1)

> Time complexity(Held-karp Algo) : O($n^22^n$)


### 3. All-pairs Shortest Paths (Flyod-warshall's Algo)

Let $A^k(i, j)$ represent cost of the path from vertex 'i'(Src) to vertex 'j'(Dest), with 'k' being the highest intermediate vertex along the path.

- $A^k(i, j) = min{A^{k+1}(i, k) + A^{k-1}(k, j), A^{k-1}(i, j)}$
- $A^0(i, j) = c(i, j)$

<details>
<summary>C++ code</summary>

```cpp
const long long INF = LLONG_MAX/4;

bool floydWarshall(int n, vector<vector<long long>> &dist, vector<vector<int>> &nxt) {
    for(int k=0; k<n; k++){
        for(int i=0; i<n; i++){
            if(dist[i][k] == INF) continue;

            for(int j=0; j<n; j++){
                if(dist[k][j] == INF) continue;
                if(dist[i][k] + dist[k][j] < dist[i][j]){
                    dist[i][j] = max(-INF, dist[i][k] + dist[k][j]);
                    nxt[i][j] = nxt[i][k];
                }
            }
        }
    }

    for(int i=0; i<n; i++){
        if(dist[i][i] < 0) return false;
    }
    return true;
}
```

</details>

>Time Complexity: O($n^3$)

> Space Complexity: O($n^2$)


### 4. 0/1 Knapsack (Binary Knapsack)

Let 01knap(n, m) represent t=profit with n-objects & KNAP of capacity m.

1. 01knap(n, m) = 01knap(n-1, m), $w_n$>m

&emsp; &emsp; &emsp; &emsp;&emsp;&emsp;&emsp;&emsp; = Max(01knap(n-1, m), 01knap(n-1, m-$w_n$)+$P_n$)

2. 01knap(n, m) = 0, n=0 or m=0

<details>
<summary>C++ code</summary>

```cpp
int knapsack01(int n, int m, const vector<int> &weight, const vector<int> & profit, vector<vector<int>> &k, vector<int> &chosen) {
    k.assign(n+1, vector<int>(m+1, 0));

    for(int i=1; i<=n; i++){
        for(int j=1; j<=m; j++){
            if(weight[i-1] > j){
                k[i][j] = k[i-1][j];
            }else{
                k[i][j] = max(k[i-1][j], k[i-1][j-weight[i-1]] + profit[i-1]);
            }
        }
    }

    int j = m;
    for(int i=n; i>=1 && j>0; i--){
        if(k[i][j] != k[i-1][j]){
            chosen.push_back(i);
            j -= weight[i-1];
        }
    }
    reverse(chosen.begin(), chosen.end());
    return k[n][m];
}
```

</details>

- Time Complexity: O(n*m)
- Space Complexity: O(n*m)

### 5. Longest Common Subsequence

#### Problem: Given strings x and y of length 'n' and 'm' chars, the problem of LCS is to determine a subsequence of longest length that is common to both 'x' and 'y'.

Let 'i' & 'j' be indices into the strings 'x' and 'y' as shown: x = <$x_1,x_2,...,x_n$>, y = <$y_1,y_2,...,y_m$>

Let L(i, j) denote the length of common subsequence of the strings x and y as defined above:

1. L(i, j) = 1 + L(i-1, j-1); if x[i] = y[j]

&emsp; &emsp;&emsp;&emsp;&emsp; = max(L(i-1, j), L(i, j-1)); if x[i]!=y[j]

Base condition: L(-1, a) = 0, L(b, -1) = 0

<details>
<summary>C++ code</summary>

```cpp
int lcsLength(const string &x, const string &y, vector<vector<int>> &L){
    int m = x.size(), n = y.size();
    L.assign(m+1, vector<int>(n+1, 0));

    for(int i=1; i<=m; i++){
        for(int j=1; j<=n; j++){
            if(x[i-1] == y[j-1]){
                L[i][j] = L[i-1][j-1] + 1;
            }else{
                L[i][j] = max(L[i-1][j], L[i][j-1]);
            }
        }
    }
    return L[m][n];
}
```

</summary>