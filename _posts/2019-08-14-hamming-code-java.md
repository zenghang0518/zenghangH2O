---
layout: post
title: 'java实现海明码算法'
date: 2019-08-14
tags: JAVA 算法
---

最近老板给安排了一个需求，是关于生成优惠券的。

需求文档看着挺多，实际上就三个要求：

> 由数字字母组成的八位字符串
>
> 不能有重复
>
> 不能过于规律，避免被仿造

## 分析

看到这个需求时，我第一个想法就是使用一个序列，然后用一个算法去加密它，生成一个新的字符串，这样就很难仿造了。

但是我找了不少成熟的加密算法，但是没有一个满足现在的需求。

一是因为加密之后的长度超出八位，二是因为优惠券在使用前需要有一个快捷的校验真假的手段（如果每一张优惠券都去数据库校验真假，一是效率低，二是容易被攻击），而现在大部分加密算法是针对密码的，没有这种校验手段。

这样的话就需要自己去实现一个加密的算法，那么问题又来了，怎么解决加密之后的碰撞问题？要知道以前流行的MD5加密算法就是因为密码碰撞才被淘汰的。

思来想去，突然想起以前上学的时候老师讲过的海明码。

> 汉明码在传输的消息流中插入验证码，当计算机存储或移动数据时，可能会产生数据位错误，以侦测并更正单一比特错误。由于汉明编码简单，它们被广泛应用于内存（RAM）。

原理简单，还可以校验错误，非常符合优惠券的需求！

## 原理

这里有一篇博客讲的很详细：[海明码（汉明码、Hamming Code）](https://blog.csdn.net/Never_Satisfied/article/details/81988928)

## 编码

网上搜了一圈，用Java来实现海明码的例子很少，仅有的一篇文章我看了一下，代码比较粗糙，最终还是决定自己去实现一个，毕竟原理还是比较简单的。

```
/**
 * 这里剔除了优惠码的转换，只展示海明码的生成
 * @param src
 * @return
 */
private static String hmCode(String src) {
    /*
     * 第一步先确定需要多少位校验码。 
     * 设数据有n位，校验码有x位。则校验码一共有2^x种取值方式。
     * 其中需要一种取值方式表示数据正确，剩下2^x−1种取值方式表示有一位数据出错。
     * 因为编码后的二进制串有n+x位，因此x应该满足：2^x−1>=n+x
     * 使不等式成立的x的最小值就是校验码的位数
     */
    String[] source = src.split("");//JDK1.7该方法会多出第一位为null
    int n = source.length;
    int x = 0;
    while((1<<x)-1<n+x) x++;
    
    byte[] finalary = new byte[n+x];//申请数存放最终数据的数组
    
    /*
     * 校验码在二进制串中的位置为2的整数幂，即１、2、4、8、16……..剩下的位置是信息码
     */
    int[] index = new int[x];//记录校验码的位置
    for(int i=0,xx=0,point=0,in=0;i<finalary.length;i++) {
        if(1<<xx==i+1 && xx<=x) {
            xx++;
            index[in++] = i;
        }else {
            if(point<n) {
                finalary[i] = Byte.parseByte(source[point++]);
            }
        }
    }
    
    /*
     * 由于奇偶校验原理一样，偶校验的计算更为简单，实际中多用偶校验
     */
    byte[] ver = new byte[x];//记录校验位的值
    for(int p=0;p<x;p++) {
        int verification = 0;
        for(int i=1<<p;i<=finalary.length;i++) {
            if(((i>>p)&1)==1) {
                verification=verification^finalary[i-1];
            }
        }
        ver[p] = (byte) (verification==0?0:1);
    }
    
    /*
     * 校验码填进校验位
     */
    for(int i=0;i<x;i++) {
        finalary[index[i]]= ver[i];
    }
    
    return finalary;
}
```

## 校验
这里只做了校验合法性，海明码还支持找到错误位和修复错误（最多一位），我这里用不到就没有去实现，原理同样很简单。

```
public static boolean hmCheck(String src) {
    String[] source = src.split("");
    int n = source.length;
    int x = 0;
    while((1<<x)-1<n) x++;
    
    for(int p=0;p<x;p++) {
        int verification = 0;
        for(int i=1<<p;i<=n;i++) {
            if(((i>>p)&1)==1) {
                verification=verification^Integer.parseInt(source[i-1]);
            }
        }
        if(verification!=0) {
            return false;
        }
    }
    
    return true;
}
```

## 运行

```
public static void main(String[] args) {	
    System.out.println(Arrays.toString(hmCode("10110")));
    System.out.println(hmCheck("011001100"));
}
```

结果为：

>[0, 1, 1, 0, 0, 1, 1, 0, 0]
>
>true