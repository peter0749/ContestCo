---
layout: post
title: 關於 Shared library
date: 2017-07-22 13:00
comments: true
external-url:
tags: gcc, g++
---

Shared library 是很方便的東西，

除了 Library 本身可重複利用，

因為主程式是在執行時，才將 Library link 進來，

也可以減少主程式佔的硬碟空間。

另外一個好處，如果主程式需要修改，但 Shared library 不用，

那麼只需要重新編譯主程式就好了，省去重新編譯 library 的時間。

## 建造一個 Shared library 

首先，先編譯好我們需要的物件檔 (object file)：

```
g++ -c -fPIC alice.cpp bob.cpp
```

在當前目錄下，應該會有 `alice.o`, `bob.o` 兩個物件檔

接下來，產生一個 Shared library:

```
g++ -shared -Wl,-soname,libXXX.so.1 -o libXXX.so.1.0.0 alice.o bob.o
```

`XXX`: 代表版本，第一個號碼不同時，通常表示兩個版號不相容。

`-Wl`: 代表告訴 linker 後面有參數。

`-soname`: 告訴 linker 這個 library 的 soname

如果 linker 沒有 `-soname` 選項時，

可以再去網路上找替代方案，例如在 Mac 上，這個選項叫 `install_name`

或者也可以用以下方式：

```
g++ -shared -o libfoo.so.1.0.0 alice.o bob.o
ln -s libfoo.so.1.0.0 libfoo.so.1
ln -s libfoo.so.1 libfoo.so
```

有些編譯器可以接受，但不一定能解決問題。

## 使用編譯好的 Shared library

記得還是要準備好 library 的 header ，

```
g++ main.cpp -I${LIB_HEADER} -L${LIB_PATH} -l${YOUR_LIB_NAME}
```

`${LIB_HEADER}`: 代表該 library 的 header 所在位置

`${LIB_PATH}`: 代表編譯好的 shared library 所在位置

`${YOUR_LIB_NAME}`: 代表這個 library 的名字，例如：`libFOO.so.xxx` 的名子就是 `FOO`

這樣就可以使用 shared library 了
