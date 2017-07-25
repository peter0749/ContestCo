---
layout: post
title: Template -- 2D Rolling Hash (Rabin-Karp)
date: 2017-07-25 7:30
comments: true
external-url:
tags: hash string
---

2D Rolling Hash (Rabin-Karp) 模板

## 實作方式

### 前置動作

要準備兩個 hash 時要使用的 base (質數) ，一個給 X 方向用、一個給 Y 方向用，

還有一個 hash 時要用的 Mod 數 (質數) 。

在產生 hash 時，X 和 Y 方向的 Mod 數必須一致，才可以得到正確結果。

產生 2D Rabin-Karp 的方法與 "子矩陣求和" 用的方法類似，

### 以子矩陣求和為出發點

假設我們有一個矩陣 A ，要在 $$O(1)$$ 內回答矩陣 A 內的某個子矩陣總和，

例如：$$A[2..3][5..6]$$ 。我們可以先求出各種 $$A[0..N-1][0..M-1]$$ 的總和，

為了方便起見，我們將上面的 $$A[0..N-1][0..M-1]$$ 總和定義為 $$S[N][M]$$

代表從 A 最左上角 $$A[0][0]$$ 累計到 $$A[N-1][M-1]$$ 的矩陣內數字總和，

然後 $$A[2..3][5..6]$$ 就可以看成，

從 $$A[2][5]$$ 往下找 $$2x2$$ 的子矩陣（包含），就會是

$$A[2..3][5..6] = S[2+2][5+2] - S[2][5+2] - S[2+2][5] + S[2][5] $$

S 就是 A 的 2D prefix sum ，要求得的結果就是，

右下 prefix sum - 上方 prefix sum - 左方 prefix sum + 左上 prefix sum

至於為什麼是這樣，可以到 [這裏](https://leetcode.com/problems/range-sum-query-2d-immutable/#/description) 寫個題目，感受一下。

### 從子矩陣求和到 2D Rabin-Karp

說明完了 2D prefix sum ，從這段開始進入正題，要如何在 $$O(1)$$ 的時間內，求出一個區域的 hash 值呢？

我們可以把剛剛的 $$S$$ 矩陣重新定義為：

$$S[i][j] = S[i-1][j] \times P_{y} + RollingHashFromLeft \times P_{x} + A[i-1][j-1] \left( mod P \right)$$

其中， $$P_{x}$$ 代表 X 方向的 base ， $$P_{y}$$ 代表 Y 方向的 base ，

$$RollingHashFromLeft$$ 代表從 X 方向累計過來的 Rolling Hash (Rabin-Karp) 值。

> 一些細節：要先將 $$S[0][...]$$ 和 $$S[...][0]$$ 初始化為 0 

$$S[i][j]$$ 會接受它上方的 Hash 值，乘上 Y 方向的 base ，也會接受左方累計來的 Rolling Hash ，乘上 X 方向的 base ，並加上矩陣中的數值。

這樣，在 Y 方向會有 Y 的 base ，在 X 方向會有 X 的 base ，就可以用下面類似子矩陣求和的方法，很方便地算出區域的 hash 值了。

### 查詢

那麼如何查詢呢？我們知道 $$S[i][j]$$ 是由它上方和左方的 hash 值，與原先矩陣中的值推得的。

那麼反過來做就好了！

從 $$A[i][j]$$ 往下找大小為 NxM 的矩陣 hash 值為：

$$ S[i+N][j+M] - S[i][j+M] \times P_{y}^{N} - S[i+N][j] \times P_{x}^M + S[i][j] \times P_{x}^M P_{y}^N $$

$$P_{x}^{M}$$ 和 $$P_{y}^{N}$$ 可以預建表，因此查詢時間就可以是 $$O(1)$$

以上就是 2D Rolling Hash (Rabin-Karp) 的建構、查詢方式，希望能幫到有需要的人！

## 範例程式碼

{% highlight cpp linenos=table %}
#include <cstring>
#include <functional>

#define UT long int
struct RollingHash2D {
#define PB push_back
#define F first
#define S second
    enum { MAXN=101, MAXM=101 };
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
//const UT RollingHash2D::px(311);
const UT RollingHash2D::px(31);
//const UT RollingHash2D::py(337);
const UT RollingHash2D::py(337);
const UT RollingHash2D::q(10000121); // 一些常數
#undef UT

int string2D[7][7] = {
    0, 0, 1, 1, 2, 3, 5,
    3, 1, 0, 9, 11, 13, 15,
    2, 3, 0, 8, 13, 0, 1,
    0, 0, 1, 1, 2, 3, 4,
    0, 7, 1, 5, 1, 0, 0, // (4, 4)
    3, 1, 0, 0, 3, 0, 7, //
    1, 4, 0, 0, 1, 3, 2
                ////
};

int patt[2][2] = {
    1, 0,
    3, 0
    //5, 7,
    //3, 5
};
RollingHash2D stringH2D;
RollingHash2D pattH2D;
#include <iostream>
#include <iomanip>
int main(void) {
    using namespace std;
    cout << "2D string to match:" << endl;
    for (int i=0; i<7; ++i) {
        for (int j=0; j<7; ++j)
            cout << setw(3) << string2D[i][j];
        if (i==4||i==1) cout << "  <----- here";
        cout << endl;
    }
    cout << "pattern (this should be located at string2D[1][1] and string2D[4][4]): " << endl;
    for (int i=0; i<2; ++i) {
        for (int j=0; j<2; ++j) {
            cout << setw(3) << patt[i][j];
        }
        cout << endl;
    }
    stringH2D.get_pow(7,7);
    pattH2D.get_pow(2,2);
    stringH2D.get_hash(&string2D[0][0], 7, 7);
    pattH2D.get_hash(&patt[0][0], 2, 2);
    long int pattH = pattH2D.partial_hash(0,0,2,2);
    cout << "pattH: " << pattH << endl;
    for (int i=0; i+2<=7; ++i) {
        for (int j=0; j+2<=7; ++j) {
            long int stringH = stringH2D.partial_hash(i,j,2,2);
            if (stringH==pattH) cout << "Found! (" << i << ',' << j << ") : " << stringH << endl;
        }
    }
    return 0;
}

{% endhighlight %}

