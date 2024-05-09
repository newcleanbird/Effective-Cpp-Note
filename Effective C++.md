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

总结：
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

总结：
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

总结：
+ 派生类内的名称会遮掩base classes 内的名称。
+ 可以通过using 或者转交函数来解决。

### 34 区分接口继承和实现继承

pure virtual 函数式提供了一个接口继承，当一个函数式pure virtual的时候，意味着所有的实现都在子类里面实现。不过pure virtual也是可以有实现的，调用他的实现的方法是在调用前加上基类的名称：
```cpp
    class Shape{
        virtual void draw() const = 0;
    }
    ps->Shape::draw();
```
总结：
+ 接口继承和实现继承不同，在public继承下，derived classes总是继承base的接口
+ pure virtual函数只具体指定接口继承
+ 简朴的（非纯）impure virtual函数具体指定接口继承以及缺省实现继承
+ non-virtual函数具体指定接口继承以及强制性的实现继承