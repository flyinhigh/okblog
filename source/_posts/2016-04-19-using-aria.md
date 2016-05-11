title: Using ngAria（译）
date: 2016-04-19 11:24:28
tags: 现实
---

原文地址：http://angularjs.blogspot.com/2014/11/using-ngaria.html

非常开心看到Angular.js官方也开始关注无障碍改造，并且提供了一个专门的模块。

********************以下为正文***********************

Angular从1.3.0 版本开始新增了可访问性模块[ngAria](https://docs.angularjs.org/guide/accessibility)。就像致力于通过[社区方式](https://github.com/angular/angular.js/pull/8342)推动无障碍的人们一样，我觉得很有必要介绍一下ngAria这个模块，它大大提高了残障人士使用互联网的体验。

这篇文章中会讲到：
+ [什么是ngAria](#什么是ngAria)
+ [为什么会有ngAria](#为什么会有ngAria)
+ [怎么样使用ngAria](#怎么样使用ngAria)
+ [新特性](#ngAria新特性)

## 什么是ngAria
ngAria的目标在于通过自动给directive添加常用的[ARIA](https://www.w3.org/WAI/intro/aria)属性，提高Angular.js自身的可访问性。开发者只需要简单的require ngAria模块，Angular.js会在程序运行时自动给内置的directive添加可访问支持。 

目前，ngAria 支持以下内置directives: **ngModel**, **ngShow**, **ngHide**, **ngDisabled**, **ngClick**, **ngDblclick** 和 **ngMessages**。现在支持的Aria属性还比较有限，不过一直在发展，后面会讲到。

>> 相关阅读: [What is WAI-ARIA, what does it do for me, and what not?](http://www.marcozehe.de/2014/03/27/what-is-wai-aria-what-does-it-do-for-me-and-what-not/)

把可访问性放到到一个模块中维护，可以方便添加和测试，也方便后期升级。但同时也容易让程序员忘记，你只有调用了ngAria模块，程序才能自动帮你做事情——不过我们始终不能代替你(所以还是把主动权交给你)。当你使用ngAria模块后，你不需要做很多事情就可以让你的用户体验得到很大的提升。所以，了解ngAria能做什么不能做什么很重要，通过这些你可以知道它在你的代码中做了哪些改变。

想知道ngAria是怎么做工作的，参见[ngAria Developer Guide](https://docs.angularjs.org/guide/accessibility)

## 为什么会有ngAria
这个世界上有相当一部分人群是依靠辅助设备，例如读屏软件，高对比度模式，盲文键盘，隐藏字幕等工具来使用互联网产品和服务的。可惜目前各种流行的MV*框架包括Angular.js的关注点大多集中在诸如移动表现，数据绑定，自动化工具和ES6支持等等很绚丽的新特性上，从而忽视了可访问性，导致由这些框架开发出来的应用可访问性的体验很差，我们热爱于在工作中创新，但是我们忽略了诸如HTML语义，键盘支持这些基本的东西。

在过去，Angular.js在可访问性方面做得不是很多——实际上，在那些不支持可访问性的地方注入可访问性特性是一件很有挑战的事情，比如一个自定义的directive `<md-checkbox>` 上的disabled状态。 

下面是渲染后的代码

```
<md-checkbox ng-model="someModel" ng-disabled="isDisabled" disabled>
    Inaccessible Checkbox
</md-checkbox>
```
这个checkbox directive模拟了一个虚拟checkbox，它是不可访问的，因为它没有给辅助设备提供任何信息并且不支持键盘操作，(辅助设备)用户不知道其是否被禁用。像ng-disabled这样的directive在那些无语义的元素上使用起来非常方便简单，这就要求我们在写代码的时候必须要考虑对应[Aria](https://www.w3.org/WAI/intro/aria)的运用，而一个牛逼的框架应该帮助你自动完成这部分工作。

**通过对ngAria的介绍，我们知道当调用ngAria模块后，程序会自动添加常用的Aria属性，用来向那些使用辅助设备的用户同步应用的状态。**

接下来是鉴证奇迹的时刻。

```
<md-checkbox ng-model="someModel" aria-disabled="true" tabindex="0">
    Checkbox
</md-checkbox>
```
*注意：该例中同时还需要 role="checkbox”和aria-labe两个可访问性属性，这连个属性在 [Angular Material](https://material.angularjs.org/latest/) 中已经内置了，而在ngAria的未来版本中会得到支持*

## 怎么样使用ngAria
使用ngAria非常简单，require ngAria module即可。

```
angular.module('myApp', ['ngAria'])...
```
在including ngAria之后，只要是ngAria支持的directive都会被自动添加ARIA支持，想要了解更多ngAria对directive作用的效果，请移步[ ngAria Developer Guide](https://docs.angularjs.org/guide/accessibility)

### 禁用Aria属性
在有些场景下可能不太适合做Aria的处理，我们可以通过$ariaProvider做一些配置，禁用指定的Aria属性。
```
angular.module('myApp', ['ngAria'], function config($ariaProvider) {
  $ariaProvider.config({
    tabindex: false
  });
});
```
有哪些属性支持做配置，可查看[API Documentation for $ariaProvider](https://docs.angularjs.org/api/ngAria/provider/$ariaProvider)

## ngAria新特性
### ngMessages
最近的一次针对表单验证模块 [ngMessage](https://docs.angularjs.org/api/ngMessages/directive/ngMessages) 的[pull request](https://github.com/angular/angular.js/pull/9834) 新增了对ARIA的支持，用来帮助那些弹出的错误提示能被读屏软件捕捉到。通过对ngMessage添加[aria-live](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Live_Regions)属性，就能很方便的实现这个功能。当动态区域内容发生了改变的时候，aria-live可以用来通知辅助设备，这是ngAria一个很典型的运用。

## ngAria的规划
### ARIA ROLE
为了更好地与辅助设备进行交互，ngAria需要提供更多的role支持自定义的控件，例如`<md-checkbox>`和`<div type="range”>`。有一个[issure](issue)已经开始申请添加这个两个role，未来更多的role也将会补充上去。

### 端对端的无障碍测试
准备工作已经开始，类似ng-hint，通过[Protractor](https://github.com/angular/protractor)的插件系统来实现无障碍自动化集成测试。未来包括HTML 层级，键盘操作，label缺失以及颜色对比度等等内容都将纳入自动化测试范畴，让发现并改正无障碍问题变得更加方便有趣。大家敬请期待。

### $mdAria.expect
[Angular Material ](https://material.angularjs.org/latest/)中提供了一个[ARIA service method](https://github.com/angular/material/blob/master/src/core/services/aria/aria.js) 来解决一个常见的无障碍问题：label缺失，这个问题是指用于向辅助工具提供页面控件对应文案说明的label经常被遗忘。针对一些常见的表单控件，比如checkbox和radiobutton，$mdAria 试图通过自动复制text中的内容到aria-label属性中解决这个问题，如果发现没有合适的text，会在javascript控制台中打印出一条警告，告诉开发者应该添加对应的label来实现可访问。我们一直在修正这个组件的bug，并且评估它在ngAria或者[ngHint](https://github.com/angular/angular-hint)中运用的可行性。

### 其他
还有一些改进也正在计划和讨论中：包括alt属性缺失检测，默认的键盘操作支持，例如escape和question，颜色对比度检测等等。我非常惊喜地看到，自从我们在[Angular Material ](https://material.angularjs.org/latest/)中尝试使用naAria之后，人们开始越来越关注可访问性，如果你觉得ngAria有需要改进的地方，请给我们留言或者提交pull request。

### Let's Make it Better
作为开发者，我们有责任做出一个可访问的并且易测试的互联网应用，框架本身也会尽可能多的自动完成这部分工作。作为一个新的模块，ngAria会持续不断的针对发现的各类可访问性问题做修复或者提示。不过，要做到在程序表现，开发者体验以及最终用户需求之间做到平衡，还需要小心仔细的计划和实施，这些都需要时间。如果你对ngAria有想法，欢迎在Github上进行评论。

### 特别感谢
这个模块最早是由[@arbus](https://github.com/arbus/ng-aria)贡献，通过Github社区不断的反馈和改进，同时也得到了Angular.js核心小组成员obias Bosch，Brian Ford, Marcy Sutton 和 Pete Bacon Darwin.的支持，在此一并感谢。

发表于 2014年11月10日，作者 [Marcy Sutton](https://plus.google.com/103109242569180284810)

