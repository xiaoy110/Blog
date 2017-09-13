---
layout: post
title:  just send email
date:   2017-09-10 23:28:58
categories: Python
tags: Python
---

# query the table of the mongo from cc_data_20170505(database)

### the result of selected written to the excel.

~~~python
from pymongo import MongoClient
import datetime
import pandas as pd


# this just for chagne type of time
# date_str = '2017-05-05 00:00:00'
# def changetime(ttime) :
#  localtime = datetime.datetime.strptime(date_str, "%Y-%m-%d %H:%M:%S")
#  return localtime


client = MongoClient('mongodb://10.10.10.10:10') #craete a client
db = client.cc_data_20170505 # connecting to db
collection = db.N5_c5_call_sheet # connecting a table
print  collection.find_one({'STATUS':'dealing'}) # select * from xxx where status = dealing
df = collection.find({'STATUS':'dealing','BEGIN_TIME':{"$gte": "2017-06-07 00:00:00","$lte": "2017-06-09 00:00:00"}})
dfw = pd.DataFrame(list(df)) #
dfw.to_excel('output2.xlsx',index=False)
~~~
