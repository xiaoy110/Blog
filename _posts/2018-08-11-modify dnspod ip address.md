---
layout: post
title:  connecting mongo
date:   2018-8-11 19:28:58
categories: Python
tags: Python
---

# I don't have explan to much

~~~python
import requests,json
class Dnspod():
    def __init__(self,token):
        self.token = token
    def getSub(self, domain):
        if len(domain.split('.')[0:-2]) == 0:
            return '@', str(domain)
        else:
            sub = ''
            for i in domain.split('.')[0:-2]:
                sub += i + '.'
            return sub[0:-1], domain.split('.')[-2] + '.' + domain.split('.')[-1]
    def getdomain(self):
        formaData = {'login_token': token, 'format': 'json'}
        url = 'https://dnsapi.cn/Domain.List'
        r = requests.post(url,formaData)
        json_all = json.loads(r.text)
        all_domains=[]
        for domain in json_all['domains']:
            all_domains.append({domain['name']:domain['id']})
        return all_domains
    def get_domain_ip(self,domains,domain_id):
        sub, root = self.getSub(domains)
        url = 'https://dnsapi.cn/Record.List'
        formaData = {'login_token': token, 'format': 'json','domain_id':domain_id}
        r = requests.post(url, formaData)
        sub_domains = json.loads(r.text)['records']
        for sub_domain in sub_domains:
            if sub_domain['name'] == sub:
                print(sub_domain['name'], sub_domain['value'],sub_domain['id'])
                return sub_domain['name'], sub_domain['id']
    def change_domain_ip(self,domains,value=None,record_type='A',record_line='默认'):
        sub,root = self.getSub(domains)
        for i in self.getdomain():
            (key,domain_id), = i.items()
            if root in i:
                try:
                    sub_domain,record_id = self.get_domain_ip(domains,domain_id)
                    url = 'https://dnsapi.cn/Record.Modify'
                    formaData = {'login_token': token, 'format': 'json', 'domain_id': domain_id,
                                 'sub_domain': sub_domain, 'record_id': record_id, 'value': value,'record_type':record_type, 'record_line':record_line}
                    r = requests.post(url, formaData)
                    print(r.text)
                except TypeError as e:
                    print('that sub domain do not exist! \n%s' % e)
                    exit(1)
if __name__ == '__main__':
    token='8888,e273c9c9b90ac48bcd1d65d3dfb75eb8'
    Dnspod(token).change_domain_ip('design.gbicom.cn','182.92.231.159')
~~~
