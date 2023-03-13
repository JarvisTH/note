该模式为一个接口提供缺省实现，这样子类型可以从这个缺省实现扩展，不必从原有接口扩展。

在很多情况下，必须让一个具体类实现某一个接口，但又不必实现所有方法，通常处理方法是具体类要实现所有方法，有用的方法要有具体实现，没用的方法为空。

**一、结构**
![](https://upload-images.jianshu.io/upload_images/9449419-87bdd559c6699caa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
public interface AbstractService{
    void serviceOperation1();
    int serviceOperation2();
    String serviceOperation3();
}
```
```
public class ServiceAdapter implements AbstractService{
    public void serviceOperation1(){}
   public int serviceOperation2(){ return 0; }
   public void serviceOperation3(){ return null; }
}
```

**二、什么情况下使用本模式**

如果不准备实现一个接口所有实现方法时。



