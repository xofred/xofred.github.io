---
title:  "Gotcha with Activerecord Import and Kiba"
tags: gem

---

*Sometimes, convention might become a problem🤷‍♂️*

Let's say we have a fetch job using [Kiba](https://github.com/thbar/kiba). It fetches records from DB:FROM to DB:TO, as long as there are newer records from DB:FROM. By 'newer' records, which means bigger `updated_at` value.

Later, we add an import job using [Activerecord-Import](https://github.com/zdennis/activerecord-import), which might perform before the fetch job. Recently, we found some of our fetch jobs are not functioning anymore, but the `updated_at` were newer, though. After some digging, we finally know the [reason](https://github.com/zdennis/activerecord-import#activerecord-timestamps).

By default, Activerecord-Import will update the `updated_at` column. This means if the import job is performed before a fetch job, the `updated_at` will get accidentally updated. Then the fetch job will consider no newer records. So either we make sure the import job is performed after the fetch job, or we tell Activerecord-Import to stay out of business, by using `timestamps: false`.
```diff
-      CrmCustomer.import records
+      CrmCustomer.import records, timestamps: false
```
