# like 模糊查询的底层算法

    全文搜索算法、模糊查询、n-gram分隔算法

## 功能介绍

百度搜索，文心一言给出的结果：

    SQL模糊查询底层通常使用全文搜索算法，如LIKE操作符和全文索引通常使用的n-gram分割算法。
    n-gram是一种将文本分割成固定大小的词组的算法，通常用于文本搜索和自然语言处理。在SQL中，全文索引通常以n-gram的方式工作，其中n取决于索引的granularity设置。

再次搜索，找到[一篇博客](https://blog.csdn.net/qq_29864051/article/details/125987138)，部分内容是这么说的：

ngram是全文解析器能够对文本进行分词，中文分词用 ngram_token_size 设定分词的大小,ngram_token_size 的值就是连续n个字的序列。

示例：使用ngram对于‘全文索引进行分词’：

```
ngram_token_size =1,分词为 ‘全‘，’文‘，’索‘，’引‘
ngram_token_size =2,分词为 ‘全文‘，’文索‘，’索引‘
ngram_token_size =3,分词为 ‘全文索‘，’文索引‘
ngram_token_size =4,分词为 ‘全文索引‘
```

查看配置ngram_token_size：

```
#查看默认分词大小 ngram_token_size=2
show variables like '%token%';
```

**innodb_ft_min_token_size**
默认3，表示最小3个字符作为一个关键词，增大该值可减少全文索引的大小

**innodb_ft_max_token_size**
默认84，表示最大84个字符作为一个关键词，限制该值可减少全文索引的大小

**ngram_token_size**
默认2，表示2个字符作为内置分词解析器的一个关键词，如对“abcd”建立全文索引，关键词为’ab’，‘bc’，‘cd’

当使用ngram分词解析器时，innodb_ft_min_token_size和innodb_ft_max_token_size 无效

创建全文索引的方法移步[刚才的博客](https://blog.csdn.net/qq_29864051/article/details/125987138)。

## 全文搜索算法 原理简介

### 起由

关于全文搜索算法，我找到了这样一篇[厉害的博客](https://www.cnblogs.com/forfuture1978/archive/2009/12/14/1623594.html)，
09年的，非常好，可惜没有作者13 14年后文章了。
以下内容源于我对这篇博客的理解，恭疏短引：

全文搜索算法出现的原因：对于大量 不定长或无固定格式 的非结构化数据，顺序扫描法效率很低。

优化思路：由于结构化数据有一定的结构可以采取一定的搜索算法加快速度，
所以，我们将非结构化数据中的一部分信息提取出来，重新组织，使其变得有一定结构，这个重新组织的 有一定结构的信息
称为索引，然后对索引搜索，从而实现加速目的。

这个先建立索引，再对索引进行搜索的过程就叫全文检索(Full-text Search)。

### 原理

#### 索引中的内容

从字符串到文件的映射是文件到字符串映射的反向过程，于是保存这种信息的索引称为反向索引。

建立一个词典，词典中每个字符串都指向包含此字符串的文档(Document)链表，此文档链表称为倒排表(Posting List)。

全文搜索相对于顺序扫描的优势之一：一次索引，多次使用。

#### 如何创建索引

##### 1. 为了方便说明索引创建过程，这里特意用两个文件为例：

文件一：Students should be allowed to go out with their friends, but not allowed to drink beer.

文件二：My friend Jerry went to school to see his students but found them drunk which is not allowed.

##### 2. 分词(Tokenizer)

1. 将文档分成一个一个单独的单词。
2. 去除标点符号。
3. 去除停词(Stop word)。

经过分词(Tokenizer)后得到的结果称为词元(Token)：
“Students”，“allowed”，“go”，“their”，“friends”，“allowed”，“drink”，“beer”，“My”，“friend”，“Jerry”，“went”，“school”，“see”，“his”，“students”，“found”，“them”，“drunk”，“allowed”。

##### 3. 词元(Token)传给语言处理组件(Linguistic Processor)

1. 变为小写(Lowercase)。
2. 将单词缩减为词根形式，如“cars”到“car”等。这种操作称为：stemming。
3. 将单词转变为词根形式，如“drove”到“drive”等。这种操作称为：lemmatization。

语言处理组件(linguistic processor)的结果称为词(Term)：
“student”，“allow”，“go”，“their”，“friend”，“allow”，“drink”，“beer”，“my”，“friend”，“jerry”，“go”，“school”，“see”，“his”，“student”，“find”，“them”，“drink”，“allow”。

##### 4. 词(Term)传给索引组件(Indexer)

（示例见原文）

1. 利用得到的词(Term)创建一个字典，包含字符串和文档id
2. 对字典按字母顺序进行排序
3. 合并相同的词(Term)成为文档倒排(Posting List)链表

表中，有几个定义：

Document Frequency 即文档频次，表示总共有多少文件包含此词(Term)。

Frequency 即词频率，表示此文件中包含了几个此词(Term)。

如果搜索“driving”，查询语句经过这里的一到三步，变为“drive”，从而可以搜索到想要的文档。

#### 如何搜索索引

经过上面的处理，搜索到的结果可能非常多，我们可能找不到最想看的文章。

##### 1. 输入查询语句

##### 2. 对查询语句进行词法分析，语法分析，及语言处理

##### 3. 搜索索引，得到符合语法树的文档

搜索索引分几小步：

1. 首先，在反向索引表中，分别找出包含lucene，learn，hadoop的文档链表。
2. 其次，对包含lucene，learn的链表进行合并操作，得到既包含lucene又包含learn的文档链表。
3. 然后，将此链表与hadoop的文档链表进行差操作，去除包含hadoop的文档，从而得到既包含lucene又包含learn而且不包含hadoop的文档链表。
4. 此文档链表就是我们要找的文档。

##### 4. 根据得到的文档和查询语句的相关性，对结果进行排序

1. 首先，一个文档有很多词(Term)
2. 对于文档之间的关系，不同的Term重要性不同。
    1. 找出词(Term)对文档的重要性的过程称为计算词的权重(Term weight)的过程。
    2. 影响权重的因素：
       Term Frequency (tf)：即此Term在此文档中出现了多少次。tf 越大说明越重要。
       Document Frequency (df)：即有多少文档包含次Term。df 越大说明越不重要。
    3. 权重 w = tf x log (n / df) 其中，n为该词在所有文档中出现的总次数
3. 判断Term之间的关系从而得到文档相关性，使用向量空间模型的算法(VSM)
    1. 把文档看作一系列词(Term)，每一个词(Term)都有一个权重(Term weight)，不同的词(Term)根据自己在文档中的权重来影响文档相关性的打分计算。
    2. 把此文档中各个词(term)的权重(term weight) 看作一个向量。
        1. Document = {term1, term2, …… ,term N}
        2. Document Vector = {weight1, weight2, …… ,weight N}
    3. 把查询语句看作一个简单的文档，也用向量来表示。
        1. Query = {term1, term 2, …… , term N}
        2. Query Vector = {weight1, weight2, …… , weight N}
    4. 把所有搜索出的文档向量及查询向量放到一个N维空间中，每个词(term)是一维。
        1. 维数不同时，取二者的并集，如果不含某个词(Term)，则权重(Term Weight)为0。
    5. 计算夹角的余弦值作为相关性的打分，夹角越小，余弦值越大，打分越高，相关性越大。

通读一遍，感觉还是原文写得好，我缩减的不够深入浅出，要点连成片反而抓不到要点了。。。

文末，另附[Lucene原理与代码分析完整版](https://www.cnblogs.com/forfuture1978/archive/2010/06/13/1757479.html)
