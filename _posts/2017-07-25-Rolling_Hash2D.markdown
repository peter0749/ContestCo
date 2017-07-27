---
layout: post
title: Template -- 2D Rolling Hash
date: 2017-07-25 7:30
comments: true
external-url:
tags: hash string
---

2D Rolling Hash 模板

## 實作方式

### 前置動作

要準備兩個 hash 時要使用的 base (質數) ，一個給 X 方向用、一個給 Y 方向用，

還有一個 hash 時要用的 Mod 數 (質數) 。

在產生 hash 時，X 和 Y 方向的 Mod 數必須一致，才可以得到正確結果。

產生 2D 的方法與 "子矩陣求和" 用的方法類似，

### 以子矩陣求和為出發點

假設我們有一個矩陣 A ，要在 $$O(1)$$ 內回答矩陣 A 內的某個子矩陣總和，

例如：$$A[2..3][5..6]$$ 。我們可以先求出各種 $$A[0..N-1][0..M-1]$$ 的總和，

為了方便起見，我們將上面的 $$A[0..N-1][0..M-1]$$ 總和定義為 $$S[N][M]$$

代表從 A 最左上角 $$A[0][0]$$ 累計到 $$A[N-1][M-1]$$ 的矩陣內數字總和，

然後 $$A[2..3][5..6]$$ 就可以看成，

從 $$A[2][5]$$ 往下找 $$2 \times 2$$ 的子矩陣（包含），就會是

$$A[2..3][5..6] = S[2+2][5+2] - S[2][5+2] - S[2+2][5] + S[2][5] $$

S 就是 A 的 2D prefix sum ，要求得的結果就是，

右下 prefix sum - 上方 prefix sum - 左方 prefix sum + 左上 prefix sum

至於為什麼是這樣，可以到 [這裏](https://leetcode.com/problems/range-sum-query-2d-immutable/#/description) 寫個題目，感受一下。

### 從子矩陣求和到 2D Rolling Hash

說明完了 2D prefix sum ，從這段開始進入正題，要如何在 $$O(1)$$ 的時間內，求出一個區域的 hash 值呢？

我們可以把剛剛的 $$S$$ 矩陣重新定義為：

$$S[i][j] = S[i-1][j] \times P_{y} + RollingHashFromLeft \times P_{x} + A[i-1][j-1] \left( mod P \right)$$

其中， $$P_{x}$$ 代表 X 方向的 base ， $$P_{y}$$ 代表 Y 方向的 base ，

$$RollingHashFromLeft$$ 代表從 X 方向（左方）累計過來的 Rolling Hash 值。

> 一些細節：要先將 $$S[0][...]$$ 和 $$S[...][0]$$ 初始化為 0 

$$S[i][j]$$ 會接受它上方的 Hash 值，乘上 Y 方向的 base ，也會接受左方累計來的 Rolling Hash ，乘上 X 方向的 base ，並加上矩陣中的數值。

這樣，在 Y 方向會有 Y 的 base ，在 X 方向會有 X 的 base ，就可以用下面類似子矩陣求和的方法，很方便地算出區域的 hash 值了。

### 查詢

那麼如何查詢呢？我們知道 $$S[i][j]$$ 是由它上方和左方的 hash 值，加上原先矩陣中的值推得的。

反過來做就好了！

從 $$A[i][j]$$ 往下找大小為 NxM 的矩陣 hash 值為：

$$ S[i+N][j+M] - S[i][j+M] \times P_{y}^{N} - S[i+N][j] \times P_{x}^M + S[i][j] \times P_{x}^M P_{y}^N $$

$$P_{x}^{M}$$ 和 $$P_{y}^{N}$$ 可以預建表，因此查詢時間就可以是 $$O(1)$$

以上就是 2D Rolling Hash 的建構、查詢方式，希望能幫到有需要的人！

## 範例程式碼

驗證程式正確性 [Uva11019 -- Matrix Matcher](https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=1960)

{% highlight cpp linenos=table %}
// AC, C++11, 0.220s
#pragma GCC optimize ("O3")
#include <cstring>
#include <functional>

#define UT long int
struct RollingHash2D {
#define PB push_back
#define F first
#define S second
    enum { MAXN=1002, MAXM=1002 };
    static const UT px, py, q; // X向 hash 的 base, Y向 hash 的 base, mod Prime
    UT h[MAXN][MAXM];
    UT cacheX[MAXM], cacheY[MAXN];

    inline int mulmod (int a, int b, int m) { // 快速模
        int d, r;
        if(a==0 or b==0) return 0;
        if(a==1 or b==1) return (a>b?a:b);
        __asm__("mull %4;"
                "divl %2"
                : "=d" (r), "=a"(d)
                : "r"(m), "a"(a), "d"(b));
        return  r;
    }

    inline void get_pow(const int n, const int m) { // Hash basis 冪次
        // n, 高 / m, 寬
        cacheX[0]=1;
        cacheY[0]=1;
        for (int i=1; i<=n; ++i) {
            cacheY[i] = this->mulmod(cacheY[i-1], py, q);
        }
        for (int i=1; i<=m; ++i) {
            cacheX[i] = this->mulmod(cacheX[i-1], px, q);
        }
    }

    inline void get_hash(int *s, int n, int m) { // 取得 hash 表
        // n, 高 / m, 寬
        // 下面要做的事和 2D prefix sum 類似
        std::memset(h, 0x00, sizeof(UT)*MAXN*MAXM);
        for (int i=1; i<=n; ++i) {
            int sum=0;
            for (int j=1; j<=m; ++j) {
                sum = (this->mulmod(sum, px, q) + (s[(i-1)*m+j-1]+1))%q; // X 向 hash
                h[i][j] = (this->mulmod(h[i-1][j],py,q) + sum)%q;     // Y 向 hash
            }
        }
    }

    UT partial_hash(int i, int j, int n, int m) {
        // i: row; j: col; n: 高; m: 寬
        // 從 s[i][j] 往下看 nxm 大小矩陣，所對應的 hash 值
        const UT &pwX = cacheX[m]; // pX^m
        const UT &pwY = cacheY[n]; // pY^n
        UT result = \
        ((h[i+n][j+m] + q - (h[i][j+m]*pwY + h[i+n][j]*pwX)%q)%q + this->mulmod(h[i][j],this->mulmod(pwX,pwY,q),q))%q;
        // 小心 overflow
        return result;
    }
#undef PB
#undef F
#undef S
};
const UT RollingHash2D::px(311);
const UT RollingHash2D::py(337);
const UT RollingHash2D::q(10000121); // 一些常數
#undef UT

RollingHash2D stringH2D;
RollingHash2D pattH2D;
#include <iostream>
#include <iomanip>

int str[1000001];
int pat[10001];

int main(void) {
    using namespace std;
    ios::sync_with_stdio(false);
    cin.tie(0);cout.tie(0);
    stringH2D.get_pow(1001,1001);
    pattH2D.get_pow(101,101);
    int T; cin >> T;
    while(T--) {
        int sh,sw,th,tw;
        cin >> sh >> sw;
        for (int i=0, k=0; i<sh; ++i) for (int j=0; j<sw; ++j) {
            char c; cin >> c;
            str[k++] = (int)c;
        }
        cin >> th >> tw;
        for (int i=0, k=0; i<th; ++i) for (int j=0; j<tw; ++j) {
            char c; cin >> c;
            pat[k++] = (int)c;
        }
        stringH2D.get_hash(str, sh, sw);
        pattH2D.get_hash(pat, th, tw);
        int cnt=0;
        int patV = pattH2D.partial_hash(0,0,th,tw);
        for (int i=0; i+th<=sh; ++i) for (int j=0; j+tw<=sw; ++j)
            if (stringH2D.partial_hash(i,j,th,tw)==patV) ++cnt;
        cout << cnt << endl;
    }
    return 0;
}
{% endhighlight %}

## 範例二

上面的只用一組 base ，有可能發生碰撞，這個時候可以考慮使用兩組 base。

{% highlight cpp linenos=table %}
#pragma GCC optimize ("O3")
#include <cstring>
#include <functional>
#define UT long long int
struct PI {
    const static UT q;
    UT x, y;
    PI(UT x, UT y) : x(x), y(y) {}
    PI(const PI &ref) : x(ref.x), y(ref.y) {}
    PI() : x(0), y(0) {}
};
const UT PI::q(10000121); // 在 mod p 底下執行
const PI operator+(const PI &l, const PI &r) {
    PI res((l.x+r.x)%PI::q,(l.y+r.y)%PI::q);
    return res;
}
const PI operator-(const PI &l, const PI &r) {
    PI res((l.x+PI::q-r.x)%PI::q, (l.y+PI::q-r.y)%PI::q);
    return res;
}
const PI operator*(const PI &l, const PI &r) {
    PI res(l.x*r.x%PI::q, l.y*r.y%PI::q);
    return res;
}
const bool operator==(const PI &l, const PI &r) { // for std::unordered_set, std::unordered_map
    return l.x==r.x&&l.y==r.y;
}
const bool operator<(const PI &l, const PI &r) {
    if (l.x!=r.x) return l.x<r.x;
    return l.y<r.y;
}
namespace std { // this give your custom class hash value
    template <>
    struct hash<PI> {
        // @overload
        std::size_t operator()(const PI& p) const {
            using std::hash;
            using std::pair;
            return hash<UT>()(p.x) ^ hash<UT>()(p.y);
        }
    };
};
struct RollingHash2D {
#define PB push_back
#define F first
#define S second
    enum { MAXN=1111, MAXM=1111 };
    static const PI px, py; // X向 hash 的 base, Y向 hash 的 base, mod Prime
    PI h[MAXN][MAXM];
    PI cacheX[MAXM], cacheY[MAXN]; // power of base X, power of base Y

    inline void get_pow(const int n, const int m) { // Hash basis 冪次
        // n, 高 / m, 寬
        cacheX[0].x=cacheX[0].y=1;
        cacheY[0].x=cacheY[0].y=1;
        for (int i=1; i<=n; ++i) {
            cacheY[i] = cacheY[i-1] * py;
        }
        for (int i=1; i<=m; ++i) {
            cacheX[i] = cacheX[i-1] * px;
        }
    }

    inline void get_hash(int *s, int n, int m) { // 取得 hash 表
        // n, 高 / m, 寬
        // 下面要做的事和 2D prefix sum 類似
        std::memset(h, 0x00, sizeof(PI)*MAXN*MAXM);
        for (int i=1; i<=n; ++i) {
            PI sum(0,0);
            for (int j=1; j<=m; ++j) {
                UT val = (UT)(s[(i-1)*m+j-1]+1);
                PI S(val, val);
                sum = sum * px + S; // X 向 hash
                h[i][j] = h[i-1][j] * py + sum;     // Y 向 hash
            }
        }
    }

    PI partial_hash(int i, int j, int n, int m) {
        // i: row; j: col; n: 高; m: 寬
        // 從 s[i][j] 往下看 nxm 大小矩陣，所對應的 hash 值
        const PI &pwX = cacheX[m]; // pX^m
        const PI &pwY = cacheY[n]; // pY^n
        PI result = h[i+n][j+m] - (h[i][j+m]*pwY + h[i+n][j]*pwX) + h[i][j]*pwX*pwY;
        // 小心 overflow
        return result;
    }
#undef PB
#undef F
#undef S
};
const PI RollingHash2D::px(47,331); // X, Y 個使用兩組 base, 就不怕碰撞了
const PI RollingHash2D::py(337,37);

RollingHash2D stringH2D;
RollingHash2D pattH2D;
#include <iostream>
#include <iomanip>

int str[1000001];
int pat[10001];

int main(void) {
    using namespace std;
    ios::sync_with_stdio(false);
    cin.tie(0);cout.tie(0);
    stringH2D.get_pow(1001,1001);
    pattH2D.get_pow(101,101);
    int T; cin >> T;
    while(T--) {
        int sh,sw,th,tw;
        cin >> sh >> sw;
        for (int i=0, k=0; i<sh; ++i) for (int j=0; j<sw; ++j) {
            char c; cin >> c;
            str[k++] = (int)c;
        }
        cin >> th >> tw;
        for (int i=0, k=0; i<th; ++i) for (int j=0; j<tw; ++j) {
            char c; cin >> c;
            pat[k++] = (int)c;
        }
        stringH2D.get_hash(str, sh, sw);
        pattH2D.get_hash(pat, th, tw);
        int cnt=0;
        PI patV = pattH2D.partial_hash(0,0,th,tw);
        for (int i=0; i+th<=sh; ++i) for (int j=0; j+tw<=sw; ++j)
            if (stringH2D.partial_hash(i,j,th,tw)==patV) ++cnt;
        cout << cnt << endl;
    }
    return 0;
}

{% endhighlight %}

附上一張推導時畫的東西：

![img](http://i.imgur.com/GZWrPsH.jpg)


