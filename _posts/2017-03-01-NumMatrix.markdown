---
layout: post
title: LeetCode 304. Range Sum Query 2D - Immutable
date: 2017-03-01 14:33
comments: true
external-url:
categories: water
---

Range Sum Query 2D - Immutable

Difficulty: Medium

## 題目敘述
求矩陣內一塊矩形範圍的和

## 傳送門
[LeetCode 304](https://leetcode.com/problems/range-sum-query-2d-immutable/?tab=Description)

## 輸入輸出
Leet Code 比較特別，有點像是要你完成一個 Function。

## 限制
它好像沒給（？）

## 程式碼:
最近初學 Java ，剛好可以練習一下。

提供 Build: O(nm), Query: O(1) 的解法，這題 Query: O(n) 也可以過。

{% highlight java linenos=table %}

public class NumMatrix {
    private int [][] prefix;
    public NumMatrix(int[][] matrix) {
        prefix = null;
        if(matrix.length==0) return;
        prefix = new int[matrix.length+1][matrix[0].length+1];
        for(int i=1; i!=prefix.length; ++i) {
            int sum=0;
            for(int j=1; j!=prefix[0].length; ++j) {
                sum+=matrix[i-1][j-1];
                prefix[i][j] = prefix[i-1][j]+sum;
            }
        }
    }
    // Build: O(nm); Query: O(1)
    public int sumRegion(int row1, int col1, int row2, int col2) {
        if(prefix==null) return 0;
        int sums = ( prefix[row2+1][col2+1]-prefix[row1][col2+1]-prefix[row2+1][col1]+prefix[row1][col1] );
        return sums;
    }
}

{% endhighlight %}
