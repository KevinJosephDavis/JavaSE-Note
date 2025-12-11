#### File的三个构造方法

```java
public File(String pathname); //根据文件路径创建文件对象
public File(String parent, String child); //根据父路径名字符串与子路径名字符串创建文件对象
public File(File parent, String child); //根据父路径对应文件对象与子路径名字符串创建文件对象
```



例：

```java
File f1 = new File("C:\\Users\\31604\\Desktop\\Personal\\exp.docx");

String parent = "C:\\Users\\31604\\Desktop\\Personal";
String child = "exp.docx";
File f2 = new File(parent,child);

//当然也可以：
File f2 = new File(parent + "\\" + child);
//这里之所以使用两个\，是因为\在java中是转义字符
//不建议手打反斜杠，因为在Linux下是/ 但在Windows下是\

```



#### File的成员方法

##### 一、判断

```java
public boolean isDirectory(); //判断是否为文件夹
public boolean isFile(); //判断是否为文件
public boolean exists(); //判断是否存在

//例：
File f1 = new File("C:\\Users\\31604\\Desktop\\Personal\\exp.docx");
System.out.println(f1.isDirectory());
System.out.println(f1.isFile());
System.out.println(f1.exists());

//结果为：
//false
//true
//true
```



##### 二、获取

```java
public long length(); //返回文件的大小（字节数）
public String getAbsolutePath(); //返回文件绝对路径
public String getPath(); //返回定义文件时使用的路径
public String getName(); //返回文件的名称，带后缀
public long lastModified(); //返回文件上一次的修改时间（毫秒）

//例：
File f = new File("C:\\Users\\31604\\Desktop\\Personal\\exp.docx");

System.out.println(f.length());
System.out.println(f.getAbsolutePath());
System.out.println(f.getPath());
System.out.println(f.getName());
System.out.println(f.lastModified());

//输出结果：
//17449
//C:\Users\31604\Desktop\Personal\exp.docx
//C:\Users\31604\Desktop\Personal\exp.docx
//exp.docx
//1757432390796

```



如果想单纯获取文件不带后缀的名称或者获取文件的后缀，有两个方法：

方法一：直接使用String的`endsWith`方法

方法二：使用String中的split方法

```java
File f = new File("C:\\Users\\31604\\Desktop\\Personal\\exp.docx");

System.out.println(f.getName().split("\\.")[0]);
System.out.println(f.getName().split("\\.")[1]);

//输出结果：
//exp
//docx
```

分隔符号是`\\.`而不是单个点的原因是：split的参数是正则表达式，在正则表达式中，单个点表示匹配任意字符。如果使用单个点，那么split就会把每个字符当作分隔符。从e开始，到x，再到p，再到d、o、c、x，发现没有能切割的地方，最终的结果是空数组。下标访问必然导致数组越界。

那么，为什么是两条杠呢？首先，在正则中，要表示单个点，而不是表示匹配任意字符，应该使用`"\."`，但在java中`\`是转义字符，因此要用两条反斜杠表示真正的`\`而不是转义字符。

当然，也可以使用`Pattern.quote()`自动转义特殊字符，这样更直观：

```java
System.out.println(f.getName().split(Pattern.quote("."))[0]);
System.out.println(f.getName().split(Pattern.quote("."))[1]);
```



还有一个问题：如何将毫秒转化为我们习惯的年月日格式？

```java
System.out.println(new SimpleDateFormat("yyyy年MM月dd日 HH:mm:ss").format(f.lastModified()));
```

`SimpleDateFormat`的格式模板是通过特定字母来表示时间单位的，常用的有：

| 字母 | 含义             | 示例                        |
| ---- | ---------------- | --------------------------- |
| y    | 年（4位）        | yyyy -> 2025                |
| M    | 月（2位）        | MM -> 06 ; M -> 6（不补零） |
| d    | 日（2位）        | dd -> 05 ; d -> 5           |
| H    | 小时（24小时制） | HH -> 09 ; H -> 9           |
| h    | 小时（12小时制） | hh -> 09 ; h -> 9           |
| m    | 分钟             | mm -> 03 ; m -> 3           |
| s    | 秒               | ss -> 02 ; s -> 2           |

如果想带上下午，就加个a：

```java
// 输出：2025/12/10 08:05:30 下午
System.out.println(new SimpleDateFormat("yyyy/MM/dd hh:mm:ss a").format(f.lastModified()));
```



`SimpleDateFormat`是线程不安全的，可以使用`DateTimeFormatter`：

```java
        DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy年MM月dd日 HH:mm:ss").withZone(ZoneId.systemDefault());
        System.out.println(dtf.format(Instant.ofEpochMilli(f.lastModified())));

        //等价于
        Instant instant = Instant.ofEpochMilli(f.lastModified());
        System.out.println(dtf.format(instant));

```

`Instant.ofEpochMilli()`的作用是：将毫秒转换为Instant对象。Instant对象是用来表示**UTC时间线上的一个瞬间**的标准类，可以理解为带标准语义的时间戳，是一个日期时间对象。`DateTimeFormatter`的format方法只接收日期时间对象而不是毫秒值。



##### 三、创建

```java
public boolean createNewFile(); //创建一个新的空文件
public boolean mkdir(); //创建单级文件夹
public boolean mkdirs(); //创建多级文件夹
```



示例：

```java
System.out.println(new File("C:\\Users\\31604\\Desktop\\Personal\\exp.docx").createNewFile());
```

我们发现编译器很快给我们反馈了一个错误：未处理的异常：java.io.IOException

我们看一下源码：

```java
    public boolean createNewFile() throws IOException {
        SecurityManager security = System.getSecurityManager();
        if (security != null) security.checkWrite(path);
        if (isInvalid()) {
            throw new IOException("Invalid file path");
        }
        return fs.createFileExclusively(path);
    }
```

可以看到，它抛出了IOException，而它是一个受检异常。所谓受检异常，就是编译器强制要求必须处理的异常（捕获/抛出），因为这类异常是程序运行之前可以预见的异常，例如权限不足、路径不存在、创建失败等。像我们熟悉的`NullPointerException`和`ArrayIndexOutOfBoundsException`都是非受检异常，编译器不强制处理，这属于编程错误。

处理方法一：使用try-catch

```java
        try {
            System.out.println(new File("C:\\Users\\31604\\Desktop\\Personal\\exp.docx").createNewFile());
        } catch(IOException e) {
            System.out.println("文件创建失败" + e.getMessage());
            e.printStackTrace();
        }
```



处理方法二：在方法上声明throws抛出异常

```java
public class test {
    public static void main(String[] args) throws IOException {

        System.out.println(new File("C:\\Users\\31604\\Desktop\\Personal\\exp.docx").createNewFile());
    }
}

```

这里把异常抛给JVM处理。不过这种方法下异常会导致程序崩溃。

实际上这两种方法，可以通过将光标放在报错位置，按下Alt+回车就可以看到java给出的两种解决方法



关于`createNewFile()`的三个点：

1.如果路径表示的文件不存在，则创建成功，返回true，反之创建失败返回false

2.如果父级路径不存在，那么会有IOException

3.`createNewFile()`创建的一定是文件，如果没有后缀名，那么就会创建一个无后缀名文件



`mkdir()`与`mkdirs()`的用法也是完全一致。

关于`mkdir()`的两个点：

1.Windows中路径唯一，如果路径已经存在，会返回false. 比如说，你创建了一个名字为ddd的文件夹，但是在该目录下已经有一个名为ddd的文件，那么就会返回false

2.不能创建多级文件夹

实际上，`mkdirs()`涵盖了`mkdir()`的所有功能，`mkdirs()`既可以创建多级文件夹，也可以创建单级文件夹。



##### 四、删除

```java
public boolean delete(); //删除文件、空文件夹
```

用法和创建类似，需要注意两个点：

1.如果删除的是文件/空文件夹，则直接删除，不走回收站

2.如果尝试删除非空文件夹，则删除失败



##### 五、获取并遍历

```java
public File[] listFiles(); //获取当前路径下的所有内容（File对象）
public static File[] listRoots(); //列出可用的系统文件根
public String[] list(); //获取当前路径下的所有内容（String对象，仅获取名字）
public String[] list(FilenameFilter filter); //利用文件名过滤器获取当前路径下的内容
public File[] listFiles(FilenameFilter filter);
public File[] listFiles(FileFilter filter);
```



示例：

```java
        for(File file : new File("C:\\Users\\31604\\Desktop\\Personal").listFiles()) {
            System.out.println(file);
        }
```

可以在控制台中看见该文件夹下的所有文件/文件夹



六个点：

1.若路径不存在，返回null

2.若路径表示文件，返回null

3.若路径表示的文件夹是空文件夹，返回长度为0的数组

4.若路径表示的文件夹非空文件夹，则其内部的文件与文件夹都会放在数组中返回

5.隐藏的文件与文件夹也会被放在数组中返回

6.若路径表示的文件夹需要权限才能访问，返回null



```java
System.out.println(Arrays.toString(File.listRoots()));
```

File的静态方法`listRoots()`可以返回系统中所有的盘符。



```java
        System.out.println(Arrays.toString(
                new File("C:\\Users\\31604\\Desktop\\Personal").list(
                        new FilenameFilter() {
                            @Override
                            public boolean accept(File dir, String name) {
                                return false;
                            }
                        }
                )
        ));
```

accept的形参，依次表示该文件夹里面每一个文件/文件夹的父级路径、子级路径。如果返回值为true，表示当前路径保留，反之不保留。现在返回false，意味着每个路径都不保留，最终输出空数组。

我们现在只要`.txt`文件：

```java
        System.out.println(Arrays.toString(
                new File("C:\\Users\\31604\\Desktop\\Personal").list(
                        new FilenameFilter() {
                            @Override
                            public boolean accept(File dir, String name) {
                                return new File(dir,name).isFile() && name.endsWith(".txt");
                            }
                        }
                )
        ));
```

当然，也可以使用listFiles方法，先获取每个File对象，再写一个if进行过滤：

```java
        for(File file: new File("C:\\Users\\31604\\Desktop\\Personal").listFiles()) {
            if(file.isFile() && file.getName().endsWith(".txt")) {
                System.out.println(file);
            }
        }
```



另外两个方法的使用也基本一致：

```java
        System.out.println(Arrays.toString(
                new File("C:\\Users\\31604\\Desktop\\Personal").listFiles(new FileFilter() {
                    @Override
                    public boolean accept(File pathname) {
                        return pathname.isFile() && pathname.getName().endsWith(".txt");
                    }
                })
        ));
```

这里的pathname表示文件/文件夹的路径名



#### File的六个综合练习

##### 练习一：在当前模块下的aaa文件夹创建a.txt文件

```java
       //由于还没有aaa文件夹，因此要先创建
        File file = new File("test\\aaa");
        file.mkdirs();
        File src = new File(file,"a.txt");
        System.out.println(src.createNewFile() + " 创建成功");
```





##### 练习二：定义一个方法，其功能是判断某个文件夹下是否有以.avi为后缀名的文件（暂时不考虑子文件夹）

```java
    public static boolean HaveAVI(File file) {
        //获取内容，看后缀
        for(File f : file.listFiles()) {
            if(f.isFile() && f.getName().endsWith(".avi")) {
                return true;
            }
        }
        return false;
    }
```



##### 练习三：在练习二的基础上，考虑子文件夹

```java
    public static void findAVI(File file) {
        //递归辅助函数
        for(File f : file.listFiles()) {
            if(f != null) { //因为可能访问到需要权限的文件夹，所以要加非空判断
                //如果是文件
                if(f.isFile()) {
                    if(f.getName().endsWith(".avi")) {
                        System.out.println(f);
                    }
                } else {
                    //如果是文件夹，就递归
                    findAVI(f);
                }
            }
        }
    }

    public static void findAVI() {
        //获取本地所有的盘符
        for(File f : File.listRoots()) {
            findAVI(f);
        }
    }
```



##### 练习四：删除一个多级文件夹

```java
    public static void delete(File src) {
        //1.先删除文件夹里的所有内容
        for(File f : src.listFiles()) {
            if(f != null) {
                if(f.isFile()) {
                    f.delete();
                } else {
                    delete(f);
                }
            }
        }

        //2.再删除自己
        src.delete();
    }
```





##### 练习五：统计文件夹大小

```java
    public static long getLen(File src) {
        long len = 0;
        for(File f : src.listFiles()) {
            if(f.isFile()) {
                len += f.length();
            } else {
                len += getLen(f);
            }
        }
        return len;
    }
```





##### 练习六：统计一个文件夹中每种文件的个数并打印。打印格式为：

**txt：3**

**doc：4**

**jpg：6**

```java
    public static HashMap<String,Integer> getcnt(File src) {
        //1.定义集合用来统计
        HashMap<String,Integer> hm = new HashMap<>();

        //2.遍历
        for(File f : src.listFiles()) {
            if(f != null) {
                //如果是文件，获取后缀名，统计
                if(f.isFile()) {
                    String[] arr = f.getName().split("\\.");
                    if(arr.length >= 2) {
                        //有可能出现没有后缀名的情况
                        String endName = arr[arr.length - 1];
                        if(hm.containsKey(endName)) {
                            //存在
                            int cnt = hm.get(endName);
                            cnt++;
                            hm.put(endName,cnt);
                        } else {
                            //不存在
                            hm.put(endName,1);
                        }
                    }
                } else {
                    //如果是文件夹，递归
                    HashMap<String,Integer> sonMap = getcnt(f);
                    //将sonMap的结果同步到hm中
                    Set<Map.Entry<String, Integer>> entries = sonMap.entrySet();
                    for (Map.Entry<String, Integer> entry : entries) {
                        String key = entry.getKey();
                        int value = entry.getValue();

                        if(hm.containsKey(key)) {
                            //如果存在了，就直接累加
                            int cnt = hm.get(key);
                            cnt += value;
                            hm.put(key,cnt);
                        } else {
                            //如果不存在，直接放进去
                            hm.put(key,1);
                        }
                    }
                    
                }
            }
        }
        return hm;
    }
```







