## Spring MVC

#### 集成Freemarker

* 查看Freemarker所有可用的变量
    * `AbstractView#render()` - 配置了`exposePathVariables`/`staticAttributes`/`requestContextAttribute`
    * `AbstractTemplateView#renderMergedOutputModel()` - 配置了`exposeRequestAttributes`/`exposeSessionAttributes`/`exposeSpringMacroHelpers`
    * `FreeMarkerView#doRender()` - 注入了`JspTaglibs`/`Session`/`Application`/`Request`/`RequestParameters`等对象

* Freemarker获取变量值顺序如下，直到找到位置。参考：`freemarker.ext.servlet.AllHttpScopesHashModel#get(key)`
    1. 从`Mvc Model`中获取
    2. 通过`request.getAttribute(key)`获取
    3. 通过`session.getAttribute(key)`获取
    4. 通过`context.getAttribute(key)`获取
    5. 还没找到则返回`null`