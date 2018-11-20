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