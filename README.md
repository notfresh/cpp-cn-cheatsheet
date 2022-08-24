# cpp8gu
cpp八股文，冲！

列举了一些问题，答案心里知道，简单的就不写答案了。有热心人可以帮忙写答案。



# 1 const的用法

# 2 构造函数的explict关键字

# 3 inline的内涵

# 4 friend的用法

# 5 类型转换4种函数

# 6 右值的内涵

# 7 初始化列表作为构造函数的参数

# 8 多态的原理

# 9 虚继承的问题

# 10 delete this合法吗

# 11 如何声明一个只能在栈上或者堆上创建的类？

# 12 智能指针的用法？各类智能指针又是怎么实现的呢?



# 13 如何查有没有内存泄漏？

这个我真的不会。



# 14 智能指针适用于什么场景?不适用于什么场景?



# 15 c++智能指针多线程下为什么会影响性能？



# 16 智能指针shared_ptr，线程安全性， 智能指针的线程安全性又如何呢?





# 17 类似于智能指针的例子在C++中还有别的吗?





# 18 智能指针的实现

```
// Created by zxzx on 2020/10/17.
//

#include <iostream>
#include <memory>
#include <string>
using namespace std;

template <typename T>
class SmartPrt{
public:
    explictSmartPrt(T* p=0) { // 为什么要声明explict, 害怕隐式调用引起的类型转换, 可以较少用户代码错误!
        m_refCount = new int(1);// 我们假设这个传入的指针一定是刚刚分配的堆内存的值，如果传入一个已有的指针，那么应该报错，因为可能会重复释放
        m_p = p;
    };

    SmartPrt(const SmartPrt& sp){
        m_p = sp.m_p; // 指向同一个内存位置
        m_refCount = sp.m_refCount;
        ++*m_refCount; // 引用数加1
    }

    SmartPrt& operator=(SmartPrt& p){
        if(this==&p) return *this; // 注意防止自己给自己赋值！！！
        decr();
        ++*p.m_refCount;
        m_p = p.m_p;
        m_refCount = p.m_refCount;
        return *this;
    }

    T* operator->(){
        if(m_p) return m_p;
        throw runtime_error("null pointer1");
    };

    T& operator*(){
        if(m_p)
            return *m_p;
        throw runtime_error("null pointer2");
    }

    ~SmartPrt(){
        decr();
        cout << "deconstructor" << endl;
    }

    int getRefCount(){
        return *m_refCount;
    }
private:
    T* m_p;
    int* m_refCount; // 这个指针

    void decr(){
        (*m_refCount)--;
        if((*m_refCount) == 0){
            cout << " free" << endl;
            delete m_p;
            delete m_refCount;

        }
    }
};
```



# 19 什么时候用 unique_ptr?

- unique_ptr的使用场景，最常见的就是拥有一个一个对象的所有权，就像传统C指针的最基础用法，保存一个申请于堆上的对象



#  20 右值和移动构造函数的意义

右值到底是什么?  
从字面上理解, 右值就是放在等号右边, 只能被赋值给变量, 而自己不能被赋值的"东西"的统称.  
比如一个直接量, 一个数字, 一个字符串, 一个没有名字的自定义类型临时对象,甚至说, 一个const类型的变量, 那都是右值.

右值的意义, 在于什么地方? 右值的意义, 在于(1)提升传参效率(2)为了编程方便.

先说个简单的, 为了编程方便. 右值是不能被轻易引用的. 一个直接量之不能被传入一个要求按引用传参的函数.  
这样就很麻烦, 要先声明一个常量引用, 然后再把这个引用传进去.

调用函数传参的时候, 有两种方式, 一个是按引用, 一个是按值.  
按值传参, 就会调用拷贝构造函数.

如果一个自定义类体积很大, 或者成员很多, 调用拷贝构造函数, 成本就很高.  
如果这个对象的目的就是单纯为了传参, 而不需要保留原来的值, 那么成本就确实显得很高, 调用构造函数毫无必要.

为了能顾把右值当做形式参数使用, 定义了 T&& 的形式作为右值. 也就有了相应的右值构造函数,或者叫转移构造函数,  
它的目的就是满足一个变量仅仅作为传参使用, 而不必另做备份之用.

右值有的时候可以变成左值, 比如当一个形参是右值,然后被传入另一个函数内, 在这个函数里, 它就是左值.

使用move 可以把一个左值变成一个右值. move()函数.

有一个疑问就是, 既然凭借引用可以传参, 为什么还有右值传参呢? 说到底, 还是为了方便直接量, 或者临时对象.

举个例子

int main() { 
​     string a; 
​     a = "Hello"; 
​     std::vector<string> vec; 
​     vec.push_back("World adaadsfadfasdfadfadfadsfadsfadfadfadsfasdfasdfasdfadsf");  
​    }

使用一个直接量的时候, 系统会默认先使用移动构造函数, 如果没有的话,就是调用拷贝构造函数.(成本相对高).  
编译器是可以自动识别左值和右值的.

最后补充一个内容. 为什么移动构造函数要用swap 而不是 检查是否是同一个对象呢?  
因为每次检查是否是自身, 有成本, 而且真正发生频率也不高, 而且swap可以复用代码, 并且延迟释放原来指针. 总之, 好处大于坏处.



# 21 new和malloc的区别

一个是c语言里的关键字,一个是c++里的关键字.  
malloc分配一块内存, 需要指定内存大小, 返回一个分配好的内存指针.  
而news是一个运算符, 可以被重载, 它成功返回的也是一个指针.  
但这两者的指针,返回的类型是不一样的, malloc返回的是无类型的指针,需要转换, 而new则本身返回的是带有类型的指针, 它的语法是,new加上类型的构造函数, 连起来解读的意思,就是new运算符先去找一块内存,或者接受一个内存地址, 接下来由构造函数对这块内存进行初始化.相比之下,
malloc就原始的多,它什么也不做.  
因此上, new运算符默认应该是调用malloc的, new运算符是为了支撑面向对象开发,而设计的运算符.

因此, new是和构造函数成对出现的, 它的反操作就是delete, delete和析构函数一起出现, 而malloc的则是free.  
当然,他们分配内存的位置, 正如我们所说, new底层一般是调用malloc的,所以他们的内存分配位置在动态内存区, 也就是堆区.
所谓的new关联的一个自由存储区, 它的范围概念大于堆内存, 这块我还不是很熟悉. 但凡使用new关键字的操作的内存区域,都是自由存储区.  
new关键字还可以传入一个地址, 从而不分配内存, 在stl编程里面,就有展示相关的语法. 比如construct方法里面.
它的意思是,单纯根据一块原始地址去初始化内存空间,不必再次分配, 这种情况是在批量预分配内存的情况下出现的, 在stl容器中,大行其道,非常的常见.

在异常处理方面, malloc分配内存失败,会返回null, 所以我们检测是否分配成功, 也是判断指针知否为null, 而new
关键字一旦失败,则会抛出异常. 这是其中的区别,new抛出的异常是 bad_alloc. 如下:


​    

    try
    {
        int *a = new int();
    }
    catch (bad_alloc)
    {
        ...
    }


c++里面的异常系统和Java的还是不是很像, 至于有没有异常继承体系[参考1],这一块我还不是很熟悉.

在复合数据结构上, 例如数组, malloc的处理办法是, 整体分配一块内存, 然后到时候统一释放.
也就是说,调用malloc得到的这个指针维护了分配的内存的大小, free的时候,自然会去释放其分配时候的大小(这块存疑, 有待确定).
new关键字,则由针对数组的特殊语法, 它有指定分配的元素数量,为什么呢?因为new可以自动根据类型计算待分配的内存的大小, 所以调用者必须在使用 new
运算符的时候 指定元素个数.    

    int arr[3] = new int[3];


然后new作为一个运算符, 是可以重载的(这块只是看过,并没有用过). 而C语言里没有重载的概念.

new的重载如下

    //这些版本可能抛出异常
    void * operator new(size_t);
    void * operator new[](size_t);
    void * operator delete (void * )noexcept;
    void * operator delete[](void *0）noexcept;
    //这些版本承诺不抛出异常
    void * operator new(size_t ,nothrow_t&) noexcept;
    void * operator new[](size_t, nothrow_t& );
    void * operator delete (void *,nothrow_t& )noexcept;
    void * operator delete[](void *0,nothrow_t& ）noexcept;


​    

另外还有个版本的,禁止重载.


​    

    void * operator new (size_t,void *) //不允许重定义这个版本的operator new


这个operator new不分配任何的内存，它只是简单地返回指针实参，然后用new表达式负责在place_address指定的地址进行对象的初始化工作。

关于,二次分配内存的问题, malloc有一个兄弟, realloc, 可以判断该内存后序是不是空闲, 从而原地扩大内存, new则屏蔽了内存分配的细节,
也就是无从谈起二次分配了。

最后补充的一点是, new关键字有一个钩子函数, new_handler,应用与new失败后, 应该如何去做的步骤. (这个了解的也不多)

# 22 对象实例的构造过程

实例的初始化，先从初始化列表开始，然后给初始化列表以外的成员变量赋予默认值。



# 23 不定参数的实现原理？



# 24 Cpp11更高版本的变化有哪些？

我被问到这个问题，我感觉是考察对新技术的敏感度。

1 cpp14引入了 make_unique, depcrecated, lambda函数支持自动推导，具体参见 https://www.infoq.cn/article/2014/09/cpp14-here-features

2 cpp17 引入了 pair和vector等容器的自动推导，结构化绑定 auto a,b = func(), 类似于go语言，init-if语句，string_view,避免字符串拷贝，内联式的命名空间等。参考 https://zhuanlan.zhihu.com/p/165640868

3 cpp20引入了内置的协程库。



# 参考

[1] 各种智能指针的应用场景，https://01io.tech/unique_ptr-vs-shared_ptr-vs-weak_ptr/#toc-4





