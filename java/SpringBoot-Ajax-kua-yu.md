# SpringBoot Ajax 跨域请求

前后端分离，浏览器在访问时，经常会出现跨域访问。浏览器对于javascript的同源策略的限制。

HTTP请求时，请求本身会返回200，但是返回结果不会走success，并且会在浏览器console中提示：

已拦截跨源请求：同源策略禁止读取位于 https://www.baidu.com/ 的远程资源。（原因：CORS 头缺少 ‘Access-Control-Allow-Origin’）。


## 解决方案

spring boot中放开跨域请求很简单。


### 增加一个configuration

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;

/**
 * 跨域访问配置
 * @author Neal Zhao
 * @date 2019
 */
@Configuration
public class MyConfiguration {
    private CorsConfiguration buildConfig() {
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.addAllowedOrigin("*");
        corsConfiguration.addAllowedHeader("*");
        corsConfiguration.addAllowedMethod("*");
        return corsConfiguration;
    }
    @Bean
    public CorsFilter corsFilter() {
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", buildConfig());
        return new CorsFilter(source);
    }
}

```

增加此类以后，非同源http访问可以正常进行了，但是会不会有什么问题呢？

对于大部分网站依然需要使用cookie作为前后端传输数据的媒介，然而默认非同源请求是不携带cookie信息的。



### 服务端允许跨域携带cookie信息

修改上面的configuration类如下：


```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;


/**
 * 跨域访问配置
 * @author Neal Zhao
 * @date 2019
 */
@Configuration
public class MyConfiguration {
    private CorsConfiguration buildConfig() {
    CorsConfiguration corsConfiguration = new CorsConfiguration();
    corsConfiguration.addAllowedOrigin("*");
    corsConfiguration.addAllowedHeader("*");
    corsConfiguration.addAllowedMethod("*");
    corsConfiguration.addExposedHeader("Content-Type");
    corsConfiguration.addExposedHeader( "X-Requested-With");
    corsConfiguration.addExposedHeader("accept");
    corsConfiguration.addExposedHeader("Origin");
    corsConfiguration.addExposedHeader( "Access-Control-Request-Method");
    corsConfiguration.addExposedHeader("Access-Control-Request-Headers");
    corsConfiguration.setAllowCredentials(true);
    return corsConfiguration;
  }
  @Bean
  public CorsFilter corsFilter() {
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/**", buildConfig());
    return new CorsFilter(source);
  }
}

```

增加信息后，在前端依然需要调整AJAX请求，才能在非同源请求中携带cookie信息


### 前端调整

```javascript
$.ajax({
    url: 'http://url.com/longin.html',
    type: 'POST',
    async: true,
    xhrFields:{
        withCredentials: true
    },
    data: {
        username: userName,
        password: pwd
    },
    success: function(respon){
        console.log(respon);
        var res=eval(respon);
    },
    error: function(){
        alert('服务器发生错误！');
    }
});
```

此时，当前端向后端服务做Ajax跨域请求时，增加


```javascript
xhrFields:{
        withCredentials:true
},
```
就会带上cookie信息了，同理会带上token/sessionID等等内容。







