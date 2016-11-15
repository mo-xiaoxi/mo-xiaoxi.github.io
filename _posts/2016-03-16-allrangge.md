---
title: 有重复元素的排列问题和排列元素问题
time: 2016.03.16 21:47:00
layout: post
catalog: true
tags:
- 算法
- C++
- 学习总结
excerpt: 总结下有重复元素的排列问题和排列元素问题，主要体现思考过程
    



---

#有重复元素的排列问题
##简介：
举例：1 1 9 9 

排列为：
​	
	1 1 9 9
	1 9 1 9
	1 9 9 1
	9 1 1 9
	9 1 9 1
	9 9 1 1
题目很容易理解，所以我也不多做叙述。（还是不懂的，可以直接google有重复元素的排列问题）

## 思路：
1. 面对这种题，我们首先考虑：无重复元素的排列问题，即全排列问题。即对于abc三个元素，可以排列为abc、acb、bac、bca、cab 和 cba。
2. 显然，全排列问题可以通过两次暴力循环，去重解决的。但是，那种方法太不温柔了，所以我们要考虑一个更加温柔的算法。
3. 考虑bac和cba这二个字符串是如何得出的。显然这二个都是abc中的 a 与后面两字符交换得到的。然后可以将abc的第二个字符和第三个字符交换得到acb。同理可以根据bac和cba来得bca和cab。
4. 因此可以知道**全排列就是从第一个数字起每个数分别与它后面的数字交换，**也可以得出这种解法每次得到的结果都是正确结果，所以复杂度为 O(n!)。
5. 而从全排列过渡到有重复元素全排列，只需要在进行变换时，对变换结果判定结果的唯一性。
6. 对于abb，其能得到bab和bba，而bab又能得到bba，那我们能不能第一个bba不求呢？ 我们有了这种思路，第一个字符a与第二个字符b交换得到bab，然后考虑第一个字符a与第三个字符b交换，此时由于第三个字符等于第二个字符，所以它们不再交换。再考虑bab，它的第二个与第三个字符交换可以得到bba。此时全排列生成完毕，即abb、bab、bba三个。
7. 这样我们也得到了在全排列中去掉重复的规则：去重的全排列就是从第一个数字起每个数分别与它后面非重复出现的数字交换。用编程的话描述就是第i个数与第j个数交换时，要求[i,j)中没有与第j个数相等的数。

## 代码：
```
//
//  main.cpp
//  2_9
//
//  Created by moxiaoxi on 16/3/15.
//  Copyright © 2016年 moxiaoxi. All rights reserved.
//
#include <stdio.h>
#include<iostream>
using namespace std;
int Count=0;

void swap(char &m,char &n)
{
    char tmp=m;
    m=n;
    n=tmp;
    
}


// 字符串匹配
bool Findsame(char list[], int k, int m){
    
    // list[m]是否有在list[k, m-1]出现过
    for (int i = k; i < m; i++) {
        if (list[m] == list[i]) {
            // 出现了
            return true;
        }
    }
    
    return false;
}


// 全排列，递归
void Perm(FILE *fp,char list[], int k, int m){
    if (k == m) {
        // 输出排列
        for (int i = 0; i <= m; i++) {
            fprintf(fp,"%c",list[i]);
        }
        Count++;// 排列数+1
        fprintf(fp,"\n");
    }else{
        
        for (int i = k; i <= m; i++) {
            if (Findsame(list, k, i)) {
                continue;//去除已出现过的排列
            }
            
            swap(list[k], list[i]);
            Perm(fp,list, k + 1, m);// 递归
            swap(list[k], list[i]);
        }
    }
}


int main(int argc, const char * argv[]) {
    int n;
    char list[100];
    FILE *fp,*fpout;
    //    if(argc!=3){
    //        printf("have not enter two file name strike any key exit");
    //        getchar();
    //        return -1;
    //    }
    //
    //    if((fp=fopen(argv[1],"r"))==NULL){
    //        printf("\nCannot open input file strike any key exit!");
    //        getchar();
    //        return -1;
    //    }
    //    if((fpout=fopen(argv[2],"w"))==NULL){
    //        printf("\nCannot open output file strike any key exit!");
    //        getchar();
    //        return -1;
    //    }
    if((fp=fopen("input.txt","r"))==NULL){
        printf("\nCannot open input file strike any key exit!");
        getchar();
        return -1;
    }
    if((fpout=fopen("output.txt","w"))==NULL){
        printf("\nCannot open output file strike any key exit!");
        getchar();
        return -1;
    }
    fscanf(fp,"%d",&n);
    fscanf(fp,"%s" ,list);
    Perm(fpout,list,0,n-1);
    fprintf(fpout,"%d",Count);
    return 0;
}

```