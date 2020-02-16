---
title:  "Vue 已有 data 对象(已初始化的 vue 实例)动态添加字段，以及阻止点击默认行为"
toc: true
toc_label: "目录"
toc_icon: "cog"
tags: vue frontend

---

*这算是 vue 的坑了吧*

### 已有 data 对象(已初始化的 vue 实例)动态添加字段
```javascript
var app = new Vue({
  el: "#app",
  data: {
    // 这里数组里面的字段已经定义好了，主要是【数据相关】的数据结构
    // 要加些状态的话怎么办呢，例如【显示/隐藏】【可编辑/不可编辑】
    // (加到 API 是可以，但明明是前端的东西，在 API 加有点怪)
    help_me_finds: [],
  },
  ready: function() {
    this.getHelpMeFinds();
  },
  methods: {
    getHelpMeFinds() {
      var self = this;

      $.ajax({
        url: '/help_me_finds',
        method: 'GET',
        success: function(res) {
          self.help_me_finds = res.help_me_finds;

          for (var i = 0; i < self.help_me_finds.length; i++) {
            // Vue 官方推荐的解决方案
            self.help_me_finds[i] = Object.assign({}, self.help_me_finds[i], {
              showCustomerModal: false,
              showSalesModal: false,
              todoEditable: false,
              todoEditable: false,
            })
          }
        },
      });
    },
  },
})
```

### 阻止点击默认行为

例如 `form` 的 `submit`，或 `a` 标签的跳转。很多时我们想执行自己写的业务逻辑，并不想浏览器触发 html 默认行为

```html
<!-- 阻止 a 标签页内跳转 -->
<!-- 例如这里我们想打开新窗口跳转 -->
<a href="help_me_find.to_do_link"
   v-if="!help_me_find.todoEditable"
   @click.prevent="openNewWindowToBasecamp(help_me_find.to_do_link)">{{ help_me_find.to_do_link }}
</a>
```

### Reference

[Reactivity in Depth](https://v1.vuejs.org/guide/reactivity.html)

[How to add a new property to vue (object) data? #3340](https://github.com/vuejs/vue/issues/3340)

[Vuejs - automatic event.preventDefault](http://stackoverflow.com/questions/49144791/ddg#49146048)
