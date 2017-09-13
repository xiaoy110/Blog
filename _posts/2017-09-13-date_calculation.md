---
layout: post
title:  date calculation
date:   2017-09-13 11:31:00
categories: Python
tags: Python
---


~~~python
import datetime
from dateutil.relativedelta import relativedelta


month = datetime.date.today().month
#获取当前月
year = datetime.date.today().year #年
st = datetime.date(year=year, month=month, day=1) + relativedelta(months=-1)
et = datetime.date(year=year, month=month, day=1) + relativedelta()
# print datetime.datetime.strptime( str(st) + " 00:00:00","%Y-%m-%d %H:%M:%S")
start_t = str(st) + " 00:00:00"
end_t = str(et) + " 00:00:00"
fname = st.strftime("%Y年%m月通话账单.xlsx")
print st,et
print start_t,end_t
print fname
---------------
2017-08-01 2017-09-01
2017-08-01 00:00:00 2017-09-01 00:00:00
2017年08月通话账单.xlsx
~~~
