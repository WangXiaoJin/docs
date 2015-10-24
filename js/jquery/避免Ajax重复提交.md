## 避免Ajax重复提交

#####　`【原创】`转载时标明来源，请支持原创 :heart_eyes:
---

　　当点击按钮触发Ajax请求时，由于点击的过快，点击了多次，会引起发
送了多次请求，后端逻辑被执行了多次。本文章提供了解决方案：

###JS代码如下：

```javascript
/**
 * 防止ajax重复提交
 * $.ajax({
 *   submitBtn: "#btn",  //【必须】触发请求的按钮，可以是函数、jquery选择器、jquery对象、dom对象
 *   disableBtns: "#btn1",//【可选】当submitBtn正在请求时，一起失效的按钮。
 *      					可以是函数、jquery选择器、jquery对象、dom对象、数组
 *   IngClass:  "class1 class2",	//【可选】按钮正在提交时显示的class，默认会自动加上submiting
 *   submiting: "保存..." //【可选】正在请求中的提示信息，默认为{0}...
 * });
 */
$(document).off(".repeatSubmit").on({
    "ajaxSend.repeatSubmit": function (event, jqXHR, opts) {
        function $obj(obj) {
            if (obj == null) return obj;
            if (Object.prototype.toString.call(obj) === '[object Function]') {
                obj = obj();
            }
            if (Object.prototype.toString.call(obj) === '[object Array]') {
                var $objs =  null;
                obj.forEach(function (item) {
                    var tmp = $obj(item);
                    $objs = $objs == null ? tmp : tmp == null ? $objs : $objs.add(tmp);
                });
                return $objs;
            } else {
                return obj.jquery ? obj : $(obj);
            }
        }
        if (opts.submitBtn = $obj(opts.submitBtn)) {
            opts.submiting = opts.submiting != null ? opts.submiting : "{0}...";
            //注：如果你的项目用到bootstrap，可以考虑吧"submiting" 换成 "submiting disabled"，
            //因为bootstrap的disabled样式已经写好，且比较完善。
            //或者你可以通过$.ajaxSetup设置默认参数
            opts.IngClass = opts.IngClass == null ? "submiting" : opts.IngClass + " submiting";

            if (opts.submitBtn.data("submiting")) {
                jqXHR.abort();
                return false;
            }
            function changeVal($btn, isSbmtBtn) {
                var data = $btn.data();
                if (data.submiting) return false;
                data.submiting = true;
                if (!data.hasOwnProperty("originalText")) {
                    data.valProp = $btn[0].tagName == "INPUT" || $btn[0].tagName == "TEXTAREA" ? true : false;
                    data.disabledProp = data.valProp || $btn[0].tagName == "BUTTON" ? true : false;
                    data.originalText = data.valProp ? $btn.val() : $btn.text();
                }
                $btn.addClass(opts.IngClass);
                if (isSbmtBtn) {
                    if (data.valProp) {
                        $btn.val(opts.submiting.replace(/\{0\}/g, data.originalText));
                    } else {
                        $btn.text(opts.submiting.replace(/\{0\}/g, data.originalText));
                    }
                }
                if (data.disabledProp)
                    $btn.prop("disabled", true);
            }
            changeVal(opts.submitBtn, true);
            if (opts.disableBtns = $obj(opts.disableBtns)) {
                opts.disableBtns.each(function () {
                    changeVal($(this), false);
                });
            }
        }
    },
    "ajaxComplete.repeatSubmit": function (event, jqXHR, opts) {
        if (opts.submitBtn && opts.submitBtn.data("submiting")) {
            function changeVal($btn, isSbmtBtn) {
                var data = $btn.data();
                if (!data.submiting) return false;
                data.submiting = false;
                $btn.removeClass(opts.IngClass);
                if (isSbmtBtn) {
                    if (data.valProp) {
                        $btn.val(data.originalText);
                    } else {
                        $btn.text(data.originalText);
                    }
                }
                if (data.disabledProp)
                    $btn.prop("disabled", false);
            }
            changeVal(opts.submitBtn, true);
            if (opts.disableBtns) {
                opts.disableBtns.each(function () {
                    changeVal($(this), false);
                });
            }
        }
    }
});
```
---

###CSS代码如下：

```css
/* 如果你的项目有在用bootstrap，可以考虑用bootstrap的.disabled的样式 */
.submiting {
    cursor: not-allowed !important; 
    opacity: 0.8;
}
```
---

###使用例子：

```javascript
//例子1 最简单写法
$("#btn").click(function () {
    $.ajax({
        url: "...",
        submitBtn: "#btn",  //【必须】触发请求的按钮
        success: function (data) {
            
        }
    });
});

//例子2
$("#btn").click(function () {
    $.ajax({
        url: "...",
        submitBtn: this,  //【必须】触发请求的按钮，this为$("#btn")
        disableBtns: "#btn1",//【可选】当submitBtn正在请求时，一起失效的按钮。可以是函数、jquery选择器、jquery对象、dom对象、数组
        //disableBtns: "#btn1, #btn2", 		//【可选】
        //disableBtns: ["#btn1", "#btn2"],	//【可选】
        IngClass: "class1 class2", //【可选】正在请求时显示的样式
        submiting: "保存...", //【可选】正在请求中的提示信息
        success: function (data) {
            
        }
    });
});

//例子3
$("#btn").click(function () {
    $.ajax({
        url: "...",
        submitBtn: $("#btn"),  //【必须】触发请求的按钮
        submiting: "拼死保存中...", //【可选】正在请求中的提示信息
        success: function (data) {
            
        }
    });
});

//例子4
$("#btn").click(function () {
    $.ajax({
        url: "...",
        submitBtn: function() {
			return anyVal > 1 ? "#btn1" : "#btn2";
		},
        submiting: "{0}...", //【可选】{0}代表原有按钮的内容
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

