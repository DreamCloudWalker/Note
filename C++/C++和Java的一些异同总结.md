## Static

java的static和c++的static多数用法是相同，包括static method、static variable。其中static variable主要用于定义该类所有实例共用的一些数据（如值不变的状态变量等），主要目的是节省内存，因为不管new多少个实例，都共用一个static varible。static method在C++中一个很重要的应用是用于回调，但在java中一个很重要的用途是用于生成singleton实例。

java还有其他static的特殊用法，如下：

#### static 代码段
static代码段只有在类载入时调用一次，以后不管再怎么调用这个类，都不会再调用这个static代码段了。

static代码段主要用来初始化static variable。

#### static 内部类
C++的类定义是没有static类型的，而java有，如果在内部类前面加上static，并不会影响内部类的内存分布，而是表示内部类不能访问外部类，包括public的字段或方法。

