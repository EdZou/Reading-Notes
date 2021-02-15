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

## 第七章 类

1. 成员函数必须声明在类的内部，但它的定义**可以在类的内部也可以在类的外部**

2. 定义在类的内部的函数是隐式的inline函数

3. `this`

   1. 使用点运算符时，调用成员函数，成员函数隐式地使用`this`访问调用它的对象
   2. 当调用`total.isbn()`时，编译器负责将total的地址传给isbn的隐式形参`this`
   3. `this`的类型是`class(这个类名) *const`，即指向对象引用的常量指针，*这意味着我们不能再一个常量对象上使用 `this`*

4. const成员函数：参数列表后跟const的成员函数，**不能改变**调用它的对象的内容

5. 编译器两步处理类：编译成员声明，再编译成员函数体

6. 构造函数

   1. 不能被声明成const函数，因为直到完成初始化，对象才取得常量属性，这也意味着，构造函数构造过程中可以对const成员写值
   2. 编译器提供的构造函数为合成构造函数，但我们不能依赖默认构造函数，三原因P236

7. 可以显式使用`=default`声明使用默认构造函数，类内使用为inline，否则不是

8. 类外定义构造函数需要指定是哪个类的成员，如`Sales_data::Sales_data(std::stream &is)`

9. 访问控制`public`, `private`和`protected`三兄弟不再赘述

10. `class`和`struct`唯一的区别，struct默认第一个访问说明符之间都是public，class都是private

11. 友元的声明只能在类的内部，可以访问private成员

12. 类内定义的别名`typedef`和`using`同样有访问限制

13. `mutable`关键字

    1. mutable成员永远不是const，即便是const成员函数也可以对其改变

14. 类内初始值（`{}`或者`=`）

    ```c++
    class WindowMgr {
        private:
        	std::vector<Screen> screens{Screen(80,24,' ')};
        	int i = 0;
    }
    ```

15. 返回`*this*`的成员函数

    ```c++
    class Screen {
        public:
        Screen &set(char); //注意返回Screen的引用
    }
    inline Screen &Screen::set(char c) {
        contents[cursor] = c;
        return *this;
    }
    ```

    该例如果set返回的不是引用，那么就会返回一个副本，不会改变原对象的值

16. P248提供了根据对象本身是否是const来重载函数的例子

17. `class Screen`是一个前向声明，它在完成定义前是一种*不完全类型(incomplete type)*，类必须要先完成定义才能通过引用或指针访问成员，如果不定义编译器无法得知分配多少内存。故而一个类的成员不能包含类自己（套娃，无法确定分配多少内存），但可以定义指向自己的指针或引用（引用本质也是指针，大小确定），或者用静态声明不完全类型

18. 友元关系不具备传递性，”朋友的朋友不是我的朋友“

19. 如果有多个重载的函数，只有精准匹配的那个版本可以获得友元权力

20. 类成员生命查找：

    如下例中

    ```c++
    typedef double Money;
    string bal;
    class Account {
        public:
        	Money balance() {return bal};
        private:
        	typedef double Money; // 不能重新定义Money
        	Money bal;
    }
    ```

    当编译器看到balance()的声明，先往**类内，balance之前的部分查找**，没有找到匹配成员再往外层作用域查找

    这里即使类内定义的Money与外层作用域一致也会报错，因为成员balance()已经使用了外部作用域的定义别名

21. 成员函数的作用于查找类似，详见p255

22. 如果有const成员，那么构造函数中必须对其进行初始化

23. 尽量用构造函数的参数作为初始化成员的值

24. 委托构造函数，本质是调用其他的构造函数并且在函数体内加上自己的部分，如下

    ```c++
    Sales_data(std::istream &is): Sales_data() { read(is, *this); }
    ```

    这个构造函数委托了另一个构造函数完成部分构造并在后续的`{}`内加上了自己的部分，这里的逻辑是**先**执行**被委托**的构造函数的初始化列表和函数体，再将控制权给委托者`{}`内的部分

25. 默认构造函数的三个情形P262

26. 隐式的类类型转换：

    1. 比如下例：

       ```c++
       Item::combine(Sales_data d);
       // 隐式创建Sales_data
       item.combine(cin);
       ```

       本质是先用cin创建一个临时的Sales_data，再将临时对象传递给item，创建完后会被丢弃

       但是只能一步转换，比如下例

       ```c++
       item.combine("123456");
       // 实际包含两步
       // "123456" -> string("123456")
       // string -> Sales_data(string)
       // 故而报错
       ```

    2. 可用关键字`explicit`抑制隐式转换，但只对**单实参**构造函数有效（因为需要多个实参的原本就不能隐式转换），可以用C++的`bind`函数来解决，类似于python的装饰器

    3. 被`explicit`限制的构造函数只能用于直接初始化，不能用于拷贝形式

    4. `static_cast`可以无视`explicit`强制隐式转换，如`item.combine(static_cast<Sales_data>(cin));`

27. *聚合类(aggregate class)*

    1. 所有成员都是public
    2. 没有定义任何构造函数
    3. 没有类内初始值
    4. 没有基类和虚函数
    5. 初始值的顺序必须与生命顺序一致

28. 数据成员都是字面类型的聚合类就是*constexpr类*，与单纯聚合类不同，可以定义constexpr构造函数（P268有字面值常量类的例子）

29. 静态成员不能是const，被所有子类共享，不能在static函数内使用`this`（因为不清楚`this`指向哪个对象）

30. 类内声明类外定义static成员时，在类外不能重复static关键字，一旦static被定义，就存在于程序的整个周期之中

31. 类内（一般在类外初始化）初始化static成员，成员必须是constexpr且初始值也要是constexpr

## 第八章 IO类

1. IO对象无拷贝或赋值
2. P279，280标注了流的条件状态和相应解释
3. `endl`刷新缓冲区等于加上`\n`，`flush`只刷新缓冲区不加额外的字符，例：`cout << "Hi" << flush`
4. 可用`cout << unitbuf`来每次操作后都刷新一次缓冲区，`cout << nounitbuf`回归正常
5. 关联两个流用`tie`，如：`cin.tie(&cout)`，这样每当cin读取数据都会先刷新cout的缓冲区，不再关联就用空指针：`cin.tie(nullptr)`
6. 文件输入/出详见P283，P286设定文件的读写模式（这里有些细节需要注意），与其它语言相差不大，不再赘述

## 第九章 顺序容器

这一部分主要都是数据结构相关，已经比较熟悉，不会在本笔记中记录过多

1. 先介绍几种基本结构：

   1. `vector`: 顺序存储，最常用，可快速访问，可变大小的数组，当元素数量超过分配大小时会重新分配（分配一个大小是原来两倍的空间）并将之前的元素拷贝过去
   2. `deque`: 双端**队列**（python中有类似实现），在头尾插入/删除很快
   3. `list`: 双向链表
   4. `forwarded_list`：单向链表
   5. `array`: 固定大小数组，不能添加或删除元素
   6. `string`: 与`vector`类似，但专门保存字符，连续存储空间

2. P295记录了详细的容器操作

3. 迭代器**左闭右开(很重要！！！)**，[begin, end)

4. 这些类型都是模板，需要在尖括号内提供具体类型来实例化（16章会说）

5. `.cbegin()`返回`const_iterator`，不能修改元素

6. `.rbegin()`返回`reverse_iterator`，按逆序寻址元素的迭代器

7. P299有容器的初始化方法，大体和之前提到的没有区别，就多了一个`C c(b,e)`，拷贝迭代器b和e之间的元素给c

8. 当一个容器初始化为另一个容器的拷贝时，两个容器的容器类型和元素类型必须完全一致

9. `array`需要在初始化时指定大小，如`array<int, 42>`表示保存42个整型，用列表初始化时，不够的元素都会被值初始化

10. `array`不支持`assign`和花括号的值列表进行赋值

    `assign`三种操作

    	1. `seq.assign(b,e)`将seq中元素替换成迭代器b，e中间的元素。b，e不能指向seq中原有的元素
     	2. `seq.assign(il)`将seq中元素替换成`initializer_list`中的元素
     	3. `seq.assign(n, t)`将seq中元素替换成n个值为t的元素

11. 允许使用`assign`对一个不同容器和数据类型但相容的类型赋值

12. `swap`本质交换指针，所以o(1)内可以完成

13. 所有容器都支持（==，!=），顺序容器支持关系运算(>,<等，本质是对元素的逐对比较)

14. P305罗列了顺序容器添加元素的做法， P311则罗列了删除元素的做法

15. insert方法返回指向插入元素的迭代器

16. `c.front()`和`c.back()`分别是头尾元素的引用，当容器为空，这两个都是未定义

17. 使用下标超界直接报错，`c.at()`会throw一个`out_of_range`异常

18. `c.erase(b, e)`一样是左闭右开，返回被删元素之后的迭代器（该例返回e）

19. `forwarded_list`不用`insert,emplace,erase`，而是用`insert_after,emplace_after,erase_after`。原因不必赘述，但凡刷过lc的都懂单向链表。但注意`lst.erase_after(b,e)`是**左开右开**

20. 改变容器大小`resize`

    1. `c.resize(n)`，n大于原本大小就把多出来的元素丢弃，n小于原本大小且必须添加新元素，就对新元素值初始化
    2. `c.resize(n,t)`，调整c的大小为n，任何新添加的元素都用t填充

21. 添加元素操作可能使容器失效

    1. 对于vector和string，如果存储空间被重新分配，那么迭代器，引用，指针（三个本质都是指针）会失效；没重新分配就不会
    2. 对于deque，插入除了首尾以外全失效；插入首尾后迭代器失效，指向deque内元素的引用和指针不会失效
    3. 对于list和forwarded_list，都不会失效

22. 添加元素操作也可能使容器失效（本质数据结构那一套，和添加对应，P315不再赘述）

23. 注意循环时每次都取`.end()`而不是保存，因为尾后迭代器很容易失效

24. vector的增长

    1. 前面也提到了vector的预分配空间是会增长的
    2. `c.capacity()`看预分配的空间大小
       1. `c.shrink_to_fit()`把capacity缩减到和size大小一致（不一定有效，只适用于vector, string, deque）
    3. `c.reserve(n)`预分配**至少**n个元素的空间

25. P321记录了string其他的拷贝方法，主要是对应拷贝char*

26. string特有的append（尾后插入）和replace（可调整被替换部分的大小）方法（P323）

27. string的搜索操作在P325，注意对大小写敏感

28. `compare`能够指定比较哪些部分的字符串，也能指定string和char*之类的比较（P327）

29. 容器适配器，这里特指`stack,queue,priority_queue`，也就是栈，队列和堆，已经刻进DNA了，这里不再赘述

## 第十章 泛型算法

算法我确实比较熟悉了，这里也并非C++ specific的内容，故而这部分笔记不会太多

1. lambda表达式（重点）

   1. 可理解为是一个未命名的inline函数

   2. 其形式：

      ```c++
      [capture list] (parameter list) -> return type {function body}
      ```

      *capture list* 可以捕获lambda所在函数中定义的局部变量，其余和普通函数类似

      注意lambda必须使用尾置返回

   3. lambda捕获值是在**创建时**拷贝而非使用时拷贝，可以通过指针或引用来使用变化的捕获值

   4. 可以通过=或&符号指定值/引用捕获

      e.g. `[=, &os]`和`[&, c]`，当混合使用隐式和显式捕获时，显式捕获（例中c和&os）必须采用和隐式捕获不同的方式

   5. 可变lambda：如果希望改变捕获的值，在parameter list后加上mutable关键字

2. 参数绑定bind（重点）

   1. 需要首先声明`using std::placeholders;`

   2. 形式为`auto newCallable = bind(oldCallable, arglist);`

   3. 比如下例：

      ```c++
      bool old_check(const string &s, string:size_type sz);
      // 该例中，new_check的sz已被绑定为42，形参的第一个会为s赋值
      auto new_check = bind(old_check, _1, 42);
      ```

      `_1`表示输入的第一个形参会为对应old func的形参赋值，`_2,_3`以此类推

3. 反向迭代器++会使迭代器向左移动（与普通迭代器相反），且`.rbegin()`返回指向最后一个元素的迭代器，而`.rend()`则返回指向**首前一个元素**的迭代器。可以通过反向迭代器直接等价于传入一个已经反向的容器

4. 可以使用`.rend().base()`来保证反向迭代器转化为正向迭代器，但是会指向`.rend()`相邻的元素

5. 迭代器类别（P366）

   1. 输入输出迭代器：只能单向输入/输出（只写/只读）
   2. 前向迭代器：可以同时输入/输出，但迭代器自身只能单向移动
   3. 双向迭代器：迭代器自身也能双向移动
   4. 随机访问迭代器：o(1)访问任意元素

6. 特定算法（forwarded_list & list）：

   1. `lst1.merge(lst2, compare)`，lst1和lst2都需要是有序的，用compare进行比较
   2. `lst.splice / flist.splice_after`（P370）

## 第十一章 关联容器

map和set，这一部分也属于最熟悉的数据结构了

1. map和multimap(关键字可重复)定义在头文件map中；set和multiset(关键字可重复)定义在头文件set中；无序容器(根据hash组织的)定义在头文件unorderer_map和unordered_set中

2. 对于有序类型必须提供比较的办法，且必须**严格弱序**：

   1. 两关键字不能同时*小于等于* 对方
   2. 大小比较具有传递性，即k1 <= k2且k2 <= k3可得k1 <= k3
   3. 两关键字都不小于等于另一个，则两关键字等价，且等价具有传递性

3. 比较函数的使用

   ```c++
   // 定义一个Sales_data对象上ISBN成员的严格弱序
   bool compareIsbn(const Sales_data &lhs, const Sales_data &rhs) {
       return lhs.isbn() < rhs.isbn();
   }
   // 定义一个bookstore对象按isbn大小排序存储Sales_data
   // decltype(compareIsbn)*说明我们要使用一个给定函数的指针
   // 用compareIsbn来初始化bookstore对象说明我们添加对象时，用compareIsbn来排序
   multiset<Sales_data, decltype(compareIsbn)*> bookstore(compareIsbn);
   ```

4. pair类型

   1. 定义在头文件utility中，保存两个数据成员，例如`pair<string, vector<int>> line;`
   2. 可以提供初始化器，例如``pair<string, string> author{"James", "Joyce"};`
   3. pair的成员是public的，分别使用`pair.first`和`pair.second`来调用
   4. 函数调用时可用列表初始化值`{"James", "Joyce"}`作为返回值（较早C++版本不行）
   5. 显式构造返回值：
      1. `return pair<string, int>("James", s.size());`
      2. 或`return make_pair("James", s.size());`

5. 关联容器操作

   1. `key_type`和`value_type`对于set是一样的，对于map不同

   2. `mapped_type`是map独有的，表示`pair.second`的类型，例子如下：

      ```c++
      set::string::value_type v1; // v1是一个string，key_type也是
      map<string, int>::key_type v2; // v2是一个string
      map<string, int>::mapped_type v3; // v3是一个int
      map<string, int>::value_type v4; // v4是一个pair<const string, int>
      ```

6. 关联容器的迭代器

   1. `auto map_iter = word_count.begin()`，这里map_iter指向一个`pair<const string, int>`对象的引用
   2.  set的迭代器是const的

7. 添加元素（P384），与之前的其他容器类似，使用insert和emplace操作，insert可根据返回值(bool，true为插入成功)确定是否插入成功

8. 删除操作（P387），使用erase操作

9. 元素的访问（P389）：

   1. `c.find(k)`返回指向第一个关键字为k的元素的迭代器
   2. `c.count(k)`返回关键字k的数量（不重复关键永远返回1或0）
   3. `c.lower_bound(k)`返回指向第一个关键字不小于k的元素的迭代器，即指向第一个关键字k的位置（找不到就和`c.upper_bound(k)`指向同一个）
   4. `c.upper_bound(k)`返回指向第一个关键字大于k的元素的迭代器，即指向最后一个关键字为k元素之后的一个元素位置
   5. `c.equal_range(k)`返回一个迭代器pair，指向关键字等于k的元素的迭代器范围，若k不存在，则pair的俩迭代器都等于`c.end()`

10. 无序容器管理桶：

    1. 每个桶保存零个或多个元素，容器将所有特定hash值的所有元素都保留在桶中（这点有别于java的红黑树）P395定义了对桶的一些管理操作

    2. 自定义的类型不能直接使用无序容器，需要提供自定义的hash模板，如下

       ```c++
       size_t hasher(const Sales_data &sd) {
           return hash<string>()(sd.isbn());
       }
       bool eqOp(const Sales_data &lhs, const Sales_data &rhs) {
           return lhs.isbn() == rhs.isbn();
       }
       // 定义一个unordered_multiset
       using SD_multiset = unorder_multiset<Sales_data, decltype(hasher)*, decltype(eqOp)*>;
       SD_multiset bookstore(42, hasher, eqOp);
       ```

       如果类（此例中Sales_data）定义了operater==，那么只需要重载hasher即可

## 第十二章