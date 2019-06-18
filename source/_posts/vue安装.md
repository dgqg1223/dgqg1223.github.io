---
title: vue环境安装
date: 2019-03-20 11:59:05
tags: vue
categories: 
    - 前端
    - vue
---

# 安装node.js
1. 下载安装node.js
2. 控制台查看版本信息
```
node -v
```
3. node集成了npm，查看npm版本信息
```
npm -v
```
4. 使用nrm修改npm源
```
npm install nrm -g
nrm ls
nrm test 
nrm use taobao
```
5.重启电脑
# 安装VUE
## 使用idea安装vue，
1. 新建idea “static Web”项目
2. 在idea控制台Treminal 初始化npm
```
npm init -y
```
3. 安装vue
```
npm install vue --save
```
4. 安装成功后在项目根目录中会出现"mode_modules"  vue目录