# TONGDA OFFICE Anywhere 10.20.200417-XXE

修改导入的xml文件，无回显读取本地文件:
远程vps主机创建xxe.dtd，内容如下所示，读取本地的flag.txt文件：

![image-20200819223543927](C:\Users\jk\AppData\Roaming\Typora\typora-user-images\image-20200819223543927.png)

提交如下poc

![image-20200819223610555](C:\Users\jk\AppData\Roaming\Typora\typora-user-images\image-20200819223610555.png)

远程主机收到请求：

![image-20200819223622585](C:\Users\jk\AppData\Roaming\Typora\typora-user-images\image-20200819223622585.png)

监听端口收到了读取的本地文件内容，内容经过base64编码如下所示：

![image-20200819223635939](C:\Users\jk\AppData\Roaming\Typora\typora-user-images\image-20200819223635939.png)


Base64解码，得到文件内容：

![image-20200819223642616](C:\Users\jk\AppData\Roaming\Typora\typora-user-images\image-20200819223642616.png)

