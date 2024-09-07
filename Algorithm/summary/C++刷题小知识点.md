## 数据结构

```c++
struct ListNode {
    int val;
    ListNode *next;

    ListNode() : val(0), next(nullptr) {}

    ListNode(int x) : val(x), next(nullptr) {}

    ListNode(int x, ListNode *next) : val(x), next(next) {}
};
```

```c++
struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    
    TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
};
```

## sort() 自定义比较函数

- 由于容器支持的迭代器类型必须为随机访问迭代器，sort() 只支持array、vector、deque 这 3 个容器。

### 自定义排序方法

- 缺点：定义的函数只能接受两个参数，就是比较的双方。无法根据 b 数组中元素的大小，来对 a 数组进行排序


```c++
bool cmp(int a, int b) {
    // 降序
    return a > b;
}

int main() {
    vector<int> arr{1, 3, 4, 2, 9, 0};
    sort(begin(arr), end(arr), cmp);
    for (auto i: arr) {
        cout << i << " ";
    }
}
```

### lambda 表达式

结构：`[capture list](parameter list) ->return type {function body}`

其中，`parameter list(参数列表)`和`function body(函数体)`与上面的cmp函数没有什么差异。`return type`虽说必须尾置，但通常可以省略。而`capture list`(捕获列表)中可以填写作用域中的参数，使它在 lambda 表达式内部可见。这就解决了上面的第二个问题。比如说，我们要捕获b数组，就写成[&b]即可(&代表引用捕获)，也就是说捕获列表使 lambda 表达式可以使用不限量的外部参数，当然，如果不想指定捕获哪些，直接写[&]，就是全部引用捕获。同时，哪里用到，就写在哪里，还不用起名字(所以也叫匿名函数)，使它既直观又方便。

```c++
// 按照string的长度对string序列排序
sort(vec.begin(),vec.end(),[](auto &s1,auto &s2){return s1.size()<s2.size();});

// 按照pair的second大小降序排序，如果相同再按first升序排序
sort(vec.begin(),vec.end(),[](auto &p1,auto &p2){return p1.second==p2.second?p1.first<p2.first:p1.second>p2.second;})
```

```c++
int main() {
    vector<int> flag = {2, 3, 5, 4, 1, 8, 7, 9, 6};
    vector<int> index = {0, 1, 2, 3, 4, 5, 6, 7, 8};
    // 根据flag数组中的元素大小，来对index数组(index对应着flag的下标)进行排序
    sort(index.begin(), index.end(), [&flag](int a, int b) { return flag[a] < flag[b]; });
    for (int i: index)
        cout << i << " ";
    return 0;
}
```

```c++
int main() {
    vector<int> vals = {2, 4, 1, 2, 2, 5, 3, 4, 4};
    vector<vector<int>> edges = {
            {0, 1},
            {2, 1},
            {0, 3},
            {4, 1},
            {4, 5},
            {3, 6},
            {7, 5},
            {2, 8}
    };
    // vals[i] 表示顶点 i 的值，edges[i][0]、edges[i][1] 为边的两个顶点
    // 根据边的两个顶点的较大值进行排序
    sort(begin(edges), end(edges),
         [&vals](vector<int> v1, vector<int> v2) {
             return max(vals[v1[0]], vals[v1[1]]) < max(vals[v2[0]], vals[v2[1]]);
         });
    for (int i = 0; i < edges.size(); ++i) {
        cout << edges[i][0] << " " << edges[i][1] << endl;
    }
}
```
