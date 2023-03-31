打开idea下载插件出现超时，网上大部分博客的解决办法是settings ---> Appearance & Behavior --> System Settings --> updates，取消选择Use Secure Connections。

但是idea 2019以后的版本已经没有这个选项了。后来了解到如下方法可以解决。



**解决办法：**

settings ---> Appearance & Behavior --> System Settings --> updates ---> HTTP Proxy ---> Auto-detect proxy settings ---> 在configuration URL 中填写：http:/127.0.0.1:1080，点击Apply，点击OK。

![image-20230302112520913](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230302112520913.png)

然后重启idea，插件可以很快地下载了。

![image-20230302112901650](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230302112901650.png)