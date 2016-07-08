## Select2笔记

#####　`【原创】`
---

###1. 自定义search规则

```html

<link href="css/select2.css" rel="stylesheet" />
<script type="text/javascript" src="js/jquery-2.1.0.js"></script>
<script type="text/javascript" src="js/select2.full.js"></script>
<script type="text/javascript" src="js/i18n/zh-CN.js"></script>

<select id="select" style="width: 200px;">
	<option value="A">XiaoMing</option>
	<option value="B">XiaoDong</option>
	<option value="C">XiaoXi</option>
</select>

```

```javascript

$("#select").select2({
	//从头部开始匹配
	matcher: function(params, data) {
		params.term = params.term || '';
	    if (data.text.toUpperCase().indexOf(params.term.toUpperCase()) == 0) {
	        return data;
	    }
	    return false;
	}
});

```

###2. Ajax获取数据

1. HTML

	```html
	
	<link href="css/select2.css" rel="stylesheet" />
	<script type="text/javascript" src="js/jquery-2.1.0.js"></script>
	<script type="text/javascript" src="js/select2.full.js"></script>
	<script type="text/javascript" src="js/i18n/zh-CN.js"></script>
	
	<select id="select" style="width: 200px;">
	</select>
	
	```

2. JSON数据

	```json
	{
		"total_count": "60",
		"items": [
			{"id": "1", "name": "text1"},
			{"id": "2", "name": "text2"},
			{"id": "3", "name": "text3"},
			{"id": "4", "name": "text4"},
			{"id": "5", "name": "text5"},
			{"id": "6", "name": "text6"},
			{"id": "7", "name": "text7"},
			{"id": "8", "name": "text8"},
			{"id": "9", "name": "text9"},
			{"id": "10", "name": "text10"},
			{"id": "11", "name": "text11"},
			{"id": "12", "name": "text12"},
			{"id": "13", "name": "text13"},
			{"id": "14", "name": "text14"},
			{"id": "15", "name": "text15"},
			{"id": "16", "name": "text16"},
			{"id": "17", "name": "text17"},
			{"id": "18", "name": "text18"},
			{"id": "19", "name": "text19"},
			{"id": "20", "name": "text20"}
		]
	}
	```

3. JS

	```javascript
	
	$("#select").select2({
		ajax: {
			url: "./test.json",
			dataType: 'json',
			//延迟250毫秒后发送请求
			delay: 250,
			data: function(params) {
				return {
					name: params.term,
					page: params.page
				};
			},
			//转换数据格式
			processResults: function(data, params) {
				params.page = params.page || 1;
				return {
					results: data.items,
					pagination: {
						more: (params.page * 20) < data.total_count
					}
				};
			},
			cache: true
		},
		//请求远程数据后格式化数据，Ajax请求必须重写此函数
		templateResult: function(repo) {
			if (repo.loading) return repo.text;
			return repo.name;
		},
		//选中后显示的内容，Ajax请求必须重写此函数
		templateSelection: function(repo) {
			return repo.name;
		}
	});
	
	```
