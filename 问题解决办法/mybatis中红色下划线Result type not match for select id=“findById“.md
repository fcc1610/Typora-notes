###### 1.问题

Dao层爆红   

![image-20230316171303647](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230316171303647.png)

###### 2. 原因

在同一个项目下多个module有同名的实体类，MyBatisX插件无法识别。

###### 3.解决办法

- 将实体类重命名

- 删掉MyBatisX插件
- 直接运行也没有问题