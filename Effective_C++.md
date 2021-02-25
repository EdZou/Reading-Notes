该笔记计划以55要点为分界，包括开头的术语（terminology）在内共56部分

## 第0部分 术语

1. 函数的签名也就是函数的类型，即返回类型+参数 ，如`std:size_t (int, int)`（官方对签名的定义不包括返回类型）
2. 这本书的不明确行为=undefined（C++ Primer中的未定义行为，这本书翻译腔太重了，怕是直接扔到google translate过了一圈）
3. Widght（小部件）一般不代表任何东西，仅作示例
4. 构造函数ctor，析构函数dtor
5. pt表示pointer to T类型，rt同理表示reference to T类型
6. TR1(Technical Report 1)是一份规范，描述了加入C++ STL的诸多新机能，所有组件都被放在命名空间tr1内，并嵌套在std命名空间中
7.  Boost是一个组织/网站，提供可移植的，有审核的，开源的C++库。大多数TR1机能以Boost的工作为基础
8. auto_ptr由C++98提出，C++11废弃，unique_ptr功能相似，但更严格一些，面对一些问题会在编译期就报错，详见[知乎文章](https://zhuanlan.zhihu.com/p/63890916)

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

本章主要是把public和protected的可能性否决，这样就只能用private修饰成员变量了

如果用public，那么更好的做法是使用**public成员函数**，并通过成员函数访问（并修改）成员变量，这样用户在使用我们的类时，也不用在意是否需要()来调用，更重要的是这样便于**封装**

比如我们需要同一个接口实现一个需要在space和efficiency上trade-off，那么public函数可以为所有的可能提供”弹性的实现“

封装的重要性：如果对client隐藏成员变量，可以**确保class约束条件总是获得维护**，因为只有成员函数可以影响他们，而且也**保留了日后变更实现的权力**

**成员变量的封装性和“当其内容改变时可能造成的代码破坏量”成反比**（条款23）

基于上述结论，假设我们有public的成员变量，而最终取消了它，那么所有使用它的客户码都会被破坏，这是一个不能预测的巨量，所以public完全没有封装性，protected相似，取消成员变量会导致所有derived class都被破坏，同样没有封装性

也就意味着，**protected并不public更有封装性**

## 条款23：宁以non-member、non-friend替换member函数

P98提供了一个clear函数的member版本和non-member版本，如下：

```c++
// member
class WebBrowser {
    public:
    	...
        void clearEverything();
    	...
};
// non-member
void clearBrowser(WebBrowser &wb) {
    wb.clearCache();
    wb.clearHistory();
    wb.removeCookies();
}
```

OOP要求数据以及操作数据的函数应该绑定在一起，但在这里并不正确

OOP同样要求数据尽可能地被封装，然而与直观相反，member函数的封装性更低。且non-member函数可以降低耦合度

封装性：

1. 封装的本质是如果某些东西被封装，那他就不再可见，越多的东西被封装，就有越少的人可以看到他们，而越少的人看到意味着我们也有更大的弹性去变化他
2. 现在考虑对象内的数据。如何量测有多少代码可以看到某一块数据呢？粗糙的量测：看到函数数量越多，封装性越低
3. non-member函数并不增加“能够访问class内的private成分”的函数数量（此例中，member的话有4个函数，而non-member只有3个（不包括自己，类外函数无法访问private））

以上论述有两点值得注意：

1. 这个论述只适用于non-member且non-friend函数（这两种和member有同样访问权力）
2. 虽然因为封装性让函数成为该class的non-member，但它可以是**另一个类的member函数**

这里non-member函数可以与class在同一个namespace中，因为namespace可以跨越多个源码文件而class不能，有如下组织方式

```c++
// 头文件 webbrowser.h
namespace WebBrowserStuff {
    class WebBrowser { ... }; // 核心机能，所有客户都需要的
    ...                       // non-member函数
}

// 头文件 webbrowserbookmarks.h
namespace WebBrowserStuff {
    ...                       // bookmarks相关功能
}

// 头文件 webbrowsercookies.h
namespace WebBrowserStuff {
    ...                       // cookies相关功能
}
```

**以上正是C++标准库的组织方式**，C++标准库由几十个头文件组成，这允许客户只对他们所用的那一小部分系统形成编译依赖（条款31），但class成员函数不能这样切割

所以如果某个客户想写下和影响下载有关的便利函数，只要在` WebBrowserStuff`建立一个头文件并声明那些便利函数即可

## 条款24：若所有参数皆需类型转换，请为此采用non-member函数

当**建立数值类型**时，令class支持隐式类型转换是可以接受的，如下：

```c++
class Rational {
    ...
    const Rational operator*(const Rational& rhs) const;
}

// 但尝试混合式运算，只有一半行得通
result = oneHalf * 2;  // 成功, oneHalf.operator*(2)
result = 2 * oneHalf   // 失败, 2.operator*(oneHalf)
```

错误发生在，当以上本质操作试图调用int的operator*时，并没有Rational的版本，于是试图去找non-member版本：`result = operator*(2, oneHalf)`，找不到就报错

而成功的版本，本质是如下过程

```c++
const Rational temp(2);
result = oneHalf * temp;
```

理想应该是定义一个non-member，如下：

```c++
const Rational operator*(const Rational& lhs, const Rational& rhs) {
    return Rational(......);
}
```

## 条款25：考虑写出一个不抛异常的swap函数

swap是异常安全性编程的脊柱，以及用来处理自我赋值的一个常见机制

默认的std版本swap只要支持copying就可以通过copy的本质方法完成swap操作

如果我们直接在std空间里进行修改，那么：

```c++
namespace std {
    template<>
    void swap<Widget>( Widget& a,
                       Widget& b) {
        swap(a.pImpl, b.pImpl);
    }
}
```

这个方法通不过编译，因为本质是一个std::swap的total template specialization版本，`swap<Widget>`，表示这是T为Widget的特化。

虽然std一般不允许改变，但可以为标准template制造特化版本，以上行为这点是合法的

代码通不过编译的原因是pImpl是private，无法直接访问

正确的方法和C++ primer上一样：

```c++
class Widget {
    public:
    	...
         void swap(Widget& other) {
            using std::swap;
            swap(pImpl, other.pImpl);
        }
}

// 修改后的特化版本
namespace std {
    template<>
    void swap<Widget>( Widget& a,
                       Widget& b) {
        a.swap(b);
    }
}
```

但如果假设我们的类是函数模板，如下：

```c++
template<typename T>
class WidgetImpl {...};
template<typename T>
class Widget {...};
```

再将上面的std::swap进行特化：

```c++
// partial特化版本
namespace std {
    template<typename T>
    void swap< Widget<T> >( Widget<T>& a,
                            Widget<T>& b) {
        a.swap(b);
    }
}
```

这个不合法，因为C++允许类template进行partial specialize但不支持function template do so

如果定义一个std::swap的重载版本，如下：

```c++
// 重载版本
namespace std {
    template<typename T>
    void swap( Widget<T>& a,
               Widget<T>& b) {
        a.swap(b);
    }
}
```

这是不正确的，因为std虽然允许特化template，但不允许添加新的template

理想的答案就是用一个non-member函数，但不再将non-member函数作为std::swap的重载或者特化版本，并将它放在WidgetStuff namespace中，以便使用

```c++
namespace WidgetStuff {
    ...
    template<typename T>
    class Widget {...};
    ...
    template<typename T>
    void swap( Widget<T>& a,
               Widget<T>& b) {
        a.swap(b);
    }
}
```

之后对Widget对象的置换都会找到这个函数

编译器看到swap的调用，会找到适当的swap，编译器默认喜欢更特化的Widget版本，所以使用`using std::swap`让std版本暴露在查找范围中，由于此时特化版本swap还未定义好，所以使用std版本

不使用`std::swap(obj1, obj2);`的原因：影响编译器的挑选，不可能再挑选一个更适合T当前版本的swap函数

总结如下：

1. 如果默认的swap函数效率可以接受那就不需要做额外的工作
2. 效率不足做以下尝试：
   1. 提供一个public的swap成员函数，该函数绝**不该抛出异常（仅限于member函数，条款29有详细说明）**
   2. 在class或template的命名空间提供一个non-member的swap函数，并令其调用swap成员函数
   3. 如果在类内定义特化的swap函数，注意用`using std::swap`让std的swap函数在编译器的视野内暴露可见

## 条款26：尽量延后变量定义式出现的时间

核心原因：只要定义了，在定义时和在离开作用域时需要析构，承担成本

假设定义后还未使用又抛出异常，等于变相增加了不必要的成本，所以最好把定义延后到确实需要它的时候

对于循环的变量定义：

1. 定义在循环外：1次构造+1次析构+n次赋值
2. 定义在循环内：n次构造+n次析构

根据class的赋值成本是否低于（一次构造+一次析构）来决定，一般定义在循环内

## 条款27：尽量少做转型操作

cast相关的操作，C++ primer里已有详细介绍，这里大概说一下

1. 旧式转型（C-style）：`(T)expression or T(expression)`，两种效果一致
2. `const_cast<T>`：修改常量属性（移除/添加）
3. `dynamic_cast<T>`：将类“安全向下转型”，唯一**无法用旧式语法执行来的动作**，也是唯一**可能耗费重大成本的动作**
4. `reinterpret_cast<T>`：实际**取决编译器**的低级转型，**不可移植**
5. `static_cast<T>`：强迫隐式转换，如：non-const -> const，int -> double，void*指针 -> typed，pointer-to-base -> pointer-to-derived。无法将const -> non-const（`const_cast`专属）

任何一种类型转换都会产生一些代码，如下情况：

```c++
class Base {...};
class Derived: public Base {...};
Derived d;
Base* pb = &d; // 会有个offset在运行期被加在Derived*指针上
```

这个例子意味着，一个对象可能拥有一个以上的地址（Base和Derived版本指针指向不同地址），C，Java，C#都不能，只有C++有，且多继承一定会发生

一个似是而非的代码：

```c++
class SpecialWindow: public Window { // derived class
    public:
    	virtual void onResize() {
            static_cast<Window>(*this).onResize();
            ...
        }
}
```

这里试图调用当前对象Window部分的onResize()函数，但本质是调用了**稍早时候转型动作建立的"this对象的base部分"的副本**上的onResize()函数，这意味着如果onResize()原本会对对象的数据进行处理，现在只能处理暂时副本，导致SpecialWindow独有的这部分onResize()函数会对数据起作用，但Window的这部分因为本质处理临时副本而没有作用，看起来就像“伤残”状态一样

正确的写法就是直接用`Window::onResize()`

`dynamic_cast`一般是在需要对derived对象处理但却又只有base指针指向derived对象，如果继承体系有4层，则需要可能在对象进行4次调用，消耗很大

有两个一般性做法可以避免：

1. 使用容器并存储直接指向derived的指针（C++ primer里不推荐这样，每个derived类都要一个特定容器不划算）
2. 在base类里提供一个什么也没做的virtual函数供给derived类去实现

这两种只是`dynamic_cast`的替代方案，不必过于抗拒`dynamic_cast`，但**必须避免连串(cascading) dynamic_casts**，如下：

```c++
typedef std::vector<std::tr1::shared_ptr<Window)> > VPW;
VPW winPtrs;
...
for (VPW::iterator iter = winPtrs.begin();
     iter != winPtrs.end(); ++iter) {
    if (SpecialWindow1 *psw1 = 
        dynamic_cast<SpecialWindow1*>(iter->get())) {...}
    else if (SpecialWindow2 *psw2 = 
        dynamic_cast<SpecialWindow2*>(iter->get())) {...}
    else if (SpecialWindow3 *psw3 = 
        dynamic_cast<SpecialWindow3*>(iter->get())) {...}
    ...
}
```

又大又慢，而且每次继承体系有变更都需要更改，基础不稳

总结：

1. 尽量避免转型，特别是需要效率时的`dynamic_cast`
2. 非要转型，就把动作隐藏在某个函数背后
3. 用C++转型而非旧式转型

## 条款28： 避免返回handles指向对象内部成分

比如一个const成员函数return传出了private成员变量的引用，那么这个成员变量本质上是public了，这正是条款3中bitwise constness的一个结果

指针，引用和迭代器都算handles，返回一个private成员（不论是变量还是private或者protected的函数）的handle，随之而来的是**降低对象封装性的风险**

绝不该令一个函数返回一个**访问级别较低**的handles

而且就算做如下处理：

```c++
const Point& upperLeft() const {return pData -> ulhc;}
```

也可能导致对象已经被销毁，但handle还在造成空悬指针

## 条款29：为“异常安全”而努力是值得的

当异常抛出时，异常安全性的函数会：

1. 不泄露任何资源（P127，new操作异常会导致mutex不释放）
2. 不允许数据破坏（P127，new操作异常会破坏掉赋值的指针，以及其他的数据imageChanges已被改变）

解决mutex被释放问题：

如条款13，14中提到，设计一个资源管理类，如条款14中的Lock类`Lock ml(&mutex);`

在解决数据破坏前，先陈述异常安全函数的保证：

1. 基本承诺：如果异常被抛出，程序内的任何函数仍然保持在有效状态之下。没有任何对象或数据结构回因此被破坏，所有类内对象都保持着前后一致的状态
2. 强烈保证：异常被抛出，程序状态不改变，（atomicity，要么完全成功，要么回滚到调用前）
3. 不抛掷（nothrow）保证：承诺绝不抛出异常，总能完成预先被承诺的功能

异常安全码必须提供上述三种保证之一。

nothrow对于C part of C++的部分不能保证，C++使用容器的抛出bad_alloc异常，可以提供nothrow保证

而对于绝大部分函数，都要从前两种选一种

于是对P127的函数做出以下修改：

1. 使用shared_ptr指向Image，帮助自动销毁
2. 重排imageChanges发生的顺序
3. 把imgSrc做一个copy and swap，对副本进行修改，修改成功再置换

强烈保证过于难以实现，即使有两个函数f1，f2都是强烈保证函数，那么假设f1执行成功，f2失败，也难以退回到f1执行前的状态。而且copy-and-swap往往十分昂贵，没有现实意义

函数提供的“异常安全性保证”遵循木桶效应，取决于调用的所有函数的“异常安全性”最弱的那个

## 条款30：透彻了解inlining的里里外外

inline函数：

1. 优势：看起来像函数，用起来也像，又不承受函数调用带来的额外开销，这是因为编译器的最优化机制通常被用来设计那些“不含函数调用”的代码
2. 缺点：inline函数的观念是，“对此函数的每一次调用”都以函数的本体直接**替代**，这会**增加目标码（obj code）的大小**，进而导致额外的paging行为，降低cache命中率，以及伴随来的效率降低

但缺点上换一个角度看，如果函数本体比“函数调用”所产出的目标码还要小，inline可以减小obj code大小并提高cache命中率！

注意inline只是对编译器的一个**申请**，**不是强制命令**

inline函数通常**一定被置于头文件中**，因为C++大多数编译器都是在编译期间进行inlining，为此他们需要知道函数长什么样子

template通常也置于头文件中，一旦被使用，编译器为了将它具现化，需要知道它长什么样子，而这个过程与inlining无关。如果这个template具现的所有函数都是inlined，那么这个template应该被声明成inline版本；如果没有理由要求每一个template具现的函数都是inline，就避免

所有对**virtual函数**的调用都会使inlining**落空**，因为直至执行前，编译器都不知道函数本体，无法替换

一个看上去是inline的函数实际是不是inline取决于编译器，所幸绝大部分编译器在如果无法将申请的inline实现时，会报warning

编译器通常**不对通过函数指针进行的调用**实施inlining

不要对构造和析构函数inline，编译器又是会生成构造和析构函数的outline副本，从而获得指针指向这些函数，而且如P138所示，构造和析构的内部其实有很多对于异常的精细处理被隐藏了

一旦设计者决定改变f，那么所有用到f的客户端程序都必须重新编译，如果f是non-inline，则只要客户选重新link就好了，如果是dll，那么改变的函数甚至可以不知不觉被应用程序吸纳

大多debugger面对inline函数**束手无策**，仅仅能在debug版本的程序中禁止发生inlining

## 条款31：将文件间的编译依存关系降至最低

当改变某个文件，导致全世界都重新编译和链接时（Xilinx实习时的PYNQ项目:P），意味着文件中的依赖比较严重。

文件之间的依赖可以举个例子，几乎所有的源码都会include其他的头文件，这时被include的头文件发生更改就会使include它的源文件全部重新编译

类要求如P140中，private成员的定义式而非仅仅是声明，因为编译器**必须在编译期**知道对象的大小

这个问题在Java，Smalltalk上并不存在，这些语言都是在编译时，分配一个指针的空间指向具体的实现。这类设计常被称为**pimpl idiom(pointer to implementation)**，这样使P142中的Person类彻底和其他细目类分离了，完成接口与实现的分离。

该分离的关键在于以**声明的依存性**替换**定义的依存性**，这也是编译依存最小化的本质：尽量让头文件自我满足，做不到也让他和声明式而非定义式相依：

1. 如果使用obj的指针或引用可以完成任务，就不要用obj本身
2. 如果可以，尽量以class声明式代替定义式（让提供class定义式的任务从“声明所在地”转移到“内含函数调用”的客户代码）
3. 为声明式和定义式提供不同的头文件。两种方法，一种将声明式作为handle class，如P145，在初始化时指定指向的定义式对象；一种是使用interface class（没有成员变量，无构造函数，当然C++不禁止成员变量和成员函数，与node，java等真正的interface不同），接口都定义为pure virtual函数

Interface class的用户为这种类创建新对象方法：调用一个特殊函数，扮演**真正被具现化的derived class的构造函数**，这种函数被称为**factory函数或virtual构造函数**。factory函数返回指针，指向动态分配得到的对象，一般该函数在interface class内被声明为static，如下

```c++
class Person {
    public:
    	...
        static std::tr1::shared_ptr<Person> 
            create(const std::string& name,
                   const Date& birthday,
                   const Address& addr); // factory函数
    	...
};
...
// 客户使用
std::tr1::shared_ptr<Person> pp(Person::create(name, birthday, addr));
```

Handle class和interface class解除了接口和实现间的耦合关系

坏处：

1. 在handle class上，每一个对象消耗的内存需要增加一个impl pointer大小，且pImpl必须初始化并指向动态分配来的impl obj，因而承受动态内存分配和销毁的和开销，以及遭遇bad_alloc的可能性
2. 对于interface class而言，每个函数都是virtual，所以每次调用都有**间接跳跃（indirect jump）**成本。且interface class的派生类一定有一个vptr，增加内存数量

## 条款32：确定你的public继承塑模出is-a关系

公开继承意味着is-a的关系，is-a关系就是好比base和derived，derived一定is a base class，但反过来并不成立，C++中的public继承严格遵守“is-a”关系

比如企鹅和鸟的例子，有几种做法：

1. 企鹅继承鸟fly的方法，如果被调用就返回异errorMsg
2. 不为企鹅定义fly方法，只要被使用编译器就会发现

但现实的理解和public继承有偏差：

P154举出的长方形（base）和正方形（derived）的关系，长方形可以独立修改长/宽，但正方形的长宽始终一致，导致了derived没法实现base的特征

## 条款33：避免遮掩继承而来的名称

这里的遮掩，在C++ primer中叫隐藏

derived class中如果定义了和base class一样的名字，那么根据编译器查找规则（C++ primer中也提到了，derived->base->namespace->global，什么时候找到什么时候停止），编译器会先找到derived内部的名字，并停止查找，进而导致base内的同名函数/变量被隐藏

如果想要达到overload的效果，那么最好使用using声明（C++ primer中的部分）

如果是derived只想继承base的一部分函数，那么不应该用public继承而应该用private继承，并使用简单的转交函数，如下

```c++
class Derived: private Base {
    public:
    	virtual void mf1() { Base::mf1(); } // 转交函数，隐式inline
}
```

转交函数是不支持using的老式编译器的替代

## 条款34：区分接口继承和实现继承

1. 成员函数的接口总是会被继承

2. 声明一个pure virtual函数是为了让derived classes只继承函数接口。但是其实base class可以为pure virtual函数提供一份定义，只是调用方法**必须指明class名称**

3. impure virtual函数的目的，就是为了让derived classes同时继承函数的接口和默认实现。在P164有一个例子，好比说modelA和modelB都是继承base类Airplane而来，他们都使用了默认的函数去划定航线。这时我们需要一个modelC派生类，但是忘记定义ModelC自己的函数版本，这可能导致极大的航空安全问题，于是有了第四种

4. 上述问题的根本原因是：ModelC没有明确说”我要“就继承了默认的函数。于是有了以下写法：

   ```c++
   class Airplane {
       public void fly(const Airport& destination) = 0;
       protected:
       	void defaultFly(const Airport& destination);
       ...
   }
   
   // 默认继承演变成以下形式：
   class ModelA: public Airplane {
       public:
       	virtual void fly(const Airport& destination) {
               defaultFly(destination);
           }
       	...
   }
   ```

   将原本的impure virtual函数转化为pure virtual并提供相应的default函数以供derived主动继承

   而这里defaultFly是non-virtual这点也很重要，没有derived class应该重新定义该函数（条款36）

   有些人会觉得分别提供default和virtual接口会造成接口污染，而这里也可以用到上面说到的pure virtual函数也可以有自己的定义这一点解决，得出代码如下：

   ```c++
   class Airplane {
       public void fly(const Airport& destination) = 0;
       ...
   };
   // 提供pure virtual的定义
   void Airplane::fly(const Airport& destination) {...};
   
   // 默认继承接着演变成以下形式：
   class ModelA: public Airplane {
       public:
       	virtual void fly(const Airport& destination) {
               Airplane::fly(destination);
           }
       	...
   }
   ```

   当然这种做法会使习惯上protected的函数成为public（小缺点）

5. 声明non-virtual是为了令derived class有继承函数的接口及一份强制性的实现

## 条款35：考虑virtual函数以外的其他选择

#### Non-Virtual Interface（NVI）手法实现Template Method模式

P170例子简单地说，就是提供一个public的接口函数，调用private的virtual实现函数（与C++模板Template并无关联），把这个public函数称为是这个private virtual函数的外覆器（wrapper）

这样可以在wrapper中完成调用virtual函数需要的**事前和事后工作**：事前如获得mutex，log，验证等，事后包括释放mutex，验证等等。如果直接调用virtual函数，这些工作都难以完成

virtual函数在NVI手法下没必要一定是private，比如需要调用base class的兄弟，这样virtual函数需要时protected

#### 藉由Function Pointers实现Strategy模式

如P172的例子，设计一个计算游戏人物血量的函数，这样的计算**完全不需要**人物本身的成分信息，对以这种情况，我们可以让构造函数接受一个指向健康计算函数的**指针**。

将strategy设计模式与virtual相比较，有以下有趣的特性：

- 同一个类但不同的实体对象可以使用不同的健康计算函数（P172 EvilBadGuy）
- 已知的任务健康计算函数可以在运行期变更（P172可以设定成员函数setHealthCalculator来替换健康计算函数）

说白了就是**本例中的健康计算函数已经不属于继承体系了**，这也同样意味着**本例中所有的健康计算函数**都没有访问non-public部分

如果需要访问，那就要减少class的封装性，比如声明non-member函数为friend或为其部分成员变量提供public的访问函数，这需要权衡利弊

#### 藉由tr1::function完成Strategy模式

上例Strategy模式里限制了只能使用non-member函数，为什么不能使用如成员函数，lambda呢。这里可以使用tr1::function模板来解决这个问题

如下：

```c++
typedef std::tr1::function<const GameCharacter&> HealthCalcFunc;
```

函数对象和function兼容即可，即参数可被**隐式**转换为const GameCharacter&且返回类型可被隐式地转换为int

std::bind改变接受参数的数量（C++ primer中有），也可以增加弹性

## 条款36：绝不重新定义继承而来的non-virtual函数

如下代码：

```c++
class B {
    public:
    	void mf();
    ...
};
class D: public B { ... };
D x;                      // x是一个D对象
// 如果有下述行为
B* pB = &x;
pB->mf();

D* pD = &x;
pD->mf();
```

如果mf()是一个non-virtual函数且D定义了自己的mf()函数，那么通过pB和pD调用的函数不是一样的

这是因为B::mf和D::mf因为是non-virtual函数，所以都是**静态绑定**的，所以通过pB调用的non-virtual函数永远是B定义的函数

另一方面，virtual函数是动态绑定，同样的上述调用方法，如果mf()是virtual函数，那么不论通过pB还是pD，最终得到的都会是D版本的mf()函数，因为他们指向的本质都是类型为D的对象

这也是这条款的原因，如果重写继承来的non-virtual函数，那么函数的版本不是取决于指向的对象，而是取决于指向对象的指针/引用类型

此外还会违反"is-a"原则

## 条款37：绝不重新定义继承而来的缺省值

条款36已经说明了定义一个non-virtual函数就不该被重新定义，所以该条款的讨论仅限于”继承一个带有default参数值的virtual函数“

所以基于上述论断，该条款的理由是：**virtual函数是动态绑定**（dynamically bound，后期绑定，late binding），**但default参数值却是静态绑定**（statically bound，前期绑定，early binding）

如下代码：

```c++
class Shape {
    public:
    	enum ShapeColor { Red, Green, Blue};
    	virtual void draw(ShapeColor color = Red) const = 0;
    ...
};

class Rectangle: public Shape {
    public:
    	// 试图赋予不同的default值，很不好
    	virtual void draw(ShapeColor color = Green) const;
    ...
};

class Circle: public Shape {
    public:
    	virtual void draw(ShapeColor color) const;
    	// 注意这里，如果是使用对象调用时，会发生静态调用
    	// 而静态调用不会从base继承default参数值，所以一定要指定参数
    	// 但如果是指针或引用就发生动态调用，可以继承default参数值
}
```

至于Rectangle为什么很不好，因为一旦通过`Shape*`指针或引用调用Rectangle的draw函数，会出现神奇的现象：用着Rectangle的函数版本却使用了base类Shape提供的default参数

因为default参数是静态绑定到base上的

为什么呢？因为如果default参数是动态绑定，那么就需要一种方法能在运行期为virtual函数决定使用哪个参数，这样做太过复杂了

这里可以使用NVI手法：

```c++
class Shape {
    public:
    	enum ShapeColor { Red, Green, Blue};
    	// Shape有自己的non-virtual函数
    	void draw(ShapeColor color = Red) {doDraw(color)};
    ...
    private:
    	// NVI virtual部分
    	virtual void doDraw(ShapeColor color) const = 0; 
};

class Rectange: public Shape {
    public:
    	...
    private:
    	// 不需要指定参数值
    	virtual void doDraw(ShapeColor color) const; 
};
```

## 条款38：通过复合塑模出has-a或”根据某物实现出“

复合意味着has-a或者"is-implemented-in-terms-of"

当程序中的对象相当于正在抽象的世界中的事物，如人，汽车等，这些属于应用域（application domain）

而其他对象如buffer，mutex，search tree这种人工制品，就属于实现域（implementation domain）

当复合发生在应用域时，表现出has-a关系，当复合发生在实现域时，表现出"is-implemented-in-terms-of"

一个运用复合的例子：比如我们现在想要把原本std::set(使用平衡查找树实现的)用linked list实现，这时我们新定义的Set类可以让std::list的模板成为private成员变量再提供成员函数操作它，这就是复合

## 条款39：明智而审慎地使用private继承

private继承意味着"is-implemented-in-terms-of"。如果让class D以private形式继承class B，那么是为了B内已经存在的某些特性而不是让B和D有任何观念上的交集

private继承纯粹是一种实现技术，不体现出继承关系，在设计上没有意义，意义只在软件实现上

P188举了一个往自定义的class Widget里加Timer，而Timer的ontick()是一个virtual函数需要实现，这时，如果我们使用private继承，固然可以自定义并使用ontick()，但它不应该出现在我们的自定义接口中

如果这里使用复合会更好，如以下方案：

定义一个内嵌类WidgetTimer，它以public继承继承Timer但是却属于Widget的private成员。这个方法有两个好处：

1. 当Widget作为base class被继承时，如果ontick()是通过private继承来的话，它的derived class必须要定义因为ontick()是virtual函数，但通过内嵌类就不会有这个问题
2. 降低编译依存性。WidgetTimer可以被移出Widget类内（我一开始就这么想的...），然后Widget只用一个指针指向WidgetTimer的对象（声明式），这样Widget就可以不依赖Timer.h

当然如果涉及空间优化，那么可以private继承而非**继承+复合**

C++默认所有**独立（非附属）**对象都需要有非零大小，假设继承一个啥都没有的base类，那么这个继承的部分就是0空间，也就是**空白基类最优化EBO**，当然C++也只在单一继承时遵循EBO，一般无法被施加于继承多个base的derived class上。EBO的是为了不让继承本身增加derived的成本

## 条款40：明智而审慎地使用多重继承

首先比如说多重继承时，两个base class都有同名同参成员函数，这时为了解决这个歧义，我们必须指定调用哪一个base class地函数，如下：

```c++
mp.BorrowableItem::CheckOut();
```

包括继承时base每个都会有副本这个问题，虽然可以通过**虚继承**解决，但是有代价：比non-virtual继承的**体积大**

除此之外，因为virtual base的初始化由继承体系中最底层的most derived class负责，因此还有其他的成本：

1. class如果derive from virtual bases而需要初始化，**必须认知**其virtual base，不管离得有多远
2. 但一个新的derived class进入继承体系，它必须承担其virtual bases的初始化责任

所以建议如下：

1. 非必要不要使用virtual base class
2. 即使使用了，避免往里面放数据，减少之后的初始化

P197详细讲述了一个多重继承的case，也就是public继承某个interface class + private继承某个协助实现的class

## 条款41：了解隐式接口和编译期多态

比如有如下模板：

```c++
template<typename T>
void doProcessing(T& w) {
    if (w.size() > 10 && w != someNastyWidget) {
        ...
    }
}
```

1. w必须支持哪一种接口，系由template中执行于w身上的操作来决定。本例中的.size()以及!=等
2. 凡涉及到**任何函数调用**（本例的.size()以及!=等），有可能造成template实例化，是这些调用得以成功，这样的实例化行为发生在编译期。**由不同的template参数实例化导致调用不同的函数，这就是编译器多态**

隐式接口是什么呢？

就这个例子里面提到了operator!=，但是Widget自身并没有显式定义相应的接口，这些接口可以从base class中来，而且只要能完成转换就行。

比如base提供的size函数是一个short型，但只要他能加上一个int型（本例中为10）并调用operator>和0对比即可。再比如后半段的operator!=，这里也不是非得让等式两边取得一个相同类型，只要能隐式转换即可

这就是template隐式接口，奠基于有效表达式。多态则通过template实例化和函数重载解析在**编译期**实现

## 条款42：了解typename的双重意义

template中出现的名称如果依赖于某个template参数，称之为从属名称（dependent names）；如果从属在class内呈嵌套状，则称为嵌套从属名称（nested dependent name）

比如`C::const_iterator iter(container.begin());`，C::const_iterator就是一个嵌套从属类型名称

然而这可能造成parsing困难，如下：

```c++
template<typename C>
void print2nd(const C& container) {
    C::const_container* X;
}
```

（C++ primer中也提到了）这么一个表达式，可以理解为定义了一个指针X，也可以理解为C的一个const_container变量与X相乘。C++有规则解析了这一歧义状态：如果解析器在template中遭遇一个嵌套从属名称，它便假设这名称不是个类型，**除非显式声明为类型**，如下

```c++
typename C::const_iterator iter(container.begin());
```

在前面加一个typename即可，遵循以下几点：

1. **仅用于验明嵌套从属类型名称**

2. typename不可以出现在base class list（基类列表）之前，也不能在初值列中作为base class的修饰符

## 条款43：学习处理模板化基类内的名称

假设我们有以下template模板：

```c++
template<typename Company>
class MsgSender {
    public:
    	...
        void sendClear(const MsgInfo& info)  {...}
    	void sendSecret(const MsgInfo& info) {...}
}

// derived class继承模板
template<typename Company>
class LoggingMsgSender: public MsgSender<Company> {
    public:
    	...
        void sendClearMsg(const MsgInfo& info) {
            // 把传送前信息写进log
            sendClear();
            // 把传送后的信息写进log
        }
}
```

但这样不会通过编译，编译器会抱怨`sendClear()`函数不存在

为什么呢？假设我们有以下特化版，该CompanyZ版本只允许`sendSecret()`

```c++
template<>
class MsgSender<CompanyZ> {
    public:
    	...
    	void sendSecret(const MsgInfo& info) {...}
}
```

这时就没有`sendClear()`方法了，这也是为什么编译器会报错，因为编译器清楚base class template有可能被特化，故而**拒绝**在templatized base classes内**寻找继承而来的名称**

而我们有三种办法令”C++不进入templatized base classes这个行为失效“

1. 在base class的函数调用动作前加上"this->"

   ```c++
   template<typename Company>
   class LoggingMsgSender: public MsgSender<Company> {
       public:
       	...
           void sendClearMsg(const MsgInfo& info) {
               // 把传送前信息写进log
               this->sendClear();
               // 把传送后的信息写进log
           }
   }
   ```

2. 使用using声明：

   ```c++
   template<typename Company>
   class LoggingMsgSender: public MsgSender<Company> {
       public:
       	...
           using MsgSender<Company>::sendClear();
       	...
           void sendClearMsg(const MsgInfo& info) {
               // 把传送前信息写进log
               sendClear();
               // 把传送后的信息写进log
           }
   }
   ```

3. 用作用域运算符指出函数位置

   ```c++
   template<typename Company>
   class LoggingMsgSender: public MsgSender<Company> {
       public:
       	...
           void sendClearMsg(const MsgInfo& info) {
               // 把传送前信息写进log
               MsgSender<Company>::sendClear();
               // 把传送后的信息写进log
           }
   }
   ```

   这个方法最不好，如果函数是virtual的话，这个方法会关闭”virtual绑定行为“

上述三种做法本质都是：对编译器**承诺** ”base class template的任何特化版本” 都会支持该接口。这种支持一般是编译器在解析derived class template所必需的。但如果最终没有支持，使用时还是会不通过编译

## 条款44：将与参数无关的代码抽离templates

使用template有可能造成代码膨胀（code bloat），有可能源码整齐但obj code却不是那么回事。这时我们使用**共性与变性分析（commonality and variability analysis）**来避免

比如说我们为一个正方matrix写一个template，如下：

```c++
template<typename T, std::size_t n>
class SquareMatrix {
    public:
    	...
        void invert();
}
// 假设我们这样用这个template
SquareMatrix<double, 5> sm1;
sm1.invert();
SquareMatrix<double, 10> sm1;
sm2.invert();
```

上述使用，两个矩阵只有常量不同，其他的都完全一致，这就明显会导致template引出代码膨胀

第一次修改就是把invert函数单独摘出来，成为一个base template让SquareMatrix去继承他

```c++
template<typename T>
class SquareMatrixBase {
    protected:
    	...
        void invert(std::size_t matrixSize); // 把size转移到这里
}
// SquareMatrix继承base如下
template<typename T, std::size_t n>
class SquareMatrix: private SquareMatrixBase<T> {
    private:
    	using SquareMatrixBase<T>::invert;
    public:
    	...
        void invert() {this->invert(n);}       // 制造一个inline调用
}
```

注意`this->invert`，如果这里不用this，base的invert()就会被derived class隐藏

这里则出现了一个新的问题，base类怎么得到SquareMatrix内部的矩阵数据呢？书中提供了一个往base里加上指针指向矩阵数据的解决方法，也可以通过new把数据放在heap里

但这同样是有代价的：

从一方面看，改进后的绑定size的invert版本可能生成比前一版本更加的代码。例如size是一个编译期常量，因此可藉由常量的广传达到最优化，可以折进被生成指令中成为直接操作数。而前一版本做不到

另一方面看，不同大小的矩阵共享一个版本的invert(前一版本)，可以减少执行文件的大小，进而降低文件work set（在虚拟内存环境下执行的process而言，所在使用的pages），强化指令高速cache中的引用集中化（locality of reference）

以上是效率上需要权衡的

还有就是改进后版本需要一个指针指向数据，这也意味着每个SquareMatrix都多了一个指针的大小，但这往往也难以做到更好了

最后就是这个条款只讨论了**non-type template parameters**带来的膨胀，其实**type parameters**也会带来膨胀，比如有些平台上int和long其实二进制表述一样但`vector<int>, vector<long>`的成员函数也可能完全相同。还有就是所有的指针都是一样的二进制表述，不需要为了指向的对象进行区分导致膨胀

总结：

1. 因**non-type template parameters**带来的膨胀可以通过函数参数（invert参数）或class成员变量消除
2. 因**type parameters**带来的膨胀也可以降低。往往通过让完全相同二进制表述的实例化类型共享代码

## 条款45：运用成员模板类型接受所有的兼容类型

所谓“智能指针”是“行为像指针”的对象，STL容器的迭代器几乎总是智能指针

但为了支持真实指针所支持的隐式转换的能力，需要程序员自己支持。因为对于template而言，`template<base>`和`template<derived>`之间没有一毛钱的关系

但继承体系中的derived classes随时可能被添加，这样似乎需要无穷无尽的构造函数。这时就需要**member function templates**，如下

```c++
template<typename T>
class SmartPtr {
    public:
    	template<typename U>                   // member template
    	SmartPtr(const SmartPtr<U> &other);    // 为了生成copy构造函数
    	...
}
```

以上代码意味着，对任何类型T和任何类型U，这里可以根据`SmartPtr<U>`生成一个`SmartPtr<T>`。这类构造函数根据u创造对象t，而u和v其实是**同一个对象的不同实例**，我们称之为**泛化（generalized）copy构造函数**

但是有时候转换是单向的->我们不希望根据`SmartPtr<Base>`来创建`SmartPtr<Derived>`，也不希望根据`SmartPtr<double>`来创建`SmartPtr<int>`。我们必须对member template创建的函数群进行筛选

这时就假设我们的SmartPtr也能有一个get()方法返回原始指针，这时我们就可以根据原始指针在构造模板中约束，如下

```c++
template<typename T>
class SmartPtr {
    public:
    	template<typename U>                   // member template
    	SmartPtr(const SmartPtr<U> &other)
    		: heldPtr(other.get())
            { ... };
    	T* get() const { return heldPtr; }
    	...
    private:
    	T* heldPtr;                            // 该SmartPtr拥有的原始指针
}
```

这时，一个`SmartPtr<U>.get()`返回的指针只有能在初值列中转换为T*，才能通过编译，这样就完成了约束

此外，member template function也可以用于赋值操作（P220，unique_ptr，shared_ptr, weak_ptr的相互转换），支持隐式转换则没有explicit修饰。还有就是传递**给shared_ptr的赋值运算符**以及**unique_ptr的copy和赋值运算符**没有const，因为涉及到转移所有权的问题

如果T,U为两个相同类型时，编译器是暗自生成一个copy构造，还是根据另一个同型对象展开构造行为时，将**泛化copy构造函数模板**实例化呢？

答案是前者，会生成普通的copy构造，因为template并不影响语言规则，class内的泛化copy构造函数模板不会阻止编译器自动生成合成copy构造函数，如果想要控制，需要**同时声明泛化copy构造函数模板和正常的copy构造函数**

## 条款46：需要类型转换时请为模板定义非函数成员

（条款24）只有non-member函数才有能力为**所有实参实施隐式类型转换**

如P222的代码：

```c++
template<typename T>
class Rational {
    public:
    	Rational(const T& numerator = 0,
                 const T& denominator = 1);
    	const T numerator() const;
    	const T denominator() const;
    	...
}
// non-member operator
template<typename T>
const Rational<T> operator* (const Rational<T>& lhs,
                             const Rational<T>& lhs);
// 使用失败案例
Rational<int> onehalf(1, 2);
Rational<int> result = oneHalf * 2; // 错误！
```

这里的过程是这样的

1. oneHalf被赋予为`Rational<int>`类型
2. operator*通过oneHalf的类型确认T的类型是`Rational<int>`
3. 发现2的类型是int而非`Rational<int>`，报错

这是因为**此例的template**在实参推导过程中，**从不将隐式类型转换纳入考虑之中**，这样的转换函数想要被使用，首先要被知道

所以我们可以在template class的friend声明里加上特定函数，这也就意味着operator*可以成为friend函数，进而“知道”转换函数的存在

这里有一个trick，如下：

```c++
template<typename T>
class Rational {
    public:
    	...
    friend const Rational operator*(const Rational<T>& lhs,
                             		const Rational<T>& lhs);
}
template<typename T>
const Rational<T> operator* (const Rational<T>& lhs,
                             const Rational<T>& lhs) {
    return Rational(lhs.numerator() * rhs.numerator,
                    lhs.denominator() * rhs.denominator()));
}
```

注意到虽然使用friend，但我们的目的其实不是为了“访问class的non-public成分”

当然，也可以让friend函数都调用**辅助函数**，注意到template class的辅助函数往往也是template，具体实现在P226，比较简单，这里不再赘述

## 条款47：请使用trait classes表现类型信息

C++ primer中提到的5中迭代器分类，这里更进一步给出了专属的卷标结构（tag struct）

```c++
struct input_iterator_tag {};
struct output_iterator_tag {};
struct forward_iterator_tag: public input_iterator_tag {};
struct bidirectional_iterator_tag: public forward_iterator_tag {};
struct random_access_iterator_tag: public bidirectional_iterator_tag {};
```

可以看出其中的is-a关系

如果需要通过是否是random accessible来判断是否用随机访问（advance模板的本质），就需要用traits在**编译期**获得某些类型信息

因为traits是一种**技术**，而不是C++的关键字或构件，它的要求是：对于built-in和user-defined类型必须表现得**一样好**，举个例子，它必须良好运作于不同类型指针和自定义的类上。

因为需要运行于指针上，所以**内置一个trait是不可行的**。标准技术是把它放进一个template及其一个或多个特化版本中。这样的template在std中由若干个，针对迭代器的trait为`iterator_traits`，如下

```c++
template<typename IterT>
struct iterator_traits;        // template,用以处理迭代器分类的相关信息
```

虽然习惯上traits由struct实现，但往往又被称为traits classes

运作方式：针对每一个类型IterT来声明某个typedef名为iterator_category，大概像这样：

```c++
template<...>
class deque {
    public:
    	class iterator {
            public:
            	typedef random_access_iterator_tag iterator_category;
            	...
        };
    ...
};
```

其他容器依次类推

这对user-defined行得通，但对指针（也是一种迭代器）行不通，于是有了`iterator_traits`的第二部分，定义一个针对指针的特化版本，如下：

```c++
template<typename IterT>
struct iterator_traits<IterT*> {
    typedef random_access_iterator_tag iterator_category;
    ...
}
```

综上可看出如何设计一个traits class

1. 确认所有的可取得的类型信息，如五种迭代器类型
2. 为该信息选一个名称，如`iterator_category`
3. 提供一个template和一组特化版本

最后对于advance的实现，根据if语句判断迭代器类型是在运行期，可以使用重载在编译期判断使用哪种迭代器递增方式

## 条款48：认识template元编程

Template metaprogramming(TMP，模板元编程)是编写template-based C++程序并执行于编译期的过程。

TMP的一大优势就是把一些工作从运行期搬到编译期，上个条款中trait的最终重载解法就是TMP。在上条advance实现中，使用if-else的实现版本时，编译器必须确保**所有源码都总是有效，即使这段源码不可能通过if的判断！**但如果是TMP，编译器从最开始就**只会针对被重载的特定的函数版本**

TMP所有的循环效果藉由递归来实现，也就是**函数式语言的特性**

比如factorial（阶乘函数），只有`factorial<0>`是特化，剩下的往`factorial<n-1>`递归就可以了

TMP所能达成的目标：

1. 确保度量单位正确（用于早期的错误侦测）
2. 优化矩阵运算。比如对于`m =m1*m2*m3*m4`这样的连续矩阵乘法，正常运算会产生3个临时矩阵，最后再和m4相乘，但如果使用TMP，就无需使用这么多内存且效率有提升
3. 生成客户定制的设计模式（custom design pattern）。如Strategy，Observer，Visitor等

## 条款49：了解new-delete的行为

当operator new抛出异常以反映一个未获满足的内存需求之前，他会**先**调用一个**客户指定的错误处理函数**，也就是所谓的new-handler。为了指定这个函数，客户必须调用`set_new_handler`（声明于`<new>`中的一个标准程序函数），std中如下：

```c++
namespace std {
    typedef void (*new_handler)();
    new_handler set_new_handler(new_handler p) throw();
}
```

new_handler是一个指针，指向没有参数返回void的函数

throw()是一份空白的一场明细，必定是nothrow(详见条款29)

set_new_handler的参数式new_handler类型，一个**指向无法分配内存时该调用的函数的指针**，返回值同样是一个指针，指向**将要被替换**的new_handler类型指针

当operator new不能满足内存申请时，他会不断地调用new-handler函数，直到找到足够内存（反复调用的代码见条款51）

一个设计良好的new-handler需要做到以下事情：

1. 让更多内存可被调用。比如程序一开始分配一大块内存，先不全给，new-handler被调用后再给
2. 安装另一个new-handler。衍生出一种做法是通过修改诸如static，namespace和global数据来改变new-handler自身的行为
3. 卸除new-handler。传nullptr
4. 抛出bad_alloc。这样异常不会被new捕捉而是传到源码new的地方
5. 不返回。调用abort()或exit



可以通过每个class提供自己的set_new_handler和operator new来实现不同class的不同处理，如下：

```c++
class Widget {
    public:
    	static std::new_handler set_new_handler(std::new_handler p) throw();
    	static void* operator new(std::size_t size) throw(std::bad_alloc);
    private:
    	static std::new_handler curHandler;
}
```

static必须在类外被定义（除非是const还是整型，见条款2），标准版set_new_handler的同作用实现如下：

```c++
std::new_handler Widget::set_new_handler(std:new_handler p) throw() {
    std::new_handler oldHandler = curHandler;
    curHandler = p;
    return oldHandler;
}
```

然后operator new做以下事情：

1. 调用标准的set_new_handler，告知Widget的错误处理函数。这会让Widget定义的new-hanlder安装为global
2. 调用global的operator new，执行内存分配。如果分配失败，调用Widget的new-handler。如果global的operator new无法分配足够内存，抛出bad_alloc。**在此情况下**，Widget的operator new必须恢复原本的global new-handler，再传播异常
3. 分配成功返回指针，Widget**析构函数**会管理global new-handler，**自动将调用前的global new-handler**恢复回来

但注意到上述的new-handler都是会改变global版本，有一种方法可以提供**class专属的new-handler**，如下：

```c++
template<typename T>
void* NewHandlerSupport<T>::operator new(std::size_t size) throw(bad_alloc) {
    NewHandlerHolder h(std::set_new_handler(curHandler));
    return ::operator new(size);
}
// class继承support函数获得专属的new-handler定义方法
class Widget: public NewHandlerSupport<Widget> {
    ...
}
```

注意这里NewHandlerSupport template的T从未被使用，T只作为区分不同derived classes存在，Template机制会自动为每一个T生成一份curHandler，这样同属一个类的所有对象共有一个new-handler

最后注意使用nothrow new只能保证operator new不抛异常，不保证像`new (std::nothrow) Widget`这样的语句不抛异常（new不抛，Widget的构造函数也可能会）

## 条款50：了解new和delete的合理替换时机

替换new和delete的三个常见理由

1. 检测运行上的错误。比如delete失败会产生内存泄漏，多次delete会使行为未定义。这时我们可以让自己的operator new分配额外的空间，然后在额外的空间里防止特定的byte pattern(签名，标识符)，之后如果byte pattern被overrun（写入分配区之后）或underrun（分配区块之前），operator delete就可以根据byte pattern来log事实及相应指针
2. 强化效能。比如根据不同的需求：小块内存，大块内存，混合大小内存分配进行性能优化
3. 为了收集使用上的数据。可以收集到内存使用的数据，做出分析，比如是FIFO还是LIFO？不同阶段的分配和归还次序有不同吗？

P249提供了一个实现，但是没有给alignment（机器上要求4 bytes或8 bytes对齐），通过alignment(对齐)可以提高访问效率

其他理由补充：

4. 增加分配和归还的速度。比如Boost的pool，专门针对大量小型对象的分配
5. 为了降低默认内存管理器带来的空间额外开销。泛用的内存管理器往往慢还使用更多内存，有额外开销，Boost的poo针对小型内存减少开销
6. 弥补默认内存分配器中的非最佳齐位（suboptimal alignment）。比如x86体系结构上doubles的访问最为快速（在8-byte齐位的前提下）
7. 使相关对象成簇集中。当我们直到某个数据结构往往被一起使用，又希望将page faults降至最低，那么可以为该数据结构另创建一个heap，是的它们被**成簇集中**在尽可能少的pages上（条款52的placement）
8. 为了获得非传统行为。比如通过C API补充分配和归还**共享内存**的区块

## 条款51：编写new和delete时需固守常规

P242一个new需要的要素：

1. 分配成功时返回指针
2. 内存不足时无限循环调用new-handler
3. 必须对应**零内存**需求

如果operator new是base class专属的，我们可以在operator new中加上以下部分：

```c++
void* Base::operator new(std::size_t size) throw(std::bad_alloc) {
    if (size != sizeof(Base)) return ::operator new(size);
}
```

注意，这里同时也处理了零内存需求：因为`sizeof(Base)`不可能是0，所以当size为0时会交付给标准的operator new处理

对于operator new[]，注意一定需要一个size_t参数，但真实分配的一定比size_t要大（我们需要一额外空间存放元素个数，条款16）

对于delete，注意C++保证**删除null指针永远安全**，而上面提到了大小不符合base的new操作会被转交给标准new，delete也做一样的事

注意如果即将被删除的对象是一个derived class，且base class欠缺virtual析构，那么size_t的数值可能不准去（所以base classes最好拥有一个virtual析构函数）

## 条款52：写了placement new也要写placement delete

如果operator new接受了除了必须有的size_t以外的参数，那么这就是所谓的placement new。有一个placement new版本十分有用，如下：

```c++
void* operator new(std::size_t, void* pMemory) throw();
```

pMemory指向**对象被构造之处**，该函数可通过`#include <new>`取用。其用途之一是负责在vector的未使用空间上创建对象，同时也是最早的placement new版本。

当执行如下操作时：

```c++
...
    static void* operator new(std::size_t, std::ostream& logStream)
    	throw(bad_alloc);
...

// 使用上述的operator new
Widget* pw = new (std::cerr) Widget;
```

因为先执行new后执行Widget构造，如果先分配成功，而Widget构造抛出异常，那么运行期系统需要释放分配的内存并恢复之前的状态。但运行期系统不请楚new的内部如何运作，只能通过查找**参数个数和类型都与operator new相同**的某个delete版本，本例中，operator delete版本是：

```c++
void operator delete(void*, std::ostream) throw();
```

如果没有参数匹配的delete版本，那么**没有任何operator delete会被调用**，造成内存泄漏

但placement delete**只有在伴随placement new调用而触发的构造函数**出现异常时才会被调用，如果使用`delete pw;`，placement delete并不会被调用，只会使用正常形式的operator delete

注意自定义的new和delete会把从base继承而来的和std里global版本的**全部隐藏**，想要同时保有的话：

1. 对于global版本（P260），建立一个base class，内含所有正常的new和delete
2. 利用继承机制和using声明把base的版本都可见

## 条款53：不要轻易忽视编译器的警告

（著名表情包：warning警告牌只有程序员开下山）

## 条款54：让自己熟悉包括TR1在内的标准程序库

这里有些报菜名的意思，大体都在C++ primer提到过了，这里不多赘述

## 条款55：让自己熟悉Boost

总的来说就是C++程序员的github社区



终于看完了，内容多干货多，这么快看完只怕是记不住，回头还要来复习#捂脸