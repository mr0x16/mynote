## 错误处理

需要处理的情况:

1. 用户输入错误

2. 设备错误

3. 物理限制

4. 代码错误

   > 例如，方法可能返回了一个错误的答案，或者错误地调用了其他的方法。计算的数组索引不合法， 试图在散列表中查找一个不存在的记录， 或者试图让一个空找执行弹出操作

### 异常分类

异常对象均是派生自` Throwable `类的实例，`Throwable`有两个分支`Error`和`Exception`。

`Error`类描述了Java 运行时系统的内部错误和资源耗尽错误。 应用程序**不应该**抛出这种类型的对象 。

`Exception`类又可分为派生于`RuntimeException`的和其他的，划分规则为：

> 由于程序错误导致的异常属于`RuntimeException`， 而程序本身没有问题，但由于像I/O错误这类问题导致的异常属于其他异常。

`RuntimeException`可以包含以下几种情况：

- 错误的类型转换
- 数组访问越界
- 访问null指针

不派生自`RuntimeException`的异常包括：

- 试图在文件尾部后面读取数据
- 尝试打开一个不存在对的文件
- 视图根据给定的字符串查找Class对象

**如果出现 `RuntimeException` 异常， 那么就一定是你的问题 **

Java 语 言 规 范 将 派 生 于 `Error` 类 或 `RuntimeException` 类的所有异常称为非受查( unchecked ) 异常， 所有其他的异常称为受查（checked) 异常 。编译器将核查是否为所有的**受査**异常提供了异常处理器 。

### 声明受查异常

> 一个方法不仅需要告诉编译器将要返回什么值，**还要告诉编译器有可能发生什么错误** 

方法应该在其首部声明所有可能抛出的异常

```java
public Fi1elnputStream(String name) throws FileNotFoundException
```

在自己编写方法时，不必将所有可能抛出的异常都进行声明。

在以下四种条件下，应该抛出异常：

1. 调用一个抛出受查异常的方法，例如上面的`FileImputStream`构造器
2. 程序运行过程中发生错误，并且利用`throw`语句抛出一个受查异常
3. 程序出现错误， 例如，`a[-l] = 0` 会抛出一个`ArraylndexOutOffloundsException`这样的
   非受查异常
4. Java 虚拟机和运行时库出现的内部错误 

**如果出现前两种情况之一， 则必须告诉调用这个方法的程序员有可能抛出异常 **

> 如果类中的一个方法声明将会抛出一个异常， 而这个异常是某个特定类的实例时，则这个方法就有可能抛出一个这个类的异常， 或者这个类的任意一个子类的异常。 例如，`FilelnputStream` 构造器声明将有可能抛出一个 `IOExcetion` 异常， 然而并不知道具体是哪种 `IOException` 异常。 它既可能是 `IOException` 异常， 也可能是其子类的异常， 例如，`FileNotFoundException` 

### 如何抛出异常

**首先要决定应该抛出什么异常。**

对于已经存在的异常类，按照以下三步将其抛出：

1. 找到一个合适的异常类；
2. 创建这个类的一个对象；
3. 将对象抛出；

### 创建异常类

> 任何标准异常类都没有能够充分地描述清楚的问题 。

需要做的只是定义一个派生于Exception 的类，或者派生于 Exception 子类的类。

```java
class FileFormatExcaption extends IOException {
    public FileFormatExcaption() {}
    public FileFormatExcaption(String gripe) {
        
    }
}
```

现在，可以抛出自己的异常了

```java
String readData(BufferedReader in) throws FileFormatException {
    more code;
    while(condition) {
        if (ch == -1) {
            if(n < len) 
                throw new FileFormatException();
        } //EOF encountered
            
    }
}
```



## 捕获异常

### 捕获异常

**如果某个异常发生的时候没有在任何地方进行捕获，那么程序就会终止执行**

通过try/catch语句块可以捕获异常

```java
try {
    code；
    more code；
    more code;
} catch (ExceptionType e) {
    handler for this type;
}
```

需要注意的是，放不知道对异常应该如何处理时，应该将异常抛出给调用者，让调用者决定如何处理异常。传递一个异常的话，在方法头部添加`throws`说明符就好。

当有**多个**异常需要捕获时可以使用多个catch子句，例如

```java
try {
    code that might throw exceptions;
} catch (FileNotFoundException e) {
    emergency action for missingfiles;
} catch (UnknowHostException e) {
    emergency action for unknown hosts;
} catch (IOException e) {
    emergency action for all other I/O problems;
}
```

要想获取异常对象更对的信息，可以使用`e.getMessage()`

要想得到异常对象的实际类型，可以使用`e.getClass().getName()`

java 7之后，可以在一个catch子句中捕捉多个异常，例如

```java
try {
    code that might throw exceptions;
} catch (FileNotFoundException | UnknowHostException e) {
    emergency action for missing files and unknown hosts;
} catch (IOException e) {
    emergency action for all other I/O problems;
}
```

注意当捕获的异常类型彼此之间**不存在子类关系**时才可以使用这一特性。

> 捕获多个异常时，异常变量隐含为final变量

### 再次抛出异常与异常链

在catch子句中可以抛出一个异常，这样做的目的是改变异常的类型。下面是一个捕获异常并再次抛出的基本用法

```java
try {
    access the database;
} catch (SQLException e) {
    throw now ServletException("database error: " + e.getMessage());
}
```

还有一种更好的处理方法，并且将原始异常设置为新异常的“原因”

```java
try {
    access the database;
} catch (SQLException e) {
    Throwable se = new ServletException("database error");
    se,initCause(e);
    throw se;
}
```

此时，当捕获这个异常时，可以使用`Throwable e = se.getCause()`来重新得到原始异常

这样的方式可以让用户抛出高级异常，而不会丢失原始异常的细节。

### 带资源的try语句

对于这样的代码模式

```java
//open a resource
try {
    //work with the resource
} finally {
   //close the resource
}
```

**这里资源需要是一个实现了AutoCloseable接口的类。**

例如：

```java
try(Scanner in = new Scanner(new FileInputStream("/usr/share/dict/words"), "18519839965")) {
    while(in.hasNext())
        System.out.println(in.next());
}
```

在上面的代码中，try块正常退出或者存在一个异常时，都会调用in.close()方法。

还可指定**多个资源**，例如：

```java
try(Scanner in =  new Scanner(new FileInputStream("/usr/share/dict/words"), "UTF-8"); PrintWriter out = new PrintWriter("out.txt")) {
    while (in.hasNext())
        out.println(in.next().toUpperCase());
}
```

**只要需要关闭资源， 就要尽可能使用带资源的 try语句 **

### 使用异常机制的技巧

#### 1. 异常处理不能代替简单测试

**基本原则：只在异常情况下使用异常机制。**

例如上百万次次的对一个空栈进行弹出操作：

```java
//先检测后弹出
if(!s.empty()) s.pop(); //用时646ms
//弹出后捕获异常
try {
    s.pop();
} catch (EmptyStackException e) {
    //do something
}//用时21739ms
```

#### 2.不要过分的细化异常

**有问题的方式，每一条语句都放在独立的try块中**

这种编程方式将导致代码量急剧膨胀。因此，有必要将**整个任务**包装在一个try块中。例如：

```java
try {
    for(i = 0;i < 100; i++) {
        n = s.pop();
        out.writeInt(n);
    }
} catch (IOException e) {
    //IO错误的处理
} catch (EmptyStackException e) {
    //栈异常的处理
}
```

#### 3. 利用异常层次结构

？

#### 4. 不要压制异常

#### 5. 在检测错误时，“苛刻”要比放任更好

例如， 当栈空时， `Stack.pop()` 是返回一个 `null`, 还是抛出一个异常？ 我们认为：在出错的地方抛出一个 `EmptyStackException`异常要比在后面抛出一个 `NullPointerException` 异常更好。

#### 6. 不要羞于传递异常

其实， 传递异常要比捕获这些异常更好 。让高层次的方法通知用户发生了错误， 或者放弃不成功的命令更加适宜。 

**5、6可以归结为：“早抛出，晚捕获”**

## 使用断言

### 断言的概念

断言机制允许在测试期间向代码中插入一下检查语句。当代码发布时，这些插入的检测语句就会被自动的移走。

Java引入了关键字`assert`。这个关键字有两种形式：

```java
assert 条件；
```

```java
assert 条件：表达式；
```

以上两种形式都会对条件进行检测，如果结果为`false`，则抛出一个`AssertionError`异常。在第二种形式中，表达式将被传入`AssertionError`的构造器，并转换成一个消息字符串。

### 启动和禁用断言

在默认情况下，断言被禁用。可以在运行程序时用 -enableassertions或者-ea选项启用，例如：

`java -enableassertion MyApp`

需要注意的是，在启用和禁用断言时不必重新编译程序。启用和禁用断言是**类加载器**的功能

对于没有类加载器的“系统类“，需要使用-enablesystemassertions/-eas开关启动断言

### 使用断言完成参数检查

**java中有三种处理系统错误的机制**

* 抛出一个异常
* 日志
* 使用断言

**那么，什么时候使用断言呢？**

* *断言是致命的、不可恢复的错误。*
* *断言的检查只用能由于**开发**和**测试**阶段*

## 记录日志

记录日志的API为了解决不停插入、删除println的问题，优点如下：

* 可以很容易地取消全部日志记录，或者仅仅取消某个级别的日志，而且打开和关闭这个操作也很容易。 
* 可以很简单地禁止日志记录的输出， 因此，将这些日志代码留在程序中的开销很小。 
* 日志记录可以被定向到不同的处理器， 用于在控制台中显示， 用于存储在文件中等。

* 日志记录器和处理器都可以对记录进行过滤。过滤器可以根据过滤实现器制定的标准丢弃那些无用的记录项。 
* 日志记录可以采用不同的方式格式化，例如，纯文本或 XML。 
* 应用程序可以使用多个日志记录器， 它们使用类似包名的这种具有层次结构的名字，例如，com.mycompany.myapp 。
* 在默认情况下，日志系统的配置由配置文件控制。 如果需要的话， 应用程序可以替换这个配置。 

### 基本日志

```java
Logger.getClobal 0,info("This is Java");
```

可用此种方法打印java日志

```java
Logger.getClobal ().setLevel (Level.OFF);
```

在适当的地方调用，可以关闭所有日志

### 高级日志

在一个专业的应用程序中，**不要**将所有的日志都记录到一个全局的日志记录器中，需要自定义日志记录器。

```java
private static final Logger myLogger = Logger.getLogger("com.mycompany.myapp")
```

> 未被任何变量引用的日志记录器可能会被垃圾回收。为了防止这种情况，要像以上代码一样，用一个静态变量存储日志记录器的一个引用。

日志记录器名也有层次结构，对于日志记录器来说，父与子之间会共享**某些**属性。比如`com.mycompany`日志记录器设置了日志级别，他的子记录器也会继承这个级别。

通常，有以下7个日志记录器级别：

* SEVERE
* WARNING
* INFO
* CONFIG
* FINE
* FINER
* FINEST

在默认情况下，只记录前三个级别。当需要设置其他级别时，可以如此设置`logger.setLevel(Level.FINE)`

这样可以把FINE和更高级别的日志记录下来。还可以用`Level.ALL`开启所有级别的记录，或者用`Level.OFF`关闭所有级别日志的记录。

对于相应级别的日志记录，有以下两种方式记录，分别是：

```java
logger.warning(message)//打印warning级别的日志
logger.fine(message)//打印fine级别的日志
```

```java
logger.log(Level.FINE, message)//打印fine级别的日志
```

可以调用`logp`方法打印调用类和方法的确切位置，可以使用以下方法

```java
void logp(Level l, String className, String methodName, String message)
```

### 修改日志管理器配置

可以通过编辑配置文件来修改日志系统的各种属性。在默认情况下，配置文件在

`jre/lib/logging.properties`

