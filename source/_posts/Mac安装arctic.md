---
title: Mac安装arctic
date: 2016-10-11 11:27:35
tags: 
- 编程
- Python
---

arctic是一个python下的tick数据储存工具，基于pandas和mongodb，可以快速将大量时间序列数据压缩存储，很适合存放期货tick数据。然而，这次在mac上安装arctic却出现了些小问题。

直接按照官网的教程，安装：
```
> pip install git+https://github.com/manahl/arctic.git
```
会出现包含以下的错误提示:
```
clang: error: unsupported option '-fopenmp'
clang: error: unsupported option '-fopenmp'
error: command 'cc' failed with exit status 1
```
查了一下，原因是Mac默认的clang编译器不支持openMP，只能用brew另外装一个了：
```
> brew install gcc
```
安装完了gcc-6，在/usr/local/bin下，如何让python在安装的时候自动链接到新的编译器呢？
修改setup.py，在最上方加上
```python
import os
os.environ["CC"] = "gcc-6"
os.environ["CXX"] = "g++-6"
```
就会自动使用gcc-6编译。

依旧出现问题：
```
gcc-6: warning: x86_64 conflicts with i386 (arch flags ignored)
gcc-6: error: unrecognized command line option '-Wshorten-64-to-32'
```
原因是在编译的时候Python自动加上了`-Wshorten-64-to-32`这个选项，而这个选项是Apple给自己的编译器加的，正常的编译器不支持这个选项。
可以运行命令
```
> python setup.py -n build
```
会打印出实际运行的编译命令:
```
gcc-6 -fno-strict-aliasing -fno-common -dynamic -arch i386 -arch x86_64 -g -Os -pipe -fno-common -fno-strict-aliasing -fwrapv -DENABLE_DTRACE -DMACOSX -DNDEBUG -Wall -Wstrict-prototypes -Wshorten-64-to-32 -DNDEBUG -g -fwrapv -Os -Wall -Wstrict-prototypes -DENABLE_DTRACE -arch i386 -arch x86_64 -pipe -I/System/Library/Frameworks/Python.framework/Versions/2.7/include/python2.7 -c src/_compress.c -o build/temp.macosx-10.12-intel-2.7/src/_compress.o -fopenmp
gcc-6 -fno-strict-aliasing -fno-common -dynamic -arch i386 -arch x86_64 -g -Os -pipe -fno-common -fno-strict-aliasing -fwrapv -DENABLE_DTRACE -DMACOSX -DNDEBUG -Wall -Wstrict-prototypes -Wshorten-64-to-32 -DNDEBUG -g -fwrapv -O s-Wall -Wstrict-prototypes -DENABLE_DTRACE -arch i386 -arch x86_64 -pipe -I/System/Library/Frameworks/Python.framework/Versions/2.7/include/python2.7 -c src/lz4.c -o build/temp.macosx-10.12-intel-2.7/src/lz4.o -fopenmp
gcc-6 -fno-strict-aliasing -fno-common -dynamic -arch i386 -arch x86_64 -g -Os -pipe -fno-common -fno-strict-aliasing -fwrapv -DENABLE_DTRACE -DMACOSX -DNDEBUG -Wall -Wstrict-prototypes -Wshorten-64-to-32 -DNDEBUG -g -fwrapv -Os -Wall -Wstrict-prototypes -DENABLE_DTRACE -arch i386 -arch x86_64 -pipe -I/System/Library/Frameworks/Python.framework/Versions/2.7/include/python2.7 -c src/lz4hc.c -o build/temp.macosx-10.12-intel-2.7/src/lz4hc.o -fopenmp
gcc-6 -bundle -undefined dynamic_lookup -arch i386 -arch x86_64 -Wl,-F. build/temp.macosx-10.12-intel-2.7/src/_compress.o build/temp.macosx-10.12-intel-2.7/src/lz4.o build/temp.macosx-10.12-intel-2.7/src/lz4hc.o -o build/lib.macosx-10.12-intel-2.7/arctic/_compress.so -fopenmp
```
把其中所有的`-Wshorten-64-to-32`删除，并在sudo下运行，最后运行
> python setup.py install


就成功搞定啦！
