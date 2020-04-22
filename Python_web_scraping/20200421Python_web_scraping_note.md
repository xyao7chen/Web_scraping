# Python 网络爬虫与信息提取

## Intro

The Website is the API... 

<img src="C:\1_Coding\202004_Datawhale_web_scraping_study\Python_web_scraping\ws_p1.png" alt="p1" title="Goal" style="zoom:80%;" />





Requests library: best third-party Python lib for web scraping - to get web information 

Beautiful soup: to extract web information 

12 units: 8 content units + 4 case study units



## Python development tools

 

Frequent Python IDE tools:

- Text tools: 

- - IDLE (the course uses) : by default, including 
- - Notepad++
- - Sublime Text (the course uses, and proposes)
- - Vim & Emacs 
- - Atom

- Integrated tools:

- - PyCharm (the course uses): 
- - Wing : paid, debugging, for big projects
- - PyDev & Eclipse : open source, need dev experience 
- - Visual Studio : Ms, debugging 
- - Anaconda & Spyder (the prof uses): nearly 900 third-party library
- - Canopy: paid, scientific calculation and data science 





## Course

### Structure  

1. Requests lib Intro 

2. robot.txt - not 

3. Real case of Request applications

   
### 1 Requests
#### 1.1. get()

`r = requests.get(url)`
构造一个向服务器请问资源的Requests对象
返回一个包含服务器资源的Response对象

`r = requests.get(url, params= None, **kwargs)`
url: 拟获取页面的ur链接
params: url的额外参数，字典或字节流格式，可选
\**kwargs: 12个控制访问的参数

**Two important objects of Request library** 
`r = requests.get(url)`

- Response: 包含所有返回的爬虫对象
- Request

**Response object:**
'''
import requests
r = requests.get("http://www.baidu.com")
print(r.status_code)
type(r)
'''

**Attributes of Response object:** 

| Attribute          | Description                              |
| :------------------|:-----------------------------------------|
| r.status_code      | HTTP请求的返回状态，200表示连接成功，404表示失败|
| r.text             | HTTP响应内容的字符串形式，即，url对应的页面内容 |
| r.encoding         | 从HTTP header中猜测的响应内容编码方式         |
| r.apparent_encoding| 从内容中分析出响应内容的编码方式（备选编码方式）  |
| r.content          | HTTP响应内容的二进制形式                     |

r.content can be used to get a picture from a web page. 





r.status_code --> 200 r.text r.encoding r.apparent_encoding r.content

​                        -->404 or others   error or abnormal



**Examples:** 

see Jupyter notbook  

**Encoding:** 

| 属性 | 说明 |
| :---- | :---- |
| r.apparent_encoding | 从内容中分析出响应内容的编码方式（备选编码方式） |
| r.content | HTTP响应内容的二进制形式 |



这两种编码有什么不同？

简单说，网络上的资源，他有他的编码，如果没有编码，我们将看不懂，没办法用有效的解析方式，使得人类可读这样的内容。所以我们需要编码这样一个概念。

r.encoding这样的一个编码方式是从header中的charset字段获得的。

如果HTTP的header有这样一个字段，说明我们访问的服务器，对它资源的编码是有要求的。而这样的编码，会获得回来，并存在于他的encoding之中。但是并不是所有的服务器都是对他的encoding有要求的，(r.encoding:)如果header中不存在charset，则默认编码为ISO-8859-1。

**Example: **

'''
r = requests.get("http://www.baidu.com")
r.header
'''
{'Cache-Control': 'private, no-cache, no-store, proxy-revalidate, no-transform', 'Connection': 'Keep-Alive', 'Content-Encoding': 'gzip', 'Content-Type': 'text/html', 'Date': 'Tue, 21 Apr 2020 07:54:25 GMT', 'Last-Modified': 'Mon, 23 Jan 2017 13:28:11 GMT', 'Pragma': 'no-cache', 'Server': 'bfe/1.0.8.18', 'Set-Cookie': 'BDORZ=27315; max-age=86400; domain=.baidu.com; path=/', 'Transfer-Encoding': 'chunked'}

可以看到，百度的header, 没有charset字段，所以访问百度的时候，默认编码为ISO-8859-1。但是不能解析中文。所以Requests库提供了一个备选编码 ——r.apparent_encoding。

r.encoding: 如果header中不存在charset，则默认编码为ISO-8859-1
r.apparent_encoding: 根据网页内容分析出的编码方式

所以，当r.encoding不能解析出正确的网站内容时，我们需要用r.apparent_encoding，来解出相关的编码信息。

这也是为什么我们把r.apparent_encoding赋予r.encoding的时候，我们就能读到r.text里面中文的原因。

#### 1.2. 爬取网页的普通代码框架
普通代码框架：一组代码，可以准确可靠的爬取网页上的内容。

| 异常                      | 说明                                |
| :------------------------|:------------------------------------|
| requests.ConnectionError | 网络连接错误异常，如DNS查询失败，拒绝连接等|
| requests.HTTPError       | HTTP错误异常                          |
| requests.URLRequired     | URL缺失异常                           |
| requests.TooManyRedirects| 超过最大重定向次数，产生重定向异常         |
| requests.ConnectTimeout  | 连接远程服务器超时异常                   |
| requests.Timeout         | 请求URL超时，产生超时异常                |


requests.ConnectionError: 防火墙拒绝连接 
requests.HTTPError:  在http协议层面出现的错误异常
requests.URLRequired:  在对 URL访问的时候，这个URL缺失造成的异常  
requests.TooManyRedirects: 当用户访问的URL，进行重定向，重定向的次数，超过了Requests库允许的最大重定向次数，而产生的异常。经常，是在我们对复杂的链接访问时，所产生的错误。
requests.ConnectTimeout: 在访问远程服务器的时候，由于访问超过了一定时间，而产生的异常。（仅包括连接远程服务器这段时间，产生的异常。）
requests.Timeout: 发起URL请求到获得URL请求内容，**整个**过程中，超过了约定时间，而产生的异常。

| 异常                       | 说明                                |
| :-------------------------|:------------------------------------|
| requests.raise_for_status | 网络连接错误异常，如DNS查询失败，拒绝连接等|

爬取网页的通用代码框架

通用代码框架，可以有效的处理，我们在访问呢或爬取网页过程中，可能出现的一些错误，或者网络不稳定的现象

```Python
import requests
def getHTMLText(url):
    try:
        r = requests.get(url, timeout=30)
        r.raise_for_status() #If the status is not 200, HTTPError occurs.
        r.encoding = r.apparent_encoding
        return r.text
    except: 
        return "Error"import requests
```

```Python
if __name__ == "__main__":
    url = "http://www.baidu.com"
    print(getHTMLText(url))
```



#### 1.3. HTTP协议，与Requests库爬取网页的主要方法

Requests库的7个主要方法
| 方法                      | 说明                                    |
| :------------------------|:----------------------------------------|
| requests.request()       | 构造一个请求，支撑以各方法的基础方法      　　　|
| requests.get()           | 获取HTML网页的主要方法，对应HTTP中的GET 　　　|
| requests.head()          | 获取HTML网页的主要方法，对应HTTP中的HEAD　　　|
| requests.post()          | 获取HTML网页头信息的主要方法，对应HTTP中的POST|
| requests.put()           | 获取HTML网页头信息的主要方法，对应HTTP中的PUT |
| requests.patch()         | 获取HTML网页头信息的主要方法，对应HTTP中的PATCH|
| requests.delete()        | 向HTML网页提交删除请求，对应HTTP中的DELETE   |

HTTP协议 
HTTP, Hypertext Transfer Protocol，超文本传输协议。
HTTP是一个基于"请求与相应"模式的，无状态的应用层协议。


#### 1.3. Requests库主要方法解析

#### 1.4. 网络爬虫引发的问题

## 2. Case study

#### 2.1. Requests.get
```
import requests
url = 'https://www.python.org/dev/peps/pep-0020/'
res = requests.get(url)
text = res.text
text
```

```
with open('zon_of_python.txt', 'w') as f:
    f.write(text[text.find('<pre')+28:text.find('</pre>')-1])
print(text[text.find('<pre')+28:text.find('</pre>')-1])
```

```
import urllib
url = 'https://www.python.org/dev/peps/pep-0020/'
res = urllib.request.urlopen(url).read().decode('utf-8')
print(res[res.find('<pre')+28:res.find('</pre>')-1])
```




#### 2.2. Requests.post

```python
import requests
def translate(word):
    url="http://fy.iciba.com/ajax.php?a=fy"

    data={
        'f': 'auto',
        't': 'auto',
        'w': word,
    }
    
    headers={
        'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.169 Safari/537.36',
    }#User-Agent会告诉网站服务器，访问者是通过什么工具来请求的，如果是爬虫请求，一般会拒绝，如果是用户浏览器，就会应答。
    response = requests.post(url,data=data,headers=headers)     #发起请求
    json_data=response.json()   #获取json数据
    #print(json_data)
    return json_data

def run(word):    
    result = translate(word)['content']['out']   
    print(result)
    return result

def main():
    with open('zon_of_python.txt') as f:
        zh = [run(word) for word in f]

    with open('zon_of_python_zh-CN.txt', 'w') as g:
        for i in zh:
            g.write(i + '\n')

if __name__ == '__main__':
    main()




```


































