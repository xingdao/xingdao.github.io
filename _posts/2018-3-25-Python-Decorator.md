---
layout: post
title: Python 装饰器
category: Python
keywords: Python, decorator
---

#### 介绍

	配合Passing, 使用的大量手脚架

#### 大致原理

    直接在 passing 中某层抛出自定义异常CouponError , 通过passing最外层 collect_exceptions 捕获 交给 message_handler处理


#### 代码

<pre class="prettyprint linenums">

# coding=utf-8

import time
import logging

from functools import wraps, partial

ServerError = {"msg": "服务器异常", "status": 500}

class MessageException(Exception):
    def __init__(self, message):
        self.message = message


def message_handler(name, e, *args, **kwargs):
    if isinstance(e, MessageException):
        return False, e.message
    else:
        logging.error('name:%s 运行错误 %s args:%s kwargs:%s' % (name, e, args, kwargs), exc_info=True)
        return False, ServerError


class CouponError(MessageException):
    def __init__(self, msg):
        self.message = {"msg": msg, "status": 400}

# 或者直接在外部导入 CouponError, 后声明 msg后使用

NoCoupon = CouponError('优惠券不存在')

def collect_exceptions(handler, *args, **kwargs):
    def decorator(real_func):
        @wraps(real_func)
        def wrapper(*real_args, **real_kwargs):
            try:
                resp = real_func(*real_args, **real_kwargs)
                return resp
            except Exception as e:
                return handler(real_func.__name__, e, *args, **kwargs)
        return wrapper
    return decorator


# 重试多次

class RetryExecute(Exception):
    pass


def retry_execute(count=5, *args, **kwargs):
    def wrapper(fn):
        @wraps(fn)
        def execute(*real_args, **real_kwargs):
            for _ in range(count):
                try:
                    return fn(*real_args, **real_kwargs)
                except RetryExecute:
                    continue
        return execute
    return wrapper


# 简单的性能检查(不让用APM就是麻烦)
def performance(func=None, log=logging,
                level=logging.DEBUG, message='{fn_name}: {time}'):
    if func is None:
        return partial(performance, log=log, level=level, message=message)

    @wraps(func)
    def wrapper(*realfn_args, **realfn_kwargs):
        t0 = time.time()

        result = func(*realfn_args, **realfn_kwargs)

        interval = round(time.time() - t0, 5)

        nonlocal log

        log.log(level, message.format(fn_name=func.__name__, time=interval))

        if interval >= 0.5:
            log.log(logging.ERROR,
                    'The {fn_name} function takes {time}s, please timely optimize.'.
                    format(fn_name=func.__name__, time=interval))

        return result
    return wrapper


# 重置dict 点操作符
class DictModel(dict):
    def __init__(self, *args, **kwargs):
        super(DictModel, self).__init__(*args, **kwargs)

    def __getattr__(self, key):
        try:
            value = self[key]
            if type(value) == dict:
                value = self[key] = DictModel(value)
                return value
            return value
        except KeyError:
            raise AttributeError(r'"DictModel" object has no attribute "%s"' % key)

    def __setattr__(self, key, value):
        self[key] = value

```
</pre>

### 参考

- [Python3-cookbook](http://python3-cookbook.readthedocs.io/zh_CN/latest/)
- [Python Tips](http://tips.pyhub.cc/zh/latest/)
