---
layout: post
title: [Template] Union Find
date: 2017-05-18 7:25
comments: true
external-url:
tags: union_find
---

Union Find 模板

用 `par` 紀錄樹上節點的 parent 是誰，用 `rank` 來優化樹高。

## 程式碼:

{% highlight cpp linenos=table %}

int par[MAXN], rank[MAXN];

void Initialize(){
    int i=((N+1)<<1);
    while(i--){
        par[i]=i;
        rank[i]=1;
    }
}

int Find(int foo){ return foo==par[foo]?foo:(par[foo]=Find(par[foo])); }
char Same(int foo, int bar) { return Find(foo)==Find(bar); }
void Union(int x, int y){
    x = Find(x);
    y = Find(y);
    if(x==y) return;
    if(rank[x]<rank[y]) par[x]=y;
    else {
        par[y]=x;
        if(rank[x]==rank[y]) ++rank[x];
    }
}

{% endhighlight %}

