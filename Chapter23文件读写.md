# Chapter23 文件读写

<font size=4>**类的继承关系图:**</font>

<img src="./../../../Pictures/Screenshots/屏幕截图 2024-08-16 111731.png" style="zoom:50%;" />

## 23.0**操作文本文件和二进制文件的一些细节：**

1）在windows平台下，文本文件的换行标志是`\r\n`。

2）在Linux平台下，文本文件的换行标志是`\n`。

3）在windows平台下，如果以文本方式打开文件，写入数据的时候，系统会将"\n"转换成"\r\n"；读取数据的时候，系统会将"\r\n"转换成"\n"。 如果以二进制方式打开文件，写和读都不会进行转换。

4）在Linux平台下，以文本或二进制方式打开文件，系统不会做任何转换。

5）以文本方式读取文件的时候，遇到换行符停止，读入的内容中没有换行符；以二制方式读取文件的时候，遇到换行符不会停止，读入的内容中会包含换行符（换行符被视为数据）。

6）在实际开发中，从兼容和语义考虑，一般：

​	**a）以文本模式打开文本文件，用行的方法操作它；**

​	**b）以二进制模式打开二进制文件，用数据块的方法操作它；**

​	**c）<font color =red>以二进制模式打开文本文件和二进制文件，用数据块的方法操作它，这种情况表示不关心数据的内容。（例如复制文件和传输文件)</font>**



## 23.1 文本文件



### 	23.1.1 写入文本文件(fout)

**包含头文件：**`#include <fstream>`

**类：**`ofstream（output file stream）`



**ofstream打开文件的模式（方式）：(但是对于ofstream，不管用哪种模式打开文件，如果文件不存在，都会创建文件。)**

```c++
ios::out     //  缺省值：会截断文件内容。

ios::trunc    // 截断文件内容。（truncate）

ios::app     // 不截断文件内容，只在文件未尾追加文件。（append）
```



#### 		23.1.1.1 一些要点

- 可以实现创建`filename` 变量 , 可以为`string`类也可以是C类型的字符串, fout.open()内部进行了函数的重载。
- **打开文件操作一定要进行判断是否打开成功。**

 失败的原因主要有：1）目录不存在；2）磁盘空间已满；3）没有权限，Linux平台下很常见。

- 文件名一般用全路径，书写的方法如下：

     1）<font color =red>"D:\data\txt\test.txt"    // 错误。</font>

     2）R"(D:\data\txt\test.txt)"  // 原始字面量，C++11标准。

     3）"D:\\data\\txt\\test.txt"  // 转义字符。

     4）"D:/tata/txt/test.txt"    // 把斜线反着写。

     5）"/data/txt/test.txt"     // Linux系统采用的方法。

  

**示例:**

```c++
using namespace std;

int main()
{
	string filename = "C:\\Users\\32939\\Desktop\\test.txt";
	//ofstream fout(filename, ios::app); 相当于下面两行代码
	ofstream fout;
	fout.open(filename, ios::app);
	if (fout.is_open() == false)
	{
		fputs("error!\n", stderr);
		exit(1);
	}
	fout << "hello gwen\n";
	fout << "hello jack";
	fout.close();
}
```



### 	23.1.2 读取文本文件(fin)

包含头文件：`#include <fstream>`

类：`ifstream`

**ifstream打开文件的模式（方式）：**

对于ifstream，如果文件不存在，则打开文件失败。

```c++
ios::in        //缺省值。

ios::binary    //以二进制方式打开文件。
```



**示例:/**/

```c++
#include<iostream>
#include<fstream>// ifstream类需要包含的头文件。
#include <string>// getline()函数需要包含的头文件。


using namespace std;

int main()
{
	string filename = "C:\\Users\\32939\\Desktop\\test.txt";
	ifstream fin; 
	fin.open(filename, ios::in);
	if (fin.is_open() == false)
	{
		cout << "open errpor!\n";
		exit(1);
	}
    .........
}
```



**第一种写法:**

```c++
    string buffer;
	while (getline(fin, buffer))
	{
		buffer += '\n';
		cout << buffer;
	}
```



**第二种写法:**

```c++
	char buffer2[101];
	while (fin.getline(buffer2 ,100))
	{
		
		cout << buffer2<<'\n';
	}
//我倒觉得不如C标准输入输出 因为fin的成员函数的返回值也是一个istream的类 不好判断读取的字符串的长度
```



**第三种写法:**

```c++
	string buffer3;
	while (fin >> buffer3)
	{
		cout << buffer3<<'\n';
	}
	//第三种写法 ,这种写法遇到空字串貌似就进入循环
```





## 23.2 二进制文件



### 	23.2.1 文本文件和二进制文件的差别

<font size=4>**文本文件:**</font>

存放的字符串, 以行的形式组织数据, 单个字符有意义,

例如:代码的源文件

<font size=4>**二进制文件:**</font>

存放的不一定是字符串, 以数据类型组织数据 ,单个字符无意义。 在记事本中打开出现乱码 , 因为记事本无法清楚二进制文件的组织形式

在磁盘中, 绝大多数文件为二进制文件, 例如:音视频 , 可执行文件等等.....



### 	23.2.2 写入二进制文件

- 打开方式需要加上`ios::binary` , 具体这么写↓

```c++
ofstream fout; 
fout.open(filename, ios::binary | ios::app); //表示追加并进行二进制书写
```

- 写入数据需要调用`fout.write()`

```c++
if (fout.is_open() == false)
{
    fputs("error\n", stderr);
	exit(1);
}
fout.write((const char*)&g1, sizeof(g1));
fout.write((const char*) & g2, sizeof(g2));
```



### 	23.2.3 读取二进制文件

- 读取数据需要调用`fin.read()`

```c++
ifstream fin; 
fin.open(filename, ios::binary | ios::app);

if (fin.is_open() == false)
{
    fputs("error\n", stderr);
    exit(1);
}
girl g;
while (fin.read((char*)&g, sizeof(g)))
{
    cout << g.memo << " " << g.name << " " << g.no << " " << g.weight << endl;
}

```





## 23.3 文件操作-随机存取



### 	23.3.1 fstream类

`fstream`类既可以读文本/二进制文件，也可以写文本/二进制文件。

`fstream`类的缺省模式是`ios::in | ios::out`，如果文件不存在，则创建文件；**但是，不会清空文件原有的内容。**

普遍的做法是：

1）如果只想写入数据**，用`ofstream`；如果只想读取数据，用`ifstream`；如果想写和读数据**，用`fstream`，这种情况不多见。不同的类体现不同的语义。

2）在Linux平台下，文件的写和读有严格的权限控制。（需要的权限越少越好）



### 	23.3.2  文件的位置指针

对文件进行读/写操作时，文件的位置指针指向当前文件读/写的位置。

很多资料用“文件读指针的位置”和“文件写指针的位置”，容易误导人。不管用哪个类操作文件，<font color =red>**文件的位置指针只有一个**</font>。

**1）获取文件位置指针**

`ofstream`类的成员函数是`tellp()`；`ifstream`类的成员函数是`tellg()`；`fstream`类两个都有，效果相同。

`std::streampos tellp();`

`std::streampos tellg();`

**2）移动文件位置指针**

`ofstream`类的函数是`seekp()`；`ifstream`类的函数是`seekg()`；`fstream`类两个都有，效果相同。

**方法一：**

```c++
std::istream & seekg(std::streampos _Pos); 

fin.seekg(128);  // 把文件指针移到第128字节。

fin.seekp(128);  // 把文件指针移到第128字节。

fin.seekg(ios::beg) // 把文件指针移动文件的开始。

fin.seekp(ios::end) // 把文件指针移动文件的结尾。
```

**方法二：**

```c++
std::istream & seekg(std::streamoff _Off,std::ios::seekdir _Way);

在ios中定义的枚举类型：
enum seek_dir {beg, cur, end}; // beg-文件的起始位置；cur-文件的当前位置；end-文件的结尾位置。

fin.seekg(30, ios::beg);  // 从文件开始的位置往后移30字节。

fin.seekg(-5, ios::cur);   // 从当前位置往前移5字节。

fin.seekg( 8, ios::cur);   // 从当前位置往后移8字节。

fin.seekg(-10, ios::end);  // 从文件结尾的位置往前移10字节。
```



## 23.4 文件操作-打开文件的模式（方式）

如果文件不存在，各种模式都会创建文件。

```python
ios::out       1）会截断文件；2）可以用seekp()移动文件指针。

ios:trunc      1）会截断文件；2）可以用seekp()移动文件指针。

ios::app      1）不会截断文件；2）文件指针始终在文件未尾，不能用seekp()移动文件指针。

ios::ate       打开文件时文件指针指向文件末尾，但是，可以在文件中的任何地方写数据。

ios::in        打开文件进行读操作，即读取文件中的数据。

ios::binary    打开文件为二进制文件，否则为文本文件。
```

> 注：ate是at end的缩写，trunc是truncate（截断）的缩写，app是append（追加）的缩写。



## 23.5 文件操作-缓冲区及流状态



### 	23.5.1文件缓冲区

文件缓冲区（缓存）是系统预留的内存空间，用于存放输入或输出的数据。

根据输出和输入流，分为输出缓冲区和输入缓冲区。

注意，在C++中，每打开一个文件，系统就会为它分配缓冲区。**不同的流，缓冲区是独立的**。

在缺省模式下，输出缓冲区中的数据满了才把数据写入磁盘，但是，这种模式不一定能满足业务的需求。

**输出缓冲区的操作：**

1）`flush()`成员函数

刷新缓冲区，把缓冲区中的内容写入磁盘文件。

2）`endl`

注意不只有换行，还可以刷新缓冲区。

```c++
fout<<"words"<<endl;
```

3）`unitbuf`

`fout << unitbuf;`

设置`fout`输出流，在每次操作之后自动刷新缓冲区。

4）`nounitbuf`

`fout << nounitbuf;`

设置`fout`输出流，让`fout`回到缺省的缓冲方式。



### 	23.5.2 流状态

**流状态有三个：eofbit、badbit和failbit，取值：1-设置；或0-清除。**

**当三个流状成都为0时，表示一切顺利，good()成员函数返回true。**

**1）eofbit**

> 当输入流操作到达文件未尾时，将设置`eofbit`。
>
> eof()成员函数检查流是否设置了`eofbit`。

**2）badbit**

> 无法诊断的失败破坏流时，将设置badbit。（例如：对输入流进行写入；磁盘没有剩余空间）。
>
> bad()成员函数检查流是否设置了badbit。

**3）failbit**

> 当输入流操作未能读取预期的字符时，将设置failbit（**非致命错误，可挽回，一般是软件错误，例如：想读取一个整数，但内容是一个字符串；文件到了未尾**）I/O失败也可能设置failbit。
>
> fail()成员函数检查流是否设置了failbit。
