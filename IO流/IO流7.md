### 打印流

打印流是高级流，只有输出流。分为字节打印流（`PrintStream`）和字符打印流（`PrintWriter`）。它有3个特点：

1.打印流只操作文件目的地，不操作数据源

2.其特有的方法可以实现数据原样写出。例如打印97，写到文件中的就是97，而不是其对应的ASCII码a

3.其特有的写出方法可以实现数据的自动刷新与自动换行。所以打印流打印一次数据相当于写出+换行+刷新。



#### 字节打印流

| 构造方法                                                     | 说明                         |
| ------------------------------------------------------------ | ---------------------------- |
| public PrintStream(OutputStream/File/String)                 | 关联字节输出流/文件/文件路径 |
| public PrintStream(String fileName, Charset charset)         | 指定字符编码                 |
| public PrintStream(OutputStream out, boolean autoFlush)      | 自动刷新                     |
| public PrintStream(OutputStream out, boolean autoFlush, String encoding) | 指定字符编码且自动刷新       |

由于字节打印流没有缓冲区，所以对于它来说开不开自动刷新都一样。写构造函数参数之前，可以先Ctrl+P看看可以写什么参数。

| 成员方法                                          | 说明                                       |
| ------------------------------------------------- | ------------------------------------------ |
| public void write(int b)                          | 常规方法：规则与之前一样，将指定的字节写出 |
| public void println(Xxx xx)                       | 特有方法：打印任意数据，自动刷新，自动换行 |
| public void print(Xxx xx)                         | 特有方法：打印任意数据，不换行             |
| public void printf(String format, Object... args) | 特有方法：带有占位符的打印语句，不换行     |

这三种特有方法可以实现数据的原样写出。



```java
public class iodemo {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        PrintStream ps = new PrintStream(new FileOutputStream("io\\a.txt"),true,Charset.forName("UTF-8"));
        ps.println(97);//写出+自动刷新+自动换行
        ps.print(true);
        ps.printf("吾妻%s","牧之原翔子")
        ps.close();

    }
}
```



#### 字符打印流

字符流底层有缓冲区，想要自动刷新需要手动开启。构造方法成员方法和字节打印流一模一样，不再赘述。



实际上，我们经常使用打印流。我们最常用的`System.out.println`就用到了打印流。Ctrl+ B可以看到，System是一个类，而out是类的一个`PrintStream`类型的静态变量。所以`System.out`相当于获取了一个打印流，它被称作是系统中的标准输出流，该流是不能关闭的，因为它在系统中是唯一的，除非我们重启JVM。这个打印流是Java启动后由虚拟机自己创建的，不需要我们手动创建。这个打印流默认指向控制台。

所以，对于`System.out.println("123")`，我们可以这么写：

```java
PrintStream ps = System.out;
ps.println("123");//写出数据，自动换行，自动刷新
```



#### 解压缩流/压缩流

解压缩流主要负责读取压缩包中的文件，因此属于输入流。压缩流是将文件中的数据写到压缩包中，因此属于输出流。

需要说明的是，压缩包里的每一个文件，在Java中都是一个`ZipEntry`对象，解压的本质就是把每一个`ZipEntry`按照层级拷贝到本地另一个文件夹中。

```java
public class iodemo {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        //1.创建一个File表示要解压的压缩包
        File src = new File("C:\\test.zip");
        //2.创建一个File表示解压后的目录（目的地）
        File dest = new File("C:\\test");
        //3.解压
        unzip(src,dest);
    }
    //定义一个方法用来解压压缩包
    public static void unzip(File src,File dest) throws IOException {
        //1.创建一个解压流对象
        ZipInputStream zis = new ZipInputStream(new FileInputStream(src));

        //2.获取到压缩包中的每一个ZipEntry对象

        ZipEntry entry;
        while ((entry=zis.getNextEntry())!=null){ //每一个文件、文件夹及其子文件夹都能获取到
            //3.判断当前的ZipEntry对象是否是目录
            if (!entry.isDirectory()){ //没有isFile方法，但有isDirectory方法
                //4.如果是目录，就创建一个目录
                new File(dest,entry.getName()).mkdirs();//解压的本质就是拷贝
            } else{
                //5.如果不是目录，就读取文件数据，并创建一个输出流对象，用来写出文件
                FileOutputStream fos = new FileOutputStream(new File(dest,entry.getName()));
                int b;
                while((b=zis.read())!=-1){
                    fos.write(b);
                }
                //6.关闭输出流
                fos.close();
                //7.关闭ZipEntry对象，表示在压缩包中的一个文件处理完毕了
                zis.closeEntry();
            }
        }
        //8.关闭解压流
        zis.close();
    }
}

```



压缩同理，就是把每一个文件/文件夹看成`ZipEntry`对象放到压缩包中。

将`a.txt`打包成一个压缩包：

```java
public class iodemo {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        //1.创建File对象表示要压缩的文件
        File src = new File("io\\a.txt");
        //2.创建File对象表示压缩包的位置
        File dest = new File("io\\");
        //3.压缩
        zip(src,dest);
    }

     //定义一个方法用来压缩文件
    public static void zip(File src,File dest) throws IOException {
        //1.创建一个压缩流对象关联压缩包
        ZipOutputStream zos = new ZipOutputStream(new FileOutputStream(new File(dest,"a.zip")));
        //2.创建一个文件输入流对象
        FileInputStream fis = new FileInputStream(src);
        //3.创建一个ZipEntry对象，表示压缩包中的每一个文件或文件夹
        ZipEntry entry = new ZipEntry(src.getName());
        //4.将ZipEntry对象添加到压缩流中
        zos.putNextEntry(entry);
        //5.读取文件数据，并写出到压缩包中
        int b;
        while((b=fis.read())!=-1){
            zos.write(b);
        }
        //6.关闭流
        zos.closeEntry();
        zos.close();
        fis.close();
    }
}
```



#### Commons-io工具包

使用步骤：

1.在项目中创建一个文件夹lib，用于存放第三方的jar包（第三方组织将写好的代码打包成一个压缩包，在Java中压缩包的后缀为jar）

2.将jar包复制粘贴到lib文件夹

3.右键点击jar包，选择Add as Library

4.在类中导包使用

其常见方法

| FileUtils类（文件/文件夹相关）                               | 说明                       |
| ------------------------------------------------------------ | -------------------------- |
| static void copyFile(File srcFile, File destFile)            | 复制文件                   |
| static void copyDirectory(File srcDir, File destDir)         | 复制文件夹                 |
| static void copyDirectoryToDirectory(File srcDir, File destDir) | 复制文件夹                 |
| static void deleteDirectory(File directory)                  | 删除文件夹                 |
| static void cleanDirectory(File directory)                   | 清空文件夹                 |
| static String readFileToString(File file, Charset encoding)  | 读取文件中的数据变成字符串 |
| static void write(File file, CharSequence data, String encoding) | 写出数据                   |



| IOUtils类                                                    | 说明       |
| ------------------------------------------------------------ | ---------- |
| public static int copy(InputStream input, OutputStream output) | 复制文件   |
| public static int copyLarge(Reader input, Writer output)     | 复制大文件 |
| public static String readLines(Reader input)                 | 读取数据   |
| public static void write(String data, OutputStream output)   | 写出数据   |



#### Hutool工具包

| 相关类            | 说明                          |
| ----------------- | ----------------------------- |
| IoUtil            | 流操作工具类                  |
| FileUtil          | 文件读写和操作的工具类        |
| FileTypeUtil      | 文件类型判断工具类            |
| WatchMonitor      | 目录、文件监听                |
| ClassPathResource | 针对ClassPath中资源的访问封装 |
| FileReader        | 封装文件读取（注意导包）      |
| FileWriter        | 封装文件写入（注意导包）      |

官网：https://hutool.cn/

API文档：https://apidoc.gitee.com/dromara/hutool/

中文使用文档：https://hutool.cn/docs/#/





