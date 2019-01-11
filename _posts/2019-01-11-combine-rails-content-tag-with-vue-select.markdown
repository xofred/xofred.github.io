---
title:  "用 vue-sélect 和 Rails conten_tag 实现下拉框过滤"
toc: true
toc_label: "目录"
toc_icon: "cog"
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
