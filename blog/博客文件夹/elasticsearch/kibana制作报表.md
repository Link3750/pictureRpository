# kibana制作报表

## Visualize Library介绍

visualize是kibana提供的一个视图，在里面可以制作各种图和表格

![image-20220112145332002](https://raw.githubusercontent.com/Link3750/pictureRpository/main/pic/202301032043895.png)

## DashBoards介绍

dashBaords 是一个展示板，用于展示一个visualize集合。

## 数据准备

使用测试服务中fond_salesmen_performance索引的数据来进行演示

## 使用lens制作报表

## Lens介绍

Lens是kibana7.5版本以后新上线的一个报表制作工具，能在不深入学习kibana的情况下方便地制作一份报表。但由于刚上线不久，可能会出现一些bug或需要优化的地方。

## 报表制作

![image-20220117100108391](https://raw.githubusercontent.com/Link3750/pictureRpository/main/pic/202301032043725.png)

![image-20220117100807790](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220117100807790.png)

![image-20220117101721685](https://raw.githubusercontent.com/Link3750/pictureRpository/main/pic/202301032043396.png)

![image-20220117102924550](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220117102924550.png)

![image-20220117103219212](https://raw.githubusercontent.com/Link3750/pictureRpository/main/pic/202301032043924.png)

自定义聚合方式支持常用的运算符号（+-*/开方，乘方，绝对值等）但当前版本自定义聚合以后导出成csv表格的话会导致冗余字段的出现。而且自定义聚合后的字段无法在另一列中作为数据源使用。

制作完成后可以保存该表格。

![image-20220117105045855](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220117105045855.png)

保存完该表格后会跳转到DashBoard界面，再次点击保存则报表制作完成。

## 报表分享

点击share->Embed code

![image-20220117110524954](https://raw.githubusercontent.com/Link3750/pictureRpository/main/pic/202301032043237.png)

snapshot是以快照形式分享报表，不会展示后续修改；saved object保存后，后续报表的修改也能实时更新。include选项会将时间选择器、筛选条件等添加到报表的展示界面中

![image-20220117110614082](https://raw.githubusercontent.com/Link3750/pictureRpository/main/pic/202301032043673.png)

## 报表导出功能

![image-20220117110944157](https://raw.githubusercontent.com/Link3750/pictureRpository/main/pic/202301032043293.png)
