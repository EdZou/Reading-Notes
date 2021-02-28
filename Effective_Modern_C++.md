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

这里先提到了`std::function`，老生常谈的部分就不提了

这里先给出`std::function`和auto的两种定义示例再讨论其不同：

```c++
std::function<bool(const std::unique_ptr<Widget>&,
                   const std::unique_ptr<Widget>&)> // std::function来存储闭包
    derefUPLess = [](const std::unique_ptr<Widget>& p1,
                     const std::unique_ptr<Widget>& p2)
					{ return *p1 < *p2; }
auto derefUPLess = [](const std::unique_ptr<Widget>& p1, // auto来存储闭包
                     const std::unique_ptr<Widget>& p2)
					{ return *p1 < *p2; }
```

auto的和闭包是**同一型别**，要求的内存量也和闭包一致；`std::function`版本存储的是一个`std::function`的**一个实例**，不管给定的闭包的尺寸，**实例的大小都是固定的**。如果实例空间不够大，那么`std::function`的构造函数就会**分配heap上的内存**来存储该闭包。所以`std::function`几乎一定会比auto声明的闭包**使用更多内存**

此外，因为编译器对于`std::function`的实现细节往往**限制内联，并且产生间接函数调用**，这也导致`std::function`声明的闭包几乎一定会比auto声明的同一闭包要**慢**

所以auto大获全胜，**尽量用auto声明闭包**

还有就是auto可以避免因为系统不同，比如（`std::vector<int>::size_type`在32-bit Windows为32位但在64-bit Windows上为64位），可以通过auto来规避风险

还有就是在如下循环中：

```c++
for (const std::pair<std::string, int>& p: map) {...}
```

因为map的key是const型，这里应该是`std::pair<const std::string, int>`，编译器会想办法把原本的元素转换为指定的显式的类型（通过copy），然后效率爆炸

所以**显式指定型别**可能导致你既不想要，也没想到的**隐式型别转换**，还是使用auto省事，如下：

```c++
for (const auto& p: map) {...}
```

## 条款6：当auto推导的型别不符合要求时，使用带显式型别的初始化物习惯用法

比如对于`std::vector<bool>`来说，它的operator[]返回的并不是元素的引用（除了bool以外都是），而是一个`std::vector<bool>::reference`型别的对象（嵌套在`std::vector<bool>`里的类），所以这里如果使用auto，会推导出一个`std::vector<bool>::reference`的型别

之所以会有`std::vector<bool>::reference`，是因为`std::vector<bool>`做了优化，把bool压缩成bit(bitmap的思想)，于是就内嵌了一个`std::vector<bool>::reference`类，并允许其向bool型别**隐式转换**

所以如果我们做如下操作：

```c++
std::vector<bool> features(const Widget& w);

// 行为undefined
auto highPriority = features(w)[5];
```

过程如下：

前提：`std::vector<bool>::reference`会让对象含有一个指针，指向一个**机器字（word）**，该机器字持有被引用的bit并基于那个bit对应字的offset。

1. features返回一个`std::vector<bool>`的临时对象temp
2. temp调用operator[]返回一个`std::vector<bool>::reference`对象，含有指针和offset
3. 由于highPriority是`std::vector<bool>::reference`对象的一个副本，所以highPriority也会有一个指向机器字的指针以及offset
4. 表达式结束时temp作为临时量被析构，而highPriority则含有一个空悬指针
5. 行为undefined

本例中`std::vector<bool>::reference`是一个**隐形代理类**，显式代理类比如`std::unique_ptr`这类的智能指针没有问题，但auto不可和本例中的隐形代理类混用，不能做如下操作：

```c++
auto someVar = "隐形"代理类别表达式;
```

使用隐形代理类的库往往会在**文档里和头文件签名**中写明，以此避免

但也不必就此放弃auto，大可以利用`static_cast`来解决(标题中带显式型别的初始化物习惯)，如下：

```c++
auto hightPriority = static_cast<bool>(features(w)[5]);
```

## 条款7：在创建对象时区分()和{}

大括号内在是**统一初始化**，在以下三个情景中：

1. 大括号指定容器的初始内容：

   ```c++
   std::vector<int> v{1,3,5};
   ```

2. 为非静态成员指定**默认初始化值**，这个也可以使用=，但不能使用()

   ```c++
   class Widget {
       ...
       private:
       	int x{0};      // x默认值为0
       	int y = 0;	   // y默认值为0
       	int z(0);	   // 失败
   }
   ```

3. uncopyable对象（比如`std::atomic`对象）可以采用大小括号初始化，但是不能使用=

   ```c++
   std::atomic<int> ai1{0};   // 可行
   std::atomic<int> ai2(0);   // 可行
   std::atomic<int> ai3 = 0;  // 失败
   ```

这就是为什么{}叫“统一初始化”，因为只有大括号能适用以上所有场合

大括号有个特性，禁止built-in型别之间进行**隐式窄化型别转换（narrowing conversion）**，比如下面代码：

```c++
double x,y,z;
...
int sum1{x+y+z}; // 错误！double的和可能不能用int表达    
int sum2(x+y+z); // 没问题，double被截断
int sum3 = x+y+z;// 同上
```

大括号还能避免**最令人痛苦的解析语法**：任何能解析为声明的都**优先**解析为声明，如下：

``` c++
Widget w1(10); // 传参10，调用构造函数
// 当试图没参数的Widget构造函数时
Widget w2();   // 编译器解析认为这是在 “声明”函数而非调用
Widget w3{};   // 成功调用无形参的构造函数
```

那么接下来就是缺点：

1. 前面也提到了，auto会把{}推导为`std::initializer_list`
2. 如果有一个或多个构造函数声明了具备`std::initializer_list`型别的形参，那么使用{}的初始化语法会**强烈的优先**选择这些构造函数

比如下面情况：

```c++
class Widget {
    public:
    	Widget(int i, bool b);
    	Widget(int i, double d);
    	Widget(std::initializer_list<bool> il); // 实际调用的版本
    ...
}

Widget w{10, 5.0}; // 报错：{}不允许窄化转换
```

只有在找不到任何办法让将实参转化为`std::initializer_list`模板中的型别时，编译器才会退而求其次，检查不同的重载版本

比如上述版本，如果`std::initializer_list<bool>`换成`std::initializer_list<std::string>`时，因为10和5.0无法转换为string，编译器才回去选择其他重载版本的构造函数

但有个特例，那就是使用空{}，这时有两种理解：1. 没有实参，执行默认构造函数；2. 理解为空`std::initializer_list`，执行对应的构造函数。但这种情况下，会认为是没有实参，执行第一种默认构造函数

如果是希望使用空`std::initializer_list`的构造函数，可以用以下写法：

```c++
Widget w4({});
```

所以作为类的设计者，必须认知到`std::initializer_list`的特性，并且注意最好把函数设计成无论使用者使用大括号还是小括号都不会影响的版本。同时创建对象时也要三思后行

## 条款8：优先选用nullptr，而非0或NULL

C++的基本观点还是0是int，如果在使用指针的语境中出现了一个0，也会勉强把他解释为空指针而已；NULL类似，只不过是赋予非int类型的0值。总之这俩都**不具备指针型别**

C++98中，以上关键点可能对int和ptr型别重载的时候有影响

nullptr的优势就在于它**没有整型型别**，准确来说也不具备指针型别，但可以把它想象成任意type指针。其实际type是`std::nullptr_t`，可以隐式转换到所有的裸指针type

P63给出了一个template根据0，NULL，nullptr推导类型不同的例子

## 条款9：优先选用别名声明，而非typedef

理由其一，好理解，如下两写法其实功能一样，但using明显更好理解

```c++
typedef void(*FP)(int, const std::string&);
using FP = void(*)(int, const std::string&);
```

压倒性的理由其二，using支持template，typedef不行（需要嵌套在struct中硬来），如下：

```c++
// 使用using
template<typename T>
using MyAllocList = std::list<T, MyAlloc<T>>;

MyAllocList<Widget> lw;

// 使用typedef，几乎需要从头做一遍
template<typename T>
struct MyAllocList {
    typedef std::list<T, MyAlloc<T>> type;
};

MyAllocList<Widget>::type lw;
```

此外，在模板内使用typedef创建一个模板对象，且模板参数是由模板形参决定时，需要给typedef的名字加上一个typename前缀，如下

```c++
template<typename T>
class Widget {
    private:
    	typename MyAllocList<T>::type list;
    	...
}
```

但如果是using，那么写typename的要求就消失了

```c++
template<typename T>
class Widget {
    private:
    	MyAllocList<T> list;
    	...
}
```

原因很简单，因为using是一个别名模板，必然是一个type，是一个**非依赖性型别**；但对于`MyAllocList<T>::type`而言，编译器**不确定**它是否是一个型别，有可能在某个特化版本中就不是，所以必须要在前面加一个typename

但C++11的一些std函数，如：

```c++
std::remove_const<T>::type;
```

也是使用在struct中内嵌typedef的手法，所以当我们在template中使用这些函数时，同样也需要在前面加上typename

到了C++14才有了对应的using模板，如下：

```c++
std::remove_const_t<T>; //与上面的等价
```

## 条款10：优先选用限定作用域的枚举型别，而非不限作用域的枚举型别

正好这里复习一下enum的scope

```c++
// 这种写法，Color内部作用域延续到花括号外，整个作用于不可重名
enum Color { black, white, red}; // 默认{0,1,2}
// 这种写法限定作用域的enum，作用域只在enum花括号内
enum class Color {...}; 
```

上面体现了限定作用域enum的第一个优势：**降低命名空间的污染**

第二个压倒性的理由：枚举量是**强类型的（strongly typed）**。不限作用域enum中的枚举量（枚举成员）可以被隐式转换为int，并可以进一步->float，这样会产生很多问题。而限定作用域的enum不会有任何隐式转换路径（当然，如果非要转换，也可以通过`static_cast`这样的强制转型来）

第三个优点是：限定作用域enum可以进行**前置声明**。不限定版本也可以，但需要额外工作。因为一切enum型别在C++里都会选择int作为底层type，对于Color这样的三个值会选择char作为底层type。而一些取值范围更大的，如下面的enum:

```c++
enum Status {
    good = 0,
    failed = 1,
    indetermined = 0xFFFFFFFF
};
```

为了**节约内存**，编译器通常为enum选一个足够的最小type

这也是为什么限定作用域的enum可以前置声明，因为它的底层类型被使用前就已知（默认int），但不限定enum可以指定。当然，如果默认的int不符合我们目的，也可以如下推翻它：

```c++
enum class Status;                // 默认底层type为int
enum class Status: std::uint32_t; // 指定底层type为uint32_t
```

所以如果不限定作用域的enum加上了指定底层type的部分，就可以前置声明了：

```c++
enum Color: std::uint32_t;
```

前置声明的缺失最大的问题就是提升了编译依赖性，一旦改了值全世界都要重新编译。但如果有前置声明，就能完成声明式和定义式的分离（Effective里的）

非限定作用域enum有一个妙用就是和`std::tuple`的域名联动：

```c++
// 比如我们定义一个tuple的userinfo
using UserInfo = std::tuple<std::string,  // name
							std::string,  // email
							std::size_t>; // reputation
// 假设用户开始使用
UserInfo uInfo;
// 不使用非限制enum时
auto val = std::get<1>(uInfo);       // 取用域1的值，程序员需要记住域1对应email
// 使用enum
enum UserInfoFields { uiName, uiEmail, uiReputation}
auto val = std::get<uiEmail>(uInfo); // 这样使用省心多了
```

如果用限定作用域的enum还需要作用域访问符和强制转型，十分繁琐，如果想要调用更简洁，需要一个`std::underlying_type`取得（见P73）

## 条款11：优先选用删除函数，而非private未定义函数

把copy和赋值操作函数声明在private并不去定义他们，意味着该类不支持这些操作，这正是Effective C++里推荐的.....（如果成员函数或友元试图访问这些操作，linker阶段就会失败）

这本书更推荐=delete的写法，发现问题更早

**=delete函数声明在public原因**：因为如果用户试图访问，如果=delete函数是在private里，那么编译器报错会说**因为private禁止访问**，所以声明在public里，编译器会报**函数被删除，这样的报错信息更优秀**

=delete函数的重要优点：**任何函数都能成为=delete函数**，但只有**成员函数**能被声明为private

这样就可以选择性的筛去一些重载版本（P76）

还有一个妙用是**阻止不应进行的模板具现**，private声明写法做不到：

```c++
template<typename T>
void processPointer(T* ptr);

// 我们不希望有void*进而char*,使用=delete
template<>
void processPointer<void>(void*) = delete;
template<>
void processPointer<char>(char*) = delete;
```

private做不到的原因：不可能给予成员函数模板的某个特化以主模板不同的访问层级（模板特化必须在namespace里而非类作用域里）

当float被隐式转换到double或int时，优先选择double

## 条款12：为意在改写的函数添加override声明

override发生的条件：

1. 基类中的函数必须virtual
2. base和derived函数名必须完全相同（析构除外）
3. base和derived形参的类型和数量完全相同
4. base和derived函数常量性完全相同
5. base和derived返回值和异常规格必须兼容
6. C++11里新提出：引用修饰词（规定函数只能用于左值/右值）完全相同

这就是为什么需要override，这个关键字会提醒编译器细细的把以上的条件都check一遍，如果不加override，编译器只会认为derived定义了自己的函数，和base中的virtual函数无关

而且添加override之后，可以改掉基类虚函数的签名，然后就能看到有多少derived受到了影响，原本override函数无法编译

位置在函数最末尾：`virtual void mf3() && override`

&左值，&&右值。在类内表示这个类的对象(*this)为左值时调用该函数；这个类的对象为右值时调用

## 条款13：优先选用const_iterator，而非iterator

C++98太难用了：

1. 没有从iter->const_iter的便捷途径
2. 而且insert/erase的位置只能由iterator指定，不接受const_iterator

C++11都改掉了。C++提供了non-member版本的`begin,end`，但诸如`cbegin,cend,rbegin,rend,crbegin,crend`这些都只有member版本函数

当然，可以自己支持这点，如：

```c++
template <class C>
auto cbegin(const C& container) -> decltype(std::begin(container))
{
    return std::begin(container);
}
```

通过`const C& container`为iterator加上了常量性，其他的对应函数也能依次类推

## 条款14：只要函数不会发出异常，就为其加上noexcept声明

理由：

1. 事关接口设计。调用方可以查询函数的noexcept状态，和成员函数是否带有const是**同等重要**的，当明知一个函数不会throw异常却不加noexcept是**接口规格缺陷**

2. 生成更好的目标代码。可以从考察C++98和C++11的版本差异开始：

   ```c++
   int f(int x) throw();   // C++98
   int f(int x) noexcept;  // C++11
   ```
   
   以上两种都是不发异常，但在C++98的异常规格下，调用stack会开解至f的调用方，在执行一些无关操作后程序终止；在C++11异常规格下，程序终止前，stack只是**可能开解**
   
   但开解调用栈和可能开解调用栈对**优化**的影响很大：对noexcept版本而言，优化器不需要在异常传出函数的前提下，将执行期stack**保持在可开解状态**；也不需要在异常逸出函数的前提下，保证对象以构造顺序的逆序完成析构（说白了就是一些异常需要考虑的问题，优化器都不用考虑了，比如异常传出函数，异常逸出函数等情况不再考虑）

一个应用就是如C++ push_back中，即使使用移动操作也不会有异常，这里就可以用noexcept来延续copy操作引入的强异常安全保证（见P91底部的过程，取决于移动构造函数是否noexcept）

但有意思的是，std中的swap是否带有noexcept声明，取决于用户定义的swap是否带有noexcept声明，例如std为std::pair准备的swap函数：

```c++
template <class T, size_t N>
void swap(T (&a)[N],
          T (&b)[N]) noexcept(noexcept(swap(*a, *b)));

template <class T1, class T2>
struct pair {
    void swap(pair& p) noexcept(noexcept(swap(first, p.first)) && 
                                noexcept(swap(second, p.second)));
    ...
}
```

noexcept表达式的声明，他们是否具备noexcept性质取决于内部的表达式结果是否为noexcept(有点逻辑与的意思)

事实上大多是函数都是异常中立（exception-neutral）的，即自身不throw异常但调用的函数会throw，这些函数都**不是noexcept**的

注意不要强行把会抛出except的函数做noexcept实现，很愚蠢

C++11中把operator delete/delete[]以及所有析构函数都隐式地声明为noexcept（标准库里都是）

即使noexcept函数内部调用了没有noexcept保证的函数，编译器也不会warning，因为有可能是C-style或C++98的等地方移植来的代码

## 条款15：只要有可能使用constexpr，就使用它

有constexpr的函数，即不能断定是const函数，也不能假定其值在编译期就已知（传入的参数是**编译期常量**，就产出编译期常量，传入的参数含有运行期值就和普通函数一样产出运行期值）

所有的constexpr对象都是const对象，且编译期已知，在只读内存中创建

C++11中constexpr函数**不能包含多于一个执行语句**，也就是return，C++14放宽了

C++14中constexpr唯一的限制就是只允许传入和返回字面类型（built-in类型和constexpr的自定义type，见P98自定义constexpr构造函数）

通常constexpr函数里不允许有I/O语句，只要可能就使用constexpr，这宣告的是“但凡C++要求使用一个常量表达式的语境都可以用我”

## 条款16：保证const对象的线程安全性

这个条款的原理我个人比较熟悉了，就是mutex和原子操作那一套，这里只把C++里的一些示例做一下笔记

如果成员函数是const而成员变量有mutable，编译器不会对多线程报警，即使是const成员函数也会有race condition

mutex用法示例（类内）：

```c++
public:
	RootsType roots() const {
        std::lock_guard<std::mutex> g(m); // 锁住mutex
		if (...) {...}
    	return ...;
    }
private:
	mutable std::mutex m;
	...
```

注意C++里不需要显式的解锁，但该例中一定要是mutable因为如果不是这个m会被认为const进而无法在roots()函数内改变

`std::mutex`和`std::atomic`都是**只移型别（move-only type）**，作为类的成员变量定义会使类失去可复制性，但保留可移动性

## 条款17： 理解特种成员函数的生成机制

特种函数就是C++会自动生成的函数（C++ primer里的合成函数），包括默认构造，析构，复制构造和复制运算符

只有一个类没有声明任何构造函数时，编译器才会自动生成（被需要时才生成），且特种函数都是public，inline的，除了析构以外都是non-virtual函数。当base为virtual析构函数，编译器为他的derived class生成析构函数时，生成的是virtual析构

C++11中移动构造/赋值也加入了特种函数，移动操作在不支持移动的对象上本质是复制

两种复制操作是**独立的**（声明一个不会影响编译器生成另一个）；但两个移动操作**不是相互独立的**（声明了一个，会阻止编译器生成另一个）。

这个地方编译器的考虑是：既然生成了用户自己的移动操作函数，那意味着用户版本和编译器生成的版本有差距，所以编译器会不再特意为用户生成了。更进一步推到**一旦显式声明了复制操作**，编译器也不会再为用户生成移动操作了，理由和上述类似

反过来也一样，一旦自定义了移动操作，编译器也会拒绝合成移动操作（定义为=delete）

指导原则**大三律（Rule of Three）**：

​	如果声明了复制构造，复制赋值，析构函数中的任意一个，就得同时声明所有3个。因为如果有改写	复制的操作，意味着类需要执行某种资源管理，这也意味着：

1. 一种复制操作中用到的资源管理，另一种复制操作也需要
2. 析构函数通常也会参与到资源管理中来

这也意味着，如果用户定义了自己的析构函数，编译器也不会为其生成复制操作（理论上，C++11并没这么做，因为C++98没这么做，怕破坏老代码）。但C++11会禁止生成移动操作

如果非要用编译器自动生成，可以显式的用=default，这种手法对于多态基类很有用

如果是**复制/移动操作函数模板**，**不会**影响编译器生成特种函数

## 条款18：使用std::unique_ptr管理所有具备所有权的资源

`std::unique_ptr`默认和裸指针有着相同的尺寸，甚至大多数操作都是一个指令。这意味着甚至可以在内存和时钟紧张的情况下使用

`std::unique_ptr`是一个只移型别，默认的资源析构是通过对`std::unique_ptr`内部的裸指针实施delete完成的；当所有权链异常时，`std::unique_ptr`会调用该资源的析构函数（违反noexcept异常规格或调用`std::abort`局部对象不会析构）

`std::unique_ptr`可以使用custom deleter，调用时可以使用lambda或函数对象，当使用自定义的deleter时，尺寸一般会增加一到两个字长（word）。

当析构器是函数对象，尺寸的变化取决于函数对象存储了多少状态，无状态的函数对象（如无捕获列表的lambda）不浪费额外的尺寸，这也意味着deleter可以用函数也可以用无捕获的lambda实现时，无捕获的lambda更理想

`std::unique_ptr`有两种形式：`std::unique_ptr<T>`单个对象以及`std::unique_ptr<T[]>`数组避免二义性。API也被设计成与使用形式相配：单个对象不提供operator[]；数组不提供operator*和operator->（拒绝隐式数组->裸指针转换）

`std::unique_ptr`还可以高效的隐式转换为`std::shared_ptr`，这也是它适合作为工厂函数返回值的原因之一

## 条款19：使用std::shared_ptr管理具备共享所有权的资源

引用计数的存在会带来性能影响：

1. `std::shared_ptr`的尺寸是裸指针的两倍。一个裸指针指向资源，一个指向引用计数
2. 引用计数的内存**必须动态分配**。条款21解释如果用`std::make_shared`创建可以避免动态分配的成本
3. 引用计数的递增和递减必须是原子操作。

和unique_ptr类似，`std::shared_ptr`也可以自定义deleter。但对于`std::unique_ptr`而言，deleter是智能指针型别的一部分；对于`std::shared_ptr`则不是。`std::shared_ptr`更有弹性，假设有俩`std::shared_ptr`如下：

```c++
auto customDeleter1 = [](Widget *pw) {...}  
auto customDeleter2 = [](Widget *pw) {...}  // 两deleter不同type

std::shared_ptr<Widget> pw1(new Widget, customDeleter1);
std::shared_ptr<Widget> pw2(new Widget, customDeleter2);
```

由于pw1和pw2是一种型别，可以被放置到同一个容器中：

```c++
std::vector<std::shared_ptr<Widget>> vpw{pw1, pw2};
```

pw1和pw2甚至可相互赋值，也都可以被传递到`std::shared_ptr<Widget>`的形参中。`std::unique_ptr`不行，因为deleter作为一部分**会影响其型别**。所以也能看出，deleter**不会影响**`std::shared_ptr`的尺寸

前面提到的指向引用计数的指针，实质上引用计数是**控制块这个更大数据结构的一部分**，控制块除了包含引用计数以外，还可能包含自定义deleter的一个复制以及`std::weak_ptr`（P123图）

控制块的创建遵循以下规则（总的来说就是第一个`std::shared_ptr`对象负责创建控制块）：

1. `std::make_shared`总是创建一个控制块
2. 从**具备专属所有权的智能指针**如`std::unique_ptr,std::auto_ptr`创建一个`std::shared_ptr`时也会创建一个控制块
3. `std::shared_ptr`的构造函数使用**裸指针作为实参**时

第三点意味着，如果**一个裸指针创建了多次`std::shared_ptr`**对象，那么就会有**多个控制块**，引用计数完全分开，且析构时被多次析构！！！

有关裸指针的两个教训：

1. 避免将裸指针传给`std::shared_ptr`的构造函数，可以使用`std::make_shared`（如果自定义deleter就用不了了）
2. 如果非要传裸指针，就直接用new返回的结果，之后可以用得到的`std::shared_ptr`去初始化其他的

P126详细介绍了如果想要把类本身加入到`std::vector<std::shared_ptr<Widget>>`这样的容器中时，可以让类继承`std::enable_shared_from_this`，再通过调用其成员函数`shared_from_this`来获得一个**不重复创建控制块**的`std::shared_ptr`

之前说了`std::unique_ptr`可以方便的转化为`std::shared_ptr`。但反过来不行，`std::shared_ptr`和其指向的资源是**至死方休**的

注意`std::shared_ptr`不能处理数组，不像unique有`std::unique_ptr<T[]>`，`std::shared_ptr`不支持operator[]，deleter也要自己提供。如果`std::shared_ptr`指向数组，那是一种很拙劣的设计

## 条款20：对于类似std::shared_ptr但有可能空悬的指针选择std::weak_ptr

`std::weak_ptr`和`std::shared_ptr`一样方便但不参与管理所有权

`std::weak_ptr`并不是独立的智能指针，而是`std::shared_ptr`的一种扩充。一般`std::weak_ptr`会通过`std::shared_ptr`来创建

expire()函数可以验证指针是否空悬，但是有race condition，比如最后一个`std::shared_ptr`正在析构，还未减少计数，然后expire()判定失败试图使用，使用时才发现是空悬指针，行为undefined

所以需要atomic**操作**

1. `std::weak_ptr::lock`, `wp.lock()`如果不空悬返回任意一个`std::shared_ptr`，如果空悬返回一个nullptr

2. 还可以通过`std::weak_ptr`构造一个`std::shared_ptr`，如下：

   ```c++
   std::shared_ptr<Widget> spw3(wp);
   ```

   如果`std::weak_ptr`是空悬指针则失败，抛出异常

`std::weak_ptr`可以用在比如Widget对象都有对应的uuid，然后对象需要有一个缓存管理器管理这些对象，这时可以用到`std::weak_ptr`来校验存储的对象是否被释放（检测空悬的功能），如下：

```c++
std::shared_ptr<const Widget> fastLoadWidget(WidgetID id) {
    static std::unordered_map<WidgetID,
    						  std::weak_ptr<const Widget>> cache;
    auto objPtr = cache[id].lock();
    if (!objPtr) {
        objPtr = loadWeight(id);   // 对象不在缓存里就加载并缓存（缓存未命中）
        cache[id] = objPtr;
    }
    return objPtr;
}
```

由于失效的`std::weak_ptr`会不断积累，可以使用**观察者模式**

P132又一`std::weak_ptr`应用的例子：A,B 使用`std::shared_ptr`指向C，当C试图指向A时，用什么指针呢？

1. 裸指针（不行）。如果A被析构，B试图通过C使用A，就会产生undefined行为
2. `std::shared_ptr`（不行）。环形引用，A,B无法被自动回收，直接内存泄漏
3. `std::weak_ptr`。以上的问题都没有，也不会影响引用计数

使用`std::weak_ptr`打破`std::shared_ptr`的环形回路**不是**特别常用的做法。如果是树这样等级严格的谱系，子节点一般只被父节点拥有，且父节点被析构子节点也被析构，可以用`std::unique_ptr`从父指向子，子指向父则可以用裸指针

当然在非严格谱系中，以及缓存和观察者列表这样的实现下`std::weak_ptr`还是不错的

`std::weak_ptr`也和`std::shared_ptr`一个尺寸，也指向控制块，只不过指向的是**第二个引用计数，弱计数**

## 条款21：优先选用std::make_unique和std::make_shared，而非直接使用new

C++14才有的`std::make_unique`，但基础版本实现并不难，见P133

第三个make函数是`std::allocate_unique`，使用allocator作为对象，这里不是重点

第一个优点，避免冗余：

```c++
auto upw1(std::make_unique<Widget>());
std::unique_ptr<Widget> upw2(new Widget); // 不用make会把类型写两次
```

第二个优点，异常安全：

如果对以下函数传值：

```c++
void processWidget(std::shared_ptr<Widget> spw, int priority);
// 使用new传值，内存泄漏
processWidget(std::shared_ptr<widget>(new Widget), priority());
```

和Effective C++上的案例一样，C++和Java，C#这样的语言不同，在执行的顺序上是未知的，如果：

1. new生成一个Widget对象
2. 执行priority()函数
3. 最后把Widget对象构造成`std::shared_ptr`

有可能在第二步抛出异常导致第一步的new对象无法被释放，就内存泄露了

`std::make_shared`就没有以上问题

还有就是`std::make_shared`更快，比如

```c++
std::shared_ptr<widget>(new Widget)
```

这个语句实质上涉及了两次内存分配（new的一次和关联控制块的一次分开），`std::make_shared`就一次

但make也有缺点：

1. 所有的make都不支持自定义deleter
2. 不能完美转发**大括号初始物**

还有就是在自定义operator new和operator delete时，精准分配处理一个`sizeof(Widget)`大小时，不适合使用make（使用更多空间）

控制块在对应的`std::make_shared`产生的最后一个`std::shared_ptr`（引用计数）和最后一个指向它的`std::weak_ptr`（弱引用）被析构之前都无法释放。

这意味着如果最后一个`std::shared_ptr`被析构和最后一个`std::weak_ptr`被析构中间的延迟，可能会造成资源的占用。如果使用make，那么`std::shared_ptr`**指向的对象也和控制块是一个生存周期**

但如果是new表达式产生的`std::shared_ptr`，可以在最后一个`std::shared_ptr`析构时**就释放指向的对象**，只留着控制块等待最后一个`std::weak_ptr`被析构

如果用new，就保证立刻把new的结果传给智能指针的构造函数，**不要再做其他的事**导致内存泄漏

## 条款22：使用Pimpl习惯用法时，将特殊成员函数的定义放在实现文件中

Effective C++老生常谈的接口和实现分离的Pimpl写法（减少编译依赖性）...简单来说就是把基类中的某个数据成员用一个指向它的具体实现的函数/类的指针来替代

第一部分是声明一个指针type的数据成员，指向一个非完整type(Pimpl)；第二部分是**动态分配和回收原本在类内的数据成员的对象**，放在实现文件中

这里的Pimpl可以使用智能指针

注意P144的例子，析构函数需要出现在Impl这个struct被定义前被声明，再在Impl的定义后被定义，因为默认析构函数隐式inline，它需要在被**创建前看到被析构对象是一个完整的对象**。移动函数也是，因为移动操作涉及到先把原本的Impl对象析构再移动

但如果使用的是`std::shared_ptr`，就不需要上面这些限制，因为对`std::unique_ptr`而言，deleter是它的一部分，这使得编译器产生更小的运行期数据结构和更快的代码。代价就是**如果要使用编译器生成的特种函数，其指向的type必须是完全型别**。但`std::shared_ptr`虽然占用大，速度慢，却不要求指向的需要是完整型别

## 条款23：理解std::move和std::forward

`std::move`和`std::forward`都不会生成任何可执行代码，连一个字节都不会生成

`std::move`和`std::forward`都仅仅是执行强制类型转换的函数（模板）。`std::move`**无条件**将实参转换为右值；`std::forward`在满足**特定条件时**执行同一种强制转换

`std::move`的本质：`static_cast<remove_reference<T>::type&&>(param)`，确保返回右值引用，**本质是强制类型转换而非字面意思的移动**

如果想去的某个对象移动操作的能力，**不要声明为常量**，不然会隐式地执行copy操作

`std::move`不实际移动任何东西，也不保证强制类型转换后的对象具备移动能力，只保证结果是个右值

`std::forward`有如下例子：

```c++
void process(const Widget& lvalArg);  // 左值重载版本
void process(Widget&& rvalArg);		  // 右值重载版本

template<typename T>
void logAndProcess(T&& param) {
    ... // log部分
    process(std::forward<T>(param));
}
```

这里因为所有函数形参都是左值，param也不例外，如果不做处理永远用不了process函数右值的重载版本，这时使用`std::forward`，当T为右值时，才会强制转换为右值

理论上`std::move`完全可以被`std::forward`取代掉，但`std::move`也有优势：方便，减少错误可能，更清晰

## 条款24：区分万能引用和右值引用

`T&&`有两种不同的含义，一种是右值引用，另一种就是万能引用（既可以是左值也可以是右值，const与否和volatile与否都可以用）

万能引用会在两种场景下现身：

1. 函数template的形参，如下：

   ```c++
   template<typename T>
   void f(T&& param);    // param是个万能引用
   ```

2. 第二为auto声明，如下：

   ```c++
   auto&& var2 = var1;  // var2是个万能引用
   ```

万能引用的共通处就是**都涉及型别推导**，如果不涉及型别推导，那么就是右值引用，如下：

```c++
Widget f(Widget&& param);  		  // param是个右值引用

template<typename T>
void f(std::vector<T>&& param);   // param是个右值引用
```

万能引用转换的param首先是个引用，然后根据引用折叠，如果初始化物是左值->param为左值引用；初始化物为右值->param为右值引用

如果想让引用成为万能引用，**涉及type推导是一个必要条件，但不充分**。引用声明的形式也是如`T&&`这样限定死的，如上例中`void f(std::vector<T>&& param)`，`std::vector<T>&&`不是`T&&`的形式，就会产生右值引用。这是传一个左值引用编译器会报错：

```c++
std::vector<int> v;
f(v);                     // 报错，不能给右值引用绑定左值
```

 即使是一个const存在，万能引用也不成立：

```c++
template<typename T>
void f(const T&& param);  // param是一个右值引用
```

但也不是满足了`T&&`形式且在template内就是万能引用

```c++
template<class T, class Allocator = allocator<T>>
class vector {
    public:
    	void push_back(T&& x);    // 这个不是万能引用！！！
}
```

这里不是万能引用的原因：push_back实际是属于一个已经推导出type的实例，而不是在push_back处进行推导

而对于`auto&&`，全都是万能引用，因为一定涉及到type推导

## 条款25：针对右值引用实施std::move，针对万能引用实施std::forward

如果形参类别是右值引用，则绑定的对象一定可供移动，当我们需要保留右值性时，就可以用到`std::move`

就像前面说到的，形参类别为万能引用是并**不保证对象一定可移动**，所以这就可以用到`std::forward`

对万能引用使用`std::move`可以让人抓狂：

```c++
class Widget {
    public:
    	template<typename T> 
    	void setName(T&& newName) {  // 万能引用，糟糕的设计
            name = std::move(newName);
        }
    private:
    	std::string name;
    	...
};

std::string getWidgetName();         // factory函数
Widget w;
auto n = getWidgetName();
w.setName(n);                        // 移入w后n的值未知
```

该例中局部变量n传入w.setName()，使用者也会假定这是在传副本，认为这是一个只读操作。但使用`std::move`使得n无条件转为右值，n被移入w.name，n的值就不确定了

如果不使用万能引用而使用重载版本，如下：

```c++
class Widget {
    public:
    	void setName(std::string&& newName) {      // 传入右值
            name = std::move(newName);
        }
    	void setName(const std::string& newName) { // 传入左值
            name = newName;      
        }
    private:
    	std::string name;
    	...
};

// 调用
w.setName("ABC");

```

使用万能引用：字面值"ABC"被传给setName，然后传给内部std::string赋值（直接从字面得到赋值）

使用重载版本：先产生一个临时变量以供形参绑定再通过移动操作给内部的w.name，最后再把临时量销毁

所以重载版本**消耗一定大**一些

还有就是如果有多个形参，那么万能引用还是一个版本，重载版本则需要2^n个，包括函数模板会有无数个形参，都是只能使用万能引用的

在return-by-value的函数中，如果**返回的是绑定到一个右值引用或万能引用的对象（输入实参）**，那么返回时使用`std::move`或`std::forward`（P165老生常谈的矩阵运算，不用这俩函数转右值就会copy一份降低效率）

但注意一定不要把这俩函数用到局部变量上。关于这点，书中提到了**返回值优化（return value optimization, RVO）**：当编译器要在一个按value返回的函数里省略对局部对象的copy/move，需要满足两个前提：

1. 局部对象type和函数返回值type一致
2. 返回的就是局部对象本身

所以编译器对于局部变量的返回其实是会**自动优化成无copy的版本的**，要么复制省略，要么隐式地调用`std::move`施加在返回对象上

一旦使用`return std::move(res)`或者`std::forward`版本，返回的就**不是局部对象的本身**，而是其引用，所以RVO这时并不满足

## 条款26：避免依万能引用型别进行重载

P170例子，一旦万能引用成为了重载版本函数之一，它几乎可以**精准匹配所有的类型**，所以会吸走大批的实参型别，往往比预想的多得多

P172给出了一个天怒人怨的例子：

```c++
class Person {
    public:
    	template<typename T>
    	explicit Person(T&& n)      // 完美转发构造函数（雷点）
    	: name(std::forward<T>(n)) {}
    	explicit Person(int idx);
    	
    	// 因为构造函数是模板，不会影响编译器自动合成复制和移动操作
    	// 编译器自动生成的复制和移动构造函数可以理解为如下:
    	Person(const Person& rhs);	// 复制构造函数
    	Person(Person&& rhs);       // 移动构造函数
};

// 一个使用案例
Person p("Nancy");
auto cloneOfP(p);                   // 试图通过p创建新的Person对象，无法过编译
```

使用案例过不了编译的原因很简单，因为底层调用了完美转发构造函数而不是复制构造，因为这里p是一个`Person& rhs`而非`const Person& rhs`，编译器会选择有万能引用的构造函数，这时，我们试图用Person类型的对象p给name赋值，自然就会失败了

如果使用`const Person p("Nancy");`初始化，那么就会正常调用复制构造函数，成功

包括如果试图用基类初值表初始化时，也会使用基类中的万能引用构造函数（P174）

## 条款27：熟悉依万能引用型别进行重载的替代方案

#### 舍弃重载

命名成多个不同的名字，不是长久之计

#### 传递const T&型别的形参

这就是条款26的第一种问题的解决方式，也分析过了，分别重载左值和右值会导致必然造成拷贝，进而效率下降，是一种值得权衡利弊的手段

#### 传值

把传递的形参从传reference改成传value，条款41会再讨论效率问题（我就觉得传value必复制效率好不起来）

#### 标签分派

如果使用万能引用是为了**完美转发**，那就只有万能引用一条路

可以让万能引用仅仅成为**形参列表的一部分**，表中还有其他的非万能引用参数，只要完美转发函数具备**充分差(sufficiently poor)**的匹配能力就可以避免重载问题

P177提供了一个通过本质在`logAndAddImpl`这个实现函数处重载的版本，`logAndAdd`使用万能引用接受所有实参，再让实现函数根据是否是int来判定函数的使用。注意这里使用`std::true_type`和`std::false_type`来在编译期完成版本选择

而这种设计中`std::true_type`和`std::false_type`就是所谓的**标签**，存在的目的就是强制重载选择向我们希望的方向推进。注意这些形参甚至**没有名字，在运行期没有任何作用**

#### 对接受万能引用的模板加以限制

这里出现了新的问题，那就是编译器自动合成的构造函数可能直接**绕过了标签分派系统**。但这其实不是问题，最怕的是编译器自动合成的构造函数**不一定保证能绕过标签分派系统**，比如上个条款26中提到的**非常量左值初始化**，会直接给到万能引用的函数去使用

解决方法是使用`std::enable_if`。`std::enable_if`可以强制编译器表现出来的行为**如同特定模板不存在一般**，这样的模板被称为**禁用**。默认所有模板都是enable的，实施了`std::enable_if`的模板只有在满足指定条件时才enable

语法例子（具体原理看”SFINAE“）：

```c++
class Person {
    public:
    	template<typename T,
    			 typename = typename std::enable_if<condtion>::type>
        explicit Person(T&& n);
}
```

但这里注意条件判断时要把1. 是否是引用；2. 是否是const；3. 是否是volatile都考虑进去。这里使用`std::decay`把以上三个属性**全都移除**（decay也可以用于把数组和函数类型**强制转换为指针类型**），具体的条件判断就交给`std::is_same`来完成，最终结果如下

```c++
// !std::is_same<Person, typename std::decay<T>::type>::value
class Person {
    public:
    	template<typename T,
    			 typename = typename std::enable_if<
                     !std::is_same<Person,
    							   typename std::decay<T>::type
                     			   >::value
                 >::type
        >
        explicit Person(T&& n);
}
```

最后就是条款26也提到的derived class使用copy和移动操作时，会被万能引用劫持的问题

这里就该使用`std::is_base_of`来判断了，改进后的版本如下：

```c++
class Person {
    public:
    	template<typename T,
    			 typename = typename std::enable_if<
                     !std::is_base_of<Person,   // 可以同时判断dervied和base自身
    							      typename std::decay<T>::type
                     			      >::value
                 >::type
        >
        explicit Person(T&& n);
}
```

最后再加上int部分，也就是前一方法中使用标签分派解决的部分

```c++
class Person {
    public:
    	template<typename T,
    			 typename = typename std::enable_if<
                     !std::is_base_of<Person,   
    							      typename std::decay<T>::type
                     			      >::value
                     &&
                     !std::is_integral<
                                      typename std::remove_reference<T>::type
                                      >::value
                 >::type
        >
        explicit Person(T&& n);
}
```

#### 权衡

前三种（舍弃重载，传递const T&型别的形参和传value）都需要对函数形参逐一指定type；后两种（标签分派和enable施加资格）则利用了完美转发，不需要指定形参type

完美转发效率更高，但也有缺点：1. 一些type无法完美转发（条款30）2. 传递非法形参时，错误信息难以理解（这也是为什么很多程序员不适用万能引用形参的原因）

P186使用`static_assert`和`std::is_constructible`来判定和报错，但这些错误信息也会混杂在长串的errorMsg中

## 条款28：理解引用折叠

C++ primer里也详细讲过，这里仅作补充

引用的引用是**非法的**，所以会产生引用折叠，P189中`std::forward`的实现和C++上的大同小异，如下：

```c++
template<typename T>
T&& forward(typename remove_reference<T>::type& param) {
    return static_cast<T&&>(param);
}
```

这里提供的分析相比于C++ primer更详细，但我当时推过了这里不再赘述

引用折叠的语境有**四种**：

1. 模板实例化
2. auto变量的type生成，如`auto&& w1 = w2;`
3. 生成和使用typedef和别名声明（using）
4. decltype的运用中

## 条款29：假定移动操作不存在，成本高，未使用

从为什么许多类别不能支持move的语义开始：C++11翻修了C++98，但老代码很多没有move支持，即使编译器能合成，也基于代码没有自定义复制，移动和析构函数。即使对于C++11，所有容器都支持move，但对有些容器而言，move根本没有更低廉；对于另一些容器，虽然低廉，但又有一些附加条件使得容器元不能满足

比如`std::array`(P194)，其他容器都是存储在堆上，move的时候把指向对象数据的指针一一复制就可以达到o(n)，但`std::array`本质还是旧式数组，都是直接存储数据，move并不会快（比move指针可慢多了）

作为对比，`std::string`可以o(1)移动以及o(n)的复制效率，**像是说移动必比复制快，实际未必**。许多string的实现都使用了**小型字符串优化（small string optimization， SSO）**。采用SSO后，小型字符串（容量不超过15）会存储在`std::string`的某个缓冲区内而不是使用heap上的储存，这时对小型字符串的移动就不会比复制更快

总之有以下几个场景，move并没啥好处：

1. 没有移动操作：对象不提供，move本质成copy
2. 移动未能更快：上例的`std::string`
3. 移动不可用：本可以发生的语境上，比如要求noexcept但move操作没有，就使用满足noexcept的复制操作
4. 撰写template时：必须想C++98一样保守的使用copy，因为不知道哪些type能用move哪些不能，会造成**不稳定**的代码

## 条款30：熟悉完美转发的失败情形

转发函数天然泛型，例子如下：

```c++
template<typename... Ts>
void fwd(Ts&&... param) {
    f(std::forward<Ts>(params));
}
```

当给定目标函数f和转发函数fwd，如果**直接把实参给f**和**先给fwd再给f**（上面实例代码）的结果不一样，那么称为**完美转发失败**

#### 大括号初始化物

比如：

```c++
void f(const std::vector<int>& v);

f({1,2,3});
fwd({1,2,3});
```

当直接使用f时，编译器会比较类型是否兼容，找隐式转换，并创建一个临时的vector传给f

但使用转发间接调用时，编译器就不会比较fwd转发的实参和f中形参的兼容性了。取而代之用推导的手法，完美转发会在以下两个条件任意一个成立时失败：

1. 编译器无法为一个或多个fwd的形参推导出type结果（编译不通过）
2. 编译器为一个或多个fwd的形参退出了**错误的**type结果。这里的错误既可以指fwd根据推导结果实例化通不过编译，也可以指fwd和f的type行为不一致（因为f可能有重载版本）

本例中由于fwd的形参未声明为`std::initializer_list`，编译器会禁止调用过程中从{1，2，3}开始推导

但auto可以绕行，因为auto会把类型推导为`std::initializer_list`：

```c++
auto il = {1,2,3};
fwd(il);                // 完美转发成功
```

#### 0和NULL用作空指针

很简单，之前也提到过被推导成整型，一定记得用nullptr

#### 仅有声明的整型static const成员变量

有个规定：不需要给出类中static const成员变量的定义，只需声明即可。因为编译器会根据这些成员的值实施常数传播（简单的来说类似于C中宏的编译期替换），从而不需要保留它们的地址

static const在声明处定义可以供其他成员使用，但f和fwd就会产生完全不同的后果。因为fwd的形参是一个万能引用，引用的底层又是指针，也**通常被当成指针处理**，然后编译器又不给static const保留地址，就会报错。而传value的f函数并不会

#### 重载的函数名字和模板名字

如下情况：

```c++
void f(int (*pf)(int));

int processVal(int value);
int processVal(int value, int priority);

f(processVal);   // 成功
fwd(processVal); // 失败
```

能运行是因为**f的声明式**让编译器弄清了哪个版本的重载函数

但对于fwd无type根本无从推导，template也是类似的情况

想要解决只能**手动指定转发的重载版本或实例**，比如使用一个指针，如下：

```c++
using processFuncType = int (*)(int);
processFuncType processValPtr = processVal;

// 重载函数
fwd(processValPtr);
// 模板
fwd(static_cast<processFuncType>(workOnVal));
```

#### 位域

如下例：

```c++
struct IPv4Header {
    std::uint32_t version:4,
    			  ...
                  totalLength:16;
};

void f(std::Size_t sz);
IPv4Header h;

f(h.totalLength);    // 成功
fwd(h.totalLength);  // 失败
```

因为fwd的形参是一个引用，而h.totalLength是一个non-const的位域。C++有明确要求：**引用不能绑定到non-const位域（不能直接取址）**

这也导致任何接受位域实参的函数实际上**只会收到副本**，也可以通过下面办法完成位域的完美转发

```c++
auto length = static_cast<std::uint16_t>(h.totalLength);
fwd(length);
```

## 条款31：避免默认捕获模式

一些术语：

1. lambda表达式，就是源码部分
2. 闭包是lambda表达式创建的运行期对象，根据不同的捕获模式，闭包会持有数据的副本或引用
3. 闭包类旧式实例化闭包的类。每个lambda表达式都会触发编译器生成一个**独一无二**的闭包类。闭包中的语句就会成为闭包类成员函数的可执行指令（C++ primer也有样例）

两种捕获写法如下：

```c++
[&](int value) { return value % divisor == 0; }
[&divisor](int value) { return value % divisor == 0; }
```

虽然两种都不安全，使用局部变量的引用**极易造成空悬指针**，但后者显式地列出lambda式所依赖的局部变量或形参是更好的软件工程实践

C++14可以在lambda形参的声明中使用auto，如`[&](const auto& value){...}`

哪怕是使用值捕获如`[=](...) { ... }`，如果捕获的是指针的副本，也会造成空悬指针

捕获只能针对**lambda表达式所在作用域的非静态局部变量（包括形参）**

P209开始一直在讨论成员函数内的lambda表达式如何捕获一个类中的static变量，

1. 有捕获this然后指针的，但这样也可能会导致空悬，因为把对象emplace到一个filter中，尔后对象自身被删除就又又又空悬指针了

2. 也可以在成员函数内直接用局部变量copy一个，然后让lambda捕获

3. **广义lambda捕获（generalized lambda capture）**是更优秀的方案，如下：

   ```c++
   void Widget::addFilter() const {
       filters.emplace_back(
           [divisor = divisor](int value) { // 广义捕获把divisor copy进闭包
               return value % divisor == 0;
           }
       )
   }
   ```

默认值捕获另一个缺点是：静态数据的生存周期隐式的依赖于**静态存储周期（static storage duration）**，static数据可以在lambda中使用，却又不能被捕获，这会让人产生”他们已经被捕获“的错觉

## 条款32：使用初始化捕获将对象移入闭包

C++14的**初始化捕获**特性，也就是**捕获一个表达式的结果**，能让对象move进闭包，C++11不支持。

因为使用表达式使得捕获的概念得到了泛化，所以又被称为**广义lambda捕获**

比如P213的例子：`auto func = [pw = std::move(pw)](){...}`

在这个捕获中，位于=左右两侧的处于不同的作用域：左侧是闭包类成员变量的名字，位于闭包类作用域；右侧是初始化表达式，位于lambda定义所在的作用域

P214提供了一个直接把闭包类得本质内容实现出来的样例。但如果我们非要用lambda式来完成，按移动捕获，可以按照以下思路：

1. 把需要捕获的对象移动到std::bind产生的函数对象中
2. 给lambda式一个指向”欲捕获对象“的引用

例子如下：

```c++
std::vector<double> data;       // 想要move进闭包的对象

// C++14版本
auto func = [data = std::move(data)] {...}

// C++11的实现
auto func = std::bind(
    [](const std::vector<double>& data) { ... },
    std::move(data)
);
```

std::bind的第一个实参是callable object，接下来的所有实参都是传给这个callable object的实参，返回一个**绑定对象（binding object）**

对于每个左值实参，在绑定对象内都是copy构造；对于右值实参都是move构造。

默认lambda生成的闭包类中operator()成员函数会带有const饰词，但binding obj得到的data副本却不带const，所以要在lambda形参声明上加上const

如果lambda声明带有mutable，那么闭包里的operator()就不会在声明时带有const饰词，相应的，形参就要省略const，如下：

```c++
auto func = std::bind(
    [](std::vector<double>& data) mutable { ... },
    std::move(data)
);
```

闭包的生命周期和binding obj是相同的，因为闭包本就是binding obj的一部分

## 条款33：对auto&&型别的形参使用decltype，以std::forward之

