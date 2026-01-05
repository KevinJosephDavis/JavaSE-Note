#### 缓冲流

有四种缓冲流：`BufferedInputStream`字节缓冲输入流，`BufferedOutputStream`字节缓冲输出流，`BufferedReader`字符缓冲输入流，`BufferedWriter`字符缓冲输出流。增加缓冲区，可以提高效率。不过对于字符流，它已经存在缓冲区，所以对效率的提升并不是很明显。



#### 字节缓冲流

| 方法名称                                    | 说明                                     |
| ------------------------------------------- | ---------------------------------------- |
| public BufferedInputStream(InputStream is)  | 把基本流包装成高级流，提高读取数据的性能 |
| public BufferedOutputStream(OutpuStream os) | 把基本流包装成高级流，提高读取数据的性能 |

底层自带长度为8192的缓冲区提高性能。

关于”包装“，实际上缓冲流是不能读写数据的，我们只不过是将它与读写流关联起来，真正干活的还是读写流。只不过有了缓冲流的加持，它的读写效率更高。

```java
public class iodemo {
    public static void main(String[] args) throws IOException {
        BufferedInputStream bis = new BufferedInputStream(new FileInputStream("io/a.txt"));
        BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("copy.txt"));
        int b;
        while((b=bis.read()) != -1) {
            bos.write(b);
        }
        bis.close();
        bos.close();
    }
}
```

在构造函数中，我们可以手动写入第二个参数size，即我们能够手动指定缓冲区的大小。

为什么只需要释放两个缓冲流的资源，不需要释放`FileInputStream`和`FileOutputStream`的资源？我们选中close并看它的源码就能看到，它会判断刷新缓冲区时是否出现了异常，如果没有异常会直接关闭基本流。如果有异常，也会使用try-catch先关闭基本流，再做其它处理。

也可以一次读写多个字节：

```java
public class iodemo {
    public static void main(String[] args) throws IOException {
        BufferedInputStream bis = new BufferedInputStream(new FileInputStream("io/a.txt"));
        BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("copy.txt"));
        byte[] bytes = new byte[1024];
        int len;
        while((len = bis.read(bytes)) != -1) {
            bos.write(bytes,0,len);
        }
        bis.close();
        bos.close();
    }
}
```



#### 字节缓冲流的读写原理

创建字节缓冲输入流与字节缓冲输出流对象后，硬盘（数据源）与内存之间就建立了两条通路，并且在内存中建立了两个缓冲区，一个是字节缓冲输入流的，另一个是字节缓冲输出流的。也就是说，它们并不是共用同一个缓冲区的。

当我们调用`b=bis.read()`，`bis`的缓冲区的数据就写到变量b，`bos.write(b)`则是将b承接的数据交给`bos`的缓冲区。可以理解成变量b就是两个缓冲区的桥梁。当`bis`的缓冲区空了，则从文件中读。当`bos`的缓冲区满了，则写给文件。

我们回顾一下字节输入输出流，发现它也是一个字节一个字节传递的（或者以一个字节数组为单位传递），那么为什么说字节缓冲流提高了效率呢？这是因为b这个桥梁与两个缓冲区都是在内存中的，读写速度比硬盘与内存之间的读写速度要快。当然这需要统一桥梁类型。使用字节输入输出流+字节数组，与使用字节缓冲流+单个字节读写的速度是不确定的，可能前者快也可能是后者。

如果使用的是byte数组，那么就相当于桥梁由变量b变成了数组byte，一次传导多个数据。



#### 字符缓冲流

虽然字符流本身已经带有长度同样为8192的缓冲区，但是它有两个比较好用的方法。

| 构造方法                        | 说明               |
| ------------------------------- | ------------------ |
| public BufferedReader(Reader r) | 把基本流变成高级流 |
| public BufferedWriter(Writer r) | 把基本流变成高级流 |

| 字符缓冲输入流特有方法   | 说明                                       |
| ------------------------ | ------------------------------------------ |
| public String readLine() | 读取一行数据，如果没有数据可读了，返回null |

需要注意的是，该方法遇到回车换行就会结束，但它并不会把回车换行读到内存中。

| 字符缓冲输出流特有方法 | 说明         |
| ---------------------- | ------------ |
| public void newLine()  | 跨平台的换行 |

前面说过，Windows\Linux\Mac的换行符是不一样的，这个跨平台的换行可以将它们统一起来。

如果想开启续写功能，需要注意不能把true写在`BufferedWriter`的构造处，应该写在基本流的构造处。因为是基本流在干活

正确写法：

```java
 BufferedWriter bw = new BufferedWriter(new FileWriter("a.txt",true));
```

错误写法：

```java
 BufferedWriter bw = new BufferedWriter(new FileWriter("a.txt"),true);
```



另外，”缓冲流自带8KB缓冲区“的说法不严谨。对于字节数组来说，缓冲区确实是8KB，因为是byte类型的。但对于字符数组，它是char类型的，其缓冲区能容纳8192个字符，而1个字符占2个字节，所以字符缓冲流底层的缓冲区大小是16KB. 准确的说法是：缓冲流自带长度为8192的缓冲区



练习：实现一个验证程序运行次数的小程序，要求如下：

1.当程序运行超过3次时给出提示：本软件只能免费使用3次，欢迎您注册会员后继续使用

2.程序运行演示如下：

对于i（0<i<4，i为整数）：第i次运行控制台输出：欢迎使用本软件，第i次使用免费

第四次及之后运行控制台输出：本软件只能免费使用3次，欢迎您注册会员后继续使用



显然不能使用计数器，即定义一个`int`类型的变量count用来记录。因为当程序结束之后count也被销毁，程序再次运行时count又被初始化了。所以我们不应该将运行的次数保存在内存中，而应该保存在本地文件中。

如果数字比较大，使用`FileReader`只能一个一个读，我们可以使用字符缓冲输入流，因为它有`readLine`方法，一次可以读一整行。

```java
public class iodemo {
    public static void main(String[] args) throws IOException {
       //1.把文件中的数字读取到内存中
        BufferedReader br = new BufferedReader(new FileReader("io/a.txt"));
        String line = br.readLine();
        int count = Integer.parseInt(line);//得到运行次数
        count++;//当前又运行了一次

        //2.判断
        //>=3 正常运行
        //<3 不运行
        if(count >= 3) {
            System.out.println("本软件只能免费使用3次，欢迎注册会员后继续使用");
        } else {
            System.out.println("欢迎使用本软件，第"+count+"次使用免费");
        }

        //3.把运行次数写回文件中
        BufferedWriter bw = new BufferedWriter(new FileWriter("io/a.txt"));
        bw.write(count+"");//如果写的是数字，写的就是该数字对应的字符，因此要写成字符串
        bw.close();
    }
}
```

需要注意，IO流要随用随创建，即什么时候用，什么时候再创建，什么时候不用，就什么时候关闭。因为如果我们在创建完`br`之后立刻创建`bw`，那么读到的line就是null，因为`bw`的创建会将文件清空。

























