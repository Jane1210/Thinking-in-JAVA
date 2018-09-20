## 第十八章 Java I/O 系统

### 一 File类

File类其实指的是 FilePath（文件路径）。

#### 1.目录列表器

要查看目录，可以使用两种方法来使用 File 对象：

- list() 方法，可获得此 File 对象包含的全部列表
- 想要获得受限列表（如扩展名为.java的文件），要用到“目录过滤器”

```java
public class DirList {
    public static void main(String[] args) {
        File path = new File(".");
        String[] list;
        if(args.length == 0)
            list = path.list();	//方法1，全部列表
        else
            list = path.list(new DirFilter(args[0]));//方法2，受限列表
        Arrays.sort(list,String.CASE_INSENSITIVE_ORDER);//按字母排序
        for(String dirItem : list)
            System.out.println(dirItem);
    }
}
//该类为了把accept()方法提供给list()使用，使ist()可以回调accept()，进而决定哪些文件包含在列表中。
//策略模式
class DirFilter implements FilenameFilter {
    private Pattern pattern;
    public DirFilter(String regex) {
        pattern = Pattern.compile(regex);
    }
    //accept()方法必须接受一个代表某个特定文件所在目录的File对象，以及包含了那个文件名的一个String
    //list()方法会为此目录对象下的每个文件名调用accept()，来判断该文件是否包含在内，判断结果由boolean表示
    //accept会使用一个正则表达式的matcher对象，来查看此正则表达式regex是否匹配这个文件的名字。
    public boolean accept(File dir, String name) {
        return pattern.matcher(name).matches();
    }
}
```

上例可以改写为匿名内部类的形式。new FilenameFilter 接口来实现。

#### 2.目录实用工具

local() 方法产生由本地目录中的文件构成的File对象数组；

walk()方法产生给定目录下的由整个目录树中的所有文件构成的List<File>(File对象比文件名更有用，因为它包含更多的信息)。这些文件是基于你提供的正则表达式选中的。

#### 3.目录的检查及创建

File类不仅仅只代表存在的文件或目录，也可以用File对象来创建新的目录或尚不存在的整个目录路径。还可查看文件的特性(大小、最后修改日期...)，检查某个File对象代表的是一个文件还是一个目录，并可以删除文件。

### 二 输入和输出

流：代表任何有能力产出数据的数据源对象或是有能力接收数据的接收端对象。

#### 1.InputStream类型

与输入有关的类都应从InputStream继承。

InputStream的作用是用来表示那些从不同数据源产生输入的类。这些数据源包括：

1）字节数组

2）String对象

3）文件

4）“管道”

5）一个由其他种类的流组成的序列，以便我们可以将它们收集合并到一个流内

6）其他数据源，如Internet连接等

每一种数据源都有相应的InputStream子类。

#### 2.OutPutStream类型

该类别的类决定了输出所要去往的目标：字节数组、文件或管道。

### 三 添加属性和有用的接口

**装饰器模式：**

使用分层对象来动态透明地向单个对象中添加责任。装饰器指定包装在最初的对象周围的所有对象都具有相同的基本接口。

Java I/O类库需要多种不同功能的组合，所以使用装饰器模式。

抽象类filter（过滤器）是所有装饰器类的基类。

FilterInputStream 和 FilterOutputStream 是用来提供装饰器类接口以控制特定输入流（InputStream）和输出流（OutputStream）的两个类，FilterInputStream 和 FilterOutputStream 分别自InputStream 和 OutputStream派生而来。这两个类是装饰器的必要条件。

#### 1.通过FilterInputStream 从 InputStream 读取数据

- DataInputStream：允许我们读取不同的基本数据类型以及String对象（所有方法都以read开头，如readFloat()）.搭配DataOutputStream通过数据“流”把数据从一个地方迁移到另一个地方。
- 其他FilterInputStream类：则在内部修改InputStream的行为方式（是否缓冲等）。

#### 2.通过FilterOutputStream 向OutStream 写入数据

- DataOutputStream：可以将各种基本数据类型以及String对象格式化输出到“流”中（所有方法都以write开头，如writeFloat()）
- PrintStream: 以可视化格式打印所有基本数据类型以及String对象。两个重要方法：print() 和 println()
- BufferedOutputStream:是个修改过的OutputStream，对数据流使用缓冲技术

### 四 Reader 和 Writer

- InputStream 和 OutputStream面向字节（8位）形式的 I/O；
- Reader 和 Writer 兼容 Unicode（16位，它用于字符国际化） 和 面向字符的I/O功能。
- 当需要把“字节”层次结构中的类和“字符”中的结合使用时，要用到“适配器”类。InputStreamReader 可以把 InputStream 转换为 Reader, OutputStreamWriter 可以把OutputStream转换为 Writer。

#### 1.数据的来源和去处

尽量尝试使用 Reader 和 Writer ,一旦无法成功编译，就使用 面向字节的InputStream 和 OutputStream。比如 java.util.zip类库就是面向字节的。

在这两个不同的继承层次结构中，接口是非常类似的，比如 InputStream 对应 Reader，FileInputStream  对应 FileReader, FileOutputStream 对应 FileWriter，诸如此类。

#### 2.更改流的行为

无论何时使用readLine(),都不应该使用 DataInputStream，而应该使用 BufferedReader，除了这一点，DataInputStream 仍是 I/O类库的首选成员。

#### 3.未发生变化的类

有些类在Java1.0和1.1之间未作改变。DataInputStream 、File、RandomAccessFile、SequenceInputStream.

### 五 自我独立的类：RandomAccessFile

该类适用于由大小已知的记录组成的文件，所以可以使用seek()将记录从一处移到另一处，然后读取或者修改记录。

它完全独立，工作方式类似于把DataInputStream 和 DataOutputStream 组合起来用（因为实现了相同的接口：DataInput和DataOutput），还添加了新方法：

getFilePointer()  :  查找当前所处的文件位置

seek()  :  在文件内移至新的位置

length()  : 判断文件的最大尺寸

### 六 I/O流的典型使用方式

#### 1.缓冲输入文件

```java
public class BufferedInputFile {
    public static String read(String filename) throws IOException {
        BufferedReader in = new BufferedReader(new FileReader(filename));
        String s;
        StringBuilder sb = new StringBuilder();
        while ((s = in.readLine()) != null)
            sb.append(s + "\n");
        in.close();
        return sb.toString();
    }

    public static void main(String[] args) throws IOException{
        System.out.println(read(System.getProperty("user.dir") + "\\src\\chapter18\\BufferedInputFile.java"));
        System.out.println(read("E:\\test.txt"));
    }
}
```

#### 2.从内存输入

```java
public class MemoryInput {
  public static void main(String[] args) throws IOException {
    StringReader in = new StringReader(
      BufferedInputFile.read(System.getProperty("user.dir") + "\\src\\chapter18\\MemoryInput.java"));
    int c;
    while((c = in.read()) != -1)//read()每次读取一个字符
      System.out.print((char)c);//read()是以int形式返回下一字节，所以打印时要类型转换
  }
}
```

#### 3.格式化的内存输入

要读取格式化数据，可以使用DataInputStream，所以必须使用InputStream.

例：一次一个字节地读取文件

```java
public class TestEOF {
    public static void main(String[] args)
            throws IOException {
        DataInputStream in = new DataInputStream(
                new BufferedInputStream(
                        new FileInputStream(System.getProperty("user.dir") + "\\src\\chapter18\\BufferedInputFile.java")));
        while (in.available() != 0) //available()检测输入是否结束
            System.out.print((char) in.readByte());
    }
}
```

#### 4.基本的文件输出

```java
public class BasicFileOutput {
    static String file = "BasisFileOutput.txt";//要输出的文件路径

    public static void main(String[] args) throws IOException{
        //从BufferedInputFile.read()读入的String结果被用来创建一个StringReader
        BufferedReader in = new BufferedReader(
                new StringReader(BufferedInputFile.read("E:\\test.txt")));
        //FileWriter对象可以向文件写入数据，BufferedWriter将其包装用以缓冲输出，再装饰成PrintWriter用以格式化输出
        PrintWriter out = new PrintWriter(new BufferedWriter(new FileWriter(file)));
        int lineCount = 1;
        String s;
        while((s = in.readLine()) != null)
            out.println(lineCount++ + ":" + s);
        out.close();
        System.out.println(BufferedInputFile.read(file));
    }
}
```

#### 5.存储和恢复数据

PrintWriter可以对数据进行格式化，但是为了输出可供另一个“流”恢复的数据，需要用DataOutputStream写入数据，并用DataInputStream恢复数据。

#### 6.读写随机访问文件

RandomAccessFile rf = new RandomAccessFile(file, "r");   // r 只读   rw 读写

RandomAccessFile 除了实现 DataInput和DataOutput 接口外，与I/O继承层次结构的其他部分实现了分离。它不支持装饰。

### 七 文件读写的实用工具

```java
public class TextFile extends ArrayList<String> {
    //读取文件并转换成字符串
    public static String read(String fileName) {
        StringBuilder sb = new StringBuilder();
        try {
            BufferedReader in = new BufferedReader(new FileReader(
                    new File(fileName).getAbsoluteFile()));
            try {
                String s;
                while ((s = in.readLine()) != null) {
                    sb.append(s);
                    sb.append("\n");
                }
            } finally {
                in.close();//保证文件正确关闭
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        return sb.toString();
    }
	// 打开文本并将其写入文件
    public static void write(String fileName, String text) {
        try {
            PrintWriter out = new PrintWriter(
                    new File(fileName).getAbsoluteFile());
            try {
                out.print(text);
            } finally {
                out.close();
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
    
    public TextFile(String fileName, String splitter) {
        super(Arrays.asList(read(fileName).split(splitter)));
        if (get(0).equals("")) remove(0);
    }

    // 按行读取
    public TextFile(String fileName) {
        this(fileName, "\n");
    }
    
    public void write(String fileName) {
        try {
            PrintWriter out = new PrintWriter(
                    new File(fileName).getAbsoluteFile());
            try {
                for (String item : this)
                    out.println(item);
            } finally {
                out.close();
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    public static void main(String[] args) {
        String file = read("TextFile.java");
        write("test.txt", file);
        TextFile text = new TextFile("test.txt");
        text.write("test2.txt");
        TreeSet<String> words = new TreeSet<String>(
                new TextFile("TextFile.java", "\\W+"));
        System.out.println(words.headSet("a"));
    }
} 
```

### 八 标准输入

#### 1.从标准输入中读取

```java
BufferedReader stdin = new BufferedReader(new InputStreamReader(System.in));
```

#### 2.将 System.out转换为 PrintWriter

System.out 是一个 PrintStream，而PrintStream是一个OutputStream.

```java
PrintWriter out = new PrintWriter(System.out, true);
```

#### 3.标准I/O重定向

如果我们突然开始在显示器上创建大量输出，而输出滚动太快以至于无法阅读时，需要用到重定向，可以重定向到另一个文件。

I/O重定向操纵的是字节流，因此使用InputStream 和 OutputStream。

```java
public class Redirecting {
    public static void main(String[] args)
            throws IOException {
        PrintStream console = System.out;
        BufferedInputStream in = new BufferedInputStream(
                new FileInputStream("Redirecting.java"));
        PrintStream out = new PrintStream(
                new BufferedOutputStream(
                        new FileOutputStream("test.out")));
        System.setIn(in);
        System.setOut(out);
        System.setErr(out);
        BufferedReader br = new BufferedReader(
                new InputStreamReader(System.in));
        String s;
        while ((s = br.readLine()) != null)
            System.out.println(s);
        out.close();
        System.setOut(console);
    }
}
```

### 九 新 I/O

java.nio.*包引入了新的Java I/O库，目的在于提高速度。速度的提高主要来自于所使用的结构更接近于操作系统执行I/O的方式：**通道和缓冲器**。

唯一直接与通道交互的缓冲器是ByteBuffer，可以存储未加工字节的缓冲器。

旧I/O库中由3个类被修改了，用以产生FileChannel.这三个类是：FileInputStream、FileOutputStream、RandomAccessFile。这些是字节操纵流，与底层的nio性质移至。Reader和Writer 这种字符模式类不能用于产生通道，但是java.nio.channels.Channels类提供了实用方法，用以在通道中产生 Reader 和Writer。

```java
public class GetChannel {
    private static final int BSIZE = 1024;
    // 这里产生的任何流类，getChannel()都会产生一个FileChannel
    // 使用 wrap() 方法将已存在的字节数组“包装到” ByteBuffer 中，再把 ByteBuffer 传送给通道
    public static void main(String[] args) throws Exception {
        //写
        FileChannel fc = new FileOutputStream("data.txt").getChannel();
        fc.write(ByteBuffer.wrap("Some text ".getBytes()));
        fc.close();
        //读写
        fc = new RandomAccessFile("data.txt", "rw").getChannel();
        fc.position(fc.size()); //可以在文件内随意移动FileChannel，这里是移到最后，以便附加其他写操作
        fc.write(ByteBuffer.wrap("Some more".getBytes()));
        fc.close();
        // 读
        fc = new FileInputStream("data.txt").getChannel();
        ByteBuffer buff = ByteBuffer.allocate(BSIZE);//只读访问必须显式使用静态allocate()方法分配ByteBuffer
        fc.read(buff);//一旦调用read()来告知FileChannel向ByteBuffer存储字节，就必须调用缓冲器上的flip()让它做好让别人读取字节的准备；如果有多次read()，还得调用clear()为每个read()做准备
        buff.flip();
        while (buff.hasRemaining())
            System.out.print((char) buff.get());
    }
}
```

nio的目标就是快速移动大量数据，所以方法分配ByteBuffer的大小显得尤其重要。allocateDirect()能达到更快的速度，以产生一个与操作系统由更高耦合性的“直接”缓冲器。但是，这种分配的开支会更大。

