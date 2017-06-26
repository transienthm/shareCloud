# 摘要
本文大多技术围绕调整磁盘文件 I/O,但是有些内容也同样适合网络 I/O 和窗口输出。
第一部分技术讨论底层的I/O问题，然后讨论诸如压缩，格式化和串行化等高级I/O问题。然而这个讨论没有包含应用设计问题，例如搜索算法和数据结构，也没有讨论系统级的问题，例如文件高速缓冲。
Java语言采取两种截然不同的磁盘文件结构。一个是基于字节流，另一个是字符序列。在Java 语言中一个字符由两个字节表示，而不是像通常的语言如c语言那样是一个字节。因此，从一个文件读取字符时需要进行转换。这个不同在某些情况下是很重要的， 就像下面的几个例子将要展示的那样。

低级I/O相关的问题：
- 缓冲
- 读写文本文件
- 格式化的代价
- 随机访问
高级I/O问题
 - 压缩
 - 高速缓冲
 - 分解
 - 串行化
 - 获取文件信息
更多信息
 - 加速I/O的基本规则
 - 避免访问磁盘
 - 避免访问底层的操作系统
 - 避免方法调用
 - 避免个别的处理字节和字符
很明显这些规则不能在所有的问题上避免，因为如果能够的话就没有实际的I/O被执行。

## 使用缓存减少读写次数开销
使用缓冲加速文件读取的示例：
>　对于一个1 MB的输入文件，以秒为单位的执行时间是:
 FileInputStream的read方法，每次读取一个字节，不用缓冲             6.9秒
 BufferedInputStream的read方法使用BufferedInputStream              0.9秒
 FileInputStream的read方法读取数据到直接缓冲                       0.4秒
或者说在最慢的方法和最快的方法间是17比1的不同。

这个巨大的加速并不能证明你应该总是使用第三种方法，即自己做缓冲。这可能是一个错误的倾向特别是在处理文件结束事件时没有仔细的实现。在可读性上它也没有其它方法好。但是记住时间花费在哪儿了以及在必要的时候如何矫正是很有用。方法2 或许是对于大多应用的 "正确" 方法.
方法 2 和 3 使用了缓冲技术, 大块文件被从磁盘读取，然后每次访问一个字节或字符。缓冲是一个基本而重要的加速I/O 的技术,而且有几个类支持缓冲(BufferedInputStream 用于字节, BufferedReader 用于字符)。
缓冲区越大I/O越快吗？典型的Java缓冲区长1024 或者 2048 字节，一个更大的缓冲区有可能加速 I/O但比重很小，大约5 到10%。
### 方法1: 读方法
第一个方法简单的使用FileInputStream的read方法:

FileInputStream的read方法每次读取文件的下一个字节，触发了大量的底层运行时系统调用
优点：编码简单，适用于小文件
缺点：读写频繁，不适用于大文件
```
import java.io.*;
 public class intro1 {
   public static void main(String args[]) {
     if (args.length != 1) {
       System.err.println("missing filename");
       System.exit(1);
     }
     try {
       FileInputStream fis = new FileInputStream(args[0]); // 建立指向文件的读写流
       int cnt = 0;
       int b;
       while ((b = fis.read()) != -1 ) {  // FileInputStream 的read方法每次 读取文件一个字节
         if (b == '\n')
           cnt++;
       }
       fis.close();
       System.out.println(cnt);
     }
     catch (IOException e) {
       System.err.println(e);
     }
   }
 }
```
### 方法 2: 使用大缓冲区
第二种方法使用大缓冲区避免了上面的问题:

BufferedInputStream的read方法 把文件的字节块读入缓冲区，然后每次读取一个字节，每次填充缓冲只需要访问一次底层存储接口
优点：避免每个字节的底层读取，编码相对不复杂
缺点：缓存占用了小量内存

```
import java.io.*;
 public class intro2 {
   public static void main(String args[]) {
    if (args.length != 1) {
      System.err.println("missing filename");
      System.exit(1);
    }
    try {
      FileInputStream fis =  new FileInputStream(args[0]);
      BufferedInputStream bis = new BufferedInputStream(fis); // 把文件读取流指向缓冲区
      int cnt = 0;
      int b;
      while ((b = bis.read()) != -1 ) {   //BufferedInputStream 的read方法 把文件的字节块独     
                                  //入缓冲区BufferedInputStream，然后每次读取一个字节
        if (b == '\n')
          cnt++;
        }
      bis.close();
      System.out.println(cnt);
    }
    catch (IOException e) {
      System.err.println(e);
    }
  }
 }
```

###　方法 3: 直接缓冲
FileInputStream的read方法直接读入字节块到直接缓冲buf，然后每次读取一个字节。
优点：速度最快，
缺点：编码稍微复杂，可读性差，占用了小量内存，
```
import java.io.*;
  public class intro3 {
    public static void main(String args[]) {
      if (args.length != 1) {
        System.err.println("missing filename");
        System.exit(1);
      }
      try {
        FileInputStream fis = new FileInputStream(args[0]);
        byte buf[] = new byte[2048];
        int cnt = 0;
        int n;
        while ((n = fis.read(buf)) != -1 ) { // FileInputStream 的read方法直接读入字节块到
                                     //直接缓冲buf，然后每次读取一个字节
          for (int i = 0; i < n; i++) {
            if (buf[i] == '\n')
              cnt++;
          }
        }
        fis.close();
        System.out.println(cnt);
      }
      catch (IOException e) {
        System.err.println(e);
      }
    }
  }
```

### 方法4: 缓冲整个文件
缓冲的极端情况是事先决定整个文件的长度，然后读取整个文件。
优点：把文件底层读取降到最少，一次，
缺点：大文件会耗尽内存。
```
import java.io.*;
  public class readfile {
    public static void main(String args[]) {
      if (args.length != 1) {
        System.err.println("missing filename");
        System.exit(1);
      }
      try {
        int len = (int)(new File(args[0]).length());
        FileInputStream fis = new FileInputStream(args[0]);
        byte buf[] = new byte[len];                  //建立直接缓冲
        fis.read(buf);                             // 读取整个文件
        fis.close();
        int cnt = 0;
        for (int i = 0; i < len; i++) {
          if (buf[i] == '\n')
            cnt++;
        }
        System.out.println(cnt);
      }
      catch (IOException e) {
        System.err.println(e);
      }
    }
  }
```
这个方法很方便，在这里文件被当作一个字节数组。但是有一个明显得问题是有可能没有读取一个巨大的文件的足够的内存。
缓冲的另一个方面是向窗口终端的文本输出。缺省情况下， System.out ( 一个PrintStream) 是行缓冲的，这意味着在遇到一个新行符后输出缓冲区被提交。

##　格式化的代价

实际上向文件写数据只是输出代价的一部分。另一个可观的代价是数据格式化。
考虑下面的字符输出程序
性能对比结果为：这些程序产生同样的输出。运行时间是:
>格式化方法
示例语句
运行时间
简单的输出一个固定字符
System.out.print(s);
1.3秒
使用简单格式"+"
String s = 字符+字符，       
System.out.print(s);

1.8秒
使用java.text包中的 MessageFormat 类的对象方法
String s = fmt.format(values);
System.out.print(s);
7.8秒
使用java.text包中的 MessageFormat 类的静态方法

7.8*1.3秒

最慢的和最快的大约是6比1。如果格式没有预编译第三种方法将更慢，使用静态的方法代替:
第三个方法比前两种方法慢很多的事实并不意味着你不应该使用它，而是你要意识到时间上的开销。
在国际化的情况下信息格式化是很重要的，关心这个问题的应用程序通常从一个绑定的资源中读取格式然后使用它。
方法 1，简单的输出一个固定的字符串，了解固有的I/O开销:
public class format1 {
    public static void main(String args[]) {
      final int COUNT = 25000;
      for (int i = 1; i <= COUNT; i++) {
        String s = "The square of 5 is 25\n";
        System.out.print(s);
      }
    }
  }
方法2，使用简单格式"+":
public class format2 {
    public static void main(String args[]) {
      int n = 5;
      final int COUNT = 25000;
      for (int i = 1; i <= COUNT; i++) {
        String s = "The square of " + n + " is " +
            n * n + "\n";
        System.out.print(s);
      }
    }
  }
方法 3，第三种方法使用java.text包中的 MessageFormat 类的对象方法:
import java.text.*;
 public class format3 {
   public static void main(String args[]) {
     MessageFormat fmt =
      new MessageFormat("The square of {0} is {1}\n");
      Object values[] = new Object[2];
    int n = 5;
    values[0] = new Integer(n);
    values[1] = new Integer(n * n);
    final int COUNT = 25000;
    for (int i = 1; i <= COUNT; i++) {
      String s = fmt.format(values);
      System.out.print(s);
     }
    }
  }
方法 4，使用MessageFormat.format(String, Object[]) 类的静态方法
import java.text.*;
  public class format4 {
    public static void main(String args[]) {
      String fmt = "The square of {0} is {1}\n";
      Object values[] = new Object[2];
      int n = 5;
      values[0] = new Integer(n);
      values[1] = new Integer(n * n);
      final int COUNT = 25000;
      for (int i = 1; i <= COUNT; i++) {
        String s =
            MessageFormat.format(fmt, values);
        System.out.print(s);
      }
    }
  }
这比前一个例子多花费1/3的时间。
随机访问的性能开销
RandomAccessFile 是一个进行随机文件I/O(在字节层次上)的类。这个类提供一个seek方法，和 C/C++中的相似,移动文件指针到任意的位置，然后从那个位置字节可以被读取或写入。
seek方法访问底层的运行时系统因此往往是消耗巨大的。一个更好的代替是在RandomAccessFile上建立你自己的缓冲，并实现一个直接的字节read方法。read方法的参数是字节偏移量（>= 0）。这样的一个例子是:
这个程序简单的读取字节序列然后输出它们。
适用的情况：如果有访问位置，这个技术是很有用的，文件中的附近字节几乎在同时被读取。例如，如果你在一个排序的文件上实现二分法查找，这个方法可能很有用。
不适用的情况：如果你在一个巨大的文件上的任意点做随机访问的话就没有太大价值。
import java.io.*;
  public class ReadRandom {
    private static final int DEFAULT_BUFSIZE = 4096;
    private RandomAccessFile raf;
    private byte inbuf[];
    private long startpos = -1;
    private long endpos = -1;
    private int bufsize;
    public ReadRandom(String name)
     throws FileNotFoundException {
      this(name, DEFAULT_BUFSIZE);
    }
    public ReadRandom(String name, int b)
        throws FileNotFoundException {
      raf = new RandomAccessFile(name, "r");
      bufsize = b;
      inbuf = new byte[bufsize];
    }
    public int read(long pos) {
      if (pos < startpos || pos > endpos) {
        long blockstart = (pos / bufsize) * bufsize;
        int n;
        try {
          raf.seek(blockstart);
          n = raf.read(inbuf);
        }
        catch (IOException e) {
          return -1;
        }
        startpos = blockstart;
        endpos = blockstart + n - 1;
        if (pos < startpos || pos > endpos)
          return -1;
      }
      return inbuf[(int)(pos - startpos)] & 0xffff;
    }
    public void close() throws IOException {
      raf.close();
    }
    public static void main(String args[]) {
      if (args.length != 1) {
        System.err.println("missing filename");
        System.exit(1);
      }
      try {
        ReadRandom rr = new ReadRandom(args[0]);
        long pos = 0;
        int c;
        byte buf[] = new byte[1];
        while ((c = rr.read(pos)) != -1) {
          pos++;
          buf[0] = (byte)c;
          System.out.write(buf, 0, 1);
        }
        rr.close();
      }
      catch (IOException e) {
        System.err.println(e);
      }
    }
  }
压缩的性能开销
Java提供用于压缩和解压字节流的类，这些类包含在java.util.zip 包里面，这些类也作为 Jar 文件的服务基础 ( Jar 文件是带有附加文件列表的 Zip 文件)。
压缩的目的是减少存储空间，同时被压缩的文件在IO速度不变的情况下会减少传输时间。
压缩时候要消耗CPU时间，占用内存。
压缩时间=数据量/压缩速度。
IO传输时间=数据容量/ IO速度。
传输数据的总时间＝压缩时间＋I/O传输时间
压缩是提高还是损害I/O性能很大程度依赖你的硬件配置，特别是和处理器和磁盘驱动器的速度相关。使用Zip技术的压缩通常意味着在数据大小上减少50%，但是代价是压缩和解压的时间。一个巨大(5到10 MB)的压缩文本文件，使用带有IDE硬盘驱动器的300-MHz Pentium PC从硬盘上读取可以比不压缩少用大约1/3的时间。
压缩的一个有用的范例是向非常慢的媒介例如软盘写数据。使用高速处理器(300 MHz Pentium)和低速软驱(PC上的普通软驱)的一个测试显示压缩一个巨大的文本文件然后在写入软盘比直接写入软盘快大约50% 。
下面的程序接收一个输入文件并将之写入一个只有一项的压缩的 Zip 文件:
import java.io.*;
 import java.util.zip.*;
  public class compress {
    public static void doit(String filein, String fileout) {
      FileInputStream fis = null;
      FileOutputStream fos = null;
      try {
        fis = new FileInputStream(filein);
        fos = new FileOutputStream(fileout);
        ZipOutputStream zos =  new ZipOutputStream(fos);
        ZipEntry ze = new ZipEntry(filein);
        zos.putNextEntry(ze);
        final int BUFSIZ = 4096;
        byte inbuf[] = new byte[BUFSIZ];
        int n;
        while ((n = fis.read(inbuf)) != -1)
          zos.write(inbuf, 0, n);
        fis.close();
        fis = null;
        zos.close();
        fos = null;
      }
      catch (IOException e) {
        System.err.println(e);
      }
      finally {
        try {
          if (fis != null)
            fis.close();
          if (fos != null)
            fos.close();
        }
        catch (IOException e) {
        }
      }
    }
  public static void main(String args[]) {
    if (args.length != 2) {
     System.err.println("missing filenames");
     System.exit(1);
    }
   if (args[0].equals(args[1])) {
     System.err.println("filenames are identical");
     System.exit(1);
      }
      doit(args[0], args[1]);
    }
  }
下一个程序执行相反的过程，将一个假设只有一项的Zip文件作为输入然后将之解压到输出文件:
import java.io.*;
 import java.util.zip.*;
  public class uncompress {
    public static void doit(String filein, String fileout) {
      FileInputStream fis = null;
      FileOutputStream fos = null;
      try {
        fis = new FileInputStream(filein);
        fos = new FileOutputStream(fileout);
        ZipInputStream zis = new ZipInputStream(fis);
        ZipEntry ze = zis.getNextEntry();
        final int BUFSIZ = 4096;
        byte inbuf[] = new byte[BUFSIZ];
        int n;
        while ((n = zis.read(inbuf, 0, BUFSIZ)) != -1)
          fos.write(inbuf, 0, n);
        zis.close();
        fis = null;
        fos.close();
        fos = null;
      }
      catch (IOException e) {
        System.err.println(e);
      }
      finally {
        try {
          if (fis != null)
            fis.close();
          if (fos != null)
            fos.close();
        }
        catch (IOException e) {
        }
      }
    }
    public static void main(String args[]) {
      if (args.length != 2) {
     System.err.println("missing filenames");
     System.exit(1);
      }
    if (args[0].equals(args[1])) {
     System.err.println("filenames are identical");
     System.exit(1);
      }
      doit(args[0], args[1]);
    }
  }

高速缓存
关于硬件的高速缓存的详细讨论超出了本文的讨论范围。但是在有些情况下软件高速缓存能被用于加速I/O。考虑从一个文本文件里面以随机顺序读取一行的情况，这样做的一个方法是读取所有的行，然后把它们存入一个ArrayList (一个类似Vector的集合类):
import java.io.*;
 import java.util.ArrayList;
  public class LineCache {
    private ArrayList list = new ArrayList();
    public LineCache(String fn) throws IOException {
      FileReader fr = new FileReader(fn);
      BufferedReader br = new BufferedReader(fr);
      String ln;
      while ((ln = br.readLine()) != null)
        list.add(ln);
      br.close();
    }
    public String getLine(int n) {
      if (n < 0)
        throw new IllegalArgumentException();
      return (n < list.size() ? (String)list.get(n) : null);
    }
    public static void main(String args[]) {
      if (args.length != 1) {
        System.err.println("missing filename");
        System.exit(1);
      }
      try {
        LineCache lc = new LineCache(args[0]);
        int i = 0;
        String ln;
        while ((ln = lc.getLine(i++)) != null)
          System.out.println(ln);
      }
      catch (IOException e) {
        System.err.println(e);
      }
    }
  }
getLine 方法被用来获取任意行。这个技术是很有用的，但是很明显对一个大文件使用了太多的内存，因此有局限性。一个代替的方法是简单的记住被请求的行最近的100 行，其它的请求直接从磁盘读取。这个安排在局域性的访问时很有用，但是在真正的随机访问时没有太大作用?
分解
分解 是指将字节或字符序列分割为像单词这样的逻辑块的过程。Java 提供StreamTokenizer 类, 像下面这样操作:
import java.io.*;
  public class token1 {
    public static void main(String args[]) {
     if (args.length != 1) {
       System.err.println("missing filename");
       System.exit(1);
      }
      try {
        FileReader fr = new FileReader(args[0]);
        BufferedReader br = new BufferedReader(fr);
        StreamTokenizer st = new StreamTokenizer(br);
        st.resetSyntax();
        st.wordChars('a', 'z');
        int tok;
        while ((tok = st.nextToken()) !=
            StreamTokenizer.TT_EOF) {
          if (tok == StreamTokenizer.TT_WORD)
            ;// st.sval has token
        }
        br.close();
      }
      catch (IOException e) {
        System.err.println(e);
      }
    }
  }
这个例子分解小写单词 (字母a-z)。如果你自己实现同等地功能，它可能像这样：
import java.io.*;
  public class token2 {
    public static void main(String args[]) {
      if (args.length != 1) {
        System.err.println("missing filename");
        System.exit(1);
      }
      try {
        FileReader fr = new FileReader(args[0]);
        BufferedReader br = new BufferedReader(fr);
        int maxlen = 256;
        int currlen = 0;
        char wordbuf[] = new char[maxlen];
        int c;
        do {
          c = br.read();
          if (c >= 'a' && c <= 'z') {
            if (currlen == maxlen) {
              maxlen *= 1.5;
              char xbuf[] =
                  new char[maxlen];
              System.arraycopy(
                  wordbuf, 0,
                  xbuf, 0, currlen);
              wordbuf = xbuf;
            }
            wordbuf[currlen++] = (char)c;
          }
          else if (currlen > 0) {
            String s = new String(wordbuf,
                0, currlen);
          // do something with s
            currlen = 0;
          }
        } while (c != -1);
        br.close();
      }
      catch (IOException e) {
        System.err.println(e);
      }
    }
  }
第二个程序比前一个运行快大约 20%，代价是写一些微妙的底层代码。
StreamTokenizer 是一种混合类，它从字符流(例如 BufferedReader)读取, 但是同时以字节的形式操作，将所有的字符当作双字节(大于 0xff) ，即使它们是字母字符。
串行化
串行化 以标准格式将任意的Java数据结构转换为字节流。例如，下面的程序输出随机整数数组:
  import java.io.*;
  import java.util.*;
  public class serial1 {
    public static void main(String args[]) {
      ArrayList al = new ArrayList();
      Random rn = new Random();
      final int N = 100000;
      for (int i = 1; i <= N; i++)
        al.add(new Integer(rn.nextInt()));
      try {
        FileOutputStream fos =　new FileOutputStream("test.ser");
        BufferedOutputStream bos =  new BufferedOutputStream(fos);
        ObjectOutputStream oos =  new ObjectOutputStream(bos);
        oos.writeObject(al);
        oos.close();
      }
      catch (Throwable e) {
        System.err.println(e);
      }
    }
  }

而下面的程序读回数组:
import java.io.*;
 import java.util.*;
  public class serial2 {
    public static void main(String args[]) {
      ArrayList al = null;
      try {
        FileInputStream fis = new FileInputStream("test.ser");
        BufferedInputStream bis = new BufferedInputStream(fis);
        ObjectInputStream ois = new ObjectInputStream(bis);
        al = (ArrayList)ois.readObject();
        ois.close();
      }
      catch (Throwable e) {
        System.err.println(e);
      }
    }
  }
注意我们使用缓冲提高I/O操作的速度。
有比串行化更快的输出大量数据然后读回的方法吗？可能没有，除非在特殊的情况下。例如，假设你决定将文本输出为64位的整数而不是一组8字节。作为文本的 长整数的最大长度是大约20个字符，或者说二进制表示的2.5倍长。这种格式看起来不会快。然而，在某些情况下，例如位图，一个特殊的格式可能是一个改 进。然而使用你自己的方案而不是串行化的标准方案将使你卷入一些权衡。
除了串行化实际的I/O和格式化开销外(使用DataInputStream和 DataOutputStream), 还有其他的开销，例如在串行化恢复时的创建新对象的需要。
注意DataOutputStream 方法也可以用于开发半自定义数据格式，例如:
import java.io.*;
  import java.util.*;
  public class binary1 {
    public static void main(String args[]) {
      try {
        FileOutputStream fos = new FileOutputStream("outdata");
        BufferedOutputStream bos = new BufferedOutputStream(fos);
        DataOutputStream dos = new DataOutputStream(bos);
        Random rn = new Random();
        final int N = 10;
        dos.writeInt(N);
        for (int i = 1; i <= N; i++) {
          int r = rn.nextInt();
          System.out.println(r);
          dos.writeInt(r);
        }
        dos.close();
      }
      catch (IOException e) {
        System.err.println(e);
      }
    }
  }

和:
 import java.io.*;
  public class binary2 {
    public static void main(String args[]) {
      try {
        FileInputStream fis = new FileInputStream("outdata");
        BufferedInputStream bis = new BufferedInputStream(fis);
        DataInputStream dis = new DataInputStream(bis);
        int N = dis.readInt();
        for (int i = 1; i <= N; i++) {
          int r = dis.readInt();
          System.out.println(r);
        }
        dis.close();
      }
      catch (IOException e) {
        System.err.println(e);
      }
    }
  }
这些程序将10个整数写入文件然后读回它们
