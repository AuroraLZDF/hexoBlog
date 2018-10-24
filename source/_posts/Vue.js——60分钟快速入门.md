---
title: Vue.js——60分钟快速入门
date: 2017-04-11 15:33:11
tags: [Vue.js, 前端框架]
categories: [Vue.js]
toc: true
cover: '/images/categories/vuejs.jpeg'
---
<h1>Vue.js介绍</h1>

[Vue.js](https://github.com/vuejs/vue)是当下很火的一个JavaScript MVVM库，它是以数据驱动和组件化的思想构建的。相比于Angular.js，Vue.js提供了更加简洁、更易于理解的API，使得我们能够快速地上手并使用Vue.js。

如果你之前已经习惯了用jQuery操作DOM，学习Vue.js时请先抛开手动操作DOM的思维，因为Vue.js是数据驱动的，你无需手动操作DOM。它通过一些特殊的HTML语法，将DOM和数据绑定起来。一旦你创建了绑定，DOM将和数据保持同步，每当变更了数据，DOM也会相应地更新。

当然了，在使用Vue.js时，你也可以结合其他库一起使用，比如jQuery。

<h2>MVVM模式</h2>

下图不仅概括了MVVM模式（Model-View-ViewModel），还描述了在Vue.js中ViewModel是如何和View以及Model进行交互的。

<img src="/images/mvvm.png" />

**ViewModel是Vue.js的核心，它是一个Vue实例**。Vue实例是作用于某一个HTML元素上的，这个元素可以是HTML的body元素，也可以是指定了id的某个元素。

当创建了ViewModel后，双向绑定是如何达成的呢？

首先，我们将上图中的DOM Listeners和Data Bindings看作两个工具，它们是实现双向绑定的关键。
从View侧看，ViewModel中的DOM Listeners工具会帮我们监测页面上DOM元素的变化，如果有变化，则更改Model中的数据；
从Model侧看，当我们更新Model中的数据时，Data Bindings工具会帮我们更新页面中的DOM元素。

<h2>Hello World示例</h2>

了解一门语言，或者学习一门新技术，编写Hello World示例是我们的必经之路。
这段代码在画面上输出"Hello World!"。

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
	</head>

	<body>
		<!--这是我们的View-->
		<div id="app">
			{{ message }}
		</div>
	</body>
	<script src="js/vue.js"></script>
	<script>
		// 这是我们的Model
		var exampleData = {
			message: 'Hello World!'
		}

		// 创建一个 Vue 实例或 "ViewModel"
		// 它连接 View 与 Model
		new Vue({
			el: '#app',
			data: exampleData
		})
	</script>
</html>
```
使用Vue的过程就是定义MVVM各个组成部分的过程的过程。

1.定义View<br>
2.定义Model<br>
3.创建一个Vue实例或"ViewModel"，它用于连接View和Model<br>

在创建Vue实例时，需要传入一个**选项对象**，选项对象可以包含数据、挂载元素、方法、模生命周期钩子等等。

在这个示例中，选项对象的`el`属性指向View，`el: '#app'`表示该Vue实例将挂载到`<div id="app">...</div>`这个元素；data属性指向Model，`data: exampleData`表示我们的Model是exampleData对象。
Vue.js有多种数据绑定的语法，最基础的形式是文本插值，使用一对大括号语法，在运行时`{{ message }}`会被数据对象的message属性替换，所以页面上会输出"Hello World!"。

<p class="note">Vue.js已经更新到2.0版本了，但由于还不是正式版，<span style="color:red;">本文的代码都是1.0.25版本的</span>。</p>

<h3>双向绑定示例</h3>

MVVM模式本身是实现了双向绑定的，在Vue.js中可以使用`v-model`指令在表单元素上创建双向数据绑定。

```html
<!--这是我们的View-->
<div id="app">
	<p>{{ message }}</p>
	<input type="text" v-model="message"/>
</div>
```
将message绑定到文本框，当更改文本框的值时，`<p>{{ message }}</p>` 中的内容也会被更新。
<img src="/images/v-model.gif" />

反过来，如果改变message的值，文本框的值也会被更新，我们可以在Chrome控制台进行尝试。
<img src="/images/v-model-r.gif" />

<p class="note">Vue实例的data属性指向exampleData，它是一个引用类型，改变了exampleData对象的属性，同时也会影响Vue实例的data属性。</p>

<h1>Vue.js的常用指令</h1>

上面用到的`v-model`是Vue.js常用的一个指令，那么指令是什么呢？

<div class="bs-callout bs-callout-info">
    <strong>
        Vue.js的指令是以v-开头的，它们作用于HTML元素，指令提供了一些特殊的特性，将指令绑定在元素上时，指令会为绑定的目标元素添加一些特殊的行为，我们可以将指令看作特殊的HTML特性（attribute）。   
    </strong>
</div>

Vue.js提供了一些常用的内置指令，接下来我们将介绍以下几个内置指令：

<ul>
    <li>v-if指令</li>
    <li>v-show指令</li>
    <li>v-else指令</li>
    <li>v-for指令</li>
    <li>v-bind指令</li>
    <li>v-on指令</li>
</ul>

Vue.js具有良好的扩展性，我们也可以开发一些自定义的指令，后面的文章会介绍自定义指令。

<h2>v-if指令</h2>

`v-if`是条件渲染指令，它根据表达式的真假来删除和插入元素，它的基本语法如下：

    v-if="expression"

expression是一个返回bool值的表达式，表达式可以是一个bool属性，也可以是一个返回bool的运算式。例如：

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
	</head>
	<body>
		<div id="app">
			<h1>Hello, Vue.js!</h1>
			<h1 v-if="yes">Yes!</h1>
			<h1 v-if="no">No!</h1>
			<h1 v-if="age >= 25">Age: {{ age }}</h1>
			<h1 v-if="name.indexOf('jack') >= 0">Name: {{ name }}</h1>
		</div>
	</body>
	<script src="js/vue.js"></script>
	<script>
		
		var vm = new Vue({
			el: '#app',
			data: {
				yes: true,
				no: false,
				age: 28,
				name: 'keepfool'
			}
		})
	</script>
</html>
```

注意：yes, no, age, name这4个变量都来源于Vue实例选项对象的data属性。
<img src="/images/v-if.png" />

这段代码使用了4个表达式：

<ul>
    <li>数据的`yes`属性为true，所以"Yes!"会被输出；</li>
    <li>数据的`no`属性为false，所以"No!"不会被输出；</li>
    <li>运算式`age >= 25`返回true，所以"Age: 28"会被输出；</li>
    <li>运算式`name.indexOf('jack') >= 0`返回false，所以"Name: keepfool"不会被输出。</li>
</ul>

<p class="note"><strong>注意：</strong>v-if指令是根据条件表达式的值来执行<strong><span style="color: #ff0000">元素的插入或者删除行为。</span></strong></p>

这一点可以从渲染的HTML源代码看出来，面上只渲染了3个\<h1\>元素，v-if值为false的\<h1\>元素没有渲染到HTML。
<img src="/images/v-if-n.png" />

为了再次验证这一点，可以在Chrome控制台更改age属性，使得表达式`age >= 25`的值为false，可以看到`<h1>Age: 28</h1>`元素被删除了。
<img src="/images/v-if-show.gif" />

<p>age是定义在选项对象的data属性中的，为什么Vue实例可以直接访问它呢？<br>这是因为<strong>每个Vue实例都会代理其选项对象里的data属性。</strong></p>

<h2>v-show指令</h2>

`v-show`也是条件渲染指令，和v-if指令不同的是，使用`v-show`指令的元素始终会被渲染到HTML，它只是简单地为元素设置CSS的style属性。

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
	</head>
	<body>
		<div id="app">
			<h1>Hello, Vue.js!</h1>
			<h1 v-show="yes">Yes!</h1>
			<h1 v-show="no">No!</h1>
			<h1 v-show="age >= 25">Age: {{ age }}</h1>
			<h1 v-show="name.indexOf('jack') >= 0">Name: {{ name }}</h1>
		</div>
	</body>
	<script src="js/vue.js"></script>
	<script>
		
		var vm = new Vue({
			el: '#app',
			data: {
				yes: true,
				no: false,
				age: 28,
				name: 'keepfool'
			}
		})
	</script>
</html>
```

