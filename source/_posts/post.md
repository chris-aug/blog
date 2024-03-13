---
title: Java 代码测试
abbrlink: 25624
date: 2024-03-13 16:30:46
tags:
categories:
description:
cover:
sticky:
comments:
katex:
aplayer:
---
## Java 代码测试
```java
public class int_reverse {
    public static void main(String[] args){
        int x =  123415566;
        int y = -123;
//        System.out.println(Integer.MAX_VALUE);
        System.out.println(reverse(x));
        System.out.println(reverse(y));
    }

    public static int reverse(int x) {
        int rev = 0;
        while (x != 0) {
            int pop = x % 10; // 求余预算取最后一位数字
            x /= 10; // 除10去掉最后一位数字

            if (rev > Integer.MAX_VALUE / 10 || (rev == Integer.MAX_VALUE / 10 && pop > Integer.MAX_VALUE % 10)) return 0;
            // Integer.MAX_VALUE % 10 == 7
            if (rev < Integer.MIN_VALUE / 10 || (rev == Integer.MIN_VALUE / 10 && pop < Integer.MIN_VALUE % 10)) return 0;
            // / Integer.MAX_VALUE % 10 == -8
            rev = rev * 10 + pop;
        }
        return rev;
    }
}
```
```sh
$ groupadd samba	#创建用户组
$ useradd -g samba sambausr #创建用户
$ smbpasswd -a sambausr	#创建smaba访问密码
```