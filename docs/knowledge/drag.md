# React 中使用拖拽

## 拖拽事件

拖拽事件涉及到两个步骤：拖起、放下，事件可以分为两类：

- 一类是被拖起的元素所触发的事件，即拖起事件；
- 一类则是拖起放下后，所在区域的目标元素触发的事件，即拖放事件。

可参考[MDN 拖拽操作](https://developer.mozilla.org/zh-CN/docs/Web/API/HTML_Drag_and_Drop_API/Drag_operations)

### 拖起事件（拖拽元素）

`拖拽元素`自身的事件。

- `onDragStart`：当鼠标按下并且开始移动拖拽元素后，触发此事件，整个拖拽周期只触发一次；
- `onDrag`：拖拽过程中会不断触发此事件；
- `onDragEnd`：鼠标松开结束拖拽后，会触发此事件的执行，整个拖拽周期只触发一次。

```html
<!--  拖拽元素  -->
<div
  onDragStart="{onDragStart}"
  onDrag="{onDrag}"
  onDragEnd="{onDragEnd}"
></div>
```

### `拖放事件`（目标元素）

当拖拽的元素拖到一个目标元素上时，`目标元素` 会触发以下事件：

- `onDragEnter`：拖拽元素移动到目标元素时，触发此事件；
- `onDragOver`：拖拽元素停留在目标元素时，会持续触发此事件；
- `onDragLeave`：拖拽元素离开目标元素时（没有在目标元素上放下），触发此事件；
- `onDrop`：拖拽元素在目标元素放下（松开鼠标），触发此事件。

```html
<!--  放置元素  -->
<div
  onDragEnter="{onDragEnter}"
  onDragOver="{onDragOver}"
  onDragLeave="{onDragLeave}"
  onDrop="{onDrop}"
></div>
```

## 数据交换

在拖起元素时，需要携带一些数据，在拖放到目标元素后，将数据交给目标元素做处理。HTML5 拖拽提供了数据交换的方式：`event.dataTransfer`

```js
//  拖拽元素
onDragStart = (e) => {
  e.dataTransfer.setData("msg", "this is test msg !");
};

// 放置元素
onDrop = (e) => {
  e.dataTransfer.getData("msg");
};
```

## 自定义原生拖动图像（拖拽效果）

原生默认的拖拽效果是拖拽元素的一个半透明的预览图，有时候我们需要自定义拖拽时的预览图来实现我们的需求，HTML5 拖拽 也提供了自定义方式：`event.dataTransfer.setDragImage(element, xOffset, yOffset);`

```js
onDragStart = (e: React.DragEvent, item: IDocListItem) => {
  // 这里要使用event.currentTaget才可以获取到chilren，event.taget获取不到
  // 这里children[0]可换成其他已挂载的element
  e.dataTransfer.setDragImage(e.currentTarget.children[0], 0, 0);
};
```

**注意**：我们自定义的元素必须是一个已经挂载到 DOM 树上的元素，否则自定义图像会失败，这一点在 React、Vue 框架中要特别注意，虚拟 DOM 对象用在这里不生效。

## 注意事项

1. 被拖拽的元素要设置`draggable = true`属性，可参考[MDN 拖拽操作](https://developer.mozilla.org/zh-CN/docs/Web/API/HTML_Drag_and_Drop_API/Drag_operations)
2. 放置元素要在`onDragOver`事件里禁用默认操作，即`event.preventDefault()`,否则自定义`onDrop`事件将不会被触发。

## 补充功能（拖拽上传）

当文件推拽到指定元素上是同样会触发上述的拖拽事件，可以在`onDrop`事件中获取到拖拽进来的文件

```html
<div
  className="{style.right}"
  onDragOver="{onDragOver}"
  onDrop="{onDrop}"
></div>
```

```js
// 禁用onDragOver的默认操作，可以阻止推拽进来的文件被浏览器直接打开或者触发下载文件，并且允许自定义`onDrop`事件会被触发。
onDragOver = (e) => {
  e.preventDefault();
};

onDrop = (e) => {
  const file = e.dataTransfer.files[0];
};
```
