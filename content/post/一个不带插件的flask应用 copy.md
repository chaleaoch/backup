---
title: "一个不带插件的flask应用2 -- 自定义migrate"
description: "一个不带插件的flask应用2 -- 自定义migrate"
keywords: "flask,插件"
date: 2022-08-21T01:12:00+08:00
lastmod: 2022-08-21T01:12:00+08:00

categories:
  - default
tags:
  - flask
  - python
# Post's origin author name
#author:
# Post's origin link URL
#link:
# Image source link that will use in open graph and twitter card
#imgs:
# Expand content on the home page
#expand: true
# It's means that will redirecting to external links
#extlink:
# Switch to enabled or disabled comment plugins in this post
#comment:
# enable: false
# Enable table of content
toc: true
# Absolute link for visit
#url: "{{ lower .Name }}.html"
# Sticky post set-top in home page and the smaller nubmer will more forward.
#weight: 1
---

# 起因

[前文](https://chaleaoch.com/post/%E4%B8%80%E4%B8%AA%E4%B8%8D%E5%B8%A6%E6%8F%92%E4%BB%B6%E7%9A%84flask%E5%BA%94%E7%94%A8/ "一个不带插件的flask应用") 介绍了自己写的`flask-redis` 和 `flask-scheduler`扩展. 今天介绍基于`flask` 和 `peewee` 如何实现半自动 migrate.

1. django 默认提供了 migration 功能.
2. django 的 migration 功能在多人开发的模式下会有冲突,经常需要--merge. 并且也不能保证完全避免问题.
3. peewee 提供了 migration 的基础 api
4. 标题和[前文](https://chaleaoch.com/post/%E4%B8%80%E4%B8%AA%E4%B8%8D%E5%B8%A6%E6%8F%92%E4%BB%B6%E7%9A%84flask%E5%BA%94%E7%94%A8/ "一个不带插件的flask应用")>)有介绍, 本项目尝试零插件实现一个 flask 应用.
   基于以上 4 点原因, 决定自己写一个半自动的`migrate`插件. 让冲突出现在`git push`阶段. 人为在代码层面解决冲突.而不是在`make migrate`的时候手忙脚乱.

# peewee 的 migrate api

提供了常见的`增删改字段`和`create table`功能

```python
db.create_tables([table_name])
migrator.add_column('comment_tbl', 'pub_date', pubdate_field)
migrator.rename_column('story', 'pub_date', 'publish_date')
migrator.drop_column('story', 'some_old_field')
migrator.drop_not_null('story', 'pub_date')
migrator.add_not_null('story', 'modified_date')
migrator.alter_column_type('person', 'email', TextField())
migrator.rename_table('story', 'stories_tbl')
migrator.add_index('story', ('pub_date',), False)
```

更多就不粘了. 参考 [文档](http://docs.peewee-orm.com/en/latest/peewee/playhouse.html#supported-operations) 即可.

# 思路及实现

思路非常简单,利用反射和`peewee`的`migrate api`, 自己写一个迁移的`class`(就是下面代码中的`Migrate`) 手动执行 sql 命令就行了.
这里需要注意的是`create_tables` 引入的`model` 不能是代码中的最新版本的`model`. 必须是在刚刚创建这个`model`是的一个快照, 实现起来也很简单, 做一份拷贝就可以了.
否则, 如果某个`model`的字段发生了变化,` create`新表的同时又`add`了一个同样的字段会报错.

```python
from playhouse.migrate import *
def parse_method_name(method_name: str) -> tuple[arrow.arrow.Arrow, int]:
    migrate_version = 0
    method_tuple = method_name.split("_")
    if len(method_tuple) == 2:
        date = method_tuple[1]
    elif len(method_tuple) == 3:
        date = method_tuple[1]
        migrate_version = method_tuple[2]
    else:
        raise Exception(f"Invalid method name: {method_name}")
    return arrow.get(date), int(migrate_version)


@click.command("migrate")
@click.argument("start_version")
@click.argument("end_version")
@with_appcontext
def create_migrations(start_version, end_version):
    start_version = arrow.get(start_version)
    end_version = arrow.get(end_version)
    execute_method_dict = dict[str, arrow.arrow.Arrow | int | Callable]
    execute_methods: list(execute_method_dict) = []
    for method_name, method in inspect.getmembers(Migrate(), predicate=inspect.ismethod):
        if method_name.startswith("migrate_"):
            date, migrate_version = parse_method_name(method_name)
            if start_version < date and date <= end_version:
                execute_methods.append(
                    {
                        "date": date,
                        "migrate_version": migrate_version,
                        "method": method,
                    }
                )
    execute_methods.sort(key=lambda x: (x["date"], x["migrate_version"]))
    for execute_method in execute_methods:
        with flask_db.database.atomic():
            try:
                execute_method["method"]()
            except Exception as e:
                print(f"e:{e},execute_method:{execute_method}")
                raise (e)

class Migrate:
    def __init__(self) -> None:
        self.migrator = PostgresqlMigrator(flask_db.database)
        self.db = flask_db.database
    def migrate_20220815(self):
        self.db.create_tables([AModel,BModel])
    def migrate_20220816(self):
        migrate(self.migrator.add_column("amodel", "field_name", CharField(max_length=255, null=True)))
```

完.
