
## 内联函数

### 定义与使用

#### 常规函数的调用过程

&nbsp;&nbsp;编译过程的最终产品是可执行程序--由一组机器语言指令组成。运行程序时，操作系统将这些指令载入到计算机内存中，因此每一条指令都有特定的内存地址。有时，比如说遇到循环或者判断语句，会跳过一些指令，向前或向后跳到特定的地址。常规函数调用也使得程序跳转到下一个地址--函数的地址，并在函数结束时返回。

&nbsp;&nbsp;更具体的过程是执行到函数调用的指令的时候，程序会立即把这个函数调用的指令的地址存储起来，将函数的参数存到堆栈中，然后跳转到标记函数起点的内存单元，逐步执行函数体，如果函数有返回值的话，还会将返回值放到寄存器里。随后调回到之前存储的指令地址，继续执行下一个指令。这就好比是正在阅读一本书，阅读的过程中突然遇到了脚注，然后标记一下，转而去阅读脚注，阅读完之后再回到之前标记的地方继续阅读。

&nbsp;&nbsp;对于比较大型的程序，函数的调用往往存在千百万甚至上亿次，这每一次的跳转都需要造成一些系统的开销，最终影响整个系统的性能。当然，对于平常个人工程实践里的小型代码来说，由于现在PC的性能普遍还可以，所以这些开销基本上可以忽略不计。当然，对于C语言来说，一些经常用到的函数，往往采用宏定义进行预编译来减少这种跳转的开销。比如编写一个程序做一些数学运算，往往类似乘方，开根号这些函数被经常用到，在程序最开头加上“#define SQUARE1(x) x*x”，这样程序在遇到SQUARE(x)的时候，就会自动转换成x*x，然而这样做本质上只是一种文本替换，从下面的分析中你能看到它所面临的困境，而使用内联函数却可以避免这些问题。

#### 内联函数的使用方法

&nbsp;&nbsp;C++内联函数提供了另外一种选择，顾名思义，内联函数的编译代码与其它程序代码“内联”起来了，编译器将使用相应的函数代码替换函数调用。因此函数在执行到内联函数时，程序无需再跳到另外一个位置处执行代码，执行完再调回来，很明显其运行速度会比常规函数要快一些。但是这么做的代价，就是程序会占用更多的内存。如果程序在10个不同的地方调用了同一个内联函数，那么这个内联函数就要创造10个副本。

&nbsp;&nbsp;所以，我们在看待内联函数的时候，应该看到其两面性，根据实际的应用场景去有选择地使用内联函数。这就好比是，如果你担心整型溢出，你可以把所有的整型变量都声明为long long，这样固然是很难溢出了，但是也会带来很多的内存消耗。因此，如果执行函数代码的时间比处理函数调用机制的时间长，则节省的时间将只占整个过程的一小部分。如果代码执行的时间很短，则内敛调用就可以节省非内联调用使用的大部分时间。

&nbsp;&nbsp;刚才只是从宏观上讲了为什么要有内联函数，以及它的利弊。现在来看看如何使用。要使用这项特性，必须采取下述措施之一：
- 在函数声明前加上关键字inline;
- 在函数定义前加上关键字inline。

&nbsp;&nbsp;通常的做法是直接省略原型，将整个定义（即函数头和所有函数代码）放在本应提供原型的地方。
并不是所有的编译器都支持内联函数，有些编译器就没有启用或者实现这种特性。此外，有时候编译器会拒绝程序员将某些函数作为内联函数，它可能认为该函数过大或注意到函数调用了自己，因为内联函数不允许递归。

&nbsp;&nbsp;下面的代码展示了使用内联函数的一个demo。
```
//Date: 2020/03/08
//Author: Pengqi
//Reference: C++ Primer Plus P254-255

#include<iostream>

using std::cout;
using std::endl;

inline double square(double x) { return x * x;}

int main() {
    double a, b;
    double c = 13.0;

    a = square(5.0);
    b = square(4.5 + 7.5);
    cout << "a = " << a << ", b = " << b <<endl;
    cout << "c = " << c;
    cout << ", c square = " << square(c++) << endl;
    cout << "Now c = " << c <<endl;

    return 0;
}
```
&nbsp;&nbsp;程序的运行结果是
```
./inline_test_1
a = 25, b = 144
c = 13, c square = 169
Now c = 14
```
&nbsp;&nbsp;输出表明，内联函数和常规函数一样，也是按值传递的。
尽管程序没有提供独立的原型，但C++原型特性仍起作用。这是因为在函数首次使用前出现的整个函数定义充当了原型。这意味着可以给square()传递int或long值，将值传递给函数前，程序自动将这个值强制转换为double类型。

### 内联函数与C宏

&nbsp;&nbsp;inline工具是C++新增的一个特性，C语言使用预处理器#define来提供宏--内联代码的原始实现。下面一段代码将可以很好地展示二者的区别。

```
//Date: 2020/03/07
//Author: Pengqi

#include<iostream>

using std::cout;
using std::endl;

#define SQUARE1(x) x*x
#define SQUARE2(x) ((x)*(x))

inline double square(double x) { return x*x; }

int main() {
    int a =3;
    cout << SQUARE1(5.0) << endl;  // 5.0*5.0
    cout << SQUARE1(4.5 + 7.5) << endl;  //4.5 + 7.5*4.5 + 7.5
    cout << SQUARE1(a++) << endl; // c++*c++

    cout << SQUARE2(5.0) << endl;
    cout << SQUARE2(4.5 + 7.5) << endl;
    cout << SQUARE2(a++) << endl;

    cout << square(5.0) << endl;
    cout << square(4.5 + 7.5) << endl;
    cout << square(a++) << endl;
}
```

&nbsp;&nbsp;编译

```
$ g++ inline_test.cpp -o inline_test
inline_test.cpp:15:22: warning: multiple unsequenced modifications to 'a'
      [-Wunsequenced]
    cout << SQUARE1(a++) << endl; // c++*c++
                     ^~
inline_test.cpp:6:20: note: expanded from macro 'SQUARE1'
#define SQUARE1(x) x*x
                   ^ ~
inline_test.cpp:19:22: warning: multiple unsequenced modifications to 'a'
      [-Wunsequenced]
    cout << SQUARE2(a++) << endl;
                     ^~
inline_test.cpp:7:22: note: expanded from macro 'SQUARE2'
#define SQUARE2(x) ((x)*(x))
                     ^   ~
2 warnings generated.
```
&nbsp;&nbsp;运行结果：
```
./inline_test
25
45.75
12
25
144
30
25
144
49
```
&nbsp;&nbsp;从上面的运行结果可以看出，SQUARE1()无法处理(4.5 + 7.5)和(c++)这两种情况，因为宏定义仅仅只是简单的文本替换。
SQUARE2()虽然在(4.5+7.5)能得到正确的结果, 但是其本身接收参数的方式并不是值传递，以至于传入c++的时候，仍然将c递增了两次。而对于内联函数square()，则是先传递c进入到函数中，再递增。