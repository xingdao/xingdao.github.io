---
layout: post
title: Python 元编程实战(混合视图)
category: Python
keywords: Python, meta, Orm, mixins
---

#### 介绍

	半吊子ORM, 加个 Mixins

    Copy from Django restframework

#### 大致原理

    和orm 那类似, 元编程获取 model,
    可以把ReadOperation, WriteOperation 更为细致的拆分, 然后组合成 Django rest_framework 一样的View
    当然可以通过 类和继承实现, 不过用元编程可以省几行代码


#### 代码

<pre class="prettyprint linenums">
# coding=utf-8
from server.database import db
from server.status import NoCoupon
from server.meta.decorators import make_passing, Passing


class MetaOperation(type):

    def __new__(cls, name, bases, tpls):
        model = tpls.get('model', '')

        @make_passing
        def query_page(index, count, params):
            data = model.query_paging(db.reader, index, count, 'id desc', **params)
            if not data:
                raise NoCoupon
            total = model.query_count(db.reader, **params)
            return Passing(data=data, index=index, count=count, total=total)

        @make_passing
        def query(params):
            return Passing(data=model.query(db.reader, **params))

        @make_passing
        def query_one(_id):
            data = model.query_one(db.reader, id=_id)
            return Passing(data=data)

        @make_passing
        def insert(payload):
            res = model.insert(db.writer, payload)
            if res:
                return Passing(data={'id': res})
            raise CouponError('编辑错误')

        @make_passing
        def update_one(_id, payload):
            data = model.update(db.writer, payload, id=_id)
            if data:
                return Passing(data=model.query_one(db.reader, id=_id))
            raise CouponError('编辑错误')

        tpls['query_page'] = staticmethod(query_page)
        tpls['query'] = staticmethod(query)
        tpls['query_one'] = staticmethod(query_one)
        tpls['insert'] = staticmethod(insert)
        tpls['update_one'] = staticmethod(update_one)
        return super().__new__(cls, name, bases, tpls)


class ReadOperation(object):
    """
    虚拟的只读类,在多继承 时欺骗编译器
    """
    @staticmethod
    def query_page(index, count, params):
        pass

    @staticmethod
    def query(params):
        pass

    @staticmethod
    def query_one(_id):
        pass


class WriteOperation(object):
    """
    虚拟的写入类,在多继承 时欺骗编译器
    """
    @staticmethod
    def update_one(_id, payload):
        pass

    @staticmethod
    def insert(payload):
        pass


class BaseOperation(metaclass=MetaOperation):
    pass


</pre>

### 参考

    [from rest_framework import mixins](http://www.django-rest-framework.org/tutorial/3-class-based-views/)
