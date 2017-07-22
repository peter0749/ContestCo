---
layout: post
title: 漸層色彩 -- ZeroJudge b427
date: 2017-07-22 12:00
comments: true
external-url:
tags: image_processing
---

就是線性內插，很直觀。

## 結果

Before:

After:

Before:

After:

Before:

After:

## ZJ 題目傳送門

[漸層色彩](https://zerojudge.tw/ShowProblem?problemid=b427)

## 主要程式碼

{% highlight cpp linenos=table %}

#ifndef __INCLUDE_LINEAR_GRADIENT_CPP
#define __INCLUDE_LINEAR_GRADIENT_CPP
#include "LinearGradient.h"
#include <stdexcept>
#include <cmath>
namespace ImageProcess
{
    void LinearGradient::linear() { // mode 0, 平行的漸層
        double dist = out.img_W-1.0;
        PixDouble left(this->inner);
        PixDouble right(this->outer);
        for (int i=0; i<out.img_H; ++i) {
            for (int j=0; j<out.img_W; ++j) {
                if (out.img_W==1) {
                    this->out.setPixel(j, i, Pix(left, 45));
                    continue;
                }
                double dl=j;
                double dr=dist-dl;
                this->out.setPixel(j, i, Pix((left*dr + right*dl)/dist, 45));
            }
        }
    }
    void LinearGradient::radial() { // mode 1, 放射狀的漸層
        using std::hypot;
        #define DIST(a,b,c,d) std::sqrt( (a-c)*(a-c) + (b-d)*(b-d) )
        PixDouble left(this->inner);
        PixDouble right(this->outer);
        double midx = (double)(out.img_W-1) / 2.0;
        double midy = (double)(out.img_H-1) / 2.0;
        double radius = DIST(midx, midy, 0, 0);
        for (int i=0; i<out.img_H; ++i) {
            for (int j=0; j<out.img_W; ++j) {
                if (out.img_W==1) {
                    this->out.setPixel(j, i, Pix(left, 45));
                    continue;
                }
                double dl = DIST(j,i,midx,midy);
                this->out.setPixel(j, i, Pix((left*(radius-dl)+right*dl) / radius ,45));
            }
        }
    }
    void LinearGradient::alphaRadial() { // mode 2, ZJ 上最下面那張圖片的效果
        double midx = (double)(out.img_W-1) / 2.0;
        double midy = (double)(out.img_H-1) / 2.0;
        double radius = DIST(midx, midy, 0, 0);
        for (int i=0; i<out.img_H; ++i) {
            for (int j=0; j<out.img_W; ++j) {
                if (out.img_W==1) {
                    continue;
                }
                double dl = DIST(j,i,midx,midy);
                PixDouble pix(this->out.getPixel(j,i));
                pix.alpha *= (radius-dl) / radius;
                this->out.setPixel(j, i, Pix(pix ,45));
            }
        }
    }
    LinearGradient::LinearGradient(const Image &_input) : out(_input), mode(2) { }
    LinearGradient::LinearGradient(const size_t W, const size_t H, const Pix &_inner,\
            const Pix   &_outer, int mode) : out(W,H), inner(_inner), outer(_outer), mode(mode) {
        if (mode!=0&&mode!=1) throw std::runtime_error("in LinearGradient constructor: mode must 0/1");
    }
    Image* LinearGradient::run() {
        switch(this->mode) {
            case 0:
                this->linear();
                break;
            case 1:
                this->radial();
                break;
            case 2:
                this->alphaRadial();
                break;
        }
        return new Image(this->out);
    }
};

#endif

{% endhighlight %}

完整程式碼：[GitHub](https://github.com/peter0749/Image_processing_practice/tree/master/ZJb427)

ZJ 上 AC 程式碼：[GitHub](https://github.com/peter0749/Image_processing_practice/blob/master/ZJb427/zjb427.cpp)

