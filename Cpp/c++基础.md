## 指针

```c++
#include <iostream>

using namespace std;

int main() {
    // 实际变量的声明
    int var = 20;
    // 指针变量的声明
    int *addr;
    // 在指针变量中存储 var 的地址
    addr = &var;
    cout << "var = " << var << endl;
    // 输出在指针变量中存储的地址
    cout << "地址addr = " << addr << endl;
    // 访问指针中地址的值
    cout << "地址addr中存的值 = " << *addr << endl;
}
```

在变量声明的时候，如果没有确切的地址可以赋值，为指针变量赋一个 NULL 值是一个良好的编程习惯。赋为 NULL 值的指针被称为**空**指针。NULL 指针是一个定义在标准库中的值为零的常量。

## 引用

引用变量是一个别名，也就是说，它是某个已存在变量的另一个名字。一旦把引用初始化为某个变量，就可以使用该引用名称或变量名称来指向变量。

引用很容易与指针混淆，它们之间有三个主要的不同：

- 不存在空引用。引用必须连接到一块合法的内存。
- 一旦引用被初始化为一个对象，就不能被指向到另一个对象。指针可以在任何时候指向到另一个对象。
- 引用必须在创建时被初始化。指针可以在任何时间被初始化。

```c++
#include <iostream>

using namespace std;

int main() {
    // 声明普通的变量
    int i;

    // 声明引用变量
    int &r = i;

    i = 5;
    cout << "i = " << i << endl;
    cout << "r = " << r << endl;

    r = 6;
    cout << "r = " << r << endl;
}
```

### 引用作为参数

```c++
#include <iostream>

using namespace std;

// 函数声明
void swap(int &x, int &y);

int main() {
    // 局部变量声明
    int a = 100;
    int b = 200;

    cout << "交换前，a 的值：" << a << endl;
    cout << "交换前，b 的值：" << b << endl;

    // 调用函数来交换值
    swap(a, b);

    cout << "交换后，a 的值：" << a << endl;
    cout << "交换后，b 的值：" << b << endl;
    // 成功交换。如果把引用都换成普通变量，则会交换失败
}

// 函数定义
void swap(int &x, int &y) {
    int temp;
    temp = x;
    x = y;
    y = temp;
}
```

### 引用作为返回值

当函数返回一个引用时，则返回一个指向返回值的隐式指针。

```c++
#include <iostream>

using namespace std;

double vals[] = {10.1, 12.6, 33.1, 24.1, 50.0};

double &setValues(int i) {
    double &ref = vals[i];
    // 返回第 i 个元素的引用，ref 是一个引用变量，ref 引用 vals[i]
    return ref;
}

// 要调用上面定义函数的主函数
int main() {
    cout << "改变前的值" << endl;
    for (int i = 0; i < 5; i++) {
        cout << "vals[" << i << "] = ";
        cout << vals[i] << endl;
    }

    setValues(1) = 20.23;
    setValues(3) = 70.8;

    cout << "改变后的值" << endl;
    for (int i = 0; i < 5; i++) {
        cout << "vals[" << i << "] = ";
        cout << vals[i] << endl;
    }
    return 0;
}
```

当返回一个引用时，要注意被引用的对象不能超出作用域。所以返回一个对局部变量的引用是不合法的，但是，可以返回一个对静态变量的引用。

```c++
int &func() {
    int q;
    //! return q; // 在编译时发生错误
    static int x;
    return x;     // 安全，x 在函数作用域外依然是有效的
}
```

```c++
#include <iostream>

using namespace std;

// 返回对静态变量的引用
int &getStaticRef() {
    static int num = 5; // 静态变量
    return num;
}

int main() {
    int &ref = getStaticRef(); // 获取对静态变量的引用
    cout << "初始值：" << ref << endl;

    ref = 10; // 修改静态变量的值

    cout << "修改后的值：" << ref << endl;
    cout << "再次调用函数后的值：" << getStaticRef() << endl;

    return 0;
}
```

## 指针和引用的使用场景

1. **需要返回函数内局部变量的内存的时候用指针**。使用指针传参需要开辟内存，用完要记得释放指针，不然会内存泄漏。而返回局部变量的引用是没有意义的
   - 引用是对已经存在的变量进行别名，而不是新建一个变量。当函数返回时，函数内的局部变量会被销毁，引用指向的内存也会被释放，因此返回引用会导致悬空引用（dangling reference）的问题，即引用指向已经被释放的内存，这会导致程序崩溃或者产生不可预期的结果。
   - 指针可以通过动态内存分配（如new）来分配内存，返回指针时可以将内存的所有权转移给调用者，避免了悬空指针的问题。但是需要注意的是，如果使用指针返回局部变量的内存，调用者需要负责释放这块内存，否则会导致内存泄漏。

2. **对栈空间大小比较敏感（比如递归）的时候使用引用**。使用引用传递不需要创建临时变量，开销要更小

3. 如果需要**避免拷贝大对象或者类**，提高效率，或者需要在函数内部修改函数外部变量的值，应该使用引用。
   - 指针和引用都可以用来在函数内部修改函数外部变量的值，但它们之间有一些重要的区别。使用指针时，需要在函数内部分配内存来存储指向外部变量的指针。如果在函数内部修改指针所指向的变量的值，那么这个指针就会失效，因为它指向的地址已经被释放了。这样会导致程序崩溃或产生未定义的行为。

4. 使用引用参数的主要原因有两个：程序员能够修改调用函数中的数据对象；通过传递引用而不是整个数据对象，可以提高程序的运行速度。
   - 只使用传递过来的值，而不对值进行修改。
     - 如果数据对象很小，如内置数据类型或小型结构，使用按值传递。
     - 如果数据对象是数组，则使用指向const的指针。
     - 如果数据对象是较大的结构，则使用const指针或者const引用，以提高程序的效率。
     - 如果数据对象是类对象，则使用const引用。因此，传递类对象参数的标准方式是按引用传递。
   - 需要修改传递过来的值。
     - 如果数据对象是内置数据类型，则使用指针。
     - 如果数据对象是数组，则只能使用指针。
     - 如果数据对象是结构体。则使用指针或者引用。
     - 如果数据对象是类对象，则使用引用。

## 结构体

```c++
#include <iostream>
#include <cstring>

using namespace std;

void printBook(struct Books book);

// 声明一个结构体类型 Books
struct Books {
    char title[50];
    char author[50];
    char subject[100];
    int book_id;
};

int main() {
    // 定义结构体类型 Books 的变量
    Books Book1{};

    // Book1 详述
    strcpy(Book1.title, "C++ 教程");
    strcpy(Book1.author, "菜鸟");
    strcpy(Book1.subject, "编程语言");
    Book1.book_id = 12345;
    // 输出 Book1 信息
    printBook(Book1);
}

// 结构体作为函数参数
void printBook(struct Books book) {
    cout << "标题 : " << book.title << endl;
    cout << "作者 : " << book.author << endl;
    cout << "类目 : " << book.subject << endl;
    cout << "ID : " << book.book_id << endl;
}
```

### typedef 关键字

```c++
using namespace std;

// 为创建的类型取一个"别名"
typedef struct Books {
    char title[50];
    char author[50];
    char subject[100];
    int book_id;
} Books;

int main() {
    // 直接使用 Books 来定义 Books 类型的变量，而不需要使用 struct 关键字
    Books Book1, Book2;
}
```

