---
layout: post
title: c++ primer
date: 2021-02-23 11:29:05
---
# 3章 字符串，向量和数组
1. 字符串和字面值的拼接
  ```c++
   string s1 = "hello", s2 = "world";
   string s4 = s1 + ",";	// 正确 
   string s5 = "hello" + ", ";	// 错误：不能把字面值直接相加
   string s6 = s1 + ", " + "world";	// 正确
   string s7 = "hello" + ", " + s2;	// 错误：不能把字面值直接相加
  ```
2. c语言头文件如name.h，c++则将这些文件命名为cname，去掉h增加c，如cstring。

3. `for (declaration : expression)`这种称为范围for语句（range for）。

4. 既有类模板，也有函数模板，vector是一个类模板，模板不是类或函数，可以将模板看作为编译器生成类或函数编写的一份说明，编译器根据模板创建类或者函数的过程称为实例化，当使用模板时，需要指出编译器应把类或函数实例化成何种类型，在模板名字后面跟一对尖括号，在括号内放上信息。

5. 某些编译器可能仍需以老式的声明语句来处理元素为vector的vector对象，如       `vector<vector<int> >`

6. end()返回的迭代器称为尾后迭代器，该迭代器指示的是容器的一个本不存在的“尾后（off the end）”元素。

7. 标准容器迭代器的运算符
  ```c++
  *iter：返回迭代器iter所指元素的引用
  iter->mem：解引用iter并获取该元素的名为mem的成员，等价于(*iter).mem
  ++iter：令iter指示容器中的下一个元素
  --iter：令iter指示容器中的上一个元素
  iter1 == iter2
  iter1 != iter2
  ```

8. 迭代器类型
  ```c++
  vector<int>::iterator it1 = vec.begin();
  string::iterator it2 = s.begin();
  vector<int>::const_iterator it3 = vec.cbegin();
  string::const_iterator it4 = s.cbegin();
  ```
9. 数组
  ```c++
  unsigned cnt = 42;
  constexpr unsigned sz = 42;
  int arr[10];
  int *parr[sz];	// 正确，sz是常量表达式
  string bad[cnt];	// 错误，cnt不是常量表达式
  string strs[get_size()];	// 当get_size是constexpr时正确，否则错误
  ```
# 4章 表达式
1. 左值右值，简单理解为：左值指向一个具体内存地址，只要不是左值既是右值。具体含义参考`https://www.jianshu.com/p/94b0221f64a5`
* 左值引用只能左值去赋值；
* 常量左值引用除了左值赋值，也可以字面值赋值也就是右值可以赋值。
2. 位运算符
* 一般来说，如果运算对象是“小整型”，则它的值会被自动提升成较大的整数类型；
* 左移运算符在右侧插入值为0的二进制位；
* 右移运算符的行为依赖其左侧运算对象类型，如果运算对象是无符号类型，在左侧插入值为0的二进制位，如果该运算对象是带符号类型，在左侧插入符号位的副本或值为0的二进制位，如何选择要视具体环境而定；
* 位异或运算符，如果两个运算对象的对应位置有且只有一个为1则运算结果中该位为1，否则为0。
3. sizeof是运算符不是函数。
4. 类型转换之显示转换
* static_cast，dynamic_cast，const_cast，reinterpret_cast

# 5章 语句
1. <stdexcept>定义的异常类
* exception：最常见的问题
* runtime_error：只有在运行时才能检测出的问题
* range_error：运行时错误，生成的结果超出了有意义的值域范围
* overflow_error：运行时错误，计算上溢
* underflow_error：运行时错误，计算下溢
* logic_error：程序逻辑错误
* domain_error：逻辑错误，参数对应的结果值不存在
* invalid_argument：逻辑错误，无效参数
* length_error：逻辑错误，试图创建一个超出该类型最大长度的对象
* out_of_range：逻辑错误，使用一个超出有效范围的值

# 6章 函数
1. 可变长参数
* initializer_list<T>：这是一种标准库类型，跟vector一样是模板类型，跟vector不一样的是元素永远是常量值；
* 如果想向initializer_list形参中传递一个值的序列，则必须把序列放在一对花括号内；
2. 返回数组指针 
* 参考代码如下
  ```c++
  #include <iostream>
  
  /* 1 */
  int t1[3] = { 1, 2, 3 };
  int(*func1())[3]
  {
  	return &t1;
  }
  
  /* 2 */
  typedef int arrT[3];
  arrT t2 = { 10, 20, 30 };
  arrT* func2()
  {
  	return &t2;
  }
  
  /* 3 尾置返回类型 */
  int t3[3] = { 100, 200, 300 };
  auto func3() -> int (*)[3]
  {
  	return &t3;
  }
  
  /* 4 */
  int t4[3] = { 1000, 2000, 3000 };
  decltype(t4)* func4()
  {
  	return &t4;
  }
  
  /* main */
  int main()
  {
  	int (*t1)[3] = func1();
  	for (int i = 0; i < sizeof(*t1) / sizeof(int); i++)
  	{
  		std::cout << (*t1)[i] << std::endl;
  	}
  
  	int (*t2)[3] = func2();
  	for (int i = 0; i < sizeof(*t2) / sizeof(int); i++)
  	{
  		std::cout << (*t2)[i] << std::endl;
  	}
  
  	int(*t3)[3] = func3();
  	for (int i = 0; i < sizeof(*t3) / sizeof(int); i++)
  	{
  		std::cout << (*t3)[i] << std::endl;
  	}
  
  	int(*t4)[3] = func4();
  	for (int i = 0; i < sizeof(*t4) / sizeof(int); i++)
  	{
  		std::cout << (*t4)[i] << std::endl;
  	}
  	return 0;
  }
  ```
3. 函数特性
* 一次函数调用其实包含一系列工作：调用前要先保存寄存器，并在返回时恢复；可能需要拷贝实参；程序转向一个新的位置继续执行；
* 一般来说，内联机制用于优化规模较小、流程直接、频繁调用的函数，很多编译器都不支持内联递归函数，而且一个75行的函数也不大可能在调用点内联展开；
* constexpr函数是指用于常量表达式的函数，有几个约定：函数的返回类型及所有形参的类型都得是字面值类型，而且函数体中必须有且只有一条return语句（函数体内可以包含其他语句，只要这些语句在运行时不执行任何操作就行）；
* 为了能把constexpr函数在编译过程中展开，它被隐式地指定为内联函数；
* constexpr函数返回的不一定是常量表达式；
* 和其他函数不一样，内联函数和constexpr函数可以在程序中多次定义。毕竟，编译器要想展开函数只有声明是不够的，还需要函数的定义。不过，对于某个给定的内联函数或constexpr函数来说，它的多个定义必须保持一致。基于这个原因，内联函数和constexpr函数通常定义在头文件中；
4. 调试
* assert(expr)是预处理宏，首先对expr求值，如果表达式为假（即0），assert输出信息并终止程序的执行，如果表达式为真（即非0），assert什么也不做；
* assert的行为依赖于一个名为NDEBUG的预处理变量的状态，如果定义了NDEBUG，则assert什么也不做，默认状态是没有定义NDEBUG的；
5. 指向函数的指针
* 当把一个函数名作为一个值使用时，该函数自动转换成指针；
  ```c++
  pf = lengthCompare;	// pf指向名为lengthCompare的函数
  pf = &lengthCompare;	// 等价的赋值语句；取地址符是可选的
  ```
* 我们还能直接使用指向函数的指针调用该函数，无需提前解引用指针；
  ```c++
  bool b1 = pf("hello", "hi");	// 调用lengthCompare函数
  bool b2 = (*pf)("hello", "hi");	// 一个等价的调用
  bool b3 = lengthCompare("hello", "hi");	// 另一个等价的调用
  ```
* 函数指针形参；
  ```c++
  void useBigger(const string &s1, const string &s2, bool pf(const string &, const string &));	// 第三个形参是函数类型，它会自动地转换成指向函数的指针
  void useBigger(const string &s1, const string &s2, bool (*pf)(const string &, const string &));	// 等价的声明：显示地将形参定义成指向函数的指针
  useBigger(s1, s2, lengthCompare);	// 自动将函数lengthCompare转换成指向该函数的指针
  ```
* 直接使用函数指针类型显得冗长而繁琐；
  ```c++
  // Func和Func2是函数类型
  typedef bool Func(const string &, const string &);
  typedef decltype(lengthCompare) Func2;
  // FuncP和FuncP2是指向函数的指针
  typedef bool(*FuncP)(const string &, const string &);
  typedef decltype(lengthCompare) *FuncP2;
  ```
* 跟数组类似，虽然不能返回一个函数，但是能返回指向函数类型的指针。然而，我们必须把返回类型写成指针形式，编译器不会自动将函数返回类型当成对应的指针类型处理；
  ```c++
  int (*f1(int))(int *, int);
  auto f1(int) -> int (*)(int *, int);
  ```
# 7章 类
1. 构造函数
* 只有当类没有声明任何构造函数时，编译器才会自动地生成默认构造函数；
* 拷贝，赋值和析构主要参考13章；
2. class或者struct
* 类既可以使用class也可以使用struct，唯一区别就是默认访问权限，struct默认是public，class默认是private；
3. 友元
* 类可以允许其他类或函数访问它的非公有成员，方法是令其他类或者函数成为它的友元（friend）；
* 一般来说，最好在类定义开始或结束前的位置集中声明友元；
4. 类的其他特性
  ```c++
  class Screen {
  public:
  	typedef std::string::size_type pos;
  	Screen() = default;	// 因为Screen有另一个构造函数，所以本函数是必需的
  	Screen(pos ht, pos wd, char c) : height(ht), width(wd), contents(ht * wd, c) {}
  	char get() const {
  		return contents[cursor];	// 读取光标处的字符，隐式内联
  	}
  	inline char get(pos ht, pos wd) const;	// 显式内联
  	Screen &move(pos r, pos c);	// 能在之后被设为内联
  private:
  	pos cursor = 0;	// 类内初始值，必需以符号=或者花括号表示
  	pos height = 0, width = 0;
  	std::string contents;
  };
  ```
5. 类的声明
* `class Screen;`对于类型Screen来说，在它声明之后定义之前是一个不完全类型（incomplete type），也就是说，此时我们已知Screen是一个类类型，但是不清楚它到底包含哪些成员；
* 不完全类型使用场景有限：可以定义指向这种类型的指针或引用，也可以声明（但不能定义）以不完全类型作为参数或者返回类型的函数；
* 直到类被定义后数据成员才能被声明成这种类型。换句话说，我们必须完成类的定义，然后编译器才能知道存储该数据成员需要多少空间。因为只有当类全部完成后类才算被定义，所以一个类的成员类型不能是该类自己。然而，一旦一个类的名字出现后，它就被认为是声明过了（但尚未定义），因此类允许包含指向它自身类型的引用或指针；
  ```c++
  class Link_screen {
  	Screen window;
  	Link_screen *next;
  	Link_screen *prev;
  };
  ```
* 成员初始化顺序，与他们在类定义中的出现顺序一致：第一个成员先被初始化，然后第二个，以此类推；
  ```c++
  class X {
  	int i;
  	int j;
  public:
  	// 未定义的：i在j之前被初始化
  	X(int val): j(val), i(j) {}
  };
  ```
6. explicit
* explicit只能用于直接初始化（阻止隐式转换）；
  ```c++
  Sales_data item1(null_book);	// 正确：直接初始化
  Sales_data item2 = null_book;	// 错误：不能将explicit构造函数用于拷贝形式的初始化过程
  ```
* 只对一个实参的构造函数有效；
* 需要多个实参的构造函数不能用于执行隐式转换，所以无须将这些构造函数指定为explicit；
* 只能在类内声明构造函数时使用explicit，在类外部定义时不应重复；
# 8章 IO库
