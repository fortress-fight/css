# Css Module

![20181126232617.png](http://resources.ffstone.top/resource/image/20181126232617.png)

了解学习 CSS 模块化相关知识

## CSS 为什么要模块化

1.  全局污染

    `CSS` 使用**全局选择器机制**来设置样式，优点是方便重写样式。缺点是所有的样式都是全局生效，样式可能被错误覆盖，因此产生了非常丑陋的 `!important`，甚至 `inline !important` 和复杂的选择器权重计数表，提高犯错概率和使用成本。`Web Components` 标准中的 `Shadow DOM` 能彻底解决这个问题，但它的做法有点极端，样式彻底局部化，造成外部无法重写样式，损失了灵活性。

2.  命名混乱

    由于全局污染的问题，多人协同开发时为了避免样式冲突，选择器越来越复杂，容易形成不同的命名风格，很难统一。样式变多后，命名将更加混乱。

3.  依赖管理不彻底

    组件应该相互独立，引入一个组件时，应该只引入它所需要的 `CSS` 样式。但现在的做法是除了要引入 `JS`，还要再引入它的 `CSS`，而且 `Saas/Less` 很难实现对每个组件都编译出单独的 `CSS`，引入所有模块的 `CSS` 又造成浪费。`JS` 的模块化已经非常成熟，如果能让 `JS` 来管理 `CSS` 依赖是很好的解决办法。`Webpack` 的 `css-loader` 提供了这种能力

4.  无法共享变量

    复杂组件要使用 `JS` 和 `CSS` 来共同处理样式，就会造成有些变量在 `JS` 和 `CSS` 中冗余，`Sass/PostCSS/CSS` 等都不提供跨 `JS` 和 `CSS` 共享变量这种能力。

5.  代码压缩不彻底

    由于移动端网络的不确定性，现在对 `CSS` 压缩已经到了变态的程度。很多压缩工具为了节省一个字节会把 `16px` 转成 `1pc`。但对非常长的 `class` 名却无能为力。

## CSS 解决方案

CSS 模块化的解决方案有很多，大体可以分为两种：

1.  `CSS in JS`

    优点：能给 CSS 提供 JS 同样强大的模块化能力  
    缺点：不能利用预处理器或者后处理器，并且处理 `:hover` 和 `:active` 比较复杂

2.  `CSS Module`

    [CSS Module](https://github.com/css-modules/css-modules) 是一种依旧使用 `CSS`，但使用 `JS` 来管理样式依赖

    优点：

    -   `CSS Module` 能最大化的结合现有 CSS 生态和 JS 模块化能力
    -   API 十分简洁
    -   发布时依旧是编译出单独的 JS 和 CSS
    -   不依赖任何框架，只要使用 `Webpack` 就能使用

## CSS Module 的使用

### CSS Module 的导入导出

使用 `Webpack`，启用 `modules`

```js
// webpack.config.js
css?modules&localIdentName=[name]__[local]-[hash:base64:5]
```

name -- 文件名
local -- 本地变量名
hash -- 随机 hash 值

加上 `modules` 即为启用，`localIdentName` 是设置生成样式的命名规则。

示例：

```css
/* components/Button.css */
.normal {
    /* normal 相关的所有样式 */
}
.disabled {
    /* disabled 相关的所有样式 */
}
```

使用：

```js
/* components/Button.js */
import styles from "./Button.css";

console.log(styles);

buttonElem.outerHTML = `<button class=${styles.normal}>Submit</button>`;
```

生成：

```html
<button class="button--normal-abc53">Submit</button>
```

`CSS Modules` 对 `CSS` 中的 `class` 名都做了处理，使用对象来保存原 `class` 和混淆后 `class` 的对应关系

通过这些简单的处理，CSS Modules 实现了以下几点：

1.  所有样式都是 local 的，解决了命名冲突和全局污染问题
2.  class 名生成规则配置灵活，可以此来压缩 class 名
3.  只需引用组件的 JS 就能搞定组件所有的 JS 和 CSS
4.  依然是 CSS，几乎 0 学习成本

### 样式默认局部

使用了 CSS Module 后，就相当于给每个 `class` 名外加了一个 `:local`，以此来实现样式的局部化。如果希望切换到全局模式，使用 `:global`

```css
.normal {
    color: green;
}

/* 以上与下面等价 */
:local(.normal) {
    color: green;
}

/* 定义全局样式 */
:global(.btn) {
    color: red;
}

/* 定义多个全局样式 */
:global {
    .link {
        color: green;
    }
    .box {
        color: yellow;
    }
}
```

### Compose 复用样式

对于样式复用，CSS Modules 只提供了唯一的方式来处理：`composes` 组合

```css
/* components/Button.css */
.base {
    /* 所有通用的样式 */
}

.normal {
    composes: base;
    /* normal 其它样式 */
}

.disabled {
    composes: base;
    /* disabled 其它样式 */
}
```

```js
import styles from "./Button.css";

buttonElem.outerHTML = `<button class=${styles.normal}>Submit</button>`;
```

生成：

```html
<button class="button--base-daf62 button--normal-abc53">Submit</button>
```

由于在 `.normal` 中 `composes` 了 `.base`，编译后会 `normal` 会变成两个 `class`。

composes 还可以组合外部文件中的样式。

```css
/* settings.css */
.primary-color {
    color: #f40;
}

/* components/Button.css */
.base {
    /* 所有通用的样式 */
}

.primary {
    composes: base;
    composes: primary-color from "./settings.css";
    /* primary 其它样式 */
}
```

对于大多数项目，有了 `composes` 后已经不再需要 Sass/Less/PostCSS。但如果你想用的话，由于 `composes` 不是标准的 CSS 语法，编译时会报错。就只能使用预处理器自己的语法来做样式复用了。

### CSS 变量导出

CSS Modules 中没有变量的概念，这里的 CSS 变量指的是 Sass 中的变量。

`:export` 关键字可以把 CSS 中的变量输出到 JS 中。示例：

```scss
/* config.scss */
$primary-color: #f40;

:export {
    primaryColor: $primary-color;
}
```

```js
/* app.js */
import style from "config.scss";

// 会输出 #F40
console.log(style.primaryColor);
```

## CSS Modules 使用技巧

CSS Modules 是对现有的 CSS 做减法。为了追求简单可控，作者建议遵循如下原则：

1.  只使用 class 名来定义样式
2.  不层叠多个 class 只使用一个 class 定义样式
3.  所有样式通过 `composes` 组合来实现复用
4.  不嵌套

> CSS Modules 只会转换 class 名和 id 选择器名相关的样式。

### 外部样式覆盖局部样式

当生成混淆的 `class` 名后，可以解决命名冲突，但因为无法预知最终 `class` 名，不能通过一般选择器覆盖。我们现在项目中的实践是可以给组件关键节点加上 `data-role` 属性，然后通过属性选择器来覆盖样式。

```js
// dialog.js
return (
    <div className={styles.root} data-role="dialog-root">
        <a className={styles.disabledConfirm} data-role="dialog-confirm-btn">
            Confirm
        </a>
        ...
    </div>
);
```

```css
/* dialog.css */
.root[data-role="dialog-root"] {
    // override style
}
```

因为 CSS Modules 只会转变类选择器，所以这里的属性选择器不需要添加 `:global`。

### 与全局样式共存

前端项目不可避免会引入 `normalize.css` 或其它一类全局 css 文件。使用 Webpack 可以让全局样式和 CSS Modules 的局部样式和谐共存。下面是我们项目中使用的 webpack 部分配置代码：

```js
module: {
    loaders: [
        {
            test: /\.jsx?$/,
            loader: "babel"
        },
        {
            test: /\.scss$/,
            exclude: path.resolve(__dirname, "src/styles"),
            loader:
                "style!css?modules&localIdentName=[name]__[local]!sass?sourceMap=true"
        },
        {
            test: /\.scss$/,
            include: path.resolve(__dirname, "src/styles"),
            loader: "style!css!sass?sourceMap=true"
        }
    ];
}
```

引用：

```js
/* src/app.js */
import "./styles/app.scss";
import Component from "./view/Component";

/* src/views/Component.js */
// 以下为组件相关样式
import "./Component.scss";
```

这样所有全局的样式都放到 `src/styles/app.scss` 中引入就可以了。其它所有目录包括 `src/views` 中的样式都是局部的。

## 总结

CSS Modules 很好的解决了 CSS 目前面临的模块化难题

1.  支持与 Sass/Less/PostCSS 等搭配使用。
2.  同时也能和全局样式灵活搭配，便于项目中逐步迁移至 CSS Modules。
3.  CSS Modules 的实现也属轻量级。

## 参考

[CSS Modules 详解及 React 中实践](https://github.com/camsong/blog/issues/5)
