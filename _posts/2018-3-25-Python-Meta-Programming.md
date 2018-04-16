---
layout: post
title: Python 元编程实战(Flask 流水线)
category: Python
keywords: Python, meta, Flask
---

### 

#### 介绍

	这是一个Flask不使用 blueprints, 后为优雅展示代码的设计
    (为什么不用 blueprints, 因为在一个比较随意的公司 233, 迫生)

#### 大致原理

    使用类似Nginx的流水线设计(更细分割的MVC)
    通过装饰器实现函数封装, 和参数检查
    当然可以通过 类和继承实现, 不过用元编程可以省几行代码

#### 代码


<pre class="prettyprint linenums">
# coding=utf-8

"""
注意装饰器在 初始化时即执行
"""

from functools import wraps


class PassingParameterError(Exception):
    pass


class PassingCalledError(Exception):
    pass


class Passing(dict):
    def __init__(self, **kwargs):
        super(Passing, self).__init__(**kwargs)

    def __str__(self):
        return "Passing<{}>".format(super(Passing, self).__str__())

    def __repr__(self):
        return "Passing<{}>".format(super(Passing, self).__repr__())


def __collect_prevfunc_params(**params):
    # 分离 限定类型 和额外参数
    restrictions = {}
    values = {}
    for name in params:
        typ = params[name]
        # print('name: %s type %s' % (name, typ), end=" ")
        try:
            isinstance(0, typ)
            restrictions.update({name: typ})
        except:
            values.update({name: typ})
    # print('restrictions %s, values %s' % (restrictions, values))
    return restrictions, values


def __check_params(restrictions, values, next_func, params):
    # 检查参数是否符合类型限制
    next_params = {}
    for k in restrictions:
        value = params.get(k, None)
        if value is None:
            raise PassingCalledError("{func_name} missing 1 required positional argument: {key}".
                                     format(func_name=next_func.__name__, key=k))
        if not isinstance(value, restrictions[k]):
            raise PassingParameterError('{key} must be a {typ}'.
                                        format(key=k, typ=restrictions[k].__name__))
        next_params[k] = value
    next_params.update(values)
    return next_params


def make_passing(next_func=None):

    @wraps(next_func)
    def input_params(**params):
        # print('params %s' % params)
        restrictions, values = __collect_prevfunc_params(**params)

        def decorator(real_func):
            @wraps(real_func)
            def wrapper(*real_args, **real_kwargs):
                resp = real_func(*real_args, **real_kwargs)
                # 只装饰 类型为 Passing 的返回值进行下一次调用
                if isinstance(resp, Passing):
                    return next_func(**__check_params(restrictions, values, next_func, resp))
                else:
                    return resp

            return wrapper

        return decorator

    return input_params


if __name__ == '__main__':

    class A(object):

        @staticmethod
        @make_passing
        def a1(c, b):
            return Passing(c=c, b=b)

        @staticmethod
        @make_passing
        def a2(c, b):
            return {'c': c, 'b': b}


    class B(object):
        @staticmethod
        @A.a2(c=int, b=str)
        @A.a1(c=int, b=str)
        def start(c, b):
            return Passing(c=c, b=b)

    print(B.start(1, '2'))
</pre>


#### 注意

    但额外参数和需要 检查的参数名字一致时会被覆盖安全检查,
    可以在 make_passing 中添加额外参数(如 level)进行流程控制 
    其次, 最好多用业界稳点套件

### 参考
- [完整Demo](https://github.com/xingdao/py_test/tree/master/flask)
- [Python3-cookbook](http://python3-cookbook.readthedocs.io/zh_CN/latest/)
- [Python Tips](http://tips.pyhub.cc/zh/latest/)
