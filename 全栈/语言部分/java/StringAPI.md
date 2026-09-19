## 常用API

| 方法名                                                       | 说明                                                     |
| ------------------------------------------------------------ | -------------------------------------------------------- |
| public int length()                                          | 获取字符串的长度返回（就是字符个数）                     |
| public char charAt(int index)                                | 获取某个索引位置处的字符返回                             |
| public char[] toCharArray()：                                | 将当前字符串转换成字符数组返回                           |
| public boolean equals(Object anObject)                       | 判断当前字符串与另一个字符串的内容一样，一样返回true     |
| public boolean equalsIgnoreCase(String anotherString)        | 判断当前字符串与另一个字符串的内容是否一样(忽略大小写)   |
| public String substring(int beginIndex, int endIndex)        | 根据开始和结束索引进行截取，得到新的字符串（包前不包后） |
| public String substring(int beginIndex)                      | 从传入的索引处截取，截取到末尾，得到新的字符串返回       |
| public String replace(CharSequence target, CharSequence replacement) | 使用新值，将字符串中的旧值替换，得到新的字符串           |
| public boolean contains(CharSequence s)                      | 判断字符串中是否包含了某个字符串                         |
| public boolean startsWith(String prefix)                     | 判断字符串是否以某个字符串内容开头，开头返回true，反之   |
| public String[] split(String regex)                          | 把字符串按照某个字符串内容分割，并返回字符串数组回来     |



## equals

equals只是比较字符串的内容，而“==”是比较字符串的地址

```java
public static void main(String[] args) {
        String a = "10";
        String c = "10";
        String d = "1";
        String e = "0";
        System.out.println(a == c);
        System.out.println(a.equals(c));
        System.out.println(a == d + e);
        System.out.println(a.equals(d + e));
    }
```

输出为true，true，false，true

a和c都指代字符串10，都是指向常量池中的字符串10，所有a和c的地址相同，而d+e也是指代字符串10，但这个 `"10"` 是在运行时新创建的对象，不在字符串常量池中。



## charAt()

char charAt(int index)，返回String中index下标位置处的char，若index不合法，抛出IndexOutOfBoundsException异常。



## getChars

public void getChars(int srcBegin, int srcEnd, char dst[], int dstBegin)，将String源中下标从srcBegin到srcEnd的字符串，复制到目标字符串中，从下标从dstBegin开始复制。当然如果下标有一个不合法，也会抛出IndexOutOfBoundsException异常。



## getBytes()

用平台默认的编码方式对String进行编码，并将结果储存到一个新的byte数组中。

```java
byte[] b_gbk = str.getBytes();  //getBytes()
```



## length

求字符串长度



## toCharArray()

将String转换成一个char数组

```java
dst = str.toCharArray(); //toCharArray()  
System.out.println(dst);   //output:I am a lucky string.
```



## equalsIgnoreCase()

和equals类似，但这个忽略大小写



## compareTo()

public int compareTo(String anotherString)，按字典顺序比较两个String的大小哦。字典顺序是说a<b<c，返回值有三种可能：1，0，-1分别表示大于，等于，小于。

```Java
if (str.compareTo("I am a unlucky string.") > 0) {   //compareTo(),Output:I am smaller  
    System.out.println("I am bigger");  
} else {  
    System.out.println("I am smaller");  
}  
```



## contains()

boolean contains(CharSequence s)，判断源String中是否含有s。包含则返回1，不包含则返回0。

```Java
if (str.contains("lucky")) {                             //contains()，Output:<span style="font-family: Arial, Helvetica, sans-serif;">I contain lucky word</span>  
    System.out.println("I contain lucky word");  
} else {  
    System.out.println("I don't contain lucky word");  
}  
```



## contentEquals()

boolean contentEquals(StringBuffer sb)，方法比较字符串到指定的CharSequence。其结果是true当且仅当此String指定序列相同的char值序列。

```Java
StringBuffer strBuf = new StringBuffer("I am a lucky string.");  
    if (str.contentEquals(strBuf)) {                             //contentEquals(),Output:The same  
        System.out.println("The same");  
    } else {  
        System.out.println("Diffenent");  
    }  
```



## regionMatches()

boolean regionMatches(boolean ignoreCase, int toffset, String other, int ooffset, int len)。第一个参数ignoreCase表示比较时是否需要忽略大小，从toffset下标开始比较String和从下表ooffset开始String other是否相等，len表示指定比较的长度。

```Java
String strNew = new String("I AM A LUCKY STRING.");  
        if (str.regionMatches(true, 7, strNew, 7, 5)) {                             //regionMatches()  
            System.out.println("The same");  //输出这一行  
        } else {  
            System.out.println("Diffenent");  
        }  
```



## startsWith()

boolean startsWith(String prefix)判断是否以prefix开头，是返回true，反之，则返回false



## endsWith()

boolean endsWith(String suffix)判断是否以prefix结尾，是返回true，反之，则返回false



## indexOf()

int indexOf(int ch)，从左往右查找ASCII码为ch的字符，如果存在则返回该字符的下标。如果不存在，则返回-1

```Java
System.out.println(str.indexOf(97));          //indexOf()     输出：2    
```





## lastIndexOf()

int indexOf(int ch)，从右往左查找ASCII码为ch的字符，如果存在则返回该字符的下标。如果不存在，则返回-1。

```Java
System.out.println(str.lastIndexOf(97));         //lastIndexOf() 输出：5  
```



## substring()

String substring(int beginIndex, int endIndex)，返回String下标从beginIndex到下标endIndex-1之间的字符串。

```Java
System.out.println(str.substring(7, 11));        //substring，输出：    luck 
```



## concat()

拼接两个字符串

```Java
System.out.println(str.concat(" Do you like me? "));        //concat，输出：I am a lucky string. Do you like me?  
```



## replace()

String replace(char oldChar, char newChar)，从这个函数原型中就可以看出就是将String中的oldChar替换成newChar啦。(全部替换)

```Java
System.out.println(str.replace('a', 'A'));        //replace，输出：I Am A lucky string.
```



## toUpperCase()和toLowerCase()

大小写转换



## trim()

将String两端的空白字符删除后，返回一个新的String对象。如果没有改变发生，则返回原String对象。

```java
strNew = "          I am a lucky string.            ";  
        System.out.println(strNew.trim());        //trim,输出：I am a lucky string.  
```



## valueOf()

返回一个表示参数内容的String，参数可以是double，int，float，char，char[]，long啊之类的，基本都可以。实际上就是类型转换啦！把别的类型的数据转换成String。一般是将基本数据类型或 `String` 类型转换为对应的包装类对象(Integer,Byte,Short,Long,Float,Double,Character,Boolean)

```Java
System.out.println(str.valueOf(8.8));      //valueOf输出：8.8  
```



## intern()

将字符串对象放入字符串常量池中，并返回常量池中的字符串引用

```Java
System.out.println(str3 == str4);         //输出：false  
        str4 = (str1 + str2).intern();            //重点：intern()  
        System.out.println(str3 == str4);         //输出：true  
```



## isEmpty

判断字符串长度是否为0



## isBlank

判断字符串是否为空



## split

split(String regex) / split(String regex, int limit)

功能：按正则表达式分割字符串，返回字符串数组（核心分割工具）

关键区别：`limit` 控制分割次数（如 `limit=-1` 保留末尾空字符串）

```Java
String str = "a,b,c,,d";
str.split(","); // ["a","b","c","","d"]（默认limit=0，丢弃末尾空串？不，这里中间空串保留）
str.split(",", 3); // ["a","b","c,,d"]（只分割2次，返回3个元素）
str.split(",", -1); // ["a","b","c","","d"]（保留所有空串）
```



## matches

matches(String regex)

判断字符串是否完全匹配正则表达式（验证手机号、邮箱等）

```Java
String phone = "13800138000";
boolean isPhone = phone.matches("1[3-9]\\d{9}"); // true（匹配手机号正则）
```



## String.join

String.join(CharSequence delimiter, CharSequence... elements)

功能：用分隔符拼接数组 / 可变参数（JDK 8+，替代 `StringBuilder` 拼接）

重载：`join(CharSequence delimiter, Iterable<? extends CharSequence> elements)`（支持集合）

```Java
String[] arr = {"a", "b", "c"};
String.join("-", arr); // "a-b-c"
List<String> list = Arrays.asList("x", "y");
String.join("|", list); // "x|y"
```



## strip()/stripLeading()`/`stripTrailing()

功能：替代 `trim()`，支持**Unicode 空白字符**（如中文全角空格 `　`）

区别：`trim()` 仅处理 ASCII 空白（`\u0020`），`strip()` 处理所有 Unicode 空白

```Java
String str = "　a b　"; // 前后是中文全角空格
str.trim(); // "　a b　"（trim() 无法处理全角空格）
str.strip(); // "a b"（去除所有Unicode空白）
str.stripLeading(); // "a b　"（仅去除开头空白）
```



## repeat

repeat(int count)

重复字符串 `count` 次（`count ≥ 0`，0 则返回空串）

```Java
"ab".repeat(3); // "ababab"
"x".repeat(0); // ""
```



## lines()

按行分割字符串，返回 `Stream<String>`（处理多行文本）

```Java
String multiLine = "a\nb\nc";
multiLine.lines().forEach(System.out::println); // 依次输出a、b、c
```



## indent()

indent(int n)

调整字符串缩进（`n > 0` 增加缩进，`n < 0` 减少缩进，基于换行符分割）

```Java
String str = "a\nb\nc";
str.indent(2); // 每行前加2个空格
str.indent(-1); // 每行前减少1个空格（最少0个）
```



## 



## 完整程序

```Java
public class StringClass {  
      
    public static void main(String[] args) {  
        String str = new String("I am a lucky string."); //构造器  
        System.out.println("My length is " + str.length()); //length()  
        System.out.println("My favoriate character is " + str.charAt(8));  //charAt() Output:u  
        char dst[] = {'a', 'p', 'o', 'o', 'r', 'g', 'i', 'r', 'l'};  
        System.out.println("Now I want to pass my lucky to a good guy");  
        str.getChars(7, 12, dst, 0);    //getChars(),Output:luckygirl  
        System.out.println(dst);  
        byte[] b_gbk = str.getBytes();  //getBytes()  
        System.out.println(b_gbk);  
        dst = str.toCharArray(); //toCharArray()  
        System.out.println(dst);   //output:I am a lucky string.  
          
        if (str.equals("I am a unlucky string.")) {   //equals()  
            System.out.println("The same");  
        } else {  
            System.out.println("Diffenent");  
        }  
          
        if (str.equalsIgnoreCase("I AM A LUCKY STRING.")) {   //equalsIgnoreCase()  
            System.out.println("The same");  
        } else {  
            System.out.println("Diffenent");  
        }  
          
        if (str.compareTo("I am a unlucky string.") > 0) {   //compareTo(),Output:I am smaller  
            System.out.println("I am bigger");  
        } else {  
            System.out.println("I am smaller");  
        }  
          
        if (str.contains("lucky")) {                             //contains()  
            System.out.println("I contain lucky word");  
        } else {  
            System.out.println("I don't contain lucky word");  
        }  
          
        StringBuffer strBuf = new StringBuffer("I am a lucky string.");  
        if (str.contentEquals(strBuf)) {                             //contentEquals(),Output:The same  
            System.out.println("The same");  
        } else {  
            System.out.println("Diffenent");  
        }  
          
        String strNew = new String("I AM A LUCKY STRING.");  
        if (str.regionMatches(true, 7, strNew, 7, 5)) {                             //regionMatches()  
            System.out.println("The same");  
        } else {  
            System.out.println("Diffenent");  
        }  
          
        if (str.startsWith("I")) {                             //startsWith()  
            System.out.println("I start with I.");  
        } else {  
            System.out.println("I don't start with I.");  
        }  
          
        if (str.endsWith("string.")) {                             //endsWith()  
            System.out.println("I end with string.");  
        } else {  
            System.out.println("I don't end with string.");  
        }  
          
        System.out.println(str.indexOf(97));          //indexOf()          
        System.out.println(str.lastIndexOf(97));         //lastIndexOf()  
        System.out.println(str.substring(7, 11));        //substring，输出：    luck      
        System.out.println(str.concat(" Do you like me? "));        //concat，输出：I am a lucky string. Do you like me?      
        System.out.println(str.replace('a', 'A'));        //replace，输出：I Am A lucky string.  
        System.out.println(str.toUpperCase());            //toUpperCase  
        strNew = "          I am a lucky string.            ";  
        System.out.println(strNew.trim());        //trim,输出：I am a lucky string.  
        System.out.println(str.valueOf(8.8));      //valueOf输出：8.8  
          
          
        String str1 = "a";   
        String str2 = "bc";   
        String str3 = "a"+"bc";   
        String str4 = str1+str2;   
  
        System.out.println(str3 == str4);         //输出：false  
        str4 = (str1 + str2).intern();            //重点：intern()  
        System.out.println(str3 == str4);         //输出：true  
    }  
}  
```

