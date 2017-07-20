---
layout: post
title: 圖像縮放 -- Bilinear Interpolation, Nearest Neighbor Interpolation
date: 2017-07-20 23:00
comments: true
external-url:
tags: image_processing
---

難得這次暑假很長，

再來練習個簡單的圖像縮放，

使用的方法有 

Bilinear Interpolation (雙線性內插)

和 

Nearest Neighbor Interpolation (最近鄰居內插)

## Bilinear Interpolation

Bilinear Interpolation 的方法：

找到影像縮放後，新像素點在原始影像對應的位置，並根據這個位置它左上、左下、右上、右下的臨近四個像素做兩次線性內插。

第一次內插是利用左上、右上；左下、右下分別做一次內插，這次內插會得到兩個點，一上一下。

再根據這兩個點，做一次內插，會得到新像素在新位置的 R,B,G,A 值。

## Nearest Neighbor Interpolation

Nearest Neighbor Interpolation 的方法：

找到影像縮放後，新像素點在原始影像對應的位置，在該位置左上、左下、右上、右下的臨近四個點中，找到距離新像素點最近的的點，並設定新像素點的值為該點的值。

## 結果

Before: 

![img1](https://github.com/peter0749/Image_processing_practice/raw/master/ZJb424_scaling/miku.png)

After:

![img2](https://github.com/peter0749/Image_processing_practice/raw/master/ZJb424_scaling/miku_big.png)

放大

![img3](https://github.com/peter0749/Image_processing_practice/raw/master/ZJb424_scaling/miku_small.png)

縮小

![img4](https://github.com/peter0749/Image_processing_practice/raw/master/ZJb424_scaling/miku_change_ratio.png)

不照比例縮放

上面的例子都是使用 Bilinear Interpolation 做成的，
Nearest Neighbor Interpolation 方法跑出來的結果就不放上來了。

## 主要程式碼

{% highlight cpp linenos=table %}

#ifndef __INCLUDE_UTILS_CLASS
#define __INCLUDE_UTILS_CLASS
//#include <stdexcept>
//#include <functional>
//#include <cmath>
#include "utils.h"
#include "Scale.h"

namespace ImageProcess
{
    Scale::Scale(double _scaleX, double _scaleY, int mode, const Image *img) : \
        mode(mode), sdata(img), tdata(new Image((size_t)(_scaleX*(double)img->img_W+0.5), (size_t)(_scaleY*(double)img->img_H+0.5))) {
            if (_scaleX<0||_scaleY<0) throw std::runtime_error("scale ratio is negative");
            if (mode!=0 && mode!=1) throw std::runtime_error("mode is not 0/1");
            if (this->sdata==NULL) throw std::runtime_error("sdata is NULL");
            if (this->tdata==NULL) throw std::runtime_error("tdata is NULL");
            scaleX = (double)this->sdata->img_W / (double)this->tdata->img_W;
            scaleY = (double)this->sdata->img_H / (double)this->tdata->img_H;
        }
    Pix Scale::bilinear(int x, int y) {
        double midX = x*scaleX, midY = y*scaleY;
        int left = midX; int right = left+1; int up = midY; int down = up+1;
        double dl = midX-left, dr = right-midX;
        double du = midY-up, dd = down-midY;

        PixDouble upperLeft(this->sdata->getPixel(left, up));

        PixDouble upperRight = (right>=this->sdata->img_W)?upperLeft:PixDouble(this->sdata->getPixel(right,up));
        PixDouble upperMid   = upperLeft*dr + upperRight*dl; // interpolation on upper pixels

        if (down>=this->sdata->img_H) return Pix(upperMid);  // no lower pixels

        /* Binary Interpolation (lower pixels)*/
        PixDouble downLeft(this->sdata->getPixel(left,down));
        PixDouble downRight = (right>=this->sdata->img_W)?downLeft:PixDouble(this->sdata->getPixel(right, down));
        PixDouble downMid = downLeft*dr + downRight*dl;

        PixDouble Mid = downMid*du + upperMid*dd;

        return Pix(Mid);
    }
    Pix Scale::nearest (int x, int y) {
        using std::pair;
        using std::fabs;
        double midX = x*scaleX, midY = y*scaleY;
        int left = midX, up = midY;
        int dirs[4][2] = {
            {0, 0},
            {1, 0},
            {0, 1},
            {1, 1}
        };
        double minDist=1e300;
        Pix ret;
        for (int i=0; i<4; ++i) {
            int nx = left+dirs[i][0];
            int ny = up+dirs[i][1];
            if (nx<0||ny<0||nx>=this->sdata->img_W||ny>=this->sdata->img_H) continue;
            double dx = midX-(double)nx;
            double dy = midY-(double)ny;
            double dist = dx*dx + dy*dy;
            if (dist<minDist) {
                minDist = dist;
                ret = this->sdata->getPixel(nx,ny);
            }
        }
        return ret;
    }
    void Scale::biLinearScaling() {
        // pass
        for (int i=0; i<tdata->img_H; ++i) {
            for (int j=0; j<tdata->img_W; ++j) {
                Pix pixel = this->bilinear(j, i);
                this->tdata->setPixel(j, i, pixel);
            }
        }
    }
    void Scale::nearestScaling() {
        // pass
        for (int i=0; i<tdata->img_H; ++i) {
            for (int j=0; j<tdata->img_W; ++j) {
                Pix pixel = this->nearest(j, i);
                this->tdata->setPixel(j, i, pixel);
            }
        }
    }

    Image* Scale::run(void) {
        switch(this->mode) {
            case 0:
                this->biLinearScaling();
                break;
            case 1:
                this->nearestScaling();
                break;
        }
        //return new Image(*(this->tdata));
        return new Image(*(this->tdata));
    }
    Scale::~Scale() {
        delete this->tdata;
    }
};

#endif

{% endhighlight %}

完整程式碼：[GitHub](https://github.com/peter0749/Image_processing_practice/tree/master/ZJb424_scaling)
