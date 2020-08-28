---
title:  "Generate Factory Bot Model For Legacy Project"
tags: test gem factory_bot rails

---

*Solving a legacy issue, kind of*

Feel free to [fork me on the GitHub](https://gist.github.com/xofred/b1e313bafa1f529f6244390ff990f4d9).

```ruby
# Solve https://github.com/thoughtbot/factory_bot_rails/issues/63
# Original credit https://stackoverflow.com/a/50319158/2180787

@run_command        = true
@force              = true
@columns_to_ignore  = %w[id created_at update_at]
@tables_to_ignore = %w[schema_migrations ar_internal_metadata]
tables = ActiveRecord::Base.connection.tables.reject{|t| (@tables_to_ignore || []).include?(t)}

tables.each do |table|
  begin
    klass = table.singularize.camelcase.constantize
    command = "rails g factory_bot:model #{klass.to_s} #{klass.columns.reject do |c|
      (@columns_to_ignore || []).include?(c.name)
    end.map do |d|
      "#{d.name}:#{d.sql_type == 'jsonb' ? 'json' : d.type}"
    end.join(' ')}"
    command << ' --force' if @force
    puts command
    puts %x{#{command}} if @run_command
    puts (1..200).to_a.map{}.join('-')
  rescue NameError # uninitialized constant SomeTableWithoutModelDefinition
    next
  end
end
```

### Reference
- [Can FactoryBot generate factories after your models have been created?](https://stackoverflow.com/questions/11702265/can-factorybot-generate-factories-after-your-models-have-been-created)
- [Feature Request: Allow creation of factory from existing model #63](https://github.com/thoughtbot/factory_bot_rails/issues/63)
