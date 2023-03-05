## SpringCloud Alibaba微服务组件笔记







## 配置中心Nacos

### 如何使用

- 使用配置中心，可以动态刷新配置，在不修改本地Jar配置的情况下直接在配置中心修改配置。

  - 如下@Value注解+@RefreshScope注解完成动态刷新配置

  ```properties
  #此为配置中心中gulimall-coupon.properties的配置。
  #服务名.properties
  coupon.user.name=limaolin
  coupon.user.age=20
  ```

  ```java
  //此为Controller中如何读取配置的Demo。 Controller类上加上@RefreshScope，这个注解的意思是每次读取都会动态加载配置，即每次读取配置项中的内容是都会去nacos配置中心读取最新配置
  @RefreshScope
  @RestController
  @RequestMapping("coupon/coupon")
  public class CouponController {
      @Autowired
      private CouponService couponService;
  
  //使用@Value(${配置项})读取配置
      @Value("${coupon.user.name}")
      private String name;
      @Value("${coupon.user.age}")
      private Integer age;
      }
  ```

  

- 使用配置中心，配置会先从bootsrap中读取配置中心的一系列配置，再从配置中心读取相关配置，如果配置中心没有，读取本地配置。

  - 如下所示（boostrap.properties）：

- ```properties
  spring.application.name=gulimall-coupon #指定当前服务的名称
  spring.cloud.nacos.config.server-addr=127.0.0.1:8848 #指定配置中心的地址
  spring.cloud.nacos.config.namespace=1986f4f3-69e0-43bb-859c-abe427b19f3a #指定配置中心的命名空间ID
  spring.cloud.nacos.config.group=prod #指定配置中心的分组
  ```

  

### 命名空间：

- 命名空间主要用来做配置隔离。
- 可以根据环境进行隔离，如创建dev、prop、test等不同环境的命名空间。
- 可以根据微服务进行隔离，再使用分组进行环境隔离

### 配置集

- SpringBoot中的每一个配置都可以看作一个配置集。

- Nacos可以加载同时加载多个配置集，只需在Bootsrap中指定即可。

- 如果单个配置集过于冗长，可以将其拆分，再使用Nacos加载多配置集。

  - 可以将数据源拆分为一个配置集，将框架相关的拆分为一个配置集，将加密相关的拆分为一个配置集。

  - 原配置application.yml，如下：

  - ```yml
    spring:
      datasource:
        username: root
        password: lml2108339.
        url: jdbc:mysql://175.178.101.182:3306/mall
        driver-class-name: com.mysql.jdbc.Driver
      cloud:
        nacos:
          discovery:
            server-addr: 127.0.0.1:8848
      application:
        name: gulimall-coupon
    mybatis-plus:
      mapper-locations: classpath:/mapper/**/*.xml
      global-config:
        db-config:
          id-type: auto
    server:
      port: 7000
    
    ```

  - 现配置拆分为三个配置集，分别为datasource.yml,framework.yml,other.yml，如下：

  ```properties
  spring.application.name=gulimall-coupon #指定当前服务的名称
  spring.cloud.nacos.config.server-addr=127.0.0.1:8848 #指定配置中心的地址
  spring.cloud.nacos.config.namespace=1986f4f3-69e0-43bb-859c-abe427b19f3a #指定配置中心的命名空间ID
  spring.cloud.nacos.config.group=prod #指定配置中心的分组
  #指定配置集0
  spring.cloud.nacos.config.ext-config[0].data-id=datasource.yml #数据集命名空间ID
  spring.cloud.nacos.config.ext-config[0].group=dev #数据集所属分组
  spring.cloud.nacos.config.ext-config[0].refresh=true #数据集是否开启动态刷新，默认为false
  #指定配置集1
  spring.cloud.nacos.config.ext-config[1].data-id=framework.yml #数据集命名空间ID
  spring.cloud.nacos.config.ext-config[1].group=dev #数据集所属分组
  spring.cloud.nacos.config.ext-config[1].refresh=true #数据集是否开启动态刷新，默认为false
  #指定配置集2
  spring.cloud.nacos.config.ext-config[2].data-id=other.yml #数据集命名空间ID
  spring.cloud.nacos.config.ext-config[2].group=dev #数据集所属分组
  spring.cloud.nacos.config.ext-config[2].refresh=true #数据集是否开启动态刷新，默认为false
  ```

  

## 跨域

### 原因

- 跨域只发生在浏览器端，本质上是因为浏览器的同源策略而产生了跨域。
- 一个HTTP请求由ip地址，端口号，协议等组成，在请求时如果出现了任一一部分不相同，都会导致跨域。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fcac2c52aba34069a488e457cc1cfcf6~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

### 预检

- 在发出跨域请求时，如果本次请求是非简单请求，那么浏览器会先发送一个options查询请求，成为预检，用于确认目标资源是否支持跨域，收到200以后才会真正发送请求。

### 如何解决

- 将资源部署在一台Nginx上，通过访问nginx进行反向代理来避免非同源请求。
- 服务端配置当前请求运行跨域，主要是添加响应头来解决，一般在网关通过过滤器解决。

```java
@Configuration
public class GulimallCorsConfiguration {
    @Bean
    public CorsWebFilter corsWebFilter(){
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        //1、配置跨域
        corsConfiguration.addAllowedHeader("*");
        corsConfiguration.addAllowedMethod("*");
        corsConfiguration.addAllowedOrigin("*");
        corsConfiguration.setAllowCredentials(true);
        source.registerCorsConfiguration("/**",corsConfiguration);
        return new CorsWebFilter(source);
    }
}
```

