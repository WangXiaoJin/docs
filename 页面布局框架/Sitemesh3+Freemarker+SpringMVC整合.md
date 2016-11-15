## Sitemesh3+Freemarker+SpringMVC整合

当整合Sitemesh3+Freemarker+SpringMVC时，你会发现decorator.html页面并不能使用Freemarker相关的功能。因为这个页面是由Sitemesh请求forward的资源，并没有走SpringMVC相关逻辑。在网上搜了下，多数网友都是用Filter来实现的，并没有和SpringMVC完美的融合，所以就自己整理了一个新的方案。  

既然我们用到了SpringMVC和Freemarker，那decorator.html页面就应该交给SpringMVC来处理，这样可以共用同一份配置同时可以应用SpringMVC提供的相关功能。实现如下：

1. web.xml - 配置Sitemesh3拦截器

	```xml
	<filter>
		<filter-name>sitemesh</filter-name>
		<filter-class>com.easycodebox.common.sitemesh3.DefaultConfigurableSiteMeshFilter</filter-class>
	</filter>
	<filter-mapping>
		<filter-name>sitemesh</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
	```

	> 为了不受其他拦截器影戏，此拦截器最好是放在其他Filter下面。纯属建议，根据个人情况而定，只要不出问题放哪都行。

2. 配置sitemesh3.xml

	```xml
	<?xml version="1.0" encoding="UTF-8"?>
	<sitemesh>
	
		<!-- 默认装饰器，所有的请求都会走这个装饰器 -->
		<mapping decorator="/decorator" />
	
		<!-- Exclude path from decoration. -->
		<mapping path="/css/*" exclude="true" />
		<mapping path="/errors/*" exclude="true" />
		<mapping path="/imgs/*" exclude="true" />
		<mapping path="/js/*" exclude="true" />
		
	</sitemesh>
	```
	
	> `<mapping decorator="/decorator" />`这个配置是重点。多数人都是这样配置的`<mapping decorator="/WEB-INF/pages/decorator.html" />`，这样配置的话就是说直接forward到`/WEB-INF/pages/decorator.html`页面，因为我相信你的SpringMVC不会配置这样请求的Controller的。而`/decorator`是forward到这个请求，首先会被SpringMVC拦截，如果SpringMVC配置了这样请求的Controller则执行此Controller，否则会查找项目中有没有路径为`/decorator`的静态资源，有则返回，没有则报404.

3. SpringMVC配置
	
	```xml
	<bean id="localeResolver" class="org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver" />

	<!-- freemarker的配置 -->
	<bean id="freemarkerConfig" class="com.easycodebox.common.web.springmvc.FreeMarkerConfigurer"
		p:freemarkerVariables-ref="properties"
		p:templateLoaderPath="${freemarker.loader_path}"
		p:defaultEncoding="${freemarker.default_encoding}" />

	<!-- freemarker视图设置 -->
	<bean id="viewResolver"
		class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver"
		p:prefix="/WEB-INF/pages/" p:suffix=".html" p:order="1"
		p:contentType="text/html;charset=utf-8"
		p:viewClass="org.springframework.web.servlet.view.freemarker.FreeMarkerView"
		p:exposeRequestAttributes="true"
		p:exposeSpringMacroHelpers="true"
		p:requestContextAttribute="request" />
	
	<!-- Support static resource -->
	<mvc:default-servlet-handler />
	```	
	
	> 以上SpringMVC配置我只贴了与整合相关的部分，其他配置自己补充

4. Controller代码

	```java
	@Resource
	private AbstractTemplateViewResolver viewResolver;
	
	@Resource
	private LocaleResolver localeResolver;

	@RequestMapping("/decorator")
	public void decorator(HttpServletRequest request, HttpServletResponse response) throws Exception {
		AbstractTemplateView view = (AbstractTemplateView)viewResolver.resolveViewName("decorator", 
				localeResolver.resolveLocale(request));
		//因为此请求是Sitemesh forward进来的，如果不做下面配置的话会重复设置参数，而Spring MVC碰到重复参数名会抛异常
		//详细逻辑请阅读 {@link AbstractTemplateView} 的 renderMergedOutputModel方法
		view.setAllowRequestOverride(true);
		view.setExposeSpringMacroHelpers(false);
		view.render(null, request, response);
	}
	```

5. decorator.html

	```html
	<!DOCTYPE HTML>
	<html>
	<head>
	[#include "/WEB-INF/common/meta.html"/]
	<title><sitemesh:write property='title' /></title>
	[#include "/WEB-INF/common/styles.html"/]
	<sitemesh:write property='head' />
	</head>
	<body class="hold-transition skin-blue sidebar-mini">
	<div class="wrapper">
	
		[#include "/WEB-INF/common/header.html"/]
		
		[#include "/WEB-INF/common/left.html"/]
		
		<div id="pjax-container" class="content-wrapper">
			
			<sitemesh:write property='body' />
			
		</div>
		
		[#include "/WEB-INF/common/footer.html"/]
		
		[#include "/WEB-INF/common/control-sidebar.html"/]
		
	</div>
	[#include "/WEB-INF/common/scripts.html"/]
	</body>
	</html>
	```

**整合到此就结束了，是不是比其他方案更简单，更加顺畅**