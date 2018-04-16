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
from server.status import message
from server.util.decorators import make_passing, Passing


class __MetaOperation(type):

    def __new__(cls, name, bases, tpls):
        model = tpls.get('model', '')

        @make_passing
        def query_page(index, count, params):
            data = model.query_paging(db.reader, index, count, 'id desc', **params)
            if not data:
                return message.EmptyOldList
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
            else:
                return message.DataEditError

        @make_passing
        def update_one(_id, payload):
            data = model.update(db.writer, payload, id=_id)
            if data:
                return Passing(data=model.query_one(db.reader, id=_id))
            else:
                return message.DataEditError

        # 如果 子类重写, 使用子类
        for x in [query_page, query, query_one, insert, update_one]:
            if str(x.__name__) not in tpls.keys():
                tpls[str(x.__name__)] = staticmethod(x)
        return super().__new__(cls, name, bases, tpls)


class ReadOperation(object):
    """
    虚拟的只读类,在多继承 时欺骗编译器
    """
    @staticmethod
    @make_passing
    def query_page(index, count, params):
        raise TypeError()

    @staticmethod
    @make_passing
    def query(params):
        raise TypeError()

    @staticmethod
    @make_passing
    def query_one(_id):
        raise TypeError()


class WriteOperation(object):
    """
    虚拟的写入类,在多继承 时欺骗编译器
    """
    @staticmethod
    @make_passing
    def update_one(_id, payload):
        raise TypeError()

    @staticmethod
    @make_passing
    def insert(payload):
        raise TypeError()


class BaseOperation(metaclass=__MetaOperation):
    pass



</pre>

### 参考

- [完整Demo](https://github.com/xingdao/py_test/tree/master/flask)
- [from rest_framework import mixins](http://www.django-rest-framework.org/tutorial/3-class-based-views/)
