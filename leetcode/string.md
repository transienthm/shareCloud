字符串简介

String 内置类型，不可理性，要更改的话考虑转StringBuffer,StringBuilder，char[]之类

对java来说，一个char的范围 [0,65535]，16位

面试题总体分析
- 和数组相关，内容广泛
	- 概念理解：字典序，哪个排在字典前面，哪个字典序就小
	- 简单操作： 插入、删除字符，旋转
	- 规则判断 罗马数字转换，是否是合法的整数、浮点数
	- 数字运算（套数加法、二进制加法）
	- 排序、交换（partition过程）
	- 字符计数(hash)： 变位词 
	- 匹配(正则表达式、全串匹配、KMP、周期判断)
	- 动态规划(LCS、编辑距离、最长回文子串)
	- 搜索(单词变换、排列组合)


例1 把一个0-1串进行排序，可以交换任意两个位置，问最少交换的次数
思路：快排partition 最左边0和最右边的1都可以不管
```
    public int exchangeTimes(String s){
        int answer = 0;
        for(int i = 0, j = s.length() - 1; i < j; i++, j--){
            for(; i < j && s.charAt(i) == '0'; i++);
            for(; i < j && s.charAt(j) == '1'; j--);
            if(i < j) answer++;
        }
        return answer;
    }
```

例2 删除一个字符串所有的a,并且复制所有的b.注:字符数组足够大
```
    public void solve(char[] chars){
        //先删除a,可以利用原来字符串的空间,过程类似插入排序
        int n = 0;//删除a后的字符数组长度
        int bCount = 0;
        for(int i = 0; s[i] != '\0'; i++){
            if(chars[i] == 'a'){
                chars[n++] = chars[i];
            }
            if(chars[i] == 'b'){
                bCount++;
            }
        }
        //从后往前复制
        //步骤1. 遍历计算新串长度,b出现的次数2. 从后往前遍历 3.旧串复制,如果遇到b,复制两遍
        int newLength = n + bCount;
        chars[newLength] = '\0';
      for(int i = n, j = newLength - 1; i >= 0; i--){
            chars[j--] = chars[i];
            if(chars[i] == 'b'){
                chars[j--] = 'b'; 
        }
    }
```
思考题:如何把字符串的空格变成"%20"
```
    public void replaceSpace(char[] chars){
        int length = 0;
        int spaceCount = 0;
        for(int i = 0; chars[i] != '\0'; i++){
            length++;
            if(chars[i] == ' '){
                spaceCount++;
            }
        }

        int newLength = length + 2 * spaceCount;
        chars[newLength] = '\0';
        for(int i = newLength - 1, j = length; j >= 0; j--){
            if(chars[j] == ' '){
                char[i] = '0';
                char[i - 1] = '2';
                char[i - 2] = '%';
                i = i - 3; 
            }else{
                char[i] = char[j];
                i = i -1;
            }
        }
    }
```

例3 一个字符串只包含*和数字，请把它的*号都放开头。
方法1 快排partition,因为过程不稳定,所以数字相对顺序会变化
```
    public void sortStar(char[] chars){
        for(int i = 0 , j = 0; j < a.length; j++){
            if(chars[i] == '*'){
                swap(a[i++], a[j])
            }
        }
    }

    private void swap(char[] chars, int i, int j){
        char temp = chars[i];
        chars[i] = chars[j];
        chars[j] = temp;
    }
```
方法2 倒着处理
```
     public void sortStar(char[] chars){
        int length = chars.length;
        int i = length - 1;
        for(int j = length - 1; j >= 0; j--){
            if(chars[j] != '*'){
                chars[i--] = chars[j];
            }
        }
        while(i >= 0){
            chars[i--] = '*';
        }
    }
```
    注意:以上两种解法的异同,方法1,用到交换和快排partition思想,但是无法保证数字的顺序;方法2,使用倒着向前处理的思想,如果必须用交换,只能采用方法1



