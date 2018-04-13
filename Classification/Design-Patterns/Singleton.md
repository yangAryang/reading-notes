### 1.懒汉模式 线程不安全

```
pulbic class Singleton {
    //定义私有的实例
    private static Singleton instance;
    //提供私有的构造器
    private Singleton() {}
    //提供公有的静态获取实例的方法
    puulic static Singleton getInstance() {
        if(null == this.insatance) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
缺点：线程不安全。lazy初始化。不建议使用
### 2.懒汉模式 线程安全
```
pulbic class Singleton {
    //定义私有的实例
    private static Singleton instance;
    //提供私有的构造器
    private Singleton() {}
    //提供实例的方法加上锁
    puulic static synchronized Singleton getInstance() {
        if(null == this.insatance) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
之所以线程安全是因为加了同步锁。
缺点:效率低下。不建议使用。

### 3.饿汉模式 线程安全

```
pulbic class Singleton {
    //定义私有的实例
    private static Singleton instance = new Singleton();
    //提供私有的构造器
    private Singleton() {}
    //提供公有的静态获取实例的方法
    puulic static Singleton getInstance() {
        //直接return
        return instance;
    }
}
```
在类加载阶段就已经完成了实例的初始化
缺点:类加载阶段完成初始化,浪费内存。
优点:没有锁,效率提高。建议使用。

### 4.double check locking(懒)

```
pulbic class Singleton {
    //定义私有的实例
    private volatile static Singleton instance;
    //提供私有的构造器
    private Singleton() {}
    //提供公有的静态获取实例的方法
    puulic static Singleton getInstance() {
    //第一次检测
       if(null == instance) {
       //锁住类对象
           synchronized(Singleton.class) {
           //再次进行判断
               if(null == instance) {
               //赋值
                   instance = new Singleton();
               }
           }
       }
       return instance;
    }
}
```
优点:双锁机制,安全性高。

### 5.静态内部类(饿)

```
public class Singleton {
//静态内部类
    private class InnerClass{
        //静态final实例  饿汉式
        private static final Singleton INSTANCE = new Singleton();
        private Singleton() {}
        //内部类.实例
        public static final Singleton getInstance() {
            return InnerClass.INSTANCE;
        }
    }
}
```
特点：内部类没有主动使用的时候不会类加载,使用的时候才会去加载
优点:方便.不浪费内存。

### 6.枚举

```
public enum Singleton {
    INSTANCE;
    public void whatEverMethod(){}
}
```
最佳实践方法,简洁,自动支持序列化,防止多实例化。

**总结：1,2不建议使用,而建议使用3. 明确要求懒加载使用5,设计序列化对象使用6,特殊需求4.**
