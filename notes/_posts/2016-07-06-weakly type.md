---
layout: post
category: note
title: PHP中弱类型安全总结
excerpt: 在PHP审计中有一类很常见的安全绕过漏洞，即弱类型转化导致的安全绕过（这里只讲PHP的，其它的弱类型语言大家可以引申地学习）
first_time: 2016.07.06 15:50:00
time: 2016.07.06 17:13:09
tags:
- Security
- PHP
- Weakly type

---

# PHP中弱类型安全总结

PHP语言中有一个很好的特性，弱类型。php内核的开发者原本是想让程序员借由这种不需要声明的体系，更加高效的开发，所以在几乎所有内置函数以及基本结构中使用了很多松散的比较和转换，防止程序中的变量因为程序员的不规范而频繁的报错，然而这却带来了安全问题。

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

--- 

###  参考：
1. http://drops.wooyun.org/tips/4483