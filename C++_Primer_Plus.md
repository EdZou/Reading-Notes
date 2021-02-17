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

## 第十二章 动态内存

面经重点:)

1. 除了静态内存和栈内存，每个程序都拥有一个内存池，也就是堆(heap)/自由空间(free store)，程序用堆来动态分配内存，需要显式的销毁

2. 引入智能指针（都定义在memory头文件中）：

   1. `shared_ptr`：允许多个指针指向同一个对象
   2. `unique_ptr`："独占"所指向的对象
   3. `weak_ptr`: 弱引用，指向shared_ptr指向的类（java里也有）

3. `shared_ptr`详解：

   1. `shared_ptr<T> sp`，空智能指针，指向T类型的对象
   2. `p.get()`返回p保存的指针（shared，unique都支持）
   3. `swap(p, q)`交换指针的内容（而非指向的对象内容，shared，unique都支持）
   4. 如果智能指针指向的对象的引用计数（python的内存管理）归零，就释放这个对象
   5. `p.use_count()`返回p共享对象的智能指针数量，可能很慢
   6. `p.unique()`如果use_count为1返回true，否则false
   7. 防止内存泄漏，比如将shared_ptr放在容器中，之后不再需要，一定记得使用erase删除容器中的shared_ptr

4. `make_shared`函数

   1. 和emplace函数作用相近
   2. 用法`shared_ptr<int> p = make_shared<int>(42);`
   3. 如果不传递参数就会进行值初始化，如`shared_ptr<int> p = make_shared<int>();`

5. 使用动态内存三原因：

   1. 程序不知道自己用多少对象
   2. 程序不知道对象的准确类型
   3. 程序需要在多个对象中共享数据

6. 直接管理内存（不用智能指针，用new/delete）

   1. 因为free store分配的空间是无名的，new只能返回一个指向分配对象的指针，如`int *pi = new int;`

   2. 可使用构造，列表初始化`new string(10, '1')`或`new vector<int>{1,2,3}`

   3. 可用auto来自动推断分配类型，但只能括号中仅有单一初始化器时使用

      ```c++
      auto p1 = new auto(obj); // 正确，推断出obj同类型的指针
      auto p2 = new auto{1,2,3}; // 错误，只能为单一初始化器
      ```

   4. 可用new分配const对象，如`const int *pci = new const int(1024);`

   5. 内存耗尽处理

      ```c++
      int *p1 = new int; // 分配失败，返回空指针，new抛出std::bad_alloc
      // 这里顺带一提，malloc不会抛出异常，只能通过返回是否是空指针判断成功与否
      int *p2 = new (nothrow) int; // 不抛出异常，和malloc相似
      ```

   6. 一次new必须且只能对应一次delete操作，多次delete同一个new的指针是未定义的

   7. 如果函数A返回一个new生成的指针，而函数B用内置（普通）指针接收，那么即使B函数周期结束，因为不是智能指针，内置指针本身被释放，但指向的new出来的对象则不会（内存泄漏）解决方法：手动在B中delete内置指针

   8. 注意如果delete一个内置指针，很可能会有其他指针指向已经被释放的内存（空悬指针）

7. shared_ptr和new的结合

   1. 注意不能进行内置指针到智能指针的隐式转换，必须使用直接初始化，如下例

      ```c++
      shared_ptr<int> p1 = new int(1024); // 错误，不能隐式转换
      shared_ptr<int> p1(new int(1024)); // 正确，直接初始化
      ```

      `shared_ptr<T> p(q, d)` p接管内置指针q所指向的对象的所有权（之后不要用内置指针q访问了），并且将q置为空，p会调用d（可不重载）来代替delete。如果q也是一个shared_ptr，那么p就是q的一个拷贝，唯一不同就是d的不同

      当shared指针被销毁，被接管的普通指针也会失效

      `p.reset(q, d)`会首先放弃原本的引用（引用计数减一，0就自动回收），将指向对象改为q，相应的delete重载为d（可选择不重载）

   2. `p.get()`智能指针的get方法返回内置指针，这意味着，如果发生以下情况

      ```c++
      shared_ptr<int> p(new int(42));
      int *q = p.get();
      shared_ptr<int> p2(q); // p2和p相互独立，不共用引用计数！！！
      ```

8. 如果new之后delete之前发生了异常，不会释放内存导致内存泄露，使用shared_ptr等智能指针能在异常时也调用析构函数/delete

9. 注意不要使用同一个内置指针初始化多个智能指针，会使智能指针相互独立

10. 不delete get()返回的指针

11. 不用get()初始化/reset()另一个智能指针

12. `unique_ptr`详解

    1. unique_ptr独占，所以不支持普通的拷贝或赋值
    2. unique_ptr操作（P418）大体上和shared相近，注意`u.release()`放弃对指针的所有权，返回内置指针，并将u置空，可以通过`p2.reset(p3.release());`转移所有权。注意release返回的内置指针需要delete，不会自动回收
    3. 我们可以拷贝/赋值一个将要销毁的，最常见从函数返回一个unique_ptr
    4. 传递删除器`unique<connection, decltype(end_connection)*> p(&c, end_connection)`

13. `weak_ptr`详解P420

    1. 与shared_ptr指向相同对象
    2. 不参与生存周期管理，不增加引用计数
    3. 使用前需要调用`w.expired()`，如果shared的引用计数>0则true，否则false
    4. `w.lock()`如果引用计数大于0则返回一个shared_ptr，不然返空的shared

14. 动态数组

    1. 本质思想：将分配和初始化分离减小消耗

    2. 动态分配一个空数组合法：

       ```c++
       char arr[0]; // 不合法，不能定义长为0的数组
       char *cp = new char[0]; // 正确，但是cp不能解引用
       
       // 通过delete []释放动态数组
       delect [] cp;
       ```

    3. unique_ptr支持动态数组的管理

       ```c++
       unique_ptr<int[]> up(new int[10]);
       up.release(); // 自动调用delete[]
       ```

    4. 但shared_ptr不支持管理动态数组，需要提供删除器，如：`shared_ptr<int> sp(new int[10], [](int *p) { delete [] p; });`

    5. 注意shared_ptr不支持下标运算和算数运算，需要用`.get()`先得到内置指针再算术运算得到对象，如`*(sp.get() + i)`

    6. `alloctor`详解

       1. 定义在memory类中，是一个模板，分配的内存为原始未构造的

       2. 分配使用方法

          ```c++
          allocator<string> alloc;
          auto const p = alloc.allocate(n); // 分配了n个未初始化的string
          ```

       3. 构造使用方法

          ```c++
          alloc.construct(q++, 10, '1'); // *q为"1111111111"
          ```

       4. 用完对象利用destory执行对象的析构函数

          ```c++
          while (q != p) {
              alloc.destory(--q);
          }
          ```

       5. 最后释放内存

          ```c++
          alloc.deallocate(p, n);
          ```

    7. 拷贝和填充未初始化内存的标准库算法P429，`uninitialized_copy`一类，这里不多赘述

## 第十三章 拷贝控制

一个类通过五种特殊的成员函数控制对象的拷贝，移动和赋值时的操作，分别是拷贝构造函数(copy constructor)，拷贝赋值函数(copy-assignment operator)，移动构造函数(move constructor)，移动赋值函数(move-assignment operator)和析构函数(destructor)，这些操作组成了拷贝控制操作

1. 拷贝构造函数

   1. 第一个参数几乎总是一个const的引用，如 `Foo(const Foo&)`
   2. 默认的合成拷贝构造函数一般依次拷贝非static成员
   3. 拷贝初始化的例子：`string nines = string(10, '1');`
   4. insert和push都是拷贝初始化，emplace的元素都是直接初始化
   5. 函数调用时，非引用类型参数需要拷贝初始化，这也是为什么拷贝构造函数必须要const的引用，避免调用的无限循环（P442）

2. 拷贝赋值函数

   1. 重载operator=，例`Foo& operator=(const Foo&)`
   2. 与拷贝构造类似，右侧对象的每个非static成员依次拷贝

3. 析构函数

   1. 释放并销毁对象的非static成员，按初始化逆序销毁
   2. 智能指针会在析构自动销毁
   3. 离开作用域或调用delete时自动调用析构，注意容器被销毁时元素也会被销毁
   4. 空析构函数体意味着没有任何需要手动销毁，成员都可以自动销毁

4. 三/五法则

   1. 如果一个类需要一个析构函数，那么几乎可以肯定它也需要一个拷贝构造和赋值函数（三个基本操作）

      比如我们new一个成员，自然需要在析构函数中定义相应的delete操作，但如果使用默认的合成拷贝函数，会导致多个对象公用一个new的对象，析构时也会有多次delete(未定义操作)

   2. 需要拷贝操作的类也需要赋值操作

5. 可以用`=delete`来阻止拷贝/赋值，如`NoCopy(const NoCopy&) = delete;`。注意析构不能是=delete的，因为一旦删除就无法回收，编译器也不会允许这个类的变量定义和创建

6. 编译器合成的拷贝控制函数可能是=delete（P450）

   1. 析构是删除或不可访问（如private）
   2. 拷贝构造函数是删除或不可访问
   3. 拷贝赋值函数是删除或不可访问，或类有一个const的或引用成员
   4. 类内的某个成员满足以上条件

7. 行为像值的类：

   1. 先将目标拷贝进一个临时量
   2. delete掉原本的值
   3. 将临时量赋予左值
   4. 注意这是为了一个对象赋予自身时也能工作的必要步骤

8. 注意定义自己的swap函数时，应该在swap函数内用`using std::swap;`而不是直接用`std::swap(a.i, b.i);`因为如果类自身有定义，swap匹配时，自身版本会优于`std::swap`

9. 移动利用`std::move`，能够保证移后源仍保持一个有效可析构的状态，使用`std::move`还能省去中间生成临时量的步骤

10. 右值引用

    1. 必须绑定到右值的引用，通过&&来获取，例`int &&rr = 12*3;`
    2. 左右值区别：左值持久，右值短暂
    3. 变量都是左值，但是可以通过`std::move`将左值转换为右值

11. 移动构造函数和移动赋值运算符

    1. 第一个参数一定是该类类型的一个右值引用，任何其他的额外参数都必须有默认实参

    2. 移动构造函数必须保证移后的源对象销毁是无害的，完成移动后，源对象不再指向被移动的资源——这些资源的所有权已经是新的对象的

    3. 编译器会认为我们移动操作有异常，所以需要`noexcept`关键字

    4. 如果类定义了拷贝构造函数，拷贝赋值运算符，析构函数中的任意一个，编译器不会为它合成移动操作，只有当上述三个都没有自定义，且类内每个非static成员都可移动，编译器才为类合成移动操作

    5. 与拷贝不同，移动操作永远不会隐式定义为=delete函数，除非以下情况

       1. 定义=default，但有成员无法移动（比如类成员是const或是const的引用，或类成员定义了拷贝却没定义移动操作，类成员的移动操作=delete）

    6. 拷贝操作和移动操作有相互作用关系，如果类定义了自己的移动操作（任意一个）且未定义自己的拷贝操作，那么两个拷贝操作都被定义为=delete

    7. 如果没有合成的移动复制，则会利用已有的拷贝操作，过程：右值->拷贝成左值->拷贝操作

    8. 使用如下

       ```c++
       StrVec::StrVec(StrVec &&s) noexcept
           // noexcept位于成员初始化器前，参数列表后
           // 成员初始化会接管s中的资源
           : elements(s.elements), first_free(s.first_free), cap(s.cap) {
               // 保证s对其运行析构是安全的
               s.elements = s.first_free = s.cap = nullptr;
           }
       ```

       `noexcept`的存在也是为了保证不会出现新元素未完全拷贝而被赋值的旧元素已被部分改变的现象（类似MySQL的原子性）

       移动操作也需要能保证自赋值

12. 通常定义一个接收const的左值的拷贝构造函数和接受非const右值引用的移动构造函数

13. 引用限定符

    1. & 左值， && 右值
    2. 跟在const限定符后，例`Foo anotherMen() const &;`
    3. 注意引用限定符，要么同名且同参数函数全都要加上，要不全都不加

## 第十四章 重载运算与类型转换

1. 对于二元运算符，左值给第一个参数，右值给第二个

2. 运算符的本质是一次调用，重载时选用与内置类型一致的含义

3. 重载作为成员或非成员（P493）：

   1. `=, [],(),->`必须成员
   2. 复合赋值运算符(`+=, -=`等)一般是成员，但非必须
   3. 改变对象状态的运算符或与给定类型密切相关（递增，递减，解引用）一般是成员
   4. 具有对称性可能转换任意一端的对象（算数，相等性比较，关系，位运算）一般非成员

4. 输入/输出运算符

   1. 输出运算符减少格式化操作，让用户控制
   2. 必须是非成员函数

5. 算术运算符一般是在一个局部变量内，操作结束返回局部变量的副本，通常定了算数运算符也会定义对应的复合赋值

6. 很有可能不存在理想的关系运算

7. 最好同时定义下标运算符的常量和非常量版本

8. 区分前置和后置运算符

   ```c++
   StrBlobPtr operator++();
   StrBlobPtr operator++(int); // 通过形参确定后置版本，并不实际使用
   StrBlobPtr operator--();
   StrBlobPtr operator--(int);
   ```

9. 箭头运算符永远不能丢掉成员访问这个定义，只能改变从那个对象获取成员

10. 函数调用运算符

    ```c++
    struct absInt {
        int operator()(int val) const { // 定义方式和其他operator一样
            return val < 0 ? -val: val;
        }
     }
    ```

11. 注意lambda也是一个函数对象，编译器将表达式翻译成一个未命名类的未命名对象，如下例中

    ```c++
    stable_sort(..., 
                [sz](const string &a, const string &b)
                { return a.size() < b.size(); } );
    // 其行为类似于
    class ShorterString {
        public:
        	ShorterString(size_t n): sz(n) { } // 对应捕获的变量
        	bool operator()(const string &a, const string &b) const
                { return a.size() < b.size(); }
    }
    // 故而stable_sort也可以直接用这个类进行替换
    stable_sort(..., ShorterString()) // 等价于第一个lambda版本
    ```

12. P510罗列了一些标准库的函数对象，比如我们使用`vector<string *>`，并希望通过string*这么一个指针来完成排序，如果我们手写lambda去完成compare的工作会报错，因为不同对象的指针间的比较是未定义的，但是`less<string*>`在本例中可以作为良好定义的compare函数

13. 很多函数共享一种函数形式，如`int(int, int)`（形参两int，返回一个int），则可以定义一个函数表来存储指向这些同函数形式的指针，如：`map<string, int(*)(int, int)> binops;`。但注意只能存相同形式的函数指针，相同形式的lambda和类调用运算符都不行（本质都是类类型）

14. 函数类型可以使用function模板（P512），和上条的函数表结合，例：`map<string, function<int(int, int)>>`，这时相同形式的lambda和类调用运算符就可以了

15. 注意不能直接将重载的函数存入function类型的对象里，如下例

    ```c++
    int add(int i, int j) { return i + j; }
    Sales_data add(const Sales_data&, const Sales_data&);
    map<string, function<int(int, int)>> binops;
    binops.insert({"+", add}); // 错误，add有二义性
    
    // 解决方法一：存储指针
    int (*fp)(int, int) = add;
    binops.insert( {"+", fp} );
    // 解决方法二：运用lambda
    binops.insert({"+", [](int a, int b) {return a + b;} })
    ```

16. 类型转换运算符

    1. 类类型转换(class-type conversions)，也被称为用户定义的类型转换

    2. 形式 `operator type() const`

    3. 只要type能作为函数的返回类型就可以，所以不允许type为数组或函数类型，但允许转换为对应的指针/引用类型

    4. 没有显式的返回值也没有形参，必须是成员函数，一般是const

    5. C++11后可以用`explicit`关键字修饰，应用如下：

       ```c++
       class SmallInt {
           public:
           	SmallInt(int i = 0): val(i) {
                   if (i < 0 || o > 255) {
                       throw std::out_of_range("Bad SmallInt Value");
                   }
               }
           	// 编译器不会自动进行转换
           	explicit operator int() const {return val}
           private:
           	std:size_t val;
       }
       SmallInt si;
       si = 4; // 正确，将4隐式转为SmallInt，再调用SmallInt::operator=，构造函数不是explicit，所以可行
       
       si + 3; // 错误，意图将si隐式地转换为int，再执行加法，但类型转换为隐式，不满足explicit的要求
       
       static_cast<int>(si) + 3; // 正确，显式使用了类型转换
       
       ```

    6. 防止二义性类型转换

       1. 为两个类提供相同的类型转换（同时定义A->B和B->A）
       2. 多个转换规则，可以联系到一起（A->B->C），所以不要创建两个转换源/对象都是算术类型的类型转换

    7. 重载函数的二义性

       1. 比如两个类都定义了相同类型成员，那么隐式调用时编译器会不清楚应该隐式初始化哪个类，如下

          ```c++
          struct C {
              C(int);
              // 其他成员
          }
          
          struct D {
              D(int);
              // 其他成员
          }
          void manip(const C&);
          void manip(const D&);
          manip(10); // 二义性，不知道隐式使用C还是D
          manip(C(10)); // 显式调用正确
          
          // ====================
          // 即使是不同的算术类型，如下
          struct C {
              C(int);
              // 其他成员
          }
          
          struct E {
              E(double);
              // 其他成员
          }
          void manip(const C&);
          void manip(const E&);
          manip(10); // 也会产生二义性
          ```

    8. 成员和非成员的二义性

       假设我们使用 `a op b`这么一个方法，编译器不清楚如下版本使用哪一个

       ```c++
       a.op(b);  // a的成员函数op
       op(a, b); // 非成员函数op
       ```

## 第十五章 面向对象程序设计（OOP）

不多说，这章全是重点

1. *派生类（derived class）* 继承（inherit）*基类（base class）*

2. 对于一些函数，基类希望它的派生类各自定义合适的版本，就将其声明为*虚函数（virtual function）*，示例：

   ```c++
   class Quote {
       public:
       	std::string isbn() const;
       	virtual double new_price(std::size_t n) const;
   }	
   ```

3. 派生类一定要使用*类派生列表（class derivation list）*明确指出它继承了那些基类（C++支持多继承，不同于Java）。形式：先是一个冒号，随后跟着以逗号分隔的基类列表，如下：

   ```c++
   // 第二个public表示继承来的成员默认public，其他访问控制关键字类似
   public Bulk_quote : public Quote {
       public:
       	(virtual) double net_price(std::size_t n) const override;
       	// 有了后面加上的override关键字，表明了这是一个重新定义的虚函数
       	// 前面的virtual可加可不加
       	// 虚函数在派生类中也是隐式的虚函数
   }
   ```

4. 动态绑定，如`double net = item.net_price(n);`，此时`net_price`尚未被确定是哪一种版本，*运行时绑定（run-time binding）*。我认为这也算是C++不完全多态的一种体现

5. 派生类必须使用基类的构造函数来初始化它的基类部分，比如下例

   ```c++
   Bulk_quote(const std::string &book, double p, std::size_t qty, double disc):
   	Quote(book, p), min_qty(qty), discount(disc) { }
   ```

   除非我们像本例一样特别指出，否则基类部分会执行基类的默认初始化

6. 如果有一个静态成员，那么它在整个继承体系中都是唯一的

7. 派生类的声明不包含其派生类列表

   ```c++
   class Bulk_quote : public Quote;  // 错误，不应该包含派生列表
   class Bulk_quote;                 // 正确
   ```

8. 如果用某个类做基类，那么它必须已经被定义而不仅仅是声明（基类不能是不完全类型）

9. 防止继承的发生：

   1. 使用`final`关键字（与Java的final大相径庭）
   2. 例：`class Last final: Base {...}`，该例中Last不能再作为基类

10. 即使一个基类的指针或引用能绑定在派生类对象上，我们也不能执行从基类到派生类的转换

11. 可传递派生类给需要积累的类型的对象，但这样的话本质是派生类自己的部分被切掉（sliced down）只使用基类的部分

12. 虚函数

    1. 虚函数形参类型必须和被继承的一致，例外是当虚函数返回类型是类本身的指针或引用时
    2. 同样的`override`关键字必须针对虚函数，且形参必须一致，只有虚函数能被重写
    3. 也可以将函数指定成final，后续只要尝试覆盖都会报错
    4. 虚函数可以拥有默认实参
    5. 通过作用域运算符可以强行不要动态绑定，执行确定的版本，比如`baseP->Quote::net_price(42)`，称为回避动态绑定。一般用于重写的虚函数需要调用基类的虚函数版本时

13. 抽象基类

    1. 有纯虚函数成员的基类就是抽象基类
    2. 明确告诉用户这个纯虚函数没有实际意义，无需真的定义，只要使用`=0`即可，且只能出现在声明处，例：`double net_price(std::size_t) const = 0;`
    3. 也可以为纯虚函数提供定义，但必须在类的外部，类内不能为一个=0的纯虚函数提供函数体

14. 派生类的初始化函数只初始化它的直接基类，如下

    ```c++
    Bulk_quote(const std::string &book, double p, std::size_t qty, double disc):
    	Disc_quote(book, p, qty, disc) { }
    ```

    因为Disc_quote继承Quote，这样一来Bulk_quote的对象包含三个子对象，一个（空的）Bulk_quote部分，一个Disc_quote子对象以及Quote子对象。各个类分别控制其对象的初始化过程

15. `protected`访问控制关键字

    1. 和private一样，成员不受用户访问
    2. 和public一样，对于派生类成员和派生类的友元可访问，private的话两者都不行，只有基类自己的成员和友元可以访问
    3. 注意派生类的友元或派生类的派生类只能通过派生类对象来访问基类的protected成员，派生类无访问特权

16. 在前面的本章第三点中也提到了公有继承，如下例

    ```c++
    public Bulk_quote : public Quote { ... }
    ```

    公有继承就是继承来的成员一样是public，私有，受保护继承同理，后两种继承不影响派生类和其友元的访问，目的是为了控制派生类用户对于基类成员的访问。如下：

    ```c++
    Public_Derived d1;  // public继承基类
    Private_Derived d2; // private继承基类
    d1.public_member(); // 能访问基类的public成员，在派生类中也是public的
    d2.public_member(); // 不能访问基类的public成员，在派生类中时private的
    ```

17. 假定D继承了B：

    1. 只有当D public继承B时，用户代码才能完成从派生类到基类的转换
    2. 不管D以什么方式继承B，D的函数成员和友元都可以完成派生类到基类的转换，因为继承方式不影响他们的访问权限
    3. 当D以public或protected继承B，D的派生类的成员和友元可以完成派生类到基类的转换；private继承就不行

18. 友元关系不能继承（父亲/孩子的朋友不是我的朋友）

19. 可以用using改变访问权限（P546），如下

    ```c++
    class Base {
        public:
        	std::size_t size() const { return n; }
        protected:
        	std::size_t n;
    }
    
    class Derived: private Base { // 所有成员访问级别都为private
        public:
        	using Base::size;     // 访问级别又从private回到了public
        protected:
        	using Base::n;        // 访问级别又从protected回到了public
    }
    ```

    我是这么理解的，权限继承本质是把基类所有的低权限承员加入到派生类相应的权限部分，所以派生类自己不受影响但派生类的派生类受影响，而using又再将部分成员进行调整

20. 如果一个名字在派生类的作用域内无法解析，那么编译器会继续在外层的基类的作用域内寻找定义。即子类可以在父类的作用于里找，但反过来不行

21. 名字查找优先于类型查找，也就是说，如果派生类的成员和基类的成员形参不一致，基类成员会被隐藏，哪怕基类成员的形参更加匹配。因为编译器一旦找到符合的函数就不会再往外层作用域查找了

22. 所以如果派生类的虚函数与基类虚函数的形参不同，那么就无法通过基类的引用或指针找到并调用派生类定义的虚函数了（P550 例子）

23. 一条基类的using声明语句（不指定形参列表）可以把该函数所有的重载实例都添加到派生类作用域中

24. 虚析构函数

    1. 如果基类的析构函数不是虚函数，那么delete一个指向派生类对象的基类指针将时未定义的
    2. 阻止合成移动操作
    3. 虚析构函数并不需要相应的拷贝构造和赋值操作
    4. 大都基类都会定义一个虚析构函数，且一旦基类自己定义了移动操作，那么一定要显式定义拷贝操作

25. 之前提到过，派生类初始化时也要提供基类部分的初始化。因此派生类在拷贝和移动操作时，也要同时拷贝和移动基类的成员。但是派生类的析构函数只销毁派生类自己的成员（避免重复delete）

    ```c++
    // 构造
    D(const D& d): Base(d) /*D成员初始值*/ {/* ... */}
    // 赋值
    D &D::operator=(const D &rhs) {
        Base::operator=(rhs);
        // 其他操作
        return *this;
    }
    ```

    像上例中对基类成员拷贝，不然基类成员会执行默认初始化从而拷贝失败

26. 执行析构函数时，先从派生类删除，然后沿着继承关系向上删除（孙类->子类->父类），销毁了派生类而为销毁基类时，对象处于*未完成的状态*

27. 类不能继承默认的拷贝，移动函数，只能由编译器来为他们合成，但可以用using，如下例

    ```c++
    using Disc_quote::Disc_quote; // 继承Disc_quote的构造函数
    ```

    using不会改变访问级别，不能指定explicit和constexpr，不直接继承默认参数，而是有n个默认参数，就生成相应的n个构造函数，每个构造函数会省略一个带有默认实参的形参

28. 当往容器里直接存放对象时，如`vector<Quote>`，那么把派生类放进容器，其派生类独有部分会被裁剪掉。可以使用（智能）指针，如`vector<shared_ptr<Quote>>`

## 第十六章 模板和泛型编程

1. 函数模板

   1. 一个模板就是一个公式

   2. 例如

      ```c++
      template<typename T>
      int compare(const T &v1, const T &v2) {
          /* */
      }
      ```

      以关键字`template`开始，后跟一个*模板参数列表(template parameter list)*，即以逗号隔开的一个或多个模板参数列表

   3. 编译器会推断出模板参数为我们实例化一个特定版本的函数（容器皆是如此）

   4. 类型参数（一般是T，实际可为任意未知名字）可以用作返回类型，参数类型，变量声明和类型转换

   5. 定义多个模板参数，可以用typename也可用class，如`template<typename T, class U>`

   6. 也可以定义非类型参数（必须是常量表达式）

      ```c++
      template<unsigned N, unsigned M>
      int compare(const char (&p1)[N], const char (&p2)[N]) {...}
      ```

      绑定到N, M的实参就是常量表达式

   7. inline和constexpr关键字放在模板参数列表之后，返回类型之前，例`template<typename T> inline T min(const T&, const T&);`

   8. 编译器遇到一个模板定义并不会生成代码，当实例化一个模板时才会生成代码

2. 类模板（P584）

   1. 经典的例子就是容器的定义和实例化

   2. 类模板不是一个类型名，而是用来实例化类型，一个实例化的类型总是包含模板参数

   3. 定义在类模板外的函数必须以template开始，形式如下

      ```c++
      template <typename T>
      ret-type Blob<T>::member-name(param-list)
      ```

      Blob是类模板的名字

      即使是类模板的类外构造函数也遵循这个形式

   4. 当我们处于**类模板作用域**内时，编译器处理模板自身的引用可以自动找到匹配的实参，例

      ```c++
      BlobPtr& operator++();
      // BlobPtr<T>& operator++();
      ```

   5. 在类外定义成员时，一开始并不在类的作用域内，直到遇到类名才进入作用域，例

      ```c++
      template <typename T>
      BlobPtr<T> BlobPtr<T>::operator++(int) { // 这行后进入作用域
          BlobPtr ret = *this; // 这里可以省略匹配实参了 
          /* */
      }
      ```

   6. 如果友元也是模板，类可以授权给所有也可以授权给部分友元实例，

      1. 部分友元：此时友元关系限定在相同实例化的对象中，比如`a<T>`和`b<T>`是友元关系，那么`a<int>`的友元关系限定在`b<int>`而不会和`b<string>`构成友元关系

      2. 如果需要让友元关系给所有a的实例化，那么定义友元时，a和b需要用不同的模板参数，例如

         ```c++
         template <typename T> class C {
             friend class Pal1<T>; // 相同实例才是友元
             template <typename X> friend class Pal2; // 所有实例都是C的友元
             friend class Pal3; // 是C所有实例的友元
         }
         ```

   7. 可以令模板参数成为友元，如`friend T;`

   8. `typedef`可以引用一个实例但是不能引用模板，但是`using`可以

      ```c++
      template <typename T> using twin = pair<T, T>;
      template <typename T> using bookNo = pair<T, unsigned>;
      ```

   9. 类模板可以声明static成员（P590），如`static std::size_t ctr;`的情况下，静态成员与模板参数无关，所有模板参数一致的实例共用一个静态成员，简单来说就是int公用int的静态，string公用string

   10. 理所应当的，模板参数名不能重用

   11. 和函数参数一样，声明中的参数名不必和定义时的参数名一致

   12. 默认C++通过作用域运算符访问的是名字不是类型，如`T::size_type *p`，程序员会不清楚这是一个T的静态成员和p相乘还是在用模板参数定义一个p的变量。编译器会默认是前者，当我们需要用后者时，需要显式指出，如`typename T::size_type *p;`

   13. 默认实参，如`template <class T = int>`

   14. P595普通类的成员模板，当在类模板外定义一个成员模板时，必须同时为类模板和成员模板提供模板参数，类模板参数在前，成员自己的模板在后，如下所示

       ```c++
       template <typename T>
       template <typename It>
       	Blob<T>::Blob(It b, It e):
       	data(make_shared(std::vector<T>(b, e)))
           {}
       ```

   15. 相同模板可能出现在多个文件中，为了避免实例化开销就有了显式实例化

       ```c++
       extern template declaration;                  // 实例化声明
       template declaration;                         // 实例化定义
       
       extern template class Blob<string>;           // 声明
       template int compare(const int&, const int&); // 定义
       ```

       当编译器遇到extern声明，他不会在本文件生成实例化的代码，因为实例化声明为extern意味着程序其他位置必定存在该实例化的非extern（声明）定义，可能有多个extern，但只有一个定义

       extern必须出现在任何使用该实例化模板的代码前

   16. 一个类模板的实例化定义会实例化类模板内的所有成员，与单纯的模板函数不同，所以因为shared_ptr能运行时改变删除器，它不能直接保存，而需要运行时的跳转操作

3. 模板实参推断（P600）

   1. 顶层const被忽略，非const对象的引用或指针可以传递给const的引用或指针形参

   2. 如果形参不是引用类型，可以对数组或函数类型完成指针转换

   3. 显式模板实参：

      ```c++
      // 编译器无法推断T1，它未出现在函数参数列表中
      template <typename T1, typename T2, typename T3>
      T1 sum(T2, T3)
      ```

      因为无法推测，所以每次调用都需要给T1一个显示模板实参，如下：

      ```c++
      auto val = sum<long long>(i, j);
      ```

      显式模板实参按从左至右的顺序与对应的参数模板匹配，比如

      ```c++
      // 编译器无法推断T1，它未出现在函数参数列表中
      template <typename T3, typename T2, typename T1>
      T1 alter_sum(T2, T3)
      
      // 这种情况，T1属于最右模板，指定T1时一定要三个都指定，如
      auto val = alter_sum(long long, int, long)(i, j)
      ```

      显式指定的实参可以完成正常的类型转换（算术类型）

   4. 尾置返回

      ```c++
      template <typename It>
      auto fcn(It beg, It end) -> decltype(*beg) {
          /* ... */
          return *beg;
      }
      ```

   5. 可使用`remove_reference<decltype(*beg)>::type`脱去引用，P606介绍了一系列的引用操作函数

   6. 函数指针必须指向模板的实例

   7. 模板参数的引用正常使用底层const

   8. 引用折叠（重点）

      1. 当我们将一个左值传递给函数的右值引用参数，且此右值引用指向模板类型参数`T&&`时，编译器推定模板参数为左值引用类型
      2. `X& &, X&& &, X& &&`都被折叠成`X&`
      3. 类型`X&& &&`折叠成`X&&`
      4. 引用折叠意味着如果函数参数是一个指向模板类型参数的右值引用，如`T&&`，那么它可以被绑定到一个左值，且推定出的函数模板是一个左值引用，所以既可以传左值也可以传右值
      5. P610提供了一个因为是左值引用还是右值的二义性导致的错误案例

   9. `std::move`就灵活运用了引用折叠（P611），通过`T&&`作为形参，然后在通过`remove_reference<T>::type&&`保证不论是输入左值->T为左值引用，还是右值->T为右值，都能够最终返回右值

   10. 转发（保存类型和const），结论上使用`std::forward()`。书上同样和`std::move`使用`T&&`作为模板形参只能解决当传入参数为左值时，能够维持其左值/右值属性和底层const，但是一旦后续的函数为接受右值引用的函数时，就会发生不能使用左值初始化右值形参的问题

   11. `std::forward`必须使用显式模板实参来调用，forward返回显示实参类型的右值引用

       1. `forward<T> -> T&&`
       2. `forward<T&> -> T& && -> T&`

4. 重载模板时，编译器选择更加特例化的版本，非模板函数会比模板函数更加匹配

5. 对于重载函数，如果缺少声明，编译器会选择最接近的函数从而发生难以预知的错误

6. 参数包：

   1. 模板参数包/函数参数包：零个或多个模板/函数参数

   2. 一般用省略号指出，如下

      ```c++
      template <typename T, typename...Args> // Args为模板参数包
      void foo(const T &t, const Args& ... rset); // rset为函数参数包
      
      // 可以用sizeof对包的大小求值
      cout << sizeof...(Args) << endl; // 模板参数的数目
      cout << sizeof...(args) << endl; // 函数参数的数目
      ```

   3. 可利用递归调用一层层剥掉包里的参数

7. 模板特例化：

   ```c++
   template <>
   int compare(const char* const &p1, const char* const &p2) { /* ... */ }
   ```

   定义一个专门处理char*类型的版本

   其中空的<>意味着我们为所有的参数模板都提供了实参

   1. 特化版本本质是一个**实例**而非重载版本

   2. 为了特化一个版本，原模版的声明必须在作用域中，且使用任何模板实例代码前，特化版本的声明也必须在作用域中，如果没有声明，编译器会使用原模板再实例化一个新的实例，很难查找错误

8. 类模板特例化

   1. 必须在原模版定义所在的命名空间里特例化它

   2. 类模板可以部分特例化，只提供部分模板实参，本身也是一个模板，使用时还是要为位置的那个部分模板提供实参

   3. 也可以指特例化成员而不是整个类，如下：

      ```c++
      template<>             // 特例化一个模板
      void Foo<int>::Bar() { // 特例化Foo<int>的成员Bar（）
          // ...
      }
      
      Foo<string> fs; // 实例化
      fs.Bar();       // 实例化
      Foo<int> fi;    // 实例化
      fi.Bar()        // 特例化版本的Foo<int>::Bar()
      ```

## 第十八章

这一部分我忽略了18.1

### 命名空间

1. 当应用程序用到多个库时，难以避免会有命名冲突，继而造成*命名空间污染(namespace pollution)*。传统方法是设置很长的实体名避免冲突，但是过长的命名费时费力且繁琐，故而有了命名空间

2. 命名空间的定义包含两个部分：首先是关键字`namespace`，随后是命名空间的名字，之后是由花括号包含的声明和定义，只要是能出现在全局作用域中就能置于命名空间内

3. 每个命名空间都是一个作用域，与作用域类似，每个命名空间内的名字都是该空间的唯一实体

4. 命名空间可以不连续，这点和作用域不一样，而这个特性可以使得我们将几个独立的接口和实现文件组成一个命名空间（类似于我们定义类和函数的方式）。即一部分在头文件中声明（对应类内的声明），定义部分置于另外的源文件中（对应类外定义）

5. P697先在头文件中开辟`cpp_primer`命名空间，空间内声明接口，之后再在源文件中开辟**同名**命名空间，并在空间内定义头文件中声明的接口（注意不把include放在命名空间内部）

6. 只要命名空间里存在合适的声明语句，那么即使是再打开的命名空间也可以使用简写模式

7. 在命名空间外可以使用作用域运算符访问并定义内部的接口

8. 命名空间内声明了模板特例化，那么就可以在命名空间外进行定义

9. ***全局命名空间***：在全局作用域内定义的名字（所有类，函数及命名空间外定义的名字）。全局作用域内定义的名字被隐式地添加到全局命名空间中。作用域符可以作用于这些成员，但因为他们是隐式的，并没有名字，如`::member_name`

10. 嵌套的命名空间：

    1. 内层命名空间的声明将隐藏外层命名空间的同名成员
    2. 内层命名空间定义的名字只在内层有效，外层访问需要添加限定符，如`namesp::sub_namesp::example_class`

11. C++11引入了**内联命名空间（inline namespace）**：

    1. 虽然也是嵌套命名，但是inline namespace中的成员可以被外层命名空间直接使用

    2. 命名：`inline namespace example_ns`

    3. inline关键字必须出现在第一次出现的地方，后续再打开就可以不写inline了

    4. 假如内层命名空间定义在另一头文件中，那么外层命名空间可以通过include使用：

       ```c++
       // ====头文件sub_namesp1.h中====
       inline namespace sub_namesp1 {}
       
       // ====另一头文件中====
       namespace example_namesp {
           #include "sub_namesp1"
       }
       ```

12. 未命名的命名空间

    1. 定义方式：`namespace {}`，namespace后紧跟花括号
    2. 内部定义的变量具有静态生命周期，第一次使用前创建，创建后直到程序结束被销毁
    3. 可以在给定文件内不连续，但不能跨越多个文件
    4. 每个文件都可以定义自己的未命名空间，但互相不相关
    5. 如果头文件定义了未命名空间，那么定义的名字在include这个头文件的不同文件中对应不同实体
    6. 定义在未命名空间中的变量可以直接使用
    7. 未命名空间中定义的变量作用域和该空间所在的作用域相同，比如这个空间就在文件最外层，那么它内部的变量作用域是整个文件。注意未命名空间中的变量一定和它外层的变量要有所区别，不然产生二义性问题

13. 为了避免像`namespace::member`这样繁琐的使用方法，C++提供了以下便利的手段

    1. 命名空间别名：

       设定一个短得多的别名，如`namespace short = long_namespace_name;`。也能用来指向一个嵌套的空间，如`namespace Qlib = cpp_primer::QueryLib;`

    2. `using`声明

       1. 一个using语句一次只能引入命名空间的一个成员（注意是一个名字，如果有重载版本只能引入所有版本的重载函数）
       2. 作用域从using开始到using语句在的作用域结束
       3. 外层同名变量被隐藏（与using指示根本性的不同之一），如果同层作用域有形参列表不同的视为重载，相同的直接报错
       4. 在类的作用域中，using声明语句只能指向基类成员
       5. 效果类似为命名空间的成员，在当前空间创造一个别名
       6. 当using声明注入的函数和现作用域有同形参同名函数的版本时会报错

    3. `using`指示

       1. 形式：`using namespace name`
       2. 如果name不是已经定义好的名字会报错
       3. 使得name里**所有**成员名变得可见
       4. 引入的作用域远比using声明更复杂，本质是把命名空间内的成员**提升**到->a. 包含命名空间本身的作用域；b. using所在的**向外一层**作用域
       5. 比如在函数f内使用`using namespace A`，那么在f看来，好比是在这个语句的地方将A中所有名字注入到和函数f所在的作用域中一样（空间A和f函数内部的空间不一样，A在外）
       6. 所以如果函数内定义了同名变量，会把using指示注入的变量隐藏掉
       7. 如果头文件中的顶层作用域含有using指示/声明，那么所有include这个头文件的文件都会被注入。所以一般都在头文件内的函数或namespace中使用using
       8. 与声明不同，与现作用域有同形参同名函数的版本不会报错，只要指明调用哪个版本避免二义性即可

    4. 所以using指示风险很大：

       1. 容易产生全局命名空间污染
       2. 且二义性错误只有在使用冲突名字才能发现，难以定位
       3. 更推荐对每个成员使用using声明

14. 对命名空间内部名字的查找也遵循常规的查找规则（从内向外，只考虑使用点前开放块中声明的名字）。在命名空间中的类中查找遵循该规则

15. 上述规则有一个重要例外，那就是给函数传递一个类类型参数（类的对象，引用，指针）给函数时，除了上述查找，都没找到时还会查找实参类**所属的空间**并将内部的成员注入

16. 如果应用程序定义了一个标准库里的已有名字

    1. 要么进入重载规则，确定调用的函数版本
    2. 要么压根不会执行标准库的版本
    3. 这就是为什么`std::move,std::forward`一般需要带上std限定，因为他们时常会无意中被重名定义

17. 当类声明友元时，友元本身并不可见，但如果一个未声明的类或函数第一次出现在友元的声明中，则认为友元是最近外层命名空间的成员，如下

    ```c++
    namespace A {
        class C {
            // 只有两友元声明，除此之外再无声明
            // 隐式成为命名空间A的成员
            friend void f2();        // 如果只在这里有声明是找不到的
            friend void f(const c&); // 根据上述查找规则可以被找到
        }
    }
    
    int main() {
        A::C cobj;
        f(cobj);  // 成功，通过A::C找到A::f
        f2();     // 错误，A::f2未被声明
    }
    ```

    上例中，f的声明中接受了类类型实参，且在C的命名空间有隐式地声明，所以可以被找到。

    这里的逻辑是

    1. main先找f，在当前作用域找不到
    2. f有实参是类C，于是进入类C所属的空间A查找
    3. 又因为f声明的形参为C，是一个未声明的类，所以f也隐式地存在于空间A中，进而被找到

### 多重继承和虚继承

1. 多重继承是指从多个直接基类中产生派生类的能力（Java不支持）

2. 形式如`class Panda: public Bear, public Endangered {}`，每个基类都有一个**可选**的访问说明符，如果被忽略默认为private，struct继承默认是public

3. 和之前讲的继承一样，继承的基类：

   1. 不能有重复
   2. 不能是final
   3. 必须之前就已经被定义过
   4. 继承的基类的个数没有限制

4. 多继承时，派生类初始化所有基类，但只能显式初始化它的直接基类

5. 基类的实际**构造顺序**和**派生类列表**中的顺序一致，与初始化值列表的顺序无关

6. C++11允许派生类从**一个或多个基类中**继承构造函数，但如果从多个基类中继承了**形参列表完全相同**的构造函数，会报错

   解决方法：为冲突的构造函数定义派生类自己的版本

7. 派生类定义拷贝移动等操作时

   1. 必须在完整的对象上完成拷贝移动操作
   2. 使用合成版本时，每个基类隐式地使用对应成员完成操作
   3. 使用合成版本时，先递归调用基类的相应操作直至顶部基类，顶部基类完成操作之后再弹栈(底层的栈活动)一路调用回到派生类自己的拷贝/赋值操作

8. 多重继承时，派生类的指针/引用仍然能自动转化为可访问基类的指针/引用，编译器**不会**对多个基类的类型进行比较和选择，如果出现了多个基类的重载版本调用，会直接报错

9. 当使用基类指针引用多继承派生类时，不属于这个基类类型的部分被slice down

10. 只有一个基类时，派生类嵌套在基类的作用域中，沿继承方向找名字；多继承时会**同时**找所有的直接继承基类，如果多个基类有同名就会发生错误：

    1. 定义本身不会出错，只要不调用二义性函数不会出错
    2. 即使同名不同形参也可能出错
    3. 即使一个是private无法访问，另一个是public/protected可访问也会报错
    4. 解决方法：要么加限定符，要么派生类自己定义一个

11. **虚继承**

    1. 尽管派生列表基类不能重复，但派生类可以多次继承同一个类（比如多继承了祖父类的两个子类，再比如继承父类+继承子类，例子：`base_ios`）

    2. 如果基类在继承链上出现了多次，派生类会有多个该类的子对象。可通过**虚继承**，表示愿意共享基类，共享的基类子对象就被称为**虚基类**，无论被继承多少次，虚基类都只有唯一一个共享的子对象

    3. （P718继承关系图）虚继承需要在虚派生类这个需求出现前就在基类里完成了虚继承的操作

    4. 使用方法：

       ```c++
       // public和virtual的顺序随意
       class Raccoon: public virtual ZooAnimal {};
       class Bear: virtual public ZoonAnimal {};
       ```

    5. 判断可见性原则：继承体系中，派生的类里定义的成员优先级比共享基类的高（P719）

    6. 虚继承的构造函数：

       1. 非虚继承时，会有多个子对象被初始化
       2. 虚继承时，先从继承的**顶端基类开始**初始化，然后沿继承路径向下初始化
       3. 如果底层派生类没有显式在初始化列表中使用顶层基类的初始化，顶层基类执行默认构造函数，如果这时顶层基类不存在默认构造函数就会报错

    7. 当有多个虚基类时，这些虚的子对象同样按照**派生列表**出现的顺序构造。具体过程：编译器按照直接基类的顺序依次检查，如果没有虚基类就类似先添加到队列中，有虚基类先初始化虚基类，检查结束后再对队列中的基类初始化。

    8. 注意析构销毁的顺序依然与构造顺序完全相反

## 第十九章

#### 控制内存分配

1. new操作的本质

   1. 调用一个`operator new / operator new[]`的标准库函数，该函数分配一块**足够大，原始的，未命名的内存空间**
   2. 编译器完成相应的构造函数构造这些对象并传入初始值
   3. 分配空间并构造完成，返回对应的指针

2. delete操作的本质与之对应：

   1. 将指针所指的对象中的元素执行对应析构函数
   2. 调用`operator delete / operator delete[]`标准库函数释放内存

3. 以上两操作都可以自定义

   1. 可以定义成成员函数也可以定义在全局作用域里
   2. 如果执行操作的类类型，编译器会首先在**类和基类**的作用域里找，找不到再去全局作用域里找
   3. 找得到就用找到的版本，找不到就用标准库的版本
   4. 可以用作用域运算符指定编译器往全局作用域找

4. 不抛出`bad_alloc`异常的接口定义

   1. 形式

      ```c++
      void *operator new(size_t, nothrow_t&) noexcept;
      void *operator new[](size_t, nothrow_t&) noexcept;
      void *operator delete(void*, nothrow_t&) noexcept;
      void *operator delete[](void*, nothrow_t&) noexcept;
      ```

   2. 类型`nothrow_t`是定义在new头文件里的一个struct，这个类型不包含任何成员

   3. new的头文件中定义了一个名为`nothrow`的const对象，用户通过这个对象申请nothrow版本。与析构函数类似，delete也不允许抛出异常，所以用noexcept

   4. 用户可在全局作用域里自定义上述任何一个，在类内上述函数是**隐式静态**的，因为new和delete不能操控类的任何成员，static关键字可加可不加

   5. 这俩函数返回值必定是void*，且`size_t`不能有默认值，new时分配对象所需的字节数量，new[]传入存储数组元素所需的空间

   6. 用户可以重载new/new[]，但是这个形参的版本：

      ```c++
      void *operator new(size_t, void*);
      ```

      **无论如何不能**被用户重载

5. （P729）使用定位new来构造对象：

   1. `new (place_address) type [size] {}`等形式
   2. `place_address`必须是一个指针
   3. 当通过地址值调用new，定位operator使用`void *operator new(size_t, void*);`在一个特定的，预先分配的内存地址上分配对象
   4. 与`construct`类似，但是`construct`必须要在`allocator`的内存上分配，但是传入定位new的指针不需要指向operator new分配的内存，甚至不要是动态内存

6. 学到这里很明显了，析构函数只清除给定对象但是不会释放空间（还得靠delete）

7. 运行时类型识别（run-time type idenfication， RTTI）由两个运算符实现：

   1. `typeid`，用于返回表达式类型
   2. `dynamic_cast`，用于将基类的引用或者指针安全地转换为派生类的指针或引用
   3. 当上述运算符被用于某类型的指针或引用时，且被使用的类型含有虚函数，运算符将使用指针或引用所绑定对象的动态类型
   4. 适用于：我们想使用**基类对象的指针或引用**执行某个**派生类操作**并且该操作**不是虚函数**。当操作被定义为虚函数时，编译器将根据对象的动态类型自动地选择正确的函数版本
   5. 可能的话定义虚函数而不是使用RTTI

8. `dynamic_cast`详解

   1. 三种形式：
      1. `dynamic_cast<type*>(e)` ，e必须是一个有效指针
      2. `dynamic_cast<type&>(e)` ，e必须是一个左值
      3. `dynamic_cast<type&&>(e)` ，e不能是左值
   2. 三种形式type都必须是一个类类型，且通常含有虚函数
   3. 三种形式中，e必须是：type的public派生类，public基类或者干脆就是目标type的类型。不符合以上任意一种会转换失败
   4. 转换失败时，如果目标是指针类型，返回0；如果目标是引用类型，抛出`bad_cast`异常
   5. 对空指针执行dynamic_cast，结果是所需类型的空指针

9. `typeid`详解

   1. 形式`typeid(e)`，其中e是任意表达式或类型的名字，e的顶层const被忽略

   2. 返回一个常量对象（type_info）的引用

   3. 如果e是引用，返回所引对象的类型

   4. 当作用于数组或函数时，返回的是数组或函数类型而不是自动转换成对应的指针类型

   5. 当e不属于类类型或类内未定义虚函数时，typeid直接返回一个运算对象的静态类型；当类类型至少定义了一个虚函数时，typeid的结果直到运行时才会获得

   6. 用于如下情况：

      ```c++
      // 两指针本质指向同一个对象
      Derived *dp = new Derived;
      Base *bp = dp;
      
      // 应用typeid
      // 如果两指针指向同一类型
      if (typeid(*bp) == typeid(*dp)) {}
      // 如果指针指向某种确定的类型
      if (typeid(*bp) == typeid(Derived)) {}
      ```

      注意因为typeid指向对象，需要使用`*bp`取出对象，如果作用于指针，返回的是该指针静态编译时的类型

10. （P734）说明了如何使用RTTI，比如用以定义一个可以对所有继承体系的类进行比较的equal虚函数，在派生类的equal里先将被比较的类（不管是该类的基类，派生类或自身都不会出错）转换为该类类型再进行比较

11. （P735）typeinfo的操作

12. 枚举类型：

    1. C++定义了限定作用域的枚举类型`enum class enum_name {a, b, c};`

    2. 不限定作用域的形式（省略class关键字）：`enum enum_name {a, b, c};`

    3. 如果enum未命名，只能在定义enum的同时定义它相应的对象

    4. 限定枚举成员的名字遵循常规作用域原则，在作用域外不可访问；不限定的enum成员和enum语句处于一个作用域

    5. 如果有两个枚举类型其成员内容和限定都一致，即使名字不同，还是重复，如下：

       ```c++
       enum color { red, yellow, green };
       enum color2 { red, yellow, green }; // 报错，重复定义
       enum class color3 { red = 1, yellow = 2, green = 1 }; // 正确，枚举的成员被隐藏
       
       color eye = green; // 正确
       color3 eye3 = green; // 错误，找到color的枚举类型，不匹配
       color3 eye3 = color3::green // 限定作用域后正确匹配
       ```

       从color3可见，枚举值可以重复，但初始化时提供的初始值必须是constexpr。同样的也可以在任何需要constexpr的地方使用枚举成员

       默认枚举值从0开始，从左至右依次加1

       可以将enum作为switch的表达式，而枚举值作为case分支

       因为是constexpr，还可以作为非类型模板的形参（P580例子），或在类的定义里初始化枚举类型的静态数据成员

    6. 不限定作用域的enum对象或成员会自动转为整型：

       ```c++
       int i = color::red; // 正确，不限定作用域隐式转化为整型
       int j = color3::red; // 错误，限定enum不会隐式转换
       ```

    7. C++11之前，成员都是由某种整型表示的，之后可以定义想使用的类型，如下

       ```c++
       enum intValues : unsigned long long {/* ... */}
       ```

       对于限定作用域的enum，不指定enum的潜在类型，默认成员类型为int

       不限定作用于的enum没有默认类型，只要潜在类型够大能容纳枚举值即可

    8. C++11后前置声明enum必须指定成员大小（如上），限定作用域可以不指定，默认int。前置声明后，之后的定义必须和前置声明的成员大小相同，不然报错

    9. 即使整型值恰好和enum的成员同类型，也不能作为enum的实参使用

    10. 但可以将非限制作用域的enum成员传给整型实参，此时，enum的成员类型会被**提升成int或者更大（取决于枚举值的潜在类型，即大小）**

13. 成员指针

    1. 是可以指向类的**非静态成员**的指针，指向类的成员而非对象。注意静态成员不属于任何类，所以指向静态成员的指针和普通指针没有区别
    2. 形式如`const string Screen::*pdata`，需要指出`classname::`。因为const指针指向的对象可以是const也可以不是，也就是pdata可以指向任何Screen对象的任何一个string成员
    3. 也可以不定义返回类型（或成员变量类型）直接用auto，如：`auto pdata = &Screen::contents`
    4. 成员指针受访问控制符限制，所以如果需要函数返回指针，可以在类内定义一个static的成员，返回一个成员指针(P741)，如`const string Screen::*pdata = Screen::data();`
    5. 使用时需要把它绑在对象上：`auto s = myScreen::*pdata`

14. 成员函数指针：

    1. 与成员指针类似，但如果函数由重载版本必须指明，如下：

       ```c++
       char (Screen::*pmf2)(Screen::pos, Screen::pos) const;
       pmf2 = &Screen::get;
       ```

       没有第一对括号，这个定义`char Screen::*pmf2(Screen::pos, Screen::pos) const;`会被视为定义一个名为pmf成员函数，并返回Screen类的一个char成员。因为声明的是普通函数不能使用const限定符，会被认为无效

       注意成员函数与指向它的成员函数指针间没有自动转换规则

       ```c++
       pmf = &Screen::get;
       pmf = Screen::get; // 错误，没有自动转换规则，要显式取地址
       ```

    2. 使用方法

       ```c++
       Screen myScreen, *pScreen = myScreen;
       char c1 = (pScreen -> *pmf)();
       char c2 = (myScreen.*pmf)(0, 0);
       ```

       两种皆可

    3. 使用using或者typedef会让代码更容易理解，如下

       ```c++
       using Action = char (Screen::*)(Screen::pos, Screen::pos) const;
       Action get = &Screen::get;
       ```

       P744有使用函数表和别名相结合的例子

    4. 成员函数指针因为未绑定到特定对象上（需利用`.*`或`->*`），不能直接传递给特定的算法，解决方法如下

       1. function，形如`function<bool (const string&)> fcn = &string::empty;`

       2. mem_fn（头文件functional中），function我们给定了成员的调用形式，mem_fn让编译器推定成员类型，可根据成员指针直接推：

          ```c++
          find_if(svec.begin(), svec.end(), mem_fn(&string::empty));
          
          // mem_fn生成的对象既可以通过指针调用也可以通过对象的引用调用
          auto f = mem_fn(&string::empty);
          f(*svec.begin());
          f(&svec[0]);
          ```

          可以认为mem_fn生成的callable object有一对重载符，一个接受string&一个接受string*

       3. bind，**必须**把函数中用于表达执行对象的隐式形参显式，如下

          ```c++
          // 实参和mem_fn一样，可以是指针也可以是引用
          auto f = bind(&string::empty, _1);
          f(*svec.begin()); // 实参为对象，传引用
          f(&svec[0]);      // 实参为指针  
          ```

15. 嵌套类

    1. 外层和内层的类相互之间访问没有特殊权限

    2. 和成员一样，嵌套类必须声明在外层类内部，但是定义在内外皆可，在外定义时需要以外层类的名字限定

    3. 嵌套类虽然没有访问特权，但是可以随意访问外层类的成员，不需要限定

    4. 构造函数也不例外，如下

       ```c++
       TextQuery::QueryResult::QueryResult (parameter_list)
           : initialization list { function body }
       ```

       第一个限定符表示`QueryResult`是`TextQuery`的嵌套类，第二个表示这是构造函数

    5. 接上例，如果在`QueryResult`（嵌套类）中定义了静态成员，那么这个成员位于`TextQuery`（外层类）的作用域之外

    6. 嵌套类在名字查找时遵循一般规则，但必须查找外层类的作用域

16. `union`：

    1. 本质是一个所有数据成员共用基地址的一种数据结构（用于找大端小端）
    2. 任意时刻只能有一个数据成员有效，当一个数据成员赋值后，其他成员便成**未定义**
    3. 分配给union对象的存储空间的大小至少要能容纳它最大的数据成员
    4. 含有构造函数和析构函数的类类型也可以是union的成员，默认union的成员都是public，也可以用访问控制修饰，这点和struct一样
    5. union不能作为基类也不能继承，所以不能含有虚函数
    6. 匿名union在其所在的作用域内该union的成员都可以直接访问
    7. 当union保存的是简单的内置成员值时，我们可以用普通赋值语句改变union的值，但是当成员为含有构造函数和析构函数的类类型时：
       1. 当我们要把union的类型改成类类型时，需要调用其构造函数
       2. 当我们要把union的类类型改成其他类型时，需要调用其析构函数
       3. 而且如果类的拷贝/赋值等操作定义为=delete，那么union的相应操作也会=delete
    8. 为了追踪union中的值类型，我们通常定义一个独立的对象称为union的判别式(discriminant)，P752定义一个enum来判定

17. 类定义在函数内部称为局部类

    1. 局部类所有成员都必须完整定义在类的内部
    2. 不允许声明静态成员
    3. 对外层访问受限，只能访问外层的类型名，静态变量和enum成员，不能访问外层作用于的局部变量
    4. 局部类内的嵌套类也必须遵循局部类的限制

18. P756位域Bitfield，嵌入式常见，限定bit的位数，必须是整形或者enum类，如下：

    ```c++
    typedef unsigned int Bit;
    class File {
        Bit mode: 2; // mode占2位
        Bit modified: 1; 
        /*
        ...
        */
    }
    ```

    取地址符&不能作用域位域，也没有指针能指向类的位域

19. `volatile`

    1. 表示它的值由程序直接控制之外的过程控制
    2. 比如多线程，比如时钟，因为这些值会被频繁更改，但是编译器的优化会将某一瞬间的值存入寄存器加快速度，volatile就是告诉编译器不要进行这样的优化
    3. 只能将volatile对象的地址赋给指向volatile的指针，只有引用是volatile时才能用他初始化volatile对象
    4. 不能用合成的拷贝/移动操作对volatile对象操作，因为合成的成员接受普通形参而非volatile形参

20. 链接指示: `extern "C"`

    1. C++使用链接指示指出任意非C++函数所用的语言
    2. 不能出现在类/函数定义的内部
    3. extern "C" 表示对C语言的支持，extern ”FORTRAN“表示对fortran语言的支持，依此类推
    4. 当include放在extern指示的部分，表示该头文件声明的所有函数都是用另一语言编写的
    5. 指向其他语言函数的指针：`extern "C" void (*pf)(int)`，其他语言的函数指针不能和C++的函数指针互相指，会报错
    6. 也可以把C++的函数导出给C语言（一般不用，C无法理解构造函数，析构函数这类OOP操作）