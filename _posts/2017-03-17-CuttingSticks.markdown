---
layout: post
title: Uva10003 -- Cutting Sticks
date: 2017-03-12 11:30
comments: true
external-url:
categories: Dynamic_Programming
---

Cutting Sticks!

## 傳送門
[Uva10003](https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=944)

## 題目概述
給一根給定長度的棍子，還有切割點。
每次切割的花費會是棍子的長度，問要將切割點全部切完，最少需要多少花費。

## 解題想法
剛開始先思考，如何確定最優解的第一刀落在哪裡？一定是在第 i, j 個斷點之間的某個斷點 k ，也就是要找到某個 k, $$i < k < j$$ 且使得這一刀是最佳解（在這裡把棍子的開頭和結尾也當作斷點）。延伸這個想法到 (i, k), (k, j) 兩個部分，這兩個部分變為原題目的子問題，得到兩邊最小花費相加的最小值，再加上原先 (i, j) 切一刀的花費（注意：cost(i, j) 當 $$j-i \leq 1$$ 時，$$ cost(i, j) = 0 $$，因為不用切），就是總共最小花費。因此間單地遞迴下去即可得到最佳解。
<center>
$$f(i,j) = cost(i, j) + \min_{k, i<k<j}\left( f(i,k)+f(k,j) \right) $$
</center>

稍微觀察不難發現，有很多 f(i, j) 是多算的，所以只要記得把 f(i, j) 做一下懶人標記，存起來，就不會重複計算。

但這個結果其實可以 Bottom-up 完成，開始把 Top-down 的遞迴想辦法轉成 Bottom-up 的 DP 表格吧！

> 這題有 $$O(N^3)$$ 和 $$O(N^2)$$ 的算法，先附上 $$O(N^3)$$ 的。

## $$O(N^3)$$程式碼:

{% highlight cpp linenos=table %}
#include <bits/stdc++.h>
using namespace std;
vector<int> cutp;
const size_t MAXN(60);
const int LIM=1e9+7;
int dp[MAXN][MAXN];

int cuts(int l, int r) {
    for(int i=r-2; i>=0; --i) {
        for(int j=i+2; j<=r; ++j) {
            int minv=LIM;
            for(int k=i+1; k<j; ++k) {
                minv=min(minv,dp[i][k]+dp[k][j]);
            }
            dp[i][j]=cutp.at(j)-cutp.at(i)+minv;
        }
    }
    return dp[l][r];
}

int main(void) {
    ios::sync_with_stdio(false);cin.tie(0);
    int n, l;
    while(cin>>l&&l) {
        cutp.clear();
        cin>>n;
        cutp.push_back(0);
        for(int i=0; i<n; ++i) {
            int t; cin>>t;
            cutp.push_back(t);
        }
        cutp.push_back(l);
        int ans=LIM;
        for(int i=0; i<=n+1; ++i) for(int j=0; j<=n+1; ++j) dp[i][j]=0;
        cout<< "The minimum cutting is " << cuts(0,n+1) <<".\n";
    }
    return 0;
}
{% endhighlight %}

之前參加活動，聽到了 DP 優化方法，其中一種叫四邊形優化，這題可以用其中的 2D/1D 凸性優化解，可以將總體複雜度壓到 $$O(N^2)$$ 。
關於這個優化，我是參考[這裡](http://chino.taipei/code-2016-0402Algorithm-DP優化之四邊形不等式優化/)的文章，非常有幫助。

觀察以後可以發現，我們要找的最佳 k ，本來在 (i, j) 的區間內，但是優化後可以發現其實下界不會低於左邊的最佳k ，上界不會高於下面的最佳 k。

## $$O(N^2)$$程式碼:

{% highlight cpp linenos=table %}
#include <bits/stdc++.h>
using namespace std;
vector<int> cutp;
const size_t MAXN(55);
const int LIM=1e9+7;
int dp[MAXN][MAXN];
int dp2[MAXN][MAXN]; //For opt 2D/1D

int cuts(int l, int r) {
    for(int i=r-2; i>=0; --i) {
        for(int j=i+2; j<=r; ++j) {
            int minv=LIM;
            int bestk=i+1;
            dp2[i][j-1]=(dp2[i][j-1]==-1?bestk:dp2[i][j-1]);
            dp2[i+1][j]=(dp2[i+1][j]==-1?j-1:dp2[i+1][j]);
            for(int k=dp2[i][j-1]; k<=dp2[i+1][j]; ++k) {
                int t=dp[i][k]+dp[k][j];
                if(t<minv) {
                    minv=t;
                    bestk=k;
                }
            }
            dp2[i][j] = bestk;
            dp[i][j]=cutp.at(j)-cutp.at(i)+minv;
        }
    }
    return dp[l][r];
}

int main(void) {
    ios::sync_with_stdio(false);cin.tie(0);
    int n, l;
    while(cin>>l&&l) {
        cutp.clear();
        cin>>n;
        cutp.push_back(0);
        for(int i=0; i<n; ++i) {
            int t; cin>>t;
            cutp.push_back(t);
        }
        cutp.push_back(l);
        int ans=LIM;
        for(int i=0;i<=n+1; ++i) for(int j=0; j<=n+1; ++j) {
            dp[i][j]=0, dp2[i][j]=-1;
        }
        cout<< "The minimum cutting is " << cuts(0,n+1) <<".\n";
    }
    return 0;
}
{% endhighlight %}

## 遞迴版$$O(N^3)$$
{% highlight cpp linenos=table %}
#include <bits/stdc++.h>
using namespace std;
vector<int> cutp;
const size_t MAXN(60);
const int LIM=1e9+7;
int dp[MAXN][MAXN];

int cuts(int l, int r) {
    if(r-l<=1) return 0; //no cut required
    if(dp[l][r]!=-1) return dp[l][r];
    int cost=cutp.at(r)-cutp.at(l);
    int minv=LIM;
    for(int i=l+1; i<r; ++i) {
        int t = cuts(l,i)+cuts(i,r);
        minv=min(minv, t);
    }
    cost+=minv;
    return dp[l][r]=cost;
}

int main(void) {
    ios::sync_with_stdio(false);cin.tie(0);
    int n, l;
    while(cin>>l&&l) {
        cutp.clear();
        cin>>n;
        cutp.push_back(0);
        for(int i=0; i<n; ++i) {
            int t; cin>>t;
            cutp.push_back(t);
        }
        cutp.push_back(l);
        int ans=LIM;
        for(int i=0; i<=n+1; ++i) for(int j=0; j<=n+1; ++j) dp[i][j]=-1;
        cout<< "The minimum cutting is " << cuts(0,n+1) <<".\n";
    }
    return 0;
}
{% endhighlight %}

## $$O(N^2)$$ 用測資產生器
記得要把陣列開大一點
{% highlight cpp linenos=table %}
// note: stdc++11
#include <bits/stdc++.h>
using namespace std;
class Gen{
    private:
        vector<int> data;
        int len, cutnum;
        string filename;
        ofstream f;
    public:
        void gen(int l, int c) {
            f << l << endl;
            f << c << endl;
            data.clear();
            for(int i=1; i<l ; ++i) data.push_back(i);
            unsigned seed = std::chrono::system_clock::now().time_since_epoch().count();
            shuffle(data.begin(), data.end(), default_random_engine(seed));
            vector<int> temp(data.begin(), data.begin()+c);
            sort(temp.begin(), temp.end());
            for(int i=0; i!=c; ++i) f << temp.at(i) << ' ';
            temp.clear();
            f << endl;
        }
        Gen(const string &s) {
            f.open(s.c_str());
            this->filename = s;
        }
        ~Gen() {
            f << 0 << endl;
            f.close(); f.clear();
        }
};

int main(void) {
    const string str("testfile.txt");
    Gen generator(str);
    int testdata [][2] = {
        {100, 10},
        {1000, 50},
        {1000, 50},
        {1000, 50},
        {1000, 50},
        {1000, 50},
        {1000, 500},
        {1000, 500},
        {1000, 500},
        {10000, 980},
        {100001, 1000},
        {1000001, 1000}
    };
    int n=sizeof(testdata)/sizeof(testdata[0]);
    int T=10;
    while(T--) {
        for(int i=0; i!=n; ++i) generator.gen(testdata[i][0], testdata[i][1]);
    }
    return 0;
}
{% endhighlight %}

一個預先產生好的測資在[這裡](https://gist.github.com/peter0749/8982ab47ebfb22031b8ca43e0994a837)，$$O(N^2)$$ 大約 2s。
