###### Mysql

1、MySQL 连接出现 Authentication plugin 'caching_sha2_password' cannot be loaded

原因：mysql8 之前的版本中加密规则是mysql_native_password,而在mysql8之后,加密规则是caching_sha2_password

解决方法：https://www.cnblogs.com/zhurong/p/9898675.html



###### 后端

1、自定义静态资源映射出错

原因：项目中放到了classpath即resources下的backend和front文件夹中，SpringBoot默认映射是static和public，因此需要手动改变映射

解决方法：config包需要与主启动类在一层级（因为默认扫描的就是主类的层级），然后写自定义配置类并继承重写方法，放行自定义的静态资源

```java
@Slf4j
@Configuration
public class WebMvcConfig extends WebMvcConfigurationSupport {
    /**
     * 设置静态资源映射
     * @param registry
     */
    @Override
    protected void addResourceHandlers(ResourceHandlerRegistry registry) {
        log.info("自定义静态资源映射成功");
        registry.addResourceHandler("/backend/**").addResourceLocations("classpath:/backend/");//放行的静态资源
        registry.addResourceHandler("/front/**").addResourceLocations("classpath:/front/");
    }
}
```

2、@AutoWired和@Resource的区别及使用

原因：标准@AutoWired时，IDEA会警告不推荐使用

解决方法：https://blog.csdn.net/youanyyou/article/details/126970723

3、登录时反复跳转到登录页面

原因：用户信息没有正确的放到session，在过滤器过滤请求时，发现session中没有用户id而跳转回登录页，是因为最开始往session中传id时，用的请求参数（`@RequestBody Employee employee`），但请求参数中是没有id的，因此id为null

解决方法：因此需要将数据库查出来employee对应的id放入session中



###### 前端：

1、如果未登录则返回未登录结果，通过输出流方式向客户端页面响应数据

后端：要在response响应中，将信息返回至客户端

```java
response.getWriter().write(JSON.toJSONString(R.error("NOTLOGIN")));
```

前端：前端自己的拦截器在收到服务器发来的响应时，如果msg中是"NOTLOGIN"，则将浏览器中的用户信息清除并跳转到登录页

```js
// 响应拦截器
service.interceptors.response.use(res => {
    if (res.data.code === 0 && res.data.msg === 'NOTLOGIN') {// 返回登录页面
        console.log('---/backend/page/login/login.html---')
        localStorage.removeItem('userInfo')
        window.top.location.href = '/backend/page/login/login.html'
    } else {
        return res.data
    }
},
```

2、修改用户账号状态不生效

原因：用雪花算法生成的id是19位的long型，但是前端js在处理时只能处理到16位，因此在分页查询出员工信息时，该员工的id只有16位，丢失了精度，因此再使用此行的员工id时，也只有16位，不足的0补全，造成id的不一致

解决方法：服务端给页面响应Json数据时，将Long型数据统一转为string字符串进行处理，在WebMVC中，扩展MVC框架的消息转换器，使用Jackson进行序列化和反序列化

```java
/**
* 扩展MVC框架的消息转换器
* @param converters
*/
@Override
protected void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
    //创建消息转换器对象
    MappingJackson2HttpMessageConverter converter = new MappingJackson2HttpMessageConverter();
    //设置对象转换器，底层使用Jackson将Java对象转换成Json
    converter.setObjectMapper(new JacksonObjectMapper());
    //将上面的消息转换器对象追加到mvc框架的转换器集合中
    converters.add(0, converter);
}
```



