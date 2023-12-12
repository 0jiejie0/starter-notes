## 背景

前段时间在做接口对接时发现字符串分割的一点小问题，当时探明了用法，近来再用起来的时候，发现淡忘了，特此记录下来，然后进一步研究源码的写法及原因。

以此，作为阅读源码的开端。

jdk1.8

## 问题描述

某信息使用逗号,分隔code和name两数据，用```data.split(",")[0]```、```data.split(",")[1]```取对应code和name，正常来讲不会有什么问题。

然而非正常情况下，就极有可能触发bug。运行时出现了空指针异常，观察源码没有发现有什么问题。

打断点调试发现，在信息是`","`时，会报这个异常，我以为这里一定没问题的，因为我写这段代码前，测试过分割`""`，也能得到`[""]`。

甚至，用逗号分割多个数据，只取0号元素时，也会报错。

测试发现，是字符串中只有分隔符导致的这个问题。

于是用逗号作分隔符，进一步测试若干(注释表示处理结果)：

```java
        "".split(",");          // [""]
        ",".split(",");         // []
        ",,,,,".split(",");     // []
        ",,,,, ".split(",");    // ["","","","",""," "]
        ",, ,,,".split(",");    // ["",""," "]
        ",, ,,, ".split(",");   // ["",""," ","",""," "]
        ", ,,,,".split(",");    // [""," "]
        " ,,,,,".split(",");    // [" "]
```

## 分析

看上边的测试结果，看起来是split会舍弃右边的空字符串，应该是Java的鬼。查看源码，发现这个方法调用了`split(regex, 0)`：

### split

```java
    /**
     * Splits this string around matches of the given
     * <a href="../util/regex/Pattern.html#sum">regular expression</a>.
     *
     * <p> The array returned by this method contains each substring of this
     * string that is terminated by another substring that matches the given
     * expression or is terminated by the end of the string.  The substrings in
     * the array are in the order in which they occur in this string.  If the
     * expression does not match any part of the input then the resulting array
     * has just one element, namely this string.
     *
     * <p> When there is a positive-width match at the beginning of this
     * string then an empty leading substring is included at the beginning
     * of the resulting array. A zero-width match at the beginning however
     * never produces such empty leading substring.
     *
     * <p> The {@code limit} parameter controls the number of times the
     * pattern is applied and therefore affects the length of the resulting
     * array.  If the limit <i>n</i> is greater than zero then the pattern
     * will be applied at most <i>n</i>&nbsp;-&nbsp;1 times, the array's
     * length will be no greater than <i>n</i>, and the array's last entry
     * will contain all input beyond the last matched delimiter.  If <i>n</i>
     * is non-positive then the pattern will be applied as many times as
     * possible and the array can have any length.  If <i>n</i> is zero then
     * the pattern will be applied as many times as possible, the array can
     * have any length, and trailing empty strings will be discarded（这里其实就说明了，方法会丢弃后面的空字符串）.
     *
     * <p> The string {@code "boo:and:foo"}, for example, yields the
     * following results with these parameters:
     *
     * <blockquote><table cellpadding=1 cellspacing=0 summary="Split example showing regex, limit, and result">
     * <tr>
     *     <th>Regex</th>
     *     <th>Limit</th>
     *     <th>Result</th>
     * </tr>
     * <tr><td align=center>:</td>
     *     <td align=center>2</td>
     *     <td>{@code { "boo", "and:foo" }}</td></tr>
     * <tr><td align=center>:</td>
     *     <td align=center>5</td>
     *     <td>{@code { "boo", "and", "foo" }}</td></tr>
     * <tr><td align=center>:</td>
     *     <td align=center>-2</td>
     *     <td>{@code { "boo", "and", "foo" }}</td></tr>
     * <tr><td align=center>o</td>
     *     <td align=center>5</td>
     *     <td>{@code { "b", "", ":and:f", "", "" }}</td></tr>
     * <tr><td align=center>o</td>
     *     <td align=center>-2</td>
     *     <td>{@code { "b", "", ":and:f", "", "" }}</td></tr>
     * <tr><td align=center>o</td>
     *     <td align=center>0</td>
     *     <td>{@code { "b", "", ":and:f" }}</td></tr>
     * </table></blockquote>
     *
     * <p> An invocation of this method of the form
     * <i>str.</i>{@code split(}<i>regex</i>{@code ,}&nbsp;<i>n</i>{@code )}
     * yields the same result as the expression
     *
     * <blockquote>
     * <code>
     * {@link java.util.regex.Pattern}.{@link
     * java.util.regex.Pattern#compile compile}(<i>regex</i>).{@link
     * java.util.regex.Pattern#split(java.lang.CharSequence,int) split}(<i>str</i>,&nbsp;<i>n</i>)
     * </code>
     * </blockquote>
     *
     *
     * @param  regex
     *         the delimiting regular expression
     *
     * @param  limit
     *         the result threshold, as described above
     *
     * @return  the array of strings computed by splitting this string
     *          around matches of the given regular expression
     *
     * @throws  PatternSyntaxException
     *          if the regular expression's syntax is invalid
     *
     * @see java.util.regex.Pattern
     *
     * @since 1.4
     * @spec JSR-51
     */
    public String[] split(String regex, int limit) {
        /* fastpath if the regex is a
         (1)one-char String and this character is not one of the
            RegEx's meta characters ".$|()[{^?*+\\", or
         (2)two-char String and the first char is the backslash and
            the second is not the ascii digit or ascii letter.
         */
        char ch = 0;
        if (((regex.value.length == 1 &&
             ".$|()[{^?*+\\".indexOf(ch = regex.charAt(0)) == -1) ||  // 分隔符是一个普通字符，或者
             (regex.length() == 2 &&
              regex.charAt(0) == '\\' &&  // 包含对正则转义的\，和一个字符
              (((ch = regex.charAt(1))-'0')|('9'-ch)) < 0 &&  // 负负相或为负（位或），有负数说明字符小于0或大于9，第二个字符不在数字范围
              ((ch-'a')|('z'-ch)) < 0 &&  // 不在小写字母范围，第二个字符不是小写字母
              ((ch-'A')|('Z'-ch)) < 0)) &&  // 分隔符是一个普通字符或者是一个加了反斜杠前缀的符号，并且
            (ch < Character.MIN_HIGH_SURROGATE ||
             ch > Character.MAX_LOW_SURROGATE)) // 并且这是一个字符 （Unicode的代理区 0xD800-0xDFFF，其内不代表任何字符）  
        { // 如果这分隔表达式是个字符，并且是 一个普通字符 或是 反斜杠加一个特殊字符
            // 实际上是指仅一个分隔符的情况，和上方的英文源码注释同义
            int off = 0;
            int next = 0;
            boolean limited = limit > 0; // 看到这里就能明白，为什么0和负数都不会限制数组长度了
            ArrayList<String> list = new ArrayList<>();
            while ((next = indexOf(ch, off)) != -1) { // next定位到每个分隔符，循环执行到待切串最后一个分隔符
                if (!limited || list.size() < limit - 1) { // 如果无数组长度限制或不是限制长度下的最后一个元素
                    list.add(substring(off, next)); // substring 原理见下
                    off = next + 1; // 截取字符串，把指针挪动到当前位置
                } else {    // last one // 限制数组长度情况下，最末元素（同这个英文单词，又同下释代码）
                    //assert (list.size() == limit - 1);
                    list.add(substring(off, value.length)); // 截取剩余串，作为最末元素
                    off = value.length; // 指针标记结束，标志所有串切割完成（不缩短长度的情况下，上个if内无法处理最末元素，因为最末元素没有分隔符不满足循环条件，也就不能完成切割）
                    break;
                }
            }
            // If no match was found, return this // 无分隔符
            if (off == 0)
                return new String[]{this};

            // Add remaining segment // 补充 不限制长度的情况下 或 限制长度和应截长度相等的情况下 的，最末元素（因为上个循环控制条件时有无分隔符，无法处理不含分隔符的最末元素）。
            if (!limited || list.size() < limit)
                list.add(substring(off, value.length));

            // Construct result // 返回值生成
            int resultSize = list.size();
            if (limit == 0) { // 仅在0 不限制结果长度的情况下，用指针标记非空串末尾
                while (resultSize > 0 && list.get(resultSize - 1).length() == 0) {
                    resultSize--; // 数组末尾指针位置为空串时，指针前移，这是本篇笔记的缘由
                }
            }
            String[] result = new String[resultSize]; // 列表截取结果放到此数组中
            return list.subList(0, resultSize).toArray(result);
        }
        return Pattern.compile(regex).split(this, limit);
    }
```

在源码方法注释的第四段中，最后一句话就说明了这一点，方法会丢弃后面的空字符串。ai助手解释说这可以简化对返回结果的处理。
例如：`"apple,banana,orange,"`，丢弃最后的空字符串可以避免对空字符串作单独的逻辑判断（这说明以后再用时，可以放心地往后写分隔符）。

其中调用的substring()方法，底层调用了String(char value[], int offset, int count)，其源码行数不多，但有点费解：

### subString 之 String(char[],int,int)

```java
    /**
     * Allocates a new {@code String} that contains characters from a subarray
     * of the character array argument.（用参数中的数组新生成一个字符串） The {@code offset} argument is the
     * index of the first character of the subarray and the {@code count}
     * argument specifies the length of the subarray.（offset 是第一个字符位置， count 是生成字符串长度） The contents of the
     * subarray are copied; subsequent modification of the character array does
     * not affect the newly created string.（复制子数组，其后续修改不影响新串）
     *
     * @param  value
     *         Array that is the source of characters
     *
     * @param  offset
     *         The initial offset
     *
     * @param  count
     *         The length
     *
     * @throws  IndexOutOfBoundsException
     *          If the {@code offset} and {@code count} arguments index
     *          characters outside the bounds of the {@code value} array
     */
    public String(char value[], int offset, int count) {
        if (offset < 0) { // 起始截取点 非法
            throw new StringIndexOutOfBoundsException(offset); // 非法偏移量
        }
        if (count <= 0) {
            if (count < 0) { // 截取长度 非法
                throw new StringIndexOutOfBoundsException(count); // 非法长度
            }
            // count = 0
            if (offset <= value.length) { // 为什么这里要校验起始点，而不直接生成空串？
                // 其实，这是为把不同异常分别归类，把结束点异常剔除到下面的if，这里明明是起始点，为什么我要说这是结束点呢？
                // 因为，原本这里是 offset + count <= len ，因为此时count确定为0，所以条件写成这样
                this.value = "".value;
                return;
            }
        }
        // 此时 offset count length 均>0，只剩一种异常情况
        // Note: offset or count might be near -1>>>1. // -1 二进制表示为FFFF FFFF,无符号右移得到7FFF FFFF，提示offset或count可能接近最大正整数，这行注释是提示维护人下面表达式可能溢出的情况
        if (offset > value.length - count) { // 校验 串结束点，原型是 offset + count > len，用减法是为避免 offset + count 过大溢出
            throw new StringIndexOutOfBoundsException(offset + count); // 非法结束点
        }
        this.value = Arrays.copyOfRange(value, offset, offset+count);
    }
```

回顾这段代码，将不同异常分流显示，起始点>0即合法，长度>0即合法，最后把二者加和，判断结束点是否合法。
这之中，表达式有些许变化，提示我们在阅读代码甚至是编写代码时，需要把各种条件判定和表达式计算揉合到一起，以减少执行代码量。

##### 但还不够，count=0单独处理比较好理解，有一点我比较疑惑，就是这个返回空串的代码位置。

* 如果仅仅是想把count=0单独处理，为什么这个if要和count<0拼到一起拼成<=0，而不是执行offset<0,count<0,offset+count>len，然后再判定count=0呢？
* 如果是为了把count=0的情况提前处理，那为什么不提到最前，写成count=0&&offset<len,offset<0,count<0,offset+count>len 呢？

 <= 和 < 的机器效率是一样的。
其实可以看到，对于count>0和count=0的两种路径而言，j8的这个结构对count=0没有太大优化，但对于count>0路径，却少了一个if判断。

* 可能这里用count<=0的写法，是为了提前分流，能够更早处理主要逻辑——count>0，同时没有延迟对count=0的处理，

流程图中对一个程序的结束位置，总是归于一个点，如果切掉这个结束点，把它看成一个树，就能发现 count>0 路径确实少了一个分叉。

### jdk7 jdk8的差异

在[一篇关于j7对j6substring内存泄漏优化的博客](https://blog.csdn.net/u013008898/article/details/117163823?ops_request_misc=&request_id=&biz_id=102&utm_term=//%20Note:%20offset%20or%20count%20might&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-2-117163823.142^v96^pc_search_result_base5&spm=1018.2226.3001.4187)，
我看到了这个方法的j7实现，jdk1.7对这个方法的实现是这样的：

```java
    public String(char value[], int offset, int count) {
        if (offset < 0) {
            throw new StringIndexOutOfBoundsException(offset);
        }
        if (count < 0) {
            throw new StringIndexOutOfBoundsException(count);
        }
        // Note: offset or count might be near -1>>>1.
        if (offset > value.length - count) {
            throw new StringIndexOutOfBoundsException(offset + count);
        }
        this.value = Arrays.copyOfRange(value, offset, offset+count);
    }
```

j7的处理过程显然更容易理解，所以j8的写法肯定是有原因的。

不幸的是，[有篇博客](https://blog.csdn.net/weixin_35955473/article/details/114575389?ops_request_misc=&request_id=&biz_id=102&utm_term=String(char%20value%5B%5D,%20int%20offse&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-2-114575389.142^v96^pc_search_result_base5&spm=1018.2226.3001.4187 )
说，split的性能不太好（不太理解，可能是因为要处理正则逻辑？），可以用java专门切割字符串的StringTokenizer类，效率更高。
或者可以用 indexOf+subString 切割字符串，效率更高，另外还有个反方法charAt。（我这里应该用代码实际测试一下）

[//]: # (todo 代码测试实验 split 和 subString 性能对比)

今天发现，两个月前我亲手写的代码，已经忘了是怎么回事，忘了是怎么写的了。

再没搜到有用的文章，有[一篇博客](https://blog.csdn.net/ningmengshuxiawo/article/details/115247532?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522170200761716800222829098%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=170200761716800222829098&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-2-115247532-null-null.142^v96^pc_search_result_base5&utm_term=public%20String%28char%20value%5B%5D%2C%20int%20offset%2C%20int%20count%29%20%7B%20%20%20%20%20%20%20%20%20if%20%28offset%20%3C%200%29%20%7B%20%2F%2F%20%E8%B5%B7%E5%A7%8B%E6%88%AA%E5%8F%96%E7%82%B9%20%E9%9D%9E%E6%B3%95%20%20%20%20%20%20%20%20%20%20%20%20%20throw%20new%20StringIndexOutOfBoundsException%28offset%29%3B%20%2F%2F%20%E9%9D%9E%E6%B3%95%E5%81%8F%E7%A7%BB%E9%87%8F%20%20%20%20%20%20%20%20%20%7D%20%20%20%20%20%20%20%20%20if%20%28count%20%3C%3D%200%29%20%7B%20%20&spm=1018.2226.3001.4187)，
详细解读了 subString 源码，但是也没能解答我[这个疑惑](#但还不够count0单独处理比较好理解有一点我比较疑惑就是这个返回空串的代码位置)。

## 用例说明

对元素数量固定多于某数的待分割数据，分割前在原数据后追加一个空格，然后分割，最后用trim去掉空格即可（不看源码的处理方式）。

```java
Utils.getString(data+" ").split(",")[0].trim()
```

这样，无论原数据如何，都不会触发异常。

又或者，在已知元素数量大于等于某数的情况下，split的第二个参数填写正数，如果不知道数量，可以用负数，即可避免舍弃末尾空字符串的情况。
split第二个可选参数表示分割数量，填0、负数或不填表示不限制元素数量，例（在源码注释中已经有示例了，）：

```java
        "3,,".split(",",6);         // ["3","",""]
        "3,,".split(",",-1);        // ["3","",""]
        "3,,".split(",",1);         // ["3,,"]
        "3,,".split(",",0);         // ["3"]
        "3,5,6,4, ".split(",",3);   // ["3","5","6,4, "]
        "3,5,6,4, ".split(",");     // ["3","5","6","4"," "]
```
