---
layout: post
title: "【WEB】Chrome实用调试技巧"
date: 2017-12-07 
description: "Chrome实用调试技巧"
categories: WEB
--- 

  

## Chrome实用调试技巧

> **如今Chrome浏览器无疑是最受前端青睐的工具，原因除了界面简洁、大量的应用插件，良好的代码规范支持、强大的V8解释器之外，还因为Chrome开发者工具提供了大量的便捷功能，方便我们前端调试代码，我们在日常开发中是越来越离不开Chrome，是否熟练掌握Chrome调试技巧恐怕也会成为考量前端技术水平的标杆。**

介绍Chrome调试技巧的文章很多，本文结合我自己的开发经验，希望从实际运用的角度为大家再一次谈一谈这些功能，也希望对大家都有所帮助和启发。

### 常用快捷键

- **``ctrl+p``** 项目中定位文件，以下查找VueJS库文件：

![](http://7xslv0.com1.z0.glb.clouddn.com/chrome-debug/shotcut-1.gif)

- **``ctrl+shif+o``** 文件中定位成员函数，以下定位到VueJS的nextTick接口：
![](http://7xslv0.com1.z0.glb.clouddn.com/chrome-debug/shotcut-2.gif)

### Snippets 随时编写代码

- Chrome在souces页面提供snippets一栏，这里我们可以随时编写JS代码，运行结果会打印到控制台。代码是全局保存的，我们在任何页面，包括新建标签页，都可以查看或运行这些代码。

- 我们不再需要为了运行一小段JS代码而新建一个HTML页面。snippets的方便之处在于，你只需要打开chrome就可以编写一份任意页面都可以运行的JS代码，而且用过snippets都知道，snippets编辑器是可以和sublime text相媲美的。

 - 某次项目中，我需要将100多页的word文档导入到页面中。考虑后续样式编写，页面的HTML结构如下：

```html
<div class="help-page_row">
    <h3 class="help-page_title">title</h3>
    <p class="help-page_desc">paragraph</p>
    <p class="help-page_desc">paragraph</p>
</div>
```

** 手工将100多页的内容组合成上面的HTML结构太过耗费时间，不太现实，所以我决定使用JS来将文档内容的标题和段落解析出来，并进行HTML包装。**

** 由于不需要视图的支持，在snippets编写这段代码是最好的选择，经过几次调试修改，最终成果如下：**

![](http://7xslv0.com1.z0.glb.clouddn.com/chrome-debug/snippet-0.png)

** 最后，将Word文档内容复制到snippets中，执行解析函数，最终的解析出来的HTML结果打印到控制台：**

![](http://7xslv0.com1.z0.glb.clouddn.com/chrome-debug/snippet-2.png)

> **snippets中可以使用控制台的copy接口，解析结果直接拷贝到剪切板会更方便
使用snippets来完成这类轻量级工作时，不需要追求代码的可读性、可维护性，我们的代码只需要在大部分场景下能够正常运行就足够了。**

**但为了满足大部分场景，代码也是需要反复调试修改。snippets最实用之处恰恰在于，随时编写，随时调试，随时修改！**

### copy 格式化拷贝

>在项目开发中，我们可能需要将后台数据拷贝到本地，作为本地数据进行调试。
如果后台返回没有格式化的JSON数据，在本地调试中我们难免会遇到手动修改数据的情况，格式不美观的JSON数据修改起来会异常困难。

 - 说到JSON的格式化，我们首先想到的是JSON.stringify的格式化功能，例如四个空格的缩进：
```javascript
JSON.stringify({name: 'lxjwlt'}, null, 4);
```

![](http://7xslv0.com1.z0.glb.clouddn.com/chrome-debug/copy-2.png)

每次格式化JSON数据都要编写这段代码实在太麻烦，我们可以使用chrome控制台的copy接口解决这一问题：

- 请求项的右键菜单中选择Copy Response拷贝响应内容
- 命令行中使用copy接口处理数据
- 得到格式化的JSON数据
![](http://7xslv0.com1.z0.glb.clouddn.com/chrome-debug/copy-1.gif)

>不仅仅是对象，copy接口对任何数据都可以进行拷贝，这里利用的是copy在拷贝数组或对象过程中，对数据进行美化的功能

## iframe 调试

**如果我们使用Webpack服务器工具webpack-dev-server访问项目的开发页面，我们会发现，开发页面被内嵌到了iframe中进行渲染。**

**由于Chrome控制台默认的上下文是window.top，控制台中无法直接对内嵌在iframe的开发页面进行操作。如果要对iframe中的页面进行DOM操作或者执行类库API，我们要通过contentWindow来获取到iframe的上下文，然后使用with语句进行调试:**


```html
<iframe id="iframe"></iframe>
// 控制台
with (document.getElementById('iframe').contentWindow) {
    inspect(document.body);
    new Vue({ /* ... */ });
    // do something...
}
```

**以上方法可以在任意浏览器上使用，但如果我们使用的是Chrome浏览器，Chrome控制台的上下文切换功能会更加方便：**

![](http://7xslv0.com1.z0.glb.clouddn.com/chrome-debug/iframe-1.gif)

我们将上下文切换到iframe中，控制台的代码都会基于iframe的上下文来执行。如果你用webpack-dev-server进行调试，你会感谢这个功能。

## debug 毫无用处？

- Chrome控制台提供debug接口，可以传入一个函数，当这个函数下次执行的时候，调试器会自动在该函数中进行断点调试。

- 我们明明可以在代码中设置断点进行调试，为什么要用到debug来设置，是为了舍弃鼠标用命令行装逼而已吗？

- 在我看来，debug函数还提供了定位功能，它能够让我们很快的找到指定的函数。下面演示怎么调试VueJS的数据驱动，如何找到VueJS数据驱动的代码入口。

- 我们都知道，VueJS的数据驱动是通过defineProperty方法对数据的getter和setter进行封装，在这个封装中实现数据变化驱动视图同步修改的功能。如果我们想研究VueJS的数据驱动，那么首先要找到封装getter和setter的地方，我们可以通过debug接口来进行定位。以下用getter方法举例。

首先我们知道VueJS实例中的数据都是映射_data属性中的值：
```javascript
var vm =  new  Vue({
    data:  {
     name:  'lxjwlt'
   }
});
vm.name === vm._data.name;  // true`
```
所以我们要找的数据实际在VueJS实例的``_data``属性中。接下来我们通过``getOwnPropertyDescriptor``获取数据的getter函数：


```javascript
Object.getOwnPropertyDescriptor(vm._data, "name").get;
```
找到了getter函数，我们就可以使用debug接口对其进行断点调试：


```javascript
debug(Object.getOwnPropertyDescriptor(vm._data, "name").get)
```

这样，当我们获取vm.name数据时，自然会触发该数据的getter函数，从而触发断点调试，自动定位到了函数所在的地方：

![](http://7xslv0.com1.z0.glb.clouddn.com/chrome-debug/debug-1.gif)

日后要调试或者定位公共API，不妨试试Chrome的debug接口功能！

## 条件breakpoint

- 在Chrome中，我们可以给断点设置表达式，当表达式为true时断点调试才会生效，这就是条件断点。

- 有了条件断点，我们在调试代码的时候能够更加精确的控制代码断点的时机，特别是一段代码会被反复运行的时候，条件断点能够跳过大多数情况，只关注我们想要的情景。除了这一点外，条件断点调试还有另一个用法。

- 在断点调试中，我们往往会检查当前代码的执行状态，如果操作比较繁琐，那么我们可以使用条件断点添加自动化操作，帮助我们减少一部分工作量。

比如我们要在断点发生后查看DOM元素，那么断点条件可以这么写：


```javascript
// 当DOM元素满足某个条件进行断点，同时查看这个元素
elem.hasAttribute('class') && inspect(elem);
```
如果不清楚操作的返回值，我们可以强行让该操作返回true，从而不影响断点的条件判断：

```javascript
elem.hasAttribute('class') && (inspect(elem) || true);
```

或者分行写：

```javascript
if (elem.hasAttribute('class')) {inspect(elem); true;}
```

![](http://7xslv0.com1.z0.glb.clouddn.com/chrome-debug/breakpoint-1.gif)

再比如，在VueJS的调试中，我们往往需要知道VueJS实例的当前状态，所以每次触发断点调试时，我们可以先使用clear接口清除控制台历史输出，再将VueJS实例的当前状态打印出来：


```javascript
vm.sum > 4 && (clear() || vm.$log() || true);
```
![](http://7xslv0.com1.z0.glb.clouddn.com/chrome-debug/breakpoint-2.gif)

如果在条件断点中定义变量，变量是定义到全局作用域上，即window对象上的

## Async调试

- Chrome调试器的Async模式是为调试异步函数所设计一个功能。

- 在Promise被广泛应用的今天，我们都知道，Promise的回调是异步执行的，没有开启Async模式前，调动栈只记录到回调函数本身，我们无法找到代码执行的顺序，这给我们调试带来巨大的困难。Async模式可以解决这个问题：

![](http://7xslv0.com1.z0.glb.clouddn.com/chrome-debug/async.gif)

开启Async模式后，异步函数之前的调用栈都会被记录下来，而且调用栈中代码执行状态也得到了保留。

来源： https://www.zybuluo.com/lxjwlt/note/434612
