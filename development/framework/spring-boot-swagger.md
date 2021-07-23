---
title: spring-boot-swagger
date: 2021-03-07 16:00:00
tags: 'Spring'
categories:
  - ['开发', '框架']
permalink: spring-boot-swagger
photo:
---

https://841809077.github.io/2020/06/03/Spring%20boot/swagger2-use-collection.html

# Spring boot Swagger2 配置使用实战

[](/2020/06/03/Spring boot/swagger2-use-collection.html#comments)

2020-06-03[springboot](/categories/springboot/)__点击__

> 今天来说一下 Spring boot 如何集成 Swagger 2, 虽然网上有很多这样的教程, 但觉得还是应该自己梳理一下, 这样对知识的掌握比较牢靠. 另外文章中也有我在开发中遇到的问题及解决方法, 统一记录下来.
>
> 真的比 postman 省心, 对于前后端联调, 测试, 用户来说都很便利. 可惜就是代码侵入性太强~ 暂时忍耐.

### [](#一, 集成 -Swagger-2 " 一, 集成 Swagger 2") 一, 集成 Swagger 2

1, 添加 pom.xml 文件依赖

<table> <tbody> <tr> <td class="gutter"> <pre> <span class="line">1</span> <br> <span class="line">2</span> <br> <span class="line">3</span> <br> <span class="line">4</span> <br> <span class="line">5</span> <br> <span class="line">6</span> <br> <span class="line">7</span> <br> <span class="line">8</span> <br> <span class="line">9</span> <br> <span class="line">10</span> <br> <span class="line">11</span> <br> <span class="line">12</span> <br> <span class="line">13</span> <br> <span class="line">14</span> <br> <span class="line">15</span> <br> <span class="line">16</span> <br> <span class="line">17</span> <br> <span class="line">18</span> <br> <span class="line">19</span> <br> <span class="line">20</span> <br> <span class="line">21</span> <br> </pre> </td> <td class="code"> <pre> <span class="line"> <span class="comment"><!-- swagger ui --> </span> </span> <br> <span class="line"> <span class="tag">< <span class="name"> dependency</span>> </span> </span> <br> <span class="line">    <span class="tag">< <span class="name"> groupId</span>> </span> io.springfox<span class="tag"></<span class="name"> groupId</span>> </span> </span> <br> <span class="line">    <span class="tag">< <span class="name"> artifactId</span>> </span> springfox-swagger2<span class="tag"></<span class="name"> artifactId</span>> </span> </span> <br> <span class="line">    <span class="tag">< <span class="name"> version</span>> </span>2.9.2<span class="tag"></<span class="name"> version</span>> </span> </span> <br> <span class="line"> <span class="tag"></<span class="name"> dependency</span>> </span> </span> <br> <span class="line"> <span class="tag">< <span class="name"> dependency</span>> </span> </span> <br> <span class="line">    <span class="tag">< <span class="name"> groupId</span>> </span> io.springfox<span class="tag"></<span class="name"> groupId</span>> </span> </span> <br> <span class="line">    <span class="tag">< <span class="name"> artifactId</span>> </span> springfox-swagger-ui<span class="tag"></<span class="name"> artifactId</span>> </span> </span> <br> <span class="line">    <span class="tag">< <span class="name"> version</span>> </span>2.9.2<span class="tag"></<span class="name"> version</span>> </span> </span> <br> <span class="line"> <span class="tag"></<span class="name"> dependency</span>> </span> </span> <br> <span class="line"> <span class="tag">< <span class="name"> dependency</span>> </span> </span> <br> <span class="line">    <span class="tag">< <span class="name"> groupId</span>> </span> io.swagger<span class="tag"></<span class="name"> groupId</span>> </span> </span> <br> <span class="line">    <span class="tag">< <span class="name"> artifactId</span>> </span> swagger-annotations<span class="tag"></<span class="name"> artifactId</span>> </span> </span> <br> <span class="line">    <span class="tag">< <span class="name"> version</span>> </span>1.5.22<span class="tag"></<span class="name"> version</span>> </span> </span> <br> <span class="line"> <span class="tag"></<span class="name"> dependency</span>> </span> </span> <br> <span class="line"> <span class="tag">< <span class="name"> dependency</span>> </span> </span> <br> <span class="line">    <span class="tag">< <span class="name"> groupId</span>> </span> io.swagger<span class="tag"></<span class="name"> groupId</span>> </span> </span> <br> <span class="line">    <span class="tag">< <span class="name"> artifactId</span>> </span> swagger-models<span class="tag"></<span class="name"> artifactId</span>> </span> </span> <br> <span class="line">    <span class="tag">< <span class="name"> version</span>> </span>1.5.22<span class="tag"></<span class="name"> version</span>> </span> </span> <br> <span class="line"> <span class="tag"></<span class="name"> dependency</span>> </span> </span> <br> </pre> </td> </tr> </tbody> </table>

曾经有一个 spring cloud 项目, 用到的 spring boot 版本是 1.x, 用了上述的 2.9.2 版本, 添加了 @EnableSwagger2 后, 项目启动失败. 将版本降低到 2.8.0 后, 项目运行正常. 测试发现, 后两个依赖也可以不用.

2, 添加 Swagger java 配置

<table> <tbody> <tr> <td class="gutter"> <pre> <span class="line">1</span> <br> <span class="line">2</span> <br> <span class="line">3</span> <br> <span class="line">4</span> <br> <span class="line">5</span> <br> <span class="line">6</span> <br> <span class="line">7</span> <br> <span class="line">8</span> <br> <span class="line">9</span> <br> <span class="line">10</span> <br> <span class="line">11</span> <br> <span class="line">12</span> <br> <span class="line">13</span> <br> <span class="line">14</span> <br> <span class="line">15</span> <br> <span class="line">16</span> <br> <span class="line">17</span> <br> <span class="line">18</span> <br> <span class="line">19</span> <br> <span class="line">20</span> <br> <span class="line">21</span> <br> <span class="line">22</span> <br> <span class="line">23</span> <br> <span class="line">24</span> <br> <span class="line">25</span> <br> <span class="line">26</span> <br> <span class="line">27</span> <br> <span class="line">28</span> <br> <span class="line">29</span> <br> <span class="line">30</span> <br> <span class="line">31</span> <br> <span class="line">32</span> <br> <span class="line">33</span> <br> <span class="line">34</span> <br> <span class="line">35</span> <br> <span class="line">36</span> <br> <span class="line">37</span> <br> <span class="line">38</span> <br> <span class="line">39</span> <br> <span class="line">40</span> <br> <span class="line">41</span> <br> <span class="line">42</span> <br> <span class="line">43</span> <br> <span class="line">44</span> <br> <span class="line">45</span> <br> <span class="line">46</span> <br> <span class="line">47</span> <br> <span class="line">48</span> <br> <span class="line">49</span> <br> <span class="line">50</span> <br> <span class="line">51</span> <br> <span class="line">52</span> <br> <span class="line">53</span> <br> <span class="line">54</span> <br> <span class="line">55</span> <br> <span class="line">56</span> <br> <span class="line">57</span> <br> <span class="line">58</span> <br> <span class="line">59</span> <br> <span class="line">60</span> <br> <span class="line">61</span> <br> <span class="line">62</span> <br> <span class="line">63</span> <br> <span class="line">64</span> <br> <span class="line">65</span> <br> <span class="line">66</span> <br> <span class="line">67</span> <br> <span class="line">68</span> <br> <span class="line">69</span> <br> <span class="line">70</span> <br> <span class="line">71</span> <br> <span class="line">72</span> <br> <span class="line">73</span> <br> <span class="line">74</span> <br> <span class="line">75</span> <br> <span class="line">76</span> <br> <span class="line">77</span> <br> <span class="line">78</span> <br> <span class="line">79</span> <br> <span class="line">80</span> <br> <span class="line">81</span> <br> <span class="line">82</span> <br> <span class="line">83</span> <br> <span class="line">84</span> <br> </pre> </td> <td class="code"> <pre> <span class="line"> <span class="keyword"> package</span> com.hollysys.hollicube.config; </span> <br> <span class="line"> </span> <br> <span class="line"> <span class="keyword"> import</span> org.springframework.context.annotation.Bean; </span> <br> <span class="line"> <span class="keyword"> import</span> org.springframework.context.annotation.Configuration; </span> <br> <span class="line"> <span class="keyword"> import</span> org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry; </span> <br> <span class="line"> <span class="keyword"> import</span> org.springframework.web.servlet.config.annotation.WebMvcConfigurationSupport; </span> <br> <span class="line"> <span class="keyword"> import</span> springfox.documentation.builders.ApiInfoBuilder; </span> <br> <span class="line"> <span class="keyword"> import</span> springfox.documentation.builders.PathSelectors; </span> <br> <span class="line"> <span class="keyword"> import</span> springfox.documentation.builders.RequestHandlerSelectors; </span> <br> <span class="line"> <span class="keyword"> import</span> springfox.documentation.service.ApiInfo; </span> <br> <span class="line"> <span class="keyword"> import</span> springfox.documentation.spi.DocumentationType; </span> <br> <span class="line"> <span class="keyword"> import</span> springfox.documentation.spring.web.plugins.Docket; </span> <br> <span class="line"> <span class="keyword"> import</span> springfox.documentation.swagger.web.*; </span> <br> <span class="line"> <span class="keyword"> import</span> springfox.documentation.swagger2.annotations.EnableSwagger2; </span> <br> <span class="line"> </span> <br> <span class="line"> <span class="comment">/**</span> </span> <br> <span class="line"> <span class="comment"> * <span class="doctag">@author</span> create17</span> </span> <br> <span class="line"> <span class="comment"> * <span class="doctag">@date</span> 2020/6/3</span> </span> <br> <span class="line"> <span class="comment"> */</span> </span> <br> <span class="line"> <span class="meta">@Configuration</span> </span> <br> <span class="line"> <span class="meta">@EnableSwagger</span>2</span> <br> <span class="line"> <span class="keyword"> public</span> <span class="class"> <span class="keyword"> class</span> <span class="title"> SwaggerConfig</span> <span class="keyword"> extends</span> <span class="title"> WebMvcConfigurationSupport</span> </span> {</span> <br> <span class="line">    <span class="meta">@Bean</span> </span> <br> <span class="line">    <span class="function"> <span class="keyword"> public</span> Docket <span class="title"> createRestApi</span> <span class="params"> () </span> </span> {</span> <br> <span class="line">        <span class="keyword"> return</span> <span class="keyword"> new</span> Docket(DocumentationType.SWAGGER_2) </span> <br> <span class="line">                .apiInfo(apiInfo()) </span> <br> <span class="line">                .select() </span> <br> <span class="line">                .apis(RequestHandlerSelectors.basePackage(<span class="string">"com.hollysys.hollicube.controller"</span>)) </span> <br> <span class="line">                .paths(PathSelectors.any()) </span> <br> <span class="line">                .build(); </span> <br> <span class="line">    } </span> <br> <span class="line"> </span> <br> <span class="line">    <span class="comment">/**</span> </span> <br> <span class="line"> <span class="comment">     * 隐藏 UI 上的 Models 模块 </span> </span> <br> <span class="line"> <span class="comment">     */</span> </span> <br> <span class="line">    <span class="meta">@Bean</span> </span> <br> <span class="line">    <span class="function"> <span class="keyword"> public</span> UiConfiguration <span class="title"> uiConfig</span> <span class="params"> () </span> </span> {</span> <br> <span class="line">        <span class="keyword"> return</span> UiConfigurationBuilder.builder() </span> <br> <span class="line">                .deepLinking(<span class="keyword"> true</span>) </span> <br> <span class="line">                .displayOperationId(<span class="keyword"> false</span>) </span> <br> <span class="line">                <span class="comment">// 隐藏 UI 上的 Models 模块 </span> </span> <br> <span class="line">                .defaultModelsExpandDepth(-<span class="number">1</span>) </span> <br> <span class="line">                .defaultModelExpandDepth(<span class="number">0</span>) </span> <br> <span class="line">                .defaultModelRendering(ModelRendering.EXAMPLE) </span> <br> <span class="line">                .displayRequestDuration(<span class="keyword"> false</span>) </span> <br> <span class="line">                .docExpansion(DocExpansion.NONE) </span> <br> <span class="line">                .filter(<span class="keyword"> false</span>) </span> <br> <span class="line">                .maxDisplayedTags(<span class="keyword"> null</span>) </span> <br> <span class="line">                .operationsSorter(OperationsSorter.ALPHA) </span> <br> <span class="line">                .showExtensions(<span class="keyword"> false</span>) </span> <br> <span class="line">                .tagsSorter(TagsSorter.ALPHA) </span> <br> <span class="line">                .validatorUrl(<span class="keyword"> null</span>) </span> <br> <span class="line">                .build(); </span> <br> <span class="line">    } </span> <br> <span class="line"> </span> <br> <span class="line">    <span class="function"> <span class="keyword"> private</span> ApiInfo <span class="title"> apiInfo</span> <span class="params"> () </span> </span> {</span> <br> <span class="line">        <span class="keyword"> return</span> <span class="keyword"> new</span> ApiInfoBuilder() </span> <br> <span class="line">                .title(<span class="string">" 访问 clientid 管理, 元数据管理, 日志管理 "</span>) </span> <br> <span class="line"> <span class="comment">//                .description("") </span> </span> <br> <span class="line"> <span class="comment">//                .version("1.0") </span> </span> <br> <span class="line"> <span class="comment">//                .contact(new Contact("","","")) </span> </span> <br> <span class="line"> <span class="comment">//                .license("") </span> </span> <br> <span class="line"> <span class="comment">//                .licenseUrl("") </span> </span> <br> <span class="line">                .build(); </span> <br> <span class="line">    } </span> <br> <span class="line"> </span> <br> <span class="line">    <span class="comment">/**</span> </span> <br> <span class="line"> <span class="comment">     * 防止 <span class="doctag">@EnableMvc</span> 把默认的静态资源路径覆盖了, 手动设置的方式 </span> </span> <br> <span class="line"> <span class="comment">     *</span> </span> <br> <span class="line"> <span class="comment">     * <span class="doctag">@param</span> registry</span> </span> <br> <span class="line"> <span class="comment">     */</span> </span> <br> <span class="line">    <span class="meta">@Override</span> </span> <br> <span class="line">    <span class="function"> <span class="keyword"> public</span> <span class="keyword"> void</span> <span class="title"> addResourceHandlers</span> <span class="params"> (ResourceHandlerRegistry registry) </span> </span> {</span> <br> <span class="line">        <span class="comment">// 解决静态资源无法访问 </span> </span> <br> <span class="line">        registry.addResourceHandler(<span class="string">"/**"</span>) </span> <br> <span class="line">                .addResourceLocations(<span class="string">"classpath:/static/"</span>); </span> <br> <span class="line">        <span class="comment">// 解决 swagger 无法访问 </span> </span> <br> <span class="line">        registry.addResourceHandler(<span class="string">"/swagger-ui.html"</span>) </span> <br> <span class="line">                .addResourceLocations(<span class="string">"classpath:/META-INF/resources/"</span>); </span> <br> <span class="line">        <span class="comment">// 解决 swagger 的 js 文件无法访问 </span> </span> <br> <span class="line">        registry.addResourceHandler(<span class="string">"/webjars/**"</span>) </span> <br> <span class="line">                .addResourceLocations(<span class="string">"classpath:/META-INF/resources/webjars/"</span>); </span> <br> <span class="line">    } </span> <br> <span class="line">} </span> <br> </pre> </td> </tr> </tbody> </table>

分析上述配置, 我们需要在 java 类上指明 @Configuration,@EnableSwagger2 注解.

另外还需要指定 controller 的包路径. 如果需要隐藏 Swagger ui 上的 Models 模块, 则需要上面的 uiConfig() 方法.

为了防止 @EnableMvc 把默认的静态资源路径覆盖, 还需要上面的 addResourceHandlers() 方法.

紧接着, 我们就可以启动项目了, Swagger 2 ui 地址为: [http://ip: port/swagger-ui.html](http://ip: port/swagger-ui.html) .

### [](#二, Swagger- 常用注解 " 二, Swagger 常用注解 ") 二, Swagger 常用注解

*   @Api(tags = "xxx 相关接口 ") : 修饰整个类, 描述 Controller 的作用.
*   @ApiOperation("xxxx") : 描述 api 接口方法.
*   @ApiModel(" 访问 clientid 表 ") : 当 @RequestParam 参数多的时候, 可以用对象来接收参数, 通常用在 @RequestBody 的 对象 内.**注意:@ApiModel 的 value 值需要保持唯一, 否则会出现覆盖的情况.**
*   @ApiModelProperty(value = " 主键 ", required = true, hidden = true) : 用对象接收参数时, 描述 Model 对象的一个字段, 也称为属性.
*   @ApiIgnore: 用于 controller 层, controller 层方法, controller 层方法参数上, 表示被 swagger ui 忽略.
*   @ApiParam(name = "", required = true, value = "clientid ID", hidden = true) : 可用于描述单个参数, 或者描述 @RequestBody 对象.
    *   name 为页面上的 Name
    *   value 为页面上的 Description
*   @ApiImplicitParams({```
     @ApiImplicitParam(name = "name", value = "clientid 名称 "),
            @ApiImplicitParam(name = "clientid", value = "clientid 编码 "),
            @ApiImplicitParam(name = "description", value = "clientid 描述信息 ")
    }) : 适用于 @RequestParam 参数少的时候, 参数多的时候可以用 @ApiModel 和 @ApiModelProperty .
    ```

参考博客: [https://www.cnblogs.com/fengli9998/p/7921601.html](https://www.cnblogs.com/fengli9998/p/7921601.html)

### [](#三, 个人小结 " 三, 个人小结 ") 三, 个人小结

#### [](#1, Swagger-ui- 页面上的 -body- 里面的 -Model "1, Swagger ui 页面上的 body 里面的 Model")1, Swagger ui 页面上的 body 里面的 Model

注解 @ApiModel 和 @ApiModelProperty 是作用在 javaBean 上的, 可以起解释说明, 是否必选, 是否隐藏的作用. 在 swagger-ui 页面上的体现形式如下图所示:

! [](https://cdn.jsdelivr.net/gh/841809077/blog-img/20200501/20200605105357.jpg)

#### [](#2, controller- 层 -swagger- 相关注解 "2, controller 层 swagger 相关注解 ")2, controller 层 swagger 相关注解

@Api,@ApiOperation,@ApiParam,@ApiIgnore,@ApiImplicitParams 都是作用在 controller 层的注解. 如下图所示:

! [](https://cdn.jsdelivr.net/gh/841809077/blog-img/20200501/20200605111122.jpg)

#### [](#3, PO, DTO, VO- 说明及使用 "3, PO, DTO, VO 说明及使用 ")3, PO, DTO, VO 说明及使用

*   PO(Persistant Object) 持久对象, 用于表示数据库中的一条记录映射成的 java 对象, 可以理解一个 PO 就是数据库中的一条记录;
*   DTO(Data Transfer Object) 数据传输对象, 前端调用时传输.
*   VO(Value Object) 表现对象, 用于表示一个与前端进行交互的 java 对象, 只包含前端需要展示的数据.

关于 java 中常见的对象类型简述 (DO, BO, DTO, VO, AO, PO) 可参考: [https://blog.csdn.net/uestcyms/article/details/80244407](https://blog.csdn.net/uestcyms/article/details/80244407) .

**当有多个 requestparam 参数的时候, 我们用 DTO 对象接收参数比较方便, 用 DTO 对象来精准无冗余地接收请求参数.**

可能这里有朋友会疑问, 为什么不用 PO 来接收请求参数呢?

因为 PO 中可能存在冗余字段, 如果用 PO 来接收参数的话, 冗余字段也会在 Swagger ui 页面上显示, 用户体验并不好, 所以我们用 DTO 来接收请求参数.

**同理, 为了避免返回给前端的数据存在冗余字段 (即不需要展示的字段), 我们可以使用 VO 来接收数据返回给前端进行交互.**

* * *

### [](#点关注, 不迷路 " 点关注, 不迷路 ") 点关注, 不迷路

好了各位, 以上就是这篇文章的全部内容了, 能看到这里的人呀, 都是**人才**.

**白嫖不好, 创作不易.** 各位的支持和认可, 就是我创作的最大动力, 我们下篇文章见!

如果本篇博客有任何错误, 请批评指教, 不胜感激 !

**再见**

* * *



### [点关注, 不迷路](#点关注, 不迷路 " 点关注, 不迷路 ")



好了各位, 以上就是这篇文章的全部内容了, 能看到这里的人呀, 都是**人才**.



**白嫖不好, 创作不易.**各位的支持和认可, 就是我创作的最大动力, 我们下篇文章见!



如果本篇博客有任何错误, 请批评指教, 不胜感激 !



! [微信公众号二维码](/img/qrcode.png " 扫一扫交个朋友 ")

[](javascript:; " 打赏 ")

↑
坚持原创技术分享, 您的支持将鼓励我继续创作!

! [微信打赏](/img/weChatMoney.png " 微信打赏 ")! [支付宝打赏](/img/alipayMoney.png " 支付宝打赏 ")

> 原文作者: create17
>
> 原文链接: [https://841809077.github.io/2020/06/03/Spring boot/swagger2-use-collection.html](https://841809077.github.io/2020/06/03/Spring boot/swagger2-use-collection.html)
>
> 版权声明: 转载请注明出处 (码字不易, 请保留作者署名及链接, 谢谢配合!)

分享到:

[Spring Boot 使用 @Valid 注解校验前端传递的参数](/2020/06/08/Spring boot/spring-boot-field-validation.html) [spring boot jpa 开发实战记录](/2020/06/02/Spring boot/spring-boot-jpa- 开发实战记录.html)
