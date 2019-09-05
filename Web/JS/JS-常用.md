## 常用

* **JS二进制、十进制、十六进制转换**
    
    * 2的8次方 - `Math.pow(2, 8)`
    * 二进制转十进制 - `parseInt("111", 2)`
    * 二进制表示 - `0B11 === 3`  / `0b11 === 3`
    * 八进制表示 - `0O11 === 9` / `0o11 === 9`
    * 十六进制表示 - `0x11 === 17` / `0X11 === 17`
    * 十进制转二进制 - `new Number("11").toString(2)`
    * 十进制转十六进制 - `new Number("11").toString(16)`
    * 十六进制转二进制 - `new Number("0x11").toString(2)`

* JS字符窜与Unicode互转
    
    * `var str = '\u975e'` -> `'非'`
    * `'𠮷'.charCodeAt(1).toString(16)` -> `'dfb7'`


* ES6 - **Template Strings**

    ```javascript
    // Basic literal string creation
    `This is a pretty little template string.`
    
    // Multiline strings
    `In ES5 this is
     not legal.`
    
    // Interpolate variable bindings
    var name = "Bob", time = "today";
    `Hello ${name}, how are you ${time}?`
    
    // Unescaped template strings
    String.raw`In ES5 "\n" is a line-feed.`
    
    // 注：此语法的值作为参数传递时，两边的`(` `)`可省略
    
    // Construct an HTTP request prefix is used to interpret the replacements and construction
    GET`http://foo.org/bar?a=${a}&b=${b}
        Content-Type: application/json
        X-Credentials: ${credentials}
        { "foo": ${foo},
          "bar": ${bar}}`(myOnReadyStateChangeHandler);
    ```

* ES6指数运算符: `**` - `2 ** 3 === 8` / `Math.pow(2, 3) === 2 ** 3`

#### 语法

* [`Abstract Equality Comparison` and `Strict Equality Comparison`](https://tc39.github.io/ecma262/#sec-abstract-equality-comparison)

* ES6 - `Proxies`

    ```javascript
    // Proxying a normal object
    var target = {};
    var handler = {
      get: function (receiver, name) {
        return `Hello, ${name}!`;
      }
    };
    
    var p = new Proxy(target, handler);
    p.world === "Hello, world!";
    ```
    
    ```javascript
    // Proxying a function object
    var target = function () { return "I am the target"; };
    var handler = {
      apply: function (receiver, ...args) {
        return "I am the proxy";
      }
    };
    
    var p = new Proxy(target, handler);
    p() === "I am the proxy";
    ```
    
    There are traps available for all of the runtime-level meta-operations:
    ```javascript
    var handler =
    {
      // target.prop
      get: ...,
      // target.prop = value
      set: ...,
      // 'prop' in target
      has: ...,
      // delete target.prop
      deleteProperty: ...,
      // target(...args)
      apply: ...,
      // new target(...args)
      construct: ...,
      // Object.getOwnPropertyDescriptor(target, 'prop')
      getOwnPropertyDescriptor: ...,
      // Object.defineProperty(target, 'prop', descriptor)
      defineProperty: ...,
      // Object.getPrototypeOf(target), Reflect.getPrototypeOf(target),
      // target.__proto__, object.isPrototypeOf(target), object instanceof target
      getPrototypeOf: ...,
      // Object.setPrototypeOf(target), Reflect.setPrototypeOf(target)
      setPrototypeOf: ...,
      // for (let i in target) {}
      enumerate: ...,
      // Object.keys(target)
      ownKeys: ...,
      // Object.preventExtensions(target)
      preventExtensions: ...,
      // Object.isExtensible(target)
      isExtensible :...
    }
    ```

#### 参考文档

* [`Can I Use` -【重要】](https://caniuse.com/) - Web特性在各浏览器中的支持情况
* [`compat-table` -【重要】](https://kangax.github.io/compat-table/es6/) - 各浏览器支持ES6语法的对比 

* [`KeyboardEvent.key` - Web中键盘按键对应的值](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key/Key_Values)
* [KeyCode - 在浏览器上按任意键后显示对应的keyCode](http://keycode.info/)

* [Mozilla参考文档 - 前端大全](https://developer.mozilla.org/zh-CN/)
  * [Mozilla 对 ECMAScript 6 的支持](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/New_in_JavaScript/ECMAScript_6_support_in_Mozilla)
  
* [`strict mode` - 严格模式](http://www.ruanyifeng.com/blog/2013/01/javascript_strict_mode.html)

* [ECMA-262最新版 -【Draft】](https://tc39.github.io/ecma262/)
* [ECMA-262最新版 -【Finished】](http://www.ecma-international.org/ecma-262/)
* [ECMAScript® 2016 Language Specification - 【ECMAScript 7.0】](http://www.ecma-international.org/ecma-262/7.0/)
* [ECMAScript® 2015 Language Specification - 【ECMAScript 6.0】](http://www.ecma-international.org/ecma-262/6.0/)

* [ECMAScript 6 入门 - 阮一峰](http://es6.ruanyifeng.com/)
* [ES6 静态`import`和动态`import()`区别](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import)
* [ECMAScript 2015 特性 - Babel官网](https://babeljs.io/docs/en/learn)
    * `Arrows and Lexical This`
    * `Classes`
    * `Enhanced Object Literals`
    * `Template Strings`
    * `Destructuring`
    * `Default + Rest + Spread`
    * `Let + Const`
    * `Iterators + For..Of`
    * `Generators`
    * `Comprehensions`
    * `Unicode`
    * `Modules`
    * `Module Loaders`
    * `Map + Set + WeakMap + WeakSet`
    * `Proxies`
    * `Symbols`
    * `Subclassable Built-ins`
    * `Math + Number + String + Object APIs`
    * `Binary and Octal Literals`
    * `Promises`
    * `Reflect API`
    * `Tail Calls`

* [What is the difference between `properties` and `attributes` in HTML?](https://stackoverflow.com/questions/6003819/what-is-the-difference-between-properties-and-attributes-in-html#answer-6004028)

* [`debounce` / `throttle ` / `requestAnimationFrame (rAF)` 详解](https://css-tricks.com/debouncing-throttling-explained-examples/)
