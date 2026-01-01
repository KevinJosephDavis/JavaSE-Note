IO流用于读写文件中的数据，或者网络中的数据。谁在读？谁在写？是程序在进行读写。或者说内存进行读写，因为程序是运行在内存中的。



#### IO流的分类

根据流的方向，可以分为输入流、输出流

根据操作文件类型，可以分为字节流、字符流。字节流可以操作所有类型的文件，字符流只能操作纯文本文件。用Windows自带的记事本打开能读懂的，就是纯文本文件。例如.txt文件，.md文件，.xml文件，.lrc文件等



#### IO流体系

字节流中有`InputStream`（字节输入流）和`OutputStream`（字节输出流）

字符流中有Reader（字符输入流）和Writer（字符输出流）

但是上述这四个类都是抽象类，不能创建对象，要看它们的子类。例如`FileInputStream`



#### `FileOutputStream`的基本用法

需求：写出一段文字到本地文件中

实现步骤：创建对象、写出数据、释放资源

```java
public class iodemo {
    public static void main(String[] args) throws IOException {
        //1.创建对象
        FileOutputStream fos = new FileOutputStream("io\\a.txt");
        
        //2.写入数据
        fos.write(97);
        
        //3.关闭流
        fos.close();
    }
}
```



流的理解：创建流对象，相当于在程序与文件之间建立了一条管道。close相当于切断管道

几个需要注意的点：

创建对象：

1.参数是字符串或者File对象都可以

2.如果文件不存在，会创建一个新文件，但要保证父级路径存在

3.如果文件已经存在，则会清空文件

写数据：

write方法的参数是整数，但实际上写道本地文件中的是整数在ASCII上对应的字符。也就是说，写入97，实际上在`a.txt`中显示的是字符a

如果真的想要输入97，那么分别写入57和55即可，因为9的ASCII码是57，7是55，这也是为什么需要字符流的原因



##### `FileOutputStream`写数据的三种方式

```java
void write(int b); //一次写一个字节数据
void write(byte[] b); //一次写一个字节数据数据
void write(byte[] b, int off, int len); //一次写一个字节数组的部分数据
```

第一个方法上面做过，看第二个方法：

```java
        FileOutputStream fos = new FileOutputStream("io\\a.txt");
        byte[] bytes = {97,98,99,100,101};
        fos.write(bytes);
        
        fos.close();
```

可以看到`a.txt`中的内容为`abcde`

第三个方法：参数一为数组，参数二为起始索引（可以理解为偏移量offset），参数三表示写入的长度

```java
        FileOutputStream fos = new FileOutputStream("io\\a.txt");
        byte[] bytes = {97,98,99,100,101};
        fos.write(bytes,1,2);

        fos.close();
```

可以看到`a.txt`中的内容为`bc`



##### `FileOutputStream`写数据的两个问题：换行与续写

```java
    public static void main(String[] args) throws IOException {

        FileOutputStream fos = new FileOutputStream("io\\a.txt");
        String s = "java";
        fos.write(s.getBytes());

        fos.close();
    }
```

可以看到`a.txt`中的内容为`java`，现在我们想要在文件中换行写一个`golang`，如何换行呢？

在Windows中，换行符为`\r\n`

在Linux中，换行符为`\n`

在Mac中，换行符为`\r`

在Windows中，java对回车换行进行了优化，虽然完整的是`\r\n`，但是我们写其中任意一个`\r`或`\n`都可以，不过建议都写上

```java
    public static void main(String[] args) throws IOException {

        FileOutputStream fos = new FileOutputStream("io\\a.txt");
        String s = "java";
        String wrap = "\r\n";
        fos.write(s.getBytes());
        fos.write(wrap.getBytes());
        String s1 = "golang";
        fos.write(s1.getBytes());

        fos.close();
    }
```



我们看一下`FileOutputStream`的构造函数源码：

```java
   public FileOutputStream(String name) throws FileNotFoundException {
        this(name != null ? new File(name) : null, false);
    }
```

可以看到，还有第二个参数，是`boolean`类型的

再往下划可以看到：

```java
    public FileOutputStream(String name, boolean append)
        throws FileNotFoundException
    {
        this(name != null ? new File(name) : null, append);
    }
```

我们看一下说明：

Creates a file output stream to write to the file with the specified name. **If the second argument is true, then bytes will be written to the end of the file rather than the beginning**. A new FileDescriptor object is created to represent this file connection.

也就是说，如果第二个参数是true，那么将会在内容的末尾写入数据而不是在新的开始处写数据。即第二个参数为true表示开启续写；false则表示关闭续写，此时创建对象会清空文件

```java
    public static void main(String[] args) throws IOException {

        FileOutputStream fos = new FileOutputStream("io\\a.txt",true);
        String s = "java";
        String wrap = "\r\n";
        fos.write(s.getBytes());
        fos.write(wrap.getBytes());
        String s1 = "golang";
        fos.write(s1.getBytes());

        fos.close();
    }
```

`a.txt`的内容变为：

```txt
java
golangjava
golang
```



#### `FileInputStream`的基本用法

现在我们将`a.txt`的内容改为`abcde`

```java
    public static void main(String[] args) throws IOException {
        FileInputStream fis = new FileInputStream("io\\a.txt");
        System.out.println(fis.read());
        fis.close();
    }
```

控制台上输出的内容为97，也就是a

几个需要注意的点：

创建字节输入流对象：

如果文件不存在，直接报错。

如果是输出流，数据是在程序中的。但如果是输入流，数据在文件中，如果文件不存在，就算和输出流一样创建文件，也没有意义，读不到内容。

写数据：

1.一次读一个字节，读出来的是数据在ASCII上对应的数字

2.读到文件末尾了，read方法返回-1



如何读到后面的内容并看到字母本身？那就循环读取+类型强转：

```java
    public static void main(String[] args) throws IOException {
        FileInputStream fis = new FileInputStream("io\\a.txt");
        int b;//记录读取到的数据
        while((b = fis.read()) != -1) {
            System.out.println((char)b);
        }
        fis.close();
    }
```

如果想要打在同一行，把ln删掉即可，即使用`System.out.print`

能否不定义这个变量b呢？不行，因为在while里面调用了一次read，指针就往后移动了一次，在`sout`处又调用了一次，指针就又往后移动了一次。也就是说最终打印出来的结果只有98、100、-1



#### 文件拷贝

拷贝的核心思想：边读边写

```java
    public static void main(String[] args) throws IOException {
        FileInputStream fis = new FileInputStream("C:\\Users\\31604\\Desktop\\Personal\\JavaSE\\IO流1.md");
        FileOutputStream fos = new FileOutputStream("io\\copy.md");
        
        int b;
        while((b = fis.read()) != -1) {
            fos.write(b);
        }
        fos.close();
        fis.close(); //先开的最后关
    }
```

如果想统计文件复制的时间，有两种方法：

方法一：使用`System.currentTimeMillis()`

```java
    public static void main(String[] args) throws IOException {
        FileInputStream fis = new FileInputStream("C:\\Users\\31604\\Desktop\\Personal\\JavaSE\\IO流1.md");
        FileOutputStream fos = new FileOutputStream("io\\copy.md");
        
        long startTime = System.currentTimeMillis();

        int b;
        while((b = fis.read()) != -1) {
            fos.write(b);
        }
        fos.close();
        fis.close();
        long endTime = System.currentTimeMillis();
        System.out.println("复制耗时：" + (endTime - startTime) + "ms");
    }
```

方法二：使用`Instant+Duration`

```java
    public static void main(String[] args) throws IOException {
        FileInputStream fis = new FileInputStream("C:\\Users\\31604\\Desktop\\Personal\\JavaSE\\IO流1.md");
        FileOutputStream fos = new FileOutputStream("io\\copy.md");

        Instant startTime = Instant.now();

        int b;
        while((b = fis.read()) != -1) {
            fos.write(b);
        }
        fos.close();
        fis.close();

        Instant endTime = Instant.now();
        Duration dr = Duration.between(startTime,endTime);
        System.out.println("复制耗时：" + dr.toMillis() + "ms");
        System.out.println("耗时：" + dr.toSeconds() + " 秒");   // 转秒
        System.out.println("耗时：" + dr.toNanos() + " 纳秒");   // 转纳秒

    }
```



不难发现，上面的例子中`FileInputStream`一次只读写一个字节，速度太慢了。通过重载read方法可以实现一次读取一个字节数组数据：

```java
public int read(byte[] buffer)
```

每次读取会尽可能把数组装满。虽然说数组越大拷贝越快，但是数组本身也占用内存。我们一般使用1024整数倍，例如`1024*1024*5`（即5兆）

```java
    public static void main(String[] args) throws IOException {
        FileInputStream fis = new FileInputStream("io\\a.txt");
        byte[] bytes = new byte[2];

        int len1 = fis.read(bytes);
        System.out.println(len1);
        System.out.println(new String(bytes));

        int len2 = fis.read(bytes);
        System.out.println(len2);
        System.out.println(new String(bytes));

        int len3 = fis.read(bytes);
        System.out.println(len3);
        System.out.println(new String(bytes));

        int len4 = fis.read(bytes);
        System.out.println(len4);
        System.out.println(new String(bytes));

        fis.close();

    }
```

a.txt的内容是：abcde

输出结果如下：

2
ab
2
cd
1
ed
-1
ed

第三次和第四次读取到的内容不是e，而是ed，这是因为bytes存放在内存中，第二次读完cd后，第三次只读到了e，那么e就只会覆盖c不会覆盖d，因此结果就是ed

修改：只将bytes的前len个元素转换为字符串并输出

```java
    public static void main(String[] args) throws IOException {
        FileInputStream fis = new FileInputStream("io\\a.txt");
        byte[] bytes = new byte[2];

        int len1 = fis.read(bytes);
        System.out.println(len1);
        System.out.println(new String(bytes));

        int len2 = fis.read(bytes);
        System.out.println(len2);
        System.out.println(new String(bytes));

        int len3 = fis.read(bytes);
        System.out.println(len3);
        System.out.println(new String(bytes,0,len3));

        fis.close();


    }
```

当我们循环读取的时候，也要指定读取长度，避免读到数组没有存内容的部分：

```java
while((len = fis.read(bytes)) != -1) {
    fos.write(bytes,0,len);
}
```



  