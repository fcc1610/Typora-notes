BrandMapper is already known to the MapperRegistry.

问题：![image-20230316193604002](C:\Users\10632\AppData\Roaming\Typora\typora-user-images\image-20230316193604002.png)



错误原因：

在mybatis-config.xml文件中重复加载了mapper映射文件（即以下两种方式冲突）

![image-20230316193722761](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230316193722761.png)



解决方式：

删掉其中一个