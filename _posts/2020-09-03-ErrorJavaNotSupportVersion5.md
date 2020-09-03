---
layout: post
title:  "Intellij idea 报错：Error : java 不支持发行版本5"
tags:   Environment Compile
date:   2020-09-03 11:19:00 +0800
categories: [Idea]
---

- 问题描述

  Intellij idea 报错：Error : java 不支持发行版本5

- 解决过程

  - 1：修改Moudles下的level与本地一致

    `File > Project Structure > Project Settings > Modules`

    ![image-20200903112209235](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2020-09-03/image-20200903112209235.png)

  - 2：修改Java Compiler下的version与本地一致

    `File > Settings > Build,Execution,Deployment > Compiler > Java Compiler`

    ![image-20200903122557481](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2020-09-03/image-20200903122557481.png)