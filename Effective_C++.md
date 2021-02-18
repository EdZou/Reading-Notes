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







