# shiro 简学

## 目的

基于毕业设计需要用到身份验证及权限控制这一功能，自己写的总是不太满意，也太繁琐，所以网上找到了这个框架，并加以学习，准备投入使用到毕设中。

## 身份验证

 ### 入门案例

```java 
package authentication;

import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.IncorrectCredentialsException;
import org.apache.shiro.authc.LockedAccountException;
import org.apache.shiro.authc.UnknownAccountException;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.mgt.DefaultSecurityManager;
import org.apache.shiro.realm.SimpleAccountRealm;
import org.apache.shiro.subject.Subject;
import org.junit.Before;
import org.junit.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class LoginTest {

    Logger logger = LoggerFactory.getLogger(LoginTest.class);

    // 初始化Realm
    SimpleAccountRealm simpleAccountRealm = new SimpleAccountRealm();

    @Before
    public void addUser(){
        // 添加用户，该方法为重载方法， 可以为用户添加角色
        simpleAccountRealm.addAccount("zhang", "123");
    }
    
    @Test
    public void testLogin(){
		// 获取token
        UsernamePasswordToken usernamePasswordToken = new UsernamePasswordToken("zhang","1223");
        
        // 即安全管理器；即所有与安全有关的操作都会与 SecurityManager 交互；且它管理着所有 Subject
        DefaultSecurityManager defaultSecurityManager = new DefaultSecurityManager();
        
        // 注册reaml
        defaultSecurityManager.setRealm(simpleAccountRealm);
        
        // 绑定 SecurityManager
        SecurityUtils.setSecurityManager(defaultSecurityManager);
        // 主体，代表了当前 “用户”，这个用户不一定是一个具体的人，与当前应用交互的任何东西都是 Subject
        Subject subject = SecurityUtils.getSubject();
        try{
            // 登录
            subject.login(usernamePasswordToken);
        }catch (UnknownAccountException unknownAccount){
            logger.error("账号不存在...", unknownAccount);
        }catch (LockedAccountException lockedAccount){
            logger.error("账号已被锁定...", lockedAccount);
        }catch (IncorrectCredentialsException incorrectCredentials){
            logger.error("密码错误...", incorrectCredentials);
        }
        System.out.println(subject.isAuthenticated());
        // 登出
         subject.logout();
    }
}
```

### 流程

![img](shiro 简学.assets/4.png)

1. 首先调用 Subject.login(token) 进行登录，其会自动委托给 Security Manager，调用之前必须通过 SecurityUtils.setSecurityManager() 设置；
2. SecurityManager 负责真正的身份验证逻辑；它会委托给 Authenticator 进行身份验证；
3. Authenticator 才是真正的身份验证者，Shiro API 中核心的身份认证入口点，此处可以自定义插入自己的实现；
4. Authenticator 可能会委托给相应的 AuthenticationStrategy 进行多 Realm 身份验证，默认 ModularRealmAuthenticator 会调用 AuthenticationStrategy 进行多 Realm 身份验证；
5. Authenticator 会把相应的 token 传入 Realm，从 Realm 获取身份验证信息，如果没有返回 / 抛出异常表示身份验证失败了。此处可以配置多个 Realm，将按照相应的顺序及策略进行访问。