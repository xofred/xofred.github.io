---
title:  "用 vue-sélect 和 Rails conten_tag 实现下拉框过滤"
toc: true
toc_label: "目录"
toc_icon: "cog"
tags: rails vue vue-select
---

*tldr：思路还是用 Rails 全栈那一套，只是没有 jquery 那么啰嗦*

### controller

```ruby
# controller

before_action :get_options

def new
    @dummy = Dummy.new
end

def edit
    @dummy = Dummy.where(id: params[:id]).first
end

def get_options
    @options = AnotherDummy.all.select("id, name")
end
```

### application.js

```javascript
// application.js

// 注意引入顺序，先引入 vue
//= require v2/vue.js
//= require v2/vue-select.js

Vue.component('v-select', VueSelect.VueSelect)
```

### view

```erb
<!-- view -->

<div id="app">
    <!-- 数据需要塞到页面，不然 js 无法直接解释 @options @dummy 的东西 -->
    <%= content_tag 'options', nil, data: @options.to_json %>
    <%= content_tag 'dummy', nil, data: @dummy.to_json %>

    <!-- 还是以 form 提交这种 -->
    <%= simple_form_for @dummy, do %>
    	<v-select v-model="dummy" :options="options" label="name"></v-select>
    	<!-- 用隐藏的 input field 来保存数据 -->
        <input type="hidden" name="dummy[another_dummy_id]" :value="option.id" />
    <% end %>
</div>

<script>
  // 告诉 vue，不要把 Rails content_tag 生成的标签，理解成他家 component 的 template
  Vue.config.ignoredElements = ['options', 'dummy'];

  var vm = new Vue({
    el: '#app',

    data: {
      // 还是需要 jquery 😂
      options: JSON.parse($('options').attr("data")),
      dummy: JSON.parse($('dummy').attr("data")),
    },
  });
</script>
```

### Reference

[Vue ignore custom component tag](https://stackoverflow.com/questions/45040689/vue-ignore-custom-component-tag)

[vue-select.js](https://sagalbot.github.io/vue-select/)

### 延伸问题

#### Q：我家静态文件没放 cdn

A：一般这种 js 库都有 cdn 地址，在 `layout/application.html.erb` 引用即可

#### Q：我家后台已经加了一吨 js 库，能否按需加载

A：把引用 js 库 cdn 地址这句放在需要的 view 页面即可

#### Q：我家 options 有几百上千个，全取出来有点不科学

A：那 controler 要单独弄个路由，弄成远程可以搜索，最后 render json。然后前端 ajax 请求数据

#### Q：这个 options 我在别的页面也想用，不想复制粘贴

A：那考虑 vuex 那套东西了，至少 action（前端拿数据） 和 mutation（前端存数据） 可以共用
