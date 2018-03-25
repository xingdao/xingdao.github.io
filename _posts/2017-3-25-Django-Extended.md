---
layout: post
title: Django 实用扩展
category: Django
keywords: Python,Django
---

### Django-Choices

#### 介绍

	更简洁的Choices

#### 样例
```python
from djchoices import DjangoChoices, ChoiceItem

class BillingTypes(DjangoChoices):
    RECHARGE = ChoiceItem(1, u'转入')
    REFUND = ChoiceItem(2, u'转出')
    REWARD = ChoiceItem(3, u'佣金奖励')
    ACTIVITY = ChoiceItem(4, u'活动奖励')

# 使用
class Billing(models.Model):
    """
    账单
    """
    reason = models.IntegerField(verbose_name='变更原因', choices=BillingTypes.choices,
                                 default=BillingTypes.RECHARGE, validators=[BillingTypes.validator])

# 使用
u"转入"==BillingTypes.values[BillingTypes.RECHARGE]           
1 == BillingTypes.RECHARGE
```


### Django-filter

#### 介绍
	
	支持多选过滤等多种方式

#### 样例
```python

import django_filters

from django_filters.filters import MultipleChoiceFilter

from financial.Money.models import Billing
from financial.Money.types import BillingTypes


class BillingReason(django_filters.FilterSet):
    reason = MultipleChoiceFilter(choices=BillingTypes.choices)
    # 设置过滤支持多选
    class Meta:
        model = FBilling
        fields = ['reason']

# 使用
class UserFBill(generics.ListAPIView):
    """对账"""

    permission_classes = (permissions.IsAuthenticated,)
    serializer_class = serializers.BillingSerializer
    filter_backends = (filters.DjangoFilterBackend,)
    filter_class = filter.BillingReason

    def get_queryset(self):
        if self.request.user.is_staff:
            return models.Billing.objects.all().order_by('-id')
        else:
            return models.Billing.objects.filter(user=self.request.user).order_by('-id')
```


### Django-import-export

#### 介绍
	
	数据导出为xls,并合并如admin中

#### 样例
```python
from import_export import resources, fields
from import_export.admin import ExportActionModelAdmin

from shop.models import Clinch


class ResourceFromClinch(resources.ModelResource):
    id = fields.Field(column_name="ID", attribute="id")
    user_name = fields.Field(column_name="用户名", attribute="user__username")

    def dehydrate_order_status(self, obj):
        return obj.related_clinch.get_clinch_status_display()

    class Meta:
        model = Clinch
        fields = ["id", "user_name"]
        export_order = ["id", "user_name"]

    def get_queryset(self):
        Clinch.objects.select_related().all()

class ClinchAdmin(ExportActionModelAdmin):
    """订单之管家服务"""

    search_fields = ['id', 'username']
    list_display = [field.name for field in Clinch._meta.get_fields()
                    if field.name not in ['periods']]
    resource_class = ResourceFromClinch

admin.site.register(Clinch, ClinchAdmin)
```



### Django-cors-headers

#### 介绍

	为React 支持跨域

#### 样例
```python
MIDDLEWARE_CLASSES = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.common.CommonMiddleware',
#    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.auth.middleware.SessionAuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```


### Django-dbbackup


#### 介绍

	数据备份

#### 样例
	
	wating


### django-celery


#### 介绍

	celery 扩展

#### 样例

	wating
