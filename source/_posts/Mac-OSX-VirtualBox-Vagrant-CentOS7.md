---
title: Mac OSX+VirtualBox+Vagrant+CentOS7
date: 2016-12-24 12:33:06
tags: VirtualBox
---
> 本来想在我阿里服务器上搭建自己的一套微服务架构，可惜我是一个穷人，
> 现只有用OSX+VirtualBox+Vagrant+CentOS7在本地先实现自己的一套本地服务

## 1.安装VirtualBox
  [VirtualBox下载地址](https://www.virtualbox.org/wiki/Downloads) 根据自己机器系统选择下载包

## 2.安装Vagrant
  1) [Vagrant](https://www.vagrantup.com/downloads.html)下载地址 选择下载包

  安装完成后，在终端输入
  ```
  vagrant -v
  ```
  2）下载Vagrant官方封装好的[系统镜像](http://www.vagrantbox.es/)
    因为我在阿里云上主要用的是Centos7，这边我使用centos7.box

  3) 添加下载好的box系统镜像到Vagrant
  ```
  vagrant box add centos7 /path/centos7.box
  ```
  centos7 名称可能自己设置

## 3.配置开发环境
  1）创建开发目录
  ```
  cd ~                    # 切换目录

  mkdir test_vagrant      # 创建文件夹

  cd test                 # 切换目录
  ```
  2）初始化开发环境
  在终端中输入  
  ```
  vagrant init centos7         #初始化
  ```
  3）启动开发环境
  在终端中输入
  ```
  vagrant up        # 启动环境
  ```
