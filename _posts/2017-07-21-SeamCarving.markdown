---
layout: post
title: Seam Carving -- ZeroJudge b438
date: 2017-07-21 18:00
comments: true
external-url:
tags: image_processing
---

這次來實作一下上上學期多媒體技術導論，

聽過的 Seam Carving 方法吧。

Seam Carving 簡單來說，就是在影像中，由上到下（或左到右），找到一條能量總和最小的路徑，然後把它刪掉，

刪掉一次，影像寬度（或高度）會少 1 ，以此達到影像縮放的功能，這種方法不會太大影響影像中的物件比例。

為什麼這種方法比較不會影響影像中的物件比例呢？因為能量（梯度）較低的像素，比較不會出現在邊界上，我們也比較難查覺出，有些像素被移除了。

考慮到計算複雜度問題，要一次找出 n 條不重疊、能量加總最小的路徑太花時間，

只好採用一點 greedy 的策略，每次只刪去目前能量最小的 1 個路徑，做 n 回合，來達成圖片縮放的功能。

## ZJ 題目傳送門

[寫這題需要知道的梯度算法](https://zerojudge.tw/ShowProblem?problemid=b436)

[Seam Carving](https://zerojudge.tw/ShowProblem?problemid=b438)

## 結果

實際使用的 code 與 ZJ 上的 code 有點差異。

另外，不小心沒控制好記憶體的使用，可能以後有時間再修吧 ~~（其實就是不想修？）~~

Before: 

![img1](https://github.com/peter0749/Image_processing_practice/raw/master/ZJb438_SeamCarving/Lenna.png)

After:

![img2](https://github.com/peter0749/Image_processing_practice/raw/master/ZJb438_SeamCarving/Lenna_to.png)

（鬥雞眼！？）

## 主要程式碼

`EnergyMap.h` :

{% highlight cpp linenos=table %}

#ifndef __INCLUDE_ENERGYMAP_HEADER
#define __INCLUDE_ENERGYMAP_HEADER
#include "utils.h"
#include <cmath>

namespace ImageProcess 
{
    class EnergyMap {
        private:
            Image simg;
            double getEnergyMap(int x, int y);
        public:
            EnergyMap(const Image &img);
            double* run(void);
            static const double xMask[3][3], yMask[3][3];
            ~EnergyMap();
    };
};

#endif

{% endhighlight %}

`EnergyMap.cpp` :

{% highlight cpp linenos=table %}

#ifndef __INCLUDE_ENERGYMAP_CPP
#define __INCLUDE_ENERGYMAP_CPP
#include "utils.h"
#include "EnergyMap.h"
#include <cmath>

namespace ImageProcess
{
    const double EnergyMap::xMask[3][3] = {
        {-1, 0, 1},
        {-2, 0, 2},
        {-1, 0, 1}
    };
    const double EnergyMap::yMask[3][3] = {
        {-1, -2, -1},
        { 0,  0,  0},
        { 1,  2,  1}
    };
    double EnergyMap::getEnergyMap(int x, int y) {
        PixDouble sx(0,0,0,0);
        PixDouble sy(0,0,0,0);
        PixDouble O(0,0,0,0);
        for (int i=0; i<3; ++i) {
            for (int j=0; j<3; ++j) {
                PixDouble t(simg.getPixel(x+j-1,y+i-1));
                sx = sx + (t*xMask[i][j]);
                sy = sy + (t*yMask[i][j]);
            }
        }
        sx = sx - O;
        sy = sy - O;
        O  = sx + sy;
        return O.r+O.g+O.b+O.alpha;
    }
    EnergyMap::EnergyMap(const Image &img) : simg(img.img_W+2, img.img_H+2) {
        /* paddle borders */
        for (int i=0; i<img.img_H; ++i) {
            simg.setPixel(0, i+1, img.getPixel(0, i));
            simg.setPixel(simg.img_W-1, i+1, img.getPixel(img.img_W-1, i));
            for (int j=0; j<img.img_W; ++j) {
                simg.setPixel(j+1, i+1, img.getPixel(j,i));
            }
        }
        for (int i=0; i<img.img_W; ++i) {
            simg.setPixel(i+1, 0, img.getPixel(i, 0));
            simg.setPixel(i+1, simg.img_H-1, img.getPixel(i, img.img_H-1));
        }

        /*paddle corners*/
        simg.setPixel(0,0, img.getPixel(0,0));
        simg.setPixel(0,simg.img_H-1, img.getPixel(0,img.img_H-1));
        simg.setPixel(simg.img_W-1,simg.img_H-1, img.getPixel(img.img_W-1,img.img_H-1));
        simg.setPixel(simg.img_W-1,0, img.getPixel(img.img_W-1,0));
    }
    double* EnergyMap::run(void) {
        double *grad = NULL;
        grad = new double[(simg.img_H-2)*(simg.img_W-2)];
        for (int i=1; i<simg.img_H-1; ++i) {
            for (int j=1; j<simg.img_W-1; ++j) {
                grad[(i-1)*(simg.img_W-2) + (j-1)] = getEnergyMap(j,i);
            }
        }
        return grad;
    }
    EnergyMap::~EnergyMap() {}
};
#endif
{% endhighlight %}

`SeamCarve.h` :

{% highlight cpp linenos=table %}

#ifndef __INCLUDE_SEAMCURVE_HEADER
#define __INCLUDE_SEAMCURVE_HEADER
#include "utils.h"
#include "EnergyMap.h"
namespace ImageProcess
{
    class SeamCarve {
        private:
            double **dp;
            int **bt;
            EnergyMap grad; // real computation
            const Image &simg; // just a reference
            Image timg;
            void dynamicCarve();
        public:
            SeamCarve(const Image &img);
            ~SeamCarve();
            Image* run(void);
    };
};
#endif

{% endhighlight %}

`SeamCarve.cpp`

{% highlight cpp linenos=table %}

#ifndef __INCLUDE_SEAMCARVE_CPP
#define __INCLUDE_SEAMCARVE_CPP
#include "utils.h"
#include "EnergyMap.h"
#include "SeamCarve.h"
namespace ImageProcess 
{
    void SeamCarve::dynamicCarve() {
        double *gradMap = NULL;
        gradMap = grad.run();
        for (int i=0; i<simg.img_H; ++i) 
            dp[i][0] = dp[i][simg.img_W+1] = 1e200;
        for (int i=1; i<=simg.img_W; ++i)
            dp[0][i] = gradMap[i-1];
        for (int i=1; i<simg.img_H; ++i) {
            for (int j=1; j<=simg.img_W; ++j) {
                int k=-1, minidx=-1;
                double minv=1e201;
                for (; k<=1; ++k) {
                    if (dp[i-1][j+k]<minv) {
                        minv = dp[i-1][j+k];
                        minidx = k;
                    }
                }
                dp[i][j] = minv + gradMap[(i*simg.img_W)+(j-1)];
                bt[i][j] = minidx;
            }
        }
        if (gradMap!=NULL) { delete gradMap; gradMap=NULL; }
    }
    SeamCarve::SeamCarve(const Image &img): simg(img), grad(img), timg(img.img_W-1, img.img_H) {
        dp = new double*[img.img_H];
        bt = new int*[img.img_H];
        for (int i=0; i<img.img_H; ++i) {
            dp[i] = new double[img.img_W+2];
            bt[i] = new int[img.img_W+2];
        }
    }
    SeamCarve::~SeamCarve() {
        for (int i=0; i<simg.img_H; ++i) {
            delete[] dp[i]; dp[i]=NULL;
            delete[] bt[i]; bt[i]=NULL;
        }
        delete[] dp; delete[] bt;
        dp=NULL;
        bt=NULL;
    }
    Image* SeamCarve::run(void) {
        int nInf=-1e9;
        this->dynamicCarve();
        double minval=1e300;
        int mindp=1;
        for (int i=1; i<=simg.img_W; ++i) {
            if (dp[simg.img_H-1][i]<minval) {
                minval = dp[simg.img_H-1][i];
                mindp  = i;
            }
        }
        for (int i=simg.img_H-1; i>=0; --i) {
            int last = bt[i][mindp];
            bt[i][mindp] = nInf; 
            mindp+=last;
        }
        for (int i=0; i<simg.img_H; ++i) {
            int k=0;
            for (int j=0; j<simg.img_W; ++j) {
                if (bt[i][j+1]==nInf) continue; // skip deleted pixel
                timg.setPixel(k++,i,simg.getPixel(j,i));
            }
        }
        return new Image(this->timg);
    }
};
#endif

{% endhighlight %}

完整程式碼：[GitHub](https://github.com/peter0749/Image_processing_practice/tree/master/ZJb438_SeamCarving)

ZJ 上 AC 程式碼：[GitHub](https://github.com/peter0749/Image_processing_practice/blob/master/ZJb438_SeamCarving/zjb438.cpp)

