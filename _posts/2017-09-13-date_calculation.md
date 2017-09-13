~~~python
month = datetime.date.today().month
year = datetime.date.today().year
st = datetime.date(year=year, month=month, day=1) + relativedelta(months=-1)
et = datetime.date(year=year, month=month, day=1) + relativedelta()
# print datetime.datetime.strptime( str(st) + " 00:00:00","%Y-%m-%d %H:%M:%S")
start_t = str(st) + " 00:00:00"
end_t = str(et) + " 00:00:00"
writer = None
fname = st.strftime("%Y年%m月通话账单.xlsx")
~~~
