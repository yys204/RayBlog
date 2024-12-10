---
title: problem-splitpanes
date: 2024-11-11 21:59:53
tags:
---

记录今天关于解决splitpanes在两个div盒子时，其中有个盒子为iframe，造成拖拽失效的解决方法。

## 问题发现

今天心血来潮，在天天当牛做马的日子里，突然想去增加项目中代码编辑器，代码框和运行结果页面两个div增加一个左右拖动的功能。这里，我使用了leader推荐的splitpanes，这个真的超级好用，我前端菜鸟一看就会的那种。[splitpanes链接](https://github.com/antoniandre/splitpanes),推荐在项目中有实现拖动的友友们去试试。

### 问题

当我按照使用文档引用完包，并按照示例添加进自己的代码时，发现一个问题，我的右边代码运行结果的div拖不动，只能在我右边的代码块拖动。附上我的代码：

```vue
<splitpanes
      class="w-full h-full"
    >
      <pane>
        <span>
          <div class="w-full h-full bg-black">
            <div class="w-full h-[32px] flex justify-between">
              <a-space :size="0">
                <template #split>
                  <a-divider type="vertical" />
                </template>
                <a-button type="link" @click="changeEditor('template')" class="text-[16px]">
                  template
                </a-button>
                <a-button type="link" @click="changeEditor('script')" class="text-[16px]">
                  script
                </a-button>
              </a-space>
              <a-space :size="0">
                <template #split>
                  <a-divider type="vertical" />
                </template>
                <a-button type="link" @click="handleFormat"><Brackets :size="16" /></a-button>
                <a-button type="link"><Undo2 :size="16" /></a-button>
                <a-button type="link" @click="handlePlay"><Play :size="16" /></a-button>
              </a-space>
            </div>
            <div
              class="w-full h-[calc(100%-32px)]"
              ref="monacoElTemplate"
              v-show="currentEditor === 'template'"
            ></div>
            <div
              class="w-full h-[calc(100%-32px)]"
              ref="monacoElScript"
              v-show="currentEditor === 'script'"
            ></div>
          </div>
        </span>
      </pane>
      <pane>
        <div class="w-full h-full">
          <iframe
            ref="editorIframe"
            frameborder="0"
            allow="geolocation *;"
            :src="iframeUrl"
            class="w-full h-full "
          >
          </iframe
        ></div>
      </pane>
    </splitpanes>
```

### 问题定位

当我重复向右拖动时，我发现每当我移入到iframe的时候，我的鼠标他就会从小手变成一个箭头，这代表着移入到iframe中的时候肯定有什么事件触发，将splitpanes本身的事件阻止了。

### 解决过程

1. 一如既往，先百度，然后github。我leader教我的，(一般你遇到的问题，90%别人也遇到过)。看github仓库中issues中的提问，好家伙，还真找到一个。
![一张图片](/images/splitspanes-issues.png)

2. 点进去查看：

![一张图片](/images/splitspanes-issues-detail.png)

大概意思就是：
因为不是同一个页面，在 iframe 上拖动和释放时可能无法捕获事件 mouseup，要用其他方法去处理。在拖动的时候，solitspanes容器会获取splitpanes--dragging这个类，在iframe上去添加一个:before改变属性，添加一个蒙层，并且将iframe 设置为 pointer-events: none。我是英语智障，英语不太好，不知道翻译的对不。

3. 按照他的提示我在iframe:before上添加了一些属性，并且在splitpanes上添加了鼠标移入移出事件来控制

```vue
 <splitpanes
      class="w-full h-full default-theme"
      :push-other-panes="false"
      @mousedown="handleMouseDown"
      @mouseup="handleMouseUp"
    >
      <pane>
        <span>
          <div class="w-full h-full bg-black">
            <div class="w-full h-[32px] flex justify-between">
              <a-space :size="0">
                <template #split>
                  <a-divider type="vertical" />
                </template>
                <a-button type="link" @click="changeEditor('template')" class="text-[16px]">
                  template
                </a-button>
                <a-button type="link" @click="changeEditor('script')" class="text-[16px]">
                  script
                </a-button>
              </a-space>
              <a-space :size="0">
                <template #split>
                  <a-divider type="vertical" />
                </template>
                <a-button type="link" @click="handleFormat"><Brackets :size="16" /></a-button>
                <a-button type="link"><Undo2 :size="16" /></a-button>
                <a-button type="link" @click="handlePlay"><Play :size="16" /></a-button>
              </a-space>
            </div>
            <div
              class="w-full h-[calc(100%-32px)]"
              ref="monacoElTemplate"
              v-show="currentEditor === 'template'"
            ></div>
            <div
              class="w-full h-[calc(100%-32px)]"
              ref="monacoElScript"
              v-show="currentEditor === 'script'"
            ></div>
          </div>
        </span>
      </pane>
      <pane>
        <div class="w-full h-full">
          <iframe
            ref="editorIframe"
            frameborder="0"
            allow="geolocation *;"
            allowTransparency="true"
            :src="iframeUrl"
            class="w-full h-full iframe"
          >
          </iframe
        ></div>
      </pane>
    </splitpanes>
```

```js
const handleMouseDown = () => {
    document.querySelector('iframe').style.pointerEvents = 'none';
  };

  const handleMouseUp = () => {
    document.querySelector('iframe').style.pointerEvents = 'auto';
  };
```

```css
/* 遮罩层样式 */
  .iframe::after {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    z-index: 10;
    background-color: rgba(255, 255, 255, 0); /* 透明背景 */
    pointer-events: none; /* 不捕获鼠标事件 */
  }
```
最后，也是解决了这个问题
