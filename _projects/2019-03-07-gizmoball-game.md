---
title: 'GizmoBall Game'
image: 
  path: /images/projects/gizmoball/gizmoball_cover.jpg
  thumbnail: /images/projects/gizmoball/gizmoball_cover.jpg
author: Marty Pang
last_modified_at: 2019-03-07T15:44:28-05:00
---

Gizmoball是MIT Opencourseware软件工程的课程项目，课程链接[点我~](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-170-laboratory-in-software-engineering-fall-2005/)。Gizmoball是一种弹球游戏，与小时候风靡一段时间的三维弹球类似。与传统弹球游戏不同的是，Gizmoball允许用户自定义创建layout。用户可以任意得添加诸如bumper，flipper和absorber这类小物件儿（gizmo）。

# Overview

Gizmoball界面有`build`和`run`两种模式。
当用户处于`build`模式时：
- 在游戏界面创建和编辑正方形，圆形，梯形的bumper；
- 在游戏界面创建和编辑flipper；
- 将flipper，bumper的触发与特定事件connect起来，包括按键触发，被小球撞击等；
- 游戏可以以配置文件的形式存档与重新载入；

下面两张截图图，左图是build模式的界面，右图是自定义完成的游戏界面。

<table>
    <tr>
        <td ><center><img src="/images/projects/gizmoball/build_mode.png" >图1  build mode </center></td>
        <td ><center><img src="/images/projects/gizmoball/layout.png"  >图2 layout</center></td>
    </tr>
</table>

当用户处于`run`模式时，用户只需根据设定玩即可。

![run_mode](/images/projects/gizmoball/run_mode.png){:  .align-center}

更多的实现细节以及按键说明见本人[GitHub仓库 - GizmoBallGame](https://github.com/MartyPang/GizmoBallGame)。