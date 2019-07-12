# Less

## 常用

### 单位转换

* `加法`、`减法`、`比较`时会转换成对应的单位值。如果不可转换，只保留左边出现的第一个单位，后面的单位忽略。

    ```less
    // numbers are converted into the same units
    @conversion-1: 5cm + 10mm; // result is 6cm
    @conversion-2: 2 - 3cm - 5mm; // result is -1.5cm
    
    // conversion is impossible
    @incompatible-units: 2 + 5px - 3cm; // result is 4px
    
    // example with variables
    @base: 5%;
    @filler: @base * 2; // result is 10%
    @other: @base + @filler; // result is 15%
    ```

* 乘法、除法不会去转换单位，因为数学里的带单位乘除后结果单位与原值是不相同的，所以没意思。

    ```less
    @base: 2cm * 3mm; // result is 6cm
    
    @color: #224488 / 2; //results in #112244
    background-color: #112244 + #111; // result is #223355
    ```

### Escaping

Escaping allows you to use any arbitrary string as property or variable value. 
Anything inside `~"anything"` or `~'anything'` is used as is with no changes except [interpolation](http://lesscss.org/features/#variables-feature-variable-interpolation).

```less
@min768: ~"(min-width: 768px)";
.element {
  @media @min768 {
    font-size: 1.2rem;
  }
}
```

results in:
```css
@media (min-width: 768px) {
  .element {
    font-size: 1.2rem;
  }
}
```

Note, as of Less 3.5, you can simply write:
```less
@min768: (min-width: 768px);
.element {
  @media @min768 {
    font-size: 1.2rem;
  }
}
```
In 3.5+, many cases previously requiring "quote-escaping" are not needed.

### [Lazy Loading](http://lesscss.org/features/#variables-feature-lazy-loading)

Variables do not have to be declared before being used.Like CSS custom properties, 
mixin and variable definitions do not have to be placed before a line where they are referenced.

Valid Less snippet:
```less
.lazy-eval {
  width: @var;
}

@var: @a;
@a: 9%;
```

this is valid Less too:
```less
.lazy-eval {
  width: @var;
  @a: 9%;
}

@var: @a;
@a: 100%;
```

both compile into:
```css
.lazy-eval {
  width: 9%;
}
```

### Comments

Both block-style and inline comments may be used:
```less
/* One heck of a block
 * style comment! */
@var: red;

// Get in line!
@var: white;
```

### [Mixins](http://lesscss.org/features/#mixins-feature)

"mix-in" properties from existing styles

You can mix-in class selectors and id selectors, e.g.

```less
.a, #b {
  color: red;
}
.mixin-class {
  .a();
}
.mixin-id {
  #b();
}
```

which results in:
```css
.a, #b {
  color: red;
}
.mixin-class {
  color: red;
}
.mixin-id {
  color: red;
}
```

> Currently and historically, the parentheses in a mixin call are optional, but optional parentheses are deprecated and will be required in a future release.

```less
.a(); 
.a;  // currently works, but deprecated; don't use
```

#### Not Outputting the Mixin

If you want to create a mixin but you do not want that mixin to be in your CSS output, put parentheses after the mixin definition.

```less
.my-mixin {
  color: black;
}
.my-other-mixin() {
  background: white;
}
.class {
  .my-mixin();
  .my-other-mixin();
}
```
outputs:
```css
.my-mixin {
  color: black;
}
.class {
  color: black;
  background: white;
}
```


## 文档

* [官网](http://lesscss.org/)
* [Less 的用法及配置项](http://lesscss.org/usage/)
* [Functions - 内置的函数](http://lesscss.org/functions/)
* [Features - In-Depth Guide](http://lesscss.org/features/)
* [Tools](http://lesscss.org/tools/)
    * [less2css.org](http://lesscss.org/less-preview/)
    * [estFiddle](http://ecomfe.github.io/est/fiddle/)
