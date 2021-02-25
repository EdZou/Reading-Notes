这就是《Effective Modern C++》的笔记了，和《Effective C++》类似，本笔记也会按照条款来进行记录

## 条款0：术语和惯例

1. C++98指的是98年的C++，数字是年份，新的年份的C++版本是老旧版本的超集
2. 熟悉的Widget，rhs，lhs，...
3. lambda表达式创建的函数对象称为闭包
4. built-in指针称为裸指针，对应的是智能指针
5. 构造函数称为`ctor`（constructor），析构函数称为`dtor`（deconstructor）

这本书比Effective C++的翻译腔好多了：）

## 条款1：理解模板型别推导

一个通用的伪码

```c++
template<typename T>
void f(ParamType param);

f(expr);                // 以某表达式调用f
```

假设如下模板：

```c++
template<typename T>
void f(const T& param);

int x = 0;
f(x);
```

该例中，T理所应当被推定为int，ParamType被推定为const int&

*“T的类型 = expr的类型 ”*  这句话不完全正确，因为T不仅依赖于expr，还依赖于ParamType，下面分类讨论

#### 1. ParamType是个指针或引用，但不是个万能引用（万能引用之后会说）

推导T的过程如下运作：

1. 若expr具有引用，**先将引用忽略**
2. 之后在对expr和paramType进行模式匹配

比如下面三个例子

```c++
template<typename T>
void f(T& param);

int x = 0;           // T是int，ParamType为int
const int cx = x;    // T是const int，ParamType为const int&
const int& rx = x;   // T是cosnt int，ParamType为const int&
```

#### 2. ParamType是个万能引用

形参的声明型别写作`T&&`就是万能引用（C++ primer里讲过，引用折叠，可用于forward）

1. 若expr是左值，则T和ParamType都会被推导为左值引用
2. 若expr是右值，则应用常规的推导原则

如下例子：

```c++
template<typename T>
void f(T&& param);

int x = 0;           // T是int&，ParamType为int&
const int cx = x;    // T是const int&，ParamType为const int&
const int& rx = x;   // T是const int&，ParamType为const int&
f(27);               // T是int, ParamType为int&&
```

#### 3. ParamType既非指针也非引用

这也就是本质传副本，推导如下：

1. 若expr有引用类型，忽略（仅仅因为参数无法修改，不代表其副本也无法修改，说白了就是**顶层const被忽略，底层const保留**那一套）
2. 如果expr是一个const或volatile对象，都直接忽略

如下例子

```c++
template<typename T>
void f(T param);

int x = 0;           // f(x)  -> T是int，ParamType为int
const int cx = x;    // f(cx) -> T是int，ParamType为int
const int& rx = x;   // f(rx) -> T是int，ParamType为int
```

#### 数组实参

如果我们有一个数组如下：

```c++
const char name[] = "ABCD"; // type为const char[5]
```

如果f为：

```c++
template<typename T>
void f(T param);
```

那么`f(name);`会把T推导为const char*(隐式转换)

但如果f为：

```c++
template<typename T>
void f(T& param);
```

`f(name;)`会把T推导为const char[13]，所以我们也可以使用参数模板，如下：

```c++
template<typename T, std::size_t N>
constexpr std::size_t arraySize(T (&)[N]) noexcept {
    return N;            // 通过template的推导，得到并返回数组大小
}

// 使用如下
int keyVals[] = {1, 2, 3, 4};
int mappedVals[arraySize(keyVals)]; // int mappedVals[4]
```

注意constexpr函数的结果在编译期就能得到

#### 函数实参

```c++
void someFunc(int, double); // type为void(int, double)

template<typename T>
void f1(T param);
template<typename T>
void f2(T& param);

f1(someFunc);               // param被推导为函数指针， void (*)(int, double)
f1(someFunc);               // param被推导为函数引用， void (&)(int, double)
```

总结就是，数组或函数类型会被退化成相应的指针，除非是初始化引用形参可以不用退化

## 条款2：理解auto型别推导

auto和T有双向的算法变换，如上例中，可以用auto来理解，如下：

```c++
auto x = 27;				// 对应 (T param) 模板         -> int
const auto cx = x;          // 对应 (const T param) 模板   -> int
const auto& rx = x;			// 对应 (const T& param) 模板  -> int
```

同样可以根据**型别饰词**来进行分类：

1. 指针或引用但非万能引用

2. 万能引用

3. 既非指针也非引用

   ```c++
   // 情况1
   const auto& rx = x;
   // 情况2
   auto&& uref1 = x;       // x是左值int       -> unref1 == int&
   auto&& uref2 = cx;      // x是左值const int -> unref2 == const int&
   auto&& uref3 = 27;		// x是右值int&& 	 -> unref3 == int&&
   // 情况3
   auto x = 27;      
   const auto cx = x;
   ```

有关于函数/数组参数的隐式转换也同样适用

注意，当使用列表初始化（花括号内元素）时，auto会把对象推导为`std::initializer_list`，如下

```c++
// 列表初始化和一般初始化得到的结果一致
int x1 = 27;
int x2(27);
int x3 = {27};
int x4{27};
// 但auto的推导却不一致
auto x1 = 27;    // auto推导为int
auto x3 = {27};  // auto推导为std::initializer_list<int>
```

如果**型别推导失败**，例如列表内类型不一致，代码通不过编译，如下例：

```c++
auto x5 = {1, 2, 3.0}; // 无法推导T的类型
```

但注意如果直接往template里传一个list，template推导不出initializer_list，如下：

```c++
template<typename T>
void f(T param);

f({13,1,12}); // 错误，template推导不出initializer_list

// 当然，可以通过以下形式，让template推导list中的数据类型
template<typename T>
void f(std::initializer_list<T> initList);

f({ 11, 23, 9 });    // T推导为int，initList类型std::initializer_list<int>
```

因为{}很麻烦，所以很多程序员都只在必要时才使用{}

C++14允许auto来说明函数返回值需要推导，以及lambda表达式也会在形参中用到auto，但这些都是template型别推导而不是auto型别推导，如下

```c++
auto createInitList() {
    return {1, 2, 3};         // 错误，本质template型别推导，推不出ini_list
}
```

lambda同理

## 条款3：理解decltype

这里主要讲到了C++ primer中也提到的**返回值型别尾序语法（trailing return type syntax）**，如下：

```c++
template<typename Container, typename Index>
auto authAndAccess(Container& c, Index i)    // 初版，可运行但有改进空间
	-> decltype(c[i]) {
    authenticateUser();
    return c[i];
}
```

这个语法的好处是可以在**指定返回值type时使用函数形参**

C++11允许单表达式的lambda式的返回值进行推导，C++14可以对所有的lambda和所有的函数推导。而且C++14中可以去掉decltype，只留auto，如下：

```c++
template<typename Container, typename Index> // C++14
auto authAndAccess(Container& c, Index i) {  // 有错误
    authenticateUser();
    return c[i];
}
```

就像条款2中说的，auto返回值型别推导本质是**template型别推导**，这意味着上述写法会丢失引用特性进而造成严重后果

如果想改进上述办法，可以使用decltype推导来保留引用特性，如下：

```c++
template<typename Container, typename Index> // C++14,能运行,还有改进空间
decltype(auto)
authAndAccess(Container& c, Index i) { 
    authenticateUser();
    return c[i];
}
```

当然，decltype不仅能保留引用特性，还能保留const特性，如下：

```c++
Widget w;
const Widget& cw = w;
auto myWidget1 = cw;            // 推导出类型为Widget
decltype(auto) myWidget2 = c2;	// 推导出类型为const Widget&
```

还有一个改进是形参，因为形参式non-const左值类型，这样传递会改变传入参数的左/右值性（无法传入右值参数）。就像条款2中提到的，我们可以通过万能引用来解决这个问题

然后根据条款25，万能引用需要应用`std::forward`，如下：

```c++
template<typename Container, typename Index> // C++14最终版
decltype(auto)
authAndAccess(Container&& c, Index i) { 
    authenticateUser();
    return std::forward<container>(c)[i];
}
```

C++11不能省略decltype的后置语句，如下：

```c++
template<typename Container, typename Index> // C++14最终版
auto
authAndAccess(Container&& c, Index i) 
->	decltype(std::forward<container>(c)[i]) { 
    authenticateUser();
    return std::forward<container>(c)[i];
}
```

最后注意，decltype在应用到**只有一个名字的表达式会返回原本类别**，但**其他所有更复杂的表达式一律返回左值引用**，比如：

```c++
int x = 0;
decltype(x);   -> int
decltype((x)); -> int&
```

## 条款4：掌握查看型别推到结果的办法

#### IDE编辑器

鼠标悬停时（vscode可以做到）

文件需要处于一种可编译的状态，但较复杂就不太有用了

#### 编译器诊断信息

（这方法真的genius）

通过编译器错误信息来显式具体的类型，如下：

```c++
// 声明一个template但是不定义，只要使用就报错
template<typename T>
class TD;

// 比如我们想看x，y的型别，做以下操作
TD<decltype(x)> xType;
TD<decltype(y)> yType;
```

之后编译器报错就会显示x，y的类型，见P36

#### 运行时输出

就是通俗的print出来看看

比如：`std::cout << typeid(x).name() << endl;`

过程就是调用`typeid`得到`std::type_info`对象，再使用`name()`成员函数得到const char*的C-style字符串

但`std::type_info::name`的调用不一定会返回有意义的内容，见P36编译器返回的结果，而且会省略掉volatile以及顶层const，这不是好结果

然后就是无敌的Boost提供的TypeIndex库，具体使用见P38，书上的例子是`boost::typeindex::type_id_with_cvr`，`with_cvr`意思就是保留顶层const，volatile以及引用饰词

## 条款5：优先选用auto，而非显式型别声明

