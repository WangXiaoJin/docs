## Ajax事件详解 ##
　　Ajax事件有全局事件和局部事件之分。全局事件，所有ajax请求都会
触发。局部事件只会监听单次请求。以下顺序为事件触发的顺序：

1. ajaxStart - `全局`事件  
  ajaxStart和ajaxStop和其他的事件处理机制不一样，这两个事件在同一个ajax请求队列中只会被执行`一次`。在触发此事件时会检查当前有触发过此事件，且没有执行完成的请求（即跑完ajaxStop），如果有则不会执行此ajaxStart事件，没有才会执行。ajaxStop也是同理。

> `注意`：当Ajax请求的参数`global = false`时，不会把此次请求计算在内，因为次请求根本就不会触发全局事件。 **此事件非常适合多次请求时显示`正在加载...`的提示信息，因为只会触发一次。**  
`例子`：当前有三个请求，第一个发送请求的会触发ajaxStart事件，最后一个执行完成的请求触发ajaxStop事件。 

```javascript
	//当你绑定两次ajaxStart事件时，肯定会触发两次。同样适用于ajaxStop。
	$(document).ajaxStart(function () {
        console.log("执行ajaxStart - A");
    }).ajaxStart(function () {
        console.log("执行ajaxStart - B");
    });
	//你会看到A、B都会打印的
```

2. beforeSend - `局部`事件  
  当beforeSend `return false`时，会`阻止`此次请求，但紧随其后会触发`ajaxError`、
`ajaxComplete`、`ajaxStop`事件。
	beforeSend(jqXHR jqXHR, PlainObject settings)

3. ajaxSend - `全局`事件  
  当且仅当beforeSend `return false`时不会触发。
	ajaxSend(Event event, jqXHR jqXHR, PlainObject ajaxOptions)

4. success - `局部`事件  
  只有在请求**`成功`**后才会执行此事件。
	success(Anything data, String textStatus, jqXHR jqXHR)

```javascript
	$.ajax({
        url: "...",
        success: function (data) {
            console.log("AA");
        }
    });

	$.ajax({
        url: "...",
        success: [function () {
            console.log("AA");
        }, function () {
            console.log("BB");
        }]
    });
	//success可以传数组或单个函数。当传数组时，数组里面每个函数都会执行。
```

6. ajaxSuccess - `全局`事件  
  只有在请求**`成功`**后才会执行此事件。
	ajaxSuccess(Event event, jqXHR jqXHR, PlainObject ajaxOptions, PlainObject data)

```javascript
	$( document ).ajaxSuccess(function(event, xhr, settings, data) {
  		if (settings.url === "ajax/test.html") {
    		$(".log").text( "Triggered ajaxComplete handler. The result is " +
      			xhr.responseText );
	  	}
	});
	//可以通过settings参数来过滤执行想要执行的请求
```
> 注意：如果想要获取请求内容，可以通过`xhr.responseXML` 或 `xhr.responseText` 获取对应的`XML`、`HTML`。

7. error - `局部`事件  
  只有在请求**`失败`**后才会执行此事件。
	error(jqXHR jqXHR, String textStatus, String errorThrown)
	textStatus值为：`null`,`"timeout"`,`"error"`,`"abort"`,`"parsererror"`。当发生HTTP请求错误时，errorThrown值为`"Not Found"`、`"Internal Server Error."`等

```javascript
	$.ajax({
        url: "...",
        error: function () {
            console.log("AA");
        }
    });

	$.ajax({
        url: "...",
        error: [function () {
            console.log("AA");
        }, function () {
            console.log("BB");
        }]
    });
	//success可以传数组或单个函数。当传数组时，数组里面每个函数都会执行。
```

> 注意：当跨域script或JSONP请求发生错误时，不会触发此事件。

8. ajaxError - `全局`事件  
  只有在请求**`失败`**后或beforeSend `return false`时才会执行此事件。
	ajaxError(Event event, jqXHR jqXHR, PlainObject ajaxSettings, String thrownError)
> 注意：当跨域script或JSONP请求发生错误时，不会触发此事件。

9. complete - `局部`事件  
  无论请求`失败或成功`都会触发此函数，当且仅当beforeSend `return false`时不会触发。
	complete(jqXHR jqXHR, String textStatus)
	textStatus值：`"success"`, `"notmodified"`, `"nocontent"`, `"error"`, `"timeout"`, `"abort"`, `"parsererror"`

```javascript
	$.ajax({
        url: "...",
        success: function (data) {
            
        },
        complete: function () {
            console.log("AA");
        }
    });

	$.ajax({
        url: "...",
        success: function (data) {
            
        },
        complete: [function () {
            console.log("AA");
        }, function () {
            console.log("BB");
        }]
    });
	//complete可以传数组或单个函数。当传数组时，数组里面每个函数都会执行。
```

10. ajaxComplete - `全局`事件  
  始终都会触发此事件。
	ajaxComplete(Event event, jqXHR jqXHR, PlainObject ajaxOptions)

```javascript
	$( document ).ajaxComplete(function(event, xhr, settings) {
  		if (settings.url === "ajax/test.html") {
    		$(".log").text( "Triggered ajaxComplete handler. The result is " +
      			xhr.responseText );
	  	}
	});
	//可以通过settings参数来过滤执行想要执行的请求
	//xhr.responseText 是请求返回的内容
```

11. ajaxStop - `全局`事件  
  参考ajaxStart

> **`注意`：自JQuery1.8以后，全局事件只能绑定在documnet上，当`global = false`时不会触发全局事件。**
