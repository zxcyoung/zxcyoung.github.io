---
layout:     post
title:      QCustomPlot使用心得
subtitle:   QT第三方绘图库的使用
date:       2019-08-26
author:     CoffeeZ
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - 心得
    - QCustomPlot
---

# QCustomPlot

#### 1.简介

​	[QCustomPlot](https://www.qcustomplot.com/)是依赖于QT的第三方绘图库，除此之外还可以用QML的Canvas、QT自带的QChart绘图，三者在性能上QCustomPlot>QCharts>Canvas，QCustomPlot的源码只有2个文件，添加供自己的工程师很方便。

QCustomPlot的官方网址：[Qt Plotting Widget *QCustomPlot* - Introduction](https://www.qcustomplot.com/ )

#### 2.源码框架

1. ##### 对象类

   ​	我使用的是QCustomPlot 2.0版本，相较于1.3版本，他的优势在于实现了分层绘制，图标数据的分离，以及坐标轴刻度的计算。观察QCustomPlot的Class Relationship Diagram如下所示：

------


![class relationship](https://raw.githubusercontent.com/zxcyoung/zxcyoung.github.io/master/img/RelationOverview.png)
=======
------

​		1.一个图表(QCustomPlot)：包含一个或者多个图层、一个或多个item(用于展示一些固定的元素，例如文本、线段等)、一个或者多个可以绘制的元素、一个布局

​		2.一个图层(QCPLayer)：包含基本的元素(QCPLayerable)

​		3.一个QCPAbstractItem：包含一个或者多个位置信息

​		4.一个坐标轴矩形(QCPAxisRect):包含多个坐标轴、有一个图例类(多个图例项)

2. ##### 可绘制元素

   ​	在一个QCustomPlot类图中最重要、用的最多的是**QCPLayerable**元素。

   ------

   ![](https://raw.githubusercontent.com/zxcyoung/zxcyoung.github.io/master/img/InheritanceOverview.png)
=======
------

   ​	1.QCPAbstractPlottable：绘图元素，包含：折线图(QCPGraph)、曲线图(QCPCurve)、柱状图(QCPBars)、QCPStatiBox(盒子图)、QCPColorMap(色谱图)、QCPFinancial(金融图)

   ​	2.QCPAbstractItem：标示项，包含：直线(QCPItemStraightLine)、线段(QCPItemLine)、曲线(QCPItemCurve)、矩形(QCPItemRect)、椭圆(QCPItemEllipse)、文本(QCPItemText)、小圆球(QCPItemTracer)、图片(QCPItemPixmap)、括弧(QCPItemBracket)

   ​	3.布局项(QCPLayoutElement)：布局项(QCPAbstractLegendItem)、坐标轴矩形(QCPAxisRect)

   ​	4.网格线(QCPGrid)：每一个坐标轴对应一个网格线

   ​	5.坐标轴(QCPAxis)：一个坐标轴矩形（QCPAxisRect）包含四个坐标轴，上下左右四个坐标轴。

   #### 3.程序示例代码

   QCPItemText:

   ```c++
   void CustomPlotItem::initTimeText()
   {
       QColor clrDef(Qt::black);
       QBrush brushDef(Qt::NoBrush);
       QPen penDef(clrDef);
       timetext = new QCPItemText(getPlot());
       timetext->setClipToAxisRect(false);
       timetext->setPadding(QMargins(3, 3, 3, 3));
       timetext->setBrush(brushDef);
       timetext->setPen(penDef);
       timetext->position->setAxes(m_map[m_list.at(0)]->axis(QCPAxis::atBottom)
               ,m_map[m_list.at(0)]->axis(QCPAxis::atLeft));
       timetext->position->setTypeX(QCPItemPosition::ptAxisRectRatio);
       timetext->position->setTypeY(QCPItemPosition::ptAxisRectRatio);
       timetext->setPositionAlignment(Qt::AlignTop | Qt::AlignHCenter);
       timetext->setFont(QFont("Arial", 5));
       timetext->position->setCoords(0.85,0);
       timetext->setText("");
   }
   ```

   应用布局建立坐标轴及Graph：

   ```c++
   void CustomPlotItem::update(QString signal)
   {
           QMargins marg(10,0,0,0);
           m_map[signal]=new QCPAxisRect(getPlot());
           m_allqueue[signal]=new QQueue<QPoint>();
           getPlot()->plotLayout()->setColumnSpacing(0);
           m_map[signal]->axis(QCPAxis::atLeft)->setTickLabelSide(QCPAxis::lsInside);
           m_map[signal]->setMinimumMargins(marg);
           QFont ticklabelfont = getPlot()->font();
           ticklabelfont.setPointSize(5);
           m_map[signal]->axis(QCPAxis::atLeft)->setTickLabelFont(ticklabelfont);
           m_map[signal]->axis(QCPAxis::atBottom)->setTickLabelFont(ticklabelfont);
           m_map[signal]->axis(QCPAxis::atBottom)->setRange(0,30);
           getPlot()->plotLayout()->addElement(m_list.size()-1, 0, m_map[signal]);
           QCPGraph *newgraph = getPlot()->addGraph(m_map[signal]->axis(QCPAxis::atBottom)
                               , m_map[signal]->axis(QCPAxis::atLeft));
           m_graphs.append(newgraph);
           QFont leftlabelfont=getPlot()->font();
           leftlabelfont.setPointSize(6);
           m_map[signal]->axis(QCPAxis::atLeft)->setLabelFont(leftlabelfont);
           m_map[signal]->axis(QCPAxis::atLeft)->setLabel(signal.split(":").at(0));
           QSharedPointer<QCPAxisTickerTime> timeTicker(new QCPAxisTickerTime);
           timeTicker->setTimeFormat("%h:%m:%s:%z");
           getPlot()->plotLayout()->simplify();
   }
   ```

   鼠标移动槽函数：

   ```c++
   //鼠标坐标转化为CustomPlot内部坐标
   float x_val = customPlot->xAxis->pixelToCoord(x_pos);
   float y_val = customPlot->yAxis->pixelToCoord(y_pos);
   //获得x轴坐标位置对应的曲线上y的值
   float line_y_val = customPlot->graph(0)->data()->at(x_val)->value;
   //曲线的上点坐标位置，用来显示QToolTip提示框
   float out_x = customPlot->xAxis->coordToPixel(x_val);
   float out_y = customPlot->yAxis->coordToPixel(line_y_val);
    
   QString str,strToolTip;
   str = QString::number(x_val,10,3);
   strToolTip += "x: ";
   strToolTip += str;
   strToolTip += "\n";
    
   str = QString::number(line_y_val,10,3);
   strToolTip += "y: ";
   strToolTip += str;
   strToolTip += "\n";
    
   QToolTip::showText(mapToGlobal(QPoint(out_x,out_y)),strToolTip,customPlot);
   ```
