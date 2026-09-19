[Java：IO流详解_java io-CSDN博客](https://blog.csdn.net/m0_47114547/article/details/135430245?ops_request_misc=elastic_search_misc&request_id=b72f42a7d7ac2cd4a975d3856a177fd1&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-135430245-null-null.142^v102^pc_search_result_base9&utm_term=javaio流&spm=1018.2226.3001.4187)

[toc]

# 基础流

## IO概述

![img](https://i-blog.csdnimg.cn/blog_migrate/d61ef1c8ef11a8dd007da4391950cdef.png)

### 什么是IO流

1. 数据的传输可以看做是一种数据的流动，按照流动的方向，以内存为基准，分为`输入input` 和`输出output` ，即流向内存是[输入流](https://so.csdn.net/so/search?q=输入流&spm=1001.2101.3001.7020)，流出内存的输出流。
2. Java中I/O操作主要是指使用`java.io`包下的内容，进行输入、输出操作。**输入**也叫做**读取**数据，**输出**也叫做作**写出**数据。

### IO流的分类

根据数据的**流向**分为：输入流和输出流。

- **输入流** ：把数据从`其他设备`上读取到`内存`中的流。
- **输出流** ：把数据从`内存` 中写出到`其他设备`上的流。

格局数据的**类型**分为：字节流和字符流。

- **字节流** ：以字节为单位，读写数据的流。
- **字符流** ：以字符为单位，读写数据的流。

> 输入：硬盘–>内存
> 输出：内存–>硬盘

### 顶级父类们

|        | 输入流                | 输出流                 |
| ------ | --------------------- | ---------------------- |
| 字节流 | 字节输入流InputStream | 字节输出流OutputStream |
| 字符流 | 字符输入流Reader      | 字符输出流Writer       |

## 字节流

> 字节流读取文件的时候，文件中不要有中文

### 一切皆为字节

一切文件数据（文本、图片、视频等）在存储时，都是以二进制数字的形式保存，都一个一个的字节，那么传输时一样如此。所以，字节流可以传输任意文件数据。

### 字节输出流 OutputStream

java.io.OutputStream 抽象类是表示字节输出流的所有类的超类，将指定的字节信息写出到目的地。它定义了字节输出流的基本共性功能方法。

* public void close() ：关闭此输出流并释放与此流相关联的任何系统资源。
* public void flush() ：刷新此输出流并强制任何缓冲的输出字节被写出。
* public void write(byte[] b)：将 b.length字节从指定的字节数组写入此输出流。
* public void write(byte[] b, int off, int len) ：从指定的字节数组写入 len字节，从偏移量 off开始输出到此输出流。
* public abstract void write(int b) ：将指定的字节输出流。

> 对于close方法：当完成流的操作时，必须调用此方法，释放系统资源

### FileOutputStream类

> OutputStream有很多子类，从最简单的一个子类开始。java.io.FileOutputStream 类是文件输出流，用于将数据写出到文件

#### 构造方法

* public FileOutputStream(File file)：创建文件输出流以写入由指定的 File对象表示的文件。
* public FileOutputStream(String name)： 创建文件输出流以指定的名称写入文件。

当你创建一个流对象时，必须传入一个文件路径。该路径下，如果没有这个文件，会创建该文件。如果有这个文件，会清空这个文件的数据。

```Java
public class FileOutputStreamConstructor throws IOException {
    public static void main(String[] args) {
   	 	// 使用File对象创建流对象
        File file = new File("a.txt");
        FileOutputStream fos = new FileOutputStream(file);
      
        // 使用文件名称创建流对象
        FileOutputStream fos = new FileOutputStream("b.txt");
    }
}
```

#### 写出字节数据

1. 写出字节

> write(int b) 方法，每次可以写出一个字节数据

> 实现需求：写出一段文字到本地文件中。（暂时不写中文）
> 实现步骤：
>       创建对象
>       写出数据
>       释放资源

```Java
public class FileOutputStreamDemo01 {
    public static void main(String[] args) throws IOException {
        //1.创建对象
        //写出 输出流 OutputStream
        //本地文件    File
        FileOutputStream fos = new FileOutputStream("my-io\\a.txt");
        //2.写出数据
        fos.write(97);
        //3.释放资源
        fos.close();
    }
}
//写入结果为a
```

> 注:
>
> * 虽然参数为int类型四个字节，但是只会保留一个字节的信息写出。
> * 流操作完毕后，必须释放系统资源，调用close方法，千万记得。
> * 创建字节输出流对象
>   - 参数是字符串表示的路径或者是File对象都是可以的
>   - 如果文件不存在会创建一个新的文件，但是要保证父级路径是存在的。
>   - 如果文件已经存在，则会清空文件
> * 写数据
>   * write方法的参数是整数，但是实际上写到本地文件中的是整数在ASCII上对应的字符
>     ‘9’
>     '7’

2. 写出字节数组

> write(byte[] b)，每次可以写出数组中的数据

```Java
void write(byte[] b)：一次写一个字节数组数据
```

```Java
public class FileOutputStreamDemo03 {
	public static void main(String[] args) throws IOException {
        //1.创建对象
        FileOutputStream fos = new FileOutputStream("my-io\\a.txt");
        //2.一次写一个字节数组数据
        byte[] bytes = {97, 98, 99, 100, 101};
        fos.write(bytes);
        //4.释放资源
        fos.close();
    }
}
//写入文件的为abcde
```

3. 写出指定长度字节数组

> write(byte[] b, int off, int len) ,每次写出从off索引开始，len个字节

> void write(byte[] b, int off, int len)  一次写一个字节数组的部分数据
> 参数一：数组
> 参数二：起始索引 
> 参数三：个数

```Java
public class FileOutputStreamDemo03 {
    public static void main(String[] args) throws IOException {
        //1.创建对象
        FileOutputStream fos = new FileOutputStream("my-io\\a.txt");
        //2.一次写一个字节数组数据
        byte[] bytes = {97, 98, 99, 100, 101};
        //3.一次写一个字节数组的部分数据
        fos.write(bytes,1,2);   // b c
        //4.释放资源
        fos.close();
    }
}
//写入文件的为bc
```

#### 数据追加续写

经过以上的演示，每次程序运行，创建输出流对象，都会清空目标文件中的数据。需要在构造方法的参数传入一个boolean类型的值，true 表示追加数据，false 表示清空原有数据。这样创建的输出流对象，就可以指定是否追加续写了。

* public FileOutputStream(File file, boolean append)： 创建文件输出流以写入由指定的 File对象表示的文件。
* public FileOutputStream(String name, boolean append)： 创建文件输出流以指定的名称写入文件。

```Java
public class FileOutputStreamDemo04 {
    public static void main(String[] args) throws IOException {
        //1.创建对象，开启续写
        FileOutputStream fos = new FileOutputStream("my-io\\a.txt",true);
        //2.写出数据
        String str = "Hello";
        byte[] bytes = str.getBytes();
        fos.write(bytes);
        //3.释放资源
        fos.close();
    }
}
```

#### 写出换行

> Windows系统里，换行符号是`\r\n`

```Java
public class FileOutputStreamDemo04 {
    public static void main(String[] args) throws IOException {
        // 使用文件名称创建流对象
        FileOutputStream fos = new FileOutputStream("my-io\\a.txt");
        // 定义字节数组
        byte[] words = {97,98,99,100,101};
        // 遍历数组
        for (int i = 0; i < words.length; i++) {
            // 写出一个字节
            fos.write(words[i]);
            // 写出一个换行, 换行符号转成数组写出
            fos.write("\r\n".getBytes());
        }
        // 关闭资源
        fos.close();
    }
}
```

> * 回车符\r和换行符\n ：
>   * 回车符：回到一行的开头（return）。
>   * 换行符：下一行（newline）。
> * 系统中的换行：
>   * Windows系统里，每行结尾是 回车+换行 ，即\r\n；
>   * Unix系统里，每行结尾只有 换行 ，即\n；
>   * Mac系统里，每行结尾是 回车 ，即\r。从 Mac OS X开始与Linux统一。

### 字节输入流InputStream

java.io.InputStream 抽象类是表示字节输入流的所有类的超类，可以读取字节信息到内存中。它定义了字节输入流的基本共性功能方法。

* public void close() ：关闭此输入流并释放与此流相关联的任何系统资源。
* public abstract int read()： 从输入流读取数据的下一个字节。
* public int read(byte[] b)： 从输入流中读取一些字节数，并将它们存储到字节数组 b中 。

> close方法，当完成流的操作时，必须调用此方法，释放系统资源。

### FileInputStream类

> java.io.FileInputStream 类是文件输入流，从文件中读取字节

#### 构造方法

创建一个流对象时，必须传入一个文件路径。该路径下，如果没有该文件，会抛出FileNotFoundException 。

* FileInputStream(File file)： 通过打开与实际文件的连接来创建一个 FileInputStream ，该文件由文件系统中的 File对象 file命名。
* FileInputStream(String name)： 通过打开与实际文件的连接来创建一个 FileInputStream ，该文件由文件系统中的路径名 name命名。

```Java
public class FileInputStreamConstructor throws IOException{
    public static void main(String[] args) {
   	 	// 使用File对象创建流对象
        File file = new File("a.txt");
        FileInputStream fos = new FileInputStream(file);
      
        // 使用文件名称创建流对象
        FileInputStream fos = new FileInputStream("b.txt");
    }
}
```

#### 读取字节数据

1. 读取字节

> read方法，每次可以读取一个字节的数据，提升为int类型，读取到文件末尾，返回-1

```Java
public class FileInputStreamDemo01 {
    public static void main(String[] args) throws IOException {
        //1.创建对象
        FileInputStream fis = new FileInputStream("my-io\\a.txt");
        //2.读取数据，返回一个字节
        int read = fis.read();
        System.out.println((char) read);
        read = fis.read();
        System.out.println((char) read);
        read = fis.read();
        System.out.println((char) read);
        read = fis.read();
        System.out.println((char) read);
        read = fis.read();
        System.out.println((char) read);
        // 读取到末尾,返回-1
        read = fis.read();
        System.out.println( read);
        //3.关闭资源
        fis.close();
    }
}
```

**循环改进读取方式**

```Java
public class FileInputStreamDemo03 {
    public static void main(String[] args) throws IOException {
        // 使用文件名称创建流对象
        FileInputStream fis = new FileInputStream("my-io\\a.txt");
        // 定义变量，保存数据
        int b;
        // 循环读取
        while ((b = fis.read()) != -1) {
            System.out.println((char) b);
        }
        // 关闭资源
        fis.close();
    }
}
```

> 虽然读取了一个字节，但是会自动提升为int类型。
>
> 流操作完毕后，必须释放系统资源，调用close方法，千万记得。

2. 使用字节数组读取

> read(byte[] b)，每次读取b的长度个字节到数组中，返回读取到的有效字节个数，读取到末尾时，返回-1

```Java
public class FileInputStreamDemo05 {
    public static void main(String[] args) throws IOException {

        //1.创建对象
        FileInputStream fis = new FileInputStream("my-io\\a.txt");
        //2.读取数据
        byte[] bytes = new byte[2];
        //一次读取多个字节数据，具体读多少，跟数组的长度有关
        //返回值：本次读取到了多少个字节数据
        int len1 = fis.read(bytes);
        System.out.println(len1);//2
        String str1 = new String(bytes,0,len1);
        System.out.println(str1);//ab

        int len2 = fis.read(bytes);
        System.out.println(len2);//2
        String str2 = new String(bytes,0,len2);
        System.out.println(str2);//cd

        int len3 = fis.read(bytes);
        System.out.println(len3);// 1
        String str3 = new String(bytes,0,len3);
        System.out.println(str3);//e

        //3.释放资源
        fis.close();
    }
}
```

### 字节流示例 -- 文件拷贝

选择一个比较小的文件，不要太大

![img](https://i-blog.csdnimg.cn/blog_migrate/8905392fb71edc63c7936b677617a9c0.png)

1. 不使用字节数组拷贝

```Java
public class FileInputStreamDemo04 {
    public static void main(String[] args) throws IOException {
        long start = System.currentTimeMillis();

        //1.创建对象
        FileInputStream fis = new FileInputStream("D:\\aaa\\movie.mp4");
        FileOutputStream fos = new FileOutputStream("my-io\\copy.mp4");
        //2.拷贝
        //核心思想：边读边写
        int b;
        while((b = fis.read()) != -1){
            fos.write(b);
        }
        //3.释放资源
        //规则：先开的最后关闭
        fos.close();
        fis.close();

        long end = System.currentTimeMillis();
        System.out.println(end - start);
    }
}
//效率较低:382352ms
```

2. 使用字节数组拷贝

```Java
public class FileInputStreamDemo06 {
    public static void main(String[] args) throws IOException {
        long start = System.currentTimeMillis();

        //1.创建对象
        FileInputStream fis = new FileInputStream("D:\\aaa\\movie.mp4");
        FileOutputStream fos = new FileOutputStream("my-io\\copy.mp4");
        //2.拷贝
        int len;
        byte[] bytes = new byte[1024 * 1024 * 5];
        while((len = fis.read(bytes)) != -1){
            fos.write(bytes,0,len);
        }
        //3.释放资源
        fos.close();
        fis.close();

        long end = System.currentTimeMillis();
        System.out.println(end - start);
    }
}
//效率更高:75ms
```

> 流的关闭原则:先开后关,后开先关

## 字符流

> 当使用字节流读取文本文件时，可能会有一个小问题。就是遇到中文字符时，可能不会显示完整的字符，那是因为一个中文字符可能占用多个字节存储。所以Java提供一些字符流类，**以字符为单位**读写数据，专门用于处理文本文件

### 字符输入流 Reader

java.io.Reader抽象类是表示用于读取字符流的所有类的超类，可以读取字符信息到内存中。它定义了字符输入流的基本共性功能方法。

* public void close() ：关闭此流并释放与此流相关联的任何系统资源。
* public int read()： 从输入流读取一个字符。
* public int read(char[] cbuf)： 从输入流中读取一些字符，并将它们存储到字符数组 cbuf中 。

### FileReader类

java.io.FileReader 类是读取字符文件的便利类。构造时使用系统默认的字符编码和默认字节缓冲区。

* 字符编码：字节与字符的对应规则。Windows系统的中文编码默认是GBK编码表；idea中是Unicode字符集、UTF-8编码
* 字节缓冲区：一个字节数组，用来临时存储字节数据。

#### 构造方法

创建一个流对象时，必须传入一个文件路径，类似于FileInputStream 。

* FileReader(File file)： 创建一个新的 FileReader ，给定要读取的File对象。
* FileReader(String fileName)： 创建一个新的 FileReader ，给定要读取的文件的名称。

```Java
public class FileReaderConstructor throws IOException{
    public static void main(String[] args) {
   	 	// 使用File对象创建流对象
        File file = new File("a.txt");
        FileReader fr = new FileReader(file);
      
        // 使用文件名称创建流对象
        FileReader fr = new FileReader("b.txt");
    }
}
```

#### 读取字符数据

> 第一步：创建对象
> public FileReader(File file)        		 创建字符输入流关联本地文件
> public FileReader(String pathname)  	创建字符输入流关联本地文件
> 第二步：读取数据
> public int read()                   			读取数据，读到末尾返回-1
> public int read(char[] buffer)      		  读取多个数据，读到末尾返回-1
> 第三步：释放资源
> public void close()                 		      释放资源/关流

1. 读取字符

> read方法，每次可以读取一个字符的数据，提升为int类型，读取到文件末尾，返回-1，循环读取

```Java
public class FileReaderDemo01 {
    public static void main(String[] args) throws IOException {

        //1.创建对象并关联本地文件
        FileReader fr = new FileReader("my-io\\a.txt");

        /**
         * 2.读取数据 read()
         * 字符流的底层也是字节流，默认也是一个字节一个字节的读取的。
         * 如果遇到中文就会一次读取多个，GBK一次读两个字节，UTF-8一次读三个字节
         * read（）细节：
         * 1.read():默认也是一个字节一个字节的读取的,如果遇到中文就会一次读取多个
         * 2.在读取之后，方法的底层还会进行解码并转成十进制。
         *   最终把这个十进制作为返回值,这个十进制的数据也表示在字符集上的数字
         *   英文：文件里面二进制数据 0110 0001
         *           read方法进行读取，解码并转成十进制97
         *   中文：文件里面的二进制数据 11100110 10110001 10001001
         *           read方法进行读取，解码并转成十进制27721
         *  想看到中文汉字，就是把这些十进制数据，再进行强转就可以了
         */

        // 循环读取
        int ch;
        while((ch = fr.read()) != -1){
            System.out.println((char)ch);
        }
        //3.释放资源
        fr.close();
    }
}
```

> 虽然读取了一个字符，但是会自动提升为int类型

2. 使用字符数组读取

> read(char[] cbuf)，每次读取b的长度个字符到数组中，返回读取到的有效字符个数，读取到末尾时，返回-1

```Java
public class FileReaderDemo02 {
    public static void main(String[] args) throws IOException {

        //1.创建对象
        FileReader fr = new FileReader("my-io\\a.txt");
        //2.读取数据
        char[] chars = new char[2];
        int len;    // 获取有效的字符
        //read(chars)：读取数据，解码，强转三步合并了，把强转之后的字符放到数组当中
        //空参的read + 强转类型转换
        while((len = fr.read(chars)) != -1){
            // 把数组中的数据变成字符串再进行打印
            System.out.println(new String(chars,0,len));
        }
        //3.释放资源
        fr.close();
    }
}
```

### 字符输出流 Writer

java.io.Writer 抽象类是表示用于写出字符流的所有类的超类，将指定的字符信息写出到目的地。它定义了字节输出流的基本共性功能方法。

* void write(int c) 写入单个字符。
* void write(char[] cbuf) 写入字符数组。
* abstract void write(char[] cbuf, int off, int len) 写入字符数组的某一部分,off数组的开始索引,len写的字符个数。
* void write(String str) 写入字符串。
* void write(String str, int off, int len) 写入字符串的某一部分,off字符串的开始索引,len写的字符个数。
* void flush() 刷新该流的缓冲。
* void close() 关闭此流，但要先刷新它。

### FileWriter类

> java.io.FileWriter 类是写出字符到文件的便利类。构造时使用系统默认的字符编码和默认字节缓冲区

#### 构造方法

创建一个流对象时，必须传入一个文件路径，类似于FileOutputStream。

- FileWriter(File file)： 创建一个新的 FileWriter，给定要读取的File对象。
- FileWriter(String fileName)： 创建一个新的 FileWriter，给定要读取的文件的名称。

```Java
public class FileWriterConstructor {
    public static void main(String[] args) throws IOException {
   	 	// 使用File对象创建流对象
        File file = new File("a.txt");
        FileWriter fw = new FileWriter(file);
      
        // 使用文件名称创建流对象
        FileWriter fw = new FileWriter("b.txt");
    }
}
```

#### 基本写出数据

**写出字符**：`write(int b)` 方法，每次可以写出一个字符数据

```Java
public class FWWrite {
    public static void main(String[] args) throws IOException {
        // 使用文件名称创建流对象
        FileWriter fw = new FileWriter("fw.txt");     
      	// 写出数据
      	fw.write(97); // 写出第1个字符
      	fw.write('b'); // 写出第2个字符
      	fw.write('C'); // 写出第3个字符
      	fw.write(30000); // 写出第4个字符，中文编码表中30000对应一个汉字。
      
      	/*
        【注意】关闭资源时,与FileOutputStream不同。
      	 如果不关闭,数据只是保存到缓冲区，并未保存到文件。
        */
        // fw.close();
    }
}
输出结果：
abC田
```

> - 虽然参数为int类型四个字节，但是只会保留一个字符的信息写出。
> - 未调用close方法，数据只是保存到了缓冲区，并未写出到文件中。

#### 关闭和刷新

因为内置缓冲区的原因，如果不关闭输出流，无法写出字符到文件中。但是关闭的流对象，是无法继续写出数据的。如果既想写出数据，又想继续使用流，就需要flush 方法了。

* flush ：刷新缓冲区，流对象可以继续使用。
* close ：先刷新缓冲区，然后通知系统释放资源，流对象不可以再被使用了。

```Java
public class FWWrite {
    public static void main(String[] args) throws IOException {
        // 使用文件名称创建流对象
        FileWriter fw = new FileWriter("fw.txt");
        // 写出数据，通过flush
        fw.write('刷'); // 写出第1个字符
        fw.flush();
        fw.write('新'); // 继续写出第2个字符，写出成功
        fw.flush();
      
      	// 写出数据，通过close
        fw.write('关'); // 写出第1个字符
        fw.close();
        fw.write('闭'); // 继续写出第2个字符,【报错】java.io.IOException: Stream closed
        fw.close();
    }
}
```

> 即便是flush方法写出了数据，操作的最后还是要调用close方法，释放系统资源。

#### 写出其他数据

> 第一步：创建对象
>     public FileWriter(File file)                            		     创建字符输出流关联本地文件
>     public FileWriter(String pathname)                      	    创建字符输出流关联本地文件
>     public FileWriter(File file,  boolean append)           	 创建字符输出流关联本地文件，续写
>     public FileWriter(String pathname,  boolean append)     创建字符输出流关联本地文件，续写
> 第二步：读取数据
>     void write(int c)                           		写出一个字符
>     void write(String str)                     	      写出一个字符串
>     void write(String str, int off, int len)    	 写出一个字符串的一部分
>     void write(char[] cbuf)                    	     写出一个字符数组
>     void write(char[] cbuf, int off, int len)           写出字符数组的一部分
> 第三步：释放资源
>     public void close()                 		      释放资源/关流

1. 写出字符数组

> write(char[] cbuf) 和 write(char[] cbuf, int off, int len) ，每次可以写出字符数组中的数据，用法类似FileOutputStream

```Java
public class FileWriterDemo01 {
    public static void main(String[] args) throws IOException {
        // 创建流对象并开启续写
        FileWriter fw = new FileWriter("my-io\\a.txt",true);

        //fw.write(25105);
        //fw.write("你好威啊???");
        char[] chars = {'a','b','c','我'};
        fw.write(chars);

        fw.close();
    }
}
```

2. 写出字符串

> write(String str) 和 write(String str, int off, int len) ，每次可以写出字符串中的数据，更为方便

```Java
public class FileWriterDemo03 {
    public static void main(String[] args) throws IOException {

        // 创建流对象，未开启续写
        FileWriter fw = new FileWriter("my-io\\a.txt");

        fw.write("我的同学各个都很厉害");
        fw.write("说话声音很好听");

        fw.flush();

        fw.write("都是人才");
        fw.write("超爱这里哟");

        fw.close();
    }
}
```

3. 续写和换行

> 操作类似于FileOutputStream

```Java
public class FileWriterDemo03 {
    public static void main(String[] args) throws IOException {
        // 使用文件名称创建流对象，可以续写数据
        FileWriter fw = new FileWriter("my-io\\a.txt",true);
        // 写出字符串
        fw.write("你好");
        // 写出换行
        fw.write("\r\n");
        // 写出字符串
        fw.write("我是张三");
        // 关闭资源
        fw.close();
    }
}
```



# 高级流

## 缓冲流

### 概述

缓冲流，也叫高效流，是对4个基本的FileXxx 流的增强，所以也是4个流，按照数据类型分类：

* 字节缓冲流：BufferedInputStream，BufferedOutputStream
* 字符缓冲流：BufferedReader，BufferedWriter

缓冲流的基本原理，是在创建流对象时，会创建一个内置的默认大小的缓冲区数组，通过缓冲区读写，减少系统IO次数，从而提高读写的效率。

### 字节缓冲流

#### 构造方法

* public BufferedInputStream(InputStream in) ：创建一个 新的缓冲输入流。
* public BufferedOutputStream(OutputStream out)： 创建一个新的缓冲输出流。

```Java
// 创建字节缓冲输入流
BufferedInputStream bis = new BufferedInputStream(new FileInputStream("bis.txt"));
// 创建字节缓冲输出流
BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("bos.txt"));
```

#### 效率测试

查询API，缓冲流读写方法与基本的流是一致的，我们通过复制大文件（375MB），测试它的效率。

1. 基本流

```Java
public class BufferedDemo {
    public static void main(String[] args) throws FileNotFoundException {
        // 记录开始时间
      	long start = System.currentTimeMillis();
		// 创建流对象
        try (
        	FileInputStream fis = new FileInputStream("jdk9.exe");
        	FileOutputStream fos = new FileOutputStream("copy.exe")
        ){
        	// 读写数据
            int b;
            while ((b = fis.read()) != -1) {
                fos.write(b);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
		// 记录结束时间
        long end = System.currentTimeMillis();
        System.out.println("普通流复制时间:"+(end - start)+" 毫秒");
    }
}

十几分钟过去了...

```

2. 缓冲流

```Java
public class BufferedDemo {
    public static void main(String[] args) throws FileNotFoundException {
        // 记录开始时间
      	long start = System.currentTimeMillis();
		// 创建流对象
        try (
        	BufferedInputStream bis = new BufferedInputStream(new FileInputStream("jdk9.exe"));
	     BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("copy.exe"));
        ){
        // 读写数据
            int b;
            while ((b = bis.read()) != -1) {
                bos.write(b);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
		// 记录结束时间
        long end = System.currentTimeMillis();
        System.out.println("缓冲流复制时间:"+(end - start)+" 毫秒");
    }
}

缓冲流复制时间:8016 毫秒
```

3. 如何更快呢？使用数组的方式

```Java
public class BufferedDemo {
    public static void main(String[] args) throws FileNotFoundException {
      	// 记录开始时间
        long start = System.currentTimeMillis();
		// 创建流对象
        try (
			BufferedInputStream bis = new BufferedInputStream(new FileInputStream("jdk9.exe"));
		 BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("copy.exe"));
        ){
          	// 读写数据
            int len;
            byte[] bytes = new byte[8*1024];
            while ((len = bis.read(bytes)) != -1) {
                bos.write(bytes, 0 , len);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
		// 记录结束时间
        long end = System.currentTimeMillis();
        System.out.println("缓冲流使用数组复制时间:"+(end - start)+" 毫秒");
    }
}
缓冲流使用数组复制时间:666 毫秒
```

### 字符缓冲流

#### 构造方法

* public BufferedReader(Reader in) ：创建一个 新的缓冲输入流。
* public BufferedWriter(Writer out)： 创建一个新的缓冲输出流。

```Java
// 创建字符缓冲输入流
BufferedReader br = new BufferedReader(new FileReader("br.txt"));
// 创建字符缓冲输出流
BufferedWriter bw = new BufferedWriter(new FileWriter("bw.txt"));
```

#### 特有方法

字符缓冲流的基本方法与普通字符流调用方式一致，不再阐述，我们来看它们具备的特有方法。

* BufferedReader：public String readLine(): 读一行文字。
* BufferedWriter：public void newLine(): 写一行行分隔符,由系统属性定义符号。

1. readLine方法在读取的时候，一次读一整行，遇到回车换行结束，但是不会把回车换行读到内存当中

```Java
public class BufferedStreamDemo03 {
    public static void main(String[] args) throws IOException {
        //1.创建字符缓冲输入流的对象
        BufferedReader br = new BufferedReader(new FileReader("my-io\\a.txt"));

        //2.读取数据
        /*String line1 = br.readLine();
        System.out.println(line1);

        String line2 = br.readLine();
        System.out.println(line2);*/

        String line;
        while ((( line = br.readLine()) != null)){
            System.out.println(line);
        }
        //3.释放资源
        br.close();
    }
}
```

2. newLine方法演示

```Java
public class BufferedStreamDemo04 {
  public static void main(String[] args) throws IOException {

      //1.创建字符缓冲输出流的对象
      BufferedWriter bw = new BufferedWriter(new FileWriter("my-io/b.txt",true));
      //2.写出数据
      bw.write("你好");
      bw.newLine();
      bw.write("我是张三");
      bw.newLine();
      //3.释放资源
      bw.close();
  }
}
```

## 转换流

### 字符编码和字符集

> GBK：中文2个字节，英文一个字节
> UTF-8：中文3个字节，英文一个字节（UTF-8不是字符集，Unicode才是字符集，UTF-8只是Unicode字符集下的一种编码方式）

#### 字符编码

编码：字符(能看懂的)–字节(看不懂的)
解码：字节(看不懂的)–>字符(能看懂的)
字符编码`Character Encoding` : 就是一套自然语言的字符与二进制数之间的对应规则。
编码表：生活中文字和计算机中二进制的对应规则

#### 字符集

- **字符集 `Charset`**：也叫编码表。是一个系统支持的所有字符的集合，包括各国家文字、标点符号、图形符号、数字等。

计算机要准确的存储和识别各种字符集符号，需要进行字符编码，一套字符集必然至少有一套字符编码。常见字符集有ASCII字符集、GBK字符集、Unicode字符集等.

![img](https://i-blog.csdnimg.cn/blog_migrate/45073f9c4a7cb4c9af34da6cb5026b68.jpeg)

* ASCII字符集 ：
  * ASCII用于显示现代英语，主要包括控制字符（回车键、退格、换行键等）和可显示字符（英文大小写字符、阿拉伯数字和西文符号）。
  * 基本的ASCII字符集，使用7位（bits）表示一个字符，共128字符。ASCII的扩展字符集使用8位（bits）表示一个字符，共256字符，方便支持欧洲常用字符。

* GBxxx字符集：
  * GB2312：简体中文码表。一个小于127的字符的意义与原来相同，但两个大于127的字符连在一起时，就表示一个汉字，这样大约可以组合了包含7000多个简体汉字，此外数学符号、罗马希腊的字母、日文的假名们都编进去了，连在ASCII里本来就有的数字、标点、字母都统统重新编了两个字节长的编码，这就是常说的"全角"字符，而原来在127号以下的那些就叫"半角"字符了。
  * GBK：最常用的中文码表。是在GB2312标准基础上的扩展规范，使用了双字节编码方案，共收录了21003个汉字，完全GB2312标准，同时支持繁体汉字以及日韩汉字等。
  * GB18030：最新的中文码表。收录汉字70244个，采用多字节编码，每个字可以由1个、2个或4个字节组成。支持中国国内少数民族的文字，同时支持繁体汉字以及日韩汉字等。

* Unicode字符集 ：

  * 它最多使用4个字节的数字来表达每个字母、符号，或者文字。有三种编码方案，UTF-8、UTF-16和UTF-32。最为常用的UTF-8编码。

  * UTF-8编码，可以用来表示Unicode标准中任何字符，它是电子邮件、网页及其他存储或传送文字的应用中，优先采用的编码。它使用一至四个字节为每个字符编码，编码规则：
    * 128个US-ASCII字符，只需一个字节编码。
    * 拉丁文等字符，需要二个字节编码。
    * 大部分常用字（含中文），使用三个字节编码。
    * 其他极少使用的Unicode辅助字符，使用四字节编码。

#### 编码引出的问题

> 在IDEA中，使用`FileReader` 读取项目中的文本文件。由于IDEA的设置，都是默认的`UTF-8`编码，所以没有任何问题。但是，当读取Windows系统中创建的文本文件时，由于Windows系统的默认是GBK编码，就会出现乱码。

```Java
public class ReaderDemo {
    public static void main(String[] args) throws IOException {
        FileReader fileReader = new FileReader("E:\\File_GBK.txt");
        int read;
        while ((read = fileReader.read()) != -1) {
            System.out.print((char)read);
        }
        fileReader.close();
    }
}
输出结果：
���
```

### InputStreamReader类

> 注意：这种做法是JDK11之前的，JDK11及之后使用FileReader指定字符集

#### 构造方法

* InputStreamReader(InputStream in): 创建一个使用默认字符集的字符流。
* InputStreamReader(InputStream in, String charsetName): 创建一个指定字符集的字符流。

```Java
InputStreamReader isr = new InputStreamReader(new FileInputStream("in.txt"));
InputStreamReader isr2 = new InputStreamReader(new FileInputStream("in.txt") , "GBK");
```

#### 指定编码读取

1. JDK11之前的做法：使用InputStreamReader

```Java
public class ConvertStreamDemo02 {
    public static void main(String[] args) throws IOException {
        //1.创建对象并指定字符编码(了解)
        InputStreamReader isr = new InputStreamReader(new FileInputStream("my-io\\gbkfile.txt"),"GBK");
        //2.读取数据
        int ch;
        while ((ch = isr.read()) != -1){
            System.out.print((char)ch);
        }
        //3.释放资源
        isr.close();
    }
}
```

2. JDK11及之后的做法：使用FileReader

```Java
public class ConvertStreamDemo02 {
    public static void main(String[] args) throws IOException {
        // JDK11之后的方法
        FileReader fr = new FileReader("my-io\\gbkfile.txt", Charset.forName("GBK"));
        //2.读取数据
        int ch2;
        while ((ch2 = fr.read()) != -1){
            System.out.print((char)ch2);
        }
        //3.释放资源
        fr.close();
    }
}
```

### OutputStreamWriter类

> 注意：这种做法是JDK11之前的，JDK11及之后使用FileWriter指定字符集。

转换流`java.io.OutputStreamWriter` ，是Writer的子类，是**从字符流到字节流的桥梁**。使用指定的字符集将字符编码为字节。它的字符集可以由名称指定，也可以接受平台的默认字符集。

#### 构造方法

* OutputStreamWriter(OutputStream in)：创建一个使用默认字符集的字符流。
* OutputStreamWriter(OutputStream in, String charsetName)：创建一个指定字符集的字符流。

```Java
OutputStreamWriter isr = new OutputStreamWriter(new FileOutputStream("out.txt"));
OutputStreamWriter isr2 = new OutputStreamWriter(new FileOutputStream("out.txt") , "GBK");
```

#### 指定编码写出

1. JDK11之前：使用OutputStreamWriter

```Java
public class ConvertStreamDemo03 {
    public static void main(String[] args) throws IOException {
        //1.创建转换流的对象(JDK11之前)
        OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream("my-io\\b.txt"),"GBK");
        //2.写出数据
        osw.write("你好你好");
        //3.释放资源
        osw.close();
    }
}
```

2. JDK11及之后：使用FileWriter

```Java
public class ConvertStreamDemo03 {
    public static void main(String[] args) throws IOException {
        FileWriter fw = new FileWriter("my-io\\c.txt", Charset.forName("GBK"));
        fw.write("你好你好");
        fw.close();
    }
}
```

## 序列化流

### 概述

Java 提供了一种对象序列化的机制。用一个字节序列可以表示一个对象，该字节序列包含该对象的数据、对象的类型和对象中存储的属性等信息。字节序列写出到文件之后，相当于文件中持久保存了一个对象的信息。

反之，该字节序列还可以从文件中读取回来，重构对象，对它进行反序列化。对象的数据、对象的类型和对象中存储的数据信息，都可以用来在内存中创建对象。

### ObjectOutputStream类

> java.io.ObjectOutputStream 类，将Java对象的原始数据类型写出到文件，实现对象的持久存储。

#### 构造方法

* public ObjectOutputStream(OutputStream out) ： 创建一个指定OutputStream的ObjectOutputStream，即把基本流变成高级流。

```Java
FileOutputStream fileOut = new FileOutputStream("employee.txt");
ObjectOutputStream out = new ObjectOutputStream(fileOut);
```

#### 序列化操作

1. 一个对象要想序列化，必须满足两个条件：

   * 该类必须实现java.io.Serializable 接口，Serializable 是一个标记接口，不实现此接口的类将不会使任何状态序列化或反序列化，会抛出NotSerializableException 。

   * 该类的所有属性必须是可序列化的。如果有一个属性不需要可序列化的，则该属性必须注明是瞬态的，使用transient 关键字修饰。

```Java
public class Student implements Serializable {
    
    private String name;
    private int age;
    
    //transient：瞬态关键字
    //作用：不会把当前属性序列化到本地文件当中
    private transient String address;

    public Student() {
    }

    public Student(String name, int age, String address) {
        this.name = name;
        this.age = age;
        this.address = address;
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

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public String toString() {
        return "Student{name = " + name + ", age = " + age + ", address = " + address + "}";
    }
}
```

2. 写出对象方法

* public final void writeObject (Object obj)：将指定的对象写出。

```Java
public class ObjectStreamDemo01 {
    public static void main(String[] args) throws IOException {
        //1.创建对象
        Student stu = new Student("zhangsan",23,"北京");

        //2.创建序列化流的对象/对象操作输出流
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("my-io\\a.txt"));

        //3.写出数据
        oos.writeObject(stu);

        //4.释放资源
        oos.close();
    }
}
```

### ObjectInputStream类

> ObjectInputStream反序列化流，将之前使用ObjectOutputStream序列化的原始数据恢复为对象。

#### 构造方法

* public ObjectInputStream(InputStream in) ： 创建一个指定InputStream的ObjectInputStream，即把基本流变成高级流。

#### 反序列化操作

如果能找到一个对象的class文件，可以进行反序列化操作，调用`ObjectInputStream`读取对象的方法：

* public final Object readObject ()` : 读取一个对象。

```Java
public class ObjectStreamDemo02 {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        //1.创建反序列化流的对象
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("my-io\\a.txt"));

        //2.读取数据
        Student o = (Student) ois.readObject();

        //3.打印对象
        System.out.println(o);

        //4.释放资源
        ois.close();
    }
}
```

> 对于JVM可以反序列化对象，它必须是能够找到class文件的类。如果找不到该类的class文件，则抛出一个 `ClassNotFoundException` 异常

#### 反序列化操作问题

当JVM反序列化对象时，能找到class文件，但是class文件在序列化对象之后，实体类进行了修改（比如修改了属性等），那么反序列化操作也会失败，抛出一个InvalidClassException异常。发生这个异常的原因如下：

* 该类的序列版本号与从流中读取的类描述符的版本号不匹配
* 该类包含未知数据类型
* 该类没有可访问的无参数构造方法

**解决：**

Serializable接口给需要序列化的类，提供了一个序列版本号。serialVersionUID该版本号的目的在于验证序列化的对象和对应类是否版本匹配。

## 打印流

> 只有写，没有读。

### 概述

平时我们在控制台打印输出，是调用`print`方法和`println`方法完成的，这两个方法都来自于`java.io.PrintStream`类，该类能够方便地打印各种数据类型的值，是一种便捷的输出方式。

### 字节打印流PrintStream

#### 构造方法

* public PrintStream(String fileName) ： 使用指定的文件名创建一个新的打印流。

```Java
PrintStream ps = new PrintStream("ps.txt")；
```

#### 改变打印流向

System.out就是PrintStream类型的，只不过它的流向是系统规定的，打印在控制台上。不过，既然是流对象，我们就可以玩一个"小把戏"，改变它的流向。

```Java
public class PrintDemo {
    public static void main(String[] args) throws IOException {
		// 调用系统的打印流,控制台直接输出97
        System.out.println(97);
      
		// 创建打印流,指定文件的名称
        PrintStream ps = new PrintStream("ps.txt");
      	
      	// 设置系统的打印流流向,输出到ps.txt
        System.setOut(ps);
      	// 调用系统的打印流,ps.txt中输出97
        System.out.println(97);
    }
}
```

#### 字节打印流基本使用

##### 构造方法

```Java
public PrintStream(OutputStream/File/String)            关联字节输出流/文件/文件路径
public PrintStream(String fileName, Charset charset)    指定字符编码
public PrintStream(OutputStreamout, boolean autoFlush)  自动刷新
public PrintStream(OutputStream out, boolean autoFlush, String encoding)    指定字符编码且自动刷新
```

##### 成员方法

```Java
public void write(int b)            常规方法：规则跟之前一样，将指定的字节写出
public void println(Xxx xx)         特有方法：打印任意数据，自动刷新，自动换行
public void print(Xxx xx)           特有方法：打印任意数据，不换行
public void printf(String format, Object... args)   特有方法：带有占位符的打印语句，不换行
```

示例:

```Java
public class PrintStreamDemo01 {
    public static void main(String[] args) throws FileNotFoundException {
        //1.创建字节打印流的对象
        PrintStream ps = new PrintStream(new FileOutputStream("my-io\\a.txt"), true, Charset.forName("UTF-8"));
        //2.写出数据
        ps.println(97);	//写出 + 自动刷新 + 自动换行
        ps.print(true);
        ps.println();
        ps.printf("%s爱上了%s","阿珍","阿强");
        //3.释放资源
        ps.close();
    }
}
```

### 字符打印流

#### 构造方法

```Java
public PrintWriter(Write/File/String)            关联字节输出流/文件/文件路径
public PrintWriter(String fileName, Charset charset)    指定字符编码
public PrintWriter(Write, boolean autoFlush)  自动刷新
public PrintWriter(Write out, boolean autoFlush, String encoding)    指定字符编码且自动刷新
```

#### 成员方法

```Java
public void write(int b)            常规方法：规则跟之前一样，将指定的字节写出
public void println(Xxx xx)         特有方法：打印任意数据，自动刷新，自动换行
public void print(Xxx xx)           特有方法：打印任意数据，不换行
public void printf(String format, Object... args)   特有方法：带有占位符的打印语句，不换行
```

示例:

```Java
public class PrintStreamDemo03 {
    public static void main(String[] args) throws IOException {
        //1.创建字符打印流的对象
        PrintWriter pw = new PrintWriter(new FileWriter("my-io\\a.txt"),true);

        //2.写出数据
        pw.println("今天你终于叫我名字了，虽然叫错了，但是没关系，我马上改");
        pw.print("你好你好");
        pw.printf("%s爱上了%s","阿珍","阿强");
        //3.释放资源
        pw.close();
    }
}
```

## 压缩流和解压缩流

> Java中只能识别zip格式的。

压缩流：负责压缩文件或者文件夹

解压缩流：负责把压缩包中的文件和文件夹解压出来

### 解压流 ZipInputStream

可以解压多级文件夹

```Java
public class ZipStreamDemo01 {
    public static void main(String[] args) throws IOException {

        //1.创建一个File表示要解压的压缩包
        File src = new File("D:\\aaa\\src.zip");
        //2.创建一个File表示解压的目的地
        File dest = new File("D:\\aaa\\dest");

        //调用方法
        unzip(src,dest);

    }

    //定义一个方法用来解压
    public static void unzip(File src,File dest) throws IOException {
        //解压的本质：把压缩包里面的每一个文件或者文件夹读取出来，按照层级拷贝到目的地当中
        //创建一个解压缩流用来读取压缩包中的数据
        ZipInputStream zip = new ZipInputStream(new FileInputStream(src));
        //要先获取到压缩包里面的每一个zipentry对象
        //表示当前在压缩包中获取到的文件或者文件夹
        ZipEntry entry;
        while((entry = zip.getNextEntry()) != null){
            System.out.println(entry);
            if(entry.isDirectory()){
                //文件夹：需要在目的地dest处创建一个同样的文件夹
                File file = new File(dest,entry.toString());
                file.mkdirs();
            }else{
                //文件：需要读取到压缩包中的文件，并把他存放到目的地dest文件夹中（按照层级目录进行存放）
                FileOutputStream fos = new FileOutputStream(new File(dest,entry.toString()));
                int b;
                while((b = zip.read()) != -1){
                    //写到目的地
                    fos.write(b);
                }
                fos.close();
                //表示在压缩包中的一个文件处理完毕了。
                zip.closeEntry();
            }
        }
        zip.close();
    }
}
```

结果:

![image-20240105085321409](https://i-blog.csdnimg.cn/blog_migrate/9912d084a3f617ea83c7ae12d1b008ce.png)

### 压缩流 ZipOutputStream

#### 压缩单个文件

```Java
public class ZipStreamDemo02 {
    public static void main(String[] args) throws IOException {
        //1.创建File对象表示要压缩的文件
        File src = new File("D:\\aaa\\a.txt");
        //2.创建File对象表示压缩包的位置
        File dest = new File("D:\\aaa");
        //3.调用方法用来压缩
        toZip(src,dest);
    }

    /**
     * 压缩
     * @param src 表示要压缩的文件
     * @param dest 表示压缩包的位置
     */
    public static void toZip(File src,File dest) throws IOException {
        //1.创建压缩流关联压缩包
        ZipOutputStream zos = new ZipOutputStream(new FileOutputStream(new File(dest,"a.zip")));
        //2.创建ZipEntry对象，表示压缩包里面的每一个文件和文件夹
        //参数：压缩包里面的路径
        ZipEntry entry = new ZipEntry("aaa\\bbb\\a.txt");
        //3.把ZipEntry对象放到压缩包当中
        zos.putNextEntry(entry);
        //4.把src文件中的数据写到压缩包当中
        FileInputStream fis = new FileInputStream(src);
        int b;
        while((b = fis.read()) != -1){
            zos.write(b);
        }
        zos.closeEntry();
        zos.close();
    }
}
```

### 压缩多级文件夹

```Java
public class ZipStreamDemo03 {
    public static void main(String[] args) throws IOException {

        //1.创建File对象表示要压缩的文件夹
        File src = new File("D:\\aaa\\src");
        //2.创建File对象表示压缩包放在哪里（压缩包的父级路径）
        File destParent = src.getParentFile();
        //3.创建File对象表示压缩包的路径
        File dest = new File(destParent,src.getName() + ".zip");
        //4.创建压缩流关联压缩包
        ZipOutputStream zos = new ZipOutputStream(new FileOutputStream(dest));
        //5.获取src里面的每一个文件，变成ZipEntry对象，放入到压缩包当中
        toZip(src,zos,src.getName());
        //6.释放资源
        zos.close();
    }

    /**
     * 获取src里面的每一个文件，变成ZipEntry对象，放入到压缩包当中
     * @param src 数据源
     * @param zos 压缩流
     * @param name 压缩包内部的路径
     */
    public static void toZip(File src, ZipOutputStream zos, String name) throws IOException {
        //1.进入src文件夹
        File[] files = src.listFiles();
        //2.遍历数组
        for (File file : files) {
            if(file.isFile()){
                //3.判断-文件，变成ZipEntry对象，放入到压缩包当中
                ZipEntry entry = new ZipEntry(name + "\\" + file.getName());
                zos.putNextEntry(entry);
                //读取文件中的数据，写到压缩包
                FileInputStream fis = new FileInputStream(file);
                int b;
                while((b = fis.read()) != -1){
                    zos.write(b);
                }
                fis.close();
                zos.closeEntry();
            }else{
                //4.判断-文件夹，递归
                toZip(file,zos,name + "\\" + file.getName());
            }
        }
    }
}
```

## 工具包Commons-io

> 官网：https://commons.apache.org/

Commons是apache开源基金组织提供的工具包，里面有很多帮助我们提高开发效率的API，比如：

* StringUtils：字符串工具类
* NumberUtils：数字工具类
* ArrayUtils：数组工具类
* RandomUtils：随机数工具类
* DateUtils：日期工具类
* StopWatch：秒表工具类
* ClassUtils：反射工具类
* SystemUtils：系统工具类
* MapUtils：集合工具类
* Beanutils：bean工具类
* Commons-io：io的工具类

其中Commons-io是apache开源基金组织提供的一组有关IO操作的开源工具包，用于提高IO流的开发效率。

> 官网：https://commons.apache.org/proper/commons-io/
>
> 文档：https://commons.apache.org/proper/commons-io/apidocs/index.html

> 使用方式：
>
> - 新建lib文件夹
> - 把第三方jar包粘贴到文件夹中
> - 右键点击add as a library

**Commons-io中的常用方法：**

```Java
FileUtils类
      static void copyFile(File srcFile, File destFile)                   复制文件
      static void copyDirectory(File srcDir, File destDir)                复制文件夹
      static void copyDirectoryToDirectory(File srcDir, File destDir)     复制文件夹
      static void deleteDirectory(File directory)                         删除文件夹
      static void cleanDirectory(File directory)                          清空文件夹
      static String readFileToString(File file, Charset encoding)         读取文件中的数据变成成字符串
      static void write(File file, CharSequence data, String encoding)    写出数据
  IOUtils类
      public static int copy(InputStream input, OutputStream output)      复制文件
      public static int copyLarge(Reader input, Writer output)            复制大文件
      public static String readLines(Reader input)                        读取数据
      public static void write(String data, OutputStream output)          写出数据
```

示例:

```Java
public class CommonsIODemo {
    public static void main(String[] args) throws IOException {
        // 文件复制
        File src = new File("my-io\\a.txt");
        File dest = new File("my-io\\copy.txt");
        FileUtils.copyFile(src,dest);

        // 文件夹复制
        File src2 = new File("D:\\aaa\\src");
        File dest2 = new File("D:\\aaa\\bbb");
        FileUtils.copyDirectoryToDirectory(src2,dest2);

        // 清除文件夹D:\aaa\bbb的内容
        File src3 = new File("D:\\aaa\\bbb");
        FileUtils.cleanDirectory(src3);
    }
}
```

## 工具包Hutool

> 官网：https://hutool.cn/
> API文档：https://plus.hutool.cn/apidocs/
> 中文使用文档：https://doc.hutool.cn/pages/index/

Commons是国人开发的开源工具包，里面有很多帮助我们提高开发效率的API，比如：

* DateUtil：日期时间工具类
* TimeInterval：计时器工具类
* StrUtil：字符串工具类
* HexUtil：16进制工具类
* HashUtil：Hash算法类
* ObjectUtil：对象工具类
* ReflectUtil：反射工具类
* TypeUtil：泛型类型工具类
* PageUtil：分页工具类
* NumberUtil：数字工具类

> 使用方式：
>
> - 新建lib文件夹
> - 把第三方jar包粘贴到文件夹中
> - 右键点击add as a library

**常用方法：**

![image-20240105100706118](https://i-blog.csdnimg.cn/blog_migrate/3ca08ac7b8fd6bd777995e28d9630547.png)

```Java
FileUtil类:
   file：根据参数创建一个file对象
   touch：创建文件，如果父目录不存在也自动创建
   writeLines：把集合中的数据写出到文件中，覆盖模式。
   appendLines：把集合中的数据写出到文件中，续写模式。
   readLines：指定字符编码，把文件中的数据，读到集合中。
   readUtf8Lines：按照UTF-8的形式，把文件中的数据，读到集合中
   copy：拷贝文件或者文件夹
```

示例:

```Java
public class HuToolDemo {
    public static void main(String[] args) {
        // 拼接路径创建File对象
        File file1 = FileUtil.file("D:\\", "aaa", "bbb", "a.txt");
        System.out.println(file1);//D:\aaa\bbb\a.txt

        // touch创建文件，如果父目录不存在也自动创建
        File touch = FileUtil.touch(file1);
        System.out.println(touch);
    }
}
```

```Java
public class HuToolDemo {
    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<>();
        list.add("aaa");
        list.add("aaa");
        list.add("aaa");

        // writeLines把集合中的数据写出到文件中，覆盖模式
        File file2 = FileUtil.writeLines(list, "D:\\aaa\\a.txt", "UTF-8");
        System.out.println(file2);

        // appendLines把集合中的数据写出到文件中，续写模式
        File file3 = FileUtil.appendLines(list, "D:\\aaa\\a.txt", "UTF-8");
        System.out.println(file3);

        // readLines指定字符编码，把文件中的数据，读到集合中
        List<String> list2 = FileUtil.readLines("D:\\aaa\\a.txt", "UTF-8");
        System.out.println(list2);
    }
}
```

## 配置文件操作IO流

> 创建一个空的配置文件 a.properties

### 向配置文件中存放数据

Properties集合+IO流：Properties跟IO流结合的操作，向a.properties中写入集合Properties中的数据

```Java
public class Test11 {
    public static void main(String[] args) throws IOException {
        //1.创建集合
        Properties prop = new Properties();

        //2.添加数据
        prop.put("aaa","bbb");
        prop.put("bbb","ccc");
        prop.put("ddd","eee");
        prop.put("fff","iii");

        //3.把集合中的数据以键值对的形式写到本地文件当中

        // 方法一：
        /*FileOutputStream fos = new FileOutputStream("my-io\\a.properties");
        prop.store(fos,"test");
        fos.close();*/

        // 方法二：
        BufferedWriter bw = new BufferedWriter(new FileWriter("my-io\\a.properties"));
        Set<Map.Entry<Object, Object>> entries = prop.entrySet();
        for (Map.Entry<Object, Object> entry : entries) {
            Object key = entry.getKey();
            Object value = entry.getValue();
            bw.write(key + "=" + value);
            bw.newLine();
        }
        bw.close();
    }
}
```

方法一：

![image-20240105102605502](https://i-blog.csdnimg.cn/blog_migrate/e4f58519c3f17e74b898e87d6a9b175c.png)

方法二：

![image-20240105102643000](https://i-blog.csdnimg.cn/blog_migrate/51681a4e144cb8be611fb1e90904e61e.png)

### 读取配置文件的数据

Properties集合+IO流：读取配置文件a.properties中的数据，存放到Properties集合中并打印出来

```Java
public class Test12 {
    public static void main(String[] args) throws IOException {
        //1.创建集合
        Properties prop = new Properties();
        //2.读取本地Properties文件里面的数据
        FileInputStream fis = new FileInputStream("my-io\\a.properties");
        prop.load(fis);
        fis.close();

        //3.打印集合
        System.out.println(prop);
    }
}
```

结果:

![image-20240105102819434](https://i-blog.csdnimg.cn/blog_migrate/ebb5d691992dea331f946935fa252336.png)