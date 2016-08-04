类的基本思想是数据抽象（封装）、继承和多态（动态绑定）

* 数据抽象：把客观事物封装成抽象的类，同时将类的接口和实现分离。（优点：可以隐藏实现细节，使得代码模块化）
* 继承：定义相似的类型，并对其相似关系建模。（优点：可以扩展已存在的代码模块）
* 多态：一定程度上忽略相似类型的区别，以统一的方式使用它们的对象。

# 构造／析构／赋值运算

当我们定义一个空类时，C++ 编译器会默认为它合成 `默认构造函数，copy构造函数，赋值操作符和一个析构函数`。因此，如果我们写下：

    class Empty(){};

这就好像写下这样的代码：

    class Empty {
    public:
        Empty() { ... }
        Empty(const Empty& rhs) { ... }
        ~Empty() { ... }
        Empty& operator=(const Empty& rhs) { ... }
    };

不过要注意以下几点：

1. 只有在被需要（被调用）的时候，它们才会被`编译器`创建；
2. 这些函数都是public的；
3. 这些函数都是inline的（即函数定义在类的定义中）
4. 如果显式地声明了其中一个函数，那么编译器将不再生成默认的函数。特别需要注意的是**自定义的拷贝构造函数不仅会覆盖默认的拷贝构造函数，也会覆盖默认的构造函数**。
5. 对于拷贝构造函数和赋值操作符来说，编译器创建的版本只是单纯地将来源对象的每一个 non-static 数据成员拷贝到目标对象中。
6. 赋值操作符函数的行为与拷贝构造函数的行为基本是相同的，但是编译器生成赋值操作符函数是有条件的，如果会产生无法完成的操作，编译器将拒绝产生这一函数。
7. 编译器生成的拷贝构造函数和赋值操作符都执行`浅拷贝操作`。当类里面有指针时，最好根据需要写执行深拷贝操作的拷贝构造函数和赋值操作符函数。

另外，还存在两种默认的函数：就是取地址运算符和取地址运算符的const版本，这两个函数在《Effective C++》中没有提及。

    Empty* operator&() { ... }
    const Empty* operator&() const { ... }

所以即使定义一个空类，下面的代码也是可以运行的：

    Empty a;
    const Empty *b = &a;
    printf("%p\n", &a);     //调用取地址运算符
    printf("%p\n", b);      //调用const取地址运算符

［[声明对象的坑](http://www.nowcoder.com/questionTerminal/95581302aa714466bc766bc51b5524fc)］

## String 类的实现

类的构造函数，赋值函数，析构函数。

    class String
    {
    public:
        String(const char *str=NULL);//普通构造函数
        String(const String &str);//拷贝构造函数
        String & operator =(const String &str);//赋值函数
        ~String();//析构函数
    private:
        char* m_data;//用于保存字符串w
    };

具体实现（[String.cpp](../Coding/String.cpp)）

## 赋值还是构造

在面向对象程序设计中，对象间互相拷贝是经常进行的操作，那么调用的到底是拷贝构造函数还是赋值操作符呢？有一个简单的判定规则：

* 左值对象已经存在的话调用的是赋值操作符，
* 左值对象在当前语句第一个出现，那么调用拷贝构造函数。

还要注意函数形参不是引用时，会调用对象的拷贝构造函数，生成一个临时的形参对象，到函数结尾，会自动销毁。函数返回非引用对象时，也会调用拷贝构造函数创建一个对象返回。

具体看代码 [Class_ConDesAssign.cpp](../Coding/Class_ConDesAssign.cpp)

［[拷贝构造还是赋值](http://www.nowcoder.com/questionTerminal/cf1a3145d1b946c1861c9d10b8629665)］

## 构造函数与析构函数

派生类构造函数调用顺序如下：

1. 基类构造函数。如果有多个基类，则构造函数的调用顺序是基类在类派生表中出现的顺序。
2. 若派生类中包含对象成员，还要进行对象成员初始化。如果有多个成员类对象则构造函数的调用顺序是对象在类中被声明的顺序。
3. 派生类构造函数。

析构函数正好和构造函数相反。 ([constructor_derived_class.cpp](../Coding/constructor_derived_class.cpp))

［[构造函数中调用虚函数](http://www.nowcoder.com/questionTerminal/adb540e6222b401eb294b093b9fc6f0e)］  
［[构造函数调用次数](http://www.nowcoder.com/questionTerminal/bf70aadeb78949c2a61b1b561a0ee784)］  
［[析构的顺序](http://www.nowcoder.com/questionTerminal/ad46fe08266341b694d2ab8a78aa071f)］  
［[析构函数调用delete](http://www.nowcoder.com/questionTerminal/2dc097fd196343f88d7efde6e9447f91)］  

关于异常抛出问题：

1. 不建议在构造函数中抛出异常。构造函数抛出异常时，析构函数将不会被执行，需要手动的去释放内存；
2. 析构函数不应该抛出异常。当析构函数中会有一些可能发生异常时，那么就必须要把这种可能发生的异常完全封装在析构函数内部，决不能让它抛出函数之外；
3. Effective C++建议，析构函数尽可能不要抛出异常。如果对象抛出异常了，现在异常处理模块为了维护系统对象数据的一致性，避免资源泄露，有必要调用析构函数释放资源，这时如果析构过程又出现异常，那么谁来保证新对象的资源释放呢？前面的异常还没处理完又来了新的异常，这样可能会陷入无限的递归嵌套中。所以，从析构函数抛出异常，C++运行时系统会处于无法决断的境遇，因此C++语言担保，当处于这一点时，会调用 terminate()来杀死进程。

## 禁止对象产生在堆（栈）中

一般情况下，编写一个类，是可以在栈或者堆分配空间。但有些时候，你想编写一个只能在栈或者只能在堆上面分配空间的类。例如说在嵌入式系统中工作，为了保证不发生内存泄漏，最好保证没有任何一个类型的对象可以从 heap 中分配出来。

在C++中，类的对象建立分为两种，一种是静态建立，如A a；另一种是动态建立，如 A* ptr=new A；这两种方式是有区别的。

1. 静态建立类对象：是由编译器为对象在栈空间中分配内存，是通过直接移动栈顶指针，挪出适当的空间，然后在这片内存空间上调用构造函数形成一个栈对象。使用这种方法，直接调用类的构造函数。
2. 动态建立类对象，是使用new运算符将对象建立在堆空间中。这个过程分为两步，第一步是执行operator new()函数，在堆空间中搜索合适的内存并进行分配；第二步是调用构造函数构造对象，初始化这片内存空间。这种方法，间接调用类的构造函数。

### 禁止对象建立在栈上

考虑当对象建立在栈上面时，由编译器分配内存空间，调用构造函数来构造栈对象。当对象使用完后，编译器会调用析构函数来释放栈对象所占的空间，编译器管理了对象的整个生命周期。`如果编译器无法调用类的析构函数，情况会是怎样的呢？`比如，类的析构函数是私有的，编译器无法调用析构函数来释放内存。所以，编译器在为类对象分配栈空间时，会先检查类的析构函数的访问性，`如果类的析构函数是私有的，则编译器不会在栈空间上为类对象分配内存`。

> 只能在堆上分配类对象，就是不能静态建立类对象，可以通过将类的析构函数设为 private 来达到目的。

如下面的类 A：

    class A
    {
    public:
        A(){}
        void destory(){delete this;}
    private:
        ~A(){}
    };  

如果使用A a;来建立对象，编译报错，提示析构函数无法访问。但是可以使用new 操作符来建立对象，构造函数是公有的，可以直接调用。此外，类中必须提供一个destory函数，来进行内存空间的释放。堆对象使用完成后，必须调用destory函数。

但是上面方法有以下缺点：

1. `无法解决继承问题`。如果A作为其它类的基类，则析构函数通常要设为virtual，然后在子类重写，以实现多态。因此析构函数不能设为private。
2. `类的使用方法不统一`，使用new建立对象，却使用destory函数释放对象，而不是使用delete。（使用delete会报错，因为delete对象的指针，会调用对象的析构函数，而析构函数类外不可访问）

还好C++提供了第三种访问控制，protected。将析构函数设为protected可以有效解决继承问题，使得类外无法访问protected成员，子类则可以访问。

为了统一类的使用方式（不要 new 和 destroy 搭配），可以将构造函数设为protected，然后提供一个public的static函数来完成构造，这样不使用new，而是使用一个函数来构造，使用一个函数来析构。代码如下：

    class A
    {
    protected:
        A(){}
        ~A(){}
    public:
        static A* create()
        {
            return new A();
        }
        void destory()
        {
            delete this;
        }
    };

这样，可以像下面这样在堆上创建、销毁对象：

    A *a = A::create();
    a->destory();

### 禁止对象产生在堆上

只有使用 new 运算符，对象才会建立在堆上，因此，只要禁用new运算符就可以实现类对象只能建立在栈上。虽然不能影响new运算符的能力（因为是C++语言内建的），但是可以利用一个事实：new运算符总是先调用 operator new，而后者我们是可以自行声明重写的。因此，`将operator new()设为私有即可禁止对象被new在堆上`。

    class A
    {
    private:
        void* operator new(size_t t){}     // 注意函数的第一个参数和返回值都是固定的
        void operator delete(void* ptr){}  // 重载了new就需要重载delete
    public:
        A(){}
        ~A(){}
    };

［[只能new创建对象](http://www.nowcoder.com/questionTerminal/9ca9a4991164463b90b6ba0fef227030)］

## 构造函数初始值列表

为变量分配地址和存储空间的称为`定义`，不分配地址的称为`声明`。一个变量可以在多个地方声明，但是只在一个地方定义。加入 extern 修饰的是变量的声明，说明此变量将在文件以外或在文件后面部分定义。

定义变量时习惯对其初始化，而非先声明、再赋值。

    string foo = "Hello";   // 定义并初始化
    string bar;             // 默认初始化为空 string 对象
    bar = "Hello";          // 为 bar 赋一个新值

就对象的数据成员来说，如果没有在构造函数的初始化列表中显式地初始化成员，则该成员在构造函数之前执行默认初始化，在构造函数中进行的是赋值操作。但是如果成员是 const 或者引用的时候，或者`成员属于某种类类型且该类没有定义默认构造函数`时，`必须`进行初始化。

    class ConstRef{
    public:
        // 正确, 使用构造函数初始化列表显式地初始化引用和 const 成员.
        ConstRef(int num):const_i(num), ref_j(num){}
        /*
        ConstRef(int num){
            const_i = num;  // 错误,不能给 const 赋值
            ref_j = num;    // 错误, ref_j 没有初始化
        }
        */
    private:
        const int const_a = 0;
        const int const_i;
        int &ref_j;
    };

构造函数初始值列表只说明用于初始化成员的值，而不限定初始化的具体执行顺序。成员的初始化顺序与它们`在类定义中的出现顺序一致`，第一个成员先被初始化，然后第二个，以此类推。如果一个成员是用另一个成员来初始化的，那么两个成员的初始化顺序就很关键了！（可能的话，**尽量避免使用某些成员初始化其他成员**）。

［[初始化顺序](http://www.nowcoder.com/questionTerminal/fb01e2436c6d453abbbf9801f794165b)］  
［[类定义static与const](http://www.nowcoder.com/questionTerminal/5ab084ff358e43f392833dbcd2963872)］  
［[必须通过构造函数初始化列表的变量](http://www.nowcoder.com/questionTerminal/da5c9884bc824b72a345c8fdfb53b79b)］  

# 数据成员与成员函数

## 数据成员

## 类的静态成员

在类成员的声明之前加上关键字 static 使得成员与类本身直接相关，而不是与类的各个对象保持关联。和其他成员一样，静态成员可以是 public 或 private 的，静态数据成员的类型可以是常量、引用、指针、类类型等。类的静态成员存在于任何对象之外，对象中不包含任何与静态数据成员有关的数据。

类的静态数据成员不属于类的任何一个对象，所以它们不是在创建类的对象时被定义的，也就是说不是由类的构造函数初始化的（`不能用构造函数初始化静态数据成员`）。通常情况下，类的静态成员不应该在类的内部初始化，必须在类的`外部定义和初始化`每个静态成员。一种例外情况是，为静态数据成员提供 const 整数类型的类内初始值（**即使是 const static 也不能用构造函数初始化列表来进行初始化**）。

类似的，静态成员函数也不与任何对象绑定在一起，不包含 this 指针。`静态成员函数不能声明为 const 的（本来就不会去改变对象的值，所以没有必要定义为const），而且不能在 static 函数体内使用 this 指针`。既可以在类的内部定义静态成员函数，也可以在外部定义静态成员函数。在类的外部定义静态成员函数时，不能重复关键字 static。

虽然类的静态成员不属于类的某个对象，但我们仍可以使用类的对象、引用或者指针来访问静态成员。

静态成员可以应用于某些普通成员不能应用的场景：

1. 静态数据成员可以是不完全类型，甚至可以是它所属的类类型。而非静态数据成员则受到限制，只能声明成它所属类的指针或引用；
2. 可以使用静态数据成员作为默认实参。普通数据成员不能作为默认实参，因为它的值本身属于对象的一部分。

## 成员函数

const函数与同名的非const函数是重载函数；const对象只能调用const函数 ，但是非const对象可以调用const函数。

［[const函数的操作](http://www.nowcoder.com/questionTerminal/09ec766d373a43769603963664e231e7)］  

`函数隐藏`是指派生类的函数屏蔽了与其同名的基类函数，规则如下：

1. 如果派生类的函数与基类的函数同名，但是参数不同。此时，不论有无 virtual 关键字，基类的函数将被隐藏（注意别与重载混淆）。
2. 如果派生类的函数与基类的函数同名，并且参数也相同，但是基类函数没有 virtual 关键字（否则是覆盖）。此时，基类的函数被隐藏（注意别与覆盖混淆）

# 继承

继承是类的重要特性。通过继承联系在一起的类构成一种层次关系，通常在层次关系的根部有一个基类，其他类则直接或者间接地从基类继承而来，这些继承得到的类称为派生类。基类负责定义在层次关系中所有类公同拥有的成员，而每个派生类定义各自特有的成员。

**派生类可以继承定义在基类中的成员，但是派生类的成员函数不一定有权访问从基类继承而来的成员**，访问权限受下面因素影响。

* 继承方式；
* 基类成员的访问权限(即public/private/protected)。

继承有三种方式，即公有(Public)继承、私有(Private)继承、保护(Protected)继承。（私有成员不能被继承）

* 公有继承就是将基类的公有成员变为自己的公有成员，基类的保护成员变为自己的保护成员。
* 保护继承是将基类的公有成员和保护成员变成自己的保护成员。
* 私有继承是将基类的公有成员和保护成员变成自己的私有成员。

三种继承方式的比较

![][1]

具体代码示例参见 [C++_Inheritance.cpp](../Coding/C++_Inheritance.cpp)

一个派生类对象包含有多个组成部分：一个含有派生类自己定义的（非静态）成员的子对象，以及一个与该派生类继承的基类对应的子对象，如果有多个基类，那么对应的子对象有多个（C++ 没有明确规定派生类的对象在内存中怎样分布）。**继承中的父类的私有变量是在子类中存在的，不能访问是编译器的行为，可以通过指针操作内存来访问的。**

具体代码示例参见 [C++_Inheritance_2.cpp](../Coding/C++_Inheritance_2.cpp)

因为在派生类对象中含有与其基类对应的组成部分，所以可以把派生类的对象当成基类对象来使用，而且也可以将基类的指针或引用绑定到派生类对象中基类部分上。这种转换通常称为 `派生类到基类的（derived-to-base）` 类型转换。**派生类向基类的转换是否可访问由使用该转换的代码决定，同时派生类的派生访问说明符也会有影响。**

如果基类定义了一个`静态成员`，则在整个继承体系中只存在该成员的唯一定义。不论从基类派生出来多少个派生类，对于每个静态成员来说都只存在唯一的实例。

［[派生类存储，隐式转换](http://www.nowcoder.com/questionTerminal/c85f9e15e6a4410a930581ae12b9a341)］  
［[子类继承父类所有对象](http://www.nowcoder.com/questionTerminal/ff91c410e28745e8ae01537d8a888283)］  
［[派生类重复定义基类数据成员](http://www.nowcoder.com/questionTerminal/ade233b99dfc4f03aba0335f9f2a3f35)］  

# 多态

C++ 中，基类必须将它的两种成员函数区分开来：一种是基类希望其派生类进行覆盖的函数，另一种是基类希望派生类直接继承而不要改变的函数。对于前者，基类通常将其定义为`虚函数（virtual）`。当我们使用指针或引用调用虚函数时，该引用将被动态绑定。根据引用或指针所绑定的对象不同，该调用可能执行基类的版本，也可执行某个派生类的版本。（成员函数如果没有被声明为虚函数，则其解析过程发生在编译时而非运行时）

基类通过在其成员函数的声明语句之前加上 virtual 关键字使得该函数执行动态绑定，**任何构造函数之外的非静态函数都可以是虚函数**。如果基类把一个函数声明为虚函数，则该函数在派生类中隐式地也是虚函数（**派生类可以不重写虚函数，必须重写纯虚函数**）。［C++ primer P528］

C++中的虚函数的作用主要是实现多态机制。关于多态，简而言之就是用父类型的指针指向其子类的实例，然后通过父类的指针调用子类的成员函数。这种技术可以让父类的指针有“多种形态”，这是一种泛型技术。

虚函数（Virtual Function）是通过一张虚函数表（Virtual Table）来实现的。每个包含有虚函数的类有一个虚表，在有虚函数的类的实例中保存了虚表的指针，所以，当我们用父类的指针来操作一个子类的时候，这张虚函数表像一个地图一样，指明了实际所应该调用的函数。

C++的编译器保证虚函数表的指针存在于对象实例中最前面的位置（这是为了保证取到虚函数表的有最高的性能）。

此外：内联函数也不能是虚函数，因为其在编译时展开。虚函数是动态绑定，运行期决定的，所以内联函数不能是虚函数。

［[虚函数地址分配](http://www.nowcoder.com/questionTerminal/d50dbed9a0f44e8092f86927cb7c259f)］  
［[虚函数表被置为0](http://www.nowcoder.com/questionTerminal/97c2bf56369845528a109bec8cfb3556)］  
［[缺省参数是静态绑定的](http://www.nowcoder.com/questionTerminal/e4d5fe27a85d43548171f32b3bc8501a)］  

# 抽象类

为了方便使用多态特性，常常需要在基类中定义虚拟函数。但是在很多情况下，基类本身生成对象是不合情理的。例如，动物作为一个基类可以派生出老虎、孔雀等子类，但动物本身生成对象明显不合常理。 
为了解决上述问题，引入了`纯虚函数`的概念，将函数定义为纯虚函数，则编译器要求在派生类中必须予以重载以实现多态性。同时含有纯虚拟函数的类称为抽象类，它不能生成对象，这样就很好地解决了上述两个问题。

纯虚函数是在基类中声明的虚函数，它在基类中没有定义，但要求任何派生类都要定义自己的实现方法。在基类中实现纯虚函数的方法是在函数原型后加“=0”，比如：

    virtual ReturnType Function()= 0; 
`抽象类`是一种特殊的类，它是为了抽象和设计的目的而建立的，它处于继承层次结构的较上层。抽象类是不能定义对象的，在实际中为了强调一个类是抽象类，可将该类的构造函数说明为保护的访问控制权限，这样就无法静态或者 new 创建该类的栈或者堆对象。

抽象类只能作为基类来使用，其纯虚函数的实现由派生类给出。**如果派生类没有重新定义纯虚函数，而派生类只是继承基类的纯虚函数，则这个派生类仍然还是一个抽象类。**如果派生类中给出了基类纯虚函数的实现，则该派生类就不再是抽象类了，它是一个可以建立对象的具体类了。

抽象类的规定如下：

1. 抽象类只能用作其他类的基类，不能建立抽象类对象（构造函数设为 protect）。
2. 抽象类不能用作参数类型、函数返回类型或显式转换的类型。
3. 可以定义指向抽象类的指针和引用，此指针可以指向它的派生类，进而实现多态性。

［[抽象类对象指针](https://www.nowcoder.com/questionTerminal/306811e56957419181789af6787e3d54)］  

# 友元

友元关系是单向的，不是对称，不能传递。关于传递性，有人比喻：父亲的朋友不一定是儿子的朋友。那关于对称性，是不是：他把她当朋友，她却不把他当朋友？

［[友元特征](http://www.nowcoder.com/questionTerminal/f1491d455d28443e9c1a0c01ddb9d6ab)］  
［[友元访问类所有成员?](http://www.nowcoder.com/questionTerminal/97701500d7064ecfa8c97ee4292c0433)］  

// TODO

# 更多阅读
Effective C++ 05  
More Effective C++ 条款 27  
[C++编译器自动生成的函数](http://www.cnblogs.com/xiaoxinxd/archive/2013/01/09/effective_cpp_05.html)  
[如何让类对象只在栈（堆）上分配空间？](http://blog.csdn.net/hxz_qlh/article/details/13135433)    
[构造函数：C++](https://msdn.microsoft.com/zh-cn/library/s16xw1a8.aspx)  
[C++ 虚函数表解析](http://blog.csdn.net/haoel/article/details/1948051)  
[深入理解C++的动态绑定和静态绑定](http://blog.csdn.net/chgaowei/article/details/6427731)  
[C++ 抽象类](http://www.cnblogs.com/balingybj/p/4771916.html)  

[1]: http://7xrlu9.com1.z0.glb.clouddn.com/C++_Class_1.png

