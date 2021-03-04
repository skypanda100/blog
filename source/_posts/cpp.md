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
* 返回数组指针
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
