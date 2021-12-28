---
title:  "Postgres Jsonb Partial Index For Not Null Querying"
tags: postgres backend rails heroku
toc: true

---

*It takes extra steps to make the partial index work*

### Before

```
\d webhooks
```

```
                                        Table "public.webhooks"
   Column   |            Type             | Collation | Nullable |               Default                
------------+-----------------------------+-----------+----------+--------------------------------------
 id         | bigint                      |           | not null | nextval('webhooks_id_seq'::regclass)
 payload    | jsonb                       |           |          | 
 created_at | timestamp without time zone |           | not null | 
 updated_at | timestamp without time zone |           | not null | 
Indexes:
    "webhooks_pkey" PRIMARY KEY, btree (id)
    "index_webhooks_on_payload" gin (payload) WHERE (payload ->> 'kind'::text) IS NOT NULL AND (payload ->> 'recording'::text) IS NOT NULL
```

```sql
EXPLAIN ANALYZE SELECT "webhooks".* FROM "webhooks"
  WHERE (payload ->> 'kind' IS NOT NULL)
  AND (payload ->> 'recording' IS NOT NULL);
```

```
                                                  QUERY PLAN                                                  
--------------------------------------------------------------------------------------------------------------
 Seq Scan on webhooks  (cost=0.00..428.85 rows=1706 width=1676) (actual time=0.119..82.476 rows=1722 loops=1)
   Filter: (((payload ->> 'kind'::text) IS NOT NULL) AND ((payload ->> 'recording'::text) IS NOT NULL))
   Rows Removed by Filter: 1
 Planning Time: 0.433 ms
 Execution Time: 82.681 ms
(5 rows)
```

It's still `Seq Scan`. The `index_webhooks_on_payload` index is useless😒

### After

Turns out, the `? field_name` and `field_name IS NOT NULL` are both required, if we want the `field_name IS NOT NULL` condition to use indexes.

```sql
CREATE INDEX idx ON webhooks ((payload ->> 'kind'))
  WHERE (payload ? 'kind' AND payload -> 'kind' IS NOT NULL);
```

```sql
CREATE INDEX idx2 ON webhooks ((payload ->> 'recording'))
  WHERE (payload ? 'recording' AND payload -> 'recording' IS NOT NULL);
```

```sql
EXPLAIN ANALYZE SELECT "webhooks".* FROM "webhooks"
  WHERE (payload ? 'kind' AND payload -> 'kind' IS NOT NULL)
  AND (payload ? 'recording' AND payload -> 'recording' IS NOT NULL);
```

```
                                                                                QUERY PLAN                                                                                 
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 Bitmap Heap Scan on webhooks  (cost=25.35..29.37 rows=1 width=1676) (actual time=1.432..2.572 rows=1722 loops=1)
   Recheck Cond: ((payload ? 'kind'::text) AND ((payload -> 'kind'::text) IS NOT NULL) AND (payload ? 'recording'::text) AND ((payload -> 'recording'::text) IS NOT NULL))
   Heap Blocks: exact=403
   ->  BitmapAnd  (cost=25.35..25.35 rows=1 width=0) (actual time=1.224..1.226 rows=0 loops=1)
         ->  Bitmap Index Scan on idx  (cost=0.00..4.36 rows=17 width=0) (actual time=0.343..0.344 rows=1722 loops=1)
         ->  Bitmap Index Scan on idx2  (cost=0.00..20.74 rows=17 width=0) (actual time=0.820..0.821 rows=1722 loops=1)
 Planning Time: 0.232 ms
 Execution Time: 2.780 ms
(8 rows)
```

It uses `Bitmap Heap Scan` instead. Yay🎉

Let's remove the testing indexes `idx` and `idx2`, and add a formal migration later. 
```sql
DROP INDEX idx;
DROP INDEX idx2;
```

### Migration and Deploy

```ruby
class AddIndexToWebhooks < ActiveRecord::Migration[5.1]
  def change
    add_index :webhooks, "((payload -> 'kind'))", {
      name: 'index_webhooks_on_payload_kind',
      where: "payload ? 'kind' AND payload -> 'kind' IS NOT NULL",
    }

    add_index :webhooks, "((payload -> 'recording'))", {
      name: 'index_webhooks_on_payload_recording',
      where: "payload ? 'recording' AND payload -> 'recording' IS NOT NULL",
    }
  end
end
```

```shell
heroku run rake --trace db:migrate
```

```shell
heroku restart
```

And check if work on production.

```shell
heroku rails c
```

```ruby
Webhook
  .where("payload ? 'kind' AND payload -> 'kind' IS NOT NULL")
  .where("payload ? 'recording' AND payload -> 'recording' IS NOT NULL")
  .explain
```

```
                                                                                QUERY PLAN
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 Bitmap Heap Scan on webhooks  (cost=14.27..16.28 rows=1 width=1679)
   Recheck Cond: ((payload ? 'kind'::text) AND ((payload -> 'kind'::text) IS NOT NULL) AND (payload ? 'recording'::text) AND ((payload -> 'recording'::text) IS NOT NULL))
   ->  BitmapAnd  (cost=14.27..14.27 rows=1 width=0)
         ->  Bitmap Index Scan on index_webhooks_on_payload_kind  (cost=0.00..2.07 rows=17 width=0)
         ->  Bitmap Index Scan on index_webhooks_on_payload_recording  (cost=0.00..12.15 rows=17 width=0)
(5 rows)
```

Production works, too.

### Reference

- [Optimizing Postgres JSONB query with not null constraint](https://stackoverflow.com/a/42528940/2180787)

- [add index on jsonb field](https://stackoverflow.com/questions/34041821/add-index-on-jsonb-field)

- [Running Rake Commands \| Heroku Dev Center](https://devcenter.heroku.com/articles/rake)
