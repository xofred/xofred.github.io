---
title:  "MySQL MATCH AGAINST On Multiple Columns"
tags: mysql backend
toc: true

---

*MATCH (column) AGAINST (keyword)*

### Before use

> `InnoDB` tables require a `FULLTEXT` index on all columns of the [`MATCH()`](https://dev.mysql.com/doc/refman/8.0/en/fulltext-search.html#function_match) expression to perform boolean queries

```ruby
# migration
class AddFulltextIndexToCatalogsOnAncestry < ActiveRecord::Migration[5.1]
  def change
    add_index :catalogs, :ancestry, {
      name: 'fulltext_index_catalogs_on_ancestry',
      type: :fulltext,
      comment: "for MATCH AGAINST purpose"
    }
  end
end
```

### VS other query function

LIKE

```ruby
Catalog.where(ids.map{|id| "ancestry LIKE \'\%#{id}\%\'"}.join(' OR ')).count
```

>    (30.3ms)  SELECT COUNT(*) FROM `catalogs` WHERE (ancestry LIKE '%16994%' OR ancestry LIKE '%20029%' OR ancestry LIKE '%20080%' OR ancestry LIKE '%3000%' OR ancestry LIKE '%3083%' OR ancestry LIKE '%3941%' OR ancestry LIKE '%3947%')
> => 3170

REGEXP

```ruby
Catalog.where('ancestry REGEXP ?', ids.join('|')).count
```

>    (86.6ms)  SELECT COUNT(*) FROM `catalogs` WHERE (ancestry REGEXP '16994|20029|20080|3000|3083|3941|3947')
> => 3170

MATCH AGAINST

```ruby
Catalog.where('MATCH ancestry AGAINST (?)', ids.join(' ')).count
```

>    (9.9ms)  SELECT COUNT(*) FROM `catalogs` WHERE (MATCH ancestry AGAINST ('16994 20029 20080 3000 3083 3941 3947'))
> => 3170

### Reference

[MySQL 8.0 Reference Manual  /  ...  /  Boolean Full-Text Searches](https://dev.mysql.com/doc/refman/8.0/en/fulltext-boolean.html)

[MySQL Like multiple values](http://stackoverflow.com/questions/4172195/ddg#7566973)

[MySQL LIKE operator Vs MATCH AGAINST](https://stackoverflow.com/questions/14776350/mysql-like-operator-vs-match-against#14776444)
