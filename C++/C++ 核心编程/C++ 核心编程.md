# C++ 核心编程

# 内存分区模型

问题：`int a = 10;`，你只知道它存了起来。但你有没有想过：它存在哪了？什么时候会被删掉？

C++ 程序在运行时，会将内存切分成 4 个主要区域

|  区域名称   |   谁来管理    |          存什么东西          |                   特点                    |
| :---------: | :-----------: | :--------------------------: | :---------------------------------------: |
|   代码区    |   操作系统    |       所有的二进制代码       |                只读、共享                 |
| 全局/静态区 |   操作系统    | 全局变量、`static`变量、常量 |         寿命最长 (程序结束才销毁)         |
|    栈区     | 编译器 (自动) |      局部变量、函数参数      | 寿命短 (出了作用域就死)、空间小、速度极快 |
|    堆区     | 程序员 (手动) |       `new` 出来的东西       |      寿命由你定、空间巨大、速度稍慢       |

**内存四区意义：**

不同区域存放的数据，赋予不同的生命周期, 给我们更大的灵活编程

```c++
#include <iostream>
using namespace std;

// --- 全局区 ---
int g_a = 10;            // 全局变量
const int c_g_a = 10;    // const修饰的全局变量 (也在全局区)

int main() {
    // --- 栈区 ---
    int a = 10;          // 局部变量
    int b = 20;          // 局部变量
    const int c_l_a = 10;// const修饰的局部变量 (注意：它在栈区，不在全局区！)
    
    char arr[] = "Hello";// 字符串数组，存放在栈区
    const char* pStr = "World"; // "World" 这个字符串常量在全局区，但指针 pStr 在栈区

    // --- 堆区 ---
    // new int(10) 在堆区开辟了一个空间
    // 指针 p 本身是在栈区 (因为它是个局部变量)
    // p 保存的是堆区那个地址
    int * p = new int(10); 

    cout << "全局变量地址: " << &g_a << endl;
    cout << "局部变量地址: " << &a << endl;
    cout << "堆区数据地址: " << p << endl;

    //释放堆区数据
    delete p; 

    return 0;
}
```

## 在程序运行前

- 也就是刚刚编译好，生成了 `.exe` 文件，但还没双击运行它的时候，程序并不占用内存。它只是硬盘上的一个文件。
- 在这个文件里，并没有堆区和栈区

### 代码区

- 存储你编写的所有 C++ 代码编译后的二进制机器指令

**特点**：

1. 共享：例如双击打开 10 个记事本进程，内存中仅需加载一份代码区副本。因所有进程执行逻辑完全一致，无需重复复制代码指令，各进程共享同一份即可正常运行。
2. 只读：使其只读的原因是防止程序意外地修改了它的指令

### 全局/静态区

- 全局变量和静态变量存放在此.
- 全局区还包含了常量区, 字符串常量和其他常量也存放在此.
- 该区域的数据在程序结束后由操作系统释放.

**包含两部分**：

1. 已初始化区：存储显式初始化的全局变量、静态变量
2. 未初始化区：存储未显式初始化或初始化为 0 的全局变量、静态变量

### 为什么没有栈和堆

1. 栈区：是函数调用时临时产生的。程序没运行，就没有函数调用，自然没有栈。
2. 堆区：是程序员代码里写了 new 才会申请的。程序没运行，没人去执行 new，自然没有堆。

```c++
//全局变量
int g_a = 10;
int g_b = 10;
//全局常量
const int c_g_a = 10;
const int c_g_b = 10;
int main() {

	//局部变量
	int a = 10;
	int b = 10;
	//打印地址
	cout << "局部变量a地址为： " << (int)&a << endl;
	cout << "局部变量b地址为： " << (int)&b << endl;

	cout << "全局变量g_a地址为： " <<  (int)&g_a << endl;
	cout << "全局变量g_b地址为： " <<  (int)&g_b << endl;

	//静态变量
	static int s_a = 10;
	static int s_b = 10;

	cout << "静态变量s_a地址为： " << (int)&s_a << endl;
	cout << "静态变量s_b地址为： " << (int)&s_b << endl;

	cout << "字符串常量地址为： " << (int)&"hello world" << endl;
	cout << "字符串常量地址为： " << (int)&"hello world1" << endl;

	cout << "全局常量c_g_a地址为： " << (int)&c_g_a << endl;
	cout << "全局常量c_g_b地址为： " << (int)&c_g_b << endl;

	const int c_l_a = 10;
	const int c_l_b = 10;
	cout << "局部常量c_l_a地址为： " << (int)&c_l_a << endl;
	cout << "局部常量c_l_b地址为： " << (int)&c_l_b << endl;

	return 0;
}
```

![image-20251124112721325](attachments\image-20251129144250650.png)

总结：

- C++中在程序运行前分为全局区和代码区
- 代码区特点是共享和只读
- 全局区中存放全局变量、静态变量、常量
- 常量区中存放 const修饰的全局常量 和 字符串常量

## 在程序运行后

### 栈区

-  由编译器自动分配释放, 存放函数的参数值,局部变量等
-  注意事项：不要返回局部变量的地址，栈区开辟的数据由编译器自动释放

```c++
#include <iostream>
using namespace std;

// 函数：返回局部变量地址（错误示例）
int * func() {
    int a = 10;  // 局部变量，函数结束时销毁
    return &a;   // 返回其地址，悬垂指针！
}

// 程序入口
int main() {
    int *p = func();  // p 指向已销毁的地址
    cout << *p << endl;  // 未定义行为，可能输出10或垃圾
    cout << *p << endl;  // 同样未定义，可能不同
    
    // 成功结束（但代码有问题）
    return 0;
}
```

### 堆区

- 由程序员分配释放,若程序员不释放,程序结束时由操作系统回收
- 在C++中主要利用new在堆区开辟内存

```c++
#include <iostream>
using namespace std;

// 函数：动态分配内存，返回指针（正确示例）
int* func() {
    int* a = new int(10);  // 在堆上分配，函数结束不销毁
    return a;              // 返回指针，外部可继续使用
}

// 程序入口
int main() {
    int *p = func();       // p 指向堆内存
    cout << *p << endl;    // 输出10
    cout << *p << endl;    // 再次输出10
    
    // 注意：使用 new 后，应在合适处 delete p; 释放内存，避免泄漏
    //delete p;
    return 0;
}
```

**总结：**

1. 堆区数据由程序员管理开辟和释放
2. 堆区数据利用new关键字进行开辟内存

## new 操作符

-  C++中利用new操作符在堆区开辟数据
-  堆区开辟的数据，由程序员手动开辟，手动释放，释放利用操作符 delete

1. 基本语法

```c++
指针变量 = new 数据类型(初始值);
```

利用`new`创建的数据，会返回该数据对应的类型的指针

```c++
#include <iostream>
using namespace std;

int main() {
    // 使用 new 在堆区分配内存
    int * p = new int(10);  // 开辟空间，初始化为10，返回地址给p
    
    // 解引用访问数据
    cout << "堆区的数据是: " << *p << endl;
    
    // 释放内存（防止泄漏）
    delete p;
    
    return 0;
}
```

2. 开辟数组

```c++
指针变量 = new 数据类型[数组长度];
```

```c++
#include <iostream>
using namespace std;

int main() {
    int n;
    
    // 提示输入数组大小
    cout << "请输入你想创建的数组大小: ";
    cin >> n;  // 用户输入变量大小
    
    // 动态分配数组（支持变量长度）
    int * arr = new int[n];
    
    // 初始化数组
    for (int i = 0; i < n; i++) {
        arr[i] = i + 100;  // 用下标访问
    }
    
    // 输出示例
    cout << "第一个元素: " << arr[0] << endl;
    
    // 释放数组内存
    delete[] arr;
    
    return 0;
}
```

```c++
//堆区开辟数组
int main() {

	int* arr = new int[10];

	for (int i = 0; i < 10; i++)
	{
		arr[i] = i + 100;
	}

	for (int i = 0; i < 10; i++)
	{
		cout << arr[i] << endl;
	}
	//释放数组 delete 后加 []
	delete[] arr;

	return 0;
}
```

# 引用

## 引用的概述与定义

1. 什么是引用：引用就是给变量起个“外号”

- 变量名：`a`
- 引用：`b`

> 本质上：`a` 和 `b` 对应的是同一块内存空间。`a`就是`b`，`b`就是`a`

2. 定义

```c++
数据类型 &别名 = 原名;
```

```c++
#include <iostream>
using namespace std;

int main() {
    int a = 10;
    
    // 创建引用：b 是 a 的别名（必须立即绑定）
    int &b = a;
    
    // 访问：b 和 a 值相同
    cout << "a = " << a << endl;  // 10
    cout << "b = " << b << endl;  // 10
    
    // 修改：通过 b 修改，a 也会变
    b = 100;
    cout << "修改后 a = " << a << endl;  // 100
    
    return 0;
}
```

3. 注意事项

- 必须初始化
- 不能是空的：不存在“空引用”
- 从一而终 (不能改变指向)

```c++
int main() {

	int a = 10;
	int b = 20;
	//int &c; //错误，引用必须初始化
	int &c = a; //一旦初始化后，就不可以更改
	c = b; //这是赋值操作，不是更改引用

	cout << "a = " << a << endl;
	cout << "b = " << b << endl;
	cout << "c = " << c << endl;

	return 0;
}
```

## 引用做函数参数

**作用：**函数传参时，可以利用引用的技术让形参修饰实参

**优点：**可以简化指针修改实参

### 核心原理

```c++
void func(int &x)
```

- 系统不会去内存里开辟新的空间来存这个参数
- 它不会把原来的数据复制一份
- 形参 x 直接变成了实参的“外号”
- 函数内部对 x 的所有操作，实际上都是直接作用在原来的内存块上

### 三种传递方式

- 值传递
- 指针/地址传递
- 引用传递

```c++
//1. 值传递
void mySwap01(int a, int b) {
	int temp = a;
	a = b;
	b = temp;
}

//2. 地址传递
void mySwap02(int* a, int* b) {
	int temp = *a;
	*a = *b;
	*b = temp;
}

//3. 引用传递
void mySwap03(int& a, int& b) {
	int temp = a;
	a = b;
	b = temp;
}

int main() {

	int a = 10;
	int b = 20;

	mySwap01(a, b);
	cout << "a:" << a << " b:" << b << endl;

	mySwap02(&a, &b);
	cout << "a:" << a << " b:" << b << endl;

	mySwap03(a, b);
	cout << "a:" << a << " b:" << b << endl;

	return 0;
}
```

运行结果

```
a:10 b:20
a:20 b:10
a:10 b:20
```

>总结：通过引用参数产生的效果同按地址传递是一样的。引用的语法更清楚简单

## 引用做函数返回值

- 作用：引用是可以作为函数的返回值存在的
- 注意：不要返回局部变量引用
- 用法：函数调用作为左值

> **左值**：能放在等号左边的值。通常指“有名字、有内存地址”的实体。即变量本身，函数调用可以被赋值

1. 语法

```c++
// 以前：返回 int 的值（复制一份带回去）
int func() { ... }

// 现在：返回 int 的引用（把变量本尊带回去）
int& func() { ... }
```

2. 返回静态/全局变量的引用

非要返回引用，那这个变量的寿命必须比函数长

```c++
//返回局部变量引用
int& test01() {
	int a = 10; //局部变量
	return a;
}

//返回静态变量引用
int& test02() {
	static int a = 20;
	return a;
}

int main() {

	//不能返回局部变量的引用
	int& ref = test01();
	cout << "ref = " << ref << endl;
	cout << "ref = " << ref << endl;

	//如果函数做左值，那么必须返回引用
	int& ref2 = test02();
	cout << "ref2 = " << ref2 << endl;
	cout << "ref2 = " << ref2 << endl;

	test02() = 1000;

	cout << "ref2 = " << ref2 << endl;
	cout << "ref2 = " << ref2 << endl;
	return 0;
}
```

运行结果

```
ref = -858993460
ref = -858993460
ref2 = 20
ref2 = 20
ref2 = 1000
ref2 = 1000
```

## 引用的本质

本质：引用的本质在c++内部实现是一个指针常量.

>1. 自动转换：把 &ref 变成了 int * const ref
>2. 自动取地址：把 b 变成了 &b
>3. 自动解引用：当你用 ref 时，编译器自动帮你加了 *

```c++
#include <iostream>
using namespace std;

int main() {
    // 使用引用
    int a = 10;
    int &ref = a;  // 定义引用，绑定到 a
    ref = 20;      // 修改引用，等同于修改 a
    
    cout << "通过引用修改后 a = " << a << endl;  // 输出 20
    
    // 等价于 const 指针（底层实现）
    int b = 10;
    int * const ptr = &b;  // const 指针，绑定到 b 的地址，不可改指向
    *ptr = 20;             // 通过解引用修改 b
    
    cout << "通过 const 指针修改后 b = " << b << endl;  // 输出 20
    
    return 0;
}
```

**本质的核心：**

1. 引用必须初始化
2. 引用一旦绑定，终身不改——指针常量
3. 使用时不用加 `*`

>在 C++ 底层实现中，引用只是一个被编译器封装好的‘指针常量’。 它在内存中占用的空间与指针相同（4或8字节）。 区别在于，编译器为引用提供了自动解引用的语法糖，让我们编写代码时更安全、更简洁。

## 常量引用

**作用：**常量引用主要用来修饰形参，防止误操作

在函数形参列表中，可以加`const`修饰形参，防止形参改变实参

### 什么是常量引用

```c++
const 数据类型 &别名 = 原名;
```

> 给变量起个外号，但通过这个外号，只能读数据，不能修改数据

```c++
int a = 10;
const int &b = a; // b 是 a 的只读别名

cout << b << endl; //输出10

// b = 20; //报错，只能读数据，不能改数据
a = 20;   //原名可以改
```

### 引用字面量

普通引用非常挑剔，它只能引用一块合法的内存变量，不能引用“数字”本身

```c++
// int &ref = 10; //报错，10 只是个数字，没有地址，引用没法绑定。
```

常量引用非常宽容，它可以直接引用数字

```c++
const int &ref = 10; 
```

底层原理

```c++
// 编译器自动生成了一个临时变量
int temp = 10;
const int &ref = temp; // 让引用指向这个临时变量
```

> 意味着函数参数如果是常量引用，调用时可以直接传数字进去，不用专门先定义一个变量。

```c++
//引用使用的场景，通常用来修饰形参
void showValue(const int& v) {
	//v += 10;
	cout << v << endl;
}

int main() {

	//int& ref = 10;  引用本身需要一个合法的内存空间，因此这行错误
	//加入const就可以了，编译器优化代码，int temp = 10; const int& ref = temp;
	const int& ref = 10;

	//ref = 100;  //加入const后不可以修改变量
	cout << ref << endl;

	//函数中利用常量引用防止误操作修改实参
	int a = 10;
	showValue(a);

	return 0;
}

//运行结果
//10
//10
```

# 函数提高

## 函数默认参数

在C++中，函数的形参列表中的形参是可以有默认值的。

```c++
返回值类型 函数名(参数类型 参数名 = 默认值) { ... }
```

```c++
#include <iostream>
#include <string>
using namespace std;

// 函数定义：自我介绍，age 有默认值 18
void introduce(string name, int age = 18) {
    cout << "我叫 " << name << "，今年 " << age << " 岁。" << endl;
}

int main() {
    // 只传名字，使用默认年龄
    introduce("张三");
    
    // 指定年龄，覆盖默认值
    introduce("李四", 30);
    
    return 0;
}
```

函数重载：

> 当默认参数遇上函数重载，很容易搞出“二选一”的死局。

```c++
// 函数 A：带默认参数
void test(int a, int b = 10) {
    cout << "调用了 A" << endl;
}

// 函数 B：只有一个参数
void test(int a) {
    cout << "调用了 B" << endl;
}

int main() {
    // test(10, 20); // ✅ 没问题，只能是 A

    // test(10); // ❌ 报错！！
    // 编译器懵了：
    // 你是想调 A (用默认值)？
    // 还是想调 B (正好一个参数)？
    // 这种情况叫“二义性”，编译不通过。
}
```

- 注意事项

1. 一旦某个参数有了默认值，那么它右边的所有参数，都必须有默认值。
2. 默认参数只能写一次！通常写在“声明”里。

## 函数占位参数

C++中函数的形参列表里可以有占位参数，用来做占位，调用函数时必须填补该位置

> 简单说，就是在定义函数时，只写参数类型，不写参数名字。

```c++
返回值类型 函数名(数据类型) { ... }
```

- 函数内部根本拿不到这个数据（因为它没名字），但在调用函数时，必须传一个数据进去填补这个位置
- 在函数体内部，没法读取、没法修改这个参数

```c++
#include <iostream>
using namespace std;

// 第二个 int 就是占位参数
// 意思是：调用我的时候，必须给我两个整数，但我只用第一个
void func(int a, int) { 
    cout << "a = " << a << endl;
    // cout << b; // ❌ 报错！根本就没有 b 这个名字
}

int main() {
    // func(10); //报错！参数太少
    
    func(10, 999); //正确。虽然 999 传进去就被丢掉了，但形式上必须有。
    
    return 0;
}
```

>占位参数主要有两个用途，一个是为了兼容旧代码，另一个是为了运算符重载

## 函数重载

在同一个作用域下，函数名相同，但参数列表不同

1. 参数个数不同

```c++
add(int a)

add(int a, int b)
```

2. 参数类型不同

```c++
print(int a)

print(double a)
```

3. 参数顺序不同

```c++
func(int a, double b)

func(double a, int b)
```

4. 返回值不同不算重载

```c++
void test() { ... }
int  test() { ... }
```

报错：当你调用 `test();` 时，编译器根本不知道你想要哪个

5. 引用作为重载条件

引用和`const`引用，在重载眼里是不一样的。

```c++
// 版本 A：接收可修改的引用
void func(int &a) {
    cout << "调用了 (int &a)" << endl;
}

// 版本 B：接收只读引用
void func(const int &a) {
    cout << "调用了 (const int &a)" << endl;
}

int main() {
    int a = 10;
    func(a); // 调用 A (因为 a 是变量，可读可写，A更匹配)
    
    func(10); // 调用 B (因为 10 是常量，只能赋给 const 引用)
}
```

6. 重载遇上默认参数

```c++
// 函数 1：带默认参数
void myFunc(int a, int b = 10) {
    cout << "Func 1" << endl;
}

// 函数 2：只有一个参数
void myFunc(int a) {
    cout << "Func 2" << endl;
}

int main() {
    // myFunc(10, 20); //没问题，只能是 Func 1
    
    // myFunc(10); //程序不知道你要那个函数，原因两个函数都可以用，报错
}
```

**示例**

```c++
#include <iostream>
using namespace std;

// 函数重载示例（同一作用域）
void func() {
    cout << "func 的调用！" << endl;
}
void func(int a) {
    cout << "func (int a) 的调用！" << endl;
}
void func(double a) {
    cout << "func (double a)的调用！" << endl;
}
void func(int a, double b) {
    cout << "func (int a ,double b) 的调用！" << endl;
}
void func(double a, int b) {
    cout << "func (double a ,int b)的调用！" << endl;
}

// 重载注意事项：引用作为重载条件
void func_ref(int &a) {
    cout << "func (int &a) 调用 " << endl;
}
void func_ref(const int &a) {
    cout << "func (const int &a) 调用 " << endl;
}

// 重载注意事项：默认参数可能产生歧义
void func2(int a, int b = 10) {
    cout << "func2(int a, int b = 10) 调用" << endl;
}
void func2(int a) {
    cout << "func2(int a) 调用" << endl;
}

int main() {
    // 函数重载调用
    func();
    func(10);
    func(3.14);
    func(10, 3.14);
    func(3.14, 10);
    
    // 引用重载
    int a = 10;
    func_ref(a);  // 无 const
    func_ref(10); // 有 const
    
    // 默认参数示例（注释掉歧义调用）
    // func2(10); // 歧义，避免使用
    return 0;
}
```

```
sh: 1: pause: not found
func 的调用！
func (int a) 的调用！
func (double a)的调用！
func (int a ,double b) 的调用！
func (double a ,int b)的调用！
func (int &a) 调用 
func (const int &a) 调用
```

# 面向对象

- C++面向对象的三大特性为：封装、继承、多态
- C++认为万事万物都皆为对象，对象上有其属性和行为
- 两大核心概念：类与对象

**例如：**

-  人可以作为对象，属性有姓名、年龄、身高、体重…，行为有走、跑、跳、吃饭、唱歌…
-  车也可以作为对象，属性有轮胎、方向盘、车灯…,行为有载人、放音乐、放空调…
-  具有相同性质的对象，我们可以抽象称为类，人属于人类，车属于车类

## 封装

### 封装的概述与定义

封装是C++面向对象三大特性之一

封装的意义：

- 将属性和行为作为一个整体，表现生活中的事物
- 将属性和行为加以权限控制

1. 封装的目的

保护数据：防止外部随意修改。

隐藏细节：使用者不需要知道内部怎么实现的，会用就行。

2. 访问权限

C++ 提供了三个关键字来控制“谁能看，谁不能看”。

|   关键字    |      权限说明      |                      谁可以访问                      |
| :---------: | :----------------: | :--------------------------------------------------: |
|  `public`   | 公共权限--完全开放 |       类内部、类外部（main函数）都可以随变用。       |
|  `private`  |   私有权限--私密   |     只有类内部的函数可以用。外部连看一眼都不行。     |
| `protected` |   保护权限--继承   | 类内部可以访问，子类（继承）可以访问。外部不能访问。 |

3. 基本语法

```c++
class 类名 {
public: // 权限标签
    // 1. 属性 (变量)
    int 变量1;
    
    // 2. 行为 (函数)
    void 函数1() {
        // ...
    }
};
```

> 成员变量：类里面的变量（属性）。

> 成员函数：类里面的函数（方法/行为）。

>成员变量（属性）全部设为 `private`。 成员函数（行为）通常设为 `public`

4. 创建对象与访问成员信息

如何访问成员变量和成员函数，在 C++ 中，主要分为两种情况：普通对象和指针对象。现在只说明普通对象

在 C++ 中，创建对象（实例化）主要有两种方式。取决于你想把对象放在内存的哪个区域（栈区还是堆区）。现在只说明栈区

```c++
//先创建对象——栈区
类名 对象名();

//在访问成员变量和函数——普通变量
对象名.成员变量;
对象名.成员函数();
```

```c++
class Cat {
public:
    string name;
    void run() {
        cout << name << " 正在跑！" << endl;
    }
};

int main() {
    // 1. 创建实体对象 (在栈区)
    Cat myCat;
    myCat.name = "汤姆"; // 访问成员变量

    // 2. 调用成员函数
    myCat.run(); 
    return 0;
}
```

5. 示例：设计一个圆类，求圆的周长

```c++
#include <iostream>
using namespace std;

// 全局常量
const double PI = 3.14;

// 类定义：封装圆类
class Circle {
public:  // 公共访问权限
    
    // 属性
    int m_r;  // 半径
    
    // 行为（方法）
    double calculateZC() {
        return 2 * PI * m_r;  // 计算周长
    }
};

// 程序入口
int main() {
    // 创建圆对象
    Circle c1;
    c1.m_r = 10;  // 赋值半径
    
    // 输出周长
    cout << "圆的周长为： " << c1.calculateZC() << endl;
    
    return 0;
}
```

6. 示例：设计一个学生类，属性有姓名和学号，可以给姓名和学号赋值，可以显示学生的姓名和学号

```c++
#include <iostream>
#include <string>
using namespace std;

// 学生类定义
class Student {
public:
    // 设置姓名
    void setName(string name) {
        m_name = name;
    }
    // 设置ID
    void setID(int id) {
        m_id = id;
    }
    // 显示学生信息
    void showStudent() {
        cout << "name:" << m_name << " ID:" << m_id << endl;
    }
    
public:
    string m_name;  // 姓名
    int m_id;       // ID
};

int main() {
    // 创建学生对象
    Student stu;
    stu.setName("德玛西亚");  // 设置姓名
    stu.setID(250);           // 设置ID
    stu.showStudent();        // 显示信息
    
    // 成功结束
    return 0;
}
```

7. 访问权限的区别

```c++
#include <iostream>
#include <string>
using namespace std;

// 类定义：三种访问权限
class Person {
public:    // 公共权限：类内/外均可访问
    string m_Name;  // 姓名
    
protected: // 保护权限：类内可访问，类外不可
    string m_Car;   // 汽车
    
private:   // 私有权限：类内可访问，类外不可
    int m_Password; // 银行卡密码
    
public:
    void func() {
        m_Name = "张三";     // 公共
        m_Car = "拖拉机";    // 保护
        m_Password = 123456; // 私有
    }
};

int main() {
    Person p;
    p.m_Name = "李四";  // 公共可访问
    // p.m_Car = "奔驰"; // 保护不可访问
    // p.m_Password = 123; // 私有不可访问
    
    return 0;
}
```

### struct和class区别

在 C++ 中，`struct` 和 `class` 几乎是一模一样的，唯一的区别在于“默认权限”。

- struct 默认权限为公共
- class 默认权限为私有

```c++
#include <iostream>
using namespace std;

class C1 {
    int m_A; // 没写权限，默认是 private
};

struct C2 {
    int m_A; // 没写权限，默认是 public
};

int main() {
    C1 c1;
    // c1.m_A = 10; //报错！"不可访问"，因为是 private

    C2 c2;
    c2.m_A = 10;    //通过！因为是 public

    return 0;
}
```

>当class类中，成员变量和方法设置访问权限时，main函数就可以使用了

### get/set方法

- 在成员属性/变量中通常设置为私有`private`，在设置`get`和`set`方法
- 可以自己控制读写权限

> 核心原理：将成员变量设置为私有，在设置get/set方法，可以使外部通过我设置的方法来访问我的成员变量——控制

1. 标准写法

写一个 `Person` 类。

Step1.私有化属性

变量名前面通常加 `m_` (member)，这是 C++ 的命名惯例，方便区分参数和成员变量。

```c++
class Person {
private:
    string m_Name; // 姓名
    int m_Age;     // 年龄
};
```

Step2.编写 Setter

用于修改数据。可以在这里加逻辑判断。

```c++
public:
    void setAge(int age) {
        if (age < 0 || age > 150) {
            cout << "年龄输入离谱，赋值失败！" << endl;
            m_Age = 0; // 给个默认值或者直接 return
            return;
        }
        m_Age = age; // 合法才赋值
    }
```

Step3.编写 Getter

用于获取数据

```c++
public:
    int getAge() {
        return m_Age; // 把数据拿出去
    }
```

2. 示例

```c++
#include <iostream>
#include <string>
using namespace std;

// 类定义：Person
class Person {
private:
    string m_Name;   // 姓名（私有）
    int m_Age;       // 年龄（私有）
    string m_Lover;  // 情人（私有，只写）
    
public:
    // 设置姓名
    void setName(string name) {
        m_Name = name;
    }
    // 获取姓名
    string getName() {
        return m_Name;
    }
    
    // 设置年龄（范围检查）
    void setAge(int age) {
        if (age < 0 || age > 150) {
            cout << "警告：年龄设置非法！" << endl;
            return;
        }
        m_Age = age;
    }
    // 获取年龄
    int getAge() {
        return m_Age;
    }
    
    // 设置情人（无获取）
    void setLover(string lover) {
        m_Lover = lover;
    }
};

// 程序入口
int main() {
    Person p;
    
    // 设置姓名
    p.setName("张三");
    
    // 无效年龄（被拦截）
    p.setAge(1000);
    
    // 有效年龄
    p.setAge(18);
    
    // 输出信息
    cout << "姓名：" << p.getName() << endl;
    cout << "年龄：" << p.getAge() << endl;
    
    return 0;
}
```

## 对象的初始化与清理

- 生活中我们买的电子产品都基本会有出厂设置，在某一天我们不用时候也会删除一些自己信息数据保证安全
- C++中的面向对象来源于生活，每个对象也都会有初始设置以及对象销毁前的清理数据的设置。

在 C++ 中，对象也有生老病死：

1. 出生（初始化）：需要设置初始值。
2. 死亡（清理）：需要释放内存、关闭文件。

C++ 提供了两个特殊的函数来自动帮我们做这两件事：构造函数和析构函数。

> 对象的初始化和清理工作是编译器强制要我们做的事情，因此如果我们不提供构造和析构，编译器会提供编译器提供的构造函数和析构函数是空实现。

- 构造函数：主要作用在于创建对象时为对象的成员属性赋值，构造函数由编译器自动调用，无须手动调用。
- 析构函数：主要作用在于对象销毁前系统自动调用，执行一些清理工作。

### 构造函数和析构函数

1. 构造函数——初始化

作用：在对象创建时，自动调用，用来初始化成员变量。

特点：

- 函数名与类名相同

- 没有返回值，连 `void` 都不写

- 可以有参数，可以重载（也就是可以有多种初始化方式）。

- 程序在调用对象时候会自动调用构造，无须手动调用,而且只会调用一次

```c++
class Person {
public:
    string name;
    int age;

    // 1. 默认构造函数 - 创建无名对象
    Person() {
        cout << "Person 无参构造调用 (出生了，但没名字)" << endl;
        name = "无名氏";
        age = 0;
    }

    // 2. 带参构造函数 - 创建指定姓名年龄的对象
    Person(string n, int a) {
        cout << "Person 有参构造调用 (带着名字出生了)" << endl;
        name = n;
        age = a;
    }
};

int main() {
    // 调用默认构造
    Person p1; 
    // 注意：Person p1(); 会被误认为函数声明
    
    // 调用带参构造 - 直接初始化
    Person p2("张三", 18); 

    cout << "p2的名字: " << p2.name << endl;
    
    return 0;
}
```

>默认构造在创建对象中与普通对象有关
>
>即默认/无参构造需要自己访问成员变量，进行创建普通对象



2. 析构函数——清理

作用：在对象销毁前，自动调用，用来执行清理工作（比如释放堆区内存）。

特点：

- 析构函数，没有返回值也不写void
- 函数名称与类名相同,在名称前加上符号 `~`
- 析构函数不可以有参数，因此不可以发生重载
- 程序在对象销毁前会自动调用析构，无须手动调用,而且只会调用一次

```c++
class Person {
public:
    // ... 构造函数省略 ...

    // 析构函数
    ~Person() {
        cout << "Person 析构函数调用" << endl;
    }
};
```

3. 示例

```c++
class Person
{
public:
	//构造函数
	Person()
	{
		cout << "Person的构造函数调用" << endl;
	}
	//析构函数
	~Person()
	{
		cout << "Person的析构函数调用" << endl;
	}

};

void test01()
{
	Person p;
}

int main() {
	
	test01();
	return 0;
}
```

### 构造函数的分类

两种分类方式：

-  按参数分为： 有参构造和无参构造
-  按类型分为： 普通构造和拷贝构造

1. 普通构造和拷贝构造的概述

- 普通构造函数：用于在对象实例化时从零构建新对象。
  - 作用是初始化对象的成员变量，按需分配对象所需资源，确保对象创建后即处于语法和逻辑上的有效状态。

- 拷贝构造函数：参数为同类对象的常量引用 `const` 类名`&`，用于基于已存在的同类对象创建新对象。
  - 作用是将原对象的成员变量值逐份拷贝至新对象，生成与原对象状态完全一致的独立实例，避免直接赋值导致的资源浅拷贝问题。

2. 语法对比

A.普通构造 (无参/有参)

```c++
// 无参
Person() { ... } 
// 有参
Person(int a) { ... }
```

B.拷贝构造

作用：使用一个已经存在的对象，来初始化一个新的对象。

```c++
类名(const 类名 & 变量名) { ... }
```

```c++
// 拷贝构造函数 - 基于现有对象创建新对象
Person(const Person &p) {
    age = p.age;  // 复制年龄
    name = p.name; // 复制姓名
    cout << "拷贝构造函数调用 (克隆成功)" << endl;
}
```

3. 示例：普通构造与拷贝构造区别

```c++
class Person {
public:
    string name;
    int age;

    // 默认构造函数
    Person() {
        cout << "Person 默认构造调用" << endl;
        name = "无名氏";
        age = 0;
    }

    // 带参构造函数
    Person(string n, int a) {
        cout << "Person 带参构造调用" << endl;
        name = n;
        age = a;
    }

    // 拷贝构造函数 - 基于现有对象创建新对象
    Person(const Person &p) {
        age = p.age;   // 复制年龄
        name = p.name; // 复制姓名
        cout << "拷贝构造函数调用 (克隆成功)" << endl;
    }
};

int main() {
    Person p1("张三", 18);
    Person p2 = p1; // 触发拷贝构造函数
    
    cout << "p2: " << p2.name << ", " << p2.age << endl;
    return 0;
}
```

### 构造函数的调用

三种调用方式：

-  括号法
-  显示法
-  隐式转换法

1. 括号法

这是最经典、最直观的写法，像调用函数一样。

- 无参：`Person p1;`

- 有参：`Person p2(10);`

- 拷贝：`Person p3(p2);`

> 注意：调用无参构造时，用`Person p1;`，不用`Person p1();`，原因：编译器会认为 `Person p1();` 是在声明一个函数

2. 显示法

这种写法非常清晰，明确告诉大家“我在创建一个 Person 对象”。

- 有参：`Person p1 = Person(10);`
- 拷贝：`Person p2 = Person(p1);`

>核心概念：匿名对象
>
>`Person(10)`单独写就是匿名对象  当前行结束之后，马上析构

>不要利用拷贝构造函数初始化匿名对象： `Person(p2);`编译器认为是对象声明

3. 隐式转换法

这是 C++ 特有的“语法糖”，写起来最像普通变量赋值。

有参：`Person p1 = 10;`

拷贝：`Person p2 = p1;`

4. 示例

```c++
//1、构造函数分类
// 按照参数分类分为 有参和无参构造   无参又称为默认构造函数
// 按照类型分类分为 普通构造和拷贝构造

class Person {
public:
	//无参（默认）构造函数
	Person() {
		cout << "无参构造函数!" << endl;
	}
	//有参构造函数
	Person(int a) {
		age = a;
		cout << "有参构造函数!" << endl;
	}
	//拷贝构造函数
	Person(const Person& p) {
		age = p.age;
		cout << "拷贝构造函数!" << endl;
	}
	//析构函数
	~Person() {
		cout << "析构函数!" << endl;
	}
public:
	int age;
};

//2、构造函数的调用
//调用无参构造函数
void test01() {
	Person p; //调用无参构造函数
}

//调用有参的构造函数
void test02() {

	//2.1  括号法，常用
	Person p1(10);
	//注意1：调用无参构造函数不能加括号，如果加了编译器认为这是一个函数声明
	//Person p2();

	//2.2 显式法
	Person p2 = Person(10); 
	Person p3 = Person(p2);
	//Person(10)单独写就是匿名对象  当前行结束之后，马上析构

	//2.3 隐式转换法
	Person p4 = 10; // Person p4 = Person(10); 
	Person p5 = p4; // Person p5 = Person(p4); 

	//注意2：不能利用 拷贝构造函数 初始化匿名对象 编译器认为是对象声明
	//Person p5(p4);
}

int main() {

	test01();
	//test02();

	return 0;
}
```

### 拷贝构造函数调用时机

C++中拷贝构造函数调用时机通常有三种情况

- 使用一个已经创建完毕的对象来初始化一个新对象
- 值传递的方式给函数参数传值
- 以值方式返回局部对象

1. 使用一个已创建的对象来初始化一个新对象

这是最直观、最标准的用法。

场景：你已经有了 `p1`，你想造一个 `p2`，并且让 `p2` 和 `p1` 一模一样。

```c++
void test01() {
    Person p1(20); // 调用有参构造
    
    // 触发拷贝构造的两种写法：
    Person p2(p1); // 写法 1：括号法
    Person p3 = p1; // 写法 2：隐式转换法 (注意：这是初始化，不是赋值)
    
    // 这里的 p2 和 p3 都是通过拷贝构造“克隆”出来的
}
```

2. 以值传递的方式给函数参数传值

场景：函数的参数是 `Person p`（不是引用，也不是指针）。当你把外面的对象传进去时，编译器为了不破坏原件，会调用拷贝构造函数，印一份副本给函数使用。

```c++
// 参数是值传递
void doWork(Person p) { 
    // 这里的 p 是形参，它是通过拷贝构造产生的副本
}

void test02() {
    Person p_new; // 调用无参构造
    
    doWork(p_new); //就在这一瞬间，调用了拷贝构造函数！
}
```

3. 以值方式返回局部对象

场景：函数内部创建了一个对象，然后把它`return`出来。

```c++
Person doSomething() {
    Person p1; // 局部对象
    return p1; // 理论上，这里会调用拷贝构造，克隆一个新的对象返回给外面
}

void test03() {
    Person p = doSomething(); // 接住返回的对象
}
```

4. 示例

```c++
class Person {
public:
	Person() {
		cout << "无参构造函数!" << endl;
		mAge = 0;
	}
	Person(int age) {
		cout << "有参构造函数!" << endl;
		mAge = age;
	}
	Person(const Person& p) {
		cout << "拷贝构造函数!" << endl;
		mAge = p.mAge;
	}
	//析构函数在释放内存之前调用
	~Person() {
		cout << "析构函数!" << endl;
	}
public:
	int mAge;
};

//1. 使用一个已经创建完毕的对象来初始化一个新对象
void test01() {

	Person man(100); //p对象已经创建完毕
	Person newman(man); //调用拷贝构造函数
	Person newman2 = man; //拷贝构造

	//Person newman3;
	//newman3 = man; //不是调用拷贝构造函数，赋值操作
}

//2. 值传递的方式给函数参数传值
//相当于Person p1 = p;
void doWork(Person p1) {}
void test02() {
	Person p; //无参构造函数
	doWork(p);
}

//3. 以值方式返回局部对象
Person doWork2()
{
	Person p1;
	cout << (int *)&p1 << endl;
	return p1;
}

void test03()
{
	Person p = doWork2();
	cout << (int *)&p << endl;
}


int main() {

	//test01();
	//test02();
	test03();

	return 0;
}
```

### 构造函数调用规则

默认情况下，编译器非常大方，会免费送你几个函数。但一旦你自己动手写了某个构造函数，编译器就会认为“你是个行家，不需要我帮忙了”，从而收回部分赠品。

1. 默认情况

写一个空类，什么构造函数都不写：

```c++
class Person {
    // 啥也没写
};
```

> 编译器会免费赠送 3 个函数

- 默认构造函数：无参，函数体为空

```c++
Person() {}
```

- 析构函数：无参，函数体为空

```c++
~Person() {}
```

- 默认拷贝构造函数：对属性进行值拷贝

```c++
Person(const Person &p) { ... }
```

> 结果：你可以随便写 `Person p;` 或者 `Person p2(p1);`，都没问题。

2. 写了“有参构造”，收回“无参构造”

- 如果用户定义有参构造函数，c++不在提供默认无参构造，但是会提供默认拷贝构造

```c++
class Person {
public:
    // 你只写了一个有参构造
    Person(int age) { 
        cout << "有参" << endl; 
    }
};

int main() {
    Person p1(18); //正确：调用你自己写的有参构造

    // Person p2;  //报错！
    // 原因：编译器不再赠送无参构造了，而你自己又没写。
    
    Person p3(p1); //正确：编译器依然赠送默认拷贝构造。
}
```

3. 写了“拷贝构造”，全部收回

- 如果用户定义拷贝构造函数，c++不会再提供其他构造函数

```c++
class Person {
public:
    // 你只写了一个拷贝构造
    Person(const Person &p) {
        cout << "拷贝" << endl;
    }
};

int main() {
    // Person p1;      //错！无参被收回了。
    // Person p2(10);  //报错！有参你也从来没写过。
    
    // 此时你陷入了僵局：你甚至无法创建出第一个对象p1，
    // 所以你也无法使用拷贝构造来创建 p3。
    // 这代码写得就没法用了。
}
```

4. 示例

```c++
class Person {
public:
	//无参（默认）构造函数
	Person() {
		cout << "无参构造函数!" << endl;
	}
	//有参构造函数
	Person(int a) {
		age = a;
		cout << "有参构造函数!" << endl;
	}
	//拷贝构造函数
	Person(const Person& p) {
		age = p.age;
		cout << "拷贝构造函数!" << endl;
	}
	//析构函数
	~Person() {
		cout << "析构函数!" << endl;
	}
public:
	int age;
};

void test01()
{
	Person p1(18);
	//如果不写拷贝构造，编译器会自动添加拷贝构造，并且做浅拷贝操作
	Person p2(p1);

	cout << "p2的年龄为： " << p2.age << endl;
}

void test02()
{
	//如果用户提供有参构造，编译器不会提供默认构造，会提供拷贝构造
	Person p1; //此时如果用户自己没有提供默认构造，会出错
	Person p2(10); //用户提供的有参
	Person p3(p2); //此时如果用户没有提供拷贝构造，编译器会提供

	//如果用户提供拷贝构造，编译器不会提供其他构造函数
	Person p4; //此时如果用户自己没有提供默认构造，会出错
	Person p5(10); //此时如果用户自己没有提供有参，会出错
	Person p6(p5); //用户自己提供拷贝构造
}

int main() {

	test01();

	system("pause");

	return 0;
}
```

### 深拷贝与浅拷贝

- 浅拷贝：简单的赋值拷贝操作
- 深拷贝：在堆区重新申请空间，进行拷贝操作

1. 概念理解

假设存在对象 A，其内部包含一个指针成员，该指针指向堆区中一块已分配的内存。现在需要通过复制对象 A，创建一个新对象 B，两种拷贝方式的核心区别如下：

- 浅拷贝：
  - 动作：仅复制对象 A 中的指针变量本身，但未复制指针指向的堆区内存—— 新对象 B 的指针，依然指向 A 所对应的那块堆区内存地址。
  - 结果：对象 A 和 B 共享同一块堆区内存资源，二者的指针本质是 “指向同一目标的两把钥匙”。
  - 隐患：若对象 A 先释放了其指向的堆区内存，此时对象 B 的指针就变成了 “野指针”，再通过 B 的指针访问内存会触发非法访问错误；若 B 后续也尝试释放该内存，会导致内存重复释放，进而引发程序崩溃或内存泄漏。

- 深拷贝：
  - 动作：不仅复制对象 A 中的指针变量，还会在堆区重新分配一块与原内存大小相同、内容完全一致的新内存，最后将新内存的地址赋值给对象 B 的指针。
  - 结果：对象 A 和 B 各自持有独立的堆区内存资源，二者的指针指向完全不同的内存地址，相互独立无关联。
  - 优点：后续 A 对自己的堆区内存进行修改、释放，或 B 对自己的内存做同样操作，都不会影响对方的资源。彻底避免了浅拷贝的安全隐患，保证了对象的独立性和内存操作的安全性。

2. 示例：浅拷贝与深拷贝的区分

```c++
#include <iostream>
using namespace std;

class Person {
public:
    int * m_Height; // 指针成员，指向堆区

    // 构造函数
    Person(int height) {
        cout << "构造函数调用" << endl;
        m_Height = new int(height); // 在堆区开辟内存
    }

    //错误示例：默认的浅拷贝构造函数（编译器自动生成）
    // 问题：只复制指针地址，不复制实际数据
    // 会导致两个对象指向同一块内存，析构时重复释放！
    /*
    Person(const Person &p) {
        m_Height = p.m_Height; // 危险！共享同一块内存
    }
    */

    //正确示例：深拷贝构造函数
    Person(const Person &p) {
        cout << "深拷贝构造函数调用" << endl;
        // 重新在堆区申请内存，并复制数据
        m_Height = new int(*p.m_Height); // 关键：new新内存 + 复制数据
    }

    // 析构函数
    ~Person() {
        cout << "析构函数调用" << endl;
        // 释放堆区内存
        if (m_Height != NULL) {
            delete m_Height;
            m_Height = NULL;
        }
        cout << "堆区内存已释放" << endl;
    }
};

int main() {
    Person p1(180);
    
    // 调用拷贝构造函数
    Person p2(p1); 

    cout << "p1的身高地址: " << p1.m_Height << endl;
    cout << "p2的身高地址: " << p2.m_Height << endl;
    cout << "p1的身高值: " << *p1.m_Height << endl;
    cout << "p2的身高值: " << *p2.m_Height << endl;

    // ❌ 如果使用浅拷贝，程序结束时会出现：
    // 1. p2先析构，释放堆内存
    // 2. p1再析构，试图释放同一块内存 → 程序崩溃！

    return 0;
}
```

**浅拷贝的危险**：

```c++
//错误：共享内存
m_Height = p.m_Height;
```

**深拷贝的正确做法**：

```c++
//正确：独立内存
m_Height = new int(*p.m_Height);
```

3. 示例

```c++
class Person {
public:
	//无参（默认）构造函数
	Person() {
		cout << "无参构造函数!" << endl;
	}
	//有参构造函数
	Person(int age ,int height) {
		
		cout << "有参构造函数!" << endl;

		m_age = age;
		m_height = new int(height);
		
	}
	//拷贝构造函数  
	Person(const Person& p) {
		cout << "拷贝构造函数!" << endl;
		//如果不利用深拷贝在堆区创建新内存，会导致浅拷贝带来的重复释放堆区问题
		m_age = p.m_age;
		m_height = new int(*p.m_height);
		
	}

	//析构函数
	~Person() {
		cout << "析构函数!" << endl;
		if (m_height != NULL)
		{
			delete m_height;
		}
	}
public:
	int m_age;
	int* m_height;
};

void test01()
{
	Person p1(18, 180);

	Person p2(p1);

	cout << "p1的年龄： " << p1.m_age << " 身高： " << *p1.m_height << endl;

	cout << "p2的年龄： " << p2.m_age << " 身高： " << *p2.m_height << endl;
}

int main() {

	test01();

	return 0;
}
```

4. 什么时候需要深拷贝

- 如果你的类属性里没有指针（只有 `int`, `string` 等），浅拷贝（编译器送的）完全够用。 
- 如果你的类属性里有指针，并且你在构造函数里 `new` 了内存，那你必须自己写深拷贝！

>总结：如果属性有在堆区开辟的，一定要自己提供拷贝构造函数，防止浅拷贝带来的问题

### 初始化列表

- C++提供了初始化列表语法，用来初始化属性
- 针对构造函数

```c++
构造函数() : 属性1(值1), 属性2(值2)... { 
    // 函数体内部可以空着
}
```

1. 写法对比

传统写法

```c++
Person(int a, int b) {
    m_A = a;
    m_B = b;
}
```

初始化列表

```c++
Person(int a, int b) : m_A(a), m_B(b) {
    // 这里的 {} 可以是空的，因为外面已经干完活了
}
```

2. 为什么要用

- 初始化列表在成员（尤其庞大类对象）创建时直接用指定值完成初始化，无需经历 “默认构造 + 赋值” 两步，效率更优；简单类型差距不明显，复杂类对象的效率差异会十分突出。

- 有两类数据，只能初始化，不能赋值：

  - 引用 (`int &a`)：引用必须在定义时绑定，不能先定义再赋值。


  - 常量 (`const int a`)：常量必须在定义时给值，之后绝不能改。

```c++
class Person {
    int &m_Ref;
    const int m_Const;

    //在它们出生的一瞬间就给值
    Person(int &r, int c) : m_Ref(r), m_Const(c) { 
        // 函数体空着就行
        //m_Ref = r; 
        //m_Const = c;
        //报错，不能被修改
    }
};
```

3. 示例

```c++
class Person {
public:

	////传统方式初始化
	//Person(int a, int b, int c) {
	//	m_A = a;
	//	m_B = b;
	//	m_C = c;
	//}

	//初始化列表方式初始化
	Person(int a, int b, int c) :m_A(a), m_B(b), m_C(c) {}
	void PrintPerson() {
		cout << "mA:" << m_A << endl;
		cout << "mB:" << m_B << endl;
		cout << "mC:" << m_C << endl;
	}
private:
	int m_A;
	int m_B;
	int m_C;
};

int main() {

	Person p(1, 2, 3);
	p.PrintPerson();


	system("pause");

	return 0;
}
```

>初始化列表的顺序，永远和成员变量声明的顺序保持一致

### 类对象作为类成员

在 C++ 中，我们不仅可以用 `int`、`string` 做成员变量，还可以用一个类对象做另一个类的成员变量。

C++类中的成员可以是另一个类的对象，我们称该成员为对象成员

```c++
class A {}
class B
{
    A a；
}
```

B类中有对象A作为成员，A为对象成员

那么当创建B对象时，A与B的构造和析构的顺序是谁先谁后？

1. 概念理解

假设我们有两个类：

- 零件类：Phone (手机)

- 整体类：Person (人)


`Person` 类里面包含了一个 `Phone` 类的对象作为成员。

```c++
class Phone {
    // ...
};

class Person {
public:
    string m_Name;
    Phone m_Phone; //类对象作为成员
};
```

2. 构造与析构的顺序

思考：当创建一个 `Person` 对象时，是人先出生，还是手机先造好？ 当销毁一个 `Person` 对象时，是人先挂掉，还是手机先报废？

- 构造顺序：先里后外
  - 先构造成员对象 (Phone)，再构造自身 (Person)
- 析构顺序：先外后里。

  - 先析构自身 (Person)，再析构成员对象 (Phone)。

3. 示例：验证顺序

```c++
#include <iostream>
#include <string>
using namespace std;

// --- 零件：手机类 ---
class Phone {
public:
    string m_PName;

    Phone(string pName) {
        m_PName = pName;
        cout << "Phone 构造函数 (手机造好了)" << endl;
    }

    ~Phone() {
        cout << "Phone 析构函数 (手机报废了)" << endl;
    }
};

// --- 整体：人类 ---
class Person {
public:
    string m_Name;
    Phone m_Phone; // 成员对象

    //初始化列表
    // m_Phone(pName) 相当于调用了 Phone 的有参构造函数
    Person(string name, string pName) : m_Name(name), m_Phone(pName) {
        cout << "Person 构造函数 (人出生了)" << endl;
    }

    ~Person() {
        cout << "Person 析构函数 (人挂了)" << endl;
    }
};

void test() {
    Person p("张三", "iPhone 15");
    // 函数结束，栈区对象 p 会销毁
}

int main() {
    test();
    return 0;
}
```

运行结果

```
Phone 构造函数 (手机造好了)
Person 构造函数 (人出生了)
Person 析构函数 (人挂了)
Phone 析构函数 (手机报废了)
```

3. 为什么要用初始化列表

如果在上面的代码中，`Person` 的构造函数你不用初始化列表，而是写在 `{}` 里面：

```c++
Person(string name, string pName) {
    m_Name = name;
    // m_Phone = pName; // 试图赋值
}
```

效率低：编译器会先调用 `Phone` 的默认无参构造，然后再调用赋值操作符把数据改进去。相当于造了两次。

报错：如果 `Phone` 类没有默认无参构造函数（只写了有参构造），上面这种写法直接编译不通过！

>当类中有对象成员时，必须使用初始化列表来初始化它

4. 示例

```c++
class Phone
{
public:
	Phone(string name)
	{
		m_PhoneName = name;
		cout << "Phone构造" << endl;
	}

	~Phone()
	{
		cout << "Phone析构" << endl;
	}
	string m_PhoneName;
};


class Person
{
public:
	//初始化列表可以告诉编译器调用哪一个构造函数
	Person(string name, string pName) :m_Name(name), m_Phone(pName)
	{
		cout << "Person构造" << endl;
	}

	~Person()
	{
		cout << "Person析构" << endl;
	}

	void playGame()
	{
		cout << m_Name << " 使用" << m_Phone.m_PhoneName << " 牌手机! " << endl;
	}

	string m_Name;
	Phone m_Phone;
};
void test01()
{
	//当类中成员是其他类对象时，我们称该成员为 对象成员
	//构造的顺序是 ：先调用对象成员的构造，再调用本类构造
	//析构顺序与构造相反
	Person p("张三" , "苹果X");
	p.playGame();

}


int main() {
	test01();
	return 0;
}
```

### 静态成员

静态成员就是在成员变量和成员函数前加上关键字`static`，称为静态成员

它是属于“类”的，而不是属于某个具体“对象”的。

1. 普通变量与静态变量

- 普通成员变量
  - 属于类的「对象级成员」，每个类实例（对象）都会拥有该成员的**独立副本**，成员的生命周期与所属对象一致。

- 静态成员变量

  - 属于类的「类级成员」，不属于任何单个对象，整个类仅维护该成员的一份**全局共享副本**，成员的生命周期与程序运行周期一致。


2. 静态成员变量

- 所有对象共享同一份数据。
- 在编译阶段分配内存。

- 类内声明，类外初始化。

基本语法：

Step1.类内声明

```c++
//static 数据类型 变量名;
static int m_A;
```

Step2.类外初始化

```c++
//类型 类名::变量名 = 值;
int Person::m_A = 100;
```

Step3.访问方式

```c++
//类名 ::变量名
Person::m_A
```

示例：

```c++
#include <iostream>
using namespace std;

class Person {
public:
    // 静态成员变量 - 类内声明
    static int m_A; 
    
    // 普通成员变量
    int m_B;
};

// 静态成员变量 - 类外初始化
int Person::m_A = 100; 

void test01() {
    Person p1;
    p1.m_A = 200; // 通过对象修改静态变量

    Person p2;
    // 所有对象共享同一个静态变量
    cout << "p1.m_A = " << p1.m_A << endl; //200
    cout << "p2.m_A = " << p2.m_A << endl; //200

    // 推荐访问方式：通过类名直接访问
    cout << "Person::m_A = " << Person::m_A << endl;//200
}

int main() {
    test01();
    return 0;
}
```

3. 静态成员函数

- 所有对象共享同一个函数。
- 不需要创建对象也能调用。

- 只能访问静态成员变量。

```c++
class Person {
public:
    static int m_A; // 静态成员变量
    int m_B;        // 普通成员变量

    // 静态成员函数
    static void func() {
        m_A = 100; //可以访问静态成员变量
        
        // m_B = 200; //错误：不能访问非静态成员
        // 原因：静态函数没有this指针，无法确定访问哪个对象的m_B
        
        cout << "静态函数调用" << endl;
    }
};

int Person::m_A = 0;

int main() {
    // 通过对象调用静态函数
    Person p;
    p.func();

    // 推荐方式：通过类名直接调用
    Person::func();

    return 0;
}
```

## 对象模型与 this 指针

### 成员变量和成员函数分开存储

在C++中，类内的成员变量和成员函数分开存储

> 只有非静态成员变量才属于类的对象上

1. 定义

在 C++ 中，类的成员变量属于 “实例数据”，每个对象会独立分配内存存储自身的成员变量；而成员函数属于 “类级别的共享代码段”，所有对象共享同一份成员函数的二进制代码，存储在程序的 “代码区”，不会随对象数量增加而重复分配内存。

2. 原因

若成员函数随每个对象实例化重复存储，会导致 “代码冗余” 和 “内存膨胀”—— 对象数量越多，重复代码占用的内存空间呈线性增长，严重降低内存利用率。因此 C++ 采用 “数据独有、代码共享” 的存储策略，平衡对象独立性与内存效率。

- 成员变量= 每个对象的 “私有数据”，必须每户独立有，所以每个对象单独存；

- 成员函数= 所有对象的 “公共操作逻辑”，没必要每户存一份，所以所有对象共用同一份代码，存在 “公共代码区”。


> C++ 通过`this` 指针实现 “共享函数访问独有数据”：当对象调用成员函数时，编译器会隐式传递该对象的地址（this 指针）给函数，让函数知道 “当前要操作哪个对象的成员变量”。

```c++
#include <iostream>
using namespace std;

// 空类
class A {
};

// 只有成员变量
class B {
    int m_A; // 4字节
};

// 成员变量 + 成员函数
class C {
    int m_A; // 4字节
    
    // 成员函数不占对象空间
    void func() {
        cout << "我是函数" << endl;
    }
};

// 成员变量 + 静态成员变量
class D {
    int m_A;        // 4字节
    static int m_B; // 静态成员在全局区，不占对象空间
};

int main() {
    A a;
    B b;
    C c;
    D d;

    cout << "空类 A 的大小: " << sizeof(a) << endl; 
    cout << "只有变量 B 的大小: " << sizeof(b) << endl; 
    cout << "变量+函数 C 的大小: " << sizeof(c) << endl; 
    cout << "带静态变量 D 的大小: " << sizeof(d) << endl; 

    return 0;
}
```

```
空类 A 的大小: 1
只有变量 B 的大小: 4
变量+函数 C 的大小: 4
带静态变量 D 的大小: 4
```

### this指针

1. 定义

> `this` 指针指向被调用的成员函数所属的对象

`this` 指针是 C++ 中非静态成员函数内的隐式常量指针，本质是成员函数的 “隐藏参数”（由编译器自动添加并传递），其指向当前调用该成员函数的对象实例（即 “谁调用成员函数，this 就指向谁”）。它的类型为：

- 普通成员函数：`类名* const`
- const 成员函数：`const 类名* const`；仅在成员函数内部有效。

2. 特性

- 隐式存在，自动传递


编译器会在成员函数的参数列表中隐式添加`this`指针参数，调用函数时自动将对象地址传入

- 指向当前对象，绑定调用者


`this`指针指向触发成员函数调用的那个对象，确保共享的成员函数能精准操作该对象的成员变量

- 常量性限制

普通成员函数中`this`是`类名* const`（指针指向不可改），const 成员函数中`this`是`const 类名* const`（指针指向和对象数据都不可改）。

3. 作用

解决 “共享成员函数如何区分操作哪个对象的成员变量” 的问题，实现成员函数对特定对象数据的精准访问。

4. 示例：编译原理

```c++
#include <iostream>

/**************  常规写法（编译器自动添加 this） **************/
class Person {
public:
    int age;
    
    // 编译器自动添加 Person* const this 参数
    void setAge(int age) {          
        this->age = age;  // 使用 this 区分成员变量和参数
    }
};

/**************  手动展开 this（展示底层原理） **************/
class PersonRaw {
public:
    int age;
};

// 将成员函数转换为普通函数，显式添加 this 参数
void setAge(PersonRaw* const this_ptr, int age) {
    this_ptr->age = age;
}

int main() {
    /*---- 常规写法 ----*/
    Person p;
    p.setAge(18);               // 编译器自动传递 &p 作为 this
    std::cout << "p.age = " << p.age << '\n';

    /*---- 手动展开写法 ----*/
    PersonRaw pr;
    setAge(&pr, 18);            // 显式传递对象地址作为 this
    std::cout << "pr.age = " << pr.age << '\n';

    return 0;
}
```

5. 用途一：解决名称冲突

当**形参**和**成员变量**同名时，C++ 遵循“就近原则”，会优先使用形参。

```c++
class Person {
public:
    int name; // 成员变量

    //报错：参数名与成员变量冲突
    Person(int name) {
        name = name; // 实际是参数自己赋值给自己
    }

    //正确：使用 this 区分
    Person(int name) {
        this->name = name; // this->name 明确指向成员变量
    }
};
```

6. 用途二：支持链式编程

这是 C++ 高级编程（如 `cout` 的原理）的基础。 如果一个函数**返回对象本身**，就可以无限地在后面点点点（`.`）。

- `this` 是指针（地址）。
- `*this` 是解引用后的对象本体。

```c++
#include <iostream>
using namespace std;

class Person {
public:
    int age;

    Person(int age) {
        this->age = age;
    }

    // 返回 Person& 引用以支持链式调用
    Person& addAge(int num) {
        this->age += num;
        
        // 返回当前对象的引用，保持链式调用
        return *this; 
    }
};

int main() {
    Person p(10);

    // 链式调用：每次调用都返回对象本身
    p.addAge(10).addAge(10).addAge(10);

    cout << "最终年龄: " << p.age << endl; // 40
    
    return 0;
}
```

7. 综合示例

```c++
class Person
{
public:

	Person(int age)
	{
		//1、当形参和成员变量同名时，可用this指针来区分
		this->age = age;
	}

	Person& PersonAddPerson(Person p)
	{
		this->age += p.age;
		//返回对象本身
		return *this;
	}

	int age;
};

void test01()
{
	Person p1(10);
	cout << "p1.age = " << p1.age << endl;

	Person p2(10);
	p2.PersonAddPerson(p1).PersonAddPerson(p1).PersonAddPerson(p1);
	cout << "p2.age = " << p2.age << endl;
}

int main() {

	test01();
	return 0;
}
```

### const 修饰成员函数

1. 常函数与常对象

- 常函数：

  - 成员函数后加`const`后我们称为这个函数为常函数

  - 常函数内不可以修改成员属性

  - 成员属性声明时加关键字`mutable`后，在常函数中依然可以修改

- 常对象：

  - 声明对象前加`const`称该对象为常对象

  - 常对象只能调用常函数

2. 定义

`const` 关键字必须写在函数声明的参数列表后面，分号前面。

不允许这个函数修改对象内部的任何成员变量

```c++
class Person {
public:
    int m_Age;

    // 这是一个常函数
    void showAge() const {
        // m_Age = 100; //报错，不允许修改
        cout << m_Age << endl; //读是可以的
    }
};
```

3. 为什么需要它

>配合常对象使用

- 普通对象可以调用普通函数和常函数。
- 常对象只能调用常函数。

```c++
class Person {
public:
    void func1() {
        cout << "我是普通函数" << endl;
    }

    void func2() const {
        cout << "我是常函数" << endl;
    }
};

int main() {
    // 普通对象：可以调用所有函数
    Person p1;
    p1.func1();  // 允许
    p1.func2();  // 允许

    // 常对象：只能调用常函数
    const Person p2;
    
    // p2.func1(); //错误：常对象不能调用普通函数
    p2.func2();    //正确：常对象可以调用常函数

    return 0;
}
```

>成员函数逻辑上只读不写，请务必加上 const

4. mutable 关键字

思考：

- 这个函数在逻辑上确实是只读的。
- 但我想在函数里顺便记录一下“被调用的次数”。

```c++
class Person {
public:
    int m_Age;
    
    // mutable 修饰的变量在常函数中也可修改
    mutable int m_CallCount = 0; 

    void showInfo() const {
        // m_Age = 100; //错误：常函数不能修改普通成员变量
        
        m_CallCount++; //正确：可以修改 mutable 成员变量
        cout << "Age: " << m_Age << endl;
    }
};

int main() {
    const Person p;
    p.showInfo(); // 第一次调用
    p.showInfo(); // 第二次调用
    
    // 常对象的 mutable 成员仍然可以被修改
    cout << "被调用次数: " << p.m_CallCount << endl; // 输出 2
    
    return 0;
}
```

5. 示例

```c++
class Person {
public:
	Person() {
		m_A = 0;
		m_B = 0;
	}

	//this指针的本质是一个指针常量，指针的指向不可修改
	//如果想让指针指向的值也不可以修改，需要声明常函数
	void ShowPerson() const {
		//const Type* const pointer;
		//this = NULL; //不能修改指针的指向 Person* const this;
		//this->mA = 100; //但是this指针指向的对象的数据是可以修改的

		//const修饰成员函数，表示指针指向的内存空间的数据不能修改，除了mutable修饰的变量
		this->m_B = 100;
	}

	void MyFunc() const {
		//mA = 10000;
	}

public:
	int m_A;
	mutable int m_B; //可修改 可变的
};


//const修饰对象  常对象
void test01() {

	const Person person; //常量对象  
	cout << person.m_A << endl;
	//person.mA = 100; //常对象不能修改成员变量的值,但是可以访问
	person.m_B = 100; //但是常对象可以修改mutable修饰成员变量

	//常对象访问成员函数
	person.MyFunc(); //常对象不能调用const的函数

}

int main() {

	test01();
	return 0;
}
```

### 空指针访问成员函数

C++中空指针也是可以调用成员函数的，但是也要注意有没有用到this指针

如果用到this指针，需要加以判断保证代码的健壮性

1. 现象与思考

```c++
#include <iostream>
using namespace std;

class Person {
public:
    int m_Age;

    // 函数1：不访问成员变量
    void showClassName() {
        cout << "我是 Person 类！" << endl;
    }

    // 函数2：访问成员变量 m_Age
    void showAge() {
        cout << "年龄是：" << m_Age << endl;
    }
};

int main() {
    // 创建空指针
    Person* p = nullptr;

    // 调用不访问成员变量的函数 - 可能不会崩溃
    p->showClassName(); //可能正常运行

    // 调用访问成员变量的函数 - 一定会崩溃
    p->showAge();       //程序崩溃（访问空指针）

    return 0;
}
```

>现象：访问成员变量的函数程序崩溃了；反之没有

2. 原理

空指针（`nullptr`）调用类的成员函数时：

- 若函数未访问成员变量 /`this`指针，则程序可正常执行；
- 若函数访问成员变量 / 间接使用`this`指针，则触发 “空指针解引用” 错误（Segmentation Fault / 访问违规）。

这一现象源于 C++ 的对象模型与`this`指针的传递机制。

原因：

- 成员函数的存储特性决定 “调用可行性”

成员函数属于类的共享代码段（存储于程序的代码区），与对象实例的存在性无关 —— 无论对象是否为空，成员函数的二进制指令始终存在且地址

可被编译器解析。因此空指针调用 “无成员变量访问” 的成员函数时，编译器仅需跳转到函数的代码地址执行，无需依赖对象实例。

- `this`指针的空值导致 “访问成员变量崩溃”

成员函数中访问成员变量时，编译器会隐式转换为`this->成员变量`的形式（`this`指针指向调用函数的对象实例）。若调用者是`nullptr`，则`this`指针为空；对空指针进行解引用（`this->m_Age`）属于未定义行为，会触发内存访问错误。

>空指针调用成员函数的崩溃与否，取决于函数是否通过this指针访问对象的非静态成员数据 —— 成员函数的执行不依赖对象实例，但成员变量的访问必须通过有效this指针（指向合法对象）。

本质：

```c++
void showAge() {
    // 编译器看到的其实是：
    cout << "年龄是：" << this->m_Age << endl;
}
```

3. 如何让代码更健壮

在成员函数中对`this`指针进行有效性校验，是针对 “空指针调用成员函数” 场景的防御性编程手段 —— 通过在函数执行核心逻辑前检查`this`是否为`nullptr`，提前拦截非法调用，避免空指针解引用导致的程序崩溃，提升代码的健壮性。

```c++
void showAge() {
    //在开头加上这个代码，进行检验
    if (this == nullptr) {
        cout << "警告：这是个空指针，无法打印年龄！" << endl;
        return; // 直接返回，防止崩盘
    }

    cout << "年龄是：" << m_Age << endl;
}
```

4. 示例

```c++
//空指针访问成员函数
class Person {
public:

	void ShowClassName() {
		cout << "我是Person类!" << endl;
	}

	void ShowPerson() {
		if (this == NULL) {
			return;
		}
		cout << mAge << endl;
	}

public:
	int mAge;
};

void test01()
{
	Person * p = NULL;
	p->ShowClassName(); //空指针，可以调用成员函数
	p->ShowPerson();  //但是如果成员函数中用到了this指针，就不可以了
}

int main() {

	test01();
	return 0;
}
```

## 友元

1. 定义

友元是 C++ 中对类封装特性的局部放宽机制：通过`friend`关键字，允许指定的外部全局函数、其他类或其他类的成员函数，直接访问当前类的 私有（private）和保护（protected）成员（这些成员默认仅类内部可访问）。本质是在保证封装性的前提下，为特定场景提供灵活的跨边界访问权限。

>让一个函数或者类，访问另一个类中私有 (private) 的成员

核心目的：解决 “特定外部实体需高效访问类私有成员” 的场景（如运算符重载、组件协作），避免为了访问私有成员而过度暴露 `public` 接口，平衡封装性与访问效率。

2. 友元有三种实现形式：

- 全局函数做友元
- 类做友元
- 成员函数做友元

3. 友元的 “三不原则”

- 单向性：
  - 友元关系是单向绑定的 —— 若 A 声明 B 为友元，B 可访问 A 的私有成员，但 A 不会自动获得访问 B 私有成员的权限。
- 无传递性：
  - 友元关系不具备传递性 —— 若 A 是 B 的友元，B 是 C 的友元，A 不会因此成为 C 的友元，无法访问 C 的私有成员
- 不可继承性：
  - 基类的友元不会自动成为派生类的友元 —— 即使基类 A 声明 B 为友元，B 也无法访问 A 的派生类 D 的私有 / 保护成员。

### 全局函数做友元

1. 定义

在类的定义体内，通过`friend`关键字声明一个非成员的全局函数为当前类的友元，该全局函数因此获得直接访问类的私有、保护成员的权限 —— 即使它不属于类的成员函数，也能突破封装边界访问类的私密数据。

2. 为什么需要 “外人” 当友元

多数场景下，类的私有成员访问可通过`public`成员函数实现，但部分语法或逻辑需求（如特定运算符重载、跨类协作的简化操作）中，成员函数无法满足语法合法性或语义合理性，此时需借助全局函数做友元，在不破坏封装的前提下实现灵活访问。

3. 经典案例：`<<` 输出运算符重载

- 成员函数的局限性：


在C++ 中，运算符重载若以成员函数形式实现，运算符的**左操作数必须是当前类的对象**（因为成员函数的隐式`this`指针指向左操作数）。若把`<<`重载为`Person`类的成员函数，调用语法会变成`p << cout`（左操作数是`Person`对象`p`，右操作数是`cout`），这与`cout << p`的常规输出语法完全矛盾，违反用户使用习惯。

- 全局函数做友元的解决方案


将`operator<<`定义为全局函数，左操作数设为`ostream&`（对应`cout`），右操作数设为`Person&`（对应要输出的对象）；同时在`Person`类中声

明该全局函数为友元，使其能访问`Person`的私有成员（如`m_Name`、`m_Age`）。

```c++
#include <iostream>
using namespace std;

class Person {
private:
    string m_Name = "张三";
    int m_Age = 18;
    // 声明全局函数为友元
    friend ostream& operator<<(ostream& out, const Person& p);
};

// 全局函数实现<<重载
ostream& operator<<(ostream& out, const Person& p) {
    // 友元权限：直接访问私有成员
    out << "姓名：" << p.m_Name << "，年龄：" << p.m_Age;
    return out; // 支持链式输出（如cout << p << endl;）
}

int main() {
    Person p;
    cout << p; //输出：姓名：张三，年龄：18
    return 0;
}
```

4. 语法格式：三步走

实现全局函数做友元，只需要三步：

1. 定义类
2. 声明友元
3. 实现函数

- 步骤 1：定义类并封装私有成员：

```c++
class Person {
private:
    string m_Name = "李四"; // 私有数据
    int m_Age = 25;         // 私有数据
    // 后续声明友元函数
};
```

- 步骤 2：类内声明友元全局函数：

```c++
friend 返回值类型 函数名(参数列表);
```

通过`friend`关键字声明目标全局函数的原型

```c++
class Person {
private:
    string m_Name = "李四";
    int m_Age = 25;
public:
    // 声明友元全局函数
    friend void printPerson(const Person& p); 
};
```

- 步骤 3：全局域实现友元函数：

在类的外部（全局作用域）定义该函数，无需加`friend`关键字，也无需通过`类名::`限定（它不是类的成员函数），直接访问类的私有成员即可。

```c++
// 全局域实现友元函数
void printPerson(const Person& p) {
    // 直接访问Person的私有成员（有友元权限）
    cout << "姓名：" << p.m_Name << "，年龄：" << p.m_Age << endl;
}
```

- 综合：

```c++
#include <iostream>
#include <string>
using namespace std;

class Person {
private:
    string m_Name = "李四";
    int m_Age = 25;
public:
    friend void printPerson(const Person& p); // 步骤2：声明友元
};

void printPerson(const Person& p) { // 步骤3：实现友元函数
    cout << "姓名：" << p.m_Name << "，年龄：" << p.m_Age << endl;
}

int main() {
    Person p;
    printPerson(p); //输出“姓名：李四，年龄：25”
    return 0;
}
```

5. 高级应用：重载 cout

```c++
#include <iostream>
#include <string>
using namespace std;

class Person {
    // 声明全局 operator<< 为友元函数
    friend ostream& operator<<(ostream &out, Person &p);

private:
    string m_Name;
    int m_Age;

public:
    Person(string name, int age) : m_Name(name), m_Age(age) {}
};

// 全局函数实现
// 友元关系允许直接访问私有成员
ostream& operator<<(ostream &out, Person &p) {
    out << "姓名：" << p.m_Name << " 年龄：" << p.m_Age;
    return out;
}

int main() {
    Person p("张三", 18);
    cout << p << endl; // 直接输出对象
    
    return 0;
}
```

### 类做友元

1. 定义

类做友元是 C++ 友元机制的批量授权形式：在类 B（被授权类）的定义体内，通过声明类 A 为友元类，此时类 A 的所有成员函数均获得直接访问类 B 的私有和保护成员的权限 —— 相当于为类 A 的全体成员函数一次性授予访问类 B 私密成员的 “批量 VIP 权限”。

2. 核心机制

与 “全局函数做友元” 的单点授权不同，类做友元是对整个类的 “集体授权”，无需为类 A 的每个成员函数单独声明友元，即可让类 A 的所有函数突

破类 B 的封装边界访问其私有成员。

3. 语法格式

```c++
friend class 类名;
```

>注意代码顺序： 两个类互相引用，通常需要先做前置声明，或者把函数实现写在最后。

4. 代码示例

```c++
#include <iostream>
#include <string>
using namespace std;

// 前置声明
class Building;

// 友元类
class GoodGay {
public:
    Building* b;  // Building指针
    
    GoodGay();    // 构造函数
    void visit(); // 参观函数
    void sleep(); // 睡觉函数
};

// 房屋类
class Building {
    // 声明 GoodGay 为友元类
    friend class GoodGay;

public:
    string m_SittingRoom; // 客厅（公共）
private:
    string m_BedRoom;     // 卧室（私有）

public:
    Building();
};

// 类外实现成员函数

Building::Building() {
    m_SittingRoom = "客厅";
    m_BedRoom = "卧室";
}

GoodGay::GoodGay() {
    b = new Building; // 创建Building对象
}

void GoodGay::visit() {
    cout << "基友正在参观：" << b->m_SittingRoom << endl;
    cout << "基友正在参观：" << b->m_BedRoom << endl; // 友元可访问私有成员
}

void GoodGay::sleep() {
    cout << "基友正在：" << b->m_BedRoom << " 睡觉zzZ" << endl; // 友元可访问私有成员
}

int main() {
    GoodGay gg;
    gg.visit();
    gg.sleep();
    
    return 0;
}
```

### 成员函数做友元

1. 定义

成员函数做友元是 C++ 友元机制中精准授权的形式：在类 B（被授权类）的定义体内，通过声明类 A 的某个特定成员函数为友元，仅该成员函数可直接访问类 B 的私有和保护成员，类 A 的其他成员函数无此权限 —— 是遵循 “最小权限原则” 的精细化权限开放机制。

2. 核心机制

与 “类做友元” 的批量授权不同，成员函数做友元仅针对类 A 中的单个成员函数开放权限，其他函数仍受封装限制，避免不必要的权限泄露，符合

软件工程的 “最小权限原则”。

3. 语法格式

```c++
friend 返回值类型 朋友类名::函数名(参数列表);
```

>注意：必须加上作用域` ::`，告诉编译器这个函数是属于谁的

4. 易错：代码顺序

C++ 编译器是 “单遍编译”，且需要“完整的类型信息” 才能处理成员访问、函数调用等操作。

当两个类互相依赖，若顺序错乱，编译器会因 “找不到完整定义” 报错。

- 步骤 1：前置声明被依赖类（Building）
  - 在所有类定义前，写`class Building;`

- 步骤 2：定义授权类（GoodGay），仅声明成员函数
  - 定义 GoodGay 类，声明成员函数（如`void visit();`），但不写函数体。

- 步骤 3：定义被授权类（Building），声明友元
  - 定义 Building 类，在其中声明`friend void GoodGay::visit();`（精准授权 GoodGay 的 visit 函数）。
- 步骤 4：实现 GoodGay 的成员函数（写函数体）
  - 在 Building 类定义后，实现 GoodGay 的 visit 函数（如`void GoodGay::visit() { ... }`）

5. 示例

```c++
#include <iostream>
#include <string>
using namespace std;

// 前置声明
class Building;

// GoodGay类声明
class GoodGay {
public:
    Building* m_Building;
    GoodGay();
    void visit(); // 仅声明
};

// Building类定义
class Building {
    // 仅授权GoodGay::visit函数为友元
    friend void GoodGay::visit();
    
private:
    string m_BedRoom = "卧室";
public:
    string m_LivingRoom = "客厅";
};

// GoodGay成员函数实现
GoodGay::GoodGay() {
    m_Building = new Building;
}

void GoodGay::visit() {
    // 友元函数可访问私有成员
    cout << "访问：" << m_Building->m_BedRoom << endl;
    cout << "访问：" << m_Building->m_LivingRoom << endl;
}

int main() {
    GoodGay gg;
    gg.visit();
    return 0;
}
```

### 类内定义与类外定义

在 C++ 中，一个成员函数可以写在两个地方：

- **类内定义**：直接把代码写在 `class` 的花括号里。
- **类内声明，类外定义**：类里只写一行“目录”，具体的代码写在外面。

1. 定义

“类内声明，类外定义” 是 C++ 中类成员函数的分离式实现机制：在类的定义体内部（类内）仅声明成员函数的原型（包括函数名、返回值类型、参数列表），而将函数的具体实现逻辑（函数体）放在类的外部（类外）完成，通过`类名::`作用域解析运算符关联函数与所属类。

2. 语法格式

```c++
返回值类型 类名::函数名()
```

3. 代码对比

- 类内定义

```c++
class Person {
public:
    // 声明和定义在一起
    void eat() {
        cout << "我在吃饭" << endl; 
    }
};
```

- 类外定义

```c++
class Person {
public:
    // 1. 类内声明：只写函数头，加分号 ; 结束
    void eat(); 
};

// ... 可以在很远的地方 ...

// 2. 类外定义：写具体的代码
//重点：必须加 Person:: 告诉编译器这是谁的函数
void Person::eat() {
    cout << "我在吃饭" << endl;
}
```

## 运算符重载

1. 定义

运算符重载是 C++ 中赋予已有运算符新语义的机制：允许用户为自定义类型重新定义运算符的行为，使得运算符能像作用于`int`、`float`等内置类型一样，直观地作用于自定义类型。本质上，运算符重载是一种特殊的函数重载，其函数名固定为`operator`后紧跟要重载的运算符（如`operator+`、`operator<<`）。

运算符重载的本质是将运算符转化为**成员函数或全局函数**—— 当编译器遇到`a + b`（`a`、`b`为自定义类型）时，会自动调用`a.operator+(b)`（成员函数重载）或`operator+(a, b)`（全局函数重载），执行用户定义的逻辑。

2. 语法格式

`operator` 关键字

实现运算符重载，本质上就是写一个函数。 只是这个函数的名字比较特殊，叫：`operator` + `符号`。

```c++
返回值类型 operator符号(参数列表) {
    // 具体的运算逻辑
}
```

- 重载加号：`operator+`
- 重载等于：`operator==`
- 重载输出：`operator<<`

3. 四不准

- 不能创造新运算符
- 不能改变优先级
- 不能改变操作数个数
- 不准重载的符号：`.`、`::`、`sizeof`、`?:`、`.*`

4. 成员函数重载与全局函数重载

- 成员函数版：`p1 + p2` $\Leftrightarrow$ `p1.operator+(p2)`
  - 限制：加号左边（发起者）必须是你的类对象
- 全局函数版：`p1 + p2` $\Leftrightarrow$ `operator+(p1, p2)`
  - 优势：加号左边可以是任何东西（只要你定义了对应的参数）。

### 加号运算符重载

1. 对象相加

C++ 中`operator+`遵循值语义—— 即 “相加操作不修改原操作数，而是生成新对象承载结果”，这与内置类型的加法逻辑完全一致。对象相加本质是 “基于原对象的属性生成新对象” 的过程，而非对原对象的修改。

2. 成员函数重载

```c++
#include <iostream>
using namespace std;

class Person {
public:
    int m_A;
    int m_B;

    Person() {}; 
    Person(int a, int b) : m_A(a), m_B(b) {}

    // 重载 + 运算符
    Person operator+(const Person &p) {
        Person temp; 
        
        // 将两个对象的对应成员相加
        temp.m_A = this->m_A + p.m_A;
        temp.m_B = this->m_B + p.m_B;

        // 返回新对象（不能返回引用，因为temp是局部变量）
        return temp; 
    }
};

int main() {
    Person p1(10, 10);
    Person p2(20, 20);

    Person p3 = p1 + p2; // 调用 operator+
    
    cout << "p3.m_A = " << p3.m_A << endl; // 30
    cout << "p3.m_B = " << p3.m_B << endl; // 30
    
    return 0;
}
```

- 当你写 `p3 = p1 + p2` 时，编译器把它翻译成了： `Person p3 = p1.operator+(p2);`

3. 全局函数重载

```c++
class Person {
    // 声明全局运算符重载函数为友元
    friend Person operator+(const Person &p1, const Person &p2);
    
private:
    int m_A;
    int m_B;

public:
    Person() {}
    Person(int a, int b) : m_A(a), m_B(b) {}
};

// 全局函数实现运算符重载
Person operator+(const Person &p1, const Person &p2) {
    Person temp;
    temp.m_A = p1.m_A + p2.m_A; // 通过友元访问私有成员
    temp.m_B = p1.m_B + p2.m_B;
    return temp;
}
```

- 当你写 `p3 = p1 + p2` 时，编译器把它翻译成了： `Person p3 = operator+(p1, p2);`

4. 规则

- 命名法则：固定函数名`operator+`
  - C++ 语法规定，加号运算符重载的函数名必须为`operator+`
- 参数法则：优先使用`cons`t引用
  - 参数建议声明为`const 类名&`
- 返回值法则：必须返回 “对象本身”，禁止返回引用
  - 返回值需为类的实例（如Person），而非引用（Person&）。因为函数内创建的结果对象是局部变量，若返回引用会指向已销毁的内存（悬空引用），导致未定义行为；返回对象则通过值拷贝传递结果，保证内存安全。
- 参数数量法则：
  - 成员函数版：仅需 1 个参数（右操作数），左操作数由隐式的this指针指向（即调用函数的对象）；
  - 全局函数版：需 2 个参数（左、右操作数），因为全局函数无this指针，需显式传递两个操作数。
- 不可变性法则：不修改原操作数
  - 函数内严禁修改`this`指针指向的对象（左操作数）或参数对象（右操作数），需严格遵循值语义；修改原对象是operator+=（复合赋值）的职责，二者语义严格区分。

### 左移运算符重载

>让 cout 能直接打印你的对象

> 只能重载为全局函数，不能是成员函数

1. 语法格式

```c++
ostream& operator<<(ostream& out, const 类名& p) {
    out << "自定义输出格式";
    return out;
}
// 返回值：ostream& (为了支持链式编程)
// 参数1：ostream& out (代表 cout)
// 参数2：const 类名& p (我们要打印的对象)
```

2. 示例：

为了能访问类里的私有成员（通常属性都是私有的），这个全局函数必须是类的友元。

```c++
#include <iostream>
#include <string>
using namespace std;

class Person {
    // 声明友元函数，允许访问私有成员
    friend ostream& operator<<(ostream& out, const Person& p);

private:
    string m_Name;
    int m_Age;

public:
    Person(string name, int age) : m_Name(name), m_Age(age) {}
};

// 重载 << 运算符
ostream& operator<<(ostream& out, const Person& p) {
    // 自定义输出格式
    out << "姓名: " << p.m_Name << " | 年龄: " << p.m_Age;
    
    // 返回输出流对象以支持链式调用
    return out;
}

int main() {
    Person p("张三", 18);

    // 直接输出对象，支持链式调用
    cout << p << endl;

    return 0;
}
```

### 递增运算符重载

- 前置递增 (`++a`)：先加，再用
- 后置递增 (`a++`)：先用，再加

1. 占位参数

- 前置 (++p)：写法 `operator++()`


- 后置 (p++)：必须加一个 int 占位参数 `operator++(int)`

2. 前置递增

```c++
// 返回引用，为了一直操作本体
MyInteger& operator++() {
    // 1. 先加
    m_Num++;
    // 2. 再返回自己
    return *this; 
}
```

- 返回值：必须是引用

3. 后置递增

```c++
// 注意1：返回值是 MyInteger (不是引用)
// 注意2：参数里有个 int (占位)
MyInteger operator++(int) {
    // 1. 记录当前状态
    MyInteger temp = *this; 
    
    // 2. 只有我自己变了
    m_Num++; 
    
    // 3. 返回的是刚才记录的旧值
    return temp;
}
```

- 参数：必须加`int`占位。

- 返回值：必须是值 (`MyInteger`)，绝对不能是引用。

4. 示例

```c++
#include <iostream>
using namespace std;

class MyInteger {
    friend ostream& operator<<(ostream& out, MyInteger myint);

public:
    MyInteger() {
        m_Num = 0;
    }

    // 前置++：返回引用，支持链式操作
    MyInteger& operator++() {
        m_Num++;
        return *this;
    }

    // 后置++：int为占位参数，用于区分重载
    // 返回临时对象，不能返回引用
    MyInteger operator++(int) {
        MyInteger temp = *this; // 保存原值
        m_Num++;                // 递增
        return temp;            // 返回原值
    }

private:
    int m_Num;
};

// 重载输出运算符
ostream& operator<<(ostream& out, MyInteger myint) {
    out << myint.m_Num;
    return out;
}

int main() {
    MyInteger myint;

    // 测试前置++
    cout << "--- 前置++ ---" << endl;
    cout << ++myint << endl; // 1
    cout << myint << endl;   // 1

    // 测试后置++
    cout << "--- 后置++ ---" << endl;
    cout << myint++ << endl; // 1 (返回原值)
    cout << myint << endl;   // 2 (已递增)

    return 0;
}
```

### 赋值运算符重载

c++编译器至少给一个类添加4个函数

- 默认构造函数(无参，函数体为空)
- 默认析构函数(无参，函数体为空)
- 默认拷贝构造函数，对属性进行值拷贝
- 赋值运算符 operator=, 对属性进行值拷贝

如果类中有属性指向堆区，做赋值操作时也会出现深浅拷贝问题

```c++
#include <iostream>
using namespace std;

class Person {
public:
    int* m_Age; // 指针成员，指向堆区内存

    // 构造函数
    Person(int age) {
        m_Age = new int(age);
    }

    // 析构函数
    ~Person() {
        if (m_Age != nullptr) {
            delete m_Age;
            m_Age = nullptr;
        }
    }

    // 赋值运算符重载（深拷贝）
    Person& operator=(const Person& p) {
        // 1. 检查自赋值
        if (this == &p) {
            return *this;
        }

        // 2. 释放原有资源
        if (m_Age != nullptr) {
            delete m_Age;
            m_Age = nullptr;
        }

        // 3. 深拷贝新数据
        m_Age = new int(*p.m_Age);

        // 4. 返回自身引用以支持链式赋值
        return *this;
    }
};

int main() {
    Person p1(18);
    Person p2(20);
    Person p3(30);

    // 赋值操作
    p2 = p1; 

    cout << "p1年龄: " << *p1.m_Age << endl;
    cout << "p2年龄: " << *p2.m_Age << endl;

    // 链式赋值
    p3 = p2 = p1; 
    
    return 0;
}
```

- 标准写法

1. 判断自我赋值
2. 释放旧内存
3. 深拷贝
4. 返回引用

### 关系运算符重载

**作用：**重载关系运算符，可以让两个自定义类型对象进行对比操作

> 只负责比较，告诉你是真还是假。

1. 语法格式

通常把关系运算符写成成员函数。

三个固定要素：

- 返回值：必须是`bool`。

- 参数：必须是`const`引用 。

- 函数属性：建议加上`const`（常函数），因为比较操作不应该修改自身。

```c++
bool operator==(const 类名& p) const {
    // 比较逻辑
}
```

2. 示例

```c++
#include <iostream>
#include <string>
using namespace std;

class Person {
public:
    string name;
    int age;

    Person(string n, int a) : name(n), age(a) {}

    // 重载 == 运算符
    bool operator==(const Person& p) const {
        return (name == p.name && age == p.age);
    }

    // 重载 != 运算符（基于 == 实现）
    bool operator!=(const Person& p) const {
        return !(*this == p);
    }
};

int main() {
    Person p1("张三", 18);
    Person p2("张三", 18);
    Person p3("李四", 20);

    if (p1 == p2) {
        cout << "p1 和 p2 是同一个人" << endl;
    }

    if (p1 != p3) {
        cout << "p1 和 p3 不是同一个人" << endl;
    }

    return 0;
}
```

```c++
#include <iostream>
#include <string>
using namespace std;

class Person {
public:
    string name;
    int age;
    
    Person(string n, int a) : name(n), age(a) {}

    // 重载 < 运算符（按年龄比较）
    bool operator<(const Person& p) const {
        return age < p.age;
    }
};

int main() {
    Person p1("老人", 80);
    Person p2("小孩", 5);

    if (p2 < p1) {
        cout << p2.name << " 比 " << p1.name << " 年轻" << endl;
    }

    return 0;
}
```

```c++
class Person
{
public:
	Person(string name, int age)
	{
		this->m_Name = name;
		this->m_Age = age;
	};

	bool operator==(Person & p)
	{
		if (this->m_Name == p.m_Name && this->m_Age == p.m_Age)
		{
			return true;
		}
		else
		{
			return false;
		}
	}

	bool operator!=(Person & p)
	{
		if (this->m_Name == p.m_Name && this->m_Age == p.m_Age)
		{
			return false;
		}
		else
		{
			return true;
		}
	}

	string m_Name;
	int m_Age;
};

void test01()
{
	//int a = 0;
	//int b = 0;

	Person a("孙悟空", 18);
	Person b("孙悟空", 18);

	if (a == b)
	{
		cout << "a和b相等" << endl;
	}
	else
	{
		cout << "a和b不相等" << endl;
	}

	if (a != b)
	{
		cout << "a和b不相等" << endl;
	}
	else
	{
		cout << "a和b相等" << endl;
	}
}


int main() {

	test01();
	return 0;
}
```

### 函数调用运算符重载

- 函数调用运算符 () 也可以重载
- 由于重载后使用的方式非常像函数的调用，因此称为仿函数
- 仿函数没有固定写法，非常灵活

1. 语法格式

```c++
返回值类型 operator()(参数列表) {
    // 逻辑代码
}
```

>对象伪装成函数

2. 示例

```c++
#include <iostream>
#include <string>
using namespace std;

class MyPrint {
public:
    // 重载函数调用运算符 () - 创建仿函数
    void operator()(string test) {
        cout << "MyPrint输出: " << test << endl;
    }
};

int main() {
    MyPrint mp;

    // 对象可以像函数一样被调用 - 仿函数
    mp("Hello C++"); 
    // 等价于: mp.operator()("Hello C++");
    
    return 0;
}
```

```c++
#include <iostream>
using namespace std;

class MyAdd {
public:
    int count; // 内部状态：记录调用次数

    MyAdd() : count(0) {}

    // 重载函数调用运算符 - 仿函数
    int operator()(int v1, int v2) {
        count++; // 每次调用增加计数器
        return v1 + v2;
    }
};

int main() {
    MyAdd add; // 创建仿函数对象

    cout << "10 + 10 = " << add(10, 10) << endl;
    cout << "20 + 20 = " << add(20, 20) << endl;

    // 仿函数保持状态：记录调用次数
    cout << "加法器被调用了 " << add.count << " 次" << endl; 

    return 0;
}
```

```c++
class MyPrint
{
public:
	void operator()(string text)
	{
		cout << text << endl;
	}

};
void test01()
{
	//重载的（）操作符 也称为仿函数
	MyPrint myFunc;
	myFunc("hello world");
}


class MyAdd
{
public:
	int operator()(int v1, int v2)
	{
		return v1 + v2;
	}
};

void test02()
{
	MyAdd add;
	int ret = add(10, 10);
	cout << "ret = " << ret << endl;

	//匿名对象调用  
	cout << "MyAdd()(100,100) = " << MyAdd()(100, 100) << endl;
}

int main() {

	test01();
	test02();
	return 0;
}
```

## 继承

**继承是面向对象三大特性之一**

我们发现，定义这些类，下级别的成员除了拥有上一级的共性，还有自己的特性。

这个时候我们就可以考虑利用继承的技术，减少重复代码

### 继承的基本语法

```c++
class 子类 : 继承方式 父类 {
    // 子类独有的东西
};
```

- 父类= 基类

- 子类= 派生类

派生类中的成员，包含两大部分：

一类是从基类继承过来的，一类是自己增加的成员。

从基类继承过过来的表现其共性，而新增的成员体现了其个性。

```c++
#include <iostream>
using namespace std;

// 基类：通用页面组件
class BasePage {
public:
    void header() {
        cout << "公共头部：登录、注册、导航..." << endl;
    }
    
    void footer() {
        cout << "公共底部：联系我们、版权信息..." << endl;
    }
    
    void left() {
        cout << "公共侧边栏：Java, Python, C++..." << endl;
    }
};

// Java页面：继承BasePage
class JavaPage : public BasePage {
public:
    void content() {
        cout << "Java学科内容：视频教程、项目实战..." << endl;
    }
};

// Python页面：继承BasePage
class PythonPage : public BasePage {
public:
    void content() {
        cout << "Python学科内容：数据分析、机器学习..." << endl;
    }
};

int main() {
    cout << "--- Java 页面 ---" << endl;
    JavaPage java;
    java.header();   // 继承自基类
    java.left();     // 继承自基类
    java.content();  // 子类特有
    java.footer();   // 继承自基类

    cout << "\n--- Python 页面 ---" << endl;
    PythonPage py;
    py.header();     // 继承自基类
    py.left();       // 继承自基类
    py.content();    // 子类特有
    py.footer();     // 继承自基类

    return 0;
}
```

### 继承方式

继承方式一共有三种：

- 公共继承
- 保护继承
- 私有继承

1. 公共继承

公共继承在逻辑上代表了 “是一个 (Is-A)” 的关系。

- 猫继承动物 $\to$ 猫是一个动物。
- 学生继承人 $\to$ 学生是一个人。
- 特斯拉继承汽车 $\to$ 特斯拉是一辆汽车。

猫、学生、特斯拉都是`public`类

>凡是父类能做的事（Public 方法），子类也一定能做。

公共继承就像是一块“透明玻璃”。它不会改变父类成员原本的权限等级（除了 private）。

|  父类原本的权限  | 子类内部能访问吗？ | 子类对象（在 main 中）能访问吗？ |   继承后的最终权限   |
| :--------------: | :----------------: | :------------------------------: | :------------------: |
|  public (公开)   |         能         |                能                |  public (保持不变)   |
| protected (保护) |         能         |               不能               | protected (保持不变) |
|  private (私有)  |        不能        |               不能               |   不可访问 (隐藏)    |

> 父类对外公开的，子类也对外公开；父类传给家族的，子类也留给家族；父类私藏的，子类看不见。

示例：

```c++
#include <iostream>
#include <string>
using namespace std;

// 基类：Person
class Person {
public:
    string name;    // 公共成员：外部可访问
protected:
    int money;      // 保护成员：仅子类和自身可访问
private:
    int password;   // 私有成员：仅自身可访问

public:
    Person() {
        name = "某人";
        money = 10000;
        password = 123;
    }
};

// 派生类：Student（公共继承）
class Student : public Person {
public:
    void test() {
        // 可访问公共成员
        cout << "名字: " << name << endl;

        // 可访问保护成员（在子类内部）
        cout << "存款: " << money << endl;

        // 不可访问私有成员
        // cout << "密码: " << password << endl; //错误
    }
};

int main() {
    Student s;
    
    // 可访问公共成员
    s.name = "张三"; //允许

    // 不可访问保护成员
    // s.money = 0; //错误

    // 不可访问私有成员
    // s.password = 0; //错误

    s.test(); // 调用子类函数
    return 0;
}
```

向上转型：

- 子类的对象，可以直接赋值给父类的指针或引用。


```c++
#include <iostream>
#include <string>
using namespace std;

// 基类：Person
class Person {
public:
    string name;
    
    Person() {
        name = "某人";
    }
};

// 派生类：Student（公共继承）
class Student : public Person {
public:
    string studentId;
    
    Student() {
        studentId = "S001";
    }
};

int main() {
    Student s;
    s.name = "小明";

    // 向上转型：基类指针指向派生类对象
    Person* p = &s; 

    // 通过基类指针访问基类成员
    cout << p->name << endl; // 输出：小明

    return 0;
}
```

2. 保护继承

```c++
#include <iostream>
using namespace std;

// 爷爷类
class Grandpa {
public:
    int m_Pub = 10;  // 公开成员
protected:
    int m_Pro = 20;  // 保护成员
private:
    int m_Pri = 30;  // 私有成员
};

// 爸爸类（保护继承自Grandpa）
class Father : protected Grandpa {
public:
    void test() {
        cout << "爸爸访问爷爷的钱: " << m_Pub << endl; //允许
        cout << "爸爸访问爷爷的宝: " << m_Pro << endl; //允许
    }
};

// 孙子类（公共继承自Father）
class GrandSon : public Father {
public:
    void test() {
        // 继承链：Grandpa的public→Father的protected→GrandSon的protected
        cout << "孙子访问爷爷的钱: " << m_Pub << endl; //允许
        cout << "孙子访问爷爷的宝: " << m_Pro << endl; //允许
    }
};

int main() {
    Father f;
    // f.m_Pub = 100; //错误：保护继承后，public变为protected
    
    GrandSon gs;
    // gs.m_Pub = 100; //错误：仍然是protected
    
    return 0;
}
```

3. 私有继承

```c++
#include <iostream>
using namespace std;

// 爷爷类
class Grandpa {
public:
    int m_Pub = 1;
protected:
    int m_Pro = 2;
private:
    int m_Pri = 3;
};

// 爸爸类（私有继承自Grandpa）
class Father : private Grandpa {
public:
    void func() {
        // 在Father内部可以访问继承来的成员
        cout << m_Pub << endl; //允许
        cout << m_Pro << endl; //允许
    }
};

// 孙子类（公共继承自Father）
class GrandSon : public Father {
public:
    void func() {
        //无法访问：私有继承后成员变为Father的私有
        // cout << m_Pub << endl; // 错误
        // cout << m_Pro << endl; // 错误
    }
};

int main() {
    Father f;
    // f.m_Pub = 100; //错误：外部无法访问私有成员
    
    return 0;
}
```

### 继承中的对象模型

思考：

- 从父类继承过来的成员，哪些属于子类对象中？
- 既然子类访问不到父类的 private 私有成员，那子类对象里肯定就没有这些数据吧？

1. 内存布局

- 里面：是父类的所有成员变量。

- 外层：才是子类自己新增的成员变量。

2. 实验

```c++
#include <iostream>
using namespace std;

class Base {
public:
    int m_A;    // 4字节
protected:
    int m_B;    // 4字节
private:
    int m_C;    // 4字节
};

class Son : public Base {
public:
    int m_D;    // 4字节
};

int main() {
    cout << "父类大小: " << sizeof(Base) << endl;  // 12字节
    cout << "子类大小: " << sizeof(Son) << endl;   // 16字节
    
    return 0;
}
```

3. 终极查看工具：Visual Studio 开发人员命令提示符

找到电脑上的 "Developer Command Prompt for VS 20xx"

进入你的代码所在的文件夹（用 cd 命令）

输入以下命令：`cl /d1 reportSingleClassLayout类名 文件名.cpp`

<img src="C:\Users\jack\Desktop\学习笔记\C&cpp\attachments\image-20251129144250650.png" alt="image-20251129144250650" style="zoom:33%;" />

```
class Son size(16):
    +---
 0  | +--- (base class Base) -- 父类开始
 0  | | m_A
 4  | | m_B
 8  | | m_C
    | +---                     -- 父类结束
12  | m_D                      -- 子类开始
    +---
```

>结论： 父类中私有成员也是被子类继承下去了，只是由编译器给隐藏后访问不到

>子类对象包含父类的所有成员变量

>父类成员在低地址（上面），子类成员在高地址（下面）

### 继承中构造和析构顺序

子类继承父类后，当创建子类对象，也会调用父类的构造函数

问题：父类和子类的构造和析构顺序是谁先谁后？

- 构造顺序：先父后子
- 析构顺序：先子后父

```c++
#include <iostream>
using namespace std;

// 基类
class Base {
public:
    Base() {
        cout << "Base 构造函数" << endl;
    }
    ~Base() {
        cout << "Base 析构函数" << endl;
    }
};

// 派生类
class Son : public Base {
public:
    Son() {
        cout << "Son 构造函数" << endl;
    }
    ~Son() {
        cout << "Son 析构函数" << endl;
    }
};

void test() {
    // 创建派生类对象
    Son s; 
}

int main() {
    cout << "创建对象：" << endl;
    test();
    cout << "对象已销毁" << endl;
    
    return 0;
}
```

```text
创建对象：
Base 构造函数
Son 构造函数
Son 析构函数
Base 析构函数
对象已销毁
```

内存生命周期：

```
Base构造 → Son构造 → [对象使用] → Son析构 → Base析构
```

>总结：继承中，先调用父类构造函数，再调用子类构造函数，析构顺序与构造相反

### 继承同名成员处理方式

问题：当子类与父类出现同名的成员，如何通过子类对象，访问到子类或父类中同名的数据呢？

- 访问子类同名成员，直接访问即可
- 访问父类同名成员，需要加作用域

>就近原则

- 默认访问子类的：如果你直接访问，编译器会认为你要找离你最近的那个（也就是子类自己的）。父类的同名成员被 “隐藏” 了。

- 加作用域访问父类的：如果你非要找父类的，必须指名道姓，加上父类的作用域。

示例：同名成员变量

```c++
#include <iostream>
using namespace std;

class Base {
public:
    int m_A = 100; // 基类成员
};

class Son : public Base {
public:
    int m_A = 200; // 派生类同名成员
};

int main() {
    Son s;

    // 直接访问：派生类成员
    cout << "Son m_A = " << s.m_A << endl; // 200

    // 作用域解析：基类成员  
    cout << "Base m_A = " << s.Base::m_A << endl; // 100

    return 0;
}
```

示例：同名成员函数

```c++
#include <iostream>
using namespace std;

class Base {
public:
    void func() {
        cout << "Base::func() 调用" << endl;
    }
};

class Son : public Base {
public:
    // 派生类定义同名函数 - 隐藏基类函数
    void func() {
        cout << "Son::func() 调用" << endl;
    }
};

int main() {
    Son s;

    // 直接调用：使用派生类函数
    s.func();           // 输出: Son::func() 调用

    // 作用域解析：显式调用基类函数
    s.Base::func();     // 输出: Base::func() 调用

    return 0;
}
```

易错：同名函数会“覆盖”重载

- 在同一个类里，函数名相同、参数不同，叫重载
- 在继承中，子类一旦出现了同名函数，父类中所有的同名函数（不管参数一不一样）都会被隐藏！

```c++
#include <iostream>
using namespace std;

class Base {
public:
    void func() {
        cout << "Base::func()" << endl;
    }
};

class Son : public Base {
public:
    // 定义不同参数的函数 - 隐藏基类所有同名函数
    void func(int a) {
        cout << "Son::func(int a) - 参数: " << a << endl;
    }
};

int main() {
    Son s;
    
    s.func(10);        //正确：调用派生类函数
    
    // s.func();       //错误：基类函数被隐藏
    s.Base::func();    //正确：显式调用基类函数

    return 0;
}
```

>继承关系中没有“重载”，只有“隐藏”

1. 子类对象可以直接访问到子类中同名成员
2. 子类对象加作用域可以访问到父类同名成员
3. 当子类与父类拥有同名的成员函数，子类会隐藏父类中同名成员函数，加作用域可以访问到父类中同名函数

### 继承同名静态成员处理方式

问题：继承中同名的静态成员在子类对象上如何进行访问？

静态成员和非静态成员出现同名，处理方式一致

- 访问子类同名成员，直接访问即可
- 访问父类同名成员，需要加作用域

1. 遮蔽

不管是不是静态的：

- 子类如果定义了同名成员，会隐藏父类的所有同名成员。

- 默认访问子类的，想访问父类的必须加作用域。

2. 同名静态成员变量

假设 Base 和 Son 都有 `static int m_A`

- 通过对象访问：

```c++
Son s;
s.m_A;        // 访问子类的
s.Base::m_A;  // 访问父类的 (加上父类作用域)
```

- 通过类名访问

这是静态成员独有的写法。既然它属于类，直接用类名找。

```c++
Son::m_A;       // 访问子类的
Base::m_A;      // 直接去父类找 (这谁都会)

//通过子类找父类
Son::Base::m_A; // 翻译：在 Son 的作用域下，找 Base 的 m_A
```

- 示例

```c++
// 静态成员函数也遵循相同规则
class Base {
public:
    static void show() {
        cout << "Base::show()" << endl;
    }
};

class Son : public Base {
public:
    static void show() {
        cout << "Son::show()" << endl;
    }
};

int main() {
    Son::show();        // 调用Son::show()
    Son::Base::show();  // 调用Base::show()
    
    return 0;
}
```

3. 同名静态成员函数

如果子类定义了同名静态函数，父类的就会被隐藏。

```c++
#include <iostream>
using namespace std;

class Base {
public:
    static void func() {
        cout << "Base::static func()" << endl;
    }
};

class Son : public Base {
public:
    static void func() {
        cout << "Son::static func()" << endl;
    }
};

int main() {
    // 调用派生类静态函数
    Son::func();           // Son::static func()
    
    // 通过派生类调用基类静态函数
    Son::Base::func();     // Base::static func()
    
    return 0;
}
```

>总结：同名静态成员处理方式和非静态处理方式一样，只不过有两种访问的方式（通过对象 和 通过类名）

### 多继承

在 C++ 中，多继承允许一个子类同时拥有两个或两个以上的父类。

1. 语法格式

```c++
class 子类 : 继承方式 父类1, 继承方式 父类2, ... {
    // ...
};
```

>每个父类前面都要单独写继承方式（比如 public）。如果你少写了，默认就是 private。

> 多继承可能会引发父类中有同名成员出现，需要加作用域区分

>总结： 多继承中如果父类中出现了同名情况，子类使用时候要加作用域

2. 示例

```c++
#include <iostream>
using namespace std;

// 基类1：飞行能力
class Flyable {
public:
    Flyable() { m_Height = 1000; }
    void fly() {
        cout << "我能飞到 " << m_Height << " 米高空！" << endl;
    }
    int m_Height;
};

// 基类2：力量能力
class Strong {
public:
    Strong() { m_Power = 9999; }
    void smash() {
        cout << "我能举起 " << m_Power << " 公斤的石头！" << endl;
    }
    int m_Power;
};

// 派生类：多重继承
class SuperHero : public Flyable, public Strong {
public:
    void saveWorld() {
        cout << "我既能飞，又大力，我要拯救世界！" << endl;
    }
};

int main() {
    SuperHero hero;

    // 访问Flyable基类的成员
    hero.fly();
    cout << "飞行高度：" << hero.m_Height << endl;

    // 访问Strong基类的成员
    hero.smash();
    cout << "力量值：" << hero.m_Power << endl;

    // 访问派生类自己的成员
    hero.saveWorld();

    // 查看对象大小（两个基类的int成员）
    cout << "SuperHero大小: " << sizeof(hero) << " 字节" << endl;

    return 0;
}
```

3. 示例：同名成员冲突

如果两个父类里，都有一个同名的变量，子类该怎么办？

解决方案：加作用域。

```c++
#include <iostream>
using namespace std;

class Base1 {
public:
    int m_A = 100;
};

class Base2 {
public:
    int m_A = 200; // 同名成员变量
};

class Son : public Base1, public Base2 {
    // 继承了两个同名的m_A
};

int main() {
    Son s;
    
    // 直接访问会产生二义性
    // cout << s.m_A << endl; //错误：不明确
    
    // 使用作用域解析消除二义性
    cout << "Base1 的 m_A: " << s.Base1::m_A << endl; // 100
    cout << "Base2 的 m_A: " << s.Base2::m_A << endl; // 200
    
    return 0;
}
```

### 菱形继承

1. 菱形继承概念：

- 两个派生类继承同一个基类
- 又有某个类同时继承者两个派生类
- 这种继承被称为菱形继承，或者钻石继承

2. 菱形继承问题：

- 羊继承了动物的数据，驼同样继承了动物的数据，当草泥马使用数据时，就会产生二义性。
- 草泥马继承自动物的数据继承了两份，其实我们应该清楚，这份数据我们只需要一份就可以。

```
      Animal (爷爷)
     /      \
  Sheep    Camel (爸爸)
     \      /
    SheepCamel (孙子)
```

3. 未处理代码

```c++
#include <iostream>
using namespace std;

class Animal {
public:
    int m_Age;
};

// 普通继承
class Sheep : public Animal {};
class Camel : public Animal {};

// 多继承 - 菱形继承问题
class SheepCamel : public Sheep, public Camel {};

int main() {
    SheepCamel sc;
    
    // 直接访问会产生二义性
    // sc.m_Age = 18; //错误：不明确是Sheep的还是Camel的
    
    // 必须使用作用域解析
    sc.Sheep::m_Age = 18;
    sc.Camel::m_Age = 28;

    cout << "羊的年龄：" << sc.Sheep::m_Age << endl; // 18
    cout << "驼的年龄：" << sc.Camel::m_Age << endl; // 28
    
    // 问题：一只羊驼有两个不同的年龄，这不合理
    return 0;
}
```

4. 解决方案：虚继承 (`virtual`)

>告诉编译器：“如果后面有人同时继承了我们俩，麻烦把我们共同的爷爷 Animal 合并成一份，大家共享。”

```c++
#include <iostream>
using namespace std;

class Animal {
public:
    int m_Age;
};

// 虚继承：解决菱形继承问题
class Sheep : virtual public Animal {};
class Camel : virtual public Animal {};

class SheepCamel : public Sheep, public Camel {};

int main() {
    SheepCamel sc;

    // 直接访问 - 无二义性
    sc.m_Age = 10; 

    // 验证数据共享
    cout << "通过Sheep访问: " << sc.Sheep::m_Age << endl; // 10
    cout << "通过Camel访问: " << sc.Camel::m_Age << endl; // 10
    cout << "直接访问: " << sc.m_Age << endl; // 10
    
    return 0;
}
```

- 菱形继承带来的主要问题是子类继承两份相同的数据，导致资源浪费以及毫无意义
- 利用虚继承可以解决菱形继承问题

## 多态

### 多态的基本概念

多态分为两类

- 静态多态: 函数重载和运算符重载属于静态多态，复用函数名
- 动态多态: 派生类和虚函数实现运行时多态

静态多态和动态多态区别：

- 静态多态的函数地址早绑定 - 编译阶段确定函数地址
- 动态多态的函数地址晚绑定 - 运行阶段确定函数地址
- 静态多态：看指针的类型
- 动态多态：看指针指向的实际对象

1. 什么是多态

多态是面向对象编程（OOP）的三大核心特性之一，指同一接口（如函数名、方法名）在不同上下文（不同对象）中表现出不同行为的能力。C++ 中多态分为两类：

- 静态多态（编译时多态）：通过函数重载、模板实现，编译器在编译阶段确定调用的具体函数；
- 动态多态（运行时多态）：通过继承 + 虚函数+ 基类指针 / 引用实现，程序运行时根据实际指向的对象类型确定调用的函数，是多态的核心体现。

多态通过抽象基类定义统一接口，派生类重写该接口实现具体逻辑；调用时只需通过基类接口操作，无需关心具体派生类类型，程序会自动匹配对应实现。

2. 多态成立的三个条件：

- 要有继承关系

- 子类要重写父类的虚函数

- 父类指针或引用指向子类对象


3. 静态多态

静态多态又称编译时多态，是 C++ 中多态的一种实现形式：编译器在编译阶段就确定要调用的具体函数 / 方法，无需运行时解析。其核心是通过 

函数重载、运算符重载、模板（函数模板 / 类模板）等机制，让同一接口名称根据参数类型、数量或模板实参的不同，绑定到不同的实现逻辑，实

现 “接口复用，编译期决议”。

- 函数重载

```c++
#include <iostream>
using namespace std;

// 函数重载：根据参数类型选择不同实现
void add(int a, int b) { 
    cout << "int add: " << a + b << endl; 
}

void add(double a, double b) { 
    cout << "double add: " << a + b << endl; 
}

int main() {
    add(1, 2);       // 调用 int 版本
    add(1.1, 2.2);   // 调用 double 版本
    
    return 0;
}
```

- 函数模板

```c++
#include <iostream>
using namespace std;

// 函数模板：通用打印函数
template<typename T>
void myPrint(T a) {
    cout << "打印: " << a << endl;
}

int main() {
    myPrint(10);      // 实例化为 myPrint<int>(10)
    myPrint(3.14);    // 实例化为 myPrint<double>(3.14)
    myPrint("Hello"); // 实例化为 myPrint<const char*>("Hello")
    
    return 0;
}
```

4. 动态多态

动态多态又称运行时多态，是 C++ 面向对象编程中多态的核心形式：程序在运行阶段才根据对象的实际类型确定要调用的成员函数，而非编译阶

段。其实现依赖继承关系、虚函数、基类指针 / 引用三大要素，通过虚函数表机制动态解析函数调用，让基类接口能适配不同派生类的具体实现。

实现动态多态的“三要素”：

- 有继承关系：子类继承父类。
- 子类重写父类的虚函数：
  - 父类函数前加 `virtual`。
  - 子类函数名、参数必须完全一样（这叫 重写/覆盖 Override，不是重载）。
- 父类指针（或引用）指向子类对象。

```c++
#include <iostream>
using namespace std;

// 基类
class Animal {
public:
    // 虚函数 - 实现多态的关键
    virtual void speak() {
        cout << "动物在说话" << endl;
    }
};

// 派生类 Cat
class Cat : public Animal {
public:
    // 重写基类虚函数
    void speak() override {
        cout << "小猫在喵喵喵" << endl;
    }
};

// 派生类 Dog
class Dog : public Animal {
public:
    void speak() override {
        cout << "小狗在汪汪汪" << endl;
    }
};

// 业务函数：接受基类引用
void doSpeak(Animal& animal) {
    // 多态调用：根据实际对象类型调用相应的函数
    animal.speak();
}

int main() {
    Cat cat;
    Dog dog;

    doSpeak(cat); // 输出: 小猫在喵喵喵
    doSpeak(dog); // 输出: 小狗在汪汪汪

    return 0;
}
```

5. 虚函数表

当你给类加上 `virtual` 关键字后，编译器偷偷做了两件事：

- 在对象里塞了个指针 (`vfptr`)：
  - 每个对象（比如 `cat`）内部，都多了一个由系统维护的指针，叫虚函数指针 (vfptr)。

- 在后台生成了一张表 (`vftable`)：
  - 这是一个数组，里面存的是函数地址。

6. 什么时候用哪个

- 用静态多态：当你明确知道参数类型，或者你在写通用的库（比如 STL），追求极致的效率。

- 用动态多态：当你需要处理异构集合（比如一个数组里既有猫又有狗），或者你需要设计插件系统、框架接口时。

7. 示例

```c++
class Animal
{
public:
	//Speak函数就是虚函数
	//函数前面加上virtual关键字，变成虚函数，那么编译器在编译的时候就不能确定函数调用了。
	virtual void speak()
	{
		cout << "动物在说话" << endl;
	}
};

class Cat :public Animal
{
public:
	void speak()
	{
		cout << "小猫在说话" << endl;
	}
};

class Dog :public Animal
{
public:

	void speak()
	{
		cout << "小狗在说话" << endl;
	}

};
//我们希望传入什么对象，那么就调用什么对象的函数
//如果函数地址在编译阶段就能确定，那么静态联编
//如果函数地址在运行阶段才能确定，就是动态联编

void DoSpeak(Animal & animal)
{
	animal.speak();
}
//
//多态满足条件： 
//1、有继承关系
//2、子类重写父类中的虚函数
//多态使用：
//父类指针或引用指向子类对象

void test01()
{
	Cat cat;
	DoSpeak(cat);


	Dog dog;
	DoSpeak(dog);
}


int main() {

	test01();
	return 0;
}
```

总结：

多态满足条件

- 有继承关系
- 子类重写父类中的虚函数

多态使用条件

- 父类指针或引用指向子类对象

> 重写：函数返回值类型，函数名，参数列表，完全一致称为重写

### 多态案例一：计算器类

```c++
//普通实现
class Calculator {
public:
	int getResult(string oper)
	{
		if (oper == "+") {
			return m_Num1 + m_Num2;
		}
		else if (oper == "-") {
			return m_Num1 - m_Num2;
		}
		else if (oper == "*") {
			return m_Num1 * m_Num2;
		}
		//如果要提供新的运算，需要修改源码
	}
public:
	int m_Num1;
	int m_Num2;
};

void test01()
{
	//普通实现测试
	Calculator c;
	c.m_Num1 = 10;
	c.m_Num2 = 10;
	cout << c.m_Num1 << " + " << c.m_Num2 << " = " << c.getResult("+") << endl;

	cout << c.m_Num1 << " - " << c.m_Num2 << " = " << c.getResult("-") << endl;

	cout << c.m_Num1 << " * " << c.m_Num2 << " = " << c.getResult("*") << endl;
}



//多态实现
//抽象计算器类
//多态优点：代码组织结构清晰，可读性强，利于前期和后期的扩展以及维护
class AbstractCalculator
{
public :

	virtual int getResult()
	{
		return 0;
	}

	int m_Num1;
	int m_Num2;
};

//加法计算器
class AddCalculator :public AbstractCalculator
{
public:
	int getResult()
	{
		return m_Num1 + m_Num2;
	}
};

//减法计算器
class SubCalculator :public AbstractCalculator
{
public:
	int getResult()
	{
		return m_Num1 - m_Num2;
	}
};

//乘法计算器
class MulCalculator :public AbstractCalculator
{
public:
	int getResult()
	{
		return m_Num1 * m_Num2;
	}
};


void test02()
{
	//创建加法计算器
	AbstractCalculator *abc = new AddCalculator;
	abc->m_Num1 = 10;
	abc->m_Num2 = 10;
	cout << abc->m_Num1 << " + " << abc->m_Num2 << " = " << abc->getResult() << endl;
	delete abc;  //用完了记得销毁

	//创建减法计算器
	abc = new SubCalculator;
	abc->m_Num1 = 10;
	abc->m_Num2 = 10;
	cout << abc->m_Num1 << " - " << abc->m_Num2 << " = " << abc->getResult() << endl;
	delete abc;  

	//创建乘法计算器
	abc = new MulCalculator;
	abc->m_Num1 = 10;
	abc->m_Num2 = 10;
	cout << abc->m_Num1 << " * " << abc->m_Num2 << " = " << abc->getResult() << endl;
	delete abc;
}

int main() {

	//test01();

	test02();
	return 0;
}
```

### 纯虚函数和抽象类

1. 什么是纯虚函数

纯虚函数是 C++ 中在基类声明但无具体实现的特殊虚函数

它的核心作用是：为派生类定义必须实现的统一接口，同时使包含纯虚函数的基类成为抽象类—— 抽象类无法实例化对象，仅

能作为接口模板被继承。

- 无函数体，仅作接口声明
- 强制派生类重写
- 基类变为抽象类，禁止实例化

```c++
virtual 返回值类型 函数名(参数列表) = 0;
```

2. 什么是抽象类

抽象类是 C++ 中包含至少一个纯虚函数的类（或通过其他方式显式声明为抽象的类），它的核心特征是无法实例化对象，仅作为 “接口模板” 供派

生类继承。抽象类的本质是为派生类定义统一的接口规范，强制派生类实现纯虚函数的具体逻辑，是 “面向接口编程” 的核心载体。

- 无法实例化对象
- 必须包含纯虚函数
- 可作为基类被继承
- 可定义普通成员

3. 示例

```c++
#include <iostream>
using namespace std;

// 抽象基类：饮品
class AbstractDrink {
public:
    // 1. 煮水 (通用实现)
    void Boil() {
        cout << "第一步：煮开水" << endl;
    }

    // 2. 冲泡 (纯虚函数)
    virtual void Brew() = 0;

    // 3. 倒入杯中 (通用实现)
    void PourInCup() {
        cout << "第三步：倒入杯中" << endl;
    }

    // 4. 加佐料 (纯虚函数)
    virtual void PutSomething() = 0;

    // 5. 制作流程 (模板方法模式)
    void makeDrink() {
        Boil();
        Brew();        // 多态调用
        PourInCup();
        PutSomething(); // 多态调用
    }
};

// 具体类：咖啡
class Coffee : public AbstractDrink {
public:
    void Brew() override {
        cout << "第二步：冲泡咖啡粉" << endl;
    }
    
    void PutSomething() override {
        cout << "第四步：加糖和牛奶" << endl;
    }
};

// 具体类：茶
class Tea : public AbstractDrink {
public:
    void Brew() override {
        cout << "第二步：冲泡茶叶" << endl;
    }
    
    void PutSomething() override {
        cout << "第四步：加柠檬" << endl;
    }
};

// 业务函数
void doWork(AbstractDrink* drink) {
    drink->makeDrink();
    delete drink;
}

int main() {
    // 制作咖啡
    cout << "制作咖啡：" << endl;
    doWork(new Coffee);

    cout << "-----------------" << endl;

    // 制作茶
    cout << "制作茶：" << endl;
    doWork(new Tea);

    return 0;
}
```

```c++
class Base
{
public:
	//纯虚函数
	//类中只要有一个纯虚函数就称为抽象类
	//抽象类无法实例化对象
	//子类必须重写父类中的纯虚函数，否则也属于抽象类
	virtual void func() = 0;
};

class Son :public Base
{
public:
	virtual void func() 
	{
		cout << "func调用" << endl;
	};
};

void test01()
{
	Base * base = NULL;
	//base = new Base; // 错误，抽象类无法实例化对象
	base = new Son;
	base->func();
	delete base;//记得销毁
}

int main() {

	test01();
	return 0;
}
```

### 虚析构和纯虚析构

多态使用时，如果子类中有属性开辟到堆区，那么父类指针在释放时无法调用到子类的析构代码

> 解决方式：将父类中的析构函数改为虚析构或者纯虚析构

虚析构和纯虚析构共性：

- 可以解决父类指针释放子类对象

- 都需要有具体的函数实现

虚析构和纯虚析构区别：

- 如果是纯虚析构，该类属于抽象类，无法实例化对象

1. 虚析构语法：

```c++
virtual ~类名(){}
```

2. 纯虚析构语法：

```c++
virtual ~类名() = 0;
```

```c++
类名::~类名(){}
```

3. 示例

```c++
class Animal {
public:

	Animal()
	{
		cout << "Animal 构造函数调用！" << endl;
	}
	virtual void Speak() = 0;

	//析构函数加上virtual关键字，变成虚析构函数
	//virtual ~Animal()
	//{
	//	cout << "Animal虚析构函数调用！" << endl;
	//}


	virtual ~Animal() = 0;
};

Animal::~Animal()
{
	cout << "Animal 纯虚析构函数调用！" << endl;
}

//和包含普通纯虚函数的类一样，包含了纯虚析构函数的类也是一个抽象类。不能够被实例化。

class Cat : public Animal {
public:
	Cat(string name)
	{
		cout << "Cat构造函数调用！" << endl;
		m_Name = new string(name);
	}
	virtual void Speak()
	{
		cout << *m_Name <<  "小猫在说话!" << endl;
	}
	~Cat()
	{
		cout << "Cat析构函数调用!" << endl;
		if (this->m_Name != NULL) {
			delete m_Name;
			m_Name = NULL;
		}
	}

public:
	string *m_Name;
};

void test01()
{
	Animal *animal = new Cat("Tom");
	animal->Speak();

	//通过父类指针去释放，会导致子类对象可能清理不干净，造成内存泄漏
	//怎么解决？给基类增加一个虚析构函数
	//虚析构函数就是用来解决通过父类指针释放子类对象
	delete animal;
}

int main() {

	test01();
	return 0;
}
```

总结：

 1. 虚析构或纯虚析构就是用来解决通过父类指针释放子类对象

 2. 如果子类中没有堆区数据，可以不写为虚析构或纯虚析构

 3. 拥有纯虚析构函数的类也属于抽象类

# 文件操作

>程序运行时产生的数据都属于临时数据，程序一旦运行结束都会被释放

>通过文件可以将数据持久化

C++中对文件操作需要包含头文件 `< fstream >`

文件类型分为两种：

1. 文本文件 - 文件以文本的**ASCII码**形式存储在计算机中
2. 二进制文件 - 文件以文本的**二进制形式**存储在计算机中，用户一般不能直接读懂它们

操作文件的三大类:

1. `ofstream`：写操作
2. `ifstream`： 读操作
3. `fstream`： 读写操作

## 文本文件

### 写文件

写文件步骤如下：

1. 包含头文件

   `\#include <fstream>`

2. 创建流对象

   `ofstream ofs;`

3. 打开文件

   `ofs.open(“文件路径”,打开方式);`

4. 写数据

   `ofs << “写入的数据”;`

5. 关闭文件

   `ofs.close();`

文件打开方式：

|  打开方式   |            解释            |
| :---------: | :------------------------: |
|   ios::in   |     为读文件而打开文件     |
|  ios::out   |     为写文件而打开文件     |
|  ios::ate   |      初始位置：文件尾      |
|  ios::app   |       追加方式写文件       |
| ios::trunc  | 如果文件存在先删除，再创建 |
| ios::binary |         二进制方式         |

**注意：** 文件打开方式可以配合使用，利用|操作符

**例如：**用二进制方式写文件 `ios::binary | ios:: out`

```c++
#include <fstream>

void test01()
{
	ofstream ofs;
	ofs.open("test.txt", ios::out);

	ofs << "姓名：张三" << endl;
	ofs << "性别：男" << endl;
	ofs << "年龄：18" << endl;

	ofs.close();
}

int main() {

	test01();
	return 0;
}
```

总结：

- 文件操作必须包含头文件 `fstream`
- 读文件可以利用 `ofstream` ，或者`fstream`类
- 打开文件时候需要指定操作文件的路径，以及打开方式
- 利用`<<`可以向文件中写数据
- 操作完毕，要关闭文件

### 读文件

读文件与写文件步骤相似，但是读取方式相对于比较多

读文件步骤如下：

1. 包含头文件

   `#include <fstream>`

2. 创建流对象

   `ifstream ifs;`

3. 打开文件并判断文件是否打开成功

   `ifs.open(“文件路径”,打开方式);`

4. 读数据

   四种方式读取

5. 关闭文件

   `ifs.close();`

```c++
#include <fstream>
#include <string>
void test01()
{
	ifstream ifs;
	ifs.open("test.txt", ios::in);

	if (!ifs.is_open())
	{
		cout << "文件打开失败" << endl;
		return;
	}

	//第一种方式
	//char buf[1024] = { 0 };
	//while (ifs >> buf)
	//{
	//	cout << buf << endl;
	//}

	//第二种
	//char buf[1024] = { 0 };
	//while (ifs.getline(buf,sizeof(buf)))
	//{
	//	cout << buf << endl;
	//}

	//第三种
	//string buf;
	//while (getline(ifs, buf))
	//{
	//	cout << buf << endl;
	//}

	char c;
	while ((c = ifs.get()) != EOF)
	{
		cout << c;
	}

	ifs.close();


}

int main() {

	test01();
	return 0;
}
```

总结：

- 读文件可以利用 `ifstream`，或者`fstream`类
- 利用`is_open`函数可以判断文件是否打开成功
- `close`关闭文件

## 二进制文件

以二进制的方式对文件进行读写操作

打开方式要指定为 `ios::binary`

### 写文件

二进制方式写文件主要利用流对象调用成员函数`write`

函数原型：

```c++
ostream& write(const char * buffer,int len);
```

- 参数解释：字符指针`buffer`指向内存中一段存储空间。`len`是读写的字节数

```c++
#include <fstream>
#include <string>

class Person
{
public:
	char m_Name[64];
	int m_Age;
};

//二进制文件  写文件
void test01()
{
	//1、包含头文件

	//2、创建输出流对象
	ofstream ofs("person.txt", ios::out | ios::binary);
	
	//3、打开文件
	//ofs.open("person.txt", ios::out | ios::binary);

	Person p = {"张三"  , 18};

	//4、写文件
	ofs.write((const char *)&p, sizeof(p));

	//5、关闭文件
	ofs.close();
}

int main() {

	test01();
	return 0;
}
```

总结：

- 文件输出流对象 可以通过write函数，以二进制方式写数据

### 读文件
二进制方式读文件主要利用流对象调用成员函数`read`

函数原型：

```c++
istream& read(char *buffer,int len);
```

- 参数解释：字符指针`buffer`指向内存中一段存储空间。`len`是读写的字节数

```c++
#include <fstream>
#include <string>

class Person
{
public:
	char m_Name[64];
	int m_Age;
};

void test01()
{
	ifstream ifs("person.txt", ios::in | ios::binary);
	if (!ifs.is_open())
	{
		cout << "文件打开失败" << endl;
	}

	Person p;
	ifs.read((char *)&p, sizeof(p));

	cout << "姓名： " << p.m_Name << " 年龄： " << p.m_Age << endl;
}

int main() {

	test01();
	return 0;
}
```

- 文件输入流对象 可以通过`read`函数，以二进制方式读数据
