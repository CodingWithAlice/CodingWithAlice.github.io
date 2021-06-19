---
layout:     post
title:     background属性的使用
subtitle:  
date:       2021-02-04
author:     
header-img: 
catalog: true
tags:
    - < CSS相关内容 >
typora-root-url: ..
---


# background属性的使用

写在前面，这篇主题是结合开发中遇到的问题，对backgroud样式进行一个系统了解学习

（虽然css一向不是我在开发中关注的重点，但是当我跳了几次坑之后，我觉得有必要整理复习一下，以后再也不想跳坑了）

如果你对这个属性非常之了解，可以出门左拐不送哟～

## 结论
推荐使用写法
```
background: url('http://...') no-repeat 0 0 / 100% 100%;
background: url('http://...') no-repeat center / 100% 100%;
```

## 背景图常见写法
``` css
backgroud:url('http://h0.beicdn.com/open202105/2079400e69e49fd1_600x525.png') no-repeat;
backgroud-size: 100% 100%;
```
例如在某工程中就已存在几百次使用
![image-20210204114248525](/../img/assets_2019/image-20210204114248525.png)

## 场景一：需要更换背景图时存在问题
当活动期需要更换背景图/特殊状态显示其他背景图时，开发人员会替换背景图
![image-20210204114327500](/../img/assets_2019/image-20210204114327500.png)
但是这里经常会被遗漏的是：backgroud属性是【所有相关属性】的缩写，当我们仅修改背景图片链接时，会覆盖 `background-size:100% 100%;` 这个属性，造成更换的背景图没有大小设置，在不同机型的表现不一致

### 常见解决方案
每次更换的时候都注意把 `background-size` 属性带上
![image-20210204114359491](/../img/assets_2019/image-20210204114359491.png)


## 场景二：标签 style 属性样式出错
版头需要根据接口返回数据透出不同背景图片：
我选择将背景图内嵌到标签的 `style` 样式中，其中 `infoImg` 默认为粉丝视角
![image-20210204114455285](/../img/assets_2019/image-20210204114455285.png)
粉丝视角展示正常
![image-20210204114510194](/../img/assets_2019/image-20210204114510194.png)
但是在vip视角展示异常，背景图片变形了
![image-20210204114528561](/../img/assets_2019/image-20210204114528561.png)

#### 结合之前的铺垫加上截图中框出的样式，可以发现就是 `background-size` 被覆盖了，这里不能再手动添加一次 `background-size` 

### 解决方案
利用 `background` 属性本就是简写，将 `background-size` 加入即可，但是为什么一定要写成 `0 0 / 100% 100%` 的形式呢？
![image-20210204114544218](/../img/assets_2019/image-20210204114544218.png)

## 系统复习 background 的使用及注意事项
直接看MDN即可，这次主看 
background-position：为每一个背景图片设置初始位置 [background-position](https://developer.mozilla.org/zh-CN/docs/Web/CSS/background-position)
background-size : 设置背景图片大小  [background-size](https://developer.mozilla.org/zh-CN/docs/Web/CSS/background-size)
![image-20210204114601844](/../img/assets_2019/image-20210204114601844.png)
background：[background](https://developer.mozilla.org/zh-CN/docs/Web/CSS/background)

![image-20210204114623975](/../img/assets_2019/image-20210204114623975.png)





