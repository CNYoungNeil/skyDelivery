# 项目笔记

## 前期准备



1. 后端基本代码。commit /push上传到GitHub repositories。
2. 前端直接使用nginx反向代理前端所有页面。
3. 通过“查询员工”来了解整个项目结构。包括：

   1. 在“EmployeeController.java”里对用户登陆成功后，JWT令牌的生成。并且返回VO对象。（接收对象为DTO，返回则封装为VO）
   2. 对用户密码进行MD5加密。（一句代码：对前端VO对象里的password进行MD5加密，然后重新赋值给password即可）(已经生成的用户就直接在数据库里修改，一般情况会在添加用户的时候，进行密码加密)
   3. 抛异常的全局类“ GlobalExceptionHandler.java”。在对应“MessageConstant.java”的常量类列举了所有报错信息。（如：登陆密码错误报错，返回的错误常量`PASSWORD_ERROR`，则会直接显示信息`密码错误`给前端）
4. 前端接口工具，YApi和Swagger。

   1. YApi直接导入JSON格式的接口即可。
   2. Swagger更广泛用户后端开发接口。在使用的时候，项目用到了knife4j的依赖，是基于swagger开发的工具包，方便更快捷操作swagger。






## Swagger的使用方法

现阶段使用Swagger用于测试**已经开发的后端接口**，想要打开Swagger测试接口平台步骤为：

1. 导入依赖：

   ```xml
           <dependency>
               <groupId>com.github.xiaoymin</groupId>
               <artifactId>knife4j-spring-boot-starter</artifactId>
               <version>3.0.2</version>
           </dependency>
   ```

2. 在配置类WebMvcConfiguration中，配置相关内容。

   ```java
     /**
        * 通过knife4j生成接口文档
        * @return
        */
       @Bean
       public Docket docket() {
           log.info("开始准备接口文档...");
           ApiInfo apiInfo = new ApiInfoBuilder()
                   .title("苍穹外卖项目接口文档")
                   .version("2.0")
                   .description("苍穹外卖项目接口文档")
                   .build();
           Docket docket = new Docket(DocumentationType.SWAGGER_2)
                   .apiInfo(apiInfo)
                   .select()
                   .apis(RequestHandlerSelectors.basePackage("com.sky.controller"))    //指定要扫描的包
                   .paths(PathSelectors.any())
                   .build();
           return docket;
       }
   
       /**
        * 设置静态资源映射
        * @param registry
        */
       protected void addResourceHandlers(ResourceHandlerRegistry registry) {
           log.info("开始设置静态资源映射...");
           registry.addResourceHandler("/doc.html").addResourceLocations("classpath:/META-INF/resources/");
           registry.addResourceHandler("/webjars/**").addResourceLocations("classpath:/META-INF/resources/webjars/");
       }
   ```

   *仅作参考，不需要自己写。*

   自己修改里面内容，类似项目名，扫描的文件路径包等等。

3. 启动项目。

4. 打开网页：http://localhost:8080/doc.html则可以访问接口。这个路径，是在第2步的静态资源映射实现的。



# 项目功能

## 新增用户



### 完善部分

1. 用户名重复，需要捕获错误信息，并返回对应信息到Result返回值内。
2. 获取创建人id和更新人id。需要使用**线程隔离ThreadLocal**的方式获取。

**获取用户id步骤：**

1. 首先，通过ThreadLocal方法可以验证的是，同一个请求，所使用的是同一个线程。
   1. 然后就可以使用Java线程库里的方法`BaseContext`，来存或取一个id。
   2. 本质上`BaseContext`就是把`ThreadLocal`的存、取、删除，这3个功能拿出来重新封装了一遍。
2. 存id：在校验jwt令牌后，get到jwt里面的`EMP_ID`的字段。此前已经在封装jwt的时候，把`EMP_ID`放进去了
3. 取id：在新增员工的service层方法里（也就是save方法里），拷贝DTO属性的时候，把`EMP_ID`直接获取，加到set的`创建用户id`和`更新用户id`中



## 分页查询





# 报错

## 响应码401

**情景**

一般在sagger测试发送请求后，没有响应，但是响应码为401。

**说明**

表示该请求被拦截器jwt令牌拦截，一般为没有token或者token过期导致的。

**解决方案**

1. 可以使用`admin`发送一次登录请求，然后获取其响应的token。再将token配置到swagger的“全局参数设置”里面。
2. 设置token响应码过期时间，可以调整为更长的时间



# 操作计较

## Debug断点处的值

有时候断点里面的**操作结果不会显示**，只会显示一些**对象或者属性的值**

比如这里的`BaseContext.getCurrentId()`计算的结果

<img src="C:\Users\Neil\AppData\Roaming\Typora\typora-user-images\image-20241112235720769.png" alt="image-20241112235720769" style="zoom:67%;" />

**查看方法**

选中之后，点击`Evaluate Expression`就可以查看计算结果

![image-20241112235822163](C:\Users\Neil\AppData\Roaming\Typora\typora-user-images\image-20241112235822163.png)
