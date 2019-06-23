---
typora-root-url: C:\Users\刘楷川\Desktop\临时笔记\vue_image
---

# vue学习笔记

## 一、模板语法

### 1. 插值

#### 1.1 文本

数据绑定使用{{message}}（双大括号），无论何时，当message值发生变化时，插值处的内容也会发生变化，除非使用  **v-once**

```vue
<p>{{message}}</p>
<p v-once>
    {{message}} //这个将不会改变
</p>
```

#### 1.2 原始HTML

双大括号只会转换成普通文本，当需要转换 成html文本时需要使用 **v-html** 指令

```vue
<p>Using v-html directive: <span v-html="rawHtml"></span></p>
<!--你的站点上动态渲染的任意 HTML 可能会非常危险，因为它很容易导致 XSS 攻击。请只对可信内容使用 HTML 插值，绝不要对用户提供的内容使用插值。-->
```

#### 1.3 特性

Mustache 语法不能作用在 HTML 特性上，遇到这种情况应该使用 [v-bind 指令](https://cn.vuejs.org/v2/api/#v-bind)：

```vue
<div v-bind:id="dynamicId"></div>
```

#### 1.4 使用JavaScript表达式

vue完全支持JavaScript表达式

### 2. 指令

指令 (Directives) 是带有 `v-` 前缀的特殊特性。指令特性的值预期是**单个 JavaScript 表达式** (`v-for` 是例外情况，稍后我们再讨论)。指令的职责是，当表达式的值改变时，将其产生的连带影响，响应式地作用于 DOM。如：

```vue
<p style="font-size: 16px; text-align: center" v-if="false"><!--设置为false时，将移除<p>元素-->
      {{msg}}
    </p>
```

#### 2.1 参数

一些指令能够接收一个“参数”，在指令名称之后以冒号表示

```vue
<input type="button" v-on:click="commit" value="改变"><!-- v-on : click -->
<a v-bind:href="url">...</a><!-- v-bind : href -->
```

#### 2.2. 动态参数

```vue
<a v-bind:[attributeName]="url"> ... </a>
```

这里的 `attributeName` 会被作为一个 JavaScript 表达式进行动态求值，求得的值将会作为最终的参数来使用。例如，如果你的 Vue 实例有一个 `data` 属性 `attributeName`，其值为 `"href"`，那么这个绑定将等价于 `v-bind:href`。

#### 2.3 修饰符

修饰符 (modifier) 是以半角句号 `.` 指明的特殊后缀，用于指出一个指令应该以特殊方式绑定。例如，`.prevent` 修饰符告诉 `v-on` 指令对于触发的事件调用 `event.preventDefault()`：

```vue
<form v-on:submit.prevent="onSubmit">...</form>
```

### 3. 缩写

#### 3.1 v-bind 缩写

```vue
<!-- 完整语法 -->
<a v-bind:href="url">...</a>

<!-- 缩写 -->
<a :href="url">...</a>
<!--
当用v-bind 加载图片时，要在图片路径加上require：image: require('../assets/images/1.jpg')
或者将图片添加到static路径下
-->
```

#### 3.2 v-on 缩写

```vue
<!-- 完整语法 -->
<a v-on:click="doSomething">...</a>

<!-- 缩写 -->
<a @click="doSomething">...</a>
```



## 二、计算属性和侦听器

### 1. 计算属性

模板内的表达式非常便利，但是设计它们的初衷是用于简单运算的。在模板中放入太多的逻辑会让模板过重且难以维护。如：

```vue
<div id="example">
  {{ message.split('').reverse().join('') }}
</div>
```

所以，对于任何复杂逻辑，你都应当使用**计算属性**。



```vue
 <h4>{{reverse}}</h4>
computed:{
    reverse : function () {
      return this.msg.split('').reverse().join('');
    }
  }
```

![1556445370742](/../../../../../../images/1556445370742.png)

#### 1.2 计算缓存vs方法

我们可以将同一函数定义为一个方法而不是一个计算属性。两种方式的最终结果确实是完全相同的。然而，不同的是**计算属性是基于它们的响应式依赖进行缓存的**。只在相关响应式依赖发生改变时它们才会重新求值。相比之下，每当触发重新渲染时，调用方法将**总会**再次执行函数。



#### 1.3 计算属性vs侦听属性

Vue 提供了一种更通用的方式来观察和响应 Vue 实例上的数据变动：**侦听属性**。

当你有一些数据需要随着其它数据变动而变动时，你很容易滥用 `watch`——特别是如果你之前使用过 AngularJS。然而，通常更好的做法是使用计算属性而不是命令式的 `watch` 回调。

侦听属性:

```vue
<div id="demo">{{ fullName }}</div>
var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Foo',
    lastName: 'Bar',
    fullName: 'Foo Bar'
  },
  watch: {
    firstName: function (val) {
      this.fullName = val + ' ' + this.lastName
    },
    lastName: function (val) {
      this.fullName = this.firstName + ' ' + val
    }
  }
})
```

计算属性

```vue
var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Foo',
    lastName: 'Bar'
  },
  computed: {
    fullName: function () {
      return this.firstName + ' ' + this.lastName
    }
  }
})
```

#### 1.4 计算属性的setter

计算属性默认只有setter

```vue
// ...
computed: {
  fullName: {
    // getter
    get: function () {
      return this.firstName + ' ' + this.lastName
    },
    // setter
    set: function (newValue) {
      var names = newValue.split(' ')
      this.firstName = names[0]
      this.lastName = names[names.length - 1]
    }
  }
}
// ...
```

现在再运行 **vm.fullName = 'John Doe'**` 时，setter 会被调用，**vm.firstName** 和 **vm.lastName**也会相应地被更新。

### 2 侦听器

虽然计算属性在大多数情况下更合适，但有时也需要一个自定义的侦听器。这就是为什么 Vue 通过 `watch` 选项提供了一个更通用的方法，来响应数据的变化。当需要在数据变化时执行异步或开销较大的操作时，这个方式是最有用的。

```html
<div id="watch-example">
  <p>
    Ask a yes/no question:
    <input v-model="question">
  </p>
  <p>{{ answer }}</p>
</div>

<!-- 因为 AJAX 库和通用工具的生态已经相当丰富，Vue 核心代码没有重复 -->
<!-- 提供这些功能以保持精简。这也可以让你自由选择自己更熟悉的工具。 -->
<script src="https://cdn.jsdelivr.net/npm/axios@0.12.0/dist/axios.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/lodash@4.13.1/lodash.min.js"></script>
<script>
var watchExampleVM = new Vue({
  el: '#watch-example',
  data: {
    question: '',
    answer: 'I cannot give you an answer until you ask a question!'
  },
  watch: {
    // 如果 `question` 发生改变，这个函数就会运行
    question: function (newQuestion, oldQuestion) {
      this.answer = 'Waiting for you to stop typing...'
      this.debouncedGetAnswer()
    }
  },
  created: function () {
    // `_.debounce` 是一个通过 Lodash 限制操作频率的函数。
    // 在这个例子中，我们希望限制访问 yesno.wtf/api 的频率
    // AJAX 请求直到用户输入完毕才会发出。想要了解更多关于
    // `_.debounce` 函数 (及其近亲 `_.throttle`) 的知识，
    // 请参考：https://lodash.com/docs#debounce
    this.debouncedGetAnswer = _.debounce(this.getAnswer, 500)
  },
  methods: {
    getAnswer: function () {
      if (this.question.indexOf('?') === -1) {
        this.answer = 'Questions usually contain a question mark. ;-)'
        return
      }
      this.answer = 'Thinking...'
      var vm = this
      axios.get('https://yesno.wtf/api')
        .then(function (response) {
          vm.answer = _.capitalize(response.data.answer)
        })
        .catch(function (error) {
          vm.answer = 'Error! Could not reach the API. ' + error
        })
    }
  }
})
</script>
```



## 三、class 和 style 绑定

### 1. 绑定 HTML Class

#### 1.1 对象语法

我们可以传给 `v-bind:class` 一个对象，以动态地切换 class：

```vue
<div class="navigate" v-bind:class="{active: !isactive}">
</div>
data () {
      return {
        msg: '这是导航栏',
        message: '',
        fullmsg:'123',
        isactive: true
      }

<!--结果-->
<div data-v-2590f5be="" class="navigate active"></div>
```

#### 1.2 数组语法

我们可以把一个数组传给 `v-bind:class`，以应用一个 class 列表：

```vue
<div v-bind:class="[first, second]"></div>

export default {
    data () {
      return {
        msg: '这是导航栏',
        message: '',
        fullmsg:'123',
        isactive: true,
        first: 'navigate',
        second: 'active'
      }
    }
}
<!--结果渲染为-->
<div data-v-2590f5be="" class="navigate active"></div>
<!--也可以使用三元表达式或者嵌套-->
<div v-bind:class="[isActive ? activeClass : '', errorClass]"></div>
<div v-bind:class="[{ active: isActive }, errorClass]"></div>
```

#### 1.3 用在组件上

当在一个自定义组件上使用 `class` 属性时，这些类将被添加到该组件的根元素上面。这个元素上已经存在的类不会被覆盖。

例如，如果你声明了这个组件：

```vue
Vue.component('my-component', {
  template: '<p class="foo bar">Hi</p>'
})
```

然后在使用它的时候添加一些 class：

```vue
<my-component class="baz boo"></my-component>
```

HTML 将被渲染为:

```vue
<p class="foo bar baz boo">Hi</p>
```

对于带数据绑定 class 也同样适用：

```vue
<my-component v-bind:class="{ active: isActive }"></my-component>
```

当 `isActive` 为 truthy[[1\]](https://cn.vuejs.org/v2/guide/class-and-style.html#footnote-1) 时，HTML 将被渲染成为：

```vue
<p class="foo bar active">Hi</p>
```

### 2. 绑定内联样式

#### 2.1 对象语法

`v-bind:style` 的对象语法十分直观——看着非常像 CSS，但其实是一个 JavaScript 对象。CSS 属性名可以用驼峰式 (camelCase) 或短横线分隔 (kebab-case，记得用单引号括起来) 来命名：

```vue
<div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
data: {
  activeColor: 'red',
  fontSize: 30
}
<!--
直接绑定到内联样式上
-->
<div v-bind:style="styleObject"></div>
data: {
  styleObject: {
    color: 'red',
    fontSize: '13px'
  }
}
```

#### 2.2 数组语法

```vue
<div v-bind:style="[baseStyles, overridingStyles]"></div>
```

#### 2.3 多重值

```vue
<div :style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>
<!--这样写只会渲染数组中最后一个被浏览器支持的值。在本例中，如果浏览器支持不带浏览器前缀的 flexbox，那么就只会渲染 display: flex。-->
```



## 四、条件渲染

### 1.v-if

#### 1.1 在<template>元素上使用v-if条件渲染分组

因为 `v-if` 是一个指令，所以必须将它添加到一个元素上。但是如果想切换多个元素呢？此时可以把一个 `<template>` 元素当做不可见的包裹元素，并在上面使用 `v-if`。最终的渲染结果将不包含 `<template>` 元素。

```vue
<template v-if="ok">
  <h1>Title</h1>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</template>
```

#### 1.2 v-else

你可以使用 `v-else` 指令来表示 `v-if` 的“else 块”：

```vue
<div v-if="false">
    <p style="font-size: 16px; text-align: center">
      {{msg}}
    </p>
  </div>
  <div v-else>
    <p style="font-size: 16px; text-align: center">
      {{msg2}}
    </p>
  </div>
```

`v-else` 元素必须紧跟在带 `v-if` 或者 `v-else-if` 的元素的后面，否则它将不会被识别。



#### 1.3 v-else-if

`v-else-if`，顾名思义，充当 `v-if` 的“else-if 块”，可以连续使用



#### 1.4 用key管理可复用元素

Vue 会尽可能高效地渲染元素，通常会复用已有元素而不是从头开始渲染。这么做除了使 Vue 变得非常快之外，还有其它一些好处。

 Vue 为你提供了一种方式来表达“这两个元素是完全独立的，不要复用它们”。只需添加一个具有唯一值的 `key` 属性

```vue
<template v-if="loginType === 'username'">
  <label>Username</label>
  <input placeholder="Enter your username" key="username-input">
</template>
<template v-else>
  <label>Email</label>
  <input placeholder="Enter your email address" key="email-input">
</template>
```

### 2. v-show

另一个用于根据条件展示元素的选项是 `v-show` 指令。用法大致一样：

```vue
<p style="font-size: 16px; text-align: center" v-show="true">
      {{msg3}}
    </p>
```

### 3. v-if vs v-show

`v-if` 是“真正”的条件渲染，因为它会确保在切换过程中条件块内的事件监听器和子组件适当地被销毁和重建。

`v-if` 也是**惰性的**：如果在初始渲染时条件为假，则什么也不做——直到条件第一次变为真时，才会开始渲染条件块。

相比之下，`v-show` 就简单得多——不管初始条件是什么，元素总是会被渲染，并且只是简单地基于 CSS 进行切换。

一般来说，`v-if` 有更高的切换开销，而 `v-show` 有更高的初始渲染开销。因此，如果需要非常频繁地切换，则使用 `v-show` 较好；如果在运行时条件很少改变，则使用 `v-if` 较好。

## 五、列表渲染

### 1、用 v-for 把一个数组对应为一组元素

我们用 `v-for` 指令根据一组数组的选项列表进行渲染。`v-for` 指令需要使用 `item in items` 形式的特殊语法，`items` 是源数据数组并且 `item` 是数组元素迭代的别名。

```vue
<ul id="example-1">
  <li v-for="item in items">
    {{ item.message }}
  </li>
</ul>
var example1 = new Vue({
  el: '#example-1',
  data: {
    items: [
      { message: 'Foo' },
      { message: 'Bar' }
    ]
  }
})
```

在 `v-for` 块中，我们拥有对父作用域属性的完全访问权限。`v-for` 还支持一个可选的第二个参数为当前项的索引。

```vue
<ul id="example-2">
  <li v-for="(item, index) in items">
    {{ parentMessage }} - {{ index }} - {{ item.message }}
  </li>
</ul>
var example2 = new Vue({
  el: '#example-2',
  data: {
    parentMessage: 'Parent',
    items: [
      { message: 'Foo' },
      { message: 'Bar' }
    ]
  }
})
```

![1556682881550](/../../../../../../vue_image/1556682881550.png)

### 2. 一个对象的v-for

你也可以用 `v-for` 通过一个对象的属性来迭代。

```vue
<ul id="v-for-object" class="demo">
  <li v-for="value in object">
    {{ value }}
  </li>
</ul>
new Vue({
  el: '#v-for-object',
  data: {
    object: {
      title: 'How to do lists in Vue',
      author: 'Jane Doe',
      publishedAt: '2016-04-10'
    }
  }
})
```

![1556683059458](/../../../../../../vue_image/1556683059458.png)

你也可以提供第二个的参数为 property 名称 (也就是键名),第三个参数为索引：

```vue
<div v-for="(value, name, index) in object">
  {{ index }}. {{ name }}: {{ value }}
</div>
```

![1556683173298](/../../../../../../vue_image/1556683173298.png)

### 3. 一个组件的v-for

暂略。。。

## 六、事件处理

### 1. 监听事件

可以用 `v-on` 指令监听 DOM 事件，并在触发时运行一些 JavaScript 代码。

```vue
<button @click="counter += 1">加一</button>
    {{counter}}

export default {
    data () {
      return {
        msg: 'hello vue component',
        msg2: 'this is true',
        msg3: 'this is v-show',
        counter: 0
      }
    }
  }
```

### 2. 事件处理方法

 `v-on` 还可以接收一个需要调用的方法名称：

```vue
<button @click="greet">打招呼</button>
 greet: function (event) {
        alert("hello,vue")
        if (event) {
		<!--输出标签-->
          alert(event.target.tagName)
        }
      }
```

![1556688776181](/../../../../../../vue_image/1556688776181.png)

### 3. 内联处理器中的方法

可以在内联 JavaScript 语句中调用方法：

```vue
<div id="example-3">
  <button v-on:click="say('hi')">Say hi</button>
  <button v-on:click="say('what')">Say what</button>
</div>
new Vue({
  el: '#example-3',
  methods: {
    say: function (message) {
      alert(message)
    }
  }
})
```

### 4. 事件修饰符

修饰符是由点开头的指令后缀来表示的。

- `.stop`
- `.prevent`
- `.capture`
- `.self`
- `.once`
- `.passive`

```vue
<!-- 阻止单击事件继续传播 -->
<a v-on:click.stop="doThis"></a>

<!-- 提交事件不再重载页面 -->
<form v-on:submit.prevent="onSubmit"></form>

<!-- 修饰符可以串联 -->
<a v-on:click.stop.prevent="doThat"></a>

<!-- 只有修饰符 -->
<form v-on:submit.prevent></form>

<!-- 添加事件监听器时使用事件捕获模式 -->
<!-- 即元素自身触发的事件先在此处理，然后才交由内部元素进行处理 -->
<div v-on:click.capture="doThis">...</div>

<!-- 只当在 event.target 是当前元素自身时触发处理函数 -->
<!-- 即事件不是从内部元素触发的 -->
<div v-on:click.self="doThat">...</div>

<!-- 点击事件将只会触发一次——2.1.4新增 -->
<a v-on:click.once="doThis"></a>

<!--Vue 还对应 addEventListener 中的 passive 选项提供了 .passive 修饰符。-->
<!-- 滚动事件的默认行为 (即滚动行为) 将会立即触发 -->
<!-- 而不会等待 `onScroll` 完成  -->
<!-- 这其中包含 `event.preventDefault()` 的情况 -->
<div v-on:scroll.passive="onScroll">...</div>
```

使用修饰符时，顺序很重要；相应的代码会以同样的顺序产生。因此，用 `v-on:click.prevent.self` 会阻止**所有的点击**，而 `v-on:click.self.prevent` 只会阻止对元素自身的点击。



不要把 `.passive` 和 `.prevent` 一起使用，因为 `.prevent` 将会被忽略，同时浏览器可能会向你展示一个警告。请记住，`.passive` 会告诉浏览器你*不*想阻止事件的默认行为。

### 5. 按键修饰符

Vue 允许为 `v-on` 在监听键盘事件时添加按键修饰符：

```vue
<!-- 只有在 `key` 是 `Enter` 时调用 `vm.submit()` -->
<input v-on:keyup.enter="submit">
```

你可以直接将 [`KeyboardEvent.key`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key/Key_Values) 暴露的任意有效按键名转换为 kebab-case 来作为修饰符。

```vue
<input v-on:keyup.page-down="onPageDown">
```

在上述示例中，处理函数只会在 `$event.key` 等于 `PageDown` 时被调用。

## 七、 表单输入绑定

### 1. 基础用法

你可以用 `v-model` 指令在表单 `<input>`、`<textarea>` 及 `<select>` 元素上创建双向数据绑定。它会根据控件类型自动选取正确的方法来更新元素。尽管有些神奇，但 `v-model` 本质上不过是语法糖。它负责监听用户的输入事件以更新数据，并对一些极端场景进行一些特殊处理。

`v-model 会忽略所有表单元素的 value、checked、selected 特性的初始值而总是将 Vue 实例的数据作为数据来源。你应该通过 JavaScript 在组件的 data 选项中声明初始值。`

`v-model` 在内部使用不同的属性为不同的输入元素并抛出不同的事件：

- `text 和 textarea 元素使用 value 属性和 input 事件；`
- `checkbox 和 radio 使用 checked 属性和 change 事件；`
- `select 字段将 value 作为 prop 并将 change 作为事件。`

#### 1.1 文本

```vue
 <input type="text" v-model="message" placeholder="输入文字">
```

#### 1.2 多行文本

```vue
<textarea v-model="longMessage" placeholder="输入长文本"></textarea>
```

#### 1.3 复选框

```vue
<input type="checkbox" v-model="checkBox" value="1">
    <label for="1">1</label>
    <input type="checkbox" v-model="checkBox" value="2">
    <label for="2">2</label>
    <input type="checkbox" v-model="checkBox" value="3">
    <label for="3">3</label>
    <p></p>
    <span>{{checkBox}}</span>

 data () {
      return {
<!--如果不加[] 将会变成Boolean型-->
        checkBox:[],
}
}
```

#### 1.4 单选框

```vue
 <p>
      <input type="radio" v-model="radio" value="One">
      <label>One</label>
      <input type="radio" v-model="radio" value="Two">
      <label>Two</label>
    </p>
    <h3>{{radio}}</h3>
```



![1556697410298](/../../../../../../vue_image/1556697410298.png)

#### 1.5 选择框

```vue
<select v-model="select">
        <option v-for="item in items" v-bind:value="item.value">
          {{item.text}}
        </option>
      </select>
      <span>{{select}}</span>


data:{
select:'',<!--select 获取value里面的值-->
        items:[
          {text: "One", value: 1},
          {text: "Two", value: 2},
          {text: "Three", value: 3}
        ],
}
```

### 2. 值绑定

### 3. 修饰符

#### 3.1 .lazy

在默认情况下，`v-model` 在每次 `input` 事件触发后将输入框的值与数据进行同步 (除了[上述](https://cn.vuejs.org/v2/guide/forms.html#vmodel-ime-tip)输入法组合文字时)。你可以添加 `lazy` 修饰符，从而转变为使用 `change`事件进行同步：

```vue
<!-- 在“change”时而非“input”时更新 -->
<input v-model.lazy="msg" >
```

#### 3.2 .number

如果想自动将用户的输入值转为数值类型，可以给 `v-model` 添加 `number` 修饰符：

```vue
<input v-model.number="age" type="number">
```

#### 3.3 .trim

如果要自动过滤用户输入的首尾空白字符，可以给 `v-model` 添加 `trim` 修饰符：

```vue
<input v-model.trim="msg">
```

## 八、 组件基础

### 1. 基本示例

```vue
// 定义一个名为 button-counter 的新组件
Vue.component('button-counter', {
  data: function () {
    return {
      count: 0
    }
  },
  template: '<button v-on:click="count++">You clicked me {{ count }} times.</button>'
})
```

```html
<div id="components-demo">
  <button-counter></button-counter>
</div>
```

```vue
new Vue({ el: '#components-demo' })
```

因为组件是可复用的 Vue 实例，所以它们与 `new Vue` 接收相同的选项，例如 `data`、`computed`、`watch`、`methods` 以及生命周期钩子等。仅有的例外是像 `el`这样根实例特有的选项。

### 2. 组件的复用

你可以将组件进行任意次数的复用：

```vue
<div id="components-demo">
  <button-counter></button-counter>
  <button-counter></button-counter>
  <button-counter></button-counter>
</div>
```

注意当点击按钮时，每个组件都会各自独立维护它的 `count`。因为你每用一次组件，就会有一个它的新**实例**被创建。

#### 2.1 data 必须是一个函数

**一个组件的 data 选项必须是一个函数**，因此每个实例可以维护一份被返回对象的独立的拷贝：

```vue
data: function () {
  return {
    count: 0
  }
}
```

### 3. 组件的组织

通常一个应用会以一棵嵌套的组件树的形式来组织：

![Component Tree](/../../../../../../vue_image/components.png)

例如，你可能会有页头、侧边栏、内容区等组件，每个组件又包含了其它的像导航链接、博文之类的组件。

为了能在模板中使用，这些组件必须先注册以便 Vue 能够识别。这里有两种组件的注册类型：**全局注册**和**局部注册**。至此，我们的组件都只是通过 `Vue.component` 全局注册的：

```vue
Vue.component('my-component-name', {
  // ... options ...
})
```

### 4. 通过 Prop 向子组件传递数据

Prop 是你可以在组件上注册的一些自定义特性。当一个值传递给一个 prop 特性的时候，它就变成了那个组件实例的一个属性。为了给博文组件传递一个标题，我们可以用一个 `props` 选项将其包含在该组件可接受的 prop 列表中：

```vue
<HelloWorld title="post the title" />

<p style="font-size: 16px; text-align: center" v-show="true">
      {{title}}
    </p>

 props:['title'],
```

```vue
 <HelloWorld title="post the title" msg3="hello world" />

 <p style="font-size: 16px; text-align: center" v-show="true">
      {{msg3}}
    </p>

export default {
    props:[
      'title',
      'msg3'
    ],
    data () {
      return {
        msg: 'hello vue component',
        msg2: 'this is true',
        msg3: 'this is v-show',
        counter: 0
      }
    },
```

![1556709959159](/../../../../../../vue_image/1556709959159.png)

### 5. 单个根元素

```vue
<blog-post
  v-for="post in posts"
  v-bind:key="post.id"
  v-bind:post="post"
></blog-post>

Vue.component('blog-post', {
  props: ['post'],
  template: `
    <div class="blog-post">
      <h3>{{ post.title }}</h3>
      <div v-html="post.content"></div>
    </div>
  `
})
```

### 6. 监听子组件事件

## 九、 组件注册

### 1. 组件名

```vue
<!--全局注册  | 组件名  -->
Vue.component('my-component-name', { /* ... */ })
```

推荐遵循 [W3C 规范](https://html.spec.whatwg.org/multipage/custom-elements.html#valid-custom-element-name)中的自定义组件名 (字母全小写且必须包含一个连字符)。这会帮助你避免和当前以及未来的 HTML 元素相冲突。

### 2. 全局注册

```vue
Vue.component('my-component-name', {
  // ... 选项 ...
})
```

### 3. 局部注册

```vue
<!--通过一个普通的 JavaScript 对象来定义组件：-->
var ComponentA = { /* ... */ }
var ComponentB = { /* ... */ }
var ComponentC = { /* ... */ }

<!--然后在 components 选项中定义你想要使用的组件：-->
new Vue({
  el: '#app',
  components: {
    'component-a': ComponentA,
    'component-b': ComponentB
  }
})
```



```vue
<!--模块系统：-->
import ComponentA from './ComponentA.vue'

export default {
  components: {
    ComponentA
  },
  // ...
}
<!--注意在 ES2015+ 中，在对象中放一个类似 ComponentA 的变量名其实是 ComponentA: ComponentA 的缩写，即这个变量名同时是：

- 用在模板中的自定义元素的名称
- 包含了这个组件选项的变量名
-->
```

### 4. 模块系统

#### 4.1 在模块系统中局部注册

```vue
import ComponentA from './ComponentA'
import ComponentC from './ComponentC'

export default {
  components: {
    ComponentA,
    ComponentC
  },
  // ...

```

#### 

#### 4.2 基础组件的自动化全局注册

使用 `require.context` 只全局注册这些非常通用的基础组件,**全局注册的行为必须在根 Vue 实例 (通过 new Vue) 创建之前发生**。

```javascript
import Vue from 'vue'
import upperFirst from 'lodash/upperFirst'
import camelCase from 'lodash/camelCase'

const requireComponent = require.context(
  // 其组件目录的相对路径
  './components',
  // 是否查询其子目录
  false,
  // 匹配基础组件文件名的正则表达式
  /Base[A-Z]\w+\.(vue|js)$/
)

requireComponent.keys().forEach(fileName => {
  // 获取组件配置
  const componentConfig = requireComponent(fileName)

  // 获取组件的 PascalCase 命名
  const componentName = upperFirst(
    camelCase(
      // 获取和目录深度无关的文件名
      fileName
        .split('/')
        .pop()
        .replace(/\.\w+$/, '')
    )
  )

  // 全局注册组件
  Vue.component(
    componentName,
    // 如果这个组件选项是通过 `export default` 导出的，
    // 那么就会优先使用 `.default`，
    // 否则回退到使用模块的根。
    componentConfig.default || componentConfig
  )
})
```

## 十、 prop

### 1. prop 类型

```javascript
props: {
  title: String,
  likes: Number,
  isPublished: Boolean,
  commentIds: Array,
  author: Object,
  callback: Function,
  contactsPromise: Promise // or any other constructor
}
```

### 2. 传递静态或动态prop

```html
<!--传递静态-->
<blog-post title="My journey with Vue"></blog-post>

<!-- 动态赋予一个变量的值 -->
<blog-post v-bind:title="post.title"></blog-post>

<!-- 动态赋予一个复杂表达式的值 -->
<blog-post
  v-bind:title="post.title + ' by ' + post.author.name"
></blog-post>
```

#### 2.1 传入一个数字

```vue
<!-- 即便 `42` 是静态的，我们仍然需要 `v-bind` 来告诉 Vue -->
<!-- 这是一个 JavaScript 表达式而不是一个字符串。-->
<blog-post v-bind:likes="42"></blog-post>

<!-- 用一个变量进行动态赋值。-->
<blog-post v-bind:likes="post.likes"></blog-post>
```

#### 2.2 传入一个布尔值

```vue
<!-- 包含该 prop 没有值的情况在内，都意味着 `true`。-->
<blog-post is-published></blog-post>

<!-- 即便 `false` 是静态的，我们仍然需要 `v-bind` 来告诉 Vue -->
<!-- 这是一个 JavaScript 表达式而不是一个字符串。-->
<blog-post v-bind:is-published="false"></blog-post>

<!-- 用一个变量进行动态赋值。-->
<blog-post v-bind:is-published="post.isPublished"></blog-post>
```

#### 2.3 传入一个数组

```vue
<!-- 即便数组是静态的，我们仍然需要 `v-bind` 来告诉 Vue -->
<!-- 这是一个 JavaScript 表达式而不是一个字符串。-->
<blog-post v-bind:comment-ids="[234, 266, 273]"></blog-post>

<!-- 用一个变量进行动态赋值。-->
<blog-post v-bind:comment-ids="post.commentIds"></blog-post>
```

#### 2.4 传入一个对象

```vue
<!-- 即便对象是静态的，我们仍然需要 `v-bind` 来告诉 Vue -->
<!-- 这是一个 JavaScript 表达式而不是一个字符串。-->
<blog-post
  v-bind:author="{
    name: 'Veronica',
    company: 'Veridian Dynamics'
  }"
></blog-post>

<!-- 用一个变量进行动态赋值。-->
<blog-post v-bind:author="post.author"></blog-post>
```

#### 2.5  传入一个对象的所有属性值

```vue
<blog-post v-bind="post"></blog-post>

post: {
  id: 1,
  title: 'My Journey with Vue'
}
```

### 3. 单向数据流

所有的 prop 都使得其父子 prop 之间形成了一个**单向下行绑定**：父级 prop 的更新会向下流动到子组件中，但是反过来则不行。这样会防止从子组件意外改变父级组件的状态，从而导致你的应用的数据流向难以理解。

额外的，每次父级组件发生更新时，子组件中所有的 prop 都将会刷新为最新的值。这意味着你**不**应该在一个子组件内部改变 prop。如果你这样做了，Vue 会在浏览器的控制台中发出警告。

注意在 JavaScript 中对象和数组是通过引用传入的，所以对于一个数组或对象类型的 prop 来说，在子组件中改变这个对象或数组本身**将会**影响到父组件的状态。

两种常见的试图改变一个 prop 的情形：

**1.这个 prop 用来传递一个初始值；这个子组件接下来希望将其作为一个本地的 prop 数据来使用。**在这种情况下，最好定义一个本地的 data 属性并将这个 prop 用作其初始值：

```vue
props: ['initialCounter'], <!--接收参数值-->
data: function () {
  return {
    counter: this.initialCounter
  }
}
```

**2. 这个 prop 以一种原始的值传入且需要进行转换。**在这种情况下，最好使用这个 prop 的值来定义一个计算属性：

```vue
props: ['size'],
computed: {
  normalizedSize: function () {
    return this.size.trim().toLowerCase()
  }
}
```

### 4. prop 验证

```javascript
Vue.component('my-component', {
  props: {
    // 基础的类型检查 (`null` 和 `undefined` 会通过任何类型验证)
    propA: Number,
    // 多个可能的类型
    propB: [String, Number],
    // 必填的字符串
    propC: {
      type: String,
      required: true
    },
    // 带有默认值的数字
    propD: {
      type: Number,
      default: 100
    },
    // 带有默认值的对象
    propE: {
      type: Object,
      // 对象或数组默认值必须从一个工厂函数获取
      default: function () {
        return { message: 'hello' }
      }
    },
    // 自定义验证函数
    propF: {
      validator: function (value) {
        // 这个值必须匹配下列字符串中的一个
        return ['success', 'warning', 'danger'].indexOf(value) !== -1
      }
    }
  }
})
```

## 十一、 自定义事件

### 1. 事件名

建议采用**kebab-case 的事件名**

### 2. 自定义组件的v-model

## 十二、 插槽

### 1. 插槽内容

```vue
<navigation-link url="/profile">
    <!--插槽内可以包含任何模板代码，包括 HTML,甚至其他组件-->
  Your Profile
</navigation-link>
<!--在 <navigation-link> 的模板中可能会写为：-->
<a
  v-bind:href="url"
  class="nav-link"
>
    <!-- <slot></slot> 会被替换为 Your Profile-->
  <slot></slot>
</a>
```

如果 `<navigation-link>` **没有**包含一个 `<slot>` 元素，则该组件起始标签和结束标签之间的任何内容都会被抛弃。

### 2. 编译作用域

```vue
<navigation-link url="/profile">
  Logged in as {{ user.name }}
</navigation-link>
```



该插槽跟模板的其它地方一样可以访问相同的实例属性 (也就是相同的“作用域”)，而**不能**访问 `<navigation-link>` 的作用域

```vue
<navigation-link url="/profile">
  Clicking here will send you to: {{ url }}
  <!--
  这里的 `url` 会是 undefined，因为 "/profile" 是
  _传递给_ <navigation-link> 的而不是
  在 <navigation-link> 组件*内部*定义的。
  -->
</navigation-link>
```

规则：**父级模板里的所有内容都是在父级作用域中编译的；子模板里的所有内容都是在子作用域中编译的。**

### 3. 后备内容

有时为一个插槽设置具体的后备 (也就是默认的) 内容是很有用的，它只会在没有提供内容的时候被渲染。

```javascript
<button type="submit">
  <slot>Submit</slot>
</button>
//当在一个父级组件中使用 <submit-button> 并且不提供任何插槽内容时：
<submit-button></submit-button>
//后备内容“Submit”将会被渲染：
<button type="submit">
  Submit
</button>
//如果我们提供内容
<submit-button>
  Save
</submit-button>

//则这个提供的内容将会被渲染从而取代后备内容：
<button type="submit">
  Save
</button>
```

### 4. 具名插槽

```vue
<template>
<div class="container">
  <header>
      <!--<slot> 元素有一个特殊的特性：name。这个特性可以用来定义额外的插槽：-->
    <slot name="header"></slot>
  </header>
  <main>
      <!--一个不带 name 的 <slot> 出口会带有隐含的名字“default”。-->
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
</template>

<script>
  export default {
    name: 'Insear'
  }
</script>

<style scoped>
.container{
  margin: 100px auto;
  color: green;
  font-size: 26px;

}
</style>

```

```vue
<insear>
    <!--在向具名插槽提供内容的时候，我们可以在一个 <template> 元素上使用 v-slot 指令，并以 v-slot 的参数的形式提供其名称：-->
      <template v-slot:header>
        this is header
      </template>
    <!--任何没有被包裹在带有 v-slot 的 <template> 中的内容都会被视为默认插槽的内容。
		也可以这样写：
	<template v-slot:default>
    	<p>A paragraph for the main content.</p>
    	<p>And another one.</p>
  	</template>		
-->
      <p>A paragraph for the main content.</p>
      <p>And another one.</p>
      <template v-slot:footer>
        <p>
          this is footer
        </p>
      </template>
    </insear>


 import Insear from './components/Insear'
  export default {
    components: {
      HelloWorld,
      Navigate,
      Blog,
      Insear
    },
```

![1556848839393](/../../../../../../vue_image/1556848839393.png)

注意 **v-slot 只能添加在一个 <template> 上** (只有[一种例外情况](https://cn.vuejs.org/v2/guide/components-slots.html#独占默认插槽的缩写语法))，这一点和已经废弃的 [`slot` 特性](https://cn.vuejs.org/v2/guide/components-slots.html#废弃了的语法)不同。

### 5. 作用域插槽

为了让 `user` 在父级的插槽内容可用，我们可以将 `user` 作为一个 `<slot>` 元素的特性绑定上去：

```vue
<span>
  <slot v-bind:user="user">
    {{ user.lastName }}
  </slot>
</span>
```

绑定在 `<slot>` 元素上的特性被称为**插槽 prop**。现在在父级作用域中，我们可以给 `v-slot` 带一个值来定义我们提供的插槽 prop 的名字

```vue
<current-user>
  <template v-slot:default="slotProps">
    {{ slotProps.user.firstName }}
  </template>
</current-user>
```

#### 5.1 独占默认插槽的缩写语法

```vue
<current-user v-slot="slotProps">
  {{ slotProps.user.firstName }}
</current-user>
```

注意默认插槽的缩写语法**不能**和具名插槽混用，因为它会导致作用域不明确：

```vue
<!-- 无效，会导致警告 -->
<current-user v-slot="slotProps">
  {{ slotProps.user.firstName }}
  <template v-slot:other="otherSlotProps">
    slotProps is NOT available here
  </template>
</current-user>
```

### 6. 具名插槽的缩写

跟 `v-on` 和 `v-bind` 一样，`v-slot` 也有缩写，即把参数之前的所有内容 (`v-slot:`) 替换为字符 `#`。例如 `v-slot:header` 可以被重写为 `#header`

然而，和其它指令一样，该缩写只在其有参数的时候才可用。

## 十三、 cmd创建vue项目

环境要求：  安装有 **Node.js**、 **vue**、 **vue-cli** 。

创建项目：

**vue init webpack projectName**

进入项目，下载依赖：

**npm install 或者 cnpm install**

运行项目：

**npm run dev**

## 十四、 路由

### 1. cmd安装

```bash
npm install vue-router
```

### 2. 引入

```javascript
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)
```

### 3. 使用

```javascript
import Vue from 'vue'
import Router from 'vue-router'
// 引入相关组件
import Admin from '../components/Admin'
import Home from '../components/Home'
import Login from '../components/Login'
import Register from '../components/Register'
import Menu from '../components/Menu'
import About from '../components/about/About'

Vue.use(Router)

const routes = [
  {
    // 跳转路径
    path: '/admin',
    // 命名，可为空,可用为动态跳转
    name: 'Admin',
    // 引入组件
    component: Admin
  },
  {
    path: '/',
    name: 'Home',
    component: Home
  },
  {
    path: '/login',
    name: 'Login',
    component: Login
  },
  {
    path: '/register',
    name: 'Register',
    component: Register
  },
  {
    path: '/menu',
    name: 'Menu',
    component: Menu
  },
  {
    path: '/about',
    name: 'About',
    component: About
  },
  {
    // 如果用户输入不存在的路径将重定向到‘/’
    path: '*',
    name: '404',
    redirect: '/'
  }
]

export default new Router({
  routes,
  // 去除“#” 号
  mode: 'history'
})

```

```vue
<template>
  <div>
    <app-header/>
    <div class="container">
        <!--路由视图-->
      <router-view></router-view>
    </div>
  </div>
</template>

<script>
import Header from './components/Header'
export default {
  name: 'App',
  components: {
    'app-header': Header
  }
}
</script>

<style>
</style>

```

```vue

<router-link class="nav-link" to="menu">菜单</router-link>
```

### 4. 相关API

```javascript
getToNextPage: function (evet) {
      // 跳转到上一页
      this.$router.go(-1)
      // 跳转到指定页面
      // 1. this.$router.push({name: 'Admin'}) 通过名字跳转
      // 2. this.$router.push('/') 通过路径跳转（较常用）
      // 3. this.$router.replace('/')  利用replace通过路径跳转
      // 4. this.$router.replace({name: 'Admin'}) 利用replace通过名字跳转
    }
```

### 5. 二级菜单

```vue
<template>
<div class="container">
  <div class="row">
    <div class="col-md-4">
      <ul class="list-group">
        <router-link tag="li" class="list-group-item list-group-item-action" to="/about/history">历史订单</router-link>
        <router-link tag="li" class="list-group-item list-group-item-action" to="/about/personal">个人信息</router-link>
        <router-link tag="li" class="list-group-item list-group-item-action" to="/about/contact">联系我们</router-link>
      </ul>
    </div>
    <div class="col-md-8">
      <router-view></router-view>
    </div>
  </div>
</div>
</template>

<script>

export default {
  name: 'About',
  methods: {
    getToNextPage: function (evet) {
      // 跳转到上一页
      this.$router.go(-1)
     
    }
  }
}
</script>

<style scoped>

</style>

```

```javascript
{
    path: '/about',
    name: 'About',
    component: About,
    redirect: '/about/history',
    // 使用二级菜单
    children: [
      {path: '/about/history', name: 'history', component: History},
      {path: '/about/personal', name: 'personal', component: Personal},
      {path: '/about/contact', name: 'contact', component: Contact}
    ]
  },
```

### 6. 路由守卫

#### 6.1 全局守卫

##### 效果图：

![1556986655194](/../../../../../../vue_image/1556986655194.png)

```javascript
// to：将要去的页面， from：目前页面， next回调方法
router.beforeEach((to, from, next) => {
  console.log(from)
    // 放行页面‘/login’ ‘/register’ ‘/’
  if (to.path === '/login' || to.path === '/register' || to.path === '/') {
    // 放行
      next()
  } else {
    alert('请先登陆！！')
      //跳转到登陆页面
    next('/login')
  }
})
```

> - **to: Route**: the target [Route Object](https://router.vuejs.org/api/#the-route-object) being navigated to.
> - **from: Route**: the current route being navigated away from.
> - **next: Function**: this function must be called to **resolve** the hook. The action depends on the arguments provided to `next`:
>   - **next()**: move on to the next hook in the pipeline. If no hooks are left, the navigation is **confirmed**.
>   - **next(false)**: abort the current navigation. If the browser URL was changed (either manually by the user or via back button), it will be reset to that of the `from` route.
>   - **next('/') or next({ path: '/' })**: redirect to a different location. The current navigation will be aborted and a new one will be started. You can pass any location object to `next`, which allows you to specify options like `replace: true`, `name: 'home'` and any option used in [`router-link`'s `to`prop](https://router.vuejs.org/api/#to) or [`router.push`](https://router.vuejs.org/api/#router-push)
>   - **next(error)**: (2.4.0+) if the argument passed to `next` is an instance of `Error`, the navigation will be aborted and the error will be passed to callbacks registered via [`router.onError()`](https://router.vuejs.org/api/#router-onerror).
>
> **Make sure to always call the next function, otherwise the hook will never be resolved.**

![1556986822295](/../../../../../../vue_image/1556986822295.png)

在main.js 里面写守护函数，如果直接在router/index.js 里面写，会报错 : 

__WEBPACK_IMPORTED_MODULE_1_vue_router__.a.beforeEach is not a function

**路由守卫局限性：如果从地址栏直接输入地址，并不能拦截**

### 7. 路由复用

```javascript
<template>
  <div>
    <app-header/>
    <div class="container">
      <router-view></router-view>
      <div class="row">
        <div class="col-sm-4">
          <router-view name="history"></router-view>
        </div>
        <div class="col-sm-4">
          <router-view name="personal"></router-view>
        </div>
        <div class="col-sm-4">
          <router-view name="contact"></router-view>
        </div>
      </div>
    </div>
  </div>
</template>

{
    path: '/',
    name: 'Home',
    components: {
//默认为展现home组件
//根据需要通过name属性添加其他组件
      default: Home,
      'history': History,
      'personal': Personal,
      'contact': Contact
    }
```

### 8. 一些补充

#### 8.1 路由添加点击事件

路由要加入`.native`才能绑定事件

```vue
<router-link class="list-group-item list-group-item-action" tag="button" :class="isActive === index ? 'active' : ''"
                     @click.native="changeActive(item,index)" v-for="(item, index) in liName" :key="index" :to="toWhere[index]"
                     >
          {{item}}
        </router-link>



data () {
    return {
      isActive: 0,
      toWhere: ['/adminIndex', '/musicManage', '/userManage', '/commitManage'],
      liName: ['后台首页', '音乐管理', '用户管理', '评论管理']
    }
  },
  methods: {
    changeActive: function (item, index) {
      this.isActive = index
    }
  }
```



## 十五、 fetch，axios

在config/index.js 配置proxy

```javascript
proxyTable: {
    // 命名
      '/apis': {
        target: 'url',
            // 异步
        changeOrigin: true,
            // 把target 简写成apis
        pathRewrite: {
          '^/apis': ''
        }
      }
    },
```

```javascript
export default {
  name: 'App',
    //调用create 方法
  create () {
      // 访问资源文件
    fetch('/apis/test/testToken.php', {
        // 访问方法
      method: 'post',
        // 传递主体
      
      body: 'hello'
    }).then(result => {
      // console.log(result)
      return result.json()
    }).then(data => {
      console.log(data)
    })
  }
}

```

### axios 使用

先安装 及引入:

```cmd
npm install axios

import axios from 'axios'

Vue.prototype.$axios = axios

```



使用：(跨域问题上面已配置好)

```js
this.$axios.post('url',[参数1...])
.then(data => {
    //处理data
})
```

配置请求头：

```javascript
axios.defaults.headers.commom['参数名（如：token）']='...'
axios.defaults.headers.post["Content-type"]='application/json'(配置请求内容)
```

## 十六、 blog 搭建

### 1. 利用vue-resource

安装vue-resource

```cmd
npm install vue-resource --save
```

发布博客post方法：

```vue
post: function () {
      // http://jsonplaceholder.typicode.com/posts
      this.$http.post('http://jsonplaceholder.typicode.com/posts', {
        title: this.blog.title,
        body: this.blog.content,
        userId: 2
      }).then(function (data) {
        console.log(data)
      })
    }
```

![1560558968187](/../../../../../../vue学习笔记.assets/1560558968187.png)

![1560558994433](/../../../../../../vue学习笔记.assets/1560558994433.png)

获取get方法：

```js
this.$http.get('http://jsonplaceholder.typicode.com/posts/' + this.id).then(function (data) {
   this.blog = data.body
   console.log(this.blog)
  })
```

### 2. axios 替换vue-resource

1. axios 安装

   ```js
   npm install axios --save
   ```

   

2. 全局配置

```js
import axios from 'axios'

const instance = axios.create({
  baseURL: 'http://jsonplaceholder.typicode.com'
// axios.defaults.headers.comment['Authorization'] = 'Token'
// axios.defaults.headers.post['Content-type'] = 'Application/urlencode'
// axios.defaults.headers.get['Accept'] = 'application/json'
})
export default instance
```

3. 引用

```js
import axios from '../API/axios-auth'

created () {
    // this.$http.get('http://jsonplaceholder.typicode.com/posts').then(function (data) {
    //   this.blogs = data.body.slice(0, 10)
    // })
    axios.get('/posts').then((res) => {
      this.blogs = res.data.slice(0, 10)
      console.log(this.blogs)
    })
  }
```

