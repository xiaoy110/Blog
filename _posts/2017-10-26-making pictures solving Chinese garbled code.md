---
layout: post
title:  Making pictures through data
date:   2017-10-26 10:28:58
categories: Python
tags: Python
---


# found top ten and the last ten
#
~~~python
import matplotlib.pylab as pylab
import matplotlib.pyplot as plt
from matplotlib.font_manager import FontManager
from pymongo import MongoClient
import pandas as pd
import datetime
from dateutil.relativedelta import relativedelta
from pylab import mpl
import subprocess

params={
    'axes.labelsize': '35',
    # set figure size
    'axes.grid': 'on' #显示网格
}
pylab.rcParams.update(params)

#solving Chinese garbled code
def get_matplot_zh_font():
    fm = FontManager()
    mat_fonts = set(f.name for f in fm.ttflist)

    output = subprocess.check_output('fc-list :lang=zh -f "%{family}\n"', shell=True)
    zh_fonts = set(f.split(',', 1)[0] for f in output.split('\n'))
    available = list(mat_fonts & zh_fonts)

    print '*' * 10, '可用的字体', '*' * 10
    for f in available:
        print f
    return available

def set_matplot_zh_font():
    available = get_matplot_zh_font()
    if len(available) > 0:
        mpl.rcParams['font.sans-serif'] = [available[0]]    # 指定默认字体
        mpl.rcParams['axes.unicode_minus'] = False
def rresult(callno):
    ty = []
    alcon = []
    for i in xrange(12):
        n = i+2
        year = datetime.date.today().year
        if n < 13:
            st = datetime.date(year=year, month=n, day=1) + relativedelta(months=-1)
            et = datetime.date(year=year, month=n, day=1) + relativedelta()
        elif n == 13:
            st = datetime.date(year=year, month=12, day=1)
            et = datetime.date(year=year, month=12, day=31)
        client = MongoClient('mongodb://10.10.10.10:10') #craete a client
        db = client.cc_data_20170505 # connecting to db
        collection = db.N00000013385_c5_call_sheet # connecting a table

        start_t = str(st) + " 00:00:00"
        if n < 13:
            end_t = str(et) + " 00:00:00"
        elif n ==13:
            end_t = str(et) + " 23:59:59"
        #有效的连接数
        connect = collection.find({'OFFERING_TIME':{"$gte": start_t,"$lte": end_t},'CALL_NO': callno,'STATUS':'dealing','CONNECT_TYPE':'dialout'}).count()
        #所有的连接数
        alconnect = collection.find({'OFFERING_TIME':{"$gte": start_t,"$lte": end_t},'CALL_NO': callno}).count()
        alcon.append(alconnect)
        if connect:
            taa= float(connect)/float(alconnect)*100
            ty.append(float('%.2f'%taa)) # Retain two decimal places

        else:
            ty.append(0)
    return ty,alcon


# months
tx = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
def ratetop10(top10):
    for i in top10['号码']:
        Num = str(0)+str(i)
        rat,alco = rresult(Num)
        #上面的图
        plt.subplot(211)
        plt.plot(tx, rat,'s-')  # use pylab to plot x and y
        plt.title(u'接通率')
        #下面的图
        plt.subplot(212)
        plt.plot(tx,alco,'s-', label=i)
        plt.title(u'通话总量')
    # 标签所在的位置
    plt.legend(loc="upper left")
    plt.show()
def ratedown10(down10):
    for i in down10['号码']:
        Num = str(0)+str(i)
        rat,alco = rresult(Num)
        #上面的图
        plt.subplot(211)
        plt.plot(tx, rat,'s-')  # use pylab to plot x and y
        plt.title(u'接通率')
        #下面的图
        plt.subplot(212)
        plt.plot(tx,alco,'s-', label=i)
        plt.title(u'通话总量')
    # 标签所在的位置
    plt.legend(loc="upper left")
    plt.show()
if __name__ == "__main__":
    get_matplot_zh_font()
    set_matplot_zh_font()
    xls = pd.read_excel('工作簿1.xlsx', names=['号码', 'a', 'b'])
    # 拉黑次数最多前10
    top10 = xls.sort_values('a', ascending=False).head(10)
    down10 = xls.sort_values('a').head(10)
    ratetop10(top10)
    ratetop10(down10)
~~~
