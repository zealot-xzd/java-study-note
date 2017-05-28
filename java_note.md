# 异常

1. 异常处理的任务就是将控制权从错误产生的地方转移到能够处理这种情况的错误处理器．
2. 引起异常事件：

        用户输入错误
        设备错误
        物理限制
        代码错误

3. 在java中，若某个方法不能采用正常的途径完成他的任务，就可以通过另外一个路径退出方法，
这种情况下方法不返回任何值，而是抛出一个封装了错误信息的对象．注意，这个方法立即退出，不返回
任何值．调用这个方法的代码也无法继续执行，此时异常处理机制开始搜索能够处理这种异常的异常处理器．

4. 异常分类
- 异常都是派生自Throwable类的一个实例，可以创建自己的异常类

 ```
 Throwable
    Error   系统内部错误和资源耗尽错误,不应该抛出这中类型的对象，出现内部错误，除了通知用户，安全终止，再无能为力
    Exception
        RuntimeException
            - 错误的类型转换
            - 数组越界
            - 访问空指针
        非RuntimeException
            - 试图在文件尾部读取数据
            - 打开不存在的文件
            - 查找Class对象，但是类不存在
```

    **`如果出现RuntimeException异常，那一定是你的问题`**，这是有道理的规则

5. java语言规范将派生自Error类和RuntimeException类的所有异常称为**`未检查异常（unchecked）`**异常，
  所有其他的异常称为**`已检查（checked）`**异常

# 异常抛出
1. 声明已检查异常：

    什么时候需要在方法中throws子句声明异常:
       - 调用一个抛出已检查异常的方法． 
       - 程序运行过程中发现错误，并且利用throw语句抛出一个已检查异常
       - 程序出错，如，a[-1].
       - java虚拟机的运行时库出现的内部错误

    ```
        class a
        {
            public int loadImage(String s) throws FileNotFoundException, EOFException
            {
                ...
            }
        }
    ```

> 不需要声明java的内部错误，即从Error继承的错误．任何程序代码都具有抛出那些异常的潜能，我们对其没有任何控制能力.

> 总之，一个方法必须声明所有可能抛出的`已检查异常`,而`未检查异常`要么不可控制(Error),要么就是应该避免发生(RuntimeException). 
> 如果方法没有声明所有可能的已检查异常，编译器就会报错．

# 如何抛出异常
1. 异常抛出语句
    ```
    throw new EOFException();
    或者
    EOFException e = new EOFException();
    throw e;
    ```

# 创建异常类
1. 定义一个派生于IOException的类，一般包含两个构造函数，一个是默认的，一个带有详细描述信息.
    ```
    class FileFormatException extends IOException
    {
        public FileFormatException(){}
        public FileFormatException(String gripe)
        {
            super(gripe);
        }
    }
    ```
# 捕获异常

1. 编译器严格执行throws说明符，如果调用一个抛出已检查异常的方法，就必须对它
进行处理，或者将它继续进行传递．
2. 怎样处理异常最好？
    ```
    通常，应该捕获那些知道如何处理的异常，而将那些不知道怎么处理的异常继续进行传递．
    如果想传递一个异常，就必须在方法的首部添加一个throws说明符，以便告知调用者这个方法可能会
    抛出异常．
    ```


