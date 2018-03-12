---
title: centos6编译gcc
categories: linux
tags: 
- Linux
- gcc
---

# centos6编译gcc

## 相关软件下载
> wget http://172.16.11.56/download/tmp/gcc-4.8.5.tar.gz

> wget http://172.16.11.56/download/tmp/mpc-1.0.3.tar.gz
>
> wget http://172.16.11.56/download/tmp/mpfr-3.1.4.tar.gz
>
> wget http://172.16.11.56/download/tmp/gmp-5.1.3.tar.gz
>

## 安装gmp
```shell
tar zxvf gmp-5.1.3.tar.gz 
cd gmp-5.1.3
./configure --prefix=/usr/local/gmp
make
make install 
```

## 安装mpfr

```shell
tar zxvf mpfr-3.1.4.tar.gz
cd mpfr-3.1.4
./configure --prefix=/usr/local/mpfr  --with-gmp=/usr/local/gmp/
make && make install
```

## 安装mpc
```shell
tar zxvf mpc-1.0.3.tar.gz
cd mpc-1.0.3
./configure --prefix=/usr/local/mpc --with-gmp=/usr/local/gmp/ -with-mpfr=/usr/local/mpfr/
make && make install
```

## 编译gcc

> 设置环境变量
```shell
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/mpc/lib:/usr/local/mpfr/lib:/usr/local/gmp/lib
yum -y install glibc-devel*
```

```shell
tar zxvf gcc-4.8.5.tar.gz
cd gcc-4.8.5
mkdir gcc-build-4.8.5 && cd gcc-build-4.8.5
../configure -enable-checking=release -enable-languages=c,c++ -disable-multilib --with-gmp=/usr/local/gmp  --with-mpfr=/usr/local/mpfr  --with-mpc=/usr/local/mpc
make
make install
cp ./x86_64-unknown-linux-gnu/libstdc++-v3/src/.libs/libstdc++.so.6.0.19 /usr/lib64/
cd /usr/lib64/
rm libstdc++.so.6 
ln -s libstdc++.so.6.0.19 libstdc++.so.6
```