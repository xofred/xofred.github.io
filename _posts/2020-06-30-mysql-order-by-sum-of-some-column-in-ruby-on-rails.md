---
title:  "MySQL Order by Sum of Some Column in Ruby on Rails"
tags: mysql ruby-on-rails

---

*No, u can't just `.order('SUM(column_name)')`, I tried*

Let's say we have a `products` table, and a `monthly_product_sales` table. We want to select all the columns from `products`, but `ORDER BY` the `SUM` of `monthly_product_sales.quantity`. Here's how we do it.

```ruby
def sales_sum_sql
    "
    SELECT SUM(monthly_product_sales.quantity) AS sum_monthly_product_sales_quantity,
			`product_id` AS product_id
    FROM `products` INNER JOIN `monthly_product_sales`
			ON `monthly_product_sales`.`product_id` = `products`.`id`
    WHERE `products`.`active` = 1
    AND (the_date BETWEEN '#{2.years.ago.utc.strftime('%Y-%m-%d %H:%M:%S')}'
      AND '#{Time.current.utc.strftime('%Y-%m-%d %H:%M:%S')}')
    GROUP BY `product_id`
    "
end
```

The `sales_sum_sql` above will produce something like this.

```
+------------------------------------+------------+
| sum_monthly_product_sales_quantity | product_id |
+------------------------------------+------------+
|                                 91 |     100011 |
|                                 29 |     100012 |
|                                 31 |     100013 |
|                                 30 |     100014 |
|                               1204 |     100022 |
|                                401 |     100023 |
|                                354 |     100024 |
|                               1682 |     100026 |
|                                813 |     100028 |
|                               8174 |     100029 |
```

Then our `products` table can `JOIN` it and `ORDER BY` its `SUM` column, which is `sum_monthly_product_sales_quantity`.

```ruby
def find_products
  @products = Product.all

  # Actully a lot of filtering happen here.
  # But in this article let's focus on the order part.

  ['asc', 'desc'].each do |order_by_direction|
    if params[:sortOption] == "sold_qty_#{order_by_direction}"
      @products = @products
        .joins("INNER JOIN (#{sales_sum_sql}) AS sales_sums ON sales_sums.product_id = products.id")
        .order("sum_monthly_product_sales_quantity #{order_by_direction}")
    end
  end

  @products = @products.page(params[:page]).per(params[:per])
end
```

The ultimate sql looks like this. It will produce 50 `products` records, ordered by the way we want in the beginning of this article.

```mysql
SELECT  DISTINCT `products`.* FROM `products` INNER JOIN (
    SELECT SUM(monthly_product_sales.quantity) AS sum_monthly_product_sales_quantity, `product_id` AS product_id
    FROM `products` INNER JOIN `monthly_product_sales` ON `monthly_product_sales`.`product_id` = `products`.`id`
    WHERE `products`.`active` = 1
    AND (the_date BETWEEN '2018-06-30 06:50:49'
      AND '2020-06-30 06:50:49')
    GROUP BY `product_id`
    ) AS sales_sums ON sales_sums.product_id = products.id WHERE `products`.`active` = 1 AND `products`.`is_value_pack` = 0 AND `products`.`used_real_time_qty` = 0 ORDER BY sum_monthly_product_sales_quantity desc LIMIT 50 OFFSET 0
```

### Reference

https://stackoverflow.com/questions/1309841/how-to-order-by-a-sum-in-mysql#1309848
