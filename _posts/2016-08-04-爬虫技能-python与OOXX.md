---
layout: post
category: 技术
tag: 技术
---
### Python与OOXX的搏斗之路

#### 打开一个网页，jandan需要伪装为浏览器

正常打开一个网页，下面的代码就可以使用（url换成其他网站），然而在就jandan网上，打不开网页，需要伪装成为浏览器

```

import urllib.request

response = urllib.request.urlopen('http://jandan.net/ooxx')

html = response.read()

print (html)

```

#### 伪装浏览器，就是替换user_agent 

在浏览器中，可以找到发送数据的user-agent

user-agent使用方法(加上user-agent就可以打开jandan的网页了)：

```

user_agent ='Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.94 Safari/537.36'

	headers = {'User-Agent':user_agent}

	req = urllib.request.Request(url,headers = headers)

	oper = urllib.request.urlopen(req)

	data = oper.read()

	dataUtf = data.decode()

```

#### 正则表达式，找到匹配地址

在网页中可以找到.jpg的地址，怎么匹配找到.jpg的地址,推荐使用正则表达式(按照一定规则匹配字符串)。

可以使用在线测试，测试自己的正则表达式。

[正则表达式，在线测试](http://tool.oschina.net/regex/)

正则表达式部分：

```

	pattern = re.compile('(http://ww\d.sinaimg.cn/large/.+?\.jpg)')

	items = re.findall(pattern,dataUtf)

```

#### 可以下载图片了

因为网站往往有反爬虫机制所以注意两个问题：

1. 再次连接.jpg的时候记得加上user-agent

2. 最好加入延时函数，以免操作速度过快被服务器阻拦

延迟函数加入方法

```

import time

time.sleep(2)

```    

猜测网站阻拦爬虫机制如下：

|策略|解决方案|

|:---|:---|

|连接是爬虫的user-agent|更换user-agent|

|操作速度过快|加入延时|

|同一个ip下载数据量过大|使用ip代理，爬下一些ip代理，随机使用|

|同一个user-agent访问站点过多|爬下一些user-agent随机使用


建立文件夹：

```

import os

folder='OOXX'

os.mkdir(folder)

os.chdir(folder)

```


下载保存图片函数

```

def saveImg(imageURL,fileName):

     user_agent ='Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.94 Safari/537.36'

     headers = {'User-Agent':user_agent}

     req = urllib.request.Request(imageURL,headers = headers)

     u = urllib.request.urlopen(req)

     data = u.read()

     f = open(fileName, 'wb')

     f.write(data)

     f.close()

```



#### 完整代码

```python```

#encoding:UTF-8

import urllib.request

import time

import re


def saveImg(imageURL,fileName):

     user_agent ='Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.94 Safari/537.36'

     headers = {'User-Agent':user_agent}

     req = urllib.request.Request(imageURL,headers = headers)

     u = urllib.request.urlopen(req)

     data = u.read()

     f = open(fileName, 'wb')

     f.write(data)

     f.close()


def seturl(pageNum):

	#opener.addheaders = [('User-Agent','Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:44.0) Gecko/20100101 Firefox/44.0')]

	#url = 'http://jandan.net/ooxx/page-2076#comments'

	url =  'http://jandan.net/ooxx/page-' + str(pageNum) + '#comments'

	user_agent ='Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.94 Safari/537.36'

	headers = {'User-Agent':user_agent}

	req = urllib.request.Request(url,headers = headers)

	oper = urllib.request.urlopen(req)

	data = oper.read()

	dataUtf = data.decode()

	return dataUtf

	#print(dataUtf)


i = 0

for j in range(2076):

	dataUtf = seturl(2076-j)

	#pattern = re.compile('http://ww\d.sinaimg.cn/large/(.+?\.jpg)',re.S)

	pattern = re.compile('(http://ww\d.sinaimg.cn/large/.+?\.jpg)')

	items = re.findall(pattern,dataUtf)

	#print (items)

	for element in items:

		i = i+1

		print(element)

		filename = str(i) + ".jpg"

		time.sleep(2)

		saveImg(element,filename)

```


#### 之后触发了反爬虫机制（以后更新与反爬虫机制的搏斗过程）





---


### 更新对抗反爬虫机制

再次确定下反爬虫机制的工作原理


|策略|解决方案|

|:---|:---|

|连接是爬虫的user-agent|更换user-agent|

|操作速度过快|加入延时|

|同一个ip下载数据量过大|使用ip代理，爬下一些ip代理，随机使用|

|同一个user-agent访问站点过多|爬下一些user-agent随机使用


因此程序中缺少，随机的ip，和随机的useragent。需要得到ip代理和useragent列表供随机使用。


#### useragent可以百度到，找一个文件保存，以供使用。

新建一个useragent.txt文档，放入以下内容。

```

Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.835.163 Safari/535.1

Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.94 Safari/537.36

Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:44.0) Gecko/20100101 Firefox/44.0

Mozilla/5.0 (Windows NT 6.1; WOW64; rv:6.0) Gecko/20100101 Firefox/6.0

Mozilla/5.0 (Windows; U; Windows NT 6.1; ) AppleWebKit/534.12 (KHTML, like Gecko) Maxthon/3.0 Safari/534.12

```

#### ip代理的设置使用方法（主要问题）

- 百度一个ip代理网站，我选择的是xici代理，从代理主页中，爬下来高匿的代理，这里我爬下来了约1000，ip地址和端口号,保存在一个proxy.txt文档中。

代码如下

```python```

import urllib.request

import random

import re

j = 1

f=open('proxy.txt','w')

for i in range(1,12,1):

		url = 'http://www.xicidaili.com/nn/'+str(i)

		headers = {'User-Agent':'Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:44.0) Gecko/20100101 Firefox/44.0'}

		req = urllib.request.Request(url,headers = headers)

		response = urllib.request.urlopen(req)

		html = response.read().decode('utf-8')

		pattern = re.compile('((?:(?:25[0-5]|2[0-4]\d|((1\d{2})|([1-9]?\d)))\.){3}(?:25[0-5]|2[0-4]\d|((1\d{2})|([1-9]?\d))))</td>.*\s+<td>([1-9][0-9]{0,3})</td>')

		items = re.findall(pattern,html)

		for element in items:

			#print(str(j)+':'+'  '+element[0]+':'+element[7])

			print(element[0]+':'+element[7])

			f.write(element[0]+':'+element[7]+'\n')

			#j = j + 1

f.close()


```

- ip代理网站，对煎蛋网进行测试

验证代码如下，将实际可用的ip保存到avilible.txt中（1000个代理ip，只有7个实际可用，阿门。。。）：

```python```

import urllib.request

import socket

import random

import time

import re

import os


timeout = 3

url = 'http://jandan.net/ooxx/'

f = open('proxy.txt','r')

iplist = f.readlines()

f.close()

f = open('useragent.txt','r')

useragent = f.readlines()

f.close()

f = open('avilible.txt','w')

avilibNum = 0

for i in range(1000):

	print(random.choice(iplist))

	#print(random.choice(useragent))

	socket.setdefaulttimeout(timeout)

	try:

		ipChoice = iplist[i]

		proxy_support = urllib.request.ProxyHandler({'http':ipChoice})

		opener = urllib.request.build_opener(proxy_support)

		#opener.addheaders = [('User-Agent','Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:44.0) Gecko/20100101 Firefox/44.0')]

		opener.addheaders = [('User-Agent',random.choice(useragent))]

		urllib.request.install_opener(opener) 

		response = urllib.request.urlopen(url)		

		html = response.read().decode('utf-8')	

		pattern = re.compile('(煎蛋正版认证)')

		items = re.search(pattern,html)

		if items != None:

			avilibNum = avilibNum+1

			print ('aviliable' + str(avilibNum))

			f.write(ipChoice)

	except Exception as ex:  

    		print (Exception,":",ex) 

	#except urllib.error.URLError as e:

	#	print(e.reason)


f.close()

#print(html)

```

### 将useragent和ip代理，设置到爬虫中

上完整代码(附件中有完整工程)：

```python```

#encoding:UTF-8

import urllib.request

import random

import time

import re

import os

def saveImg(imageURL,fileName):

	data = urlInit(imageURL)

	f = open(fileName, 'wb')

	f.write(data)

	f.close()


def seturl(pageNum):

	url =  'http://jandan.net/ooxx/page-' + str(pageNum)

	data = urlInit(url)

	dataUtf = data.decode()

	return dataUtf


def findTotlePages():

	url =  'http://jandan.net/ooxx/'

	data = urlInit(url)

	dataUtf = data.decode()

	pattern = re.compile('http://jandan.net/ooxx/page-(.+?)#comments')

	items = re.findall(pattern,dataUtf)

	return items[0]


def urlInit(urlInput):

		items = setRandom.getlast()

		proxy_support = urllib.request.ProxyHandler({'http':items[0].strip()})

		#proxy_support = urllib.request.ProxyHandler({'http':'121.193.143.249:80'})

		opener = urllib.request.build_opener(proxy_support)

		url =  urlInput

		#user_agent ='Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.94 Safari/537.36'

		#headers = {'User-Agent':user_agent}

		headers = {'User-Agent':items[1].strip()}

		urllib.request.install_opener(opener) 

		req = urllib.request.Request(url,headers = headers)

		oper = urllib.request.urlopen(req)

		data = oper.read()

		return data


class setRandom:

	iplist = ''

	useragent = ''

	iplast = ''

	userlast = ''

	def setit():

		f = open('proxy.txt','r')

		ip = f.readlines()

		f.close()

		f = open('useragent.txt','r')

		user = f.readlines()

		f.close()

		setRandom.iplist = ip

		setRandom.useragent = user

	def getit():

		ip = random.choice(setRandom.iplist)

		user =random.choice(setRandom.useragent)

		setRandom.iplast = ip

		setRandom.userlast = user

	def getlast():

		return setRandom.iplast,setRandom.userlast


setRandom.setit()

setRandom.getit()

url = 'http://jandan.net/ooxx/'

totleNumb = int(findTotlePages())

i = 0

folder='OOXX'

os.mkdir(folder)

os.chdir(folder)


for j in range(totleNumb):

	try:

		#setRandom(url,useragent)

		dataUtf = seturl(totleNumb-j)

		pattern = re.compile('(http://ww\d.sinaimg.cn/large/.+?\.jpg)')

		items = re.findall(pattern,dataUtf)

		for element in items:

			i = i+1

			print(element)

			filename = str(i) + ".jpg"

			time.sleep(2)

			saveImg(element,filename)

	except Exception as ex:

		setRandom.getit()

		print (Exception,":",ex) 

```

效果：


- 说明下：

1. 当程序出现异常时，重新设置ip和useragent，ip和useragent，在文件中，使用random函数随机选取。有上图可见，当出现异常现象时，程序可以继续进行下去。

2. 另外一项更新：原本程序无法识别首页是第几页，手工赋给了一个2076，上面代码可以使用findTotlePages函数得到全部页码。

3. enjoy it欢迎交流


