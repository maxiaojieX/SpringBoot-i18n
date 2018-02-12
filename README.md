
# SpringBoot国际化使用

* ### [简介](#description) 
* ### [快速使用 Demo](#quick) 
* ### [加入配置](#config)
* ### [雷区警告](#worn) 


<hr>
<br>
<h3 id="description">简介</h2>
&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp SpringBoot默认支持国际化，采用默认区域解析器AcceptHeaderLocalResolver检验HTTP请求头部accept-languager来解析区域。如果我们希望从accept-language中获取区域，那么我们可以使用他的默认配置，只需要添加国际化资源文件即可，SpringBoot默认的国际化资源文件名以 messages 开头。
<br><br>
<h3 id="quick">快速使用</h2>
&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp 1. 首先在resources下创建文件夹 i18n,然后新建以下国际化资源文件，命名为: <br>&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp &nbsp
messages.properties <br>&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp &nbsp
messages_en_US.properties <br>&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp &nbsp
messages_zh_CN.properties <br>&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp &nbsp 三个文件下对应的内容分别是: <br>

```
hello=中国(默认)
```
```
hello=China
```
```
hello=中国
```
<br>
&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp 2. 在主配置文件中引用这些文件<br>&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp &nbsp
spring.messages.basename=i18n/messages
<br><br>&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp 3. OK,到此已经完成了，放心大胆的使用，没有问题。
<br><br>&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp 4. 如果想要在代码中获取资源的话，我这里给出了一个Util。供参考:

```java
@Component
public class I18nMessageUtil {

    @Resource
    private MessageSource messageSource;
    public String getMessage(String key) {
        Locale locale = LocaleContextHolder.getLocale();
        return messageSource.getMessage(key,null,locale);
    }
}
```
<br> 
&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp传入key就能获取对应地区的资源文件Value
<br><br>
<h3 id="config">加入配置</h2>
&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp 如果我们还需要一些其他的配置，例如设置默认区域和使用url参数来改变区域，在csdn或者博客源中一些例子也是这样写的，但是对新人使用可能会造成误区，以为必须要存在配置类，我刚开始就以为是这样。所以有必要再次说一下，如果我们期望用accept-language来识别区域，那么我们只需要按上面的步骤就可以了，不需要添加任何配置类。
<br>&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp 下面贴出配置类:

```java
@Configuration
@EnableAutoConfiguration
@ComponentScan
public class I18nConfig extends WebMvcConfigurerAdapter{

    @Bean
    public SessionLocaleResolver localeResolver() {
        
        SessionLocaleResolver slr = new SessionLocaleResolver();
        // 设置默认语言
        slr.setDefaultLocale(Locale.CHINESE);
        return slr;
    }

    @Bean
    public LocaleChangeInterceptor localeChangeInterceptor() {
        LocaleChangeInterceptor lci = new LocaleChangeInterceptor();
        // 参数名
        lci.setParamName("lang");
        return lci;
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(localeChangeInterceptor());
    }
}
```
<br>
&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp这样就设置了默认区域语言为中文，并且可以在url中拼接 "?lang=..." 来改变区域 
<br><br>
<h3 id="worn">雷区警告</h2>
&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp 1. Chrom浏览器测试雷区：Chrom浏览器高级设置accept-language时，默认的中文无法移除，即使我们添加了英语并且置顶，在请求时，使用Request获取的Local是置顶的en，但是LocaleContextHolder获取的Local依然是zh。火狐是OK的，它可以随意修改accept-language的值。
<br>&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp 2. Linux测试雷区：都知道Linux下命令 curl -v http://url    -H 'Accept-Language: en_US' 可以修改发起请求的accetp-language，但是在测试时发现这样的话获取不到Local，Local不是null，而是""空字符串。Be careful ！