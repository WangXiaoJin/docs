## 避免Ajax重复提交

#####　`【原创】`转载时标明来源，请支持原创 :heart_eyes:
---

　　当点击按钮触发Ajax请求时，由于点击的过快，点击了多次，会引起发
送了多次请求，后端逻辑被执行了多次。本文章提供了解决方案：

###JS代码如下：

```javascript
	$(document).ajaxSend(function (event, jqXHR, opts) {
        //opts.submitBtn 可以是函数、jquery选择器、jquery对象、dom对象
        function $obj(obj) {
            if (Object.prototype.toString.call(obj) === '[object Function]') {
                obj = obj();
            }
            return obj == null || obj.jquery ? obj : $(obj);
        }
        if (opts.submitBtn = $obj(opts.submitBtn)) {
            var $btn = opts.submitBtn,
                data = $btn.data();
            if (data.submiting) {
                jqXHR.abort();
                return false;
            }
            opts.submiting = opts.submiting != null ? opts.submiting : "{0}...";

            data.submiting = true;
            if (!data.hasOwnProperty("originalText")) {
                data.valProp = $btn[0].tagName == "INPUT" || $btn[0].tagName == "TEXTAREA" ? true : false;
                data.disabledProp = data.valProp || $btn[0].tagName == "BUTTON" ? true : false;
                data.originalText = data.valProp ? $btn.val() : $btn.text();
            }
            if (data.valProp) {
                $btn.val(opts.submiting.replace(/\{0\}/g, data.originalText)).addClass("submiting");
            } else {
                $btn.text(opts.submiting.replace(/\{0\}/g, data.originalText)).addClass("submiting");
            }
            if (data.disabledProp)
                $btn.prop("disabled", true);
        }
    }).ajaxComplete(function (event, jqXHR, opts) {
        if (opts.submitBtn && opts.submitBtn.data("submiting")) {
            var $btn = opts.submitBtn,
                data = opts.submitBtn.data();
            data.submiting = false;
            if (data.valProp) {
                $btn.val(data.originalText).removeClass("submiting");
            } else {
                $btn.text(data.originalText).removeClass("submiting");
            }
            if (data.disabledProp)
                $btn.prop("disabled", false);
        }
    });
```
---

###CSS代码如下：

```css
	.submiting {
        cursor: not-allowed !important; 
        opacity: 0.8;
    }
```
---

###使用例子：

```javascript
	//例子1
	$("#btn").click(function () {
        $.ajax({
            url: "...",
            submitBtn: "#btn",  //触发请求的按钮
            submiting: "保存...", //正在请求中的提示信息
            success: function (data) {
                
            }
        });
    });

	//例子2
	$("#btn").click(function () {
        $.ajax({
            url: "...",
            submitBtn: $("#btn"),  //触发请求的按钮
            submiting: "拼死保存中...", //正在请求中的提示信息
            success: function (data) {
                
            }
        });
    });

	//例子3
	$("#btn").click(function () {
        $.ajax({
            url: "...",
            submitBtn: function() {
				return anyVal > 1 ? "#btn1" : "#btn2";
			},
            submiting: "{0}...", //{0}代表原有按钮的内容
            success: function (data) {
                
            }
        });
    });
	
```

```html
	//例子1
	<button type="button" id="btn1" >保存</button>

	//例子2
	<input type="button" id="btn2" value="保存" />
	
	//例子3
    <div id="btn3">保存</div>

	//例子4
    <a id="btn4" href="javascript:void(0);" >保存</a>
```

> 提示： 想进一步了解Ajax事件的，请参考[Ajax事件](./Ajax事件--ajaxStart、ajaxSend、ajaxSuccess、ajaxComplete等.md)

