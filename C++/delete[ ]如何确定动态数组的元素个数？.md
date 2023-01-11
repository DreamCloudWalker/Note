本文研究并解释delete [ ]操作符如何确定动态数组的元素个数。

本文转载自重庆大学2019级风流倜傥的于卓浩同学应作者的要求写的文章：
https://puluter.cn/20200331/delete/

[上述形容词是海洋饼干叔叔加的，与事实相符]



# 1. 简介

在面向对象编程中，我们经常会用到这样的动态分配数组：

```
Person* a = new Person[100];
```

在上述申请数组的过程中，我们使用到了new []这个表达式来完成，它会调用类的构造函数初始化这个数组的所有对象，有多少对象，就会执行多少次构造函数。如果我们用完了这个数组，想要释放空间，就需要调用：

```
delete[] a;
```

在这个过程中，我们使用了delete[]操作符来完成对象释放。
但是两个问题出现了：

- 如何知道a数组的内存空间大小？
- 如何知道要调用几次析构函数（a数组的元素个数）？

显然，想要知道数组有多长，我们必然要存下这个数组的长度。C++中也正是这么做的。本文讨论具体工作原理。



# 2. 结合代码的分析

> 注：本文适用于64位mingw。在32位mingw下后文内8字节应为4字节，long long应为int。

我们先定义一个自定义的类。

```
class Yu {
public:
    int iNumber;//Yu类内将只有这一个int变量
    //即 一个Yu对象的大小=一个int的大小=4字节
    Yu(){iNumber = 1;}
    ~Yu(){}
    ...
};
```

同时，在main中声明一个长度为len(20194134)的Yu类型的数组，再获取该数组开头的地址并打印。

```
const long long len = 20194134;
int main(){
  ...
  Yu* testArr = new Yu[len];
  char* p1 = (char*) testArr;
  cout << "Address for the Array: "<< testArr <<endl;
  ...
}
```

输出为：Address for the Array: 0x2670048

理论上，20194134个Yu对象，总共需要 20194134 x sizeof(Yu) = 80776536字节的空间。实际申请的堆（自由内存区）空间稍有出入，我们可以通过重载new [ ]操作符来研究。

我们重载new[]了操作符：

```
void* operator new[](size_t sz){
    printf("|Length: %lld\n|Real Size: %lld\n|Raw Size(int(4)*length): %lld\n|Gap: %lld\n",len,sz,len*4,(long long)sz-len*4);
    void* o = malloc(sz);
    return o;
}
```

在这个过程中，我们打印四个关键数值：

- 数组的长度(len)
- new[]过程中实际申请的内存空间大小(sz)
- 数组理论上需要的内存空间 （数组长度 x sizeof(Yu) = len x 4 )
- 实际空间与理论空间的差 ( sz - len x 4)

上述程序的执行结果为：

```
|Length: 20194134
|Real Size: 80776544
|Raw Size(int(4)*length): 80776536
|Gap: 8
```

这里会发现，编译器传递给new [ ]操作符的空间大小比实际需要多8个字节。而8个字节，恰好是一个long long变量的大小，实践中，**这8个字节用于存储动态数组的元素个数**。

为了弄清楚这8个字节的具体位置，我们重载了delete [ ]函数：

```
void operator delete[](void *o){
    cout<<"Destruct from: "<<o<<endl;
    free(o);
}
int main(){
    ...
    char* p1 = (char*) testArr;
    cout << "Address for the Array: "<< testArr <<endl;
    ...
    delete[] testArr;
    return 0;
}
```

最后几行的输出为：

```
Address for the Array: 0x2670048
...
Destruct from: 0x2670040
```

成了！我们发现，解构时得到的地址（0x2670040）恰好是数组的地址（0x2670048）减8。

即：0x2670048 = 0x2670040 + 8

接下来，我们更进一步，取出数组地址-8对应地址的一个long long变量，看一下它的值会是什么.

接下来获取该数组前的8字节，识别为long long并打印。

```
int main(){
    ...
    char* p1 = (char*) testArr;
    cout << "Address for the Array: "<< testArr <<endl;
    cout<< "The long long before the Array: " << *(long long*)(p1-len_ll)<<endl;
    delete[] testArr;
    ...
}
```

输出为：

```
Address for the Array: 0x2670048
The long long before the Array: 20194134   【这个值就是数组长度】
Destuct from: 0x2670040
```

代码给出的结果证明了前述的猜想：**自定义类数组前的8个字节，是一个long long类型的变量，储存了该数组的长度。**



# 3. 结论

![img](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/37812bcf20d74ef6833588f5c39be13b~noop.image?_iz=58558&from=article.pc_detail&x-expires=1674026127&x-signature=CRnmHsQPjXw%2Fg5k3v2xcPcGZl7g%3D)



我们以Yu* a = new Yu[2]为例进行说明。表面上，我们需要sizeof(Yu) x 2共8个字节的空间，但事实上，new [ ]操作符会从堆里申请8 + 8 = 16个字节的空间。其中，前8个字节用于存储数组的元素个数，后续空间用于存放数组元素。具体到本例，变量a得到的是数组首元素的地址，它事实上等于真实的堆空间地址 + 8！

当delete [ ]a被执行时：

- delete [ ]操作符会把a值 - 8，获得真实的**堆空间首地址**；
- 从堆空间首地址获得数组的**元素个数**（本例为2）；
- 依据元素个数及a值逐个执行全部数组元素的析构函数；
- 最后，以堆空间首地址为依据，通过free( )函数向操作系统归还堆空间。

本例中，如果执行delete a而不是delete [ ]a，可能导致两个后果：

- 仅有数组的首元素被正确析构；
- 释放堆空间时向操作系统提供的地址是不正确的，后果未知。

正是基于上述理由，书里反复强调，new/delete, new [ ] /delete [ ]要配对使用。



# 4. 完整实验代码

```
#include <iostream>
#include <string>
using namespace std;

const long long len = 20194134;

class Yu {
public:
    int iNumber;
    Yu(){iNumber = 1;}
    ~Yu(){}
    //重载 new[]操作符
    void* operator new[](size_t sz){
        /*打印四个关键数据：
        ①数组的长度
        ②new[]过程中申请的内存空间大小
        ③数组实际需要的内存空间
        ④ ②、③的差值
        */
        printf("|Length: %lld\n|Real Size: %lld\n|Raw Size(int(4)*length): %lld\n|Gap: %lld\n",len,sz,len*4,(long long)sz-len*4);
        //完成内存分配
        void* o = malloc(sz);
        return o;
    }
    //重载delete[]操作符
    void operator delete[](void *o){
        //打印析构的开端地址
        cout<<"Destruct from: "<<o<<endl;
        //完成内存释放
        free(o);
    }
};

int main(){
    // 打印long long的大小
    int len_ll = sizeof(long long);
    cout<<"Size of long long: "<<len_ll<<endl;
    //生成一个长度为len(20194134)的Yu类型的数组
    Yu* testArr = new Yu[len];
    //获取该数组开头的地址并打印
    char* p1 = (char*) testArr;
    cout << "Address for the Array: "<< testArr <<endl;
    //获取该数组前的8字节，识别为long long 并打印
    cout<< "The long long before the Array: " << *(long long*)(p1-len_ll)<<endl;
    //释放数组空间
    delete[] testArr;
    return 0;
}
```

# 完整输出：

```
Size of long long: 8
|Length: 20194134
|Real Size: 80776544
|Raw Size(int(4)*length): 80776536
|Gap: 8
Address for the Array: 0x2670048
The long long before the Array: 20194134
Destuct from: 0x2670040
```



本案例节选自作者编写的教材及配套实验指导书。

《C++编程基础及应用》（高等教育出版社，出版过程中）

《Python编程基础及应用》，高等教育出版社

《Python编程基础及应用实验教程》，高等教育出版社