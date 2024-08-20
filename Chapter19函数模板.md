# Chapter 19 函数模板



##    19.1 函数模板基础

模板的定义 , 和应用场景就先不说了 primer 之前有读过

```c++
//basic template
template <typename T >
函数定义
{
    ......
}
//示例
template <typename T >
T Swap(T& a, T& b)
{
	T temp = a;
	a = b;
	b = temp;
}
```



函数模板使用:

- #### 自动推导数据类型

```c++
int main()
{
	int a = 30, b = 10;
	Swap(a, b);
}
```

- #### 手动添加数据类型

```c++
int main()
{
    int a= 30 , b = 10 ;
    Swap<int>(a,b) ;
}
```



1. 可以为类的成员函数创建模板，但不能是**虚函数和析构函数**。

在创建函数模板时 , 有时候会存在函数定义过长的情况 ,这边建议可以换行书写

```c++
//构造函数
template <typename T, size_t BlockSize> 
MemoryPool<T, BlockSize>::MemoryPool()  
noexcept
{
    currentBlock_ = nullptr;
    currentSlot_ = nullptr;
    lastSlot_ = nullptr;
    freeSlots_ = nullptr;
}

//成员函数
template <typename T, size_t BlockSize>
MemoryPool<T, BlockSize>&
MemoryPool<T, BlockSize>::operator=(MemoryPool&& memoryPool)
noexcept
{
    if (this != &memoryPool)
    {
        std::swap(currentBlock_, memoryPool.currentBlock_);
        currentSlot_ = memoryPool.currentSlot_;
        lastSlot_ = memoryPool.lastSlot_;
        freeSlots_ = memoryPool.freeSlots;
    }
    return *this;
}
```



2. 函数模板支持多个通用数据类型的参数。

3. **函数模板支持重载**，可以有非通用数据类型的参数。





##   19.2 函数模板具体化

世界上事物 都不是绝对的 , 有普遍的就有特殊的

可以提供一个具体化的函数定义，当编译器找到与函数调用匹配的具体化定义时，将使用该定义，不再寻找模板。

- 具体化（特例化、特化）的语法：

```c++
template<>
void 函数模板名<数据类型>(参数列表)
    或者
template<> 
void 函数模板名 (参数列表)
{
	// 函数体。
}

```

**下面给出示例(仅声明部分)**

```c++
template <typename T>
void Swap(T& a, T& b);   // 交换两个变量的值函数模板。

template<> 
void Swap<CGirl>(CGirl& g1, CGirl& g2);   // 交换两个超女对象的排名。
```



###   	19.2.2编译器使用各种函数的规则：

1）具体化优先于常规模板，普通函数优先于具体化和常规模板。

2）如果希望使用函数模板，可以用空模板参数强制`Swap<>(a,b)`使用函数模板。

3）如果函数模板能产生更好的匹配，将优先于普通函数。





##   19.3 函数模板分文件编写

函数模板只是函数的描述，没有实体，<font color =red>**创建函数模板的代码放在头文件中**。</font>

函数模板的具体化有实体，编译的原理和普通函数一样，所以，声明放在头文件中，定义放在源文件中。





## 19.4 decltype关键字 (C++11)

在[C++](https://baike.baidu.com/item/C%2B%2B)11中，decltype[操作符](https://baike.baidu.com/item/操作符/8978896)，用于查询[表达式](https://baike.baidu.com/item/表达式/7655228)的数据类型。

语法：

```c++
decltype(expression) var;
```

**decltype分析表达式并得到它的类型，不会计算执行[表达式](https://so.csdn.net/so/search?q=表达式&spm=1001.2101.3001.7020)。<font color =red>函数调用也一种表达式</font>，因此不必担心在使用decltype时执行了函数。**

decltype推导规则（按步骤）：

> 1）如果expression是一个没有用括号括起来的标识符，则var的类型与该标识符的类型相同，包括const等限定符。
>
> 2）如果expression是一个函数调用，则var的类型与函数的返回值类型相同（函数不能返回void，但可以返回void *）。
>
> 3）如果expression是一个左值（能取地址）(要排除第一种情况)、或者用括号括起来的标识符，那么var的类型是expression的引用。
>
> 4）如果上面的条件都不满足，则var的类型与expression的类型相同。
>
> 如果需要多次使用decltype，可以结合typedef和using。
