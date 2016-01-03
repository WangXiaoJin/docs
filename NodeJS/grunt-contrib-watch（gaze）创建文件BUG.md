## grunt-contrib-watch（gaze）创建文件BUG

#####　`【参考】` <http://stackoverflow.com/questions/31679375/grunt-contrib-watch-doesnt-see-new-files> 、 <http://stackoverflow.com/questions/24296839/why-dont-newly-added-files-trigger-my-gulp-watch-task>
---

grunt-contrib-watch：v0.6.1  
gaze：v0.5.2

当使用grunt-contrib-watch模块监听文件变动时，你可能会碰到创建新文件时，并没有触发`added`事件。这个问题归根于grunt-contrib-watch模块使用了`gaze`模块，而`gaze`存在如下问题：

1. 当路径以`./`开头，不触发`added`事件（**解决方案**：去除`./`）
2. 当路径为绝对路径`'/data/html/css/**/*.css'`，不触发`added`事件（**解决方案**：换成相对路径`'../html/css/**/*.css'`）
3. 当被监听的路径为空文件夹，不触发`added`事件。例如：当监听路径为`'css/**/*.css'`且`css`为空文件夹时，不触发`added`事件。（**解决方案**：监听路径改为`['css', 'css/**/*.css']`）

> `gaze（v0.5.2）` 使用时需要注意两点：  
> 1、`'js/**/*.js'`路径只会监听有文件后缀名为`.js`所在文件夹，只有在此文件夹下创建文件或文件夹才会触发事件。例如：有真是路径为`js/a/b/file.js`，且`js/a`文件夹里没有后缀为`.js`的文件，当你`js/a`文件夹下面创建文件或文件夹是不会触发`added`事件的，但是在`js/a/b`文件夹下面创建就会触发`added`事件，因为其下面`file.js`符合规则。这一点非常重要，总有人会以为上面的匹配规则能监听`js/`下面所有的目录。如果要监听指定目录下面的所有子层级目录，则必须使用 `js/**` 。  
> 2、监听规则有可继承性。即`js/a/b/file.js`匹配路径规则时，在`js/a/b`下面创建的文件夹及其子文件（'*.js'格式）也会被监听，会触发相同事件、相同`tasks`。

###例子

1、 **无效写法：**

```javascript
watch: {
    css: {
        files: ['./css/**/*.css', '!./css/**/*.min.css'],
        tasks: ['cssmin:batch']
    }
}
```

**有效写法：**

```javascript
watch: {
    css: {
        files: ['css/**/*.css', '!css/**/*.min.css'],
        tasks: ['cssmin:batch']
    }
}
```

2、 **无效写法：**

```javascript
watch: {
    css: {
        files: ['/data/html/css/**/*.css', '!/data/html/css/**/*.min.css'],
        tasks: ['cssmin:batch']
    }
}
```

**有效写法：**

```javascript
watch: {
    css: {
        files: ['../html/css/**/*.css', '!../html/css/**/*.min.css'],
        tasks: ['cssmin:batch']
    }
}
```

3、 **无效写法：**

```javascript
watch: {
    css: {
    	//css文件夹为空文件夹
        files: ['css/**/*.css', '!css/**/*.min.css'],
        tasks: ['cssmin:batch']
    }
}
```

**有效写法：**

```javascript
watch: {
    css: {
        files: ['css', 'css/**/*.css', '!css/**/*.min.css'],
        tasks: ['cssmin:batch']
    }
}
```
