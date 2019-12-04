---
title:      简要说明程序框架
date:       2019-08-28
---

# 简介

​	整个程序基于ViCANdo与QT开发，基于ViCANdo官方SDK例程HellowordPresenter， 绘图库采用QT第三方绘图库[QCustomPlot](https://www.qcustomplot.com/)，其[官方文档](https://www.qcustomplot.com/documentation/index.html)说明了详细的结构以及使用方法，也可安装其说明文档到QT，具体方法请自行百度~。整个程序编译称dll插件供ViCANdo调用。

# 程序流程

	1. QML文件（DBCansys）提供dbc文件及ViCANdo project source列表，选择dbc及source后，qml中的SourceEventListener会读取收到的CAN报文；dbc文件的解析通过程序中js文件实现，在DBCAnsys中程序见下面程序。

```javascript
SourceEventListener {
                    id:sourceevent
                    source: source1
                    dbc: dbc1
                    filter: flitermsglist
                    onMessageReceived: {} ...
```

```js
import "matchlist.js" as Maol
	Maol.toplist(Maol.matlisnodes(dbc1),Maol.matlisignal(dbc1.messages));
```

 2. QML将读到的CAN报文信息发送到C++程序（qmlplot.cpp），关于每个变量的commen在程序已注明。qmlplot将受到的信息以队列形式存在C++中Queue中，当队列内容大于某一值时清空队列；收到的CAN报文以addData的形式添加到qcustomplot实例化后的对象中。

 3. qmlplot处理受到的CAN报文，按照指定显示方式呈现，最后QML文件（DBCAnsys）import qmlpolt注册到QML中的对象并显示。

    注册内容在readdata.cpp中，程序如下：

```javascript
ReadData::ReadData(VxProject* project, VxSource* source)
    : VxQuickBasePresenter(project, source, QUrl("qrc:/qml/DBCansys.qml"))
{
    qmlRegisterType<CustomPlotItem>("CustomPlot", 1, 0, "CustomPlotItem");
}
```

​	需要在QML中import并引用：

```javascript
import CustomPlot 1.0
CustomPlotItem {
                        id: customPlot
                        width:parent.width
                        height:parent.height
                        Component.onCompleted: {
                            initCustomPlot();
                        }
                    }
```

# 程序释义

## 1. QML

 1. DBCAnsys

     此文件是显示的主文件，在readdata中引用（见下程序），程序主要分为三个部分：（1）登陆界面（LoginInterface.qml），选择dbc文件以及ViCANdo source;(2)DBC信号显示及选择界面（PoupView）；（3）显示界面，此界面有三部分组成，① 顶层功能按键（TopMenu.qml）② 左侧已选择信号（LeftSigList.qml）③右侧显示CAN曲线界面（在主文件DBCAnsys.qml中）

	2. PoupView

    此文件引用了TreeView，将读取到的DBC信息以树列表的形式显示在界面中，数据信息以json的格式传入，在DBC中以模型的形式呈现。

## 2. readdata

​	继承自ViCANdo自带SDK中的头文件VxQuickBasePresenter，想修改显示文本见cpp文件：

```c++
VxPresenter* ReadDataDescriptor::createInstance(VxProject* project, VxSource* source)
{
    project->logMessage("this is the can data curve!");
    return new ReadData(project, source);
}
VxPresenterDescriptor* ReadData::getDescriptor()
{
    /* The unique key/ID of the descriptor of this presenter in ViCANdo */
    return VxPresenterFactory::instance()->getDescriptor("can-data-curve");
}
```

## 3.XxwTracer

​	继承自QObject，包含两个类，分别是XxwTracer和XxwTraceLine，XxwTracer显示曲线坐标文本，包含箭头文本框等，XxwTraceLine显示坐标点的横坐向直线，即为图像中的Cursor。

## 4.qmlplot

​	继承自QQuickPaintedItem，主要是将C++中QPainter所画线转换为QML图像，文件包含两个类，分别是DDuiBasePlotItem和CustomPlotItem。DDuiBasePlotItem主要处理鼠标事件和显示框大小更改后的同步更新，下面程序展示了鼠标事件的转换过程。

```c++
void DDuiBasePlotItem::mousePressEvent( QMouseEvent* event )
{
    qDebug() << Q_FUNC_INFO;
    routeMouseEvents( event );
}
void DDuiBasePlotItem::routeMouseEvents( QMouseEvent* event )
{
    if (m_CustomPlot)
    {
        QMouseEvent* newEvent = new QMouseEvent( event->type(), event->localPos(), event->button(), event->buttons(), event->modifiers());
        QCoreApplication::postEvent( m_CustomPlot, newEvent );
    }
}
```

​	CustomPlotItem为绘图的主要界面，其中的getPlot（）继承自DDuiBasePlotItem的QCustomplot私有参数m_qcustomplot。

# 参考链接

1. [qcustomplot官网](https://www.qcustomplot.com/)
2. [qcustomplot参考文档](https://www.qcustomplot.com/documentation/index.html )
3. [个人使用心得](https://zxcyoung.github.io/2019/08/26/QCustomPlot%E4%BD%BF%E7%94%A8%E5%BF%83%E5%BE%97/ )
4. [qcustomplot使用分享](https://www.cnblogs.com/swarmbees/category/908110.html )
5. [qcustomplot绘图整理](https://blog.csdn.net/qq_20088499/article/details/83026005 )
6. [ViCANdo Developers’ manual](https://www.zuragon.com/docs/sdkmanual/ViCANdoSdkDoc.html )
7. [QT与qml混合编程](https://blog.csdn.net/foruok/article/details/32698603 )
8. [数据类型转换](https://my.oschina.net/TemetNosce/blog/872807 )
