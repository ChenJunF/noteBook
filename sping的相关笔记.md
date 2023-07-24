# sping的相关笔记

## url相关类：RequestMappingHandlerMapping.class

RequestMappingHandlerMapping也是在DispatcherServlet的初始化过程中自动加载的，默认会自动加载所有实现HandlerMapping接口的bean，且我们可以通过serOrder来设置优先级，系统默认会加载RequestMappingHandlerMapping、BeanNameUrlHandlerMapping、SimpleUrlHandlerMapping 并且按照顺序使用
这个类可以获取到所有的接口的url和方法。



```java
public String[] urlRout(){
        applicationContext.getBean(RequestMappingHandlerMapping.class);
        RequestMappingHandlerMapping mapping = applicationContext.getBean(RequestMappingHandlerMapping.class);
        // 获取url与类和方法的对应信息
        Map<RequestMappingInfo, HandlerMethod> map = mapping.getHandlerMethods();
        List<String> router = new ArrayList<>();
        for (Map.Entry<RequestMappingInfo, HandlerMethod> m : map.entrySet()) {
            RequestMappingInfo info = m.getKey();
            HandlerMethod method = m.getValue();
            Ignore methodAnnotation = method.getMethodAnnotation(Ignore.class);
            if (methodAnnotation != null) {
                PatternsRequestCondition p = info.getPatternsCondition();
                for (String url : p.getPatterns()) {
                    router.add(url);
                }
            }
        }
        String[] array = router.toArray(new String[router.size()]);
        return array;
    }
```



可以通过这种方式拿到所有controller的url，并且拿到该url的方法，根据方法判断是否存在@Ignore注解，要是有该注解，security并对该地址放行（不用必须登录）。

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter implements ApplicationContextAware {

    @Autowired
    private JwtAuthenticationTokenFilter jwtAuthenticationTokenFilter;



    @Autowired
    private WebApplicationContext applicationContext;


    //创建BCryptPasswordEncoder注入容器
    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }


    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManager();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                //关闭csrf
                .csrf().disable()
                //不通过Session获取SecurityContext
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .authorizeRequests()
                // 对于登录接口 允许匿名访问
                .antMatchers("/user/login").anonymous()
                .antMatchers(urlRout()).permitAll()
                // 除上面外的所有请求全部需要鉴权认证
                .anyRequest().authenticated();
        //把token校验过滤器添加到过滤器链中
        http.addFilterBefore(jwtAuthenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);
    }

  //想在上面的过滤链上加注入路径



    public String[] urlRout(){
        applicationContext.getBean(RequestMappingHandlerMapping.class);
        RequestMappingHandlerMapping mapping = applicationContext.getBean(RequestMappingHandlerMapping.class);
        // 获取url与类和方法的对应信息
        Map<RequestMappingInfo, HandlerMethod> map = mapping.getHandlerMethods();
        List<String> router = new ArrayList<>();
        for (Map.Entry<RequestMappingInfo, HandlerMethod> m : map.entrySet()) {
            RequestMappingInfo info = m.getKey();
            HandlerMethod method = m.getValue();
            Ignore methodAnnotation = method.getMethodAnnotation(Ignore.class);
            if (methodAnnotation != null) {
                PatternsRequestCondition p = info.getPatternsCondition();
                for (String url : p.getPatterns()) {
                    router.add(url);
                }
            }
        }
        String[] array = router.toArray(new String[router.size()]);
        return array;
    }
}

```



在spring中要时getBean()的内容是一个注解时，那么该返回值一定是一个注解类，没有更多的信息。而我们想要拿到更多的信息需要往实现类上靠，spring拿到这个注解，是怎么解析的这个类上面，或者是在存储信息的类上靠。



<img src="file:///C:/Users/admin/Pictures/Typedown/d8841272-64dc-43e3-84da-2f3835cd1a67.png" title="" alt="d8841272-64dc-43e3-84da-2f3835cd1a67" data-align="inline">

由该图可以看出RequestMappingHandlerMapping是最后的实现类，其信息是最多的。





docker run -d -p 3306:3306 --name name1 -v /mydata:/var/lib/mysql  -e MYSQY_ROOT_PASSWORD=abc123 镜像名:端口



vim dockerFile

FROM image

VOLUME

ADD 

EXPOSE 

CMD 




