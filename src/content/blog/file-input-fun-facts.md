---
author: Jax
pubDatetime: 2024-09-01T23:45:00Z
modDatetime: 2024-09-01T23:45:00Z
title: 这还是你认识的那个文件 input 吗？
slug: file-input-fun-facts
featured: true
draft: false
tags:
  - 文件操作
description:
  文件 input 的那些你不知道的秘密
---
> 欢迎来到 Jax 的专栏「Web 玩转文件操作」，快来了解 Web 端关于文件操作的方方面面！
> 

对于咱们前端开发者来说，`<input type="file">` 可以算是老朋友了。它让用户可以导入本地文件，应用场景包括上传头像、管理后台配置图片等。除了 input 元素的通用特性，我们还可以通过 `multiple` 特性来控制单选/多选，通过 `accept` 来过滤掉不支持的文件格式，通过 dom 对象的 `files` 属性获取到所选的文件……可以说，我们和这个老伙计的配合已经相当默契了。

但是今天，我将向你展示这位老伙计不为大众所熟知的一面。下文中的几个“彩蛋”特性，会让你更全面、更立体地认识它。

## 世界大同：就连 MacOS 也有 C 盘

如本文开头所说，用户选择文件后，我们可以通过 `files` 属性获取文件。

```html
<input id="picker" type="file" />
```

```jsx
const picker = document.querySelector('#picker')
const files = picker.files
```

但对于其他 input —— 比如 `<input type=”text”>` ——来说，我们往往是通过 `value` 属性获取用户的输入值。既然文件被交给了 `files` 属性，那么 `value` 属性又承担了什么呢？

假设我们选中了 example.png 这个图片文件，然后打印一下 input 元素的 `value` 值，会得到这样的字符串：

```jsx
C:\fakepath\example.png
```

好奇怪的输出结果！我一个 MacOS 哪来的 C 盘？况且这张图片也不在 fakepath 这个文件夹里啊？

其实，不论是在有 C 盘的 WIndows 系统，还是没有 C 盘的 MacOS、Android 上，`value` 的值总是以「C:\fakepath\」开头。

原来在早期的 IE  浏览器中，为了避免泄露用户的真实文件系统结构，文件 input 的 `value` 值被特意设置成了「虚拟目录 + 文件名」的格式。虽然 fakepath 这个名字听起来跟闹着玩儿似的，但还是被一直沿用到了各大浏览器厂商的实现中。

另外，如果 input 上设置了 `multiple`，就算选择了多个文件，`value` 的值也并不会是多个文件路径，而是第一个选中文件的虚拟路径。

## 不要把宝全押在 `accept` 上

在上传头像时，我们会希望用户仅上传图片格式的文件，于是我们会这么做：

```html
<input type="file" accept="image/*" />
```

但实际上，把 `accept` 设为 image/*，并不能保证我们接收到的就一定是图片文件。

以 MacOS 为例，在文件选择界面的下方有一个按钮，点击后会显示出格式菜单：

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/2be50a8c-83cf-4fe1-bfcc-e64378d6abe8/0dab6d13-9be1-46bd-b9eb-9482ef0bcacf/image.png)

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/2be50a8c-83cf-4fe1-bfcc-e64378d6abe8/3e816e2f-da37-43e3-bd38-7acd62e2a38d/image.png)

切换到「所有文件(All Files)」选项后，你会发现你可以选中任意格式的文件，`accept` 这员守城大将就这样被轻松击败了。

所以在安全要求比较高的场景下，我们还是需要对最终接收到的文件做一层保险校验。

## 调兵遣将的虎符 —— `capture`

利用文件 input 的 `capture` 特性，我们可以轻松调起移动端设备的摄像头、麦克风，实现拍照、录像、录音。

### 摄像头

给文件 input 设置 `capture` 属性，并把 `accept` 设置为 image/*，可直接进入拍照模式；如果 `accept` 的值为 video/*，则可进入录像模式。

```html
// 录制视频
<input type="file" capture accept="video/*" />
// 拍照
<input type="file" capture accept="image/*" />
```

另外，如果你用 iPhone 来测试，可以给 `capture` 赋值为 user 或 environment，二者可分别打开前置/后置摄像头。

### 麦克风

如果 `accept` 的值是 audio/*，并设置了 `capture`，则可以直接进入录音模式：

```html
 <input type="file" capture accept="audio/*" />
```

但是注意！这个组合技的设备差异性极大！以下是我的三台测试机的表现：

- iPhone 12 Safari：打开了视频录制
- 红米 K40 Chrome：打开了录音机(Recorder)
- 华为 Mate 9 鸿蒙系统 Chrome：点击无响应

## 巧用 `webkitdirectory`

先来一道脑筋急转弯儿，请听题！

说有一个嵌套的文件夹结构，层数未知，每个文件夹下的文件数未知，问：如何用最少的代码计算出所有文件的数量？

```jsx
root
├── dir
│   ├── a.js
│   ├── dir
│   │    └── a.jsx
│   ├── c.vue
│   ├── d.js
│   ……
├── dir
│   └── a.vue
├── b.css
│
……
```

一般来说，我们通过文件 input 只能选择文件，但只要我们加上一个魔法特性，就能选中文件夹了。这个魔法的咒语就是 `webkitdirectory`。

```jsx
<input type="file" id="fileInput7" webkitdirectory />
```

选中文件夹并确认后，浏览器会弹出确认弹窗，提醒我们确认安全性：

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/2be50a8c-83cf-4fe1-bfcc-e64378d6abe8/a4bdfb54-8db7-4c7a-892b-2886f555d5b7/image.png)

点击「Upload」后，我们可以通过 `input.files` 拿到一个 `FileList` 格式的数组，包含了所选文件夹中的所有文件。

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/2be50a8c-83cf-4fe1-bfcc-e64378d6abe8/4c6c35c4-b98a-484d-83e7-53cb3e17b2c7/image.png)

读到这里，想必你肯定对前文的脑筋急转弯儿有答案了吧？

这个小小的骚操作，让我们无需写循环和递归遍历就能计算出文件数，真是毫无用处的技能呢。

我们再回头看一眼刚刚的 `FileList`，其中的元素对象有一个 `webkitRelativePath` 属性，其值是相对于所选文件夹的路径：

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/2be50a8c-83cf-4fe1-bfcc-e64378d6abe8/8994ca37-48cd-46d8-9de6-ee4d2ffc630d/image.png)

因此，即使层层嵌套的结构在 input 后被拍平了，但层级结构还记录在每个文件对象的属性里，并没有丢失。

## `cancel` 事件的隐藏坑

打开文件选择窗口后，如果点击取消按钮或者按 ESC 键，窗口会消失，退出选择流程。这个退出行为是可以通过 `cancel` 这个事件名来监听的。

但我想说的不是这个。

`cancel` 事件还有一个触发时机：选择了和上次相同的文件。

例如，我们首次选择了 a.txt、点击确认回到 Web 页，然后再点击 input，在弹出的窗口中再次选择同一个 a.txt 文件，点击确认按钮，此时也会触发 `cancel` 事件。

这个逻辑是有一定可能性坑到开发者的。因为即使在第二次选择时，a.txt 的文件内容已经更新了，但就因为文件名没变，仍然会无差别触发 `cancel` 事件，进而执行绑定的监听逻辑。不是那么精准，对吧。

## 写在结尾

恭喜你读完了本文，你真棒！

这次我们一起探究了文件 input 的几个彩蛋特性，没想到我们再熟悉不过的 input 也能玩出花来。请继续关注我的专栏，下一篇我们将会把瞄准镜对准 File API！

我是 Jax，在畅游 Web 技术领域的第 7 年，我仍然是坚定不移的 JavaScript 迷弟，Web 开发带给我太多乐趣。如果你也喜欢 Web 技术，或者想讨论本文内容，欢迎来聊！你可以通过下列方式找到我：

掘金：https://juejin.cn/user/1134351730353207

GitHub：https://github.com/JaxNext

微信：JaxNext