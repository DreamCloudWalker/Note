## Static

java的static和c++的static多数用法是相同，包括static method、static variable。其中static variable主要用于定义该类所有实例共用的一些数据（如值不变的状态变量等），主要目的是节省内存，因为不管new多少个实例，都共用一个static varible。static method在C++中一个很重要的应用是用于回调，但在java中一个很重要的用途是用于生成singleton实例。

java还有其他static的特殊用法，如下：

#### static 代码段
static代码段只有在类载入时调用一次，以后不管再怎么调用这个类，都不会再调用这个static代码段了。

static代码段主要用来初始化static variable。

#### static 内部类
C++的类定义是没有static类型的，而java有，如果在内部类前面加上static，并不会影响内部类的内存分布，而是表示内部类不能访问外部类，包括public的字段或方法。



## C++ vector 与Java ArrayList

在Java的ArrayList.add(e)中，传入的是引用，因此当你传入e以后，再改变e的成员，则ArrayList里的e也同样会改变，因为本身e和ArrayList中的e就是同一个东西。

而C++的vector.push_back(e)则会调用拷贝构造函数，因此当你传入e以后，再改变e的成员，则vector里的e不会变，因为已经是两个对象了。
