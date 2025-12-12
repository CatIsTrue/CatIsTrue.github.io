---
title: C++ explicit关键字什么时候用？
tags:
  - C++
  - Modern C++
categories: C++
abbrlink: e7024663
date: 2019-06-05 17:18:00
---

看到项目代码里的构造函数，有时候用`explicit`,有时候不用，引发了我的思考，到底这个关键字什么时候用呢？

带着这个问题，我查看了一些资料。


比较官方的解释是：

`explicit`关键字用于**禁止构造函数的隐式类型转换**。

当构造函数被标记为`explicit`时，编译器不会自动使用该构造函数进行隐式转换。

---

听得似懂非懂。


还是找些具体的例子看看来说吧。

## 没有explicit导致的重载决议歧义错误

C++ 有一个默认特性：如果一个构造函数只接受一个参数，编译器会默认认为它定义了一种“隐式转换路径”。

在这个例子里，由于单参数构造函数没有explicit，所以没有禁止隐式转换，导致重载决议歧义错误，出现编译错误。
代码如下：
```cpp
// errDemo.cc
#include <iostream>

class Cents {
    int cents_;
public:
    Cents(int c) : cents_(c) { std::cout << "Cents: " << c << "\n"; }
};

class Dollar {
    int dollar_;
public:
    Dollar(int c) : dollar_(c) { std::cout << "Dollar: " << c << "\n"; }
};

void payBill(Cents amount) {
    std::cout << "Paying with cents\n";
}

void payBill(Dollar amount) {
    std::cout << "Paying with dollars\n";
}

int main() {
    payBill(500);   // ❌ 编译错误：ambiguous call
    return 0;
}
```

由于两个函数的匹配程度完全相同，都需要一次用户定义的隐式转换， 编译器无法确定选择哪一个，导致编译错误
![重载决议歧义错误](/images/image-17.png)


这时，我们通过使用 explicit，要求必须明确意图的进行调用，于是修改上面的代码
```cpp
// 1. 构造函数前加explicit关键字
explicit Cents(int c) : cents_(c) { std::cout << "Cents: " << c << "\n"; }  // 禁止隐式转换
explicit Dollar(int c) : dollar_(c) { std::cout << "Dollar: " << c << "\n"; }  // 禁止隐式转换


// 2.明确调用意图
// payBill(500);           // ❌ 此时会报编译错误：没有匹配的重载
                           //  error: no matching function for call to ‘payBill(int)
payBill(Cents(500));    // ✅ 必须显式转换， 明确意图
payBill(Dollar(500));   // ✅ 必须显式转换， 明确意图
```

![明确意图，显式转换](/images/image-18.png)

---

## 没有explicit导致的性能陷阱

在性能关键、资源敏感的软件开发中，没有使用`explicit`，超出预期的隐式转换，可能无意间消耗内存资源。
代码如下：
```cpp
class Matrix {
    double* data;
    size_t size_;
public:
    Matrix(size_t size) : size_(size) {
        data = new double[size * size];  // 昂贵的内存分配！
    }
};

void compute(const Matrix& m);

// 性能陷阱
compute(1000);  // 意外创建了1000x1000的矩阵！
```

这时，通过使用 explicit，要求必须明确意图的进行调用，修改上面的代码
```cpp
// 1. 构造函数前加explicit关键字
explicit Matrix(size_t size) : size_(size) {
    data = new double[size * size]; 
}

// 2.明确调用意图
// compute(1000);           // ❌ 编译错误
compute(Matrix(4));   // ✅ 明确知道在创建矩阵
```

这种场景，主要防止的是，有可能开发人员根本没有意识到会触发内存分配，或者误在性能敏感的代码中意外创建大对象。
当然，如果的确需要创建，就显式构造，开发知道自己在干什么，可以对自己的行为负责。


## 加上explicit对列表初始化的影响

分析完单参数构造函数非必要最好加上`explicit`关键字之后，我们再看看，如果多参数构造函数加上`explicit`，又会有什么影响呢？

官方的说法是：当`explicit`构造函数接受`std::initializer_list`时，会失去所有数量的初值的隐式转换能力。
（说起来有点绕口，直接看例子就清晰了。）

```cpp
class Container {
public:
    // 非explicit版本
    Container(std::initializer_list<int> values);
};

class SafeContainer {
public:
    // explicit版本
    explicit SafeContainer(std::initializer_list<int> values);
};

// 对比效果
Container c1 = {};           // ✅ 0个初值 隐式转换
Container c2 = {42};         // ✅ 1个初值 隐式转换
Container c3 = {1, 2, 3};    // ✅ 多个初值 隐式转换

SafeContainer sc1 = {};      // ❌ 失去0个初值的隐式转换
SafeContainer sc2 = {42};    // ❌ 失去1个初值的隐式转换
SafeContainer sc3 = {1, 2, 3}; // ❌ 失去多个初值的隐式转换

// 必须使用直接初始化
SafeContainer sc4{};         // ✅ 0个初值
SafeContainer sc5{42};       // ✅ 1个初值
SafeContainer sc6{1, 2, 3};  // ✅ 多个初值
```

## 加上explicit对多参数构造函数的影响

如果是普通的多参数构造函数加上`explicit`，又会有什么影响呢？
先说答案：直接初始化仍然不受影响，但拷贝初始化会受到影响。

代码如下：
```cpp
class P {
public:
    explicit P(int a, int b, int c);
};

P obj1{1, 2, 3};        // ✅ 直接初始化，总是可以
P obj2(1, 2, 3);        // ✅ 直接初始化，总是可以


P obj3 = {1, 2, 3};     // ❌ 拷贝初始化，explicit会阻止
P obj4 = P{1, 2, 3};    // ✅ 显式构造后拷贝，可以
```

为什么加`explicit`会影响拷贝初始化呢？
分析下来是因为拷贝初始化需要两步：
1. 用`{1, 2, 3}`创建临时对象（需要隐式调用构造函数）
2. 将临时对象拷贝给目标变量
而`explicit`阻止了第一步的隐式调用。

---

有些场景需要加，但是有些场景我们又不应该使用explicit

## 拷贝和移动构造函数绝对不要加`explicit`

虽然 C++ 语法上允许你给拷贝/移动构造函数加上 explicit，但在工程实践中，这样做基本等于“自杀”，或者说是给使用者（包括你自己）制造巨大的麻烦。

如果说拷贝和移动构造函数加了`explicit`，这会发生什么？
答案是：会破坏基本语义，破坏函数传递，破坏容器使用，破坏返回值语义。
品一品，是不是这么回事。
上代码细看。

```cpp
// 永远不要这样做
explicit MyClass(const MyClass& other);  // ❌
explicit MyClass(MyClass&& other);       // ❌

// 1.破坏基本语义
MyClass obj1;
MyClass obj2(obj1); // ✅ 允许：直接调用（像函数调用一样）
MyClass obj3 = obj1;  // ❌ 如果拷贝构造是explicit，这会编译错误！因为等号 "=" 语义上要求隐式转换。
                      // 这意味着你无法再使用最自然的 = 进行赋值初始化了。

// 2.破坏函数传递
void process(MyClass obj);  // 按值传递
MyClass original;
process(original);  // ❌ 如果拷贝构造是explicit，无法传递参数！这是因为为了把 original 传进函数，需要构造一个临时对象。这是一次隐式动作，被 explicit 拦截了。
// 你被迫要写成这样这种恶心的代码：
process(MyClass(original)); // ✅ 显式强转
                        // 如果所有的类都这样写，C++ 的函数调用简直没法看了。

// 3.破坏容器使用
std::vector<MyClass> vec;
MyClass obj;
vec.push_back(obj);  // ❌ 容器无法工作！
// 标准库容器依赖于拷贝/移动语义 
// 很多 STL 操作内部逻辑是：在内存中放置新对象时，使用了 = 语义或者隐式构造语义。
// 如果拷贝/移动构造函数是 explicit 的，你的类基本上就告别 STL 标准库了，除非你用非常小心翼翼的 emplace 操作，但也很容易踩雷。

// 4.破坏返回值语义
MyClass createObject() {
    MyClass obj;
    return obj;  // ❌ 无法返回对象！
}
// 在语义检查阶段，return 语句被视为一种“隐式构造”
```


关于拷贝/移动构造函数加`explicit`会破坏返回值语义，有些小伙伴可能像我一样，在这里会有一个疑问：“C++ 不是有 RVO/NRVO (返回值优化) 吗？编译器不是会把拷贝直接消除掉吗？既然消除了，为什么还要检查构造函数？”

这是因为这里有两步：

语义检查 (Semantic Check)：编译器的第一步是检查“如果你要拷贝，代码写得对不对”。这一步要求拷贝/移动构造函数必须是可访问的且非 explicit 的。

代码生成与优化 (Code Generation & Optimization)：只有通过了语义检查，编译器才会进行第二步优化（RVO），在运行时消除这次拷贝。

所以，结论就是：即使 RVO 会在运行时消除拷贝，`explicit` 依然会在编译时导致报错，因为它阻断了语义检查阶段的合法性。

---

正确的做法
```cpp
class MyClass {
public:
    // ✅ 正常的拷贝和移动构造函数
    MyClass(const MyClass& other) = default;
    MyClass(MyClass&& other) = default;
    
    // 或者如果不需要拷贝，就删除它们
    MyClass(const MyClass&) = delete;
    MyClass(MyClass&&) = delete;
};
```

---

## 总结

`explicit` 的核心目的是防止意外的类型转换（比如把 int 变成 String）。

但是，拷贝和移动本身就是同一种类型的传递，它们在 C++ 语言层面被设计为一种“自然流动”的操作。阻断这种流动（加上 explicit），就会导致这个类在 C++ 的生态系统中寸步难行。


参考 Google C++ 编程规范的建议,我的使用原则目前是：

（1）原则上，所有的单参数构造函数都应该加上 `explicit`。
  *除非你真的希望用户使用这种隐式转换带来便利。（例如模拟基础数据类型）*

（2）多参数构造函数（C++11 之前通常不需要，但 C++11 引入了列表初始化 {}）按具体场景决定加不加`explicit`：
  *如果构造函数支持列表初始化，且你不想让 {1, 2} 隐式变成你的对象，也要加 explicit。*

（3）拷贝/移动构造函数，绝对不要加 `explicit`。

(大家有什么使用习惯吗？欢迎评论区交流)
