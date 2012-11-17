---
layout: post
title: "kanban in process"
date: 2012-11-17 11:16
comments: true
categories: [kanban,scrum,agile]
---

重新做了一块看板应用到一个新项目组，并执行三周，继续探索中...

## 简介

看板的三个主要原则：

* 可视化流程
* 限制工作的数量
* 改善周期

### 约定

* 所有原始需求放到redmine中
* 所有的Bug通过redmine提交
* 黄色便签代表 backlog/story/task
* 红色便签代表 bug
* 当前任务项贴上自己的磁贴标记
* 每周一收集上一周的工作数据
* 每天早上10点进行立会
* 按需召开产品分析会议
    * 看板上任务不够了
    * 功能优先级调整了
* 流程顺序，从上到下从左到右
* 按需发布，由产品负责人决定
* 每周一收集上周的WIP产生CFD(Cumulative Flow Graph)

### 需求分析会议上需要明确的内容

* 任务分解
* 设计需时
* 开发需时
* 按功能优先级制定冲刺目标

## 版本控制的约定

### 开发过程

* 基于gitflow的分支模型
* `master`分支设置为产品分支，并设置为保护模式，开发人员不能向该分支push代码
* `develop`分支设置为开发分支，开发人员每进行一个功能或bug修改需要新建分支操作并完成后合并回来

```
git flow feature start <feature_name>
git add <file>
git ci 
# and more ...
# need a help?
# git flow feature publish <feature_name>
# done!
git flow feature finish <feature_name>
git pull origin develop
git push origin develop
```

### 发布过程

* 产品人员决定需要发布时间和特性
* 所有列入发布范围的特性生成一个`release branch`待验收测试
* 所有未列入发布范围的功能或缺陷停止提交到`develop`
* 所有基于`release branch`的修改在该分支上进行
* 发布成功后，发布版本打上指定版本标签，`develop`允许开放提交

### 版本标签

版本标签以`x.y.z`标记，

- x 代表重大版本变更
- y 代表较多功能特性增加及变更
- z 代表针对现有功能做出变更


