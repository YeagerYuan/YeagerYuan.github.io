---
title: 用AssetStudio获取Steam上Unity游戏资产
date: 2025-01-25 17:07:38
tags: unity
cover: pictures/mita01.png
---

首先下载AssetStudio，Github网址： [Releases · Perfare/AssetStudio](https://github.com/Perfare/AssetStudio/releases) 

我下载的是第二个，第一个不知道为啥没找到exe文件，也可以自己下了都试试。

![1737818347604](%E7%94%A8AssetStudio%E8%8E%B7%E5%8F%96Steam%E4%B8%8AUnity%E6%B8%B8%E6%88%8F%E8%B5%84%E4%BA%A7/1737818347604.png)

下载后解压，点击运行程序。

![1737818451546](%E7%94%A8AssetStudio%E8%8E%B7%E5%8F%96Steam%E4%B8%8AUnity%E6%B8%B8%E6%88%8F%E8%B5%84%E4%BA%A7/1737818451546.png)

打开steam，找到想要解包的游戏地址，复制（只能是Unity开发的游戏，有些游戏可能有保护也无法解包）。

![1737818607362](%E7%94%A8AssetStudio%E8%8E%B7%E5%8F%96Steam%E4%B8%8AUnity%E6%B8%B8%E6%88%8F%E8%B5%84%E4%BA%A7/1737818607362.png)

选择file->load folder.

![1737818663186](%E7%94%A8AssetStudio%E8%8E%B7%E5%8F%96Steam%E4%B8%8AUnity%E6%B8%B8%E6%88%8F%E8%B5%84%E4%BA%A7/1737818663186.png)

粘贴地址即可。

![1737818701130](%E7%94%A8AssetStudio%E8%8E%B7%E5%8F%96Steam%E4%B8%8AUnity%E6%B8%B8%E6%88%8F%E8%B5%84%E4%BA%A7/1737818701130.png)

解包过程比较长，会一直出现报错，点击确定即可。但是不点击的话似乎就不会继续解包下去。

![1737797842210](%E7%94%A8AssetStudio%E8%8E%B7%E5%8F%96Steam%E4%B8%8AUnity%E6%B8%B8%E6%88%8F%E8%B5%84%E4%BA%A7/1737797842210.png)

解包完成后，点击Asset List查看解包文件，Mesh的是模型文件，还有其它种类的资产都会在里面。

![1737802334773](%E7%94%A8AssetStudio%E8%8E%B7%E5%8F%96Steam%E4%B8%8AUnity%E6%B8%B8%E6%88%8F%E8%B5%84%E4%BA%A7/1737802334773.png)

点击Export可以导出全部资产到本地，或者指定资产导出。

![1737802295833](%E7%94%A8AssetStudio%E8%8E%B7%E5%8F%96Steam%E4%B8%8AUnity%E6%B8%B8%E6%88%8F%E8%B5%84%E4%BA%A7/1737802295833.png)

全部导出时间比较长，中间没有确定，挂机等待即可。

![1737818232014](%E7%94%A8AssetStudio%E8%8E%B7%E5%8F%96Steam%E4%B8%8AUnity%E6%B8%B8%E6%88%8F%E8%B5%84%E4%BA%A7/1737818232014.png)

导出目录如下(以米塔为例，xml文件是另外导出的资产目录)：

![1737818930610](%E7%94%A8AssetStudio%E8%8E%B7%E5%8F%96Steam%E4%B8%8AUnity%E6%B8%B8%E6%88%8F%E8%B5%84%E4%BA%A7/1737818930610.png)