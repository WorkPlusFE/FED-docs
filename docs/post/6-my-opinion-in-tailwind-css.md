# 轻体验一番 Tailwind CSS 后，来谈谈我对它的看法

> [lwy1010](https://github.com/lwy1010) / 2021-2-1

如果前段时间你有关注一些技术社区分享的 [2020 年 CSS 发展状态报告](https://2020.stateofcss.com/en-US/)，你会发现一个叫 Tailwind CSS 的家伙，它的满意度、关注度、使用率和认知率都名列前茅，同时 GitHub 上 35K Star 也在侧面反映了这家伙大有来头。然后我有去体验了一番，接下来就来谈谈我对 Tailwind CSS 的一些看法。

## 什么是 Tailwind CSS

[Tailwind CSS](https://tailwindcss.com/)一个功能类优先(utility-first)的 CSS 框架，它提供了一系列功能类，让你可以在 HTML 中通过组合这些功能类的方式去构建用户界面，下面是官网的一个小例子：

<img :src="$withBase('/tailwindcss-demo.png')" alt="tailwindcss-demo">

这看起来似乎有点复杂，让我们先来简单说明一下：如 w-32 表示 width: 8rem 大小的宽度，简而言之，Tailwind CSS 将平时我们要写 CSS 样式类抽象成了一个一个可以复用的 CSS 功能类，然后通过组合这些功能类去编写页面的样式。

## 好与坏之间取决与你关注点

<img :src="$withBase('/rem-vs-acss.png')" alt="rem-vs-acss">

对于实现上面的一个简单的效果，以下是传统的方式与 Tailwind CSS utility-first 理念的对比：

```html
<!-- 传统的方式 -->
<div class="chat-notification">
  <div class="chat-notification-logo-wrapper">
    <img
      class="chat-notification-logo"
      src="/img/logo.svg"
      alt="ChitChat Logo"
    />
  </div>
  <div class="chat-notification-content">
    <h4 class="chat-notification-title">ChitChat</h4>
    <p class="chat-notification-message">You have a new message!</p>
  </div>
</div>

<style>
  .chat-notification {
    display: flex;
    max-width: 24rem;
    margin: 0 auto;
    padding: 1.5rem;
    border-radius: 0.5rem;
    background-color: #fff;
    box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
  }
  .chat-notification-logo-wrapper {
    flex-shrink: 0;
  }
  .chat-notification-logo {
    height: 3rem;
    width: 3rem;
  }
  .chat-notification-content {
    margin-left: 1.5rem;
    padding-top: 0.25rem;
  }
  .chat-notification-title {
    color: #1a202c;
    font-size: 1.25rem;
    line-height: 1.25;
  }
  .chat-notification-message {
    color: #718096;
    font-size: 1rem;
    line-height: 1.5;
  }
</style>
```

```html
<!-- Tailwind CSS utility-first的方式 -->
<div
  class="p-6 max-w-sm mx-auto bg-white rounded-xl shadow-md flex items-center space-x-4"
>
  <div class="flex-shrink-0">
    <img class="h-12 w-12" src="/img/logo.svg" alt="ChitChat Logo" />
  </div>
  <div>
    <div class="text-xl font-medium text-black">ChitChat</div>
    <p class="text-gray-500">You have a new message!</p>
  </div>
</div>
```

传统的方式对 html 友好，CSS 服务于 HTML 的结构，一个 class 中有多个 CSS 属性，这也是日常工作中主流的方式，这也是《禅意花园》所提倡的“关注点分离原则”。这种方式的代码如果维护起来，通常是直接在现有的 class 上面修改，这样重复的样式也越来越多，如果项目一直在维护，很容易导致 CSS 代码的变得冗余、繁琐和难以维护，毕竟很多数时候，我们更在意效果的实现。

而 Tailwind CSS 的 utility-first 的方式其实是上古时期原子化 CSS 理念的一种实现，它推崇样式代码的复用，每一个 class 都代表着一种 CSS 属性。原子化 CSS 最早由雅虎提出来，代表作是 ACSS，表示的是原子化 CSS(Atomic CSS)。这种方式可以快速构建用户界面且样式代码的复用可以更好地减少 CSS 代码的体积。但这种方式挑战了传统意义上编写 CSS 的最佳实践——关注点分离原则，而且有被滥用后带来维护成本爆炸的风险，所以在早期没有火起来，直到 Tailwind CSS 的诞生。

通过简单对比后，你会发现事物的好于坏只是取决了你的关注点，切确来说你是更关注对 HTML 友好的方式还是更喜欢样式代码复用的方式。毕竟如今的 Tailwind CSS 配合现代化的工具(postcss、[purgecss](https://github.com/FullHuman/purgecss)、编辑器的 IntelliSense 插件等)后，可以很好地解决项目的包体积、定制与拓展性、项目上手和维护成本等问题了。

## 总结

Tailwind CSS 带着原子化 CSS 的理念卷土重来，同时也确实给出了一些不错的答案：

- 丰富的预设。响应式；暗黑模式；丰富的颜色和调色版等。

- 良好的定制性和拓展性。通过`tailwind.config.js`几乎可以自定义一切预设类的内容；通过@apply 指令来拓展你自己的类等。

- 不错的生态。兼容各大主流框架；提供了 VS Code IntelliSense 插件；直观易读的文档等。

当然也有一些缺点：

- 要去熟悉一套新的 class 语法等。

总的来说，丰富的预设、良好的定制性和拓展性、不错的生态等都表明了它是一个完整的原子化 CSS 的解决方案，我想这应该也是它收获 35K 多 star 的主要原因，但能否持续的火热甚至成为下一个 Bootstrap 级别的 CSS 框架，那就要看更多的开发人员对原子化 CSS 这种理念买不买账了。

## 参考链接

- [[css-utility-classes-and-separation-of-concerns](https://adamwathan.me/css-utility-classes-and-separation-of-concerns)]：Tailwind CSS 作者在设计 Tailwind CSS 的一些思考，推荐阅读下。
