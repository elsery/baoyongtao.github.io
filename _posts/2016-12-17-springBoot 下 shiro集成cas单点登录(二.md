---
layout: post
title: "springBoot 下 shiro集成cas单点登录(二"
date: 2016-12-17
description: "cas，shiro"
description: java 
--- 
>下面开始整合shiro  (springBoot项目)所以配置方式和spring不一样 但是原理一样 通用

maven添加相关依赖
```xml
		<!-- cas -->
		<dependency>
			<groupId>org.jasig.cas.client</groupId>
			<artifactId>cas-client-core</artifactId>
			<version>3.1.12</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/org.apache.shiro/shiro-cas -->
		<dependency>
			<groupId>org.apache.shiro</groupId>
			<artifactId>shiro-cas</artifactId>
			<version>1.2.4</version>
		</dependency>
```
重修shiro核心文件
ShiroCasConfiguration.java
```java
package com.example.springboot.commom.config.shiro;
import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.Map;
import javax.servlet.Filter;
import org.apache.shiro.cache.ehcache.EhCacheManager;
import org.apache.shiro.cas.CasFilter;
import org.apache.shiro.cas.CasSubjectFactory;
import org.apache.shiro.spring.LifecycleBeanPostProcessor;
import org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator;
import org.springframework.boot.context.embedded.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.filter.DelegatingFilterProxy;
import com.example.springboot.service.Userservice;
/**
 * Shiro集成Cas配置
 *
 * @author   Sharker
 * @create    2016年12月17日
 */
@Configuration
public class ShiroCasConfiguration {
    private static final Logger logger = LoggerFactory.getLogger(ShiroCasConfiguration.class);
    // CasServerUrlPrefix
    public static final String casServerUrlPrefix = "https://sso.myhost.org:8443/cas";
    // Cas登录页面地址
    public static final String casLoginUrl = casServerUrlPrefix + "/login";
    // Cas登出页面地址
    public static final String casLogoutUrl = casServerUrlPrefix + "/logout";
    // 当前工程对外提供的服务地址
    public static final String shiroServerUrlPrefix = "http://sso.myhost.org/spring-boot";
    // casFilter UrlPattern
    public static final String casFilterUrlPattern = "/shiro-cas";
    // 登录地址
    public static final String loginUrl = casLoginUrl + "?service=" + shiroServerUrlPrefix + casFilterUrlPattern;
    @Bean
    public EhCacheManager getEhCacheManager() {  
        EhCacheManager em = new EhCacheManager();  
        em.setCacheManagerConfigFile("classpath:ehcache-shiro.xml");  
        return em;  
    }  
    @Bean(name = "shiroCasRealm")
    public ShiroCasRealm shiroCasRealm(EhCacheManager cacheManager) {  
        ShiroCasRealm realm = new ShiroCasRealm(); 
        realm.setCacheManager(cacheManager);
        return realm;
    } 
    /**
     * 注册DelegatingFilterProxy（Shiro）
     *
     * @param dispatcherServlet
     * @return
     * @author   Sharker
     * @create  2016年12月13日
     */
    @Bean
    public FilterRegistrationBean filterRegistrationBean() {
        FilterRegistrationBean filterRegistration = new FilterRegistrationBean();
        filterRegistration.setFilter(new DelegatingFilterProxy("shiroFilter"));
        //  该值缺省为false,表示生命周期由SpringApplicationContext管理,设置为true则表示由ServletContainer管理  
        filterRegistration.addInitParameter("targetFilterLifecycle", "true");
        filterRegistration.setEnabled(true);
        filterRegistration.addUrlPatterns("/*");
        return filterRegistration;
    }
    @Bean(name = "lifecycleBeanPostProcessor")
    public LifecycleBeanPostProcessor getLifecycleBeanPostProcessor() {
        return new LifecycleBeanPostProcessor();
    }
    @Bean
    public DefaultAdvisorAutoProxyCreator getDefaultAdvisorAutoProxyCreator() {
        DefaultAdvisorAutoProxyCreator daap = new DefaultAdvisorAutoProxyCreator();
        daap.setProxyTargetClass(true);
        return daap;
    }
    @Bean(name = "securityManager")
    public DefaultWebSecurityManager getDefaultWebSecurityManager(ShiroCasRealm shiroCasRealm) {
        DefaultWebSecurityManager dwsm = new DefaultWebSecurityManager();
        dwsm.setRealm(shiroCasRealm);
//      <!-- 用户授权/认证信息Cache, 采用EhCache 缓存 --> 
        dwsm.setCacheManager(getEhCacheManager());
        // 指定 SubjectFactory
        dwsm.setSubjectFactory(new CasSubjectFactory());
        return dwsm;
    }
    @Bean
    public AuthorizationAttributeSourceAdvisor getAuthorizationAttributeSourceAdvisor(DefaultWebSecurityManager securityManager) {
        AuthorizationAttributeSourceAdvisor aasa = new AuthorizationAttributeSourceAdvisor();
        aasa.setSecurityManager(securityManager);
        return aasa;
    }
    /**
     * 加载shiroFilter权限控制规则（从数据库读取然后配置）
     *
     * @author   Sharker
     * @create  2016年12月14日
     */
    private void loadShiroFilterChain(ShiroFilterFactoryBean shiroFilterFactoryBean, Userservice userservice){
        /////////////////////// 下面这些规则配置最好配置到配置文件中 ///////////////////////
        Map<String, String> filterChainDefinitionMap = new LinkedHashMap<String, String>();
        filterChainDefinitionMap.put(casFilterUrlPattern, "casFilter");// shiro集成cas后，首先添加该规则
        // authc：该过滤器下的页面必须验证后才能访问，它是Shiro内置的一个拦截器org.apache.shiro.web.filter.authc.FormAuthenticationFilter
        // anon：它对应的过滤器里面是空的,什么都没做
        logger.info("##################从数据库读取权限规则，加载到shiroFilter中##################");
        filterChainDefinitionMap.put("/sys/login", "anon");
        filterChainDefinitionMap.put("/static/**", "anon");  
        filterChainDefinitionMap.put("/**", "user");  
        filterChainDefinitionMap.put("/logout", "logout");  
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
    }
    /**
     * CAS过滤器
     *
     * @return
     * @author   Sharker
     * @create  2016年12月17日
     */
    @Bean(name = "casFilter")
    public CasFilter getCasFilter() {
        CasFilter casFilter = new CasFilter();
        casFilter.setName("casFilter");
        casFilter.setEnabled(true);
        // 登录失败后跳转的URL，也就是 Shiro 执行 CasRealm 的 doGetAuthenticationInfo 方法向CasServer验证tiket
        casFilter.setFailureUrl(loginUrl);// 我们选择认证失败后再打开登录页面
        return casFilter;
    }
    /**
     * @return
     * @author   Sharker
     * @create  2016年12月14日
     */
    @Bean(name = "shiroFilter")
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(DefaultWebSecurityManager securityManager, CasFilter casFilter, Userservice userservice) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        // 必须设置 SecurityManager  
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        // 如果不设置默认会自动寻找Web工程根目录下的"/login.jsp"页面
        shiroFilterFactoryBean.setLoginUrl(loginUrl);
        // 登录成功后要跳转的连接
        shiroFilterFactoryBean.setSuccessUrl("/sys/index");
        shiroFilterFactoryBean.setUnauthorizedUrl("/403");
        // 添加casFilter到shiroFilter中
        Map<String, Filter> filters = new HashMap<>();
        filters.put("casFilter", casFilter);
        shiroFilterFactoryBean.setFilters(filters);
        loadShiroFilterChain(shiroFilterFactoryBean, userservice);
        return shiroFilterFactoryBean;
    }
}
```
ShiroCasRealm.java文件
```java
package com.example.springboot.commom.config.shiro;
import javax.annotation.PostConstruct;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.cas.CasRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import com.example.springboot.mapper.UserDao;
import com.example.springboot.vo.User;
public class ShiroCasRealm extends CasRealm{
    private static final Logger logger = LoggerFactory.getLogger(ShiroCasRealm.class);
    @Autowired
    private UserDao userDao; 
    @PostConstruct
    public void initProperty(){
//      setDefaultRoles("ROLE_USER");
        setCasServerUrlPrefix(ShiroCasConfiguration.casServerUrlPrefix);
        // 客户端回调地址
        setCasService(ShiroCasConfiguration.shiroServerUrlPrefix + ShiroCasConfiguration.casFilterUrlPattern);
    }
    /**
     * 权限认证，为当前登录的Subject授予角色和权限 
     * @see 经测试：本例中该方法的调用时机为需授权资源被访问时 
     * @see 经测试：并且每次访问需授权资源时都会执行该方法中的逻辑，这表明本例中默认并未启用AuthorizationCache 
     * @see 经测试：如果连续访问同一个URL（比如刷新），该方法不会被重复调用，Shiro有一个时间间隔（也就是cache时间，在ehcache-shiro.xml中配置），超过这个时间间隔再刷新页面，该方法会被执行
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        logger.info("##################执行Shiro权限认证##################");
        //获取当前登录输入的用户名，等价于(String) principalCollection.fromRealm(getName()).iterator().next();
        String loginName = (String)super.getAvailablePrincipal(principalCollection); 
        //到数据库查是否有此对象
        User user=userDao.getByUserName(loginName);// 实际项目中，这里可以根据实际情况做缓存，如果不做，Shiro自己也是有时间间隔机制，2分钟内不会重复执行该方法
        if(user!=null){
            //权限信息对象info,用来存放查出的用户的所有的角色（role）及权限（permission）
            //这里没有做校验 只有名字对就行
            SimpleAuthorizationInfo info=new SimpleAuthorizationInfo();
            return info;
        }
        // 返回null的话，就会导致任何用户访问被拦截的请求时，都会自动跳转到unauthorizedUrl指定的地址
        return null;
    }
}
```

核心就是这样
参考:
http://blog.csdn.net/catoop/article/details/50534006
http://blog.csdn.net/frinder/article/details/7969925
http://blog.csdn.net/small_love/article/details/6664831/
http://www.ibm.com/developerworks/cn/opensource/os-cn-cas/index.html

更多关于cas得文章 
http://blog.csdn.net/zhurhyme/article/category/2362091
