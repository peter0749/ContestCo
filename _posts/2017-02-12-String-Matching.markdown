---
layout: post
title: 字串比對
date: 2017-02-12 14:03
comments: true
external-url:
categories: string
---

String matching

練習一下基本的字串比對

## 傳送門:
[POJ3461](http://poj.org/problem?id=3461)

## 程式碼:
主要練習了兩種方法，一種用 MP 算法、一種用 hash

## MP
改天再練習 KMP ，很怕 MP 算法會被 Hack

{% highlight cpp linenos=inline %}

#include <iostream>
#include <cstdio>
#include <cstdlib>
#include <cstring>
#define STRL 1000010
using namespace std;
int failure[STRL];
char pattern[STRL];
char tstring[STRL];
int Pos[STRL];

int MorrisPratt(char *t, char *p, int *stk){
    int i, j, top=0;
    int tlen = strlen(t);
    int plen = strlen(p);
    if(plen > tlen) return 0;

    for(i=1, j=failure[0]=-1; i<plen; ++i){
        while(j>=0 && p[j+1]!=p[i])
            j = failure[j];
        if(p[j+1]==p[i]) ++j;
        failure[i] = j;
    }

    for(i=0, j=-1; i<tlen; ++i){
        while(j>=0 && p[j+1]!=t[i])
            j = failure[j];

        if(p[j+1]==t[i]) ++j;

        if(j==plen-1){
            //printf("at: %d\n", i-plen+1);
            j=failure[j];//Optional
            stk[top++] = i-plen+1;
        }
    }
    return top;
}

int main(void){
    int succCount, i;
    int T;
    ios::sync_with_stdio(false);
    cin>>T;
    while(T--){
        cin >> pattern >> tstring;
        succCount = MorrisPratt(tstring, pattern, Pos);             
        cout << succCount << '\n';
        /*
        for(i=0; i<succCount; ++i){
            cout << Pos[i] << '\n';
        }
        */
    }
    return 0;
}

{% endhighlight %}

## hash

{% highlight cpp linenos=table %}

#include <iostream>
#include <string>
#include <cstdio>
#include <cstdlib>
#include <cstring>
#define STRL 1000010
#define PURGE(X) sscanf( (X) , "%s" , (X) )
using namespace std;
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

int solv() {
    int plen=strlen(pat);
    int slen=strlen(str);
    if(plen>slen) return 0;
    hsaPow = 1uLL;
    unsigned int pol = plen-1; //highest power
    UT temp = hsaBase;
    //Initialize fast-pow
    while(pol) {
        if(pol&1) hsaPow = (hsaPow*temp) % MOD1d7;
        temp = (temp * temp) % MOD1d7;
        pol>>=1;
    }
    hsaS = gethash(str, plen); hsaP = gethash(pat, plen);
    int ans = 0;
    for(int i=0 ;; ++i) {
        if(hsaS==hsaP) ++ans;
        if(i+plen==slen) break; //Warn
        hsaS = (hsaS-((UT)str[i]*hsaPow)%MOD1d7+MOD1d7) % MOD1d7;
        hsaS = ( (hsaS*hsaBase)%MOD1d7 + (UT)str[i+plen] ) % MOD1d7;
    }
    return ans;
}

int main(void) {
    int T;
    fgets(str, sizeof(str), stdin);
    sscanf(str, "%d", &T);
    while(T-- ) {
        fgets(pat, sizeof(pat), stdin);
        fgets(str, sizeof(str), stdin);
        PURGE(pat);
        PURGE(str);
        printf("%d\n",solv());
    }
    return 0;
}

{% endhighlight %}

