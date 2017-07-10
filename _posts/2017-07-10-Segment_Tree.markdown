---
layout: post
title: Template -- Segment Tree
date: 2017-07-11 0:20
comments: true
external-url:
tags: rmq, segment_tree
---

## 線段樹模板

支援區間查詢、修改。

{% highlight cpp linenos=table %}

const size_t TREEND=4;
const size_t TREENN=MAXN*TREEND;
int  seg[TREENN]; // neg, pos
int lazy[TREENN];  // neg, pos

#define LEFT(i) ((i)<<1)
#define RIGHT(i) ((i)<<1|1)

template <class T>
inline T operationOnPull(const T &a, const T &b) {
    // pull operation here
    // example: return a + b;
}

template <class T>
inline T operationOnPush(const T &a, const T &b) {
    // push operation here
    // example: return a + b;
}

inline void init(int n, int *seg, int *lazy) {
    n*=TREEND;
    memset(seg, 0x00, sizeof(seg[0])*(n+1));
    memset(lazy, 0x00, sizeof(lazy[0])*(n+1));
}

#define IN_RANGE (qL<=L && qR>=R)
#define OO_RANGE (L>qR || R<qL)

inline void _push(int lev, int seg[], int lazy[]) {
    if (!lazy[lev]) return;

    // !!!make sure this node's information is updated before enter here!!!
    // push operation here
    // update left child, right child
    // push lazy tag onto them
    // and unset this lazy tag
    // example: seg[LEFT(lev)] = operationOnPush<int>(seg[LEFT(lev)],  (lazy[LEFT(lev)] = lazy[lev])); 
    //          seg[RIGHT(lev)] = operationOnPush<int>(seg[RIGHT(lev)], (lazy[RIGHT(lev)] = lazy[lev])); 
    //

    lazy[lev] = 0;
}

// 更新
inline void _pull(int lev, int seg[]) {
    // update self
    // pull from left child and right child
    // example: seg[lev] = operationOnPull<int>(seg[LEFT(lev)] , seg[RIGHT(lev)]);
}

void update(int L, int R, int qL, int qR, int lev, int op, int seg[], int lazy[]) {
    if (OO_RANGE) return;
    int M = (L+R)>>1;
    if (IN_RANGE) {
        // your update operation here
        /* 
        *example:
        *seg[lev] = operationOnPull<int>(seg[lev],op);
        */

        lazy[lev] = op; // lazy tag
        return;
    }
    _push(lev, seg, lazy);
    update(L, M, qL, qR, LEFT(lev), op, seg, lazy);
    update(M+1, R, qL,qR, RIGHT(lev),op,seg, lazy);
    _pull(lev, seg);
}

int query(int L, int R, int qL, int qR, int flag, int seg[], int lazy[]) {
    int M=(L+R)>>1;
    if (OO_RANGE) return 0; // 沒東西
    if (IN_RANGE) return seg[flag];
    _push(flag, seg, lazy);
    int left=0, right=0;
    left = query(L,M,qL,qR,LEFT(flag),seg,lazy);
    right = query(M+1,R,qL,qR,RIGHT(flag),seg,lazy);
    _pull(flag, seg); // if update (optional)
    return seg[flag]; // if update (optional)
    //return operationOnPull<int>(left, right); // not update
}
{% endhighlight %}


