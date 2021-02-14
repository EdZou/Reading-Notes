# 《C++ Primer》的读书笔记

该笔记主要是总结一下书中的要点，便于以后查阅

## 第一章 开始

1. C-like函数的介绍，包括花括号内的函数体

2. 编译及运行程序的命令：`cc prog1.cc` 。在win中会生成`prog1.exe` 而unix会生成 `a.out`

3. 运行文件：

   ​	Win: `.\prog1  #.exe后缀可省略`

   ​	Unix: `./a.out  #与运行.sh脚本一样使用./`

4. 两种系统都可以用`echo`语句获得返回值

   ​	Unix: ```echo $?```

   ​	Windows: ```echo %ERRORLEVEL%```

5. GNU编译器的使用

   ​	```g++ -o prog1 prog1.cc``` -> 生成一个object（在这里也就是可执行文件名）为prog1.exe的文件

6. 输入输出流的使用：

   ​	`std::cout << "Hello World" << std::endl` 标准输出(注意`endl`会刷新缓冲区buffer)

   ​	`std::cin >> param1 >> param1` 标准输入（第一个值存入param1，第二个值存入param2）

   ​	`std::cerr` 标准错误

   ​	`std::clog` 一般日志

7. 注释

   ```c++
   /*
   
    *a sample interpretation
   
    */
   ```

   以及 `//` 符号, 可嵌套注释（一些编辑器支持选中后`ctrl + /`多行注释）

8. `while`，`if` 和`for`语句

   ​	因为太熟悉了，这里只做一些简单的补充

   ​	在`for(A;B;C,D)`中，

   ​		A为循环初始化，可忽略

   ​		B为判断条件，条件符合继续循环，不符合中断

   ​		C, D为每次循环后需要执行的语句

9. 类的成员函数，也被称为方法，使用`.`运算符找到成员，`()`调用运算符使用

   ​	如：`item1.isbn()`

## 第二章 变量和基本类型

1. 算术类型的尺寸（P30），因为C++不像Java有JVM，C++的基本类型的比特数量根据不同机器有所差别，P30记录了这些算术类型最小的尺寸

   **重要区别** ：Java中int稳定为4 byte = 32 bit，而C++中int最小可为 2 byte = 16 bit

2. signed/unsigned（signed char 表示 -128-127）

   ​	选择原则：

   	1. 当知道为非负数选unsigned
    	2. 整数运算选int，超过范围用long long
    	3. 算术表达式避开char & bool
    	4. 浮点数运算最好double，可能比float还快，但注意long double的消耗过大一般不宜使用

3. 类型转换：

   1. 对于bool值而言，0值被转化为`false`，非零值为`true`
   2. 但将bool值转化其他类型时，`false`转化为0，`true`转化为1
   3. 浮点 -> 整型把小数点截断
   4. 整型 -> 浮点如果整型容量过大（如：long long -> float）会导致精度损失
   5. 当给unsigned类型超出范围的值时，如给unsigned char赋予-2时，结果是取模后的余数（mod为无符号类型的表示范围，此例中unsigned char表示0-255，mod = 256），其结果为254
   6. 如果给signed类型超出范围的值，结果为**未定义的（undefined）**

4. 字面常量：

   1. 三种常见进制：`20 /*十进制*/   024 /*八进制*/  0x14 /*十六进制*/`

   2. 字符串字面值如：`"Hello World"`编译器会自动在字符串结尾处加上`\0`（以前用C写lc老是忘，特此备注）

   3. 字符串太长时分行书写：

      ```c++
      std::cout << "Hello 123456789"
          		 "World" << endl
      ```

   4. `\`的转义符（P36罗列了常用的转义符）

   5. P37罗列了字面值类型常用的前/后缀（如：u(U), l(L), ll(LL)等）

5. 变量定义：

   1. C++中需要类型说明符(type specifier)
   2. 列表初始化：C++中可在赋值和初始化时使用花括号，如：`int i = {0}; 或者 int i{0};`但这种方法可能导致丢失数据，最好还是使用`int i = 0; 或者 int i(0);`
   3. 当内置类型未被显式初始化时，定义在函数内的变量不被初始化，定义在函数外的变量一般默认初始化

6. 变量声明和定义：

   1. 假设我们声明而非定义一个变量，如：

      ```c++
      extern int i; //一般extern声明在头文件中，定义在其他.cpp文件中
      ```

      这就是一种将定义与声明分离的方法

   2. 标识符规范：驼峰（someItem）,下划线（some_item）

   3. 变量作用域：起始于声明语句，一般以声明语句作用域的末端为结束

7. 复合类型：

   1. 这里特指**引用(reference)**是C++及其重要的特征，其底层就是指针(这部分实验证明曾在某个博客上看到)

      ```c++
      int val = 1024;
      int &ref = val; //ref是val的引用，也就是val的别名，可以通过ref对val进行修改
      int i = ref; // i -> 1024
      ```

   2. 一个变量可以有多个引用，但不能有引用的引用

   3. 定义引用必须是用左值（左/右值的概念后面会提到）

   4. 指针也可以有引用，用法和其他非指针一样

8. 指针（我之前使用C，个人非常熟悉了，这里不多赘述）

   1. 但注意一下C++中定义空指针的写法：

      ```c++
      int *p1 = nullptr; // C++中标准的写法, 等价于int *p1 = 0;
      int *p1 = NULL // C的老写法，也等价于int *p1 = 0;
      ```

   2. 两个**指向相同类型的指针**比较时，可用`==`和`!=`，以`==`为例，存放地址值相同则true，不然就是false

9. 定义多个变量：

   1. 一种让人误会的写法：

      ```c++
      int *p1, p2; //这种写法p1是int*指针而p2是int值
      int *p1, *p2 //正确定义两个指针
      ```

10. const限定符(重点)

    1. 默认const对象仅在文件内有效，如：`const int queueSize = 128`，编译器编译时会找到所有`queueSize`并使用128替换

    2. 需要其他文件共享：

       ```c++
       extern const int queueSize; //file1.h 头文件
       extern const int queueSize = 128; //file1.cc 定义并可被其他文件访问
       ```

    3. const的引用(P55):

       1. 初始化const引用时可以用**任意**表达式(非const对象,字面值和一般表达式)，只要该表达式能转化为对应的引用类型，其本质是：

          ```c++
          double val = 1.23
          const int &ref = val
          -->
          const int temp = val;
          const int &ref = temp; // 绑定到临时对象上
          ```

       2. 而如果ref不是常量，编译器认为程序员希望通过ref修改val的值，但是ref实际指向一个临时量，故而报错。

       3. 常量的引用只会限制通过该const引用进行修改，而不限制其他的手段，如：

          ```c++
          int i = 1;
          const int &ref1 = i;
          int &ref2 = i;
          ref1 = 0; // 报错，不能通过const引用修改值
          ref2 = 0; // 正确，即使i被const引用，也可以通过其他的非const引用对其进行修改
          ```

    4. 指针与const：

       1. 常量必须由const指针指向，但是const指针可以指向非const变量，与const引用一样，我们仅仅是不能通过const指针对其指向的对象进行修改，但不禁止通过其他手段修改其指向的对象
       2. `*`在const前则为常量指针（顶层const，不能改变自己的值），即指针本身是个常量，不允许++，--等操作。如`int *const ptr1 = &i;`以及`const int i = 0;`
       3. `*`在const后（底层const，引用的声明都是底层const）意味着该指针指向的对象为常量，即不允许通过本指针修改其指向的对象, 但可以修改指针本身，如`const int *ptr2 = &j`, 也可以定义一个常量指针指向常量， 如`const int *const ptr2 = &j`
       4. 当进行拷贝时，右值具有底层const时，左值必须拥有相同的底层const，不然会报错。但左值拥有底层const时，右值不必要

    5. constexpr常量表达式

       1. 值不会改变，且编译过程中就可以得到结果

          ```c++
          const int val1 = 100; //是常量表达式
          int val2 = 100; //不是常量表达式，会改变
          const int val3 = get_val() //不是常量表达式，运行时才能得到结果
          ```

       2. 可以使用constexpr显式地让编译器检测是否为常量表达式

          ```c++
          constexpr int val1 = 100; //正确
          constexpr int val2 = val1 + 1; //正确
          constexpr int val3 = get_val() //只有当get_val()是一个constexpr函数时正确
          ```

       3. 一般算术类型，引用和指针都是字面值类型

       4. 当constexpr限定指针时，它只会影响顶层const，即将指针定义为常量指针，如

          ```c++
          const int *ptr1 = nullptr; //底层const，指向对象为常量int的普通指针
          constexpr int *ptr2 = nullptr; //顶层const，ptr2自身为常量指针，等同于int *const ptr2 = nullptr;
          ```

11. 类型处理

    1. 传统方法`typedef`, 如`typedef name nick_name`, 将nick_name作为name的别名

    2. 也可以用`using`, 如`using nick_name = name`, 将nick_name作为name的别名

    3. 对于指针：

       ```c++
       typedef char *pstring; // pstring是char*的别名
       /* 注意！！！
        * 不能完全直接替换，如下例中
        */
       const pstring ptr1 = 0; //ptr1是指向char的常量指针
       /*
        * 也就是说该例中，ptr1不是等同于 const char *ptr1 = 0；
        * 而是等同于 char *const ptr1 = 0;
        */
       const pstring *ptr2 = 0; //ptr2是一个普通指针，指向一个指向char的常量指针
       // 上例中该式等同于char* const *ptr2 = 0;
       ```

    4. `auto`作为自动分析类型符号，之后会有更详细的说明，可以用在函数返回值中。这里注意`auto`一般会**自动忽略**顶层const，**保留**底层const

    5. `auto`不会自动保留引用特性：

       ```c++
       const int ci = 0, &cr = ci;
       auto a = ci; // a是一个整型，不保留const
       auto b = cr; // b和a一样是一个整型，既不保留const也不保留引用
       ```

    6. 如果希望`auto`推出一个顶层const，需要手动添加`const auto f = ci;`，注意以下例子:

       ```c++
       // 将引用的类型设为auto
       const int i = 0;
       auto &g = i; // 此时g为常量的引用，顶层const得到保留
       auto &h = 42; // 错误：不能为非常量引用绑定字面值
       const auto &j = 42; // 正确：可以常量引用绑定字面值
       // auto同时对多个变量初始化需要保证类型一致（对const敏感）
       ```

    7. `decltype`可分析函数的返回值类型，不同于`auto`，`decltype`会保留包括顶层const和引用在内的特性（引用除了在`decltype`中展现底层是指针的特性，在其他所有地方都作为一个同义别名存在）

    8. ```c++
       // 一种特殊情况
       int i = 42, *p = &i, &r = i;
       decltype(r + 0) b; // 正确，因为（r+0）返回int
       decltype(*p) c; // 错误，c是int&，引用需要初始化
       ```

       如果`decltype`中表达式解引用，那么返回的就是引用值（上例c）

    9. 如果`decltype`内部加上了`()`，则会被推测为表达式从而变成引用类型，如

       ```c++
       int i = 42;
       decltype((i)) d; //错误， d推断为int&，引用需要初始化
       ```

    10. 头文件保护符 `#ifdef/#ifndef` + `endif`来保证不会重复定义

        

## 第三章 字符串，向量和数组

1. `using`语句，用于确定命名空间，比如`using std:cout` 去使用标准库的cout

2. `std::string`

   1. 初始化的几种方式

      ```c++
      string s2(s1);
      string s2 = s1;
      string s3("Hello"); // 直接构造，更快
      string s3 = "Hello";  // 拷贝构造，需要生成临时量
      string s4(n, 's'); // s4等于n个's'组成的string，直接构造
      ```

   2. P77有一些C++ string的基本操作，和python中基本类似这里不再赘述

   3. 注意用`cin >> str;`读取string，遇到空白（空格，换行，制表等等）就会终止读取，按行读取需用`getline(cin, str)`

   4. `str.size()`返回的是类型为`string::size_type`类型的值

   5. string类型的比较就是按顺序比，谁的第一个不同的字符更大谁就大，与长度无直接关系

   6. string的相加：

      1. 不同于python可以自由相加，C++中以下情况注意是不行的

         ```c++
         string s1("World");
         string s2 = "Hello" + " " + s1; // 错误
         // +的两边之至少要有一个是string，该例中"Hello"和" "本质都是char*，没有opertaor+
         ```

   7. 一些cctype头文件中提供的string处理方法（P82）

   8. 可用下标符号`[]`进行随机访问

3. `std::vector`

   1. 初始化大体与`std::string`类似，多了一个列表初始化：

      ```c++
      vector<T> v5{a,b,c,...};
      vector<T> v5 = {a,b,c,...}
      ```

      vector的默认初始化是一个空表

      值初始化：

      ```c++
      vector<int> ivec(10); // 10个0元素
      vector<string> svec(10); // 10个空string
      ```

   2. for循环访问vector`for(auto &i : v)`，来将v中的所有元素（的引用）进行遍历

   3. vector支持随机访问，但是添加元素不要用下标而是`push_back()`

4. 迭代器（重点）

   1. `begin()`和`end()`，end返回（本不存在的）尾后迭代器
   2. 迭代器指向元素的引用，`*iter`是一个元素的引用， `iter->mem`取元素的成员，用++/--对迭代器进行移动
   3. 可用`.cbegin()`和`.cend()`来返回`vector<T>::const_iterator`
   4. P99记录了迭代器的运算，string/vector都是支持随机访问的，迭代器之间可以通过减法算距离，返回值类型为`difference_type`

5. 数组

   1. 类似于vector但是大小固定

   2. 如字符数组：

      ```c++
      char a1[] = {'1','2','3'}; // 初始化没有空字符'\0'，与C一脉相承，编译器不报警，大坑
      char a2[] = {'1','2','3','\0'}; // 有显式空字符
      char a3[] = "123"; // 有隐式的空字符
      const char a4[3] = "123"; //报错，没有空间存放尾字符
      ```

   3. 不允许拷贝和赋值

   4. 复杂声明：

      1. `int *ptr[10]` 是10个指针的数组（指针的数组）
      2. `int &ref[10]`字面是10个引用的数组，但是实际上不存在
      3. `int (*Parray)[10] = &arr`是指Parray指向10个元素的数组（数组的指针）
      4. `int (&Parray)[10] = arr`是数组的引用

   5. 一般情况下数组会自动转化为指针，但使用了`decltype`之后就不会了

      ```c++
      // ia是一个10整型的数组
      decltype(ia) ia3 = {0,1,2,...,9};
      string nums[] = {0,1,2,...,9};
      ia3 = nums // 错误，不能让整型指针给数组赋值
      ```

   6. 数组的指针几乎能做和迭代器一样的事，两个指针相减返回类型是`ptrdiff_t`，必须要两个指针指向同一数组内的元素才有意义。

   7. 下标随机访问可以使用负数，如`int k = p[-2]`，与python不同，C++中k得到的是p指针左移两次之后的对象

6. C-style字符串(char [] ...)

   1. 很多C++操作都是未定义的，比如比较需要用`strcmp()`函数去比较字符串内容，如果用普通的关系运算符，本质是比较地址值的大小，没有意义

7. 多维数组，只要没有访问到最低一层的维度，本质存的都是相应的指针

## 第四章 表达式

1. 左值和右值

   1. C中位于表达式左侧就是左值，右侧就是右值
   2. C++更加复杂，但总结就是：对象被用作右值时，用的是对象的值（内容），被用作左值时，用的是对象的身份（内存中的位置）
   3. 一些常用的情况：
      1. 赋值运算需要非常量左值，得到一个左值
      2. 取地址符从一个左值运算对象得到一个指针（右值）
      3. 解引用，下标运算，迭代器解引用得到的求值结果都是左值

2. 这一章一大部分都是结合律和优先级的介绍（详见P147表），这里不多赘述

3. 注意`sizeof`运算符：

   1. 两种形式：`sizeof (type)`和`sizeof expr`
   2. 第二种形式返回表达式**结果**的大小（所占字节数量）

4. 隐式转换：

   1. 算术转换会把小的变量转换成大的再计算
   2. 除去`decltype, &, sizeof, typeid`以外的操作时，数组可被自动转化为头元素的指针
   3. 根据零值转化为bool型
   4. 非常量->常量

5. 显示转换：

   1. `static_cast`：

      1. 不包含底层const
      2. 可以将大算术类型赋予小算术类型（无警告，可能精度损失）

   2. `const_cast`:

      1. 只改变底层const，不能改类型（会报错）
      2. **只能**通过该方法将常量对象转为非常量或者非常量->常量

   3. `reinterpret_cast`:

      1. 将对象的位模式重新解释，如：

         ```c++
         int *pi;
         char *pc = reinterpret_cast<char*>(ip);
         ```

         虽然pc变成了`char*`，但pc指向的对象仍然是int，如果把pc完全当成`char*` 而使用其他操作会有非常大的危险性，且编译器不会有反应

   4. 早期强制转型：

      1. `type (expr);` 函数形式转换
      2. `(type) expr;` C语言风格转换，与`reinterpret_cast`相似

## 第五章 语句

这一部分其实也比较熟悉了，这里仅作简单的补充

1. 花括号又称块，也就是复合语句，可用于防止悬垂else
2. 多分支用switch...case...
3. try语句必须对应一个或多个catch语句，catch能够接取throw抛出的异常
4. P176罗列了一些标准的异常处理语句

## 第六章 函数

这一部分同样是所有语言都共有，这里仅把一些C++特有的和我个人不够熟悉的部分记录下来

1. C++中局部变量会**隐藏**外层作用域中同名的变量

2. 局部静态对象和类中static一样，直至程序终止才被销毁，会被所有的基类/子类/函数自身调用共用

3. 分离式编译：

   假设我们将`fact`函数定义在fact.cc文件中，而它的声明在chapter6.h头文件中，且factMain.cc中定义了main函数并调用了`fact`函数，我们为了生成exe文件，需要：

   ```sh
   CC factMain.cc fact.cc #生成factMain.exe(windows) 或者 a.out(unix)
   CC factMain.cc fact.cc -o main #生成main.exe(windows) 或者 main(unix)
   ```

   而如果这个factMain.cc调用了非常多其他文件，而我们只修改了其中fact.cc，大多数编译器可以分离式编译，其过程如下：

   ```sh
   CC -c factMain.cc # generates factMain.o
   CC -c fact.cc # generates fact.o
   CC factMain.o fact.o # 生成factMain.exe(windows) 或者 a.out(unix)
   CC factMain.o fact.o -o main #生成main.exe(windows) 或者 main(unix)
   ```

4. 参数传递

   1. 主要是传参和传引用，传参是传一个copy的副本，函数的内外两变量不相关，而传引用传的是外部参数的引用，虽然不需要拷贝的过程，但在函数内部对引用的修改也会改变外部变量的值

   2. const形参和实参：

      1. 向函数内传参数时，顶层const被忽略，且当形参有顶层const时，传入参数为常量或非常量皆可：

         ```c++
         void fcn(const int i); // 调用时i可为const也可不是，但fcn不能改变i的值
         // 重载时不能以形参有无const进行区分
         // 如果定义以下：
         void fcn(int i); // 报错：重复定义fcn函数
         ```

      2. 尽量把不会改变的形参定义为常量引用，如`print(const string &s)`

   3. 数组形参：

      1. 以下三种形参等价：

         ```c++
         void print(const int*);
         void print(const int[]);
         void print(const int[10]); //这里的10作为维度表示我们期望数组维度为10，但实际不一定
         ```

      2. 数组的引用形参：

         如果使用`void print(int (&arr)[10])`作为函数定义，那么与上面不同，我们**只能**使用大小为10的数组作为参数

      3. 传多维数组本质传指针

5. main处理命令行选项

   比如我们使用下列命令行：

   ```
   prog -d -o ofile data
   ```

   对可执行文件prog传参

   那么，对于`int main(int argc, char* argv[]){...}`有：

    1. `argc` 为参数的数量+1，该例中为5，也是`argv`中字符的数量

    2. `argv`时C-style字符串的指针，类型为`char**`

       本例中

       ```c++
       char **argv = {"prog", "-d", "-o", "ofile", "data", 0} // 0意味着'\0'
       ```

6. 可变形参

   1. `initializer_list<T>`,提供的操作在P198
   2. 在实参数量未知但参数类型全部相同时可以使用，比如有数量未知的字符串时，可用`initializer_list<string>`作为形参
   3. 省略形参，之后会介绍到用递归的方法调用

7. 返回类型和return语句

   1. void函数return语句可省略
   2. 不要返回局部变量的引用或者指针（尽量返回副本），因为函数结束后局部变量会被释放，返回的引用/指针会指向未定义的空间
   3. 尾置返回类型：`auto func(int i) -> int(*)[10]`
   4. 使用decltype: `decltype(i) *arrProcess(int j)`返回一个i类型的指针

8. 函数重载

   1. 同名函数根据形参类型的不同，有编译器决定调用哪个函数
   2. 就像前面提过的，形参的顶层const不会作为区分的依据，会报重复定义的错

9. `const_cast`和重载

   如下例所示，我们可以用`const_cast`作为重载的辅助

   ```c++
   const string &shorterString(const string &s1, const string &s2){
       return s1.size() <= s2.size()? s1: s2;
   }
   // 非常量重载版本使用const_cast
   
   string &shorterString(string &s1, string &s2){
       auto &r = shorterString(const_cast<const string&>(s1),
                               const_cast<const string&>(s2));
       return const_cast<string&>(r);
   }
   ```

10. 重载与作用域

    1. 不同作用域中无法重载，例如：

       ```c++
       void A(int i);
       void A(double i);
       void B(string& s) {
           void A(long long i); // 该声明会把之前外层作用域所有的A函数实现给隐藏
           //B函数的作用域内只有该long long版本的A函数了
           A(3.14); // 报错，A(double i)被隐藏了
       }
       ```

11. 默认实参

    1. 注意一旦某个形参被赋予了默认值，之后所有的形参都必须要有默认值

    2. 使用默认形参只要省略就好

    3. 与python不同，C++只能省略尾部的实参，如`window = screen(, , '?')`的写法是错误的

    4. 假如给定以下声明：

       ```c++
       string screen(sz, sz, char = '');
       // 我们不能修改一个已经存在的默认值
       string screen(sz, sz, char = '*'); // 错误的，报重复声明
       // 但我们可以添加默认实参
       string screen(sz = 0, sz = 1, char); // 正确
       ```

    5. 如果用外部作用域变量作为默认实参，那么如果改变外部作用域的值，默认参数也会随之变化，在新的作用域里定义的同名变量则会隐藏外部变量，故而不会改变默认实参

       如下所示：

       ```c++
       sz wd = 80;
       char def = ' ';
       sz ht();
       string screen(sz = ht(), sz = wd, char = def); // 使用默认实参
       
       void f2() {
           def = '*';
           sz wd = 100;
           screen() // 此时，def值被改变，默认实参变为'*'
           // 但wd被隐藏，默认实参仍然是80而非100
       }
       ```

12. 内联函数和constexpr函数

    1. inline用于规模小并频繁被调用的函数，减少开销，很多编译器不支持

    2. constexpr函数：

       1. 所有形参类型以及返回值类型都是字面值类型，函数体中有且只有一个return

       2. 是一个隐式的inline函数

       3. 函数体内允许有空语句，类型别名，using声明等

          如下函数：

          ```c++
          constexpr size_t scale(size_t cnt) {return new_sz() & cnt;}
          int arr1[scale(2)]; // 正确，传入参数为constexpr，scale(2)也是constexpr
          int i  = 2;
          int arr2[scale(i)]; // 错误，scale(i)不是constexpr
          ```

    3. inline和constexpr函数一般定义在头文件中

13. assert预处理宏，一般`assert(expr)`，位于头文件*cassert.h*中，一般用于单元测试

14. 预处理器定义的有用的名字：

    1. `__func__`一个const char的静态数组，存放函数名字
    2. `__FILE__`存放文件名的字符串字面值
    3. `__LINE__`存放当前行号的整型字面值
    4. `__TIME__`存放文件编译时间的字符串字面值
    5. `__DATE__`存放文件编译日期的字符串字面值

15. 实参匹配的优先级P219

16. 函数指针：

    1. 规则如下：

       ```c++
       // 定义一个函数指针仅仅有输入输出类型，而未初始化
       bool (*pf) (const string &, const string &);
       // 定义一个和函数指针同样形式的函数
       bool lengthComp(const string &, const string &);
       // 下面两种赋值都是可以的
       pf = lengthComp;
       pf = &lenthComp;
       // 使用时，以下三种皆可
       bool b1 = pf("123", "456");
       bool b2 = (*pf)("123", "456");
       bool b3 = lengthComp("123", "456");
       // 可以使pf指向nullptr
       pf = 0;
       // 注意！！！指向不同类型的函数指针不存在转换规则，也就是要严格匹配
       bool strComp(const char*, const char*);
       pf = strComp; // 错误，输入形参不匹配
       ```

    2. 重载函数的指针

       同样需要精准匹配

       ```c++
       void f(int*);
       void f(double);
       
       void (*pf1)(int*) = f; // 指向第一种f函数版本
       ```

    3. 函数指针作为形参

       1. 被自动的转化为函数指针类型

       2. 避免繁琐，可以使用`typedef`和`decltype`来声明函数类型

          ```c++
          typedef bool Func1(const string &, const string &);
          typedef decltype(lengthComp) Func2; // Func1和Func2等价，都是函数类型
          ```

          

    4. 

       

## 第七章