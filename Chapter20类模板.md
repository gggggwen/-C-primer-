# Chapter20 类模板 



##   20.1 类模板基础

```C++
templat <class T>
class 类模板名
{
    .....
}
```



- 类模板的使用

```c++
Stack<string> s(5);
```







##   20.2 类模板具体化



###    20.2.1 完全具体化

具体化程度高的类优先于具体化程度低的类，具体化的类优先于没有具体化的类。

具体化的模板类，成员函数类外实现的代码应该放在源文件中。

```c++
template<class T1, class T2>
class AA {                 // 类模板。
public:
	T1 m_x;
	T2 m_y;

	AA(const T1 x, const T2 y) :m_x(x), m_y(y) { cout << "类模板：构造函数。\n"; }
	void show() const;
};

template<class T1, class T2>
void AA<T1, T2>::show() const {    // 成员函数类外实现。
	cout << "类模板：x = " << m_x << ", y = " << m_y << endl;
}


// 类模板完全具体化
template<>
class AA<int, string> {
public:
	int      m_x;
	string m_y;

	AA(const int x, const string y) :m_x(x), m_y(y) { cout << "完全具体化：构造函数。\n"; }
	void show() const;
};

void AA<int, string>::show() const {    // 成员函数类外实现。
	cout << "完全具体化：x = " << m_x << ", y = " << m_y << endl;
}

```





###     20.2.2 部分具体化

即指定某一个泛型为特定的数据类型 

```c++
// 类模板部分具体化
template<class T1>
class AA<T1, string> {
public:
	T1       m_x;
	string m_y;

	AA(const T1 x, const string y) :m_x(x), m_y(y) { cout << "部分具体化：构造函数。\n"; }
	void show() const;
};

template<class T1>
void AA<T1, string>::show() const {    // 成员函数类外实现。
	cout << "部分具体化：x = " << m_x << ", y = " << m_y << endl;
}

```





## 20.3 类模板继承



### 20.3.1 模板类继承普通类*

**要把普通类的构造函数准备好!!!**





### 20.3.2 普通类继承模板的实例化版本

**注意把普通类的构造函数修改好就行**

```c++
class A:public B<int ,string>
{
    ...
    A(int a ,int b ,string s_b) : c_a(a), c_b(b) , c_sb(s_b)
    {
        .......
    }
}

//使用
int main()
{
    A aa(10 ,100 ,"hahha");
}
```



### 20.3.2 普通类继承模板类*

- **普通类变成模板类才能继承模板类**

```c++
template<class T1, class T2>
class BB      // 模板类BB。
{
public:
	T1 m_x;
	T2 m_y;
	BB(const T1 x, const T2 y) : m_x(x), m_y(y) { cout << "调用了BB的构造函数。\n"; }
	void func2() const { cout << "调用了func2()函数：x = " << m_x << ", y = " << m_y << endl; }
};

template<class T1, class T2>
class AA:public BB<T1,T2>     // 普通类AA变成了模板类，才能继承模板类。
{
public:
	int m_a;
	AA(int a, const T1 x, const T2 y) : BB<T1,T2>(x,y),m_a(a) { cout << "调用了AA的构造函数。\n"; }
	void func1() { cout << "调用了func1()函数：m_a=" << m_a << endl;; }
};

int main()
{
	AA<int,string> aa(3,8, "我是一只傻傻鸟。");
	aa.func1();
	aa.func2();
}

```



###  20.3.3 类模板继承类模板

- **注意派生类`<>`内添加基类的泛型数据 , 同时也调用基类的构造函数** 

```c++
template<class T, class T1, class T2> 
class CC :public BB<T1, T2>   // 模板类继承模板类。
{
public:
	T m_a;
	CC(const T a, const T1 x, const T2 y) : BB<T1, T2>(x, y), m_a(a) { cout << "调用了CC的构造函数。\n"; }
	void func3() { cout << "调用了func3()函数：m_a=" << m_a << endl;; }
};

int main()
{
	CC<int,int,string> cc(3,8, "我是一只傻傻鸟。");
	cc.func3();
	cc.func2();

```









## 20.4 类模板与函数



模板类可以用于函数的参数和返回值，有三种形式：

1）普通函数，参数和返回值是模板类的<font color=blue>实例化版本</font>。

2）函数模板，参数和返回值是某种的模板类。

3）函数模板，参数和返回值是任意类型（支持普通类和模板类和其它类型）。



```c++
template<class T1, class T2>
class AA    // 模板类AA。
{
public:
    T1 m_x;
    T2 m_y;
    AA(const T1 x, const T2 y) : m_x(x), m_y(y) { }
    void show() const { cout << "show()  x = " << m_x << ", y = " << m_y << endl; }
};

```

例如下面例子 我需要一个函数能够返回一个类模板 , 并且以类模板为参数传递 :给出了一下三份代码:



- 这样的形式书写极不规范 , 函数实际需要传递两个参数 ,**但是在显示调用时 , 只显示传递了一个参数 **

```c++
template <typename T1,typename T2>
AA<T1, T2> func(AA<T1, T2>& aa)
{
    aa.show();
    cout << "调用了func(AA<T1, T2> &aa)函数。\n";
    return aa;
}

int main()
{
    AA<int ,int> aa(10,10);
    func(aa);
}
```



- 将泛型数据T 可以假定位某一个类模板 既使得代码更加规范, 也使得代码的能够让所有具有show()函数的类/类模板使用

```c++
template <typename T>
T func(T &aa)
{
    aa.show();
    cout << "调用了func(AA<T> &aa)函数。\n";
    return aa;
}

int main()
{
    AA<int, string> aa(3, "我是一只傻傻鸟。");
    func(aa);
}

```





##  20.5 类模板与友元



###     20.5.1 非模板有元

友元函数不是模板函数，而是利用模板类参数生成的函数，只能在类内实现。并且不支持函数重载

```c++
#include <iostream>         // 包含头文件。
using namespace std;        // 指定缺省的命名空间。

template<class T1, class T2>
class AA    
{
    T1 m_x;
    T2 m_y;
public:
    AA(const T1 x, const T2 y) : m_x(x), m_y(y) { }
    // 非模板友元：友元函数不是模板函数，而是利用模板类参数生成的函数，只能在类内实现。
    friend void show(const AA<T1, T2>& a)
    {
        cout << "x = " << a.m_x << ", y = " << a.m_y << endl;
    }
   /* friend void show(const AA<int, string>& a);
    friend void show(const AA<char, string>& a);*/ //不支持重载
};

//void show(const AA<int, string>& a)
//{
//    cout << "x = " << a.m_x << ", y = " << a.m_y << endl;
//}
//
//void show(const AA<char, string>& a)
//{
//    cout << "x = " << a.m_x << ", y = " << a.m_y << endl;
//}

int main()
{
    AA<int, string> a(88, "我是一只傻傻鸟。");
    show(a);

    AA<char, string> b(88, "我是一只傻傻鸟。");
    show(b);
}

```



### 20.5.2 约束模板友元



<font size=4>**步骤有点复杂**</font>:

- 在类模板前进行声明
- 在类模板汇总声明为友元函数
- 最后在类模板后进行**函数模板的定义以及具体化版本**



```c++
// 约束模板友元：模板类实例化时，每个实例化的类对应一个友元函数。
template <typename T>
void show(T& a);                                // 第一步：在模板类的定义前面，声明友元函数模板。

template<class T1, class T2>
class BB    // 模板类BB。
{
    friend void show<>(BB<T1, T2>& a);          // 第二步：在模板类中，再次声明友元函数模板。
    T1 m_x;
    T2 m_y;

public:

    BB(const T1 x, const T2 y) : m_x(x), m_y(y) { }
};

template <typename T>                                 // 第三步：友元函数模板的定义。
void show(T& a)
{
    cout << "通用：x = " << a.m_x << ", y = " << a.m_y << endl;
}

template <>                                                    // 第三步：具体化版本。
void show(BB<int, string>& a)
{
    cout << "具体BB<int, string>：x = " << a.m_x << ", y = " << a.m_y << endl;
}

int main()
{
    BB<int, string> b1(88, "我是一只傻傻鸟。");
    show(b1);         // 将使用具体化的版本。

    BB<char, string> b2(88, "我是一只傻傻鸟。");
    show(b2);        // 将使用通用的版本。
}
```



##  20.6 成员的类模板

<font size=4>**主要分为:类模板为成员  以及函数模板为成员**</font>:

```c++
using namespace std;
template<class T1, class T2>
class CGirl
{
public:
    template<class T>
    class CBoy   //类模板为成员
    {
    public:
        T m_a;
        T1 m_b;
        void show(T t);
        CBoy() {}
    };

    CBoy<std::string> boy;

    template<typename T> //函数模板为成员
    void func();
};


//嵌套的函数模板或者类模板中的函数可以在类模板外部定义
template<class T1, class T2>
template<typename T>
void CGirl<T1, T2>::func()
{
    std::cout << "func\n";
}

template<class T1, class T2>
template<class T>
void CGirl<T1, T2>::CBoy<T>::show(T t)
{
    std::cout << t << endl;
    return;
}

int main() {
    CGirl<int, double> girl;
    girl.func<int>();

    girl.boy.m_a = "Hello";
    girl.boy.show("World");

    return 0;
}
```





## 20.7 将模板类用作参数



```c++
#include <iostream>         // 包含头文件。
using namespace std;        // 指定缺省的命名空间。

template <class T, int len>
class LinkList             // 链表类模板。
{
public:
    T*   m_head;          // 链表头结点。
    int  m_len = len;   // 表长。
    void insert()     { cout << "向链表中插入了一条记录。\n"; }
    void ddelete()  { cout << "向链表中删除了一条记录。\n"; }
    void update()   { cout << "向链表中更新了一条记录。\n"; }
};

template <class T, int len>
class Array                // 数组类模板。
{
public:
    T*   m_data;          // 数组指针。
    int  m_len = len;   // 表长。
    void insert()     { cout << "向数组中插入了一条记录。\n"; }
    void ddelete()  { cout << "向数组中删除了一条记录。\n"; }
    void update()   { cout << "向数组中更新了一条记录。\n"; }
};

// 线性表模板类：tabletype-线性表类型，datatype-线性表的数据类型。
template<template<class, int >class tabletype, class datatype, int len>
class LinearList
{
public:
    tabletype<datatype, len> m_table;     // 创建线性表对象。

    void insert()    { m_table.insert(); }         // 线性表插入操作。
    void ddelete() { m_table.ddelete(); }      // 线性表删除操作。
    void update()  { m_table.update(); }      // 线性表更新操作。

    void oper()     // 按业务要求操作线性表。
    {
        cout << "len=" << m_table.m_len << endl;
        m_table.insert();
        m_table.update();
    }
};

int main()
{
    // 创建线性表对象，容器类型为链表，链表的数据类型为int，表长为20。
    LinearList<LinkList, int, 20>  a;   
    a.insert();   
    a.ddelete();   
    a.update();

    // 创建线性表对象，容器类型为数组，数组的数据类型为string，表长为20。

```

