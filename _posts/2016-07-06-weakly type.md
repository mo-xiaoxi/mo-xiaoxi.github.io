---
layout: post
category: note
title: PHP中弱类型安全总结
excerpt: 在PHP审计中有一类很常见的安全绕过漏洞，即弱类型转化导致的安全绕过（这里只讲PHP的，其它的弱类型语言大家可以引申地学习）
first_time: 2016.07.06 15:50:00
time: 2016.07.06 17:13:09
catalog: true
tags:
- Security
- PHP
- Weakly type



---

# PHP中弱类型安全总结

PHP语言中有一个很好的特性，弱类型。php内核的开发者原本是想让程序员借由这种不需要声明的体系，更加高效的开发，所以在几乎所有内置函数以及基本结构中使用了很多松散的比较和转换，防止程序中的变量因为程序员的不规范而频繁的报错，然而这却带来了安全问题。

-------

## 预备知识

### 1. 弱类型内核实现

php中弱类型是通过zval结构实现的。

在PHP中声明的变量，在ZE中都是用结构体zval来保存的

zval的定义在zend/zend.h

```
typedef struct _zval_struct zval;  
 
struct _zval_struct {  
    /* Variable information */ 
    zvalue_value value;     /* value */ 
    zend_uint refcount__gc;  
    zend_uchar type;    /* active type */ 
    zend_uchar is_ref__gc;  
};  
 
typedef union _zvalue_value {  
    long lval;  /* long value */ 
    double dval;    /* double value */ 
    struct {  
        char *val;  
        int len;  
    } str;  
    HashTable *ht;  /* hash table value */ 
    zend_object_value obj;  
} zvalue_value;
```

其中type来判定变量类型

### 2. intval函数

在一些涉及到int的函数对比中，常常需要使用intval函数来进行转化。

#### 定义和用法

获取变量的整数值，允许以使用特定的进制返回。默认10进制

注:如果参数为整数，则不做任何处理。

#### 语法

intval (var, base)

| 参数   |                   描述                   |
| :--- | :------------------------------------: |
| var  | 必须。可以是任何标量类型。 intval() 不能用于数组 或 对象(类)。 |
| base |           可选。转化所使用的进制，默认10进制           |

如果 base 是 0，通过检测 var 参数的格式来决定使用的进制：

如果字符串包括了 "0x" (或 "0X") 的前缀，使用 16 进制

如果字符串以 "0" 开始，使用 8 进制

其他使用 10 进制

eg:

```
<?php 
echo intval("0x1a", 0), "\n"; // 使用16进制。 结果 "26" 
echo intval("057", 0), "\n"; // 使用8进制。 结果 "47" 
echo intval("57"),"\n"; // 使用10进制。结果57
echo intval("42", 0), "\n"; //  结果 "42" 
?>
```

### 3. in_array()

#### 定义和用法

in_array() 函数搜索数组中是否存在指定的值。

注释：如果 search 参数是字符串且 type 参数被设置为 TRUE，则搜索区分大小写。

#### 语法

in_array(search,array,type)

| 参数     |                   描述                   |
| :----- | :------------------------------------: |
| search |             必需。规定要在数组搜索的值。             |
| array  |              必需。规定要搜索的数组。              |
| type   | 可选。如果设置该参数为 true，则检查搜索的数据与数组的值的类型是否相同。 |

------

## 变量的强制转换
### 1. int遇上string

#### eg1 数学运算

当php进行一些数学计算的时候

```
var_dump(0 == '0'); // true
var_dump(0 == 'abcdefg'); // true  
var_dump(0 === 'abcdefg'); // false
var_dump(1 == '1abcdef'); // true 
```

当有一个对比参数是整型的时候，会把另外一个参数强制转换为整数。（转换的时候会使用intval函数）

值得一提的是，这里的intval函数是从字符串的第一个非数字开始的。即1231asd会转化为1231，asd转换为0，1adas转化为1，如此类推

所以下面的语句就很容易绕过

```
if($a>1000){
    mysql_query('update ... .... set value=$a')
}
```
 这里开发者会认为进入mysql_query()中的$a一定是整数。然而，你输入1002/**/union 也能进入这个语句


####  eg2 函数的松散判断


var_dump(in_array("abc", $array1));</br>
var_dump(in_array("1bc", $array2));

对于上面的语句，如果$array1=array(0),$array2=array(1),语句会返回true

![image](https://moxiaoxi.info/img/post/weak1.png)

同理，array_search也是一样的道理

我们可以构造int 0 或者1 来绕过检查函数。


总结一下，*在所有php认为是int的地方输入string，都会被强制转换*，比如

```
$a = 'asdfgh'；//字符串类型的a</br>
echo $a[2]；  //根据php的offset 会输出'd'</br>
echo $a[x]；  //根据php的预测，这里应该是int型，那么输入string，就会被intval成为0 也就是输出'a'
```

###  2.当数组遇上string

由php手册我们知道

Array转换整型int/浮点型float会返回元素个数；

转换bool返回Array中是否有元素；转换成string返回'Array'，并抛出warning。

看下面的代码：

```
if(!strcmp($c[1],$d) && $c[1]!==$d){
...
}
```

我们可以构造下列输入：

```
http://localhost:8888/1.php?a[]=1
var_dump(strcmp($_GET[a],'a'));
```
这样strcmp就为null，即能绕过检查函数


弱类型对比图：
![image](http://static.wooyun.org/drops/20150103/2015010315472853884E2F88EF5-B7BA-426E-9707-3CACF59C19F1.jpg)

---

###  参考：
1. http://drops.wooyun.org/tips/4483