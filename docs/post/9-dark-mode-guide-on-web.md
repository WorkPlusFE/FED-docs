# Web 深色模式适配简明指南

> [lwy1010](https://github.com/lwy1010) / 2021-4-29

现代人的日均屏幕使用时间正在日益增长，同时苹果、谷歌等等纷纷在其产品中引入了深色模式(Dark Mode)，所以我们有理由相信，未来无论移动端 app、web 程序或者桌面应用，深色模式都会成为用户界面的标准模式之一。

作为一个有职业素养的 web 开发工程师，我们理应掌握如何为 web 程序适配深色模式，那么今天就通过这篇文章给大家分享下如何为 web 程序适配深色模式。

## 深色模式的实现原理

先让我们化身为用户去体验下主流 web 程序的深色模式的交互——首次进入时 web 程序会为我们“预设”一个默认的主题（通常为浅色模式，也可以是深色模式）；然后在 web 程序界面某个地方可以让我们在浅色和深色模式之间进行切换，除此之外在重新打开时 web 程序它会默认使用我们最后一次偏好的主题（如深色模式）。

简单地体验下整个交互可以梳理出适配深色模式的核心思路就两点：

1. 为用户“预设”一个默认主题
2. 提供切换主题的功能并记录用户偏好的主题

很好，下面让我们站在 web 开发工程师的角度来思考如何一步步实现我们刚梳理出的思路：

先来看第一点，对于用户进入 web 程序时为其“预设”一个默认的主题这点，能想到的最简单的做法就是先默认一个模式（如浅色模式）。然而这种做法的用户体验并不好，如今深色模式得到了主流的操作系统的的支持，如果我们能在浏览器中获取到用户在其操作系统中所偏好的模式，然后让页面主题能跟随系统自动切换深浅模式，这样的做法对用户来说体验会好一点（试想一下，我们更偏好深色模式且在系统中设置了深色模式，整个工作环境都处于深色模式的我们打开一个 web 程序得到的却是一个亮色的界面，这是多么糟糕的体验）。

值得庆幸的是在浏览器中通过媒体查询可以让我们的思路得以实现，借助媒体查询有两种方式可以实现页面主题跟随系统自动切换深浅模式：

### CSS 方式

CSS 的媒体特性 [prefers-color-scheme](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@media/prefers-color-scheme) 可以检测用户是否有将系统的主题色设置为浅色或者深色，它可以有三种取值： light、 dark 和 no preference：

```css
@media (prefers-color-scheme: light) {
  /* 用户系统为浅色主题时生效 */
}

@media (prefers-color-scheme: dark) {
  /* 用户系统为深色主题时生效 */
}

@media (prefers-color-scheme: no-preference) {
  /* 浏览器不支持prefers-color-scheme特性或者未得知用户系统偏好的模式时生效 */
}
```

这时配合 [CSS 变量](https://developer.mozilla.org/zh-CN/docs/Web/CSS/--*)即可动态设置对应的主题：

```css
:root {
  --bg: #fff;
  --text-color: #222;
  --link-color: #0033cc;
}

body {
  color: var(--text-color);
  background: var(--bg);
}

body a {
  color: var(--link-color);
}

@media (prefers-color-scheme: dark) {
  :root {
    --bg: #121212;
    --text-color: #eee;
    --link-color: #809fff;
  }
}
```

为了减少文章的篇幅和呈现更好的示例效果，你可以点击[这里](https://codepen.io/lwy1010/pen/mdRvjvb) 查看示例代码及效果(Mac 系统的用户在偏好设置->通用->外观主题选项中切换外观来查看效果；win10 系统的用户可以在设置->颜色->切换应用模式选项中切换应用模式来查看效果)。

### JavaScript 方式

通过浏览器提供的 [window.matchMedia](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/matchMedia) 方法可以检测到用户在系统中设置偏好的主题，它的使用方法也比较简单：

```javascript
const prefersDarkScheme = window.matchMedia("(prefers-color-scheme: dark)");
if (prefersDarkScheme.matches) {
  document.body.classList.add("dark");
} else {
  document.body.classList.remove("dark");
}
```

得到 matchMedia 方法的查询结果后可以配合 CSS 变量来动态设置对应的主题：

```css
:root {
  --bg: #fff;
  --text-color: #222;
  --link-color: #0033cc;
}

:root .dark {
  --bg: #121212;
  --text-color: #eee;
  --link-color: #809fff;
}

body {
  color: var(--text-color);
  background: var(--bg);
}
body a {
  color: var(--link-color);
}
```

同理，点击 [这里](https://codepen.io/lwy1010/pen/MWJLzjb) 查看示例的完整代码及效果。

### 存储用户偏好的主题

现在我们的 web 程序可以根据用户系统自动切换深浅模式了，根据上面我们梳理出来的思路，现在我们还需要为用户提供切换模式的功能并且记录他们的偏好，以确保他们能在 web 程序中能覆盖系统中的偏好设置并且在后续访问 web 程序时保持一致。有很多方法可以实现这一点，这里通过一个 简单的 JavaScript 脚本程序和浏览器的[localStorage](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/localStorage)本地存储来实现：

```js
const currentTheme = localStorage.getItem("theme");
const prefersDarkScheme = window.matchMedia("(prefers-color-scheme: dark)");
const btn = document.querySelector(".btn-toggle");

// 自动切换
const autoToggleTheme = (e) => {
  if (e.matches) {
    document.body.classList.add("dark");
  } else {
    document.body.classList.remove("dark");
  }
};

// 手动切换
const handleToggleTheme = () => {
  if (prefersDarkScheme.onchange) prefersDarkScheme.onchange = null;
  document.body.classList.toggle("dark");
  let theme = "light";
  if (document.body.classList.contains("dark")) theme = "dark";
  localStorage.setItem("theme", theme);
};

if (!currentTheme) {
  prefersDarkScheme.onchange = autoToggleTheme;
} else {
  document.body.classList.add(currentTheme);
}

btn.addEventListener("click", handleToggleTheme);
```

同理，你可以点击 [这里](https://codepen.io/lwy1010/pen/MWJLzjb) 查看示例的完整代码及效果。

## 其他细节

到目前为止，我们掌握了为 web 程序适配深色模式的原理，然而一个舒适的深色模式并非将底色变黑，将文字变白这么简单（如果你对这方面感兴趣，文章尾部有提供了一些相关的链接或者你也可以自行搜索诸如"dark mode design"的之类关键字进行了解）。虽然说在分工明确的互联网时代，模式中主题颜色的设计规范更多会交由设计师来处理，但是我们在处理事情时细心一点可以让事情变得更好，让我们来看看关于适配深色模式的一些值得注意细节：

### 添加过渡效果

在深浅模式切换的过程添加一个切换效果可以让模式间的切换有个平滑过渡的效果，让用户有着更好的感观体验。借助 CSS 的[transition](https://developer.mozilla.org/zh-CN/docs/Web/CSS/transition)属性可以实现我们的想法：

```css
body {
  --duration: 0.5s;
  --timing: ease;

  color: var(--color);
  background-color: var(--background-color);

  /* 在模式切换过程中给color和background-color属性添加过渡效果 */
  transition: background-color var(--duration) var(--timing), color var(
        --duration
      ) var(--timing);
}
```

同理，点击 [这里](https://codepen.io/lwy1010/pen/PoWgwQN) 查看完整代码示例及效果。

### 图片的处理

图片是在 web 程序中常见的元素，在深色模式下，太亮的图片可能令用户感到不舒服，所以我们可以在不同模式下给用户展示不同效果的图片。有很多方式可以实现这一点，就 CSS 层面来说，我们可以借助 CSS 的 [opacity](https://developer.mozilla.org/zh-CN/docs/Web/CSS/opacity) 属性、[mix-blend-mode](https://developer.mozilla.org/zh-CN/docs/Web/CSS/mix-blend-mode) 属性(如果是背景的话则用 [background-blend-mode](https://developer.mozilla.org/zh-CN/docs/Web/CSS/background-blend-mode) 属性)或者 [filter](https://developer.mozilla.org/zh-CN/docs/Web/CSS/filter) 函数等方式来对图片的展示效果进行调整处理，下面是一个可供参考的示例：

```css
@media (prefers-color-scheme: dark) {
  img {
    /* 在深色模式下图片的亮度比原来低点，对比度比原来高点，这样子图片的显示效果会柔和一点且用户看起来舒服点 */
    filter: brightness(0.8) contrast(1.2);
  }
}
```

同理，点击[这里](https://codepen.io/lwy1010/pen/WNRWQNm)查看上述示例的完整代码及效果。

除此之外，你甚至可以为不同模式提供特地优化过的图片，然后借助 [\<picture\>](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/picture) 元素加载对应模式下的图片：

```html
<picture>
  <!-- 在深色模式时加载 -->
  <source media="prefers-color-scheme: dark" srcset="" />
  <!-- 在亮色模式时或在不支持prefers-color-scheme特性的浏览器时加载  -->
  <source
    media="(prefers-color-scheme: light), (prefers-color-scheme: no-preference)"
    srcset=""
  />
  <!-- 在不支持picture元素的浏览器时加载 -->
  <img src="" />
</picture>
```

### 图标的处理

图标也是我们在 web 程序中的老朋友了，在深色模式下，我们也要对图标进行一些处理以提升用户的体验。在 web 程序中的图标大多为字体图标或者 svg 格式的图标，如果我们使用的是 svg 格式的 图标，我们可以通过 fill、stroke 属性来改变填充色和描边色。而如果使用的是字体图标，则可以简单地改变 color 属性来改变图标的颜色，下面是一些可供参考的示例：

```css
/* svg图标 */
body.dark svg {
  fill: #efefef;
}

/* 字体图标(这里以font-awesome图标库作为示例)*/
body.dark .fa {
  color: #efefef;
}
```

## 总结

上面我们通过许多示例阐述了为 web 程序适配深色模式的基本原理和一些值得我们去注意的细节，希望对你有所帮助，感觉你的阅读:)

## 扩展阅读及参考链接

- [Create A Dark/Light Mode Switch with CSS Variables](https://dev.to/ananyaneogi/create-a-dark-light-mode-switch-with-css-variables-34l8)

- [A Complete Guide to Dark Mode on the Web](https://css-tricks.com/a-complete-guide-to-dark-mode-on-the-web/#top-of-site)

- [A guide to implemen­ting dark modes on websites](https://www.kooslooijesteijn.net/blog/add-dark-mode-to-website)

- [The future is dark](https://medium.com/by-digiti/the-future-is-dark-8c3bdadf9fdc)

- [Dark Mode - Unmistified!](https://dev.to/akhilarjun/dark-mode-unmistified-1ji6)

- [WeGame 暗色模式实践总结](https://mp.weixin.qq.com/s?src=11&timestamp=1619586633&ver=3035&signature=4k6SXzntS*2tIyZLzhmc6QQcc5iTuASxtmOU56Ape7p14FSOinGUgst0Akj-SO3zu6PlVOIZjDIusqWQAkfdaUhS*HL115MN3EVrEA5cz4jKUWiAlSlXJCXu1wZpruqq&new=1)
