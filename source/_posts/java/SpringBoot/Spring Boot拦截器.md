


拦截器可以根据 URL 对请求进行拦截，主要应用于登陆校验、权限验证、乱码解决、性能监控和异常处理等功能上。

在 Spring Boot 项目中，使用拦截器功能通常需要以下 3 步：
1. 定义拦截器；
2. 注册拦截器；
3. 指定拦截规则（如果是拦截所有，静态资源也会被拦截）。

# 定义拦截器
在 Spring Boot 中定义拦截器只需要创建一个拦截器类，并实现`HandlerInterceptor`接口即可。

`HandlerInterceptor`接口中定义以下 3 个方法。

| 返回值类型 | 方法声明 | 描述 |
| :--: | :--: | :--: |
| boolean | preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) | 该方法在控制器处理请求方法前执行，其返回值表示是否中断后续操作，返回 true 表示继续向下执行，返回 false 表示中断后续操作。 |
| void | postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) | 该方法在控制器处理请求方法调用之后、解析视图之前执行，可以通过此方法对请求域中的模型和视图做进一步修改。 |
| void | afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)| 该方法在视图渲染结束后执行，可以通过此方法实现资源清理、记录日志信息等工作。 |

在`net.biancheng.www.interceptor`中创建一个名为`LoginInterceptor`的拦截器类，对登陆进行拦截。
```java
package net.biancheng.www.interceptor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
@Slf4j
public class LoginInterceptor implements HandlerInterceptor {
  /**
    * 目标方法执行前
    *
    * @param request
    * @param response
    * @param handler
    * @return
    * @throws Exception
    */
  @Override
  public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    Object loginUser = request.getSession().getAttribute("loginUser");
    if (loginUser == null) {
      //未登录，返回登陆页
      request.setAttribute("msg", "您没有权限进行此操作，请先登陆！");
      request.getRequestDispatcher("/index.html").forward(request, response);
      return false;
    } else {
      //放行
      return true;
    }
  }
  /**
    * 目标方法执行后
    *
    * @param request
    * @param response
    * @param handler
    * @param modelAndView
    * @throws Exception
    */
  @Override
  public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
    log.info("postHandle执行{}", modelAndView);
  }
  /**
    * 页面渲染后
    *
    * @param request
    * @param response
    * @param handler
    * @param ex
    * @throws Exception
    */
  @Override
  public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
    log.info("afterCompletion执行异常{}", ex);
  }
}
```
# 注册拦截器
创建一个实现了`WebMvcConfigurer`接口的配置类（使用了`@Configuration`注解的类），重写`addInterceptors()`方法，并在该方法中调用`registry.addInterceptor()`方法将自定义的拦截器注册到容器中。

在配置类`MyMvcConfig`中，添加以下方法注册拦截器。
```java
@Configuration
public class MyMvcConfig implements WebMvcConfigurer {
  ......
  @Override
  public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(new LoginInterceptor());
  }
}
```
# 指定拦截规则
在使用`registry.addInterceptor()`方法将拦截器注册到容器中后，我们便可以继续指定拦截器的拦截规则了。
```java
@Slf4j
@Configuration
public class MyConfig implements WebMvcConfigurer {
  ......
  @Override
  public void addInterceptors(InterceptorRegistry registry) {
    log.info("注册拦截器");
    registry.addInterceptor(new LoginInterceptor()).addPathPatterns("/**") //拦截所有请求，包括静态资源文件
            .excludePathPatterns("/", "/login", "/index.html", "/user/login", "/css/**", "/images/**", "/js/**", "/fonts/**"); //放行登录页，登陆操作，静态资源
  }
}
```
在指定拦截器拦截规则时，调用了两个方法，这两个方法的说明如下：
* `addPathPatterns`：该方法用于指定拦截路径，例如拦截路径为`/**`，表示拦截所有请求，包括对静态资源的请求。
* `excludePathPatterns`：该方法用于排除拦截路径，即指定不需要被拦截器拦截的请求。
