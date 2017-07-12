---
layout: post
title: "多线程下的单例模式"
date: 2017-07-11
excerpt: "几种多线程下单利模式的实现形式，延迟加载、静态内部类、static块以及枚举实现"
tags: 
- 多线程
- Java
comments: true
---



### DCL双检查锁机制来实现多线程环境中的延迟加载单例设计模式
使用双检测机制来解决问题，既保证了不需要同步代码的异步执行性，又保证了单例的效果。
```java
class MyObject {
    private static MyObject object;
    private MyObject() {
    }
    public static MyObject getInstance() {
        if (object == null) {
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            synchronized (MyObject.class) {
                if (object == null)
                    object = new MyObject();
            }
        }
        return object;
    }
}
```

### 使用静态内部类实现单利模式
```java
class MyObject {
    private static class ObjectHandler {
        private static MyObject object = new MyObject();
    }

    private MyObject() {
    }

    public static MyObject getInstance() {
        return ObjectHandler.object;
    }
}
```

### 序列化与反序列化的单例模式实现
静态内部类可以达到线程安全问题，但如果遇到序列化对象时，使用默认的方式运行得到的结果还是多例的，解决办法是在反序列化中使用readResolve()方法：
`protected Object readResolve(){
return ObjectHandler.object;
}`

### 使用static代码块实现单利模式
静态代码块中的代码在使用类的时候就已经执行了，所以可以应用静态代码块的这个特性来实现单利模式。
```java
class MyObject{
    private static MyObject instance = null;
    private MyObject() {
    }
    static {
        instance = new MyObject();
    }
    public static MyObject getInstance() {
        return MyObject.instance;
    }
}
```

### 使用enum枚举数据类型实现单例模式
枚举enum和静态代码块的特性相似，在使用枚举类时，构造方法会被自动调用，也可以应用这个特性实现单例设计模式。
```java
enum MyObject {
    ConnectionFactory;
    private Connection connection;
    private MyObject() {
        try {
            System.out.println("invoke MyObject's constructor");
            String url = "";
            String username = "";
            String password = "";
            String driverName = "";
            Class.forName(driverName);
            connection = DriverManager.getConnection(url, username, password);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    public Connection getConnection() {
        return connection;
    }
}
```

### 完善使用enum枚举实现单例模式
为了遵循“职责单一原则”，对上案例修改。
```java
class MyObject {
    enum SingletonConnection {
        ConnectionFactory;
        private Connection connection;
        private SingletonConnection() {
            try {
                System.out.println("invoke SingletonConnection's constructor");
                String url = "";
                String username = "";
                String password = "";
                String driverName = "";
                Class.forName(driverName);
                connection = DriverManager.getConnection(url, username, password);
            } catch (ClassNotFoundException e) {
                e.printStackTrace();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        public Connection getConnection() {
            return connection;
        }
    }
    public static Connection getConnection() {
        return SingletonConnection.ConnectionFactory.getConnection();
    }
}
```
