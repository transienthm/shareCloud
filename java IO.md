# JAVA IO

##　字节与字符
###　字节
8个bit ＝ 一个byte，bit即0/1，
> 维基百科：比特（英语：Bit），亦称二进制位，指二进制中的一位，是信息的最小单位

###　多少个字节可以表示一个字符呢？
与**编码**有关。

Java中的一个char是2个字节。java采用unicode，2个字节来表示一个字符，这点与C语言中不同，c语言中采用ASCII，在大多数系统中，一个char通常占1个字节，但是在0~127整数之间的字符映射，unicode向下兼容ASCII。而Java采用unicode来表示字符，一个中文或英文字符的unicode编码都占2个字节，但如果采用其他编码方式，一个字符占用的字节数则各不相同。

## 字节流与字符流
![SouthEast][1]

字节流入即InputStream,出即OutputStream
字符流入即Reader，出即Writer

## 转换流

## 缓存与性能
http://langgufu.iteye.com/blog/2106526
```
    
```



## try catch finally
```
    try{
        inputStream = new FileInputStream(new File(""));
    }
```

  [1]: /img/bVPRwT