## 背景

Java代码中用字符串切换数据处理方式，switch和if各自的适用场景记不太清了，搜索到[这篇博客](https://blog.csdn.net/qq_40776361/article/details/123188112)
，重温一下。

## 正文

### if写法：

```java
        if(boolean){

        }else if(boolean){

        }else if(boolean){

        }else{

        }
```

### switch写法：

```java
        switch(表达式){
        case 值1:语句1 break;
        case 值2:语句2 break;
        ...
default :语句n break;
        }
```

### 二者区别

1. 分支较多时，switch效率高。switch为随机访问，确定条件值后直接跳转特定分支，但if要依次遍历各条件，才能找到合适分支。
2. switch以空间换时间。由汇编代码可知，switch要生成跳表，占空间多，特别是case常量分布范围很大，但实际值少的情况下，switch空间利用率会大大变低。
3. if的条件灵活，适用场景多。switch条件只能是常量，无法使用非常量。例如 if (a > 1 && a < 100)，无法用switch处理。
