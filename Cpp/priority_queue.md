## priority_queue

`priority_queue `容器适配器定义了一个元素有序排列的队列。默认队列头部的元素优先级最高。因为它是一个队列，所以只能访问第一个元素，这也意味着优先级最高的元素总是第一个被处理。

`priority_queue `模板有 3 个参数，其中两个有默认的参数；第一个参数是存储对象的类型，第二个参数是存储元素的底层容器，第三个参数是函数对象，它定义了一个用来决定元素顺序的断言。因此模板类型是：

```c++
template <typename T, typename Container=vector<T>, typename Compare=less<T>> class priority_queue
```

`priority_queue` 实例默认有一个 `vector `容器。函数对象类型 `less<T>` 是一个默认的排序断言，定义在头文件 `function `中，决定了容器中最大的元素会排在队列前面。fonction 中定义了 `greater<T>`，用来作为模板的最后一个参数对元素排序，最小元素会排在队列前面。

![img](priority_queue.assets/2-1P913134031947.jpg)

可以如下所示生成一个空的优先级队列：

```c++
priority_queue<string> words; 
```

可以用适当类型的对象初始化一个优先级队列：

```c++
string wrds[] { "one", "two", "three", "four"};
priority_queue<string> words { begin(wrds),end(wrds)}; // "two" "three" "one" "four" 
```

初始化列表中的序列可以来自于任何容器，并且不需要有序。优先级队列会对它们进行排序。

拷贝构造函数会生成一个和现有对象同类型的 priority_queue 对象，它是现有对象的一个副本。例如：

```c++
priority_queue<string> copy_words {words};
```

也有带右值引用参数的拷贝构造函数，它可以移动一个实参对象。

当对容器内容反向排序时，最小的元素会排在队列前面，这时候需要指定 3 个模板类型参数：

```c++
string wrds[] {"one", "two", "three", "four"};
priority_queue<string, vector<string>, greater<string>> words1 {begin (wrds) , end (wrds) }; // "four" "one" "three" "two"
```

这会通过使用 operator>() 函数对字符串对象进行比较，进而生成一个优先级队列，因此这会和它们在队列中的顺序相反。

优先级队列可以使用任何容器来保存元素，只要容器有成员函数 front()、push_back()、pop_back()、size()、empty()。这显然包含了 deque 容器，因此这里也可以用 deque 来代替：

```c++
string wrds [] {"one", "two", "three", "four"};
priority_queue<string, deque<string>> words {begin(wrds), end(wrds)};  // "two" "three" "one" "four" 
```

这个 words 优先级队列在 deque 容器中保存了一些 wrds 数组中的字符串，这里使用默认的比较断言。priority_queue 构造函数会生成一个和第二个类型参数同类型的容器来保存元素，这也是 priority_queue 对象的底层容器。

可以生成 vector 或 deque 容器，然后用它们来初始化 priority_queue：

```c++
vector<int> values{21, 22, 12, 3, 24, 54, 56};
// 第一个参数是一个用来对元素排序的函数对象，第二个参数是一个提供初始元素的容器
priority_queue<int> numbers{less<int>(), values};
```

在队列中用函数对象对 vector 元素的副本排序。values 中元素的顺序没有变，但是优先级队列中的元素顺序变为：56 54 24 22 21 12 3。优先级队列中用来保存元素的容器是私有的，因此只能通过调用 priority_queue 对象的成员函数来对容器进行操作。

如果想使用不同类型的函数，需要指定全部的模板类型参数。例如：

```c++
priority_queue<int, vector<int>,greater<int>> numbersl {greater<int>(), values};
```

第三个类型参数是一个比较对象类型。如果要指定这个参数，必须指定前两个参数——元素类型和底层容器类型。

### priority_queue 操作

对 priority_queue 进行操作有一些限制：

- `push(const T& obj)`：将 obj 的副本放到容器的适当位置，这通常会包含一个排序操作。
- `push(T&& obj)`：将 obj 放到容器的适当位置，这通常会包含一个排序操作。
- `emplace(T constructor a rgs...)`：通过调用传入参数的构造函数，在序列的适当位置构造一个T对象。为了维持优先顺序，通常需要一个排序操作。
- `top()`：返回优先级队列中第一个元素的引用。
- `pop()`：移除第一个元素。
- `size()`：返回队列中元素的个数。
- `empty()`：如果队列为空的话，返回true。
- `swap(priority_queue<T>& other)`：和参数的元素进行交换，所包含对象的类型必须相同。

priority_queue 也实现了赋值运算，可以将右操作数的元素赋给左操作数；同时也定义了拷贝和移动版的赋值运算符。需要注意的是，priority_queue 容器并没有定义比较运算符。因为需要保持元素的顺序，所以添加元素通常会很慢。