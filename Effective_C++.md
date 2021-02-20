该笔记计划以55要点为分界，包括开头的术语（terminology）在内共56部分

## 第0部分 术语

1. 函数的签名也就是函数的类型，即返回类型+参数 ，如`std:size_t (int, int)`（官方对签名的定义不包括返回类型）
2. 这本书的不明确行为=undefined（C++ Primer中的未定义行为，这本书翻译腔太重了，怕是直接扔到google translate过了一圈）
3. Widght（小部件）一般不代表任何东西，仅作示例
4. 构造函数ctor，析构函数dtor
5. pt表示pointer to T类型，rt同理表示reference to T类型
6. TR1(Technical Report 1)是一份规范，描述了加入C++ STL的诸多新机能，所有组件都被放在命名空间tr1内，并嵌套在std命名空间中
7.  Boost是一个组织/网站，提供可移植的，有审核的，开源的C++库。大多数TR1机能以Boost的工作为基础

## 条款01： 视C++为一个语言联邦

1. C++原本就是为C加上OOP特性，然后被渐渐支持多种范式，应该把C++视为由多个联邦（sublanguage）组成的国家，在某个sublanguage中，各种守则和通例都简单易懂
2. 主要的sublanguage：
   1. C。区块，语句，预处理器，内置数据类型，数组，指针都是C的东西。但C一堆东西没有...，条款2谈到预处理外的方法，13谈到对象管理原则
   2. Object-Oriented C++。这部分就是C with class理念追求的，OOP的那些特性，如：类，封装，继承，多态，虚函数等等
   3. Template C++。C++的泛型编程部分，条款48有概述
   4. STL。一个模板库。对container，iterators，algo，function objects的规约有着紧密地配合和协调
3. 对内置类型而言，pass-by-value往往比pass-by-reference效率更高。但如果有自定义构造/析构函数在，pass-by-reference-const往往更好。参数传递详见条款20

## 条款02：尽量以const，enum，inline替换#define

即尽量以编译器替换预处理器

因为#define宏指令在C语言的编译过程中，会在预处理部分把所有宏命令替换掉，假设有一个如下语句：

```c++
#define ASPECT_RATIO 1.653
```

有可能因为预编译过程中使`ASPECT_RATIO`直接被替换，导致编译过程中`ASPECT_RATIO`压根没有进入记号表

*p.s. 记号表参见[stackoverflow question](https://stackoverflow.com/questions/69112/what-is-a-symbol-table/69169), 大致就是 obj文件需要通过 linker把源文件中定义的数据和函数与程序相连，而obj文件有一个数据结构叫 symbol table，它把这些数据和函数 map成 linker能理解的形式。当函数调用时，C++不会直接把源码的最终地址给调用，而是给一个placeholder让 linker在各个obj文件的 symbol table里找对应的最终地址，并将最终地址放在 linker那*

所以当我们运行而得到错误或进行symbolic debug时，错误信息可能提到1.653而不是`ASPECT_RATIO`，导致难以debug

#### 解决方法：使用const

```c++
const double AspectRatio = 1.653;
```

优点：

1. 必定会被编译器看到，回避#define的问题
2. 比#define使用的空间小，因为预处理替换宏的时候很可能出现多个1.653浮点数，但是用const变量时，被引用到的只有一个1.653

文中提及的两个特殊性情况：

1. 定义常量指针指向常量，通常常量定义放在头文件中，需要写两次const

   ```c++
   const char *const authorName = "ABC";
   // 该例中，往往string都比char*对象更合适，所以可以写成如下形式
   const std::string authorName("ABC");
   ```

   *左边的const限制指向的对象是常量（底层），右边的const限制指针自身是常量（顶层）

2. class内的专属常量，为了确保唯一性和const，定义一个静态常量，如下：

   ```c++
   class Example {
       private:
       	static const int numTurns = 5;
       	int scores[numTurns];
       ...
   };
   ```

   然而这个是**声明式**而非定义式，一般C++要求对**所有**使用的东西提供一个定义式

   但当他是一个class专属const又是static还是整型，那么可以只声明不定义，如果非要定义，如下：

   ```c++
   const int Example::numTurns; // 定义式，因为已经在声明时获得初值，不需要再设
   ```

   注意无法用#define创造class专属常量，因为#define不重视scope，只要定义一直起效（除非手动#undef），不具备封装性。而const有

   如果旧式编译器不支持类内声明初始值，可以如下：

   ```c++
   class Example {
       private:
       	static const double numTurns;
       ...
   };
   const double Example::numTurns = 5.0;
   ```

   再如果编译器既不允许类内声明初始值，又像上述例子中`Example::scores[numTurns]`一样需要这个值，可以使用enum hack的trick

   ```c++
   class Example {
       private:
       	enum { numTurns = 5 };
       	int scores[numTurns];
       ...
   };
   ```

   enum的好处：

   1. 有些时候更像#define而不是const，或许这正是我们想要的（例如取enum和#define的地址一般都不合法，而取const对象的地址合法）
   2. 实用主义，很多代码用了，必须认识

##### 再回到#define

另一个常见误用就是用其实现macros宏。宏看起来像函数，但不会招致函数调用带来的额外开销，如下宏夹带着宏实参：

```c++
#define CALL_WITH_MAX(a,b) f((a) > (b)) ? (a) : (b)
```

**这样有无数缺点！！！**

1. 首先必须给所有实参加上小括号

2. 即使加上了也可能遭重，如下：

   ```c++
   int a = 5, b = 0;
   CALL_WITH_MAX(++a, b);    // a被累加两次
   CALL_WITH_MAX(++a, b+10); // a被累加一次
   // "++a"本身作为实参被调用两次:宏一次，这里一次
   ```

   所以直接用template inline函数，有宏的效率，又有一般函数的安全性

   ```c++
   template <typename T>
   inline void CallMaxWith(const T& a, const T& b) {
       f(a > b ? a : b);
   }
   ```

   既不需要给参数加括号，也保证安全性。

本条款总结，虽然做了这么多努力，但是有#include和#ifdef/#ifndef的存在，预处理器还在被使用，但我们应该有意识地不去使用预处理器

## 条款03：尽可能使用const

1. 再次重申：*左边的const限制指向的对象是常量（底层），右边的const限制指针自身是常量（顶层）

2. 声明迭代器为const和声明const指针一样（顶层const），需要用底层迭代器使用`const_iterator`

3. P20 `operator[]`的例子，注意返回的分别是`const char&`和`char&`，因为如果返回char，那么如同`tb[0] = 'x';`的写法就是错误的，因为内置类型传value本质传副本

4. `bitwise constness(physical constness)`和`logical constness`:

   1. bitwise: 成员函数只有在不更改对象的任何成员变量才叫const（即不改变一个bit），这种观点比较便于编译器侦测，只要侦测变量的赋值动作即可，这样意味着成员函数不能更改对象内**任何非静态成员变量**

   2. 但如果一个成员函数改变了”指针所指的内容“且只有指针（指向的内容不是）属于对象，那么这个成员函数即使不是十足的const也满足bitwise const

   3. 于是有了logical派：只要客户端检测不出，可以修改对象内的某些bits，即使用`mutable`关键字

   4. 当使用const和非const重载函数且有等价实现时，可以使用`const_cast`来避免重复代码（C++ primer的样例代码也用了相同的trick），如下：

      ```c++
      class TextBlock {
          public:
          	const char& operator[](std::size_t position) const {
                  /*
                  ...
                  */
                  return text[position];
              }
          	char& operator[](std::size_t position) const {
                  return const_cast<char&>(
                      static_cast<const TextBlock&>(*this)[position]
                  );
              }
      }
      ```

      `const_cast`处理返回结果，`static_cast`是为了避免递归调用非const自己的版本

      ”运用const成员函数实现出非const版本“

      注意在避免重复代码时，不要试图”运用非const成员函数实现出const版本“，因为在非const函数内调用const函数有可能**改变承诺不能改变的对象**(调用前把`const_cast<char&>`把*this的const属性去掉，很危险)

## 条款04：确定对象被使用前已被初始化

1. 读取未初始化的值会导致未定义行为

2. 如果使用的是C-language part，那么因为初始化可能造成运行成本，就不保证初始化；但如果进入non-C part就不一样了，这也就是了array(C的数组风格)不保证初始化，但是vector有保证

3. 不要混淆初始化和赋值：C++规定，对象成员变量的初始化动作发生在进入构造函数本体**之前**，如果如P27一样赋值，那么这个过程实际上是：1.在进入函数体前，对象已经被=default初始化，进入函数体后才又被赋值了

4. 使用member initialization list（初值列）：

   ```c++
   ABEntry::ABEntry(const std::string& name, const std::string& address,
                    const std::list<PhoneNumber>& phones)
   : theName(name),
     theAddress(address),
     thePhones(phones),
     numTimesConsulted(0),
   { } // 现在构造函数本体不必有任何动作
   ```

   只有一次初始化过程，不需要多余的赋值过程

   ```c++
   ABEntry::ABEntry(const std::string& name, const std::string& address,
                    const std::list<PhoneNumber>& phones)
   : theName(),
     theAddress(),
     thePhones(),
     numTimesConsulted(),
   { } // 现在构造函数本体不必有任何动作
   ```

   规定在初值列中列出**所有**成员变量，即使成员是内置类型，包括成员是const或者引用，都使用初值列

5. C++有着**固定**的成员初始化顺序，base class的初始化永远早于derived class，即使在初值列的出现顺序不符合也完全不影响实际的初始化顺序，所以最好**让声明的次序与实际初始化顺序一致**

6. 函数内的static对象被称为local static对象，位于其他的地方（类内，global，namespace，file内）都是non-local static

7. 编译单元（translation unit）指的是产出单一目标obj文件的源码，基本上是单一源码文件+所含入的头文件

8. 当有两个源文件，每个都含有non-local static对象时，一个编译单元的non-local static对象的初始化可能用到了另一个编译单元的non-local static对象，这时被用到的non-local static对象可能尚未初始化，因为C++对”不同定义单元内的non-local static对象“的初始化顺序没有定义（P30实例）

9. 第8点的解决方法：将每个non-local static对象搬到自己的**专属函数**内，且该对象在函数内也被声明为static，函数返回一个reference指向non-local static对象。这种手法的本质是用local static对象替换了non-local static对象（singleton的常用手法），如下：

   ```c++
   class FileSystem { ... };
   FileSystem& tfs() {
       static FileSystem fs;     // 定义并初始化一个non-local static对象
       return fs;                // 返回一个引用
   }
   
   class Directory { ... };
   Directory::Directory( params ) {
       std::size_t disks = tfs().numDisks(); // 使用专属函数访问non-local static对象
   }
   ```

## 条款05：了解C++默默编写并调用那些函数

编译器不能合成copy的三种情况

1. 成员含有引用。当拷贝引用时，因为C++首先**不允许引用改指向其他对象**，如果拷贝指向的对象，那么是拷贝对象的副本还是共用对象呢？所以C++拒绝
2. 成员含有const。因为更改const违法，C++不知道如何操作拷贝
3. base class的拷贝操作声明为private。这样derived class生成的拷贝操作无法访问基类的private部分也无法完成基类的拷贝

## 条款06：若不想使用编译器自动生成的函数，就该明确拒绝

当我们不想定义的类使用copy操作时，必须显式地阻止，因为编译器会为我们生成，且**所有编译器产出的函数都是public**。解决方法：

1. 将copy操作声明为private并刻意不实现他们。在member函数和friend函数试图调用copy操作时会出现linker error，C++ iostream就是这么实现的。

   这个方法可以更进一步把连接器时期的错误移动至编译器（更早检测出问题），即使用base class：

   ```c++
   class UnCopyable {
       protected:
       	UnCopyable() {}
       	UnCopyable() {}
       private:
       	UnCopyable(const UnCopyable&);
       	UnCopyable& operator=(const UnCopyable&);
   }
   // 后续的不希望含有copy操作的都继承这个基类
   class HomeForSale: private UnCopyable {...}
   ```

   UnCopyable的实现有些微妙，包括不一定要以public继承它，以及UnCopyable的析构函数不需要是virtual等

   boost实现了noncopyable版本以供使用

## 条款07：为多态基类声明virtual析构函数

#### why virtual 析构函数？

这个条款在使用factory模式时很有用，好比说P40的例子，工厂函数**返回一个base class的指针，指向新生成的derived class对象**。同时为了遵守工厂函数的规则，返回的对象必须**位于heap**，这也就意味着我们需要手动释放。

如果这时，我们的base class有一个non-virtual的析构函数，就会引发**灾难**--如果通过base class的指针试图删除derived class，就很可能保留derived class独有的部分未删除进而导致内存泄漏

解决方法很简单，就是给base class一个virtual的析构函数

virtual函数的目的是允许derived class的实现可以customized，一般class不含virual函数往往意味着它并不意图被用作一个base class

#### 不能滥用虚构函数

欲实现出virtual函数，对象必须携带某些信息，用以决定运行时调用哪一个virtual版本。这个信息通常由**vptr（virtual table pointer）**指出，vptr指向一个由**函数指针**构成的数组，称为**vtbl（virtual table）**，这个指针自然也会占用内存大小，比如下面的这个类：

```c++
class Point {
    public:
    	Point(int xCoord, int yCoord);
    	~Point();
    private:
    	int x, y;
}
```

在这个类中，因为虚构函数需要vptr指针，这个类的占用从64 bits -> 96 bits(32 位系统) / 128 bits(64位系统)，增加了50%-100%的占用。

所以虚构函数不能滥用

#### STL容器都是不带virtual析构函数的，不要企图继承

polymorphic base class（多态性质基类，如P40的例子，需要一个基类指向派生类并且能使用派生类的性质）应该声明一个virtual析构函数，如果class带有任何virtual函数，也应该拥有一个virtual析构

如果class声明的目的不是为了作为基类或作为多态，就不该有virtual析构函数

## 条款08：别让异常逃离析构函数

C++并不禁止析构函数throw异常，但它并不鼓励，比如下面这个例子

```c++
class Widget {
    public:
    	...
        ~Widget() {...}; // 析构函数允许抛出异常
    void doSomething() {
        std::vector<Widget> v;
        ...              // v在省略部分被自动销毁
    }
}
```

假设v中有多个元素，第一个元素析构时抛出异常，这时其他的元素还是正常被销毁，但这时又出现了第二个抛出异常的元素，对于C++而言，**同时出现两个异常程序不结束就会出现未定义行为**。本例中的vector换成std或TR1的任意容器都会导致未定义行为

P45例子中，假设一个DBConnection有建立连接的静态函数，我们建立一个DBConn来进行管理，那么可以在析构函数中使用try...catch...方法，try to `db.close()`，catch到异常后记录并使用`std::abort()`来避免未定义行为（也可以不abort直接将异常吞掉，当然这不是好主意）

以上的方法不论吞不吞掉异常都不够好，因为无法对”导致close抛出异常“的情况作出反应

下面的策略会更好一些

即重新设计DBConn接口，**使得client有机会对可能出现的问题作出反应**。比如DBConn可以自己提供一个close函数，因而赋予客户一个机会得以处理”因该操作发生的异常“，如下：

```C++
class DBConn {
    public:
    	...
        void close() { // 定义供给用户使用
            db.close();
            closed = true;
        }
    
    	~DBConn() {
            if (!closed) {
                try {
                    db.close();
                } catch(...) {
                    ... // 记录失败，然后abort或者吞下异常
                }
            }
        }
    private:
    	DBConnection db;
    	bool closed;
}
```

这是给用户一个处理的机会，如果他们不需要这个机会大可以忽略我们提供的close()函数而使用DBConn的析构函数去调用

总结：

1. 析构函数绝对不要throw异常，应该捕捉所有异常并处理
2. 如果用户需要对某个操作的运行异常做出反应，那么class中应该提供一个普通函数以供操作

## 条款09：决不在构造和析构过程中调用virtual函数

（P49）假设我们有一个基类，它的构造函数里调用了自己的virtual函数，那么即使派生类自己重写了虚函数，在调用派生类的构造函数时发生以下过程：

1. 在初始化派生类时，首先会调用基类的构造函数，从基类开始构造，而此时不仅派生类自己还完全没有开始初始化，在此时运行期类型信息也会把**此时的派生类认为是基类（不完全状态）**，且在结束基类部分的初始化开始派生类独有部分初始化前，该对象不会成为一个派生类对象

   这时构造函数就会使用基类自己的virtual函数，进而引发错误

2. 析构函数也是同样的道理。调用基类的析构函数时，这时派生类独有的部分都已经被销毁，只剩下了基类部分，此时也被认为是基类而非派生类对象

一般这种错误比较明显，一些编译器会发warning

但还有一些如下情况：

```c++
class Transaction {
    public:
    	Transaction() { init(); }        // 明面上调用non-virtual
    	virtual void logTransaction = 0;
    	...
    private:
    	void init() {
            ...
            logTransaction();            // 实际上在这里还是调用了virtual
        }
}
```

这种错误编译器不会发现，这里是一个=0的pure virtual函数还算幸运，因为被调用到程序会马上中止，如果不是的话就非常难debug

总结：

在构造和析构时调用的虚函数永远不会下降到派生类的版本，如果非要让基类使用派生类的版本，可以像P51那样，在派生类里定义一个private static再传给基类的构造函数

## 条款10：令operator=返回一个reference to *this

连锁赋值：

```c++
int x,y,z;
x = y = z = 1;
// 其本质是
x = (y = (z = 1));
```

为了实现连锁赋值，赋值运算符要返一个指向操作符左侧的实参的引用，如下

```c++
Widget& operator=(const Widget& rhs) {
    ...
    return *this;                      // 返回左侧对象的引用
}
```

包括赋值相关运算如+=，-=等都要遵循这个条款

虽然不强制，但是最好遵守

## 条款11：在operator=中处理”自我赋值“

虽然一般不会直接用到，但是自我赋值可能在不经意间发生，比如：

```c++
a[i] = a[j]; // i == j时，潜在的自我赋值
*px = *py;   // px,py指向同一个对象时，潜在的自我赋值
```

我们需要**自赋值安全(self-assignment-safe)**来完成条款13，14中资源管理的要求

两种解决方法对比：

1. 使用**证同测试（identity test）**

   ```c++
   Widget& Widget(const Widget &rhs) {
       if (this = &rhs) return *this;
       
       delete pb;
       pb = new Bitmap(*rhs.pb);
       return *this
   }
   ```

2. 用临时值保存再执行赋值

   ```c++
   Widget& Widget(const Widget &rhs) {
       Bitmap *pOrig = pb;
       pb = new Bitmap(*rhs.pb);
       delete pb;
       return *this
   }
   ```

这两种方法都可以行得通，但是第一种方法如果频率过高，因为identity test需要成本，它会使代码更”大“一些（包括原始码和目标码）并导入一个新的**控制流（control flow）**，两因素都会降低执行速度，prefetching，caching和pipelining等指令的的效率都会降低

第三个替代方案（swap）：

```c++
class Widget {
    ...
    void swap(Widget& rhs) {
    ...
    Widget& Widget::operator=(const Widget &rhs) {
        Widget temp(rhs);   // copy动作委托给构造函数
        swap(temp);
        return *this
    };
    }
}
```

代码清晰性略低，但是将copy的动作委托给构造函数是一个很聪明的做法，有时编译器可据此生成更高效的代码

## 条款12：复制对象时勿忘记其每一个成分

简单的来说就是当用户自定义复制操作时，编译器不会为用户检查是否复制了所有的元素

P58尤其注意当定义派生类拷贝操作时，不显式把基类的copy操作完成，编译器会**按默认初始化一个新的基类部分**

任何时候为derived class写copy操作时，都要小心翼翼地copy其基类部分，而且往往基类的copy都是private，只能通过copying函数调用，如下：

```c++
PriorityCustomer::operator=(const PriorityCustomer& rhs) {
    logCall(...);             // 记录copy操作
    Customer::operator=(rhs); // 对base class的部分拷贝,显式调用base的copy
    priority = rhs.priority;
    return *this;
}
```

还有就是，**不要试图**让一个copy操作（比如copy构造）共用另一个拷贝操作（如copy赋值），当重复代码很多时，应该定义**第三个函数**让两个操作使用以减少重复代码。

## 条款13：以对象管理资源

P62给出了一个C++ primer里也有类似的例子，好比有一个函数f里囊括了对象的create和delete，这样可行吗？答案是不行的

一方面，如果有return分支在delete前，很可能尚未释放内存，程序就已经结束了。而且就算程序员注意到了这点，之后负责维护代码的程序员也很可能不注意导致内存泄露

解决方法：

把资源放入对象，然后利用C++**析构函数自动调用机制**来保证资源被释放（智能指针），如：

```c++
std::auto_ptr<Investment> PInv(createInvestment());
```

两个关键想法：

1. 获得资源后立刻放进管理对象（Resource Acquisition is Initialization，RAII）
2. 管理对象运用析构函数保证资源被释放（即条款8中不让异常逃出析构函数）

接下来就是C++ primer中一样的shared_ptr和unique_ptr，这里不再赘述

但注意这里提出了C++ primer中未提到的环状引用（即已经无用的两个元素互相引用），python中对这个问题的处理方法是直接所有引用数量都减一，之后如果环状引用中有的元素减一仍然有引用，就证明它还在被使用

有个问题是auto_ptr和tr1::shared_ptr两者调用析构函数都是使用delete而非delete[]，这意味着在动态分配的array上使用这两者并不理想

如果非要，Boost有例如boost::scoped_array和boost::shared_array这样的针对C++动态分配数组的类

## 条款14：在资源管理类中小心copying行为

条款13中的观念适用于heap-based资源，然而有时候有些资源并不在heap上，这是需要我们自定义的资源管理类

P66假设我们使用C API处理类型为Mutex的互斥对象，含有以下两函数

```c++
void lock(Mutex* pm);
void unlock(Mutex* pm);

// 管理类如下：
class Lock {
    public:
    	explicit Lock(Mutex* pm): mutexPtr(pm) {
            lock(mutexPtr);
        }
    	~Lock() { unlock(mutexPtr); }
}
```

而用户的使用也符合RAII：

```c++
Mutex m;
...
{
    Lock(&m);    // 初始化时就锁定了Mutex
    ...          // 退出作用域Lock销毁时自动释放
}

// 然而当Lock被复制时？
Lock ml1(&m);
Lock ml2(ml1);
```

面对复制有几种选择：

1. 禁止复制（之前提到的Uncopyable）

2. 对底层资源使用**引用计数法**，只不过这次释放资源的行为是unlock而非默认的delete，即：

   ```c++
   class Lock {
       public:
       	explicit Lock(Mutex* pm): mutexPtr(pm, unlock) {
               lock(mutexPtr.get());  // get()会返回非shared指针，并不好
           }
       private:
       	std::tr1::shared_ptr<Mutex> mutexPtr;
   }
   ```

3. 复制底层资源，即deep copy

4. 转移底层资源的所有权（unique_ptr类似的实现）

## 条款15：在管理资源类中提供对原始资源的访问

在条款14中也提到了，我们可以通过`pInt.get()`来显式地把智能指针转换为普通指针
当然，auto_ptr和tr1::shared_ptr都支持通过指针取值操作符（operator->和operator*）**隐式**转化成底部的普通指针

P70例子，给了一个如果从Font往FontHanle转换需要显式get转换，虽然是用户把FontHandle对象交给Font类，但是每当用户需要使用FontHanlde时都需要get，这显得很繁琐

而P71提供了一种隐式转换会更轻松自如：

```c++
class Font {
    public:
    	...
        operator FontHandle() const { return f; } // 需要时直接隐式转换
    	...
};
```

但这样会增加错误发生机会，如下：

```c++
Font f1(getFont());
FontHandle f2 = f1;  
// 原意是拷贝Font对象，现在反而变成了先将f1转化为FontHandle再拷贝给f2
```

这非常不好，很有可能f1将其FontHandle销毁，但f2不知道，从而产生一个空悬指针

总结：

1. 资源管理类（RAII class）应提供一个**取得所管理资源的方法**
2. 显式和隐式的转换都可以，显式会更安全但繁琐，隐式方便但有隐患

## 条款16：成对使用new'和delete时要采取相同形式

这个条款总结就一句话

如果调用new时使用[]，你必须在相应的delete时也使用[]；如果调用new时未使用[]，在相应的delete调用时也不该使用[]

注意`typedef`可能将[]隐藏，但还是需要调用delete[]。所以也最好不要对数组对象调用`typedef`

## 条款17：以独立语句将new ed对象置入智能指针

考虑在以下语句中：

```c++
processWidget(std::tr1::shared_ptr<Widget>(new Widget), priority());
```

这里的参数含有三个步骤：

- 调用new Widget
- 调用priority()
- 调用std::tr1::shared_ptr构造函数

C++和Java/C#不同，上述三个步骤的差距是弹性的，而另外两个语言有特定次序

这也意味着如果执行顺序就像上述顺序一样，那么有可能在第二个步骤调用priority()时，priority抛出异常导致中断，这时，new Widget已经被执行，但创造的对象却还未放入智能指针导致**内存泄露**

所以要用**独立语句**将对象放入智能指针：

```c++
std::tr1::shared_ptr<Widget> pw(new Widget);
processWidget(pw, priority);
```

## 条款18：让接口容易被正确使用，不易被误用

理想上，如果用户企图使用某个接口而不能获得他想要的结果，编译器不该通过；如果编译器通过了编译，那么这个作为就应该是用户想要的

P79直接导入新类型以防用户错用

P80**以静态函数替换对象来表现某个特定月份**，因为non-local static的初始化顺序不确定

比如所有C++ STL容器都统一提供了一个size()方法来获知容器内的对象数量

再比如在条款13中使用智能指针管理对象，有时候用户会忘记智能指针，我们可以直接在fatory函数中就直接返回一个智能指针（优秀的设计）：

```c++
std::tr1::shared_ptr<Investment> createInvestment();
```

又比如智能指针中不是期待用户使用我们定义的析构函数，而是在使用智能指针时就定义好deleter也是一种良好的设计

将原始指针传给智能指针构造函数，比先初始化智能指针指向null再改变赋值更好（见条款26）

## 条款19：设计class犹如设计type

1. 新的对象如何被创建和销毁？

   构造和析构函数，以及内存管理部分：

   operator new; operator new[]; operator delete; operator delete[]

2. 对象的初始化和赋值的区别？

   初值列表，以及Object::Object( params ) 和 operator=()的区别

3. 新的type对象pass-by-value意味着什么？

   传值本质传副本，依赖于copy构造函数

4. 什么是新type的”合法值“？

   只有某些数值集是有效的

5. 新type需要套配合某个”继承图系“吗？

   需要配合会受到那些class的舒服，特别是他们的函数是”virtual还是non-virtual“（条款34和36），如果允许其他class继承自定义的class，那会影响自定义的析构函数virtua与否（条款7）

6. 新type需要什么样的转换？

   希望T1->T2，就必须在class T1内写一个operator T2或在class T2内息一个non-explicit-one-argument函数（可被单一实参调用）的函数，隐式显式选一个（条款15）

7. 什么操作符和函数对此新type合理？

   见条款23，24，26

8. 什么标准函数需要驳回？

   这些声明成private

9. 谁来取用新type的成员？

   区分public，protected，private

10. 什么是新type的undelared interface？

    对效率，异常安全性以及资源运用（多任务锁定和动态内存）提供何种保证？保证将带来约束

11. 新type有多一般化？

    如果并非一个type而是一个type家族，就应该用template

12. 真的需要一个新的type吗？

    说不定单纯定义一个或多个non-member函数或template或更合适

## 条款20：宁以pass-by-reference-to-const替换pass-by-value

默认C++是以传value的方法，实际都是传副本，默认的函数返回值也同样是传副本，这样往往会多次拷贝，十分昂贵

注意pass-by-reference-to-const中的const是最重要的，保证引用的源对象不会被改变。传引用也可以解决**slicing问题**，当一个derived class以传值形式且形参为base class时会被slice但传引用不会

和我在CSDN上查到的一篇文章一样，C++ primer的笔记我也提到过，引用的底层是指针，**传引用本质=传指针**

小型对象copy的消耗也可能很大，比如有些编译器允许把一个原生double放进缓存器内，但拒绝把仅有一个double的对象放入

还有一个理由是自定义对象的大小可能发生变化，之前copy消耗不大，在维护后可能type大小改变，传value就不理想了

## 条款21：必须返回对象时，别妄想返回其reference

这个条款书中主要分析了几种试图返回reference的方法：

1. 创建新对象两种：

   ```c++
   const Rational& operator*(const Rational& rhs, const Rational& lhs) {
       Rational res(lhs.n * lhs.n, rhs.n * rhs.n);
       return res;
   }
   ```

   让res作为local value在栈区被创建

   第一种问题出在如果这样返回引用，作为local value的res已经被销毁了，会变成空悬指针

   ```c++
   const Rational& operator*(const Rational& rhs, const Rational& lhs) {
       Rational* res = new Rational(lhs.n * lhs.n, rhs.n * rhs.n);
       return res;
   }
   ```

   让res作为new的对象在堆区被创建

   第二种问题出在难以delete，因为这是个被返回的指针，调用它的函数不一定能delete它，会造成内存泄露，尤其是在以下情况：

   ```c++
   Rational w,x,y,z;
   w = x*y*z; // 本质operator*(operator*(x,y),z)
   ```

   本质有两次new，需要两次对应的delete，中间那次new出来的指针已经被隐藏了，必定内存泄露

2. 使用local static对象

   ```c++
   const Rational& operator*(const Rational& rhs, const Rational& lhs) {
       static Rational* res;
       res = ...;
       return res;
   }
   ```

   当出现以下情况时：

   ```c++
   Rational a,b,c,d;
   ...
   if ((a*b) == (c*d)) {...}
   ```

   这时`(a*b) == (c*d)`永远是true，因为本质是local static == local static

正确方法，就返回一个对象：

```c++
inline const Rational operator*(const Rational& rhs, const Rational& lhs) {
    return Rational res(lhs.n * lhs.n, rhs.n * rhs.n);
}
```

结论是，当在必须”返回一个reference和返回一个object“中抉择，选择行为正确的就好，效率就交给编译器厂商了:)

## 条款22：将成员变量声明为private

