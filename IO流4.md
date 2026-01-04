#### 字符流与字节流的使用场景

字节流：拷贝任意类型的文件

字符流：对纯文本文件读/写数据



练习一：拷贝文件夹，考虑子文件夹

练习二：文件加密。加密原理：对原始文件中每一个字节数据进行更改，然后将更改后的数据存到新的文件中。解密原理：读取加密之后的文件，按照加密的规则反向操作，变成原始文件。

练习三：修改文本文件中的数据。将文本文件中的“2-1-9-4-7-8”排序，即修改为“1-2-4-7-8-9”



练习一：

我在C盘下新建了一个空文件夹`dest`，用于存放拷贝过来的文件。源文件夹是C盘下的`goproject`文件夹，里面有3个go项目，其中一个是chatroom，里面有很多子文件夹与文件。

我们先创建两个File对象，分别表示数据源与目的地。然后调用`copydir`方法。

在`copydir`方法中，分四步走：

1.进入数据源

2.遍历数组

3.如果是文件，拷贝

4.如果是文件夹，递归

```java
public class iodemo {
    public static void main(String[] args) throws IOException {
        File src = new File("C:\\goproject");
        File dst = new File("C:\\dest");
        copyDir(src,dst);
    }

    private static void copyDir(File src, File dst) throws IOException {
        dst.mkdirs();//因为dst可能不存在，所以先创建。就算已经存在了，也只是返回false而已
        File[] files = src.listFiles();
        for (File file : files) {
            if (file.isDirectory()) {
                copyDir(file,new File(dst,file.getName()));
            } else {
                //拷贝，使用字节流。从文件开始，到文件结束。
                FileInputStream fis = new FileInputStream(file);
                FileOutputStream fos = new FileOutputStream(new File(dst,file.getName()));
                byte[] bytes = new byte[1024];
                int len = 0;
                while((len = fis.read(bytes)) != -1) {
                    fos.write(bytes,0,len);
                }
                fos.close();
                fis.close();
            }
        }

    }
}
```

这里默认了`src`是不需要权限即可访问的。如果需要权限，则`listFiles()`返回null，那么就需要加一个非空判断。

对于`copyDir(file,new File(dst,file.getName()));`：我们知道`copyDir`的第一个参数`src`表示数据源，即它表示“拷贝谁”，第二个参数表示“拷贝到哪里”。现在file是文件夹，那么我们就要拷贝file，即解决了“拷贝谁”地问题，那么就确定了第一个参数是file. 我们要将file拷贝到哪里？假设我们要将a文件夹拷贝到b文件夹，那么对于a的子文件夹aa，拷贝完成后，在b文件夹下也会有一个子文件夹aa. 所以file应该拷贝到目标文件夹`dst`下 与file同名的子文件夹，而现在`dst`下没有这个同名的文件夹，所以我们要把它new出来。new File的第一个参数为File类型的parent，第二个参数为String类型的child，顾名思义。所以第一个参数应该是这个同名文件夹的父文件夹，即`dst`，它的名字应该是和file同名的，因此调用`getName`方法。

对于`FileOutputStream fos = new FileOutputStream(new File(dst,file.getName()));`：我们要将数据通过`fis`从源文件中读出来，通过`fos`写到目标文件中，即“从文件开始，到文件结束”。因此我们要给`fos`绑定目标文件，即解决“将数据写给谁”的问题。假设我们要将a文件夹拷贝到b文件夹，那么对于a中的文件`aa.txt`，拷贝完成后，在b文件夹下也会有一个文件`aa.txt`. 所以应该将file的数据 拷贝到目标文件夹`dst`下 与file同名的文件，但现在这个文件不存在，因为我们要new一个。构造函数的参数为什么是这两个，和上面的原因一样。



练习二：文件加密。加密原理：对原始文件中每一个字节数据进行更改，然后将更改后的数据存到新的文件中。解密原理：读取加密之后的文件，按照加密的规则反向操作，变成原始文件。

```java
public class iodemo {
    public static void main(String[] args) throws IOException {
        File Src = new File("a.jpg");
        File SrcAfterEncryption = new File("b.jpg");
        encryption(Src,SrcAfterEncryption);
        File SrcAfterDecryption = new File("c.jpg");
        decryption(SrcAfterEncryption,SrcAfterDecryption);
    }
    
    private static void encryption(File src, File dst) throws IOException {
        FileInputStream fis = new FileInputStream(src);
        FileOutputStream fos = new FileOutputStream(dst);
        int b;
        while((b=fis.read()) != -1){
            fos.write(b^128);
        }
        fos.close();
        fis.close();
    }
    
    private static void decryption(File src, File dst) throws IOException {
        FileInputStream fis = new FileInputStream(src);
        FileOutputStream fos = new FileOutputStream(dst);
        int b;
        while((b=fis.read()) != -1){
            fos.write(b^128);
        }
        fos.close();
        fis.close();
    }
}
```

需要说明的是：如果一个数先后与同一个数进行异或运算，那么这个数保持不变。即a^b^b=a，这是因为异或具有结合律。(a^b)^b = a^(b^b)，而自身与自身进行异或，结果为0，又因为任何数与0进行异或，结果都是它本身，所以这个结论得证。

对于加密与解密，我们可以将其视为拷贝，只不过在拷贝的过程中动了一点手脚。对图片a进行加密，就是将它的每个字节与128进行异或，改变它，得到不可打开的图片b. 想要解密时，就将图片b“拷贝”到图片c，只不过拷贝的过程中，将每个字节都与128进行异或，相当于图片a的每个字节先后与128进行异或，回到它本身，实现了解密。



练习三：修改文本文件中的数据。将文本文件中的“2-1-9-4-7-8”排序，即修改为“1-2-4-7-8-9”

```java
public class iodemo {
    public static void main(String[] args) throws IOException {
        FileReader fr = new FileReader("io/a.txt");
        StringBuilder sb = new StringBuilder();
        int ch;
        while((ch=fr.read()) != -1){
            sb.append((char)ch);
        }
        //System.out.println(sb);调试
        fr.close();

        String[] StrArr = sb.toString().split("-");//得到的2 1 9 4 7 8都是String类型的，并不是int类型的
        ArrayList<Integer> arr = new ArrayList<>();
        for (String s : StrArr) {
            arr.add(Integer.parseInt(s));
        }
        //System.out.println(arr);调试
        Collections.sort(arr);

        FileWriter fw = new FileWriter("io/a.txt");
        for (int i = 0; i < arr.size(); i++) {
            if(i!=arr.size()-1){
                fw.write(arr.get(i)+"-");
            } else{
                fw.write(arr.get(i)+"");//不能漏掉双引号，它表示原样写出，因为write方法的参数如果是整数，写入的是该整数对应的ASCII码字符
            }
        }
        fw.close();
    }
}
```

对于排序和写入，还可以这么写：

```java
public class iodemo {
    public static void main(String[] args) throws IOException {
        FileReader fr = new FileReader("io/a.txt");
        StringBuilder sb = new StringBuilder();
        int ch;
        while((ch=fr.read()) != -1){
            sb.append((char)ch);
        }
        //System.out.println(sb);调试
        fr.close();

        Integer[] array = Arrays.stream(sb.toString()
                        .split("-"))
                .map(Integer::parseInt)
                .sorted()
                .toArray(Integer[]::new);

        FileWriter fw = new FileWriter("io/a.txt");
        fw.write(Arrays.toString(array)
                .replace(", ","-")//注意逗号后还有一个空格
                .substring(1,array.length-1));

        fw.close();
    }
}
```

只写入了"1-2-"，这是因为`array.length`表示的并不是”长度“，而是数组的元素个数，所以其值为6.这也是好理解的，因为数组的元素是Integer类的，所以自然不会算上分隔符以及头尾的中括号。

但调用完replace之后，字符串变为[1-2-4-7-8-9]，字符串长度为13（因为要算上分隔符与头尾的中括号）。而`array.length-1`为5，所以最多只能囊括下标为4的元素。改正如下：

```java
public class iodemo {
    public static void main(String[] args) throws IOException {
        FileReader fr = new FileReader("io/a.txt");
        StringBuilder sb = new StringBuilder();
        int ch;
        while((ch=fr.read()) != -1){
            sb.append((char)ch);
        }
        //System.out.println(sb);调试
        fr.close();

        Integer[] array = Arrays.stream(sb.toString()
                        .split("-"))
                .map(Integer::parseInt)
                .sorted()
                .toArray(Integer[]::new);

        FileWriter fw = new FileWriter("io/a.txt");
        String s = Arrays.toString(array)
                        .replace(", ","-");
        fw.write(s.substring(1,s.length()-1));

        fw.close();
    }
}
```

