### 函数指针
* 函数指针声明来接收函数，如下：
```
void test(void(*p)(int, int)) {
    // 以下3种写法（调用方法）都一样，因为这是一个明确的函数指针
    p(9, 9);
    (*p)(9, 9);
    (p)(9, 9);
    // 但不能用 (&p)(9, 9);
}
```

* strlen， 计算长度不会会加上\0的长度，因此如果new一个新的char字符串，需要
```
new_src = new char[strlen(old_src) + 1]
```
char str[] = {'a', 'b', 'c'};和char *str = "abc"，其中str[]不会自动加'\0'，如果用printf可能打印异常。需要创建时手动加入'\0'。比如char str[] = {'a', 'b', 'c', '\0'}; 指针的写法会自动加上'\0';

* realloc需要传递原数据指针，因为不一定内存足够让新数据可以拼接到老数据，有可能需要整体拷贝到一段新地址；

* 为什么有sizeof还需要strlen
    * 比如用来计算一个数组的长度，如果用sizeof计算，定义的函数如果将int intarr[]作为形参，会被编译器优化成指针，如果用sizeof计算会得到错误的结果；
    
* 指针赋值时要特别注意栈区弹栈时内存被释放的问题，e.g.
```
void subStr(char **ret, char *src, int start, int end) {
    char *tmp = src;    // 定义临时指针，不破坏src指针
    char retStr[end - start];   // 合理分配内存，不要浪费
    
    int cnt = 0;
    for (int i = start; i < end; i++) {
        retStr[cnt] = *(tmp + i);
        cnt++;
    }
    
    // *ret = retStr;   // 不能这样写，弹栈后内存已被释放，或者可以用堆区的解决方案，比如把char retStr[end - start];改成char *retStr = malloc(end - start); 但需要在函数外自己free
    strcpy(*ret, retStr);
}
```
但简单写法应该如下：
``` 
void subStr(char *ret, char *src, int start, int end) {
    for (int i = start; i < end; i++) {
        *(ret++) = *(src + i);
    }
}
```
或
``` 
void subStr(char *ret, char *src, int start, int end) {
    strncpy(ret, src + start, end - start);
}
```
* new/delete和malloc/free
    * new会调用构造函数，malloc不会，同理delete会调用析构，free不会；
* 拷贝构造函数，同默认构造函数一样，默认存在
    * 类名 对象1 = 对象2；这样赋值时触发执行，会寻求对象2的地址对应的成员的值，再寻求对象1地址对应成员进行赋值；
    ```
    Person person1("张山", 33); // 栈区分配内存
    Person person2 = person1;   // 触发拷贝构造函数,person1和person2的地址不一样.不会触发构造函数
    
    Person person3;
    person3 = person1;  // 不会触发拷贝构造函数，可以自定义=运算符重载来控制这里的逻辑
    
    Person *person4 = new Person("李四", 44);
    Person *person5 = person4;  // 也不会触发拷贝构造函数，因为只是指针指向改变
    
    // 拷贝构造函数实际形式如下：(但如果自己实现了，会覆盖默认的拷贝构造)
    class Person {
    public: 
        // 这个形参也是传递进来的变量的旧地址，和这个函数里的this的地址不一样（this != &person）
        Persion(const Person &person) { // 常量引用，只读。
            this->name = person.name;
            this->age = person.age;
        }
    private: 
        char *name;
        int age;
    };
    ```
    * 注意事项：
        * 指针赋值不会触发；
        * 对象传递给函数做形参，也会调用拷贝构造函数；
* 常量指针与指针常量
    * 常量指针
    ```
    int num1 = 9;
    int num2 = 8;
    const int *numP1 = &num1;   // 不能修改存放地址所对应的值
    // *numP1 = 100;    // 会报错
    numP1 = &num2;      // 但允许重新指向
    ```
    * 指针常量
    ```
    int num1 = 9;
    int num2 = 8;
    int* const numP1 = &num1;
    *numP1 = 100;       // 允许修改存放地址所对应的值
    // numP1 = &num2;      // 不允许重新指向，会报错
    ```
* 浅拷贝与深拷贝
    * 浅拷贝（新地址和旧地址指向同一个堆区空间）代码，可能导致析构函数重复释放崩溃：
    ```
    class Person {
    public: 
        Person(char* name) {
            this->name = (char *)malloc(sizeof(char *) * 10);
            strcpy(this->name, name);
        }
        ~Person() {
            free(this->name);
            this->name = NULL;
        }
        Persion(const Person &person) {
            // person是旧地址， this新地址，被赋值的对象也是新地址
            // 新地址->name = 旧地址，是浅拷贝
            this->name = person.name;
            this->age = person.age;
        } // 拷贝构造函数执行完会创建出新地址this
    private: 
        char *name;
        int age;
    };
    
    void showPerson(Person person) {
        cout<<"showPerson 形参地址："<<&person<<endl; 
    }
    
    void main() {
        Person person("张三", 33);
        showPerson(person);
        showPerson(person);
        // 连续两次调用，会因为重复释放新地址而崩溃
    } // 如果showPerson，这里也会崩溃，main函数弹栈时会释放旧地址。就是说释放一次新地址，一次老地址，也会崩溃。因为在拷贝构造函数中，新地址和旧地址都指向堆区创建的同一片内存空间。
    ```
    * 深拷贝（每次拷贝构造函数创建新的空间）
    ```
    class Person {
    public: 
        Person(char* name) {
            this->name = (char *)malloc(sizeof(char *) * 10);
            strcpy(this->name, name);
        }
        ~Person() {
            free(this->name);
            this->name = NULL;
        }
        Persion(const Person &person) {
            // 深拷贝
            this->name = (char *)malloc(sizeof(char *) * 10);
            strcpy(this->name, person.name);
            this->age = person.age;
        } // 拷贝构造函数执行完会创建出新地址this
    private: 
        char *name;
        int age;
    };
    ```
    * 默认拷贝构造函数为浅拷贝；
    * 凡是涉及堆成员，必须使用深拷贝；
* static关键字
    * 可以通过类名调用静态成员（字段/函数），比如Person::name;
    * 静态属性必须要先初始化，再实现，实现时不需要增加static关键字；
    * 静态函数只能操作静态属性和方法（同Java），非静态可以操作静态；
* C++区域划分（栈区，堆区，代码区，全局区（静态区，常量区，字符串区））
    * 默认构造函数执行时，会在栈区开辟空间，暴露地址（this）；
    * 静态区没有this区分；
    ```
    class Student {
    public: 
        static int id;
        int age;
    };
    
    Student stu1;
    stu1.id = 888;
    Student stu2;
    stu2.id = 999;
    Student::id = 111;
    // 最后，stu1.id == stu2.id == Student::id == 111，这3个共享一块区域，在静态区
    ```
* C++没有默认值，不像Java，如果不赋值，默认值是系统值，可以给指针或整形赋值NULL，即0;
* 友元函数（实现的时候不需要加friend关键字）
```
class Person {
public: 
    Person(int a) : age(a)
    
    int getAge() {
        return this->age;
    }
    
    // 定义友元函数
    friend void updateFrientAge(Person *person, int age);
private:
    int age = NULL;
};

void updateAge(Person *person, int age) {
    // person->age = age; // 不能修改私有成员
}

// 友元函数的实现，可以访问所有私有成员
void updateFrientAge(Person *person, int age) {
    person->age = age;
}

int main() {
    Person person = Person(9);
    
    return 0;
}
```
Java通过友元类实现反射，Java每个类，内部还有一个友元的Class类来访问私有成员；

* 类继承
    * 默认私有继承，不能读取父类的成员变量(在类里面可以，外面不行)，如果要访问，需要写成：class Son : public Father{};
    * 支持多继承，出现多个父类有同样方法时需要手动规避二义性，如：mainActivity.BaseActivity2::show();
    * 虚继承也可解决歧义，最后都用祖先类的成员：class BaseActivity2 : virtual public Object {};
    * 构造函数实现：
    ```
    
    ```
* 多态（虚函数）：程序运行期间才能决定调用哪个类的函数
    * 形参是父类，可传入各种子类；
    * C++默认关闭多态（Java默认开启），需要通过虚函数开启。非虚函数默认会执行父类的结果；
    * 纯虚函数（类似（但不是）Java的抽象类）：virtual void init() = 0;
    * 如果子类没有重写父类的纯虚函数，自己会变成抽象类。如果一个类全是纯虚函数，那就类似（但不是，不能直接实例化，必须写一个继承的实现类）Java的接口；
* 模板函数
    * C++无泛型，模板函数非常类似Java的泛型
    ```
    template <typename T>
    void add(T num1, T num2) {
        cout<<"模板函数 = "<<num1 + num2<<endl;
    }
    ```
* 函数谓词
    * stl的set容器，基于红黑树结构，会自动排序；
    * 自定义谓词，用来比较对象，如下：
    ```
    // 参考stl的set源码的less和greater
    struct doCompareAction {
        
        boolean operator() (const Person& __x, const Person& __y) {
            return __x.id < __y.id;    
        }
    };
    
    void main() {
        set<Person, doCompareAction> setVar;
        
        // 构建对象
        Person p1("张三", 1);
        Person p2("李四", 2);
        Person p3("王五", 3);
        
        // 构建对象插入set容器
        setVar.insert(p1);
        setVar.insert(p2);
        setVar.insert(p3);
        
        // set迭代器遍历
        for (set<Person>::iterator it = setVar.begin(); it != setVar.end(); it++) {
            cout<<"name = "<<it->name.c_str()<<", id = "<<it->id<<endl;
        }
    }
    ```
* stl的map容器，会对key进行排序
    * #include<map>
    * map<int, string> mapVar; 
    * mapVar.insert(pair<int, string>(1, "1"));
    * mapVar.insert(make_pair(2, "2"));
    * mapVar.insert(map<int, string>::value_type(3, "3"));
    * mapVar[4] = "4";  // 该方法同样key会覆盖，前面几种不允许相同的key
    * 迭代器 for(map<int, string>::iterator it = mapVar.begin(); it != mapVar.end(); it++)
    * 查找：map<int, string> ::iterator findResult = mapVar.find(3); if (findResult != mapVar.end())  // 找到
* multimap,key可以重复，key重复的数据可以分组，key会进行排序；
* 仿函数 == 谓词，运算符重载了括号（对象后面+括号doCompareAction()），很像函数调用
    * 如果是多个参数，就叫多元谓词；
* 容器存入对象后，对象的生命周期：
    * 容器传入时，会调用拷贝构造函数，存进去的是个新对象（不同于Java）；
    ```
    int main() {
        vector<Person> = vectorVar;
        
        // main弹栈时析构第一次
        Person person = Person("张三");
        
        // main弹栈时insert弹栈，析构第二次
        vectorVar.insert(vectorVar.begin(), person);    // 第一次拷贝构造函数执行
        person.setName("李四");
        
        // newPerson被main函数弹栈，析构第三次
        Person newPerson = vectorVar.front(); // front里面的person是旧地址，外面的newPerson是新地址。 第二次拷贝构造函数执行
        
        cout<<"newPerson name = "<<newPerson.getName().c_str() <<endl;  // 输出的是张三
        
        return 0;
    }
    ```
* 函数适配器（解决equal_to需要比较的内容没有）
    * 无法使用：find_if(setVar.begin(), setVar.end(), equal_to<string>("Hello"), "Hello");
    * 需要改成：find_if(setVar.begin(), setVar.end(), bind2nd(equal_to<string>(), "Hello"));
* 算法包（C++的不像Java容器，它的算法包，stl容器包等是分开的）
    * 排序：sort(vectorVar.begin(), vectorVar.end(), less<int>());
    * 打乱：random_shuffle(vectorVar.begin(), vectorVar.end());
    * 容器拷贝：copy(vectorVar.begin(), vectorVar.end(), vectorRet.begin());
* C++线程
    * C++11自带Thread（封装了pthreads），原来有pthreads。JDK用的是pthreads，Android也一样;
    * VS,mingw没有pthreads这些,CLion配置Cygwin默认有，macOS, linux pathred, android ndk pathred都默认有；
    * pthreads使用参考：
    ```
    // 系统pthread_create函数声明参考
    int pthread_create(pthread_t *,     // 线程ID
                       const pthread_attr_t *, // 线程属性
                       void *(*)(void *), // 函数指针
                       void * // 给函数指针传递的内容，void*可以传递任何内容
                       )；
                       
    
    // 写一个用于异步执行的函数,相当于Java的Thread.run()函数
    void * customPthreadTask(void *pVoid) {
        int num = *static_cast<int *>(pVoid);   // 强转
        cout<<"异步线程执行，num = "<<num<<endl;
        
        return 0;   // 坑，注意这里必须返回，不然会报错且不好定位
    }
    ```
    * 线程锁：pthread_mutex_t mutex;    // 定义一个互斥锁，注意，需要初始化（使用pthread_mutex_init(&mutex, NULL);初始化，结束后还需要用pthread_mutex_destroy(&mutex);销毁），大多数平台不允许野指针.
    
    锁住的区域用： pthread_mutex_lock(&mutex);和pthread_mutex_unlock(&mutex);之间，类似synchnorized;
    * C++条件遍历+互斥锁 == Java的wait/notify
    ```
    // 阻塞队列
    template<typename T>
    
    class SafeQueue{
    private: 
        queue<T> queue;
        pthread_mutex_t mutex;  // 互斥锁
        pthread_cond_t cond;    // 条件变量，实现等待等功能，也不允许有野指针
    public:
        SafeQueue() {
            pthread_mutex_init(&mutex, 0);
            pthread_cond_init(&cond, 0);
        }
        
        ~SafeQueue() {
            pthread_mutex_destroy(&mutex);
            pthread_cond_destroy(&cond);
        }
        
        void add(T t) {
            pthread_mutex_lock(&mutex);
            
            queue.push(t);
            
            // pthread_cond_signal();  // 相当于Java的notify
            pthread_cond_broadcast();   // 相当于Java的notifyAll
            
            cout<<"notify to pop"<<endl;
            
            pthread_mutex_unlock(&mutex);
        }
        
        // 采用引用方式，不直接return值
        void get(T &t) {
            pthread_mutex_lock(&mutex);
            while (queue.empty()) { // 注意不要用if，防止被系统唤醒，走了wait后面的流程
                cout<<"I am waiting"<<endl;
                pthread_cond_wait(&cond, &mutex);   // 相当于Java的wait
            }
            
            cout<<"I am awake"<<endl;
            t = queue.front();
            queue.pop();    // 删除元素
            
            pthread_mutex_unlock(&mutex);
        }
    };
    ```
* 智能指针，不智能，少用。原理基于引用计数，本身是一个对象，在函数退栈时根据引用计数决定是否释放对象。存在循环引用问题。
    * shared_ptr
    ```
    Person *person = new Person();
    shared_ptr<Person> sharedPtr1(person);  // 加入智能指针，交给它去管理
    ```
    循环引用（依赖）问题如下：
    ```
    class Person1{
    public: 
        shared_ptr<Person2> person2;    // 引用了Person2的智能指针
        ~Person1() {
            count<<"Person1 析构函数执行"<<endl;
        }
    };
    
    class Person2{
    public: 
        shared_ptr<Person1> person1;    // 引用了Person1的智能指针
        ~Person2() {
            count<<"Person2 析构函数执行"<<endl;
        }
    };
    
    // 堆区开辟
    Person1 *person1 = new Person1();
    Person2 *person2 = new Person2();
    
    shared_ptr<Person1> sharedPtr1(person1);  //+1 = 1
    shared_ptr<Person2> sharedPtr2(person2);  //+1 = 1
    
    cout<<"前 sharedPtr1引用计数为："<<sharedPtr1.use_count()<<endl;
    cout<<"前 sharedPtr2引用计数为："<<sharedPtr2.use_count()<<endl;
    
    person1->person2 = sharedPtr2;  // 调用智能指针的重载的=号时会导致引用计数+1
    person2->person1 = sharedPtr1;
    
    cout<<"后 sharedPtr1引用计数为："<<sharedPtr1.use_count()<<endl;   // 2
    cout<<"后 sharedPtr2引用计数为："<<sharedPtr2.use_count()<<endl;   // 2
    
    ```
    * unique_ptr，设计的简单，独占式智能指针。不允许对象赋值（未重载=）；
    * wake_ptr，没有引用计数，不会存在循环引用问题；
    * 手写实现智能指针：
    ```
    using namespace std;
    
    template<typename T>
    class shared_ptr {
    private: 
        T *object;  // 指向管理的对象
        int *cnt;   // 引用计数
    public:
        shared_ptr（）{
            cnt = new int(1);
            object = NULL;
        }
        
        shared_ptr(T *t) : object(t) {
            cnt = new int(1);
        }
        
        ~shared_ptr() {
            if (--(*cnt) == 0) {
                if (object) {
                    delete object;
                    object = NULL;
                }
                delete cnt;
                cnt = NULL;
            }
        }
        
        // 拷贝构造函数
        shared_ptr(const shared_ptr<T> &p) {
            ++(*p.cnt);
            
            // 要把前一个对象释放，才在后面赋值下一个对象
            if (--(*cnt) == 0) {
                if (object) {
                    delete object;
                    object = NULL;
                }
                delete cnt;
                cnt = NULL;
            }
            
            object = p.object;
            cnt = p.cnt;
        }
        
        shared_ptr<T> & operator = (const shared_ptr<T> & p) {
            ++(*p.cnt);
            
            if (--(*cnt) == 0) {
                if (object) {
                    delete object;
                    object = NULL;
                }
                delete cnt;
                cnt = NULL;
            }
            
            object = p.object;
            cnt = p.cnt;
            
            return *this;   // 运算符重载的返回值
        }
        
        int use_count() {
            return *(this->cnt);
        }
    };
    ```
* 四种类型转换：
    * const_cast: const修饰的都可以转换。比如可以转成非常量指针。
    ```
    const Person *p1 = new Person();    // 常量指针
    // p1->name = "张山"；  // 报错，不能修改值
    Person *p2 = const_cast<Person *>(p1);  // 修改成非常量指针
    p1->name = "张山";
    ```
    * static_cast: 静态转换，编译器转换，指针相关的操作可以用，子父类的转换也可以用。常用来转换void *；
    ```
    class Father {
    public:
        void show() {
            cout<<"Father"<<endl;
        }
    };
    
    class Son : public Father {
    public:
        void show() {
            cout<<"Son"<<endl;
        }
    };
    
    Father *father = new Father;
    father->show();
    
    // 静态转换看左边
    Son *son = static_cast<Son *>(father);
    son->show();
    
    delete father;  // 注意回收时，是谁new出来就delete哪个
    ```
    * dynamic_cast:
    动态转换，运行期间转换。子父类多态可以用动态转换。动态转换必须让父类成为虚函数；
    ```
    class Father {
    public:
        // 动态转换父类必须是虚函数
        virtual void show() {
            cout<<"Father"<<endl;
        }
    };
    
    class Son : public Father {
    public:
        void show() {
            cout<<"Son"<<endl;
        }
    };
    
    // 动态类型转换要看右边
    // Father *father = new Father;    // 这种case会转换失败，因为已经new出了父类，转不了子类
    Father *father = new Son;   // 能转换成功
    Son *son = dynamic_cast<Son *>(father);
    if (son) {  // 转换成功
        son->show();
    } else {
        // 转换失败
    }
    
    delete father;  // 注意回收时，是谁new出来就delete哪个
    ```
    * reinterpret_cast 强制转换。比static_cast强大，static_cast能做的它都能做，同时附加新功能，比如把对象变成数值或把数值变成对象。
    ```
    Person *person = new Person();
    long personValue = reinterpret_cast<long>(person);
    
    Person *person2 = reinterpret_cast<Person *>(personValue);
    ```
    OpenCV就经常用这种方式，把很多指针转换成数值存储在Java层，后面再转换回去。
    * C++11新特性，nullptr，可以代替NULL，也可以传递给指针形参，解决了NULL的二义性。
* 宏定义
    * 1.预处理（宏展开，宏替换） -> 2.预编译（代码检查） -> 3.汇编 -> 4.链接
    * 基于文本替换，无函数栈操作；
* 