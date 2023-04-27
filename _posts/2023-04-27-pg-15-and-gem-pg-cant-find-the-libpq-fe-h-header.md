---
title:  "pg 15 and gem pg Can't find the 'libpq-fe.h header"
tags: mac postgres pg gem

---

*The `--with-pg-config` option in gem install is the key*

I download the [pg 15 from EDB](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads). During the install wizard, it will tell where pg  stuffs will be as following:

> Installation Directory: /Library/PostgreSQL/15
>
> Server Installation Directory: /Library/PostgreSQL/15
>
> Data Directory: /Library/PostgreSQL/15/data
>
> Database Port: 5432
>
> Database Superuser: postgres
>
> Operating System Account: postgres
>
> Database Service: postgresql-15
>
> Command Line Tools Installation Directory: /Library/PostgreSQL/15
>
> pgAdmin4 Installation Directory: /Library/PostgreSQL/15/pgAdmin 4
>
> Stack Builder Installation Directory: /Library/PostgreSQL/15
>
> Installation Log: /tmp/install-postgresql.log

As the [best answer from stack overflow](http://stackoverflow.com/questions/6040583/ddg#6040822) suggests, I must specify the `pg_config`. In my case, it's in the ` /Library/PostgreSQL/15`:

```shell
gem install pg -v '1.2.3' -- --with-pg-config=/Library/PostgreSQL/15/bin/pg_config
```

Problem solved.

> gem install pg -v '1.2.3' -- --with-pg-config=/Library/PostgreSQL/15/bin/pg_config
> Building native extensions with: '--with-pg-config=/Library/PostgreSQL/15/bin/pg_config'
> This could take a while...
> Successfully installed pg-1.2.3
> Parsing documentation for pg-1.2.3
> Installing ri documentation for pg-1.2.3
> Done installing documentation for pg after 2 seconds
> 1 gem installed

And I continue `bundle`:

> ...
>
> Using pg 1.2.3
> ...
> Bundle complete! 18 Gemfile dependencies, 75 gems now installed.
> Use `bundle info [gemname]` to see where a bundled gem is installed.

Reference:

[Can't find the 'libpq-fe.h header when trying to install pg gem](http://stackoverflow.com/questions/6040583/ddg#6040822)

[Open source PostgreSQL packages and installers from EDB](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads)
