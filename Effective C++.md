# Effective C++
## 第一章 让自己习惯C++
### 01 视C++为一个语言联邦
1. C++是个多重范型编程语言：
    1. 过程形式(procedural):侧重于通过一系列的过程（即函数或例行程序）来解决问题，一种线性的编程方法，其中数据通常作为参数传递给函数，而函数则对这些数据进行处理。
    2. 面向对象(object-oriented):侧重于使用对象来模拟现实世界的实体。
    3. 函数形式(functional)：函数式编程是一种将计算视为数学函数评估的编程风格。在这种风格中，函数是一等公民，意味着它们可以像任何其他数据类型一样被传递和操作。
    4. 范型形式(generic):泛型编程允许程序员编写代码来操作多种数据类型，而无需为每种类型重复编写相同的代码。
    5. 元编程形式(metaprograming):C++元编程是一种在编译时执行计算和操纵程序实体的技术，它允许程序员在编译阶段就确定某些属性，从而避免了运行时的开销。

2. C++由几个重要的次语言构成:
    C语言：区块，语句，预处理器，数组，指针等等。
    Object-Oriented C++:class，封装，继承，多态......（动态绑定等等）
    Template C++:涉及泛型编程，内置数种可供套用的函数或者类。
    STL:标准模板库STL，主要涉及容器，算法和迭代器

**总结：**
+ C++高效编程守则视状况而变化，取决于使用哪个子语言

### 02 用const, enum, inline 替换 #define
即以编译器代替预处理器：以“常量”替换“宏”
1. #define 修饰的记号，在预处理的时候，已经全部被替换成了某个数值，如果出错，错误信息可能会提到这个数值，而不会提到这个记号。在纠错方面很花时间，因为其他程序员不知道这个数值代表什么。对于单纯常量，最好以 const 对象或 enums 替换 #defines
```cpp
//enum hack 补偿做法：
enum 枚举量{para1 = value1, para2 = value2,......}
//将一个枚举类型的数值当作 int 类型使用
//和 #define 很像，都不能取地址，但它没有 #define 的缺点
```
2. #define 不能定义类的常量，因为被 #define 定义的常量可以被全局访问，它不能提供任何封装性。
3. #define 修饰的宏书写繁琐且容易出错，inline 函数可以避免这种情况：

**总结：**
+ 对于单纯的常量，最好用const和enums替换#define， 对于形似函数的宏，最好改用inline函数替换#define

### 03 尽可能使用 const
1. const *  和 * const
左底层，右顶层。左值，右指向。
星号左边表示被指物事常量，星号右边表示指针自身是常量。

2. const成员函数，不能修改成员变量，只能调用const函数

**总结：**
+ 将某些东西声明为 const可帮助编译器侦测出错误用法，const可被施加于任何作用域内的对象、函数参数、函数返回类型、成员函数本体。
+ 编译器强制实施 bitwise constness，但你编写程序时应该使用“概念”上的常量性。
+ 当 const 和 non_const 成员函数有着实质等价的实现时，令 non_const 的版本调用 const 版本可避免代码重复。

### 04 确定对象被使用前以先被初始化
1. 永远在使用对象之前将它初始化。(记得区分 初始化 和 赋值)
   1. 初始化的效率高，赋值的效率低。
   2. 对于构造函数而言，使用 “初始化列表”(成员初始列)。
   3. 使用初始化列表时，尽量按照成员变量声明次序。
2. 使用Singleton模式(单例模式)，将每个非局部静态对象搬进自己的专属函数内(该对象在此函数内被声明为static)，函数返回一个reference指向它所含的对象。
   1. 确保获得的引用指向一个历经初始化的对象。
   2. 这样做的原理在于C++对于函数内的local static对象会在“该函数被调用期间，且首次遇到的时候”被初始化。当然我们需要避免“A受制于B，B也受制于A”
```cpp
原代码：
"A.h"
class FileSystem{
public:
    std::size_t numDisks() const;
};
extern FileSystem tfs;

"B.h"
class Directory{
public:
    Directory(params){
        std::size_t disks = tfs.numDisks(); //使用tfs
    }
}
Director tempDir(params);

修改后：
"A.h"
class FileSystem{...}    //同前
FileSystem& tfs(){       //这个函数用来替换tfs对象，他在FileSystem class 中可能是一个static，            
  static FileSystem fs;//定义并初始化一个local static对象，返回一个reference
  return fs;
}

"B.h"
class Directory{...}     // 同前
Directory::Directory(params){
  std::size_t disks = tfs().numDisks();
}

Directotry& tempDir(){   //这个函数用来替换tempDir对象，他在Directory class中可能是一个static，
  static Directory td; //定义并初始化local static对象，返回一个reference指向上述对象
  return td;
}
```
**总结：**
+ 为内置型对象进行手工初始化，因为C++不保证初始化它们。
+ 构造函数最好使用初始化列表，而不要在本体内使用赋值操作，成员初始列中的成员变量其排列次序应该和它们在class中的声明次序相同。
+ 为免除“跨编译单元之初始化次序”问题，请以 local static 对象替换 non_local static 对象。

## 第二章 构造/析构/赋值运算
### 05 了解C++自动生成和调用的函数。

**总结：**
+ 编译器可以暗自为 class 创建 default 构造函数，copy 构造函数，copy assignment 操作符，以及析构函数。

### 06 若不想使用编译器自动生成的函数，就该明确拒绝。

这一条主要是针对类设计者而言的，有一些类可能从需求上不允许两个相同的类，例如某一个类表示某一个独一无二的交易记录，那么编译器自动生成的拷贝和复制函数就是无用的，而且是不想要的。

**总结：**
1. 可将编译器自动生成的成员函数声明为private,并且不与实现，使用像Uncopytable 这样的base class 也是一种做法。
2. C++11将函数声明为“Delete”

### 07 为多态基类声明Virtual 析构函数

其主要原因是如果基类没有virtual析构函数，那么派生类在析构的时候，如果是delete 了一个base基类的指针，那么派生的对象就会没有被销毁，引起内存泄漏。
```cpp
class TimeKeeper
{
public:
    TimeKeeper();
    ~TimeKeeper();          // 错误做法
    virtual ~TimeKeeper();  // 正确做法
    virtual getTimeKeeper();
}

class AtomicClock : public TimeKeeper{...}

int main()
{
    TimeKeeper *ptk = new AtomicClock;
    delete ptk; // 基类析构函数不是虚函数，释放指向派生类的基类指针，会造成内存泄露
}
```

**总结：**
+ 如果一个函数是多态性质的基类，应该有virtual 析构函数
+ 如果一个class带有任何virtual函数，他就应该有一个virtual的析构函数
+ 如果一个class不是多态基类，也没有virtual函数，就不应该有virtual析构函数

补充：C++11：类名后加“final”表示不能被继承。

### 08 别让异常逃离析构函数
C++不喜欢析构函数吐出异常，多个析构函数同时出现异常，会导致未定义行为。

这里主要是因为如果循环析构10个Widgets，如果每一个Widgets都在析构的时候抛出异常，就会出现多个异常同时存在的情况，这里如果把每个异常控制在析构的话就可以解决这个问题：解决方法为：
```cpp
原代码：
class DBConn{
public:
    ~DBConn(){
        db.close();
    }
private:
    DBConnection db;
}

修改后的代码：    
class DBConn{
public:
    void close(){
        db.close();
        closed = true;
    }

    ~DBConn(){
        if(!closed){
            try{
                db.close();
            }
            catch(...){
                std::abort();
            }
        }
    }
private:
    bool closed;
    DBConnection db;
}
```
这种做法就可以一方面将close的的方法交给用户，另一方面在用户忽略的时候还能够做“强迫结束程序”或者“吞下异常”的操作。相比较而言，交给用户是最好的选择，因为用户有机会根据实际情况操作异常。

**总结：**
+ 析构函数绝对不要吐出异常。如果一个被析构函数调用的函数可能抛出异常，析构函数应该捕捉任何异常，然后吞下它们（不传播）或结束程序。
+ 如果客户需要对某个操作函数运行期间抛出的异常做出反应，那么 class 应该提供一个普通函数（而非在析构函数中）执行该操作。

### 09 绝不在构造和析构过程中调用 virtual 函数

主要是因为有继承的时候会调用错误版本的函数:
```cpp
原代码：
class Transaction{
public:
    Transaction(){
        logTransaction();
    }
    Virtual void logTransaction const() = 0;
};

class BuyTransaction:public Transaction{
    public:
        virtual void logTransaction() const;
};
int main()
{
    BuyTransaction b;
}

或者有一个更难发现的版本：

class Transaction{
public:
    Transaction(){init();}
    virtual void logTransaction() const = 0;
private:
    void init(){
        logTransaction();   // 构造函数期间，调用虚函数
    }
};
这个时候代码会调用 Transaction 版本的logTransaction，因为在构造函数里面是先调用了父类的构造函数，所以会先调用父类的logTransaction版本，解决方案是不在构造函数里面调用，或者将需要调用的virtual弄成non-virtual的

修改以后：
class Transaction{
public:
    explicit Transaction(const std::string& logInfo);
    void logTransaction(const std::string& logInfo) const; //non-virtual 函数
}
Transaction::Transaction(const std::string& logInfo){
    logTransaction(logInfo); //non-virtual函数
}
class BuyTransaction: public Transaction{
public:
    BuyTransaction(parameters):Transaction(createLogString(parameters)){...} //将log信息传递给base class 构造函数
private:
    static std::string createLogString(parameters); //注意这个函数是用来给上面那个函数初始化数据的，这个辅助函数的方法
}
```

1. 在 base class 构造期间， virtual 函数不是 virtual 函数。
   1. 如果父类中存在一个虚函数，并且该虚函数被构造函数所调用，子类也改写了该虚函数。当子类构造时，会先构造父类部分，此时调用父类的构造函数时，调用的虚函数是父类的版本而非子类的版本。
2. 解释原因：基类构造函数执行早于派生类对象，当基类构造函数执行时，派生类成员变量尚未初始化，如果此期间调用的虚函数下降至派生类阶层，会导致不明确行为。"要求使用对象内部尚未初始化的部分是危险的"
3. 根本原因：派生类对象的基类构造期间，对象类型是基类而非派生类。
同样析构函数调用基类虚构函数阶段，也视为此刻对象类型是基类。
4. 如果想在基类中调用如日志信息时，在派生类中将必要的构造信息向上传递给基类构造函数。

**总结：**
+ 在构造和析构期间不要调用 virtual 函数，因为这类调用从不下降至 derived class （比起当前执行构造函数和析构函数的那层）。

### 10 operator= 返回一个 reference to this
为了实现“连锁赋值”，主要是为了支持连读和连写，赋值操作符必须返回一个 reference 指向操作符的左侧实参。
```cpp
class Widget{
public:
    ...
    Widget& operator=(int rhs) {
        ...
        return *this;
    }
}
```
注意，这只是个协议，并无强制性。如果不遵循它，代码一样可通过编译。然而这份协议被所有内置类型和标准程序库提供的类型如 ring, vector, complex, rl::shared_ptr 或即将提供的类型(见条款 54)共同遵守。

**总结：**
+ 令赋值 (assignment) 操作符返回一个 reference to hiso

### 11 在 operator= 中处理“自我赋值”

主要是要处理 a[i] = a[j] 或者 *px = *py这样的自我赋值。有可能会出现一场安全性问题，或者在使用之前就销毁了原来的对象，例如
```cpp
原代码：
class Bitmap{...}
class Widget{
private:
    Bitmap *pb;
};
Widget& Widget::operator=(const Widget& rhs){
    delete pb; // 当this和rhs是同一个对象的时候，就相当于直接把rhs的bitmap也销毁掉了
    pb = new Bitmap(*rhs.pb);
    return *this;
}

修改后的代码
class Widget{
    void swap(Widget& rhs);    //交换this和rhs的数据
};
Widget& Widget::operator=(const Widget& rhs){
    Widget temp(rhs)           //为rhs数据制作一个副本
    swap(temp);                //将this数据和上述副本数据交换
    return *this;
}//出了作用域，原来的副本销毁

或者有一个效率不太高的版本：
Widget& Widget::operator=(const Widget& rhs){
    Bitmap *pOrig = pb;       //记住原先的pb
    pb = new Bitmap(*rhs.pb); //令pb指向 *pb的一个副本
    delete pOrig;            //删除原先的pb
    return *this;
}
```

**总结：**
+ 确保当对象自我赋值的时候operator=有比较良好的行为，包括两个对象的地址，语句顺序，以及copy-and-swap
+ 确定任何函数如果操作一个以上的对象，而其中多个对象是同一个对象时，其行为仍然正确。

### 12 复制对象时勿忘其每一个成分

**总结：**
+ 当编写一个copy或者拷贝构造函数，应该确保复制成员里面的所有变量，以及所有基类的成员
+ 不要尝试用一个拷贝构造函数调用另一个拷贝构造函数，如果想要精简代码的话，应该把所有的功能机能放到第三个函数里面，并且由两个拷贝构造函数共同调用
+ 当新增加一个变量或者继承一个类的时候，很容易出现忘记拷贝构造的情况，所以每增加一个变量都需要在拷贝构造里面修改对应的方法

## 第三章 资源管理
### 13 以对象管理资源

主要是为了防止在delete语句执行前return，所以需要用对象来管理这些资源。这样当控制流离开f以后，该对象的析构函数会自动释放那些资源。
例如shared_ptr就是这样的一个管理资源的对象。他是在自己的析构函数里面做delete操作。所以如果自己需要管理资源的时候，也要在类内进行delete，通过对象来管理资源。

1. 获得资源后立即放进管理对象(managing object)内。
   1. 资源获取即初始化。(Resource Acquisition Is Initialization, RAII)
2. 管理对象(managing object)运用析构函数确保资源被释放。

**总结：**
+ 建议使用shared_ptr，不建议使用auto_ptr，更不建议自己手动管理内存。
+ 如果需要自定义shared_ptr，请通过定义自己的资源管理类来对资源进行管理。

### 14 在资源管理类中小心copying行为

在资源管理类里面，如果出现了拷贝复制行为的话，需要注意这个复制具体的含义，从而保证和我们想要的效果一样

思考下面代码在复制中会发生什么：
```cpp
    class Lock{
    public:
        explicit Lock(Mutex *pm):mutexPtr(pm){
            lock(mutexPtr);//获得资源锁
        }
        ~Lock(){unlock(mutexPtr);}//释放资源锁
    private:
        Mutex *mutexPtr;
    }
    Lock m1(&m)//锁定m
    Lock m2(m1);//好像是锁定m1这个锁。。而我们想要的是除了复制资源管理对象以外，还想复制它所包括的资源（deep copy）。通过使用shared_ptr可以有效避免这种情况。
```

需要注意的是：copy函数有可能是编译器自动创建出来的，所以在使用的时候，一定要注意自动生成的函数是否符合我们的期望。

**总结：**
+ 复制RAII对象（Resource Acquisition Is Initialization）必须一并复制他所管理的资源（深拷贝）
+ 普通的RAII做法是：禁止拷贝，使用引用计数方法

### 15 在资源管理类中提供对原始资源的访问

例如：shared_ptr<>.get()这样的方法，或者->和*方法来进行取值。但是这样的方法可能稍微有些麻烦，有些人会使用一个隐式转换，但是经常会出错：
```cpp
class Font; class FontHandle;
void changeFontSize(FontHandle f, int newSize){    }//需要调用的API

Font f(getFont());
int newFontSize = 3;
changeFontSize(f.get(), newFontSize);//显式的将Font转换成FontHandle

class Font{
    operator FontHandle()const { return f; }//隐式转换定义
}
changeFontSize(f, newFontSize)//隐式的将Font转换成FontHandle

但是容易出错，例如
Font f1(getFont());
FontHandle f2 = f1;就会把Font对象换成了FontHandle才能复制
```

**总结：**
+ 每一个资源管理类RAII都应该有一个直接获得资源的方法
+ 隐式转换对客户比较方便，显式转换比较安全，具体看需求

### 16 成对使用new和delete时要采取相同形式

**总结：**
+ 即： 使用new[]的时候要使用delete[], 使用new的时候一定不要使用delete[]

### 17 以独立语句将new的对象置入智能指针

由于编译器的优化，同一条语句中的代码执行顺序并不完全会按照预期。
```cpp
int priority();
void processWidget(shared_ptr<Widget> pw, int priority);
processWidget(new Widget, priority());// 错误，这里函数是explicit的，不允许隐式转换（shared_ptr需要给他一个普通的原始指针
processWidget(shared_ptr<Widget>(new Widget), priority()) // 可能会造成内存泄漏

内存泄漏的原因为：先执行new Widget，再调用priority， 最后执行shared_ptr构造函数，那么当priority的调用发生异常的时候，new Widget返回的指针就会丢失了。当然不同编译器对上面这个代码的执行顺序不一样。所以安全的做法是：

shared_ptr<Widget> pw(new Widget);
processWidget(pw, priority());
```

**总结：**
+ 凡是有new语句的，尽量放在单独的语句当中，特别是当使用new出来的对象放到智能指针里面的时候

## 第四章 设计与声明
### 18 让接口容易被正确使用，不易被误用

要思考用户有可能做出什么样子的错误，考虑下面的代码：
```cpp
    Date(int month, int day, int year);
    这一段代码可以有很多问题，例如用户将day和month顺序写反（因为三个参数都是int类型的），可以修改成：
    Date(const Month &m, const Day &d, const Year &y);//注意这里将每一个类型的数据单独设计成一个类，同时加上const限定符
    为了让接口更加易用，可以对month加以限制，只有12个月份
    class Month{
        public:
        static Month Jan(){return Month(1);}//这里用函数代替对象，主要是方式第四条：non-local static对象的初始化顺序问题
    }
    
    而对于一些返回指针的问题函数，例如：
    Investment *createInvestment();//智能指针可以防止用户忘记delete返回的指针或者delete两次指针，但是可能存在用户忘记使用智能指针的情况，那么方法：
    std::shared_ptr<Investment> createInvestment();就可以强制用户使用智能指针，或者更好的方法是另外设计一个函数：
    std::shared_ptr<Investment>pInv(0, get)
```

**总结：**
+ 好的接口很容易被正确使用，不容易被误用。你应该努力设计好的接口。
+ “促进正确使用”的办法包括接口的一致性，以及与内置类型的行为兼容。
+ “阻止误用”的办法包括建立新类型、限制类型上的操作，束缚对象值，以及消除客户的资源管理责任。
+ shared_ptr支持定制删除器，从而防范dll问题，可以用来解除互斥锁等。

### 19 设计class犹如设计type

如何设计class：
+ 新的class对象应该被如何创建和构造
+ 对象的初始化和赋值应该有什么样的差别（不同的函数调用，构造函数和赋值操作符）
+ 新的class如果被pass by value（以值传递），意味着什么（copy构造函数）
+ 什么是新type的“合法值”（成员变量通常只有某些数值是有效的，这些值决定了class必须维护的约束条件）
+ 新的class需要配合某个继承图系么（会受到继承类的约束）
+ 新的class需要什么样的转换（和其他类型的类型变换）
+ 什么样的操作符和函数对于此type而言是合理的（决定声明哪些函数，哪些是成员函数）
+ 什么样的函数必须为private的 
+ 新的class是否还有相似的其他class，如果是的话就应该定义一个class template
+ 你真的需要一个新type么？如果只是定义新的derived class或者为原来的class添加功能，说不定定义non-member函数或者templates更好

**总结：**
+ Class的设计就是type的设计，在定义一个新type前，请确定你已经考虑过本条款覆盖的所有讨论主题。

### 20 以pass-by-reference-to-const替换pass-by-value

使用常引用传递替换值传递，可以避免构造函数和析构函数的开销，也能避免切割问题。

**总结：**
+ 尽量以"常引用传递"替换"值传递"，前者通常比较高效，并可避免切割问题(slice problem)。
+ 以上规则并不适用于内置类型，以及STL的迭代器和函数对象，对他们而言，使用值传递更为合适。

### 21 必须返回对象时，别妄想返回其reference

主要是很容易返回一个已经销毁的局部变量，如果想要在堆上用new创建的话，则用户无法delete，如果想要在全局空间用static的话，也会出现大量问题,所以正确的写法是：
```cpp
    inline const Rational operator * (const Rational &lhs, const Rational &rhs){
        return Rational(lhs.n * rhs.n, lhs.d * rhs.d);
    }
```
当然这样写的代价就是成本太高，效率会比较低

**总结：**
+ 绝不要返回指针和引用指向一个局部栈对象(函数返回后，局部对象被销毁)，或返回引用指向一个堆对象(容易无法释放堆内存，造成内存泄漏)，或返回指针和引用指向一个局部静态变量而有可能同时需要多个这样的对象(涉嫌多线程安全、返回两个相同的静态变量判断无条件相等问题)。
+ 最佳实践是，按值返回即可。

### 22 将成员变量声明为private

应该将成员变量弄成private，然后用过public的成员函数来访问他们，这种方法的好处在于可以更精准的控制成员变量，包括控制读写，只读访问等。

同时，如果public的变量发生了改变，如果这个变量在代码中广泛使用，那么将会有很多代码遭到了破坏，需要重新写

另外protected 并不比public更具有封装性，因为protected的变量，在发生改变的时候，他的子类代码也会受到破坏

**总结：**
+ 切记将成员变量声明为private，这可赋予客户访问数据的一致性、可细微划分访问控制、允许约束条件获得保证，并提供class作者以充分的实现弹性。
+ protected 并不比 public更具封装性。

### 23 以non-member、non-friend替换member函数

```cpp
区别如下：
class WebBrowser{
    public:
    void clearCache();
    void clearHistory();
    void removeCookies();
}

member 函数：
class WebBrowser{
    public:
    ......
    void clearEverything(){ clearCache(); clearHistory();removeCookies();}
}

non-member non-friend函数：
void clearBrowser(WebBrowser& wb){
    wb.clearCache();
    wb.clearHistory();
    wb.removeCookies();
}
```

这里的原因是：member可以访问class的private函数，enums，typedefs等，但是non-member函数则无法访问上面这些东西，所以non-member non-friend函数更好

这里还提到了namespace的用法，namespace可以用来对某些便利函数进行分割，将同一个命名空间中的不同类型的方法放到不同文件中(这也是C++标准库的组织方式)，例如：

```cpp
"webbrowser.h"
namespace WebBrowserStuff{
    class WebBrowser{...};
    //所有用户需要的non-member函数
}

"webbrowserbookmarks.h"
namespace WebBrowserStuff{
    //所有与书签相关的便利函数
}
```

非成员非友元函数替换成员函数：
1. 增加封装性：当一个函数不需要直接访问类的私有成员时，将其作为非成员函数可以实现更好的封装。这样做可以限制对类内部实现的直接访问，从而减少对类内部细节的依赖。
2. 提高包裹弹性：非成员函数可以更容易地在不同的上下文中被重用，因为它们不依赖于特定的类实例。这样的设计使得代码更加模块化，便于维护和测试。
3. 增强功能扩展性：非成员函数可以更容易地进行扩展，因为它们不直接绑定到类的层次结构上。这意味着可以在不改变原有类的情况下添加新的功能，有助于保持类的稳定和减少修改带来的风险。
4. 减少编译依赖：非成员函数通常不会增加类的编译依赖，这有助于提高编译效率，尤其是在大型项目中。

**总结：**
+ 宁可拿非成员非友元函数替换成员函数，这样做可以增加封装性，包裹弹性和技能扩充性。

### 24.若所有参数皆需类型转换，请为此采用non-member函数
例如想要将一个int类型变量和Rational变量做乘法，如果是成员函数的话，发生隐式转换的时候会因为不存在int到Rational的类型变换而出错：
```cpp
成员函数：
class Rational{
public:
    Rational(int numerator = 0, int denominator = 1);
    int numerator() const;
    int denominator() const;
    const Rational operator* (const Rational& rhs)const;

private:
    int numerator;   // 分子
    int denominator; // 分母
}
Rational oneHalf;
result = oneHalf * 2;
result = 2 * oneHalf;//出错，因为没有int转Rational函数

非成员函数
class Rational{}
const Rational operator*(const Rational& lhs, const Rational& rhs)
{
    return (lhs.numerator() * rhs.numerator(), lhs.denominator() * rhs.denominator());
}
```

成员函数的反面是非成员函数，而非友元函数。

**总结：**
+ 如果你需要为某个函数的所有参数(包括this指针所指的那个隐喻参数)进行类型转换，那么这个函数必须是个non-member。

### 25 考虑写出一个不抛异常的swap函数
```cpp
修改前代码：
class Widget{
    public:
    Widget& operator=(const Widget& rhs){
        *pImpl = *(rhs.pImpl);//低效
    }
    private:
    WidgetImpl* pImpl;
}

修改后代码：
namespace WidgetStuff{
    template<typename T>
    class Widget{
        void swap(Widget& other){
            using std::swap;      //此声明是std::swap的一个特例化，
            swap(pImpl, other.pImpl);
        }
    };
    ...
    template<typename T>           //non-member swap函数
    void swap(Widget<T>& a, Widget<T>& b){//这里并不属于 std命名空间
        a.swap(b);
    }    
}
```

**总结：**
+ 当std::swap对我们的类型效率不高的时候，应该提供一个swap成员函数，且保证这个函数不抛出异常（因为swap是要帮助class提供强烈的异常安全性的）
+ 如果提供了一个member swap，也应该提供一个non-member swap调用前者，对于classes（而不是templates），需要特例化一个std::swap
+ 调用swap时应该针对std::swap使用using std::swap声明，然后调用swap并且不带任何命名空间修饰符
+ 不要再std内加对于std而言是全新的东西（不符合C++标准）

## 第五章 实现 (Implementations)
### 26 尽可能延后变量定义式的出现时间

主要是防止变量在定义以后没有使用，影响效率，应该在用到的时候再定义，同时通过default构造而不是赋值来初始化。

对于循环，将变量定义在循环外好还是循环内好？
循环外：1个构造函数 + 1个析构函数 + n个赋值操作
循环内：n个构造函数 + n个析构函数
当一组赋值操作开销低于一组构造函数和一组析构函数时，使用循环外好，否则循环内。
考虑前者作用域比后者大，除非效率高度敏感，否则选择循环内。

**总结：**
+ 尽可能延后变量表达式的出现。这样做可增加程序的清晰度并改善程序效率。

### 27 尽量不要进行强制类型转换

1. 主要是因为：
   1. 从int转向double容易出现精度错误
   2. 将一个类转换成他的父类也容易出现问题

2. 旧式转型：
   1. (T) expression
   2. T(expression)

3. 新式转型：
   1. const_cast:用于去除或添加常量性。需要注意的是，使用const_cast去除const属性后，修改原本应该是const的对象的值是未定义行为。
   2. dynamic_cast:用于在类层次结构中进行安全的向下转型（即从基类指针或引用转换为派生类指针或引用）。它会检查转换是否有效，如果无效则返回空指针（对于指针类型）或引发异常（对于引用类型）。需要注意的是，dynamic_cast只能用于含有虚函数的类。
   3. reinterpret_cast:用于不同类型之间的低级强制转换，它将源类型的二进制表示直接解释为目标类型。这种转换通常用于处理指针和整数之间的转换，但需要谨慎使用，因为它可能导致未定义行为。
   4. static_cast:用于在相关类型之间进行转换，如非const转换为const、void指针转换为其他类型指针、基本数据类型之间的转换等。需要注意的是，它不能用于不兼容类型之间的转换。

**总结：**
+ 尽量避免转型，特别是在注重效率的代码中避免dynamic_cast，试着用无需转型的替代设计
+ 如果转型是必要的，试着将他封装到韩束背后，让用户调用该函数，而不需要在自己的代码里面转型
+ 如果需要转型，使用新式的static_cast等转型，比原来的（int）好很多（更明显，分工更精确）

### 28 避免返回handles指向对象内部成分
```cpp
修改前代码：
class Rectangle{
    public:
    Point& upperLeft() const { return pData->ulhc; }
    Point& lowerRight() const { return pData->lrhc; }
}
如果修改成：
class Rectangle{
    public:
    const Point& upperLeft() const { return pData->ulhc; }
    const Point& lowerRight() const { return pData->lrhc; }
}
则仍然会出现悬吊的变量，例如：
const Point* pUpperLeft = &(boundingBox(*pgo).upperLeft());
boundingBox会返回一个temp的新的，暂时的Rectangle对象，在这一整行语句执行完以后，temp就变成空的了，就成了悬吊的变量
```

**总结：**
+ 尽量不要返回指向private变量的指针引用等
+ 如果真的要用，尽量使用const进行限制，同时尽量避免悬吊的可能性

### 29. 为“异常安全”而努力是值得的

异常安全函数具有以下三个特征之一：
+ 如果异常被抛出，程序内的任何事物仍然保持在有效状态下，没有任何对象或者数据结构被损坏，前后一致。在任何情况下都不泄露资源，在任何情况下都不允许破坏数据，一个比较典型的反例：
+ 如果异常被抛出，则程序的状态不被改变，程序会回到调用函数前的状态
+ 承诺绝不抛出异常
```cpp
原函数：
class PrettyMenu{
    public:
    void changeBackground(std::istream& imgSrc); //改变背景图像
    private:
    Mutex mutex; // 互斥器
};

void changeBackground(std::istream& imgSrc){
    lock(&mutex);               //取得互斥器
    delete bgImage;             //摆脱旧的背景图像
    ++imageChanges;             //修改图像的变更次数
    bgImage = new Image(imgSrc);//安装新的背景图像
    unlock(&mutex);             //释放互斥器
}
```
当异常抛出的时候，这个函数就存在很大的问题：
+ 不泄露任何资源：当new Image(imgSrc)发生异常的时候，对unlock的调用就绝不会执行，于是互斥器就永远被把持住了
+ 不允许数据破坏：如果new Image(imgSrc)发生异常，bgImage就是空的，而且imageChanges也已经加上了
```cpp 
修改后代码：
void PrettyMenu::changeBackground(std::istream& imgSrc){
    Lock ml(&mutex);    //Lock是第13条中提到的用对象管理资源的类
    bgImage.reset(new Image(imgSrc));
    ++imageChanges; //放在后面
}
```

时间不断前进，我们与时俱进！

**总结：**
+ 异常安全函数的三个特征
+ 第二个特征往往能够通过copy-and-swap实现出来，但是并非对所有函数都可实现或具备现实意义
+ 函数提供的异常安全保证，通常最高只等于其所调用各个函数的“异常安全保证”中最弱的那个。即函数的异常安全保证具有连带性
                  
### 30. 透彻了解inlining的里里外外

inline 函数的过度使用会让程序的体积变大，内存占用过高

而编译器是可以拒绝将函数inline的，不过当编译器不知道该调用哪个函数的时候，会报一个warning

尽量不要为template或者构造函数设置成inline的，因为template inline以后有可能为每一个模板都生成对应的函数，从而让代码过于臃肿
同样的道理，构造函数在实际的过程中也会产生很多的代码，例如下面的：
```cpp
    class Derived : public Base{
        public:
        Derived(){} // 看起来是空白的构造函数
    }
    实际上：
    Derived::Derived{
        //100行异常处理代码
    }
```

**总结：**
+ 将大多数inlining限制在小型、被频繁调用的函数身上，这可使日后的调试过程和二进制升级(binary upgradability)更容易，也可使潜在的代码膨胀问题最小化，使程序的速度提升机会最大化。
+ 不要只是因为function templates出现在头文件，就将它们声明为inline 

### 31. 将文件间的编译依存关系降至最低

这个关系其实指的是一个文件包含另外一个文件的类定义等

那么如何实现解耦呢,通常是将实现定义到另外一个类里面，如下：
```cpp
原代码：
class Person{
private
    Dates m_data;
    Addresses m_addr;
}

添加一个Person的实现类，定义为PersonImpl，修改后的代码：
class PersonImpl;
class Person{
private:
    shared_ptr<PersonImpl> pImpl;
}
```
在上面的设计下,就实现了解耦，即“实现和接口分离”

与此相似的接口类还可以使用全虚函数
```cpp
class Person{
    public:
    virtual ~Person();
    virtual std::string name() const = 0;
    virtual std::string birthDate() const = 0;
}
```
然后通过继承的子类来实现相关的方法

这种情况下这些virtual函数通常被成为factory工厂函数

1. 使用前向声明：在需要使用某个类的对象指针或引用时，可以使用前向声明来避免包含整个类的头文件。例如，如果有一个类A，需要在类B中使用A的对象指针，可以在B的头文件中使用前向声明：
  
2. 减少头文件的包含：尽量避免在头文件中包含不必要的头文件，特别是那些可能导致大量代码被包含的头文件，如、等。可以将这些头文件的包含放到源文件中，以减少编译时间。

3. 使用#pragma once：在头文件的开头使用#pragma once指令，可以避免头文件被重复包含，从而减少编译时间。但这种方法可能不适用于所有编译器，因此建议优先使用#ifndef/#define/#endif方法来防止头文件重复包含。

4. 将实现细节放在源文件中：尽量将类的实现细节（如成员函数的实现）放在源文件中，而不是在头文件中。这样可以减少头文件的大小，降低编译时间。

5. 使用Pimpl技巧：对于复杂的类，可以使用Pimpl（Pointer to Implementation）技巧将一个类的实现细节封装在一个私有的派生类中，并通过一个指向该派生类的指针来访问这些实现细节。从而减少头文件的依赖关系。这样可以提高代码的可维护性和可读性。

Pimpl例子：
```cpp
// MyClass.h
class MyClass {
public:
    MyClass();
    ~MyClass();
    void doSomething();

private:
    class Impl; // 前向声明
    Impl* pimpl; // 指向实现的指针
};

// MyClass.cpp
#include "MyClass.h"
#include <iostream>

class MyClass::Impl {
public:
    Impl() {
        std::cout << "Impl constructor" << std::endl;
    }

    ~Impl() {
        std::cout << "Impl destructor" << std::endl;
    }

    void doSomething() {
        std::cout << "Impl doSomething" << std::endl;
    }
};

MyClass::MyClass() : pimpl(new Impl()) {}
MyClass::~MyClass() { delete pimpl; }
void MyClass::doSomething() { pimpl->doSomething(); }

上述代码展示了一个简单的使用Pimpl的例子。

首先，在头文件MyClass.h中定义了一个名为MyClass的类。这个类有一个私有的派生类Impl，用于封装实现细节。同时，MyClass还包含一个指向Impl的指针pimpl，用于访问这些实现细节。

接下来，在源文件MyClass.cpp中实现了MyClass和Impl类。Impl类包含了一些成员函数，如构造函数、析构函数和doSomething()函数。这些函数的具体实现可以根据需要进行修改。

在MyClass的构造函数中，通过new操作符创建了一个Impl对象，并将其地址赋给pimpl指针。这样，MyClass就可以通过pimpl指针来访问Impl类的实现细节。

同样地，在MyClass的析构函数中，通过delete操作符释放了pimpl指针所指向的Impl对象。

最后，在MyClass的成员函数doSomething()中，通过调用pimpl->doSomething()来执行Impl类中的相应操作。

通过使用Pimpl技巧，将实现细节封装在私有派生类中，可以降低头文件之间的依赖关系，提高代码的可维护性和可读性。
```

**总结：**
+ 应该让文件依赖于声明而不依赖于定义，可以通过上面两种方法实现
+ 程序头文件应该有且仅有声明

## 第六章 继承和面向对象设计

### 32 确定你的public继承塑模出is-a关系

public类继承指的是单向的更一般化的，例如：
```cpp
class Student : public Person{...};
```
其意义指的是student是一个person，但是person不一定是一个student。

这里经常会出的错误是，将父类可能不存在的功能实现出来，例如：
```cpp    
class Bird{
    virtual void fly();
}
class Penguin:public Bird{...};//企鹅是不会飞的
```
这个时候就需要通过设计来排除这种错误，例如通过定义一个FlyBird

**总结：**
+ "public继承"意味着 is-a。适用于base classed 身上的每一件事情一定也适用于derived classes 身上，因为每一个 derived class对象也都是一个 base class 对象。
+ 公开继承下，满足父类的地方，一定满足子类，反之满足子类的地方不一定满足父类。

### 33  避免遮掩继承而来的名称

举例：
```cpp
class Base{
    public:
    virtual void mf1() = 0;
    virtual void mf1(int);
    virtual void mf2();
    void         mf3();
    void         mf3(double);
}
class Derived:public Base{
    public:
    virtual void mf1();
    void         mf3();
}
```
这种问题可以通过 
```cpp
using Base::mf1;

或者
virtual void mf1(){//转交函数
    Base::mf1();
}
```
    来解决，但是尽量不要出现这种遮蔽的行为

**总结：**
+ 派生类内的名称会遮掩base classes 内的名称。
+ 可以通过using 或者转交函数来解决。

### 34 区分接口继承和实现继承

1. **所谓接口继承，就是派生类只继承函数的接口，也就是声明；而实现继承，就是派生类同时继承函数的接口和实现。**
(1)接口继承（Interface Inheritance）
    1. 定义：接口继承指的是派生类只继承基类的函数声明（即接口），而不继承具体的实现细节。这意味着派生类获得了调用这些函数的能力，但必须自己提供函数的具体实现。
    2. 目的：主要用来表达类型之间的“is-a”关系，以及确保多态行为。它允许程序员设计出可以处理基类指针或引用，但执行派生类操作的代码，这是通过虚函数来实现的。
    3. 示例：使用纯虚函数定义接口，强制派生类实现这些函数，但不提供具体行为。
(2)实现继承（Implementation Inheritance）
    1. 定义：实现继承不仅包括接口的继承，还包含了函数的具体实现。派生类不仅知道要调用哪些函数，还直接继承了这些函数的执行逻辑。
    2. 目的：当希望派生类共享基类的实现或者基类提供的功能足够通用，无需在派生类中重新实现时，使用实现继承可以减少代码重复并提高代码复用。
    3. 示例：基类提供了具体函数的实现，派生类可以直接使用这些实现，也可以选择覆盖（override）它们。

2. 虚函数、纯虚函数、非虚函数。
（1）虚函数：
虚函数是指一个类中你希望重载的成员函数，当你用一个基类指针或引用指向一个继承类对象的时候，你调用一个虚函数，实际调用的是继承类的版本。——MSDN
虚函数用来表现基类和派生类的成员函数之间的一种关系.
虚函数的定义在基类中进行,在需要定义为虚函数的成员函数的声明前冠以关键字 virtual.
基类中的某个成员函数被声明为虚函数后,此虚函数就可以在一个或多个派生类中被重新定义.
在派生类中重新定义时,其函数原型,包括返回类型,函数名,参数个数,参数类型及参数的先后顺序,都必须与基类中的原型完全相同.
虚函数是重载的一种表现形式,是一种动态的重载方式.
（2）纯虚函数：
纯虚函数在基类中没有定义，它们被初始化为0。
任何用纯虚函数派生的类，都要自己提供该函数的具体实现。
定义纯虚函数
virtual void fun(void) = 0;
（3）非虚函数：
一般成员函数，无virtual关键字修饰。

3. 将虚函数、纯虚函数和非虚函数的功能与接口继承与实现继承联系起来：
（1）声明一个纯虚函数（pure virtual）的目的是为了让派生类只继承函数接口，也就是上面说的接口继承。
纯虚函数一般是在不方便具体实现此函数的情况下使用。也就是说基类无法为继承类规定一个统一的缺省操作，但继承类又必须含有这个函数接口，并对其分别实现。但是，在C++中，我们是可以为纯虚函数提供定义的，只不过这种定义对继承类来说没有特定的意义。因为继承类仍然要根据各自需要实现函数。
通俗说，纯虚函数就是要求其继承类必须含有该函数接口，并对其进行实现。是对继承类的一种接口实现要求，但并不提供缺省操作，各个继承类必须分别实现自己的操作。
（2）声明非纯虚函数（impure virtual）的目的是让继承类继承该函数的接口和缺省实现。
与纯虚函数唯一的不同就是其为继承类提供了缺省操作，继承类可以不实现自己的操作而采用基类提供的默认操作。
（3）声明非虚函数（non-virtual）的目的是为了令继承类继承函数接口及一份强制性实现。
相对于虚函数来说，非虚函数对继承类要求的更为严格，继承类不仅要继承函数接口，而且也要继承函数实现。也就是为继承类定义了一种行为。(因此，对于派生类绝对不要重新定义继承而来的非虚函数，可参见条款36)

4. 理解：
   1. 纯虚函数：主要用于定义接口，强制派生类实现特定的函数，使得基类成为一个接口规范。
   2. 虚函数：设计接口时，预期将来会有多种实现方式，或者需要在运行时根据对象类型选择合适的函数实现。
   3. 普通成员函数：适用于那些不需要在派生类中重写的行为，或不涉及基类和派生类之间动态转换的场景。

**总结：**
+ 接口继承和实现继承不同，在public继承下，derived classes总是继承base的接口
+ pure virtual函数只具体指定接口继承
+ 简朴的（非纯）impure virtual函数具体指定接口继承以及缺省实现继承
+ non-virtual函数具体指定接口继承以及强制性的实现继承

### 35 考虑 virtual 函数以外的其他选择
1. **方法一，基于虚函数的方法**
在人物角色的基类增加一个成员函数heathValue，返回一个整数，表示人物的健康程度，并将声明为virtual．
```cpp
class GameCharacter {
public:
    virtual int healthValue() const;
    ...
};
```
heathValue声明为虚函数，因而派生类可以重新定义它，从而获得达到不同的人物可能不同的方式计算他们的健康指数的要求．

但是没有声明为纯函数，这表示会有个计算健康指数的缺省算法．

2. **方法二，藉由Non-Virtual Interface手法实现Template Method模式 NVI**
该设计是令客户通过public non-virtual成员函数间接调用private virtual函数，相当对virtual函数进行一层的包装，可以称为是virtual函数的外覆器(warpper).
```cpp
class GameCharacter {
public:
    int healthValue() const
    {
        ...
        int retVal = doHealthValue();
        ...
        return retVal;
    }
    ...
private:
    virtual int doHealthValue() const
    {
        ...
    }
};
```
NVI手法的一个优点可以确保在一个virtual函数被调用之前设定好适当的场景，并在调用结束之后清理场景．

"事前工作"可以包括锁定互斥器，制造运转日志记录项，验证class约束条件，验证函数先决条件等等．

"事后工作"可以包括互斥器解除锁定，验证函数的事后条件，再次验证class约束条件等等．

3. **方法三，藉由函数指针实现Strategy模式**
每个人物的构造函数接受一个指针，指向一个健康计算函数，调用该函数进行实际计算．
```cpp
class GameCharacter;
int defaultHealthCalc(const GameCharacter& gc);
class GameCharacter {
public:
    typedef int (*HealthCalcFunc)(const GameCharacter&);
    explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc):healthFunc(hcf)
    {}
    int healthValue() const
    {
        return healthFunc(*this);
    }
    ...
private:
    HealthCalcFunc healthFunc;
};
```
该方法的优点，同一人物类型之不同实体可以有不同的健康计算函数，只需要在构造实例时，传入不同的计算函数的指针．

某已知人物之健康计算函数可在运动期变更，可以在GameCharacter里提供一个成员函数setHealthCalc，用来替换当前的健康指数计算函数．

该方法的缺点，如果需要利用GameCharacter的non-public信息进行计算健康指数时，由于计算函数是non-member non-friend函数，将出现无法访问的问题．

如果让计算函数访问成功，则需要降低GameCharacter的封装性．

4. 方法四，藉由tr1::function完成Strategy模式

不再使用函数指针，而是改用一个类型为tr1::function的对象．

可以是函数指针，函数对象，或成员函数指针，只要其签名式兼容于需求端．
```cpp
class GameCharacter;
int defaultHealthCalc(const GameCharacter& gc);
class GameCharacter {
public:
    typedef    std::tr1::function<int (const GameCharacter&)> HealthCalcFunc;
    explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc):healthFunc(hcf)
    {}
    int healthValue() const
    {
        return healthFunc(*this);
    }
    ...
private:
    HealthCalcFunc healthFunc;
};

short calcHealth(const GameCharacter&);
struct HealthCalculator {
    int operator()(const GameCharacter&) const
    {...}
};
class GameLevel {
public:
    float health(const GameCharacter&) const;
    ...
};
class EvilBadGuy:public GameCharacter{
...
};
EvilBadGuy ebg1(calcHealth);            //函数
EvilBadGuy ebg2(HealthCalculator());    //函数对象
GameLevel currentLevel;
EvilBadGuy ebg3(std::tr1::bind(&GameLevel::health,currentLevel,_1);    //成员函数
```
优点，以tr1::function替换函数指针之后，可以允许客户在计算人物健康指数时使用任何兼容的可调用物。

5. 方法五，古典的Strategy模式

将健康计算函数做成一个分离的继承体系中的virtual成员函数．

每个GameCharacter对象都内含一个指针，指向一个来自HealthCalcFunc继承体系的对象
```cpp
class GameCharacter;
class HealthCalcFunc {
public:
    ...
    virtual int calc(const GameCharacter& gc) const
    {...}
    ...
};
HealthCalcFunc defaultHealthCalc;
class GameCharacter {
public:
    explicit GameCharacter(HealthCalcFunc* phcf = &defaultHealthCalc):pHealthFunc(phcf)
    {}
    int healthValue() const
    {
        return pHealthFunc->calc(*this);
    }
    ...
private:
    HealthCalcFunc* pHealthFunc;
};
```
优点，只要为HealthCalcFunc继承体系添加一个派生类，就可以将一个既有的健康算法纳入使用。

启发：针对具体的应用问题，需要认真分析其应用的特点，以及应用的后续扩展等问题，再从众多的方法，选取最合适的方法。

不能先入为主的，随便的套用一个方法，这样可能会导致应用的后续扩展问题．总之，遇到问题，先想，再比较，最后确定方案。

**总结：**:这一节表示当我们为了解决问题而寻找某个特定设计方法时，不妨考虑virtual函数的替代方案
+ 使用NVI手法，他是用public non-virtual成员函数包裹较低访问性（private和protected）的virtual函数
+ 将virtual函数替换成“函数指针成员变量”，这是strategy设计模式的一种表现形式
+ 以tr1::function成员变量替换virtual函数，因而允许使用任何可调用物（callable entity）搭配一个兼容与需求的签名式
+ 将继承体系内的virtual函数替换成另一个继承体系内的virtual函数

+ 将机能从成员函数移到class外部函数，带来的一个缺点是：非成员函数无法访问class的non-public成员
+ tr1::function对象就像一般函数指针，这样的对象可接纳“与给定之目标签名式兼容”的所有可调用物（callable entities）


### 36 绝不重新定义继承而来的non-virtual函数
```cpp
class B{
public:
    void mf();
}
class D : public B{
public:
    void mf();
};

D x;

B *pB = &x; pB->mf(); //调用B版本的mf
D *pD = &x; pD->mf(); // 调用D版本的mf
```
即使不考虑这种代码层的差异，如果这样重定义的话，也不符合之前的“每一个D都是一个B”的定义\

**总结：**
- 绝对不要重新定义继承而来的non-virtual

### 37 绝不重新定义继承而来的缺省参数值
```cpp
class Shape{
public:
    enum ShapeColor {Red, Green, Blue};
    virtual void draw(ShapeColor color=Red)const = 0;
};
class Rectangle : public Shape{
public:
    virtual void draw(ShapeColor color=Green)const;//和父类的默认参数不同
}
Shape* pr = new Rectangle; // 注意此时pr的静态类型是Shape，但是他的动态类型是Rectangle
pr->draw(); //virtual函数是动态绑定，而缺省参数值是静态绑定，所以会调用Red
```

解决方案之一就是NVI(non-virtual interface)条款35做法
```cpp
class Shape{
public:
    enum Color{RED,GREEN,BLUE};
    void draw(Color color = RED) const{
           ...
           doDraw(color);
           ...
    }
    ...
private:
   virtual void doDraw(Color color) const = 0;  
};

class Circle:public Shape{
    ...
private:
    virtual void doDraw(Color color){ ... }
};
```
由于draw是non-virtual而non-virtual绝对不会被重新改写(条款36),所以color的缺省值总是为RED。

**总结：**
- 绝对不要重新定义一个继承而来的缺省参数值，因为缺省参数值都是静态绑定，而virtual函数－你唯一应该覆写的东西－却是动态绑定。

### 38 通过复合塑模出has-a或"根据某物实现出

1. Is-a（继承）：使用public继承表达“是一个”关系，意味着派生类是基类的一种特殊形式，继承了基类的所有属性和行为。这种关系在UML中通常表示为泛化（Generalization）。

2. Has-a（复合）：通过包含另一个类的对象作为成员变量来表达“有一个”关系，表明一个类拥有另一个类的实例，利用该实例来实现部分功能。这对应于UML中的聚合或组合（Aggregation/Composition）。

3. 复合的优势：
- 设计灵活性：复合允许在不修改现有类的情况下轻松添加或替换组件，从而提供更高的设计灵活性和可扩展性。
- 降低耦合度：相比继承，复合降低了类之间的耦合，因为修改基类不会直接影响到使用复合的类，这有助于遵循开放/封闭原则。
- 清晰的职责划分：通过明确哪些功能是由内部对象处理的，哪些是由包含它的类处理的，可以更清晰地定义类的职责。

4. 根据某物实现出：
- 这个表述强调了实现层面的关联，意味着一个类的实现是基于另一个类的功能，但并不意味着它是那个类的一个子类型。使用复合，一个类可以内部实现对另一个类的使用，从而“根据某物实现出”某些功能，而不是直接通过继承关系来表达这种功能的延伸。

5. 何时使用复合：
- 当两个类之间不存在明确的is-a关系，或者这种关系不是基于行为的自然延伸时。
- 当你希望实现松耦合、易于维护和测试的设计时。
- 当你想在不修改基类的前提下，能够自由地改变或扩展类的行为时。

**总结：**
- 复合的意义和public继承完全不同。
- 在应用域，复合意味着has-a(有一个)。在实现域，复合意味着is-implemented-in-terms-of(根据某物实现出)。

### 39 明智而审慎地使用private继承
1. 某派生类private继承于基类之后：
   1. 基类中的所有内容（不论是public、protected、private）在派生类中都是不可访问的
   2. 不能再将派生类对象转换为基类对象
   3. 基类仍然可以重写/隐藏基类的成员方法

2. private继承意为implemented-terms-of（根据某物实现出）：
假设你让class D以private继承于class B，用意为采用class B内的某些特性来实现class D，再无其他意义了
借助条款34的属于：private继承意味只有实现部分（也就是基类中已经实现的函数）被继承，接口部分（基类中只定义还未实现的）应该被省去

3. 与类的复合模式的关系：
在条款38中介绍了类的复合（composition）模式，其中类的复合也有着“is-implemented-in-terms-of”的意义
两者有着同样的意义，但是建议：尽可能使用复合，必要时才使用private继承
何时才必要使用private呢？
主要是当protected成员和/或virtual函数牵扯进来的时候。当派生类想要访问基类的protected成分或者基类想要重写一个或多个virtual函数
还有一种情况，就是当空间方面的利害关系足以踢翻private继承的支柱时（下面介绍）

4. EBO（空基类最优化）：在某些情况下，使用private继承可以利用空基类优化（Empty Base Optimization），使得派生类的大小不因为空基类的存在而增加。这对于设计轻量级对象特别有用。

5. 替代复合：在考虑是否使用private继承时，应首先考虑是否可以使用复合（即包含一个对象作为成员变量）来达到相同的设计目标。复合通常更为简单且直接，减少了继承的复杂性。

6. 特殊情况下的选择：尽管一般推荐使用复合，但在以下情况可能考虑private继承：
    1. 需要访问或覆盖基类的受保护成员。
    2. 基类中有虚函数，且派生类需要成为多态体系的一部分。
    3. 希望利用EBO来避免空基类带来的空间开销。

7. 审慎使用：由于private继承改变了基类成员的访问级别，并且可能引入额外的设计复杂度，因此应该在充分理解其后果后才使用，并且只有当复合或其他设计模式不适用或不足够时才考虑。

**总结：**
+ private继承意为“is-implemented-in-terms-of（根据某物实现出）”。它通常比复合（composition）的级别低。但是当derived class需要访问protected base class的成员，或需要重新定义继承而来的virtual函数时，这么设计时合理的
+ 和复合（composition）不同，private继承可以造成empty base最优化。这对致力于“对象尺寸最小化”的程序库开发者而言，可能很重要

### 40 明智而审慎地使用多重继承

1. 复杂性增加：多重继承导致类的继承关系网更复杂，可能难以理解和维护。开发者需要清楚地了解每个基类的角色和责任，以及它们如何协同工作。

2. 二义性问题：如果多个基类中存在同名成员（比如成员变量或函数），那么在派生类中直接访问这些成员时可能会引起二义性，需要使用作用域解析运算符（::）来明确指定来源。

3. 菱形继承问题：当多个基类中存在共同的基类时，如果不使用虚拟继承（virtual inheritance），会导致基类的实例在派生类中有多个副本，即所谓的“菱形问题”。虚拟继承可以解决这个问题，但会引入额外的大小、速度和初始化开销。

4. 设计考量：多重继承往往用于实现“混入”（Mix-in）特性，即向类添加特定功能而不改变其核心行为。在考虑使用多重继承前，应评估是否有其他更简单的设计模式（如复合、委托或策略模式）可以达到相同目的。

5. 接口清晰性：多重继承可能导致派生类的接口变得模糊不清，因为客户端可能不清楚哪些行为来自于哪个基类，这会影响代码的可读性和可维护性。

6. 使用原则：只有在确信多重继承能够带来显著的设计优势，并且其他替代方案不足以满足需求时，才应考虑使用。同时，应仔细规划继承结构，避免不必要的复杂性，并确保正确处理可能的二义性问题。

7. virtual 继承的成本会较高：
- 使用 virtual 继承的那些 classes 所产生的对象往往比使用 non-virtual 继承的体积要大；
- 访问 virtual base classes 成员变量时，也比访问 non-virtual base classes 的成员变量速度慢；
- virtual base classes 初始化由继承体系中的最低层（most derived）class 负责；

8. 多重继承正当用途:“public继承某个Interface class” 和 “private 继承某个协助实现的class”的两相组合。

**总结：**
+ 多重继承比单一继承复杂。它可可能导致新的歧义性，以及对virtual继承的需要。
+ virtual继承会增加大小、速度、初始化（及赋值）复杂度等等成本。如果virtual base classes不带任何数据，将是最具使用价值的情况。
+ 多重继承的确有正当用途。其中一个情节涉及“public继承某个Interface class”和“private继承某个协助实现的class”的两项组合。

## 第七章 模板与泛型编程
### 41 了解隐式接口和编译期多态
对于面向对象编程：以显式接口（explicit interfaces）和运行期多态（runtime polymorphism）解决问题：
```cpp    
    class Widget {
    public:
        Widget();
        virtual ~Widget();
        virtual std::size_t size() const;
        void swap(Widget& other); //第25条
    }
    
    void doProcessing(Widget& w){
        if(w.size()>10){...}
    }
```
+ 在上面这段代码中，由于w的类型被声明为Widget，所以w必须支持Widget接口，我们可以在源码中找出这个接口，看看他是什么样子（explicit interface），也就是他在源码中清晰可见
+ 由于Widget的某些成员函数是virtual，w对于那些函数的调用将表现运行期多态，也就是运行期间根据w的动态类型决定调用哪一个函数

在templete编程中：隐式接口（implicit interface）和编译器多态（compile-time polymorphism）更重要：
```cpp
    template<typename T>
    void doProcessing(T& w)
    {
        if(w.size()>10){...}
    }
```
+ 在上面这段代码中，w必须支持哪一种接口，由template中执行于w身上的操作来决定，例如T必须支持size等函数。这叫做隐式接口
+ 凡涉及到w的任何函数调用，例如operator>，都有可能造成template具现化，使得调用成功，根据不同的T调用具现化出来不同的函数，这叫做编译期多态

- 隐式接口：函数模板，类型不清楚，对我们来说接口是隐藏的。
- 显示接口：我们常规的头文件接口声明就是显示接口，明确了返回值，参数。
- 编译期多态：编译时实例化模板确定哪个重载函数被调用。
- 运行期多态：运行时哪一个virtual函数该被绑定。

**总结：**
- class和template都支持接口和多态。
- 对class而言接口是显示的。多态则是通过virtual函数发生于运行期。
- 对template而言，接口是隐式的。多态则通过template实例化和函数重载解析，发生于编译器。

### 42 了解typename的双重意义

模版声明有两种形式：
```cpp
template<class T> class Widget;
template<typename T> class Widget;
```
这里声明模版参数时，它们的意义完全相同。

不过对于typename在模版中除了声明模版参数外还有几处特别的用处要注意！
```cpp
template<typename C>
void print2nd(const C& container)
{
  C::const_iterator* x;
  ...
}
```
这里有个新名词要了解，嵌套从属类型：即属于模版类型C下的类型，形式：C::xxx。

上面对应的就是C::const_iterator，这里是有歧义的，C::const_iterator是一个类型了还是一个变量了，如果作为类型上面就是定义一个指针x，如果作为变量就是乘x。对于这种嵌套从属类型，编译器一般默认当变量处理。如果要当类型处理就必须在其前面加关键字typename。
```cpp
typename C::const_iterator* x;  // 这样就显示告诉编译器，C::const_iterator是一个自定义类型
```

另外对于嵌套从属类型前面加typename，有两处特例不能加。即不能出现在基类和成员初始化列表的嵌套从属类型里(除此之外都要加)。

```cpp
template<typename T>
class Derived : public Base<T>::Nested  // 不能加typename
{
 public:
  	explicit Derived(int x) : Base<T>::Nested(x)  // 不能加typename
    {
       typename Base<T>::Nested temp;   // 这里要加
    }
}
```

**总结：**
- 声明template参数时，前缀关键字class和typename可互换，意义一样。
- 请使用关键字typename标识嵌套从属类型，但不得在基类或成员初始化列表内使用。

### 43 注意处理模版化基类内的名称
```cpp
template<typename T>
class LoggingMsgSender : public MsgSender<T>  // 模版化基类
{
 public:
  	...
    void sendClearMsg(const MsgInfo& info)
    {
      	...
      	sendClear(info);  // 如果这个接口属于基类的，这里也不认识，因为基类是什么这时编译器不知道
       	...
    }
}
```
像上面的sendClear接口模版化基类里是否存在，编译器是不确定的，所以这种编译会报错。有下面3种方式解决这种问题，就是明确告诉编译器假设它存在。

1. 通过this->sendClear(info);调用，假设sendClear在this中。
2. 调用前加using声明using MsgSender<T>::sendClear;，明确告诉编译器sendClear在模版基类中。
3. 调用时明白指明，MsgSender<T>::sendClear(info);

**总结：**
- 可在派生类模版内通过this->指明基类模版的成员名称(1)，或者由一个明白写出的属于基类的修饰符完成(2, 3)。

### 44 将与参数无关的代码抽离template
**template是一个节省时间和避免代码重复的一个奇方妙法。**不再需要键入20个类似的class而每一个带有15个成员函数，你只需键入一个class template，留给编译器去实例化那20个你需要的相关class和300个函数。(它们只有在被使用时才会实例化)

template虽然给我们提供了方便，但是注意如果使用不当，很容易导致代码膨胀(执行文件变大)。其结果有可能源码看起来合身而整齐，但目标码却不是那么回事。在template代码中，重复是隐藏的，所以你必须训练自己去感受当template被实例化多次时可能发生的重复。
```cpp
template<typename T, std::size_t n>  // 这里T称为模版的类型参数，n是非类型参数
class SquareMatrix {
public:
  	...
    void invert();
}

// 实例化
SquareMatrix<double, 5> sm1;
sml.invert();

SquareMatrix<double, 10> sm2;
sm2.invert();
```
上面这段模版封装，多次实例化，其中invert也会实例多份，虽然它们二进制实现一样。这就是隐晦的重复代码
```cpp
template<typename T>
class SquareMatrixbase {
protected:
  	...
    void invert(std::size_t matrixSize);
  	...
}

template<typename T, std::size_t n>
class SqureMatrix : public SquareMatrixbase<T> {
private:
  	using SquareMatrixBase<T>::invert;
  	...
public:
  	...
    void invert() {
      	this->invert(n);
    }
}
```
把重复逻辑移到基类中，所有模版类共有，这样就减少了代码膨胀了。

本条款想表达的是使用template时要注意多次实例化后可能带来的代码重复，要尽量避免这种重复代码。这就是我的理解。

TODO: 翻译的请记住条款描述得有点抽象，没深刻理解～待日后回顾重新理解！

**总结：**
- template生成多个class和多个函数，所以任何template代码都不该与某个造成膨胀的template参数产生相依关系。
- 因非类型模版参数而造成的代码膨胀，往往可消除，做法是以函数参数或class成员变量替换template参数。
- 因类型参数而造成的代码膨胀，往往可以降低，做法是让带有完全相同二进制实现的代码共享，如放基类中。

### 45 使用成员函数模版接受所有兼容类型
本条款想要表达的是我们封装的模版所有操作行为要和普通类保持一致。即隐式行为要一致。如不同类型可隐式相互转换。
```cpp
template<typename T>
class SmartPrt {
 public:
  	SmartPrt(const SmartPrt& other);  //正常的copy构造函数，取消编译器自动生成
  
  	template<typename U>   // 泛化的copy构造函数(成员函数模版)，接受不同类型对象转换
  	SmartPrt(const SmartPrt<U>& other) : heldPtr(other.get())
    {
      	...
    }
  	T* get() const {return heldPtr;};
  	...
 private:
  	T* heldPtr;
}
```
不过注意泛化的成员函数(即成员函数模版)并不会影响编译器自动生成类默认函数规则。所以如果你要完全自定义类行为，默认产生的函数除了泛化版本，对应的正常化版本也要声明。
**总结：**
- 请使用成员函数模版生成可接受所有兼容类型的函数。
- 如果你声明成员函数模版用于泛化copy构造函数或赋值操作符，你还是需要声明对应正常的copy构造函数和赋值操作符函数。

### 46 需要类型转换时请为模版定义非成员函数
对应条款24，这里只是模版实现。规则一致，但它们写法上有所区别了。
```cpp
template<typename T>
class Rational {
public:
  	Rational(const T& numerator = 0,
    				 const T& denominator = 1);
  	const T numerator() const;
  	const T denominator() const;
  	...
}

// 需要隐式转换的接口定义为非成员函数
template<typename T>
const Rational<T> operator* (const Rational<T>& lhs,
                             const Rational<T>& rhs)
{...};

// 使用
Rational<int> oneHalf(1, 2);
Rational<int> result = oneHalf * 2;  // 这里会编译错误，2不能隐式转换
```
上面只是把24条款示例改为模版实现，然而模版版本是编译不过的，因为编译器并不知道2要转换为什么。编译器推断不了模版的隐式转换。

对于模版我们只能通过friend和inline特性来实现非成员函数的定义。
```cpp
template<typename T>
class Rational {
public:
  	...
    // 这里Rational是Rational<T>的简写形式，在类模版内部可以简写。
    friend const Rational operator*(const Rational& lhs,
                                   	const Rational& rhs)
    {
      	return Rational(lhs.numerator() * rhs.numerator(),
                       	lhs.denominator() * rhs.denominator());
    }
}
```
这样就可以编译，连接通过了。

**总结：**
- 当我们编写一个class template，而它所提供的函数要支持隐式转换时，请将这些函数定义为class template内部的friend函数。

### 47 请使用traits class表现类型信息

1. 设计并实现一个traits class：
    1. 确认若干你希望将来可取得的类型相关信息。例如对于迭代器，我们希望将来可取得其分类。
    2. 为该信息选择一个名称（例如iterator_category）
    3. 提供一个template和一组特化版本（例如稍早说的iterator_traits)，内含你希望支持的类型相关信息。
2. 使用一个traits class：
    1. 建立一组重载函数（身份像劳工）或函数模板，彼此间的差异只在于各自控制的traits参数。令每个函数的实现码与其接受的traits信息相应和。
    2. 建立一个控制函数（身份像工头）或函数模板，它调用上述那些“劳工函数”并传递traits class的相关信息。
    3. Traits class使得“类型相关信息”在编译期可用。它以template和“template 特化”完成实现。
    4. 整合重载技术后，traits classes有可能在编译期对类型执行if…else测试。

**总结：**
- Traits class 使得类型相关信息在编译器可用。它们以template和template特化完成实现。
- 整合重载技术后，traits class有可能在编译期对类型执行if…else测试。(重载是编译期确定，if是运行期确定)

### 48 认识template元编程
47条款的示例就是使用的模版元编程技术，它是一种把运行期的代码转移到编译期完成的技术。这种技术可能永远不会成为主流，但是如果你是一个程序库开发员，那这种技术就是家常便饭了。

通过模版或重载技术，把如if这种运行期的判断转换为编译期重载函数自动匹配。

它有两个特点：
1. 它让某些事情更容易。如果没有它，那些事情将是困难的，甚至不可能的。
2. 由于它将工作从运行期转移到编译期。这可更早发现错误，而且更高效、较小的可执行文件、较短的运行期、较少的内存需求。不过它会使编译时间变长。

**总结：**
- 模版元编程可将工作由运行期转移到编译期，因而得以实现早期错误发现和更高的执行效率。
- 模版元编程可被用来生成客户定制代码，也可用来避免生成对某些特殊类型并不适合的代码。

## 第八章 定制new和delete

### 49 了解new-handler的行为

当new无法申请到新的内存的时候，会不断的调用new-handler，直到找到足够的内存,new_handler是一个错误处理函数：
```cpp
    namespace std{
        typedef void(*new_handler)();
        new_handler set_new_handler(new_handler p) throw();
    }
```
一个设计良好的new-handler要做下面的事情：
+ 让更多内存可以被使用
+ 安装另一个new-handler，如果目前这个new-handler无法取得更多可用内存，或许他知道另外哪个new-handler有这个能力，然后用那个new-handler替换自己
+ 卸除new-handler
+ 抛出bad_alloc的异常
+ 不返回，调用abort或者exit

new-handler无法给每个class进行定制，但是可以重写new运算符，设计出自己的new-handler
此时这个new应该类似于下面的实现方式：
```cpp
    void* Widget::operator new(std::size_t size) throw(std::bad_alloc){
        NewHandlerHolder h(std::set_new_handler(currentHandler));      // 安装Widget的new-handler
        return ::operator new(size);                                   //分配内存或者抛出异常，恢复global new-handler
    }
```

**总结：**
- set_new_handler允许客户指定一个函数，在内存分配无法获得满足时被调用。
- 让new不抛异常是一个颇为局限的工具，因为它只是保证了内存分配时不抛异常，后续调用构造函数还是可能抛出异常。=> new做了两件事：1. 分配内存 2. 调用类的构造函数。

### 50 了解new和delete的合理替换时机

+ 用来检测运用上的错误，如果new的内存delete的时候失败掉了就会导致内存泄漏，定制的时候可以进行检测和定位对应的失败位置
+ 为了强化效率（传统的new是为了适应各种不同需求而制作的，所以效率上就很中庸）
+ 可以收集使用上的统计数据
+ 为了增加分配和归还内存的速度
+ 为了降低缺省内存管理器带来的空间额外开销
+ 为了弥补缺省分配器中的非最佳对齐位
+ 为了将相关对象成簇集中起来

但是要自定义一个合适的new/delete并非易事，如内存对齐(对齐指令执行效率最高)，可移植性、线程安全…等等细节。所以我的建议是在你确定要自定义new/delete之前，请先确定你程序瓶颈是否真的由默认new/delete引起，而且现在也有商业产品可以替代编译器自带的内存管理器。或者也有一些开源的产品可以使用，如Boost的Pool就是对于常见的分配大量小型对象很有帮助。

**总结：**
- 有许多理由需要写个自定的new和delete，包括改善性能，对heap运用错误进行调试，收集heap使用信息。

### 51 编写 new和 delete 时需固守常规

+ 重写new的时候要保证49条的情况，要能够处理0bytes内存申请等所有意外情况
+ 重写delete的时候，要保证删除null指针永远是安全的

**总结：**
operator new 1. 应该内含一个无限循环，并在其中尝试分配内存，如果它无法满足内存需求，就该调用new-handler。2. 它也应该有能力处理0字节申请。3. Class的专属版本则还应该处理“比正确大小更大的申请”(被继承后, new 派生对象，这时可以走编译器默认new操作)。
operator delete应该在收到null指针时不做任何事情。Class专属版本还应该处理“比正确大小更大的申请”(同上)。

### 52 写了placement new也要写placement delete

如果operator new接受的参数除了一定会有的size_t之外还有其他的参数，这个就是所谓的palcement new
```cpp
void* operator new(std::size_t, void* pMemory) throw(); //placement new
static void operator delete(void* pMemory) throw();     //palcement delete，此时要注意名称遮掩问题
```

**总结：**
+ 当你写一个operator new, 请确定也写出了对应的operator delete。如果没有这样做，你的程序可能会发生隐晦而时断时续的内存泄漏。
+ 当你声明new和delete，请确定不要无意识地(非故意)遮掩了它们的正常版本。


## 第九章 杂项讨论
### 53 不要轻忽编译器的警告

**总结：**
+ 严肃对待编译器发出的warning， 努力在编译器最高警告级别下无warning
+ 同时不要过度依赖编译器的警告，因为不同的编译器对待事情的态度可能并不相同，换一个编译器警告信息可能就没有了

### 54. 让自己熟悉包括TR1在内的标准程序库

TR1是C++标准程序库第一次扩充，包含14个新组件，统统都放在std::tr1命名空间下。

1. 智能指针tr1::shared_ptr和tr1::weak_ptr。
2. tr1::function，可表示任何函数，是一个模板。在条款35中有使用。
3. tr1::bind，同样35示范中有它用法。
4. hash table，用来实现set, multiset, map和multi-map容器的hash版本。
5. 正则表达式。
6. tr1::tuple，标准库中的pair template的新一代制品，可持有任意个数的对象(pair只能持有两个对象)。
7. tr1::array，是一个STL化的数组。
8. tr1::mem_fn，生成指向成员的指针的包装对象。
9. tr1::reference_wrapper, 一个让引用的行为更像对象的设施。
10. 随机数生成工具。
11. 数学特殊函数。
12. C99兼容扩充。
13. Type traits，见条款47。
14. tr1::result_of，这是一个模板，用来推导函数调用的返回类型。

**总结：**
- C++标准程序库的主要功能由STL、iostreams、locales组成。并包含C99标准程序库。
- TR1添加了智能指针、一般化函数指针、hash-based容器、正则表达式以及另外10个组件的支持。
- TR1自身只是一份规范。为获得TR1提供的好处，你需要一份实现。一个好的实现来源是Boost。

### 55 让自己熟悉Boost

你正在寻找一个高质量、源码开放、平台独立、编译器独立的程序库吗？看看Boost吧。有兴趣加入一个由雄心勃勃充满才干的C++开发人员组成的社群，致力发展当前最高技术水平的程序库吗？看看Boost吧！想要一瞥未来的C++可能长相吗？看看Boost吧！

![Boost官网](https://www.boost.org/)
**总结：**
- Boost是一个社群，也是一个网站。致力于免费、源码开放、同僚复审的C++程序库开发。Boost在C++标准化过程中扮演深具影响力的角色。
- Boost提供了许多TR1组件的实现，以及其他许多程序库。