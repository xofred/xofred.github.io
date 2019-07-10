---
title:  "在 rails+vue+elementui 项目中使用 jsonb"
toc: true
toc_label: "目录"
toc_icon: "cog"
tags: jsonb rails vue
---

*tldr 只要是字符串，vue 都可以直接双向绑定，不管层级多深*

# 在 rails+vue+elementui 项目中使用 jsonb

## 查询语句

```ruby
class Admin::GiftsController < Admin::BaseController
  def index
    gs = Gift.all
    # 【names】为 jsonb 字段
    # 用来保存【中文】【英文】【日文】名称
    gs = gs.where("names->>'cn' LIKE :q OR names->>'en' LIKE :q OR names->>'ja' LIKE :q", q: params[:name]) if params[:name].present?
  end
end
```

## mass assignment 白名单

```ruby
class Admin::GiftsController < Admin::BaseController
  def create
    Gift.create(gift_param)
  end

  def update
    @g = Gift.where(id: params[:id]).first
    @g.update(gift_param)
  end

  private
  def gift_param
    # 【names】为 jsonb 字段
    # 用来保存【中文】【英文】【日文】名称
    params.permit(:tag, { names: [:cn, :en, :ja] }, :img, :position, :coins, :animation, :status, :kind, :festival_id, :description, :description_background)
  end
end
```

## 前台

```vue
<!-- elementui 表格列数据 -->
<el-table-column
  prop="names.cn"
  label="中文名称">
</el-table-column>

<!-- elementui 表单输入框 -->
<el-dialog title="虚拟物品" :visible.sync="addVisible">
  <el-form :model="form">
    <el-form-item label="中文名称">
      <el-input v-model="form.names.cn" auto-complete="off" class="add-input">
      </el-input>
    </el-form-item>
    <el-form-item label="英文名称">
      <el-input v-model="form.names.en" auto-complete="off" class="add-input">
      </el-input>
    </el-form-item>
    <el-form-item label="日文名称">
      <el-input v-model="form.names.ja" auto-complete="off" class="add-input">
      </el-input>
    </el-form-item>
  </el-form>
</el-dialog>

<script>
export default {
  data() {
    gifts: {// 表格数据
      names: {
        cn: '',
        en: '',
        ja: '',
      },
    },
    addVisible: false,
    form: {// 表单数据
      names: {
        cn: '',
        en: '',
        ja: '',
      },
  },

  methods: {
    clearForm() {// 清空表单数据
      this.form = {
        names: {
          cn: '',
          en: '',
          ja: '',
        },
      },
    },
  },
}
</script>
```

## 参考

[Postgresql json like query](https://stackoverflow.com/questions/42918348/postgresql-json-like-query)

[action_controller_overview.html#strong-parameters](https://edgeguides.rubyonrails.org/action_controller_overview.html#strong-parameters)
