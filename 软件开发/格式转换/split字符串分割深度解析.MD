## 背景

前段时间在做接口对接时发现字符串分割的一点小问题，当时探明了用法，近来再用起来的时候，发现淡忘了，特此记录下来，然后进一步研究源码的写法及原因。
以此，作为阅读源码的开端。

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
            boolean limited = limit > 0;
            ArrayList<String> list = new ArrayList<>();
            while ((next = indexOf(ch, off)) != -1) {
                if (!limited || list.size() < limit - 1) {
                    list.add(substring(off, next));
                    off = next + 1;
                } else {    // last one
                    //assert (list.size() == limit - 1);
                    list.add(substring(off, value.length));
                    off = value.length;
                    break;
                }
            }
            // If no match was found, return this
            if (off == 0)
                return new String[]{this};

            // Add remaining segment
            if (!limited || list.size() < limit)
                list.add(substring(off, value.length));

            // Construct result
            int resultSize = list.size();
            if (limit == 0) {
                while (resultSize > 0 && list.get(resultSize - 1).length() == 0) {
                    resultSize--;
                }
            }
            String[] result = new String[resultSize];
            return list.subList(0, resultSize).toArray(result);
        }
        return Pattern.compile(regex).split(this, limit);
    }
```
在源码方法注释的第四段中，最后一句话就说明了这一点，方法会丢弃后面的空字符串。ai助手解释说这可以简化对返回结果的处理。
例如：`"apple,banana,orange,"`，丢弃最后的空字符串可以避免对空字符串作单独的逻辑判断（这说明以后再用时，可以放心地往后写分隔符）。

## 用例说明

对元素数量固定多于某数的待分割数据，分割前在原数据后追加一个空格，然后分割，最后用trim去掉空格即可（不看源码的处理方式）。

```java
Utils.getString(data+" ").split(",")[0].trim()
```

这样，无论原数据如何，都不会触发异常。

又或者，在已知元素数量大于等于某数的情况下，split的第二个参数填写正数，如果不知道数量，可以用负数，即可避免舍弃末尾空字符串的情况。
split第二个可选参数表示分割数量，填0、负数或不填表示不限制元素数量，例（在源码注释中已经有示例了，）：

```java
        "3,,".split(",",6);   // ["3","",""]
        "3,,".split(",",-1);   // ["3","",""]
        "3,,".split(",",1);   // ["3,,"]
        "3,,".split(",",0);   // ["3"]
        "3,5,6,4, ".split(",",3);   // ["3","5","6,4, "]
        "3,5,6,4, ".split(",");     // ["3","5","6","4"," "]
```
