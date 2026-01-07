#### 转换流

转换流属于字符流，它也是一种高级流，用来包装基本流。其中转换输入流为`InputStreamReader`，转换输出流为`OutputStreamWriter`，为什么这么命名呢？

转换流是字符流与字节流的桥梁。我们以读取数据为例。读取数据，先需要一个数据源，然后将数据源中的数据读取到内存中。在创建转换输入流对象时，要包装一个字节输入流。包装完后，这个字节流获得了字符流的特性，例如读取数据不会乱码了、可以根据字符集一次读取多个字节。这也是其名字由来。前面的`InputStream`表示它能将字节流转换为字符流，后面的Reader表示它是字符流的语言，继承了Reader

写数据同理，只不过转换输出流将字符流转换为字节流。因为在数据的目的地，也就是文件中，它真实的存储形式就是一个又一个的字节。

因此，如果我们在字节流中想要用字符流的方法，或者想要指定字符集读写数据（JDK11后已淘汰），就可以使用转换流。



```java
public class iodemo {
    public static void main(String[] args) throws IOException {
        //1.创建对象并按照指定字符编码
        InputStreamReader isr = new InputStreamReader(new FileInputStream("io/gbkfile.txt"),"GBK");
        
        //2.读取数据
        int ch;
        while((ch = isr.read()) != -1) {
            System.out.println((char)ch);
        }
        
        //3.关闭流
        isr.close();
    }
}
```

JDK11以后的替代方案如下：

```java
public class iodemo {
    public static void main(String[] args) throws IOException {
        
        //1.创建对象并按照指定字符编码
        FileReader fr = new FileReader("io/gbkfile.txt", Charset.forName("GBK"));

        //2.读取数据
        int ch;
        while((ch = fr.read()) != -1) {
            System.out.println((char)ch);
        }

        //3.关闭流
        fr.close();
    }
}
```



写出数据类似

```java
public class iodemo {
    public static void main(String[] args) throws IOException {
        OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream("io/a.txt"), "GBK");
        osw.write("你好，哥们");
        osw.close();
    }
}
```

直接在IDEA里面打开`a.txt`会是乱码，因为IDEA默认是UTF-8，我们可以在本地打开。

JDK11后的替代方案：

```java
public class iodemo {
    public static void main(String[] args) throws IOException {
        FileWriter fw = new FileWriter("io/gbkfile.txt", Charset.forName("GBK"));
        fw.write("你好，哥们");
        fw.close();
    }
}
```



需求：将本地文件的GBK文件，转成UTF-8

JDK11以前的方案：

```java
public class iodemo {
    public static void main(String[] args) throws IOException {
        InputStreamReader isr = new InputStreamReader(new FileInputStream("io/a.txt"), Charset.forName("GBK"));
        OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream("io/b.txt"), Charset.forName("UTF-8"));
        int ch;
        while((ch = isr.read()) != -1) {
            osw.write(ch);
        }
        osw.close();
        isr.close();
    }
}
```

JDK11后的替代方案：

```java
public class iodemo {
    public static void main(String[] args) throws IOException {
        FileReader fr = new FileReader("io/a.txt", Charset.forName("GBK"));
        FileWriter fw = new FileWriter("io/b.txt", Charset.forName("UTF-8"));
        int ch;
        while((ch = fr.read()) != -1) {
            fw.write(ch);
        }
        fw.close();
        fr.close();
    }
}
```



练习：利用字节流读取文件中的数据，每次读一整行，且不能出现乱码。

由于字节流在读中文的时候会出现乱码，并且字节流中没有读一整行的方法，因此要考虑先将字节流转换为字符流，再将字符流包装成字符缓冲流。

```java
public class iodemo {
    public static void main(String[] args) throws IOException {
        FileInputStream fis = new FileInputStream("io/a.txt");
        InputStreamReader isr = new InputStreamReader(fis);
        BufferedReader br = new BufferedReader(isr);

        String line;
        while((line = br.readLine()) != null) {
            System.out.println(line);
        }
        br.close();
        isr.close();
        fis.close();
    }
}
```

当然也可以简化一下：

```java
public class iodemo {
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream("io/a.txt")));

        String line;
        while((line = br.readLine()) != null) {
            System.out.println(line);
        }
        br.close();
    }
}
```



#### 序列化流

序列化流与反序列化流都属于字节流。序列化流（`ObjectOutputStream`）负责输出数据，反序列化流（`ObjectInputStream`）负责写入数据。

序列化流（又名对象操作输出流）可以将Java中的对象写到本地文件中。

| 构造方法                                    | 说明                 |
| ------------------------------------------- | -------------------- |
| public ObjectOutputStream(OutputStream out) | 把基本流包装成高级流 |



| 成员方法                                  | 说明                 |
| ----------------------------------------- | -------------------- |
| public final void writeObject(Object obj) | 把对象序列化到文件中 |



我们创建一个Student类作为演示。

```java
public class iodemo {
    public static void main(String[] args) throws IOException {
        //1.创建学生对象
        Student stu = new Student("张三",18);

        //2.创建序列化流对象
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("io\\a.txt"));

        //3.写出数据
        oos.writeObject(stu);

        //4.释放资源
        oos.close();
    }
}
```

运行后，发现报了一个`NotSerializableException`的异常。这是序列化流的细节：使用序列化流将对象保存到文件时，会出现`NotSerializableException`异常。解决方法是：让JavaBean类实现Serializable接口。

我们发现Serializable接口中没有抽象方法，因此我们在Student类中也不需要实现。这种没有抽象方法的接口被称作标记接口。在这里，实现Serializable接口相当于标记Student类是可以被序列化的。相当于一个物品的合格证。

```java
public class Student implements Serializable {
    private String name;
    private int age;

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

 

#### 反序列化流

反序列化流（又名对象操作输入流）可以把序列化到本地文件中的对象读取到程序中。

| 构造方法                                 | 说明               |
| ---------------------------------------- | ------------------ |
| public ObjectInputStream(InputStream in) | 把基本流变成包装流 |

| 成员方法                   | 说明                                   |
| -------------------------- | -------------------------------------- |
| public Object readObject() | 把序列化到本地文件中的对象读取到程序中 |

注意这个方法的返回值是`Object`类型的，如果要使用，需要做一次强转。

```java
public class iodemo {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        //1.创建反序列化流对象
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("io\\a.txt"));

        //2.读取数据
        Object o = ois.readObject();

        //3.打印对象
        System.out.println(o);

        //4.释放资源
        ois.close();

    }
}
```

可以看到，输出的是地址值。这是因为JavaBean类没有重写`toString`方法。重写之后再运行一次，发现又报错了，报错信息：

local class incompatible: stream classdesc serialVersionUID = -6802632944716881899, local class serialVersionUID = -175568608660738196

这是因为我们重写`toString`方法，是对Student类的修改。而`serialVersionUID`是Java序列化的类版本标识。JVM靠它判断反序列化的字节流是否和当前类匹配。当Student类实现Serializable接口时，JVM会根据它的成员变量、静态变量、构造方法、成员方法等内容自动生成一个long类型的`serialVersionUID`，可以理解为版本号。序列化时也会把这个版本号写到文件中。后续但凡对序列化后的类进行了一点修改，JVM也会重新为它生成一个`serialVersionUID`，反序列化时发现两个不一致，就报错了。

解决方法：

第一步：手动定义版本号。首先private是必须的，因为我们不希望版本号变更，所以在private的同时也不会为它提供get和set方法。static表示共享，表示这个类所有的对象都共享这个版本号。final表示最终，即版本号不会再发生变化。long是数据类型。并且只能命名为`serialVersionUID`，否则JVM不认它。

```java
public class Student implements Serializable {
    private static final long serialVersionUID = 1L;
    
    private String name;
    private int age;

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

便捷方法：点击文件 -> 设置 -> 输入Serializable -> 勾选 不带 `serialVersionUID`的可序列化类 -> 勾选 transient字段在反序列化时未被初始化

然后我们删掉定义版本号那一行，就会发现在Student类名字处有警告：Student未定义`serialVersionUID`字段。选中Student，Alt+回车，就可以选择添加该字段了。

然后我们重新将对象序列化到本地文件，再反序列化即可看到正常输出。固定版本号的好处在于，序列化之后你可以对类做任何修改，都不会因此导致反序列化出现版本号不匹配的问题。

但需要注意的是，不要一上来就生成版本号，因为这个版本号是根据类的内容计算出来的。



如果我们不想把一些变量序列化到本地文件，我们可以为它加上transient关键字（瞬态关键字），它的作用就是不会把当前属性序列化到本地文件中。

```java
private transient String address;
```



如果序列化了多个对象，由于个数不确定，该如何进行反序列化？

反序列化的时候，如果读不到了，就会报错：`EOFException`

解决方法是：在序列化时，先把对象放进一个`ArrayList`中，再直接序列化这个`ArrayList`. `ArrayList`实现了Serializable接口，也有自己的版本号。

