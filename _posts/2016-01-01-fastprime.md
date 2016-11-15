---
title: 快速素数测试
time: 2016.01.01 21:47:00
layout: post
catalog: true
tags:
- Security
- 算法
- 因式分解
excerpt: 快速素数测试的具体原理讲解




---

#快速素数测试

---

##前提知识
###1. 费马小定理
有一任意正整数N，P为素数，且N不能被P整除（显然N和P互质），则有：N^P%P=N(即：N的P次方除以P的余数是N)。
N^P%P=N＝>(N^P-N)%P=0=>(N＊(N^(P-1)-1))%P=0
=>(N^(P-1)-1))%P＝0 因为n p互质＝>
N^(P-1)%P=1,即 a(p-1)≡1（mod p）
证明：可见:[阮一峰先生写的证明](http://www.ruanyifeng.com/blog/2013/06/rsa_algorithm_part_one.html "阮一峰先生写的证明") 主要是先分5种情况，证明欧拉函数。然后再证明欧拉定理，而欧拉定理的一个特例就是费马小定理。
###2.积模分解公式
如果有：X%Z=0，即X能被Z整除，则有：(X+Y)%Z=Y%Z
设有X、Y和Z三个正整数，则必有：(X＊Y)%Z=((X%Z)＊(Y%Z))%Z
分情况讨论：
```c
1.当X和Y都比Z大时，必有整数A和B使下面的等式成立：
X=Z*I+A（1）
Y=Z*J+B（2）
将（1）和（2）代入(X＊Y)modZ得：((Z＊I+A)(Z＊J+B))%Z乘开，再把前三项的Z提一个出来，变形为：(Z*(Z*I*J+I*A+I*B)+A*B)%Z（3）
因为Z*(Z*I*J+I*A+I*B)是Z的整数倍……晕，又来了。
概据引理，（3）式可化简为：(A*B)%Z又因为：A=X%Z，B=Y%Z，代入上面的式子，就成了原式了。
2.当X比Z大而Y比Z小时，一样的转化：
X=Z*I+A
代入(X*Y)%Z得：
(Z*I*Y+A*Y)%Z
根据引理，转化得：(A*Y)%Z
因为A=X%Z，又因为Y=Y%Z，代入上式，即得到原式。
同理，当X比Z小而Y比Z大时，原式也成立。
3.当X比Z小，且Y也比Z小时，X=X%Z，Y=Y%Z，所以原式成立。
```

-----

##一些重要函数分析
###1. 蒙格马利算法快速求幂
```c
unsigned Power(unsigned n, unsigned p) 
{ // 计算n的p次方
    unsigned odd = 1; //oddk用来计算“剩下的”乘积
    while (p > 1)
    { // 一直计算到指数小于或等于1
       if (( p & 1 )!=0)
      { // 判断p是否奇数，偶数的最低位必为0
             odd *= n; // 若odd为奇数，则把“剩下的”乘起来
      }
      n *= n; // 主体乘方
      p /= 2; // 指数除以2
     }
    return n * odd; // 最后把主体和“剩下的”乘起来作为结果
}
```
转化成幂模算法
```c
//蒙格马利算法快速求幂 快速计算(n^p)%m的值
unsigned Montgomery(unsigned n,unsigned p,unsigned m)
{ //快速计算(n^p)%m的值
    unsigned k=1;
    if(m!=0)
        n%=m;
    else
        return -1;//error  m为0的状态
    while(p!=1)
    {
        if(0!=(p&1))// 如果指数p是奇数，则说明计算后会剩一个多余的数，那么在这里把它乘到结果中
            k=(k*n)%m;//把“剩下的”乘起来
        n=(n*n)%m;//主体乘方
        p>>=1;//p /= 2; // 指数除以2
    }
    return (n*k)%m;
}
```
###2.利用费马小定理进行素数判定
若一个数为素数，那么其与素数表中一个数，必满足费马小定理。所以，从概率上，我们可以通过素数表中的一群数来测试当前的数是否满足费马小定理。如果，它一直满足，则说明其很大概率上是一个素数。如果有一次没满足，则说明其为合数。
```c
int IsPrime(unsigned n)//判断是否是素数
{
    if ( n < 2 )
    {
        // 小于2的数即不是合数也不是素数
        return 0;
    }
	static unsigned aPrimeList[] = {
    	2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41,
   	 43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97
	};//素数表
    const int nListNum = sizeof(aPrimeList) / sizeof(unsigned);
    for (int i=0;i<nListNum;++i)
    {
        // 按照素数表中的数对当前素数进行判断
        if (1!=Montgomery(aPrimeList[i],n-1,n)&&n!=aPrimeList[i]) // 蒙格马利算法
        {
            return 0;
        }
    }
    return 1;
}
```
##现今最快的方法：拉宾米勒测试
拉宾米勒测试是一个不确定的算法，只能从概率意义上判定一个数可能是素数，但并不能确保。算法流程如下:
       1. 选择T个随机数A，并且有A<N成立。
       2. 找到R和M，使得N=2*R*M+1成立。
       快速得到R和M的方式：N用二进制数B来表示，令C=B-1。因为N为奇数（素数都是奇数），所以C的最低位为0，从C的最低位的0开始向高位统计，一直到遇到第一个1。这时0的个数即为R，M为B右移R位的值。
       3.如果A^M%N=1，则通过A对于N的测试，然后进行下一个A的测试
       4.如果A^M%N!=1，那么令i由0迭代至R，进行下面的测试
       5.如果A^((2^i)*M)%N=N-1则通过A对于N的测试，否则进行下一个i的测试 
       6.如果i=r，且尚未通过测试，则此A对于N的测试失败，说明N为合数。
       7.进行下一个A对N的测试，直到测试完指定个数的A
       通过验证得知，当T为素数，并且A是平均分布的随机数，那么测试有效率为1 / ( 4 ^ T )。如果T > 8那么测试失误的机率就会小于10^(-5)，这对于一般的应用是足够了。如果需要求的素数极大，或着要求更高的保障度，可以适当调高T的值。
       ```c
nt RabbinMillerTest( unsigned n )
{
    if (n<2)
    {
        // 小于2的数即不是合数也不是素数
        return 0;
    }
    
    for(int i=0;i<nPrimeListSize;++i)
    {
        // 按照素数表中的数对当前素数进行判断
        if (n/2+1<=aPrimeList[i])
        {
            // 如果已经小于当前素数表的数，则一定是素数
            return 1;
        }
        if (0==n%aPrimeList[i])
        {
            // 余数为0则说明一定不是素数
            return 0;
        }
    }
    // 找到r和m，使得n = 2^r * m + 1;
    int r = 0, m = n - 1; // ( n - 1 ) 一定是合数
    while ( 0 == ( m & 1 ) )
    {
        m >>= 1; // 右移一位
        r++; // 统计右移的次数
    }
    //const unsigned nTestCnt = 8; // 表示进行测试的次数
    for ( unsigned i = 0; i < nTestCnt; ++i )
    {
        // 利用随机数进行测试，
        int a = aPrimeList[ rand() % nPrimeListSize ];
        if ( 1 != Montgomery( a, m, n ) )
        {
            int j = 0;
            int e = 1;
            for ( ; j < r; ++j )
            {
                if ( n - 1 == Montgomery( a, m * e, n ) )
                {
                    break;
                }
                e <<= 1;
            }
            if (j == r)
            {
                return 0;
            }
        }
    }
    return 1;
}

```

----

# 参考：
1. [判断一个数是否为素数](http://blog.csdn.net/arvonzhang/article/details/8564836 "判断一个数是否为素数")
2. [欧拉函数](http://www.ruanyifeng.com/blog/2013/06/rsa_algorithm_part_one.html)
```