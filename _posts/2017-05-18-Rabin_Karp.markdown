---
layout: post
title: [Template] Rolling Hash
date: 2017-05-18 7:30
comments: true
external-url:
tags: hash string
---

Rolling Hash (Rabin-Karp) 模板

想想我們是怎麼將字串轉成十進位數字，都是一樣的道理。

## 字串轉 Hash（基本款）

{% highlight cpp linenos=table %}

typedef unsigned long long int UT;
char str[STRL], pat[STRL];
const UT hsaBase = 37, MOD1d7=1000000007;
UT hsaPow, hsaS, hsaP;

UT gethash(char *S, int len) {
    UT h=0;
    for(int i=0; i!=len; ++i ) {
        h = (h * hsaBase  ) % MOD1d7;
        h = (h + (UT)S[i]) % MOD1d7;
    }
    return h;
}

{% endhighlight %}

## 建構 Hash 表：

用兩組 base 和 mod 檢查碰撞和其他鳥事...

{% highlight cpp linenos=table %}

#define PB push_back
#define F first
#define S second
typedef long long int UT;
typedef pair< UT, UT > PII;
const PII p(29,31), q(10000103, 10000121);
const int MAXN(1010);
PII h[MAXN], rh[MAXN], cache[MAXN];

inline void get_pow(PII *h, const PII base, const int n, const PII P) {
    h[0].F=h[0].S=1;
    for(int i=1; i<=n; ++i) {
        h[i].F = (h[i-1].F*base.F)%P.F;
        h[i].S = (h[i-1].S*base.S)%P.S;
    }
}

inline void get_hash(string &s, PII *h, bool rev=false) {
    // rev 代表是否要反向建 Hash
    h[0].F=h[0].S=h[s.length()+1].S=h[s.length()+1].F=0;
    ++h; // index is shifted
    if(!rev) {
        for(int i=0; i!=s.length(); ++i ){
            h[i].F = ((h[i-1].F*p.F%q.F)+s[i]+1)%q.F;
            h[i].S = ((h[i-1].S*p.S%q.S)+s[i]+1)%q.S;
        }
    } else { //reverse
        for(int i=s.length()-1, j=0; i!=-1; --i,  ++j ){
            h[j].F = ((h[j-1].F*p.F%q.F)+s[i]+1)%q.F;
            h[j].S = ((h[j-1].S*p.S%q.S)+s[i]+1)%q.S;
        }
    }
}

// pw 代表 在 get_pow() 得到的 hash 值
// 下面的函數用來取得 從原字串 i 位置開始長度 n 子字串的 hash 值
inline UT partial_hash(PII *h, PII const &pw, int i, int n) {
    h++; //shift index
    UT temp1, temp2;//Lazy dog...
    temp1 = ((q.F - h[i-1].F*pw.F%q.F) + h[i+n-1].F)%q.F;
    temp2 = ((q.S - h[i-1].S*pw.S%q.S) + h[i+n-1].S)%q.S;
    if (temp1!=temp2) {
        return -1;
        // 失敗了，在外面做其他事情解決它
    } else return temp1;
}

{% endhighlight %}

更多例子：

[字串比對](https://peter0749.github.io/ContestCo/String-Matching.html)
[LPS](https://peter0749.github.io/ContestCo/LPS.html)

