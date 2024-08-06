# Chapter16 运算符重载



##    16.1 运算符重载基础

C++将运算符重载扩展到自定义的数据类型，**它可以让对象操作更美观**。 例如:字符串string用加号（+）拼接、cout用两个左尖括号（<<）输出。

运算符重载模板:

```c++
返回值类型 operator需重载的运算符(形参)
{
     ........
}
```

<font color=red>**如果说需要支持赋值操作请将返回值类型改为对象或者结构体的引用**</font>

示例:

```c++
struct CGirl
{
		
private:
	int m_score;
public:
	string m_name;
	CGirl(string name, int score):m_name(name),m_score(score)
	{}
	void show()
	{	cout << "name :" << m_name << "    score :" << m_score << endl;}
	CGirl& operator+( int score) //如果想要支持`=`的运算符重载则需要返回对象的引用
	{
		cout << "调用了operator+\n";
		m_score = m_score + score;
		return *this; //注意返回值为this指针的解引用
	}
};

int main()
{
	CGirl g = CGirl("gwen", 90);
	g.show();
	g=g + 100+100;
	g.show();
}
```

**注意：**

1）返回自定义数据类型的引用可以让多个运算符表达式串联起来。（不要返回局部变量的引用）

2）重载函数参数列表中的顺序决定了操作数的位置。

​     eg:  `g = 100+g`会报错, 运算符重载的本质也是函数的调用!

​             `g = g+g`    也会报错相当于调用`CGirl& operator+(CGirl& g1,CGirl& g2)`

3）重载函数的参数列表中至少有一个是用户自定义的类型，防止程序员为内置数据类型重载运算符。<font color = blue>(属于编程规范,不按照此规定会报错)</font>

4）如果运算符重载既可以是成员函数也可以是全局函数，应该优先考虑成员函数，这样更符合运算符重载的初衷。

5）重载函数不能违背运算符原来的含义和优先级。<font color = blue>(属于编程规范)</font>



## 16.2 左移运算符重载

重载左移运算符（<<）用于输出自定义对象的成员变量，在实际开发中很有价值（调试和日志）。

`cout`是`iostream ` 头文件中的一个全局对象 为`ostream`类在对`<<`左移运算符进行重载时 一般会与`cout`有关:

- **当重载运算符作为全局函数时:**

  如果要输出对象的私有成员，可以配合友元一起使用。

```c++
ostream& operator<<(ostream& cout, CGirl& g)
{
	cout << "name :" << g.m_name  << endl;
	return cout;
}

int main()
{
    CGirl g = CGirl("gwen", 90);
    cout << g;
}
```

- **当重载运算符作为成员函数时:**

```c++
ostream& operator<<(ostream& cout)
	{
		cout << "name: " << m_name << "  score:" << m_score << endl;
		return cout
	}
int main()
  {
      CGirl g = CGirl("gwen", 90);
      g<<cout;
  }
```


<font color = red>**注意如果在成员函数中重载运算符 , 因为对象本身只能隐式地传递因此对象本身操作数的位置只能在cout 的左边,**</font>**所以说 , 为了编程规范一般重载左移运算符无成员函数的版本**





##  16.3 下标运算符重载

如果对象中有数组，重载下标运算符[]，操作[对象中的数组](https://so.csdn.net/so/search?q=对象数组&spm=1001.2101.3001.7020)将像操作普通数组一样方便。

**下标运算符必须以成员函数的形式进行重载。**

下标运算符重载函数的语法：

`返回值类型 &perator[](参数);`

   使用第一种声明方式，[]不仅可以访问数组元素，还可以修改数组元素。

`  const 返回值类型 &operator[](参数) const;`

  使用第二种声明方式，[]只能访问而不能修改数组元素。

在实际开发中，我们应该同时提供以上两种形式，这样做是为了适应const对象，因为[通过const 对象只能调用const成员函数](http://c.biancheng.net/view/2232.html)，如果不提供第二种形式，那么将无法访问const对象的任何数组元素。

```c++
	string& operator[](int index)
	{
		return boys[index];
	}

    const string& operator[](int index) const 
	{
		return boys[index];
	}
```





## 16.4 重载赋值运算符

C++编译器可能会给类添加四个函数：

l 默认构造函数，空实现。

l 默认析构函数，空实现。

l 默认拷贝构造函数，对成员变量进行浅拷贝。

l 默认赋值函数, **对成员变量进行浅拷贝**。 



对象的赋值运算是用一个已经存在的对象，给另一个已经存在的对象赋值。

如果类的定义中没有重载赋值函数，编译器就会提供一个默认赋值函数。

如果类中重载了赋值函数，编译器将不提供默认赋值函数。

重载赋值函数的语法：`类名 & operator=(const 类名 & 源对象);`



注意：

- l 编译器提供的默认赋值函数，是浅拷贝。

- l 如果对象中不存在堆区内存空间，默认赋值函数可以满足需求，否则需要深拷贝。

- l 赋值运算和拷贝构造不同：拷贝构造是指原来的对象不存在，用已存在的对象进行构造；赋值运算是指已经存在了两个对象，把其中一个对象的成员变量的值赋给另一个对象的成员变量。





## 16.5 new 和delete运算符的重载

重载new和delete运算符的目是为了自定义内存分配的细节。（内存池：快速分配和归还，无碎片）

在C++中，使用new时，编译器做了两件事情：

1）调用标准库函数operator new()分配内存；//这里是标准库函数自定义的 ,没有重载的那种

2）调用构造函数初始化内存；

使用delete时，也做了两件事情：

1）调用析构函数；

2）调用标准库函数operator delete()释放内存。//这里是标准库函数自定义的 ,没有重载的那种

<font color = purple>**构造函数和析构函数由编译器调用**</font>，我们无法控制。但是，可以<font color =red>**重载内存分配函数operator new()和释放函数operator delete()**</font>。



<font size=4>基本格式:</font>

```c++
void* operator new(size_t size)
{
	cout << "调用了new的重载" << endl;
	void* ptr = malloc(size);
	return ptr;
}

void operator delete(void* ptr)
{
	cout << "调用了delete的重载" << endl;
	if (ptr == 0) return;
	free(ptr);
}
```

一个类重载new和delete时，尽管不必显式地使用static，但实际上仍在创建static成员函数。

编译器看到使用new创建自定义的类的对象时，它选择成员版本的operator new()而不是全局版本的new()。

```c++
 CGirl& operator=(const CGirl& g)
    {
        if (this == &g) return *this;          // 如果是自己给自己赋值。
        
        if (g.m_ptr == nullptr)    // 如果源对象的指针为空，则清空目标对象的内存和指针。
        {
            if (m_ptr != nullptr) { delete m_ptr; m_ptr = nullptr; }
        }
        else    // 如果源对象的指针不为空。
        {
            // 如果目标对象的指针为空，先分配内存。
            if (m_ptr == nullptr) m_ptr = new int;
            // 然后，把源对象内存中的数据复制到目标对象的内存中。
            memcpy(m_ptr, g.m_ptr, sizeof(int));
        }
        
        m_bh = g.m_bh; m_name = g.m_name;
        cout << "调用了重载赋值函数。\n" << endl; 
        return *this;
    }
};

```



## 16.6 内存池

预先分配一大块内存 ,减少内存碎片 提升分配和释放速度 。

上一小节讲到**new和delete运算符的重载主要还是用于内存池**,下面来实现一个简易的内存池:

```c++

class CGirl
{
public:
	int m_size; 
	int m_num;
	static char* m_pool;
	static bool init_pool()
	{
		if (m_pool != 0) std::cout << "已初始化!\n";
		m_pool = (char*)malloc(18);
		memset((void*)m_pool, 0, 18);
		if (m_pool == 0)
		{
			std::cout << "内存分配失败" << std::endl;
			return false; 
		}
		std::cout << "内存已分配" << std::endl;
		return true;}
	static bool free_pool()
	{
		if (m_pool == 0) return true;
		free(m_pool);
		std::cout << "内存池已释放\n";
		return true; 
	}

	void* operator new(size_t size)
	{
		if (m_pool[0] == 0)
		{
			std:: cout << "已分配第一块内存块\n";
			m_pool[0] = 1;
			return m_pool+1;

		}
		if (m_pool[9] == 0)
		{
			std::cout << "已分配第二块内存块\n";
			m_pool[9] = 1;
			return m_pool+10;
		}
		void* temp = malloc(8);
		std::cout << "已分配系统资源\n";
		return temp; 
	}
	void operator delete(void* ptr)
	{
		if (ptr == 0) return;
		if (ptr == m_pool + 1)
		{
			std::cout << "已释放第一块内存块\n";
			m_pool[0] = 0;
			return;
		}
		if (ptr == m_pool + 10)
		{
			std::cout << "已释放第二块内存块\n";
			m_pool[9] = 0;
			return;
		}
		free(ptr);//属于系统资源的情况
		std::cout << "已释放系统资源" << std::endl;
	}
	CGirl(int num, int size) { m_size = size, m_num = num;std::cout << "调用了构造函数CGirl()\n"; }
	~CGirl() { std::cout << "调用了析构函数~CGirl()\n"; }
};


```

- 为什么`m_pool`需要用`static`修饰?

  ​     new和delete重载默认为`static`修饰 , 又由于<font color =red>**静态成员函数只能操作静态成员变量, 非静态成员函数只能操作非静态成员变量**</font>,  初始化和释放内存池的函数也应当用`static`修饰。 且注意<font color =blue>**静态成员变量需要在类外进行初始化 **</font>

```c++
char* CGirl::m_pool =nullptr;
```

- 为什么重载的delete函数,想要释放**内存池资源**不需要调用`free`?

​            当释放内存池管理的内存块时，`operator delete`不应调用`free`来释放它们，因为这些内存块实际上是由内存池管理的，并且整个池可能会在适当的时间被释放。`free`调用会导致内存错误。





## 16.7 一元运算符重载



可重载的一元运算符。

1）++ 自增    2）-- 自减   3）! 逻辑非   4）& 取地址

5）~ 二进制反码  6）* 解引用  7）+ 一元加  8） - 一元求反

一元运算符通常出现在它们所操作的对象的左边。

但是，自增运算符++和自减运算符--有前置和后置之分。

[C++](http://c.biancheng.net/cplus/) 规定，重载++或--时，如果重载函数有一个int形参，编译器处理后置表达式时将调用这个重载函数。

成员函数版：`CGirl &operator++();       // ++前置`

成员函数版：`CGirl operator++(int);      // 后置++`

非成员函数版：`CGirl &operator++(CGirl &);  // ++前置`

非成员函数版：`CGirl operator++(CGirl &,int); // 后置++`





## 16.8 自动类型转换

对于内置类型，如果两种数据类型是兼容的，C++可以自动转换，如果从更大的数转换为更小的数，可能会被截断或损失精度。

**如果某种类型与类相关，从某种类型转换为类类型是有意义的。**`string str = "我是一只傻傻鸟。";`

在C++中，<font color = red>**将一个参数的构造函数用作自动类型转换函数，它是自动进行的，不需要显式的转换**</font>

```c++
CGirl(int num ) 
{
     m_numm = num ;m_name.clear() ; m_weight  = 0 ;
}
```



```c++
CGirl g1(8);     // 常规的写法。

CGirl g1 = CGirl(8);  // 显式转换。

CGirl g1 = 8;     // 隐式转换。

CGirl g1;     // 创建对象。
g1 = 8;      // 隐式转换，用CGirl(8)创建临时对象，再赋值给g。 这种既会调用默认构造函数也会进行类型转换
```



<font color = blue>**但是注意:如果自动类型转换有二义性，编译将报错。**</font>

```c++
short func()
{return 10 ;}

int main()
{
    CGirl g1 = 10 ;
    //报错 
}
```

如果说在`CGirl`类中既有`double m_weight` 又有 `int m_num` ,且**同时定义两种变量数据类型的自动类型转换函数**, 编译器将会报错因为它不知道该调用哪个类型转换函数

###     16.8.1 explicit

将构造函数用作自动类型转换函数似乎是一项不错的特性，但有时候会导致意外的类型转换。(比如我上面说的例子)<font color = blue>explicit关键字用于关闭这种自动特性，但仍允许显式转换</font>。

```c++
explicit CGirl(int bh);

CGirl g=8;    // 错误。

CGirl g=CGirl(8); // 显式转换，可以。

CGirl g=(CGirl)8; // 显式转换，可以。
```

在实际开发中，<font color = purple>**如果强调的是构造，建议使用explicit，如果强调的是类型转换，则不使用explicit。**</font>