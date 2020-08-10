# Proxy Pattern
代理模式

## 概念
在不改变原始类代码的情况下，通过引入代理类来给原始类提供附加功能。

## 使用场景
+ 远程代理，如RPC框架，通过将网络通信、数据编码等细节隐藏起来，从而使开发者可以像本地使用函数一样；
+ 缓冲代理，为某操作提供临时的缓存存储空间，从而避免某些方法的重复执行，优化系统性能，如在Web项目中给Dao层添加缓冲代理类以提供缓存功能；
+ 虚拟代理，在真实对象创建成功之前虚拟代理扮演真实对象的替身，当真实对象创建完成后虚拟代理再将用户的请求转发给真实对象，常用在占用系统资源较多或者加载时间较长的对象中，如加载图片时可以先使用loading图，等待图片加载完成后再替换成真实图片；
+ 业务场景，如业务系统中非功能性需求：监控、统计、鉴权、限流、事务、幂等和日志等，通过代理类添加附加功能从而与业务功能解耦；
+ 其他场景：保护代理、防火墙代理、同步化代理、智能引用代理；

## 实现方式

### 静态代理
基于接口而非实现编程，代理类通过实现和原始类相同的接口或者是继承相同的父类，再将原始类进行替换。

#### Java Sample

```java
public interface IUserController {
    UserVo login(String phonne, String password);
    UserVo register(String phonne, String password);
}

public class UserController implements IUserController {
    @Override
    public login(String phonne, String password) {
        // 实现登录逻辑并返回UserVO
    }
    @Override
    public register(String phonne, String password) {
        // 实现注册逻辑并返回UserVO
    }
}

public class UserControllerProxy implements IUserController {
    private UserController userController
    
    public UserControllerProxy(UserController userController) {
        this.userController = userController;
    }
    
    @Override
    public login(String phonne, String password) {
        // 委托
        UserVo userVo = userController.login(phone, password)
        // 省略代理实现的逻辑，如监控、统计、鉴权等操作
        return userVo
    }
    @Override
    public register(String phonne, String password) {
        // 委托
        UserVo userVo = userController.register(phone, password)
        // 省略代理实现的逻辑，如监控、统计、鉴权等操作
        return userVo
    }
}

// 因为原始类和代理类实现相同的接口，因此在使用的过程中将UserController替换成UserControllerProxy即可，不需要做太多代码改动
IUserController userController = new UserControllerProxy(new UserController())
```

#### Golang Sample


## 类图

## 总结反思