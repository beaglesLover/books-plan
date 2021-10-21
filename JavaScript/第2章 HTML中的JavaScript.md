# 第2章 HTML中的JavaScript

####@Author: An Hsu

## 2.1 \<script>元素

**\<script>元素具有以下关键属性**：

* `src`: 可选。表示要执行的外部代码文件，可以代替直接在文档中嵌入的脚本。**指定了`src`属性的script元素内不应再嵌入代码**。

  外部JS的文件扩展名不是必须的，因为浏览器不会检查文件的扩展名，这为使用服务器端脚本语言动态生成JS代码或在浏览器中将JS扩展语言（TS，JSX）转变为JS提供了可能。服务器经常根据文件扩展名确定相应的正确MIME类型，如果不适用扩展名，则要保证服务器可以正确返回MIME类型。

  使用外部扩展JS代码时，标签内部的JS代码不会被解析。

* `type`:可选。表示代码块中脚本语言的内容类型（Multipurpose Internet Mail Extensions, MIME类型）

  其值只能包含以下几种：

  * ”text/javascript“ （默认）
  * ”text/ecmascript"
  * “application/javascript”
  * “application/ecmascript”
  * "module" (ES6)

  使用ES6模块时，这个值需要是"module"，代码会被当做JS模块处理，默认为defer方式：

  ```javascript
  <script type="module"> 
    import {firstName, lastName, year} from './profile';
  </script>
  ```

* `defer`: 可选。表示脚本可以延迟到文档被完全解析和显示之后再执行，相当于告诉浏览器立即下载脚本，但是延迟执行。只对外部脚本有效。如果包含多个具有defer属性的脚本，HTML5规范要求他们按照出现的先后依次执行，而这若干个脚本会先于事件`DOMContentLoaded`事件执行。但在现实情况下却不一样按照上述规范执行，**所以最好只对一个脚本设置`defer`属性。**

* `async` :可选。表示应立即开始下载，但不能阻止其他页面动作，只对外部脚本文件有效。与`defer`不同的是其不会按照先后顺序执行。指定这个属性的目的在于不让页面等待两个脚本下载和执行，从而一步加载页面其他内容。

  `async`和`defer`属性都会使scrpit标签异步加载，但是执行顺序不同，详见图：

  ![bVCBBR](https://cdn.jsdelivr.net/gh/xuan97916/image-hosting@master/20211021/bVCBBR.1caebxch9g00.jpeg)

  其中蓝色代表网络读取，红色代表执行时间，都是针对脚本的，绿色代表HTML解析

  1. 对于一个普通的script标签而言，其加载和解析是同步的，会阻塞DOM的渲染，这就是一般将script标签写在body底部的原因，防止加载脚本使得页面处于白屏状态，同时因为JS有可能需要DOM操作，所以需要在DOM全部渲染之后再执行操作。对于多个script元素，其按照在html文件中的先后顺序依次执行。
  2. 对于使用`defer`和`async`参数，其在网络加载阶段相较于HTML解析都是异步的。
  3. 使用`async`参数对于可以不依赖任何脚本或不被任何脚本依赖的脚本比较合适，如Google Analytics（？）

  在实际情况下，浏览器会对上述情况做出优化，对于chrome浏览器而言其WebKit的渲染过程如下：

  >1. 当用户输入URL时，WebKit调用资源加载器加载URL对应网页
  >
  >2. 加载器依赖网络模块建立连接，发送请求并接收答复
  >
  >3. WebKit接收到网页或资源的数据，可能是同步或者异步获取的
  >
  >4. 网页被交给HTML解释器转变为一系列的Token
  >
  >5. 解释器根据Token创建节点（Node），形成DOM树
  >
  >6. 如果节点是JS代码，调用JS引擎解释并执行
  >
  >7. JS代码可能会修改DOM树的结构
  >
  >8. 如果节点需要依赖其他资源（如图片、CSS等），调用资源加载器加载他们，但是他们是异步的，不会阻碍当前DOM树的继续创建，如果是JS资源URL（没有标记异步方式），则需要停止当前DOM树的创建，直到JS的资源加载并被JS引擎执行后才继续DOM树的创建
  >
  >   ​																													            	*—— 引用自《WebKit技术内幕》*

  `async`属性相当于单独创建一个进程独立加载和执行，而`defer`属性相当于将script放到boby底部的效果。

* `crossorigin`:可选。配置相关请求的CORS（跨源资源共享），默认不使用。如果src中跨站js文件出现了错误，我们无法通过监听window.onerror来将错误打印，所以需要添加这个属性来获取跨站文件的错误信息：

  ``````javascript
  // 在js中添加crossorigin
  <script>
        src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.js"
        crossorigin="anonymous">
  </script>
  ``````

  其值可能为：

  * `anonymous` 采用普通方式设置对此元素的CORS请求
  * `use-credentials` 采用凭证的方式设置对此元素的CORS请求

* `integrity`: 可选。提供一个Hash值，对比接收到的资源和指定的加密签名来验证资源的完整性，确保从CDN得到的资源不会被篡改。

  ``````javascript
  <script src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.js" 
     integrity="sha384-9u9lzb/hr8e14GLHe5TEOrTiH3Qtw5DX2Zw9X/g7cqj81W2McEMx5CKOszxdb8jg" crossorigin="anonymous"></script>
  
  //这个属性也可对style进行校验
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/animate.css/3.7.0/animate.min.css" 
    integrity="sha384-xyZLiqnBEFn1hDkS8VeG/YHoqOjS/ucimT8TI6GDr9+ZP1UNbZr6d/q0ldMi/xvL" crossorigin="anonymous">
  ``````

### 2.1.1 动态加载脚本

向DOM中动态添加`script`元素同样可以加载脚本：

```javascript
let script = document.createElement("script");  //创建一个script元素
script.scr = 'example.js';
document.head.appendChild(script); //将script元素添加到DOM中
```

在把HTMLElemet元素添加到DOM且执行到这段代码前不会发送请求。这种方式在默认情况下是使用异步方式加载的，相当于添加了`async`属性。

兼容性问题：部分浏览器不支持`async`属性，需要将其设置为同步加载方式，即设置`script.async = false`。但这种获取资源的方式对于浏览器预加载器不可见，影响在资源获取队列的优先级从而严重影响性能。需要在文档头部显式声明这些动态文件以便使得预加载器知道这些动态请求文件的存在：

``````javascript
<link rel="preload" href ='example.js'>
``````

### 2.1.2 XHTML中的变化

* 在XHTML (Extensible HyperText Markup Language. 扩展超文本标记语言) 中，使用JS必须指定`type="text/javascript"`

* 在XHTML中，JS语法中的小于号（“\<”）会被解释成一个标签的开始，且作为标签开始的小于号后面不能有空格。对于XHTML，一般使用以下解决方案：

  * 方案一：将所有的小于号替换为HTML对应的实体形式 `(&lt;)`

  * 方案二：把所有代码包含到一个CDATA块中。在XHTML中，CDATA块表示文档中可以包含任意文本的区块，并不会作为标签解析

    ```XHTML
    <script type="text/javascript"><![CDATA[
          function compare(a, b) {
            if (a < b) {
              console.log("A is less than B");
            } else if (a > b) {
              console.log("A is greater than B");
            } else {
              console.log("A is equal to B");
            }
    }
     ]]></script>
    ```

    当在浏览器不兼容XHTML的情况下，CDATA需要使用JS注释来抵消。

## 2.2 行内代码和外部文件

通常认为JS代码应该尽可能放在外部文件中，原因如下：

* **可维护性**：使用一个目录保存所有的代码，便于管理于维护，使得HTML与JS解耦。
* **缓存**：如果两个页面使用同一个JS文件，浏览器可以在其缓存中找到，页面加载速度提高

在支持SYDY/HTTP2的浏览器中，可以将JS组件拆分成多个轻量级独立组件发送给客户端，这样当客户端后续再次进行请求时，由于前次已经将部分组件缓存到浏览器中，后续加载会更快。

## 2.3 文档模式

在IE5.5中首次提出了`doctype`文档模式的概念，最开始文档模式具有两种：

* 混杂模式（quirks mode， 怪异模式，兼容模式）：让IE像IE5一样支持非标准特性，在W3C出台之前
* 标准模式（standards mode，严格模式）：让IE具有标准兼容行为

这两种模式的主要区别只体现在CSS渲染内容方面，但对JS也有副作用。随后其他浏览器也跟进了文档模式，并出现了

* 准标准模式（almost standards mode）：浏览器支持的标准特性更多，但是没有标准规定那么严格，主要区别在于如何对待图片元素周围的空白

**一般提到标准模式，指的是出混杂模式之外的模式**

### 2.3.1 不同文档模式的触发

1. 混杂模式：省略文档开头的`doctype`声明作为判断依据（所有浏览器），但是在混杂模式下不同浏览器差异很大

2. 标准模式：可以使用以下方式开启

   ```javascript
   // HTML 4.01 Strict: 
   <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
   "http://www.w3.org/TR/html4/strict.dtd">
   
   // XHTML 1.0 Strict:
   <!DOCTYPE html PUBLIC
   "-//W3C//DTD XHTML 1.0 Strict//EN"
   "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
   
   // HTML5:
   <!DOCTYPE html>
   ```

### 2.3.2 混杂模式和标准模式的区别

1. **scrollLeft 和 scrollTop：**

   * 标准模式：发生在`<html>`元素上
   * 混杂模式：可以通过`<body>`元素检测到属性变化

2. 盒模型

   * 标准模式：`width`指的是`content`部分的宽度

     <img src="https://cdn.jsdelivr.net/gh/xuan97916/image-hosting@master/20211021/bVbRzn2.20pkzt2hpvc0.png" alt="bVbRzn2" style="zoom:50%;" />

   * 混杂模式：`width`不含`margin`， 是`content+padding+border`三部分的宽度之和

     <img src="https://cdn.jsdelivr.net/gh/xuan97916/image-hosting@master/20211021/bVbRzRi.4zwr55pp23g.png" alt="bVbRzRi" style="zoom: 60%;" />

3. 尺寸单位
   * 标准模式：所有尺寸都必须包含单位，若不含单位则被忽略
   * 混杂模式：可以把 style.width设置为"20"，相当于"20px"

### 2.3.3 如何检测两种模式

使用`document.compatMode`检测当前浏览器处于的渲染模式：

- 标准模式下，值为"CSS1Compat"
- 混杂模式下，值为"BackCompat"

## 2.4 \<noscript>元素

此元素用于给不支持JS的浏览器提供替代内容，适用于以下场景：

* 浏览器不支持JS脚本
* 浏览器对脚本的支持被关闭

```javascript
//在文件中加入这个标签可以使得在不支持JS的浏览器显示标签中的话，如果浏览器支持则不会看到
<noscript>
<p>This page requires a JavaScript-enabled browser.</p> 
</noscript>
```

## 参考资料

1. 景初. (2016, Aug 31). 浅谈script标签的defer和async. Retrieved from https://segmentfault.com/a/1190000006778717

2. 寒水寺一禅. (2019, April 3). \<script> 属性详解. Retrieved from https://segmentfault.com/a/1190000018750934
3. Nightire. (2014, Aug 19). defer和async的区别. Retrieved from https://segmentfault.com/q/1010000000640869
4. 朱永盛. (2014), 'WebKit技术内幕', 电子工业出版社
5. 咸肉. (2021, Feb 24). 文档模式：标准模式、混杂模式.  Retrieved from https://juejin.cn/post/6932769062909018125

## 版权声明

本作品采用[知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议](http://creativecommons.org/licenses/by-nc-sa/4.0/)进行许可。

This work is licensed under a [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-nc-sa/4.0/).

![88x31-20211021201945538](https://cdn.jsdelivr.net/gh/xuan97916/image-hosting@master/20211021/88x31-20211021201945538.a4j2o8v1ioo.png)

