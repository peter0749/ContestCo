---
layout: post
title: Uva108 -- Maximum Sum
date: 2017-03-12 11:30
comments: true
external-url:
categories: Dynamic_Programming
---

Prefix sum and DP.

## 傳送門
[Uva108](https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=44)

## 題目概述
給你一個矩陣，問你裡面最大的子矩陣總和是多少？

## 解題想法
這題和一維的「找最大子陣列」很類似。
先將每一列，都算出該列的 prefix sum 備用。
之後窮舉每對欄的組合，這時已經固定了欄的範圍，
之後在這幾欄的範圍內，利用之前算到的 prefix sum ，用 $$O(1)$$ 時間算出每一列在該範圍內的總和。之後直接套用「一維找最大子陣列」問題，就可以問二維的最大子陣列問題了。

## 程式碼:

{% highlight cpp linenos=table %}
#include <bits/stdc++.h>
using namespace std;
const size_t MAXN(200);
int arr[MAXN][MAXN]; // init 0

int solv(int n) {
    for(int i=1; i<=n; ++i) {
        for(int j=1; j<=n; ++j) arr[i][j]+=arr[i][j-1];//prefix
    }
    int ans=-1e9;
    for(int i=1; i<=n; ++i) {
        for(int j=i; j<=n; ++j) {
            int r=0, t=0;
            for(int k=1; k<=n; ++k) {
                t = arr[k][j] - arr[k][i-1];
                //cout<<i<<' '<<j<<' '<<t<<endl;
                r = (t+r>0)? r+t:0;
                ans=max(ans,r);
            }
        }
    }
    return ans;
}

int main(void) {
    ios::sync_with_stdio(false);
    cin.tie(0);
    int n;
    while(cin>>n) {
        for(int i=1; i<=n; ++i) {
            for(int j=1; j<=n; ++j) {
                cin>>arr[i][j];
            }
        }
        cout<<solv(n)<<'\n';
    }
    return 0;
}
{% endhighlight %}

