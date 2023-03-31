**问题：**jsp 页面中无法显示 EL表达式，直接将EL表达式的内容写了出来

![image-20230310171937578](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230310171937578.png)



**原因：**

低版本的servlet默认不支持EL表达式，需要手动设置。



**解决办法：**

在jsp页面的顶端加上如下代码，设置EL表达式可见。

```jsp
<%@ page isELIgnored="false" %>
```



**结果：**

![image-20230310172331317](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230310172331317.png)





