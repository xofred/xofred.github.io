---
title:  "在 erb 表格里使用 vue 定制单元格实现列表页批量更新"
---

*tl; dr: 思路是，将 `ruby` 数组循环下标作为`v-model` 的数组下标。这样可以使用 `vue`的双向绑定了*

由于运营后台有些较复杂的 rails helper，同时有较重前端的交互。在有限的时间里，只能快粗猛了

```erb
<!-- 运营后台某个 model 的列表页 -->
<div id="app">
  <!-- 去添加页面 -->
  <div class="row">
    <div class="col-12">
      <h2>
        Some Model List
        <%= link_to new_some_model_path, class: 'btn btn-success no-border pull-right' do %>
          <i class="icon-plus"></i>
          Add
        <% end %>
      </h2>
    </div>
  </div>
    
  <hr>
    
  <!-- 筛选 -->
  <div class="row mb-3">
    <div class="col-12">
      <%= form_tag({ action: :index }, method: :get, class: 'form-inline') do %>
        <label>Some Relation</label>
        <%= select_tag "some_relation_id",
            options_from_collection_for_select(@some_relations, "id", "name", selected: params[:some_relation_id]),
            prompt: 'All',
            class: 'form-control mr-2' %>

        <label>Some Select Field</label>
        <%= select_tag "some_select_field",
            options_for_select(@options, params[:some_select_field]),
            prompt: 'All',
            class: 'form-control mr-2' %>

        <label>Some Search Field</label>
        <%= text_field_tag 'q', params[:q] || '' %>

        <button type="submit" class="btn btn-primary" data-disable-with="Searching...">
          <span><i class="cui-magnifying-glass"></i> Search</span>
        </button>
      <% end %>
    </div>
  </div>

  <!-- 表格展示 -->
  <div class="row">
    <div class="col-lg-12">
      <% if @some_models.any? %>
        <!-- Toggle 是一个很简单的开关控件，代码和样式在另外的地方，事先已经引入 -->
        <!-- 另外，由于 Toggle 用了 es6 语法，文件名要改为 js 以外的东西，例如 jsx -->
        <!-- 这样 rails 预编译静态文件时就会跳过这个文件了，不然会无法解析，报语法错误 -->
        <Toggle v-model="editable" v-on:change="updateSomeModels"></Toggle>{{ toggleText }}
        <table class="table table-responsive-sm table-bordered table-striped table-sm">
          <thead>
            <tr>
              <!-- 省略 th 部分，关键是下面的单元格 -->
            </tr>
          </thead>
          <tbody>
            <% @some_models.each_with_index do |some_model, i| %>
              <% relation = some_model.some_model_relation %>
              <tr>
                <td><%= some_model.some_display_field %></td>
                <td><%= some_complex_helper(some_model) %></td>
                <td>
                  <% if relation.present? %>
                    <%= link_to relation.name, "/some_model_relations/#{relation.id}/edit", class: 'btn btn-link' %>
                  <% else %>
                    N/A
                  <% end %>
                </td>
                <td><%= some_model.some_more_display_field %></td>
                <!-- 之前还真没想过能这么用 -_-! -->
                <td>
                  <select v-model="someModelDatas[<%= i %>].some_select_field" :disabled="!editable">
                    <option v-for="k in Object.keys(options)">{{ k }}</option>
                  </select>
                </td>
                <td>
                  <select v-model="someModelDatas[<%= i %>].another_select_field" :disabled="!editable">
                    <option v-for="k in Object.keys(anotherOptions)">{{ k }}</option>
                  </select>
                </td>
                <td><%= some_model.some_more_display_field %></td>
                <td>
                  <input
                    type="number"
                    min="0"
                    id="some_model[some_input_field]"
                    v-model="someModelDatas[<%= i %>].some_input_field"
                    :disabled="!editable"
                    >
                  </input>
                </td>
                <td><%= link_to 'Edit', edit_some_model_path(some_model), class: 'btn btn-link' %></td>
              </tr>
              <% end %>
          </tbody>
        </table>

        <%= paginate @some_models, views_prefix: 'layouts/v2' %>
      <% else %>
        <p>No Record Matches</p>
      <% end %>
    </div>
  </div>
</div>

<script>
    var app = new Vue({
      el: '#app',

      data: {
        editable: false,
        errors: [],
        options: <%= raw @options.to_json %>,
        anotherOptions: <%= raw @another_options.to_json %>,
        someModelDatas: <%= raw @some_models.to_json %>,
      },

      computed: {
        toggleText: function() {
          var act = this.editable === false ? "edit" : "save"
          return("Click to " + act)
        },
      },

      methods: {
        updateSomeModels: function() {
          if (this.editable) {
            return
          }

          // json 传到后端前要做序列化，要指明 contentType
          var updateAllRequest = $.ajax({
            url: '<%= "#{update_all_some_models_path}" %>',
            method: "POST",
            data: JSON.stringify({ SomeModels: this.someModelDatas}),
            dataType: "json",
            contentType: "application/json;charset=utf-8",
          });

          updateAllRequest.done(function( result ) {
            alert( result['msg'] );
            location.reload();
          });

          updateAllRequest.fail(function( result ) {
            alert( "Request failed: " + result['msg'] );
          });
        },
      },
  })
</script>
```

```ruby
# 后端部分
# 省略 index、new、edit、create、update 等部分，只关注列表页批量更新
class SomeModelsController < V2::ApplicationController
  def update_all
    params[:SomeModels].each do |f|
      @some_model = SomeModel.find(f['id'])
      if !@some_model.update(
          some_select_field: f['some_select_field'],
          another_select_field: f['another_select_field'],
          some_input_field: f['some_input_field'].to_i
      )
        return render json: { status: 403, msg: "#{@some_model.some_unique_field}: #{@some_model.errors.full_messages}" }
      end
    end

    return render json: { status: 200, msg: 'Update all successfully.' }
  end
end
```
