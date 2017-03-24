---
layout: post
title: 基數排序（整數）
date: 2017-02-19 19:50
comments: true
external-url:
tags: sort
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

{% highlight cpp linenos=table %}
#include <bits/stdc++.h>
using namespace std;

template<class T>
class RadixSort {
typedef typename vector<T>::iterator VITE;
    private:
        vector<T>data;
        T base;
        void count_sort(const T exp, T *count, vector<T> *&temp) {
            fill(count, count+base, 0x00);
            for(VITE v=data.begin(); v!=data.end(); ++v) {
                ++count[ (*v/exp)%base ];
            }
            partial_sum(count, count+base, count);
            for(int i=data.size()-1; i!=-1; --i) {
                temp->at(--count[(data.at(i)/exp)%base]) = data.at(i);
            }
            data=*temp;
        }
    public:
        void radix_sort() {
            T maxv=*max_element(data.begin(), data.end());
            vector<T> *temp = new vector<T>(data.size());
            T *count = new T[base];
            for(T v=1; maxv / v > 0; v*=base) count_sort(v, count, temp);
            delete[] count; count=NULL;
            delete temp; temp=NULL;
        }
        inline void push_back(T d) {
            data.push_back(d);
        }
        inline void insert(const VITE b, const VITE l, const VITE r) {
            data.insert(b,l,r);
        }
        inline T get(size_t n) { return data.at(n); }
        RadixSort() {
            this->base=10;
        }
        inline size_t size(void) {
            return data.size();
        }
        inline void clear(void) {
            data.clear();
        }
        RadixSort(T b) {
            this->base = b;
        }
};

int main(void) {
    ios::sync_with_stdio(false); cin.tie(0); cout.tie(0);
    RadixSort<int> arr(255);
    int N;
    while(cin>>N) {
        for(int i=0; i<N; ++i) {
            int temp; cin>>temp;
            arr.push_back(temp);
        }
        arr.radix_sort();
        cout<<arr.get(0);
        for(int i=1; i!=arr.size(); ++i) cout<<' '<<arr.get(i);
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

radix sort 大概 70~80ms

quick sort 大概 0.2s
