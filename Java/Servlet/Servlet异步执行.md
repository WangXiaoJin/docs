# Servlet异步执行


## Servlet原生功能

* [Tomcat的异步Servlet实现原理](https://zhuanlan.zhihu.com/p/22018499)
* [JEE 6 and Spring MVC](http://codingjunkie.net/jee-6-and-spring-mvc/)
* [Asynchronous processing support in Servlet 3.0 - 讲解了部分HTTP内部机制 -【重要】](https://www.javaworld.com/article/2077995/java-concurrency-asynchronous-processing-support-in-servlet-3-0.html)
* [Tenfold increase in server throughput with Servlet 3.0 asynchronous processing](https://www.nurkiewicz.com/2011/03/tenfold-increase-in-server-throughput.html)
    * [`Token bucket` - 控制带宽](https://en.wikipedia.org/wiki/Token_bucket)

## Spring MVC 实现

* [Spring MVC 3.2 Preview: Introducing Servlet 3, Async Support](https://spring.io/blog/2012/05/07/spring-mvc-3-2-preview-introducing-servlet-3-async-support/)
* [Spring MVC 3.2 Preview: Techniques for Real-time Updates](https://spring.io/blog/2012/05/08/spring-mvc-3-2-preview-techniques-for-real-time-updates/)
    * Traditional Polling
    * Long Polling
    * HTTP Streaming
    * WebSocket Protocol
* [Spring MVC 3.2 Preview: Making a Controller Method Asynchronous -【重要】](https://spring.io/blog/2012/05/10/spring-mvc-3-2-preview-making-a-controller-method-asynchronous/)
    * 讲解原理
    * [例子](https://github.com/spring-projects/spring-mvc-showcase/blob/master/src/main/java/org/springframework/samples/mvc/async/CallableController.java#L13)
* [Spring MVC 3.2 Preview: Adding Long Polling to an Existing Web Application](https://spring.io/blog/2012/05/14/spring-mvc-3-2-preview-adding-long-polling-to-an-existing-web-application)
    * DeferredResult例子
* [Spring MVC 3.2 Preview: Chat Sample](https://spring.io/blog/2012/05/16/spring-mvc-3-2-preview-chat-sample/)

* [Asynchronous Requests - SpringMVC官网](https://docs.spring.io/spring/docs/5.1.9.RELEASE/spring-framework-reference/web.html#mvc-ann-async)

The MVC configuration exposes the following options related to asynchronous request processing:

* Java configuration: Use the `configureAsyncSupport` callback on `WebMvcConfigurer`.
* XML namespace: Use the `<async-support>` element under `<mvc:annotation-driven>`.

You can configure the following:

* Default timeout value for async requests, which if not set, depends on the underlying Servlet container (for example, 10 seconds on Tomcat).
* `AsyncTaskExecutor` to use for blocking writes when streaming with Reactive Types and for executing `Callable` instances returned from controller methods.
We highly recommended configuring this property if you stream with reactive types or have controller methods that return `Callable`, 
since by default, it is a `SimpleAsyncTaskExecutor` （**参考`RequestMappingHandlerAdapter`**）.
* `DeferredResultProcessingInterceptor` implementations and `CallableProcessingInterceptor` implementations.

Note that you can also set the default timeout value on a DeferredResult, a ResponseBodyEmitter, and an SseEmitter. For a Callable, you can use WebAsyncTask to provide a timeout value.

```java
/**
 * @author WangXiaoJin
 * @date 2019-09-03 22:34
 */
@Configuration
public class MvcConfig implements WebMvcConfigurer {

    /**
     * 用于处理异步请求的线程池，创建测线程前缀可以通过{@link ThreadPoolTaskExecutor#setThreadNamePrefix(String)}配置，
     * 不配置则使用此 beanName 作为前缀。
     *
     * @return AsyncTaskExecutor
     */
    @Bean
    public AsyncTaskExecutor mvcAsync() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        int procs = Runtime.getRuntime().availableProcessors();
        executor.setCorePoolSize(procs);
        executor.setMaxPoolSize(procs * 2);
        executor.setQueueCapacity(100);
        executor.setKeepAliveSeconds(60);
        // 允许 CorePool 的线程超时并销毁
        executor.setAllowCoreThreadTimeOut(true);
        // 等待任务执行完再关闭
        executor.setWaitForTasksToCompleteOnShutdown(true);
        // 终止时等待的最大时间
        executor.setAwaitTerminationSeconds(120);
        return executor;
    }

    /**
     * 配置异步请求的处理
     *
     * @param config 配置对象
     * @see RequestMappingHandlerAdapter
     */
    @Override
    public void configureAsyncSupport(AsyncSupportConfigurer config) {
        // 处理异步请求的超时时间
        config.setDefaultTimeout(30000);
        // 默认：new SimpleAsyncTaskExecutor("MvcAsync") ，参考 RequestMappingHandlerAdapter
        config.setTaskExecutor(mvcAsync());
    }

}
```

```java
package org.springframework.samples.mvc.async;

import java.util.concurrent.Callable;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.context.request.async.WebAsyncTask;

@Controller
@RequestMapping("/async/callable")
public class CallableController {


	@GetMapping("/response-body")
	public @ResponseBody Callable<String> callable() {

		return new Callable<String>() {
			@Override
			public String call() throws Exception {
				Thread.sleep(2000);
				return "Callable result";
			}
		};
	}

	@GetMapping("/view")
	public Callable<String> callableWithView(final Model model) {
		return () -> {
			Thread.sleep(2000);
			model.addAttribute("foo", "bar");
			model.addAttribute("fruit", "apple");
			return "views/html";
		};
	}

	@GetMapping("/exception")
	public @ResponseBody Callable<String> callableWithException(
			final @RequestParam(required=false, defaultValue="true") boolean handled) {

		return () -> {
			Thread.sleep(2000);
			if (handled) {
				// see handleException method further below
				throw new IllegalStateException("Callable error");
			}
			else {
				throw new IllegalArgumentException("Callable error");
			}
		};
	}

	@GetMapping("/custom-timeout-handling")
	public @ResponseBody WebAsyncTask<String> callableWithCustomTimeoutHandling() {
		Callable<String> callable = () -> {
			Thread.sleep(2000);
			return "Callable result";
		};
		return new WebAsyncTask<String>(1000, callable);
	}

	@ExceptionHandler
	@ResponseBody
	public String handleException(IllegalStateException ex) {
		return "Handled exception: " + ex.getMessage();
	}

}
```

```java
package org.springframework.samples.mvc.async;

import java.util.Queue;
import java.util.concurrent.ConcurrentLinkedQueue;

import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.context.request.async.DeferredResult;
import org.springframework.web.servlet.ModelAndView;

@Controller
@RequestMapping("/async")
public class DeferredResultController {

	private final Queue<DeferredResult<String>> responseBodyQueue = new ConcurrentLinkedQueue<>();

	private final Queue<DeferredResult<ModelAndView>> mavQueue = new ConcurrentLinkedQueue<>();

	private final Queue<DeferredResult<String>> exceptionQueue = new ConcurrentLinkedQueue<>();


	@GetMapping("/deferred-result/response-body")
	public @ResponseBody DeferredResult<String> deferredResult() {
		DeferredResult<String> result = new DeferredResult<>();
		this.responseBodyQueue.add(result);
		return result;
	}

	@GetMapping("/deferred-result/model-and-view")
	public DeferredResult<ModelAndView> deferredResultWithView() {
		DeferredResult<ModelAndView> result = new DeferredResult<>();
		this.mavQueue.add(result);
		return result;
	}

	@GetMapping("/deferred-result/exception")
	public @ResponseBody DeferredResult<String> deferredResultWithException() {
		DeferredResult<String> result = new DeferredResult<>();
		this.exceptionQueue.add(result);
		return result;
	}

	@GetMapping("/deferred-result/timeout-value")
	public @ResponseBody DeferredResult<String> deferredResultWithTimeoutValue() {

		// Provide a default result in case of timeout and override the timeout value
		// set in src/main/webapp/WEB-INF/spring/appServlet/servlet-context.xml

		return new DeferredResult<>(1000L, "Deferred result after timeout");
	}

	@Scheduled(fixedRate=2000)
	public void processQueues() {
		for (DeferredResult<String> result : this.responseBodyQueue) {
			result.setResult("Deferred result");
			this.responseBodyQueue.remove(result);
		}
		for (DeferredResult<String> result : this.exceptionQueue) {
			result.setErrorResult(new IllegalStateException("DeferredResult error"));
			this.exceptionQueue.remove(result);
		}
		for (DeferredResult<ModelAndView> result : this.mavQueue) {
			result.setResult(new ModelAndView("views/html", "javaBean", new JavaBean("bar", "apple")));
			this.mavQueue.remove(result);
		}
	}

	@ExceptionHandler
	@ResponseBody
	public String handleException(IllegalStateException ex) {
		return "Handled exception: " + ex.getMessage();
	}

}
```

## 其他参考文献

* [HTTP1.0、HTTP1.1 和 HTTP2.0 的区别](https://juejin.im/entry/5981c5df518825359a2b9476)
* [spring-mvc-showcase - SpringMVC例子](https://github.com/spring-projects/spring-mvc-showcase) 
