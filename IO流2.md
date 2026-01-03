#### IO流中不同JDK版本捕获异常的方式

try-catch异常处理

```java
        try {
            FileOutputStream fos = new FileOutputStream("a.txt");
            fos.write(97);
            fos.close();
        } catch(IOException e) {
            e.printStackTrace();
        }
```

如果在`fos.write(97)`出现异常，那么下面的close操作就无法被执行，如何解决？

```java
        try {
            FileOutputStream fos = new FileOutputStream("a.txt");
            fos.write(97);
        } catch(IOException e) {
            e.printStackTrace();
        } finally {
            fos.close();
        }
```

finally中的代码一定会被执行，除非JVM停止

但是我们发现它无法解析fos，这是因为fos是局部变量。那么能否将它移动到try-catch-finally语句块的外面？可以，但是不能将整个创建对象的过程都移动到try的外面，因为创建对象的代码有编译时异常。所以我们在try-catch-finally外面声明对象，在try里面创建对象。

```java
        FileOutputStream fos = null;
        try {
            fos = new FileOutputStream("a.txt");
            fos.write(97);
        } catch(IOException e) {
            e.printStackTrace();
        } finally {
            fos.close();
        }
    }
```

注意到close方法可能也会抛出异常，我们在finally中嵌套一个try-catch

```java
        FileOutputStream fos = null;
        try {
            fos = new FileOutputStream("a.txt");
            fos.write(97);
        } catch(IOException e) {
            e.printStackTrace();
        } finally {
            try {
                fos.close();
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        }
    }
```

但是依然有问题。如果创建对象时的路径不存在，那么fos为空，在finally中调用null.close()显然会出现空指针异常。因此我们还要加一个非空判断

```java
        FileOutputStream fos = null;
        try {
            fos = new FileOutputStream("a.txt");
            fos.write(97);
        } catch(IOException e) {
            
            e.printStackTrace();
        } finally {
            if(fos!=null) {
                try {
                    fos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
```



代码麻烦的地方在于释放资源，有更加简便的方法。在JDK7中Java推出了AutoCloseable接口，通过实现这个接口，可以在特定的情况下自动释放资源

基本做法：

```java
try {
    可能出现异常的代码;
} catch(异常类名 变量名) {
    异常的处理代码;
} finally {
    执行所有资源释放操作;
}

手动释放资源
```

JDK7方案：

```java
try(创建流对象1;创建流对象2) {
    可能出现异常的代码;
} catch(异常类名 变量名) {
    异常的处理代码;
}

资源用完最终自动释放
```

注意：只有实现了AutoCloseable的类才能在小括号中创建对象。它表示整个try-catch语句执行完毕后，小括号中的流就会自动释放资源。但是这容易导致小括号内的代码可读性较差。

JDK9方案：

```java
创建流对象1;
创建流对象2;
try(流1；流2) {
    可能出现异常的代码;
} catch(异常类名 变量名) {
    异常的处理代码;
}

资源用完最终自动释放
```



#### 乱码与字符集

前面的字节流读取文件时，我们的文件里都没有中文内容。如果有中文内容会如何？

我们将原来的`a.txt`里的内容改成“轻音少女”，再使用字节流读取文件：

```java
        FileInputStream fis = new FileInputStream("io/a.txt");
        int b;
        while((b=fis.read())!=-1) {
            System.out.println((char)b);
        }
        fis.close();
```

可以看到控制台中输出的是乱码。为什么会出现乱码？

我们先了解字符集

在计算机中，任意数据都是以二进制形式进行存储的。其中一个0/1就叫做一个bit，即一个比特位，8个bit为一个byte，即一个字节。存储英文只需要一个字节。因为ASCII字符集有128个编码，而一个byte可以表示256个数据。

假设我们现在存储字母a，它对应的ASCII码是97，对应的二进制是110 0001，但由于字节是计算机中最小的存储单元，97只有7位，不足8位，所以需要编码，将字符集中查询到的数据按照一定的规则进行计算变成真实的、实际能存储在硬盘中的二进制数据。而ASCII的编码规则为前面补0直到8位，因此会将97转换为所以会以0110 0001的形式存入计算机。如果我们想从硬盘中读取这个数据，就需要解码，解码与编码相反，把实际存储在硬盘中的二进制数据按照一定的规则进行计算变成字符集中对应的数字。因为ASCII是前面补零，对值没有影响，因此直接转成十进制97，再查找ASCII码，找到a

不难发现，如果只有ASCII码，是无法存储中文的，因为没有对应的数字。GB2312应运而生。后续还有BIG5（台湾地区繁体中文标准字符集）。后来又推出了我们熟知的GBK，作为GB2312的扩展版。windows系统默认使用的是GBK，只不过系统显示ANSI，这是因为台湾地区使用BIG5，日本使用Shift-JS，韩国使用EUC-KR，为了照顾用户，windows就把这些字符集起了一个公共的名字ANSI

随着信息化时代的发展，大洋洲、南美洲、非洲的国家也要使用互联网，Unicode字符集应运而生。Uni就是universal的前三个字母，因此它也被称作万国码。Unicode字符集是国际标准字符集，它将世界各种语言的每个字符定义一个唯一的编码，以满足跨语言、跨平台的文本信息转换。

GBK是完全兼容ASCII的，所以存储规则类似。当我们存储字符a，同样是查询GBK，找到其对应的数字97，转成二进制110 0001，然后进行编码。其编码规则与ASCII一样。

如果我们要存储“汉”这个字，也是一样。查询GBK，找到其对应的数字47802，转成二进制10111010 10111010，我们发现这是两个字节。因为汉字数量太多，所以需要16个bit，最多能记录65536个数据，足够了。这也是GBK的规则1：汉字用两个字节存储。其中第一个字节叫做高位字节，第二个叫做低位字节。这涉及到GBK的规则2：高位字节二进制一定以1开头，转成十进制后是一个负数。这是为了和英文区分开，因为GBK兼容ASCII，而ASCII的编码规则是补0。GBK汉字编码规则：不需要变动。直接存储到硬盘中即可。

例如，10111010 10111010 01100001是GBK字符集编码之后的二进制，那么它表示的就是“汉a”，因为GBK中一个英文字母一个字节，二进制第一位是0（ASCII只有127个，最多占7位），中文汉字两个字节，二进制第一位为1

对于Unicode，过程也是类似的。如果我们要存储字母a，那么就查询Unicode，发现其对应的数字为97，再进行编码。但是它的编码规则比较不同。UTF-16编码规则是用2-4个字节保存。UTF就是Unicode Transfer Format的缩写，16就表示在这种编码方法中最常用的就是转成16个bit位，因此a的UTF-16编码就是00000000 01100001. 而UTF-32更加粗暴，它固定使用四个字节保存，不过你是中文还是英文。它们对abcd这种只需要一个字节就能表示的字母来说，存储确实不友好。所以后来出现了UTF-8编码规则，它是我们在实际开发中最为常见的。UTF-8编码规则的内容是：用1-4个字节保存。不同的语言使用不同的字节数保存。ASCII中的字符就使用1个字节，简体中文使用3个字节。不过，如果是3个字节，最高位字节前面要加1110，次高位与最低为字节前面要加10.如果是2个字节，最高位前面补110，最低位前面补10.如果是4个字节，最高位前面补11110，其它字节前面补10.其余的比特位用实际查询到的数字填充即可。

还是以a为例，查询Unicode找到其对应数字为97，二进制形式为110 0001，使用UTF-8编码规则进行编码。它规定一个字节的就在前面补零，所以最终结果就是0110 0001.如果要存储的是“汉”，查询Unicode找到其对应数字为27721，二进制形式为01101100 01001001，使用UTF-8规则进行编码。由于中文使用3个字节存储，最高位字节前面补1110，后面的都补10，因此最终结果为**1110**0110 **10**110001 **10**001001

所以，UTF-8并不是字符集，它是Unicode字符集的一种编码方式。

回到之前的问题，为什么会出现乱码？原因有两个：

1.读取数据时未读完整个汉字。字节流之所以叫字节流，是因为它一次只读一个字节。我们知道UTF-8编码中，中文使用3个字节存储，如果只读最高位字节，将它转换为二进制形式后，在Unicode字符集中可能不存在对应的汉字，或者是其它的汉字。如果不存在，可能就会返回问号，或者方块。

2.编码与解码的方式不统一。以“汉”字为例，查询Unicode得到其对应数字为27721，二进制形式为1101100 01001001，使用UTF-8进行编码，结果为1110110 10110001 10001001.如果使用UTF-8解码，那么就能得到正确结果。但如果使用GBK进行解码，因为GBK中一个汉字使用两个字节并且以1开头，所以它会先读取前两个字节，转成十进制数字59057，得到汉字“姹”，再读最后一个字节，转成十进制数字-119，-119在GBK中没有对应汉字，因此只能返回一个问号（问号被包裹在一个菱形内）。

所以，想要不产生乱码：

1.不要用字节流读取文本文件

2.编码解码时使用同一个码表，同一个编码方式

虽然字节流读取中文会乱码，但拷贝不会乱码。我们还是以a.txt中的内容为“轻音少女”为例，看看原因。

```java
        FileInputStream fis = new FileInputStream("io/a.txt");
        int b;
        while((b=fis.read())!=-1) {
            System.out.println((char)b);
        }
        fis.close();
```

前面我们知道运行这段代码，会输出乱码。因为我们使用的是字节流去读取文件。

拷贝：

```java
        FileInputStream fis = new FileInputStream("io/a.txt");
        FileOutputStream fos = new FileOutputStream("io/copy.txt");

        int b;
        while((b=fis.read())!=-1) {
            fos.write(b);
        }
        fos.close();
        fis.close();
```

打开copy.txt，可以看到使用字节流进行拷贝是完全可以的。

其实理由很简单，拷贝的时候是一个字节一个字节去拷贝的，因此从数据源到目的地，并没有数据丢失。如果记事本使用的字符集与编码表和数据源是一样的，那就不会出现乱码了。



#### Java中的编码与解码

Java中编码的方法：

| String类中的方法                           | 说明                 |
| ------------------------------------------ | -------------------- |
| public byte[] getBytes()                   | 使用默认方式进行编码 |
| public byte[] getBytes(String charsetName) | 使用指定方式进行解码 |

Java中解码的方法：

| String类中的方法                        | 说明                 |
| --------------------------------------- | -------------------- |
| String(byte[] bytes)                    | 使用默认方式进行解码 |
| String(byte[] bytes,String charsetName) | 使用指定方式进行解码 |

需要注意的是，IDEA的默认方式是UTF-8，而eclipse的默认方式是GBK



```java
        String str = "你好";
        byte[] bytes = str.getBytes();
        System.out.println(Arrays.toString(bytes));

        byte[] gbks = str.getBytes("GBK");
        System.out.println(Arrays.toString(gbks));
```

默认方式编码（UTF-8），输出结果为：

[-28, -67, -96, -27, -91, -67]

刚好是6个，前3个表示“你”，后3个表示“好”

GBK编码，输出结果为：

[-60, -29, -70, -61]

刚好是4个，前两个与后两个分别对应“你”和“好”

解码同理：

```java
        String str1 = new String(bytes);
        System.out.println(str1);

        String str2 = new String(gbks,"GBK");
        System.out.println(str2);
```

输出两个“你好”

当然，如果编码与解码对应方式不同，肯定会出现乱码。例如str2使用UTF-8解码，由于它使用GBK编码，所以必然出现乱码。需要注意的是，不一定是所有都是乱码，由于UTF-8与GBK都兼容ASCII，所以ASCII的那128个都是可以正常解码的。











