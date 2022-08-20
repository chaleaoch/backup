---
title: "一个不带插件的flask应用"
description: "一个不带插件的flask应用"
keywords: "flask,插件"
date: 2022-08-19T08:41:00+08:00
lastmod: 2022-08-19T08:41:00+08:00

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

事情的起因是这样的, 曾经维护过一个别人做的 flask 项目,引用了大量的插件, 在维护过程中需要调查插件的源码来解决问题, 发现有的插件功能很单一,但是却写的很复杂(因为开源插件要考虑各种实际使用场景). 有时间精力还不如自己写一个. 恰好得到一个从零开始的项目. 决定尝试做一个零插件的 flask 项目. 目前来看效果还不错.

> 需要说明的是: 这里的零插件是指零 flask 插件,不是不用任何 python 插件.

# 项目构成

一个很简单的 dashboard, 主要用到如下组件

```txt
python = "~3.10"
Flask = "~2.1.1"
python-dotenv = "~0.20.0"
pydantic = "~1.9.0"
requests = "~2.27.1"
peewee = "~3.14.10"
arrow = "~1.2.2"
psycopg2 = "~2.9.3"
wtf-peewee = "~3.0.4"
APScheduler = "~3.9.1"
redis = "~4.3.3"
SQLAlchemy = "~1.4.37"
gunicorn = "~20.1.0"
Cython = "^0.29.30"
```

可能用到的 flask 插件是: `flask-redis`, `flask-sqlalchemy`, `flask-peewee`, `flask-pydantic`

## flask-redis

先说 flask-redis
src/extentions.py

```python
from redis import Redis
class FlaskRedis:
    def __init__(self) -> None:
        self.client = None

    def init(self, app):
        self.client = Redis.from_url(app.config["REDIS_URL"])
redis_client = FlaskRedis()
```

只需要在需要的地方

```python
from extensions import redis_client

def get_redis_data(key) -> list[Any] | dict[Any, Any] | None:
    ret = redis_client.client.get(f"{key}")
```

是不是超级简单,完全满足项目需要. 也不需要记忆开源插件的配置.

## flask apscheduler

再譬如 scheduler <br />
`src/extentioons.py`

```python
from apscheduler.schedulers.blocking import BlockingScheduler
class FlaskScheduler:
    def init(self, app):
        scheduler = BlockingScheduler()

        scheduler.configure(**app.config["SCHEDULER_OPTIONS"])

        jobs = app.config.get("SCHEDULER_JOBS", [])
        for job in jobs:
            scheduler.add_job(**job)
        self.scheduler = scheduler
scheduler = FlaskScheduler()
```

当然,这里需要在 create_app 中 init 一下

```python
def create_app(test_config: dict[str, Any] = None) -> Flask:
    ...
    redis_client.init(app)
    scheduler.init(app)
    ...
```

scheduler 的使用需要通过 flask 中的 click 配合实现:<br />
`/src/cli/scheduler.py`

```python
from flask.cli import with_appcontext
from extensions import scheduler
from flask.cli import AppGroup

scheduler_cli = AppGroup("scheduler")


@scheduler_cli.command("start")
@with_appcontext
def start():
    scheduler.scheduler.start()


@scheduler_cli.command("stop")
@with_appcontext
def stop():
    scheduler.scheduler.stop()
```

文章写到这里, 发现这个项目还有一个替换`flask-migrate`的实现. 但是现在已经是早上 8 点半了. 我要去上班了. 等有时间在下篇文章中介绍.
