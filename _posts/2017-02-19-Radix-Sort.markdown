---
layout: post
title: 基數排序（整數）
date: 2017-02-19 19:50
comments: true
external-url:
categories: sort
---

Radix Sort!

用 ZeroJudge 上的題目測一下正確性。

## 傳送門
[ZJa233](https://zerojudge.tw/ShowProblem?problemid=a233)

## 限制

<center>
$$1 \leq N \leq 1000000$$
</center>
測資會卡教科書上的 Quick Sort

## 程式碼:
其實是亂寫的~~

{% highlight cpp linenos=table %}
#include <bits/stdc++.h>
using namespace std;

inline void rxd(vector<unsigned int> &arr) {
    unsigned int maxv = *max_element(arr.begin(), arr.end());
    if(maxv==0) return;
    maxv = 32-__builtin_clz(maxv);
    vector<unsigned int> bucket[2];
    for(unsigned int d=0; d<maxv; ++d) {
        bucket[0].clear(); bucket[1].clear();
        for(vector<unsigned int>::iterator i=arr.begin();\
            i!=arr.end(); ++i) {
            bucket[ ((*i)>>d)&1u ].push_back(*i);
        }
        arr.clear();
        #define UPDATE(X,Y) \
            X.insert( X.end(), Y.begin(), Y.end())
        UPDATE(arr, bucket[0]);
        UPDATE(arr, bucket[1]);
    }
}

vector<unsigned int> arr;

int main(void) {
    ios::sync_with_stdio(false);cin.tie(0);
    int N;
    while(cin>>N) {
        for(int i=0; i<N; ++i) {
            unsigned int temp; cin>>temp;
            arr.push_back(temp);
        }
        rxd(arr);
        vector<unsigned int>::iterator i=arr.begin();                          
        cout<<*i;
        for(++i; i!=arr.end(); ++i) cout<<' '<<*i;
        cout<<'\n';
        arr.clear();
    }
    return 0;
}
{% endhighlight %}

順手改一下，讓他也支援 int

{% highlight cpp linenos=table %}
#include <bits/stdc++.h>
using namespace std;

inline void rxd(vector<unsigned int> &arr) {
    unsigned int maxv = *max_element(arr.begin(), arr.end());
    if(maxv==0) return;
    maxv = 32-__builtin_clz(maxv);
    vector<unsigned int> bucket[2];
    for(unsigned int d=0; d<maxv; ++d) {
        bucket[0].clear(); bucket[1].clear();
        for(vector<unsigned int>::iterator i=arr.begin();\
            i!=arr.end(); ++i) {
            bucket[ ((*i)>>d)&1u ].push_back(*i);
        }
        arr.clear();
        #define UPDATE(X,Y) \
            X.insert( X.end(), Y.begin(), Y.end())
        UPDATE(arr, bucket[0]);
        UPDATE(arr, bucket[1]);
    }
}

void split_rxd(vector<int> &arr) {
    vector<unsigned int> pos, neg;
    for(vector<int>::iterator i=arr.begin(); i!=arr.end() ; ++i ) {
        if(*i<0) neg.push_back((unsigned int)-*i);
        else pos.push_back((unsigned int)*i);
    }
    arr.clear();
    if(!pos.empty()) rxd(pos);
    if(!neg.empty()) rxd(neg);
    for(vector<unsigned int>::reverse_iterator i=neg.rbegin();\
        i!=neg.rend(); ++i) {
        arr.push_back((int)(-*i));
    }
    neg.clear();
    arr.insert(arr.end(), pos.begin(), pos.end() );
    pos.clear();
    //end of sorting
}
vector<int> arr;

int main(void) {
    ios::sync_with_stdio(false);
    //cin.tie(0);
    int N;
    while(cin>>N) {
        for(int i=0; i<N; ++i) {
            int temp; cin>>temp;
            arr.push_back(temp);
        }
        split_rxd(arr);
        vector<int>::iterator i=arr.begin();
        cout<<*i;
        for(++i; i!=arr.end(); ++i) cout<<' '<<*i;
        cout<<'\n';
        arr.clear();
    }
    return 0;
}
{% endhighlight %}

還有內建的 qsort
{% highlight cpp linenos=table %}

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#define MAX 1000000
int arr[MAX];

int cmp(const void *a, const void *b) {
    return *(const int*)a - *(const int*)b;
}

int main(void) {
    int n, i;
    while(scanf("%d", &n)!=EOF) {
        for(i=0; i<n; i++)scanf("%d", arr+i);
        qsort(arr,n,sizeof(int),cmp);
        printf("%d",arr[0]);
        for(i=1; i<n; i++)printf(" %d",arr[i]);
        printf("\n");
    }
    return 0;
}

{% endhighlight %}

## 速度
qsort 大概 0.1s

radix sort 大概 80ms

quick sort 大概 0.2s
