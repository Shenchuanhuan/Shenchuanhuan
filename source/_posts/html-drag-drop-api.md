---
title: HTML Drag and Drop API notes
date: 2022-11-08
tags: [Html, Drag, Drop]
categories: [笔记]
---

### 拖拽事件类型：
| Event | Fires when... | info |
|:-----:|:--------|:-----------|
| drag | 被拖拽实体（元素或者文字选中片段）被拖拽时 |
| dragend | 拖拽动作结束时(比如说松开鼠标按钮或者按`Esc`键）|在`source element`上触发，无论事件完成还是取消，都会触发。可以通过检查`dropEffect`的值来确认当次拖拽事件是否成功。
| dragenter | 被拖拽实体进入有效的接收拖拽区域时 |
| dragleave | 被拖拽实体离开有效的接收拖拽区域时 |
| dragover | 被拖拽实体在有效接收区域中移动时触发，每几百毫秒触发一次 |
| dragstart | 用户开始拖拽实体 |
| drop | 将拖拽实体丢在有效区域时触发 |

> **note**
>> Neither dragstart nor dragend events are fired when dragging a file into the browser from the OS.

### 接口
* [DragEvent](https://developer.mozilla.org/en-US/docs/Web/API/DragEvent)
> 有一个`dataTransfer`属性，是`DataTransfer`对象，该属性用于保存event的数据，它的`setData`方法可以往拖拽数据里添加数据。
* [DataTransfer](https://developer.mozilla.org/en-US/docs/Web/API/DataTransfer)
> 对象描述了拖拽事件的状态（`copy`还是`move`）,拖拽数据（一个或多个），每个拖拽对象的`MIME type`。对象还提供了修改数据的方法（`add` or `remove` items)。
* [setData](https://developer.mozilla.org/en-US/docs/Web/API/DataTransfer/setData)
* 拖拽数据由两部分组成：数据的格式（或类型）和数据值。数据的格式是类型字符串，数据值也是一个字符串。使用场景之一：拖拽开始时设置数据，拖拽结束或拖拽过程中，检测数据的类型是否正确，再确定是否接收
* 可以多次使用`setData`设置不同类型的数据值
* 使用`clearData(type)`来清除数据，如果没有type参数，则会清除所有数据。
[setDragImage](https://developer.mozilla.org/en-US/docs/Web/API/DataTransfer/setDragImage)

> 实现拖拽时，应该只用到以上两个接口。前两个接口各浏览器广泛支持，后两个接口有诸多兼容性问题。
* [DataTransferItem](https://developer.mozilla.org/en-US/docs/Web/API/DataTransferItem)
* [DataTransferItemList](https://developer.mozilla.org/en-US/docs/Web/API/DataTransferItemList)


### 拖拽实现要点
* 可拖拽元素必须有`draggable`属性

```javascript
<p id="p1" draggable="true">This element is draggable.</p>
```
* 一般来说接收拖拽的元素要监听`dragover`事件，通过`e.preventDefault()`会阻止浏览器的默认拖拽行为（被拖拽实体直接在浏览器中打开），在`React`里，对应的事件名称是`onDragOverCapture`

* 在HTML中，只有图片、链接、文字片段有默认的拖拽行为（images & . links`draggable`属性默认为`true`）。其它元素要实现拖拽，必须做到以下三点：
    1. 设置元素的`draggable`属性为`true`
    2. 监听`dragstart`事件
    3. 在上面的监听器里设置拖拽数据

* 可以在`dragenter` 或 `dragover`的`events`中修改`dropEffect`属性，如果为`none`，则表示目标不接收任何源文件。

### Drag Effect
>设置`effectAllowed`属性来决定拖拽方式：
* copy: 复制拖拽节点到目标节点
* move: 移动拖拽节点到目标节点
* link: 在拖拽节点和目标节点之间建立链接
* copyMove: copy or move only
* copyLink: copy or link only
* linkMove: link or move only
* all: copy, move, or link
```
// for instance
event.dataTransfer.effectAllowed = "copy";
```


### Note
> When an element is made draggable, text or other elements within it can no longer be selected in the normal way by clicking and dragging with the mouse. Instead, the user must hold down the Alt key to select text with the mouse, or use the keyboard.


#### DEMOS
* [Drag and Drop: Copy and Move elements with DataTransfer](https://mdn.github.io/dom-examples/drag-and-drop/copy-move-DataTransfer.html)
* [Drag and Drop: Copy and Move elements with DataTransferItemList interface](https://mdn.github.io/dom-examples/drag-and-drop/copy-move-DataTransferItemList.html)
* [Dragging and dropping files (Firefox only)](https://jsfiddle.net/9C2EF/)
* [Dragging and dropping files (All browsers)](https://jsbin.com/hiqasek/edit?html,js,output)
* [A parking project using the Drag and Drop API](https://park.glitch.me/)
* [local demo](/demos/drag-drop-demo.html)

# 参考资料
[HTML Drag and Drop API](https://developer.mozilla.org/en-US/docs/Web/API/HTML_Drag_and_Drop_API)