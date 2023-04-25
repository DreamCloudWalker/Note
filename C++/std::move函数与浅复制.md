写这篇文章源自实现一个简单矩阵计算，第一次搞明白了move操作到底是什么，该如何定义。顺带着，对浅复制内存操作的理解也更深了一层。

## 首先给一个粗暴的结论，**move操作特别像浅复制**。

这句话当然是不严谨的，但是初学者完全可以用这个理解构建整体印象，然后修正细节。



## **1.std::move是什么**

move的作用是偷取其他变量里的资源变为自己的，如内存、线程等等，而不必自己再从0获取。很显然这样可以节约一些程序开销。初次接触move函数是在读算法书时看别人的类定义时遇到的，一番搜索后，传送到了神秘的知识库，进而第一次比较全面的清楚了c++左值和右值的概念，解决了多年前看C++的书时看不懂的遗留问题。

[c++ move函数到底是什么意思？](https://www.zhihu.com/question/64205844)

在黄金C位的答案里，答者这样写

> std::move 并不会真正地移动对象，真正的移动操作是在移动构造函数、移动赋值函数等完成的，std::move 只是将参数转换为右值引用而已（相当于一个 static_cast）。

这句话当然是正确的。但是我作为初学者第一次看了之后依然一头雾水，然后呢？

然后答者觉得比较简单，就没有详细解释了。然而这并没有卵用，还是不清楚move该如何使用。于是继续搜寻，在许多其他的博客和解答里会提到下面的话

> move的本质就是帮助编译器选择重载函数, 告诉编译器"请尽量把此参数当做右值来处理"
> 跟**移动构造函数**和**移动赋值函数**的**实现**有关。

这句话仍然对初学者不够友好，所以当现在搞明白问题后，给一个总结

> std::move并不产生任何实际效果，其仅仅是把一个左值(可以认为程序里的大部分场合使用的都是左值)强制变为右值，让编译器调用处理右值的程序。这就是关键，你还得自己写处理右值的程序，也就是移动构造函数和移动赋值函数。使用std::move的核心就是你如何实现上述两个函数。也就是说，一个完整的move操作是这样的 Matrix A = std::move(B)，这个操作包含了右值转换，move构造函数，move赋值函数.

那么如何实现？还记得开篇说move和浅复制类似吗，现在解释来了。

假设有如下代码定义：

```cpp
class Matrix{
    private:
        T** m   = nullptr;
        int row = 0;
        int col = 0;
    public:
        Matrix(){
            //你自己定义的构造函数
        }
        //复制构造,深复制
        Matrix(const Matrix& rhs){
            row = rhs.row;
            col = rhs.col;

            m = new T*[row];
            for(int i=0;i<row;i++){
                m[i] = new T[col];
            }

            for(int i=0;i<row;i++)
                for(int j=0;j<col;j++){
                    m[i][j] = rhs.m[i][j];
            }
        }
        //浅复制
        void shallowCopy(const Matrix& rhs){
            Matrix temp;
            swap(temp,*this);
            this->m = rhs.m;
            this->row = rhs.row;
            this->col = rhs.col;
        }
        // deep copy assignment
        Matrix& operator=(const Matrix& rhs){
            Matrix temp(rhs);
            swap(*this,temp);
            std::cout<<"copy assignment"<<std::endl;
            return *this;
        }
```

上面是浅复制和深复制的实现。浅复制仅仅是让指针指向改变，深复制则要重新申请内存并逐个赋值。很明显的，深复制成本非常高。浅复制的问题是，如果让两个Matrix变量里的指针指向同一个地址，当一个变量被析构后，另一个变量再析构时将导致二次delete问题，即内存泄漏。这个问题的解决在更后面，先看move操作。

那么我们的move要如何写呢？既然move是偷资源，那偷完后(改变指向)，把源头清空不就好了？比浅复制更省事，直接不存在两个指针指向同一地址的问题。需要关注的是如何清空源头，又不影响其包含的资源

```cpp
 //move constructor      
 Matrix(Matrix&& rhs){
            row = rhs.row;
            col = rhs.col;
            m = rhs.m;
            
            //move后，将source内元素置空
            rhs.row = 0;
            rhs.col = 0;
            rhs.m = nullptr;
            rhs.~Matrix();

            std::cout<<"move constructor"<<std::endl;
        }
```

可以看到做法正是和浅复制一摸一样的操作，不同之处在于，之后需要把源头里的指针赋值为nullptr并显式调用析构函数。这样就实现了move的功能描述：

```text
你的已经变成我的，只剩下空值甚至变量都不复存在。我的还是我的
```

析构函数如下

```text
        ~Matrix(){
            if(m!=nullptr){
                for(int i=0;i<row;i++)                  
                    delete[] m[i];
                delete[] m;
                m = nullptr;
            }
        }
```

可以看到析构函数开头有nullptr判断，结合move构造函数的写法，一个指针原本指向的内存地址交给了其他变量，自己设置为nullprt，当自己面临析构时，nullptr就会告诉析构函数，我已经被抢光了，不必再做内存delete。

而move赋值则同copy赋值一样如法炮制

```text
        Matrix& operator=(Matrix&& rhs){
            if(this->m != rhs.m){
                Matrix temp(std::move(rhs));
                swap(*this,temp);
            }                                   
            std::cout<<"move assignment"<<std::endl;
            return *this;           
        }
```

开头的if判断是为了防止把自己move赋值给自己。你会问这是什么傻操作，什么情况会出现。右值累加赋值时，就会出现这种情况。比如 A = std::move(A) + B；当然这个操作需要重载加法函数，参考[这位博主](https://link.zhihu.com/?target=https%3A//blog.csdn.net/clangpp/article/details/38884953)注意他的代码有小bug，复制构造和赋值会内存泄漏



## 2.浅复制二次析构问题

回到上面说的浅复制二次析构，最开始我设想的解决办法是利用上面的nullprt赋值。想法很简单，当第一次析构后，指针被赋值为nullptr，这样第二次遇到时做一个判断就跳过delete。然后现实当然是被打脸了，根本不行。来自某位大佬的指点让我明白了原因，和如何解决。

这里涉及到一些基础知识问题，[程序栈和堆](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/wuaihua/p/7256872.html%23%3A~%3Atext%3D%E5%A0%86%E6%98%AF%E5%9C%A8%E7%A8%8B%E5%BA%8F%E8%BF%90%E8%A1%8C%2C%E5%AD%98%E6%94%BE%E4%B8%B4%E6%97%B6%E6%95%B0%E6%8D%AE%E7%9A%84%E5%9C%B0%E6%96%B9%E3%80%82)。在下面的这句简单内存地址申请里

```text
int** m = new T*[row];
```

栈和堆都参与了。m是一个栈变量，内存申请在堆上，该栈变量指向堆地址。

所以即便再写

```text
int** n = m;
```

这句的结果是定义了另一个栈变量，指向刚才申请好的内存地址。

**m和n是不同变量,修改m为nullptr不改变n的状态。**

这就是上面我的方法失败的原因，给浅复制的源头指针设置为nullptr不改变复制后指针的状态，其根本不是nullptr，所以当析构时，那句nullptr判断也就不起作用了。

那么解决方案呢？办法就是模仿智能指针，在类定义里加一个私有变量bool copy_flag;

如果是初次申请内存时的变量，copyflag设置为0，对以后所有的浅复制的变量,copyflag设置为1.析构时先检查copy_flag，然后只对源头变量才做delete。

这样就实现了我们想用浅复制，但浅复制对象销毁时不该影响源头变量的目的