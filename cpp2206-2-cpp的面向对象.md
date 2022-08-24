# C++面向对象的重要知识点梳理





最近在学习C++，于是打算整理一些专题知识，梳理自己的学习成果。一来是回忆和整合，加深理解，再一个是希望大佬指点。

今天说的是 C++面向对象的重要话题。

## 目录

- [背景](#背景)
- [正文](#正文)
- [系列历史文章](#系列历史文章)
- [征求互动](#征求互动)
- [参考](#参考)



## 背景

面向对象是C++ 突破C的重要特性，今天就聊一聊我理解的C++其他面向对象的特质。



## 正文

### 成员变量的初始化

在C++ Primer plus 第十四章里有提到详细的初始化过程。先初始化成员变量，如果成员变量是类，就调用其默认构造函数。一般变量就赋默认值。

对此我有些疑惑。比如如果成员变量是一个结构体，一个char，一个int的指针，都会赋什么值呢？

实例的初始化，先从初始化列表开始，然后给初始化列表以外的成员变量赋予默认值。

```c++
class Member{
public:
    int m=10;
};

class Test{
public:
    int b;
    int *c; //初始化为NULL
    Member *m;  //初始化为NULL
    Member m2;
    void f1(){
        cout << b << endl;
        cout << (c == NULL) << endl;
        cout << (m == NULL) << endl;
        cout << m2.m << endl;
    }
};

void main_f11(){
    Test* t = NULL;
    cout << t->b << endl;

    Test* t2 = new Test();
    cout << t2->b << endl;
    // error, code 11,没有打印
}

void main_f12(){
    Test* t2 = new Test();
    t2->f1();
    // 0
    //1
    //1
    //10
}
```



### 种类繁多的继承方式

为什么要设计这么多的继承方式？各自的目的是什么，各自的应用场景是什么？如果正确的使用防止采坑，这是我们必须要知道的。

如果protect 继承，那么派生类得到的对应属性是什么类型的呢？

最常见的是public继承，也就是说基类的public属性会被派生类继承为public属性，而protect继承则会把大于等于protect的属性继承为protect属性，private同理。

有几种关系，分别为has-a, is-a。什么意思呢？has-a的意思就是，有一个类的实例，被当做其他类的成员，可以调用这个类的方法和属性，这在设计模式里叫做拥有，而is-a，则是继承了一个类，成为了这个类的子类，也可以拥有这个类的方法和属性。区别在于，能不能使用多态，如果需要多态，就要is-a模型，而如果has-a，则需要持有这个对象即可。

那么对于private继承来说,它有什么用呢？

private继承。《Effecitive C++》里提到，private继承单纯作为一种实现方式，举个例子，B private继承A，也就是仅仅把A里面的内容拷贝到B里面去，是一种工程实现，而非一种设计思想，单纯的为了代码复用而已。

总之，private继承和protect继承都是一种实现继承，单纯为了代码复用，没有多态的功能。只有public继承才有多态的效果，而且默认是private继承。

再来说说多继承，其缺点是什么呢？

如果有继承两个类，而这个两个类B和C继承自同一个基类A，则这两个类B和C又被D继承，那么A的属性会在D里面保留两份，这显然是不对的，如何避免，使用虚拟继承即可，virtual关键字，不仅仅修饰了方法，还作为状语修饰了继承动作，虚拟继承（virtual）的用处是解决菱形继承，难道就这一种用途吗?我们不得而知。

那么virtual继承又会引来哪些问题呢？又是如何实现的呢？这个问题该怎么解决呢? 日后再说。



### 继承的模型

子类的实例里会有父类的隐藏实例。是的，这里有一篇详细的文章[参考5]讨论了派生对象的内存模型。这里说的和C++ Primer plus的13.4.2里面说的内存模型不一致。

我们说，一个基类如果有虚函数，那么就会有一个虚函数表，如果一个对应的派生类继承基类后，并没有自己添加新的虚函数，那么可能会共享虚函数表（vtbl)，只有派生类新增了虚方法，才有必要完全拷贝一个虚函数表，也就是 copy on write 技术。这只是一种猜测，而看很多博客，结论应该是不同的编译器的实现是不同的。

继承的模型的话，派生类会有N代基类的全部属性，具体的占据内存的大小会根据内存对齐有所调整。至于成员变量的访问方式，根据继承方式的不同，会有不同的权限。这一块的控制应该还是由编译器去完成的，对于权限的控制是一个复杂的东西。我们做的，就是按照编译器的要求，正确的填写语法。



### 再谈构造函数

构造函数分为好几种，复制构造函数，有参构造函数。

构造函数有一个修饰用的关键字叫做 explicit，这个问题需要我们认真深挖一下。去网上随便查询一下，都知道explict关键字修饰构造函数的用处是防止隐形转换，从而调用构造函数。

我们知道，C++经常会偷偷的调用构造函数而让我们不知道，其中涉及隐式转换。**在C++中常常调用隐式转换分为，（1）形参调用隐式转换和（2）返回值调用隐式转换（3）赋值符号调用隐式转换。**



举个**返回值调用隐式转换**的例子:

```c++
class Base{
    int a;
public:
    Base():a(0){
        // cout << "Base" << endl;
    }

    Base(int pa):a(pa){}
    virtual ~Base(){
        //cout << "Base deconstructor" << endl;
    };

    Base(const Base & item){ //拷贝函数的形式必须是引用传参
        cout << "copy Base" << endl;
    }

    virtual void f1(){
        cout << "Base f1" << endl;
    }
};

class Derived: public Base{
    int *b;
public:
    Derived(): b(new int(0)){
        // cout << "Derived" << endl;
    }

    Derived(int pb): b(new int(pb)){}

    ~Derived(){
        // cout << "Derived deconstructor" << endl;
    }

    void f1(){
        cout << "Derived f1" << endl;
    }
};

void main_f4(){
    Base item = Derived(); //这里明显说明了调用了拷贝构造函数，发生了类型强转
    // 打印结果：
    // copy Base
		// Base f1
}

void main_f5(){
    Base * item = new Derived; //只有引用和指针不用类型强转
    item->f1();
    delete item;
    // 打印结果：Derived f1
}
```



再回顾一下，发生隐式转换的时候，编译器无疑会寻找最合适的构造函数来实现隐式转换。而这种隐式转换在带来方便的同时，也会因为我们疏忽大意带来意想不到的后果。explict就是为了防止意想不到而出现的关键字，它会禁止自动隐式转换。为此，我们需要在拷贝构造函数、普通构造函数前面加上explict。



### 封装与封装的后门友元函数

如果一个基类的属性是private，那么被继承过来了吗，有可能访问到吗？不能。

友元函数的设计思路是什么？友元的设计可以极大的方便函数式编程，把友元函数本身当做一个对象传进去。纯粹面向对象的缺点，以Java8及以前的版本为例，没有把抽象的过程当做一个对象的，而C++，则依旧保留着这个概念。



### 重载和重写的细节深挖

为什么编译器会在重载的时候自动把基类对应的同名不同参函数隐藏起来?

上次说了，我们说《Effective C++》6.33解释了几个原因。（1）避免默认继承太多前几代的函数方法。



重载是一个点比较多的东西，它要求，函数同名，函数参数列表不同（跟参数名无关，跟参数类型有关）。但是重载的一个点是，参数列表中如果有修饰符呢？有无修饰符能区分他们吗？不能，举个例子：

```cpp
void const_test_f1(Base b){
}
void const_test_f1(const Base b){ //编译器会报错，重复定义。
}
```



重写同样是一个点比较多的东西，它要求，函数同名，函数参数列表相同（跟参数名无关，跟参数类型有关），返回类型相同（可以是子类）。



### 如何自己实现一个无错的拷贝构造函数/赋值函数？

拷贝构造函数的实现需要注意的事情有（1）如果有动态内存变量（指代指针和引用），深浅拷贝的问题。

赋值函数的实现需要注意的事情有（1）如果有动态内存变量（指代指针和引用），深浅拷贝的问题，（2）对于赋值函数，检测被赋值的变量是不是和赋值变量指向同一个对象 ，（3）如果（2）不满足，那么需要释放被赋值的原有变量指向的内存。

不同的类实现起来难度不一样，需要定制化，我们模拟普通的 string 类写一个拷贝构造函数来举例。

给定一个类的声明，请我们实现一个类的方法。

```cpp
class String{
public:
    String(void);
    String(const char *chars=NULL); //普通构造函数
    String(const String& str); //拷贝构造函数
    String& operator=(const String& str);
    ~String(void); //析构
private:
    char* m_chars;  //这里需要说明的是，m_chars不能被声明为数组类型，char[] m_chars，因为数组名无法被赋值。
};
```



接下来，只要注意实现的几个点就可以了。

```cpp
using namespace std;

class String{
public:
    String(void);
    String(const char *chars=NULL); //普通构造函数
    String(const String& str); //拷贝构造函数
    String& operator=(const String& str);
    ~String(void); //析构
private:
    char* m_chars;  //这里需要说明的是，m_chars不能被声明为数组类型，char[] m_chars，因为数组名无法被赋值。
};

String::String(){
    cout << "default constructor" << endl;
    m_chars = new char[1];
    *m_chars = '\0'; //非数组不能使用[]运算符，只能使用指针赋值
}

String::String(const char* chars){
    cout << "char pointer constructor" << endl;
    if(chars == NULL){
        m_chars = new char[1];
        *m_chars = '\0'; //非数组不能使用[]运算符，只能使用指针赋值
    }else{
        m_chars = new char[strlen(chars)+1];
        strcpy(m_chars, chars);
    }
}

String::String(const String &str){ //在拷贝构造函数里，可以访问私有属性,这点记住
    cout << "copy constructor" << endl;
    m_chars = new char[strlen(str.m_chars)+1];
    strcpy(m_chars, str.m_chars);
}

String::~String() {
    cout << "deconstructor" << endl;
    if(m_chars != NULL){ // 防止重复释放同一个内存导致系统崩溃,因为存在赋值和拷贝
        delete m_chars; //先释放原有原有内存，归还操作系统
        m_chars = NULL; //再把指针指向NULL，防止野指针
    }
}

String & String::operator=(const String &str) { // 赋值函数也可以访问私有属性，所以务必给赋值函数参数加上const
    cout << "operator= " << endl;
    if( &str == this) return *this; //是指向同一个内存，直接释放
    else{
        delete m_chars; //先释放原有原有内存，归还操作系统
    }
    m_chars = new char[strlen(str.m_chars)+1];
    strcpy(m_chars, str.m_chars);
    return *this; // 即便返回引用类型，我们只需要返回对象的本体即可，编译器会做处理，我们不用返回引用
}

void test_str_len(){
    char c[3];
    char cc[3] = "ab";
    strcpy(c, cc); // C 的库函数，需要学习一下
    cout << c << endl;
}

void test_String_f1(){
    String a = "abc";
    {
        String b = a;
        // delete b; delete无法操作普通对象，只能操作指针
        String *c = &a;
        // delete c; 注意！delete只能释放堆内存，不能释放栈内存，释放栈内存会报错：pointer being freed was not allocated

        String null_str(NULL);// 防止歧义
    }
    String d = "aefg";
    a = d;
    // 打印结果
    // char pointer constructor
    //copy constructor
    //char pointer constructor
    //deconstructor
    //char pointer constructor
    //operator=
    //deconstructor
    //deconstructor
    //deconstructor
}

void default_null(char * chars=NULL){
    cout << "call" << endl;
}

void test_default_null(){
    default_null();
}

int main() {
    //main_f4();
     test_String_f1();
    // test_default_null();
    return 0;
}
```



> 最后有一个问题，应该把拷贝构造函数返回的值放在堆内存里，还是栈内存里呢？应该是放在堆里面，因为我们操作栈内存。

这个难点在于能否熟练使用**C语言字串函数**。



### 面向对象编程更近一步，完善自定义String类的功能

来一个挑战，但是我这次不会做。先挖个坑，比如实现一个完整的String类的功能，要挑战一下吗？我先把类定义放在这里。

```cpp
#include <iostream>
using namespace std;

//DIY_string
class String{
public:
    String(const char* chars=NULL);
    String(const String&);
    String(const String&, int n);
    String(char c, int n);
    ~String();
    String& operator=(const char *chars);
    String& operator=(String &str);
    String& operator+(String &str);
    String& operator+=(String &str);
    char operator[](int index);
    // char operator[](int start, int end); // 胆子再大一点，实现类似python的 a[1:3]这样的
    String& substr(int start, int offset);
    String& index(String &str, int start=0);  // 第一次出现的位置
    bool operator==(String &str);
    friend ostream& operator<<(ostream &out, String &str);
    friend istream& operator>>(istream &in, String &str);
private:
    char * m_chars;
public:
    int size();
    int size_; // 成员函数不能和成员变量重名
};
```



### 一个无关紧要的测试

展示一段代码，更好的理解C++中的面向对象的模型

```cpp
class Test{
public:
    int b;
    int *c;
    Member *m;
    Member m2;
    void f1(){
        cout << b << endl;
        cout << (c == NULL) << endl;
        cout << (m == NULL) << endl;
        cout << m2.m << endl;
    }
    void f2(){
        cout << "Test::f2" << endl;
    }
};

void main_f11(){
    Test* t = NULL;
    cout << t->b << endl;

    Test* t2 = new Test();
    cout << t2->b << endl;
    // 结果：
    // error, code 11
}

void test_f12(){
    // 注意一个Java程序员不能理解地方，如果一个对象是NULL，它是仍然可以调用这个方法的，只要这个方法可以避免使用this指针
    // 因为方法不用创建对象就已经存在
    Test* t = NULL;
    t->f2();

    // 结果：
    // Test::f2
}
```



### this 指针

如果有人问我，关于this指针你知道哪些？我会告诉它，this指针，是一个const类型的指针，this指针是非静态成员函数里可以直接调用的，不可以指向其他地址。似乎有些少。所以，还有什么？

this指针在构造函数体之前的初始化列表中不能使用，因为对象还没有创建，this指针还不存在。而在构造函数体内则是存在的。每一个成员变量前面都有一个隐藏的this指针。



## 经典问题

1 构造一个派生类的过程？为什么先构造虚表指针，然后再构造成员变量呢？

2 如果在类C的构造函数中调用C的虚函数，C被D继承。D的构造函数执行的过程中，构建C时，会调用D的对应的虚函数吗？

3 如果派生类没有重写基类的虚函数，那么派生类的虚函数表是会和基类共用一张吗？同理，对于成员变量呢？

4 基类B 有一个普通非虚方法f1，里面用到了一个公共属性 m1, 子类D public 继承了 基类B，D改写了m1的值, D的实例d调用方法 f1，请问d使用的是基类的成员变量还是派生类的成员变量？

```cpp
class B{
public:
    int a = 1;
    int a2 = 1;
    void f1(){
        cout << a << endl;
    }
};

class D: public B{
public:
    int a = 2;
    void f2(){
        cout << a << endl;
    }
};

void test_extend(){
    B *b = new D;
    cout << sizeof(*b) << endl;
    b->f1();
    //8
    //1
}

void test_extend2(){
    D *b = new D;
    cout << sizeof(*b) << endl;
    b->f2();
    //12
    //2
}

void test_extend3(){
    B *b = NULL;
    cout << "size of NULL point b:" << sizeof(*b) << endl;
    // size of NULL point b:8
}

```



当我继续调整，

```cpp
class B{
public:
    int a = 1;
    int a2 = 1;
    virtual void f1(){
        cout << "base "<< a << endl;
    }
};

class D: public B{
public:
    int a = 2;
    void f2(){
        cout << a << endl;
    }
};

void test_extend(){
    B *b = new D;
    cout << sizeof(*b) << endl;
    b->f1();
    // 16
    // base 1,这个时候
}

```

当我在D 中重写了f1方法，奇怪的事发生了。

```cpp
class B{
public:
    int a = 1;
    int a2 = 1;
    virtual void f1(){
        cout << "base "<< a << endl; // 隐藏的this指针指向自己的 a
    }
};

class D: public B{
public:
    int a = 2;
    void f2(){
        cout << a << endl;
    }
    virtual void f1(){
        cout << "derived "<< a << endl;  // 隐藏的this指针指向自己的 a
    }
};

void test_extend(){
    B *b = new D;
    cout << sizeof(*b) << endl;
    b->f1();

    // 16
    //derived 2
}
```

每个属性前都有一个隐藏的 this指针，指向当前的本类对象。所以，结论是，你继承了一个virtual方法，但是不重写的话，还是调用的原来基类的属性。原理是每个属性前面都有一个 默认的this指针，指向当前类的实例



5 多重继承 class D:public B1, public B2;请问继承的顺序是怎么样的？先构造哪一个类？他们的成员变量再内存中的排布是怎么样的？

继承的顺序是B1，B2。先构造B1，在构造B2，最后构造D，成员变量的位置是，B1的成员变量在最前面，B2在后面，D的成员变量在最后。







## 系列历史文章

微信公众号历史推文里有。



## 征求互动

如果你觉得读完有帮助，请点个赞。
如果你觉得读完有打动你，欢迎和我留言互动。
如果你觉得我哪里写的不好，请留下你的看法。
如果觉得有改进的地方，也请不吝赐教。
如果你喜欢我的公众号，请点个关注。

## 参考

[1]c++继承详解之一——继承的三种方式、派生类的对象模型
https://blog.csdn.net/lixungogogo/article/details/51118524

[2]Effecitve C++

[3]C++ Primer Plus, 13.4.2章，虚函数和动态绑定

[4]C和C++ 程序员面试秘籍，董山海

[5]C++继承模型

https://www.cnblogs.com/claireyuancy/p/6905648.html

[6]《深度探索C++对象模型》 P99-P123.

[7] 派生类的虚函数表问题

https://blog.csdn.net/cyd_shuihan/article/details/52587982

(2)虚函数表分析 https://leehao.blog.csdn.net/article/details/50688337

(3)图解C++虚函数 https://linyt.blog.csdn.net/article/details/51811314



