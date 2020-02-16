---
title:  "纯 CSS 居中，以及超出高度自动显示滚动条"
tags: css frontend

---

*css 真香*

### 代码
```css
.modal-help-me-find {
  width: 800px;

  /* 居中相关 */
  position: fixed;
    top: 50%;
    left: 70%; /* 左边有排 sidebar 写死了位置，所以不是 50% */
  transform: translate(-50%, -50%);

  /* 超出高度自动显示滚动条 */
  overflow-y: auto;
  max-height: calc(100% - 100px);
}
```

### Reference

[Pure CSS Dynamic Sized Centered Modals](http://lynn.io/2014/02/22/modalin/)

[Add scrollbar to modal window](https://stackoverflow.com/questions/44288362/add-scrollbar-to-modal-window)
