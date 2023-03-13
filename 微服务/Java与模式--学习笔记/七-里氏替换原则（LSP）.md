**一、什么是LSP**
简单来说，一个软件实体如果使用的是一个基类的话，那么一定适用于其子类，且察觉不出基类与子类对象的区别。

反过来的代换则不成立，即一个软件实体使用的是一个子类的话，那么它一定不适用于基类。

**二、从代码重构角度理解**
如果有两个具体类A和B之间关系违反了LSP。可以根据情况重构：
1.创建一个新的抽象类C，作为两个具体类的超类，将A和B的共同行为移到C中。
![](https://upload-images.jianshu.io/upload_images/9449419-51cfeba88a10d51b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.从B到A的继承关系改为委派关系。
![](https://upload-images.jianshu.io/upload_images/9449419-d17cd0e55b71e1b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
